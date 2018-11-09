# |default

如果过滤的值未定义或为空，则`|default`过滤器返回作为第一个参数传递的值，否则返回过滤的值。

    {{ variable|default('The variable is not defined') }}

    {{ variable.foo|default('The foo property on variable is not defined') }}

    {{ variable['foo']|default('The foo key in variable is not defined') }}

    {{ ''|default('The variable is empty')  }}

在某些方法调用中使用变量的表达式上使用`default`过滤器时，每当变量可以未定义时，请务必使用`default`过滤器：

    {{ variable.method(foo|default('bar'))|default('bar') }}
