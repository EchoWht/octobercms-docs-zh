# 数据库：入门

- [介绍](#introduction)
- [配置](#configuration)
    - [读/写连接](#read-write-connections)
- [运行原始SQL查询](#running-queries)
- [多个数据库连接](#accessing-connections)
- [数据库事务](#database-transactions)
- [数据库事件](#database-events)
    - [查询日志记录](#query-logging)


<a name="introduction"></a>
## 介绍

连接到数据库和运行查询是一个简单的过程，可以使用原始SQL，[查询构建器](../database/query) 或[活动记录模型](../database/model)来支持。 管理数据库表和填充种子数据由[migration and seeder process](../database/structure)处理。

原始SQL和使用查询构建器将执行得更快，应该用于简单的任务。 Active Record是流行框架Ruby On Rails使用的一种方法。 它允许一个简单的界面来执行重复的任务，如创建，读取，更新和删除数据库记录。 您可以了解有关[维基百科上的活动记录模式](http://en.wikipedia.org/wiki/Active_record_pattern)的更多信息。

<a name="configuration"></a>
## 配置

应用程序的数据库配置位于`config / database.php`文件中。 在此文件中，您可以定义所有数据库连接，以及指定默认情况下应使用的连接。 此文件中提供了所有受支持的数据库系统的示例。

<a name="read-write-connections"></a>
### 读/写连接

有时您可能希望为SELECT语句使用一个数据库连接，而对INSERT，UPDATE和DELETE语句使用另一个数据库连接。 无论您使用的是原始查询，查询构建器还是模型，都可以轻松指定使用哪个连接。

要了解应如何配置读/写连接，让我们看一下这个例子：

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
    ],

请注意，配置数组中添加了两个键：`read`和`write`。 这两个键的数组值都包含一个键：`host`。 `read`和`write`连接的其余数据库选项将从主`mysql`数组合并。

如果我们希望覆盖主数组中的值，我们只需要在`read`和`write`数组中放置项。 因此，在这种情况下，`192.168.1.1`将用作“读”连接，而`192.168.1.2`将用作“写”连接。 主`mysql`数组中的数据库凭据，前缀，字符集和所有其他选项将在两个连接中共享。

<a name="running-queries"></a>
## 运行原始SQL查询

配置数据库连接后，可以使用`Db` facade运行查询。 `Db` facade为每种类型的查询提供了方法：`select`，`update`，`insert`，`delete`和`statement`。

#### 运行选择查询

要运行基本查询，我们可以在`Db`facade上使用`select`方法：

    $users = Db::select('select * from users where active = ?', [1]);

传递给`select`方法的第一个参数是原始SQL查询，而第二个参数是需要绑定到查询的任何参数绑定。 通常，这些是`where`子句约束的值。 参数绑定提供了针对SQL注入的保护。

`select`方法将始终返回结果的`数组'。 数组中的每个结果都是一个PHP`stdClass`对象，允许您访问结果的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 使用命名绑定

您可以使用命名绑定执行查询，而不是使用`？`来表示参数绑定：

    $results = Db::select('select * from users where id = :id', ['id' => 1]);

#### 运行insert语句

要执行`insert`语句，可以在`Db`facade上使用`insert`方法。 与`select`类似，此方法将原始SQL查询作为其第一个参数，并将绑定作为第二个参数：

    Db::insert('insert into users (id, name) values (?, ?)', [1, 'Joe']);

#### 运行更新语句

应使用`update`方法更新数据库中的现有记录。 该语句将返回受该语句影响的行数：

    $affected = Db::update('update users set votes = 100 where name = ?', ['John']);

#### 运行删除语句

应该使用`delete`方法从数据库中删除记录。 与`update`一样，将返回已删除的行数：

    $deleted = Db::delete('delete from users');

#### 运行一般性声明

某些数据库语句不应返回任何值。 对于这些类型的操作，您可以在`Db`facade上使用`statement`方法：

    Db::statement('drop table users');

<a name="accessing-connections"></a>
## 多个数据库连接

使用多个连接时，您可以通过`Db`facade上的`connection`方法访问每个连接。 传递给`connection`方法的`name`应该对应于`config/database.php`配置文件中列出的连接之一：

    $users = Db::connection('foo')->select(...);

您还可以使用连接实例上的`getPdo`方法访问原始的底层PDO实例：

    $pdo = Db::connection()->getPdo();

<a name="database-transactions"></a>
## 数据库事务

要在数据库事务中运行一组操作，可以在`Db`facade上使用`transaction`方法。 如果在事务`Closure`中抛出异常，则事务将自动回滚。 如果`Closure`成功执行，则事务将自动提交。 使用`transaction`方法时，您无需担心手动回滚或提交：

    Db::transaction(function () {
        Db::table('users')->update(['votes' => 1]);

        Db::table('posts')->delete();
    });

#### 手动使用事务

如果您想手动开始事务并完全控制回滚和提交，可以在`Db`facade上使用`beginTransaction`方法：

    Db::beginTransaction();

您可以通过`rollBack`方法回滚事务：

    Db::rollBack();

最后，您可以通过`commit`方法提交事务：

    Db::commit();

> **注意:** 使用`Db` facade的事务方法还控制[query builder](../database/query) 和[model queries](../database/model)的事务。

<a name="database-events"></a>
## 数据库事件

如果您希望接收应用程序执行的每个SQL查询，可以使用`listen`方法。 此方法对于记录查询或调试很有用。

    Db::listen(function($sql, $bindings, $time) {
        //
    });

就像[事件注册](../services/events#event-registration),一样，你可以在[插件注册文件](../plugin/registration#registration-methods)的`boot`方法中注册你的查询监听器。 或者，插件可以在插件目录中提供名为**init.php**的文件，您可以使用该文件来放置此逻辑。

<a name="query-logging"></a>
### 查询日志记录

启用查询日志记录后，会在内存中保留已为当前请求运行的所有查询。 调用`enableQueryLog`方法来启用此功能。

    Db::connection()->enableQueryLog();

要获取已执行查询的数组，可以使用`getQueryLog`方法：

    $queries = Db::getQueryLog();

但是，在某些情况下，例如插入大量行时，这可能会导致应用程序使用过多的内存。 要禁用日志，可以使用`disableQueryLog`方法：

    Db::connection()->disableQueryLog();

> **注意**: 为了更快地调试，调用`trace_sql` [帮助方法](../services/error-log#helpers) 可能更有用。
