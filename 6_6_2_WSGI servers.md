# 如何使用WSGI 部署 #

Django 首要的部署平台是[WSGI](http://www.wsgi.org/)，它是Python Web 服务器和应用的标准。

Django 的`startproject` 管理命名为你设置一个简单的默认WSGI 配置，你可以根据你项目的需要做调整并指定任何与WSGI 兼容的应用服务器使用。

Django 包含以下WSGI 服务器的入门文档：

+ [如何使用Apache 和mod_wsgi 部署Django](http://python.usyiyi.cn/django/howto/deployment/wsgi/modwsgi.html)
+ [从Apache 中利用Django 的用户数据库进行认证](http://python.usyiyi.cn/django/howto/deployment/wsgi/apache-auth.html)
+ [如何使用Gunicorn 部署Django (100%)](http://python.usyiyi.cn/django/howto/deployment/wsgi/gunicorn.html)
+ [如何使用uWSGI 部署Django (100%)](http://python.usyiyi.cn/django/howto/deployment/wsgi/uwsgi.html)

## application 对象 ##

使用WSGI 部署的核心概览是`application` 可调用对象，应用服务器使用它来与你的代码进行交换。在Python 模块中，它通常一个名为`application` 的对象提供给服务器使用。

`startproject` 命令创建一个`<project_name>/wsgi.py` 文件，它就包含这样一个`application` 可调用对象。

它既可用于Django 的开发服务器，也可以用于线上WSGI 的部署。

WSGI 服务器从它们的配置中获得`application` 可调用对象的路径。Django 内建的服务器，叫做`runserver` 和`runfcgi` 命令，是从`WSGI_APPLICATION` 设置中读取它。默认情况下，它设置为`<project_name>.wsgi.application`，指向`<project_name>/wsgi.py` 中的`application` 可调用对象。

## 配置settings 模块 ##

当WSGI 服务器加载你的应用时，Django 需要导入settings 模块 —— 这里是你的全部应用定义的地方。

Django 使用`DJANGO_SETTINGS_MODULE` 环境变量来定位settings 模块。它包含settings 模块的路径，以点分法表示。对于开发环境和线上环境，你可以使用不同的值；这完全取决于你如何组织你的settings。

如果这个变量没有设置，默认的`wsgi.py` 设置为`mysite.settings`，其中`mysite` 为你的项目的名称。这是`runserver` 如何找到默认的`settings` 文件的机制。

> 注
>
> 因为环境变量是进程范围的，当你在同一个进程中运行多个Django 站点时，它将不能工作。使用`mod_wsgi` 就是这个情况。
>
> 为了避免这个问题，可以使用mod_wsgi 的守护进程模式，让每个站点位于它自己的守护进程中，或者在`wsgi.py`中通过强制使用`os.environ["DJANGO_SETTINGS_MODULE"] = "mysite.settings"` 来覆盖这个值。

## 运用WSGI 中间件 ##

你可以简单地封装application 对象来运用 [WSGI 中间件](https://www.python.org/dev/peps/pep-3333/#middleware-components-that-play-both-sides)。 例如，你可以在`wsgi.py` 的底下添加以下这些行：

```
from helloworld.wsgi import HelloWorldApplication
application = HelloWorldApplication(application)
```

如果你结合使用 Django 的application 与另外一个WSGI application 框架，你还可以替换Django WSGI 的application 为一个自定义的WSGI application。

> 注
>
> 某些第三方的WSGI 中间件在处理完一个请求后不调用响应对象上的`close` —— most notably Sentry’s error reporting middleware up to version 2.0.7。这些情况下，不会发送`request_finished` 信号。这可能导致数据库和memcache 服务的空闲连接。

&zwj;

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[WSGI servers](https://docs.djangoproject.com/en/1.8/howto/deployment/wsgi/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
