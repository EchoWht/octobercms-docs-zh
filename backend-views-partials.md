# 后端视图和Partials

- [Partials和提示](#partials)
    - [提示部分](#hints)
    - [检查提示是否隐藏](#checking-hints)
- [布局和子布局](#layouts)
    - [带侧边栏的表单](#layout-form-with-sidebar)

<a name="partials"></a>
## Partials

后端Partials是扩展名为**htm**的文件，它们位于[控制器视图](#introduction)目录中。 部分文件名应以下划线开头：*_partial.htm*。 可以从后端页面或其他部分呈现部分。 使用控制器的`makePartial`方法渲染部分。 该方法有两个参数 - 部分名称和可选的变量数组传递给partial。 例：

    <?= $this->makePartial('sidebar', ['showHeader' => true]) ?>

<a name="hints"></a>
### 提示部分

您可以在后端呈现信息面板，称为提示，用户可以隐藏。 第一个参数应该是唯一键，用于记住提示是否已隐藏。 第二个参数是对局部视图的引用。 除了一些提示属性之外，第三个参数可以是一些传递给partial的额外视图变量。

    <?= $this->makeHintPartial('my_hint_key', 'my_hint_partial', ['foo' => 'bar']) ?>

您还可以通过将键值设置为空值来禁用隐藏提示的功能。 此提示将始终显示：

    <?= $this->makeHintPartial(null, 'my_hint_partial') ?>

可以使用以下属性：

属性 | 描述
------------- | -------------
**type** | 设置提示的颜色，支持的类型：危险(danger)，信息(info)，成功(success)，警告(warning)。 默认值：信息(info)。
**title** | 在提示中添加标题部分。
**subtitle** | 除标题外，还在标题部分添加第二行。
**icon** | 除标题外，还会在标题部分添加一个图标。

<a name="checking-hints"></a>
### 检查提示是否隐藏

如果您正在使用提示，您可能会发现检查用户是否隐藏它们很有用。 这可以使用`isBackendHintHidden`方法轻松完成。 它需要一个参数，这是您在对`makeHintPartial`的原始调用中指定的唯一键。 如果提示被隐藏，则该方法将返回true，否则返回false：

    <?php if ($this->isBackendHintHidden('my_hint_key')): ?>
        <!-- 隐藏提示时执行某些操作 -->
    <?php endif ?>

<a name="layouts"></a>
## 布局和子布局

后端布局驻留在插件的可选 **layouts/** 目录中。 使用控制器对象的`$layout`属性设置自定义布局。 它默认为名为`default`的系统布局。

    /**
     * @var string 用于视图的布局
     */
    public $layout = 'mycustomlayout';

布局还提供了将自定义CSS类附加到BODY标记的选项。 这可以使用控制器的`$bodyClass`属性设置。

    /**
     * @var string Body CSS类添加到布局中
     */
    public $bodyClass = 'compact-container';

这些正文类可用于默认布局：

- **compact-container** - 这些正文类可用于默认布局：
- **slim-container** - 左右不使用填充。
- **breadcrumb-flush** - 告诉页面面包屑与下面的元素齐平。

<a name="layout-form-with-sidebar"></a>
### 带侧边栏的表格

布局也可以与局部相同的方式使用，更像是全局局部。 系统提供了一个名为`form-with-sidebar`的例子，并演示了一种实现子布局结构的新方法。

在使用此布局样式之前，请确保控制器通过在控制器的操作方法或构造函数中设置它来使用body类`compact-container`。

    $this->bodyClass = 'compact-container';

在使用此布局样式之前，请确保控制器通过在控制器的操作方法或构造函数中设置它来使用body类`compact-container`。

    <!-- 主要内容 -->
    <?php Block::put('form-contents') ?>
        Main content
    <?php Block::endPut() ?>

    <!-- 侧边栏 -->
    <?php Block::put('form-sidebar') ?>
        Side content
    <?php Block::endPut() ?>

    <!-- 布局 -->
    <?php Block::put('body') ?>
        <?= Form::open(['class'=>'layout stretch']) ?>
            <?= $this->makeLayout('form-with-sidebar') ?>
        <?= Form::close() ?>
    <?php Block::endPut() ?>

通过覆盖每个后端布局使用的**body**占位符，在最后一节中执行布局。 它用`<form />`HTML标签包装所有内容，并呈现名为 **form-with-sidebar**的子布局。 该文件位于`modules\backend\layouts\form-with-sidebar.htm`中。
