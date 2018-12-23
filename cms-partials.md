# CMS Partial(部件)

- [介绍](#introduction)
- [渲染Partial](#rendering-partials)
- [将变量传递给Partial](#partial-variables)
- [动态Partial](#dynamic-partials)
    - [Partial的执行生命周期](#partial-life-cycle)
    - [生命周期限制](#life-cycle-limitations)

<a name="introduction"></a>
## 介绍

Partials包含可重复使用的Twig标记块，可以在整个网站的任何地方使用。Partials对于在不同页面或布局上重复的页面元素非常有用。一个很好的局部示例是页脚，用于不同的[页面布局](cms-layouts.md). 。此外，[使用AJAX更新页面内容](ajax-update-partials.md)更需要partials。

Partial模板文件驻留在主题目录的 **/partials** 子目录中。 Partial 文件应是 **htm** 后后缀名。最简单的Partial示例:

    <p>This is a partial</p>

[配置](cms-themes.md#configuration-section) 部分对于Partial是可选的，，可以包含后端用户界面中显示的可选 **description** 。参数。下一个示例显示了Partial描述:

    description = "partial示例"
    ==
    <p>这是一个partial</p>

partial配置部分还可以包含组件定义。 [组件](cms-components.md)在另一篇文章中进行了解释。

<a name="rendering-partials"></a>
## 渲染partials

`{% partial "partial-name" %}` Twig 标签渲染partial。 T标签有一个必需参数 - 没有扩展名的partial文件名。请记住，如果从[子目录](cms-themes.md#subdirectories)中引用partial，则应指定子目录名称。  `{% partial %}`标签可以在页面，布局或其他部分内使用。引用partial页面的示例：

    <div class="sidebar">
        {% partial "sidebar-contacts" %}
    </div>

<a name="partial-variables"></a>
## 将变量传递给partials

我们通常需要将变量从外部代码传递给partial变量。这使得partial更有用。例如，您可以使用partial展示博客帖子列表。如果您可以将帖子集合传递给partial，则可以在博客存档页面，博客类别页面等上使用相同的partial。您可以通过在`{%partial%}`标记中的partial名称后指定变量来将变量传递给partials：
  
    <div class="sidebar">
        {% partial "blog-posts" posts=posts %}
    </div>

您还可以新建变量在partial中使用：

    <div class="sidebar">
        {% partial "sidebar-contacts" city="Vancouver" country="Canada" %}
    </div>

在partial内部，可以像任何其他标记变量一样使用：

    <p>Country: {{ country }}, city: {{ city }}.</p>


<a name="dynamic-partials"></a>
## 动态partials

Partials像页面一样，也可以使用任何Twig功能。有关详细信息，请参阅[动态页面](cms-pages.md#dynamic-pages) 文档。

<a name="partial-life-cycle"></a>
### Partial执行生命周期

可以在partials的PHP部分中定义特殊函数：`onStart`和`onEnd`。 `onStart`函数在部分渲染之前和partials[组件](cms-components.md)执行之前执行。 `onEnd`函数在渲染之前和partials[组件](cms-components.md) 执行之后执行。在onStart和onEnd函数中，您可以将变量注入Twig环境。使用`array notation`将变量传递给页面：

    ==
    function onStart()
    {
        $this['hello'] = "Hello world!";
    }
    ==
    <h3>{{ hello }}</h3>

October提供的模板语言在[标记指南](../markup)中有具体描述。执行处理程序的整体顺序在[动态布局](cms-layouts.md#dynamic-layouts)的文章中有提到.

<a name="life-cycle-limitations"></a>
### 生命周期限制

由于它们实例化较晚，因此在渲染页面期间，partials限制的生命周期会受到一些限制。它们不遵循标准执行过程，如[布局执行生命周期](cms-layouts.md#dynamic-layouts)中所述。应注意以下限制：

1. AJAX事件未注册，将无法正常运行。
1. 生命周期函数不能返回任何值。
1. 在渲染partials时将发生常规的POST表单处理。

通常，partials中的组件使用是为基本组件设计的，这些组件在没有太多处理的情况下呈现简单标记，例如 *Like* 或 *Tweet* 按钮。