# 更新记录

- [Laravel 4.2](#laravel-4.2)
- [Laravel 4.1](#laravel-4.1)

<a name="laravel-4.2"></a>
## Laravel 4.2

通过在4.2版本的安装目录下执行 `php artisan changes` 可以获取此次版本的完整更新列表，或者 查看[Github 上的更新记录](https://github.com/laravel/framework/blob/4.2/src/Illuminate/Foundation/changes.json)。此记录只含括当前版本的主要功能改进和变更。

> **附注:** 在 4.2 发布周期内，许多小的BUG修正与功能强化被整合至各个 4.1 的子发行版本中。所以最好确认 Laravel 4.1 版本的更新列表。 

### PHP 所需最低版本为 5.4

Laravel 4.2 需要 PHP 5.4 或者更高的版本。此 PHP 的更新版本让我们可以使用 PHP 的新特性：`traits` 来为如[Laravel Cashier](/docs/4.2/billing) 来提供更具表达力的接口。PHP 5.4 相比 PHP 5.3 ，在性能、速度、执行力效率上都有显著的提高。

### Laravel Forge

Larvel Forge是一个 Web 应用程序，它提供了一个简单的接口去建立、管理您云端上的 PHP 服务器，如 Linode, DigitalOcean, Rackspace 和 Amazon EC2。支持自动化 nginx 设定、SSH 密钥管理、Cron job 自动化， 通过 NewRelic 或者 Papertrail 进行服务器监控，"Push To Deploy", Laravel queue worker 设定等等。Forge 提供最简单且更实惠的方式来部署所有您的 Laravel 应用程序。

默认在 Laravel 4.2 的安装里， `app/config/database.php` 配置文件已为 Forge 设定完成，能让新应用在平台上的更方便的部署更加方便。

关于 Laravel Forge 的更多信息可以在[官方 Forge 网站](https://forge.laravel.com)上找到。

### Laravel Homestead

Laravel Homestead 是一个为开发强壮的 Laravel 和 PHP 应用程序的所提供的官方 Vagrant 环境，是一个离线的，预先打好包的`Vagrant box`。绝大多数的 `box` 依赖软件在发布前已经部署处理完成，这让`box`可以极快的被启用。Homestead 包含 Nginx 1.6, PHP 5.6, MySQL, Postres, Redis, Memcached, Beanstalk, Node, Gulp, Grunt 和 Bower。Homestead 包含一个简单的 `Homestead.yaml` 配置文件，让您在单一个封装包中能管理多个 Laravel 应用程序。

默认的 Laravel 4.2 安装中包含的 `app/config/local/database.php` 配置文件使用 Homestead 的数据库作为默认。让 Laravel 初始化安装与设定更为方便。

官方文件已经更新并包含在 [Homestead 文件](/docs/4.2/homestead)中。

### Laravel Cashier

Laravel Cashier是一个简单、具有表现力的资源库，用来管理 Stripe 的订阅帐务。虽然在安装中此组件是选用状态，我们仍将Cashier文件包含在主要 Laravel 文件中。此发布版本修复了多个错误、支持多货币还有支持最新的 Stripe API。

### Queue Workers 常驻软件

Artisan `queue:work` 命令现在支持 `--daemon` 参数让 worker 可以以"常驻软件"启用。代表 worker 可以持续的处理队列工作不需要重启框架。这让一个复杂的应用程序部署过程中，使得 CPU 的使用有显著的降低。

更多关于 Queue Workers 常驻软件信息请阅读 [queue 文件](/docs/4.2/queues#daemon-queue-worker)。

### Mail API Drivers

Laravel 4.2 为 `Mail` 函数采用了新的 Mailgun 和 Mandrill API 驱动。对许多应用程序而言，他提供了比 SMTP 更快也更可靠的方法来发送邮件。新的驱动使用了 Guzzle 4 HTTP 资源库。

### 软删除 Traits

对于软删除和全作用域更简洁的方案
PHP 5.4 的 `traits` 提供了一个更加简洁的软删除架构和全局作用域, 这些新架构为框架提供了更有扩展性的功能, 并且让框架更加简洁.

更多关于软删除的文档请见: [Eloquent documentation](/docs/4.2/eloquent#soft-deleting).

### 更为方便的 认证(auth) & Remindable Traits

得益于 PHP 5.4 traits , 我们有了一个更简洁的 用户认证 和 密码提醒接口, 这也让 `User` 模型文件更加精简.

### "简易分页"

一个新的 `simplePaginate` 方法已被加入到查询以及 Eloquent 查询器中。让您在分页视图中，使用简单的"上一页"和"下一页"链接查询更为高效。

### 迁移确认

在正式环境中，破坏性的迁移动作将会被再次确认。如果希望取消提示字串确认请使用 `--force` 参数。

<a name="laravel-4.1"></a>
## Laravel 4.1

### 完整更新列表

通过在4.1版本的安装目录下执行 `php artisan changes` 来获取此次版本的完整更新列表，或者 查看[Github 上的更新纪录](https://github.com/laravel/framework/blob/4.1/src/Illuminate/Foundation/changes.json)。此纪录只含括当前版本的主要功能改进和变更。

### 新的 SSH 组件

一个全新的 `SSH` 组件在此发行版本中登场。此功能让您可以轻易的 SSH 至远程服务器并执行命令。更多信息，可以参阅 [SSH 组件文件](/docs/4.2/ssh)。

新的 `php artisan tail` 命令就是使用这个新的 SSH 组件。更多的信息，请参阅 `tail` [命令集文件](http://laravel.com/docs/4.2/ssh#tailing-remote-logs)。

### Boris In Tinker

如果您的系统支持 [Boris REPL](https://github.com/d11wtq/boris)，`php artisan thinker` 命令将会使用到它。系统中也必须先行安装好 `readline` 和 `pcntl` 两个 PHP 扩展包。如果您没这些扩展包，从 4.0 之后将会使用到它。

### Eloquent 强化

Eloquent 新增了新的 `hasManyThrough` 关系链。想要了解更多，请参见 [Eloquent 文件](/docs/4.2/eloquent#has-many-through)。

一个新的 `whereHas` 方法也同时登场，他将允许[检索基于关系模型的约束](/docs/4.2/eloquent#querying-relations)。

### 数据库读写分离

Query Builder 和 Eloquent 目前通过数据库层，已经可以自动做到读写分离。更多的信息，请参考[文件](/docs/4.2/database#read-write-connections)。

### 队列优先级

支持通过在 `queue:listen` 后面传递一组以逗号分隔的列表来指定队列优先级。

### 失败队列工作处理

现在队列将会自动处理失败的工作，只要在 `queue:listen` 后加上 `--tries` 即可。更多的失败工作处理可以参见 [队列文件](/docs/4.2/queues#failed-jobs)。

### 缓存标签

缓存“区块”已经被“标签”取代。缓存标签允许您将多个“标签”指向同一个缓存对象，而且可以清空所有被指定某个标签的所有对象。更多使用缓存标签信息请见[缓存文件](/docs/4.2/cache#cache-tags)。

### 更具弹性的密码提醒

密码提醒引擎已经可以提供更强大的开发弹性，如：认证密码，显示状态信息等等。使用强化的密码提醒引擎，更多的信息[请参阅文件](/docs/4.2/security#password-reminders-and-reset)。

### 强化路由引擎

Laravel 4.1 拥有一个完全重新编写的路由层。API 一样不变。然而与 4.0 相比，速度快上 100%。整个引擎大幅的简化，且对于路由表达式的编译大大减少对 Symfony Routing 的依赖。

### 强化 Session 引擎

此发行版本中，我们亦发布了全新的 Session 引擎。如同路由增进的部分，新的 Session 曾更加简化且更快速。我们不再使用 Symfony 的 Session 处理工具，并且使用更简单、更容易维护的自定义解法。


### Doctrine DBAL

如果您有在您的迁移中使用到 `renameColumn`，之后您必须在 `composer.json` 里加入 `doctrine/dbal` 扩展包。此扩展包不再默认包含在 Laravel 之中。
