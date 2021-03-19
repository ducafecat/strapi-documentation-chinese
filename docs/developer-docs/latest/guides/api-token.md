# API 令牌

在本指南中，我们将了解如何创建一个 API 令牌系统来作为经过身份验证的用户执行请求。

这个特性在我们的 [roadmap](https://portal.productboard.com/strapi/1-public-roadmap/c/40-api-access-token-with-permissions) 。这个指南是一个工作方法，以实现这个功能之前，我们支持它在原生 strapi。

## 引言

目标是能够使用查询参数 `token` 以用户身份验证请求 API 终结点 `eg. /restaurants?token=my-secret-token` 。

为了在开发中实现这个功能，我们必须自定义用户权限插件。本指南将帮助您了解如何自定义所有应用程序。你可以阅读更多关于 [Strapi 插件和定制的内容](/developer-docs/latest/development/plugin-customization.md)。

## 创建令牌内容类型

要管理您的令牌，您必须创建一个新的名为 `token` 的 Content Type。

- `string` attribute named `token`
- `relation` attribute **Token** (`user`) - **Token** has and belongs to one **User** - **User** (`token`)

然后添加一些用户并创建一些链接到这些用户的令牌。

## 设置文件以覆盖

我们现在必须自定义验证 `token` 标记的函数。Strapi 有一个使用 `JWT` 令牌的 Authentication 流程，我们将重用这个函数来定制验证。

下面是管理 JWT [验证的函数](https://github.com/strapi/strapi/blob/master/packages/strapi-plugin-users-permissions/config/policies/permissions.js)。

为了能够对其进行定制，您必须在应用程序中创建一个新文件 `./extensions/users-permissions/config/policies/permissions.js`。

然后复制 GitHub 上的原始函数并粘贴到新文件中。

完成后，Strapi 应用程序将使用这个函数而不是核心函数。我们准备定制它。

## 添加令牌验证逻辑

您必须更新这个函数的第一行。

**Path —** `./extensions/users-permissions/config/policies/permissions.js`

```js
const _ = require('lodash');

module.exports = async (ctx, next) => {
  let role;

 if (ctx.state.user) {
    // request is already authenticated in a different way
    return next();
  }

  // add the detection of `token` query parameter
  if (
    (ctx.request && ctx.request.header && ctx.request.header.authorization) ||
    (ctx.request.query && ctx.request.query.token)
    ) {
    try {
      // init `id` and `isAdmin` outside of validation blocks
      let id;
      let isAdmin;

      if (ctx.request.query && ctx.request.query.token) {
        // find the token entry that match the token from the request
        const [token] = await strapi.query('token').find({token: ctx.request.query.token});

        if (!token) {
          throw new Error(`Invalid token: This token doesn't exist`);
        } else {
          if (token.user && typeof token.token === 'string') {
            id = token.user.id;
          }
          isAdmin = false;
        }

        delete ctx.request.query.token;
      } else if (ctx.request && ctx.request.header && ctx.request.header.authorization) {
        // use the current system with JWT in the header
        const decrypted = await strapi.plugins[
          'users-permissions'
        ].services.jwt.getToken(ctx);

        id = decrypted.id;
        isAdmin = decrypted.isAdmin || false;
      }

      // this is the line that already exist in the code
      if (id === undefined) {
        throw new Error('Invalid token: Token did not contain required fields');
      }

      ...
```

还有，嗒嗒！您现在可以创建一个令牌，将其链接到用户，并在 url 中使用它作为查询参数的令牌。
