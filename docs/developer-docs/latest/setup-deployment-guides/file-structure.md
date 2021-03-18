# 工程项目结构

默认情况下，Strapi 项目的结构如下所示:

- `/.cache`: 包含用于构建管理面板的文件.
- [`/admin`](/developer-docs/latest/development/admin-customization.md): _(optional)_ 包含您的管理定制文件.
- `/api`: 包含将项目的业务逻辑拆分为每个 API 的子文件夹.
  - `**`
    - `/config`: 包含 API 的配置 ([`routes`](/developer-docs/latest/development/backend-customization.md#routing), [`policies`](/developer-docs/latest/development/backend-customization.md#policies), etc.).
    - [`/controllers`](/developer-docs/latest/development/backend-customization.md#controllers): 包含 API 的自定义控制器.
    - [`/models`](/developer-docs/latest/development/backend-customization.md#models): 包含 API 的模型.
    - [`/services`](/developer-docs/latest/development/backend-customization.md#services): 包含 API 的定制服务.
- `/build`: 包含您的管理面板 UI 构建.
- [`/config`](/developer-docs/latest/setup-deployment-guides/configurations.md)
  - [`/functions`](/developer-docs/latest/setup-deployment-guides/configurations.md#functions): 包含项目的生命周期或一般功能.
    - [`/responses`](/developer-docs/latest/development/backend-customization.md#requests-responses): 包含自定义响应.
      - `404.js`: 包含用于构建自定义 404 消息的模板.
    - [`bootstrap.js`](/developer-docs/latest/setup-deployment-guides/configurations.md#bootstrap): 包含在应用程序启动时执行的代码.
    - [`cron.js`](/developer-docs/latest/setup-deployment-guides/configurations.md#cron-tasks): 包含 cron 任务.
  - [`server.js`](/developer-docs/latest/setup-deployment-guides/configurations.md#server): 包含项目的一般配置.
  - [`database.js`](/developer-docs/latest/setup-deployment-guides/configurations.md#database): 包含项目的数据库配置.
- `/extensions`: 包含扩展已安装插件的文件.
- [`/hooks`](/developer-docs/latest/setup-deployment-guides/configurations.md#hooks): 包含项目的自定义钩子.
- [`/middlewares`](/developer-docs/latest/setup-deployment-guides/configurations.md#middlewares): 包含项目的自定义中间件.
- [`/plugins`](/developer-docs/latest/setup-deployment-guides/configurations.md#plugins): 包含您的本地插件.
- [`/public`](/developer-docs/latest/setup-deployment-guides/configurations.md#public-assets): 包含外部世界可访问的文件.
- `/node_modules`: 包含项目使用的 npm 包.
