# CMS Pages

- [介绍](#introduction)
- [页面配置](#configuration)
    - [URL语法](#url-syntax)
- [动态页面](#dynamic-pages)
    - [页面执行的生命周期](#page-life-cycle)
    - [发送自定义响应](#life-cycle-response)
    - [处理表单](#handling-forms)
- [404 页面](#404-page)
- [错误 页面](#error-page)
- [页面变量](#page-variables)
- [以编程方式注入页面静态资源](#injecting-assets)

<a name="introduction"></a>
## 介绍


所有网站都有网页。October的页面用页面模板表示。页面模板文件位于主题目录的 **/pages** 子目录中。页面文件名称不会影响路由，但最好根据页面功能命名页面。文件以 **htm** 作为扩展名。页面需要[Configuration](cms-themes.md#configuration-section)和[Twig](cms-themes.md#twig-section)模板部分，但[PHP部分](cms-themes.md#php-section)是可选的。您可以在下面看到最简单的主页示例。
    url = "/"
    ==
    <h1>Hello, world!</h1>

<a name="configuration"></a>
## 页面配置

页面配置在页面模板文件的[配置部分](cms-themes.md#configuration-section)中定义。页面配置定义了路由和呈现页面和页面[组件](cms-components.md)所需的页面参数，这在另一篇文章中有解释。页面支持以下配置参数：

参数 | 描述
------------- | -------------
**url** | 页面URL，必需。 URL语法如下所述。
**title** | 页面标题，必需。
**layout** | 页面[layout](cms-layouts.md)，可选。如果指定，则应包含布局文件的名称，不带扩展名，例如：`default`。
**description** | 后端接口的页面描述，可选。

<a name="url-syntax"></a>
### URL语法

页面URL通过**url**配置参数来定义。 URL应以正斜杠字符开头，并且可以包含参数。没有参数的URL是固定和严格的。在以下示例中，页面URL为`/blog`。

    url = "/blog"

带参数的URL更灵活。对于任何地址，例如`/blog/post/something`，将显示以下示例中定义的URL模式的页面。 URL参数可以通过October组件或页面[PHP代码](cms-themes.md#php-section)部分访问。

    url = "/blog/post/:post_id"

您可以从页面PHP部分访问URL参数的方法(有关详细信息，请参阅[动态页面](#dynamic-pages)部分)例如：

    url = "/blog/post/:post_id"
    ==
    function onStart()
    {
        $post_id = $this->param('post_id');
    }
    ==

参数名称应与PHP变量名称兼容。要使参数，请在其名称后添加问号：

    url = "/blog/post/:post_id?"

URL中间的参数是必填的。在下一个示例中，`:post_id`参数被标记为可选，但要按需进行处理

    url = "/blog/:post_id?/comments"

可选参数可以具有默认值，如果URL中未显示实际参数值，则默认值将用作回退值。默认值不能包含任何星号，管道符号或问号。在**问号**之后指定默认值。在下一个例子中，`category_id`参数对于URL`/blog/category`是'10`。

    url = "/blog/category/:category_id?10"

您还可以使用正则表达式来验证参数。要添加验证表达式，请在参数名称(或问号)后面添加管道符号并指定表达式。表达式中不允许使用正斜杠符号。例子：

    url = "/blog/:post_id|^[0-9]+$/comments" - 对应例子 /blog/10/comments
    ...
    url = "/blog/:post_id|^[0-9]+$" - 对应例子 /blog/3
    ...
    url = "/blog/:post_name?|^[a-z0-9\-]+$" - 对应例子 /blog/my-blog-post

通过在参数后放置**星号**，可以使用特殊的*通配符*参数。与常规参数不同，通配符参数可以匹配一个或多个URL段。 URL只能包含单个通配符参数，不能使用正则表达式，或者后跟可选参数。

    url = "/blog/:category*/:slug"

例如像这个URL `/color/:color/make/:make*/edit` 例如 `/color/brown/make/volkswagen/beetle/retro/edit` 并获取以下参数值:

<div class="content-list" markdown="1">
- color: `brown`
- make: `volkswagen/beetle/retro`
</div>

> **注意:** 子目录不影响页面URL  -  URL仅使用**url**参数定义。

<a name="dynamic-pages"></a>
## 动态页面

在页面模板的[Twig部分](cms-themes.md#twig-section)内，您可以使用任何[October提供的函数，过滤器和标签](../markup))。任何动态页面都需要**变量**。October页面变量可以通过页面或布局[PHP部分](cms-themes.md#php-section)或[Components](cms-components.md)来准备。在本文中，我们将介绍如何在PHP部分中准备变量。

<a name="page-life-cycle"></a>
### 页面执行的生命周期

可以在页面和布局的PHP部分中定义特殊函数：`onInit`，`onStart`和`onEnd`。当初始化所有组件并处理AJAX请求之前，执行`onInit`函数。 `onStart`函数在页面执行开始时执行。 `onEnd`函数在呈现页面之前和页面[Components](cms-components.md)执行之后执行。在onStart和onEnd函数中，您可以将变量注入Twig环境。使用`array notation`将变量传递给页面：

    url = "/"
    ==
    function onStart()
    {
        $this['hello'] = "Hello world!";
    }
    ==
    <h3>{{ hello }}</h3>

下一个例子更复杂。它展示了如何从数据库加载博客文章集合并在页面上显示(Acme\Blog插件是虚构的)

    url = "/blog"
    ==
    use Acme\Blog\Classes\Post;

    function onStart()
    {
      $this['posts'] = Post::orderBy('created_at', 'desc')->get();
    }
    ==
    <h2>Latest posts</h2>
    <ul>
        {% for post in posts %}
            <h3>{{ post.title }}</h3>
            {{ post.content }}
        {% endfor %}
    </ul>

October提供的默认变量和Twig扩展名在[标记指南](../markup)中描述。执行处理程序的整体顺序在[动态布局](cms-layouts.md#dynamic-layouts)文章中有所描述。

<a name="life-cycle-response"></a>
### 发送自定义响应

执行生命周期中定义的所有方法都能够暂停进程并返回响应。只需返回生命周期功能的响应即可。下面的示例不会加载任何页面内容并将字符串*Hello world*返回给浏览器：

    function onStart()
    {
        return 'Hello world!';
    }

以下是一个非常有用的示例，加载页面前`Redirect`触发重定向

    public function onStart()
    {
        return Redirect::to('http://google.com');
    }

<a name="handling-forms"></a>
### 处理表单

您可以使用页面或布局中定义的方法处理标准表单[PHP部分](cms-themes.md#php-section) (处理AJAX请求在[AJAX Framework](ajax-introduction.md) 文章中进行了解释). 使用[form_open()](markup#standard-form) 函数定义一个引用事件方法的表单。例如：

    {{ form_open({ request: 'onHandleForm' }) }}
        请输入一些文字<input type="text" name="value"/>
        <input type="submit" value="提交"/>
    {{ form_close() }}
    <p>最后一次提交的内容是: {{ lastValue }}</p>

可以通过以下方式在页面或布局[PHP部分](cms-themes.md#php-section) 中定义onHandleForm函数:

    function onHandleForm()
    {
        $this['lastValue'] = post('value');
    }

方法使用`post`函数加载值并初始化页面`lastValue`属性变量，将该变量显示在第一个示例中的表单下方

> **注意:** 如果在页面布局中定义了具有相同名称的方法，则页面和页面[组件](cms-components.md) October 将执行页面方法。如果在组件和布局中定义了方法，则将执行布局方法。方法的优先级是：页面，布局，组件。

如果要引用特定[组件][component](cms-components.md)中定义的方法, 在处理程序引用中使用组件名称或别名

    {{ form_open({ request: 'myComponent::onHandleForm' }) }}

<a name="404-page"></a>
## 404 页面

如果主题包含URL为`/404`的页面，则在系统找不到请求的页面时显示该页面。

<a name="error-page"></a>
## 错误页面

默认情况下，任何错误都将显示一个详细的错误页面，其中包含发生错误的文件内容，行号和堆栈跟踪。您可以通过在`config/app.php`脚本中将配置值`debug`设置为**false**并创建一个URL为`/error`的页面来显示自定义错误页面。

<a name="page-variables"></a>
## 页面变量

可以通过引用`$this->page`在[PHP代码部分](cms-themes.md#php-section) 或 [组件](cms-components.md) 中访问页面的属性。

    function onEnd()
    {
        $this->page->title = '一个不同的页面标题';
    }

也可以使用[`this.page` variable](../markup/this-page)在标记中访问它们。例如：

    <p>这个页面的标题是: {{ this.page.title }}</p>

更多详情请看[`this.page` 标记教程](../markup/this-page).

<a name="injecting-assets"></a>
## 以编程方式注入页面资源

如有需要，您可以使用控制器的`addCss`和`addJs`方法将资源(CSS和JavaScript文件)注入页面。它可以在页面的[PHP部分](cms-themes.md#php-section)中定义的`onStart`函数或[layout](layout)模板。 例如:

    function onStart()
    {
        $this->addCss('assets/css/hello.css');
        $this->addJs('assets/js/app.js');
    }

如果`addCss`和`addJs`方法参数中指定的路径以斜杠(/)开头，那么它将相对于网站的根目录。如果文件路径不是以斜杠开头，那么它的路径相对于主题目录。

注入的文件可以通过数组的形式，例如：

    function onStart()
    {
        $this->addCss(['assets/css/hello.css', 'assets/css/goodbye.css']);
        $this->addJs(['assets/js/app.js', 'assets/js/nav.js']);
    }

可以使用combiner注入和编译LESS和SCSS资源：

    function onStart()
    {
        $this->addCss(['assets/less/base.less']);
    }

为了在页面或[布局](layout) 上输出注入的资源，请使用 [{% styles %}](markup/tag-styles) 和 [{% scripts %}](markup/tag-scripts) 标签。例如：

    <head>
        ...
        {% styles %}
    </head>
    <body>
        ...
        {% scripts %}
    </body>
