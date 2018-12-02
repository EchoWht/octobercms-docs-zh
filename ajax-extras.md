# AJAX额外功能

- [进度条](#loader-stripe)
- [表单校验](#ajax-validation)
    - [抛出一个校验错误](#throw-validation-exception)
    - [显示错误信息](#error-messages)
    - [根据字段显示错误](#field-errors)
- [加载按钮](#loader-button)
- [Flash消息](#ajax-flash)
- [使用示例](#usage-example)

使用AJAX框架时，您可以选择指定** extras **后缀，其中包含其他StyleSheet和JavaScript文件。 在前端CMS页面中处理AJAX请求时，这些功能非常有用。

    {% framework extras %}

<a name="loader-stripe"></a>
## 进度条

T您应注意的第一个功能是在运行AJAX请求时显示在页面顶部的加载指示器。 该指标挂钩到AJAX框架使用的[global events](../ajax/javascript-api#global-events)。

当AJAX请求启动时，会触发“ajaxPromise”事件，该事件显示指示符并将鼠标光标置于加载状态。 `ajaxFail`和`ajaxDone`事件用于检测请求何时完成，指示符再次隐藏。

<a name="ajax-validation"></a>
## 表单校验

您可以在表单上指定`data-request-validate`属性以启用验证功能。

    <form
        data-request="onSubmit"
        data-request-validate>
        <!-- ... -->
    </form>

<a name="throw-validation-exception"></a>
### 抛出一个校验错误

在服务器端AJAX处理程序中，您可以使用`ValidationException`类抛出[validation exception](../services/error-log＃validation-exception)以使字段无效，其中第一个参数是数组。 该数组应使用键的字段名称和值的错误消息。

    function onSubmit()
    {
        throw new ValidationException(['name' => 'Name 字段不能为空~']);
    }

> **注意**: 您还可以传递[validation service](../services/validation)的实例作为异常的第一个参数。

<a name="error-messages"></a>
### 显示错误信息

在表单内部，您可以使用容器元素上的`data-validate-error`属性显示第一条错误消息。 容器内的内容将设置为错误消息，并且元素将可见。

    <div data-validate-error></div>

要显示多个错误消息，请包含具有`data-message`属性的元素。 在此示例中，段落标记将被复制并设置为存在的每条消息的内容。

    <div class="alert alert-danger" data-validate-error>
        <p data-message></p>
    </div>

要在AJAX失效上添加自定义类，请挂钩`ajaxInvalidField`和`ajaxPromise` JS事件。

    $(window).on('ajaxInvalidField', function(event, fieldElement, fieldName, errorMsg, isFirst) {
        $(fieldElement).closest('.form-group').addClass('has-error');
    });
    
    $(document).on('ajaxPromise', '[data-request]', function() {
        $(this).closest('form').find('.form-group.has-error').removeClass('has-error');
    });

<a name="field-errors"></a>
### 根据字段显示错误

或者，您可以通过定义使用`data-validate-for`属性的元素，将字段名称作为值来显示单个字段的验证消息。

    <!-- Input 字段 -->
    <input name="phone"/>

    <!-- 该字段的验证消息 -->
    <div data-validate-for="phone"></div>

如果元素保留为空，则将使用服务器中的验证文本填充该元素。 否则，您可以指定任何您喜欢的文本。

    <div data-validate-for="phone">
        哎呀..电话号码无效！
    </div>

<a name="loader-button"></a>
## 加载按钮

当任何元素包含`data-attach-loading`属性时，在AJAX请求期间将向其添加CSS类`oc-loading`。 这个类将使用`:after` CSS选择器在按钮和锚元素上生成一个*loading spinner*。

    <form data-request="onSubmit">
        <button data-attach-loading>
            Submit
        </button>
    </form>

    <a
        href="#"
        data-request="onDoSomething"
        data-attach-loading>
        Do something
    </a>

<a name="ajax-flash"></a>
## Flash消息

在表单上指定`data-request-flash`属性，以便在成功的AJAX请求中使用flash消息。

    <form
        data-request="onSuccess"
        data-request-flash>
        <!-- ... -->
    </form>

结合在事件处理程序中使用`Flash`facade，在请求完成后将出现一条flash消息。

    function onSuccess()
    {
        Flash::success('You did it!')
    }

为了与基于AJAX的闪存消息保持一致，您可以通过将此代码放入页面或布局来在页面加载时呈现[标准flash消息](../markup/tag-flash)。

    {% flash %}
        <p
            data-control="flash-message"
            class="flash-message fade {{ type }}"
            data-interval="5">
            {{ message }}
        </p>
    {% endflash %}

<a name="usage-example"></a>
## 使用示例

下面是表单验证的完整示例。 它调用`onDoSomething`事件处理程序，触发加载提交按钮，对表单字段执行验证，然后显示成功的flash消息。

    <form
        data-request="onDoSomething"
        data-request-validate
        data-request-flash>

        <div>
            <input name="name"/>
            <span data-validate-for="name"></span>
        </div>

        <div>
            <input name="email"/>
            <span data-validate-for="email"></span>
        </div>

        <button data-attach-loading>
            Submit
        </button>

        <div class="alert alert-danger" data-validate-error>
            <p data-message></p>
        </div>

    </form>

AJAX事件处理程序查看客户端发送的POST数据，并将一些规则应用于验证程序。 如果验证失败，则抛出`ValidationException`，否则返回`Flash::success`消息。

    function onDoSomething()
    {
        $data = post();

        $rules = [
            'name' => 'required',
            'email' => 'required|email',
        ];

        $validation = Validator::make($data, $rules);

        if ($validation->fails()) {
            throw new ValidationException($validation);
        }

        Flash::success('Jobs done!');
    }
