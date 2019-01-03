# 数据库关系

- [介绍](#introduction)
- [定义关系](#defining-relationships)
    - [详细定义](#detailed-relationships)
- [关系类型](#relationship-types)
    - [一对一](#one-to-one)
    - [一对多](#one-to-many)
    - [多对多](#many-to-many)
    - [Has Many Through](#has-many-through)
    - [多态关系](#polymorphic-relations)
    - [多对多的多态关系](#many-to-many-polymorphic-relations)
- [关系查询](#querying-relations)
    - [通过关系方法访问](#querying-method)
    - [通过动态属性访问](#querying-dynamic-property)
    - [查询关系存在](#querying-existence)
- [Eager loading](#eager-loading)
    - [Constraining eager loads](#constraining-eager-loads)
    - [Lazy eager loading](#lazy-eager-loading)
- [插入相关模型](#inserting-related-models)
    - [通过关系方法插入](#inserting-method)
    - [通过动态属性插入](#inserting-dynamic-property)
    - [多对多的关系](#inserting-many-to-many-relations)
    - [触摸父时间戳](#touching-parent-timestamps)
- [延期绑定](#deferred-binding)

<a name="introduction"></a>
## 介绍

数据库表通常彼此相关。 例如，博客文章可能有很多评论，或者订单可能与放置它的用户有关。 October使管理和处理这些关系变得容易，并支持几种不同类型的关系。

> **注意:** 如果要在查询中选择特定列并且还要加载关系，则需要确保包含键控数据的列(即`id`，`foreign_key`等)包含在select语句中。 否则，October无法连接关系。

<a name="defining-relationships"></a>
## 定义关系

模型关系定义为模型类的属性。 定义关系的一个例子：

    class User extends Model
    {
        public $hasMany = [
            'posts' => 'Acme\Blog\Models\Post'
        ]
    }

像模型本身这样的关系也可以作为强大的[查询构建器](database-query.md)，访问关系作为函数提供强大的方法链接和查询功能。 例如：

    $user->posts()->where('is_active', true)->get();

关联关系也可以通过属性的方式访问

    $user->posts;

<a name="detailed-relationships"></a>
### 详细定义

每个定义都可以是一个数组，其中键是关系名称，值是详细数组。 detail数组的第一个值始终是相关的模型类名，所有其他值都是必须具有键名的参数。

    public $hasMany = [
        'posts' => ['Acme\Blog\Models\Post', 'delete' => true]
    ];

以下是可与所有关系一起使用的参数：

参数 | 描述
------------- | -------------
**order** | 排序多个记录的顺序。
**conditions** | 使用raw where查询语句过滤关系。
**scope** | 使用提供的范围方法过滤关系。
**push** | 如果设置为false，则不会通过`push`保存此关系，默认值为：true。
**delete** | 如果设置为true，则删除主模型或删除关系时将删除相关模型，默认值为false。
**count** | 如果设置为true，则结果仅包含`count`列，用于计算关系，默认值：false。

使用 **order**和**condition**的示例过滤器：

    public $belongsToMany = [
        'categories' => [
            'Acme\Blog\Models\Category',
            'order'      => 'name desc',
            'conditions' => 'is_active = 1'
        ]
    ];

使用**scope**的示例过滤器：

    class Post extends Model
    {
        public $belongsToMany = [
            'categories' => [
                'Acme\Blog\Models\Category',
                'scope' => 'isActive'
            ]
        ];
    }

    class Category extends Model
    {
        public function scopeIsActive($query)
        {
            return $query->where('is_active', true)->orderBy('name', 'desc');
        }
    }

使用**count**的示例过滤器：

    public $belongsToMany = [
        'users' => ['Backend\Models\User'],
        'users_count' => ['Backend\Models\User', 'count' => true]
    ];

<a name="relationship-types"></a>
## 关系类型

可以使用以下关系类型：

- [一对一](#one-to-one)
- [一对多](#one-to-many)
- [多对多](#many-to-many)
- [Has Many Through](#has-many-through)
- [多态关系](#polymorphic-relations)
- [多对多的多态关系](#many-to-many-polymorphic-relations)

<a name="one-to-one"></a>
### 一对一

一对一的关系是一种非常基本的关系。 例如，“用户”模型可能与一个“电话”相关联。 为了定义这种关系，我们在`User`模型的`$hasOne`属性中添加一个`phone`条目。

    <?php namespace Acme\Blog\Models;

    use Model;

    class User extends Model
    {
        public $hasOne = [
            'phone' => 'Acme\Blog\Models\Phone'
        ];
    }

一旦定义了关系，我们就可以使用相同名称的model属性检索相关记录。 这些属性是动态的，允许您像访问模型上的常规属性一样访问它们：

    $phone = User::find(1)->phone;

该模型基于模型名称假定关系的外键。 在这种情况下，自动假设`Phone`模型具有`user_id`外键。 如果要覆盖此约定，可以将`key`参数传递给定义：

    public $hasOne = [
        'phone' => ['Acme\Blog\Models\Phone', 'key' => 'my_user_id']
    ];

此外，该模型假定外键应具有与父级的“id”列匹配的值。 换句话说，它将在`Phone`记录的`user_id`列中查找用户的`id`列的值。 如果您希望关系使用除`id`以外的值，则可以将`otherKey`参数传递给定义：

    public $hasOne = [
        'phone' => ['Acme\Blog\Models\Phone', 'key' => 'my_user_id', 'otherKey' => 'my_id']
    ];

#### 定义关系的倒数

现在我们可以从`User`访问`Phone`模型了。 让我们做相反的事情并在`Phone`模型上定义一个关系，让我们访问拥有手机的`User`。 我们可以使用`$belongsTo`属性定义`hasOne`关系的反转：

    class Phone extends Model
    {
        public $belongsTo = [
            'user' => 'Acme\Blog\Models\User'
        ];
    }

在上面的例子中，模型将尝试匹配`Phone`模型中的`user_id`和`User`模型上的`id`。 它通过检查关系定义的名称并使用`_id`为名称添加后缀来确定默认外键名称。 但是，如果`Phone`模型上的外键不是`user_id`，您可以使用定义上的`key`参数传递自定义键名：

    public $belongsTo = [
        'user' => ['Acme\Blog\Models\User', 'key' => 'my_user_id']
    ];

如果您的父模型不使用`id`作为其主键，或者您希望将子模型连接到不同的列，则可以将`otherKey`参数传递给定义父表的自定义键的定义：

    public $belongsTo = [
        'user' => ['Acme\Blog\Models\User', 'key' => 'my_user_id', 'otherKey' => 'my_id']
    ];

#### 默认模型

`belongsTo`关系允许您定义一个默认模型，如果给定的关系为“null”，将返回该默认模型。 此模式通常称为[空对象模式](https://en.wikipedia.org/wiki/Null_Object_pattern)，可以帮助删除代码中的条件检查。 在下面的示例中，如果没有“user”附加到帖子，`user`关系将返回一个空的`Acme\Blog\Models\User`模型：

    public $belongsTo = [
        'user' => ['Acme\Blog\Models\User', 'default' => true]
    ];

要使用属性填充默认模型，可以将数组传递给`default`参数：

    public $belongsTo = [
        'user' => [
            'Acme\Blog\Models\User',
            'default' => ['name' => 'Guest']
        ]
    ];

<a name="one-to-many"></a>
### 一对多

一对多关系用于定义单个模型拥有任何数量的其他模型的关系。 例如，博客文章可能有无数的评论。 与所有其他关系一样，定义了一对多关系，在模型上的`$hasMany`属性中添加一个条目：

    class Post extends Model
    {
        public $hasMany = [
            'comments' => 'Acme\Blog\Models\Comment'
        ];
    }

请记住，模型将自动确定`Comment`模型上的正确外键列。 按照惯例，它将采用拥有模型的`下划线命名`名称，并以`_id`为后缀。 因此，对于这个例子，我们可以假设`Comment`模型上的外键是`post_id`。

一旦定义了关系，我们就可以通过访问`comments`属性来访问`Comment`集合。 请记住，由于模型提供了“动态属性”，我们可以像访问模型中的属性一样访问关系：

    $comments = Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

当然，由于所有关系也充当查询构建器，您可以通过调用`comments`方法并继续将条件链接到查询来添加更多约束以检索哪些`Comment`：

    $comments = Post::find(1)->comments()->where('title', 'foo')->first();

与`hasOne`关系一样，您也可以通过分别在定义上传递`key`和`otherKey`参数来覆盖外键和本地键：

    public $hasMany = [
        'comments' => ['Acme\Blog\Models\Comment', 'key' => 'my_post_id', 'otherKey' => 'my_id']
    ];

#### 定义关系的倒数

现在我们可以访问所有帖子的评论，让我们定义一个关系，以允许评论访问其父帖子。 要定义`hasMany`关系的反转，请在子模型上定义`$belongsTo`属性：

    class Comment extends Model
    {
        public $belongsTo = [
            'post' => 'Acme\Blog\Models\Post'
        ];
    }

一旦定义了关系，我们就可以通过访问`post`“动态属性”来检索`Comment`的`Post`模型：

    $comment = Comment::find(1);

    echo $comment->post->title;

在上面的例子中，模型将尝试将`Comment`模型中的`post_id`与`Post`模型上的`id`相匹配。 它通过检查关系的名称并使用`_id`对其进行后缀来确定默认外键名称。 但是，如果`Comment`模型上的外键不是`post_id`，则可以使用`key`参数传递自定义键名：

    public $belongsTo = [
        'post' => ['Acme\Blog\Models\Post', 'key' => 'my_post_id']
    ];

如果您的父模型不使用`id`作为其主键，或者您希望将子模型连接到不同的列，则可以将`otherKey`参数传递给定义父表的自定义键的定义：

    public $belongsTo = [
        'post' => ['Acme\Blog\Models\Post', 'key' => 'my_post_id', 'otherKey' => 'my_id']
    ];

<a name="many-to-many"></a>
### 多对多

多对多关系比`hasOne`和`hasMany`关系稍微复杂一些。 这种关系的一个示例是具有许多角色的用户，其中角色也由其他用户共享。 例如，许多用户可能具有“管理员”的角色。 要定义这种关系，需要三个数据库表：`users`，`roles`和`role_user`。 `role_user`表是从相关模型名称的字母顺序派生而来的，包含`user_id`和`role_id`列。

下面的示例显示了用于创建连接表的[数据库表结构](plugin-updates.md#migration-files) 。

    Schema::create('role_user', function($table)
    {
        $table->integer('user_id')->unsigned();
        $table->integer('role_id')->unsigned();
        $table->primary(['user_id', 'role_id']);
    });

定义了多对多关系，在模型类的`$belongsToMany`属性中添加一个条目。 例如，让我们在`User`模型上定义`roles`方法：

    class User extends Model
    {
        public $belongsToMany = [
            'roles' => 'Acme\Blog\Models\Role'
        ];
    }

定义关系后，您可以使用`roles`动态属性访问用户的角色：

    $user = User::find(1);

    foreach ($user->roles as $role) {
        //
    }

当然，像所有其他关系类型一样，您可以调用`roles`方法继续将查询约束链接到关系上：

    $roles = User::find(1)->roles()->orderBy('name')->get();

如前所述，为了确定关系的连接表的表名，模型将按字母顺序连接两个相关的模型名称。 但是，您可以自由地覆盖此约定。 您可以通过将`table`参数传递给`belongsToMany`定义来实现：

    public $belongsToMany = [
        'roles' => ['Acme\Blog\Models\Role', 'table' => 'acme_blog_role_user']
    ];

除了自定义连接表的名称之外，您还可以通过将其他参数传递给`belongsToMany`定义来自定义表上键的列名。 `key`参数是您定义关系的模型的外键名称，而`otherKey`参数是您要加入的模型的外键名称：

    public $belongsToMany = [
        'roles' => [
            'Acme\Blog\Models\Role',
            'table'    => 'acme_blog_role_user',
            'key'      => 'my_user_id',
            'otherKey' => 'my_role_id'
        ]
    ];

#### 定义关系的倒数

要定义多对多关系的反转，只需在相关模型上放置另一个`$belongsToMany`属性。 要继续我们的用户角色示例，让我们在`Role`模型上定义`users`关系：

    class Role extends Model
    {
        public $belongsToMany = [
            'users' => 'Acme\Blog\Models\User'
        ];
    }

正如您所看到的，该关系的定义与其“User”对应关系完全相同，只是简单地引用了`Acme\Blog\Models\User` 模型。 由于我们正在重用`$belongsToMany`属性，因此在定义多对多关系的反转时，所有常用的表和键自定义选项都可用。

#### 检索中间表列

正如您已经了解的那样，处理多对多关系需要存在中间连接表。 模型提供了一些与此表交互的非常有用的方法。 例如，假设我们的`User`对象有许多与它相关的`Role`对象。 访问此关系后，我们可以使用模型上的`pivot`属性访问中间表：

    $user = User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

请注意，我们检索的每个`Role`模型都会自动分配一个`pivot`属性。 此属性包含表示中间表的模型，可以像任何其他模型一样使用。

默认情况下，只有模型键才会出现在`pivot`对象上。 如果数据透视表包含额外属性，则必须在定义关系时指定它们：

    public $belongsToMany = [
        'roles' => [
            'Acme\Blog\Models\Role',
            'pivot' => ['column1', 'column2']
        ]
    ];

如果您希望数据透视表自动维护`created_at`和`updated_at`时间戳，请在关系定义中使用`timestamps`参数：

    public $belongsToMany = [
        'roles' => ['Acme\Blog\Models\Role', 'timestamps' => true]
    ];

这些是`belongsToMany`关系支持的参数：

参数 | 描述
------------- | -------------
**table** | 连接表的名称。
**key** | 定义模型的键列名称。
**otherKey** | 相关模型的键列名称。
**pivot** | 在连接表中找到的数据透视列数组，属性可通过`$model-> pivot`获得。
**pivotModel** | 指定在访问数据透视关系时要返回的自定义模型类。 默认为`October\Rain\Database\Pivot`。
**timestamps** | 如果为true，则连接表应包含`created_at`和`updated_at`列。 默认值：false

<a name="has-many-through"></a>
### Has Many Through

has-many-through关系为通过中间关系访问远程关系提供了方便的捷径。 例如，“Country”模型可能通过中间“User”模型具有许多“Post”模型。 在此示例中，您可以轻松收集给定国家/地区的所有博客帖子。 让我们看一下定义这种关系所需的表：

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

虽然`posts`不包含`country_id`列，但`hasManyThrough`关系通过`$country-> posts`提供对国家帖子的访问。 要执行此查询，模型会检查中间`users`表上的`country_id`。 找到匹配的用户ID后，它们用于查询`posts`表。

现在我们已经检查了关系的表结构，让我们在`Country`模型上定义它：

    class Country extends Model
    {
        public $hasManyThrough = [
            'posts' => [
                'Acme\Blog\Models\Post',
                'through' => 'Acme\Blog\Models\User'
            ],
        ];
    }

传递给`$hasManyThrough`关系的第一个参数是我们希望访问的最终模型的名称，而`through`参数是中间模型的名称。

执行关系查询时将使用典型的外键约定。 如果您想自定义关系的键，可以将它们作为`key`和`throughKey`参数传递给`$hasManyThrough`定义。 `key`参数是中间模型上的外键的名称，而`throughKey`参数是最终模型上的外键的名称。

    public $hasManyThrough = [
        'posts' => [
            'Acme\Blog\Models\Post',
            'key'        => 'my_country_id',
            'through'    => 'Acme\Blog\Models\User',
            'throughKey' => 'my_user_id'
        ],
    ];

<a name="polymorphic-relations"></a>
### 多态关系

#### 表结构

多态关系允许模型在单个关联上属于多个其他模型。 例如，假设您要为员工和产品存储照片。 使用多态关系，您可以为这两种情况使用单个“照片”表。 首先，让我们检查构建这种关系所需的表结构：

    staff
        id - integer
        name - string

    products
        id - integer
        price - integer

    photos
        id - integer
        path - string
        imageable_id - integer
        imageable_type - string

需要注意的两个重要的列是`photos`表上的`imageable_id`和`imageable_type`列。 `imageable_id`列将包含拥有人员或产品的ID值，而`imageable_type`列将包含拥有模型的类名。 `imageable_type`列是ORM如何确定在访问`imageable`关系时要返回的拥有模型的“类型”。

#### 模型结构

接下来，让我们检查构建此关系所需的模型定义：

    class Photo extends Model
    {
        public $morphTo = [
            'imageable' => []
        ];
    }

    class Staff extends Model
    {
        public $morphMany = [
            'photos' => ['Acme\Blog\Models\Photo', 'name' => 'imageable']
        ];
    }

    class Product extends Model
    {
        public $morphMany = [
            'photos' => ['Acme\Blog\Models\Photo', 'name' => 'imageable']
        ];
    }

#### 检索多态关系

一旦定义了数据库表和模型，您就可以通过模型访问关系。 例如，要访问工作人员的所有照片，我们可以简单地使用`photos`动态属性：

    $staff = Staff::find(1);

    foreach ($staff->photos as $photo) {
        //
    }

您还可以通过访问`morphTo`关系的名称从多态模型中检索多态关系的所有者。 在我们的例子中，这是`Photo`模型中的`imageable`定义。 因此，我们将其作为动态属性访问：

    $photo = Photo::find(1);

    $imageable = $photo->imageable;

`Photo`模型上的`imageable`关系将返回`Staff`或`Product`实例，具体取决于拥有该照片的模型类型。

#### 自定义多态类型

默认情况下，完全限定的类名用于存储相关的模型类型。 例如，给定上面的示例，其中`Photo`可能属于`Staff`或`Product`，默认的`imageable_type`值是'Acme\Blog\Models\Staff`或`Acme\Blog\Models\Product `分别。

使用自定义多态类型可以将数据库与应用程序的内部结构分离。 您可以定义关系“变形图”以为每个模型而不是类名提供自定义名称：

    use October\Rain\Database\Relations\Relation;

    Relation::morphMap([
        'staff' => 'Acme\Blog\Models\Staff',
        'product' => 'Acme\Blog\Models\Product',
    ]);

在[插件注册文件](plugin-registration.md#registration-methods)的`boot`方法中注册`morphMap`的最常见的地方。

<a name="many-to-many-polymorphic-relations"></a>
### 多对多的多态关系

#### 表结构

除了传统的多态关系，您还可以定义“多对多”多态关系。 例如，博客“Post”和“Video”模型可以与“Tag”模型共享多态关系。 使用多对多多态关系，您可以拥有一个在博客帖子和视频中共享的唯一标记列表。 首先，让我们检查表结构：

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

#### 模型结构

接下来，我们准备定义模型上的关系。 `Post`和`Video`模型都将在基础模型类的`$morphToMany`属性中定义`tags`关系：

    class Post extends Model
    {
        public $morphToMany = [
            'tags' => ['Acme\Blog\Models\Tag', 'name' => 'taggable']
        ];
    }

#### 定义关系的倒数

接下来，在`Tag`模型上，您应该为每个相关模型定义关系。 因此，对于这个例子，我们将定义一个`posts`关系和一个`videos`关系：

    class Tag extends Model
    {
        public $morphedByMany = [
            'posts'  => ['Acme\Blog\Models\Post', 'name' => 'taggable'],
            'videos' => ['Acme\Blog\Models\Video', 'name' => 'taggable']
        ];
    }

#### 检索关系

一旦定义了数据库表和模型，您就可以通过模型访问关系。 例如，要访问帖子的所有标签，您只需使用`tags`动态属性：

    $post = Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

您还可以通过访问`$morphedByMany`属性中定义的关系名称，从多态模型中检索多态关系的所有者。 在我们的例子中，这是`Tag`模型上的`posts`或`videos`方法。 因此，您将作为动态属性访问这些关系：

    $tag = Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## 查询关系

由于可以通过函数调用所有类型的模型关系，因此可以调用这些函数来获取关系的实例，而无需实际执行关系查询。 此外，所有类型的关系也可用作[查询构建器](database-query.md)，允许您在最终针对数据库执行SQL之前继续将约束链接到关系查询。

For example, imagine a blog system in which a `User` model has many associated `Post` models:

    class User extends Model
    {
        public $hasMany = [
            'posts' => ['Acme\Blog\Models\Post']
        ];
    }

<a name="querying-method"></a>
### 通过关系方法访问

您可以使用`posts`方法查询** posts **关系并为关系添加其他约束。 这使您能够链接任何[查询构建器](database-query.md)方法的关系。

    $user = User::find(1);

    $posts = $user->posts()->where('is_active', 1)->get();

    $post = $user->posts()->first();

<a name="querying-dynamic-property"></a>
### 通过动态属性访问

如果您不需要为关系查询添加其他约束，则可以简单地访问该关系，就像它是属性一样。 例如，继续使用我们的`User`和`Post`示例模型，我们可以使用`$user-> posts`属性访问所有用户的帖子。

    $user = User::find(1);

    foreach ($user->posts as $post) {
        //...
    }

动态属性是“延迟加载”，这意味着它们只会在您实际访问它们时加载它们的关系数据。 因此，开发人员经常使用[eager loading](#eager-loading)来预加载他们知道将在加载模型后访问的关系。 预先加载可显着减少必须执行以加载模型关系的SQL查询。

<a name="querying-existence"></a>
### 查询关系存在

访问模型的记录时，您可能希望根据关系的存在来限制结果。 例如，假设您要检索至少有一条评论的所有博客帖子。 为此，您可以将关系的名称传递给`has`方法：

    //Retrieve all posts that have at least one comment...
    $posts = Post::has('comments')->get();

您还可以指定运算符并计数以进一步自定义查询：

    //Retrieve all posts that have three or more comments...
    $posts = Post::has('comments', '>=', 3)->get();

嵌套的`has`语句也可以使用“点”表示法构造。 例如，您可以检索至少有一条评论和投票的所有帖子：

    //Retrieve all posts that have at least one comment with votes...
    $posts = Post::has('comments.votes')->get();

如果你需要更多的功能，你可以使用`whereHas`和`orWhereHas`方法在你的`has`查询中放置“where”条件。 这些方法允许您向关系约束添加自定义约束，例如检查`Comment`的内容：

    //Retrieve all posts with at least one comment containing words like foo%
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="eager-loading"></a>
## Eager loading

当作为属性访问关系时，关系数据是“延迟加载”。 这意味着在您首次访问该属性之前，实际上不会加载关系数据。 但是，在查询父模型时，模型可以“急切加载”关系。 急切加载缓解了N + 1查询问题。 为了说明N + 1查询问题，请考虑与`Author`相关的`Book`模型：

    class Book extends Model
    {
        public $belongsTo = [
            'author' => ['Acme\Blog\Models\Author']
        ];
    }

现在让我们检索所有书籍及其作者：

    $books = Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

此循环将执行1个查询以检索表中的所有书籍，然后对每个书籍执行另一个查询以检索作者。 因此，如果我们有25本书，这个循环将运行26个查询：1个用于原始书籍，25个额外查询用于检索每本书的作者。

值得庆幸的是，我们可以使用预先加载将此操作减少到只有2个查询。 查询时，您可以使用`with`方法指定应该急切加载哪些关系：

    $books = Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

对于此操作，将只执行两个查询：

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### Eager loading multiple relationships

有时您可能需要在单个操作中急切地加载几个不同的关系。 为此，只需将其他参数传递给`with`方法：

    $books = Book::with('author', 'publisher')->get();

#### Nested eager loading

要急切加载嵌套关系，您可以使用“点”语法。 例如，让我们在一个声明中急切地加载本书的所有作者和作者的所有个人联系人：

    $books = Book::with('author.contacts')->get();

<a name="constraining-eager-loads"></a>
### Constraining eager loads

有时您可能希望加载关系，但也为热切加载查询指定其他查询约束。 这是一个例子：

    $users = User::with([
        'posts' => function ($query) {
            $query->where('title', 'like', '%first%');
        }
    ])->get();

在这个例子中，如果帖子的`title`列包含单词`first`，模型将只会加载帖子。 当然，您可以调用其他[查询构建器](database-query.md)方法来进一步自定义预先加载操作：

    $users = User::with([
        'posts' => function ($query) {
            $query->orderBy('created_at', 'desc');
        }
    ])->get();

<a name="lazy-eager-loading"></a>
### Lazy eager loading

有时您可能需要在检索到父模型后急切加载关系。 例如，如果您需要动态决定是否加载相关模型，这可能很有用：

    $books = Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

如果需要在预先加载的查询上设置其他查询约束，可以将`Closure`传递给`load`方法：

    $books->load([
        'author' => function ($query) {
            $query->orderBy('published_date', 'asc');
        }
    ]);

<a name="inserting-related-models"></a>
## 插入相关模型

就像[查询关系](#querying-relations)一样，October支持使用方法或动态属性方法定义关系。 例如，也许你需要为`Post`模型插入一个新的`Comment`。 您可以直接从关系中插入“Comment”，而不是在“Comment”上手动设置`post_id`属性。

<a name="inserting-method"></a>
### 通过关系方法插入

October提供了为关系添加新模型的便捷方法。 主要模型可以添加到关系中或从关系中删除。 在每种情况下，关系分别相关联或解除关联。

#### 添加方法

使用`add`方法关联新关系。

    $comment = new Comment(['message' => 'A new comment.']);

    $post = Post::find(1);

    $comment = $post->comments()->add($comment);

请注意，我们没有将`comments`关系作为动态属性访问。 相反，我们调用`comments`方法来获取关系的实例。 `add`方法会自动将相应的`post_id`值添加到新的`Comment`模型中。

如果需要保存多个相关模型，可以使用`addMany`方法：

    $post = Post::find(1);

    $post->comments()->addMany([
        new Comment(['message' => 'A new comment.']),
        new Comment(['message' => 'Another comment.']),
    ]);

#### 删除方法

相比之下，`remove`方法可用于解除关系，使其成为孤立的记录。

    $post->comments()->remove($comment);

在多对多关系的情况下，记录将从关系的集合中删除。

    $post->categories()->remove($category);

在“属于”关系的情况下，您可以使用`dissociate`方法，该方法不需要传递给它的相关模型。

    $post->author()->dissociate();

#### 添加透视数据

当使用多对多关系时，`add`方法接受一组额外的中间“pivot”表属性作为其第二个参数作为数组。

    $user = User::find(1);

    $pivotData = ['expires' => $expires];

    $user->roles()->add($role, $pivotData);

`add`方法的第二个参数也可以指定[deferred binding](#deferred-binding)在作为字符串传递时使用的会话密钥。 在这些情况下，可以将透视数据作为第三个参数提供。

    $user->roles()->add($role, $sessionKey, $pivotData);

#### 创建方法

虽然`add`和`addMany`接受一个完整的模型实例，你也可以使用`create`方法，该方法接受PHP属性数组，创建模型并将其插入数据库。

    $post = Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

在使用`create`方法之前，请务必查看属性[mass assignment](database-model.md#mass-assignment)的文档，因为PHP数组中的属性受模型的“可填充”定义的限制。

<a name="inserting-dynamic-property"></a>
### 通过动态属性插入

关系可以通过其属性直接设置，就像访问它们一样。 使用此方法设置关系将覆盖先前存在的任何关系。 之后应该像保存任何属性一样保存模型。

    $post->author = $author;

    $post->comments = [$comment1, $comment2];

    $post->save();

或者，您可以使用主键设置关系，这在使用HTML表单时很有用。

    //Assign to author with ID of 3
    $post->author = 3;

    //Assign comments with IDs of 1, 2 and 3
    $post->comments = [1, 2, 3];

    $post->save();

可以通过将NULL值分配给属性来解除关系。

    $post->author = null;

    $post->comments = null;

    $post->save();

与[延迟绑定](#deferred-binding)类似，在不存在的模型上定义的关系在内存中延迟，直到保存为止。 在这个例子中，帖子还不存在，所以不能通过`$post-> comments`在评论中设置`post_id`属性。 因此，关联被推迟到通过调用“save”方法创建帖子。

    $comment = Comment::find(1);

    $post = new Post;

    $post->comments = [$comment];

    $post->save();

<a name="inserting-many-to-many-relations"></a>
### 多对多的关系

#### 附加/分离

在处理多对多关系时，模型提供了一些额外的辅助方法，以便更方便地使用相关模型。 例如，假设用户可以拥有多个角色，而角色可以拥有许多用户。 要通过在连接模型的中间表中插入记录来将角色附加到用户，请使用`attach`方法：

    $user = User::find(1);

    $user->roles()->attach($roleId);

将关系附加到模型时，您还可以传递要插入到中间表中的其他数据数组：

    $user->roles()->attach($roleId, ['expires' => $expires]);

当然，有时可能需要从用户中删除角色。 要删除多对多关系记录，请使用`detach`方法。 `detach`方法将从中间表中删除相应的记录; 但是，两种模型都将保留在数据库中：

    //Detach a single role from the user...
    $user->roles()->detach($roleId);

    //Detach all roles from the user...
    $user->roles()->detach();

为方便起见，`attach`和`detach`也接受ID数组作为输入：

    $user = User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

#### 同步为方便起见

您也可以使用`sync`方法构建多对多关联。 `sync`方法接受要放在中间表上的ID数组。 将从中间表中删除任何不在给定数组中的ID。 因此，在此操作完成后，只有数组中的ID将存在于中间表中：

    $user->roles()->sync([1, 2, 3]);

您还可以使用ID传递其他中间表值：

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

<a name="touching-parent-timestamps"></a>
### 触摸父时间戳

当模型`belongsTo`或`belongsToMany`另一个模型，例如属于`Post`的`Comment`时，有时更新子模型更新时父代的时间戳是有帮助的。 例如，当更新`Comment`模型时，您可能希望自动“触摸”拥有的`Post`的`updated_at`时间戳。 只需添加一个`touches`属性，其中包含子模型的关系名称：

    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
         */
        protected $touches = ['post'];

        /**
         * Relations
         */
        public $belongsTo = [
            'post' => ['Acme\Blog\Models\Post']
        ];
    }

现在，当您更新`Comment`时，拥有的`Post`也会更新其`updated_at`列：

    $comment = Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();

<a name="deferred-binding"></a>
## 延期绑定

延迟绑定允许您推迟模型关系绑定，直到主记录提交更改。 如果您需要准备一些模型(例如文件上载)并将它们与另一个尚不存在的模型相关联，这将特别有用。

您可以使用**session key 会话密钥**将任意数量的 **slave** 模型推迟到 **master** 模型。 当主记录与会话密钥一起保存时，将自动更新与从记录的关系。 后端[表单行为](../backend/form) 会自动支持延迟绑定，但您可能希望在其他位置使用此功能。

<a name="deferred-session-key"></a>
### 生成会话密钥session key

延迟绑定需要会话密钥。 您可以将会话密钥视为事务标识符。 应使用相同的会话密钥来绑定/解除绑定关系并保存主模型。 您可以使用PHP`unitqid()`函数生成会话密钥。 请注意，[表单助手](../cms/markup#forms)会自动生成包含会话密钥的隐藏字段。

    $sessionKey = uniqid('session_key', true);

<a name="defer-binding"></a>
### 推迟关系绑定

除非保存帖子，否则下一个示例中的`Comment`不会添加到帖子中。

    $comment = new Comment;
    $comment->content = "Hello world!";
    $comment->save();

    $post = new Post;
    $post->comments()->add($comment, $sessionKey);

> **注意**: 尚未保存`$post`对象，但如果保存发生，则会创建关系。

<a name="defer-unbinding"></a>
### 推迟关系解除绑定

除非保存帖子，否则不会删除下一个示例中的`Comment`。

    $comment = Comment::find(1);
    $post = Post::find(1);
    $post->comments()->delete($comment, $sessionKey);

<a name="list-all-bindings"></a>
### 列出所有绑定

使用关系的`withDeferred`方法加载所有记录，包括deferred。 结果还将包括现有关系。

    $post->comments()->withDeferred($sessionKey)->get();

<a name="cancel-all-bindings"></a>
### 取消所有绑定

取消延迟绑定并删除从属对象而不是将它们留作孤儿是一个好主意。

    $post->cancelDeferred($sessionKey);

<a name="commit-all-bindings"></a>
### 提交所有绑定

通过为会话密钥提供`save`方法的第二个参数，可以在保存主模型时提交(绑定或取消绑定)所有延迟绑定。

    $post = new Post;
    $post->title = "First blog post";
    $post->save(null, $sessionKey);

相同的方法适用于模型的`create`方法：

    $post = Post::create(['title' => 'First blog post'], $sessionKey);

<a name="lazily-commit-bindings"></a>
### 懒提交绑定

如果在保存时无法提供`$sessionKey`，则可以使用下一个代码随时提交绑定：

    $post->commitDeferred($sessionKey);

<a name="cleanup-bindings"></a>
### 清理孤立的绑定

销毁尚未提交且超过1天的所有绑定：

    October\Rain\Database\Models\DeferredBinding::cleanUp(1);

> **注意:** October会自动销毁超过5天的延迟绑定。 当后端用户登录系统时会发生这种情况。
