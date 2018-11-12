# this.environment

您可以通过`this.environment`访问当前环境对象，它返回一个引用[当前环境配置](../setup/configuration#environment-config)的字符串。

## 例如

如果网站在测试环境中运行，以下示例将显示横幅：

    {% if this.environment == 'test' %}

        <div class="banner">Test Environment</div>

    {% endif %}
