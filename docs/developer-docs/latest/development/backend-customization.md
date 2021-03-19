# 后端定制

<!--- BEGINNING OF ROUTING --->

## Routing 路由

`./api/**/config/routes.json` 文件定义客户机的所有可用端点.

默认情况下，Strapi 为所有的 `Content Types` 生成端点，更多内容参考 [Content API](/developer-docs/latest/developer-resources/content-api/content-api.md#api-endpoints) 。

### 如何创建一条路由?

您必须在您的一个 api 文件夹中编辑 `routes.json` 文件(`./api/**/config/routes.json`)并手动向 routes 数组中添加一个新的 `route` array 对象。

**Path —** `./api/**/config/routes.json`.

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/restaurants",
      "handler": "Restaurant.find",
      "config": {
        "policies": []
      }
    },
    {
      "method": "PUT",
      "path": "/restaurants/bulkUpdate",
      "handler": "Restaurant.bulkUpdate",
      "config": {
        "policies": []
      }
    },
    {
      "method": "POST",
      "path": "/restaurants/:id/reservation",
      "handler": "Restaurant.reservation",
      "config": {
        "policies": ["isAuthenticated", "hasCreditCard"]
      }
    }
  ]
}
```

- `method` (string): 命中路由的方法的方法或数组 (e.g. `GET`, `POST`, `PUT`, `HEAD`, `DELETE`, `PATCH`).
- `path` (string): 启动路径 URL `/` (e.g. `/restaurants`).
- `handler` (string): 当按照此语法命中路由时执行的操作 `<Controller>.<action>`.
- `config`
  - `policies` (array): 策略名称或路径的数组 ([更多查看](#policies))

::: tip
如果你不想让 [用户&权限插件](/developer-docs/latest/development/plugins/users-permissions.md) 检查路由，你可以排除整个 `config` 对象。
:::

### 动态参数

Strapi 使用的路由器允许您创建可以使用参数和简单正则表达式的动态路由。这些参数将在 `ctx.params` 对象中公开。有关更多细节，请参考 [PathToRegex](https://github.com/pillarjs/path-to-regexp) 文档。

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/restaurants/:category/:id",
      "handler": "Restaurant.findOneByCategory",
      "config": {
        "policies": []
      }
    },
    {
      "method": "GET",
      "path": "/restaurants/:region(\\d{2}|\\d{3})/:id", // Only match when the first parameter contains 2 or 3 digits.
      "handler": "Restaurant.findOneByRegion",
      "config": {
        "policies": []
      }
    }
  ]
}
```

#### 例子

带有 URL 参数的路由定义

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/restaurants/:id",
      "handler": "Restaurant.findOne",
      "config": {
        "policies": []
      }
    }
  ]
}
```

获取控制器中的 URL 参数

```js
module.exports = {
  findOne: async ctx => {
    // const id = ctx.params.id;
    const { id } = ctx.params;
    return id;
  },
};
```

<!--- BEGINNING OF POLICIES --->

## Policies 策略

策略是能够在每个请求到达控制器的操作之前对其执行特定逻辑的函数。它们主要用于简单地保护业务逻辑。项目的每个路由都可以关联到一组策略。例如，您可以创建名为 `isAdmin` 的策略，该策略显然检查请求是否由管理员用户发送，并将其用于关键路由。

策略定义在了 `./api/**/config/policies/` 目录中. 它们分别通过 `strapi.api.**.config.policies` 和 `strapi.plugins.**.config.policies` 配置策略。全局策略定义在 `./config/policies/` ，可通过 `strapi.config.policies` 访问。

### 如何制定策略 policy?

有几种方法可以创建一个策略 policy.

- 使用 CLI `strapi generate:policy isAuthenticated`.<br>阅读 [CLI documentation](/developer-docs/latest/developer-resources/cli/CLI.md) 了解详情.
- 手动创建一个 JavaScript 文件，名字 `isAuthenticated.js` 放在 `./config/policies/` 目录下.

**Path —** `./config/policies/isAuthenticated.js`.

```js
module.exports = async (ctx, next) => {
  if (ctx.state.user) {
    // Go to next policy or will reach the controller's action.
    return await next();
  }

  ctx.unauthorized(`You're not logged in!`);
};
```

在这个示例中，我们正在验证一个会话是否打开。如果是这种情况，我们调用 `next()` 方法来执行下一个策略或控制器的操作。否则，返回一个 `401` 错误。

> 401 HTTP 状态码表示没有登录认证

### 用法

要将策略应用于路由，需要将一组策略与其关联。有两种策略: global 策略和 scoped 策略。

::: warning

要使用 GraphQL 应用策略，请参阅 [以下指南](/developer-docs/latest/development/plugins/graphql.md#execute-a-policy-before-a-resolver)。
:::

#### Global 策略

全局策略可以关联到项目中的任何路由。

**Path —** `./api/restaurant/routes.json`.

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/restaurants",
      "handler": "Restaurant.find",
      "config": {
        "policies": ["global::isAuthenticated"]
      }
    }
  ]
}
```

在 `Restaurant.js` 控制器中执行 `find` 操作之前，将调用位于 `./config/policies/isAuthenticated.js` 中的全局策略 `isAuthenticated` 。

::: tip
您可以在这个数组中放置任意多的策略，但是要注意对性能的影响。
:::

#### 插件策略

插件可以在你的应用程序中添加和公开策略。例如，插件 Users & Permissions 附带了有用的策略，以确保用户得到良好的身份验证或拥有执行操作的权限。

**Path —** `./api/restaurant/config/routes.json`.

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/restaurants",
      "handler": "Restaurant.find",
      "config": {
        "policies": ["plugins::users-permissions.isAuthenticated"]
      }
    }
  ]
}
```

位于 `users-permissions` 插件中的策略 `isAuthenticated` 将在 `Restaurant.js` 控制器中的 `find` 操作之前执行。

#### 插件策略

API 策略可以与在 API 中定义的路由相关联，这些路由是在 API 中声明的。

**Path —** `./api/restaurant/config/policies/isAdmin.js`.

```js
module.exports = async (ctx, next) => {
  if (ctx.state.user.role.name === 'Administrator') {
    // Go to next policy or will reach the controller's action.
    return await next();
  }

  ctx.unauthorized(`You're not allowed to perform this action!`);
};
```

**Path —** `./api/restaurant/config/routes.json`.

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/restaurants",
      "handler": "Restaurant.find",
      "config": {
        "policies": ["isAdmin"]
      }
    }
  ]
}
```

位于 `./api/restaurant/config/policies/isAdmin.js` 中的策略 `isAdmin` 将在 `Restaurant.js` 控制器中的 `find` 操作之前执行。

#### 使用 api 之外的策略

要在另一个 api 中使用策略，可以使用以下语法引用它: `{apiName}.{policyName}` 。

**Path —** `./api/category/config/routes.json`.

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/categories",
      "handler": "Category.find",
      "config": {
        "policies": ["restaurant.isAdmin"]
      }
    }
  ]
}
```

### 高级使用

如上所述，策略在控制器的操作之前执行。它看起来像是在控制器动作 `before` 可以执行的动作。您还可以在 `after`。

**Path —** `./config/policies/custom404.js`.

```js
module.exports = async (ctx, next) => {
  // Indicate to the server to go to
  // the next policy or to the controller's action.
  await next();

  // The code below will be executed after the controller's action.
  if (ctx.status === 404) {
    ctx.body = 'We cannot find the resource.';
  }
};
```

<!--- BEGINNING OF CONTROLLERS --->

## Controllers 控制器

控制器是 JavaScript 文件，其中包含一组方法，称为客户机根据请求的路由到达的动作。这意味着每次客户机请求路由时，操作都执行业务逻辑编码并发送回响应。它们在 MVC 模式中表示 c。在大多数情况下，控制器将包含项目的大部分业务逻辑。

```js
module.exports = {
  // GET /hello
  async index(ctx) {
    return 'Hello World!';
  },
};
```

在这个例子中，任何时候浏览器指向应用程序上的 `/hello` URL，页面都会显示文本: `Hello World!`.

控制器定义在每个 `./api/**/controllers/` .放在这些文件夹中的每个 JavaScript 文件都将作为控制器加载。它们也可以通过 `strapi.controllers` 和 `strapi.api.**.controllers` 全局变量。

### 核心控制器

当您创建一个新的 `Content Type` 时，您将看到一个新的空控制器已被创建。这是因为 Strapi 默认为您的模型构建了一个通用控制器，并允许您在生成的文件中覆盖和扩展它。

#### 扩展模型控制器

以下是核心方法(及其当前实现)。您只需将此代码复制并粘贴到自己的控制器文件中，即可自定义方法。

::: warning
在下面的示例中，我们将假设您的控制器、服务和模型命名为 `restaurant`
:::

##### Utils 工具类

首先引入 'strapi-utils' 包

```js
const { parseMultipartData, sanitizeEntity } = require('strapi-utils');
```

- `parseMultipartData`: 此函数解析 Strapi 的 formData 格式.
- `sanitizeEntity`: 此函数从模型及其关系中删除所有私有字段.

##### Collection Type 集合类型

:::: tabs

::: tab find

##### `find`

```js
const { sanitizeEntity } = require('strapi-utils');

module.exports = {
  /**
   * Retrieve records.
   *
   * @return {Array}
   */

  async find(ctx) {
    let entities;
    if (ctx.query._q) {
      entities = await strapi.services.restaurant.search(ctx.query);
    } else {
      entities = await strapi.services.restaurant.find(ctx.query);
    }

    return entities.map(entity => sanitizeEntity(entity, { model: strapi.models.restaurant }));
  },
};
```

:::

::: tab findOne

##### `findOne`

```js
const { sanitizeEntity } = require('strapi-utils');

module.exports = {
  /**
   * Retrieve a record.
   *
   * @return {Object}
   */

  async findOne(ctx) {
    const { id } = ctx.params;

    const entity = await strapi.services.restaurant.findOne({ id });
    return sanitizeEntity(entity, { model: strapi.models.restaurant });
  },
};
```

:::

::: tab count

##### `count`

```js
module.exports = {
  /**
   * Count records.
   *
   * @return {Number}
   */

  count(ctx) {
    if (ctx.query._q) {
      return strapi.services.restaurant.countSearch(ctx.query);
    }
    return strapi.services.restaurant.count(ctx.query);
  },
};
```

:::

::: tab create

##### `create`

```js
const { parseMultipartData, sanitizeEntity } = require('strapi-utils');

module.exports = {
  /**
   * Create a record.
   *
   * @return {Object}
   */

  async create(ctx) {
    let entity;
    if (ctx.is('multipart')) {
      const { data, files } = parseMultipartData(ctx);
      entity = await strapi.services.restaurant.create(data, { files });
    } else {
      entity = await strapi.services.restaurant.create(ctx.request.body);
    }
    return sanitizeEntity(entity, { model: strapi.models.restaurant });
  },
};
```

:::

::: tab update

##### `update`

```js
const { parseMultipartData, sanitizeEntity } = require('strapi-utils');

module.exports = {
  /**
   * Update a record.
   *
   * @return {Object}
   */

  async update(ctx) {
    const { id } = ctx.params;

    let entity;
    if (ctx.is('multipart')) {
      const { data, files } = parseMultipartData(ctx);
      entity = await strapi.services.restaurant.update({ id }, data, {
        files,
      });
    } else {
      entity = await strapi.services.restaurant.update({ id }, ctx.request.body);
    }

    return sanitizeEntity(entity, { model: strapi.models.restaurant });
  },
};
```

:::

::: tab delete

##### `delete`

```js
const { sanitizeEntity } = require('strapi-utils');

module.exports = {
  /**
   * Delete a record.
   *
   * @return {Object}
   */

  async delete(ctx) {
    const { id } = ctx.params;

    const entity = await strapi.services.restaurant.delete({ id });
    return sanitizeEntity(entity, { model: strapi.models.restaurant });
  },
};
```

:::

::::

##### Single Type 单一类型

:::: tabs

::: tab find

##### `find`

```js
const { sanitizeEntity } = require('strapi-utils');

module.exports = {
  /**
   * Retrieve the record.
   *
   * @return {Object}
   */

  async find(ctx) {
    const entity = await strapi.services.restaurant.find();
    return sanitizeEntity(entity, { model: strapi.models.restaurant });
  },
};
```

:::

::: tab update

##### `update`

```js
const { parseMultipartData, sanitizeEntity } = require('strapi-utils');

module.exports = {
  /**
   * Update the record.
   *
   * @return {Object}
   */

  async update(ctx) {
    let entity;
    if (ctx.is('multipart')) {
      const { data, files } = parseMultipartData(ctx);
      entity = await strapi.services.restaurant.createOrUpdate(data, {
        files,
      });
    } else {
      entity = await strapi.services.restaurant.createOrUpdate(ctx.request.body);
    }

    return sanitizeEntity(entity, { model: strapi.models.restaurant });
  },
};
```

:::

::: tab delete

##### `delete`

```js
const { sanitizeEntity } = require('strapi-utils');

module.exports = {
  /**
   * Delete the record.
   *
   * @return {Object}
   */

  async delete(ctx) {
    const entity = await strapi.services.restaurant.delete();
    return sanitizeEntity(entity, { model: strapi.models.restaurant });
  },
};
```

:::

::::

### 自定义控制器

您还可以创建自定义控制器来构建自己的业务逻辑和 API Endpoint 端点。

创建控制器有两种方法:

- 使用 CLI `strapi generate:controller restaurant`.<br>阅读 [CLI documentation](/developer-docs/latest/developer-resources/cli/CLI.md#strapi-generate-controller) 了解详情.
- 手动创建 JavaScript 文件 `./api/**/controllers`.

#### 添加 Endpoints

每个控制器的操作必须是一个 `async` 异步函数。

每个操作都接收一个 `context` (`ctx`) 对象作为第一个参数，其中包含 [request context](/developer-docs/latest/development/backend-customization.md#responses) 和 [response context](/developer-docs/latest/development/backend-customization.md#responses) 。

#### 例子

在这个例子中, 我们定义了一个路由 `route` 文件 `./api/hello/config/routes.json`, 处理程序 `Hello.index`. 有关路由的详细信息，请参阅 [Routing documentation](#routing)。

这意味着每次向服务器发送请求 `GET /hello` 时，Strapi 将调用 `Hello.js` 控制器中的 `index` 操作。我们的索引动作将返回 `Hello World！`. 您还可以返回一个 JSON 对象。

**Path —** `./api/hello/config/routes.json`.

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/hello",
      "handler": "Hello.index",
      "config": {
        "policies": []
      }
    }
  ]
}
```

**Path —** `./api/hello/controllers/Hello.js`.

```js
module.exports = {
  // GET /hello
  async index(ctx) {
    ctx.send('Hello World!');
  },
};
```

::: tip
路由处理程序只能访问 `./api/**/controllers` 文件夹中定义的控制器。
:::

<!--- REQUESTS AND RESPONSES --->

## Requests & Responses 请求 & 回应

### Requests 请求

上下文对象(`ctx`)包含所有与请求相关的信息，可以通过 `ctx.request`、[控制器](/developer-docs/latest/development/backend-customization.md#controllers) 和 [策略](/developer-docs/latest/development/backend-customization.md#policies) 访问它们。

Strapi 通过 `ctx.request.body` 和 `ctx.request.files` 传递 `body`

有关更多信息，请参考 [Koa 请求文档](http://koajs.com/#request)。

### Responses 回应

上下文对象(`ctx`)包含一系列用于管理服务器响应的值和函数。可以通过 `ctx.response`、[controllers](/developer-docs/latest/development/backend-customization.md#controllers) 和 [policies](/developer-docs/latest/development/backend-customization.md#policies) 访问它们。

详细信息, 访问 [Koa response documentation](http://koajs.com/#response).

<!--- BEGINNING OF SERVICES --->

## Services 服务

服务是一组可重用的函数。它们在尊重 DRY (不要重复自己) 编程概念和简化 [controllers](#controllers) 逻辑方面特别有用。

### 核心服务

当您创建一个新的 `Content Type` 或者一个新的模型时，您将看到一个新的空服务已经被创建。这是因为 Strapi 默认为您的模型构建了一个通用服务，并允许您在生成的文件中覆盖和扩展它。

#### 扩展模型服务

以下是核心方法(及其当前实现)。您只需将此代码复制并粘贴到您自己的服务文件中，即可自定义方法。

你可以在 [这里](#queries) 阅读 `strapi.query` 调用。

::: tip
在下面的示例中，您的控制器、服务和模型被命名为 `restaurant`。
:::

#### Utils 工具函数

如果你正在扩展 `create` 或者 `update` 服务，首先需要下面的工具函数:

```js
const { isDraft } = require('strapi-utils').contentTypes;
```

- `isDraft`: 此函数检查条目是否为草稿.

##### Collection Type 集合类型

:::: tabs

::: tab find

##### `find`

```js
module.exports = {
  /**
   * Promise to fetch all records
   *
   * @return {Promise}
   */
  find(params, populate) {
    return strapi.query('restaurant').find(params, populate);
  },
};
```

- `params` (object): 这表示用于查找请求的过滤器.<br>
  对象遵循 URL 查询格式, [请参阅这份文件.](/developer-docs/latest/developer-resources/content-api/content-api.md#api-parameters).

```json
{
  "name": "Tokyo Sushi"
}
// or
{
  "_limit": 20,
  "name_contains": "sushi"
}
```

- `populate` (array): 你必须提到你想要填充的数据 `["author", "author.name", "comment", "comment.content"]`

:::

::: tab findOne

##### `findOne`

```js
module.exports = {
  /**
   * Promise to fetch record
   *
   * @return {Promise}
   */

  findOne(params, populate) {
    return strapi.query('restaurant').findOne(params, populate);
  },
};
```

- `params` (object): 这表示用于查找请求的过滤器.<br>
  对象遵循 URL 查询格式, [请参阅这份文件.](/developer-docs/latest/developer-resources/content-api/content-api.md#api-parameters).

```json
{
  "name": "Tokyo Sushi"
}
// or
{
  "name_contains": "sushi"
}
```

- `populate` (array): 你必须提到你想要填充的数据 `["author", "author.name", "comment", "comment.content"]`

:::

::: tab count

##### `count`

```js
module.exports = {
  /**
   * Promise to count record
   *
   * @return {Promise}
   */

  count(params) {
    return strapi.query('restaurant').count(params);
  },
};
```

- `params` (object): 这表示用于查找请求的过滤器.<br>
  对象遵循 URL 查询格式, [请参阅这份文件.](/developer-docs/latest/developer-resources/content-api/content-api.md#api-parameters).

```json
{
  "name": "Tokyo Sushi"
}
// or
{
  "name_contains": "sushi"
}
```

:::

::: tab create

##### `create`

```js
const { isDraft } = require('strapi-utils').contentTypes;

module.exports = {
  /**
   * Promise to add record
   *
   * @return {Promise}
   */

  async create(data, { files } = {}) {
    const isDraft = isDraft(data, strapi.models.restaurant);
    const validData = await strapi.entityValidator.validateEntityCreation(
      strapi.models.restaurant,
      data,
      { isDraft }
    );

    const entry = await strapi.query('restaurant').create(validData);

    if (files) {
      // automatically uploads the files based on the entry and the model
      await strapi.entityService.uploadFiles(entry, files, {
        model: 'restaurant',
        // if you are using a plugin's model you will have to add the `source` key (source: 'users-permissions')
      });
      return this.findOne({ id: entry.id });
    }

    return entry;
  },
};
```

:::

::: tab update

##### `update`

```js
const { isDraft } = require('strapi-utils').contentTypes;

module.exports = {
  /**
   * Promise to edit record
   *
   * @return {Promise}
   */

  async update(params, data, { files } = {}) {
    const existingEntry = await db.query('restaurant').findOne(params);

    const isDraft = isDraft(existingEntry, strapi.models.restaurant);
    const validData = await strapi.entityValidator.validateEntityUpdate(
      strapi.models.restaurant,
      data,
      { isDraft }
    );

    const entry = await strapi.query('restaurant').update(params, validData);

    if (files) {
      // automatically uploads the files based on the entry and the model
      await strapi.entityService.uploadFiles(entry, files, {
        model: 'restaurant',
        // if you are using a plugin's model you will have to add the `source` key (source: 'users-permissions')
      });
      return this.findOne({ id: entry.id });
    }

    return entry;
  },
};
```

- `params` (object): it should look like this `{id: 1}`

:::

::: tab delete

##### `delete`

```js
module.exports = {
  /**
   * Promise to delete a record
   *
   * @return {Promise}
   */

  delete(params) {
    return strapi.query('restaurant').delete(params);
  },
};
```

- `params` (object): it should look like this `{id: 1}`

:::

::: tab search

##### `search`

```js
module.exports = {
  /**
   * Promise to search records
   *
   * @return {Promise}
   */

  search(params) {
    return strapi.query('restaurant').search(params);
  },
};
```

- `params` (object): 这表示用于查找请求的过滤器.<br>
  对象遵循 URL 查询格式, [请参阅这份文件.](/developer-docs/latest/developer-resources/content-api/content-api.md#api-parameters).

```json
{
  "name": "Tokyo Sushi"
}
// or
{
  "name_contains": "sushi"
}
```

:::

::: tab countSearch

##### `countSearch`

```js
module.exports = {
  /**
   * Promise to count searched records
   *
   * @return {Promise}
   */
  countSearch(params) {
    return strapi.query('restaurant').countSearch(params);
  },
};
```

- `params` (object): 这表示用于查找请求的过滤器.<br>
  对象遵循 URL 查询格式, [请参阅这份文件.](/developer-docs/latest/developer-resources/content-api/content-api.md#api-parameters).

```json
{
  "name": "Tokyo Sushi"
}
// or
{
  "name_contains": "sushi"
}
```

:::

::::

##### Single Type 单一类型

:::: tabs

::: tab find

##### `find`

```js
const _ = require('lodash');

module.exports = {
  /**
   * Promise to fetch the record
   *
   * @return {Promise}
   */
  async find(populate) {
    const results = await strapi.query('restaurant').find({ _limit: 1 }, populate);
    return _.first(results) || null;
  },
};
```

- `populate` (array): 你必须提到你想要填充的数据 `["author", "author.name", "comment", "comment.content"]`

:::

::: tab createOrUpdate

##### `createOrUpdate`

```js
const _ = require('lodash');

module.exports = {
  /**
   * Promise to add/update the record
   *
   * @return {Promise}
   */

  async createOrUpdate(data, { files } = {}) {
    const results = await strapi.query('restaurant').find({ _limit: 1 });
    const entity = _.first(results) || null;

    let entry;
    if (!entity) {
      entry = await strapi.query('restaurant').create(data);
    } else {
      entry = await strapi.query('restaurant').update({ id: entity.id }, data);
    }

    if (files) {
      // automatically uploads the files based on the entry and the model
      await strapi.entityService.uploadFiles(entry, files, {
        model: 'restaurant',
        // if you are using a plugin's model you will have to add the `plugin` key (plugin: 'users-permissions')
      });
      return this.findOne({ id: entry.id });
    }

    return entry;
  },
};
```

:::

::: tab delete

##### `delete`

```js
module.exports = {
  /**
   * Promise to delete a record
   *
   * @return {Promise}
   */

  delete() {
    const results = await strapi.query('restaurant').find({ _limit: 1 });
    const entity = _.first(results) || null;

    if (!entity) return;

    return strapi.query('restaurant').delete({id: entity.id});
  },
};
```

:::

::::

### 自定义服务

您还可以创建自定义服务来构建自己的业务逻辑。

创建服务有两种方法。

- 使用 CLI `strapi generate:service restaurant`.<br>阅读 [CLI documentation](/developer-docs/latest/developer-resources/cli/CLI.md) 了解详情.
- 手动创建 JavaScript 文件 `./api/**/services/`.

#### 例子

服务的目标是存储可重用的函数。`email` 服务可以从我们的代码库中的不同功能发送电子邮件:

**Path —** `./api/email/services/Email.js`.

```js
const nodemailer = require('nodemailer');

// Create reusable transporter object using SMTP transport.
const transporter = nodemailer.createTransport({
  service: 'Gmail',
  auth: {
    user: 'user@gmail.com',
    pass: 'password',
  },
});

module.exports = {
  send: (from, to, subject, text) => {
    // Setup e-mail data.
    const options = {
      from,
      to,
      subject,
      text,
    };

    // Return a promise of the function that sends the email.
    return transporter.sendMail(options);
  },
};
```

::: tip
请确保为此示例安装了 `nodemailer` (`npm install nodemailer`)。
:::

现在可以通过 `strapi.services` 全局变量使用该服务。我们可以在代码库的另一部分使用它。例如下面这个控制器:

**Path —** `./api/user/controllers/User.js`.

```js
module.exports = {
  // GET /hello
  signup: async ctx => {
    // Store the new user in database.
    const user = await User.create(ctx.query);

    // Send an email to validate his subscriptions.
    strapi.services.email.send('welcome@mysite.com', user.email, 'Welcome', '...');

    // Send response to the server.
    ctx.send({
      ok: true,
    });
  },
};
```

<!--- BEGINNING OF QUERIES --->

## Queries 查询

Strapi 提供了一个实用函数 `strapi.query` 来进行数据库查询。

您只需调用 `strapi.query('modelName', 'pluginName')` 就可以访问任何模型的查询 API。

这些查询为您处理特定的 Strapi 功能，如 `components`, `dynamic zones`, `filters` 和 `search`。

### API 参考

:::: tabs

::: tab find

#### `find`

此方法返回匹配 Strapi 筛选器的条目列表。您还可以传递一个 populate 选项来指定要填充的关系。

##### 例子

**Find by id**:

```js
strapi.query('restaurant').find({ id: 1 });
```

**Find by in IN, with a limit**:

```js
strapi.query('restaurant').find({ _limit: 10, id_in: [1, 2] });
```

**Find by date orderBy name**:

```js
strapi.query('restaurant').find({ date_gt: '2019-01-01T00:00:00Z', _sort: 'name:desc' });
```

**Find by id not in and populate a relation. Skip the first ten results**

```js
strapi.query('restaurant').find({ id_nin: [1], _start: 10 }, ['category', 'category.name']);
```

:::

::: tab findOne

#### `findOne`

此方法返回与某些基本参数匹配的第一个条目。您还可以传递一个 populate 选项来指定要填充的关系。

##### 例子

**Find one by id**:

```js
strapi.query('restaurant').findOne({ id: 1 });
```

**Find one by name**:

```js
strapi.query('restaurant').findOne({ name: 'restaurant name' });
```

**Find one by name and creation_date**:

```js
strapi.query('restaurant').findOne({ name: 'restaurant name', date: '2019-01-01T00:00:00Z' });
```

**Find one by id and populate a relation**

```js
strapi.query('restaurant').findOne({ id: 1 }, ['category', 'category.name']);
```

:::

::: tab create

#### `create`

在数据库中创建一个条目并返回该条目。

##### 例子

```js
strapi.query('restaurant').create({
  name: 'restaurant name',
  // this is a dynamiczone field. the order is persisted in db.
  content: [
    {
      __component: 'blog.rich-text',
      title: 'Some title',
      subTitle: 'Some sub title',
    },
    {
      __component: 'blog.quote',
      quote: 'Some interesting quote',
      author: 1,
    },
  ],
  // this is a component field. the order is persisted in db.
  opening_hours: [
    {
      day_interval: 'Mon',
      opening_hour: '7:00 PM',
      closing_hour: '11:00 PM',
    },
    {
      day_interval: 'Tue',
      opening_hour: '7:00 PM',
      closing_hour: '11:00 PM',
    },
  ],
  // pass the id of a media to link it to the entry
  cover: 1,
  // automatically creates the relations when passing the ids in the field
  reviews: [1, 2, 3],
});
```

:::

::: tab update

#### `update`

更新数据库中的条目并返回该条目。

##### 例子

**Update by id**

```js
strapi.query('restaurant').update(
  { id: 1 },
  {
    name: 'restaurant name',
    content: [
      {
        __component: 'blog.rich-text',
        title: 'Some title',
        subTitle: 'Some sub title',
      },
      {
        __component: 'blog.quote',
        quote: 'Some interesting quote',
        author: 1,
      },
    ],
    opening_hours: [
      {
        day_interval: 'Mon',
        opening_hour: '7:00 PM',
        closing_hour: '11:00 PM',
      },
      {
        day_interval: 'Tue',
        opening_hour: '7:00 PM',
        closing_hour: '11:00 PM',
      },
    ],
    // pass the id of a media to link it to the entry
    cover: 1,
    // automatically creates the relations when passing the ids in the field
    reviews: [1, 2, 3],
  }
);
```

在使用组件或动态区域更新条目时，请注意，如果发送的组件没有任何 `id`，则前面的组件将被删除和替换。你可以通过发送它们的 `id` 和其他字段来更新组件:

**通过 id 进行更新并更新以前的组件**

```js
strapi.query('restaurant').update(
  { id: 1 },
  {
    name: 'Mytitle',
    content: [
      {
        __component: 'blog.rich-text',
        id: 1,
        title: 'Some title',
        subTitle: 'Some sub title',
      },
      {
        __component: 'blog.quote',
        id: 1,
        quote: 'Some interesting quote',
        author: 1,
      },
    ],
    opening_hours: [
      {
        id: 2,
        day_interval: 'Mon',
        opening_hour: '7:00 PM',
        closing_hour: '11:00 PM',
      },
      {
        id: 1,
        day_interval: 'Tue',
        opening_hour: '7:00 PM',
        closing_hour: '11:00 PM',
      },
    ],
    // pass the id of a media to link it to the entry
    cover: 1,
    // automatically creates the relations when passing the ids in the field
    reviews: [1, 2, 3],
  }
);
```

**按名称部分更新**

```js
strapi.query('restaurant').update(
  { title: 'specific name' },
  {
    title: 'restaurant name',
  }
);
```

:::

::: tab delete

#### `delete`

在删除之前删除一个条目并返回其值。您可以使用传递的参数一次删除多个条目。

##### 例子

**按 id 删除一个**

```js
strapi.query('restaurant').delete({ id: 1 });
```

**按字段删除多个**

```js
strapi.query('restaurant').delete({ district: '_18th' });
```

:::

::: tab count

#### `count`

返回匹配 Strapi 筛选器的条目数。

##### 例子

**按地区计算**

```js
strapi.query('restaurant').count({ district: '_1st' });
```

**按名称计数包含**

```js
strapi.query('restaurant').count({ name_contains: 'food' });
```

**按日期计数小于**

```js
strapi.query('restaurant').count({ date_lt: '2019-08-01T00:00:00Z' });
```

:::

::: tab search

#### `search`

基于对所有字段的搜索返回条目(此特性将返回 sqlite 中的所有条目)。

##### 例子

**首先从 20 开始搜索 10**

```js
strapi.query('restaurant').search({ _q: 'my search query', _limit: 10, _start: 20 });
```

**搜索和排序**

```js
strapi.query('restaurant').search({ _q: 'my search query', _limit: 100, _sort: 'date:desc' });
```

:::

::: tab countSearch

#### `countSearch`

返回基于搜索的条目总数(此特性将返回 sqlite 上的所有条目)。

##### 例子

```js
strapi.query('restaurant').countSearch({ _q: 'my search query' });
```

:::

::::

### 自定义查询

当您希望自定义服务或创建新服务时，必须使用底层 ORM 模型构建查询。

要访问底层模型:

```js
strapi.query(modelName, plugin).model;
```

然后您可以在模型上运行任何可用的查询。有关更多细节，您应该参考特定 ORM 文档:

:::: tabs

::: tab SQL

#### Bookshelf

文档: [https://bookshelfjs.org/](https://bookshelfjs.org/)

**例子**

```js
const result = await strapi
  .query('restaurant')
  .model.query(qb => {
    qb.where('id', 1);
  })
  .fetch();

const fields = result.toJSON();
```

#### Knex

可以用 `Knex.js` 直接对数据库进行构建和自定义查询。

文档: [http://knexjs.org/#Builder](http://knexjs.org/#Builder)

您可以使用以下命令访问 Knex 实例:

```js
const knex = strapi.connections.default;
```

然后可以使用 Knex 构建自己的定制查询。您将失去模型的所有功能，但是如果您正在构建一个更加自定义的模式，这可能会很方便。请注意，如果您使用的是 [draft system](/developer-docs/latest/concepts/draft-and-publish.md) ，那么在发布 Draft 列之前，Strapi 将使其为空。

**例子**

```js
const _ = require('lodash');

const knex = strapi.connections.default;
const result = await knex('restaurants')
  .where('cities', 'berlin')
  .whereNot('cities.published_at', null)
  .join('chefs', 'restaurants.id', 'chefs.restaurant_id')
  .select('restaurants.name as restaurant')
  .select('chef.name as chef')

// Lodash's groupBy method can be used to
// return a grouped key-value object generated from
// the response

return (_.groupBy(result, 'chef');
```

**我们强烈建议在对 DB 进行查询之前对任何字符串进行净化。**

千万不要试图对直接来自前端的数据进行原始查询; 如果要查找 `raw` 原始查询，请参阅文档的 [这一部分](http://knexjs.org/#Raw-Bindings) 。

:::

::: tab MongoDB

#### Mongoose

文档: [https://mongoosejs.com/](https://mongoosejs.com/)

**例子**

```js
const result = strapi.query('restaurant').model.find({
  date: { $gte: '2019-01-01T00.00.00Z' },
});

const fields = result.map(entry => entry.toObject());
```

:::

::::

<!--- BEGINNING OF MODELS --->

## Models 模型

### 概念

#### 内容类型的模型

模型是数据库结构的表示。它们被分成两个独立的文件。一个包含模型选项(例如: lifecycle hooks)的 JavaScript 文件，以及一个表示存储在数据库中的数据结构的 JSON 文件。

**Path —** `./api/restaurant/models/Restaurant.js`.

```js
module.exports = {
  lifecycles: {
    // Called before an entry is created
    beforeCreate(data) {},
    // Called after an entry is created
    afterCreate(result) {},
  },
};
```

**Path —** `./api/restaurant/models/Restaurant.settings.json`.

```json
{
  "kind": "collectionType",
  "connection": "default",
  "info": {
    "name": "restaurant",
    "description": "This represents the Restaurant Model"
  },
  "attributes": {
    "cover": {
      "collection": "file",
      "via": "related",
      "plugin": "upload"
    },
    "name": {
      "default": "",
      "type": "string"
    },
    "description": {
      "default": "",
      "type": "text"
    }
  }
}
```

在这个示例中，有一个 `Restaurant` 模型，其中包含了 `cover`、 `name` 和 `description` 属性。

#### 组件的模型

另一种类型的模型是命名 `components` 。组件是一种数据结构，可以在一个或多个其他 API 模型中使用。没有与生命周期相关的，只有一个 JSON 文件定义。

**Path —** `./components/default/simple.json`

```json
{
  "connection": "default",
  "collectionName": "components_default_simples",
  "info": {
    "name": "simple",
    "icon": "arrow-circle-right"
  },
  "options": {},
  "attributes": {
    "name": {
      "type": "string"
    }
  }
}
```

在这个示例中，有一个包含属性 `name` 的 `Simple` 组件。并且组件是 `default` 类别。

#### 模型在哪里定义？

**Content Types** 内容类型模型在每个中定义 `./api/**/models/` 这些文件夹中的每个 JavaScript 或 JSON 文件都将作为模型加载。它们也可以通过 `strapi.models` 和 `strapi.api.**.models` 建立全局变量模型。在项目的任何地方都可以使用，它们包含它们所引用的 ORM 模型对象。按照惯例，模型的名称应该用小写字母书写。

**Components** 模型定义在 `./components` 文件夹。每个组件都必须位于子文件夹(组件的类别名称)中。

### 如何创建一个模型？

::: tip
如果您刚刚开始，在管理界面中直接使用 `Content Type Builder` 生成一些模型是非常方便的。然后，您可以在代码级别查看生成的模型映射。用户界面接管了许多验证任务，让您对可用的特性有一种感觉。
:::

#### 对于内容类型模型

使用 CLI 并运行以下命令 `strapi generate:model restaurant name:string description:text`。

阅读 [CLI documentation](/developer-docs/latest/developer-resources/cli/CLI.md) 了解详情。

这将创建两个位于 `./api/restaurant/models` 的文件:

- `Restaurant.settings.json`: 包含属性和设置列表。 JSON 格式使文件易于编辑.
- `Restaurant.js`: 导入 `Restaurant.settings.json` 并通过附加设置和生命周期回调对其进行扩展.

::: tip
当您使用 CLI (`strapi generate:api <name>`)创建一个新的 API 时，会自动创建一个模型。
:::

#### 对于组件模型

要创建组件，必须使用管理面板中的 Content Type Builder，因为没有组件的 cli 生成器。

或者，您可以通过遵循前面描述的文件路径并遵循下面描述的文件结构来手动创建组件。

### 模型设置

可以在模型上设置其他设置:

- `kind` (string) - 定义模型是否为集合类型 (`collectionType`) 的单一类别 (`singleType`) - _只适用于内容类型_
- `connection` (string) - 必须使用的连接名称. 默认值: `default`.
- `collectionName` (string) - 存储数据的集合名称(或表名称).
- `globalId` (string) - 此模型的全局变量名 (case-sensitive) - _只适用于内容类型_
- `attributes` (object) - 定义模型的数据结构. 查找可用的选项 [below](#define-the-attributes).

**Path —** `Restaurant.settings.json`.

```json
{
  "kind": "collectionType",
  "connection": "mongo",
  "collectionName": "Restaurants_v1",
  "globalId": "Restaurants",
  "attributes": {}
}
```

在本例中，可以通过 `Restaurants` 全局变量访问模型 `Restaurant`。数据将存储在 `Restaurants_v1` `mongo` 集合或表中，模型将使用 `./config/database.js`

::: warning
如果没有在 JSON 文件中手动设置，Strapi 将采用文件名 `globalId`。`globalId` 作为关系和 Strapi api 中模型的参考。如果您选择重命名它(通过重命名文件或更改 `globalId` 的值) ，则必须手动迁移表并更新引用。请注意，您不应该更改 Strapi 的模型 `globalId` (插件和核心模型) ，因为它们直接用于 Strapi api 和其他模型的关系中。
:::

::: tip
可以随时更改 `connection` 值，但是应该注意，没有自动数据迁移过程。另外，如果新连接不使用相同的 ORM，那么您将不得不重写查询。
:::

### 模型资料

`model-json` 表示关于模型的信息。当显示模型时，此信息用于管理界面。

- `name`: 模型的名称，如管理界面中所示.
- `description`: 模型的描述.
- `icon`: `fontawesome V5` 的名字 - _只适用于组件_

**Path —** `Restaurant.settings.json`.

```json
{
  "info": {
    "name": "restaurant",
    "description": ""
  }
}
```

### 模型选项

模型上的选项键 model-json 状态。

- `timestamps`: 时间戳, 这告诉模型使用哪些属性作为时间戳。接受 `boolean` 或字符串 `Array` ，其中第一个元素为 `create date`，第二个元素为 `update date`。对于 Bookshelf 设置为 true 时的默认值是 `["created_at", "updated_at"]` ，对于 MongoDB 设置为 `["createdAt", "updatedAt"]` 。
- `privateAttributes`: 这种配置允许将一组属性视为私有属性，即使它们实际上没有在模型中定义为属性。接受字符串 `Array` 。当使用 MongoDB 时，它可以用来从 API 响应中删除时间戳或 `_v` 。模型中定义的 `privateAttributes` 集与全局 Strapi 配置中定义的 `privateAttributes` 合并。
- `populateCreatorFields`: 配置 API 响应是否应该包含 `created_by` 和 `updated_by` 字段。接受一个布尔值。默认值为 `false` 。
- `draftAndPublish`: 启用草稿和发布特性。接受一个 `boolean`。默认值为 `false`。

**Path —** `Restaurant.settings.json`.

```json
{
  "options": {
    "timestamps": true,
    "privateAttributes": ["id", "created_at"],
    "populateCreatorFields": true,
    "draftAndPublish": false
  }
}
```

### 定义属性

目前有以下几种类型:

- `string`
- `text`
- `richtext`
- `email`
- `password`
- `integer`
- `biginteger`
- `float`
- `decimal`
- `date`
- `time`
- `datetime`
- `boolean`
- `enumeration`
- `json`
- `uid`

#### Validations 验证

您可以对属性应用基本的验证。以下支持的验证只有 MongoDB 数据库连接支持。如果使用 SQL 数据库，应该使用本地 SQL 约束来应用它们。

- `required` (boolean) — 如果为真，则为此属性添加一个必需的验证器.
- `unique` (boolean) — 是否在此属性上定义唯一索引.
- `index` (boolean) — 在此属性上添加索引，这将创建一个 [single field index](https://docs.mongodb.com/manual/indexes/#single-field) 可以在后台运行. _只支持 MongoDB._
- `max` (integer) — 检查值是否大于或等于给定的最大值.
- `min` (integer) — 检查值是否小于或等于给定的最小值.

**安全验证**

为了改善开发人员在开发或使用管理面板时的体验，该框架通过以下这些“安全验证”来增强属性:

- `private` (boolean) — 如果为真，属性将从服务器响应中删除. (这对隐藏敏感数据很有用).
- `configurable` (boolean) - 如果属性为 false，则无法从 Content Type Builder 插件配置该属性.
- `autoPopulate` (boolean) - 如果为 false，相关数据将不会在 REST 响应中填充。(这不会停止查询 GraphQL 上的关系数据)

#### 例外

**uid**

- `targetField`(string) — The value is the name of an attribute that has `string` of the `text` type.
- `options` (string) — 该值是传递给 [the underlying `uid` generator](https://github.com/sindresorhus/slugify). 一个警告是 `uid` 必须遵守以下正则表达式 `/^[A-Za-z0-9-_.~]*$`.

#### 例子

**Path —** `Restaurant.settings.json`.

```json
{
  ...
  "attributes": {
    "title": {
      "type": "string",
      "min": 3,
      "max": 99,
      "unique": true
    },
    "description": {
      "default": "My description",
      "type": "text",
      "required": true
    },
    "slug": {
      "type": "uid",
      "targetField": "title"
    }
    ...
  }
}
```

### Relations 关系

关系允许您在内容类型之间创建链接(关系)。

:::: tabs

::: tab One-Way

单向关系有助于将一个条目链接到另一个条目。但是，只能查询其中一个模型的链接项。

##### 例子

A `pet` can be owned by someone (a `user`).

**Path —** `./api/pet/models/Pet.settings.json`.

```json
{
  "attributes": {
    "owner": {
      "model": "user"
    }
  }
}
```

**例子**

```js
// Create a pet
const xhr = new XMLHttpRequest();
xhr.open('POST', '/pets', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    owner: '5c151d9d5b1d55194d3209be', // The id of the user you want to link
  })
);
```

:::

::: tab Many-way

多向关系有助于将一个条目链接到许多其他条目。但是，只能查询其中一个模型的链接项。

##### 例子

A `pet` can be owned by many people (multiple `users`).

**Path —** `./api/pet/models/Pet.settings.json`.

```json
{
  "attributes": {
    "owners": {
      "collection": "user"
    }
  }
}
```

**例子**

```js
// Create a pet
const xhr = new XMLHttpRequest();
xhr.open('POST', '/pets', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    owners: ['5c151d9d5b1d55194d3209be', '5fc666a5bf16f48ed050ef5b'], // The id of the users you want to link
  })
);
```

:::

::: tab One-to-One

当一个实体只能链接到另一个实体时，一对一关系非常有用。**反之亦然**。

##### 例子

A `user` can have one `address`. And this address is only related to this `user`.

**Path —** `./api/user/models/User.settings.json`.

```json
{
  "attributes": {
    "address": {
      "model": "address",
      "via": "user"
    }
  }
}
```

**Path —** `./api/address/models/Address.settings.json`.

```json
{
  "attributes": {
    "user": {
      "model": "user"
    }
  }
}
```

**例子**

```js
// Create an address
const xhr = new XMLHttpRequest();
xhr.open('POST', '/addresses', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    user: '5c151d9d5b1d55194d3209be', // The id of the user you want to link
  })
);
```

:::

::: tab One-to-Many

当一个条目可以链接到另一个 Content Type 的多个条目时，一对多关系非常有用。另一个 Content Type 条目只能链接到一个条目。

##### 例子

A `user` can have many `articles`, and an `article` can be related to only one `user` (author).

**Path —** `./api/user/models/User.settings.json`.

```json
{
  "attributes": {
    "articles": {
      "collection": "article",
      "via": "author"
    }
  }
}
```

**Path —** `./api/article/models/Article.settings.json`.

```json
{
  "attributes": {
    "author": {
      "model": "user"
    }
  }
}
```

**例子**

```js
// Create an article
const xhr = new XMLHttpRequest();
xhr.open('POST', '/articles', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    author: '5c151d9d5b1d55194d3209be', // The id of the user you want to link
  })
);

// Update an article
const xhr = new XMLHttpRequest();
xhr.open('PUT', '/users/5c151d9d5b1d55194d3209be', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    articles: ['5c151d51eb28fd19457189f6', '5c151d51eb28fd19457189f8'], // Set of ALL articles linked to the user (existing articles + new article or - removed article)
  })
);
```

:::

::: tab Many-to-Many

当一个条目可以链接到另一个内容类型的多个条目时，多对多关系非常有用。另一个 Content Type 的条目可以链接到许多条目。

##### 例子

A `product` can be related to many `categories` and a `category` can have many `products`.

**Path —** `./api/product/models/Product.settings.json`.

```json
{
  "attributes": {
    "categories": {
      "collection": "category",
      "via": "products",
      "dominant": true,
      "collectionName": "products_categories__categories_products" // optional
    }
  }
}
```

**NOTE**:
(NoSQL databases only) The `dominant` key defines which table/collection should store the array that defines the relationship. Because there are no join tables in NoSQL, this key is required for NoSQL databases (e.g. MongoDB).

**NOTE**:
(NoSQL databases only) The `collectionName` key defines the name of the join table. It has to be specified once, in the `dominant` attribute of the relation. If it is not specified, Strapi will use a generated default one. It is useful to define the name of the join table when the name generated by Strapi is too long for the database you use.

**Path —** `./api/category/models/Category.settings.json`.

```json
{
  "attributes": {
    "products": {
      "collection": "product",
      "via": "categories"
    }
  }
}
```

**例子**

```js
// Update a product
const xhr = new XMLHttpRequest();
xhr.open('PUT', '/products/5c151d9d5b1d55194d3209be', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    categories: ['5c151d51eb28fd19457189f6', '5c151d51eb28fd19457189f8'], // Set of ALL categories linked to the product (existing categories + new category or - removed category).
  })
);
```

:::

::: tab Polymorphic

当您不知道哪种类型的模型将与您的条目相关联，或者当您希望将不同类型的模型连接到一个模型时，多态关系是一种解决方案。常见的用例是可以与不同类型的模型(文章、产品、用户等)相关联的 `Image` 模型。

##### Single vs Many

Let's stay with our `Image` model which might belong to **a single `Article` or `Product` entry**.

**NOTE**:
In other words, it means that an `Image` entry can be associated to one entry. This entry can be a `Article` or `Product` entry.

Also our `Image` model might belong to **many `Article` or `Product` entries**.

**NOTE**:
In other words, it means that an `Article` entry can relate to the same image as a `Product` entry.

**Path —** `./api/image/models/Image.settings.json`.

```json
{
  "attributes": {
    "related": {
      "collection": "*",
      "filter": "field"
    }
  }
}
```

##### Filter

`filter` 属性是可选的(但我们强烈建议每次都使用它)。如果提供了，则添加一个新的匹配级别来检索相关数据。

例如，`Product` 模型可能有两个与 `Image` 模型相关联的属性。为了区分哪些图像附加到封面字段和哪些图像附加到 `pictures` 字段，我们需要保存并提供给数据库。

**Path —** `./api/article/models/Product.settings.json`.

```json
{
  "attributes": {
    "cover": {
      "model": "image",
      "via": "related"
    },
    "pictures": {
      "collection": "image",
      "via": "related"
    }
  }
}
```

`filter` 属性的值是存储信息的列的名称。

##### 例子

An `Image` model might belong to many `Article` models or `Product` models.

**Path —** `./api/image/models/Image.settings.json`.

```json
{
  "attributes": {
    "related": {
      "collection": "*",
      "filter": "field"
    }
  }
}
```

**Path —** `./api/article/models/Article.settings.json`.

```json
{
  "attributes": {
    "avatar": {
      "model": "image",
      "via": "related"
    }
  }
}
```

**Path —** `./api/article/models/Product.settings.json`.

```json
{
  "attributes": {
    "pictures": {
      "collection": "image",
      "via": "related"
    }
  }
}
```

:::

::::

### Components 组件

组件字段允许您在内容类型和组件结构之间创建关系。

##### 例子

让我们说，我们创建了一个 `restaurant` 类的 `openinghours` 组件。

**Path —** `./api/restaurant/models/Restaurant.settings.json`.

```json
{
  "attributes": {
    "openinghours": {
      "type": "component",
      "repeatable": true,
      "component": "restaurant.openinghours"
    }
  }
}
```

- `repeatable` (boolean): Could be `true` or `false` that let you create a list of data.
- `component` (string): It follows this format `<category>.<componentName>`.

:::: tabs

::: tab Create

Create a restaurant with non-repeatable component

```js
const xhr = new XMLHttpRequest();
xhr.open('POST', '/restaurants', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    openinghour: {
      opening_at: '10am',
      closing_at: '6pm',
      day: 'monday',
    },
  })
);
```

Create a restaurant with repeatable component

```js
const xhr = new XMLHttpRequest();
xhr.open('POST', '/restaurants', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    openinghours: [
      {
        opening_at: '10am',
        closing_at: '6pm',
        day: 'monday',
      },
      {
        opening_at: '10am',
        closing_at: '6pm',
        day: 'tuesday',
      },
    ],
  })
);
```

:::

::: tab Update

Update a restaurant with non-repeatable component

```js
const xhr = new XMLHttpRequest();
xhr.open('PUT', '/restaurants/1', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    openinghour: {
      id: 1, // the ID of the entry
      opening_at: '11am',
      closing_at: '7pm',
      day: 'wednesday',
    },
  })
);
```

Update a restaurant with repeatable component

```js
const xhr = new XMLHttpRequest();
xhr.open('PUT', '/restaurants/2', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    openinghours: [
      {
        "id": 1 // the ID of the entry you want to update
        "opening_at": "10am",
        "closing_at": "6pm",
        "day": "monday"
      },
      {
        "id": 2, // you also have to put the ID of entries you don't want to update
        "opening_at": "10am",
        "closing_at": "6pm",
        "day": "tuesday"
      }
    ]
  })
);
```

**NOTE** if you don't specify the `ID` it will delete and re-create the entry and you will see the `ID` value change.

:::

::: tab Delete

Delete a restaurant with non-repeatable component

```js
const xhr = new XMLHttpRequest();
xhr.open('PUT', '/restaurants/1', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    openinghour: null,
  })
);
```

Delete a restaurant with repeatable component

```js
const xhr = new XMLHttpRequest();
xhr.open('PUT', '/restaurants/2', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    openinghours: [
      {
        "id": 1 // the ID of the entry you want to keep
        "opening_at": "10am",
        "closing_at": "6pm",
        "day": "monday"
      }
    ]
  })
);
```

:::

::::

### Dynamic Zone

Dynamic Zone 字段允许您创建一个灵活的空间，以便根据组件的混合列表编写内容。

##### 例子

假设我们在 `article` 类别中创建了一个 `slider` 和 `content` 组件。

**Path —** `./api/article/models/Article.settings.json`.

```json
{
  "attributes": {
    "body": {
      "type": "dynamiczone",
      "components": ["article.slider", "article.content"]
    }
  }
}
```

- `components` (array): 遵循此格式的组件数组 `<category>.<componentName>`.

:::: tabs

::: tab Create

```js
const xhr = new XMLHttpRequest();
xhr.open('POST', '/articles', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    body: [
      {
        __component: 'article.content',
        content: 'This is a content',
      },
      {
        __component: 'article.slider',
        name: 'Slider name',
      },
    ],
  })
);
```

:::

::: tab Update

```js
const xhr = new XMLHttpRequest();
xhr.open('PUT', '/restaurant/2', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    body: [
      {
        "id": 1 // the ID of the entry you want to update
        "__component": "article.content",
        "content": "This is an updated content",
      },
      {
        "id": 2, // you also have to put the ID of entries you don't want to update
        "__component": "article.slider",
        "name": "Slider name",
      }
    ]
  })
);
```

**NOTE** if you don't specify the `ID` it will delete and re-create the entry and you will see the `ID` value change.

:::

::: tab Delete

```js
const xhr = new XMLHttpRequest();
xhr.open('PUT', '/restaurant/2', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(
  JSON.stringify({
    body: [
      {
        "id": 1 // the ID of the entry you want to keep
        "__component": "article.content",
        "content": "This is an updated content",
      }
    ]
  })
);
```

:::

::::

### 生命周期挂钩

生命周期钩子是在调用 Strapi [`queries`](/developer-docs/latest/development/backend-customization.md#queries) 时触发的函数。当您在管理面板中管理内容或使用 `queries` 开发自定义代码时，它们将自动触发

要配置 `ContentType` 生命周期钩子，可以在 `{modelName}.js` 中设置 `lifecycles` 键。文件位于 `./api/{apiName}/models` 文件夹。

#### 可用的生命周期挂钩

:::: tabs

::: tab find

**`beforeFind(params, populate)`**

_Parameters:_

| Name   | Type   | Description                         |
| ------ | ------ | ----------------------------------- |
| params | Object | Find params _(e.g: limit, filters)_ |

---

**`afterFind(results, params, populate)`**

_Parameters:_

| Name     | Type          | Description                            |
| -------- | ------------- | -------------------------------------- |
| results  | Array{Object} | The results found for the `find` query |
| params   | Object        | Find params _(e.g: limit, filters)_    |
| populate | Array{string} | Populate specific relations            |

:::

::: tab findOne

**`beforeFindOne(params, populate)`**

_Parameters:_

| Name   | Type   | Description                  |
| ------ | ------ | ---------------------------- |
| params | Object | Find params _(e.g: filters)_ |

---

**`afterFindOne(result, params, populate)`**

_Parameters:_

| Name     | Type          | Description                               |
| -------- | ------------- | ----------------------------------------- |
| result   | Object        | The results found for the `findOne` query |
| params   | Object        | Find params _(e.g: filters)_              |
| populate | Array{string} | Populate specific relations               |

:::

::: tab create

**`beforeCreate(data)`**

_Parameters:_

| Name | Type   | Description                              |
| ---- | ------ | ---------------------------------------- |
| data | Object | Input data to the entry was created with |

---

**`afterCreate(result, data)`**

_Parameters:_

| Name   | Type   | Description                              |
| ------ | ------ | ---------------------------------------- |
| result | Object | Created entry                            |
| data   | Object | Input data to the entry was created with |

:::

::: tab update

**`beforeUpdate(params, data)`**

_Parameters:_

| Name   | Type   | Description                              |
| ------ | ------ | ---------------------------------------- |
| params | Object | Find params _(e.g: filters)_             |
| data   | Object | Input data to the entry was created with |

---

**`afterUpdate(result, params, data)`**

_Parameters:_

| Name   | Type   | Description                              |
| ------ | ------ | ---------------------------------------- |
| result | Object | Updated entry                            |
| params | Object | Find params _(e.g: filters)_             |
| data   | Object | Input data to the entry was created with |

:::

::: tab delete

**`beforeDelete(params)`**

_Parameters:_

| Name   | Type   | Description                  |
| ------ | ------ | ---------------------------- |
| params | Object | Find params _(e.g: filters)_ |

---

**`afterDelete(result, params)`**

_Parameters:_

| Name   | Type   | Description                  |
| ------ | ------ | ---------------------------- |
| result | Object | Deleted entry                |
| params | Object | Find params _(e.g: filters)_ |

:::

::: tab count

**`beforeCount(params)`**

_Parameters:_

| Name   | Type   | Description                  |
| ------ | ------ | ---------------------------- |
| params | Object | Find params _(e.g: filters)_ |

---

**`afterCount(result, params)`**

_Parameters:_

| Name   | Type    | Description                  |
| ------ | ------- | ---------------------------- |
| result | Integer | The count matching entries   |
| params | Object  | Find params _(e.g: filters)_ |

:::

::: tab search

**`beforeSearch(params, populate)`**

_Parameters:_

| Name     | Type          | Description                  |
| -------- | ------------- | ---------------------------- |
| params   | Object        | Find params _(e.g: filters)_ |
| populate | Array{string} | Populate specific relations  |

---

**`afterSearch(result, params)`**

_Parameters:_

| Name     | Type          | Description                  |
| -------- | ------------- | ---------------------------- |
| results  | Array{Object} | The entries found            |
| params   | Object        | Find params _(e.g: filters)_ |
| populate | Array{string} | Populate specific relations  |

:::

::: tab countSearch

**`beforeCountSearch(params)`**

_Parameters:_

| Name   | Type   | Description                  |
| ------ | ------ | ---------------------------- |
| params | Object | Find params _(e.g: filters)_ |

---

**`afterCountSearch(result, params)`**

_Parameters:_

| Name   | Type    | Description                  |
| ------ | ------- | ---------------------------- |
| result | Integer | The count matching entries   |
| params | Object  | Find params _(e.g: filters)_ |

:::

::::

#### 例子

**Path —** `./api/restaurant/models/Restaurant.js`.

```js
module.exports = {
  /**
   * Triggered before user creation.
   */
  lifecycles: {
    async beforeCreate(data) {
      data.isTableFull = data.numOfPeople === 4;
    },
  },
};
```

::: tip
您可以改变其中一个参数来改变它的属性。确保不要重新分配参数，因为它不会产生任何效果:

**This will Work**

```js
module.exports = {
  lifecycles: {
    beforeCreate(data) {
      data.name = 'Some fixed name';
    },
  },
};
```

**This will NOT Work**

```js
module.exports = {
  lifecycles: {
    beforeCreate(data) {
      data = {
        ...data,
        name: 'Some fixed name',
      };
    },
  },
};
```

:::

#### 定制使用

当您构建定制 ORM 特定查询时，生命周期将不会被触发。然而，如果您愿意，您可以直接调用生命周期函数。

**Bookshelf example**

**Path -** `./api/{apiName}/services/{serviceName}.js`

```js
module.exports = {
  async createCustomEntry() {
    const ORMModel = strapi.query(modelName).model;

    const newCustomEntry = await ORMModel.forge().save();

    // trigger manually
    ORMModel.lifecycles.afterCreate(newCustomEntry.toJSON());
  },
};
```

::: tip
当直接调用生命周期函数时，您需要确保使用预期的参数调用它。
:::

<!--- BEGINNING OF WEBHOOKS --->

## Webhooks

Webhook 是应用程序用于通知其他应用程序发生事件的构造。更准确地说，webhook 是一个用户定义的 HTTP 回调。使用 webhook 是告诉第三方提供者开始某些处理(CI、构建、部署...)的好方法。

Webhook 的工作方式是通过 HTTP 请求(通常是 POST 请求)将信息传递给接收应用程序。

### 用户内容类型的网钩

为了防止无意中将任何用户信息发送到其他应用程序，Webhooks 不适用于 User 内容类型。如果需要通知其他应用程序用户集合中的更改，可以通过在文件中创建 [Lifecycle hooks](#lifecycle-hooks) 来实现 `./extensions/users-permissions/models/User.js` .

### 可用配置

您可以在文件 `./config/server.js` 中设置 webhook 配置。

- `webhooks`
  - `defaultHeaders`: 你可以设置默认的标题来使用你的 webhook 请求。此选项被 webhook 本身中设置的头部覆盖.

**Example configuration**

```js
module.exports = {
  webhooks: {
    defaultHeaders: {
      'Custom-Header': 'my-custom-header',
    },
  },
};
```

### 安全 webhooks

大多数时候，webhooks 对公共 URL 发出请求，因此有可能有人发现这个 URL 并发送错误的信息。

为了防止这种情况发生，您可以使用认证标记发送标头。使用管理面板，你必须为每一个 webhook 做到这一点。另一种方法是定义 `defaultHeaders` 来添加到每个 webhook 请求中。

您可以通过更新 `./config/server.js` 文件来配置这些全局头文件:

:::: tabs

::: tab Simple token

```js
module.exports = {
  webhooks: {
    defaultHeaders: {
      Authorization: 'Bearer my-very-secured-token',
    },
  },
};
```

:::

::: tab Environment variable

```js
module.exports = {
  webhooks: {
    defaultHeaders: {
      Authorization: `Bearer ${process.env.WEBHOOK_TOKEN}`,
    },
  },
};
```

::::

如果您正在自己开发 webhook 处理程序，现在可以通过读取标头来验证标记。

<!--- ### Usage

To access the webhook configuration panel, go to `Settings` > `Webhooks`.

![Webhooks home](../assets/concepts/webhooks/home.png)

#### Create a webhook

点击 `Add new webhook` and fill in the form.

![create](../assets/concepts/webhooks/create.png)

#### Trigger a webhook

You can test out a webhook with a test event: `trigger-test`. Open the webhook you want to trigger.

![Trigger ](../assets/concepts/webhooks/trigger_start.png)

点击 the `Trigger` button.

![Trigger pending](../assets/concepts/webhooks/trigger.png)

You will see the trigger request appear and get the result.

![Trigger result](../assets/concepts/webhooks/trigger_result.png)

#### 启用或禁用 a webhook

You can enable or disable a webhook from the list view directly.

![Disable webhook](../assets/concepts/webhooks/disable.png)

#### Update a webhook

You can edit any webhook by clicking on the `pen` icon in the webhook list view.

![Update webhook](../assets/concepts/webhooks/list.png)

#### Delete a webhook

You can delete a webhook by clicking on the `trash` icon.

![Delete webhook](../assets/concepts/webhooks/disable.png) --->

### 可用事件

默认情况下，Strapi webhooks 可以由以下事件触发:

| Name              | Description                                           |
| ----------------- | ----------------------------------------------------- |
| `entry.create`    | Triggered when a Content Type entry is created.       |
| `entry.update`    | Triggered when a Content Type entry is updated.       |
| `entry.delete`    | Triggered when a Content Type entry is deleted.       |
| `entry.publish`   | Triggered when a Content Type entry is published.\*   |
| `entry.unpublish` | Triggered when a Content Type entry is unpublished.\* |
| `media.create`    | Triggered when a media is created.                    |
| `media.update`    | Triggered when a media is updated.                    |
| `media.delete`    | Triggered when a media is deleted.                    |

\* 仅当在此内容类型上启用 `draftAndPublish` 时。

### Payloads

:::tip NOTE
私有字段和密码不会在有效负载中发送。
:::

#### Headers

当有效负载传递到你的 webhook 的 URL 时，它将包含特定的头部:

| Header           | Description                                |
| ---------------- | ------------------------------------------ |
| `X-Strapi-Event` | Name of the event type that was triggered. |

#### `entry.create`

创建新条目时触发此事件。

**Example payload**

```json
{
  "event": "entry.create",
  "created_at": "2020-01-10T08:47:36.649Z",
  "model": "address",
  "entry": {
    "id": 1,
    "geolocation": {},
    "city": "Paris",
    "postal_code": null,
    "category": null,
    "full_name": "Paris",
    "created_at": "2020-01-10T08:47:36.264Z",
    "updated_at": "2020-01-10T08:47:36.264Z",
    "cover": null,
    "images": []
  }
}
```

#### `entry.update`

更新条目时触发此事件。

**Example payload**

```json
{
  "event": "entry.update",
  "created_at": "2020-01-10T08:58:26.563Z",
  "model": "address",
  "entry": {
    "id": 1,
    "geolocation": {},
    "city": "Paris",
    "postal_code": null,
    "category": null,
    "full_name": "Paris",
    "created_at": "2020-01-10T08:47:36.264Z",
    "updated_at": "2020-01-10T08:58:26.210Z",
    "cover": null,
    "images": []
  }
}
```

#### `entry.delete`

删除条目时触发此事件。

**Example payload**

```json
{
  "event": "entry.delete",
  "created_at": "2020-01-10T08:59:35.796Z",
  "model": "address",
  "entry": {
    "id": 1,
    "geolocation": {},
    "city": "Paris",
    "postal_code": null,
    "category": null,
    "full_name": "Paris",
    "created_at": "2020-01-10T08:47:36.264Z",
    "updated_at": "2020-01-10T08:58:26.210Z",
    "cover": null,
    "images": []
  }
}
```

#### `entry.publish`

发布条目时触发此事件。

**Example payload**

```json
{
  "event": "entry.publish",
  "created_at": "2020-01-10T08:59:35.796Z",
  "model": "address",
  "entry": {
    "id": 1,
    "geolocation": {},
    "city": "Paris",
    "postal_code": null,
    "category": null,
    "full_name": "Paris",
    "created_at": "2020-01-10T08:47:36.264Z",
    "updated_at": "2020-01-10T08:58:26.210Z",
    "published_at": "2020-08-29T14:20:12.134Z",
    "cover": null,
    "images": []
  }
}
```

#### `entry.unpublish`

未发布条目时触发此事件。

**Example payload**

```json
{
  "event": "entry.unpublish",
  "created_at": "2020-01-10T08:59:35.796Z",
  "model": "address",
  "entry": {
    "id": 1,
    "geolocation": {},
    "city": "Paris",
    "postal_code": null,
    "category": null,
    "full_name": "Paris",
    "created_at": "2020-01-10T08:47:36.264Z",
    "updated_at": "2020-01-10T08:58:26.210Z",
    "published_at": null,
    "cover": null,
    "images": []
  }
}
```

#### `media.create`

当您在创建条目时或通过媒体界面上传文件时，将触发此事件。

**Example payload**

```json
{
  "event": "media.create",
  "created_at": "2020-01-10T10:58:41.115Z",
  "media": {
    "id": 1,
    "name": "image.png",
    "hash": "353fc98a19e44da9acf61d71b11895f9",
    "sha256": "huGUaFJhmcZRHLcxeQNKblh53vtSUXYaB16WSOe0Bdc",
    "ext": ".png",
    "mime": "image/png",
    "size": 228.19,
    "url": "/uploads/353fc98a19e44da9acf61d71b11895f9.png",
    "provider": "local",
    "provider_metadata": null,
    "created_at": "2020-01-10T10:58:41.095Z",
    "updated_at": "2020-01-10T10:58:41.095Z",
    "related": []
  }
}
```

#### `media.update`

当您替换媒体或通过媒体界面更新媒体的元数据时，将触发此事件。

**Example payload**

```json
{
  "event": "media.update",
  "created_at": "2020-01-10T10:58:41.115Z",
  "media": {
    "id": 1,
    "name": "image.png",
    "hash": "353fc98a19e44da9acf61d71b11895f9",
    "sha256": "huGUaFJhmcZRHLcxeQNKblh53vtSUXYaB16WSOe0Bdc",
    "ext": ".png",
    "mime": "image/png",
    "size": 228.19,
    "url": "/uploads/353fc98a19e44da9acf61d71b11895f9.png",
    "provider": "local",
    "provider_metadata": null,
    "created_at": "2020-01-10T10:58:41.095Z",
    "updated_at": "2020-01-10T10:58:41.095Z",
    "related": []
  }
}
```

#### `media.delete`

只有在通过媒体界面删除媒体时才会触发此事件。

**Example payload**

```json
{
  "event": "media.delete",
  "created_at": "2020-01-10T11:02:46.232Z",
  "media": {
    "id": 11,
    "name": "photo.png",
    "hash": "43761478513a4c47a5fd4a03178cfccb",
    "sha256": "HrpDOKLFoSocilA6B0_icA9XXTSPR9heekt2SsHTZZE",
    "ext": ".png",
    "mime": "image/png",
    "size": 4947.76,
    "url": "/uploads/43761478513a4c47a5fd4a03178cfccb.png",
    "provider": "local",
    "provider_metadata": null,
    "created_at": "2020-01-07T19:34:32.168Z",
    "updated_at": "2020-01-07T19:34:32.168Z",
    "related": []
  }
}
```
