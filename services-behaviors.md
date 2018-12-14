# 行为

- [介绍](#introduction)
- [与性状比较](#compare-traits)
- [扩展构造函数](#constructor-extension)
- [用法示例](#usage-example)

<a name="introduction"></a>
## 介绍

行为增加了类具有*私有特征(private traits)*的能力，也称为行为。 这些类似于[原生PHP特征trait](http://php.net/manual/en/language.oop5.traits.php) ，另外它们有一些明显的好处：

1. 行为有自己的构造函数。
1. 行为可以有私有或受保护的方法。
1. 方法和属性名称可以安全地冲突。
1. 可以动态地扩展类的类。

<a name="compare-traits"></a>
## 与性状比较

PHP trait的用法 如下所示

    class MyClass
    {
        use \October\Rain\UtilityFunctions;
        use \October\Rain\DeferredBinding;
    }

行为的用法如下所示

    class MyClass extends \October\Rain\Extension\Extendable
    {
        public $implement = [
            'October.Rain.UtilityFunctions',
            'October.Rain.DeferredBinding',
        ];
    }

trait的定义方法如下所示：

    trait UtilityFunctions
    {
        public function sayHello()
        {
            echo "Hello from " . get_class($this);
        }
    }

行为的定义方法如下所示

    class UtilityFunctions extends \October\Rain\Extension\ExtensionBase
    {
        protected $parent;

        public function __construct($parent)
        {
            $this->parent = $parent;
        }

        public function sayHello()
        {
            echo "Hello from " . get_class($this->parent);
        }
    }

扩展对象始终作为第一个参数传递给Behavior的构造函数。 

总结一下:
- 扩展 \October\Rain\Extension\ExtensionBase 以将您的类声明为行为
- 想要实现的类 - 行为需要扩展\October\Rain\Extension\ExtensionBase

> **注意**: 参见[使用traits而不是基础类](#using-traits) 

<a name="constructor-extension"></a>
## 扩展构造函数

任何使用`Extendable`或`ExtendableTrait`的类都可以使用静态`extend`方法扩展其构造函数。 参数应传递一个闭包，该闭包将作为类构造函数的一部分进行调用。

    MyNamespace\Controller::extend(function($controller) {
        //
    });
    
#### 动态声明属性

Properties can be declared on an extendable object by calling `addDynamicProperty` and passing a property name and value.

    Post::extend(function($model) {
        $model->addDynamicProperty('tagsCache', null);
    });
    
> **Note**: Attempting to set undeclared properties through normal means (`$this->foo = 'bar';`) on an object that implements the **October\Rain\Extension\ExtendableTrait** will not work. It won't throw an exception, but it will not autodeclare the property either. `addDynamicProperty` must be called in order to set previously undeclared properties on extendable objects.

#### 检索动态属性

可以使用从中继承的getDynamicProperties函数检索动态创建的属性
ExtendableTrait。

因此，检索所有动态属性将如下所示：

    $model->getDynamicProperties();

这将返回一个关联数组[key => value]，其中键是动态属性名称
而值是属性值。

如果我们知道我们想要什么属性，我们可以简单地将键(属性名称)附加到函数：

    $model->getDynamicProperties()[$key];

#### 动态创建方法

可以通过调用“addDynamicMethod”并传递方法名称和可调用对象(如“Closure”)来为可扩展对象创建方法。

    Post::extend(function($model) {
        $model->addDynamicProperty('tagsCache', null);
    
        $model->addDynamicMethod('getTagsAttribute', function() use ($model) {
            if ($this->tagsCache) {
                return $this->tagsCache;
            } else {
                return $this->tagsCache = $model->tags()->lists('name');
            }
        });
    });
    
#### 动态实现行为

这种扩展构造函数的独特功能允许动态实现行为，例如：

    /**
     * 扩展RainLab.Users控制器以包含RelationController行为
     */
    RainLab\Users\Controllers\Users::extend(function($controller) {

        // 动态实现列表控制器行为
        $controller->implement[] = 'Backend.Behaviors.RelationController';
        
        // 为RelationController行为动态声明relationConfig属性
        $controller->addDynamicProperty('relationConfig', '$/myvendor/myplugin/controllers/users/config_relation.yaml');
    });

<a name="usage-example"></a>
## 用法示例

#### 行为/扩展类

    <?php namespace MyNamespace\Behaviors;

    class FormController extends \October\Rain\Extension\ExtensionBase
    {
        /**
         * @var 引用扩展对象。
         */
        protected $controller;

        /**
         * Constructor
         */
        public function __construct($controller)
        {
            $this->controller = $controller;
        }

        public function someMethod()
        {
            return "I come from the FormController Behavior!";
        }

        public function otherMethod()
        {
            return "You might not see me...";
        }
    }

#### 继承类

这个`Controller`类将实现`FormController`行为，然后这些方法将可用(混入)到类中。 我们将覆盖`otherMethod`方法。

    <?php namespace MyNamespace;

    class Controller extends \October\Rain\Extension\Extendable
    {

        /**
         * Implement the FormController behavior
         */
        public $implement = [
            'MyNamespace.Behaviors.FormController'
        ];

        public function otherMethod()
        {
            return "I come from the main Controller!";
        }
    }

#### 使用扩展

    $controller = new MyNamespace\Controller;

    // 输出: I come from the FormController Behavior!
    echo $controller->someMethod();

    // 输出: I come from the main Controller!
    echo $controller->otherMethod();

    // 输出: You might not see me...
    echo $controller->asExtension('FormController')->otherMethod();

#### 检测已使用的扩展

要检查对象是否已使用行为进行扩展，可以在对象上使用`isClassExtendedWith`方法。

    $controller->isClassExtendedWith('Backend.Behaviors.RelationController');

下面是使用此方法动态扩展第三方插件的`UsersController`以避免其他插件也扩展上述第三方插件的示例。

    UsersController::extend(function($controller) {

        // 如果尚未实现，则实现行为
        if (!$controller->isClassExtendedWith('Backend.Behaviors.RelationController')) {
            $controller->implement[] = 'Backend.Behaviors.RelationController';
        }

        // 定义属性(如果尚未定义)
        if (!isset($controller->relationConfig)) {
            $controller->addDynamicProperty('relationConfig');
        }

        // 安全的拼接配置
        $myConfigPath = '$/myvendor/myplugin/controllers/users/config_relation.yaml';

        $controller->relationConfig = $controller->mergeConfig(
            $controller->relationConfig,
            $myConfigPath
        );

    }

### 软定义

如果行为类不存在(如特征trait)，则会抛出*Class not found*错误。 在某些情况下，如果系统中存在某种行为，您可能希望禁止此错误，以用于条件实现。 您可以通过在类名的开头放置一个`@`符号来完成此操作。

    class User extends \October\Rain\Extension\Extendable
    {
        public $implement = ['@RainLab.Translate.Behaviors.TranslatableModel'];
    }

如果类名称`RainLab\Translate\Behaviors\TranslatableModel`不存在，则不会引发任何错误。 这相当于以下代码：

    class User extends \October\Rain\Extension\Extendable
    {
        public $implement = [];

        public function __construct()
        {
            if (class_exists('RainLab\Translate\Behaviors\TranslatableModel')) {
                $this->implement[] = 'RainLab.Translate.Behaviors.TranslatableModel';
            }

            parent::__construct();
        }
    }

<a name="using-traits"></a>
### 使用Traits替代基础类

对于那些您可能不希望扩展`ExtensionBase`或`Extendable`类的情况，您可以使用这些特性。 您的类必须按如下方式实现：

首先让我们创建一个充当行为的类，即。 可以由其他类实现。

    <?php namespace MyNamespace\Behaviours;

    class WaveBehaviour
    {
        use \October\Rain\Extension\ExtensionTrait;

        /**
         * 使用Extensiontrait时，您的行为也必须实现此方法
         * @see \October\Rain\Extension\ExtensionBase
         */
        public static function extend(callable $callback)
        {
            self::extensionExtendCallback($callback);
        }

        public function wave()
        {
            echo "*waves*<br>";
        }
    }

现在让我们创建一个能够使用ExtendableTrait实现行为的类。

    class AI
    {
        use \October\Rain\Extension\ExtendableTrait;

        /**
         * @var array Extensions implemented by this class.
         */
        public $implement;
        /**
         * Constructor
         */
        public function __construct()
        {
            $this->extendableConstruct();
        }
        public function __get($name)
        {
            return $this->extendableGet($name);
        }
        public function __set($name, $value)
        {
            $this->extendableSet($name, $value);
        }
        public function __call($name, $params)
        {
            return $this->extendableCall($name, $params);
        }
        public static function __callStatic($name, $params)
        {
            return self::extendableCallStatic($name, $params);
        }
        public static function extend(callable $callback)
        {
            self::extendableExtendCallback($callback);
        }

        public function youGotBrains()
        {
            echo "I've got an AI!<br>";
        }
    }

AI类现在能够使用行为。 让我们扩展它并让这个类实现WaveBehaviour。

    <?php namespace MyNamespace\Classes;

    class Robot extends AI
    {
        public $implement = [
            'MyNamespace.Behaviours.WaveBehaviour'
        ];

        public function identify()
        {
            echo "I'm a Robot<br>";
            echo $this->youGotBrains();
            echo $this->wave();
        }
    }

您现在可以如下使用Robot：

        $robot = new Robot();
        $robot->identify();

会输出：

    I'm a Robot
    I've got an AI!
    *waves*

记得:
- 当使用`ExtensionTrait`时，应该将来自`ExtensionBase`的方法应用于该类。

- 当使用`ExtendableTrait`时，`Extendable`中的方法应该应用于该类。
