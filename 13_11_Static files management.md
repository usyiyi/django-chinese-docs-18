

# 静态文件应用

`django.contrib.staticfiles` 从你的应用（和其他你指定的地方）收集所有静态文件到同一个地方，这样产品就能很容易的被维护

看看这里

对于静态文件的应用和一些用法示例的介绍，请参阅[_管理静态文件（CSS，图像） _](../../howto/static-files/index.html). 如果你想知道如何部署静态文件, 请参阅 [_部署静态文件_](../../howto/static-files/deployment.html).

## 设置

查看[_staticfiles settings_](../settings.html#settings-staticfiles)了解更多设置细节

*   [STATIC_ROOT](../settings.html#std:setting-STATIC_ROOT)
*   [STATIC_URL](../settings.html#std:setting-STATIC_URL)
*   [STATICFILES_DIRS](../settings.html#std:setting-STATICFILES_DIRS)
*   [STATICFILES_STORAGE](../settings.html#std:setting-STATICFILES_STORAGE)
*   [STATICFILES_FINDERS](../settings.html#std:setting-STATICFILES_FINDERS)

## 管理命令

`django.contrib.staticfiles` 公开三个管理命令

### 搜集静态文件

`django-admin collectstatic`

搜集静态文件到 [`STATIC_ROOT`](../settings.html#std:setting-STATIC_ROOT).

默认情况下，重复文件名以类似于模板分辨率工作原理的方式解析：将使用首先在指定位置之一找到的文件。如果您感到困惑，[`findstatic`](#django-admin-findstatic)命令可以帮助您显示找到的文件。

使用[`enabled finders`](../settings.html#std:setting-STATICFILES_FINDERS)搜索文件。默认值是查看由[`STATICFILES_DIRS`](../settings.html#std:setting-STATICFILES_DIRS)中定义的所有位置以及在由[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS)设置指定的应用程序的`'static'`目录中定义的所有位置。

[`collectstatic`](#django-admin-collectstatic)管理命令在每次运行后调用[`STATICFILES_STORAGE`](../settings.html#std:setting-STATICFILES_STORAGE)的[`post_process()`](#django.contrib.staticfiles.storage.StaticFilesStorage.post_process "django.contrib.staticfiles.storage.StaticFilesStorage.post_process")方法，并传递管理所发现的路径列表命令。它还接收[`collectstatic`](#django-admin-collectstatic)的所有命令行选项。默认情况下，它由[`CachedStaticFilesStorage`](#django.contrib.staticfiles.storage.CachedStaticFilesStorage "django.contrib.staticfiles.storage.CachedStaticFilesStorage")使用。

默认情况下，收集的文件从[`FILE_UPLOAD_PERMISSIONS`](../settings.html#std:setting-FILE_UPLOAD_PERMISSIONS)接收权限，收集的目录从[`FILE_UPLOAD_DIRECTORY_PERMISSIONS`](../settings.html#std:setting-FILE_UPLOAD_DIRECTORY_PERMISSIONS)接收权限。如果您希望对这些文件和/或目录使用不同的权限，可以暂存[_static files storage classes_](#staticfiles-storages)，并指定`file_permissions_mode`和/或`directory_permissions_mode`例如：

```
from django.contrib.staticfiles import storage

class MyStaticFilesStorage(storage.StaticFilesStorage):
    def __init__(self, *args, **kwargs):
        kwargs['file_permissions_mode'] = 0o640
        kwargs['directory_permissions_mode'] = 0o760
        super(MyStaticFilesStorage, self).__init__(*args, **kwargs)

```

然后将[`STATICFILES_STORAGE`](../settings.html#std:setting-STATICFILES_STORAGE)设置为`'path.to.MyStaticFilesStorage'`。

New in Django 1.7:

覆盖`file_permissions_mode`和`directory_permissions_mode`的功能是Django 1.7中的新功能。以前，文件权限始终使用[`FILE_UPLOAD_PERMISSIONS`](../settings.html#std:setting-FILE_UPLOAD_PERMISSIONS)和始终使用的目录权限[`FILE_UPLOAD_DIRECTORY_PERMISSIONS`](../settings.html#std:setting-FILE_UPLOAD_DIRECTORY_PERMISSIONS)。

一些常用的选项是：

`--noinput`

不要提示用户输入任何类型。

`-i` `&lt;pattern&gt;`

`--ignore` `&lt;pattern&gt;`

忽略与此glob样式模式匹配的文件或目录。使用多次忽略更多。

`-n`

`--dry-run`

除了修改文件系统之外，执行所有操作。

`-c`

`--clear`

在尝试复制或链接原始文件之前清除现有文件。

`-l`

`--link`

创建指向每个文件的符号链接，而不是复制。

`--no-post-process`

不要调用配置的[`STATICFILES_STORAGE`](../settings.html#std:setting-STATICFILES_STORAGE)存储后端的[`post_process()`](#django.contrib.staticfiles.storage.StaticFilesStorage.post_process "django.contrib.staticfiles.storage.StaticFilesStorage.post_process")方法。

`--no-default-ignore`

不要忽略常见的私有glob样式模式`'CVS'`，`'.*'`和`'*~'`。

有关选项的完整列表，请参阅命令自己的帮助，运行：

```
$ python manage.py collectstatic --help

```

### findstatic

`django-admin findstatic`

使用已启用的查找器搜索一个或多个相对路径。

例如：

```
$ python manage.py findstatic css/base.css admin/js/core.js
Found 'css/base.css' here:
 /home/special.polls.com/core/static/css/base.css
 /home/polls.com/core/static/css/base.css
Found 'admin/js/core.js' here:
 /home/polls.com/src/django/contrib/admin/media/js/core.js

```

默认情况下，找到所有匹配的位置。要仅返回每个相对路径的第一个匹配，请使用`--first`选项：

```
$ python manage.py findstatic css/base.css --first
Found 'css/base.css' here:
 /home/special.polls.com/core/static/css/base.css

```

这是一个调试助手；它会显示给定路径将收集哪个静态文件。

通过将[`--verbosity`](../django-admin.html#django-admin-option---verbosity)标志设置为0，可以抑制额外的输出，只需获取路径名称：

```
$ python manage.py findstatic css/base.css --verbosity 0
/home/special.polls.com/core/static/css/base.css
/home/polls.com/core/static/css/base.css

```

另一方面，通过将[`--verbosity`](../django-admin.html#django-admin-option---verbosity)标志设置为2，可以获取所有搜索的目录：

```
$ python manage.py findstatic css/base.css --verbosity 2
Found 'css/base.css' here:
 /home/special.polls.com/core/static/css/base.css
 /home/polls.com/core/static/css/base.css
Looking in the following locations:
 /home/special.polls.com/core/static
 /home/polls.com/core/static
 /some/other/path/static

```

New in Django 1.7:

添加了搜索其目录的其他输出。

### runserver

`django-admin runserver`

Overrides the core [`runserver`](../django-admin.html#django-admin-runserver) command if the `staticfiles` app is [`installed`](../settings.html#std:setting-INSTALLED_APPS) and adds automatic serving of static files and the following new options.

`--nostatic`

使用`--nostatic`选项可以完全禁止使用应用程序提供静态文件。仅当应用位于项目的[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS)设置中时，此选项才可用。

用法示例：

```
django-admin runserver --nostatic

```

`--insecure`

使用`--insecure`选项强制使用应用程式提供静态档案，即使[`DEBUG`](../settings.html#std:setting-DEBUG)设定为`False`通过使用此功能，您可以确认**严重无效**以及可能**不安全**。这只适用于本地开发，应**从不用于生产**，并且仅当应用程序位于项目的[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS)设置时可用。[`runserver`](../django-admin.html#django-admin-runserver)`--insecure`不适用于[`CachedStaticFilesStorage`](#django.contrib.staticfiles.storage.CachedStaticFilesStorage "django.contrib.staticfiles.storage.CachedStaticFilesStorage")。

用法示例：

```
django-admin runserver --insecure

```

## 存储

### StaticFilesStorage

_class_ `storage.``StaticFilesStorage`

使用[`STATIC_ROOT`](../settings.html#std:setting-STATIC_ROOT)设置作为基本文件系统位置和[`STATIC_URL`](../settings.html#std:setting-STATIC_URL)设置的[`FileSystemStorage`](../files/storage.html#django.core.files.storage.FileSystemStorage "django.core.files.storage.FileSystemStorage")存储后端的子类分别作为基本URL。

`storage.StaticFilesStorage.``post_process`(_paths_, _**options_)

此方法在每次运行后由[`collectstatic`](#django-admin-collectstatic)管理命令调用，并将找到的文件的本地存储和路径作为字典以及命令行选项传递。

[`CachedStaticFilesStorage`](#django.contrib.staticfiles.storage.CachedStaticFilesStorage "django.contrib.staticfiles.storage.CachedStaticFilesStorage")在幕后使用它来替换路径与它们的哈希对等体，并适当地更新缓存。

### ManifestStaticFilesStorage

New in Django 1.7.

_class_ `storage.``ManifestStaticFilesStorage`

[`StaticFilesStorage`](#django.contrib.staticfiles.storage.StaticFilesStorage "django.contrib.staticfiles.storage.StaticFilesStorage")存储后端的子类，通过将文件内容的MD5哈希附加到文件名来存储它处理的文件名。例如，文件`css/styles.css`也将另存为`css/styles.55e7cbb9ba48.css`。

此存储的目的是为了在一些页面仍然引用这些文件的情况下继续提供旧文件，例如。因为它们由您或第三方代理服务器缓存。此外，如果您希望将[远期Expires标头](https://developer.yahoo.com/performance/rules.html#expires)应用于已部署的文件，以加快后续网页访问的加载时间，这将非常有帮助。

存储后端会自动使用缓存副本的路径（使用[`post_process()`](#django.contrib.staticfiles.storage.StaticFilesStorage.post_process "django.contrib.staticfiles.storage.StaticFilesStorage.post_process")方法）替换保存的文件中与其他已保存文件匹配的路径。默认情况下，用于查找这些路径（`django.contrib.staticfiles.storage.HashedFilesMixin.patterns`）的正则表达式涵盖[@import](http://www.w3.org/TR/CSS2/cascade.html#at-import)规则和[url() / t3&gt;](http://www.w3.org/TR/CSS2/syndata.html#uri) [级联样式表](http://www.w3.org/Style/CSS/)的语句。例如，`'css/styles.css'`文件带有内容

```
@import url("../admin/css/base.css");

```

将替换为调用`ManifestStaticFilesStorage`存储后端的[`url()`](../files/storage.html#django.core.files.storage.Storage.url "django.core.files.storage.Storage.url")方法，最终保存一个`'css/styles.55e7cbb9ba48.css'`具有以下内容：

```
@import url("../admin/css/base.27e20196a850.css");

```

要启用`ManifestStaticFilesStorage`，您必须确保满足以下要求：

*   [`STATICFILES_STORAGE`](../settings.html#std:setting-STATICFILES_STORAGE)设置设置为`'django.contrib.staticfiles.storage.ManifestStaticFilesStorage'`
*   [`DEBUG`](../settings.html#std:setting-DEBUG)设置为`False`
*   您可以使用`staticfiles` [`static`](#std:templatetag-staticfiles-static)模板标记来引用模板中的静态文件
*   您已使用[`collectstatic`](#django-admin-collectstatic)管理命令收集了所有静态文件

由于创建MD5哈希值会对运行时的网站造成负担，因此`staticfiles`会自动将所有已处理文件的哈希值映射存储在名为`staticfiles.json` 。当您运行[`collectstatic`](#django-admin-collectstatic)管理命令时，会发生这种情况。

由于运行[`collectstatic`](#django-admin-collectstatic)的要求，在运行测试时，通常不应使用此存储器，因为`collectstatic`不作为正常测试设置的一部分运行。在测试期间，请确保[`STATICFILES_STORAGE`](../settings.html#std:setting-STATICFILES_STORAGE)设置设置为像`'django.contrib.staticfiles.storage.StaticFilesStorage'`（默认值）。

`storage.ManifestStaticFilesStorage.``file_hash`(_name_, _content=None_)

创建文件的散列名称时使用的方法。需要返回给定文件名和内容的哈希值。默认情况下，它从内容的块计算MD5哈希，如上所述。随意覆盖此方法使用自己的哈希算法。

### CachedStaticFilesStorage

_class_ `storage.``CachedStaticFilesStorage`

`CachedStaticFilesStorage`类似于[`ManifestStaticFilesStorage`](#django.contrib.staticfiles.storage.ManifestStaticFilesStorage "django.contrib.staticfiles.storage.ManifestStaticFilesStorage")类，但使用Django的[_caching framework_](../../topics/cache.html)来存储处理文件的哈希名称，而不是静态清单文件`staticfiles.json`。这在您无权访问文件系统的情况下非常有用。

如果要覆盖存储使用的高速缓存后端的某些选项，只需在名为`'staticfiles'`的[`CACHES`](../settings.html#std:setting-CACHES)设置中指定自定义条目即可。它会回到使用`'default'`缓存后端。

## 模板标签

### 静态的

使用配置的[`STATICFILES_STORAGE`](../settings.html#std:setting-STATICFILES_STORAGE)存储来为给定的相对路径创建完整的网址，例如：

```
{% load static from staticfiles %}
<img src="{% static "images/hi.jpg" %}" alt="Hi!" />

```

上一个示例等于使用`"images/hi.jpg"`调用[`STATICFILES_STORAGE`](../settings.html#std:setting-STATICFILES_STORAGE)实例的`url`方法。这在使用非本地存储后端部署文件时特别有用，如[_Serving static files from a cloud service or CDN_](../../howto/static-files/deployment.html#staticfiles-from-cdn)提供静态文件中所述。

如果您希望在不显示静态网址的情况下检索静态网址，则可以使用略有不同的调用：

```
{% load static from staticfiles %}
{% static "images/hi.jpg" as myphoto %}
<img src="{{ myphoto }}" alt="Hi!" />

```

## 查找模块

`staticfiles`查找器具有`searched_locations`属性，它是查找器搜索的目录路径的列表。用法示例：

```
from django.contrib.staticfiles import finders

result = finders.find('css/base.css')
searched_locations = finders.searched_locations

```

New in Django 1.7.

已添加`searched_locations`属性。

## 其他帮助

除了[`staticfiles`](#module-django.contrib.staticfiles "django.contrib.staticfiles: An app for handling static files.")应用程序之外还有一些其他助手可以使用静态文件：

*   [`django.template.context_processors.static()`](../templates/api.html#django.template.context_processors.static "django.template.context_processors.static")上下文处理器，将[`STATIC_URL`](../settings.html#std:setting-STATIC_URL)添加到用[`RequestContext`](../templates/api.html#django.template.RequestContext "django.template.RequestContext")上下文渲染的每个模板上下文。
*   内置模板标记[`static`](../templates/builtins.html#std:templatetag-static)，它接受一个路径，并使用静态前缀[`STATIC_URL`](../settings.html#std:setting-STATIC_URL)将其链接。
*   内置模板标记[`get_static_prefix`](../templates/builtins.html#std:templatetag-get_static_prefix)用于将模板变量填充为静态前缀[`STATIC_URL`](../settings.html#std:setting-STATIC_URL)以用作变量或直接。
*   类似的模板标签[`get_media_prefix`](../templates/builtins.html#std:templatetag-get_media_prefix)，其工作方式类似于[`get_static_prefix`](../templates/builtins.html#std:templatetag-get_static_prefix)，但使用[`MEDIA_URL`](../settings.html#std:setting-MEDIA_URL)。

### 静态文件开发视图

静态文件工具主要用于帮助将静态文件成功部署到生产环境中。这通常意味着一个单独的，专用的静态文件服务器，这在开发本地时是很麻烦的开销。因此，`staticfiles`应用程序附带了一个**快速和脏的帮助视图**，您可以使用它在开发中本地提供文件。

`views.``serve`(_request_, _path_)

此视图函数在开发中提供静态文件。

警告

此视图仅在[`DEBUG`](../settings.html#std:setting-DEBUG)为`True`时有效。

这是因为此视图**严重无效**，可能**不安全**。这只适用于本地开发，应**从不用于生产**。

Changed in Django 1.7:

当[`DEBUG`](../settings.html#std:setting-DEBUG)为`False`时，此视图现在将引发[`Http404`](../../topics/http/views.html#django.http.Http404 "django.http.Http404")异常而不是[`ImproperlyConfigured`](../exceptions.html#django.core.exceptions.ImproperlyConfigured "django.core.exceptions.ImproperlyConfigured")。

注意

要猜测提供的文件的内容类型，此视图依赖于来自Python标准库的[`mimetypes`](https://docs.python.org/3/library/mimetypes.html#module-mimetypes "(in Python v3.4)")模块，该模块本身依赖于底层平台的映射文件。如果您发现此视图没有为特定文件返回正确的内容类型，则很可能是平台的地图文件需要更新。这可以通过在Debian发行版上安装或更新Red Hat发行版上的`mailcap`软件包或`mime-support`来实现。

此视图由[`runserver`](../django-admin.html#django-admin-runserver)自动启用（[`DEBUG`](../settings.html#std:setting-DEBUG)设置为`True`）。要使用不同本地开发服务器的视图，请将以下代码段添加到主URL配置的末尾：

```
from django.conf import settings
from django.contrib.staticfiles import views

if settings.DEBUG:
    urlpatterns += [
        url(r'^static/(?P<path>.*)$', views.serve),
    ]

```

注意，模式（`r'^ static /'`）的开头应该是你的[`STATIC_URL`](../settings.html#std:setting-STATIC_URL)设置。

由于这有点麻烦，还有一个帮助函数，将为你做这个：

`urls.``staticfiles_urlpatterns`()

这将返回正确的URL模式，用于将静态文件提供给已定义的模式列表。使用它像这样：

```
from django.contrib.staticfiles.urls import staticfiles_urlpatterns

# ... the rest of your URLconf here ...

urlpatterns += staticfiles_urlpatterns()

```

这将检查您的[`STATIC_URL`](../settings.html#std:setting-STATIC_URL)设置，并将视图连接到相应的静态文件。不要忘记适当地设置[`STATICFILES_DIRS`](../settings.html#std:setting-STATICFILES_DIRS)设置，让`django.contrib.staticfiles`知道除了应用程序目录中的文件之外还要查找文件的位置。

警告

只有[`DEBUG`](../settings.html#std:setting-DEBUG) 设置为 `True` ，并且 [`STATIC_URL`](../settings.html#std:setting-STATIC_URL) 设置不为空和完整的URL路径，比如 `http://static.example.com/`，这个帮助功能才会工作。

这是因为该视图 **效率不高** 并且很可能 **不安全**. 该视图仅适用于本地开发 ，而**不应该用于项目实际生产环境**.

### Specialized test case to support ‘live testing’

_class_ `testing.``StaticLiveServerTestCase`

这个单元测试TestCase子类扩展[`django.test.LiveServerTestCase`](../../topics/testing/tools.html#django.test.LiveServerTestCase "django.test.LiveServerTestCase")。

就像它的父类，你可以使用它来编写测试，涉及运行测试中的代码，并使用 HTTP测试工具（例如 Selenium，PhantomJS 等），因为它需要同时发布静态素材。

但是考虑到它使用了上面描述的[`django.contrib.staticfiles.views.serve()`](#django.contrib.staticfiles.views.serve "django.contrib.staticfiles.views.serve")视图，它可以在测试执行时透明地覆盖由`staticfiles` finders。这意味着您不需要在测试设置之前或作为测试设置的一部分运行[`collectstatic`](#django-admin-collectstatic)。

New in Django 1.7:

`StaticLiveServerTestCase`是Django 1.7中的新功能。以前，它的功能由[`django.test.LiveServerTestCase`](../../topics/testing/tools.html#django.test.LiveServerTestCase "django.test.LiveServerTestCase")提供。

