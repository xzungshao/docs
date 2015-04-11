# IoC 容器

- [介绍](#introduction)
- [基本用法](#basic-usage)
- [何处注册绑定](#where-to-register)
- [自动解析](#automatic-resolution)
- [应用](#practical-usage)
- [服务提供者](#service-providers)
- [容器事件](#container-events)

<a name="introduction"></a>
## 介绍

Laravel 的控制反转 ( IoC, inversion of control ) 容器是管理类依赖的强力工具。 依赖注入 ( Dependency injection ) 是一种移除 hard-coded 类依赖的方式。相较之下，在执行的时候才注入依赖，
可以拥有更好的弹性，在替换依赖实体时相当容易。

理解 Laravel IoC 容器对于建立大型的应用程序，以及改进 Laravel 核心是很重要的。

<a name="basic-usage"></a>
## 基本用法

#### 绑定类型到容器

IoC 容器有两种解析依赖的方式：经由闭包函数或自动解析。我们将先探索闭包函数的用法。首先，绑定一个「类型」到容器：

	App::bind('foo', function($app)
	{
		return new FooBar;
	});

#### 从容器中解析类型

	$value = App::make('foo');

当 `App::make` 方法被调用，绑定的闭包函数会被调用并返回结果。

#### 绑定「共享」的类型到容器 

有时候，您可能希望绑定到容器的类型只会被解析一次，之后的调用都返回相同的实例：

	App::singleton('foo', function()
	{
		return new FooBar;
	});

#### 绑定已存在的实例到容器

您也可以使用 `instance` 方法，绑定一个已经存在的实例到容器：

	$foo = new Foo;

	App::instance('foo', $foo);

<a name="where-to-register"></a>
## 何处注册绑定

IoC 绑定跟「注册事件处理」或是「注册路由」一样，通常称为「引导代码」。换句话说，IoC 绑定后等待请求，在路由或控制器调用时才实际执行。像其他的「引导代码」一样，`start` 文件总是注册 IoC 绑定的一个选择。或者，您可以建立一个 `app/ioc.php` 文件（文件名不重要），并且从 `start` 文件引入。

如果您的应用程序有很多 IoC 绑定，或是想要分门别类，在不同文件组织绑定，您可以注册绑定在[
服务提供者](#service-providers)。

<a name="automatic-resolution"></a>
## 自动解析

在很多情境下，IoC 容器不需要额外设定就有能力自动解析类。例如：

	class FooBar {

		public function __construct(Baz $baz)
		{
			$this->baz = $baz;
		}

	}

	$fooBar = App::make('FooBar');

虽然我们没有注册 FooBar 类绑定到容器，它还是可以解析类，甚至自动注入 `Baz` ！

如果容器里没有找到对应的类型绑定，容器会利用 PHP 的 Reflection 检查类，并且解读传入构造函数的类型提示。利用这些信息，让容器可以自动建立类实例。

#### 绑定实例的接口

然而，有些时候，一个类可能需要依赖接口，而不是一个「具体的类型」。这些情况下，`App::bind` 方法用来通知容器要注入哪个接口实例：

	App::bind('UserRepositoryInterface', 'DbUserRepository');

考虑以下的控制器：

	class UserController extends BaseController {

		public function __construct(UserRepositoryInterface $users)
		{
			$this->users = $users;
		}

	}

既然我们已经绑定 `UserRepositoryInterface` 到一个具体类型，`DbUserRepository` 会在控制器建立时自动注入。

<a name="practical-usage"></a>
## 应用

在 Laravel 里，很多时候使用 IoC 容器，可以增加应用程序的弹性与可测试性。一个基本的例子是用在解析控制器。所有的控制器都会经由 IoC 容器解析，意味着您可以在控制器的构造函数注入类型提示依赖，之后依赖就会自动被注入。

#### 注入类型提示 ( Type-Hinting ) 依赖到控制器

	class OrderController extends BaseController {

		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		public function getIndex()
		{
			$all = $this->orders->all();

			return View::make('orders', compact('all'));
		}

	}

在这个例子里， `OrderRepository` 类会自动被注入到控制器。意味着在[单元测试](/docs/4.2/testing)时，可以绑定一个 "mock" 的 `OrderRepository` 到容器里，之后被注入到 控制器，让您不用在测试时一定要和数据库层互动。

#### 其他 Ioc 应用例子

[Filters](/docs/4.2/routing#route-filters)，[composers](/docs/4.2/responses#view-composers)，和[事件处理](/docs/4.2/events#using-classes-as-listeners)也可以使用 IoC 容器解析。只要在注册的时候设定要使用的类名称：

	Route::filter('foo', 'FooFilter');

	View::composer('foo', 'FooComposer');

	Event::listen('foo', 'FooHandler');

<a name="service-providers"></a>
## 服务提供者（ Service Provider ）

使用服务提供者是一个很好的方式，可以把相关的 IoC 注册放到到同一个地方。可以将服务提供者 想像成是一个在应用程序里启动组件的方式。您可以在里面注册自定义的会员认证，绑定应用程序的 储存库类到 IoC 容器，或甚至设定自定义的 Artisan 命令。

事实上，大部份的 Laravel 核心组件都有服务提供者，所有被注册的服务提供者都列在 `app/config/app.php` 配置文件的 `providers` 数组里。

#### 定义一个服务提供者

要建立一个服务提供者，只要继承 `Illuminate\Support\ServiceProvider` 类，然后在里面定义一个 `register` 方法：

	use Illuminate\Support\ServiceProvider;

	class FooServiceProvider extends ServiceProvider {

		public function register()
		{
			$this->app->bind('foo', function()
			{
				return new Foo;
			});
		}

	}

注意，在 `register` 方法里，经由 `$this->app` 使用 IoC 容器。当您建立了一个服务 而且准备要注册到您的应用程序里时，只要把它加到您的 `app` 配置文件的 `providers` 数组里即可。

#### 在执行期间注册服务提供者

您也可以使用 `App::register` 在执行期间注册服务提供者 ：

	App::register('FooServiceProvider');

<a name="container-events"></a>
## 容器事件

#### 注册解析事件的监听

每当容器解析一个对象时就会触发事件，您可以使用 `resolving` 方法监听这个事件：

	App::resolvingAny(function($object)
	{
		//
	});

	App::resolving('foo', function($foo)
	{
		//
	});

注意，被解析的对象会被传到闭包函数。
