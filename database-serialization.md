# 模型序列化

- [介绍](#introduction)
- [基础用法](#basic-usage)
- [隐藏JSON中的属性](#hiding-attributes-from-json)
- [将值附加到JSON](#appending-values-to-json)

<a name="introduction"></a>
## 介绍

构建JSON API时，通常需要将模型和关系转换为数组或JSON。 模型包括进行这些转换的便捷方法，以及控制序列化中包含哪些属性。

<a name="basic-usage"></a>
## 基础用法

#### 将模型转换为数组

要将模型及其加载的[relationships](relations)转换为数组，可以使用`toArray`方法。 此方法是递归的，因此所有属性和所有关系（包括关系关系）都将转换为数组：

    $user = User::with('roles')->first();

    return $user->toArray();

您还可以将[collections](collections) 转换为数组：

    $users = User::all();

    return $users->toArray();

#### 将模型转换为JSON

要将模型转换为JSON，可以使用`toJson`方法。 与`toArray`一样，`toJson`方法是递归的，因此所有属性和关系都将转换为JSON：

    $user = User::find(1);

    return $user->toJson();

或者，您可以将模型或集合强制转换为字符串，该字符串将自动调用`toJson`方法：

    $user = User::find(1);

    return (string) $user;

由于模型和集合在转换为字符串时会转换为JSON，因此可以直接从应用程序的路由，AJAX处理程序或控制器返回Model对象：

    Route::get('users', function () {
        return User::all();
    });

<a name="hiding-attributes-from-json"></a>
## 隐藏JSON中的属性

有时，您可能希望限制模型数组或JSON表示中包含的属性（如密码）。 为此，请在模型中添加`$hidden`属性定义：

    <?php namespace Acme\Blog\Models;

    use Model;

    class User extends Model
    {
        /**
         * 应该为数组隐藏的属性。
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

或者，您可以使用`$visible`属性来定义应包含在模型的数组和JSON表示中的属性的白名单：

    class User extends Model
    {
        /**
         * 应该在数组中可见的属性。
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

<a name="appending-values-to-json"></a>
## 将值附加到JSON

有时，您可能需要添加数据库中没有相应列的数组属性。 为此，首先为值定义[转换器](../database/mutators)：

    class User extends Model
    {
        /**
         * 获取用户的管理员标志。
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

创建访问器后，将属性名称添加到模型的`appends`属性中：

    class User extends Model
    {
        /**
         * 要附加到模型的数组形式的访问器。
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

将属性添加到`appends`列表后，它将包含在模型的数组和JSON表单中。 `appends`数组中的属性也将遵循模型上配置的`visible`和`hidden`设置。
