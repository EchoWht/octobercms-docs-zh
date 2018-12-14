# 组件开发

- [介绍](#introduction)
- [组件类定义](#component-class-definition)
    - [组件注册](#component-registration)
- [组件属性](#component-properties)
    - [下拉属性](#dropdown-properties)
    - [页面列表属性](#page-list-properties)
- [路由参数](#routing-parameters)
- [处理页面执行周期](#page-cycle)
    - [页面执行生命周期处理程序](#page-cycle-handlers)
    - [组建初始化](#page-cycle-init)
    - [页面周期响应](#page-cycle-response)
- [AJAX处理](#ajax-handlers)
- [默认标签](#default-markup)
- [组件部分(cms-partials.md)](#component-partials)
    - [引用自身 "self"](#referencing-self)
    - [唯一标识符](#unique-identifier)
- [在代码里引用partials](#render-partial-method)
- [使用组件注入页面资源](#component-assets)

<a name="introduction"></a>
## 介绍

组件文件和目录位于插件目录的 **/components** 子目录中。 每个组件都有一个定义组件类的PHP文件和一个可选的组件partials目录。 组件partials目录名称与以小写字母书写的组件类名称匹配。 组件目录结构的示例：

    plugins/
      acme/
        myplugin/
          components/
            componentname/       <=== 组件partials目录
              default.htm        <=== 组件默认标记（可选）
            ComponentName.php    <=== 组件类文件
          Plugin.php

组件必须使用`registerComponents`方法在[Plugin注册类中注册](#component-registration)。

<a name="component-class-definition"></a>
## 组件类定义

**组件类文件**定义组件功能和[组件属性](#component-properties)。 组件类文件名应与组件类名匹配。 组件类应该继承`\Cms\Classes\ComponentBase`类。 下一个示例的组件应该在plugins/acme/blog/components/BlogPosts.php文件中定义。

    namespace Acme\Blog\Components;

    class BlogPosts extends \Cms\Classes\ComponentBase
    {
        public function componentDetails()
        {
            return [
                'name' => 'Blog Posts',
                'description' => 'Displays a collection of blog posts.'
            ];
        }

        // This array becomes available on the page as {{ component.posts }}
        public function posts()
        {
            return ['First Post', 'Second Post', 'Third Post'];
        }
    }

`componentDetails`方法是必需的。该方法应该返回一个带有两个键的数组：`name`和`description`。 名称和说明显示在CMS后端用户界面中。

当此[组件附加到页面或布局](../cms/components)时，类属性和方法在页面上通过组件变量可用，该组件变量的名称与组件短名称或别名匹配。 例如，如果上一个示例中的BlogPost组件是在具有短名称的页面上定义的：

    url = "/blog"

    [blogPosts]
    ==

您可以通过`blogPosts`变量访问其`posts`方法。 请注意，Twig支持方法的属性表示法，因此您不需要使用括号。

    {% for post in blogPosts.posts %}
        {{ post }}
    {% endfor %}

<a name="component-registration"></a>
### 组件注册

必须通过覆盖[Plugin注册类](registration＃registration-file)中的`registerComponents`方法来注册组件。 这告诉CMS有关组件并提供**短名称**以便使用它。 注册组件的示例：

    public function registerComponents()
    {
        return [
            'October\Demo\Components\Todo' => 'demoTodo'
        ];
    }

这将使用默认别名**demoTodo**注册Todo组件类。 有关使用组件的更多信息，请参见[CMS组件文章](../cms/components)。

<a name="component-properties"></a>
## 组件属性

将组件添加到页面或布局时，可以使用属性对其进行配置。 使用组件类的`defineProperties`方法定义属性。 下一个示例显示了如何定义组件属性：

    public function defineProperties()
    {
        return [
            'maxItems' => [
                 'title'             => 'Max items',
                 'description'       => 'The most amount of todo items allowed',
                 'default'           => 10,
                 'type'              => 'string',
                 'validationPattern' => '^[0-9]+$',
                 'validationMessage' => 'The Max Items property can contain only numeric symbols'
            ]
        ];
    }

该方法应该返回一个数组，其中属性键作为索引，属性参数作为值。 属性键用于访问组件类中的组件属性值。 使用具有以下键的数组定义属性参数：

键 | 描述
------------- | -------------
**title** | 标题，必需的属性，它由CMS后端的组件Inspector使用。
**description** | 必需的属性描述，它由CMS后端的组件Inspector使用。
**default** | 选填，将组件添加到CMS后端中的页面或布局时使用的默认属性值。
**type** | 选填，指定属性类型。 该类型定义了在Inspector中显示属性的方式。 目前支持的类型是**string**，**checkbox**和**dropdown**。 默认值：**string**。
**validationPattern** | 用户在检查器中输入属性值时使用的可选正则表达式。 验证只能用于**string**属性。
**validationMessage** | 验证失败时显示的可选错误消息。
**required** | 可选，该字段必填，不填是提示validationMessage中的内容
**placeholder** | 字符串和下拉属性的可选占位符。
**options** | 下拉属性的可选选项数组。
**depends** | 下拉属性所依赖的属性名称数组。 请参阅下面的[下拉属性](#dropdown-properties)。
**group** | 一个可选的组名。 组在Inspector中创建部分，简化用户体验。 在多个属性中使用相同的组名来组合它们。
**showExternalParam** | 指定Inspector中属性的外部参数编辑器的可见性。 默认值：**true**。

在组件内部，您可以使用`property`方法读取属性值：

    $this->property('maxItems');

如果未定义属性值，则可以提供默认值作为`property`方法的第二个参数：

    $this->property('maxItems', 6);

您还可以将所有属性加载为数组：

    $properties = $this->getProperties();
    
要从组件的Twig partials访问该属性，请使用引用Component对象的`__SELF__`变量：

   `{{ __SELF__.property('maxItems') }}`

<a name="dropdown-properties"></a>
### 下拉属性

下拉属性的选项列表可以是静态的或动态的。 静态选项使用属性定义的`options`元素定义。示例：

    public function defineProperties()
    {
        return [
            'units' => [
                'title'       => 'Units',
                'type'        => 'dropdown',
                'default'     => 'imperial',
                'placeholder' => 'Select units',
                'options'     => ['metric'=>'Metric', 'imperial'=>'Imperial']
            ]
        ];
    }

显示Inspector时，可以从服务器动态获取选项列表。 如果在下拉属性定义中省略`options`参数，则选项列表被视为动态。 组件类必须定义返回选项列表的方法。 该方法应具有以下格式的名称：`get*Property*Options`，其中**Property**是属性名称，例如：`getCountryOptions`。 该方法返回一个选项数组，其中选项值为键，选项标签为值。 动态下拉列表定义的示例：

    public function defineProperties()
    {
        return [
            'country' => [
                'title'   => 'Country',
                'type'    => 'dropdown',
                'default' => 'us'
            ]
        ];
    }

    public function getCountryOptions()
    {
        return ['us'=>'United states', 'ca'=>'Canada'];
    }

动态下拉列表可以依赖于其他属性。 例如，州名单可能取决于所选国家。 依赖项在属性定义中使用`depends`参数声明。 下一个示例定义了两个动态下拉属性，状态列表取决于国家/地区：

    public function defineProperties()
    {
        return [
            'country' => [
                'title'       => 'Country',
                'type'        => 'dropdown',
                'default'     => 'us'
            ],
            'state' => [
                'title'       => 'State',
                'type'        => 'dropdown',
                'default'     => 'dc',
                'depends'     => ['country'],
                'placeholder' => 'Select a state'
            ]
        ];
    }

要加载状态列表，您应该知道Inspector中当前选择的国家/地区。 Inspector将所有属性值POST到`getPropertyOptions`处理程序，因此您可以执行以下操作：

    public function getStateOptions()
    {
        $countryCode = Request::input('country'); // Load the country property value from POST

        $states = [
            'ca' => ['ab'=>'Alberta', 'bc'=>'British columbia'],
            'us' => ['al'=>'Alabama', 'ak'=>'Alaska']
        ];

        return $states[$countryCode];
    }

<a name="page-list-properties"></a>
### 页面列表属性

有时组件需要创建指向网站页面的链接。 例如，博客帖子列表包含指向博客帖子详细信息页面的链接。 在这种情况下，组件应该知道帖子详细信息页面文件名（然后它可以使用[页面Twig过滤器](../cms/markup＃page-filter)）。 October包含一个用于创建动态下拉页面列表的帮助程序。 下一个示例定义了postPage属性，该属性显示了一个页面列表：

    public function defineProperties()
    {
            return [
                'postPage' => [
                    'title' => 'Post page',
                    'type' => 'dropdown',
                    'default' => 'blog/post'
                ]
            ];
    }

    public function getPostPageOptions()
    {
        return Page::sortBy('baseFileName')->lists('baseFileName', 'baseFileName');
    }

<a name="routing-parameters"></a>
## 路由参数

组件可以直接访问[页面URL](../cms/pages#url-syntax)中定义的路由参数值。

    // 返回URL段值，例如： /page/:post_id
    $postId = $this->param('post_id');

在某些情况下，[组件属性](#component-properties)可以充当硬编码值或引用URL中的值。

这个硬编码示例显示了使用标识符“2”的博客文章：

    url = "/blog/hard-coded-page"

    [blogPost]
    id = "2"

或者，可以使用[external property value](../cms/components#external-property-values)从页面URL动态引用该值：

    url = "/blog/:my_custom_parameter"

    [blogPost]
    id = "{{ :my_custom_parameter }}"

在这两种情况下，都可以使用`property`方法检索值：

    $this->property('id');

如果需要访问路由参数名称：

    $this->paramName('id'); // 返回 "my_custom_parameter"

<a name="page-cycle"></a>
## 处理页面执行周期

通过覆盖组件类中的`onRun`方法，组件可以参与页面执行循环事件。 每次加载页面或布局时，CMS控制器都会执行此方法。 在方法内部，您可以通过`page`属性将变量注入到Twig环境中：

    public function onRun()
    {
        // 加载页面或布局并将组件附加到该代码时，将执行此代码。

        $this->page['var'] = 'value'; // 将一些变量注入页面
    }

<a name="page-cycle-handlers"></a>
### 页面执行生命周期处理程序

当页面加载时，October执行可以在布局和页面[PHP部分](../cms/themes#php-section)和组件类中定义的处理函数。 执行处理程序的顺序如下：

1. 布局Layout `onInit()` function.
1. 页面Page `onInit()` function.
1. 布局Layout `onStart()` function.
1. 布局中的组件Layout components `onRun()` method.
1. 布局Layout `onBeforePageStart()` function.
1. 页面Page `onStart()` function.
1. 页面组件Page components `onRun()` method.
1. 页面Page `onEnd()` function.
1. 组件Layout `onEnd()` function.

<a name="page-cycle-init"></a>
### 组建初始化

有时您可能希望在首次实例化组件类时执行代码。 您可以覆盖组件类中的`init`方法来处理任何初始化逻辑，这将在AJAX处理程序之前和页面执行生命周期之前执行。 例如，此方法可用于动态地将另一个组件附加到页面。

    public function init()
    {
        $this->addComponent('Acme\Blog\Components\BlogPosts', 'blogPosts');
    }

<a name="page-cycle-response"></a>
### 页面周期响应

与[页面执行生命周期](../cms/layouts#layout-life-cycle)中的所有方法一样，如果组件中的`onRun`方法返回一个值，这将在此时停止循环并返回 响应浏览器。 这里我们使用`Response`facade返回一个拒绝访问的消息：

    public function onRun()
    {
        if (true) {
            return Response::make('Access denied!', 403);
        }
    }

<a name="ajax-handlers"></a>
## AJAX处理

组件可以托管AJAX事件处理程序。 它们在组件类中定义，就像它们可以在[页面或布局代码](../ajax/handlers)中定义一样。 在组件类中定义的示例AJAX处理程序方法：

    public function onAddItem()
    {
        $value1 = post('value1');
        $value2 = post('value2');
        $this->page['result'] = $value1 + $value2;
    }

如果此组件的别名是*demoTodo*，则可以通过`demoTodo::onAddItem`访问此处理程序。 有关将AJAX与组件一起使用的详细信息，请参阅[调用组件中定义的AJAX处理程序(../ajax/handlers#calling-handlers))文章。

<a name="default-markup"></a>
## 默认标签

所有组件都可以带有默认标记，当将其包含在带有`{％component％}`标记的页面上时使用该标记，尽管这是可选的。 默认标记保存在**组件partials部分目录**中，该目录与小写的组件类具有相同的名称。

默认组件标记应放在名为**default.htm**的文件中。 例如，Demo ToDo组件的默认标记在文件 **/plugins/october/demo/components/todo/default.htm** 中定义。 然后可以使用`{％component％}`标记将其插入页面的任何位置：

    url = "/todo"

    [demoTodo]
    ==
    {% component 'demoTodo' %}

默认标记还可以采用在渲染时覆盖[组件属性](#component-properties) 的参数。

    {% component 'demoTodo' maxItems="7" %}

这些属性在`onRun`方法中不可用，因为它们是在页面循环完成后建立的。 相反，可以通过覆盖组件类中的`onRender`方法来处理它们。 CMS控制器在呈现默认标记之前执行此方法。

    public function onRender()
    {
        // 此代码将在页面或布局上呈现默认组件标记之前执行。

        $this->page['var'] = 'Maximum items allowed: ' . $this->property('maxItems');
    }

<a name="component-partials"></a>
## 组件部分(cms-partials.md)

除了默认标记之外，组件还可以提供可以在前端或默认标记本身内使用的其他部分。 如果Demo ToDo组件具有**分页**部分，它将位于 **/plugins/october/demo/components/todo/pagination.htm** 中并使用以下内容显示在页面上：

    {% partial 'demoTodo::pagination' %}

可以使用比较容易的方法，即上下文。 如果在组件partial内部调用，它将直接引用自身。 如果在主题partial内部调用，它将扫描页面/布局上使用的所有组件以获得匹配的部分名称并使用它。

    {% partial '@pagination' %}

通过将部分文件放在名为 **components/partials** 的目录中，多个组件可以共享部分。 当找不到通常的组件部分时，此目录中找到的部分用作后备。 例如，位于 **/plugins/acme/blog/components/partials/shared.htm** 中的共享部分可以由任何组件在页面上显示，使用：

    {% partial '@shared' %}

<a name="referencing-self"></a>
### 引用自身 "self"

组件可以使用`__SELF__`变量在其partials中引用它们自己。 默认情况下，它将返回组件的短名称或[别名](../cms/components#aliases)。

    <form data-request="{{__SELF__}}::onEventHandler">
        [...]
    </form>

组件也可以引用自己的属性。

    {% for item in __SELF__.items() %}
        {{ item }}
    {% endfor %}

如果在组件partial中你需要渲染另一个组件，则部分地将`__SELF__`变量与部分名称连接起来：

    {% partial __SELF__~"::screenshot-list" %}

<a name="unique-identifier"></a>
### 唯一标识符

如果在同一页面上调用两次相同的组件，则可以使用`id`属性来引用每个实例。

    {{__SELF__.id}}

每次显示组件时，ID都是唯一的。

    <!-- ID: demoTodo527c532e9161b -->
    {% component 'demoTodo' %}

    <!-- ID: demoTodo527c532ec4c33 -->
    {% component 'demoTodo' %}

<a name="render-partial-method"></a>
## 在代码里引用partials

您可以使用`renderPartial`方法以编程方式在PHP代码中呈现组件部分。 这将检查组件的名为`component-partial.htm`的部分，并将结果作为字符串返回。 第二个参数用于传递视图变量。

    $content = $this->renderPartial('component-partial.htm');

    $content = $this->renderPartial('component-partial.htm', [
        'name' => 'John Smith'
    ]);

例如，要将partial作为对[AJAX 处理](../ajax/handlers)的响应呈现：

    function onGetTemplate()
    {
        return ['#someDiv' => $this->renderPartial('component-partial.htm')];
    }

另一个例子可能是通过从`onRun` [页面循环方法](#page-cycle)返回一个值来覆盖整个页面视图响应。 此代码将使用“Response”facade专门返回XML响应：

    public function onRun()
    {
        $content = $this->renderPartial('default.htm');
        return Response::make($content)->header('Content-Type', 'text/xml');
    }

<a name="component-assets"></a>
## 使用组件注入页面资源

组件可以将静态文件（CSS和JavaScript文件）注入它们所附加的页面或布局。 使用控制器的`addCss`和`addJs`方法将静态文件添加到CMS控制器。 它可以在组件的`onRun`方法中完成。 请阅读有关[在页面文章中注入静态文件](../cms/page#injecting-assets).的更多详细信息。 例：

    public function onRun()
    {
        $this->addJs('/plugins/acme/blog/assets/javascript/blog-controls.js');
    }

如果`addCss`和`addJs`方法参数中指定的路径以斜杠（/）开头，那么它将相对于网站根目录。 如果资产路径不以斜杠开头，则它相对于组件目录。
