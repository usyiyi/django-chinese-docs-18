

# 编写并运行测试用例

参考

[_testing tutorial_](../../intro/tutorial05.html)，[_testing tools reference_](tools.html)和[_advanced testing topics_](advanced.html)。

本文档分为2个主要单元。首先，我们讲解如何利用Django编写测试.之后，我们讲解如何运行测试。

## 编写测试

Django的单元测试使用的是Python标准库：[`unittest`](https://docs.python.org/3/library/unittest.html#module-unittest "(in Python v3.4)")。该模块是采用基于类的测试。

unittest2

从 1.7 版本开始不推荐使用

Python 2.7对`unittest`库引入了一些重大更改，添加了一些非常有用的功能。为了确保每个Django项目都能从这些新功能中受益，Django使用了一个Python 2.7的`unittest`副本，用于Python 2.6兼容性。

由于Django不再支持2.7以前版本的Python版本，因此不推荐使用`django.utils.unittest`。只需使用`unittest`即可。

下面是一个对[`django.test.TestCase`](tools.html#django.test.TestCase "django.test.TestCase")子类化的实例，前者同时也是[`unittest`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "(in Python v3.4)")的子类。If your tests rely on database access such as creating or querying models, be sure to create your test classes as subclasses of django.test.TestCase rather than unittest.TestCase.Using unittest.TestCase avoids the cost of running each test in a transaction and flushing the database, but if your tests interact with the database their behavior will vary based on the order that the test runner executes them. This can lead to unit tests that pass when run in isolation but fail when run in a suite

```
from django.test import TestCase
from myapp.models import Animal

class AnimalTestCase(TestCase):
    def setUp(self):
        Animal.objects.create(name="lion", sound="roar")
        Animal.objects.create(name="cat", sound="meow")

    def test_animals_can_speak(self):
        """Animals that can speak are correctly identified"""
        lion = Animal.objects.get(name="lion")
        cat = Animal.objects.get(name="cat")
        self.assertEqual(lion.speak(), 'The lion says "roar"')
        self.assertEqual(cat.speak(), 'The cat says "meow"')

```

当你 [_run your tests_](#running-tests),测试工具的默认操作是寻找所有的测试用例（即[`unittest.`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "(in Python v3.4)")的子类TestCase）在名称以`test`开头的任何文件中，自动从这些测试用例构建测试套件，并运行该套件。

更多关于 [`unittest`](https://docs.python.org/3/library/unittest.html#module-unittest "(in Python v3.4)")的细节，请参阅 Python文档。

警告

如果您的测试依赖于数据库访问，例如创建或查询模型，请务必将测试类创建为[`django.test.TestCase`](tools.html#django.test.TestCase "django.test.TestCase")的子类，而不是[`unittest.`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "(in Python v3.4)")测试用例。

使用[`unittest.`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "(in Python v3.4)")TestCase避免了在事务中运行每个测试和刷新数据库的成本，但是如果您的测试与数据库交互，它们的行为将根据测试运行器执行它们的顺序而变化。这可能导致单元测试在孤立运行时传递，但在套件中运行时失败。

## 运行测试

完成测试后，请使用您的项目`manage.py`实用程序的[`test`](../../ref/django-admin.html#django-admin-test)命令运行它们：

```
$ ./manage.py test

```

测试发现基于unittest模块的[_built-in test discovery_](https://docs.python.org/3/library/unittest.html#unittest-test-discovery "(in Python v3.4)")。默认情况下，这将在当前工作目录下名为“test * .py”的任何文件中发现测试。

您可以通过向`./ manage.py 测试`提供任意数量的“测试标签”来指定要运行的特定测试。每个测试标签可以是一个完整的Python虚线路径到一个包，模块，`TestCase`子类或测试方法。例如：

```
# Run all the tests in the animals.tests module
$ ./manage.py test animals.tests

# Run all the tests found within the 'animals' package
$ ./manage.py test animals

# Run just one test case
$ ./manage.py test animals.tests.AnimalTestCase

# Run just one test method
$ ./manage.py test animals.tests.AnimalTestCase.test_animals_can_speak

```

您还可以提供目录的路径，以发现该目录下面的测试：

```
$ ./manage.py test animals/

```

You can specify a custom filename pattern match using the `-p` (or `--pattern`) option, if your test files are named differently from the `test*.py` pattern:

```
$ ./manage.py test --pattern="tests_*.py"

```

如果正在测试的时候按下了 `Ctrl-C` , 测试程序会等待当前正在运行的测试完毕之后再退出。在正常退出期间，测试运行程序将输出任何测试失败的详细信息，报告运行了多少测试以及遇到了多少错误和失败，并像往常一样销毁任何测试数据库。因此，如果您忘记传递[`--failfast`](../../ref/django-admin.html#django-admin-option---failfast)选项，注意某些测试意外失败，并想要获取有关失败的详细信息，那么按`Ctrl-C`而无需等待完整的测试运行完成。

如果您不想等待当前运行的测试完成，您可以再次按`Ctrl-C`，测试运行将立即停止，但不会正常停止。不会报告在中断之前运行的测试的详细信息，并且将不会销毁由运行创建的任何测试数据库。

测试时启用警告

建议您在启用Python警告后执行测试：`python -Wall manage.py 测试 t4 &gt;`。`-Wall`标志告诉Python显示弃用警告。与许多其他Python库一样，Django使用这些警告来标记特性何时消失。它也可以标记你的代码中的区域，不是严格错误，但可以从更好的实现中受益。

### 测试数据库

需要数据库的测试（即模型测试）不会使用您的“真实”（生产）数据库。为测试创建单独的空白数据库。

无论测试通过还是失败，当所有测试都已执行时，测试数据库将被销毁。

New in Django 1.8:

您可以通过向测试命令添加[`--keepdb`](../../ref/django-admin.html#django-admin-option---keepdb)标志来防止测试数据库被破坏。这将在运行之间保留测试数据库。如果数据库不存在，它将首先被创建。任何迁移也将应用，以保持最新。

默认情况下，测试数据库通过在[`DATABASES`](../../ref/settings.html#std:setting-DATABASES)中定义的数据库的[`NAME`](../../ref/settings.html#std:setting-NAME)设置的前面加上`test_`当使用SQLite数据库引擎时，测试将默认使用内存数据库（即，数据库将在内存中创建，完全绕过文件系统）。如果要使用其他数据库名称，请在[`DATABASES`](../../ref/settings.html#std:setting-DATABASES)中的任何给定数据库的[`TEST`](../../ref/settings.html#std:setting-DATABASE-TEST)字典中指定[`NAME`](../../ref/settings.html#std:setting-TEST_NAME)。

Changed in Django 1.7:

在PostgreSQL上，[`USER`](../../ref/settings.html#std:setting-USER)还需要对内置的`postgres`数据库进行读取访问。

除了使用单独的数据库，否则测试运行程序将使用您的设置文件中的所有相同的数据库设置：[`ENGINE`](../../ref/settings.html#std:setting-DATABASE-ENGINE)，[`USER`](../../ref/settings.html#std:setting-USER)，[`HOST`](../../ref/settings.html#std:setting-HOST)测试数据库由[`USER`](../../ref/settings.html#std:setting-USER)指定的用户创建，因此您需要确保给定的用户帐户具有在系统上创建新数据库的足够权限。

要对测试数据库的字符编码进行细粒度控制，请使用[`CHARSET`](../../ref/settings.html#std:setting-TEST_CHARSET) TEST选项。如果您使用MySQL，还可以使用[`COLLATION`](../../ref/settings.html#std:setting-TEST_COLLATION)选项来控制测试数据库使用的特定归类。有关这些和其他高级设置的详细信息，请参阅[_settings documentation_](../../ref/settings.html)。

如果使用带有Python 3.4+和SQLite 3.7.13+的SQLite内存数据库，将启用[共享缓存](https://www.sqlite.org/sharedcache.html)，因此您可以编写测试，以便在线程之间共享数据库。

Changed in Django 1.7:

在[`TEST`](../../ref/settings.html#std:setting-DATABASE-TEST)数据库设置中的不同选项在数据库设置字典中是单独的选项，前缀为`TEST_`。

New in Django 1.8:

添加了使用如上所述的共享缓存的SQLite的能力。

在运行测试时从生产数据库中查找数据？

如果您的代码在编译模块时尝试访问数据库，则会在测试数据库设置之前_发生，可能会出现意外结果。_例如，如果您在模块级代码中有数据库查询，并且存在真实数据库，生产数据可能会污染您的测试。_这是一个坏主意，在你的代码_中有这样的导入时数据库查询 - 重写你的代码，以便它不这样做。

New in Django 1.7:

这也适用于[`ready()`](../../ref/applications.html#django.apps.AppConfig.ready "django.apps.AppConfig.ready")的自定义实现。

也可以看看

[_advanced multi-db testing topics_](advanced.html#topics-testing-advanced-multidb)。

### 执行测试的顺序

为了保证所有`TestCase`代码以干净的数据库开始，Django测试运行器以下列方式重新排序测试：

*   所有[`TestCase`](tools.html#django.test.TestCase "django.test.TestCase")子类首先运行。
*   然后，运行所有其他基于Django的测试（基于[`SimpleTestCase`](tools.html#django.test.SimpleTestCase "django.test.SimpleTestCase")的测试用例，包括[`TransactionTestCase`](tools.html#django.test.TransactionTestCase "django.test.TransactionTestCase")），而不保证或强制执行其中的特定顺序。
*   然后是任何其他[`unittest.`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "(in Python v3.4)")运行可能更改数据库而不将其恢复到其原始状态的TestCase测试（包括doctests）。

注意

测试的新排序可能揭示对测试用例排序的意外依赖。这是依赖于通过给定的[`TransactionTestCase`](tools.html#django.test.TransactionTestCase "django.test.TransactionTestCase")测试在数据库中保留的状态的doctests的情况，它们必须被更新以能够独立运行。

New in Django 1.8:

通过将[`--reverse`](../../ref/django-admin.html#django-admin-option---reverse)传递到测试命令，可以反转组内的执行顺序。这可以帮助确保您的测试彼此独立。

### 回滚仿真

迁移中加载的任何初始数据只能在`TestCase`测试中使用，而不能在`TransactionTestCase`测试中使用，并且只能在支持事务的后端（最重要的例外是MyISAM） 。对于依赖于`TransactionTestCase`（例如[`LiveServerTestCase`](tools.html#django.test.LiveServerTestCase "django.test.LiveServerTestCase")和[`StaticLiveServerTestCase`](../../ref/contrib/staticfiles.html#django.contrib.staticfiles.testing.StaticLiveServerTestCase "django.contrib.staticfiles.testing.StaticLiveServerTestCase")）的测试也是如此。

Django can reload that data for you on a per-testcase basis by setting the `serialized_rollback` option to `True` in the body of the `TestCase` or `TransactionTestCase`, but note that this will slow down that test suite by approximately 3x.

第三方应用程序或针对MyISAM开发的应用程序需要进行设置；但是，一般来说，您应该针对事务数据库开发自己的项目，并且对大多数测试使用`TestCase`，因此不需要此设置。

初始序列化通常非常快速，但如果您希望从此过程中排除某些应用（并加快测试运行速度），您可以将这些应用添加到[`TEST_NON_SERIALIZED_APPS`](../../ref/settings.html#std:setting-TEST_NON_SERIALIZED_APPS)。

没有迁移的应用程序不受影响；`initial_data` fixtures像往常一样重新载入。

### 其他测试条件

无论配置文件中[`DEBUG`](../../ref/settings.html#std:setting-DEBUG)设置的值如何，所有Django测试都将运行[`DEBUG`](../../ref/settings.html#std:setting-DEBUG) = False。这是为了确保您的代码的观察输出与生产设置中将看到的匹配。

缓存不会在每次测试后清除，如果您在生产环境中运行测试，运行“manage.py test fooapp”可以将测试中的数据插入到实时系统的缓存中，因为与数据库不同，单独的“测试缓存”不是用过的。此行为[可能会在未来更改](https://code.djangoproject.com/ticket/11505)。

### 了解测试输出

当您运行测试时，您会看到一些消息，因为测试运行器准备自己。您可以使用命令行上的`verbosity`选项控制这些消息的详细程度：

```
Creating test database...
Creating table myapp_animal
Creating table myapp_mineral
Loading 'initial_data' fixtures...
No fixtures found.

```

这告诉你测试运行器正在创建一个测试数据库，如上一节所述。

一旦创建了测试数据库，Django将运行你的测试。如果一切顺利，你会看到这样的：

```
----------------------------------------------------------------------
Ran 22 tests in 0.221s

OK

```

但是，如果有测试失败，您将看到有关哪些测试失败的完整详细信息：

```
======================================================================
FAIL: test_was_published_recently_with_future_poll (polls.tests.PollMethodTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/dev/mysite/polls/tests.py", line 16, in test_was_published_recently_with_future_poll
    self.assertEqual(future_poll.was_published_recently(), False)
AssertionError: True != False

----------------------------------------------------------------------
Ran 1 test in 0.003s

FAILED (failures=1)

```

此错误输出的完整解释超出了本文档的范围，但它是非常直观。有关详细信息，请参阅Python的[`unittest`](https://docs.python.org/3/library/unittest.html#module-unittest "(in Python v3.4)")库文档。

请注意，对于任何数量的失败和错误测试，测试运行程序脚本的返回码为1。如果所有测试通过，返回码为0。如果您在shell脚本中使用测试运行程序脚本并需要在该级别上测试成功或失败，则此功能非常有用。

### 加快测试速度

在最近的Django版本中，默认密码hasher设计相当慢。如果在测试期间您正在认证许多用户，则可能需要使用自定义设置文件，并将[`PASSWORD_HASHERS`](../../ref/settings.html#std:setting-PASSWORD_HASHERS)设置为更快的散列算法：

```
PASSWORD_HASHERS = (
    'django.contrib.auth.hashers.MD5PasswordHasher',
)

```

不要忘记也可以在[`PASSWORD_HASHERS`](../../ref/settings.html#std:setting-PASSWORD_HASHERS)中包含灯具中使用的任何散列算法（如果有的话）。

