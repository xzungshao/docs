# 单元测试

- [介绍](#introduction)
- [定义并执行测试](#defining-and-running-tests)
- [测试环境](#test-environment)
- [在测试中调用路由](#calling-routes-from-tests)
- [模拟 Facades](#mocking-facades)
- [框架 Assertions](#framework-assertions)
- [辅助方法](#helper-methods)
- [重置应用程序](#refreshing-the-application)

<a name="introduction"></a>
## 介绍

Laravel 在创立时就有考虑到单元测试。事实上，它能立即使用被引入的 PHPUnit 做测试，而且默认已经为您的应用程序建立了 `phpunit.xml` 文件。 除了 PHPUnit 以外，Laravel 也利用 Symfony HttpKernel、 DomCrawler 和 BrowserKit 组件让您在测试的时候模拟一个网页浏览器来检查和处理您的视图。

在 `app/tests` 文件夹中有提供一个测试例子。在安装好新的 Laravel 应用程序之后，只要在命令行上执行 `phpunit` 来进行测试流程。

<a name="defining-and-running-tests"></a>
## 定义并执行测试

要建立一个测试案例，只要在 `app/tests` 文件夹建立新的测试文件。测试类必须继承自 `TestCase`，接着您可以如平常使用 PHPUnit 一般去定义测试方法。

#### 测试类例子

	class FooTest extends TestCase {

		public function testSomethingIsTrue()
		{
			$this->assertTrue(true);
		}

	}

您可以从命令行执行 `phpunit` 命令来执行应用程序的所有测试。

> **注意:** 如果您定义自己的 `setUp` 方法， 请记得调用 `parent::setUp`。

<a name="test-environment"></a>
## 测试环境

当执行单元测试的时候，Laravel 会自动将环境设置在 `testing` 中。另外 Laravel 会在测试环境中加载 `session` 和 `cache` 的配置文件，在测试环境中这两个驱动会被设定为 `array` (空数组)，表示在测试的时候不对 session 或 cache 数据进行持久化操作。必要时你也可以创建其他的测试环境。

<a name="calling-routes-from-tests"></a>
## 在测试中调用路由

#### 从单一测试中调用路由

您可以使用 `call` 方法方便地调用您的其中一个路由来进行测试:

	$response = $this->call('GET', 'user/profile');

	$response = $this->call($method, $uri, $parameters, $files, $server, $content);

接着您可以检查 `Illuminate\Http\Response` 对象:

	$this->assertEquals('Hello World', $response->getContent());

#### 在测试中调用控制器

您也可以在测试中调用控制器 :

	$response = $this->action('GET', 'HomeController@index');

	$response = $this->action('GET', 'UserController@profile', array('user' => 1));

`getContent` 方法会回传求值后的字符串内容. 如果您的路由回传一个 `View`, 您可以通过 `original` 属性获取它:

	$view = $response->original;

	$this->assertEquals('John', $view['name']);

您可以使用 `callSecure` 方法去调用 HTTPS 路由:

	$response = $this->callSecure('GET', 'foo/bar');

> **注意:** 在测试环境中， 路由过滤器是被禁用的。如果要启用它们，必须增加 `Route::enableFilters()` 到您的测试中。

### DOM Crawler

您也可以通过调用路由来取得 DOM Crawler 实例来检查内容：

	$crawler = $this->client->request('GET', '/');

	$this->assertTrue($this->client->getResponse()->isOk());

	$this->assertCount(1, $crawler->filter('h1:contains("Hello World!")'));

如需更多如何使用 DOM Crawler 的信息，请参考它的[官方文件](http://symfony.com/doc/master/components/dom_crawler.html).

<a name="mocking-facades"></a>
## 模拟 Facades

当测试的时候，您或许常会想要模拟调用 Laravel 静态 facade。举个例子，参考下面的控制器行为:

	public function getIndex()
	{
		Event::fire('foo', array('name' => 'Dayle'));

		return 'All done!';
	}

我们可以在 facade 上使用 `shouldReceive` 方法，模拟调用 `Event` 类，它将会回传一个 [Mockery](https://github.com/padraic/mockery) mock 对象实例。

#### 模拟 Facade

	public function testGetIndex()
	{
		Event::shouldReceive('fire')->once()->with('foo', array('name' => 'Dayle'));

		$this->call('GET', '/');
	}

> **注意:** 您不应该模拟 `Request` facade。当执行您的测试，应该传递想要的输入数据给 `call` 方法。

<a name="framework-assertions"></a>
## 框架 Assertions

Laravel 附带几个 `assert` 方法，能让测试更简单一点:

#### Assert 回应为 OK

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertResponseOk();
	}

#### Assert 回应状态码

	$this->assertResponseStatus(403);

#### Assert 回应为重定向

	$this->assertRedirectedTo('foo');

	$this->assertRedirectedToRoute('route.name');

	$this->assertRedirectedToAction('Controller@method');

#### Assert 回应带数据的视图

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertViewHas('name');
		$this->assertViewHas('age', $value);
	}

#### Assert 回应带数据的 Session

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertSessionHas('name');
		$this->assertSessionHas('age', $value);
	}

#### Assert 回应带错误信息的 Session

    public function testMethod()
    {
        $this->call('GET', '/');

        $this->assertSessionHasErrors();

        // Asserting the session has errors for a given key...
        $this->assertSessionHasErrors('name');

        // Asserting the session has errors for several keys...
        $this->assertSessionHasErrors(array('name', 'age'));
    }

#### Assert 回应带数据的旧输入内容

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertHasOldInput();
	}

<a name="helper-methods"></a>
## 辅助方法

`TestCase` 类包含几个辅助方法让应用程序的测试更为简单。

#### 从测试里设定和刷新 Sessions

	$this->session(['foo' => 'bar']);

	$this->flushSession();

#### 设定目前经过验证的用户

您可以使用 `be` 方法设定目前经过验证的用户:

	$user = new User(array('name' => 'John'));

	$this->be($user);

您可以从测试中使用 `seed` 方法重新填充您的数据库:

#### 从测试中重新填充数据库

	$this->seed();

	$this->seed('DatabaseSeeder');

更多建立填充数据的信息可以在文件的 [迁移与数据填充](/docs/4.2/migrations#database-seeding) 部分找到。

<a name="refreshing-the-application"></a>
## 重置应用程序

您可能已经知道，您可以通过 `$this->app` 在任何测试方法中获取您的 Laravel `应用程序本体` / IoC 容器。这个应用程序对象实例会在每个测试类里被重置。如果您希望在给定的方法中手动强制重置应用程序，则可以在测试方法使用 `refreshApplication` 方法。 这将会重置任何额外的绑定， 例如那些从测试案例执行开始被放到 IoC 容器的 mocks。
