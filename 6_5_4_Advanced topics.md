

# 高级测试主题

## 请求工厂

_class_ `RequestFactory`

[`RequestFactory`](#django.test.RequestFactory "django.test.RequestFactory")与测试客户端共享相同的API。但是，RequestFactory不是像浏览器一样工作，而是提供一种方法来生成可用作任何视图的第一个参数的请求实例。这意味着您可以像测试任何其他函数一样测试视图函数 - 作为一个黑盒子，具有完全已知的输入，测试特定的输出。

[`RequestFactory`](#django.test.RequestFactory "django.test.RequestFactory")的API是测试客户端API的一个稍微受限的子集：

*   It only has access to the HTTP methods [`get()`](tools.html#django.test.Client.get "django.test.Client.get"), [`post()`](tools.html#django.test.Client.post "django.test.Client.post"), [`put()`](tools.html#django.test.Client.put "django.test.Client.put"), [`delete()`](tools.html#django.test.Client.delete "django.test.Client.delete"), [`head()`](tools.html#django.test.Client.head "django.test.Client.head"), [`options()`](tools.html#django.test.Client.options "django.test.Client.options"), and [`trace()`](tools.html#django.test.Client.trace "django.test.Client.trace").
*   这些方法接受所有相同的参数_，除了`follows`的_。因为这只是一个生产请求的工厂，所以由您来处理响应。
*   它不支持中间件。如果视图正常工作需要，会话和身份验证属性必须由测试本身提供。

### 例

以下是使用请求工厂的简单单元测试：

```
from django.contrib.auth.models import AnonymousUser, User
from django.test import TestCase, RequestFactory

from .views import my_view

class SimpleTest(TestCase):
    def setUp(self):
        # Every test needs access to the request factory.
        self.factory = RequestFactory()
        self.user = User.objects.create_user(
            username='jacob', email='jacob@…', password='top_secret')

    def test_details(self):
        # Create an instance of a GET request.
        request = self.factory.get('/customer/details')

        # Recall that middleware are not supported. You can simulate a
        # logged-in user by setting request.user manually.
        request.user = self.user

        # Or you can simulate an anonymous user by setting request.user to
        # an AnonymousUser instance.
        request.user = AnonymousUser()

        # Test my_view() as if it were deployed at /customer/details
        response = my_view(request)
        self.assertEqual(response.status_code, 200)

```

## 测试和多个数据库

### 测试主/副本配置

如果您使用主/副本（由某些数据库称为主/从属）测试多数据库配置，则创建测试数据库的这种策略会出现问题。创建测试数据库时，不会有任何复制，因此，在主节点上创建的数据将不会在副本上看到。

为了弥补这一点，Django允许您定义数据库是_测试镜像_。考虑以下（简化）示例数据库配置：

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'myproject',
        'HOST': 'dbprimary',
         # ... plus some other settings
    },
    'replica': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'myproject',
        'HOST': 'dbreplica',
        'TEST_MIRROR': 'default'
        # ... plus some other settings
    }
}

```

In this setup, we have two database servers: `dbprimary`, described by the database alias `default`, and `dbreplica` described by the alias `replica`. 如您所料，`dbreplica`已由数据库管理员配置为`dbprimary`的只读副本，因此在正常活动中，对`default`将显示在`replica`上。

如果Django创建了两个独立的测试数据库，这将破坏任何期望复制发生的测试。但是，`replica`数据库已配置为测试镜像（使用[`TEST_MIRROR`](../../ref/settings.html#std:setting-TEST_MIRROR)设置），表示在测试下，`replica` `default`的镜像。

配置测试环境时，`replica`的测试版本将_不会_创建。相反，到`replica`的连接将被重定向到指向`default`。因此，对`default`的写入将出现在`replica`上 - 但是因为它们实际上是同一个数据库，而不是因为两个数据库之间存在数据复制。

### 控制测试数据库的创建顺序

默认情况下，Django将假定所有数据库都依赖于`default`数据库，因此，始终首先创建`default`数据库。但是，不能保证测试设置中任何其他数据库的创建顺序。

如果数据库配置需要特定的创建顺序，则可以使用[`TEST_DEPENDENCIES`](../../ref/settings.html#std:setting-TEST_DEPENDENCIES)设置指定存在的依赖关系。考虑以下（简化）示例数据库配置：

```
DATABASES = {
    'default': {
         # ... db settings
         'TEST_DEPENDENCIES': ['diamonds']
    },
    'diamonds': {
        # ... db settings
         'TEST_DEPENDENCIES': []
    },
    'clubs': {
        # ... db settings
        'TEST_DEPENDENCIES': ['diamonds']
    },
    'spades': {
        # ... db settings
        'TEST_DEPENDENCIES': ['diamonds','hearts']
    },
    'hearts': {
        # ... db settings
        'TEST_DEPENDENCIES': ['diamonds','clubs']
    }
}

```

在此配置下，将首先创建`diamonds`数据库，因为它是唯一没有依赖关系的数据库别名。接下来将创建`default`和`clubs`别名（虽然不保证创建此对的顺序）；那么`hearts`；和最后`spades`。

如果在[`TEST_DEPENDENCIES`](../../ref/settings.html#std:setting-TEST_DEPENDENCIES)定义中有任何循环依赖性，则会引发`ImproperlyConfigured`异常。

## 高级功能`TransactionTestCase`

`TransactionTestCase.``available_apps`

警告

此属性是专用API。它可以在未来没有弃用期的情况下更改或删除，例如以适应应用程序加载的更改。

它用于优化Django自己的测试套件，其中包含数百个模型，但在不同应用程序中的模型之间没有关系。

默认情况下，`available_apps`设置为`None`。每次测试后，Django调用[`flush`](../../ref/django-admin.html#django-admin-flush)以重置数据库状态。这将清空所有表并发出[`post_migrate`](../../ref/signals.html#django.db.models.signals.post_migrate "django.db.models.signals.post_migrate")信号，该信号会为每个模型重新创建一个内容类型和三个权限。此操作与模型的数量成比例地变得昂贵。

将`available_apps`设置为应用程序列表指示Django表现为只有这些应用程序中的模型可用。`TransactionTestCase`的行为更改如下：

*   [`post_migrate`](../../ref/signals.html#django.db.models.signals.post_migrate "django.db.models.signals.post_migrate")在每次测试之前触发，以便为可用应用中的每个模型创建内容类型和权限（如果缺少）。
*   每次测试后，Django只清空与可用应用程序中的模型对应的表。但是，在数据库级别，截断可能级联到不可用应用程序中的相关模型。此外，[`post_migrate`](../../ref/signals.html#django.db.models.signals.post_migrate "django.db.models.signals.post_migrate")不会触发；在选择正确的应用程序集后，它将由下一个`TransactionTestCase`触发。

由于数据库未完全刷新，如果测试创建不包含在`available_apps`中的模型实例，则它们将泄漏，并且可能导致无关的测试失败。注意使用会话的测试；默认会话引擎将它们存储在数据库中。

由于冲洗数据库后不会发出[`post_migrate`](../../ref/signals.html#django.db.models.signals.post_migrate "django.db.models.signals.post_migrate")，因此在`TransactionTestCase`之后的状态与`TestCase`后的状态不同：由侦听器创建的行[`post_migrate`](../../ref/signals.html#django.db.models.signals.post_migrate "django.db.models.signals.post_migrate")。考虑[_order in which tests are executed_](overview.html#order-of-tests)，这不是问题，只要给定测试套件中的`TransactionTestCase`声明`available_apps`没有一个。

`available_apps`在Django自己的测试套件中是必需的。

`TransactionTestCase.``reset_sequences`

在`TransactionTestCase`上设置`reset_sequences = True测试运行：`

```
class TestsThatDependsOnPrimaryKeySequences(TransactionTestCase):
    reset_sequences = True

    def test_animal_pk(self):
        lion = Animal.objects.create(name="lion", sound="roar")
        # lion.pk is guaranteed to always be 1
        self.assertEqual(lion.pk, 1)

```

除非明确测试主键序列号，否则建议不要在测试中硬编码主键值。

使用`reset_sequences = True`会降低测试速度，因为主键重置是一个相对昂贵的数据库操作。

## 使用Django测试运行器测试可重用的应用程序

如果您正在编写[_reusable application_](../../intro/reusable-apps.html)，您可能需要使用Django测试运行器来运行自己的测试套件，从而从Django测试基础架构中受益。

通常的做法是应用程序代码旁边的_tests_目录，具有以下结构：

```
runtests.py
polls/
    __init__.py
    models.py
    ...
tests/
    __init__.py
    models.py
    test_settings.py
    tests.py

```

让我们来看看一些这些文件：

runtests.py

```
#!/usr/bin/env python
import os
import sys

import django
from django.conf import settings
from django.test.utils import get_runner

if __name__ == "__main__":
    os.environ['DJANGO_SETTINGS_MODULE'] = 'tests.test_settings'
    django.setup()
    TestRunner = get_runner(settings)
    test_runner = TestRunner()
    failures = test_runner.run_tests(["tests"])
    sys.exit(bool(failures))

```

这是您调用以运行测试套件的脚本。它设置Django环境，创建测试数据库并运行测试。

为了清楚起见，此示例仅包含使用Django测试运行器所需的最低限度。您可能需要添加用于控制详细程度的命令行选项，传递特定测试标签以运行等。

tests/test_settings.py

```
SECRET_KEY = 'fake-key'
INSTALLED_APPS = [
    "tests",
]

```

此文件包含运行应用测试所需的[_Django settings_](../settings.html)。

同样，这是一个最小的例子；您的测试可能需要额外的设置才能运行。

由于在运行测试时，_测试_包包含在[`INSTALLED_APPS`](../../ref/settings.html#std:setting-INSTALLED_APPS)中，因此您可以在其`models.py`文件中定义纯测试模型。

## 使用不同的测试框架

显然，[`unittest`](https://docs.python.org/3/library/unittest.html#module-unittest "(in Python v3.4)")不是唯一的Python测试框架。虽然Django不为替代框架提供显式支持，但它提供了一种方法来调用为替代框架构建的测试，就像他们是正常的Django测试一样。

当您运行`./ manage.py 测试`时，Django会查看[`TEST_RUNNER`](../../ref/settings.html#std:setting-TEST_RUNNER)设置来确定要做什么。默认情况下，[`TEST_RUNNER`](../../ref/settings.html#std:setting-TEST_RUNNER)指向`'django.test.runner.DiscoverRunner'`。这个类定义了默认的Django测试行为。这种行为包括：

1.  执行全局预测试设置。
2.  在名称与模式`test*.py`匹配的当前目录下的任何文件中查找测试。
3.  创建测试数据库。
4.  运行`migrate`将模型和初始数据安装到测试数据库中。
5.  运行发现的测试。
6.  销毁测试数据库。
7.  执行全局后测试拆卸。

如果您在该类中定义了自己的测试运行器类并指向[`TEST_RUNNER`](../../ref/settings.html#std:setting-TEST_RUNNER)，则Django将在您运行`./ manage.py 测试时运行测试`。这样，可以使用可以从Python代码执行的任何测试框架，或者修改Django测试执行过程以满足您可能遇到的任何测试需求。

### 定义测试运行器

测试运行器是定义`run_tests()`方法的类。Django附带一个定义默认Django测试行为的`DiscoverRunner`类。此类定义了`run_tests()`入口点，以及`run_tests()`用来设置，执行和删除测试套件的其他方法。

_class_ `DiscoverRunner`(_pattern='test*.py'_, _top_level=None_, _verbosity=1_, _interactive=True_, _failfast=True_, _keepdb=False_, _reverse=False_, _debug_sql=False_, _**kwargs_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner)

`DiscoverRunner`将在符合`pattern`的任何文件中搜索测试。

`top_level`可用于指定包含顶级Python模块的目录。通常Django可以自动计算出来，所以没有必要指定这个选项。如果指定，通常应该是包含您的`manage.py`文件的目录。

`verbosity`确定将打印到控制台的通知和调试信息量；`0`为无输出，`1`为正常输出，`2`为详细输出。

如果`interactive`是`True`，则测试套件有权在执行测试套件时询问用户指示。此行为的一个示例是请求删除现有测试数据库的权限。如果`interactive`是`False`，则测试套件必须能够在没有任何手动干预的情况下运行。

如果`failfast`为`True`，则在检测到第一个测试失败后，测试套件将停止运行。

如果`keepdb`是`True`，测试套件将使用现有数据库，或者如果需要，创建一个。如果`False`，将创建​​一个新数据库，提示用户删除现有的数据（如果存在）。

如果`reverse`是`True`，测试用例将按相反的顺序执行。这可能有助于调试未正确隔离且具有副作用的测试。使用此选项时，将保留[_Grouping by test class_](overview.html#order-of-tests)。

如果`debug_sql`为`True`，失败的测试用例将输出记录到[_django.db.backends logger_](../logging.html#django-db-logger)的SQL查询以及回溯。如果`verbosity`是`2`，则输出所有测试中的查询。

Django可以不时地通过添加新的参数来扩展测试运行器的能力。`**kwargs`声明允许此扩展。如果您子类化`DiscoverRunner`或编写自己的测试运行器，请确保接受`**kwargs`。

您的测试运行程序还可以定义其他命令行选项。Create or override an `add_arguments(cls, parser)` class method and add custom arguments by calling `parser.add_argument()` inside the method, so that the [`test`](../../ref/django-admin.html#django-admin-test) command will be able to use those arguments.

Changed in Django 1.8:

以前，您必须向子类化测试运行器提供`option_list`属性，以向[`test`](../../ref/django-admin.html#django-admin-test)命令可以使用的命令行选项列表中添加选项。

添加了`keepdb`，`reverse`和`debug_sql`参数。

#### 属性

`DiscoverRunner.``test_suite`

New in Django 1.7.

用于构建测试套件的类。默认情况下，它设置为`unittest.`TestSuite。如果您希望实现不同的收集测试的逻辑，这可以被覆盖。

`DiscoverRunner.``test_runner`

New in Django 1.7.

这是用于执行单独测试和格式化结果的低级测试运行器的类。默认情况下，它设置为`unittest.`TextTestRunner。尽管在命名约定中存在不幸的相似性，但这不是与`DiscoverRunner`相同类型的类，它涵盖了更广泛的职责。您可以覆盖此属性以修改运行和报告测试的方式。

`DiscoverRunner.``test_loader`

这是加载测试的类，无论是从TestCases或模块或其他方式，并将它们捆绑到测试套件中为跑步者执行。默认情况下，它设置为`unittest.defaultTestLoader`。如果您的测试将以不寻常的方式加载，您可以覆盖此属性。

`DiscoverRunner.``option_list`

这是`optparse`选项的元组，它将被送入管理命令的`OptionParser`以解析参数。有关更多详细信息，请参阅Python的`optparse`模块的文档。

自1.8版起已弃用：您现在应该覆盖[`add_arguments()`](#django.test.runner.DiscoverRunner.add_arguments "django.test.runner.DiscoverRunner.add_arguments")类方法以添加由[`test`](../../ref/django-admin.html#django-admin-test)管理命令接受的自定义参数。

#### 方法

`DiscoverRunner.``run_tests`(_test_labels_, _extra_tests=None_, _**kwargs_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner.run_tests)

运行测试套件。

`test_labels`允许您指定要运行的测试并支持多种格式（有关支持的格式的列表，请参阅[`DiscoverRunner.build_suite()`](#django.test.runner.DiscoverRunner.build_suite "django.test.runner.DiscoverRunner.build_suite")）。

`extra_tests`是要添加到测试运行程序执行的套件中的额外`TestCase`实例的列表。除了在`test_labels`中列出的模块中发现的那些测试之外，还运行这些额外的测试。

此方法应返回失败的测试数。

_classmethod_ `DiscoverRunner.``add_arguments`(_parser_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner.add_arguments)

New in Django 1.8.

覆盖此类方法以添加由[`test`](../../ref/django-admin.html#django-admin-test)管理命令接受的自定义参数。请参见[`argparse.`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.add_argument "(in Python v3.4)")ArgumentParser.add_argument()有关将参数添加到解析器的详细信息。

`DiscoverRunner.``setup_test_environment`(_**kwargs_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner.setup_test_environment)

通过调用[`setup_test_environment()`](#django.test.utils.setup_test_environment "django.test.utils.setup_test_environment")并将[`DEBUG`](../../ref/settings.html#std:setting-DEBUG)设置为`False`来设置测试环境。

`DiscoverRunner.``build_suite`(_test_labels_, _extra_tests=None_, _**kwargs_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner.build_suite)

构造与提供的测试标签匹配的测试套件。

`test_labels`是描述要运行的测试的字符串列表。测试标签可以采用以下四种形式之一：

*   `path.to.test_module.`TestCase.test_method - 在测试用例中运行单个测试方法。
*   `path.to.test_module.`TestCase - 在测试用例中运行所有的测试方法。
*   `path.to.module` - 在命名的Python包或模块中搜索并运行所有测试。
*   `path/to/directory` - 搜索并运行命名目录下的所有测试。

如果`test_labels`的值为`None`，则测试运行器将在名称与其`pattern`匹配的当前目录下的所有文件中搜索测试以上）。

`extra_tests`是要添加到测试运行程序执行的套件中的额外`TestCase`实例的列表。除了在`test_labels`中列出的模块中发现的那些测试之外，还运行这些额外的测试。

返回准备运行的`TestSuite`实例。

`DiscoverRunner.``setup_databases`(_**kwargs_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner.setup_databases)

创建测试数据库。

返回一个数据结构，提供足够的详细信息来撤销已做的更改。在测试结束时，此数据将提供给`teardown_databases()`函数。

`DiscoverRunner.``run_suite`(_suite_, _**kwargs_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner.run_suite)

运行测试套件。

返回运行测试套件所产生的结果。

`DiscoverRunner.``teardown_databases`(_old_config_, _**kwargs_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner.teardown_databases)

销毁测试数据库，恢复预测试条件。

`old_config`是定义数据库配置中需要反转的更改的数据结构。它是`setup_databases()`方法的返回值。

`DiscoverRunner.``teardown_test_environment`(_**kwargs_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner.teardown_test_environment)

恢复预测试环境。

`DiscoverRunner.``suite_result`(_suite_, _result_, _**kwargs_)[[source]](../../_modules/django/test/runner.html#DiscoverRunner.suite_result)

计算并返回基于测试套件的返回码，以及该测试套件的结果。

### 测试实用程序

#### django.test.utils

为了帮助创建自己的测试运行器，Django在`django.test.utils`模块中提供了一些实用程序方法。

`setup_test_environment`()[[source]](../../_modules/django/test/utils.html#setup_test_environment)

执行任何全局预测试设置，例如安装模板呈现系统的设置和设置虚拟电子邮件发件箱。

`teardown_test_environment`()[[source]](../../_modules/django/test/utils.html#teardown_test_environment)

执行任何全局后测试拆卸，例如删除模板系统中的黑魔法钩子，并恢复正常的电子邮件服务。

#### django.db.connection.creation

数据库后端的创建模块还提供了一些在测试期间可用的实用程序。

`create_test_db`([_verbosity=1_, _autoclobber=False_, _serialize=True_, _keepdb=False_])

创建新的测试数据库，并对其运行`migrate`。

`verbosity`具有与`run_tests()`中相同的行为。

`autoclobber`描述了如果发现与测试数据库具有相同名称的数据库，则会发生的行为：

*   如果`autoclobber`是`False`，则会要求用户批准销毁现有数据库。如果用户不批准，则调用`sys.exit`。
*   如果autoclobber为`True`，则数据库将被销毁，而无需咨询用户。

`serialize`确定Django在运行测试之前是否将数据库序列化为内存中的JSON字符串（用于在没有事务的情况下在测试之间恢复数据库状态）。如果您没有任何具有[_serialized_rollback=True_](overview.html#test-case-serialized-rollback)的测试类，您可以将其设置为`False`以加快创建时间。

New in Django 1.7.1:

如果您使用默认测试运行器，则可以使用[`TEST`](../../ref/settings.html#std:setting-DATABASE-TEST)字典中的[`SERIALIZE`](../../ref/settings.html#std:setting-TEST_SERIALIZE)条目

`keepdb`确定测试运行是否应使用现有数据库，或创建一个新的数据库。如果`True`，则将使用或创建现有数据库（如果不存在）。如果`False`，将创建​​一个新数据库，提示用户删除现有的数据（如果存在）。

返回它创建的测试数据库的名称。

`create_test_db()`具有修改[`DATABASES`](../../ref/settings.html#std:setting-DATABASES)中[`NAME`](../../ref/settings.html#std:setting-NAME)的值的副作用，以匹配测试数据库的名称。

Changed in Django 1.7:

已添加`serialize`参数。

Changed in Django 1.8:

添加了`keepdb`参数。

`destroy_test_db`(_old_database_name_[, _verbosity=1_, _keepdb=False_])

销毁在[`DATABASES`](../../ref/settings.html#std:setting-DATABASES)中名称为[`NAME`](../../ref/settings.html#std:setting-NAME)的值的数据库，并将[`NAME`](../../ref/settings.html#std:setting-NAME)设置为`old_database_name`的值。

`verbosity`参数与[`DiscoverRunner`](#django.test.runner.DiscoverRunner "django.test.runner.DiscoverRunner")具有相同的行为。

如果`keepdb`参数为`True`，则与数据库的连接将关闭，但数据库不会被销毁。

Changed in Django 1.8:

添加了`keepdb`参数。

## 与coverage.py集成

代码覆盖率描述了已经测试了多少源代码。它显示你的代码的哪些部分是由测试和哪些不是。它是测试应用程序的重要组成部分，因此强烈建议检查测试的覆盖率。

Django可以轻松地与[coverage.py](http://nedbatchelder.com/code/coverage/)集成，这是一个用于测量Python程序代码覆盖率的工具。首先，[安装coverage.py](https://pypi.python.org/pypi/coverage)。接下来，从包含`manage.py`的项目文件夹中运行以下命令：

```
coverage run --source='.' manage.py test myapp

```

这将运行您的测试并收集项目中已执行文件的coverage数据。您可以通过键入以下命令查看此数据的报告：

```
coverage report

```

请注意，一些Django代码在运行测试时已执行，但由于传递给上一个命令的`source`标志，因此未在此处列出。

有关更多选项（如已注释的HTML列表，详细说明错过的行），请参阅[coverage.py](http://nedbatchelder.com/code/coverage/)文档。

