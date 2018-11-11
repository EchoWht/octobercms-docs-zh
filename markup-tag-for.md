# {% for %}

`{％for％}`和`{％endfor％}`标签将遍历集合中的每个值。 集合可以是实现`Traversable`接口的数组或对象。

    <ul>
        {% for user in users %}
            <li>{{ user.username }}</li>
        {% endfor %}
    </ul>

您还可以访问键和值：

    <ul>
        {% for key, user in users %}
            <li>{{ key }}: {{ user.username }}</li>
        {% endfor %}
    </ul>

如果集合为空，则可以使用else渲染替换块(居然还有这骚操作)：

    <ul>
        {% for user in users %}
            <li>{{ user.username }}</li>
        {% else %}
            <li><em>There are no users found</em></li>
        {% endfor %}
    </ul>

## 循环集合

如果你确实需要迭代一组数字，你可以使用`..`运算符：

    {% for i in 0..10 %}
        - {{ i }}
    {% endfor %}

上面的代码片段将打印0到10之间的所有数字。

它也适用于字母：

    {% for letter in 'a'..'z' %}
        - {{ letter }}
    {% endfor %}

`··`运算符可以在两边表达任何表达式：

    {% for letter in 'a'|upper..'z'|upper %}
        - {{ letter }}
    {% endfor %}

## 添加条件

与PHP不同，在循环中没有“break”或“continue”的功能，但是你仍然可以过滤集合。 以下示例跳过所有未激活的`users`：

    <ul>
        {% for user in users if user.active %}
            <li>{{ user.username }}</li>
        {% endfor %}
    </ul>

## 循环变量

在`for`循环块中，您可以访问一些特殊变量：

变量 | 描述
------------- | -------------
`loop.index` | 循环的当前迭代。 （1索引）
`loop.index0` | 循环的当前迭代。 （0索引）
`loop.revindex` |  循环结束时的迭代次数（1个索引）
`loop.revindex0` | 循环结束时的迭代次数（0索引）
`loop.first` | 如果是第一次迭代，则为True
`loop.last` |  如果是最后一次迭代，则为True
`loop.length` | 集合的长度
`loop.parent` | 父上下文

    {% for user in users %}
        {{ loop.index }} - {{ user.username }}
    {% endfor %}
