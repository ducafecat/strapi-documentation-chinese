---
sidebarDepth: 2
---

# 单元测试

在本指南中，我们将看到如何使用测试框架为 Strapi 应用程序运行基本单元测试。

::: tip
在这个例子中，我们将使用 [Jest](https://jestjs.io/) 测试框架，侧重于简单性，使用 [Supertest](https://github.com/visionmedia/supertest) Super-agent 驱动库，使用流畅的 API 测试 node.js 的 HTTP 服务器
:::

::: warning
请注意，由于 Windows 锁定 SQLite 文件的方式，如果您在 Windows 上使用 SQLite 数据库，那么本指南将不起作用
:::

## 安装测试工具

`Jest` 包含一组用于创建和设计测试用例的准则或规则——这是一组实践和工具的组合，旨在帮助测试人员更有效地进行测试。

`Supertest` 允许您测试所有的 api 路由，因为它们是 [http.Server](https://nodejs.org/api/http.md#http_class_http_server) 的实例

`sqlite3` 用于创建磁盘上的数据库，该数据库在测试之间创建和删除。

:::: tabs

::: tab yarn
`yarn add --dev jest supertest sqlite3`
:::

::: tab npm
`npm install jest supertest sqlite3 --save-dev`
:::
::::

完成后，将其添加到 `package.json` 文件中

向 `scripts` 部分添加 `test` 命令

```json
  "scripts": {
    "develop": "strapi develop",
    "start": "strapi start",
    "build": "strapi build",
    "strapi": "strapi",
    "test": "jest --forceExit --detectOpenHandles"
  },
```

然后在文件底部添加这些行

```json
  "jest": {
    "testPathIgnorePatterns": [
      "/node_modules/",
      ".tmp",
      ".cache"
    ],
    "testEnvironment": "node"
  }
```

这将告诉 `Jest` 不要在文件夹中不应该查找测试。

## 引言

### 测试环境

测试框架必须有一个干净的空环境才能执行有效的测试，也不能干扰当前的数据库。

一旦 `jest` 运行，它就会使用 `test` [environment](/developer-docs/latest/setup-deployment-guides/configurations.md#environment) 切换到 `test` ，因此我们需要为此创建一个特殊的环境设置。为 test env 创建一个新配置 `./config/env/test/database.json` 并添加以下值 `"filename": ".tmp/test.db"` ——原因是我们希望有一个单独的 sqlite 数据库用于测试，这样我们的测试就不会触及真正的数据。这个文件将是临时的，每次测试完成后，我们都会删除那个每次在干净的数据库上运行测试的文件。

整个文件看起来像这样:

**Path —** `./config/env/test/database.json`

```json
{
  "defaultConnection": "default",
  "connections": {
    "default": {
      "connector": "bookshelf",
      "settings": {
        "client": "sqlite",
        "filename": ".tmp/test.db"
      },
      "options": {
        "useNullAsDefault": true,
        "pool": {
          "min": 0,
          "max": 1
        }
      }
    }
  }
}
```

### Strapi 实例

为了测试我们需要一个运行在测试环境中的 strapi 实例，基本上我们希望得到 strapi 应用的实例作为对象，类似于为 [process manager](process-manager.md) 创建一个实例。

这些任务需要添加一些文件——让我们创建一个文件夹测试，所有 `tests` 都将放在这个文件夹中，放在文件夹 `helpers` 旁边，而主要的 Strapi helper 将放在文件 Strapi.js 中。

**Path —** `./tests/helpers/strapi.js`

```js
const Strapi = require('strapi');
const http = require('http');

let instance;

async function setupStrapi() {
  if (!instance) {
    /** the following code in copied from `./node_modules/strapi/lib/Strapi.js` */
    await Strapi().load();
    instance = strapi; // strapi is global now
    await instance.app
      .use(instance.router.routes()) // populate KOA routes
      .use(instance.router.allowedMethods()); // populate KOA methods

    instance.server = http.createServer(instance.app.callback());
  }
  return instance;
}
module.exports = { setupStrapi };
```

### 测试 strapi 实例

我们需要一个用于测试的主条目文件，这个文件也将用于测试 helper 文件。

**Path —** `./tests/app.test.js`

```js
const fs = require('fs');
const { setupStrapi } = require('./helpers/strapi');

/** this code is called once before any test is called */
beforeAll(async done => {
  await setupStrapi(); // singleton so it can be called many times
  done();
});

/** this code is called once before all the tested are finished */
afterAll(async done => {
  const dbSettings = strapi.config.get('database.connections.default.settings');

  //delete test database after all tests
  if (dbSettings && dbSettings.filename) {
    const tmpDbFile = `${__dirname}/../${dbSettings.filename}`;
    if (fs.existsSync(tmpDbFile)) {
      fs.unlinkSync(tmpDbFile);
    }
  }
  done();
});

it('strapi is defined', () => {
  expect(strapi).toBeDefined();
});
```

实际上，这就是我们编写单元测试所需的全部内容。只要运行 `yarn test` ，看看你的第一次测试的结果

Actually this is all we need for writing unit tests. Just run `yarn test` and see a result of your first test

```bash
yarn run v1.13.0
$ jest
 PASS  tests/app.test.js
  ✓ strapi is defined (2 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        4.187 s
Ran all test suites.
✨  Done in 5.73s.
```

> Note: 如果您收到 `Jest` 的超时错误，请在`app.test.js` 文件中 `beforeAll` 方法之前添加以下行: `jest.setTimeout(15000)` ，并根据需要调整毫秒值。

### 基本端点控制器测试.

::: tip
在这个示例中，我们将使用来自 [controllers](/developer-docs/latest/development/backend-customization.md#example-2) 部分的 `Hello world` `/hello` 端点。

<!-- the link below is reported to have a missing hash by the check-links plugin, but everything is fine 🤷 -->

:::

有些人可能会说 API 测试不是单元测试，而是有限的集成测试，不管命名法如何，让我们继续测试第一个端点。

我们将测试端点是否正常工作，以及 route `/hello` 是否返回 `Hello World`

让我们创建一个单独的测试文件，`supertest` 将用于检查端点是否按预期工作。

**Path —** `./tests/hello/index.js`

```js
const request = require('supertest');

it('should return hello world', async done => {
  await request(strapi.server) // app server is an instance of Class: http.Server
    .get('/hello')
    .expect(200) // Expect response http code 200
    .then(data => {
      expect(data.text).toBe('Hello World!'); // expect the response text
    });
  done();
});
```

然后将这段代码包含到该文件底部的 `./tests/app.test.js` 中

```js
require('./hello');
```

然后进行 `yarn test`，测试结果应该返回

```bash
➜  my-project yarn test
yarn run v1.13.0
$ jest --detectOpenHandles
 PASS  tests/app.test.js (5.742 s)
  ✓ strapi is defined (4 ms)
  ✓ should return hello world (208 ms)

[2020-05-22T14:37:38.018Z] debug GET /hello (58 ms) 200
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        6.635 s, estimated 7 s
Ran all test suites.
✨  Done in 9.09s.
```

### 测试 `auth` 端点控制器.

在这个场景中，我们将使用两个测试来测试身份验证登录端点

1. 测试 `/auth/local` 这应该登录用户和返回 `jwt` token
2. 测试 `/users/me` 它应该返回用户基于 `Authorization` header 头信息

**Path —** `./tests/user/index.js`

```js
const request = require('supertest');

// user mock data
const mockUserData = {
  username: 'tester',
  email: 'tester@strapi.com',
  provider: 'local',
  password: '1234abc',
  confirmed: true,
  blocked: null,
};

it('should login user and return jwt token', async done => {
  /** Creates a new user and save it to the database */
  await strapi.plugins['users-permissions'].services.user.add({
    ...mockUserData,
  });

  await request(strapi.server) // app server is an instance of Class: http.Server
    .post('/auth/local')
    .set('accept', 'application/json')
    .set('Content-Type', 'application/json')
    .send({
      identifier: mockUserData.email,
      password: mockUserData.password,
    })
    .expect('Content-Type', /json/)
    .expect(200)
    .then(data => {
      expect(data.body.jwt).toBeDefined();
    });

  done();
});

it('should return users data for authenticated user', async done => {
  /** Gets the default user role */
  const defaultRole = await strapi.query('role', 'users-permissions').findOne({}, []);

  const role = defaultRole ? defaultRole.id : null;

  /** Creates a new user an push to database */
  const user = await strapi.plugins['users-permissions'].services.user.add({
    ...mockUserData,
    username: 'tester2',
    email: 'tester2@strapi.com',
    role,
  });

  const jwt = strapi.plugins['users-permissions'].services.jwt.issue({
    id: user.id,
  });

  await request(strapi.server) // app server is an instance of Class: http.Server
    .get('/users/me')
    .set('accept', 'application/json')
    .set('Content-Type', 'application/json')
    .set('Authorization', 'Bearer ' + jwt)
    .expect('Content-Type', /json/)
    .expect(200)
    .then(data => {
      expect(data.body).toBeDefined();
      expect(data.body.id).toBe(user.id);
      expect(data.body.username).toBe(user.username);
      expect(data.body.email).toBe(user.email);
    });

  done();
});
```

然后将这段代码包含到该文件底部的 `./tests/app.test.js` 中

```js
require('./user');
```

上面的所有测试应该返回一个控制台输出，如

```bash
➜  my-project git:(master) yarn test

yarn run v1.13.0
$ jest --forceExit --detectOpenHandles
[2020-05-27T08:30:30.811Z] debug GET /hello (10 ms) 200
[2020-05-27T08:30:31.864Z] debug POST /auth/local (891 ms) 200
 PASS  tests/app.test.js (6.811 s)
  ✓ strapi is defined (3 ms)
  ✓ should return hello world (54 ms)
  ✓ should login user and return jwt token (1049 ms)
  ✓ should return users data for authenticated user (163 ms)

Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        6.874 s, estimated 9 s
Ran all test suites.
✨  Done in 8.40s.
```
