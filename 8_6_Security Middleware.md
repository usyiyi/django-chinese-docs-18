

# 中间件

这篇文档介绍了Django自带的所有中间件组件。 要查看关于如何使用它们以及如何编写自己的中间件，请见[_中间件使用指导_](../topics/http/middleware.html)。



## 可用的中间件



### 缓存中间件



_class_ `UpdateCacheMiddleware`[[source]](../_modules/django/middleware/cache.html#UpdateCacheMiddleware)





_class_ `FetchFromCacheMiddleware`[[source]](../_modules/django/middleware/cache.html#FetchFromCacheMiddleware)



开启全站范围的缓存。 如果开启了这些缓存，任何一个由Django提供的页面将会被缓存，缓存时长是由你在[`CACHE_MIDDLEWARE_SECONDS`](settings.html#std:setting-CACHE_MIDDLEWARE_SECONDS)配置中定义的。详见[_缓存文档_](../topics/cache.html)。





### “通用”的中间件



_class_ `CommonMiddleware`[[source]](../_modules/django/middleware/common.html#CommonMiddleware)



给完美主义者增加一些便利条件：

*   禁止访问[`DISALLOWED_USER_AGENTS`](settings.html#std:setting-DISALLOWED_USER_AGENTS)中设置的用户代理，这项配置应该是一个已编译的正则表达式对象的列表。

*   基于[`APPEND_SLASH`](settings.html#std:setting-APPEND_SLASH)和[`PREPEND_WWW`](settings.html#std:setting-PREPEND_WWW)的设置来重写URL。

    如果[`APPEND_SLASH`](settings.html#std:setting-APPEND_SLASH)设为`True`，并且初始URL 没有以斜线结尾以及在URLconf 中没找到对应定义，这时形成一个斜线结尾的新URL。如果这个新的URL存在于URLconf，那么Django 重定向请求到这个新URL上。否则，按正常情况处理初始的URL。

    比如，如果你没有为`foo.com/bar`定义有效的正则，但是_为_`foo.com/bar/`定义了有效的正则，`foo.com/bar`将会被重定向到`foo.com/bar/`。

    如果[`PREPEND_WWW`](settings.html#std:setting-PREPEND_WWW)设为`True`，前面缺少 "www."的url将会被重定向到相同但是以一个"www."开头的url。

    两种选项都是为了规范化url。其中的哲学就是，任何一个url应该在一个地方仅存在一个。技术上来讲，URL `foo.com/bar` 区别于`foo.com/bar/` —— 搜索引擎索引会把这里分开处理 —— 因此最佳实践就是规范化URL。

*   基于[`USE_ETAGS`](settings.html#std:setting-USE_ETAGS) 设置来处理ETag。如果设置[`USE_ETAGS`](settings.html#std:setting-USE_ETAGS)为`True`，Django会通过MD5-hashing处理页面的内容来为每一个页面请求计算Etag，并且如果合适的话，它将会发送携带`Not Modified`的响应。



`CommonMiddleware.response_redirect_class`



New in Django 1.8.

默认为[`HttpResponsePermanentRedirect`](request-response.html#django.http.HttpResponsePermanentRedirect "django.http.HttpResponsePermanentRedirect")。它继承了`CommonMiddleware`，并覆写了属性来自定义中间件发出的重定向。



_class_ `BrokenLinkEmailsMiddleware`[[source]](../_modules/django/middleware/common.html#BrokenLinkEmailsMiddleware)



*   向[`MANAGERS`](settings.html#std:setting-MANAGERS)发送死链提醒邮件（详见[_错误报告_](../howto/error-reporting.html)）。





### GZip中间件



_class_ `GZipMiddleware`[[source]](../_modules/django/middleware/gzip.html#GZipMiddleware)





警告

安全研究员最近发现，当压缩技术（包括`GZipMiddleware`）用于一个网站的时候，网站会受到一些可能的攻击。此外，这些方法可以用于破坏Django的CSRF保护。在你的站点使用`GZipMiddleware`之前，你应该先仔细考虑一下你的站点是否容易受到这些攻击。 如果你不确定是否会受到这些影响，应该避免使用?`GZipMiddleware`。详见[the BREACH paper (PDF)](http://breachattack.com/resources/BREACH%20-%20SSL,%20gone%20in%2030%20seconds.pdf)和[breachattack.com](http://breachattack.com)。



为支持GZip压缩的浏览器（一些现代的浏览器）压缩内容。

建议把这个中间件放到中间件配置列表的第一个，这样压缩响应内容的处理会到最后才 发生。

如果满足下面条件的话，内容不会被压缩：

*   消息体的长度小于200个字节。
*   响应已经设置了`Content-Encoding`协议头。
*   请求（浏览器）没有发送包含`gzip`的`Accept-Encoding`协议头。

你可以通过这个[`gzip_page()`](../topics/http/decorators.html#django.views.decorators.gzip.gzip_page "django.views.decorators.gzip.gzip_page")装饰器使用独立的GZip压缩。





### 带条件判断的GET中间件



_class_ `ConditionalGetMiddleware`[[source]](../_modules/django/middleware/http.html#ConditionalGetMiddleware)



处理带有条件判断状态GET操作。 如果一个请求包含 `ETag`?或者`Last-Modified`协议头，并且请求包含`If-None-Match`或`If-Modified-Since`，这时响应会被 替换为[`HttpResponseNotModified`](request-response.html#django.http.HttpResponseNotModified "django.http.HttpResponseNotModified")。

另外，它会设置`Date`和`Content-Length`响应头。





### 地域性中间件



_class_ `LocaleMiddleware`[[source]](../_modules/django/middleware/locale.html#LocaleMiddleware)



基于请求中的数据开启语言选择。 它可以为每个用户进行定制。 详见[_国际化文档_](../topics/i18n/translation.html)。



`LocaleMiddleware.response_redirect_class`



默认为[`HttpResponseRedirect`](request-response.html#django.http.HttpResponseRedirect "django.http.HttpResponseRedirect")。继承自`LocaleMiddleware`并覆写了属性来自定义中间件发出的重定向。





### 消息中间件



_class_ `MessageMiddleware`[[source]](../_modules/django/contrib/messages/middleware.html#MessageMiddleware)



开启基于Cookie和会话的消息支持。详见[_消息文档_](contrib/messages.html)。





### 安全中间件



警告

如果你的部署环境允许的话，让你的前端web服务器展示`SecurityMiddleware`提供的功能是个好主意。这样一来，如果有任何请求没有被Django处理（比如静态媒体或用户上传的文件），它们会拥有和向Django 应用的请求相同的保护。





_class_ `SecurityMiddleware`[[source]](../_modules/django/middleware/security.html#SecurityMiddleware)



New in Django 1.8.

`django.middleware.security.SecurityMiddleware`为请求/响应循环提供了几种安全改进。每一种可以通过一个选项独立开启或关闭。

*   [SECURE_BROWSER_XSS_FILTER](settings.html#std:setting-SECURE_BROWSER_XSS_FILTER)
*   [SECURE_CONTENT_TYPE_NOSNIFF](settings.html#std:setting-SECURE_CONTENT_TYPE_NOSNIFF)
*   [SECURE_HSTS_INCLUDE_SUBDOMAINS](settings.html#std:setting-SECURE_HSTS_INCLUDE_SUBDOMAINS)
*   [SECURE_HSTS_SECONDS](settings.html#std:setting-SECURE_HSTS_SECONDS)
*   [SECURE_REDIRECT_EXEMPT](settings.html#std:setting-SECURE_REDIRECT_EXEMPT)
*   [SECURE_SSL_HOST](settings.html#std:setting-SECURE_SSL_HOST)
*   [SECURE_SSL_REDIRECT](settings.html#std:setting-SECURE_SSL_REDIRECT)



#### HTTP Strict Transport Security (HSTS)

对于那些应该只能通过HTTPS访问的站点，你可以通过设置[HSTS协议头](http://en.wikipedia.org/wiki/Strict_Transport_Security)，通知现代的浏览器，拒绝用不安全的连接来连接你的域名。这会降低你受到SSL-stripping的中间人（MITM）攻击的风险。

如果你将[`SECURE_HSTS_SECONDS`](settings.html#std:setting-SECURE_HSTS_SECONDS)设置为一个非零值，`SecurityMiddleware`会在所有的HTTPS响应中设置这个协议头。

开启HSTS的时候，首先使用一个小的值来测试它是个好主意，例如，让[`SECURE_HSTS_SECONDS = 3600`](settings.html#std:setting-SECURE_HSTS_SECONDS)为一个小时。每当浏览器在你的站点看到HSTS协议头，都会在提供的时间段内拒绝使用不安全（HTTP）的方式连接到你的域名。一旦你确认你站点上的所有东西都以安全的方式提供（例如，HSTS并不会干扰任何事情），建议你增加这个值，这样不常访问你站点的游客也会被保护（比如，一般设置为31536000秒，一年）。

另外，如果你将?[`SECURE_HSTS_INCLUDE_SUBDOMAINS`](settings.html#std:setting-SECURE_HSTS_INCLUDE_SUBDOMAINS)设置为`True`,，`SecurityMiddleware`会将`includeSubDomains`标签添加到`Strict-Transport-Security`协议头中。强烈推荐这样做（假设所有子域完全使用HTTPS），否则你的站点仍旧有可能由于子域的不安全连接而受到攻击。



警告

HSTS策略在你的整个域中都被应用，不仅仅是你所设置协议头的响应中的url。所以，如果你的整个域都设置为HTTPS only，你应该只使用HSTS策略。

适当遵循HSTS协议头的浏览器，会通过显示警告的方式，拒绝让用户连接到证书过期的、自行签署的、或者其他SSL证书无效的站点。如果你使用了HSTS，确保你的证书处于一直有效的状态！





注意

如果你的站点部署在负载均衡器或者反向代理之后，并且`Strict-Transport-Security`协议头没有添加到你的响应中，原因是Django有可能意识不到这是一个安全连接。你可能需要设置[`SECURE_PROXY_SSL_HEADER`](settings.html#std:setting-SECURE_PROXY_SSL_HEADER)。







#### X-Content-Type-Options: nosniff

一些浏览器会尝试猜测他们所得内容的类型，而不是读取`Content-Type`协议头。虽然这样有助于配置不当的服务器正常显示内容，但也会导致安全问题。

如果你的站点允许用户上传文件，一些恶意的用户可能会上传一个精心构造的文件，当你觉得它无害的时候，文件会被浏览器解释成HTML或者Javascript。

欲知更多有关这个协议头和浏览器如何处理它的内容，你可以在[IE安全博客](http://blogs.msdn.com/b/ie/archive/2008/09/02/ie8-security-part-vi-beta-2-update.aspx)中读到它。

要防止浏览器猜测内容类型，并且强制它一直使用?`Content-Type`协议头中提供的类型，你可以传递`X-Content-Type-Options: nosniff`协议头。`SecurityMiddleware`将会对所有响应这样做，如果[`SECURE_CONTENT_TYPE_NOSNIFF`](settings.html#std:setting-SECURE_CONTENT_TYPE_NOSNIFF) 设置为`True`。

注意在大多数Django不涉及处理上传文件的部署环境中，这个设置不会有任何帮助。例如，如果你的[`MEDIA_URL`](settings.html#std:setting-MEDIA_URL)被前端web服务器直接处理（例如nginx和Apache），你可能想要在那里设置这个协议头。而在另一方面，如果你使用Django执行为了下载文件而请求授权之类的事情，并且你不能使用你的web服务器设置协议头，这个设置会很有用。





#### X-XSS-Protection: 1; mode=block

一些浏览器能够屏蔽掉出现[XSS攻击](http://en.wikipedia.org/wiki/Cross-site_scripting)的内容。通过寻找页面中GET或者POST参数中的JavaScript内容来实现。如果JavaScript在服务器的响应中被重放，页面就会停止渲染，并展示一个错误页来取代。

[X-XSS-Protection协议头](http://blogs.msdn.com/b/ie/archive/2008/07/02/ie8-security-part-iv-the-xss-filter.aspx)用来控制XSS过滤器的操作。

要在浏览器中启用XSS过滤器，并且强制它一直屏蔽可疑的XSS攻击，你可以在协议头中传递`X-XSS-Protection: 1; mode=block`。 如果[`SECURE_BROWSER_XSS_FILTER`](settings.html#std:setting-SECURE_BROWSER_XSS_FILTER)设置为`True`，`SecurityMiddleware`会在所有响应中这样做。



警告

浏览器的XSS过滤器是一个十分有效的手段，但是不要过度依赖它。它并不能检测到所有的XSS攻击，也不是所有浏览器都支持这一协议头。确保你[_校验和过滤_](../topics/security.html#cross-site-scripting)了所有的输入来防止XSS攻击。







#### SSL重定向

如果你同时提供HTTP和HTTPS连接，大多数用户会默认使用不安全的（HTTP）链接。为了更高的安全性，你应该重定向所有的HTTP 连接到HTTPS。

如果你将[`SECURE_SSL_REDIRECT`](settings.html#std:setting-SECURE_SSL_REDIRECT)设置为True，`SecurityMiddleware`会将HTTP链接永久地（HTTP 301，permanently）重定向到HTTPS连接。



注意

由于性能因素，最好在Django外面执行这些重定向，在[nginx](http://nginx.org)这种前端负载均衡器或者反向代理服务器中执行。[`SECURE_SSL_REDIRECT`](settings.html#std:setting-SECURE_SSL_REDIRECT)专门为这种部署情况而设计，当这不可选择的时候。



如果[`SECURE_SSL_HOST`](settings.html#std:setting-SECURE_SSL_HOST)设置有一个值，所有重定向都会发到值中的主机，而不是原始的请求主机。

如果你站点上的一些页面应该以HTTP方式提供，并且不需要重定向到HTTPS，你可以[`SECURE_REDIRECT_EXEMPT`](settings.html#std:setting-SECURE_REDIRECT_EXEMPT)设置中列出匹配那些url的正则表达式。



注意

如果你在负载均衡器或者反向代理服务器后面部署应用，而且Django不能辨别出什么时候一个请求是安全的，你可能需要设置[`SECURE_PROXY_SSL_HEADER`](settings.html#std:setting-SECURE_PROXY_SSL_HEADER)。









### 会话中间件



_class_ `SessionMiddleware`[[source]](../_modules/django/contrib/sessions/middleware.html#SessionMiddleware)



开启会话支持。详见[_会话文档_](../topics/http/sessions.html)。





### 站点中间件



_class_ `CurrentSiteMiddleware`[[source]](../_modules/django/contrib/sites/middleware.html#CurrentSiteMiddleware)



New in Django 1.7.

向每个接收到的`HttpRequest`对象添加一个`site`属性，表示当前的站点。详见[_站点文档_](contrib/sites.html#site-middleware)。





### 认证中间件



_class_ `AuthenticationMiddleware`[[source]](../_modules/django/contrib/auth/middleware.html#AuthenticationMiddleware)



向每个接收到的`HttpRequest`对象添加`user`属性，表示当前登录的用户。详见[_web请求中的认证_](../topics/auth/default.html#auth-web-requests)。



_class_ `RemoteUserMiddleware`[[source]](../_modules/django/contrib/auth/middleware.html#RemoteUserMiddleware)



使用web服务器提供认证的中间件。详见[_使用REMOTE_USER进行认证_](../howto/auth-remote-user.html)。



_class_ `SessionAuthenticationMiddleware`[[source]](../_modules/django/contrib/auth/middleware.html#SessionAuthenticationMiddleware)



New in Django 1.7.

当用户修改密码的时候使用户的会话失效。详见[_密码更改时的会话失效_](../topics/auth/default.html#session-invalidation-on-password-change)。在[`MIDDLEWARE_CLASSES`](settings.html#std:setting-MIDDLEWARE_CLASSES)中，这个中间件必须出现在[`django.contrib.auth.middleware.AuthenticationMiddleware`](#django.contrib.auth.middleware.AuthenticationMiddleware "django.contrib.auth.middleware.AuthenticationMiddleware")之后。





### CSRF保护中间件



_class_ `CsrfViewMiddleware`[[source]](../_modules/django/middleware/csrf.html#CsrfViewMiddleware)



添加跨站点请求伪造的保护，通过向POST表单添加一个隐藏的表单字段，并检查请求中是否有正确的值。详见[_CSRF保护文档_](csrf.html)。





### X-Frame-Options中间件



_class_ `XFrameOptionsMiddleware`[[source]](../_modules/django/middleware/clickjacking.html#XFrameOptionsMiddleware)



[_通过X-Frame-Options协议头进行简单的点击劫持保护_](clickjacking.html)。







## 中间件的排序

下面是一些关于Django中间件排序的提示。

1.  [`UpdateCacheMiddleware`](#django.middleware.cache.UpdateCacheMiddleware "django.middleware.cache.UpdateCacheMiddleware")

    放在修改`大量`协议头的中间件(`SessionMiddleware`, `GZipMiddleware`, `LocaleMiddleware`)之前。

2.  [`GZipMiddleware`](#django.middleware.gzip.GZipMiddleware "django.middleware.gzip.GZipMiddleware")

    放在任何可能修改或使用响应消息体的中间件之前。

    放在`UpdateCacheMiddleware`之后：会修改`大量`的协议头。

3.  [`ConditionalGetMiddleware`](#django.middleware.http.ConditionalGetMiddleware "django.middleware.http.ConditionalGetMiddleware")

    放在`CommonMiddleware`之前：当[`USE_ETAGS`](settings.html#std:setting-USE_ETAGS) = `True`时会使用它的`Etag` 协议头。

4.  [`SessionMiddleware`](#django.contrib.sessions.middleware.SessionMiddleware "django.contrib.sessions.middleware.SessionMiddleware")

    放在`UpdateCacheMiddleware`之后：会修改 `大量`协议头。

5.  [`LocaleMiddleware`](#django.middleware.locale.LocaleMiddleware "django.middleware.locale.LocaleMiddleware")

    放在`SessionMiddleware`（由于使用会话数据）和?`CacheMiddleware`（由于要修改`大量`协议头）之后的最上面。

6.  [`CommonMiddleware`](#django.middleware.common.CommonMiddleware "django.middleware.common.CommonMiddleware")

    放在任何可能修改相应的中间件之前（因为它会生成`ETags`）。

    在`GZipMiddleware`之后，不会在压缩后的内容上再去生成`ETag`。

    尽可能放在靠上面的位置，因为[`APPEND_SLASH`](settings.html#std:setting-APPEND_SLASH)或者[`PREPEND_WWW`](settings.html#std:setting-PREPEND_WWW)设置为?`True`时会被重定向。

7.  [`CsrfViewMiddleware`](#django.middleware.csrf.CsrfViewMiddleware "django.middleware.csrf.CsrfViewMiddleware")

    放在任何假设CSRF攻击被处理的视图中间件之前。

8.  [`AuthenticationMiddleware`](#django.contrib.auth.middleware.AuthenticationMiddleware "django.contrib.auth.middleware.AuthenticationMiddleware")

    放在`SessionMiddleware`之后：因为它使用会话存储。

9.  [`MessageMiddleware`](#django.contrib.messages.middleware.MessageMiddleware "django.contrib.messages.middleware.MessageMiddleware")

    放在`SessionMiddleware`之后：会使用基于会话的存储。

10.  [`FetchFromCacheMiddleware`](#django.middleware.cache.FetchFromCacheMiddleware "django.middleware.cache.FetchFromCacheMiddleware")

    放在任何修改`大量`协议头的中间件之后：协议头被用来从缓存的哈希表中获取值。

11.  [`FlatpageFallbackMiddleware`](contrib/flatpages.html#django.contrib.flatpages.middleware.FlatpageFallbackMiddleware "django.contrib.flatpages.middleware.FlatpageFallbackMiddleware")

    应该放在最底下，因为它是中间件中的最后一手。

12.  [`RedirectFallbackMiddleware`](contrib/redirects.html#django.contrib.redirects.middleware.RedirectFallbackMiddleware "django.contrib.redirects.middleware.RedirectFallbackMiddleware")

    应该放在最底下，因为它是中间件中的最后一手。



