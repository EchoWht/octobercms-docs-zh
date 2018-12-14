# 后端控件

- [记分牌](#scoreboards)
- [指标](#indicators)
- [饼形图](#pie-chart)
- [条形图](#bar-chart)

后端用户界面包含许多可在页面上使用的HTML控件。

<a name="scoreboards"></a>
## 记分牌

记分板控件通常显示在后端列表上方，并显示一些摘要或最重要的数据。 控件可以包含任何图表和指标（见下文）。 列表小部件上方显示的记分板控件标签示例：

    <div class="scoreboard">
        <div data-control="toolbar">
            <div class="scoreboard-item control-chart" data-control="chart-pie">
                <ul>
                    <li data-color="#95b753">Published <span>84</span></li>
                    <li data-color="#e5a91a">Drafts <span>12</span></li>
                    <li data-color="#cc3300">Deleted <span>18</span></li>
                </ul>
            </div>

            <div class="scoreboard-item control-chart" data-control="chart-bar">
                <ul>
                    <li data-color="#95b753">Published <span>84</span></li>
                    <li data-color="#e5a91a">Drafts <span>12</span></li>
                    <li data-color="#cc3300">Deleted <span>18</span></li>
                </ul>
            </div>

            <div class="scoreboard-item title-value">
                <h4>Weight</h4>
                <p>100</p>
                <p class="description">unit: kg</p>
            </div>
        </div>
    </div>

    <?= $this->listRender() ?>

![image](https://github.com/octobercms/docs/blob/master/images/list-scoreboard.png?raw=true) {.img-responsive .frame}

请注意，您应该为记分板元素使用**scoreboard-item**类。

<a name="indicators"></a>
## 指标

指标是简单的报告元素，具有标题，值和描述。 你可以在value元素上使用`positive`和`negative`类。 [Font Autumn](http://daftspunk.github.io/Font-Autumn/)图标类允许在值之前添加图标。

    <div class="scoreboard-item title-value">
        <h4>Weight</h4>
        <p>100</p>
        <p class="description">unit: kg</p>
    </div>

    <div class="scoreboard-item title-value">
        <h4>Comments</h4>
        <p class="positive">44</p>
        <p class="description">previous month: 32</p>
    </div>

    <div class="scoreboard-item title-value">
        <h4>Length</h4>
        <p class="negative">31</p>
        <p class="description">previous: 42</p>
    </div>

    <div class="scoreboard-item title-value">
        <h4>Latest commenter</h4>
        <p class="oc-icon-star">John Smith</p>
        <p class="description">registered: yes</p>
    </div>

    <div class="scoreboard-item title-value" data-control="goal-meter" data-value="88">
        <h4>goal meter</h4>
        <p>88%</p>
        <p class="description">37 posts remain</p>
    </div>

![image](https://github.com/octobercms/docs/blob/master/images/name-title-indicators.png?raw=true) {.img-responsive .frame}

> **注意:** 该示例在记分板区域的上下文中给出。 如果使用[报告窗口小部件]（窗口小部件#report-widgets）partial中的指示符，则不应使用类**scoreboard-item**。

<a name="pie-chart"></a>
## 饼形图

饼图以圆形图输出信息，可选标签位于中心。 示例标签：

    <div
        class="control-chart centered wrap-legend"
        data-control="chart-pie"
        data-size="200"
        data-center-text="100">
        <ul>
            <li>Label 1 <span>100</span></li>
            <li>Label 2 <span>100</span></li>
            <li>Label 3 <span>100</span></li>
        </ul>
    </div>

![image](https://github.com/octobercms/docs/blob/master/images/traffic-sources.png?raw=true) {.img-responsive .frame}

<a name="bar-chart"></a>
## 条形图

下一个示例显示条形图标记。 **wrap-legend**类是可选的，它管理图例布局。 **data-height**和**data-full-width**属性也是可选的。

    <div
        class="control-chart wrap-legend"
        data-control="chart-bar"
        data-height="100"
        data-full-width="1">
        <ul>
            <li>Label 1 <span>100</span></li>
            <li>Label 2 <span>100</span></li>
            <li>Label 3 <span>100</span></li>
        </ul>
    </div>

![image](https://github.com/octobercms/docs/blob/master/images/bar-chart.png?raw=true) {.img-responsive .frame}
