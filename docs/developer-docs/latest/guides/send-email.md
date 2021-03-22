# 以编程方式发送电子邮件

在本指南中，我们将看到如何使用电子邮件插件发送电子邮件到你想要的地方在你的应用程序。

对于这个例子，我们希望收到一封电子邮件，当一篇新的文章的评论被张贴，如果它包含不好的话。

## 引言

这里我们需要添加一些自定义逻辑，并在通过 `POST/comments` 端点创建 `Comment` 时调用电子邮件服务。

要做到这一点，你必须首先理解一些概念。

当您创建一个内容类型时，它会生成一个带有以下 [endpoints](/developer-docs/latest/developer-resources/content-api/content-api.md#api-endpoints) 列表的 API。

这些端点中的每一个都触发一个 [controller actions](/developer-docs/latest/development/backend-customization.md#controllers) 。下面是创建内容类型时默认存在的控制器操作的列表。

如果您检查您生成的 API 的控制器文件 `./api/{content-type}/controller/{Content-Type}.js` ，你会看到一个空文件。这是因为所有的默认逻辑都是由 Strapi 管理的。但是您可以使用自己的代码覆盖这些操作。

这就是我们添加自定义代码所要做的。

## 例子

为了使代码示例易于理解，我们只需要一个 `Comment` content 类型，省略 `Author` 和 `Article` 关系。

因此，让我们创建一个只有一个名为 `content` 的 **Text** 字段的 `Comment` content 类型。

创建内容类型时，允许为 Public 角色创建函数。

为了检查注释中是否有坏词，我们将使用 `bad-words` [node module](https://www.npmjs.com/package/bad-words) 。你必须在你的应用程序中安装它。

## 覆盖控制器操作

要自定义创建注释的函数，我们必须重写 `create` 函数。

首先，为了看到差异，让我们请求 `POST/comment` `that is fucking good!`,

对于内容属性。你会看到你的评论被成功创建。

现在让我们开始定制。

**Path —** `./api/comment/controller/Comment.js`

```js
module.exports = {
  async create() {
    return 'strapi';
  },
};
```

在保存新函数之后，让我们重新启动 `POST /comment` 请求。

## 返回注释创建

现在我们知道了需要更新的函数，让我们回到原来的函数。

在 [控制器文档](/developer-docs/latest/development/backend-customization.md#extending-a-model-controller) 中，您将找到每个操作的默认实现。它将帮助您覆盖创建逻辑。

**Path —** `./api/comment/controller/Comment.js`

```js
const { parseMultipartData, sanitizeEntity } = require('strapi-utils');

module.exports = {
  async create(ctx) {
    let entity;
    if (ctx.is('multipart')) {
      const { data, files } = parseMultipartData(ctx);
      entity = await strapi.services.comment.create(data, { files });
    } else {
      entity = await strapi.services.comment.create(ctx.request.body);
    }
    return sanitizeEntity(entity, { model: strapi.models.comment });
  },
};
```

现在评论的创造又回来了。

## 应用我们的改变

我们想要检查评论的内容是否包含任何不好的词语。

如果有，我们想使用 [Email plugin](/developer-docs/latest/development/plugins/email.md) 发送邮件

**Path —** `./api/comment/controller/Comment.js`

```js
const { parseMultipartData, sanitizeEntity } = require('strapi-utils');

const Filter = require('bad-words');
const filter = new Filter();

module.exports = {
  async create(ctx) {
    let entity;
    if (ctx.is('multipart')) {
      const { data, files } = parseMultipartData(ctx);
      entity = await strapi.services.comment.create(data, { files });
    } else {
      entity = await strapi.services.comment.create(ctx.request.body);
    }

    entity = sanitizeEntity(entity, { model: strapi.models.comment });

    // check if the comment content contains a bad word
    if (entity.content !== filter.clean(entity.content)) {
      // send an email by using the email plugin
      await strapi.plugins['email'].services.email.send({
        to: 'paulbocuse@strapi.io',
        from: 'admin@strapi.io',
        subject: 'Comment posted that contains a bad words',
        text: `
          The comment #${entity.id} contain a bad words.

          Comment:
          ${entity.content}
        `,
      });
    }

    return entity;
  },
};
```

塔达，它起作用了。
