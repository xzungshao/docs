# 扩展框架

- [介绍](#introduction)
- [管理者和工厂](#managers-and-factories)
- [在哪里扩展](#where-to-extend)
- [缓存](#cache)
- [Session](#session)
- [认证](#authentication)
- [基于 IoC 的扩展](#ioc-based-extension)
- [扩展请求](#request-extension)

<a name="introduction"></a>
## 介绍

Laravel 提供许多可扩展的地方让您自定义框架核心组件的行为，或甚至完全地取代它们。 例如，`HasherInterface` 接口定义了哈希工具，您可以基于应用程序的需求来实现它。 您也可以扩展 `Request` 对象，让您加入自己的便利 "辅助" 方法。 甚至您可以加入全新的认证、缓存和 session 驱动！

Laravel 组件一般以两种方式来扩展： 在 IoC 容器里绑定新的实现，或用 "工厂" 设计模式实现的 `Manager` 类来注册扩展。 在这个章节我们将会探索多种扩展框架的方法和查看必要的代码。

> **备注:** 记住， Laravel 组件通常用两种方法的其中之一来扩展： IoC 绑定和 `Manager` 类。 管理者类作为 "工厂" 设计模式的实现，并负责实体化基于驱动的工具，例如：缓存和 session。

<a name="managers-and-factories"></a>
## 管理者和工厂

Laravel 有几个 `Manager` 类用来管理建立基于驱动的组件。 这些类包括缓存、session、认证和队列组件。 管理者类负责基于应用程序的设定建立一个特定的驱动实现。 例如， `CacheManager` 类可以建立 APC、 Memcached 、文件和各种其他的缓存驱动实现.

这些管理者都拥有 `extend` 方法，它可以简单地用来注入新的驱动解析功能到管理者。 我们将会在下面随着如何注入自定义驱动支持给它们的例子，涵盖这些管理者的内容。

> **备注:** 建议花点时间来探索 Laravel 附带的各种 `Manager` 类，例如： `CacheManager` 和 `SessionManager`。 看过这些类将会让您更彻底了解 Laravel 表面下是如何运作。 所有的管理者类继承 `Illuminate\Support\Manager` 基底类， 它提供一些有用、常见的功能给每一个管理者。

<a name="where-to-extend"></a>
## 在哪里扩展

这份文件涵盖如何扩展各种 Laravel 的组件，但是您可能想知道要在哪里放置您的扩展代码。 就像其他大部分的启动代码，您可以自由的在您的 `start` 文件放置一些扩展，缓存和认证扩展是这个方法的好例子。 其他扩展，像 `Session`，必须放置到服务提供者的 `register` 方法中，因为他们在请求生命周期的非常早期就被需要。

<a name="cache"></a>
## 缓存

为了扩展 Laravel 缓存工具，我们将会对 `CacheManager` 使用 `extend` 方法，它被用来绑定一个自定义驱动解析器到管理者，并且是全部的管理者类通用的。 例如，注册一个新的缓存驱动命名为 "mongo"，我们将执行以下操作：

	Cache::extend('mongo', function($app)
	{
		// Return Illuminate\Cache\Repository instance...
	});

传递到 `extend` 方法的第一个参数是驱动的名称。 这将会对应到您的 `app/config/cache.php` 配置文件里的 `driver` 选项。 第二个参数是个应该回传 `Illuminate\Cache\Repository` 实体的闭包。 `$app` 实体将被传递到闭包，它是 `Illuminate\Foundation\Application` 和 IoC 容器的实体。

要建立我们的自定义缓存驱动，首先需要实现 `Illuminate\Cache\StoreInterface` 接口。 所以，我们的 MongoDB 缓存实现将会看起来像这样：

	class MongoStore implements Illuminate\Cache\StoreInterface {

		public function get($key) {}
		public function put($key, $value, $minutes) {}
		public function increment($key, $value = 1) {}
		public function decrement($key, $value = 1) {}
		public function forever($key, $value) {}
		public function forget($key) {}
		public function flush() {}

	}

我们只需要使用 MongoDB 连接来实现这些方法。 当我们的实现完成，我们可以完成我们的自定义驱动注册：

	use Illuminate\Cache\Repository;

	Cache::extend('mongo', function($app)
	{
		return new Repository(new MongoStore);
	});

就像您可以在上面的例子看到的，当在建立自定义缓存驱动的时候，您可以使用基本的 `Illuminate\Cache\Repository` 类。 通常不需要建立您自己的储存库类。

如果您在考虑要把您的自定义缓存驱动代码放在哪里，请考虑把它放上 Packagist！ 或者，您可以在您的应用程序的主要文件夹中建立一个 `Extensions` 命名空间。 例如，如果应用程序命名为 `Snappy`，您可以在 `app/Snappy/Extensions/MongoStore.php` 放置缓存扩展。 然而，必须牢记在心 Laravel 没有严格的应用程序架构，您可以依照喜好自由的组织您的应用程序。

> **备注:** 如果您曾经考虑要在哪放置一段代码，请总是考虑服务提供者。 就像我们曾经讨论过的，用服务提供者来组织框架扩展是个组织您的代码的好方法.

<a name="session"></a>
## Session

以自定义 session 驱动来扩展 Laravel 跟扩展缓存系统一样简单。 再一次的，我们将会使用 `extend` 方法来注册我们的自定义代码：

	Session::extend('mongo', function($app)
	{
		// Return implementation of SessionHandlerInterface
	});

### 在哪里扩展 Session

Session 需要用与其他扩展如 Cache 和 Auth 不同地方式扩展。 因为 sessions 在请求生命周期的非常早期就被启用，注册扩展在 `start` 文件将会太晚。 作为替代，将会需要[服务提供者](/docs/4.2/ioc#service-providers) 。 您应该放置您的 session 扩展代码在您的服务提供者的 `register` 方法，并且提供者应该被放置在 `providers` 设定数组里、默认的 `Illuminate\Session\SessionServiceProvider` **下面**。

### 实现 Session 扩展

要注意我们的自定义缓存驱动应该实现 `SessionHandlerInterface`。 这个接口在 PHP 5.4+ 核心被引入。 如果您使用 PHP 5.3，Laravel 将会为您定义这个接口，所以您可以向下兼容。 这个接口只包含少数我们需要实现的简单方法。 一个空的 MongoDB 实现将会看起来像这样：

	class MongoHandler implements SessionHandlerInterface {

		public function open($savePath, $sessionName) {}
		public function close() {}
		public function read($sessionId) {}
		public function write($sessionId, $data) {}
		public function destroy($sessionId) {}
		public function gc($lifetime) {}

	}

因为这些方法不像缓存 `StoreInterface` 一样容易理解，让我们快速地看过这些方法做些什么：

- `open` 方法通常会被用在基于文件的 session 储存系统。 因为 Laravel 附带一个 `file` session 驱动，您几乎不需要在这个方法放任何东西。 您可以让它留空。 PHP 要求我们去实现这个方法，事实上明显的是个差劲的接口设计 (我们将会晚点讨论它)。
- `close` 方法，就像 `open` 方法，通常也可以忽略。 对大部份的驱动来说，并不需要它。
- `read` 方法应该回传与给定 `$sessionId` 关联的 session 数据的字串形态。 当您的驱动取回或储存 session 数据时不需要做任何序列化或进行其他编码，因为 Laravel 将会为您进行序列化。
- `write` 方法应该写入给定 `$data` 字串与 `$sessionId` 的关联到一些永久存储系统，例如 MongoDB、 Dynamo、等等。
- `destroy` 方法应该从永久存储移除与 `$sessionId` 关联的数据。
- `gc` 方法应该销毁所有比给定 `$lifetime` UNIX 时间戳记还旧的 session 数据。 对于会自己到期的系统如 Memcached 和 Redis，这个方法可以留空。

当 `SessionHandlerInterface` 被实现完成，我们已经准备好用 Session 管理者注册它：

	Session::extend('mongo', function($app)
	{
		return new MongoHandler;
	});

当 session 驱动已经被注册，我们可以在我们的 `app/config/session.php` 配置文件使用 `mongo` 驱动 。

> **备注:** 记住，如果您写了个自定义 session 处理器，请在 Packagist 分享它！

<a name="authentication"></a>
## 认证

认证可以通过与缓存和 session 工具相同的方法来扩展。 再一次的，我们将会使用我们已经熟悉的 `extend` 方法：

	Auth::extend('riak', function($app)
	{
		// Return implementation of Illuminate\Auth\UserProviderInterface
	});

`UserProviderInterface` 实现只负责从持久化存储中抓取 `UserInterface` 实现，例如： MySQL、 Riak，等等。 这两个接口让 Laravel 认证机制无论用户数据如何储存或用什么种类的类来代表它都能继续运作。

让我们来看一下 `UserProviderInterface`：

	interface UserProviderInterface {

		public function retrieveById($identifier);
		public function retrieveByToken($identifier, $token);
		public function updateRememberToken(UserInterface $user, $token);
		public function retrieveByCredentials(array $credentials);
		public function validateCredentials(UserInterface $user, array $credentials);

	}

`retrieveById` 函数通常接收一个代表用户的数字键，例如： MySQL 数据库的自动递增 ID。 符合 ID 的 `UserInterface` 实现应该被取回并被方法回传。

`retrieveByToken` 函数通过用户唯一的 `$identifier` 和储存在 `remember_token` 字段的 "记住我" `$token` 取得用户。 跟前面的方法一样，应该回传 `UserInterface` 实现。

`updateRememberToken` 方法用新的 `$token` 更新 `$user` 的 `remember_token` 字段。 新 token 的值 可以是在 "记住我" 成功地登入时生成的一个新的 token，或当用户登出时变为 null.

`retrieveByCredentials` 方法接收当尝试登入应用程序时，传递到 `Auth::attempt` 方法的凭证数组。 这个方法应该接着 "查询" 背后的永久存储，看用户是否符合这些凭证。 这个方法通常会对 `$credentials['username']` 用 "where" 条件查询。 **这个方法不应该尝试做任何密码验证或认证。**

`validateCredentials` 方法应该通过比较给定 `$user` 与 `$credentials` 来验证用户。 举例来说，这个方法可以比较 `$user->getAuthPassword()` 字串跟 `Hash::make` 后的 `$credentials['password']`。

现在我们已经看过 `UserProviderInterface` 的每个方法，接着我们来看一下 `UserInterface`。 记住，提供者应该从 `retrieveById` 和 `retrieveByCredentials` 方法回传这个接口的实现：

	interface UserInterface {

		public function getAuthIdentifier();
		public function getAuthPassword();

	}

这个接口很简单。 `getAuthIdentifier` 方法应该回传用户的 "主键"。 在 MySQL 后台，同样，这将会是个自动递增的主键。 `getAuthPassword` 应该回传用户哈希过的密码。 这个接口让认证系统可以与任何用户类一起运作，无论您使用什么 ORM 或储存抽象层。 默认， Laravel 包含一个实现这个接口的 `User` 类在 `app/models` 文件夹里，所以您可以参考这个类当作实现的例子。

最后，当我们已经实现了 `UserProviderInterface`，我们准备好用 `Auth` facade 来注册我们的扩展：

	Auth::extend('riak', function($app)
	{
		return new RiakUserProvider($app['riak.connection']);
	});

在您用 `extend` 方法注册驱动之后，在您的 `app/config/auth.php` 配置文件切换到新驱动。
<a name="ioc-based-extension"></a>
## 基于 IoC 的扩展

几乎每个 Laravel 框架引入的服务提供者会绑定对象到 IoC 容器中。 您可以在 `app/config/app.php` 配置文件中找到应用程序的服务提供者清单。 当您有时间的时候，您应该浏览一下这里面每一个提供者的源代码。 通过这样做，您将会对每一个提供者为框架增加了什么功能有更多的了解，以及各种服务用什么键来绑定到 IoC 容器。

举个例子， `HashServiceProvider` 绑定一个 `hash` 键到 IoC 容器，它将解析成 `Illuminate\Hashing\BcryptHasher` 实例。 您可以轻松地通过在您的应用程序中重写这个 IoC 绑定，扩展并重写这个类。 例如：

	class SnappyHashProvider extends Illuminate\Hashing\HashServiceProvider {

		public function boot()
		{
			App::bindShared('hash', function()
			{
				return new Snappy\Hashing\ScryptHasher;
			});

			parent::boot();
		}

	}

要注意的是这个类扩展 `HashServiceProvider`，不是默认的 `ServiceProvider` 基底类。 当您扩展了服务提供者，在您的 `app/config/app.php` 配置文件把 `HashServiceProvider` 换成您扩展的提供者名称。

这是扩展任何被绑定在容器的核心类的普遍方法。 实际上，每个被以这种方式绑定在容器的核心类都可以被重写。 再次强调，看过被框架引入的服务提供者将会使您熟悉每个类被绑在容器的哪里，以及它们是用什么键绑定的。 这也是可以了解更多关于 Laravel 是如何结合它们的好方法。

<a name="request-extension"></a>
## 扩展请求

因为它是框架非常基础的部件并且在请求周期的非常早期就被实例化，扩展 `Request` 类跟前面的例子有一点不同。

首先，继承 laravel 的 `Request` 基类：

	<?php namespace QuickBill\Extensions;

	class Request extends \Illuminate\Http\Request {

		// Custom, helpful methods here...

	}

当您扩展了类，打开 `bootstrap/start.php` 文件。 这个文件是当应用程序受到请求后非常早被引入的文件之一。 需要注意的是第一个被执行的动作是建立 Laravel `$app` 实例：

	$app = new \Illuminate\Foundation\Application;

当一个新的应用程序实例被建立，它将会建立一个新的 `Illuminate\Http\Request` 实例并用 `request` 键把它绑定到 IoC 容器。 所以，我们需要一个方法去指定一个应该被用作 "默认" 请求类型的自定义类，对吧？ 并且值得庆幸的是，应用程序实例的 `requestClass` 方法就做了这件事！ 所以，我们可以在 `bootstrap/start.php` 文件的最上面加这行：

	use Illuminate\Foundation\Application;

	Application::requestClass('QuickBill\Extensions\Request');

每当您指定了自定义请求类， Laravel 将会在任何建立 `Request` 实体的时候使用这个类，便利地让您总是有一个可用的自定义请求类实例，甚至在单元测试也有！
