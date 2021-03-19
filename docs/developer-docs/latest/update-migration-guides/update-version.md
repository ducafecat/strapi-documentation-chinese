# 更新 Strapi 版本

有了这个指南，你就会知道如何将你的应用程序升级到 Strapi 的最新版本。

::: tip 注意
当一个新版本的 Strapi 是可用的，您将得到通知，在您的终端，并在管理面板。还要注意的是，你可以在管理面板的 **Settings > Application** 部分查看更多信息:

- 你现在使用的 Strapi 和 Node 的版本.
- 如果相关，我们建议您升级到的版本.

:::

::: warning CAUTION
在您开始之前，请确保您的服务器没有运行，直到指南结束！
:::

## 升级你的依赖关系

从升级 package.json 中的所有 Strapi 包开始，例如从 `3.2.4` 升级到 `3.2.5` :

:::: tabs

::: tab 3.2.4

```json
{
  //...
  "dependencies": {
    "strapi": "3.2.4",
    "strapi-admin": "3.2.4",
    "strapi-connector-bookshelf": "3.2.4",
    "strapi-plugin-content-manager": "3.2.4",
    "strapi-plugin-content-type-builder": "3.2.4",
    "strapi-plugin-email": "3.2.4",
    "strapi-plugin-graphql": "3.2.4",
    "strapi-plugin-upload": "3.2.4",
    "strapi-plugin-users-permissions": "3.2.4",
    "strapi-utils": "3.2.4"
    //...
  }
}
```

:::

::: tab 3.2.5

```json
{
  //...
  "dependencies": {
    "strapi": "3.2.5",
    "strapi-admin": "3.2.5",
    "strapi-connector-bookshelf": "3.2.5",
    "strapi-plugin-content-manager": "3.2.5",
    "strapi-plugin-content-type-builder": "3.2.5",
    "strapi-plugin-email": "3.2.5",
    "strapi-plugin-graphql": "3.2.5",
    "strapi-plugin-upload": "3.2.5",
    "strapi-plugin-users-permissions": "3.2.5",
    "strapi-utils": "3.2.5"
    //...
  }
}
```

:::

::::

在编辑文件后，运行 `yarn install` 或 `npm install` 来安装指定的版本。

::: tip
如果这个操作不起作用，你可能应该移除你的 `yarn.lock` 或 `package-lock.json` 。如果仍然不能工作，请在删除文件夹 `node_modules` 之后再试一次。
:::

## 重建您的管理面板

新版本可能会对需要重新生成的管理面板进行更改。使用以下命令之一重新构建管理面板:

```bash
yarn build --clean
# or
npm run build -- --clean
```

## 扩展

如果使用 [extensions](/developer-docs/latest/development/plugin-customization.md) 创建自定义代码或修改现有代码，则需要更新代码并将版本与存储库上的新更改进行比较。不更新你的扩展会以我们无法预料的方式 **break your app** 。

## 迁移指南

有时候，Strapi 会引入一些 **breaking changes** ，这些变化需要的不仅仅是前面的步骤。这就是设置 [Migration guides](/developer-docs/latest/update-migration-guides/migration-guides.md) 页面的原因。只需确保在更新版本时，迁移指南是否存在。

## 开始你的应用程序

如果你已经遵循了上述的信息，你可以以下列方式开始你的申请:

```bash
yarn develop
# or
npm run develop
```

🎉 恭喜你，你的应用程序已经迁移了！
