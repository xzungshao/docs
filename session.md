# 会话

- [配置信息](#configuration)
- [Session 用法](#session-usage)
- [快闪数据（Flash Data）](#flash-data)
- [数据库 Sessions](#database-sessions)
- [Session 驱动](#session-drivers)

<a name="configuration"></a>
## 配置信息

由于 HTTP 协定是无状态（Stateless）的，所以 session 提供一种储存用户数据的方法。Laravel 支持了多种 session 后端驱动，并通过清楚、统一的 API 提供使用。也内建支持如 [Memcached](http://memcached.org), [Redis](http://redis.io) 和数据库的后端驱动。

session 的配置文件配置在 `app/config/session.php` 中，请务必看一下 session 配置文件中可用的选项设定及注释。Laravel 默认使用 `file` 的 session 驱动，它在多数的应用中可以良好运作。

#### 保留键值

Laravel 框架在内部有使用 `flash` 作为 session 的键值，所以应该您在使用 session 时应避免使用此名称。


<a name="session-usage"></a>
## Session 用法

#### 储存数据到 Session 中

	Session::put('key', 'value');

#### 储存数据进 Session 数组值中

	Session::push('user.teams', 'developers');

#### 从 Session 取回数据

	$value = Session::get('key');

#### 从 Session 取回数据，若无则回传默认值

	$value = Session::get('key', 'default');

	$value = Session::get('key', function() { return 'default'; });

#### 从 Session 取回数据，并删除

	$value = Session::pull('key', 'default');

#### 从 Session 取出所有数据

	$data = Session::all();

#### 判断数据在 Session 中是否存在

	if (Session::has('users'))
	{
		//
	}

#### 移除 Session 中指定的数据

	Session::forget('key');

#### 清空整个 Session

	Session::flush();

#### 重新产生 Session ID

	Session::regenerate();

<a name="flash-data"></a>
## 快闪数据（Flash Data）

有时您可能希望暂存一些数据，并只在下次请求有效。您可以使用 `Session::flash` 方法来达成目的：

	Session::flash('key', 'value');

#### 刷新当前快闪数据，延长到下次请求

	Session::reflash();

#### 只刷新指定快闪数据

	Session::keep(array('username', 'email'));

<a name="database-sessions"></a>
## 数据库 Sessions

当使用 `database` session 驱动时，您必需创建一张储存 session 的数据库表。下面的例子使用 `Schema` 来建表：

	Schema::create('sessions', function($table)
	{
		$table->string('id')->unique();
		$table->text('payload');
		$table->integer('last_activity');
	});

当然您也可以使用 Artisan 命令 `session:table` 来建表：

	php artisan session:table

	composer dump-autoload

	php artisan migrate

<a name="session-drivers"></a>
## Session 驱动

session 配置文件中的 "driver" 定义了 session 数据将以哪种方式被储存。Laravel 提供了许多良好的驱动：

- `file` - sessions 将储存在 `app/storage/sessions` 文件夹中。
- `cookie` - sessions 将安全储存在加密的 cookies 中。
- `database` - sessions 将储存在您的应用程序数据库中。
- `memcached` / `redis` - sessions 将储存在一个高速缓存的系统中。
- `array` - sessions 将单纯的以 PHP 数组储存，只存在当次请求。

> **注意：** 运行 [unit tests](/docs/4.2/testing) 时，session 驱动会被设定为 `array`，所以不会留下任何 session 数据。
