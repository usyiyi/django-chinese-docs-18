{% raw %}

# Django 模板语言：面向Python程序员

本文从技术的角度解释Django 模板系统 —— 它如何工作以及如何继承它。如果你正在查找语言语法方面的参考，参见[_Django 模板语言_](language.html)。

假设你已经理解了模板、上下文、变量、标签和渲染。如果你不熟悉这些概念，从阅读 [_Django 模板语言_](../../topics/templates.html#template-language-intro)起步吧。



## 概述

在Python中使用模板系统有三个步骤：

1.  配置[`引擎`](#django.template.Engine "django.template.Engine")。
2.  将模板代码编译成[`模板`](#django.template.Template "django.template.Template")。
3.  根据[`上下文`](#django.template.Context "django.template.Context")渲染模板。

对于这些步骤，Django 的项目一般使用[_高级的、与后端无关的API_](../../topics/templates.html#template-engines) ，而不用模板系统的底层API：

1.  Django?对[`TEMPLATES`](../settings.html#std:setting-TEMPLATES) ?设置中的每个[`DjangoTemplates`](../../topics/templates.html#django.template.backends.django.DjangoTemplates "django.template.backends.django.DjangoTemplates") 后端实例化一个[`引擎`](#django.template.Engine "django.template.Engine")。[`DjangoTemplates`](../../topics/templates.html#django.template.backends.django.DjangoTemplates "django.template.backends.django.DjangoTemplates") 封装[`引擎`](#django.template.Engine "django.template.Engine")并将它适配成通用的模板后端API。
2.  [`django.template.loader`](../../topics/templates.html#module-django.template.loader "django.template.loader") 模块提供一些函数例如[`get_template()`](../../topics/templates.html#django.template.loader.get_template "django.template.loader.get_template") 来加载模板。它们返回一个`django.template.backends.django.Template`，这个返回值封装了真正的[`django.template.Template`](#django.template.Template "django.template.Template")。
3.  在上一步中获得的`模板` 有一个[`render()`](../../topics/templates.html#django.template.backends.base.Template.render "django.template.backends.base.Template.render") 方法， 该方法将上下文和HTTP 请求添加到[`Context`](#django.template.Context "django.template.Context")中并将渲染委托给底层的[`模板`](#django.template.Template "django.template.Template")。





## 配置引擎



_class_ `Engine`(_[dirs][, app_dirs][, allowed_include_roots][, context_processors][, debug][, loaders][, string_if_invalid][, file_charset]_)



New in Django 1.8.

实例化`Engine` 时，所有的参数必须通过关键字参数传递：

*   `dirs` 是一个列表，包含引擎查找模板源文件的目录。它用于配置[`filesystem.`](#django.template.loaders.filesystem.Loader "django.template.loaders.filesystem.Loader")Loader。

    默认为一个空的列表。

*   `app_dirs` 只影响`loaders` 的默认值。参见下文。

    默认为`False`。

*   `allowed_include_roots` 是一个字符串列表，它们表示`{% ssi %}` 模板标签的前缀。这是一个安全措施，让模板的作者不能访问他们不应该访问的文件。

    例如，如果`'allowed_include_roots'` 为`['/home/html', '/var/www']`，那么`{% ssi /home/html/foo.txt %}` 可以工作，而`{% ssi /etc/passwd %}` 不能工作。

    默认为一个空的列表。

    

    Deprecated since version 1.8: 废弃`allowed_include_roots`。

    

*   `context_processors` 是一个Python 可调用对象的路径列表，它们用于模板渲染时填充其上下文。这些可调用对象接收一个HTTP 请求对象作为它们的参数，并返回一个 [`字典`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.4)") 用于合并到上下文中。

    它默认为一个空的列表。

    更多信息，参见[`RequestContext`](#django.template.RequestContext "django.template.RequestContext")。

*   `debug` 是一个布尔值，表示打开/关闭模板的调试模式。如果为`True`，模板引擎将保存额外的调试信息，这些信息可以用来显示模板渲染过程中引发的异常的详细报告。

    它默认为`False`。

*   `loaders` 是模板加载类的一个列表，这些类由字符串表示。每个`Loader` 类知道如何从一个特定的源导入模板。还可以使用元组代替字符串表示这些类。元组的第一个元素应该是`Loader` 类的名称，接下来的元素将在`Loader` 初始化时用于初始化。

    它默认为包含下面内容的列表：

    *   `'django.template.loaders.filesystem.Loader'`
    *   `'django.template.loaders.app_directories.`Loader' 当前仅当`app_dirs` 为`True`。

    细节参见[_Loader types_](#template-loaders)。

*   `string_if_invalid` 表示模板系统遇到不合法（例如，拼写错误）的变量时应该使用的字符串。

    默认为空字符串。

    细节参见[_如何处理不合法的变量_](#invalid-template-variables)。

*   `file_charset` 是读取磁盘上的模板文件使用的字符集。

    默认为`'utf-8'`。







_static_ `Engine.get_default`()



当Django 项目配置仅配置一个[`DjangoTemplates`](../../topics/templates.html#django.template.backends.django.DjangoTemplates "django.template.backends.django.DjangoTemplates") 引擎时，这个方法返回底层的[`引擎`](#django.template.Engine "django.template.Engine")。在其它情况下，它将引发[`ImproperlyConfigured`](../exceptions.html#django.core.exceptions.ImproperlyConfigured "django.core.exceptions.ImproperlyConfigured")。

It’s required for preserving APIs that rely on a globally available, implicitly configured engine. Any other use is strongly discouraged.







`Engine.from_string`(_template_code_)



编译给定的template_code 并返回一个[`Template`](#django.template.Template "django.template.Template") 对象。







`Engine.get_template`(_template_name_)



根据给定的名称，编译并返回一个[`Template`](#django.template.Template "django.template.Template") 对象。







`Engine.select_template`(_self_, _template_name_list_)



类似[`get_template()`](#django.template.Engine.get_template "django.template.Engine.get_template")，不同的是它接收一个名称列表并返回找到的第一个模板。









## 加载模板

建议调用[`Engine`](#django.template.Engine "django.template.Engine") 的工厂方法创建[`Template`](#django.template.Template "django.template.Template")：[`get_template()`](#django.template.Engine.get_template "django.template.Engine.get_template")、[`select_template()`](#django.template.Engine.select_template "django.template.Engine.select_template") 和 [`from_string()`](#django.template.Engine.from_string "django.template.Engine.from_string")。

在[`TEMPLATES`](../settings.html#std:setting-TEMPLATES)?只定义一个[`DjangoTemplates`](../../topics/templates.html#django.template.backends.django.DjangoTemplates "django.template.backends.django.DjangoTemplates") 引擎的Django 项目中，可以直接实例化[`Template`](#django.template.Template "django.template.Template")。



_class_ `Template`



这个类位于`django.template.Template`。其构造函数接收一个参数 —— 原始的模板代码：





```
from django.template import Template

template = Template("My name is {{ my_name }}.")

```











幕后

系统只会解析一次原始的模板代码 —— 当创建`Template` 对象的时候。在此之后，处于性能考虑，会在内部将它存储为一个树形结构。

解析器本身非常快。大部分解析的动作只需要调用一个简短的正则表达式。







## 渲染上下文

一旦编译好[`Template`](#django.template.Template "django.template.Template") 对象，你就可以用上下文渲染它了。你可以使用不同的上下文多次重新渲染相同的模板。



_class_ `Context`(_[dict_][, current_app]_)



这个类位于`django.template.Context`。其构造函数接收两个可选的参数：

*   一个字典，映射变量名到变量的值。

*   当前应用的名称。该应用的名称用于帮助[_解析带命名空间的URLs_](../../topics/http/urls.html#topics-http-reversing-url-namespaces)。如果没有使用带命名空间的URL，可以忽略这个参数。

    

    Deprecated since version 1.8: 废弃`current_app` 参数。如果需要它，则必须使用[`RequestContext`](#django.template.RequestContext "django.template.RequestContext") 代替[`Context`](#django.template.Context "django.template.Context")。

    

细节参见下文的[_使用上下文对象_](#playing-with-context)。







`Template.render`(_context_)



使用[`Context`](#django.template.Context "django.template.Context") 调用[`Template`](#django.template.Template "django.template.Template") 对象的`render()` 方法来“填充”模板：





```
>>> from django.template import Context, Template
>>> template = Template("My name is {{ my_name }}.")

>>> context = Context({"my_name": "Adrian"})
>>> template.render(context)
"My name is Adrian."

>>> context = Context({"my_name": "Dolores"})
>>> template.render(context)
"My name is Dolores."

```











### 变量及其查找

变量名必须由字母、数字、下划线（不能以下划线开头）和点组成。

点在模板渲染时有特殊的含义。变量名中点表示**查找**。具体一点，当模板系统遇到变量名中的一个点时，它会按下面的顺序进行查找：

*   字典查找。例如：`foo["bar"]`
*   属性查找。例如：`foo.bar`
*   列表索引查找。例如：`foo[bar]`

注意，像`{{ foo.bar }}` 这种模版表达式中的“bar”，如果在模版上下文中存在，将解释为一个字符串字面量而不是使用变量“bar”的值。

模板系统使用找到的第一个可用的类型。这是一个短路逻辑。下面是一些示例：





```
>>> from django.template import Context, Template
>>> t = Template("My name is {{ person.first_name }}.")
>>> d = {"person": {"first_name": "Joe", "last_name": "Johnson"}}
>>> t.render(Context(d))
"My name is Joe."

>>> class PersonClass: pass
>>> p = PersonClass()
>>> p.first_name = "Ron"
>>> p.last_name = "Nasty"
>>> t.render(Context({"person": p}))
"My name is Ron."

>>> t = Template("The first stooge in the list is {{ stooges.0 }}.")
>>> c = Context({"stooges": ["Larry", "Curly", "Moe"]})
>>> t.render(c)
"The first stooge in the list is Larry."

```





如果变量的任何部分是可调用的，模板系统将尝试调用它。例如：





```
>>> class PersonClass2:
...     def name(self):
...         return "Samantha"
>>> t = Template("My name is {{ person.name }}.")
>>> t.render(Context({"person": PersonClass2}))
"My name is Samantha."

```





可调用的变量比只需直接查找的变量稍微复杂一些。需要记住下面几点：

*   如果变量在调用时引发一个异常，该异常将会传播，除非该异常的`silent_variable_failure` 属性的值为`True`。如果异常_确实_ 具有一个`silent_variable_failure` 属性且值为`True`，该变量将渲染成引擎的`string_if_invalid` 配置的值（默认为一个空字符串）。例如：

    

    

```
>>> t = Template("My name is {{ person.first_name }}.")
    >>> class PersonClass3:
    ...     def first_name(self):
    ...         raise AssertionError("foo")
    >>> p = PersonClass3()
    >>> t.render(Context({"person": p}))
    Traceback (most recent call last):
    ...
    AssertionError: foo

    >>> class SilentAssertionError(Exception):
    ...     silent_variable_failure = True
    >>> class PersonClass4:
    ...     def first_name(self):
    ...         raise SilentAssertionError
    >>> p = PersonClass4()
    >>> t.render(Context({"person": p}))
    "My name is ."
    
```

    

    

    注意，[`django.core.exceptions.ObjectDoesNotExist`](../exceptions.html#django.core.exceptions.ObjectDoesNotExist "django.core.exceptions.ObjectDoesNotExist") 的`silent_variable_failure = True`，它是Django 数据库API 所有`DoesNotExist` 异常的基类。所以，如果你在Django 模板中使用Django 模型对象，任何 `DoesNotExist` 异常都将默默地失败。

*   只有在变量不需要参数时才可调用。否则，系统将返回引擎的`string_if_invalid` 选项。

*   很显然，调用某些变量会带来副作用，允许模板系统访问它们将是愚蠢的还会带来安全漏洞。

    每个Django 模型对象的[`delete()`](../models/instances.html#django.db.models.Model.delete "django.db.models.Model.delete") 方法就是一个很好的例子。模板系统不应该允许下面的行为：

    

    

```
I will now delete this valuable data. {{ data.delete }}
    
```

    

    

    设置可调用变量的`alters_data` 属性可以避免这点。如果变量设置`alters_data=True` ，模板系统将不会调用它，而会无条件使用`string_if_invalid` 替换这个变量。Django 模型对象自动生成的[`delete()`](../models/instances.html#django.db.models.Model.delete "django.db.models.Model.delete") 和[`save()`](../models/instances.html#django.db.models.Model.save "django.db.models.Model.save") 方法自动 设置`alters_data=True`。 例如：

    

    

```
def sensitive_function(self):
        self.database_record.delete()
    sensitive_function.alters_data = True
    
```

    

    

*   有时候，处于某些原因你可能想关闭这个功能，并告诉模板系统无论什么情况下都不要调用变量。设置可调用对象的`do_not_call_in_templates` 属性的值为`True` 可以实现这点。模板系统的行为将类似这个变量是不可调用的（例如，你可以访问可调用对象的属性）。





### 如何处理不合法的变量

一般情况下，如果变量不存在，模板系统会插入引擎`string_if_invalid` 配置的值，其默认设置为`''`（空字符串）。

过滤器只有在`string_if_invalid` 设置为`''`（空字符串）时才会应用到不合法的变量上。如果`string_if_invalid` 设置为任何其它的值，将会忽略变量的过滤器。

这个行为对于`if`、`for` 和 `regroup` 模板标签有些不同。如果这些模板便签遇到不合法的变量，会将该变量解释为`None`。在这些模板标签中的过滤器对不合法的变量也会始终应用。

如果`string_if_invalid` 包含`'%s'`，这个格式标记将会被非法变量替换。



只用于调试目的！

虽然`string_if_invalid` 是一个很有用的调试工具，但是，将它作为“默认的开发工具”是个很糟糕的主意。

许多模板包括Admin 站点，在遇到不存在的变量时，依赖模板系统的沉默行为。如果你赋值非`''` 的值给`string_if_invalid`，使用这些模板和站点可能遇到渲染上的问题。

一般情况下，只有在调试一个特定的模板问题时才启用`string_if_invalid`，一旦调试完成就要清除。







### 内置的变量

每个上下文都包含`True`、`False` 和 `None`。和你期望的一样，这些变量将解析为对应的Python 对象。





### 字符串字面值的局限

Django 的模板语言没有办法转义它自己的语法用到的字符。例如，如果你需要输出字符序列`{%` 和`%}`，你需要用到[`templatetag`](builtins.html#std:templatetag-templatetag) 标签。

如果你想在模板过滤器或标签中包含这些序列，会存在类似的问题。例如，当解析block 标签时，Django 的模板解析器查找`%}` 之后出现的第一个`{%`。这将导致不能使用`"%}"` 这个字符串字面量。例如，下面的表达式将引发一个`TemplateSyntaxError`：





```
{% include "template.html" tvar="Some string literal with %} in it." %}

{% with tvar="Some string literal with %} in it." %}{% endwith %}

```





在过滤器参数中使用反向的序列会触发同样的问题：





```
{{ some.variable|default:"}}" }}

```





如果你需要使用这些字符串序列，可以将它们保存在模板变量中，或者自定义模板标签或过滤器来绕过这个限制。







## 使用Context 对象

大部分时候，你将通过传递一个完全填充的字典给`Context()` 来实例化一个[`Context`](#django.template.Context "django.template.Context") 对象。你也可以使用标准的字典语法在`Context` 对象实例化之后，向它添加和删除元素：





```
>>> from django.template import Context
>>> c = Context({"foo": "bar"})
>>> c['foo']
'bar'
>>> del c['foo']
>>> c['foo']
Traceback (most recent call last):
...
KeyError: 'foo'
>>> c['newvariable'] = 'hello'
>>> c['newvariable']
'hello'

```







`Context.get`(_key_, _otherwise=None_)



如果`key` 在Context 中，则返回`key` 的值，否则返回`otherwise`。







`Context.pop`()





`Context.push`()





_exception_ `ContextPopException`



`Context` 对象是一个栈。也就是说，你可以`push()` 和`pop()` 它。如果你`pop()` 得太多，它将引发`django.template.ContextPopException`：





```
>>> c = Context()
>>> c['foo'] = 'first level'
>>> c.push()
{}
>>> c['foo'] = 'second level'
>>> c['foo']
'second level'
>>> c.pop()
{'foo': 'second level'}
>>> c['foo']
'first level'
>>> c['foo'] = 'overwritten'
>>> c['foo']
'overwritten'
>>> c.pop()
Traceback (most recent call last):
...
ContextPopException

```





New in Django 1.7.

你还可以使用`push()` 作为上下文管理器以确保调用对应的`pop()`。





```
>>> c = Context()
>>> c['foo'] = 'first level'
>>> with c.push():
...     c['foo'] = 'second level'
...     c['foo']
'second level'
>>> c['foo']
'first level'

```





所有传递给`push()` 的参数将传递给`dict` 构造函数用于构造新的上下文层级。





```
>>> c = Context()
>>> c['foo'] = 'first level'
>>> with c.push(foo='second level'):
...     c['foo']
'second level'
>>> c['foo']
'first level'

```







`Context.update`(_other_dict_)



除了`push()` 和`pop()` 之外，`Context` 对象还定义一个`update()` 方法。它的工作方式类似`push()`，不同的是它接收一个字典作为参数并将该字典压入栈。





```
>>> c = Context()
>>> c['foo'] = 'first level'
>>> c.update({'foo': 'updated'})
{'foo': 'updated'}
>>> c['foo']
'updated'
>>> c.pop()
{'foo': 'updated'}
>>> c['foo']
'first level'

```





将`Context` 用作栈在[_一些自定义的标签中_](../../howto/custom-template-tags.html#howto-writing-custom-template-tags) 非常方便。



`Context.flatten`()



New in Django 1.7.

利用`flatten()` 方法，你可以获得字典形式的全部`Context` 栈，包括内建的变量。





```
>>> c = Context()
>>> c['foo'] = 'first level'
>>> c.update({'bar': 'second level'})
{'bar': 'second level'}
>>> c.flatten()
{'True': True, 'None': None, 'foo': 'first level', 'False': False, 'bar': 'second level'}

```





`flatten()` 方法在内部还被用来比较`Context` 对象。





```
>>> c1 = Context()
>>> c1['foo'] = 'first level'
>>> c1['bar'] = 'second level'
>>> c2 = Context()
>>> c2.update({'bar': 'second level', 'foo': 'first level'})
{'foo': 'first level', 'bar': 'second level'}
>>> c1 == c2
True

```





在单元测试中，`flatten()` 的结果可以用于比较`Context` 和`dict`：





```
class ContextTest(unittest.TestCase):
    def test_against_dictionary(self):
        c1 = Context()
        c1['update'] = 'value'
        self.assertEqual(c1.flatten(), {
            'True': True,
            'None': None,
            'False': False,
            'update': 'value',
        })

```







### 子类化Context：RequestContext



_class_ `RequestContext`(_request[, dict_][, processors]_)



Django 带有一个特殊的`Context` 类`django.template.RequestContext`，它与通常的`django.template.Context` 行为有少许不同。第一个不同点是，它接收一个[`HttpRequest`](../request-response.html#django.http.HttpRequest "django.http.HttpRequest") 作为第一个参数。例如：





```
c = RequestContext(request, {
    'foo': 'bar',
})

```





第二个不同点是，它根据引擎的 `context_processors` 配置选项自动向上下文中填充一些变量。

`context_processors` 选项是一个可调用对象 —— 叫做**上下文处理器** —— 的列表，它们接收一个请求对象作为它们的参数并返回需要向上下文中添加的字典。在生成的默认设置文件中，默认的模板引擎包含下面几个上下文处理器：





```
[
    'django.template.context_processors.debug',
    'django.template.context_processors.request',
    'django.contrib.auth.context_processors.auth',
    'django.contrib.messages.context_processors.messages',
]

```





Changed in Django 1.8:

Django 1.8 将模板内建的上下文处理器从`django.core.context_processors` 移动到`django.template.context_processors`中。



除此之外，[`RequestContext`](#django.template.RequestContext "django.template.RequestContext") 始终启用`'django.template.context_processors.csrf'`。这是一个安全相关的上下文处理器，Admin 和其它Contrib 应用需要它，而且为了防止意外的错误配置，它被有意硬编码在其中且在`context_processors` 选项中不可以关闭。

每个处理器按顺序启用。这意味着，如果一个处理器向上下文添加一个变量，而第二个处理器添加一个相同名称的变量，第二个将覆盖第一个。默认的处理器会在下面解释。



上下文处理器应用的时机

上下文处理器应用在上下文数据的顶端。也就是说，上下文处理器可能覆盖你提供给[`Context`](#django.template.Context "django.template.Context") 或[`RequestContext`](#django.template.RequestContext "django.template.RequestContext") 的变量，所以要注意避免与上下文处理器提供的变量名重复。

如果想要上下文数据的优先级高于上下文处理器，使用下面的模式：





```
from django.template import RequestContext

request_context = RequestContext(request)
request_context.push({"my_name": "Adrian"})

```





Django 通过这种方式允许上下文数据在[`render()`](../../topics/http/shortcuts.html#django.shortcuts.render "django.shortcuts.render") 和 [`TemplateResponse`](../template-response.html#django.template.response.TemplateResponse "django.template.response.TemplateResponse") 等API 中覆盖上下文处理器。



你还可以赋予[`RequestContext`](#django.template.RequestContext "django.template.RequestContext") 一个额外的处理器列表，使用第三个可选的位置参数`processors`。在下面的示例中，[`RequestContext`](#django.template.RequestContext "django.template.RequestContext") 实例获得一个`ip_address` 变量：





```
from django.http import HttpResponse
from django.template import RequestContext

def ip_address_processor(request):
    return {'ip_address': request.META['REMOTE_ADDR']}

def some_view(request):
    # ...
    c = RequestContext(request, {
        'foo': 'bar',
    }, [ip_address_processor])
    return HttpResponse(t.render(c))

```









### 内建的模板上下文处理器





### 上下文处理器

下面是每个内置的上下文处理器所做的事情：



#### django.contrib.auth.context_processors.auth

如果启用这个处理器，每个`RequestContext` 将包含以下变量：

*   `user` – 一个 `auth.`User实例代表当前登录的用户 (或者 一个 `AnonymousUser` 实例, 如果用户没有登录).
*   `perms` – 一个 `django.contrib.auth.context_processors.`PermWrapper实例, 代表当前登录用户所拥有的权限.





#### django.template.context_processors.debug

如果开启这个处理器，每一个`RequestContext`将会包含两个变量—但是只有当你的?[`DEBUG`](../settings.html#std:setting-DEBUG)配置设置为`True`时有效。请求的IP地址?(`request.`META['REMOTE_ADDR']) is in the [`INTERNAL_IPS`](../settings.html#std:setting-INTERNAL_IPS) setting:

*   `debug` – `True`. 你可以在模板中用它测试是否在[`DEBUG`](../settings.html#std:setting-DEBUG) 模式。
*   `sql_queries` – ?一个`{'sql': ..., 'time': ...}` 字典的列表，表示请求期间到目前为止发生的每个SQL 查询及花费的时间。这个列表按查询的顺序排序，并直到访问时才生成。





#### django.template.context_processors.i18n

如果启用这个处理器，每个`RequestContext` 将包含两个变量：

*   `LANGUAGES` – [`LANGUAGES`](../settings.html#std:setting-LANGUAGES) 设置的值。
*   `LANGUAGE_CODE` – `request.`LANGUAGE_CODE, if it exists. 否则为[`LANGUAGE_CODE`](../settings.html#std:setting-LANGUAGE_CODE) 设置的值。

更多信息，参见[_国际化和本地化_](../../topics/i18n/index.html)。





#### django.template.context_processors.media

如果启用这个处理器，每个`RequestContext` 将包含一个`MEDIA_URL` 变量，表示[`MEDIA_URL`](../settings.html#std:setting-MEDIA_URL) 设置的值。





#### django.template.context_processors.static



`static`()[[source]](../../_modules/django/template/context_processors.html#static)



如果启用这个处理器，每个`RequestContext` 将包含一个`STATIC_URL` 变量，表示[`STATIC_URL`](../settings.html#std:setting-STATIC_URL) 设置的值。





#### django.template.context_processors.csrf

上下文处理器添加一个token，这个token是?[`csrf_token`](builtins.html#std:templatetag-csrf_token) 模版标签需要的，用来针对[_Cross Site Request Forgeries_](../csrf.html).





#### django.template.context_processors.request

如果启用这个处理器，每个`RequestContext` 将包含一个`request` 变量，表示当前的[`HttpRequest`](../request-response.html#django.http.HttpRequest "django.http.HttpRequest")。





#### django.contrib.messages.context_processors.messages

如果启用这个处理器，每个`RequestContext` 将包含下面两个变量：

*   `messages` – 通过[_消息框架_](../contrib/messages.html)设置的消息（字符串形式）列表。
*   `DEFAULT_MESSAGE_LEVELS` – 消息等级名称到[_它们数值_](../contrib/messages.html#message-level-constants) 的映射。

Changed in Django 1.7:

添加`DEFAULT_MESSAGE_LEVELS` 变量。









### 编写你自己的上下文处理器

一个上下文处理器有一个非常简单的接口：它是一个参数的Python函数，这个参数是一个[`HttpRequest`](../request-response.html#django.http.HttpRequest "django.http.HttpRequest")对象，并且返回一个字典，这个字典会被添加到模版上下文中。每个上下文处理器_必须_返回一个字典。

Custom context processors can live anywhere in your code base. All Django cares about is that your custom context processors are pointed to by the `'context_processors'` option in your [`TEMPLATES`](../settings.html#std:setting-TEMPLATES) setting — or the `context_processors` argument of [`Engine`](#django.template.Engine "django.template.Engine") if you’re using it directly.







## 加载模板

通常情况下，你会将模板存储在文件系统上的文件中而不是自己使用底层的[`Template`](#django.template.Template "django.template.Template") API。保存模板的目录叫做**模板目录**。

Django 在许多地方查找模板目录，这取决于你的模板加载设置（参见下文中的“加载器类型”），但是指定模板目录最基本的方法是使用[`DIRS`](../settings.html#std:setting-TEMPLATES-DIRS) 选项。



### [`DIRS`](../settings.html#std:setting-TEMPLATES-DIRS)

Changed in Django 1.8:

这个值以前通过`TEMPLATE_DIRS` 设置定义。



设置文件中[`TEMPLATES`](../settings.html#std:setting-TEMPLATES) 设置的[`DIRS`](../settings.html#std:setting-TEMPLATES-DIRS) 选项或者[`Engine`](#django.template.Engine "django.template.Engine") 的`dirs` 参数用于告诉Django 你的模板目录。它应该设置为一个字符串列表，包含模板目录的完整路径：





```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            '/home/html/templates/lawrence.com',
            '/home/html/templates/default',
        ],
    },
]

```





模板可以位于任何位置，只要Web 服务器可以读取这些目录和模板。它们可以具有任何扩展名例如`.html` 或`.txt`，或者完全没有扩展名。

注意，这些路径应该使用Unix 风格的前向斜杠，即使在Windows 上。





### 加载器类型

默认情况下，Django 使用基于文件系统的模板加载器，但是Django 自带几个其它的模板加载器，它们知道如何从其它源加载模板。

默认情况下，某些其它的加载器是被禁用的，但是你可以向[`TEMPLATES`](../settings.html#std:setting-TEMPLATES) 设置中的`DjangoTemplates` 添加一个`'loaders'` 选项或者传递一个`loaders` 参数给[`Engine`](#django.template.Engine "django.template.Engine") 来激活它们。`loaders` 应该为一个字符串列表或元组，每个字符串表示一个模板加载器类。下面是Django 自带的模板加载器：

`django.template.loaders.filesystem.Loader`



_class_ `filesystem.Loader`



根据[`DIRS`](../settings.html#std:setting-TEMPLATES-DIRS)，从文件系统加载模板。

该加载器默认是启用的。当然，只有你将[`DIRS`](../settings.html#std:setting-TEMPLATES-DIRS) 设置为一个非空的列表，它才能找到模板：





```
TEMPLATES = [{
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'DIRS': [os.path.join(BASE_DIR, 'templates')],
}]

```









`django.template.loaders.app_directories.`Loader



_class_ `app_directories.Loader`



从文件系统加载Django 应用中的模板。对于[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 中的每个应用，该加载器会查找其下面的一个`templates` 子目录。如果该目录存在，Django 将在那里查找模板。

这意味着你可以将模板保存在每个单独的应用中。这还使得发布带有默认模板的Django 应用非常方便。

例如，对于这个设置：





```
INSTALLED_APPS = ('myproject.polls', 'myproject.music')

```





...`get_template('foo.html')` 将按顺序在下面的目录中查找`foo.html`：

*   `/path/to/myproject/polls/templates/`
*   `/path/to/myproject/music/templates/`

...并将使用第一个找到的模板。

[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 的顺序非常重要！例如，如果你想自定义Django Admin，你可能选择使用`myproject.polls` 中自己的`admin/base_site.html`覆盖`django.contrib.admin` 中标准的`admin/base_site.html` 。那么你必须确保在[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 中`myproject.polls` 位于`django.contrib.admin`_之前_，否则仍将加载 `django.contrib.admin` 中的模板并忽略你自己的模板。

注意，加载器在第一次运行时会做一些优化：它缓存一个含具有 `templates` 子目录的[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 包的列表。

你可以简单地通过设置[`APP_DIRS`](../settings.html#std:setting-TEMPLATES-APP_DIRS) 为`True` 来启用这个加载器：





```
TEMPLATES = [{
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'APP_DIRS': True,
}]

```









`django.template.loaders.eggs.Loader`



_class_ `eggs.Loader`



与上面的`app_directories` 类似，只是它从Python eggs 中而不是从文件系统中加载模板。

这个加载器在默认情况下是禁用的。





`django.template.loaders.cached.Loader`



_class_ `cached.Loader`



默认情况下，每当模版需要被渲染，模版系统将会读取和编译你的模版。但是，Django模版系统是非常高速的，?the overhead from reading and compiling?templates can add up.

基于缓存的模版加载器是一个基于类的加载器，that you configure with a list of other loaders?that it should wrap.The wrapped loaders are used to locate unknown templates when they are first encountered. 接下来，基于缓存加载器将编译过的`Template`存储在内存中。The cached `Template` instance is returned for subsequent requests to load the same template.

For example, to enable template caching with the `filesystem` and `app_directories` template loaders you might use the following settings:





```
TEMPLATES = [{
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'DIRS': [os.path.join(BASE_DIR, 'templates')],
    'OPTIONS': {
        'loaders': [
            ('django.template.loaders.cached.Loader', [
                'django.template.loaders.filesystem.Loader',
                'django.template.loaders.app_directories.Loader',
            ]),
        ],
    },
}]

```







Note

All of the built-in Django template tags are safe to use with the cached loader, but if you’re using custom template tags that come from third party packages, or that you wrote yourself, you should ensure that the `Node` implementation for each tag is thread-safe. For more information, see [_template tag thread safety considerations_](../../howto/custom-template-tags.html#template-tag-thread-safety).



这个加载器在默认情况下是禁用的。





`django.template.loaders.locmem.Loader`

New in Django 1.8.



_class_ `locmem.Loader`



从一个Python 目录中加载模板。它主要用于测试。

这个加载器接收一个模板字典作为它的第一个参数：





```
TEMPLATES = [{
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'OPTIONS': {
        'loaders': [
            ('django.template.loaders.locmem.Loader', {
                'index.html': 'content here',
            }),
        ],
    },
}]

```





这个加载器在默认情况下是禁用的。





Django 按照`'loaders'` 中选项的顺序使用模板加载器。它逐个使用每个加载器直到某个加载器找到一个匹配的模板。





### 自定义加载器

自定义`加载器`应该继承`django.template.loaders.base.Loader` 并覆盖`load_template_source()` 方法，这个方法接收一个`template_name` 参数、从磁盘（或其它地方）加载模板、然后返回一个元组：`(template_string, template_origin)`。

Changed in Django 1.8:

`django.template.loaders.base.Loader` 以前定义在 `django.template.loader.BaseLoader` 中。



`Loader`类的`load_template()` 方法通过调用`load_template_source()` 获取模板字符串、从模板源中实例化一个`Template`、然后返回一个元组：`(template, template_origin)`。







## 模板的origin 属性

New in Django 1.7.

当[`Engine`](#django.template.Engine "django.template.Engine") 使用`debug=True` 初始化时，它的模板将具有一个`origin` 属性，其值取决于模板加载的源。对于Django 初始化的引擎，`debug` 默认为[`DEBUG`](../settings.html#std:setting-DEBUG) 设置的值。



_class_ `loader.LoaderOrigin`



从模板加载器创建的模板将使用 `django.template.loader.LoaderOrigin` 类。



`name`



模板加载器返回的模板路径。对于从文件系统读取模板的加载器，它为模板的完整路径。







`loadname`



传递给模板加载器的模板相对路径。











_class_ `StringOrigin`



从`Template` 类创建的模板将使用`django.template.StringOrigin` 类。



`source`



用于创建模板的字符串。










{% endraw %}
