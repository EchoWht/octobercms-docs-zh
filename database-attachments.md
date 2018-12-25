# 数据库附件

- [文件附件](#file-attachments)
    - [创建新附件](#creating-attachments)
    - [查看附件](#viewing-attachments)
    - [用法示例](#attachments-usage-example)
    - [验证示例](#attachments-validation-example)


<a name="file-attachments"></a>
## 文件附件

Model可以使用[多态关系](database-relations.md#polymorphic-relations)的子集支持文件附件。 `$attachOne`或`$attachMany`关系用于将文件链接到名为“attachments”的数据库记录。在几乎所有情况下，`System\Models\File`Model用于维护这种关系，其中对文件的引用作为记录存储在`system_files`表中，并且和父Model具有多态关系。

在下面的示例中，Model具有单个头像附件Model和许多照片附件Model。

单个文件附件：

    public $attachOne = [
        'avatar' => 'System\Models\File'
    ];

多个文件附件：

    public $attachMany = [
        'photos' => 'System\Models\File'
    ];

注意：在上面的示例中，使用的密钥名称与文件上传字段名称相同。 在Model和`System\Models\File`Model之间创建多态关系时，如果您有一个与文件上传字段名称共享同一名称的列，则可能会导致意外结果。


受保护的附件将上传到应用程序的**uploads/protected**目录，该目录无法从Web直接访问。 通过将*public*参数设置为“false”来定义受保护的文件附件：

    public $attachOne = [
        'avatar' => ['System\Models\File', 'public' => false]
    ];

<a name="creating-attachments"></a>
### 创建新附件

对于一对一关系(`$attachOne`)，您可以通过Model关系直接创建附件，方法是使用`Input::file`方法设置其值，该方法从上传中读取文件数据。

    $model->avatar = Input::file('file_input');

您还可以将字符串传递给`data`属性，该属性包含本地文件的绝对路径。

    $model->avatar = '/path/to/somefile.jpg';
    
有时直接从(raw)数据创建`File`实例也很有用：

    $file = (new System\Models\File)->fromData('Some content', 'sometext.txt');

对于一对多关系(`$attachMany`)，您可以在关系上使用`create`方法，注意文件对象与`data`属性相关联。 如果您愿意，这种方法也可以用于单一关系。

    $model->avatar()->create(['data' => Input::file('file_input')]);

或者，您可以预先准备文件Model，然后再手动关联该关系。 请注意，必须使用此方法显式设置`is_public`属性。

    $file = new System\Models\File;
    $file->data = Input::file('file_input');
    $file->is_public = true;
    $file->save();

    $model->avatar()->add($file);

<a name="viewing-attachments"></a>
### 查看附件

`getPath`方法返回上传的公共文件的完整URL。 以下代码将打印类似**example.com/uploads/public/path/to/avatar.jpg**

    echo $model->avatar->getPath();

返回多个附件文件路径：

    foreach ($model->photos as $photo) {
        echo $photo->getPath();
    }

`getLocalPath`方法将返回本地文件系统中上传文件的绝对路径。

    echo $model->avatar->getLocalPath();

要直接输出文件内容，请使用`output`方法，这将包含下载文件所需的标题：

    echo $model->avatar->output();

您可以使用`getThumb`方法调整图像大小。 该方法需要3个参数 - 图像宽度，图像高度和选项参数。 支持以下选项：

选项 | 描述
------------- | -------------
**mode** | auto(自适应), exact(确切), portrait, landscape, crop。 默认: auto
**quality** |质量 0 - 100. 默认: 95
**interlace** | boolean: false (默认), true
**extension** | auto, jpg, png, gif. 默认: jpg

宽度**width**和高度**height** *参数应指定为数字或自动比例缩放的。

    echo $model->avatar->getThumb(100, 100, ['mode' => 'crop']);

<a name="attachments-usage-example"></a>
### 用法示例

本节显示了Model附件功能的完整用法示例 - 从定义Model中的关系到在页面上显示上传的图像。

在Model中定义与`System\Models\File`类的关系，例如：

    class Post extends Model
    {
        public $attachOne = [
            'featured_image' => 'System\Models\File'
        ];
    }

构建上传文件的表单：

    <?= Form::open(['files' => true]) ?>

        <input name="example_file" type="file">

        <button type="submit">Upload File</button>

    <?= Form::close() ?>

在服务器上处理上传的文件并将其附加到Model：

    // Find the Blog Post model
    $post = Post::find(1);

    // 保存Blog PostModel的精选图像
    if (Input::hasFile('example_file')) {
        $post->featured_image = Input::file('example_file');
    }

或者，您可以使用[延迟绑定](database-relations.md#deferred-binding)来推迟关系：

    // 找到Blog PostModel
    $post = Post::find(1);

    // 从html前端获取上传的文件
    $fileFromPost = Input::file('example_file');

    // 如果存在，请使用延迟会话密钥将其另存为图像
    if ($fileFromPost) {
        $post->featured_image()->create(['data' => $fileFromPost], $sessionKey);
    }

在页面上显示上传的文件：

    // 找到Blog PostModel
    $post = Post::find(1);

    // 查找图像地址，否则使用默认地址
    if ($post->featured_image) {
        $featuredImage = $post->featured_image->getPath();
    }
    else {
        $featuredImage = 'http://placehold.it/220x300';
    }

    <img src="<?= $featuredImage ?>" alt="Featured Image">

如果需要访问文件的所有者，可以使用`File`Model的`attachment`属性：

    public $morphTo = [
        'attachment' => []
    ];
    
例如：  

    $user = $file->attachment;
    
有关更多信息，请阅读[多态关系](database-relations.md#polymorphic-relations)

<a name="attachments-validation-example"></a>
### 验证示例

下面的示例使用[数组验证](services-validation.md#validating-arrays)来验证`$attachMany`关系。

    use October\Rain\Database\Traits\Validation;
    use System\Models\File;
    use Model;
    
    class Gallery extends Model
    {
        use Validation;

        public $attachMany = [
            'photos' => File::class
        ];
    
        public $rules = [
            'photos'   => 'required',
            'photos.*' => 'image|max:1000|dimensions:min_width=100,min_height=100'
        ];
    
        /* some other code */
    }

有关上面使用的`attribute.*`语法的更多信息，请参阅[validating arrays](services-validation.md#validating-arrays)。
