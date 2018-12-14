# 模型转换器

- [介绍](#introduction)
- [访问器和转换器](#accessors-and-mutators)
- [日期变异者](#date-mutators)
- [属性转换](#attribute-casting)

<a name="introduction"></a>
## 介绍

访问器和转换器允许您在从模型中检索属性或设置其值时格式化属性。 例如，您可能希望使用[加密服务](../services/encryption) 在存储在数据库中时对值进行加密，然后在模型上访问该属性时自动解密该属性。

除了自定义访问器和转换器之外，您还可以自动将日期字段转换为[Carbon](https://github.com/briannesbitt/Carbon) 实例，甚至[将文本值转换为JSON](#attribute-casting)。

<a name="accessors-and-mutators"></a>
## 访问器和转换器

#### 定义一个访问器

要定义访问器，请在模型上创建一个`getFooAttribute`方法，其中`Foo`是您要访问的列的“驼峰”名称。 在这个例子中，我们将为`first_name`属性定义一个访问器。 尝试检索`first_name`的值时，将自动调用访问器：

    <?php namespace Acme\Blog\Models;

    use Model;

    class User extends Model
    {
        /**
         * 获取用户的名
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

如您所见，列的原始值将传递给访问器，允许您操作并返回值。 要访问访问器处理过的值，您可以只访问`first_name`属性：

    $user = User::find(1);

    $firstName = $user->first_name;

#### 定义一个转换器

要定义一个转换器，请在模型上定义一个`setFooAttribute`方法，其中`Foo`是您要访问的列的“驼峰”名称。 在这个例子中，让我们为`first_name`属性定义一个转换器。 当我们尝试在模型上设置`first_name`属性的值时，将自动调用此转换器：

    <?php namespace Acme\Blog\Models;

    use Model;

    class User extends Model
    {
        /**
         * 设置用户的名
         *
         * @param  string  $value
         * @return string
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

转换器将接收在属性上设置的值，允许您操作该值并在模型的内部`$attributes`属性上设置操纵值。 例如，如果我们尝试将`first_name`属性设置为`Sally`：

    $user = User::find(1);

    $user->first_name = 'Sally';

这里将使用值`Sally`调用`setFirstNameAttribute`函数。 然后，转换器将`strtolower`函数应用于name，并在内部`$attributes`数组中设置其值。

<a name="date-mutators"></a>
## 日期转换器

默认情况下，October的模型会将`created_at`和`updated_at`列转换为[Carbon](https://github.com/briannesbitt/Carbon)对象的实例，该对象提供各种有用的方法并扩展原生PHP`DateTime`类。

您可以通过覆盖模型的`$dates`属性来自定义哪些字段会自动转换，甚至完全禁用此转换：

    class User extends Model
    {
        /**
         * 应该转换为日期的属性。
         *
         * @var array
         */
        protected $dates = ['created_at', 'updated_at', 'disabled_at'];
    }

当列被视为日期时，您可以将其值设置为UNIX时间戳，日期字符串(`Ymd`)，日期时间字符串，当然还有`DateTime`/`Carbon`实例，日期值将自动 正确存储在您的数据库中：

    $user = User::find(1);

    $user->disabled_at = Carbon::now();

    $user->save();

如上所述，当检索`$dates`属性中列出的属性时，它们将自动转换为[Carbon](https://github.com/briannesbitt/Carbon) 实例，允许您使用任何Carbon的方法 在你的属性上：

    $user = User::find(1);

    return $user->disabled_at->getTimestamp();

默认情况下，时间戳格式为“Y-m-d H:i:s”。 如果需要自定义时间戳格式，请在模型上设置`$dateFormat`属性。 此属性确定数据库中日期属性的存储方式，以及将模型序列化为数组或JSON时的格式：

    class Flight extends Model
    {
        /**
         * 模型日期列的存储格式。
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>
## 属性转换

模型上的`$casts`属性提供了将属性转换为常见数据类型的便捷方法。 `$casts`属性应该是一个数组，其中键是要转换的属性的名称，而值是您希望转换为列的类型。 支持的强制类型是：`integer`，`real`，`float`，`double`，`string`，`boolean`，`object`和`array`。

例如，让我们将`is_admin`属性转换为一个布尔值，该属性作为整数(`0`或`1`)存储在我们的数据库中：

    class User extends Model
    {
        /**
         * 需要转换为原生类型的属性。
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

现在，当您访问它时，`is_admin`属性将始终强制转换为布尔值，即使基础值作为整数存储在数据库中：

    $user = User::find(1);

    if ($user->is_admin) {
        //
    }

#### 数组转换

在处理存储为序列化JSON的列时，`array`强制转换类型特别有用。 例如，如果您的数据库具有包含序列化JSON的`TEXT`字段类型，则在您的Eloquent模型上访问该属性时，将“array”强制转换为该属性会自动将该属性反序列化为PHP数组：

    class User extends Model
    {
        /**
         * 需要转换为原生类型的属性。
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

定义了强制转换后，您可以访问`options`属性，它将自动从JSON反序列化为PHP数组。 设置`options`属性的值时，给定的数组将自动序列化为JSON以进行存储：

    $user = User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
