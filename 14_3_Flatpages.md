

# 简单的页面应用程序

Django有一个可选的“简单页面”的应用它可以让你存储简单的扁平化结构的HTML内容在数据库中，你可以通过Django的管理界面和一个Python API处理要管理内容。

一个浮动页面是一个简单的包含有URL，标题和内容的对象。使用它作为一次性的，特殊用途的页面，比如“关于我们”或者“隐私政策”的页面，那些你想要保存在数据库，但是你又不想开发一个自定义的Django应用。

一个简单的页面应用程序可以使用自定义模板或者默认模板，系统的单页面模板。它可以和一个或多个站点相关联。

如果您希望将内容放在自定义模板中，该内容字段可能会任意的留下空白。

这里有一些基于Django的简单页面的示例：

*   [http://www.lawrence.com/about/contact/](http://www.lawrence.com/about/contact/)
*   [http://www2.ljworld.com/site/rules/](http://www2.ljworld.com/site/rules/)

## 安装

按照下面这些步骤，安装单页面的应用

1.  通过向您的[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS)添加`'django.contrib.sites'`，安装[`sites framework`](sites.html#module-django.contrib.sites "django.contrib.sites: Lets you operate multiple Web sites from the same database and Django project")

    另外，请确保您已将[`SITE_ID`](../settings.html#std:setting-SITE_ID)正确设置为设置文件所代表的网站的ID。这通常是`1`（即`SITE_ID = 1`重新使用sites框架来管理多个网站，它可以是不同网站的ID。

2.  将`'django.contrib.flatpages'`添加到您的[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS)设置。

然后：

1.  在URLconf中添加条目。例如：

    ```
    urlpatterns = [
        url(r'^pages/', include('django.contrib.flatpages.urls')),
    ]

    ```

要么：

1.  将`'django.contrib.flatpages.middleware.FlatpageFallbackMiddleware'`添加到您的[`MIDDLEWARE_CLASSES`](../settings.html#std:setting-MIDDLEWARE_CLASSES)设置。
2.  运行命令[`manage.py migrate`](../django-admin.html#django-admin-migrate)。

## 怎么运行的

`manage.py migrate`在数据库中创建两个表：`django_flatpage`和`django_flatpage_sites`。`django_flatpage`是一个简单的查找表，只是将URL映射到标题和一堆文本内容。`django_flatpage_sites`将平面页面与网站相关联。

### 使用URLconf

有多种方法可以将平面网页纳入您的URLconf。您可以将特定路径专用于平面页面：

```
urlpatterns = [
    url(r'^pages/', include('django.contrib.flatpages.urls')),
]

```

您还可以将其设置为“总线”模式。在这种情况下，将模式放置在其他url模式的末尾非常重要：

```
from django.contrib.flatpages import views

# Your other patterns here
urlpatterns += [
    url(r'^(?P<url>.*/)$', views.flatpage),
]

```

警告

如果您将[`APPEND_SLASH`](../settings.html#std:setting-APPEND_SLASH)设置为`False`，则必须删除catchall模式中的斜杠，否则不会匹配尾部斜杠。

另一个常见的设置是为一组有限的已知网页使用平面网页，并对网址进行硬编码，因此您可以使用[`url`](../templates/builtins.html#std:templatetag-url)模板标记来引用它们：

```
from django.contrib.flatpages import views

urlpatterns += [
    url(r'^about-us/$', views.flatpage, {'url': '/about-us/'}, name='about'),
    url(r'^license/$', views.flatpage, {'url': '/license/'}, name='license'),
]

```

### 使用中间件

[`FlatpageFallbackMiddleware`](#django.contrib.flatpages.middleware.FlatpageFallbackMiddleware "django.contrib.flatpages.middleware.FlatpageFallbackMiddleware")可以完成所有的工作。

_class_ `FlatpageFallbackMiddleware`[[source]](../../_modules/django/contrib/flatpages/middleware.html#FlatpageFallbackMiddleware)

每次任何Django应用程序引发404错误，此中间件检查flatpages数据库的请求的URL作为最后手段。Specifically, it checks for a flatpage with the given URL with a site ID that corresponds to the [`SITE_ID`](../settings.html#std:setting-SITE_ID) setting.

如果找到匹配，则遵循此算法：

*   如果flatpage有一个自定义模板，它将加载该模板。否则，它会加载模板`flatpages/default.html`。
*   它传递那个模板一个单一的上下文变量，`flatpage`，这是平面对象。它在呈现模板时使用[`RequestContext`](../templates/api.html#django.template.RequestContext "django.template.RequestContext")。

如果结果网址引用有效的平面网页，则中间件将仅添加尾部斜杠和重定向（通过查看[`APPEND_SLASH`](../settings.html#std:setting-APPEND_SLASH)设置）。重定向是永久的（301状态码）。

如果它没有找到匹配，请求继续照常处理。

中间件仅针对404s激活 - 不是500秒或任何其他状态代码的响应。

Flatpages将不应用视图中间件

由于`FlatpageFallbackMiddleware`仅在网址解析失败并生成404后才应用，因此返回的响应将不应用任何[_view middleware_](../../topics/http/middleware.html#view-middleware)方法。只有通过正常URL解析成功路由到视图的请求才应用视图中间件。

请注意，[`MIDDLEWARE_CLASSES`](../settings.html#std:setting-MIDDLEWARE_CLASSES)的顺序很重要。通常，您可以将[`FlatpageFallbackMiddleware`](#django.contrib.flatpages.middleware.FlatpageFallbackMiddleware "django.contrib.flatpages.middleware.FlatpageFallbackMiddleware")放在列表的结尾。这意味着它将在处理响应时首先运行，并确保任何其他响应处理中间件看到真实的平面响应，而不是404。

有关中间件的更多信息，请阅读[_middleware docs_](../../topics/http/middleware.html)。

确保您的404模板工作正常

请注意，[`FlatpageFallbackMiddleware`](#django.contrib.flatpages.middleware.FlatpageFallbackMiddleware "django.contrib.flatpages.middleware.FlatpageFallbackMiddleware")只会在另一个视图成功生成404响应时执行。如果另一个视图或中间件类尝试生成404，但最终会引发异常，则响应将变为HTTP 500（“内部服务器错误”），并且[`FlatpageFallbackMiddleware`](#django.contrib.flatpages.middleware.FlatpageFallbackMiddleware "django.contrib.flatpages.middleware.FlatpageFallbackMiddleware")将不会尝试提供平面页。

## 如何添加，更改和删除flatpages

### 通过管理界面

如果您已激活自动Django管理界面，您应该在管理索引页上看到一个“Flatpages”部分。在编辑系统中的任何其他对象时编辑平铺页。

### 通过Python API

_class_ `FlatPage`[[source]](../../_modules/django/contrib/flatpages/models.html#FlatPage)

平铺页由标准的[_Django model_](../../topics/db/models.html)表示，它位于[django / contrib / flatpages / models.py](https://github.com/django/django/blob/master/django/contrib/flatpages/models.py)中。您可以通过[_Django database API_](../../topics/db/queries.html)访问平面对象。

检查重复的平面网址。

如果您通过自己的代码添加或修改广告页，则可能需要检查同一网站中的重复的平展页网址。管理员使用的平面表单执行此验证检查，并且可以从`django.contrib.flatpages.forms.FlatPageForm`导入并在您自己的视图中使用。

## 平板模板

默认情况下，通过模板`flatpages/default.html`呈现平铺页，但您可以覆盖特定平面页：在管理中，标题为“高级选项”的折叠字段集（单击将其展开）包含用于指定模板名称的字段。如果您通过Python API创建平面网页，则可以简单地将模板名称设置为`FlatPage`对象上的字段`template_name`。

创建`flatpages/default.html`模板是您的责任；在模板目录中，只需创建一个包含文件`default.html`的`flatpages`目录。

平板模板传递单个上下文变量，`flatpage`，这是平面对象。

以下是一个示例`flatpages/default.html`模板：

```
<!DOCTYPE html>
<html>
<head>
<title>{{ flatpage.title }}</title>
</head>
<body>
{{ flatpage.content }}
</body>
</html>

```

由于您已将原始HTML输入到平面页面的管理页面，因此`flatpage.title`和`flatpage.content`都标记为**而不是** [_automatic HTML escaping_](../templates/language.html#automatic-html-escaping)。

## 获取列表[`FlatPage`](#django.contrib.flatpages.models.FlatPage "django.contrib.flatpages.models.FlatPage")

flatpages应用提供了一个模板代码，可让您遍历[_current site_](sites.html#hooking-into-current-site-from-views)上的所有可用平面网页。

Like all custom template tags, you’ll need to [_load its custom tag library_](../templates/language.html#loading-custom-template-libraries) before you can use it. 加载库后，您可以通过[`get_flatpages`](#std:templatetag-get_flatpages)标记检索所有当前的平铺页：

```
{% load flatpages %}
{% get_flatpages as flatpages %}
<ul>
    {% for page in flatpages %}
        <li><a href="{{ page.url }}">{{ page.title }}</a></li>
    {% endfor %}
</ul>

```

### 显示`registration_required`

默认情况下，[`get_flatpages`](#std:templatetag-get_flatpages)模板标签只会显示标记为`registration_required = False t3 &gt;。`如果要显示注册保护的纯页，则需要使用`for`子句指定已认证的用户。

例如：

```
{% get_flatpages for someuser as about_pages %}

```

如果您提供匿名用户，则[`get_flatpages`](#std:templatetag-get_flatpages)的行为将与您未提供用户的行为相同，即只显示公开的页面。

### 根据基本网址限制平面广告

可以应用可选参数`starts_with`，以将返回的页面限制为以特定基本URL开头的页面。此参数可以作为字符串传递，或作为要从上下文解析的变量传递。

例如：

```
{% get_flatpages '/about/' as about_pages %}
{% get_flatpages about_prefix as about_pages %}
{% get_flatpages '/about/' for someuser as about_pages %}

```

## 整合[`django.contrib.sitemaps`](sitemaps.html#module-django.contrib.sitemaps "django.contrib.sitemaps: A framework for generating Google sitemap XML files.")

_class_ `FlatPageSitemap`[[source]](../../_modules/django/contrib/flatpages/sitemaps.html#FlatPageSitemap)

[`sitemaps.`](#django.contrib.flatpages.sitemaps.FlatPageSitemap "django.contrib.flatpages.sitemaps.FlatPageSitemap")FlatPageSitemap类查看为当前[`SITE_ID`](../settings.html#std:setting-SITE_ID)定义的所有公开可见的[`flatpages`](#module-django.contrib.flatpages "django.contrib.flatpages: A framework for managing simple ?flat? HTML content in a database.")（请参阅[`sites documentation`](sites.html#module-django.contrib.sites "django.contrib.sites: Lets you operate multiple Web sites from the same database and Django project")这些条目仅包含[`location`](sitemaps.html#django.contrib.sitemaps.Sitemap.location "django.contrib.sitemaps.Sitemap.location")属性 - 不是[`lastmod`](sitemaps.html#django.contrib.sitemaps.Sitemap.lastmod "django.contrib.sitemaps.Sitemap.lastmod")，[`changefreq`](sitemaps.html#django.contrib.sitemaps.Sitemap.changefreq "django.contrib.sitemaps.Sitemap.changefreq")或[`priority`](sitemaps.html#django.contrib.sitemaps.Sitemap.priority "django.contrib.sitemaps.Sitemap.priority")。

Changed in Django 1.8:

此类可从旧版本的Django中的`django.contrib.sitemaps.FlatPageSitemap`获得。

### 示例

这里是一个使用[`FlatPageSitemap`](#django.contrib.flatpages.sitemaps.FlatPageSitemap "django.contrib.flatpages.sitemaps.FlatPageSitemap")的URLconf示例：

```
from django.conf.urls import url
from django.contrib.flatpages.sitemaps import FlatPageSitemap
from django.contrib.sitemaps.views import sitemap

urlpatterns = [
    # ...

    # the sitemap
    url(r'^sitemap\.xml$', sitemap,
        {'sitemaps': {'flatpages': FlatPageSitemap}},
        name='django.contrib.sitemaps.views.sitemap'),
]

```

