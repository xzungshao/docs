# 模板

- [控制器版面布局](#controller-layouts)
- [Blade 模板](#blade-templating)
- [其他 Blade 控制语法结构](#other-blade-control-structures)
- [扩展 Blade](#extending-blade)

<a name="controller-layouts"></a>
## 控制器版面布局

在 Laravel 中使用模板的一种方式就是利用控制器中的 `layout` 属性。通过指定控制器中的 `layout` 属性值，被指定使用的视图将会被自动创建出来并作出相应的界面回应。

#### 在控制器中定义一个版面布局

	class UserController extends BaseController {

		/**
		 * The layout that should be used for responses.
		 */
		protected $layout = 'layouts.master';

		/**
		 * Show the user profile.
		 */
		public function showProfile()
		{
			$this->layout->content = View::make('user.profile');
		}

	}

<a name="blade-templating"></a>
## Blade 模板

Blade 是 Laravel 所提供的一个简单却又非常强大的模板引擎。Blade 是使用 _模板继承（template inheritance）_ 及 _区块（sections）_ 来创建出视图。所有的 Blade 模板的后缀名都要命名为 `.blade.php`。

#### 定义一个 Blade 版面布局

	<!-- Stored in app/views/layouts/master.blade.php -->

	<html>
		<body>
			@section('sidebar')
				This is the master sidebar.
			@show

			<div class="container">
				@yield('content')
			</div>
		</body>
	</html>

#### 在视图模板中使用 Blade 版面布局

	@extends('layouts.master')

	@section('sidebar')
		@parent

		<p>This is appended to the master sidebar.</p>
	@stop

	@section('content')
		<p>This is my body content.</p>
	@stop

请注意在视图中 `extend` 一个 Blade 版面布局会直接将版面布局中定义的区块部分用视图的区块部份重写掉。如果想要将版面布局中的区块内容也能在继承此版面布局的视图中呈现，那就要在区块中使用 `@parent` 语法，这样就可以让您将视图的区块内容附加到版面布局区块的内容，而不会是用重写的方式。我们可能会在侧边栏区块或是页尾区块使用类似的技巧。

有时候，如您不确定这个区块内容有没有被定义，您可能会想要传一个默认的内容给 `@yield`。您可以使用第二个参数传入一个默认值给 `@yield`：

	@yield('section', 'Default Content')

<a name="other-blade-control-structures"></a>
## 其他 Blade 控制语法结构

#### 在 Blade 视图中打印（Echoing）数据

	Hello, {{{ $name }}}.

	The current UNIX timestamp is {{{ time() }}}.

#### 检查数据是否存在后再打印数据

有时候您想要打印一个变量，但您不确定这个变量是否存在，基本上，您会想要这样写：

	{{{ isset($name) ? $name : 'Default' }}}

然而，除了写这种三元运算符语法之外，Blade 让您可以使用下面这种更简便的语法：

	{{{ $name or 'Default' }}}

#### 使用花括号显示文字

如果您需要显示的一串字符串刚好被花刮号包起来，您可以在花刮号之前加上 `@` 符号前缀来跳出 Blade 引擎的解析：

	@{{ This will not be processed by Blade }}

当然，所有用户所提供的数据应该都要被处理过以避免一些安全性问题。所以您可以使用三重花刮号来避免用户输入的危险字串：

	Hello, {{{ $name }}}.

如果您希望打印用户输入的原生文字而不希望跳脱这些字串，您可以使用双重花刮号来打印原生文字：

	Hello, {{ $name }}.

> **特别注意：** 在您的应用程序打印用户所提供的内容时要非常小心！请记得永远使用三重花刮号的语法来过滤内容中的 HTML 字串实体。

#### If 叙述语法

	@if (count($records) === 1)
		I have one record!
	@elseif (count($records) > 1)
		I have multiple records!
	@else
		I don't have any records!
	@endif

	@unless (Auth::check())
		You are not signed in.
	@endunless

#### 循环

	@for ($i = 0; $i < 10; $i++)
		The current value is {{ $i }}
	@endfor

	@foreach ($users as $user)
		<p>This is user {{ $user->id }}</p>
	@endforeach

	@forelse($users as $user)
		<li>{{ $user->name }}</li>
	@empty
		<p>No users</p>
	@endforelse

	@while (true)
		<p>I'm looping forever.</p>
	@endwhile

#### 包含子视图

	@include('view.name')

您可以利用传数组的方式将数据传给包含进来的子视图：

	@include('view.name', array('some'=>'data'))

#### 重写区块

在默认的情况中，区块的内容会附加进去之前的同一个区块内容里，如果想要重写掉前面区块中的内容，您可以使用 `overwrite` 叙述语法：

	@extends('list.item.container')

	@section('list.item.content')
		<p>This is an item of type {{ $item->type }}</p>
	@overwrite

#### 显示语言行

	@lang('language.line')

	@choice('language.line', 1)

#### 注释

	{{-- This comment will not be in the rendered HTML --}}

<a name="extending-blade"></a>
## 扩展 Blade

Blade 甚至让您可以定义自己的控制语法结构。当一个 Blade 文件被编译时，每一个自定义扩展语法会与视图内容一起被调用，让您可以做任何如单纯的 `str_replace` 操作至更为复杂的正规表达式操作。

Blade 的编译器带有一些辅助方法 `createMatcher` 及 `createPlainMatcher`，这些辅助方法可以产生您需要的表达式帮助您建制自己的自定义扩展语法。

其中 `createPlainMatcher` 方法是用在没有参数的语法命令如 `@endif` 及 `@stop` 等，而 `createMatcher` 方法是用在有参数的语法命令。

下面的例子创建了一个 `@datetime($var)` 语法命令，这个命令单纯是让 `$var` 调用 `->format()`：

	Blade::extend(function($view, $compiler)
	{
		$pattern = $compiler->createMatcher('datetime');

		return preg_replace($pattern, '$1<?php echo $2->format(\'m/d/Y H:i\'); ?>', $view);
	});
