

# Sitemap 框架

Django 自带了一个高级的网站地图创建框架， 这使得创建XML格式的[网站地图](http://www.sitemaps.org/) 变得容易。

## 概述

一个站点地图是一个在你网站上的用来告诉搜索引擎你的页面更新的多频繁和某些页面在你的网站中的重要关系的索引的XML文件此信息有助于搜索引擎为您的网站编制索引。

Django sitemap 框架通过让你在 Python 代码中表达此信息，自动创建此 XML 文件。

它的工作原理很像 Django 的[_联合框架_](syndication.html)。为了创建网站地图，只需编写 [`Sitemap`](#django.contrib.sitemaps.Sitemap "django.contrib.sitemaps.Sitemap") 类，并在 [_URLconf_](../../topics/http/urls.html) 中指向该类。

## 安装

安装网站地图APP的步骤如下：

1.  在[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS)设置中添加`'django.contrib.sitemaps'` .
2.  确认你的[`TEMPLATES`](../settings.html#std:setting-TEMPLATES) 设置包含 `DjangoTemplates` 后端，并将`APP_DIRS` 选项设置为 `True`. 当然默认值就是这样的，只有当你曾经修改过这些设置，才需要修改这个配置。
3.  确认你已经安装[`sites framework`](sites.html#module-django.contrib.sites "django.contrib.sites: Lets you operate multiple Web sites from the same database and Django project").

(注意: 网站地图APP并不安装任何数据库表 。需要修改[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS)的唯一原因是，以便[`Loader()`](../templates/api.html#django.template.loaders.app_directories.Loader "django.template.loaders.app_directories.Loader")模板加载器可以找到默认模板。）

## 初始化

`views.``sitemap`(_request_, _sitemaps_, _section=None_, _template_name='sitemap.xml'_, _content_type='application/xml'_)

为了在你的Django网站激活网站地图生成功能, 请把以下代码添加 [_URLconf_](../../topics/http/urls.html):

```
from django.contrib.sitemaps.views import sitemap

url(r'^sitemap\.xml$', sitemap, {'sitemaps': sitemaps},
    name='django.contrib.sitemaps.views.sitemap')

```

当客服端访问 `/sitemap.xml`时，这将告诉Django生成一个网站地图。

网站地图的文件名并不重要，重要的是文件的位置。搜索引擎只会索引网站的当前 URL 层级及下属层级。例如，如果 `sitemap.xml` 位于根目录中，它可能会引用网站中的任何 URL。但是，如果站点地图位于 `/content/sitemap.xml`，则它只能引用以 `/content/` 开头的网址。

网站地图视图需要一个额外的必需参数：`{'sitemaps'： sitemaps}`。`sitemaps` 应是一个字典，将小节的标签（例如 `blog` 或 `news`）映射到其 [`Sitemap`](#django.contrib.sitemaps.Sitemap "django.contrib.sitemaps.Sitemap") 类（例如，`BlogSitemap` 或 `NewsSitemap`）。它也可以映射到 [`Sitemap`](#django.contrib.sitemaps.Sitemap "django.contrib.sitemaps.Sitemap") 类的_实例_（例如，`BlogSitemap（some_var）`）。

## Sitemap 类

[`Sitemap`](#django.contrib.sitemaps.Sitemap "django.contrib.sitemaps.Sitemap") 类是一个简单的 Python 类，表示站点地图中“一部分”条目。例如，一个 [`Sitemap`](#django.contrib.sitemaps.Sitemap "django.contrib.sitemaps.Sitemap") 类可以表示 Weblog 的所有条目，而另一个可以表示事件日历中的所有事件。

在最简单的情况下，所有这些部分都集中到一个 `sitemap.xml` 中，但也可以使用框架，为每个部分生成一个站点地图索引，它引用单个站点地图文件。（请参阅下面的[创建网站地图索引](#creating-a-sitemap-index)。）

[`Sitemap`](#django.contrib.sitemaps.Sitemap "django.contrib.sitemaps.Sitemap") 类必须继承自 `django.contrib.sitemaps.Sitemap`。它们可以位于你的代码库中的任何地方。

## 一个简单示例

假设你有一个博客系统，拥有 `Entry` 模型，并且您希望站点地图包含指向各个博客条目的所有链接。以下是您的Sitemap类别的外观：

```
from django.contrib.sitemaps import Sitemap
from blog.models import Entry

class BlogSitemap(Sitemap):
    changefreq = "never"
    priority = 0.5

    def items(self):
        return Entry.objects.filter(is_draft=False)

    def lastmod(self, obj):
        return obj.pub_date

```

注意：

*   [`changefreq`](#django.contrib.sitemaps.Sitemap.changefreq "django.contrib.sitemaps.Sitemap.changefreq")和[`priority`](#django.contrib.sitemaps.Sitemap.priority "django.contrib.sitemaps.Sitemap.priority")分别是对应于`&lt;changefreq&gt;`和`&lt;priority&gt;`它们可以作为函数调用，例如这个例子中的[`lastmod`](#django.contrib.sitemaps.Sitemap.lastmod "django.contrib.sitemaps.Sitemap.lastmod")。
*   [`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")只是一个返回对象列表的方法。返回的对象将传递给与网站地图属性（[`location`](#django.contrib.sitemaps.Sitemap.location "django.contrib.sitemaps.Sitemap.location")，[`lastmod`](#django.contrib.sitemaps.Sitemap.lastmod "django.contrib.sitemaps.Sitemap.lastmod")，[`changefreq`](#django.contrib.sitemaps.Sitemap.changefreq "django.contrib.sitemaps.Sitemap.changefreq")和[`priority`](#django.contrib.sitemaps.Sitemap.priority "django.contrib.sitemaps.Sitemap.priority")
*   [`lastmod`](#django.contrib.sitemaps.Sitemap.lastmod "django.contrib.sitemaps.Sitemap.lastmod") 应返回 Python `datetime` 对象。
*   在此示例中没有 [`location`](#django.contrib.sitemaps.Sitemap.location "django.contrib.sitemaps.Sitemap.location") 方法，但你可以提供此方法来指定对象的 URL。默认情况下，[`location()`](#django.contrib.sitemaps.Sitemap.location "django.contrib.sitemaps.Sitemap.location")在每个对象上调用`get_absolute_url()`并返回结果。

## Sitemap 类参考

_class_ `Sitemap`[[source]](../../_modules/django/contrib/sitemaps.html#Sitemap)

`Sitemap`类可以定义以下方法/属性：

`items`[[source]](../../_modules/django/contrib/sitemaps.html#Sitemap.items)

**必需。**返回对象列表的方法。框架不关心它们是什么_类型_；所有重要的是这些对象被传递到[`location()`](#django.contrib.sitemaps.Sitemap.location "django.contrib.sitemaps.Sitemap.location")，[`lastmod()`](#django.contrib.sitemaps.Sitemap.lastmod "django.contrib.sitemaps.Sitemap.lastmod")，[`changefreq()`](#django.contrib.sitemaps.Sitemap.changefreq "django.contrib.sitemaps.Sitemap.changefreq")和[`priority()`](#django.contrib.sitemaps.Sitemap.priority "django.contrib.sitemaps.Sitemap.priority")方法。

`location`[[source]](../../_modules/django/contrib/sitemaps.html#Sitemap.location)

**可选的.** 进入一个方法或属性

如果它是一个方法, 它应该为[`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")返回的对象返回绝对路径.

如果它是一个属性，它的值应该是一个字符串，表示[`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")返回的_每个_对象的绝对路径。

在这两种情况下，“绝对路径”表示不包含协议或域的URL。例子：

*   好：`'/foo/bar/'`
*   错误：`'example.com/foo/bar/'`
*   错误：`'http://example.com/foo/bar/'`

如果未提供[`location`](#django.contrib.sitemaps.Sitemap.location "django.contrib.sitemaps.Sitemap.location")，框架将调用[`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")返回的每个对象上的`get_absolute_url()`方法。

要指定`'http'`之外的协议，请使用[`protocol`](#django.contrib.sitemaps.Sitemap.protocol "django.contrib.sitemaps.Sitemap.protocol")。

`lastmod`

**可选。**方法或属性。

如果它是一个方法，它应该接受一个参数 - 由[`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")返回的对象，并返回对象的最后修改日期/时间，如Python `datetime.datetime`

If it’s an attribute, its value should be a Python `datetime.datetime` object representing the last-modified date/time for _every_ object returned by [`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items").

New in Django 1.7.

如果网站地图中的所有项目都有[`lastmod`](#django.contrib.sitemaps.Sitemap.lastmod "django.contrib.sitemaps.Sitemap.lastmod")，则由[`views.sitemap()`](#django.contrib.sitemaps.views.sitemap "django.contrib.sitemaps.views.sitemap")生成的网站地图会有`Last-Modified`最新`lastmod`。您可以激活[`ConditionalGetMiddleware`](../middleware.html#django.middleware.http.ConditionalGetMiddleware "django.middleware.http.ConditionalGetMiddleware")，使Django对具有`If-Modified-Since`标头的请求作出适当响应，如果没有更改，则会阻止发送站点地图。

`changefreq`

**可选。**方法或属性。

如果它是一个方法，它应该接受一个参数 - 由[`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")返回的对象 - 并将对象的更改频率作为Python字符串返回。

如果是属性，则其值应为表示[`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")返回的_每个_对象的更改频率的字符串。

不管您使用方法还是属性，[`changefreq`](#django.contrib.sitemaps.Sitemap.changefreq "django.contrib.sitemaps.Sitemap.changefreq")的可能值为：

*   `'always'`
*   `'hourly'`
*   `'daily'`
*   `'weekly'`
*   `'monthly'`
*   `'yearly'`
*   `'never'`

`priority`

**可选。**方法或属性。

如果它是一个方法，它应该接受一个参数 - 由[`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")返回的对象 - 并返回对象的优先级，如字符串或浮点数。

如果它是一个属性，它的值应该是一个字符串或浮动，表示[`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")返回的_每个_对象的优先级。

[`priority`](#django.contrib.sitemaps.Sitemap.priority "django.contrib.sitemaps.Sitemap.priority")的示例值：`0.4`，`1.0`。页面的默认优先级为`0.5`。有关详情，请参阅[sitemaps.org文档](http://www.sitemaps.org/protocol.html#prioritydef)。

`protocol`

**可选。**

此属性定义网站地图中的网址的协议（`'http'`或`'https'`）。如果未设置，则使用请求站点地图的协议。如果Sitemap是在请求的上下文之外构建的，则默认为`'http'`。

`limit`

**可选。**

此属性定义网站地图的每个网页上包含的最大网址数。其值不应超过`50000`的默认值，这是[Sitemaps协议](http://www.sitemaps.org/protocol.html#index)中允许的上限。

`i18n`

New in Django 1.8.

**可选。**

一个boolean属性，用于定义是否应使用您的所有[`LANGUAGES`](../settings.html#std:setting-LANGUAGES)生成此网站地图的网址。默认值为`False`。

## 快捷方式

对于常见的情况，Sitemap框架提供了一些方便的类：

_class_ `FlatPageSitemap`[[source]](../../_modules/django/contrib/sitemaps.html#FlatPageSitemap)

自1.8版起已弃用：请改用[`django.contrib.flatpages.sitemaps.FlatPageSitemap`](flatpages.html#django.contrib.flatpages.sitemaps.FlatPageSitemap "django.contrib.flatpages.sitemaps.FlatPageSitemap")。

[`django.contrib.sitemaps.FlatPageSitemap`](#django.contrib.sitemaps.FlatPageSitemap "django.contrib.sitemaps.FlatPageSitemap")类查看为当前[`SITE_ID`](../settings.html#std:setting-SITE_ID)定义的所有公开可见的[`flatpages`](flatpages.html#module-django.contrib.flatpages "django.contrib.flatpages: A framework for managing simple ?flat? HTML content in a database.")（请参阅[`sites documentation`](sites.html#module-django.contrib.sites "django.contrib.sites: Lets you operate multiple Web sites from the same database and Django project")），并在站点地图中创建一个条目。这些条目仅包含[`location`](#django.contrib.sitemaps.Sitemap.location "django.contrib.sitemaps.Sitemap.location")属性 - 不是[`lastmod`](#django.contrib.sitemaps.Sitemap.lastmod "django.contrib.sitemaps.Sitemap.lastmod")，[`changefreq`](#django.contrib.sitemaps.Sitemap.changefreq "django.contrib.sitemaps.Sitemap.changefreq")或[`priority`](#django.contrib.sitemaps.Sitemap.priority "django.contrib.sitemaps.Sitemap.priority")。

_class_ `GenericSitemap`[[source]](../../_modules/django/contrib/sitemaps.html#GenericSitemap)

[`django.contrib.sitemaps.GenericSitemap`](#django.contrib.sitemaps.GenericSitemap "django.contrib.sitemaps.GenericSitemap")类允许您通过传递一个必须至少包含`queryset`条目的字典来创建站点地图。此查询集将用于生成站点地图的项目。它还可以具有`date_field`条目，其指定从`queryset`检索的对象的日期字段。这将用于生成的站点地图中的[`lastmod`](#django.contrib.sitemaps.Sitemap.lastmod "django.contrib.sitemaps.Sitemap.lastmod")属性。您还可以将[`priority`](#django.contrib.sitemaps.Sitemap.priority "django.contrib.sitemaps.Sitemap.priority")和[`changefreq`](#django.contrib.sitemaps.Sitemap.changefreq "django.contrib.sitemaps.Sitemap.changefreq")关键字参数传递到[`GenericSitemap`](#django.contrib.sitemaps.GenericSitemap "django.contrib.sitemaps.GenericSitemap")构造函数，以指定所有网址的这些属性。

### 例

以下是使用[`GenericSitemap`](#django.contrib.sitemaps.GenericSitemap "django.contrib.sitemaps.GenericSitemap")的[_URLconf_](../../topics/http/urls.html)的示例：

```
from django.conf.urls import url
from django.contrib.sitemaps import GenericSitemap
from django.contrib.sitemaps.views import sitemap
from blog.models import Entry

info_dict = {
    'queryset': Entry.objects.all(),
    'date_field': 'pub_date',
}

urlpatterns = [
    # some generic view using info_dict
    # ...

    # the sitemap
    url(r'^sitemap\.xml$', sitemap,
        {'sitemaps': {'blog': GenericSitemap(info_dict, priority=0.6)}},
        name='django.contrib.sitemaps.views.sitemap'),
]

```

## 静态视图的Sitemap

通常，您希望搜索引擎抓取工具索引既不是对象详细信息页面也不是平面页的视图。解决方案是在`items`中显式列出这些视图的网址名称，并在网站地图的`location`方法中调用[`reverse()`](../urlresolvers.html#django.core.urlresolvers.reverse "django.core.urlresolvers.reverse")。例如：

```
# sitemaps.py
from django.contrib import sitemaps
from django.core.urlresolvers import reverse

class StaticViewSitemap(sitemaps.Sitemap):
    priority = 0.5
    changefreq = 'daily'

    def items(self):
        return ['main', 'about', 'license']

    def location(self, item):
        return reverse(item)

# urls.py
from django.conf.urls import url
from django.contrib.sitemaps.views import sitemap

from .sitemaps import StaticViewSitemap
from . import views

sitemaps = {
    'static': StaticViewSitemap,
}

urlpatterns = [
    url(r'^$', views.main, name='main'),
    url(r'^about/$', views.about, name='about'),
    url(r'^license/$', views.license, name='license'),
    # ...
    url(r'^sitemap\.xml$', sitemap, {'sitemaps': sitemaps},
        name='django.contrib.sitemaps.views.sitemap')
]

```

## 创建网站地图索引

`views.``index`(_request_, _sitemaps_, _template_name='sitemap_index.xml'_, _content_type='application/xml'_, _sitemap_url_name='django.contrib.sitemaps.views.sitemap'_)

站点地图框架还能够创建引用单个站点地图文件的站点地图索引，您的`sitemaps`字典中定义的每个部分一个。唯一的区别是：

*   您在URLconf中使用两个视图：[`django.contrib.sitemaps.views.index()`](#django.contrib.sitemaps.views.index "django.contrib.sitemaps.views.index")和[`django.contrib.sitemaps.views.sitemap()`](#django.contrib.sitemaps.views.sitemap "django.contrib.sitemaps.views.sitemap")。
*   [`django.contrib.sitemaps.views.sitemap()`](#django.contrib.sitemaps.views.sitemap "django.contrib.sitemaps.views.sitemap")视图应采用`section`关键字参数。

这里是上面的例子的相关URLconf行：

```
from django.contrib.sitemaps import views

urlpatterns = [
    url(r'^sitemap\.xml$', views.index, {'sitemaps': sitemaps}),
    url(r'^sitemap-(?P<section>.+)\.xml$', views.sitemap, {'sitemaps': sitemaps}),
]

```

这将自动生成引用`sitemap-flatpages.xml`和`sitemap-blog.xml`的`sitemap.xml`文件。[`Sitemap`](#django.contrib.sitemaps.Sitemap "django.contrib.sitemaps.Sitemap")类和`sitemaps`类别根本不会更改。

如果您的其中一个Sitemap包含超过50,000个网址，就应该建立索引档。在这种情况下，Django会自动对网站地图分页，索引会反映出来。

如果您不使用vanilla网站地图视图（例如，如果使用缓存装饰器包装），则必须为您的网站地图视图命名，并将`sitemap_url_name`传递到索引视图：

```
from django.contrib.sitemaps import views as sitemaps_views
from django.views.decorators.cache import cache_page

urlpatterns = [
    url(r'^sitemap\.xml$',
        cache_page(86400)(sitemaps_views.index),
        {'sitemaps': sitemaps, 'sitemap_url_name': 'sitemaps'}),
    url(r'^sitemap-(?P<section>.+)\.xml$',
        cache_page(86400)(sitemaps_views.sitemap),
        {'sitemaps': sitemaps}, name='sitemaps'),
]

```

## 模板定制

如果您希望为网站上可用的每个站点地图或站点地图索引使用不同的模板，您可以通过将`template_name`参数传递到`sitemap`和`index`视图：

```
from django.contrib.sitemaps import views

urlpatterns = [
    url(r'^custom-sitemap\.xml$', views.index, {
        'sitemaps': sitemaps,
        'template_name': 'custom_sitemap.html'
    }),
    url(r'^custom-sitemap-(?P<section>.+)\.xml$', views.sitemap, {
        'sitemaps': sitemaps,
        'template_name': 'custom_sitemap.html'
    }),
]

```

这些视图返回[`TemplateResponse`](../template-response.html#django.template.response.TemplateResponse "django.template.response.TemplateResponse")实例，允许您在渲染之前轻松自定义响应数据。有关详细信息，请参阅[_TemplateResponse documentation_](../template-response.html)。

### 上下文变量

当自定义[`index()`](#django.contrib.sitemaps.views.index "django.contrib.sitemaps.views.index")和[`sitemap()`](#django.contrib.sitemaps.views.sitemap "django.contrib.sitemaps.views.sitemap")视图的模板时，您可以依赖于以下上下文变量。

### 指数

变量`sitemaps`是每个站点地图的绝对网址列表。

### Sitemap

变量`urlset`是应显示在网站地图中的网址列表。每个网址都会显示在[`Sitemap`](#django.contrib.sitemaps.Sitemap "django.contrib.sitemaps.Sitemap")类中定义的属性：

*   `changefreq`
*   `item`
*   `lastmod`
*   `location`
*   `priority`

已为每个网址添加了`item`属性，以允许更灵活地自定义模板，例如[Google新闻站点地图](https://support.google.com/news/publisher/answer/74288?hl=en)。假设Sitemap的[`items()`](#django.contrib.sitemaps.Sitemap.items "django.contrib.sitemaps.Sitemap.items")会传回含有`publication_data`和`tags`的项目清单，系统就会产生Google新闻相容的Sitemap：

```
<?xml version="1.0" encoding="UTF-8"?>
<urlset
  xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
  xmlns:news="http://www.google.com/schemas/sitemap-news/0.9">
{% spaceless %}
{% for url in urlset %}
  <url>
    <loc>{{ url.location }}</loc>
    {% if url.lastmod %}<lastmod>{{ url.lastmod|date:"Y-m-d" }}</lastmod>{% endif %}
    {% if url.changefreq %}<changefreq>{{ url.changefreq }}</changefreq>{% endif %}
    {% if url.priority %}<priority>{{ url.priority }}</priority>{% endif %}
    <news:news>
      {% if url.item.publication_date %}<news:publication_date>{{ url.item.publication_date|date:"Y-m-d" }}</news:publication_date>{% endif %}
      {% if url.item.tags %}<news:keywords>{{ url.item.tags }}</news:keywords>{% endif %}
    </news:news>
   </url>
{% endfor %}
{% endspaceless %}
</urlset>

```

## 正在Ping Google

你可能希望在 Sitemap 更改时“ping”Google，以便让其重新索引你的网站。sitemaps框架提供了一个函数：[`django.contrib.sitemaps.ping_google()`](#django.contrib.sitemaps.ping_google "django.contrib.sitemaps.ping_google")。

`ping_google`()[[source]](../../_modules/django/contrib/sitemaps.html#ping_google)

[`ping_google()`](#django.contrib.sitemaps.ping_google "django.contrib.sitemaps.ping_google")使用可选参数`sitemap_url`，该参数应为网站站点地图的绝对路径（例如`'/sitemap.xml'`如果未提供此参数，则[`ping_google()`](#django.contrib.sitemaps.ping_google "django.contrib.sitemaps.ping_google")将尝试通过在URLconf中执行反向查找来确定您的站点地图。

[`ping_google()`](#django.contrib.sitemaps.ping_google "django.contrib.sitemaps.ping_google")如果无法确定您的站点地图网址，则会引发例外`django.contrib.sitemaps.SitemapNotFound`。

先注册Google！

只有您已经使用[Google网站管理员工具](http://www.google.com/webmasters/tools/)注册了您的网站，[`ping_google()`](#django.contrib.sitemaps.ping_google "django.contrib.sitemaps.ping_google")命令才会生效。

调用[`ping_google()`](#django.contrib.sitemaps.ping_google "django.contrib.sitemaps.ping_google")的一个有用方法是从模型的`save()`方法：

```
from django.contrib.sitemaps import ping_google

class Entry(models.Model):
    # ...
    def save(self, force_insert=False, force_update=False):
        super(Entry, self).save(force_insert, force_update)
        try:
            ping_google()
        except Exception:
            # Bare 'except' because we could get a variety
            # of HTTP-related exceptions.
            pass

```

然而，更有效的解决方案是从cron脚本或其他计划的任务调用[`ping_google()`](#django.contrib.sitemaps.ping_google "django.contrib.sitemaps.ping_google")。该函数向Google的服务器发出HTTP请求，因此您可能不想在每次调用`save()`时引入该网络开销。

### 正在通过Google Ping`manage.py`

`django-admin ping_google`

将站点地图应用程序添加到您的项目后，您还可以使用`ping_google`管理命令ping Google：

```
python manage.py ping_google [/sitemap.xml]

```

