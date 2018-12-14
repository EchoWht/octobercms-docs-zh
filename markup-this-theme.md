# this.theme

You can access the current theme object via `this.theme` and it returns the object `Cms\Classes\Theme`, a reference to the [theme customization object](themes-development.md#customization).

## 属性

`this.theme`将提供对任何主题定制定义的表单字段值的直接访问。 它本身也具有以下属性。

### id

将主题目录名称转换为CSS友好标识符。

    <body class="theme-{{ this.theme.id }}">

如果主题目录是**website**，则会生成“theme-website”的类名。

### config

包含`theme.yaml`文件中找到的所有主题配置值的数组。

    <meta name="description" content="{{ this.theme.config.description }}">
