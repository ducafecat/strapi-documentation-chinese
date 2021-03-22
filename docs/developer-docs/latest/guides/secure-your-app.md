# 确保你的应用程序安全

在本指南中，我们将看到如何通过使用第三方提供商来保护您的 Strapi 应用程序。

::: tip
在这个例子中，我们将使用 [Sqreen](https://sqreen.com)。
:::

他们的 [onboarding](https://my.sqreen.com/new-application#nodejs-agent) 真的很容易跟踪和理解。

## 安装 Sqreen

Sqreen 是一个应用程序安全管理工具，它可以根据您的堆栈定制保护，从而使您的安全性具有前所未有的可见性，并且能够在生产环境中扩展它。

你必须在你的应用程序中安装 `Sqreen` nodejs 模块。

:::: tabs

::: tab yarn
`yarn add sqreen`
:::

::: tab npm
`npm install sqreen`
:::

::::

## 以编程方式启动应用程序

我们必须在用于启动 Strapi 的文件中 `require Sqreen` node_module。

为此，您必须创建一个 `server.js` 文件，以便通过运行 `node server.js` 启动应用程序。

**Path —** `./server.js`

```js
const strapi = require('strapi');
strapi().start();
```

现在您可以运行 `node server.js`，它将启动您的应用程序。

## 注入并配置 Sqreen 代理

通过遵循他们的 Node.js 入职，我们需要服务器启动的 Sqreen 节点模块。而且，Sqreen 必须在斯特拉皮工作之前被要求！

这就是我们创建 `server.js` 文件的原因。

为此，您必须更新此文件。

**Path —** `./server.js`

```js
require('sqreen');
const strapi = require('strapi');
strapi().start();
```

要让 Strapi 和 Sqreen 同步，您必须使用凭证创建一个 `./sqreen.json` 文件。

然后用 `node server.js` 启动服务器，我们就完成了。
