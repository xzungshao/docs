# 表单与 HTML

- [开启表单](#opening-a-form)
- [CSRF 保护](#csrf-protection)
- [表单模型绑定](#form-model-binding)
- [标签（Label）](#labels)
- [文字字段（Text）、多行文字字段（Text Area）、密码字段（Password）和隐藏字段（Hidden Field）](#text)
- [核取方块（Checkboxe）和单选按钮（Radio Button）](#checkboxes-and-radio-buttons)
- [文件输入](#file-input)
- [下拉式选单](#drop-down-lists)
- [按钮](#buttons)
- [自定义宏（Macro）](#custom-macros)
- [产生 URL](#generating-urls)

<a name="opening-a-form"></a>
## 开启表单

#### 开启表单

	{{ Form::open(array('url' => 'foo/bar')) }}
		//
	{{ Form::close() }}

默认表单使用 POST 方法，当然您也可以指定传参其他表单的方法：

	echo Form::open(array('url' => 'foo/bar', 'method' => 'put'))

> **注意：** 因为 HTML 表单只支持 `POST` 和 `GET` 方法，所以在使用 `PUT` 及 `DELETE` 的方法时，Laravel 将会自动加入隐藏的 `_method` 字段到表单中，来伪装表单传送的方法。

您也可以建立指向命名的路由或控制器至表单：

	echo Form::open(array('route' => 'route.name'))

	echo Form::open(array('action' => 'Controller@method'))

您也可以传递路由参数：

	echo Form::open(array('route' => array('route.name', $user->id)))

	echo Form::open(array('action' => array('Controller@method', $user->id)))

如果您的表单允许上传文件，可以加入 `files` 选项到参数中:

	echo Form::open(array('url' => 'foo/bar', 'files' => true))

<a name="csrf-protection"></a>
## CSRF 保护

#### 添加 CSRF Token 到表单

Laravel 提供简易的方法，让您可以保护您的应用程序不受到 CSRF (跨网站请求伪造) 攻击。首先 Laravel 会自动在用户的 session 中放置随机的 token，如果你使用 `Form::open` 方法并使用 `POST`，`PUT` 或者 `DELETE`，这个 CSRF 参数会用隐藏字段的方式自动加到您的表单中，您也可以使用 `token` 的方法去产生这个隐藏的 CSRF token 字段：

	echo Form::token();

#### 附加 CSRF 过滤器到路由

	Route::post('profile', array('before' => 'csrf', function()
	{
		//
	}));

<a name="form-model-binding"></a>
## 表单模型绑定

#### 开启模型表单

您经常会想要将模型内容加入至表单中，您可以使用 `Form::model` 方法实现：

	echo Form::model($user, array('route' => array('user.update', $user->id)))

现在当您产生表单元素时，如 text 字段，模型的值将会自动比对到字段名称，并设定此字段值，举例来说，用户模型的 `email` 属性，将会设定到名称为 `email` 的 text 字段的字段值，不仅如此，当 Session 中有与字段名称相符的名称， Session 的值将会优先于模型的值，而优先顺序如下：

1. Session 的数据 (旧的输入值)
2. 明确传递的数据
3. 模型属性数据

这样可以允许您快速地建立表单，不仅是绑定模型数据，也可以在服务器端数据验证错误时，轻松的回填用户输入的旧数据！

> **注意：** 当使用 `Form::model` 方法时，必须确保有使用 `Form::close` 方法来关闭表单！

<a name="labels"></a>
## 标签（Label）

#### 产生标签（Label）元素

	echo Form::label('email', 'E-Mail Address');

#### 指定额外的 HTML 属性

	echo Form::label('email', 'E-Mail Address', array('class' => 'awesome'));

> **注意：** 在建立标签时，任何您建立的表单元素名称与标签相符时，将会自动在 ID 属性建立与标签名称相同的 ID。

<a name="text"></a>
## 文字字段、多行文字字段、密码字段,、隐藏字段

#### 产生文字字段

	echo Form::text('username');

#### 指定默认值

	echo Form::text('email', 'example@gmail.com');


> **注意：** *hidden* 和 *textarea* 方法和 *text* 方法使用属性参数是相同的。

#### 产生密码输入字段

	echo Form::password('password');

#### 产生其他输入字段

	echo Form::email($name, $value = null, $attributes = array());
	echo Form::file($name, $attributes = array());

<a name="checkboxes-and-radio-buttons"></a>
## 复选框和单选项钮

#### 产生复选框或单选按钮

	echo Form::checkbox('name', 'value');

	echo Form::radio('name', 'value');

#### 产生已选取的复选框或单选按钮

	echo Form::checkbox('name', 'value', true);

	echo Form::radio('name', 'value', true);

<a name="number"></a>
## Number

#### Generating A Number Input

	echo Form::number('name', 'value');

<a name="file-input"></a>
## 文件输入

#### 产生文件输入

	echo Form::file('image');

> **注意：**表单必须已将 `files` 选项设定为 `true`。

<a name="drop-down-lists"></a>
## 下拉式选单

#### 产生下拉式选单

	echo Form::select('size', array('L' => 'Large', 'S' => 'Small'));

#### 产生选择默认值的下拉式选单

	echo Form::select('size', array('L' => 'Large', 'S' => 'Small'), 'S');

#### 产生群组清单

	echo Form::select('animal', array(
		'Cats' => array('leopard' => 'Leopard'),
		'Dogs' => array('spaniel' => 'Spaniel'),
	));

#### 产生范围的下拉式选单

    echo Form::selectRange('number', 10, 20);

#### 产生月份名称的清单

    echo Form::selectMonth('month');

<a name="buttons"></a>
## 按钮

#### 产生提交按钮

	echo Form::submit('Click Me!');

> **注意：** 需要产生按钮（Button）元素吗? 可以试着使用 *button* 方法去产生按钮（Button）元素，button 方法与 *submit* 使用属性参数是相同的。

<a name="custom-macros"></a>
## 自定义宏

#### 注册表单宏

您可以轻松的定义您自己的表单类的辅助方法叫「宏（macros）」，首先只要注册 宏 ，并给预期名称及封闭函数：

	Form::macro('myField', function()
	{
		return '<input type="awesome">';
	});

现在您可以使用注册的名称调用您的宏：

#### 调用自定义表单宏

	echo Form::myField();


<a name="generating-urls"></a>
##产生 URL

更多产生 URL 的信息，请参阅文件[辅助方法](/docs/4.2/helpers#urls)。
