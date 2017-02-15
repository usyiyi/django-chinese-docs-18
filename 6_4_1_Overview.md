

# django-admin和manage.py

`django-admin` 是用于管理Django的命令行工具集。本文档将概述它的全部功能。

Changed in Django 1.7:

在1.7之前的版本, `django-admin` 以 `django-admin.py`.存在

此外, `manage.py` 会在每个Django项目中自动生成。`manage.py` 是一个对`django-admin`的小包装，它可以在交付给`django-admin`之前为你做一些事情:

*   他把你的项目包放在python的系统目录中 `sys.path`。
*   它用于设置 [`DJANGO_SETTINGS_MODULE`](../topics/settings.html#envvar-DJANGO_SETTINGS_MODULE) 环境变量，因此它指向工程的  `settings.py` 文件。
*   通过调用[`django.setup()`](applications.html#django.setup "django.setup") 来初始化 Django 的内部变量。

New in Django 1.7:

Django1.7之前的版本没有[`django.setup()`](applications.html#django.setup "django.setup")

如果你是通过自带的 `setup.py`工具来安装Django 的，`django-admin` 脚本应该在你的系统路径里面。如果不在你的系统路径里面, 你应该能在你 Python 安装目录下的 `site-packages/django/bin` 里找到，可以创建一个软链接到你的系统路径里面, 比如 `/usr/local/bin`。

对于 Windows 用户, 由于不能使用软链接, 你可以拷贝 `django-admin.exe` 到你某个系统路径里面, 或者是将其添加到系统的 `PATH`里面 (在`设置 - 控制面板 - 系统 - 高级系统设置 - 环境变量...`)。

如果你在编写某个项目的时候, 通常使用`manage.py` 要比 `django-admin` 容易一些。如果你需要在不同的 Django 设置文件中来回切换，可以使用`django-admin` 加上 [`DJANGO_SETTINGS_MODULE`](../topics/settings.html#envvar-DJANGO_SETTINGS_MODULE) 或是 [`--settings`](#django-admin-option---settings) 参数。

这篇文档中的控制台操作示例中, 一直在使用 `django-admin` ，不过你想用 `manage.py` 也是可以的。

## 用法

```
$ django-admin <command> [options]
$ manage.py <command> [options]

```

`command` 应是本节文档中所列出的其中一项。`options`, 可选项，应该是可选值中的0项或多项。

### 获取运行时帮助

`django-admin help`

运行 `django-admin help`显示使用信息和每个应用的命令列表。

运行 `django-admin help --commands` 显示一个包含所有可用命令的列表

运行 `django-admin help &lt;command&gt;` 来显示某一个命令的描述及其可用的命令列表。

### 应用程序名称

许多地方都提及“app names”， “app name”是包含models的一个基本package单位。例如，如果你的 [`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS) 中包含 `'mysite.blog'`，那么app name就是 `blog`.

### 确定你使用的版本

`django-admin version`

键入 `django-admin version` 来获取你当前所使用的Django版本。

输出内容的格式在是[**PEP 386**](http://www.python.org/dev/peps/pep-0386):

```
1.4.dev17026
1.4a1
1.4

```

### 显示调试内容

使用[`--verbosity`](#django-admin-option---verbosity)可指定`django-admin`应打印到控制台的通知和调试信息量。有关详细信息，请参阅[`--verbosity`](#django-admin-option---verbosity)选项的文档。

## 可用命令

### 检查&lt;appname ...="" appname=""&gt;&lt;/appname&gt;

`django-admin check`

Changed in Django 1.7.

使用[_system check framework_](checks.html)来检查整个Django项目是否存在常见问题。

系统检查框架将确认您的已安装型号或管理员注册没有任何问题。它还将提供通过将Django升级到新版本引入的常见兼容性问题的警告。其他库和应用程序可能会引入自定义检查。

默认情况下，所有应用都将被选中。您可以通过提供应用标签列表作为参数来检查应用的子集：

```
python manage.py check auth admin myapp

```

如果你没有指定任何一个应用，那么将对全部的应用进行检查。

`--tag` `&lt;tagname&gt;`

[_system check framework_](checks.html)执行许多不同类型的检查。Uses the system check framework to inspect the entire Django project for common problems.您可以使用这些标记将执行的检查仅限于特定类别中的检查。例如，要仅执行安全性和兼容性检查，您将运行：

```
python manage.py check --tag security --tag compatibility

```

`--list-tags`

列出所有可用的标签。

`--deploy`

New in Django 1.8.

`--deploy`选项激活仅与部署设置相关的一些其他检查。

您可以在本地开发环境中使用此选项，但由于本地开发设置模块可能没有很多生产设置，因此您可能希望将`check`命令指向不同的设置模块，通过设置`DJANGO_SETTINGS_MODULE`环境变量，或传递`--settings`选项：

```
python manage.py check --deploy --settings=production_settings

```

或者，您可以直接在生产或分段部署中运行它，以验证正确的设置是否正在使用（省略`--settings`）。您甚至可以将它作为集成测试套件的一部分。

### 编译消息

`django-admin compilemessages`

将由[`makemessages`](#django-admin-makemessages)创建的.po文件编译为.mo文件以与内置gettext支持一起使用。请参阅[_Internationalization and localization_](../topics/i18n/index.html)。

使用[`--locale`](#django-admin-option---locale)选项（或其较短版本`-l`如果未提供，则处理所有语言环境。

使用[`--exclude`](#django-admin-option---exclude)选项（或其较短版本`-x`）指定要从处理中排除的区域设置。如果未提供，则不排除语言环境。

您可以传递`--use-fuzzy`选项（或`-f`）将模糊翻译包含到编译文件中。

Changed in Django 1.8:

添加了`--exclude`和`--use-fuzzy`选项。

用法示例：

```
django-admin compilemessages --locale=pt_BR
django-admin compilemessages --locale=pt_BR --locale=fr -f
django-admin compilemessages -l pt_BR
django-admin compilemessages -l pt_BR -l fr --use-fuzzy
django-admin compilemessages --exclude=pt_BR
django-admin compilemessages --exclude=pt_BR --exclude=fr
django-admin compilemessages -x pt_BR
django-admin compilemessages -x pt_BR -x fr

```

### createcachetable

`django-admin createcachetable`

创建用于数据库高速缓存后端的高速缓存表。有关详细信息，请参阅[_Django’s cache framework_](../topics/cache.html)。

[`--database`](#django-admin-option---database)选项可用于指定要安装缓存表的数据库。

Changed in Django 1.7:

不再需要提供高速缓存表名称或[`--database`](#django-admin-option---database)选项。Django从您的设置文件中获取此信息。如果已配置多个高速缓存或多个数据库，则将创建所有高速缓存表。

### dbshel​​l

`django-admin dbshell`

使用[`USER`](settings.html#std:setting-USER)，[`PASSWORD`](settings.html#std:setting-PASSWORD)等中指定的连接参数，为`ENGINE`设置中指定的数据库引擎运行命令行客户端。 ，设置。

*   对于PostgreSQL，它运行`psql`命令行客户端。
*   对于MySQL，它运行`mysql`命令行客户端。
*   对于SQLite，它运行`sqlite3`命令行客户端。

此命令假定程序在您的`PATH`上，以便简单调用程序名称（`psql`，`mysql`，`sqlite3`没有办法手动指定程序的位置。

[`--database`](#django-admin-option---database)选项可用于指定要在其上打开shell的数据库。

### diffsettings

`django-admin diffsettings`

显示现在的设置文件和默认的设置文件之间的差异

在默认设置中没有出现的设置后面跟着 `"###"`. 例如，默认设置中没有定义 [`ROOT_URLCONF`](settings.html#std:setting-ROOT_URLCONF), 所以在输出 `diffsettings`时 [`ROOT_URLCONF`](settings.html#std:setting-ROOT_URLCONF) 后面跟着 `"###"` 

可以提供[`--all`](#django-admin-option---all)选项以显示所有设置，即使它们具有Django的默认值。此类设置的前缀为`"###"`。

### dumpdata&lt;app_label ...="" app_label="" app_label.model=""&gt;&lt;/app_label&gt;

`django-admin dumpdata`

该命令将所有与被命名应用相关联的数据库中的数据输出到标准输出。

如果在dumpdate命令后面未指定Django应用名，则Django项目中安装的所有应用的数据都将被dump到fixture中。

 `dumpdata`命令的输出可作为[`loaddata`](#django-admin-loaddata)命令的输入

请注意，`dumpdata`使用模型上的默认管理器来选择要转储的记录。如果您使用[_custom manager_](../topics/db/managers.html#custom-managers)作为默认管理器，并过滤一些可用记录，则不会转储所有对象。

可以提供[`--all`](#django-admin-option---all)选项来指定`dumpdata`应使用Django的基本管理器，转储可能会被自定义管理器过滤或修改的记录。

`--format` `&lt;fmt&gt;`

默认情况下，`dumpdata`将以JSON格式输出其输出，但您可以使用`--format`选项指定另一种格式。目前支持的格式列在[_Serialization formats_](../topics/serialization.html#serialization-formats)中。

`--indent` `&lt;num&gt;`

默认情况下，`dumpdata`将在一行上输出所有数据。这对于人类来说不容易阅读，因此您可以使用`--indent`选项来漂亮地打印具有多个缩进空格的输出。

可以提供[`--exclude`](#django-admin-option---exclude)选项以防止特定应用或模型（以`app_label.`ModelName）。如果将模型名称指定为`dumpdata`，则转储的输出将限制为该模型，而不是整个应用程序。您还可以混合应用程序名称和型号名称。

[`--database`](#django-admin-option---database)选项可用于指定要从中转储数据的数据库。

`--natural-foreign`

New in Django 1.7.

当指定此选项时，Django将使用`natural_key()`模型方法将任何外键和多对多关系序列化到定义方法的类型的对象。如果您要转储`contrib.auth` `Permission`物件或`contrib.contenttypes` `ContentType`物件，旗。有关此和下一个选项的更多详细信息，请参阅[_natural keys_](../topics/serialization.html#topics-serialization-natural-keys)文档。

`--natural-primary`

New in Django 1.7.

当指定此选项时，Django不会在此对象的序列化数据中提供主键，因为它可以在反序列化期间计算。

`--natural`

自1.7版起已弃用：等同于[`--natural-foreign`](#django-admin-option---natural-foreign)选项；使用它。

使用[_natural keys_](../topics/serialization.html#topics-serialization-natural-keys)来表示任何外键和多对多关系与提供自然键定义的模型。

`--pks`

默认情况下，`dumpdata`将输出模型的所有记录，但您可以使用`--pks`选项指定要过滤的主键的逗号分隔列表。这仅在转储一个模型时可用。

`--output`

New in Django 1.8.

缺省，`dumpdata`命令会将所有经序列化之后的数据输出到标准输出。使用--output选项允许指定数据被写入的文件。

### 冲刷

`django-admin flush`

从数据库中删除所有数据，重新执行任何后同步处理程序，并重新安装任何初始数据夹具。

可以提供[`--noinput`](#django-admin-option---noinput)选项以禁止所有用户提示。

[`--database`](#django-admin-option---database)选项可用于指定要刷新的数据库。

#### --no-initial-data

使用`--no-initial-data`可避免加载initial_data fixture。

### inspectdb

`django-admin inspectdb`

内部预测由[`NAME`](settings.html#std:setting-NAME)设置指向的数据库中的数据库表和视图，并将Django模型模块（`models.py`文件）输出到标准输出。

如果您有要使用Django的旧数据库，请使用此选项。该脚本将检查数据库并为其中的每个表或视图创建一个模型。

您可能会期望，创建的模型将为表或视图中的每个字段都有一个属性。请注意，`inspectdb`在其字段名称输出中有一些特殊情况：

*   如果`inspectdb`无法将列类型映射到模型字段类型，则会使用`TextField`并插入Python注释`'This在字段旁边的字段 类型 是 a 在生成的模型中。`
*   If the database column name is a Python reserved word (such as `'pass'`, `'class'` or `'for'`), `inspectdb` will append `'_field'` to the attribute name. 例如，如果表具有`'for'`的列，则生成的模型将具有`'for_field'`字段，并设置`db_column`属性集到`'for'`。`inspectdb` will insert the Python comment `'Field renamed because it was a Python reserved word.'` next to the field.

此功能意味着作为一个快捷方式，而不是作为明确的模型生成。运行它之后，您将需要自己查看生成的模型以进行自定义。特别是，您需要重新排列模型的顺序，以便引用其他模型的模型正确排序。

主键自动对PostgreSQL，MySQL和SQLite进行自动检查，在这种情况下，Django在需要时将`primary_key=True`。

`inspectdb`适用于PostgreSQL，MySQL和SQLite。外键检测只适用于PostgreSQL和某些类型的MySQL表。

在模型字段上指定[`default`](models/fields.html#django.db.models.Field.default "django.db.models.Field.default")时，Django不会创建数据库默认值。类似地，数据库默认值不会转换为模型字段默认值，也不能通过`inspectdb`以任何方式检测。

默认情况下，`inspectdb`创建非托管模型。也就是说，模型的`Meta`类中的`托管 = False`告诉Django不要管理每个表的创建，修改和删除。如果您希望允许Django管理表的生命周期，您需要将[`managed`](models/options.html#django.db.models.Options.managed "django.db.models.Options.managed")选项更改为`True`（或者直接删除它，因为`True`

[`--database`](#django-admin-option---database)选项可用于指定要内省的数据库。

New in Django 1.8:

添加了检查数据库视图的功能。在以前的版本中，仅检查表（而不是视图）。

### loaddata

`django-admin loaddata`

搜索并将命名夹具的内容加载到数据库中。

[`--database`](#django-admin-option---database)选项可用于指定要将数据加载到的数据库。

`--ignorenonexistent`

[`--ignorenonexistent`](#django-admin-option---ignorenonexistent)选项可用于忽略可能自灯具最初生成后移除的字段和模型。

`--app`

[`--app`](#django-admin-option---app)选项可用于指定单个应用程序来查找fixture，而不是浏览所有应用程序。

Changed in Django 1.7:

`--app`已添加。

Changed in Django 1.8:

`--ignorenonexistent`也忽略不存在的模型。

#### What’s a “fixture”?

_fixture_是包含数据库的序列化内容的文件集合。每个夹具具有唯一的名称，并且包括夹具的文件可以在多个应用中分布在多个目录上。

Django将在三个位置搜索灯具：

1.  在每个安装的应用程序的`fixtures`目录中
2.  在[`FIXTURE_DIRS`](settings.html#std:setting-FIXTURE_DIRS)设置中命名的任何目录中
3.  在灯具命名的文字路径

Django将加载它找到的任何和所有的灯具在这些位置匹配提供的灯具名称。

如果命名的夹具具有文件扩展名，则只加载该类型的夹具。例如：

```
django-admin loaddata mydata.json

```

将只加载名为`mydata`的JSON fixture。fixture扩展名必须与[_serializer_](../topics/serialization.html#serialization-formats)（例如，`json`或`xml`）的注册名称相对应。

如果你省略扩展，Django将搜索所有可用的灯具类型匹配的夹具。例如：

```
django-admin loaddata mydata

```

将寻找任何夹具类型称为`mydata`。如果fixture目录包含`mydata.json`，则该fixture将作为JSON夹具加载。

命名的灯具可以包括目录组件。这些目录将包含在搜索路径中。例如：

```
django-admin loaddata foo/bar/mydata.json

```

将针对每个安装的应用程序搜索`&lt;app_label&gt;/fixtures/foo/bar/mydata.json`，`&lt;dirname&gt;/foo/bar/mydata.json`在[`FIXTURE_DIRS`](settings.html#std:setting-FIXTURE_DIRS)和文本路径`foo/bar/mydata.json`中。

当夹具文件被处理时，数据被保存到数据库。不调用模型定义的[`save()`](models/instances.html#django.db.models.Model.save "django.db.models.Model.save")方法，并且将使用`raw=True`调用任何[`pre_save`](signals.html#django.db.models.signals.pre_save "django.db.models.signals.pre_save")或[`post_save`](signals.html#django.db.models.signals.post_save "django.db.models.signals.post_save") &gt;因为实例只包含模型本地的属性。例如，您可以禁用处理程序访问相关字段，这些字段在夹具加载期间不存在，否则会引发异常：

```
from django.db.models.signals import post_save
from .models import MyModel

def my_handler(**kwargs):
    # disable the handler during fixture loading
    if kwargs['raw']:
        return
    ...

post_save.connect(my_handler, sender=MyModel)

```

你也可以写一个简单的装饰器来封装这个逻辑：

```
from functools import wraps

def disable_for_loaddata(signal_handler):
    """
 Decorator that turns off signal handlers when loading fixture data.
 """
    @wraps(signal_handler)
    def wrapper(*args, **kwargs):
        if kwargs['raw']:
            return
        signal_handler(*args, **kwargs)
    return wrapper

@disable_for_loaddata
def my_handler(**kwargs):
    ...

```

请注意，只要灯具反序列化，这个逻辑将禁用信号，而不仅仅是在`loaddata`期间。

注意，夹具文件的处理顺序是未定义的。然而，所有灯具数据被安装为单个事务，因此一个灯具中的数据可以引用另一个灯具中的数据。如果数据库后端支持行级约束，那么将在事务结束时检查这些约束。

[`dumpdata`](#django-admin-dumpdata)命令可用于为`loaddata`生成输入。

#### 压缩夹具

灯具可以按`zip`，`gz`或`bz2`格式压缩。例如：

```
django-admin loaddata mydata.json

```

将查找`mydata.json`，`mydata.json.zip`，`mydata.json.gz`或`mydata.json.bz2`。使用包含在压缩压缩归档文件中的第一个文件。

注意，如果发现两个具有相同名称但不同类型的灯具（例如，如果在同一夹具目录中找到`mydata.json`和`mydata.xml.gz` ），灯具安装将中止，并且调用`loaddata`中安装的任何数据将从数据库中删除。

MySQL与MyISAM和fixtures

MySQL的MyISAM存储引擎不支持事务或约束，因此如果您使用MyISAM，您将不会获得夹具数据的验证，或者如果找到多个事务文件，则回滚。

#### 数据库特定夹具

如果您在多数据库设置中，您可能需要将夹具数据加载到一个数据库，但不加载到另一个数据库。在这种情况下，您可以添加数据库标识符到您的灯具的名称。

For example, if your [`DATABASES`](settings.html#std:setting-DATABASES) setting has a ‘master’ database defined, name the fixture `mydata.master.json` or `mydata.master.json.gz` and the fixture will only be loaded when you specify you want to load data into the `master` database.

### makemessages

`django-admin makemessages`

运行在当前目录的整个源树上，并拉出所有标记为要翻译的字符串。它在conf / locale（在Django树中）或locale（对于项目和应用程序）目录中创建（或更新）消息文件。更改消息文件后，您需要使用[`compilemessages`](#django-admin-compilemessages)编译它们以与内置gettext支持一起使用。有关详细信息，请参阅[_i18n documentation_](../topics/i18n/translation.html#how-to-create-language-files)。

`--all`

使用`--all`或`-a`选项更新所有可用语言的消息文件。

用法示例：

```
django-admin makemessages --all

```

`--extension`

使用`--extension`或`-e`选项指定要检查的文件扩展名列表（默认值：“.html”，“.txt”）。

用法示例：

```
django-admin makemessages --locale=de --extension xhtml

```

使用逗号分隔多个扩展程序或多次使用-e或-extension：

```
django-admin makemessages --locale=de --extension=html,txt --extension xml

```

使用[`--locale`](#django-admin-option---locale)选项（或其较短版本`-l`

New in Django 1.8.

使用[`--exclude`](#django-admin-option---exclude)选项（或其较短版本`-x`）指定要从处理中排除的区域设置。如果未提供，则不排除语言环境。

用法示例：

```
django-admin makemessages --locale=pt_BR
django-admin makemessages --locale=pt_BR --locale=fr
django-admin makemessages -l pt_BR
django-admin makemessages -l pt_BR -l fr
django-admin makemessages --exclude=pt_BR
django-admin makemessages --exclude=pt_BR --exclude=fr
django-admin makemessages -x pt_BR
django-admin makemessages -x pt_BR -x fr

```

Changed in Django 1.7:

在与现有po文件合并时，向`msgmerge`命令添加`--previous`选项。

`--domain`

使用`--domain`或`-d`选项更改消息文件的域。目前支持：

*   `django`对于所有`*.py`，`*.html`和`*.txt`
*   `djangojs`用于`*.js`文件

`--symlinks`

在查找新的翻译字符串时，使用`--symlinks`或`-s`选项跟随符号链接到目录。

用法示例：

```
django-admin makemessages --locale=de --symlinks

```

`--ignore`

使用`--ignore`或`-i`选项忽略与给定[`glob`](https://docs.python.org/3/library/glob.html#module-glob "(in Python v3.4)")样式模式匹配的文件或目录。使用多次忽略更多。

默认使用这些模式：`'CVS'`，`'.*'`，`'*~'`，`'*.pyc'`

用法示例：

```
django-admin makemessages --locale=en_US --ignore=apps/* --ignore=secret/*.html

```

`--no-default-ignore`

使用`--no-default-ignore`选项禁用[`--ignore`](#django-admin-option---ignore)的默认值。

`--no-wrap`

使用`--no-wrap`选项禁用在语言文件中将长消息行分成几行。

`--no-location`

使用`--no-location`选项不写入`＃： filename：line 文件。`请注意，使用此选项使技术熟练的翻译人员更难理解每个邮件的上下文。

`--keep-pot`

使用`--keep-pot`选项可防止Django在创建.po文件之前删除其生成的临时.pot文件。这对于调试可能阻止创建最终语言文件的错误非常有用。

也可以看看

有关如何自定义[`makemessages`](#django-admin-makemessages)传递到`xgettext`的关键字的说明，请参阅[_Customizing the makemessages command_](../topics/i18n/translation.html#customizing-makemessages)。

### makemigrations [&lt;app_label&gt;]&lt;/app_label&gt;

`django-admin makemigrations`

New in Django 1.7.

根据检测到的模型改变创建新的迁移。迁移与应用程序之间的关系以及更多内容将在[_迁移文档_](../topics/migrations.html)中深入介绍。

提供一个或多个应用程序名称作为参数将限制为指定的应用程序创建的迁移以及所需的任何依赖项（例如，`ForeignKey`的另一端的表）。

`--empty`

`--empty`选项将导致`makemigrations`输出指定应用程序的空迁移，以进行手动编辑。此选项仅适用于高级用户，除非您熟悉迁移格式，迁移操作以及迁移之间的依赖关系，否则不应使用此选项。

`--dry-run`

`--dry-run`选项显示在不实际将任何迁移文件写入磁盘的情况下将进行的迁移。与` - verbosity 3`一起使用此选项还将显示将要写入的完整迁移文件。

`--merge`

`--merge`选项允许修复迁移冲突。可以提供[`--noinput`](#django-admin-option---noinput)选项以在合并期间抑制用户提示。

`--name``, -n`

New in Django 1.8.

`--name`选项允许您为迁移指定自定义名称，而不是生成的名称。

`--exit``, -e`

New in Django 1.8.

The `--exit` option will cause `makemigrations` to exit with error code 1 when no migration are created (or would have been created, if combined with `--dry-run`).

### migrate [&lt;app_label&gt;[&lt;migrationname&gt;]]&lt;/migrationname&gt;&lt;/app_label&gt;

`django-admin migrate`

New in Django 1.7.

使数据库状态与当前模型集和迁移集同步。迁移与应用程序之间的关系以及更多内容将在[_迁移文档_](../topics/migrations.html)中深入介绍。

此命令的行为根据提供的参数而有所不同：

*   无参数：所有已迁移的应用程式都会执行所有迁移作业，所有未迁移的应用程式都会与资料库同步处理，
*   `&lt;app_label&gt;`：指定的应用程序运行其迁移，直到最近的迁移。这可能涉及运行其他应用程序的迁移，由于依赖性。
*   `＆lt； app_label＆gt； ＆lt； migrationname＆gt；`：将数据库模式设置为应用指定迁移的状态，相同的应用程序。如果之前已迁移过指定的迁移，则可能会导致不应用迁移。使用名称`zero`取消应用应用程序的所有迁移。

与`syncdb`不同，此命令不会提示您创建超级用户（如果不存在）（假设您正在使用[`django.contrib.auth`](../topics/auth/index.html#module-django.contrib.auth "django.contrib.auth: Django's authentication framework.")）。如果您愿意，可以使用[`createsuperuser`](#django-admin-createsuperuser)来执行此操作。

[`--database`](#django-admin-option---database)选项可用于指定要迁移的数据库。

`--fake`

`--fake`选项会告诉Django将迁移标记为已应用或未应用，但没有实际运行SQL来更改数据库模式。

这适用于高级用户，如果他们手动应用更改，则直接操作当前迁移状态；请注意，使用`--fake`会使迁移状态表进入需要手动恢复的状态，以使迁移正常运行。

New in Django 1.8.

`--fake-initial`

`--fake-initial`选项可用于允许Django跳过应用程序的初始迁移（如果所有数据库表都具有由所有[`CreateModel`](migration-operations.html#django.db.migrations.operations.CreateModel "django.db.migrations.operations.CreateModel")操作创建的所有模型的名称）迁移已经存在。此选项适用于首次针对预先使用迁移的数据库运行迁移时使用。但是，此选项不会检查匹配的表名以外的匹配数据库模式，因此只有在您确信现有模式与初始迁移中记录的模式匹配时才能安全使用。

自1.8版起已弃用：`--list`选项已移至[`showmigrations`](#django-admin-showmigrations)命令。

### runfcgi [options]

`django-admin runfcgi`

自1.7版起已弃用：FastCGI支持已弃用，将在Django 1.9中删除。

启动一组适用于支持FastCGI协议的任何Web服务器的FastCGI进程。有关详细信息，请参见[_FastCGI deployment documentation_](../howto/deployment/fastcgi.html)。需要来自[flup](http://www.saddi.com/software/flup/)的Python FastCGI模块。

在内部，它包装由[`WSGI_APPLICATION`](settings.html#std:setting-WSGI_APPLICATION)设置指定的WSGI应用程序对象。

此命令接受的选项将传递到FastCGI库，不要使用其他Django管理命令通常使用的`'--'`前缀。

`protocol`

`protocol=PROTOCOL`

使用协议。_PROTOCOL_可以是`fcgi`，`scgi`，`ajp`等（默认为`fcgi` ）

`host`

`host=HOSTNAME`

要监听的主机名。

`port`

`port=PORTNUM`

端口监听。

`socket`

`socket=FILE`

UNIX套接字监听。

`method`

`method=IMPL`

可能的值：`prefork`或`threaded`（默认为`prefork`）

`maxrequests`

`maxrequests=NUMBER`

在子进程被杀死和新子进程被分叉之前处理的请求数（0表示没有限制）。

`maxspare`

`maxspare=NUMBER`

最大备用进程/线程数。

`minspare`

`minspare=NUMBER`

最小数量的备用进程/线程。

`maxchildren`

`maxchildren=NUMBER`

进程/线程的硬限制数。

`daemonize`

`daemonize=BOOL`

是否从终端分离。

`pidfile`

`pidfile=FILE`

将生成的进程标识写入文件_FILE_。

`workdir`

`workdir=DIRECTORY`

在进行守护进程时切换到目录_DIRECTORY_。

`debug`

`debug=BOOL`

设置为true以启用flup跟踪。

`outlog`

`outlog=FILE`

将stdout写入_FILE_文件。

`errlog`

`errlog=FILE`

将stderr写入_FILE_文件。

`umask`

`umask=UMASK`

在进行守护进程时使用的掩码。该值被解释为八进制数（默认值为`0o22`）。

用法示例：

```
django-admin runfcgi socket=/tmp/fcgi.sock method=prefork daemonize=true \
    pidfile=/var/run/django-fcgi.pid

```

将FastCGI服务器作为守护程序运行，并将生成的PID写入文件。

### runserver [IP地址 : 端口号]

`django-admin runserver`

启用本地上一个轻量级的Web服务器。默认情况下，服务器运行在IP地址`127.0.0.1`的8000端口上。当然，你也可以显式地传递一个IP地址和端口号给它。

如果您以一个普通用户的身份来运行脚本, 你可能没有权限在低位端口上运行。低端口数(即1024以下)是预留出来给超级用户（root）的。

这个服务器使用的WSGI application对象是通过在[`WSGI_APPLICATION`](settings.html#std:setting-WSGI_APPLICATION) .中的设置指定的

不要在生产环境的设置中使用这个服务器。他是没有通过安全审查或者性能测试的。（这是怎么回事。我们所关心的事情是Web框架而不是Web服务器，所以改进这个服务器使它来达到生产环境的性能已经超出了我们的讨论范围。

如果有需要，每当有访问请求时，这一开发服务器便会自动重新载入代码。你并不需要在每次代码有变更后重启服务器来使它生效。然而，一些诸如添加新文件等变更并不会引发服务器的自动重启，所以在这种情况下你仍需手动重启。

Changed in Django 1.7:

当文件完成变更后也会引发开发服务器的重新启动。

如果你使用Linux系统，并且安装了[pyinotify](https://pypi.python.org/pypi/pyinotify/)，内核信号机制会自动引起开发服务器的重新启动 (而不用每秒去轮询文件的最后更改时间戳)。这为大型项目提供了更好的扩展，减少了对代码修改的响应时间，更强大的变化检测和电池使用减少。

New in Django 1.7:

加入对`pyinotify` 的支持

当你启动服务器之后，在服务器运行过程中每当你的Python代码有变更时，系统的检测框架将会检查整个项目中是否存在一些直观的错误 (参见 [`check`](#django-admin-check)). 如果检测到了错误，这些错误信息将会输出至标准输出。

你可以运行多个服务，只要保证它们监听不同的端口。多次执行 `django-admin runserver`即可

注意默认的IP `127.0.0.1`，它是不可被网络中的其它主机所访问的。若使开发服务器在网络中被其它主机可见，请使用该主机的IP地址 (如： `192.168.2.1`) 或 `0.0.0.0` ，或者使用`::` (在IPv6可用的情况下).

You can provide an IPv6 address surrounded by brackets (e.g. `[200a::1]:8000`).你可以使用在括号内使用IPv6地址This will automatically enable IPv6 support.这样会使ipv6地址自动生效

A hostname containing ASCII-only characters can also be used.也可以使用只包含ASCII的主机名.

如果启用了[_staticfiles_](contrib/staticfiles.html) contrib应用程序（在新项目中为默认），则[`runserver`](#django-admin-runserver)命令将被其自己的[_runserver_](contrib/staticfiles.html#staticfiles-runserver)命令覆盖。

如果先前未执行[`migrate`](#django-admin-migrate)，则存储迁移历史记录的表在第一次运行`runserver`时创建。

`--noreload`

使用 `--noreload`命令选项可以禁止自动更新这意味着当server开始运行以后，不论你对python代码做了什么修改都_不会_影响到已经加载到内存中的Python模块.

用法示例：

```
django-admin runserver --noreload

```

`--nothreading`

开发服务器默认开启多线程使用 `--nothreading`选项可以禁止开发服务器中多线程的使用。

`--ipv6``, -6`

使用 `--ipv6` (or shorter `-6`) 选项告诉Django为开发服务器使用IPV6。这会将默认的IP地址从`127.0.0.1` 改为`::1`。

用法示例：

```
django-admin runserver --ipv6

```

#### 使用不同的端口和地址的示例

端口8000在IP地址`127.0.0.1`：

```
django-admin runserver

```

端口8000在IP地址`1.2.3.4`：

```
django-admin runserver 1.2.3.4:8000

```

端口7000在IP地址`127.0.0.1`：

```
django-admin runserver 7000

```

端口7000在IP地址`1.2.3.4`：

```
django-admin runserver 1.2.3.4:7000

```

端口8000在IPv6地址`::1`：

```
django-admin runserver -6

```

IPv6地址`::1`：

```
django-admin runserver -6 7000

```

端口7000在IPv6地址`2001:0db8:1234:5678::9`：

```
django-admin runserver [2001:0db8:1234:5678::9]:7000

```

端口8000在主机的IPv4地址`localhost`：

```
django-admin runserver localhost:8000

```

端口8000在主机的IPv6地址`localhost`：

```
django-admin runserver -6 localhost:8000

```

#### 使用开发服务器提供静态文件

默认设置中，开发服务器不会为你的网站提供任何的静态文件(比如CSS 文件, images, [`MEDIA_URL`](settings.html#std:setting-MEDIA_URL) 下的文件等等). 如果要配置Django以提供静态媒体，请阅读[_Managing static files (CSS, images)_](../howto/static-files/index.html)。

### shell

`django-admin shell`

启动Python交互式解释器。

如果安装了任何一个，Django将使用[IPython](http://ipython.scipy.org/)或[bpython](http://bpython-interpreter.org/)。如果您安装了一个丰富的shell，但是想强制使用“plain”Python解释器，请使用`--plain`选项，如下所示：

```
django-admin shell --plain

```

如果您想要指定IPython或bpython作为您的解释器（如果已安装），您可以使用`-i`或`--interface`选项指定替代解释器接口：

IPython：

```
django-admin shell -i ipython
django-admin shell --interface ipython

```

bpython：

```
django-admin shell -i bpython
django-admin shell --interface bpython

```

When the “plain” Python interactive interpreter starts (be it because `--plain` was specified or because no other interactive interface is available) it reads the script pointed to by the [`PYTHONSTARTUP`](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONSTARTUP "(in Python v3.4)") environment variable and the `~/.pythonrc.py` script. 如果您不希望此行为，您可以使用`--no-startup`选项。例如。：

```
django-admin shell --plain --no-startup

```

### showmigrations [&lt;app_label&gt;[&lt;app_label&gt;]]&lt;/app_label&gt;&lt;/app_label&gt;

`django-admin showmigrations`

New in Django 1.8.

展示项目中所有的迁移

`--list``, -l`

`--list`选项列出了Django知道的所有应用程序，每个应用程序可用的迁移以及是否应用每个迁移（由`[X]`旁边的迁移名称）。

还列出了尚未迁移的应用，但已打印`（无 迁移）`。

`--plan``, -p`

`--plan`选项显示Django将执行的迁移计划以应用迁移。任何提供的应用标签都会被忽略，因为计划可能超出这些应用。与`--list`相同，应用的迁移由`[X]`标记。对于2以上的冗长，还将显示迁移的所有依赖性。

### sql&lt;app_label ...="" app_label=""&gt;&lt;/app_label&gt;

`django-admin sql`

打印给定应用程序名称的CREATE TABLE SQL语句。

[`--database`](#django-admin-option---database)选项可用于指定要为其打印SQL的数据库。

### sqlall&lt;app_label ...="" app_label=""&gt;&lt;/app_label&gt;

`django-admin sqlall`

打印给定应用程序名称的CREATE TABLE和初始数据SQL语句。

有关如何指定初始数据的说明，请参阅[`sqlcustom`](#django-admin-sqlcustom)的说明。

[`--database`](#django-admin-option---database)选项可用于指定要为其打印SQL的数据库。

Changed in Django 1.7:

`sql*`管理命令现在遵守[`DATABASE_ROUTERS`](settings.html#std:setting-DATABASE_ROUTERS)的`allow_migrate()`方法。如果模型已同步到非默认数据库，请使用[`--database`](#django-admin-option---database)标志为这些模型获取SQL（以前它们将始终包含在输出中）。

### sqlclear&lt;app_label ...="" app_label=""&gt;&lt;/app_label&gt;

`django-admin sqlclear`

打印给定应用程序名称的DROP TABLE SQL语句。

[`--database`](#django-admin-option---database)选项可用于指定要为其打印SQL的数据库。

### sqlcustom&lt;app_label ...="" app_label=""&gt;&lt;/app_label&gt;

`django-admin sqlcustom`

打印给定应用程序名称的自定义SQL语句。

对于每个指定应用中的每个模型，此命令查找文件`&lt;app_label&gt;/sql/&lt;modelname&gt;.sql`，其中`&lt;app_label&gt;`应用名称和`&lt;modelname&gt;`是模型的名称小写。例如，如果您有包含`Story`模型的应用`news`，`sqlcustom`将尝试读取文件`news/sql/story.sql`并将其附加到此命令的输出。

每个SQL文件（如果给出）都应包含有效的SQL。在执行所有模型的表创建语句之后，SQL文件将直接管道传输到数据库中。使用此SQL钩子可以进行任何表修改，或将任何SQL函数插入数据库。

请注意，处理SQL文件的顺序未定义。

[`--database`](#django-admin-option---database)选项可用于指定要为其打印SQL的数据库。

### sqldropindexes&lt;app_label ...="" app_label=""&gt;&lt;/app_label&gt;

`django-admin sqldropindexes`

打印给定应用程序名称的DROP INDEX SQL语句。

[`--database`](#django-admin-option---database)选项可用于指定要为其打印SQL的数据库。

### sqlflush

`django-admin sqlflush`

打印将为[`flush`](#django-admin-flush)命令执行的SQL语句。

[`--database`](#django-admin-option---database)选项可用于指定要为其打印SQL的数据库。

### sqlindexes&lt;app_label ...="" app_label=""&gt;&lt;/app_label&gt;

`django-admin sqlindexes`

打印给定应用程序名称的CREATE INDEX SQL语句。

[`--database`](#django-admin-option---database)选项可用于指定要为其打印SQL的数据库。

### sqlmigrate&lt;app_label&gt;&lt;migrationname&gt;&lt;/migrationname&gt;&lt;/app_label&gt;

`django-admin sqlmigrate`

打印命名迁移的SQL。这需要活动的数据库连接，它将用于解析约束名称；这意味着您必须针对要在以后应用它的数据库的副本生成SQL。

请注意，`sqlmigrate`不会对其输出进行着色。

[`--database`](#django-admin-option---database)选项可用于指定要为其生成SQL的数据库。

`--backwards`

默认情况下，创建的SQL用于以向前方向运行迁移。传递`--backwards`以生成用于取消应用迁移的SQL。

### sqlsequencereset&lt;app_label ...="" app_label=""&gt;&lt;/app_label&gt;

`django-admin sqlsequencereset`

打印用于重置给定应用程序名称的序列的SQL语句。

序列是一些数据库引擎使用的索引，用于跟踪自动递增字段的下一个可用数字。

使用此命令可以生成SQL，这将修复序列与其自动递增的字段数据不同步的情况。

[`--database`](#django-admin-option---database)选项可用于指定要为其打印SQL的数据库。

### squashmigrations&lt;app_label&gt;&lt;migration_name&gt;&lt;/migration_name&gt;&lt;/app_label&gt;

`django-admin squashmigrations`

将`app_label`的迁移（包括`migration_name`）迁移到较少的迁移中（如果可能）。由此造成的被挤压的迁移可以安全地与未被挖掘的迁移。有关详细信息，请参阅[_Squashing migrations_](../topics/migrations.html#migration-squashing)。

`--no-optimize`

默认情况下，Django将尝试优化迁移中的操作，以减少生成的文件的大小。如果此过程失败或创建不正确的迁移，则传递`--no-optimize`，但也请提交有关行为的Django错误报告，因为优化意味着安全。

### startapp &lt;app_label&gt;[destination]&lt;/app_label&gt;

`django-admin startapp`

在当前目录或给定目标中为给定应用程序名称创建Django应用程序目录结构。

默认情况下，创建的目录包含一个`models.py`文件和其他应用程序模板文件。（有关详细信息，请参阅[源](https://github.com/django/django/tree/master/django/conf/app_template/)。）如果只给出应用程序名称，则将在当前工作目录中创建应用程序目录。

如果提供了可选的目标，Django将使用现有目录，而不是创建一个新目录。您可以使用“。”来表示当前工作目录。

例如：

```
django-admin startapp myapp /Users/jezdez/Code/myapp

```

`--template`

使用`--template`选项，您可以使用自定义应用程序模板，通过为应用程序模板文件提供目录路径，或者提供压缩文件的路径（`.tar.gz`，`.tar.bz2`，`.tgz`，`.tbz`，`.zip`）模板文件。

例如，在创建`myapp`应用程序时，这将在给定目录中查找应用模板：

```
django-admin startapp --template=/Users/jezdez/Code/my_app_template myapp

```

Django还将使用应用模板文件接受压缩归档的URL（`http`，`https`，`ftp`），即时下载和提取。

例如，利用Github的功能将仓库公开为zip文件，您可以使用以下URL：

```
django-admin startapp --template=https://github.com/githubuser/django-app-template/archive/master.zip myapp

```

当Django复制应用程序模板文件时，它还通过模板引擎呈现特定文件：扩展名与`--extension`选项（默认情况下为`py`）匹配的文件，其名称通过`--name`选项传递。使用的[`template context`](templates/api.html#django.template.Context "django.template.Context")

*   传递给`startapp`命令的任何选项（在命令支持的选项之间）
*   `app_name` - 传递给命令的应用程序名称
*   `app_directory` - 新创建的应用程序的完整路径
*   `docs_version` - 文档版本：`'dev'`或`'1.x'`

警告

当使用Django模板引擎（默认情况下，所有`*.py`文件）呈现应用程序模板文件时，Django也将替换所有包含的模板变量。例如，如果其中一个Python文件包含解释与模板呈现相关的特定功能的docstring，则可能会导致不正确的示例。

要解决此问题，您可以使用[`templatetag`](templates/builtins.html#std:templatetag-templatetag)模板标签来“转义”模板语法的各个部分。

### startproject &lt;projectname&gt;[destination]&lt;/projectname&gt;

`django-admin startproject`

在当前目录或给定目标中为给定项目名称创建Django项目目录结构。

默认情况下，新目录包含`manage.py`和项目包（包含`settings.py`和其他文件）。有关详细信息，请参阅[模板源](https://github.com/django/django/tree/master/django/conf/project_template/)。

如果仅给出项目名称，则项目目录和项目包将命名为`&lt;projectname&gt;`，并且将在当前工作目录中创建项目目录。

如果提供了可选目标，Django将使用现有目录作为项目目录，并创建`manage.py`和其中的项目包。使用'。'表示当前工作目录。

例如：

```
django-admin startproject myproject /Users/jezdez/Code/myproject_repo

```

与[`startapp`](#django-admin-startapp)命令一样，`--template`选项允许您指定自定义项目模板的目录，文件路径或URL。有关受支持的项目模板格式的详细信息，请参阅[`startapp`](#django-admin-startapp)文档。

例如，在创建`myproject`项目时，将在给定目录中查找项目模板：

```
django-admin startproject --template=/Users/jezdez/Code/my_project_template myproject

```

Django还将接受使用项目模板文件的压缩归档的URL（`http`，`https`，`ftp`），即时下载和提取它们。

例如，利用Github的功能将仓库公开为zip文件，您可以使用以下URL：

```
django-admin startproject --template=https://github.com/githubuser/django-project-template/archive/master.zip myproject

```

当Django复制项目模板文件时，它还通过模板引擎呈现某些文件：扩展名与`--extension`选项（默认情况下为`py`）匹配的文件和文件其名称通过`--name`选项传递。使用的[`template context`](templates/api.html#django.template.Context "django.template.Context")

*   传递到`startproject`命令的任何选项（在命令支持的选项之间）
*   `project_name` - 传递给命令的项目名称
*   `project_directory` - 新创建项目的完整路径
*   `secret_key` - [`SECRET_KEY`](settings.html#std:setting-SECRET_KEY)设置的随机密钥
*   `docs_version` - 文档版本：`'dev'`或`'1.x'`

另请参阅[`startapp`](#django-admin-startapp)中提及的[_rendering warning_](#render-warning)。

### syncdb

`django-admin syncdb`

自1.7版起已弃用：此命令已被弃用，支持[`migrate`](#django-admin-migrate)命令，它执行旧的行为以及执行迁移。它现在只是该命令的别名。

[`migrate`](#django-admin-migrate)的别名。

### 测试&lt;app identifier="" or="" test=""&gt;&lt;/app&gt;

`django-admin test`

对所有安装的模型运行测试。有关详细信息，请参阅[_Testing in Django_](../topics/testing/index.html)。

`--failfast`

`--failfast`选项可用于停止运行测试，并在测试失败后立即报告故障。

`--testrunner`

`--testrunner`选项可用于控制用于执行测试的测试运行器类。如果提供此值，它将覆盖由[`TEST_RUNNER`](settings.html#std:setting-TEST_RUNNER)设置提供的值。

`--liveserver`

`--liveserver`选项可用于覆盖运行服务器（与[`LiveServerTestCase`](../topics/testing/tools.html#django.test.LiveServerTestCase "django.test.LiveServerTestCase")配合使用）的默认地址。默认值为`localhost:8081`。

`--keepdb`

New in Django 1.8.

`--keepdb`选项可用于在测试运行之间保留测试数据库。这具有跳过创建和销毁操作的优点，这大大减少了运行测试的时间，特别是在大型测试套件中。如果测试数据库不存在，它将在第一次运行时创建，然后保存用于每次后续运行。任何未应用的迁移也将在运行测试套件之前应用于测试数据库。

`--reverse`

New in Django 1.8.

`--reverse`选项可用于以相反的顺序对测试用例排序。这可能有助于调试未正确隔离且具有副作用的测试。使用此选项时，将保留[_Grouping by test class_](../topics/testing/overview.html#order-of-tests)。

`--debug-sql`

New in Django 1.8.

`--debug-sql`选项可用于为失败的测试启用[_SQL logging_](../topics/logging.html#django-db-logger)。如果[`--verbosity`](#django-admin-option---verbosity)是`2`，则也会输出通过测试中的查询。

### testserver

`django-admin testserver`

使用给定fixture的数据运行Django开发服务器（如[`runserver`](#django-admin-runserver)）。

例如，此命令：

```
django-admin testserver mydata.json

```

...将执行以下步骤：

1.  创建测试数据库，如[_The test database_](../topics/testing/overview.html#the-test-database)中所述。
2.  使用来自给定夹具的夹具数据填充测试数据库。（有关灯具的更多信息，请参阅上述[`loaddata`](#django-admin-loaddata)的文档。）
3.  运行Django开发服务器（如[`runserver`](#django-admin-runserver)），指向此新创建的测试数据库，而不是生产数据库。

这在许多方面是有用的：

*   当您编写视图如何使用某些灯具数据执行[_unit tests_](../topics/testing/overview.html)时，您可以使用`testserver`手动与Web浏览器中的视图进行交互。
*   假设你正在开发你的Django应用程序，并且有一个你想要交互的数据库的“原始”副本。您可以将数据库转储到fixture（使用[`dumpdata`](#django-admin-dumpdata)命令，如上所述），然后使用`testserver`运行带有该数据的Web应用程序。有了这个安排，你有以任何方式混乱你的数据的灵活性，知道你正在做的任何数据更改只是对一个测试数据库。

Note that this server does _not_ automatically detect changes to your Python source code (as [`runserver`](#django-admin-runserver) does). 但它会检测对模板的更改。

`--addrport` `[port number or ipaddr:port]`

使用`--addrport`指定默认值`127.0.0.1:8000`的其他端口或IP地址和端口。此值遵循完全相同的格式，并且具有与[`runserver`](#django-admin-runserver)命令的参数完全相同的功能。

例子：

要使用`fixture1`和`fixture2`在端口7000上运行测试服务器：

```
django-admin testserver --addrport 7000 fixture1 fixture2
django-admin testserver fixture1 fixture2 --addrport 7000

```

（上面的语句是等价的。我们包括它们两者，以证明选项在fixture参数之前或之后是无关紧要的。）

使用`test`灯具在1.2.3.4:7000上运行：

```
django-admin testserver --addrport 1.2.3.4:7000 test

```

可以提供[`--noinput`](#django-admin-option---noinput)选项以禁止所有用户提示。

### 验证

`django-admin validate`

自1.7版起已弃用：由[`check`](#django-admin-check)命令替换。

验证所有已安装的模型（根据[`INSTALLED_APPS`](settings.html#std:setting-INSTALLED_APPS)设置），并将验证错误打印到标准输出。

## 应用程序提供的命令

一些命令仅在[_implements_](../howto/custom-management-commands.html)的`django.contrib`应用程序已启用[`enabled`](settings.html#std:setting-INSTALLED_APPS)时可用。本节按照其应用程序对它们进行分组。

### django.contrib.auth

#### 更改密码

`django-admin changepassword`

此命令仅在安装了Django的[_authentication system_](../topics/auth/index.html)（`django.contrib.auth`）时可用。

允许更改用户的密码。它提示您输入作为参数给定的用户密码的两倍。如果它们都匹配，新密码将立即更改。如果您不提供用户，该命令将尝试更改其用户名与当前用户名匹配的密码。

使用`--database`选项指定要为用户查询的数据库。如果没有提供，Django将使用`default`数据库。

用法示例：

```
django-admin changepassword ringo

```

#### createsuperuser

`django-admin createsuperuser`

此命令仅在安装了Django的[_authentication system_](../topics/auth/index.html)（`django.contrib.auth`）时可用。

创建超级用户帐户（具有所有权限的用户）。如果您需要创建初始超级用户帐户，或者需要以编程方式为您的网站生成超级用户帐户，这将非常有用。

以交互方式运行时，此命令将提示输入新超级用户帐户的密码。当以非交互方式运行时，将不会设置密码，并且超级用户帐户将无法登录，直到为其手动设置密码。

`--username`

`--email`

可以使用命令行上的`--username`和`--email`参数提供新帐户的用户名和电子邮件地址。如果未提供其中任何一个，则`createsuperuser`将在以交互方式运行时提示输入。

使用`--database`选项指定将保存超级用户对象的数据库。

New in Django 1.8.

如果要自定义数据输入和验证，可以对管理命令进行子类化，并覆盖`get_input_data()`。有关现有实现和方法参数的详细信息，请参阅源代码。例如，如果您在[`REQUIRED_FIELDS`](../topics/auth/customizing.html#django.contrib.auth.models.CustomUser.REQUIRED_FIELDS "django.contrib.auth.models.CustomUser.REQUIRED_FIELDS")中有`ForeignKey`，并且希望允许创建实例而不是输入现有实例的主键，那么这将非常有用。

### django.contrib.gis

#### ogrinspect

此命令仅在安装[_GeoDjango_](contrib/gis/index.html)（`django.contrib.gis`）时可用。

请参阅GeoDjango文档中的[`description`](contrib/gis/commands.html#django-admin-ogrinspect)。

### django.contrib.sessions

#### 清算

`django-admin clearsessions`

可以作为cron作业运行或直接清除过期的会话。

### django.contrib.sitemaps

#### ping_google

此命令仅在安装了[_Sitemaps framework_](contrib/sitemaps.html)（`django.contrib.sitemaps`）时可用。

请参阅Sitemap说明文件中的[`description`](contrib/sitemaps.html#django-admin-ping_google)。

### django.contrib.staticfiles

#### 集体

仅当安装了[_static files application_](../howto/static-files/index.html)（`django.contrib.staticfiles`）时，此命令才可用。

请参阅[_staticfiles_](contrib/staticfiles.html)文档中的[`description`](contrib/staticfiles.html#django-admin-collectstatic)。

#### findstatic

仅当安装了[_static files application_](../howto/static-files/index.html)（`django.contrib.staticfiles`）时，此命令才可用。

请参阅[_staticfiles_](contrib/staticfiles.html)文档中的[`description`](contrib/staticfiles.html#django-admin-findstatic)。

## 默认选项

虽然一些命令可能允许自己的自定义选项，但每个命令允许以下选项：

`--pythonpath`

用法示例：

```
django-admin migrate --pythonpath='/home/djangoprojects/myproject'

```

将给定的文件系统路径添加到Python [导入搜索路径](http://www.diveintopython.net/getting_to_know_python/everything_is_an_object.html)。如果未提供，`django-admin`将使用`PYTHONPATH`环境变量。

请注意，在`manage.py`中，此选项不必要，因为它会为您设置Python路径。

`--settings`

用法示例：

```
django-admin migrate --settings=mysite.settings

```

显式指定要使用的设置模块。设置模块应该是Python包语法，例如。 `mysite.settings`。如果未提供，`django-admin`将使用`DJANGO_SETTINGS_MODULE`环境变量。

请注意，在`manage.py`中，此选项不是必需的，因为默认情况下它会使用当前项目中的`settings.py`。

`--traceback`

用法示例：

```
django-admin migrate --traceback

```

默认情况下，每当发生[`CommandError`](../howto/custom-management-commands.html#django.core.management.CommandError "django.core.management.CommandError")时，`django-admin`都会显示一个简单的错误消息，但是对于任何其他异常都会显示完整的堆栈跟踪。如果指定`--traceback`，则`CommandError`引发时，`django-admin`也将输出完整的堆栈跟踪。

`--verbosity`

用法示例：

```
django-admin migrate --verbosity 2

```

使用`--verbosity`可指定`django-admin`应打印到控制台的通知和调试信息量。

*   `0`表示无输出。
*   `1`表示正常输出（默认）。
*   `2`表示详细输出。
*   `3`表示_非常_详细输出。

`--no-color`

New in Django 1.7.

用法示例：

```
django-admin sqlall --no-color

```

默认情况下，`django-admin`将格式化要着色的输出。例如，错误将以红色打印到控制台，SQL语句将突出显示语法。要防止这种情况并输出纯文本，请在运行命令时传递`--no-color`选项。

## 常用选项

以下选项不适用于每个命令，但是它们对于多个命令是通用的。

`--database`

用于指定命令将在其上操作的数据库。如果未指定，此选项将默认为别名`default`。

例如，要使用别名`master`从数据库转储数据：

```
django-admin dumpdata --database=master

```

`--exclude`

从输出其内容的应用程序中排除特定应用程序。例如，要从dumpdata的输出中特别排除`auth`应用程序，您需要调用：

```
django-admin dumpdata --exclude=auth

```

如果要排除多个应用程序，请使用多个`--exclude`指令：

```
django-admin dumpdata --exclude=auth --exclude=contenttypes

```

`--locale`

使用`--locale`或`-l`如果未提供，则处理所有语言环境。

`--noinput`

使用`--noinput`选项来禁止所有用户提示，例如“您确定吗？”确认消息。如果`django-admin`作为无人参与的自动脚本执行，这是非常有用的。

## 额外的美味

### 语法着色

如果您的终端支持ANSI颜色输出，则`django-admin` / `manage.py`命令将使用漂亮的颜色编码输出。如果你将命令的输出传递到另一个程序，它不会使用颜色代码。

在Windows下，本机控制台不支持ANSI转义序列，因此默认情况下没有颜色输出。但是，您可以安装[ANSICON](http://adoxa.altervista.org/ansicon/)第三方工具，Django命令将检测其存在，并将使用其服务的颜色输出，就像在基于Unix的平台上。

用于语法高亮的颜色可以自定义。Django附带三个调色板：

*   `dark`，适用于在黑色背景上显示白色文字的端子。这是默认调色板。
*   `light`，适用于在白色背景上显示黑色文本的终端。
*   `nocolor`，禁用语法高亮显示。

您可以通过设置`DJANGO_COLORS`环境变量来指定要使用的调色板来选择调色板。例如，要在Unix或OS / X BASH shell下指定`light`选项板，您将在命令提示符下运行以下命令：

```
export DJANGO_COLORS="light"

```

您还可以自定义所使用的颜色。Django指定了使用颜色的多个角色：

*   `error` - 主要错误。
*   `notice` - 一个小错误。
*   `sql_field` - SQL中模型字段的名称。
*   `sql_coltype` - SQL中的模型字段的类型。
*   `sql_keyword` - 一个SQL关键字。
*   `sql_table` - SQL中模型的名称。
*   `http_info` - 1XX HTTP信息服务器响应。
*   `http_success` - 2XX HTTP成功服务器响应。
*   `http_not_modified` - 304 HTTP未修改服务器响应。
*   `http_redirect` - 除304之外的3XX HTTP重定向服务器响应。
*   `http_not_found` - 404 HTTP未找到服务器响应。
*   `http_bad_request` - 除404之外的4XX HTTP错误请求服务器响应。
*   `http_server_error` - 5XX HTTP Server错误响应。

可以从以下列表中为这些角色中的每个角色分配特定的前景和背景颜色：

*   `black`
*   `red`
*   `green`
*   `yellow`
*   `blue`
*   `magenta`
*   `cyan`
*   `white`

然后可以使用以下显示选项修改每种颜色：

*   `bold`
*   `underscore`
*   `blink`
*   `reverse`
*   `conceal`

颜色规范遵循以下模式之一：

*   `role=fg`
*   `role=fg/bg`
*   `role=fg,option,option`
*   `role=fg/bg,option,option`

where `role` is the name of a valid color role, `fg` is the foreground color, `bg` is the background color and each `option` is one of the color modifying options. 然后通过分号分隔多个颜色规范。例如：

```
export DJANGO_COLORS="error=yellow/blue,blink;notice=magenta"

```

将指定使用蓝色闪烁的黄色显示错误，并使用品红色显示通知。所有其他颜色的角色将保持不着色。

也可以通过扩展基本调色板来指定颜色。如果您在颜色规范中放置调色板名称，则该调色板隐含的所有颜色将被加载。所以：

```
export DJANGO_COLORS="light;error=yellow/blue,blink;notice=magenta"

```

将指定使用浅色调调色板中的所有颜色，_除了_用于将按指定重写的错误和通知的颜色。

New in Django 1.7.

在Django 1.7中添加了通过依赖于ANSICON应用程序在Windows上支持来自`django-admin` / `manage.py`实用程序的颜色编码输出。

### Bash完成

如果使用Bash shell，请考虑安装Django bash完成脚本，该脚本位于Django发行版中的`extras/django_bash_completion`中。它可以启用`django-admin`和`manage.py`命令的制表符完成，以便您可以，例如...

*   键入`django-admin`。
*   按[TAB]查看所有可用选项。
*   键入`sql`，然后选择[TAB]，以查看其名称以`sql`开头的所有可用选项。

有关如何添加自定义操作，请参阅[_Writing custom django-admin commands_](../howto/custom-management-commands.html)。

