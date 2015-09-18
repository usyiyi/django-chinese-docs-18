# 按需内容处理

HTTP客户端可能发送一些协议头来告诉服务端它们已经看过了哪些资源。这在获取网页（使用HTTP`GET`请求）时非常常见，可以避免发送客户端已经获得的完整数据。然而，相同的协议头可用于所有HTTP方法(`POST`, `PUT`, `DELETE`, 以及其它)。

对于每一个Django从视图发回的页面（响应），都会提供两个HTTP协议头：`ETag`和`Last-Modified`。这些协议头在HTTP响应中是可选的。它们可以由你的视图函数设置，或者你可以依靠&nbsp;[`CommonMiddleware`](../ref/middleware.html#django.middleware.common.CommonMiddleware "django.middleware.common.CommonMiddleware") 中间件来设置`ETag` 协议头。

当你的客户端再次请求相同的资源时，它可能会发送&nbsp;[If-modified-since](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.25) 或者[If-unmodified-since](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.28)的协议头，包含之前发送的最后修改时间；或者&nbsp;[If-match](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.24) 或[If-none-match](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.26)协议头，包含之前发送的`ETag`。如果页面的当前版本匹配客户端发送的`ETag`，或者如果资源没有被修改，会发回304状态码，而不是一个完整的回复，告诉客户端没有任何修改。根据协议头，如果页面被修改了，或者不匹配客户端发送的&nbsp;`ETag`，会返回412（先决条件失败，Precondition Failed）状态码。

当你需要更多精细化的控制时，你可以使用每个视图的按需处理函数。

Changed in Django 1.8:

向按需视图处理添加`If-unmodified-since`协议头的支持

## The `condition`

有时（实际上是经常），你可以创建一些函数来快速计算出资源的[ETag](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.11)值或者最后修改时间，**并不**需要执行构建完整视图所需的所有步骤。Django可以使用这些函数来为视图处理提供一个“early bailout”的选项。来告诉客户端，内容自从上次请求并没有任何改动。

这两个函数作为参数传递到`django.views.decorators.http.condition`装饰器中。这个装时期使用这两个函数（如果你不能既快又容易得计算出来，你只需要提供一个）来弄清楚是否HTTP请求中的协议头匹配那些资源。如果它们不匹配，会生成资源的一份新的副本，并调用你的普通视图。

`condition`装饰器的签名为i：

```
condition(etag_func=None, last_modified_func=None)

```

计算ETag的最后修改时间的两个函数，会以相同的顺序传入`request`对象和相同的参数，就像它们封装的视图函数那样。`last_modified_func`函数应该返回一个标准的datetime值，它制订了资源修改的最后时间，或者资源不存在为&nbsp;`None`。传递给`etag`装饰器的函数应该返回一个表示资源[Etag](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.11)的字符串，或者资源不存在时为`None`。

用一个例子可以很好展示如何使用这一特性。假设你有这两个模型，表示一个简单的博客系统：

```
import datetime
from django.db import models

class Blog(models.Model):
    ...

class Entry(models.Model):
    blog = models.ForeignKey(Blog)
    published = models.DateTimeField(default=datetime.datetime.now)
    ...

```

如果头版展示最后的博客文章，仅仅在你添加新文章的时候修改，你可以非常快速地计算出最后修改时间。你需要这个博客每一篇文章的最后&nbsp;`发布` 日期。实现它的一种方式是：

```
def latest_entry(request, blog_id):
    return Entry.objects.filter(blog=blog_id).latest("published").published

```

接下来你可以使用这个函数，来为你的头版视图事先探测未修改的页面：

```
from django.views.decorators.http import condition

@condition(last_modified_func=latest_entry)
def front_page(request, blog_id):
    ...

```

## 只计算一个值的快捷方式

一个普遍的原则是，如果你提供了计算&nbsp;ETag_和_最后修改时间的函数，你应该这样做：你并不知道HTTP客户端会发给你哪个协议头，所以要准备好处理两种情况。但是，有时只有二者之一容易计算，并且Django只提供给你计算ETag或最后修改日期的装饰器。

`django.views.decorators.http.etag` 和`django.views.decorators.http.last_modified`作为`condition`装饰器，传入相同类型的函数。他们的签名是：

```
etag(etag_func)
last_modified(last_modified_func)

```

我们可以编写一个初期的示例，它仅仅使用最后修改日期的函数，使用这些装饰器之一：

```
@last_modified(latest_entry)
def front_page(request, blog_id):
    ...

```

...或者：

```
def front_page(request, blog_id):
    ...
front_page = last_modified(latest_entry)(front_page)

```

### Use `condition`

如果你想要测试两个先决条件，把`etag` 和`last_modified`装饰器链到一起看起来很不错。但是，这会导致不正确的行为：

```
# Bad code. Don't do this!
@etag(etag_func)
@last_modified(last_modified_func)
def my_view(request):
    # ...

# End of bad code.

```

第一个装饰器不知道后面的任何事情，并且可能发送“未修改”的响应，即使第二个装饰器会处理别的事情。`condition`装饰器同时更使用两个回调函数，来弄清楚哪个是正确的行为。

## 使用带有其它HTTP方法的装饰器

`condition`装饰器不仅仅对`GET` 和 `HEAD`请求有用（`HEAD`请求在这种情况下和`GET`相同）。它也可以用于为 `POST`, `PUT` 和 `DELETE`请求提供检查。在这些情况下，不是要返回一个“未修改（not modified，314）”的响应，而是要告诉服务端，它们尝试修改的资源在此期间被修改了。

例如，考虑以下客户端和服务端之间的交互：

1.  客户端请求`/foo/`。
2.  服务端回复一些带有`"abcd1234"`ETag的内容。
3.  客户端发送HTTP `PUT` 请求到 `/foo/` 来更新资源。同时也发送了`If-Match: "abcd1234"` 协议头来指定尝试更新的版本。
4.  服务端检查是否资源已经被修改，通过和`GET` 上所做的相同方式计算ETag（使用相同的函数）。如果资源 _已经_ 修改了，会返回412状态码，意思是“先决条件失败（precondition failed）”。
5.  客户端在接收到412响应之后，发送 `GET`请求到 `/foo/`，来在更新之前获取内容的新版本。

重要的事情是，这个例子展示了在所有情况下，ETag和最后修改时间值都采用相同函数计算。实际上，你 **应该** 使用相同函数，以便每次都返回相同的值。

## 使用中间件按需处理来比较

你可能注意到，Django已经通过[`django.middleware.http.ConditionalGetMiddleware`](../ref/middleware.html#django.middleware.http.ConditionalGetMiddleware "django.middleware.http.ConditionalGetMiddleware") 和 [`CommonMiddleware`](../ref/middleware.html#django.middleware.common.CommonMiddleware "django.middleware.common.CommonMiddleware").提供了简单和直接的`GET` 的按需处理。这些中间件易于使用并且适用于多种情况，然而它们的功能有一些高级用法上的限制：

*   它们在全局上用于你项目中的所有视图。
*   它们不会代替你生成响应本身，这可能要花一些代价。
*   它们只适用于HTTP `GET` 请求。

在这里，你应该选择最适用于你特定问题的工具。如果你有办法快速计算出ETag和修改时间，并且如果一些视图需要花一些时间来生成内容，你应该考虑使用这篇文档描述的`condition`装饰器。如果一些都执行得非常快，坚持使用中间件在如果视图没有修改的条件下也会使发回客户端的网络流量也会减少。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Conditional content processing](https://docs.djangoproject.com/en/1.8/topics/conditional-view-processing/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
