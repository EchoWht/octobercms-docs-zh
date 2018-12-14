# 后台用户和权限

- [用户和权限](#users-and-permissions)
- [后端用户帮助器](#backend-auth-facade)
- [注册权限](#permission-registration)
- [限制对后端页面的访问](#page-access)
- [限制对功能的访问](#features)

后端的用户管理包括角色，组，权限，密码重置和登录限制等功能。 插件还可以注册控制对后端功能的访问权限。

<a name="users-and-permissions"></a>
## 用户和权限

对权限系统控制对OctoberCMS实例的所有部分的访问。在最低级别，有超级用户(`is_superuser`标志设置为true的用户)，管理员(用户)和权限。 `\Backend\Models\User`模型是容纳有关用户的所有重要信息的容器。

超级用户可以访问系统中的所有内容，只能由他们自己或其他超级用户管理;即使管理员具有`backend.manage_users`权限，常规管理员也看不到它们，也不能编辑它们。

权限是`author.plugin.permission_name`形式的字符串键，通过在Edit Administrator页面上直接赋值或通过用户的Role继承来授予用户。

检查用户是否具有特定权限时，将继承该用户角色的权限设置，然后由直接应用于该用户的任何权限进行覆盖。例如，如果用户**Bob**有角色**Genius**，角色**Genius**有`eat_cake`权限，但 **Bob** 有`eat_cake`权限专门设置为拒绝然后**Bob**不会去`eat_cake`。但是，如果**Bob**拥有直接分配给他的权限`eat_vegetables`，但**Genius**角色没有，那么**Bob**仍然会进入`eat_vegetables`。

角色(`\Backend\Models\UserRole`)是权限分组，其名称和描述用于标识角色。管理员一次只能为其分配一个角色。可以将角色分配给多个管理员。 October默认发布了两个系统角色，`developer`和`publisher`。可以创建具有自己的权限组合的任意数量的自定义角色并将其应用于用户。

> **注意：** 具有`manage_users`权限的任何用户都可以管理角色的分配，但只能管理其他用户(而不是自己)，并且只能由超级用户创建或修改角色。

> **注意：** 系统角色(`developer`，`publisher`，以及`is_system`设置为`true`的任何角色)不能通过后端更改其权限，它们在代码中定义，并附加到任何权限 它们也直接在代码中定义。

组(`\Backend\Models\UserGroup`)是用于对管理员进行分组的组织工具，可以将它们视为“用户类别”。 它们与权限无关，严格用于组织目的。 例如，如果您想向“总部员工”组中的所有用户发送电子邮件，您只需执行`Mail::sendTo(UserGroup::where('code'，'head-office-staff'))->get()->users，'author.plugin::mail.important_notification'，$data);`

<a name="backend-auth-facade"></a>
## 后端用户帮助器

全局`BackendAuth`facade可用于管理管理用户，主要继承`October\Rain\Auth\Manager`类。 要注册新的管理员用户帐户，请使用`BackendAuth::register`方法。

    $user = BackendAuth::register([
        'name' => 'Some User',
        'login' => 'someuser',
        'email' => 'some@website.tld',
        'password' => 'changeme',
        'password_confirmation' => 'changeme'
    ]);

`BackendAuth::check`方法是检查用户是否已登录的快速方法。要返回已登录的用户模型，请改用“BackendAuth::getUser”。 此外，活动用户将在任何[后端控制器](backend-controllers-ajax.md)中以`$this->user`的形式提供。

    // 如果已登录，则返回true。
    $loggedIn = BackendAuth::check();

    // 返回已登录的用户
    $user = BackendAuth::getUser();

    // 从控制器返回已登录的用户
    $user = $this->user;

您可以使用`BackendAuth::findUserByLogin`方法通过登录名查找用户。

    $user = BackendAuth::findUserByLogin('someuser');

您可以通过提供`BackendAuth::authenticate`的登录名和密码来验证用户身份。 您还可以通过将`Backend\Models\User`模型与`BackendAuth::login`一起传递给用户进行身份验证。

    // Authenticate user by credentials
    $user = BackendAuth::authenticate([
        'login' => post('login'),
        'password' => post('password')
    ]);

    // Sign in as a specific user
    BackendAuth::login($user);

<a name="permission-registration"></a>
## 注册权限

插件可以通过覆盖[Plugin注册类](plugin-registration.md#registration-file)中的`registerPermissions`方法来注册后端用户权限。 权限被定义为一个数组，其中的键对应于权限键，而值对应于权限描述。 权限密钥由作者姓名，插件名称和功能名称组成。 这是一个示例代码：

    acme.blog.access_categories

下一个示例显示如何注册后端权限项。 权限是使用权限密钥和说明定义的。 在后端权限管理用户界面权限显示为复选框列表。 后端控制器可以使用插件定义的权限来限制用户访问[页面](#page-access)或[features](#features)。

    public function registerPermissions()
    {
        return [
            'acme.blog.access_posts' => [
                'label' => 'Manage the blog posts',
                'tab' => 'Blog',
                'order' => 200,
            ],
            // ...
        ];
    }

您还可以将`roles`选项指定为一个数组，每个值都作为角色API代码。 使用此代码创建角色时，它将成为系统角色，始终将此权限授予具有该角色的用户。

    public function registerPermissions()
    {
        return [
            'acme.blog.access_categories' => [
                'label' => 'Manage the blog categories',
                'tab' => 'Blog',
                'order' => 200,
                'roles' => ['developer']
            ]
            // ...
        ];
    }

<a name="page-access"></a>
## 限制对后端页面的访问

在后端控制器类中，您可以指定访问控制器提供的页面所需的权限。 它是使用`$requiredPermissions`控制器的属性完成的。 此属性应包含权限键数组。 如果用户权限与列表中的任何权限匹配，则框架将允许用户查看控制器页面。

    <?php namespace Acme\Blog\Controllers;

    use Backend\Classes\BackendController;

    class Posts extends BackendController
    {
        public $requiredPermissions = ['acme.blog.access_posts'];
    }

您还可以使用**星号**符号来指示“所有权限”条件。 在下一个示例中，对于具有以“acme.blog”开头的任何权限的所有用户，都可以访问控制器页面。：

    public $requiredPermissions = ['acme.blog.*'];

<a name="features"></a>
## 限制对功能的访问

后端用户模型具有允许确定用户是否具有特定权限的方法。 您可以使用此功能来限制后端用户界面的功能。 后端用户支持的权限方法是`hasAccess`和`hasPermission`。 这两种方法都有两个参数：权限键字符串(或键字符串数组)和一个可选参数，指示需要使用第一个参数列出的所有权限。

如果用户是超级用户(`is_superuser`设置为`true`)，`hasAccess`方法返回** true **表示任何权限。 `hasPermission`方法更严格，只有当用户在其帐户中或通过其角色实际具有指定的权限时才返回true。 通常，`hasAccess`是首选方法，因为它尊重超级用户的绝对能力。 以下示例显示如何使用控制器代码中的方法：

    if ($this->user->hasAccess('acme.blog.*')) {
        // ...
    }

    if ($this->user->hasPermission([
        'acme.blog.access_posts',
        'acme.blog.access_categories'
    ])) {
        // ...
    }

您还可以使用后端视图中的方法隐藏用户界面元素。 下面的示例演示了如何隐藏编辑类别[后端表单](backend-forms.md)上的按钮：

    <?php if ($this->user->hasAccess('acme.blog.delete_categories')): ?>
        <button
            type="button"
            class="oc-icon-trash-o btn-icon danger pull-right"
            data-request="onDelete"
            data-load-indicator="Deleting Category..."
            data-request-confirm="Do you really want to delete this category?">
        </button>
    <?php endif ?>
