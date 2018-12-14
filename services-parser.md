# 解析器服务

- [介绍](#introduction)
- [Markdown解析器](#markdown-parser)
- [Twig模板解析器](#twig-parser)
- [Bracket解析器](#bracket-parser)
- [YAML配置解析器](#yaml-parser)
- [初始化(INI)配置解析器](#ini-parser)
    - [October风格的INI](#october-ini)
- [动态语法解析器](#dynamic-syntax-parser)
    - [查看模式](#syntax-view-mode)
    - [编辑模式](#syntax-editor-mode)
    - [支持的标签](#syntax-supported-tags)

<a name="introduction"></a>
## 介绍

October使用多种标准来处理标记，模板和配置。 每个都经过精心挑选，以尽可能简化您的开发过程和学习曲线。 例如，[主题中找到的对象](../cms/themes) 在其模板结构中使用[Twig](#twig-parser)和[INI格式](#ini-parser) 。 下面更详细地描述每个解析器。

<a name="markdown-parser"></a>
## Markdown解析器

Markdown允许您编写易于阅读且易于编写的纯文本格式，然后转换为HTML。 `Markdown` facade用于解析Markdown语法，基于[GitHub flavored markdown](https://help.github.com/articles/github-flavored-markdown/)。 Markdown的一些简单例子：

    This text is **bold**, this text is *italic*, this text is ~~crossed out~~.

    # The largest heading (an <h1> tag)
    ## The second largest heading (an <h2> tag)
    ...
    ###### The 6th largest heading (an <h6> tag)

使用`Markdown::parse`方法将Markdown渲染为HTML：

    $html = Markdown::parse($markdown);

您也可以使用`| md`过滤器[解析前端标记中的Markdown](../markup/filter-md)。

    {{ '**Text** is bold.'|md }}

<a name="twig-parser"></a>
## Twig模板解析器

Twig是一个简单但功能强大的模板引擎，它将HTML模板解析为优化的PHP代码，它是[前端标记](../markup)，[视图内容](../services/response-view#views)和[邮件消息内容](../services/mail#message-content)。

`Twig` facade用于解析Twig语法，你可以使用`Twig::parse`方法将Twig渲染为HTML。

    $html = Twig::parse($twig);

第二个参数可用于将变量传递给Twig标记。

    $html = Twig::parse($twig, ['foo' => 'bar']);

可以通过[插件注册文件](../plugin/registration#extending-twig).扩展Twig解析器以注册自定义功能。

<a name="bracket-parser"></a>
## Bracket解析器

October还附带了一个简单的括号模板解析器，作为Twig解析器的替代，目前用于将变量传递给[主题内容块](../cms/content#content-variables)。 此引擎可以更快地呈现HTML，并且更适合非技术用户。 这个解析器没有facade，因此完全限定的`October\Rain\Parse\Bracket`类应该与`parse`方法一起使用。

    use October\Rain\Parse\Bracket;

    $html = Bracket::parse($content, ['foo' => 'bar']);

语法使用单数*花括号*来渲染变量：

    <p>Hello there, {foo}</p>

您还可以传递一组对象作为变量进行解析。

    $html = Template::parse($content, ['likes' => [
        ['name' => 'Dogs'],
        ['name' => 'Fishing'],
        ['name' => 'Golf']
    ]]);

可以使用以下语法迭代该数组：

    <ul>
        {likes}
            <li>{name}</li>
        {/likes}
    </ul>

<a name="yaml-parser"></a>
## YAML配置解析器

YAML(“YAML不是标记语言”)是一种配置格式，类似于Markdown，它被设计成易于阅读且易于编写的格式，可转换为PHP数组。 实际上它几乎用于October的后端开发，例如[表单字段](../backend/forms#form-fields)和[list column](../backend/lists##list-columns) 定义。 一些YAML的例子：

    receipt:     Acme Purchase Invoice
    date:        2015-10-02
    user:
        name:    Joe
        surname: Blogs

`Yaml` facade用于解析YAML，你可以使用`Yaml::parse`方法将YAML呈现给PHP数组：

    $array = Yaml::parse($yamlString);

使用`parseFile`方法解析文件的内容：

    $array = Yaml::parseFile($filePath);

解析器还支持反向操作，从PHP数组输出YAML格式。 您可以使用`render`方法：

    $yamlString = Yaml::render($array);

<a name="ini-parser"></a>
## 初始化(INI)配置解析器

INI文件格式是用于定义简单配置文件的标准，通常由[主题模板内的组件](../cms/components)使用。 它可以被认为是YAML格式的堂兄，虽然与YAML不同，它非常简单，对拼写错误不太敏感，也不依赖于缩进。 它支持带有节的基本键值对，例如：

    receipt = "Acme Purchase Invoice"
    date = "2015-10-02"

    [user]
    name = "Joe"
    surname = "Blogs"

`Ini` facade用于解析INI，你使用`Ini::parse`方法将INI渲染到PHP数组：

    $array = Ini::parse($iniString);

使用`parseFile`方法解析文件的内容：

    $array = Ini::parseFile($filePath);

解析器还支持反向操作，从PHP数组输出INI格式。 您可以使用`render`方法：

    $iniString = Ini::render($array);

<a name="october-ini"></a>
### October风格的INI

通常，PHP函数`parse_ini_string`使用的INI解析器仅限于3级深度的数组。 例如：

    level1Value = "foo"
    level1Array[] = "bar"

    [level1Object]
    level2Value = "hello"
    level2Array[] = "world"
    level2Object[level3Value] = "stop here"

October使用*October风格的INI*扩展了此功能，以允许无限深度的数组，受HTML表单语法的启发。 继上面的示例之后，支持以下语法：

    [level1Object]
    level2Object[level3Array][] = "Yay!"
    level2Object[level3Object][level4Value] = "Yay!"
    level2Object[level3Object][level4Array][] = "Yay!"
    level2Object[level3Object][level4Object][level5Value] = "Yay!"
    ; ... 超越无限！

<a name="dynamic-syntax-parser"></a>
## 动态语法解析器

动态语法是October独有的模板引擎，从根本上支持两种渲染模式。 解析模板将产生两个结果，**view**或**edit**模式。 以此模板文本为例，`{text} ... {/ text}`标签的内部部分代表**view**模式的默认文本，而内部属性，`name`和`label `，用作**edit**模式的属性。

    <h1>{text name="websiteName" label="Website Name"}Our wonderful website{/text}</h1>

此解析器没有facade，因此完全限定的`October\Rain\Parse\Syntax\Parser`类应与`parse`方法一起使用。 `parse`方法的第一个参数将模板内容作为字符串并返回一个`Parser`对象。

    use October\Rain\Parse\Syntax\Parser as SyntaxParser;

    $syntax = SyntaxParser::parse($content);

<a name="syntax-view-mode"></a>
### 查看模式

假设我们使用上面的第一个示例作为模板内容，调用`render`方法本身将使用默认文本呈现模板：

    echo $syntax->render();
    // <h1>Our wonderful website</h1>

就像任何模板引擎一样，将变量数组传递给`render`的第一个参数将替换模板中的变量。 这里`websiteName`的默认值被替换为我们的新值：

    echo $syntax->render(['websiteName' => 'OctoberCMS']);
    // <h1>OctoberCMS</h1>

作为一个额外的功能，调用`toTwig`方法将输出准备状态的模板，以便由[Twig引擎](#twig-parser).进行渲染。

    echo $syntax->toTwig();
    // <h1>{{ websiteName }}</h1>

<a name="syntax-editor-mode"></a>
### 编辑模式

到目前为止，动态语法解析器与常规模板引擎没有太大区别，但编辑器模式是动态语法的实用程序变得更加明显的地方。 编辑器模式解锁了一个新的可能性领域，例如，[布局将自定义表单字段注入页面](http://octobercms.com/plugin/rainlab-pages)属于它们或[动态构建的表单用于电子邮件](http://octobercms.com/plugin/responsiv-campaign).。

继续上面的示例，在`Parser`对象上调用`toEditor`方法将返回一个PHP数组属性，这些属性定义如何填充变量，例如由表单构建器。

    $array = $syntax->toEditor();
    // 'websiteName' => [
    //     'label' => 'Website name',
    //     'default' => 'Our wonderful website',
    //     'type' => 'text'
    // ]

您可能会注意到这些属性与 [表格字段定义](../backend/forms#form-fields)中的选项非常相似。 这是故意的，因此这两个特征相互补充。 我们现在可以轻松地将上面的数组转换为YAML并写入`fields.yaml`文件：

    $form = [
        'fields' => $syntax->toEditor()
    ];

    File::put('fields.yaml', Yaml::render($form));

<a name="syntax-supported-tags"></a>
### 支持的标签

动态语法分析器可以使用各种标记类型，这些标记类型旨在匹配常见的[表单字段类型](../backend/forms#field-types)。

#### Text

单行输入，用于较小的文本块。

    {text name="websiteName" label="Website Name"}Our wonderful website{/text}

#### Textarea

多行输入，用于较大的文本块。

    {textarea name="websiteDescription" label="Website Description"}
        This is our vision for things to come
    {/textarea}

### Dropdown

呈现下拉表单字段。

    {dropdown name="dropdown" label="Pick one" options="One|Two"}{/dropdown}

### Radio

呈现表单单选字段。

    {radio name="radio" label="Thoughts?" options="y:Yes|n:No|m:Maybe"}{/radio}

#### Variable

完全按照`type`属性中的定义呈现表单字段类型。 此标记将只设置一个变量，并在视图模式下呈现为空字符串。

    {variable type="text" name="name" label="Name"}John{/variable}

#### Rich editor

富内容的文本输入(WYSIWYG)。

    {richeditor name="content" label="Main content"}Default text{/richeditor}

以Twig渲染为

    {{ content|raw }}

#### Markdown

Markdown内容的文本输入。

    {markdown name="content" label="Markdown content"}Default text{/markdown}

以Twig渲染为

    {{ content|md }}

<!--
#### Checkbox

在内部呈现条件内容(仍在开发中)

    {checkbox name="showHeader" label="Show heading" default="true"}
        <p>This content will be shown if the checkbox is ticked</p>
    {/checkbox}

以Twig渲染为

    {% if checkbox %}
        {{ showHeader }}
    {% endif %}
-->

#### Media finder

媒体库项目的文件选择器。 此标记值将包含文件的相对路径。

    {mediafinder name="logo" label="Logo"}defaultlogo.png{/mediafinder}

以Twig渲染为

    {{ logo|media }}

#### File upload

文件的文件上传输入。 此标记值将包含文件的完整路径。

    {fileupload name="logo" label="Logo"}defaultlogo.png{/fileupload}

#### Repeater

渲染一个包含其他字段的重复部分。

    {repeater name="content_sections" prompt="Add another content section"}
        <h2>{text name="title" label="Title"}Title{/text}</h2>
        <p>{textarea name="content" label="Content"}Content{/textarea}</p>
    {/repeater}

以Twig渲染为

    {% for fields in repeater %}
        <h2>{{ fields.title }}</h2>
        <p>{{ fields.content|raw }}</p>
    {% endfor %}

调用`$syntax-> toEditor`将为转发器字段返回一个不同的数组：

    'repeater' => [
        'label' => 'Website name',
        'type' => 'repeater',
        'fields' => [

            'title' => [
                'label' => 'Title',
                'default' => 'Title',
                'type' => 'text'
            ],
            'content' => [
                'label' => 'Content',
                'default' => 'Content',
                'type' => 'textarea'
            ]

        ]
    ]
    
转发器字段还支持组模式，与动态语法分析器一起使用，如下所示：

    {variable name="sections" type="repeater" prompt="Add another section" tab="Sections" 
        groups="$/author/plugin/repeater_fields.yaml"}{/variable}

这是repeater_fields.yaml组配置文件的示例：

    quote:
        name: Quote
        description: Quote item
        icon: icon-quote-right
        fields:
            quote_position:
                span: auto
                label: Quote Position
                type: radio
                options:
                    left: Left
                    center: Center
                    right: Right
            quote_content:
                span: auto
                label: Details
                type: textarea
                
有关转发器组模式的更多信息，请参阅[Repeater Widget](../backend/forms#widget-repeater)。       
