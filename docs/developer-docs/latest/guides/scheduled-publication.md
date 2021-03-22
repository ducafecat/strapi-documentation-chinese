# 预定出版物

本指南将解释如何创建一个文章计划系统。

## 引言

这里我们想要的是能够为一篇文章设置一个发布日期，并且在这个日期，将`草稿`状态切换到`发布`状态。

## 例子

对于这个示例，我们必须向 Article Content Type 添加 `publish_at` 属性。

- 点击 the Content Type Builder link 在左边的菜单里
- 选择 **Article** Content Type
- 添加 another field
  - `date` attribute named `publish_at` with `datetime` type

并添加一些具有不同日期和状态的数据，以便能够查看发布过程。

确保在当前日期之前创建一些带有草稿状态和 `publish_at` 的条目。

我们的目标是每分钟都检查是否有条款草案的`publish_at` 发表时间比当前日期低。

## 创建一个 CRON 任务

为了每分钟执行一个函数，我们将使用 CRON 任务。

下面是这个特性的 [完整文档](/developer-docs/latest/setup-deployment-guides/configurations.md#cron-tasks) 。如果 CRON 任务需要基于特定时区运行，请查看完整的文档。

**Path —** `./config/functions/cron.js`

```js
module.exports = {
  '*/1 * * * *': () => {
    console.log('1 minute later');
  },
};
```

确保启用的 cron config 在 `./config/server.js` 文件中设置为 true。

::: tip
请注意，如果您计划使用 `pm2` 或基于节点的集群，Strapi 的内置 CRON 特性将不起作用。你需要在 Strapi 以外的地方执行这些 CRON 任务。
:::

## 业务逻辑

现在我们可以开始编写发布逻辑了。获取所有带有 `publish_at` 的条款草案的代码在当前日期之前。

然后我们将更新所有这些文章中已发表的 `published_at`。

**Path —** `./config/functions/cron.js`

```js
module.exports = {
  '*/1 * * * *': async () => {
    // fetch articles to publish
    const draftArticleToPublish = await strapi.api.article.services.article.find({
      _publicationState: 'preview',
      publish_at_lt: new Date(),
    });

    // update published_at of articles
    draftArticleToPublish.forEach(async article => {
      await strapi.api.article.services.article.update(
        { id: article.id },
        { published_at: new Date() }
      );
    });
  },
};
```

And tada!
