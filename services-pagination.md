# 分页

- [基本用法](#basic-usage)
    - [分页查询构造器结果](#paginating-query-builder-results)
    - [分页模型结果](#paginating-eloquent-results)
    - [手动创建分页器](#manually-creating-a-paginator)
- [在视图中显示结果](#displaying-results-in-a-view)
- [将结果转换为JSON](#converting-results-to-json)

<a name="basic-usage"></a>
## 基本用法

分页记录有几种方法，最常见的是在查询或模型上调用`paginate`方法。 这将返回一个特殊的分页集合，该集合添加了用于显示结果的额外方法。

<a name="paginating-query-builder-results"></a>
### 分页查询构造器结果

有几种方法可以对项目进行分页。 最简单的方法是在[查询生成器](../database/query)或[模型查询(../database/model)上使用`paginate`方法。 `paginate`方法自动根据用户正在查看的当前页面设置适当的限制和偏移。 默认情况下，当前页面由HTTP请求上的“？page”查询字符串参数的值检测。 当然，该值会自动检测并自动插入到由分页器生成的链接中。

首先，让我们看看在查询中调用`paginate`方法。 在这个例子中，传递给`paginate`的唯一参数是你想要“每页”显示的项目数。 在这种情况下，让我们指定我们希望每页显示“15”项：

    $users = Db::table('users')->paginate(15);

> **注意:** 目前，使用`groupBy`语句的分页操作无法有效执行。 如果需要使用带有分页结果集的`groupBy`，建议您查询数据库并手动创建分页器。

#### 简单的分页

如果您只需要在分页视图中显示简单的“Next”和“Previous”链接，则可以选择使用`simplePaginate`方法来执行更有效的查询。 如果在渲染视图时不需要为每个页码显示链接，则这对大型数据集非常有用：

    $users = Db::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### 分页模型结果

您还可以对[数据库模型](../database/model)查询进行分页。 在这个例子中，我们将每页“15”项对“User”模型进行分页。 如您所见，语法与分页查询构建器结果几乎完全相同：

    $users = User::paginate(15);

当然，在对查询设置其他约束之后，可以调用`paginate`，例如`where`子句：

    $users = User::where('votes', '>', 100)->paginate(15);

在分页模型时，您也可以使用`simplePaginate`方法：

    $users = User::where('votes', '>', 100)->simplePaginate(15);

您可以通过传递第二个参数手动指定页码，这里我们每页分页“15”项，指定我们在页面“2”上：

    $users = User::where('votes', '>', 100)->paginate(15, 2);

<a name="manually-creating-a-paginator"></a>
### 手动创建分页器

有时您可能希望手动创建分页实例，并向其传递一系列项目。您可以根据需要创建`Illuminate\Pagination\Paginator`或`Illuminate\Pagination\LengthAwarePaginator`实例。

`Paginator`类不需要知道结果集中的项目总数，因此该类没有检索最后一页索引的方法。 `LengthAwarePaginator`接受与`Paginator`几乎相同的参数，但它确实需要计算结果集中项的总数。

换句话说，`Paginator`对应于查询构建器和模型上的`simplePaginate`方法，而`LengthAwarePaginator`对应于`paginate`方法。

手动创建paginator实例时，应手动“切片”传递给paginator的结果数组。如果您不确定如何执行此操作，请查看[array_slice](http://php.net/manual/en/function.array-slice.php) PHP函数。

<a name="displaying-results-in-a-view"></a>
## 在视图中显示结果

当您在查询构建器或模型查询上调用`paginate`或`simplePaginate`方法时，您将收到一个paginator实例。 调用`paginate`方法时，您将收到`Illuminate\Pagination\LengthAwarePaginator`的实例。 调用`simplePaginate`方法时，您将收到`Illuminate\Pagination\Paginator`的实例。 这些对象提供了几种描述结果集的方法。 除了这些帮助器方法之外，paginator实例也是迭代器，并且可以作为数组循环。

因此，一旦检索到结果，您可以使用Twig显示结果并呈现页面链接：

    <div class="container">
        {% for user in users %}
            {{ user.name }}
        {% endfor %}
    </div>

    {{ users.render|raw }}

`render`方法将呈现结果集中其余页面的链接。 这些链接中的每一个都已包含正确的`?page`查询字符串变量。 `render`方法生成的HTML与[Bootstrap CSS framework](https://getbootstrap.com)兼容。

> **注意:** 从Twig模板调用`render`方法时，请务必使用`| raw`过滤器，以便不转义HTML链接。

#### 自定义分页器URI

`setPath`方法允许您在生成链接时自定义分页器使用的URI。 例如，如果您希望paginator生成类似`http：//example.com/custom/url?page = N`的链接，则应将`custom/url`传递给`setPath`方法：

    $users = User::paginate(15);
    $users->setPath('custom/url');

#### 附加分页链接

您可以使用`appends`方法添加到分页链接的查询字符串。 例如，要将`＆sort=votes`附加到每个分页链接，您应该对`appends`进行以下调用：

    {{ users.appends({sort: 'votes'}).render|raw }}

如果您希望在分页器的URL中附加“哈希片段”，可以使用`fragment`方法。 例如，要将`#foo`附加到每个分页链接的末尾，请对`fragment`方法进行以下调用：

    {{ users.fragment('foo').render|raw }}

#### 其他辅助方法

您还可以通过以下方法在paginator实例上访问其他分页信息：

    $results->count()
    $results->currentPage()
    $results->hasMorePages()
    $results->lastPage() (使用simplePaginate时不可用)
    $results->nextPageUrl()
    $results->perPage()
    $results->previousPageUrl()
    $results->total() (使用simplePaginate时不可用)
    $results->url($page)

<a name="converting-results-to-json"></a>
## 将结果转换为JSON

paginator结果类实现了`Illuminate\Contracts\Support\JsonableInterface`并公开了`toJson`方法，因此将分页结果转换为JSON非常容易。 您也可以通过从路由或AJAX处理程序返回它来将paginator实例转换为JSON：

    Route::get('users', function () {
        return User::paginate();
    });

来自分页器的JSON将包括元信息，例如`total`，`current_page`，`last_page`等。 实际的结果对象可以通过JSON数组中的`data`键获得。 以下是通过从路由返回paginator实例而创建的JSON示例：

#### 示例Paginator JSON

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "next_page_url": "http://octobercms.app?page=2",
       "prev_page_url": null,
       "from": 1,
       "to": 15,
       "data":[
            {
                // Result Object
            },
            {
                // Result Object
            }
       ]
    }
