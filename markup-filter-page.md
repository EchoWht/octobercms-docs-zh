# |page

`|}page`过滤器使用页面文件名(没有扩展名)作为参数创建指向页面的链接。 例如，如果有about.htm页面，您可以使用以下代码生成指向它的链接：

    <a href="{{ 'about'|page }}">About Us</a>

请记住，如果您从子目录中引用页面，则应指定子目录名称：

    <a href="{{ 'contacts/about'|page }}">About Us</a>

> **注意**: [主题文档](cms-themes.md#subdirectories)有关于子目录用法的更多详细信息。

您可以通过过滤空字符串来创建指向当前页面的链接：

    <a href="{{ ''|page }}">Refresh page</a>

<a name="reverse-routing"></a>
## 反向路由

当链接到定义了URL参数的页面时，`|page`过滤器通过将数组作为第一个参数传递来支持反向路由。

    url = "/blog/post/:post_id"
    ==
    [...]

鉴于以上内容可在CMS页面文件**post.htm**中找到，您可以使用以下链接链接到此页面：

    <a href="{{ 'post'|page({ post_id: 10 }) }}">
        Blog post #10
    </a>

如果网站地址是__http：//octobercms.com__，则上面的示例将输出以下内容：

    <a href="http://octobercms.com/blog/post/10">
        Blog post #10
    </a>

<a name="persistent-parameters"></a>
## 持久性URL参数

如果已在环境中显示URL参数，则`| page`过滤器将自动使用它。

    url = "/blog/post/:post_id"

    url = "/blog/post/edit/:post_id"

如果有两个页面，**post.htm**和**post-edit.htm**，并且定义了上述URL，则可以链接到任一页面而无需定义`post_id`参数。

    <a href="{{ 'post-edit'|page }}">
        Edit this post
    </a>

当**post.htm**页面上出现上述标记时，它将输出以下内容：

    <a href="http://octobercms.com/blog/post/edit/10">
        Edit this post
    </a>

如果当前页面中的id真实存在， 您可以通过将第二个参数传递为“false”来禁用在此页面显示：

    <a href="{{ 'post'|page(false) }}">
        Unknown blog post
    </a>

或者通过定义默认的值：

    <a href="{{ 'post'|page({ post_id: 6 }) }}">
        Blog post #6
    </a>
