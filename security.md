# 认证与安全性

- [设定](#configuration)
- [储存密码](#storing-passwords)
- [用户认证](#authenticating-users)
- [手动登入用户](#manually)
- [保护路由](#protecting-routes)
- [HTTP 简易认证](#http-basic-authentication)
- [忘记密码与密码重设](#password-reminders-and-reset)
- [加密](#encryption)
- [认证驱动](#authentication-drivers)

<a name="configuration"></a>
## 设定

Laravel 的目标就是要让实现认证机制变得简单。事实上，几乎所有的设置默认就已经完成了。有关认证的配置文件都放在 `app/config/auth.php` 里，而在这些文件里也都包含了良好的注释描述每一个选项的所对应的认证行为及动作。

Laravel 默认在 `app/models` 文件夹内就包含了一个使用 Eloquent 认证驱动的`User` 模型。请记得在建立模型结构时，密码字段至少要有 60 个字串宽度。

假如您的应用程序并不是使用 Eloquent ，您也可以使用 Laravel 的查询构造器做  `database` 认证驱动。

> **注意：** 在开始之前，请先确认您的 `users` (或其他同义) 数据库表包含一个名为 `remember_token` 长度为100的string类型、可接受 null 的字段。这个字段将会被用来储存「记住我」的 session token。

<a name="storing-passwords"></a>
## 储存密码

Laravel 的 `Hash` 类提供了安全的 Bcrypt 哈希演算法：

#### 对密码加密

	$password = Hash::make('secret');

#### 验证密码

	if (Hash::check('secret', $hashedPassword))
	{
		// The passwords match...
	}

#### 确认密码是否需要重新加密

	if (Hash::needsRehash($hashed))
	{
		$hashed = Hash::make('secret');
	}

<a name="authenticating-users"></a>
## 用户认证

您可以使用 `Auth::attempt` 方法来验证用户成功登录之前的信息确认：

	if (Auth::attempt(array('email' => $email, 'password' => $password)))
	{
		return Redirect::intended('dashboard');
	}

需提醒的是 `email`并不是一个必要的字段，在这里仅用于示范。您可以使用数据库里任何类似于「用户名称」的字段做为帐号的唯一标识。若用户尚未登入的话，认证筛选器会使用 `Redirect::intended` 方法重定向跳转用户至指定的 URL。我们可指定一个备用 URI ，当预定重定向跳转位置不存在时使用。

当 `attempt` 方法被调用时，`auth.attempt` [事件](/docs/4.2/events) 将会被触发。假如认证成功的话，则 `auth.login` 事件会接着被触发。

#### 判定用户是否已登入

判定一个用户是否已经登入您的应用程序，您可以使用 `check` 这个方法：

	if (Auth::check())
	{
		// The user is logged in...
	}

#### 认证一个用户并且「记住」他

假如您想要在您的应用程序内提供「记住我」的选项，您可以在 `attempt` 方法的第二个参数复制为 `true` ，这样就可以保留用户的认证身份 (或直到他手动登出为止)。当然，您的 `users` 数据库表必需包括一个字串类型的 `remember_token` 字段来储存「记住我」的标记。

	if (Auth::attempt(array('email' => $email, 'password' => $password), true))
	{
		// The user is being remembered...
	}

**注意：** 假如 `attempt` 方法回传 `true` ，则表示用户已经登入您的应用程序。

#### 通过记住我来认证用户
假如您让用户通过「记住我」的方式来登入，则您可以使用 `viaRemember` 方法来判定用户是否拥有「记住我」cookie 来认证用户登入：

	if (Auth::viaRemember())
	{
		//
	}

#### 条件认证用户

在认证过程中，您可能会想要增加额外的认证条件：

    if (Auth::attempt(array('email' => $email, 'password' => $password, 'active' => 1)))
    {
        // The user is active, not suspended, and exists.
    }

> **注意：** 为了增加对 session 的保护，用户的 session ID 将会在用户认证完成后自动重新产生。

#### 取得已登入的用户信息

当用户完成认证后，您就可以通过模型来取得相关的数据：

	$email = Auth::user()->email;

取得登入用户的 ID，您可以使用 `id` 方法：

	$id = Auth::id();

若想要通过用户的 ID 来登入应用程序，可直接使用 `loginUsingId` 方法：

	Auth::loginUsingId(1);

#### 验证用户信息而不要登入

`validate` 方法可以让您验证用户信息而不真的登入应用程序：

	if (Auth::validate($credentials))
	{
		//
	}

#### 在单一请求内登入用户

您也可以使用 `once` 方法来让用户在单一请求内登入。不会有任何 session 或 cookie 被产生：

	if (Auth::once($credentials))
	{
		//
	}

#### 将用户登出

	Auth::logout();

<a name="manually"></a>
## 手动登入用户

假如您需要将一个已存在的用户实例登入您的应用程序，只需要简单的调用 `login` 方法，并且将该实例作为参数传入即可：

	$user = User::find(1);

	Auth::login($user);

这个方式和用用户帐号密码来 `attempt` 的功能是一样的。

<a name="protecting-routes"></a>
## 保护路由

路由过滤器可让用户仅能访问特定的链接。Laravel 默认提供 `auth` 过滤器，其被定义在 `app/filters.php` 文件内。

#### 保护特定路由

	Route::get('profile', array('before' => 'auth', function()
	{
		// Only authenticated users may enter...
	}));

### CSRF 保护

Laravel 提供一个简易的方式来保护您的应用程序免于跨站攻击。

#### 在表单内引入 CSRF 标记

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

#### 验证表单的 CSRF 标记

    Route::post('register', array('before' => 'csrf', function()
    {
        return 'You gave a valid CSRF token!';
    }));

<a name="http-basic-authentication"></a>
## HTTP 简易认证

HTTP 简易认证提供了一个快速的方式来认证用户而不用特定设置一个「登入」页。在您的路由内设定 `auth.basic` 过滤器则可启动这个功能：

#### 用 HTTP 简易认证保护您的路由

	Route::get('profile', array('before' => 'auth.basic', function()
	{
		// Only authenticated users may enter...
	}));

`basic` 过滤器默认将会使用 `email` 字段来做用户认证，假如您想要使用其他字段来做认证的话，可在您的 `app/filters.php` 文件内将想要拿来做认证的字段当成第一个参数传给 `basic` 方法：

	Route::filter('auth.basic', function()
	{
		return Auth::basic('username');
	});

#### 设定无状态的 HTTP 简易过滤器

一般在实现 API 认证时，往往会想要使用 HTTP 简易认证而不要产生任何 session 或 cookie。我们可以通过定义一个过滤器并回传 `onceBasic` 方法来达成：

	Route::filter('basic.once', function()
	{
		return Auth::onceBasic();
	});

假如您是使用 PHP FastCGI，HTTP 简易认证默认是无法正常运作的。请在您的 `.htaccess` 文件内新增以下代码来启动这个功能：

	RewriteCond %{HTTP:Authorization} ^(.+)$
	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="password-reminders-and-reset"></a>
## 忘记密码与重设

### 模型与数据库表

大多数的web应用程序都会提供用户忘记密码的功能。为了不让开发者重复实现这个功能，Laravel 提供了方便的方法来发送忘记密码通知及密码重设的功能。在开始之前，请先确认您的 `User` 模型实现了 `Illuminate\Auth\Reminders\RemindableInterface` 接口。当然，默认 Laravel 的 `User` 模型本身就已实现，并且引入`Illuminate\Auth\Reminders\RemindableTrait` 来包括所有需要实现的接口方法。

#### 实现 RemindableInterface

	use Illuminate\Auth\Reminders\RemindableTrait;
	use Illuminate\Auth\Reminders\RemindableInterface;

	class User extends Eloquent implements RemindableInterface {

		use RemindableTrait;

	}

#### 生成 Reminder 数据库表迁移

接下来，我们需要生成一个数据库表来储存重设密码标记。为了产生这个数据库表的迁移文件，需要执行 `auth:reminders-table` artisan 命令：

	php artisan auth:reminders-table

	php artisan migrate

### 密码重设控制器

然后我们已经准备好生成密码重设控制器，您可以使用 `auth:reminders-controller` artisan 命令来自动生成这个控制器，它会在您的 `app/controllers` 文件夹内创建一个 `RemindersController.php` 文件。

	php artisan auth:reminders-controller

产生出来的控制器已经具备 `getRemind` 方法来显示您的忘记密码表单。您所需要做就是建立一个 `password.remind` [视图](/docs/4.2/responses#views)。这个视图需要具备一个 `email` 字段的表单，且这个表单应该要 POST 到 `RemindersController@postRemind` 动作。

一个简单的 `password.remind` 表单视图应该看起来像这样：

	<form action="{{ action('RemindersController@postRemind') }}" method="POST">
		<input type="email" name="email">
		<input type="submit" value="Send Reminder">
	</form>

除了 `getRemind` 外，控制器还包括了一个 `postRemind` 方法来处理发送忘记密码通知信给您的用户。这个方法会预期在 `POST` 参数内会有 `email` 字段。假如忘记密码通知信成功的寄发给用户，则会有一个 `status` 信息被暂存在 session 内；假如寄发失败的话，则取而代之的会有一个 `error` 信息被暂存。

在 `postRemind` 方法内，您可以在发送出去前修改信息的内容：

	Password::remind(Input::only('email'), function($message)
	{
		$message->subject('Password Reminder');
	});

您的用户将会收到一封电子邮件内有一个重设密码的链接指向 `getReset` 控制器方法。忘记密码标记是用来验证密码重设程序是否正确，也会传递给对应的控制器方法。这个动作已经设定会回传一个 `password.reset` 视图。这个 `token` 会被传递给视图，而您需要将这个 `token` 放在一个隐形的表单字段内。另外，您的重设密码表单应该包括 `email`、`password` 和 `password_confirmation` 字段。这个表单应该 POST 到 `RemindersController@postReset` 方法。

一个 `password.reset` 视图表单应该看起来像这样：

	<form action="{{ action('RemindersController@postReset') }}" method="POST">
		<input type="hidden" name="token" value="{{ $token }}">
		<input type="email" name="email">
		<input type="password" name="password">
		<input type="password" name="password_confirmation">
		<input type="submit" value="Reset Password">
	</form>

最后，`postReset` 方法则是专职处理重设过后的密码。在这个控制器方法里，Closure 传递 `Password::reset` 方法并且设定 `User` 内的 `password` 属性后调用 `save` 方法。当然，这个 Closure 会假定您的 `User` 模型是一个 [Eloquent 模型](/docs/4.2/eloquent) 。当然，您可以自由地修改这个 Closure 内容来符合您的应用程序数据库储存方式。

假如密码成功的重设，则用户会被重定向跳转至您的应用程序根目录。同样的，您可以自由的更改重定向跳转的 URL；假如密码重设失败的话，用户会被重定向跳转至重设密码表单页，而且会有 `error` 信息被暂存在 session 内。

### 密码验证

默认 `Password::reset` 方法会验证密码符合且大于等于六个字串。您可以用 `Password::validator` 方法 (接受 Closure) 来调整这些默认值。通过这个 Closure，您可以用任何方式来做密码验证。需提醒的是，您不一定要验证密码是否符合，因为框架自动会帮您完成这项工作。

	Password::validator(function($credentials)
	{
		return strlen($credentials['password']) >= 6;
	});

> **注意：** 密码重设标记默认会在一个小时后过期。您可以通过 `app/config/auth.php` 案内的 `reminder.expire` 选项来调整这个设定。

<a name="encryption"></a>
## 加密

Laravel 通过 mcrypt PHP 扩展来提供 AES 强度的加密算法：

#### 加密一个值

	$encrypted = Crypt::encrypt('secret');

> **注意：** 记得在 `app/config/app.php` 文件里设定一个 16, 24 或 32 字串的随机字做 `key` ，否则这个加密算法结果将不够安全。

#### 解密一个值

	$decrypted = Crypt::decrypt($encryptedValue);

#### 设定暗号及模式

您可以设定加密器的暗号及模式：

	Crypt::setMode('ctr');

	Crypt::setCipher($cipher);

<a name="authentication-drivers"></a>
## 认证驱动

Laravel 默认提供 `database` 及 `eloquent` 两种认证驱动。假如您需要更多有关增加额外认证驱动的详细信息，请参考 [认证扩充文件](/docs/4.2/extending#authentication)
