<!--
  译者：Github@wizardforcel
-->

# 中间件 #

中间件是一个介入Django的请求和响应的处理过程中的钩子框架。它是一个轻量级，底层的“插件”系统，用于在全局修改Django的输入或输出。

中间件组件责任处理某些特殊的功能。例如，Django包含一个中间件组件，AuthenticationMiddleware ，使用会话将用户和请求关联。

这篇文档讲解了中间件如何工作，如何激活中间件，以及如何编写自己的中间件。Django集成了一些内置的中间件可以直接开箱即用。它们被归档在 内置中间件参考.

## 激活中间件 ##

要激活一个中间件组件，需要把它添加到你Django配置文件中的MIDDLEWARE_CLASSES 列表中。

在MIDDLEWARE_CLASSES中，每一个中间件组件用字符串的方式描述：一个完整的Python全路径加上中间件的类名称。例如，使用 django-admin startproject创建工程的时候生成的默认值：

```
MIDDLEWARE_CLASSES = (
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.auth.middleware.SessionAuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'django.middleware.security.SecurityMiddleware',
)
```

Django的程序中，中间件不是必需的 —  只要你喜欢，MIDDLEWARE_CLASSES可以为空 — 但是强烈推荐你至少使用CommonMiddleware。

MIDDLEWARE_CLASSES中的顺序非常重要，因为一个中间件可能依赖于另外一个。例如，AuthenticationMiddleware在会话中储存已认证的用户。所以它必须在SessionMiddleware之后运行。一些关于Django中间件类的顺序的常见提示，请见Middleware ordering。

## 钩子和应用顺序 ##

在请求阶段中，调用视图之前，Django会按照MIDDLEWARE_CLASSES中定义的顺序自顶向下应用中间件。会用到两个钩子：

+ process_request()
+ process_view()

在响应阶段中，调用视图之后，中间件会按照相反的顺序应用，自底向上。会用到三个钩子：

+ process_exception() （仅当视图抛出异常的时候）
+ process_template_response() （仅用于模板响应）
+ process_response()

![](https://docs.djangoproject.com/en/1.8/_images/middleware.svg)

如果你愿意的话，你可以把它想象成一颗洋葱：每个中间件都是包裹视图的一层“皮”。

每个钩子的行为接下来会描述。

## 编写自己的中间件 ##

编写自己的中间件很容易的。每个中间件组件是一个单独的Python的class，你可以定一个或多个下面的这些方法：

### process_request ###

**process_request(request)**

request是一个HttpRequest 对象。

在Django决定执行哪个视图(view)之前，process_request()会被每次请求调用。

它应该返回一个None 或一个HttpResponse对象。如果返回 None, Django会继续处理这个请求，执行其他process_request()中间件，然后process_view()中间件显示对应的视图。如果它返回一个HttpResponse对象，Django便不再会去调用其他的请求(request), 视图(view)或其他中间件，或对应的视图；处理HttpResponse的中间件会处理任何返回的响应(response)。

### process_view ###

**process_view(request, view_func, view_args, view_kwargs)**

request是一个HttpRequest对象。view_func是 Django会调用的一个Python的函数。(它确实是一个函数对象，不是函数的字符名称。) view_args是一个会被传递到视图的位置参数列表，而view_kwargs 是一个会被传递到视图的关键字参数字典。 view_args和 view_kwargs 都不包括第一个视图参数(request)。

process_view()会在Django调用视图(view)之前被调用。

它将返回None 或一个HttpResponse 对象。如果返回 None，将会继续处理这个请求，执行其他的process_view() 中间件，然后显示对应的视图。如果返回HttpResponse对象，Django就不再会去调用其他的视图（view），异常中间件（exception middleware）或对应的视图 ；它会把响应中间件应用到HttpResponse上，并返回结果。

> 注意
> 
> 在中间件内部，从process_request或者process_view方法中访问request.POST或者request.REQUEST将会阻碍该中间 件之后的所有视图无法修改request的上传处理程序，  一般情况要避免这样使用。

> 类CsrfViewMiddleware可以被认为是个例外     ，因为它提供了csrf_exempt() 和 csrf_protect()两个允许视图来精确控制     在哪个点需要开启CSRF验证。

### process_template_response ###

**process_template_response(request, response)**

request是一个HttpRequest对象。response是一个TemplateResponse对象（或等价的对象），由Django视图或者中间件返回。

如果响应的实例有render()方法，process_template_response()在视图刚好执行完毕之后被调用，这表明了它是一个TemplateResponse对象（或等价的对象）。

这个方法必须返回一个实现了render方法的响应对象。它可以修改给定的response对象，通过修改 response.template_name和response.context_data或者它可以创建一个全新的 TemplateResponse或等价的对象。

你不需要显式渲染响应 —— 一旦所有的模板响应中间件被调用，响应会自动被渲染。

在一个响应的处理期间，中间件以相反的顺序运行，这包括process_template_response()。

### process_response ###

**process_response(request, response)**

request是一个HttpRequest对象。response是Django视图或者中间件返回的HttpResponse或者StreamingHttpResponse对象。

process_response()在所有响应返回浏览器之前被调用。

这个方法必须返回HttpResponse或者StreamingHttpResponse对象。它可以改变已有的response，或者创建并返回新的HttpResponse或StreamingHttpResponse对象。

不像 process_request()和process_view()方法，即使同一个中间件类中的process_request()和process_view()方法会因为前面的一个中间件返回HttpResponse而被跳过，process_response()方法总是会被调用。特别是，这意味着你的process_response()方法不能依赖于process_request()方法中的设置。

最后，记住在响应阶段中，中间件以相反的顺序被应用，自底向上。意思是定义在MIDDLEWARE_CLASSES最底下的类会最先被运行。

### 处理流式响应 ###

不像HttpResponse，StreamingHttpResponse并没有content属性。所以，中间件再也不能假设所有响应都带有content属性。如果它们需要访问内容，他们必须测试是否为流式响应，并相应地调整自己的行为。

```
if response.streaming:
    response.streaming_content = wrap_streaming_content(response.streaming_content)
else:
    response.content = alter_content(response.content)
```

> 注意
> 
> 我们需要假设streaming_content可能会大到在内存中无法容纳。响应中间件可能会把它封装在新的生成器中，但是一定不要销毁它。封装一般会实现成这样：
> 
```
def wrap_streaming_content(content):
    for chunk in content:
        yield alter_content(chunk)
```
> 

### process_exception ###

**process_exception(request, exception)**

request是一个HttpRequest对象。exception是一个被视图中的方法抛出来的 Exception对象。

当一个视图抛出异常时，Django会调用process_exception()来处理。process_exception()应该返回一个None 或者一个HttpResponse对象。如果它返回一个HttpResponse对象，模型响应和响应中间件会被应用，响应结果会返回给浏览器。Otherwise, default exception handling kicks in.

再次提醒，在处理响应期间，中间件的执行顺序是倒序执行的，这包括process_exception。如果一个异常处理的中间件返回了一个响应，那这个中间件上面的中间件都将不会被调用。

### \_\_init\_\_ ###

大多数的中间件类都不需要一个初始化方法，因为中间件的类定义仅仅是为process\_\*提供一个占位符。如果你确实需要一个全局的状态那就可以通过\_\_init\_\_来加载。然后要铭记如下两个警告：

Django初始化你的中间件无需任何参数，因此不要定义一个有参数的\_\_init\_\_方法。
不像process\_\*每次请求到达都要调用\_\_init\_\_只会被调用一次，就是在Web服务启动的时候。


### 标记中间件不被使用 ###

有时在运行时决定是否一个中间件需要被加载是很有用的。 在这种情况下，你的中间件中的 \_\_init\_\_方法可以抛出一个django.core.exceptions.MiddlewareNotUsed异常。Django会从中间件处理过程中移除这部分中间件，并且当DEBUG为True的时候在django.request记录器中记录调试信息。

```
1.8中的修改：

之前 MiddlewareNotUsed异常不会被记录。
```

## 指导准则 ##

+ 中间件的类不能是任何类的子类。
+ 中间件可以存在与你Python路径中的任何位置。 Django所关心的只是被包含在MIDDLEWARE_CLASSES中的配置。
+ 将Django’s available middleware作为例子随便看看。
+ 如果你认为你写的中间件组建可能会对其他人有用，那就把它共享到社区！ 让我们知道它，我们会考虑把它添加到Django中。