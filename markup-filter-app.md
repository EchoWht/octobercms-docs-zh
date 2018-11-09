# |app

`|app`过滤器返回相对于网站公共路径的地址。 结果是绝对URL，包括域名和协议，到filter参数中指定的位置。 过滤器可以应用于任何路径。

    <link rel="icon" href="{{ '/favicon.ico'|app }}" />

如果网站地址是__http://octobercms.com__，则上面的示例将输出以下内容：

    <link rel="icon" href="http://octobercms.com/favicon.ico" />

它也可以用于静态URL：

    <a href="{{ '/about-us'|app }}">
        About Us
    </a>

以上将输出：

    <a href="http://octobercms.com/about-us">
        About us
    </a>

> **注意**: 建议使用`|page`过滤器链接到其他页面。
