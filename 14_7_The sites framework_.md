

# The “sites” framework

Django 原生带有一个可选的“sites”框架。它是一个钩子，用于将对象和功能与特定的站点关联，它同时还是域名和你的Django 站点名称之间的对应关系所保存的地方。

如果你的Django 不只为一个站点提供支持，而且你需要区分这些不同的站点，你就可以使用它。

Sites 框架主要依据一个简单的模型：



_class_ `models.Site`



用来存储Web站点的`domain` ?和`name` 属性的模型



`domain`



与Web站点关联的域名。







`name`



Web 站点的名称。









[`SITE_ID`](../settings.html#std:setting-SITE_ID) 设置指定与特定的设置文件关联的[`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 对象在数据库中ID。如果省略该设置，[`get_current_site()`](#django.contrib.sites.shortcuts.get_current_site "django.contrib.sites.shortcuts.get_current_site") 函数将会通过比较[`domain`](#django.contrib.sites.models.Site.domain "django.contrib.sites.models.Site.domain") 与[`request.get_host()`](../request-response.html#django.http.HttpRequest.get_host "django.http.HttpRequest.get_host") 方法中得到的主机名，来得到当前的Site。

怎样使用取决于你，但是django自动的在几个方面通过一些简单的约定使用它。



## 示例

为什么要使用Sites 框架？通过例子能最好的解释。



### 关联内容到多个站点

通过Django开发的站点[LJWorld.com](http://www.ljworld.com/) 和[Lawrence.com](http://www.lawrence.com/) 是位于Lawrence, Kansas 的同一家机构Lawrence Journal-World newspaper 运营的。LJWorld.com 关注新闻，而Lawrence.com 关注当地的环境问题。但是有时编辑需要发布同一篇文章到_两个_站点。

无脑的解决方法是要求站点发布者发布同一内容两次：一次到LJWorld.com，一次到 Lawrence.com。但这是很低效的行为，而且在数据库中必须存储同一内容很多次（多副本存储，浪费资源）。

最好的解决方法很简单：两个站点用相同的文章数据库，一篇文章可以关联一个或者多个站点。用Django 模型的术语，它通过`Article` 模型的一个[`多对多字段`](../models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField")表示：





```
from django.db import models
from django.contrib.sites.models import Site

class Article(models.Model):
    headline = models.CharField(max_length=200)
    # ...
    sites = models.ManyToManyField(Site)

```





这很快很好的完成了几件事：

*   它使得站点编辑者利用一个接口(Django admin)编辑多站点上的所有内容。

*   它意味着同一个内容不用往数据库存入两次；在数据库中仅仅只有一条记录。

*   对于两个站点，开发者可以使用相同的Django 视图代码。显示内容的视图代码需要检查，以确保请求的内容属于当前的站点。就像下面一样:




```
from django.contrib.sites.shortcuts import get_current_site

    def article_detail(request, article_id):
        try:
            a = Article.objects.get(id=article_id, sites__id=get_current_site(request).id)
        except Article.DoesNotExist:
            raise Http404("Article does not exist on this site")
        # ...

```









### 关联内容到单独的站点

类似地，你可以用[`ForeignKey`](../models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey") 关联一个模型到[`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 模型实现多对一关系。

例如，一篇文章只允许在一个单独的站点，你应该像这样用模型：





```
from django.db import models
from django.contrib.sites.models import Site

class Article(models.Model):
    headline = models.CharField(max_length=200)
    # ...
    site = models.ForeignKey(Site)

```





这个好处和上节描述的好处是相同的。





### 在视图中获得当前的Site

你可以在Django 视图中使用Sites 框架基于正在调用的视图所在的Site 实现特定的功能。例如：





```
from django.conf import settings

def my_view(request):
    if settings.SITE_ID == 3:
        # Do something.
        pass
    else:
        # Do something else.
        pass

```





当然，这样硬编码Site ID 比较丑陋。这种硬编码是你最需要尽快修复的。完成这件事情的更清洁的方法是检查当前站点的域名：





```
from django.contrib.sites.shortcuts import get_current_site

def my_view(request):
    current_site = get_current_site(request)
    if current_site.domain == 'foo.com':
        # Do something
        pass
    else:
        # Do something else.
        pass

```





它还有一个优点是检查Sites 框架是否安装，如果没有安装将返回一个 [`RequestSite`](#django.contrib.sites.requests.RequestSite "django.contrib.sites.requests.RequestSite") 实例。

如果你不能访问request 对象，你可以使用[`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 模型管理器的`get_current()` 方法。此时，你需要确保你的设置文件包含[`SITE_ID`](../settings.html#std:setting-SITE_ID) 设置。下面的示例与前面的示例等同：





```
from django.contrib.sites.models import Site

def my_function_without_request():
    current_site = Site.objects.get_current()
    if current_site.domain == 'foo.com':
        # Do something
        pass
    else:
        # Do something else.
        pass

```









### 显示当前的域名

LJWorld.com 和Lawrence.com 都具有邮件通知功能，它让读者注册以在新闻发生时获得通知。这很简单：读者通过网页表单注册，然后立即收到一封邮件说 “感谢您的订阅”。

将这个注册过程的代码实现两次是低效而冗余的，所以这两个站点在后台使用相同的代码。但是每个Site 的“感谢您的订阅”的通知需要不同。通过使用[`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 对象，我们可以抽象这个通知并利用当前Site 的[`name`](#django.contrib.sites.models.Site.name "django.contrib.sites.models.Site.name") 和[`domain`](#django.contrib.sites.models.Site.domain "django.contrib.sites.models.Site.domain") 的值。

下面是该表单处理视图的一个例子：





```
from django.contrib.sites.shortcuts import get_current_site
from django.core.mail import send_mail

def register_for_newsletter(request):
    # Check form values, etc., and subscribe the user.
    # ...

    current_site = get_current_site(request)
    send_mail('Thanks for subscribing to %s alerts' % current_site.name,
        'Thanks for your subscription. We appreciate it.\n\n-The %s team.' % current_site.name,
        'editor@%s' % current_site.domain,
        [user.email])

    # ...

```





在Lawrence.com 网站上，这封邮件的标题为“Thanks for subscribing to lawrence.com alerts.”。在LJWorld.com 网站上，这封邮件的标题为“Thanks for subscribing to LJWorld.com alerts.”。邮件体的行为相同。

注意，更加灵活（但是更沉重）的方法是使用Django 的模板系统。假设Lawrence.com 和LJWorld.com 具有不同的模板目录（[`DIRS`](../settings.html#std:setting-TEMPLATES-DIRS)），你可以很容易地根据模板系统写出：





```
from django.core.mail import send_mail
from django.template import loader, Context

def register_for_newsletter(request):
    # Check form values, etc., and subscribe the user.
    # ...

    subject = loader.get_template('alerts/subject.txt').render(Context({}))
    message = loader.get_template('alerts/message.txt').render(Context({}))
    send_mail(subject, message, 'editor@ljworld.com', [user.email])

    # ...

```





在这种情况下，你必须为LJWorld.com 和Lawrence.com 模板目录都创建`subject.txt` 和`message.txt` 模板文件。它更灵活，但是也更复杂。

尽可能地发掘[`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 对象的用法以删除不需要的复杂性和冗余是个不错的主意。





### 获取当前域名的url全路径

Django 的`get_absolute_url()` 可以很方便地获得对象不带域名的URL，但是某些情况下，你可能想显示完整的URL，带有`http://`和域名以及其它部分。要实现这点，你可以使用Sites 框架。一个简单的示例：





```
>>> from django.contrib.sites.models import Site
>>> obj = MyModel.objects.get(id=3)
>>> obj.get_absolute_url()
'/mymodel/objects/3/'
>>> Site.objects.get_current().domain
'example.com'
>>> 'http://%s%s' % (Site.objects.get_current().domain, obj.get_absolute_url())
'http://example.com/mymodel/objects/3/'

```











## 启用Sites 框架

按照以下步骤启用Sites 框架：

1.  添加`'django.contrib.sites'` 到你的[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 设置中。

2.  定义[`SITE_ID`](../settings.html#std:setting-SITE_ID) 设置：





```
SITE_ID = 1

```





3.  运行[`migrate`](../django-admin.html#django-admin-migrate)。

`django.contrib.sites` 注册一个[`post_migrate`](../signals.html#django.db.models.signals.post_migrate "django.db.models.signals.post_migrate") 信号处理器，它创建一个默认的Site`example.com`，其域名为`example.com`。在Django 创建测试数据库之后，也会创建该Site。你可以使用[_数据迁移_](../../topics/migrations.html#data-migrations)来为你的项目设置正确的name 和domain。

为了在线上环境中启用多个Site，你应该为每个`SITE_ID` 创建一个单独的设置文件（可以从一个共同的设置文件导入，以避免重复共享的配置），然后为每个Site 指定合适的[`DJANGO_SETTINGS_MODULE`](../../topics/settings.html#envvar-DJANGO_SETTINGS_MODULE)。





## Caching the current `Site`

因为当前站点储存在数据库,每一次调用 `Site.objects.get_current()`都会导致数据库查询。但是Django还是比这个聪明滴, 当前站点被放在缓存当中了, 所以后续的调用返回的都是缓存的数据而不是直接查询数据库。

如果出于一些原因你想要强制用数据库查询, 你可以告诉Django清除缓存，用下面这个方法 `Site.objects.clear_cache()`:





```
# First call; current site fetched from database.
current_site = Site.objects.get_current()
# ...

# Second call; current site fetched from cache.
current_site = Site.objects.get_current()
# ...

# Force a database query for the third call.
Site.objects.clear_cache()
current_site = Site.objects.get_current()

```









## The `CurrentSiteManager`



_class_ `managers.CurrentSiteManager`



如果 [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 在你的应用中非常的关键， 你可以考虑用 [`CurrentSiteManager`](#django.contrib.sites.managers.CurrentSiteManager "django.contrib.sites.managers.CurrentSiteManager") 在你的模型中(s). 它是一个 model[_manager_](../../topics/db/managers.html)用来自动过滤，留下只与当前站点有关的数据查询 [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site").



Mandatory [`SITE_ID`](../settings.html#std:setting-SITE_ID)

`CurrentSiteManager` 只有在你定义了[`SITE_ID`](../settings.html#std:setting-SITE_ID) 在setting 中才起作用。



使用 [`CurrentSiteManager`](#django.contrib.sites.managers.CurrentSiteManager "django.contrib.sites.managers.CurrentSiteManager") ，你只要直接把他添加到你的model 中。For example:





```
from django.db import models
from django.contrib.sites.models import Site
from django.contrib.sites.managers import CurrentSiteManager

class Photo(models.Model):
    photo = models.FileField(upload_to='/home/photos')
    photographer_name = models.CharField(max_length=100)
    pub_date = models.DateField()
    site = models.ForeignKey(Site)
    objects = models.Manager()
    on_site = CurrentSiteManager()

```





通过这个model, `Photo.objects.all()` 将会返回所有在数据库中的 `Photo`对象，但是 `Photo.on_site.all()`只会返回 与当前site相关的`Photo`对象, 这是根据 [`SITE_ID`](../settings.html#std:setting-SITE_ID) 在setting的设置。

换句话说，这两种表达方式是等价的:





```
Photo.objects.filter(site=settings.SITE_ID)
Photo.on_site.all()

```





[`CurrentSiteManager`](#django.contrib.sites.managers.CurrentSiteManager "django.contrib.sites.managers.CurrentSiteManager")是如何知道哪个`Photo`字段是 [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site")的? 通常来说， [`CurrentSiteManager`](#django.contrib.sites.managers.CurrentSiteManager "django.contrib.sites.managers.CurrentSiteManager")查找一个 [`ForeignKey`](../models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey") 它的名字叫`site` 或者是一个 [`ManyToManyField`](../models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField")字段 ，叫做 `sites`来筛选出. 如果你用名字不叫`site` or `sites`的字段来表示一个与[`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site")对象相关联,，那么你就需要在你的model中显示得传递自定义的字段名给[`CurrentSiteManager`](#django.contrib.sites.managers.CurrentSiteManager "django.contrib.sites.managers.CurrentSiteManager")。下面的model, 它有一个字段叫做 `publish_on`, 说明了这个问题：





```
from django.db import models
from django.contrib.sites.models import Site
from django.contrib.sites.managers import CurrentSiteManager

class Photo(models.Model):
    photo = models.FileField(upload_to='/home/photos')
    photographer_name = models.CharField(max_length=100)
    pub_date = models.DateField()
    publish_on = models.ForeignKey(Site)
    objects = models.Manager()
    on_site = CurrentSiteManager('publish_on')

```





如果你尝试使用[`CurrentSiteManager`](#django.contrib.sites.managers.CurrentSiteManager "django.contrib.sites.managers.CurrentSiteManager") 并且传递了一个并不存在的字段名称给他, Django 就会引发一个 `ValueError`.

最后, 注意你可能会想要保持一个正常的 (non-site-specific) `Manager` 在你的model, 虽然你使用了 [`CurrentSiteManager`](#django.contrib.sites.managers.CurrentSiteManager "django.contrib.sites.managers.CurrentSiteManager"). 就像 [_manager documentation_](../../topics/db/managers.html)当中的解释那样，如果你手动定义了一个manager,Django是不会为你自动创建 `objects = models.`Manager() manager。也请注意某些 Django组件 –即, Django admin site 和通用视图– 使用的是 _first_定义 在你model中的manager，所以如果你希望你的admin site可以连接到所有对象 (不仅仅是特定的站点对象), 那就设置 `objects = models.`Manager() 在你的 model中, 并且在你定义[`CurrentSiteManager`](#django.contrib.sites.managers.CurrentSiteManager "django.contrib.sites.managers.CurrentSiteManager")之前。





## Site middleware

New in Django 1.7.

如果你经常使用这个模式：





```
from django.contrib.sites.models import Site

def my_view(request):
    site = Site.objects.get_current()
    ...

```





这里有些方法可以防止这种重复调用。添加 [`django.contrib.sites.middleware.CurrentSiteMiddleware`](../middleware.html#django.contrib.sites.middleware.CurrentSiteMiddleware "django.contrib.sites.middleware.CurrentSiteMiddleware") 到[`MIDDLEWARE_CLASSES`](../settings.html#std:setting-MIDDLEWARE_CLASSES). 中间件设置 `site` 属性给每一次request对象, 所以你可以用 `request.site` 来获取当前site。





## Django是如何使用的站点框架

虽然不强制要求你的网站使用site框架，但是我们鼓励你使用它，因为在一些地方Django利用它。 即使你的Django只在支持单个站点, 你也应该花两秒时间来给你的站点对象创建`domain` 和`name`,并且设置它的ID在你的 [`SITE_ID`](../settings.html#std:setting-SITE_ID) setting中。

下面是Django 如何使用sites framework:

*   在 [`redirects framework`](redirects.html#module-django.contrib.redirects "django.contrib.redirects: A framework for managing redirects."),每一个redirect都和特定的站点相关联。当Django查找一个 redirect, 它就考虑在当前的站点中查找。
*   在 [`flatpages 框架`](flatpages.html#module-django.contrib.flatpages "django.contrib.flatpages: A framework for managing simple ?flat? HTML content in a database."), 每一个flatpage 都被关联到特定的站点。当一个 flatpage 被创建， 你指定它的 [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site"),并且[`FlatpageFallbackMiddleware`](flatpages.html#django.contrib.flatpages.middleware.FlatpageFallbackMiddleware "django.contrib.flatpages.middleware.FlatpageFallbackMiddleware") 在返回flatpages 中检查当前站以显示。
*   在 [`syndication framework`](syndication.html#module-django.contrib.syndication "django.contrib.syndication: A framework for generating syndication feeds, in RSS and Atom, quite easily.")中, 模板的 `title` and `description` 自动访问变量`{{ site }}`, 这个 [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 代表当前站点的站点对象。. 此外，挂钩提供项URL将使用当前 [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site")对象的`domain`，如果你不指定一个完全合格的域名。
*   在[`authentication framework`](../../topics/auth/index.html#module-django.contrib.auth "django.contrib.auth: Django's authentication framework.")中， [`django.contrib.auth.views.login()`](../../topics/auth/default.html#django.contrib.auth.views.login "django.contrib.auth.views.login") 视图传递当前 [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 名称的模板`{{ site_name }}`.
*   快捷视图 (`django.contrib.contenttypes.views.shortcut`) 使用当前[`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site")对象的的域 计算对象的URL。
*   在管理框架, “view on site” 链接使用当前 [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 算出将重定向的域名.





## RequestSite objects

一些 [_django.contrib_](index.html)应用有利用到 sites framework 但是它们的架构不会_require_ sites framework必须安装在你的数据库中。有些人不想, 或者不能安装site ?framework所要求的_able_在他们的数据库中。) 出于这种情况，framework 提供了一个 [`django.contrib.sites.requests.RequestSite`](#django.contrib.sites.requests.RequestSite "django.contrib.sites.requests.RequestSite")类，当你数据支持的站点框架不可用的时候做一个回退



_class_ `requests.RequestSite`



A class that shares the primary interface of [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") (i.e., it has `domain` and `name` attributes) but gets its data from a Django [`HttpRequest`](../request-response.html#django.http.HttpRequest "django.http.HttpRequest") object rather than from a database.



`__init__`(_request_)



Sets the `name` and `domain` attributes to the value of [`get_host()`](../request-response.html#django.http.HttpRequest.get_host "django.http.HttpRequest.get_host").







Deprecated since version 1.7: This class used to be defined in `django.contrib.sites.models`. The old import location will work until Django 1.9.







A [`RequestSite`](#django.contrib.sites.requests.RequestSite "django.contrib.sites.requests.RequestSite") object has a similar interface to a normal [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") object, except its [`__init__()`](#django.contrib.sites.requests.RequestSite.__init__ "django.contrib.sites.requests.RequestSite.__init__") method takes an [`HttpRequest`](../request-response.html#django.http.HttpRequest "django.http.HttpRequest") object. It’s able to deduce the `domain` and `name` by looking at the request’s domain. It has `save()` and `delete()` methods to match the interface of [`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site"), but the methods raise [`NotImplementedError`](https://docs.python.org/3/library/exceptions.html#NotImplementedError "(in Python v3.4)").





## get_current_site shortcut

最后,为了避免重复的回退代码，site framework 提供了一个 [`django.contrib.sites.shortcuts.get_current_site()`](#django.contrib.sites.shortcuts.get_current_site "django.contrib.sites.shortcuts.get_current_site") 功能。



`shortcuts.get_current_site`(_request_)



这是函数是用来检查`django.contrib.sites` 是否安装并且返回一个基于request的[`Site`](#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 对象或者一个[`RequestSite`](#django.contrib.sites.requests.RequestSite "django.contrib.sites.requests.RequestSite") 对象。



Deprecated since version 1.7: This function used to be defined in `django.contrib.sites.models`. The old import location will work until Django 1.9.



Changed in Django 1.8:

This function will now lookup the current site based on [`request.get_host()`](../request-response.html#django.http.HttpRequest.get_host "django.http.HttpRequest.get_host") if the [`SITE_ID`](../settings.html#std:setting-SITE_ID) setting is not defined.
