# 应用

- [应用容器](#app-container)
- [服务提供者](#service-providers)
- [应用事件](#application-events)
- [应用帮助](#application-helpers)

<a name="app-container"></a>
## 应用容器

控制反转(IoC)容器是用于管理类依赖性的工具。 依赖注入是一种删除硬编码类依赖关系的方法。 相反，依赖项是在运行时注入的，因为可以轻松地交换依赖项实现，从而实现更大的灵活性。

#### 将类型绑定到容器中

IoC容器有两种方法可以解决依赖关系：通过Closure回调或自动解决。 首先，我们将探讨Closure回调。 首先，可以将“类型”绑定到容器中：

    App::bind('foo', function($app) {
        return new FooBar;
    });

#### 解决容器中的类型

    $value = App::make('foo');

当调用`App::make`方法时，执行Closure回调并返回结果。

#### 将“共享”类型绑定到容器中

有时您可能希望将某些内容绑定到仅应解析一次的容器中，并且在后续调用容器时应返回相同的实例：

    App::singleton('foo', function() {
        return new FooBar;
    });

#### 将现有实例绑定到容器中

您还可以使用`instance`方法将现有对象实例绑定到容器中：

    $foo = new Foo;

    App::instance('foo', $foo);

#### 将接口绑定到实现类

在某些情况下，类可能依赖于接口实现，而不是“具体类型”。 在这种情况下，必须使用`App::bind`方法来通知容器注入哪个接口实现：

    App::bind('UserRepositoryInterface', 'DbUserRepository');

现在考虑以下代码：

    $users = App::make('UserRepositoryInterface');

由于我们已将`UserRepositoryInterface`绑定到具体类型，因此`DbUserRepository`将在创建时自动注入此控制器。

<a name="where-to-register"></a>
### 在哪里注册绑定

IoC绑定，如[事件处理程序](events)，通常属于“引导代码(bootstrap code)”类别。 换句话说，它们准备应用程序以实际处理请求，并且通常需要在实际调用路由或控制器之前执行。 最常见的地方是[插件注册文件](../plugin/registration#registration-methods).的`boot`方法。 或者，插件可以在插件目录中提供名为**init.php**的文件，您可以使用该文件放置IoC注册逻辑。

<a name="service-providers"></a>
## 服务提供者

服务提供者是在单个位置创建库和执行与组相关的IoC注册的好方法。 在服务提供程序中，您可以注册自定义身份验证驱动程序，使用IoC容器注册应用程序的存储库类，甚至可以设置自定义Artisan命令。

实际上，[插件注册文件](../plugin/registration)继承了服务提供者，而大多数核心组件都包含服务提供者。 您的应用程序的所有注册服务提供程序都列在`config/app.php`配置文件的`providers`数组中。

#### 定义服务提供者

要创建服务提供者，只需扩展`October\Rain\Support\ServiceProvider`类并定义`register`方法：

    use October\Rain\Support\ServiceProvider;

    class FooServiceProvider extends ServiceProvider
    {

        public function register()
        {
            $this->app->bind('foo', function() {
                return new Foo;
            });
        }

    }

请注意，在`register`方法中，可以通过`$this-> app`属性使用应用程序IoC容器。 一旦您创建了一个提供程序并准备将其注册到您的应用程序，只需将其添加到`app`配置文件中的`providers`数组即可。

#### 在运行时注册服务提供商

您也可以使用`App::register`方法在运行时注册服务提供者：

    App::register('FooServiceProvider');

<a name="application-events"></a>
## 应用事件

#### 请求事件

您可以在使用`before`和`after`方法路由请求之前注册特殊事件：

    App::before(function ($request) {
        // 在路由请求之前执行的代码
    });

    App::after(function ($request) {
        // 请求路由后执行的代码
    });

#### 容器事件

每次解析对象时，服务容器都会触发一个事件。 你可以使用`resolving`方法听这个事件：

    App::resolving(function ($object, $app) {
        // 当容器解析任何类型的对象时调用...
    });

    App::resolving('foo', function ($fooBar, $app) {
        // 当容器使用提示“foo”解析对象时调用...
    });

    App::resolving('Acme\Blog\Classes\FooBar', function ($fooBar, $app) {
        // 当容器解析“FooBar”类型的对象时调用...
    });

如您所见，正在解析的对象将被传递给回调，允许您在将对象提供给其使用者之前设置该对象的任何其他属性。

<a name="application-helpers"></a>
## 应用帮助

#### 查找应用程序环境

您可以使用`environment`方法来寻找由[environment configuration](../setup/configuration#environment-config)确定的应用程序环境。

    // production
    App::environment();

#### 确定执行上下文

可以使用`runningInBackend`方法了解当前请求是否在管理后端区域中执行。

    App::runningInBackend();

您还可以使用`runningInConsole`方法检查执行代码是否在[命令行界面](../console/commands)中进行：

    App::runningInConsole();
