# 数据库特征

- [Hashable(可哈希)](#hashable)
- [Purgeable(可清除)](#purgeable)
- [Encryptable(可加密)](#encryptable)
- [Sluggable](#sluggable)
- [Revisionable](#revisionable)
- [Sortable(可排序)](#sortable)
- [简单的树](#simple-tree)
- [嵌套树](#nested-tree)
- [验证](#validation)
- [软删除](#soft-deleting)
- [Nullable(可为空)](#nullable)

Model特征用于实现通用功能。译者注：《Modern PHP(中文版)》书中第23页中，将trait译为`性状`，trait定义为是类的部分实现(即常量，属性和方法)，可以混入一个或者多个现有的PHP类中，性状有两个作用：表明类可以做什么(像是接口)；提供模块化实现(像是类)

<a name="hashable"></a>
## Hashable(可哈希)

在Model实例上首次设置属性时，会立即对哈希属性进行哈希处理。 要在Model实例中散列属性，请应用`October\Rain\Database\Traits\Hashable`特征并使用包含要散列的属性的数组声明`$hashable`属性。

    class User extends Model
    {
        use \October\Rain\Database\Traits\Hashable;

        /**
         * @var array List of attributes to hash.
         */
        protected $hashable = ['password'];
    }

<a name="purgeable"></a>
## Purgeable(可清除)

创建或更新Model实例时，Purged属性不会保存到数据库中。 要清除Model实例中的属性，请应用`October\Rain\Database\Traits\Purgeable`特征并使用包含要清除的属性的数组声明`$purgeable`属性。

    class User extends Model
    {
        use \October\Rain\Database\Traits\Purgeable;

        /**
         * @var array List of attributes to purge.
         */
        protected $purgeable = ['password_confirmation'];
    }

在Model实例被保存之前，在触发[Model事件](#model-events)之前，包括验证，将清除定义的属性。 使用`getOriginalPurgeValue`查找已清除的值。

    return $user->getOriginalPurgeValue('password_confirmation');

<a name="encryptable"></a>
## Encryptable

与[hashable trait](#hashable)类似，加密属性在设置时加密，但在检索属性时也会解密。 要加密Model实例中的属性，请应用`October\Rain\Database\Traits\Encryptable`特征，并使用包含要加密的属性的数组声明`$encryptable`属性。

    class User extends Model
    {
        use \October\Rain\Database\Traits\Encryptable;

        /**
         * @var array 要加密的属性列表。
         */
        protected $encryptable = ['api_key', 'api_secret'];
    }
    
> **注意:** 加密属性将作为加密/解密过程的一部分进行序列化和反序列化。 不要同时创建一个`encryptable`属性[`jsonable`](model#standard-properties) ，因为`jsonable`进程将尝试解码已经被加密器反序列化的值。

<a name="sluggable"></a>
## Sluggable

Slugs是页面URL中常用的有意义的代码。 要为您的Model实例自动生成一个独特的slug，请应用`October\Rain\Database\Traits\Sluggable`特性并声明一个`$slugs`属性。

    class User extends Model
    {
        use \October\Rain\Database\Traits\Sluggable;

        /**
         * @var array 为这些属性生成唯一标示。
         */
        protected $slugs = ['slug' => 'name'];
    }

`$slugs`属性应该是一个数组，其中键是slug的目标列，值是用于生成slug的源字符串。 在上面的例子中，如果`name`列设置为**Cheyenne**，那么`slug`列将被设置为**cheyenne**，**cheyenne-1**或**cheyenne在创建Model实例之前-2**等。

要从多个源生成slug，请将另一个数组作为源值传递：

    protected $slugs = [
        'slug' => ['first_name', 'last_name']
    ];

Slugs仅在首次创建Model实例时生成。 要覆盖或禁用此功能，只需手动设置slug属性：

    $user = new User;
    $user->name = 'Remy';
    $user->slug = 'custom-slug';
    $user->save(); // 不会产生slug

更新Model实例时，使用`slugAttributes`方法重新生成slugs：

    $user = User::find(1);
    $user->slug = null;
    $user->slugAttributes();
    $user->save();

<a name="revisionable"></a>
## Revisionable

OctoberModel实例可以通过存储修订来记录值的变化历史。 要存储Model实例的修订版，请应用`October\Rain\Database\Traits\Revisionable`特征并声明一个`$revisionable`属性，该属性包含一个包含要监视更改的属性的数组。 您还需要定义一个名为`revision_history`的`$morphMany` [Model实例关系](relations)，它引用名为`revisionable`的`System\Models\Revision`类，这是存储修订历史数据的地方。

    class User extends Model
    {
        use \October\Rain\Database\Traits\Revisionable;

        /**
         * @var array 监视这些属性更改记录。
         */
        protected $revisionable = ['name', 'email'];

        /**
         * @var array 关系
         */
        public $morphMany = [
            'revision_history' => ['System\Models\Revision', 'name' => 'revisionable']
        ];
    }

默认情况下，将保留500条记录，但是可以通过在Model实例上声明具有新限制值的`$revisionableLimit`属性来修改此记录。

    /**
     * @var int 要保留的最大修订记录数。
     */
    public $revisionableLimit = 8;

可以像任何其他关系一样访问修订历史记录：

    $history = User::find(1)->revision_history;

    foreach ($history as $record) {
        echo $record->field . ' updated ';
        echo 'from ' . $record->old_value;
        echo 'to ' . $record->new_value;
    }

修订记录可选地使用`user_id`属性支持用户关系。 您可以在Model实例中包含`getRevisionableUser`方法，以跟踪进行修改的用户。

    public function getRevisionableUser()
    {
        return BackendAuth::getUser()->id;
    }

<a name="sortable"></a>
## Sortable

排序Model实例将在“sort_order”中存储一个数值，该值维护集合中每个单独Model实例的排序顺序。 要存储Model实例的排序顺序，请应用`October\Rain\Database\Traits\Sortable`特征并确保您的模式已定义了要使用的列(例如：`$table->integer('sort_order')->default(0);`)。

    class User extends Model
    {
        use \October\Rain\Database\Traits\Sortable;
    }
    

您可以通过定义`SORT_ORDER`常量来修改用于标识排序顺序的键名：

    const SORT_ORDER = 'my_sort_order_column';

使用`setSortableOrder`方法在单个记录或多个记录上设置排序。

    // 将用户的顺序设置为1 ...
    $user->setSortableOrder($user->id, 1);

    // 分别设置记录1,2,3到3,2,1的顺序......
    $user->setSortableOrder([1, 2, 3], [3, 2, 1]);
    
> **注意:** 如果将此特征添加到先前已存在数据(行)的Model实例中，则可能需要初始化该特征，然后该特征才能正常工作。 为此，手动更新每一行的`sort_order`列或对数据运行查询，将记录的`id`列复制到`sort_order`列(例如`UPDATE myvendor_myplugin_mymodelrecords SET sort_order = id`)。

<a name="simple-tree"></a>
## Simple Tree

一个简单的树Model实例将使用`parent_id`列维护Model实例之间的父子关系。 要使用简单树，请应用`October\Rain\Database\Traits\SimpleTree`特征。

    class Category extends Model
    {
        use \October\Rain\Database\Traits\SimpleTree;
    }

这个特性会自动注入两个名为`parent`和`children`的[Model实例关系](../database/relations)，它相当于以下定义：

    public $belongsTo = [
        'parent'    => ['User', 'key' => 'parent_id'],
    ];

    public $hasMany = [
        'children'    => ['User', 'key' => 'parent_id'],
    ];

您可以通过定义`PARENT_ID`常量来修改用于标识父级的键名：

    const PARENT_ID = 'my_parent_column';

使用此特征的Model实例集合将返回添加`toNested`方法的`October\Rain\Database\TreeCollection`类型。 要构建一个eager加载的树结构，请返回带有eager加载关系的记录。

    Category::all()->toNested();
    
### 渲染

为了呈现所有级别的项目及其子级，您可以使用递归处理

    {% macro renderChildren(item) %}
        {% import _self as SELF %}
        {% if item.children is not empty %}
            <ul>
                {% for child in item.children %}
                    <li>{{ child.name }}{{ SELF.renderChildren(child) | raw }}</li>
                {% endfor %}
            </ul>
        {% endif %}
    {% endmacro %}

    {% import _self as SELF %}
    {{ SELF.renderChildren(category) | raw }}

<a name="nested-tree"></a>
## 嵌套树

[嵌套集Model实例](https://en.wikipedia.org/wiki/Nested_set_model) 是一种使用`parent_id`，`nest_left`，`nest_right`和`nest_depth`列维护Model实例之间层次结构的高级技术。 要使用嵌套集Model实例，请应用`October\Rain\Database\Traits\NestedTree`特征。 “SimpleTree”特性的所有功能在本Model实例中都是固有的。

    class Category extends Model
    {
        use \October\Rain\Database\Traits\NestedTree;
    }

### 创建根节点

默认情况下，所有节点都创建为根：

    $root = Category::create(['name' => 'Root category']);

或者，您可能会发现自己需要将现有节点转换为根节点：

    $node->makeRoot();

您也可以使它的`parent_id`列无效，其作用与`makeRoot'相同。

    $node->parent_id = null;
    $node->save();

### 插入节点

您可以通过以下关系直接插入新节点：

    $child1 = $root->children()->create(['name' => 'Child 1']);

或者对现有节点使用`makeChildOf`方法：

    $child2 = Category::create(['name' => 'Child 2']);
    $child2->makeChildOf($root);

###删除节点

使用`delete`方法删除节点时，该节点的所有后代也将被删除。 请注意，不会为子Model实例触发delete [Model实例事件](../database/model#model-events) 。

    $child1->delete();

###获取节点的嵌套级别

`getLevel`方法将返回节点的当前嵌套级别或深度。

    // 根节点的时候是0 
    $node->getLevel()

### 移动节点

有几种移动节点的方法：

- `moveLeft()`:找到左边的兄弟并向左移动。
- `moveRight()`: 找到合适的兄弟并向右移动。
- `moveBefore($otherNode)`: 移动到左侧的节点
- `moveAfter($otherNode)`:移动到右侧的节点 ...
- `makeChildOf($otherNode)`:使节点成为...的孩子
- `makeRoot()`:使当前节点成为根节点。

<a name="validation"></a>
## 验证

OctoberModel实例使用内置的[Validator类](../services/validation)。 验证规则在Model实例类中定义为名为`$rules`的属性，类必须使用特性`October\Rain\Database\Traits\Validation`：

    class User extends Model
    {
        use \October\Rain\Database\Traits\Validation;

        public $rules = [
            'name'                  => 'required|between:4,16',
            'email'                 => 'required|email',
            'password'              => 'required|alpha_num|between:4,8|confirmed',
            'password_confirmation' => 'required|alpha_num|between:4,8'
        ];
    }

> **注意**: 您也可以将[数组语法](../services/validation#basic-usage) 用于验证规则。

Model实例在调用`save`方法时自动验证。

    $user = new User;
    $user->name = 'Actual Person';
    $user->email = 'a.person@example.com';
    $user->password = 'passw0rd';

    // Returns false if model is invalid
    $success = $user->save();

> **注意:** 您还可以使用`validate`方法随时验证Model实例。

<a name="retrieving-validation-errors"></a>
### 检索验证错误

当Model实例无法验证时，“Illuminate\Support\MessageBag”对象将附加到Model实例。 包含验证失败消息的对象。 使用`errors`方法或`$validationErrors`属性检索验证错误消息集合实例。 使用`errors()->all()`检索所有验证错误。 使用`validationErrors->get('attribute')`检索*specific*属性的错误。

> **注意:** 该Model实例利用了MessagesBag对象，该对象具有格式错误的[简单而优雅的方法](../services/validation#working-with-error-messages) 。

<a name="overriding-validation"></a>
### 覆盖验证

无论是否存在验证错误，`forceSave`方法都会验证Model实例并进行保存。

    $user = new User;

    // 创建没有验证的用户
    $user->forceSave();

<a name="custom-error-messages"></a>
### 自定义错误消息

与Validator类一样，您可以使用[相同语法](../services/validation#custom-error-messages)设置自定义错误消息。

    class User extends Model
    {
        public $customMessages = [
           'required' => 'The :attribute field is required.',
            ...
        ];
    }

<a name="custom-attribute-names"></a>
### 自定义属性名称

您还可以使用`$attributeNames`数组设置自定义属性名称。

    class User extends Model
    {
        public $attributeNames = [
           'email' => 'Email Address',
            ...
        ];
    }

<a name="dynamic-validation-rules"></a>
### 动态验证规则

您可以通过覆盖`beforeValidate` [Model实例事件](../database/model#events) 方法动态应用规则。 在这里，我们检查`is_remote`属性是否为'false`，然后动态地将`latitude`和`longitude`属性设置为必需字段。

    public function beforeValidate()
    {
        if (!$this->is_remote) {
            $this->rules['latitude'] = 'required';
            $this->rules['longitude'] = 'required';
        }
    }

<a name="custom-validation-rules"></a>
### 自定义验证规则

您还可以使用[相同方式](../services/validation#custom-validation-rules)为Validator服务创建自定义验证规则。

<a name="soft-deleting"></a>
## 软删除

软删除Model实例时，实际上并未从数据库中删除它。 而是在记录上设置`deleted_at`时间戳。 要为Model实例启用软删除，请将`October\Rain\Database\Traits\SoftDelete`特征应用于Model实例，并将deleted_at列添加到`$dates`属性：

    class User extends Model
    {
        use \October\Rain\Database\Traits\SoftDelete;

        protected $dates = ['deleted_at'];
    }

要向表中添加`deleted_at`列，可以使用迁移中的`softDeletes`方法：

    Schema::table('posts', function ($table) {
        $table->softDeletes();
    });

现在，当您在Model实例上调用`delete`方法时，`deleted_at`列将被设置为当前时间戳。 查询使用软删除的Model实例时，“已删除”Model实例将不包含在查询结果中。

要确定给定的Model实例实例是否已被软删除，请使用`trashed`方法：

    if ($user->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### 查询软删除的Model实例

#### 包括软删除Model实例

如上所述，软删除的Model实例将自动从查询结果中排除。 但是，您可以使用查询中的`withTrashed`方法强制软删除的Model实例出现在结果集中：

    $users = User::withTrashed()->where('account_id', 1)->get();

`withTrashed`方法也可用于[关系](relations)查询：

    $flight->history()->withTrashed()->get();

#### 仅检索软删除的Model实例

`onlyTrashed`方法将检索**仅**软删除的Model实例：

    $users = User::onlyTrashed()->where('account_id', 1)->get();

#### 恢复软删除的Model实例

有时您可能希望“取消删除”软删除的Model实例。 要将软删除的Model实例恢复为活动状态，请在Model实例实例上使用`restore`方法：

    $user->restore();

您还可以在查询中使用`restore`方法快速恢复多个Model实例：

    // 恢复单个Model实例实例...
    User::withTrashed()->where('account_id', 1)->restore();

    // 恢复所有相关Model实例实例......
    $user->posts()->restore();

#### 永久删除Model实例

有时您可能需要从数据库中真正删除Model实例。 要从数据库中永久删除软删除的Model实例，请使用`forceDelete`方法：

    // 强制删除单个Model实例实例...
    $user->forceDelete();

    // 强制删除所有相关Model实例......
    $user->posts()->forceDelete();

<a name="soft-deleting-relations"></a>
### 软删除关系

当两个相关Model实例启用了软删除时，您可以通过在[关系定义](../database/relations#detailed-relationships)中定义`softDelete`选项来级联delete事件。 在此示例中，如果用户Model实例被软删除，则属于该用户的注释也将被软删除。

    class User extends Model
    {
        use \October\Rain\Database\Traits\SoftDelete;

        public $hasMany = [
            'comments' => ['Acme\Blog\Models\Comment', 'softDelete' => true]
        ];
    }

> **注意:** 如果相关Model实例不使用软删除特征，则将其视为与`delete`选项相同并永久删除。

在这些相同的条件下，当主Model实例恢复时，所有使用`softDelete`选项的相关Model实例也将被恢复。

    // 恢复用户的同时也恢复相关的评论
    $user->restore();

<a name="nullable"></a>
## 可为空

当为空时，可空属性设置为“NULL”。 要使Model实例中的属性无效，请应用`October\Rain\Database\Traits\Nullable`特征并使用包含要无效的属性的数组声明`$nullable`属性。

    class Product extends Model
    {
        use \October\Rain\Database\Traits\Nullable;

        /**
         * @var array 可为空属性
         */
        protected $nullable = ['sku'];
    }
