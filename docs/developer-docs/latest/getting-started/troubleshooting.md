---
sidebarDepth: 0
---

# 常见问题

以下是您在使用 Strapi 时可能遇到的最常见问题的答案和解决方案。

## 为什么我不能在产品/阶段中创建或更新内容类型？

Strapi 将模型配置文件(定义模型模式的文件)存储在诸如 `api/restaurant/models/restaurant.settings.json` 之类的文件中。由于 Node.js 的工作原理，为了使更改生效，需要 Node 重新启动服务器。这可能会导致生产服务停机，同样，这些更改应该在某种源代码控制中进行跟踪。

一般来说，你的工作 “流程” 将遵循以下顺序:

- 开发 - 在主机上本地开发 Strapi 应用程序，然后将更改推入源代码管理
- 开发迭代 - 将更改从源代码管理部署到 “类生产” 环境进行测试
- 生产 - 如果不需要其他更改，部署到生产环境
- 版本迭代, 建议您正确地对应用程序进行版本和测试

目前和将来都没有计划允许在生产环境中创建或更新模型，目前也没有计划将模型设置移入数据库。目前还没有已知或推荐的解决办法。

## Strapi 是否负责内容的部署或迁移？

Strapi 目前不提供任何工具，用于在不同环境之间迁移或部署数据更改(即。从开发到生产)。除了 Content-Manager 设置之外，要了解有关此选项的更多信息，请参阅以下 [CLI documentation](/developer-docs/latest/developer-resources/cli/CLI.md#strapi-configuration-dump)。

## 用户无法登录到管理面板

随着 Strapi beta 版本的发布，发生了一个根本性的变化，“终端用户”(REST 和 GraphQL 用户)从 Administrators (管理面板用户)中分离出来，普通用户无法访问管理面板。如果你想了解更多关于为什么做了这个改变，你可以阅读 [blog post](https://strapi.io/blog/why-we-split-the-management-of-the-admin-users-and-end-users)。

Strapi 已经发布了新的管理和权限(RBAC-基于角色的访问控制) ，允许对用户在管理面板中可以访问的内容进行一定程度的控制，并包括一些字段级别的权限。您现在还可以为内容类型、单一类型、插件和设置等内容指定角色的权限。

当这个新的插件发布时，有两个版本:

- Community Edition 社区版
- Enterprise Edition 企业版

默认情况下，Community Edition 包含 3 个预定义角色(Administrators、 Editor、 Author)。升级到企业版将解锁无限数量的角色。在版本的基础上还会有一些其他字段级别的权限限制，我们将为“基本”与“高级”RBAC 特性中包含的内容构建详细的指南。要了解更多关于什么是包括以及价格请参阅我们的 [定价页面](https://strapi.io/pricing) 。

## 关系没有维持他们的排序顺序

对于组件，有一个称为 `order` 的隐藏字段，允许条目维护它们的排序，但是对于关系，则没有这样的字段。如果考虑组件条目与基于关系的条目的典型计数(回顾一下它们在后端的功能是相同的) ，通常会有更多的关系。如果关系也有一个 `order` 字段应用于它们，那么在试图更新订单时可能会导致严重的性能下降，同样，在关系可能被附加到多个条目的情况下，维护订单也会相当困难。

目前没有推荐的方法来自动处理这个问题，相反，您可能需要创建自定义控制器来在自己的项目中处理这个问题。

## 为什么我的应用程序的数据库和上传的文件重置在 PaaS

如果您使用 `--quickstart` 来创建 Strapi 项目，默认情况下使用 SQLite 数据库。PaaS 系统(Heroku、 DigitalOcean Apps、 Google App Engine 等)文件系统通常是 [ephemeral](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem) 或只读的，这意味着每次重置 dyno (容器)时，所有文件系统更改都丢失。而且，由于 SQLite 和本地上传都存储在文件系统上，自上次 dyno 重置以来对它们所做的任何更改都将被删除。通常，dynos 至少每天重置一次，在大多数情况下，每天重置多次，或者在新代码被推送到这些服务时重置。

建议您使用类似 Heroku 的 PostgreSQL 这样的数据库附加组件，或者使用类似 MongoDB 的 Atlas 之类的东西作为数据库。对于文件上传，您需要使用第三方提供商之一，如 Cloudinary 或 AWS S3。

## 我可以在模型设置中存储我的内容管理器布局配置吗

目前 Strapi 不支持这一点，添加了一个配置: `config:dump` 和 `config:restore` 命令，以便在不同的部署和环境之间移动时更容易地迁移这些设置。

我们不能在模型设置中存储这些配置，原因如下:

- 它将在内容国际化和管理界面中的翻译的情况下产生冲突
- 根据角色和权限，布局可能不同
- 尽管无论创建什么内容，模型都是相同的，但贡献界面可能不同。例如，我们有一个想法，创建一个移动应用程序的贡献者。标签和布局配置可根据设备和界面的不同而不同

由于所有这些原因，以及其他原因，我们认为这将是一个错误，如果我们将配置存储在模型设置文件中，可能会使用户感到困惑。最终的解决方案是简化跨环境的迁移和部署。

## 如何定制插件

Strapi 使用一个叫做 [扩展](/developer-docs/latest/development/plugin-customization.md) 的系统，因为插件存储在 `node_modules` 文件夹中。由于这种扩展工作，Strapi 检测更新的版本的文件，并用它作为替代那些存储在 `node_modules` 。如果您熟悉 React 和 "ejecting" 文件，概念是类似的。

你可以修改这些文件而不需要分叉插件包，但是你会失去更新的能力。在每个版本发布之后，您需要将您的更改与新版本中的更改进行比较，并相应地修改您的文件版本。

## 我可以添加自己的第三方认证提供者吗

是的，您可以遵循以下 [指南](/developer-docs/latest/development/plugins/users-permissions.md#adding-a-new-provider-to-your-project) ，或者您可以查看 [用户权限](https://github.com/strapi/strapi/tree/master/packages/strapi-plugin-users-permissions) 并提交一个拉请求，以包括每个人的提供者。最终 Strapi 确实计划从目前的授权/纯粹的提供者转移到一个类似于上传提供者的分离性质的系统。

然而，目前还没有关于这次迁移的 ETA。

## 允许我更改默认 ID 类型或名称吗

不，目前没有能力改变默认的 id 名称，也不允许您切换数据类型(如在书架上的 UUID 和在 mongoose 上的整数) ，对此的支持正在寻找 Strapi v4。

## 可以对组件、动态区域和多态关系进行过滤/深度过滤吗

目前不可能对组件、动态区域或多态关系进行过滤。这是我们正在研究的东西，如果你想了解更多信息，请看下面的 [GitHub issue](https://github.com/strapi/strapi/issues/5124)。

## 如何使用 Strapi 设置 SSL

> 🐱: 可以用 nginx 反向代理 80 -> 1337

在本地没有实现 SSL 解决方案，这是因为以低端口直接向公共 web 提供 Node.js 应用程序是极不安全的。

在基于 Linux 的操作系统上，你需要 root 权限来绑定到 1024 以下的任何端口，而典型的 SSL 端口是 443，你需要以 root 身份运行你的应用程序。

同样，由于 Strapi 是基于 Node.js 的，为了更改 SSL 证书(比如说当它到期时) ，你需要重新启动你的应用程序，这个更改才能生效。

由于这两个问题，建议您使用 Nginx、 Apache、 Traefik 或许多其他代理应用程序来处理到 Strapi 的路由。环境 [server.json](/developer-docs/latest/setup-deployment-guides/configurations.md#server) 中有处理上游代理的设置。代理块需要填写所有设置，并修改任何后端插件，如认证提供者和上传插件，以替换你的标准本地主机: `localhost:1337` URL。

## X 功能还可用吗

> 🐱: 看着像许愿列表

您可以查看 [ProductBoard roadmap](https://portal.productboard.com/strapi) ，以查看当前正在处理哪些特性请求，哪些特性请求尚未启动。
