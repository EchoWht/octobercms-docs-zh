# 表单和HTML

- [介绍](#introduction)
- [打开表单](#opening-a-form)
- [表单令牌](#form-tokens)
- [CSRF保护](#csrf-protection)
- [表单模型绑定](#form-model-binding)
- [标签Labels](#labels)
- [文本字段](#text)
- [复选框和单选按钮](#checkboxes-and-radio-buttons)
- [文件选择](#file-input)
- [数字输入](#number)
- [下拉列表](#drop-down-lists)
- [按钮](#buttons)
- [自定义宏](#custom-macros)

<a name="introduction"></a>
## 介绍

October提供各种有用的函数与`Html`facade，用于处理HTML和表单。 虽然大多数示例都将使用PHP语言，但所有这些功能都可以通过简单的转换直接转换为[Twig标记](../markup)。

    // PHP
    <?= Form::open(..) ?>

    // Twig
    {{ form_open(...) }}

如上所示，在Twig中，所有以`form_`为前缀的函数将直接绑定到`Form`facade，并使用*snake_case下划线格式*提供对方法的访问。 有关使用前端中的表单帮助程序的信息，请参阅[标记指南以获取更多信息](../markup/function-form)。

<a name="opening-a-form"></a>
## 打开表单

可以使用`Form::open`方法打开表单，该方法将一组属性作为第一个参数传递：

    <?= Form::open(['url' => 'foo/bar']) ?>
        //
    <?= Form::close() ?>

默认情况下，将假定使用`POST`方法，但是，您可以自由指定另一种方法：

    Form::open(['url' => 'foo/bar', 'method' => 'put'])

> **注意:** 由于HTML表单只支持`POST`和`GET`，因此`PUT`和`DELETE`方法将通过自动在表单中添加`_method`隐藏字段。

您也可以传递常规HTML属性：

    Form::open(['url' => 'foo/bar', 'class' => 'pretty-form'])

如果您的表单要接受文件上传，请在您的数组中添加`files`选项：

    Form::open(['url' => 'foo/bar', 'files' => true])

您还可以打开指向页面或组件中的处理程序方法的表单：

    Form::open(['request' => 'onSave'])

#### 支持AJAX的表单

同样，可以使用`Form::ajax`方法打开支持AJAX的表单，其中第一个参数是处理程序方法名称：

    Form::ajax('onSave')

`Form::ajax`的第二个参数应该包含以下属性：

    Form::ajax('onSave', ['confirm' => 'Are you sure?'])

您还可以将partials作为另一个数组传递给更新：

    Form::ajax('onSave', ['update' => [
            'control-panel' => '#controlPanel',
            'layout/sidebar' => '#layoutSidebar'
        ]
    ])

> **注意**: 通过删除`data-request -`前缀，大多数[来自AJAX框架的数据属性](../ajax/attributes-api)可用。

<a name="form-tokens"></a>
## 表单令牌

#### CSRF保护

如果你[保护已启用](../setup/configuration#csrf-protection)，使用带有`POST`，`PUT`或`DELETE`的`Form::open`方法将自动为表单添加一个CSRF令牌 作为一个隐藏的领域。 或者，如果您希望为隐藏的CSRF字段生成HTML，则可以使用`token`方法：

    <?= Form::token() ?>

#### 延迟绑定会话密钥

用于[延迟绑定](../database/relations#deferred-binding) 的会话密钥将作为隐藏字段添加到每个表单。 如果要手动生成此字段，可以使用`sessionKey`方法：

    <?= Form::sessionKey() ?>

<a name="form-model-binding"></a>
## 表单模型绑定

#### 打开模型表单

您可能希望根据模型的内容填充表单。 为此，请使用`Form::model`方法：

    <?= Form::model($user, ['id' => 'userForm']) ?>

现在，当您生成表单元素(如文本输入)时，与字段名称匹配的模型值将自动设置为字段值。 例如，对于名为`email`的文本输入，用户模型的`email`属性将被设置为值。 如果会话闪存数据中的项目与输入名称匹配，则该项目将优先于模型的值。 优先级如下：

1. 会话闪存数据(旧输入)
2. 明确地传递价值
3. 模型属性数据
4. 现有的回发价值

这使您可以快速构建不仅绑定到模型值的表单，而且如果服务器上存在验证错误，则可以轻松地重新填充。 您可以使用`Form::value`手动访问这些值：

    <input type="text" name="name" value="<?= Form::value('name') ?>" />

您可以传递默认值作为第二个参数：

    <?= Form::value('name', 'John Travolta') ?>

> **注意:** 使用`Form::model`时，请务必使用`Form::close`关闭表单！

<a name="labels"></a>
## Labels标签

#### 生成标签元素

    <?= Form::label('email', 'E-Mail Address') ?>

#### 指定额外的HTML属性

    <?= Form::label('email', 'E-Mail Address', ['class' => 'awesome']) ?>

> **注意:** 创建标签后，使用与标签名称匹配的名称创建的任何表单元素都将自动接收与标签名称匹配的ID。

<a name="text"></a>
## 文本字段

#### 生成文本输入

    <?= Form::text('username') ?>

#### 指定默认值

    <?= Form::text('email', 'emailaddress@example.com') ?>

> **注意:** *hidden*和*textarea*方法与* text *方法具有相同的签名。

#### 生成密码输入

    <?= Form::password('password') ?>

#### 生成其他输入

    <?= Form::email($name, $value = null, $attributes = []) ?>
    <?= Form::file($name, $attributes = []) ?>

<a name="checkboxes-and-radio-buttons"></a>
## 复选框和单选按钮

#### 生成复选框和单选按钮

    <?= Form::checkbox('name', 'value') ?>

    <?= Form::radio('name', 'value') ?>

#### 生成已选中的复选框或单选按钮

    <?= Form::checkbox('name', 'value', true) ?>

    <?= Form::radio('name', 'value', true) ?>

<a name="number"></a>
## 数字

#### 生成数字输入

    <?= Form::number('name', 'value') ?>

<a name="file-input"></a>
## 文件选择

#### 生成一个文件选择框

    <?= Form::file('image') ?>

> **注意:** 必须打开表单，并将`files`选项设置为`true`。

<a name="drop-down-lists"></a>
## 下拉列表

#### 生成一个下拉列表

    <?= Form::select('size', ['L' => 'Large', 'S' => 'Small']) ?>

#### 生成具有所选默认值的下拉列表

    <?= Form::select('size', ['L' => 'Large', 'S' => 'Small'], 'S') ?>

#### 生成分组列表

    <?= Form::select('animal', [
        'Cats' => ['leopard' => 'Leopard'],
        'Dogs' => ['spaniel' => 'Spaniel'],
    ]) ?>

#### 生成带范围的下拉列表

    <?= Form::selectRange('number', 10, 20) ?>

#### 生成包含范围，选定值和空白选项的下拉列表

    <?= Form::selectRange('number', 10, 20, 2, ['emptyOption' => 'Choose...']) ?>
    
#### 生成包含月份名称的列表

    <?= Form::selectMonth('month') ?>

#### 生成包含月份名称，选定值和空白选项的列表

    <?= Form::selectMonth('month', 2, ['emptyOption' => 'Choose month...']) ?>
    
<a name="buttons"></a>
## 按钮

#### 生成提交按钮

    <?= Form::submit('Click Me!') ?>

> **注意:** 需要创建一个按钮元素？ 尝试*按钮*方法。 它与*submit*具有相同的签名。

<a name="custom-macros"></a>
## 自定义宏

#### 注册表单宏

很容易定义自己的自定义Form类助手，称为“宏”。 这是它的工作原理。 首先，只需使用给定名称和Closure注册宏：

    Form::macro('myField', function() {
        return '<input type="awesome">';
    })

现在您可以使用其名称调用您的宏：

#### 调用自定义表单宏

    <?= Form::myField() ?>
