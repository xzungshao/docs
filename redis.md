# Redis

- [介绍](#introduction)
- [配置文件](#configuration)
- [使用方式](#usage)
- [管道](#pipelining)

<a name="introduction"></a>
## 介绍

[Redis](http://redis.io) 是开源，先进的键值对储存库。由于它可用的键包含了[字串](http://redis.io/topics/data-types#strings)，[哈希](http://redis.io/topics/data-types#hashes)，[列表](http://redis.io/topics/data-types#lists)，[集合](http://redis.io/topics/data-types#sets)，和[有序集合](http://redis.io/topics/data-types#sorted-sets)，因此常被称作数据结构服务器。

> **提醒：** 如果您通过 PECL 安装了 Redis PHP extension，则需要重新命名 `app/config/app.php` 里的 Redis 别名。

<a name="configuration"></a>
## 配置文件

应用程序的 Redis 配置文件在 **app/config/database.php** 。在这个文件里，您会看到 **redis** 数组，里面有应用程序使用的 Redis 服务器数据：

	'redis' => array(

		'cluster' => true,

		'default' => array('host' => '127.0.0.1', 'port' => 6379),

	),

默认的服务器设定对于开发应该是足够的。然而，您可以根据使用环境自由修改数组数据。只要给每个 Redis 一个名称，并且设定服务器的 host 和 port。

`cluster` 选项会让 Laravel 的 Redis 客户端在所有 Redis 节点间执行客户端分片（ client-side sharding ），让您建立节点池，并因此拥有大量的 RAM 可用。然而，客户端分片的节点不能执行容错转移；因此，这主要适合用可以从另一台主要数据储存库取得的缓存数据。

如果您的 Redis 服务器需要认证，您可以在 Redis 服务器配置文件里加入 `password` 参数设定。

<a name="usage"></a>
## 使用方式

您可以经由 `Redis::connection` 方法得到 Redis 实例：

	$redis = Redis::connection();

您会得到一个使用 Redis 默认服务器的实例。如果您没有使用服务器丛集，您可以在 `connection` 方法传入定义在 Redis 配置文件的服务器名称，以连到特定服务器：

	$redis = Redis::connection('other');

一旦您有了 Redis 客户端实例，就可以使用实例发出任何 [Redis 命令](http://redis.io/commands)。Laravel 使用魔术方法传递命令到服务器：

	$redis->set('name', 'Taylor');

	$name = $redis->get('name');

	$values = $redis->lrange('names', 5, 10);

注意，传入命令的参数仅只是传递到魔术方法里。当然，您不一定要使用魔术方法，您也可以使用 `command` 方法传递命令到服务器：

	$values = $redis->command('lrange', array(5, 10));

若您只想对默认服务器下命令，可以使用 `Redis` 类的静态魔术方法：

	Redis::set('name', 'Taylor');

	$name = Redis::get('name');

	$values = Redis::lrange('names', 5, 10);

> **提示：** 也可以使用 Redis 作为 Laravel 的[缓存](/docs/4.2/cache) 和 [session](/docs/4.2/session) 驱动。

<a name="pipelining"></a>
## 管道

当您想要一次发送很多命令到服务器时可以使用管道。使用 `pipeline` 方法：

#### 发送多个命令到服务器

	Redis::pipeline(function($pipe)
	{
		for ($i = 0; $i < 1000; $i++)
		{
			$pipe->set("key:$i", $i);
		}
	});
