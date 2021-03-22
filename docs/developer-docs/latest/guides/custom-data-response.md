# 自定义数据响应

在本指南中，我们将看到如何定制 API 的响应。

## 引言

为了能够更新默认数据响应，必须首先了解它是如何工作的

当您创建一个内容类型时，它会生成一个带有以下 [endpoints](/developer-docs/latest/developer-resources/content-api/content-api.md#api-endpoints) 列表的 API。

这些端点中的每一个都触发一个 [控制器操作](/developer-docs/latest/development/backend-customization.md#controllers) 。下面是创建内容类型时默认存在的控制器操作的列表。

如果您检查您生成的 API 的控制器文件 `./api/{content-type}/controller/{Content-Type}.js` ，你会看到一个空文件。这是因为所有的默认逻辑都是由 Strapi 管理的。但是您可以使用自己的代码覆盖这些操作。

这就是我们管理自定义数据响应所要做的。

## 例子

在我们的示例中，我们将使用一个带有厨师的餐厅类型。默认情况下，当您取餐厅时，您将获得厨师的所有信息。让我们考虑一下，出于隐私原因，你不想暴露厨师的邮件。

为了强制执行这个规则，我们将自定义获取所有餐馆的操作，并从返回的数据中删除电子邮件。

按照这个例子，你需要创建一个内容类型的 `restaurant` ，并添加以下字段定义:

- `string` attribute named `name`
- `text` attribute named `description`
- `relation` attribute **Restaurant** (`chef`) - **User** has many **Restaurants** - **Users** (`restaurants`)

然后添加一些数据。

## 覆盖控制器操作

为了定制获取所有餐厅的函数，我们必须覆盖 `find` 函数。

首先，为了看出区别，让我们请求 `GET /restaurants`。您将看到您创建的所有数据。现在让我们开始定制。

**Path —** `./api/restaurant/controller/Restaurant.js`

```js
module.exports = {
  async find() {
    return 'strapi';
  },
};
```

保存新函数之后，让我们重新启动 `GET /restaurants` 请求。

## 把数据拿回来

我们现在知道了需要更新的函数，但是我们只是想定制返回的 `restaurant` 值。

在 [controller 文章](/developer-docs/latest/development/backend-customization.md#extending-a-model-controller) 中，您将找到每个操作的默认实现。它将帮助您覆盖提取逻辑。

**Path —** `./api/restaurant/controller/Restaurant.js`

```js
const { sanitizeEntity } = require('strapi-utils');

module.exports = {
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

现在，关于 `GET /restaurants` 的数据又回来了

## 应用我们的改变

我们可以看到 `find` 函数返回 `map` 的结果，而 `map` 函数只是对所有条目进行 `sanitizes` 。

因此，我们不仅要返回经过 `sanitized` 的条目，还要删除每个餐厅的厨师电子邮件。

**Path —** `./api/restaurant/controller/Restaurant.js`

```js
const { sanitizeEntity } = require('strapi-utils');

module.exports = {
  async find(ctx) {
    let entities;
    if (ctx.query._q) {
      entities = await strapi.services.restaurant.search(ctx.query);
    } else {
      entities = await strapi.services.restaurant.find(ctx.query);
    }

    return entities.map(entity => {
      const restaurant = sanitizeEntity(entity, {
        model: strapi.models.restaurant,
      });
      if (restaurant.chef && restaurant.chef.email) {
        delete restaurant.chef.email;
      }
      return restaurant;
    });
  },
};
```

然后，电子邮件消失了。

::: tip
本指南可以应用于任何其他控制器操作。
:::
