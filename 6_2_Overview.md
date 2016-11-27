

# 应用

New in Django 1.7.

Django 包含一个由已安装应用组成的注册表，它保存应用的配置并提供自省。它还维护一个可用的[_模型_](../topics/db/models.html)的列表。

这个注册表叫做[`apps`](#django.apps.apps "django.apps.apps")，位于[`django.apps`](#module-django.apps "django.apps") 中：





```
>>> from django.apps import apps
>>> apps.get_app_config('admin').verbose_name
'Admin'

```







## 项目和应用

历史上，Django 使用**项目** 这个词来描述Django 安装的一个应用。现在项目主要通过一个设置模块定义。

**应用** 这个词表示一个Python 包，它提供某些功能的集合。应用可以在各种项目中重用。



注

最近，这个词有些令人困惑，因为使用词语“Web ?应用”来描述和Django 项目等同的事物渐渐变得常见。



应用包含模型、视图、模板、模板标签、静态文件、URL、中间件等等。它们一般通过[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 设置和其它例如URLconf、[`MIDDLEWARE_CLASSES`](settings.html#std:setting-MIDDLEWARE_CLASSES) 设置以及模板继承等机制接入到项目中。

Django 的应用只是一个代码集，它与框架的其它部分进行交互，理解这点很重要。没有类似一个`应用` 对象这样的东西。然而，有一些地方Django 需要与安装的应用交互，主要用于配置和自省。这是为什么应用注册表要在[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 实例中维持每个安装的应用的元数据。





## 配置应用

要配置一个应用，请子类化[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 并将这个子类的路径放在[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 中。

当[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 包含一个应用模块的路径后，Django 将在这个模块中检查一个`default_app_config` 变量。

如果这个变量有定义，它应该是这个应用的[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 子类的路径。

如果没有`default_app_config`，Django 将使用[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 基类。



### 对于应用的开发者

如果你正在创建一个名叫“Rock ’n’ roll”的可插式应用，下面展示了如何给admin提供一个合适的名字





```
# rock_n_roll/apps.py

from django.apps import AppConfig

class RockNRollConfig(AppConfig):
    name = 'rock_n_roll'
    verbose_name = "Rock ’n’ roll"

```





可以让你的应用像下面一样以默认方式加载这个[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig")的子类





```
# rock_n_roll/__init__.py

default_app_config = 'rock_n_roll.apps.RockNRollConfig'

```





这将使得[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 只包含`'rock_n_roll'` 时将使用`RockNRollConfig`。这允许你使用[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 功能而不用要求你的用户更新他们的[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 设置。

当然，你也可以告诉你的用户将`'rock_n_roll.apps.RockNRollConfig'` 放在他们的[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 设置中。 你甚至可以提供几个具有不同行为的[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 子类，并让使用者通过他们的 [`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 设置选择。

建议的惯例做法是将配置类放在应用的`apps` 子模块中。但是，Django 不强制这一点。

你必须包含[`name`](#django.apps.AppConfig.name "django.apps.AppConfig.name") 属性来让Django 决定该配置适用的应用。你可以定义[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") API 参考中记录的任何属性。



注

如果你的代码在应用的`__init__.py` 中导入应用程序注册表，名称`apps` 将与`apps` 子模块发生冲突。最佳做法是将这些代码移到子模块，并将其导入。一种解决方法是以一个不同的名称导入注册表︰





```
from django.apps import apps as django_apps

```











### 对于应用的使用者

如果你在`anthology`项目中使用"Rock ’n’ rol"，但您希望显示成"Gypsy jazz"，你可以提供你自己的配置︰





```
# anthology/apps.py

from rock_n_roll.apps import RockNRollConfig

class GypsyJazzConfig(RockNRollConfig):
    verbose_name = "Gypsy jazz"

# anthology/settings.py

INSTALLED_APPS = [
    'anthology.apps.GypsyJazzConfig',
    # ...
]

```





再说一次，在`apps` 子模块定义特定于项目的配置类是一项惯例，并不要求。







## 应用程序配置



_class_ `AppConfig`



应用程序配置对象存储应用程序的元数据。某些属性可以配置 [`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 子类中。其他由Django 设置且是只读的。







### 可配置的属性



`AppConfig.name`



应用的完整Python 路径，例如`django.contrib.admin`。

这个属性定义配置适用于哪个应用程序。它必须在所有 [`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 子类中设置。

它在整个Django 项目中必须是唯一的。







`AppConfig.label`



应用的缩写名称，例如`'admin'`。

此属性允许重新标记应用，当两个应用程序有冲突的标签。它默认为`name` 的最后一部分。它应该是一个有效的 Python 标识符。

它在整个Django 项目中必须是唯一的。







`AppConfig.verbose_name`



应用的适合阅读的名称，例如“Administration”。

此属性默认为`label.title()`。







`AppConfig.path`



应用目录的文件系统路径，如 `'/ usr/lib/python3.4/dist-packages/django/contrib/admin'`。

在大多数情况下，Django 可以自动检测并此设置，但你也可以在[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 子类上提供一个显式的类属性以覆盖它 。在有些情况下是需要这样的；例如，如果应用的包是一个具有多个路径的[命名空间包](#namespace-package)。









### 只读属性



`AppConfig.module`



应用的根模块，例如`<module 'django.contrib.admin' from 'django/contrib/admin/__init__.pyc'>`。







`AppConfig.models_module`



包含模型的模块，例如`<module 'django.contrib.admin.models' from 'django/contrib/admin/models.pyc'>`。

如果应用不包含`models`模块，它可能为`None`。注意与数据库相关的信号，如 [`pre_migrate`](signals.html#django.db.models.signals.pre_migrate "django.db.models.signals.pre_migrate") 和 [`post_migrate`](signals.html#django.db.models.signals.post_migrate "django.db.models.signals.post_migrate") 只有在应用具有`models` 模块时才发出。









### 方法



`AppConfig.get_models`()



返回可迭代的[`Model`](models/instances.html#django.db.models.Model "django.db.models.Model")类。







`AppConfig.get_model`(_model_name_)



返回具有给定 `model_name` 的 [`Model`](models/instances.html#django.db.models.Model "django.db.models.Model")。如果没有这种模型存在，抛出[`LookupError`](https://docs.python.org/3/library/exceptions.html#LookupError "(in Python v3.4)")。`model_name` 是不区分大小写。







`AppConfig.ready`()



子类可以重写此方法以执行初始化任务，如注册信号。它在注册表填充完之后立即调用。

你不可以在定义应用程序配置类的模块中导入模型中，但你可以使用 [`get_model()`](#django.apps.AppConfig.get_model "django.apps.AppConfig.get_model") 来通过名称访问模型类，像这样︰





```
def ready(self):
    MyModel = self.get_model('MyModel')

```







警告

虽然你可以如上文所述访问模型类，请避免与在你的[`ready()`](#django.apps.AppConfig.ready "django.apps.AppConfig.ready") 实现中与数据库交互。包括执行查询（[`save()`](models/instances.html#django.db.models.Model.save "django.db.models.Model.save")、[`delete()`](models/instances.html#django.db.models.Model.delete "django.db.models.Model.delete")、管理方法等）的模型方法，以及通过 `django.db.connection` 的原始SQL 查询。你的[`ready()`](#django.apps.AppConfig.ready "django.apps.AppConfig.ready") 方法将在每个管理命令启动期间运行。例如，即使测试数据库配置与生产设置是分离的，`manage.py test` 仍会执行一些针对您的 **production** 数据库的查询 ！





注

在常规的初始化过程中，`ready` 方法只由Django 调用一次。但在一些极端情况下，特别是在欺骗安装好的应用的测试，`ready` 可能被调用不止一次。在这种情况下，编写幂等方法，或者在`AppConfig` 类上放置一个标识来防止应该执行一次的代码重复运行。











### Namespace packages as apps (Python 3.3+)

Python versions 3.3 and later support Python packages without an `__init__.py` file. These packages are known as “namespace packages” and may be spread across multiple directories at different locations on `sys.path` (see [**PEP 420**](http://www.python.org/dev/peps/pep-0420)).

Django applications require a single base filesystem path where Django (depending on configuration) will search for templates, static assets, etc. Thus, namespace packages may only be Django applications if one of the following is true:

1.  The namespace package actually has only a single location (i.e. is not spread across more than one directory.)
2.  The [`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") class used to configure the application has a [`path`](#django.apps.AppConfig.path "django.apps.AppConfig.path") class attribute, which is the absolute directory path Django will use as the single base path for the application.

If neither of these conditions is met, Django will raise [`ImproperlyConfigured`](exceptions.html#django.core.exceptions.ImproperlyConfigured "django.core.exceptions.ImproperlyConfigured").







## 应用的注册表



`apps`



应用的注册表提供下列公共API。没有在下面列出的方法被认为是私有的，恕不另行通知。







`apps.ready`



布尔属性， 当填充完注册表设置为`True`。







`apps.get_app_configs`()



返回一个由[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig") 实例组成可迭代对象。







`apps.get_app_config`(_app_label_)



返回具有给定`app_label` 的[`AppConfig`](#django.apps.AppConfig "django.apps.AppConfig")。如果没有这种模型存在，抛出[`LookupError`](https://docs.python.org/3/library/exceptions.html#LookupError "(in Python v3.4)")。







`apps.is_installed`(_app_name_)



检查注册表中是否存在具有给定名称的应用。`app_name` 是应用的完整名称，例如`django.contrib.admin` 。







`apps.get_model`(_app_label_, _model_name_)



返回具有给定`app_label` 和`model_name`的[`Model`](models/instances.html#django.db.models.Model "django.db.models.Model")。作为快捷方式，此方法还接受`app_label.model_name` 形式的一个单一参数。`model_name` 不区分大小写。

如果没有这种模型存在，抛出[`LookupError`](https://docs.python.org/3/library/exceptions.html#LookupError "(in Python v3.4)")。使用不包含一个点的单个参数调用时将引发[`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError "(in Python v3.4)")。









## 初始化过程



### 应用是如何载入的

当Django 启动时，[`django.setup()`](#django.setup "django.setup") 负责填充应用注册表。



`setup`()[[source]](../_modules/django.html#setup)



配置Django：

*   加载设置。
*   设置日志。
*   初始化应用注册表。

自动调用此函数︰

*   当运行一个通过Django 的WSGI 支持 的HTTP 服务器。
*   当调用管理命令。

在其他情况下它必须显式调用，例如在普通的 Python 脚本中。





应用注册表初始化分三个阶段。在每个阶段，Django 以[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 中的顺序处理所有应用。

1.  首先，Django 会导入[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS)中的所有应用。

    如果它是一个应用配置类，Django 导入应用的根包，通过其[`name`](#django.apps.AppConfig.name "django.apps.AppConfig.name") 属性。如果它是一个Python 包，Django 创建应用的一个默认配置。

    _在这个阶段，你的代码不应该将任何模型导入！_

    换句话说，你的应用程序的根包和定义应用配置类的模块不应该导入任何模型，即使是间接导入。

    严格地讲，Django 允许应用配置加载后导入模型。然而，为了避免[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 的顺序带来不必要的约束，强烈推荐在这一阶段不导入任何模型。

    这一阶段完成后，操作应用配置的API 开始变得可用，例如[`get_app_config()`](#django.apps.apps.get_app_config "django.apps.apps.get_app_config")。

2.  然后Django 试图导入每个应用的`models` 子模块，如果有的话。

    你必须在应用的`models.py` 或`models/__init__.py` 中定义或导入所有模型。否则，应用注册表在此时可能不会完全填充，这可能导致ORM 出现故障。

    一旦完成该步骤, ?[`get_model()`](#django.apps.apps.get_model "django.apps.apps.get_model") 之类的 model API 可以使用了.

3.  最后，Django 运行每个应用程序配置的[`ready()`](#django.apps.AppConfig.ready "django.apps.AppConfig.ready") 方法。





### 故障排除

下面是一些在你初始化的时候可能经常碰到的问题:

*   `AppRegistryNotReady` 发生这种情况是当导入应用的配置或模型模块时触发取决于应用注册表的代码。

    例如，[`ugettext()`](utils.html#django.utils.translation.ugettext "django.utils.translation.ugettext") 使用应用注册表来查找应用中的翻译目录。若要在导入时翻译，你需要[`ugettext_lazy()`](utils.html#django.utils.translation.ugettext_lazy "django.utils.translation.ugettext_lazy")。（使用[`ugettext()`](utils.html#django.utils.translation.ugettext "django.utils.translation.ugettext") 将是一个bug，因为翻译会发生在导入的时候，而不是取决于每个请求的语言。）

    模型模块中在导入时使用ORM 执行数据库查询也会引发此异常。ORM 直到所有的模型都可用才能正常运转。

    另一个常见的罪魁祸首，是[`django.contrib.auth.get_user_model()`](../topics/auth/customizing.html#django.contrib.auth.get_user_model "django.contrib.auth.get_user_model")。请在导入时使用[`AUTH_USER_MODEL`](settings.html#std:setting-AUTH_USER_MODEL) 设置来引用用户模型。

    如果在一个独立的 Python 脚本中你忘了调用[`django.setup()`](#django.setup "django.setup")，也会发生此异常。

*   `ImportError: cannot import name ...` 如果导入出现循环，则会发生这种情况。

    要消除这种问题，应尽量减少模型模块之间的依赖项，并在导入时尽可能少做工作。为了避免在导入时执行代码，你可以移动它到一个函数和缓存其结果。当你第一次需要其结果时，将执行该代码。这一概念被称为"惰性求值"。

*   `django.contrib.admin` 在安装的应用中自动发现`admin`。要阻止它，请更改你的[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 以包含 `'django.contrib.admin.apps.SimpleAdminConfig'` 而不是 `'django.contrib.admin'`。





