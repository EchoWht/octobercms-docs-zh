# |media

`|media`过滤器返回相对于[媒体管理库](../cms/mediamanager)的公共路径的地址。 结果是filter参数中指定的媒体文件的URL。

    <img src="{{ 'banner.jpg'|media }}" />

如果媒体管理器地址是__http://cdn.octobercms.com__，则上面的示例将输出以下内容：

    <img src="http://cdn.octobercms.com/banner.jpg" />
