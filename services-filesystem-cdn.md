# 文件系统/ CDN

- [介绍](#introduction)
- [配置](#configuration)
- [基本用法](#basic-usage)
    - [获取磁盘实例](#obtaining-disk-instances)
    - [检索文件](#retrieving-files)
    - [存储文件](#storing-files)
    - [删除文件](#deleting-files)
    - [目录](#directories)

<a name="introduction"></a>
## 介绍

由于Laravel和精彩的[Flysystem](https://github.com/thephpleague/flysystem)PHP包，October提供了强大的文件系统抽象。 Flysystem集成提供了易于使用的驱动程序，可用于本地文件系统，Amazon S3和Rackspace云存储。 更好的是，在这些存储选项之间切换非常简单，因为每个系统的API保持不变。

<a name="configuration"></a>
## 配置

文件系统配置文件位于`config/filesystems.php`。 在此文件中，您可以配置所有“磁盘”。 每个磁盘代表一个特定的存储驱动程序和存储位置 配置文件中包含每个支持的驱动程序的示例配置。 因此，只需修改配置即可反映您的存储首选项和凭据。

当然，您可以根据需要配置任意数量的磁盘，甚至可能有多个使用相同驱动程序的磁盘。

#### 本地驱动

使用`local`驱动程序时，请注意所有文件操作都与配置文件中定义的`root`目录相关。 默认情况下，此值设置为`storage/app`目录。 因此，以下方法将文件存储在`storage/app/file.txt`中：

    Storage::disk('local')->put('file.txt', 'Contents');

#### 其他驱动的先决条件

在使用S3或Rackspace驱动程序之前，您需要安装[Drivers plugin]](http://octobercms.com/plugin/october-drivers)。

<a name="basic-usage"></a>
## 基本用法

<a name="obtaining-disk-instances"></a>
### 获取磁盘实例

“Storage”facade可用于与您配置的任何磁盘进行交互。 例如，您可以在立面上使用`put`方法将头像存储在默认磁盘上。 如果在没有先调用`disk`方法的情况下调用`Storage`facade上的方法，方法调用将自动传递给默认磁盘：

    $user = User::find($id);

    Storage::put(
        'avatars/'.$user->id,
        file_get_contents(Request::file('avatar')->getRealPath())
    );

使用多个磁盘时，您可以使用`Storage`facade上的`disk`方法访问特定磁盘。 当然，您可以继续链接方法来执行磁盘上的方法：

    $disk = Storage::disk('s3');

    $contents = Storage::disk('local')->get('file.jpg')

<a name="retrieving-files"></a>
### 检索文件

`get`方法可用于检索给定文件的内容。 该方法将返回该文件的原始字符串内容：

    $contents = Storage::get('file.jpg');

`exists`方法可用于确定磁盘上是否存在给定文件：

    $exists = Storage::disk('s3')->exists('file.jpg');

#### 文件元信息

`size`方法可用于以字节为单位获取文件的大小：

    $size = Storage::size('file1.jpg');

`lastModified`方法返回上次修改文件的UNIX时间戳：

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
### 存储文件

`put`方法可用于在磁盘上存储文件。 您也可以将PHP`resource`传递给`put`方法，该方法将使用Flysystem的底层流支持。 在处理大文件时，强烈建议使用流：

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

`copy`方法可用于将现有文件复制到磁盘上的新位置：

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

`move`方法可用于将现有文件移动到新位置：`move`方法可用于将现有文件移动到新位置：

    Storage::move('old/file1.jpg', 'new/file1.jpg');

#### 预先添加/附加到文件

`prepend`和`append`方法允许您轻松地在文件的开头或结尾插入内容：

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

<a name="deleting-files"></a>
### 删除文件

`delete`方法接受从磁盘中删除的单个文件名或文件数组：

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

<a name="directories"></a>
### 目录

#### 获取目录中的所有文件

`files`方法返回给定目录中所有文件的数组。 如果要检索给定目录（包括所有子目录）中的所有文件的列表，可以使用`allFiles`方法：

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### 获取目录中的所有目录

`directories`方法返回给定目录中所有目录的数组。 此外，您可以使用`allDirectories`方法获取给定目录及其所有子目录中所有目录的列表：

    $directories = Storage::directories($directory);

    // 递归...
    $directories = Storage::allDirectories($directory);

#### 创建一个目录

`makeDirectory`方法将创建给定目录，包括任何所需的子目录：

    Storage::makeDirectory($directory);

#### 删除目录

最后，`deleteDirectory`可用于从磁盘中删除目录，包括其所有文件：

    Storage::deleteDirectory($directory);
