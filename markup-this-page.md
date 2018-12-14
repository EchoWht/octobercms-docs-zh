# this.page

您可以通过`this.page`访问当前页面对象，它返回对象`Cms\Classes\Page`。 该对象也可以[在PHP代码中访问](cms-pages.md/#page-variables).。

## 属性

`this.page`具有以下属性。

### layout

如果已定义，则引用此页面使用的布局名称。 不要与`this.layout`混淆。

    {{ this.page.layout }}

### id

将页面文件名和文件夹名称转换为CSS友好标识符。

    <body class="page-{{ this.page.id }}">

如果页面文件是**home/index.htm**，那么这将生成一个类名为`page-home-index`。

### title

页面标题由配置定义。

    <h1>{{ this.page.title }}</h1>

### description

配置定义的页面描述。

    <p>{{ this.page.description }}</p>

### meta_title

一个替代的`标题'字段，通常更具描述性的SEO目的。

    <title>{{ this.page.meta_title }}</title>

### meta_description

一个替代的`description`字段，通常更具描述性的SEO目的。

    <meta name="description" content="{{ this.page.meta_description }}">

### hidden

只有登录的后端用户才能访问隐藏页面。

    {% if this.page.hidden %}
        <p>Note to other admins: We are currently working on this page.</p>
    {% endif %}

### fileName

带有扩展名的主题中的页面文件名。

### baseFileName

没有扩展名的主题中的页面文件名。
