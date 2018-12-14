# form()

以`form_`为前缀的函数执行在处理表单时有用的任务。 帮助程序直接映射到`Form` PHP类及其方法。 例如：

    {{ form_close() }}

是PHP的等价物如下：

    <?= Form::close() ?>

> **注意**: *camelCase驼峰命名*中的方法应转换为*snake_case下划线命名*。

## form_open()

输出标准FORM开口标签(非AJAX)。

    {{ form_open() }}

属性可以在第一个参数中传递。

    {{ form_open({ class: 'form-horizontal' }) }}

上面的示例将输出如下：

    <form class="form-horizontal">

有一些特殊选项也可以与属性一起使用。

    {{ form_open({ request: 'onUpdate' }) }}

该功能支持以下选项：

选项 | 描述
------------- | -------------
**method** | 请求方法。 对应**method**FORM标签属性。 例如：POST，GET，PUT，DELETE
**request** | 发布表单时在服务器上执行的处理程序名称。 有关事件处理程序的详细信息，请参阅[处理表单](cms-pages.md#handling-forms) 文章。
**url** | 指定将表单发布到的URL。 对应**action** FORM标签属性。
**files** | 确定表单是否将提交文件。 可接受的值：**true**和**false**。
**model** | 表单模型绑定的模型对象。

## form_ajax()

输出启用AJAX的FORM开始标签。 `form_ajax()`函数的第一个参数是AJAX处理程序名称。 处理程序可以在布局或页面[PHP部分](cms-themes.md#php-section) 代码中定义，也可以在组件中定义。 您可以在[AJAX Framework](ajax-introduction.md) 文章中找到有关AJAX的更多信息。

    {{ form_ajax('onUpdate') }}

属性可以在第二个参数中传递。

    {{ form_ajax('onSave', { class: 'form-horizontal'}) }}

上面的示例将输出如下：

    <form data-request="onSave" class="form-horizontal">

有一些特殊选项也可以与属性一起使用。

    {{ form_ajax('onDelete', { data: { id: 2 }, confirm: 'Really delete this record?' }) }}

    {{ form_ajax('onRefresh', { update: { statistics: '#statsPanel' } }) }}

该功能支持以下选项：

选项 | 描述
------------- | -------------
**success** | 要在成功结果上执行的JavaScript字符串。
**error** | 要在失败的结果上执行的JavaScript字符串。
**confirm** | 发送请求之前要显示的确认消息。
**redirect** | 成功结果后，重定向到URL。
**update** | 成功更新的部分数组，格式如下：{'partial':'#element'}。
**data** | 请求中包含的额外数据采用以下格式：{'myvar':'myvalue'}。

## form_close()

输出标准FORM结束标签。 此标签通常可用于提供使用的一致性。

    {{ form_close() }}

上面的示例将输出如下：

    </form>
