# AJAX数据属性API

- [AJAX数据属性API](#data-attributes)
- [使用示例](#data-attribute-examples)

<a name="data-attributes"></a>
## AJAX数据属性API

数据属性API允许您在没有任何JavaScript的情况下发出AJAX请求。在许多情况下，数据属性API比JavaScript API更简洁 - 您编写更少的代码来获得相同的结果。支持的AJAX数据属性是：

Attribute | Description
------------- | -------------
**data-request** | 指定AJAX处理程序名称。
**data-request-confirm** | 使用确认弹窗。在发送请求之前显示确认。如果用户单击“取消”按钮，则不会发送请求
**data-request-redirect** | 指定在成功的AJAX请求后重定向浏览器的URL。
**data-request-update** | 指定要更新的部分和页面元素(CSS选择器)的列表。格式如下：`partial：selector,partial：selector`。在某些情况下需要使用引号，例如：`'my-partial'：'#myelement'`。如果选择器字符串前面带有`@`符号，则从服务器接收的内容将附加到元素，而不是替换现有内容。如果选择器字符串前面带有“^”符号，则内容将被添加到前面。
**data-request-data** | 指定要发送到服务器的其他POST参数。格式如下：`var：value，var：value`。如果需要，请使用引号：`var：'some string'`。该属性可以在触发元素上使用，例如在具有`data-request`属性的按钮上，在触发元素的最近元素和父表单元素上。框架合并了`data-request-data`属性的值。如果不同元素上的属性定义具有相同名称的参数，则框架使用以下优先级：触发元素`data-request-data`，更接近的父元素`data-request-data`，表单输入数据。
**data-request-before-update** | 指定在更新页面内容之前直接执行的JavaScript代码。在JavaScript代码中，您可以访问以下变量：`this`(页面元素触发请求)，`context`对象，从服务器接收的`data`对象，`textStatus`文本字符串和`jqXHR `对象。
**data-request-success** | 指定在成功完成请求后要执行的JavaScript代码。在JavaScript代码中，您可以访问以下变量：`this`(页面元素触发请求)，`context`对象，从服务器接收的`data`对象，`textStatus`文本字符串和`jqXHR `对象。
**data-request-error** | 指定在请求遇到错误时要执行的JavaScript代码。在JavaScript代码中，您可以访问以下变量：`this`(页面元素触发请求)，`context`对象，`textStatus`文本字符串和`jqXHR`对象。
**data-request-complete** | 指定在请求成功完成或遇到错误时要执行的JavaScript代码。在JavaScript代码中，您可以访问以下变量：`this`(页面元素触发请求)，`context`对象，`textStatus`文本字符串和`jqXHR`对象。
**data-request-loading** | 为请求运行时要显示的元素指定CSS选择器。您可以使用此选项显示AJAX加载指示器。该功能使用jQuery的`show()`和`hide()`函数来管理元素可见性。
**data-request-form** | 明确指定用于获取表单数据的表单元素。如果未指定，则使用与触发元素最接近的元素，包括元素本身是否为表单。
**data-request-flash** | 指定此选项时，指示服务器清除并发送带响应的Flash消息。 [附加功能](ajax-extras.md#ajax-flash)也使用此选项。
**data-request-files** | 当指定请求将接受文件上传时，这需要浏览器支持`FormData`接口。
**data-track-input** | 可以应用于也具有`data-request`属性的文本，数字或密码输入字段。定义时，输入字段会在用户在字段中键入内容时自动发送AJAX请求。可选属性值可以定义框架在发送请求之前等待的间隔(以毫秒为单位)。

当为元素指定`data-request`属性时，该元素在用户与之交互时触发AJAX请求。根据元素的类型，将在以下事件上触发请求：

元素 | 事件
------------- | -------------
**Forms** | 当提交表单时。
**Links, buttons** | 当单击元素时。
**Text, number, and password fields** | 当文本被更改时，并且仅当呈现`data-track-input`属性时。
**Dropdowns, checkboxes, radios** | 当元素被选中时

<a name="data-attribute-examples"></a>
## 使用示例

提交表单时触发`onCalculate`处理程序。使用**calcresult** partial更新标识符为`result`的元素：

    <form data-request="onCalculate" data-request-update="calcresult: '#result'">

在发送请求之前单击“删除”按钮时请求确认：

    <form ... >
        ...
        <button data-request="onDelete" data-request-confirm="Are you sure?">Delete</button>

成功请求后重定向到另一个页面：

    <form data-request="onLogin" data-request-redirect="/admin">

成功请求后显示弹出窗口：

    <form data-request="onLogin" data-request-success="alert('Yay!')">

使用值`update`发送POST参数`mode`：

    <form data-request="onUpdate" data-request-data="mode: 'update'">

在多个元素之间发送值为“7”的POST参数`id`：

    <div data-request-data="id: 7">
        <button data-request="onDelete">Delete</button>
        <button data-request="onSave">Update</button>
    </div>

含有[文件上传](services-request-input.md#files)和请求：

    <form data-request="onSubmit" data-request-files>
        <input type="file" name="photo" accept="image/*"/>
        <button type="submit">Submit</button>
    </form>
