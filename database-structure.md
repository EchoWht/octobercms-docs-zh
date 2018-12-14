# 数据库结构和数据填充

- [介绍](#introduction)
- [迁移结构](#migration-structure)
    - [创建表](#creating-tables)
    - [重命名/删除表](#renaming-and-dropping-tables)
    - [创建列](#creating-columns)
    - [修改列](#modifying-columns)
    - [删除列](#dropping-columns)
    - [创建索引](#creating-indexes)
    - [删除索引](#dropping-indexes)
    - [外键约束](#foreign-key-constraints)
- [填充类结构](#seeder-structure)
    - [Calling additional seeders](#calling-additional-seeders)

<a name="introduction"></a>
## 介绍

迁移和填充文件允许您构建/修改和填充数据库表。 它们主要由[插件更新文件](../ plugin/updates)使用，并与插件的版本历史配对。 所有类都存储在插件的`updates`目录中。 迁移应该讲述一个关于您的数据库历史的故事，这个故事可以向前和向后播放，以构建和拆除表格。

<a name="migration-structure"></a>
## 迁移结构

迁移文件应该定义一个扩展`October\Rain\Database\Updates\Migration`类的类，它包含两个方法：`up`和`down`。 `up`方法用于向数据库添加新的表，列或索引，而`down`方法是`up`的逆向操作。 在这两种方法中，您可以使用[schema builder(表格一整天的人如果生成器)](#creating-tables) 来表达式创建和修改表。 例如，让我们看一下创建`october_blog_posts`表的示例迁移：

    <?php namespace Acme\Blog\Updates;

    use Schema;
    use October\Rain\Database\Updates\Migration;

    class CreatePostsTable extends Migration
    {
        public function up()
        {
            Schema::create('october_blog_posts', function($table)
            {
                $table->engine = 'InnoDB';
                $table->increments('id');
                $table->string('title');
                $table->string('slug')->index();
                $table->text('excerpt')->nullable();
                $table->text('content');
                $table->timestamp('published_at')->nullable();
                $table->boolean('is_published')->default(false);
                $table->timestamps();
            });
        }

        public function down()
        {
            Schema::drop('october_blog_posts');
        }
    }

<a name="creating-tables"></a>
### 创建表

要创建新的数据库表，请在`Schema`facade上使用`create`方法。 `create`方法接受两个参数。 第一个是表的名称，而第二个是`Closure`，它接收用于定义新表的对象：

    Schema::create('users', function ($table) {
        $table->increments('id');
    });

当然，在创建表时，您可以使用任何模式构建器的[列方法](#creating-columns) 来定义表的列。

#### 重命名/删除表

您可以使用`hasTable`和`hasColumn`方法轻松检查是否存在表或列：

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### 连接和存储引擎

如果要对不是默认连接的数据库连接执行模式操作，请使用`connection`方法：

    Schema::connection('foo')->create('users', function ($table) {
        $table->increments('id');
    });

要为表设置存储引擎，请在架构构建器上设置`engine`属性：

    Schema::create('users', function ($table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });

<a name="renaming-and-dropping-tables"></a>
### 重命名/删除表

要重命名现有数据库表，请使用`rename`方法：

    Schema::rename($from, $to);

要删除现有表，可以使用`drop`或`dropIfExists`方法：

    Schema::drop('users');

    Schema::dropIfExists('users');

<a name="creating-columns"></a>
### 创建列

要更新现有表，我们将在`Schema`facade上使用`table`方法。 与`create`方法一样，`table`方法接受两个参数，即表的名称和一个`Closure`，它接收一个我们可以用来向表中添加列的对象：

    Schema::table('users', function ($table) {
        $table->string('email');
    });

####可用的列类型

当然，schema生成器包含构建表时可能使用的各种列类型：

命令  | 描述
------------- | -------------
`$table->bigIncrements('id');`  |  使用“UNSIGNED BIG INTEGER”等效增加ID(主键)。
`$table->bigInteger('votes');`  |  对应数据库中的BIGINT类型 
`$table->binary('data');`  |  对应数据库中的BLOB类型。
`$table->boolean('confirmed');`  | 对应数据库中的 BOOLEAN类型。
`$table->char('name', 4);`  | 对应数据库中的长度为4的CHAR类型。
`$table->date('created_at');`  |  对应数据库中的DATE类型。
`$table->dateTime('created_at');`  |  对应数据库中的DATETIME类型。
`$table->decimal('amount', 5, 2);`  |  对应数据库中的DECIMAL精度和比例。
`$table->double('column', 15, 8);`  |  对应数据库中的DOUBLE精度，总共15位，小数点后8位。
`$table->enum('choices', ['foo', 'bar']);` | 对应数据库中的ENUM类型。
`$table->float('amount');`  |  对应数据库中的FLOAT类型。
`$table->increments('id');`  |  使用“UNSIGNED INTEGER”等效增加ID(主键)。
`$table->integer('votes');`  |  对应数据库中的INTEGER类型。
`$table->json('options');`  |  对应数据库中的JSON类型。
`$table->jsonb('options');`  |  对应数据库中的JSONB类型。
`$table->longText('description');`  |  对应数据库中的LONGTEXT类型。
`$table->mediumInteger('numbers');`  | 对应数据库中的MEDIUMINT类型。
`$table->mediumText('description');`  | 对应数据库中的MEDIUMTEXT类型。.
`$table->morphs('taggable');`  |  添加INTEGER类型的`taggable_id`列和STRING类型的`taggable_type`列。
`$table->nullableTimestamps();`  |  与`timestamps()`相同，但允许NULL。
`$table->rememberToken();`  |  将`remember_token`添加为VARCHAR(100)NULL。
`$table->smallInteger('votes');`  |  对应数据库中的SMALLINT类型。
`$table->softDeletes();`  |  为软删除添加`deleted_at`列。
`$table->string('email');`  |  对应数据库中的VARCHAR类型。
`$table->string('name', 100);`  |  对应数据库中的VARCHAR类型长度为100的字符串。
`$table->text('description');`  |  对应数据库中的TEXT类型。
`$table->time('sunrise');`  |  对应数据库中的TIME类型。
`$table->tinyInteger('numbers');`  |  对应数据库中的TINYINT类型。
`$table->timestamp('added_on');`  |  对应数据库中的TIMESTAMP类型。
`$table->timestamps();`  |  添加 `created_at` 和 `updated_at` 列。

#### 列修饰符

除了上面列出的列类型之外，还可以在添加列时使用其他几个列“修饰符”。 例如，要使列“可为空”，您可以使用`nullable`方法：

    Schema::table('users', function ($table) {
        $table->string('email')->nullable();
    });

下面是所有可用列修饰符的列表。 此列表不包括[索引修饰符](#creating-indexes)：

方法  | 描述
------------- | -------------
`->nullable()`  | 允许将NULL值插入列中
`->default($value)`  |  为列指定“默认”值
`->unsigned()`  |  将`integer`列设置为`UNSIGNED`
`->first()`  |  将列放在表的首列(仅限MySQL)
`->after('column')`  |  将列放在某一列之后(仅限MySQL)

<a name="modifying-columns"></a>
### 修改列

`change`方法允许您将现有列修改为新类型，或修改列的属性。 例如，您可能希望增加字符串列的大小。 使用`change`方法，让我们将`name`列的大小从25增加到50：

    Schema::table('users', function ($table) {
        $table->string('name', 50)->change();
    });

我们还可以将列修改为可为空：

    Schema::table('users', function ($table) {
        $table->string('name', 50)->nullable()->change();
    });

<a name="renaming-columns"></a>
#### 重命名列

要重命名列，可以在Schema上使用`renameColumn`方法：

    Schema::table('users', function ($table) {
        $table->renameColumn('from', 'to');
    });

> **注意:** 目前不支持使用`enum`列重命名表中的列。

<a name="dropping-columns"></a>
### 删除列

要删除列，请使用Schema上的“dropColumn”方法：

    Schema::table('users', function ($table) {
        $table->dropColumn('votes');
    });

您可以通过将列名组成的数组传递给`dropColumn`方法从表中删除多个列：

    Schema::table('users', function ($table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

<a name="creating-indexes"></a>
### 创建索引

架构构建器支持多种类型的索引。 首先，让我们看一个指定列的值应该是唯一的示例。 要创建索引，我们可以简单地将`unique`方法链接到列定义：

    $table->string('email')->unique();

或者，您可以在定义列后创建索引。 例如：

    $table->unique('email');

您甚至可以将一组列传递给索引方法以创建复合索引：

    $table->index(['account_id', 'created_at']);

在大多数情况下，您应手动指定索引的名称作为第二个参数，以避免系统自动生成太长的索引：

    $table->index(['account_id', 'created_at'], 'account_created');

#### 可用的索引类型

命令  | 描述
------------- | -------------
`$table->primary('id');`  |  添加主键。
`$table->primary(['first', 'last']);`  |  添加复合键。
`$table->unique('email');`  |  添加唯一索引。
`$table->index('state');`  |  添加基本索引。

<a name="dropping-indexes"></a>
### 删除索引

要删除索引，必须指定索引的名称。 如果没有手动指定名称，系统将自动生成一个名称，只需连接表名，索引列的名称和索引类型。 这里有些例子：

命令  | 描述
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  从“users”表中删除主键。
`$table->dropUnique('users_email_unique');`  |  从“users”表中删除唯一索引。
`$table->dropIndex('geo_state_index');`  |  从“geo”表中删除基本索引。

<a name="foreign-key-constraints"></a>
### 外键约束

还支持创建外键约束，这些约束用于强制数据库级别的引用完整性。 例如，让我们在`posts`表上定义一个`user_id`列，引用`users`表上的`id`列：

    Schema::table('posts', function ($table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });

和刚才一样，您可以通过将第二个参数传递给`foreign`方法来手动指定约束的名称：

    $table->foreign('user_id', 'user_foreign')
        ->references('id')
        ->on('users');

您还可以为约束的“on delete”和“on update”属性指定所需的操作：

    $table->foreign('user_id')
          ->references('id')
          ->on('users')
          ->onDelete('cascade');

要删除外键，可以使用`dropForeign`方法。 外键约束使用与索引相同的命名约定。 因此，如果没有手动指定，我们将连接表名和约束中的列，然后将名称后缀为“_foreign”：

    $table->dropForeign('posts_user_id_foreign');

<a name="seeder-structure"></a>
## 填充结构

与迁移文件一样，填充类默认只包含一个方法：`run`并应扩展`Seeder`类。 执行更新过程时会调用`run`方法。 在此方法中，您可以根据需要将数据插入数据库。 您可以使用[query builder](database-query.md)手动插入数据，也可以使用[model classes](database-model.md)。 在下面的示例中，我们将使用`run`方法中的`User`模型创建一个新用户：

    <?php namespace Acme\Users\Updates;

    use Seeder;
    use Acme\Users\Models\User;

    class SeedUsersTable extends Seeder
    {
        public function run()
        {
            $user = User::create([
                'email'                 => 'user@example.com',
                'login'                 => 'user',
                'password'              => 'password123',
                'password_confirmation' => 'password123',
                'first_name'            => 'Actual',
                'last_name'             => 'Person',
                'is_activated'          => true
            ]);
        }
    }

或者，使用`Db::table` [query builder](database-query.md) 方法可以实现相同的目的：

    public function run()
    {
        $user = Db::table('users')->insert([
            'email'                 => 'user@example.com',
            'login'                 => 'user',
            [...]
        ]);
    }

<a name="calling-additional-seeders"></a>
### 其他填充器

在`DatabaseSeeder`类中，您可以使用`call`方法来执行其他填充类。 使用`call`方法可以将数据库填充分解为多个文件，这样就不会有单个的填充类变得非常大。 只需传递您希望运行的填充类的名称：

    /**
     * 执行数据库填充
     *
     * @return void
     */
    public function run()
    {
        Model::unguard();

        $this->call('Acme\Users\Updates\UserTableSeeder');
        $this->call('Acme\Users\Updates\PostsTableSeeder');
        $this->call('Acme\Users\Updates\CommentsTableSeeder');
    }
