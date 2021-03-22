---
sidebarDepth: 2
---

# 过程管理器

在本指南中，我们将看到如何使用流程管理器启动 Strapi 应用程序。

::: tip
在这个例子中，我们将使用 [PM2](https://pm2.keymetrics.io/)。
:::

## 安装 PM2

`PM2` 允许您保持您的 Strapi 项目活着，并重新加载它没有停机。

你将在全球安装 PM2

:::: tabs

::: tab yarn
`yarn global add pm2`
:::

::: tab npm
`npm install pm2 -g`
:::

::::

## 基本用法

### 创建/编辑 server.js

使用 PM2 启动应用程序的基本用法是运行一个命令，如下面的 `pm2 start server.js`。

但我们现在面临一个问题。在你的项目中，你没有一个 `.js` 文件来运行 Strapi 应用程序。

因此，首先让我们创建一个 `server.js` 文件，它可以让您运行 `pm2` 命令。

**Path —** `./server.js`

```js
const strapi = require('strapi');
strapi().start();
```

现在你可以通过运行 `pm2 start server.js` 来启动服务器了。

### 从 strapi 命令开始

默认情况下有两个重要的命令。

- `yarn develop` 以开发模式启动项目.
- `yarn start` 开始制作你的应用程序.

您还可以使用 `yarn start` 命令启动流程管理器。

`pm2 start npm --name app -- run start`

## 配置文件

`PM2` 允许您创建一个配置文件来保存所有信息，以便在任何时候正确启动服务器。

通过运行 `pm2 init`，它将在你的应用程序中嵌入一个 `ecosystem.config.js`。

然后用下面的代码替换该文件的内容。

```js
module.exports = {
  apps: [
    {
      name: 'app',
      script: 'npm',
      args: 'start',
    },
  ],
};
```

然后运行 `pm2 start ecosystem.config.js` 启动 pm2 进程。

您可以在 [PM2 生态系统文件文档](https://pm2.keymetrics.io/docs/usage/application-declaration/) 中看到可用配置的完整文档。
