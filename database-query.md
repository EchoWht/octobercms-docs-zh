# 数据库查询

- [介绍](#introduction)
- [检索结果](#retrieving-results)
    - [合计](#aggregates)
- [Selects](#selects)
- [Joins](#joins)
- [Unions](#unions)
- [Where clauses](#where-clauses)
    - [Advanced where clauses](#advanced-where-clauses)
- [Ordering, grouping, limit, & offset](#ordering-grouping-limit-and-offset)
- [Inserts](#inserts)
- [Updates](#updates)
- [Deletes](#deletes)
- [悲观锁](#pessimistic-locking)
- [缓存查询](#caching-queries)

<a name="introduction"></a>
## 介绍

数据库查询构建器为创建和运行数据库查询提供了方便，流畅的界面。 它可用于在您的应用程序中执行大多数数据库操作，并适用于所有受支持的数据库系统。

> **注意:** 查询构建器使用PDO参数绑定来保护您的应用程序免受SQL注入攻击。 无需清除作为绑定传递的字符串。

<a name="retrieving-results"></a>
## 检索结果

#### 从表中检索所有行

要开始流畅的查询，请在`Db`外观上使用`table`方法。 `table`方法为给定的表返回一个流畅的查询构建器实例，允许您将更多约束链接到查询，然后最终得到结果。 在这个例子中，让我们从表中`get`所有记录：

    $users = Db::table('users')->get();

与[raw queries](../database/basics＃running-queries)一样，`get`方法返回一个结果的`array`，其中每个结果都是PHP`stdClass`对象的一个实例。 您可以通过访问列作为对象的属性来访问每个列的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 从表中检索单个行/列

如果您只需要从数据库表中检索单行，则可以使用`first`方法。 此方法将返回单个`stdClass`对象：

    $user = Db::table('users')->where('name', 'John')->first();

    echo $user->name;

如果您甚至不需要整行，则可以使用`value`方法从记录中提取单个值。 此方法将直接返回列的值：

    $email = Db::table('users')->where('name', 'John')->value('email');

#### 从表中获得结果

如果需要处理数千个数据库记录，请考虑使用`chunk`方法。 此方法一次检索结果的一小部分“块”，并将每个块提供给`Closure`进行处理。 此方法对于编写处理数千条记录的[控制台命令](../console/development)非常有用。 例如，让我们一次使用100个记录的整个`users`表：

    Db::table('users')->chunk(100, function($users) {
        foreach ($users as $user) {
            //
        }
    });

您可以通过从`Closure`返回`false`来阻止进一步处理块：

    Db::table('users')->chunk(100, function($users) {
        //Process the records...

        return false;
    });

#### 检索列值列表

如果要检索包含单个列值的数组，可以使用`lists`方法。 在这个例子中，我们将检索角色标题数组：

    $titles = Db::table('roles')->lists('title');

    foreach ($titles as $title) {
        echo $title;
    }

 您还可以为返回的数组指定自定义键列：

    $roles = Db::table('roles')->lists('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="aggregates"></a>
### Aggregates

查询构建器还提供了各种聚合方法，例如`count`，`max`，`min`，`avg`和`sum`。 您可以在构建查询后调用以下任何方法：

    $users = Db::table('users')->count();

    $price = Db::table('orders')->max('price');

当然，您可以将这些方法与其他子句组合以构建查询：

    $price = Db::table('orders')
        ->where('is_finalized', 1)
        ->avg('price');

<a name="selects"></a>
## Selects

#### 指定select子句

当然，您可能并不总是希望从数据库表中选择所有列。 使用`select`方法，您可以为查询指定自定义`select`子句：

    $users = Db::table('users')->select('name', 'email as user_email')->get();

`distinct`方法允许您强制查询返回不同的结果：

    $users = Db::table('users')->distinct()->get();

如果您已经有一个查询构建器实例，并且希望在其现有的select子句中添加一列，则可以使用`addSelect`方法：

    $query = Db::table('users')->select('name');

    $users = $query->addSelect('age')->get();

#### 原始表达

有时您可能需要在查询中使用原始表达式。 这些表达式将作为字符串注入到查询中，因此请注意不要创建任何SQL注入点！ 要创建原始表达式，可以使用`Db :: raw`方法：

    $users = Db::table('users')
        ->select(Db::raw('count(*) as user_count, status'))
        ->where('status', '<>', 1)
        ->groupBy('status')
        ->get();

<a name="joins"></a>
## Joins

#### 内连接语句

查询构建器还可用于编写连接语句。 要执行基本SQL“内部联接 inner join”，可以在查询构建器实例上使用`join`方法。 传递给`join`方法的第一个参数是您需要连接的表的名称，而其余参数指定连接的列约束。 当然，正如您所看到的，您可以在单个查询中连接到多个表：

    $users = Db::table('users')
        ->join('contacts', 'users.id', '=', 'contacts.user_id')
        ->join('orders', 'users.id', '=', 'orders.user_id')
        ->select('users.*', 'contacts.phone', 'orders.price')
        ->get();

#### 左连接语句

如果您想执行“左连接”而不是“内连接”，请使用`leftJoin`方法。 `leftJoin`方法与`join`方法具有相同的签名：

    $users = Db::table('users')
        ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
        ->get();

#### 高级连接语句

您还可以指定更高级的连接子句。 首先，将`Closure`作为第二个参数传递给`join`方法。 `Closure`将收到一个`JoinClause`对象，它允许你指定`join`子句的约束：

    Db::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();

如果您想在连接上使用“where”样式子句，则可以在连接上使用`where`和`orWhere`方法。 这些方法不是比较两列，而是将列与值进行比较：

    Db::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                ->where('contacts.user_id', '>', 5);
        })
        ->get();

<a name="unions"></a>
## Unions

查询构建器还提供了将两个查询“联合”在一起的快速方法。 例如，您可以创建一个初始查询，然后使用`union`方法将其与第二个查询结合起来：

    $first = Db::table('users')
        ->whereNull('first_name');

    $users = Db::table('users')
        ->whereNull('last_name')
        ->union($first)
        ->get();

`unionAll`方法也可用，并且具有与`union`相同的方法签名。

<a name="where-clauses"></a>
## Where子句

#### 简单的where子句

要在查询中添加`where`子句，请在查询构建器实例上使用`where`方法。 对where`的最基本调用需要三个参数。 第一个参数是列的名称。 第二个参数是一个运算符，它可以是数据库支持的任何运算符。 第三个参数是要针对列进行评估的值。

例如，这是一个验证“votes”列的值等于100的查询：

    $users = Db::table('users')->where('votes', '=', 100)->get();

为方便起见，如果您只想验证列是否等于给定值，您可以直接将值作为第二个参数传递给`where`方法：

    $users = Db::table('users')->where('votes', 100)->get();

当然，在编写`where`子句时，您可以使用各种其他运算符：

    $users = Db::table('users')
        ->where('votes', '>=', 100)
        ->get();

    $users = Db::table('users')
        ->where('votes', '<>', 100)
        ->get();

    $users = Db::table('users')
        ->where('name', 'like', 'T%')
        ->get();

#### "Or" 语句

您可以将约束链接在一起，以及向查询添加“或”子句。 `orWhere`方法接受与`where`方法相同的参数：

    $users = Db::table('users')
        ->where('votes', '>', 100)
        ->orWhere('name', 'John')
        ->get();

#### "Where between" 语句

`whereBetween`方法验证列的值是否在两个值之间：

    $users = Db::table('users')
        ->whereBetween('votes', [1, 100])->get();

`whereNotBetween`方法验证列的值是否在两个值之外：

    $users = Db::table('users')
        ->whereNotBetween('votes', [1, 100])
        ->get();

#### "Where in" 语句

`whereIn`方法验证给定列的值是否包含在给定数组中：

    $users = Db::table('users')
        ->whereIn('id', [1, 2, 3])
        ->get();

`whereNotIn`方法验证给定列的值是否包含在给定数组中：

    $users = Db::table('users')
        ->whereNotIn('id', [1, 2, 3])
        ->get();

#### "Where null" 语句

`whereNull`方法验证给定列的值是否为“NULL”：

    $users = Db::table('users')
        ->whereNull('updated_at')
        ->get();

`whereNotNull`方法验证列的值是**不是**`NULL`：

    $users = Db::table('users')
        ->whereNotNull('updated_at')
        ->get();

<a name="advanced-where-clauses"></a>
## 高级where子句

#### 参数分组

有时您可能需要创建更高级的where子句，例如“where exists”或嵌套参数分组。 Laravel查询构建器也可以处理这些。 首先，让我们看一下在括号内分组约束的示例：

    Db::table('users')
        ->where('name', '=', 'John')
        ->orWhere(function ($query) {
            $query->where('votes', '>', 100)
                ->where('title', '<>', 'Admin');
        })
        ->get();

如您所见，将`Closure`传递给`orWhere`方法指示查询构建器开始一个约束组。 `Closure`将接收一个查询构建器实例，您可以使用它来设置括号内应包含的约束。 上面的示例将生成以下SQL：

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

#### Exists 语句

`whereExists`方法允许你编写`where exist` SQL子句。 `whereExists`方法接受一个`Closure`参数，该参数将接收一个查询构建器实例，允许您定义应放在“exists”子句中的查询：

    Db::table('users')
        ->whereExists(function ($query) {
            $query->select(Db::raw(1))
                ->from('orders')
                ->whereRaw('orders.user_id = users.id');
        })
        ->get();

上面的查询将生成以下SQL：

    select * from users where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, grouping, limit, & offset

#### 排序

`orderBy`方法允许您按给定列对查询结果进行排序。 `orderBy`方法的第一个参数应该是你想要排序的列，而第二个参数控制排序的方向，可以是`asc`或`desc`：

    $users = Db::table('users')
        ->orderBy('name', 'desc')
        ->get();

#### 分组

`groupBy`和`having`方法可用于对查询结果进行分组。 `having`方法的签名类似于`where`方法：

    $users = Db::table('users')
        ->groupBy('account_id')
        ->having('account_id', '>', 100)
        ->get();

`havingRaw`方法可用于将原始字符串设置为`having`子句的值。 例如，我们可以找到销售额超过2,500美元的所有部门：

    $users = Db::table('orders')
        ->select('department', Db::raw('SUM(price) as total_sales'))
        ->groupBy('department')
        ->havingRaw('SUM(price) > 2500')
        ->get();

#### Limit 和 offset

要限制从查询返回的结果数，或者在查询中跳过给定数量的结果(`OFFSET`)，您可以使用`skip`和`take`方法：

    $users = Db::table('users')->skip(10)->take(5)->get();

<a name="inserts"></a>
## Inserts

查询构建器还提供了一个`insert`方法，用于将记录插入数据库表。 `insert`方法接受要插入的列名和值数组：

    Db::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

您甚至可以通过传递一个数组数组来调用`insert`来向表中插入几条记录。 每个数组表示要插入表中的行：

    Db::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### 自动递增ID

如果表具有自动递增ID，请使用`insertGetId`方法插入记录，然后检索ID：

    $id = Db::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> **注意:** 使用PostgreSQL数据库驱动程序时，insertGetId方法要求将自动递增列命名为“id”。 如果要从不同的“序列”中检索ID，可以将序列名称作为第二个参数传递给`insertGetId`方法。

<a name="updates"></a>
## Updates

除了将记录插入数据库之外，查询构建器还可以使用`update`方法更新现有记录。 `update`方法与`insert`方法一样，接受包含要更新的列的列和值对的数组。 您可以使用`where`子句约束`update`查询：

    Db::table('users')
        ->where('id', 1)
        ->update(['votes' => 1]);

#### I递增/递减

查询构建器还提供了用于递增或递减给定列的值的便捷方法。 与手动编写`update`语句相比，这只是一个捷径，提供了更具表现力和简洁的界面。

这两种方法都接受至少一个参数：要修改的列。 可以可选地传递第二个参数以控制列应该递增/递减的量。

    Db::table('users')->increment('votes');

    Db::table('users')->increment('votes', 5);

    Db::table('users')->decrement('votes');

    Db::table('users')->decrement('votes', 5);

您还可以在操作期间指定要更新的其他列：

    Db::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes

查询构建器还可用于通过`delete`方法从表中删除记录：

    Db::table('users')->delete();

您可以通过在调用`delete`方法之前添加`where`子句来约束`delete`语句：

    Db::table('users')->where('votes', '<', 100)->delete();

如果您希望截断整个表，这将删除所有行并将自动递增ID重置为零，您可以使用`truncate`方法：

    Db::table('users')->truncate();

<a name="pessimistic-locking"></a>
## 悲观锁

查询构建器还包含一些函数，可帮助您对`select`语句执行“悲观锁定”。 要使用“共享锁”运行语句，可以在查询中使用`sharedLock`方法。 在事务提交之前，共享锁可防止选定的行被修改：

    Db::table('users')->where('votes', '>', 100)->sharedLock()->get();

或者，您可以使用`lockForUpdate`方法。 “for update”锁可防止修改行或使用另一个共享锁选择：

    Db::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

<a name="caching-queries"></a>
## 缓存查询

<a name="adding-constraints"></a>
### 持久缓存

您可以使用[缓存服务](../services/cache)轻松缓存查询结果。 在准备查询时，只需链接`remember`或`rememberForever`方法即可。

    $users = Db::table('users')->remember(10)->get();

在此示例中，查询结果将缓存十分钟。 在缓存结果时，将不会对数据库运行查询，并且将从为应用程序指定的默认缓存驱动程序中加载结果。

<a name="adding-constraints"></a>
### 内存缓存

通过使用内存缓存可以防止同一请求中的重复查询。 默认情况下，对于[由模型准备的查询](../database/model＃retrievaling-models)启用此功能，但不启用直接使用`Db`facade生成的功能。

    Db::table('users')->get(); //Result from database
    Db::table('users')->get(); //Result from database

    Model::all(); //Result from database
    Model::all(); //Result from in-memory cache

您可以使用`enableDuplicateCache`或`disableDuplicateCache`方法启用或禁用重复缓存。

    Db::table('users')->enableDuplicateCache()->get();

如果查询存储在缓存中，则在使用insert，update，delete或truncate语句时将自动清除该查询。 但是，您可以使用`flushDuplicateCache`方法手动清除缓存。

    Db::flushDuplicateCache();

> **注意**: 通过命令行界面(CLI)运行时，内存缓存完全禁用。
