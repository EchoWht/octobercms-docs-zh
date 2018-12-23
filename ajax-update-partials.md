# AJAX更新Partials

- [拉取Partials更新](#pulling-updates)
    - [更新定义](#update-definition)
- [部分更新](#pushing-updates)
- [将变量传递给partials](#passing-variables)

当处理程序执行时，它可以通过推或拉来准备在页面上更新的partial，这可以使用一些提供的变量进行渲染。

<a name="pulling-updates"></a>
## 拉取Partials更新

客户端浏览器可以在执行AJAX请求时请求从服务器更新partial，这被认为是*拉取内容更新*。以下代码在调用`onRefreshTime` [事件处理程序](ajax-handlers.md)后，在页面上的`#myDiv`元素内部呈现** mytime ** partial。

    <div id="myDiv">{% partial 'mytime' %}</div>

[data attributes API](ajax-attributes-api.md)使用`data-request-update`属性。

    <!-- 属性 API -->
    <button
        data-request="onRefreshTime"
        data-request-update="mytime: '#myDiv'">
        Go
    </button>

[JavaScript API](ajax-javascript-api.md)使用`update`配置选项：

    <!-- JavaScript API -->
    $.request('onRefreshTime', {
        update: { mytime: '#myDiv' }
    })

<a name="update-definition"></a>
### 更新定义

应更新内容的定义被指定为类似JSON的对象，其中：

- 左侧(key) **partial 名称**
- 右侧(value) **页面**

以下将要求使用** mypartial **内容更新`#myDiv`元素。

    mypartial: '#myDiv'

多个partial用逗号隔开。

    firstpartial: '#myDiv', secondpartial: '#otherDiv'

如果partial名称包含斜杠或破折号，那么在左边`引用`是很重要的。

    'folder/mypartial': '#myDiv', 'my-partial': '#myDiv'

目标元素将始终位于右侧，因为它也可以是JavaScript中的HTML元素。

    mypartial: document.getElementById('myDiv')

<a name="pushing-updates"></a>
## 部分更新

相比之下，[AJAX处理程序](ajax-handlers.md)可以*从服务器端将内容更新*推送到客户端浏览器。为了推送更新，处理程序返回一个数组，其中key是要更新的HTML元素(使用简单的CSS选择器)，value是要更新的内容。

以下示例将使用在partial** mypartial **中找到的内容更新页面id为 ** myDiv **的元素。 `onRefreshTime`处理程序调用`renderPartial`方法在PHP中呈现partial内容。

    function onRefreshTime()
    {
        return [
            '#myDiv' => $this->renderPartial('mypartial')
        ];
    }

> **注意:** 键名必须以标识符`#`或类`.`字符开头，以触发内容更新。(译者注：只用id或者class会不会扩展性不够高呢？)

<a name="passing-variables"></a>
## 将变量传递给partials

根据执行上下文，[AJAX事件处理程序](ajax-handlers.md)使变量可用于不同的partials。

- 在页面或布局[PHP部分](cms-themes.md#php-section)中使用`$this[]`。
- 在[组件类](plugin-components.md#ajax-handlers)中使用`$this-> page[]`。
- 在[后端区域](backend-controllers-ajax.md#ajax)中使用`$this-> vars[]`。

这些示例将为每个上下文的部分partials**结果**变量：

   //来自页面或布局PHP代码部分
    $this['result'] = 'Hello world!';

   //来自组件类
    $this->page['result'] = 'Hello world!';

   //来自后台控制器或者小部件
    $this->vars['result'] = 'Hello world!';

然后可以使用partials中的Twig访问其值：

    <!-- Hello world! -->
    {{ result }}
