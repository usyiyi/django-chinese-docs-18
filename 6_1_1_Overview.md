# Django 的设置 #

Django 的设置文件包含你安装的Django 的所有配置。这页文档解释设置是如何工作以及有哪些设置。

## 基础 ##

设置文件只是一个Python 模块，带有模块级别的变量。

下面是一些示例设置：

```
ALLOWED_HOSTS = ['www.example.com']
DEBUG = False
DEFAULT_FROM_EMAIL = 'webmaster@example.com'
```

> 注
>
> 如果你设置`DEBUG` 为`False`，那么你应该正确设置`ALLOWED_HOSTS` 的值。

因为设置文件是一个Python 模块，所以适用以下情况：

+ 不允许出现Python 语法错误。
+ 它可以使用普通的Python 语法动态地设置。例如：

```
MY_SETTING = [str(i) for i in range(30)]
```

+ 它可以从其它设置文件导入值。

## 指定设置文件 ##

`DJANGO_SETTINGS_MODULE`

当你使用Django 时，你必须告诉它你正在使用哪个设置。这可以使用环境变量`DJANGO_SETTINGS_MODULE` 来实现。

`DJANGO_SETTINGS_MODULE` 的值应该使用Python 路径的语法，例如`mysite.settings`。注意，设置模块应该在Python 的导入查找路径 中。

### django-admin 工具 ###

当使用`django-admin` 时， 你可以设置只设置环境变量一次，或者每次运行该工具时显式传递设置模块。

例如（Unix Bash shell）：

```
export DJANGO_SETTINGS_MODULE=mysite.settings
django-admin runserver
```

例如（Windows shell）：

```
set DJANGO_SETTINGS_MODULE=mysite.settings
django-admin runserver
```

使用--settings 命令行参数可以手工指定设置：

```
django-admin runserver --settings=mysite.settings
```

### 在服务器上(mod_wsgi) ###

在线上服务器环境中，你需要告诉WSGI 的application 使用哪个设置文件。可以使用os.environ 实现：

```
import os

os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'
```

阅读[Django mod_wsgi 文档](http://python.usyiyi.cn/django/howto/deployment/wsgi/modwsgi.html) 以获得关于Django WSGI application 的更多和其它常见信息。

## 默认的设置 ##

Django 的设置文件不需要定义所有的设置。每个设置都有一个合理的默认值。这些默认值位于`django/conf/global_settings.py` 模块中。

下面是Django 用来编译设置的算法：

+ 从`global_settings.py` 中加载设置。
+ 从指定的设置文件中加载设置，如有必要则覆盖全局的设置。

注意，设置文件不 应该从global_settings 中导入，因为这是多余的。

### 查看改变的设置 ###

有一个简单的方法可以查看哪些设置与默认的设置不一样了。python `manage.py` `diffsettings` 命令显示当前的设置文件和Django 默认设置之间的差异。

获取更多信息，查看`diffsettings` 的文档。

## 在Python 代码中使用设置 ##

在Django 应用中，可以通过导入`django.conf.settings` 对象来使用设置。例如：

```
from django.conf import settings

if settings.DEBUG:
    # Do something
```

注意，`django.conf.settings` 不是一个模块 —— 它是一个对象。所以不可以导入每个单独的设置：

```
from django.conf.settings import DEBUG  # This won't work.
```

还要注意，你的代码不应该 从`global_settings` 或你自己的设置文件中导入。`django.conf.settings` 抽象出默认设置和站点特定设置的概念；它表示一个单一的接口。它还可以将代码从你的设置所在的位置解耦出来。

## 运行时改变设置 ##

请不要在应用运行时改变设置。例如，不要在视图中这样做：

```
from django.conf import settings

settings.DEBUG = True   # Don't do this!
```

给设置赋值的唯一地方是在设置文件中。

## 安全 ##

因为设置文件包含敏感的信息，例如数据库密码，你应该尽一切可能来限制对它的访问。例如，修改它的文件权限使得只有你和Web 服务器使用者可以读取它。这在共享主机的环境中特别重要。

## 可用的设置 ##

完整的可用设置清单，请参见[设置参考](http://python.usyiyi.cn/django/ref/settings.html)。

## 创建你自己的设置 ##

没有什么可以阻止你为自己的Django 应用创建自己的设置。只需要遵循下面的一些惯例：

+ 设置名称全部是大写
+ 不要使用一个已经存在的设置

对于序列类型的设置，Django 自己使用元组而不是列表，但这只是一个习惯。

## 不用DJANGO_SETTINGS_MODULE 设置 ##

有些情况下，你可能想绕开`DJANGO_SETTINGS_MODULE` 环境变量。例如，如果你正在使用自己的模板系统，而你不想建立指向设置模块的环境变量。

这些情况下，你可以手工配置Django 的设置。实现这点可以通过调用：

`django.conf.settings.configure(default_settings, **settings)`

例如：

```
from django.conf import settings

settings.configure(DEBUG=True)
```

可以传递`configure()` 给任意多的关键字参数，每个关键字参数表示一个设置及其值。每个参数的名称应该都是大写，与上面讲到的设置名称相同。如果某个设置没有传递给`configure()` 而且在后面需要使用到它，Django 将使用其默认设置的值。

当你在一个更大的应用中使用到Django 框架的一部分，有必要以这种方式配置Django —— 而且实际上推荐这么做。

所以，当通过`settings.configure()` 配置时，Django 不会对进程的环境变量做任何修改（参见`TIME_ZONE` 文档以了解为什么会发生）。在这些情况下，它假设你已经完全控制你的环境变量。

## 自定义默认的设置 ##

如果你想让默认值来自其它地方而不是`django.conf.global_settings`，你可以传递一个提供默认设置的模块或类作为`default_settings` 参数（或第一个位置参数）给`configure()` 调用。

在下面的示例中，默认的设置来自`myapp_defaults`， 并且设置`DEBUG` 为`True`，而不论它在`myapp_defaults` 中的值是什么：

```
from django.conf import settings
from myapp import myapp_defaults

settings.configure(default_settings=myapp_defaults, DEBUG=True)
```

下面的示例和上面一样，只是使用`myapp_defaults` 作为一个位置参数：

```
settings.configure(myapp_defaults, DEBUG=True)
```

正常情况下，你不需要用这种方式覆盖默认值。Django 的默认值以及足够好使，你可以安全地使用它们。注意，如果你传递一个新的默认模块，你将完全取代 Django 的默认值，所以你必须指定每个可能用到的设置的值。完整的设置清单，参见`django.conf.settings.global_settings`。

### configure() 和DJANGO_SETTINGS_MODULE 两者必居其一 ###

如果你没有设置`DJANGO_SETTINGS_MODULE` 环境变量，你 必须 在使用到读取设置的任何代码之前调用`configure()` 。

如果你没有设置`DJANGO_SETTINGS_MODULE` 且没有调用 `configure()`，在首次访问设置时Django 将引发一个`ImportError` 异常。

如果你设置了`DJANGO_SETTINGS_MODULE`，并访问了一下设置，然后 调用`configure()`，Django 将引发一个`RuntimeError` 表示该设置已经有配置。有个属性正好可以用于这个情况：

例如：

```
from django.conf import settings
if not settings.configured:
    settings.configure(myapp_defaults, DEBUG=True)
```

另外，多次调用`configure()`或者在设置已经访问过之后调用 `configure()` 都是错误的。

归结为一点：只使用`configure()` 或 `DJANGO_SETTINGS_MODULE` 中的一个。不可以两个都用和都不用。

> 另见
>
> [设置参考](http://python.usyiyi.cn/django/ref/settings.html)
> 包含完整的核心设置和contrib 应用设置的列表。

&zwj;

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Overview](https://docs.djangoproject.com/en/1.8/topics/settings/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
