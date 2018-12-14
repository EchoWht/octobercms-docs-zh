# 后台导入和导出

- [介绍](#introduction)
- [配置行为](#configuring-import-export)
    - [导入页面](#import-page)
    - [导出页面](#export-page)
- [导入和导出视图](#import-export-views)
    - [导入视图](#import-view)
    - [导出视图](#export-view)
- [定义导入模型](#import-model)
- [定义导出模型](#export-model)
- [自定义选项](#custom-options)
- [与列表行为集成](#list-behavior-integration)

<a name="introduction"></a>
## 介绍

**导入导出行为** 是一个控制器修饰符，提供导入和导出数据的功能。 该行为提供了两个名为Import和Export的页面。 “导入”页面允许用户上传CSV文件并将列与数据库匹配。 “导出”页面相反，允许用户将数据库中的列作为CSV文件下载。 该行为提供了控制器动作`import()`和`export()`。

行为配置分为两部分，每部分依赖于特殊的模型类以及列表和表单字段定义文件。 要使用导入和导出行为，应将其添加到控制器类的`$implement`属性中。 此外，应定义`$importExportConfig`类属性，其值应引用用于配置行为选项的YAML文件。


    namespace Acme\Shop\Controllers;

    class Products extends Controller
    {
        public $implement = [
            'Backend.Behaviors.ImportExportController',
        ];

        public $importExportConfig = 'config_import_export.yaml';

        // [...]
    }

<a name="configuring-import-export"></a>
## 配置行为

`$importExportConfig`属性中引用的配置文件以YAML格式定义。 该文件应放入控制器的[views目录](controllers-views-ajax/#introduction)。 以下是配置文件的示例：

    # ===================================
    #  Import/Export Behavior Config
    # ===================================

    import:
        title: Import subscribers
        modelClass: Acme\Campaign\Models\SubscriberImport
        list: $/acme/campaign/models/subscriber/columns.yaml

    export:
        title: Export subscribers
        modelClass: Acme\Campaign\Models\SubscriberExport
        list: $/acme/campaign/models/subscriber/columns.yaml

下面列出的配置选项是可选的。 如果您希望行为支持 [Import](#import-page)或[Export](#export-page)或两者，请定义它们。

选项 | 描述
------------- | -------------
**defaultRedirect** | 在未定义特定重定向页面时用作回退重定向页面。
**import** | 配置数组或对“导入”页面的配置文件的引用。
**export** | 配置数组或对“导出”页面的配置文件的引用。

<a name="import-page"></a>
### 导入页面

要支持“导入”页面，请将以下配置添加到YAML文件中：

    import:
        title: Import subscribers
        modelClass: Acme\Campaign\Models\SubscriberImport
        list: $/acme/campaign/models/subscriberimport/columns.yaml
        redirect: acme/campaign/subscribers

“导入”页面支持以下配置选项：

选项 | 描述
------------- | -------------
**title** | 页面标题，可以参考[本地化字符串](../plugin/localization)。
**list** | 定义可用于导入的列表列。
**form** | 提供用作导入选项的其他字段，可选。
**redirect** | 导入完成时的重定向页面，可选
**permissions** | 执行操作所需的用户权限，可选

<a name="export-page"></a>
### 导出页面

要支持“导出”页面，请将以下配置添加到YAML文件中：

    export:
        title: Export subscribers
        modelClass: Acme\Campaign\Models\SubscriberExport
        list: $/acme/campaign/models/subscriberexport/columns.yaml
        redirect: acme/campaign/subscribers

“导出”页面支持以下配置选项：

选项 | 描述
------------- | -------------
**title** | 页面标题，可以参考[本地化字符串](../plugin/localization)。
**fileName** | 用于导出文件的文件名，默认为** export.csv **。
**list** | 定义可用于导出的列表列。
**form** | 提供用作导入选项的其他字段，可选。
**redirect** | 导出完成时的重定向页面，可选。
**useList** | 设置为true或列表定义的值以启用[与列表集成](#list-behavior-integration)，默认值：false。

<a name="import-export-views"></a>
## 导入和导出视图

对于每个页面功能[导入](#import-page)和[导出](#export-page)，您应该提供一个[视图文件](controllers-views-ajax/#introduction)和相应的名称 - **import.htm**和**export.htm**。

导入/导出行为向控制器类添加了两个方法：`importRender`和`exportRender`。 这些方法根据上述YAML配置文件呈现导入和导出部分。

<a name="import-view"></a>
### 导入视图

**import.htm**视图表示允许用户导入数据的导入页面。 典型的导入页面包含面包屑，导入部分本身和提交按钮。 ** data-request **属性应该引用行为提供的`onImport` AJAX处理程序。 以下是典型的import.htm视图文件的内容。

    <?= Form::open(['class' => 'layout']) ?>

        <div class="layout-row">
            <?= $this->importRender() ?>
        </div>

        <div class="form-buttons">
            <button
                type="submit"
                data-control="popup"
                data-handler="onImportLoadForm"
                data-keyboard="false"
                class="btn btn-primary">
                Import records
            </button>
        </div>

    <?= Form::close() ?>

<a name="export-view"></a>
### 导出视图

**export.htm**视图表示允许用户从数据库导出文件的“导出”页面。 典型的“导出”页面包含面包屑，导出部分本身和提交按钮。 **data-request**属性应该引用行为提供的`onExport` AJAX处理程序。 以下是典型的export.htm表单的内容。

    <?= Form::open(['class' => 'layout']) ?>

        <div class="layout-row">
            <?= $this->exportRender() ?>
        </div>

        <div class="form-buttons">
            <button
                type="submit"
                data-control="popup"
                data-handler="onExportLoadForm"
                data-keyboard="false"
                class="btn btn-primary">
                Export records
            </button>
        </div>

    <?= Form::close() ?>

<a name="import-model"></a>
## 定义导入模型

对于导入数据，您应该为此过程创建一个专用模型，该模型扩展了`Backend\Models\ImportModel`类。 这是一个示例类定义：

    class SubscriberImport extends \Backend\Models\ImportModel
    {
        /**
         * @var array 要应用于数据的规则。
         */
        public $rules = [];

        public function importData($results, $sessionKey = null)
        {
            foreach ($results as $row => $data) {

                try {
                    $subscriber = new Subscriber;
                    $subscriber->fill($data);
                    $subscriber->save();

                    $this->logCreated();
                }
                catch (\Exception $ex) {
                    $this->logError($row, $ex->getMessage());
                }

            }
        }
    }

该类必须定义一个名为`importData`的方法，用于处理导入的数据。 第一个参数`$results`将包含一个包含要导入的数据的数组。 第二个参数`$sessionKey`将包含用于请求的会话密钥。

方法 | 描述
------------- | -------------
`logUpdated()` | 更新记录时调用。（Called when a record is updated.）
`logCreated()` | 当创建记录时调用。（Called when a record is created.）
`logError(rowIndex, message)` | 导入记录时出现问题。
`logWarning(rowIndex, message)` | 用于提供软警告，例如修改值。
`logSkipped(rowIndex, message)` | 在未导入（跳过）整行数据时使用。

<a name="export-model"></a>
## 定义导出模型

要导出数据，您应该创建一个扩展`Backend\Models\ExportModel`类的专用模型。 这是一个例子：

    class SubscriberExport extends \Backend\Models\ExportModel
    {
        public function exportData($columns, $sessionKey = null)
        {
            $subscribers = Subscriber::all();
            $subscribers->each(function($subscriber) use ($columns) {
                $subscriber->addVisible($columns);
            });
            return $subscribers->toArray();
        }
    }

该类必须定义一个名为`exportData`的方法，用于返回导出数据。 第一个参数`$columns`是要导出的列名数组。 第二个参数`$sessionKey`将包含用于请求的会话密钥。

<a name="custom-options"></a>
## 自定义选项

导入和导出表单都支持可以使用表单字段引入的自定义选项，这些表单字段分别在导入或导出配置中的**表单**选项中定义。 然后将这些值传递给导入/导出模型，并在处理期间可用。

    import:
        [...]
        form: $/acme/campaign/models/subscriberimport/fields.yaml

    export:
        [...]
        form: $/acme/campaign/models/subscriberexport/fields.yaml

指定的表单字段将显示在导入/导出页面上。 这是一个示例`fields.yaml`文件内容：

    # ===================================
    #  Form Field Definitions
    # ===================================

    fields:

        auto_create_lists:
            label: Automatically create lists
            type: checkbox
            default: true

可以使用导入模型的`importData`方法中的`$this-> auto_create_lists`访问上面名为** auto_create_lists **的表单字段的值。 如果这是导出模型，则该值将在`exportData`方法中可用。

    class SubscriberImport extends \Backend\Models\ImportModel
    {
        public function importData($results, $sessionKey = null)
        {
            if ($this->auto_create_lists) {
                // Do something
            }

            [...]
        }
    }

<a name="list-behavior-integration"></a>
## 与列表行为集成

有一种替代方法可以导出使用[list行为](lists) 来提供导出数据的数据。 要使用此功能，您应该将`Backend.Behaviors.ListController`定义到控制器类的`$implement`字段。 您不需要使用导出视图，所有设置都将从列表中提取。 这是唯一需要的配置：

    export:
        useList: true

如果您使用[多个列表定义](lists#multiple-list-definitions)，那么您可以提供列表定义：

    export:
        useList: orders
        fileName: orders.csv

`useList`选项还支持扩展配置选项。


    export:
        useList:
            definition: orders
            raw: true

支持以下配置选项：

选项 | 描述
------------- | -------------
**definition** | 源记录的列表定义，可选。
**raw** | 列表定义来自源记录，optional.output来自记录的原始属性值，默认值：false。
