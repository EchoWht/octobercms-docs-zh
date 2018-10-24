# AJAX事件处理程序

- [AJAX处理程序](#ajax-handlers)
    - [调用处理程序](#calling-handlers)
- [AJAX处理程序中的重定向](#redirections-in-handlers)
- [从AJAX处理程序返回数据](#returning-data-from-handlers)
- [Throwing an AJAX exception](#throw-ajax-exception)
- [抛出一个AJAX异常](#before-handler)

<a name="ajax-handlers"></a>
## AJAX处理程序

AJAX事件处理程序是PHP函数，可以在页面或布局[PHP部分](../cms/themes＃php-section)或[components](../cms/components)中定义。处理程序名称应具有以下格式：`onName`。所有处理程序都支持使用[更新partials](../ajax/update-partials)作为AJAX请求的一部分。

    function onSubmitContactForm()
    {
       //...
    }

如果在页面和布局中一起定义了两个具有相同名称的函数，则将执行页面处理程序。 [components](../cms/components)中定义的处理程序具有最低优先级。

<a name="calling-handlers"></a>
### 调用处理程序

每个AJAX请求都应该使用[data attributes API](../ajax/attributes-api)或[JavaScript API](../ajax/javascript-api)指定处理程序名称。发出请求后，服务器将搜索所有已注册的处理程序并找到它找到的第一个处理程序。

    <!-- Attributes API -->
    <button data-request="onSubmitContactForm">Go</button>

    <!-- JavaScript API -->
    <script> $.request('onSubmitContactForm') </script>

如果两个组件注册相同的处理程序名称，建议在处理程序前加上[组件短名称或别名](../cms/components＃aliases)。如果组件使用** mycomponent **的别名，则可以使用`mycomponent::onName`来定位处理程序。

    <button data-request="mycomponent::onSubmitContactForm">Go</button>

如果用户更改页面上使用的组件别名，您可能需要使用[`__SELF__`](https://octobercms.com/docs/plugin/components#referencing-self)引用变量而不是硬编码别名。

    <form data-request="{{ __SELF__ }}::onCalculate" data-request-update="'{{ __SELF__ }}::calcresult': '#result'">

#### 通用处理程序

有时您可能需要发出AJAX请求，其唯一目的是更新页面内容，而不需要执行任何代码。为此，您可以使用`onAjax`处理程序。此处理程序随处可用，无需编写任何代码。(译者注：这个有什么作用呢？)

    <button data-request="onAjax">什么也不做</button>

<a name="redirections-in-handlers"></a>
## AJAX处理程序中的重定向

如果需要将浏览器重定向到另一个链接，请从AJAX处理程序返回`Redirect`对象。一旦从服务器返回响应，框架将重定向浏览器。示例AJAX处理程序：

    function onRedirectMe()
    {
        return Redirect::to('http://google.com');
    }

<a name="returning-data-from-handlers"></a>
## 从AJAX处理程序返回数据

通常情况下，您可能希望从AJAX处理程序返回结构化数据。如果AJAX处理程序返回一个数组，您可以在`success`事件处理程序中访问它的元素。示例AJAX处理程序：

    function onFetchDataFromServer()
    {
       /* Some server-side code */

        return [
            'totalUsers' => 1000,
            'totalProjects' => 937
        ];
    }

可以使用数据属性API获取数据：

    <form data-request="onHandleForm" data-request-success="console.log(data)">

与JavaScript API相同：

    <form
        onsubmit="$(this).request('onHandleForm', {
            success: function(data) {
                console.log(data);
            }
        }); return false;">

<a name="throw-ajax-exception"></a>
## 抛出一个AJAX异常

您可以使用`AjaxException`类抛出[AJAX异常](../services/error-log#ajax-exception)，响应为错误，同时保留正常发送响应内容的功能。只需将响应内容作为异常的第一个参数传递。

    throw new AjaxException([
        'error' => 'Not enough questions',
        'questionsNeeded' => 2
    ]);

> **注意**: 抛出此异常类型[partials将更新](../ajax/update-partials)正常。

<a name="before-handler"></a>
## 在处理程序之前运行代码

有时您可能希望在处理程序执行之前执行代码。将[onInit]函数定义为[页面执行生命周期](../cms/layouts＃dynamic-pages)的一部分，允许代码在每个AJAX处理程序之前运行。

    function onInit()
    {
       //来自页面或者布局
    }

您可以在[组件类](../plugin/components＃page-cycle-init)或[后台小部件类](../backend/widgets)中定义`init`方法。

    function init()
    {
       //来自组件或者小部件
    }
