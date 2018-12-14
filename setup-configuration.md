# 应用配置

- [Web服务器配置](#webserver-configuration)
    - [Apache 配置](#apache-configuration)
    - [Nginx 配置](#nginx-configuration)
    - [Lighttpd 配置](#lighttpd-configuration)
    - [IIS 配置](#iis-configuration)
- [应用配置](#app-configuration)
    - [Debug 模式](#debug-mode)
    - [Safe 模式](#safe-mode)
    - [CSRF 保护](#csrf-protection)
    - [应用更新](#edge-updates)
    - [使用公共文件夹](#public-folder)
- [环境配置](#environment-config)
    - [定义基础环境](#base-environment)
    - [域名环境配置](#domain-environment)
    - [切换为DotEnv配置](#dotenv-configuration)

October的所有配置文件都放在**config**文件夹下，每个配置都有文档说明，您可以浏览文档并按自己的需要配置。

<a name="webserver-configuration"></a>
## Web服务器配置

October has basic configuration that should be applied to your webserver. Common webservers and their configuration can be found below.
October 具有应用于web服务器的基本配置。下面为常见的服务器及其配置。

<a name="apache-configuration"></a>
### Apache 配置

如果您的web服务器运行的时Apache，有一些额外的系统需求:
If your webserver is running Apache there are some extra system requirements:

1. 需要开启mod_rewrite模块。
1. 开启AllowOverride配置。

在某些情况下，您可能需要取消注释`.htaccess`文件中的这几行:

    ##
    ## You may need to uncomment the following line for some hosting environments,
    ## if you have installed to a subdirectory, enter the name here also.
    ##
    # RewriteBase /

如果您已经安装到子目录中，您还应该添加子目录的名称：

    RewriteBase /mysubdirectory/

<a name="nginx-configuration"></a>
### Nginx 配置

在 Nginx 环境下配置您的网站需要进行以下细微更改。

`nano /etc/nginx/sites-available/default`

在 **server** 区块中使用以下代码。如果 October 安装到子目录中，请将第一个/替换为 October 的安装目录：
    location/{
        # Let OctoberCMS handle everything by default.
        # The path not resolved by OctoberCMS router will return OctoberCMS's 404 page.
        # Everything that does not match with the whitelist below will fall into this.
        rewrite ^/.*$/index.php last;
    }

    # Pass the PHP scripts to FastCGI server
    location ~ ^/index.php {
        # Write your FPM configuration here

    }

    # Whitelist
    ## Let October handle if static file not exists
    location ~ ^/favicon\.ico { try_files $uri /index.php; }
    location ~ ^/sitemap\.xml { try_files $uri /index.php; }
    location ~ ^/robots\.txt { try_files $uri /index.php; }
    location ~ ^/humans\.txt { try_files $uri /index.php; }

    ## Let nginx return 404 if static file not exists
    location ~ ^/storage/app/uploads/public { try_files $uri 404; }
    location ~ ^/storage/app/media { try_files $uri 404; }
    location ~ ^/storage/temp/public { try_files $uri 404; }

    location ~ ^/modules/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/resources { try_files $uri 404; }
    location ~ ^/modules/.*/behaviors/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/behaviors/.*/resources { try_files $uri 404; }
    location ~ ^/modules/.*/widgets/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/widgets/.*/resources { try_files $uri 404; }
    location ~ ^/modules/.*/formwidgets/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/formwidgets/.*/resources { try_files $uri 404; }
    location ~ ^/modules/.*/reportwidgets/.*/assets { try_files $uri 404; }
    location ~ ^/modules/.*/reportwidgets/.*/resources { try_files $uri 404; }

    location ~ ^/plugins/.*/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/resources { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/behaviors/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/behaviors/.*/resources { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/reportwidgets/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/reportwidgets/.*/resources { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/formwidgets/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/formwidgets/.*/resources { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/widgets/.*/assets { try_files $uri 404; }
    location ~ ^/plugins/.*/.*/widgets/.*/resources { try_files $uri 404; }

    location ~ ^/themes/.*/assets { try_files $uri 404; }
    location ~ ^/themes/.*/resources { try_files $uri 404; }

<a name="lighttpd-configuration"></a>
### Lighttpd 配置

如果您的web服务器用的是Lighttpd，您可以使用以下配置来运行OctoberCMS，使用您常用的编辑器打开配置文件。

`nano /etc/lighttpd/conf-enabled/sites.conf`

请复制以下代码，然后更改  **host address** 和 **server.document-root** 来匹配您的项目。

    $HTTP["host"] =~ "domain.example.com" {
        server.document-root = "/var/www/example/"

        url.rewrite-once = (
            "^/(plugins|modules/(system|backend|cms))/(([\w-]+/)+|/|)assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
            "^/(system|themes/[\w-]+)/assets/([\w-]+/)+[-\w^&'@{}[\],$=!#().%+~/ ]+\.(jpg|jpeg|gif|png|svg|swf|avi|mpg|mpeg|mp3|flv|ico|css|js|woff|ttf)(\?.*|)$" => "$0",
            "^/storage/app/uploads/public/[\w-]+/.*$" => "$0",
            "^/storage/temp/public/[\w-]+/.*$" => "$0",
            "^/(favicon\.ico|robots\.txt|sitemap\.xml)$" => "$0",
            "(.*)" => "/index.php$1"
        )
    }

<a name="iis-configuration"></a>
### IIS 配置

如果您的web服务器用的是nternet 信息服务 (IIS)，您可以通过在 **web.config** 配置文件中使用以下配置来运行 October。
If your webserver is running Internet Information Services (IIS) you can use the following in your **web.config** configuration file to run OctoberCMS.

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <system.webServer>
            <rewrite>
                <rules>
                    <clear />
                    <rule name="OctoberCMS to handle all non-whitelisted URLs" stopProcessing="true">
                       <match url="^(.*)$" ignoreCase="false" />
                       <conditions logicalGrouping="MatchAll">
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/.well-known/*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/uploads/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/app/media/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/storage/temp/public/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/themes/.*/(assets|resources)/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/plugins/.*/(assets|resources)/.*" negate="true" />
                           <add input="{REQUEST_FILENAME}" matchType="IsFile" pattern="^/modules/.*/(assets|resources)/.*" negate="true" />
                       </conditions>
                       <action type="Rewrite" url="index.php" appendQueryString="true" />
                   </rule>
                </rules>
            </rewrite>
        </system.webServer>
    </configuration>

<a name="app-configuration"></a>
## 应用配置

<a name="debug-mode"></a>
### 调试模式

Debug调试模式的配置是通过`config/app.php` 文件中的`debug` 参数来设置的，默认为启用。

启用时，当遇到错误时此设置将显示详细的错误消息以及其他调试信息。虽然在开发过程中很有用，但在实际使用中，网站处于生产环境时，应始终禁用调试模式，来避免将敏感信息显示给最终的用户。

启用时，调试模式有以下功能：

1. 显示[错误的详细信息](../cms/pages#error-page)。
1. 显示用户身份验证失败的原因。
1. 默认情况下，[合并的静态资源](../markup/filter-theme) 不会被压缩。
1. 默认情况下，禁用[安全模式](#safe-mode) 。

> **重要**: 生产环境下，应将 `app.debug` 始终设置为 `false`

<a name="safe-mode"></a>
### 安全模式

安全模式的配置是通过`config/cms.php` 文件中的`enableSafeMode` 参数来设置的，默认为`null`。

如果启用安全模式，出于安全原因，CMS 模板中的 PHP 代码区块将被禁用。如果设置为 `null，则在禁用[debug调试模式](#debug-mode)时启用安全模式。

<a name="csrf-protection"></a>
### CSRF 保护

October 提供了一种简单的方法来保护您的程序不受跨站点请求伪造攻击。首先，用户 `session` 中将放置随机令牌。然后，当使用开始表单标记时，将令牌添加到页面并随每个请求一起提交。

默认情况下禁用 CSRF 保护，您可以使用 `config/cms.php` 配置文件中的 `enableCsrfProtection` 参数来启用。

<a name="edge-updates"></a>
### 应用更新

October和一些商店扩展会有实施两两种分支，以确保平台的整体稳定性和完整性，除了默认的`稳定版` 还有对应的`测试版`。

您可以通过更改`config/cms.php`配置文件中的`edgeUpdates` 参数来使用测试版或者稳定版。

    /*
    |--------------------------------------------------------------------------
    | Bleeding edge updates
    |--------------------------------------------------------------------------
    |
    | If you are developing with October, it is important to have the latest
    | code base, set this value to 'true' to tell the platform to download
    | and use the development copies of core files and plugins.
    |
    */

    'edgeUpdates' => false,

> **注意:** 对于插件开发人员，我们建议通过插件设置页面为市场上列出的插件启用 **Test updates(更新测试)** 

> **注意:** 如果使用[Composer](../console/commands#console-install-composer)管理更新, 然后使用以下内容替换`composer.json`文件中的默认OctoberCMS要求，以便直接从开发分支上下载更新。

    "october/rain": "dev-develop as 1.0",
    "october/system": "dev-develop",
    "october/backend": "dev-develop",
    "october/cms": "dev-develop",
    "laravel/framework": "5.5.*@dev",

<a name="public-folder"></a>
### 使用公用文件夹

为了在生产环境中获得最高安全性，您可以将Web服务器配置为使用 **public/** 文件夹，以确保只能访问公共文件。首先，您需要使用`october：mirror`命令生成公用文件夹。

    php artisan october:mirror public/

这将在项目的基础目录中创建一个名为 **public/** 的新目录，从此处您应该修改Web服务器配置以使用此新路径作为主目录，也称为 *wwwroot* 。
  
> **注意**: 可能需要使用系统管理员或 *sudo* 权限执行上述命令。在每次系统更新后或安装新插件时执行也应该使用系统管理员或 *sudo* 权限。

<a name="environment-config"></a>
## 环境配置

<a name="base-environment"></a>
### 定义基础环境

根据运行应用程序的环境设置不同的配置值通常是很有用的。您可以通过设置`APP_ENV`环境变量来实现，默认情况下它设置为 **production** 。有两种常用方法可以更改此值：

1. 直接在您的web服务器上设置`APP_ENV`值

    例如，在Apache中，下面这一行可以添加到`.htaccess`或`httpd.config`文件中：

        SetEnv APP_ENV "dev"

2. 在根目录中创建 **.env** 文件 添加以下代码：

        APP_ENV=dev

在上述两个示例中，环境都设置为新值`dev`。现在可以在路径 **config/dev** 中创建配置文件，并覆盖应用程序的基本配置。

例如，要为`dev`环境使用不同的MySQL数据库，复制以下代码，创建名为 **config/dev/database.php** 的文件：

    <?php

    return [
        'connections' => [
            'mysql' => [
                'host'     => 'localhost',
                'port'     => '',
                'database' => 'database',
                'username' => 'root',
                'password' => ''
            ]
        ]
    ];

<a name="domain-environment"></a>
### 域名环境配置

October支持使用特定hostname检测到的环境。您可以将这些主机名放在环境配置文件中，例如 **config/environment.php**。

使用下面的文件内容，当通过 **global.website.tld**访问应用程序时，环境将设置为`global`，同样适用于其他环境。

    <?php

    return [
        'hosts' => [
            'global.website.tld' => 'global',
            'local.website.tld' => 'local',
        ]
    ];

<a name="dotenv-configuration"></a>
### 切换为DotEnv配置

作为[基本环境配置](#base-environment)的替代方法，您可以在环境中放置常用值，而不是使用配置文件。然后使用[DotEnv](https://github.com/vlucas/phpdotenv)语法访问配置。运行`october：env`命令将公共配置值移动到环境中:

    php artisan october:env

这将在项目根目录中创建一个 **.env**文件，并修改配置文件以使用`env` helper函数。第一个参数包含在环境中找到的键名，第二个参数包含可选的默认值。

    'debug' => env('APP_DEBUG', true),

您的`.env`文件不应该提交给应用程序的版本控制工具，因为使用您的应用程序的每个开发人员或服务器都可能需要不同的环境配置。
