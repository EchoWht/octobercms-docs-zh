# 集合

- [介绍](#introduction)
- [创建集合](#creating-collections)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 介绍

`October\Rain\Support\Collection`类为处理数据数组提供了一个流畅，方便的包装器。 例如，请查看以下代码。 我们将从数组中创建一个新的集合实例，在每个元素上运行`strtoupper`函数，然后删除所有空元素：

    $collection = new October\Rain\Support\Collection(['stewie', 'brian', null]);

    $collection = $collection
        ->map(function ($name) {
            return strtoupper($name);
        })
        ->reject(function ($name) {
            return empty($name);
        })
    ;

`Collection`类允许您链式调用方法以执行流畅的映射和减少底层数组。 通常，每个`Collection`方法都返回一个全新的`Collection`实例。

<a name="creating-collections"></a>
## 创建集合

如上所述，将数组传递给`October\Rain\Support\Collection`类的构造函数将返回给定数组的新实例。 因此，创建集合非常简单：

    $collection = new October\Rain\Support\Collection([1, 2, 3]);

默认情况下，[数据库模型](../database/model)的集合总是作为`Collection`实例返回; 但是，您可以随意使用`Collection`类，方便您的应用。

<a name="available-methods"></a>
## 可用方法

对于本文档的其余部分，我们将讨论`Collection`类中可用的每个方法。 请记住，所有这些方法都可以链式调用，以便流畅地操作底层数组。 此外，几乎每个方法都返回一个新的`Collection`实例，允许您在必要时保留集合的原始副本。

您可以从此表中选择任何方法以查看其用法示例：

[all](#method-all)
,[chunk](#method-chunk)
,[collapse](#method-collapse)
,[contains](#method-contains)
,[count](#method-count)
,[diff](#method-diff)
,[each](#method-each)
,[filter](#method-filter)
,[first](#method-first)
,[flatten](#method-flatten)
,[flip](#method-flip)
,[forget](#method-forget)
,[forPage](#method-forpage)
,[get](#method-get)
,[groupBy](#method-groupby)
,[has](#method-has)
,[implode](#method-implode)
,[intersect](#method-intersect)
,[isEmpty](#method-isempty)
,[keyBy](#method-keyby)
,[keys](#method-keys)
,[last](#method-last)
,[map](#method-map)
,[merge](#method-merge)
,[pluck](#method-pluck)
,[pop](#method-pop)
,[prepend](#method-prepend)
,[pull](#method-pull)
,[push](#method-push)
,[put](#method-put)
,[random](#method-random)
,[reduce](#method-reduce)
,[reject](#method-reject)
,[reverse](#method-reverse)
,[search](#method-search)
,[shift](#method-shift)
,[shuffle](#method-shuffle)
,[slice](#method-slice)
,[sort](#method-sort)
,[sortBy](#method-sortby)
,[sortByDesc](#method-sortbydesc)
,[splice](#method-splice)
,[sum](#method-sum)
,[take](#method-take)
,[toArray](#method-toarray)
,[toJson](#method-tojson)
,[transform](#method-transform)
,[unique](#method-unique)
,[values](#method-values)
,[where](#method-where)
,[whereLoose](#method-whereloose)
,[zip](#method-zip)


<a name="method-listing"></a>
## 方法列表

<a name="method-all"></a>
#### `all()`

`all`方法只返回集合表示的底层数组：

    $collection = new Collection([1, 2, 3]);

    $collection->all();

    //[1, 2, 3]

<a name="method-chunk"></a>
#### `chunk()` 

`chunk`方法将集合分成多个给定大小的较小集合：

    $collection = new Collection([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    //[[1, 2, 3, 4], [5, 6, 7]]

在使用网格系统时，此方法在[CMS页面](../cms/pages)中特别有用，例如[Bootstrap](http://getbootstrap.com/css/#grid)。 想象一下，您有一组要在网格中显示的模型：

    {% for chunk in products.chunk(3) %}
        <div class="row">
            {% for product in chunk %}
                <div class="col-xs-4">{{ product.name }}</div>
            {% endfor %}
        </div>
    {% endfor %}

<a name="method-collapse"></a>
#### `collapse()`

`collapse`方法将一组数组折叠成一个扁平集合：

    $collection = new Collection([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    //[1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-contains"></a>
#### `contains()`

`contains`方法确定集合是否包含给定项：

    $collection = new Collection(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    //true

    $collection->contains('New York');

    //false

您还可以将键/值对传递给`contains`方法，该方法将确定集合中是否存在给定的对：

    $collection = new Collection([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    //false

最后，您还可以将回调传递给`contains`方法来执行您自己的真值测试：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $collection->contains(function ($key, $value) {
        return $value > 5;
    });

    //false

<a name="method-count"></a>
#### `count()`

`count`方法返回集合中的项目总数：

    $collection = new Collection([1, 2, 3, 4]);

    $collection->count();

    //4

<a name="method-diff"></a>
#### `diff()` 

`diff`方法将集合与另一个集合或普通的PHP`数组进行比较：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    //[1, 3, 5]

<a name="method-each"></a>
#### `each()`

`each`方法迭代集合中的项目并将每个项目传递给给定的回调：

    $collection = $collection->each(function ($item, $key) {
        //
    });

从你的回调中返回`false`以打破循环：

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-every"></a>
#### `every()` 

`every`方法创建一个由第n个元素组成的新集合：

    $collection = new Collection(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->every(4);

    //['a', 'e']

您可以选择将offset作为第二个参数传递：

    $collection->every(4, 1);

    //['b', 'f']

<a name="method-filter"></a>
#### `filter()` 

`filter`方法按给定的回调过滤集合，只保留那些通过给定真值测试的项：

    $collection = new Collection([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($item) {
        return $item > 2;
    });

    $filtered->all();

    //[3, 4]

对于“filter”的反转，请参见[reject](#method-reject)方法。

<a name="method-first"></a>
#### `first()`

`first`方法返回集合中传递给定真值测试的第一个元素：

    new Collection([1, 2, 3, 4])->first(function ($key, $value) {
        return $value > 2;
    });

    //3

您也可以调用没有参数的`first`方法来获取集合中的第一个元素。 如果集合为空，则返回“null”：

    new Collection([1, 2, 3, 4])->first();

    //1

<a name="method-flatten"></a>
#### `flatten()`

`flatten`方法将多维集合展平为单个维度：

    $collection = new Collection(['name' => 'peter', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    //['peter', 'php', 'javascript'];

<a name="method-flip"></a>
#### `flip()`

`flip`方法用它们对应的值交换集合的键：

    $collection = new Collection(['name' => 'peter', 'platform' => 'october']);

    $flipped = $collection->flip();

    $flipped->all();

    //['peter' => 'name', 'october' => 'platform']

<a name="method-forget"></a>
#### `forget()`

`forget`方法通过其键从集合中删除一个项：

    $collection = new Collection(['name' => 'peter', 'platform' => 'october']);

    $collection->forget('name');

    $collection->all();

    //['platform' => 'october']

> **注意:** 与大多数其他集合方法不同，`forget`不会返回新的修改集合; 它修改了他调用的原集合。

<a name="method-forpage"></a>
#### `forPage()`

`forPage`方法返回一个新集合，其中包含将在给定页码上出现的项：

    $collection = new Collection([1, 2, 3, 4, 5, 6, 7, 8, 9])->forPage(2, 3);

    $collection->all();

    //[4, 5, 6]

该方法分别需要页面编号和每页显示的项目数。

<a name="method-get"></a>
#### `get()` 

`get`方法返回给定键的项。 如果密钥不存在，则返回“null”：

    $collection = new Collection(['name' => 'peter', 'platform' => 'october']);

    $value = $collection->get('name');

    //peter

您可以选择传递默认值作为第二个参数：

    $collection = new Collection(['name' => 'peter', 'platform' => 'october']);

    $value = $collection->get('foo', 'default-value');

    //default-value

您甚至可以将回调作为默认值传递。 如果指定的键不存在，将返回回调的结果：

    $collection->get('email', function () {
        return 'default-value';
    });

    //default-value

<a name="method-groupby"></a>
#### `groupBy()`

`groupBy`方法按给定键对集合的项进行分组：

    $collection = new Collection([
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
                ['account_id' => 'account-x10', 'product' => 'Chair'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

除了传递字符串`key`之外，您还可以传递回调。 回调应返回您希望键入组的值：

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
                ['account_id' => 'account-x10', 'product' => 'Chair'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

<a name="method-has"></a>
#### `has()` 

`has`方法确定集合中是否存在给定的键：

    $collection = new Collection(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('email');

    //false

<a name="method-implode"></a>
#### `implode()`

`implode`方法连接集合中的项目。 它的参数取决于集合中项目的类型。

如果集合包含数组或对象，则应传递要连接的属性的键，以及要在值之间放置的“粘合”字符串：

    $collection = new Collection([
        ['account_id' => 1, 'product' => 'Chair'],
        ['account_id' => 2, 'product' => 'Desk'],
    ]);

    $collection->implode('product', ', ');

    //Chair, Desk

如果集合包含简单的字符串或数值，只需将“glue”作为方法的唯一参数传递：

    new Collection([1, 2, 3, 4, 5])->implode('-');

    //'1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {.collection-method}

`intersect`方法删除给定`array`或集合中不存在的任何值：

    $collection = new Collection(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    //[0 => 'Desk', 2 => 'Chair']

如您所见，生成的集合将保留原始集合的键。

<a name="method-isempty"></a>
#### `isEmpty()`

如果集合为空，`isEmpty`方法返回`true`; 否则返回`false`：

    new Collection([])->isEmpty();

    //true

<a name="method-keyby"></a>
#### `keyBy()`

按给定键键入集合：

    $collection = new Collection([
        ['product_id' => 'prod-100', 'name' => 'chair'],
        ['product_id' => 'prod-200', 'name' => 'desk'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Chair'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Desk'],
        ]
    */

如果多个项具有相同的键，则只有最后一个项将出现在新集合中。

您也可以传递自己的回调，该回调应该返回值以通过以下方式键入集合：

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Chair'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Desk'],
        ]
    */


<a name="method-keys"></a>
#### `keys()` 

`keys`方法返回所有集合的键：

    $collection = new Collection([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Chair'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Desk'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    //['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` 

`last`方法返回集合中传递给定真值测试的最后一个元素：

    new Collection([1, 2, 3, 4])->last(function ($key, $value) {
        return $value < 3;
    });

    //2

您也可以调用没有参数的`last`方法来获取集合中的最后一个元素。 如果集合为空，则返回“null”。

    new Collection([1, 2, 3, 4])->last();

    //4

<a name="method-map"></a>
#### `map()`

`map`方法遍历集合并将每个值传递给给定的回调。 回调可以自由修改项目并将其返回，从而形成一个新的修改项目集合：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    //[2, 4, 6, 8, 10]

> **注意:** 像大多数其他集合方法一样，`map`返回一个新的集合实例; 它不会修改他调用的原集合。 如果要转换原始集合，请使用[`transform`](#method-transform)方法。

<a name="method-merge"></a>
#### `merge()`

`merge`方法将给定数组合并到集合中。 与集合中的字符串键匹配的数组中的任何字符串键都将覆盖集合中的值：

    $collection = new Collection(['product_id' => 1, 'name' => 'Desk']);

    $merged = $collection->merge(['price' => 100, 'discount' => false]);

    $merged->all();

    //['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]

如果给定数组的键是数字，则值将附加到集合的末尾：

    $collection = new Collection(['Bookcase', 'Chair']);

    $merged = $collection->merge(['Desk', 'Door']);

    $merged->all();

    //['Bookcase', 'Chair', 'Desk', 'Door']

<a name="method-pluck"></a>
#### `pluck()` 

`pluck`方法检索给定键的所有集合值：

    $collection = new Collection([
        ['product_id' => 'prod-100', 'name' => 'Chair'],
        ['product_id' => 'prod-200', 'name' => 'Desk'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    //['Chair', 'Desk']

您还可以指定希望如何键入生成的集合：

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    //['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` 

`pop`方法删除并返回集合中的最后一项：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $collection->pop();

    //5

    $collection->all();

    //[1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` 

`prepend`方法将一个项添加到集合的开头：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    //[0, 1, 2, 3, 4, 5]

<a name="method-pull"></a>
#### `pull()` 

`pull`方法通过其键从集合中删除并返回一个项：

    $collection = new Collection(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    //'Desk'

    $collection->all();

    //['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` 

`push`方法将一个项追加到集合的末尾：

    $collection = new Collection([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    //[1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` 

`put`方法设置集合中给定的键和值：

    $collection = new Collection(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    //['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` 

`random`方法从集合中返回一个随机项：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $collection->random();

    //4 - (retrieved randomly)

您可以选择将整数传递给`random`。 如果该整数大于“1”，则返回一组项目：

    $random = $collection->random(3);

    $random->all();

    //[2, 4, 5] - (随机检索)

<a name="method-reduce"></a>
#### `reduce()`

`reduce`方法将集合减少为单个值，将每次迭代的结果传递给后续迭代：

    $collection = new Collection([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    //6

第一次迭代时`$carry`的值为`null`; 但是，您可以通过将第二个参数传递给`reduce`来指定其初始值：

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    //10

<a name="method-reject"></a>
#### `reject()` 

`reject`方法使用给定的回调过滤集合。 对于要从结果集合中删除的任何项，回调应返回“true”：

    $collection = new Collection([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($item) {
        return $item > 2;
    });

    $filtered->all();

    //[1, 2]

对于`reject`方法的反转，请参见[`filter`](#method-filter)方法。

<a name="method-reverse"></a>
#### `reverse()` {.collection-method}

`reverse`方法反转集合项的顺序：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $reversed = $collection->reverse();

    $reversed->all();

    //[5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()`

`search`方法在集合中搜索给定值，如果找到则返回其密钥。 如果找不到该项，则返回“false”。

    $collection = new Collection([2, 4, 6, 8]);

    $collection->search(4);

    //1

搜索是使用“松散”比较完成的。 要使用严格比较，请将“true”作为方法的第二个参数传递：

    $collection->search('4', true);

    //false

或者，您可以传入自己的回调来搜索通过真实测试的第一个项目：

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    //2

<a name="method-shift"></a>
#### `shift()` 

`shift`方法删除并返回集合中的第一个项：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $collection->shift();

    //1

    $collection->all();

    //[2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` 

`shuffle`方法随机洗牌集合中的项目：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    //[3, 2, 5, 1, 4] (随机生成)

<a name="method-slice"></a>
#### `slice()`

`slice`方法返回从给定索引开始的集合切片：

    $collection = new Collection([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    //[5, 6, 7, 8, 9, 10]

如果要限制返回切片的大小，请将所需大小作为方法的第二个参数传递：

    $slice = $collection->slice(4, 2);

    $slice->all();

    //[5, 6]

返回的切片将具有新的数字索引键。 如果要保留原始键，请将“true”作为方法的第三个参数传递。

<a name="method-sort"></a>
#### `sort()` 

`sort`方法对集合进行排序：

    $collection = new Collection([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    //[1, 2, 3, 4, 5]

已排序的集合保留原始数组键。 在这个例子中，我们使用[`values`](#method-values)方法将键重置为连续编号的索引。

有关对嵌套数组或对象的集合进行排序，请参阅[`sortBy`](#method-sortby)和[`sortByDesc`](#method-sortbydesc)方法。

如果您的排序需求更高级，您可以使用自己的算法将回调传递给`sort`。 请参阅[`usort`](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters)上的PHP文档，这是集合的`sort`方法调用的内容。 引擎盖。

<a name="method-sortby"></a>
#### `sortBy()`

`sortBy`方法按给定键对集合进行排序：

    $collection = new Collection([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

已排序的集合保留原始数组键。 在这个例子中，我们使用[`values`](#method-values)方法将键重置为连续编号的索引。

您还可以传递自己的回调以确定如何对集合值进行排序：

    $collection = new Collection([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` 

此方法与[`sortBy`](#method-sortby)方法具有相同的签名，但会以相反的顺序对集合进行排序。

<a name="method-splice"></a>
#### `splice()`

`splice`方法从指定的索引处删除并返回一条项目：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    //[3, 4, 5]

    $collection->all();

    //[1, 2]

您可以传递第二个参数来限制生成的块的大小：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    //[3]

    $collection->all();

    //[1, 2, 4, 5]

此外，您可以传递包含新项的第三个参数来替换从集合中删除的项：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    //[3]

    $collection->all();

    //[1, 2, 10, 11, 4, 5]

<a name="method-sum"></a>
#### `sum()`

`sum`方法返回集合中所有项的总和：

    new Collection([1, 2, 3, 4, 5])->sum();

    //15

如果集合包含嵌套数组或对象，则应传递一个键以用于确定要求和的值：

    $collection = new Collection([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    //1272

此外，您可以传递自己的回调以确定要汇总的集合的哪些值：

    $collection = new Collection([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    //6

<a name="method-take"></a>
#### `take()`

`take`方法返回一个具有指定项数的新集合：

    $collection = new Collection([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    //[0, 1, 2]

您还可以传递一个负整数，以从集合的末尾获取指定数量的项：

    $collection = new Collection([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    //[4, 5]

<a name="method-toarray"></a>
#### `toArray()` 

`toArray`方法将集合转换为普通的PHP`数组`。 如果集合的值是[数据库模型](../database/model)，模型也将转换为数组：

    $collection = new Collection(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> **注意:** `toArray`还将其所有嵌套对象转换为数组。 如果要按原样获取底层数组，请改用[`all`](#method-all)方法。

<a name="method-tojson"></a>
#### `toJson()`

`toJson`方法将集合转换为JSON：

    $collection = new Collection(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    //'{"name":"Desk","price":200}'

<a name="method-transform"></a>
#### `transform()`

`transform`方法遍历集合并使用集合中的每个项调用给定的回调。 集合中的项目将替换为回调返回的值：

    $collection = new Collection([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    //[2, 4, 6, 8, 10]

> **注意:** 与大多数其他集合方法不同，`transform`修改集合本身。 如果您希望改为创建新集合，请使用[`map`](#method-map)方法。

<a name="method-unique"></a>
#### `unique()` 

`unique`方法返回集合中的所有唯一项：

    $collection = new Collection([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    //[1, 2, 3, 4]

返回的集合保留原始数组键。 在这个例子中，我们使用[`values`](#method-values)方法将键重置为连续编号的索引。

处理嵌套数组或对象时，您可以指定用于确定唯一性的键：

    $collection = new Collection([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

您也可以传递自己的回调以确定项目唯一性：

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

<a name="method-values"></a>
#### `values()`

`values`方法返回一个新的集合，其中键重置为连续的整数：

    $collection = new Collection([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */
<a name="method-where"></a>
#### `where()` 

`where`方法按给定的键/值对过滤集合：

    $collection = new Collection([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
    */

检查项目值时，`where`方法使用严格的比较。 使用[`whereLoose`](#where-loose)方法使用“松散”比较进行过滤。

<a name="method-whereloose"></a>
#### `whereLoose()`

该方法与[`where`](#method-where)方法具有相同的签名; 但是，所有值都使用“松散”比较进行比较。

<a name="method-zip"></a>
#### `zip()`

`zip`方法将给定数组的值与相应索引处的集合值合并在一起：

    $collection = new Collection(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    //[['Chair', 100], ['Desk', 200]]
