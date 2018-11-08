# 数据库行为

- [可清除的](#purgeable)

模型行为用于实现通用功能。 
与[Traits(特征)](traits)不同，这些可以直接实现
in a class or by extending the class. You can read more about behaviors [here](../services/behaviors).
在class类中或通过扩展class类。 您可以阅读更多有关行为[此处](../services/behaviors)的内容。

<a name="purgeable"></a>
## 可清除的

创建或更新模型时，清除的属性不会保存到数据库中。 清除
模型中的属性，实现`October.Rain.Database.Behaviors.Purgeable`行为并声明
一个`$purgeable`属性，带有一个包含要清除的属性的数组。

    class User extends Model
    {
        public $implement = [
            'October.Rain.Database.Behaviors.Purgeable'
        ];

        /**
         * @var array List of attributes to purge.
         */
        public $purgeable = [];
    }
    
您还可以在类中动态实现此行为。

    /**
     * 扩展RainLab.User用户模型以实现可清除行为。
     */
    RainLab\User\Models\User::extend(function($model) {

        // Implement the purgeable behavior dynamically
        $model->implement[] = 'October.Rain.Database.Behaviors.Purgeable';
        
        // Declare the purgeable property dynamically for the purgeable behavior to use
        $model->addDynamicProperty('purgeable', []);
    });

保存模型时，在[模型事件](#model-events)之前将清除定义的属性
被触发，包括验证。 使用`getOriginalPurgeValue`查找已清除的值。

    return $user->getOriginalPurgeValue($propertyName);
