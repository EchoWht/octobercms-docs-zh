# 安装
[原文](https://github.com/octobercms/docs/blob/master/setup-installation.md)
- [系统最低要求](#system-requirements)
- [向导方式安装](#wizard-installation)
    - [安装中经常遇到的错误](#troubleshoot-installation)
- [命令行安装](#command-line-installation)
- [安装完成后](#post-install-steps)
    - [删除安装文件](#delete-install-files)
    - [检验配置文件](#config-review)
    - [设置调度器](#crontab-setup)
    - [设置队列](#queue-setup)
<a name="system-requirements"></a>
## 系统最低要求

October CMS 对应的服务器要求:

1. PHP的版本7.0及以上
1. 需安装PDO扩展
1. 需安装cURL扩展
1. 需安装OpenSSL扩展
1. 需安装Mbstring扩展
1. 需安装ZipArchive库
1. 需安装GD库

有些操作系统可能要求您手动安装PHP JSON和XML扩展。例如，当使用Ubuntu时，可以分别通过`apt-get install php7.0-json` and `apt-get install php7.0-xml` 命令来安装。

当使用SQLServer数据库引擎时，您需要安装[group concatenation](https://groupconcat.codeplex.com/) user-defined aggregate。

<a name="wizard-installation"></a>
## 向导方式安装(在线安装)

推荐使用向导方式来安装October。 它比命令行安装会更简单，不需要其他特殊技能。

1. 在服务器上新建一个空的目录。它可以是子目录、域名根目录或子域名的目录。
1. [下载安装文件](http://octobercms.com/download)。
1. 将安装程序存档解压到准备好的目录中。
1. 赋予安装目录及其所有子目录和文件的写入权限。
1. 在浏览器中访问install.php。
1. 根据提示进行安装。

![image](images/wizard-installer.png?raw=true)

<a name="troubleshoot-installation"></a>
### 安装中经常遇到的错误

1. **An error 500 is displayed when downloading the application files**: 您可能需要增加或禁用Web服务器上的超时限制。比如, Apache 的 FastCGI 有的会默认设置为30 秒`-idle-timeout`。

1. **A blank screen is displayed when opening the application**: 检查目录`/storage`中的文件及文件夹的权限是否可写。

1. **An error code "liveConnection" is displayed**: 安装程序将使用端口80测试安装服务器的连接。检查您的服务器是否可以通过PHP访问80端口。与您的主机提供商联系，或者经常在服务器防火墙设置中可发现。

1. **The back-end area displays "Page not found" (404)**: 如果应用程序找不到数据库，则后端将显示404页。尝试启用[debug mode](../setup/configuration#debug-mode)查看详细的错误信息。

> **注意:** 可以通过`install_files/install.log`文件查看详细日志。

<a name="command-line-installation"></a>
## 命令行安装

如果你更喜欢使用命令行或者使用composer来安装，程序提供了CLI进行安装[控制台页面](../console/commands#console-install)。

<a name="post-install-steps"></a>
## 安装完成后

在安装完成后，您可能需要进行一些设置。

<a name="delete-install-files"></a>
### 删除安装文件

如果您使用的是[向导安装](#wizard-installation) 出于安全原因，您应该删除安装文件。ctober永远不会自动删除系统中的文件，所以你应该手动删除这些文件和目录：

    install_files/      <== 安装文件夹
    install.php         <== 安装入口

<a name="config-review"></a>
### 检验配置文件

配置文件存放在应用的**config**文件夹。虽然每个文件都包含对每个设置的描述，但是重要的是要检查可用的[常见配置选项](../setup/configuration)。

例如，在生产环境中，您可能希望启用[CSRF 保护](../setup/configuration#csrf-protection)。当在开发环境中，您可能会启用[插件更新](../setup/configuration#edge-updates)。

虽然大多数配置是可选的，但是我们强烈建议在生产环境中禁用[调试模式](../setup/configuration#debug-mode)。

<a name="crontab-setup"></a>
### 设置调度器

关于*计划任务* 的正确操作, 你应该在你的服务器上添加以下Cron入口。编辑crontab通常使用`crontab -e`命令执行。

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
请确保将 **/path/to/artisan** 替换为October根目录下artisan文件的绝对路径。此Cron将每分钟调用命令调度程序。然后October评估所有计划任务并且执行预期的任务。

> **注意**: If you are adding this to `/etc/cron.d` 之后需要立即指定一个用户 `* * * * *`.

<a name="queue-setup"></a>
### 设置队列

您可以选择设置一个外部队列来处理排队的作业，默认情况下，这些工作将由平台异步处理。可以通过在“config/queue.php”中设置“default”参数来进行修改。
如果您决定使用`数据库`队列驱动程序，最好为命令“php artisan queue:work --once”添加一个Crontab入口，以处理队列中第一个可用的作业。