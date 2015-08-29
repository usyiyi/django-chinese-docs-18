# 编写你的第一个 Django 程序 第1部分 #

让我们通过例子来学习。

在本教程中，我们将引导您创建一个基本的投票应用。

它将包含两部分：

+ 一个公共网站，可让人们查看投票的结果和让他们进行投票。
+ 一个管理网站，可让你添加、修改和删除投票项目。

我们假设你已经 安装了 Django 。你可以运行以下命令来验证是否已经安装了 Django 和运行着的版本号：

```
python -c "import django; print(django.get_version())"
```

你应该看到你安装的 Django 版本或一个提示你 “No module named django” 的错误。此外，还应该检查下你的版本与本教程的版本是否一致。 若不一致，你可以参考 Django 版本对应的教程或者更新 Django 到最新版本。

请参考 如何安装 Django 中的意见先删除旧版本的 Django 再安装一个新的。

> 在哪里可以获得帮助：
>
> 如果您在学习本教程中遇到问题，请在 django-users 上发贴或者在 #django on irc.freenode.net 上与其他可能会帮助您的 Django 用户交流。

## 创建一个项目 ##

如果这是你第一次使用 Django ，那么你必须进行一些初始设置。也就是通过自动生成代码来建立一个 Django 项目 project – 一个 Django 项目的设置集，包含了数据库配置、 Django 详细选项设置和应用特性配置。

在命令行中，使用 cd 命令进入你想存储代码所在的目录，然后运行以下命令：

```
django-admin.py startproject mysite
```

这将在当前目录创建一个 mysite 目录。如果失败了，请查看 Problems running django-admin.py.

> Note
>
> 你需要避免使用 python 保留字或 Django 组件名作为项目的名称。尤其是你应该避免使用的命名如： django (与 Django 本身会冲突) 或者 test (与 Python 内置的包名会冲突).

> 这段代码应该放在哪里？
>
> 如果你有一般 PHP 的编程背景（未使用流行的框架），可能会将你的代码放在 Web 服务器的文档根目录下（例如：``/var/www``）。而在 Django 中，你不必这么做。将任何 Python 代码放在你的 Web 服务器文档根目录不会是一个好主意，因为这可能会增加人们通过 Web 方式查看到你的代码的风险。这不利于安全。

将你的代码放在你的文档根目录 以外 的某些目录, 例如 /home/mycode 。

让我们来看看 startproject 都创建了些什么:

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

> 和你看到的不一样？
>
> 默认的项目布局最近刚刚改变过。如果你看到的是一个“扁平”结构的目录布局（没有内层 mysite/ 目录），你很可能正在使用一个和本教程版本不一致的 Django 版本。你需要切换到对应的旧版教程或者使用较新的 Django 版本。

这些文件是：

+ 外层 mysite/ 目录只是你项目的一个容器。对于 Django 来说该目录名并不重要; 你可以重命名为你喜欢的。
+ manage.py: 一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互。 你可以在 django-admin.py and manage.py 中查看关于 manage.py 所有的细节。
+ 内层 mysite/ 目录是你项目中的实际 Python 包。该目录名就是 Python 包名，通过它你可以导入它里面的任何东西。 (e.g. import mysite.settings).
+ mysite/__init__.py: 一个空文件，告诉 Python 该目录是一个 Python 包。(如果你是 Python 新手，请查看官方文档了解 关于包的更多内容 。)
+ mysite/settings.py: 该 Django 项目的设置/配置。请查看 Django settings 将会告诉你如何设置。
+ mysite/urls.py: 该 Django 项目的 URL 声明; 一份由 Django 驱动的网站“目录”。请查看 URL dispatcher 可以获取更多有关 URL 的信息。
+ mysite/wsgi.py: 一个 WSGI 兼容的 Web 服务器的入口，以便运行你的项目。请查看 How to deploy with WSGI 获取更多细节。

## 开发用服务器 ##

让我们来验证是否工作。从外层 mysite 目录切换进去，若准备好了就运行命令 ``python manage.py runserver``。你将会看到命令行输出如下内容：

```
Performing system checks...

0 errors found
May 13, 2015 - 15:50:53
Django version 1.8, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

你已经启动了 Django 开发服务器，一个纯粹的由 Python 编写的轻量级 Web 服务器。我们在 Django 内包含了这个服务器，这样你就可以迅速开发了，在产品投入使用之前不必去配置一台生产环境下的服务器 – 例如 Apache 。

现在是一个很好的提示时机：**不要** 在任何类似生产环境中使用此服务器。它仅适用于开发环境。(我们提供的是 Web 框架的业务，而不是 Web 服务器。)

现在服务器正在运行中，请在你的 Web 浏览器中访问 http://127.0.0.1:8000/ 。 你会看到一个令人愉悦的，柔和的淡蓝色 “Welcome to Django” 页面。它工作正常！

> 更改端口号
>
> 默认情况下，runserver 命令启动的开发服务器只监听本地 IP 的 8000 端口。
>
> 如果你想改变服务器的端口，把它作为一个命令行参数传递即可。例如以下命令启动的服务器将监听 8080 端口：
>
```
python manage.py runserver 8080
```
>
> 如果你想改变服务器 IP ，把它和端口号一起传递即可。因此，要监听所有公共 IP 地址（如果你想在其他电脑上炫耀你的工作），请使用：
>
```
python manage.py runserver 0.0.0.0:8000
```
> 有关开发服务器的完整文档可以在 runserver 内参考。

## 数据库设置 ##

现在，编辑 mysite/settings.py 。 这是一个普通的 Python 模块，包含了代表 Django 设置的模块级变量。 更改 DATABASES 中 'default' 下的以下键的值，以匹配您的数据库连接设置。

+ ENGINE – 从 'django.db.backends.postgresql_psycopg2', 'django.db.backends.mysql', 'django.db.backends.sqlite3', 'django.db.backends.oracle' 中选一个， 至于其他请查看 also available.
+ NAME – 你的数据库名。如果你使用 SQLite，该数据库将是你计算机上的一个文件；在这种情况下，NAME 将是一个完整的绝对路径，而且还包含该文件的名称。如果该文件不存在，它会在第一次同步数据库时自动创建（见下文）。

当指定路径时，总是使用正斜杠，即使是在 Windows 下(例如：``C:/homes/user/mysite/sqlite3.db``) 。

+ USER – 你的数据库用户名 ( SQLite 下不需要) 。
+ PASSWORD – 你的数据库密码 ( SQLite 下不需要) 。
+ HOST – 你的数据库主机地址。如果和你的数据库服务器是同一台物理机器，请将此处保留为空 (或者设置为 127.0.0.1) ( SQLite 下不需要) 。查看 HOST 了解详细信息。

如果你是新建数据库，我们建议只使用 SQLite ，将 ENGINE 改为 'django.db.backends.sqlite3' 并且将 NAME 设置为你想存放数据库的地方。 SQLite 是内置在 Python 中的，因此你不需要安装任何东西来支持你的数据库。

> Note
>
> 如果你使用 PostgreSQL 或者 MySQL，确保你已经创建了一个数据库。还是通过你的数据库交互接口中的 “CREATE DATABASE database_name;” 命令做到这一点的。
> 如果你使用 SQLite ，你不需要事先创建任何东西 - 在需要的时候，将会自动创建数据库文件。
当你编辑 settings.py 时，将 TIME_ZONE 修改为你所在的时区。默认值是美国中央时区（芝加哥）。

同时，注意文件底部的 INSTALLED_APPS 设置。它保存了当前 Django 实例已激活的所有 Django 应用。每个应用可以被多个项目使用，而且你可以打包和分发给其他人在他们的项目中使用。

默认情况下，INSTALLED_APPS 包含以下应用，这些都是由 Django 提供的：

+ django.contrib.auth – 身份验证系统。
+ django.contrib.contenttypes – 内容类型框架。
+ django.contrib.sessions – session 框架。
+ django.contrib.sites – 网站管理框架。
+ django.contrib.messages – 消息框架。
+ django.contrib.staticfiles – 静态文件管理框架。

这些应用在一般情况下是默认包含的。

所有这些应用中每个应用至少使用一个数据库表，所以在使用它们之前我们需要创建数据库中的表。要做到这一点，请运行以下命令：

```
python manage.py syncdb
```

syncdb 命令参照 INSTALLED_APPS 设置，并在你的 settings.py 文件所配置的数据库中创建必要的数据库表。每创建一个数据库表你都会看到一条消息，接着你会看到一个提示询问你是否想要在身份验证系统内创建个超级用户。按提示输入后结束。

如果你感兴趣，可以在你的数据库命令行下输入：``dt`` (PostgreSQL), SHOW TABLES; (MySQL), 或 .schema (SQLite) 来列出 Django 所创建的表。

> 极简主义者
>
> 就像我们上面所说的，一般情况下以上应用都默认包含在内，但不是每个人都需要它们。如果不需要某些或全部应用，在运行 syncdb 命令前可从 INSTALLED_APPS 内随意注释或删除相应的行。syncdb 命令只会为 INSTALLED_APPS 内的应用创建表。

## 创建模型 ##

现在你的项目开发环境建立好了， 你可以开工了。

你通过 Djaong 编写的每个应用都是由 Python 包组成的，这些包存放在你的 Python path 中并且遵循一定的命名规范。 Django 提供了个实用工具可以自动生成一个应用的基本目录架构，因此你可以专注于编写代码而不是去创建目录。

> 项目 ( Projects ) vs. 应用 ( apps )
>
> 项目与应用之间有什么不同之处？应用是一个提供功能的 Web 应用 – 例如：一个博客系统、一个公共记录的数据库或者一个简单的投票系统。 项目是针对一个特定的 Web 网站相关的配置和其应用的组合。一个项目可以包含多个应用。一个应用可以在多个项目中使用。

你的应用可以存放在 Python path 中的任何位置。在本教材中，我们将通过你的 manage.py 文件创建我们的投票应用，以便它可以作为顶层模块导入，而不是作为 mysite 的子模块。

要创建你的应用，请确认与 manage.py 文件在同一的目录下并输入以下命令：

```
python manage.py startapp polls
```

这将创建一个 polls 目录，其展开的样子如下所示：:

```
polls/
    __init__.py
    models.py
    tests.py
    views.py
```

此目录结构就是投票应用。

在 Django 中编写一个有数据库支持的 Web 应用的第一步就是定义你的模型 – 从本质上讲就是数据库设计及其附加的元数据。

> 哲理
>
> 模型是有关你数据的唯一且明确的数据源。它包含了你所要存储的数据的基本字段和行为。 Django 遵循 DRY 原则 。目标是为了只在一个地方定义你的数据模型就可从中自动获取数据。

在这简单的投票应用中，我们将创建两个模型： Poll 和 Choice``。``Poll 有问题和发布日期两个字段。``Choice`` 有两个字段： 选项 ( choice ) 的文本内容和投票数。每一个 Choice 都与一个 Poll 关联。

这些概念都由简单的 Python 类来表现。编辑 polls/models.py 文件后如下所示：

```
from django.db import models

class Poll(models.Model):
    question = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    poll = models.ForeignKey(Poll)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

代码很简单。每个模型都由继承自 django.db.models.Model 子类的类来描述。 每个模型都有一些类变量，每一个类变量都代表了一个数据库字段。

每个字段由一个 Field 的实例来表现 – 比如 CharField 表示字符类型的字段和 DateTimeField 表示日期时间型的字段。这会告诉 Django 每个字段都保存了什么类型的数据。

每一个 Field 实例的名字就是字段的名字（如： question 或者 pub_date ），其格式属于亲和机器式的。在你的 Python 的代码中会使用这个值，而你的数据库会将这个值作为表的列名。

你可以在初始化 Field 实例时使用第一个位置的可选参数来指定人类可读的名字。这在Django的内省部分中被使用到了，而且兼作文档的一部分来增强代码的可读性。若字段未提供该参数，Django 将使用符合机器习惯的名字。在本例中，我们仅定义了一个符合人类习惯的字段名 Poll.pub_date 。对于模型中的其他字段，机器名称就已经足够替代人类名称了。

一些 Field 实例是需要参数的。 例如 CharField 需要你指定 `~django.db.models.CharField.max_length`。这不仅适用于数据库结构，以后我们还会看到也用于数据验证中。

一个 Field 实例可以有不同的可选参数； 在本例中，我们将 votes 的 default 的值设为 0 。

最后，注意我们使用了 ForeignKey 定义了一个关联。它告诉 Django 每一个``Choice`` 关联一个 Poll 。 Django 支持常见数据库的所有关联：多对一（ many-to-ones ），多对多（ many-to-manys ） 和 一对一 （ one-to-ones ）。

## 激活模型 ##

刚才那点模型代码提供给 Django 大量信息。有了这些 Django 就可以做：

为该应用创建对应的数据库架构 (CREATE TABLE statements) 。
为 Poll 和 Choice 对象创建 Python 访问数据库的 API 。
但首先，我们需要告诉我们的项目已经安装了 polls 应用。

> 哲理
>
> Django 应用是“可插拔的”：你可以在多个项目使用一个应用，你还可以分发应用，因为它们没有被捆绑到一个给定的 Django 安装环境中。

再次编辑 settings.py 文件，在 INSTALLED_APPS 设置中加入 'polls' 字符。因此结果如下所示：

```
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Uncomment the next line to enable the admin:
    # 'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
    # 'django.contrib.admindocs',
    'polls',
)
```

现在 Django 已经知道包含了 polls 应用。让我们运行如下命令：

```
python manage.py sql polls
```

你将看到类似如下所示内容 ( 有关投票应用的 CREATE TABLE SQL 语句 )：

```
BEGIN;
CREATE TABLE "polls_poll" (
    "id" serial NOT NULL PRIMARY KEY,
    "question" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
CREATE TABLE "polls_choice" (
    "id" serial NOT NULL PRIMARY KEY,
    "poll_id" integer NOT NULL REFERENCES "polls_poll" ("id") DEFERRABLE INITIALLY DEFERRED,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL
);
COMMIT;
```

请注意如下事项：

+ 确切的输出内容将取决于您使用的数据库会有所不同。
+ 表名是自动生成的，通过组合应用名 (polls) 和小写的模型名 – poll 和 choice 。 ( 你可以重写此行为。)
+ 主键 (IDs) 是自动添加的。( 你也可以重写此行为。)
+ 按照惯例，Django 会在外键字段名上附加 "_id" 。 ( 是的，你仍然可以重写此行为。)
+ 外键关系由 REFERENCES 语句显示声明。
+ 生成 SQL 语句时针对你所使用的数据库，会为你自动处理特定于数据库的字段，例如 auto_increment (MySQL), serial (PostgreSQL), 或 or integer primary key (SQLite) 。 在引用字段名时也是如此 – 比如使用双引号或单引号。 本教材的作者所使用的是 PostgreSQL，因此例子中输出的是 PostgreSQL 的语法。
+ 这些 sql 命令其实并没有在你的数据库中运行过 - 它只是在屏幕上显示出来，以便让你了解 Django 认为什么样的 SQL 是必须的。 如果你愿意，可以把 SQL 复制并粘帖到你的数据库命令行下去执行。 但是，我们很快就能看到， Django 提供了一个更简单的方法来执行此 SQL 。

如果你感兴趣，还可以运行以下命令：

+ python manage.py validate – 检查在构建你的模型时是否有错误。
+ python manage.py sqlcustom polls – 输出为应用定义的任何 custom SQL statements ( 例如表或约束的修改 ) 。
+ python manage.py sqlclear polls – 根据存在于你的数据库中的表 (如果有的话) ，为应用输出必要的 DROP TABLE 。
+ python manage.py sqlindexes polls – 为应用输出 CREATE INDEX 语句。
+ python manage.py sqlall polls – 输出所有 SQL 语句：sql, sqlcustom, 和 sqlindexes 。

看看这些输出的命令可以帮助你理解框架底层实际上处理了些什么。

现在，再次运行 syncdb 命令在你的数据库中创建这些模型对应的表：

```
python manage.py syncdb
```

syncdb 命令会给在 INSTALLED_APPS 中有但数据库中没有对应表的应用执行 sqlall 操作。 该操作会为你上一次执行 syncdb 命令以来在项目中添加的任何应用创建对应的表、初始化数据和创建索引。 syncdb 命令只要你喜欢就可以任意调用，并且它仅会创建不存在的表。

请阅读 django-admin.py documentation 文档了解 manage.py 工具更多的功能。

## 玩转 API ##

现在，我们进入 Python 的交互式 shell 中玩弄 Django 提供给你的 API 。要调用 Python sell ，使用如下命令：

```
python manage.py shell
```

我们当前使用的环境不同于简单的输入 “python” 进入的 shell 环境，因为 manage.py 设置了 DJANGO_SETTINGS_MODULE 环境变量，该变量给定了 Django 需要导入的 settings.py 文件所在路径。

> 忽略 manage.py
>
> 若你不想使用 manage.py ，也是没有问题的。 仅需要将 DJANGO_SETTINGS_MODULE 环境变量值设为 mysite.settings 并在与 manage.py 文件所在同一目录下运行 python （ 或确保目录在 Python path 下，那 import mysite 就可以了 ）。
>
> 想了解更多的信息，请参考 django-admin.py 文档 。

一旦你进入了 shell，就可通过 database API 来浏览数据：:

```
>>> from polls.models import Poll, Choice   # Import the model classes we just wrote.

#  系统中还没有 polls 。
>>> Poll.objects.all()
[]

# 创建一个新 Poll 。
# 在默认配置文件中时区支持配置是启用的，
# 因此 Django 希望为 pub_date 字段获取一个 datetime  with tzinfo 。使用了 timezone.now()
# 而不是 datetime.datetime.now() 以便获取正确的值。
>>> from django.utils import timezone
>>> p = Poll(question="What's new?", pub_date=timezone.now())

# 保存对象到数据库中。你必须显示调用 save() 方法。
>>> p.save()

# 现在对象拥有了一个ID 。请注意这可能会显示 "1L" 而不是 "1"，取决于
# 你正在使用的数据库。 这没什么大不了的，它只是意味着你的数据库后端
# 喜欢返回的整型数作为 Python 的长整型对象而已。
>>> p.id
1

# 通过 Python 属性访问数据库中的列。
>>> p.question
"What's new?"
>>> p.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# 通过改为属性值来改变值，然后调用 save() 方法。
>>> p.question = "What's up?"
>>> p.save()

# objects.all() 用以显示数据库中所有的 polls 。
>>> Poll.objects.all()
[<Poll: Poll object>]
```

请稍等。``<Poll: Poll object>`` 这样显示对象绝对是无意义的。 让我们编辑 polls 模型（ 在 polls/models.py 文件中 ） 并且给 Poll 和 Choice 都添加一个 __unicode__() 方法来修正此错误：

```
class Poll(models.Model):
    # ...
    def __unicode__(self):
        return self.question

class Choice(models.Model):
    # ...
    def __unicode__(self):
        return self.choice_text
```

给你的模型添加 __unicode__() 方法是很重要的， 不仅是让你在命令行下有明确提示，而且在 Django 自动生成的管理界面中也会使用到对象的呈现。

> 为什么是 __unicode__() 而不是 __str__()?
>
> 如果你熟悉 Python，那么你可能会习惯在类中添加 __str__() 方法而不是 __unicode__() 方法。 We use 我们在这里使用 __unicode__() 是因为 Django 模型默认处理的是 Unicode 格式。当所有存储在数据库中的数据返回时都会转换为 Unicode 的格式。
>
> Django 模型有个默认的 __str__() 方法 会去调用 __unicode__() 并将结果转换为 UTF-8 编码的字符串。这就意味着 unicode(p) 会返回一个 Unicode 字符串，而 str(p) 会返回一个以 UTF-8 编码的普通字符串。
>
> 如果这让你感觉困惑，那么你只要记住在模型中添加 __unicode__() 方法。 运气好的话，这些代码会正常运行。

请注意这些都是普通的 Python 方法。让我们来添加个自定义方法，为了演示而已：

```
import datetime
from django.utils import timezone
# ...
class Poll(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

请注意，增加了 import datetime 和 from django.utils import timezone, 是为了分别引用 Python 的标准库 datetime 模块和 Django 的 django.utils.timezone 中的 time-zone-related 实用工具 。如果你不熟悉在 Python 中处理时区，你可以在 时区支持文档 学到更多。

保存这些更改并且再次运行 python manage.py shell 以开启一个新的 Python shell:

```
>>> from polls.models import Poll, Choice

# 确认我们附加的  __unicode__() 正常运行。
>>> Poll.objects.all()
[<Poll: What's up?>]

# Django 提供了一个丰富的数据库查询 API ，
# 完全由关键字参数来驱动。
>>> Poll.objects.filter(id=1)
[<Poll: What's up?>]
>>> Poll.objects.filter(question__startswith='What')
[<Poll: What's up?>]

# 获取今年发起的投票。
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Poll.objects.get(pub_date__year=current_year)
<Poll: What's up?>

# 请求一个不存在的 ID ，这将引发一个异常。
>>> Poll.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Poll matching query does not exist. Lookup parameters were {'id': 2}

# 根据主键查询是常见的情况，因此 Django 提供了一个
# 主键精确查找的快捷方式。
# 以下代码等同于 Poll.objects.get(id=1).
>>> Poll.objects.get(pk=1)
<Poll: What's up?>

# 确认我们自定义方法正常运行。
>>> p = Poll.objects.get(pk=1)
>>> p.was_published_recently()
True

# 给 Poll 设置一些 Choices 。通过 create 方法调用构造方法去创建一个新
# Choice 对象实例，执行 INSERT 语句后添加该 choice 到
# 可用的 choices 集中并返回这个新建的 Choice 对象实例。 Django 创建了
# 一个保存外键关联关系的集合 ( 例如 poll 的 choices) 以便可以通过 API
# 去访问。
>>> p = Poll.objects.get(pk=1)

# 从关联对象集中显示所有 choices  -- 到目前为止还没有。
>>> p.choice_set.all()
[]

# 创建三个 choices 。
>>> p.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> p.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = p.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice 对象拥有访问它们关联的 Poll 对象的 API 。
>>> c.poll
<Poll: What's up?>

# 反之亦然： Poll 对象也可访问 Choice 对象。
>>> p.choice_set.all()
[<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]
>>> p.choice_set.count()
3

# 只要你需要 API 会自动连续关联。
# 使用双下划线来隔离关联。
# 只要你想要几层关联就可以有几层关联，没有限制。
# 寻找和今年发起的任何 poll 有关的所有 Choices
# ( 重用我们在上面建立的 'current_year' 变量 )。
>>> Choice.objects.filter(poll__pub_date__year=current_year)
[<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]

# 让我们使用 delete() 删除 choices 中的一个。
>>> c = p.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

欲了解更多有关模型关系的信息，请查看 访问关联对象 。欲了解更多有关如何使用双下划线来通过 API 执行字段查询的，请查看 字段查询 。 如需完整的数据库 API 信息，请查看我们的 数据库 API 参考 。

当你对 API 有所了解后, 请查看 教程 第2部分 来学习 Django 的自动生成的管理网站是如何工作的。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Part 1: Models](https://docs.djangoproject.com/en/1.8/intro/tutorial01/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
