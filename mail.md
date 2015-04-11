# 邮件发送

- [设定](#configuration)
- [基本用法](#basic-usage)
- [内嵌附件](#embedding-inline-attachments)
- [邮件队列](#queueing-mail)
- [邮件与本地开发](#mail-and-local-development)

<a name="configuration"></a>
## 设定

Laravel 利用一个热门的函数库 [SwiftMailer](http://swiftmailer.org) 来建立一个干净简单的 API。邮件配置文件为 `app/config/mail.php`，里面有些选项可以让您更改您的 SMTP 连接地址、凭证以及全局的寄件者 `from`。您可以使用任何的 SMTP 服务器。如果您想要使用 PHP 内建的 `mail` 函数来发送邮件，您可以将配置文件中的 `driver` 改为 `mail` 即可。`sendmail` 的驱动一样可以使用。


### 第三方邮件服务 API 驱动

Laravel 也包含了 Mailgun 和 Mandrill 服务的 HTTP API 的驱动方式。这些 API 比使用 SMTP 服务器更简单快速。这些驱动都需要在您的应用程序里预先安装 Guzzle 4 HTTP 函数库。您可以通过在您的 `composer.json` 文件中增加一行如下，来将 Guzzle 4 安装到您的项目中：

	"guzzlehttp/guzzle": "~4.0"

#### Mailgun 驱动

使用 Mailgun 驱动前，先在 `app/config/mail.php` 配置文件中将 `driver` 设定为 `mailgun`。再来，如果您的项目中没有 `app/config/services.php` 请先建立他。并确定它包含了如下的内容：

	'mailgun' => array(
		'domain' => 'your-mailgun-domain',
		'secret' => 'your-mailgun-key',
	),


### Log 驱动

如果您的 `app/config/mail.php` 配置文件的 `driver` 被设定为 `log` 时，所有的邮件将会被写进日志中，且并不会真的被寄出。这主要用在快速本地除错和内容验证上。

<a name="basic-usage"></a>
## 基本用法

 `Mail::send` 方法是用来发送电子邮件：

	Mail::send('emails.welcome', array('key' => 'value'), function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

`send` 方法的第一个参数为邮件内容的视图名称，第二个参数 `$data` 为传递至视图的变量，而第三个参数为一个闭包让您可以指定更多发送电子邮件的信息选项。

> **注意:** 变量默认被传递进邮件的视图中，且允许内嵌键值数组。所以最好避免传递名为 `message` 的变量到您的视图中造成冲突。

除了使用 HTML 视图外，您也可以指定一个纯文字的视图：

	Mail::send(array('html.view', 'text.view'), $data, $callback);


或者您也可以指定视图类为 `html` 或 `text`：

	Mail::send(array('text' => 'view'), $data, $callback);


您也可以为电子邮件信息指定其他选项如附加文件或是任何的副本收件者：

	Mail::send('emails.welcome', $data, function($message)
	{
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');

		$message->attach($pathToFile);
	});


当附加文件到一个电子邮件中时，您也可以指定一个 MIME type 及/或 一个显示名称：

	$message->attach($pathToFile, array('as' => $display, 'mime' => $mime));

> **注意:** 在 `Mail::send` 的闭包中使用 SwiftMailer 的 message 扩充实例，允许您可以调用任何类中的方法来建立您的电子邮件。

<a name="embedding-inline-attachments"></a>
## 内嵌附件

内嵌图片到邮件中通常是一件很麻烦的事，然而Laravel 提供一个便利的方式让您内嵌图片到您的电子邮件当中且接收相对应的 CID。

#### 内嵌图片到电子邮件的视图中

	<body>
		Here is an image:

		<img src="<?php echo $message->embed($pathToFile); ?>">
	</body>

#### 内嵌数据到电子邮件的视图中

	<body>
		Here is an image from raw data:

		<img src="<?php echo $message->embedData($data, $name); ?>">
	</body>

注意 `$message` 这个变量一定会被 `Mail` 类传递到电子邮件的视图当中。

<a name="queueing-mail"></a>
## 邮件队列

#### 加入一个邮件到队列中

发送电子邮件会大幅的延长您应用程序的反应时间，许多开发者选择让电子邮件放进队列中，并在后台(非即时)发送。Laravel 使用内建的 [unified queue API](/docs/4.2/queues) 让您可以方便的将要发送的电子邮件加入到队列之中，只要使用 `Mail` 类的 `queue` 方法：

	Mail::queue('emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

您也可以通过使用 `later` 方法来指定发送电子邮件的延迟秒数：

	Mail::later(5, 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

假如您想要指定一个特别的队列或管道来发送邮件，您可以使用 `queueOn` 和 `laterOn` 方法：

	Mail::queueOn('queue-name', 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

<a name="mail-and-local-development"></a>
## 邮件与本地开发

在开发应用程序的寄信功能时，通常会希望在本地或是开发环境中关闭发送功能。您可以调用 `Mail::pretend` 方法，或是在 `app/config/mail.php` 配置文件的 `pretend` 为 `true` 可以达到。当在 `pretend` 模式开启下，邮件将会被写到您的应用程序日志中取代实际寄出信件。


#### 开启 "Pretend" 发送模式

	Mail::pretend();
