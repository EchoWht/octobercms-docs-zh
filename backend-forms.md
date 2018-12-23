# 后端表单

- [介绍](#introduction)
- [配置表单行为](#configuring-form)
    - [新建页面](#form-create-page)
    - [更新页面](#form-update-page)
    - [预览页面](#form-preview-page)
- [定义表单字段](#form-fields)
    - [标签选项](#form-tab-options)
    - [字段选项](#form-field-options)
- [可用的字段类型](#field-types)
- [表单小部件](#form-widgets)
- [表单视图](#form-views)
    - [新建页面](#form-create-view)
    - [更新页面](#form-update-view)
    - [预览页面](#form-preview-view)
- [将条件应用于字段](#field-conditions)
    - [输入预设转换器](#field-input-preset)
    - [触发事件](#field-trigger-events)
    - [字段依赖性](#field-dependencies)
    - [防止字段提交](#prevent-field-submission)
- [扩展表单行为](#extend-form-behavior)
    - [覆盖控制器action](#overriding-action)
    - [扩展表单模型查询](#extend-model-query)
    - [扩展表单字段](#extend-form-fields)
    - [过滤表单字段](#filter-form-fields)
- [验证表单字段](#validate-form-fields)

<a name="introduction"></a>
## 介绍

**表单行为** 是一个控制器修饰符，用于轻松地将表单功能添加到后端页面。 该行为提供了三个名为Create，Update和Preview的页面。 “预览”页面是“更新”页面的只读版本。 当您使用表单行为时，您不需要在控制器中定义`create`，`update`和`preview`操作 - 行为就是为您完成的。 但是，您应该提供相应的视图文件。

表单行为取决于表单[字段定义](#form-fields)和[模型类](database-model.md)。 为了使用表单行为，您应该将它添加到控制器类的`$implement`属性中。 此外，应定义`$formConfig`类属性，其值应引用用于配置行为选项的YAML文件。

    namespace Acme\Blog\Controllers;

    class Categories extends \Backend\Classes\Controller
    {
        public $implement = ['Backend.Behaviors.FormController'];

        public $formConfig = 'form_config.yaml';
    }

> **注意:** 表单和[列表行为](backend-lists.md)通常在同一个控制器中一起使用。

<a name="configuring-form"></a>
## 配置表单行为

`$formConfig`属性中引用的配置文件以YAML格式定义。 该文件应放入控制器的[views目录](controllers-views-ajax/#introduction)。 以下是典型表单行为配置文件的示例：

    # ===================================
    #  Form Behavior Config
    # ===================================

    name: Blog Category
    form: $/acme/blog/models/post/fields.yaml
    modelClass: Acme\Blog\Post

    create:
        title: New Blog Post

    update:
        title: Edit Blog Post

    preview:
        title: View Blog Post

The following fields are required in the form configuration file:

字段 | 描述
------------- | -------------
**name** | 此表单管理的对象的名称。
**form** | 配置数组或对表单字段定义文件的引用，请参阅[表单字段](#form-fields)
**modelClass** | 模型类名称，表单数据将加载并保存在此模型中。

下面列出的配置选项是可选的。 如果希望表单行为支持[Create](#form-create-page), [Update](#form-update-page) 或 [Preview](#form-preview-page)页面，请定义它们。

选项 | 描述
------------- | -------------
**defaultRedirect** | 在未定义特定重定向页面时用作回退重定向页面。
**create** | 配置数组或对“新建”页面的配置文件的引用。
**update** | 配置数组或对“更新”页面的配置文件的引用。
**preview** | 配置数组或对预览页面的配置文件的引用。

<a name="form-create-page"></a>
### 新建页面

要支持“新建”页面，请将以下配置添加到YAML文件中：

    create:
        title: New Blog Post
        redirect: acme/blog/posts/update/:id
        redirectClose: acme/blog/posts
        flashSave: Post has been created!

“创建”页面支持以下配置选项：

选项 | 描述
------------- | -------------
**title** | 页面标题，可以参考[本地化字符串](plugin-localization.md).。
**redirect** | 保存记录时的重定向页面。
**redirectClose** | 保存记录时的重定向页面，并随请求一起发送**close** post变量。
**flashSave** | 保存记录时显示的flash消息，可以参考[本地化字符串](plugin-localization.md)。
**form** | 仅覆盖创建页面的默认表单字段定义。

<a name="form-update-page"></a>
### 更新页面

要支持“更新”页面，请将以下配置添加到YAML文件中：

    update:
        title:编辑文章
        redirect:acme/blog/posts
        flashSave:保存成功
        flashDelete:删除成功

“更新”页面支持以下配置选项：

选项 | 描述
------------- | -------------
**title** | 页面标题，可以参考[本地化字符串](../插件/本地化)。
**redirect** | 保存记录时的重定向页面。
**redirectClose** | 保存记录时的重定向页面和请求一起发送**close** post变量。
**flashSave** | 保存记录时显示的flash消息，可以参考[本地化字符串](../插件/本地化)。
**flashDelete** | 删除记录时显示的flash消息，可以参考[本地化字符串](../插件/本地化)。
**form** | 仅覆盖更新页面的默认表单字段定义。

<a name="form-preview-page"></a>
### 预览页面

要支持“预览”页面，请将以下配置添加到YAML文件中：

    preview:
        title:博客详情页

“预览”页面支持以下配置选项：

选项  | 描述
------------- | -------------
**title** | 页面标题，可以参考[本地化字符串](../插件/本地化)。
**form** | 仅覆盖预览页面的默认表单字段定义。

<a name="form-fields"></a>
## 定义表单字段

表单字段使用YAML文件定义。 表单字段配置由表单行为用于创建表单控件并将它们绑定到模型字段。 该文件放在插件的**models**目录的子目录中。 子目录名称与以小写字母书写的模型类名称匹配。 文件名无关紧要，但**fields.yaml**和**form_fields.yaml**是常用名称。 示例表单字段文件位置：

    plugins/
      acme/
        blog/
          models/        <=== Plugin models directory
            post/        <=== Model configuration directory
              fields.yaml    <=== Model form fields config file
            Post.php         <=== model class

字段可以放在三个区域，**外部区域**，**主要标签**或**次要标签**。 下一个示例显示了表单字段定义文件的典型内容。

    # ===================================
    #  Form Field Definitions
    # ===================================

    fields:
        blog_title:
            label: Blog Title
            description: The title for this blog

        published_at:
            label: Published date
            description: When this blog post was published
            type: datepicker

        [...]

    tabs:
        fields:
            [...]

    secondaryTabs:
        fields:
            [...]

可以使用[Relation Widget](#widget-relation)或[Relation Manager](backend-relations.md#relationship-types)呈现相关模型中的字段。 例外是OneToOne或morphOne相关字段，必须定义为**relation[field]**，然后可以指定为模型的任何其他字段：

        user_name:
            label: User Name
            description: The name of the user
        avatar[name]:
            label: Avatar
            description: will be saved in the Avatar table
        published_at:
            label: Published date
            description: When this blog post was published
            type: datepicker

        [...]

<a name="form-tab-options"></a>
### 标签选项

对于每个选项卡定义，即`tabs`和`secondaryTabs`，您可以指定以下选项：

选项 | 描述
------------- | -------------
**stretch** | 指定此选项卡是否伸展以适合父高度。
**defaultTab** | 分配字段的默认选项卡。 默认值：杂项。
**cssClass** | 将一个CSS类分配给选项卡容器。
**paneCssClass** | 将CSS类分配给单个选项卡窗格。 值是数组，键是制表符索引或标签，值是CSS类。

<a name="form-field-options"></a>
### 字段选项

对于每个字段，您可以指定这些选项(如果适用)：

选项 | 描述
------------- | -------------
**label** | 向用户显示表单字段时的名称。
**type** | 定义应该如何呈现此字段(请参阅下面的[可用字段类型](#field-types) )。 默认值：文字。
**span** | 将表单字段对齐到一侧。 选项：auto, left, right, full。 默认值：full。
**size** | 指定使用它的字段的字段大小，例如textarea字段. Options: tiny, small, large, huge, giant.
**placeholder** | 如果字段支持占位符值。
**comment** | 在字段下方放置描述性注释。
**commentAbove** | 在该字段上方显示评论。
**commentHtml** | 允许的HTML标记内的comment。选项：true，false。
**default** | 指定字段的默认值。
**defaultFrom** | 从另一个字段的值中获取默认值。
**tab** | 将字段分配给选项卡。
**cssClass** | 向字段容器分配CSS类。
**readOnly** | 防止字段被修改。选项： true, false.
**disabled** | 防止字段被修改，并将其从保存的数据中排除。选项：true, false.
**hidden** | 从视图隐藏字段，并将其从保存的数据中排除。选项： true, false.
**stretch** | 指定此字段是否延伸到适合父级的高度。
**context** | 什么时候应该使用上下文应该提醒显示场。上下文可以通过使用符号`@` 在“字段名称”名称，例如，`name@update`。
**dependsOn** | 其他字段名的数组[依赖于](#field-dependencies)，当修改其他字段时，该字段将更新。
**trigger** | 使用[触发器事件](#field-trigger-events).指定该字段的条件。
**preset** | 允许字段值最初由另一个字段的值设置，使用[输入预置转换器](#field-input-preset)进行转换。
**required** | 在字段标签旁边放置一个红色的星号以指示它是必需的(请确保在模型上设置验证，因为这不是由表单控制器执行的)。
**attributes** | 指定要添加到表单字段元素的自定义HTML属性。
**containerAttributes** | 指定要添加到表单字段容器的自定义HTML属性。

<a name="field-types"></a>
## 可用的字段类型

有各种本机字段类型可用于**类型**设置。 对于更高级的表单字段，可以使用[表单窗口小部件](#form-widgets)。
1. [Text](#field-text)
1. [Number](#field-number)
1. [Password](#field-password)
1. [Textarea](#field-textarea)
1. [Dropdown](#field-dropdown)
1. [Radio List](#field-radio)
1. [Balloon Selector](#field-balloon)
1. [Checkbox](#field-checkbox)
1. [Checkbox List](#field-checkboxlist)
1. [Switch](#field-switch)
1. [Section](#field-section)
1. [Partial](#field-partial)
1. [Hint](#field-hint)
1. [Widget](#field-widget)

<a name="field-text"></a>
### Text

`text` - 呈现单行文本框。 如果未指定，则使用此默认类型。

    blog_title:
        label: Blog Title
        type: text

<a name="field-number"></a>
### Number

`number` - 呈现仅包含数字的单行文本框。

    your_age:
        label: Your Age
        type: number

<a name="field-password"></a>
### Password

`password ` - 呈现单行密码字段。

    user_password:
        label: Password
        type: password

<a name="field-textarea"></a>
### Textarea

`textarea` - 呈现多行文本框。 尺寸也可以用可能的值指定： tiny, small, large, huge, giant.

    blog_contents:
        label: Contents
        type: textarea
        size: large

<a name="field-dropdown"></a>
### Dropdown

`dropdown` - 使用指定选项呈现下拉列表。 有4种方法可以提供下拉选项。 第一个方法直接在YAML文件中定义`options`：

    status_type:
        label: Blog Post Status
        type: dropdown
        options:
            draft: Draft
            published: Published
            archived: Archived

第二种方法使用模型类中声明的方法定义选项。 如果省略options元素，框架需要在模型中定义名为 get*FieldName*Options 的方法。 使用上面的示例，模型应该具有`getStatusTypeOptions`方法。 此方法的第一个参数是此字段的当前值，第二个参数是整个表单的当前数据对象。 此方法应返回**key=>label**格式的选项数组。

    status_type:
        label: Blog Post Status
        type: dropdown

提供模型类中的下拉选项：

    public function getStatusTypeOptions($value, $formData)
    {
        return ['all' => 'All', ...];
    }

第三个全局方法`getDropdownOptions`也可以在模型中定义，这将用于模型的所有下拉字段类型。 此方法的第一个参数是字段名称，第二个参数是字段的当前值，第三个参数是整个表单的当前数据对象。 它应该以**key=>label**格式返回一个选项数组。

    public function getDropdownOptions($fieldName, $value, $formData)
    {
        if ($fieldName == 'status') {
            return ['all' => 'All', ...];
        }
        else {
            return ['' => '-- none --'];
        }
    }

第四种方法使用模型类中声明的特定方法。 在下一个示例中，应该在模型类中定义`listStatuses`方法。 此方法接收与`getDropdownOptions`方法相同的所有参数，并应返回**key=>label**格式的选项数组。

    status:
        label: Blog Post Status
        type: dropdown
        options: listStatuses

将下拉选项提供给模型类：

    public function listStatuses($fieldName, $value, $formData)
    {
        return ['published' => 'Published', ...];
    }

要在没有选择时定义行为，可以指定`emptyOption`值以包含可以重新选择的空选项。

    status:
        label: Blog Post Status
        type: dropdown
        emptyOption: -- no status --

或者，您可以使用`placeholder`选项来使用无法重新选择的“单向”空选项。

    status:
        label: Blog Post Status
        type: dropdown
        placeholder: -- select a status --

默认情况下，下拉列表具有搜索功能，允许快速选择值。 可以通过将`showSearch`选项设置为`false`来禁用它。

    status:
        label: Blog Post Status
        type: dropdown
        showSearch: false

<a name="field-radio"></a>
### Radio List

`radio` - 呈现单选按钮列表，其中一次只能选择一个项目。

    security_level:
        label: Access Level
        type: radio
        options:
            all: All
            registered: Registered only
            guests: Guests only

单选按钮列表还可以支持次要描述。

    security_level:
        label: Access Level
        type: radio
        options:
            all: [All, Guests and customers will be able to access this page.]
            registered: [Registered only, Only logged in member will be able to access this page.]
            guests: [Guests only, Only guest users will be able to access this page.]

无线电列表支持三种定义选项的方式，与[下拉字段类型](#field-dropdown)完全相同。 对于无线电列表，该方法可以返回简单数组：**key=>value**或用于提供描述的数组数组：**key=>[label,description]**

<a name="field-balloon"></a>
### Balloon Selector

`balloon-selector` - 呈现一个列表，其中一次只能选择一个项目。

    gender:
        label: Gender
        type: balloon-selector
        options:
            female: Female
            male: Male

气球选择器支持三种定义选项的方式，与[下拉字段类型](#field-dropdown)完全相同。

<a name="field-checkbox"></a>
### Checkbox

`checkbox` - 呈现单个复选框。

    show_content:
        label: Display content
        type: checkbox
        default: true

<a name="field-checkboxlist"></a>
### Checkbox List

`checkboxlist` - 呈现复选框列表。

    permissions:
        label: Permissions
        type: checkboxlist
        options:
            open_account: Open account
            close_account: Close account
            modify_account: Modify account

复选框列表支持三种定义选项的方法，与[下拉字段类型](#field-dropdown)完全相同，并且还支持[单选字段类型](#field-radio)中的辅助描述。

<a name="field-switch"></a>
### Switch

`switch` - 呈现一个开关按钮。

    show_content:
        label: Display content
        type: switch
        comment: Flick this switch to display content
        on: myauthor.myplugin::lang.models.mymodel.show_content.on
        off: myauthor.myplugin::lang.models.mymodel.show_content.off

<a name="field-section"></a>
### Section

`section` - 呈现节标题和子标题。 `label`和`comment`值是可选的，包含标题和子标题的内容。

    user_details_section:
        label: User details
        type: section
        comment: This section contains details about the user.

<a name="field-partial"></a>
### Partial

`partial` - 呈现partial，`path`值可以引用部分视图文件，否则字段名称用作partial名称。 在partial内部，这些变量是可用的：`$value`是默认字段值，`$model`是用于字段的模型，`$field`是配置的类对象`Backend\Classes\FormField`。

    content:
        type: partial
        path: $/acme/blog/models/comments/_content_field.htm

<a name="field-hint"></a>
### Hint

`hint` - 与`partial`字段相同，但在一个可以被用户隐藏的提示容器内呈现。

    content:
        type: hint
        path: content_field

<a name="field-widget"></a>
### Widget

`widget` - 呈现自定义表单窗口小部件，`type`字段可以直接引用窗口小部件的类名或注册的别名。

    blog_content:
        type: Backend\FormWidgets\RichEditor
        size: huge

<a name="form-widgets"></a>
## 表单小部件

标准中包含各种表单小部件，但插件通常提供自己的自定义表单小部件。 您可以在[Form Widgets](backend-widgets.md#form-widgets)文章上阅读更多内容。

1. [Code editor](#widget-codeeditor)
1. [Color picker](#widget-colorpicker)
1. [Date picker](#widget-datepicker)
1. [File upload](#widget-fileupload)
1. [Record finder](#widget-recordfinder)
1. [Media finder](#widget-mediafinder)
1. [Relation](#widget-relation)
1. [Repeater](#widget-repeater)
1. [Rich editor/WYSIWYG](#widget-richeditor)
1. [Markdown editor](#widget-markdowneditor)
1. [Tag list](#widget-taglist)

<a name="widget-codeeditor"></a>
### Code editor

`codeeditor` - 为格式化代码或标记呈现纯文本编辑器。 请注意，可以通过在后端为管理员定义的代码编辑器首选项继承选项。

    css_content:
        type: codeeditor
        size: huge
        language: html

选项 | 描述
------------- | -------------
**language** | 代码语言，例如，php，css，js，html。 默认值：php。
**showGutter** | 显示带行号的Gutter。 默认值：true。
**wrapWords** | 自动换行。 默认为true。
**fontSize** | 文字字体大小。 默认值：12。

<a name="widget-colorpicker"></a>
### Color picker
`colorpicker` - 渲染控件以选择十六进制颜色值。

    color:
        label: Background
        type: colorpicker

选项 | 描述
------------- | -------------
**availableColors** |  可用颜色列表。

<a name="widget-datepicker"></a>
### Date picker

`datepicker` - 呈现用于选择日期和时间的文本字段。

    published_at:
        label: Published
        type: datepicker
        mode: date

选项 | 选项
------------- | -------------
**mode** | 预期结果，date, datetime或time。 默认值：datetime。
**format** |  提供明确的日期显示格式。 例如：Y-m-d
**minDate** | 可以选择的最短/最早日期。 默认值：2000-01-01。
**maxDate** | 可以选择的最长/最晚日期。 默认值：2020-12-31。
**firstDay** | 一周的第一天。 默认值：0(星期日)。
**showWeekNumber** | 在行首显示周数。 默认值：false
**ignoreTimezone** | 显示与存储时完全相同的日期时间，忽略October和后端用户指定的时区

<a name="widget-fileupload"></a>
### File upload

`fileupload` - 呈现图像或常规文件的文件上传器。 字段名称必须使用attachOne或attachMany关系。

    avatar:
        label: Avatar
        type: fileupload
        mode: image
        imageHeight: 260
        imageWidth: 260
        thumbOptions:
            mode: crop
            offset:
                - 0
                - 0
            quality: 90
            sharpen: 0
            interlace: false
            extension: auto

选项 | 描述
------------- | -------------
**mode** | 预期的文件类型，文件或图像。 默认值：图片。
**imageWidth** | 如果使用图像类型，图像将调整为此宽度，可选。
**imageHeight** | 如果使用图像类型，图像将调整为此高度，可选。
**fileTypes** | 上传者接受的文件扩展名，可选。 例如：`zip，txt`
**mimeTypes** | 上传者接受的MIME类型，可以是文件扩展名，也可以是完全限定名，可选。 例如：`bin，txt`
**useCaption** | 允许为文件设置标题和描述。 默认值：true
**prompt** | 要为上传按钮显示的文本，仅适用于文件，可选。
**thumbOptions** | 传递给文件的缩略图生成方法的选项
**attachOnUpload** | 如果存在父记录，则在上载时自动附加上载的文件，而不是在保存父记录时使用延迟绑定附加。 默认为false。

<a name="widget-recordfinder"></a>
### Record finder

`recordfinder` - 呈现包含相关记录详细信息的字段。 展开字段会显示一个弹出列表以搜索大量记录。 仅受单一关系支持。

    user:
        label: User
        type: recordfinder
        list: $/rainlab/user/models/user/columns.yaml
        prompt: Click the %s button to find a user
        nameFrom: name
        descriptionFrom: email

选项 | 描述
------------- | -------------
**nameFrom**  |用于显示名称的关系中使用的列名称。默认值：名称。
**descriptionFrom**  |用于显示描述的关系中使用的列名。默认值：说明。
**title**  |要在弹出窗口的标题部分显示的文本。
**prompt**  |没有选择记录时显示的文本。 `%s`字符代表搜索图标。
**list**  |配置数组或对列表列定义文件的引用，请参阅[list columns](backend-lists.md#list-columns)。
**recordsPerPage**  |每页显示的记录，没有页面使用0。默认值：10
**conditions**  |指定要应用于列表模型查询的raw where查询语句。
**scope**  |指定**相关表单模型**中定义的[查询范围方法](database-model.md#query-scopes)以始终应用于列表查询。第一个参数将包含窗口小部件将其值附加到的模型，即父模型。
**searchMode**  |将搜索策略定义为包含所有单词，任何单词或精确短语。支持的选项：all，any，exact。默认值：全部。
**searchScope**  |指定在**相关表单模型**中定义的[查询范围方法](database-model.md#query-scopes)以应用于搜索查询，第一个参数将包含搜索项。

<a name="widget-mediafinder"></a>
### Media finder

`mediafinder` - 呈现用于从媒体管理器库中选择项目的字段。 展开字段会显示媒体管理器以查找文件。 结果选择是一个字符串作为文件的相对路径。

    background_image:
        label: Background image
        type: mediafinder
        mode: image

选项 | 描述
------------- | -------------
**mode** | 预期的文件类型，文件或图像。 默认值：文件。
**prompt** | 没有选择项目时显示的文本。 `%s`字符代表媒体管理器图标。
**imageWidth** | 如果使用图像类型，预览图像将显示为此宽度，可选。
**imageHeight** | 如果使用图像类型，预览图像将显示到此高度，可选。

<a name="widget-relation"></a>
### Relation

`relation` - 根据字段关系类型呈现下拉列表或复选框列表。 单数关系显示下拉列表，多个关系显示复选框列表。 用于显示每个关系的标签来自`nameFrom`或`select`定义。

    categories:
        label: Categories
        type: relation
        nameFrom: title

或者，您可以使用自定义`select`语句填充标签。 任何有效的SQL语句都适用于此处

    user:
        label: User
        type: relation
        select: concat(first_name, ' ', last_name)

您还可以提供一个模型范围，用于使用`scope`属性过滤结果。

选项 | 描述
------------- | -------------
**nameFrom** | 用于显示关系标签的模型属性名称。 默认值：名称。
**select** | 用于名称的自定义SQL select语句。
**emptyOption** | 没有可用选择时显示的文本。
**scope** | 指定**相关表单模型**中定义的[查询范围方法](database-model.md#query-scopes)以始终应用于列表查询。

<a name="widget-repeater"></a>
### Repeater

`repeater` - 呈现一组重复的表单字段。

    extra_information:
        type: repeater
        titleFrom: title_when_collapsed
        form:
            fields:
                added_at:
                    label: Date added
                    type: datepicker
                details:
                    label: Details
                    type: textarea
                title_when_collapsed:
                    label: This field is the title when collapsed
                    type: text

选项 | 描述
------------- | -------------
**form** | 对表单字段定义文件的引用，请参见[后端表单字段](#form-fields)。 也可以使用内联字段。
**prompt** | 要为创建按钮显示的文本。 默认值：添加新项目。
**titleFrom** | 项目中字段的名称，用作折叠项目的标题
**minItems** | 最低要求。 不使用组时预先显示这些项目
**maxItems** | 转发器内允许的最大项目数。
**groups** | 引用一组表单字段，将转发器置于组模式(见下文)。 也可以使用内联定义。

转发器字段支持组模式，该模式允许为每次迭代选择一组自定义字段。

    content:
        type: repeater
        prompt: Add content block
        groups: $/acme/blog/config/repeater_fields.yaml

这是组配置文件的示例，它位于 **/plugins/acme/blog/config/repeater_fields.yaml**中。 或者，这些定义可以与转发器一起指定。

    textarea:
        name: Textarea
        description: Basic text field
        icon: icon-file-text-o
        fields:
            text_area:
                label: Text Content
                type: textarea
                size: large

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

每个组必须指定唯一键，并且该定义支持以下选项。

选项 | 描述
------------- | -------------
**name** | 组的名称。
**description** | 该小组的简要说明。
**icon** | 定义组的图标，可选。
**fields** | 属于该组的表单字段，请参见[后端表单字段](#form-fields)。

> **注意**: 组密钥与保存的数据一起存储为`_group`属性。

<a name="widget-richeditor"></a>
### 富文本编辑器/WYSIWYG

`richeditor` - 为富格式文本呈现可视化编辑器，也称为WYSIWYG编辑器。

    html_content:
        type: richeditor
        toolbarButtons: bold|italic
        size: huge

选项 | 描述
------------- | -------------
**toolbarButtons** | 哪些按钮显示在编辑器工具栏上。

可用的工具栏按钮是：

    fullscreen, bold, italic, underline, strikeThrough, subscript, superscript, fontFamily, fontSize, |, color, emoticons, inlineStyle, paragraphStyle, |, paragraphFormat, align, formatOL, formatUL, outdent, indent, quote, insertHR, -, insertLink, insertImage, insertVideo, insertAudio, insertFile, insertTable, undo, redo, clearFormatting, selectAll, html

> **注意**: `|` 将在工具栏中插入一个垂直分隔线，并在`-`中插入一个水平分隔线。

<a name="widget-markdowneditor"></a>
### Markdown编辑器

`markdown` - 为markdown格式化文本呈现基本编辑器。

    md_content:
        type: markdown
        size: huge
        mode: split

选项 | 描述
------------- | -------------
**mode** | 预期的视图模式，tab或split。 默认标签页。

<a name="widget-taglist"></a>
### 标签列表

`taglist` - 呈现用于输入标签列表的字段。

    tags:
        type: taglist
        separator: space

标签列表可以支持三种定义“选项”的方式，与[下拉字段类型](#field-dropdown)完全相同。

    tags:
        type: taglist
        options:
            - Red
            - Blue
            - Orange

您可以使用名为**relation**的`mode`，其中字段名称是[多对多关系](database-relations.md#many-to-many)。 这将通过关系自动获取和分配标签。 如果支持自定义标记，则会在分配之前创建它们。

    tags:
        type: taglist
        mode: relation

选项 | 描述
------------- | -------------
**mode** | 控制返回值的方式，字符串，数组或关系。 默认值：字符串 string
**separator** | 使用指定字符(逗号或空格)分隔标记。 默认值：comma,
**customTags** | 允许用户手动输入自定义标签。 默认值：true
**options** | 指定预定义选项的方法或数组。 设置为true以使用模型`get*Field*Options`方法。 可选的。
**nameFrom** | 如果使用关系模式，则显示标签名称的模型属性名称。 默认值：name
**useKey** | 使用密钥而不是值来保存和读取数据。 默认值：false

<a name="form-views"></a>
## 表单视图

对于每个页面，您的表单支持 [Create](#form-create-page), [Update](#form-update-page) 和 [Preview](#form-preview-page) ，您应该提供[view file](#introduction)使用相应的名称 -  **create.htm**, **update.htm** and **preview.htm**。

表单行为向控制器类添加了两个方法：`formRender`和`formRenderPreview`。 这些方法呈现使用上述YAML文件配置的表单控件。

<a name="form-create-view"></a>
### 新建页面

**create.htm**视图表示允许用户创建新记录的“创建”页面。 典型的“创建”页面包含面包屑，表单本身和表单按钮。 ** data-request **属性应该引用表单行为提供的`onSave` AJAX处理程序。 以下是典型的create.htm表单的内容。

    <?= Form::open(['class'=>'layout']) ?>

        <div class="layout-row">
            <?= $this->formRender() ?>
        </div>

        <div class="form-buttons">
            <div class="loading-indicator-container">
                <button
                    type="button"
                    data-request="onSave"
                    data-request-data="close:true"
                    data-hotkey="ctrl+enter, cmd+enter"
                    data-load-indicator="Creating Category..."
                    class="btn btn-default">
                    Create and Close
                </button>
                <span class="btn-text">
                    or <a href="<?= Backend::url('acme/blog/categories') ?>">Cancel</a>
                </span>
            </div>
        </div>

    <?= Form::close() ?>

<a name="form-update-view"></a>
### 更新页面

**update.htm**视图表示允许用户更新或删除现有记录的“更新”页面。 典型的“更新”页面包含面包屑，表单本身和表单按钮。 “更新”页面与“创建”页面非常相似，但通常具有“删除”按钮。 **data-request**属性应该引用表单行为提供的`onSave` AJAX处理程序。 以下是典型update.htm表单的内容。

    <?= Form::open(['class'=>'layout']) ?>

        <div class="layout-row">
            <?= $this->formRender() ?>
        </div>

        <div class="form-buttons">
            <div class="loading-indicator-container">
                <button
                    type="button"
                    data-request="onSave"
                    data-request-data="close:true"
                    data-hotkey="ctrl+enter, cmd+enter"
                    data-load-indicator="Saving Category..."
                    class="btn btn-default">
                    Save and Close
                </button>
                <button
                    type="button"
                    class="oc-icon-trash-o btn-icon danger pull-right"
                    data-request="onDelete"
                    data-load-indicator="Deleting Category..."
                    data-request-confirm="Do you really want to delete this category?">
                </button>
                <span class="btn-text">
                    or <a href="<?= Backend::url('acme/blog/categories') ?>">Cancel</a>
                </span>
            </div>
        </div>

    <?= Form::close() ?>

<a name="form-preview-view"></a>
### 预览页面

** preview.htm **视图表示预览页面，允许用户以只读模式预览现有记录。 典型的预览页面包含面包屑和表单本身。 以下是典型的preview.htm表单的内容。

    <div class="form-preview">
        <?= $this->formRenderPreview() ?>
    </div>

<a name="field-conditions"></a>
## 将条件应用于字段

有时您可能希望在某些条件下操纵表单字段的值或外观，例如，如果勾选了复选框，您可能希望隐藏输入。 通过使用触发器API或字段依赖性，有几种方法可以做到这一点。 输入预设转换器主要用于转换字段值。 下面更详细地描述这些选项。

<a name="field-input-preset"></a>
### 输入预设转换器

输入预设转换器使用“预设”[表单字段选项](#form-field-options)定义，并允许您将输入到元素中的文本转换为另一个输入元素中的URL，slug或文件名值。

在这个例子中，当用户在`title`字段中输入文本时，我们将自动填写`url`字段值。 如果为标题键入了文本**Hello world**，则URL将跟随**/hello-world**的转换值。 仅当目标字段(`url`)为空且未触及时才会发生此行为。
 
    title:
        label: Title

    url:
        label: URL
        preset:
            field: title
            type: url

或者，`preset`值也可以是仅引用**字段**的字符串，然后`type`选项将默认为**slug**。

    slug:
        label: Slug
        preset: title

以下选项可用于“预设”选项：

选项 | 描述
------------- | -------------
**field** | 定义另一个字段名称以从中获取值。
**type** | 指定转换类型。 请参阅下面的支持值。
**prefixInput** | 可选，使用CSS选择器为转换后的值添加前缀，并在提供的输入元素中找到该值。

以下是支持的类型：

输入 | 描述
------------- | -------------
**exact** | 复制确切的值
**slug** | 将复制的值格式化为slug
**url** | 与slug相同但以/为前缀
**camel** | 使用camelCase格式化复制的值
**file** | 将复制的值格式化为文件名，并用空格替换为空格

<a name="field-trigger-events"></a>
### 触发事件

触发事件使用`trigger` [表单字段选项](#form-field-options) 定义，是一个使用JavaScript的基于浏览器的简单解决方案。 它允许您根据其他元素的状态更改元素属性，例如可见性或值。 这是一个示例定义：

    is_delayed:
        label: Send later
        comment: Place a tick in this box if you want to send this message at a later time.
        type: checkbox

    send_at:
        label: Send date
        type: datepicker
        cssClass: field-indent
        trigger:
            action: show
            field: is_delayed
            condition: checked

在上面的例子中，只有在选中`is_delayed`字段时才会显示`send_at`表单字段。 换句话说，如果检查了另一个表单输入(字段)(条件)，该字段将显示(动作)。 `trigger`定义指定了以下选项：

选项 | 描述
------------- | -------------
**action** | 定义满足条件时应用于此字段的操作。 支持的值：show，hide，enable，disable，empty。
**field** | 定义将触发操作的其他字段名称。 通常，字段名称是指同一级别形式的字段。 例如，如果此字段位于[转发器窗口小部件](#widget-repeater) 中，则仅检查相同[转发器窗口小部件](#widget-repeater) 中的字段。 但是，如果字段名称前面带有插入符号`^`，如：`^ parent_field`，它将引用[repeater widget](#widget-repeater) 或形成比字段本身高一级的插入符号。 另外，如果使用多个插入符号^ ^，它将引用更高级别：`^^ grand_parent_field`，`^^^ grand_grand_parent_field`等。
**condition** | 确定指定字段应满足的条件，以使条件被视为“真”。 支持的值：选中，未选中，值[somevalue]。

<a name="field-dependencies"></a>
### 字段依赖性

在定义`dependsOn` [表单字段选项](#form-field-options)时，表单字段可以依赖于其他字段，这提供了更强大的服务器端解决方案。 当定义的其他字段发生更改时，定义字段将使用AJAX框架进行更新。 这是一个示例定义：

    country:
        label: Country
        type: dropdown

    state:
        label: State
        type: dropdown
        dependsOn: country

在上面的例子中，`state`表单字段将在`country`字段具有更改的值时刷新。 发生这种情况时，当前表单数据将填入模型中，因此下拉选项可以使用它。

    public function getCountryOptions()
    {
        return ['au' => 'Australia', 'ca' => 'Canada'];
    }

    public function getStateOptions()
    {
        if ($this->country == 'au') {
            return ['act' => 'Capital Territory', 'qld' => 'Queensland', ...];
        }
        elseif ($this->country == 'ca') {
            return ['bc' => 'British Columbia', 'on' => 'Ontario', ...];
        }
    }

此示例对于操作模型值很有用，但它无权访问表单字段定义。 您可以通过在模型中定义`filterFields`方法来过滤表单字段，如[过滤表单字段](#filter-form-fields) 部分所述。

<a name="prevent-field-submission"></a>
### 防止字段提交

有时您可能需要阻止提交字段。 为此，只需在表单配置文件中的字段名称前添加下划线(\ __)。 以下划线开头的表单字段将自动清除，不再保存到模型中。

    address:
        label: Title
        type: text

    _map:
        label: Point your address on the map
        type: mapviewer

<a name="extend-form-behavior"></a>
## 扩展表单行为

有时您可能希望修改默认表单行为，有几种方法可以执行此操作。

<a name="overriding-action"></a>
### 覆盖控制器action

您可以将自己的逻辑用于控制器中的`create`，`update`或`preview`操作方法，然后可选地调用Form行为父方法。

    public function update($recordId, $context = null)
    {
        //
        //Do any custom code here
        //

        //Call the FormController behavior update() method
        return $this->asExtension('FormController')->update($recordId, $context);
    }

<a name="extend-model-query"></a>
### 扩展表单模型查询

可以通过覆盖控制器类中的`formExtendQuery`方法来扩展表单[数据库模型](database-model.md)的查询查询。 此示例将通过将**withTrashed**作用域应用于查询来确保仍可以找到并更新软删除的记录：

    public function formExtendQuery($query)
    {
        $query->withTrashed();
    }

<a name="extend-form-fields"></a>
### 扩展表单字段

您可以通过在控制器类上调用`extendFormFields`静态方法从外部扩展另一个控制器的字段。 此方法可以采用三个参数，**$form**表示Form小部件对象，**$model**表示表单使用的模型，**$context**是包含表单上下文的字符串。 以此控制器为例：

    class Categories extends \Backend\Classes\Controller
    {
        public $implement = ['Backend.Behaviors.FormController'];

        public $formConfig = 'form_config.yaml';
    }

使用`extendFormFields`方法，您可以向此控制器呈现的任何表单添加额外的字段。 由于这可能会影响此控制器使用的所有表单，因此最好检查**$model**的类型是否正确。 这是一个例子：

    Categories::extendFormFields(function($form, $model, $context)
    {
        if (!$model instanceof MyModel) {
            return;
        }

        $form->addFields([
            'my_field' => [
                'label'   => 'My Field',
                'comment' => 'This is a custom field I have added.',
            ],
        ]);

    });

您还可以通过覆盖控制器类中的`formExtendFields`方法在内部扩展表单字段。 这只会影响`FormController`行为使用的形式。

    class Categories extends \Backend\Classes\Controller
    {
        [...]

        public function formExtendFields($form)
        {
            $form->addFields([...]);
        }
    }

$form对象提供以下方法。

方法 | 描述
------------- | -------------
**addFields** | 向外部区域添加新字段
**addTabFields** | 向选项卡区域添加新字段
**addSecondaryTabFields** | 将新字段添加到辅助选项卡区域
**removeField** | 从任何区域删除字段

每个方法都采用类似于[表单字段配置](#form-fields)的字段数组。

<a name="filter-form-fields"></a>
### 过滤表单字段

您可以通过覆盖所用模型中的`filterFields`方法来过滤表单字段定义。 这允许您根据模型数据操纵可见性和其他字段属性。 该方法有两个参数**$fields**表示已经由[field configuration](#form-fields)定义的字段的对象，**$context**表示活动表单context。

    public function filterFields($fields, $context = null)
    {
        if ($this->source_type == 'http') {
            $fields->source_url->hidden = false;
            $fields->git_branch->hidden = true;
        }
        elseif ($this->source_type == 'git') {
            $fields->source_url->hidden = false;
            $fields->git_branch->hidden = false;
        }
        else {
            $fields->source_url->hidden = true;
            $fields->git_branch->hidden = true;
        }
    }

上面的例子将通过检查Model属性`source_type`的值在某些字段上设置`hidden`标志。 当表单首次加载时以及通过[定义的字段依赖性](#field-dependencies)更新时，将应用此逻辑。

<a name="validate-form-fields"></a>
## 验证表单字段
要验证表单的字段，您可以在模型中使用[验证](../database/traits#validation)特征。
