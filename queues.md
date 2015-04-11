# 队列

- [设定](#configuration)
- [基本用法](#basic-usage)
- [队列闭包](#queueing-closures)
- [运行队列监听](#running-the-queue-listener)
- [常驻队列工作](#daemon-queue-worker)
- [推送队列](#push-queues)
- [失败的工作](#failed-jobs)

<a name="configuration"></a>
## 设定

Laravel 队列组件提供了一个统一的队列服务API，队列允许您将一个任务延后执行，例如可以将邮件延后至您指定的时间再发送，进而大幅的加快您开发网站应用程序的效率。

队列的配置文件在 `app/config/queue.php`，在这个文件里您将可以找到框架中每种不同的队列服务的连接配置，其中包含了 [Beanstalkd](http://kr.github.com/beanstalkd)，[IronMQ](http://iron.io)，[Amazon SQS](http://aws.amazon.com/sqs)，[Redis](http://redis.io)，以及同步(本地端使用)驱动设定。

下列的 composer.json 表示您的队列服务所需要的依赖环境：

- Beanstalkd: `pda/pheanstalk ~3.0`
- Amazon SQS: `aws/aws-sdk-php`
- IronMQ: `iron-io/iron_mq`

<a name="basic-usage"></a>
## 基本用法

#### 推送一个工作至队列

要推送一个新的工作至队列，请使用 `Queue::push` 方法：

	Queue::push('SendEmail', array('message' => $message));

#### 定义一个工作处理程序

`push` 方法的第一个参数为应该处理这个工作的类方法名称，第二个参数是一个要传递至处理程序的数组，一个工作处理程序应该参照下列定义：

	class SendEmail {

		public function fire($job, $data)
		{
			//
		}

	}

注意如果您未在第一个参数指定工作处理的类及方法(如 SendEmail@process)，那 `fire` 为类中默认必需的方法用来接收被推送至队列 `Job` 实例以及 `data`。

#### 指定一个特定的处理程序方法

假如您想要用一个 `fire` 以外的方法处理工作，您可以在推送工作时指定方法如下：

	Queue::push('SendEmail@send', array('message' => $message));

#### 指定队列使用特定连接

您也可指定队列工作送至指定的连接：

	Queue::push('SendEmail@send', array('message' => $message), 'emails');

#### 发送相同的数据去多个连接

如果您需要传送一样的数据去几个不同的队列服务器，您也可以使用 `Queue::bulk` 方法：

	Queue::bulk(array('SendEmail', 'NotifyUser'), $payload);

#### 延迟执行一个工作

有时后您也希望延迟一个队列工作的执行，举例来说您希望一个队列工作在客户注册 15 分钟后发送一个 e-mail，您可以使用 `Queue::later` 方法来完成这件事情：

	$date = Carbon::now()->addMinutes(15);

	Queue::later($date, 'SendEmail@send', array('message' => $message));

在这个例子中，我们使用 [Carbon](https://github.com/briannesbitt/Carbon) 日期函数库来指定我们希望队列工作希望延迟的时间，另外您也可传送一个整数来设定您希望延迟的秒数。

> **注意：** The Amazon SQS 服务有 900s 的延迟限制 (15 分钟)。

#### 删除一个处理中的工作

当您已经开始处理完成一个队列工作，它就必需在队列中删除，我们可以通过 `Job` 实例中的 `delete` 方法来完成这件事：

	public function fire($job, $data)
	{
		// Process the job...

		$job->delete();
	}

#### 释放一个工作回到队列中

假如您希望将一个工作释放回队列之中，您可以通过 `release` 方法来完成这件事情：

	public function fire($job, $data)
	{
		// Process the job...

		$job->release();
	}

您也可以指定秒数来让这个工作延迟释放：

	$job->release(5);

#### 检查工作执行次数

当一个工作执行后发生错误，这个工作将会自动的释放回队列当中，您可以通过 `attempts` 方法来检查这个工作已经被执行的次数：

	if ($job->attempts() > 3)
	{
		//
	}

#### 取得一个工作的 ID

您也可以取得这个工作的识别码：

	$job->getJobId();

<a name="queueing-closures"></a>
## 队列闭包

您也可以推送一个闭包去队列，这个方法非常的方便及快速的来处理需要使用队列的简单的任务：

	Queue::push(function($job) use ($id)
	{
		Account::delete($id);

		$job->delete();
	});

> **注记:** 要让一个组件变量可以在队列闭包中可以使用我们会通过 `use` 命令，试着传送主键及重复使用的相关模组在您的队列工作中，这可以避免其他的序列化行为。

当使用 Iron.io [push queues](#push-queues) 时,您应该在队列闭包中采取一些其他的预防措施，我们应该在执行工作收到队列数据时检查token是否真来自 Iron.io，举例来说您推送一个队列工作到 `https://yourapp.com/queue/receive?token=SecretToken`，接下来在您的工作收到队列的请求时，您就可以检查token的值是否正确。

<a name="running-the-queue-listener"></a>
## 执行一个队列监听

Laravel 内含一个 Artisan 命令，它将推送到队列的工作拉来下执行，您可以使用 `queue:listen` 命令，来执行这件常驻任务：

#### 开始队列监听

	php artisan queue:listen

您也可以指定特定队列连接让监听器使用：

	php artisan queue:listen connection

注意当这个任务开始时，这将会一直持续执行到他被手动停止，您也可以使用一个处理监控如 [Supervisor](http://supervisord.org/) 来确保这个队列监听不会停止执行。

您也可以在 `listen` 命令中使用逗号分隔不同的队列连接来设定队列的重要性：

	php artisan queue:listen --queue=high,low

在这个范列中 `high-connection` 将总是会优先处理队列的工作，相对于 `low-connection`

#### 指定工作超时参数

您也可以设定给每个工作允许执行的秒数：

	php artisan queue:listen --timeout=60

#### 指定队列休息时间

此外，您也可以指定让监听器在拉取新工作时要等待几秒：

	php artisan queue:listen --sleep=5

注意队列只会在没有队列内容时停止工作，假如队列中仍有许多可执行的工作，队列监听将持续不中断的处理

#### 处理队列上的一个工作

当您只想处理队列上的一个工作您可以使用 `queue:work` 命令：

	php artisan queue:work

<a name="daemon-queue-worker"></a>
## 常驻队列处理器

`queue:work` 也包含了一个 `--daemon` 选项能强迫队列处理器可以持续处理工作，即使框架重新启动了也不会停止。这种方式比起 `queue:listen` 来说，可以更有效的减少CPU的使用量，不过代价是要增加了您布署时的复杂性。

当开始一个队列处理器处于常驻模式，使用 `--daemon` 标示：

	php artisan queue:work connection --daemon

	php artisan queue:work connection --daemon --sleep=3

	php artisan queue:work connection --daemon --sleep=3 --tries=3

如您所见 `queue:work` 命令支持 `queue:listen` 大多相同的选项参数，您也可使用 `php artisan help queue:work` 命令来观看全部可用的选项参数。

### 布署常驻队列处理器

最简单的布署一个应用程序使用常驻队列处理器的方式就是将应用程序在开始布署时使用维护模式，您可以使用 `php artisan down` 命令来完成这件事情，当这个应用程序在维护模式，Laravel 将不会运行队列上的任何新工作，但会持续的处理已存在的工作。

最简单的重启工作的方法是在你的部署脚本中加上如下一行：

	php artisan queue:restart

> **Note:** This command relies on the cache system to schedule the restart. By default, APCu does not work for CLI commands. If you are using APCu, add `apc.enable_cli=1` to your APCu configuration.

### Coding For Daemon Queue Workers

Daemon queue workers do not restart the framework before processing each job. Therefore, you should be careful to free any heavy resources before your job finishes. For example, if you are doing image manipulation with the GD library, you should free the memory with `imagedestroy` when you are done.

<a name="push-queues"></a>
## 推送队列

您可以利用 Laravel 4 强大的队列架构来进行推送队列工作，不需要执行任何的常驻事务或后台监听，目前只支持 [Iron.io](http://iron.io) 驱动，在您开始前建立一个 Iron.io 帐号及新增您的 Iron 凭证到 `app/config/queue.php` 配置文件。

#### 注册一个推送队列订阅

接下来，您可以使用 `queue:subscribe` 命令注册一个URL来接收新的队列推送工作：

	php artisan queue:subscribe queue_name http://foo.com/queue/receive

现在当您登入您的Iron仪表板，您将会看到您新的推送队列，以及订阅的 URL，您可以订阅许多的URLs给您希望的队列，接下来建立一个 route 给您的 `queue/receive` 及从 `Queue::marshal` 方法回传回应：

	Route::post('queue/receive', function()
	{
		return Queue::marshal();
	});

`marshal` 方法会将工作处理到正确的类，而发送工作到队列中只要使用一样的 `Queue::push` 方法。

<a name="failed-jobs"></a>
## 已失败的工作

事情往往不会如您预期的一样，有时后您推送工作到队列会失败，别担心，Laravel 包含一个简单的方法去指定一个工作最多可以被执行几次，在工作被执行到一定的次数时，他将会新增至 `failed_jobs` 数据库表里，然后失败工作的数据库表名称可以在 `app/config/queue.php` 里进行设定：

要新增一个migration建立 `failed_jobs` 数据库表，您可以使用 `queue:failed-table` 命令：

	php artisan queue:failed-table

您可以指定一个最大值来限制一个工作应该最多被执行几次通过 `--tries` 这个选项参数在您执行 `queue:listen` 的时后：

	php artisan queue:listen connection-name --tries=3

假如您会想注册一个事件，这个事件会将会在队列失败时被调用，您可以使用 `Queue::failing` 方法，这个事件是一个很好的机会让您可以通知您的团队通过 e-mail 或 [HipChat](https://www.hipchat.com)。

	Queue::failing(function($connection, $job, $data)
	{
		//
	});

要看到所有的失败工作，您可以使用 `queue:failed` 命令：

	php artisan queue:failed

`queue:failed` 命令将会列出工作的 ID、连接、队列名称及失败的时间，工作的 ID 也可以重新执行一个已经失败的工作，例如一个已经失败的工作他的 ID 是 5，我们可以使用下面的命令：

	php artisan queue:retry 5

假如您会想删除一个已失败的工作，您可以使用 `queue:forget` 命令：

	php artisan queue:forget 5

要删除全部失败的工作您可以使用 `queue:flush` 命令：

	php artisan queue:flush
