# 错误和日志

- [介绍](#introduction)
- [配置](#configuration)
- [异常类型](#exception-types)
    - [应用异常](#application-exception)
    - [系统异常](#system-exception)
    - [验证异常](#validation-exception)
    - [AJAX异常](#ajax-exception)
- [异常处理](#exception-handling)
- [HTTP异常](#http-exceptions)
    - [自定义错误页面](#custom-error-page)
- [日志](#logging)
    - [帮助函数](#helpers)

<a name="introduction"></a>
## 介绍

首次开始使用OctoberCMS时，已经为您配置了错误和异常处理。 可以通过两种方式访问事件日志：

1. 可以通过打开文件`storage/logs/system.log`在文件系统中查看事件日志。
1. 或者，可以通过后台目录 *系统>日志>事件日志* ，通过管理区域查看。

当前端页面显示出错误页面和一些[异常类型](#exception-types)时会产生错误日志

<a name="configuration"></a>
## 配置

#### 错误细节

应用程序通过浏览器显示的错误详细信息量由`config/app.php`配置文件中的`debug`配置选项控制。 默认情况下，详细的错误报告会*打开*，因此查看详细的错误信息会很有帮助，这些信息对调试和故障排除问题很有用。 关闭此功能时，如果页面出现问题，将显示一般错误消息。

对于本地开发，您应该将`debug`值设置为`true`。 在生产环境中，此值应始终为“false”。

    /*
    |--------------------------------------------------------------------------
    | Application Debug Mode
    |--------------------------------------------------------------------------
    |
    | When your application is in debug mode, detailed error messages with
    | stack traces will be shown on every error that occurs within your
    | application. If disabled, a simple generic error page is shown.
    |
    */

    'debug' => false,

#### 日志文件模式

十月支持`single`，`daily`，`syslog`和`errorlog`日志记录模式。 例如，如果您希望使用每日的日志替代日志文件，则只需在`config/app.php`配置文件中设置`log`值：

    'log' => 'daily'

<a name="exception-types"></a>
## 可用的异常

October开始提供几种基本的异常类型。

<a name="application-exception"></a>
### 应用异常

作为`ApplicationException`别名的`October\Rain\Exception\ApplicationException`类是在简单应用程序条件失败时使用的最常见的异常类型。

    throw new ApplicationException('You must be logged in to do that!');

错误消息将被简化，并且永远不会包含任何敏感信息，如php文件和行号。

<a name="system-exception"></a>
### 系统异常

`October\Rain\Exception\SystemException`类，别名为`SystemException`，用于对系统运行至关重要且始终记录的错误。

    throw new SystemException('Unable to contact the mail server API');

抛出此异常时，将显示详细的错误消息，其中包含文件和行号。

<a name="validation-exception"></a>
### 验证异常

`October\Rain\Exception\ValidationException`类，别名为`ValidationException`，用于直接与表单提交和无效字段相关的错误。 该消息应包含一个包含字段和错误消息的数组。

    throw new ValidationException(['username' => 'Sorry that username is already taken!']);

您还可以传递[验证服务](validation)的实例。

    $validation = Validator::make(...);

    if ($validation->fails()) {
        throw new ValidationException($validation);
    }

当抛出此异常时，[AJAX框架](../ajax/introduction)将以可用格式提供此信息并聚焦第一个无效字段。

<a name="ajax-exception"></a>
### AJAX异常

`October\Rain\Exception\AjaxException`类，别名为`AjaxException`，被认为是“智能错误”并将返回HTTP代码406.这允许它们传递响应内容，就好像它们是成功的响应一样。

    throw new AjaxException(['#flashMessages' => $this->renderPartial(...)]);

抛出此异常时，[AJAX框架](../ajax/introduction) 将遵循标准错误工作流程，但也将刷新指定的部分。

<a name="exception-handling"></a>
## 异常处理

所有异常都由`October\Rain\Foundation\Exception\Handler`类处理。 该类包含两个方法：`report`和`render`，指示是否应记录错误以及如何响应错误。

但是，如果需要，您可以使用`App::error`方法指定自定义处理程序。 处理程序根据它们处理的异常的类型提示进行调用。 例如，您可以创建仅处理`RuntimeException`实例的处理程序：

    App::error(function(RuntimeException $exception) {
        // Handle the exception...
    });

如果异常处理程序返回响应，则该响应将发送到浏览器，并且不会调用其他错误处理程序：

    App::error(function(InvalidUserException $exception) {
        return 'Sorry! Something is wrong with this account!';
    });

要监听PHP致命错误，您可以使用`App::fatal`方法：

    App::fatal(function($exception) {
        //
    });

如果您有多个异常处理程序，则应从最通用到最具体的定义它们。 因此，例如，应该在自定义异常类型（如`SystemException`）之前定义处理所有类型`Exception`异常的处理程序。

### 放置错误处理程序的位置

错误处理程序注册（如[事件处理程序](events)）通常属于“引导代码”类别。 换句话说，它们准备应用程序以实际处理请求，并且通常需要在实际调用路由或控制器之前执行。 最常见的地方是[插件注册文件](../plugin/registration#registration-methods)的`boot`方法。 或者，插件可以在插件目录中提供名为**init.php**的文件，您可以使用该文件来放置错误处理程序注册。

<a name="http-exceptions"></a>
## HTTP异常

一些例外描述了来自服务器的HTTP错误代码。 例如，这可能是“未找到页面”错误（404），“未授权错误”（401）或甚至是开发者生成500错误。 要从应用程序的任何位置生成此类响应，请使用以下命令：

    App::abort(404);

`abort`方法将立即引发一个由异常处理程序呈现的异常。 （可选）您可以提供响应文本：

    App::abort(403, 'Unauthorized action.');

可以在请求的生命周期中的任何时间使用此方法。

<a name="custom-error-page"></a>
### 自定义错误页面

默认情况下，任何错误都将显示一个详细的错误页面，其中包含发生错误的文件内容，行号和堆栈跟踪。 您可以通过在`config/app.php`脚本中将配置值`debug`设置为**false**并创建一个URL为`/error`的页面来显示自定义错误页面。

<a name="logging"></a>
## 日志

默认情况下，October配置为为您的应用程序创建一个日志文件，该文件存储在`storage/logs`目录中。 您可以使用`Log`facade将信息写入日志：

    $user = User::find(1);
    Log::info('Showing user profile for user: '.$user->name);

记录器提供[RFC 5424](http://tools.ietf.org/html/rfc5424)中定义的八个记录级别：**emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** 和 **debug**

    Log::emergency($error);
    Log::alert($error);
    Log::critical($error);
    Log::error($error);
    Log::warning($error);
    Log::notice($error);
    Log::info($error);
    Log::debug($error);

#### 上下文信息

还可以将一组上下文数据传递给日志方法。 此上下文数据将被格式化并显示日志消息：

    Log::info('User failed to login.', ['id' => $user->id]);

<a name="helpers"></a>
### 帮助方法

有一些全局帮助程序方法可以使日志记录更容易。 `trace_log`函数是`Log::info`的别名，支持使用数组和异常作为消息。

    // Write a string value
    $val = 'Hello world';
    trace_log('The value is '.$val);

    // Dump an array value
    $val = ['Some', 'array', 'data'];
    trace_log($val);

    // Trace an exception
    try {
        //
    }
    catch (Exception $ex) {
        trace_log($ex);
    }

`trace_sql`函数启用数据库日志记录，调用它时会记录发送到数据库的每个命令。 这些记录仅出现在`system.log`文件中，并且不会出现在管理区域日志中，因为它存储在数据库中并会导致反馈循环。

    trace_sql();

    Db::table('users')->count();

    // select count(*) as aggregate from users
