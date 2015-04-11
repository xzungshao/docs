# 错误与日志

- [配置文件](#configuration)
- [错误处理](#handling-errors)
- [HTTP异常](#http-exceptions)
- [处理404错误](#handling-404-errors)
- [日志](#logging)

<a name="configuration"></a>
## 配置文件

日志处理程序注册在[启动文件](/docs/4.2/lifecycle#start-files) `app/start/global.php` 文件里面。日志都默认储存在一个单独的文件中，您可以依照实际需求自定义日志。因为 Laravel 使用了非常流行的 [Monolog](https://github.com/Seldaek/monolog) 日志库，您可以利用 Monolog 提供的多种方式管理你的日志。

例如，如果您想每天使用一个文件记录日志而不是使用单独的庞大文件，您可以照着下面的例子更改开始配置文件：

	$logFile = 'laravel.log';

	Log::useDailyFiles(storage_path().'/logs/'.$logFile);

### 错误显示

错误显示默认为开启，意味着当错误发生时，将有错误页面显示详细的调用追踪和错误信息。您可以关掉错误显示的选项，把`app/config/app.php`里的`debug`选项改成`false`。

> **注意：**强烈建议在生产环境中关掉错误显示。

<a name="handling-errors"></a>
## 错误处理

`app/start/global.php`里默认有一个处理所有异常的异常处理程序：

	App::error(function(Exception $exception)
	{
		Log::error($exception);
	});

这是最基本的异常处理程序，然而您可以依照需求设定更多异常处理程序。异常处理程序会依照异常的类型提示( type-hint )被调用。例如，您可以创造一个只处理`RuntimeException`的异常处理程序：

	App::error(function(RuntimeException $exception)
	{
		// Handle the exception...
	});

如果异常处理程序回传一个 response，response 会直接回传到浏览器，而其他异常处理程序将不会被调用：

	App::error(function(InvalidUserException $exception)
	{
		Log::error($exception);

		return 'Sorry! Something is wrong with this account!';
	});

为了监听 PHP fetal errors，您可以利用`App::fatal`方法：

	App::fatal(function($exception)
	{
		//
	});

如果您有很多异常处理程序，他们应该依照从最通用到最特定的顺序被定义。例如，一个对应处理类型为`Exception`的异常处理程序，应该被定义在一个对应处理自定义异常类型，如`Illuminate\Encryption\DecryptException`的异常处理程序之前。

### 何处定义异常处理程序

默认上没有注册异常处理程序的地方。Laravel 可以让您自由设定。选择之一是定义程序在 `start/global.php`中，一般来说，这是一个让您方便写入任何"bootstrapping" 代码的地方。如果文件变得很拥挤，可以建立一个 `app/errors.php`文件，并且在`start/global.php`中引入。第三个选择是建立 [service provider](/docs/4.2/ioc#service-providers) 以注册程序。再一次强调，这个问题并没有正确的答案，您直接选择一个让您觉得舒适的地方注册异常处理程序即可。

<a name="http-exceptions"></a>
## HTTP 异常处理

一些异常处理表示来自服务器的 HTTP 错误码，例如可能是「找不到页面」错误(404)，未授权错误(401)，或甚至是工程师导致的500错误。使用下列方法以回传这些回应：

	App::abort(404);

或是您可以选择提供一个回应：

	App::abort(403, 'Unauthorized action.');

您可以在请求回应的生命周期中任何时间点使用这个方法。

<a name="handling-404-errors"></a>
## 404错误处理

您可以注册一个错误处理程序处理所有"404 Not Found"错误，让您可以简单的回传自定义的404错误页面。

	App::missing(function($exception)
	{
		return Response::view('errors.missing', array(), 404);
	});

<a name="logging"></a>
## 日志

Laravel 提供一个建立在强大的 [Monolog](http://github.com/seldaek/monolog) 上的日志工具。Laravel 默认设定在应用程序里建立单一日志文件，这个文件储存在`app/storage/logs/laravel.log`。	您可以像下面这样写入信息：

	Log::info('This is some useful information.');

	Log::warning('Something could be going wrong.');

	Log::error('Something is really going wrong.');

日志工具提供了七种定义在 [RFC 5424](http://tools.ietf.org/html/rfc5424) 的级别：**debug**, **info**, **notice**, **warning**, **error**, **critical**, and **alert**

可以在传入上下文相关的数组到 log 的方法里：

	Log::info('Log message', array('context' => 'Other helpful information'));

Monolog 提供很多额外的方法可以记录。若有需要，您可以使用 Laravel 里使用的 Monolog 实例：

	$monolog = Log::getMonolog();

您可以注册事件捕捉所有传到日志的信息：

#### 注册日志监听程序

	Log::listen(function($level, $message, $context)
	{
		//
	});
