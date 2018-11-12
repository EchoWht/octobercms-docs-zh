# {% if %}

`{％if％}`和`{％endif％}`标签将表示一个表达式，并且与PHP的if语句相同。 在最简单的形式中，您可以使用它来测试表达式是否计算为“true”：

    {% if online == false %}
        <p>The website is in maintenance mode.</p>
    {% endif %}

您还可以测试数组是否为空：

    {% if users %}
        <ul>
            {% for user in users %}
                <li>{{ user.username }}</li>
            {% endfor %}
        </ul>
    {% endif %}

> **注意**: 如果要测试变量是否已定义，请使用`{％if users is defined％}`。

您还可以使用`not`来判断变量是否为`false`：

    {% if not user.subscribed %}
        <p>You are not subscribed to our mailing list.</p>
    {% endif %}

对于多个表达式，可以使用`{％elseif％}`和`{％else％}`：

    {% if kenny.sick %}
        Kenny is sick.
    {% elseif kenny.dead %}
        You killed Kenny! You bastard!!!
    {% else %}
        Kenny looks okay so far.
    {% endif %}

## 表达规则

确定表达式是真还是假的规则与PHP中的相同，这里是边缘情况规则：

值 | 布尔
------------- | -------------
*空字符串* | false
*数字零* | false
*仅限空白字符串* | true
*空数组* | false
*null* | false
*非空数组* | true
*对象* | true
