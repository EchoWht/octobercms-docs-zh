# CMS 主题

- [介绍](#introduction)
- [目录结构](#directory-structure)
    - [子目录](#subdirectories)
- [模板结构](#template-structure)
    - [配置区块](#configuration-section)
    - [PHP代码区块](#php-section)
    - [Twig标记区块](#twig-section)
- [主题日志](#theme-logging)

<a name="introduction"></a>
## 介绍

主题定义了您的网站或 Web 应用程序的外观。October主题完全基于文件，您可以使用任意版本控制系统进行管理，如 Git。
本文简要介绍了 October 主题，您可以在pages(页面)，partials(部件)，layouts(布局)和content files(内容)相关页面查看更多细节。

对象 | 描述
------------- | -------------
[Pages](cms-pages.md) |网站的页面
[Partials](cms-partials.md) | 可以复用的 HTML标记.
[Layouts](cms-layouts.md) | 页面脚手架.
[Content files](cms-content.md) | 可以与页面或布局分开编辑的文本，HTML或[Markdown](http://daringfireball.net/projects/markdown/syntax)块。
**Asset files(资源文件夹)** |  图片, CSS 和 JavaScript 文件。

<a name="directory-structure"></a>
## 目录结构

您可以在下面看到一个示例主题目录结构。每个October主题用一个单独的目录表示，通常一个活动主题用于显示一个网站。此示例显示“网站”主题目录。

    themes/
      website/           <=== 主题起始文件夹
        pages/           <=== Pages 文件夹，放置主要页面
          home.htm
        layouts/         <=== Layouts 文件夹，放置布局文件
          default.htm
        partials/        <=== Partials 文件夹，可以复用的部件
          sidebar.htm
        content/         <=== Content 内容文件夹
          intro.htm
        assets/          <=== Assets 资源文件夹
          css/
            my-styles.css
          js/
          images/

> 使用主题可以通过 `config/cms.php`文件中的`activeTheme`参数或系统> CMS>前端主题后端页面上的主题选择器设置。使用Theme Selector设置的主题会覆盖`config/cms.php`文件中的值。

<a name="subdirectories"></a>
### 子目录

October支持pages, partials, layouts 和 content 文件的单级子目录( **assets** 目录可以具有任何结构)。简化组织大型网站的目录。在下面的示例目录结构中，您可以看到pages和partials目录包含 **blog** 子目录，而content目录包含 **home** 子目录。

    themes/
      website/
        pages/
          home.htm
          blog/                  <=== Subdirectory
            archive.htm
            category.htm
        partials/
          sidebar.htm
          blog/                  <=== Subdirectory
            category-list.htm
        content/
          footer-contacts.txt
          home/                  <=== Subdirectory
            intro.htm
        ...
要从子目录引用partials文件或content文件，请在模板名称前指定子目录名称。例如引用子目录中的partial：

    {% partial "blog/category-list" %}

> **注意:** 模板路径始终是绝对的。如果在某个partial中，您从同一子目录中引用另一个partial，则仍需要指定子目录名称。
<a name="template-structure"></a>
## 模板结构

Pages, partials 和 layout 模板最多可包含3个部分： **configuration**, **PHP 代码**, and **Twig 标记**.
每个部分通过 `==` 分割。
例如:

    url = "/blog"
    layout = "default"
    ==
    function onStart()
    {
        $this['posts'] = ...;
    }
    ==
    <h3>Blog archive</h3>
    {% for post in posts %}
        <h4>{{ post.title }}</h4>
        {{ post.content }}
    {% endfor %}

<a name="configuration-section"></a>
### Configuration(配置)部分

配置部分设置模板参数。支持的配置参数特定于不同的CMS模板，并在相应的文档文章中进行了描述。配置部分使用简单的[INI格式](http://en.wikipedia.org/wiki/INI_file)，其中字符串参数值包含在引号内。例如下面page模板的配置:

    url = "/blog"
    layout = "default"

    [component]
    parameter = "value"

<a name="php-section"></a>
### PHP 代码部分

每次渲染模板之前，PHP部分中的代码都会执行。对于所有CMS模板，PHP部分是可选的，其内容取决于定义它的模板类型。 PHP代码部分可以包含可选的开始和闭合PHP标记，以在文本编辑器中启用语法突出显示。 开始和闭合标签应始终在两行分隔符`==`的中间。

    url = "/blog"
    layout = "default"
    ==
    <?
    function onStart()
    {
        $this['posts'] = ...;
    }
    ?>
    ==
    <h3>Blog archive</h3>
    {% for post in posts %}
        <h4>{{ post.title }}</h4>
        {{ post.content }}
    {% endfor %}

在PHP部分中，您只能使用PHP`use`关键字定义函数并引用名称空间。 PHP部分中不允许其他PHP代码。这是因为在解析页面时PHP部分被转换为PHP类。例如：

    url = "/blog"
    layout = "default"
    ==
    <?
    use Acme\Blog\Classes\Post;

    function onStart()
    {
        $this['posts'] = Post::get();
    }
    ?>
    ==

作为设置变量的一般方法，您应该在`$this`上使用数组访问方法，尽管为了简单起见，您可以将**对象访问用作只读**，例如：

    // Write via array
    $this['foo'] = 'bar';

    // Read via array
    echo $this['foo'];

    // Read-only via object
    echo $this->foo;

<a name="twig-section"></a>
### Twig标记部分

The Twig section defines the markup to be rendered by the template. In the Twig section you can use functions, tags and filters [provided by October](../markup), all the [native Twig features](http://twig.sensiolabs.org/documentation), or those [provided by plugins](plugin-registration.md#extending-twig). The content of the Twig section depends on the template type (page, layout or partial). You will find more information about specific Twig objects further in the documentation.

Twig部分定义了模板要呈现的标记。在Twig部分，您可以使用函数，标签和过滤器[October提供](../markup)，所有[原生Twig功能](http://twig.sensiolabs.org/documentation)，或者[插件](plugin-registration.md#extending-twig)。 Twig部分的内容取决于模板类型(页面，布局或部分)。您可以在文档中找到有关特定Twig对象的更多信息。


<a name="theme-logging"></a>
## 主题日志

OctoberCMS附带了一个非常有用的功能，默认情况下禁用，称为主题日志。

由于布局和页面将大部分数据存储在平面文件中，因此您或您的客户可能会意外丢失内容。例如，切换页面的布局将修改页面的支架，因此导致数据丢失。

要启用主题日志记录，只需转到**设置->日志设置**并启用**日志主题更改**。现在记录所有更改。

可以在**设置->主题日志**中查看主题更改日志。每次更改都会概述已添加/删除的内容，以及之前和之后已更改文件的副本。如有必要，您可以使用此信息来确定适当的操作以还原更改。
