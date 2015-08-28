# Django 的快捷函数 #

`django.shortcuts` 收集了“跨越” 多层MVC 的辅助函数和类。 换句话讲，这些函数/类为了方便，引入了可控的耦合。

## render ##

`render(request, template_name[, context][, context_instance][, content_type][, status][, current_app][, dirs][, using])[source]`

结合一个给定的模板和一个给定的上下文字典，并返回一个渲染后的 `HttpResponse` 对象。

`render()` 与以一个强制使用`RequestContext`的`context_instance` 参数调用`render_to_response()` 相同。

Django 不提供返回`TemplateResponse` 的快捷函数，因为`TemplateResponse` 的构造与`render()` 提供的便利是一个层次的。

### 必选的参数 ###

`request`

用于生成响应的请求对象。

`template_name`

要使用的模板的完整名称或者模板名称的一个序列。

### 可选的参数 ###

`context`

添加到模板上下文的一个字典。默认是一个空字典。如果字典中的某个值是可调用的，视图将在渲染模板之前调用它。

```
Django 1.8 的改变：

context 参数之前叫做dictionary。这个名字在Django 1.8 中废弃并将在Django 2.0 中删除。
```

`context_instance`

渲染模板的上下文实例。默认情况下，模板将使用`RequestContext` 实例（ 值来自`request` 和`context`）渲染。

```
版本 1.8 以后废弃：

废弃context_instance 参数。仅仅使用context。
```

`content_type`

生成的文档要使用的MIME 类型。默认为`DEFAULT_CONTENT_TYPE` 设置的值。

`status`

响应的状态码。默认为200。

`current_app`

指示哪个应用包含当前的视图。更多信息，参见[带命名空间的URL 的解析](http://python.usyiyi.cn/django/topics/http/urls.html#topics-http-reversing-url-namespaces)。

```
版本1.8 以后废弃：

废弃current_app 参数。你应该设置request.current_app。
```

`using`

用于加载模板使用的模板引擎的名称。

```
Changed in Django 1.8:

增加using 参数。
```

```
Changed in Django 1.7:

增加dirs 参数。
```

```
Deprecated since version 1.8:

废弃dirs 参数。
```

## 示例 ##

下面的示例渲染模板`myapp/index.html`，MIME 类型为`application/xhtml+xml`：

```
from django.shortcuts import render

def my_view(request):
    # View code here...
    return render(request, 'myapp/index.html', {"foo": "bar"},
        content_type="application/xhtml+xml")
```

这个示例等同于：

```
from django.http import HttpResponse
from django.template import RequestContext, loader

def my_view(request):
    # View code here...
    t = loader.get_template('myapp/index.html')
    c = RequestContext(request, {'foo': 'bar'})
    return HttpResponse(t.render(c),
        content_type="application/xhtml+xml")
```

## render_to_response ##

`render_to_response(template_name[, context][, context_instance][, content_type][, status][, dirs][, using])[source]`

根据一个给定的上下文字典渲染一个给定的目标，并返回渲染后的HttpResponse。

### 必选的参数 ###

`template_name`

使用的模板的完整名称或者模板名称的序列。如果给出的是一个序列，将使用存在的第一个模板。关于如何查找模板的更多信息请参见 模板加载的文档 。

### 可选的参数 ###

`context`

添加到模板上下文中的字典。默认是个空字典。如果字典中的某个值是可调用的，视图将在渲染模板之前调用它。

```
Changed in Django 1.8:

context 参数之前叫做dictionary。 这个名字在Django 1.8 中废弃并将在Django 2.0 中删除。
```

`context_instance`

渲染模板使用的上下文实例。默认情况下，模板将`Context` 实例（值来自`context`）渲染。如果你需要使用上下文处理器，请使用`RequestContext` 实例渲染模板。你的代码看上去像是这样：

```
return render_to_response('my_template.html',
                          my_context,
                          context_instance=RequestContext(request))
```

```
版本1.8 以后废弃：

废弃context_instance 参数。 仅仅使用context。
```

`content_type`

生成的文档使用的MIME 类型。默认为`DEFAULT_CONTENT_TYPE` 设置的值。

`status`

相应的状态码。默认为200。

`using`

加载模板使用的模板引擎的名称。

```
Changed in Django 1.8:

添加status 和using 参数。
```

```
Changed in Django 1.7:

增加dirs 参数。
```

```
Deprecated since version 1.8:

废弃dirs 参数。
```

## 示例 ##

下面的示例渲染模板`myapp/index.html`，MIIME 类型为`application/xhtml+xml`：

```
from django.shortcuts import render_to_response

def my_view(request):
    # View code here...
    return render_to_response('myapp/index.html', {"foo": "bar"},
        content_type="application/xhtml+xml")
```

这个示例等同于：

```
from django.http import HttpResponse
from django.template import Context, loader

def my_view(request):
    # View code here...
    t = loader.get_template('myapp/index.html')
    c = Context({'foo': 'bar'})
    return HttpResponse(t.render(c),
        content_type="application/xhtml+xml")
```

## redirect ##

`redirect(to, [permanent=False, ]*args, **kwargs)[source]`

为传递进来的参数返回HttpResponseRedirect 给正确的URL 。

参数可以是：

+ 一个模型：将调用模型的`get_absolute_url()` 函数
+ 一个视图，可以带有参数：将使用`urlresolvers.reverse` 来反向解析名称
+ 一个绝对的或相对的URL，将原样作为重定向的位置。

默认返回一个临时的重定向；传递`permanent=True `可以返回一个永久的重定向。

```
Django 1.7 中的改变：

增加使用相对URL 的功能。
```

## 示例 ##

你可以用多种方式使用`redirect()` 函数。

通过传递一个对象；将调用`get_absolute_url()` 方法来获取重定向的URL：

```
from django.shortcuts import redirect

def my_view(request):
    ...
    object = MyModel.objects.get(...)
    return redirect(object)
```

通过传递一个视图的名称，可以带有位置参数和关键字参数；将使用`reverse()` 方法反向解析URL：

```
def my_view(request):
    ...
    return redirect('some-view-name', foo='bar')
```

传递要重定向的一个硬编码的URL：

```
def my_view(request):
    ...
    return redirect('/some/url/')
```

完整的URL 也可以：

```
def my_view(request):
    ...
    return redirect('http://example.com/')
```

默认情况下，`redirect()` 返回一个临时重定向。以上所有的形式都接收一个`permanent` 参数；如果设置为`True`，将返回一个永久的重定向：

```
def my_view(request):
    ...
    object = MyModel.objects.get(...)
    return redirect(object, permanent=True)
```

## get_object_or_404 ##

`get_object_or_404(klass, *args, **kwargs)[source]`

在一个给定的模型管理器上调用`get()`，但是引发`Http404` 而不是模型的`DoesNotExist` 异常。

### 必选的参数 ###

`klass`

获取该对象的一个`Model` 类，`Manager`或`QuerySet` 实例。

`**kwargs`

查询的参数，格式应该可以被`get()` 和`filter()`接受。

## 示例 ##

下面的示例从`MyModel` 中使用主键1 来获取对象：

```
from django.shortcuts import get_object_or_404

def my_view(request):
    my_object = get_object_or_404(MyModel, pk=1)
```

这个示例等同于：

```
from django.http import Http404

def my_view(request):
    try:
        my_object = MyModel.objects.get(pk=1)
    except MyModel.DoesNotExist:
        raise Http404("No MyModel matches the given query.")
```

最常见的用法是传递一个 `Model`，如上所示。然而，你还可以传递一个`QuerySet `实例：

```
queryset = Book.objects.filter(title__startswith='M')
get_object_or_404(queryset, pk=1)
```

上面的示例有点做作，因为它等同于：

```
get_object_or_404(Book, title__startswith='M', pk=1)
```

但是如果你的queryset 来自其它地方，它就会很有用了。

最后你还可以使用一个管理器。如果你有一个自定义的管理器，它将很有用：

```
get_object_or_404(Book.dahl_objects, title='Matilda')
```

你还可以使用关联的 管理器：

```
author = Author.objects.get(name='Roald Dahl')
get_object_or_404(author.book_set, title='Matilda')
```

注：与get()一样，如果找到多个对象将引发一个`MultipleObjectsReturned` 异常。

## get_list_or_404 ##

`get_list_or_404(klass, *args, **kwargs)[source]`

返回一个给定模型管理器上filter() 的结果，并将结果映射为一个列表，如果结果为空则返回Http404。

## 必选的参数 ##

`klass`

获取该列表的一个`Model`、`Manager` 或`QuerySet` 实例。

`**kwargs`

查寻的参数，格式应该可以被`get()` 和`filter()` 接受。

示例

下面的示例从`MyModel` 中获取所有发布出来的对象：

```
from django.shortcuts import get_list_or_404

def my_view(request):
    my_objects = get_list_or_404(MyModel, published=True)
```

这个示例等同于：

```
from django.http import Http404

def my_view(request):
    my_objects = list(MyModel.objects.filter(published=True))
    if not my_objects:
        raise Http404("No MyModel matches the given query.")
```

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Shortcuts](https://docs.djangoproject.com/en/1.8/topics/http/shortcuts/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
