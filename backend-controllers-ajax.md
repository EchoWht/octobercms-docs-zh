# 后端控制器和AJAX

- [介绍](#introduction)
    - [类定义](#class-definition)
    - [控制器属性](#controller-properties)
- [动作，视图和路由(Actions, views and routing)](#actions-views-routing)
- [将数据传递给视图](#passing-data-to-views)
- [设置导航上下文](#navigation-context)
- [使用AJAX处理程序](#ajax)
    - [后端AJAX处理程序](#ajax-handlers)
    - [触发AJAX请求](#triggering-ajax-requests)

<a name="introduction"></a>
## 介绍

October后端实现了MVC模式。 控制器管理后端页面并实现表单和列表等各种功能。 本文介绍如何开发后端控制器以及如何配置控制器行为。

每个控制器都用一个PHP脚本表示，该脚本位于Plugin目录的**/controllers **子目录中。 控制器视图是驻留在控制器视图目录中的`.htm`文件。 控制器视图目录名称与以小写形式写入的控制器类名称匹配。 视图目录还可以包含控制器配置文件。 控制器目录结构的示例：

    plugins/
      acme/
        blog/
          controllers/
            users/                <=== 控制器视图目录
              _partial.htm        <=== 控制器partial文件
              config_form.yaml    <=== 控制器配置文件
              index.htm           <=== 控制器视图文件
            Users.php             <=== 控制器类
          Plugin.php

<a name="class-definition"></a>
### 类定义

控制器类必须扩展`\Backend\Classes\Controller`类。 与任何其他插件类一样，控制器应该属于[plugin namespace](../plugin/registration#namespaces)。 插件中使用的Controller的最基本表示如下所示：

    namespace Acme\Blog\Controllers;

    class Posts extends \Backend\Classes\Controller {

        public function index()    // <=== Action method
        {

        }
    }

通常，每个控制器都实现了处理单一类型数据的功能 - 例如博客文章或类别。 下面描述的所有后端行为都采用此约定。

<a name="controller-properties"></a>
### 控制器属性

后端控制器基类定义了许多属性，允许配置页面外观和管理页面安全性：

属性 | 描述
------------- | -------------
**$fatalError** | 允许存储动作方法中生成的致命异常，以便在视图中显示它。
**$user** |  包含对后端用户对象的引用。
**$suppressView** | 允许阻止视图显示。 可以在操作方法或控制器构造函数中更新。
**$params** | 路由参数的数组。
**$action** | 当前请求中正在执行的操作方法的名称。
**$publicActions** | 定义了一个没有后端用户身份验证的可用操作数组。 可以在类定义中重写。
**$requiredPermissions** | 查看此页面所需的权限。 可以在类定义或控制器构造函数中设置。 有关详细信息，请参阅[用户和权限](users)。
**$pageTitle** | 设置页面标题。 可以在action方法中设置。
**$bodyClass** | 用于自定义布局的body类属性。 可以在控制器构造函数或操作方法中设置。
**$guarded** | 控制器特定的方法，不能被称为动作。 可以在控制器构造函数中进行扩展。
**$layout** | 为控制器视图指定自定义布局(请参阅下面的[layouts](#layouts))。

<a name="actions-views-routing"></a>
## 动作，视图和路由(Actions, views and routing)

公共控制器方法(称为**动作**)耦合到**视图文件**，其表示对应于动作的页面。 后端视图文件使用PHP语法。 **index.htm**视图文件内容的示例，对应于**index**动作方法：

    <h1>Hello World</h1>

此页面的URL由作者姓名，插件名称，控制器名称和操作名称组成。

    backend/[author name]/[plugin name]/[controller name]/[action name]  ，backend/[作者名]/[插件名]/[控制器名]/[方法名]
    
上述控制器可通过以下url访问：

    http://example.com/backend/acme/blog/users/index

<a name="passing-data-to-views"></a>
## 将数据传递给视图

使用控制器的`$vars`属性将任何数据直接传递给您的视图：

    $this->vars['myVariable'] = 'value';

现在可以在视图中直接访问使用`$vars`属性传递的变量：

    <p>变量值是<?= $myVariable ?></p>

<a name="navigation-context"></a>
## 设置导航上下文

插件可以在[插件注册文件](../plugin/registration#navigation-menus)中注册后端导航菜单和子菜单。 导航上下文确定当前后端页面的后端菜单和子菜单是活动的。 您可以使用`BackendMenu`类设置导航上下文：

    BackendMenu::setContext('Acme.Blog', 'blog', 'categories');

第一个参数指定作者和插件名称。 第二个参数设置菜单代码。 可选的第三个参数指定子菜单代码。 通常在控制器构造函数中调用`BackendMenu::setContext`。

    namespace Acme\Blog\Controllers;

    class Categories extends \Backend\Classes\Controller {

    public function __construct()
    {
        parent::__construct();

        BackendMenu::setContext('Acme.Blog', 'blog', 'categories');
    }

您可以使用控制器类的`$pageTitle`属性设置后端页面的标题(请注意，表单和列表行为可以执行此操作)：

    $this->pageTitle = 'Blog categories';

<a name="ajax"></a>
## 使用AJAX处理程序

后端AJAX框架使用相同的[AJAX库](../ajax/introduction) 作为前端页面。 库将自动加载到后端页面上。

<a name="ajax-handlers"></a>
### 后端AJAX处理程序

后端AJAX处理程序可以在控制器类或[小部件](widgets)中定义。 在控制器类中，AJAX处理程序被定义为公共方法，名称以`on`字符串开头：**onCreateTemplate**，**onGetTemplateList**等。

后端AJAX处理程序可以返回数据数组，抛出异常或重定向到另一个页面(请参阅[AJAX事件处理程序](../ajax/handlers))。 您可以使用`$this->vars`来设置变量，使用控制器的`makePartial`方法来呈现部分并将其内容作为响应数据的一部分返回。

    public function onOpenTemplate()
    {
        if (Request::input('someVar') != 'someValue') {
            throw new ApplicationException('Invalid value');
        }

        $this->vars['foo'] = 'bar';

        return [
            'partialContents' => $this->makePartial('some-partial')
        ];
    }

<a name="triggering-ajax-requests"></a>
### 触发AJAX请求

可以使用数据属性API或JavaScript API触发AJAX请求。 有关详细信息，请参阅[前端AJAX库](../ajax/introduction)。 以下示例显示如何使用后端按钮触发请求。

    <button
        type="button"
        data-request="onDoSomething"
        class="btn btn-default">
        Do something
    </button>

> **注意**: 您可以使用前缀`widget::onName`专门定位窗口小部件的AJAX处理程序。 有关更多详细信息，请参阅[widget AJAX处理程序文章](../backend/widgets#generic-ajax-handlers)。
