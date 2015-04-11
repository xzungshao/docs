# Facades

- [介绍](#introduction)
- [解释](#explanation)
- [实际用法](#practical-usage)
- [建立 Facades](#creating-facades)
- [模拟 Facades](#mocking-facades)
- [Facade 类参考](#facade-class-reference)

<a name="introduction"></a>
## 介绍

Facades 提供一个静态接口让类可以在应用程序的 [IoC 容器](/docs/4.2/ioc) 里运用。 Laravel 附带许多 facades，甚至您可能已经在使用它们即使您并不知道! Laravel 的 "facades" 在 IoC 容器里面作为的基底类的静态代理，提供有简洁、易表达优点的语法，同时维持比传统的静态方法更高的可测试性和弹性。

您偶尔或许会希望为您的应用程序和扩展包建立自己的 facades，所以让我们来探索这些类的概念、开发和用法。

> **备注:** 在深入 facades 之前，强烈建议您先熟悉 Laravel [IoC 容器](/docs/4.2/ioc)。

<a name="explanation"></a>
## 解释

在 Laravel 应用程序的环境中， facade 是个提供从容器获取对象的类。 使这个机制可以运作的原因在 `Facade` 类中。 Laravel 的 facades 和任何您建立的自定义 facades，将会继承基本的 `Facade` 类。

您的 facade 类只需要去实现一个方法： `getFacadeAccessor`。 `getFacadeAccessor` 方法的工作是定义要从容器解析什么。 基本的 `Facade` 类利用 `__callStatic()` 魔术方法去从您的 facade 调用到解析对象。

所以当您对 facade 调用，例如 `Cache::get`， Laravel 从 IoC 容器解析缓存管理类出来，并对类调用 `get` 方法。 用术语来说， Laravel Facades 是使用 Laravel IoC 容器作为服务定位器的便捷语法。

<a name="practical-usage"></a>
## 实际用法

在下面的例子，对 Laravel 缓存系统进行调用。 简单看过这代码，有人可能会以为静态方法 `get` 是被 `Cache` 类调用。

	$value = Cache::get('key');

然而，如果我们去看 `Illuminate\Support\Facades\Cache` 类， 您将会看到他没有静态方法 `get`:

	class Cache extends Facade {

		/**
		 * Get the registered name of the component.
		 *
		 * @return string
		 */
		protected static function getFacadeAccessor() { return 'cache'; }

	}

Cache 类继承基本的 `Facade` 类并定义方法 `getFacadeAccessor()`。 记住，这个方法的工作是回传 IoC 绑定的名称。

当用户参考 `Cache` facade 的任何静态方法， Laravel 会从 IoC 容器解析被绑定 `cache` ，并对该对象执行被请求的方法 (在这个例子， `get`)。

所以我们的 `Cache::get` 调用可以被重写成这样：

	$value = $app->make('cache')->get('key');

<a name="creating-facades"></a>
## 建立 Facades

为您的应用程序或扩展包建立 facade 是很简单的。 您只需要 3 个东西：

- 一个 IoC 绑定。
- 一个 facade 类。
- 一个 facade 别名设定。

让我们来看个例子。 这里我们有一个定义为 `PaymentGateway\Payment` 的类。

	namespace PaymentGateway;

	class Payment {

		public function process()
		{
			//
		}

	}

这个类可以存在在您的 `app/models` 文件夹，或者任何其他 Composer 知道如何自动载入的文件夹。

我们需要可以从 IoC 容器解析出这个类。 所以，让我们来加个绑定：

	App::bind('payment', function()
	{
		return new \PaymentGateway\Payment;
	});

注册这个绑定的好方式是建立新的 [服务提供者](/docs/4.2/ioc#service-providers) 命名为 `PaymentServiceProvider`，并把这个绑定加到 `register` 方法。 再来您可以从 `app/config/app.php` 文件设定让 Laravel 载入您的服务提供者。

接下来，我们可以建立我们自己的 facade 类：

	use Illuminate\Support\Facades\Facade;

	class Payment extends Facade {

		protected static function getFacadeAccessor() { return 'payment'; }

	}

最后，如果我们希望，我们可以在 `app/config/app.php` 配置文件为我们的 facade 加个别名到 `aliases` 数组。 现在我们可以对 `Payment` 类的实体调用 `process` 方法。

	Payment::process();

### 自动载入别名的附注

在 `aliases` 数组中的类在某些实体中不能使用，因为 [PHP 将不会尝试去自动载入未定义的类型暗示类](https://bugs.php.net/bug.php?id=39003)。 如果 `\ServiceWrapper\ApiTimeoutException` 命别名为 `ApiTimeoutException`， 在 `\ServiceWrapper` 命名空间外面的  `catch(ApiTimeoutException $e)` 将永远捕捉不到异常，即便有异常被抛出。类似的问题在有类型暗示的别名类模型一样会发生。 唯一的解决办法就是放弃别名并用 `use` 在每一个文件的最上面引入您希望暗示类型的类。

<a name="mocking-facades"></a>
## 模拟 Facades

单元测试是为什么现在 facades 采用这样的工作方式的重要面向。 事实上，可测试性甚至是 facades 存在的主要理由。 想要获得更多信息，请查看文件的 [mocking facades](/docs/4.2/testing#mocking-facades) 部分。

<a name="facade-class-reference"></a>
## Facade 类参考

您将会在下面找到每一个 facade 和它的基底类。 这是个可以从一个给定的 facade 根源快速地深入 API 文件的有用工具。 可应用的 [IoC 绑定](/docs/4.2/ioc) 关键字也包含在里面。

Facade  |  Class  |  IoC Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/4.2/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Console\Application](http://laravel.com/api/4.2/Illuminate/Console/Application.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/4.2/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel.com/api/4.2/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/4.2/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/4.2/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/4.2/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/4.2/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/4.2/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/4.2/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/4.2/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/4.2/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/4.2/Illuminate/Filesystem/Filesystem.html)  |  `files`
Form  |  [Illuminate\Html\FormBuilder](http://laravel.com/api/4.2/Illuminate/Html/FormBuilder.html)  |  `form`
Hash  |  [Illuminate\Hashing\HasherInterface](http://laravel.com/api/4.2/Illuminate/Hashing/HasherInterface.html)  |  `hash`
HTML  |  [Illuminate\Html\HtmlBuilder](http://laravel.com/api/4.2/Illuminate/Html/HtmlBuilder.html)  |  `html`
Input  |  [Illuminate\Http\Request](http://laravel.com/api/4.2/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/4.2/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/4.2/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/4.2/Illuminate/Mail/Mailer.html)  |  `mailer`
Paginator  |  [Illuminate\Pagination\Factory](http://laravel.com/api/4.2/Illuminate/Pagination/Factory.html)  |  `paginator`
Paginator (Instance)  |  [Illuminate\Pagination\Paginator](http://laravel.com/api/4.2/Illuminate/Pagination/Paginator.html)  |
Password  |  [Illuminate\Auth\Reminders\PasswordBroker](http://laravel.com/api/4.2/Illuminate/Auth/Reminders/PasswordBroker.html)  |  `auth.reminder`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/4.2/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel.com/api/4.2/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/4.2/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/4.2/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/4.2/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/4.2/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Support\Facades\Response](http://laravel.com/api/4.2/Illuminate/Support/Facades/Response.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/4.2/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/4.2/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/4.2/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/4.2/Illuminate/Session/Store.html)  |
SSH  |  [Illuminate\Remote\RemoteManager](http://laravel.com/api/4.2/Illuminate/Remote/RemoteManager.html)  |  `remote`
SSH (Instance)  |  [Illuminate\Remote\Connection](http://laravel.com/api/4.2/Illuminate/Remote/Connection.html)  |
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/4.2/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/4.2/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/4.2/Illuminate/Validation/Validator.html) |
View  |  [Illuminate\View\Factory](http://laravel.com/api/4.2/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/4.2/Illuminate/View/View.html)  |
