# 错误报告

当你运行一个公开站点时，你应该始终关闭[`DEBUG`](../ref/settings.html#std:setting-DEBUG) 设置。这会使你的服务器运行得更快，也会防止恶意用户看到由错误页面展示的一些应用细节。

但是，运行在&nbsp;[`DEBUG`](../ref/settings.html#std:setting-DEBUG)为`False`的情况下，你不会看到你的站点所生成的错误 -- 每个人都只能看到公开的错误页面。你需要跟踪部署的站点上的错误，所以可以配置Django来生成带有错误细节的报告。

## 报告邮件

### 服务器错误

[`DEBUG`](../ref/settings.html#std:setting-DEBUG) 为 `False`的时候，无论什么时候代码产生了未处理的异常，并且出现了服务器内部错误（HTTP状态码 500），Django 都会给[`ADMINS`](../ref/settings.html#std:setting-ADMINS)设置中的用户发送邮件。 这会向管理员提供任何错误的及时通知。&nbsp;[`ADMINS`](../ref/settings.html#std:setting-ADMINS)会得到一份错误的描述，完整的Python&nbsp;traceback，以及HTTP请求和导致错误的详细信息。

注意

为了发送邮件，DJango需要一些设置来告诉它如何连接到邮件服务器。最起码，你需要指定 [`EMAIL_HOST`](../ref/settings.html#std:setting-EMAIL_HOST) ，可能需要 [`EMAIL_HOST_USER`](../ref/settings.html#std:setting-EMAIL_HOST_USER) 和[`EMAIL_HOST_PASSWORD`](../ref/settings.html#std:setting-EMAIL_HOST_PASSWORD)，尽管所需的其他设置可能也依赖于你的邮件服务器的配置。邮件相关设置的完整列表请见 [_Django设置文档_](../ref/settings.html)。

Django通常从[root@localhost](mailto:root%40localhost)发送邮件。但是一些邮件提供商会拒收所有来自这个地址的邮件。修改[`SERVER_EMAIL`](../ref/settings.html#std:setting-SERVER_EMAIL)设置可以使用不同的发信人地址。

将收信人的邮箱地址放入[`ADMINS`](../ref/settings.html#std:setting-ADMINS)设置中来激活这一行为。

另见

服务器错误邮件使用日志框架来发送，所以你可以通过&nbsp;[_自定义你的日志配置_](../topics/logging.html)自定义这一行为。

### 404错误

也可以配置Django来发送关于死链的邮件（404"找不到页面"错误）。Django在以下情况发送404错误的邮件：

*   [`DEBUG`](../ref/settings.html#std:setting-DEBUG)为 `False`；
*   你的[`MIDDLEWARE_CLASSES`](../ref/settings.html#std:setting-MIDDLEWARE_CLASSES) 设置含有 [`django.middleware.common.BrokenLinkEmailsMiddleware`](../ref/middleware.html#django.middleware.common.BrokenLinkEmailsMiddleware "django.middleware.common.BrokenLinkEmailsMiddleware")。

如果符合这些条件，无论什么时候你的代码产生404错误，并且请求带有referer， Django 都会给[`MANAGERS`](../ref/settings.html#std:setting-MANAGERS)中的用户发送邮件。 (It doesn’t bother to email for 404s that don’t have a referer – those are usually just people typing in broken URLs or broken Web ‘bots).

注意

[`BrokenLinkEmailsMiddleware`](../ref/middleware.html#django.middleware.common.BrokenLinkEmailsMiddleware "django.middleware.common.BrokenLinkEmailsMiddleware") 必须出现在其它拦截404错误的中间件之前，比如 [`LocaleMiddleware`](../ref/middleware.html#django.middleware.locale.LocaleMiddleware "django.middleware.locale.LocaleMiddleware") 或者 [`FlatpageFallbackMiddleware`](../ref/contrib/flatpages.html#django.contrib.flatpages.middleware.FlatpageFallbackMiddleware "django.contrib.flatpages.middleware.FlatpageFallbackMiddleware")。把它放在你的[`MIDDLEWARE_CLASSES`](../ref/settings.html#std:setting-MIDDLEWARE_CLASSES)设置的最上面。

你可以通过调整[`IGNORABLE_404_URLS`](../ref/settings.html#std:setting-IGNORABLE_404_URLS)设置，告诉Django停止报告特定的404错误。它应该为一个元组，含有编译后的正则表达式对象。例如：

```
import re
IGNORABLE_404_URLS = (
    re.compile(r'\.(php|cgi)$'),
    re.compile(r'^/phpmyadmin/'),
)

```

在这个例子中，任何以`.php` 或者`.cgi`结尾URL的404错误都_不会_报告。任何以`/phpmyadmin/`开头的URL也不会。

下面的例子展示了如何排除一些浏览器或爬虫经常请求的常用URL：

```
import re
IGNORABLE_404_URLS = (
    re.compile(r'^/apple-touch-icon.*\.png$'),
    re.compile(r'^/favicon\.ico$'),
    re.compile(r'^/robots\.txt$'),
)

```

（要注意这些是正则表达式，所以需要在句号前面添加反斜线来对它转义。）

如果你打算进一步自定义[`django.middleware.common.BrokenLinkEmailsMiddleware`](../ref/middleware.html#django.middleware.common.BrokenLinkEmailsMiddleware "django.middleware.common.BrokenLinkEmailsMiddleware") 的行为（比如忽略来自web爬虫的请求），你应该继承它并覆写它的方法。

另见

404错误使用日志框架来记录。通常，日志记录会被忽略，但是你可以通过编写合适的处理器和[_配置日志_](../topics/logging.html)，将它们用于错误报告。

## 过滤错误报告

### 过滤敏感的信息

错误报告对错误的调试及其有用，所以对于这些错误，通常它会尽可能多的记录下相关信息。例如，通常DJango会为产生的异常记录[完整的traceback](http://en.wikipedia.org/wiki/Stack_trace)，[traceback 帧](http://en.wikipedia.org/wiki/Stack_frame)的每个局部变量，以及[`HttpRequest`](../ref/request-response.html#django.http.HttpRequest "django.http.HttpRequest")的[_属性_](../ref/request-response.html#httprequest-attributes)。

然而，有时特定的消息类型十分敏感，并不适合跟踪消息，比如用户的密码或者信用卡卡号。所以Django提供一套函数装饰器，来帮助你控制需要在生产环境（也就是[`DEBUG`](../ref/settings.html#std:setting-DEBUG)为 `False`的情况）中的错误报告中过滤的消息：[`sensitive_variables()`](#django.views.decorators.debug.sensitive_variables "django.views.decorators.debug.sensitive_variables")和[`sensitive_post_parameters()`](#django.views.decorators.debug.sensitive_post_parameters "django.views.decorators.debug.sensitive_post_parameters")。

`sensitive_variables`(_*variables_)[[source]](../_modules/django/views/decorators/debug.html#sensitive_variables)

如果你的代码中一个函数（视图或者常规的回调）使用可能含有敏感信息的局部变量，你可能需要使用`sensitive_variables` 装饰器，来阻止错误报告包含这些变量的值。

```
from django.views.decorators.debug import sensitive_variables

@sensitive_variables('user', 'pw', 'cc')
def process_info(user):
    pw = user.pass_word
    cc = user.credit_card_number
    name = user.name
    ...

```

在上面的例子中，`user`, `pw` 和`cc` 变量的值会在错误报告中隐藏并且使用星号(<cite>**********</cite>) 来代替，虽然`name` 变量的值会公开。

要想有顺序地在错误报告中隐藏一个函数的所有局部变量，不要向`sensitive_variables` 装饰器提供任何参数：

```
@sensitive_variables()
def my_function():
    ...

```

使用多个装饰器的时候

如果你想要隐藏的变量也是一个函数的参数（例如，下面例子中的`user`），并且被装饰的函数有多个装饰器，你需要确保将`@sensitive_variables` 放在装饰器链的顶端。这种方法也会隐藏函数参数，尽管它通过其它装饰器传递：

```
@sensitive_variables('user', 'pw', 'cc')
@some_decorator
@another_decorator
def process_info(user):
    ...

```

`sensitive_post_parameters`(_*parameters_)[[source]](../_modules/django/views/decorators/debug.html#sensitive_post_parameters)

如果你的代码中一个视图接收到了可能带有敏感信息的，带有[`POST 参数`](../ref/request-response.html#django.http.HttpRequest.POST "django.http.HttpRequest.POST")的[`HttpRequest`](../ref/request-response.html#django.http.HttpRequest "django.http.HttpRequest")对象，你可能需要使用`sensitive_post_parameters`&nbsp; 装饰器，来阻止错误报告包含这些参数的值。

```
from django.views.decorators.debug import sensitive_post_parameters

@sensitive_post_parameters('pass_word', 'credit_card_number')
def record_user_profile(request):
    UserProfile.create(user=request.user,
                       password=request.POST['pass_word'],
                       credit_card=request.POST['credit_card_number'],
                       name=request.POST['name'])
    ...

```

在上面的例子中，`pass_word` 和 `credit_card_number` POST参数的值会在错误报告中隐藏并且使用星号(<cite>**********</cite>) 来代替，虽然`name`变量的值会公开。

要想有顺序地在错误报告中隐藏一个请求的所有POST 参数，不要向`sensitive_post_parameters`&nbsp; 装饰器提供任何参数：

```
@sensitive_post_parameters()
def my_view(request):
    ...

```

所有POST参数按顺序被过滤出特定[`django.contrib.auth.views`](../topics/auth/default.html#module-django.contrib.auth.views "django.contrib.auth.views") 视图的错误报告（`login`, `password_reset_confirm`, `password_change`, `add_view` 和`auth`中的`user_change_password`），来防止像是用户密码这样的敏感信息的泄露。

### 自定义错误报告

所有[`sensitive_variables()`](#django.views.decorators.debug.sensitive_variables "django.views.decorators.debug.sensitive_variables")&nbsp; 和 [`sensitive_post_parameters()`](#django.views.decorators.debug.sensitive_post_parameters "django.views.decorators.debug.sensitive_post_parameters")分别用敏感变量的名字向被装饰的函数添加注解，以及用POST敏感参数的名字向`HttpRequest`对象添加注解，以便在错误产生时可以随后过滤掉报告中的敏感信息。Django的默认错误包告过滤器[`django.views.debug.SafeExceptionReporterFilter`](#django.views.debug.SafeExceptionReporterFilter "django.views.debug.SafeExceptionReporterFilter")会完成实际的过滤操作。
产生错误报告的时候，这个过滤器使用装饰器的注解来将相应的值替换为星号&nbsp;(<cite>**********</cite>) 。如果你希望为你的整个站点覆写或自定义这一默认的属性，你需要定义你自己的过滤器类，并且通过[`DEFAULT_EXCEPTION_REPORTER_FILTER`](../ref/settings.html#std:setting-DEFAULT_EXCEPTION_REPORTER_FILTER) 设置来让Django使用它。

```
DEFAULT_EXCEPTION_REPORTER_FILTER = 'path.to.your.CustomExceptionReporterFilter'

```

你也可能会以更精细的方式来控制在提供的视图中使用哪种过滤器，通过设置&nbsp;`HttpRequest`的`exception_reporter_filter`属性。

```
def my_view(request):
    if request.user.is_authenticated():
        request.exception_reporter_filter = CustomExceptionReporterFilter()
    ...

```

你的自定义过滤器类需要继承自&nbsp;[`django.views.debug.SafeExceptionReporterFilter`](#django.views.debug.SafeExceptionReporterFilter "django.views.debug.SafeExceptionReporterFilter")，并且可能需要覆写以下方法：

_class _`SafeExceptionReporterFilter`[[source]](../_modules/django/views/debug.html#SafeExceptionReporterFilter)

`SafeExceptionReporterFilter.``is_active`(_request_)[[source]](../_modules/django/views/debug.html#SafeExceptionReporterFilter.is_active)

如果其它方法中操作的过滤器已激活，返回`True`。如果 [`DEBUG`](../ref/settings.html#std:setting-DEBUG)为`False`，通常过滤器是激活的。

`SafeExceptionReporterFilter.``get_request_repr`(_request_)

Returns the representation string of the request object, that is, the value that would be returned by `repr(request)`, except it uses the filtered dictionary of POST parameters as determined by [`SafeExceptionReporterFilter.get_post_parameters()`](#django.views.debug.SafeExceptionReporterFilter.get_post_parameters "django.views.debug.SafeExceptionReporterFilter.get_post_parameters").

`SafeExceptionReporterFilter.``get_post_parameters`(_request_)[[source]](../_modules/django/views/debug.html#SafeExceptionReporterFilter.get_post_parameters)

返回过滤后的POST参数字典。通常它会把敏感参数的值以星号&nbsp;(<cite>**********</cite>)替换。

`SafeExceptionReporterFilter.``get_traceback_frame_variables`(_request_, _tb_frame_)[[source]](../_modules/django/views/debug.html#SafeExceptionReporterFilter.get_traceback_frame_variables)

返回过滤后的，所提供traceback帧的局部变量的字典。通常它会把敏感变量的值以星号&nbsp;(<cite>**********</cite>)替换。

另见

你也可以通过编写自定义的[_exception middleware_](../topics/http/middleware.html#exception-middleware)来建立自定义的错误报告。如果你编写了自定义的错误处理器，模拟Django内建的错误处理器，只在[`DEBUG`](../ref/settings.html#std:setting-DEBUG) 为 `False`时报告或记录错误是个好主意。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Tracking code errors by email](https://docs.djangoproject.com/en/1.8/howto/error-reporting/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
