# 路由

- [基本路由](#basic-routing)
- [路由参数](#route-parameters)
- [路由筛选](#route-filters)
- [路由命名](#named-routes)
- [路由群组](#route-groups)
- [子域名路由](#sub-domain-routing)
- [前缀路由](#route-prefixing)
- [路由模型绑定](#route-model-binding)
- [404 错误](#throwing-404-errors)
- [控制器路由](#routing-to-controllers)

<a name="basic-routing"></a>
## 基本路由

应用程序大多数的路由都会被定义在 `app/routes.php` 中。最简单的一个路由是由一个 URI 和闭包回调(Closure callback)构成的。

#### 基本 GET 路由

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### 基本 POST 路由

	Route::post('foo/bar', function()
	{
		return 'Hello World';
	});

#### 在一个路由中注册多种请求

	Route::match(array('GET', 'POST'), '/', function()
	{
		return 'Hello World';
	});

#### 在一个路由中回应所有 HTTP 请求

	Route::any('foo', function()
	{
		return 'Hello World';
	});

#### 强制路由使用 HTTPS

	Route::get('foo', array('https', function()
	{
		return 'Must be over HTTPS';
	}));

通常情况下，您需要生成 URLs 到你的路由中，您可以使用 `URL::to` 方法来实现：

	$url = URL::to('foo');

<a name="route-parameters"></a>
## 路由参数

	Route::get('user/{id}', function($id)
	{
		return 'User '.$id;
	});

#### 可选路由参数

	Route::get('user/{name?}', function($name = null)
	{
		return $name;
	});

#### 带默认值的可选路由参数

	Route::get('user/{name?}', function($name = 'John')
	{
		return $name;
	});

#### 正规表达式路由

	Route::get('user/{name}', function($name)
	{
		//
	})
	->where('name', '[A-Za-z]+');

	Route::get('user/{id}', function($id)
	{
		//
	})
	->where('id', '[0-9]+');

#### 传递数组并使用 Where 筛选

如果需要，您可以传递限制条件的数组：

	Route::get('user/{id}/{name}', function($id, $name)
	{
		//
	})
	->where(array('id' => '[0-9]+', 'name' => '[a-z]+'))

#### 定义全局正则表达式 Pattern

如果您有常用的正则表达式 Pattern，您可以使用 `pattern` 方法：

	Route::pattern('id', '[0-9]+');

	Route::get('user/{id}', function($id)
	{
		// Only called if {id} is numeric.
	});

#### 获取路由参数值

如果您要在路由之外获取路由参数值，您可以使用 `Route::input` 方法：

	Route::filter('foo', function()
	{
		if (Route::input('id') == 1)
		{
			//
		}
	});

<a name="route-filters"></a>
## 路由筛选器

路由筛选器能便捷的给指定的路由提供访问限制，这对于您的站点需要认证的情况下非常有用。在 Laravel 框架中包含了数个筛选器，如 `auth`, `auth.basic`, `guest` 和 `csrf` 筛选器。他们都放在 `app/filters.php` 中。

> **注意：** 路由筛选器在应用环境为 `testing` 下是无效的。

#### 定义一个路由筛选器

	Route::filter('old', function()
	{
		if (Input::get('age') < 200)
		{
			return Redirect::to('home');
		}
	});

如果筛选器有了回应，这个回应将会直接被视为该请求的回应，且路由将不会继续被执行，任何路由的 `after` 筛选器都将直接被取消。

#### 对路由加上筛选器

	Route::get('user', array('before' => 'old', function()
	{
		return 'You are over 200 years old!';
	}));

#### 对控制器动作加上筛选器

	Route::get('user', array('before' => 'old', 'uses' => 'UserController@showProfile'));

#### 对单一路由加上多个筛选器

	Route::get('user', array('before' => 'auth|old', function()
	{
		return 'You are authenticated and over 200 years old!';
	}));

#### 通过数组加上多个筛选器

	Route::get('user', array('before' => array('auth', 'old'), function()
	{
		return 'You are authenticated and over 200 years old!';
	}));

#### 指定筛选器参数

	Route::filter('age', function($route, $request, $value)
	{
		//
	});

	Route::get('user', array('before' => 'age:200', function()
	{
		return 'Hello World';
	}));

在筛选器接收到一个 `$response` 会被当成第三个参数传递进筛选器：

	Route::filter('log', function($route, $request, $response)
	{
		//
	});

#### 筛选器样式

您可以依据路由符合的 URI 来指定其筛选器：

	Route::filter('admin', function()
	{
		//
	});

	Route::when('admin/*', 'admin');

在上面的例子中，`admin` 筛选器将会套用在所有以 `admin/` 开头的路由中。`*` 通常用作通配符，他会匹配任何的字串组合。

您一样可以筛选指定的 HTTP 动作：

	Route::when('admin/*', 'admin', array('post'));

#### 筛选器类

进阶的筛选，您可以使用类来取代闭包。因为所有的类都可以通过 [IoC Container](/docs/4.2/ioc) 来实例化, 你可以利用依赖注入来获取更高的可测试性.

#### 注册基于类的筛选器

	Route::filter('foo', 'FooFilter');

默认下，`FooFilter` 类的 `filter` 方法将会被调用：

	class FooFilter {

		public function filter()
		{
			// Filter logic...
		}

	}

如果您不希望使用 `filter` 方法，只要指定其他方法即可：

	Route::filter('foo', 'FooFilter@foo');

<a name="named-routes"></a>
## 路由命名

路由命名在产生重定向跳转与 URLs 至路由时更为方便。您可以指定一个名称给指定的路由：

	Route::get('user/profile', array('as' => 'profile', function()
	{
		//
	}));

您一样可以为控制器动作指定一个路由名称：

	Route::get('user/profile', array('as' => 'profile', 'uses' => 'UserController@showProfile'));

现在您可以在产生 URLs 或重定向跳转时使用该路由名称：

	$url = URL::route('profile');

	$redirect = Redirect::route('profile');

您一样可以通过 `currentRouteName` 方法来取得正在执行中的路由名称：

	$name = Route::currentRouteName();

<a name="route-groups"></a>
## 路由群组

有时候您需要套用筛选器到一个群组的路由上。不需要为每个路由去套用筛选器，您只需使用路由群组:

	Route::group(array('before' => 'auth'), function()
	{
		Route::get('/', function()
		{
			// Has Auth Filter
		});

		Route::get('user/profile', function()
		{
			// Has Auth Filter
		});
	});

您一样可以在 `group` 数组中使用 `namespace` 参数，指定在这 group 中的控制器都有一个共同的命名空间：

	Route::group(array('namespace' => 'Admin'), function()
	{
		//
	});

<a name="sub-domain-routing"></a>
## 子域名路由

Laravel 路由一样可以处理通配的子域名，并且从域名中传递您的通配符参数：

#### 注册子域名路由

	Route::group(array('domain' => '{account}.myapp.com'), function()
	{

		Route::get('user/{id}', function($account, $id)
		{
			//
		});

	});

<a name="route-prefixing"></a>
## 前缀路由

群组路由可以通过群组的描述数组中使用 `prefix` 选项，将群组内的路由加上前缀：

	Route::group(array('prefix' => 'admin'), function()
	{

		Route::get('user', function()
		{
			//
		});

	});

<a name="route-model-binding"></a>
## 路由模型绑定

模型绑定提供一个方便的方式将模型实体注入到您的路由中。例如，要注入一个用户 ID 您可以注入符合给定 ID 的整个用户模型实体。首先，使用 `Route::model` 方法可以指定作为参数的模型：

#### 绑定参数至模型

	Route::model('user', 'User');

接着，定义一个路由并包括一个 `{user}` 参数：

	Route::get('profile/{user}', function(User $user)
	{
		//
	});

既然我们绑定了 `{user}` 参数到 `User` 模型，则 `User` 实体就会被注入到路由内。因此，假定有一个请求送至 `profile/1` 则会注入一个 ID 为 1 的 `User` 实体。

> **注意：** 假如在数据库内没有任何一个模型实体符合，则会抛出 404 错误。

假如您想指定自定的「找不到」行为，您可以在 `model` 方法里的第三个参数指定一个 Closure：

	Route::model('user', 'User', function()
	{
		throw new NotFoundHttpException;
	});

在某些情况下，您可能会希望可以使用您自定义的路由绑定方式。这时您可以使用 `Route::bind` 方法来达成：

	Route::bind('user', function($value, $route)
	{
		return User::where('name', $value)->first();
	});

<a name="throwing-404-errors"></a>
## 404 错误

有两种方式可以在路由内手动触发 404 错误。第一种是调用 `App::abort` 方法：

	App::abort(404);

第二种，您可以抛出一个 `Symfony\Component\HttpKernel\Exception\NotFoundHttpException` 实体。

有关如何处理 404 异常状况和自定义回应的详细信息，可以参考 [错误](/docs/4.2/errors#handling-404-errors) 章节内的说明。

<a name="routing-to-controllers"></a>
## 控制器路由

Laravel 允许您不止可以路由至 Closures，也可以路由至控制器类，甚至可以路由至 [资源控制器](/docs/4.2/controllers#resource-controllers)。

更多详细信息可参考 [控制器](/docs/4.2/controllers) 一节内的说明。
