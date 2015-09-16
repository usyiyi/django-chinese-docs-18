# Django安全

这份文档是 Django 的安全功能的概述。 它包括给 Django 驱动的网站一些加固建议。

## 跨站脚本&nbsp;(XSS) 防护

XSS攻击允许用户注入客户端脚本到其他用户的浏览器里。 这通常是通过存储在数据库中的恶意脚本，它将检索并显示给其他用户，或者通过让用户点击一个链接，这将导致攻击者的 JavaScript 被用户的浏览器执行。 然而，XSS 攻击可以来自任何不受信任的源数据，如 Cookie 或 Web 服务，任何没有经过充分处理就包含在网页中的数据。

使用 Django 模板保护你免受多数 XSS 攻击。 然而，重要的是要了解它提供了什么保护及其局限性。

Django 模板会[_ 编码特殊字符 _](../ref/templates/language.html#automatic-html-escaping) ，这些字符在 HTML 中都是特别危险的。 虽然这可以防止大多数恶意输入的用户，但它不能完全保证万无一失。 例如，它不会防护以下内容：

```
<style class=>...</style>

```

如果 `var` 设置为 `'class1 onmouseover=javascript:func()'`, 这可能会导致在未经授权的 JavaScript 的执行，取决于浏览器如何呈现不完整的 HTML。 （对属性值使用引号可以修复这种情况。）

同样重要的是`is_safe`要特别小心的用在 自定义模板标签，[`safe`](../ref/templates/builtins.html#std:templatefilter-safe) 模板标签，[`mark_safe`](../ref/utils.html#module-django.utils.safestring "django.utils.safestring: Functions and classes for working with strings that can be displayed safely without further escaping in HTML.") ，还有 autoescape 被关闭的时候。

此外，如果您使用的是模板系统输出 HTML 以外的东西，可能会有完全不同的字符和单词需要编码。

你也应该在数据库中存储 HTML 的时候要非常小心，尤其是当 HTML 被检索然后展示出来。

## 跨站请求伪造&nbsp;(CSRF) 防护

CSRF 攻击允许恶意用户在另一个用户不知情或者未同意的情况下，以他的身份执行操作。

Django 对大多数类型的 CSRF 攻击有内置的保护，在适当情况下你可以[_开启并使用它_](../ref/csrf.html#using-csrf) 。 然而，对于任何解决技术，都有它的局限性。 例如，CSRF 模块可以在全局范围内或为特定视图被禁用 。 您应该只在您知道在做什么的情况下操作。 还有其他 [_限制_](../ref/csrf.html#csrf-limitations) 如果你的网站有子域名并且在你的控制之外。

[_CSRF 防护_](../ref/csrf.html#how-csrf-works) 是通过检查每个 POST 请求的一个随机数（nonce）来工作。 这确保了恶意用户不能简单“回放”你网站上面表单的POST，以及让另一个登录的用户无意中提交表单。恶意用户必须知道这个随机数，它是用户特定的（存在cookie里）。

使用&nbsp;[_HTTPS_](#security-recommendation-ssl)来部署的时候，`CsrfViewMiddleware`会检查HTTP referer协议头是否设置为同源的URL（包括子域和端口）。因为HTTPS提供了附加的安全保护，转发不安全的连接请求时，必须确保链接使用 HTTPS，并使用HSTS支持的浏览器。

使用`csrf_exempt`装饰器来标记视图时，要非常小心，除非这是极其必要的。

## SQL 注入保护

SQl注入是一种攻击类型，恶意用户可以在系统数据库中执行任意SQL代码。这可能会导致记录删除或者数据泄露。

通过使用Django的查询集，产生的SQL会由底层数据库驱动正确地转义。然而，Django也允许开发者编写[_原始查询_](db/sql.html#executing-raw-queries)或者执行[_自定义sql_](db/sql.html#executing-custom-sql)。这些功能应该谨慎使用，并且你应该时刻小心正确转义任何用户可以控制的参数。另外，你在使用[`extra()`](../ref/models/querysets.html#django.db.models.query.QuerySet.extra "django.db.models.query.QuerySet.extra")的时候应该谨慎行事。

## 点击劫持保护

点击劫持是一类攻击，恶意站点在一个frame中包裹了另一个站点。这类攻击可能导致用户被诱导在目标站点做出一些无意识的行为。

Django在[`X-Frame-Options 中间件`](../ref/middleware.html#django.middleware.clickjacking.XFrameOptionsMiddleware "django.middleware.clickjacking.XFrameOptionsMiddleware")的表单中中含有&nbsp;[_点击劫持保护 _](../ref/clickjacking.html#clickjacking-prevention)，它在支持的浏览器中可以保护站点免于在frame中渲染。也可以在每个视图中禁止这一保护，或者配置要发送的额外的协议头。

对于任何不需要将页面包装在三方站点的frame中，或者只需要包含它的一部分的站点，都强烈推荐启用这一中间件。

## SSL/HTTPS

把你的站点部署在HTTPS下总是更安全的，尽管在所有情况下不都有效。如果不这样，恶意的网络用户可能会嗅探授权证书，或者其他在客户端和服务端之间传输的信息，或者一些情况下 --&nbsp;**活跃的**网络攻击者 -- 会修改在两边传输的数据。

如果你想要使用HTTPS提供的保护，并且在你的服务器上开启它，你需要遵循一些额外的步骤：

*   如果必要的话，设置 [`SECURE_PROXY_SSL_HEADER`](../ref/settings.html#std:setting-SECURE_PROXY_SSL_HEADER)，确保你已经彻底了解警告。未能实现它会导致CSRF方面的缺陷，也是很危险的！

*   设置重定向，以便HTTP下的请求可以重定向到HTTPS。

    这可以通过自定义的中间件来实现。请注意[`SECURE_PROXY_SSL_HEADER`](../ref/settings.html#std:setting-SECURE_PROXY_SSL_HEADER)下的警告。对于反向代理的情况，配置web主服务器来重定向到HTTPS或许是最简单也许是最安全的做法。

*   使用“安全的”cookie。

    如果浏览器的连接一开始通过HTTP，这是大多数浏览器的通常情况，已存在的cookie可能会被泄露。因此，你应该将[`SESSION_COOKIE_SECURE`](../ref/settings.html#std:setting-SESSION_COOKIE_SECURE) 和[`CSRF_COOKIE_SECURE`](../ref/settings.html#std:setting-CSRF_COOKIE_SECURE)设置为`True`。这会使浏览器只在HTTPS连接中发送这些cookie。要注意这意味着会话在HTTP下不能工作，并且CSRF保护功能会在HTTP下阻止接受任何POST数据（如果你把所有HTTP请求都重定向到HTTPS之后就没问题了）。

*   使用 HTTP 强制安全传输 (HSTS)

    HSTS 是一个HTTP协议头，它通知浏览器，到特定站点的所有链接都一直使用HTTPS。通过和重定向HTTP请求到HTTPS一起使用，确保连接总是享有附加的SSL安全保障，由一个已存在的成功的连接提供。HSTS通常在web服务器上面配置。

## Host 协议头验证

在某些情况下，Django使用客户端提供的`Host` 协议头来构造URL。虽然这些值可以被审查，来防止跨站脚本攻击（XSS），但是一个假的`Host`值可以用于跨站请求伪造（CSRF），有害的缓存攻击，以及email中的有害链接。

Because even seemingly-secure web server configurations are susceptible to fake `Host` headers, Django validates `Host` headers against the [`ALLOWED_HOSTS`](../ref/settings.html#std:setting-ALLOWED_HOSTS) setting in the [`django.http.HttpRequest.get_host()`](../ref/request-response.html#django.http.HttpRequest.get_host "django.http.HttpRequest.get_host") method.

验证只通过[`get_host()`](../ref/request-response.html#django.http.HttpRequest.get_host "django.http.HttpRequest.get_host")来应用；如果你的代码从`request.META`中直接访问`Host`协议头，就会绕开这一安全防护。

详见完整的[`ALLOWED_HOSTS`](../ref/settings.html#std:setting-ALLOWED_HOSTS)文档。

Warning

Previous versions of this document recommended configuring your web server to ensure it validates incoming HTTP `Host` headers. While this is still recommended, in many common web servers a configuration that seems to validate the `Host` header may not in fact do so. For instance, even if Apache is configured such that your Django site is served from a non-default virtual host with the `ServerName` set, it is still possible for an HTTP request to match this virtual host and supply a fake `Host` header. Thus, Django now requires that you set [`ALLOWED_HOSTS`](../ref/settings.html#std:setting-ALLOWED_HOSTS) explicitly rather than relying on web server configuration.

另外，就像1.3.1，如果你的配置需要它的话，Django 需要你显式开启对`X-Forwarded-Host` 协议头的支持(通过 [`USE_X_FORWARDED_HOST`](../ref/settings.html#std:setting-USE_X_FORWARDED_HOST) 这只)。

## Session 会话安全

类似于部署在站点上的[_CSRF 限制_](../ref/csrf.html#csrf-limitations) 使不受信任的用户不能访问任何子域，[`django.contrib.sessions`](http/sessions.html#module-django.contrib.sessions "django.contrib.sessions: Provides session management for Django projects.")也有一些限制。详见[_安全中会话的话题指南_](http/sessions.html#topics-session-security)。

## 用户上传的内容

注意

考虑[_在云服务器或CDN上面部署静态文件_](../howto/static-files/deployment.html#staticfiles-from-cdn)来避免一些此类问题。

*   如果你的站点接受上传文件，强烈推荐你在web服务器配置中，将这些上传限制为合理的大小，来避免拒绝服务（DOS）攻击。在Apache中，这可以简单地使用[LimitRequestBody](http://httpd.apache.org/docs/2.2/mod/core.html#limitrequestbody)指令。

*   如果你自己处理静态文件，确保像Apache的`mod_php`的处理器已关闭，它会将静态文件执行为代码。你并不希望用户能够通过上传和请求一个精心构造的文件来执行任意代码。

*   Django’s media upload handling poses some vulnerabilities when that media is served in ways that do not follow security best practices. Specifically, an HTML file can be uploaded as an image if that file contains a valid PNG header followed by malicious HTML. This file will pass verification of the library that Django uses for [`ImageField`](../ref/models/fields.html#django.db.models.ImageField "django.db.models.ImageField") image processing (Pillow). When this file is subsequently displayed to a user, it may be displayed as HTML depending on the type and configuration of your web server.

    No bulletproof technical solution exists at the framework level to safely validate all user uploaded file content, however, there are some other steps you can take to mitigate these attacks:

    *   One class of attacks can be prevented by always serving user uploaded content from a distinct top-level or second-level domain. This prevents any exploit blocked by [same-origin policy](http://en.wikipedia.org/wiki/Same-origin_policy) protections such as cross site scripting. For example, if your site runs on `example.com`, you would want to serve uploaded content (the [`MEDIA_URL`](../ref/settings.html#std:setting-MEDIA_URL) setting) from something like `usercontent-example.com`. It’s _not_ sufficient to serve content from a subdomain like `usercontent.example.com`.
    2.  除此之外，应用可以选择为用户上传的文件定义一个允许的文件扩展名的白名单，并且配置web服务器直处理这些文件。

## 额外的安全话题

虽然Django提供了开箱即用的，良好的安全保护，但是合理地部署你的应用，以及利用web服务器、操作系统和其他组件的安全保护仍然很重要。

*   确保你的Python代码在web服务器的根目录外。这会确保你的Python代码不会意外被解析为纯文本（或者意外被执行）。
*   小心处理任何[_用户上传的文件_](../ref/models/fields.html#file-upload-security)。
*   Django并不限制验证用户的请求。要保护对验证系统的暴力破解攻击，你可以考虑部署一个DJango的插件或者web服务器模块来限制这些请求。
*   秘密保存[`SECRET_KEY`](../ref/settings.html#std:setting-SECRET_KEY)。
*   使用防火墙来限制缓存系统和数据库的访问是个好主意。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Security overview](https://docs.djangoproject.com/en/1.8/topics/security/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
