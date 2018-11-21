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

`$ importExportConfig`属性中引用的配置文件以YAML格式定义。 该文件应放入控制器的[views目录](controllers-views-ajax/#introduction)。 以下是配置文件的示例：

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

对于每个页面功能[导入](#import-page)和[导出](#export-page)，您应该提供一个[视图文件](controllers-views-ajax/＃introduction)和相应的名称 - **import.htm**和**export.htm**。

The import/export behavior adds two methods to the controller class: `importRender` and `exportRender`. These methods render the importing and exporting sections as per the YAML configuration file described above.

<a name="import-view"></a>
### Import view

The **import.htm** view represents the Import page that allows users to import data. A typical Import page contains breadcrumbs, the import section itself, and the submission buttons. The **data-request** attribute should refer to the `onImport` AJAX handler provided by the behavior. Below is a contents of the typical import.htm view file.

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
### Export view

The **export.htm** view represents the Export page that allows users to export a file from the database. A typical Export page contains breadcrumbs, the export section itself, and the submission buttons. The **data-request** attribute should refer to the `onExport` AJAX handler provided by the behavior. Below is a contents of the typical export.htm form.

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
## Defining an import model

For importing data you should create a dedicated model for this process which extends the `Backend\Models\ImportModel` class. Here is an example class definition:

    class SubscriberImport extends \Backend\Models\ImportModel
    {
        /**
         * @var array The rules to be applied to the data.
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

The class must define a method called `importData` used for processing the imported data. The first parameter `$results` will contain an array containing the data to import. The second parameter `$sessionKey` will contain the session key used for the request.

Method | Description
------------- | -------------
`logUpdated()` | Called when a record is updated.
`logCreated()` | Called when a record is created.
`logError(rowIndex, message)` | Called when there is a problem with importing the record.
`logWarning(rowIndex, message)` | Used to provide a soft warning, like modifying a value.
`logSkipped(rowIndex, message)` | Used when the entire row of data was not imported (skipped).

<a name="export-model"></a>
## Defining an export model

For exporting data you should create a dedicated model which extends the `Backend\Models\ExportModel` class. Here is an example:

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

The class must define a method called `exportData` used for returning the export data. The first parameter `$columns` is an array of column names to export. The second parameter `$sessionKey` will contain the session key used for the request.

<a name="custom-options"></a>
## Custom options

Both import and export forms support custom options that can be introduced using form fields, defined in the **form** option in the import or export configuration respectively. These values are then passed to the Import / Export model and are available during processing.

    import:
        [...]
        form: $/acme/campaign/models/subscriberimport/fields.yaml

    export:
        [...]
        form: $/acme/campaign/models/subscriberexport/fields.yaml

The form fields specified will appear on the import/export page. Here is an example `fields.yaml` file contents:

    # ===================================
    #  Form Field Definitions
    # ===================================

    fields:

        auto_create_lists:
            label: Automatically create lists
            type: checkbox
            default: true

The value of the form field above called **auto_create_lists** can be accessed using `$this->auto_create_lists` inside the `importData` method of the import model. If this were the export model, the value would be available inside the `exportData` method instead.

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
## Integration with list behavior

There is an alternative approach to exporting data that uses the [list behavior](lists) to provide the export data. In order to use this feature you should have the `Backend.Behaviors.ListController` definition to the `$implement` field of the controller class. You do not need to use an export view and all the settings will be pulled from the list. Here is the only configuration needed:

    export:
        useList: true

If you are using [multiple list definitions](lists#multiple-list-definitions), then you can supply the list definition:

    export:
        useList: orders
        fileName: orders.csv

The `useList` option also supports extended configuration options.


    export:
        useList:
            definition: orders
            raw: true

The following configuration options are supported:

Option | Description
------------- | -------------
**definition** | the list definition to source records from, optional.
**raw** | output the raw attribute values from the record, default: false.
