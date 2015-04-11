# 设定

- [简介](#introduction)
- [环境设定](#environment-configuration)
- [供应商设定](#provider-configuration)
- [保护敏感设定](#protecting-sensitive-configuration)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 简介

所有关于 Laravel 框架的设定文件都被放置在 `app/config` 目录下。每个文件里的所有选项都有注释，因此您可以轻松地察看这些文件，请尽可能的去熟悉这些选项配置。

有时候，您可能在运行时需要获取这些设定值，您可以使用 `Config` 类：

#### 获取一个选项的值

	Config::get('app.timezone');

如果选项值不存在，您可以指定一个默认值：

	$timezone = Config::get('app.timezone', 'UTC');

#### 设定选项值

注意，"点"式语法可以用来获取不同设定文件里的选项。您还可以在运行阶段更改设定值:

	Config::set('database.default', 'sqlite');

在运行阶段设定的选项值只在该次请求中有效，不会对其他的请求造成影响。

<a name="environment-configuration"></a>
## 环境配置
通常应用程序需要根据不同的运行环境而有不同的配置设定值。例如，您会希望在您的本地开发机器上会有与正式环境不同的缓存驱动（cache driver），通过配置文件，这是非常容易达成的。

在 `config` 目录下建立与环境名称相同的目录，例如 `local`。接下来，创建您想要重写的设定文件，并且设定该环境所希望的设定值。例如，您要在 `app/config/local` 建立 `cache.php` 文件，内容如下：

	<?php

	return array(

		'driver' => 'file',

	);

> **注:** 请勿使用 'testing' 当作环境名称，它是专门为单元测试保留的。

注意，您不需要为基本设定文件中的_所有_选项设定选项值，只需要指定您需要覆蓋的配置选项即可。环境配置文件将会以层叠替换的方式覆蓋基本设定文件。

接下来，我们需要让框架知道如何确认其运行环境。默认环境是 `production`。然而，您可以在安装目录下的 `bootstrap/start.php` 文件中设定其他环境。在该文件中，您可以找到 `$app->detectEnvironment` 函数。该数组将会用来侦测当前的运行环境。您可以根据您的需求增加环境或者是机器名称。

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

在这个例子中，'local' 是运行环境的名称而 'your-machine-name' 是您的服务器的主机名称。在 Linux 和 Mac 上，您可以通过命令行执行 `hostsname` 查到您的主机名称。

如果您想要更灵活的环境侦测方式，可以传递一个 `闭包（Closure）` 给 `detectEnvironment` 函数，这样您就可以按照您想要的方式侦测了：

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

#### 获取目前的运行环境

您也可以通过 `environment` 函数来取的目前运行阶段的环境：

	$environment = App::environment();

您也可以传递参数至 `environment` 函数中，来确认目前的环境是否与参数相符合：

	if (App::environment('local'))
	{
		// 当环境为 local 时
	}

	if (App::environment('local', 'staging'))
	{
		// 环境为 local 或 staging...
	}

<a name="provider-configuration"></a>
### 供应商设定

当使用环境设定时，您会想要"附加"环境[服务供应商](/docs/4.2/ioc#service-providers) 到您主要的 `app` 配置文件中。然而，如果您这样做，您会发现环境 `app` 供应商将会覆蓋掉您主要 `app` 配置文件的供应商。如果要附加供应商，在您的环境 `app` 配置文件中使用 `append_config` helper 方法：

	'providers' => append_config(array(
		'LocalOnlyServiceProvider',
	))

<a name="protecting-sensitive-configuration"></a>
## 保护敏感设定

对于一个"实际"的应用程序，不把敏感的设定放在您的配置文件中, 是明智的抉择。如数据库密码, Stripe API Key 和 `加密金钥` 都应该尽可能地不要存在配置文件中。所以，该放在哪里？很庆幸的，Laravel 提供了一个非常简单的方式：使用 "点开头" 的仿环境变量文件来保存这种类型的设定。

首先，[设定您的应用程序](/docs/4.2/configuration#environment-configuration)来识别您的机器是否在 `local` 环境中。然后创建一个 `.env.local.php` 文件在您的项目最上层目录中（一般而言与 `composer.json` 文件同目录)。`.env.local.php` 将会回传一个键值组的数组，类似于一般的 Laravel 配置文件：

	<?php

	return array(

		'TEST_STRIPE_KEY' => 'super-secret-sauce',

	);


所有在这个文件中的键值组将会被回传，并转成 PHP 的超级全局变量 `$_ENV` 和 `$_SERVER`。您可以在您的配置文件中通过这些全局变量获取：

	'key' => $_ENV['TEST_STRIPE_KEY']

请确认将 `.env.local.php` 加进您的 `.gitignore` 文件中。他允许您的团队成员可以创建自己的本机环境配置文件，也可以将敏感设定从版本控制系统中隐藏。

现在，您可以在正式环境中，创建一个包含正式环境敏感设定值的 `.env.php` 于您项目的最上层目录。如同 `.env.local.php`，正式环境的 `.env.php` 也不应该在版本控制系统中。

> **附注:** 您可以为每个有支持的环境创建所属的配置文件。例如，`development` 环境下，如果 `.env.development.php` 文件若存在将会自动读取进来。然而， `production` 环境会始终使用 `.env.php` 文件。

<a name="maintenance-mode"></a>
## 维护模式

当您的应用程序处于维护模式时，所有的路由都会指向一个自定义的视图。当您要更新或进行维护工作时，“关闭”整个网站是很简单的。

启用维护模式，只要执行 Artisan 命令 'down'：

	php artisan down

关闭维护模式，只要执行 Artisan 命令 'up'：

	php artisan up

如果您想要自定义维护模式的页面，您只需要增加下面内容至应用程序里的 `app/start/global.php` 文件中：

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});

如果传给 `down` 函数的闭包回传 'NULL' 值，`该此请求将会略过维护模式`。

### 维护模式与队列

当应用程序处于维护模式中，将不会处理任何[队列工作](/docs/4.2/queues)。所有的队列工作将会在应用程序离开维护模式后继续被进行。
