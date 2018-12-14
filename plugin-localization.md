# 本地化

- [本地化目录和文件结构](#file-structure)
- [访问本地化字符串](#accessing-strings)
- [覆盖本地化字符串](#overriding)

插件可以在插件目录的**lang**子目录中具有本地化文件。 插件的本地化文件会自动注册。 自动支持本地化字符串在后端用户界面菜单，表单标签等中。 - 如果您提供本地化密钥而不是真实字符串，系统将尝试从本地化文件加载它。 在其他情况下，您需要加载[使用API](#accessing-strings)的本地化字符串。

> **注意**: 为了翻译前端内容，[可以使用的这个插件](http://octobercms.com/plugin/rainlab-translate)。

<a name="file-structure"></a>
## 本地化目录和文件结构

下面是插件的lang目录的示例：

    plugins/
      acme/
        todo/             <=== Plugin directory
          lang/           <=== Localization directory
            en/           <=== Language directory
              lang.php    <=== Localization file
            fr/
              lang.php


**lang.php**文件应该定义并返回数组，例如：

    <?php

    return [
        'app' => [
            'name' => 'OctoberCMS',
            'tagline' => 'Getting back to basics'
        ]
    ];
    
**validation.php**文件具有与**lang.php**类似的结构，用于指定[自定义验证](https://octobercms.com/docs/services/validation#localization) 消息 在语言文件中，例如：

    <?php

    return [
        'required' => 'We need to know your xxx!',
        'email.required' => 'We need to know your e-mail address!',
    ];  

<a name="accessing-strings"></a>
## 访问本地化字符串

可以使用`Lang`类加载本地化字符串。 它接受的参数是本地化键字符串，它由插件名称，本地化文件名和从文件返回的数组内的本地化字符串的路径组成。 下一个示例从*plugins/acme/blog/lang/en/lang.php*文件加载**app.name**字符串(该语言在`config/app.php`配置中使用`locale`参数设置 文件)：

    echo Lang::get('acme.blog::lang.app.name');

<a name="overriding"></a>
## 覆盖本地化字符串

系统用户可以覆盖插件本地化字符串而无需更改插件的文件。 这是通过将本地化文件添加到**lang**目录来完成的。 例如，要覆盖**acme/blog**插件的lang.php文件，您应该在以下位置创建该文件：

    lang/               <=== App localization directory
      en/               <=== Language directory
        acme/           <=== Plugin/Module directory
          blog/         <===^
            lang.php    <=== Localization override file

该文件只需包含您要覆盖的字符串，无需替换整个文件。 例：

    <?php

    return [
        'app' => [
            'name' => 'OctoberCMS!'
        ]
    ];
