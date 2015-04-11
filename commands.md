# Artisan 开发

- [简介](#introduction)
- [自定义命令](#building-a-command)
- [注册命令](#registering-commands)
- [调用其它命令](#calling-other-commands)

<a name="introduction"></a>
## 简介

除了 Artisan 本身提供的命令之外，您也可以建立与您的应用程序相关的命令，这些自建命令将会存放在 `app/commands` 目录底下；然而，您可以任意选择存放位置，只要您的命令能够被 composer.json 自动载入。

<a name="building-a-command"></a>
## 自定义命令

### 生成类

要创建一个新的命令，您可以使用 command:make 这个 Artisan 命令，这将产生一个命令基本文件协助您开始编码：

#### 生成一个新的命令类

	php artisan command:make FooCommand

默认情况下，生成的命令将被储存在 `app/commands` 目录；然而，您可以指定自定义路径或命名空间：

	php artisan command:make FooCommand --path=app/classes --namespace=Classes

在创建命令时，加上 `--command` 这个选项，将可以指定这个命令的名称：

	php artisan command:make AssignUsers --command=users:assign

### 撰写自定义命令

当自定义命令生成后，您需再填写命令的 `名称` 与 `描述`，这部份将会显示在命令行表清单的画面上。

当您的自定义命令被执行时，将会调用 `fire` 方法，您可以在此加入任何的逻辑判断。

### 参数与选项

`getArguments` 与 `getOptions` 方法是用来接收要传入您的自定义命令的地方，这两个方法都会回传一组命令数组，并由数组清单所组成。

当定义 `arguments` 时，该数组对应的值表示如下：

	array($name, $mode, $description, $defaultValue)

参数 `mode` 可以是下列其中一项： `InputArgument::REQUIRED` 或 `InputArgument::OPTIONAL`.

当定义 `options` 时，该数组对应的值表示如下：

	array($name, $shortcut, $mode, $description, $defaultValue)

对选项而言, 参数 `mode` 可以是下列其中一项： `InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE`.

该 `VALUE_IS_ARRAY` 模式表示调用命令时可以传入多个值：

	php artisan foo --option=bar --option=baz

该 `VALUE_NONE` 模式表示将选项当作是"开关"

	php artisan foo --option

### 取得输入

当您的命令执行时，您需要让您的应用程序可以获取到这些参数和选项的值，
要做到这一点，您可以使用 `argument` 和 `option` 方法：

#### 取得自定义命令的输入参数

	$value = $this->argument('name');

#### 取得所有自定义命令的输入参数

	$arguments = $this->argument();

#### 取得自定义命令的输入选项

	$value = $this->option('name');

#### 取得所有自定义命令的输入选项

	$options = $this->option();

### 产生输出

显示信息到终端上，您可以使用 `info`, `comment`, `question` 和`error`方法，每一种方法将会对应到一个 ANSI 颜色。

#### 显示信息到终端

	$this->info('Display this on the screen');

#### 显示错误信息到终端

	$this->error('Something went wrong!');

### 询问式输入

您也可以使用 `ask` 和 `confirm` 方法来提示用户进行输入：

#### 提示用户进行输入

	$name = $this->ask('What is your name?');

#### 提示用户进行加密输入

	$password = $this->secret('What is the password?');

#### 提示用户进行确认

	if ($this->confirm('Do you wish to continue? [yes|no]'))
	{
		//
	}

您也可以指定一个默认值给 `confirm` 方法，可以是 `true` 或 `false`：

	$this->confirm($question, true);

<a name="registering-commands"></a>
## 注册命令

#### 注册一个 Artisan 命令

当您的自定义命令完成后，您需要向 Artisan 注册才能使用，通常是在 `app/start/artisan.php` ，在此文件内，您可以使用 `Artisan::add` 方法注册该命令：

	Artisan::add(new CustomCommand);

#### 在 IoC Container 内注册命令

如果您的自定义命令是在应用程序 [IoC container](/docs/4.2/ioc) 内注册，您需要使用 `Artisan::resolve` 方法让 Artisan 可以使用：

	Artisan::resolve('binding.name');

#### 在 Service Provider 内注册命令

如果您需要从 service provider 注册命令，您应该在 provider 的  `boot` 方法内调用 `commands` 方法，传入 [IoC container](/docs/4.2/ioc) 绑定此命令：

	public function boot()
	{
		$this->commands('command.binding');
	}

<a name="calling-other-commands"></a>
## 调用其它命令

有时候您可能希望在您的命令内部调用其它命令，此时您可以使用 `call` 方法：

	$this->call('command:name', array('argument' => 'foo', '--option' => 'bar'));
