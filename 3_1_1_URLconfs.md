# URL调度器 #

简洁、优雅的URL 模式在高质量的Web 应用中是一个非常重要的细节。Django 允许你任意设计你的URL，不受框架束缚。

不要求有`.php` 或`.cgi`，更不会要求类似`0,2097,1-1-1928,00` 这样无意义的东西。

参见万维网的发明者Berners-Lee 的[Cool URIs don’t change](http://www.w3.org/Provider/Style/URI)，里面有关于为什么URL 应该保持整洁和有意义的卓越的论证。

## 概览 ##

为了给一个应用设计URL，你需要创建一个Python 模块，通常称为URLconf（URL configuration）。这个模块是纯粹的Python 代码，包含URL 模式（简单的正则表达式）到Python 函数（你的视图）的简单映射。

映射可短可长，随便你。它可以引用其它的映射。而且，因为它是纯粹的Python 代码，它可以动态构造。

Django 还提供根据当前语言翻译URL 的一种方法。更多信息参见[国际化文档](http://python.usyiyi.cn/django/topics/i18n/translation.html#url-internationalization)。

## Django 如何处理一个请求 ##

当一个用户请求Django 站点的一个页面，下面是Django 系统决定执行哪个Python 代码使用的算法：

1. Django 决定要使用的根`URLconf` 模块。通常，这个值就是`ROOT_URLCONF` 的设置，但是如果进来的`HttpRequest` 对象具有一个`urlconf` 属性（通过中间件`request` `processing` 设置），则使用这个值来替换`ROOT_URLCONF` 设置。
2. Django 加载该Python 模块并寻找可用的`urlpatterns`。它是`django.conf.urls.url()` 实例的一个Python 列表。
3. Django 依次匹配每个URL 模式，在与请求的URL 匹配的第一个模式停下来。
4. 一旦其中的一个正则表达式匹配上，Django 将导入并调用给出的视图，它是一个简单的Python 函数（或者一个基于类的视图）。视图将获得如下参数:
  + 一个`HttpRequest` 实例。
  + 如果匹配的正则表达式没有返回命名的组，那么正则表达式匹配的内容将作为位置参数提供给视图。
  +  关键字参数由正则表达式匹配的命名组组成，但是可以被`django.conf.urls.url()`的可选参数`kwargs`覆盖。
5. 如果没有匹配到正则表达式，或者如果过程中抛出一个异常，Django 将调用一个适当的错误处理视图。请参见下面的错误处理。

## 例子 ##

下面是一个简单的 URLconf：

```
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^articles/2003/$', views.special_case_2003),
    url(r'^articles/([0-9]{4})/$', views.year_archive),
    url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive),
    url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),
]
```

注：

+ 若要从URL 中捕获一个值，只需要在它周围放置一对圆括号。
+ 不需要添加一个前导的反斜杠，因为每个URL 都有。例如，应该是`^articles` 而不是 `^/articles`。
+ 每个正则表达式前面的'r' 是可选的但是建议加上。它告诉Python 这个字符串是“原始的” —— 字符串中任何字符都不应该转义。参见Dive Into Python 中的解释。

一些请求的例子：

+ `/articles/2005/03/` 请求将匹配列表中的第三个模式。Django 将调用函数`views.month_archive(request, '2005', '03')`。
+ `/articles/2005/3/` 不匹配任何URL 模式，因为列表中的第三个模式要求月份应该是两个数字。
+ `/articles/2003/` 将匹配列表中的第一个模式不是第二个，因为模式按顺序匹配，第一个会首先测试是否匹配。请像这样自由插入一些特殊的情况来探测匹配的次序。
+ `/articles/2003` 不匹配任何一个模式，因为每个模式要求URL 以一个反斜线结尾。
+ `/articles/2003/03/03/` 将匹配最后一个模式。Django 将调用函数`views.article_detail(request, '2003', '03', '03')`。

## 命名组 ##

上面的示例使用简单的、没有命名的正则表达式组（通过圆括号）来捕获URL 中的值并以位置 参数传递给视图。在更高级的用法中，可以使用命名的正则表达式组来捕获URL  中的值并以关键字 参数传递给视图。

在Python 正则表达式中，命名正则表达式组的语法是`(?P<name>pattern)`，其中`name` 是组的名称，`pattern` 是要匹配的模式。

下面是以上URLconf  使用命名组的重写：

```
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^articles/2003/$', views.special_case_2003),
    url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<day>[0-9]{2})/$', views.article_detail),
]
```

这个实现与前面的示例完全相同，只有一个细微的差别：捕获的值作为关键字参数而不是位置参数传递给视图函数。例如：

+ `/articles/2005/03/` 请求将调用`views.month_archive(request, year='2005', month='03')`函数，而不是`views.month_archive(request, '2005', '03')`。
+ `/articles/2003/03/03/` 请求将调用函数`views.article_detail(request, year='2003', month='03', day='03')`。

在实际应用中，这意味你的URLconf 会更加明晰且不容易产生参数顺序问题的错误 —— 你可以在你的视图函数定义中重新安排参数的顺序。当然，这些好处是以简洁为代价；有些开发人员认为命名组语法丑陋而繁琐。

### 匹配/分组算法 ###

下面是URLconf 解析器使用的算法，针对正则表达式中的命名组和非命名组：

1. 如果有命名参数，则使用这些命名参数，忽略非命名参数。
2. 否则，它将以位置参数传递所有的非命名参数。

根据[传递额外的选项给视图函数](http://python.usyiyi.cn/django/topics/http/urls.html#passing-extra-options-to-view-functions)（下文），这两种情况下，多余的关键字参数也将传递给视图。

## URLconf 在什么上查找 ##

URLconf 在请求的URL 上查找，将它当做一个普通的Python 字符串。不包括GET和POST参数以及域名。

例如，http://www.example.com/myapp/ 请求中，URLconf 将查找`myapp/`。

在 http://www.example.com/myapp/?page=3 请求中，URLconf 仍将查找`myapp/`。

URLconf 不检查请求的方法。换句话讲，所有的请求方法 —— 同一个URL的`POST`、`GET`、`HEAD`等等 —— 都将路由到相同的函数。

## 捕获的参数永远是字符串 ##

每个捕获的参数都作为一个普通的Python 字符串传递给视图，无论正则表达式使用的是什么匹配方式。例如，下面这行URLconf 中：

```
url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
```

... `views.year_archive()` 的`year` 参数将是一个字符串，即使`[0-9]{4}` 值匹配整数字符串。

## 指定视图参数的默认值 ##

有一个方便的小技巧是指定视图参数的默认值。 下面是一个URLconf 和视图的示例：

```
# URLconf
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^blog/$', views.page),
    url(r'^blog/page(?P<num>[0-9]+)/$', views.page),
]

# View (in blog/views.py)
def page(request, num="1"):
    # Output the appropriate page of blog entries, according to num.
    ...
```

在上面的例子中，两个URL模式指向同一个视图`views.page` —— 但是第一个模式不会从URL 中捕获任何值。如果第一个模式匹配，`page()` 函数将使用`num`参数的默认值"1"。如果第二个模式匹配，`page()` 将使用正则表达式捕获的`num` 值。

## 性能 ##

`urlpatterns` 中的每个正则表达式在第一次访问它们时被编译。这使得系统相当快。

## urlpatterns 变量的语法 ##

`urlpatterns` 应该是`url()` 实例的一个Python 列表。

## 错误处理 ##

当Django 找不到一个匹配请求的URL 的正则表达式时，或者当抛出一个异常时，Django 将调用一个错误处理视图。

这些情况发生时使用的视图通过4个变量指定。它们的默认值应该满足大部分项目，但是通过赋值给它们以进一步的自定义也是可以的。

完整的细节请参见[自定义错误视图](http://python.usyiyi.cn/django/topics/http/views.html#customizing-error-views)。

这些值可以在你的根URLconf 中设置。在其它URLconf 中设置这些变量将不会生效果。

它们的值必须是可调用的或者是表示视图的Python 完整导入路径的字符串，可以方便地调用它们来处理错误情况。

这些值是：

+ `handler404` —— 参见`django.conf.urls.handler404`。
+ `handler500` —— 参见`django.conf.urls.handler500`。
+ `handler403` —— 参见`django.conf.urls.handler403`。
+ `handler400` —— 参见`django.conf.urls.handler400`。

## 包含其它的URLconfs ##

在任何时候，你的urlpatterns 都可以包含其它URLconf 模块。这实际上将一部分URL 放置与其它URL 下面。

例如，下面是URLconf for the Django 网站自己的URLconf 中一个片段。它包含许多其它URLconf：

```
from django.conf.urls import include, url

urlpatterns = [
    # ... snip ...
    url(r'^community/', include('django_website.aggregator.urls')),
    url(r'^contact/', include('django_website.contact.urls')),
    # ... snip ...
]
```

注意，这个例子中的正则表达式没有包含$（字符串结束匹配符），但是包含一个末尾的反斜杠。每当Django 遇到`include()`（`django.conf.urls.include()`）时，它会去掉URL 中匹配的部分并将剩下的字符串发送给包含的URLconf 做进一步处理。

另外一种包含其它URL 模式的方式是使用一个url() 实例的列表。例如，请看下面的URLconf：

```
from django.conf.urls import include, url

from apps.main import views as main_views
from credit import views as credit_views

extra_patterns = [
    url(r'^reports/(?P<id>[0-9]+)/$', credit_views.report),
    url(r'^charge/$', credit_views.charge),
]

urlpatterns = [
    url(r'^$', main_views.homepage),
    url(r'^help/', include('apps.help.urls')),
    url(r'^credit/', include(extra_patterns)),
]
```

在这个例子中，`/credit/reports/` URL将被 `credit.views.report()` 这个Django 视图处理。

这种方法可以用来去除URLconf 中的冗余，其中某个模式前缀被重复使用。例如，考虑这个URLconf：

```
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^(?P<page_slug>[\w-]+)-(?P<page_id>\w+)/history/$', views.history),
    url(r'^(?P<page_slug>[\w-]+)-(?P<page_id>\w+)/edit/$', views.edit),
    url(r'^(?P<page_slug>[\w-]+)-(?P<page_id>\w+)/discuss/$', views.discuss),
    url(r'^(?P<page_slug>[\w-]+)-(?P<page_id>\w+)/permissions/$', views.permissions),
]
```

我们可以改进它，通过只声明共同的路径前缀一次并将后面的部分分组：

```
from django.conf.urls import include, url
from . import views

urlpatterns = [
    url(r'^(?P<page_slug>[\w-]+)-(?P<page_id>\w+)/', include([
        url(r'^history/$', views.history),
        url(r'^edit/$', views.edit),
        url(r'^discuss/$', views.discuss),
        url(r'^permissions/$', views.permissions),
    ])),
]
```

### 捕获的参数 ###

被包含的URLconf 会收到来之父URLconf 捕获的任何参数，所以下面的例子是合法的：

```
# In settings/urls/main.py
from django.conf.urls import include, url

urlpatterns = [
    url(r'^(?P<username>\w+)/blog/', include('foo.urls.blog')),
]

# In foo/urls/blog.py
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^$', views.blog.index),
    url(r'^archive/$', views.blog.archive),
]
```

在上面的例子中，捕获的"`username`"变量将被如期传递给包含的 URLconf。

## 嵌套的参数 ##

正则表达式允许嵌套的参数，Django 将解析它们并传递给视图。当反查时，Django 将尝试填满所有外围捕获的参数，并忽略嵌套捕获的参数。考虑下面的URL 模式，它带有一个可选的`page` 参数：

```
from django.conf.urls import url

urlpatterns = [
    url(r'blog/(page-(\d+)/)?$', blog_articles),                  # bad
    url(r'comments/(?:page-(?P<page_number>\d+)/)?$', comments),  # good
]
```

两个模式都使用嵌套的参数，其解析方式是：例如`blog/page-2/` 将匹配`blog_articles`并带有两个位置参数`page-2/` 和2。第二个`comments` 的模式将匹配`comments/page-2/` 并带有一个值为2 的关键字参数`page_number`。这个例子中外围参数是一个不捕获的参数`(?:...)`。

`blog_articles` 视图需要最外层捕获的参数来反查，在这个例子中是`page-2/`或者没有参数，而`comments`可以不带参数或者用一个`page_number`值来反查。

嵌套捕获的参数使得视图参数和URL 之间存在强耦合，正如`blog_articles` 所示：视图接收URL（`page-2/`）的一部分，而不只是视图感兴趣的值。这种耦合在反查时更加显著，因为反查视图时我们需要传递URL 的一个片段而不只是page 的值。

作为一个经验的法则，当正则表达式需要一个参数但视图忽略它的时候，只捕获视图需要的值并使用非捕获参数。

## 传递额外的选项给视图函数 ##

URLconfs 具有一个钩子，让你传递一个Python 字典作为额外的参数传递给视图函数。

`django.conf.urls.url()` 函数可以接收一个可选的第三个参数，它是一个字典，表示想要传递给视图函数的额外关键字参数。

例如：

```
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^blog/(?P<year>[0-9]{4})/$', views.year_archive, {'foo': 'bar'}),
]
```

在这个例子中，对于`/blog/2005/`请求，Django 将调用`views.year_archive(request, year='2005', foo='bar')`。

这个技术在[Syndication 框架](http://python.usyiyi.cn/django/ref/contrib/syndication.html) 中使用，来传递元数据和选项给视图。

> 处理冲突
>
> URL 模式捕获的命名关键字参数和在字典中传递的额外参数有可能具有相同的名称。当这种情况发生时，将使用字典中的参数而不是URL 中捕获的参数。

## 传递额外的选项给include() ##

类似地，你可以传递额外的选项给include()。当你传递额外的选项给include() 时，被包含的URLconf 的每一 行将被传递这些额外的选项。

例如，下面两个URLconf 设置功能上完全相同：

设置一次：

```
# main.py
from django.conf.urls import include, url

urlpatterns = [
    url(r'^blog/', include('inner'), {'blogid': 3}),
]

# inner.py
from django.conf.urls import url
from mysite import views

urlpatterns = [
    url(r'^archive/$', views.archive),
    url(r'^about/$', views.about),
]
```

设置两次：

```
# main.py
from django.conf.urls import include, url
from mysite import views

urlpatterns = [
    url(r'^blog/', include('inner')),
]

# inner.py
from django.conf.urls import url

urlpatterns = [
    url(r'^archive/$', views.archive, {'blogid': 3}),
    url(r'^about/$', views.about, {'blogid': 3}),
]
```

注意，额外的选项将永远传递给被包含的URLconf 中的每一行，无论该行的视图实际上是否认为这些选项是合法的。由于这个原因，该技术只有当你确定被包含的URLconf 中的每个视图都接收你传递给它们的额外的选项。

## URL 的反向解析 ##

在使用Django 项目时，一个常见的需求是获得URL 的最终形式，以用于嵌入到生成的内容中（视图中和显示给用户的URL等）或者用于处理服务器端的导航（重定向等）。

人们强烈希望不要硬编码这些URL（费力、不可扩展且容易产生错误）或者设计一种与URLconf 毫不相关的专门的URL 生成机制，因为这样容易导致一定程度上产生过期的URL。

换句话讲，需要的是一个DRY 机制。除了其它有点，它还允许设计的URL 可以自动更新而不用遍历项目的源代码来搜索并替换过期的URL。

获取一个URL 最开始想到的信息是处理它视图的标识（例如名字），查找正确的URL 的其它必要的信息有视图参数的类型（位置参数、关键字参数）和值。

Django 提供一个办法是让URL 映射是URL 设计唯一的地方。你填充你的URLconf，然后可以双向使用它：

+ 根据用户/浏览器发起的URL 请求，它调用正确的Django 视图，并从URL 中提取它的参数需要的值。
+ 根据Django 视图的标识和将要传递给它的参数的值，获取与之关联的URL。

第一种方式是我们在前面的章节中一直讨论的用法。第二种方式叫做反向解析URL、反向URL 匹配、反向URL 查询或者简单的URL 反查。

在需要URL 的地方，对于不同层级，Django 提供不同的工具用于URL 反查：

+ 在模板中：使用url 模板标签。
+ 在Python 代码中：使用`django.core.urlresolvers.reverse()` 函数。
+ 在更高层的与处理Django 模型实例相关的代码中：使用`get_absolute_url()` 方法。

例子

考虑下面的URLconf：

```
from django.conf.urls import url

from . import views

urlpatterns = [
    #...
    url(r'^articles/([0-9]{4})/$', views.year_archive, name='news-year-archive'),
    #...
]
```

根据这里的设计，某一年nnnn对应的归档的URL是`/articles/nnnn/`。

你可以在模板的代码中使用下面的方法获得它们：

```
<a href="{% url 'news-year-archive' 2012 %}">2012 Archive</a>

<ul>
{% for yearvar in year_list %}
<li><a href="{% url 'news-year-archive' yearvar %}">{{ yearvar }} Archive</a></li>
{% endfor %}
</ul>
```

在Python 代码中，这样使用：

```
from django.core.urlresolvers import reverse
from django.http import HttpResponseRedirect

def redirect_to_year(request):
    # ...
    year = 2006
    # ...
    return HttpResponseRedirect(reverse('news-year-archive', args=(year,)))
```

如果出于某种原因决定按年归档文章发布的URL应该调整一下，那么你将只需要修改URLconf 中的内容。

在某些场景中，一个视图是通用的，所以在URL 和视图之间存在多对一的关系。对于这些情况，当反查URL 时，只有视图的名字还不够。请阅读下一节来了解Django 为这个问题提供的解决办法。

## 命名URL 模式 ##

为了完成上面例子中的URL 反查，你将需要使用命名的URL 模式。URL 的名称使用的字符串可以包含任何你喜欢的字符。不只限制在合法的Python 名称。

当命名你的URL 模式时，请确保使用的名称不会与其它应用中名称冲突。如果你的URL 模式叫做`comment`，而另外一个应用中也有一个同样的名称，当你在模板中使用这个名称的时候不能保证将插入哪个URL。

在URL 名称中加上一个前缀，比如应用的名称，将减少冲突的可能。我们建议使用`myapp-comment` 而不是`comment`。

## URL 命名空间 ##

### 简介 ###

URL 命名空间允许你反查到唯一的[命名URL 模式](http://python.usyiyi.cn/django/topics/http/urls.html#naming-url-patterns)，即使不同的应用使用相同的URL 名称。第三方应用始终使用带命名空间的URL 是一个很好的实践（我们在教程中也是这么做的）。类似地，它还允许你在一个应用有多个实例部署的情况下反查URL。换句话讲，因为一个应用的多个实例共享相同的命名URL，命名空间将提供一种区分这些命名URL 的方法。

在一个站点上，正确使用URL 命名空间的Django 应用可以部署多次。例如，`django.contrib.admin` 具有一个`AdminSite` 类，它允许你很容易地[部署多个管理站点的实例](http://python.usyiyi.cn/django/ref/contrib/admin/index.html#multiple-admin-sites)。在下面的例子中，我们将讨论在两个不同的地方部署教程中的polls 应用，这样我们可以为两种不同的用户（作者和发布者）提供相同的功能。

一个URL 命名空间有两个部分，它们都是字符串：

**应用命名空间**

它表示正在部署的应用的名称。一个应用的每个实例具有相同的应用命名空间。例如，可以预见Django 的管理站点的应用命名空间是'`admin`'。

**实例命名空间**

它表示应用的一个特定的实例。实例的命名空间在你的全部项目中应该是唯一的。但是，一个实例的命名空间可以和应用的命名空间相同。它用于表示一个应用的默认实例。例如，Django 管理站点实例具有一个默认的实例命名空间'admin'。
URL 的命名空间使用':' 操作符指定。例如，管理站点应用的主页使用'`admin:index`'。它表示'`admin`' 的一个命名空间和'`index`' 的一个命名URL。

命名空间也可以嵌套。命名URL'`sports:polls:index`' 将在命名空间'`polls`'中查找'`index`'，而`poll` 定义在顶层的命名空间'`sports`' 中。

### 反查带命名空间的URL ###

当解析一个带命名空间的URL（例如'`polls:index`'）时，Django 将切分名称为多个部分，然后按下面的步骤查找：

1. 首先，Django 查找匹配的应用的命名空间(在这个例子中为'`polls`'）。这将得到该应用实例的一个列表。

2. 如果有定义当前 应用，Django 将查找并返回那个实例的URL 解析器。当前 应用可以通过请求上的一个属性指定。希望可以多次部署的应用应该设置正在处理的`request`上的`current_app` 属性。

```
Changed in Django 1.8:

在以前版本的Django 中，你必须在用于渲染模板的每个`Context` 或 `RequestContext`上设置`current_app` 属性。
```

当前应用还可以通过`reverse()` 函数的一个参数手工设定。

3. 如果没有当前应用。Django 将查找一个默认的应用实例。默认的应用实例是[实例命名空间](http://python.usyiyi.cn/django/topics/http/urls.html#term-instance-namespace) 与[应用命名空间](http://python.usyiyi.cn/django/topics/http/urls.html#term-application-namespace) 一致的那个实例（在这个例子中，`polls` 的一个叫做'`polls`' 的实例）。

4. 如果没有默认的应用实例，Django 将该应用挑选最后部署的实例，不管实例的名称是什么。

5. 如果提供的命名空间与第1步中的[应用命名空间](http://python.usyiyi.cn/django/topics/http/urls.html#term-application-namespace) 不匹配，Django 将尝试直接将此命名空间作为一个[实例命名空间](http://python.usyiyi.cn/django/topics/http/urls.html#term-instance-namespace)查找。

如果有嵌套的命名空间，将为命名空间的每个部分重复调用这些步骤直至剩下视图的名称还未解析。然后该视图的名称将被解析到找到的这个命名空间中的一个URL。

### 例子 ###

为了演示解析的策略，考虑教程中polls 应用的两个实例：'`author-polls`' 和'`publisher-polls`'。假设我们已经增强了该应用，在创建和显示投票时考虑了实例命名空间。

```
#urls.py

from django.conf.urls import include, url

urlpatterns = [
    url(r'^author-polls/', include('polls.urls', namespace='author-polls', app_name='polls')),
    url(r'^publisher-polls/', include('polls.urls', namespace='publisher-polls', app_name='polls')),
]
```

```
#polls/urls.py

from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^$', views.IndexView.as_view(), name='index'),
    url(r'^(?P<pk>\d+)/$', views.DetailView.as_view(), name='detail'),
    ...
]
```

根据以上设置，可以使用下面的查询：

+ 如果其中一个实例是当前实例 —— 如果我们正在渲染'`author-polls`' 实例的`detail` 页面 —— '`polls:index`' 将解析成'`author-polls`' 实例的主页面；例如下面两个都将解析成"`/author-polls/`"。

在基于类的视图的方法中：

```
reverse('polls:index', current_app=self.request.resolver_match.namespace)
```

和在模板中：

```
{% url 'polls:index' %}
```

注意，在模板中的反查需要添加`request` 的`current_app` 属性，像这样：

```
def render_to_response(self, context, **response_kwargs):
    self.request.current_app = self.request.resolver_match.namespace
    return super(DetailView, self).render_to_response(context, **response_kwargs)
```

+ 如果没有当前实例 —— 如果我们在站点的其它地方渲染一个页面 —— '`polls:index`' 将解析到最后注册的`polls`的一个实例。因为没有默认的实例（命名空间为'polls'的实例），将使用注册的`polls` 的最后一个实例。它将是'`publisher-polls`'，因为它是在`urlpatterns`中最后一个声明的。
+ '`author-polls:index`' 将永远解析到 '`author-polls`' 实例的主页（'`publisher-polls`' 类似）。

如果还有一个默认的实例 —— 例如，一个名为'`polls`' 的实例 —— 上面例子中唯一的变化是当没有当前实例的情况（上述第二种情况）。在这种情况下 '`polls:index`' 将解析到默认实例而不是`urlpatterns` 中最后声明的实例的主页。

### URL 命名空间和被包含的URLconf ###

被包含的URLconf 的命名空间可以通过两种方式指定。

首先，在你构造你的URL 模式时，你可以提供 应用 和 实例的命名空间给`include()` 作为参数。例如：

```
url(r'^polls/', include('polls.urls', namespace='author-polls', app_name='polls')),
```

这将包含`polls.urls` 中定义的URL 到应用命名空间 '`polls`'中，其实例命名空间为'`author-polls`'。

其次，你可以include 一个包含嵌套命名空间数据的对象。如果你`include()` 一个`url()` 实例的列表，那么该对象中包含的URL 将添加到全局命名空间。然而，你还可以`include()` 一个3个元素的元组：

```
(<list of url() instances>, <application namespace>, <instance namespace>)
```

例如：

```
from django.conf.urls import include, url

from . import views

polls_patterns = [
    url(r'^$', views.IndexView.as_view(), name='index'),
    url(r'^(?P<pk>\d+)/$', views.DetailView.as_view(), name='detail'),
]

url(r'^polls/', include((polls_patterns, 'polls', 'author-polls'))),
```

这将include 命名的URL 模式到给定的应用和实例命名空间中。

例如，Django 的管理站点部署的实例叫`AdminSite`。`AdminSite` 对象具有一个`urls` 属性：一个3元组，包含管理站点中的所有URL 模式和应用的命名空间'`admin`'以及管理站点实例的名称。你`include()`到你项目的`urlpattern`s 中的是这个`urls` 属性。

请确保传递一个元组给`include()`。如果你只是传递3个参数：`include(polls_patterns, 'polls', 'author-polls')`，Django 不会抛出一个错误，但是根据`include()` 的功能，'`polls`' 将是实例的命名空间而'`author-polls`' 将是应用的命名空间，而不是反过来的。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[URLconfs](https://docs.djangoproject.com/en/1.8/topics/http/urls/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
