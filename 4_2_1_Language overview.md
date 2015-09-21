{% raw %}

# Django模版语言

本文将介绍Django模版系统的语法。如果您需要更多该系统如何工作的技术细节，以及希望扩展它，请浏览 [_The Django template language: for Python programmers_](api.html).

Django模版语言的设计致力于在性能和简单上取得平衡。
它的设计使习惯于使用HTML的人也能够自如应对。如果您有过使用其他模版语言的经验，像是 [Smarty](http://www.smarty.net/) 或者 [Jinja2](http://jinja.pocoo.org/), 那么您将对Django的模版语言感到一见如故。

理念

如果您有过编程背景，或者您使用过一些在HTML中直接混入程序代码的语言，那么现在您需要记住，Django的模版系统并不是简单的将Python嵌入到HTML中。
设计决定了：模版系统致力于表达外观，而不是程序逻辑。

Django的模版系统提供了和一些程序结构功能类似的标签——用于布尔判断的 [`if`](builtins.html#std:templatetag-if) 标签, 用于循环的 [`for`](builtins.html#std:templatetag-for) 标签等等。——但是这些都不是简单的作为Python代码那样来执行的，并且，模版系统也不会随意执行Python表达式。只有下面列表中的标签、过滤器和语法才是默认就被支持的。 （但是您也可以根据需要添加 [_ 您自己的扩展 _](../../howto/custom-template-tags.html) &nbsp;到模版语言中）。

## 模版

模版是纯文本文件。它可以产生任何基于文本的的格式（HTML，XML，CSV等等）。

模版包括在使用时会被值替换掉的 **变量**，和控制模版逻辑的 **标签**。

下面是一个小模版，它说明了一些基本的元素。后面的文档中会解释每个元素。

```
{% extends "base_generic.html" %}

{% block title %}{{ section.title }}{% endblock %}

{% block content %}
<h1>{{ section.title }}</h1>

{% for story in story_list %}
<h2>
  <a href="{{ story.get_absolute_url }}">
    {{ story.headline|upper }}
  </a>
</h2>
<p>{{ story.tease|truncatewords:"100" }}</p>
{% endfor %}
{% endblock %}

```

理念

为什么要使用基于文本的模版，而不是基于XML的（比如Zope的TAL）呢？我们希望Django的模版语言可以用在更多的地方，而不仅仅是XML/HTML模版。在线上世界，我们在email、Javascript和CSV中使用它。你可以在任何基于文本的格式中使用这个模版语言。

还有，让人类编辑HTML简直是施虐狂的做法！

## 变量

变量看起来就像是这样： `{{ variable }}`. 当模版引擎遇到一个变量，它将计算这个变量，然后用结果替换掉它本身。变量的命名包括任何字母数字以及下划线 (`"_"`)的组合。点(`"."`) 也在会变量部分中出现，不过它有特殊的含义，我们将在后面说明。重要的是， _你不能在变量名称中使用空格和标点符号。_

使用点 (`.`) 来访问变量的属性。

幕后

从技术上来说，当模版系统遇到点，它将以这样的顺序查询：

*   字典查询（Dictionary lookup）
*   属性或方法查询（Attribute or method lookup）
*   数字索引查询（Numeric index lookup）

如果计算结果的值是可调用的，它将被无参数的调用。调用的结果将成为模版的值。

这个查询顺序，会在优先于字典查询的对象上造成意想不到的行为。例如，思考下面的代码片段，它尝试循环 `collections.defaultdict`:

```
{% for k, v in defaultdict.iteritems %}
    Do something with k and v here...
{% endfor %}

```

因为字典查询首先发生，行为奏效了并且提供了一个默认值，而不是使用我们期望的 `.iteritems()` 方法。在这种情况下，考虑首先转换成字典。

在前文的例子中， `{{ section.title }}`将被替换为 &nbsp;`section` 对象的&nbsp;`title` 属性。

如果你使用的变量不存在， 模版系统将插入 `string_if_invalid` 选项的值， 它被默认设置为`''` (空字符串) 。

注意模版表达式中的“bar”， 比如 `{{ foo.bar }}` 将被逐字直译为一个字符串，而不是使用变量“bar”的值，如果这样一个变量在模版上下文中存在的话。

## 过滤器

您可以通过使用 **过滤器**来改变变量的显示。

过滤器看起来是这样的：`{{ name|lower }}`。这将在变量 `{{ name }}` 被过滤器 [`lower`](builtins.html#std:templatefilter-lower) 过滤后再显示它的值，该过滤器将文本转换成小写。使用管道符号 (`|`)来应用过滤器。

过滤器能够被“串联”。一个过滤器的输出将被应用到下一个。`{{ text|escape|linebreaks }}` 就是一个常用的过滤器链，它编码文本内容，然后把行打破转成`<p>` 标签。

一些过滤器带有参数。过滤器的参数看起来像是这样： `{{ bio|truncatewords:30 }}`。这将显示 `bio` 变量的前30个词。

过滤器参数包含空格的话，必须被引号包起来；例如，连接一个有逗号和空格的列表，你需要使用 `{{ list|join:", " }}`。

Django提供了大约六十个内置的模版过滤器。你可以在 [_内置过滤器参考手册_](builtins.html#ref-templates-builtins-filters)中阅读全部关于它们的信息。为了体验一下它们的作用，这里有一些常用的模版过滤器：

[`default`](builtins.html#std:templatefilter-default)

如果一个变量是false或者为空，使用给定的默认值。否则，使用变量的值。例如：

```
{{ value|default:"nothing" }}

```

如果 `value`没有被提供，或者为空， 上面的例子将显示“`nothing`”。

[`length`](builtins.html#std:templatefilter-length)

返回值的长度。它对字符串和列表都起作用。例如：

```
{{ value|length }}

```

如果 `value` 是 `['a', 'b', 'c', 'd']`，那么输出是 `4`。

[`filesizeformat`](builtins.html#std:templatefilter-filesizeformat)

将值格式化为一个 “人类可读的” 文件尺寸 （例如 `'13 KB'`, `'4.1 MB'`, `'102 bytes'`, 等等）。例如：

```
{{ value|filesizeformat }}

```

如果 `value` 是 123456789，输出将会是 `117.7 MB`。

再说一下，这仅仅是一些例子；查看 [_内置过滤器参考手册_](builtins.html#ref-templates-builtins-filters) 来获取完整的列表。

您也可以创建自己的自定义模版过滤器；参考 [_自定义模版标签和过滤器_](../../howto/custom-template-tags.html)。

更多

Django’s admin interface can include a complete reference of all template tags and filters available for a given site. See [_The Django admin documentation generator_](../contrib/admin/admindocs.html).

## 标签

标签看起来像是这样的： `{% tag %}`。标签比变量更加复杂：一些在输出中创建文本，一些通过循环或逻辑来控制流程，一些加载其后的变量将使用到的额外信息到模版中。

一些标签需要开始和结束标签 （例如`{% tag %} ... `标签 内容 ... {% endtag %}）。

Django自带了大约24个内置的模版标签。你可以在 [_内置标签参考手册_](builtins.html#ref-templates-builtins-tags)中阅读全部关于它们的内容。为了体验一下它们的作用，这里有一些常用的标签：

[`for`](builtins.html#std:templatetag-for)

循环数组中的每个元素。例如，显示 `athlete_list`中提供的运动员列表：

```
<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% endfor %}
</ul>

```

[`if`](builtins.html#std:templatetag-if), `elif`, and `else`

计算一个变量，并且当变量是“true”是，显示块中的内容：

```
{% if athlete_list %}
    Number of athletes: {{ athlete_list|length }}
{% elif athlete_in_locker_room_list %}
    Athletes should be out of the locker room soon!
{% else %}
    No athletes.
{% endif %}

```

在上面的例子中，如果 `athlete_list` 不是空的，运动员的数量将显示为 `{{ athlete_list|length }}` 的输出。另一方面， 如果 `athlete_in_locker_room_list` 不为空， 将显示 “Athletes should be out...” 这个消息。如果两个列表都是空的，将显示 “No athletes.” 。

您也可以在[`if`](builtins.html#std:templatetag-if) 标签中使用过滤器和多种运算符：

```
{% if athlete_list|length > 1 %}
   Team: {% for athlete in athlete_list %} ... {% endfor %}
{% else %}
   Athlete: {{ athlete_list.0.name }}
{% endif %}

```

当上面的例子工作时，需要注意，大多数模版过滤器返回字符串，所以使用过滤器做数学的比较通常都不会像您期望的那样工作。[`length`](builtins.html#std:templatefilter-length) 是一个例外。

[`block`](builtins.html#std:templatetag-block) and [`extends`](builtins.html#std:templatetag-extends)

Set up [template inheritance](#id1) (see below), a powerful way
of cutting down on “boilerplate” in templates.

再说一下，上面的仅仅是整个列表的一部分；查看 [_内置标签参考手册_](builtins.html#ref-templates-builtins-tags) 来获取完整的列表。

您也可以创建您自己的自定义模版标签；参考 [_自定义模版标签和过滤器_](../../howto/custom-template-tags.html)。

更多

Django’s admin interface can include a complete reference of all template tags and filters available for a given site. See [_The Django admin documentation generator_](../contrib/admin/admindocs.html).

## 注释

要注释模版中一行的部分内容，使用注释语法 `{# #}`.

例如，这个模版将被渲染为 `'hello'`：

```
{# greeting #}hello

```

注释可以包含任何模版代码，有效的或者无效的都可以。例如：

```
{# {% if foo %}bar{% else %} #}

```

这个语法只能被用于单行注释 （在 `{#` 和 `#}` 分隔符中，不允许有新行）。如果你需要注释掉模版中的多行内容，请查看 [`comment`](builtins.html#std:templatetag-comment) 标签。

## 模版继承

Django模版引擎中最强大也是最复杂的部分就是模版继承了。模版继承可以让您创建一个基本的“骨架”模版，它包含您站点中的全部元素，并且可以定义能够被子模版覆盖的 **blocks** 。

通过从下面这个例子开始，可以容易的理解模版继承：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <link rel="stylesheet" href="style.css" />
    <title>{% block title %}My amazing site{%/span> endblock %}</title>
</head>

<body>
    <div id="sidebar">
        {% block sidebar %}
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/blog/">Blog</a></li>
        </ul>
        {% endblock %}
    </div>

    <div id="content">
        {% block content %}{% endblock %}
    </div>
</body>
</html>

```

这个模版，我们把它叫作 `base.html`， 它定义了一个可以用于两列排版页面的简单HTML骨架。“子模版”的工作是用它们的内容填充空的blocks。

在这个例子中， [`block`](builtins.html#std:templatetag-block) 标签定义了三个可以被子模版内容填充的block。&nbsp;[`block`](builtins.html#std:templatetag-block) 告诉模版引擎： 子模版可能会覆盖掉模版中的这些位置。

子模版可能看起来是这样的：

```
{% extends "base.html" %}/span>

{% block title %}My amazing blog{% endblock %}

{% block content %}
{% for entry in blog_entries %}
    <h2>{{ entry.title }}</h2>
    <p>{{ entry.body }}</p>
{% endfor %}
{% endblock %}

```

[`extends`](builtins.html#std:templatetag-extends) 标签是这里的关键。它告诉模版引擎，这个模版“继承”了另一个模版。当模版系统处理这个模版时，首先，它将定位父模版——在此例中，就是“base.html”。

那时，模版引擎将注意到 `base.html` 中的三个 [`block`](builtins.html#std:templatetag-block) 标签，并用子模版中的内容来替换这些block。根据 `blog_entries` 的值，输出可能看起来是这样的：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <link rel="stylesheet" href="style.css" />
    <title>My amazing blog</title>
</head>

<body>
    <div id="sidebar">
        <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/blog/">Blog</a></li>
        </ul>
    </div>

    <div id="content">
        <h2>Entry one</h2>
        <p>This is my first entry.</p>

        <h2>Entry two</h2>
        <p>This is my second entry.</p>
    </div>
</body>
</html>

```

请注意，子模版并没有定义 `sidebar` block，所以系统使用了父模版中的值。父模版的 `{% block %}` 标签中的内容总是被用作备选内容（fallback）。

您可以根据需要使用多级继承。使用继承的一个常用方式是类似下面的三级结构：

*   创建一个 `base.html` 模版来控制您整个站点的主要视觉和体验。
*   为您的站点的每一个“部分”创建一个`base_SECTIONNAME.html` 模版。&nbsp;例如， `base_news.html`, `base_sports.html`。这些模版都继承自 `base.html` ，并且包含了每部分特有的样式和设计。
*   为每一种页面类型创建独立的模版，例如新闻内容或者博客文章。这些模版继承了有关的部分模版（section template）。

这种方式使代码得到最大程度的复用，并且使得添加内容到共享的内容区域更加简单，例如，部分范围内的导航。

这里是使用继承的一些提示：

*   如果你在模版中使用 [`{% extends %}`](builtins.html#std:templatetag-extends) 标签，它必须是模版中的第一个标签。其他的任何情况下，模版继承都将无法工作。

*   在base模版中设置越多的 [`{% block %}`](builtins.html#std:templatetag-block) 标签越好。请记住，子模版不必定义全部父模版中的blocks，所以，你可以在大多数blocks中填充合理的默认内容，然后，只定义你需要的那一个。多一点钩子总比少一点好。

*   如果你发现你自己在大量的模版中复制内容，那可能意味着你应该把内容移动到父模版中的一个 `{% block %}` 中。

*   If you need to get the content of the block from the parent template, the `{{ block.super }}` variable will do the trick. This is useful if you want to add to the contents of a parent block instead of completely overriding it. Data inserted using `{{ block.super }}` will not be automatically escaped (see the [next section](#automatic-html-escaping)), since it was already escaped, if necessary, in the parent template.

*   为了更好的可读性，你也可以给你的 `{% endblock %}` 标签一个 _名字_ 。例如：

    ```
    {% block content %}
    ...
    {% endblock content %}

    ```

    在大型模版中，这个方法帮你清楚的看到哪一个　 `{% block %}` 标签被关闭了。

最后，请注意您并不能在一个模版中定义多个相同名字的 [`block`](builtins.html#std:templatetag-block) 标签。这个限制的存在是因为block标签的作用是“双向”的。这个意思是，block标签不仅提供了一个坑去填，它还在 _父模版_中定义了填坑的内容。如果在一个模版中有两个名字一样的 [`block`](builtins.html#std:templatetag-block) 标签，模版的父模版将不知道使用哪个block的内容。

## 自动HTML转义

当从模版中生成HTML时，总会有这样一个风险：值可能会包含影响HTML最终呈现的字符。例如，思考这个模版片段：

```
Hello, {{ name }}

```

首先，它看起来像是一个无害的方式来显示用户的名字，但是设想一下，如果用户像下面这样输入他的名字，会发生什么：

```
<script>alert('hello')</script>

```

使用这个名字值，模版将会被渲染成这样：

```
Hello, <script>alert('hello')</script>

```

...这意味着，浏览器将会弹出一个Javascript警示框！

类似的，如果名字包含一个 `'<'` 符号（比如下面这样），会发生什么呢？

```
<b>username

```

这将会导致模版呗渲染成这样：

```
Hello, <b>username

```

...进而这将导致网页的剩余部分都被加粗！

显然，用户提交的数据都被应该被盲目的信任，并且被直接插入到你的网页中，因为一个怀有恶意的用户可能会使用这样的漏洞来做一些可能的坏事。这种类型的安全问题被叫做 [跨站脚本（Cross Site Scripting）](http://en.wikipedia.org/wiki/Cross-site_scripting) (XSS) 攻击。

为避免这个问题，你有两个选择：

*   第一， 你可以确保每一个不被信任的值都通过 [`escape`](builtins.html#std:templatefilter-escape) 过滤器（下面的文档中将提到）运行，它将把潜在的有害HTML字符转换成无害的。This was the default solution in Django for its first few years, but the problem is that it puts the onus on _you_, the developer / template author, to ensure you’re escaping everything. It’s easy to forget to escape data.
*   第二，你可以利用Django的自动HTML转义。 本节描述其余部分描述的是自动转义是如何工作的

By default in Django, every template automatically escapes the output of every variable tag. Specifically, these five characters are escaped:

*   `<`会转换为`<`
*   `>`会转换为`>`
*   `'` (单引号) 会转换为`&#39;`
*   `"` (双引号)会转换为 `"`
*   `&` 会转换为 `&amp;`

我们要再次强调这是默认行为。如果你使用Django的模板系统，会处于保护之下。

### 如果关闭它

如果你不希望数据自动转义，在站点、模板或者变量级别，你可以使用几种方法来关闭它。

然而你为什么想要关闭它呢？由于有时，模板变量含有一些你_打算_渲染成原始HTML的数据，你并不想转义这些内容。例如，你可能会在数据库中储存一些HTML代码，并且直接在模板中嵌入它们。或者，你可能使用Django的模板系统来生成_不是_HTML的文本 -- 比如邮件信息。

#### 用于独立变量

使用[`safe`](builtins.html#std:templatefilter-safe)过滤器来关闭独立变量上的自动转移：

```
This will be escaped: {{ data }}
This will not be escaped: {{ data|safe }}

```

_safe_是_safe from further escaping_或者_can be safely interpreted as HTML_的缩写。
在这个例子中，如果`data`含有`'<b>'`，输出会是：

```
This will be escaped: <b>
This will not be escaped: <b>

```

#### 用于模板代码块

要控制模板上的自动转移，将模板（或者模板中的特定区域）包裹在[`autoescape`](builtins.html#std:templatetag-autoescape)标签 中，像这样：

```
{% autoescape off %}
    Hello {{ name }}
{% endautoescape %}

```

[`autoescape`](builtins.html#std:templatetag-autoescape)标签接受`on` 或者 `off`作为它的参数。有时你可能想在自动转移关闭的情况下强制使用它。下面是一个模板的示例：

```
Auto-escaping is on by default. Hello {{ name }}

{% autoescape off %}
    This will not be auto-escaped: {{ data }}.

    Nor this: {{ other_data }}
    {% autoescape on %}
        Auto-escaping applies again: {{ name }}
    {% endautoescape %}
{% endautoescape %}

```

自动转移标签作用于扩展了当前模板的模板，以及通过&nbsp;[`include`](builtins.html#std:templatetag-include) 标签包含的模板，就像所有block标签那样。例如：

base.html
```
{% autoescape off %}
<h1>{% block title %}{% endblock %}</h1>
{% block content %}
{% endblock %}
{% endautoescape %}

```

child.html
```
{% extends "base.html" %}
{% block title %}This &amp; that{% endblock %}
{% block content %}{{ greeting }}{% endblock %}

```

由于自动转义标签在base模板中关闭，它也会在child模板中关闭，导致当&nbsp;`greeting`变量含有`<b>Hello!</b>`字符串时，会渲染HTML。

```
<h1>This &amp; that</h1>
<b>Hello!</b>

```

### 注释

通常，模板的作用并不非常担心自动转义。Python一边的开发者（编写视图和自定义过滤器的人）需要考虑数据不应被转移的情况，以及合理地标记数据，让这些东西在模板中正常工作。

如果你创建了一个模板，它可能用于你不确定自动转移是否开启的环境，那么应该向任何需要转移的变量添加&nbsp;[`escape`](builtins.html#std:templatefilter-escape)过滤器。当自动转移打开时，[`escape`](builtins.html#std:templatefilter-escape)过滤器_双重过滤_数据没有任何危险 --&nbsp;[`escape`](builtins.html#std:templatefilter-escape)过滤器并不影响自动转义的变量。

### 字符串字面值和自动转义

像我们之前提到的那样，过滤器参数可以是字符串：

```
{{ data|default:"This is a string literal." }}

```

所有字面值字符串在插入模板时都&nbsp;**不会**带有任何自动转义 -- 它们的行为类似于通过&nbsp;[`safe`](builtins.html#std:templatefilter-safe)过滤器传递。背后的原因是，模板作者可以控制字符串字面值得内容，所以它们可以确保在模板编写时文本经过正确转义。

也即是说你可以编写

```
{{ data|default:"3 < 2" }}

```

...而不是：

```
{{ data|default:"3 < 2" }}  {# Bad! Don't do this. #}

```

这并不影响来源于模板自身的数据。模板内容在必要时仍然会自动转移，因为它们不受模板作者的控制。

## 访问方法调用

大多数对象上的方法调用同样可用于模板中。这意味着模板必须拥有对除了类属性（像是字段名称）和从视图中传入的变量之外的访问。例如，Django ORM提供了[_“entry_set”_](../../topics/db/queries.html#topics-db-queries-related) 语法用于查找关联到外键的对象集合。所以，提供一个模型叫做“comment”，并带有一个关联到&nbsp;“task” 模型的外键，你就可以遍历给定任务附带的所有评论，像这样：

```
{% for comment in task.comment_set.all %}
    {{ comment }}
{% endfor %}

```

与之类似，[_QuerySets_](../models/querysets.html)提供了&nbsp;`count()`方法来计算含有对象的总数。因此，你可以像这样获取所有关于当前任务的评论总数：

```
{{ task.comment_set.all.count }}

```

当然，你可以轻易访问已经显式定义在你自己模型上的方法：

models.py
```
class Task(models.Model):
    def foo(self):
        return "bar"

```

template.html
```
{{ task.foo }}

```

由于Django有意限制了模板语言中逻辑处理的总数，不能够在模板中传递参数来调用方法。数据应该在视图中处理，然后传递给模板用于展示。

## 自定义标签和过滤器库

特定的应用提供自定义的标签和过滤器库。要在模板中访问它们，确保应用在[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS)之内（在这个例子中我们添加了`'django.contrib.humanize'`），之后在模板中使用[`load`](builtins.html#std:templatetag-load)标签：

```
{% load humanize %}

{{ 45000|intcomma }}

```

上面的例子中，&nbsp;[`load`](builtins.html#std:templatetag-load)标签加载了`humanize`标签库，之后我们可以使用`intcomma`过滤器。如果你开启了[`django.contrib.admindocs`](../contrib/admin/admindocs.html#module-django.contrib.admindocs)，你可以查询admin站点中的文档部分，来寻找你的安装中的自定义库列表。

[`load`](builtins.html#std:templatetag-load)标签可以接受多个库名称，由空格分隔。例如：

```
{% load humanize i18n %}

```

关于编写你自己的自定义模板库，详见[_自定义模板标签和过滤器_](../../howto/custom-template-tags.html)。

### 自定义库和模板继承

当你加载一个自定义标签或过滤器库时，标签或过滤器只在当前模板中有效 -- 并不是带有模板继承关系的任何父模板或者子模版中都有效。

例如，如果一个模板`foo.html`带有`{% load humanize %}`，子模版（例如，带有`{% extends "foo.html" %}`）中_不能_&nbsp;访问humanize模板标签和过滤器。子模版需要添加自己的&nbsp;`{% load humanize %}`。

这个特性是可维护性和逻辑性的缘故。

另见

[_The Templates Reference_](index.html)

Covers built-in tags, built-in filters, using an alternative template,
language, and more.

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Language overview](https://docs.djangoproject.com/en/1.8/ref/templates/language/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。

{% endraw %}
