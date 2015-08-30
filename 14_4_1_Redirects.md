# 重定向应用 #

Django 原生自带一个可选的重定向应用。它将简单的重定向保存到数据库中并处理重定向。它默认使用HTTP 响应状态码`301 Moved Permanently`。

## 安装 ##

请依照下面的步骤安装重定向应用：

1. 确保`django.contrib.sites` 框架已经安装。
2. 添加'`django.contrib.redirects`' 到 `INSTALLED_APPS` 设置中。
3. 添加'`django.contrib.redirects.middleware.RedirectFallbackMiddleware`' 到`MIDDLEWARE_CLASSES` 设置中。
4. 运行命令`manage.py migrate`。

## 它是如何工作的 ##

`manage.py migrate` 在数据库中创建一张`django_redirect` 表。它是一张简单的查询表，具有`site_id`、`old_path` 和`new_path` 字段。

`RedirectFallbackMiddleware` 完成所有的工作。每当Django 的应用引发一个404 错误，该中间件将到重定向数据库中检查请求的URL。它会根据`old_path` 和`SITE_ID` 设置的站点ID 查找重定向的路径。

+ 如果找到匹配的记录且`new_path `不为空，它将使用301(“Moved Permanently”)重定向到`new_path` 。你可以子类化`RedirectFallbackMiddleware` 并设置 `response_redirect_class` 为`django.http.HttpResponseRedirect` 来使用302 Moved Temporarily 重定向。
+ 如果找到匹配的记录而`new_path` 为空，它将发送一个410 (“Gone”) HTTP 头和空（没有内容的）响应。
+ 如果没有找到匹配的记录，请求将继续正常处理。

这个中间件只针对404 错误启用 —— 不能用于500 或其它状态码。

注意`MIDDLEWARE_CLASSES` 的顺序很重要。通常可以将`RedirectFallbackMiddleware` 放在列表的最后，因为它最后执行。

更多的信息可以阅读[中间件的文档](http://python.usyiyi.cn/django/topics/http/middleware.html)。

## 如何添加、修改和删除重定向 ##

### 通过Admin 接口 ###

如果你已经启用Django 自动生成的`Admin` 接口，你应该可以在`Admin` 的主页看到“Redirects”部分。编辑这些重定向，就像编辑系统中的其它对象一样。

### 通过Python API ###

`class models.Redirect`

重定向通过一个标准的Django 模型表示，位于`django/contrib/redirects/models.py`。你可以通过Django 的数据库API 访问重定向对象。

## 中间件 ##

`class middleware.RedirectFallbackMiddleware`

你可以通过创建`RedirectFallbackMiddleware` 的子类并覆盖`response_gone_class` 和/或`response_redirect_class` 来修改中间件使用的`HttpResponse`类。

`response_gone_class`

```
New in Django 1.7.
```

`HttpResponse` 类，用于找不到请求路径的`Redirect`或找到的`new_path` 值为空的时候。

默认为`HttpResponseGone`。

`response_redirect_class`

```
New in Django 1.7.
```

处理重定向的`HttpResponse` 类。

默认为`HttpResponsePermanentRedirect`。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Redirects](https://docs.djangoproject.com/en/1.8/ref/contrib/redirects/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
