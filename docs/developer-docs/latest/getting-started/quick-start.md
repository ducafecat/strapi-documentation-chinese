# 快速入门指南

准备好让 Strapi 在**3 分钟内**启动并运行 🚀.

<div class="video-container">
  <iframe width="853" height="480" src="https://www.youtube.com/embed/zd0_S_FPzKg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

请参阅以下步骤，了解详情。这个快速启动非常接近 [FoodAdvisor](https://github.com/strapi/foodadvisor) 应用程序。

(在继续之前，请确保您的机器上正确安装了 [Node.js 和 npm 正确安装](/developer-docs/latest/setup-deployment-guides/installation/cli.md#step-1-make-sure-requirements-are-met) 。您可以 [安装 Yarn v1.2.0+ 包](https://yarnpkg.com/en/) .)

::: warning 警告

在 3.1.5 中，**Roles & Permissions** 部分已经迁移到了 **Settings** 部分。
:::

## 1. 安装 Strapi 并创建一个新项目

:::: tabs

::: tab yarn

```bash
yarn create strapi-app my-project --quickstart
```

:::

::: tab npx

```bash
npx create-strapi-app my-project --quickstart
```

:::

::::

## 2. 创建一个管理员用户

打开 [**http://localhost:1337/admin**](http://localhost:1337/admin).

完成初始化，并创建第一个 **Administrator** 用户.

## 3. 创建餐厅内容类型

打开 [**PLUGINS** - **Content Type Builder**](http://localhost:1337/admin/plugins/content-type-builder), 左侧菜单.

- 点击 **"+ Create new collection type"** link
- Type `restaurant`, and click `Continue`
- 点击 **Text** field
- Type `name` in the **Name** field
- Click over to the **ADVANCED SETTINGS** tab, and check the `Required field` and the `Unique field`
- 点击 **"+ Add another Field"** button
- 点击 **Rich Text** field
- Type `description` under the **BASE SETTINGS** tab, in the **Name** field
- Click `Finish`
- 点击 **Save** button and wait for Strapi to restart

## 4. 创建分类内容类型

返回到 [**PLUGINS** - **Content Type Builder**](http://localhost:1337/admin/plugins/content-type-builder), 左侧菜单.

- 点击 **"+ Create new collection type"** link
- Type `category`, and click `Continue`
- 点击 **Text** field
- Type `name` under the **BASE SETTINGS** tab, in the **Name** field
- Click over to the **ADVANCED SETTINGS** tab, and check the `Required field` and the `Unique field`
- 点击 **"+ Add another field"** button
- 点击 **Relation** field
- On the right side, click the **Category** dropdown and select, `Restaurant`
- In the center, select the icon that represents `many-to-many`. The text should read, `Categories has and belongs to many Restaurants`
- Click `Finish`
- 点击 **Save** button and wait for Strapi to restart

## 5. 在“餐厅”内容类型中添加内容

打开 [**COLLECTION TYPES** - **Restaurants**](http://localhost:1337/admin/plugins/content-manager/collectionType/application::restaurant.restaurant), 左侧菜单.

- 点击 **+ Add New Restaurants** 按钮 Type `Biscotte Restaurant` in the **Name** 字段 Type `Welcome to Biscotte restaurant! Restaurant Biscotte offers a cuisine based on fresh, quality products, often local, organic when possible, and always produced by passionate producers.` into the **Description** 字段
- Click **Save**.

如果返回 [**COLLECTION TYPES** - **Restaurants**](http://localhost:1337/admin/plugins/content-manager/collectionType/application::restaurant.restaurant) ，您将在条目中看到您的餐厅列表。

## 6. 在“类别”内容类型中添加类别

打开 [**COLLECTION TYPES** - **Categories**](http://localhost:1337/admin/plugins/content-manager/collectionType/application::category.category).

- 点击 **+ Add New Categories** 按钮 Type `French Food` in the **Name** 字段 Select `Biscotte Restaurant`, on the right in the **Restaurant (0)** dropdown.
- Click **Save**.

返回到 [**COLLECTION TYPES** - **Categories**](http://localhost:1337/admin/plugins/content-manager/collectionType/application::category.category)。你会在参赛作品中看到法国食品的分类。

- 点击 **+ Add New Categories** 按钮 Type `Brunch` in the **Category** 字段 **DO NOT ADD** `Biscotte Restaurant` to the `Restaurants` dropdown on the right.
- Click **Save**.

您将在条目中看到 **早午餐** 类别。

返回到 [**COLLECTION TYPES** - **Restaurants**](http://localhost:1337/admin/plugins/content-manager/collectionType/application::restaurant.restaurant).

- 点击 `Biscotte Restaurant`.
- On the right, under **Categories(1)**, `select` the `Add an item...`, and add **Brunch** as a category for this restaurant, and click the **Save** 按钮

现在您已经看到了使用关系字段类型在内容类型之间添加和连接关系的**两种不同方法**。

## 7. 设置角色和权限

打开 [**GENERAL** > **Settings** > **USERS & PERMISSIONS PLUGIN** > **Roles**](http://localhost:1337/admin/settings/users-permissions/roles).

- 点击 **Public** Role.
- Scroll down under **Permissions**, open the **Application** tab and find **Restaurant**. 点击 checkbox next to **find** and **findone**.
- Repeat and find **Category**. 点击 checkbox next to **find** and **findone**.
- Click **Save**.

## 8. 发布内容

默认情况下，您创建的任何内容都保存为草稿:

返回到 [**COLLECTION TYPES** - **Categories**](http://localhost:1337/admin/plugins/content-manager/collectionType/application::category.category)

- 点击 **Draft** button on the **Brunch** category.
- Click **Publish** 按钮
- In the **Please confirm** dialog, click **Yes, publish** 按钮
- Repeat for the **French food** category and **Biscotte Restaurant**.

## 9. 使用内容类型的 API

我们到了! **餐馆的名单**可以在 [`http://localhost:1337/restaurants`](http://localhost:1337/restaurants) 餐厅找到。

::: tip 祝贺你
👏 恭喜你，你已经完成了 **Strapi 快速入门指南** 。下一步去哪里？

- 使用您最喜欢的框架、前端或后端编程语言来使用您的 API.
- 学习如何使用 Strapi with React ([Gatsby](https://strapi.io/blog/build-a-static-blog-with-gatsby-and-strapi) or [Next.js](https://strapi.io/blog/nextjs-react-hooks-strapi-food-app-1)) or Vue.js ([Nuxt.js](https://strapi.io/blog/cooking-a-deliveroo-clone-with-nuxt-vue-js-graphql-strapi-and-stripe-setup-part-1-7/)).
- 你可以看一下 [教程](https://strapi.io/tutorials) 深入学习 Strapi.
- 寻求帮助 [our community forum](https://forum.strapi.io).
- 阅读 [source code](https://github.com/strapi/strapi), [contribute](https://github.com/strapi/strapi/blob/master/CONTRIBUTING.md) or [give a star](https://github.com/strapi/strapi) on GitHub.
- 跟我们走 [Twitter](https://twitter.com/strapijs) 来获得最新的消息.
- [加入 vibrant 和 Strapi 社区](https://slack.strapi.io) 在 Slack.
  :::

### 使用以下技术之一来使用你的 API:

  <IntegrationLinks>
  </IntegrationLinks>
