# 任务调度

- [介绍](#introduction)
- [定义时间表](#defining-schedules)
    - [安排频率选项](#schedule-frequency-options)
    - [防止任务重叠](#preventing-task-overlaps)
- [任务输出](#task-output)
- [任务hooks](#task-hooks)

<a name="introduction"></a>
## 介绍

过去，开发人员为他们需要安排的每项任务生成了一个Cron。 然而，这有时会令人头疼。 您的任务计划不再处于源代码管理中，您必须通过SSH连接到服务器才能添加Cron条目。 命令调度程序允许您在应用程序本身内流畅且明确地定义命令调度，并且您的服务器上只需要一个Cron条目。

> **注意**: 有关如何设置调度程序任务的说明，请参阅[安装指南](../setup/installation#crontab-setup) 。

<a name="defining-schedules"></a>
## 定义时间表

您可以通过覆盖[Plugin注册类](registration#registration-file)中的`registerSchedule`方法来定义所有计划任务。 该方法将采用单个`$schedule`参数，用于定义命令及其频率。

首先，我们来看一个安排任务的例子。 在这个例子中，我们将安排每天午夜调用`Closure`。 在`Closure`中，我们将执行数据库查询以清除表：

    class Plugin extends PluginBase
    {
        [...]

        public function registerSchedule($schedule)
        {
            $schedule->call(function () {
                \Db::table('recent_users')->delete();
            })->daily();
        }
    }

除了调度`Closure`调用之外，您还可以安排[控制台命令](../console/commands) 和操作系统命令。 例如，您可以使用`command`方法来安排控制台命令：

    $schedule->command('cache:clear')->daily();

`exec`命令可用于向操作系统发出命令：

    $schedule->exec('node /home/acme/script.js')->daily();

<a name="schedule-frequency-options"></a>
### 安排频率选项

当然，您可以为任务分配各种计划：

方法  | 描述
------------- | -------------
`->cron('* * * * *');`  |  在自定义Cron计划上运行任务
`->everyMinute();`  |  每分钟运行一次任务
`->everyFiveMinutes();`  | 每五分钟运行一次任务
`->everyTenMinutes();`  |  每十分钟运行一次任务
`->everyThirtyMinutes();`  |  每30分钟运行一次任务
`->hourly();`  | 每小时运行一次任务
`->daily();`  |  每天午夜运行任务
`->dailyAt('13:00');`  |  每天13:00运行任务
`->twiceDaily(1, 13);`  |  每天1:00和13:00运行任务
`->weekly();`  |  每周运行一次任务
`->monthly();`  | 每个月运行一次任务

这些方法可以与其他约束相结合，以创建更精细调整的计划，这些计划仅在一周的某些日子运行。 例如，要安排命令在每周星期一运行：

    $schedule->call(function () {
        // Runs once a week on Monday at 13:00...
    })->weekly()->mondays()->at('13:00');

以下是其他计划约束的列表：

方法  | 描述
------------- | -------------
`->weekdays();`  |  将任务限制为工作日
`->sundays();`  |  将任务限制为星期日
`->mondays();`  |  将任务限制为星期一
`->tuesdays();`  |  将任务限制在星期二
`->wednesdays();`  |  将任务限制在星期三
`->thursdays();`  |  将任务限制在星期四
`->fridays();`  |  将任务限制在星期五
`->saturdays();`  |  将任务限制在星期六
`->when(Closure);`  | 根据真实测试限制任务

#### 测试约束

`when`方法可用于根据给定真值测试的结果限制任务的执行。 换句话说，如果给定的`Closure`返回'true`，只要没有其他约束条件阻止任务运行，任务就会执行：

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

<a name="preventing-task-overlaps"></a>
### 防止任务重叠

默认情况下，即使先前的任务实例仍在运行，也会运行计划任务。 为了防止这种情况，您可以使用`withoutOverlapping`方法：

    $schedule->command('emails:send')->withoutOverlapping();

在这个例子中，如果电子邮件：send` [console命令](../console/commands)尚未运行，它将每分钟运行一次。 如果您的任务执行时间变化很大，那么`withoutOverlapping`方法特别有用，可以防止您需要准确预测给定任务需要多长时间。

<a name="task-output"></a>
## 任务输出

调度程序提供了几种方便的方法来处理由计划任务生成的输出。 首先，使用`sendOutputTo`方法，您可以将输出发送到文件以供以后检查：

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

使用`emailOutputTo`方法，您可以将输出通过电子邮件发送到您选择的电子邮件地址。 请注意，必须首先使用`sendOutputTo`方法将输出发送到文件。 在通过电子邮件发送任务输出之前，您应该配置[邮件服务](../services/mail)：

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> **注意:** `emailOutputTo`和`sendOutputTo`方法是`command`方法特有的，`call`不支持。

<a name="task-hooks"></a>
## 任务hooks

使用`before`和`after`方法，您可以指定在计划任务完成之前和之后执行的代码：

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // Task is about to start...
             })
             ->after(function () {
                 // Task is complete...
             });

#### Pinging URLs

使用`pingBefore`和`thenPing`方法，调度程序可以在任务完成之前或之后自动ping给定URL。 此方法对于通知外部服务您的计划任务正在开始或完成非常有用：

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

> 您需要安装[Drivers plugin](http://octobercms.com/plugin/october-drivers) 才能使用`pingBefore($url)`或`thenPing($url)`功能。
