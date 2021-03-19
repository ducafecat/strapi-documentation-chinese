# 管理面板定制

## 管理扩展

管理面板是一个类似于插件的 `node_module` ，只是它封装了应用程序所有已安装的 [插件](/developer-docs/latest/development/plugin-customization.md) 。

要扩展此包，您必须在应用程序的根目录创建一个 `admin` 文件夹。

在这个文件夹中，您可以覆盖管理文件和函数。

## 定制选项

管理面板可以根据您的需要定制，因此您可以使它反映您的身份。

::: warning
要应用更改，需要 [重新构建](#build) 管理面板
:::

### 更改访问网址

默认情况下，管理面板通过 [http://localhost:1337/admin](http://localhost:1337/admin) 。但是，出于安全原因，您可以轻松地更新此路径。有关更高级的设置，请参阅 [服务器配置](/developer-docs/latest/setup-deployment-guides/configurations.md#server) 文档。

**Path —** `./config/server.js`.

```js
module.exports = ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
  port: env.int('PORT', 1337),
  admin: {
    url: '/dashboard',
  },
});
```

这个面板可以通过上面的配置通过 [http://localhost:1337/dashboard](http://localhost:1337/dashboard) 打开新的窗口。

### 开发模式

要启用前端开发模式，您需要使用 `--watch-admin` 标志启动应用程序。

```bash
cd my-app
strapi develop --watch-admin
```

使用这个选项，你可以做以下操作:

#### 自定义 `strapi-admin` 软件包

在 `my-app/admin/src/` 中添加的所有文件都将被替换或添加

**示例: 更改应用程序的可用区域设置**

```bash
# Create both the admin and admin/src/translations folders
cd my-app && mkdir -p admin/src/translations
# Change the available locales of the administration panel
touch admin/src/i18n.js
# Change the import and exports of the translations files
touch admin/src/translations/index.js
```

**Path -** `my-app/admin/src/translations/index.js`

```js
import en from './en.json';
import fr from './fr.json';

const trads = {
  en,
  fr,
};

export const languageNativeNames = {
  en: 'English',
  fr: 'Français',
};

export default trads;
```

::: tip
通过这个修改，只有英语和法语将可用于您的管理
:::

#### 定制插件

与后端覆盖系统类似，在 `my-app/extensions/<plugin-name>/admin/` 中添加的任何文件都将被复制并使用，而不是原始文件(请小心使用)。

**Example: 更改内容编辑器 WYSIWYG**

```bash
cd my-app/extensions
# Create the content manager folder
mkdir content-manager && cd content-manager
# Create the admin folder
mkdir -p admin/src
# Create the components folder and the WysiwygWithErrors one
cd admin/src && mkdir -p components/WysiwygWithErrors
# Create the index.js so the original file is overridden
touch components/WysiwygWithErrors/index.js
```

**Path -** `my-app/extensions/content-manager/admin/src/components/WysiwygWithErrors/index.js`

```js
import React from 'react';
import MyNewWYSIWYG from 'my-awesome-lib';

// This is a dummy example
const WysiwygWithErrors = props => <MyNewWYSIWYG {...props} />;

export default WysiwygWithErrors;
```

#### Styles 样式

可以在 `./node_modules/strapi-admin/src/` 中轻松找到 AdminUI 包源。

例如，要更改左上角显示的管理面板的颜色，请复制 `./node_modules/strapi-admin/admin/src/components/LeftMenu/LeftMenuHeader` 文件夹 `./admin/src/components/LeftMenu/LeftMenuHeader` (如果这些文件夹不存在，则创建它们)并更改其中的样式 `./admin/src/components/LeftMenu/LeftMenuHeader/Wrapper.js` .

因此，您正在替换通常位于 `node_modules/strapi-admin/admin/src` 中的文件，并将它们指向 `admin/src/some/file/path` 。

要应用更改，需要重新构建管理面板

```
npm run build
```

#### Logo 标志

要更改左上角显示的管理面板的徽标，请添加您的自定义图像到 `./admin/src/assets/images/logo-strapi.png`。

要更改登录页面的徽标，请在 `./admin/src/assets/images/logo_strapi.png` 添加自定义图像。

::: tip
确保你的图像和现有的图像大小一样(434px x 120px)。
:::

#### 禁用教学视频

要禁用包含教程视频的信息框，请在 `./admin/src/config.js` 创建一个文件

添加以下配置:

```js
export const LOGIN_LOGO = null;
export const SHOW_TUTORIALS = false;
export const SETTINGS_BASE_URL = '/settings';
export const STRAPI_UPDATE_NOTIF = true;
```

#### 更改主机和端口

默认情况下，前端开发服务器运行在 `localhost:8000` 上。但是，您可以通过更新以下配置来更改此设置:

**Path —** `./config/server.js`.

```js
module.exports = ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
  port: env.int('PORT', 1337),
  admin: {
    host: 'my-host', // only used along with `strapi develop --watch-admin` command
    port: 3000, // only used along with `strapi develop --watch-admin` command
  },
});
```

### Build 构建

若要生成管理，请从项目的根目录运行以下命令。

:::: tabs

::: tab yarn

```
yarn build
```

:::

::: tab npm

```
npm run build
```

:::

::: tab strapi

```
strapi build
```

:::

::::

这将替换位于 `./build` 。

访问 [http://localhost:1337/admin](http://localhost:1337/admin)，确保你的更新已经被考虑在内。

## 自定义 Webpack 配置

为了扩展 webpack 的使用，可以在 `admin/admin.config.js` 中定义一个扩展配置的函数，如下所示:

```js
module.exports = {
  webpack: (config, webpack) => {
    // Note: we provide webpack above so you should not `require` it
    // Perform customizations to webpack config
    // Important: return the modified config
    config.plugins.push(new webpack.IgnorePlugin(/\/__tests__\//));

    return config;
  },
};
```

## Deployment 部署

管理只不过是一个调用 API 的 React 前端应用程序。前端和后端是独立的，可以部署在不同的服务器上，这就给我们带来了不同的场景:

1. 将整个项目部署在同一台服务器上.
2. 将管理面板部署到 API 之外的另一台服务器(AWS S3、 Azure、 etc)上

让我们深入研究每种情况下的构建配置。

### 将整个项目部署在同一台服务器上.

您不需要触摸配置文件中的任何内容。这是默认行为，生成配置将自动设置。服务器将在指定的端口启动，管理面板将通过 `http://yourdomain.com:1337/dashboard` 访问。

您可能希望更改访问管理面板的路径。下面是修改路径所需的配置:

**Path —** `./config/server.js`.

```js
module.exports = ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
  port: env.int('PORT', 1337),
  admin: {
    url: '/dashboard', // We change the path to access to the admin (highly recommended for security reasons).
  },
});
```

**您必须重新构建管理面板才能使其工作。** [Build instructions](/developer-docs/latest/development/admin-customization.md#build).

### 将管理面板部署到 API 之外的另一台服务器 (AWS S3, Azure, etc).

在不同的服务器上部署前端和后端是非常常见的。以下是处理这种情况所需的配置:

**Path —** `./config/server.js`.

```js
module.exports = ({ env }) => ({
  host: env('HOST', '0.0.0.0'),
  port: env.int('PORT', 1337),
  url: 'http://yourbackend.com',
  admin: {
    url: '/', // Note: The administration will be accessible from the root of the domain (ex: http://yourfrontend.com/)
    serveAdminPanel: false, // http://yourbackend.com will not serve any static admin files
  },
});
```

使用此配置运行 `yarn build` 之后，将创建/覆盖文件夹 `build` 。然后，您可以使用这个文件夹从另一个服务器上提供您选择的域名(例如: `http://yourfrontend.com`)。

然后管理 URL 将是 `http://yourfrontend.com` ，来自面板的每个请求将在 `http://yourfrontend.com` 时间点击后端。

::: tip 注意
如果你给 `url` 添加一个路径选项，它就不会给你的应用加上前缀。为此，您还需要使用 Nginx 这样的代理服务器。更多内容请点击[这里](/developer-docs/latest/setup-deployment-guides/deployment.md#optional-software-guides)。
:::

## 忘记密码电子邮件

### 自定义忘记密码的电子邮件

您可能需要自定义忘记密码的电子邮件。您可以通过提供自己的模板(格式为 [lodash template](https://lodash.com/docs/4.17.15#template) 来实现这一点 )。

该模板将使用以下变量进行编译: `url`, `user.email`, `user.username`, `user.firstname`, `user.lastname` 。

### 例子

**Path -** `./config/servers.js`

```js
const forgotPasswordTemplate = require('./email-templates/forgot-password');

module.exports = ({ env }) => ({
  // ...
  admin: {
    // ...
    forgotPassword: {
      from: 'support@mywebsite.fr',
      replyTo: 'support@mywebsite.fr',
      emailTemplate: forgotPasswordTemplate,
    },
    // ...
  },
  // ...
});
```

**Path -** `./config/email-templates/forgot-password.js`

```js
const subject = `Reset password`;

const html = `<p>Hi <%= user.firstname %></p>
<p>Sorry you lost your password. You can click here to reset it: <%= url %></p>`;

const text = `Hi <%= user.firstname %>
Sorry you lost your password. You can click here to reset it: <%= url %>`;

module.exports = {
  subject,
  text,
  html,
};
```
