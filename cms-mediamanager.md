# 多媒体文件管理

- [Amazon S3](#amazon-s3)
- [Rackspace CDN](#rackspace-cdn)
- [音频和视频播放器](#audio-and-video-players)
- [其他配置选项](#configuration-options)
- [事件](#events)
- [问题排查](#troubleshooting)

默认情况下，文件管理器使用目录是 storage/app/media子目录。要使用Amazon S3或Rackspace CDN，您需要修改系统配置。

> 您需要先安装[Drivers plugin](http://octobercms.com/plugin/october-drivers)才能使用Amazon S3或Rackspace CDN功能。

请注意，更改介文件管理器配置后，应重置其缓存。您可以通过按媒体管理器工具栏中的**刷新**按钮来完成此操作。

<a name="amazon-s3"></a>
## 配置Amazon S3

要将Amazon S3与OctoberCMS一起使用，您应该在bucket和API用户中创建S3 bucket，文件夹。

注册Amazon AWS账户或使用现有帐户登录AWS控制台。打开S3管理面板。创建一个新bucket并为其分配其名称(bucket的名称将是您的公共文件URL的一部分)

在bucket中创建 **media** 文件夹。文件夹名称无关紧要。此文件夹将是媒体库的根目录。

默认情况下，无法直接访问S3bucket中的文件。要使bucket处于公共状态，请返回bucket列表并单击bucket。单击右侧边栏中的**属性**按钮。展开**权限**标签。单击**编辑bucket策略**链接。将以下代码粘贴到策略弹出窗口。将桶名称替换为您的实际bucket名称：

    {
        "Version": "2008-10-17",
        "Id": "Policy1397632521960",
        "Statement": [
            {
                "Sid": "Stmt1397633323327",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "*"
                },
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::BUCKETNAME/*"
            }
        ]
    }

单击**保存**按钮以应用策略。该策略提供对bucket中所有文件夹和目录的公共只读访问权限。如果您打算将bucket用于其他需求，则可以设置对bucket中特定文件夹的公共访问权限，只需在** Resource **值中指定目录名称： 

    "arn:aws:s3:::BUCKETNAME/media/*"

您还应该创建一个API用户，OctoberCMS将使用该用户来管理bucket文件。在AWS控制台中，转到IAM部分。转到“用户”选项卡并创建新用户。用户名无关紧要。确保在创建新用户时选中“为每个用户生成访问密钥”复选框。 AWS创建用户后，它允许您查看安全凭证 - 用户**访问密钥ID**和**秘密访问密钥**。复制密钥并将其放入临时文本文件中。

返回用户列表，然后单击刚刚创建的用户。在**Permissions**部分中，单击**Attach Policy**按钮。在列表中选择**AmazonS3FullAccess**策略，然后单击**附加策略**按钮。

现在您拥有更新OctoberCMS配置的所有信息。打开**config/filesystem.php**脚本并找到**disks**部分。它已包含s3配置，您需要替换API凭据和bucket信息参数：

参数 | 值
------------- | -------------
**key** | the **Access Key ID** value of the user that you created before.
**secret** | the **Secret Access Key** value of the user that you created fore.
**bucket** | your bucket name.
**region** | the bucket region code, see below.

您可以在存储区属性中的S3管理控制台中找到存储区域。 “属性”选项卡显示区域名称，例如Oregon。 S3驱动程序配置需要bucket代码。使用此表查找bucket的代码(您还可以查看[AWS文档](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region))

Region | Code
------------- | -------------
<span class="nowrap">**US Standard**</span> | us-east-1
<span class="nowrap">**US West (Oregon)**</span> | us-west-2
<span class="nowrap">**US West (N. California)**</span> | us-west-1
<span class="nowrap">**EU (Ireland)**</span> | eu-west-1
<span class="nowrap">**EU (Frankfurt)**</span> | eu-central-1
<span class="nowrap">**Asia Pacific (Singapore)**</span> | ap-southeast-1
<span class="nowrap">**Asia Pacific (Sydney)**</span> | ap-southeast-2
<span class="nowrap">**Asia Pacific (Tokyo)**</span> | ap-northeast-1
<span class="nowrap">**South America (Sao Paulo)**</span> | sa-east-1

例如：

    'disks' => [
        ...
        's3' => [
            'driver' => 's3',
            'key'    => 'XXXXXXXXXXXXXXXXXXXX',
            'secret' => 'xxxXxXX+XxxxxXXxXxxxxxxXxxXXXXXXXxxxX9Xx',
            'region' => 'us-west-2',
            'bucket' => 'my-bucket'
        ],
        ...
    ]

Save **config/filesystem.php** script and open **config/cms.php** script. Find the section **storage**. In the **media** parameter update **disk**, **folder** and **path** parameters: 

Parameter | Value
------------- | -------------
**disk** | use **s3** value.
**folder** | the name of the folder you created in S3 bucket.
**path** | the public path of the folder in the bucket, see below.

To obtain the path of the folder, open AWS console and go to S3 section. Navigate to the bucket and click the folder you created before. Upload any file to the folder and click the file. Click **Properties** button in the right sidebar. The file URL is in the **Link** parameter. Copy the URL and remove the file name and the trailing slash from it.

Example storage configuration:

    'storage' => [
        ...
        'media' => [
            'disk'   => 's3',
            'folder' => 'media',
            'path' => 'https://s3-us-west-2.amazonaws.com/your-bucket-name/media'
        ]
    ]

Congratulations! Now you're ready to use Amazon S3 with OctoberCMS. Note that you can also configure Amazon CloudFront CDN  to work with your bucket. This topic is not covered in this document, please refer to [CloudFront documentation](http://aws.amazon.com/cloudfront/). After you configure CloudFront, you will need to update the **path** parameter in the storage configuration.

<a name="rackspace-cdn"></a>
## Configuring Rackspace CDN acces 这段先不翻了，国内网络应该用不了吧(🐶手动狗头)。

To use Rackspace CDN with OctoberCMS, you should create Rackspace CDN container, folder in the container and API user.

Log into Rackspace management console and navigate to Storage/Files page. Create a new container. The container name doesn't matter, it will be a part of your public file URLs. Select **Public (Enabled CDN)** type for the new container.

Create **media** folder in the container. The folder name doesn't matter. This folder will be a root of your Media Library.

You should create an API user that OctoberCMS will use for managing files in the CDN container. Open Account/User Management page in Rackspace console. Click **Create user** button. Fill in the user name (for example october.cdn.api), password, security question and answer. In the **Product Access** section select **Custom** and in the CDN row select **Admin**. Use **No Access** role in the **Account** section and use **Technical Contact** type in the **Contact Information** section. Save the user account. After saving the account you will see the Login Details section with the **API Key** row that contains a value you need to use in OctoberCMS configuration files.

Now you have all the information to update OctoberCMS configuration. Open **config/filesystem.php** script and find the **disks** section. It already contains Rackspace configuration, you need to replace the API credentials and container information parameters:

Parameter | Value
------------- | -------------
**username** | Rackspace user name (for example october.cdn.api).
**key** | the user's **API Key** that you can copy from Rackspace user profile page.
**container** | the container name.
**region** | the bucket region code, see below.
**endpoint** | leave the value as is.
**region** | you can find the region in the CDN container list in Rackspace control panel. The code is a 3-letter value, for example it's **ORD** for Chicago. 

Example configuration after update:

    'disks' => [
        ...
        'rackspace' => [
            'driver'    => 'rackspace',
            'username'  => 'october.api.cdn',
            'key'       => 'xx00000000xxxxxx0x0x0x000xx0x0x0',
            'container' => 'my-bucket',
            'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
            'region'    => 'ORD'
        ],
        ...
    ]

Save **config/filesystem.php** script and open **config/cms.php** script. Find the section **storage**. In the **media** parameter update **disk**, **folder** and **path** parameters: 

Parameter | Value
------------- | -------------
**disk** | use **rackspace** value.
**folder** | the name of the folder you created in CDN container.
**path** | the public path of the folder in the container, see below.

To obtain the path of the folder, go to the CDN container list in Rackspace console. Click the container and open the media folder. Upload any file. After the file is uploaded, click it. The file will open in a new browser tab. Copy the file URL and remove the file name and trailing slash from it. 

Example storage configuration:

    'storage' => [
        ...
        'media' => [
            'disk'   => 'rackspace',
            'folder' => 'media',
            'path' => 'https://xxxxxxxxx-xxxxxxxxx.r00.cf0.rackcdn.com/media'
        ]
    ]

Congratulations! Now you're ready to use Rackspace CDN with OctoberCMS.

<a name="audio-and-video-players"></a>
## 音频和视频播放器

默认情况下，系统使用HTML5音频和视频标签来播放音频和视频文件：

    <video src="video.mp4" controls></video>

或

    <audio src="audio.mp3" controls></audio>

这种方法是可以覆盖的。如果有 **oc-audio-player.html** 和 **oc-video-player.html** CMS partials，它们将用于显示音频和视频内容。在partials内部使用变量 **src** 输出到源文件的链接。例：

    <video src="{{ src }}" width="320" height="200" controls preload></video>


如果您不想使用HTML5播放器，则可以在partial中提供其他标签。有一个[第三方脚本](https://html5media.info/)，支持旧版浏览器中的HTML5视频和音频标签。

由于部分是使用Twig编写的，因此您可以根据命名约定自动添加备用视频源。例如，如果有一个约定，即每个全分辨率视频的分辨率视频总是较小，而较小分辨率的文件扩展名为“iphone.mp4”，则生成的标记可能如下所示：

    <video controls>
        <source
            src="{{ src }}"
            media="only screen and (min-device-width: 568px)"></source>
        <source
            src="{{ src|replace({'.mp4': '.iphone.mp4'}) }}"
            media="only screen and (max-device-width: 568px)"></source>
    </video>

<a name="configuration-options"></a>
## 其他配置选项

There are several options that allow you to fine-tune the Media Manager. All of them could be defined in **config/cms.php** script, in the **storage/media** section, for example:
有几个选项可以让您微调媒体管理器。所有这些都可以在 **storage/media**部分的 **config/cms.php**脚本中定义，例如：
    'storage' => [
        ...

        'media' => [
            ...
            'ignore' => ['.svn', '.git', '.DS_Store']
        ]
    ],


参数名 | 参数值的含义
------------- | -------------
**ignore** | 要忽略的文件和目录名称列表。默认为['.svn', '.git', '.DS_Store'].
**ttl** | 指定缓存的生存时间，以分钟为单位。默认值为10.添加，更新或删除库项目时，缓存会自动失效。
**imageExtensions** | 与图片类型对应的文件扩展名。默认值为 **['gif', 'png', 'jpg', 'jpeg', 'bmp']**.
**videoExtensions** | 与视频类型对应的文件扩展名。默认值为 **['mp4', 'avi', 'mov', 'mpg']**.
**audioExtensions** | 与音频类型对应的文件扩展名。默认值为**['mp3', 'wav', 'wma', 'm4a']**.

<a name="events"></a>
## 活动

媒体管理器提供了一些您可以监听的[事件](services-events.md)，以提高可扩展性。

事件 | 描述 | 参数
------------- | ------------- | -------------
**folder.delete** | 文件夹删除时执行 | `(string) $path`
**file.delete** | 文件删除时执行 | `(string) $path`
**folder.rename** | 文件夹重命名时执行 | `(string) $originalPath`, `(string) $newPath`
**file.rename** | 文件重命名时执行 | `(string) $originalPath`, `(string) $newPath`
**folder.create** | 文件夹创建时执行 | `(string) $newFolderPath`
**folder.move** | 文件夹移动时执行 | `(string) $path`, `(string) $dest`
**file.move** | 文件移动时执行 | `(string) $path`, `(string) $dest`
**file.upload** | 文件上传时执行 | `(string) $filePath`, `(\Symfony\Component\HttpFoundation\File\UploadedFile) $uploadedFile`

**要使用这些钩子事件，可以直接继承`Backend\Widgets\MediaManager`类：**

    Backend\Widgets\MediaManager::extend(function($widget) {
        $widget->bindEvent('file.rename', function ($originalPath, $newPath) {
            // Update custom references to path here
        });
    });
    
**或者通过`Event` facade来全局监听(每个事件都以`media`为前缀，并将实例化的`Backend\Widgets\MediaManager`对象作为第一个参数传递)：**

    Event::listen('media.file.rename', function($widget, $originalPath, $newPath) {
        // Update custom references to path here
    });

<a name="troubleshooting"></a>
## 问题处理

使用远程服务的最常见问题是SSL连接问题。如果您收到SSL错误，请确保您的服务器具有公共证书颁发机构(CA)的新SSL证书。
