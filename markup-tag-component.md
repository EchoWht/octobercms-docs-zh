# {% component %}

`{％component％}`标签将解析[CMS组件](cms-partials.md)的默认标签内容并将其显示在页面上。 并非所有组件都提供默认标签，请查阅组件的相关文档。

    {% component "blogPosts" %}

这将使组件部分具有固定名称**default.htm**，并且基本上是以下的别名：

    {% partial "blogPosts::default" %}

<a name="variables"></a>
## 变量

一些组件在渲染时支持[传递变量](cms-components.md#component-variables)

    {% component "blogPosts" postsPerPage="5" %}

<a name="customizing-components"></a>
## 自定义组件

在大多数情况下，不需要`{％component％}`标签，并且标签是作为组件API的使用示例提供的。 组件旨在定制，这可以通过两种方式完成：

1. [将默认标签移动到部分partial](cms-components.md#moving-default-markup)
1. [覆盖组件partials](cms-components.md#overriding-partials)
