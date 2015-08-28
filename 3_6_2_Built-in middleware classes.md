<!--
  译者：Github@wizardforcel
-->

# 中间件 #

这篇文档介绍了Django自带的所有中间件组件。 要查看关于如何使用它们以及如何编写自己的中间件，请见中间件使用指导。

## 可用的中间件 ##

### 缓存中间件 ###

**class UpdateCacheMiddleware[source]**

**class FetchFromCacheMiddleware[source]**

开启全站范围的缓存。 如果开启了这些缓存，任何一个由Django提供的页面将会被缓存，缓存时长是由你在CACHE_MIDDLEWARE_SECONDS配置中定义的。详见缓存文档。

### "常用"的中间件 ###

**class CommonMiddleware[source]**

给完美主义者增加一些便利条件：

+ 禁止访问DISALLOWED_USER_AGENTS中设置的用户代理，这项配置应该是一个已编译的正则表达式对象的列表。
+ 基于APPEND_SLASH和PREPEND_WWW的设置来重写URL。

如果APPEND_SLASH设为True并且一开始的的URL没有以斜线结尾，并且在URLconf中也没找到对应定义，这时形成一个一个斜线结尾新的URL。如果这个新的URL存在于URLconf，这时Django会重定向请求到这个新URL上，否则，一开始的URL按正常情况处理。

比如，foo.com/bar将会被重定向到foo.com/bar/，如果你没有为foo.com/bar定义有效的正则，但是为foo.com/bar/定义了有效的正则。

如果PREPEND_WWW设为True，前面缺少 "www."的url将会被重定向到相同但是以一个"www."开头的url。

两种选项都是为了规范化url。其中的哲学就是，任何一个url应该在一个地方仅存在一个。技术上来讲，url foo.com/bar  区别于foo.com/bar/ --  搜索引擎索引会把这里分开处理 -- 因此，最佳实践就是规范化url。

+ 基于  USE_ETAGS  设置来处理ETag。如果设置USE_ETAGS为True，Django会通过MD5-hashing处理页面的内容来为每一个页面请求计算Etag，并且如果合适的话，它将会发送携带Not Modified的响应。

**CommonMiddleware.response_redirect_class**

```
Django 1.8中新增
```

默认为HttpResponsePermanentRedirect。它继承了CommonMiddleware，并覆写了属性来自定义中间件发出的重定向。

**class BrokenLinkEmailsMiddleware[source]**

+ 向MANAGERS发送死链提醒邮件（详见错误报告）。

### GZip中间件 ###

**class GZipMiddleware[source]**

> 警告
> 
> 安全研究员最近发现，当压缩技术（包括GZipMiddleware）用于一个网站的时候，网站会受到一些可能的攻击。此外，这些方法可以用于破坏Django的CSRF保护。在你的站点使用GZipMiddleware之前，你应该先仔细考虑一下你的站点是否容易受到这些攻击。 如果你不确定是否会受到这些影响，应该避免使用 GZipMiddleware。详见the BREACH paper (PDF)和breachattack.com。

为支持GZip压缩的浏览器（一些现代的浏览器）压缩内容。

建议把这个中间件放到中间件配置列表的第一个，这样压缩响应内容的处理会到最后才 发生。

如果满足下面条件的话，内容不会被压缩：

+ 消息体的长度小于200个字节。
+ 响应已经设置了Content-Encoding协议头。
+ 请求（浏览器）没有发送包含gzip的Accept-Encoding协议头。

你可以通过这个gzip_page()装饰器使用独立的GZip压缩。

### 带条件判断的GET中间件 ###

**class ConditionalGetMiddleware[source]**

处理带有条件判断状态GET操作。 如果一个请求包含 ETag 或者Last-Modified协议头，并且请求包含If-None-Match或If-Modified-Since，这时响应会被 替换为HttpResponseNotModified。

另外，它会设置Date和Content-Length响应头。

### 本地中间件 ###

**class LocaleMiddleware[source]**

基于请求中的数据开启语言选择。 它可以为每个用户进行定制。 详见国际化文档。

**LocaleMiddleware.response_redirect_class**

默认为HttpResponseRedirect。继承自LocaleMiddleware并覆写了属性来自定义中间件发出的重定向。

### 消息中间件 ###

**class MessageMiddleware[source]**

开启基于cookie或会话的消息支持。详见消息文档。

### 安全中间件 ###

> 警告
> 
> 如果你的部署环境允许的话，让你的前端web服务器展示SecurityMiddleware提供的功能是个好主意。这样一来，如果有任何请求没有被Django处理（比如静态媒体或用户上传的文件），他们会拥有和向Django应用的请求相同的保护。

**class SecurityMiddleware[source]**

```
Django 1.8中新增
```

django.middleware.security.SecurityMiddleware为请求/响应循环提供了几种安全改进。每一种可以通过一个选项独立开启或关闭。

+ SECURE_BROWSER_XSS_FILTER
+ SECURE_CONTENT_TYPE_NOSNIFF
+ SECURE_HSTS_INCLUDE_SUBDOMAINS
+ SECURE_HSTS_SECONDS
+ SECURE_REDIRECT_EXEMPT
+ SECURE_SSL_HOST
+ SECURE_SSL_REDIRECT

### HTTP Strict Transport Security (HSTS) ###

对于那些应该只能通过HTTPS访问的站点，你可以通过设置HSTS协议头，通知现代的浏览器，拒绝用不安全的连接来连接你的域名。这会降低你受到SSL-stripping的中间人（MITM）攻击的风险。

如果你将SECURE_HSTS_SECONDS设置为一个非零值，SecurityMiddleware会在所有的HTTPS响应中设置这个协议头。

开启HSTS的时候，首先使用一个小的值来测试它是个好主意，例如，让SECURE_HSTS_SECONDS = 3600为一个小时。每当浏览器在你的站点看到HSTS协议头，都会在提供的时间段内绝对使用不安全（HTTP）的方式连接到你的域名。一旦你确认你站点上的所有东西都以安全的方式提供（例如，HSTS并不会干扰任何事情），建议你增加这个值，这样不常访问你站点的游客也会被保护（比如，一般设置为31536000秒，一年）。

另外，如果你将 SECURE_HSTS_INCLUDE_SUBDOMAINS设置为True,，SecurityMiddleware会将includeSubDomains标签添加到Strict-Transport-Security协议头中。强烈推荐这样做（假设所有子域完全使用HTTPS），否则你的站点仍旧有可能由于子域的不安全连接而受到攻击。

> 警告
> 
> HSTS策略在你的整个域中都被应用，不仅仅是你所设置协议头的响应中的url。所以，如果你的整个域都设置为HTTPS only，你应该只使用HSTS策略。
> 
> 适当遵循HSTS协议头的浏览器，会通过显示警告的方式，拒绝让用户连接到证书过期的、自行签署的、或者其他SSL证书无效的站点。如果你使用了HSTS，确保你的证书处于一直有效的状态！

> 注意
> 
> 如果你的站点部署在负载均衡器或者反向代理之后，并且Strict-Transport-Security协议头没有添加到你的响应中，原因是Django有可能意识不到这是一个安全连接。你可能需要设置SECURE_PROXY_SSL_HEADER。

### X-Content-Type-Options: nosniff ###

一些浏览器会尝试猜测他们所得内容的类型，而不是读取Content-Type协议头。虽然这样有助于配置不当的服务器正常显示内容，但也会导致安全问题。

如果你的站点允许用户上传文件，一些恶意的用户可能会上传一个精心构造的文件，当你觉得它无害的时候，文件会被浏览器解释成HTML或者Javascript。

欲知更多有关这个协议头和浏览器如何处理它的内容，你可以在IE安全博客中读到它。

要防止浏览器猜测内容类型，并且强制它一直使用 Content-Type协议头中提供的类型，你可以传递X-Content-Type-Options: nosniff协议头。SecurityMiddleware将会对所有响应这样做，如果SECURE_CONTENT_TYPE_NOSNIFF 设置为True。

注意在大多数Django不涉及处理上传文件的部署环境中，这个设置不会有任何帮助。例如，如果你的MEDIA_URL被前端web服务器直接处理（例如nginx和Apache），你可能想要在那里设置这个协议头。而在另一方面，如果你使用Django执行为了下载文件而请求授权之类的事情，并且你不能使用你的web服务器设置协议头，这个设置会很有用。

### X-XSS-Protection: 1; mode=block ###

一些浏览器能够屏蔽掉出现XSS攻击的内容。通过寻找页面中GET或者POST参数中的JavaScript内容来实现。如果JavaScript在服务器的响应中被重放，页面就会停止渲染，并展示一个错误页来取代。

X-XSS-Protection协议头用来控制XSS过滤器的操作。

要在浏览器中启用XSS过滤器，并且强制它一直屏蔽可疑的XSS攻击，你可以在协议头中传递X-XSS-Protection: 1; mode=block。  如果SECURE_BROWSER_XSS_FILTER设置为True，SecurityMiddleware会在所有响应中这样做。

> 警告
> 
> 浏览器的XSS过滤器是一个十分有效的手段，但是不要过度依赖它。它并不能检测到所有的XSS攻击，也不是所有浏览器都支持这一协议头。确保你校验和过滤了所有的输入来防止XSS攻击。

### SSL重定向 ###

如果你同时提供HTTP和HTTPS连接，大多数用户会默认使用不安全的（HTTP）链接。为了更高的安全性，你应该讲所有HTTP连接重定向到HTTP连接。

如果你将SECURE_SSL_REDIRECT设置为True，SecurityMiddleware会将HTTP链接永久地（HTTP 301，permanently）重定向到HTTPS连接。

> 注意
> 
> 由于性能因素，最好在Django外面执行这些重定向，在nginx这种前端负载均衡器或者反向代理服务器中执行。SECURE_SSL_REDIRECT专门为这种部署情况而设计，当这不可选择的时候。

如果SECURE_SSL_HOST设置有一个值，所有重定向都会发到值中的主机，而不是原始的请求主机。

如果你站点上的一些页面应该以HTTP方式提供，并且不需要重定向到HTTPS，你可以SECURE_REDIRECT_EXEMPT设置中列出匹配那些url的正则表达式。

> 注意
> 
> 如果你在负载均衡器或者反向代理服务器后面部署应用，而且Django不能辨别出什么时候一个请求是安全的，你可能需要设置SECURE_PROXY_SSL_HEADER。

### 会话中间件 ###

**class SessionMiddleware[source]**

开启会话支持。详见会话文档。

### 站点中间件 ###

**class CurrentSiteMiddleware[source]**

```
Django 1.7中新增
```

向每个接收到的HttpRequest对象添加一个site属性，表示当前的站点。详见站点文档。

### 认证中间件 ###

**class AuthenticationMiddleware[source]**

向每个接收到的HttpRequest对象添加user属性，表示当前登录的用户。详见web请求中的认证。

**class RemoteUserMiddleware[source]**

使用web服务器提供认证的中间件。详见使用REMOTE_USER进行认证。

**class SessionAuthenticationMiddleware[source]**

```
Django 1.7中新增
```

当用户修改密码的时候使用户的会话失效。详见密码更改时的会话失效。在MIDDLEWARE_CLASSES中，这个中间件必须出现在django.contrib.auth.middleware.AuthenticationMiddleware之后。

### CSRF保护中间件 ###

**class CsrfViewMiddleware[source]**

添加跨站点请求伪造的保护，通过向POST表单添加一个隐藏的表单字段，并检查请求中是否有正确的值。详见CSRF保护文档。

### X-Frame-Options中间件 ###

**class XFrameOptionsMiddleware[source]**

通过X-Frame-Options协议头进行简单的点击劫持保护。

## 中间件的排序 ##

下面是一些关于Django中间件排序的提示。

**UpdateCacheMiddleware**

放在修改大量协议头的中间件(SessionMiddleware, GZipMiddleware, LocaleMiddleware)之前。

**GZipMiddleware**

放在任何可能修改或使用响应消息体的中间件之前。

放在UpdateCacheMiddleware之后：会修改大量的协议头。

**ConditionalGetMiddleware**

放在CommonMiddleware之前：当USE_ETAGS = True时会使用它的Etag 协议头。

**SessionMiddleware**

放在UpdateCacheMiddleware之后：会修改 大量协议头。

**LocaleMiddleware**

放在SessionMiddleware（由于使用会话数据）和 CacheMiddleware（由于要修改大量协议头）之后的最上面。

**CommonMiddleware**

放在任何可能修改相应的中间件之前（因为它会生成ETags）。

在GZipMiddleware之后，不会在压缩后的内容上再去生成ETag。

尽可能放在靠上面的位置，因为APPEND_SLASH或者PREPEND_WWW设置为 True时会被重定向。

**CsrfViewMiddleware**

放在任何假设CSRF攻击被处理的视图中间件之前。

**AuthenticationMiddleware**

放在SessionMiddleware之后：因为它使用会话存储。

**MessageMiddleware**

放在SessionMiddleware之后：会使用基于会话的存储。

**FetchFromCacheMiddleware**

放在任何修改大量协议头的中间件之后：协议头被用来从缓存的哈希表中获取值。

**FlatpageFallbackMiddleware**

应该放在最底下，因为他是中间件中的底牌。

**RedirectFallbackMiddleware**

应该放在最底下，因为他是中间件中的底牌。