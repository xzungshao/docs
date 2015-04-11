# 分页

- [设置](#configuration)
- [使用](#usage)
- [加入分页链接](#appending-to-pagination-links)
- [转换至 JSON](#converting-to-json)
- [自定义表示器（Presenter）](#custom-presenters)

<a name="configuration"></a>
## 设置

在其他的框架中，实现分页是令人感到苦恼的事，但是 Laravel 能让它实现得很轻松。在 `app/config/view.php` 文件中有设置选项可以设定相关参数，`pagination` 选项需要指定用哪个视图来建立分页，而 Laravel 默认包含两种视图。

`pagination::slider` 视图将会基于现在的页面智能的显示「范围」的页数链接，`pagination::simple` 视图将仅显示「上一页」和「下一页」的按钮。**两种视图都兼容  Twitter Bootstrap 框架**

<a name="usage"></a>
## 使用

有几种方法可用来操作分页数据。最简单的是在查询构造器或 Eloquent 模型使用 `paginate` 方法。

#### 对数据库结果分页

	$users = DB::table('users')->paginate(15);

#### 对 Eloquent 模型分页

您也可以对 [Eloquent](/docs/4.2/eloquent) 模型分页：

	$allUsers = User::paginate(15);

	$someUsers = User::where('votes', '>', 100)->paginate(15);

传送给 `paginate` 方法的参数是您希望每页要显示的数据条数，只要您取得查询结果后，您可以在视图中显示，并使用 `links` 方法去建立分页链接：

	<div class="container">
		<?php foreach ($users as $user): ?>
			<?php echo $user->name; ?>
		<?php endforeach; ?>
	</div>

	<?php echo $users->links(); ?>

这就是所有建立分页系统的步骤了! 您会注意到我们还没有告知 Laravel 我们目前的页面是哪一页，放心，这个信息 Laravel 会自动帮您做好。

如果您想要指定自定义的视图来使用分页，您可以使用 `links` 方法：

	<?php echo $users->links('view.name'); ?>

您也可以通过以下方法获得额外的分页信息：

- `getCurrentPage`
- `getLastPage`
- `getPerPage`
- `getTotal`
- `getFrom`
- `getTo`
- `count`


#### 「简单分页」

如果您只是要在您的分页视图显示「上一页」和「下一页」链接，您可以使用 `simplePaginate`  方法来执行更高效率的搜寻。当您不需要精准的显示页码在视图上时，且数据集比较大时，这个方法非常有用：

	$someUsers = User::where('votes', '>', 100)->simplePaginate(15);

#### 手动建立分页

有的时候您可能会想要从数组中数据手动建立分页实例。您可以使用 `Paginator::make` 方法：

	$paginator = Paginator::make($items, $totalItems, $perPage);

#### 自定义分页 URL

您还可以通过 `setBaseUrl` 方法自定义使用的 URL：

	$users = User::paginate();

	$users->setBaseUrl('custom/url');

上面的例子将建立 URL，类似以下内容： http://example.com/custom/url?page=2

<a name="appending-to-pagination-links"></a>
## 加入分页链接

您可以使用 `appends` 方法增加搜寻字串到分页链接中：

	<?php echo $users->appends(array('sort' => 'votes'))->links(); ?>

这样会产生类似下列的链接：

	http://example.com/something?page=2&sort=votes

如果您想要将「哈希片段 hash」加到分页的 URL中，您可以使用 `fragment` 方法：

	<?php echo $users->fragment('foo')->links(); ?>

此方法调用后将产生 Url，看起来像这样：

	http://example.com/something?page=2#foo

<a name="converting-to-json"></a>
## 转换至 JSON

`Paginator` 类实现 `Illuminate\Support\Contracts\JsonableInterface` 接口的 `toJson` 公开方法。 由路由返回的值，您可以将 `Paginator` 实例传换成 JSON。JSON 表单的实例会包含一些「元」信息，例如 `total`, `current_page`, `last_page`, `from` , `to`。该实例数据将可通过在 JSON 数组中 `data` 的键取得。

<a name="custom-presenters"></a>
## 自定义表示器（Presenter）

默认的分页表示器是兼容 Bootstarp 的。不过，您也可以自定义表示器（presenter）

### 扩展抽象表示器（Presenter）

实现 `Illuminate\Pagination\Presenter` 类的抽象方法。Zurb Foundation 的表示器（presenter）例子如下：

    class ZurbPresenter extends Illuminate\Pagination\Presenter {

        public function getActivePageWrapper($text)
        {
            return '<li class="current"><a href="">'.$text.'</a></li>';
        }

        public function getDisabledTextWrapper($text)
        {
            return '<li class="unavailable"><a href="">'.$text.'</a></li>';
        }

        public function getPageLinkWrapper($url, $page, $rel = null)
        {
            return '<li><a href="'.$url.'">'.$page.'</a></li>';
        }

    }

### 使用自定义表示器（Presenter）

首先，在 `app/views` 建立新的视图，这将会作为您的自定义表示器（presenter）。并且用新的视图取代 `app/config/view.php` 的 `pagination` 设定。最后，类似下方的代码会放在您的自定义表示器（presenter）视图中。

    <ul class="pagination">
        <?php echo with(new ZurbPresenter($paginator))->render(); ?>
    </ul>
