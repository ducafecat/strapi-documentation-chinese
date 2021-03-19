---
sidebarDepth: 3
---

# 内容 API

## API Endpoints

当您创建一个 `Content Type` 时，您将有一定数量的 **REST API endpoints** 端点可用于与它交互。

::: warning
组件没有 API 端点
:::

作为一个 **例子**，让我们考虑以下模型:

**Content Types**:

- `Restaurant` (Collection Type)
- `Homepage` (Single Type)

**Components**:

- `Opening hours` (category: `restaurant`)
- `Title With Subtitle` (category: `content`)
- `Image With Description` (category: `content`)

---

:::: tabs

::: tab Content Types

#### `Restaurant` Content Type

| Fields        | Type        | Description                          | Options              |
| :------------ | :---------- | :----------------------------------- | :------------------- |
| name          | string      | Restaurant's title                   |                      |
| slug          | uid         | Restaurant's slug                    | `targetField="name"` |
| cover         | media       | Restaurant's cover image             |                      |
| content       | dynamiczone | The restaurant profile content       |                      |
| opening_hours | component   | Restaurant's opening hours component | `repeatable`         |

---

#### `Homepage` Content Type

| Fields   | Type        | Description        | Options |
| :------- | :---------- | :----------------- | :------ |
| title    | string      | Homepage title     |         |
| subTitle | string      | Homepage sub title |         |
| content  | dynamiczone | Homepage content   |         |

:::

::: tab Components

#### `Opening hours` Component

| Fields       | Type   | Description         |
| :----------- | :----- | :------------------ |
| day_interval | string | Meta's day interval |
| opening_hour | string | Meta's opening hour |
| closing_hour | string | Meta's closing hour |

---

#### `Title With Subtitle` Component

| Fields   | Type   | Description   |
| :------- | :----- | :------------ |
| title    | string | The title     |
| subTitle | string | The sub title |

---

#### `Image With Description` Component

| Fields      | Type   | Description           |
| :---------- | :----- | :-------------------- |
| image       | media  | The image file        |
| title       | string | The image title       |
| description | text   | The image description |

:::

::::

### Endpoints

下面是为每种 **Content Types** 生成的端点列表。

<style lang="stylus">
#endpoint-table
  table
    display table
    width 100%

  tr
    border none
    &:nth-child(2n)
      background-color white

  tbody
    tr
      border-top 1px solid #dfe2e5

  th, td
    border none
    padding 1.2em 1em
    border-right 1px solid #dfe2e5
    &:last-child
      border-right none


</style>

:::: tabs

::: tab Collection Type

<div id="endpoint-table">

| Method | Path                    | Description                          |
| :----- | :---------------------- | :----------------------------------- |
| GET    | `/{content-type}`       | Get a list of {content-type} entries |
| GET    | `/{content-type}/:id`   | Get a specific {content-type} entry  |
| GET    | `/{content-type}/count` | Count {content-type} entries         |
| POST   | `/{content-type}`       | Create a {content-type} entry        |
| DELETE | `/{content-type}/:id`   | Delete a {content-type} entry        |
| PUT    | `/{content-type}/:id`   | Update a {content-type} entry        |

</div>

:::

::: tab Single Type

<div id="endpoint-table">

| Method | Path              | Description                       |
| :----- | :---------------- | :-------------------------------- |
| GET    | `/{content-type}` | Get the {content-type} content    |
| PUT    | `/{content-type}` | Update the {content-type} content |
| DELETE | `/{content-type}` | Delete the {content-type} content |

</div>

:::

::::

#### 例子

下面是一些 Content Type 示例

##### Single Types

:::: tabs

::: tab Homepage

`Homepage` **Content Type**

<div id="endpoint-table">

| Method | Path        | Description                 |
| :----- | :---------- | :-------------------------- |
| GET    | `/homepage` | Get the homepage content    |
| PUT    | `/homepage` | Update the homepage content |
| DELETE | `/homepage` | Delete the homepage content |

</div>

:::

::: tab Contact

`Contact` **Content Type**

<div id="endpoint-table">

| Method | Path       | Description                |
| :----- | :--------- | :------------------------- |
| GET    | `/contact` | Get the contact content    |
| PUT    | `/contact` | Update the contact content |
| DELETE | `/contact` | Delete the contact content |

</div>

:::

::::

##### Collection Types

:::: tabs

::: tab Restaurant

`Restaurant` **Content Type**

<div id="endpoint-table">

| Method | Path                 | Description               |
| :----- | :------------------- | :------------------------ |
| GET    | `/restaurants`       | Get a list of restaurants |
| GET    | `/restaurants/:id`   | Get a specific restaurant |
| GET    | `/restaurants/count` | Count restaurants         |
| POST   | `/restaurants`       | Create a restaurant       |
| DELETE | `/restaurants/:id`   | Delete a restaurant       |
| PUT    | `/restaurants/:id`   | Update a restaurant       |

</div>

:::

::: tab Article

`Article` **Content Type**

<div id="endpoint-table">

| Method | Path              | Description            |
| :----- | :---------------- | :--------------------- |
| GET    | `/articles`       | Get a list of articles |
| GET    | `/articles/:id`   | Get a specific article |
| GET    | `/articles/count` | Count articles         |
| POST   | `/articles`       | Create a article       |
| DELETE | `/articles/:id`   | Delete a article       |
| PUT    | `/articles/:id`   | Update a article       |

</div>

:::

::: tab Product

`Product` **Content Type**

<div id="endpoint-table">

| Method | Path              | Description            |
| :----- | :---------------- | :--------------------- |
| GET    | `/products`       | Get a list of products |
| GET    | `/products/:id`   | Get a specific product |
| GET    | `/products/count` | Count products         |
| POST   | `/products`       | Create a product       |
| DELETE | `/products/:id`   | Delete a product       |
| PUT    | `/products/:id`   | Update a product       |

</div>

:::

::: tab Category

`Category` **Content Type**

<div id="endpoint-table">

| Method | Path                | Description              |
| :----- | :------------------ | :----------------------- |
| GET    | `/categories`       | Get a list of categories |
| GET    | `/categories/:id`   | Get a specific category  |
| GET    | `/categories/count` | Count categories         |
| POST   | `/categories`       | Create a category        |
| DELETE | `/categories/:id`   | Delete a category        |
| PUT    | `/categories/:id`   | Update a category        |

</div>

:::

::: tab Tag

`Tag` **Content Type**

<div id="endpoint-table">

| Method | Path          | Description        |
| :----- | :------------ | :----------------- |
| GET    | `/tags`       | Get a list of tags |
| GET    | `/tags/:id`   | Get a specific tag |
| GET    | `/tags/count` | Count tags         |
| POST   | `/tags`       | Create a tag       |
| DELETE | `/tags/:id`   | Delete a tag       |
| PUT    | `/tags/:id`   | Update a tag       |

</div>

:::

::::

### Get entries

返回与查询过滤器匹配的条目。您可以在这里阅读更多有关参数的内容 [这里](#api-parameters) 。

:::: tabs

::: tab Request

**示例 request**

```js
GET http://localhost:1337/restaurants
```

:::

::: tab Response

**示例 response**

```json
[
  {
    "id": 1,
    "name": "Restaurant 1",
    "cover": {
      "id": 1,
      "name": "image.png",
      "hash": "123456712DHZAUD81UDZQDAZ",
      "sha256": "v",
      "ext": ".png",
      "mime": "image/png",
      "size": 122.95,
      "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
      "provider": "local",
      "provider_metadata": null,
      "created_at": "2019-12-09T00:00:00.000Z",
      "updated_at": "2019-12-09T00:00:00.000Z"
    },
    "content": [
      {
        "__component": "content.title-with-subtitle",
        "id": 1,
        "title": "Restaurant 1 title",
        "subTitle": "Cozy restaurant in the valley"
      },
      {
        "__component": "content.image-with-description",
        "id": 1,
        "image": {
          "id": 1,
          "name": "image.png",
          "hash": "123456712DHZAUD81UDZQDAZ",
          "sha256": "v",
          "ext": ".png",
          "mime": "image/png",
          "size": 122.95,
          "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
          "provider": "local",
          "provider_metadata": null,
          "created_at": "2019-12-09T00:00:00.000Z",
          "updated_at": "2019-12-09T00:00:00.000Z"
        },
        "title": "Amazing photography",
        "description": "This is an amazing photography taken..."
      }
    ],
    "opening_hours": [
      {
        "id": 1,
        "day_interval": "Tue - Sat",
        "opening_hour": "7:30 PM",
        "closing_hour": "10:00 PM"
      }
    ]
  }
]
```

:::

::::

### Get an entry

按 id 返回一个条目。

:::: tabs

::: tab Request

**示例 request**

```js
GET http://localhost:1337/restaurants/1
```

:::

::: tab Response

**示例 response**

```json
{
  "id": 1,
  "title": "Restaurant 1",
  "cover": {
    "id": 1,
    "name": "image.png",
    "hash": "123456712DHZAUD81UDZQDAZ",
    "sha256": "v",
    "ext": ".png",
    "mime": "image/png",
    "size": 122.95,
    "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
    "provider": "local",
    "provider_metadata": null,
    "created_at": "2019-12-09T00:00:00.000Z",
    "updated_at": "2019-12-09T00:00:00.000Z"
  },
  "content": [
    {
      "__component": "content.title-with-subtitle",
      "id": 1,
      "title": "Restaurant 1 title",
      "subTitle": "Cozy restaurant in the valley"
    },
    {
      "__component": "content.image-with-description",
      "id": 1,
      "image": {
        "id": 1,
        "name": "image.png",
        "hash": "123456712DHZAUD81UDZQDAZ",
        "sha256": "v",
        "ext": ".png",
        "mime": "image/png",
        "size": 122.95,
        "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
        "provider": "local",
        "provider_metadata": null,
        "created_at": "2019-12-09T00:00:00.000Z",
        "updated_at": "2019-12-09T00:00:00.000Z"
      },
      "title": "Amazing photography",
      "description": "This is an amazing photography taken..."
    }
  ],
  "opening_hours": [
    {
      "id": 1,
      "day_interval": "Tue - Sat",
      "opening_hour": "7:30 PM",
      "closing_hour": "10:00 PM"
    }
  ]
}
```

:::

::::

### Count entries

返回与查询筛选器匹配的条目数。您可以在这里阅读更多有关参数的信息 [这里](#api-parameters) 。

:::: tabs

::: tab Request

**示例 request**

```js
GET http://localhost:1337/restaurants/count
```

:::

::: tab Response

**示例 response**

```
1
```

:::

::::

### Create an entry

创建一个条目并返回其值。

:::: tabs

::: tab Request

**示例 request**

```js
POST http://localhost:1337/restaurants
```

```json
{
  "title": "Restaurant 1",
  "cover": 1,
  "content": [
    {
      "__component": "content.title-with-subtitle",
      "title": "Restaurant 1 title",
      "subTitle": "Cozy restaurant in the valley"
    },
    {
      "__component": "content.image-with-description",
      "image": 1, // user form data to upload the file or an id to reference an exisiting image
      "title": "Amazing photography",
      "description": "This is an amazing photography taken..."
    }
  ],
  "opening_hours": [
    {
      "day_interval": "Tue - Sat",
      "opening_hour": "7:30 PM",
      "closing_hour": "10:00 PM"
    }
  ]
}
```

:::

::: tab Response

**示例 response**

```json
{
  "id": 1,
  "title": "restaurant 1",
  "cover": {
    "id": 1,
    "name": "image.png",
    "hash": "123456712DHZAUD81UDZQDAZ",
    "sha256": "v",
    "ext": ".png",
    "mime": "image/png",
    "size": 122.95,
    "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
    "provider": "local",
    "provider_metadata": null,
    "created_at": "2019-12-09T00:00:00.000Z",
    "updated_at": "2019-12-09T00:00:00.000Z"
  },
  "content": [
    {
      "__component": "content.title-with-subtitle",
      "id": 1,
      "title": "Restaurant 1 title",
      "subTitle": "Cozy restaurant in the valley"
    },
    {
      "__component": "content.image-with-description",
      "id": 1,
      "image": {
        "id": 1,
        "name": "image.png",
        "hash": "123456712DHZAUD81UDZQDAZ",
        "sha256": "v",
        "ext": ".png",
        "mime": "image/png",
        "size": 122.95,
        "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
        "provider": "local",
        "provider_metadata": null,
        "created_at": "2019-12-09T00:00:00.000Z",
        "updated_at": "2019-12-09T00:00:00.000Z"
      },
      "title": "Amazing photography",
      "description": "This is an amazing photography taken..."
    }
  ],
  "opening_hours": [
    {
      "id": 1,
      "day_interval": "Tue - Sat",
      "opening_hour": "7:30 PM",
      "closing_hour": "10:00 PM"
    }
  ]
}
```

:::

::::

### Update an entry

根据 id 部分更新条目并返回其值。未在查询中发送的字段不会在数据库中更改。如果要清除 `null` 值，请发送空值。

:::: tabs

::: tab Request

**示例 request**

```js
PUT http://localhost:1337/restaurants/1
```

```json
{
  "title": "Restaurant 1",
  "content": [
    {
      "__component": "content.title-with-subtitle",
      // editing one of the previous item by passing its id
      "id": 2,
      "title": "Restaurant 1 title",
      "subTitle": "Cozy restaurant in the valley"
    },
    {
      "__component": "content.image-with-description",
      "image": 1, // user form data to upload the file or an id to reference an exisiting image
      "title": "Amazing photography",
      "description": "This is an amazing photography taken..."
    }
  ],
  "opening_hours": [
    {
      // adding a new item
      "day_interval": "Sun",
      "opening_hour": "7:30 PM",
      "closing_hour": "10:00 PM"
    },
    {
      // editing one of the previous item by passing its id
      "id": 1,
      "day_interval": "Mon - Sat",
      "opening_hour": "7:30 PM",
      "closing_hour": "10:00 PM"
    }
  ]
}
```

:::

::: tab Response

**示例 response**

```json
{
  "id": 1,
  "title": "Restaurant 1",
  "cover": {
    "id": 1,
    "name": "image.png",
    "hash": "123456712DHZAUD81UDZQDAZ",
    "sha256": "v",
    "ext": ".png",
    "mime": "image/png",
    "size": 122.95,
    "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
    "provider": "local",
    "provider_metadata": null,
    "created_at": "2019-12-09T00:00:00.000Z",
    "updated_at": "2019-12-09T00:00:00.000Z"
  },
  "content": [
    {
      "__component": "content.title-with-subtitle",
      "id": 1,
      "title": "Restaurant 1 title",
      "subTitle": "Cozy restaurant in the valley"
    },
    {
      "__component": "content.image-with-description",
      "id": 2,
      "image": {
        "id": 1,
        "name": "image.png",
        "hash": "123456712DHZAUD81UDZQDAZ",
        "sha256": "v",
        "ext": ".png",
        "mime": "image/png",
        "size": 122.95,
        "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
        "provider": "local",
        "provider_metadata": null,
        "created_at": "2019-12-09T00:00:00.000Z",
        "updated_at": "2019-12-09T00:00:00.000Z"
      },
      "title": "Amazing photography",
      "description": "This is an amazing photography taken..."
    }
  ],
  "opening_hours": [
    {
      "id": 1,
      "day_interval": "Mon - Sat",
      "opening_hour": "7:30 PM",
      "closing_hour": "10:00 PM"
    },
    {
      "id": 2,
      "day_interval": "Sun",
      "opening_hour": "7:30 PM",
      "closing_hour": "10:00 PM"
    }
  ]
}
```

:::

::::

### Delete an entry

按 id 删除条目并返回其值。

:::: tabs

::: tab Request

**示例 request**

```js
DELETE http://localhost:1337/restaurants/1
```

:::

::: tab Response

**示例 response**

```json
{
  "id": 1,
  "title": "Restaurant 1",
  "cover": {
    "id": 1,
    "name": "image.png",
    "hash": "123456712DHZAUD81UDZQDAZ",
    "sha256": "v",
    "ext": ".png",
    "mime": "image/png",
    "size": 122.95,
    "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
    "provider": "local",
    "provider_metadata": null,
    "created_at": "2019-12-09T00:00:00.000Z",
    "updated_at": "2019-12-09T00:00:00.000Z"
  },
  "content": [
    {
      "__component": "content.title-with-subtitle",
      "id": 1,
      "title": "Restaurant 1 title",
      "subTitle": "Cozy restaurant in the valley"
    },
    {
      "__component": "content.image-with-description",
      "id": 2,
      "image": {
        "id": 1,
        "name": "image.png",
        "hash": "123456712DHZAUD81UDZQDAZ",
        "sha256": "v",
        "ext": ".png",
        "mime": "image/png",
        "size": 122.95,
        "url": "http://localhost:1337/uploads/123456712DHZAUD81UDZQDAZ.png",
        "provider": "local",
        "provider_metadata": null,
        "created_at": "2019-12-09T00:00:00.000Z",
        "updated_at": "2019-12-09T00:00:00.000Z"
      },
      "title": "Amazing photography",
      "description": "This is an amazing photography taken..."
    }
  ],
  "opening_hours": [
    {
      "id": 1,
      "day_interval": "Mon - Sat",
      "opening_hour": "7:30 PM",
      "closing_hour": "10:00 PM"
    },
    {
      "id": 2,
      "day_interval": "Sun",
      "opening_hour": "7:30 PM",
      "closing_hour": "10:00 PM"
    }
  ]
}
```

:::

::::

## API 参数

<!-- See the [parameter concepts](../concepts/parameters.md) for details. -->
<!-- The links in comments lines 810 and 818 are broken, so this file was to the checklinksIgnoredFiles in config.js to prevent false positives when checking for broken links-->

::: warning
默认情况下，过滤器只能从 Content Type Builder 和 [CLI](../cli/CLI.md) 生成的 `find` 和 `count` 终结点中使用。
:::

<!-- This sentance belongs in the above warning block but was removed and commented until new docs can be written -->
<!-- If you need to implement a filter system somewhere else, read the [programmatic usage](../concepts/parameters.md) section. -->

### 可供使用的营办商

现有的操作符分为五个不同类别:

- [Filters](#filters)
- [Sort](#sort)
- [Limit](#limit)
- [Start](#start)
- [Publication State](#publication-state)

### 过滤器

当使用过滤器时，您可以在查询参数的根中传递简单的过滤器，或者在 `_where` 参数中传递它们。

过滤器用作字段名的后缀:

| Filter            | Description                     |
| :---------------- | :------------------------------ |
| No suffix or `eq` | Equal                           |
| `ne`              | Not equal                       |
| `lt`              | Less than                       |
| `gt`              | Greater than                    |
| `lte`             | Less than or equal to           |
| `gte`             | Greater than or equal to        |
| `in`              | Included in an array            |
| `nin`             | Not included in an array        |
| `contains`        | Contains                        |
| `ncontains`       | Doesn't contain                 |
| `containss`       | Contains, case sensitive        |
| `ncontainss`      | Doesn't contain, case sensitive |
| `null`            | Is null or not null             |

#### 例子

##### Find users having `John` as first name.

`GET /users?firstName=John`
or
`GET /users?firstName_eq=John`

##### Find restaurants having a price equal or greater than `3`.

`GET /restaurants?price_gte=3`

##### Find multiple restaurant with id 3, 6, 8.

`GET /restaurants?id_in=3&id_in=6&id_in=8`

##### Using `_where`

`GET /restaurants?_where[price_gte]=3`

`GET /restaurants?_where[0][price_gte]=3&[0][price_lte]=7`

### 复杂查询

::: tip 注意
从 v3.1.0 开始可以使用 `OR` 和 `AND` 操作
:::

在构建更复杂的查询时，必须结合使用 `_where` 查询参数和 [`qs`](https://github.com/ljharb/qs) 库。

我们正在利用 `qs` 解析嵌套对象的能力来创建更复杂的查询。

这将使您完全有能力创建具有逻辑 `AND` 和 `OR` 操作的复杂查询。

::: tip 注意
我们强烈建议直接使用 `qs` 来生成复杂的查询，而不是手动创建它们。
:::

#### `AND` 操作

当在筛选中指定表达式数组时，筛选隐式支持 `AND` 操作。

**例子**

1 星级以下、价格低于或等于 20 星级的餐厅:

```js
const query = qs.stringify({
  _where: [{ stars: 1 }, { pricing_lte: 20 }],
});

await request(`/restaurants?${query}`);
// GET /restaurants?_where[0][stars]=1&_where[1][pricing_lte]=20
```

定价大于或等于 20 而定价小于或等于 50 的餐厅:

```js
const query = qs.stringify({
  _where: [{ pricing_gte: 20 }, { pricing_lte: 50 }],
});

await request(`/restaurants?${query}`);
// GET /restaurants?_where[0][pricing_gte]=20&_where[1][pricing_lte]=50
```

#### `OR` operator

若要使用 `OR` 操作，您将需要使用 `_or` 筛选器并指定一个表达式数组来执行该操作。

**例子**

1 星级或价格大于 30 星级的餐厅:

```js
const query = qs.stringify({ _where: { _or: [{ stars: 1 }, { pricing_gt: 30 }] } });

await request(`/restaurant?${query}`);
// GET /restaurants?_where[_or][0][stars]=1&_where[_or][1][pricing_gt]=30
```

定价低于 10 美元或高于 30 美元的餐厅:

```js
const query = qs.stringify({ _where: { _or: [{ pricing_lt: 10 }, { pricing_gt: 30 }] } });

await request(`/restaurant?${query}`);
// GET /restaurants?_where[_or][0][pricing_lt]=10&_where[_or][1][pricing_gt]=30
```

#### 隐式 `OR` 操作

当在表达式中传递值数组时，查询引擎隐式使用 `OR` 操作。

**例子**

拥有一星或两星的餐厅:

`GET /restaurants?stars=1&stars=2`

or

```js
const query = qs.stringify({ _where: { stars: [1, 2] } });

await request(`/restaurant?${query}`);
// GET /restaurants?_where[stars][0]=1&_where[stars][1]=2
```

::: tip 注意
当使用 `in` 和 `nin` 滤波器时，数组不会转换为 OR。
:::

#### Combining AND and OR operators

(2 颗星，定价低于 80)或(1 颗星，定价高于或等于 50)的餐厅

```js
const query = qs.stringify({
  _where: {
    _or: [
      [{ stars: 2 }, { pricing_lt: 80 }], // implicit AND
      [{ stars: 1 }, { pricing_gte: 50 }], // implicit AND
    ],
  },
});

await request(`/restaurants?${query}`);
// GET /restaurants?_where[_or][0][0][stars]=2&_where[_or][0][1][pricing_lt]=80&_where[_or][1][0][stars]=1&_where[_or][1][1][pricing_gte]=50
```

**这也适用于深度过滤**

有(2 颗星，定价低于 80)或(1 颗星，供应法国食品)的餐厅

```js
const query = qs.stringify({
  _where: {
    _or: [
      [{ stars: 2 }, { pricing_lt: 80 }], // implicit AND
      [{ stars: 1 }, { 'categories.name': 'French' }], // implicit AND
    ],
  },
});

await request(`/restaurants?${query}`);
// GET /restaurants?_where[_or][0][0][stars]=2&_where[_or][0][1][pricing_lt]=80&_where[_or][1][0][stars]=1&_where[_or][1][1][categories.name]=French
```

::: warning
在创建嵌套查询时，请确保深度小于 20，否则查询字符串解析将暂时失败。
:::

### 深度过滤

查找属于星级等于 5 `GET /restaurants?chef.restaurant.star=5` 的餐厅的厨师所拥有的餐厅

::: warning
使用深度过滤器查询 API 可能会导致性能问题。如果某个深度过滤查询速度太慢，我们建议使用优化版本的查询构建自定义路由。
:::

::: tip

这个功能不允许你过滤嵌套模型，例如查找用户，只返回他们昨天以前发布的信息。

要做到这一点，有三种选择:

- 建立一个自定义路由.
- 修改你的服务.
- 使用 [GraphQL](/developer-docs/latest/development/plugins/graphql.md#query-api).
  :::

::: warning
这个特性不适用于多态关系。这种关系类型用于 `media`、`component` 和 `dynamic zone` 字段。
:::

### 排序

根据特定字段进行排序。

#### 例子

##### Sort users by email.

- ASC: `GET /users?_sort=email:ASC`
- DESC: `GET /users?_sort=email:DESC`

##### Sorting on multiple fields

- `GET /users?_sort=email:asc,dateField:desc`
- `GET /users?_sort=email:DESC,username:ASC`

### 限制

限制返回结果的大小。 默认值 `100`

#### 例子

##### 将结果长度限制为 30.

`GET /users?_limit=30`

您可以通过传递一个等于 `-1` 的限制来要求完整的数据集。

### 开始

跳过特定数量的条目(特别适用于分页)。

#### 例子

##### 获取结果的第二页.

`GET /users?_start=10&_limit=10`

### 出版状态

::: tip 注意
此参数只能在启动了 **Draft & Publish** 特性的模型上使用
:::

只选择与所提供的发布状态匹配的项。

处理状态如下:

- `live`: 只返回已发布的条目 (default)
- `preview`: 同时返回草稿及已发表的作品

#### 例子

##### 获得已发表的文章

`GET /articles`
OR
`GET /articles?_publicationState=live`

##### 同时获得发表和起草的文章

`GET /articles?_publicationState=preview`

::: tip 注意
如果只想检索草稿条目，可以将 `preview` 模式和 `published_at` 字段组合在一起。`GET /articles?_publicationState=preview&published_at_null=true`
:::
