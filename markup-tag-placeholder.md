# {% placeholder %}

`{%placeholder%}`标签将呈现一个占位符部分，通常[在Layouts中使用](cms-layouts.md#placeholders)。 此标记将返回使用`{%put%}`标记添加的任何占位符内容，或任何已定义的默认内容(可选)。
    
    {% placeholder name %}

然后可以在任何后续页面或部分页面中将内容注入占位符。

    {% put name %}
        <p>Place this text in the name placeholder</p>
    {% endput %}

<a name="default-placeholder-content"></a>
## 默认占位符内容

占位符可以具有可以由页面替换或补充的默认内容。 如果未在页面上定义具有默认内容的占位符的`{%put%}`标记，则会显示默认占位符内容。 布局模板中的占位符定义示例：

    {% placeholder sidebar default %}
        <p><a href="/contacts">Contact us</a></p>
    {% endplaceholder %}

该页面可以向占位符注入更多内容。 `{%default%}`标记指定应显示默认占位符内容的位置。 如果未使用标记，则占位符内容将被完全替换。

    {% put sidebar %}
        <p><a href="/services">Services</a></p>
        {% default %}
    {% endput %}

<a name="checking-placeholder-exits"></a>
## 检查占位符是否存在

在布局模板中，您可以使用`placeholder()`函数检查占位符内容是否存在。 这使您可以根据页面是否提供占位符内容生成不同的标记。 例：

    {% if placeholder('sidemenu') %}
        <!-- Markup for a page with a sidebar -->
        <div class="row">
            <div class="col-md-3">
                {% placeholder sidemenu %}
            </div>
            <div class="col-md-9">
                {% page %}
            </div>
        </div>
    {% else %}
        <!-- Markup for a page without a sidebar -->
        {% page %}
    {% endif %}

<a name="custom-placeholder-attributes"></a>
## 自定义属性

`placeholder`标签接受两个可选属性 ＆mdash; `title`和`type`。 CMS本身不使用`title`属性，但可以被其他插件使用。 type属性管理占位符类型。 目前支持两种类型＆mdash; **text**和**html**。 文本占位符的内容在显示之前进行转义。 标题和类型属性应在占位符名称和`default`属性之后定义(如果已显示)。 例：

    {% placeholder ordering title="Ordering information" type="text" %}

具有默认内容，标题和类型属性的占位符示例。

    {% placeholder ordering default title="Ordering information" type="text" %}
        There is no ordering information for this product.
    {% endplaceholder %}
