# 控制器 (Controllers)

- [基本控制器](#basic-controllers)
- [控制器过滤器(Controller filters)](#controller-filters)
- [隐式控制器](#implicit-controllers)
- [RESTful 资源控制器](#restful-resource-controllers)
- [处理遗漏的方法](#handling-missing-methods)

<a name="basic-controllers"></a>
## 基本 Controllers

除了在 `routes.php` 文件中定义所有路由层逻辑外，您可能也想利用控制器来整合这些行为。控制器可以利用更先进的框架特性的优势，如自动的[依赖注入](/docs/4.2/ioc)整合相关的路由逻辑到同一个类中。

控制器一般存放在 `app/controllers` 目录下，这个目录已默认注册在 `composer.json` 的 `classmap` 中。然而，控制器可以放在任何目录或是子目录中。路由定义与控制器类存放在哪个位置并无关系。所以，只要 Composer 知道如何自动载入控制器类，您就可以把控制器放在任何您想要的地方。

下面是一个基本的控制器类例子：

	class UserController extends BaseController {

		/**
		 * Show the profile for the given user.
		 */
		public function showProfile($id)
		{
			$user = User::find($id);

			return View::make('user.profile', array('user' => $user));
		}

	}

所有的控制器都应该继承自 `BaseController` 类。`BaseController` 也放在 `app/controllers` 目录下，`BaseController` 可以作为放置共同控制器逻辑的地方。`BaseController` 继承了框架的 `Controller` 类。现在，我们可以像这样将请求从路由导至控制器中：

	Route::get('user/{id}', 'UserController@showProfile');

如果您选择使用 PHP 命名空间来分层或整合您的控制器，只要在定义路由时使用完整的类名称：

	Route::get('foo', 'Namespace\FooController@method');

> **注意：**既然我们使用 [Composer](http://getcomposer.org) 自动载入 PHP 类，因此只要 Composer 知道如何载入控制器，控制器文件就可以放在文件系统的任何地方。对您的应用程序而言，并没有强制控制器文件要使用怎样的目录结构。路由至控制器是完全与文件系统脱离的。

您可以为控制器路由指定名称：

	Route::get('foo', array('uses' => 'FooController@method',
											'as' => 'name'));

您可以使用 `URL::action` 方法或者是 `action` 辅助方法来产生到控制器行为的 URL：

	$url = URL::action('FooController@method');

	$url = action('FooController@method');

您可以利用 `currentRouteAction` 方法取得正在执行的控制器行为名称：

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## 控制器过滤器

[过滤器(Filter)](/docs/4.2/routing#route-filters) 在控制器路由中定义方式，如同"一般"的路由一样。

	Route::get('profile', array('before' => 'auth',
				'uses' => 'UserController@showProfile'));

然而，您也可以在控制器中指定过滤器：

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth', array('except' => 'getLogin'));

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
								array('fooAction', 'barAction')));
		}

	}

您可以在控制器里使用闭合函数定义过滤器：

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

如果您想使用控制器里的方法当做过滤器，您可以使用`@`语法定义过滤器：

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('@filterRequests');
		}

		/**
		 * Filter the incoming requests.
		 */
		public function filterRequests($route, $request)
		{
			//
		}

	}

<a name="implicit-controllers"></a>
## 隐式控制器

Laravel 让您可以简单的经由定义一个路由规则来处理控制器里的行为。首先，使用 `Route::controller` 方法定义路由：

	Route::controller('users', 'UserController');

`controller` 方法可以接收两个参数，第一个是控制器对应的基本 URI，第二个是控制器的类名称。接下来，只要把对应的 HTTP 请求动词前缀加在控制器方法前：

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

		public function anyLogin()
		{
			//
		}

	}

`index` 方法会对应到 controller 的根 URI，以上面的例子来说，就是 `users`。

若您的控制器方法包含很多单词，您可以在 URI 使用 "破折号(-)" 来对应方法。例如 `UserController` 中，如下的方法会对应到 `users/admin-profile` URI：

	public function getAdminProfile() {}

<a name="restful-resource-controllers"></a>
## RESTful 资源控制器

资源控制器可以简单的建立跟资源相关的 RESTful 控制器。例如，您可能想要建立控制器来管理应用程序里储存的照片。通过 Artisan 命令行工具里的 `controller:make` 以及使用 `Route::resource` 方法，可以很快的创建控制器。

命令行执行以下命令建立控制器：

	php artisan controller:make PhotoController

然后我们可以注册一个资源化路由至控制器上：

	Route::resource('photo', 'PhotoController');

此单一路由定义创建了处理一系列对图片资源的 RESTful 行为路由。同样的，刚才产生的控制器文件里面, 对这些动作已经有预先建立好的对应方法，以及注释告知对应的 URI 和所处理的请求动作。

#### 资源控制器对应的动作

Verb      | Path                        | Action       | Route Name
----------|-----------------------------|--------------|---------------------
GET       | /resource                   | index        | resource.index
GET       | /resource/create            | create       | resource.create
POST      | /resource                   | store        | resource.store
GET       | /resource/{resource}        | show         | resource.show
GET       | /resource/{resource}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{resource}        | update       | resource.update
DELETE    | /resource/{resource}        | destroy      | resource.destroy

有时您可能只需要对应部分的资源的动作：

	php artisan controller:make PhotoController --only=index,show

	php artisan controller:make PhotoController --except=index

您也可以在路由定义时指定只需要对应的动作：

	Route::resource('photo', 'PhotoController',
					array('only' => array('index', 'show')));

	Route::resource('photo', 'PhotoController',
					array('except' => array('create', 'store', 'update', 'destroy')));

默认所有的资源控制器动作都有路由名称，您也可以在选项传入 `names` 数组，重写这些名称：

	Route::resource('photo', 'PhotoController',
					array('names' => array('create' => 'photo.build')));

#### 处理嵌套资源控制器

为了使用嵌套资源控制器，在路由定义时使用"点"表示法：

	Route::resource('photos.comments', 'PhotoCommentController');

这个路由规则会注册一个"嵌套"资源，可以对应如下的 URLs：`photos/{photoResource}/comments/{commentResource}`。

	class PhotoCommentController extends BaseController {

		public function show($photoId, $commentId)
		{
			//
		}

	}

#### 增加额外路由规则到 Resource Controller

如果您需要增加额外的 route 规则到默认的 resource controller，您应该在定义`Route::resource`之前定义这些规则：

	Route::get('photos/popular', 'PhotoController@method');
	Route::resource('photos', 'PhotoController');

<a name="handling-missing-methods"></a>
## 对应缺失的方法

当使用 `Route::controller` 时，可以定义一个 catch-all 方法，当 controller 找不到对应的方法时就会被调用，这个方法名称为`missingMethod`, 会传入请求的方法和参数数组：

#### 定义一个 Catch-All 方法

	public function missingMethod($parameters = array())
	{
		//
	}

If you are using resource controllers, you should define a `__call` magic method on the controller to handle any missing methods.
