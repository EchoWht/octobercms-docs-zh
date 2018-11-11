# {% content %}

`{％content％}`标签将在页面上显示[CMS内容块](../cms/content)。 要显示名为** contacts.htm **的内容块，请在引用为字符串的`content`标记后传递文件名。

    {% content "contacts.htm" %}

子目录中的内容块可以以相同的方式呈现。

    {% content "sidebar/content.htm" %}

> **注意**: [主题文档](../cms/themes#subdirectories)有关于子目录用法的更多详细信息。

内容块可以呈现为纯文本：

    {% content "readme.txt" %}

您还可以使用Markdown语法：

    {% content "changelog.md" %}

内容块也可以与[布局占位符](../cms/layouts#placeholders)结合使用：

    {% put sidebar %}
        {% content 'sidebar-content.htm' %}
    {% endput %}

<a name="variables"></a>
## 变量

您可以通过在文件名后指定变量来将变量传递给内容块：

    {% content "welcome.htm" name=user.name %}

您还可以分配新变量以在内容中使用：

    {% content "location.htm" city="Vancouver" country="Canada" %}

在内容中，可以使用基本语法使用单数*花括号*来访问变量：

    <p>Country: {country}, city: {city}.</p>

您还可以将变量集合作为简单数组传递：

    {% content "welcome.htm" likes=[
        {name:'Dogs'},
        {name:'Fishing'},
        {name:'Golf'}
    ] %}

通过使用一组开始和结束括号来访问变量集合：

    <ul>
        {likes}
            <li>{name}</li>
        {/likes}
    </ul>

> **注意**: 内容块中不支持Twig语法，请考虑使用[CMS partial](../cms/partials)。
