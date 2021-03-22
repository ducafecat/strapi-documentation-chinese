# 草稿系统

本指南将解释如何创建一个草稿系统，允许您管理草案、发布和归档状态。

::: warning
原生的 `Draft & Publish 特性` 已经在 3.2 版本中发布。我们建议您使用本地特性而不是本指南。

如果你想了解“强制过滤”的概念，这篇指南仍然很有用。
:::

## 引言

这里我们想要的是只获取具有 `published` 状态的数据。

但是我们不想使用 [parameters](/developer-docs/latest/developer-resources/content-api/content-api.md#api-parameters) (比如 `/articles?status=published` ) ，因为您可以轻松地伪造参数。

要做到这一点，你必须首先理解一些概念。

当您创建一个内容类型时，它会生成一个带有以下 [endpoints](/developer-docs/latest/developer-resources/content-api/content-api.md#api-endpoints) 列表的 API。

这些端点中的每一个都触发一个 [控制器操作](/developer-docs/latest/development/backend-customization.md#controllers)。下面是创建内容类型时默认存在的控制器操作的列表。

如果您检查您生成的 API 的控制器文件。`./api/{content-type}/controller/{Content-Type}.js`，你会看到一个空文件。这是因为所有的默认逻辑都是由 Strapi 管理的。但是您可以使用自己的代码覆盖这些操作。

这就是我们将要做的，默认情况下过滤到 `published` 状态。

## 例子

在我们的示例中，我们将使用 Article 内容类型。默认情况下，当您获取文章时，您将获得所有文章。假设您不想公开处于 `draft` 或 `archive` 状态的文章。

为了强制执行这个规则，我们将自定义获取所有文章的操作，以只获取已 `published` 的文章。

要遵循这个例子，你需要创建一个内容类型的 `articles`，并添加以下字段定义:

- `string` attribute named `title`
- `text` attribute named `content`
- `enumeration` attribute named `status` with `draft`, `published`, `archive`

```js
"attributes": {
    "title": {
      "type": "string"
    },
    "content": {
      "type": "text"
    },
    "status": {
      "type": "enumeration",
      "enum": [
        "archive",
        "draft",
        "published"
      ]
    }
  }
```

然后添加一些不同 `status` 的数据。

## 覆盖控制器操作

要自定义获取所有文章的函数，我们必须重写 `find` 函数。

首先，为了看出区别，让我们请求 `GET /articles`。您将看到您创建的所有数据。

现在让我们开始定制。

**Path —** `./api/article/controller/Article.js`

```js
module.exports = {
  async find() {
    return 'strapi';
  },
};
```

保存新函数之后，让我们重新启动 `GET /articles` 请求。

## 把数据拿回来

我们现在知道了需要更新的函数，但是我们只想定制返回的项目值。

在 [控制器 文章](/developer-docs/latest/development/backend-customization.md#extending-a-model-controller) 中，您将找到每个操作的默认实现。它将帮助您覆盖提取逻辑。

**Path —** `./api/article/controller/Article.js`

```js
const { sanitizeEntity } = require('strapi-utils');

module.exports = {
  async find(ctx) {
    let entities;
    if (ctx.query._q) {
      entities = await strapi.services.article.search(ctx.query);
    } else {
      entities = await strapi.services.article.find(ctx.query);
    }

    return entities.map(entity => sanitizeEntity(entity, { model: strapi.models.article }));
  },
};
```

现在数据又回到了 `GET /articles` 上

## 应用我们的改变

在这里，我们希望强制它获取具有等于已 `published` 的状态的文章。

这样做的方法是将 `ctx.query.status` 设置为 `published`。

**Path —** `./api/article/controller/Article.js`

```js
const { sanitizeEntity } = require('strapi-utils');

module.exports = {
  async find(ctx) {
    let entities;

    ctx.query = {
      ...ctx.query,
      status: 'published',
    };

    if (ctx.query._q) {
      entities = await strapi.services.article.search(ctx.query);
    } else {
      entities = await strapi.services.article.find(ctx.query);
    }

    return entities.map(entity => sanitizeEntity(entity, { model: strapi.models.article }));
  },
};
```

草稿和存档的文章不见了。

::: tip
本指南可以应用于任何其他控制器操作。
:::
