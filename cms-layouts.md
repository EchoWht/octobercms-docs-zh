# CMS 布局

- [介绍](#introduction)
- [占位符](#placeholders)
- [动态布局](#dynamic-layouts)
    - [布局执行的生命周期](#layout-life-cycle)

<a name="introduction"></a>
## 介绍


布局是定义页面框架，即在页面上重复的所有内容，例如页眉和页脚。布局通常包含HTML标签以及HEAD，TITLE和BODY标签。

布局模板位于主题目录的 **/layouts** 子目录中。布局模板文件应以后缀名 **htm** 结尾。在布局文件中，您应该使用`{％page％}`标签来输出页面内容。最简单的布局示例：

    <html>
        <body>
            {% page %}
        </body>
    </html>

To use a layout for a [page](cms-pages.md) the page should refer the layout file name (without extension) in the [Configuration](cms-themes.md#configuration-section) section. Remember that if you refer a layout from a [subdirectory](cms-themes.md#subdirectories) you should specify the subdirectory name. Example page template using the default.htm layout:
在[page页面](cms-pages.md)中使用布局时，page应该在[Configuration](cms-themes.md#configuration-section)部分中引用布局文件名称(不带扩展名)。请记住，如果从[子目录](cms-themes.md#subdirectories)引用布局，则应指定子目录名称。使用default.htm布局的示例页面模板：
    url = "/"
    layout = "default"
    ==
    <p>Hello, world!</p>

当请求示例中的页面时，其page内容将与layout布局合并，或者更准确地说 - 布局的`{％page％}`标签将替换为page页面中的内容。前面的示例将生成以下内容：

    <html>
        <body>
            <p>Hello, world!</p>
        </body>
    </html>

注意，您可以在布局中使用[partials](cms-partials.md)。这使您可以在不同布局之间共享公共标签元素。例如，您可以使用partials输出网站CSS和JavaScript链接。这种方法简化了资源管理 - 如果要添加JavaScript引用，则应修改单个partials而不是编辑所有layout布局。

[配置](cms-themes.md#configuration-section)部分对于layout布局是可选的。支持的配置参数是 **name** 和 **description**。参数是可选的，并在后端用户界面中使用。带有描述的示例布局模板：

    description = "简单的布局"
    ==
    <html>
        <body>
            {% page %}
        </body>
    </html>

<a name="placeholders"></a>
## 占位符

占位符允许page页面将内容注入布局。占位符在布局模板中通过`{％placeholder％}`标签定义。下一个示例显示了在HTML HEAD部分中定义的占位符 **head** 的布局模板。

    <html>
        <head>
            {% placeholder head %}
        </head>
        ...


pages可以使用`{％put％}`和`{％endput％}`标记向占位符注入内容。以下示例演示了一个简单的页面模板，该模板将CSS链接注入前一个示例中定义的占位符 **head**中。

    url = "/my-page"
    layout = "default"
    ==
    {% put head %}
        <link href="/themes/demo/assets/css/page.css" rel="stylesheet">
    {% endput %}

    <p>这里是页面的内容</p>

M有关占位符的更多信息可以在[标记指南](../markup/tag-placeholder)中找到。

<a name="dynamic-layouts"></a>
## 动态布局

布局像page页面一样，可以使用任何Twig功能。有关详细信息，请参阅[动态页面](cms-pages.md#dynamic-pages) 文档。

<a name="layout-life-cycle"></a>
### 布局执行的生命周期

在布局的[PHP部分](cms-themes.md#php-section)中，您可以定义以下函数来处理页面执行生命周期：`onInit`，`onStart`，`onBeforePageStart`和`onEnd`。


`onInit`方法会在当初始化所有组件并处理AJAX请求之前执行， `onStart`函数在页面处理开始时执行。`onBeforePageStart`函数在布局[组件](cms-components.md)运行之后但在执行页面的`onStart`函数之前执行。在呈现页面后执行`onEnd`函数。执行处理程序的顺序如下：

1. Layout `onInit()` function.
1. Page `onInit()` function.
1. Layout `onStart()` function.
1. Layout components `onRun()` method.
1. Layout `onBeforePageStart()` function.
1. Page `onStart()` function.
1. Page components `onRun()` method.
1. Page `onEnd()` function.
1. Layout `onEnd()` function.