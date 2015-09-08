# Contracts
合约
- [Introduction / 简介](#introduction)
- [Why Contracts? / 为什么合约？](#why-contracts)
- [Contract Reference / 合约参考 ](#contract-reference)
- [How To Use Contracts / 怎样去使用合约?](#how-to-use-contracts)

<a name="introduction"></a>
## Introduction

Laravel's Contracts are a set of interfaces that define the core services provided by the framework. For example, a `Illuminate\Contracts\Queue\Queue` contract defines the methods needed for queueing jobs, while the `Illuminate\Contracts\Mail\Mailer` contract defines the methods needed for sending e-mail.

laravel 的contracts 是由框架驱动的一组定义了核心服务的接口。例如，一个`Illuminate\Contracts\Queue\Queue` 合约定义了队列任务所需要的方法，而 `Illuminate\Contracts\Mail\Mailer` 合约定义了发送邮件所有需要的方法。

ps： contracts 其实就是接口。

Each contract has a corresponding implementation provided by the framework. For example, Laravel provides a queue implementation with a variety of drivers, and a mailer implementation that is powered by [SwiftMailer](http://swiftmailer.org/).

每个contract 都有一个由框架提供的相应实现。例如,laravel 提供了有多种驱动的队列实现 ，一个由[SwiftMailer] 提供的 mailer 实现。

All of the Laravel contracts live in [their own GitHub repository](https://github.com/illuminate/contracts). This provides a quick reference point for all available contracts, as well as a single, decoupled package that may be utilized by package developers.

所有的laravel  contracts  都放在他们各自的github库上，除了提供了所有可用的contracts一个快速的参考，也可以单独作为一个低耦合的扩展包让其他扩展包开发者使用。

### Contracts Vs. Facades


Laravel's [facades](/docs/{{version}}/facades) provide a simple way of utilizing Laravel's services without needing to type-hint and resolve contracts out of the service container. However, using contracts allows you to define explicit dependencies for your classes. For most applications, using a facade is just fine. However, if you really need the extra loose coupling that contracts can provide, keep reading!

laravel 的facades 提供了一种使用laravel 的服务的简单方法，这种方法不需要使用类型提示也不用在服务容器外对contracts解析。
然而，使用contracts 容许你为你的类定义清晰的依赖。对于大多数应用，使用facade是很不错的，然而，如果你确实需要附加松耦合，那么contracts 能够提供这项功能，请不要走开！

<a name="why-contracts"></a>
## Why Contracts?

为什么是 contracts?

You may have several questions regarding contracts. Why use interfaces at all? Isn't using interfaces more complicated? Let's distil the reasons for using interfaces to the following headings: loose coupling and simplicity.

你可能有一些关于contracts的问题。为什么都要使用接口呢？使用接口不是更难以理解吗？下面的标题是一个使用接口的精简理由： 松散耦合和简洁

### Loose Coupling

松散耦合

First, let's review some code that is tightly coupled to a cache implementation. Consider the following:

首先，我们来回顾一些代码，代码中一个缓存实现是紧耦合。思考以下代码：

    <?php

    namespace App\Orders;

    class Repository
    {
        /**
         * The cache.
         */
        protected $cache;

        /**
         * Create a new repository instance.
         *
         * @param  \SomePackage\Cache\Memcached  $cache
         * @return void
         */
        public function __construct(\SomePackage\Cache\Memcached $cache)
        {
            $this->cache = $cache;
        }

        /**
         * Retrieve an Order by ID.
         *
         * @param  int  $id
         * @return Order
         */
        public function find($id)
        {
            if ($this->cache->has($id))    {
                //
            }
        }
    }

In this class, the code is tightly coupled to a given cache implementation. It is tightly coupled because we are depending on a concrete Cache class from a package vendor. If the API of that package changes our code must change as well.

在上面的类里，代码跟缓存实现之间是紧耦合。理由是它会依赖于扩展包库（ package vendor ）的特定缓存类。一旦这个扩展包的 API 更改了，我们的代码也要跟着改变。

Likewise, if we want to replace our underlying cache technology (Memcached) with another technology (Redis), we again will have to modify our repository. Our repository should not have so much knowledge regarding who is providing them data or how they are providing it.

同样的，如果我们想使用另一项技术（如：redis）去替换底层的缓存技术（Memcached）,我们将不得不再次修改我们的repository.我们的repository 不应该有太多关于是谁在提供他们数据或者他们是怎样提供的等细节。

**Instead of this approach, we can improve our code by depending on a simple, vendor agnostic interface:**
比起上面的做法，我们可以改用一个简单、和扩展包无关的接口来改进代码：
    <?php

    namespace App\Orders;

    use Illuminate\Contracts\Cache\Repository as Cache;

    class Repository
    {
        /**
         * Create a new repository instance.
         *
         * @param  Cache  $cache
         * @return void
         */
        public function __construct(Cache $cache)
        {
            $this->cache = $cache;
        }
    }
    
ps： 构造函数中的Cache 类型提示 指定一个cache的contracts(接口)，而不再像上面的例子中类型提示的是一个具体的类；我们随时可以通过‘换装’将不同的Cache 的实现(memcached or redis  or other)绑定到 Cache 上，因此这里我们总可以得到一个缓存对象（or 实例），不管它是memcached 还是redis  .

Now the code is not coupled to any specific vendor, or even Laravel. Since the contracts package contains no implementation and no dependencies, you may easily write an alternative implementation of any given contract, allowing you to replace your cache implementation without modifying any of your cache consuming code.

现在代码不和任何指定的vendor耦合，甚至 laravel . 由于contracts 即不包含实现也不包含依赖，你可以很容易的为任何给定的contracts编写一个可选的实现；同时容许你去替换你的缓存实现，在不修改任何你‘使用’缓存的代码的情况下。

### Simplicity

简洁性

When all of Laravel's services are neatly defined within simple interfaces, it is very easy to determine the functionality offered by a given service. **The contracts serve as succinct documentation to the framework's features.**

如果所有laravel 的服务都通过简单的接口作简单的定义，就很容易判定给定的服务提供的功能。** 针对框架的特征 contracts 可以充当简洁的文档**

In addition, when you depend on simple interfaces, your code is easier to understand and maintain. Rather than tracking down which methods are available to you within a large, complicated class, you can refer to a simple, clean interface.

此外，当你依赖简单的接口， 你的代码更容易去管理和维护。比起去追踪搜索一个大型复杂的类里有哪些可用的方法，你有一个简单，干净的接口可以参考。

<a name="contract-reference"></a>
## Contract Reference

contract 参考

This is a reference to most Laravel Contracts, as well as their Laravel "facade" counterparts:

这里是大大部分的laravel contracts ，以及他们对应的laravel "facade" 

ps:思考contract 和 facade 的关系？

Contract  |  References Facade
------------- | -------------
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/master/Broadcasting/Broadcaster.php)  | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="how-to-use-contracts"></a>
## How To Use Contracts

怎么去使用 contracts 呢？

So, how do you get an implementation of a contract? It's actually quite simple.
你怎么去获取一个contract 的实现呢？ 它非常简单。

Many types of classes in Laravel are resolved through the [service container](/docs/{{version}}/container), including controllers, event listeners, middleware, queued jobs, and even route Closures. So, to get an implementation of a contract, you can just "type-hint" the interface in the constructor of the class being resolved.

在laravel 中很多类型的类都是通过service container 被解析，包括控制器 ,时间监听器，中间件，队列任务，乃至路由闭包 。 因此，去获取一个contract的实现，你只要在能够被解析的类的构造器里“类型提示”接口就可以了。

For example, take a look at this event listener:
例如，看这个事件监听器：

    <?php

    namespace App\Listeners;

    use App\User;
    use App\Events\NewUserRegistered;
    use Illuminate\Contracts\Redis\Database;

    class CacheUserInformation
    {
        /**
         * The Redis database implementation.
         */
        protected $redis;

        /**
         * Create a new event handler instance.
         *
         * @param  Database  $redis
         * @return void
         */
        public function __construct(Database $redis)
        {
            $this->redis = $redis;
        }

        /**
         * Handle the event.
         *
         * @param  NewUserRegistered  $event
         * @return void
         */
        public function handle(NewUserRegistered $event)
        {
            //
        }
    }

When the event listener is resolved, the service container will read the type-hints on the constructor of the class, and inject the appropriate value. To learn more about registering things in the service container, check out [its documentation](/docs/{{version}}/container).

当事件监听器被解析，服务容器将在类的构造器中读类型提示，并注入恰当的值。要学习更多关于在服务容器注册东西，请切换到它的文档（/docs/{{version}}/container）
