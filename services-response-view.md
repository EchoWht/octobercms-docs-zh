# 视图和响应

- [基础响应](#basic-responses)
    - [将标题附加到响应](#attaching-headers-to-responses)
    - [将cookie附加到响应中](#attaching-cookies-to-responses)
- [其他响应类型](#other-response-types)
    - [视图响应](#view-responses)
    - [JSON响应](#json-responses)
    - [文件下载](#file-downloads)
- [重定向](#redirects)
    - [返回带有Flash数据的重定向](#redirect-flash-data)
    - [重定向到以前的URL](#redirecting-previous-url)
    - [重定向到当前页面](#redirecting-current-page)
- [响应宏](#response-macros)
- [视图](#views)

<a name="basic-responses"></a>
## 基础响应

可以从页面使用的几乎PHP方法返回响应。 这包括[布局执行生命周期](../cms/layouts#layout-life-cycle)和[AJAX处理程序定义](../ajax/handlers)中包含的所有CMS方法。

#### 从CMS方法返回字符串

从CMS页面返回字符串，布局或组件方法将暂停此过程并覆盖默认行为，因此它将显示“Hello World”字符串而不是显示页面。

    public function onStart()
    {
        return 'Hello World';
    }

#### 从AJAX处理程序返回字符串

从AJAX处理程序返回一个字符串将使用默认键`result`将该字符串添加到响应集合中。 请求的部分内容仍将包含在回复中。

    public function onDoSomething()
    {
        return 'Hello World';
        // ['result' => 'Hello World']
    }

#### 从路由返回字符串

从[路由定义](../services/router) 返回字符串将与CMS方法相同，并将字符串显示为响应。

    Route::get('/', function() {
        return 'Hello World';
    });

#### 创建自定义响应

对于更强大的解决方案，返回一个`Response`对象，提供构建HTTP响应的各种方法。 我们将在本文中进一步探讨此主题。

    $contents = 'Page not found';
    $statusCode = 404;
    return Response::make($contents, $statusCode);

<a name="attaching-headers-to-responses"></a>
### 将标题附加到响应

请记住，大多数响应方法都是可链接的，可以流畅地构建响应。 例如，您可以使用`header`方法向响应添加一系列标头，然后再将其发送回用户：

    return Response::make($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

一个实际的例子可能是返回XML响应：

    return Response::make($xmlString)->header('Content-Type', 'text/xml');

<a name="attaching-cookies-to-responses"></a>
### 将cookie附加到响应中

`withCookie`方法允许您轻松地将cookie附加到响应中。 例如，您可以使用withCookie方法生成cookie并将其附加到响应实例：

    return Response::make($content)->withCookie('name', 'value');

`withCookie`方法接受其他可选参数，允许您进一步自定义cookie的属性：

    ->withCookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

<a name="other-response-types"></a>
## 其他响应类型

`Response`facade可用于方便地生成其他类型的响应实例。

<a name="view-responses"></a>
### 视图响应

如果您需要访问`Response`类方法，但想要返回[view](#views)作为响应内容，为方便起见，您可以使用`Response::view`方法：

    return Response::view('acme.blog::hello')->header('Content-Type', $type);

<a name="json-responses"></a>
### JSON响应

`json`方法会自动将`Content-Type`头设置为application/json，并使用`json_encode` PHP函数将给定的数组转换为JSON：

    return Response::json(['name' => 'Steve', 'state' => 'CA']);

如果您想创建一个JSONP响应，除了`setCallback`之外，您可以使用`json`方法：

    return Response::json(['name' => 'Steve', 'state' => 'CA'])
        ->setCallback(Input::get('callback'));

<a name="file-downloads"></a>
### 文件下载

`download`方法可用于生成强制用户浏览器在给定路径下载文件的响应。 `download`方法接受文件名作为方法的第二个参数，这将确定下载文件的用户看到的文件名。 最后，您可以将HTTP标头数组作为方法的第三个参数传递：

    return Response::download($pathToFile);

    return Response::download($pathToFile, $name, $headers);

    return Response::download($pathToFile)->deleteFileAfterSend(true);

> **注意:** 管理文件下载的Symfony HttpFoundation要求下载的文件具有ASCII文件名。

<a name="redirects"></a>
## 重定向

重定向响应通常是`Illuminate\Http\RedirectResponse`类的实例，并包含将用户重定向到另一个URL所需的正确标头。 生成`RedirectResponse`实例的最简单方法是在`Redirect`外观上使用`to`方法。

    return Redirect::to('user/login');

<a name="redirect-flash-data"></a>
### 返回带有Flash数据的重定向

重定向到新URL和[向会话flash数据](../services/session)通常是同时完成的。 因此，为方便起见，您可以在单个方法链中创建一个`RedirectResponse`实例并将数据闪存到会话中：

    return Redirect::to('user/login')->with('message', 'Login Failed');

> **注意:** 由于`with`方法将数据\到会话，您可以使用典型的`Session::get`方法检索数据。

<a name="redirecting-previous-url"></a>
#### 重定向到以前的URL

您可能希望将用户重定向到之前的位置，例如，在表单提交后。 您可以使用`back`方法执行此操作：

    return Redirect::back();

    return Redirect::back()->withInput();

<a name="redirecting-current-page"></a>
#### 重定向到当前页面

有时您只想刷新当前页面，可以使用`refresh`方法执行此操作：

    return Redirect::refresh();

<a name="response-macros"></a>
## 响应宏

如果您想定义一个可以在各种路由和控制器中重用的自定义响应，可以使用`Response::macro`方法：

    Response::macro('caps', function($value) {
        return Response::make(strtoupper($value));
    });

`macro`函数接受一个名字作为它的第一个参数，一个Closure作为它的第二个参数。 当调用`Response`类上的宏名时，将执行宏的Closure：

    return Response::caps('foo');

您可以在[插件注册文件]的`boot`方法中定义您的宏(../plugin/registration#registration-methods)。 或者，插件可以在插件目录中提供名为**init.php**的文件，您可以使用该文件来放置宏注册。

<a name="views"></a>
## 视图

视图是存储基于系统的表示逻辑的好方法，例如API或端点使用的标记，或与CMS和后端区域共享的标记。 [邮件服务](../services/mail) 也使用视图来提供默认模板内容。 视图通常存储在插件的`views`目录中。

一个简单的视图可能看起来像这样：

    <!-- View stored in plugins/acme/blog/views/greeting.htm -->

    <html>
        <body>
            <h1>Hello, {{ name }}</h1>
        </body>
    </html>

也可以使用PHP模板通过使用`.php`扩展来解析视图：

    <!-- View stored in plugins/acme/blog/views/greeting.php -->

    <html>
        <body>
            <h1>Hello, <?php echo $name; ?></h1>
        </body>
    </html>

可以使用`View::make`方法将此视图返回给浏览器：

    return View::make('acme.blog::greeting', ['name' => 'Charlie']);

第一个参数是一个“路径提示”，它包含插件名称，由两个冒号`::`分隔，后跟视图文件名。 传递给`View::make`的第二个参数是应该可供视图使用的数据数组。

> **注意**: 路径提示区分大小写，插件名称应始终为小写。

#### 将数据传递给视图

    // 使用传统方法
    $view = View::make('acme.blog::greeting')->with('name', 'Steve');

    // 使用魔术方法
    $view = View::make('acme.blog::greeting')->withName('steve');

在上面的例子中，变量`name`可以从视图中访问，并且包含`Steve`。 如上所述，如果要传递数据数组，可以将第二个参数作为`make`方法：

    $view = View::make('acme.blog::greeting', $data);

还可以跨所有视图共享一段数据：

    View::share('name', 'Steve');

#### 将子视图传递给视图

有时您可能希望将视图传递到另一个视图。 例如，给定存储在`plugins/acme/blog/views/child/view.php`的子视图，我们可以将它传递给另一个视图，如下所示：

    $view = View::make('acme.blog::greeting')->nest('child', 'acme.blog::child.view');

    $view = View::make('acme.blog::greeting')->nest('child', 'acme.blog::child.view', $data);

然后可以从父视图呈现子视图：

    <html>
        <body>
            <h1>Hello!</h1>
            {{ child|raw }}
        </body>
    </html>

#### 确定视图是否存在

如果需要检查视图是否存在，请使用`View::exists`方法：

    if (View::exists('acme.blog::mail.customer')) {
        //
    }
