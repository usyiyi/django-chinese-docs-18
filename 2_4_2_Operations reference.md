

# 迁移操作

迁移文件由一个或多个`Operation`组成，这些对象声明性地记录迁移应对数据库执行的操作。

Django还使用这些`Operation`对象来计算您的模型在历史上的样子，并计算自上次迁移后对模型所做的更改，以便自动编写迁移；这就是为什么他们是声明性的，因为它意味着Django可以轻松地将它们全部加载到内存中，并通过它们运行，而不用触及数据库，以确定您的项目应该是什么样子。

还有更专门的`Operation`对象，用于像[_data migrations_](../topics/migrations.html#data-migrations)和高级手动数据库操作。如果要封装常用的自定义更改，也可以编写自己的`Operation`类。

如果您需要一个空的迁移文件来写自己的`Operation`对象，只需使用`python manage.py makemigrations - 空的 yourappname`，但请注意，手动添加模式更改操作可能会混淆迁移自动检测器并导致运行[`makemigrations`](django-admin.html#django-admin-makemigrations)输出不正确的代码。

所有的核心Django操作都可以从`django.db.migrations.operations`模块获得。

有关介绍材料，请参阅[_migrations topic guide_](../topics/migrations.html)。

## 模式操作

### CreateModel

_class_ `CreateModel`(_name_, _fields_, _options=None_, _bases=None_, _managers=None_)

在项目历史中创建一个新模型，并在数据库中创建一个相应的表来匹配它。

`name`是型号名称，如写在`models.py`文件中。

`fields`是`（field_name， field_instance）`的2元组的列表。字段实例应为未绑定字段（因此只有`models.`CharField()，而不是一个字段从另一个模型）。

`options`是来自模型的`Meta`类的值的可选字典。

`bases`是此模型继承的其他类的可选列表；它可以包含类对象以及格式为`"appname.ModelName"`的字符串，如果你想依赖另一个模型（所以你从历史版本继承）。如果没有提供，它默认只继承标准的`models.Model`。

`managers`获取`（manager_name， manager_instance）`的2元组列表。列表中的第一个管理器将成为迁移期间此模型的默认管理器。

Changed in Django 1.8:

已添加`managers`参数。

### 删除模型

_class_ `DeleteModel`(_name_)

从数据库中删除项目历史中的模型及其表。

### 重命名模型

_class_ `RenameModel`(_old_name_, _new_name_)

将模型从旧名称重命名为新名称。

如果您立即更改模型的名称及其几个字段，则可能必须手动添加此字段；到自动检测器，这看起来像您删除了一个具有旧名称的模型，并添加了一个具有不同名称的模型，并且其创建的迁移将丢失旧表中的任何数据。

### AlterModelTable

_class_ `AlterModelTable`(_name_, _table_)

更改模型的表名（`Meta`子类上的[`db_table`](models/options.html#django.db.models.Options.db_table "django.db.models.Options.db_table")选项）。

### AlterUniqueTogether

_class_ `AlterUniqueTogether`(_name_, _unique_together_)

更改模型的唯一约束集（`Meta`子类上的[`unique_together`](models/options.html#django.db.models.Options.unique_together "django.db.models.Options.unique_together")选项）。

### AlterIndexTogether

_class_ `AlterIndexTogether`(_name_, _index_together_)

更改模型的自定义索引集（`Meta`子类上的[`index_together`](models/options.html#django.db.models.Options.index_together "django.db.models.Options.index_together")选项）。

### AlterOrderWithRespectTo

_class_ `AlterOrderWithRespectTo`(_name_, _order_with_respect_to_)

在`Meta`子类上为[`order_with_respect_to`](models/options.html#django.db.models.Options.order_with_respect_to "django.db.models.Options.order_with_respect_to")选项创建或删除所需的`_order`列。

### AlterModelOptions

_class_ `AlterModelOptions`(_name_, _options_)

存储对`permissions`和`verbose_name`的杂项模型选项（模型`Meta`不会影响数据库，但会对[`RunPython`](#django.db.migrations.operations.RunPython "django.db.migrations.operations.RunPython")实例保留这些更改以使用。`options`应是将选项名称映射到值的字典。

### AlterModelManagers

New in Django 1.8.

_class_ `AlterModelManagers`(_name_, _managers_)

更改迁移期间可用的管理器。

### AddField

_class_ `AddField`(_model_name_, _name_, _field_, _preserve_default=True_)

向模型中添加字段。`model_name`是模型的名称，`name`是字段的名称，`field`是未绑定的Field实例在`models.py`中 - 例如，`models.`IntegerField（null = True）。

The `preserve_default` argument indicates whether the field’s default value is permanent and should be baked into the project state (`True`), or if it is temporary and just for this migration (`False`) - usually because the migration is adding a non-nullable field to a table and needs a default value to put into existing rows. 它不影响在数据库中直接设置默认值的行为 - Django从不设置数据库默认值，并始终将它们应用于Django ORM代码。

### RemoveField

_class_ `RemoveField`(_model_name_, _name_)

从模型中删除字段。

记住，当颠倒这实际上是添加一个字段到模型；如果字段不可为空，这可能使此操作不可逆（除了任何数据丢失，这当然是不可逆的）。

### AlterField

_class_ `AlterField`(_model_name_, _name_, _field_, _preserve_default=True_)

更改字段的定义，包括对其类型，[`null`](models/fields.html#django.db.models.Field.null "django.db.models.Field.null")，[`unique`](models/fields.html#django.db.models.Field.unique "django.db.models.Field.unique")，[`db_column`](models/fields.html#django.db.models.Field.db_column "django.db.models.Field.db_column")和其他字段属性的更改。

The `preserve_default` argument indicates whether the field’s default value is permanent and should be baked into the project state (`True`), or if it is temporary and just for this migration (`False`) - usually because the migration is altering a nullable field to a non-nullable one and needs a default value to put into existing rows. 它不影响在数据库中直接设置默认值的行为 - Django从不设置数据库默认值，并始终将它们应用于Django ORM代码。

请注意，并非所有数据库都可以进行所有更改 - 例如，您无法更改类似`models.`TextField()转换为数字类型字段，如`models.`大多数数据库上的IntegerField()。

Changed in Django 1.7.1:

已添加`preserve_default`参数。

### RenameField

_class_ `RenameField`(_model_name_, _old_name_, _new_name_)

更改字段名称（除非设置[`db_column`](models/fields.html#django.db.models.Field.db_column "django.db.models.Field.db_column")，否则其列名称）。

## 特别行动

### RunSQL

_class_ `RunSQL`(_sql_, _reverse_sql=None_, _state_operations=None_, _hints=None_)

允许在数据库上运行任意SQL - 对于Django不能直接支持的数据库后端的更高级功能（如部分索引）非常有用。

`sql`和`reverse_sql`（如果提供）应为在数据库上运行的SQL字符串。在大多数数据库后端（除PostgreSQL之外），Django将在执行它们之前将SQL拆分为单独的语句。这需要安装[sqlparse](https://pypi.python.org/pypi/sqlparse) Python库。

您还可以传递字符串或2元组的列表。后者用于以与[_cursor.execute()_](../topics/db/sql.html#executing-custom-sql)相同的方式传递查询和参数。这三个操作是等效的：

```
migrations.RunSQL("INSERT INTO musician (name) VALUES ('Reinhardt');")
migrations.RunSQL(["INSERT INTO musician (name) VALUES ('Reinhardt');", None])
migrations.RunSQL(["INSERT INTO musician (name) VALUES (%s);", ['Reinhardt']])

```

如果要在查询中包括文字百分号，则必须将它们加倍（如果您正在传递参数）。

`state_operations`参数是这样的，您可以根据项目状态提供等同于SQL的操作；例如，如果您手动创建列，则应在此处传递包含`AddField`操作的列表，以便自动检测器仍然具有模型的最新状态（否则，当您下一次运行`makemigrations`，它将不会看到任何添加该字段的操作，因此将尝试再次运行它）。

可选的`hints`参数将作为`**hints`传递到数据库路由器的[`allow_migrate()`](../topics/db/multi-db.html#allow_migrate "allow_migrate")方法，以帮助他们进行路由决策。有关数据库提示的更多详细信息，请参阅[_Hints_](../topics/db/multi-db.html#topics-db-multi-db-hints)。

Changed in Django 1.7.1:

如果你想在没有参数的查询中包含文字百分号，你不需要再加上它们。

Changed in Django 1.8:

添加了将参数传递到`sql`和`reverse_sql`查询的功能。

已添加`hints`参数。

`RunSQL.``noop`

New in Django 1.8.

当希望操作不在给定方向执行任何操作时，将`RunSQL.noop`属性传递到`sql`或`reverse_sql`。这在使操作可逆时尤其有用。

### RunPython

_class_ `RunPython`(_code_, _reverse_code=None_, _atomic=True_, _hints=None_)

在历史上下文中运行自定义Python代码。`code`（和`reverse_code`如果提供）应该是接受两个参数的可调用对象；第一个是包含与项目历史中的操作位置匹配的历史模型的`django.apps.registry.Apps`的实例，第二个是[`SchemaEditor`](schema-editor.html#django.db.backends.base.schema.BaseDatabaseSchemaEditor "django.db.backends.base.schema.BaseDatabaseSchemaEditor")的实例。

可选的`hints`参数将作为`**hints`传递到数据库路由器的[`allow_migrate()`](../topics/db/multi-db.html#allow_migrate "allow_migrate")方法，以帮助他们做出路由决策。有关数据库提示的更多详细信息，请参阅[_Hints_](../topics/db/multi-db.html#topics-db-multi-db-hints)。

New in Django 1.8:

已添加`hints`参数。

建议您将代码写为迁移文件中`Migration`类上方的单独函数，并将其传递到`RunPython`。这里有一个使用`RunPython`在`Country`模型上创建一些初始对象的示例：

```
# -*- coding: utf-8 -*-
from django.db import models, migrations

def forwards_func(apps, schema_editor):
    # We get the model from the versioned app registry;
    # if we directly import it, it'll be the wrong version
    Country = apps.get_model("myapp", "Country")
    db_alias = schema_editor.connection.alias
    Country.objects.using(db_alias).bulk_create([
        Country(name="USA", code="us"),
        Country(name="France", code="fr"),
    ])

class Migration(migrations.Migration):

    dependencies = []

    operations = [
        migrations.RunPython(
            forwards_func,
        ),
    ]

```

这通常是您将用于创建[_data migrations_](../topics/migrations.html#data-migrations)，运行自定义数据更新和更改以及您需要访问ORM和/或Python代码的任何其他操作。

如果你从South升级，这基本上是南模式作为一个操作 - 一个或两个方法向前和向后，可用的ORM和模式操作。大多数情况下，您应该能够翻译`orm.Model`或`orm [“appname”， “Model”]` 引用此处的`apps.get_model（“appname”， “Model”）`的数据迁移的代码不变。但是，`apps`只会引用当前应用程序中的模型，除非将其他应用程序中的迁移添加到迁移的依赖关系中。

与[`RunSQL`](#django.db.migrations.operations.RunSQL "django.db.migrations.operations.RunSQL")很相似，请确保如果您在此处更改模式，则可以在Django模型系统范围之外（例如，触发器）或使用[`SeparateDatabaseAndState`](#django.db.migrations.operations.SeparateDatabaseAndState "django.db.migrations.operations.SeparateDatabaseAndState")添加将反映您对模型状态的更改的操作，否则版本化的ORM和自动检测器将停止正常工作。

默认情况下，`RunPython`将在事务中运行其内容，即使在不支持DDL事务（例如，MySQL和Oracle）的数据库上。这应该是安全的，但如果您尝试使用这些后端提供的`schema_editor`，可能会导致崩溃；在这种情况下，请设置`atomic=False`。

警告

`RunPython`不会神奇地改变模型的连接；您调用的任何模型方法将转到默认数据库，除非您为它们提供当前数据库别名（可从`schema_editor.connection.alias`获得，其中`schema_editor`是您的第二个参数功能）。

_static_ `RunPython.``noop`()

New in Django 1.8.

当希望操作在给定方向不执行任何操作时，将`RunPython.noop`方法传递到`code`或`reverse_code`。这在使操作可逆时尤其有用。

### SeparateDatabaseAndState

_class_ `SeparateDatabaseAndState`(_database_operations=None_, _state_operations=None_)

高度专业化的操作，允许您混合和匹配数据库（模式更改）和操作的状态（自动检测器供电）方面。

它接受两个操作列表，并且当被要求应用状态时将使用状态列表，并且当被要求对数据库应用更改时将使用数据库列表。不要使用此操作，除非你非常确定你知道你在做什么。

## 写你自己

操作有一个相对简单的API，它们的设计使您可以轻松地编写自己的内容来补充内置的Django。`Operation`的基本结构如下所示：

```
from django.db.migrations.operations.base import Operation

class MyCustomOperation(Operation):

    # If this is False, it means that this operation will be ignored by
    # sqlmigrate; if true, it will be run and the SQL collected for its output.
    reduces_to_sql = False

    # If this is False, Django will refuse to reverse past this operation.
    reversible = False

    def __init__(self, arg1, arg2):
        # Operations are usually instantiated with arguments in migration
        # files. Store the values of them on self for later use.
        pass

    def state_forwards(self, app_label, state):
        # The Operation should take the 'state' parameter (an instance of
        # django.db.migrations.state.ProjectState) and mutate it to match
        # any schema changes that have occurred.
        pass

    def database_forwards(self, app_label, schema_editor, from_state, to_state):
        # The Operation should use schema_editor to apply any changes it
        # wants to make to the database.
        pass

    def database_backwards(self, app_label, schema_editor, from_state, to_state):
        # If reversible is True, this is called when the operation is reversed.
        pass

    def describe(self):
        # This is used to describe what the operation does in console output.
        return "Custom Operation"

```

您可以使用此模板并使用它，但我们建议您查看`django.db.migrations.operations`中的内置Django操作 - 它们很容易阅读，并涵盖了很多示例使用像`ProjectState`的迁移框架的半内部方面以及用于获取历史模型的模式。

有些事情要注意：

*   您不需要了解太多关于`ProjectState`的信息，只需编写简单的迁移；只需知道它有一个`apps`属性，可以访问应用程序注册表（然后您可以调用`get_model`）。
*   `database_forwards`和`database_backwards`都获得两个状态传递给他们；这些只是代表`state_forwards`方法应用的差异，但是为了方便和速度的原因给你。
*   `to_state`中的data_backwards方法是_较旧的_状态；即，一旦迁移已经完成反转将是当前状态的那个。
*   您可能会看到内置操作的`references_model`实现；这是自动检测代码的一部分，并且与自定义操作无关。

作为一个简单的例子，让我们做一个加载PostgreSQL扩展（它包含一些PostgreSQL的更令人兴奋的功能）的操作。它很简单；没有模型状态更改，它所做的是运行一个命令：

```
from django.db.migrations.operations.base import Operation

class LoadExtension(Operation):

    reversible = True

    def __init__(self, name):
        self.name = name

    def state_forwards(self, app_label, state):
        pass

    def database_forwards(self, app_label, schema_editor, from_state, to_state):
        schema_editor.execute("CREATE EXTENSION IF NOT EXISTS %s" % self.name)

    def database_backwards(self, app_label, schema_editor, from_state, to_state):
        schema_editor.execute("DROP EXTENSION %s" % self.name)

    def describe(self):
        return "Creates extension %s" % self.name

```

