# 点击劫持保护 #

点击劫持中间件和装饰器提供了简捷易用的，对[点击劫持](http://en.wikipedia.org/wiki/Clickjacking)的保护。这种攻击在恶意站点诱导用户点击另一个站点的被覆盖元素时出现，另一个站点已经加载到了隐藏的`frame`或`iframe`中。

## 点击劫持的示例 ##

假设一个在线商店拥有一个页面，已登录的用户可以点击“现在购买”来购买一个商品。用户为了方便，可以选择一直保持商店的登录状态。一个攻击者的站点可能在他们自己的页面上会创建一个“我喜欢Ponies”的按钮，并且在一个透明的`iframe`中加载商店的页面，把“现在购买”的按钮隐藏起来覆盖在“我喜欢Ponies”上。如果用户访问了攻击者的站点，点击“我喜欢Ponies”按钮会触发对“现在购买”按钮的无意识的点击，不知不觉中购买了商品。

## 点击劫持的防御 ##

现代浏览器遵循[X-Frame-Options](https://developer.mozilla.org/en/The_X-FRAME-OPTIONS_response_header)协议头，它表明一个资源是否允许加载到`frame`或者`iframe`中。如果响应包含值为`SAMEORIGIN`的协议头，浏览器会在`frame`中只加载同源请求的的资源。如果协议头设置为`DENY`，浏览器会在加载`frame`时屏蔽所有资源，无论请求来自于哪个站点。

Django提供了一些简单的方法来在你站点的响应中包含这个协议头：

+ 一个简单的中间件，在所有响应中设置协议头。
+ 一系列的视图装饰器，可以用于覆盖中间件，或者只用于设置指定视图的协议头。

## 如何使用 ##

### 为所有响应设置X-Frame-Options ###

要为你站点中所有的响应设置相同的`X-Frame-Options`值，将`'django.middleware.clickjacking.XFrameOptionsMiddleware'`设置为 `MIDDLEWARE_CLASSES`：

```
MIDDLEWARE_CLASSES = (
    ...
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    ...
)
```

这个中间件可以在startproject生成的设置文件中开启。

通常，这个中间件会为任何开放的`HttpResponse`设置`X-Frame-Options`协议头为`SAMEORIGIN`。如果你想用 `DENY`来替代它，要设置`X_FRAME_OPTIONS`：

```
X_FRAME_OPTIONS = 'DENY'
```

使用这个中间件时可能会有一些视图，你并不想为它设置`X-Frame-Options`协议头。对于这些情况，你可以使用一个视图装饰器来告诉中间件不要设置协议头：

```
from django.http import HttpResponse
from django.views.decorators.clickjacking import xframe_options_exempt

@xframe_options_exempt
def ok_to_load_in_a_frame(request):
    return HttpResponse("This page is safe to load in a frame on any site.")
```

### 为每个视图设置 X-Frame-Options ###

Django提供了以下装饰器来为每个基础视图设置`X-Frame-Options`协议头。

```
from django.http import HttpResponse
from django.views.decorators.clickjacking import xframe_options_deny
from django.views.decorators.clickjacking import xframe_options_sameorigin

@xframe_options_deny
def view_one(request):
    return HttpResponse("I won't display in any frame!")

@xframe_options_sameorigin
def view_two(request):
    return HttpResponse("Display in a frame if it's from the same origin as me.")
```

注意你可以在中间件的连接中使用装饰器。使用装饰器来覆盖中间件。

## 限制 ##

`X-Frame-Options`协议头只在现代浏览器中保护点击劫持。老式的浏览器会忽视这个协议头，并且需要 [其它点击劫持防范技巧](http://en.wikipedia.org/wiki/Clickjacking#Prevention)。

### 支持 X-Frame-Options 的浏览器 ###

+ Internet Explorer 8+
+ Firefox 3.6.9+
+ Opera 10.5+
+ Safari 4+
+ Chrome 4.1+

### 另见 ###

浏览器对`X-Frame-Options`支持情况的[完整列表](https://developer.mozilla.org/en/The_X-FRAME-OPTIONS_response_header#Browser_compatibility)。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Clickjacking protection](https://docs.djangoproject.com/en/1.8/ref/clickjacking/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
