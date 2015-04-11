# 迁移和数据填充

- [介绍](#introduction)
- [建立迁移文件](#creating-migrations)
- [执行迁移](#running-migrations)
- [回滚迁移](#rolling-back-migrations)
- [数据填充](#database-seeding)

<a name="introduction"></a>
## 介绍

迁移是一种数据库的版本控制。可以让团队在修改数据库结构的同时，保持彼此的进度一致。迁移通常会和[结构生成器](/docs/4.2/schema)一起使用，可以简单的管理数据库结构。

<a name="creating-migrations"></a>
## 建立迁移文件

使用 Artisan CLI 的 `migrate:make` 命令建立迁移文件：

	php artisan migrate:make create_users_table

迁移文件会建立在 `app/database/migrations` 目录下，文件名会包含时间戳，用于在执行迁移时用来决定顺序。

您也可以在建立迁移命令加上 `--path` 参数指定要生成的目录，路径要相对于应用程序所在的根目录。

	php artisan migrate:make foo --path=app/migrations

`--table` 和 `--create` 参数可以用来指定数据库表名称，以及迁移文件是否要建立新的数据库表。

	php artisan migrate:make add_votes_to_user_table --table=users

	php artisan migrate:make create_users_table --create=users

<a name="running-migrations"></a>
## 执行迁移

#### 执行所有未执行迁移

	php artisan migrate

#### 执行指定路径下所有未执行迁移

	php artisan migrate --path=app/foo/migrations

#### 执行特定扩展包的所有未执行迁移

	php artisan migrate --package=vendor/package

> **提示：** 如果在执行迁移时发生 "class not found"  错误，试着先执行 `composer dump-autoload`  命令后再进行一次。

### 在上线（ Production ）环境执行强制迁移

有些迁移操作是具有破坏性的，意味着可能让您遗失原本储存的数据。为了防止您在上线环境执行到这些迁移命令，您会被提示要在执行迁移前进行确认。加上 `--force` 参数执行强制迁移：

	php artisan migrate --force

<a name="rolling-back-migrations"></a>
## 回滚迁移

#### 回滚上一次的迁移

	php artisan migrate:rollback

#### 回滚所有迁移

	php artisan migrate:reset

#### 回滚所有迁移并且再执行一次

	php artisan migrate:refresh

	php artisan migrate:refresh --seed

<a name="database-seeding"></a>
## 数据填充

Laravel 可以简单的使用 seed 类填充测试数据到数据库。所有的 seed 类放在 `app/database/seeds` 目录下。可以使用任何您想要的类名称，但是应该遵守某些大小写规范，如 `UserTableSeeder` 之类。默认已经有一个 `DatabaseSeeder` 类。在这个类里，使用 `call` 方法执行其他的 seed 类，让您控制填充的顺序。

#### Seed 类例子

	class DatabaseSeeder extends Seeder {

		public function run()
		{
			$this->call('UserTableSeeder');

			$this->command->info('User table seeded!');
		}

	}

	class UserTableSeeder extends Seeder {

		public function run()
		{
			DB::table('users')->delete();

			User::create(array('email' => 'foo@bar.com'));
		}

	}

要执行数据填充，可以使用 Artisan CLI 的 `db:seed` 命令：

	php artisan db:seed

默认 `db:seed` 命令会执行 `DatabaseSeeder`，可以使用它来调用其他 seed 类，不过，也可以使用 `--class` 参数指定要单独执行的类：

	php artisan db:seed --class=UserTableSeeder

您可以也使用 `migrate:refresh` 命令填充数据，它会回滚并且再次执行所有迁移：

	php artisan migrate:refresh --seed
