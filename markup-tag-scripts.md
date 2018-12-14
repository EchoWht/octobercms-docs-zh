# {% scripts %}

`{％scripts％}`标记将JavaScript文件引用插入到应用程序注入的脚本中。 标记通常在结束BODY标记之前定义：

    <body>
        ...
        {% scripts %}
    </body>

> **注意**: 此标记应仅在给定的页面循环中出现一次，以防止重复引用。

## 注入脚本

可以通过[components](plugin-components.md#component-assets)或[pages](cms-pages.md#injecting-assets)以编程方式在PHP中注入JavaScript文件的链接。

    function onStart()
    {
        $this->addJs('assets/js/app.js');
    }

您还可以使用**脚本**匿名[占位符](cms-layouts.md#placeholders)将原始标记注入`{％scripts％}`标记。 在页面或布局中使用`{％put％}`标记将内容添加到占位符：

    {% put scripts %}
        <script type="text/javascript" src="/themes/demo/assets/js/menu.js"></script>
    {% endput %}
