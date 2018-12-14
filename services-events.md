# 事件

- [基本用法](#basic-usage)
- [订阅用法](#events-subscribing)
    - [在哪里注册监听器](#event-registration)
    - [使用优先级订阅](#subscribing-priority)
    - [暂停活动](#subscribing-halting)
    - [通配符监听器](#wildcard-listeners)
- [触发事件](#events-firing)
    - [通过引用传递参数](#event-pass-by-reference)
    - [排队的事件](#queued-events)
- [使用类作为监听器](#using-classes-as-listeners)
    - [订阅个别方法](#event-class-method)
    - [订阅整个Class](#event-class-subscribe)
- [事件触发器特性](#event-emitter-trait)

<a name="basic-usage"></a>
## 基本用法

`Event`类提供了一个简单的观察器实现，允许您订阅和监听应用程序中的事件。 例如，您可以在用户登录并更新其上次登录日期时收听。

    Event::listen('auth.login', function($user) {
        $user->last_login = new DateTime;
        $user->save();
    });

这是使用`Event::fire`方法提供的事件，该方法被称为用户登录逻辑的一部分，从而使逻辑可扩展。

    Event::fire('auth.login', [$user]);

<a name="events-subscribing"></a>
## 订阅活动

`Event::listen`方法主要用于订阅事件，可以在应用程序代码中的任何位置完成。 第一个参数是事件名称。

    Event::listen('acme.blog.myevent', ...);

第二个参数可以是一个闭包，指定触发事件时应该发生的事情。 闭包可以接受由[触发事件](#events-firing).提供的可选的一些参数。

    Event::listen('acme.blog.myevent', function($arg1, $arg2) {
        // Do something
    });

您还可以传递对任何可调用对象或[专用事件类](#using-classes-as-listeners)的引用，而这将被使用。

    Event::listen('auth.login', [$this, 'LoginHandler']);

> **注意**: 可调用方法可以选择指定全部，部分参数或不指定任何参数。 无论哪种方式，事件都不会抛出任何错误，除非它指定太多。

<a name="event-registration"></a>
### 在哪里注册监听器

最常见的地方是[插件注册文件](plugin-registration.md#registration-methods)的`boot`方法。

    class Plugin extends PluginBase
    {
        [...]

        public function boot()
        {
            Event::listen(...);
        }
    }

或者，插件可以在插件目录中提供名为**init.php**的文件，您可以使用该文件来放置事件注册逻辑。 例如：

    <?php

    Event::listen(...);

由于这些方法都不具备“正确”，因此请根据应用程序的大小选择您认为合适的方法。

<a name="subscribing-priority"></a>
### 使用优先级订阅

订阅事件时，您还可以将优先级指定为第三个参数。 优先级较高的监听器将首先运行，而具有相同优先级的监听器将按订阅顺序运行。

    // Run first
    Event::listen('auth.login', function() { ... }, 10);

    // Run second
    Event::listen('auth.login', function() { ... }, 5);

<a name="subscribing-halting"></a>
### 暂停活动

有时您可能希望停止将事件传播给其他监听器。 您可以通过从您的监听器返回“false”来执行此操作：

    Event::listen('auth.login', function($event) {
        // Handle the event

        return false;
    });

<a name="wildcard-listeners"></a>
### 通配符监听器

注册事件监听器时，可以使用星号指定通配符监听器。 以下监听器将处理以`foo`开头的所有事件。

    Event::listen('foo.*', function($param) {
        // Handle the event...
    });

您可以使用`Event::firing`方法确切地确定触发了哪个事件：

    Event::listen('foo.*', function($param) {
        if (Event::firing() == 'foo.bar') {
            // ...
        }
    });

<a name="events-firing"></a>
## 触发事件

您可以在代码中的任何位置使用`Event::fire`方法来使逻辑可扩展。 这意味着其他开发人员，甚至是您自己的内部代码，可以“挂钩”到这个代码点并注入特定的逻辑。 第一个参数应该是事件名称。

    Event::fire('myevent')

使用插件命名空间代码为事件名称添加前缀，这样可以防止与其他插件发生冲突。

    Event::fire('acme.blog.myevent');

第二个参数是一个值数组，它们将作为参数传递给订阅它的[事件监听器](#events-subscribing) 。

    Event::fire('acme.blog.myevent', [$arg1, $arg2]);

第三个参数指定事件是否应该是[暂停事件](#subscribing-halting)，这意味着如果返回“非null”值，它应该暂停。 默认情况下，此参数设置为false。

    Event::fire('acme.blog.myevent', [...], true);

如果事件暂停，则返回的第一个值将被捕获。

    // 单个结果，事件停止
    $result = Event::fire('acme.blog.myevent', [...], true);

否则，它以数组的形式返回所有事件的所有响应的集合。

    // 多个结果，所有事件都被触发
    $results = Event::fire('acme.blog.myevent', [...]);

<a name="event-pass-by-reference"></a>
## 通过引用传递参数

处理或过滤传递给事件的值时，可以在变量前加上“＆”以通过引用传递它。 这允许多个监听器操纵结果并将其传递给下一个结果。

    Event::fire('cms.processContent', [&$content]);

在监听事件时，还需要在闭包定义中使用`＆`符号声明参数。 在下面的示例中，`$content`变量将在结果中附加“AB”。

    Event::listen('cms.processContent', function (&$content) {
        $content = $content . 'A';
    });

    Event::listen('cms.processContent', function (&$content) {
        $content = $content . 'B';
    });

<a name="queued-events"></a>
### 排队的事件

可以在[与队列结合](services-queues.md)中推迟触发事件。 使用`Event::queue`方法将事件“排队”以进行触发但不立即触发它。

    Event::queue('foo', [$user]);

您可以使用`Event::flush`方法刷新所有排队的事件。

    Event::flush('foo');

<a name="using-classes-as-listeners"></a>
## 使用类作为监听器

在某些情况下，您可能希望使用类来处理事件而不是Closure。 类事件监听器将从[Application IoC容器](database-application.md)中解析出来，为您的监听器提供依赖注入的全部功能。

<a name="event-class-method"></a>
### 订阅个别方法

事件类可以像任何其他方法一样使用`Event::listen`方法注册，将类名作为字符串传递。

    Event::listen('auth.login', 'LoginHandler');

默认情况下，将调用`LoginHandler`类的`handle`方法：

    class LoginHandler
    {
        public function handle($data)
        {
            // ...
        }
    }

如果您不希望使用默认的`handle`方法，则可以指定应订阅的方法名称。

    Event::listen('auth.login', 'LoginHandler@onLogin');

<a name="event-class-subscribe"></a>
### 订阅整个类

事件订阅者是可以从类本身内订阅多个事件的类。 订阅者应定义一个`subscribe`方法，该方法将传递一个事件调度程序实例。

    class UserEventHandler
    {
        /**
         * 处理用户登录事件。
         */
        public function userLogin($event)
        {
            // ...
        }

        /**
         * 处理用户注销事件。
         */
        public function userLogout($event)
        {
            // ...
        }

        /**
         * 注册订阅者的监听器。
         *
         * @param  Illuminate\Events\Dispatcher  $events
         * @return array
         */
        public function subscribe($events)
        {
            $events->listen('auth.login', 'UserEventHandler@userLogin');

            $events->listen('auth.logout', 'UserEventHandler@userLogout');
        }
    }

订户定义后，可以使用`Event::subscribe`方法注册。

    Event::subscribe(new UserEventHandler);

您也可以使用[Application IoC容器](database-application.md)来解析您的订阅者。 为此，只需将订阅者的名称传递给`subscribe`方法即可。

    Event::subscribe('UserEventHandler');

<a name="event-emitter-trait"></a>
## 事件发射器特性

有时您希望将事件绑定到对象的单个实例。 您可以通过在类中实现`October\Rain\Support\Traits\Emitter`特征来使用替代事件系统。

    class UserManager
    {
        use \October\Rain\Support\Traits\Emitter;
    }

这个特性提供了一种用`bindEvent`监听事件的方法。

    $manager = new UserManager;
    $manager->bindEvent('user.beforeRegister', function($user) {
        // Check if the $user is a spammer
    });

`fireEvent`方法用于触发事件。

    $manager = new UserManager;
    $manager->fireEvent('user.beforeRegister', [$user]);

这些事件仅发生在本地对象上而不是全局上。
