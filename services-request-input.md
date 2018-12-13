# 请求和输入

- [基本输入](#basic-input)
- [Cookies](#cookies)
- [Old input](#old-input)
- [文件](#files)
- [请求信息](#request-information)

<a name="basic-input"></a>
## 基本输入

您可以使用一些简单的方法访问所有用户输入。 使用`Input`facade时，您无需担心请求的HTTP谓词，因为所有谓词都以相同的方式访问输入。 全局`input` [helper function](../services/helpers)是`Input::get`的别名。

#### 检索输入值

    $name = Input::get('name');

#### 如果输入值不存在，则检索默认值

    $name = Input::get('name', 'Sally');

#### 确定是否存在输入值

    if (Input::has('name')) {
        //
    }

#### 获取请求的所有输入

    $input = Input::all();

#### 仅获取一些请求输入

    $input = Input::only('username', 'password');

    $input = Input::except('credit_card');

处理带有“数组”输入的表单时，可以使用点表示法来访问数组：

    $input = Input::get('products.0.name');

> **注意:** 一些JavaScript库（如Backbone）可以将输入作为JSON发送到应用程序。 您可以通过`Input::get`访问此数据，就像正常一样。

<a name="cookies"></a>
## Cookies

默认情况下，October创建的所有Cookie都会加密并使用身份验证代码进行签名，这意味着如果客户端更改了这些Cookie，则会将其视为无效。 在`cookie.unencryptedCookies`配置密钥中命名的cookie将不会被加密。

> **注意:** Cookie使用APP_KEY加密，因此如果客户知道APP_KEY，则可能会由客户制作Cookie。 如果您的应用程序的加密密钥掌握在恶意方的手中，则该方可以使用加密密钥创建cookie值，并利用继承到PHP对象序列化/反序列化的漏洞，例如在应用程序中调用任意类方法。 为了缓解这种情况，如果您怀疑它已被泄露，请务必轮换您的APP_KEY，并确保始终确认您从cookie中收到的数据是在您使用它之前所预期的。

#### 检索cookie值

    $value = Cookie::get('name');

#### 将新cookie附加到响应

    $response = Response::make('Hello World');

    $response->withCookie(Cookie::make('name', 'value', $minutes));

#### 为下一个响应排队cookie

如果您想在创建响应之前设置cookie，请使用`Cookie::queue`方法。 cookie将自动附加到您的应用程序的最终响应中。

    Cookie::queue($name, $value, $minutes);

#### 创建一个永远持续的cookie

    $cookie = Cookie::forever('name', 'value');

#### 处理没有加密的cookie

如果您不希望某些cookie被加密或解密，您可以在配置中指定它们。
例如，当您想要通过cookie将数据从前端传递到服务器端后端时，这很有用，反之亦然。

将不应加密或解密的cookie的名称添加到`config/cookie.php`配置文件中的`unencryptedCookies`参数。

    /*
    |--------------------------------------------------------------------------
    | 没有加密的Cookies
    |--------------------------------------------------------------------------
    |
    | OctoberCMS默认加密/解密cookie。 您可以指定cookie
    | 这里不应该加密或解密。 这很有用
    | 例如，当您想要将数据从前端传递到服务器端后端时
    | 通过cookie，反之亦然。
    |
    */

    'unencryptedCookies' => [
        'my_cookie',
    ],

或者对于插件，您也可以从插件的`Plugin.php`动态添加这些插件。

    public function boot()
    {
        Config::push('cookie.unencryptedCookies', "my_cookie");
    }

<a name="old-input"></a>
## 老得input数据

您可能需要保留一个请求的输入，直到下一个请求。 例如，在检查表单是否存在验证错误后，您可能需要重新填充表单。

####flash 输入到会话

    Input::flash();

#### 仅flash会话的一些输入

    Input::flashOnly('username', 'email');

    Input::flashExcept('password');

由于您经常需要将flash输入与重定向到上一页相关联，因此您可以轻松地将输入闪烁链接到重定向。

    return Redirect::to('form')->withInput();

    return Redirect::to('form')->withInput(Input::except('password'));

> **注意:** 您可以使用[Session](../services/session)类在请求之间刷新其他数据。

#### 检索旧数据

    Input::old('username');

<a name="files"></a>
## 文件

#### 检索上传的文件

    $file = Input::file('photo');

#### 确定文件是否已上传

    if (Input::hasFile('photo')) {
        //
    }

`file`方法返回的对象是`Symfony\Component\HttpFoundation\File\UploadedFile`类的一个实例，它扩展了PHP`SplFileInfo`类，并提供了各种与文件交互的方法。

#### 确定上传的文件是否有效

    if (Input::file('photo')->isValid()) {
        //
    }

#### 移动上传的文件

    Input::file('photo')->move($destinationPath);

    Input::file('photo')->move($destinationPath, $fileName);

#### 检索上传文件的路径

    $path = Input::file('photo')->getRealPath();

#### 检索上传文件的原始名称

    $name = Input::file('photo')->getClientOriginalName();

#### 检索上传文件的扩展名

    $extension = Input::file('photo')->getClientOriginalExtension();

#### 检索上传文件的大小

    $size = Input::file('photo')->getSize();

#### 检索上传文件的MIME类型

    $mime = Input::file('photo')->getMimeType();

<a name="request-information"></a>
## 请求信息

`Request`类提供了许多方法来检查应用程序的HTTP请求，并扩展了`Symfony\Component\HttpFoundation\Request`类。 这儿是一些精彩片段。

#### 检索请求URI

    $uri = Request::path();

#### 检索请求方法

    $method = Request::method();

    if (Request::isMethod('post')) {
        //
    }

#### 确定请求路径是否与模式匹配

    if (Request::is('admin/*')) {
        //
    }

#### 获取请求URL

    $url = Request::url();

#### 检索请求URI段

    $segment = Request::segment(1);

#### 检索请求标头

    $value = Request::header('Content-Type');

#### 从$_SERVER中检索值

    $value = Request::server('PATH_INFO');

#### 确定请求是否通过HTTPS

    if (Request::secure()) {
        //
    }

#### 确定请求是否使用AJAX

    if (Request::ajax()) {
        //
    }

#### 确定请求是否具有JSON内容类型

    if (Request::isJson()) {
        //
    }

#### 确定请求是否要求JSON

    if (Request::wantsJson()) {
        //
    }

#### 检查请求的响应格式

`Request::format`方法将根据HTTP Accept标头返回请求的响应格式：

    if (Request::format() == 'json') {
        //
    }
