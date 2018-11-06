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

表单行为取决于表单[字段定义](#form-fields)和[模型类](../database/model)。 为了使用表单行为，您应该将它添加到控制器类的`$implement`属性中。 此外，应定义`$formConfig`类属性，其值应引用用于配置行为选项的YAML文件。

    namespace Acme\Blog\Controllers;

    class Categories extends \Backend\Classes\Controller
    {
        public $implement = ['Backend.Behaviors.FormController'];

        public $formConfig = 'form_config.yaml';
    }

> **注意:** 表单和[列表行为](lists)通常在同一个控制器中一起使用。

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

下面列出的配置选项是可选的。 如果希望表单行为支持[Create](#form-create-page), [Update](#form-update-page) or [Preview](#form-preview-page)页面，请定义它们。

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
**title** | 页面标题，可以参考[本地化字符串](../plugin/localization).。
**redirect** | 保存记录时的重定向页面。
**redirectClose** | 保存记录时的重定向页面，并随请求一起发送**close** post变量。
**flashSave** | 保存记录时显示的flash消息，可以参考[本地化字符串](../plugin/localization)。
**form** | 仅覆盖创建页面的默认表单字段定义。

<a name="form-update-page"></a>
### 更新页面

要支持“更新”页面，请将以下配置添加到YAML文件中：

    update:
        title: 编辑文章
        redirect: acme/blog/posts
        flashSave:  保存成功
        flashDelete: 删除成功

“更新”页面支持以下配置选项：

选项 | 描述
------------- | -------------
**title** | 页面标题，可以参考[本地化字符串]（../插件/本地化）。
**redirect** | 保存记录时的重定向页面。
**redirectClose** | 保存记录时的重定向页面和请求一起发送**close** post变量。
**flashSave** | 保存记录时显示的flash消息，可以参考[本地化字符串]（../插件/本地化）。
**flashDelete** | 删除记录时显示的flash消息，可以参考[本地化字符串]（../插件/本地化）。
**form** | 仅覆盖更新页面的默认表单字段定义。

<a name="form-preview-page"></a>
### 预览页面

要支持“预览”页面，请将以下配置添加到YAML文件中：

    preview:
        title: 博客详情页

“预览”页面支持以下配置选项：

选项  | 描述
------------- | -------------
**title** | 页面标题，可以参考[本地化字符串]（../插件/本地化）。
**form** | 仅覆盖预览页面的默认表单字段定义。

<a name="form-fields"></a>
## 定义表单字段

表单字段使用YAML文件定义。 表单字段配置由表单行为用于创建表单控件并将它们绑定到模型字段。 该文件放在插件的**models**目录的子目录中。 子目录名称与以小写字母书写的模型类名称匹配。 文件名无关紧要，但**fields.yaml**和**form_fields.yaml**是常用名称。 示例表单字段文件位置：

    plugins/
      acme/
        blog/
          models/            <=== Plugin models directory
            post/            <=== Model configuration directory
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

可以使用[Relation Widget](＃widget-relation)或[Relation Manager](relations＃relationship-types)呈现相关模型中的字段。 例外是OneToOne或morphOne相关字段，必须定义为**relation[field]**，然后可以指定为模型的任何其他字段：

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

对于每个字段，您可以指定这些选项（如果适用）：

选项 | 描述
------------- | -------------
**label** | 向用户显示表单字段时的名称。
**type** | defines how this field should be rendered (see [Available fields types](#field-types) below). Default: text.
**span** | aligns the form field to one side. Options: auto, left, right, full. Default: full.
**size** | specifies a field size for fields that use it, for example, the textarea field. Options: tiny, small, large, huge, giant.
**placeholder** | if the field supports a placeholder value.
**comment** | places a descriptive comment below the field.
**commentAbove** | places a comment above the field.
**commentHtml** | allow HTML markup inside the comment. Options: true, false.
**default** | specifies the default value for the field.
**defaultFrom** | takes the default value from the value of another field.
**tab** | assigns the field to a tab.
**cssClass** | assigns a CSS class to the field container.
**readOnly** | prevents the field from being modified. Options: true, false.
**disabled** | prevents the field from being modified and excludes it from the saved data. Options: true, false.
**hidden** | hides the field from the view and excludes it from the saved data. Options: true, false.
**stretch** | specifies if this field stretches to fit the parent height.
**context** | specifies what context should be used when displaying the field. Context can also be passed by using an `@` symbol in the field name, for example, `name@update`.
**dependsOn** | an array of other field names this field [depends on](#field-dependencies), when the other fields are modified, this field will update.
**trigger** | specify conditions for this field using [trigger events](#field-trigger-events).
**preset** | allows the field value to be initially set by the value of another field, converted using the [input preset converter](#field-input-preset).
**required** | places a red asterisk next to the field label to indicate it is required (make sure to setup validation on the model as this is not enforced by the form controller).
**attributes** | specify custom HTML attributes to add to the form field element.
**containerAttributes** | specify custom HTML attributes to add to the form field container.

<a name="field-types"></a>
## 可用的字段类型

There are various native field types that can be used for the **type** setting. For more advanced form fields, a [form widget](#form-widgets) can be used instead.

<style>
    .collection-method-list {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="content-list collection-method-list" markdown="1">
- [Text](#field-text)
- [Number](#field-number)
- [Password](#field-password)
- [Textarea](#field-textarea)
- [Dropdown](#field-dropdown)
- [Radio List](#field-radio)
- [Balloon Selector](#field-balloon)
- [Checkbox](#field-checkbox)
- [Checkbox List](#field-checkboxlist)
- [Switch](#field-switch)
- [Section](#field-section)
- [Partial](#field-partial)
- [Hint](#field-hint)
- [Widget](#field-widget)
</div>

<a name="field-text"></a>
### Text

`text` - renders a single line text box. This is the default type used if none is specified.

    blog_title:
        label: Blog Title
        type: text

<a name="field-number"></a>
### Number

`number` - renders a single line text box that takes numbers only.

    your_age:
        label: Your Age
        type: number

<a name="field-password"></a>
### Password

`password ` - renders a single line password field.

    user_password:
        label: Password
        type: password

<a name="field-textarea"></a>
### Textarea

`textarea` - renders a multiline text box. A size can also be specified with possible values: tiny, small, large, huge, giant.

    blog_contents:
        label: Contents
        type: textarea
        size: large

<a name="field-dropdown"></a>
### Dropdown

`dropdown` - renders a dropdown with specified options. There are 4 ways to provide the drop-down options. The first method defines `options` directly in the YAML file:

    status_type:
        label: Blog Post Status
        type: dropdown
        options:
            draft: Draft
            published: Published
            archived: Archived

The second method defines options with a method declared in the model's class. If the options element is omitted, the framework expects a method with the name `get*FieldName*Options` to be defined in the model. Using the example above, the model should have the `getStatusTypeOptions` method. The first argument of this method is the current value of this field and the second is the current data object for the entire form. This method should return an array of options in the format **key => label**.

    status_type:
        label: Blog Post Status
        type: dropdown

Supplying the dropdown options in the model class:

    public function getStatusTypeOptions($value, $formData)
    {
        return ['all' => 'All', ...];
    }

The third global method `getDropdownOptions` can also be defined in the model, this will be used for all dropdown field types for the model. The first argument of this method is the field name, the second is the currect value of the field, and the third is the current data object for the entire form. It should return an array of options in the format **key => label**.

    public function getDropdownOptions($fieldName, $value, $formData)
    {
        if ($fieldName == 'status') {
            return ['all' => 'All', ...];
        }
        else {
            return ['' => '-- none --'];
        }
    }

The fourth method uses a specific method declared in the model's class. In the next example the `listStatuses` method should be defined in the model class. This method receives all the same arguments as the `getDropdownOptions` method, and should return an array of options in the format **key => label**.

    status:
        label: Blog Post Status
        type: dropdown
        options: listStatuses

Supplying the dropdown options to the model class:

    public function listStatuses($fieldName, $value, $formData)
    {
        return ['published' => 'Published', ...];
    }

To define the behavior when there is no selection, you may specify an `emptyOption` value to include an empty option that can be reselected.

    status:
        label: Blog Post Status
        type: dropdown
        emptyOption: -- no status --

Alternatively you may use the `placeholder` option to use a "one-way" empty option that cannot be reselected.

    status:
        label: Blog Post Status
        type: dropdown
        placeholder: -- select a status --

By default the dropdown has a searching feature, allowing quick selection of a value. This can be disabled by setting the `showSearch` option to `false`.

    status:
        label: Blog Post Status
        type: dropdown
        showSearch: false

<a name="field-radio"></a>
### Radio List

`radio` - renders a list of radio options, where only one item can be selected at a time.

    security_level:
        label: Access Level
        type: radio
        options:
            all: All
            registered: Registered only
            guests: Guests only

Radio lists can also support a secondary description.

    security_level:
        label: Access Level
        type: radio
        options:
            all: [All, Guests and customers will be able to access this page.]
            registered: [Registered only, Only logged in member will be able to access this page.]
            guests: [Guests only, Only guest users will be able to access this page.]

Radio lists support three ways of defining the options, exactly like the [dropdown field type](#field-dropdown). For radio lists the method could return either the simple array: **key => value** or an array of arrays for providing the descriptions: **key => [label, description]**

<a name="field-balloon"></a>
### Balloon Selector

`balloon-selector` - renders a list, where only one item can be selected at a time.

    gender:
        label: Gender
        type: balloon-selector
        options:
            female: Female
            male: Male

Balloon selectors support three ways of defining the options, exactly like the [dropdown field type](#field-dropdown).

<a name="field-checkbox"></a>
### Checkbox

`checkbox` - renders a single checkbox.

    show_content:
        label: Display content
        type: checkbox
        default: true

<a name="field-checkboxlist"></a>
### Checkbox List

`checkboxlist` - renders a list of checkboxes.

    permissions:
        label: Permissions
        type: checkboxlist
        options:
            open_account: Open account
            close_account: Close account
            modify_account: Modify account

Checkbox lists support three ways of defining the options, exactly like the [dropdown field type](#field-dropdown) and also support secondary descriptions, found in the [radio field type](#field-radio).

<a name="field-switch"></a>
### Switch

`switch` - renders a switchbox.

    show_content:
        label: Display content
        type: switch
        comment: Flick this switch to display content
        on: myauthor.myplugin::lang.models.mymodel.show_content.on
        off: myauthor.myplugin::lang.models.mymodel.show_content.off

<a name="field-section"></a>
### Section

`section` - renders a section heading and subheading. The `label` and `comment` values are optional and contain the content for the heading and subheading.

    user_details_section:
        label: User details
        type: section
        comment: This section contains details about the user.

<a name="field-partial"></a>
### Partial

`partial` - renders a partial, the `path` value can refer to a partial view file otherwise the field name is used as the partial name. Inside the partial these variables are available: `$value` is the default field value, `$model` is the model used for the field and `$field` is the configured class object `Backend\Classes\FormField`.

    content:
        type: partial
        path: $/acme/blog/models/comments/_content_field.htm

<a name="field-hint"></a>
### Hint

`hint` - identical to a `partial` field but renders inside a hint container that can be hidden by the user.

    content:
        type: hint
        path: content_field

<a name="field-widget"></a>
### Widget

`widget` - renders a custom form widget, the `type` field can refer directly to the class name of the widget or the registered alias name.

    blog_content:
        type: Backend\FormWidgets\RichEditor
        size: huge

<a name="form-widgets"></a>
## 表单小部件

There are various form widgets included as standard, although it is common for plugins to provide their own custom form widgets. You can read more on the [Form Widgets](widgets#form-widgets) article.

<div class="content-list collection-method-list" markdown="1">
- [Code editor](#widget-codeeditor)
- [Color picker](#widget-colorpicker)
- [Date picker](#widget-datepicker)
- [File upload](#widget-fileupload)
- [Record finder](#widget-recordfinder)
- [Media finder](#widget-mediafinder)
- [Relation](#widget-relation)
- [Repeater](#widget-repeater)
- [Rich editor / WYSIWYG](#widget-richeditor)
- [Markdown editor](#widget-markdowneditor)
- [Tag list](#widget-taglist)
</div>

<a name="widget-codeeditor"></a>
### Code editor

`codeeditor` - renders a plaintext editor for formatted code or markup. Note the options may be inherited by the code editor preferences defined for the Administrator in the back-end.

    css_content:
        type: codeeditor
        size: huge
        language: html

Option | Description
------------- | -------------
**language** | code language, for example, php, css, js, html. Default: php.
**showGutter** | shows a gutter with line numbers. Default: true.
**wrapWords** | breaks long lines on to a new line. Default true.
**fontSize** | the text font size. Default: 12.

<a name="widget-colorpicker"></a>
### Color picker
`colorpicker` - renders controls to select a hexadecimal color value.

    color:
        label: Background
        type: colorpicker

Option | Description
------------- | -------------
**availableColors** |  list of available colors.

<a name="widget-datepicker"></a>
### Date picker

`datepicker` - renders a text field used for selecting date and times.

    published_at:
        label: Published
        type: datepicker
        mode: date

Option | Description
------------- | -------------
**mode** | the expected result, either date, datetime or time. Default: datetime.
**format** | provide an explicit date display format. Eg: Y-m-d
**minDate** | the minimum/earliest date that can be selected. Default: 2000-01-01.
**maxDate** | the maximum/latest date that can be selected. Default: 2020-12-31.
**firstDay** | the first day of the week. Default: 0 (Sunday).
**showWeekNumber** | show week numbers at head of row. Default: false
**ignoreTimezone** | display datetime exactly as it is stored, ignoring October's and the backend user's specified timezones

<a name="widget-fileupload"></a>
### File upload

`fileupload` - renders a file uploader for images or regular files. The field name must use an attachOne or attachMany relation.

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

Option | Description
------------- | -------------
**mode** | the expected file type, either file or image. Default: image.
**imageWidth** | if using image type, the image will be resized to this width, optional.
**imageHeight** | if using image type, the image will be resized to this height, optional.
**fileTypes** | file extensions that are accepted by the uploader, optional. Eg: `zip,txt`
**mimeTypes** | MIME types that are accepted by the uploader, either as file extension or fully qualified name, optional. Eg: `bin,txt`
**useCaption** | allows a title and description to be set for the file. Default: true
**prompt** | text to display for the upload button, applies to files only, optional.
**thumbOptions** | options to pass to the thumbnail generating method for the file
**attachOnUpload** | Automatically attaches the uploaded file on upload if the parent record exists instead of using deferred binding to attach on save of the parent record. Defaults to false.

<a name="widget-recordfinder"></a>
### Record finder

`recordfinder` - renders a field with details of a related record. Expanding the field displays a popup list to search large amounts of records. Supported by singular relationships only.

    user:
        label: User
        type: recordfinder
        list: $/rainlab/user/models/user/columns.yaml
        prompt: Click the %s button to find a user
        nameFrom: name
        descriptionFrom: email

Option | Description
------------- | -------------
**nameFrom** | the column name to use in the relation used for displaying the name. Default: name.
**descriptionFrom** | the column name to use in the relation used for displaying a description. Default: description.
**title** | text to display in the title section of the popup.
**prompt** | text to display when there is no record selected. The `%s` character represents the search icon.
**list** | a configuration array or reference to a list column definition file, see [list columns](lists#list-columns).
**recordsPerPage** | records to display per page, use 0 for no pages. Default: 10
**conditions** | specifies a raw where query statement to apply to the list model query.
**scope** | specifies a [query scope method](../database/model#query-scopes) defined in the **related form model** to apply to the list query always. The first argument will contain the model that the widget will be attaching its value to, i.e. the parent model.
**searchMode** | defines the search strategy to either contain all words, any word or exact phrase. Supported options: all, any, exact. Default: all.
**searchScope** | specifies a [query scope method](../database/model#query-scopes) defined in the **related form model** to apply to the search query, the first argument will contain the search term.

<a name="widget-mediafinder"></a>
### Media finder

`mediafinder` - renders a field for selecting an item from the media manager library. Expanding the field displays the media manager to locate a file. The resulting selection is a string as the relative path to the file.

    background_image:
        label: Background image
        type: mediafinder
        mode: image

Option | Description
------------- | -------------
**mode** | the expected file type, either file or image. Default: file.
**prompt** | text to display when there is no item selected. The `%s` character represents the media manager icon.
**imageWidth** | if using image type, the preview image will be displayed to this width, optional.
**imageHeight** | if using image type, the preview image will be displayed to this height, optional.

<a name="widget-relation"></a>
### Relation

`relation` - renders either a dropdown or checkbox list according to the field relation type. Singular relationships display a dropdown, multiple relationships display a checkbox list. The label used for displaying each relation is sourced by the `nameFrom` or `select` definition.

    categories:
        label: Categories
        type: relation
        nameFrom: title

Alternatively, you may populate the label using a custom `select` statement. Any valid SQL statement works here.

    user:
        label: User
        type: relation
        select: concat(first_name, ' ', last_name)

You can also provide a model scope to use to filter the results with the `scope` property.

Option | Description
------------- | -------------
**nameFrom** | a model attribute name used for displaying the relation label. Default: name.
**select** | a custom SQL select statement to use for the name.
**emptyOption** | text to display when there is no available selections.
**scope** | specifies a [query scope method](../database/model#query-scopes) defined in the **related form model** to apply to the list query always.

<a name="widget-repeater"></a>
### Repeater

`repeater` - renders a repeating set of form fields defined within.

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

Option | Description
------------- | -------------
**form** | a reference to form field definition file, see [backend form fields](#form-fields). Inline fields can also be used.
**prompt** | text to display for the create button. Default: Add new item.
**titleFrom** | name of field within items to use as the title for the collapsed item
**minItems** | minimum items required. Pre-displays those items when not using groups
**maxItems** | maximum number of items to allow within the repeater.
**groups** | references a group of form fields placing the repeater in group mode (see below). An inline definition can also be used.

The repeater field supports a group mode which allows a custom set of fields to be chosen for each iteration.

    content:
        type: repeater
        prompt: Add content block
        groups: $/acme/blog/config/repeater_fields.yaml

This is an example of a group configuration file, which would be located in **/plugins/acme/blog/config/repeater_fields.yaml**. Alternatively these definitions could be specified inline with the repeater.

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

Each group must specify a unique key and the definition supports the following options.

Option | Description
------------- | -------------
**name** | the name of the group.
**description** | a breif description of the group.
**icon** | defines an icon for the group, optional.
**fields** | form fields belonging to the group, see [backend form fields](#form-fields).

> **Note**: The group key is stored along with the saved data as the `_group` attribute.

<a name="widget-richeditor"></a>
### Rich editor / WYSIWYG

`richeditor` - renders a visual editor for rich formatted text, also known as a WYSIWYG editor.

    html_content:
        type: richeditor
        toolbarButtons: bold|italic
        size: huge

Option | Description
------------- | -------------
**toolbarButtons** | which buttons to show on the editor toolbar.

The available toolbar buttons are:

    fullscreen, bold, italic, underline, strikeThrough, subscript, superscript, fontFamily, fontSize, |, color, emoticons, inlineStyle, paragraphStyle, |, paragraphFormat, align, formatOL, formatUL, outdent, indent, quote, insertHR, -, insertLink, insertImage, insertVideo, insertAudio, insertFile, insertTable, undo, redo, clearFormatting, selectAll, html

> **Note**: `|` will insert a vertical separator line in the toolbar and `-` a horizontal one.

<a name="widget-markdowneditor"></a>
### Markdown editor

`markdown` - renders a basic editor for markdown formatted text.

    md_content:
        type: markdown
        size: huge
        mode: split

Option | Description
------------- | -------------
**mode** | the expected view mode, either tab or split. Default: tab.

<a name="widget-taglist"></a>
### Tag list

`taglist` - renders a field for inputting a list of tags.

    tags:
        type: taglist
        separator: space

A tag list can support three ways of defining the `options`, exactly like the [dropdown field type](#field-dropdown).

    tags:
        type: taglist
        options:
            - Red
            - Blue
            - Orange

You may use the `mode` called **relation** where the field name is a [many-to-many relationship](../database/relations#many-to-many). This will automatically source and assign tags via the relationship. If custom tags are supported, they will be created before assignment.

    tags:
        type: taglist
        mode: relation

Option | Description
------------- | -------------
**mode** | controls how the value is returned, either string, array or relation. Default: string
**separator** | separate tags with the specified character, either comma or space. Default: comma
**customTags** | allows custom tags to be entered manually by the user. Default: true
**options** | specifies a method or array for predefined options. Set to true to use model `get*Field*Options` method. Optional.
**nameFrom** | if relation mode is used, a model attribute name for displaying the tag name. Default: name
**useKey** | use the key instead of value for saving and reading data. Default: false

<a name="form-views"></a>
## 表单视图

For each page your form supports [Create](#form-create-page), [Update](#form-update-page) and [Preview](#form-preview-page) you should provide a [view file](#introduction) with the corresponding name - **create.htm**, **update.htm** and **preview.htm**.

The form behavior adds two methods to the controller class: `formRender` and `formRenderPreview`. These methods render the form controls configured with the YAML file described above.

<a name="form-create-view"></a>
### 新建页面

The **create.htm** view represents the Create page that allows users to create new records. A typical Create page contains breadcrumbs, the form itself, and the form buttons. The **data-request** attribute should refer to the `onSave` AJAX handler provided by the form behavior. Below is a contents of the typical create.htm form.

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

The **update.htm** view represents the Update page that allows users to update or delete existing records. A typical Update page contains breadcrumbs, the form itself, and the form buttons. The Update page is very similar to the Create page, but usually has the Delete button. The **data-request** attribute should refer to the `onSave` AJAX handler provided by the form behavior. Below is a contents of the typical update.htm form.

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

The **preview.htm** view represents the Preview page that allows users to preview existing records in the read-only mode. A typical Preview page contains breadcrumbs and the form itself. Below is a contents of the typical preview.htm form.

    <div class="form-preview">
        <?= $this->formRenderPreview() ?>
    </div>

<a name="field-conditions"></a>
## 将条件应用于字段

Sometimes you may want to manipulate the value or appearance of a form field under certain conditions, for example, you may want to hide an input if a checkbox is ticked. There are a few ways you can do this, either by using the trigger API or field dependencies. The input preset converter is primarily used to converting field values. These options are described in more detail below.

<a name="field-input-preset"></a>
### 输入预设转换器

The input preset converter is defined with the `preset` [form field option](#form-field-options) and allows you to convert text entered into an element to a URL, slug or file name value in another input element.

In this example we will automatically fill out the `url` field value when a user enters text in the `title` field. If the text **Hello world** is typed in for the Title, the URL will follow suit with the converted value of **/hello-world**. This behavior will only occur when the destination field (`url`) is empty and untouched.

    title:
        label: Title

    url:
        label: URL
        preset:
            field: title
            type: url

Alternatively, the `preset` value can also be a string that refers to the **field** only, the `type` option will then default to **slug**.

    slug:
        label: Slug
        preset: title

The following options are available for the `preset` option:

Option | Description
------------- | -------------
**field** | defines the other field name to source the value from.
**type** | specifies the conversion type. See below for supported values.
**prefixInput** | optional, prefixes the converted value with the value found in the supplied input element using a CSS selector.

Following are the supported types:

Type | Description
------------- | -------------
**exact** | copies the exact value
**slug** | formats the copied value as a slug
**url** | same as slug but prefixed with a /
**camel** | formats the copied value with camelCase
**file** | formats the copied value as a file name with whitespace replaced with dashes

<a name="field-trigger-events"></a>
### 触发事件

Trigger events are defined with the `trigger` [form field option](#form-field-options) and is a simple browser based solution that uses JavaScript. It allows you to change elements attributes such as visibility or value, based on another elements' state. Here is a sample definition:

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

In the above example the `send_at` form field will only be shown if the `is_delayed` field is checked. In other words, the field will show (action) if the other form input (field) is checked (condition). The `trigger` definition specifies these options:

Option | Description
------------- | -------------
**action** | defines the action applied to this field when the condition is met. Supported values: show, hide, enable, disable, empty.
**field** | defines the other field name that will trigger the action. Normally the field name refers to a field in the same level form. For example, if this field is in a [repeater widget](#widget-repeater), only fields in that same [repeater widget](#widget-repeater) will be checked. However, if the field name is preceeded by a caret symbol `^` like: `^parent_field`, it will refer to a [repeater widget](#widget-repeater) or form one level higher than the field itself. Additionally, if more than one caret `^` is used, it will refer that many levels higher: `^^grand_parent_field`, `^^^grand_grand_parent_field`, etc.
**condition** | determines the condition the specified field should satisfy for the condition to be considered "true". Supported values: checked, unchecked, value[somevalue].

<a name="field-dependencies"></a>
### 字段依赖性

Form fields can depend on others when defining the `dependsOn` [form field option](#form-field-options) which provides a more robust server side solution. When the defined other fields change, the defining field will update using the AJAX framework. Here is a sample definition:

    country:
        label: Country
        type: dropdown

    state:
        label: State
        type: dropdown
        dependsOn: country

In the above example the `state` form field will refresh when the `country` field has a changed value. When this occurs, the current form data will be filled in the model so the dropdown options can use it.

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

This example is useful for manipulating the model values, but it does not have access to the form field definitions. You can filter the form fields by defining a `filterFields` method inside the model, described in the [Filtering form fields](#filter-form-fields) section.

<a name="prevent-field-submission"></a>
### 防止字段提交

Sometimes you may need to prevent a field from being submitted. In order to do that, just add an underscore (\_) before the name of the field in the form configuration file. Form fields beginning with an underscore are purged automatically and no longer saved to the model.

    address:
        label: Title
        type: text

    _map:
        label: Point your address on the map
        type: mapviewer

<a name="extend-form-behavior"></a>
## 扩展表单行为

Sometimes you may wish to modify the default form behavior and there are several ways you can do this.

<a name="overriding-action"></a>
### 覆盖控制器action

You can use your own logic for the `create`, `update` or `preview` action method in the controller, then optionally call the Form behavior parent method.

    public function update($recordId, $context = null)
    {
        //
        // Do any custom code here
        //

        // Call the FormController behavior update() method
        return $this->asExtension('FormController')->update($recordId, $context);
    }

<a name="extend-model-query"></a>
### 扩展表单模型查询

The lookup query for the form [database model](../database/model) can be extended by overriding the `formExtendQuery` method inside the controller class. This example will ensure that soft deleted records can still be found and updated, by applying the **withTrashed** scope to the query:

    public function formExtendQuery($query)
    {
        $query->withTrashed();
    }

<a name="extend-form-fields"></a>
### 扩展表单字段

You can extend the fields of another controller from outside by calling the `extendFormFields` static method on the controller class. This method can take three arguments, **$form** will represent the Form widget object, **$model** represents the model used by the form and **$context** is a string containing the form context. Take this controller for example:

    class Categories extends \Backend\Classes\Controller
    {
        public $implement = ['Backend.Behaviors.FormController'];

        public $formConfig = 'form_config.yaml';
    }

Using the `extendFormFields` method you can add extra fields to any form rendered by this controller. Since this has the potential to affect all forms used by this controller, it is a good idea to check the **$model** is of the correct type. Here is an example:

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

You can also extend the form fields internally by overriding the `formExtendFields` method inside the controller class. This will only affect the form used by the `FormController` behavior.

    class Categories extends \Backend\Classes\Controller
    {
        [...]

        public function formExtendFields($form)
        {
            $form->addFields([...]);
        }
    }

The following methods are available on the $form object.

Method | Description
------------- | -------------
**addFields** | adds new fields to the outside area
**addTabFields** | adds new fields to the tabbed area
**addSecondaryTabFields** | adds new fields to the secondary tabbed area
**removeField** | remove a field from any areas

Each method takes an array of fields similar to the [form field configuration](#form-fields).

<a name="filter-form-fields"></a>
### 过滤表单字段

You can filter the form field definitions by overriding the `filterFields` method inside the Model used. This allows you to manipulate visibility and other field properties based on the model data. The method takes two arguments **$fields** will represent an object of the fields already defined by the [field configuration](#form-fields) and **$context** represents the active form context.

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

The above example will set the `hidden` flag on certain fields by checking the value of the Model attribute `source_type`. This logic will be applied when the form first loads and also when updated by a [defined field dependency](#field-dependencies).

<a name="validate-form-fields"></a>
## 验证表单字段

To validate the fields of your form you can make use of the [Validation](../database/traits#validation) trait in your model.
