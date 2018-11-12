# {% verbatim %}

`{％verbatim％}`标签将整个部分标记为不应解析的原始文本。

    {% verbatim %}<p>Hello, {{ name }}</p>{% endverbatim %}

以上内容将在浏览器中呈现如下：

    <p>Hello, {{ name }}</p>

例如，AngularJS使用相同的模板语法，因此您可以决定为每个变量使用哪些变量。

    <p>Hello {{ name }}, this is parsed by Twig</p>

    {% verbatim %}
        <p>Hello {{ name }}, this is parsed by AngularJS</p>
    {% endverbatim %}
