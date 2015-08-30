# TemplateResponse 和 SimpleTemplateResponse #

标准的`HttpResponse` 对象是静态的结构。在构造的时候提供给它们一个渲染之前的内容，但是当内容改变时它们却不能很容易地完成相应的改变。

然而，有时候允许装饰器或者中间件在响应被构造之后修改它是很有用的。例如，你可能想改变使用的模板，或者添加额外的数据到上下文中。

`TemplateResponse` 提供了实现这一点的方法。与基本的HttpResponse 对象不同，`TemplateResponse` 对象会记住视图提供的模板和上下文的详细信息来计算响应。响应的最终结果在后来的响应处理过程中直到需要时才计算。

## SimpleTemplateResponse 对象 ##

`class SimpleTemplateResponse[source]`

### 属性 ###

`SimpleTemplateResponse.template_name`

渲染的模板的名称。接收一个与后端有关的模板对象（例如`get_template()` 返回的对象）、模板的名称或者一个模板名称列表。

例如：`['foo.html', 'path/to/bar.html']`

```
Deprecated since version 1.8:

template_name 以前只接受一个 Template。
```

`SimpleTemplateResponse.context_data`

渲染模板时用到的上下文数据。它必须是一个`dict`。

例如：`{'foo': 123}`

```
Deprecated since version 1.8:

context_data 以前只接受一个 Context。
```

`SimpleTemplateResponse.rendered_content[source]`

使用当前的模板和上下文数据渲染出来的响应内容。

`SimpleTemplateResponse.is_rendered[source]`

一个布尔值，表示响应内容是否已经渲染。

### 方法 ###

`SimpleTemplateResponse.__init__(template, context=None, content_type=None, status=None, charset=None, using=None)[source]`

使用给定的模板、上下文、Content-Type、HTTP 状态和字符集初始化一个`SimpleTemplateResponse` 对象。

`template`

一个与后端有关的模板对象（例如`get_template()` 返回的对象）、模板的名称或者一个模板名称列表。

```
Deprecated since version 1.8:

template 以前只接受一个Template。
```

`context`

一个`dict`，包含要添加到模板上下文中的值。它默认是一个空的字典。

```
Deprecated since version 1.8:

context 以前只接受一个Context。
```

`content_type`

HTTP `Content-Type `头部包含的值，包含MIME 类型和字符集的编码。 如果指定`content_type`，则使用它的值。 否则，使用`DEFAULT_CONTENT_TYPE`。

`status`

响应的HTTP 状态码。

`charset`

响应编码使用的字符集。 如果没有给出则从`content_type`中提取，如果提取不成功则使用 `DEFAULT_CHARSET` 设置。

`using`

加载模板使用的模板引擎的名称。

```
Changed in Django 1.8:

添加charset 和using 参数。
```

`SimpleTemplateResponse.resolve_context(context)[source]`

预处理即将用于渲染模板的上下文数据。接受包含上下文数据的一个`dict`。默认返回同一个`dict`。

若要自定义上下文，请覆盖这个方法。

```
Changed in Django 1.8:

resolve_context 返回一个dict。它以前返回一个Context。
```

```
Deprecated since version 1.8:

resolve_context 不再接受Context。
```

`SimpleTemplateResponse.resolve_template(template)[source]`

解析渲染用到的模板实例。接收一个与后端有关的模板对象（例如`get_template()` 返回的对象）、模板的名称或者一个模板名称列表。

返回将被渲染的模板对象。

若要自定义模板的加载，请覆盖这个方法。

```
Changed in Django 1.8:

resolve_template 返回一个与后端有关的模板对象。它以前返回一个Template。
```

```
Deprecated since version 1.8:

resolve_template 不再接受一个 Template。
```

`SimpleTemplateResponse.add_post_render_callback()[source]`

添加一个渲染之后调用的回调函数。这个钩子可以用来延迟某些特定的处理操作（例如缓存）到渲染之后。

如果`SimpleTemplateResponse` 已经渲染，那么回调函数将立即执行。

调用的时只传递给回调函数一个参数 —— 渲染后的 `SimpleTemplateResponse` 实例。

如果回调函数返回非`None` 值，它将用作响应并替换原始的响应对象（以及传递给下一个渲染之后的回调函数，以此类推）。

`SimpleTemplateResponse.render()[source]`

设置`response.content` 为`SimpleTemplateResponse.rendered_content` 的结果，执行所有渲染之后的回调函数，最后返回生成的响应对象。

`render()` 只在第一次调用它时其作用。以后的调用将返回第一次调用的结果。

## TemplateResponse 对象 ##

`class TemplateResponse[source]`

`TemplateResponse` 是`SimpleTemplateResponse` 的子类，而且能知道当前的`HttpRequest`。

### 方法 ###

`TemplateResponse.__init__(request, template, context=None, content_type=None, status=None, current_app=None, charset=None, using=None)[source]`

使用给定的模板、上下文、`Content-Type`、HTTP 状态和字符集实例化一个`TemplateResponse` 对象。

`request`

An `HttpRequest` instance.

`template`

一个与后端有关的模板对象（例如`get_template()` 返回的对象）、模板的名称或者一个模板名称列表。

```
Deprecated since version 1.8:

template 以前只接受一个Template。
```

`context`

一个`dict`，包含要添加到模板上下文中的值。 它默认是一个空的字典。

```
Deprecated since version 1.8:

context 以前只接受一个Context。
```

`content_type`

HTTP `Content-Type` 头部包含的值，包含MIME 类型和字符集的编码。如果指定`content_type`，则使用它的值。否则，使用`DEFAULT_CONTENT_TYPE`。

`status`

响应的HTTP 状态码。

`current_app`

包含当前视图的应用。更多信息，参见带命名空间的URL 解析策略。

```
Deprecated since version 1.8:

废弃current_app 参数。你应该去设置request.current_app。
```

`charset`

响应编码使用的字符集。如果没有给出则从content_type中提取，如果提取不成功则使用 DEFAULT_CHARSET 设置。

`using`

加载模板使用的模板引擎的名称。

```
Changed in Django 1.8:

添加charset 和using 参数。
```

## 渲染的过程 ##

在`TemplateResponse` 实例返回给客户端之前，它必须被渲染。渲染的过程采用模板和上下文变量的中间表示形式，并最终将它转换为可以发送给客户端的字节流。

三种情况下会渲染`TemplateResponse`：

+ 当使用SimpleTemplateResponse.render() 方法显示渲染TemplateResponse 实例的时候。
+ 当通过给response.content 赋值显式设置响应内容的时候。
+ 在应用模板响应中间件之后，应用响应中间件之前。

`TemplateResponse` 只能渲染一次。`SimpleTemplateResponse.render()` 的第一次调用设置响应的内容；以后的响应不会改变响应的内容。

然而，当显式给`response.content` 赋值时，修改会始终生效。如果你想强制重新渲染内容，你可以重新计算渲染的内容并手工赋值给响应的内容：

```
# Set up a rendered TemplateResponse
>>> from django.template.response import TemplateResponse
>>> t = TemplateResponse(request, 'original.html', {})
>>> t.render()
>>> print(t.content)
Original content

# Re-rendering doesn't change content
>>> t.template_name = 'new.html'
>>> t.render()
>>> print(t.content)
Original content

# Assigning content does change, no render() call required
>>> t.content = t.rendered_content
>>> print(t.content)
New content
```

### 渲染后的回调函数 ###

某些操作 —— 例如缓存 —— 不可以在没有渲染的模板上执行。它们必须在完整的渲染后的模板上执行。

如果你正在使用中间件，解决办法很容易。中间件提供多种在从视图退出时处理响应的机会。如果你向响应中间件添加一些行为，它们将保证在模板渲染之后执行。

然而，如果正在使用装饰器，就不会有这样的机会。装饰器中定义的行为会立即执行。

为了补偿这一点（以及其它类似的使用情形）`TemplateResponse` 允许你注册在渲染完成时调用的回调函数。使用这个回调函数，你可以延迟某些关键的处理直到你可以保证渲染后的内容是可以访问的。

要定义渲染后的回调函数，只需定义一个接收一个响应作为参数的函数并将这个函数注册到模板响应中：

```
from django.template.response import TemplateResponse

def my_render_callback(response):
    # Do content-sensitive processing
    do_post_processing()

def my_view(request):
    # Create a response
    response = TemplateResponse(request, 'mytemplate.html', {})
    # Register the callback
    response.add_post_render_callback(my_render_callback)
    # Return the response
    return response
```

`my_render_callback()` 将在`mytemplate.html` 渲染之后调用，并将被传递一个`TemplateResponse` 实例作为参数。

如果模板已经渲染，回调函数将立即执行。

##  使用TemplateResponse 和SimpleTemplateResponse ##

`TemplateResponse` 对象和普通的`django.http.HttpResponse` 一样可以用于任何地方。它可以用来作为`render()` 和`render_to_response()` 的另外一种选择。

例如，下面这个简单的视图使用一个简单模板和包含查询集的上下文返回一个`TemplateResponse`：

```
from django.template.response import TemplateResponse

def blog_index(request):
    return TemplateResponse(request, 'entry_list.html', {'entries': Entry.objects.all()})
```

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[TemplateResponse objects](https://docs.djangoproject.com/en/1.8/ref/template-response/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
