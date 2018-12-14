# AJAX 入门

- [介绍](#introduction)
- [引入AJAX框架](#framework-script)
- [AJAX请求是如何工作的](#how-ajax-works)
- [使用示例](#usage-example)

<a name="introduction"></a>
## 介绍

ctober包含一jquery框架，它带来了一整套AJAX功能，允许您从服务器加载数据而无需刷新浏览器页面。可以在[CMS主题](cms-themes.md)和[后端管理区域](backend-controllers-ajax.md#ajax)中的任何位置使用相同的库。

AJAX框架有两种形式，您可以使用[JavaScript API](ajax-javascript-api.md)或[数据属性API](ajax-attributes-api.md)。数据属性API不需要任何JavaScript知识就可以在October中使用AJAX。

<a name="framework-script"></a>
## 引入AJAX框架

AJAX框架在您的[CMS主题](cms-themes.md)中是可选的，要使用您应该包含它的库，将`{％framework％}`标记放在[page]中的任何位置(../cms)/pages)或[layout](cms-layouts.md)。这增加了对October前端JavaScript库的引用。该库需要jQuery，因此应首先加载它，例如：

    <script src="{{ 'assets/javascript/jquery.js'|theme }}"></script>

    {% framework %}

`{％framework％}`标签支持可选的**extras**参数。如果指定了此参数，则标记会为[extra features](ajax-extras.md)添加StyleSheet和JavaScript文件，包括表单验证和加载指示符。

    {% framework extras %}

<a name="how-ajax-works"></a>
## AJAX请求是如何工作的

页面可以发出由数据属性提示或使用JavaScript的AJAX请求。每个请求在服务器上调用**事件处理程序** - 也称为[AJAX处理程序](ajax-handlers.md) - 并且可以使用partials更新页面元素。 AJAX请求最适合表单，因为表单数据会自动发送到服务器。这是请求工作流程：

1. 客户端浏览器通过提供处理程序名称和其他可选参数来发出AJAX请求。
2. 服务器找到[AJAX处理程序](ajax-handlers.md)并执行它。
3. 处理程序执行所需的业务逻辑，并通过注入页面变量来更新环境。
4. 客户端使用`update`选项请求的服务器[渲染部分](ajax-update-partials.md)。
5. 服务器发送包含呈现的部分标记的响应。
6. 客户端框架使用从服务器接收的部分数据更新页面元素。

>**注意**: 根据页面上下文，将呈现[CMS部分](cms-partials.md)或[backend partial](backend-controllers-ajax.md)视图。

<a name="usage-example"></a>
## 使用示例

下面是一个使用数据属性API定义启用AJAX的表单的简单示例。该表单将向**onTest**处理程序发出一个AJAX请求，并请求使用**mypartial**部分标记更新结果。

    <!-- 支持AJAX的表单 -->
    <form data-request="onTest" data-request-update="mypartial: '#myDiv'">

        <!-- 输入两个值 -->
        <input name="value1"> + <input name="value2">

        <!-- Action button -->
        <button type="submit">计算</button>

    </form>

    <!-- 结果 -->
    <div id="myDiv"></div>

>**注意**: `value1`和`value2`的表单数据随AJAX请求自动发送。

**mypartial**partial包含读取`result`变量的标记。

    结果是 {{ result }}

**onTest**处理程序方法使用`input` 的[helper方法](../services/helper#method-input)获取表单数据，结果传递给`result`页面变量。

    function onTest()
    {
        $this->page['result'] = input('value1') + input('value2');
    }

这个例子可以这样理解：“当提交表单时，向**onTest**处理程序发出一个AJAX请求。当处理程序完成时，渲染**mypartial**partial并将其内容注入**#myDiv**元素。“
