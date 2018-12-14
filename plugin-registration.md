# 插件(Plugins)注册

- [介绍](#introduction)
    - [目录结构](#directory-structure)
    - [插件命名空间(namespaces)](#namespaces)
- [注册文件](#registration-file)
    - [支持的方法](#registration-methods)
    - [基本插件信息](#basic-plugin-information)
- [路由和初始化](#routing-initialization)
- [依赖关系定义](#dependency-definitions)
- [继承Twig](#extending-twig)
- [导航菜单](#navigation-menus)
- [注册中间件(middleware)](#registering-middleware)
- [提升权限](#elevated-plugin)

<a name="introduction"></a>
## 介绍

插件是通过扩展将新功能添加到CMS的基础。 本文介绍了组件注册。 注册过程允许插件声明其功能，如[组件](components)或后端菜单和页面。 插件可以执行的一些示例：

1. 定义 [组件](components).
1. 定义 [用户权限](../backend/users).
1. 添加 [设置页面](settings#backend-pages), [菜单项](#navigation-menus), [列表](../backend/lists) 和 [表单](../backend/forms).
1. 创建 [数据库表结构和数据](updates).
1. 修改 [核心或其他插件的功能](events).
1. 准备classes, [后端控制器](../backend/controllers-views-ajax), 视图, 资源和其他文件。

<a name="directory-structure"></a>
### 目录结构

插件在应用程序目录的**/plugins**子目录中。 插件目录结构的示例：

    plugins/
      acme/              <=== Author name
        blog/            <=== Plugin name
          classes/
          components/
          controllers/
          models/
          updates/
          ...
          Plugin.php     <=== 插件注册文件

并非所有插件目录都是必需的。 唯一需要的文件是下面描述的**Plugin.php**。 如果你的插件只提供一个[组件](components)，你的插件目录可能会简单得多，如下所示：

    plugins/
      acme/              <=== 作者名称
        blog/            <=== 扩展名称
          components/
          Plugin.php     <=== 插件注册文件

> **注意:** 如果您正在为[Marketplace](http://octobercms.com/help/site/marketplace)开发插件，则需要 [updates/version.yaml](updates) 文件。

<a name="namespaces"></a>
### 插件命名空间(namespaces)

插件名称空间非常重要，特别是如果您要在[October市场](http://octobercms.com/plugins)上发布插件。 当您在Marketplace上注册为作者时，将要求您提供作为所有插件的根命名空间的作者代码。 注册时，您只能指定一次作者代码。 市场提供的默认作者代码由作者姓氏和名字组成，如：JohnSmith。 注册后无法更改代码。 您应该在根命名空间下定义所有插件名称空间，例如`\JohnSmith\Blog`。

<a name="registration-file"></a>
## 注册文件

**Plugin.php**文件，称为*插件注册文件*，是一个初始化脚本，用于声明插件的核心功能和信息。 注册文件可以提供以下内容：

1. 有关插件，其名称和作者的信息。
1. 扩展CMS的注册方法。

注册脚本应使用插件命名空间。 注册脚本应该定义一个名为`Plugin`的类，应扩展`\System\Classes\PluginBase`类。 插件注册类唯一需要的方法是`pluginDetails`。 插件注册文件的示例：

    namespace Acme\Blog;

    class Plugin extends \System\Classes\PluginBase
    {
        public function pluginDetails()
        {
            return [
                'name' => 'Blog Plugin',
                'description' => 'Provides some really cool blog features.',
                'author' => 'ACME Corporation',
                'icon' => 'icon-leaf'
            ];
        }

        public function registerComponents()
        {
            return [
                'Acme\Blog\Components\Post' => 'blogPost'
            ];
        }
    }

<a name="registration-methods"></a>
### 支持的方法

插件注册类支持以下方法：

方法 | 描述
------------- | -------------
**pluginDetails()** | 返回有关插件的信息。
**register()** | 注册方法，在首次注册插件时调用。
**boot()** | 引导方法，在请求路由之前调用。
**registerMarkupTags()** | 注册可在CMS中使用的[其他标记标记](#extended-twig)。
**registerComponents()** | 注册此插件的[组件](components#component-registration)。
**registerNavigation()** | 注册此插件的[后端导航菜单项](#navigation-menus)。
**registerPermissions()** | 注册此插件的[后端权限](../backend/users#permission-registration)。
**registerSettings()** | 注册此插件的[后端设置链接](settings#link-registration)。
**registerFormWidgets()** | 注册此插件的[后端表单小部件](../backend/widgets#form-widget-registration)。
**registerReportWidgets()** | 注册此插件的[后端报告小部件](../backend/widgets#report-widget-registration)，包括仪表板小部件。
**registerListColumnTypes()** | 注册此插件提供的[自定义列表列类型](../backend/lists#custom-column-types) 。
**registerMailTemplates()** | 注册此插件提供的[邮件视图模板](mail#mail-template-registration) 。
**registerSchedule()** | 注册定期执行的[计划任务](../plugin/scheduling#defining-schedules) 。

<a name="basic-plugin-information"></a>
### 基本插件信息

`pluginDetails`是插件注册类的必需方法。 它应该返回一个包含以下键的数组：

键 | 描述
------------- | -------------
**name** | 插件名称，必填。
**description** | 插件描述，必需。
**author** | 插件作者姓名，必填。
**icon** | 插件图标的名称。 可以在[UI文档](../ui/icon)中找到可用图标的完整列表。   此字体提供的任何图标名称均有效，例如：**icon-glass**， **icon-music**。
**iconSvg** | 用于代替标准图标的SVG图标，可选。 SVG图标应为矩形，可以支持多种颜色。
**homepage** | 指向作者网站地址的链接，可选。

<a name="routing-initialization"></a>
## 路由和初始化

插件注册文件可以包含两个方法`boot`和`register`。 使用这些方法，您可以执行任何您喜欢的操作，例如注册路径或将处理程序附加到事件。

注册插件时立即调用`register`方法。 在路由请求之前调用`boot`方法。 因此，如果您的操作依赖于另一个插件，则应使用引导方法。 例如，在`boot`方法中，您可以扩展模型：

    public function boot()
    {
        User::extend(function($model) {
            $model->hasOne['author'] = ['Acme\Blog\Models\Author'];
        });
    }

> **注意:** 在更新过程中不会调用`boot`和`register`方法来保护系统免受严重错误的影响。要克服此限制，请使用[提升权限](#elevated-plugin).

插件还可以提供名为**routes.php**的文件，其中包含自定义路由逻辑，如[路由器服务](../services/router)中所定义。 例如：

    Route::group(['prefix' => 'api_acme_blog'], function() {

        Route::get('cleanup_posts', function(){ return Posts::cleanUp(); });

    });

<a name="dependency-definitions"></a>
## 依赖关系定义

插件可以通过在[插件注册文件](#registration-file)中定义`$require`属性来依赖其他插件，该属性应该包含一系列被认为是需求的插件名称。 依赖于**Acme.User**插件的插件可以通过以下方式声明此要求：

    namespace Acme\Blog;

    class Plugin extends \System\Classes\PluginBase
    {
        /**
         * @var array Plugin dependencies
         */
        public $require = ['Acme.User'];

        [...]
    }

依赖关系定义将影响插件的运行方式和[更新过程如何应用更新](../plugin/updates#update-process)。 安装过程将尝试自动安装任何依赖项，但是如果在系统中检测到插件而没有任何依赖项，则会禁用它以防止系统错误。

依赖关系定义可能很复杂，但应注意防止循环引用。 应始终指向依赖图，并将循环依赖视为设计错误。

<a name="extending-twig"></a>
## 继承Twig

可以使用插件注册类的`registerMarkupTags`方法在CMS中注册自定义Twig过滤器和函数。 下一个示例注册两个Twig过滤器和两个函数。

    public function registerMarkupTags()
    {
        return [
            'filters' => [
                // A global function, i.e str_plural()
                'plural' => 'str_plural',

                // A local method, i.e $this->makeTextAllCaps()
                'uppercase' => [$this, 'makeTextAllCaps']
            ],
            'functions' => [
                // A static method call, i.e Form::open()
                'form_open' => ['October\Rain\Html\Form', 'open'],

                // Using an inline closure
                'helloWorld' => function() { return 'Hello World!'; }
            ]
        ];
    }

    public function makeTextAllCaps($text)
    {
        return strtoupper($text);
    }

<a name="navigation-menus"></a>
## 导航菜单

插件可以通过覆盖[插件注册类](#registration-file).的`registerNavigation`方法来扩展后端导航菜单。 本节介绍如何将菜单项添加到后端导航区域。 注册带有两个子菜单项的顶级导航菜单项的示例：

    public function registerNavigation()
    {
        return [
            'blog' => [
                'label'       => 'Blog',
                'url'         => Backend::url('acme/blog/posts'),
                'icon'        => 'icon-pencil',
                'permissions' => ['acme.blog.*'],
                'order'       => 500,

                'sideMenu' => [
                    'posts' => [
                        'label'       => 'Posts',
                        'icon'        => 'icon-copy',
                        'url'         => Backend::url('acme/blog/posts'),
                        'permissions' => ['acme.blog.access_posts']
                    ],
                    'categories' => [
                        'label'       => 'Categories',
                        'icon'        => 'icon-copy',
                        'url'         => Backend::url('acme/blog/categories'),
                        'permissions' => ['acme.blog.access_categories']
                    ]
                ]
            ]
        ];
    }

注册后端导航时，可以使用[本地化字符串](localization)作为“label”值。 后端导航也可以通过`permissions`值控制，并对应于定义的[后端用户权限](../backend/users)。 后端导航在整个导航菜单项上显示的顺序由“order”值控制。 数字越大意味着该项目将在菜单项的顺序中稍后出现，而较低的数字意味着它将在较早时出现。

要使子菜单项可见，您可以使用`BackendMenu::setContext`方法在后端控制器中[设置导航上下文](../backend/controllers-ajax#navigation-context) 。 这将使父菜单项处于活动状态，并在侧边菜单中显示子项。

<a name="registering-middleware"></a>
## 注册中间件(middleware)

要注册自定义中间件，您可以在引导方法中使用以下调用来扩展您希望添加中间件的Controller类。

    public function boot()
    {
        \Cms\Classes\CmsController::extend(function($controller) {
            $controller->middleware('Path\To\Custom\Middleware');
        });
    }

或者，您可以通过以下方式将其直接推送到内核中。

    public function boot()
    {
        // Add a new middleware to beginning of the stack.
        $this->app['Illuminate\Contracts\Http\Kernel']
             ->prependMiddleware('Path\To\Custom\Middleware');

        // Add a new middleware to end of the stack.
        $this->app['Illuminate\Contracts\Http\Kernel']
             ->pushMiddleware('Path\To\Custom\Middleware');
    }

<a name="elevated-plugin"></a>
## 提升权限

默认情况下，插件仅限于访问系统的某些区域。 这是为了防止可能将管理员锁定在后端的严重错误。 在没有提升权限的情况下访问这些区域时，插件的`boot`和`register` [初始化方法](#routing-initialization) 将不会触发。

请求 | 描述
------------- | -------------
**/combine** | 资源组合器生成器URL
**/backend/system/updates** | 网站更新上下文
**/backend/system/install** | 安装路径
**/backend/backend/auth** | 后端认证路径(登录，注销)
**october:up** | 运行所有挂起的迁移的CLI命令
**october:update** | 用于触发更新过程的CLI命令

定义`$elevated`属性以授予插件的提升权限。

    /**
     * @var bool Plugin requires elevated permissions.
     */
    public $elevated = true;
