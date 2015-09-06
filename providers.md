# Service Providers / 服务提供者

- [Introduction / 简介](#introduction)
- [Writing Service Providers / 编写服务提供者](#writing-service-providers)
    - [The Register Method / 注册方法](#the-register-method)
    - [The Boot Method / 启动方法 ](#the-boot-method)
- [Registering Providers / 注册提供者](#registering-providers)
- [Deferred Providers / 延期提供者](#deferred-providers)

<a name="introduction / 简介"></a>
## Introduction
Service providers are the central place of all Laravel application bootstrapping. Your own application, as well as all of Laravel's core services are bootstrapped via service providers.

服务提供者是所有laravel应用引导载入的中心。你自己的应用和所有laravel 的核心服务一样通过服务提供者被引导载入。

But, what do we mean by "bootstrapped"? In general, we mean **registering** things, including registering service container bindings, event listeners, middleware, and even routes. Service providers are the central place to configure your application.

但是，我们做什么才算是被引导载入呢？通常，我们注册一些事情的含义，包括注册服务容器绑定，事件监听器，中间件，以及路由。
服务提供者是装配你的应用的中心。

ps: registering service container bindings 是laravel 一项很重要的核心的操作。

If you open the `config/app.php` file included with Laravel, you will see a `providers` array. These are all of the service provider classes that will be loaded for your application. Of course, many of them are "deferred" providers, meaning they will not be loaded on every request, but only when the services they provide are actually needed.

如果你打开laravel 自带的 ‘config/app.php’文件，你将看到一个‘provider’数组。这里所有的服务驱动类都将被加载到你的应用中去。当然
，他们中的很多是‘deferred’（服务）提供者，意味着他们不会在每个请求中都被加载，仅仅是在他们提供的服务在被需要的时候才会被加载。

In this overview you will learn how to write your own service providers and register them with your Laravel application.

在这篇概述性文章中，你将学习到怎样在你的laravel 应用中编写你自己的服务提供者并把他们注册到你的laravel 应用中。

<a name="writing-service-providers / 编写服务提供者"></a>
## Writing Service Providers

编写服务提供者

All service providers extend the `Illuminate\Support\ServiceProvider` class. This abstract class requires that you define at least one method on your provider: `register`. Within the `register` method, you should **only bind things into the [service container](/docs/{{version}}/container)**. You should never attempt to register any event listeners, routes, or any other piece of functionality within the `register` method. 

所有的服务提供者都继承自'Illuminate\Support\ServiceProvider'类。这个抽象类需要你在你的提供者（类）中至少定义一个'register'方法.在这个'register'方法内部，你应该**仅仅绑定事物到[服务容器]里(关于servicecontainer参考文档：/docs/{{version}}/container)**. 你绝对不要尝试在'register'方法里注册任何的事件监听器，路由，或者任何其他功能模块。

The Artisan CLI can easily generate a new provider via the `make:provider` command:

Artisan CLI 能够很容易通过'make:provider' 命令生成一个新的提供者（类）

    php artisan make:provider RiakServiceProvider

<a name="the-register-method / 注册方法"></a>
### The Register Method

register 方法

As mentioned previously, within the `register` method, you should only bind things into the [service container](/docs/{{version}}/container). You should never attempt to register any event listeners, routes, or any other piece of functionality within the `register` method. Otherwise, you may accidently use a service that is provided by a service provider which has not loaded yet.

正如前面提到的那样，在'register' 方法中，你应该仅仅绑定服务到[服务容器](关于服务容器可参考:/docs/{{version}}/container). 你绝不应该尝试在'register'方法里注册任何的事件监听器，路由，或任何其他的功能。否则，你可能会意外使用到一个由服务提供者提供但并没有被加载的服务。

ps: 一个service provider  可以提供多种服务。

Now, let's take a look at a basic service provider:

现在，让我们看看一个基础的服务提供者：

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton('Riak\Contracts\Connection', function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

This service provider only defines a `register` method, and uses that method to define an implementation of `Riak\Contracts\Connection` in the service container. If you don't understand how the service container works, check out [its documentation](/docs/{{version}}/container).

这个服务提供者仅仅定义了一个'register'方法，并且使用这个方法在服务容器里定义了一个'Riak\Contracts\Connection'（接口）的实现 （即返回了一个接口的实例）。如果你不理解服务容器是怎样工作的，请切换到文档这里：/docs/{{version}}/container

ps: 服务容器，应该就是ioc容器了； 是通过DI(依赖注入)实现的IOC吗？是的

<a name="the-boot-method"></a>
### The Boot Method

boot 方法

So, what if we need to register a view composer within our service provider? This should be done within the `boot` method. **This method is called after all other service providers have been registered**, meaning you have access to all other services that have been registered by the framework:

那么，假如我们需要去注册一个可视化的composer到我们的服务提供者该怎么办？ 这应该在'boot'方法里来做。**这个方法在所有其他的服务提供者都已经被注册后被调用**，意味着，你可以在这里使用所有其他经由框架被注册的服务。

ps ： 即 register 方法中注册的服务 ； 一个自定义的服务提供者可以同时包含register() 和 boot() 方法。

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

#### Boot Method Dependency Injection

boot 方法依赖注入

We are able to type-hint dependencies for our `boot` method. The [service container](/docs/{{version}}/container) will automatically inject any dependencies you need:

我们可以对'boot'方法中的依赖做类型提示 。 服务容器（/docs/{{version}}/container）将自动注入你需要的所有依赖。

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $factory)
    {
        $factory->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## Registering Providers

注册提供者

All service providers are registered in the `config/app.php` configuration file. This file contains a `providers` array where you can list the names of your service providers. By default, a set of Laravel core service providers are listed in this array. These providers bootstrap the core Laravel components, such as the mailer, queue, cache, and others.

所有的服务提供者都在'config/app.php'配置文件被注册。这个文件包含一个'providers'数组，你可以从这个数组里遍历你所有服务提供者的名称。默认，一组laravel 核心服务提供者被列在这个数组里。这些提供者引导加载核心的laravel组件，如邮件发送器，队列，缓存以及其他。

To register your provider, simply add it to the array:

注册你的提供者，简单的添加它到这个数组里：

    'providers' => [
        // Other Service Providers

        'App\Providers\AppServiceProvider',
    ],

<a name="deferred-providers"></a>
## Deferred Providers

延期提供者

If your provider is **only** registering bindings in the [service container](/docs/{{version}}/container), you may choose to defer its registration until one of the registered bindings is actually needed. Deferring the loading of such a provider will improve the performance of your application, since it is not loaded from the filesystem on every request.
如果你的（服务）提供者是仅仅（并不加载）注册绑定在服务容器（/docs/{{version}}/container）中。你可能选择推迟它的注册，除非注册绑定的服务提供者中有一个需要被用到。延期这样一个服务提供者的加载将有助于提高你的应用的性能，因为，它不必在每次请求的时候都从文件系统里加载。

To defer the loading of a provider, set the `defer` property to `true` and define a `provides` method. The `provides` method returns the service container bindings that the provider registers:

要延期一个（服务）提供者的加载，设置'defer' 属性为'true' 并且定义一个'provides' 方法。 'provides'方法应该返回服务容器为这个提供者注册的绑定。

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Indicates if loading of the provider is deferred.
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton('Riak\Contracts\Connection', function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return ['Riak\Contracts\Connection'];
        }

    }


Laravel compiles and stores a list of all of the services supplied by deferred service providers, along with the name of its service provider class. Then, only when you attempt to resolve one of these services does Laravel load the service provider.

Laravel 编译并保存所有由延缓服务提供者所提供的服务清单，以及其服务提供者的类名称。只有在当你尝试解析其中的服务时， Laravel 才会加载服务提供者。
