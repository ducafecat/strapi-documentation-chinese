# 认证请求

在本指南中，您将看到如何作为经过身份验证的用户请求 API。

## 引言

为了向您展示关于 [角色和权限](/developer-docs/latest/development/plugins/users-permissions.md) 部分的许多概念，我们将使用许多用户和角色。

在此之后，我们将看到获取 JWT 的 [身份验证工作流](/developer-docs/latest/development/plugins/users-permissions.md#authentication) ，并将其用于 API 请求。

我们将有一组用户，将能够只提取 **Articles** 和其他组，将能够提取，创建和更新 **Articles**。

## 设置

### 创建内容类型

让我们创建一个新的内容类型 **Article**

- 点击 `Content Type Builder` 在左边的菜单里
- 然后 `+ Create new content-type`
- 填充 `Display name` with `article`
- 创建 2 个字段
  - **Text** short named `title`
  - **Rich text** named `content`
- And save this new Content Type

然后从内容管理器创建一些 **Articles** 。

### 创建角色和权限

我们将创建两个新的组来管理不同类型用户的可用操作。

- 点击 `Settings` 在左边的菜单里
- 选择 `Roles` 下面 `USERS & PERMISSIONS PLUGIN` 部分
- 然后 `+ Add New Role`
- 填充 `name` 与 `Author`
- 点击 `Select All` 选中文章类型.
- 点击保存这个新角色

然后对 `Reader` 组重复操作， `find`、 `findOne` 和 `count`。

### 创建用户

最后使用以下数据创建 **2 个用户** 。

**User 1**

- **username**: author
- **email**: author@strapi.io
- **password**: strapi
- **role**: Author

**User 2**

- **username**: reader
- **email**: reader@strapi.io
- **password**: strapi
- **role**: Reader

## 作为一个读者登录

要以用户身份登录，您必须遵循 [登录文档](/developer-docs/latest/development/plugins/users-permissions.md#login) 。

这是是身份验证 `/auth/local` 的 API 路由。

你必须在 **POST** 中请求它。

:::: tabs

::: tab axios

```js
import axios from 'axios';

const { data } = await axios.post('http://localhost:1337/auth/local', {
  identifier: 'reader@strapi.io',
  password: 'strapi',
});

console.log(data);
```

:::

::: tab Postman

例如，如果使用 Postman，则必须将 `body` 设置为 `JSON (application/JSON)`类型的 `raw`。

```json
{
  "identifier": "reader@strapi.io",
  "password": "strapi"
}
```

:::

::::

API 响应在 `JWT` 键中包含 `用户的JWT`

```json
{
    "jwt": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiaWF0IjoxNTc2OTM4MTUwLCJleHAiOjE1Nzk1MzAxNTB9.UgsjjXkAZ-anD257BF7y1hbjuY3ogNceKfTAQtzDEsU",
    "user": {
        "id": 1,
        "username": "reader",
        ...
    }
}
```

您必须将这个 `JWT` 存储在您的应用程序中，这很重要，因为您必须在接下来的请求中使用它。

### Fetch articles

让我们获取您创建的文章。

要做到这一点，您必须在 **GET** 中执行请求 `/articles` 路由。

```js
import axios from 'axios';

const { data } = await axios.get('http://localhost:1337/articles');

console.log(data);
```

这里您应该会收到一个 **403 错误**，因为您作为 Public 用户不允许访问这些**文章**。

您应该在请求中使用 `JWT` 来说明您可以作为 `Reader 用户` 访问这些数据。

```js
import axios from 'axios';

const { data } = await axios.get('http://localhost:1337/articles', {
  headers: {
    Authorization:
      'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiaWF0IjoxNTc2OTM4MTUwLCJleHAiOjE1Nzk1MzAxNTB9.UgsjjXkAZ-anD257BF7y1hbjuY3ogNceKfTAQtzDEsU',
  },
});

console.log(data);
```

而且你可以访问这些数据。

### 创建一篇文章

为此，您必须在 **POST** 中请求 `/articles` 路由

```js
import axios from 'axios';

const {data} = await axios
  .post('http://localhost:1337/articles',  {
    data: {
      title: 'my article'
      content: 'my super article content'
    },
    headers: {
      'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiaWF0IjoxNTc2OTM4MTUwLCJleHAiOjE1Nzk1MzAxNTB9.UgsjjXkAZ-anD257BF7y1hbjuY3ogNceKfTAQtzDEsU'
    }
  });

console.log(data);
```

如果您以 **Reader 用户** 的身份请求此操作，您将收到一个 **403 错误**。这是因为 **Reader 角色**无法访问 **Article** Content Type 的 **create** 函数。

要解决这个问题，必须使用 **Author 用户**登录，并在请求中使用其 `JWT` 创建一篇文章。

完成这些之后，您将能够创建一篇**文章**。
