# 格式本地化

## 概览

Django的格式化系统可以在模板中使用当前[_地区_](index.html#term-locale-name)特定的格式，来展示日期、时间和数字。也可以处理表单中输入的本地化。

当它被开启时，访问相同内容的两个用户可能会看到以不同方式格式化的日期、时间和数字，这取决于它们的当前地区的格式。

格式化系统默认是禁用的。需要在你的设置文件中设置[`USE_L10N = True`](../../ref/settings.html#std:setting-USE_L10N)来启用它。

注意

为了方便起见，[`django-admin startproject创建的默认的`settings.py`文件包含了&nbsp;`](../../ref/django-admin.html#django-admin-startproject)`[`USE_L10N = True`](../../ref/settings.html#std:setting-USE_L10N) 的设置。`但是要注意，要开启千位分隔符的数字格式化，你需要在你的设置文件中设置[`USE_THOUSAND_SEPARATOR = True`](../../ref/settings.html#std:setting-USE_THOUSAND_SEPARATOR)。或者，你也可以在你的模板中使用[`intcomma`](../../ref/contrib/humanize.html#std:templatefilter-intcomma)来格式化数字。

注意

[`USE_I18N`](../../ref/settings.html#std:setting-USE_I18N) 是另一个独立的并且相关的设置，它控制着Django是否应该开启翻译。详见[_翻译_](translation.html)。

## 表单中的本地化识别输入

格式化开启之后，Django可以在表单中使用本地化格式来解析日期、时间和数字。也就是说，在表单上输入时，它会尝试不同的格式和地区来猜测用户使用的格式。

注意

Django对于展示数据，使用和解析数据不同的格式。尤其是，解析日期的格式不能使用`%a`（星期名称的缩写），`%A` （星期名称的全称），`%b` （月份名称的缩写），&nbsp;`%B`（月份名称的全称），或者`%p`（上午/下午）。

只是使用`localize`参数，就能开启表单字段的本地化输入和输出：

```
class CashRegisterForm(forms.Form):
   product = forms.CharField()
   revenue = forms.DecimalField(max_digits=4, decimal_places=2, localize=True)

```

## 在模板中控制本地化

当你使用[`USE_L10N`](../../ref/settings.html#std:setting-USE_L10N)来开启格式化的时候，Django会尝试使用地区特定的格式，无论值在模板的什么位置输出。

然而，这对于本地化的值不可能总是十分合适，如果你在输出JavaScript或者机器阅读的XML，你会想要使用去本地化的值。你也可能想只在特定的模板中使用本地化，而不是任何位置都使用。

DJango提供了`l10n`模板库，包含以下标签和过滤器，来实现对本地化的精细控制。

### 模板标签

#### localize

在包含的代码块内开启或关闭模板变量的本地化。

这个标签可以对本地化进行比[`USE_L10N`](../../ref/settings.html#std:setting-USE_L10N)更加精细的操作。

这样做来为一个模板激活或禁用本地化：

```
{% load l10n %}

{% localize on %}
    {{ value }}
{% endlocalize %}

{% localize off %}
    {{ value }}
{% endlocalize %}

```

注意

在&nbsp;`{% localize %}`代码块内并不遵循f [`USE_L10N`](../../ref/settings.html#std:setting-USE_L10N)的值。

对于在每个变量基础上执行相同工作的模板过滤器，参见[`localize`](#std:templatefilter-localize) 和 [`unlocalize`](#std:templatefilter-unlocalize)。

### 模板过滤器

#### localize

强制单一值的本地化。

例如：

```
{% load l10n %}

{{ value|localize }}

```

使用[`unlocalize`](#std:templatefilter-unlocalize)来在单一值上禁用本地化。使用[`localize`](#std:templatetag-localize) 模板标签来在大块的模板区域内控制本地化。

#### unlocalize

强制单一值不带本地化输出。

例如：

```
{% load l10n %}

{{ value|unlocalize }}

```

使用[`localize`](#std:templatefilter-localize)来强制单一值的本地化。使用[`localize`](#std:templatetag-localize)模板标签来在大块的模板区域内控制本地化。

## 创建自定义的格式文件

Django为许多地区提供了格式定义，但是有时你可能想要创建你自己的格式，因为你的的确并没有现成的格式文件，或者你想要覆写其中的一些值。

Changed in Django 1.8:

添加了指定[`FORMAT_MODULE_PATH`](../../ref/settings.html#std:setting-FORMAT_MODULE_PATH)为列表的功能。之前只支持单一的字符串值。

指定你首先放置格式文件的位置来使用自定义格式。把你的[`FORMAT_MODULE_PATH`](../../ref/settings.html#std:setting-FORMAT_MODULE_PATH)设置设置为格式文件存在的包名来使用它，例如：

```
FORMAT_MODULE_PATH = [
    'mysite.formats',
    'some_app.formats',
]

```

文件并不直接放在这个目录中，而是放在和地区名称相同的目录中，文件也必须名为`formats.py`。

需要这样一个结构来自定义英文格式：

```
mysite/
    formats/
        __init__.py
        en/
            __init__.py
            formats.py

```

其中`formats.py`包含自定义的格式定义。例如：

```
from __future__ import unicode_literals

THOUSAND_SEPARATOR = '\xa0'

```

使用非间断空格(Unicode `00A0`)作为千位分隔符，来代替英语中默认的逗号。

## 提供本地化格式的限制

一些地区对数字使用上下文敏感的格式，Django的本地化系统不能自动处理它。

### 瑞士(德语)

瑞士的数字格式化取决于被格式化的数字类型。对于货币值，使用逗号作为千位分隔符，以及使用小数点作为十进制分隔符。对于其它数字，逗号用于十进制分隔符，空格用于千位分隔符。Django提供的本地格式使用通用的分隔符，即逗号用于十进制分隔符，空格用于千位分隔符。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[ocalized Web UI formatting and form input](https://docs.djangoproject.com/en/1.8/topics/i18n/formatting/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
