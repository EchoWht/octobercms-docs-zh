# 路由服务

- [基本路由](#basic-routing)
- [路由参数](#route-parameters)
    - [必需参数](#required-parameters)
    - [可选参数](#parameters-optional-parameters)
    - [正则表达式约束](#parameters-regular-expression-constraints)
- [路由命名](#named-routes)
- [路由组](#route-groups)
    - [子域名路由](#route-group-sub-domain-routing)
    - [路由前缀](#route-group-prefixes)
    - [路由中间件](#route-middleware)
- [抛出404错误](#throwing-404-errors)

<a name="basic-routing"></a>
## 基本路由

虽然[后端控制器](../backend/controllers-views-ajax)的路由是自动处理的，CMS页面在[页面配置](cms-pages.md#configuration)中定义了自己的URL路由，路由器service主要用于定义固定API和端点。 

您可以通过在与[插件注册文件](plugin-registration.md)相同的目录中创建名为**routes.php**的文件来定义这些路由。 最基本的路由只接受URI和`Closure`：

    Route::get('/', function () {
        return 'Hello World';
    });

    Route::post('foo/bar', function () {
        return 'Hello World';
    });

    Route::put('foo/bar', function () {
        //
    });

    Route::delete('foo/bar', function () {
        //
    });

#### 注册一个多动作的路由

有时您可能需要注册响应多个HTTP动作的路由。 您可以使用`Route`facade上的`match`方法执行此操作：

    Route::match(['get', 'post'], '/', function () {
        return 'Hello World';
    });

您也可以使用`any`方法注册响应所有HTTP动作的路由：

    Route::any('foo', function () {
        return 'Hello World';
    });

#### 生成路由的URL

您可以使用`Url`facade为您的路由生成网址：

    $url = Url::to('foo');

<a name="route-parameters"></a>
## 路由参数

<a name="required-parameters"></a>
### 必需参数

有时您需要捕获路由中的URI段，例如，您可能需要从URL获取用户的ID。 您可以通过定义路由参数来执行此操作：

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

您可以根据路由的要求定义任意数量的路由参数：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

路由参数始终包含在单个*大括号*内。 执行路由时，参数将传递到路由的“Closure”中。

> **注意:** 路由参数不能包含`-`字符。 使用下划线(`_`)代替。

<a name="parameters-optional-parameters"></a>
### 可选参数

有时您可能需要指定路由参数，但可以选择存在该路由参数。 您可以在参数名称后面放置一个“？”标记：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 正则表达式约束

您可以使用路由实例上的`where`方法约束路由参数的格式。 `where`方法接受参数的名称和定义参数应该如何约束的正则表达式：

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="named-routes"></a>
## 命名路由

命名路由允许您方便地为特定路由生成URL或重定向。 在定义路由时，可以使用`as`数组键为路由指定名称：

    Route::get('user/profile', ['as' => 'profile', function () {
        //
    }]);

#### 路由组和命名路由

如果使用[路由组](#route-groups)，则可以在路由组属性数组中指定`as`关键字，允许您为组内的所有路由设置公共路由名称前缀：

    Route::group(['as' => 'admin::'], function () {
        Route::get('dashboard', ['as' => 'dashboard', function () {
            // Route named "admin::dashboard"
        }]);
    });

#### 生成指向路由的URL

为给定路由指定名称后，您可以在生成URL时使用路由名称，或通过`Url::route`方法重定向：

    $url = Url::route('profile');

    $redirect = Response::redirect()->route('profile');

如果路由定义参数，您可以将参数作为第二个参数传递给`route`方法。 给定的参数将自动插入到URL中：

    Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
        //
    }]);

    $url = Url::route('profile', ['id' => 1]);

<a name="route-groups"></a>
## 路由组

路由组允许您跨大量路由共享路由属性，而无需在每条路由上定义这些属性。 共享属性以数组格式指定为`Route::group`方法的第一个参数。

<a name="route-group-sub-domain-routing"></a>
### 子域名路由

路由组也可用于路由通配符子域。 可以像路由URI一样为子域分配路由参数，允许您捕获子域的一部分以供路由或控制器使用。 可以使用组属性数组上的`domain`键指定子域：

    Route::group(['domain' => '{account}.example.com'], function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### 路由前缀

“prefix”组数组属性可用于为组中的每个路由添加给定URI。 例如，您可能希望使用`admin`为组内的所有路由URI添加前缀：

    Route::group(['prefix' => 'admin'], function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

您还可以使用`prefix`参数指定分组路由的公共参数：

    Route::group(['prefix' => 'accounts/{account_id}'], function () {
        Route::get('detail', function ($account_id) {
            // Matches The accounts/{account_id}/detail URL
        });
    });

<a name="route-middleware"></a>
### 路由中间件

在插件的`boot()`方法中注册中间件将为每个请求全局注册它。
如果你想一次将中间件注册到一个路由，你应该这样做：

    Route::get('info', 'Acme\News@info')->middleware('Path\To\Your\Middleware');

对于路由组，可以这样做：

    Route::group(['middleware' => 'Path\To\Your\Middleware'], function() {
        Route::get('info', 'Acme\News@info');
    });

最后，如果您想将一组中间件分配给一条路由，您可以这样做

    Route::middleware(['Path\To\Your\Middleware'])->group(function() {
        Route::get('info', 'Acme\News@info');
    });

您当然可以在一个组中添加多个中间件，在上面的示例中只使用一个用于方便。

<a name="throwing-404-errors"></a>
## 抛出404错误

有两种方法可以手动触发路径中的404错误。 首先，您可以使用`abort`帮助器。 `abort`帮助器只是抛出一个带有指定状态代码的`Symfony\Component\HttpFoundation\Exception\HttpException`：

    App::abort(404);

其次，您可以手动抛出`Symfony\Component\HttpKernel\Exception\NotFoundHttpException`的实例。

有关处理404异常和对这些错误使用自定义响应的更多信息，请参阅文档的[错误和日志](services-error-log.md) 部分。