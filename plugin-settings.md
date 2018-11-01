# 插件设置和配置

- [介绍](#introduction)
- [数据库设置](#database-settings)
    - [写入设置模型](#writing-settings)
    - [从设置模型中读取](#reading-settings)
- [后端设置页面](#backend-pages)
    - [设置链接注册](#link-registration)
    - [设置页面导航上下文](#settings-page-context)
- [基于文件的配置](#file-configuration)

<a name="introduction"></a>
## 介绍

有两种配置插件的方法 - 使用后端设置表单和配置文件。 使用具有后端页面的数据库设置可提供更好的用户体验，但它们会为初始开发带来更多开销。 基于文件的配置适用于很少修改的配置。

<a name="database-settings"></a>
## 数据库设置

您可以通过在模型类中实现`SettingsModel`行为来创建用于在数据库中存储设置的模型。 此模型可以直接用于创建后端设置表单。 您无需创建数据库表和控制器，以根据设置模型创建后端设置表单。

设置模型类应该扩展Model类并实现`System.Behaviors.SettingsModel`行为。 与任何其他模型一样，设置模型应在插件目录的**models**子目录中定义。 下一个示例中的模型应该在`plugins/acme/demo/models/Settings.php`脚本中定义。

    <?php namespace Acme\Demo\Models;

    use Model;

    class Settings extends Model
    {
        public $implement = ['System.Behaviors.SettingsModel'];

        // A unique code
        public $settingsCode = 'acme_demo_settings';

        // Reference to field configuration
        public $settingsFields = 'fields.yaml';
    }

设置模型需要`$settingsCode`属性。 它定义了唯一设置键，用于将设置保存到数据库。

如果要根据模型构建后端设置表单，则需要`$settingsFields`属性。 该属性指定包含表单字段定义的YAML文件的名称。 表单字段在[后端表单](../backend/forms)文章中描述。 应将YAML文件放在名称与小写的模型类名称匹配的目录中。 对于上一个示例中的模型，目录结构如下所示：

    plugins/
      acme/
        demo/
          models/
            settings/        <=== Model files directory
              fields.yaml    <=== Model form fields
            Settings.php     <=== Model script

设置模型[可以注册](#backend-pages)显示在**后端设置页面**上，但不是必需的 - 您可以像设置任何其他模型一样设置和读取设置值。

<a name="writing-settings"></a>
### 写入设置模型

设置模型具有静态“set”方法，允许保存单个或多个值。 您还可以使用标准模型功能来设置模型属性并保存模型。

    use Acme\Demo\Models\Settings;

    ...

    // Set a single value
    Settings::set('api_key', 'ABCD');

    // Set an array of values
    Settings::set(['api_key' => 'ABCD']);

    // Set object values
    $settings = Settings::instance();
    $settings->api_key = 'ABCD';
    $settings->save();

<a name="reading-settings"></a>
### 从设置模型中读取

设置模型具有静态`get`方法，使您可以加载单个属性。 此外，当您使用`instance`方法实例化模型时，它会从数据库加载属性，您可以直接访问它们。

    // Outputs: ABCD
    echo Settings::instance()->api_key;

    // Get a single value
    echo Settings::get('api_key');

    // Get a value and return a default value if it doesn't exist
    echo Settings::get('is_activated', true);


<a name="backend-pages"></a>
## 后端设置页面

后端包含用于容纳设置和配置的专用区域，可通过单击主菜单中的<strong>设置</ strong>链接进行访问。 “设置”页面包含指向系统和其他插件注册的配置页面的链接列表。

<a name="link-registration"></a>
### 设置链接注册

可以通过覆盖[Plugin注册类](registration#registration-file)中的`registerSettings`方法来扩展后端设置导航链接。 创建配置链接时，您有两个选项 - 创建指向特定后端页面的链接，或创建指向设置模型的链接。 下一个示例显示如何创建指向后端页面的链接。

    public function registerSettings()
    {
        return [
            'location' => [
                'label'       => 'Locations',
                'description' => 'Manage available user countries and states.',
                'category'    => 'Users',
                'icon'        => 'icon-globe',
                'url'         => Backend::url('acme/user/locations'),
                'order'       => 500,
                'keywords'    => 'geography place placement'
            ]
        ];
    }

> **注意:** 后端设置页面应[设置设置上下文](#settings-page-context)，以便在系统页面侧栏中标记相应的设置菜单项。 自动检测设置模型的设置上下文。

以下示例创建指向设置模型的链接。 设置模型是上面在[数据库设置](#database-settings)部分中描述的设置API的一部分。

    public function registerSettings()
    {
        return [
            'settings' => [
                'label'       => 'User Settings',
                'description' => 'Manage user based settings.',
                'category'    => 'Users',
                'icon'        => 'icon-cog',
                'class'       => 'Acme\User\Models\Settings',
                'order'       => 500,
                'keywords'    => 'security location',
                'permissions' => ['acme.users.access_settings']
            ]
        ];
    }

设置搜索功能使用可选的`keywords`参数。 如果未提供关键字，则搜索仅使用设置项标签和说明。

<a name="settings-page-context"></a>
### 设置页面导航上下文

就像[在控制器中设置导航上下文](../backend/controllers-views-ajax#navigation-context)一样，后端设置页面应该设置设置导航上下文。 为了将系统页面侧栏中的当前设置链接标记为活动，需要它。 使用`System \ Classes \ SettingsManager`类设置设置上下文。 通常可以在控制器构造函数中完成：

    public function __construct()
    {
        parent::__construct();

        [...]

        BackendMenu::setContext('October.System', 'system', 'settings');
        SettingsManager::setContext('You.Plugin', 'settings');
    }

`setContext`方法的第一个参数是以下格式的设置项所有者：**author.plugin**。 第二个参数是设置名称，与[注册后端设置页面](#link-registration)时提供的相同。

<a name="file-configuration"></a>
## 基于文件的配置

插件可以在插件目录的`config`子目录中有一个配置文件`config.php`。 配置文件是PHP脚本，用于定义和返回**数组**。 示例配置文件`plugins/acme/demo/config/config.php`:

    <?php

    return [
        'maxItems' => 10,
        'display' => 5
    ];

使用`Config`类访问配置文件中定义的配置值。 `Config::get($name, $default = null)``方法接受以下格式的插件和参数名称：**Acme.Demo::maxItems**。 第二个可选参数定义了在配置参数不存在时要返回的默认值。 例：

    use Config;

    ...

    $maxItems = Config::get('acme.demo::maxItems', 50);

应用程序可以通过创建配置文件`config/author/plugin/config.php`来覆盖插件配置，例如`config/acme/todo/dev/config.php`，或`config/acme/todo/config.php`适用于不同的环境。 在重写的配置文件中，您只能返回要覆盖的值：

    <?php

    return [
        'maxItems' => 20
    ];

如果你想在不同的环境中使用单独的配置（例如：**dev**，**production**），只需在`config/author/plugin/environment/config.php`中创建另一个文件。 用环境名称替换**环境**。 这将与`config/author/plugin/config.php`合并。

例如:

**config/author/plugin/production/config.php:**

    <?php

    return [
        'maxItems' => 25
    ];

当`APP_ENV`设置为 **production(生产环境)** 时，这会将`maxItems`设置为25。
