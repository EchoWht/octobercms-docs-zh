# 数据集合

- [介绍](#introduction)
- [用法示例](#usage-examples)
- [自定义集合](#custom-collections)
- [数据填充](#data-feed)
    - [创建一个新的填充](#creating-feed)
    - [处理结果](#data-feed-processing)
    - [对结果进行排序](#data-feed-ordering)

<a name="introduction"></a>
## 介绍

模型返回的所有多结果集都是`Illuminate \ Database \ Eloquent \ Collection`对象的实例，包括通过`get`方法检索的结果或通过关系访问的结果。 `Collection`对象扩展了[base collection]（../ services / collections），因此它自然地继承了许多用于流畅地处理底层模型数组的方法。

当然，所有集合也可以作为迭代器，允许您循环遍历它们，就像它们是简单的PHP数组一样：

    $users = User::where('is_active', true)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

但是，集合比数组更强大，并使用直观的界面显示各种map/reduce操作。 例如，让我们过滤所有活动模型，并为每个过滤后的用户收集名称：

    $users = User::get();

    $names = $users->filter(function ($user) {
            return $user->is_active === true;
        })
        ->map(function ($user) {
            return $user->name;
        });

<a name="usage-examples"></a>
## 用法示例

以下是一些返回`Collection`对象的方法示例：

    $collection = User::all();
    $collection = User::where('name', 'Joe')->get();
    $collection = User::find(3)->groups;

#### 转换为JSON或数组

集合也可以转换为PHP数组或JSON：

    $phpArray = $collection->toArray();

    $jsonString = $collection->toJson();

#### 找到第一个匹配的Model实例

`first`方法将返回匹配条件的第一个对象。

    $found = $collection->first(function($model) {
        return $model->color == 'red';
    });

#### 迭代集合

`each`方法可让您轻松循环遍历内部的每个对象。

    $collection->each(function($model) {
        $model->is_cool = true;
    });

#### 检查key

如果集合内部具有指定主键值的模型，则`contains`方法将返回true。

    if ($collection->contains(3)) {
        // Collection有一个ID为3的Model实例
    }

<a name="custom-collections"></a>
## 自定义集合

如果您需要使用自己的扩展方法使用自定义`Collection`对象，则可以覆盖模型上的`newCollection`方法：

    class User extends Model
    {
        /**
         * 创建一个新的Collection实例。
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

一旦定义了`newCollection`方法，只要模型返回`Collection`实例，您就会收到自定义集合的实例。 如果您想为插件或应用程序中的每个模型使用自定义集合，则应该在所有模型扩展的模型基类上覆盖`newCollection`方法。

    use October\Rain\Database\Collection as CollectionBase;

    class CustomCollection extends CollectionBase
    {
    }

<a name="data-feed"></a>
## 数据填充

数据源允许您将多个模型类组合到一个集合中。 这对于在支持分页使用的同时创建数据源和数据流非常有用。 它的工作原理是在调用`get`方法之前添加处于准备状态的模型对象，然后将它们组合起来，使集合的行为与常规数据集相同。

`DataFeed`类模仿常规模型并支持`limit`和`paginate`方法。

<a name="creating-feed"></a>
### 创建填充

下一个示例将User，Post和Comment模型合并到一个集合中，并返回前10个记录。

    $feed = new October\Rain\Database\DataFeed;
    $feed->add('user', new User);
    $feed->add('post', Post::where('category_id', 7));

    $feed->add('comment', function() {
        $comment = new Comment;
        return $comment->where('approved', true);
    });

    $results = $feed->limit(10)->get();

<a name="data-feed-processing"></a>
### 处理结果

`get`方法将返回包含结果的`Collection`对象。 可以使用`tag_name`属性来区分记录，该属性在添加模型时被设置为第一个参数。

    foreach ($results as $result) {

        if ($result->tag_name == 'post')
            echo "New Blog Post: " . $record->title;

        elseif ($result->tag_name == 'comment')
            echo "New Comment: " . $record->content;

        elseif ($result->tag_name == 'user')
            echo "New User: " . $record->name;

    }

<a name="data-feed-ordering"></a>
### 对结果进行排序

结果可以根据单个数据列排序，所有数据集都使用共享默认值，或者使用“add”方法单独指定。 结果的排序方向也必须分享。

    // 如果存在则由updated_at排序，否则为created_at
    $feed->add('user', new User, 'ifnull(updated_at, created_at)');

    // 根据id排序
    $feed->add('comments', new Comment, 'id');

    // 按名称排序（下面指定默认值）
    $feed->add('posts', new Post);

    // 指定默认列和排序方向
    $feed->orderBy('name', 'asc')->get();
