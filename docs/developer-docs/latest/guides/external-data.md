# 获取外部数据

本指南解释了如何从外部服务中获取数据以便在应用程序中使用。

在这个示例中，我们将看到如何每天获取 Docker 拉计数，以便将结果存储在数据库中。

## 内容类型设置

首先，我们需要创建一个 `Content Type`，在这个例子中，我们将它命名为 `date` 和 `count` 属性 `hit`。

你的内容类型应该是这样的:

**Path —** `./api/hit/models/Hit.settings.json`

```json
{
  "connection": "default",
  "collectionName": "hits",
  "info": {
    "name": "hit",
    "description": ""
  },
  "options": {
    "increments": true,
    "timestamps": true,
    "comment": ""
  },
  "attributes": {
    "count": {
      "type": "integer"
    },
    "date": {
      "type": "date"
    }
  }
}
```

## 获取数据

现在我们将创建一个函数，该函数将在您的 strapi 应用程序中的任何地方都可用。

**Path —** `./config/functions/docker.js`

```js
const axios = require('axios');

module.exports = async () => {
  const { data } = await axios.get('https://hub.docker.com/v2/repositories/strapi/strapi/');

  console.log(data);
};
```

`data` 包含从 dockerhub API 接收的所有数据。这里我们需要在数据库中添加 `pull_count` 值。

## 创建一个 `hit entry` 条目

让我们以编程方式创建条目。

**Path —** `./config/functions/docker.js`

```js
const axios = require('axios');

module.exports = async () => {
  const { data } = await axios.get('https://hub.docker.com/v2/repositories/strapi/strapi/');

  await strapi.query('hit').create({
    date: new Date(),
    count: data.pull_count,
  });
};
```

使用这段代码，每次调用这个函数时，它都会获取 docker repo 的数据，并在 Strapi 数据库中插入当前的 `pull_count` 和相应的日期。

## 调用函数

下面是如何调用应用程序 `strapi.config.functions.docker()` 中的函数

所以让我们每天凌晨 2 点执行这个函数，为此我们将使用 [CRON tasks](/developer-docs/latest/setup-deployment-guides/configurations.md#cron-tasks)。

**Path —** `./config/functions/cron.js`

```js
module.exports = {
  '0 2 * * *': () => {
    strapi.config.functions.docker();
  },
};
```
