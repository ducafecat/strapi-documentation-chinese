# 命令行接口 (CLI)

Strapi 提供了一个全功能的命令行界面(CLI) ，它可以让您在几秒钟内支架和管理您的项目。

## strapi new

创建一个新项目。

```bash
strapi new <name>

options: [--no-run|--use-npm|--debug|--quickstart|--dbclient=<dbclient> --dbhost=<dbhost> --dbport=<dbport> --dbname=<dbname> --dbusername=<dbusername> --dbpassword=<dbpassword> --dbssl=<dbssl> --dbauth=<dbauth> --dbforce]
```

- **strapi new &#60;name&#62;**<br/>
  Strapi new < name > 生成一个名为 < name > 的新项目，并通过 npm 注册表安装默认插件。

- **strapi new &#60;name&#62; --debug**<br/>
  Strapi new < name > -- debug 将显示在数据库连接期间触发的完整错误消息。

- **strapi new &#60;name&#62; --quickstart**<br/>
  使用快速启动系统创建你的应用程序。

- **strapi new &#60;name&#62; --quickstart --no-run**<br/>
  使用快速启动系统创建应用程序，创建后不要启动应用程序。

- **strapi new &#60;name&#62; --dbclient=&#60;dbclient&#62; --dbhost=&#60;dbhost&#62; --dbport=&#60;dbport&#62; --dbname=&#60;dbname&#62; --dbusername=&#60;dbusername&#62; --dbpassword=&#60;dbpassword&#62; --dbssl=&#60;dbssl&#62; --dbauth=&#60;dbauth&#62; --dbforce**<br/>

  生成一个名为 < name > 的新项目，跳过交互式数据库配置并使用这些选项初始化。

  - **&#60;dbclient&#62;** can be `mongo`, `postgres`, `mysql`.
  - **&#60;dbssl&#62;** and **&#60;dbauth&#62;** are available only for `mongo` and are optional.
  - **--dbforce** Allows you to overwrite content if the provided database is not empty. Only available for `postgres`, `mysql`, and is optional.

## strapi develop

**Alias**: `dev`

启动启用 autoReload 的 Strapi 应用程序。

Strapi 在运行时修改/创建文件，并且在创建新文件时需要重新启动。为了实现这一点，`strapi develop` 添加了一个文件监视器，并在必要时重新启动应用程序。

```
strapi develop
options: [--no-build |--watch-admin |--browser ]
```

- **strapi develop**<br/>
  启用 autoReload 启动应用程序.
- **strapi develop --no-build**<br/>
  启用 autoReload 启动应用程序， 并跳过管理面板生成过程.
- **strapi develop --watch-admin**<br/>
  启用 autoReload 启动应用程序， 前端开发服务器的情况下启动应用程序。它允许您自定义管理面板.
- **strapi develop --watch-admin --browser 'google chrome'**<br/>
  启用 autoReload 启动应用程序， 前端开发服务器启动应用程序。它允许您自定义管理面板。提供一个浏览器名称来代替默认名称,`false` means stop opening the browser. 意味着停止打开浏览器.

::: tip
千万不要使用此命令在生产环境中运行 Strapi 应用程序。
:::

## strapi start

启动禁用 autoReload 的 Strapi 应用程序。

这个命令用于运行 Strapi 应用程序，而不需要重新启动和写入文件(针对生产用途)。

某些功能在 `strapi start` 模式下被禁用，因为它们需要重新启动应用程序。

允许的环境变量:
| Property | Description | Type | Default |
| --------- | ----------- | ----- | ------- |
| STRAPI_HIDE_STARTUP_MESSAGE | If `true` then Strapi will not show startup message on boot. Values can be `true` or `false` | string | `false` |
| STRAPI_LOG_LEVEL | Values can be 'fatal', 'error', 'warn', 'info', 'debug', 'trace' | string | `debug` |
| STRAPI_LOG_TIMESTAMP | Enables or disables the inclusion of a timestamp in the log message. Values can be `true` or `false` | string | `false`|
| STRAPI_LOG_FORCE_COLOR | Values can be `true` or `false` | string | `true` |
| STRAPI_LOG_PRETTY_PRINT | If pino-pretty module will be used to format logs. Values can be `true` or `false` | string | `true` |

## strapi build

建立你的管理面板.

```bash
strapi build

options: [--no-optimization]
```

- **strapi build**<br/>
  建立管理面板并最小化资产.
- **strapi build --clean**<br/>
  生成管理面板并删除以前的生成和. 缓存文件夹.
- **strapi build --no-optimization**<br/>
  在不最小化资产的情况下构建管理面板。构建持续时间更快.

## strapi configuration:dump

**Alias**: `config:dump`

将配置转储到文件或 stdout，以帮助您迁移到生产环境。

转储格式将是一个 JSON 数组。

```
strapi configuration:dump

Options:
  -f, --file <file>  Output file, default output is stdout
  -p, --pretty       Format the output JSON with indentation and line breaks (default: false)
```

**例子**

- `strapi configuration:dump -f dump.json`
- `strapi config:dump --file dump.json`
- `strapi config:dump > dump.json`

这些例子都是等价的。

::: warning
在配置应用程序时，通常需要输入第三方服务(例如身份验证提供者)的凭据。请注意，这些凭据也将被转储到此命令的输出中。如果有疑问，应该避免将转储文件提交到版本控制系统中。以下是一些你可以探索的方法:

- 将文件直接复制到所需的环境中，并在其中运行 restore 命令.
- 将文件放在安全位置，并在部署时使用正确的凭据下载该文件.
- 在提交文件之前对其进行加密，并在运行还原命令时对其解密.

:::

## strapi configuration:restore

**Alias**: `config:restore`

将配置转储还原到应用程序中。

输入格式必须是 JSON 数组。

```
strapi configuration:restore

Options:
  -f, --file <file>          Input file, default input is stdin
  -s, --strategy <strategy>  Strategy name, one of: "replace", "merge", "keep". Defaults to: "replace"
```

**例子**

- `strapi configuration:restore -f dump.json`
- `strapi config:restore --file dump.json -s replace`
- `cat dump.json | strapi config:restore`
- `strapi config:restore < dump.json`

这些例子都是等价的。

**Strategies**

运行 restore 命令时，您可以选择三种不同的策略:

- **replace**: 将创建缺少的键并替换现有的键.
- **merge**: 将创建缺少的键并将现有键与其新值合并.
- **keep**: 将创建缺少的密钥并保持现有密钥不变.

## strapi admin:reset-user-password

**Alias** `admin:reset-password`

别名管理员: 重置密码

重置管理员用户的密码。您可以将电子邮件和新密码作为选项传递，或者如果调用该命令而不传递选项，则可以交互式地设置它们。

**例子**

```bash
strapi admin:reset-user-password --email=chef@strapi.io --password=Gourmet1234
```

**选择**

| Option         | Type   | Description               |
| -------------- | ------ | ------------------------- |
| -e, --email    | string | The user email            |
| -p, --password | string | New password for the user |
| -h, --help     |        | display help for command  |

## strapi generate:api

脚手架一个完整的 API 及其配置、控制器、模型和服务。

```bash
strapi generate:api <name> [<attribute:type>]

options: [--plugin <name>]
```

- **strapi generate:api &#60;name&#62;**<br/>
  Generates an API called **&#60;name&#62;** in the `./api` folder at the root of your project.

- **strapi generate:api &#60;name&#62; --draft-and-publish=true**<br/>
  Generates an API called **&#60;name&#62;** in the `./api` folder at the root of your project and enabled the draft/publish feature.

- **strapi generate:api &#60;name&#62; &#60;attribute:type&#62;**<br/>
  Generates an API called **&#60;name&#62;** in the `./api` folder at the root of your project. The model will already contain an attribute called **&#60;attribute&#62;** with the type property set to **&#60;type&#62;**.

  例子: `strapi generate:api product name:string description:text price:integer`

- **strapi generate:api &#60;name&#62; --plugin &#60;plugin&#62;**<br/>
  Generates an API called **&#60;name&#62;** in the `./plugins/<plugin>` folder.

  例子: `strapi generate:api product --plugin content-manager`

::: tip
文件名的第一个字母是大写的。
:::

## strapi generate:controller

创建一个新控制器。

```bash
strapi generate:controller <name>

options: [--api <name>|--plugin <name>]
```

- **strapi generate:controller &#60;name&#62;**<br/>
  Generates an empty controller called **&#60;name&#62;** in the `./api/<name>/controllers` folder.

  例子: `strapi generate:controller category` will create the controller at `./api/category/controllers/Category.js`.

- **strapi generate:controller &#60;name&#62; --api &#60;api&#62;**<br/>
  Generates an empty controller called **&#60;name&#62;** in the `./api/<api>/controllers` folder.

  例子: `strapi generate:controller category --api product` will create the controller at `./api/product/controllers/Category.js`.

- **strapi generate:controller &#60;name&#62; --plugin &#60;plugin&#62;**<br/>
  Generates an empty controller called **&#60;name&#62;** in the `./plugins/<plugin>/controllers` folder.

::: tip
文件名的第一个字母是大写的。
:::

## strapi generate:model

创建一个新模型。

```bash
strapi generate:model <name> [<attribute:type>]

options: [--api <name>|--plugin <name>|--draft-and-publish <boolean>]
```

- **strapi generate:model &#60;name&#62;**<br/>
  Generates an empty model called **&#60;name&#62;** in the `./api/<name>/models` folder. It will create two files.
  The first one will be **&#60;name&#62;.js** which contains your lifecycle callbacks and another **&#60;name&#62;.settings.json** that will list your attributes and options.

  例子: `strapi generate:model category` will create these two files `./api/category/models/Category.js` and `./api/category/models/Category.settings.json`.

- **strapi generate:model &#60;name&#62; &#60;attribute:type&#62;**<br/>
  Generates an empty model called **&#60;name&#62;** in the `./api/<name>/models` folder. The file **&#60;name&#62;.settings.json** will already contain a list of attribute with their associated **&#60;type&#62;**.

  例子: `strapi generate:model category name:string description:text` will create these two files `./api/category/models/Category.js` and `./api/category/models/Category.settings.json`. This last file will contain two attributes `name` with the type `string` and `description` with type `text`.

- **strapi generate:model &#60;name&#62; --api &#60;api&#62;**<br/>
  Generates an empty model called **&#60;name&#62;** in the `./api/<api>/models` folder.

  例子: `strapi generate:model category --api product` will create these two files:

  - `./api/product/models/Category.js`
  - `./api/product/models/Category.settings.json`.

* **strapi generate:model &#60;name&#62; --plugin &#60;plugin&#62;**<br/>
  Generates an empty model called **&#60;name&#62;** in the `./plugins/<plugin>/models` folder.

* **strapi generate:model &#60;name&#62; --draft-and-publish=true**<br/>
  Generates an empty model called **&#60;name&#62;** in the `./plugins/<plugin>/models` folder with the draft/publish feature enabled

::: tip
文件名的第一个字母是大写的。
:::

## strapi generate:service

创建一个新的服务。

```bash
strapi generate:service <name>

options: [--api <name>|--plugin <name>]
```

- **strapi generate:service &#60;name&#62;**<br/>
  Generates an empty service called **&#60;name&#62;** in the `./api/<name>/services` folder.

  例子: `strapi generate:service category` will create the service at `./api/category/services/Category.js`.

- **strapi generate:service &#60;name&#62; --api &#60;api&#62;**<br/>
  Generates an empty service called **&#60;name&#62;** in the `./api/<api>/services` folder.

  例子: `strapi generate:service category --api product` will create the service at `./api/product/services/Category.js`.

- **strapi generate:service &#60;name&#62; --plugin &#60;plugin&#62;**<br/>
  Generates an empty service called **&#60;name&#62;** in the `./plugins/<plugin>/services` folder.

::: tip
文件名的第一个字母是大写的。
:::

## strapi generate:policy

创建一个新的策略。

```bash
strapi generate:policy <name>

options: [--api <name>|--plugin <name>]
```

- **strapi generate:policy &#60;name&#62;**<br/>
  Generates an empty policy called **&#60;name&#62;** in the `./config/policies` folder.

  例子: `strapi generate:policy isAuthenticated` will create the policy at `./config/policies/isAuthenticated.js`.

- **strapi generate:policy &#60;name&#62; --api &#60;api&#62;**<br/>
  Generates an empty policy called **&#60;name&#62;** in the `./api/<api>/config/policies` folder. This policy will be scoped and only accessible by the **&#60;api&#62;** routes.

  例子: `strapi generate:policy isAuthenticated --api product` will create the policy at `./api/product/config/policies/isAuthenticated.js`.

- **strapi generate:policy &#60;name&#62; --plugin &#60;plugin&#62;**<br/>
  Generates an empty policy called **&#60;name&#62;** in the `./plugins/<plugin>/config/policies` folder. This policy will be scoped and accessible only by the **&#60;plugin&#62;** routes.

## strapi generate:plugin

创建一个新的插件框架。

```bash
strapi generate:plugin <name>
```

- **strapi generate:plugin &#60;name&#62;**<br/>
  Generates an empty plugin called **&#60;name&#62;** in the `./plugins` folder.

  例子: `strapi generate:plugin user` will create the plugin at `./plugins/user`.

有关更多信息 [local plugins](/developer-docs/latest/development/local-plugins-customization.md) section to know more.

## strapi install

在项目中安装插件。

```bash
strapi install <name>
```

- **strapi install &#60;name&#62;**<br/>
  Installs a plugin called **&#60;name&#62;**.

  例子: `strapi install graphql` will install the plugin `strapi-plugin-graphql`

::: warning
一些插件有管理面板集成，你的管理面板可能需要重建。这可能需要一些时间。
:::

## strapi uninstall

从项目中卸载插件。

```bash
strapi uninstall <name>

options [--delete-files]
```

- **strapi uninstall &#60;name&#62;**<br/>
  Uninstalls a plugin called **&#60;name&#62;**.

  例子: `strapi uninstall graphql` will remove the plugin `strapi-plugin-graphql`

- **strapi uninstall &#60;name&#62; --delete-files**<br/>
  Uninstalls a plugin called **&#60;name&#62;** and removes the files in `./extensions/name/`

  例子: `strapi uninstall graphql --delete-files` will remove the plugin `strapi-plugin-graphql` and all the files in `./extensions/graphql`

::: warning
一些插件有管理面板集成，你的管理面板可能需要重建。这可能需要一些时间。
:::

## strapi console

实时启动应用程序中的服务器和 eval 命令。

```bash
strapi console
```

## strapi version

打印当前全局安装的 Strapi 版本。

```bash
strapi version
```

## strapi help

列出 CLI 命令。

```
strapi help
```
