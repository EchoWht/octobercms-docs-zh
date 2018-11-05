# 小工具

- [通用小部件](#generic-widgets)
    - [类定义](#generic-class-definition)
    - [AJAX处理程序](#generic-ajax-handlers)
    - [将小部件绑定到控制器](#generic-binding)
- [表单小部件](#form-widgets)
    - [类定义](#form-class-definition)
    - [表单小部件属性](#form-widget-properties)
    - [表单小部件注册](#form-widget-registration)
    - [加载表单数据](#form-widget-load-data)
    - [保存表单数据](#form-widget-save-data)
- [报告窗口小部件](#report-widgets)
    - [报告小部件类](#report-class-definition)
    - [报告小部件属性](#report-properties)
    - [报告小部件注册](#report-widget-registration)

窗口小部件是自包含的功能块，可以解决不同的任务。 窗口小部件始终具有用户界面和后端控制器（窗口小部件类），用于准备窗口小部件数据并处理窗口小部件用户界面生成的AJAX请求。

<a name="generic-widgets"></a>
## 通用小部件

窗口小部件是前端[组件](../cms/components)的后端等效项。 它们是相似的，因为它们是模块化的功能集，提供部分并使用别名命名。 关键区别在于后端窗口小部件使用YAML标记进行配置，并将自身绑定到后端页面。

窗口小部件类驻留在插件目录的**widgets**目录中。 目录名称与以小写字母书写的窗口小部件类的名称相匹配。 小部件可以提供资产和部分。 示例窗口小部件目录结构如下所示：

    widgets/
      /form
        /partials
          _form.htm     <=== Widget partial file
        /assets
          /js
            form.js     <=== Widget JavaScript file
          /css
            form.css    <=== Widget StyleSheet file
      Form.php          <=== Widget class

<a name="generic-class-definition"></a>
### 类定义

通用窗口小部件类必须扩展`Backend\Classes\WidgetBase`类。 与任何其他插件类一样，通用窗口小部件控制器应属于[插件命名空间](../plugin/registration#namespaces)。 示例小部件控制器类定义：

    <?php namespace Backend\Widgets;

    use Backend\Classes\WidgetBase;

    class Lists extends WidgetBase
    {
        /**
         * @var string A unique alias to identify this widget.
         */
        protected $defaultAlias = 'list';

        // ...
    }

窗口小部件类必须包含**render()**方法，用于通过呈现窗口小部件来生成窗口小部件标记。 例：

    public function render()
    {
        return $this->makePartial('list');
    }

要将变量传递给partials，您可以将它们添加到`$vars`属性中。

    public function render()
    {
        $this->vars['var'] = 'value';

        return $this->makePartial('list');
    }

或者，您可以将变量传递给makePartial()方法的第二个参数：

    public function render()
    {
        $this->vars['var'] = $value;

        return $this->makePartial('list', ['var' => 'value']);
    }

<a name="generic-ajax-handlers"></a>
### AJAX处理程序

窗口小部件实现与[后端控制器](controllers-views-ajax#ajax)相同的AJAX方法。 AJAX处理程序是窗口小部件类的公共方法，名称以**on**前缀开头。 小部件AJAX处理程序和后端控制器的AJAX处理程序之间的唯一区别是，当您在小部件部分中引用它时，您应该使用小部件的`getEventHandler`方法返回小部件的处理程序名称。

    <a
        href="javascript:;"
        data-request="<?= $this->getEventHandler('onPaginate') ?>"
        title="Next page">Next</a>

从widget类或部分调用时，AJAX处理程序将自己定位。 例如，如果窗口小部件使用**mywidget**的别名，则处理程序将以`mywidget::onName`为目标。 以上将输出以下属性值：

    data-request="mywidget::onPaginate"

<a name="generic-binding"></a>
### 将小部件绑定到控制器

在您可以开始在后端页面或部分页面中使用它之前，应将窗口小部件绑定到[后端控制器](controllers-views-ajax) 。 使用widget的`bindToController`方法将其绑定到控制器。 初始化窗口小部件的最佳位置是控制器的构造函数。 例：

    public function __construct()
    {
        parent::__construct();

        $myWidget = new MyWidgetClass($this);
        $myWidget->alias = 'myWidget';
        $myWidget->bindToController();
    }

绑定小部件后，您可以在控制器的视图中访问它，或者通过其别名来访问它：

    <?= $this->widget->myWidget->render() ?>

<a name="form-widgets"></a>
## 表单小部件

使用表单小部件，您可以向后端[表单](../backend/forms)添加新的控件类型。 它们提供了为模型提供数据所共有的功能。 表单小部件必须在[插件注册文件](../plugin/registration#registration-methods)中注册。

<a name="form-class-definition"></a>
### 类定义

表单窗口小部件类必须扩展`Backend\Classes\FormWidgetBase`类。 与任何其他插件类一样，通用窗口小部件控制器应属于[插件命名空间](../plugin/registration#namespaces)。 已注册的窗口小部件可用于后端[表单字段定义](../backend/forms#form-fields) 文件。 示例窗体小部件类定义：

    namespace Backend\Widgets;

    use Backend\Classes\FormWidgetBase;

    class CodeEditor extends FormWidgetBase
    {
        /**
         * @var string A unique alias to identify this widget.
         */
        protected $defaultAlias = 'codeeditor';

        public function render() {}
    }

<a name="form-widget-properties"></a>
### 表单小部件属性

表单小部件可能具有可以使用[表单字段配置](../backend/forms#form-fields)设置的属性。 只需在类上定义可配置属性，然后调用`fillFromConfig`方法在`init`方法定义中填充它们。

    class DatePicker extends FormWidgetBase
    {
        //
        // Configurable properties
        //

        /**
         * @var bool Display mode: datetime, date, time.
         */
        public $mode = 'datetime';

        /**
         * @var string the minimum/earliest date that can be selected.
         * eg: 2000-01-01
         */
        public $minDate = null;

        /**
         * @var string the maximum/latest date that can be selected.
         * eg: 2020-12-31
         */
        public $maxDate = null;

        //
        // Object properties
        //

        /**
         * {@inheritDoc}
         */
        protected $defaultAlias = 'datepicker';

        /**
         * {@inheritDoc}
         */
        public function init()
        {
            $this->fillFromConfig([
                'mode',
                'minDate',
                'maxDate',
            ]);
        }

        // ...
    }

然后，在使用窗口小部件时，可以从[窗体字段定义](../backend/forms#form-fields) 设置属性值。

    born_at:
        label: Date of Birth
        type: datepicker
        mode: date
        minDate: 1984-04-12
        maxDate: 2014-04-23

<a name="form-widget-registration"></a>
### 表单小部件注册

插件应该通过覆盖[Plugin注册类](../plugin/registration#registration-file)中的`registerFormWidgets`方法来注册表单小部件。 该方法返回一个数组，其中包含键中的窗口小部件类和窗口小部件短代码作为值。 例：

    public function registerFormWidgets()
    {
        return [
            'Backend\FormWidgets\CodeEditor' => 'codeeditor',
            'Backend\FormWidgets\RichEditor' => 'richeditor'
        ];
    }

短代码是可选的，可以在[表单字段定义](forms#field-widget)中引用窗口小部件时使用，它应该是一个唯一值，以避免与其他表单字段冲突。

<a name="form-widget-load-data"></a>
### 加载表单数据

表单小部件的主要目的是与您的模型交互，这意味着在大多数情况下通过数据库加载和保存值。 当表单窗口小部件呈现时，它将使用`getLoadValue`方法请求其存储的值。 `getId`和`getFieldName`方法将返回表单中使用的HTML元素的唯一标识符和名称。 这些值通常在渲染时传递给窗口小部件。

    public function render()
    {
        $this->vars['id'] = $this->getId();
        $this->vars['name'] = $this->getFieldName();
        $this->vars['value'] = $this->getLoadValue();

        return $this->makePartial('myformwidget');
    }

在基本级别，表单小部件可以使用输入元素发回用户输入值。 从上面的例子中，在**myformwidget** partial中，可以使用准备好的变量来呈现元素。

    <input id="<?= $id ?>" name="<?= $name ?>" value="<?= e($value) ?>" />

<a name="form-widget-save-data"></a>
### 保存表单数据

当需要用户输入并将其存储在数据库中时，表单小部件将在内部调用`getSaveValue`来请求该值。 要修改此行为，只需覆盖表单窗口小部件类中的方法。

    public function getSaveValue($value)
    {
         return $value;
    }

在某些情况下，您有意不希望给出任何值，例如，表单窗口小部件显示信息而不保存任何内容。 返回从`Backend\Classes\FormField`类派生的名为`FormField::NO_SAVE_DATA`的特殊常量，以忽略该值。

    public function getSaveValue($value)
    {
         return \Backend\Classes\FormField::NO_SAVE_DATA;
    }

<a name="report-widgets"></a>
## 报告窗口小部件

报告窗口小部件可用于后端仪表板和其他后端报告容器中。 报告窗口小部件必须在[插件注册文件](../plugin/registration#widget-registration)中注册。

<a name="report-class-definition"></a>
### 报告小部件类

报表小部件类应该扩展`Backend\Classes\ReportWidgetBase`类。 与任何其他插件类一样，通用窗口小部件控制器应属于[插件命名空间](../plugin/registration#namespaces)。 该类应覆盖`render`方法，以便呈现小部件本身。 与所有后端小部件类似，报告小部件使用部分和特殊目录布局。 示例目录布局：

    plugins/
      rainlab/                    <=== Author name
        googleanalytics/          <=== Plugin name
          reportwidgets/          <=== Report widgets directory
            trafficsources        <=== Widget files directory
              partials
                _widget.htm
            TrafficSources.php    <=== Widget class file

示例报告小部件类定义：

    namespace RainLab\GoogleAnalytics\ReportWidgets;

    use Backend\Classes\ReportWidgetBase;

    class TrafficSources extends ReportWidgetBase
    {
        public function render()
        {
            return $this->makePartial('widget');
        }
    }

窗口小部件可以包含要在窗口小部件中显示的任何HTML标记。 应使用**report-widget**类将标记包装到DIV元素中。 使用H3元素输出小部件标题是可取的。 示例小部件部分：

    <div class="report-widget">
        <h3>Traffic sources</h3>

        <div
            class="control-chart"
            data-control="chart-pie"
            data-size="200"
            data-center-text="180">
            <ul>
                <li>Direct <span>1000</span></li>
                <li>Social networks <span>800</span></li>
            </ul>
        </div>
    </div>

![image](https://raw.githubusercontent.com/octobercms/docs/master/images/traffic-sources.png)

在内部报表小部件中，您可以使用任何[图表或指针](controls)，列表或任何其他您想要的标记。 请记住，报表小部件扩展了通用后端小部件，您可以在报表小部件中使用任何小部件功能。 下一个示例显示了列表报告窗口小部件标记。

    <div class="report-widget">
        <h3>Top pages</h3>

        <div class="table-container">
            <table class="table data" data-provides="rowlink">
                <thead>
                    <tr>
                        <th><span>Page URL</span></th>
                        <th><span>Pageviews</span></th>
                        <th><span>% Pageviews</span></th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>/</td>
                        <td>90</td>
                        <td>
                            <div class="progress">
                                <div class="bar" style="90%"></div>
                                <a href="/">90%</a>
                            </div>
                        </td>
                    </tr>
                    <tr>
                        <td>/docs</td>
                        <td>10</td>
                        <td>
                            <div class="progress">
                                <div class="bar" style="10%"></div>
                                <a href="/docs">10%</a>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>

<a name="report-properties"></a>
### 报告小部件属性

报表小部件可能具有用户可以使用Inspector管理的属性：

![image](https://github.com/octobercms/docs/blob/master/images/report-widget-inspector.png?raw=true)

应该在widget类的`defineProperties`方法中定义属性。 这些属性在[文章组件](../plugin/components#component-properties)中描述。 例：

    public function defineProperties()
    {
        return [
            'title' => [
                'title'             => 'Widget title',
                'default'           => 'Top Pages',
                'type'              => 'string',
                'validationPattern' => '^.+$',
                'validationMessage' => 'The Widget Title is required.'
            ],
            'days' => [
                'title'             => 'Number of days to display data for',
                'default'           => '7',
                'type'              => 'string',
                'validationPattern' => '^[0-9]+$'
            ]
        ];
    }

<a name="report-widget-registration"></a>
### 报告小部件注册

插件可以通过覆盖[插件注册类](../plugin/registration#registration-file)中的`registerReportWidgets`方法来注册报表小部件。 该方法应返回一个数组，其中包含键中的窗口小部件类以及值中的窗口小部件标签和上下文。 例：

    public function registerReportWidgets()
    {
        return [
            'RainLab\GoogleAnalytics\ReportWidgets\TrafficOverview' => [
                'label'   => 'Google Analytics traffic overview',
                'context' => 'dashboard'
            ],
            'RainLab\GoogleAnalytics\ReportWidgets\TrafficSources' => [
                'label'   => 'Google Analytics traffic sources',
                'context' => 'dashboard'
            ]
        ];
    }

**label**元素定义Add Widget弹出窗口的小部件名称。 **context**元素定义了可以使用窗口小部件的上下文。 October的报告窗口小部件系统允许在任何页面上托管报告容器，容器上下文名称是唯一的。 仪表板页面上的窗口小部件容器使用**仪表板**上下文。
