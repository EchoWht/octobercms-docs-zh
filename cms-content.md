# CMS内容

- [介绍](#introduction)
- [渲染内容](#rendering-content-blocks)
- [将变量传递给内容模块](#content-variables)
    - [全局变量](#content-global-variables)

内容块是文本，HTML或[Markdown](http://daringfireball.net/projects/markdown/syntax，可以与页面或布局分开编辑。它们旨在仅保存静态内容并支持基本模板变量。而[Partials](partials)更灵活，应该用于生成动态内容。

<a name="introduction"></a>
## 介绍

内容块文件存放在主题目录的 **/content** 子目录中。内容文件支持以下扩展：

后缀名 | 描述
------------- | -------------
**htm** | 可以使用html标签
**txt** | 纯文本
**md** | 支持markdown语法


文件后缀名会影响内容块在后端用户界面（使用WYSIWYG编辑器或纯文本编辑器）中的显示方式以及块在网站上的呈现方式。 Markdown块在显示之前会转换为HTML。

<a name="rendering-content-blocks"></a>
## 渲染内容

使用`{% content 'file.htm' %}` 标签在[page](pages), [partial](partials) 或 [layout](layouts)中渲染内容块。 例如：

    url = "/contacts"
    ==
    <div class="contacts">
        {% content 'contacts.htm' %}
    </div>

<a name="content-variables"></a>
## 将变量传递给内容模块

有时您可能需要将变量从外部代码传递到内容块。虽然内容块不支持使用Twig标记，但它们确实支持使用具有基本语法的变量。您可以通过在`{％content％}`标记中的内容块名称之后指定变量来将变量传递给内容块：

    {% content 'welcome.htm' name='John' %}

在内容块中，可以使用单个*大括号*使用变量：

    <h1>欢迎{name}~</h1>

可以在[标记指南](../markup/tag-content)中找到更多信息。

<a name="content-global-variables"></a>
### 全局变量

您可以使用`View :: share`方法注册全局可用于所有内容块的变量。

    View::share('site_name', 'OctoberCMS');

可以在[插件注册文件](../plugin/registration)的寄存器或引导方法内调用此代码。使用上面的示例，变量`{site_name}`将在所有内容块中可用。