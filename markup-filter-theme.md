# |theme

`|theme`过滤器返回相对于网站的活动主题路径的地址。 结果是绝对URL，包括域和协议，指向filter参数中指定的资源路径。 主题资源通常位于主题目录的**assets**子目录中。

    <script type="text/javascript" src="{{ 'assets/js/menu.js'|theme }}"></script>

如果网站地址是__http://octobercms.com__并且活动主题称为“网站”，则上面的示例将输出以下内容：

    <script type="text/javascript" src="http://october.com/themes/website/assets/js/menu.js"></script>

<a name="combine-css-javascript"></a>
## 合并CSS和JavaScript

过滤器还可用于通过传递文件数组来合并相同类型的资源。

    <link href="{{ [
        'assets/css/styles1.css',
        'assets/css/styles2.css'
    ]|theme }}" rel="stylesheet">

> **注意**: 您可以使用`config/cms.php`脚本中的`enableAssetMinify`参数启用资源压缩。 默认情况下，禁用压缩。

<a name="combiner-aliases"></a>
### 合并器别名

合并组合器支持替换文件路径的常用别名，这些别名将以`@`符号开头。 例如，[AJAX框架资产](../ajax/introduction#framework-script)可以包含在合并器中：

    <script src="{{ [
        '@jquery',
        '@framework',
        '@framework.extras',
        'assets/javascript/app.js'
    ]|theme }}"></script>

支持以下别名：

别名 | 描述
------------- | -------------
`@jquery` | 引用后端使用的jQuery库（v2.1.3）。（JavaScript）
`@framework` | AJAX框架附加，替换为`{％framework％}`标签。（JavaScript）
`@framework.extras` | AJAX框架附加组件，替换为`{％framework extras％}`标签。 （JavaScript，CSS）

相同的别名可用于JavaScript或CSS，例如`@framework.extras`。 在列表中至少需要一个带有文件扩展名的显式引用来确定使用哪个。

<a name="external-combiner-paths"></a>
### 外部组合器路径

在某些情况下，您可能希望在主题之外组合文件，这可以通过在路径前添加符号来创建动态路径来实现。 例如，以`〜/`开头的路径将创建相对于应用程序的路径：

    <script src="{{ ['~/modules/system/assets/js/framework.js']|theme }}"></script>

支持这些符号来创建动态路径：

符号 | 描述
------------- | -------------
`$` | 相对于plugins目录
`~` | 相对于应用程序目录
