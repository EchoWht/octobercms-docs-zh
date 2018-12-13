# 会话服务

- [配置](#configuration)
- [会话使用](#session-usage)
- [Flash数据](#flash-data)

<a name="configuration"></a>
## 配置

由于HTTP驱动的应用程序是无状态的，因此会话提供了跨请求存储有关用户的信息的方法。 October发布了各种会话后端，可通过干净，统一的API使用。 支持流行的后端，如[Memcached](http://memcached.org)，[Redis](http://redis.io)和数据库，包括开箱即用。

会话配置存储在`config/session.php`中。 请务必查看此文件中可用的记录良好的选项。 默认情况下，October配置为使用`file`会话驱动程序，该驱动程序适用于大多数应用程序。

<div class="content-list" markdown="1">
- `file` - 会话存储在 `storage/framework/sessions`文件。
- `cookie` - 会话存储在安全的加密cookie中。
- `database` - 会话存储在应用程序使用的数据库中。
- `memcached`/`redis` -会话存储在这些快速的基于缓存的存储之一中。
- `array` - 会话存储在一个简单的PHP数组中，不会在请求之间保留。
</div>

> **注意:** array驱动程序通常用于运行单元测试，以防止会话数据持久存在。

#### 保留键

October 在内部使用`flash`会话密钥，因此您不应该通过该名称向会话添加项目。

<a name="session-usage"></a>
## 会话使用

#### 在会话中存储数据

使用`Session`facade，您可以调用各种函数来与底层数据进行交互。 例如，`put`方法在会话中存储新的数据：

    Session::put('key', 'value');

#### 推送到数组会话值

`push`方法可用于将新值推送到作为数组的会话值。 例如，如果`user.teams`键包含一个团队名称数组，您可以将新值推送到数组上，如下所示：

    Session::push('user.teams', 'developers');

#### 从会话中检索数据

从会话中检索值时，您还可以将默认值作为第二个参数传递给`get`方法。 如果会话中不存在指定的密钥，则将返回此默认值。 如果将`Closure`作为缺省值传递给`get`方法，则会执行`Closure`并返回其结果：

    $value = Session::get('key');

    $value = Session::get('key', 'default');

    $value = Session::get('key', function() { return 'default'; });

#### 从会话中检索所有数据

如果您想从会话中检索所有数据，可以使用`all`方法：

    $data = Session::all();

#### 检索数据并忘记它

`pull`方法将从会话中检索和删除项目：

    $value = Session::pull('key', 'default');

#### 确定会话中是否存在项目

`has`方法可用于检查会话中是否存在项目。 如果项存在，此方法将返回`true`：

    if (Session::has('users')) {
        //
    }

#### 从会话中删除数据

`forget`方法将从会话中删除一段数据。 如果要从会话中删除所有数据，可以使用`flush`方法：

    Session::forget('key');

    Session::flush();

#### 重新生成会话ID

如果需要重新生成会话ID，可以使用`regenerate`方法：

    Session::regenerate();

<a name="flash-data"></a>
## Flash数据

有时您可能希望仅在下一个请求中将项目存储在会话中。 您可以使用`Session::flash`方法执行此操作。 使用此方法存储在会话中的数据仅在后续HTTP请求期间可用，然后将被删除。 Flash数据主要用于短期状态消息：

    Session::flash('key', 'value');

如果您需要保留闪存数据以获得更多请求，您可以使用`reflash`方法，该方法将保留所有闪存数据以获取额外请求。 如果您只需要保留特定的闪存数据，可以使用`keep`方法：

    Session::reflash();

    Session::keep(['username', 'email']);
