# 编写自定义的django-admin命令 #

应用可以通过`manage.py`注册它们自己的动作。例如，你可能想为你正在发布的Django应用添加一个`manage.py`动作。在本页文档中，我们将为教程中的 `polls`应用构建一个自定义的 `closepoll`命令。

要做到这点，只需向该应用添加一个`management/commands`目录。Django将为该目录中名字没有以下划线开始的每个Python模块注册一个`manage.py`命令。例如：

```
polls/
    __init__.py
    models.py
    management/
        __init__.py
        commands/
            __init__.py
            _private.py
            closepoll.py
    tests.py
    views.py
```

在Python 2上，请确保`management`和`management/commands`两个目录都包含`__init__.py` 文件，否则将检测不到你的命令。

在这个例子中，`closepoll`命令对任何项目都可使用，只要它们在`INSTALLED_APPS`里包含`polls`应用。

`_private.py`将不可以作为一个管理命令使用。

`closepoll.py`模块只有一个要求 – 它必须定义一个`Command`类并扩展自`BaseCommand`或其 子类。

> 独立的脚本
>
> 自定义的管理命令主要用于运行独立的脚本或者UNIX crontab和Windows周期任务控制面板周期性执行的脚本。

要实现这个命令，需将`polls/management/commands/closepoll.py`编辑成这样：

```
from django.core.management.base import BaseCommand, CommandError
from polls.models import Poll

class Command(BaseCommand):
    help = 'Closes the specified poll for voting'

    def add_arguments(self, parser):
        parser.add_argument('poll_id', nargs='+', type=int)

    def handle(self, *args, **options):
        for poll_id in options['poll_id']:
            try:
                poll = Poll.objects.get(pk=poll_id)
            except Poll.DoesNotExist:
                raise CommandError('Poll "%s" does not exist' % poll_id)

            poll.opened = False
            poll.save()

            self.stdout.write('Successfully closed poll "%s"' % poll_id)
```

```
Changed in Django 1.8:

在Django 1.8之前，管理命令基于optparse模块，位置参数传递给*args，可选参数传递给**options。现在，管理命令使用argparse解析参数，默认所有的参数都传递给**options，除非你命名你的位置参数为args（兼容模式）。对于新的命令，鼓励你仅仅使用**options。
```

> 注
>
> 当你使用管理命令并希望提供控制台输出时，你应该写到`self.stdout`和`self.stderr`，而不能直接打印到 `stdout`和`stderr`。通过使用这些代理方法，测试你自定义的命令将变得非常容易。还请注意，你不需要在消息的末尾加上一个换行符，它将被自动添加，除非你指定`ending`参数：
>
```
self.stdout.write("Unterminated line", ending='')
```
>

新的自定义命令可以使用`python manage.py closepoll <poll_id>`调用。

`handle()`接收一个或多个`poll_ids`并为他们中的每个设置 `poll.opened`为`False`。如果用户访问任何不存在的`polls`，将引发一个`CommandError`。`poll.opened`属性在教程中并不存在，只是为了这个例子将它添加到`polls.models.Poll`中。

## 接收可选参数 ##

通过接收额外的命令行选项，可以简单地修改`closepoll`来删除一个给定的`poll`而不是关闭它。这些自定义的选项可以像下面这样添加到 `add_arguments()`方法中：

```
class Command(BaseCommand):
    def add_arguments(self, parser):
        # Positional arguments
        parser.add_argument('poll_id', nargs='+', type=int)

        # Named (optional) arguments
        parser.add_argument('--delete',
            action='store_true',
            dest='delete',
            default=False,
            help='Delete poll instead of closing it')

    def handle(self, *args, **options):
        # ...
        if options['delete']:
            poll.delete()
        # ...
```

```
Changed in Django 1.8:

之前，只支持标准的optparse库，你必须利用optparse.make_option()扩展命令option_list变量。
```

选项（在我们的例子中为`delete`）在`handle`方法的`options`字典参数中可以访问到。更多关于`add_argument`用法的信息，请参考`argparse`的Python 文档。

除了可以添加自定义的命令行选项， 管理命令还可以接收一些默认的选项，例如`--verbosity`和`--traceback`。

## 管理命令和区域设置 ##

默认情况下，`BaseCommand.execute()`方法使转换失效，因为某些与Django一起的命令完成的任务要求一个与项目无关的语言字符串（例如，面向用户的内容渲染和数据库填入）。

```
Changed in Django 1.8:

在之前的版本中，Django强制使用"en-us"区域设置而不是使转换失效。
```

如果，出于某些原因，你的自定义的管理命令需要使用一个固定的区域设置，你需要在你的`handle()`方法中利用I18N支持代码提供的函数手工地启用和停用它：

```
from django.core.management.base import BaseCommand, CommandError
from django.utils import translation

class Command(BaseCommand):
    ...
    can_import_settings = True

    def handle(self, *args, **options):

        # Activate a fixed locale, e.g. Russian
        translation.activate('ru')

        # Or you can activate the LANGUAGE_CODE # chosen in the settings:
        from django.conf import settings
        translation.activate(settings.LANGUAGE_CODE)

        # Your command logic here
        ...

        translation.deactivate()
```

另一个需要可能是你的命令只是简单地应该使用设置中设置的区域设置且Django应该保持不让它停用。你可以使用`BaseCommand.leave_locale_alone`选项实现这个功能。

虽然上面描述的场景可以工作，但是考虑到系统管理命令对于运行非统一的区域设置通常必须非常小心，所以你可能需要：

+ 确保运行命令时`USE_I18N`设置永远为`True`（this is a good example of the potential problems stemming from a dynamic runtime environment that Django commands avoid offhand by deactivating translations）。
+ Review the code of your command and the code it calls for behavioral differences when locales are changed and evaluate its impact on predictable behavior of your command.

## 测试 ##

关于如何测试自定义管理命令的信息可以在[测试文档](http://python.usyiyi.cn/django/topics/testing/tools.html#topics-testing-management-commands)中找到。

## Command 对象 ##

`class BaseCommand`

所有管理命令最终继承的基类。

如果你想获得解析命令行参数并在响应中如何调用代码的所有机制，可以使用这个类；如果你不需要改变这个行为，请考虑使用它的子类。

继承`BaseCommand`类要求你实现`handle()`方法。

### 属性 ###

所有的属性都可以在你派生的类中设置，并在`BaseCommand`的子类中使用。

`BaseCommand.args`

一个字符串，列出命令接收的参数，适合用于帮助信息；例如，接收一个应用名称列表的命令可以设置它为‘`<app_label app_label ...>`’。

```
Deprecated since version 1.8:

现在，应该在add_arguments()方法中完成，通过调用parser.add_argument()方法。参见上面的closepoll例子。
```

`BaseCommand.can_import_settings`

一个布尔值，指示该命令是否需要导入Django的设置的能力；如果为`True`，`execute()`将在继续之前验证这是否可能。默认值为`True`。

`BaseCommand.help`

命令的简短描述，当用户运行`python manage.py help <command>`命令时将在帮助信息中打印出来。

`BaseCommand.missing_args_message`

```
New in Django 1.8.
```

如果你的命令定义了必需的位置参数，你可以自定义参数缺失时返回的错误信息。默认是由`argparse`输出的 (“too few arguments”)。

`BaseCommand.option_list`

这是optparse选项列表，将赋值给命令的OptionParser用于解析命令。

```
Deprecated since version 1.8:

现在，你应该覆盖`add_arguments()`方法来添加命令行接收的自定义参数。参见上面的例子。
```

`BaseCommand.output_transaction`

一个布尔值，指示命令是否输出SQL语句；如果为`True`，输出将被自动用`BEGIN;`和`COMMIT;`封装。默认为`False`。

`BaseCommand.requires_system_checks`

```
New in Django 1.7.
```

一个布尔值；如果为`True`，在执行该命令之前将检查整个Django项目是否有潜在的问题。如果`requires_system_checks`缺失，则使用`requires_model_validation`的值。如果后者的值也缺失，则使用默认值（`True`）。同时定义`requires_system_checks`和`requires_model_validation`将导致错误。

`BaseCommand.requires_model_validation`

```
Deprecated since version 1.7:

被requires_system_checks代替
```

一个布尔值；如果为`True`，将在执行命令之前作安装的模型的验证。默认为`True`。若要验证一个单独应用的模型而不是全部应用的模型，可以调用在`handle()`中调用`validate()`。

`BaseCommand.leave_locale_alone`

一个布尔值，指示设置中的区域设置在执行命令过程中是否应该保持而不是强制设成‘en-us’。

默认值为`False`。

如果你决定在你自定义的命令中修改该选项的值，请确保你知道你正在做什么。 如果它创建对区域设置敏感的数据库内容，这种内容不应该包含任何转换（比如`django.contrib.auth`权限发生的情况），因为将区域设置变成与实际上默认的‘en-us’ 不同可能导致意外的效果。更进一步的细节参见上面的[管理命令和区域设置](http://python.usyiyi.cn/django/howto/custom-management-commands.html#id1)一节。

当`can_import_settings`选项设置为`False`时，该选项不可以也为`False`，因为尝试设置区域设置需要访问`settings`。这种情况将产生一个`CommandError`。

## 方法 ##

`BaseCommand`有几个方法可以被覆盖，但是只有`handle()`是必须实现的。

> 在子类中实现构造函数
>
> 如果你在`BaseCommand的`子类中实现`__init__`，你必须调用`BaseCommand`的`__init__`：
>
```
class Command(BaseCommand):
    def __init__(self, *args, **kwargs):
        super(Command, self).__init__(*args, **kwargs)
        # ...
```
>

`BaseCommand.add_arguments(parser)`

```
New in Django 1.8.
```

添加解析器参数的入口，以处理传递给命令的命令行参数。自定义的命令应该覆盖这个方法以添加命令行接收的位置参数和可选参数。当直接继承`BaseCommand`时不需要调用`super()`。

`BaseCommand.get_version()`

返回Django的版本，对于所有内建的Django命令应该都是正确的。用户提供的命令可以覆盖这个方法以返回它们自己的版本。

`BaseCommand.execute(*args, **options)`

执行这个命令，如果需要则作系统检查（通过 `requires_system_checks`属性控制）。如果该命令引发一个`CommandError`，它将被截断并打印到标准错误输出。

> 在你的代码中调用管理命令
>
> 不应该在你的代码中直接调用`execute()`来执行一个命令。请使用`call_command`。

`BaseCommand.handle(*args, **options)`

命令的真正逻辑。子类必须实现这个方法。

`BaseCommand.check(app_configs=None, tags=None, display_num_errors=False)`

```
New in Django 1.7.
```

利用系统的检测框架检测全部Django项目的潜在问题。严重的问题将引发`CommandError`；警告会输出到标准错误输出；次要的通知会输出到标准输出。

如果`app_configs`和`tags`都为`None`，将进行所有的系统检查。`tags`可以是一个要检查的标签列表，比如`compatibility`或`models`。

`BaseCommand.validate(app=None, display_num_errors=False)`

```
Deprecated since version 1.7:

被check命令代替
```

如果`app`为`None`，那么将检查安装的所有应用的错误。

### BaseCommand 的子类 ###

`class AppCommand`

这个管理命令接收一个或多个安装的应用标签作为参数，并对它们每一个都做一些动作。

子类不用实现`handle()`，但必须实现`handle_app_config()`，它将会为每个应用调用一次。

`AppCommand.handle_app_config(app_config, **options)`

对`app_config`完成命令行的动作，其中`app_config`是`AppConfig`的实例，对应于在命令行上给出的应用标签。

> Changed in Django 1.7:
>
> 以前，AppCommand子类必须实现`handle_app(app, **options)`，其中`app`是一个模型模块。新的API可以不需要模型模块来处理应用。迁移的最快的方法如下：
>
```
def handle_app_config(app_config, **options):
    if app_config.models_module is None:
        return                                  # Or raise an exception.
    app = app_config.models_module
    # Copy the implementation of handle_app(app_config, **options) here.
```
>
> 然而，你可以通过直接使用`app_config`的属性来简化实现。

`class LabelCommand`

这个管理命令接收命令行上的一个或多个参数（标签），并对它们每一个都做一些动作。

子类不用实现`handle()`，但必须实现`handle_label()`，它将会为每个标签调用一次。

`LabelCommand.handle_label(label, **options)`

对`label`完成命令行的动作，`label`是命令行给出的字符串。

`class NoArgsCommand`

```
Deprecated since version 1.8:

使用BaseCommand代替，它默认也不需要参数。
```

这个命令不接收命令行上的参数。

子类不需要实现`handle()`，但必须实现`handle_noargs()`；`handle()`本身已经被覆盖以保证不会有参数传递给命令。

`NoArgsCommand.handle_noargs(**options)`

完成这个命令的动作

### Command 的异常 ###

`class CommandError`

异常类，表示执行一个管理命令时出现问题。

如果这个异常是在执行一个来自命令行控制台的管理命令时引发，它将被捕获并转换成一个友好的错误信息到合适的输出流（例如，标准错误输出）；因此，引发这个异常（并带有一个合理的错误描述）是首选的方式来指示在执行一个命令时某些东西出现错误。

如果管理命令从代码中通过call_command调用，那么需要时捕获这个异常由你决定。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Adding custom commands](https://docs.djangoproject.com/en/1.8/howto/custom-management-commands/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
