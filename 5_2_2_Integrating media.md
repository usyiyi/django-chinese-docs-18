# 表单素材 ( `Media` 类)

渲染有吸引力的、易于使用的web表单不仅仅需要HTML -- 同时也需要CSS样式表，并且，如果你打算使用奇妙的web2.0组件，你也需要在每个页面包含一些JavaScript。任何提供的页面都需要CSS和JavaScript的精确配合，它依赖于页面上所使用的组件。

这就是素材定义所导入的位置。Django允许你将一些不同的文件 -- 像样式表和脚本 -- 与需要这些素材的表单和组件相关联。例如，如果你想要使用日历来渲染DateField，你可以定义一个自定义的日历组件。这个组件可以与渲染日历所需的CSS和JavaScript关联。当日历组件用在表单上的时候，Django可以识别出所需的CSS和JavaScript文件，并且提供一个文件名的列表，以便在你的web页面上简单地包含这些文件。

素材和Django&nbsp;Admin

Django的Admin应用为日历、过滤选择等一些东西定义了一些自定义的组件。这些组件定义了素材的需求，DJango Admin使用这些自定义组件来代替Django默认的组件。Admin模板只包含在提供页面上渲染组件所需的那些文件。

如果你喜欢Django Admin应用所使用的那些组件，可以在你的应用中随意使用它们。它们位于`django.contrib.admin.widgets`。

选择哪个JavaScript工具包？

现在有许多JavaScript工具包，它们中许多都包含组件（比如日历组件），可以用于提升你的应用。Django 有意避免去称赞任何一个JavaScript工具包。每个工具包都有自己的有点和缺点 -- 要使用适合你需求的任何一个。Django 有能力集成任何JavaScript工具包。

## 作为静态定义的素材

定义素材的最简单方式是作为静态定义。如果使用这种方式，定义在`Media`内部类中出现，内部类的属性定义了需求。

这是一个简单的例子：

```
from django import forms

class CalendarWidget(forms.TextInput):
    class Media:
        css = {
            'all': ('pretty.css',)
        }
        js = ('animations.js', 'actions.js')

```

上面的代码定义了&nbsp;`CalendarWidget`，它继承于`TextInput`。每次CalendarWidget在表单上使用时，表单都会包含CSS文件`pretty.css`，以及JavaScript文件`animations.js` 和 `actions.js`。

静态定义在运行时被转换为名为`media`的组件属性。`CalendarWidget`实例的素材列表可以通过这种方式获取：

```
>>> w = CalendarWidget()
>>> print(w.media)
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>

```

下面是所有可能的`Media`选项的列表。它们之中没有必需选项。

### css

各种表单和输出媒体所需的，描述CSS的字典。

字典中的值应该为文件名称的列表或者元组。对于如何指定这些文件的路径，详见[_路径的章节_](#form-asset-paths)。

字典中的键位输出媒体的类型。它们和媒体声明中CSS文件接受的类型相同：&nbsp;‘all’, ‘aural’, ‘braille’, ‘embossed’, ‘handheld’, ‘print’, ‘projection’, ‘screen’, ‘tty’ 和‘tv’。如果你需要为不同的媒体类型使用不同的样式表，要为每个输出媒体提供一个CSS文件的列表。下面的例子提供了两个CSS选项 -- 一个用于屏幕，另一个用于打印：

```
class Media:
    css = {
        'screen': ('pretty.css',),
        'print': ('newspaper.css',)
    }

```

如果一组CSS文件适用于多种输出媒体的类型，字典的键可以为输出媒体类型的逗号分隔的列表。在下面的例子中，TV和投影仪具有相同的媒体需求：

```
class Media:
    css = {
        'screen': ('pretty.css',),
        'tv,projector': ('lo_res.css',),
        'print': ('newspaper.css',)
    }

```

如果最后的CSS定义即将被渲染，会变成下面的HTML：

```
<link href="http://static.example.com/pretty.css" type="text/css" media="screen" rel="stylesheet" />
<link href="http://static.example.com/lo_res.css" type="text/css" media="tv,projector" rel="stylesheet" />
<link href="http://static.example.com/newspaper.css" type="text/css" media="print" rel="stylesheet" />

```

### js

所需的JavaScript文件由一个元组来描述。如何制定这些文件的路径，详见[_路径一节_](#form-asset-paths)。

### extend

一直布尔值，定义了`Media`声明的继承行为。

通常，任何使用静态`Media`定义的对象都会继承所有和父组件相关的素材。无论父对象如何定义它自己的需求，都是这样。例如，如果我们打算从上面的例子中扩展我们的基础日历控件：

```
>>> class FancyCalendarWidget(CalendarWidget):
...     class Media:
...         css = {
...             'all': ('fancy.css',)
...         }
...         js = ('whizbang.js',)

>>> w = FancyCalendarWidget()
>>> print(w.media)
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<link href="http://static.example.com/fancy.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>
<script type="text/javascript" src="http://static.example.com/whizbang.js"></script>

```

&nbsp;FancyCalendar 组件继承了所有父组件的素材。如果你不想让`Media` 以这种方式被继承，要向`Media` 声明中添加 `extend=False` 声明：

```
>>> class FancyCalendarWidget(CalendarWidget):
...     class Media:
...         extend = False
...         css = {
...             'all': ('fancy.css',)
...         }
...         js = ('whizbang.js',)

>>> w = FancyCalendarWidget()
>>> print(w.media)
<link href="http://static.example.com/fancy.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/whizbang.js"></script>

```

如果你需要对继承进行更多控制，要使用[_动态属性_](#dynamic-property)来定义你的素材。动态属性可以提供更多的控制，来控制继承哪个文件。

## Media as a dynamic property

如果你需要对素材需求进行更多的复杂操作，你可以直接定义`media`属性。这可以通过定义一个返回`forms.Media`实例的组件属性来实现。`forms.Media`的构造器接受 `css` 和 `js`关键字参数，和在静态媒体定义中的格式相同。

例如，我们的日历组件的静态定义可以定义成动态形式：

```
class CalendarWidget(forms.TextInput):
    def _media(self):
        return forms.Media(css={'all': ('pretty.css',)},
                           js=('animations.js', 'actions.js'))
    media = property(_media)

```

对于如何构建动态`media` 属性的的返回值，详见[媒体对象](#media-objects)一节。

## 素材定义中的路径

用于指定素材的路径可以是相对的或者绝对的。如果路径以 `/`，`http://` 或者`https://`开头，会被解释为绝对路径。所有其它的路径会在开头追加合适前缀的值。

作为&nbsp;[_staticfiles app_](../../ref/contrib/staticfiles.html)的简介的一部分，添加了两个新的设置，它们涉及到渲染完整页面所需的“静态文件”：[`STATIC_URL`](../../ref/settings.html#std:setting-STATIC_URL) 和[`STATIC_ROOT`](../../ref/settings.html#std:setting-STATIC_ROOT)。

Django 会检查是否[`STATIC_URL`](../../ref/settings.html#std:setting-STATIC_URL)设置不是`None`，来寻找合适的前缀来使用，并且会自动回退使用[`MEDIA_URL`](../../ref/settings.html#std:setting-MEDIA_URL)。例如，如果你站点的 [`MEDIA_URL`](../../ref/settings.html#std:setting-MEDIA_URL) 是 `'http://uploads.example.com/'` 并且 [`STATIC_URL`](../../ref/settings.html#std:setting-STATIC_URL) 是`None`：

```
>>> from django import forms
>>> class CalendarWidget(forms.TextInput):
...     class Media:
...         css = {
...             'all': ('/css/pretty.css',),
...         }
...         js = ('animations.js', 'http://othersite.com/actions.js')

>>> w = CalendarWidget()
>>> print(w.media)
<link href="/css/pretty.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://uploads.example.com/animations.js"></script>
<script type="text/javascript" src="http://othersite.com/actions.js"></script>

```

但如果[`STATIC_URL`](../../ref/settings.html#std:setting-STATIC_URL) 为 `'http://static.example.com/'`：

```
>>> w = CalendarWidget()
>>> print(w.media)
<link href="/css/pretty.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://othersite.com/actions.js"></script>

```

## `Media`对象

当你访问表单上的一个组件的`media`属性时，返回值是一个`forms.Media`对象。就像已经看到的那样，表示&nbsp;`Media` 对象的字符串，是在你的HTML页面的`<head>` 代码段包含相关文件所需的HTML。

然而，`Media`对象具有一些其它的有趣属性。

### 素材的子集

如果你仅仅想得到特定类型的文件，你可以使用下标运算符来过滤出你感兴趣的媒体。例如：

```
>>> w = CalendarWidget()
>>> print(w.media)
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>

>>> print(w.media['css'])
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />

```

当你使用下标运算符的时候，返回值是一个新的&nbsp;`Media`对象，但是只含有感兴趣的媒体。

### 组合`Media`对象

`Media` 对象可以添加到一起。添加两个`Media`的时候，产生的`Media`对象含有二者指定的素材的并集：

```
>>> from django import forms
>>> class CalendarWidget(forms.TextInput):
...     class Media:
...         css = {
...             'all': ('pretty.css',)
...         }
...         js = ('animations.js', 'actions.js')

>>> class OtherWidget(forms.TextInput):
...     class Media:
...         js = ('whizbang.js',)

>>> w1 = CalendarWidget()
>>> w2 = OtherWidget()
>>> print(w1.media + w2.media)
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>
<script type="text/javascript" src="http://static.example.com/whizbang.js"></script>

```

## 表单上的`Media`

组件并不是唯一拥有`media`定义的对象 -- 表单可以定义`media`。在表单上定义`media` 的规则和组件上面一样：定义可以为静态的或者动态的。声明的路径和继承规则也严格一致。

无论是否你定义了`media`，&nbsp;_所有_表单对象都有`media`属性。这个属性的默认值是，向所有属于这个表单的组件添加`media`定义的结果。

```
>>> from django import forms
>>> class ContactForm(forms.Form):
...     date = DateField(widget=CalendarWidget)
...     name = CharField(max_length=40, widget=OtherWidget)

>>> f = ContactForm()
>>> f.media
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>
<script type="text/javascript" src="http://static.example.com/whizbang.js"></script>

```

如果你打算向表单关联一些额外的素材 -- 例如，表单布局的CSS -- 只是向表单添加`Media`声明就可以了：

```
>>> class ContactForm(forms.Form):
...     date = DateField(widget=CalendarWidget)
...     name = CharField(max_length=40, widget=OtherWidget)
...
...     class Media:
...         css = {
...             'all': ('layout.css',)
...         }

>>> f = ContactForm()
>>> f.media
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<link href="http://static.example.com/layout.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>
<script type="text/javascript" src="http://static.example.com/whizbang.js"></script>

```

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Integrating media](https://docs.djangoproject.com/en/1.8/topics/forms/media/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
