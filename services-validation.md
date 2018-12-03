# 验证

- [基本用法](#basic-usage)
- [使用错误消息](#working-with-error-messages)
- [错误消息和视图](#error-messages-and-views)
- [可用的验证规则](#available-validation-rules)
- [有条件地添加规则](#conditionally-adding-rules)
- [验证数组](#validating-arrays)
- [自定义错误消息](#custom-error-messages)
- [自定义验证规则](#custom-validation-rules)

<a name="basic-usage"></a>
## 基本用法

验证器类是一个简单，方便的工具，用于验证数据并通过`Validator`类检索验证错误消息。

#### 基本验证示例

    $validator = Validator::make(
        ['name' => 'Joe'],
        ['name' => 'required|min:5']
    );

传递给`make`方法的第一个参数是验证下的数据。 第二个参数是应该应用于数据的验证规则。

#### 使用数组指定规则

可以使用“管道”字符或作为数组的单独元素来分隔多个规则。

    $validator = Validator::make(
        ['name' => 'Joe'],
        ['name' => ['required', 'min:5']]
    );

#### 验证多个字段

    $validator = Validator::make(
        [
            'name' => 'Joe',
            'password' => 'lamepassword',
            'email' => 'email@example.com'
        ],
        [
            'name' => 'required',
            'password' => 'required|min:8',
            'email' => 'required|email|unique:users'
        ]
    );

一旦创建了`Validator`实例，就可以使用`failed`(或`pass`)方法来执行验证。

    if ($validator->fails()) {
        // 给定的数据未通过验证
    }

给定的数据未通过验证

    $messages = $validator->messages();

您还可以访问失败的验证规则数组，而不显示消息。 为此，请使用`failed`方法：

    $failed = $validator->failed();

#### 验证文件

`Validator`类提供了几个验证文件的规则，例如`size`，`mimes`等。 验证文件时，您可以将它们与其他数据一起传递给验证器。

<a name="working-with-error-messages"></a>
## 使用错误消息

在`Validator`实例上调用`messages`方法后，您将收到一个`Illuminate\Support\MessageBag`实例，它有各种方便的方法来处理错误消息。

#### 检索字段的第一条错误消息

    echo $messages->first('email');

#### 检索字段的所有错误消息

    foreach ($messages->get('email') as $message) {
        //
    }

#### 检索所有字段的所有错误消息

    foreach ($messages->all() as $message) {
        //
    }

#### 确定字段是否存在消息

    if ($messages->has('email')) {
        //
    }

#### 检索具有格式的错误消息

    echo $messages->first('email', '<p>:message</p>');

> **注意:** 默认情况下，使用Bootstrap兼容语法格式化消息。

#### 使用格式检索所有错误消息

    foreach ($messages->all('<li>:message</li>') as $message) {
        //
    }

<a name="error-messages-and-views"></a>
## 错误消息和视图

完成验证后，您将需要一种简单的方法将错误消息返回到您的视图。 这可以在October方便地处理。 以下面的路线为例：

    public function onRegister()
    {
        $rules = [];

        $validator = Validator::make(Input::all(), $rules);

        if ($validator->fails()) {
            return Redirect::to('register')->withErrors($validator);
        }
    }

请注意，当验证失败时，我们使用`withErrors`方法将`Validator`实例传递给Redirect。 此方法将错误消息刷新到会话，以便它们在下一个请求时可用。

October将始终检查会话数据中的错误，并自动将它们绑定到视图(如果可用)。 **因此，重要的是要注意，在每个请求**中，所有页面中都会始终提供`errors`变量，这样您就可以方便地假设`errors`变量始终定义并且可以安全使用。 `errors`变量将是`MessageBag`的一个实例。

因此，在重定向之后，您可以在视图中使用自动绑定的`errors`变量：

    {{ errors.first('email') }}

### 命名错误包

如果您在一个页面上有多个表单，您可能希望将`MessageBag`命名为错误。 这将允许您检索特定表单的错误消息。 只需将名称作为第二个参数传递给`withErrors`：

    return Redirect::to('register')->withErrors($validator, 'login');

然后，您可以从`$errors`变量访问命名的`MessageBag`实例：

    {{ errors.login.first('email') }}

<a name="available-validation-rules"></a>
## 可用的验证规则

以下是所有可用验证规则及其功能的列表：

-  [已接受](#rule-accepted)
-  [有效网址](#rule-active-url)
-  [之后(日期)](#rule-after)
-  [字母](#rule-alpha)
-  [字母字符](#rule-alpha-dash)
-  [字母数字](#rule-alpha-num)
-  [数组](#rule-array)
-  [之前(日期)](#rule-before)
-  [之间](#rule-between)
-  [Boolean](#rule-boolean)
-  [确认](#rule-confirmed)
-  [日期](#rule-date)
-  [日期格式](#rule-date-format)
-  [不同](#rule-different)
-  [数字](#rule-digits)
-  [数字之间](#rule-digits-between)
-  [电子邮件](#rule-email)
-  [存在(数据库)](#rule-exist)
-  [图像(文件)](#rule-image)
-  [在之中](#rule-in)
-  [整数](#rule-integer)
-  [IP地址](#rule-ip)
-  [最大](#rule-max)
-  [MIME类型](#rule-mimes)
-  [分钟](#rule-min)
-  [不在](#rule-not-in)
-  [可为空](#rule-nullable)
-  [数字](#rule-numeric)
-  [正则表达式](#rule-regex)
-  [必填](#rule-required)
-  [必填如果](#rule-required-if)
-  [必需](#rule-required-with)
-  [全部必需](#rule-required-with-all)
-  [不需要](#rule-required-without)
-  [必不可少](#rule-required-without-all)
-  [相同](#rule-same)
-  [尺寸](#rule-size)
-  [字符](#rule-string)
-  [时区](#rule-timezone)
-  [唯一(数据库)](#rule-unique)
-  [URL](#rule-url)


<a name="rule-accepted"></a>
#### accepted 可接受的

验证字段必须是_yes_，_on_或_1_。 这有助于验证“服务条款”的接受程度。

<a name="rule-active-url"></a>
#### active_url 有效网址

根据`checkdnsrr` PHP函数，验证字段必须是有效的URL。

<a name="rule-after"></a>
#### after:_date_ 之后(日期)

验证字段必须是给定日期之后的值。 日期将传递到PHP`strtotime`函数。

<a name="rule-alpha"></a>
#### alpha 字母

验证字段必须完全是字母字符。

<a name="rule-alpha-dash"></a>
#### alpha_dash 字母字符

验证字段可能包含字母数字字符，以及破折号和下划线。

<a name="rule-alpha-num"></a>
#### alpha_num 字母数字

验证字段必须是完全字母数字字符。

<a name="rule-array"></a>
#### array 数组

验证字段必须是数组类型。

<a name="rule-before"></a>
#### before:_date_ 之前(日期)

验证字段必须是给定日期之前的值。 日期将传递到PHP`strtotime`函数。

<a name="rule-between"></a>
#### between:_min_,_max_ 之间

验证字段的大小必须在给定的_min_和_max_之间。 字符串，数字和文件的评估方式与`size`规则相同。

<a name="rule-boolean"></a>
#### boolean

验证字段必须能够转换为布尔值。 接受的输入是`true`，`false`，`1`，`0`，`"1"`和`"0"`。

<a name="rule-confirmed"></a>
#### confirmed 确认

验证字段必须具有匹配字段`foo_confirmation`。 例如，如果验证字段是`password`，则输入中必须存在匹配的`password_confirmation`字段。

<a name="rule-date"></a>
#### date 日期

根据`strtotime` PHP函数，验证字段必须是有效日期。

<a name="rule-date-format"></a>
#### date_format:_format_ 日期格式

验证字段必须与根据`date_parse_from_format` PHP函数定义的_format_匹配。

<a name="rule-different"></a>
#### different:_field_ 不同

给定的_field_必须与验证字段不同。给定的_field_必须与验证字段不同。

<a name="rule-digits"></a>
#### digits:_value_ 数字

验证字段必须为_numeric_，且必须具有_value_的确切长度。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_ 数字之间

验证字段的长度必须在给定的_min_和_max_之间。

<a name="rule-email"></a>
#### email 电子邮件

验证字段必须格式化为电子邮件地址。

<a name="rule-exists"></a>
#### exists:_table_,_column_ 存在(数据库)

验证字段必须存在于给定的数据库表中。

#### 存在规则的基本用法

    'state' => 'exists:states'

#### 指定自定义列名称

    'state' => 'exists:states,abbreviation'

您还可以指定将添加为查询的“where”子句的更多条件：

    'email' => 'exists:staff,email,account_id,1'

将`NULL`作为“where”子句值传递将添加对`NULL`数据库值的检查：

    'email' => 'exists:staff,email,deleted_at,NULL'

<a name="rule-image"></a>
#### image 图片

验证文件必须是图像(jpeg，png，bmp或gif)

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

验证字段必须包含在给定的值列表中。

<a name="rule-integer"></a>
#### integer

验证字段必须具有整数值。

<a name="rule-ip"></a>
#### ip

验证字段必须格式化为IP地址。

<a name="rule-max"></a>
#### max:_value_

验证字段必须小于或等于最大_value_。 字符串，数字和文件的评估方式与[`size`](#rule-size)规则相同。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

验证中的文件必须具有与列出的扩展名之一相对应的MIME类型。

#### MIME规则的基本用法

    'photo' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min:_value_

验证字段必须具有最小_value_。 字符串，数字和文件的评估方式与[`size`](＃rule-size)规则相同。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

验证字段不得包含在给定的值列表中。

<a name="rule-nullable"></a>
#### nullable

验证字段可能是`null`。 这在验证诸如字符串和可以包含`null`值的整数之类的原语时特别有用。

<a name="rule-numeric"></a>
#### numeric

验证字段必须具有数值。

<a name="rule-regex"></a>
#### regex:_pattern_

验证字段必须与给定的正则表达式匹配。

**注意:** 使用`regex`模式时，可能需要在数组中指定规则而不是使用管道分隔符，尤其是在正则表达式包含管道符时。

<a name="rule-required"></a>
#### required

验证字段必须存在于输入数据中。

<a name="rule-required-if"></a>
#### required_if:_field_,_value_,...

如果_field_字段等于任何_value_，则必须存在验证字段。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

如果存在任何其他指定字段，则验证字段必须仅存在。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

如果存在所有其他指定字段，则验证字段必须仅存在。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

只有当任何其他指定字段不存在时，验证字段必须存在。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

只有当所有其他指定字段都不存在时，验证字段才必须存在。

<a name="rule-same"></a>
#### same:_field_

指定的_field_值必须与验证时字段的值匹配。

<a name="rule-size"></a>
#### size:_value_

验证字段的大小必须与给定的_value_匹配。 对于字符串数据，_value_对应于字符数。 对于数字数据，_value_对应于给定的整数值。 对于文件，_size_对应于以千字节为单位的文件大小。

<a name="rule-string"></a>
#### string:_value_

验证字段必须是字符串类型。

<a name="rule-timezone"></a>
#### timezone

根据`timezone_identifiers_list` PHP函数，验证字段必须是有效的时区标识符。

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

验证字段在给定数据库表上必须是唯一的。 如果未指定`column`选项，则将使用字段名称。

#### 唯一规则的基本用法

    'email' => 'unique:users'

#### 指定自定义列名称

    'email' => 'unique:users,email_address'

#### 强制使用唯一规则忽略给定的ID

    'email' => 'unique:users,email_address,10'

#### 添加其他where子句

您还可以指定将添加为查询的“where”子句的更多条件：

    'email' => 'unique:users,email_address,NULL,id,account_id,1'

在上面的规则中，只有`account_id`为“1”的行才会包含在唯一检查中。

<a name="rule-url"></a>
#### url

验证字段必须格式化为URL。

> **注意:** 这个函数使用PHP的`filter_var`方法。

<a name="conditionally-adding-rules"></a>
## 有条件地添加规则

在某些情况下，如果输入数组中存在该字段，您可能希望仅对**字段**运行验证检查。 要快速完成此操作，请将“有时”规则添加到规则列表中：

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

在上面的例子中，`email`字段只有在`$data`数组中存在时才会被验证。

#### 复杂的条件验证

有时，您可能希望仅在另一个字段的值大于100时才需要给定字段。或者，只有存在另一个字段时，您可能需要两个字段才能获得给定值。 添加这些验证规则并不一定非常痛苦。 首先，使用永不改变的_static rules_创建一个`Validator`实例：

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

我们假设我们的网络应用程序适用于游戏收藏家。 如果游戏收藏家在我们的应用程序中注册并拥有超过100个游戏，我们希望他们解释为什么他们拥有这么多游戏。 例如，也许他们经营游戏转售店，或者他们只是喜欢收藏。 要有条件地添加此要求，我们可以在`Validator`实例上使用`有时`方法。

    $v->sometimes('reason', 'required|max:500', function($input) {
        return $input->games >= 100;
    });

传递给`sometimes`方法的第一个参数是我们有条件地验证的字段的名称。 第二个参数是我们想要添加的规则。 如果`Closure`在第三个参数返回时返回`true`，则会添加规则。 这种方法使得构建复杂的条件验证变得轻而易举。 您甚至可以一次为多个字段添加条件验证：

    $v->sometimes(['reason', 'cost'], 'required', function($input) {
        return $input->games >= 100;
    });

> **注意:** 传递给`Closure`的`$input`参数将是`Illuminate\Support\Fluent`的一个实例，可以用作访问输入和文件的对象。

<a name="validating-arrays"></a>
## 验证数组

验证基于数组的表单输入字段不一定非常痛苦。 您可以使用“点表示法”来验证数组中的属性。 例如，如果传入的HTTP请求包含`photos[profile]`字段，您可以像这样验证它：

    $validator = Validator::make(Input::all(), [
        'photos.profile' => 'required|image',
    ]);

您还可以验证数组的每个元素。 例如，要验证给定数组输入字段中的每个电子邮件是否唯一，您可以执行以下操作：

    $validator = Validator::make(Input::all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

同样，在语言文件中指定验证消息时，可以使用`*`字符，这样就可以轻松地为基于数组的字段使用单个验证消息：

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],


<a name="custom-error-messages"></a>
## 自定义错误消息

如果需要，您可以使用自定义错误消息进行验证，而不是默认值。 有几种方法可以指定自定义消息。

#### 将自定义消息传递给验证器

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

> *注意:* `:attribute`占位符将被验证字段的实际名称替换。 您还可以在验证消息中使用其他占位符。

#### 其他验证占位符

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute must be between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### 为给定属性指定自定义消息

有时，您可能希望仅为特定字段指定自定义错误消息：

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### 在语言文件中指定自定义消息

在某些情况下，您可能希望在语言文件中指定自定义消息，而不是直接将它们传递给“Validator”。 为此，请将您的消息添加到插件的`lang/xx/validation.php`语言文件中的数组中。

    return  [
        'required' => 'We need to know your e-mail address!',
        'email.required' => 'We need to know your e-mail address!',
    ];

然后在你对`Validator::make`的调用中使用`Lang:get`来使用你的自定义文件。

    Validator::make($formValues, $validations, Lang::get('acme.blog::validation'));

<a name="custom-validation-rules"></a>
## 自定义验证规则

#### 注册自定义验证规则

有各种有用的验证规则; 但是，您可能希望指定一些自己的。 注册自定义验证规则的一种方法是使用`Validator::extend`方法：

    Validator::extend('foo', function($attribute, $value, $parameters) {
        return $value == 'foo';
    });

自定义验证器Closure接收三个参数：要验证的`$attribute`的名称，属性的`$value`，以及传递给规则的`$parameters`数组。

您也可以将类和方法传递给`extend`方法而不是Closure：

    Validator::extend('foo', 'FooValidator@validate');

请注意，您还需要为自定义规则定义错误消息。 您可以使用内联自定义消息数组或在验证语言文件中添加条目来执行此操作。

#### 扩展验证器类

您可以扩展Validator类本身，而不是使用Closure回调来扩展Validator。 为此，编写一个扩展`Illuminate\Validation\Validator`的Validator类。 您可以通过在其前面添加`validate`来为类添加验证方法：

    <?php

    class CustomValidator extends Illuminate\Validation\Validator
    {

        public function validateFoo($attribute, $value, $parameters)
        {
            return $value == 'foo';
        }

    }

#### 注册自定义验证器解析程序

接下来，您需要注册自定义Validator扩展：

    Validator::resolver(function($translator, $data, $rules, $messages, $customAttributes) {
        return new CustomValidator($translator, $data, $rules, $messages, $customAttributes);
    });

创建自定义验证规则时，有时可能需要为错误消息定义自定义占位符替换。 您可以通过如上所述创建自定义Validator，并向验证器添加`replaceXXX`函数来实现。

    protected function replaceFoo($message, $attribute, $rule, $parameters)
    {
        return str_replace(':foo', $parameters[0], $message);
    }

如果您想在不扩展`Validator`类的情况下添加自定义消息“replacer”，您可以使用`Validator::replacer`方法：

    Validator::replacer('rule', function($message, $attribute, $rule, $parameters) {
        //
    });
