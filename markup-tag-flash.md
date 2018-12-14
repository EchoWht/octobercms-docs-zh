# {% flash %}

`{％flash％}`和`{％endflash％}`标签将呈现存储在用户会话中的任何flash消息，由Flash`PHP类设置。 里面的`message`变量将包含flash消息文本，里面的标记将重复多个flash消息。

    <ul>
        {% flash %}
            <li>{{ message }}</li>
        {% endflash %}
    </ul>

您可以使用表示flash消息类型变量; **success成功**，**error错误**，**info信息**或**warning警告**。

    {% flash %}
        <div class="alert alert-{{ type }}">
            {{ message }}
        </div>
    {% endflash %}

您还可以指定`type`来过滤给定类型的Flash消息。 下一个示例将仅显示**success成功**消息，如果有**error错误**消息则不会显示。

    {% flash success %}
        <div class="alert alert-success">{{ message }}</div>
    {% endflash %}

## 设置Flash消息

Flash消息可以通过[Components](cms-components.md) 设置，也可以通过`Flash`类在页面或布局[PHP部分](cms-themes.md#php-section) 中设置。

    <?php

    function onSave()
    {
        // 设置成功的消息
        Flash::success('Settings successfully saved!');

        // 设置错误的消息
        Flash::error('Error saving settings');

        // 设置警告的消息
        Flash::warning('There was a problem but no worries');

        // 设置信息性消息
        Flash::info('Just a heads up about the settings');
    }
