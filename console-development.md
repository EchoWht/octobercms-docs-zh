# 控制台开发

- [介绍](#introduction)
- [建立一个命令](#building-a-command)
    - [定义参数](#defining-arguments)
    - [定义选项](#defining-options)
    - [写输出](#writing-output)
    - [检索输入](#retrieving-input)
- [注册命令](#registering-commands)
- [调用其他命令](#calling-other-commands)

<a name="introduction"></a>
## 介绍

除了提供的控制台命令之外，您还可以构建自己的自定义命令以使用您的应用程序。 您可以将自定义命令存储在插件**控制台**目录中。 您可以使用[命令行脚手架工具](../console/scaffolding#scaffold-create-command)生成类文件。

<a name="building-a-command"></a>
## 建立一个命令

如果要创建名为`acme:mycommand`的控制台命令，可以在名为**plugins/acme/blog/console/MyCommand.php**的文件中为该命令创建关联类，并粘贴以下内容以获取开始：

    <?php namespace Acme\Blog\Console;

    use Illuminate\Console\Command;
    use Symfony\Component\Console\Input\InputOption;
    use Symfony\Component\Console\Input\InputArgument;

    class MyCommand extends Command
    {
        /**
         * @var string 控制台命令名称。
         */
        protected $name = 'acme:mycommand';

        /**
         * @var string 控制台命令描述。
         */
        protected $description = 'Does something cool.';

        /**
         * 执行console命令。
         * @return void
         */
        public function handle()
        {
            $this->output->writeln('Hello world!');
        }

        /**
         * 获取控制台命令参数。
         * @return array
         */
        protected function getArguments()
        {
            return [];
        }

        /**
         * 获取控制台命令选项。
         * @return array
         */
        protected function getOptions()
        {
            return [];
        }

    }

一旦创建了类，就应该填写类的`name`和`description`属性，这些属性将在命令`list`屏幕上显示命令时使用。

执行命令时将调用`handle`方法。 您可以在此方法中放置任何命令逻辑。

<a name="defining-arguments"></a>
### 定义参数

通过从`getArguments`方法返回数组值来定义参数，您可以在其中定义命令接收的任何参数。 例如：

        /**
         * 获取控制台命令参数。
         * @return array
         */
        protected function getArguments()
        {
            return [
                ['example', InputArgument::REQUIRED, 'An example argument.'],
            ];
        }

定义`arguments`时，数组定义值表示以下内容：

    array($name, $mode, $description, $defaultValue)

参数`mode`可以是以下任何一个：`InputArgument::REQUIRED`或`InputArgument::OPTIONAL`。 

<a name="defining-options"></a>
### 定义选项

通过从`getOptions`方法返回数组值来定义选项。 与参数类似，此方法应返回一个命令数组，这些命令由数组选项列表描述。 例如：

        /**
         * 获取控制台命令选项。
         * @return array
         */
        protected function getOptions()
        {
            return [
                ['example', null, InputOption::VALUE_OPTIONAL, 'An example option.', null],
            ];
        }

定义`options`时，数组定义值表示以下内容：

    array($name, $shortcut, $mode, $description, $defaultValue)

对于选项，参数`mode`可以是：`InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE`.

`VALUE_IS_ARRAY`模式表示在调用命令时可以多次使用该开关：

    php artisan foo --option=bar --option=baz

`VALUE_NONE`选项表示该选项仅用作“开关”：

    php artisan foo --option

<a name="retrieving-input"></a>
### 检索输入

当您的命令正在执行时，您显然需要访问应用程序接受的参数和选项的值。 为此，您可以使用`argument`和`option`方法：

#### 检索命令参数的值

    $value = $this->argument('name');

#### 检索所有参数

    $arguments = $this->argument();

#### 检索命令选项的值

    $value = $this->option('name');

#### 检索所有选项

    $options = $this->option();

<a name="writing-output"></a>
### 写输出

要将输出发送到控制台，您可以使用`info`，`comment`，`question`和`error`方法。 这些方法中的每一种都将使用适当的ANSI颜色用于其目的。

#### 发送信息

    $this->info('Display this on the screen');

#### 发送错误消息

    $this->error('Something went wrong!');

#### 要求用户输入

您也可以使用`ask`和`confirm`方法来提示用户输入：

    $name = $this->ask('What is your name?');

#### 要求用户提供秘密输入

    $password = $this->secret('What is the password?');

#### 要求用户确认

    if ($this->confirm('Do you wish to continue? [yes|no]'))
    {
        //
    }

您还可以为`confirm`方法指定一个默认值，该方法应为`true`或`false`：

    $this->confirm($question, true);
    
#### 进度条

对于长时间运行的任务，显示进度指示器可能会有所帮助。 使用输出对象，我们可以启动，前进和停止进度条。 首先，定义流程将迭代的总步骤数。 然后，在处理每个项目后推进Progress Bar：

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

有关更多高级选项，请查看[Symfony Progress Bar组件文档](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html)。

<a name="registering-commands"></a>
## 注册命令

#### 注册控制台命令

命令类完成后，您需要注册它以便可以使用它。 这通常使用`registerConsoleCommand`辅助方法在[插件注册文件](../plugin/registration#registration-methods)的`register`方法中完成。

    class Blog extends PluginBase
    {
        public function pluginDetails()
        {
            [...]
        }

        public function register()
        {
            $this->registerConsoleCommand('acme.mycommand', 'Acme\Blog\Console\MyConsoleCommand');
        }
    }

或者，插件可以在插件目录中提供名为**init.php**的文件，您可以使用该文件来放置命令注册逻辑。 在此文件中，您可以使用`Artisan::add`方法注册命令：

    Artisan::add(new Acme\Blog\Console\MyCommand);

#### 在应用程序容器中注册命令

如果您的命令在[应用程序容器](../services/application#app-container),中注册，您可以使用`Artisan::resolve`方法将其提供给Artisan：

    Artisan::resolve('binding.name');

#### 在服务提供商中注册命令

如果需要在[服务提供者](application#service-providers)中注册命令，则应该从提供者的`boot`方法调用`commands`方法，传递[container](application#app-container)绑定 对于命令：

    public function boot()
    {
        $this->app->singleton('acme.mycommand', function() {
            return new \Acme\Blog\Console\MyConsoleCommand;
        });

        $this->commands('acme.mycommand');
    }

<a name="calling-other-commands"></a>
## 调用其他命令

有时您可能希望从命令中调用其他命令。 您可以使用`call`方法执行此操作：

    $this->call('october:up');

您还可以将参数作为数组传递：

    $this->call('plugin:refresh', ['name' => 'October.Demo']);

以及选项：

    $this->call('october:update', ['--force' => true]);

