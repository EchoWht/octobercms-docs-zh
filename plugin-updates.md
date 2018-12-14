# 版本历史

- [介绍](#introduction)
- [更新过程](#update-process)
    - [插件依赖](#plugin-depedencies)
- [插件版本文件](#version-file)
    - [重要更新](#important-updates)
    - [迁移和种子文件](#migration-seed-files)

<a name="introduction"></a>
## 介绍

插件维护更改日志是很好的做法，该日志记录代码中的任何更改或改进。 除了编写有关更改的注释之外，此过程还具有以正确顺序执行[迁移和种子文件](database-structure.md) 的有用功能。

更改日志存储在插件的 **/updates** 目录中名为`version.yaml`的YAML文件中，该文件与迁移和种子文件共存。 此示例显示典型的插件更新目录结构：

    plugins/
      author/
        myplugin/
          updates/                      <=== 更新文件夹
            version.yaml                <=== 插件版本文件
            create_tables.php           <=== 数据库脚本
            seed_the_database.php       <=== 迁移文件
            create_another_table.php    <=== 迁移文件

<a name="update-process"></a>
## 更新过程

在更新期间，系统将通知用户最近对插件的更改，它还可以提示他们[重要或重大更改](#important-updates)。 任何给定的迁移或种子文件只有在成功更新后才会被执行。 October发生以下任何一种情况时，October会自动执行更新过程：

1. 当管理员登录后端时。
1. 使用后端区域中的更新功能更新系统时。
1. 当从控制台目录的命令行中调用[console command](console-commands.md#console-up-command) `php artisan october:up` 时。

> **注意:** 插件[初始化过程](plugin-registration.md#routing-initialization)在更新过程中被禁用，这应该是迁移和播种脚本中的一个考虑因素。

<a name="plugin-depedencies"></a>
### 插件依赖

基于[插件注册文件中定义的依赖项](plugin-registration.md#dependency-definitions)，以特定顺序应用更新。会先更新依赖插件。

    <?php namespace Acme\Blog;

    class Plugin extends \System\Classes\PluginBase
    {
        public $require = ['Acme.User'];
    }

在以上示例中，**Acme.User**完全更新完成之后才更新**Acme.Blog** 。

<a name="version-file"></a>
## 插件版本文件

**version.yaml**文件， 叫做*扩展版本文件*, 包含版本注释内容和正确执行顺序的数据库脚本。请阅读[数据库结构](database-structure.md)文章来了解更多的迁移文件的信息。 如果您要将插件提交到[Marketplace](http://octobercms.com/help/site/marketplace)。则需要以下示例的版本文件：

    1.0.1: First version
    1.0.2: Second version
    1.0.3: Third version
    1.1.0: !!! Important update
    1.1.1:
        - Update with a migration and seed
        - create_tables.php
        - seed_the_database.php

以上所示，应该有一个代表更新消息后面的版本号的密钥，它可以是字符串，也可以是包含更新消息的数组。 对于引用迁移或种子文件的更新，第一行始终是注释，然后后续行是脚本文件名。 没有关联更新文件的注释示例：

    1.0.1: A single comment that uses no update scripts.

<a name="important-updates"></a>
### 重要更新

有时，插件需要引入会破坏已经使用插件的网站的功能。 如果** version.yaml **文件中的更新注释以三个惊叹号(`!!!`)开头，那么它将被视为*重要*并且将要求用户在更新之前进行确认。 重要更新评论的示例：

    1.1.0: !!! This is an important update that contains breaking changes.

当系统检测到重要更新时，它将提供三个选项以继续：

1. 确认更新
1. 跳过此插件(仅限一次)
1. 跳过这个插件(总是)

确认注释将照常更新插件，或者如果跳过注释，则不会更新。

<a name="migration-seed-files"></a>
### 迁移和种子文件

如前所述，更新还定义何时应该应用[迁移和种子文件](database-structure.md) 。 包含评论和更新的更新行：

    1.1.1:
        - This update will execute the two scripts below.
        - some_upgrade_file.php
        - some_seeding_file.php

更新文件名应使用*snake_case*，而包含的PHP类应使用*CamelCase*。 对于名为**some_upgrade_file.php**的文件，相应的类将是`SomeUpgradeFile`。

    <?php namespace Acme\Blog\Updates;

    use Schema;
    use October\Rain\Database\Updates\Migration;

    /**
     * some_upgrade_file.php
     */
    class SomeUpgradeFile extends Migration
    {
        ///
    }
