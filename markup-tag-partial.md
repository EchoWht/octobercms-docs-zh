# {% partial %}

`{%partial%}`标签将解析[CMS partial](cms-partials.md)并在页面上呈现partial内容。 要显示名为**footer.htm**的partial，只需在引用为字符串的`partial`标记后面传递名称即可。

    {% partial "footer" %}

可以以相同的方式呈现子目录中的partial内容。

    {% partial "sidebar/menu" %}

> **注意**: [主题文档](cms-themes.md#subdirectories) 有关于子目录用法的更多详细信息。

partial名称也可以是变量：

    {% set tabName = "profile" %}
    {% partial tabName %}

<a name="variables"></a>
## 变量

您可以通过在partial名称后指定变量来将变量传递给partial：

    {% partial "blog-posts" posts=posts %}

您还可以分配新变量以在partial中使用：

    {% partial "location" city="Vancouver" country="Canada" %}

在partial内部，可以像任何其他标记变量一样访问变量：

    <p>Country: {{ country }}, city: {{ city }}.</p>

<a name="checking-partial-exits"></a>
## 检查partial存在

在任何模板中，您可以使用`partial()`函数检查是否存在partial内容。 这使您可以根据partial是否存在生成不同的标记。 例：

    {% set cardPartial = 'my-cards/' ~ cardCode %}

    {% if partial(cardPartial) %}
        {% partial cardPartial %}
    {% else %}
        <p>Card not found!</p>
    {% endif %}
