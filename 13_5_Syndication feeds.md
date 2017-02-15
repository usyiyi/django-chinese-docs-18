{% raw %}

# Feed聚合框架

Django带有一个高聚合的框架让创造[RSS](http://www.whatisrss.com/) 和 [Atom](http://tools.ietf.org/html/rfc4287)更容易。

创建聚合feed，你要做的仅仅是写一个简短的Python类。你想要创造多少，就能创造多少feeds。

Django 也带有一个低等级的生成feed的API。如果你想要生成一个外部的Web内容或者其他普通的方式，你可以使用它。

## 高等级的框架

### 概述

高等级的feed聚合框架由[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类提供。新建一个feed，写一个[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类，然后指向你的[_URLconf_](../../topics/http/urls.html).

### Feed 类

一个 [`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类就是一个提供聚合种子的Python类。一个简单的种子（例如新闻的信息种子，或者只展示博客最新消息）更多功能的种子（例如展示博客中允许展示的特定类别的条目）。

Feed类继承自[`django.contrib.syndication.views.Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")。它们可以在你代码中的任意一处。

[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类需要是你[_URLconf_](../../topics/http/urls.html)中的实例。

### 一个简单的示例

这个简单的示例，演示了某站点的最近五条新闻的记录：

```
from django.contrib.syndication.views import Feed
from django.core.urlresolvers import reverse
from policebeat.models import NewsItem

class LatestEntriesFeed(Feed):
    title = "Police beat site news"
    link = "/sitenews/"
    description = "Updates on changes and additions to police beat central."

    def items(self):
        return NewsItem.objects.order_by('-pub_date')[:5]

    def item_title(self, item):
        return item.title

    def item_description(self, item):
        return item.description

    # item_link is only needed if NewsItem has no get_absolute_url method.
    def item_link(self, item):
        return reverse('news-item', args=[item.pk])

```

配置一个URL到这个feed，在[_URLconf_](../../topics/http/urls.html)中配置一个入口。例如:

```
from django.conf.urls import url
from myproject.feeds import LatestEntriesFeed

urlpatterns = [
    # ...
    url(r'^latest/feed/$', LatestEntriesFeed()),
    # ...
]

```

注意:

*    Feed类继承于[`django.contrib.syndication.views.Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed").
*   `title`, `link` 和 `description` 分别对应 RSS的 `&lt;title&gt;`, `&lt;link&gt;` 和 `&lt;description&gt;` .
*   `items()`就是一个返回包含feed `&lt;item&gt;`对象列表的方法.当然，这个例子返回 `NewsItem`对象是使用 Django的 [_object-relational mapper_](../models/querysets.html), `items()` 没有返回模型的实例。当然，你可以从Django模型中很容易的获取一些数据，`items()`可以返回任何你想要的对象。
*   如果你要创建一个Atom feed，而不是RSS feed，你需要使用`subtitle`属性替代`description`。查看[Publishing Atom and RSS feeds in tandem](#publishing-atom-and-rss-feeds-in-tandem)里面的例子。

还有一件事没做。在一个 RSS feed中, 每一个 `&lt;item&gt;` 都有一个`&lt;title&gt;`, `&lt;link&gt;` 和`&lt;description&gt;`. 我们需要告诉框架那些数据放进这些对象中。

*   在 `&lt;title&gt;`和`&lt;description&gt;`的内容中，Django尝试着在[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类中召集`item_title()`和`item_description()`方法。他们会传入一个自己内部的单独的参数 `item`。这些都是可选的; 默认的是，unicode表示的对象都被使用了

    如果你想要做一些特殊的格式化title或者description，[_Django templates_](../templates/language.html)可以帮助你。他们的路径会被[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类中的`title_template`和`description_template` 参数做特殊处理。会通过模板内容的两个变量来返回每一条记录到模板中：

    *   `{% obj %}`，当前的对象（你在`items()`中返回的对象）。
    *   `{% site %}` – 一个[`django.contrib.sites.models.Site`](sites.html#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 对象代表当前站点。它对`{% site.domain %}` or `{% site.name %}`有用。如果你_not_没有Django站点，它会被设置为一个[`RequestSite`](sites.html#django.contrib.sites.requests.RequestSite "django.contrib.sites.requests.RequestSite")对象。从 [_RequestSite section of the sites framework documentation_](sites.html#id3) 获取更多信息。

    从下方的 [a complex example](#a-complex-example) 使用一个描述的模板。

    `Feed.``get_context_data`(_**kwargs_)

    如果您需要提供超过前面提到的两个变量，还有一种方法可以将附加信息传递到标题和描述模板。您可以在`Feed`子类中提供`get_context_data`方法的实现。例如:

    ```
    from mysite.models import Article
    from django.contrib.syndication.views import Feed

    class ArticlesFeed(Feed):
        title = "My articles"
        description_template = "feeds/articles.html"

        def items(self):
            return Article.objects.order_by('-pub_date')[:5]

        def get_context_data(self, **kwargs):
            context = super(ArticlesFeed, self).get_context_data(**kwargs)
            context['foo'] = 'bar'
            return context

    ```

    和模板：

    ```
    Something about {{ foo }}: {{ obj.description }}

    ```

    此方法将由`items()`返回的列表中的每个项目调用一次，并使用以下关键字参数：

    *   `item`：当前项目。出于向后兼容性原因，此上下文变量的名称为`{％ obj ％}`。
    *   `obj`：`get_object()`返回的对象。默认情况下，这不会暴露给模板，以避免与`{％ obj ％}` ，但您可以在实现`get_context_data()`时使用它。
    *   `site`：如上所述的当前网站。
    *   `request`：当前请求。

    `get_context_data()`的行为模仿了[_generic views_](../../topics/class-based-views/generic-display.html#adding-extra-context)的行为 - 你应该调用`super()`从父类中检索上下文数据，添加您的数据并返回修改后的字典。

*   要指定`&lt;link&gt;`的内容，您有两个选项。对于`items()`中的每个项目，Django首先尝试调用[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类上的`item_link()`方法。以类似于标题和描述的方式，它传递单个参数`item`。如果该方法不存在，Django尝试对该对象执行`get_absolute_url()`方法。`get_absolute_url()`和`item_link()`应将项目的网址作为普通的Python字符串返回。与`get_absolute_url()`一样，`item_link()`的结果将直接包含在URL中，因此您负责对所有必要的URL进行引用并将其转换为ASCII方法本身。

### 一个复杂的例子

该框架还通过参数支持更复杂的feed。

例如，网站可以为城市中的每个警察节拍提供最近的犯罪的RSS源。为每个警察节拍创建一个单独的[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")课；这将违反[_DRY principle_](../../misc/design-philosophies.html#dry)并且将数据耦合到编程逻辑。相反，联合框架可让您访问从[_URLconf_](../../topics/http/urls.html)传递的参数，以便Feed可以根据Feed的网址中的信息输出项目。

警察打击饲料可以通过这样的URL访问：

*   `/beats/613/rss/` - 返回613的最近犯罪。
*   `/beats/1424/rss/` - 返回1424年最近的犯罪。

这些可以与[_URLconf_](../../topics/http/urls.html)行匹配，例如：

```
url(r'^beats/(?P<beat_id>[0-9]+)/rss/$', BeatFeed()),

```

与视图类似，URL中的参数与请求对象一起传递到`get_object()`方法。

以下是这些特定于节拍的Feed的代码：

```
from django.contrib.syndication.views import FeedDoesNotExist
from django.shortcuts import get_object_or_404

class BeatFeed(Feed):
    description_template = 'feeds/beat_description.html'

    def get_object(self, request, beat_id):
        return get_object_or_404(Beat, pk=beat_id)

    def title(self, obj):
        return "Police beat central: Crimes for beat %s" % obj.beat

    def link(self, obj):
        return obj.get_absolute_url()

    def description(self, obj):
        return "Crimes recently reported in police beat %s" % obj.beat

    def items(self, obj):
        return Crime.objects.filter(beat=obj).order_by('-crime_date')[:30]

```

To generate the feed’s `&lt;title&gt;`, `&lt;link&gt;` and `&lt;description&gt;`, Django uses the `title()`, `link()` and `description()` methods. 在前面的示例中，它们是简单的字符串类属性，但是本示例说明它们可以是字符串_或_方法。对于`title`，`link`和`description`中的每一个，Django都遵循此算法：

*   首先，它尝试调用传递`obj`参数的方法，其中`obj`是由`get_object()`返回的对象。
*   如果没有，它试图调用没有参数的方法。
*   如果没有，它使用类属性。

还要注意，`items()`也遵循相同的算法 - 首先，它尝试`items(obj)`，然后`items()` `items`类属性（它应该是一个列表）。

我们正在使用模板作为项目描述。它可以很简单：

```
{{ obj.description }}

```

但是，您可以根据需要自由添加格式。

下面的`ExampleFeed`类提供了有关[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类的方法和属性的完整文档。

### 指定Feed的类型

默认情况下，此框架中生成的Feed使用RSS 2.0。

要更改此属性，请向您的[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类添加`feed_type`属性，如下所示：

```
from django.utils.feedgenerator import Atom1Feed

class MyFeed(Feed):
    feed_type = Atom1Feed

```

请注意，您将`feed_type`设置为类对象，而不是实例。

目前可用的Feed类型有：

*   [`django.utils.feedgenerator.Rss201rev2Feed`](../utils.html#django.utils.feedgenerator.Rss201rev2Feed "django.utils.feedgenerator.Rss201rev2Feed")（RSS 2.01。默认。）
*   [`django.utils.feedgenerator.RssUserland091Feed`](../utils.html#django.utils.feedgenerator.RssUserland091Feed "django.utils.feedgenerator.RssUserland091Feed")（RSS 0.91.）
*   [`django.utils.feedgenerator.Atom1Feed`](../utils.html#django.utils.feedgenerator.Atom1Feed "django.utils.feedgenerator.Atom1Feed")（Atom 1.0.）

### 附件

要指定机柜（例如用于创建Podcast Feed的机柜），请使用`item_enclosure_url`，`item_enclosure_length`和`item_enclosure_mime_type`挂钩。有关用法示例，请参见下面的`ExampleFeed`类。

### 语言

由联合框架创建的Feed会自动包含适当的`&lt;language&gt;`标记（RSS 2.0）或`xml:lang`属性（Atom）。这直接来自您的[`LANGUAGE_CODE`](../settings.html#std:setting-LANGUAGE_CODE)设置。

### 网址

`link`方法/属性可以返回绝对路径（例如`"/blog/"`）或具有完全限定域和协议的URL（例如`"http://www.example.com/blog/"`）。如果`link`未返回域，则整合框架将根据您的[`SITE_ID setting`](../settings.html#std:setting-SITE_ID)。

Atom Feed需要定义Feed的当前位置的`＆lt； link rel =“self”＆gt；`。

### 串联发布Atom和RSS Feed

有些开发人员喜欢提供Atom _和_ RSS版本的Feed。使用Django很容易：只需创建[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类的子类，并将`feed_type`设置为不同的类型即可。然后更新您的URLconf以添加额外的版本。

这里有一个完整的例子：

```
from django.contrib.syndication.views import Feed
from policebeat.models import NewsItem
from django.utils.feedgenerator import Atom1Feed

class RssSiteNewsFeed(Feed):
    title = "Police beat site news"
    link = "/sitenews/"
    description = "Updates on changes and additions to police beat central."

    def items(self):
        return NewsItem.objects.order_by('-pub_date')[:5]

class AtomSiteNewsFeed(RssSiteNewsFeed):
    feed_type = Atom1Feed
    subtitle = RssSiteNewsFeed.description

```

注意

在此示例中，RSS提要使用`description`，而Atom提要使用`subtitle`。这是因为Atom Feed不提供Feed级“说明”，但他们提供“字幕”。

如果您在[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类中提供`description`，Django将_不会_自动将其放入`subtitle`字幕和描述不一定是同一件事。而应定义`subtitle`属性。

在上面的示例中，我们只需将Atom Feed的`subtitle`设置为RSS Feed的`description`，因为它已经很短。

和附带的URLconf：

```
from django.conf.urls import url
from myproject.feeds import RssSiteNewsFeed, AtomSiteNewsFeed

urlpatterns = [
    # ...
    url(r'^sitenews/rss/$', RssSiteNewsFeed()),
    url(r'^sitenews/atom/$', AtomSiteNewsFeed()),
    # ...
]

```

### Feed类引用

_class_ `views.``Feed`

此示例说明[`Feed`](#django.contrib.syndication.views.Feed "django.contrib.syndication.views.Feed")类的所有可能的属性和方法：

```
from django.contrib.syndication.views import Feed
from django.utils import feedgenerator

class ExampleFeed(Feed):

    # FEED TYPE -- Optional. This should be a class that subclasses
    # django.utils.feedgenerator.SyndicationFeed. This designates
    # which type of feed this should be: RSS 2.0, Atom 1.0, etc. If
    # you don't specify feed_type, your feed will be RSS 2.0\. This
    # should be a class, not an instance of the class.

    feed_type = feedgenerator.Rss201rev2Feed

    # TEMPLATE NAMES -- Optional. These should be strings
    # representing names of Django templates that the system should
    # use in rendering the title and description of your feed items.
    # Both are optional. If a template is not specified, the
    # item_title() or item_description() methods are used instead.

    title_template = None
    description_template = None

    # TITLE -- One of the following three is required. The framework
    # looks for them in this order.

    def title(self, obj):
        """
 Takes the object returned by get_object() and returns the
 feed's title as a normal Python string.
 """

    def title(self):
        """
 Returns the feed's title as a normal Python string.
 """

    title = 'foo' # Hard-coded title.

    # LINK -- One of the following three is required. The framework
    # looks for them in this order.

    def link(self, obj):
        """
 # Takes the object returned by get_object() and returns the URL
 # of the HTML version of the feed as a normal Python string.
 """

    def link(self):
        """
 Returns the URL of the HTML version of the feed as a normal Python
 string.
 """

    link = '/blog/' # Hard-coded URL.

    # FEED_URL -- One of the following three is optional. The framework
    # looks for them in this order.

    def feed_url(self, obj):
        """
 # Takes the object returned by get_object() and returns the feed's
 # own URL as a normal Python string.
 """

    def feed_url(self):
        """
 Returns the feed's own URL as a normal Python string.
 """

    feed_url = '/blog/rss/' # Hard-coded URL.

    # GUID -- One of the following three is optional. The framework looks
    # for them in this order. This property is only used for Atom feeds
    # (where it is the feed-level ID element). If not provided, the feed
    # link is used as the ID.

    def feed_guid(self, obj):
        """
 Takes the object returned by get_object() and returns the globally
 unique ID for the feed as a normal Python string.
 """

    def feed_guid(self):
        """
 Returns the feed's globally unique ID as a normal Python string.
 """

    feed_guid = '/foo/bar/1234' # Hard-coded guid.

    # DESCRIPTION -- One of the following three is required. The framework
    # looks for them in this order.

    def description(self, obj):
        """
 Takes the object returned by get_object() and returns the feed's
 description as a normal Python string.
 """

    def description(self):
        """
 Returns the feed's description as a normal Python string.
 """

    description = 'Foo bar baz.' # Hard-coded description.

    # AUTHOR NAME --One of the following three is optional. The framework
    # looks for them in this order.

    def author_name(self, obj):
        """
 Takes the object returned by get_object() and returns the feed's
 author's name as a normal Python string.
 """

    def author_name(self):
        """
 Returns the feed's author's name as a normal Python string.
 """

    author_name = 'Sally Smith' # Hard-coded author name.

    # AUTHOR EMAIL --One of the following three is optional. The framework
    # looks for them in this order.

    def author_email(self, obj):
        """
 Takes the object returned by get_object() and returns the feed's
 author's email as a normal Python string.
 """

    def author_email(self):
        """
 Returns the feed's author's email as a normal Python string.
 """

    author_email = 'test@example.com' # Hard-coded author email.

    # AUTHOR LINK --One of the following three is optional. The framework
    # looks for them in this order. In each case, the URL should include
    # the "http://" and domain name.

    def author_link(self, obj):
        """
 Takes the object returned by get_object() and returns the feed's
 author's URL as a normal Python string.
 """

    def author_link(self):
        """
 Returns the feed's author's URL as a normal Python string.
 """

    author_link = 'http://www.example.com/' # Hard-coded author URL.

    # CATEGORIES -- One of the following three is optional. The framework
    # looks for them in this order. In each case, the method/attribute
    # should return an iterable object that returns strings.

    def categories(self, obj):
        """
 Takes the object returned by get_object() and returns the feed's
 categories as iterable over strings.
 """

    def categories(self):
        """
 Returns the feed's categories as iterable over strings.
 """

    categories = ("python", "django") # Hard-coded list of categories.

    # COPYRIGHT NOTICE -- One of the following three is optional. The
    # framework looks for them in this order.

    def feed_copyright(self, obj):
        """
 Takes the object returned by get_object() and returns the feed's
 copyright notice as a normal Python string.
 """

    def feed_copyright(self):
        """
 Returns the feed's copyright notice as a normal Python string.
 """

    feed_copyright = 'Copyright (c) 2007, Sally Smith' # Hard-coded copyright notice.

    # TTL -- One of the following three is optional. The framework looks
    # for them in this order. Ignored for Atom feeds.

    def ttl(self, obj):
        """
 Takes the object returned by get_object() and returns the feed's
 TTL (Time To Live) as a normal Python string.
 """

    def ttl(self):
        """
 Returns the feed's TTL as a normal Python string.
 """

    ttl = 600 # Hard-coded Time To Live.

    # ITEMS -- One of the following three is required. The framework looks
    # for them in this order.

    def items(self, obj):
        """
 Takes the object returned by get_object() and returns a list of
 items to publish in this feed.
 """

    def items(self):
        """
 Returns a list of items to publish in this feed.
 """

    items = ('Item 1', 'Item 2') # Hard-coded items.

    # GET_OBJECT -- This is required for feeds that publish different data
    # for different URL parameters. (See "A complex example" above.)

    def get_object(self, request, *args, **kwargs):
        """
 Takes the current request and the arguments from the URL, and
 returns an object represented by this feed. Raises
 django.core.exceptions.ObjectDoesNotExist on error.
 """

    # ITEM TITLE AND DESCRIPTION -- If title_template or
    # description_template are not defined, these are used instead. Both are
    # optional, by default they will use the unicode representation of the
    # item.

    def item_title(self, item):
        """
 Takes an item, as returned by items(), and returns the item's
 title as a normal Python string.
 """

    def item_title(self):
        """
 Returns the title for every item in the feed.
 """

    item_title = 'Breaking News: Nothing Happening' # Hard-coded title.

    def item_description(self, item):
        """
 Takes an item, as returned by items(), and returns the item's
 description as a normal Python string.
 """

    def item_description(self):
        """
 Returns the description for every item in the feed.
 """

    item_description = 'A description of the item.' # Hard-coded description.

    def get_context_data(self, **kwargs):
        """
 Returns a dictionary to use as extra context if either
 description_template or item_template are used.

 Default implementation preserves the old behavior
 of using {'obj': item, 'site': current_site} as the context.
 """

    # ITEM LINK -- One of these three is required. The framework looks for
    # them in this order.

    # First, the framework tries the two methods below, in
    # order. Failing that, it falls back to the get_absolute_url()
    # method on each item returned by items().

    def item_link(self, item):
        """
 Takes an item, as returned by items(), and returns the item's URL.
 """

    def item_link(self):
        """
 Returns the URL for every item in the feed.
 """

    # ITEM_GUID -- The following method is optional. If not provided, the
    # item's link is used by default.

    def item_guid(self, obj):
        """
 Takes an item, as return by items(), and returns the item's ID.
 """

    # ITEM_GUID_IS_PERMALINK -- The following method is optional. If
    # provided, it sets the 'isPermaLink' attribute of an item's
    # GUID element. This method is used only when 'item_guid' is
    # specified.

    def item_guid_is_permalink(self, obj):
        """
 Takes an item, as returned by items(), and returns a boolean.
 """

    item_guid_is_permalink = False  # Hard coded value

    # ITEM AUTHOR NAME -- One of the following three is optional. The
    # framework looks for them in this order.

    def item_author_name(self, item):
        """
 Takes an item, as returned by items(), and returns the item's
 author's name as a normal Python string.
 """

    def item_author_name(self):
        """
 Returns the author name for every item in the feed.
 """

    item_author_name = 'Sally Smith' # Hard-coded author name.

    # ITEM AUTHOR EMAIL --One of the following three is optional. The
    # framework looks for them in this order.
    #
    # If you specify this, you must specify item_author_name.

    def item_author_email(self, obj):
        """
 Takes an item, as returned by items(), and returns the item's
 author's email as a normal Python string.
 """

    def item_author_email(self):
        """
 Returns the author email for every item in the feed.
 """

    item_author_email = 'test@example.com' # Hard-coded author email.

    # ITEM AUTHOR LINK -- One of the following three is optional. The
    # framework looks for them in this order. In each case, the URL should
    # include the "http://" and domain name.
    #
    # If you specify this, you must specify item_author_name.

    def item_author_link(self, obj):
        """
 Takes an item, as returned by items(), and returns the item's
 author's URL as a normal Python string.
 """

    def item_author_link(self):
        """
 Returns the author URL for every item in the feed.
 """

    item_author_link = 'http://www.example.com/' # Hard-coded author URL.

    # ITEM ENCLOSURE URL -- One of these three is required if you're
    # publishing enclosures. The framework looks for them in this order.

    def item_enclosure_url(self, item):
        """
 Takes an item, as returned by items(), and returns the item's
 enclosure URL.
 """

    def item_enclosure_url(self):
        """
 Returns the enclosure URL for every item in the feed.
 """

    item_enclosure_url = "/foo/bar.mp3" # Hard-coded enclosure link.

    # ITEM ENCLOSURE LENGTH -- One of these three is required if you're
    # publishing enclosures. The framework looks for them in this order.
    # In each case, the returned value should be either an integer, or a
    # string representation of the integer, in bytes.

    def item_enclosure_length(self, item):
        """
 Takes an item, as returned by items(), and returns the item's
 enclosure length.
 """

    def item_enclosure_length(self):
        """
 Returns the enclosure length for every item in the feed.
 """

    item_enclosure_length = 32000 # Hard-coded enclosure length.

    # ITEM ENCLOSURE MIME TYPE -- One of these three is required if you're
    # publishing enclosures. The framework looks for them in this order.

    def item_enclosure_mime_type(self, item):
        """
 Takes an item, as returned by items(), and returns the item's
 enclosure MIME type.
 """

    def item_enclosure_mime_type(self):
        """
 Returns the enclosure MIME type for every item in the feed.
 """

    item_enclosure_mime_type = "audio/mpeg" # Hard-coded enclosure MIME type.

    # ITEM PUBDATE -- It's optional to use one of these three. This is a
    # hook that specifies how to get the pubdate for a given item.
    # In each case, the method/attribute should return a Python
    # datetime.datetime object.

    def item_pubdate(self, item):
        """
 Takes an item, as returned by items(), and returns the item's
 pubdate.
 """

    def item_pubdate(self):
        """
 Returns the pubdate for every item in the feed.
 """

    item_pubdate = datetime.datetime(2005, 5, 3) # Hard-coded pubdate.

    # ITEM UPDATED -- It's optional to use one of these three. This is a
    # hook that specifies how to get the updateddate for a given item.
    # In each case, the method/attribute should return a Python
    # datetime.datetime object.

    def item_updateddate(self, item):
        """
 Takes an item, as returned by items(), and returns the item's
 updateddate.
 """

    def item_updateddate(self):
        """
 Returns the updateddated for every item in the feed.
 """

    item_updateddate = datetime.datetime(2005, 5, 3) # Hard-coded updateddate.

    # ITEM CATEGORIES -- It's optional to use one of these three. This is
    # a hook that specifies how to get the list of categories for a given
    # item. In each case, the method/attribute should return an iterable
    # object that returns strings.

    def item_categories(self, item):
        """
 Takes an item, as returned by items(), and returns the item's
 categories.
 """

    def item_categories(self):
        """
 Returns the categories for every item in the feed.
 """

    item_categories = ("python", "django") # Hard-coded categories.

    # ITEM COPYRIGHT NOTICE (only applicable to Atom feeds) -- One of the
    # following three is optional. The framework looks for them in this
    # order.

    def item_copyright(self, obj):
        """
 Takes an item, as returned by items(), and returns the item's
 copyright notice as a normal Python string.
 """

    def item_copyright(self):
        """
 Returns the copyright notice for every item in the feed.
 """

    item_copyright = 'Copyright (c) 2007, Sally Smith' # Hard-coded copyright notice.

```

## 低级框架

在后台，高级RSS框架使用较低级别的框架来生成订阅源的XML。此框架存在于单个模块中：[django / utils / feedgenerator.py](https://github.com/django/django/blob/master/django/utils/feedgenerator.py)。

您自己使用此框架，用于生成较低级别的Feed。您还可以创建自定义Feed生成器子类，以与`feed_type` `Feed`选项一起使用。

### 联合供稿 classes

[`feedgenerator`](../utils.html#module-django.utils.feedgenerator "django.utils.feedgenerator: Syndication feed generation library -- used for generating RSS, etc.")模块包含基类：

*   [django.utils.feedgenerator.SyndicationFeed](../utils.html#django.utils.feedgenerator.SyndicationFeed "django.utils.feedgenerator.SyndicationFeed")

和几个子类：

*   [django.utils.feedgenerator.RssUserland091Feed](../utils.html#django.utils.feedgenerator.RssUserland091Feed "django.utils.feedgenerator.RssUserland091Feed")
*   [django.utils.feedgenerator.Rss201rev2Feed](../utils.html#django.utils.feedgenerator.Rss201rev2Feed "django.utils.feedgenerator.Rss201rev2Feed")
*   [django.utils.feedgenerator.Atom1Feed](../utils.html#django.utils.feedgenerator.Atom1Feed "django.utils.feedgenerator.Atom1Feed")

这三个类中的每一个都知道如何将某种类型的feed呈现为XML。他们共享这个接口：

[`SyndicationFeed.__init__()`](../utils.html#django.utils.feedgenerator.SyndicationFeed.__init__ "django.utils.feedgenerator.SyndicationFeed.__init__")

使用给定的元数据字典初始化Feed，该元数据字典应用于整个Feed。必需的关键字参数为：

*   `title`
*   `link`
*   `description`

还有一堆其他可选关键字：

*   `language`
*   `author_email`
*   `author_name`
*   `author_link`
*   `subtitle`
*   `categories`
*   `feed_url`
*   `feed_copyright`
*   `feed_guid`
*   `ttl`

您传递给`__init__`的任何其他关键字参数将存储在`self.feed`中以与[自定义Feed生成器](#custom-feed-generators)配合使用。

所有参数都应该是Unicode对象，除了`categories`，它应该是Unicode对象序列。

[`SyndicationFeed.add_item()`](../utils.html#django.utils.feedgenerator.SyndicationFeed.add_item "django.utils.feedgenerator.SyndicationFeed.add_item")

向具有给定参数的Feed中添加项目。

必需的关键字参数为：

*   `title`
*   `link`
*   `description`

可选的关键字参数为：

*   `author_email`
*   `author_name`
*   `author_link`
*   `pubdate`
*   `comments`
*   `unique_id`
*   `enclosure`
*   `categories`
*   `item_copyright`
*   `ttl`
*   `updateddate`

将为[自定义Feed生成器](#custom-feed-generators)存储额外的关键字参数。

所有参数，如果给定，应该是Unicode对象，除了：

*   `pubdate`应为Python [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)")对象。
*   `updateddate`应为Python [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)")对象。
*   `enclosure`应为[`django.utils.feedgenerator.Enclosure`](../utils.html#django.utils.feedgenerator.Enclosure "django.utils.feedgenerator.Enclosure")的实例。
*   `categories`应为Unicode对象序列。

New in Django 1.7:

已添加可选的`updateddate`参数。

[`SyndicationFeed.write()`](../utils.html#django.utils.feedgenerator.SyndicationFeed.write "django.utils.feedgenerator.SyndicationFeed.write")

Outputs the feed in the given encoding to outfile, which is a file-like object.

[`SyndicationFeed.writeString()`](../utils.html#django.utils.feedgenerator.SyndicationFeed.writeString "django.utils.feedgenerator.SyndicationFeed.writeString")

Returns the feed as a string in the given encoding.

例如，要创建Atom 1.0订阅源并将其打印到标准输出：

```
>>> from django.utils import feedgenerator
>>> from datetime import datetime
>>> f = feedgenerator.Atom1Feed(
...     title="My Weblog",
...     link="http://www.example.com/",
...     description="In which I write about what I ate today.",
...     language="en",
...     author_name="Myself",
...     feed_url="http://example.com/atom.xml")
>>> f.add_item(title="Hot dog today",
...     link="http://www.example.com/entries/1/",
...     pubdate=datetime.now(),
...     description="<p>Today I had a Vienna Beef hot dog. It was pink, plump and perfect.</p>")
>>> print(f.writeString('UTF-8'))
<?xml version="1.0" encoding="UTF-8"?>
<feed xmlns="http://www.w3.org/2005/Atom" xml:lang="en">
...
</feed>

```

### 自定义Feed生成器

如果您需要生成自定义Feed格式，您有几个选项。

如果Feed格式是完全自定义的，您需要将`SyndicationFeed`作为子类，并完全替换`write()`和`writeString()`方法。

但是，如果Feed格式是RSS或Atom分拆（即[GeoRSS](http://georss.org/)，Apple的[iTunes podcast格式](http://www.apple.com/itunes/podcasts/specs.html)等），则您有了更好的选择。这些类型的Feed通常会向底层格式添加额外的元素和/或属性，并且有一组方法可以`SyndicationFeed`调用以获取这些额外的属性。因此，您可以对相应的Feed生成器类（`Atom1Feed`或`Rss201rev2Feed`）进行子类化，并扩展这些回调。他们是：

`SyndicationFeed.root_attributes(self, )`

Return a `dict` of attributes to add to the root feed element (`feed`/`channel`).

`SyndicationFeed.add_root_elements(self, handler)`

Callback to add elements inside the root feed element (`feed`/`channel`). `handler` is an [`XMLGenerator`](https://docs.python.org/3/library/xml.sax.utils.html#xml.sax.saxutils.XMLGenerator "(in Python v3.4)") from Python’s built-in SAX library; you’ll call methods on it to add to the XML document in process.

`SyndicationFeed.item_attributes(self, item)`

Return a `dict` of attributes to add to each item (`item`/`entry`) element. The argument, `item`, is a dictionary of all the data passed to `SyndicationFeed.add_item()`.

`SyndicationFeed.add_item_elements(self, handler, item)`

Callback to add elements to each item (`item`/`entry`) element. `handler` and `item` are as above.

警告

如果您覆盖任何这些方法，请务必调用超类方法，因为它们为每种Feed格式添加了必需的元素。

例如，您可能开始实现iTunes RSS Feed生成器，如：

```
class iTunesFeed(Rss201rev2Feed):
    def root_attributes(self):
        attrs = super(iTunesFeed, self).root_attributes()
        attrs['xmlns:itunes'] = 'http://www.itunes.com/dtds/podcast-1.0.dtd'
        return attrs

    def add_root_elements(self, handler):
        super(iTunesFeed, self).add_root_elements(handler)
        handler.addQuickElement('itunes:explicit', 'clean')

```

显然，对于一个完整的自定义feed类，还有很多工作要做，但上面的例子应该展示基本的想法。

{% endraw %}