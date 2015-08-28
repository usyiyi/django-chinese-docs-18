# 基于类的视图 #

视图是一个可调用对象，它接收一个请求然后返回一个响应。这个可调用对象可以不只是函数，Django 提供一些可以用作视图的类。它们允许你结构化你的视图并且利用继承和混合重用代码。后面我们将介绍一些用于简单任务的通用视图，但你可能想要设计自己的可重用视图的结构以适合你的使用场景。完整的细节，请参见基于类的视图的参考文档。

+ [基于类的视图简介](http://python.usyiyi.cn/django/topics/class-based-views/intro.html)
+ [内建的基于类的通用视图](http://python.usyiyi.cn/django/topics/class-based-views/generic-display.html)
+ [使用基于类的视图处理表单](http://python.usyiyi.cn/django/topics/class-based-views/generic-editing.html)
+ [使用混合来扩展视图类](http://python.usyiyi.cn/django/topics/class-based-views/mixins.html)

## 基本的示例 ##

Django 提供基本的视图类，它们适用于广泛的应用。所有的视图类继承自`View`类，它负责连接视图到URL、HTTP 方法调度和其它简单的功能。`RedirectView`用于简单的HTTP 重定向，`TemplateView`扩展基类来渲染模板。

## 在URLconf 中的简单用法 ##

使用通用视图最简单的方法是在URLconf 中创建它们。如果你只是修改基于类的视图的一些简单属性，你可以将它们直接传递给`as_view()`方法调用：

```
from django.conf.urls import url
from django.views.generic import TemplateView

urlpatterns = [
    url(r'^about/', TemplateView.as_view(template_name="about.html")),
]
```

传递给`as_view()`的参数将覆盖类中的属性。在这个例子中，我们设置`TemplateView`的`template_name`。可以使用类似的方法覆盖`RedirectView`的`url`属性。

## 子类化通用视图 ##

第二种，功能更强一点的使用通用视图的方式是继承一个已经存在的视图并在子类中覆盖其属性（例如`template_name`）或方法（例如`get_context_data`）以提供新的值或方法。例如，考虑只显示一个模板`about.html`的视图。Django 有一个通用视图`TemplateView`来做这件事，所以我们可以简单地子类化它，并覆盖模板的名称：

```
# some_app/views.py
from django.views.generic import TemplateView

class AboutView(TemplateView):
    template_name = "about.html"
```

然后我们只需要添加这个新的视图到我们的URLconf 中。`TemplateView`是一个类不是一个函数，所以我们将URL 指向类的`as_view()`方法，它让基于类的视图提供一个类似函数的入口：

```
# urls.py
from django.conf.urls import url
from some_app.views import AboutView

urlpatterns = [
    url(r'^about/', AboutView.as_view()),
]
```

关于如何使用内建的通用视图的更多信息，参考下一主题通用的基于类的视图。

## 支持其它HTTP 方法 ##

假设有人想通过HTTP 访问我们的书库，它使用视图作为API。这个API 客户端将随时连接并下载自上次访问以来新出版的书籍的数据。如果没有新的书籍，仍然从数据库中获取书籍、渲染一个完整的响应并发送给客户端将是对CPU 和带宽的浪费。如果有个API 用于查询书籍最新发布的时间将会更好。

我们在URLconf 中映射URL 到书籍列表视图：

```
from django.conf.urls import url
from books.views import BookListView

urlpatterns = [
    url(r'^books/$', BookListView.as_view()),
]
```

下面是这个视图：

```
from django.http import HttpResponse
from django.views.generic import ListView
from books.models import Book

class BookListView(ListView):
    model = Book

    def head(self, *args, **kwargs):
        last_book = self.get_queryset().latest('publication_date')
        response = HttpResponse('')
        # RFC 1123 date format
        response['Last-Modified'] = last_book.publication_date.strftime('%a, %d %b %Y %H:%M:%S GMT')
        return response
```

如果该视图从GET 请求访问，将在响应中返回一个普通而简单的对象列表（使用`book_list.html`模板）。但如果客户端发出一个`HEAD`请求，响应将具有一个空的响应体而`Last-Modified`头部会指示最新发布的书籍的时间。基于这个信息，客户端可以下载或不下载完整的对象列表。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Overview](https://docs.djangoproject.com/en/1.8/topics/class-based-views/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
