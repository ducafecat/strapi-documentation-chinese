# Installing using Platform.sh

[Platform.sh](https://platform.sh/) gives you an easy way to get started and deploy your Strapi application.

You can find the template [source code](https://github.com/platformsh-templates/strapi#customizations) on Platform.sh GitHub 了解详情.

[[toc]]

## Step 1: Create a Platform.sh account

You must have a [Platform.sh](https://console.platform.sh/) account before doing these steps.

## Step 2: Create a project

You can use the **One-Click** button or follow these steps.

<a href="https://console.platform.sh/projects/create-project?template=https://raw.githubusercontent.com/platformsh/template-builder/master/templates/strapi/.platform.template.yaml&utm_content=strapi&utm_source=github&utm_medium=button&utm_campaign=deploy_on_platform">
    <img src="https://platform.sh/images/deploy/lg-blue.svg" alt="Deploy on Platform.sh" width="180px" />
</a>

- 点击 `+ Add project` button
- Select `Use a template` option and click `Next`
- 填充 your `Project name` and `Region` then click `Next`

**Choose Strapi template.**

Search for the Strapi template using the search bar.

- 点击 the search bar
- 填充 `strapi`
- 选择 Strapi template
- Click `Next`

::: tip INFO
After few second a Strapi application will be setup with a PostgreSQL database.
:::

## Step 3: Visit your app

Now to visit your application.

- 点击 `Settings`
- 然后 `Domains` 在左边的菜单里

You will see the Platform.sh domain name that is used for your app.
