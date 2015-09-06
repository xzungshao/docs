# Service Container / 服务容器

- [Introduction / 简介](#introduction)
- [Binding / 绑定（粘合）](#binding)
    - [Binding Interfaces To Implementations / 绑定接口到实现 ](#binding-interfaces-to-implementations)
    - [Contextual Binding / 上下文绑定](#contextual-binding)
    - [Tagging / 加标记](#tagging)
- [Resolving / 解析](#resolving)
- [Container Events / 容器事件](#container-events)

ps : ioc 容器的作用本身就似粘合剂 ， 因此这里的binding理解为粘合更形象些

<a name="introduction / 简介"></a>
## Introduction

The Laravel service container is a powerful tool for managing class dependencies and performing dependency injection. Dependency injection is a fancy phrase that essentially means this: class dependencies are "injected" into the class via the constructor or, in some cases, "setter" methods.

laravel 服务容器是一个用于管理类的依赖性和实现依赖注入的强大工具。依赖注入是一个华丽的用词，它本质上的意思是： 类的依赖被注入进类里，是通过构造器（一般是只构造函数），或者， 某些情况下，“setter” 方法来完成的。

Let's look at a simple example:

让我们来看一个简单的例子：

    <?php

    namespace App\Jobs;

    use App\User;
    use Illuminate\Contracts\Mail\Mailer;
    use Illuminate\Contracts\Bus\SelfHandling;

    class PurchasePodcast implements SelfHandling
    {
        /**
         * The mailer implementation.
         */
        protected $mailer;

        /**
         * Create a new instance.
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function __construct(Mailer $mailer)
        {
            $this->mailer = $mailer;
        }

        /**
         * Purchase a podcast.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

In this example, the `PurchasePodcast` job needs to send e-mails when a podcast is purchased. So, we will **inject** a service that is able to send e-mails. Since the service is injected, we are able to easily swap it out with another implementation. We are also able to easily "mock", or create a dummy implementation of the mailer when testing our application.

在这个例子中，当播客被购买时， PurchasePodcast命令处理器需要发送一封电子邮件。所以，我们将注入一个服务来提供这个能力。当这个服务被注入以后，我们就可以轻易地切换到不同的实现(接口的换装功能，参考：contracts)。当测试我们的应用程序时，我们同样也可以轻易地「模拟」，或者创建一个虚拟的发信服务实现，来帮助我们进行测试。

A deep understanding of the Laravel service container is essential to building a powerful, large application, as well as for contributing to the Laravel core itself.

对于去构建一个大型的功能强大的应用以及为laravel 核心做贡献，深入理解laravel 服务容器是很有必要的。

<a name="binding / 绑定"></a>
## Binding

绑定

Almost all of your service container bindings will be registered within [service providers](/docs/{{version}}/providers), so all of these examples will demonstrate using the container in that context. However, there is no need to bind classes into the container if they do not depend on any interfaces. The container does not need to be instructed how to build these objects, since it can automatically resolve such "concrete" objects using PHP's reflection services.

几乎你的所有服务容器的绑定将被注册到服务提供者里（/docs/{{version}}/providers），因此，在这种情况下，所有这些例子将表明正在使用容器。 然而，如果他们不依赖任何接口，就不需要去绑定类到容器。容器不需要被指示去怎样构建这些对象，因为它能够使用php的反射服务自动解析实际“存在”的对象。

Within a service provider, you always have access to the container via the `$this->app` instance variable. We can register a binding using the `bind` method, passing the class or interface name that we wish to register along with a `Closure` that returns an instance of the class:

在一个服务提供者里，你总是可以通过‘$this->app’ 实例变量连接到容器。我们能够利用‘bind’方法注册一个绑定，

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app['HttpClient']);
    });

Notice that we receive the container itself as an argument to the resolver. We can then use the container to resolve sub-dependencies of the object we are building.

注意到我们接收容器本身作为一个参数传递给解析器（即上面匿名函数的参数：$app）。当我们正在构建的同时，我们能够使用容器去解析对象的子依赖。

#### Binding A Singleton

绑定单例

The `singleton` method binds a class or interface into the container that should only be resolved one time, and then that same instance will be returned on subsequent calls into the container:

'singleton'  方法绑定一个类或接口到容器，这个方法仅仅被解析一次；然后，在容器里随后的调用，都将会返回相同的实例。

    $this->app->singleton('FooBar', function ($app) {
        return new FooBar($app['SomethingElse']);
    });

#### Binding Instances

绑定实例

You may also bind an existing object instance into the container using the `instance` method. The given instance will always be returned on subsequent calls into the container:

你也可能利用'instance' 方法绑定一个存在的对象实例到容器里。这个被给定的实例将总是被返回，在容器里随后的调用中。

    $fooBar = new FooBar(new SomethingElse);

    $this->app->instance('FooBar', $fooBar);

<a name="binding-interfaces-to-implementations"></a>
### Binding Interfaces To Implementations

绑定接口到实现

A very powerful feature of the service container is its ability to bind an interface to a given implementation. For example, let's assume we have an `EventPusher` interface and a `RedisEventPusher` implementation. Once we have coded our `RedisEventPusher` implementation of this interface, we can register it with the service container like so:

服务容器一个非常强大的功能是它能够绑定一个接口到（被给）实现；例如，我们假定有一个'EventPusher'接口和一个'RedisEventPusher'实现。一旦，我们编写完这个接口的'RedisEventPusher'实现，我们就能在服务容器里像下面这样注册它：

    $this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');

ps: 绑定接口到实现，就是接口的‘换装’特征，即将接口和该接口的实现进行绑定，可参考：http://laravelbase.com/posts/12

This tells the container that it should inject the `RedisEventPusher` when a class needs an implementation of `EventPusher`. Now we can type-hint the `EventPusher` interface in a constructor, or any other location where dependencies are injected by the service container:

当一个类需要一个'EventPusher' 的实现的时候，这将告诉容器应该注入'RedisEventPusher'（实现）。现在，我们可以在一个构造器（构造函数）中类型提示 'EventPusher'接口 ，或者在任何其他通过服务容器注入依赖的地方。
    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### Contextual Binding

上下文的绑定

Sometimes you may have two classes that utilize the same interface, but you wish to inject different implementations into each class. For example, when our system receives a new Order, we may want to send an event via [PubNub](http://www.pubnub.com/) rather than Pusher. Laravel provides a simple, fluent interface for defining this behavior:

有些时候，你可能有两个使用相同接口的类，但是你希望在这两个类里注入不同的实现。例如，当我们的系统接收到一个新订单，我们可能想通过[PubNub]发送一个事件而不是Pusher. laravel 为规范这个行为，提供了一个简洁，流畅的接口：

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give('App\Services\PubNubEventPusher');

You may even pass a Closure to the `give` method:

你甚至可以传递一个匿名函数给'give' 方法：

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give(function () {
                      // Resolve dependency...
                  });

<a name="tagging"></a>
### Tagging

标记

Occasionally, you may need to resolve all of a certain "category" of binding. For example, perhaps you are building a report aggregator that receives an array of many different `Report` interface implementations. After registering the `Report` implementations, you can assign them a tag using the `tag` method:

偶尔，你可能需要去解绑定中的某个“类别”。例如，假定你正在构建一个报告汇总器，它接收一个包含很多不同‘Report’接口实现的数组。在注册（这些）'Report'实现后，你能利用 'tag' 方法指派给他们一个tag。

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Once the services have been tagged, you may easily resolve them all via the `tagged` method:
这时服务已经被标记，你能够通过'tagged'方法很容易的去解析他们所有。

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });
    
ps :  $app->tagged('reports') 应该返回一个数组。

<a name="resolving"></a>
## Resolving

解析

There are several ways to resolve something out of the container. First, you may use the `make` method, which accepts the name of the class or interface you wish to resolve:

从容器解析出实例有几种方法。 首先，你可以使用'make' 方法，它接受你希望去解析的类或接口的名称：

    $fooBar = $this->app->make('FooBar');

Secondly, you may access the container like an array, since it implements PHP's `ArrayAccess` interface:

第二，你可以像一个数组一样使用容器，因为它（容器）实现了 PHP的'ArrayAccess'接口：

    $fooBar = $this->app['FooBar'];

Lastly, but most importantly, you may simply "type-hint" the dependency in the constructor of a class that is resolved by the container, including [controllers](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [queue jobs](/docs/{{version}}/queues), [middleware](/docs/{{version}}/middleware), and more. In practice, this is how most of your objects are resolved by the container.

最后，但是是最重要的，你可以在一个被容器解析过的类的构造器中简单使用类型提示指明依赖，这些类包括[controller](/docs/{{version}}/controllers),[event listeners](/docs/{{version}}/events),[queue jobs](/docs/{{version}}/queues),[middleware] (/docs/{{version}}/middleware), 等等。 在实践中，这是你的对象被容器解析的大多数方式。

The container will automatically inject dependencies for the classes it resolves. For example, you may type-hint a repository defined by your application in a controller's constructor. The repository will automatically be resolved and injected into the class:

对于它解析的类，容器将自动注入依赖。例如，你可以在一个controller的构造器中类型提示一个已经在你的应用中被定义的库。这个库将自动被解析并被注入进这个类（controller）:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## Container Events

容器事件

The service container fires an event each time it resolves an object. You may listen to this event using the `resolving` method:

服务容器每次解析一个对象会触发一个事件。你可以利用‘resolving’方法监听这事件：

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

    $this->app->resolving(function (FooBar $fooBar, $app) {
        // Called when container resolves objects of type "FooBar"...
    });

As you can see, the object being resolved will be passed to the callback, allowing you to set any additional properties on the object before it is given to its consumer.

就像你看到的那样，被解析的对象将被传递给回调函数，在它被交付给他的用户前，容许你去设置任何附加的属性到这个对象上。
