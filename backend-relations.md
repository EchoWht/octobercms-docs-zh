# 关系

- [介绍](#introduction)
- [配置关系行为](#configuring-relation)
- [关系类型](#relationship-types)
    - [对多](#has-many)
    - [属于很多](#belongs-to-many)
    - [属于很多(与数据透视数据)](#belongs-to-many-pivot)
    - [属于](#belongs-to)
    - [有一个](#has-one)
- [显示关系管理器](#relation-display)
- [扩展关系行为](#extend-relation-behavior)

<a name="introduction"></a>
## 介绍

`Relation behavior` 是一个控制器修饰符，用于轻松管理页面上的复杂[模型](../database/model) 关系。 不要与仅提供简单管理的 [List relation columns](lists#column-types) or [Form relation fields](forms#widget-relation) 混淆。

关系行为取决于[关系定义](#relation-definitions)。 为了使用关系行为，您应该将`Backend.Behaviors.RelationController`定义添加到控制器类的`$implement`字段中。 此外，应定义`$relationConfig`类属性，其值应引用用于[配置行为选项](#configuring-relation)的YAML文件。

    namespace Acme\Projects\Controllers;

    class Projects extends Controller
    {
        public $implement = [
            'Backend.Behaviors.FormController',
            'Backend.Behaviors.RelationController',
        ];

        public $formConfig = 'config_form.yaml';
        public $relationConfig = 'config_relation.yaml';
    }

>**注意:**关系行为通常与[表单行为](form)一起使用。

<a name="configuring-relation"></a>
## 配置关系行为

`$relationConfig`属性中引用的配置文件以YAML格式定义。 该文件应放入控制器的[views目录](controllers-views-ajax/#introduction)。 所需的配置取决于目标模型与相关模型之间的[关系类型](#relationship-types)。

关系配置文件中的第一级字段定义目标模型中的关系名称。 例如：

    class Invoice {
        public $hasMany = [
            'items' => ['Acme\Pay\Models\InvoiceItem'],
        ];
    }

具有称为`items`的关系的*Invoice*模型应使用相同的关系名称定义第一级字段：

    # ===================================
    #  Relation Behavior Config
    # ===================================

    items:
        label: Invoice Line Item
        view:
            list: $/acme/pay/models/invoiceitem/columns.yaml
            toolbarButtons: create|delete
        manage:
            form: $/acme/pay/models/invoiceitem/fields.yaml
            recordsPerPage: 10

然后，以下选项用于每个关系名称定义：

选项 | 描述
------------- | -------------
**label**| 需要一个关于单一时态的关系的标签。
**view**| 特定于视图容器的配置，请参见下文。
**manage**| 特定于管理弹出窗口的配置，请参见下文。
**pivot**| 表单字段定义文件的引用，用于[与数据透视表数据的关系](#belongs-to-many-pivot)。
**emptyMessage**| 关系为空时显示的消息，可选。
**readOnly**| 禁用添加，更新，删除或创建关系的功能。 默认值：false
**deferredBinding**| [可以使用会话密钥延迟所有绑定操作](../database/model#deferred-binding)。 默认值：false

可以为**view**或**manage**选项指定这些配置值，适用于列表，表单或两者的呈现类型。

选项 | 类型 | 描述
------------- | ------------- | -------------
**form**| Form | 对表单字段定义文件的引用，请参见[后端表单字段](forms#form-fields)。
**list**| List | 列表列定义文件的引用，请参见[后端列表列](lists#list-columns)。
**showSearch**| List | 显示用于搜索记录的输入。 默认值：false
**showSorting**| List | 显示每列的排序链接。 默认值：true
**defaultSort**| List | 在未定义用户首选项时设置默认排序列和方向。 支持字符串或带有“column”和“direction”键的数组。
**recordsPerPage**| List | 每页显示的最大行数。
**conditions**| List | 指定要应用于列表模型查询的raw where查询语句。
**scope**| List | 指定**相关表单模型**中定义的[查询范围方法](../database/model#query-scopes)以始终应用于列表查询。 此关系将附加到的模型(即**父模型**)作为第二个参数(`$query`是第一个参数)传递给此范围方法。

只能为**view**选项指定这些配置值。

选项 | 类型 | 描述
------------- | ------------- | -------------
**showCheckboxes**| List | 显示每条记录旁边的复选框。
**recordUrl**| List | 将每个列表记录链接到另一个页面。 例如：**users/update/:id**。 `：id`部分被记录标识符替换。
**recordOnClick**| List | 单击记录时要执行的自定义JavaScript代码。
**toolbarPartial**| Both | 使用工具栏按钮引用控制器部分文件。 例如：**_relation_toolbar.htm**。 此选项会覆盖*toolbarButtons*选项。
**toolbarButtons**| Both | 要显示的按钮组，可以是数组或管道分隔的字符串。 设置为“false”以显示没有按钮。 可用选项包括：添加，创建，更新，删除，删除，链接，取消链接。 例如：**添加\|删除**

只能为**manage**选项指定这些配置值。

选项 | 类型 | 描述
------------- | ------------- | -------------
**title**| Both | 一个弹出标题，可以参考[本地化字符串](../plugin/localization)。
**context**| Form | 正在显示的表单的上下文。 可以是带有键的字符串或数组：create，update。

<a name="relationship-types"></a>
## 关系类型

关系管理器的显示方式取决于目标模型中的关系定义。 关系类型也将确定配置要求，这些以**粗体**显示。 可以使用以下关系类型：

- [对多](#has-many)
- [属于很多](#belongs-to-many)
- [属于很多(with Pivot Data)](#belongs-to-many-pivot)
- [属于](#belongs-to)
- [有一个](#has-one)

<a name="has-many"></a>
### 对多

1. 相关记录显示为列表(**view.list**)。
1. 单击记录将显示更新表单(**manage.form**)。
1. 单击*添加*将显示选择列表(**manage.list**)。
1. 单击*创建*将显示创建表单(**manage.form**)。
1. 单击*删除*将销毁记录。
1. 单击*删除*将孤立关系。

例如，如果*Blog Post*有许多*Comments*，则将目标模型设置为博客帖子，并使用**list**定义中的列显示注释列表。 单击注释将打开一个弹出窗体，其中包含**形式**中定义的字段以更新注释。 可以以相同的方式创建注释。 以下是关系行为配置文件的示例：

    # ===================================
    #  Relation Behavior Config
    # ===================================

    comments:
        label: Comment
        manage:
            form: $/acme/blog/models/comment/fields.yaml
            list: $/acme/blog/models/comment/columns.yaml
        view:
            list: $/acme/blog/models/comment/columns.yaml
            toolbarButtons: create|delete

<a name="belongs-to-many"></a>
### Belongs to many

1. 相关记录显示为列表(**view.list**)。
1. 单击*添加*将显示选择列表(**manage.list**)。
1. 单击*创建*将显示创建表单(**manage.form**)。
1. 单击*删除*将销毁数据透视表记录。
1. 单击*删除*将孤立关系。

例如，如果*User*属于许多*Roles*，则将目标模型设置为用户，并使用**list**定义中的列显示角色列表。 可以向用户添加和删除现有角色。 以下是关系行为配置文件的示例：

    # ===================================
    #  Relation Behavior Config
    # ===================================

    roles:
        label: Role
        view:
            list: $/acme/user/models/role/columns.yaml
            toolbarButtons: add|remove
        manage:
            list: $/acme/user/models/role/columns.yaml
            form: $/acme/user/models/role/fields.yaml

<a name="belongs-to-many-pivot"></a>
### Belongs to many (with Pivot Data)

1. 相关记录显示为列表(**view.list**)。
1. 单击记录将显示更新表单(**pivot.form**)。
1. 单击*添加*将显示选择列表(**manage.list**)，然后显示数据输入表单(**pivot.form**)。
1. 单击*删除*将销毁数据透视表记录。

继续*Belongs To Many*关系中的示例，如果角色也带有到期日期，则单击角色将打开一个弹出窗体，其中包含**pivot**中定义的字段以更新到期日期。 以下是关系行为配置文件的示例：
    
    # ===================================
    #  Relation Behavior Config
    # ===================================

    roles:
        label: Role
        view:
            list: $/acme/user/models/role/columns.yaml
        manage:
            list: $/acme/user/models/role/columns.yaml
        pivot:
            form: $/acme/user/models/role/fields.yaml

通过`pivot`关系定义表单字段和列表列时，可以使用数据透视数据，请参阅下面的示例：

    # ===================================
    #  Relation Behavior Config
    # ===================================

    teams:
        label: Team
        view:
            list:
                columns:
                    name:
                        label: Name
                    pivot[team_color]:
                        label: Team color
        manage:
            list:
                columns:
                    name:
                        label: Name
        pivot:
            form:
                fields:
                    pivot[team_color]:
                        label: Team color

>**注意:**[deferred bindings](../database/relations#deferred-binding)目前不支持数据透视数据，因此父模型应该存在。

<a name="belongs-to"></a>
### Belongs to

1. 相关记录显示为预览表单(**view.form**)。
1. 单击*创建*将显示创建表单(**manage.form**)。
1. 单击*更新*将显示更新表单(**manage.form**)。
1. 单击*链接*将显示选择列表(**manage.list**)。
1. 单击*取消链接*将孤立关系。
1. 单击*删除*将销毁记录。

例如，如果*Phone*属于*Person*，则关系管理器将显示包含**形式**中定义的字段的表单。 单击“链接”按钮将显示要与电话关联的人员列表。 单击取消链接按钮将使电话与人员分离。

    # ===================================
    #  Relation Behavior Config
    # ===================================

    person:
        label: Person
        view:
            form: $/acme/user/models/person/fields.yaml
            toolbarButtons: link|unlink
        manage:
            form: $/acme/user/models/person/fields.yaml
            list: $/acme/user/models/person/columns.yaml

<a name="has-one"></a>
### Has one

1. 相关记录显示为预览表单(**view.form**)。
1. 单击*创建*将显示创建表单(**manage.form**)。
1. 单击*更新*将显示更新表单(**manage.form**)。
1. 单击*链接*将显示选择列表(**manage.list**)。
1. 单击*取消链接*将孤立关系。
1. 单击*删除*将销毁记录。

例如，如果*Person*有一个*Phone*，则关系管理器将显示表单，其中包含**格式**中为电话定义的字段。 单击“更新”按钮时，将显示一个弹出窗口，其中的字段现在可以编辑。 如果Person已有电话，则字段将更新，否则将为其创建新电话。

    # ===================================
    #  Relation Behavior Config
    # ===================================

    phone:
        label: Phone
        view:
            form: $/acme/user/models/phone/fields.yaml
            toolbarButtons: update|delete
        manage:
            form: $/acme/user/models/phone/fields.yaml
            list: $/acme/user/models/phone/columns.yaml

<a name="relation-display"></a>
## 显示关系管理器

在可以在任何页面上管理关系之前，必须首先通过调用`initRelation`方法在控制器中初始化目标模型。

    $post = Post::where('id', 7)->first();
    $this->initRelation($post);

>**注意:**[表单行为](forms)将在其创建，更新和预览操作上自动初始化模型。

然后可以通过调用`relationRender`方法显示关系管理器以获得指定的关系定义。 例如，如果要在[预览](forms#form-preview-view) 页面上显示关系管理器，**preview.htm**视图内容可能如下所示：

    <?= $this->formRenderPreview() ?>

    <?= $this->relationRender('comments') ?>

您可以通过将选项作为第二个参数传递，指示关系管理器以只读模式呈现：

    <?= $this->relationRender('comments', ['readOnly' => true]) ?>
    
<a name="extend-relation-behavior"></a>
## 扩展关系行为

有时您可能希望修改默认关系行为，有几种方法可以执行此操作。

- [扩展关系配置](#extend-relation-config)
- [扩展视图小部件](#extend-view-widget)
- [扩展管理小部件](#extend-manage-widget)
- [扩展枢轴小部件](#extend-pivot-widget)
- [扩展刷新结果](#extend-refresh-results)

<a name="extend-relation-config"></a>
### 扩展关系配置

提供操纵关系配置的机会。 以下示例可用于根据模型的属性注入不同的columns.yaml文件。 

    public function relationExtendConfig($config, $field, $model)
    {
       //Make sure the model and field matches those you want to manipulate
        if (!$model instanceof MyModel || $field != 'myField')
            return;

       //Show a different list for business customers
        if ($model->mode == 'b2b') {  
            $config->view['list'] = '$/author/plugin_name/models/mymodel/b2b_columns.yaml';
        }
    }
    
<a name="extend-view-widget"></a>
### 扩展视图小部件

提供操作视图窗口小部件的机会。
>**注意**：视图窗口小部件尚未完全初始化，因此并非所有公共方法都能按预期工作！ 有关更多信息，请参阅[如何删除列](#remove-column)。

例如，您可能希望根据模型的属性切换showCheckboxes。

    public function relationExtendViewWidget($widget, $field, $model)
    {
       //Make sure the model and field matches those you want to manipulate
        if (!$model instanceof MyModel || $field != 'myField')
            return;
            
        if ($model->constant) {
            $widget->showCheckboxes = false;
        }
    }

<a name="remove-column"></a>
#### 如何删除列

由于窗口小部件尚未在运行时周期的这一点完成初始化，因此无法调用$widget-> removeColumn()。 [ListController文档](/docs/backend/lists#extend-list-columns) 中描述的addColumns()方法将按预期工作，但要删除列，我们需要在其中监听`list.extendColumns`事件 relationExtendViewWidget()方法。 以下示例显示如何删除列：

    public function relationExtendViewWidget($widget, $field, $model)
    {               
       //Make sure the model and field matches those you want to manipulate
        if (!$model instanceof MyModel || $field != 'myField')
            return;
            
       //Will not work!
        $widget->removeColumn('my_column');
        
       //This will work
        $widget->bindEvent('list.extendColumns', function () use($widget) {
            $widget->removeColumn('my_column');
        });
    }
    
<a name="extend-manage-widget"></a>
### 扩展管理小部件
    
提供操纵关系的管理窗口小部件的机会。

    public function relationExtendManageWidget($widget, $field, $model)
    {
       //Make sure the field is the expected one
        if ($field != 'myField')
            return; 
            
       //manipulate widget as needed
    }

<a name="extend-pivot-widget"></a>
### 扩展枢轴小部件
    
提供操纵关系的透视小部件的机会。

    public function relationExtendPivotWidget($widget, $field, $model)
    {
       //Make sure the field is the expected one
        if ($field != 'myField')
            return; 
            
       //manipulate widget as needed
    }
    
<a name="extend-refresh-results"></a>
### 扩展刷新结果
    
当管理窗口小部件进行更改时，通常会刷新视图窗口小部件，您可以使用此方法在此过程发生时注入其他容器。 返回一个带有额外值的数组以发送到浏览器，例如：
    public function relationExtendRefreshResults($field)
    {
       //Make sure the field is the expected one
        if ($field != 'myField')
            return;
            
        return ['#myCounter' => 'Total records: 6'];
    }    
