# this.param

您可以通过`this.param`访问当前的URL参数，它返回一个PHP数组。

## 访问页面参数

此示例演示如何访问页面中的“tab”URL参数。

    url = "/account/:tab"
    ==
    {% if this.param.tab == 'details' %}

        <p>Here are all your details</p>

    {% elseif this.param.tab == 'history' %}

        <p>You are viewing a blast from the past</p>

    {% endif %}

如果参数名称也是变量，则可以使用数组语法。

    url = "/account/:post_id"
    ==
    {% set name = 'post_id' %}

    <p>The post ID is: {{ this.param[name] }}</p>
