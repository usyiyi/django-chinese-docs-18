# 如何使用会话 #
Django 提供对匿名会话的完全支持。其会话框架让你根据各个站点的访问者存储和访问任意数据。它在服务器端存储数据并抽象Cookie 的发送和接收。Cookie 包含会话的ID —— 不是数据本身（除非你使用基于Cookie 的后端）。

## 启用会话 ##

会话是通过一个中间件实现的。

为了启用会话功能，需要这样做：

编辑`MIDDLEWARE_CLASSES` 设置并确保它包含'`django.contrib.sessions.middleware.SessionMiddleware`'。`django-admin startproject`创建的默认的`settings.py`已经启用`SessionMiddleware`。
如果你不想使用会话，你也可以从`MIDDLEWARE_CLASSES`中删除`SessionMiddleware`行，并从`INSTALLED_APPS`中删除'`django.contrib.sessions`'。它将节省一些性能消耗。

## 配置会话引擎 ##

默认情况下，Django 存储会话到你的数据库中（使用`django.contrib.sessions.models.Session`模型）。虽然这很方便，但是在某些架构中存储会话在其它地方会更快，所以可以配置Django 来存储会话到你的文件系统上或缓存中。

### 使用数据库支持的会话 ###

如果你想使用数据库支持的会话，你需要添加'`django.contrib.sessions`' 到你的`INSTALLED_APPS`设置中。

在配置完成之后，请运行`manage.py migrate`来安装保存会话数据的一张数据库表。

### 使用基于缓存的会话 ###

为了更好的性能，你可能想使用一个基于缓存的会话后端。

为了使用Django 的缓存系统来存储会话数据，你首先需要确保你已经配置好你的缓存；详细信息参见缓存的文档。

> 警告
>
> 你应该只在使用Memcached 缓存系统时才使用基于缓存的会话。基于本地内存的缓存系统不会长时间保留数据，所以不是一个好的选择，而且直接使用文件或数据库会话比通过文件或数据库缓存系统要快。另外，基于本地内存的缓存系统不是多进程安全的，所以对于生产环境可能不是一个好的选择。

如果你在`CACHES`中定义多个缓存，Django 将使用默认的缓存。若要使用另外一种缓存，请设置`SESSION_CACHE_ALIAS`为该缓存的名字。

配置好缓存之后，对于如何在缓存中存储数据你有两个选择：

+ 对于简单的缓存会话存储，可以设置`SESSION_ENGINE` 为"`django.contrib.sessions.backends.cache`" 。此时会话数据将直接存储在你的缓存中。然而，缓存数据将可能不会持久：如果缓存填满或者缓存服务器重启，缓存数据可能会被清理掉。
+ 若要持久的缓存数据，可以设置`SESSION_ENGINE`为"`django.contrib.sessions.backends.cached_db`"。它的写操作使用缓存 —— 对缓存的每次写入都将再写入到数据库。对于读取的会话，如果数据不在缓存中，则从数据库读取。

两种会话的存储都非常快，但是简单的缓存更快，因为它放弃了持久性。大部分情况下，`cached_db`后端已经足够快，但是如果你需要榨干最后一点的性能，并且接收让会话数据丢失，那么你可使用`cache`后端。

如果你使用`cached_db` 会话后端，你还需要遵循[使用数据库支持的会话](http://python.usyiyi.cn/django/topics/http/sessions.html#using-database-backed-sessions)中的配置说明。

```
Changed in Django 1.7:

在1.7 版之前，`cached_db` 永远使用`default`缓存而不是`SESSION_CACHE_ALIAS`。
```

### 使用基于文件的缓存 ###

要使用基于文件的缓存，请设置`SESSION_ENGINE`为"`django.contrib.sessions.backends.file`"。

你可能还想设置`SESSION_FILE_PATH`（它的默认值来自`tempfile.gettempdir()`的输出，大部分情况是`/tmp`）来控制Django在哪里存储会话文件。请保证你的Web 服务器具有读取和写入这个位置的权限。

### 使用基于Cookie 的会话 ###

要使用基于Cookie 的会话，请设置`SESSION_ENGINE` 为"`django.contrib.sessions.backends.signed_cookies`"。此时，会话数据的存储将使用Django 的加密签名 工具和`SECRET_KEY` 设置。

> 注
>
> 建议保留SESSION_COOKIE_HTTPONLY 设置为True 以防止从JavaScript 中访问存储的数据。


> 警告
>
> 如果`SECRET_KEY` 没有保密并且你正在使用 `PickleSerializer`，这可能导致远端执行任意的代码。
>
> 拥有`SECRET_KEY` 的攻击者不仅可以生成篡改的会话数据而你的站点将会信任这些数据，而且可以远程执行任何代码，就像数据是通过pickle 序列化过的一样。
>
> 如果你使用基于Cookie 的会话，请格外注意你的安全秘钥对于任何可以远程访问的系统都是永远完全保密的。
>
> 会话数据经过签名但没有加密。
>
> 如果使用基于Cookie的会话，则会话数据可以被客户端读取。
>
> MAC（消息认证码）被用来保护数据不被客户端修改，所以被篡改的会话数据将是变成不合法的。如果保存Cookie的客户端（例如你的浏览器）不能保存所有的会话Cookie或丢失数据，会话同样会变得不合法。尽管Django 对数据进行压缩，仍然完全有可能超过每个Cookie [常见的4096 个字节的限制](http://tools.ietf.org/html/rfc2965#section-5.3)。
>
> 没有更新保证
>
> 还要注意，虽然MAC可以保证数据的权威性（由你的站点生成，而不是任何其他人）和完整性（包含全部的数据并且是正确的），它不能保证是最新的，例如返回给你发送给客户端的最新的数据。这意味着对于某些会话数据的使用，基于Cookie 可能让你受到重放攻击。其它方式的会话后端在服务器端保存每个会话并在用户登出时使它无效，基于Cookie 的会话在用户登出时不会失效。因此，如果一个攻击者盗取用户的Cookie，它们可以使用这个Cookie 来以这个用户登录即使用户已登出。Cookies 只能被当做是“过期的”，如果它们比你的SESSION_COOKIE_AGE要旧。
>
> 性能
>
> 最后，Cookie 的大小对[你的网站的速度](http://yuiblog.com/blog/2007/03/01/performance-research-part-3/) 有影响。

## 在视图中使用会话 ##

当`SessionMiddleware` 激活时，每个`HttpRequest` 对象 —— 传递给Django 视图函数的第一个参数 —— 将具有一个`session` 属性，它是一个类字典对象。

你可以在你的视图中任何地方读取并写入 `request.session`。你可以多次编辑它。

`class backends.base.SessionBase`

这是所有会话对象的基类。它具有以下标准的字典方法：

`__getitem__(key)`

例如：`fav_color = request.session['fav_color']`

`__setitem__(key, value)`

例如：`request.session['fav_color'] = 'blue'`

`__delitem__(key)`

例如：`del request.session['fav_color']`。如果给出的key 在会话中不存在，将抛出 `KeyError`。

`__contains__(key)`

例如：`'fav_color' in request.session`

`get(key, default=None)`

例如：`fav_color = request.session.get('fav_color', 'red')`

`pop(key)`

例如：`fav_color = request.session.pop('fav_color')`

`keys()`

`items()`

`setdefault()`

`clear()`

它还具有这些方法：

`flush()`

删除当前的会话数据并删除会话的Cookie。这用于确保前面的会话数据不可以再次被用户的浏览器访问（例如，`django.contrib.auth.logout()` 函数中就会调用它）。

```
Changed in Django 1.8:

删除会话Cookie 是Django 1.8 中的新行为。以前，该行为用于重新生成会话中的值，这个值会在Cookie 中发回给用户。
```

`set_test_cookie()`

设置一个测试的Cookie 来验证用户的浏览器是否支持Cookie。因为Cookie 的工作方式，只有到用户的下一个页面才能验证。更多信息参见下文的设置测试的Cookie。

`test_cookie_worked()`

返回`True` 或`False`，取决于用户的浏览器时候接受测试的Cookie。因为Cookie的工作方式，你必须在前面一个单独的页面请求中调用`set_test_cookie()`。更多信息参见下文的设置测试的Cookie。

`delete_test_cookie()`

删除测试的Cookie。使用这个函数来自己清理。

`set_expiry(value)`

设置会话的超时时间。你可以传递一系列不同的值：

+ 如果`value` 是一个整数，会话将在这么多秒没有活动后过期。例如，调用`request.session.set_expiry(300)` 将使得会话在5分钟后过期。
+ 若果value 是一个 `datetime` 或`timedelta` 对象，会话将在这个指定的日期/时间过期。注意`datetime` 和`timedelta` 值只有在你使用P`ickleSerializer` 时才可序列化。
+ 如果`value` 为0，那么用户会话的Cookie将在用户的浏览器关闭时过期。
+ 如果`value` 为`None`，那么会话转向使用全局的会话过期策略。

过期的计算不考虑读取会话的操作。会话的过期从会话上次修改的时间开始计算。

`get_expiry_age()`

返回会话离过期的秒数。对于没有自定义过期的会话（或者设置为浏览器关闭时过期的会话），它将等于`SESSION_COOKIE_AGE`。

该函数接收两个可选的关键字参数：

+ `modification`：会话的最后一次修改时间，类型为一个`datetime` 对象。默认为当前的时间。
+ `expiry`：会话的过期信息，类型为一个`datetime` 对象、一个整数（以秒为单位）或`None`。默认为通过`set_expiry()`保存在会话中的值，如果没有则为`None`。

`get_expiry_date()`

返回过期的日期。对于没有自定义过期的会话（或者设置为浏览器关闭时过期的会话），它将等于从现在开始`SESSION_COOKIE_AGE`秒后的日期。

这个函数接受与`get_expiry_age()`一样的关键字参数。

`get_expire_at_browser_close()`

返回`True` 或`False`，取决于用户的会话Cookie在用户浏览器关闭时会不会过期。

`clear_expired()`

从会话的存储中清除过期的会话。这个类方法被`clearsessions`调用。

`cycle_key()`

创建一个新的会话，同时保留当前的会话数据。`django.contrib.auth.login()` 调用这个方法来减缓会话的固定。

### 会话的序列化 ###

在1.6 版以前，在保存会话数据到后端之前Django 默认使用pickle 来序列化它们。如果你使用的是签名的Cookie 会话后端 并且`SECRET_KEY` 被攻击者知道（Django 本身没有漏洞会导致它被泄漏），攻击者就可以在会话中插入一个字符串，在unpickle 之后可以在服务器上执行任何代码。在因特网上这个攻击技术很简单并很容易查到。尽管Cookie 会话的存储对Cookie 保存的数据进行了签名以防止篡改，`SECRET_KEY` 的泄漏会立即使得可以执行远端的代码。

这种攻击可以通过JSON而不是pickle序列化会话数据来减缓。为了帮助这个功能，Django 1.5.3 引入一个新的设置，`SESSION_SERIALIZER`，来自定义会话序列化的格式。为了向后兼容，这个设置在Django 1.5.x 中默认为`django.contrib.sessions.serializers.PickleSerializer`，但是为了增强安全性，在Django 1.6 中默认为`django.contrib.sessions.serializers.JSONSerializer`。即使在编写你自己的序列化方法讲述的说明中，我们也强烈建议依然使用JSON 序列化，特别是在你使用的是Cookie 后端时。

#### 绑定的序列化方法 ####

`class serializers.JSONSerializer`

对 `django.core.signing`中的JSON 序列化方法的一个包装。只可以序列基本的数据类型。

另外，因为JSON 只支持字符串作为键，注意使用非字符串作为`request.session` 的键将不工作：

```
>>> # initial assignment
>>> request.session[0] = 'bar'
>>> # subsequent requests following serialization & deserialization
>>> # of session data
>>> request.session[0]  # KeyError
>>> request.session['0']
'bar'
```

参见[编写你自己的序列化器](http://python.usyiyi.cn/django/topics/http/sessions.html#custom-serializers) 一节以获得更多关于JSON 序列化的限制。

`class serializers.PickleSerializer`

支持任意Python 对象，但是正如上面描述的，可能导致远端执行代码的漏洞，如果攻击者知道了`SECRET_KEY`。

#### 编写你自己的序列化器 ####

注意，与`PickleSerializer`不同，`JSONSerializer` 不可以处理任意的Python 数据类型。这是常见的情况，需要在便利性和安全性之间权衡。如果你希望在JSON 格式的会话中存储更高级的数据类型比如`datetime` 和 `Decimal`，你需要编写一个自定义的序列化器（或者在保存它们到`request.session`中之前转换这些值到一个可JSON 序列化的对象）。虽然序列化这些值相当简单直接 （`django.core.serializers.json.DateTimeAwareJSONEncoder` 可能帮得上忙），编写一个解码器来可靠地取出相同的内容却能困难。例如，返回一个`datetime` 时，它可能实际上是与`datetime` 格式碰巧相同的一个字符串）。

你的序列化类必须实现两个方法，`dumps(self, obj)` 和`loads(self, data)` 来分别序列化和去序列化会话数据的字典。

### 会话对象指南 ###

`在request.session` 上使用普通的Python 字符串作为字典的键。这主要是为了方便而不是一条必须遵守的规则。
以一个下划线开始的会话字典的键被Django保留作为内部使用。
不要新的对象覆盖`request.session`，且不要访问或设置它的属性。要像Python 字典一样使用它。

### 例子 ###

下面这个简单的视图在一个用户提交一个评论后设置`has_commented` 变量为`True`。它不允许一个用户多次提交评论：

```
def post_comment(request, new_comment):
    if request.session.get('has_commented', False):
        return HttpResponse("You've already commented.")
    c = comments.Comment(comment=new_comment)
    c.save()
    request.session['has_commented'] = True
    return HttpResponse('Thanks for your comment!')
```

登录站点一个“成员”的最简单的视图：

```
def login(request):
    m = Member.objects.get(username=request.POST['username'])
    if m.password == request.POST['password']:
        request.session['member_id'] = m.id
        return HttpResponse("You're logged in.")
    else:
        return HttpResponse("Your username and password didn't match.")
```

...下面是登出一个成员的视图，已经上面的login()：

```
def logout(request):
    try:
        del request.session['member_id']
    except KeyError:
        pass
    return HttpResponse("You're logged out.")
```

标准的`django.contrib.auth.logout()` 函数实际上所做的内容比这个要多一点以防止意外的数据泄露。它调用的`request.session`的`flush()`方法。我们使用这个例子来演示如何利用会话对象来工作，而不是一个完整的`logout()`实现。

## 设置测试的Cookie ##

为了方便，Django 提供一个简单的方法来测试用户的浏览器时候接受Cookie。只需在一个视图中调用`request.session`的`set_test_cookie()`方法，并在接下来的视图中调用`test_cookie_worked()` —— 不是在同一个视图中调用。

由于Cookie的工作方式，在`set_test_cookie()` 和`test_cookie_worked()` 之间这种笨拙的分离是必要的。当你设置一个Cookie，直到浏览器的下一个请求你不可能真实知道一个浏览器是否接受了它。

使用`delete_test_cookie()` 来自己清除测试的Cookie是一个很好的实践。请在你已经验证测试的Cookie 已经工作后做这件事。

下面是一个典型的使用示例：

```
def login(request):
    if request.method == 'POST':
        if request.session.test_cookie_worked():
            request.session.delete_test_cookie()
            return HttpResponse("You're logged in.")
        else:
            return HttpResponse("Please enable cookies and try again.")
    request.session.set_test_cookie()
    return render_to_response('foo/login_form.html')
```

## 在视图外使用会话 ##

> 注
>
> 这一节中的示例直接从`django.contrib.sessions.backends.db`中导`入SessionStore` 对象。在你的代码中，你应该从`SESSION_ENGINE` 指定的会话引擎中导入`SessionStore`，如下所示：

```
>>> from importlib import import_module
>>> from django.conf import settings
>>> SessionStore = import_module(settings.SESSION_ENGINE).SessionStore
```

在视图的外面有一个API 可以使用来操作会话的数据：

```
>>> from django.contrib.sessions.backends.db import SessionStore
>>> s = SessionStore()
>>> # stored as seconds since epoch since datetimes are not serializable in JSON.
>>> s['last_login'] = 1376587691
>>> s.save()
>>> s.session_key
'2b1189a188b44ad18c35e113ac6ceead'

>>> s = SessionStore(session_key='2b1189a188b44ad18c35e113ac6ceead')
>>> s['last_login']
1376587691
```

为了减缓会话固话攻击，不存在的会话的键将重新生成：

```
>>> from django.contrib.sessions.backends.db import SessionStore
>>> s = SessionStore(session_key='no-such-session-here')
>>> s.save()
>>> s.session_key
'ff882814010ccbc3c870523934fee5a2'
```

如果你使用的是`django.contrib.sessions.backends.db` 后端，每个会话只是一个普通的Django 模型。`Session` 模型定义在 `django/contrib/sessions/models.py`中。因为它是一个普通的模型，你可以使用普通的Django 数据库API 来访问会话：

```
>>> from django.contrib.sessions.models import Session
>>> s = Session.objects.get(pk='2b1189a188b44ad18c35e113ac6ceead')
>>> s.expire_date
datetime.datetime(2005, 8, 20, 13, 35, 12)
```

注意，你需要调用`get_decoded()` 以获得会话的字典。这是必需的，因为字典是以编码后的格式保存的：

```
>>> s.session_data
'KGRwMQpTJ19hdXRoX3VzZXJfaWQnCnAyCkkxCnMuMTExY2ZjODI2Yj...'
>>> s.get_decoded()
{'user_id': 42}
```

## 会话何时保存 ##

默认情况下，Django 只有在会话被修改时才会保存会话到数据库中 —— 即它的字典中的任何值被赋值或删除时：

```
# Session is modified.
request.session['foo'] = 'bar'

# Session is modified.
del request.session['foo']

# Session is modified.
request.session['foo'] = {}

# Gotcha: Session is NOT modified, because this alters
# request.session['foo'] instead of request.session.
request.session['foo']['bar'] = 'baz'
```

上面例子的最后一种情况，我们可以通过设置会话对象的`modified`属性显式地告诉会话对象它已经被修改过：

```
request.session.modified = True
```

若要修改这个默认的行为，可以设置 `SESSION_SAVE_EVERY_REQUEST` 为`True`。当设置为`True`时，Django 将对每个请求保存会话到数据库中。

注意会话的Cookie 只有在一个会话被创建或修改后才会发送。如果`SESSION_SAVE_EVERY_REQUEST` 为`True`，会话的Cookie 将在每个请求中发送。

类似地，会话Cookie 的`expires` 部分在每次发送会话Cookie 时更新。

如果响应的状态码时500，则会话不会被保存。

## 浏览器时长的会话 VS. 持久的会话 ##

你可以通过`SESSION_EXPIRE_AT_BROWSER_CLOSE`设置来控制会话框架使用浏览器时长的会话，还是持久的会话。

默认情况下，`SESSION_EXPIRE_AT_BROWSER_CLOSE`设置为`False`，表示会话的Cookie 保存在用户的浏览器中的时间为`SESSION_COOKIE_AGE`。如果你不想让大家每次打开浏览器时都需要登录时可以这样使用。

如果`SESSION_EXPIRE_AT_BROWSER_CLOSE` 设置为`True`，Django 将使用浏览器时长的Cookie —— 用户关闭他们的浏览器时立即过期。如果你想让大家在每次打开浏览器时都需要登录时可以这样使用。

这个设置是一个全局的默认值，可以通过显式地调`request.session` 的`set_expiry()` 方法来覆盖，在上面的在视图中使用会话中有描述。

> 注
>
> 某些浏览器（例如Chrome）提供一种设置，允许用户在关闭并重新打开浏览器后继续使用会话。在某些情况下，这可能干扰`SESSION_EXPIRE_AT_BROWSER_CLOSE` 设置并导致会话在浏览器关闭后不会过期。在测试启用`SESSION_EXPIRE_AT_BROWSER_CLOSE`设置的Django 应用时请注意这点。

## 清除存储的会话 ##

随着用户在你的网站上创建新的会话，会话数据可能会在你的会话存储仓库中积累。如果你正在使用数据库作为后端，`django_session` 数据库表将持续增长。如果你正在使用文件作为后端，你的临时目录包含的文件数量将持续增长。

要理解这个问题，考虑一下数据库后端发生的情况。当一个用户登入时，Django 添加一行到`django_session` 数据库表中。每次会话数据更新时，Django 将更新这行。如果用户手工登出，Django 将删除这行。但是如果该用户不登出，该行将永远不会删除。以文件为后端的过程类似。

Django 不提供自动清除过期会话的功能。因此，定期地清除会话是你的任务。Django 提供一个清除用的管理命令来满足这个目的：`clearsessions`。建议定义调用这个命令，例如作为一个每天运行的Cron 任务。

注意，以缓存为后端不存在这个问题，因为缓存会自动删除过期的数据。以cookie 为后端也不存在这个问题，因为会话数据通过用户的浏览器保存。

## 设置 ##

一些[Django 设置](http://python.usyiyi.cn/django/ref/settings.html#settings-sessions) 让你可以控制会话的行为：

+ [SESSION_CACHE_ALIAS](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_CACHE_ALIAS)
+ [SESSION_COOKIE_AGE](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_COOKIE_AGE)
+ [SESSION_COOKIE_DOMAIN](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_COOKIE_DOMAIN)
+ [SESSION_COOKIE_HTTPONLY](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_COOKIE_HTTPONLY)
+ [SESSION_COOKIE_NAME](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_COOKIE_NAME)
+ [SESSION_COOKIE_PATH](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_COOKIE_PATH)
+ [SESSION_COOKIE_SECURE](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_COOKIE_SECURE)
+ [SESSION_ENGINE](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_ENGINE)
+ [SESSION_EXPIRE_AT_BROWSER_CLOSE](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_EXPIRE_AT_BROWSER_CLOSE)
+ [SESSION_FILE_PATH](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_FILE_PATH)
+ [SESSION_SAVE_EVERY_REQUEST](http://python.usyiyi.cn/django/ref/settings.html#std:setting-SESSION_SAVE_EVERY_REQUEST)

## 会话的安全 ##

一个站点下的子域名能够在客户端为整个域名设置Cookie。如果子域名不收信任的用户控制且允许来自子域名的Cookie，那么可能发生会话固定。

例如，一个攻击者可以登录`good.example.com`并为他的账号获取一个合法的会话。如果该攻击者具有`bad.example.com`的控制权，那么他可以使用这个域名来发送他的会话ID给你，因为子域名允许在`*.example.com`上设置Cookie。当你访问`good.example.com`时，你将被登录成攻击者而没有注意到并输入你的敏感的个人信息（例如，信用卡信息）到攻击者的账号中。

另外一个可能的攻击是，如果`good.example.com`设置它的 `SESSION_COOKIE_DOMAIN` 为"`.example.com`" ，这将导致来自该站点的会话Cookie 被发送到`bad.example.com`。

## 技术细节 ##

+ 当使用`JSONSerializer`时，会话字典接收任何可json 序列化的值，当使用`PickleSerializer`时接收任何pickleable 的Python对象。更多信息参见`pickle` 模块。
+ 会话数据存储在数据中名为`django_session` 的表中。
+ Django 只发送它需要的Cookie。如果你没有设置任何会话数据，它将不会发送会话Cookie。

## URL 中的会话ID ##

Django 会话框架完全地、唯一地基于Cookie。它不像PHP一样，实在没办法就把会话的ID放在URL 中。这是一个故意的设计。这个行为不仅使得URL变得丑陋，还使得你的网站易于受到通过"Referer" 头部窃取会话ID的攻击。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Sessions](https://docs.djangoproject.com/en/1.8/topics/http/sessions/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
