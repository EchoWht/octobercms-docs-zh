# 邮件

- [介绍](#introduction)
- [发送邮件](#sending-mail)
    - [附件](#attachments)
    - [内联附件](#inline-attachments)
    - [邮件队列](#queueing-mail)
- [消息内容](#message-content)
    - [邮件视图](#mail-views)
    - [邮件模版](#mail-templates)
    - [邮件布局](#mail-layouts)
    - [注册邮件模板](#mail-template-registration)
    - [全局变量](#mail-global-variables)
- [邮件和本地开发](#mail-and-local-development)

<a name="introduction"></a>
## 介绍

October提供SMTP，Mailgun，SparkPost，Amazon SES，PHP的`mail`功能和`sendmail`的驱动程序，使您可以快速开始通过您选择的本地或云服务发送邮件。 有两种方法可以配置邮件服务，可以通过*设置>邮件设置*使用后端界面，也可以通过更新默认配置值。 在这些示例中，我们将更新配置值。

### 驱动先决条件

在使用Mailgun，SparkPost或SES驱动程序之前，您需要安装[Drivers plugin](http://octobercms.com/plugin/october-drivers).。

#### Mailgun 驱动

要使用Mailgun驱动程序，请将`config/mail.php`配置文件中的`driver`选项设置为`mailgun`。 接下来，验证您的`config/services.php`配置文件包含以下选项：

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### SparkPost 驱动

要使用SparkPost驱动程序，请将`config/mail.php`配置文件中的`driver`选项设置为`sparkpost`。 接下来，验证您的`config/services.php`配置文件包含以下选项：

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### SES 驱动

要使用Amazon SES驱动程序，请将`config/mail.php`配置文件中的`driver`选项设置为`ses`。 然后，验证您的`config/services.php`配置文件包含以下选项：

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  //e.g. us-east-1
    ],

<a name="sending-mail"></a>
## 发送邮件

要发送消息，请在`Mail`facade上使用`send`方法，该方法接受三个参数。 第一个参数是一个唯一的*邮件代码*，用于定位[邮件视图](#mail-views)或[邮件模板](#mail-templates)。 第二个参数是您希望传递给视图的数据数组。 第三个参数是一个`Closure`回调函数，它接收一个消息实例，允许您自定义邮件消息的收件人，主题和其他方面：

    //这些变量在消息中以Twig的形式提供
    $vars = ['name' => 'Joe', 'user' => 'Mary'];

    Mail::send('acme.blog::mail.message', $vars, function($message) {

        $message->to('admin@domain.tld', 'Admin Person');
        $message->subject('This is a reminder');

    });

由于我们在上面的示例中传递了一个包含`name`键的数组，因此我们可以使用以下Twig标记在电子邮件视图中显示该值：

    {{ name }}

> **注意:** 你应该避免在你的消息中传递一个`message`变量，这个变量总是被传递并允许[内联嵌入附件](#adlection)。

#### 快速发送

October还包括一个名为`sendTo`的替代方法，可以简化发送邮件：

    //使用无名称发送到地址
    Mail::sendTo('admin@domain.tld', 'acme.blog::mail.message', $params);

    //使用对象的属性发送
    Mail::sendTo($user, 'acme.blog::mail.message', $params);

    //发送到多个地址
    Mail::sendTo(['admin@domain.tld' => 'Admin Person'], 'acme.blog::mail.message', $params);

    //或者，发送不带参数的原始消息
    Mail::rawTo('admin@domain.tld', 'Hello friend');

`sendTo`中的第一个参数用于收件人可以采用不同的值类型：

类型 | 描述
------------- | -------------
String | 单个收件人地址，未定义名称。
Array | 多个收件人，其中数组键是地址，值是名称。
Object | 单个收件人对象，其中 *email* 属性用于地址，*name* 可选地用于名称。
Collection | 如上所述的收件人对象的集合。

`sendTo`的完整签名如下：

    Mail::sendTo($recipient, $message, $params, $callback, $options);

- `$recipient` 定义如上。
- `$message` 是原始发送的模板名称或消息内容。
- `$params` 模板内可用的变量数组。
- `$callback` 使用一个参数调用，如`send`方法所述的消息构建器(可选，默认为null)。 如果不是可调用值，则作为下一个选项参数的替代。
- `$options` 作为数组传递的自定义发送选项(可选)

支持以下自定义发送`$options`

- **queue** 指定是对消息进行排队还是直接发送(可选，默认为false)。
- **bcc** 指定将收件人添加为Bcc或常规To地址(默认为false)。

#### 构建消息

如前所述，`send`方法的第三个参数是`Closure`，允许您在电子邮件本身上指定各种选项。 使用此结束，您可以指定消息的其他属性，例如抄送，盲抄送等：

    Mail::send('acme.blog::mail.welcome', $vars, function($message) {

        $message->from('us@example.com', 'October');
        $message->to('foo@example.com')->cc('bar@example.com');

    });

以下是`$message`消息构建器实例上可用方法的列表：

    $message->from($address, $name = null);
    $message->sender($address, $name = null);
    $message->to($address, $name = null);
    $message->cc($address, $name = null);
    $message->bcc($address, $name = null);
    $message->replyTo($address, $name = null);
    $message->subject($subject);
    $message->priority($level);
    $message->attach($pathToFile, array $options = []);

    //附加原始$data字符串中的文件...
    $message->attachData($data, $name, array $options = []);

    //Get the underlying SwiftMailer message instance...
    $message->getSwiftMessage();

> **注意:** 传递给`Mail::send` Closure的消息实例扩展了[SwiftMailer](http://swiftmailer.org)消息类，允许您调用该类的任何方法来构建您的电子邮件。

#### 邮寄纯文本

默认情况下，假定给`send`方法的视图包含HTML。 但是，通过将数组作为第一个参数传递给`send`方法，除了HTML视图之外，您还可以指定要发送的纯文本视图：

    Mail::send(['acme.blog::mail.html', 'acme.blog::mail.text'], $data, $callback);

或者，如果您只需要发送纯文本电子邮件，则可以使用数组中的`text`键指定：

    Mail::send(['text' => 'acme.blog::mail.text'], $data, $callback);

#### 邮寄解析的原始字符串

如果您希望直接通过电子邮件发送原始字符串，可以使用`raw`方法。 此内容将由Markdown解析。

    Mail::raw('Text to e-mail', function ($message) {
        //
    });

此外，这个字符串将由Twig解析，如果您希望将变量传递给此环境，请使用`send`方法，将内容作为`raw`键传递。

    Mail::send(['raw' => 'Text to email'], $vars, function ($message) {
        //
    });

#### 邮寄原始字符串

如果传递包含`text`或`html`键的数组，这将是发送邮件的显式请求。 不使用布局或markdown解析。

    Mail::raw([
        'text' => 'This is plain text',
        'html' => '<strong>This is HTML</strong>'
    ], function ($message) {
        //
    });

<a name="attachments"></a>
### 附件

要向电子邮件添加附件，请在传递给Closure的`$message`对象上使用`attach`方法。 `attach`方法接受文件的完整路径作为其第一个参数：

    Mail::send('acme.blog::mail.welcome', $data, function ($message) {
        //

        $message->attach($pathToFile);
    });

将文件附加到消息时，您还可以通过将`array`作为第二个参数传递给`attach`方法来指定显示名称和/或MIME类型：

    $message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);

<a name="inline-attachments"></a>
### 内联附件

#### 在邮件内容中嵌入图像

将内嵌图像嵌入电子邮件通常很麻烦; 但是，有一种方便的方法可以将图像附加到您的电子邮件中并检索相应的CID。 要嵌入内嵌图像，请在电子邮件视图中的`message`变量上使用`embed`方法。 请记住，`message`变量可用于所有邮件视图：

    <body>
        Here is an image:

        <img src="{{ message.embed(pathToFile) }}">
    </body>

如果您计划使用排队的电子邮件，请确保该文件的路径是绝对的。 要实现这一点，您只需使用[app filter](../markup/filter-app)：

    <body>
        Here is an image:
        {% set pathToFile = 'storage/app/media/path/to/file.jpg' | app %}
        <img src="{{ message.embed(pathToFile) }}">
    </body>   
    
#### 在邮件内容中嵌入原始数据

如果您已经有一个希望嵌入电子邮件消息的原始数据字符串，则可以在`message`变量上使用`embedData`方法：

    <body>
        Here is an image from raw data:

        <img src="{{ message.embedData(data, name) }}">
    </body>

<a name="queueing-mail"></a>
### 排队邮件

#### 邮件使用队列

由于发送邮件消息可以大大延长应用程序的响应时间，因此许多开发人员选择将消息排队以进行后台发送。 使用内置的[统一队列API](services-queues.md)很容易。 要对邮件消息进行排队，请在`Mail`facade上使用`queue`方法：

    Mail::queue('acme.blog::mail.welcome', $data, function ($message) {
        //
    });

此方法将自动将作业推送到队列以在后台发送邮件消息。 当然，在使用此功能之前，您需要[配置队列](services-queues.md)。

#### 延迟消息排队

如果您希望延迟传送排队的电子邮件，可以使用`later`方法。 要开始，只需将您希望延迟发送消息的秒数作为方法的第一个参数传递：

    Mail::later(5, 'acme.blog::mail.welcome', $data, function ($message) {
        //
    });

#### 推送到特定队列

如果您希望指定一个特定的队列来推送消息，您可以使用`queueOn`和`laterOn`方法：

    Mail::queueOn('queue-name', 'acme.blog::mail.welcome', $data, function ($message) {
        //
    });

    Mail::laterOn('queue-name', 5, 'acme.blog::mail.welcome', $data, function ($message) {
        //
    });

<a name="message-content"></a>
## 消息内容

可以使用邮件视图或邮件模板在October发送邮件。 邮件视图由 **/views** 目录中文件系统中的应用程序或插件提供。 而使用后端界面通过 *系统>邮件模版* 管理邮件模板。 所有邮件消息都支持使用Twig进行标记。

可选地，邮件视图可以[使用`registerMailTemplates`方法[在插件注册文件中注册](#mail-template-registration)。 这将自动生成邮件模板，并允许使用后端界面对其进行自定义。

<a name="mail-views"></a>
### 邮件视图

邮件视图驻留在文件系统中，使用的代码表示视图文件的路径。 例如，使用代码 **author.plugin::mail.message**发送邮件将使用以下文件中的内容：

    plugins/                <=== Plugins directory
      author/               <=== "author" segment
        plugin/             <=== "plugin" segment
          views/            <=== View directory
            mail/           <=== "mail" segment
              message.htm    <=== "message" segment

邮件视图文件中的内容最多可包含3个部分：**配置**，**纯文本**和 **HTML标记**。 截面用`==`序列分隔。 例如：

    subject = "Your product has been added to OctoberCMS project"
    ==

    Hi {{ name }},

    Good news! User {{ user }} just added your product "{{ product }}" to a project.

    This message was sent using no formatting (plain text)
    ==

    <p>Hi {{ name }},</p>

    <p>Good news! User {{ user }} just added your product <strong>{{ product }}</strong> to a project.</p>

    <p>This email was sent using formatting (HTML)</p>

> **注意:** 邮件视图中支持基本的Twig标记和表达式。

**纯文本**部分是可选的，视图只能包含配置和HTML标记部分。

    subject = "Your product has been added to OctoberCMS project"
    ==

    <p>Hi {{ name }},</p>

    <p>This email does not support plain text.</p>

    <p>Sorry about that!</p>

#### 配置部分

配置部分设置邮件视图参数。 支持以下配置参数：

参数 | 描述
------------- | -------------
**subject** | 邮件主题，必填。
**layout** | [邮件布局](#mail-layouts)代码，可选。 默认值为`default`。

<a name="mail-templates"></a>
### 使用邮件模板

邮件模板驻留在数据库中，可以通过 *设置>邮件>邮件模板* 在后端区域中创建。 模板中指定的**代码**是唯一标识符，一旦创建就无法更改。

发送这些电子邮件的过程是相同的。 例如，如果您使用代码 *this.is.my.email* 创建模板，则可以使用以下PHP代码发送它：

    Mail::send('this.is.my.email', $data, function($message) use ($user)
    {
        [...]
    });

> **注意:** 如果系统中不存在邮件模板，则此代码将尝试查找具有相同代码的邮件视图。

#### 自动生成的模板

邮件模板也可以通过[已注册的邮件视图](#mail-template-registration)自动生成。 **code**值与邮件视图路径相同(例如：author.plugin：mail.message)。 如果邮件视图定义了 **layout** 参数，则将使用此参数为模板提供布局。

首次保存生成的模板时，将在为指定的代码发送邮件时使用自定义内容。 在此上下文中，邮件视图可以被视为*默认视图*。

<a name="mail-layouts"></a>
### 使用邮件布局

通过选择 *设置>邮件>邮件模板* 并单击 *布局* 选项卡，可以创建邮件布局。 这些行为就像CMS布局一样，它们包含邮件消息的脚手架。 邮件视图和模板支持使用邮件布局。

默认情况下，十月带有两个主要邮件布局：

布局 | Code | 描述
------------- | ------------- | -------------
Default | default | Used for public facing, front-end mail
System | system | Used for internal, back-end mail

<a name="mail-template-registration"></a>
### 注册邮件模板

邮件视图可以注册为在后端自动生成的模板，以便进行自定义。 可以通过*设置>邮件模板*菜单自定义邮件模板。 可以通过覆盖[插件注册类](plugin-registration.md #registration-file)的`registerMailTemplates`方法来注册模板。

    public function registerMailTemplates()
    {
        return [
            'rainlab.user::mail.activate' => 'Activation mail sent to new users.',
            'rainlab.user::mail.restore'  => 'Password reset instructions for front-end users.'
        ];
    }

该方法应返回一个数组，其中键是[邮件视图名称](#mail-views)，该值提供有关邮件模板用途的简要说明。

<a name="mail-global-variables"></a>
### 全局变量

您可以使用`View::share`方法注册所有邮件模板全局可用的变量。

    View::share('site_name', 'OctoberCMS');

可以在[插件注册文件](plugin-registration.md)的寄存器或引导方法内调用此代码。 使用上面的示例，变量`{{site_name}}`将在所有邮件模板中可用。

<a name="mail-and-local-development"></a>
## 邮件和本地开发

在开发发送电子邮件的应用程序时，您可能不希望实际发送电子邮件到实时电子邮件地址。 有几种方法可以“禁用”实际发送的电子邮件。

#### 日志驱动

一种解决方案是在本地开发期间使用`log`邮件驱动程序。 此驱动程序会将所有电子邮件写入您的日志文件以供检查。 有关按环境配置应用程序的更多信息，请查看[配置文档](../setup/configuration)。

#### Universal to

另一种解决方案是设置框架发送的所有电子邮件的通用接收者。 这样，应用程序生成的所有电子邮件都将发送到特定地址，而不是发送邮件时实际指定的地址。 这可以通过`config/mail.php`配置文件中的`to`选项来完成：

    'to' => [
        'address' => 'dev@example.com',
        'name' => 'Dev Example'
    ],

#### 假装邮件模式

您可以使用`Mail::pretend`方法动态禁用发送邮件。 当邮件程序处于伪装模式时，邮件将写入应用程序的日志文件，而不是发送给收件人。

    Mail::pretend();
