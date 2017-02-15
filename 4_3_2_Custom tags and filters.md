{% raw %}

# 自定义模板标签和过滤器

为了解决应用中展示逻辑的需求，Django的模板语言提供了各式各样的[_内建标签以及过滤器_](../ref/templates/builtins.html)。然而，你或许会发现模板内建的这些工具集合不一定能全部满足你的功能需要。在Python中，你可以通过自定义标签或过滤器的方式扩展模板引擎的功能，并使用[`{{ load }}`](../ref/templates/builtins.html#std:templatetag-load)标签在你的模板中进行调用。

## 代码布局

自定义模板标签和过滤器必须位于Django 的某个应用中。如果它们与某个已存在的应用相关，那么将其与应用绑在一起才有意义；否则，就应该创建一个新的应用来包含它。

这个应用应该包含一个`templatetags` 目录，和`models.py`、`views.py`等文件处于同一级别目录下。如果目录不存在则创建它——不要忘记创建`__init__.py` 文件以使得该目录可以作为Python 的包。在添加这个模块以后，在模板里使用标签或过滤器之前你将需要重启服务器。

你的自定义的标签和过滤器将放在`templatetags` 目录下的一个模块里。这个模块的名字是你稍后将要载入标签时使用的，所以要谨慎的选择名字以防与其他应用下的自定义标签和过滤器名字冲突。

例如，你的自定义标签/过滤器在一个名为`poll_extras.py`的文件中，那么你的app目录结构看起来应该是这样的：

```
polls/
    __init__.py
    models.py
    templatetags/
        __init__.py
        poll_extras.py
    views.py

```

然后你可以在模板中像如下这样使用：

```
{% load poll_extras %}

```

为了让[`{{ load }}`](../ref/templates/builtins.html#std:templatetag-load) 标签工作，包含自定义标签的应用必须在[`INSTALLED_APPS`](../ref/settings.html#std:setting-INSTALLED_APPS)中。这是一种安全功能︰它允许你在单个主机上Host 许多模板库的Python 代码，而不必让每个Django 都可以访问所有的模板库。

在 `templatetags` 包中放多少个模块没有限制。只需要记住[`{% load %}`](../ref/templates/builtins.html#std:templatetag-load) 声明将会载入给定模块名中的标签/过滤器，而不是应用的名称。

为了成为一个可用的标签库，这个模块必须包含一个名为 `register`的变量，它是`template.`Library 的一个实例，所有的标签和过滤器都是在其中注册的。所以把如下的内容放在你的模块的顶部：

```
from django import template

register = template.Library()

```

幕后

对于大量的示例，请阅读Django的默认过滤器和标记的源代码。它们分别位于`django/template/defaultfilters.py` 和`django/template/defaulttags.py` 中。

有关[`load`](../ref/templates/builtins.html#std:templatetag-load) 标签的更多信息，请阅读其文档。

## 编写自定义模板过滤器

自定义过滤器就是一个带有一个或两个参数的Python 函数：

*   （输入的）变量的值 —— 不一定是字符串形式。
*   参数的值 —— 可以有一个初始值，或者完全不要这个参数。

例如，在`{{ var|foo:"bar" }}`中，`foo`过滤器应当传入变量`var`和参数 `"bar"`。

由于模板语言没有提供异常处理，任何从过滤器中抛出的异常都将会显示为服务器错误。因此，如果有合理的值可以返回，过滤器应该避免抛出异常。在模板中有一个明显错误的情况下，引发一个异常可能仍然要好于用静默的失败来掩盖错误。

这是一个定义过滤器的例子：

```
def cut(value, arg):
    """Removes all values of arg from the given string"""
    return value.replace(arg, '')

```

下面是这个过滤器应该如何使用：

```
{{ somevariable|cut:"0" }}

```

大多数过滤器没有参数。在这种情况下，你的函数不带这个参数即可。示例︰

```
def lower(value): # Only one argument.
    """Converts a string into all lowercase"""
    return value.lower()

```

### 注册自定义过滤器

`django.template.Library.``filter`()

一旦你写好了你的自定义过滤器函数，你就开始需要把它注册为你的 `Library`实例，来让它在Django模板语言中可用：

```
register.filter('cut', cut)
register.filter('lower', lower)

```

`Library.filter()`方法需要两个参数：

1.  过滤器的名称（一个字符串对象）
2.  编译的函数 – 一个Python函数（不要把函数名写成字符串）

你还可以把`register.filter()`用作装饰器：

```
@register.filter(name='cut')
def cut(value, arg):
    return value.replace(arg, '')

@register.filter
def lower(value):
    return value.lower()

```

如果你像上面第二个例子一样没有声明 `name` 参数，Django将使用函数名作为过滤器的名字。

最后，`register.filter()` 还接收三个关键字参数，`is_safe`、`needs_autoescape` 和`expects_localtime`。这些参数将在下边[_过滤器和自动转义_](#filters-auto-escaping) 以及[_过滤器和时区_](#filters-timezones) 章节中介绍。

### 期望字符串的过滤器

`django.template.defaultfilters.``stringfilter`()

如果你正在编写一个只希望用一个字符串来作为第一个参数的模板过滤器，你应当使用`stringfilter`装饰器。这将在对象被传入你的函数之前把这个对象转换成它的字符串值：

```
from django import template
from django.template.defaultfilters import stringfilter

register = template.Library()

@register.filter
@stringfilter
def lower(value):
    return value.lower()

```

用这种方式，你甚至可以给这个过滤器传递一个整数，并且不会出现`AttributeError` (因为整数没有 `lower()`方法).

### 过滤器和自动转义

编写一个自定义的过滤器时，请考虑一下过滤器如何与Django 的自定转义行为相互作用。请注意有三种类型的字符串可以传递给模板中的代码：

*   **原始字符串** 即Python 原生的`str` 或`unicode` 类型。输出时，如果自动转义生效则进行转义，否则保持不变。

*   **安全字符串** 是指在输出时已经被标记为安全而不用进一步转义的字符串。任何必要的转义已经完成。它们通常用于包含HTML 的输出，并希望在客户端解释为原始的形式。

    在内部，这些字符串是`SafeBytes` 或`SafeText` 类型。它们共享一个公共基类`SafeData`，所以你可以使用类似的代码测试他们︰

    ```
    if isinstance(value, SafeData):
        # Do something with the "safe" string.
        ...

    ```

*   **标记为“需要转义”的字符串** 在输出时_始终转义_，无论它们是否在[`autoescape`](../ref/templates/builtins.html#std:templatetag-autoescape) 块。然而，即使已经应用自动转义，这些字符也只会转义一次。

    在内部，这些字符串是`EscapeBytes` 或`EscapeText` 类型。通常你不需要担心这些；它们用于[`escape`](../ref/templates/builtins.html#std:templatefilter-escape) 过滤器的实现。

模板过滤代码最终是这两种中的一个：

1.  你的过滤器没有引进任何HTML 不安全字符（`&lt;`、`&gt;`、`'`、`"` 或`&`）到结果中。在这种情况下，你可以让Django 照顾你的所有的自动转义处理。你需要做的就是当你注册过滤器函数的时候将`is_safe` 标志设置为`True`，如下所示︰

    ```
    @register.filter(is_safe=True)
    def myfilter(value):
        return value

    ```

    这个标志告诉Django 如果"安全"的字符串传递到您的筛选器，结果仍将是"安全"，如果一个非安全字符串传递，如果必要Django 会自动转义它。

    你可以认为这个的意思是"此过滤器是安全的 —— 它没有引入任何不安全的HTML 的可能性。

    `is_safe` 存在的必要原因是因为有很多正常的字符串操作会将一个`SafeData` 对象转换回正常的`str` 或`unicode` 对象而不是试图捕获它们，Django 在过滤器完成之后会修复这种破坏。

    例如，假设你有一个过滤器将字符串`xx` 添加到任何输入的末尾。因为这没有引入危险的HTML 字符（已经存在的除外）的结果，你应该使用`is_safe`标记你的过滤器：

    ```
    @register.filter(is_safe=True)
    def add_xx(value):
        return '%sxx' % value

    ```

    当这个过滤器用在模板中启用自动转义的地方时，如果输入没有标记为“安全”，Django 将对输出进行转义。

    默认情况下，`is_safe` 为`False`，你可以在不需要的任何过滤器中省略它。

    决定你的过滤器是否真的会保持安全字符串是安全的时要小心。如果你在_删除_字符，可能会无意中在结果留下不平衡的 HTML 标记或实体。例如，从输入删除`&gt;` 可能将`&lt;a&gt;` 转变成`&lt;a`，这将需要对输出进行转义，避免造成问题。同样，删除一个分号（`;`） 可以将`&amp;` 转变成`&amp`，不再是一个有效的实体，因此需要进一步的转义。大多数情况下不会这么棘手，但在审查你的代码时要留意任何类似的问题。

    标记过滤器位`is_safe` 将强制过滤器的返回值为字符串。如果你的过滤器应返回一个布尔值或其他非字符串值，则将其标记`is_safe` 会有意想不到的后果 （如将布尔值 False 转换为字符串 'False'）。

2.  或者，你的过滤器代码手动照顾任何必要的转义。这在你正引入新的HTML 标记到结果中时是必要的。你想标记输出为安全的而不用进一步的转义，所以你需要自己处理输入输出。

    用[`django.utils.safestring.mark_safe()`](../ref/utils.html#django.utils.safestring.mark_safe "django.utils.safestring.mark_safe") 标记输出为安全字符。

    但你要小心。你需要做的不仅仅只是标记作为安全输出。您需要确保它真的_是_安全的，而你做什么取决于自动转义是否有效。这个想法的目的是编写的过滤器在无论模板自动转义是打开或关闭时都可以工作，这样模板作者使用起来更简单。

    为了使你的过滤器知道当前的自动转义状态，当你注册过滤器函数时需要设置`needs_autoescape` 标志为`True`。（如果不指定此标志，则默认为`False`）。此标志告诉Django 你的过滤器函数想要被传递一个额外的关键字参数，称为`autoescape`，如果启用自动转义则为`True`，否则为`False`。建议设置`autoescape` 参数的默认值设置为`True`，这样如果从Python 代码中调用该函数则会自动启用转义。

    例如，让我们编写一个强调字符串第一个字符的过滤器︰

    ```
    from django import template
    from django.utils.html import conditional_escape
    from django.utils.safestring import mark_safe

    register = template.Library()

    @register.filter(needs_autoescape=True)
    def initial_letter_filter(text, autoescape=True):
        first, other = text[0], text[1:]
        if autoescape:
            esc = conditional_escape
        else:
            esc = lambda x: x
        result = '&lt;strong&gt;%s&lt;/strong&gt;%s' % (esc(first), esc(other))
        return mark_safe(result)

    ```

    `needs_autoescape`标志和`autoescape`关键字参数意味着我们的函数将知道当调用过滤器时自动转义是否有效。我们使用`autoescape`来决定是否需要通过`django.utils.html.conditional_escape`传递输入数据。（在后一种情况下，我们只使用identity函数作为“escape”函数。）`conditional_escape()`函数与`escape()`类似，只不过它只转义了**而不是** a `SafeData`如果将`SafeData`实例传递给`conditional_escape()`，则数据不会改变。

    最后，在上面的例子中，我们记得将结果标记为安全的，这样我们的HTML直接插入到模板中，而不需要进一步转义。

    在这种情况下，没有必要担心`is_safe`标志（虽然包括它不会伤害任何东西）。每当你手动处理自动转义问题并返回一个安全的字符串，`is_safe`标志不会改变任何方式。

警告

在重用内置过滤器时避免XSS漏洞

Changed in Django 1.8.

Django的内置过滤器默认情况下具有`autoescape=True`，以便获得正确的自动转义行为并避免跨站点脚本漏洞。

在旧版本的Django中，在重新使用Django的内置过滤器时请小心，因为`autoescape`默认为`None`。您需要传递`autoescape=True`才能获得自动转义。

例如，如果您想编写一个称为`urlize_and_linebreaks`的自定义过滤器，它结合了[`urlize`](../ref/templates/builtins.html#std:templatefilter-urlize)和[`linebreaksbr`](../ref/templates/builtins.html#std:templatefilter-linebreaksbr)过滤器，过滤器将如下所示：

```
from django.template.defaultfilters import linebreaksbr, urlize

@register.filter(needs_autoescape=True)
def urlize_and_linebreaks(text, autoescape=True):
    return linebreaksbr(
        urlize(text, autoescape=autoescape),
        autoescape=autoescape
    )

```

然后：

```
{{ comment|urlize_and_linebreaks }}

```

将等同于︰

```
{{ comment|urlize|linebreaksbr }}

```

### 过滤器和时区

如果你在编写操作[`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)") 对象的自定义过滤器，你注册时通常需要将 `expects_localtime` 标志设置为`True`：

```
@register.filter(expects_localtime=True)
def businesshours(value):
    try:
        return 9 <= value.hour < 17
    except AttributeError:
        return ''

```

当设置了此标志，如果你的过滤器的第一个参数是时区相关的日期时间值，Django 会根据[_模板中的时区转换规则_](../topics/i18n/timezones.html#time-zones-in-templates) 在把它传递到你的过滤器之前将其转换为当前时区。

## 编写自定义的模板标签

标签比过滤器更复杂，因为标签可以做任何事情。Django 提供大量的快捷方式，使编写大多数类型的标签更容易。首先我们要探讨这些快捷方式，然后再解释当快捷方式不够用时如何为这些情况从头开始编写标签。

### 简单的标签

`django.template.Library.``simple_tag`()

许多模板标签接收多个参数 —— 字符串或模板变量 —— 并在基于输入的参数和一些其它外部信息进行一些处理后返回一个字符串。例如，`current_time` 标签可能接受一个格式字符串，并返回与之对应的格式化后的时间。

为了简单化这些类型标签的创建，Django 提供一个辅助函数`simple_tag`。这个函数是`django.template.Library` 的一个方法，接受一个任意数目的参数的函数，将其包装在一个`render` 函数和上面提到的其他必要位，并在模板系统中注册它。

我们的`current_time` 函数从而可以这样写︰

```
import datetime
from django import template

register = template.Library()

@register.simple_tag
def current_time(format_string):
    return datetime.datetime.now().strftime(format_string)

```

关于`simple_tag` 辅助函数几件值得注意的事项︰

*   检查所需参数的数量等等，在我们的函数调用的时刻已经完成，所以我们不需要做了。
*   参数（如果有）的引号都已经被截掉，所以我们收到的只是一个普通字符串。
*   如果该参数是一个模板变量，传递给我们的函数是当前变量的值，不是变量本身。

如果你的模板标签需要访问当前上下文，你可以在注册标签时使用`takes_context` 参数︰

```
@register.simple_tag(takes_context=True)
def current_time(context, format_string):
    timezone = context['timezone']
    return your_get_current_time_method(timezone, format_string)

```

请注意，第一个参数_必须_称作`context`。

`takes_context` 选项的工作方式的详细信息，请参阅[_包含标签_](#howto-custom-template-tags-inclusion-tags)。

如果你需要重命名你的标签，你可以给它提供自定义的名称︰

```
register.simple_tag(lambda x: x - 1, name='minusone')

@register.simple_tag(name='minustwo')
def some_function(value):
    return value - 2

```

`simple_tag` 函数可以接受任意数量的位置参数和关键字参数。例如：

```
@register.simple_tag
def my_tag(a, b, *args, **kwargs):
    warning = kwargs['warning']
    profile = kwargs['profile']
    ...
    return ...

```

然后在模板中，可以将任意数量的由空格分隔的参数传递给模板标签。像在Python 中一样，关键字参数的值的设置使用等号（"`=`"） ，并且必须在位置参数之后提供。例如：

```
{% my_tag 123 "abcd" book.title warning=message|lower profile=user.profile %}

```

### Inclusion 标签

`django.template.Library.``inclusion_tag`()

另一种常见类型的模板标签是通过渲染_另外一个_模板来显示一些数据。例如，Django 的Admin 界面使用自定义模板标签显示"添加/更改"表单页面底部的按钮。这些按钮看起来总是相同，但链接的目标根据正在编辑的对象而变化 —— 所以它们是使用小模板展示当前对象详细信息很好的例子。（在Admin 界面这种情况下，它是`submit_row` 标记）。

这些类型的标签被称为"Inclusion 标签"。

示例最能体现如何编写Inclusion 标签。让我们编写一个根据给定的[_教程_](../intro/tutorial01.html#creating-models)中创建的`Poll` 对象输出一个选项列表的标签。标签的用法像这样︰

```
{% show_results poll %}

```

... 输出将像这样：

```
<ul>
  <li>First choice</li>
  <li>Second choice</li>
  <li>Third choice</li>
</ul>

```

首先，定义接收这个参数并产生数据字典作为结果的函数。这里重要的一点是，我们只需要返回一个字典，不需要任何复杂的东西。它将用做模板片段的模板上下文。例如：

```
def show_results(poll):
    choices = poll.choice_set.all()
    return {'choices': choices}

```

接下来，创建用于渲染标签输出的模板。这个模板是标签固定的功能︰标签的编写者指定它，不是模板设计者。在我们的例子中，模板非常简单︰

```
<ul>
{% for choice in choices %}
    <li> {{ choice }} </li>
{% endfor %}
</ul>

```

现在，通过调用`Library` 对象的`inclusion_tag()` 方法创建并注册Inclusion 标签。在我们的示例中，如果上面的模板叫做`results.html` 文件，并位于模板加载程序搜索的目录，我们将这样注册标签︰

```
# Here, register is a django.template.Library instance, as before
@register.inclusion_tag('results.html')
def show_results(poll):
    ...

```

或者可以使用[`django.template.Template`](../ref/templates/api.html#django.template.Template "django.template.Template") 实例注册Inclusion标签︰

```
from django.template.loader import get_template
t = get_template('results.html')
register.inclusion_tag(t)(show_results)

```

......当首次创建该函数时。

有时，你的Inclusion 标签可能要求一大堆参数，这让模板作者非常痛苦，因为不仅要传递这些参数还要记住它们的顺序。为了解决这个问题， Django 提供了一个`takes_context` 选项给Inclusion 标签。如果你在创建模板标签时指定`takes_context`，这个标签将不需要必选参数，当标签被调用的时候底层的Python 函数将有一个参数 —— 模板上下文。

比如说,当你想要写一个 inclusion tag总是应用在上下文中，包含 `home_link` 和 `home_title` 这两个用来返回主页的变量。 如下所示:

```
@register.inclusion_tag('link.html', takes_context=True)
def jump_link(context):
    return {
        'link': context['home_link'],
        'title': context['home_title'],
    }

```

注意函数的第一个参数_必须_叫做`context`。

在`register.inclusion_tag()`这一行，我们指定了`takes_context=True` 和模板的名字。这里是模板`link.html` 看起来的样子：

```
Jump directly to <a href="{{ link }}">{{ title }}</a>.

```

然后，当任何时候你想调用这个自定义的标签， load 它的 library 然后不需要任何参数就是调用它，就像这样：

```
{% jump_link %}

```

注意当你使用`takes_context=True`，就不需要传递参数给这个模板标签。它会自己去获取上下文。

`takes_context` 参数默认为`False`。当它设置为`True` 时，会传递上下文对象给这个标签，如本示例所示。这是这个示例和前面的`inclusion_tag` 示例的唯一区别。

`inclusion_tag` 函数可以接受任意数量的位置参数和关键字参数。例如：

```
@register.inclusion_tag('my_template.html')
def my_tag(a, b, *args, **kwargs):
    warning = kwargs['warning']
    profile = kwargs['profile']
    ...
    return ...

```

然后在模板中，可以将任意数量的由空格分隔的参数传递给模板标签。像在Python 中一样，关键字参数的值的设置使用等号（"`=`"） ，并且必须在位置参数之后提供。例子：

```
{% my_tag 123 "abcd" book.title warning=message|lower profile=user.profile %}

```

### 赋值标签

`django.template.Library.``assignment_tag`()

为了简单化设置上下文中变量的标签的创建，Django 提供一个辅助函数`assignment_tag`。这个函数方式的工作方式与[_simple_tag_](#howto-custom-template-tags-simple-tags) 相同，不同之处在于它将标签的结果存储在指定的上下文变量中而不是直接将其输出。

我们之前的`current_time` 函数从而可以这样写︰

```
@register.assignment_tag
def get_current_time(format_string):
    return datetime.datetime.now().strftime(format_string)

```

然后你可以使用`as` 参数后面跟随变量的名称将结果存储在模板变量中，并将它输出到你觉得合适的地方︰

```
{% get_current_time "%Y-%m-%d %I:%M %p" as the_time %}
<p>The time is {{ the_time }}.</p>

```

如果你的模板标签需要访问当前上下文，你可以在注册标签时使用`takes_context` 参数：

```
@register.assignment_tag(takes_context=True)
def get_current_time(context, format_string):
    timezone = context['timezone']
    return your_get_current_time_method(timezone, format_string)

```

注意函数的第一个参数_必须_叫做`context`。

`takes_context` 选项的工作方式的详细信息，请参阅[_包含标签_](#howto-custom-template-tags-inclusion-tags)。

`assignment_tag` 函数可以接受任意数量的位置参数和关键字参数。例如：

```
@register.assignment_tag
def my_tag(a, b, *args, **kwargs):
    warning = kwargs['warning']
    profile = kwargs['profile']
    ...
    return ...

```

然后在模板中，可以将任意数量的由空格分隔的参数传递给模板标签。像在Python 中一样，关键字参数的值的设置使用等号（"`=`"） ，并且必须在位置参数之后提供。例子：

```
{% my_tag 123 "abcd" book.title warning=message|lower profile=user.profile as the_result %}

```

### 自定义模板标签进阶

有时创建自定义模板标签的基本功能是不够的。别担心，Django 给你建立模板标签所需的从底层访问完整的内部。

### 概述

模板系统的运行分为两步︰编译和渲染。若要定义一个自定义的模板标签，你指定编译如何工作以及渲染如何工作。

当Django 编译一个模板时，它将原始模板文本拆分成节点。每个节点是`django.template.Node` 的一个实例，并且有一个`render()` 方法。编译后的模板就是一个简单`Node` 对象的列表。当你在编译后的模板对象上调用`render()` 时，该模板将结合给定的上下文调用每个`Node` 的`render()`。结果所有串联在一起形成该模板的输出。

因此，若要定义一个自定义的模板标签，你需要指定原始模板标签如何被转换成一个`Node(节点)` （编译函数），以及该节点的`render()` 方法会进行的渲染动作

### 写编译函数

解析器处理每个模板标签时，会调用标签上下文对应的函数和对象本身。这个函数会会返回一个`Node`实例

例如，让我们写一个简单的模板标签的完整实现：​​`}} current_time }}`根据[`strftime()`](https://docs.python.org/3/library/time.html#time.strftime "(in Python v3.4)")语法中标记中给出的参数格式化的当前日期/时间。在任何事情之前决定标记语法是个好主意。在我们的例子中，我们假设标签应该像这样使用：

```
<p>The time is {% current_time "%Y-%m-%d %I:%M %p" %}.</p>

```

此函数的解析器应抓取参数并创建`节点`对象：

```
from django import template

def do_current_time(parser, token):
    try:
        # split_contents() knows not to split quoted strings.
        tag_name, format_string = token.split_contents()
    except ValueError:
        raise template.TemplateSyntaxError(
            "%r tag requires a single argument" % token.contents.split()[0]
        )
    if not (format_string[0] == format_string[-1] and format_string[0] in ('"', "'")):
        raise template.TemplateSyntaxError(
            "%r tag's argument should be in quotes" % tag_name
        )
    return CurrentTimeNode(format_string[1:-1])

```

笔记：

*   `parser`是模板解析器对象。在这个例子中我们不需要它。
*   `token.contents`是标记的原始内容的字符串。在我们的示例中，它是`'current_time “％Y-％m-％d ％I：％M ％p “'`”。
*   `token.split_contents()`方法将空格上的参数分隔开，同时将带引号的字符串保存在一起。更直接的`token.contents.split()`将不会那么健壮，因为它会在_所有_空间（包括引用字符串中的那些）上天真地分割。始终使用`token.split_contents()`是一个好主意。
*   此函数负责提高`django.template.TemplateSyntaxError`，包含有用的消息，任何语法错误。
*   `TemplateSyntaxError`异常使用`tag_name`变量。不要在错误消息中硬编码标记的名称，因为它会将标记的名称与您的函数相关联。`token.contents.split()[0]`将“永远”是您的标记的名称 - 即使标记没有参数。
*   该函数返回`CurrentTimeNode`，其中包含节点需要了解的关于此标记的所有内容。在这种情况下，它只传递参数 - `“％Y-％m-％d ％I：％M ％p” t3 &gt;`。模板标记的前导和尾部引号在`format_string[1:-1]`中删除。
*   解析是非常低级的。Django开发人员已经尝试在这个解析系统之上编写小框架，使用诸如EBNF语法的技术，但是这些实验使得模板引擎太慢。它是低级的，因为这是最快的。

### 写入渲染器

编写自定义标记的第二步是定义具有`render()`方法的`Node`子类。

继续上面的例子，我们需要定义`CurrentTimeNode`：

```
import datetime
from django import template

class CurrentTimeNode(template.Node):
    def __init__(self, format_string):
        self.format_string = format_string

    def render(self, context):
        return datetime.datetime.now().strftime(self.format_string)

```

笔记：

*   `__init__()`从`do_current_time()`获取`format_string`。始终通过其`__init__()`将任何选项/参数/参数传递到`Node`。
*   `render()`方法是工作实际发生的地方。
*   `render()`通常应该默认失败，特别是在生产环境中。但在某些情况下，特别是`context.template.engine.debug`是`True`时，此方法可能会引发异常，使调试更容易。例如，如果几个核心标记接收到错误的数字或类型的参数，则会产生`django.template.TemplateSyntaxError`。

最终，编译和渲染的这种解耦导致了高效的模板系统，因为模板可以呈现多个上下文而不必被多次解析。

### 自动转义注意事项

模板标签的输出**不是**自动运行通过自动转义过滤器。但是，在编写模板标记时，您仍然需要记住一些事情。

如果模板的`render()`函数将结果存储在上下文变量中（而不是返回字符串中的结果），则应小心调用`mark_safe()`当变量最终呈现时，它会受到当时有效的自动转义设置的影响，因此应该避免进一步转义的内容需要标记为这样。

此外，如果您的模板标记为执行某些子呈现创建了一个新的上下文，请将auto-escape属性设置为当前上下文的值。`Context`类的`__init__`方法使用一个名为`autoescape`的参数，可以用于此目的。例如：

```
from django.template import Context

def render(self, context):
    # ...
    new_context = Context({'var': obj}, autoescape=context.autoescape)
    # ... Do something with new_context ...

```

这不是一个很常见的情况，但它是有用的，如果你自己渲染一个模板。例如：

```
def render(self, context):
    t = context.template.engine.get_template('small_fragment.html')
    return t.render(Context({'var': obj}, autoescape=context.autoescape))

```

Changed in Django 1.8:

在Django 1.8中添加了`Context`对象的`template`属性。[`context.template.engine.get_template`](../ref/templates/api.html#django.template.Engine.get_template "django.template.Engine.get_template") must be used instead of [`django.template.loader.get_template()`](../topics/templates.html#django.template.loader.get_template "django.template.loader.get_template") because the latter now returns a wrapper whose `render` method doesn’t accept a [`Context`](../ref/templates/api.html#django.template.Context "django.template.Context").

如果我们在此示例中忽略将当前的`context.autoescape`值传递给我们的新`Context`，则结果将自动转义 ，如果在[`}} autoescape off }}`](../ref/templates/builtins.html#std:templatetag-autoescape)块。

### 线程安全注意事项

一旦节点被解析，其`render`方法可以被调用任何次数。由于Django有时在多线程环境中运行，因此单个节点可以响应于两个单独的请求而用不同的上下文同时呈现。因此，确保您的模板标记是线程安全的很重要。

为了确保你的模板标签是线程安全的，你不应该在节点本身存储状态信息。例如，Django提供了一个内置的[`cycle`](../ref/templates/builtins.html#std:templatetag-cycle)模板标签，每次呈现时都会在给定字符串列表之间循环：

```
{% for o in some_list %}
    <tr class="{% cycle 'row1' 'row2' %}">
        ...
    </tr>
{% endfor %}

```

`CycleNode`的简单实现可能如下所示：

```
import itertools
from django import template

class CycleNode(template.Node):
    def __init__(self, cyclevars):
        self.cycle_iter = itertools.cycle(cyclevars)

    def render(self, context):
        return next(self.cycle_iter)

```

但是，假设我们有两个模板同时从上面呈现模板片段：

1.  线程1执行其第一次循环迭代，`CycleNode.render()`返回'row1'
2.  线程2执行其第一次循环迭代，`CycleNode.render()`返回'row2'
3.  线程1执行其第二次循环迭代，`CycleNode.render()`返回'row1'
4.  线程2执行其第二循环迭代，`CycleNode.render()`返回'row2'

CycleNode是迭代，但它是全局迭代。就线程1和线程2而言，它总是返回相同的值。这显然不是我们想要的！

为了解决这个问题，Django提供了与当前正在渲染的模板的`context`相关联的`render_context`。`render_context`的行为类似于Python字典，应该用于在`render`方法的调用之间存储`Node`状态。

让我们重构我们的`CycleNode`实现以使用`render_context`：

```
class CycleNode(template.Node):
    def __init__(self, cyclevars):
        self.cyclevars = cyclevars

    def render(self, context):
        if self not in context.render_context:
            context.render_context[self] = itertools.cycle(self.cyclevars)
        cycle_iter = context.render_context[self]
        return next(cycle_iter)

```

请注意，存储在`Node`的整个生命周期中不会改变的全局信息作为属性是完全安全的。在`CycleNode`的情况下，`cyclevars`参数在实例化`Node`后不会改变，因此我们不需要将其`render_context`。但是，当前正在渲染的模板特定的状态信息（如`CycleNode`的当前迭代）应存储在`render_context`中。

注意

请注意我们如何使用`self`来限定`render_context`内的`CycleNode`特定信息。在给定模板中可能有多个`CycleNodes`，因此我们需要注意不要破坏另一个节点的状态信息。执行此操作的最简单方法是始终使用`self`作为`render_context`中的键。如果你跟踪几个状态变量，使`render_context[self]`一个字典。

### 注册标签

最后，按照上述[_writing custom template filters_](#howto-writing-custom-template-tags)中的说明，向模块的`Library`实例注册代码。例：

```
register.tag('current_time', do_current_time)

```

`tag()`方法有两个参数：

1.  模板标记的名称 - 字符串。如果省略，将使用编译函数的名称。
2.  编译函数 - 一个Python函数（不是作为字符串的函数的名称）。

与过滤器注册一样，也可以将其用作装饰器：

```
@register.tag(name="current_time")
def do_current_time(parser, token):
    ...

@register.tag
def shout(parser, token):
    ...

```

如果您省略`name`参数，如上面的第二个示例，Django将使用函数的名称作为标记名称。

### 将模板变量传递给标记

虽然您可以使用`token.split_contents()`向模板标记传递任意数量的参数，但所有参数都会作为字符串文字解压缩。为了将动态内容（模板变量）传递给模板标签作为参数，需要更多的工作。

虽然先前的示例已将当前时间格式化为字符串并返回了字符串，但假设您希望从对象传递[`DateTimeField`](../ref/models/fields.html#django.db.models.DateTimeField "django.db.models.DateTimeField")，并具有date-time的模板标记格式：

```
<p>This post was last updated at {% format_time blog_entry.date_updated "%Y-%m-%d %I:%M %p" %}.</p>

```

最初，`token.split_contents()`将返回三个值：

1.  标记名称`format_time`。
2.  字符串`'blog_entry.date_updated'`（不带周围的引号）。
3.  格式字符串`'“％Y-％m-％d ％I：％M ％p”' 。``split_contents()`的返回值将包括字符串文字的前导和尾部引号，如下所示。

现在您的标记应该开始看起来像这样：

```
from django import template

def do_format_time(parser, token):
    try:
        # split_contents() knows not to split quoted strings.
        tag_name, date_to_be_formatted, format_string = token.split_contents()
    except ValueError:
        raise template.TemplateSyntaxError(
            "%r tag requires exactly two arguments" % token.contents.split()[0]
        )
    if not (format_string[0] == format_string[-1] and format_string[0] in ('"', "'")):
        raise template.TemplateSyntaxError(
            "%r tag's argument should be in quotes" % tag_name
        )
    return FormatTimeNode(date_to_be_formatted, format_string[1:-1])

```

您还必须更改渲染器以检索`blog_entry`对象的`date_updated`属性的实际内容。这可以通过使用`django.template`中的`Variable()`类来实现。

要使用`Variable`类，只需使用要解析的变量名称实例化它，然后调用`variable.resolve(context)`。所以，例如：

```
class FormatTimeNode(template.Node):
    def __init__(self, date_to_be_formatted, format_string):
        self.date_to_be_formatted = template.Variable(date_to_be_formatted)
        self.format_string = format_string

    def render(self, context):
        try:
            actual_date = self.date_to_be_formatted.resolve(context)
            return actual_date.strftime(self.format_string)
        except template.VariableDoesNotExist:
            return ''

```

如果变量分辨率无法解析在页面的当前上下文中传递给它的字符串，则会抛出`VariableDoesNotExist`异常。

### 在上下文中设置变量

上面的例子只是输出一个值。通常，如果您的模板标记设置模板变量而不是输出值，则更灵活。这样，模板作者可以重用您的模板标签创建的值。

要在上下文中设置变量，只需对`render()`方法中的上下文对象使用字典分配。以下是更新版本的`CurrentTimeNode`，它设置模板变量`current_time`，而不是输出：

```
import datetime
from django import template

class CurrentTimeNode2(template.Node):
    def __init__(self, format_string):
        self.format_string = format_string
    def render(self, context):
        context['current_time'] = datetime.datetime.now().strftime(self.format_string)
        return ''

```

请注意，`render()`返回空字符串。`render()`应始终返回字符串输出。如果所有模板标签都设置了一个变量，`render()`应该返回空字符串。

以下说明如何使用这个新版本的标记：

```
{% current_time "%Y-%M-%d %I:%M %p" %}<p>The time is {{ current_time }}.</p>

```

上下文中的变量范围

上下文中的任何变量集都只能在分配给它的模板的相同`block`中使用。这种行为是故意的；它为变量提供了一个范围，使它们不与其他块中的上下文冲突。

但是，`CurrentTimeNode2`有一个问题：变量名称`current_time`是硬编码的。这意味着您需要确保您的模板在其他地方不使用`{{ current_time }}` ，因为`}} current_time }}`会盲目地覆盖该变量的值。一个更清洁的解决方案是使模板标签指定输出变量的名称，如下所示：

```
{% current_time "%Y-%M-%d %I:%M %p" as my_current_time %}
<p>The current time is {{ my_current_time }}.</p>

```

为此，您需要重构编译函数和`Node`类，如下所示：

```
import re

class CurrentTimeNode3(template.Node):
    def __init__(self, format_string, var_name):
        self.format_string = format_string
        self.var_name = var_name
    def render(self, context):
        context[self.var_name] = datetime.datetime.now().strftime(self.format_string)
        return ''

def do_current_time(parser, token):
    # This version uses a regular expression to parse tag contents.
    try:
        # Splitting by None == splitting by spaces.
        tag_name, arg = token.contents.split(None, 1)
    except ValueError:
        raise template.TemplateSyntaxError(
            "%r tag requires arguments" % token.contents.split()[0]
        )
    m = re.search(r'(.*?) as (\w+)', arg)
    if not m:
        raise template.TemplateSyntaxError("%r tag had invalid arguments" % tag_name)
    format_string, var_name = m.groups()
    if not (format_string[0] == format_string[-1] and format_string[0] in ('"', "'")):
        raise template.TemplateSyntaxError(
            "%r tag's argument should be in quotes" % tag_name
        )
    return CurrentTimeNode3(format_string[1:-1], var_name)

```

这里的区别在于`do_current_time()`抓取格式字符串和变量名称，将两者传递到`CurrentTimeNode3`。

最后，如果您只需要为自定义上下文更新模板标记提供一个简单的语法，则可以考虑使用上面介绍的[_assignment tag_](#howto-custom-template-tags-assignment-tags)快捷方式。

### 解析直到另一个块标记

模板标签可以协同工作。例如，标签[`}} comment }}`](../ref/templates/builtins.html#std:templatetag-comment)标签隐藏所有内容，直到`}} endcomment }}`。要创建这样的模板标签，请在编译函数中使用`parser.parse()`。

以下是简化的`}} 注释 }}`标记的实现方法：

```
def do_comment(parser, token):
    nodelist = parser.parse(('endcomment',))
    parser.delete_first_token()
    return CommentNode()

class CommentNode(template.Node):
    def render(self, context):
        return ''

```

注意

[`}} comment }}`](../ref/templates/builtins.html#std:templatetag-comment)的实际执行方式略有不同， `}} 注释 }}`和`}} }}`。它通过调用`parser.skip_past('endcomment')`而不是`parser.parse(('endcomment',))`，然后紧跟`parser.delete_first_token()`，从而避免生成节点列表。

`parser.parse()`使用块标签的名称的元组来解析，直到“”。它返回一个`django.template.NodeList`的实例，它是所有`Node`对象的列表，解析器遇到“'before”'遇到任何名为的标签元组。

In `"nodelist = parser.parse(('endcomment',))"` in the above example, `nodelist` is a list of all nodes between the `}} comment }}` and `}} endcomment }}`, not counting `}} comment }}` and `}} endcomment }}` themselves.

调用`parser.parse()`后，解析器尚未“消耗”`}} endcomment } }`标记，因此代码需要显式调用`parser.delete_first_token()`。

`CommentNode.render()`只返回一个空字符串。`}} 注释之间的任何内容 }}`和`}} endcomment }}`被忽略。

### 解析直到另一个块标签，并保存内容

在上一个示例中，`do_comment()`舍弃了`}} 注释之间的所有内容 }} t2 &gt;和`}} endcomment }}`。`而不是这样做，可以用块标记之间的代码做一些事情。

For example, here’s a custom template tag, `}} upper }}`, that capitalizes everything between itself and `}} endupper }}`.

用法：

```
{% upper %}This will appear in uppercase, {{ your_name }}.{% endupper %}

```

和前面的例子一样，我们使用`parser.parse()`。但是这次，我们将得到的`节点`传递给`节点`：

```
def do_upper(parser, token):
    nodelist = parser.parse(('endupper',))
    parser.delete_first_token()
    return UpperNode(nodelist)

class UpperNode(template.Node):
    def __init__(self, nodelist):
        self.nodelist = nodelist
    def render(self, context):
        output = self.nodelist.render(context)
        return output.upper()

```

这里唯一的新概念是`UpperNode.render()`中的`self.nodelist.render（context）`。

For more examples of complex rendering, see the source code of [`}} for }}`](../ref/templates/builtins.html#std:templatetag-for) in `django/template/defaulttags.py` and [`}} if }}`](../ref/templates/builtins.html#std:templatetag-if) in `django/template/smartif.py`.

{% endraw %}