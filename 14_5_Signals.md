# 信号

Django包含一个“信号的分发器”，允许解耦的应用在信号出现在框架的任何地方时，都能获得通知。简单来说，信号允许指定的&nbsp;_发送器_通知一系列的_接收器_，一些操作已经发生了。当一些代码会相同事件感兴趣时，会十分有帮助。

Django 提供了[_一系列的内建信号_](../ref/signals.html)，允许用户的代码获得DJango的特定操作的通知。这包含一些有用的通知：

*   [`django.db.models.signals.pre_save`](../ref/signals.html#django.db.models.signals.pre_save "django.db.models.signals.pre_save") & [`django.db.models.signals.post_save`](../ref/signals.html#django.db.models.signals.post_save "django.db.models.signals.post_save")

    在模型&nbsp;[`save()`](../ref/models/instances.html#django.db.models.Model.save "django.db.models.Model.save")方法调用之前或之后发送。

*   [`django.db.models.signals.pre_delete`](../ref/signals.html#django.db.models.signals.pre_delete "django.db.models.signals.pre_delete") & [`django.db.models.signals.post_delete`](../ref/signals.html#django.db.models.signals.post_delete "django.db.models.signals.post_delete")

    在模型[`delete()`](../ref/models/instances.html#django.db.models.Model.delete "django.db.models.Model.delete")方法或查询集的[`delete()`](../ref/models/querysets.html#django.db.models.query.QuerySet.delete "django.db.models.query.QuerySet.delete") 方法调用之前或之后发送。

*   [`django.db.models.signals.m2m_changed`](../ref/signals.html#django.db.models.signals.m2m_changed "django.db.models.signals.m2m_changed")

    模型上的 [`ManyToManyField`](../ref/models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 修改时发送。

*   [`django.core.signals.request_started`](../ref/signals.html#django.core.signals.request_started "django.core.signals.request_started") & [`django.core.signals.request_finished`](../ref/signals.html#django.core.signals.request_finished "django.core.signals.request_finished")

    Django建立或关闭HTTP 请求时发送。

关于完整列表以及每个信号的完整解释，请见[_内建信号的文档_](../ref/signals.html) 。

你也可以[定义和发送你自己的自定义信号](#defining-and-sending-signals)；见下文。

## 监听信号

你需要注册一个_接收器_函数来接受信号，它在信号使用[`Signal.connect()`](#django.dispatch.Signal.connect "django.dispatch.Signal.connect")发送时被调用：

`Signal.``connect`(_receiver_[, _sender=None_, _weak=True_, _dispatch_uid=None_])

<table class="docutils field-list" frame="void" rules="none"><colgroup><col class="field-name"><col class="field-body"></colgroup><tbody valign="top"><tr class="field-odd field"><th class="field-name">Parameters:</th><td class="field-body">*   **receiver** – 和这个信号连接的回调函数。详见[_接收器函数_](#receiver-functions)。
*   **sender** – 指定一个特定的发送器，来从它那里接受信号。详见[_连接由指定发送器发送的信号_](#connecting-to-specific-signals)。
*   **weak** – DJango通常以弱引用储存信号处理器。这就是说，如果你的接收器是个局部变量，可能会被垃圾回收。当你调用信号的`connect()`方法是，传递&nbsp;`weak=False`来防止这样做。
*   **dispatch_uid** – 一个信号接收器的唯一标识符，以防信号多次发送。详见[_防止重复的信号_](#preventing-duplicate-signals)。

</td></tr></tbody></table>

让我们来看一看它如何通过注册在每次在HTTP请求结束时调用的信号来工作。我们将会连接到[`request_finished`](../ref/signals.html#django.core.signals.request_finished "django.core.signals.request_finished") 信号。

### 接收器函数

首先，我们需要定义接收器函数。接受器可以是Python函数或者方法：

```
def my_callback(sender, **kwargs):
    print("Request finished!")

```

注意函数接受`sender`函数，以及通配符关键字参数(`**kwargs`)；所有信号处理器都必须接受这些参数。

我们[过一会儿](#connecting-to-signals-sent-by-specific-senders)再关注发送器，现在先看一看`**kwargs`参数。所有信号都发送关键字参数，并且可以在任何时候修改这些关键字参数。在&nbsp;[`request_finished`](../ref/signals.html#django.core.signals.request_finished "django.core.signals.request_finished")的情况下，它被记录为不发送任何参数，这意味着我们可能需要像`my_callback(sender)`一样编写我们自己的信号处理器。

这是错误的 -- 实际上，如果你这么做了，Django会抛出异常。这是因为无论什么时候信号中添加了参数，你的接收器都必须能够处理这些新的参数。

### 连接接收器函数

有两种方法可以将一个接收器连接到信号。你可以采用手动连接的方法：

```
from django.core.signals import request_finished

request_finished.connect(my_callback)

```

或者使用[`receiver()`](#django.dispatch.receiver "django.dispatch.receiver") 装饰器来自动连接：

`receiver`(_signal_)

<table class="docutils field-list" frame="void" rules="none"><colgroup><col class="field-name"><col class="field-body"></colgroup><tbody valign="top"><tr class="field-odd field"><th class="field-name">Parameters:</th><td class="field-body">**signal** – A signal or a list of signals to connect a function to.</td></tr></tbody></table>

下面是使用装饰器连接的方法：

```
from django.core.signals import request_finished
from django.dispatch import receiver

@receiver(request_finished)
def my_callback(sender, **kwargs):
    print("Request finished!")

```

现在，我们的`my_callback`函数会在每次请求结束时调用。

这段代码应该放在哪里？

严格来说，信号处理和注册的代码应该放在你想要的任何地方，但是推荐避免放在应用的根模块和`models`模块中，以尽量减少产生导入代码的副作用。

实际上，信号处理通常定义在应用相关的`signals`子模块中。信号接收器在你应用配置类中的[`ready()`](../ref/applications.html#django.apps.AppConfig.ready "django.apps.AppConfig.ready") 方法中连接。如果你使用；额&nbsp;[`receiver()`](#django.dispatch.receiver "django.dispatch.receiver")装饰器，只是在[`ready()`](../ref/applications.html#django.apps.AppConfig.ready "django.apps.AppConfig.ready")内部导入`signals`子模块就可以了。

Changed in Django 1.7:

由于[`ready()`](../ref/applications.html#django.apps.AppConfig.ready "django.apps.AppConfig.ready")并不在Django之前版本中存在，信号的注册通常在`models`模块中进行。

注意

[`ready()`](../ref/applications.html#django.apps.AppConfig.ready "django.apps.AppConfig.ready") 方法会在测试期间执行多次，所以你可能想要[_防止重复的信号_](#preventing-duplicate-signals)，尤其是打算在测试中发送它们的情况。

### 连接由指定发送器发送的信号

一些信号会发送多次，但是你只想接收这些信号的一个确定的子集。例如，考虑 [`django.db.models.signals.pre_save`](../ref/signals.html#django.db.models.signals.pre_save "django.db.models.signals.pre_save") 信号，它在模型保存之前发送。大多数情况下，你并不需要知道&nbsp;_任何_模型何时保存 -- 只需要知道一个_特定的_模型何时保存。

在这些情况下，你可以通过注册来接收只由特定发送器发出的信号。对于[`django.db.models.signals.pre_save`](../ref/signals.html#django.db.models.signals.pre_save "django.db.models.signals.pre_save")的情况， 发送者是被保存的模型类，所以你可以认为你只需要由某些模型发出的信号：

```
from django.db.models.signals import pre_save
from django.dispatch import receiver
from myapp.models import MyModel

@receiver(pre_save, sender=MyModel)
def my_handler(sender, **kwargs):
    ...

```

`my_handler`函数只在`MyModel`实例保存时被调用。

不同的信号使用不同的对象作为他们的发送器；对于每个特定信号的细节，你需要查看[_内建信号的文档_](../ref/signals.html)。

### 防止重复的信号

在一些情况下，向接收者发送信号的代码可能会执行多次。这会使你的接收器函数被注册多次，并且导致它对于同一信号事件被调用多次。

如果这样的行为会导致问题（例如在任何时候模型保存时使用信号来发送邮件），传递一个唯一的标识符作为&nbsp;`dispatch_uid`参数来标识你的接收器函数。标识符通常是一个字符串，虽然任何可计算哈希的对象都可以。最后的结果是，对于每个唯一的`dispatch_uid`值，你的接收器函数都只被信号调用一次：

```
from django.core.signals import request_finished

request_finished.connect(my_callback, dispatch_uid="my_unique_identifier")

```

## 定义和发送信号

你的应用可以利用信号功能来提供自己的信号。

### 定义信号

_class _`Signal`([_providing_args=list_])

所有信号都是&nbsp;[`django.dispatch.Signal`](#django.dispatch.Signal "django.dispatch.Signal") 的实例。`providing_args`是一个列表，包含参数的名字，它们由信号提供给监听者。理论上是这样，但是实际上并没有任何检查来保证向监听者提供了这些参数。

例如：

```
import django.dispatch

pizza_done = django.dispatch.Signal(providing_args=["toppings", "size"])

```

这段代码声明了`pizza_done`信号，它向接受者提供`toppings`和&nbsp;`size` 参数。

要记住你可以在任何时候修改参数的列表，所以首次尝试的时候不需要完全确定API。

### 发送信号

Django中有两种方法用于发送信号。

`Signal.``send`(_sender_, _**kwargs_)

`Signal.``send_robust`(_sender_, _**kwargs_)

调用&nbsp;[`Signal.send()`](#django.dispatch.Signal.send "django.dispatch.Signal.send")或者[`Signal.send_robust()`](#django.dispatch.Signal.send_robust "django.dispatch.Signal.send_robust")来发送信号。你必须提供`sender` 参数（大多数情况下它是一个类），并且可以提供尽可能多的关键字参数。

例如，这样来发送我们的`pizza_done`信号：

```
class PizzaStore(object):
    ...

    def send_pizza(self, toppings, size):
        pizza_done.send(sender=self.__class__, toppings=toppings, size=size)
        ...

```

`send()` 和`send_robust()`都会返回一个含有二元组的列表 `[(receiver, response), ... `]，它代表了被调用的接收器函数和他们的响应值。

`send()` 与 `send_robust()`在处理接收器函数产生的异常时有所不同。`send()`_不会_ 捕获任何由接收器产生的异常。它会简单地让错误往上传递。所以在错误产生的情况，不是所有接收器都会获得通知。

`send_robust()`捕获所有继承自Python `Exception`类的异常，并且确保所有接收器都能得到信号的通知。如果发生了错误，错误的实例会在产生错误的接收器的二元组中返回。

New in Django 1.8:

调用`send_robust()`的时候，所返回的错误的`__traceback__`属性上会带有&nbsp;traceback。

## 断开信号

`Signal.``disconnect`([_receiver=None_, _sender=None_, _weak=True_, _dispatch_uid=None_])

调用[`Signal.disconnect()`](#django.dispatch.Signal.disconnect "django.dispatch.Signal.disconnect")来断开信号的接收器。&nbsp;[`Signal.connect()`](#django.dispatch.Signal.connect "django.dispatch.Signal.connect")中描述了所有参数。如果接收器成功断开，返回&nbsp;`True` ，否则返回`False`。

`receiver`参数表示要断开的已注册接收器。如果`dispatch_uid` 用于定义接收器，可以为`None`。

Changed in Django 1.8:

增加了返回的布尔值。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Signals](https://docs.djangoproject.com/en/1.8/topics/signals/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
