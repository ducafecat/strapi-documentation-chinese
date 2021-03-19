---
sidebarDepth: 3
---

# 配置

应用程序配置位于 `config` 文件夹中。所有配置文件都在启动时加载，可以通过配置提供程序访问。

当你有一个 `./config/server.js` 文件，配置如下:

```js
module.exports = {
  host: '0.0.0.0',
};
```

你可以通过以下方式访问它:

```js
strapi.config.get('server.host', 'defaultValueIfUndefined');
```

使用点符号可以访问嵌套键 `dot-notation` 。

:::tip 注意
文件名用作访问配置的前缀。
:::

## 所需配置

### 数据库

此文件允许您定义用于存储应用程序内容的数据库连接。

::: tip 注意
您可以在本地安装过程中找到 [受支持的数据库和版本](/developer-docs/latest/setup-deployment-guides/installation/cli.md#databases) 。
:::

**Path —** `./config/database.js`.

:::: tabs

::: tab Bookshelf

- `defaultConnection` (string): Connection by default for models which are not related to a specific `connection`. 默认值: `default`.
- `connections` List of all available connections.
  - `default`
    - `connector` (string): Connector used by the current connection. Will be `bookshelf`.
    - `settings` Useful for external session stores such as Redis.
      - `client` (string): Database client to create the connection. `sqlite` or `postgres` or `mysql`.
      - `host` (string): Database host name. 默认值: `localhost`.
      - `port` (integer): Database port.
      - `database` (string): Database name.
      - `username` (string): Username used to establish the connection.
      - `password` (string): Password used to establish the connection.
      - `timezone` (string): Set the default behavior for local time. 默认值: `utc` [Timezone options](https://www.php.net/manual/en/timezones.php).
      - `schema` (string): Set the default database schema. **Used only for Postgres DB.**
      - `ssl` (boolean/object): For ssl database connection. Object is used to pass certificate files as strings.
    - `options` Options used for database connection.
      - `debug` (boolean): Show database exchanges and errors.
      - `autoMigration` (boolean): To disable auto tables/columns creation for SQL database.
      - `pool` Options used for database connection pooling. For default value and more information, look at [Knex's pool config documentation](https://knexjs.org/#Installation-pooling).
        - `min` (integer): Minimum number of connections to keep in the pool.
        - `max` (integer): Maximum number of connections to keep in the pool.
        - `acquireTimeoutMillis` (integer): Maximum time in milliseconds to wait for acquiring a connection from the pool.
        - `createTimeoutMillis` (integer): Maximum time in milliseconds to wait for creating a connection to be added to the pool.
        - `idleTimeoutMillis` (integer): Number of milliseconds to wait before destroying idle connections.
        - `reapIntervalMillis` (integer): How often to check for idle connections in milliseconds.
        - `createRetryIntervalMillis` (integer): How long to idle after a failed create before trying again in milliseconds.

:::

::: tab Mongoose

- `defaultConnection` (string): Connection by default for models which are not related to a specific `connection`. 默认值: `default`.
- `connections` List of all available connections.
  - `default`
    - `connector` (string): Connector used by the current connection. Will be `mongoose`.
    - `settings` Useful for external session stores such as Redis.
      - `client` (string): Database client to create the connection. Will be `mongo`.
      - `host` (string): Database host name. 默认值: `localhost`.
      - `port` (integer): Database port. 默认值: `27017`.
      - `database` (string): Database name.
      - `username` (string): Username used to establish the connection.
      - `password` (string): Password used to establish the connection.
      - `uri` (string): This can overide all previous configurations - _optional_
    - `options` Options used for database connection.
      - `ssl` (boolean): For ssl database connection.
      - `sslCA` (string): Pass content (not filepath!) of server's root CA for ssl connection.
      - `debug` (boolean): Show database exchanges and errors.
      - `authenticationDatabase` (string): Connect with authentication.

:::

::::

:::: tabs

::: tab PostgreSQL

```js
module.exports = ({ env }) => ({
  defaultConnection: 'default',
  connections: {
    default: {
      connector: 'bookshelf',
      settings: {
        client: 'postgres',
        host: env('DATABASE_HOST', 'localhost'),
        port: env.int('DATABASE_PORT', 5432),
        database: env('DATABASE_NAME', 'strapi'),
        username: env('DATABASE_USERNAME', 'strapi'),
        password: env('DATABASE_PASSWORD', 'strapi'),
        schema: env('DATABASE_SCHEMA', 'public'), // Not Required
        ssl: {
          rejectUnauthorized: env.bool('DATABASE_SSL_SELF', false), // For self-signed certificates
        },
      },
      options: {
        ssl: env.bool('DATABASE_SSL', false),
      },
    },
  },
});
```

请注意，如果您需要客户端 SSL CA 验证，您将需要使用带有 fs 模块的 `ssl:{}` 对象将您的 CA 证书转换为字符串。你可以在下面看到一个例子:

```js
module.exports = ({ env }) => ({
  defaultConnection: 'default',
  connections: {
    default: {
      connector: 'bookshelf',
      settings: {
        client: 'postgres',
          ...
        ssl: {
          ca: fs.readFileSync(`${__dirname}/path/to/your/ca-certificate.crt`).toString(),
        },
      },
      options: {
        ssl: true
      },
    },
  },
});
```

:::

::: tab MySQL/MariaDB

```js
module.exports = ({ env }) => ({
  defaultConnection: 'default',
  connections: {
    default: {
      connector: 'bookshelf',
      settings: {
        client: 'mysql',
        host: env('DATABASE_HOST', 'localhost'),
        port: env.int('DATABASE_PORT', 3306),
        database: env('DATABASE_NAME', 'strapi'),
        username: env('DATABASE_USERNAME', 'strapi'),
        password: env('DATABASE_PASSWORD', 'strapi'),
      },
      options: {},
    },
  },
});
```

:::

::: tab SQLite

```js
module.exports = ({ env }) => ({
  defaultConnection: 'default',
  connections: {
    default: {
      connector: 'bookshelf',
      settings: {
        client: 'sqlite',
        filename: env('DATABASE_FILENAME', '.tmp/data.db'),
      },
      options: {
        useNullAsDefault: true,
      },
    },
  },
});
```

:::

::: tab MongoDB

```js
module.exports = ({ env }) => ({
  defaultConnection: 'default',
  connections: {
    default: {
      connector: 'mongoose',
      settings: {
        client: 'mongo',
        host: env('DATABASE_HOST', 'localhost'),
        port: env.int('DATABASE_PORT', 27017),
        database: env('DATABASE_NAME', 'strapi'),
        username: env('DATABASE_USERNAME', 'strapi'),
        password: env('DATABASE_PASSWORD', 'strapi'),
      },
      options: {
        authenticationDatabase: env('AUTHENTICATION_DATABASE'),
        ssl: env('DATABASE_SSL'),
      },
    },
  },
});
```

:::

::::

::: tip
请参阅 [数据库指南](/developer-docs/latest/setup-deployment-guides/configurations.md#databases-installation-guides) 以获得更多细节。
:::

#### 数据库配置

配置文件对多服务器不友好。因此，我们为配置创建了一个数据存储，您希望在生产中进行更新。

##### Get settings

- `environment` (string): 设置要存储数据的环境。默认情况下，它的当前环境(如果您的配置是环境不可知的，则可以是空字符串).
- `type` (string): 设置配置是否为 `api`, `plugin` or `core`. 默认是 `core`.
- `name` (string): 你必须设置插件或 api 名称, 如果 `type` 是 `api` 或者 `plugin`.
- `key` (string, required): 要存储的密钥的名称.

```js
// strapi.store(object).get(object);

// create reusable plugin store variable
const pluginStore = strapi.store({
  environment: strapi.config.environment,
  type: 'plugin',
  name: 'users-permissions',
});

await pluginStore.get({ key: 'grant' });
```

##### Set settings

- `value` (any, required): 要存储的值.

```js
// strapi.store(object).set(object);

// create reusable plugin store variable
const pluginStore = strapi.store({
  environment: strapi.config.environment,
  type: 'plugin',
  name: 'users-permissions'
});

await pluginStore.set({
  key: 'grant',
  value: {
    ...
  }
});
```

#### 数据库安装指南

Strapi 为您提供了为您的项目选择最合适的数据库的选项。它目前支持 **PostgreSQL**, **MongoDB**, **SQLite**, **MySQL** 和 **MariaDB**。下面的文档介绍了如何在本地(出于开发目的)和各种托管服务器或云服务器解决方案(出于登台或生产目的)上安装这些数据库。

::: tip 提示
[部署指南](/developer-docs/latest/setup-deployment-guides/deployment.md) 中包含了部署 Strapi 本身。
:::

<DatabasesLinks>
</DatabasesLinks>

### 服务器

:::: tabs

::: tab 最小

#### 最小化服务器配置

这是用任何新项目创建的默认配置，所有这些键都是必需的，环境配置不需要包含所有这些值，只要它们存在于默认值中。`/config/server.js`.

**Path —** `./config/server.js`.

```js
module.exports = ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
  port: env.int('PORT', 1337),
  admin: {
    auth: {
      secret: env('ADMIN_JWT_SECRET', 'someSecretKey'),
    },
  },
});
```

:::

::: tab 完整

#### 全服务器配置

这是一个完整配置的示例，通常某些密钥不需要出现在环境配置中，并且不需要所有这些密钥。请看下面的表格，看看每个键是做什么的。

**Path —** `./config/server.js`.

```javascript
module.exports = ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
  port: env.int('PORT', 1337),
  socket: '/tmp/nginx.socket', // only use if absolutely required
  emitErrors: false,
  url: env('PUBLIC_URL', 'https://api.example.com'),
  proxy: env.bool('IS_PROXIED', true),
  cron: {
    enabled: env.bool('CRON_ENABLED', false),
  },
  admin: {
    auth: {
      events: {
        onConnectionSuccess(e) {
          console.log(e.user, e.provider);
        },
        onConnectionError(e) {
          console.error(e.error, e.provider);
        },
      },
      secret: env('ADMIN_JWT_SECRET', 'someSecretKey'),
    },
    url: env('PUBLIC_ADMIN_URL', '/dashboard'),
    autoOpen: false,
    watchIgnoreFiles: [
      './my-custom-folder', // Folder
      './scripts/someScript.sh', // File
    ],
    host: 'localhost', // Only used for --watch-admin
    port: 8003, // Only used for --watch-admin
    serveAdminPanel: env.bool('SERVE_ADMIN', true),
    forgotPassword: {
      from: 'no-reply@example.com',
      replyTo: 'no-reply@example.com',
    },
  },
});
```

:::

::::

##### Available options

| Property                                | Description                                                                                                                                                                                                                                                                                                                                                                                                 | Type              | Default                                                                                                                          |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `host`                                  | 主机名                                                                                                                                                                                                                                                                                                                                                                                                      | string            | `localhost`                                                                                                                      |
| `port`                                  | 服务器应在其上运行的端口                                                                                                                                                                                                                                                                                                                                                                                    | integer           | `1337`                                                                                                                           |
| `socket`                                | 提供此选项时，对套接字进行侦听。Listens on a socket. Host and port are cosmetic when this option is provided and likewise use `url` to generate proper urls when using this option. This option is useful for running a server without exposing a port and using proxy servers on the same machine (e.g [Heroku nginx buildpack](https://github.com/heroku/heroku-buildpack-nginx#requirements-proxy-mode)) | string \| integer | `/tmp/nginx.socket`                                                                                                              |
| `emitErrors`                            | 允许将错误发送到 `koa` 当它们发生时，以便附加自定义逻辑或使用错误报告服务.                                                                                                                                                                                                                                                                                                                                  | boolean           | `false`                                                                                                                          |
| `url`                                   | 服务器的公共 url。Public url of the server. Required for many different features (ex: reset password, third login providers etc.). Also enables proxy support such as Apache or Nginx, example: `https://mywebsite.com/api`. The url can be relative, if so, it is used with `http://${host}:${port}` as the base url. An absolute url is however **recommended**.                                          | string            | `''`                                                                                                                             |
| `proxy`                                 | 代理，设置 koa 变量 `app.proxy`. When `true`, proxy header fields will be trusted.                                                                                                                                                                                                                                                                                                                          | boolean           | `false`                                                                                                                          |
| `cron`                                  | 计划任务 Cron configuration (powered by [`node-schedule`](https://github.com/node-schedule/node-schedule))                                                                                                                                                                                                                                                                                                  | Object            |                                                                                                                                  |
| `cron.enabled`                          | 是否启用计划任务 启用或禁用 CRON tasks to schedule jobs at specific dates.                                                                                                                                                                                                                                                                                                                                  | boolean           | `false`                                                                                                                          |
| `admin`                                 | 管理面板配置                                                                                                                                                                                                                                                                                                                                                                                                | Object            |                                                                                                                                  |
| `admin.auth`                            | 认证配置                                                                                                                                                                                                                                                                                                                                                                                                    | Object            |                                                                                                                                  |
| `admin.auth.secret`                     | 用于对 JWT 令牌进行编码                                                                                                                                                                                                                                                                                                                                                                                     | string            | `undefined`                                                                                                                      |
| `admin.auth.events`                     | 为身份验证注册的所有事件订阅者的记录                                                                                                                                                                                                                                                                                                                                                                        | object            | `{}`                                                                                                                             |
| `admin.auth.events.onConnectionSuccess` | 当管理用户成功登录到管理面板时调用                                                                                                                                                                                                                                                                                                                                                                          | function          | `undefined`                                                                                                                      |
| `admin.auth.events.onConnectionError`   | 当管理员用户未能登录到管理面板时调用                                                                                                                                                                                                                                                                                                                                                                        | function          | `undefined`                                                                                                                      |
| `admin.url`                             | 管理面板的网址。Default value: `/admin`. Note: If the url is relative, it will be concatenated with `url`.                                                                                                                                                                                                                                                                                                  | string            | `/admin`                                                                                                                         |
| `admin.autoOpen`                        | 启动时启用或禁用管理打开                                                                                                                                                                                                                                                                                                                                                                                    | boolean           | `true`                                                                                                                           |
| `admin.watchIgnoreFiles`                | 添加在开发过程中不应该被监视的自定义文件. See more [这里](https://github.com/paulmillr/chokidar#path-filtering) (property `ignored`).                                                                                                                                                                                                                                                                       | Array(string)     | `[]`                                                                                                                             |
| `admin.host`                            | 对管理面板使用不同的主机. Only used along with `strapi develop --watch-admin`                                                                                                                                                                                                                                                                                                                               | string            | `localhost`                                                                                                                      |
| `admin.port`                            | 对管理面板使用不同的端口. Only used along with `strapi develop --watch-admin`                                                                                                                                                                                                                                                                                                                               | string            | `8000`                                                                                                                           |
| `admin.serveAdminPanel`                 | 如果为 false，将不会提供管理面板. Note: the `index.html` will still be served, see [defaultIndex option](/developer-docs/latest/setup-deployment-guides/configurations.md#global-middlewares)                                                                                                                                                                                                               | boolean           | `true`                                                                                                                           |
| `admin.forgotPassword`                  | 设置自定义忘记密码的电子邮件 (更多信息: [Forgot Password Email](/developer-docs/latest/development/admin-customization.md#forgot-password-email))                                                                                                                                                                                                                                                           | Object            | {}                                                                                                                               |
| `admin.forgotPassword.emailTemplate`    | 定义的电子邮件模板 [email plugin](/developer-docs/latest/development/plugins/email.md#programmatic-usage)                                                                                                                                                                                                                                                                                                   | Object            | [Default template](https://github.com/strapi/strapi/tree/master/packages/strapi-admin/config/email-templates/forgot-password.js) |
| `admin.forgotPassword.from`             | 发件人邮件地址                                                                                                                                                                                                                                                                                                                                                                                              | string            | 默认值 defined in your [provider configuration](/developer-docs/latest/development/plugins/email.md#configure-the-plugin)        |
| `admin.forgotPassword.replyTo`          | 要求接收者回复的默认地址或地址                                                                                                                                                                                                                                                                                                                                                                              | string            | 默认值 defined in your [provider configuration](/developer-docs/latest/development/plugins/email.md#configure-the-plugin)        |

### 格式

您可以使用 `.js` 或 `.json` 文件来配置您的应用程序。

当使用 `.js` 时，你可以导出一个对象:

```js
module.exports = {
  mySecret: 'someValue',
};
```

或者一个返回配置对象的函数(推荐用法)。该函数将访问 `env` [ utility](#casting-environment-variables) 。

```js
module.exports = ({ env }) => {
  return {
    mySecret: 'someValue',
  };
};
```

## 可选配置

### 环境

如果您需要特定环境的特定静态配置，并且使用环境变量会变得单调乏味，那么可以在 `./config/env/{env}/{filename}` 。

这些配置将合并到 `./config` 文件夹。该环境基于 `NODE_ENV` 环境变量(默认为 `development`)。

当使用 `NODE_ENV=production` 启动 Strapi 时，它将从 `./config/*` 和 `./config/env/production/*` 生产配置中定义的所有内容都将覆盖默认配置。

结合环境变量，这种模式变得非常强大。

例子:

`./config/server.js`

```js
module.exports = {
  host: '127.0.0.1',
};
```

`./config/env/production/server.js`

```js
module.exports = ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
});
```

启动应用程序时:

```bash
yarn start
# uses host 127.0.0.1
```

```bash
NODE_ENV=production yarn start
# uses host 0.0.0.0
```

```bash
HOST=10.0.0.1 NODE_ENV=production yarn start
# uses host 10.0.0.1
```

### 环境变量

#### Strapi 环境变量列表

有些设置只能通过环境变量进行修改。下面是这些设置的相关环境变量名称:

| name                                 | description                                                     | type    | default         |
| ------------------------------------ | --------------------------------------------------------------- | ------- | --------------- |
| `STRAPI_DISABLE_UPDATE_NOTIFICATION` | 不要在终端显示关于更新 strapi 的通知消息                        | boolean | `false`         |
| `STRAPI_HIDE_STARTUP_MESSAGE`        | 不要在终端显示启动消息                                          | boolean | `false`         |
| `STRAPI_TELEMETRY_DISABLED`          | 不要向 Strapi 发送使用数据                                      | boolean | `false`         |
| `STRAPI_LOG_TIMESTAMP`               | 在日志中添加时间戳信息                                          | boolean | `false`         |
| `STRAPI_LOG_LEVEL`                   | 选择日志级别 `fatal`, `error`, `warn`, `info`, `debug`, `trace` | string  | `'info'`        |
| `STRAPI_LOG_FORCE_COLOR`             | 强制颜色即使在不允许启用颜色的环境中也要显示(例如: 在 TTY 之外) | boolean | `true`          |
| `STRAPI_LOG_PRETTY_PRINT`            | 日志行显示为文本而不是对象                                      | boolean | `true`          |
| `STRAPI_LICENSE`                     | 激活企业版的许可证密钥                                          | string  | `undefined`     |
| `NODE_ENV`                           | 应用程序运行的环境类型                                          | string  | `'development'` |
| `BROWSER`                            | 启动后打开浏览器中的管理面板                                    | boolean | `true`          |
| `ENV_PATH`                           | 包含环境变量的文件的路径                                        | string  | `'./.env'`      |

#### 使用环境变量的配置

在大多数情况下，环境之间会有不同的配置。例如: 数据库凭据。

您可以在配置文件中定义这些变量，而不是将这些凭据写入配置文件。在应用程序的根目录下创建一个 `.env` 文件。

例子:\*\*

**Path —** `.env`

```
DATABASE_PASSWORD=acme
```

如果要自定义。在启动应用程序之前，你可以设置一个叫做 `ENV_PATH` 的环境变量文件 `.env`:

```sh
$ ENV_PATH=/absolute/path/to/.env npm run start
```

现在，您可以在配置文件和应用程序中访问这些变量。可以使用 `process.env.{varName}` 在任何地方访问这些变量。

在您的配置文件中，您可以访问一个 `env` 实用程序，该实用程序允许定义默认值和强制转换值。

**Path —** `./config/database.js`

```js
module.exports = ({ env }) => ({
  connections: {
    default: {
      settings: {
        password: env('DATABASE_PASSWORD'),
      },
    },
  },
});
```

#### 铸造环境变量

```js
// Returns the env if defined without casting it
env('VAR', 'default');

// Cast to int (using parseInt)
env.int('VAR', 0);

// Cast to float (using parseFloat)
env.float('VAR', 3.14);

// Cast to boolean (check if the value is equal to 'true')
env.bool('VAR', true);

// Cast to js object (using JSON.parse)
env.json('VAR', { key: 'value' });

// Cast to an array (syntax: ENV_VAR=[value1, value2, value3] | ENV_VAR=["value1", "value2", "value3"])
env.array('VAR', [1, 2, 3]);

// Case to date (using new Date(value))
env.date('VAR', new Date());
```

### API

**Path —** `./config/api.js`.

```js
module.exports = ({ env }) => ({
  responses: {
    privateAttributes: ['_v', 'id', 'created_at'],
  },
  rest: {
    defaultLimit: 100,
    maxLimit: 250,
  },
});
```

#### 可选方案

| Property                      | Description                                                                                                    | Type         | Default |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------- | ------------ | ------- |
| `responses`                   | 全局 API 响应配置                                                                                              | Object       |         |
| `responses.privateAttributes` | E.g. 一组全局定义的属性作为私有属性处理 `_v` 当使用 MongoDb 或时间戳时 `created_at`, `updated_at` 可以视为私有 | String array | `[]`    |
| `rest`                        | REST API 配置                                                                                                  | Object       |         |
| `rest.defaultLimit`           | 指定默认值 `_limit` API 调用中使用的参数                                                                       | Integer      | `100`   |
| `rest.maxLimit`               | 指定可以请求的最大允许数字 `_limit`. 默认为 `null` 所有结果都会出来                                            | Integer      | `null`  |

### 插件

插件就像一个小型的独立子应用程序。它有自己的业务逻辑，包括专用模型、控制器、服务、中间件或钩子。它还可以在管理面板中集成自己的 UI。

::: tip

更多信息请参考 [插件文档](/developer-docs/latest/development/local-plugins-customization.md) 。
:::

### 钩子

钩子是向核心添加功能的模块，它们在服务器引导期间加载。

#### 结构

##### 文件结构

```js
module.exports = strapi => {
  const hook = {
    /**
     * Default options
     */

    defaults: {
      // config object
    },

    /**
     * Initialize the hook
     */

    async initialize() {
      // await someAsyncCode()
      // const settings = {...this.defaults, ...strapi.config.hook.settings.**};
    },
  };

  return hook;
};
```

- `defaults` (object): 包含默认配置.
- `initialize` (function): 在服务器引导期间调用.

钩子的 [配置](#configuration-and-activation) 可以通过 `strapi.config.hook.settings.**` 访问。

钩子可以通过 `strapi.hook` 变量访问。

##### 节点模块

在 `./node_modules` 文件夹中，遵循此名称模式 `strapi-hook-*` 的每个文件夹都将作为钩子加载。

钩子需要遵循下面的结构:

```
/strapi-hook-[...]
└─── lib
     - index.js
- LICENSE.md
- package.json
- README.md
```

`index.js` 是钩子的入口点，它应该看起来像上面的例子。

##### 自定义本地钩子

框架允许直接从项目加载钩子，而不必从 npm 安装钩子。这是一个很好的方法，可以利用钩子系统的特性来编写不需要在应用程序之间共享的代码。要做到这一点，你必须创建一个 `./hooks` 文件夹放在项目的根目录下，并将 hooks 放入其中。

```
/project
└─── admin
└─── api
└─── config
│    - hook.js
└─── hooks
│   └─── strapi-documentation
│        - index.js
│   └─── strapi-server-side-rendering
│        - index.js
└─── public
- favicon.ico
- package.json
- server.js
```

#### 配置和激活

要使用自定义选项激活和配置钩子，您需要添加/编辑您的 `./config/hook.js` 文件放在 Strapi 应用程序中。钩子特定的 `timeout` 值将覆盖全局超时值，默认超时值为 1000 毫秒。

```js
module.exports = {
  timeout: 2000,
  settings: {
    'hook-name': {
      enabled: true,
      timeout: 3000,
    },
  },
};
```

### 中间件

中间件是根据请求以类似堆栈的方式组合和执行的函数。如果您不熟悉 Koa 的中间件堆栈，我们强烈建议您阅读 [Koa 介绍文档](http://koajs.com/#introduction) 。

#### 结构

##### 文件结构

```js
module.exports = strapi => {
  return {
    // can also be async
    initialize() {
      strapi.app.use(async (ctx, next) => {
        // await someAsyncCode()

        await next();

        // await someAsyncCode()
      });
    },
  };
};
```

- `initialize` (function): 在服务器引导期间调用.

中间件可以通过 `strapi.middleware` 变量访问。

##### NodeJS 模块

在 `./node_modules` 文件夹中，遵循此名称模式的 `strapi-middleware-*` 每个文件夹都将作为中间件加载。

中间件需要遵循以下结构:

```
/middleware
└─── lib
     - index.js
- LICENSE.md
- package.json
- README.md
```

`index.js` 是您的中间件的入口点，它应该类似于上面的例子。

##### 自定义中间件

该框架允许应用程序覆盖默认中间件并添加新中间件。你必须创建一个 `./middlewares` 文件夹放在项目的根目录下，并将中间件放入其中。

```
/project
└─── api
└─── config
└─── middlewares
│   └─── responseTime // It will override the core default responseTime middleware.
│        - index.js
│   └─── views // It will be added into the stack of middleware.
│        - index.js
└─── public
- favicon.ico
- package.json
- server.js
```

每一个中间件都将被注入到 Koa 堆栈中。要管理加载订单，请参阅 [中间件订单部分](#load-order) 。

#### 配置和激活

要配置应用程序的中间件，需要在 Strapi 应用程序中创建或编辑 `./config/middleware.js` 文件。

默认情况下，这个文件不存在，您必须创建它。

可选方案

- `timeout` (integer): 定义加载中间件所允许的最大毫秒.
- `load` (Object): 请参阅详细信息 [这里](#load-order)
- `settings` (Object): 每个中间件的配置
  - `{middlewareName}` (Object): 配置一个中间件
    - `enabled` (boolean): 告诉 Strapi 是否运行中间件

##### 设置

例子:

**Path —** `./config/middleware.js`.

```js
module.exports = {
  //...
  settings: {
    cors: {
      origin: ['http://localhost', 'https://mysite.com', 'https://www.mysite.com'],
    },
  },
};
```

##### 加载顺序

中间件被异步注入到 Koa 堆栈中。有时候这些中间件需要按照特定的顺序加载。若要定义加载顺序，请创建或编辑文件 `./config/middleware.js` .

**Path —** `./config/middleware.js`.

```js
module.exports = {
  load: {
    before: ['responseTime', 'logger', 'cors', 'responses'],
    order: [
      "Define the middlewares' load order by putting their name in this array in the right order",
    ],
    after: ['parser', 'router'],
  },
};
```

- `load`:
  - `before`: 首先需要加载的中间件数组。这个数组的顺序很重要.
  - `order`: 需要按特定顺序加载的中间件数组.
  - `after`: 需要在堆栈末尾加载的中间件数组。这个数组的顺序很重要.

#### 核心中间件配置

Strapi 的核心包括了一小部分中间件，用于性能、安全和大错误处理。

- boom
- cors
- cron
- csp
- favicon
- gzip
- hsts
- ip
- language
- logger
- p3p
- parser
- public
- responses
- responseTime
- router
- session
- xframe
- xss

::: tip

无法禁用下列中间件: responses, router, logger and boom.
:::

##### 全局中间件

- `favicon`
  - `path` (string): 图标文件的路径. 默认值: `favicon.ico`.
  - `maxAge` (integer): Cache-control max-age 指令. 默认值 毫秒: `86400000`.
- `public`
  - `path` (string): 公用文件夹的路径: `./public`.
  - `maxAge` (integer): Cache-control max-age 指令. 默认值 毫秒: `60000`.
  - `defaultIndex` (boolean): 将缺省索引页面显示为 `/` 和 `/index.html`. 默认值: `true`.

##### 请求中间件

- `session`
  - `enabled` (boolean): 启用或禁用会话. 默认值: `false`.
- `logger`
  - `level` (string): 默认日志级别. 默认值: `debug`.
  - `exposeInContext` (boolean): Expose logger in context so it can be used through `strapi.log.info(‘my log’)`. 默认值: `true`.
  - `requests` (boolean): 在上下文中暴露日志记录器. 默认值: `false`.
- `parser` (See [koa-body](https://github.com/dlau/koa-body#options) 了解详情)
  - `enabled`(boolean): 启用或禁用 解析器. 默认值: `true`.
  - `multipart` (boolean): 启用或禁用 multipart bodies 解析. 默认值: `true`.
  - `jsonLimit` (string|integer): The byte (if integer) 限制 JSON body. 默认值: `1mb`.
  - `formLimit` (string|integer): The byte (if integer) 限制 form body. 默认值: `56k`.
  - `queryStringParser` (看 [qs](https://github.com/ljharb/qs) 以获取完整的选项列表).
    - `arrayLimit` (integer): 查询字符串中数组的最大长度。索引大于限制的任何数组成员都将转换为索引作为键的对象。默认值: `100`.
    - `depth` (integer): 嵌套查询字符串对象的最大解析深度. 默认值: `20`.

::: tip

这个会话在 `mongo` 作为客户机时不起作用。我们应该使用的包现在已经坏了。
:::

##### 响应中间件

- [`gzip`](https://en.wikipedia.org/wiki/Gzip)
  - `enabled` (boolean): 启用或不启用 GZIP 响应压缩.
  - `options` (Object): 允许从 [koa-compress](https://github.com/koajs/compress#options).
- `responseTime`
  - `enabled` (boolean): 允许或不允许 `X-Response-Time header` to response. 默认值: `false`.
- `poweredBy`
  - `enabled` (boolean): 允许或不允许 `X-Powered-By` header to response. 默认值: `true`.
  - `value` (string): 头部的值. 默认值: `Strapi <strapi.io>`

::: tip

通过 `koa-compress` 压缩的 `gzip` 在默认情况下使用 [Brotli](https://en.wikipedia.org/wiki/Brotli) ，但在大多数情况下没有使用合理的默认配置。如果您在启用 `gzip` 时遇到响应速度慢的问题，可以考虑通过传递 `{br: false}` 作为一个选项来禁用 Brotli。还可以使用 `{br: { params: { // YOUR PARAMS HERE } }}` 传递更合理的 PARAMS
:::

##### 安全中间件

- [`csp`](https://en.wikipedia.org/wiki/Content_Security_Policy)
  - `enabled` (boolean): 启用或禁用 CSP 以避免 XSS 跨网站脚本攻击和数据注入攻击.
  - `policy` (string): 配置 `Content-Security-Policy` 响应头. 如果没有指定，则使用默认值: `undefined`.
- [`p3p`](https://en.wikipedia.org/wiki/P3P)
  - `enabled` (boolean): 启用或禁用 p3p.
- [`hsts`](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)
  - `enabled` (boolean): 启用或禁用 HSTS.
  - `maxAge` (integer): 生效的秒数 HSTS. 默认值: `31536000`.
  - `includeSubDomains` (boolean): 将 HSTS 应用于主机的所有子域. 默认值: `true`.
- [`xframe`](https://en.wikipedia.org/wiki/Clickjacking)
  - `enabled` (boolean): 启用或禁用 `X-FRAME-OPTIONS` 响应头.
  - `value` (string): 头部的值, e.g. DENY, SAMEORIGIN or ALLOW-FROM uri. 默认值: `SAMEORIGIN`.
- [`xss`](https://en.wikipedia.org/wiki/Cross-site_scripting)
  - `enabled` (boolean): 启用或禁用 XSS 来防止旧版 IE 浏览器(IE8)中的跨网站脚本攻击.
- [`cors`](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)
  - `enabled` (boolean): 启用或禁用 CORS 以防止您的服务器从其他域请求.
  - `origin` (string or array): Allowed URLs (`http://example1.com, http://example2.com`, `['http://www.example1.com', 'http://example1.com']` or allows everyone `*`). 默认值: `*`.
  - `expose` (array): 配置 `Access-Control-Expose-Headers` CORS header. If not specified, no custom headers are exposed. 默认值: `["WWW-Authenticate", "Server-Authorization"]`.
  - `maxAge` (integer): 配置 `Access-Control-Max-Age` CORS header. 默认值: `31536000`.
  - `credentials` (boolean): 配置 `Access-Control-Allow-Credentials` CORS header. 默认值: `true`.
  - `methods` (array)|String - 配置 `Access-Control-Allow-Methods` CORS header. 默认值: `["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS", "HEAD"]`.
  - `headers` (array): 配置 `Access-Control-Allow-Headers` CORS header. If not specified, defaults to reflecting the headers specified in the request's Access-Control-Request-Headers header. 默认值: `["Content-Type", "Authorization", "X-Frame-Options"]`.
- `ip`
  - `enabled` (boolean): 启用或禁用 IP blocker. 默认值: `false`.
  - `whiteList` (array): Whitelisted IPs. 默认值: `[]`.
  - `blackList` (array): Blacklisted IPs. 默认值: `[]`.

#### 例子

创建自定义中间件.

**Path —** `./middlewares/timer/index.js`

```js
module.exports = strapi => {
  return {
    initialize() {
      strapi.app.use(async (ctx, next) => {
        const start = Date.now();

        await next();

        const delta = Math.ceil(Date.now() - start);

        ctx.set('X-Response-Time', delta + 'ms');
      });
    },
  };
};
```

在环境设置中启用中间件.

首先加载一个中间件

**Path —** `./config/middleware.js`

```js
module.exports = {
  load: {
    before: ['timer', 'responseTime', 'logger', 'cors', 'responses', 'gzip'],
    order: [
      "Define the middlewares' load order by putting their name in this array is the right order",
    ],
    after: ['parser', 'router'],
  },
  settings: {
    timer: {
      enabled: true,
    },
  },
};
```

### 功能

`./config/functions/` 包含一组 JavaScript 文件，以便添加动态和基于逻辑的配置。

可以通过 `strapi.config.functions['fileName']();` 访问该文件夹中公开的所有函数;

<!-- The text above will be identified as a broken link by the check-links VuePress plugin, because its syntax looks like an empty link. You can safely ignore the error. -->

#### Bootstrap

**Path —** `./config/functions/bootstrap.js`.

`bootstrap` 函数在每个服务器启动时调用。您可以使用它在服务器生命周期的此时添加特定的逻辑。

下面是一些用例:

- 创建一个管理员用户，如果没有的话.
- 用一些必要的数据填充数据库.
- 加载一些环境变量.

引导函数可以是同步或异步的.

**同步**

```js
module.exports = () => {
  // some sync code
};
```

**返回 promise 函数**

```js
module.exports = () => {
  return new Promise(/* some code */);
};
```

**异步**

```js
module.exports = async () => {
  await someSetup();
};
```

#### CRON 任务

CRON 任务允许您使用可选的循环规则为特定日期的执行调度作业(任意函数)。它在任何给定的时间只使用一个计时器(而不是每秒/分钟重新评估即将到来的作业)。

此功能由 [`node-schedule`](https://www.npmjs.com/package/node-schedule) 模块提供，点击了解详情。

::: warning

确保 `enabled` 的 cron 配置在 `./config/server.js` 文件中设置为 true。
:::

Cron 格式:

```
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    |
│    │    │    │    │    └ day of week (0 - 7) (0 or 7 is Sun)
│    │    │    │    └───── month (1 - 12)
│    │    │    └────────── day of month (1 - 31)
│    │    └─────────────── hour (0 - 23)
│    └──────────────────── minute (0 - 59)
└───────────────────────── second (0 - 59, OPTIONAL)
```

要定义 CRON 作业，添加如下逻辑:

**Path —** `./config/functions/cron.js`.

```js
module.exports = {
  /**
   * Simple example.
   * Every monday at 1am.
   */

  '0 0 1 * * 1': () => {
    // Add your own logic here (e.g. send a queue of email, create a database backup, etc.).
  },
};
```

如果您的 CRON 任务需要基于特定的时区运行，那么您可以像下面这样配置任务:

```js
module.exports = {
  /**
   * CRON task with timezone example.
   * Every monday at 1am for Asia/Dhaka timezone.
   * List of valid timezones: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
   */

  '0 0 1 * * 1': {
    task: () => {
      // Add your own logic here (e.g. send a queue of email, create a database backup, etc.).
    },
    options: {
      tz: 'Asia/Dhaka',
    },
  },
};
```

#### 数据库 ORM 定制

当出现这些插件时，它们会被加载，以便您可以自定义数据库连接实例，例如添加一些插件、自定义参数等。

你需要使用正常的 `npm install the-plugin-name` 或者其他支持的包工具(比如 yarn)来安装插件，然后按照下面的例子来加载它们。

:::: tabs

::: tab Mongoose

举个例子，对于使用 MongoDB 的 `mongoose-simple-random` 插件，你可以这样注册:

**Path —** `./config/functions/mongoose.js`.

```js
'use strict';

const random = require('mongoose-simple-random');

module.exports = (mongoose, connection) => {
  mongoose.plugin(random);
};
```

:::

::: tab Bookshelf

另一个例子是为 MySQL 使用 `bookshelf-uuid` 插件，你可以这样注册它:

**Path —** `./config/functions/bookshelf.js`.

```js
'use strict';

module.exports = (bookshelf, connection) => {
  bookshelf.plugin('bookshelf-uuid');
};
```

:::

::::

### 公共资产

公共资产是静态文件，如 images, video, css, etc，您希望外部世界可以访问这些文件。

因为 API 可能需要为静态资产服务，所以每个新的 Strapi 项目默认都包含一个名为 `/public` 的文件夹。如果请求的路径与任何其他已定义的路由不匹配，并且与公共文件名匹配，则可以访问位于此目录中的任何文件。

例子:

可以通过 `/company-logo.png` URL 访问名为 `company-logo.png` 在 `./public/` 目录下的图像。

::: tip

如果请求对应于一个文件夹名称( `/pictures` url 将尝试提供 `public/pictures/index.html` 文件) ，那么将提供 `index.html` 文件。
:::

::: warning

`.` 开头的都是隐藏文件，不对外公开。例如 `.htaccess` 或 `.gitignore` 不能访问。
:::

### 单点登录 <GoldBadge link="https://strapi.io/pricing/" withLinkIcon />

---

Strapi 上的单点登录允许您为管理面板配置额外的登录和注册方法。

::: warning 注意

目前不可能将唯一的 SSO 提供者与用于 Strapi 帐户的电子邮件地址关联，这意味着对 Strapi 帐户的访问不能仅限于一个 SSO 提供者。有关解决这个问题的更多信息和解决方法，请参考 [专门的 GitHub issue](https://github.com/strapi/strapi/issues/9466#issuecomment-783587648) 。
:::

#### 先决条件

- 需要在 3.5.0 或更高版本上运行 Strapi 应用程序.
- 要在应用程序上配置 SSO，您将需要一个带有 Gold 计划的 EE 许可证.
- 确保 Strapi 是您可以通过供应商访问的应用程序的一部分。例如，使用 Microsoft (Azure) Active Directory，您必须首先询问具有正确权限的人，以便将 Strapi 添加到允许的应用程序列表中。请参考您的提供商文档以了解更多相关信息

#### 用法

SSO 配置存在于 `/config/server.js` 中应用程序的服务器配置中。

##### 访问配置

提供者的配置应该在服务器配置的 `admin.auth.providers` 路径中编写。

`admin.auth.providers` 是以数组的方式配置 [provider 配置](#provider-configuration).

```javascript
module.exports = ({ env }) => ({
  // ...
  admin: {
    // ...
    auth: {
      providers: [], // The providers' configuration lives there
    },
  },
});
```

##### Provider 配置

Provider 提供者的配置是一个 Javascript 对象，它具有以下属性:

| Name             | Required | Type     | Description                                                  |
| ---------------- | -------- | -------- | ------------------------------------------------------------ |
| `uid`            | true     | string   | 策略的 UID。它必须匹配策略的名称                             |
| `displayName`    | true     | string   | 将在登录页面上用于引用提供程序的名称                         |
| `icon`           | false    | string   | 一个图像 URL。如果指定，它将替换登录页面上的 `displayName`   |
| `createStrategy` | true     | function | 工厂函数，它将为您的供应商构建并返回一个新的 `passport` 策略 |

::: tip

`uid` 属性是每个 strategy 的唯一标识符，通常可以在 strategy 包中找到。如果您不确定它指的是什么，请与策略的维护人员联系。
:::

###### The `createStrategy` Factory

Passport 策略通常使用两个参数实例化它: 配置对象和验证函数。

<!-- Title below is supposed to be an h7, so one level deeper than "The `createStrategy` Factory. But h7 is not a thing, so using bold instead. 🤷 -->

**配置对象**

配置对象取决于 strategy 需求，但通常要求在提供者端建立连接后将回调 URL 重定向到。

可以使用 `getStrategyCallbackURL` 方法为提供程序生成特定的回调 URL。这个 URL 也需要写在提供者端，以便允许从它重定向。

回调 URL 的格式如下: `/admin/connect/<provider_uid>` 。

::: tip

`strapi.admin.services.passport.getStrategyCallbackURL` 是一个 Strapi 帮助器，您可以使用它获取特定提供者的回调 URL。它接受提供程序名称作为参数并返回 URL。
:::

如果需要，这也是为 OAuth2 应用程序放置客户机 ID 和密钥的地方。

**验证功能**

这里使用 verify 函数作为中间件，允许用户对从提供者 API 返回的数据进行转换和额外处理。

这个函数总是以一个 `done` 方法作为最后的参数，用于将需要的数据传输到 SSO 的 Strapi 层。

它的签名如下:`void done(error: any, data: object);` 它遵循以下规则:

- 如果 `error` 没有设置 `null`, 然后发送的数据被忽略，控制器将抛出一个错误.
- 如果 SSO 的自动注册特性被禁用，则 `data` 对象只需要由 `email` 适配.
- 如果启用了 SSO 的自动注册功能，那么您将需要（除了 `email` 之外）定义一个 `username` 属性，或者在 `data` 对象中定义 `firstname` 和 `lastname` 。

###### 添加一个 provider

添加新的提供程序意味着为管理员添加登录的新方法。

为了获得更大的灵活性和更多的供应商选择，Strapi 使用了 [Passport.js](http://www.passportjs.org/) 。因此，任何不需要额外自定义数据的有效护照策略都应该与 Strapi 协同工作。

::: warning

像 [ldapauth](https://github.com/vesse/passport-ldapauth) 这样的策略不能立即生效，因为它们需要从管理面板发送额外的数据。如果希望向应用程序添加 LDAP 提供程序，则需要编写 [custom strategy](http://www.passportjs.org/packages/passport-custom/) 。你也可以使用 `Okta` 和 `Auth0` 等服务作为桥接服务。
:::

###### 配置 provider

要配置提供者，请遵循以下步骤:

1. 请确保从已安装的包或本地文件导入服务器配置文件中的策略.
2. 你需要添加一个新项到 `admin.auth.providers` 在服务器配置中使用匹配 [format given above](#provider-configuration)
3. 重新启动应用程序，提供商应该出现在您的管理员登录页面.

##### 例子

:::::: tabs

::::: tab Google

使用: [passport-google-oauth2](https://github.com/mstade/passport-google-oauth2)

:::: tabs

::: tab yarn

```bash
yarn add passport-google-oauth2
```

:::

::: tab npm

```bash
npm install --save passport-google-oauth2
```

::::

`/config/server.js`

```jsx
'use strict';

const GoogleStrategy = require('passport-google-oauth2');

module.exports = ({ env }) => ({
  // ...
  admin: {
    // ...
    auth: {
      /// ...
      providers: [
        {
          uid: 'google',
          displayName: 'Google',
          icon: 'https://cdn2.iconfinder.com/data/icons/social-icons-33/128/Google-512.png',
          createStrategy: strapi =>
            new GoogleStrategy(
              {
                clientID: env('GOOGLE_CLIENT_ID'),
                clientSecret: env('GOOGLE_CLIENT_SECRET'),
                scope: [
                  'https://www.googleapis.com/auth/userinfo.email',
                  'https://www.googleapis.com/auth/userinfo.profile',
                ],
                callbackURL: strapi.admin.services.passport.getStrategyCallbackURL('google'),
              },
              (request, accessToken, refreshToken, profile, done) => {
                done(null, {
                  email: profile.email,
                  firstname: profile.given_name,
                  lastname: profile.family_name,
                });
              }
            ),
        },
      ],
    },
  },
});
```

:::::

::::: tab Github

使用: [passport-github](https://github.com/cfsghost/passport-github)

:::: tabs

::: tab yarn

```bash
yarn add passport-github2
```

:::

::: tab npm

```bash
npm install --save passport-github2
```

:::

::::

`/config/server.js`

```jsx
'use strict';

const GithubStrategy = require('passport-github2');

module.exports = ({ env }) => ({
  // ...
  admin: {
    // ...
    auth: {
      // ...
      providers: [
        {
          uid: 'github',
          displayName: 'Github',
          icon: 'https://cdn1.iconfinder.com/data/icons/logotypes/32/github-512.png',
          createStrategy: strapi =>
            new GithubStrategy(
              {
                clientID: env('GITHUB_CLIENT_ID'),
                clientSecret: env('GITHUB_CLIENT_SECRET'),
                scope: ['user:email'],
                callbackURL: strapi.admin.services.passport.getStrategyCallbackURL('github'),
              },
              (accessToken, refreshToken, profile, done) => {
                done(null, {
                  email: profile.emails[0].value,
                  username: profile.username,
                });
              }
            ),
        },
      ],
    },
  },
});
```

:::::

::::: tab Discord

使用: [passport-discord](https://github.com/nicholastay/passport-discord#readme)

:::: tabs

::: tab yarn

```bash
yarn add passport-discord
```

:::

::: tab npm

```bash
npm install --save passport-discord
```

:::

::::

`/config/server.js`

```jsx
'use strict';

const DiscordStrategy = require('passport-discord');

module.exports = ({ env }) => ({
  // ...
  admin: {
    // ...
    auth: {
      // ...
      providers: [
        {
          uid: 'discord',
          displayName: 'Discord',
          icon: 'https://cdn0.iconfinder.com/data/icons/free-social-media-set/24/discord-512.png',
          createStrategy: strapi =>
            new DiscordStrategy(
              {
                clientID: env('DISCORD_CLIENT_ID'),
                clientSecret: env('DISCORD_SECRET'),
                callbackURL: strapi.admin.services.passport.getStrategyCallbackURL('discord'),
                scope: ['identify', 'email'],
              },
              (accessToken, refreshToken, profile, done) => {
                done(null, {
                  email: profile.email,
                  username: `${profile.username}#${profile.discriminator}`,
                });
              }
            ),
        },
      ],
    },
  },
});
```

:::::
::::: tab Microsoft

使用: [passport-azure-ad-oauth2](https://github.com/auth0/passport-azure-ad-oauth2#readme)

:::: tabs

::: tab yarn

```bash
yarn add passport-azure-ad-oauth2 jsonwebtoken
```

:::

::: tab npm

```bash
npm install --save passport-azure-ad-oauth2 jsonwebtoken
```

:::

::::

`/config/server.js`

```jsx
'use strict';

const AzureAdOAuth2Strategy = require('passport-azure-ad-oauth2');
const jwt = require('jsonwebtoken');

module.exports = ({ env }) => ({
  // ...
  admin: {
    // ...
    auth: {
      // ...
      providers: [
        {
          uid: 'azure_ad_oauth2',
          displayName: 'Microsoft',
          icon:
            'https://upload.wikimedia.org/wikipedia/commons/thumb/9/96/Microsoft_logo_%282012%29.svg/320px-Microsoft_logo_%282012%29.svg.png',
          createStrategy: strapi =>
            new AzureAdOAuth2Strategy(
              {
                clientID: env('MICROSOFT_CLIENT_ID', ''),
                clientSecret: env('MICROSOFT_CLIENT_SECRET', ''),
                scope: ['user:email'],
                tenant: env('MICROSOFT_TENANT_ID', ''),
                callbackURL: strapi.admin.services.passport.getStrategyCallbackURL(
                  'azure_ad_oauth2'
                ),
              },
              (accessToken, refreshToken, params, profile, done) => {
                var waadProfile = jwt.decode(params.id_token, '', true);
                done(null, {
                  email: waadProfile.upn,
                  username: waadProfile.upn,
                });
              }
            ),
        },
      ],
    },
  },
});
```

:::::
::::::

##### 高级定制

###### 管理面板网址

如果管理面板所在的主机/端口与 Strapi 服务器不同，则需要修改管理 URL。为此，请转到 `/config/server.js` 配置文件并调整 `admin.url` 字段。

例如，如果你的管理应用程序是在 `https://api.example.com` 上启动的，那么你的配置如下所示:

`/config/server.js`

```javascript
module.exports = () => ({
  // ...
  admin: {
    // ...
    url: 'https://api.example.com/admin',
  },
});
```

###### 自定义逻辑

在某些情况下，您需要为您的连接工作流编写额外的逻辑，例如:

- 限制特定域的连接和注册
- 触发连接尝试的动作
- 分析

最简单的方法是插入您的策略的 verify 函数并编写一些代码。

例如，如果你想只允许拥有官方 strapi.io 电子邮件地址的人，你可以这样实例化你的策略:

```javascript
const strategyInstance = new Strategy(configuration, ({ email, username }, done) => {
  // If the email ends with @strapi.io
  if (email.endsWith('@strapi.io')) {
    // Then we continue with the data given by the provider
    return done(null, { email, username });
  }

  // Otherwise, we continue by sending an error to the done function
  done(new Error('Forbidden email address'));
});
```

###### 认证事件

SSO 特性添加了一个新的 [身份验证事件](/developer-docs/latest/setup-deployment-guides/configurations.md#available-options): `onSSOAutoRegistration`.

每当使用 SSO 添加的自动注册特性创建用户时，都会触发此事件。它包含创建的用户(`event.user`)和用于进行注册的提供程序(event.provider)。

例子:

`/config/server.js`

```javascript
module.exports = () => ({
  // ...
  admin: {
    // ...
    auth: {
      // ...
      events: {
        onConnectionSuccess(e) {},
        onConnectionError(e) {},
        // ...
        onSSOAutoRegistration(e) {
          const { user, provider } = e;

          console.log(
            `A new user (${user.id}) has been automatically registered using ${provider}`
          );
        },
      },
    },
  },
});
```
