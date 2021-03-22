# 自定义管理

在本指南中，我们将看到如何自定义管理面板。

## 引言

在这个例子中，我们将看到两件事:

1. 管理面板本身的定制，通过更新 `/admin/` 主页;
2. 如何更新插件的接口，通过替换 `date` 内容管理器列表视图中的格式.

首先，您将必须阅读 [管理面板定制](/developer-docs/latest/development/admin-customization.md) ，它将帮助您了解如何定制所有的应用程序。

## 更新管理员主页

如果遵循自定义概念，则可以在应用程序中创建 `./admin` 文件夹。

### 指定要更新的文件

然后，您必须调查 [`strapi-admin`](https://github.com/strapi/strapi/tree/master/packages/strapi-admin) 包，以找到用于显示管理面板主页的文件。

这里是[主页容器](https://github.com/strapi/strapi/tree/master/packages/strapi-admin/admin/src/containers/HomePage/index.js) ，您必须更新。

### 退出文件

让我们弹出这个文件，以便能够定制它。

**Path —** `./admin/src/containers/HomePage/index.js`

在这个新文件中，粘贴当前的 [HomePage container](https://github.com/strapi/strapi/tree/master/packages/strapi-admin/admin/src/containers/HomePage/index.js) 代码。

要运行应用程序，请运行以下命令:

:::: tabs

::: tab yarn

```bash
yarn develop --watch-admin
```

:::

::: tab npm

```bash
npm run develop -- --watch-admin
```

:::

::::

如果你访问管理员，主页上什么都不会改变，这很正常！

### 自定义文件

为了让这个例子保持简单，我们将把 HomePage 简化成一个更简单的设计。

**Path —** `./admin/src/containers/HomePage/index.js`

```js
import React, { memo } from 'react';

import { Block, Container } from './components';

const HomePage = ({ global: { plugins }, history: { push } }) => {
  return (
    <>
      <Container className="container-fluid">
        <div className="row">
          <div className="col-12">
            <Block>Hello World!</Block>
          </div>
        </div>
      </Container>
    </>
  );
};

export default memo(HomePage);
```

现在管理面板主页应该只包含句子 `Hello World`。

## 更新内容管理器

如果遵循自定义概念，则可以在应用程序中创建 `./extensions/content-manager` 文件夹。

::: tip
为了能够查看更新，您需要具有具有 `date` 属性的 Content Type。
:::

### 指定要更新的文件

然后，您必须调查 [`strapi-plugin-content-manager`](https://github.com/strapi/strapi/tree/master/packages/strapi-plugin-content-manager) 包，以找到用于格式化列表视图日期的文件。

下面是 [Row component](https://github.com/strapi/strapi/blob/master/packages/strapi-plugin-content-manager/admin/src/components/CustomTable/Row.js) ，它需要一个 [dedicated file](https://github.com/strapi/strapi/blob/master/packages/strapi-plugin-content-manager/admin/src/utils/dateFormats.js) 来修改日期显示。

### 退出文件

让我们弹出文件，以便能够定制它。

**Path —** `./extensions/content-manager/admin/src/utils/dateFormats.js`

在这个新文件中，粘贴当前的 [dateFormats](https://github.com/strapi/strapi/blob/master/packages/strapi-plugin-content-manager/admin/src/utils/dateFormats.js) 代码。

要运行应用程序，请运行以下命令:

:::: tabs

::: tab yarn

```bash
yarn develop --watch-admin
```

:::

::: tab npm

```bash
npm run develop -- --watch-admin
```

:::

::::

如果您访问您的内容类型的条目列表视图，什么都不会改变。这是正常的！

### Customize the file

在我们的示例中，我们希望更改日期的格式。我们必须在这个文件中找到管理日期格式的行。

下面是你需要找到的代码:

```js
const dateFormats = {
  ...defaultDateFormats,
  // Customize the format by un-commenting the one you wan to override it corresponds to the type of your field
  // date: 'dddd, MMMM Do YYYY',
  // datetime: 'dddd, MMMM Do YYYY HH:mm',
  // time: 'HH:mm A',
  // timestamp: 'dddd, MMMM Do YYYY HH:mm',
};
```

现在让我们替换日期: `date: 'dddd, MMMM Do YYYY'` by `date: 'YYYY/MM/DD';`;

而且，日期现在将以新的格式显示。

## 建立新的管理员

现在你有了你想要的管理面板。但是在整个过程中，管理面板在运行时更新，因为命令 `yarn develop --watch-admin` 或 `npm run develop -- --watch-admin`。

如果你开始你的应用程序使用 `yarn start` 或 `yarn develop` 管理将是老版本。未应用更新。

为此，您必须使用下面的命令 `yarn build` 管理面板。
