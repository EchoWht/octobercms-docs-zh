# 后端列表

- [简介](＃介绍)
- [配置列表行为](#configuration-list)
    - [添加工具栏](#additional-toolbar)
    - [过滤列表](＃adding-filters)
- [定义列表列](＃list-columns)
    - [列选项](＃column-options)
- [可用列类型](＃column-types)
- [显示列表](#shisting-list)
- [多个列表定义](＃multiple-list-definitions)
- [使用列表过滤器](＃list-filters)
    - [范围选项](＃filter-scope-options)
    - [可用范围类型](＃范围类型)
- [扩展列表行为](#extend-list-behavior)
    - [覆盖控制器动作](＃覆盖动作)
    - [覆盖视图](＃覆盖视图)
    - [扩展列定义](#extend-list-columns)
    - [注入CSS class](#inject-row-class)
    - [扩展过滤器范围](#extend-filter-scopes)
    - [扩展模型查询](#extend-model-query)
    - [扩展记录集](#extend-records-collection)
    - [自定义列类型](＃custom-column-types)

<a name="introduction"></a>
## 简介

**列表行为**是一个控制器修饰符，用于轻松地将记录列表添加到页面。 该行为提供可排序和可搜索的列表，其列表中包含可选链接。 该行为提供了控制器操作`index`，但是列表可以在任何地方呈现，并且可以使用多个列表定义。

列表行为取决于列表[列定义](#list-columns)和[模型类](../database/model)。 为了使用列表行为，您应该将它添加到控制器类的`$implement`属性中。 此外，应定义`$listConfig`类属性，其值应引用用于配置行为选项的YAML文件。

    namespace Acme\Blog\Controllers;

    class Categories extends \Backend\Classes\Controller
    {
        public $implement = ['Backend.Behaviors.ListController'];

        public $listConfig = 'list_config.yaml';
    }

>**注意:**通常，列表和[表单行为](form)在同一个控制器中一起使用。

<a name="configuring-list"></a>
## 配置列表行为
   
`$listConfig`属性中引用的配置文件以YAML格式定义。 该文件应放入控制器的[views目录](controllers-views-ajax/＃introduction)。 以下是典型列表行为配置文件的示例:

    # ===================================
    #  List Behavior Config
    # ===================================

    title: Blog Posts
    list: ~/plugins/acme/blog/models/post/columns.yaml
    modelClass: Acme\Blog\Models\Post
    recordUrl: acme/blog/posts/update/:id

列表配置文件中需要以下字段:

字段 | 描述
------------- | -------------
**title**| 此列表的标题。
**list**| 配置数组或对列列定义文件的引用，请参阅[列表列](＃list-columns)。
**modelClass**| 模型类名称，列表数据从此模型加载。

下面列出的配置选项是可选的。

选项 | 描述
------------- | -------------
**filter**| 过滤器配置，请参阅[过滤列表](＃adds-filters)。
**recordUrl**| 将每个列表记录链接到另一个页面。例如:**users/update:id**。 `:id`部分被记录标识符替换。这允许您链接列表行为和[表单行为](form)。
**recordOnClick**| 单击记录时要执行的自定义JavaScript代码。
**noRecordsMessage**| 当没有找到记录时显示的消息，可以参考[本地化字符串](../plugin/localization)。
**recordsPerPage**| 每页显示的记录，没有页面使用0。默认值:0
**showPageNumbers**| 显示带分页的页码。在处理大型表时，禁用此选项可提高列表性能。默认值:true
**toolbar**| 引用Toolbar Widget配置文件或带配置的数组(参见下文)。
**showSorting**| 显示每列的排序链接。默认值:true
**defaultSort**| 在未定义用户首选项时设置默认排序列和方向。支持字符串或带有“column”和“direction”键的数组。
**showCheckboxes**| 显示每条记录旁边的复选框。默认值:false。
**showSetup**| 显示列表列设置按钮。默认值:false。
**showTree**| 显示父/子记录的树层次结构。默认值:false。
**treeExpanded**| 如果默认情况下应该扩展树节点。默认值:false。
**customViewPath**| 指定自定义视图路径以覆盖列表使用的部分，可选。

<a name="adding-toolbar"></a>
### 添加工具栏

要在列表中包含工具栏，请将以下配置添加到列表配置YAML文件中:

    toolbar:
        buttons: list_toolbar
        search:
            prompt: Find records

工具栏配置允许:

选项 | 描述
------------- | -------------
**buttons**| 使用工具栏按钮引用控制器部分文件。 例如:**_list_toolbar.htm**
**search**| 引用Search Widget配置文件或带配置的数组。

搜索配置支持以下选项:

选项 | 描述
------------- | -------------
**prompt**| 当没有活动搜索时显示的占位符，可以参考[本地化字符串](../插件/本地化)。
**mode**| 将搜索策略定义为包含所有单词，任何单词或精确短语。 支持的选项:all，any，exact。 默认值:全部。
**scope**| 指定在**列表模型**中定义的[查询范围方法](../database/model＃query-scopes)以应用于搜索查询，第一个参数将包含搜索项。
**searchOnEnter**| 将此设置为true将使搜索小部件在开始搜索之前等待按下Enter键(默认行为是，在有人在搜索字段中输入内容然后暂停一小段时间后，它会自动开始搜索)。 默认值:false。

上面提到的工具栏按钮部分应该包含带有一些按钮的工具栏控件定义。 部分还可以包含带有图表的[记分板控件](控制＃记分板)。 使用**New Post**按钮引用由[form behavior](表单)提供的**create**操作的工具栏部分示例:

    <div data-control="toolbar">
        <a
            href="<?= Backend::url('acme/blog/posts/create') ?>"
            class="btn btn-primary oc-icon-plus">New Post</a>
    </div>

<a name="adding-filters"></a>
### 过滤列表

要按用户定义的输入过滤列表，请将以下列表配置添加到YAML文件:

    filter: config_filter.yaml

**filter**选项应引用[filter configuration file](#list-filters)路径或提供带有配置的数组。

<a name="list-columns"></a>
## 定义列表列
   
列列是使用YAML文件定义的。 列配置使用列配置来创建记录表并在表格单元格中显示模型列。 该文件放在插件的**models**目录的子目录中。 子目录名称与以小写字母书写的模型类名称匹配。 文件名无关紧要，但**columns.yaml**和**list_columns.yaml**是常用名称。 示例列表列文件位置:

    plugins/
      acme/
        blog/
          models/                 <=== Plugin models directory
            post/                 <=== Model configuration directory
              list_columns.yaml    <=== Model list columns config file
            Post.php               <=== model class


下一个示例显示了列列定义文件的典型内容。

    # ===================================
    #  List Column Definitions
    # ===================================

    columns:
        name: Name
        email: Email

<a name="column-options"></a>
### 列选项

对于每列，可以指定这些选项(如果适用):

选项 | 描述
------------- | -------------
**label**| 向用户显示列表列时的名称。
**type**| 定义如何呈现此列(请参阅下面的[列类型](＃column-types))。
**default**| 如果value为空，则指定列的默认值。
**searchable**| 在列表搜索结果中包含此列。默认值:false。
**invisible**| 指定默认情况下是否隐藏此列。默认值:false。
**sortable**| 指定是否可以对此列进行排序。默认值:true。
**clickable**| 如果设置为false，则在单击列时禁用默认单击行为。默认值:true。
**select**| 定义用于该值的自定义SQL select语句。
**valueFrom**| 定义要用于值的模型属性。
**relation**| 定义模型关系列。
**useRelationCount**| 使用定义的`relation`的计数作为此列的值。默认值:false
**cssClass**| 将CSS类分配给列容器。
**width**| 设置列宽，可以百分比(10％)或像素(50px)指定。可以有一个没有指定宽度的列，它将被拉伸以占用可用空间。
**align**| 列对齐。可能的值是“left”，“right”和“center”。

<a name="column-types"></a>
## 可用的列类型
   
有许多列类型可用于**类型**设置，它们控制列表列的显示方式。 除了下面指定的本机列类型之外，您还可以[定义自定义列类型](#custom-column-types)。

- [Text](#column-text)
- [Number](#column-number)
- [Switch](#column-switch)
- [Date & Time](#column-datetime)
- [Date](#column-date)
- [Time](#column-time)
- [Time since](#column-timesince)
- [Time tense](#column-timetense)
- [Select](#column-select)
- [Relation](#column-relation)
- [Partial](#column-partial)
- [Colorpicker](#column-colorpicker)

<a name="column-text"></a>
### Text

`text`  - 显示一个左对齐的文本列

    full_name:
        label: Full Name
        type: text

<a name="column-number"></a>
### Number

`number`  - 显示一个数字列，右对齐

    age:
        label: Age
        type: number

<a name="column-switch"></a>
### Switch

`switch`  - 显示布尔列的开启或关闭状态。

    enabled:
        label: Enabled
        type: switch

<a name="column-datetime"></a>
### Date & Time

`datetime`  - 将列值显示为格式化的日期和时间。 下一个示例将日期显示为**Thu，1975年12月25日下午2:15**。

    created_at:
        label: Date
        type: datetime
        # Display datetime exactly as it is stored, ignores October's and the backend user's specified timezones.
        ignoreTimezone: true

您还可以指定自定义日期格式，例如**1975年12月25日星期四02:15:16 PM**:

    created_at:
        label: Date
        type: datetime
        format: l jS \of F Y h:i:s A

<a name="column-date"></a>
### Date

`date`  - 将列值显示为日期格式**M j,Y**

    created_at:
        label: Date
        type: date
        # Display datetime exactly as it is stored, ignores October's and the backend user's specified timezones.
        ignoreTimezone: true

<a name="column-time"></a>
### Time

`time`  - 将列值显示为时间格式**g:i A**

    created_at:
        label: Date
        type: time
        # Display datetime exactly as it is stored, ignores October's and the backend user's specified timezones.
        ignoreTimezone: true

<a name="column-timesince"></a>
### Time since

`timesince`  - 显示从值到当前时间的人类可读时间差。 例如:**10分钟前**

    created_at:
        label: Date
        type: timesince
        # Display datetime exactly as it is stored, ignores October's and the backend user's specified timezones.
        ignoreTimezone: true

<a name="column-timetense"></a>
### Time tense

`timetense`  - 使用当前日期的语法时态显示24小时时间和日期。 例如:**今天12:49**，**昨天4:00**或**2015年9月18日14:33**。

    created_at:
        label: Date
        type: timetense
        # Display datetime exactly as it is stored, ignores October's and the backend user's specified timezones.
        ignoreTimezone: true

<a name="column-select"></a>
### Select

`select`  - 允许使用自定义select语句创建列。 任何有效的SQL SELECT语句都适用于此处。

    full_name:
        label: Full Name
        select: concat(first_name, ' ', last_name)

<a name="column-relation"></a>
### Relation

`relation`  - 允许显示相关列，可以提供关系选项。 此选项的值必须是模型上Active Record [relationship](../database/relations)的名称。 在下一个示例中，**name**值将转换为相关模型中的name属性(例如:`$model-> name`)。

    group:
        label: Group
        relation: groups
        select: name

要显示显示相关记录数的列，请使用`useRelationCount`选项。

    users_count:
        label: Users
        relation: users
        useRelationCount: true
        
>**注意:**使用列上的`relation`选项会将`select`ed列中的值加载到此列指定的属性中。 建议您将显示关系数据的列命名为名称，而不与现有模型属性冲突，如以下示例所示:

**最佳实践:**

     group_name:
         label: Group
         relation: group
         select: name

**糟糕的做法:**

    # 这将覆盖$record-> group_id的值，这将破坏从列表视图访问关系
    group_id:
        label: Group
        relation: group
        select: name

<a name="column-partial"></a>
### Partial

`partial`  - 渲染partial，`path`值可以引用partial视图文件，否则列名称用作partial名称。 在partial内部，这些变量是可用的:`$value`是默认单元格值，`$record`是用于单元格的模型，`$column`是配置的类对象`Backend::Classes::ListColumn`。

    content:
        type: partial
        path: ~/plugins/acme/blog/models/comments/_content_column.htm
<a name="column-colorpicker"></a>
### Color Picker

`colorpicker`  - 显示colorpicker列的颜色

    color:
        label: Background
        type: colorpicker

<a name="displaying-list"></a>
## 显示列表

通常列表显示在索引[view](controllers-views-ajax/＃introduction)文件中。 由于列表包含工具栏，因此视图文件将仅包含单个`listRender`方法调用。

    <?= $this->listRender() ?>

<a name="multiple-list-definitions"></a>
## 多个列表定义

列表行为可以使用命名定义在同一控制器中支持多个列表。 `$listConfig`属性可以定义为一个数组，其中键是定义名称，值是配置文件。

    public $listConfig = [
        'templates' => 'config_templates_list.yaml',
        'layouts' => 'config_layouts_list.yaml'
    ];

然后，通过在调用`listRender`方法时将定义名称作为第一个参数传递，可以显示每个定义:

    <?= $this->listRender('templates') ?>

<a name="list-filters"></a>
## 使用列表过滤器

可以通过[添加过滤器定义](＃adds-filters)过滤列表配置来过滤列表。 类似地，过滤器由它们自己的包含过滤器范围的配置文件驱动，每个范围是可以过滤列表的方面。 下一个示例显示了过滤器定义文件的典型内容。

    # ===================================
    # Filter Scope Definitions
    # ===================================

    scopes:

        category:
            label: Category
            modelClass: Acme\Blog\Models\Category
            conditions: category_id in (:filtered)
            nameFrom: name

        status:
            label: Status
            type: group
            conditions: status in (:filtered)
            options:
                pending: Pending
                active: Active
                closed: Closed

        published:
            label: Hide published
            type: checkbox
            default: 1
            conditions: is_published <> true

        approved:
            label: Approved
            type: switch
            default: 2
            conditions:
                - is_approved <> true
                - is_approved = true

        created_at:
            label: Date
            type: date
            conditions: created_at >= ':filtered'


        published_at:
            label: Date
            type: daterange
            conditions: created_at >= ':after' AND created_at <= ':before'


<a name="filter-scope-options"></a>
### 范围选项

对于每个范围，您可以指定这些选项(如果适用):

选项 | 描述
------------- | -------------
**label**| 向用户显示过滤器范围时的名称。
**type**| 定义如何呈现此范围(请参阅下面的[范围类型](＃scope-types))。 默认值:组。
**conditions**| 指定要应用于列表模型查询的raw where查询语句，`:filtered`参数表示已过滤的值。
**scope**| 指定在**列表模型**中定义的[查询范围方法](../database/model＃query-scopes)以应用于列表查询，第一个参数将包含过滤的值。
**options**| 如果通过多个项过滤使用的选项，此选项可以在`modelClass`模型中指定数组或方法名称。
**nameFrom**| 如果按多个项过滤，则显示名称的属性，取自`modelClass`模型的所有记录。
**default**| 可以是整数(开关，复选框，数字)或数组(组，日期范围，数字范围)或字符串(日期)。

<a name="scope-types"></a>
### 可用范围类型

这些类型可用于确定如何显示过滤器范围。

- [Group](#filter-group)
- [Checkbox](#filter-checkbox)
- [Switch](#filter-switch)
- [Date](#filter-date)
- [Date range](#filter-daterange)
- [Number](#filter-number)
- [Number range](#filter-numberrange)
- [Text](#filter-text)

<a name="filter-group"></a>
### Group

`group` - 通过一组项过滤列表，通常是通过相关模型，并且需要`nameFrom`或`options`定义。 例如:状态名称为打开，关闭等。

    status:
        label: Status
        type: group
        conditions: status in (:filtered)
        default: 
            pending: Pending
            active: Active
        options:
            pending: Pending
            active: Active
            closed: Closed
            
<a name="filter-checkbox"></a>
### Checkbox

`checkbox` - 用作二进制复选框，以将预定义条件或查询应用于列表，可以打开也可以关闭。 使用0表示关闭，使用1表示打开默认值

    published:
        label: Hide published
        type: checkbox
        default: 1
        conditions: is_published <> true
        
<a name="filter-switch"></a>
### Switch

`switch` - 用作切换器在两个预定义条件或查询列表之间切换，不确定，打开或关闭。 使用0表示关闭，1表示不确定，2表示默认值


    approved:
        label: Approved
        type: switch
        default: 1
        conditions:
            - is_approved <> true
            - is_approved = true
            
<a name="filter-date"></a>
### Date

`date` - 显示要选择的单个日期的日期选择器。

    created_at:
        label: Date
        type: date
        minDate: '2001-01-23'
        maxDate: '2030-10-13'
        yearRange: 10
        conditions: created_at >= ':filtered'
        
<a name="filter-daterange"></a>
### Date Range

`daterange` - 显示要选择作为日期范围的两个日期的日期选择器。 条件参数以`:before`和`:after`传递。

    published_at:
        label: Date
        type: daterange
        minDate: '2001-01-23'
        maxDate: '2030-10-13'
        yearRange: 10
        conditions: created_at >= ':after' AND created_at <= ':before'
        
 
 使用日期和日期范围的默认值
 
 ```php
 myController::extendListFilterScopes(function($filter) {
                'Date Test' => [
                    'label' => 'Date Test',
                    'type' => 'daterange',
                    'default' => $this->myDefaultTime(),
                    'conditions' => "created_at >= ':after' AND created_at <= ':before'"
                ],
            ]);
        });
  
 //return value must be instance of carbon
  public function myDefaultTime() {
        return [
            0 => Carbon::parse('2012-02-02'),
            1 => Carbon::parse('2012-04-02'),
        ];
    }
 ```
        
<a name="filter-number"></a>
### Number

`number` - 显示要输入的单个数字的输入。

    age:
        label: Age
        type: number
        default: 14
        conditions: age >= ':filtered'
        
<a name="filter-numberrange"></a>
### Number Range

`numberrange` - 显示要输入的两个数字的输入作为数字范围。 条件参数以`:min`和`:max`传递。

    visitors:
        label: Visitor Count
        type: numberrange
        default: 
            0:10
            1:20
        conditions: vistors >= ':min' and visitors <= ':max'

<a name="filter-text"></a>
### Text

`text` - 显示要输入的字符串的文本输入。 您可以指定将在输入大小属性中注入的`size`属性(默认值:10)。

    username:
        label: Username
        type: text
        conditions: username = :value
        size: 2
            
<a name="extend-list-behavior"></a>
## 扩展列表行为

有时您可能希望修改默认列表行为，有几种方法可以执行此操作。

- [覆盖控制器方法](＃overriding-action)
- [覆盖视图](＃overriding-views)
- [扩展列定义](#extend-list-columns)
- [注入CSS行类](#inject-row-class)
- [扩展过滤器范围](#extend-filter-scopes)
- [扩展模型查询](#extend-model-query)
- [自定义列类型](＃custom-column-types)

<a name="overriding-action"></a>
### 覆盖控制器方法

您可以将自己的逻辑用于控制器中的`index`操作方法，然后可选地调用List behavior`index`父方法。

    public function index()
    {
       //
       //Do any custom code here
       //

       //Call the ListController behavior index() method
        $this->asExtension('ListController')->index();
    }

<a name="overriding-views"></a>
### 覆盖视图

`ListController`行为有一个主容器视图，您可以通过在控制器目录中创建名为`_list_container.htm`的特殊文件来覆盖它。 以下示例将向列表添加侧栏:

    <?php if ($toolbar): ?>
        <?= $toolbar->render() ?>
    <?php endif ?>

    <?php if ($filter): ?>
        <?= $filter->render() ?>
    <?php endif ?>

    <div class="row row-flush">
        <div class="col-sm-3">
            [Insert sidebar here]
        </div>
        <div class="col-sm-9 list-with-sidebar">
            <?= $list->render() ?>
        </div>
    </div>

该行为将调用`Lists`小部件，该小部件还包含您可以覆盖的众多视图。 这可以通过指定[列出配置选项](#configuration-list)中描述的`customViewPath`选项来实现。 窗口小部件将首先在此路径中查找视图，然后回退到默认位置。

    # Custom view path
    customViewPath: $/acme/blog/controllers/reviews/list

>**注意**: 最好使用子目录，例如`list`，以避免冲突。

例如，要修改列表正文行标记，请在控制器目录中创建名为`list/_list_body_row.htm`的文件。

    <tr>
        <?php foreach ($columns as $key => $column): ?>
            <td><?= $this->getColumnValue($record, $column) ?></td>
        <?php endforeach ?>
    </tr>

<a name="extend-list-columns"></a>
### 扩展列定义

您可以通过在控制器类上调用`extendListColumns`静态方法从外部扩展另一个控制器的列。 此方法可以采用两个参数，**$list**表示Lists小部件对象，**$model**表示列表使用的模型。 以此控制器为例:

    class Categories extends \Backend\Classes\Controller
    {
        public $implement = ['Backend.Behaviors.ListController'];

        public $listConfig = 'list_config.yaml';
    }

使用`extendListColumns`方法，您可以将额外的列添加到此控制器呈现的任何列表中。 检查**$model**的类型是否正确是个好主意。 这是一个例子:

        Categories::extendListColumns(function($list, $model) {

            if (!$model instanceof MyModel)
                return;

            $list->addColumns([
                'my_column' => [
                    'label' => 'My Column'
                ]
            ]);

        });

您还可以通过覆盖控制器类中的`listExtendColumns`方法在内部扩展列表列。

    class Categories extends \Backend\Classes\Controller
    {
        [...]

        public function listExtendColumns($list)
        {
            $list->addColumns([...]);
        }
    }

$list对象提供以下方法。

方法 | 描述
------------- | -------------
**addColumns**| 将新列添加到列表中
**removeColumn**| 从列表中删除列

每个方法都采用类似于[列列配置](＃list-columns)的列数组。

<a name="inject-row-class"></a>
### 注入CSS行 class

您可以通过在控制器类上添加`listInjectRowClass`方法来注入自定义css行类。 此方法可以采用两个参数，**$record**表示单个模型记录，**$definition**包含List窗口小部件定义的名称。 您可以返回包含行类的任何字符串值。 这些类将添加到行的HTML标记中。

    class Lessons extends \Backend\Classes\Controller
    {
        [...]

        public function listInjectRowClass($lesson, $definition)
        {
           //Strike through past lessons
            if ($lesson->lesson_date->lt(Carbon::today())) {
                return 'strike';
            }
        }
    }

<a name="extend-filter-scopes"></a>
### 扩展过滤范围

您可以通过在控制器类上调用`extendListFilterScopes`静态方法，从外部扩展另一个控制器的过滤器作用域。 此方法可以使用参数**$filter**来表示Filter小部件对象。 以此控制器为例:

        Categories::extendListFilterScopes(function($filter) {
            $filter->addScopes([
                'my_scope' => [
                    'label' => 'My Filter Scope'
                ]
            ]);
        });

> 提供的作用域数组类似于[列出过滤器配置](＃list-filters)。

您还可以在内部将过滤器范围扩展到控制器类，只需覆盖`listFilterExtendScopes`方法。

    class Categories extends \Backend\Classes\Controller
    {
        [...]

        public function listFilterExtendScopes($filter)
        {
            $filter->addScopes([...]);
        }
    }

$filter对象提供了以下方法。

方法 | 描述
------------- | -------------
**addScopes**| 添加新范围以过滤小部件
**removeScope**| 从过滤器小部件中删除范围

<a name="extend-model-query"></a>
### 扩展模型查询

可以通过覆盖控制器类中的`listExtendQuery`方法来扩展列表[数据库模型](../database/model)的查询查询。 此示例将通过将**withTrashed**作用域应用于查询来确保列表数据中包含软删除的记录:

    public function listExtendQuery($query)
    {
        $query->withTrashed();
    }

[列表过滤器](＃list-filters)模型查询也可以通过覆盖`listFilterExtendQuery`方法进行扩展:

    public function listFilterExtendQuery($query, $scope)
    {
        if ($scope->scopeName == 'status') {
            $query->where('status', '<>', 'all');
        }
    }
    
<a name="extend-records-collection"></a>
### 扩展记录集

可以通过覆盖控制器类中的`listExtendRecords`方法来扩展列表使用的记录集合。 此示例使用[记录集合](../database/collection)上的`sort`方法更改记录的排序顺序。

    public function listExtendRecords($records)
    {
        return $records->sort(function ($a, $b) {
            return $a->computedVal() > $b->computedVal();
        });
    }

<a name="custom-column-types"></a>
### 自定义列类型

自定义列表列类型可以使用[插件注册类](../plugin/registration#registration-methods)的“registerListColumnTypes”方法在后端注册。 该方法应返回一个数组，其中键是类型名称，值是可调用函数。 可调用函数接收三个参数，原生`$value`，`$column`定义对象和模型`$record`对象。

    public function registerListColumnTypes()
    {
        return [
           //A local method, i.e $this->evalUppercaseListColumn()
            'uppercase' => [$this, 'evalUppercaseListColumn'],

           //Using an inline closure
            'loveit' => function($value) { return 'I love '. $value; }
        ];
    }

    public function evalUppercaseListColumn($value, $column, $record)
    {
        return strtoupper($value);
    }

使用自定义列表列类型就像使用`type`选项按名称调用它一样简单。

    # ===================================
    #  List Column Definitions
    # ===================================

    columns:
        secret_code:
            label: Secret code
            type: uppercase
