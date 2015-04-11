# 查询构造器

- [介绍](#introduction)
- [Selects](#selects)
- [Joins](#joins)
- [进阶 Wheres](#advanced-wheres)
- [聚合](#aggregates)
- [原生表达式](#raw-expressions)
- [新增](#inserts)
- [更新](#updates)
- [删除](#deletes)
- [Unions](#unions)
- [悲观锁定](#pessimistic-locking)
- [缓存查询结果](#caching-queries)

<a name="introduction"></a>
## 介绍

数据库查询构造器 (query builder) 提供方便流畅的接口来建立、执行数据库查询语法。在您的应用程序里面，它可以被使用在大部分的数据
库操作，而且它在所有支持的数据库系统上都可以执行。

> **注意:** Laravel 查询构造器使用 PDO 参数绑定，以保护应用程序免于SQL注入攻击 (SQL injection)，因此传入的参数不需过滤额外的特殊字符串。

<a name="selects"></a>
## Selects

#### 从数据库表中取得所有的数据列

	$users = DB::table('users')->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

#### 从数据库表中取得单一数据列

	$user = DB::table('users')->where('name', 'John')->first();

	var_dump($user->name);

#### 从数据库表中取得单一数据列的单一字段

	$name = DB::table('users')->where('name', 'John')->pluck('name');

#### 取得单一字段值的列表

	$roles = DB::table('roles')->lists('title');

这个方法将会回传含有数据库表 role 的 title 字段值的数组。您也可以通过下面的方法，为回传的数组指定自定义键值。

	$roles = DB::table('roles')->lists('title', 'name');

#### 指定查询子句 (Select Clause)

	$users = DB::table('users')->select('name', 'email')->get();

	$users = DB::table('users')->distinct()->get();

	$users = DB::table('users')->select('name as user_name')->get();

#### 增加查询子句到现有的的查询中

	$query = DB::table('users')->select('name');

	$users = $query->addSelect('age')->get();

#### 使用 where 及运算符

	$users = DB::table('users')->where('votes', '>', 100)->get();

#### "or" 语法

	$users = DB::table('users')
	                    ->where('votes', '>', 100)
	                    ->orWhere('name', 'John')
	                    ->get();

#### 使用 Where Between

	$users = DB::table('users')
	                    ->whereBetween('votes', array(1, 100))->get();

#### 使用 Where Not Between

	$users = DB::table('users')
	                    ->whereNotBetween('votes', array(1, 100))->get();

#### 使用 Where In 与数组

	$users = DB::table('users')
	                    ->whereIn('id', array(1, 2, 3))->get();

	$users = DB::table('users')
	                    ->whereNotIn('id', array(1, 2, 3))->get();

#### 使用 Where Null 找有未设定的值的数据

	$users = DB::table('users')
	                    ->whereNull('updated_at')->get();

#### 排序(Order By), 分群(Group By), 及 Having

	$users = DB::table('users')
	                    ->orderBy('name', 'desc')
	                    ->groupBy('count')
	                    ->having('count', '>', 100)
	                    ->get();

#### 偏移(Offset) 及 限制(Limit)

	$users = DB::table('users')->skip(10)->take(5)->get();

<a name="joins"></a>
## Joins

查询构造器也可以使用 join 语法，看看下面的例子：

#### 基本的 Join 语法

	DB::table('users')
	            ->join('contacts', 'users.id', '=', 'contacts.user_id')
	            ->join('orders', 'users.id', '=', 'orders.user_id')
	            ->select('users.id', 'contacts.phone', 'orders.price')
	            ->get();

#### Left Join 语法

	DB::table('users')
		    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
		    ->get();

您也可以指定更进阶的 join 子句

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')->orOn(...);
	        })
	        ->get();

如果您想在您的 join 中使用 where 型式的子句，您可以在 join 子句里使用 `where` 或 `orWhere` 方法。下面的方法将会比较 contacts 数据库表中的 user_id 的字段值，而不是比较两个字段。

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')
	        	     ->where('contacts.user_id', '>', 5);
	        })
	        ->get();

<a name="advanced-wheres"></a>
## 进阶 Wheres

#### 群组化参数

有些时候您需要更进阶的 where 子句，如 "where exists" 或多维数组参数。Laravel 的查询构造器也可以处理这样的情况；

	DB::table('users')
	            ->where('name', '=', 'John')
	            ->orWhere(function($query)
	            {
	            	$query->where('votes', '>', 100)
	            	      ->where('title', '<>', 'Admin');
	            })
	            ->get();

上面的查询语法会产生下方的 SQL：

	select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

#### Exists 语法

	DB::table('users')
	            ->whereExists(function($query)
	            {
	            	$query->select(DB::raw(1))
	            	      ->from('orders')
	            	      ->whereRaw('orders.user_id = users.id');
	            })
	            ->get();

上面的查询语法会产生下方的 SQL：

	select * from users
	where exists (
		select 1 from orders where orders.user_id = users.id
	)

<a name="aggregates"></a>
## 聚合

查询构造器也提供各式各样的聚合方法，如 	`count`, `max`, `min`, `avg` 及 `sum`。

#### 使用聚合方法

	$users = DB::table('users')->count();

	$price = DB::table('orders')->max('price');

	$price = DB::table('orders')->min('price');

	$price = DB::table('orders')->avg('price');

	$total = DB::table('users')->sum('votes');

<a name="raw-expressions"></a>
## Raw Expressions

有些时候您需要使用 raw expression 在查询语句里，这样的表达式会成为字串插入至查询中，因此要小心勿建立任何 SQL 注入的攻击点。要建立 raw expression，您可以使用 `DB::raw` 方法：

#### 使用 Raw Expression

	$users = DB::table('users')
	                     ->select(DB::raw('count(*) as user_count, status'))
	                     ->where('status', '<>', 1)
	                     ->groupBy('status')
	                     ->get();

<a name="inserts"></a>
## 新增

#### 新增一条数据进数据库表

	DB::table('users')->insert(
		array('email' => 'john@example.com', 'votes' => 0)
	);

#### 新增自动递增 (Auto-Incrementing) ID 的数据至数据库表

如果数据库表有自动递增的ID，可以使用 `insertGetId` 新增数据并回传该 ID：

	$id = DB::table('users')->insertGetId(
		array('email' => 'john@example.com', 'votes' => 0)
	);

> **注意:** 当使用 PostgreSQL 时，insertGetId 方法会预先自动递增字段名为 "id" 的值。

#### 新增多条数据进数据库表

	DB::table('users')->insert(array(
		array('email' => 'taylor@example.com', 'votes' => 0),
		array('email' => 'dayle@example.com', 'votes' => 0),
	));

<a name="updates"></a>
## 更新

#### 更新数据库表中的数据

	DB::table('users')
	            ->where('id', 1)
	            ->update(array('votes' => 1));

#### 对字段递增或递减数值

	DB::table('users')->increment('votes');

	DB::table('users')->increment('votes', 5);

	DB::table('users')->decrement('votes');

	DB::table('users')->decrement('votes', 5);

您也可以同时更新其他字段

	DB::table('users')->increment('votes', 1, array('name' => 'John'));

<a name="deletes"></a>
## 删除

#### 删除数据库表中的数据

	DB::table('users')->where('votes', '<', 100)->delete();

#### 删除数据库表中的所有数据

	DB::table('users')->delete();

#### 清空数据库表

	DB::table('users')->truncate();

<a name="unions"></a>
## Unions

查询构造器也提供一个快速的方法去"合并 (union)"两个查询的结果：

	$first = DB::table('users')->whereNull('first_name');

	$users = DB::table('users')->whereNull('last_name')->union($first)->get();

`unionAll` 方法也可以使用，它与 `union` 方法的使用方式一样。

<a name="pessimistic-locking"></a>
## 悲观锁定 (Pessimistic Locking)

查询构造器提供了少数函数协助您在 SELECT 语句中做到“悲观锁定”。

您只要在 SELECT 查询语句中使用 `sharedLock`：

	DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

在 select 语法中，要"锁住更新(lock for update)"，您仅需在查询语句中使用 `lockForUpdate` 方法即可：

	DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

<a name="caching-queries"></a>
## 缓存查询结果

使用 `remember` 或者 `rememberForever` 方法，您可以轻松的缓存查询结果：

	$users = DB::table('users')->remember(10)->get();

这个例子中，查询结果将会缓存 10 分钟。当结果被缓存时，查询语句将不会被执行，而会从应用程序指定的缓存驱动器中载入缓存的结果。

如果您正在使用的是 [支持标签的缓存驱动器](/docs/4.2/cache#cache-tags)，您也可以为缓存增加标签：

	$users = DB::table('users')->cacheTags(array('people', 'authors'))->remember(10)->get();
