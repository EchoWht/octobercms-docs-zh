# 缓存

- [配置](#configuration)
- [缓存用法](#cache-usage)
    - [从缓存中检索数据](#retrieving-items-from-the-cache)
    - [将数据存储在缓存中](#storing-items-in-the-cache)
    - [从缓存中删除数据](#removing-items-from-the-cache)

<a name="configuration"></a>
## 配置

October为各种缓存系统提供了统一的API，缓存配置位于`config/cache.php`。 在此文件中，您可以指定在整个应用程序中默认使用的缓存驱动程序。 开箱即用支持[Memcached](http://memcached.org)和[Redis](http://redis.io)等热门缓存系统。

缓存配置文件还包含各种其他选项，这些选项在文件中记录，因此请务必阅读这些选项。 默认情况下，OctoberCMS配置为使用`file`缓存驱动程序，该驱动程序将序列化的缓存对象存储在文件系统中。 对于较大的应用程序，建议您使用内存缓存，如Memcached或APC。 您甚至可以为同一驱动程序配置多个缓存配置。

### 缓存条件

#### 数据库

`database`缓存驱动程序使用数据库代替文件系统。 由于数据库结构已经可用，因此不需要使用此类型的其他配置。

#### Memcached

使用Memcached缓存需要安装[Memcached PECL包](http://pecl.php.net/package/memcached)。

默认配置使用基于[Memcached::addServer]的TCP/IP(http://php.net/manual/en/memcached.addserver.php)：

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

您还可以将`host`选项设置为UNIX套接字路径。 如果这样做，`port`选项应设置为`0`：

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

> 您需要先安装[Drivers plugin](http://octobercms.com/plugin/october-drivers) 才能使用Redis缓存驱动程序。

应用程序的Redis配置位于`config/database.php`配置文件中。 在此文件中，您将看到一个`redis`数组，其中包含应用程序使用的Redis服务器：

    'redis' => [

        'cluster' => false,

        'default' => [
            'host'     => '127.0.0.1',
            'port'     => 6379,
            'database' => 0,
        ],

    ],

您可以在Redis连接定义中定义`options`数组值，允许您指定一组Predis [客户端选项](https://github.com/nrk/predis/wiki/Client-Options).。

如果您的Redis服务器需要身份验证，您可以通过向Redis服务器配置阵列添加“password”配置项来提供密码。

<a name="cache-usage"></a>
## 使用缓存

虽然大多数缓存是在October内部处理的，但`Cache`facade提供了一些简单的方法来缓存您自己的数据。

<a name="retrieving-items-from-the-cache"></a>
### 从缓存中检索数据

`Cache`facade上的`get`方法用于从缓存中检索数据。 如果缓存中不存在该项，则返回“null”。 如果您愿意，您可以将第二个参数传递给`get`方法，指定您希望在数据不存在时返回的自定义默认值：

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

你甚至可以传递一个`Closure`作为默认值。 如果缓存中不存在指定的项，则返回`Closure`的结果。 传递Closure允许您推迟从数据库或其他外部服务检索默认值：

    $value = Cache::get('key', function() {
        return Db::table(...)->get();
    });

#### 检查数据是否存在

`has`方法可用于确定缓存中是否存在数据：

    if (Cache::has('key')) {
        //
    }

#### I递增/递减值

`increment`和`decrement`方法可用于调整缓存中整数项的值。 这两种方法都可以选择接受第二个参数，表示增加或减少数据值的数量：

    Cache::increment('key');

    Cache::increment('key', $amount);

    Cache::decrement('key');

    Cache::decrement('key', $amount);

#### 检索或更新

有时您可能希望从缓存中检索数据，但如果请求的数据不存在，还会存储默认值。 例如，您可能希望从缓存中检索所有用户，如果它们不存在，则从数据库中检索它们并将它们添加到缓存中。 您可以使用`Cache :: remember`方法执行此操作：

    $value = Cache::remember('users', $minutes, function() {
        return Db::table('users')->get();
    });

如果缓存中不存在该项，则将执行传递给`remember`方法的`Closure`，并将其结果放入缓存中。

您也可以结合使用`remember`和`forever`方法：

    $value = Cache::rememberForever('users', function() {
        return Db::table('users')->get();
    });

#### 检索并删除

如果您需要从缓存中检索数据然后将其删除，您可以使用`pull`方法。 与`get`方法一样，如果缓存中不存在该项，则返回`null`：

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### 将数据存储在缓存中

您可以在`Cache`facade上使用`put`方法将数据存储在缓存中。 将数据放入缓存时，您需要指定缓存该值的分钟数：

    Cache::put('key', 'value', $minutes);

您也可以传递表示缓存数据到期时间的PHP`DateTime`实例，而不是传递数据到期前的分钟数：

    $expiresAt = Carbon::now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

如果缓存存储中尚不存在“add”方法，则只将该项添加到缓存中。 如果数据实际添加到缓存中，该方法将返回`true`。 否则，该方法将返回`false`：

    Cache::add('key', 'value', $minutes);

`forever`方法可用于永久存储缓存中的项。 必须使用`forget`方法从缓存中手动删除这些值：

    Cache::forever('key', 'value');

<a name="removing-items-from-the-cache"></a>
### 从缓存中删除数据

您可以使用`Cache`facade上的`forget`方法从缓存中删除数据：

    Cache::forget('key');

您可以使用`flush`方法清除整个缓存：

    Cache::flush();

刷新缓存*不会*依照缓存前缀，并将从缓存中删除所有条目。 清除其他应用程序共享的缓存时请仔细考虑。
