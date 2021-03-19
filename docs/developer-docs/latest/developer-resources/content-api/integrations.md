# 集成

Strapi 生成了一个 API 让你访问你的内容。但是如何将 React，Ruby，Gatsby 应用程序连接到它呢？了解什么是 API 很重要。

## 什么是 API?

API 是应用程序编程接口(applicationprogramminginterface)的首字母缩写，它是一种允许两个应用程序相互通信的软件中介。如果您想要将 React 应用程序与 Strapi 连接起来，我们说 React 是客户机，而 Strapi 是系统。实际上，React 将通过 HTTP 请求与 Strapi 进行通信。然后 Strapi 会回复你的客户。

如果您的 Strapi 应用程序包含餐馆，并且您希望在 React 应用程序中列出这些餐馆，那么您所需要做的就是向 Strapi 发出 HTTP 请求，该请求将小心翼翼地给您一个包含您的餐馆的响应。

[API Endpoints](/developer-docs/latest/developer-resources/content-api/content-api.md#api-endpoints) 将为您提供与 Strapi API 交互的所有键。

## Get started

今天，任何编程语言都有一个 HTTP 客户机，它允许您执行对 API 的请求，因此可以与 API 进行交互。Javascript 有 [Axios](https://github.com/axios/axios) ，[Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) ，Ruby 有 [Faraday](https://github.com/lostisland/faraday) ，[HTTParty](https://github.com/jnunemaker/httparty) ，[Requests](https://requests.readthedocs.io/en/master/) 等等。

将 Strapi 与多种框架、前端或后端编程语言集成在下面。

<IntegrationLinks>
</IntegrationLinks>
