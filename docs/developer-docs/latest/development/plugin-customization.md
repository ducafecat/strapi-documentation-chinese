# Strapi 插件

在 Strapi，你可以在你的 `node_modules` 中安装插件。这允许简单的更新和尊重最佳实践。要自定义那些已安装的插件，您可以在 `/extensions` 目录中工作。它包含所有插件的可定制文件。

一些插件会在这些文件夹中创建文件，这样你就可以修改它们了。您还可以手动创建某些文件以添加某些自定义配置。

扩展文件夹结构:

- `extensions/`
  - `**`: 插件 Id
    - `admin`: 你可以通过创建一个同名的文件来扩展插件的管理，这样做会覆盖原来的文件.
    - `config`: 您可以通过添加带有自定义配置的 settings.json 文件来扩展插件的配置.
    - `models`: 包含您已经覆盖的插件模型(例如，当您向用户模型添加关系时).
    - `controllers`: 可以通过创建名称相同的控制器并覆盖某些方法来扩展插件的控制器.
    - `services`: 你可以通过创建相同名称的服务和覆盖某些方法来扩展插件的服务.

::: warning
在使用 **extensions** 时，每当升级 strapi 版本时，都需要更新代码。不更新你的 **extensions** 并将其与存储库中的新变化进行比较，可能会以我们在 [迁移指南](/developer-docs/latest/update-migration-guides/migration-guides.md) 中无法预料的方式破坏你的应用程序。
:::

<PluginsLinks>
</PluginsLinks>
