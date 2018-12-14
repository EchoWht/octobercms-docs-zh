# AJAX JavaScript API

- [JavaScript API](#javascript-api)
- [使用示例](#javascript-examples)
- [全局AJAX事件](#global-events)
- [使用示例](#global-events-examples)

<a name="javascript-api"></a>
## JavaScript API

JavaScript API比数据属性API更强大。 `request`方法可以与表单内部或表单元素上的任何元素一起使用。当该方法与表单内的元素一起使用时，它将被转发到表单。

`request`方法有一个必需的参数--AJAX处理程序名称。例：

    <form onsubmit="$(this).request('onProcess'); return false;">
        ...

`request`方法的第二个参数是options对象。您可以使用与[jQuery AJAX函数](http://api.jquery.com/jQuery.ajax/)兼容的任何选项和方法。以下选项特定于October框架：

选项 | 描述
------------- | -------------
**update** | 一个对象，指定要更新的列表部分和页面元素(作为CSS选择器)：{'partial'：'#select'}。 如果选择器字符串前面带有“@”符号，则从服务器接收的内容将附加到元素，而不是替换现有内容。
**confirm** | 确认字符串。 如果设置，则在发送请求之前显示确认。 如果用户单击“取消”按钮，则取消请求。
**data** | 一个可选对象，指定要与表单数据一起发送到服务器的数据：{key：'value'}。
**redirect** | string，指定在成功请求后将浏览器重定向到的URL。
**beforeUpdate** | 在更新页面元素之前执行的回调函数。 该函数获取3个参数：从服务器接收的数据对象，文本状态字符串和jqXHR对象。 函数内的`this`变量解析为请求内容 - 一个包含2个属性的对象：`handler`和`options`表示原始的request()参数。
**success** | 在成功请求的情况下执行的回调函数。 如果提供此选项，它将覆盖默认框架的功能：元素未更新，未触发`beforeUpdate`事件，不会触发`ajaxUpdate`和`ajaxUpdateComplete`事件。 事件处理程序获取3个参数：从服务器接收的数据对象，文本状态字符串和jqXHR对象。 但是，您仍然可以调用函数内部调用`this.success(...)`的默认框架功能。
**error** | 回调函数在发生错误时执行。 默认情况下，会显示警报消息。 如果覆盖此选项，则不会显示警报消息。 处理程序获取3个参数：jqXHR对象，文本状态字符串和错误对象 - 请参阅[jQuery AJAX函数](http://api.jquery.com/jQuery.ajax/)。
**complete** | 回调函数在成功或错误的情况下执行。
**form** | 一个表单元素，用于获取随请求一起发送的表单数据，作为选择器字符串或表单元素传递。
**flash** | 如果为true，则指示服务器清除并发送带响应的任何Flash消息。 默认值：false
**files** | 如果为true，请求将接受文件上传，这需要浏览器支持`FormData`接口。 默认值：false
**loading** | 请求运行时要显示的可选字符串或对象。 字符串应该是元素的CSS选择器，对象应该支持`show()`和`hide()`函数来管理可见性。 使用[framework extras](ajax-extras.md)时，可以传递全局对象`$.oc.stripeLoadIndicator`。

您还可以通过将新函数作为选项传递来覆盖某些请求逻辑。这些逻辑处理程序可用。

操作 | 描述
------------- | -------------
**handleConfirmMessage(message)** | 在请求用户确认时调用。
**handleErrorMessage(message)** | 应在显示错误消息时调用。
**handleValidationMessage(message, fields)** | 使用验证时，会聚焦第一个无效字段。
**handleFlashMessage(message, type)** | 使用**flash**选项提供flash消息时调用(参见上文)。
**handleRedirectResponse(url)** | 当浏览器重定向到另一个位置时调用。

<a name="javascript-examples"></a>
## 使用示例

在发送onDelete请求之前请求确认:

    $('form').request('onDelete', {
        confirm: 'Are you sure?',
        redirect: '/dashboard'
    })

请求`onCalculate`方法后，用**result** CSS类将渲染的**calcresult**部分注入到页面元素:

    $('form').request('onCalculate', {
        update: {calcresult: '.result'}
    })

请求`onCalculate`的时候传递额外的数据到后台:

    $('form').request('onCalculate', {data: {value: 55}})

在请求`onCalculate`方法更新页面元素之前执行一些js方法:

    $('form').request('onCalculate', {
        update: {calcresult: '.result'},
        beforeUpdate: function() { /* 在这里可以写一些js */ }
    })

如果`onCalculate`请求成功后，可以编写一些自定义的js方法，以及调用默认的`success`方法

    $('form').request('onCalculate', {success: function(data) {
        //...写在这...
        this.success(data);
    }})

执行没有表单元素的请求:

    $.request('onCalculate', {
        success: function() {
            console.log('Finished!');
        }
    })

运行`onCalculate`处理程序，如果成功，在默认的`success`函数完成后运行一些自定义代码：

    $('form').request('onCalculate', {success: function(data) {
        this.success(data).done(function() {
            //... do something after parent success() is finished ...
        });
    }})

<a name="global-events"></a>
## 全局AJAX事件

AJAX框架在更新的元素，触发元素，窗体和窗口对象上触发多个事件。 无论使用哪种API(数据属性API或JavaScript API)，都会触发事件。

事件 | 描述
------------- | -------------
**ajaxBeforeSend** | 在发送请求之前在窗口对象上触发。
**ajaxBeforeUpdate** | 在请求完成后，但在页面更新之前，直接在表单对象上触发。 处理程序获取5个参数：事件对象，上下文对象，从服务器接收的数据对象，状态文本字符串和jqXHR对象。
**ajaxUpdate** | 在使用框架更新页面元素后触发。 处理程序获取5个参数：事件对象，上下文对象，从服务器接收的数据对象，状态文本字符串和jqXHR对象。
**ajaxUpdateComplete** | 在框架更新所有元素之后在窗口对象上触发。 处理程序获取5个参数：事件对象，上下文对象，从服务器接收的数据对象，状态文本字符串和jqXHR对象。
**ajaxSuccess** | 请求成功完成后在表单对象上触发。 处理程序获取5个参数：事件对象，上下文对象，从服务器接收的数据对象，状态文本字符串和jqXHR对象。
**ajaxError** | 如果请求遇到错误，则在表单对象上触发。 处理程序获取5个参数：事件对象，上下文对象，错误消息，状态文本字符串和jqXHR对象。
**ajaxErrorMessage** | 如果请求遇到错误，则在窗口对象上触发。 处理程序获取2个参数：从服务器返回的事件对象和错误消息。
**ajaxConfirmMessage** | 当给出“confirm”选项时，在窗口对象上触发。 处理程序获取2个参数：作为`confirm`选项的一部分分配给处理程序的事件对象和文本消息。 这对于实现自定义确认逻辑/接口而不是本机javascript确认框非常有用。

这些事件在触发元素上触发:

事件 | 描述
------------- | -------------
**ajaxSetup** | 在请求形成之前触发，允许通过`context.options`对象修改选项。
**ajaxPromise** | 在发送AJAX请求之前直接触发。
**ajaxFail** | 如果AJAX请求失败，最终会触发。
**ajaxDone** | 如果AJAX请求成功，最终会触发。
**ajaxAlways** | 无论AJAX请求失败还是成功，都会触发。

<a name="global-events-examples"></a>
## 使用示例

当在元素上触发`ajaxUpdate`事件时执行JavaScript代码。

    $('.calcresult').on('ajaxUpdate', function() {
        console.log('Updated!');
    })

使用逻辑处理程序执行显示Flash消息的单个请求。

    $.request('onDoSomething', {
        flash: 1,
        handleFlashMessage: function(message, type) {
            $.oc.flashMsg({ text: message, class: type })
        }
    })

全局应用配置到所有AJAX请求。

    $(document).on('ajaxSetup', function(event, context) {
        // 在所有AJAX请求上启用对消息的AJAX处理
        context.options.flash = true

        // 在所有AJAX请求上启用StripeLoadIndicator
        context.options.loading = $.oc.stripeLoadIndicator

        // 通过触发类型错误的flashMsg处理错误消息
        context.options.handleErrorMessage = function(message) {
            $.oc.flashMsg({ text: message, class: 'error' })
        }

        // 通过触发消息类型的flashMsg来处理Flash消息
        context.options.handleFlashMessage = function(message, type) {
            $.oc.flashMsg({ text: message, class: type })
        }
    })
