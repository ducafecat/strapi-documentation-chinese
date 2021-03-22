# JWT 验证

在本指南中，我们将了解如何使用第三方服务验证 `JWT` (JSON Web Token)。

当您使用身份验证路由 `POST /auth/local` 登录时，Strapi 生成一个 `JWT`，它允许您的用户将您的 API 请求为身份验证的 API。

```json
{
  "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNTcxODIyMDAzLCJleHAiOjE1NzQ0MTQwMDN9.T5XQGSDZ6TjgM5NYaVDbYJt84qHZTrtBqWu1Q3ShINw",
  "user": {
    "email": "admin@strapi.io",
    "id": 1,
    "username": "admin"
  }
}
```

这些用户在应用程序的数据库中进行管理，可以通过管理仪表板进行管理。

我们现在可以想象您有一个来自 [Auth0](https://auth0.com) 的 `JWT` ，并且您希望在允许用户使用 Strapi API 端点之前确保 `JWT` 是正确的。

## 自定义 JWT 验证函数

我们将更新验证 `JWT` 的函数。该功能由 **Users & Permissions** [插件](/developer-docs/latest/development/plugin-customization.md) 提供。

下面是我们必须定制的文件: [permission.js](https://github.com/strapi/strapi/blob/master/packages/strapi-plugin-users-permissions/config/policies/permissions.js)

- 我们必须创建一个遵循此路径的文件 `./extensions/users-permissions/config/policies/permissions.js`.
- 你必须在这个新文件中添加与原始文件相同的内容.

现在我们可以创建自定义验证代码了。

## 写出我们自己的逻辑

首先，我们必须定义我们在哪里编写代码。

```js
const _ = require('lodash');

module.exports = async (ctx, next) => {
  let role;

  if (ctx.request && ctx.request.header && ctx.request.header.authorization) {
    try {
      const { id, isAdmin = false } = await strapi.plugins[
        'users-permissions'
      ].services.jwt.getToken(ctx);

      ...

    } catch (err) {
      // It will be there!

      return handleErrors(ctx, err, 'unauthorized');
    }
```

如果令牌不是来自 Strapi，则 `jwt.getToken` 将抛出一个错误。因此，如果它不是 Strapi `JWT` 令牌，那么让我们测试它是否是 `Auth0` 令牌。

在抛出错误之前，我们必须编写验证代码。

通过使用 [Auth0 get user profile](https://auth0.com/docs/api/authentication?http#get-user-info) 文档，您将验证有效用户与当前 `JWT` 匹配

```js
const _ = require('lodash');
const axios = require('axios');

module.exports = async (ctx, next) => {
  let role;

  if (ctx.request && ctx.request.header && ctx.request.header.authorization) {
    try {
      const { id, isAdmin = false } = await strapi.plugins[
        'users-permissions'
      ].services.jwt.getToken(ctx);

      ...

    } catch (err) {
      try {
        const data = await axios({
          method: 'post',
          url: 'http://YOUR_DOMAIN/userinfo',
          headers: {
            Authorization: ctx.request.header.authorization
          }
        });

        // if you want do more validation test
        // feel free to add your code here.

        return await next();
      } catch (error) {
        return handleErrors(ctx, new Error('Invalid token: Token did not match with Strapi and Auth0'), 'unauthorized');
      }
    }
```

::: warning
在代码示例中，我们使用 `axios`，因此必须安装依赖项才能使其工作。如果您愿意，您可以选择另一个库。
:::
