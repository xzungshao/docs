# Request 生命周期

- [概览](#overview)
- [Request 生命周期](#request-lifecycle)
- [启动文件](#start-files)
- [应用程序事件](#application-events)

<a name="overview"></a>
## 概览

在“真实世界”里使用任何工具，只要您了解其运作原理，您会觉得很有底气。应用程序开发上也是相同的。当您了解开发工具的功能，您使用上会更得心应手。而这份文档的目的就是要给予您一个优良、高层次的Laravel 框架运作概览。随着了解整个框架越多，框架的组件和功能就不会显得那么“神秘”，开发应用也会更加得心应手。除了关于 request 生命周期的高层次概览外，我们也会介绍"启动"文件和应用程序事件。

如果您对许多东西不明白，不用担心，只要了解其基本原理即可，随着探索的深入您将会得到更多的知识，渐渐能自动的查漏补缺。

<a name="request-lifecycle"></a>
## 请求的生命周期

发送给应用程序的所有请求，都会被引导至 `public/index.php` 脚本里处理。如果使用 Apache，Laravel 包含的 `.htaccess` 文件会将所有的请求都转发给 `index.php` 处理。从这里开始，Laravel 就会进行处理请求跟回应给客户端的流程。了解 Laravel 引导过程的总体思路是有帮助的。

至此，了解 Laravel 的引导过程的所需掌握的重要概念就是**服务提供商**。您可以在您的 `app/config/app.php` 配置文件中找到 `providers` 的数组，这里列出所有的服务提供商。这些提供商是 Laravel 的主要引导机制。但在深入探讨服务提供商之前，我们先回到 `index.php`。在一个请求进入到您的 `index.php` 后，`bootstrap/start.php` 文件将接着被载入，这份文件会创建一个新的 Laravel `Application` 对象，并也作为一个[IoC 容器](/docs/4.2/ioc)。

在创建 `Application` 对象之后，框架会设置一部分路径信息并进行[环境检测](/docs/4.2/configuration#environment-configuration)。然后，一个内部的 Laravel 引导脚本将会被调用。这个文件深藏在 Laravel 的代码中，并且依据您的配置文件做一些相关设定，如：时区、错误报告等等。但除了这些设定与琐碎的配置选项外，他也做一些比较重要的事情：注册所有您应用程序中设定的服务提供商。

比较简单的服务提供商仅含有一个方法：`register`。此方法用于应用程序对象注册服务提供商时，应用程序会通过所属的 `register` 方法来调用服务提供商的 `register` 方法。此方法中，服务提供商注册[IoC 容器](/docs/4.2/ioc)。本质上来说，每个服务提供商会绑定一个以上容器中的[封包](http://us3.php.net/manual/en/functions.anonymous.php)，让您可以在应用程序中获取这些绑定服务。所以例如，`QueueServiceProvider` 注册封包来处理多个[队列](/docs/4.2/queues)相关类。当然，服务提供商注册可以被使用在任何引导任务上，不只是注册在容器中的事情。服务提供商亦可以注册事件监听器，view composers, artisan 命令等等。

在所有的事件提供者都被注册后，您的 `app/start` 文件将会被载入。最后，您的 `app/routes.php` 文件才会被载入。一旦您的 `routes.php` 文件被载入后，请求对象将会被发送至应用程序中，如此他才能被指派至其中某个路由中。

好，让我们来总结一下：

1. 请求进入 `public/index.php` 文件中。
2. `bootstrap/start.php` 文件创建应用程序和环境检测。
3. 内部 `framework/start.php` 文件配置设定和载入服务提供商。
4. 应用程序 `app/start` 文件被载入。
5. 应用程序 `app/routes.php` 文件被载入。
6. 请求对象传入应用程序中，回传响应对象。
7. 响应对象回传客户端。

现在您对于 Laravel 如何处理一个请求有应该个比较好的了解，让我们更近些了解 "起始文件"！

<a name="start-files"></a>
## 启动文件

您的应用程序的启动文件被储存在 `app/start` 目录下。默认下，三个文件将被包含在您的应用程序中：`global.php`, `local.php` 和 `artisan.php`。关于 `artisan.php` 更多的信息可以参考 [Artisan 命令行](/docs/4.2/commands#registering-commands)文件。

`global.php` 起始文件默认包含了一些基本数据，如 [日志](/docs/4.2/errors)的注册和加载您的 `app/filters.php` 文件。然而，您可以自由添加任何您想添加的内容。不管何种运行环境，他都会自动被包含进去在每个_请求_里。另一方面，`local.php` 文件仅会在 `local` 环境下才会被执行。更多关于运行环境的信息，请参照 [设定](/docs/4.2/configuration)文件。

当然，除了 `local` 外您还有其他的运行环境，您一样可以为这些运行环境创建起始文件。当应用程序运行在该运行环境时将会自动载入对应文件。所以例如，如果您在 `bootstrap/environment.php` 中设定了一个 `development` 运行环境，您可以创建 `app/start/development.php` 文件，当应用程序在该运行环境中接收请求时会自动载入该文件。

### 启动文件需要存放什么

启动文件作为一个简单的地方来放置任何“引导程序”。例如，您可以注册一个 View composer，配置您的日志喜好或是一些 PHP 设置等等。完全取决与您。当然，把您的所有的引导代码都丢在起始文件中也会造成混乱，建议将一些引导代码放置到[服务提供商](/docs/4.2/ioc#service-providers)中。

<a name="application-events"></a>
## 应用程序事件

#### 注册应用程序事件

您可以注册 `before`, `after`, `finish` 和 `shutdown` 应用程序事件，来对请求前或请求后做相应的处理：

	App::before(function($request)
	{
		//
	});

	App::after(function($request, $response)
	{
		//
	});

监听器将会对您的应用程序在每次请求执行 `before` 和 `after` 函数。这些事件对于全局筛选或是全局修改回应上很有帮助。您可以将他们注册在您的其中一个“起始文件”中或者是[服务提供商](/docs/4.2/ioc#service-providers)中。

您也可以注册一个监听器在 `matched` 的事件上，当传入的请求被匹配到一个路由，但该路由尚未被执行的情况下，该事件将会被触发：

	Route::matched(function($route, $request)
	{
		//
	});

`finish` 事件在您的应用程序响应给客户端之后被调用。应用程结束之前，在这儿做点什么是不错的选择。`shutdown` 在所有 `finish` 事件处理器都结束之后立即被调用，这是软件结束之前做点什么的最后机会。但在大部分的情况下，您用不到这些事件。
