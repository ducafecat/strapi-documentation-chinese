# API 参考

- strapi
  - [.admin](#strapi-admin)
  - [.api](#strapi-api)
  - [.app](#strapi-app)
  - [.config](#strapi-config)
  - [.controllers](#strapi-controllers)
  - [.hook](#strapi-hook)
  - [.load()](#strapi-load)
  - [.log](#strapi-log)
  - [.middleware](#strapi-middleware)
  - [.models](#strapi-models)
  - [.plugins](#strapi-plugins)
  - [.query()](#strapi-query)
  - [.reload()](#strapi-reload)
  - [.router](#strapi-router)
  - [.server](#strapi-server)
  - [.services](#strapi-services)
  - [.start()](#strapi-start)
  - [.stop()](#strapi-stop)

## strapi.admin

该对象包含包含在 `strapi-admin` 包中的控制器、模型、服务和配置。

## strapi.api

对象中包含的 api `./api` 文件夹。通过使用 `strapi.api[:name]` ，您可以访问控制器、服务、模型定义以及。`./api/:name` 文件夹

## strapi.app

返回 Koa 实例。

## strapi.config

返回表示项目配置的对象。中的每个 JavaScript 或 JSON 文件 `./config` 文件夹将解析为 `strapi.config` 对象。

## strapi.controllers

返回项目中可用的控制器的对象。中的每个 JavaScript 文件 `./api/**/controllers` 文件夹将被解析为 `strapi.controllers` 对象。由于这个对象，您可以访问项目中任何地方的每个控制器的操作。

::: tip
这个对象不包括管理员控制器和插件控制器。
:::

## strapi.hook

返回项目中可用的钩子的对象。每个符合此模式的文件夹 `strapi-*` 并位于 `./node_modules` 或 `/hooks` 文件夹将挂载到 `strapi.hook` 对象中。

## strapi.load

返回一个解析应用程序的配置、钩子、中间件和 api 的函数。它也加载中间件和挂钩与以前加载的配置。此方法可用于更新通过 `strapi` 全局变量可用的引用，而无需重新启动服务器。但是，如果不重新启动服务器，就不会考虑新的配置。

## strapi.log

返回 Logger (Pino)实例。

## strapi.middleware

返回项目中可用的中间件的对象。文件夹中的每个文件夹 `./middlewares` 文件夹也将挂载到 `strapi.middleware` 对象中。

## strapi.models

返回项目中可用模型的对象。中的每个 JavaScript 或 JSON 文件 `./api/**/models` 文件夹将被解析为 `strapi.models` 对象。还有所有的 `strapi.models.**` 对象与 ORM (Mongoose，Bookshelf)返回的模型实例合并。它允许通过 `strapi.models.**` 调用 ORM 方法 (ex: `strapi.models.users.find()`).

## strapi.plugins

返回项目中可用的插件对象。每个插件对象包含相关的控制器、模型、服务和配置。

## strapi.query

这个工具函数允许绑定模型与特定于每个 ORM 的查询函数(例如: `mongoose` 或 `bookshelf`)。有关详细信息，请参阅[Queries section](/developer-docs/latest/development/backend-customization.md#queries) 。

## strapi.reload

返回一个重新加载整个应用程序的函数(带有停机时间)。

## strapi.router

返回 Router (Joi 路由器)实例。

## strapi.server

返回 [`http.Server`](https://nodejs.org/api/http.md#http_class_http_server) 实例。

## strapi.services

返回项目中可用的服务对象。中的每个 JavaScript 文件 `./api/**/services` 文件夹将被解析为 `strapi.services` 对象。

## strapi.start

返回一个加载配置、中间件和钩子的函数。然后，它执行引导程序文件，冻结全局变量并侦听配置的端口。

## strapi.stop

返回一个关闭服务器并销毁当前连接的函数。
