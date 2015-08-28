# 内建的视图 #

有几个Django 的内建视图在编写视图 中讲述，文档的其它地方也会有所讲述。

## 开发环境中的文件服务器 ##

**static.serve(request, path, document_root, show_indexes=False)**

在本地的开发环境中，除了你的项目中的静态文件，可能还有一些文件，出于方便，你希望让Django 来作为服务器。serve() 视图可以用来作为任意目录的服务器。（该视图不能用于生产环境，应该只用于开发时辅助使用；在生产环境中你应该使用一个真实的前端Web 服务器来服务这些文件）。

最常见的例子是用户上传文档到`MEDIA_ROOT` 中。`django.contrib.staticfiles` 用于静态文件且没有对用户上传的文件做处理，但是你可以通过在URLconf 中添加一些内容来让Django 作为MEDIA_ROOT 的服务器：

```
from django.conf import settings
from django.views.static import serve

# ... the rest of your URLconf goes here ...

if settings.DEBUG:
    urlpatterns += [
        url(r'^media/(?P<path>.*)$', serve, {
            'document_root': settings.MEDIA_ROOT,
        }),
   ]
```

注意，这里的代码片段假设你的`MEDIA_URL`的值为`'/media/'`。它将调用`serve()` 视图，传递来自URLconf 的路径和（必选的）`document_root` 参数。

因为定义这个URL 模式显得有些笨拙，Django 提供一个小巧的URL 辅助函数`static()`，它接收`MEDIA_URL`这样的参数作为前缀和视图的路径如`'django.views.static.serve'`。其它任何函数参数都将透明地传递给视图。

## 错误视图 ##

Django 原生自带几个默认视图用于处理HTTP 错误。若要使用你自定义的视图覆盖它们，请参见自定义错误视图。

### 404 (page not found) 视图 ###

**defaults.page_not_found(request, template_name='404.html')**

当你在一个视图中引发Http404 时，Django 将加载一个专门的视图用于处理404 错误。默认为`django.views.defaults.page_not_found()` 视图，它产生一个非常简单的“Not Found” 消息或者渲染`404.html`模板，如果你在根模板目录下创建了它的话。

默认的404 视图将传递一个变量给模板：`request_path`，它是导致错误的URL。

关于404 视图需要注意的3点：

+ 如果Django 在检测URLconf 中的每个正则表达式后没有找到匹配的内容也将调用404 视图。
+ 404 视图会被传递一个`RequestContext`并且可以访问模板上下文处理器提供的变量（例如`MEDIA_URL`）。
+ 如果DEBUG 设置为`True`（在你的`settings` 模块中），那么将永远不会调用404 视图，而是显示你的URLconf 并带有一些调试信息。

### 500 (server error) 视图 ###

**defaults.server_error(request, template_name='500.html')**

类似地，在视图代码中出现运行时错误，Django 将执行特殊情况下的行为。如果一个视图导致异常，Django 默认情况下将调用`django.views.defaults.server_error` 视图，它产生一个非常简单的“Server Error” 消息或者渲染`500.html`，如果你在你的根模板目录下定义了它的话。

默认的500 视图不会传递变量给`500.html` 模板，且使用一个空`Context` 来渲染以减少再次出现错误的可能性。

如果`DEBUG` 设置为`True`（在你的`settings` 模块中），那么将永远不会调用500 视图，而是显示回溯并带有一些调试信息。

### 403 (HTTP Forbidden) 视图 ###

**defaults.permission_denied(request, template_name='403.html')**

与404 和500 视图一样，Django 具有一个处理403 Forbidden 错误的视图。如果一个视图导致一个403 视图，那么Django 将默认调用`django.views.defaults.permission_denied`视图。

该视图加载并渲染你的根模板目录下的`403.html`，如果这个文件不存在则根据RFC 2616（HTTP 1.1 Specification）返回“403 Forbidden”文本。

`django.views.defaults.permission_denied` 通过`PermissionDenied` 异常触发。若要拒绝访问一个视图，你可以这样视图代码：

```
from django.core.exceptions import PermissionDenied

def edit(request, pk):
    if not request.user.is_staff:
        raise PermissionDenied
    # ...
```

### 400 (bad request) 视图 ###

**defaults.bad_request(request, template_name='400.html')**

当Django 中引发一个`SuspiciousOperation` 时，它可能通过Django 的一个组件处理（例如重设会话的数据）。如果没有特殊处理，Django 将认为当前的请求时一个'bad request' 而不是一个server error。

`django.views.defaults.bad_request` 和server_error 视图非常相似，除了返回400 状态码来表示错误来自客户端的操作。

`bad_request` 视图同样只是在`DEBUG` 为`False` 时使用。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Built-in Views](https://docs.djangoproject.com/en/1.8/ref/views/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
