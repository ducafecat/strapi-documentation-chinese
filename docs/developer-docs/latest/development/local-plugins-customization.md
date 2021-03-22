# 本地插件

## 快速启动

Strapi 允许你创建与外部插件工作方式完全相同的本地插件。所有的本地插件都将位于 `./plugins` 文件夹。

### 开发环境设置

创建一个开发项目

1. 创建一个新项目 `cd .. && strapi new myDevelopmentProject`.
2. `cd myDevelopmentProject && strapi develop` 启动 Strapi 项目

### 插件开发设置

在新的终端窗口中:

生成一个新的插件: `cd /path/to/myDevelopmentProject && strapi generate:plugin my-plugin`

::: tip 注意
在你成功生成了一个插件之后，你需要运行 `strapi build` ，将新的插件添加到管理面板中。
:::

## 插件文件夹和文件架构

插件的逻辑位于其根目录 `./plugins/**` .每个插件的管理面板相关部分都包含在 `/admin` 文件夹中。

文件夹和文件结构如下:

```
plugin/
└─── admin/ # Contains the plugin's front-end
|     └─── src/ # Source code directory
|          └─── index.js # Entry point of the plugin
|          └─── pluginId.js # Name of the plugin
|          |
|          └─── components/ # Contains the list of React components used by the plugin
|          └─── containers/
|          |    └─── App/ # Container used by every others containers
|          |    └─── Initializer/ # This container is required, it is used to executed logic right after the plugin is mounted.
|          └─── translations/ # Contains the translations to make the plugin internationalized
|               └─── en.json
|               └─── index.js # File that exports all the plugin's translations.
|               └─── fr.json
└─── config/ # Contains the configurations of the plugin
|     └─── functions/
|     |    └─── bootstrap.js # Asynchronous bootstrap function that runs before the app gets started
|     └─── policies/ # Folder containing the plugin's policies
|     └─── queries/ # Folder containing the plugin's models queries
|     └─── routes.json # Contains the plugin's API routes
└─── controllers/ # Contains the plugin's API controllers
└─── middlewares/ # Contains the plugin's middlewares
└─── models/ # Contains the plugin's API models
└─── services/ # Contains the plugin's API services
```

<!--- DEVELOPMENT OF LOCAL PLUGIN --->

## 后端开发

本节将解释插件的 “后端部分” 是如何工作的。

### Routes

插件 API 路由在 `./plugins/**/config/routes.json` 文件中定义。

::: tip
请参考 [路由器文档](/developer-docs/latest/development/backend-customization.md#routing) 中的信息。
:::

**路由前缀**

插件的每个路由都以插件的名称作为前缀(例如: `/my-plugin/my-plugin-route`)。使用 `prefix` 可以将此选项更改为自定义选项。您可以通过将 `config.prefix` 键设置为空字符串来禁用前缀。

```json
{
  "method": "GET",
  "path": "/my-plugin-route",
  "handler": "MyPlugin.action",
  "config": {
    "policies": [],
    "prefix": "my-custom-prefix"
  }
}
```

### CLI 命令行

CLI 可用于在插件文件夹中生成文件。

有关更多信息，请参考 [CLI 文档](/developer-docs/latest/developer-resources/cli/CLI.md) 。

### Controllers 控制器

控制器包含根据请求的路由执行的功能。

有关更多信息，请参阅 [控制器文档](/developer-docs/latest/development/backend-customization.md#controllers)。

### Models 模型

插件可以有自己的模型。

#### Table/Collection 表格/集合命名

有时候插件注入的模型和你的名字一样。让我们举个简单的例子。

你已经有 `User` 模型在你的 `./api/user/models/User.settings.json` API.你决定安装 `Users & Permissions` 插件。这个插件还包含一个 `User` 模型。为了避免冲突，插件的模型不是全局公开的，这意味着你不能像这样访问插件的模型:

```js
module.exports = {
  findUser: async function(params) {
    // This `User` global variable will always make a reference the User model defining in your `./api/xxx/models/User.settings.json`.
    return await User.find();
  },
};
```

此外，表/集合名称不是 `users` ，因为您已经有了一个 `users` 模型。这就是为什么框架会自动将插件的名称作为这个模型的表/集合名称的前缀。这意味着在我们的示例中，我们的插件 `Users & Permissions` 的用户模型的表/集合名称将是 `users-permissions_users`。如果希望强制插件模型的表/集合名称，可以在模型中添加 `collectionName` 属性。

有关更多信息，请参阅 [模型文档](/developer-docs/latest/development/backend-customization.md#models) 。

### Policies 策略

#### 全局策略

在当前的 Strapi 项目中，插件也可以使用全球公开的策略。

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/",
      "handler": "MyPlugin.index",
      "config": {
        "policies": ["global::isAuthenticated"]
      }
    }
  ]
}
```

#### 插件策略

插件可以有自己的策略，比如添加安全规则。例如，如果插件包含一个名为 `isAuthenticated` 的策略，那么使用这个策略的语法是:

```json
{
  "routes": [
    {
      "method": "GET",
      "path": "/",
      "handler": "MyPlugin.index",
      "config": {
        "policies": ["plugins::myplugin.isAuthenticated"]
      }
    }
  ]
}
```

有关更多信息 [Policies 文章](/developer-docs/latest/development/backend-customization.md#policies) 了解详情.

## 前端开发

Strapi 的管理面板和插件系统旨在成为一种简单而强大的方式来创建新的功能。

管理面板是一个 [React](https://facebook.github.io/react/) 应用程序，可以嵌入其他 React 应用程序。这些 React 应用程序是每个 Strapi 插件的 `admin` 部分。

### 环境设置

要启用本地插件开发，您需要启动应用程序时激活前端开发模式:

:::: tabs

::: tab yarn

```bash
$ cd my-app
$ yarn develop --watch-admin
```

:::

::: tab npm

```bash
$ cd my-app
$ npm run develop -- --watch-admin
```

:::

::::

### API

#### Strapi 全局变量

`administration` 公开了一个所有插件都可以访问的全局变量。

##### `strapi.backendURL`

检索后端网址(例如 http://localhost:1337)。

##### `strapi.currentLanguage`

检索管理面板默认语言(例如 `en-US`)

##### `strapi.languages`

管理面板支持的语言数组. (e.g. `['ar', 'en', 'fr', ...]`).

##### `strapi.lockApp()`

显示将阻止用户与应用程序交互的加载程序。

##### `strapi.unlockApp()`

删除加载程序，以便用户可以与应用程序交互

##### `strapi.notification`

在代码中的任何地方使用此命令。

```js
strapi.notification.toggle(config);
```

The properties of the config object are as follows:

| key             | type          | default                  | Description                                                                                                                  |
| --------------- | ------------- | ------------------------ | :--------------------------------------------------------------------------------------------------------------------------- |
| type            | string        | success                  | `success`, `warning` or `info`                                                                                               |
| message         | object/string | app.notification.success | The main message to display (works with i18n message object, `{ id: 'app.notification.success', defaultMessage: 'Saved!' }`) |
| title           | object/string | null                     | Add a title to the notification                                                                                              |
| link            | object        | null                     | Add a link to the notification message `{ url: String, label: String|Object, target:String }`                                |
| timeout         | number        | 2500                     | Time in ms before the notification is closed                                                                                 |
| blockTransition | boolean       | false                    | Block the notification transitions to remove the timeout                                                                     |
| uid             | string        | null                     | Custom the notification uid                                                                                                  |

之前的通知 API 仍在工作，但将在控制台中显示警告消息

```js
strapi.notification.error('app.notification.error');
strapi.notification.info('app.notification.info');
strapi.notification.success('app.notification.success');
strapi.notification.warning('app.notification.warning');
```

##### `strapi.remoteURL`

管理网址(例如 http://localhost:4000/admin )。

#### 主插件对象

每个插件导出一个对象中的所有配置 `my-plugin/admin/src/index.js`

以下是它的属性:

| key                       | type    | Description                                                                                                                                                                                                             |
| ------------------------- | ------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| blockerComponent          | node    | Props can be either `null` or React node (e.g. `() => <div />`)                                                                                                                                                         |
| blockerComponentProps     | object  | Props to provide to customise the [blockerComponent](https://github.com/strapi/strapi/blob/58588e10e5d15921b0966e20ce1bc6cde70df5cc/packages/strapi-helper-plugin/lib/src/components/BlockerComponent/index.js#L81-L86) |
| description               | string  | Plugin's description retrieved from the package.json                                                                                                                                                                    |
| id                        | string  | Id of the plugin from the `package.json`                                                                                                                                                                                |
| initializer               | node    | Refer to the [Initializer 文章](#initializer)                                                                                                                                                                           |
| injectedComponents        | array   | Refer to the [Injected Component 文章](#injected-components)                                                                                                                                                            |
| isReady                   | boolean | The app will load until this property is true                                                                                                                                                                           |
| mainComponent             | node    | The plugin's App container,                                                                                                                                                                                             |
| menu                      | object  | Define where the link of your plugin will be set. Without this your plugin will not display a link 在左边的菜单里                                                                                                       |
| name                      | string  | The plugin's name retrieved from the package.json                                                                                                                                                                       |
| pluginLogo                | file    | The plugin's logo                                                                                                                                                                                                       |
| preventComponentRendering | boolean | Whether or not display the plugin's blockerComponent instead of the main component                                                                                                                                      |
| settings                  | object  | Refer to the [Plugins settings API](/developer-docs/latest/development/local-plugins-customization.md#plugin-s-front-end-settings-api)                                                                                  |
| reducers                  | object  | The plugin's redux reducers                                                                                                                                                                                             |
| trads                     | object  | The plugin's translation files                                                                                                                                                                                          |

#### 在主菜单中显示插件的链接

为了在主菜单中显示一个插件链接，插件需要导出一个菜单对象。

**Path —** `plugins/my-plugin/admin/src/index.js`.

```js
import pluginPkg from '../../package.json';
import pluginLogo from './assets/images/logo.svg';
import App from './containers/App';
import lifecycles from './lifecycles';
import trads from './translations';
import pluginId from './pluginId';

export default strapi => {
  const pluginDescription = pluginPkg.strapi.description || pluginPkg.description;
  const icon = pluginPkg.strapi.icon;
  const name = pluginPkg.strapi.name;
  const plugin = {
    blockerComponent: null,
    blockerComponentProps: {},
    description: pluginDescription,
    icon,
    id: pluginId,
    initializer: null,
    isRequired: pluginPkg.strapi.required || false,
    layout: null,
    lifecycles,
    mainComponent: App,
    name,
    pluginLogo,
    preventComponentRendering: false,
    trads,
    menu: {
      // Set a link into the PLUGINS section
      pluginsSectionLinks: [
        {
          destination: `/plugins/${pluginId}`, // Endpoint of the link
          icon,
          label: {
            id: `${pluginId}.plugin.name`, // Refers to a i18n
            defaultMessage: 'My PLUGIN',
          },
          name,
          // If the plugin has some permissions on whether or not it should be accessible
          // depending on the logged in user's role you can set them here.
          // Each permission object performs an OR comparison so if one matches the user's ones
          // the link will be displayed
          permissions: [{ action: 'plugins::content-type-builder.read', subject: null }],
        },
      ],
    },
  };

  return strapi.registerPlugin(plugin);
};
```

#### 初始化程序

在创建新插件时，默认情况下会生成该组件。使用此组件在应用程序加载时执行一些逻辑。当逻辑已经执行时，该组件应该发出 `isReady` 事件，以便用户可以与应用程序交互。

::: tip 注意
下面是 content-type-builder 内容类型构建器插件的 `Initializer` 组件。

它检查是否启用了自动重新加载特性，并根据这个值更改插件的主要组件。
:::

```js
/**
 *
 * Initializer
 *
 */

import React from 'react';
import PropTypes from 'prop-types';

import pluginId from '../../pluginId';

class Initializer extends React.PureComponent {
  // eslint-disable-line react/prefer-stateless-function
  componentDidMount() {
    const {
      admin: { autoReload, currentEnvironment },
    } = this.props;

    let preventComponentRendering;
    let blockerComponentProps;

    if (currentEnvironment === 'production') {
      preventComponentRendering = true;
      blockerComponentProps = {
        blockerComponentTitle: 'components.ProductionBlocker.header',
        blockerComponentDescription: 'components.ProductionBlocker.description',
        blockerComponentIcon: 'fa-ban',
        blockerComponentContent: 'renderButton',
      };
    } else {
      // Don't render the plugin if the server autoReload is disabled
      preventComponentRendering = !autoReload;
      blockerComponentProps = {
        blockerComponentTitle: 'components.AutoReloadBlocker.header',
        blockerComponentDescription: 'components.AutoReloadBlocker.description',
        blockerComponentIcon: 'fa-refresh',
        blockerComponentContent: 'renderIde',
      };
    }

    // Prevent the plugin from being rendered if currentEnvironment === PRODUCTION
    this.props.updatePlugin(pluginId, 'preventComponentRendering', preventComponentRendering);
    this.props.updatePlugin(pluginId, 'blockerComponentProps', blockerComponentProps);
    // Emit the event plugin ready
    this.props.updatePlugin(pluginId, 'isReady', true);
  }

  render() {
    return null;
  }
}

Initializer.propTypes = {
  admin: PropTypes.object.isRequired,
  updatePlugin: PropTypes.func.isRequired,
};

export default Initializer;
```

#### 注入组件

(Coming soon)

#### Routing 路由选择

路由基于 [React Router V5](https://reacttraining.com/react-router/web/guides/philosophy) ，由于它的实现，每个路由都在 `containers/App/index.js` 文件中声明。

::: tip
插件中定义的每条路由都必须以插件 id 作为前缀。
:::

**Route 声明 :**

假设您想要创建一个路由/用户，该路由 `/user` 具有与容器 `UserPage` 关联的 params `/:id`。

宣言内容如下:

**Path —** `plugins/my-plugin/admin/src/containers/App/index.js`.

```js
import React from 'react';
import pluginId from '../../pluginId';

import UserPage from '../UserPage';

// ...

class App extends React.Component {
  // ...

  render() {
    return (
      <div>
        <Switch>
          <Route exact path={`/plugins/${pluginId}/user/:id`} component={UserPage} />
        </Switch>
      </div>
    );
  }
}

// ...
```

#### Styling

管理面板使用 [styled-components](https://styled-components.com/) 来编写 css。

#### i18n

[React Intl](https://github.com/yahoo/react-intl) 提供 React 组件和 API 来格式化日期、数字和字符串，包括多元化和处理翻译。

**用法**

我们建议将所有组件文本设置在翻译文件夹中。

下面的例子展示了如何在你的插件中使用 i18n。

用相关的消息定义所有的 id:

**用相关的消息定义所有的 ids:**

**Path —** `./plugins/my-plugin/admin/src/translations/en.json`.

```json
{
  "notification.error.message": "An error occurred"
}
```

**Path —** `./plugins/my-plugin/admin/src/translations/fr.json`

```json
{
  "notification.error.message": "Une erreur est survenue"
}
```

**组件内部的用法**

**Path —** `./plugins/my-plugin/admin/src/components/Foo/index.js`.

```js
import { FormattedMessage } from 'react-intl';
import SomeOtherComponent from 'components/SomeOtherComponent';

const Foo = props => (
  <div className={styles.foo}>
    <FormattedMessage id="my-plugin.notification.error.message" />
    <SomeOtherComponent {...props} />
  </div>
);

export default Foo;
```

请参考 [the 文章](https://github.com/yahoo/react-intl/wiki/Components#formattedmessage) 了解更广泛的用法。

#### Global context

所有的插件都包装在 `GlobalContextProvider` 中，在这个对象中你可以访问所有的插件对象以及其他实用程序。

用法:

**功能组件内部:**

```js
import React from 'react';
import { useGlobalContext } from 'strapi-helper-plugin';

const Foo = () => {
  const globalContext = useGlobalContext();

  console.log(globalContext);

  return <div>Foo</div>;
};
```

**类组件内部:**

```js
import React from 'react';
import { GlobalContext } from 'strapi-helper-plugin';

class Foo extends React.Component {
  static contextType = GlobalContext;

  render() {
    console.log(this.context);

    return <div>Foo</div>;
  }
}
```

<!--- LOCAL PLUGIN API --->

## 插件的前端字段 API

作为插件开发者，您可能需要在应用程序中添加自定义字段。要做到这一点，一个 **Field API** 是可用的，以便插件注册一个字段，这将是可用的所有插件。

::: warning NOTE

目前，只有内容管理器使用此 API 扩展其当前字段。

:::

### 注册一个新 field

注册一个字段可以用两种不同的方式:

1. 在插件的加载阶段
2. 使用 provided `react-hook` 在组件中.

#### 在加载插件时注册一个字段

在插件的加载阶段注册一个字段可以这样做:

1. 创建一个新的字段类型 (在此示例中 一个 **`media`** 字段类型):

**Path —** `plugins/my-plugin/admin/src/components/InputMedia/index.js`.

```js
import React from 'react';
const InputMedia = props => {
  // Check out the provided props
  console.log(props);

  return <div>InputMedia</div>;
};

export default InputMedia;
```

2. 将字段注册到应用程序中:

**Path —** `plugins/my-plugin/admin/src/index.js`.

```js
import pluginPkg from '../../package.json';
import InputMedia from './components/InputMedia';
import pluginId from './pluginId';

export default strapi => {
  const pluginDescription = pluginPkg.strapi.description || pluginPkg.description;

  const plugin = {
    blockerComponent: null,
    blockerComponentProps: {},
    description: pluginDescription,
    icon: pluginPkg.strapi.icon,
    id: pluginId,
    initializer: () => null,
    injectedComponents: [],
    isReady: true,
    mainComponent: null,
    name: pluginPkg.strapi.name,
    preventComponentRendering: false,
    trads: {},
  };

  strapi.registerField({ type: 'media', Component: InputMedia });

  return strapi.registerPlugin(plugin);
};
```

通过这样做，所有的插件从您的项目将能够使用新注册的 **Field** 字段类型。

#### 在 React 组件中注册一个字段

注册 **Field** 的另一种方法是使用提供的 `react-hook`: **`useStrapi`**，它可以在 `Initializer` 组件中完成，因此当用户登录时可以直接访问它，如果您决定在另一个组件中注册您的插件而不是 `Initializer`，那么 **Field** 将只在组件挂载后才在管理面板中注册(用户已经导航到注册 **Field** 的视图)。

1. 注册 **Field** 字段在 `Initializer` 组件:

**Path —** `plugins/my-plugin/admin/src/containers/Initializer/index.js`.

```js
/**
 *
 * Initializer
 *
 */

import { useEffect, useRef } from 'react';
import PropTypes from 'prop-types';
import { useStrapi } from 'strapi-helper-plugin';
import pluginId from '../../pluginId';
import InputMedia from './components/InputMedia';

const Initializer = ({ updatePlugin }) => {
  const {
    strapi: { fieldApi },
  } = useStrapi();
  const ref = useRef();
  ref.current = updatePlugin;

  useEffect(() => {
    // Register the new field
    fieldApi.registerField({ type: 'media', Component: InputMedia });

    ref.current(pluginId, 'isReady', true);
  }, []);

  return null;
};

Initializer.propTypes = {
  updatePlugin: PropTypes.func.isRequired,
};

export default Initializer;
```

2. 添加一个 `Initializer` 组件作为你的插件，当用户登录时触发:

```js
import pluginPkg from '../../package.json';
import pluginLogo from './assets/images/logo.svg';
import App from './containers/App';
import Initializer from './containers/Initializer';
import lifecycles from './lifecycles';
import trads from './translations';
import pluginId from './pluginId';

export default strapi => {
  const pluginDescription = pluginPkg.strapi.description || pluginPkg.description;
  const plugin = {
    blockerComponent: null,
    blockerComponentProps: {},
    description: pluginDescription,
    icon: pluginPkg.strapi.icon,
    id: pluginId,
    initializer: Initializer,
    injectedComponents: [],
    isRequired: pluginPkg.strapi.required || false,
    layout: null,
    lifecycles,
    mainComponent: App,
    name: pluginPkg.strapi.name,
    pluginLogo,
    preventComponentRendering: false,
    trads,
  };

  return strapi.registerPlugin(plugin);
};
```

### 使用 Field API

使用 **Field** API 只能通过使用提供的 `react-hook` **`useStrapi`** 来完成。下面是 **content-manager** 插件的一个例子:

**Path —** `~/strapi-plugin-content-manager/admin/src/components/Inputs/index.js`.

```js
import React, { memo, useMemo } from 'react';
// Other imports
// ...
// Import the Inputs component from our component library Buffet.js
import { Inputs as InputsIndex } from '@buffetjs/custom';

// Import the Hook with which you can access the Field API
import { useStrapi } from 'strapi-helper-plugin';

function Inputs({ autoFocus, keys, layout, name, onBlur }) {
  // This is where you will access the field API
  const {
    strapi: { fieldApi },
  } = useStrapi();

  // Other boilerplate code
  // ...

  return (
    <FormattedMessage id={errorId}>
      {error => {
        return (
          <InputsIndex
            {...metadatas}
            autoComplete="new-password"
            autoFocus={autoFocus}
            didCheckErrors={didCheckErrors}
            disabled={disabled}
            error={
              isEmpty(error) || errorId === temporaryErrorIdUntilBuffetjsSupportsFormattedMessage
                ? null
                : error
            }
            inputDescription={description}
            description={description}
            contentTypeUID={layout.uid}
            customInputs={{
              json: InputJSONWithErrors,
              wysiwyg: WysiwygWithErrors,
              uid: InputUID,

              // Retrieve all the fields that other plugins have registered
              ...fieldApi.getFields(),
            }}
            multiple={get(attribute, 'multiple', false)}
            attribute={attribute}
            name={keys}
            onBlur={onBlur}
            onChange={onChange}
            options={enumOptions}
            step={step}
            type={getInputType(type)}
            validations={validations}
            value={inputValue}
            withDefaultValue={false}
          />
        );
      }}
    </FormattedMessage>
  );
}
```

### Field API 定义

| Method        | Param         | Description        |
| :------------ | :------------ | :----------------- |
| getField      | {String} type | 根据类型检索 Field |
| getFields     |               | 检索所有 Field     |
| registerField | {Object}      | 注册一个 Field     |
| removeField   |               | 删除一个 Field     |

## 插件的前端设置 API

作为插件开发者，您可能需要在主应用程序 **`Settings`** 视图中添加一些设置(它对应于菜单中的 `Settings` 链接)。要做到这一点，一个 API 是可用的，以便插件添加链接到主视图。

这些设置可以直接声明到主插件对象中，这样就可以动态地将它们注入到视图中。

### 增加一个设置

插件的前端部分导出一个函数，该函数在管理面板中注册该插件。参数由两个主要参数组成:

- `registerPlugin`: `Function`
- `settingsBaseURL`: `String`

#### 创建视图菜单中的链接

每个带有设置对象的插件都会在视图的菜单中创建一个新的部分。

菜单部分可以声明如下:

**Path —** `plugins/my-plugin/admin/src/index.js`.

```js
import pluginPkg from '../../package.json';
import pluginId from './pluginId';

export default strapi => {
  const pluginDescription = pluginPkg.strapi.description || pluginPkg.description;

  // Declare the links that will be injected into the settings menu
  const menuSection = {
    // Unique id of the section
    id: pluginId,
    // Title of Menu section using i18n
    title: {
      id: `${pluginId}.foo`,
      defaultMessage: 'Super cool setting',
    },
    // Array of links to be displayed
    links: [
      {
        // Using string
        title: 'Setting page 1',
        to: `${strapi.settingsBaseURL}/${pluginId}/setting1`,
        name: 'setting1',
        permissions: [{ action: 'plugins::my-plugin.action-name', subject: null }], // This key is not mandatory it can be null, undefined or an empty array
      },
      {
        // Using i18n with a corresponding translation key
        title: {
          id: `${pluginId}.bar`,
          defaultMessage: 'Setting page 2',
        },
        to: `${strapi.settingsBaseURL}/${pluginId}/setting2`,
        name: 'setting2',
        // Define a specific component if needed:
        Component: () => <div />,
      },
    ],
  };

  const plugin = {
    blockerComponent: null,
    blockerComponentProps: {},
    description: pluginDescription,
    icon: pluginPkg.strapi.icon,
    id: pluginId,
    initializer: () => null,
    injectedComponents: [],
    isReady: true,
    mainComponent: null,
    name: pluginPkg.strapi.name,
    preventComponentRendering: false,
    settings: {
      menuSection,
    },
    trads: {},
  };

  return strapi.registerPlugin(plugin);
};
```

在这一点上，该插件创建了一个新的部分(**_Super cool setting_**) ，将包含两个链接`Setting page 1`和`Setting page 2`这些链接不指向任何组件作为相应的一个尚未宣布。

#### 声明设置组件

导出的 Setting 组件接收 `settingsBaseURL` 作为道具，以便生成一个动态路由，该路由应用于将创建的两个端点与其相应的组件关联起来。

通过上面的配置，我们可以轻松地创建我们的插件设置视图。

**Path —** `plugins/my-plugin/admin/src/containers/Settings/index.js`.

```js
import React from 'react';
import PropTypes from 'prop-types';
import { Switch, Route } from 'react-router-dom';
import pluginId from '../../pluginId';

const SettingPage1 = () => (
  <div>
    <h1>Setting Page 1</h1>
  </div>
);
const SettingPage2 = () => (
  <div>
    <h1>Setting Page 2</h1>
  </div>
);

const Settings = ({ settingsBaseURL }) => {
  return (
    <Switch>
      <Route component={SettingPage1} path={`${settingsBaseURL}/${pluginId}/setting1`} />
      <Route component={SettingPage2} path={`${settingsBaseURL}/${pluginId}/setting2`} />
    </Switch>
  );
};

Settings.propTypes = {
  settingsBaseURL: PropTypes.string.isRequired,
};

export default Settings;
```

现在 `Settings` 组件已经在你的插件中声明，剩下的唯一事情就是将它添加到你的设置配置中:

**Path —** `plugins/my-plugin/admin/src/index.js`.

```js
import pluginPkg from '../../package.json';
// Import the component
import Settings from './containers/Settings';
import pluginId from './pluginId';

export default strapi => {
  const pluginDescription = pluginPkg.strapi.description || pluginPkg.description;

  // Declare the links that will be injected into the settings menu
  const menuSection = {
    id: pluginId,
    title: {
      id: `${pluginId}.foo`,
      defaultMessage: 'Super cool setting',
    },
    links: [
      {
        title: 'Setting page 1',
        to: `${strapi.settingsBaseURL}/${pluginId}/setting1`,
        name: 'setting1',
        permissions: [{ action: 'plugins::my-plugin.action-name', subject: null }],
      },
      {
        title: {
          id: `${pluginId}.bar`,
          defaultMessage: 'Setting page 2',
        },
        to: `${strapi.settingsBaseURL}/${pluginId}/setting2`,
        name: 'setting2',
      },
    ],
  };

  const plugin = {
    blockerComponent: null,
    blockerComponentProps: {},
    description: pluginDescription,
    icon: pluginPkg.strapi.icon,
    id: pluginId,
    initializer: () => null,
    injectedComponents: [],
    isReady: true,
    mainComponent: null,
    name: pluginPkg.strapi.name,
    preventComponentRendering: false,
    settings: {
      mainComponent: Settings,
      menuSection,
    },
    trads: {},
  };

  return strapi.registerPlugin(plugin);
};
```

### 全局添加一个设置

为了将链接添加到设置视图的全局部分，您需要创建一个全局数组，其中包含您要添加的链接:

**Path —** `plugins/my-plugin/admin/src/index.js`.

```js
import pluginPkg from '../../package.json';
// Import the component
import Settings from './containers/Settings';
import SettingLink from './components/SettingLink';
import pluginId from './pluginId';

export default strapi => {
  const pluginDescription = pluginPkg.strapi.description || pluginPkg.description;

  // Declare the links that will be injected into the settings menu
  const menuSection = {
    id: pluginId,
    title: {
      id: `${pluginId}.foo`,
      defaultMessage: 'Super cool setting',
    },
    links: [
      {
        title: 'Setting page 1',
        to: `${strapi.settingsBaseURL}/${pluginId}/setting1`,
        name: 'setting1',
      },
      {
        title: {
          id: `${pluginId}.bar`,
          defaultMessage: 'Setting page 2',
        },
        to: `${strapi.settingsBaseURL}/${pluginId}/setting2`,
        name: 'setting2',
      },
    ],
  };

  const plugin = {
    blockerComponent: null,
    blockerComponentProps: {},
    description: pluginDescription,
    icon: pluginPkg.strapi.icon,
    id: pluginId,
    initializer: () => null,
    injectedComponents: [],
    isReady: true,
    mainComponent: null,
    name: pluginPkg.strapi.name,
    preventComponentRendering: false,
    settings: {
      // Add a link into the global section of the settings view
      global: {
        links: [
          {
            title: 'Setting link 1',
            to: `${strapi.settingsBaseURL}/setting-link-1`,
            name: 'settingLink1',
            Component: SettingLink,
            // Bool : https://reacttraining.com/react-router/web/api/Route/exact-bool
            exact: false,
            permissions: [{ action: 'plugins::my-plugin.action-name', subject: null }],
          },
        ],
      },
      mainComponent: Settings,
      menuSection,
    },
    trads: {},
  };

  return strapi.registerPlugin(plugin);
};
```

::: danger
目前不可能添加到另一个插件的设置部分的链接
:::
