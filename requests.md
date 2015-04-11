# 请求与输入

- [基本输入数据](#basic-input)
- [Cookies](#cookies)
- [旧输入数据](#old-input)
- [上传文件](#files)
- [请求信息](#request-information)

<a name="basic-input"></a>
## 基本输入数据

您可以经由几个简洁的方法拿到用户的输入数据。不需要担心发出请求时使用的 HTTP 响应方式，取得输入数据的方式都是相同的。

#### 取得特定输入数据

	$name = Input::get('name');

#### 取得特定输入数据，若没有便则取默认值

	$name = Input::get('name', 'Sally');

#### 确认是否有输入数据

	if (Input::has('name'))
	{
		//
	}

#### 取得所有发出请求时传入的输入数据

	$input = Input::all();

#### 取得部分发出请求时传入的输入数据

	$input = Input::only('username', 'password');

	$input = Input::except('credit_card');

如果是「数组」形式的输入数据，可以使用「点」语法取得数组：

	$input = Input::get('products.0.name');

> **提醒：** 有些 JavaScript 函数库如 Backbone 可能会送出 JSON 格式的输入数据，但是一样可以使用 `Input::get` 取得数据。

<a name="cookies"></a>
## Cookies

Laravel 建立的 cookie 会加密并且加上认证记号，意味着如果cookie被客户端擅自改动，会导致 cookie 失效。

#### 取得 Cookie 值

	$value = Cookie::get('name');

#### 加上新的 Cookie 到回应

	$response = Response::make('Hello World');

	$response->withCookie(Cookie::make('name', 'value', $minutes));

#### 加入 Cookie 队列到下一个回应

如果您想在回应被建立前设定 cookie ，使用 `Cookie::queue()` 方法。 Cookie 会在最后自动加到回应里。

	Cookie::queue($name, $value, $minutes);

#### 建立永久有效的 Cookie

	$cookie = Cookie::forever('name', 'value');

<a name="old-input"></a>
## 旧输入数据

您可能想要在用户下一次发送请求前，保留这次的输入数据。例如，您可能需要在表单验证失败后重新填入之前输入的表单值。

#### 将输入数据存成一次性 Session 

	Input::flash();

#### 将部分输入数据存成一次性 Session

	Input::flashOnly('username', 'email');

	Input::flashExcept('password');

您很可能常常需要在重新跳转至前一页，并将输入数据存成一次性 Session 。只要在重定向跳转方法串接的方法中传入输入数据，就能简单地完成。

	return Redirect::to('form')->withInput();

	return Redirect::to('form')->withInput(Input::except('password'));

> **提示：** 您可以使用 [Session](/docs/4.2/session) 类将不同请求数据存成其他一次性 Session。

#### 取得旧输入数据

	Input::old('username');

<a name="files"></a>
## 上传文件

#### 取得上传文件

	$file = Input::file('photo');

#### 确认文件是否有上传

	if (Input::hasFile('photo'))
	{
		//
	}

`file` 方法回传的对象是 `Symfony\Component\HttpFoundation\File\UploadedFile` 的实例， `UploadedFile` 继承了 PHP 的 `SplFileInfo` 类并且提供了很多方法和文件互动。

#### 确认上传的文件是否有效

	if (Input::file('photo')->isValid())
	{
		//
	}

#### 移动上传文件

	Input::file('photo')->move($destinationPath);

	Input::file('photo')->move($destinationPath, $fileName);

#### 取得上传文件所在的路径

	$path = Input::file('photo')->getRealPath();

#### 取得上传文件的原始名称

	$name = Input::file('photo')->getClientOriginalName();

#### 取得上传文件的后缀名

	$extension = Input::file('photo')->getClientOriginalExtension();

#### 取得上传文件的大小

	$size = Input::file('photo')->getSize();

#### 取得上传文件的 MIME 类型

	$mime = Input::file('photo')->getMimeType();

<a name="request-information"></a>
## 请求信息

`Request` 类提供很多方法检查 HTTP 请求，它继承了 `Symfony\Component\HttpFoundation\Request` 类，下面是一些使用方式。

#### 取得请求 URI

	$uri = Request::path();

#### 取得请求方法

	$method = Request::method();

	if (Request::isMethod('post'))
	{
		//
	}

#### 确认请求路径是否符合特定格式

	if (Request::is('admin/*'))
	{
		//
	}

#### 取得请求 URL

	$url = Request::url();

#### 取得请求 URI 部分片段

	$segment = Request::segment(1);

#### 取得请求 header 

	$value = Request::header('Content-Type');

#### 从 $_SERVER 取得值

	$value = Request::server('PATH_INFO');

#### 确认是否为 HTTPS 请求

	if (Request::secure())
	{
		//
	}

#### 确认是否为 AJAX 请求

	if (Request::ajax())
	{
		//
	}

#### 确认请求是否有 JSON Content Type

	if (Request::isJson())
	{
		//
	}

#### 确认是否要求 JSON 回应

	if (Request::wantsJson())
	{
		//
	}

#### 确认要求的回应格式

`Request::format` 方法会基于 HTTP Accept 标头回传请求的回应格式：

	if (Request::format() == 'json')
	{
		//
	}
