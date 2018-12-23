# {% styles %}

`{%styles%}`标签呈现CSS链接到应用程序注入的样式表文件。 标签通常在页面或布局的HEAD部分中定义：

    <head>
        ...
        {% styles %}
    </head>

> **注意**: 此标记应仅在给定的页面循环中出现一次，以防止重复引用。

## 注入样式

StyleSheet文件的链接可以通过[components](plugin-components.md#component-assets) 或[pagesmaticmatically](cms-pages.md#injecting-assets)注入PHP。

    function onStart()
    {
        $this->addCss('assets/css/hello.css');
    }

您还可以使用**styles** anonymous [placeholder](cms-layouts.md#placeholders)将原始标记注入`{%styles%}`标记。 在页面或布局中使用`{%put%}`标记将内容添加到占位符：

    {% put styles %}
        <link href="/themes/demo/assets/css/page.css" rel="stylesheet" />
    {% endput %}
