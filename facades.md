# Facades
门面
- [Introduction \ 简介](#introduction)
- [Using Facades \ 使用facade](#using-facades)
- [Facade Class Reference \ facade class 参考](#facade-class-reference)

<a name="introduction"></a>
## Introduction

Facades provide a "static" interface to classes that are available in the application's [service container](/docs/{{version}}/container). Laravel ships with many facades, and you have probably been using them without even knowing it! Laravel "facades" serve as "static proxies" to underlying classes in the service container, providing the benefit of a terse, expressive syntax while maintaining more testability and flexibility than traditional static methods.

facades提供一个“静态”的接口, 为那些在application的服务容器中有效的类.laravel 附带很多的 facades , 并且你或许已经在不知觉中使用它们了！laravel "facades" 作为为服务容器中底层类的“静态代理”，它具有简洁，语法易表达的优点
相比传统静态方法，维护起来有更好的可测试性和灵活性。

ps： container ships  有集装箱货运船的意思

<a name="using-facades"></a>
## Using Facades

使用 facades

In the context of a Laravel application, a facade is a class that provides access to an object from the container. The machinery that makes this work is in the `Facade` class. Laravel's facades, and any custom facades you create, will extend the base `Illuminate\Support\Facades\Facade` class.

在一个laravel 应用环境中， 一个facade 是一个提供从容器访问对象的类。 Facade 类是让这个机制可以运作的原因。laravel 的facades,以及任何你自定义的facades ， 都将继承基础类：'Illuminate\Support\Facades\Facade'.


A facade class only needs to implement a single method: `getFacadeAccessor`. It's the `getFacadeAccessor` method's job to define what to resolve from the container. The `Facade` base class makes use of the `__callStatic()` magic-method to defer calls from your facade to the resolved object.

一个facade 类仅仅需要去实现一个方法：‘getFacadeAccessor’ . 'getFacadeAccessor' 方法的工作是定义要从容器解析什么。基类 Facade 利用 __callStatic() 魔术方法来从你的 facade 调用到解析出来的对象。

ps: facade 类似是在用户代码和服务容器间的传话，我们调用一个facade类，实际是告诉facade类：你要从服务容器了解析出指定的绑定的对象给我，如 ：
 $user = Cache::get('user:'.$id);
这行代码会调用 Cache facade 类‘联系’service container 实例化一个Cache类并返回这个实例，facade 类就是做这个联系工作的。
因此，上面这行代码和下面这行代码是等效的：
$value = $app->make('cache')->get('key');

In the example below, a call is made to the Laravel cache system. By glancing at this code, one might assume that the static method `get` is being called on the `Cache` class:

在下面的例子里，调用了laravel 缓存系统的缓存。 通过看这个代码，有人可能会认为静态方法 'get' 在'Cache' 类被调用：

    <?php

    namespace App\Http\Controllers;

    use Cache;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

Notice that near the top of the file we are "importing" the `Cache` facade. This facade serves as a proxy to accessing the underlying implementation of the `Illuminate\Contracts\Cache\Factory` interface. Any calls we make using the facade will be passed to the underlying instance of Laravel's cache service.

注意在文件的顶部附近，我们导入了 ‘Cache’ facade . 这个facade充当一个访问`Illuminate\Contracts\Cache\Factory` 接口底层实现的代理角色。我们使用的facade 任何调用都将通过laravel的缓存服务（service container 中已绑定该服务，Cache  facade 会返回一个该服务绑定到service container时所有使用的名称）的底层实例来担负。

ps : facade 是laravel 服务容器的一个便捷调用语法  或者说 Laravel Facades 是使用 Laravel 服务容器作为服务定位器的便捷语法。
     Cache::get('user:'.$id); 和 $app->make('Cache')->get('user:'.$id)  是一样的效果： 
     前者用到了facade ，后者直接使用的服务容器。

If we look at that `Illuminate\Support\Facades\Cache` class, you'll see that there is no static method `get`:

如果我们查看 `Illuminate\Support\Facades\Cache`  类，你将看到其实没有静态方法'get' :

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

Instead, the `Cache` facade extends the base `Facade` class and defines the method `getFacadeAccessor()`. Remember, this method's job is to return the name of a service container binding. When a user references any static method on the `Cache` facade, Laravel resolves the `cache` binding from the [service container](/docs/{{version}}/container) and runs the requested method (in this case, `get`) against that object.

替代的是，'Cache' facade 继承自基础'Facade' 类并且定义了方法 `getFacadeAccessor()` .  记住，这个方法的工作是返回（cache 服务在）服务容器绑定的名称。当一个用户在'Cache' facade 上引用任何静态方法 ，laravel就会从service container 解析'Cache'绑定并且对解析后生成的对象执行被请求的方法（当前情况下，'get'）。

<a name="facade-class-reference"></a>
## Facade Class Reference

Facade 类参考

Below you will find every facade and its underlying class. This is a useful tool for quickly digging into the API documentation for a given facade root. The [service container binding](/docs/{{version}}/container) key is also included where applicable.

下面你将找到每个facade以及他们的底层类。这是个可以从一个给定的 facade 根源快速地深入 API 文档的有用工具。可应用的 服务容器绑定关键字也包含在里面。

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Console\Application](http://laravel.com/api/{{version}}/Illuminate/Console/Application.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel.com/api/{{version}}/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Hash  |  [Illuminate\Contracts\Hashing\Hasher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Input  |  [Illuminate\Http\Request](http://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Password  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel.com/api/{{version}}/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |
Storage  |  [Illuminate\Contracts\Filesystem\Factory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) |
View  |  [Illuminate\View\Factory](http://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/{{version}}/Illuminate/View/View.html)  |
