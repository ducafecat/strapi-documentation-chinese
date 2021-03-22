# 安装第三方客户端

本指南将解释如何与第三方客户机建立连接，并在代码中随处使用它。

在我们的示例中，我们将使用 GitHub Node.JS 客户端 [OctoKit REST.js](https://github.com/octokit/rest.js/)。

本指南还可用于设置 `Axios` 客户端实例。

## 安装

首先，你必须在你的应用程序中运行以下命令之一来安装客户端软件包:

:::: tabs

::: tab yarn
`yarn add @octokit/rest`
:::

::: tab npm
`npm install @octokit/rest`
:::

::::

## 创建一个钩

为了初始化客户端，我们将使用 [hooks system](/developer-docs/latest/setup-deployment-guides/configurations.md#hooks)。

钩子在服务器启动时加载一次。

让我们创建我们的 GitHub 钩子。

**Path —** `./hooks/github/index.js`

```js
module.exports = strapi => {
  return {
    async initialize() {
      console.log('my hook is loaded');
    },
  };
};
```

钩子创建后，我们将其设置为 `enabled`，以便 Strapi 加载它。您将需要创建或编辑文件 `./config/hook.js`.

**Path —** `./config/hook.js`

```js
module.exports = {
  settings: {
    github: {
      enabled: true,
    },
  },
};
```

现在你可以启动你的应用程序了，你应该看到一个日志 `my hook is loaded` 在你的终端。

## 初始化客户端

首先让我们更新配置文件来添加您的 [GitHub token](https://github.com/settings/tokens)。

通过跟随 [文章](https://octokit.github.io/rest.js/#authentication) ，您还将找到使用 GitHub 应用程序的方法。

**Path —** `./config/hook.js`

```js
module.exports = {
  settings: {
    github: {
      enabled: true,
      token: process.env.GITHUB_TOKEN,
    },
  },
};
```

**Path -** `.env`

```
GITHUB_TOKEN=bf78d4fc3c1767019870476d6d7cc8961383d80f
```

现在我们必须加载 GitHub 客户机。

**Path —** `./hooks/github/index.js`

```js
const GitHubAPI = require('@octokit/rest');

module.exports = strapi => {
  return {
    async initialize() {
      const { token } = strapi.config.get('hook.settings.github');

      strapi.services.github = new GitHubAPI({
        userAgent: `${strapi.config.get('info.name')} v${strapi.config.get('info.version')}`,
        auth: `token ${token}`,
      });
    },
  };
};
```

这就是了。

现在可以在代码中随处使用 `strapi.services.github` 来使用 GitHub 客户机。

为了简单地测试它是否有效，让我们更新一下 `bootstrap.js` 函数来记录您的 GitHub 配置文件。

**Path —** `./config/functions/bootstrap.js`

```js
module.exports = async () => {
  const data = await strapi.services.github.users.getAuthenticated();
  console.log(data);
};
```

重启你的服务器，你会看到你的 GitHub 档案数据。
