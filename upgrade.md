# 升级指南

- [从 4.2 升级到 4.3](#upgrade-4.3)
- [从 4.1 升级到 4.2](#upgrade-4.2)
- [从 4.1.x 升级到 4.1.29](#upgrade-4.1.29)
- [从 4.1.25 升级到 4.1.26](#upgrade-4.1.26)
- [从 4.0 升级至 4.1](#upgrade-4.1)

<a name="upgrade-4.3"></a>
## 从 4.2 升级到 4.3

### Beanstalk Queuing

Laravel 4.3 开始默认载入 `"pda/pheanstalk": "~3.0"` 取代 Laravel 4.2 `"pda/pheanstalk": "~2.1"`。

<a name="upgrade-4.2"></a>
## 从 4.1 升级到 4.2

### PHP 5.4+

Laravel 4.2 需要 PHP 5.4.0 以上。

### 默认加密

增加一个新的 `cipher` 选项在您的 `app/config/app.php` 配置文件中。其选项值应为 `MCRYPT_RIJNDAEL_256`。

	'cipher' => MCRYPT_RIJNDAEL_256

该设置可用于设定所使用的 Laravel 加密工具的默认加密方法。

> **附注:** 在 Laravel 4.2，默认加密方法为`MCRYPT_RIJNDAEL_128` (AES), 被认为是最安全的加密. 必须将加密改回`MCRYPT_RIJNDAEL_256` 来解密在 Laravel <= 4.1 下加密的 cookies/values 

### 软删除模型改用traits

如果您在模型下有使用软删除，现在 `softDeletes` 的属性已经被移除。您现在要使用 `SoftDeletingTrait` 如下：

	use Illuminate\Database\Eloquent\SoftDeletingTrait;

	class User extends Eloquent {
		use SoftDeletingTrait;
	}

您一样必须手动增加 `deleted_at` 字段到您的 `dates` 属性中：

	class User extends Eloquent {
		use SoftDeletingTrait;

		protected $dates = ['deleted_at'];
	}

而所有软删除的 API 使用方式维持相同。

> **附注:** `SoftDeletingTrait` 无法在基本模型下被使用。他只能在一个实际模型类中使用。

### 视图 / 分页 / 环境 类改名

如果您直接使用 `Illuminate\View\Environment` 或 `Illuminate\Pagination\Environment` 类，请更新您的代码将其改为参照 `Illuminate\View\Factory` 和 `Illuminate\Pagination\Factory`。改名后的这两个类更可以代表他们的功能。

### 分页呈现的额外参数

如果你继承了 `Illuminate\Pagination\Presenter` 类， 它的抽象方法 `getPageLinkWrapper` 已经改变参数了， 增加了一个 `rel` 参数：

	abstract public function getPageLinkWrapper($url, $page, $rel = null);

### Iron.Io Queue 加密

如果您使用 Iron.io queue 驱动，您将需要增加一个新的 `encrypt` 选项到您的 queue 配置文件中：

    'encrypt' => true

<a name="upgrade-4.1.29"></a>
## 从 4.1.x 升级到 4.1.29

Laravel 4.1.29 对于所有的数据库驱动加强了 column quoting 的部分。当您的模型中**没有**使用 `fillable` 属性，他保护您的应用程序不会受到 mass assignment 漏洞影响。如果您在模型中使用 `fillable` 属性来防范 mass assignment, 您的应用程序将不会有漏洞。如果您使用 `guarded` 且在 "更新" 或 "储存" 类型的函数中，传递了末端用户控制的数组，那您应该立即升级到 `4.1.29` 以避免 mass assignment 的风险。

升级到 Laravel 4.1.29，只要 `composer update` 即可。在这个发行版本中没有重大的更新。

<a name="upgrade-4.1.26"></a>
## 从 4.1.25 升级到 4.1.26

Laravel 4.1.26 采用了针对 "记住我" cookies 的安全性更新。在此更新之前，如果一个记住我的 cookies 被恶意用户劫持，该 cookie 将还可以生存很长一段时间，即使真实用户重设密码或者登出。

此更动需要在您的 `users` (或者类似的) 的数据库表中增加一个额外的 `remember_token` 字段。在更新之后，当用户每次登入您的应用程序将会有一个全新的 token 生成。这个 token 也会在用户登出应用程序后被更新。这个更新的影响为：如果一个"记住我"的 cookie 被劫持，只要用户登出应用程序将会废除该 cookie。

### 升级路径

首先，增加一个新的字段，可空值、属性为 VARCHAR(100)、TEXT 或同类型的字段 `remember_token` 到您的 `users` 数据库表中。

然后，如果您使用 Eloquent 认证驱动，依照下面更新您的 `User` 类的三个方法：

	public function getRememberToken()
	{
		return $this->remember_token;
	}

	public function setRememberToken($value)
	{
		$this->remember_token = $value;
	}

	public function getRememberTokenName()
	{
		return 'remember_token';
	}

> **附注:** 所有现存的 "记住我" sessions 在此更新后将会失效，所以应用程序的所有用户将会被迫重新登入。

### 扩展包管理者

两个新的方法被加入到 `Illuminate\Auth\UserProviderInterface` interface. 例子实现方式可以在默认驱	动中找到:

	public function retrieveByToken($identifier, $token);

	public function updateRememberToken(UserInterface $user, $token);

The `Illuminate\Auth\UserInterface` also received the three new methods described in the "Upgrade Path".

<a name="upgrade-4.1"></a>
## 从 4.0 升级至 4.1

### 升级您的 Composer 相依性

升级您的应用程序至 Laravel 4.1，将 `composer.json` 里的 `laravel/framework` 版本更改至 `4.1.*`。

### 文件置换

将您的 `public/index.php` 置换成 [这个 repository 的干净版本](https://github.com/laravel/laravel/blob/master/public/index.php)。

同样的，将您的 `artisan` 置换成 [这个 repository 的干净版本](https://github.com/laravel/laravel/blob/master/artisan)。

### 新增配置文件及选项

更新您在配置文件 `app/config/app.php` 里的 `aliases` 和 `providers` 数组。而更新的选项值可以在[这个文件](https://github.com/laravel/laravel/blob/master/app/config/app.php)中找到。请确定将您后来加入自定义和扩展包所需的 providers / aliases 加回数组中。

从 [这个 repository](https://github.com/laravel/laravel/blob/master/app/config/remote.php)增加 `app/config/remote.php` 文件。

在您的 `app/config/session.php` 增加新的选项 `expire_on_close`。而默认值为 `false`。

在您的 `app/config/queue.php` 文件里新增 `failed` 设定区块。以下为区块的默认值：

	'failed' => array(
		'database' => 'mysql', 'table' => 'failed_jobs',
	),

**（非必要）** 在您的 `app/config/view.php` 里，将 `pagination` 设定选项更新为 `pagination::slider-3`。

### 更新控制器（Controllers）

如果 `app/controllers/BaseController.php` 有 `use` 语句在最上面，将 `use Illuminate\Routing\Controllers\Controller;` 改为 `use Illuminate\Routing\Controller;`。

### 更新密码提醒

密码提醒功能已经大幅修正拥有更大的弹性。您可以执行 Artisan 命令 `php artisan auth:reminders-controller` 来检查新的存根控制器。您也可以浏览 [更新文档](/docs/4.2/security#password-reminders-and-reset) 然后相应的更新您的应用程序。

更新您的 `app/lang/en/reminders.php` 语系文件来符合[这个新版文件](https://github.com/laravel/laravel/blob/master/app/lang/en/reminders.php)。

### 更新环境检测

为了安全因素，不再使用 url 来检测应用程序的环境。因为这些值很容易被伪造欺骗，继而让攻击者通过请求来达到变更环境。所以您必须改为使用机器的 hostname（在 Mac & Ubuntu 下执行 `hostname` 出来的值）

（译按：的确原有方式有安全性考量，但对于现行 VirtualHost 大量使用下，反而这样改会造成不便）

### 更简单的日志文件

Laravel 目前只会产生单一的日志文件: `app/storage/logs/laravel.log`。然而，您还是可以通过设定您的 `app/start/global.php` 文件来配置他的行为。

### 删除重定向结尾的斜线

在您的 `bootstrap/start.php` 文件中，移除调用 `$app->redirectIfTrailingSlash()`。这个方法已不再需要了，因为之后将会被框架里的 `.htaccess` 处理。

然后，用[新版](https://github.com/laravel/laravel/blob/master/public/.htaccess)替换掉您 Apache 中的 `.htaccess` 文件，来处理结尾的斜线问题。

### 获取当前路由

获取当前路由的方法由 `Route::getCurrentRoute()` 改为 `Route::current()`。

### Composer 更新

一旦您完成以上的更新，您可以执行 `composer update` 来更新应用程序的核心文件。如果有 class load 错误，请在 `update` 之后加上 `--no-scripts`，如: `composer update --no-scripts`。

### 通配符事件监听

通配符事件的监听器不再附件事件到对应的处理函数的参数中。 如果你需要找到被触发的事件， 应该使用 `Event::firing()` 。
