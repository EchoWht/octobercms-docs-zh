# 安装
[原文](https://github.com/octobercms/docs/blob/master/setup-installation.md)
- [系统最低要求](#system-requirements)

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

推荐使用向导方式来安装October. 它比命令行安装会更简单，不需要其他特殊技能。

1. 在服务器上新建一个空的目录。它可以是子目录、域名根目录或子域名的目录。
1. [下载安装文件](http://octobercms.com/download).
1. 将安装程序存档解压到准备好的目录中。
1. 赋予安装目录及其所有子目录和文件的写入权限。
1. 在浏览器中访问install.php.
1. 根据提示进行安装.

![image](images/wizard-installer.png?raw=true)

<a name="troubleshoot-installation"></a>
### 安装中经常遇到的错误

1. **An error 500 is displayed when downloading the application files**: 您可能需要增加或禁用Web服务器上的超时限制。比如, Apache 的 FastCGI 有的会默认设置为30 秒`-idle-timeout`.

1. **A blank screen is displayed when opening the application**: 检查目录`/storage`中的文件及文件夹的权限是否可写.

1. **An error code "liveConnection" is displayed**: 安装程序将使用端口80测试安装服务器的连接。检查您的服务器是否可以通过PHP访问80端口。与您的主机提供商联系，或者经常在服务器防火墙设置中可发现。

1. **The back-end area displays "Page not found" (404)**: 如果应用程序找不到数据库，则后端将显示404页。尝试启用[debug mode](../setup/configuration#debug-mode)查看详细的错误信息.

> **注意:** 可以通过`install_files/install.log`文件查看详细日志.
