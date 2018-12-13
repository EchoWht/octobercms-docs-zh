# 使用Components组件

- [介绍](#introduction)
- [组件别名](#aliases)
- [使用外部属性值](#external-property-values)
- [将变量传递给组件](#component-variables)
- [自定义默认标记](#customizing-default-markup)
    - [将默认标签移动到partial](#moving-default-markup)
    - [覆盖组件部分](#overriding-partials)
- [“查看包”组件](#viewbag-component)


组件是可配置的、可以附加到任何页面，部分或布局构建元素。组件是October的主要特征。每个组件都实现了一些扩展您的网站的功能。组件可以在页面上输出HTML标签，但是没有必要 - 组件的其他重要功能是处理[AJAX请求](../ajax/introduction)处理表单提交的请求和处理页面执行周期，允许注入变量页面或提高网站的安全性。

本文介绍了组件的基础知识，并没有解释如何使用[AJAX组件](../ajax/handlers)或[开发组件](../plugin/components)作为插件的一部分。

> **注意:**  在partials中使用组件功能有限，这在[动态partials](partials＃dynamic-partials)文章中有更详细的描述。

<a name="introduction"></a>
## 介绍

如果使用后端用户界面，则可以通过单击“组件”面板中的组件，将其添加到页面，部分和布局中。如果使用文本编辑器，则可以通过将组件名称添加到模板文件的[配置](themes＃configuration-section)部分来实现将组件附加到页面或布局。下一个示例演示如何将To-do组件添加到页面中：
    title = "Components demonstration"
    url = "/components"

    [demoTodo]
    maxItems = 20
    ==
    ...

这将使用组件定义的属性来初始化这个组件。某些属性是必需的，某些属性具有默认值。如果您不确定组件支持哪些属性，请参阅组件开发人员提供的文档，或使用October后端的Inspector。单击页面或布局组件面板中的组件时，将打开Inspector。

When you refer a component, it automatically creates a page variable that matches the component name (`demoTodo` in the previous example). Components that provide HTML markup can be rendered on a page with the `{% component %}` tag, like this:
当您引用组件时，它会自动创建一个与组件名称匹配的页面变量（前一个示例中的“demoTodo”）。通过`{％component％}`标签将组件在页面中展示出来，如下所示：

    {% component 'demoTodo' %}

> **注意:** 如果将具有相同名称的两个组件一起放在page页面和layout布局，则page页面组件将覆盖layout布局组件的任何属性。

<a name="aliases"></a>
## 组件别名

如果有两个插件注册具有相同名称的组件，则可以使用其完全限定的类名称附加组件并为其分配 *别名*
    
    [October\Demo\Components\Todo demoTodoAlias]
    maxItems = 20

该部分中的第一个参数是类名，第二个参数是附加到页面时将使用的组件别名。如果指定了组件别名，则在页面中的任何位置都可以使用这个别名。例如：

    {% component 'demoTodoAlias' %}

别名还允许您通过定义不同的别名在同一页面上定义同一类的多个组件。这使您可以在页面上使用同一组件的多个实例。

    [demoTodo todoA]
    maxItems = 10
    [demoTodo todoB]
    maxItems = 20

<a name="external-property-values"></a>
## 使用外部属性值

默认情况下，属性值在定义组件时的Configuration部分中初始化，属性值是静态的，如下所示：

    [demoTodo]
    maxItems = 20
    ==
    ...

但是，有一种方法可以使用外部参数来初始化属性，URL参数或[partial](partials)参数（对于partials中定义的组件）。对于应从partials变量加载的值，请使用`{{paramName}}`语法：

    [demoTodo]
    maxItems = {{ maxItems }}
    ==
    ...

假设在上面的例子中，组件 **demoTodo** 是在partial中定义的，它将使用从 **maxItems** partial变量加载的值进行初始化：

    {% partial 'my-todo-partial' maxItems='10' %}

要从URL参数加载属性值，请使用`{{：paramName}}`语法，其中名称以冒号（`:`）开头，例如：

    [demoTodo]
    maxItems = {{ :maxItems }}
    ==
    ...

组件所属的页面应该具有相应的[URL参数](pages#url-syntax) 定义：

    url = "/todo/:maxItems"

在October的后端，您可以使用Inspector工具为组件属性分配外部值。在Inspector中，您不需要使用花括号来输入参数名称。检查器中的每个字段右侧都有一个图标，用于打开外部参数名称编辑器。为部分变量输入参数名称为“paramName”，为URL参数输入`：paramName`。

<a name="component-variables"></a>
## 将变量传递给组件

Components can be designed to use variables at the time they are rendered, similar to [Partial variables](partials#partial-variables), they can be specified after the component name in the `{% component %}` tag. The specified variables will explicitly override the value of the [component properties](../plugin/components#component-properties), including [external property values](#external-property-values).
组件可以设计为在渲染时使用变量，类似于[Partial变量](partials＃partial-variables)，它们可以在`{％component％}`标签中的组件名称之后定义。定义的变量将显式覆盖[组件属性](./ plugin/components＃component-properties)的值，​​包括[外部属性值](#external-property-values)

In this example, the **maxItems** property of the component will be set to *7* at the time the component is rendered:
在此示例中，组件的 **maxItems* *属性将设置为 *7*

    {% component 'demoTodoAlias' maxItems='7' %}

> **注意**: 不是所有的组件在渲染时支持将变量传递给组件。

<a name="customizing-default-markup"></a>
## 自定义默认标记

组件提供的标记通常用作组件的使用示例。在某些情况下，您可能希望修改组件的外观和内容。 [将默认标记移动到主题部分](＃moving-default-markup)修改这个组件。 [覆盖组件部分](#overriding-partials)对于改为自定义的内容是很方便的。

<a name="moving-default-markup"></a>
### 将默认标签移动到partial

每个组件都可以有一个名为 **default.htm** 的入口文件部分，它在调用`{％component％}`标标签时展示，在下面的例子中我们假设该组件叫做 **blogPost**。

    url = "blog/post"

    [blogPost]
    ==
    {% component "blogPost" %}

输出的内容将根据插件中**components/blogpost/default.htm**文件的内容进行渲染。您可以从该文件中复制所有内容并将其直接粘贴到page(页面)中，或者粘贴到名为 **blog-post.htm** 的新partial(部分)。

    <h1>{{ __SELF__.post.title }}</h1>
    <p>{{ __SELF__.post.description }}</p>

在文件的内部，您可能会注意到引用了一个名为`__SELF__`的变量，这是指组件对象本身，应该替换为页面上使用的组件别名，在本例中它是`blogPost`。如下：

    <h1>{{ blogPost.post.title }}</h1>
    <p>{{ blogPost.post.description }}</p>

在主题内任何位置使用组件只修改这个partial就可以，现在可以在主题或者页面中使用这个带有component的partial了。

    {% partial 'blog-post.htm' %}

可以对组件部分目录中找到的所有其他部分重复此过程。

<a name="overriding-partials"></a>
### 覆盖组件部分

All component partials can be overridden using the theme partials. If a component called **channel** uses the **title.htm** partial.
可以使用theme主题部分覆盖所有component组件部分。如果名为 **channel** 的组件使用 **title.htm** partial。

    url = "mypage"

    [channel]
    ==
    {% component "channel" %}

我们可以通过在我们的主题中创建一个名为 **partials/channel/title.htm** 的文件来覆盖partial文件。

文件路径分解如下

文件夹名称 |描述
------------- | -------------
**partials** | 主题部分目录
**channel** | 组件别名（partial部分子目录）
**title.htm** | 部分要覆盖的组件

通过简单地为组件分配同名别名，可以将partial部分子目录名称自定义为任何名称。例如，为**channel**组件分配不同的别名**foobar**，覆盖目录也会更改：

    [channel foobar]
    ==
    {% component "foobar" %}

现在我们可以通过在我们的主题中创建一个名为**partials/foobar/title.htm**的来覆盖**title.htm**。

<a name="viewbag-component"></a>
## “查看包”组件

October有一个名为`viewBag`的特殊组件，可用于任何页面或布局。它允许在标记区域内轻松定义和访问ad hoc属性作为变量。一个很好的用法示例是在页面内定义活动菜单项：

    title = "About"
    url = "/about.html"
    layout = "default"

    [viewBag]
    activeMenu = "about"
    ==

    <p>Page content...</p>

然后，在页面、布局或部分标记内部使用`viewBag`变量调用组件定义的任何属性都恶意。例如，在此布局中，如果`viewBag.activeMenu`值设置为 **about**，则**active**class将添加到列表项中：

    description = "Default layout"
    ==
    [...]

    <!-- Main navigation -->
    <ul>
        <li class="{{ viewBag.activeMenu == 'about' ? 'active' }}">About</li>
        [...]
    </ul>

> **注意**: viewBag组件在管理后台是看不到的，仅适用于基于文件的编辑。它也可以被其他插件用来存储数据。