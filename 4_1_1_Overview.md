{% raw %}

# 模版

作为Web 框架，Django 需要一种很便利的方法以动态地生成HTML。最常见的做法是使用模板。模板包含所需HTML 输出的静态部分，以及一些特殊的语法，描述如何将动态内容插入。创建HTML 页面模板的一个示例，参见[_教程第3部分_](../intro/tutorial03.html)。

Django 项目可以配置一个或多个模板引擎（甚至是零，如果你不需要使用模板）。Django 的模板系统自带内建的后台 —— 称为Django 模板语言（DTL），以及另外一种流行的[Jinja2](http://jinja.pocoo.org/)。其他的模板语言的后端，可查找第三方库。

Django 为加载和渲染模板定义了一套标准的API，与具体的后台无关。加载包括根据给定的标识找到模板然后预处理，通常会将它编译好放在内存中。渲染表示使用Context 数据对模板插值并返回生成的字符串。

[_Django 模板语言_](../ref/templates/language.html) 是Django 原生的模板系统。直到Django 1.8，这是唯一可用的内置选项。尽管，它闭门造车，并且偏重某些方面，但是它仍然是一个优秀的模版库。如果没有特别紧急的理由选择另外一种后台，你应该使用DTL，特别是你编写可插拔的应用并打算发布其模板的时候。Django 中包含模板的标准应用，例如[_django.contrib.admin_](../ref/contrib/admin/index.html)，都使用DTL。

又由于历史遗留原因,通用支持的模板引擎和Django实现的模板语言都在`django.template` 命名空间中.

## 模板引擎的支持

New in Django 1.8:

Django1.8 中增加了对多种模板引擎的支持和[`TEMPLATES`](../ref/settings.html#std:setting-TEMPLATES) 设置。

### 配置

模板引擎通过[`TEMPLATES`](../ref/settings.html#std:setting-TEMPLATES) 设置来配置。它是一个设置选项列表，与引擎一一对应。默认的值为空。由[`startproject`](../ref/django-admin.html#django-admin-startproject) 命令生成的`settings.py`  定义了一些有用的值：

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            # ... some options here ...
        },
    },
]

```

[`BACKEND`](../ref/settings.html#std:setting-TEMPLATES-BACKEND) 是一个指向实现了Django模板后端API的模板引擎类的带点的Python路径。内置的后端有 [`django.template.backends.django.DjangoTemplates`](#django.template.backends.django.DjangoTemplates "django.template.backends.django.DjangoTemplates") 和 [`django.template.backends.jinja2.Jinja2`](#django.template.backends.jinja2.Jinja2 "django.template.backends.jinja2.Jinja2").

由于绝大多数引擎都是从文件加载模板的，所以每种模板引擎都包含两项通用设置：

*   [`DIRS`](../ref/settings.html#std:setting-TEMPLATES-DIRS) 定义了一个目录列表，模板引擎按列表顺序搜索这些目录以查找模板源文件。
*   [`APP_DIRS`](../ref/settings.html#std:setting-TEMPLATES-APP_DIRS) 告诉模板引擎是否应该进入每个已安装的应用中查找模板。每种模板引擎后端都定义了一个惯用的名称作为应用内部存放模板的子目录名称。（译者注：例如django为它自己的模板引擎指定的是 ‘templates’ ，为jinja2指定的名字是‘jinja2’）

特别的是，django允许你有多个模板引擎后台实例，且每个实例有不同的配置选项。在这种情况下你必须为每个配置指定一个唯一的[`NAME`](../ref/settings.html#std:setting-TEMPLATES-NAME) .

[`OPTIONS`](../ref/settings.html#std:setting-TEMPLATES-OPTIONS)  中包含了具体的backend设置

### 用法

 `django.template.loader` 定义了两个函数以加载模板。

`get_template`(_template_name[, dirs][, using]_)[[source]](../_modules/django/template/loader.html#get_template)

该函数使用给定的名称加载模板并返回一个`Template` 对象.

真正的返回值类型取决于那个用来加载模版的后台引擎。每个后台都有各自的 `Template`类。

`get_template()`尝试获取每个模板直到有一个成功满足。如果模板不能成功找到，将会抛出[`TemplateDoesNotExist`](#django.template.TemplateDoesNotExist "django.template.TemplateDoesNotExist"). 如果能够找到模板但是包含非法值，将会抛出 [`TemplateSyntaxError`](#django.template.TemplateSyntaxError "django.template.TemplateSyntaxError").

模板的查找和加载机制取决于每种后台引擎和配置

如果你想使用指定的模板引擎进行查找,请将模板引擎的[`NAME`](../ref/settings.html#std:setting-TEMPLATES-NAME) 赋给 get_template的`using` 参数

Changed in Django 1.7:

 添加了`dirs`参数。

自1.8版起已弃用： `dirs` 参数被移除。

Changed in Django 1.8:

添加了 参数`using` 。

Changed in Django 1.8:

`get_template()` 返回了一个基于后端引擎的 `Template`而不是[`django.template.Template`](../ref/templates/api.html#django.template.Template "django.template.Template").这个类。

`select_template`(_template_name_list[, dirs][, using]_)[[source]](../_modules/django/template/loader.html#select_template)

`select_template()` 和 `get_template()`很相似, 只不过它用了一个模板名称的列表作为参数。按顺序搜索模板名称列表内的模板并返回第一个存在的模板。

Changed in Django 1.7:

增加`dirs` 参数。

自1.8版起已弃用：废弃`dirs` 参数。

Changed in Django 1.8:

已添加`using`参数。

Changed in Django 1.8:

`select_template()`返回与后端相关的`Template`，而不是[`django.template.Template`](../ref/templates/api.html#django.template.Template "django.template.Template")。

如果导入模板失败，`django.template`中定义的两个异常，有可能会被抛出：

_exception_ `TemplateDoesNotExist`

在找不到该模板时，抛出该错误。

_exception_ `TemplateSyntaxError`

当模板内出现错误时，将被抛出

由`get_template()` 和`select_template()` 返回的`Template` 对象必须要有一个`render()`方法，协议如下：

`Template.``render`(_context=None_, _request=None_)

通过给定的 context 对该模板进行渲染。

如果提供了 `context` ，那么它必须是一个[`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.4)")对象. 如果没有提供，引擎将是用空 context 对模板进行渲染。

如果要提供`request`参数 ，必须使用 [`HttpRequest`](../ref/request-response.html#django.http.HttpRequest "django.http.HttpRequest")对象. 之后模板引擎会使它连同CSRF token一起在模板中可用.具体如何实现由相应模板后端决定.

关于搜索算法的例子。该例子下 [`TEMPLATES`](../ref/settings.html#std:setting-TEMPLATES) 的配置是:

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            '/home/html/example.com',
            '/home/html/default',
        ],
    },
    {
        'BACKEND': 'django.template.backends.jinja2.Jinja2',
        'DIRS': [
            '/home/html/jinja2',
        ],
    },
]

```

如果你调用函数`get_template('story_detail.html')`, Django按以下顺序查找story_detail.html：

*   `/home/html/example.com/story_detail.html`（`'django'` engine）
*   `/home/html/default/story_detail.html`（`'django'` engine）
*   `/home/html/jinja2/story_detail.html`（`'jinja2'`引擎）

如果你调用函数 `select_template(['story_253_detail.html', 'story_detail.html'])`, Django按一下顺序查找：

*   `/home/html/example.com/story_253_detail.html`（`'django'` engine）
*   `/home/html/default/story_253_detail.html`（`'django'`引擎）
*   `/home/html/jinja2/story_253_detail.html`（`'jinja2'`引擎）
*   `/home/html/example.com/story_detail.html`（`'django'` engine）
*   `/home/html/default/story_detail.html`（`'django'` engine）
*   `/home/html/jinja2/story_detail.html`（`'jinja2'`引擎）

当Django发现模板存在后便停止搜寻。

提示

你可以通过 [`select_template()`](#django.template.loader.select_template "django.template.loader.select_template") 来实现更为灵活的模板加载。例如，如果您撰写了新闻报道并希望某些报道有自定义模板，请使用`select_template（['story_％s_detail.html' ％ t2 &gt; story.id， 'story_detail.html']）`。这允许你为每个故事使用一个个性化的模板,同时有一个后背模板给没有个性化模板的故事使用.

如果可能（其实这会更好）—将模版文件，放在包含模版的目录的子目录下。基本思路是，使得每个APP的的模版子目录下都有一个子目录来唯一对应这个APP。

这样做可以增强你的APP可用性。将所有的模版放在根模版目录下会引发混淆。

要在一个子目录内加载模板，像这样使用斜线就好了：

```
get_template('news/story_detail.html')

```

作为上述使用相同的 [`模板`](../ref/settings.html#std:setting-TEMPLATES) 选项，这将会尝试加载下列模板︰

*   `/home/html/example.com/news/story_detail.html`（`'django'` engine）
*   `/home/html/default/news/story_detail.html`（`'django'` engine）
*   `/home/html/jinja2/news/story_detail.html`（`'jinja2'`引擎）

另外，为了减少加载模板、渲染模板等重复工作，django提供了处理这些工作的快捷函数。

`render_to_string`(_template_name[, context][, context_instance][, request][, using]_)[[source]](../_modules/django/template/loader.html#render_to_string)

`render_to_string()` 会像 [`get_template()`](#django.template.loader.get_template "django.template.loader.get_template")一样加载模板并立即调用 `render()` 方法。 它需要以下参数。

`template_name`

The name of the template to load and render. If it’s a list of template names, Django uses [`select_template()`](#django.template.loader.select_template "django.template.loader.select_template") instead of [`get_template()`](#django.template.loader.get_template "django.template.loader.get_template") to find the template.

`context`

要用作模板的上下文进行渲染的[`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.4)")。

Changed in Django 1.8:

用于调用`dictionary`的`context`参数。该名称在Django 1.8中已弃用，将在Django 2.0中删除。

`context`现在是可选的。如果没有提供空的上下文将被使用。

`context_instance`

一个[`Context`](../ref/templates/api.html#django.template.Context "django.template.Context") t实例，或者是子类的实例（例如，[`RequestContext`](../ref/templates/api.html#django.template.RequestContext "django.template.RequestContext")的实例），可以像template的context一样使用它。

自1.8版起已弃用：`context_instance`已经被废弃。使用`context`和如果需要`request`。

`request`

[`HttpRequest`](../ref/request-response.html#django.http.HttpRequest "django.http.HttpRequest")是可选的，并且在整个模版渲染期都是可用的。 

New in Django 1.8:

已添加`request`参数。

同时请看[`render()`](http/shortcuts.html#django.shortcuts.render "django.shortcuts.render") 和[`render_to_response()`](http/shortcuts.html#django.shortcuts.render_to_response "django.shortcuts.render_to_response")快捷方式，它们调用[`render_to_string()`](#django.template.loader.render_to_string "django.template.loader.render_to_string") ，并将渲染结果放入到一个合适的[`HttpResponse`](../ref/request-response.html#django.http.HttpResponse "django.http.HttpResponse") 中用以从view中返回

最后，您可以直接使用配置的引擎：

`engines`

模板引擎可在`django.template.engines`中使用：

```
from django.template import engines

django_engine = engines['django']
template = django_engine.from_string("Hello {{ name }}!")

```

在此示例中，查找键 - `'django'`是引擎的[`NAME`](../ref/settings.html#std:setting-TEMPLATES-NAME)。

### 内置(模板)后端

_class_ `DjangoTemplates`[[source]](../_modules/django/template/backends/django.html#DjangoTemplates)

设置[`BACKEND`](../ref/settings.html#std:setting-TEMPLATES-BACKEND) 为 `'django.template.backends.django.DjangoTemplates'` 来配置Django模板引擎。

当 [`APP_DIRS`](../ref/settings.html#std:setting-TEMPLATES-APP_DIRS) 为 `True` 时, `DjangoTemplates` 引擎会在已安装应用的 `templates` 子目录中查找模板文件。 这个通用名称是保持向后兼容的。

`DjangoTemplates` 引擎 [`OPTIONS`](../ref/settings.html#std:setting-TEMPLATES-OPTIONS) 配置项中接受以下参数:

*   `'allowed_include_roots'`: 这是一个字符串列表，表示这些字符串允许出现在`{% ssi %}`中，作为被允许使用的模板标签。这是一个安全的措施，使模板作者不能访问他们不应该访问的文件。

    例如, 如果 `'allowed_include_roots'` 是 `['/home/html', '/var/www']`,那么 `{% ssi /home/html/foo.txt %}` 就会生效, 但是 `{% ssi /etc/passwd %}`就无效。

    它默认是个空的 list.

    自1.8版起已弃用：`allowed_include_roots`已弃用，因为已弃用{％ssi％}标记。

*   `'context_processors'`: 是一个包含以"."为分隔符的python调用路径的列表，在一个template被request渲染时，它可以被调用以产生context的数据。这些可调用要求一个请求对象作为其参数，并返回要合并到上下文中的项目的[`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.4)")。

    它默认为空列表。

    有关详细信息，请参阅[`RequestContext`](../ref/templates/api.html#django.template.RequestContext "django.template.RequestContext")。

*   `'debug'`：打开/关闭模板调试模式的布尔值。如果它`True`，那么奇怪的错误页面将显示模板渲染期间引发的任何异常的详细报告。此报告包含模板的相关代码段，并突出显示相应的行。

    它默认和setting中的 [`DEBUG`](../ref/settings.html#std:setting-DEBUG)有相同的值。

*   `'loaders'`：模板加载器类的虚拟Python路径列表。每个`Loader`类知道如何从特定源导入模板。你可以选择使用字符串元组来代替字符串。元组的第一项是 `Loader`类名，接下来的项在初始化期间会被传递给`Loader`。

    默认值取决于[`DIRS`](../ref/settings.html#std:setting-TEMPLATES-DIRS)和[`APP_DIRS`](../ref/settings.html#std:setting-TEMPLATES-APP_DIRS)的值。

    有关详细信息，请参见[_Loader types_](../ref/templates/api.html#template-loaders)。

*   `'string_if_invalid'`：作为字符串的输出，模板系统应该用于无效（例如拼写错误的）变量。

    它默认为空字符串。

    有关详细信息，请参见[_How invalid variables are handled_](../ref/templates/api.html#invalid-template-variables)。

*   `'file_charset'`：用于读取磁盘上的模板文件的字符集。

    它默认为[`FILE_CHARSET`](../ref/settings.html#std:setting-FILE_CHARSET)的值。

_class_ `Jinja2`[[source]](../_modules/django/template/backends/jinja2.html#Jinja2)

需要安装 [Jinja2](http://jinja.pocoo.org/) :

```
$ pip install Jinja2

```

设置 [`BACKEND`](../ref/settings.html#std:setting-TEMPLATES-BACKEND) 为 `'django.template.backends.jinja2.Jinja2'` 时，则启用 [Jinja2](http://jinja.pocoo.org/) 引擎。

当 [`APP_DIRS`](../ref/settings.html#std:setting-TEMPLATES-APP_DIRS) 为 `True`时, `Jinja2` 引擎则在已安装应用的名为 `jinja2` 的子目录中查找模板文件。

最重要的配置项是 [`OPTIONS`](../ref/settings.html#std:setting-TEMPLATES-OPTIONS) 中的 `'environment'`项。它应该是一个带点的Python路径，调用它则可返回 Jinja2 的环境变量。它的默认值是 `'jinja2.`Environment'。Django调用该可调用项并传递其他选项作为关键字参数。此外，Django添加了与Jinja2不同的默认值，用于几个选项：

*   `'autoescape'`：`True`
*   `'loader'`：为[`DIRS`](../ref/settings.html#std:setting-TEMPLATES-DIRS)和[`APP_DIRS`](../ref/settings.html#std:setting-TEMPLATES-APP_DIRS)
*   `'auto_reload'`：`settings.`调试
*   `'undefined'`：`DebugUndefined if settings。`DEBUG else 未定义

有意将默认配置保持为最小。`Jinja2`后端不会创建Django风格的环境。它不知道Django上下文处理器，过滤器和标签。为了使用Django特定的API，您必须将它们配置到环境中。

例如，你可以创建一个包含以下内容的`myproject/jinja2.py` ：

```
from __future__ import absolute_import  # Python 2 only

from django.contrib.staticfiles.storage import staticfiles_storage
from django.core.urlresolvers import reverse

from jinja2 import Environment

def environment(**options):
    env = Environment(**options)
    env.globals.update({
        'static': staticfiles_storage.url,
        'url': reverse,
    })
    return env

```

并将`'environment'`选项设置为`'myproject.jinja2.environment'`。

之后你可以用下面的内容构建Jinja2模板：

```
<img src="{{ static('path/to/company-logo.png') }}" alt="Company Logo">

<a href="{{ url('admin:index') }}">Administration</a>

```

标签和过滤器的概念尽管在Django模板语言和Jinja2中都存在，但使用起来却不相同。因为Jinja2支持传递参数给模版中的可调用对象，在Django中许多需要模版标记和过滤的特性，在Jinja2可以通过简单的调用函数来得到，比如上面所显示的例子。Jinja2的全局命名空间移除了模版上下文处理器的需求。Django模版语言没有等同于Jinja2 tests的东西。

### 定制后端

为了使用其他的模板系统，这里将介绍如何实现一个自定义模板后台。模板后台都是继承自`django.template.backends.base.BaseEngine`类的。它必须实现 `get_template()`方法，也可实现可选函数`from_string()`。下面是一个虚构的`foobar`模板库的示例：

```
from django.template import TemplateDoesNotExist, TemplateSyntaxError
from django.template.backends.base import BaseEngine
from django.template.backends.utils import csrf_input_lazy, csrf_token_lazy

import foobar

class FooBar(BaseEngine):

    # Name of the subdirectory containing the templates for this engine
    # inside an installed application.
    app_dirname = 'foobar'

    def __init__(self, params):
        params = params.copy()
        options = params.pop('OPTIONS').copy()
        super(FooBar, self).__init__(params)

        self.engine = foobar.Engine(**options)

    def from_string(self, template_code):
        try:
          return Template(self.engine.from_string(template_code))
        except foobar.TemplateCompilationFailed as exc:
            raise TemplateSyntaxError(exc.args)

    def get_template(self, template_name):
        try:
            return Template(self.engine.get_template(template_name))
        except foobar.TemplateNotFound as exc:
            raise TemplateDoesNotExist(exc.args)
        except foobar.TemplateCompilationFailed as exc:
            raise TemplateSyntaxError(exc.args)

class Template(object):

    def __init__(self, template):
        self.template = template

    def render(self, context=None, request=None):
        if context is None:
            context = {}
        if request is not None:
            context['request'] = request
            context['csrf_input'] = csrf_input_lazy(request)
            context['csrf_token'] = csrf_token_lazy(request)
        return self.template.render(context)

```

有关详细信息，请参阅[DEP 182](https://github.com/django/deps/blob/master/accepted/0182-multiple-template-engines.rst)。

## Django模板语言

### 语法

关于本节

以下是关于Django模板语法的概览。更多细节请见 [_模板语言语法参考_](../ref/templates/language.html).

Django模板是一个简单的文本文档，或用Django模板语言标记的一个Python字符串。 某些结构是被模板引擎解释和识别的。主要的有变量和标签。

模板是由context来进行渲染的。渲染的过程是用在context中找到的值来替换模板中相应的变量，并执行相关tags。其他的一切都原样输出。

Django模板语言的语法包括四个结构。

#### 变量

变量的值是来自context中的输出, 这类似于字典对象的keys到values的映射关系。

变量是被 `{{` 和 `}}`括起来的部分，例如：

```
My first name is {{ first_name }}. My last name is {{ last_name }}.

```

如果使用一个 context包含 `{'first_name': 'John', 'last_name': 'Doe'}`, 这个模板渲染后的情况将是:

```
My first name is John. My last name is Doe.

```

字典查询，属性查询和列表索引查找都是通过一个点符号来实现：

```
{{ my_dict.key }}
{{ my_object.attribute }}
{{ my_list.0 }}

```

如果一个变量被解析为一个可调用的，模板系统会调用它不带任何参数，并使用调用它的结果来代替这个可调用对象本身。

#### 标签

标签在渲染的过程中提供任意的逻辑。

这个定义是刻意模糊的。例如，一个标签可以输出内容，作为控制结构，例如“if”语句或“for”循环从数据库中提取内容，甚至可以访问其他的模板标签。

Tags是由`{%`和 `%}` 来定义的，例如：

```
{% csrf_token %}

```

大部分标签都接受参数

```
{%< cycle 'odd' 'even' %}

```

部分标签要求使用起始和闭合标签：

```
{%< if user.is_authenticated %}Hello, {{ user.username }}.{%< endif %}

```

[_Django内置标签参考文档_](../ref/templates/builtins.html#ref-templates-builtins-tags) 和 [_编写定制化标签指引_](../howto/custom-template-tags.html#howto-writing-custom-template-tags)都可以参阅。

#### 过滤器

过滤器会更改变量或标签参数的值。

看上去像这样：

```
{{ django|title }}

```

例如在 `{'django': 'the web framework for perfectionists with deadlines'}`这个context中，django变量的值都是小写，经title过滤器渲染后则变成：

```
The Web Framework For Perfectionists With Deadlines

```

有些过滤器看起来更像参数：

```
{{ my_date|date:"Y-m-d" }}

```

具体可以查看 [_内置过滤器参考_](../ref/templates/builtins.html#ref-templates-builtins-filters)和 [_开发自定义过滤器指南_](../howto/custom-template-tags.html#howto-writing-custom-template-filters)这两篇文档.

#### 注释

注释看起来像这样:{# this won't be rendered #}

A [`{% comment %}`](../ref/templates/builtins.html#std:templatetag-comment) 标签提供多行注释。

### 组件

关于这个部分

本文只是对Django模板语言API作一个简单概述. 具体请参阅 [_API参考指引_](../ref/templates/api.html).

#### 引擎

[`django.template.Engine`](../ref/templates/api.html#django.template.Engine "django.template.Engine") 封装Django模板系统的一个实例。其主要原因是直接实例化一个 [`引擎`](../ref/templates/api.html#django.template.Engine "django.template.Engine") 在Django项目之外进行使用。

[`django.template.backends.django.DjangoTemplates`](#django.template.backends.django.DjangoTemplates "django.template.backends.django.DjangoTemplates")是[`django.template.Engine`](../ref/templates/api.html#django.template.Engine "django.template.Engine")的简化版,他们都是指向Django模板后端的API。

#### 模板

[`django.template.Template`](../ref/templates/api.html#django.template.Template "django.template.Template") 表示一个已编译的模板。模板可以通过 [`Engine.get_template()`](../ref/templates/api.html#django.template.Engine.get_template "django.template.Engine.get_template")或 [`Engine.from_string()`](../ref/templates/api.html#django.template.Engine.from_string "django.template.Engine.from_string")来获取。

同样地, `django.template.backends.django.Template` 是[`django.template.Template`](../ref/templates/api.html#django.template.Template "django.template.Template")的一个简化版，他们指向共同的模板API.

#### 上下文

[`django.template.Context`](../ref/templates/api.html#django.template.Context "django.template.Context") 除了持有context数据以外，还持有一些元数据. 它被传递到[`Template.render()`](../ref/templates/api.html#django.template.Template.render "django.template.Template.render")用于渲染一个模板.

[`django.template.RequestContext`](../ref/templates/api.html#django.template.RequestContext "django.template.RequestContext") 是[`Context`](../ref/templates/api.html#django.template.Context "django.template.Context")的一个子类，它存储当前的[`HttpRequest`](../ref/request-response.html#django.http.HttpRequest "django.http.HttpRequest")并且运行了模板的context处理器.

一般的API不具备这样的概念。大多数情况下，context数据是在一个普通的[`字典`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.4)")里传递，而当前的[`HttpRequest`](../ref/request-response.html#django.http.HttpRequest "django.http.HttpRequest") 如果有需要的话是另外传递的。

#### 加载器

模板加载器负责定位模板，加载它们，并返回[`模板`](../ref/templates/api.html#django.template.Template "django.template.Template")对象.

Django提供几个[_内置的模板加载器_](../ref/templates/api.html#template-loaders)并且支持[_自定义的模板加载器_](../ref/templates/api.html#custom-template-loaders).

#### 上下文处理器

Context处理器是这样的函数：接收当前的 [`HttpRequest`](../ref/request-response.html#django.http.HttpRequest "django.http.HttpRequest") 作为参数，并返回一个 [`字典`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.4)")，该字典中包含了将要添加到渲染的context中的数据。

它们的主要用途是添加所有的模板context共享的公共数据，而不需要在每个视图中重复代码。

Django提供了很多 [_内置的context处理器_ ](../ref/templates/api.html#context-processors). 实现自定义context处理器很简单，只要定义一个函数。

{% endraw %}