# 编写视图 #

一个视图函数，或者简短来说叫做视图，是一个简单的Python函数，它接受web请求，并且返回web响应。响应可以是一张网页的HTML内容，一个重定向，一个404错误，一个XML文档，或者一张图片. . . 是任何东西都可以。无论视图本身包含什么逻辑，都要返回响应。代码写在哪里也无所谓，只要它在你的Python目录下面。除此之外没有更多的要求了——可以说“没有什么神奇的地方”。为了能够把代码放在某个地方，惯例是把视图放在叫做views.py的文件中，然后把它放到你的项目或者应用目录里。

## 一个简单的视图 ##

下面是一个返回当前日期和时间作为HTML文档的视图：

```
from django.http import HttpResponse
import datetime

def current_datetime(request):
    now = datetime.datetime.now()
    html = "<html><body>It is now %s.</body></html>" % now
    return HttpResponse(html)
```

让我们逐行阅读上面的代码：

+ 首先，我们从 django.http模块导入了HttpResponse类，以及Python的datetime库。
+ 接着，我们定义了current_datetime函数。它是一个视图函数。每个视图函数都应接收HttpRequest对象作为第一个参数，一般叫做request。
+ 注意视图函数的名称并不重要；不需要用一个统一的命名方式来命名，以便让Django识别它。我们将其命名为current_datetime，是因为这个名称能够精确地反映出它的功能。
+ 这个视图会返回一个HttpResponse对象，其中包含生成的响应。每个视图函数都要返回HttpResponse对象。（有例外，我们接下来会讲。）

> Django中的时区
> 
> Django中包含一个TIME_ZONE设置，默认为America/Chicago。可能并不是你住的地方，所以你可能会在设置文件里修改它。

## 把你的URL映射到视图 ##

所以，再重复一遍，这个视图函数返回了一个包含当前日期和时间的HTML页面。你需要创建URLconf来展示在特定的URL这一视图； 详见URL 分发器。

## 返回错误 ##

在Django中返回HTTP错误是相当容易的。有一些HttpResponse的子类代表不是200（“OK”）的HTTP状态码。你可以在request/response文档中找到所有可用的子类。你可以返回那些子类的一个实例，而不是普通的HttpResponse ，来表示一个错误。例如：

```
from django.http import HttpResponse, HttpResponseNotFound

def my_view(request):
    # ...
    if foo:
        return HttpResponseNotFound('<h1>Page not found</h1>')
    else:
        return HttpResponse('<h1>Page was found</h1>')
```

由于一些状态码不太常用，所以不是每个状态码都有一个特化的子类。然而，如HttpResponse文档中所说的那样，你也可以向HttpResponse的构造器传递HTTP状态码，来创建你想要的任何状态码的返回类。例如：

```
from django.http import HttpResponse

def my_view(request):
    # ...

    # Return a "created" (201) response code.
    return HttpResponse(status=201)
```

由于404错误是最常见的HTTP错误，所以处理这一错误的方式非常便利。

## Http404异常 ##

**class django.http.Http404**

当你返回一个像HttpResponseNotFound这样的错误时，它会输出这个错误页面的HTML作为结果：

```
return HttpResponseNotFound('<h1>Page not found</h1>')
```

为了便利起见，也因为你的站点有个一致的404页面是个好主意，Django提供了Http404异常。如果你在视图函数中的任何地方抛出Http404异常，Django都会捕获它，并且带上HTTP404错误码返回你应用的标准错误页面。

像这样：

```
from django.http import Http404
from django.shortcuts import render_to_response
from polls.models import Poll

def detail(request, poll_id):
    try:
        p = Poll.objects.get(pk=poll_id)
    except Poll.DoesNotExist:
        raise Http404("Poll does not exist")
    return render_to_response('polls/detail.html', {'poll': p})
```

为了尽可能利用 Http404，你应该创建一个用来在404错误产生时展示的模板。这个模板应该叫做404.html，并且在你的模板树中位于最顶层。

如果你在抛出Http404异常时提供了一条消息，当DEBUG为True时它会出现在标准404模板的展示中。你可以将这些消息用于调试；但他们通常不适用于404模板本身。

## 自定义错误视图 ##

Django中默认的错误视图对于大多数web应用已经足够了，但是如果你需要任何自定义行为，重写它很容易。只要在你的URLconf中指定下面的处理器（在其他任何地方设置它们不会有效）。

handler404覆盖了page_not_found()视图：

```
handler404 = 'mysite.views.my_custom_page_not_found_view'
```

handler500覆盖了server_error()视图：

```
handler500 = 'mysite.views.my_custom_error_view'
```

handler403覆盖了permission_denied()视图：

```
handler403 = 'mysite.views.my_custom_permission_denied_view'
```

handler400覆盖了bad_request()视图：

```
handler400 = 'mysite.views.my_custom_bad_request_view'
```