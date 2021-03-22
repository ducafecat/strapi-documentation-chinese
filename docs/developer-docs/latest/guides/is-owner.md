# 创建所有者策略

本指南将解释如何仅限制内容编辑内容的作者。

## 引言

通常要求条目的作者是唯一允许编辑或删除条目的用户。

这是一个要求很多的特性，在本指南中我们将展示如何实现它。

## 例子

对于这个例子，我们需要一个文章内容类型。

为此 Content Type 添加 `text` 字段和 `relation` 字段。

`relation` 字段是与用户的 **many-to-one** 关系。一个用户可以有多个文章和一个文章可以只有一个用户。

指定文章内容类型和用户端文章的字段作者。

现在我们可以开始定制了。

## 默认情况下作者

当我们通过 `POST/articles` 创建一个新文章时，我们需要将经过身份验证的用户设置为文章的作者。

为此，我们将自定义 Article API 的 `create` `controller` 函数。

**我们的逻辑是这样:**
先看这里控制器核心代码 [core controllers](/developer-docs/latest/development/backend-customization.md#core-controllers) 的代码。我们还将使用此 [文章](/developer-docs/latest/development/plugins/users-permissions.md#user-object-in-strapi-context) 访问当前经过身份验证的用户信息。

**Path —** `./api/article/controllers/Article.js`

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
      data.author = ctx.state.user.id;
      entity = await strapi.services.article.create(data, { files });
    } else {
      ctx.request.body.author = ctx.state.user.id;
      entity = await strapi.services.article.create(ctx.request.body);
    }
    return sanitizeEntity(entity, { model: strapi.models.article });
  },
};
```

现在，当创建一篇文章时，经过身份验证的用户将自动设置为文章的作者。

## 限制更新

现在，我们将只对作者更新文章。

我们将使用与前面相同的概念。

**Path —** `./api/article/controllers/Article.js`

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
      data.author = ctx.state.user.id;
      entity = await strapi.services.article.create(data, { files });
    } else {
      ctx.request.body.author = ctx.state.user.id;
      entity = await strapi.services.article.create(ctx.request.body);
    }
    return sanitizeEntity(entity, { model: strapi.models.article });
  },

  /**
   * Update a record.
   *
   * @return {Object}
   */

  async update(ctx) {
    const { id } = ctx.params;

    let entity;

    const [article] = await strapi.services.article.find({
      id: ctx.params.id,
      'author.id': ctx.state.user.id,
    });

    if (!article) {
      return ctx.unauthorized(`You can't update this entry`);
    }

    if (ctx.is('multipart')) {
      const { data, files } = parseMultipartData(ctx);
      entity = await strapi.services.article.update({ id }, data, {
        files,
      });
    } else {
      entity = await strapi.services.article.update({ id }, ctx.request.body);
    }

    return sanitizeEntity(entity, { model: strapi.models.article });
  },
};
```

还有，嗒嗒！

::: tip
对于删除操作，它将与更新操作完全相同。
:::
