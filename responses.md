# 视图（ View ）与响应（ Response ）

- [基本响应](#basic-responses)
- [重定向跳转](#redirects)
- [视图](#views)
- [视图组件](#view-composers)
- [特殊响应](#special-responses)
- [响应宏](#response-macros)

<a name="basic-responses"></a>
## 基本响应

#### 从路由回传字串

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### 建立自定义响应

`Response` 实例继承了 `Symfony\Component\HttpFoundation\Response` 类，其提供了很多方法建立 HTTP 响应。

	$response = Response::make($contents, $statusCode);

	$response->header('Content-Type', $value);

	return $response;

如果想要使用 `Response` 类的方法，但最终回传视图给用户，您可以使用简便的 `Response::view` 方法：

	return Response::view('hello')->header('Content-Type', $type);

#### 附加 Cookies 到响应

	$cookie = Cookie::make('name', 'value');

	return Response::make($content)->withCookie($cookie);

<a name="redirects"></a>
## 重定向跳转

#### 回传重定向跳转

	return Redirect::to('user/login');

#### 回传重定向跳转并且加上快闪数据（ Flash Data ）

	return Redirect::to('user/login')->with('message', 'Login Failed');

> **提示：** `with` 方法会设定快闪数据到 session，所以可以使用 `Session::get` 取得数据。

#### 回传根据路由名称的重定向跳转

	return Redirect::route('login');

#### 回传根据路由名称的重定向跳转，并给予路由参数赋值

	return Redirect::route('profile', array(1));

#### 回传根据路由名称的重定向跳转，并给予特定名称路由参数赋值

	return Redirect::route('profile', array('user' => 1));

#### 回传根据控制器动作的重定向跳转

	return Redirect::action('HomeController@index');

#### 回传根据控制器动作的重定向跳转，并给予参数赋值

	return Redirect::action('UserController@profile', array(1));

#### 回传根据控制器动作的重定向跳转，并给予特定名称参数赋值

	return Redirect::action('UserController@profile', array('user' => 1));

<a name="views"></a>
## Views

视图通常包含 HTML，并且提供便利的方式分开控制器和表现层。视图储存在 `app/views` 目录下。

一个简单的视图可能看起来如下：

	<!-- View stored in app/views/greeting.php -->

	<html>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

视图可以像这样回传到用户浏览器：

	Route::get('/', function()
	{
		return View::make('greeting', array('name' => 'Taylor'));
	});

`View::make` 方法传入的第二个参数是可以在视图里使用的数组数据。

#### 传递数据到视图

	// 使用通用方式
	$view = View::make('greeting')->with('name', 'Steve');

	// 使用魔术方法
	$view = View::make('greeting')->withName('steve');

上面的例子里，将可以在视图里使用变量 `$name` ，其值为 `Steve` 。

您可以传入数组作为 `make` 方法第二个参数：

	$view = View::make('greetings', $data);

您也可以设定所有视图共用数据：

	View::share('name', 'Steve');

#### 传递子视图到视图

有时候您可能想要传递子视图到另一个视图。例如，有一个子视图存在 `app/views/child/view.php`，可以像这样将它传递到另一个视图：

	$view = View::make('greeting')->nest('child', 'child.view');

	$view = View::make('greeting')->nest('child', 'child.view', $data);

如此可以在视图里渲染子视图：

	<html>
		<body>
			<h1>Hello!</h1>
			<?php echo $child; ?>
		</body>
	</html>

#### 确认视图是否存在

如果您需要确认视图是否存在，使用 `View::exists` 方法：

	if (View::exists('emails.customer'))
	{
		//
	}

<a name="view-composers"></a>
## 视图组件

视图组件是当渲染视图时调用的回调函数或类方法。如果您想在每次渲染某些视图时绑定数据，视图组件可以把这样的逻辑组织在同一个地方。因此，视图组件的作用可能如 "view models" 或是 "presenters"。

#### 定义一个组件

	View::composer('profile', function($view)
	{
		$view->with('count', User::count());
	});

之后每当 `profile` 视图被渲染时， `count` 变量就会被绑定到视图。

您也可以把一个组件同时附加到很多视图。

	View::composer(array('profile','dashboard'), function($view)
	{
		$view->with('count', User::count());
	});

如使用类作为组件，提供了可以从 [IoC 容器](/docs/4.2/ioc)自动解析组件的好处，您可以像这样做：

	View::composer('profile', 'ProfileComposer');

一个视图组件类应该像这样定义：

	class ProfileComposer {

		public function compose($view)
		{
			$view->with('count', User::count());
		}

	}

#### 定义很多组件

您可以使用 `composers` 方法群组视图组件：

	View::composers(array(
		'AdminComposer' => array('admin.index', 'admin.profile'),
		'UserComposer' => 'user',
		'ProductComposer@create' => 'product'
	));

> **提示：** 组件类没有一定要放在什么地方，您可以将它们放在任何地方，只要可以使用 `composer.json` 自动载入即可。

### 视图创建者

视图 **创建者** 几乎和组件运作方式一样；只是他们会在视图初始化时就立刻建立起来。要注册一个创建者，只要使用 `creator` 方法：

	View::creator('profile', function($view)
	{
		$view->with('count', User::count());
	});

<a name="special-responses"></a>
## 特殊响应

#### 建立 JSON 响应

	return Response::json(array('name' => 'Steve', 'state' => 'CA'));

#### 建立 JSONP 响应

	return Response::json(array('name' => 'Steve', 'state' => 'CA'))->setCallback(Input::get('callback'));

#### 建立下载文件响应

	return Response::download($pathToFile);

	return Response::download($pathToFile, $name, $headers);

> **提醒：** 管理文件下载的扩展包，Symfony HttpFoundation，要求下载文件名必须为 ASCII 。

<a name="response-macros"></a>
## 响应宏

如果您想要自定义可以在很多路由和控制器重复使用的响应，可以使用 `Response::macro` 方法：

	Response::macro('caps', function($value)
	{
		return Response::make(strtoupper($value));
	});

`macro` 方法第一个参数为宏名称，第二个参数为闭合函数。闭合函数会在 `Response` 调用宏名称的时候被执行：

	return Response::caps('foo');

您可以把定义自己的宏放在 `app/start` 里面的文件。又或者，您可以将宏组织成独立的文件，并且从其中一个 `start` 文件里引入。
