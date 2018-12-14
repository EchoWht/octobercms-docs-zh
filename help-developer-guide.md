# 开发者指南

- [编写文档](#writing-docs)
- [PSR标准的例外情况](#psr-exceptions)
    - [控制器方法可以有一个下划线](#psr-exception-methods)
    - [后续表达式在新一行](#psr-exception-newline-expressions)
- [开发人员标准和模式](#developer-standards)
    - [Vendor命名](#vendor-naming)
    - [仓库命名](#repository-naming)
    - [PHP变量命名](#variable-naming)
    - [HTML元素命名](#element-naming)
    - [视图文件命名](#view-naming)
    - [类文件命名](#class-naming)
    - [事件命名](#event-naming)
    - [数据库表命名](#db-table-naming)
    - [组件命名](#component-naming)
    - [控制器命名](#controller-naming)
    - [模型命名](#model-naming)
    - [模型scopes](#model-scopes)
    - [Class guidance](#class-guide)
- [环境配置](#environment-config)
    - [使用MySQL的严格模式](#strict-trans-tables)

<a name="writing-docs"></a>
## 编写文档

非常欢迎您对October文档的作出贡献。 如果您想贡献，请遵循以下规则。 如何设计完美的October文档页面：

1. 每个至少有一个H2标头的页面应该有一个TOC列表。 TOC列表应该是H1标头之后的第一个元素。 TOC应该链接到页面上的所有H2标头。
1. 即使有引言部分，TOC下面也应该有介绍性文本。如果不是真的需要，你可能想要摆脱介绍部分。不要单独留下TOC。
1. 尝试仅使用H2和H3标头。
1. 每个H2和H3标题应该有一个定义为“<a name="page-cycle-handlers"> </a>”的链接
1. 仅对TOC列表使用UL标签。
1. 避免短，1句，段落。合并短段落并尝试更加冗长。
1. 避免在代码部分下方悬挂段落。将这些段落与代码块上方的文本合并。
1. 对代码相关的所有内容使用内联`code`标签 - 变量名，函数名，语法示例等。
1. 将**strong**标签用于其他所有内容。
1. 不要犹豫与其他文档文章交叉链接。不需要在同一段落中添加指向同一篇文章的链接。
1. 参见[cms-pages.md](https://github.com/octobercms/docs/blob/master/cms-pages.md)或[cms-themes.md](https://github.com /octobercms/docs/blob/master/cms-themes.md)文件供您参考。

<a name="psr-exceptions"></a>
## PSR标准的例外情况

October使用的PSR标准有一些例外。

<a name="psr-exception-methods"></a>
### 控制器方法可以有一个下划线

PSR-2声明方法必须在**camelCase**中。 但是，在Backend控制器中，October将为AJAX处理程序添加操作名称，以定义受控上下文。 例如：

    public function index()
    {
        // This is the index page (index action)
    }

    public function index_onDoSomething()
    {
        // AJAX handler only works on the index action
    }

    public function onDoSomethingElse()
    {
        // AJAX handler works globally for all actions
    }

必须为这些方案授予例外。

<a name="psr-exception-newline-expressions"></a>
### 后续表达式在新一行

PSR-2没有明确声明后续表达式应与右括号位于同一行。

以下代码被认为是有效的，建议更好的逻辑间隔：

    if ($expr1) {
        // if body
    }
    elseif ($expr2) {
        // elseif body
    }
    else {
        // else body;
    }

    try {
        // try body
    }
    catch (FirstExceptionType $e) {
        // catch body
    }
    catch (OtherExceptionType $e) {
        // catch body
    }

基于技术性，这是可接受的偏好，在这种情况下，当使用SHOULD，MUST等时，PSR-1和PSR-2不明确。 但是，在撰写本文时，PSR-2 codesniffer规则表明它无效，因此可能需要例外。

<a name="developer-standards"></a>
## 开发人员标准和模式

本节介绍了我们强烈建议每个人遵循的一些标准，特别是如果您要在Marketplace上发布您的产品。

<a name="vendor-naming"></a>
### Vendor命名

命名空间中的vendor或作者代码必须以大写字符开头，不应包含下划线或短划线。 这些是有效名称的示例：

    Acme.Blog
    RainLab.User
    Happygilmore.Golf

这些是**不是**有效的名称示例：

    acme.blog
    rainLab.user
    Happy_gilmore.Golf

<a name="repository-naming"></a>
### 仓库命名

将工作发布到版本管理库(例如Git)时，请使用以下命名作为约定。 插件应该用`-plugin`后缀和可选的`oc -`前缀命名。

    blog-plugin
    oc-blog-plugin

主题应该用`-theme`后缀和可选的`oc -`前缀命名。

    happy-theme
    oc-happy-theme

<a name="variable-naming"></a>
### PHP变量命名

使用 **camelCase** 除以下情况外：

1.数据库属性和关系应该使用**snake_case**
1.返回参数和HTML元素应该使用**snake_case**
1.语言键应使用**snake_case**

<a name="element-naming"></a>
### HTML元素命名

Form元素名称应该使用snake_case(下划线)

    <input name="first_name">

如果名称是数组，则数组键可以是StudlyCase或snake_case。

    <input name="ForumMember[first_name]">
    <input name="forum_member[first_name]">

元素ID应为驼峰式或连字符(短划线)

    <div id="firstNameGroup">
        <input id="firstName">
    </div>

    <div id="first-name-group">
        <input id="first-name">
    </div>

元素类名称应使用连字符(破折号)

    <div class="form-group">
        <input class="form-control">
    </div>

<a name="view-naming"></a>
### 视图文件命名

Partial视图应以下划线字符开头。 而Controller和Layout视图不以下划线字符开头。 由于视图通常位于单个文件夹中，因此下划线(_)和短划线( - )字符可用于组织文件。 短划线用作空格字符的替代。 下划线用作斜杠字符(文件夹或命名空间)的替代。

    index_fancy-layout.htm       <== Index\Fancy layout
    form-with-sidebar.htm        <== Form with sidebar
    _field-container.htm         <== Field container (partial)
    _field_baloon-selector.htm   <== Field\Baloon Selector (partial)

视图文件必须以`.htm`文件扩展名结尾。

<a name="class-naming"></a>
### 类文件命名

类通常放在`classes`目录中。 我们建议后缀和前缀。

1. Manager
1. Builder
1. Writer
1. Reader
1. Handler
1. Container
1. Protocol
1. Target
1. Converter
1. Controller
1. View
1. Factory
1. Entity
1. Engine
1. Bag

> 不要对命名感到不拿耐烦。 是的，名字非常重要，但它们不足以浪费大量时间。 如果你不能在五分钟内想出一个好名字，继续前进。

<a name="event-naming"></a>
### 事件命名

指定[事件名称](../../docs/services/events)时。 *after*不在事件中使用，只使用*before*。 例如：

1. **beforeSetAttribute** - 这个事件来*before*任何默认逻辑。
1. **setAttribute** - 此事件来自*after*任何默认逻辑。

可能的事件应涵盖全局和本地版本。 全局事件应以模块或插件名称为前缀。 例如：

    // For global events, it is prefixed with the module or plugin code
    Event::fire('cms.page.end');

    // For local events, the prefix is not required
    $this->fireEvent('page.end');

避免在事件名称中使用诸如*onSomething*之类的术语，可以使用单词*bind*/*fire*表示此动作词。

最好将调用对象作为第一个参数传递给全局事件，本地事件不应该需要这个。 当暂停时，本地事件优先于全局事件，或者在处理时优先于本地事件。

    // Local event
    if ($this->fireEvent('beforeAddContent', [$message, $view], true) === false)
        return;

    // Global event
    if (Event::fire('mailer.beforeAddContent', [$this, $message, $view], true) === false)
        return;

当期望多个结果时，很容易组合这样的数组：

    // Combine local and global event results
    $eventResults = array_merge(
        $this->fireEvent('form.beforeRefresh', [$saveData]),
        Event::fire('backend.form.beforeRefresh', [$this, $saveData])
    );

<a name="db-table-naming"></a>
### 数据库表命名

表名应以作者和插件名称为前缀。

    acme_blog_xxx

布尔列名称应以“is_”为前缀

    is_activated
    is_visible

这是因为模型属性可能会发生冲突，例如，Model类中的`public $visible;`与具有相同名称的数据库列冲突。 某些列名称是例外，例如`notify_user`。

如果您的插件扩展了属于其他插件的表，则添加的列名应该以作者和插件名为前缀：

    acme_blog_category_id

作者和插件名称的首字母缩写也是可以接受的：

    ab_category_id

<a name="component-naming"></a>
### 组件命名

组件类通常位于`components`目录中。 组件的名称应代表其主要功能。

要显示记录列表，请使用`List`后缀，例如：

    ProductList
    ProductReviewList
    CategoryList

要显示单个记录，请使用`Details`后缀，例如：

    ProductDetails
    CategoryDetails

使用后缀有助于避免与控制器和模型名称冲突。 或者，对于名称具有描述性且不冲突的情况，您可以为没有后缀的组件命名：

    ProductReviews
    CustomerCheckout
    SeoDirectory
    UserProfile

<a name="controller-naming"></a>
### 控制器命名

控制器通常放在`controllers`目录中，用于后端控制器。 控制器的名称应为复数，例如：

    People
    Products
    Categories
    ProductCategories

<a name="model-naming"></a>
### 模型命名

模型通常放在`models`目录中。 模型的名称应该是单数，例如：

    Person
    Product
    Category
    ProductCategory

在扩展其他模型时，您应该在字段前面添加至少插件名称。

    User::extend(function($model) {
        $model->hasOne['forum_member'] = ['RainLab\Forum\Models\Member'];
    });

完全限定的插件名称也是可以接受的，例如：

    $user->rainlab_forum_member

<a name="model-scopes"></a>
### 模型范围

如果模型范围返回用于链接的查询对象，则应使用“apply”作为前缀，以指示它们正在应用于查询。 定义为：

    public function scopeApplyUser($query, $user)
    {
        return $query->where('user_id', $user->id);
    }

然后应用于模型：

    $model->applyUser($user);

虽然`apply`是理想的前缀名称，但以下是我们为链接范围推荐的一些其他前缀：

    - is
    - for
    - with
    - without
    - filter

如果范围返回除查询之外的任何内容，则可以使用任何名称。 非链式范围的一些可接受的名称：

    - find
    - get
    - list
    - lists

<a name="class-guide"></a>
### Class引导

这些要点应以轻松的方式考虑：

1. 在类中，属性和方法应声明为`protected`，以支持`private`。 所以所有类都可以用作基类。
1. 如果属性包含单个值(不是数组)，则使属性“public”而不是get/set方法。
1. 如果属性包含集合(是一个数组)，则使用get`getProperties`，`getProperty`和`setProperty`创建属性`protected`。

<a name="environment-config"></a>
## 环境配置

<a name="strict-trans-tables"></a>
### 使用MySQL的严格模式

当启用MySQL[STRICT_TRANS_TABLES模式](http://dev.mysql.com/doc/refman/5.0/en/sql-mode.html) 时，服务器将执行严格的数据类型验证。 强烈建议在开发期间在MySQL中启用此模式。 这允许您在代码使用enabled strict模式到达客户端服务器之前查找错误。 可以在my.cnf(Unix)或my.ini(Windows)文件中启用该模式：

    sql_mode=STRICT_TRANS_TABLES
