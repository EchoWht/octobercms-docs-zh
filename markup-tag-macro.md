# {% macro %}

`{％macro％}`标签允许您在模板中定义自定义函数，类似于常规编程语言。

    {% macro input() %}
        ...
    {% endmacro %}

或者，您可以在结束标记之后包含标签的名称，以提高可读性：

    {% macro input() %}
        ...
    {% endmacro input %}

下面的例子定义了一个名为`input()`的函数，它接受4个参数，相关的值作为标记内部的变量进行访问。

    {% macro input(name, value, type, size) %}
        <input
            type="{{ type|default('text') }}"
            name="{{ name }}"
            value="{{ value|e }}"
            size="{{ size|default(20) }}" />
    {% endmacro %}

> **注意**: 宏参数不指定默认值，并且始终被视为可选。

<a name="calling-macros"></a>
## 调用宏

在使用宏之前，需要先使用`{％import％}`标签“导入”它。 如果宏在同一模板中定义，则可以使用特殊的`_self`变量。

    {% import _self as form %}

这里宏函数被赋值给`form`变量，可以像任何其他函数一样被调用。

    <p>{{ form.input('username') }}</p>
    <p>{{ form.input('password', null, 'password') }}</p>

宏可以在[主题部分](cms-partials.md)中定义并按名称导入。 要从名为$macros/form.htm **的部分导入宏，只需在引用为字符串的`import`标记后传递名称即可。

    {% import 'macros/form' as form %}

或者，您可以从[系统视图文件](services-response-view.md#views)导入宏，这些宏将被接受。 要从$plugins/acme/blog/views/macros.htm导入**，只需传递路径提示即可。

    {% import 'acme.blog::macros' as form %}

<a name="nested-macros"></a>
## 嵌套宏

如果要在同一模板中的另一个宏中使用宏，则需要在本地导入它。

    {% macro input(name, value, type, size) %}
        <input
            type="{{ type|default('text') }}"
            name="{{ name }}"
            value="{{ value|e }}"
            size="{{ size|default(20) }}" />
    {% endmacro %}

    {% macro wrapped_input(name, value, type, size) %}
        {% import _self as form %}

        <div class="field">
            {{ form.input(name, value, type, size) }}
        </div>
    {% endmacro %}

<a name="context-variable"></a>
## 上下文变量

宏无权访问当前页面变量。

    <!-- October CMS -->
    {{ site_name }} 

    {% macro myFunction() %}
        <!-- NULL -->
        {{ site_name }}
    {% endmacro %}

您可以使用特殊的`_context`变量将变量传递给函数。

    {% macro myFunction(vars) %}
        {{ vars.site_name }}
    {% endmacro %}

    {% import _self as form %}

    <!-- October CMS -->
    {{ form.myFunction(_context) }}
