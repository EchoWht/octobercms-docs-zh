# 脚手架命令

- [脚手架命令](#scaffolding-commands)
    - [创建一个插件](#scaffold-create-plugin)
    - [创建一个组件](#scaffold-create-component)
    - [创建一个模型](#scaffold-create-model)
    - [创建一个后端控制器](#scaffold-create-controller)
    - [创建表单小部件](#scaffold-create-formwidget)
    - [创建一个控制台命令](#scaffold-create-command)

<a name="scaffolding-commands"></a>
## 脚手架命令

使用scaffolding命令加快开发进程。

<a name="scaffold-create-plugin"></a>
### 创建一个插件

`create:plugin`命令为插件生成插件文件夹和基本文件。 该参数指定作者和插件名称。

    php artisan create:plugin Acme.Blog

<a name="scaffold-create-component"></a>
### 创建一个组件

`create:component`命令创建一个新的组件类和默认的组件视图。 第一个参数指定作者和插件名称。 第二个参数指定组件类名称。

    php artisan create:component Acme.Blog Post

<a name="scaffold-create-model"></a>
### 创建一个模型

`create:model`命令生成新模型所需的文件。 第一个参数指定作者和插件名称。 第二个参数指定模型类名称。

    php artisan create:model Acme.Blog Post

<a name="scaffold-create-controller"></a>
### 创建一个后端控制器

`create:controller`命令生成控制器，配置和视图文件。 第一个参数指定作者和插件名称。 第二个参数指定控制器类名。

    php artisan create:controller Acme.Blog Posts

<a name="scaffold-create-formwidget"></a>
### 创建表单小部件

`create:formwidget`命令生成后端表单窗口小部件，视图和基本资源文件。 第一个参数指定作者和插件名称。 第二个参数指定表单窗口小部件类名称。

    php artisan create:formwidget Acme.Blog CategorySelector

<a name="scaffold-create-command"></a>
### 创建一个控制台命令

`create:command`命令生成[新控制台命令](console-development.md)。 第一个参数指定作者和插件名称。 第二个参数指定命令名称。

    php artisan create:command RainLab.Blog MyCommand
