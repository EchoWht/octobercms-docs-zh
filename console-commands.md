# 命令行界面

- [控制台安装](#console-install)
    - [快速安装](#console-install-quick)
    - [Composer安装](#console-install-composer)
- [设置和维护](#maintenance-commands)
    - [安装命令](#console-install-command)
    - [系统更新](#console-update-command)
    - [数据库迁移](#console-up-command)
- [插件管理](#plugin-commands)
    - [安装插件](#plugin-install-command)
    - [刷新插件](#plugin-refresh-command)
    - [删除插件](#plugin-remove-command)
- [主题管理](#theme-commands)
    - [安装主题](#theme-install-command)
    - [列出主题](#theme-list-command)
    - [启用主题](#theme-use-command)
    - [删除主题](#theme-remove-command)
- [实用命令](#utility-commands)
    - [清除应用缓存](#cache-clear-command)
    - [删除演示数据](#october-fresh-command)
    - [镜像公共目录](#cache-clear-command)
    - [启用DotEnv配置](#october-env-command)
    - [其他命令](#october-util-command)

October包括几个命令行界面(CLI)命令和实用程序，允许安装October，更新它，以及加快开发过程。 控制台命令基于Laravel的[Artisan](http://laravel.com/docs/artisan) 工具。 您可以[开发自己的控制台命令](console-development.md)或使用提供的[脚手架命令](console-scaffolding.md)加速开发。

<a name="console-install"></a>
## 控制台安装

可以使用本机系统或[Composer](http://getcomposer.org/) 来执行控制台安装，以管理依赖项。 这两种方法都会下载October的应用程序文件，可以立即使用。 如果您打算使用数据库，请确保在安装后运行[安装命令行](#console-install-command)。

<a name="console-install-quick"></a>
### 快速安装

在您的终端中运行此命令以获取October的最新的代码：

    curl -s https://octobercms.com/api/installer | php

如果你没有curl，可以执行以下命令：

    php -r "eval('?>'.file_get_contents('https://octobercms.com/api/installer'));"

<a name="console-install-composer"></a>
### Composer安装

在终端中使用`create-project`下载应用程序源代码。 以下命令将安装到名为 **/myoctober** 的目录中。

    composer create-project october/october myoctober

完成此任务后，打开文件 **config/cms.php** 并启用`disableCoreUpdates`设置。 这将禁用October提供的核心更新。

    'disableCoreUpdates' => true,
    
如果您正在开发一个站点并希望在更新时获得October的最新和最大的更改，那么更新`composer.json`文件以使用以下内容; 这使您可以测试开发分支的最新改进。

    "october/rain": "dev-develop as 1.0",
    "october/system": "dev-develop",
    "october/backend": "dev-develop",
    "october/cms": "dev-develop",
    "laravel/framework": "5.5.*@dev",

更新October时，在执行[数据库迁移](#console-up-command)之前，正常使用composer update命令。

    composer update

Composer被配置为查看插件目录内部的composer依赖，这些将包含在更新中。

> **注意:** 要将composer与使用[安装向导](../setup/installation#wizard-installation)安装的October实例一起使用，只需复制`tests/`目录，`composer.json`文件和`server.php `从[GitHub](https://github.com/octobercms/october) 文件到你的October实例，然后运行`composer install`。

<a name="maintenance-commands"></a>
## 设置和维护

<a name="console-install-command"></a>
### 安装命令

`october:install`命令将指导您完成首次设置OctoberCMS的过程。 它将询问数据库配置，应用程序URL，加密密钥和管理员详细信息。

    php artisan october:install

您还可以检查**config/app.php**和**config/cms.php**以更改其他的配置。

> **注意:** 运行`october:env`后无法运行`october:install`。 `october:env`获取现有的配置值，并将它们放在`.env`文件中，同时在配置文件中调用`env()`替换原始值。 `october:install`现在无法在配置文件中替换对`env()`的调用，因为管理过于复杂。

<a name="console-update-command"></a>
### 系统更新

`october:update`命令将从October网站api请求更新。 它将更新核心应用程序和插件文件，然后执行数据库迁移。

    php artisan october:update

> **注意**: 如果[使用composer安装](#console-install-composer)，则不会下载核心应用程序文件，并且在运行此命令之前应调用`composer update`。

<a name="console-up-command"></a>
### 数据库迁移

`october:up`命令将执行数据库迁移，创建数据库表并执行由系统提供的数据脚本和[插件版本历史](plugin-updates.md)。 迁移命令可以多次运行，它只会执行一次迁移或脚本，这意味着只应用新的更改。

    php artisan october:up

反向命令`october:down`将反转所有迁移，删除数据库表并删除数据。 使用此命令时应小心。 [插件刷新命令](#plugin-refresh-command)是调试单个插件的有用替代方法。

    php artisan october:down

<a name="plugin-commands"></a>
## 插件管理

October包含许多用于管理插件的命令。

<a name="plugin-install-command"></a>
### 安装插件

`plugin:install`  - 按名称下载并安装插件。 下一个示例将安装一个名为**AuthorName.PluginName**的插件。 请注意，您的安装应绑定到项目以使用此命令。 您可以在October的网站上[帐户/项目](https://octobercms.com/account/project/dashboard) 部分创建项目。

    php artisan plugin:install AuthorName.PluginName

<a name="plugin-refresh-command"></a>
### 刷新插件

`plugin:refresh` - 破坏插件的数据库表并重新创建它们。 此命令对开发很有用。

    php artisan plugin:refresh AuthorName.PluginName

<a name="plugin-remove-command"></a>
### 删除插件

`plugin:remove` - 破坏插件的数据库表并从文件系统中删除插件文件。

    php artisan plugin:remove AuthorName.PluginName

<a name="theme-commands"></a>
## 主题管理

October包含许多用于管理主题的命令。

<a name="theme-install-command"></a>
### 安装主题

`theme:install` - 从[Marketplace](https://octobercms.com/themes/)下载并安装主题。 以下示例将在`/themes/authorname-themename`中安装主题

    php artisan theme:install AuthorName.ThemeName

如果您希望在自定义目录中安装主题，只需提供第二个参数。 以下示例将下载`AuthorName.ThemeName`并将其安装在`/themes/my-theme`中

    php artisan theme:install AuthorName.ThemeName my-theme

<a name="theme-list-command"></a>
### 列出主题

`theme:list` - 列出已安装的主题。 使用 **-m** 选项在市场中包含热门主题。

    php artisan theme:list

<a name="theme-use-command"></a>
### 启用主题

`theme:use` - 切换活动主题。 以下示例将切换到`/themes/rainlab-vanilla`中的主题

    php artisan theme:use rainlab-vanilla

<a name="theme-remove-command"></a>
### 删除主题

`theme:remove` - 删除主题。 以下示例将删除目录`/themes/rainlab-vanilla`

    php artisan theme:remove rainlab-vanilla

<a name="utility-commands"></a>
## 实用命令

October包含许多实用程序命令。

<a name="cache-clear-command"></a>
### 清除应用缓存

`cache:clear` - 清除应用程序，twig和combiner缓存目录。 例：

    php artisan cache:clear

<a name="october-fresh-command"></a>
### 删除演示数据

`october:fresh` - 删除October随附的演示主题和插件。

    php artisan october:fresh

<a name="cache-clear-command"></a>
### 镜像公共目录

`october:mirror` - 使用符号链接创建服务应用程序所需的公共文件的镜像副本。 [设置公用文件夹](../setup/configuration#public-folder)时使用此命令。

    php artisan october:mirror public/

<a name="october-env-command"></a>
### 启用DotEnv配置

`october:env` - 将常见配置值更改为[DotEnv语法](../setup/configuration#dotenv-configuration)。

    php artisan october:env

<a name="october-util-command"></a>
### 其他命令

`october：util` - 执行常规实用程序任务的通用命令，例如清理文件或合并文件。 传递给此命令的参数将确定使用的任务。

#### 编译资源

编译输出JavaScript(js)，StyleSheets(less)，语言(lang)或所有(静态资源)文件。

    php artisan october:util compile assets
    php artisan october:util compile lang
    php artisan october:util compile js
    php artisan october:util compile less

要在没有缩小的情况下合并，请传参数`--debug`。

    php artisan october:util compile js --debug

#### 拉取更新

这将在所有主题和插件目录上执行命令`git pull`。

    php artisan october:util git pull

#### 清除缩略图

删除uploads目录中所有生成的缩略图。

    php artisan october:util purge thumbs
