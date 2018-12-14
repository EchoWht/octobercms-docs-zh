# 模板

October扩展了[Twig模板语言](http://twig.sensiolabs.org/documentation) ，其中包含许多函数，标签，过滤器和变量。 这些扩展允许您使用CMS功能并访问模板中的页面环境信息。

## 变量

使用*双花括号*在页面上打印模板变量。

    {{ variable }}

变量也可以表示*表达式*。

    {{ isAjax ? 'Yes' : 'No' }}

变量可以与`~`字符连接。

    {{ 'Your name: ' ~ name }}

十月在`this`变量下提供全局变量，如**变量**部分所列。

## 标签

标签是Twig的一个独特功能，包含`{%%}`字符。

    {% tag %}

标签提供了更流畅的方式来描述模板逻辑。

    {% if stormCloudComing %}
        呆在家里
    {% else %}
        去外面玩
    {% endif %}

`{％set％}`标签可用于在模板内设置变量。

    {% set activePage = 'blog' %}

标签可以采用许多不同的语法，并列在**标签**部分下。

## 过滤器

过滤器充当单个实例的变量的修饰符，并使用*管道符号*后跟过滤器名称来应用。

    {{ 'string'|filter }}

过滤器可以像函数一样使用参数。

    {{ price|currency('USD') }}

过滤器可以连续应用。

    {{ 'October Glory'|upper|replace({'October': 'Morning'}) }}

过滤器列在**过滤器**部分下。

## 方法

函数允许执行逻辑，返回结果充当变量。

    {{ function() }}

函数可以带参数。

    {{ dump(variable) }}

函数列在**函数**部分下。

## 访问逻辑

关于Twig最重要的一点是它如何访问PHP层。 为方便起见，`{{foo.bar}}`对PHP对象执行以下检查：

1. 检查`foo`是否为数组，`bar`是否为有效元素。
1. 如果没有，如果`foo`是一个对象，检查`bar`是否为有效属性。
1. 如果没有，如果`foo`是一个对象，检查`bar`是一个有效的方法(即使bar是构造函数 - 使用`__construct()`代替)。
1. 如果没有，如果`foo`是一个对象，检查`getBar`是否是一个有效的方法。
1. 如果没有，如果`foo`是一个对象，检查`isBar`是否是一个有效的方法。
1. 如果不是，则返回“null”值。

## 不支持的功能

Twig提供的一些功能在October不受支持。 它们列在等效功能旁边。

标签 | 相等
------------- | -------------
`{% extend %}` | 使用[Layouts](Layouts)`{％placeholder％}
`{% include %}` | 使用 `{% partial %}` 或 `{% content %}`
