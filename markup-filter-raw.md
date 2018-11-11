# |raw

October的输出变量会自动转义，`|raw`过滤器会将值标记为“安全”，如果`raw`是最后应用的过滤器，则不会转义。

    {# 此变量不会被转义 #}
    {{ variable|raw }}

在表达式中使用`raw`过滤器时要小心：

    {% set hello = '<strong>Hello</strong>' %}
    {% set hola = '<strong>Hola</strong>' %}

    {{ false ? '<strong>Hola</strong>' : hello|raw }}

    {# 以上内容不会与之相同 #}
    {{ false ? hola : hello|raw }}

    {# 但渲染相同 #}
    {{ (false ? hola : hello)|raw }}
