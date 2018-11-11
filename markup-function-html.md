# html()

以`html_`为前缀的函数执行在处理html标记时有用的任务。 帮助程序直接映射到`Html` PHP类及其方法。 例如：

    {{ html_strip() }}

是PHP的等价物如下：

    <?= Html::strip() ?>

> **注意**: *camelCase*中的方法应转换为*snake_case*。

## html_strip()

从字符串中删除HTML。

    {{ html_strip('<strong>Hello world</strong>') }}

## html_limit()

通过适当的标记处理限制具有特定长度的HTML。

    {{ html_limit('<p>Post content...</p>', 100) }}

要在应用限制时添加后缀，请将其作为第三个参数传递。 默认为“...”。

    {{ html_limit('<p>Post content...</p>', 100, '... Read more!') }}

## html_clean()

清除HTML以防止大多数XSS攻击。

    {{ html_clean('<script>window.location = "http://google.com"</script>') }}

## html_email()

混淆电子邮件地址以防止垃圾邮件机器人发送垃圾邮件。

    {{ html_email('a@b.c') }}

例如:

    <a href="mailto: {{ html_email('a@b.c')|raw }}">Email me</a>

    <!-- 以上将输出 -->
    <a href="mailto: &#109;&#97;&#105;&#108;&#x74;o&#x3a;&#97;&#64;b.&#x63;">Email me</a>
