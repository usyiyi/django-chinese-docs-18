

# Request 对象和Response 对象



## 概述

Django 使用Request 对象和Response 对象在系统间传递状态。

当请求一个页面时，Django会建立一个包含请求元数据的 [`HttpRequest`](#django.http.HttpRequest "django.http.HttpRequest") 对象。 当Django 加载对应的视图时，[`HttpRequest`](#django.http.HttpRequest "django.http.HttpRequest") 对象将作为视图函数的第一个参数。每个视图会返回一个[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse") 对象。

本文档对[`HttpRequest`](#django.http.HttpRequest "django.http.HttpRequest") 和[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse") 对象的API 进行说明，这些API 定义在[`django.http`](#module-django.http "django.http: Classes dealing with HTTP requests and responses.") 模块中。





## HttpRequest 对象



_class_ `HttpRequest`





### 属性

下面除非特别说明，所有属性都认为是只读的。`会话(session)` 属性是个例外，需要注意。



`HttpRequest.scheme`



New in Django 1.7.

一个字符串，表示请求的方案（通常是`http` 或`https`）。







`HttpRequest.body`



一个字节字符串，表示原始HTTP 请求的正文。它对于处理非HTML 形式的数据非常有用：二进制图像、XML等。 如果要处理常规的表单数据，应该使用`HttpRequest.POST`。

你也可以使用”类文件“形式的接口从HttpRequest 中读取数据。参见[`HttpRequest.read()`](#django.http.HttpRequest.read "django.http.HttpRequest.read")。







`HttpRequest.path`



一个字符串，表示请求的页面的完整路径，不包含域名。

例如：`"/music/bands/the_beatles/"`







`HttpRequest.path_info`



在某些Web 服务器配置下，主机名后的URL 部分被分成脚本前缀部分和路径信息部分。`path_info` 属性将始终包含路径信息部分，不论使用的Web 服务器是什么。使用它代替[`path`](#django.http.HttpRequest.path "django.http.HttpRequest.path") 可以让代码在测试和开发环境中更容易地切换。

例如，如果应用的`WSGIScriptAlias` 设置为`"/minfo"`，那么当`path` 是`"/minfo/music/bands/the_beatles/"` 时`path_info` 将是`"/music/bands/the_beatles/"`。







`HttpRequest.method`



一个字符串，表示请求使用的HTTP 方法。必须使用大写。例如：





```
if request.method == 'GET':
    do_something()
elif request.method == 'POST':
    do_something_else()

```











`HttpRequest.encoding`



一个字符串，表示提交的数据的编码方式（如果为`None` 则表示使用[`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET) 设置）。这个属性是可写的，你可以修改它来修改访问表单数据使用的编码。接下来对属性的任何访问（例如从`GET` 或 `POST` 中读取数据）将使用新的`encoding` 值。如果你知道表单数据的编码不在[`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET) 中，则使用它。







`HttpRequest.GET`



一个类似于字典的对象，包含HTTP GET 的所有参数。详情请参考下面的[`QueryDict`](#django.http.QueryDict "django.http.QueryDict") 文档。







`HttpRequest.POST`



一个包含所有给定的HTTP POST参数的类字典对象，提供了包含表单数据的请求。详情请参考下面的[`QueryDict`](#django.http.QueryDict "django.http.QueryDict") 文档。如果需要访问请求中的原始或非表单数据，可以使用[`HttpRequest.body`](#django.http.HttpRequest.body "django.http.HttpRequest.body") 属性。

POST 请求可以带有空的`POST` 字典 —— 如果通过HTTP POST 方法请求一个表单但是没有包含表单数据的话。因此，不应该使用`if request.POST` 来检查使用的是否是POST 方法；应该使用`if request.method == "POST"`（参见上文）。

注意：`POST` _不_包含上传的文件信息。参见`FILES`。







`HttpRequest.REQUEST`





Deprecated since version 1.7: 使用更显式的`GET` 和`POST` 代替。



一个类似于字典的对象，它首先搜索`POST`，然后搜索`GET`，主要是为了方便。灵感来自于PHP 的`$_REQUEST`。

例如，如果`GET = {"name": "john"}` 而`POST = {"age": '34'}`，`REQUEST["name"]` 将等于`"john"`以及`REQUEST["age"]` 将等于`"34"`。

强烈建议使用`GET` 和`POST` 而不要用`REQUEST`，因为它们更加明确。







`HttpRequest.COOKIES`



一个标准的Python 字典，包含所有的cookie。键和值都为字符串。







`HttpRequest.FILES`



一个类似于字典的对象，包含所有的上传文件。`FILES` 中的每个键为`<input type="file" name="" />` 中的`name`。

更多信息参见[_管理文件_](../topics/files.html)。

注意，`FILES` 只有在请求的方法为POST 且提交的`<form>` 带有`enctype="multipart/form-data"` 的情况下才会包含数据。否则，`FILES` 将为一个空的类似于字典的对象。







`HttpRequest.META`



一个标准的Python 字典，包含所有的HTTP 头部。具体的头部信息取决于客户端和服务器，下面是一些示例：

*   `CONTENT_LENGTH` —— 请求的正文的长度（是一个字符串）。
*   `CONTENT_TYPE` —— 请求的正文的MIME 类型。
*   `HTTP_ACCEPT` —— 响应可接收的Content-Type。
*   `HTTP_ACCEPT_ENCODING` —— 响应可接收的编码。
*   `HTTP_ACCEPT_LANGUAGE`?—— 响应可接收的语言。
*   `HTTP_HOST` —— 客服端发送的HTTP Host 头部。
*   `HTTP_REFERER` —— Referring 页面。
*   `HTTP_USER_AGENT` —— 客户端的user-agent 字符串。
*   `QUERY_STRING` —— 单个字符串形式的查询字符串（未解析过的形式）。
*   `REMOTE_ADDR` —— 客户端的IP 地址。
*   `REMOTE_HOST` —— 客户端的主机名。
*   `REMOTE_USER` —— 服务器认证后的用户。
*   `REQUEST_METHOD` —— 一个字符串，例如`"GET"` 或`"POST"`。
*   `SERVER_NAME` —— 服务器的主机名。
*   `SERVER_PORT` —— 服务器的端口（是一个字符串）。

从上面可以看到，除`CONTENT_LENGTH` 和`CONTENT_TYPE` 之外，请求中的任何HTTP 头部转换为`META` 的键时，都会将所有字母大写并将连接符替换为下划线最后加上`HTTP_` 前缀。所以，一个叫做`X-Bender` 的头部将转换成`META` 中的`HTTP_X_BENDER` 键。







`HttpRequest.user`



一个[`AUTH_USER_MODEL`](settings.html#std:setting-AUTH_USER_MODEL) 类型的对象，表示当前登录的用户。如果用户当前没有登录，`user` 将设置为[`django.contrib.auth.models.AnonymousUser`](contrib/auth.html#django.contrib.auth.models.AnonymousUser "django.contrib.auth.models.AnonymousUser") 的一个实例。你可以通过[`is_authenticated()`](contrib/auth.html#django.contrib.auth.models.User.is_authenticated "django.contrib.auth.models.User.is_authenticated") 区分它们，像这样：





```
if request.user.is_authenticated():
    # Do something for logged-in users.
else:
    # Do something for anonymous users.

```





`user` 只有当Django 启用[`AuthenticationMiddleware`](middleware.html#django.contrib.auth.middleware.AuthenticationMiddleware "django.contrib.auth.middleware.AuthenticationMiddleware") 中间件时才可用。更多信息，参见[_Django 中的用户认证_](../topics/auth/index.html)。







`HttpRequest.session`



一个既可读又可写的类似于字典的对象，表示当前的会话。只有当Django 启用会话的支持时才可用。完整的细节参见[_会话的文档_](../topics/http/sessions.html)。







`HttpRequest.urlconf`



不是由Django 自身定义的，但是如果其它代码（例如，自定义的中间件类）设置了它，Django 就会读取它。如果存在，它将用来作为当前的请求的Root URLconf，并覆盖[`ROOT_URLCONF`](settings.html#std:setting-ROOT_URLCONF) 设置。细节参见[_Django 如何处理请求_](../topics/http/urls.html#how-django-processes-a-request)。







`HttpRequest.resolver_match`



一个[`ResolverMatch`](urlresolvers.html#django.core.urlresolvers.ResolverMatch "django.core.urlresolvers.ResolverMatch") 的实例，表示解析后的URL。这个属性只有在URL 解析方法之后才设置，这意味着它在所有的视图中可以访问，但是在在URL 解析发生之前执行的中间件方法中不可以访问（比如`process_request`，但你可以使用`process_view` 代替）。









### 方法



`HttpRequest.get_host`()



根据从`HTTP_X_FORWARDED_HOST`（如果打开[`USE_X_FORWARDED_HOST`](settings.html#std:setting-USE_X_FORWARDED_HOST)）和`HTTP_HOST` 头部信息返回请求的原始主机。如果这两个头部没有提供相应的值，则使用`SERVER_NAME` 和`SERVER_PORT`，在[**PEP 3333**](http://www.python.org/dev/peps/pep-3333) 中有详细描述。

例如：`"127.0.0.1:8000"`



注

当主机位于多个代理的后面，[`get_host()`](#django.http.HttpRequest.get_host "django.http.HttpRequest.get_host") 方法将会失败。有一个解决办法是使用中间件重写代理的头部，例如下面的例子：





```
class MultipleProxyMiddleware(object):
    FORWARDED_FOR_FIELDS = [
        'HTTP_X_FORWARDED_FOR',
        'HTTP_X_FORWARDED_HOST',
        'HTTP_X_FORWARDED_SERVER',
    ]

    def process_request(self, request):
        """
 Rewrites the proxy headers so that only the most
 recent proxy is used.
 """
        for field in self.FORWARDED_FOR_FIELDS:
            if field in request.META:
                if ',' in request.META[field]:
                    parts = request.META[field].split(',')
                    request.META[field] = parts[-1].strip()

```





这个中间件应该放置在所有依赖于[`get_host()`](#django.http.HttpRequest.get_host "django.http.HttpRequest.get_host") 的中间件之前 —— 例如，[`CommonMiddleware`](middleware.html#django.middleware.common.CommonMiddleware "django.middleware.common.CommonMiddleware") 和[`CsrfViewMiddleware`](middleware.html#django.middleware.csrf.CsrfViewMiddleware "django.middleware.csrf.CsrfViewMiddleware")。









`HttpRequest.get_full_path`()



返回`path`，如果可以将加上查询字符串。

例如：`"/music/bands/the_beatles/?print=true"`







`HttpRequest.build_absolute_uri`(_location_)



返回`location` 的绝对URI。如果location 没有提供，则设置为`request.get_full_path()`。

如果URI 已经是一个绝对的URI，将不会修改。否则，使用请求中的服务器相关的变量构建绝对URI。

例如：`"http://example.com/music/bands/the_beatles/?print=true"`







`HttpRequest.get_signed_cookie`(_key_, _default=RAISE_ERROR_, _salt=''_, _max_age=None_)



返回签名过的Cookie 对应的值，如果签名不再合法则返回`django.core.signing.BadSignature`。如果提供`default` 参数，将不会引发异常并返回default 的值。

可选参数`salt` 可以用来对安全密钥强力攻击提供额外的保护。`max_age` 参数用于检查Cookie 对应的时间戳以确保Cookie 的时间不会超过`max_age` 秒。

示例：





```
>>> request.get_signed_cookie('name')
'Tony'
>>> request.get_signed_cookie('name', salt='name-salt')
'Tony' # assuming cookie was set using the same salt
>>> request.get_signed_cookie('non-existing-cookie')
...
KeyError: 'non-existing-cookie'
>>> request.get_signed_cookie('non-existing-cookie', False)
False
>>> request.get_signed_cookie('cookie-that-was-tampered-with')
...
BadSignature: ...
>>> request.get_signed_cookie('name', max_age=60)
...
SignatureExpired: Signature age 1677.3839159 > 60 seconds
>>> request.get_signed_cookie('name', False, max_age=60)
False

```





更多信息参见[_密钥签名_](../topics/signing.html)。







`HttpRequest.is_secure`()



如果请求时是安全的，则返回`True`；即请求是通过HTTPS 发起的。







`HttpRequest.is_ajax`()



如果请求是通过`XMLHttpRequest` 发起的，则返回`True`，方法是检查`HTTP_X_REQUESTED_WITH` 头部是否是字符串`'XMLHttpRequest'`。大部分现代的JavaScript 库都会发送这个头部。如果你编写自己的XMLHttpRequest 调用（在浏览器端），你必须手工设置这个值来让`is_ajax()` 可以工作。

如果一个响应需要根据请求是否是通过AJAX 发起的，并且你正在使用某种形式的缓存例如Django 的[`cache middleware`](middleware.html#module-django.middleware.cache "django.middleware.cache: Middleware for the site-wide cache.")， 你应该使用[`vary_on_headers('HTTP_X_REQUESTED_WITH')`](../topics/http/decorators.html#django.views.decorators.vary.vary_on_headers "django.views.decorators.vary.vary_on_headers") 装饰你的视图以让响应能够正确地缓存。







`HttpRequest.read`(_size=None_)





`HttpRequest.readline`()





`HttpRequest.readlines`()





`HttpRequest.xreadlines`()





`HttpRequest.__iter__`()



这几个方法实现类文件的接口用于读取HttpRequest· 实例。这使得可以用流的方式读取进来的请求。常见的使用常见是使用迭代的解析器处理一个大型的XML而不用在内存中构建一个完整的XML 树。

根据这个标准的接口，一个HttpRequest 实例可以直接传递给XML 解析器，例如ElementTree：





```
import xml.etree.ElementTree as ET
for element in ET.iterparse(request):
    process(element)

```















## QueryDict 对象



_class_ `QueryDict`



在[`HttpRequest`](#django.http.HttpRequest "django.http.HttpRequest") 对象中，`GET` 和`POST` 属性是`django.http.QueryDict` 的实例，它是一个自定义的类似字典的类，用来处理同一个键带有多个值。这个类的需求来自某些HTML 表单元素传递多个值给同一个键，`<select multiple>` 是一个显著的例子。

`request.POST` 和`request.GET` 的`QueryDict` 在一个正常的请求/响应循环中是不可变的。若要获得可变的版本，需要使用`.copy()`。



### 方法

[`QueryDict`](#django.http.QueryDict "django.http.QueryDict") 实现了字典的所有标准方法，因为它是字典的子类。下面列出了不同点：



`QueryDict.__init__`(_query_string=None_, _mutable=False_, _encoding=None_)



基于`query_string` 实例化`QueryDict` 一个对象。





```
>>> QueryDict('a=1&a=2&c=3')
<QueryDict: {'a': ['1', '2'], 'c': ['3']}>

```





If `query_string` 没被传入, ?`QueryDict` 的结果 是空的 （将没有键和值).

你所遇到的大部分?对象都是不可修改的，例如request.POST和`request.GET。`如果需要实例化你自己的可以修改的对象，通过往它的`__init__()`方法来传递参数 `mutable=True` 可以实现。 .

设置键和值的字符串都将从`encoding` 转换为unicode。如果没有指定编码的话，默认会设置为[`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET).

Changed in Django 1.8:

在以前的版本中，`query_string` 是一个必需的位置参数。









`QueryDict.__getitem__`(_key_)



返回给出的key 的值。如果key 具有多个值，`__getitem__()` 返回最新的值。如果key 不存在，则引发`django.utils.datastructures.MultiValueDictKeyError`。（它是Python 标准`KeyError` 的一个子类，所以你仍然可以坚持捕获`KeyError`。）







`QueryDict.__setitem__`(_key_, _value_)



设置给出的key 的值为`[value]`（一个Python 列表，它具有唯一一个元素`value`）。注意，这和其它具有副作用的字典函数一样，只能在可变的`QueryDict` 上调用（如通过`copy()` 创建的字典）。







`QueryDict.__contains__`(_key_)



如果给出的key 已经设置，则返回`True`。它让你可以做`if "foo" in request.GET` 这样的操作。







`QueryDict.get`(_key_, _default_)



使用与上面`__getitem__()` 相同的逻辑，但是当key 不存在时返回一个默认值。







`QueryDict.setdefault`(_key_, _default_)



类似标准字典的`setdefault()` 方法，只是它在内部使用的是`__setitem__()`。







`QueryDict.update`(_other_dict_)



接收一个`QueryDict` 或标准字典。类似标准字典的`update()` 方法，但是它_附加_到当前字典项的后面，而不是替换掉它们。例如：





```
>>> q = QueryDict('a=1', mutable=True)
>>> q.update({'a': '2'})
>>> q.getlist('a')
['1', '2']
>>> q['a'] # returns the last
['2']

```











`QueryDict.items`()



类似标准字典的`items()` 方法，但是它使用的是和`__getitem__` 一样返回最新的值的逻辑。例如：





```
>>> q = QueryDict('a=1&a=2&a=3')
>>> q.items()
[('a', '3')]

```











`QueryDict.iteritems`()



类似标准字典的`iteritems()` 方法。类似[`QueryDict.items()`](#django.http.QueryDict.items "django.http.QueryDict.items")，它使用的是和[`QueryDict.__getitem__()`](#django.http.QueryDict.__getitem__ "django.http.QueryDict.__getitem__") 一样的返回最新的值的逻辑。







`QueryDict.iterlists`()



类似[`QueryDict.iteritems()`](#django.http.QueryDict.iteritems "django.http.QueryDict.iteritems")，只是它将字典中的每个成员作为列表。







`QueryDict.values`()



类似标准字典的`values()` 方法，但是它使用的是和`__getitem__` 一样返回最新的值的逻辑。例如：





```
>>> q = QueryDict('a=1&a=2&a=3')
>>> q.values()
['3']

```











`QueryDict.itervalues`()



类似[`QueryDict.values()`](#django.http.QueryDict.values "django.http.QueryDict.values")，只是它是一个迭代器。





另外，`QueryDict` 具有以下方法︰



`QueryDict.copy`()



返回对象的副本，使用Python 标准库中的`copy.deepcopy()`。此副本是可变的即使原始对象是不可变的。







`QueryDict.getlist`(_key_, _default_)



以Python 列表形式返回所请求的键的数据。如果键不存在并且没有提供默认值，则返回空列表。它保证返回的是某种类型的列表，除非默认值不是列表。







`QueryDict.setlist`(_key_, _list__)



设置给定的键为`list_`（与`__setitem__()` 不同)。







`QueryDict.appendlist`(_key_, _item_)



将项追加到内部与键相关联的列表中。







`QueryDict.setlistdefault`(_key_, _default_list_)



类似`setdefault`，除了它接受一个列表而不是单个值。







`QueryDict.lists`()



类似[`items`](#django.http.QueryDict.items "django.http.QueryDict.items")，只是它将字典中的每个成员作为列表。例如：





```
>>> q = QueryDict('a=1&a=2&a=3')
>>> q.lists()
[('a', ['1', '2', '3'])]

```











`QueryDict.pop`(_key_)



返回给定键的值的列表，并从字典中移除它们。如果键不存在，将引发`KeyError`。例如 ︰





```
>>> q = QueryDict('a=1&a=2&a=3', mutable=True)
>>> q.pop('a')
['1', '2', '3']

```











`QueryDict.popitem`()



删除字典任意一个成员（因为没有顺序的概念），并返回二值元组，包含键和键的所有值的列表。在一个空的字典上调用时将引发`KeyError`。例如 ︰





```
>>> q = QueryDict('a=1&a=2&a=3', mutable=True)
>>> q.popitem()
('a', ['1', '2', '3'])

```











`QueryDict.dict`()



返回`QueryDict` 的`dict` 表示形式。对于`QueryDict` 中的每个(key, list)对 ，`dict` 将有(key, item) 对，其中item 是列表中的一个元素，使用与[`QueryDict.__getitem__()`](#django.http.QueryDict.__getitem__ "django.http.QueryDict.__getitem__")相同的逻辑：





```
>>> q = QueryDict('a=1&a=3&a=5')
>>> q.dict()
{'a': '5'}

```











`QueryDict.urlencode`([_safe_])



在查询字符串格式返回数据的字符串形式。示例︰





```
>>> q = QueryDict('a=2&b=3&b=5')
>>> q.urlencode()
'a=2&b=3&b=5'

```





可选地，urlencode 可以传递不需要编码的字符。例如︰





```
>>> q = QueryDict(mutable=True)
>>> q['next'] = '/a&b/'
>>> q.urlencode(safe='/')
'next=/a%26b/'

```















## HttpResponse 对象



_class_ `HttpResponse`



与由Django自动创建的[`HttpRequest`](#django.http.HttpRequest "django.http.HttpRequest") 对象相比，[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse") 对象由程序员创建.你创建的每个视图负责初始化实例,填充并返回一个 [`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse").

?[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse") 类是在[`django.http`](#module-django.http "django.http: Classes dealing with HTTP requests and responses.")模块中定义的。



### 用法



#### 传递字符串

典型的应用是传递一个字符串作为页面的内容到[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse") 构造函数:





```
>>> from django.http import HttpResponse
>>> response = HttpResponse("Here's the text of the Web page.")
>>> response = HttpResponse("Text only, please.", content_type="text/plain")

```





如果你想增量增加内容，你可以将`response` 看做一个类文件对象





```
>>> response = HttpResponse()
>>> response.write("<p>Here's the text of the Web page.</p>")
>>> response.write("<p>Here's another paragraph.</p>")

```









#### 传递迭代器

最后你可以传递给`HttpResponse` 一个迭代器而不是字符串. `HttpResponse` 将立即处理这个迭代器, 把它的内容存成字符串，并丢弃它

如果你需要从迭代器到客户端的数据数据流的形式响应, 你必须用[`StreamingHttpResponse`](#django.http.StreamingHttpResponse "django.http.StreamingHttpResponse") 类代替;.





#### 配置 header fields

把它当作一个类似字典的结构，从你的response中设置和移除一个header field。





```
>>> response = HttpResponse()
>>> response['Age'] = 120
>>> del response['Age']

```





注意！与字典不同的是，如果要删除的header field不存在，`del`不会抛出`KeyError`异常。

For setting the `Cache-Control` and `Vary` header fields, it is recommended to use the [`patch_cache_control()`](utils.html#django.utils.cache.patch_cache_control "django.utils.cache.patch_cache_control") and [`patch_vary_headers()`](utils.html#django.utils.cache.patch_vary_headers "django.utils.cache.patch_vary_headers") methods from [`django.utils.cache`](utils.html#module-django.utils.cache "django.utils.cache: Helper functions for controlling caching."), since these fields can have multiple, comma-separated values. “补丁”方法确保其他值，例如，通过中间件添加的，不会删除。

HTTP header fields 不能包含换行。当我们尝试让header field包含一个换行符（CR 或者 LF），那么将会抛出一个`BadHeaderError`异常。





#### 告诉浏览器以文件附件的形式处理服务器的响应

让浏览器以文件附件的形式处理响应, 需要声明 `content_type` 类型 和设置 `Content-Disposition` 头信息. 例如，下面是 如何给浏览器返回一个微软电子表格：





```
>>> response = HttpResponse(my_data, content_type='application/vnd.ms-excel')
>>> response['Content-Disposition'] = 'attachment; filename="foo.xls"'

```





There’s nothing Django-specific about the `Content-Disposition` header, but it’s easy to forget the syntax, so we’ve included it here.







### 属性



`HttpResponse.content`



一个用来代替content的字节字符串，如果必要，则从一个Unicode对象编码而来。







`HttpResponse.charset`



New in Django 1.8.

一个字符串，用来表示response将会被编码的字符集。如果没有在`HttpResponse`实例化的时候给定这个字符集，那么将会从`content_type` 中解析出来。并且当这种解析成功的时候，[`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET)选项将会被使用。







`HttpResponse.status_code`



响应（response）的[HTTP 响应状态码](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10)







`HttpResponse.reason_phrase`



The HTTP reason phrase for the response.







`HttpResponse.streaming`



这个选项总是`False`。

由于这个属性的存在，使得中间件（middleware）能够却别对待流式response和常规response。







`HttpResponse.closed`



New in Django 1.8.

`True` if the response has been closed.









### 方法



`HttpResponse.__init__`(_content=''_, _content_type=None_, _status=200_, _reason=None_, _charset=None_)



使用页面的内容（content）和content-type来实例化一个`HttpResponse`对象。

`content` 应该是一个迭代器或者字符串。如果它是一个迭代器，那么他应该返回的是一串字符串，并且这些字符串连接起来形成response的内容（content）。如果不是迭代器或者字符串，那么在其被接收的时候将转换成字符串。

`content_type`是可选地通过字符集编码完成的MIME类型，并且用于填充HTTP `Content-Type`头部。如果没有设定, 会从 [`DEFAULT_CONTENT_TYPE`](settings.html#std:setting-DEFAULT_CONTENT_TYPE) 和 [`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET) 设定中提取, 作为默认值: “<cite>text/html;</cite> charset=utf-8”.

`status` 是 [HTTP 响应状态码](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10) 。.

`reason` 是HTTP响应短语 如果没有指定, 则使用默认响应短语.

`charset` 在response中被编码的字符集。如果没有给定，将会从 `content_type`中提取, 如果提取不成功, 那么 [`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET) 的设定将被使用.

New in Django 1.8:

The `charset` parameter was added.









`HttpResponse.__setitem__`(_header_, _value_)



由给定的首部名称和值设定相应的报文首部。 `header` 和 `value` 都应该是字符串类型。







`HttpResponse.__delitem__`(_header_)



根据给定的首部名称来删除报文中的首部。如果对应的首部不存在将沉默地（不引发异常）失败。不区分大小写。







`HttpResponse.__getitem__`(_header_)



根据首部名称返回其值。不区分大小写。







`HttpResponse.has_header`(_header_)



通过检查首部中是否有给定的首部名称（不区分大小写），来返回`True` 或 `False` 。







`HttpResponse.setdefault`(_header_, _value_)



New in Django 1.8.

设置一个首部，除非该首部 header 已经存在了。







`HttpResponse.set_cookie`(_key_, _value=''_, _max_age=None_, _expires=None_, _path='/'_, _domain=None_, _secure=None_, _httponly=False_)



设置一个Cookie。参数与Python 标准库中的[`Morsel`](https://docs.python.org/3/library/http.cookies.html#http.cookies.Morsel "(in Python v3.4)") Cookie 对象相同。

*   `max_age` 以秒为单位，如果Cookie 只应该持续客户端浏览器的会话时长则应该为`None`（默认值）。如果没有指定`expires`，则会通过计算得到。

*   `expires` 应该是一个?UTC `"Wdy, DD-Mon-YY HH:MM:SS GMT"` 格式的字符串，或者一个`datetime.datetime` 对象。如果`expires` 是一个`datetime` 对象，则`max_age` 会通过计算得到。

*   如果你想设置一个跨域的Cookie，请使用`domain` 参数。例如，`domain=".lawrence.com"` 将设置一个www.lawrence.com、blogs.lawrence.com 和calendars.lawrence.com 都可读的Cookie。否则，Cookie 将只能被设置它的域读取。

*   如果你想阻止客服端的JavaScript 访问Cookie，可以设置`httponly=True`。

    [HTTPOnly](https://www.owasp.org/index.php/HTTPOnly) 是包含在HTTP 响应头部Set-Cookie 中的一个标记。它不是[**RFC 2109**](http://tools.ietf.org/html/rfc2109.html) 中Cookie 的标准，也并没有被所有的浏览器支持。但是，如果使用，它是一种降低客户端脚本访问受保护的Cookie 数据风险的有用的方法。



警告

[**RFC 2109**](http://tools.ietf.org/html/rfc2109.html) 和[**RFC 6265**](http://tools.ietf.org/html/rfc6265.html) 都声明客户端至少应该支持4096 个字节的Cookie。对于许多浏览器，这也是最大的大小。如果视图存储大于4096 个字节的Cookie，Django 不会引发异常，但是浏览器将不能正确设置Cookie。









`HttpResponse.set_signed_cookie`(_key_, _value_, _salt=''_, _max_age=None_, _expires=None_, _path='/'_, _domain=None_, _secure=None_, _httponly=True_)



与[`set_cookie()`](#django.http.HttpResponse.set_cookie "django.http.HttpResponse.set_cookie") 类似，但是在设置之前将[_用密钥签名_](../topics/signing.html)。通常与[`HttpRequest.get_signed_cookie()`](#django.http.HttpRequest.get_signed_cookie "django.http.HttpRequest.get_signed_cookie") 一起使用。你可以使用可选的`salt` 参考来增加密钥强度，但需要记住将它传递给对应的[`HttpRequest.get_signed_cookie()`](#django.http.HttpRequest.get_signed_cookie "django.http.HttpRequest.get_signed_cookie") 调用。







`HttpResponse.delete_cookie`(_key_, _path='/'_, _domain=None_)



删除指定的key 的Cookie。如果key 不存在则什么也不发生。

由于Cookie 的工作方式，`path` 和`domain` 应该与`set_cookie()` 中使用的值相同 —— 否则Cookie 不会删掉。







`HttpResponse.write`(_content_)



此方法使[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")实例是一个类似文件的对象。







`HttpResponse.flush`()



此方法使[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")实例是一个类似文件的对象。







`HttpResponse.tell`()



此方法使[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")实例是一个类似文件的对象。







`HttpResponse.getvalue`()



New in Django 1.8.

返回[`HttpResponse.content`](#django.http.HttpResponse.content "django.http.HttpResponse.content")的值。此方法使[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")实例是一个类似流的对象。







`HttpResponse.writable`()



New in Django 1.8.

始终为`True`。此方法使[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")实例是一个类似流的对象。







`HttpResponse.writelines`(_lines_)



New in Django 1.8.

将一个包含行的列表写入响应。不添加行分隔符。此方法使[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")实例是一个类似流的对象。









### HttpResponse的子类

Django包含了一系列的`HttpResponse`衍生类（子类），用来处理不同类型的HTTP 响应（response）。与 `HttpResponse`相同, 这些衍生类（子类）存在于[`django.http`](#module-django.http "django.http: Classes dealing with HTTP requests and responses.")之中。



_class_ `HttpResponseRedirect`



构造函数的第一个参数是必要的 — 用来重定向的地址。这些能够是完全特定的URL地址（比如，`'http://www.yahoo.com/search/'`），或者是一个不包含域名的绝对路径地址（例如，?`'/search/'`）。关于构造函数的其他参数，可以参见?[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")。注意！这个响应会返回一个302的HTTP状态码。



`url`



这个只读属性，代表响应将会重定向的URL地址（相当于`Location` response hader）。











_class_ `HttpResponsePermanentRedirect`



与[`HttpResponseRedirect`](#django.http.HttpResponseRedirect "django.http.HttpResponseRedirect")一样，但是它会返回一个永久的重定向（HTTP状态码301）而不是一个“found”重定向（状态码302）。







_class_ `HttpResponseNotModified`



构造函数不会有任何的参数，并且不应该向这个响应（response）中加入内容（content）。使用此选项可指定自用户上次请求（状态代码304）以来尚未修改页面。







_class_ `HttpResponseBadRequest`



与[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")的行为类似，但是使用了一个400的状态码。







_class_ `HttpResponseNotFound`



与[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")的行为类似，但是使用的404状态码。







_class_ `HttpResponseForbidden`



与[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")类似，但使用403状态代码。







_class_ `HttpResponseNotAllowed`



与[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")类似，但使用405状态码。构造函数的第一个参数是必须的：一个允许使用的方法构成的列表（例如，`['GET', 'POST']`）。







_class_ `HttpResponseGone`



与[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")类似，但使用410状态码。







_class_ `HttpResponseServerError`



与[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")类似，但使用500状态代码。







注意

如果[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")的自定义子类实现了`render`方法，Django会将其视为模拟[`SimpleTemplateResponse`](template-response.html#django.template.response.SimpleTemplateResponse "django.template.response.SimpleTemplateResponse")，且`render`方法必须自己返回一个有效的响应对象。









## JsonResponse 对象

New in Django 1.7.



_class_ `JsonResponse`





`JsonResponse.__init__`(_data_, _encoder=DjangoJSONEncoder_, _safe=True_, _**kwargs_)



[`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse") 的一个子类，用户帮助创建JSON 编码的响应。它从父类继承大部分行为，并具有以下不同点：

它的默认`Content-Type` 头部设置为`application/json`。

它的第一个参数`data`，应该为一个`dict` 实例。如果`safe` 参数设置为`False`，它可以是任何可JSON 序列化的对象。

`encoder`，默认为 `django.core.serializers.json.DjangoJSONEncoder`，用于序列化data。关于这个序列化的更多信息参见[_JSON 序列化_](../topics/serialization.html#serialization-formats-json)。

布尔参数`safe` 默认为`True`。如果设置为`False`，可以传递任何对象进行序列化（否则，只允许`dict` 实例）。如果`safe` 为`True`，而第一个参数传递的不是`dict` 对象，将抛出一个[`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError "(in Python v3.4)")。







### 用法

典型的用法如下：





```
>>> from django.http import JsonResponse
>>> response = JsonResponse({'foo': 'bar'})
>>> response.content
'{"foo": "bar"}'

```







#### 序列化非字典对象

若要序列化非`dict` 对象，你必须设置`safe` 参数为`False`：





```
>>> response = JsonResponse([1, 2, 3], safe=False)

```





如果不传递`safe=False`，将抛出一个[`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError "(in Python v3.4)")。



警告

在[EcmaScript 第5版之前](http://www.ecma-international.org/publications/standards/Ecma-262.htm)，这可能会使JavaScript `Array` 构造函数崩溃。出于这个原因，Django 默认不允许传递非字典对象给[`JsonResponse`](#django.http.JsonResponse "django.http.JsonResponse") 构造函数。然而，现代的大部分浏览器都已经实现EcmaScript 5，它删除了这种攻击性的数组。所以可以不用关注这个安全预防措施。







#### 修改默认的JSON 编码器

如果你需要使用不同的JSON 编码器类，你可以传递`encoder` 参数给构造函数：





```
>>> response = JsonResponse(data, encoder=MyJSONEncoder)

```













## StreamingHttpResponse objects



_class_ `StreamingHttpResponse`



[`StreamingHttpResponse`](#django.http.StreamingHttpResponse "django.http.StreamingHttpResponse")类被用来从Django流式化一个响应（response）到浏览器。如果生成响应太长或者是有使用太多的内存，你可能会想要这样做。例如，它对于[_生成大型CSV文件_](../howto/outputting-csv.html#streaming-csv-files)非常有用。



基于性能的考虑

Django是为了那些短期的请求（request）设计的。流式响应将会为整个响应期协同工作进程。这可能导致性能变差。

总的来说，你需要将代价高的任务移除请求—响应的循环，而不是求助于流式响应。



[`StreamingHttpResponse`](#django.http.StreamingHttpResponse "django.http.StreamingHttpResponse") 不是 [`HttpResponse`](#django.http.HttpResponse "django.http.HttpResponse")的衍生类（子类），因为它实现了完全不同的应用程序接口（API）。尽管如此，除了以下的几个明显不同的地方，其他几乎完全相同：

*   应该提供一个迭代器给它，这个迭代器生成字符串来构成内容（content）
*   你不能直接访问它的内容（content），除非迭代它自己的响应对象。这只在响应被返回到客户端的时候发生。
*   它没有 `content` 属性。取而代之的是，它有一个 [`streaming_content`](#django.http.StreamingHttpResponse.streaming_content "django.http.StreamingHttpResponse.streaming_content") 属性。
*   你不能使用类似文件对象的`tell()`或者 `write()` 方法。那么做会抛出一个异常。

[`StreamingHttpResponse`](#django.http.StreamingHttpResponse "django.http.StreamingHttpResponse") should only be used in situations where it is absolutely required that the whole content isn’t iterated before transferring the data to the client. 由于内容无法访问，许多中间件无法正常工作。例如，无法为流式响应生成`ETag`和`Content-Length`头。



### 属性



`StreamingHttpResponse.streaming_content`



一个表示内容（content）的字符串的迭代器







`StreamingHttpResponse.status_code`



响应的[HTTP状态码](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10)。







`StreamingHttpResponse.reason_phrase`



响应的HTTP原因短语。







`StreamingHttpResponse.streaming`



这个选项总是 `True`.











## FileResponse 对象

New in Django 1.8.



_class_ `FileResponse`



[`FileResponse`](#django.http.FileResponse "django.http.FileResponse")是[`StreamingHttpResponse`](#django.http.StreamingHttpResponse "django.http.StreamingHttpResponse")的衍生类（子类），为二进制文件做了优化。如果?wsgi server 来提供，则使用了 [wsgi.file_wrapper](https://www.python.org/dev/peps/pep-3333/#optional-platform-specific-file-handling) ，否则将会流式化一个文件为一些小块。

`FileResponse` 需要通过二进制模式打开文件，如下:





```
>>> from django.http import FileResponse
>>> response = FileResponse(open('myfile.png', 'rb'))

```







