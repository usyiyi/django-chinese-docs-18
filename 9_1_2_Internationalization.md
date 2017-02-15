{% raw %}

# 翻译

## 概述

为了让Django项目可翻译，你必须添加一些钩子到你的Python 代码和模板中。这些钩子叫做[_翻译字符串_](index.html#term-translation-string)。它们告诉Django：“如果这个文本的翻译可用，应该将它翻译成终端用户的语言。”你需要标记这些可翻译的字符串；系统只会翻译它知道的字符串。

Django 提供一些工具用于提取翻译字符串到[_消息文件_](index.html#term-message-file)中。这个文件方便翻译人员提供翻译字符串的目标语言。翻译人员填充完消息文件后，必须编译它。这个过程依赖GNU gettext 工具集。

完成这些事情之后，Django 将负责根据用户的语言偏好将网页翻译成对应的语言。

Django 的国际化钩子默认是打开的，这表示框架的某些地方已经有相关的I18N 钩子。如果你不需要使用国际化，你可以花两秒钟将在设置文件中设置[`USE_I18N = False`](../../ref/settings.html#std:setting-USE_I18N)。 这样的话，Django 将做一些优化而不加载国际化的机制。

注

还有一个独立但是相关的设置[`USE_L10N`](../../ref/settings.html#std:setting-USE_L10N)，它控制Django 是否应该实现本地化格式。更多信息参见[_本地化格式_](formatting.html)。

注

确保你的项目已经启用翻译（最快的方法是检查[`MIDDLEWARE_CLASSES`](../../ref/settings.html#std:setting-MIDDLEWARE_CLASSES) 是否包含[`django.middleware.locale.LocaleMiddleware`](../../ref/middleware.html#django.middleware.locale.LocaleMiddleware "django.middleware.locale.LocaleMiddleware")）。如果还没有，请参见[_Django 如何发现语言偏好_](#how-django-discovers-language-preference)。

## 国际化：在Python 代码中

### 标准的翻译

函数[`ugettext()`](../../ref/utils.html#django.utils.translation.ugettext "django.utils.translation.ugettext") 用于指定标准的翻译。习惯上会将它导入成一个别名 `_` 以节省打字。

注

Python 的标准库`gettext` 模块将`_()` 安装进全局命名空间中，并作为`gettext()` 的别名。在Django 中，我们选择不遵守这个实践，原因有：

1.  对国际化的字符集（Unicode）的支持，[`ugettext()`](../../ref/utils.html#django.utils.translation.ugettext "django.utils.translation.ugettext") 比`gettext()` 更有用。有时候，对于特定的文件，你应该使用[`ugettext_lazy()`](../../ref/utils.html#django.utils.translation.ugettext_lazy "django.utils.translation.ugettext_lazy") 作为默认的翻译方法。如果`_()` 不在全局命名空间中，开发人员必须想清楚哪一个是最合适的翻译函数。
2.  下划线字符（`_`）在Python 的交互式shell 和doctest 测试中，用于表示“前一个结果”。安装全局的`_()` 函数会引起混乱。显式地导入`ugettext()` 为`_()` 将避免这个问题。

在下面的示例中，文本`"Welcome to my site."` 标记为一个翻译字符串：

```
from django.utils.translation import ugettext as _
from django.http import HttpResponse

def my_view(request):
    output = _("Welcome to my site.")
    return HttpResponse(output)

```

很明显，你可以不用别名来编写这段代码。下面的例子与前面完全一样：

```
from django.utils.translation import ugettext
from django.http import HttpResponse

def my_view(request):
    output = ugettext("Welcome to my site.")
    return HttpResponse(output)

```

翻译在计算生成的值上进行。下面的例子与前面两个完全一样：

```
def my_view(request):
    words = ['Welcome', 'to', 'my', 'site.']
    output = _(' '.join(words))
    return HttpResponse(output)

```

翻译可以在变量上进行。下面同样是一个完全一样的示例：

```
def my_view(request):
    sentence = 'Welcome to my site.'
    output = _(sentence)
    return HttpResponse(output)

```

(告诫使用变量或计算的值，如在前面的两个例子， 是Django的翻译字符串检测工具, [`django-admin makemessages`](../../ref/django-admin.html#django-admin-makemessages), 将不能够找到这些字符串。 后面有[`makemessages`](../../ref/django-admin.html#django-admin-makemessages) 的更多信息）。

传递给`_()` 或`ugettext()` 的字符串可以通过Python 标准的命名字符串插值语法接收占位符。示例：

```
def my_view(request, m, d):
    output = _('Today is %(month)s  %(day)s.') % {'month': m, 'day': d}
    return HttpResponse(output)

```

这种技术可以让语言相关的翻译重新排序。例如，英语翻译可能是`"Today is November 26."`，而西班牙语可能是`"Hoy es 26 de Noviembre."` —— 月份和天数的占位符交换位置了。

由于这个原因，每当有多个参数的时候，你都应该使用命名的字符串插值（例如，`%(day)s`）而不是位置插值（例如，`%s` 或`%d`）。如果使用位置插值，翻译将不能重新排序占位符。

### 给翻译人员的注释

如果你想给翻译人员一些提示，可以添加一个以`Translators` 为前缀的注释，例如：

```
def my_view(request):
    # Translators: This message appears on the home page only
    output = ugettext("Welcome to my site.")

```

这个注释会在生成的`.po` 文件中出现在可翻译的结构上方，而且可以在大部分翻译工具中显示出来。

注

只是为了完整性，下面是生成的`.po` 文件片段：

```
#. Translators: This message appears on the home page only
# path/to/python/file.py:123
msgid "Welcome to my site."
msgstr ""

```

它在模板中也可以工作。更多细节参见[_模板中给翻译人员的注释_](#translator-comments-in-templates)。

### 标记字符串为no-op

[`django.utils.translation.ugettext_noop()`](../../ref/utils.html#django.utils.translation.ugettext_noop "django.utils.translation.ugettext_noop") 函数用于标记字符串为一个翻译字符串但是不用翻译它。该字符串将在后面依据一个变量翻译。

如果你的常量字符串需要在不同的系统和用户之间交互 —— 例如数据库中的字符串，它们应该保存在源语言中，但是需要在最后例如呈现给用户的时刻翻译，可以使用它。

### 多元化

函数[`django.utils.translation.ungettext()`](../../ref/utils.html#django.utils.translation.ungettext "django.utils.translation.ungettext") 用于指定多元化的消息。

`ungettext` 接收三个参数：单数形式的翻译字符串、复数形式的翻译字符串和对象的个数。

这个函数用于当你的Django 应用所要本地化的语言中，[复数形式](http://www.gnu.org/software/gettext/manual/gettext.html#Plural-forms) 比英语中的要复杂时（‘object’ 表示单数，‘objects’ 表示所有`count` 不等于一的情形，无论具体的值是多少）。

例如：

```
from django.utils.translation import ungettext
from django.http import HttpResponse

def hello_world(request, count):
    page = ungettext(
        'there is %(count)d object',
        'there are %(count)d objects',
    count) % {
        'count': count,
    }
    return HttpResponse(page)

```

这个例子中对象的数字作为`count`变量传递给翻译语言

需要注意的是多元化是复杂的，在每种语言的工作方式不同。 `count` 计数到1的比较并不总是正确的规则。此代码看起来复杂，但会产生某些语言不正确的结果：

```
from django.utils.translation import ungettext
from myapp.models import Report

count = Report.objects.count()
if count == 1:
    name = Report._meta.verbose_name
else:
    name = Report._meta.verbose_name_plural

text = ungettext(
    'There is %(count)d  %(name)s available.',
    'There are %(count)d  %(name)s available.',
    count
) % {
    'count': count,
    'name': name
}

```

不要尝试自己去实现单复数逻辑, 这样会出错。这种情况下, 可以考虑这么做:

```
text = ungettext(
    'There is %(count)d  %(name)s object available.',
    'There are %(count)d  %(name)s objects available.',
    count
) % {
    'count': count,
    'name': Report._meta.verbose_name,
}

```

注意

在使用 `ungettext()`的时候, 确保你你使用在每一个占位符中使用同一个名称。在上面的例子中, 你可以注意到我们是怎么在两种翻译中使用`name`这个变量的。 下面这个例子中, 除了在某些语言中会表达不正确之外, 还可能会造成错误:

```
text = ungettext(
    'There is %(count)d  %(name)s available.',
    'There are %(count)d  %(plural_name)s available.',
    count
) % {
    'count': Report.objects.count(),
    'name': Report._meta.verbose_name,
    'plural_name': Report._meta.verbose_name_plural
}

```

运行 [`django-admin compilemessages`](../../ref/django-admin.html#django-admin-compilemessages)时，你会得到一个错误:

```
a format specification for argument 'name', as in 'msgstr[0]', doesn't exist in 'msgid'

```

注意

复数形式和po文件

Django不支持在po文件中的自定义复数方程。当所有翻译目录被合并时，仅考虑主Django po文件的复数形式（在`django/conf/locale/&lt;lang_code&gt;/LC_MESSAGES/django.po`）。所有其他po文件中的多个表单将被忽略。因此，您不应在项目或应用程序po文件中使用不同的复数方程。

### 上下文标记

有时候，词语有几种含义，例如英语中的`"May"`，指的是月份名称和动词。要使翻译者能够在不同的上下文中正确翻译这些单词，您可以使用[`django.utils.translation.pgettext()`](../../ref/utils.html#django.utils.translation.pgettext "django.utils.translation.pgettext")函数或[`django.utils.translation.npgettext()`](../../ref/utils.html#django.utils.translation.npgettext "django.utils.translation.npgettext")

在所得到的`.po`文件中，字符串将随着同一字符串存在不同的上下文标记而频繁出现（上下文将出现在`msgctxt`行）翻译给他们每个不同的翻译。

例如：

```
from django.utils.translation import pgettext

month = pgettext("month name", "May")

```

要么：

```
from django.db import models
from django.utils.translation import pgettext_lazy

class MyThing(models.Model):
    name = models.CharField(help_text=pgettext_lazy(
        'help text for MyThing model', 'This is the help text'))

```

将出现在`.po`文件中：

```
msgctxt "month name"
msgid "May"
msgstr ""

```

上下文标记也受[`trans`](#std:templatetag-trans)和[`blocktrans`](#std:templatetag-blocktrans)模板标记支持。

### 延迟翻译

使用[`django.utils.translation`](#module-django.utils.translation "django.utils.translation")中的翻译函数的惰性版本（可以通过名称中的`lazy`后缀轻松识别）来平移字符串 - 当访问该值而不是当他们被叫。

这些函数存储对字符串的惰性引用 - 而不是实际的翻译。当字符串在字符串上下文中使用时，例如在模板呈现中，翻译本身将被完成。

当对这些函数的调用位于在模块加载时执行的代码路径中时，这是至关重要的。

这在定义模型，表单和模型表单时很容易发生，因为Django实现了这些，使得它们的字段实际上是类级别的属性。因此，请确保在以下情况下使用延迟翻译：

#### 模型字段和关系`verbose_name`

例如，要翻译以下模型中_名称_字段的帮助文本，请执行以下操作：

```
from django.db import models
from django.utils.translation import ugettext_lazy as _

class MyThing(models.Model):
    name = models.CharField(help_text=_('This is the help text'))

```

您可以使用[`verbose_name`](../../ref/models/options.html#django.db.models.Options.verbose_name "django.db.models.Options.verbose_name")选项将[`ForeignKey`](../../ref/models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")，[`ManyToManyField`](../../ref/models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField")或[`OneToOneField`](../../ref/models/fields.html#django.db.models.OneToOneField "django.db.models.OneToOneField")关系标记为可翻译的名称：

```
class MyThing(models.Model):
    kind = models.ForeignKey(ThingKind, related_name='kinds',
                             verbose_name=_('kind'))

```

就像你在[`verbose_name`](../../ref/models/options.html#django.db.models.Options.verbose_name "django.db.models.Options.verbose_name")中所做的那样，你应该为关系提供一个小写的详细名称文本，因为Django会在需要时自动定义它。

#### 模型详细名称值

建议始终提供显式的[`verbose_name`](../../ref/models/options.html#django.db.models.Options.verbose_name "django.db.models.Options.verbose_name")和[`verbose_name_plural`](../../ref/models/options.html#django.db.models.Options.verbose_name_plural "django.db.models.Options.verbose_name_plural")选项，而不是依赖于以英语为中心的回退和有些朴素的确定django通过查看模型的类名执行的详细名称：

```
from django.db import models
from django.utils.translation import ugettext_lazy as _

class MyThing(models.Model):
    name = models.CharField(_('name'), help_text=_('This is the help text'))

    class Meta:
        verbose_name = _('my thing')
        verbose_name_plural = _('my things')

```

#### 模型方法`short_description`

对于模型方法，您可以使用`short_description`属性向Django和管理网站提供翻译：

```
from django.db import models
from django.utils.translation import ugettext_lazy as _

class MyThing(models.Model):
    kind = models.ForeignKey(ThingKind, related_name='kinds',
                             verbose_name=_('kind'))

    def is_mouse(self):
        return self.kind.type == MOUSE_TYPE
    is_mouse.short_description = _('Is it a mouse?')

```

### 使用延迟翻译对象

可以在Python中使用unicode字符串（类型为`unicode`的对象）的任何地方使用`ugettext_lazy()`调用的结果。如果您尝试在预期的字节（`str`对象）使用它，事情将无法正常工作，因为`ugettext_lazy()`对象不知道如何将自身转换为字节。你不能在一个bytestring中使用一个unicode字符串，所以这是正常的Python行为。例如：

```
# This is fine: putting a unicode proxy into a unicode string.
"Hello %s" % ugettext_lazy("people")

# This will not work, since you cannot insert a unicode object
# into a bytestring (nor can you insert our unicode proxy there)
b"Hello %s" % ugettext_lazy("people")

```

如果您看到像`“hello ＆lt； django.utils.functional ...＆gt；”`的输出，将`ugettext_lazy()`的结果转换为字节。这是你的代码中的一个错误。

如果你不喜欢长的`ugettext_lazy`名称，可以将其命名为`_`（下划线），如下所示：

```
from django.db import models
from django.utils.translation import ugettext_lazy as _

class MyThing(models.Model):
    name = models.CharField(help_text=_('This is the help text'))

```

使用`ugettext_lazy()`和`ungettext_lazy()`在模型和效用函数中标记字符串是一种常见的操作。当你在代码中的其他地方使用这些对象时，你应该确保你不会意外地将它们转换为字符串，因为它们应该尽可能晚地转换（以便正确的区域设置生效）。这需要使用下面描述的辅助函数。

#### 懒惰翻译和复数

当对多个字符串（`[u]n[p]gettext_lazy`）使用延迟转换时，通常不知道字符串定义时的`number`参数。因此，您有权将`number`参数传递一个键名称，而不是整数。然后在字符串插值期间，在该键下的字典中查找`number`。这里的例子：

```
from django import forms
from django.utils.translation import ungettext_lazy

class MyForm(forms.Form):
    error_message = ungettext_lazy("You only provided %(num)d argument",
        "You only provided %(num)d arguments", 'num')

    def clean(self):
        # ...
        if error:
            raise forms.ValidationError(self.error_message % {'num': number})

```

如果字符串只包含一个未命名的占位符，则可以直接使用`number`参数进行插值：

```
class MyForm(forms.Form):
    error_message = ungettext_lazy("You provided %d argument",
        "You provided %d arguments")

    def clean(self):
        # ...
        if error:
            raise forms.ValidationError(self.error_message % number)

```

#### 连接字符串：string_concat()

标准的Python字符串连接（`''.join([...])`）不能用于包含延迟翻译对象的列表。相反，您可以使用[`django.utils.translation.string_concat()`](../../ref/utils.html#django.utils.translation.string_concat "django.utils.translation.string_concat")，创建一个延迟对象，连接它的内容_和_仅当结果包含在一个字符串。例如：

```
from django.utils.translation import string_concat
from django.utils.translation import ugettext_lazy
...
name = ugettext_lazy('John Lennon')
instrument = ugettext_lazy('guitar')
result = string_concat(name, ': ', instrument)

```

In this case, the lazy translations in `result` will only be converted to strings when `result` itself is used in a string (usually at template rendering time).

#### 延迟翻译的其他用途

对于任何其他情况下，你想延迟翻译，但必须将可翻译字符串作为参数传递给另一个函数，你可以将这个函数包装在一个懒惰的调用自己。例如：

```
from django.utils import six  # Python 3 compatibility
from django.utils.functional import lazy
from django.utils.safestring import mark_safe
from django.utils.translation import ugettext_lazy as _

mark_safe_lazy = lazy(mark_safe, six.text_type)

```

然后稍后：

```
lazy_string = mark_safe_lazy(_("<p>My <strong>string!</strong></p>"))

```

### 语言的本地化名称

`get_language_info`()[[source]](../../_modules/django/utils/translation.html#get_language_info)

`get_language_info()`函数提供有关语言的详细信息：

```
>>> from django.utils.translation import get_language_info
>>> li = get_language_info('de')
>>> print(li['name'], li['name_local'], li['bidi'])
German Deutsch False

```

字典的`name`和`name_local`属性分别包含英语和语言本身的语言名称。`bidi`属性仅对双向语言为True。

语言信息的来源是`django.conf.locale`模块。类似的访问此信息可用于模板代码。见下文。

## 国际化：在模板代码中

[_Django 模板_](../../ref/templates/language.html) 中的翻译使用两个模板标签，语法与Python 代码中使用的语法有稍许不同。为了让你的模板能够访问这些标签，需要将`{% load i18n %}` 放置在模板的顶部。和所有模板标签一样，这个标签需要在所有使用翻译的模板中加载，即使这些模板扩展自已经加载`i18n` 标签的模板。

### 反式 template tag

`{% trans %}` 模板标签翻译一个常量字符串（位于单引号或双引号中）或变量：

```
<title>{% trans "This is the title." %}</title>
<title>{% trans myvar %}</title>

```

如果带有`noop` 选项，变量的查找仍然继续但是会忽略翻译。这对于需要在未来进行翻译的内容非常有用：

```
<title>{% trans "myvar" noop %}</title>

```

在内部，内联的翻译使用[`ugettext()`](../../ref/utils.html#django.utils.translation.ugettext "django.utils.translation.ugettext") 调用。

当一个模板变量（上面的`myvar`）传递给该标签时， 该标签会首先在运行时刻将变量解析成一个字符串，然后在消息目录中查找这个字符串。

`{% trans %}` 不可以将模板标签嵌入到字符串中。如果你的翻译字符串需要带有变量（占位符），可以使用[`{% blocktrans %}`](#std:templatetag-blocktrans)。

如果你想提前翻译字符串但是不显示出来，你可以使用以下语法：

```
{% trans "This is the title" as the_title %}

<title>{{ the_title }}</title>
<meta name="description" content="{{ the_title }}">

```

实际应用中，你将使用它来获取字符串，然后在多处使用或者作为参数传递给其它模板标签或过滤器：

```
{% trans "starting point" as start %}
{% trans "end point" as end %}
{% trans "La Grande Boucle" as race %}

<h1>
  <a href="/" title="{% blocktrans %}Back to '{{ race }}' homepage{% endblocktrans %}">{{ race }}</a>
</h1>
<p>
{% for stage in tour_stages %}
    {% cycle start end %}: {{ stage }}{% if forloop.counter|divisibleby:2 %}<br />{% else %}, {% endif %}
{% endfor %}
</p>

```

利用`context` 关键字，`{% trans %}` 还支持[_contextual markers_](#contextual-markers)：

```
{% trans "May" context "month name" %}

```

### blocktrans template tag

与[`trans`](#std:templatetag-trans)标签相反，`blocktrans`标签允许您通过使用占位符来标记由文字和可变内容组成的复杂句子进行翻译：

```
{% blocktrans %}This string will have {{ value }} inside.{% endblocktrans %}

```

要翻译模板表达式，例如访问对象属性或使用模板过滤器，您需要将表达式绑定到本地变量，以在翻译块中使用。例子：

```
{% blocktrans with amount=article.price %}
That will cost $ {{ amount }}.
{% endblocktrans %}

{% blocktrans with myvar=value|filter %}
This will have {{ myvar }} inside.
{% endblocktrans %}

```

您可以在单个`blocktrans`标记中使用多个表达式：

```
{% blocktrans with book_t=book|title author_t=author|title %}
This is {{ book_t }} by {{ author_t }}
{% endblocktrans %}

```

注意

The previous more verbose format is still supported: `{% blocktrans with book|title as book_t and author|title as author_t %}`

Other block tags (for example `{% for %}` or `{% if %}`) are not allowed inside a `blocktrans` tag.

如果解析其中一个块参数失败，则通过使用[`deactivate_all()`](../../ref/utils.html#django.utils.translation.deactivate_all "django.utils.translation.deactivate_all")函数暂时停用当前活动的语言，blocktrans将会回退到默认语言。

该标签还提供了复数。使用它：

*   指定并绑定名称为`count`的计数器值。该值将用于选择正确的复数形式。
*   Specify both the singular and plural forms separating them with the `{% plural %}` tag within the `{% blocktrans %}` and `{% endblocktrans %}` tags.

一个例子：

```
{% blocktrans count counter=list|length %}
There is only one {{ name }} object.
{% plural %}
There are {{ counter }} {{ name }} objects.
{% endblocktrans %}

```

一个更复杂的例子：

```
{% blocktrans with amount=article.price count years=i.length %}
That will cost $ {{ amount }} per year.
{% plural %}
That will cost $ {{ amount }} per {{ years }} years.
{% endblocktrans %}

```

除了计数器值之外，当您同时使用复数特性和绑定值到局部变量时，请记住`blocktrans`结构在内部转换为`ungettext`调用。这意味着相同的[_notes regarding ungettext variables_](#pluralization-var-notes)适用。

反向URL查找不能在`blocktrans`中执行，应该预先检索（并存储）：

```
{% url 'path.to.view' arg arg2 as the_url %}
{% blocktrans %}
This is a URL: {{ the_url }}
{% endblocktrans %}

```

`{% blocktrans %}` also supports [_contextual markers_](#contextual-markers) using the `context` keyword:

```
{% blocktrans with name=user.username context "greeting" %}Hi {{ name }}{% endblocktrans %}

```

`{％ blocktrans ％}`支持的另一个功能是`trimmed`选项。此选项将从`{％ blocktrans ％}`的内容的开头和结尾删除换行符。标签，在行的开头和结尾替换任何空格，并使用空格字符将所有行合并为一个，以将它们分隔开。这对缩进`{％ blocktrans ％}`标记而不缩进缩进字符的内容非常有用在PO文件中的相应条目中，这使得翻译过程更容易。

例如，以下`{％ blocktrans ％}`标记：

```
{% blocktrans trimmed %}
  First sentence.
  Second paragraph.
{% endblocktrans %}

```

将导致输入`“第一 句。 第二 PO文件，`“ 第一 句。 第二 &gt;`，如果未指定`trimmed`选项。`

Changed in Django 1.7:

已添加`trimmed`选项。

### 传递给标记和过滤器的字符串文字

您可以使用熟悉的`_()`语法将作为参数传递的字符串文字转换为标记和过滤器：

```
{% some_tag _("Page not found") value|yesno:_("yes,no") %}

```

在这种情况下，标记和过滤器都将看到翻译的字符串，因此他们不需要知道翻译。

注意

在此示例中，转换基础结构将传递字符串`"yes,no"`，而不是单个字符串`"yes"`和`"no"`翻译后的字符串需要包含逗号，以便过滤器解析代码知道如何拆分参数。例如，德语翻译者可以将字符串`"yes,no"`翻译为`"ja,nein"`（保持逗号不变）。

### 模板中翻译员的评论

与[_Python code_](#translator-comments)一样，可以使用注释指定这些笔记的注释，可以使用[`comment`](../../ref/templates/builtins.html#std:templatetag-comment)标签：

```
{% comment %}Translators: View verb{% endcomment %}
{% trans "View" %}

{% comment %}Translators: Short intro blurb{% endcomment %}
<p>{% blocktrans %}A multiline translatable
literal.{% endblocktrans %}</p>

```

或使用`{#` ...`#}`[_one-line comment constructs_](../../ref/templates/language.html#template-comments)：

```

<button type="submit">{% trans "Go" %}</button>

{% blocktrans %}Ambiguous translatable block of text{% endblocktrans %}

```

注意

为了完整性，这些是所得到的`.po`文件的相应片段：

```
#. Translators: View verb
# path/to/template/file.html:10
msgid "View"
msgstr ""

#. Translators: Short intro blurb
# path/to/template/file.html:13
msgid ""
"A multiline translatable"
"literal."
msgstr ""

# ...

#. Translators: Label of a button that triggers search
# path/to/template/file.html:100
msgid "Go"
msgstr ""

#. Translators: This is a text of the base template
# path/to/template/file.html:103
msgid "Ambiguous translatable block of text"
msgstr ""

```

### 在模板中切换语言

如果您要在模板中选择语言，可以使用`language`模板标记：

```
{% load i18n %}

{% get_current_language as LANGUAGE_CODE %}
<!-- Current language: {{ LANGUAGE_CODE }} -->
<p>{% trans "Welcome to our page" %}</p>

{% language 'en' %}
    {% get_current_language as LANGUAGE_CODE %}
    <!-- Current language: {{ LANGUAGE_CODE }} -->
    <p>{% trans "Welcome to our page" %}</p>
{% endlanguage %}

```

虽然第一次出现的“欢迎来到我们的页面”使用当前语言，第二次将始终是英语。

### 其他标签

这些标签还需要`{％ 加载 i18n ％}`。

*   `{％ get_available_languages 为 LANGUAGES ％} 第一个元素是[_language code_](index.html#term-language-code)的元组列表，第二个是语言名称（翻译为当前活动的语言环境）。`
*   `{％ get_current_language 作为 LANGUAGE_CODE ％} 当前用户的首选语言，作为字符串。`示例：`en-us`。（请参阅[_How Django discovers language preference_](#how-django-discovers-language-preference)。）
*   `{％ get_current_language_bidi 为 LANGUAGE_BIDI ％} 当前语言环境的方向。`如果为True，则为从右到左的语言，例如：希伯来语，阿拉伯语。如果为False，则为从左到右的语言，例如：英语，法语，德语等。

If you enable the `django.template.context_processors.i18n` context processor then each `RequestContext` will have access to `LANGUAGES`, `LANGUAGE_CODE`, and `LANGUAGE_BIDI` as defined above.

Changed in Django 1.8:

默认情况下，新项目的`i18n`上下文处理器未启用。

您还可以使用提供的模板标记和过滤器检索有关任何可用语言的信息。要获取有关单一语言的信息，请使用`标签： get_language_info ％}`

```
{% get_language_info for LANGUAGE_CODE as lang %}
{% get_language_info for "pl" as lang %}

```

然后，您可以访问信息：

```
Language code: {{ lang.code }}<br />
Name of language: {{ lang.name_local }}<br />
Name in English: {{ lang.name }}<br />
Bi-directional: {{ lang.bidi }}

```

您还可以使用`{％ get_language_info_list ％}`模板标记来检索语言列表在[`LANGUAGES`](../../ref/settings.html#std:setting-LANGUAGES)中指定的活动语言）。有关如何使用`显示语言选择器的示例，请参阅[_the section about the set_language redirect view_](#set-language-redirect-view)部分{％ get_language_info_list }`。

除了[`LANGUAGES`](../../ref/settings.html#std:setting-LANGUAGES)样式嵌套元组之外，`{％ get_language_info_list ％}`语言代码列表。如果你在你的观点：

```
context = {'available_languages': ['en', 'es', 'fr']}
return render(request, 'mytemplate.html', context)

```

您可以在模板中迭代这些语言：

```
{% get_language_info_list for available_languages as langs %}
{% for lang in langs %} ... {% endfor %}

```

还有简单的过滤器为了方便：

*   `{{ LANGUAGE_CODE | language_name }}`（“德语”）
*   `{{ LANGUAGE_CODE | language_name_local }}`（“Deutsch”）
*   `{{ LANGUAGE_CODE | language_bidi }}`（假）

## 国际化：在JavaScript代码中

将翻译添加到JavaScript会带来一些问题：

*   JavaScript代码无法访问`gettext`实现。
*   JavaScript代码无权访问`.po`或`.mo`文件；它们需要由服务器交付。
*   JavaScript的翻译目录应尽可能小。

Django为这些问题提供了一个集成的解决方案：它将翻译传递给JavaScript，因此您可以在JavaScript中调用`gettext`等。

### 的`javascript_catalog`

`javascript_catalog`(_request_, _domain='djangojs'_, _packages=None_)[[source]](../../_modules/django/views/i18n.html#javascript_catalog)

这些问题的主要解决方案是[`django.views.i18n.javascript_catalog()`](#django.views.i18n.javascript_catalog "django.views.i18n.javascript_catalog")视图，它发送一个JavaScript代码库，其函数模仿`gettext`接口，翻译字符串数组。这些翻译字符串取自应用程序或Django core，根据您在`info_dict`或URL中指定的内容。还包括[`LOCALE_PATHS`](../../ref/settings.html#std:setting-LOCALE_PATHS)中列出的路径。

你这样挂钩：

```
from django.views.i18n import javascript_catalog

js_info_dict = {
    'packages': ('your.app.package',),
}

urlpatterns = [
    url(r'^jsi18n/$', javascript_catalog, js_info_dict),
]

```

`packages`中的每个字符串都应采用Python点分包格式（与[`INSTALLED_APPS`](../../ref/settings.html#std:setting-INSTALLED_APPS)中的字符串格式相同），并且应该指向包含`locale`如果指定多个包，则所有这些目录将合并到一个目录中。如果您的JavaScript使用来自不同应用程序的字符串，这将非常有用。

翻译的优先级是使得稍后在`packages`参数中出现的包比在开始处出现的包具有更高的优先级，这在针对相同文字冲突翻译的情况下是重要的。

默认情况下，视图使用`djangojs` gettext域。这可以通过更改`domain`参数来更改。

您可以通过将包放入URL模式来使视图动态：

```
urlpatterns = [
    url(r'^jsi18n/(?P<packages>\S+?)/$', javascript_catalog),
]

```

这样，您可以将包指定为由URL中的“+”号分隔的包名称列表。如果您的网页使用来自不同应用程式的程式码，且这项变更经常发生，而且您不想提取一个大型目录档案，这项功能就特别实用。作为安全措施，这些值只能是`django.conf`或来自[`INSTALLED_APPS`](../../ref/settings.html#std:setting-INSTALLED_APPS)设置的任何包。

在[`LOCALE_PATHS`](../../ref/settings.html#std:setting-LOCALE_PATHS)设置中列出的路径中找到的JavaScript翻译也始终包括在内。为了与用于Python和模板的翻译查找顺序算法保持一致，[`LOCALE_PATHS`](../../ref/settings.html#std:setting-LOCALE_PATHS)中列出的目录具有最高优先级，首先出现的优先级高于稍后出现的优先级。

### 使用JavaScript翻译目录

要使用目录，只需拉入动态生成的脚本，如下所示：

```
<script type="text/javascript" src="{% url 'django.views.i18n.javascript_catalog' %}"></script>

```

这使用反向URL查找来查找JavaScript目录视图的URL。加载目录后，您的JavaScript代码可以使用以下方法：

*   `gettext`
*   `ngettext`
*   `interpolate`
*   `get_format`
*   `gettext_noop`
*   `pgettext`
*   `npgettext`
*   `pluralidx`

#### gettext

`gettext`函数的行为与您的Python代码中的标准`gettext`接口类似：

```
document.write(gettext('this is to be translated'));

```

#### ngettext

`ngettext`函数提供了一个多元化单词和短语的接口：

```
var object_count = 1 // or 0, or 2, or 3, ...
s = ngettext('literal for the singular case',
        'literal for the plural case', object_count);

```

#### 插

`interpolate`函数支持动态填充格式字符串。插值语法是从Python借用的，因此`interpolate`函数支持位置和命名插值：

*   位置插值：`obj`包含一个JavaScript Array对象，其元素值然后按照它们出现的相同顺序在其相应的`fmt`占位符中顺序插值。例如：

    ```
    fmts = ngettext('There is %s object. Remaining: %s',
            'There are %s objects. Remaining: %s', 11);
    s = interpolate(fmts, [11, 20]);
    // s is 'There are 11 objects. Remaining: 20'

    ```

*   Named interpolation: This mode is selected by passing the optional boolean `named` parameter as `true`. `obj`包含JavaScript对象或关联数组。例如：

    ```
    d = {
        count: 10,
        total: 50
    };

    fmts = ngettext('Total: %(total)s, there is %(count)s object',
    'there are %(count)s of a total of %(total)s objects', d.count);
    s = interpolate(fmts, d, true);

    ```

你不应该使用字符串插值，但是这仍然是JavaScript，所以代码必须重复正则表达式替换。这不像Python中的字符串插值那么快，所以保持它在那些你真正需要它的情况下（例如，与`ngettext`一起产生正确的复数）。

#### get_format

`get_format`函数可以访问配置的i18n格式设置，并可以检索给定设置名称的格式字符串：

```
document.write(get_format('DATE_FORMAT'));
// 'N j, Y'

```

它可以访问以下设置：

*   [日期格式](../../ref/settings.html#std:setting-DATE_FORMAT)
*   [DATE_INPUT_FORMATS](../../ref/settings.html#std:setting-DATE_INPUT_FORMATS)
*   [DATETIME_FORMAT](../../ref/settings.html#std:setting-DATETIME_FORMAT)
*   [DATETIME_INPUT_FORMATS](../../ref/settings.html#std:setting-DATETIME_INPUT_FORMATS)
*   [DECIMAL_SEPARATOR](../../ref/settings.html#std:setting-DECIMAL_SEPARATOR)
*   [FIRST_DAY_OF_WEEK](../../ref/settings.html#std:setting-FIRST_DAY_OF_WEEK)
*   [MONTH_DAY_FORMAT](../../ref/settings.html#std:setting-MONTH_DAY_FORMAT)
*   [NUMBER_GROUPING](../../ref/settings.html#std:setting-NUMBER_GROUPING)
*   [SHORT_DATE_FORMAT](../../ref/settings.html#std:setting-SHORT_DATE_FORMAT)
*   [SHORT_DATETIME_FORMAT](../../ref/settings.html#std:setting-SHORT_DATETIME_FORMAT)
*   [THOUSAND_SEPARATOR](../../ref/settings.html#std:setting-THOUSAND_SEPARATOR)
*   [时间格式](../../ref/settings.html#std:setting-TIME_FORMAT)
*   [TIME_INPUT_FORMATS](../../ref/settings.html#std:setting-TIME_INPUT_FORMATS)
*   [YEAR_MONTH_FORMAT](../../ref/settings.html#std:setting-YEAR_MONTH_FORMAT)

这对于保持与Python呈现的值的格式一致性很有用。

#### gettext_noop

这模拟`gettext`函数，但什么都不做，返回任何传递给它的：

```
document.write(gettext_noop('this will not be translated'));

```

这对于剔除将来需要翻译的代码部分很有用。

#### pgettext

`pgettext`函数的行为类似于Python变体（[`pgettext()`](../../ref/utils.html#django.utils.translation.pgettext "django.utils.translation.pgettext")），提供上下文翻译词：

```
document.write(pgettext('month name', 'May'));

```

#### npgettext

`npgettext`函数的行为类似于Python变体（[`npgettext()`](../../ref/utils.html#django.utils.translation.npgettext "django.utils.translation.npgettext")），提供了一个**多元化**

```
document.write(npgettext('group', 'party', 1));
// party
document.write(npgettext('group', 'party', 2));
// parties

```

#### 复数

`pluralidx`函数以与[`pluralize`](../../ref/templates/builtins.html#std:templatefilter-pluralize)模板过滤器类似的方式工作，确定给定的`count`是否应使用字的复数形式：

```
document.write(pluralidx(0));
// true
document.write(pluralidx(1));
// false
document.write(pluralidx(2));
// true

```

在最简单的情况下，如果不需要自定义复数，则对于所有其他数字，对于整数`1`和`true`返回`false`。

然而，在所有语言中，复数并不是这么简单。如果语言不支持复数，则提供空值。

此外，如果围绕复数有复杂的规则，目录视图将呈现条件表达式。这将评估`true`（应为pluralize）或`false`（应**不** pluralize）值。

### 性能注意事项

[`javascript_catalog()`](#django.views.i18n.javascript_catalog "django.views.i18n.javascript_catalog")视图在每个请求中从`.mo`文件生成目录。由于它的输出是恒定的 - 至少对于给定版本的网站 - 它是缓存的一个很好的候选人。

服务器端缓存将减少CPU负载。它很容易用[`cache_page()`](../cache.html#django.views.decorators.cache.cache_page "django.views.decorators.cache.cache_page")装饰器实现。要在翻译更改时触发缓存无效，请提供版本相关的密钥前缀，如下面的示例所示，或者根据版本相关的网址映射视图。

```
from django.views.decorators.cache import cache_page
from django.views.i18n import javascript_catalog

# The value returned by get_version() must change when translations change.
@cache_page(86400, key_prefix='js18n-%s' % get_version())
def cached_javascript_catalog(request, domain='djangojs', packages=None):
    return javascript_catalog(request, domain, packages)

```

客户端缓存可以节省带宽，并加快网站加载速度。如果您使用的是ETags（[`USE_ETAGS = True`](../../ref/settings.html#std:setting-USE_ETAGS)），则表示您已完成付款。否则，您可以应用[_conditional decorators_](../conditional-view-processing.html#conditional-decorators)。在以下示例中，无论何时重新启动应用程序服务器，缓存都将失效。

```
from django.utils import timezone
from django.views.decorators.http import last_modified
from django.views.i18n import javascript_catalog

last_modified_date = timezone.now()

@last_modified(lambda req, **kw: last_modified_date)
def cached_javascript_catalog(request, domain='djangojs', packages=None):
    return javascript_catalog(request, domain, packages)

```

您甚至可以预先生成JavaScript目录作为部署过程的一部分，并将其作为静态文件提供。这种激进的技术在[django-statici18n](http://django-statici18n.readthedocs.org/en/latest/)中实现。

## 国际化：在URL模式中

Django提供了两种机制来国际化URL模式：

*   将语言前缀添加到网址格式的根目录，使[`LocaleMiddleware`](../../ref/middleware.html#django.middleware.locale.LocaleMiddleware "django.middleware.locale.LocaleMiddleware")可以从请求的网址中检测要激活的语言。
*   使网址模式可通过[`django.utils.translation.ugettext_lazy()`](../../ref/utils.html#django.utils.translation.ugettext_lazy "django.utils.translation.ugettext_lazy")函数进行翻译。

警告

使用这些功能之一需要为每个请求设置活动语言；换句话说，您需要在[`MIDDLEWARE_CLASSES`](../../ref/settings.html#std:setting-MIDDLEWARE_CLASSES)设置中设置[`django.middleware.locale.LocaleMiddleware`](../../ref/middleware.html#django.middleware.locale.LocaleMiddleware "django.middleware.locale.LocaleMiddleware")。

### URL模式中的语言前缀

`i18n_patterns`(_prefix_, _pattern_description_, _..._)[[source]](../../_modules/django/conf/urls/i18n.html#i18n_patterns)

自1.8版起已弃用：`i18n_patterns()`的`prefix`参数已被弃用，不会在Django 2.0中受支持。只需传递[`django.conf.urls.url()`](../../ref/urls.html#django.conf.urls.url "django.conf.urls.url")实例的列表即可。

此函数可以在根URLconf中使用，Django会自动将当前活动语言代码添加到[`i18n_patterns()`](#django.conf.urls.i18n.i18n_patterns "django.conf.urls.i18n.i18n_patterns")中定义的所有网址模式。网址格式示例：

```
from django.conf.urls import include, url
from django.conf.urls.i18n import i18n_patterns

from about import views as about_views
from news import views as news_views
from sitemap.views import sitemap

urlpatterns = [
    url(r'^sitemap\.xml$', sitemap, name='sitemap_xml'),
]

news_patterns = [
    url(r'^$', news_views.index, name='index'),
    url(r'^category/(?P<slug>[\w-]+)/$', news_views.category, name='category'),
    url(r'^(?P<slug>[\w-]+)/$', news_views.details, name='detail'),
]

urlpatterns += i18n_patterns(
    url(r'^about/$', about_views.main, name='about'),
    url(r'^news/', include(news_patterns, namespace='news')),
)

```

定义这些网址格式后，Django会自动将语言前缀添加到由`i18n_patterns`函数添加的网址格式。例：

```
from django.core.urlresolvers import reverse
from django.utils.translation import activate

>>> activate('en')
>>> reverse('sitemap_xml')
'/sitemap.xml'
>>> reverse('news:index')
'/en/news/'

>>> activate('nl')
>>> reverse('news:detail', kwargs={'slug': 'news-slug'})
'/nl/news/news-slug/'

```

警告

[`i18n_patterns()`](#django.conf.urls.i18n.i18n_patterns "django.conf.urls.i18n.i18n_patterns")只能在根URLconf中使用。在包含的URLconf中使用它会引发[`ImproperlyConfigured`](../../ref/exceptions.html#django.core.exceptions.ImproperlyConfigured "django.core.exceptions.ImproperlyConfigured")异常。

警告

确保您没有可能与自动添加的语言前缀相冲突的非前缀网址格式。

### 翻译网址格式

网址格式也可以使用[`ugettext_lazy()`](../../ref/utils.html#django.utils.translation.ugettext_lazy "django.utils.translation.ugettext_lazy")函数标记为可翻译。例：

```
from django.conf.urls import include, url
from django.conf.urls.i18n import i18n_patterns
from django.utils.translation import ugettext_lazy as _

from about import views as about_views
from news import views as news_views
from sitemaps.views import sitemap

urlpatterns = [
    url(r'^sitemap\.xml$', sitemap, name='sitemap_xml'),
]

news_patterns = [
    url(r'^$', news_views.index, name='index'),
    url(_(r'^category/(?P<slug>[\w-]+)/$'), news_views.category, name='category'),
    url(r'^(?P<slug>[\w-]+)/$', news_views.details, name='detail'),
]

urlpatterns += i18n_patterns(
    url(_(r'^about/$'), about_views.main, name='about'),
    url(_(r'^news/'), include(news_patterns, namespace='news')),
)

```

创建翻译后，[`reverse()`](../../ref/urlresolvers.html#django.core.urlresolvers.reverse "django.core.urlresolvers.reverse")函数将返回活动语言的URL。例：

```
from django.core.urlresolvers import reverse
from django.utils.translation import activate

>>> activate('en')
>>> reverse('news:category', kwargs={'slug': 'recent'})
'/en/news/category/recent/'

>>> activate('nl')
>>> reverse('news:category', kwargs={'slug': 'recent'})
'/nl/nieuws/categorie/recent/'

```

警告

在大多数情况下，最好只在语言代码前缀的模式块中使用已翻译的URL（使用[`i18n_patterns()`](#django.conf.urls.i18n.i18n_patterns "django.conf.urls.i18n.i18n_patterns")），以避免粗心大意的翻译URL导致与非翻译的网址格式。

### 在模板中反转

如果在模板中反转本地化URL，它们总是使用当前语言。要链接到其他语言的网址，请使用[`language`](#std:templatetag-language)模板标记。它在所包含的模板部分中启用给定的语言：

```
{% load i18n %}

{% get_available_languages as languages %}

{% trans "View this category in:" %}
{% for lang_code, lang_name in languages %}
    {% language lang_code %}
    <a href="{% url 'category' slug=category.slug %}">{{ lang_name }}</a>
    {% endlanguage %}
{% endfor %}

```

[`language`](#std:templatetag-language)标签需要将语言代码作为唯一的参数。

## 本地化：如何创建语言文件

一旦应用程序的字符串文字已经标记为以后翻译，翻译本身需要被写入（或获得）。这是如何工作。

### 消息文件

第一步是为新语言创建[_message file_](index.html#term-message-file)。消息文件是纯文本文件，表示单一语言，包含所有可用的翻译字符串以及如何以给定语言表示它们。消息文件具有`.po`文件扩展名。

Django提供了一个工具，[`django-admin makemessages`](../../ref/django-admin.html#django-admin-makemessages)，自动创建和维护这些文件。

Gettext实用程序

The `makemessages` command (and `compilemessages` discussed later) use commands from the GNU gettext toolset: `xgettext`, `msgfmt`, `msgmerge` and `msguniq`.

支持的`gettext`实用程序的最低版本为0.15。

要创建或更新消息文件，请运行以下命令：

```
django-admin makemessages -l de

```

...其中`de`是您要创建的消息文件的[_locale name_](index.html#term-locale-name)。例如，巴西葡萄牙语为`pt_BR`，奥地利德语为`de_AT`，印尼语为`id`。

脚本应该从两个地方之一运行：

*   Django项目的根目录（包含`manage.py`的目录）。
*   您的一个Django应用程序的根目录。

脚本运行在项目源代码树或应用程序源代码树中，并拉出所有标记为翻译的字符串（请参阅[_How Django discovers translations_](#how-django-discovers-translations)，并确保[`LOCALE_PATHS`](../../ref/settings.html#std:setting-LOCALE_PATHS)已正确配置）。它在目录`locale/LANG/LC_MESSAGES`中创建（或更新）消息文件。在`de`示例中，文件将为`locale/de/LC_MESSAGES/django.po`。

Changed in Django 1.7:

当从项目的根目录运行`makemessages`时，提取的字符串将自动分发到正确的消息文件。也就是说，从包含`locale`目录的应用的文件中提取的字符串将放在该目录下的消息文件中。从没有任何`locale`目录的应用的文件中提取的字符串将进入[`LOCALE_PATHS`](../../ref/settings.html#std:setting-LOCALE_PATHS)中列出的目录下的消息文件，否则会生成错误，如果[`LOCALE_PATHS`](../../ref/settings.html#std:setting-LOCALE_PATHS)为空。

默认情况下，[`django-admin makemessages`](../../ref/django-admin.html#django-admin-makemessages)检查包含`.html`或`.txt`如果要覆盖该默认值，请使用`--extension`或`-e`选项指定要检查的文件扩展名：

```
django-admin makemessages -l de -e txt

```

使用逗号分隔多个扩展程序和/或使用`-e`或`--extension`多次：

```
django-admin makemessages -l de -e html,txt -e xml

```

警告

当[_creating message files from JavaScript source code_](#creating-message-files-from-js-code)创建消息文件时，您需要使用特殊的'djangojs'域，**而不是** `-e js `。

使用Jinja2模板？

[`makemessages`](../../ref/django-admin.html#django-admin-makemessages)不能理解Jinja2模板的语法。要从包含Jinja2模板的项目中提取字符串，请改用[Babel](http://babel.pocoo.org/)。

下面是一个示例`babel.cfg`的配置文件：

```
# Extraction from Python source files
[python: **.py]

# Extraction from Jinja2 templates
[jinja2: **.jinja]
extensions = jinja2.ext.with_

```

请确保您列出了您使用的所有扩展程序！否则Babel将无法识别这些扩展定义的标签，并会忽略包含它们的Jinja2模板。

Babel提供与[`makemessages`](../../ref/django-admin.html#django-admin-makemessages)类似的功能，可以替换它，而不依赖于`gettext`。有关详细信息，请阅读有关[处理消息目录](http://babel.pocoo.org/docs/messages/)的文档。

没有gettext？

如果您没有安装`gettext`实用程序，[`makemessages`](../../ref/django-admin.html#django-admin-makemessages)将创建空文件。如果是这种情况，请安装`gettext`实用程序，或只复制英文消息文件（`locale/en/LC_MESSAGES/django.po`）点；它只是一个空的翻译文件。

在Windows上工作？

如果您使用Windows并需要安装GNU gettext实用程序，因此[`makemessages`](../../ref/django-admin.html#django-admin-makemessages)可以工作，有关详细信息，请参阅[_gettext on Windows_](#gettext-on-windows)。

`.po`文件的格式很简单。每个`.po`文件包含一小部分元数据，例如翻译维护者的联系信息，但文件的大部分是**消息**的列表 - 翻译字符串之间的简单映射和特定语言的实际翻译文本。

例如，如果您的Django应用程序包含文本`“欢迎 到 我的 网站的翻译字符串” t4&gt;`，像这样：

```
_("Welcome to my site.")

```

... then [`django-admin makemessages`](../../ref/django-admin.html#django-admin-makemessages)将创建一个包含以下代码片段的`.po`文件：

```
#: path/to/python/module.py:23
msgid "Welcome to my site."
msgstr ""

```

快速解释：

*   `msgid`是翻译字符串，它显示在源中。不要改变它。
*   `msgstr`是您放置语言特定翻译的位置。它开始是空的，所以你有责任改变它。确保您在翻译周围保留引号。
*   为了方便，每个消息以以`#`为前缀并位于`msgid`之上的注释行的形式包括翻译字符串所从的文件名和行号收集。

长消息是一种特殊情况。在那里，紧跟`msgstr`（或`msgid`）之后的第一个字符串是空字符串。然后内容本身将被写在下几行作为每行一个字符串。这些字符串是直接连接的。不要忘记字符串中的尾随空格；否则，它们将被粘在一起，没有空格！

注意你的字符集

由于`gettext`工具的内部工作方式，因为我们要在Django的核心和应用程序中允许非ASCII源字符串，您**必须**使用UTF-8作为编码为您的PO文件（创建PO文件时的默认值）。这意味着每个人都将使用相同的编码，这在Django处理PO文件时很重要。

要对所有源代码和模板重新审核新的翻译字符串，并更新所有**所有**语言的邮件文件，请运行：

```
django-admin makemessages -a

```

### 编译消息文件

创建消息文件后 - 每次对其进行更改时，都需要将其编译为更有效的形式，以供`gettext`使用。使用[`django-admin compilemessages`](../../ref/django-admin.html#django-admin-compilemessages)实用程序执行此操作。

此工具运行所有可用的`.po`文件，并创建`.mo`文件，这是优化为`gettext`使用的二进制文件。在运行[`django-admin makemessages`](../../ref/django-admin.html#django-admin-makemessages)的同一目录中，运行[`django-admin compilemessages`](../../ref/django-admin.html#django-admin-compilemessages)像这样：

```
django-admin compilemessages

```

而已。您的翻译已准备就绪。

在Windows上工作？

If you’re using Windows and need to install the GNU gettext utilities so [`django-admin compilemessages`](../../ref/django-admin.html#django-admin-compilemessages) works see [_gettext on Windows_](#gettext-on-windows) for more information.

.po文件：编码和BOM使用。

Django只支持以UTF-8编码并且没有任何BOM（字节顺序标记）的`.po`文件，因此如果您的文本编辑器在默认情况下将这样的标记添加到文件的开头，则需要重新配置它。

### 从JavaScript源代码创建消息文件

您可以使用[`django-admin makemessages`](../../ref/django-admin.html#django-admin-makemessages)工具，以与其他Django邮件文件相同的方式创建和更新邮件文件。唯一的区别是，您需要通过提供`-d 来明确指定在gettext中的内容是什么，在这种情况下为`djangojs` djangojs`参数，如下所示：

```
django-admin makemessages -d djangojs -l de

```

这将创建或更新用于德语的JavaScript的消息文件。更新邮件文件后，只需以与对普通Django邮件文件相同的方式运行[`django-admin compilemessages`](../../ref/django-admin.html#django-admin-compilemessages)即可。

### gettext on Windows

这只适用于要提取消息ID或编译消息文件（`.po`）的人员。翻译工作本身只涉及编辑此类型的现有文件，但如果要创建自己的消息文件，或者想要测试或编译更改的消息文件，则需要使用`gettext`实用程序：

*   从GNOME服务器下载以下zip文件[https://download.gnome.org/binaries/win32/dependencies/](https://download.gnome.org/binaries/win32/dependencies/)

    *   `gettext-runtime-X.zip`
    *   `gettext-tools-X.zip`

    `X`是版本号，我们需要`0.15`或更高版本。

*   将两个文件中的`bin`目录的内容提取到系统上相同的文件夹（即`C：Program Filesgettext-utils / t2&gt;）`

*   更新系统路径：

    *   `Control Panel &gt; System &gt; Advanced &gt; Environment Variables`.
    *   在`系统 变量`列表中，单击`Path`，单击`Edit`。
    *   在`变量 值结束时添加`； C：Program Filesgettext-utilsin / t5&gt;``字段。

只要`xgettext - version`命令正常工作，您还可以使用您在其他地方获得的`gettext`二进制。如果在命令`xgettext - version`中输入命令，则不要尝试使用`gettext` Windows命令提示符导致弹出窗口说“xgettext.exe生成错误，将被Windows关闭”。

### 自定义`makemessages`

如果要向`xgettext`传递附加参数，则需要创建自定义[`makemessages`](../../ref/django-admin.html#django-admin-makemessages)命令并覆盖其`xgettext_options`属性：

```
from django.core.management.commands import makemessages

class Command(makemessages.Command):
    xgettext_options = makemessages.Command.xgettext_options + ['--keyword=mytrans']

```

如果您需要更多灵活性，还可以向自定义[`makemessages`](../../ref/django-admin.html#django-admin-makemessages)命令中添加一个新参数：

```
from django.core.management.commands import makemessages

class Command(makemessages.Command):

    def add_arguments(self, parser):
        super(Command, self).add_arguments(parser)
        parser.add_argument('--extra-keyword', dest='xgettext_keywords',
                            action='append')

    def handle(self, *args, **options):
        xgettext_keywords = options.pop('xgettext_keywords')
        if xgettext_keywords:
            self.xgettext_options = (
                makemessages.Command.xgettext_options[:] +
                ['--keyword=%s' % kwd for kwd in xgettext_keywords]
            )
        super(Command, self).handle(*args, **options)

```

## 杂

### 的`set_language`

`set_language`(_request_)[[source]](../../_modules/django/views/i18n.html#set_language)

方便起见, Django自带了一个, [`django.views.i18n.set_language()`](#django.views.i18n.set_language "django.views.i18n.set_language")视图, 作用是设置用户语言偏好并重定向返回到前一页面

在URLconf中加入下面这行代码来激活这个视图：

```
url(r'^i18n/', include('django.conf.urls.i18n')),

```

(注意这个例子使得这个视图在`/i18n/setlang/`中有效.)

警告

请确保您不要在[`i18n_patterns()`](#django.conf.urls.i18n.i18n_patterns "django.conf.urls.i18n.i18n_patterns")中包含上述网址 - 它需要与语言无关才能正常工作。

该视图需要通过`POST`方法调用，并在请求中设置`language`参数。如果启用了会话支持，则视图会在用户的会话中保存语言选择。否则，它会将语言选项保存在默认名为`django_language`的Cookie中。（名称可以通过[`LANGUAGE_COOKIE_NAME`](../../ref/settings.html#std:setting-LANGUAGE_COOKIE_NAME)设置更改。）

设置语言选择后，Django重定向用户，遵循此算法：

*   Django在`POST`数据中查找`next`参数。
*   如果不存在或为空，Django会尝试`Referrer`标头中的网址。
*   如果这是空的 - 例如，如果用户的浏览器禁止该标题 - 那么用户将被重定向到`/`（网站根）作为后备。

这里是HTML模板代码的示例：

```
{% load i18n %}
<form action="{% url 'set_language' %}" method="post">
{% csrf_token %}
<input name="next" type="hidden" value="{{ redirect_to }}" />
<select name="language">
{% get_current_language as LANGUAGE_CODE %}
{% get_available_languages as LANGUAGES %}
{% get_language_info_list for LANGUAGES as languages %}
{% for language in languages %}
<option value="{{ language.code }}"{% if language.code == LANGUAGE_CODE %} selected="selected"{% endif %}>
    {{ language.name_local }} ({{ language.code }})
</option>
{% endfor %}
</select>
<input type="submit" value="Go" />
</form>

```

在此示例中，Django在`redirect_to`上下文变量中查找用户将被重定向到的网页的网址。

### 明确设定使用语言

您可能需要显式设置当前会话的活动语言。例如，可能从另一个系统检索用户的语言偏好。您已介绍过[`django.utils.translation.activate()`](../../ref/utils.html#django.utils.translation.activate "django.utils.translation.activate")。这只适用于当前线程。要为整个会话持久保存语言，还需修改会话中的[`LANGUAGE_SESSION_KEY`](../../ref/utils.html#django.utils.translation.LANGUAGE_SESSION_KEY "django.utils.translation.LANGUAGE_SESSION_KEY")：

```
from django.utils import translation
user_language = 'fr'
translation.activate(user_language)
request.session[translation.LANGUAGE_SESSION_KEY] = user_language

```

您通常希望同时使用：[`django.utils.translation.activate()`](../../ref/utils.html#django.utils.translation.activate "django.utils.translation.activate")将更改此线程的语言，并修改会话使此首选项在未来的请求中保持不变。

如果您没有使用会话，该语言将保留在Cookie中，其名称在[`LANGUAGE_COOKIE_NAME`](../../ref/settings.html#std:setting-LANGUAGE_COOKIE_NAME)中配置。例如：

```
from django.utils import translation
from django import http
from django.conf import settings
user_language = 'fr'
translation.activate(user_language)
response = http.HttpResponse(...)
response.set_cookie(settings.LANGUAGE_COOKIE_NAME, user_language)

```

### 使用视图和模板外部的翻译

虽然Django提供了一组丰富的i18n工具以用于视图和模板，但它并不限制对Django特定代码的使用。Django翻译机制可以用于将任意文本翻译成Django支持的任何语言（当然，只要存在适当的翻译目录）。您可以加载翻译目录，将其激活并将文本翻译为您选择的语言，但请记住切换回原始语言，因为激活翻译目录是基于每个线程完成的，这样的更改将影响在同一个线程中运行的代码。

例如：

```
from django.utils import translation

def welcome_translated(language):
    cur_language = translation.get_language()
    try:
        translation.activate(language)
        text = translation.ugettext('welcome')
    finally:
        translation.activate(cur_language)
    return text

```

无论[`LANGUAGE_CODE`](../../ref/settings.html#std:setting-LANGUAGE_CODE)和中间件设置的语言如何，使用值“de”调用此函数将给予`"Willkommen"`

特别感兴趣的函数是`django.utils.translation.get_language()`，它返回当前线程中使用的语言`django.utils.translation.activate()`用于当前线程的翻译目录，以及用于检查Django是否支持给定语言的`django.utils.translation.check_for_language()`。

为了帮助编写更简洁的代码，还有一个上下文管理器`django.utils.translation.override()`，用于在输入时存储当前语言，并在退出时恢复。有了它，上面的例子变成：

```
from django.utils import translation

def welcome_translated(language):
    with translation.override(language):
        return translation.ugettext('welcome')

```

### 语言cookie

可以使用多种设置来调整语言Cookie选项：

*   [LANGUAGE_COOKIE_NAME](../../ref/settings.html#std:setting-LANGUAGE_COOKIE_NAME)

New in Django 1.7.

*   [LANGUAGE_COOKIE_AGE](../../ref/settings.html#std:setting-LANGUAGE_COOKIE_AGE)
*   [LANGUAGE_COOKIE_DOMAIN](../../ref/settings.html#std:setting-LANGUAGE_COOKIE_DOMAIN)
*   [LANGUAGE_COOKIE_PATH](../../ref/settings.html#std:setting-LANGUAGE_COOKIE_PATH)

## 实现细节

### Django 翻译的特点

Django 的翻译机制使用Python 自带的`gettext` 模块。如果你了解`gettext`，你应该注意到Django 翻译方式的这些特点：

*   字符串域是`django` 或`djangojs`。这个字符串域用于区分不同的而程序，这些程序将它们的数据保存在共同的消息文件库中（通常是`/usr/share/locale/`）。`django` 域用于Python 和模板的翻译字符串，而且还会加载到全局翻译目录中。`djangojs` 域只用于JavaScript 的翻译目录，以确保它尽可能的小。
*   Django 没有单独使用`xgettext`。它使用Python 对`xgettext` 和 `msgfmt` 进行的封装。最主要是为了方便。

### Django 如何发现语言偏好

一旦你准备好翻译，或者你只是想使用Django 自带的翻译，你需要为你的应用启用翻译。

在后台，Django 有一个非常灵活的模型决定应该使用哪种语言 —— 所有用户还是特定的用户，或者两种都可以。

要设置安装范围的语言首选项，请设置[`LANGUAGE_CODE`](../../ref/settings.html#std:setting-LANGUAGE_CODE)。Django使用这种语言作为默认翻译 - 如果没有找到更好的匹配翻译通过场所中间件（见下文）使用的方法之一的最终尝试。

如果你想要的是用你的母语运行Django，你只需设置[`LANGUAGE_CODE`](../../ref/settings.html#std:setting-LANGUAGE_CODE)并确保相应的[_message files_](index.html#term-message-file)及其编译版本（`.mo`）。

如果你想让每个用户指定首选语言，那么你还要 使用 `LocaleMiddleware`. `LocaleMiddleware` 会打开基于请求数据的语言选择。它为每个人用户的内容进行个性化。

要使用`LocaleMiddleware`，请将`'django.middleware.locale.LocaleMiddleware'`添加到您的[`MIDDLEWARE_CLASSES`](../../ref/settings.html#std:setting-MIDDLEWARE_CLASSES)设置。由于中间件顺序很重要，因此您应该遵循以下准则：

*   确保它是安装的第一个中间件之一。
*   它应该在`SessionMiddleware`之后，因为`LocaleMiddleware`使用会话数据。它应该在`CommonMiddleware`之前，因为`CommonMiddleware`需要一个激活的语言才能解析请求的URL。
*   如果使用`CacheMiddleware`，请在其后放置`LocaleMiddleware`。

例如，你的[`MIDDLEWARE_CLASSES`](../../ref/settings.html#std:setting-MIDDLEWARE_CLASSES)也许看起来像这样子

```
MIDDLEWARE_CLASSES = (
   'django.contrib.sessions.middleware.SessionMiddleware',
   'django.middleware.locale.LocaleMiddleware',
   'django.middleware.common.CommonMiddleware',
)

```

（有关中间件的详情，请参阅[_middleware documentation_](../http/middleware.html)。）

`LocaleMiddleware`尝试通过以下算法确定用户的语言首选项：

*   首先，它在请求的URL中查找语言前缀。这仅在您在根URLconf中使用`i18n_patterns`函数时才会执行。有关语言前缀以及如何将网址格式国际化的详细信息，请参见[_Internationalization: in URL patterns_](#url-internationalization)中。

*   否则，它会在当前用户的会话中查找[`LANGUAGE_SESSION_KEY`](../../ref/utils.html#django.utils.translation.LANGUAGE_SESSION_KEY "django.utils.translation.LANGUAGE_SESSION_KEY")键。

    Changed in Django 1.7:

    在以前的版本中，键名为`django_language`，并且`LANGUAGE_SESSION_KEY`常数不存在。

*   如果没有，它寻找一个cookie。

    所使用的Cookie名称由[`LANGUAGE_COOKIE_NAME`](../../ref/settings.html#std:setting-LANGUAGE_COOKIE_NAME)设置设置。（默认名称为`django_language`。）

*   如果没有，它会查看`Accept-Language` HTTP标头。此标题由您的浏览器发送，并按优先级顺序告诉服务器您喜欢哪种语言。Django尝试标题中的每种语言，直到找到一个可用的翻译。

*   否则，它使用全局[`LANGUAGE_CODE`](../../ref/settings.html#std:setting-LANGUAGE_CODE)设置。

笔记：

*   在每个位置，语言首选项应为标准[_language format_](index.html#term-language-code)，作为字符串。例如，巴西葡萄牙语是`pt-br`。

*   如果基本语言可用，但指定的子语言不是，Django使用基本语言。例如，如果用户指定`de-at`（奥地利德语），但Django只有`de`可用，则Django使用`de`。

*   只能选择[`LANGUAGES`](../../ref/settings.html#std:setting-LANGUAGES)设置中列出的语言。如果要将语言选择限制为所提供语言的一个子集（因为您的应用程序不提供所有这些语言），请将[`LANGUAGES`](../../ref/settings.html#std:setting-LANGUAGES)设置为语言列表。例如：

    ```
    LANGUAGES = (
      ('de', _('German')),
      ('en', _('English')),
    )

    ```

    此示例将可用于自动选择的语言限制为德语和英语（以及任何子语言，如de-ch或en-us）。

*   如果您定义自定义[`LANGUAGES`](../../ref/settings.html#std:setting-LANGUAGES)设置（如上一个项目符号所述），则可以将语言名称标记为翻译字符串 - 但使用[`ugettext_lazy()`](../../ref/utils.html#django.utils.translation.ugettext_lazy "django.utils.translation.ugettext_lazy")而不是[`ugettext()`](../../ref/utils.html#django.utils.translation.ugettext "django.utils.translation.ugettext")以避免循环导入。

    下面是一个示例设置文件：

    ```
    from django.utils.translation import ugettext_lazy as _

    LANGUAGES = (
        ('de', _('German')),
        ('en', _('English')),
    )

    ```

一旦`LocaleMiddleware`确定用户的首选项，就会将此首选项设置为`request.`每个[`HttpRequest`](../../ref/request-response.html#django.http.HttpRequest "django.http.HttpRequest")的LANGUAGE_CODE。随意在您的视图代码中读取此值。这里有一个简单的例子：

```
from django.http import HttpResponse

def hello_world(request, count):
    if request.LANGUAGE_CODE == 'de-at':
        return HttpResponse("You prefer to read Austrian German.")
    else:
        return HttpResponse("You prefer to read another language.")

```

请注意，使用静态（无中间件）翻译时，语言位于`settings.`LANGUAGE_CODE，而在动态（中间件）翻译时，它在`request.`LANGUAGE_CODE。

### Django是如何找到翻译文件的

在运行时，Django构建一个内存中的文字 - 翻译目录。为了实现这一点，它通过遵循该算法关于其检查不同文件路径以加载编译的[_message files_](index.html#term-message-file)（`.mo`）和优先级的顺序来寻找翻译多个翻译为同一个字面量：

1.  [`LOCALE_PATHS`](../../ref/settings.html#std:setting-LOCALE_PATHS)中列出的目录具有最高优先级，首先出现的优先级高于稍后出现的优先级。
2.  然后，它会查找并使用[`INSTALLED_APPS`](../../ref/settings.html#std:setting-INSTALLED_APPS)中列出的每个已安装应用程序中是否存在`locale`目录。首先出现的优先级高于稍后出现的优先级。
3.  最后，在`django/conf/locale`中提供的Django提供的基本翻译用作后备。

也可以看看

JavaScript资源中包含的文字的翻译是根据类似但不完全相同的算法查找的。有关详细信息，请参阅[_javascript_catalog view documentation_](#javascript-catalog-view)。

在所有情况下，包含翻译的目录的名称应使用[_locale name_](index.html#term-locale-name)符号命名。例如。 `de`，`pt_BR`，`es_AR`等。

这样，您可以编写包含自己的翻译的应用程序，并且可以覆盖项目中的基本翻译。或者，您可以从几个应用程序构建一个大项目，并将所有翻译成一个大的通用消息文件，特定于您正在撰写的项目。这是你的选择。

所有消息文件存储库的结构都是相同的。他们是：

*   搜索设置文件中[`LOCALE_PATHS`](../../ref/settings.html#std:setting-LOCALE_PATHS)中列出的所有路径`&lt;language&gt;/LC_MESSAGES/django.(`po | mo）
*   `$APPPATH/locale/&lt;language&gt;/LC_MESSAGES/django.(`po | mo）
*   `$PYTHONPATH/django/conf/locale/&lt;language&gt;/LC_MESSAGES/django.(`po | mo）

要创建邮件文件，请使用[`django-admin makemessages`](../../ref/django-admin.html#django-admin-makemessages)工具。And you use [`django-admin compilemessages`](../../ref/django-admin.html#django-admin-compilemessages) to produce the binary `.mo` files that are used by `gettext`.

您还可以运行[`django-admin compilemessages --settings=path.to.settings`](../../ref/django-admin.html#django-admin-compilemessages)您的[`LOCALE_PATHS`](../../ref/settings.html#std:setting-LOCALE_PATHS)设置中的所有目录。

{% endraw %}