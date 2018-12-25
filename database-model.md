# 活动记录模型

- [介绍](#introduction)
- [定义模型](#defining-models)
    - [支持的属性](#standard-properties)
- [检索模型](#retrieving-models)
    - [检索多个模型](#retrieving-multiple-models)
    - [检索单个模型](#retrieving-single-models)
    - [检索合计](#retrieving-aggregates)
- [插入和更新模型](#inserting-and-updating-models)
    - [简单的inserts插入](#basic-inserts)
    - [简单的updates更新](#basic-updates)
    - [批量处理](#mass-assignment)
- [删除模型](#deleting-models)
- [查询范围](#query-scopes)
- [事件](#events)
- [扩展模型](#extending-models)

<a name="introduction"></a>
## 介绍

October根据[Laravel的Eloquent](http://laravel.com/docs/eloquent)提供了一个美观而简单的Active Record实现，用于处理您的数据库。 每个数据库表都有一个相应的“模型”，用于与该表进行交互。 模型允许您查询表中的数据，以及在表中插入新记录。

模型类放在插件目录的**models**子目录中。 模型目录结构的示例：

    plugins/
      acme/
        blog/
          models/
            user/             <=== 模型配置目录
              columns.yaml    <=== 模型配置文件
              fields.yaml     <==^
            User.php          <=== 模型类
          Plugin.php

模型配置目录可以包含模型的[list column](backend-lists.md#list-columns) 和[form field](backend-forms.md#form-fields) 定义。 模型配置目录名称与以小写形式写入的模型类名称匹配。

<a name="defining-models"></a>
## 定义模型

在大多数情况下，您应该为每个数据库表创建一个模型类。 所有模型类都必须扩展`Model`类。 插件中使用的模型的最基本表示如下所示：

    namespace Acme\Blog\Models;

    use Model;

    class Post extends Model
    {
        /**
         * 与模型关联的表。
         *
         * @var string
         */
        protected $table = 'acme_blog_posts';
    }

`$table` protected字段指定与模型对应的数据库表。 表名是作者的名称，插件和复数记录类型名称。

<a name="standard-properties"></a>
### 支持的属性

除了[模型特征](database-traits.md)提供的标准属性之外，还可以在模型上找到一些标准属性。 例如：

    class User extends Model
    {
        protected $primaryKey = 'id';

        public $exists = false;

        protected $dates = ['last_seen_at'];

        public $timestamps = true;

        protected $jsonable = ['permissions'];

        protected $guarded = ['*'];
    }

属性 | 描述
------------- | -------------
**$primaryKey** | 用于标识模型的主键名称。
**$incrementing** | boolean，如果为false则表示主键不是递增的整数值。
**$exists** | boolean， 如果为true则表示模型存在。
**$dates** | 取值后，值将转换为Carbon/DateTime对象的实例。
**$timestamps** | boolean，如果为true将自动设置created_at和updated_at字段。
**$jsonable** | 值在保存之前编码为JSON，并在获取后转换为数组。
**$fillable** | 值是[批量处理](#mass-assignment)可访问的字段。   
**$guarded** | 值是从[批量处理](#mass-assignment)保护的字段。
**$visible** | 值是[序列化模型数据](database-serialization.md)时可见的字段。
**$hidden** | 值是[序列化模型数据](database-serialization.md)时隐藏的字段。

<a name="property-primary-key"></a>
#### 主键

模型将假设每个表都有一个名为`id`的主键列。 您可以定义`$primaryKey`属性来覆盖此约定。

    class Post extends Model
    {
        /**
         * 模型的主键。
         *
         * @var string
         */
        protected $primaryKey = 'id';
    }

<a name="property-incrementing"></a>
#### 递增

模型将假定主键是递增的整数值，这意味着默认情况下主键将自动转换为整数。 如果要使用非递增或非数字主键，则必须将公共`$incrementing`属性设置为false。

    class Message extends Model
    {
        /**
         * 模型的主键不是整数。
         *
         * @var bool
         */
        public $incrementing = false;
    }

<a name="property-timestamps"></a>
#### 时间戳

默认情况下，模型会在表上存在`created_at`和`updated_at`列。 如果您不希望自动管理这些列，请将模型上的`$timestamps`属性设置为`false`：

    class Post extends Model
    {
        /**
         * 指示模型是否应加时间戳。
         *
         * @var bool
         */
        public $timestamps = false;
    }

如果需要自定义时间戳的格式，请在模型上设置`$dateFormat`属性。 此属性确定数据库中日期属性的存储方式，以及将模型序列化为数组或JSON时的格式：

    class Post extends Model
    {
        /**
         * 模型日期列的存储格式。
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="property-jsonable"></a>
#### 值存储为JSON

当属性名称传递给`$jsonable`属性时，值将作为JSON从数据库序列化和反序列化：

    class Post extends Model
    {
        /**
         * @var array 使用JSON进行编码和解码的属性名称。
         */
        protected $jsonable = ['data'];
    }

<a name="retrieving-models"></a>
## 检索模型

从数据库请求数据时，模型将主要使用`get`或`first`方法检索值，具体取决于您是否[检索多个模型](#retrieving-multiple-models)或[检索单个模型](#retrieving-single-models) 。 从Model派生的查询返回[October\Rain\Database\Builder](../api/october/rain/database/builder)的实例。

> **注意**: 默认情况下，所有模型查询都具已[启用内存缓存](database-query.md#caching-queries) 。

<a name="retrieving-multiple-models"></a>
### 检索多个模型

一旦创建了模型和[其关联的数据库表](database-structure.md#migration-structure)，就可以开始从数据库中检索数据了。 将每个模型视为功能强大的[查询构建器](database-query.md)，允许您查询与模型关联的数据库表。 例如：

    $flights = Flight::all();

<a name="accessing-column-values"></a>
#### 访问列值

如果您有模型实例，则可以通过访问相应的属性来访问模型的列值。 例如，让我们遍历查询返回的每个`Flight`实例，并回显`name`列的值：

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="adding-constraints"></a>
#### 添加其他约束

`all`方法将返回模型表中的所有结果。 由于每个模型都用作[查询构建器](database-query.md)，您还可以向查询添加约束，然后使用`get`方法检索结果：

    $flights = Flight::where('active', 1)
        ->orderBy('name', 'desc')
        ->take(10)
        ->get();

> **注意:** 由于模型是查询构建器，因此您应该熟悉[查询构建器](database-query.md)中可用的所有方法。 您可以在模型查询中使用任何这些方法。

<a name="returning-collections"></a>
#### 集合

对于像检索多个结果的`all`和`get`这样的方法，将返回一个`Collection`的实例。 此类提供[各种有用的方法](database-collection.md) 来处理结果。 当然，您可以像数组一样循环遍历此集合：

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
#### 分块结果

如果需要处理数千条记录，请使用`chunk`命令。 `chunk`方法将检索模型的“块”，将它们提供给给定的`Closure`进行处理。 在处理大型结果集时，使用`chunk`方法将节省内存：

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

传递给该方法的第一个参数是您希望每个“块”接收的记录数。 将为从数据库检索的每个块调用作为第二个参数传递的Closure。

<a name="retrieving-single-models"></a>
### 检索单个模型

检索单个模型除了检索给定表的所有记录之外，您还可以使用“find”和“first”检索单个记录。 这些方法返回单个模型实例，而不是返回模型集合：

    // 通过主键检索模型
    $flight = Flight::find(1);

    // 检索与查询约束匹配的第一个模型
    $flight = Flight::where('active', 1)->first();

<a name="model-not-found-exception"></a>
#### 没有找到对应的数据

有时，如果找不到模型，您可能希望抛出异常。 这在路线或控制器中特别有用。 `findOrFail`和`firstOrFail`方法将检索查询的第一个结果。 但是，如果没有找到结果，将抛出`Illuminate\Database\Eloquent\ModelNotFoundException`：

    $model = Flight::findOrFail(1);

    $model = Flight::where('legs', '>', 100)->firstOrFail();

当[开发API](services-router.md)时，如果未捕获到异常，则会自动将“404”HTTP响应发送回用户，因此不必编写显式检查来返回“404”。 使用这些方法时的响应：

    Route::get('/api/flights/{id}', function ($id) {
        return Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### 检索聚合

您还可以使用查询构建器提供的`count`，`sum`，`max`和其他[aggregate functions](database-query.md#aggregates) 。 这些方法返回适当的标量值而不是完整的模型实例：

    $count = Flight::where('active', 1)->count();

    $max = Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## 插入和更新模型

插入和更新数据是模型的基础特征，与传统的SQL语句相比，它使过程毫不费力。

<a name="basic-inserts"></a>
### 简单的inserts插入

要在数据库中创建新记录，只需创建一个新的模型实例，在模型上设置属性，然后调用`save`方法：

    $flight = new Flight;
    $flight->name = 'Sydney to Canberra';
    $flight->save();

在这个例子中，我们只是创建一个`Flight`模型的新实例并分配`name`属性。 当我们调用`save`方法时，会将一条记录插入到数据库中。 `created_at`和`updated_at`时间戳也将自动设置，因此无需手动设置它们。

<a name="basic-updates"></a>
### 简单的updates更新

“save”方法也可用于更新数据库中已存在的模型。 要更新模型，您应该检索它，设置您想要更新的任何属性，然后调用`save`方法。 同样，`updated_at`时间戳将自动更新，因此无需手动设置其值：

    $flight = Flight::find(1);
    $flight->name = 'Darwin to Adelaide';
    $flight->save();

还可以针对与给定查询匹配的任意数量的模型执行更新。 在这个例子中，所有 `active` 并且具有`destination` 目的地的航班将被标记为延迟：

    Flight::where('is_active', true)
        ->where('destination', 'Perth')
        ->update(['delayed' => true]);

`update`方法需要一个列和值对的数组，表示应该更新的列。

<a name="mass-assignment"></a>
### 批量处理

您也可以使用`create`方法将新模型保存在一行中。 插入的模型实例将从该方法返回给您。 但是，在这样做之前，您需要在模型上指定`fillable`或`guarded`属性，因为所有模型都可以防止批量处理。 请注意，`fillable`或`guarded`都不会影响后端表单的提交，只会应用在`create`或`fill`方法。

当用户通过请求传递意外的HTTP参数时，会发生批量处理漏洞，并且该参数会更改数据库中您不期望的列。 例如，恶意用户可能通过HTTP请求发送`is_admin`参数，然后将其映射到模型的`create`方法，允许用户将自己升级为管理员。

首先，您应该定义要批量处理的模型属性。 您可以使用模型上的`$fillable`属性执行此操作。 例如，让我们的`Flight`模型的`name`属性可以分配：

    class Flight extends Model
    {
        /**
         * 可批量处理的属性。
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

一旦我们将属性赋予质量可分配，我们就可以使用`create`方法在数据库中插入新记录。 `create`方法返回保存的模型实例：

    $flight = Flight::create(['name' => 'Flight 10']);

虽然`$fillable`可以作为可以批量处理的属性的“白名单”，但您也可以选择使用`$guarded`。 `$guarded`属性应包含一系列您不希望可批量处理的属性。 不在数组中的所有其他属性将是可批量处理的。 因此，`$guarded`功能就像一个“黑名单”。 当然，你应该使用`$fillable`或`$guarded` - 而不是两者：

    class Flight extends Model
    {
        /**
         * 不可批量处理的属性。
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

在上面的例子中，除了'price` **之外的所有属性**都是可批量处理的。

#### 其他创建方法

S有时您可能希望仅实例化模型的新实例。 您可以使用`make`方法执行此操作。 `make`方法只返回一个新实例而不保存或创建任何东西。

    $flight = Flight::make(['name' => 'Flight 10']);

    // 功能与以下相同
    $flight = new Flight;
    $flight->fill(['name' => 'Flight 10']);

您可以使用另外两种方法通过批量处理属性来创建模型：`firstOrCreate`和`firstOrNew`。 `firstOrCreate`方法将尝试使用给定的列/值对定位数据库记录。 如果在数据库中找不到该模型，则将插入具有给定属性的记录。

`firstOrNew`方法，如`firstOrCreate`将尝试在匹配给定属性的数据库中查找记录。 但是，如果未找到模型，则将返回新的模型实例。 请注意，`firstOrNew`返回的模型尚未保存到数据库中。 你需要手动调用`save`来保存它：

    // 按属性检索Flight，如果没有创建它
    $flight = Flight::firstOrCreate(['name' => 'Flight 10']);

    //按属性检索Flight，或实例化新实例
    $flight = Flight::firstOrNew(['name' => 'Flight 10']);

<a name="deleting-models"></a>
## 删除模型

要删除模型，请在模型实例上调用`delete`方法：

    $flight = Flight::find(1);

    $flight->delete();

#### 按键删除现有模型

在上面的例子中，我们在调用`delete`方法之前从数据库中检索模型。 但是，如果您知道模型的主键，则可以删除模型而不检索它。 为此，请调用`destroy`方法：

    Flight::destroy(1);

    Flight::destroy([1, 2, 3]);

    Flight::destroy(1, 2, 3);

#### 按查询删除模型

您还可以在一组模型上运行删除查询。 在此示例中，我们将删除所有标记为非活动的Flight：

    $deletedRows = Flight::where('active', 0)->delete();

> **注意**: 值得一提的是，直接从查询中删除记录时，[模型事件](#model-events)不会触发。

<a name="query-scopes"></a>
## 查询范围

范围允许您定义可在整个应用程序中轻松重用的常见约束集。 例如，您可能需要经常检索所有被视为“受欢迎”的用户。 要定义范围，只需在模型方法前加上“scope”：

    class User extends Model
    {
        /**
         * 范围查询仅包括热门用户。
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * 查询范围仅包括活动用户。
         */
        public function scopeActive($query)
        {
            return $query->where('is_active', 1);
        }
    }

#### 利用查询范围

定义范围后，可以在查询模型时调用范围方法。 但是，调用方法时不需要包含`scope`前缀。 您甚至可以将调用链接到各种范围，例如：

    $users = User::popular()->active()->orderBy('created_at')->get();

#### 动态范围

有时您可能希望定义接受参数的范围。 要开始使用，只需将其他参数添加到示波器即可。 范围参数应在`$query`参数后定义：

    class User extends Model
    {
        /**
         * 查询范围仅包括给定类型的用户。
         */
        public function scopeApplyType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

现在，您可以在调用范围时传递参数：

    $users = User::applyType('admin')->get();

<a name="events"></a>
## 事件

模型可以触发多个事件，允许您连接到模型生命周期中的各个点。 事件允许您在每次在数据库中保存或更新特定模型类时轻松执行代码。 通过覆盖类中的特殊方法来定义事件，可以使用以下方法覆盖：

事件 | 描述
------------- | -------------
**beforeCreate** | 保存模型之前，首次创建时。
**afterCreate** | 保存模型后，首次创建时。
**beforeSave** | 在保存模型之前，创建或更新模型。
**afterSave** | 保存模型后，创建或更新。
**beforeValidate** | 在验证提供的模型数据之前。
**afterValidate** | 在验证提供的模型数据之后。
**beforeUpdate** | 在保存现有模型之前。
**afterUpdate** | 保存现有模型后。
**beforeDelete** | 在删除现有模型之前。
**afterDelete** | 删除现有模型后。
**beforeRestore** | 在恢复软删除模型之前。
**afterRestore** | 恢复软删除模型后。
**beforeFetch** | 在填充现有模型之前。
**afterFetch** | 在填充现有模型之后。

使用事件的示例：

    /**
     * Generate a URL slug for this model
     */
    public function beforeCreate()
    {
        $this->slug = Str::slug($this->name);
    }
    
> **注意:** 如果尚未提交，则在`afterSave`模型事件中将无法使用[延迟绑定](database-relations.md#deferred-binding)(即：文件附件)创建的关系。 要访问未提交的绑定，请在关系上使用`withDeferred($sessionKey)` 方法。 示例： `$this->images->withDeferred(post('_session_key'))->get();`

<a name="basic-usage"></a>
### 使用示例

无论何时第一次保存新模型，都会触发`beforeCreate`和`afterCreate`事件。 如果数据库中已存在模型并且调用了`save`方法，则会触发`beforeUpdate`/`afterUpdate`事件。 但是，在这两种情况下，`beforeSave` /`afterSave`事件都会触发。

例如，让我们定义一个事件监听器，它在首次创建模型时填充slug属性：

    /**
     * 为此模型生成URL slug
     */
    public function beforeCreate()
    {
        $this->slug = Str::slug($this->name);
    }

从事件返回`false`将取消`save` /`update`操作：

    public function beforeCreate()
    {
        if (!$user->isValid()) {
            return false;
        }
    }
    
可以使用`original`属性访问旧值。 例如：

    public function afterUpdate()
    {
        if ($this->title != $this->original['title']) {
            // title changed
        }
    }

您可以使用`bindEvent`方法从外部绑定到[local events](services-events.md)以获取模型的单个实例。 事件名称应与方法覆盖名称相同，前缀为“model.”。

    $flight = new Flight;
    $flight->bindEvent('model.beforeCreate', function() use ($model) {
        $model->slug = Str::slug($model->name);
    })

<a name="extending-models"></a>
## 扩展模型

由于模型[配备使用行为]](services-behaviors.md)，它们可以使用静态`extend`方法进行扩展。 该方法采用闭包并将模型对象传递给它。

在闭包内部，您可以向模型添加关系。 在这里，我们扩展`Backend\Models\User`模型以包含引用`Acme\Demo\Models\Profile`模型的配置文件(有一个)关系。

    \Backend\Models\User::extend(function($model) {
        $model->hasOne['profile'] = ['Acme\Demo\Models\Profile', 'key' => 'user_id'];
    });

此方法也可用于绑定到[local events](#events)，以下代码侦听`model.beforeSave`事件。

    \Backend\Models\User::extend(function($model) {
        $model->bindEvent('model.beforeSave', function() use ($model) {
            // ...
        });
    });

> **注意:** 通常，放置代码的最佳位置是在插件注册类`boot`方法中，因为这将在每个请求上运行，确保您对模型的扩展在任何地方都可用。

另外，存在一些扩展受保护模型属性的方法。

    \Backend\Models\User::extend(function($model) {
        // add cast attributes
        $model->addCasts([
            'some_extended_field' => 'int',
        ]);
        
        // add a date attribute
        $model->addDateAttribute('updated_at');
        
        // add fillable or jsonable fields
        // these methods accept one or more strings, or an array of strings
        $model->addFillable('first_name');
        $model->addJsonable('some_data');
    });
    
