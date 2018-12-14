# 后端排序和重新排序

- [介绍](#introduction)
- [配置重新排序行为](#configuring-reorder)
- [显示重新排序页面](#reorder-display)
- [扩展模型查询](#extend-model-query)

<a name="introduction"></a>
## 介绍

**重新排序行为**是一个控制器修饰符，它提供了对数据库记录进行排序和重新排序的功能。 该行为使用控制器操作`reorder`提供名为Reorder的页面。 此页面显示带有拖动句柄的记录列表，允许对它们进行排序，并在某些情况下进行重组。

行为取决于[model class](database-model.md) ，它必须实现以下[model traits](database-traits.md)之一：

1. `October\Rain\Database\Traits\Sortable`
1. `October\Rain\Database\Traits\NestedTree`

为了使用重新排序行为，您应该将它添加到控制器类的`$implement`属性中。 此外，应定义`$reorderConfig`类属性，其值应引用用于配置行为选项的YAML文件。

    namespace Acme\Shop\Controllers;

    class Categories extends Controller
    {
        public $implement = [
            'Backend.Behaviors.ReorderController',
        ];

        public $reorderConfig = 'config_reorder.yaml';

        // [...]
    }

<a name="configuring-reorder"></a>
## 配置重新排序行为

`$reorderConfig`属性中引用的配置文件以YAML格式定义。 该文件应放入控制器的[views目录](controllers-views-ajax/#introduction)。 以下是配置文件的示例：

	# ===================================
	#  Reorder Behavior Config
	# ===================================

	# Reorder Title
	title: Reorder Categories

	# Attribute name
	nameFrom: title

	# Model Class name
	modelClass: Acme\Shop\Models\Category

	# Toolbar widget configuration
	toolbar:
	    # Partial for toolbar buttons
	    buttons: reorder_toolbar


可以使用下面列出的配置选项。

选项 | 描述
------------- | -------------
**title** | 用于页面标题。
**nameFrom** | 指定应将哪个属性用作每个记录的标签。
**modelClass** | 模型类名称，记录数据从此模型加载。
**toolbar** | 引用Toolbar Widget配置文件或带配置的数组。

<a name="reorder-display"></a>
## 显示重新排序页面

您应该提供名为**reorder.htm**的[view file](controllers-views-ajax/#introduction) 。 此视图表示允许用户重新排序记录的“重新排序”页面。 由于重新排序包括工具栏，因此视图文件将仅包含单个`reorderRender`方法调用。

    <?= $this->reorderRender() ?>

<a name="extend-model-query"></a>
## 扩展模型查询

可以通过覆盖控制器类中的`reorderExtendQuery`方法来扩展列表[数据库模型](database-model.md) 的查询查询。 此示例将通过将**withTrashed**作用域应用于查询来确保列表数据中包含软删除的记录：

	public function reorderExtendQuery($query)
	{
	    $query->withTrashed();
	}
