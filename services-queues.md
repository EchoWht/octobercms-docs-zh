# 队列服务

- [配置](#configuration)
- [基本用法](#basic-usage)
- [队列闭包](#queueing-closures)
- [运行队列](#running-the-queue-worker)
- [Supervisor配置](#supervisor-configuration)
- [守护进程队列工作者](#daemon-queue-worker)
- [推送队列](#push-queues)
- [失败的任务](#failed-jobs)

<a name="configuration"></a>
## 配置

队列允许您将处理耗时的任务（例如发送电子邮件）推迟到稍后的时间，从而大大加快了对应用程序的Web请求。

队列配置文件存储在`config/queue.php`中。 在此文件中，您将找到包含的每个队列驱动程序的连接配置，例如数据库， [Beanstalkd](http://kr.github.com/beanstalkd), [IronMQ](http://iron.io), [Amazon SQS](http://aws.amazon.com/sqs), [Redis](http://redis.io),，null和同步（用于本地使用）驱动程序。 `null`队列驱动程序只是丢弃排队的作业，因此永远不会执行它们。

### 先决条件

在使用Amazon SQS，Beanstalkd，IronMQ或Redis驱动程序之前，您需要安装[Drivers plugin](http://octobercms.com/plugin/october-drivers)。

<a name="basic-usage"></a>
## 基本用法

#### 将作业推送到队列中

要将新作业推送到队列，请使用`Queue::push`方法:

    Queue::push('SendEmail', ['message' => $message]);

`push`方法的第一个参数是应该用于处理作业的类的名称。 第二个参数是应该传递给处理程序的数据数组。 应该像这样定义一个作业处理程序:

    class SendEmail
    {
        public function fire($job, $data)
        {
            //
        }
    }

请注意，唯一需要的方法是`fire`，它接收一个`Job`实例以及被推入队列的`data`数组。

#### 指定自定义处理程序方法

如果您希望作业使用`fire`以外的方法，则可以在推送作业时指定方法:

    Queue::push('SendEmail@send', ['message' => $message]);

#### 指定作业的队列名称

为jobaYou指定队列名称还可以指定作业应发送到的队列/管道:

    Queue::push('SendEmail@send', ['message' => $message], 'emails');

#### 延迟作业的执行

有时您可能希望延迟执行排队的作业。 例如，您可能希望将注册后15分钟向客户发送电子邮件的作业排队。 您可以使用`Queue::later`方法完成此操作:

    $date = Carbon::now()->addMinutes(15);

    Queue::later($date, 'SendEmail', ['message' => $message]);

在此示例中，我们使用[Carbon](https://github.com/briannesbitt/Carbon) 日期库来指定我们希望分配给作业的延迟。 或者，您可以将要延迟的秒数作为整数传递。

> **注意:** Amazon SQS服务的延迟限制为900秒（15分钟）。

#### 队列和模型

如果排队的作业在其数据中采用模型，则只有模型的标识符将序列化到队列中。 实际处理作业时，队列系统将自动从数据库中重新检索完整的模型实例。 它对您的应用程序完全透明，可以防止序列化完整模型实例时出现的问题。

#### 删除已处理的作业

处理完作业后，必须从队列中删除它，这可以通过`Job`实例上的`delete`方法完成:

    public function fire($job, $data)
    {
        // Process the job...

        $job->delete();
    }

#### 将作业释放回队列

如果您希望将作业释放回队列，可以通过`release`方法执行此操作:

    public function fire($job, $data)
    {
        // Process the job...

        $job->release();
    }

您还可以指定在作业发布之前等待的秒数:

    $job->release(5);

#### 检查运行尝试次数

如果在处理作业时发生异常，它将自动释放回队列。 您可以使用`attempts`方法检查运行作业的尝试次数:

    if ($job->attempts() > 3) {
        //
    }

#### 访问作业ID

您还可以访问作业标识符:

    $job->getJobId();

<a name="queueing-closures"></a>
## 队列闭包

您也可以将Closure推入队列。 这对于需要排队的快速，简单的任务非常方便:

#### 将闭包推入队列

    Queue::push(function($job) use ($id) {
        Account::delete($id);

        $job->delete();
    });

> **注意:** 不要在排队的Closures通过`use`指令使用对象，可以传递主键并从队列作业中重新拉出关联的模型。 这通常可以避免意外的序列化行为。

使用Iron.io [推送队列](#push-queues)时，您应该采取额外的预防措施排队闭包。 接收队列消息的端点应检查令牌以验证请求是否实际来自Iron.io. 例如，您的推送队列端点应该类似于:`https://example.com/queue/receive？token = SecretToken`。 然后，您可以在编组队列请求之前检查应用程序中的秘密令牌的值。

<a name="running-the-queue-worker"></a>
## 运行队列工作程序

October 包括一些将处理队列中的作业的[控制台命令](../console/commands)。

要在将新作业推入队列时处理它们，请运行`queue:work`命令:

    php artisan queue:work

此任务启动后，它将继续运行，直到手动停止。 您可以使用[Supervisor]（#supervisor-configuration）等进程监视器来确保队列工作程序不会停止运行。

队列工作进程将引导的应用程序状态存储在内存中。 它们在启动后无法识别代码中的更改。 部署更改时，请重新启动队列工作程序。

#### 处理单个工作

要仅处理队列中的第一个作业，请使用`--once`选项:

    php artisan queue:work --once

#### 指定连接和队列

您还可以指定工作人员应使用的队列连接:

    php artisan queue:work connection


您可以将逗号分隔的队列连接列表传递给`work`命令以设置队列优先级:

    php artisan queue:work --queue=high,low

在这个例子中，在从`low`队列转移到作业之前，将始终处理`high`队列上的作业。

#### 指定作业超时参数

您还可以设置允许每个作业运行的时间长度（以秒为单位）:

    php artisan queue:work --timeout=60

#### 指定队列睡眠持续时间

此外，您可以指定轮询新作业之前要等待的秒数:

    php artisan queue:work --sleep=5

请注意，如果队列中没有作业，则队列仅“休眠”。 如果有更多可用的作业，队列将继续工作而不会休眠。

<a name="supervisor-configuration"></a>
## Supervisor配置

### 安装Supervisor

Supervisor是Linux操作系统的进程监视器，如果失败，将自动重启`queue:work`进程。 要在Ubuntu上安装Supervisor，您可以使用以下命令:

    sudo apt-get install supervisor

### 配置Supervisor

Supervisor配置文件通常存储在`/etc/supervisor/conf.d`目录中。 在此目录中，您可以创建任意数量的配置文件，以指示supervisor如何监视您的进程。 例如，让我们创建一个`october-worker.conf`文件，它启动并监视`queue:work`进程:

    [program:october-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /path/to/october/artisan queue:work --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=october
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/path/to/october/worker.log
    
在这个例子中，`numprocs`指令将指示Supervisor运行8`queue:work`进程并监视所有进程，如果失败则自动重启它们。 当然，您应该更改命令指令的`queue:work`部分以反映您所需的队列连接。 应将`user`指令更改为有权运行该命令的用户的名称。

### 开始Supervisor

创建配置文件后，您可以使用以下命令更新Supervisor配置并启动进程:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start october-worker:*
    
有关Supervisor的更多信息，请参阅[Supervisor文档](http://supervisord.org/index.html)。

<a name="daemon-queue-worker"></a>
## 守护进程队列工作者

`queue:work`还包括一个`--daemon`选项，用于强制队列工作者继续处理作业而无需重新启动框架。 与`queue:work`命令相比，这会显着降低CPU使用率，但会增加部署期间需要耗尽当前正在执行的作业队列的复杂性。

要以守护进程模式启动队列工作程序，请使用`--daemon`标志：

    php artisan queue:work connection --daemon

    php artisan queue:work connection --daemon --sleep=3

    php artisan queue:work connection --daemon --sleep=3 --tries=3

您可以使用`php artisan help queue:work`命令查看所有可用选项。

### 使用守护程序队列工作程序进行部署

使用守护程序队列工作程序部署应用程序的最简单方法是在部署开始时将应用程序置于维护模式。 这可以使用后端设置区域完成。 应用程序处于维护模式后，October将不接受队列中的任何新作业，但将继续处理现有作业。

重新启动worker的最简单方法是在部署脚本中包含以下命令：

    php artisan queue:restart

此命令将指示所有队列工作程序在完成当前作业的处理后重新启动。

> **注意:** 此命令依赖于缓存系统来安排重新启动。 默认情况下，APCu不适用于CLI命令。 如果您使用的是APCu，请将“apc.enable_cli = 1”添加到您的APCu配置中。

### 守护程序队列工作程序的编码

守护程序队列工作程序在处理每个作业之前不会重新启动平台。 因此，在工作完成之前，您应该小心释放任何重型资源。 例如，如果您使用GD库进行图像处理，则在完成后应使用`imagedestroy`释放内存。

同样，数据库连接在长时运行的守护程序使用时可能会断开连接。 您可以使用`Db::reconnect`方法来确保您有新的连接。

<a name="failed-jobs"></a>
## 失败的任务

由于事情并不总是按计划进行，因此有时排队的工作会失败。 别担心，它发生在我们最好的人身上！ 有一种方便的方法可以指定尝试作业的最大次数。 作业超过此尝试次数后，它将被插入到“failed_jobs”表中。 可以通过`config/queue.php`配置文件配置失败的作业表名称。

您可以使用`queue：work`命令中的`--tries`开关指定应尝试作业的最大次数：

    php artisan queue:work connection-name --tries=3

如果要注册在队列作业失败时将调用的事件，可以使用`Queue::failing`方法。 此活动是通过电子邮件或其他第三方服务通知您的团队的绝佳机会。

    Queue::failing(function($connection, $job, $data) {
        //
    });

您还可以直接在队列作业类上定义`failed`方法，允许您在发生故障时执行特定于作业的操作：

    public function failed($data)
    {
        // Called when the job is failing...
    }
    
原始数据`data`也将自动传递给失败的方法。   

### 重试失败的作业

要查看所有失败的作业，可以使用`queue:failed` Artisan命令：

    php artisan queue:failed

`queue:failed`命令将列出作业ID，连接，队列和失败时间。 作业ID可用于重试失败的作业。 例如，要重试ID为5的失败作业，应发出以下命令：

    php artisan queue:retry 5

如果要删除失败的作业，可以使用`queue:forget`命令：

    php artisan queue:forget 5

要删除所有失败的作业，可以使用`queue:flush`命令：

    php artisan queue:flush
