# 错误捕获

在本指南中，我们将了解如何捕获错误并将其发送到所需的应用程序监视/错误跟踪软件。

::: tip
在这个例子中，我们将使用 [Sentry](https://sentry.io)。
:::

## 创建一个中间件

将使用一个 [中间件](/developer-docs/latest/setup-deployment-guides/configurations.md#middlewares) 来捕捉错误，然后将其发送到 Sentry。

- 创建 `./middlewares/sentry/index.js`.

**Path —** `./middlewares/sentry/index.js`

```js
module.exports = strapi => {
  return {
    initialize() {
      strapi.app.use(async (ctx, next) => {
        await next();
      });
    },
  };
};
```

## 处理错误

这是 [Node.js 客户端文档](https://docs.sentry.io/platforms/node/)。

安装 `yarn add @sentry/node` or `npm install @sentry/node --save`.

- 然后写一些逻辑，我们来尝试捕捉错误

**Path —** `./middlewares/sentry/index.js`

```js
const Sentry = require('@sentry/node');
Sentry.init({
  dsn: 'https://<key>@sentry.io/<project>',
  environment: strapi.config.environment,
});

module.exports = strapi => {
  return {
    initialize() {
      strapi.app.use(async (ctx, next) => {
        try {
          await next();
        } catch (error) {
          Sentry.captureException(error);
          throw error;
        }
      });
    },
  };
};
```

::: warning
调用 `throw(error);` 非常重要，以避免停止中间件堆栈。如果您不重新抛出错误，那么它将不会被 Strapi 的错误格式化程序处理，并且 api 将永远不会响应客户机。
:::

## 配置中间件

为了确保您的中间件捕获所有错误，包括其他中间件中的错误，需要在堆栈的开始添加它，但要在 `boom` 中间件之后添加。

之后，哨兵中间件捕获并重新抛出错误，`boom` 将向客户机返回适当的 HTTP 响应。

**Path —** `./config/middleware.js`

```js
module.exports = {
  load: {
    before: ['boom', 'sentry', ...],
    ...
  },
};
```

最后，您必须启用中间件。

**Path —** `./config/middleware.js`

```js
module.exports = {
  settings: {
    sentry: {
      enabled: true,
    },
  },
};
```
