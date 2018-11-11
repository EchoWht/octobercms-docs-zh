# str()

以`str_`为前缀的函数执行在处理字符串时有用的任务。 帮助程序直接映射到`Str` PHP类及其方法。 例如：

    {{ str_camel() }}

对应的PHP的方法如下：

    <?= Str::camel() ?>

> **注意**: *camelCase驼峰命名*中的方法应转换为*snake_case下划线命名*。

## str_limit()

限制字符串中的字符数。

    {{ str_limit('The quick brown fox...', 100) }}

要在应用限制时添加后缀，请将其作为第三个参数传递。 默认为“...”。

    {{ str_limit('The quick brown fox...', 100, '... Read more!') }}

## str_words()

限制字符串中的单词数。

    {{ str_words('The quick brown fox...', 100) }}

要在应用限制时添加后缀，请将其作为第三个参数传递。 默认为“...”。

    {{ str_words('The quick brown fox...', 100, '... Read more!') }}

## str_camel()

将值转换为*camelCase*。

    // 输出: helloWorld
    {{ str_camel('hello world') }}

## str_studly()

将值转换为*StudlyCase*。

    // 输出: HelloWorld
    {{ str_studly('hello world') }}

## str_snake()

将值转换为*snake_case*。

    // 输出: hello_world
    {{ str_snake('hello world') }}

第二个参数可以提供分隔符。

    // 输出: hello---world
    {{ str_snake('hello world', '---') }}

## str_plural()

获取复数形式的英文单词。

    // 输出: chickens
    {{ str_plural('chicken') }}
