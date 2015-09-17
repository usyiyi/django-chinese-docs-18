# 系统检查框架

New in Django 1.7\.

系统检查框架是为了验证Django项目的一系列静态检查。它可以检测到普遍的问题，并且提供如何修复的提示。这个框架可以被扩展，所以你可以轻易地添加你自己的检查。

检查可以由[`check`](../ref/django-admin.html#django-admin-check)命令显式触发。检查会在大多数命令之前隐式触发，包括[`runserver`](../ref/django-admin.html#django-admin-runserver) 和 [`migrate`](../ref/django-admin.html#django-admin-migrate)。由于性能因素，检查不作为在部署中使用的WSGI栈的一部分运行。如果你需要在你的部署服务器上运行系统检查，显式使用[`check`](../ref/django-admin.html#django-admin-check)来触发它们。

严重的错误会完全阻止Django命令(像[`runserver`](../ref/django-admin.html#django-admin-runserver))的运行。少数问题会通过控制台来报告。如果你检查了警告的原因，并且愿意无视它，你可以使用你项目设置文件中的[`SILENCED_SYSTEM_CHECKS`](../ref/settings.html#std:setting-SILENCED_SYSTEM_CHECKS) 设置，来隐藏特定的警告。

[_系统检查参考_](../ref/checks.html)中列出了所有Django可执行的所有检查。

## 编写你自己的检查

这个框架十分灵活，允许你编写函数，执行任何其他类型的所需检查。下面是一个桩（stub）检查函数的例子：

```
from django.core.checks import register

@register()
def example_check(app_configs, **kwargs):
    errors = []
    # ... your check logic here
    return errors

```

检查函数_必须_接受 `app_configs`参数；这个参数是要被检查的应用列表。如果是None，检查会运行在项目中_所有_安装的应用上。`**kwargs`参数用于进一步的扩展。

### 消息

这个函数必须返回消息的列表。如果检查的结果中没有发现问题，检查函数必须返回一个空列表。

_class _`CheckMessage`(_level_, _msg_, _hint_, _obj=None_, _id=None_)

由检查方法产生的警告和错误必须是[`CheckMessage`](#django.core.checks.CheckMessage "django.core.checks.CheckMessage")的示例。[`CheckMessage`](#django.core.checks.CheckMessage "django.core.checks.CheckMessage")的实例封装了一个可报告的错误或者警告。它同时也提供了可应用到消息的上下文或者提示，以及一个用于过滤的唯一的标识符。

它的概念非常类似于[_消息框架_](../ref/contrib/messages.html)或者 [_日志框架_](logging.html)中的消息。消息使用表明其严重性的`level` 来标记。

构造器的参数是：

`level`

The severity of the message. Use one of the
predefined values: `DEBUG`, `INFO`, `WARNING`, `ERROR`,
`CRITICAL`. If the level is greater or equal to `ERROR`, then Django
will prevent management commands from executing. Messages with
level lower than `ERROR` (i.e. warnings) are reported to the console,
but can be silenced.

`msg`

A short (less than 80 characters) string describing the problem. The string
should _not_ contain newlines.

`hint`

A single-line string providing a hint for fixing the problem. If no hint
can be provided, or the hint is self-evident from the error message, the
hint can be omitted, or a value of `None` can be used.

`obj`

Optional. An object providing context for the message (for example, the
model where the problem was discovered). The object should be a model, field,
or manager or any other object that defines `__str__` method (on
Python 2 you need to define `__unicode__` method). The method is used while
reporting all messages and its result precedes the message.

`id`

Optional string. A unique identifier for the issue. Identifiers should
follow the pattern `applabel.X001`, where `X` is one of the letters
`CEWID`, indicating the message severity (`C` for criticals,
`E` for errors and so). The number can be allocated by the application,
but should be unique within that application.

也有一些快捷方式，使得创建通用级别的消息变得简单。当使用这些方法时你可以忽略`level`参数，因为它由类名称暗示。

_class _`Debug`(_msg_, _hint_, _obj=None_, _id=None_)

_class _`Info`(_msg_, _hint_, _obj=None_, _id=None_)

_class _`Warning`(_msg_, _hint_, _obj=None_, _id=None_)

_class _`Error`(_msg_, _hint_, _obj=None_, _id=None_)

_class _`Critical`(_msg_, _hint_, _obj=None_, _id=None_)

消息是可比较的。你可以轻易地编写测试：

```
from django.core.checks import Error
errors = checked_object.check()
expected_errors = [
    Error(
        'an error',
        hint=None,
        obj=checked_object,
        id='myapp.E001',
    )
]
self.assertEqual(errors, expected_errors)

```

### 注册和标记检查

最后，你的检查函数必须使用系统检查登记处来显式注册。

`register`(_*tags)(function_)

你可以向&nbsp;`register`传递任意数量的标签来标记你的检查。Tagging checks is useful since it allows you to run only a certain group of checks. For example, to register a compatibility check, you would make the following call:

```
from django.core.checks import register, Tags

@register(Tags.compatibility)
def my_check(app_configs, **kwargs):
    # ... perform compatibility checks and collect errors
    return errors

```

New in Django 1.8\.

你可以注册“部署的检查”，它们只和产品配置文件相关，像这样：

```
@register(Tags.security, deploy=True)
def my_check(app_configs, **kwargs):
    ...

```

这些检查只在 [`--deploy`](../ref/django-admin.html#django-admin-option---deploy) 选项传递给[`check`](../ref/django-admin.html#django-admin-check) 命令的情况下运行。

你也可以通过向`register`传递一个可调用对象（通常是个函数）作为第一个函数，将&nbsp;`register`作为函数使用，而不是一个装饰器。

下面的代码和上面等价：

```
def my_check(app_configs, **kwargs):
    ...
register(my_check, Tags.security, deploy=True)

```

Changed in Django 1.8:

添加了将注册用作函数的功能。

### 字段、模型和管理器检查

在一些情况下，你并不需要注册检查函数 -- 你可以直接使用现有的注册。

字段、方法和模型管理器都实现了`check()` 方法，它已经使用检查框架注册。如果你想要添加额外的检查，你可以扩展基类中的实现，进行任何你需要的额外检查，并且将任何消息附加到基类生成的消息中。强烈推荐你将每个检查分配到单独的方法中。

考虑一个例子，其中你要实现一个叫做`RangedIntegerField`的自定义字段。这个字段向`IntegerField`的构造器中添加`min` 和 `max` 参数。你可能想添加一个检查，来确保用户提供了小于等于最大值的最小值。下面的代码段展示了如何实现这个检查：

```
from django.core import checks
from django.db import models

class RangedIntegerField(models.IntegerField):
    def __init__(self, min=None, max=None, **kwargs):
        super(RangedIntegerField, self).__init__(**kwargs)
        self.min = min
        self.max = max

    def check(self, **kwargs):
        # Call the superclass
        errors = super(RangedIntegerField, self).check(**kwargs)

        # Do some custom checks and add messages to `errors`:
        errors.extend(self._check_min_max_values(**kwargs))

        # Return all errors and warnings
        return errors

    def _check_min_max_values(self, **kwargs):
        if (self.min is not None and
                self.max is not None and
                self.min > self.max):
            return [
                checks.Error(
                    'min greater than max.',
                    hint='Decrease min or increase max.',
                    obj=self,
                    id='myapp.E001',
                )
            ]
        # When no error, return an empty list
        return []

```

如果你想要向模型管理器添加检查，应该在你的[`Manager`](db/managers.html#django.db.models.Manager "django.db.models.Manager")的子类上执行同样的方法。

如果你想要向模型类添加检查，方法也_大致_相同：唯一的不同是检查是类方法，并不是实例方法：

```
class MyModel(models.Model):
    @classmethod
    def check(cls, **kwargs):
        errors = super(MyModel, cls).check(**kwargs)
        # ... your own checks ...
        return errors

```

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[System check framework](https://docs.djangoproject.com/en/1.8/topics/checks/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
