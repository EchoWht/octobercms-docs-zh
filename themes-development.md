# 主题开发

- [主题信息文件](#theme-information)
- [版本文件](#version-file)
- [主题预览图片](#preview-image)
- [主题定制](#customization)
- [主题依赖](#dependencies)

主题目录可以包括 **theme.yaml**，**version.yaml**和**assets/images/theme-preview.png**文件。 这些文件对于本地开发是可选的，但对于在OctoberCMS Marketplace上发布的主题是必需的。

<a name="theme-information"></a>
## 主题信息文件

主题信息文件**theme.yaml**包含主题描述，作者姓名，作者网站的URL和一些其他信息。 该文件应放在主题根目录中：

    themes/
      demo/
        theme.yaml    <=== 主题信息文件

**theme.yaml**文件支持以下字段：

字段 | 描述
------------- | -------------
**name** | 指定主题名称，必需。
**author** | 指定作者姓名，必填。
**homepage** | 指定作者网站URL。
**description** | 主题描述，必填。
**previewImage** | 自定义预览图像，相对于主题目录的路径，例如：`assets/images/preview.png`，可选。
**code** | 主题代码，可选。 该值在OctoberCMS市场上用于初始化主题代码值。 如果未提供主题代码，则主题目录名称将用作代码。 从Marketplace安装主题时，代码将用作新的主题目录名称。
**form** | 表单字段定义文件的配置数组或引用，用于[主题自定义](#customizeization)，可选。
**require** | 用于[主题依赖项](#dependencies)的插件名称数组，可选。

主题信息文件的示例：

    name: "OctoberCMS Demo"
    description: "Demonstrates the basic concepts of the front-end theming."
    author: "OctoberCMS"
    homepage: "http://octobercms.com"
    code: "demo"

<a name="version-file"></a>
## 版本文件

主题版本文件**version.yaml**定义当前主题版本和更改日志。 该文件应放在主题根目录中：

    themes/
      demo/
        ...
        version.yaml    <=== 版本文件

文件格式如下：

    1.0.1: Theme initialization
    1.0.2: Added more features
    1.0.3: Some features are removed

<a name="preview-image"></a>
## 主题预览图片

主题预览图像用于后端主题选择器。 图像文件**theme-preview.png**应放在主题的**assets/images**目录中：

    themes/
      demo/
        assets/
          images/
            theme-preview.png    <=== 预览图片

图像宽度应至少为600px。 理想的宽高比为1.5，例如600x400px。

<a name="customization"></a>
## 主题定制

主题可以通过在主题信息文件中定义`form`键来支持配置值。 此键应包含配置数组或对表单字段定义文件的引用，有关详细信息，请参阅[表单字段](backend-forms.md#form-fields )。

以下是如何定义名为**site_name**的网站名称配置字段的示例：

    name: My Theme
    # [...]

    form:
        fields:
            site_name:
                label: Site name
                comment: The website name as it should appear on the front-end
                default: My Amazing Site!

然后可以使用名为`this.theme`的[默认页面变量](../cms/markup#default-variables)在任何Theme模板中访问该值。

    <h1>Welcome to {{ this.theme.site_name }}!</h1>

您还可以在单独的文件中定义配置，其中路径相对于主题。 以下定义将从主题内的文件**config/fields.yaml**中获取表单字段。

    name: My Theme
    # [...]

    form: config/fields.yaml

<a name="combiner-vars"></a>
### 组合变量

使用`|theme` [过滤器和组合器](../markup/filter-theme)组合的资源可以将值传递给支持过滤器，例如LESS过滤器。 在定义表单字段时，只需指定`assetVar`选项，该值应包含所需的变量名称。

    form:
        fields:
            # [...]

            link_color:
                label: Link color
                type: colorpicker
                assetVar: 'link-color'

在上面的例子中，选择的颜色值将在less文件中作为`@link-color`使用。 假设我们有以下样式表参考：

    <link href="{{ ['assets/less/theme.less']|theme }}" rel="stylesheet">

使用**themes/yourtheme/assets/less/theme.less**中的一些示例内容：

    a { color: @link-color }

<a name="dependencies"></a>
## 主题依赖

主题可以通过在[主题信息文件](#theme-information)中定义**require**选项来依赖于插件，该选项应该提供一系列被认为是需求的插件名称。 依赖于**Acme.Blog**和**Acme.User**的主题可以像这样定义这个要求：

    name: "OctoberCMS Demo"
    # [...]

    require:
        - Acme.User
        - Acme.Blog

首次安装主题时，系统将尝试同时安装所需的插件。
