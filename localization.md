# 本地化

- [介绍](#introduction)
- [语言文件](#language-files)
- [基本用法](#basic-usage)
- [复数](#pluralization)
- [验证本地化](#validation)
- [重写扩展包的语言文件](#overriding-package-language-files)

<a name="introduction"></a>
## 介绍

Laravel 的 `Lang` 类提供方便的方法来取得多种语言的字串，让您简单地在应用程序里支持多种语言。

<a name="language-files"></a>
## 语言文件

语言字串储存在 `app/lang` 文件夹的文件里。 在这个文件夹里应该要有子文件夹给每一个应用程序支持的语言。

	/app
		/lang
			/en
				messages.php
			/es
				messages.php

#### 语言文件例子

语言文件简单地返回键跟字串的数组。例如：

	<?php

	return array(
		'welcome' => 'Welcome to our application'
	);

#### 在执行时变换默认语言

应用程序的默认语言被储存在 `app/config/app.php` 配置文件。 您可以在任何时候用 `App::setLocale` 方法变换现行语言：

	App::setLocale('es');

#### 设定备用语言

您也可以设定 "备用语言"，它将会在当现行语言没有给定的语句时被使用。 就像默认语言，备用语言也可以在 `app/config/app.php` 配置文件设定：

	'fallback_locale' => 'en',

<a name="basic-usage"></a>
## 基本用法

#### 从语言文件取得句子

	echo Lang::get('messages.welcome');

 传递给 `get` 方法的字串的第一个部分是语言文件的名称，第二个部分是应该被取得的句子的名称。

> **备注**: 如果句子不存在， `get` 方法将会返回键的名称。

您也可以使用 `trans` 辅助方法，它是 `Lang::get` 方法的别名。

	echo trans('messages.welcome');

#### 在句子中做替代

您也可以在语言文件中定义占位符：

	'welcome' => 'Welcome, :name',

接着，传递替代用的第二个参数给 `Lang::get` 方法：

	echo Lang::get('messages.welcome', array('name' => 'Dayle'));

#### 判断语言文件是否有句子

	if (Lang::has('messages.welcome'))
	{
		//
	}

<a name="pluralization"></a>
## 复数

复数是个复杂的问题，不同语言对于复数有很多种复杂的规则。 您可以简单地在您的语言文件里管理它。 您可以用 "管道" 字串区分字串的单数和复数形态：

	'apples' => 'There is one apple|There are many apples',

接着您可以用 `Lang::choice` 方法取得语句：

	echo Lang::choice('messages.apples', 10);

您也可以提供一个地区参数来指定语言。 举个例，如果您想要使用俄语 (ru)：

	echo Lang::choice('товар|товара|товаров', $count, array(), 'ru');

因为 Laravel 的翻译器继承了 Symfony 翻译组件，您也可以很容易地建立更明确的复数规则：

	'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',


<a name="validation"></a>
## 验证

要验证本地化的错误和信息，可以看一下 <a href="/docs/4.2/validation#localization">验证的文件</a>.

<a name="overriding-package-language-files"></a>
## 重写扩展包的语言文件

许多扩展包附带它们自有的语句。 您可以通过放置文件在 `app/lang/packages/{locale}/{package}` 文件夹重写它们，而不是改变扩展包的核心文件来调整这些句子。 所以，举个例子，如果您需要重写 `skyrim/hearthfire` 扩展包在 `messages.php` 的英文语句 ，您可以放置语言文件在： `app/lang/packages/en/hearthfire/messages.php`。 您可以只定义您想要重写的语句在这个文件里，任何您没有重写的语句将会仍从扩展包的语言文件载入。
