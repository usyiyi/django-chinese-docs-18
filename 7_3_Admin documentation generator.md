# Django管理文档生成器 #

Django的`admindocs`应用从模型、视图、模板标签以及模板过滤器中，为任何`INSTALLED_APPS`中的应用获取文档。并且让文档可以在`Django admin`中使用。

在某种程度上，你可以使用`admindocs`来快为你自己的代码生成文档。这个应用的功能十分有限，然而它主要用于文档模板、模板标签和过滤器。例如，需要参数的模型方法在文档中会有意地忽略，因为它们不能从模板中调用。这个应用仍旧有用，因为它并不需要你编写任何额外的文档（除了`docstrings`），并且在 `Django admin`中使用很方便。

## 概览 ##

要启用`admindocs`，你需要执行以下步骤：

+ 向 `INSTALLED_APPS`添加`django.contrib.admindocs`。
+ 向你的`urlpatterns`添加(`r'^admin/doc/'`, `include('django.contrib.admindocs.urls')`)。 确保它在`r'^admin/'` 这一项 之前包含，以便`/admin/doc/ `的请求不会被后面的项目处理。
+ 安装`docutils` Python 模块 (http://docutils.sf.net/)。
+ 可选的： 使用`admindocs`的书签功能需要安装`django.contrib.admindocs.middleware.XViewMiddleware`。

一旦完成这些步骤，你可以开始通过你的`admin`接口和点击在页面右上方的“Documentation”链接来浏览文档。

## 文档助手 ##

下列特定的标记可以用于你的`docstrings`，来轻易创建到其他组件的超链接：

Django Component | reStructuredText roles
-|-
Models | :model:\`app_label.ModelName\`
Views | :view:\`app_label.view_name\`
Template tags | :tag:\`tagname\`
Template filters | :filter:\`filtername\`
Templates | :template:\`path/to/template.html\`

## 模型参考 ##

`admindocs`页面的`models`部分描述了系统中每个模型，以及所有可用的字段和方法（不带任何参数）。虽然模型的属性没有任何参数，但他们没有列出。和其它模型的关联以超链接形式出现。描述由字段上的`help_text`属性，或者从模型方法的`docstrings`导出。

带有有用文档的模型看起来像是这样：

```
class BlogEntry(models.Model):
    """
    Stores a single blog entry, related to :model:`blog.Blog` and
    :model:`auth.User`.

    """
    slug = models.SlugField(help_text="A short label, generally used in URLs.")
    author = models.ForeignKey(User)
    blog = models.ForeignKey(Blog)
    ...

    def publish(self):
        """Makes the blog entry live on the site."""
        ...
```

## 视图参考 ##

你站点中的每个URL都在·页面中有一个单独的记录，点击提供的URL会向你展示相应的视图。有一些有用的东西，你可以在你的视图函数的·中记录：

+ 视图所做工作的一个简短的描述。
+ 上下文，或者是视图的模板中可用变量的列表。
+ 用于当前视图的模板的名称。

例如：

```
from django.shortcuts import render

from myapp.models import MyModel

def my_view(request, slug):
    """
    Display an individual :model:`myapp.MyModel`.

    **Context**

    ``mymodel``
        An instance of :model:`myapp.MyModel`.

    **Template:**

    :template:`myapp/my_template.html`

    """
    context = {'mymodel': MyModel.objects.get(slug=slug)}
    return render(request, 'myapp/my_template.html', context)
```

## 模板标签和过滤器参考 ##

`admindocs`的`tags` 和`filters`部分描述了Django自带的所有标签和过滤器（事实上，内建的标签参考 和 内建的过滤器参考文档直接来自于那些页面）。你创建的，或者由三方应用添加的任何标签或者过滤器，也会在这一部分中展示。

## 模板参考 ##

虽然`admindocs` 并不包含一个地方来保存模板，但如果你在结果页面中使用<code>:template:\`path/to/template.html\`</code>语法，会使用Django的模板加载器来验证该模板的路径。这是一个非常便捷的方法，来检查是否存在特定的模板，以及展示模板在文件系统的何处存放。

## 包含的书签 ##

`admindocs`页面上有一些很有用的书签：

Documentation for this page

Jumps you from any page to the documentation for the view that generates that page.

Show object ID

Shows the content-type and unique ID for pages that represent a single object.

Edit this object

Jumps to the admin page for pages that represent a single object.

为使用这些书签，你需要用带有`is_staff` 设置为 `True`的`User`登录`Django admin`，或者安装了`XViewMiddleware`并且你通过 `INTERNAL_IPS`中的IP地址访问站点。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Admin documentation generator](https://docs.djangoproject.com/en/1.8/ref/contrib/admin/admindocs/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
