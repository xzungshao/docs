# Laravel Cashier

- [介绍](#introduction)
- [配置文件](#configuration)
- [订购方案](#subscribing-to-a-plan)
- [免信用卡试用](#no-card-up-front)
- [订购转换](#swapping-subscriptions)
- [订购数量](#subscription-quantity)
- [取消订购](#cancelling-a-subscription)
- [恢复订购](#resuming-a-subscription)
- [确认订购状态](#checking-subscription-status)
- [处理交易失败](#handling-failed-payments)
- [处理其它 Stripe Webhooks](#handling-other-stripe-webhooks)
- [收据](#invoices)

<a name="introduction"></a>
## 介绍

Laravel Cashier 提供语义化，流畅的接口和 [Stripe](https://stripe.com) 的订购管理服务连接。它几乎处理了所有复杂的订购管理相关逻辑。除了基本的订购管理外， Cashier 还可以处理折扣券，订购转换，管理订购「数量」、服务有效期限，甚至产生收据的 PDF 。

<a name="configuration"></a>
## 配置文件

#### Composer

首先，把 Cashier 扩展包加到 `composer.json`：

	"laravel/cashier": "~2.0"

#### 服务提供者

接下来，在 `app` 配置文件注册 `Laravel\Cashier\CashierServiceProvider`。

#### 迁移

使用 Cashier 前，我们需要增加几个字段到数据库。别担心，您可以使用 `cashier:table` Artisan 命令，建立迁移文件来新增必要字段。例如，要增加字段到 users 数据库表，使用 `php artisan cashier:table users` 。建立完迁移文件后，只要执行 `migrate` 命令即可。

#### 设定模型

接下来，把 BillableTrait 和日期字段参数加到模型 (model) 里：

	use Laravel\Cashier\BillableTrait;
	use Laravel\Cashier\BillableInterface;

	class User extends Eloquent implements BillableInterface {

		use BillableTrait;

		protected $dates = ['trial_ends_at', 'subscription_ends_at'];

	}

#### Stripe Key

最后，在起始文件里加入 Stripe key：

	User::setStripeKey('stripe-key');

<a name="subscribing-to-a-plan"></a>
## 订购方案

当有了模型实例，您可以很简单的处理客户订购的 Stripe 里的方案：

	$user = User::find(1);

	$user->subscription('monthly')->create($creditCardToken);

如果您想在处理订购的时候使用折扣券，可以使用 `withCoupon` 方法：

	$user->subscription('monthly')
	     ->withCoupon('code')
	     ->create($creditCardToken);

`subscription` 方法会自动建立与 Stripe 的交易，以及将 Stripe customer ID 和其他相关帐款信息更新到数据库。如果您的方案有在 Stripe 设定试用期，试用到期时间也会自动记录起来。

如果您的方案有试用期间，但是**没有**在 Stripe 里设定，您必须在订购后手动储存试用到期时间。

	$user->trial_ends_at = Carbon::now()->addDays(14);

	$user->save();

### 自定义额外用户详细数据

如果您想自定义额外的顾客详细数据，您可以将数组作为 `create` 方法的第二个参数传入：

	$user->subscription('monthly')->create($creditCardToken, [
		'email' => $email, 'description' => 'Our First Customer'
	]);

想知道更多 Stripe 支持的额外字段，请阅读 Stripe 的线上文件 [建立客户](https://stripe.com/docs/api#create_customer).

<a name="no-card-up-front"></a>
## 免信用卡试用

如果您提供免信用卡试用服务，把 `cardUpFront` 属性设为 `false`：

	protected $cardUpFront = false;

建立帐号时，记得把试用到期时间记录起来：

	$user->trial_ends_at = Carbon::now()->addDays(14);

	$user->save();

<a name="swapping-subscriptions"></a>
## 订购转换

使用 `swap` 方法可以把用户转换到新的订购方案：

	$user->subscription('premium')->swap();

如果用户还在试用期间，试用服务会跟之前一样可用。如果订单有「数量限制」，也会和之前一样。

<a name="subscription-quantity"></a>
## 订购数量

有时候订购行为会跟「数量限制」有关。例如，您的应用程序可能会跟用户每个月收取 $10 元。您可以使用 `increment` 和 `decrement` 方法简单的调整订购数量：

	$user = User::find(1);

	$user->subscription()->increment();

	// Add five to the subscription's current quantity...
	$user->subscription()->increment(5);

	$user->subscription()->decrement();

	// Subtract five to the subscription's current quantity...
	$user->subscription()->decrement(5);

<a name="cancelling-a-subscription"></a>
## 取消订购

取消订购相当简单：

	$user->subscription()->cancel();

当客户取消订购时， Cashier 会自动更新数据库的 `subscription_ends_at` 字段。这个字段会被用来判断 `subscribed` 方法是否该回传 `false` 。例如，如果顾客在三月一号取消订购，但是服务可以使用到三月五号为止，那么 `subscribed` 方法在三月五号前都会传回 `true` 。

<a name="resuming-a-subscription"></a>
## 恢复订购

如果您想要恢复客户之前取消的订购，使用 `resume` 方法：

	$user->subscription('monthly')->resume($creditCardToken);

如果客户取消订购后，在服务过期前恢复，他们不用在当下付款。他们的订购服务会重新激活，而付款时间还是依据之前的付款周期。

<a name="checking-subscription-status"></a>
## 确认订购状态

要确认用户是否订购了服务，使用 `subscribed` 方法：

	if ($user->subscribed())
	{
		//
	}

`subscribed` 方法很适合用在路由过滤：

	Route::filter('subscribed', function()
	{
		if (Auth::user() && ! Auth::user()->subscribed())
		{
			return Redirect::to('billing');
		}
	});

您可以使用 `onTrial` 方法，确认用户是否还在试用期间：

	if ($user->onTrial())
	{
		//
	}

要确认用户是否曾经订购但是已经取消了服务，可已使用 `cancelled` 方法：

	if ($user->cancelled())
	{
		//
	}

您可能想确认用户是否已经取消订单，但是服务还没有到期。例如，如果用户在三月五号取消了订购，但是服务会到三月十号才过期。那么用户到三月十号前都是有效期间。注意， `subscribed` 方法在过期前都会回传 `true` 。

	if ($user->onGracePeriod())
	{
		//
	}

`everSubscribed` 方法可以用来确认用户是否订购过您的方案：

	if ($user->everSubscribed())
	{
		//
	}

`onPlan` 方法可以用方案 ID 来确认用户是否订购某方案：

	if ($user->onPlan('monthly'))
	{
		//
	}

<a name="handling-failed-payments"></a>
## 处理交易失败

如果顾客的信用卡过期了呢？无需担心，Cashier 包含了 Webhook 控制器，可以帮您简单的取消顾客的订单。只要把路由注册到控制器：

	Route::post('stripe/webhook', 'Laravel\Cashier\WebhookController@handleWebhook');

失败的交易会经由控制器捕捉并进行处理。控制器会进行至多三次再交易尝试，都失败后才会取消顾客的订单。上面的 `stripe/webhook` URI 只是一个例子，您必须使用设定在 Stripe 里的 URI 才行。

<a name="handling-other-stripe-webhooks"></a>
## 处理其它 Stripe Webhooks

如果您想要处理额外的 Stripe webhook 事件，可以继承 Webhook 控制器。您的方法名称要对应到 Cashier 预期的名称，尤其是方法名称应该使用 `handle` 前缀，后面接着您想要处理的 Stripe webhook 。例如，如果您想要处理 `invoice.payment_succeeded` webhook ，您应该增加一个 `handleInvoicePaymentSucceeded` 方法到控制器。

	class WebhookController extends Laravel\Cashier\WebhookController {

		public function handleInvoicePaymentSucceeded($payload)
		{
			// Handle The Event
		}

	}

> **注意** 除了更新您数据库里的订购信息以外， Webhook 控制器也可能会经由 Stripe API 取消您的订购。

<a name="invoices"></a>
## 收据

您可以很简单的经由 `invoices` 方法拿到客户的收据数组：

	$invoices = $user->invoices();

您可以使用这些辅助方法，列出收据的相关信息给客户看：

	{{ $invoice->id }}

	{{ $invoice->dateString() }}

	{{ $invoice->dollars() }}

使用 `downloadInvoice` 方法产生收据的 PDF 下载。看吧，它的调用是多么的简单：

	return $user->downloadInvoice($invoice->id, [
		'vendor'  => 'Your Company',
		'product' => 'Your Product',
	]);
