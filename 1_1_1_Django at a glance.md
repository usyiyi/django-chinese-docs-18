{% raw %}

# Django 初探 #

由于Django是在一个快节奏的新闻编辑室环境下开发出来的，因此它被设计成让普通的网站开发工作简单而快 捷。以下简单介绍了如何用 Django 编写一个数据库驱动的Web应用程序。

本文档的目标是给你描述足够的技术细节能让你理解Django是如何工作的，但是它并不表示是一个新手指南或参考目录 – 其实这些我们都有! 当你准备新建一个项目，你可以 *从新手指南开始* 或者 *深入阅读详细的文档*.

## 设计你的模型(model) ##

尽管你在 Django 中可以不使用数据库，但是它提供了一个完善的可以用 Python 代码描述你的数据库结构的对象关联映射(ORM)。

*数据模型语法* 提供了许多丰富的方法来展现你的模型 – 到目前为止，它已经解决了两个多年积累下来数据库架构问题。下面是个简单的例子，可能被保存为 mysite/news/models.py:

```
class Reporter(models.Model):
    full_name = models.CharField(max_length=70)

    def __unicode__(self):
        return self.full_name

class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter)

    def __unicode__(self):
        return self.headline
```

## 安装它 ##

下一步，运行 Django 命令行工具来自动创建数据库表：

```
manage.py syncdb
```

syncdb 命令会查找你所有可用的模型(models)然后在你的数据库中创建还不存在的数据库表。

## 享用便捷的 API ##

接着，你就可以使用一个便捷且功能丰富的 *Python API* 来访问你的数据。API 是动态生成的，不需要代码生成:

```
# 导入我们在 "news "应用中创建的模型。
>>> from news.models import Reporter, Article

# 在系统中还没有 reporters 。
>>> Reporter.objects.all()
[]

# 创建一个新的 Reporter 。
>>> r = Reporter(full_name='John Smith')

# 将对象保存到数据库。你需要显示的调用 save() 方法。
>>> r.save()

# 现在它拥有了一个ID。
>>> r.id
1

# 现在新的 reporter 已经存在数据库里了。
>>> Reporter.objects.all()
[<Reporter: John Smith>]

# 字段被表示为一个 Python 对象的属性。
>>> r.full_name
'John Smith'

# Django 提供了丰富的数据库查询 API。
>>> Reporter.objects.get(id=1)
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__startswith='John')
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__contains='mith')
<Reporter: John Smith>
>>> Reporter.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Reporter matching query does not exist. Lookup parameters were {'id': 2}

# 创建一个 article.
>>> from datetime import date
>>> a = Article(pub_date=date.today(), headline='Django is cool',
...     content='Yeah.', reporter=r)
>>> a.save()

# 现在 article 已经存在数据库里了。
>>> Article.objects.all()
[<Article: Django is cool>]

# Article 对象有 API 可以访问到关联到 Reporter 对象。
>>> r = a.reporter
>>> r.full_name
'John Smith'

# 反之亦然：Reporter 对象也有访问 Article 对象的API。
>>> r.article_set.all()
[<Article: Django is cool>]

# API 会在幕后高效的关联表来满足你的关联查询的需求。
# 以下例子是找出名字开头为 "John" 的 reporter 的所有 articles 。
>>> Article.objects.filter(reporter__full_name__startswith="John")
[<Article: Django is cool>]

# 通过更改一个对象的属性值，然后再调用 save() 方法来改变它。
>>> r.full_name = 'Billy Goat'
>>> r.save()

# 调用 delete() 方法来删除一个对象。
>>> r.delete()
```

## 一个动态的管理接口：它不仅仅是个脚手架 – 还是个完整的房子 ##

一旦你的 models 被定义好，Django 能自动创建一个专业的，可以用于生产环境的 *管理界面* – 一个可让授权用户添加，修改和删除对象的网站。它使用起来非常简单只需在你的 admin site 中注册你的模型即可。:

```
# In models.py...

from django.db import models

class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter)


# In admin.py in the same directory...

import models
from django.contrib import admin

admin.site.register(models.Article)
```

这种设计理念是你的网站一般是由一个员工,或者客户，或者仅仅是你自己去编辑 – 而你应该不会想要仅仅为了管理内容而去创建后台界面。

在一个创建 Django 应用的典型工作流中，首先需要创建模型并尽可能快地启动和运行 admin sites， 让您的员工(或者客户)能够开始录入数据。然后,才开发展现数据给公众的方式。

## 设计你的 URLs ##

一个干净的，优雅的 URL 方案是一个高质量 Web 应用程序的重要细节。 Django 鼓励使用漂亮的 URL 设计，并且不鼓励把没必要的东西放到 URLs 里面，像 .php 或 .asp.

为了给一个 app 设计 URLs，你需要创建一个 Python 模块叫做 [URLconf](/topics/http/urls)。这是一个你的 app 内容目录， 它包含一个简单的 URL 匹配模式与 Python 回调函数间的映射关系。这有助于解耦 Python 代码和 URLs 。

这是针对上面 Reporter/Article 例子所配置的 URLconf 大概样子:

```
from django.conf.urls import patterns

urlpatterns = patterns('',
    (r'^articles/(\d{4})/$', 'news.views.year_archive'),
    (r'^articles/(\d{4})/(\d{2})/$', 'news.views.month_archive'),
    (r'^articles/(\d{4})/(\d{2})/(\d+)/$', 'news.views.article_detail'),
)
```

上面的代码映射了 URLs ，从一个简单的正则表达式，到 Python 回调函数(“views”)所在的位置。 正则表达式通过圆括号来“捕获” URLs 中的值。当一个用户请求一个页面时， Django 将按照顺序去匹配每一个模式，并停在第一个匹配请求的 URL 上。(如果没有匹配到， Django 将会展示一个404的错误页面。) 整个过程是极快的，因为在加载时正则表达式就进行了编译。

一旦有一个正则表达式匹配上了，Django 将导入和调用对应的视图，它其实就是一个简单的 Python 函数。每个视图将得到一个 request 对象 – 它包含了 request 的 meta 信息 – 和正则表达式所捕获到的值。

例如：如果一个用户请求了个 URL “/articles/2005/05/39323/”, Django 将会这样调用函数 news.views.article_detail(request, '2005', '05', '39323').

## 编写你的视图(views) ##

每个视图只负责两件事中的一件：返回一个包含请求页面内容的 HttpResponse 对象; 或抛出一个异常如 Http404 。至于其他就靠你了。

通常，一个视图会根据参数来检索数据，加载一个模板并且根据该模板来呈现检索出来的数据。 下面是个接上例的 year_archive 例子

```
def year_archive(request, year):
    a_list = Article.objects.filter(pub_date__year=year)
    return render_to_response('news/year_archive.html', {'year': year, 'article_list': a_list})
```

这个例子使用了 Django 的 [模板系统](/topics/templates)，该模板系统功能强大且简单易用，甚至非编程人员也会使用。

设计你的模板(templates)

上面的例子中载入了 news/year_archive.html 模板。

Django 有一个模板搜索路径板，它让你尽可能的减少冗余而重复利用模板。在你的 Django设置中，你可以指定一个查找模板的目录列表。如果一个模板没有在这个 列表中，那么它会去查找第二个，然后以此类推。

假设找到了模板 news/year_archive.html 。下面是它大概的样子:

```
{% extends "base.html" %}

{% block title %}Articles for {{ year }}{% endblock %}

{% block content %}
<h1>Articles for {{ year }}</h1>

{% for article in article_list %}
    <p>{{ article.headline }}</p>
    <p>By {{ article.reporter.full_name }}</p>
    <p>Published {{ article.pub_date|date:"F j, Y" }}</p>
{% endfor %}
{% endblock %}
```
变量使用双花括号包围。`{{ article.headline }} 表示 “输出 article 的 headline 属性”。而点符号不仅用于表示属性查找，还可用于字典的键值查找、索引查找和函数调用。

注意 `{{ article.pub_date|date:"F j, Y" }}` 使用了 Unix 风格的“管道”(“|”符合)。这就是所谓的模板过滤器，一种通过变量来过滤值的方式。本例中，Python datetime 对象被过滤成指定的格式(在 PHP 的日期函数中可以见到这种变换)。

你可以无限制地串联使用多个过滤器。你可以编写自定义的过滤器。你可以定制自 己的模板标记，在幕后运行自定义的 Python 代码。

最后，Django 使用了“模板继承”的概念：这就是 `{% extends "base.html" %}` 所做的事。它意味着 “首先载入名为 ‘base’ 的模板中的内容到当前模板，然后再处理本模板中的其余内容。”总之，模板继承让你在模板间大大减少冗余内容：每一个模板只需要定义它独特的部分即可。

下面是使用了 *静态文件* 的 “base.html” 模板的大概样子:

```
{% load staticfiles %}
<html>
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <img src="{% static "images/sitelogo.png" %}" alt="Logo" />
    {% block content %}{% endblock %}
</body>
</html>
```

简单地说，它定义了网站的外观（含网站的 logo ），并留下了个“洞”让子模板来填充。这使站点的重新设计变得非常容易，只需改变一个文件 – “base.html” 模板。

它也可以让你创建一个网站的多个版本，不同的基础模板，而重用子模板。 Django 的创建者已经利用这一技术来创造了显著不同的手机版本的网站 – 只需创建一个新的基础模板。

请注意，如果你喜欢其他模板系统，那么你可以不使用 Django 的模板系统。 虽然 Django 的模板系统特别集成了 Django 的模型层，但并没有强制你使用它。同理，你也可以不使用 Django 的数据库 API。您可以使用其他数据库抽象层，您可以读取 XML 文件，你可以从磁盘中读取文件，或任何你想要的方法去操作数据。 Django 的每个组成部分： 模型、视图和模板都可以解耦，以后会谈到。

## 这仅仅是一点皮毛 ##

这里只是简要概述了 Django 的功能。以下是一些更有用的功能：

一个 *缓存框架* 可以与 memcached 或其他后端缓存集成。
一个 *聚合框架* 可以让创建 RSS 和 Atom 的 feeds 同写一个小小的 Python 类一样容易。
更性感的自动创建管理站点功能 – 本文仅仅触及了点皮毛。
显然，下一步你应该 下载 Django，阅读 [入门教程](/intro/tutorial01) 并且加入 *社区*. 感谢您的关注！

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Overview](https://docs.djangoproject.com/en/1.8/intro/overview/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。

{% endraw %}
