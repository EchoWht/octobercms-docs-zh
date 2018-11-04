# 扩展插件

- [根据事件扩展](#extending-with-events)
    - [订阅事件](#subscribing-to-events)
    - [声明事件](#declaring-events)
- [扩展后端视图](#backend-view-events)
- [用法示例](#usage-examples)
    - [扩展用户模型](#extending-user-model)
    - [扩展后端表单](#extending-backend-form)
    - [扩展后端列表](#extending-backend-list)
    - [扩展组件](#extending-component)
    - [扩展后端菜单](#extending-backend-menu)


<!-- Behaviors行为：Mixin概念，动态实现，扩展构造函数 -->
<!-- IoC/Facades:替换对象-->

<a name="extending-with-events"></a>
## 根据事件扩展

插件主要使用[事件服务](../services/events)进行扩展，以注入或修改核心类和其他插件的功能。

<a name="subscribing-to-events"></a>
### 订阅事件

订阅事件最常见的地方是[插件注册文件](registration#registration-methods)的`boot`方法。 例如，当用户首次注册时，您可能希望将它们添加到第三方邮件列表，这可以通过订阅**rainlab.user.register**全局事件来实现。

    public function boot()
    {
        Event::listen('rainlab.user.register', function($user) {
            // Code to register $user->email to mailing list
        });
    }

通过扩展模型的构造函数和使用本地事件可以实现相同的目的。

    User::extend(function($model) {
        $model->bindEvent('user.register', function() use ($model) {
            // Code to register $model->email to mailing list
        });
    });

<a name="declaring-events"></a>
### 声明事件

您可以在全局或本地声明事件，下面是声明全局事件的示例：

    Event::fire('acme.blog.beforePost', ['first parameter', 'second parameter']);

本地等效项要求代码位于调用对象的上下文中。

    $this->fireEvent('blog.beforePost', ['first parameter', 'second parameter']);

> **注意:** 将本地事件放在全局事件之前是一种好习惯，所以本地事件优先。

订阅此事件后，参数在处理程序方法中可用。 例如：

    // Global
    Event::listen('acme.blog.beforePost', function($param1, $param2) {
        echo 'Parameters: ' . $param1 . ' ' . $param2;
    });

    // Local
    $this->bindEvent('blog.beforePost', function($param1, $param2) {
        echo 'Parameters: ' . $param1 . ' ' . $param2;
    });

<a name="backend-view-events"></a>
## 扩展后端视图

有时您可能希望允许后端视图文件或部分扩展，例如工具栏。 这可以使用所有后端控制器中的`fireViewEvent`方法。

将此代码放在您的视图文件中：

    <div class="footer-area-extension">
        <?= $this->fireViewEvent('backend.auth.extendSigninView') ?>
    </div>

这将允许其他插件通过挂钩事件并返回所需的标记来将HTML注入此区域。

    Event::listen('backend.auth.extendSigninView', function($controller) {
        return '<a href="#">Sign in with Google!</a>';
    });

> **注意**: 事件处理程序中的第一个参数将始终是调用对象（控制器）。

上面的示例将输出以下标记：

    <div class="footer-area-extension">
        <a href="#">Sign in with Google!</a>
    </div>

<a name="usage-examples"></a>
## 用法示例

这些是如何使用事件的一些实际示例。

<a name="extending-user-model"></a>
### 扩展用户模型

此示例将通过绑定到其本地事件来修改`User`模型的`model.getAttribute`事件。 这是在[插件注册文件](registration#routing-initialization)的`boot`方法内执行的。 在两种情况下，当访问`$ model-> foo`属性时，它将返回值*bar*。

    class Plugin extends PluginBase
    {
        [...]

        public function boot()
        {
            // Local event hook that affects all users
            User::extend(function($model) {
                $model->bindEvent('model.getAttribute', function($attribute, $value) {
                    if ($attribute == 'foo') {
                        return 'bar';
                    }
                });
            });

            // Double event hook that affects user #2 only
            User::extend(function($model) {
                $model->bindEvent('model.afterFetch', function() use ($model) {
                    if ($model->id != 2) {
                        return;
                    }

                    $model->bindEvent('model.getAttribute', function($attribute, $value) {
                        if ($attribute == 'foo') {
                            return 'bar';
                        }
                    });
                });
            });
        }
    }

<a name="extending-backend-form"></a>
### 扩展后端表单

此示例将修改`Backend\Widget\Form`类的`backend.form.extendFields`全局事件，并在表单用于修改用户的条件下注入一些额外的字段值。 此事件也在[插件注册文件](registration#routing-initialization)的`boot`方法内订阅。

    class Plugin extends PluginBase
    {
        [...]

        public function boot()
        {
            // Extend all backend form usage
            Event::listen('backend.form.extendFields', function($widget) {

                // Only for the User controller
                if (!$widget->getController() instanceof \RainLab\User\Controllers\Users) {
                    return;
                }

                // Only for the User model
                if (!$widget->model instanceof \RainLab\User\Models\User) {
                    return;
                }

                // Add an extra birthday field
                $widget->addFields([
                    'birthday' => [
                        'label'   => 'Birthday',
                        'comment' => 'Select the users birthday',
                        'type'    => 'datepicker'
                    ]
                ]);

                // Remove a Surname field
                $widget->removeField('surname');
            });
        }
    }

如果您需要扩展后端表单并同时添加对翻译这些新字段的支持，则必须在呈现实际表单之前添加字段。 这可以通过`backend.form.extendFieldsBefore`事件来完成。
```
    public function boot()
    {
        Event::listen('backend.form.extendFieldsBefore', function($widget) {
            
            // You should always check to see if you're extending correct model/controller
            if(!$widget->model instanceof \Foo\Example\Models\Bar) {
                return;
            }
            
            // Here you can't use addFields() because it will throw you an exception because form is not yet created 
            // and it does not have tabs and fields
            // For this example we will pretend that we want to add a new field named example_field
            $widget->fields['example_field'] = [
                'label' => 'Example field',
                'comment' => 'Your example field',
                'type' => 'text',
            ];
            
            // Ok that's it about adding field inside form, now we need to tell our model that this field is translatable
        });
        
        // We will pretend that our model already implements RainLab Translatable behavior and $translatable property, 
        // if it does not you'll have to add it with addDynamicProperty()
        \Foo\Example\Models\Bar::extend(function($model) {
            $model->translatable[] = 'example_field';
        });
    }
```
> 请记住在类文件的顶部添加`use Event`以使用全局事件。

<a name="extending-backend-list"></a>
### 扩展后端列表

此示例将修改`Backend\Widget\Lists`类的`backend.list.extendColumns`全局事件，并在使用列表修改用户的条件下注入一些额外的列值。 此事件也在[插件注册文件](registration#routing-initialization)的`boot`方法内订阅。

    class Plugin extends PluginBase
    {
        [...]

        public function boot()
        {
            // Extend all backend list usage
            Event::listen('backend.list.extendColumns', function($widget) {

                // Only for the User controller
                if (!$widget->getController() instanceof \RainLab\User\Controllers\Users) {
                    return;
                }

                // Only for the User model
                if (!$widget->model instanceof \RainLab\User\Models\User) {
                    return;
                }

                // Add an extra birthday column
                $widget->addColumns([
                    'birthday' => [
                        'label' => 'Birthday'
                    ]
                ]);

                // Remove a Surname column
                $widget->removeColumn('surname');
            });
        }
    }

> 请记住在类文件的顶部添加`use Event`以使用全局事件。

<a name="extending-component"></a>
### 扩展组件

这个例子将在`Topic`组件中声明一个新的全局事件`rainlab.forum.topic.post`和名为`topic.post`的本地事件。 这在[Component class definition]（components＃component-class-definition）中执行。

    class Topic extends ComponentBase
    {
        public function onPost()
        {
            [...]

            /*
             * Extensibility
             */
            Event::fire('rainlab.forum.topic.post', [$this, $post, $postUrl]);
            $this->fireEvent('topic.post', [$post, $postUrl]);
        }
    }

接下来，这将演示如何从[页面执行生命周期](../cms/layouts#dynamic-pages)中挂钩这个新事件。 当在`Topic`组件（上面）中调用`onPost`事件处理程序时，这将写入跟踪日志。

    [topic]
    slug = "{{ :slug }}"
    ==
    function onInit()
    {
        $this['topic']->bindEvent('topic.post', function($post, $postUrl) {
            trace_log('A post has been submitted at '.$postUrl);
        });
    }

<a name="extending-backend-menu"></a>
### 扩展后端菜单

此示例将使用 *...* 替换后端中CMS和Pages的标签。

    class Plugin extends PluginBase
    {
        [...]

        public function boot()
        {
            Event::listen('backend.menu.extendItems', function($manager) {

                $manager->addMainMenuItems('October.Cms', [
                    'cms' => [
                        'label' => '...'
                    ]
                ]);

                $manager->addSideMenuItems('October.Cms', 'cms', [
                    'pages' => [
                        'label' => '...'
                    ]
                ]);

            });
        }
    }

同样，我们可以删除具有相同事件的菜单项：

    Event::listen('backend.menu.extendItems', function($manager) {

        $manager->removeMainMenuItem('October.Cms', 'cms');
        $manager->removeSideMenuItem('October.Cms', 'cms', 'pages');

    });
