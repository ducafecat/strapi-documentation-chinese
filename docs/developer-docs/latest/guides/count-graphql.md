# 使用 GraphQL 进行计数

本指南解释了如何使用 GraphQL 查询计数数据。

计数聚合目前有一些问题，该特性也不适用于 `Bookshelf` (SQL 数据库)。

有了这个指南，我们将编写我们自己的计数查询。

下面是 [GraphQL 文档](/developer-docs/latest/development/plugins/graphql.md#customize-the-graphql-schema) ，我们将使用它来实现我们的目标。

## 安装应用程序

在本例中，我们将使用一个 **Restaurant** API。

确保你有一个包含一些条目的 **Content Type**。

## 创建 schema.graphql 文件

为了能够添加一个新的自定义查询(或 mutation ) ，我们必须在您的 **Restaurant** API 中创建一个 `schema.graphql.js` 文件。

**Path** — `./api/restaurant/config/schema.graphql.js`

```js
module.exports = {
  query: ``,
  resolver: {
    Query: {},
  },
};
```

## 创建计数查询

`count` 查询将调用 `Restaurant` API 的 [`count`](/developer-docs/latest/development/backend-customization.md#core-services) 服务函数。

它需要一个 JSON 对象作为参数，因此我们将在 GraphQL 查询中添加一个 `where` 选项。

**Path** — `./api/restaurant/config/schema.graphql.js`

```js
module.exports = {
  query: `
    restaurantsCount(where: JSON): Int!
  `,
  resolver: {
    Query: {
      restaurantsCount: {
        description: 'Return the count of restaurants',
        resolverOf: 'application::restaurant.restaurant.count',
        resolver: async (obj, options, ctx) => {
          return await strapi.api.restaurant.services.restaurant.count(options.where || {});
        },
      },
    },
  },
};
```

而且，tada，你现在可以请求你的内容类型的 `count`。

## 查询示例

- 数一数所有的餐馆

```
{
  restaurantsCount
}
```

- 计算所有的餐厅，有 `_3rd` 作为地区价值

基于 [FoodAdvisor](https://github.com/strapi/foodadvisor) 约束模型。

```
{
  restaurantsCount(where: { district: "_3rd" })
}
```
