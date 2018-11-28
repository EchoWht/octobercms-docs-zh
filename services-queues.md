# 队列服务

- [配置](#configuration)
- [基本用法](#basic-usage)
- [Queueing closures](#queueing-closures)
- [运行队列](#running-the-queue-worker)
- [Supervisor configuration](#supervisor-configuration)
- [守护进程队列工作者](#daemon-queue-worker)
- [推送队列](#push-queues)
- [失败的任务](#failed-jobs)

<a name="configuration"></a>
## 配置

队列允许您将处理耗时的任务（例如发送电子邮件）推迟到稍后的时间，从而大大加快了对应用程序的Web请求。

队列配置文件存储在`config / queue.php`中。 在此文件中，您将找到包含的每个队列驱动程序的连接配置，例如数据库， [Beanstalkd](http://kr.github.com/beanstalkd), [IronMQ](http://iron.io), [Amazon SQS](http://aws.amazon.com/sqs), [Redis](http://redis.io),，null和同步（用于本地使用）驱动程序。 `null`队列驱动程序只是丢弃排队的作业，因此永远不会执行它们。

### 先决条件

在使用Amazon SQS，Beanstalkd，IronMQ或Redis驱动程序之前，您需要安装[Drivers plugin](http://octobercms.com/plugin/october-drivers)。

<a name="basic-usage"></a>
## 基本用法

#### 将作业推送到队列中

要将新作业推送到队列，请使用`Queue :: push`方法：

    Queue::push('SendEmail', ['message' => $message]);

`push`方法的第一个参数是应该用于处理作业的类的名称。 第二个参数是应该传递给处理程序的数据数组。 应该像这样定义一个作业处理程序：

    class SendEmail
    {
        public function fire($job, $data)
        {
            //
        }
    }

请注意，唯一需要的方法是`fire`，它接收一个`Job`实例以及被推入队列的`data`数组。

#### 指定自定义处理程序方法

如果您希望作业使用`fire`以外的方法，则可以在推送作业时指定方法：

    Queue::push('SendEmail@send', ['message' => $message]);

#### 指定作业的队列名称

为jobaYou指定队列名称还可以指定作业应发送到的队列/管道：

    Queue::push('SendEmail@send', ['message' => $message], 'emails');

#### 延迟作业的执行

有时您可能希望延迟执行排队的作业。 例如，您可能希望将注册后15分钟向客户发送电子邮件的作业排队。 您可以使用`Queue :: later`方法完成此操作：

    $date = Carbon::now()->addMinutes(15);

    Queue::later($date, 'SendEmail', ['message' => $message]);

在此示例中，我们使用[Carbon](https://github.com/briannesbitt/Carbon) 日期库来指定我们希望分配给作业的延迟。 或者，您可以将要延迟的秒数作为整数传递。

> **注意:** Amazon SQS服务的延迟限制为900秒（15分钟）。

#### 队列和模型

如果排队的作业在其数据中采用模型，则只有模型的标识符将序列化到队列中。 实际处理作业时，队列系统将自动从数据库中重新检索完整的模型实例。 它对您的应用程序完全透明，可以防止序列化完整模型实例时出现的问题。

#### 删除已处理的作业

处理完作业后，必须从队列中删除它，这可以通过`Job`实例上的`delete`方法完成：

    public function fire($job, $data)
    {
        // Process the job...

        $job->delete();
    }

#### 将作业释放回队列

如果您希望将作业释放回队列，可以通过`release`方法执行此操作：

    public function fire($job, $data)
    {
        // Process the job...

        $job->release();
    }

您还可以指定在作业发布之前等待的秒数：

    $job->release(5);

#### 检查运行尝试次数

如果在处理作业时发生异常，它将自动释放回队列。 您可以使用`attempts`方法检查运行作业的尝试次数：

    if ($job->attempts() > 3) {
        //
    }

#### 访问作业ID

您还可以访问作业标识符：

    $job->getJobId();

<a name="queueing-closures"></a>
## 队列闭包

您也可以将Closure推入队列。 这对于需要排队的快速，简单的任务非常方便：

#### 将闭包推入队列

    Queue::push(function($job) use ($id) {
        Account::delete($id);

        $job->delete();
    });

> **注意:** 不要在排队的Closures通过`use`指令使用对象，可以传递主键并从队列作业中重新拉出关联的模型。 这通常可以避免意外的序列化行为。

使用Iron.io [推送队列](#push-queues)时，您应该采取额外的预防措施排队闭包。 接收队列消息的端点应检查令牌以验证请求是否实际来自Iron.io. 例如，您的推送队列端点应该类似于：`https：//example.com/queue/receive？token = SecretToken`。 然后，您可以在编组队列请求之前检查应用程序中的秘密令牌的值。

<a name="running-the-queue-worker"></a>
## 运行队列工作程序

October 包括一些将处理队列中的作业的[控制台命令](../console/commands)。

要在将新作业推入队列时处理它们，请运行`queue：work`命令：

    php artisan queue:work

此任务启动后，它将继续运行，直到手动停止。 您可以使用[Supervisor]（#supervisor-configuration）等进程监视器来确保队列工作程序不会停止运行。

Queue worker processes store the booted application state in memory. They will not recognize changes in your code after they have been started. When deploying changes, restart queue workers.

#### Processing a single job

To process only the first job on the queue, use the `--once` option:

    php artisan queue:work --once

#### Specifying the connection & Queue

You may also specify which queue connection the worker should utilize:

    php artisan queue:work connection


You may pass a comma-delimited list of queue connections to the `work` command to set queue priorities:

    php artisan queue:work --queue=high,low

In this example, jobs on the `high` queue will always be processed before moving onto jobs from the `low` queue.

#### Specifying the job timeout parameter

You may also set the length of time (in seconds) each job should be allowed to run:

    php artisan queue:work --timeout=60

#### Specifying queue sleep duration

In addition, you may specify the number of seconds to wait before polling for new jobs:

    php artisan queue:work --sleep=5

Note that the queue only "sleeps" if no jobs are on the queue. If more jobs are available, the queue will continue to work them without sleeping.

<a name="supervisor-configuration"></a>
## Supervisor configuration

### Installing Supervisor

Supervisor is a process monitor for the Linux operating system, and will automatically restart your `queue:work` process if it fails. To install Supervisor on Ubuntu, you may use the following command:

    sudo apt-get install supervisor

### Configuring Supervisor

Supervisor configuration files are typically stored in the `/etc/supervisor/conf.d` directory. Within this directory, you may create any number of configuration files that instruct supervisor how your processes should be monitored. For example, let's create a `october-worker.conf` file that starts and monitors a `queue:work` process:

    [program:october-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /path/to/october/artisan queue:work --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=october
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/path/to/october/worker.log
    
In this example, the `numprocs` directive will instruct Supervisor to run 8 `queue:work` processes and monitor all of them, automatically restarting them if they fail. Of course, you should change the `queue:work` portion of the command directive to reflect your desired queue connection. The `user` directive should be changed to the name of a user that has permission to run the command.

### Starting Supervisor

Once the configuration file has been created, you may update the Supervisor configuration and start the processes using the following commands:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start october-worker:*
    
For more information on Supervisor, consult the [Supervisor documentation](http://supervisord.org/index.html).

<a name="daemon-queue-worker"></a>
## Daemon queue worker

The `queue:work` also includes a `--daemon` option for forcing the queue worker to continue processing jobs without ever re-booting the framework. This results in a significant reduction of CPU usage when compared to the `queue:work` command, but at the added complexity of needing to drain the queues of currently executing jobs during your deployments.

To start a queue worker in daemon mode, use the `--daemon` flag:

    php artisan queue:work connection --daemon

    php artisan queue:work connection --daemon --sleep=3

    php artisan queue:work connection --daemon --sleep=3 --tries=3

You may use the `php artisan help queue:work` command to view all of the available options.

### Deploying with daemon queue workers

The simplest way to deploy an application using daemon queue workers is to put the application in maintenance mode at the beginning of your deployment. This can be done using the back-end settings area. Once the application is in maintenance mode, October will not accept any new jobs off of the queue, but will continue to process existing jobs.

The easiest way to restart your workers is to include the following command in your deployment script:

    php artisan queue:restart

This command will instruct all queue workers to restart after they finish processing their current job.

> **Note:** This command relies on the cache system to schedule the restart. By default, APCu does not work for CLI commands. If you are using APCu, add `apc.enable_cli=1` to your APCu configuration.

### Coding for daemon queue workers

Daemon queue workers do not restart the platform before processing each job. Therefore, you should be careful to free any heavy resources before your job finishes. For example, if you are doing image manipulation with the GD library, you should free the memory with `imagedestroy` when you are done.

Similarly, your database connection may disconnect when being used by long-running daemon. You may use the `Db::reconnect` method to ensure you have a fresh connection.

<a name="failed-jobs"></a>
## Failed jobs

Since things don't always go as planned, sometimes your queued jobs will fail. Don't worry, it happens to the best of us! There is a convenient way to specify the maximum number of times a job should be attempted. After a job has exceeded this amount of attempts, it will be inserted into a `failed_jobs` table. The failed jobs table name can be configured via the `config/queue.php` configuration file.

You can specify the maximum number of times a job should be attempted using the `--tries` switch on the `queue:work` command:

    php artisan queue:work connection-name --tries=3

If you would like to register an event that will be called when a queue job fails, you may use the `Queue::failing` method. This event is a great opportunity to notify your team via e-mail or another third party service.

    Queue::failing(function($connection, $job, $data) {
        //
    });

You may also define a `failed` method directly on a queue job class, allowing you to perform job specific actions when a failure occurs:

    public function failed($data)
    {
        // Called when the job is failing...
    }
    
The original array of `data` will also be automatically passed onto the failed method.    

### Retrying failed jobs

To view all of your failed jobs, you may use the `queue:failed` Artisan command:

    php artisan queue:failed

The `queue:failed` command will list the job ID, connection, queue, and failure time. The job ID may be used to retry the failed job. For instance, to retry a failed job that has an ID of 5, the following command should be issued:

    php artisan queue:retry 5

If you would like to delete a failed job, you may use the `queue:forget` command:

    php artisan queue:forget 5

To delete all of your failed jobs, you may use the `queue:flush` command:

    php artisan queue:flush
