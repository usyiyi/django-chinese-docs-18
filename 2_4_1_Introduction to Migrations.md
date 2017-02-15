

# 迁移

New in Django 1.7.

迁移是Django用于同步你的发生改变的模型(添加一个字段，删除一个模型，等等。) 到你的数据库。 它的设计是很智能的, 但是你还是需要了解什么时候进行迁移, 什么时候去启动它们, 以及可能遇到的指令问题。

## 简短历史

在 1.7版本之前, Django 只支持添加新模型到数据库；无法通过`syncdb`命令来修改或移除已存在的模型 (已被[`migrate`](../ref/django-admin.html#django-admin-migrate)代替)。

第三方工具，最著名的是[South](http://south.aeracode.org), 为这些额外的功能提供支持,但是它还是被认为是很重要的部分并且加入到django的核心里面。

## 命令集

有一些你用于进行迁移的命令集和数据库架构的django操作

*   [`migrate`](../ref/django-admin.html#django-admin-migrate), 负责执行迁移, 以及撤销和列出迁移的状态。
*   [`makemigrations`](../ref/django-admin.html#django-admin-makemigrations), 负责基于你的模型修改创建一个新的迁移
*   [`sqlmigrate`](../ref/django-admin.html#django-admin-sqlmigrate), 展示迁移的sql语句

值得注意的是，迁移是建立和运行在每一个app的基础上。 特别的,确实存在 _没有用迁移的app_ (它们被称为“非迁移”apps) -相比利用遗留的保存行为，它们只是在这个基础上添加一个新的模型.

你可以想象 migrations相当一个你的数据库的一个版本控制系统。`makemigrations` 命令负责保存你的模型变化到一个迁移文件 - 和 commits很类似 - 同时 `migrate`负责将改变提交到数据库。

每个app 的迁移文件会保存到每个相应app的“migrations”文件夹里面,并且准备如何去执行它, 作为一个分布式代码库。 每当在你的开发机器或是你同事的机器并且最终在你的生产机器上运行同样的迁移，你应当再创建这些文件。

注意

通过修改 [`MIGRATION_MODULES`](../ref/settings.html#std:setting-MIGRATION_MODULES) 设置，可以覆盖那些每个app 都包含  migrations 的 package 

同样的方式，同样的数据集，将产生一致的结果, 那意味着你在开发和筹划中在按照同样的原理运行的情况下得到的结果和线上是一致的

Django 将迁移你对模型和字段做出的任何改变 - 甚至包括那些对数据库没有影响的操作， -在历史记录存放所有改变是正确重建field的唯一方式，你可能会在以后的数据迁移中使用到这些选项 （比如，在你定制了校验器的时候）。

## 后台支持

Django附带的所有后端都支持迁移，以及任何第三方后端，如果他们已编程支持模式更改（通过[_SchemaEditor_](../ref/schema-editor.html)类完成）。

但是，一些数据库比其他数据库在模式迁移方面更有能力；下面覆盖的一些警告.

### PostgreSQL

PostgreSQL是所有数据库中最能够支持的模式；唯一的警告是添加具有默认值的列将导致表的完全重写，其时间与其大小成比例。

因此，建议您始终使用`null=True`创建新列，因为这样会立即添加。

### MySQL

MySQL缺少对模式更改操作的事务的支持，这意味着如果迁移无法应用，则必须手动取消批准更改才能再次尝试（不可能回滚到更早的点）。

此外，MySQL将为每个模式操作完全重写表，通常需要与表中的行数成正比的时间来添加或删除列。在较慢的硬件上，这可能比每分钟每百万行更糟 - 向表中添加几列只有几百万行可能会锁定您的网站超过十分钟。

最后，MySQL对列，表和索引的名称长度有相当小的限制，以及索引涵盖的所有列的组合大小的限制。这意味着在其他后端可能的索引将无法在MySQL下创建。

### SQLite

SQLite具有非常少的内置模式更改支持，因此Django尝试通过以下方式模拟它：

*   使用新模式创建新表
*   复制数据
*   删除旧表
*   重命名新表以匹配原始名称

这个过程一般效果很好，但它可能很慢，偶尔会出现问题。不建议您在生产环境中运行和迁移SQLite，除非您非常了解风险及其限制；支持Django的设计允许开发人员在其本地机器上使用SQLite开发不太复杂的Django项目，而不需要一个完整的数据库。

## 工作流程

迁移工作很简单。 修改你的模型 -比如添加字段和移除一个模型 - 然后运行 [`makemigrations`](../ref/django-admin.html#django-admin-makemigrations):

```
$ python manage.py makemigrations
Migrations for 'books':
  0003_auto.py:
    - Alter field author on book

```

你的原型会扫描和比较你当前迁移文件里面的版本,同时新的迁移文件会被创建. 请务必查阅输出，看看`makemigrations` 是如何理解你做出的更改 - 迁移不是完全准确的, 对于一些复杂的更改，它可能不会检测到你所期望的东西。

一旦你有了新的迁移文件， 你应该把它们提交到你的数据库以确保它们像预期的那样工作：

```
$ python manage.py migrate
Operations to perform:
  Synchronize unmigrated apps: sessions, admin, messages, auth, staticfiles, contenttypes
  Apply all migrations: books
Synchronizing apps without migrations:
  Creating tables...
  Installing custom SQL...
  Installing indexes...
Installed 0 object(s) from 0 fixture(s)
Running migrations:
  Applying books.0003_auto... OK

```

命令分两步运行，首先，它会同步未迁移的应用（与旧的`syncdb`命令所做的操作相同），然后对未应用的更改进行迁移。

一旦迁移应用后，在版本控制系统中将迁移与模型的改变作为同一次提交-那样的话，其他开发者（或者你的生产服务器）检查代码时，他们将同时看到模型的改变及伴随的迁移。

New in Django 1.8.

If you want to give the migration(s) a meaningful name instead of a generated one, you can use the [`--name`](../ref/django-admin.html#django-admin-option---name) option:使用--name 选项自定义migrations迁移文件名称。

```
$ python manage.py makemigrations --name changed_my_model your_app_label

```

### 版本控制

由于迁移存储在版本控制中，因此您偶尔会遇到这样的情况：您和另一个开发人员同时提交了迁移到同一个应用程序，导致两个迁移具有相同的编号。

不要担心 - 数字只是为了开发人员的参考，Django只关心每个迁移有不同的名称。迁移指定文件中依赖的其他迁移（包括在同一应用中的早期迁移），因此可以检测同一应用的两次新迁移是否未订购。

当发生这种情况时，Django会提示你并给你一些选项。如果它认为它足够安全，它会提供自动线性化两个迁移为你。如果没有，您必须自行修改迁移作业，别担心，这并不难，在下面的[_Migration files_](#migration-files)中会有更多说明。

## 依赖关系

虽然迁移是基于应用程序的，但是模型所包含的表和关系太复杂，无法一次为一个应用程序创建。当您进行需要其他方式运行的迁移时，例如，您可以在`books`应用程序中将`ForeignKey`添加到您的`authors`导致的迁移将包含对`authors`中的迁移的依赖。

这意味着当您运行迁移时，`authors`迁移首先运行，创建`ForeignKey`引用的表，然后进行`ForeignKey`如果没有发生这种情况，迁移将尝试创建`ForeignKey`列，而不引用现有的表，并且您的数据库会抛出错误。

此依赖关系行为会影响大多数将操作限制为单个应用程序的迁移操作。限制到单个应用程序（在`makemigrations`或`migrate`）是一个尽力而为的承诺，而不是保证；任何其他需要用来获取依赖关系的应用程序都会正确。

不过请注意，由于没有迁移，未迁移的应用程式无法依赖已迁移的应用程式。这意味着通常不可能有一个未迁移的应用程序具有`ForeignKey`或`ManyToManyField`到已迁移的应用程序；一些情况可能会工作，但它最终会失败。

警告

即使事情似乎与未迁移的应用程序一起工作，这取决于已迁移的应用程序，Django可能不会生成所有必需的外键约束！

如果您使用可交换模型（例如`AUTH_USER_MODEL`），这一点尤其明显，因为使用可交换模型的每个应用都需要进行迁移（如果您不幸）。随着时间的推移，越来越多的第三方应用将获得迁移，但在此期间，您可以自行迁移（如果您愿意，可以使用[`MIGRATION_MODULES`](../ref/settings.html#std:setting-MIGRATION_MODULES)将这些模块存储在应用自己的模块之外） ），或保持应用程序与您的用户模型未迁移。

## 迁移文件

迁移存储为磁盘格式，此处称为“迁移文件”。这些文件实际上只是正常的Python文件，具有约定的对象布局，以声明样式编写。

基本迁移文件如下所示：

```
from django.db import migrations, models

class Migration(migrations.Migration):

    dependencies = [("migrations", "0001_initial")]

    operations = [
        migrations.DeleteModel("Tribble"),
        migrations.AddField("Author", "rating", models.IntegerField(default=0)),
    ]

```

加载迁移文件（作为Python模块）时，Django查找的是`django.db.migrations.Migration`的子类，名为`Migration`。然后它检查这个对象的四个属性，大多数时间只使用其中的两个：

*   `dependencies`，此依赖关系的迁移列表。
*   `operations`，定义此迁移操作的`Operation`类的列表。

operations 是关键，它们是一个陈述性说明的集合，能告诉Django需要产生什么样的schema 更改。Django扫描它们并构建所有应用程序的所有模式更改的内存中表示，并使用它来生成使模式更改的SQL。

内存结构也用于计算模型和迁移的当前状态之间的差异；Django运行所有的更改，按顺序，在内存中的模型集，以找出你的模型的上次你运行`makemigrations`的状态。然后使用这些模型与您的`models.py`文件中的模型进行比较，以确定您所做的更改。

您应该很少（如果曾经）需要手动编辑迁移文件，但是如果需要，完全可以手动编写它们。一些更复杂的操作不是自动检测的，只能通过手写迁移提供，所以如果你必须编辑它们，不要害怕。

### 自定义字段

您无法修改已迁移的自定义字段中的位置参数的数量，而不提高`TypeError`。旧迁移将调用具有旧签名的修改的`__init__`方法。So if you need a new argument, please create a keyword argument and add something like `assert 'argument_name' in kwargs` in the constructor.

### 模型管理者

New in Django 1.8.

您可以选择将管理器序列化为迁移，并使它们在[`RunPython`](../ref/migration-operations.html#django.db.migrations.operations.RunPython "django.db.migrations.operations.RunPython")操作中可用。这是通过在管理器类上定义`use_in_migrations`属性来实现的：

```
class MyManager(models.Manager):
    use_in_migrations = True

class MyModel(models.Model):
    objects = MyManager()

```

如果您使用[`from_queryset()`](db/managers.html#django.db.models.from_queryset "django.db.models.from_queryset")函数动态生成管理器类，则需要从生成的类继承以使其可导入：

```
class MyManager(MyBaseManager.from_queryset(CustomQuerySet)):
    use_in_migrations = True

class MyModel(models.Model):
    objects = MyManager()

```

请参阅迁移中有关[_历史模型_](#historical-models)的注意事项，了解其中的影响。

## 将迁移添加到应用

将迁移添加到新应用程序很简单 - 它们已预先配置为接受迁移，因此，一旦进行了某些更改，只需运行[`makemigrations`](../ref/django-admin.html#django-admin-makemigrations)即可。

如果您的应用程序已经有模型和数据库表，但还没有迁移（例如，您是根据以前的Django版本创建的），则需要将其转换为使用迁移；这是一个简单的过程：

```
$ python manage.py makemigrations your_app_label

```

这将为您的应用程序进行新的初始迁移。现在，执行`python manage.py migrate - fake-initial Django将检测到您有一个初始迁移_和_，它想要创建的表已经存在，并将标记为已应用迁移。`（没有[`--fake-initial`](../ref/django-admin.html#django-admin-option---fake-initial)标志，[`migrate`](../ref/django-admin.html#django-admin-migrate)命令将错误输出，因为它要创建的表已经存在。

注意，这只工作给予两件事：

*   你没有改变你的模型，因为你制作了他们的表。要使迁移正常工作，必须先进行初始迁移，然后进行更改，因为Django会将更改与迁移文件进行比较，而不是数据库。
*   您没有手动编辑数据库 - Django将无法检测到您的数据库与您的模型不匹配，您只会在迁移尝试修改这些表时收到错误。

## 历史模型

当您运行迁移时，Django使用存储在迁移文件中的模型的历史版本。如果您使用[`RunPython`](../ref/migration-operations.html#django.db.migrations.operations.RunPython "django.db.migrations.operations.RunPython")操作编写Python代码，或者如果在数据库路由器上使用`allow_migrate`方法，则会暴露给这些版本的模型。

因为不可能序列化任意Python代码，所以这些历史模型不会有你定义的任何自定义方法。但是，他们将具有相同的字段，关系，管理员（仅限于`use_in_migrations = True`）和`Meta`选项（也已版本化，因此它们可能与您当前的不同）。

警告

这意味着，当您在迁移中访问对象时，您不会有对对象调用的自定义`save()`方法，并且您不会有任何自定义构造函数或实例方法。计划适当！

对诸如`upload_to`和`limit_choices_to`等字段选项中的函数的引用以及具有`use_in_migrations = t6 &gt; True`在迁移中序列化，因此只要存在引用它们的迁移，函数和类就需要保留。还需要保留任何[_custom model fields_](../howto/custom-model-fields.html)，因为这些是通过迁移直接导入的。

此外，模型的基类只存储为指针，因此只要存在包含对它们的引用的迁移，就必须始终保持基类。在正面，这些基类的方法和管理器通常继承，所以如果你绝对需要访问这些，你可以选择将它们移动到超类。

## 删除模型字段时的注意事项

New in Django 1.8.

与上一节中介绍的“对历史函数的引用”注意事项类似，如果在旧迁移中引用了自定义模型字段，则从项目或第三方应用中移除自定义模型字段将会导致问题。

为了帮助解决这种情况，Django提供了一些模型字段属性，以帮助使用[_system checks framework_](checks.html)来取消模型字段。

将`system_check_deprecated_details`属性添加到模型字段，类似于以下内容：

```
class IPAddressField(Field):
    system_check_deprecated_details = {
        'msg': (
            'IPAddressField has been deprecated. Support for it (except '
            'in historical migrations) will be removed in Django 1.9.'
        ),
        'hint': 'Use GenericIPAddressField instead.',  # optional
        'id': 'fields.W900',  # pick a unique ID for your field.
    }

```

在您选择的弃用期之后（Django本身的字段的两个主要版本），将`system_check_deprecated_details`属性更改为`system_check_removed_details`并更新字典类似于：

```
class IPAddressField(Field):
    system_check_removed_details = {
        'msg': (
            'IPAddressField has been removed except for support in '
            'historical migrations.'
        ),
        'hint': 'Use GenericIPAddressField instead.',
        'id': 'fields.E900',  # pick a unique ID for your field.
    }

```

您应该保留字段的数据库迁移操作所需的方法，例如`__init__()`，`deconstruct()`和`get_internal_type()`只要引用此字段的任何迁移存在，就保留此存根字段。例如，在压缩迁移并删除旧迁移后，您应该可以完全删除该字段。

## 数据迁移

除了更改数据库模式之外，您还可以使用迁移来更改数据库中的数据，如果需要，还可以与模式一起使用。

更改数据的迁移通常称为“数据迁移”；他们最好写成单独的迁移，坐在模式迁移。

Django不能为你自动生成数据迁移，就像它对模式迁移一样，但是写它们并不困难。Django中的迁移文件由[_Operations_](../ref/migration-operations.html)组成，用于数据迁移的主要操作是[`RunPython`](../ref/migration-operations.html#django.db.migrations.operations.RunPython "django.db.migrations.operations.RunPython")。

首先，让一个空的迁移文件，你可以工作（Django会把文件放在正确的地方，建议一个名字，并为你添加依赖关系）：

```
python manage.py makemigrations --empty yourappname

```

接下来, 我们打开刚刚生成的迁移文件; 你应该可以看到类似以下的内容:

```
# -*- coding: utf-8 -*-
from django.db import models, migrations

class Migration(migrations.Migration):

    dependencies = [
        ('yourappname', '0001_initial'),
    ]

    operations = [
    ]

```

现在，所有你需要做的是创建一个新函数，并使用[`RunPython`](../ref/migration-operations.html#django.db.migrations.operations.RunPython "django.db.migrations.operations.RunPython")。[`RunPython`](../ref/migration-operations.html#django.db.migrations.operations.RunPython "django.db.migrations.operations.RunPython")需要一个可调用作为其参数，它需要两个参数 - 第一个是一个[_app registry_](../ref/applications.html)，它将所有模型的历史版本加载到其中以匹配历史记录迁移就位，第二个是[_SchemaEditor_](../ref/schema-editor.html)，您可以使用它手动实现数据库模式更改（但要小心，这样做可能会使迁移自动检测器混乱）。

让我们编写一个简单的迁移，用`first_name`和`last_name`的组合值填充新的`name`字段（我们来到了我们的感觉不是每个人都有名和姓）。所有我们需要做的是使用历史模型并在行上进行迭代：

```
# -*- coding: utf-8 -*-
from django.db import models, migrations

def combine_names(apps, schema_editor):
    # We can't import the Person model directly as it may be a newer
    # version than this migration expects. We use the historical version.
    Person = apps.get_model("yourappname", "Person")
    for person in Person.objects.all():
        person.name = "%s  %s" % (person.first_name, person.last_name)
        person.save()

class Migration(migrations.Migration):

    dependencies = [
        ('yourappname', '0001_initial'),
    ]

    operations = [
        migrations.RunPython(combine_names),
    ]

```

完成后，我们可以正常运行`python manage.py migrate`与其他迁移一起放置。

您可以传递第二个可调用项到[`RunPython`](../ref/migration-operations.html#django.db.migrations.operations.RunPython "django.db.migrations.operations.RunPython")，以便在向后迁移时运行您希望执行的任何逻辑。如果省略此可调用项，向后迁移将引发异常。

### 通过其他应用访问模型

When writing a `RunPython` function that uses models from apps other than the one in which the migration is located, the migration’s `dependencies` attribute should include the latest migration of each app that is involved, otherwise you may get an error similar to: `LookupError: No installed app with label 'myappname'` when you try to retrieve the model in the `RunPython` function using `apps.get_model()`.

在下面的示例中，我们在`app1`中有一个迁移，需要使用`app2`中的模型。我们不关心`move_m1`的细节，而是需要从两个应用程序访问模型的事实。因此，我们添加了一个依赖关系，指定`app2`的最后一次迁移：

```
class Migration(migrations.Migration):

    dependencies = [
        ('app1', '0001_initial'),
        # added dependency to enable using models from app2 in move_m1
        ('app2', '0004_foobar'),
    ]

    operations = [
        migrations.RunPython(move_m1),
    ]

```

### 更多高级迁移

如果你对更高级的迁移操作感兴趣, 或者想写自定义迁移文件, see the [_migration operations reference_](../ref/migration-operations.html) and the “how-to” on [_writing migrations_](../howto/writing-migrations.html).

## 挤压迁移

鼓励你自由迁徙，不要担心你有多少；迁移代码被优化以一次处理几百个而没有大的减速。然而，最终你会想从几百个迁移回到几个，这就是挤压的地方。

压缩是将现有的许多迁移集减少到仍然表示相同更改的一个（或有时几个）迁移的行为。

Django通过采取所有现有迁移，提取其`Operation`并将它们全部顺序，然后在它们上运行优化器来尝试并减少列表的长度，这样做 - 例如，它知道[`CreateModel`](../ref/migration-operations.html#django.db.migrations.operations.CreateModel "django.db.migrations.operations.CreateModel")和[`DeleteModel`](../ref/migration-operations.html#django.db.migrations.operations.DeleteModel "django.db.migrations.operations.DeleteModel")互相取消，并且知道[`AddField`](../ref/migration-operations.html#django.db.migrations.operations.AddField "django.db.migrations.operations.AddField")可以滚动到[`CreateModel`](../ref/migration-operations.html#django.db.migrations.operations.CreateModel "django.db.migrations.operations.CreateModel")。

一旦操作序列被尽可能地减少 - 可能的数量取决于模型是多么密切，如果你有任何[`RunSQL`](../ref/migration-operations.html#django.db.migrations.operations.RunSQL "django.db.migrations.operations.RunSQL")或[`RunPython`](../ref/migration-operations.html#django.db.migrations.operations.RunPython "django.db.migrations.operations.RunPython")操作（它可以'通过优化） - Django然后将其写回一组新的初始迁移文件。

这些文件被标记为说它们替换以前被压缩的迁移，因此它们可以与旧的迁移文件共存，并且Django将根据您在历史记录中的位置智能地在它们之间切换。如果您仍然部分地通过您压缩的迁移集，它将继续使用它们，直到它到达结束，然后切换到压缩的历史记录，而新的安装将只使用新的压缩迁移，并跳过所有的旧那些。

这使您可以压缩，而不是混乱当前生产中尚未完全更新的系统。建议的过程是压缩，保留旧文件，提交和释放，等待所有系统升级到新版本（或如果您是第三方项目，只是确保您的用户升级版本，而不跳过任何） ，然后删除旧的文件，提交并做第二个版本。

支持所有这一切的命令是[`squashmigrations`](../ref/django-admin.html#django-admin-squashmigrations) - 只是将它的应用程序标签和迁移名称，你想要压缩，它会工作：

```
$ ./manage.py squashmigrations myapp 0004
Will squash the following migrations:
 - 0001_initial
 - 0002_some_change
 - 0003_another_change
 - 0004_undo_something
Do you wish to proceed? [yN] y
Optimizing...
  Optimized from 12 operations to 7 operations.
Created new squashed migration /home/andrew/Programs/DjangoTest/test/migrations/0001_squashed_0004_undo_somthing.py
  You should commit this migration but leave the old ones in place;
  the new migration will be used for new installs. Once you are sure
  all instances of the codebase have applied the migrations you squashed,
  you can delete them.

```

注意，Django中的模型相互依赖性可能变得非常复杂，压缩可能导致不运行的迁移；（在这种情况下，您可以使用`--no-optimize`再试一次，虽然您也应该报告问题），或者使用`CircularDependencyError`您可以手动解决它。

要手动解析`CircularDependencyError`，请将循环依赖关系循环中的一个ForeignKeys拆分为单独的迁移，并使用它移动其他应用程序的依赖关系。如果您不确定，请问当您从模型中创建全新的迁移时，makemigrations如何处理这个问题。在将来的Django版本中，将会更新squashmigrations以尝试自己解决这些错误。

一旦您压缩了迁移，您应该将其替换为它替换的迁移，并将此更改分发到应用程序的所有正在运行的实例，确保它们运行`migrate`以将更改存储在其数据库中。

完成此操作后，您必须通过以下方式将压缩的迁移转换为正常的初始迁移：

*   删除它替换的所有迁移文件
*   删除缩小迁移的`Migration`类中的`replaces`参数（这是Django如何解释它是一个被压缩的迁移）

注意

一旦您压制了迁移，您就不应该重新压缩被压缩的迁移，直到您完全转换为正常迁移为止。

## 序列化值

迁移只是包含模型的旧定义的Python文件 - 因此，为了编写它们，Django必须获取模型的当前状态并将它们序列化到一个文件中。

虽然Django可以序列化大多数东西，但有一些事情，我们不能序列化为有效的Python表示 - 没有Python标准如何将值转换回代码（`repr()`仅适用于基本值，并且不指定导入路径）。

Django可以序列化以下内容：

*   `int`, `long`, `float`, `bool`, `str`, `unicode`, `bytes`, `None`
*   `list`，`set`，`tuple`，`dict`
*   `datetime.date`，`datetime.time`和`datetime.datetime`实例（包括时区感知的实例）
*   `decimal.`小数实例
*   任何Django字段
*   任何函数或方法引用（例如`datetime.datetime.today`）（必须在模块的顶级作用域中）
*   任何类引用（必须在模块的顶级作用域中）
*   使用自定义`deconstruct()`方法（[_see below_](#custom-deconstruct-method)）的任何内容

Changed in Django 1.7.1:

支持串行化时区感知的数据时间。

Django只能在Python 3上序列化以下内容：

*   在类体内使用的未绑定方法（见下文）

Django不能序列化：

*   嵌套类
*   任意类实例（例如`MyClass（4.3， 5.7）`）
*   Lambdas

由于`__qualname__`仅在Python 3中引入，Django只能序列化Python 3上的以下模式（在类体中使用的未绑定方法），并且将无法序列化对它的引用Python 2：

```
class MyModel(models.Model):

    def upload_to(self):
        return "something dynamic"

    my_file = models.FileField(upload_to=upload_to)

```

如果您使用的是Python 2，我们建议您移动upload_to的方法以及接受可调用项（例如`default`）的类似参数，使其位于主模块正文中，而不是类主体中。

### 添加deconstruct()方法

您可以让Django通过为类提供`deconstruct()`方法来序列化您自己的自定义类实例。它不需要参数，应该返回一个三元组的元组`（path， args， kwargs）`

*   `path`应该是类的Python路径，类名包含在最后一部分中（例如`myapp.custom_things.`我的课）。如果你的类在模块的顶层不可用，那么它是不可序列化的。
*   `args`应该是要传递给类'`__init__`方法的位置参数列表。这个列表中的所有内容都应该是可序列化的。
*   `kwargs`应该是传递到类'`__init__`方法的关键字参数的dict。每个值本身都是可序列化的。

注意

此返回值不同于自定义字段的`deconstruct()`方法[_for custom fields_](../howto/custom-model-fields.html#custom-field-deconstruct-method)

Django将写出这个值作为你的类的实例化与给定的参数，类似于它写出对Django字段的引用的方式。

为了防止每次运行[`makemigrations`](../ref/django-admin.html#django-admin-makemigrations)时创建新的迁移，您还应该向装饰类添加一个`__eq__()`方法。这个函数将被Django的迁移框架调用来检测状态之间的变化。

只要你的类的构造函数的所有参数都是可序列化的，你可以使用`django.utils.deconstruct`中的`@deconstructible`类装饰器来添加`deconstruct()`方法：

```
from django.utils.deconstruct import deconstructible

@deconstructible
class MyCustomClass(object):

    def __init__(self, foo=1):
        self.foo = foo
        ...

    def __eq__(self, other):
        return self.foo == other.foo

```

装饰器添加逻辑来捕获和保存参数到它们的构造函数中，然后在调用deconstruct()时返回这些参数。

## Python 2 和 3 支持

为了生成支持Python 2和3的迁移，模型和字段中使用的所有字符串文字（例如`verbose_name`，`related_name`等）必须始终为在Python 2和3（而不是Python 2中的字节和Python 3中的文本，未标记的字符串文字的默认情况）下的字符串或文本（unicode）字符串。否则在Python 3下运行[`makemigrations`](../ref/django-admin.html#django-admin-makemigrations)会生成虚假的新迁移，将所有这些字符串属性转换为文本。

实现这个的最简单的方法是遵循Django的[_Python 3 porting guide_](python3.html)中的建议，并确保所有的模块以`开头 __ future __ t4&gt; import unicode_literals`，以便所有未标记的字符串文字总是unicode，而不管Python版本。当您将它添加到在Python 2上生成的现有迁移的应用程序时，下一次在Python 3上运行[`makemigrations`](../ref/django-admin.html#django-admin-makemigrations)可能会生成许多更改，因为它将所有的bytestring属性转换为文本字符串；这是正常的，应该只发生一次。

## 多Django版本支持

如果您是使用模型的第三方应用的维护者，则可能需要提供支持多个Django版本的迁移。在这种情况下，您应始终使用希望支持的Django版本运行[`makemigrations`](../ref/django-admin.html#django-admin-makemigrations) **。**

迁移系统将根据与其他Django相同的策略保持向后兼容性，因此在Django X.Y上生成的迁移文件应该在Django X.Y + 1上保持不变。然而，迁移系统不承诺向前兼容性。可能会添加新功能，并且使用较新版本的Django生成的迁移文件可能无法在旧版本上运行。

## 从 South 升级

如果你有使用 [South](http://south.aeracode.org)创建的迁移, 使用`django.db.migrations`升级的过程非常简单：

*   确认所有的安装都以South创建的迁移完全更新。
*   从[`INSTALLED_APPS`](../ref/settings.html#std:setting-INSTALLED_APPS)中移除 `'south'`
*   删除你所有的迁移文件（编号标识的），但不能删除所在目录或 `__init__.py` - 也需确认删除 `.pyc` 文件。
*   运行 `python manage.py makemigrations`. Django应该看到空的迁移目录，并以新的格式进行新的初始迁移。
*   执行`python manage.py 迁移 - fake-initial`。Django会看到初始迁移的表已经存在，并将它们标记为已应用，而不运行它们。（Django不会检查表模式是否与您的模型匹配，只是正确的表名存在）。

而已！唯一的复杂因素是如果你有一个循环依赖循环的外键；在这种情况下，`makemigrations`可能会进行多个初始迁移，您需要将它们全部标记为已应用：

```
python manage.py migrate --fake yourappnamehere

```

Changed in Django 1.8:

[`--fake-initial`](../ref/django-admin.html#django-admin-option---fake-initial)标志已添加到[`migrate`](../ref/django-admin.html#django-admin-migrate)；以前，如果检测到现有表，初始迁移总是自动伪造应用。

### 库/第三方应用

如果你是一个库或应用程序维护者，并且希望支持南迁移（对于Django 1.6及以下版本）和Django迁移（对于1.7及以上版本），应在应用程序中保留两个并行迁移集，每个格式一个。

为了帮助这一点，South 1.0将首先在`south_migrations`目录中自动查找South格式迁移，然后查看`migrations`，这意味着用户的项目将透明地使用正确的只要将您的南迁移位于`south_migrations`目录中，并将您的Django迁移位于`migrations`目录中即可。

更多信息请参考 [South 1.0 发布说明](http://south.readthedocs.org/en/latest/releasenotes/1.0.html#library-migration-path).

参考

[_The Migrations Operations Reference_](../ref/migration-operations.html)

Covers the schema operations API, special operations, and writing your own operations.

[_The Writing Migrations “how-to”_](../howto/writing-migrations.html)

Explains how to structure and write database migrations for different scenarios you might encounter.

