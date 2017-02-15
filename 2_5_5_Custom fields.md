

# 编写自定义 model 字段

## 介绍

[_model 参考_](../topics/db/models.html)文档已经介绍了如何使用 Django 的标准字段类；例如 [`CharField`](../ref/models/fields.html#django.db.models.CharField "django.db.models.CharField")， [`DateField`](../ref/models/fields.html#django.db.models.DateField "django.db.models.DateField")，等等。对于很多应用来说，这些类足够用了。 但是在某些情况下， 你所用的Django 不具备你需要的某些精巧功能，或是你想使用的字段与 Django 自带字段完全不同。

Django 内置的字段类型并不能覆盖所有可能遇到的数据库的列类型，仅仅是些普通的字段类型，例如`VARCHAR`和`INTEGER`。对于更多不常用的列类型，比如地理定位数据和诸如[PostgreSQL自定义类型](http://www.postgresql.org/docs/current/interactive/sql-createtype.html)的自定义字段，你可以定义你自己的Django `Field`  子类。

有两种实现方式：你可以编写一个复杂的 Python 对象，让它以某种方式将数据序列化，以适应某个数据库的列类型； 或是你创建一个`Field`的子类，从而让你可以使用 model 中的对象。

### 示例对象

创建自定义字段需要注意很多细节。 为了使这一章内容容易理解，自始至终我们都只使用这一个例子：包装一个 Python 对象来表示手中[桥牌](http://en.wikipedia.org/wiki/Contract_bridge)的详细信息。不用担心，这个例子并不要求你会玩桥牌。 你只要知道 52 张牌被平均分配给四个玩家，按惯例，他们被称之为_北_, _东_, _南_ 和 _西_。我们的类看起来就象这个样子：

```
class Hand(object):
    """A hand of cards (bridge style)"""

    def __init__(self, north, east, south, west):
        # Input parameters are lists of cards ('Ah', '9s', etc)
        self.north = north
        self.east = east
        self.south = south
        self.west = west

    # ... (other possibly useful methods omitted) ...

```

这只是一个普通的 Python 类，并没有对 Django 做特别的设定。 在 model 中我们可以象下面这样使用 Hand (我们假设 model 中的 `hand` 属性是 `Hand` 类的一个实例)：

```
example = MyModel.objects.get(pk=1)
print(example.hand.north)

new_hand = Hand(north, east, south, west)
example.hand = new_hand
example.save()

```

我们就象使用任何 Python 类一样，对 model 中的 `hand` 属性进行赋值和取值。利用这一点让 Django 知道如何处理保存和载入一个对象。

为了在 model 中使用 `Hand` 类，我们**不必**为这个类做任何的改动。这是非常有用的，它表示着你可以很容易地为已存在的类编写模型支持，而不必改动类的原代码。

注意

你可能只想利用自定义数据库列类型，将数据处理成模型中的标准 Python 类型， 比如：字符串，浮点数等等。 这种情况与我们的 `Hand` 例子非常相似，我们随着文档的展开对两者的差异进行比较。

## 后台原理

### 数据库存储

可以简单的认为 model 字段提供了一种方法来接受普通的 Python 对象，比如布尔值，`datetime`，或是象`Hand`这样更复杂的对象，然后在操作数据库时，对对象进行格式转换以适应数据库。（还有序列化也是同样处理，但接下来我们会看到，一旦我们掌握了数据库这方面的转换，再对序列化做处理就游刃有余了）

模型中的字段必须以某种方式转化为现有的数据库字段类型。不同的数据库提供了不同的有效的列类型的集合，但是规则仍然是相同的：这些是唯一工作类型。任何你想存储在数据库中必须适应这些类型之一。

通常情况下，你可以写一个Django字段来匹配特定的数据库行的类型，或有一个相当直接的的方式将你的数据转化为一个字符串。

以我们的 `Hand` 为例，我们可以将卡片的数据以预先决定好的顺序，连接转化为一个104个字符的字符串– 也就是说， _north_ 卡片排在第一, 然后是 _east_, _south_ 和 _west_ 。所以`Hand` 对象可以在数据库中以 text或者character 的columns的形式储存

### 字段类是什么？

所有Django的字段（当我们在本文档中提到_字段_时，我们总是指模型字段，而不是指[_表单字段_](../ref/forms/fields.html)）是[`django.db.models.Field的子类`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")。Django记录有关字段的大多数信息对于所有字段都是通用的 - 名称，帮助文本，唯一性等。存储所有该信息由`Field`处理。我们将详细了解`Field`以后可以做什么；现在，足以说，一切都来自`Field`，然后自定义类行为的关键片段。

重要的是要意识到Django字段类不是你的model的属性。模型属性包含普通的Python对象。当创建模型类时，在模型中定义的字段类实际上存储在`Meta`类中（此处的具体细节不重要）。这是因为当你只是创建和修改属性时，字段类不是必需的。相反，它们提供了在属性值和存储在数据库中或发送到[_serializer_](../topics/serialization.html)之间进行转换的机制。

创建自己的自定义字段时请记住这一点。您编写的Django `Field`子类提供了以各种方式在Python实例和数据库/序列化器值之间进行转换的机制（例如，在存储值和使用值之间存在差异）。如果这听起来有点棘手，不要担心 - 这将变得更清楚在下面的例子。只要记住，当你想要一个自定义字段时，你最终会创建两个类：

*   第一类是用户将操作的Python对象。他们将它分配给模型属性，他们将从它读取用于显示的目的，这样的东西。这是我们示例中的`Hand`类。
*   第二个类是`Field`子类。这是知道如何在其永久存储形式和Python表单之间来回转换你的第一个类的类。

## 编写字段子类

在规划[`Field`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")子类时，请先考虑您的新字段与之最相似的现有[`Field`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")类。你可以继承一个现有的Django字段并保存自己一些工作吗？如果没有，你应该继承[`Field`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")类，所有类都从其中降序。

初始化您的新字段是一个问题，从您的案例中分离出任何特定于常见参数的参数，并将后者传递到[`Field`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")的`__init__()`

在我们的示例中，我们将调用字段`HandField`。（最好调用[`Field`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")子类`&lt;Something&gt;Field`，因此很容易被识别为[`Field`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")子类）。它不像任何现有字段，因此我们将直接从[`Field`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")子类化：

```
from django.db import models

class HandField(models.Field):

    description = "A hand of cards (bridge style)"

    def __init__(self, *args, **kwargs):
        kwargs['max_length'] = 104
        super(HandField, self).__init__(*args, **kwargs)

```

我们的`HandField`接受大多数标准字段选项（见下面的列表），但我们确保它具有固定长度，因为它只需要持有52个卡片值加上他们的衣服；共104个字符。

注意

许多Django的模型字段接受他们不做任何事情的选项。例如，您可以将[`editable`](../ref/models/fields.html#django.db.models.Field.editable "django.db.models.Field.editable")和[`auto_now`](../ref/models/fields.html#django.db.models.DateField.auto_now "django.db.models.DateField.auto_now")同时传递到[`django.db.models.DateField`](../ref/models/fields.html#django.db.models.DateField "django.db.models.DateField")，它将忽略[`editable`](../ref/models/fields.html#django.db.models.Field.editable "django.db.models.Field.editable")参数（[`auto_now`](../ref/models/fields.html#django.db.models.DateField.auto_now "django.db.models.DateField.auto_now")被设置意味着`editable=False`）。在这种情况下不会出现错误。

此行为简化了字段类，因为它们不需要检查不必要的选项。它们只是将所有选项传递给父类，然后不再使用它们。这取决于你是否希望字段对他们选择的选项更加严格，或者使用当前字段的更简单，更宽松的行为。

`Field.__init__()`方法采用以下参数：

*   [verbose_name](../ref/models/fields.html#django.db.models.Field.verbose_name "django.db.models.Field.verbose_name")
*   `name`
*   [primary_key(关键字)](../ref/models/fields.html#django.db.models.Field.primary_key "django.db.models.Field.primary_key")
*   [max_length(最大长度)](../ref/models/fields.html#django.db.models.CharField.max_length "django.db.models.CharField.max_length")
*   [unique(唯一性)](../ref/models/fields.html#django.db.models.Field.unique "django.db.models.Field.unique")
*   [blank(空白)](../ref/models/fields.html#django.db.models.Field.blank "django.db.models.Field.blank")
*   [null(空值)](../ref/models/fields.html#django.db.models.Field.null "django.db.models.Field.null")
*   [db_index(数据库(创建)索引)](../ref/models/fields.html#django.db.models.Field.db_index "django.db.models.Field.db_index")
*   `rel`：用于相关字段（如[`ForeignKey`](../ref/models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")）。仅供高级使用。
*   [default(默认值)](../ref/models/fields.html#django.db.models.Field.default "django.db.models.Field.default")
*   [editable(可编辑)](../ref/models/fields.html#django.db.models.Field.editable "django.db.models.Field.editable")
*   `serialize`：如果`False`，当模型传递给Django的[_serializers_](../topics/serialization.html)时，字段不会被序列化。默认为`True`。
*   [unique_for_date](../ref/models/fields.html#django.db.models.Field.unique_for_date "django.db.models.Field.unique_for_date")
*   [unique_for_month](../ref/models/fields.html#django.db.models.Field.unique_for_month "django.db.models.Field.unique_for_month")
*   [unique_for_year](../ref/models/fields.html#django.db.models.Field.unique_for_year "django.db.models.Field.unique_for_year")
*   [choices(选择)](../ref/models/fields.html#django.db.models.Field.choices "django.db.models.Field.choices")
*   [help_text](../ref/models/fields.html#django.db.models.Field.help_text "django.db.models.Field.help_text")
*   [db_column](../ref/models/fields.html#django.db.models.Field.db_column "django.db.models.Field.db_column")
*   [`db_tablespace`](../ref/models/fields.html#django.db.models.Field.db_tablespace "django.db.models.Field.db_tablespace")：仅用于创建索引（如果后端支持[_tablespaces_](../topics/db/tablespaces.html)）。您通常可以忽略此选项。
*   [`auto_created`](../ref/models/fields.html#django.db.models.Field.auto_created "django.db.models.Field.auto_created")：`True`如果自动创建字段，就像模型继承使用的[`OneToOneField`](../ref/models/fields.html#django.db.models.OneToOneField "django.db.models.OneToOneField")。仅供高级使用。

在上面列表中没有解释的所有选项具有与正常Django字段相同的含义。有关示例和详细信息，请参阅[_field documentation_](../ref/models/fields.html)。

### Field deconstruction(model域 解析)

New in Django 1.7:

`deconstruct()`是Django 1.7及更高版本中migrations框架的一部分。如果您有来自以前版本的自定义字段，则需要添加此方法才能在迁移中使用它们。

写入`__init__()`方法的目的是写入`deconstruct()`方法。这个方法告诉Django如何获取一个新字段的实例，并将其减少为序列化形式 - 特别是要传递给`__init__()`的参数以重新创建它。

如果您没有在继承域的顶部添加任何额外的选项，则无需编写新的`deconstruct()`方法。然而，如果你改变了在`__init__()`中传递的参数（就像我们在`HandField`中），你需要补充传递的值。

`deconstruct()`的约定很简单；它返回一个包含四个项目的元组：字段的属性名称，字段类的完整导入路径，位置参数（作为列表）和关键字参数（作为dict）。注意，这不同于自定义类的`deconstruct()`方法[_for custom classes_](../topics/migrations.html#custom-deconstruct-method)

作为一个自定义字段作者，你不需要关心前两个值；基础`Field`类具有所有代码以计算字段的属性名称和导入路径。然而，你必须关心位置和关键字参数，因为这些可能是你正在改变的事情。

例如，在我们的`HandField`类中，我们总是强制在`__init__()`中设置max_length。基础`Field`类上的`deconstruct()`方法将会看到这个，并尝试在关键字参数中返回它；因此，我们可以从关键字参数中删除它的可读性：

```
from django.db import models

class HandField(models.Field):

    def __init__(self, *args, **kwargs):
        kwargs['max_length'] = 104
        super(HandField, self).__init__(*args, **kwargs)

    def deconstruct(self):
        name, path, args, kwargs = super(HandField, self).deconstruct()
        del kwargs["max_length"]
        return name, path, args, kwargs

```

如果您添加了一个新的关键字参数，则需要编写代码以将其值自动添加到`kwargs`中：

```
from django.db import models

class CommaSepField(models.Field):
    "Implements comma-separated storage of lists"

    def __init__(self, separator=",", *args, **kwargs):
        self.separator = separator
        super(CommaSepField, self).__init__(*args, **kwargs)

    def deconstruct(self):
        name, path, args, kwargs = super(CommaSepField, self).deconstruct()
        # Only include kwarg if it's not the default
        if self.separator != ",":
            kwargs['separator'] = self.separator
        return name, path, args, kwargs

```

更复杂的示例超出了本文档的范围，但请记住 - 对于Field实例的任何配置，`deconstruct()`必须返回可以传递到`__init__`那个状态。

如果您在`Field`超类中为参数设置新的默认值，请特别注意；你想要确保它们总是包含，而不是消失，如果他们采取旧的默认值。

另外，尽量避免返回值作为位置参数；在可能的情况下，返回值作为关键字参数，以实现最大的未来兼容性。当然，如果你改变事物的名字比它们在构造函数的参数列表中的位置更多，你可能更喜欢位置，但是请记住，人们将从序列化版本重构你的领域一段时间（可能是几年）这取决于你的迁移活多久。

您可以通过查看包含字段的迁移来查看解构的结果，您可以通过解构和重建字段来测试单元测试中的解构：

```
name, path, args, kwargs = my_field_instance.deconstruct()
new_instance = MyField(*args, **kwargs)
self.assertEqual(my_field_instance.some_attribute, new_instance.some_attribute)

```

### 记录自定义字段

和往常一样，你应该记录你的字段类型，让用户知道它是什么。除了为开发人员提供文档字符串外，您还可以允许管理应用程序的用户通过[_django.contrib.admindocs_](../ref/contrib/admin/admindocs.html)应用程序查看字段类型的简短说明。为此，只需在自定义字段的[`description`](../ref/models/fields.html#django.db.models.Field.description "django.db.models.Field.description")类属性中提供描述性文本即可。在上面的示例中，`admindocs`应用程序为`HandField`显示的描述将是“A手牌（桥牌）”。

在[`django.contrib.admindocs`](../ref/contrib/admin/admindocs.html#module-django.contrib.admindocs "django.contrib.admindocs: Django's admin documentation generator.")显示中，字段描述由`field.__dict__`，允许描述包含字段的参数。例如，[`CharField`](../ref/models/fields.html#django.db.models.CharField "django.db.models.CharField")的描述是：

```
description = _("String (up to %(max_length)s)")

```

### 有用的方法

创建[`Field`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")子类后，您可能会考虑覆盖几个标准方法，具体取决于字段的行为。下面的方法列表大约是重要性的降序，所以从顶部开始。

#### 自定义数据库类型

假设您创建了一个名为`mytype`的PostgreSQL自定义类型。您可以子类化`Field`并实现[`db_type()`](../ref/models/fields.html#django.db.models.Field.db_type "django.db.models.Field.db_type")方法，如下所示：

```
from django.db import models

class MytypeField(models.Field):
    def db_type(self, connection):
        return 'mytype'

```

一旦您拥有`MytypeField`，就可以在任何模型中使用它，就像任何其他`Field`类型：

```
class Person(models.Model):
    name = models.CharField(max_length=80)
    something_else = MytypeField()

```

如果您打算构建一个不依赖于数据库的应用程序，则应考虑数据库列类型的差异。例如，PostgreSQL中的日期/时间列类型称为`timestamp`，而MySQL中的相同列称为`datetime`。在[`db_type()`](../ref/models/fields.html#django.db.models.Field.db_type "django.db.models.Field.db_type")方法中处理此问题的最简单的方法是检查`connection.settings_dict['ENGINE']`属性。

例如：

```
class MyDateField(models.Field):
    def db_type(self, connection):
        if connection.settings_dict['ENGINE'] == 'django.db.backends.mysql':
            return 'datetime'
        else:
            return 'timestamp'

```

当框架为您的应用程序构建`CREATE TABLE`语句时，Django会调用[`db_type()`](../ref/models/fields.html#django.db.models.Field.db_type "django.db.models.Field.db_type")当构造包含模型字段的`WHERE`子句时，也就是当使用诸如`get()`，`filter()`和`exclude()`并将模型字段作为参数。它不会在任何其他时间调用，因此它可以执行稍微复杂的代码，例如`connection.settings_dict`检查上面的示例。

某些数据库列类型接受参数，例如`CHAR(25)`，其中参数`25`表示最大列长度。在这种情况下，如果在模型中指定参数，而不是在`db_type()`方法中硬编码，则它更灵活。例如，使用`CharMaxlength25Field`没有什么意义，如下所示：

```
# This is a silly example of hard-coded parameters.
class CharMaxlength25Field(models.Field):
    def db_type(self, connection):
        return 'char(25)'

# In the model:
class MyModel(models.Model):
    # ...
    my_field = CharMaxlength25Field()

```

这样做的更好的方法是使参数在运行时可指定 - 即当类被实例化时。要这样做，只需实现`Field.__init__()`，像这样：

```
# This is a much more flexible example.
class BetterCharField(models.Field):
    def __init__(self, max_length, *args, **kwargs):
        self.max_length = max_length
        super(BetterCharField, self).__init__(*args, **kwargs)

    def db_type(self, connection):
        return 'char(%s)' % self.max_length

# In the model:
class MyModel(models.Model):
    # ...
    my_field = BetterCharField(25)

```

最后，如果您的列需要真正复杂的SQL设置，请从[`db_type()`](../ref/models/fields.html#django.db.models.Field.db_type "django.db.models.Field.db_type")返回`None`。这将导致Django的SQL创建代码跳过此字段。然后，您将负责以某种其他方式在正确的表中创建列，当然，但是这让你有一种方法来告诉Django脱离方式。

#### 将值转换为Python对象

Changed in Django 1.8:

历史上，Django提供了一个称为`SubfieldBase`的元类，它在赋值时总是调用[`to_python()`](../ref/models/fields.html#django.db.models.Field.to_python "django.db.models.Field.to_python")。这对于自定义数据库转换，聚合或值查询没有很好地发挥作用，因此已被替换为[`from_db_value()`](../ref/models/fields.html#django.db.models.Field.from_db_value "django.db.models.Field.from_db_value")。

如果您的自定义[`Field`](../ref/models/fields.html#django.db.models.Field "django.db.models.Field")类处理比字符串，日期，整数或浮点数更复杂的数据结构，则可能需要覆盖[`from_db_value()`](../ref/models/fields.html#django.db.models.Field.from_db_value "django.db.models.Field.from_db_value")和[`to_python()`](../ref/models/fields.html#django.db.models.Field.to_python "django.db.models.Field.to_python")。

如果存在于字段子类，则在从数据库加载数据（包括在聚合和[`values()`](../ref/models/querysets.html#django.db.models.query.QuerySet.values "django.db.models.query.QuerySet.values")调用）的所有情况下将调用`from_db_value()`。

`to_python()`通过反序列化和从表单使用的[`clean()`](../ref/models/instances.html#django.db.models.Model.clean "django.db.models.Model.clean")方法调用。

作为一般规则，`to_python()`应优雅地处理以下任何参数：

*   正确类型的实例（例如，在我们正在进行的示例中，`Hand`）。
*   字符串
*   `None`（如果字段允许`null=True`）

在我们 `HandField` 类种,我们使用 VARCHAR 域在数据库中存储我们的数据, 因此我们需要在`from_db_value()`函数中有能力处理字符串跟 `None`值。在`to_python()`中，我们还需要处理`Hand`实例：

```
import re

from django.core.exceptions import ValidationError
from django.db import models

def parse_hand(hand_string):
    """Takes a string of cards and splits into a full hand."""
    p1 = re.compile('.{26}')
    p2 = re.compile('..')
    args = [p2.findall(x) for x in p1.findall(hand_string)]
    if len(args) != 4:
        raise ValidationError("Invalid input for a Hand instance")
    return Hand(*args)

class HandField(models.Field):
    # ...

    def from_db_value(self, value, expression, connection, context):
        if value is None:
            return value
        return parse_hand(value)

    def to_python(self, value):
        if isinstance(value, Hand):
            return value

        if value is None:
            return value

        return parse_hand(value)

```

注意，我们总是从这些方法返回`Hand`实例。这是我们要存储在模型属性中的Python对象类型。

对于`to_python()`，如果在值转换期间出现任何错误，您应该引发一个[`ValidationError`](../ref/exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")异常。

#### 将Python对象转换为查询值

由于使用数据库需要以两种方式进行转换，因此如果覆盖[`to_python()`](../ref/models/fields.html#django.db.models.Field.to_python "django.db.models.Field.to_python")，您还必须重写[`get_prep_value()`](../ref/models/fields.html#django.db.models.Field.get_prep_value "django.db.models.Field.get_prep_value")将Python对象转换回查询值。

例如：

```
class HandField(models.Field):
    # ...

    def get_prep_value(self, value):
        return ''.join([''.join(l) for l in (value.north,
                value.east, value.south, value.west)])

```

警告

如果您的自定义字段使用MySQL的`CHAR`，`VARCHAR`或`TEXT`类型，则必须确保[`get_prep_value()`](../ref/models/fields.html#django.db.models.Field.get_prep_value "django.db.models.Field.get_prep_value")当对这些类型执行查询并且提供的值是整数时，MySQL执行灵活和意外匹配，这可能导致查询在其结果中包含意外的对象。如果您始终从[`get_prep_value()`](../ref/models/fields.html#django.db.models.Field.get_prep_value "django.db.models.Field.get_prep_value")返回字符串类型，则不会发生此问题。

#### 将查询值转换为数据库值

某些数据类型（例如，日期）需要采用特定格式，才能供数据库后端使用。[`get_db_prep_value()`](../ref/models/fields.html#django.db.models.Field.get_db_prep_value "django.db.models.Field.get_db_prep_value")是应进行这些转换的方法。将用于查询的特定连接作为`connection`参数传递。这允许您使用后端特定的转换逻辑（如果需要）。

例如，Django对其[`BinaryField`](../ref/models/fields.html#django.db.models.BinaryField "django.db.models.BinaryField")使用以下方法：

```
def get_db_prep_value(self, value, connection, prepared=False):
    value = super(BinaryField, self).get_db_prep_value(value, connection, prepared)
    if value is not None:
        return connection.Database.Binary(value)
    return value

```

如果您的自定义字段在保存时需要进行特殊转换，但与用于常规查询参数的转换不同，则可以覆盖[`get_db_prep_save()`](../ref/models/fields.html#django.db.models.Field.get_db_prep_save "django.db.models.Field.get_db_prep_save")。

#### 保存前预处理值

如果要在保存之前预处理值，可以使用[`pre_save()`](../ref/models/fields.html#django.db.models.Field.pre_save "django.db.models.Field.pre_save")。例如，在[`auto_now`](../ref/models/fields.html#django.db.models.DateField.auto_now "django.db.models.DateField.auto_now")或[`auto_now_add`](../ref/models/fields.html#django.db.models.DateField.auto_now_add "django.db.models.DateField.auto_now_add")的情况下，Django的[`DateTimeField`](../ref/models/fields.html#django.db.models.DateTimeField "django.db.models.DateTimeField")使用此方法正确设置属性。

如果您覆盖此方法，则必须在结尾返回属性的值。如果对值进行任何更改，您还应该更新模型的属性，以便保存对模型的引用的代码将始终看到正确的值。

#### 准备用于数据库查找的值

与值转换一样，为数据库查找准备一个值是一个两阶段过程。

[`get_prep_lookup()`](../ref/models/fields.html#django.db.models.Field.get_prep_lookup "django.db.models.Field.get_prep_lookup")执行查找准备的第一阶段：类型转换和数据验证。

在查找（在SQL中使用`WHERE`约束）时，准备传递到数据库的`value`。The `lookup_type` parameter will be one of the valid Django filter lookups: `exact`, `iexact`, `contains`, `icontains`, `gt`, `gte`, `lt`, `lte`, `in`, `startswith`, `istartswith`, `endswith`, `iendswith`, `range`, `year`, `month`, `day`, `isnull`, `search`, `regex`, and `iregex`.

New in Django 1.7:

如果您使用[_Custom lookups_](custom-lookups.html)，则`lookup_type`可以是项目自定义查找使用的任何`lookup_name`。

您的方法必须准备好处理所有这些`lookup_type`值，如果`value`的排序错误，则应提出`ValueError`当你期望一个对象，例如）或一个`TypeError`如果你的字段不支持这种类型的查找。对于许多字段，您可以通过处理需要对字段进行特殊处理的查找类型，并将剩余部分传递给父类的[`get_db_prep_lookup()`](../ref/models/fields.html#django.db.models.Field.get_db_prep_lookup "django.db.models.Field.get_db_prep_lookup")方法。

如果你需要实现[`get_db_prep_save()`](../ref/models/fields.html#django.db.models.Field.get_db_prep_save "django.db.models.Field.get_db_prep_save")，你通常需要实现[`get_prep_lookup()`](../ref/models/fields.html#django.db.models.Field.get_prep_lookup "django.db.models.Field.get_prep_lookup")。If you don’t, [`get_prep_value()`](../ref/models/fields.html#django.db.models.Field.get_prep_value "django.db.models.Field.get_prep_value") will be called by the default implementation, to manage `exact`, `gt`, `gte`, `lt`, `lte`, `in` and `range` lookups.

您可能还希望实现此方法以限制可以与自定义字段类型一起使用的查找类型。

注意，对于`"range"`和`"in"`查找中，`get_prep_lookup`将接收一个对象列表需要将它们转换为适当类型的事物列表以传递到数据库。大多数时候，你可以重用`get_prep_value()`，或者至少考虑一些常见的部分。

例如，以下代码实现`get_prep_lookup`以将接受的查找类型限制为`exact`和`in`：

```
class HandField(models.Field):
    # ...

    def get_prep_lookup(self, lookup_type, value):
        # We only handle 'exact' and 'in'. All others are errors.
        if lookup_type == 'exact':
            return self.get_prep_value(value)
        elif lookup_type == 'in':
            return [self.get_prep_value(v) for v in value]
        else:
            raise TypeError('Lookup type %r not supported.' % lookup_type)

```

要执行查找所需的特定于数据库的数据转换，您可以覆盖[`get_db_prep_lookup()`](../ref/models/fields.html#django.db.models.Field.get_db_prep_lookup "django.db.models.Field.get_db_prep_lookup")。

#### 指定模型字段的表单字段

要自定义[`ModelForm`](../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")使用的表单字段，您可以覆盖[`formfield()`](../ref/models/fields.html#django.db.models.Field.formfield "django.db.models.Field.formfield")。

表单字段类可以通过`form_class`和`choices_form_class`参数指定；如果字段具有指定的选项，则使用后者，否则。如果不提供这些参数，将使用[`CharField`](../ref/forms/fields.html#django.forms.CharField "django.forms.CharField")或[`TypedChoiceField`](../ref/forms/fields.html#django.forms.TypedChoiceField "django.forms.TypedChoiceField")。

所有`kwargs`字典都直接传递到表单字段的`__init__()`方法。通常，你需要做的是为`form_class`（也许`choices_form_class`）参数设置好的默认值，然后委托进一步处理到父类。这可能需要您编写自定义表单字段（甚至是窗体小部件）。有关此信息，请参阅[_forms documentation_](../topics/forms/index.html)。

继续我们正在进行的示例，我们可以将[`formfield()`](../ref/models/fields.html#django.db.models.Field.formfield "django.db.models.Field.formfield")方法写为：

```
class HandField(models.Field):
    # ...

    def formfield(self, **kwargs):
        # This is a fairly standard way to set up some defaults
        # while letting the caller override them.
        defaults = {'form_class': MyFormField}
        defaults.update(kwargs)
        return super(HandField, self).formfield(**defaults)

```

这假设我们已经导入了一个`MyFormField`字段类（它有自己的默认小部件）。本文档不包括编写自定义表单字段的详细信息。

#### 仿真内置字段类型

如果您创建了[`db_type()`](../ref/models/fields.html#django.db.models.Field.db_type "django.db.models.Field.db_type")方法，则不需要担心[`get_internal_type()`](../ref/models/fields.html#django.db.models.Field.get_internal_type "django.db.models.Field.get_internal_type") - 它不会被使用太多。但有时，您的数据库存储在类型上与其他字段类似，因此您可以使用其他字段的逻辑来创建正确的列。

例如：

```
class HandField(models.Field):
    # ...

    def get_internal_type(self):
        return 'CharField'

```

不管我们使用哪个数据库后端，这将意味着[`migrate`](../ref/django-admin.html#django-admin-migrate)和其他SQL命令创建用于存储字符串的正确的列类型。

如果[`get_internal_type()`](../ref/models/fields.html#django.db.models.Field.get_internal_type "django.db.models.Field.get_internal_type")返回Django对于您正在使用的数据库后端不为已知的字符串，也就是说，它不会出现在`django.db.backends.&lt;db_name&gt;.base.`DatabaseWrapper.data_types - 字符串仍然被序列化器使用，但是默认的[`db_type()`](../ref/models/fields.html#django.db.models.Field.db_type "django.db.models.Field.db_type")方法将返回`None`。有关这可能有用的原因，请参见[`db_type()`](../ref/models/fields.html#django.db.models.Field.db_type "django.db.models.Field.db_type")的文档。如果你要在Django之外的其他地方使用serializer输出，那么将描述性字符串作为序列化字段的字段类型是一个有用的想法。

#### 转换字段数据以进行序列化

要自定义序列化程序如何序列化值，您可以覆盖[`value_to_string()`](../ref/models/fields.html#django.db.models.Field.value_to_string "django.db.models.Field.value_to_string")。调用`Field._get_val_from_obj(obj)`是获取值序列化的最佳方式。例如，由于我们的`HandField`使用字符串作为其数据存储，我们可以重用一些现有的转换代码：

```
class HandField(models.Field):
    # ...

    def value_to_string(self, obj):
        value = self._get_val_from_obj(obj)
        return self.get_prep_value(value)

```

### 一些一般建议

编写自定义字段可能是一个棘手的过程，特别是如果您在Python类型与数据库和序列化格式之间进行复杂的转换。这里有几个提示，使事情进行得更顺利：

1.  查看现有的Django字段（在`django/db/models/fields/__init__.py`）获取灵感。尝试找到一个类似于你想要的字段，并扩展它一点，而不是从头创建一个全新的字段。
2.  将一个`__str__()`（`__unicode__()`在Python 2）方法放在类中作为一个字段。有很多地方的字段代码的默认行为是调用[`force_text()`](../ref/utils.html#django.utils.encoding.force_text "django.utils.encoding.force_text")上的值。（在本文档的示例中，`value`将是`Hand`实例，而不是`HandField`）。So if your `__str__()` method (`__unicode__()` on Python 2) automatically converts to the string form of your Python object, you can save yourself a lot of work.

## 写一个`FileField`

除了上述方法，处理文件的字段还有一些其他特殊要求，必须考虑到。`FileField`提供的大多数机制（例如控制数据库存储和检索）可以保持不变，使子类处理支持特定类型文件的挑战。

Django提供了一个`File`类，用作文件内容和操作的代理。这可以被子类化以自定义如何访问文件，以及可用的方法。它位于`django.db.models.fields.files`，其默认行为在[_file documentation_](../ref/files/file.html)中解释。

一旦创建了`File`的子类，就必须告诉新的`FileField`子类使用它。为此，只需将新的`File`子类分配给`FileField`子类的特殊`attr_class`属性。

### 几个建议

除了上面的细节，还有一些指南可以大大提高字段代码的效率和可读性。

1.  Django自己的`ImageField`（在`django/db/models/fields/files.py`）的源代码是一个很好的例子来说明如何子类化`FileField`以支持特定类型的文件，因为其包含上述所有技术。
2.  尽可能缓存文件属性。由于文件可以存储在远程存储系统中，因此检索它们可能花费额外的时间或甚至金钱，这并不总是必需的。一旦检索到文件以获得关于其内容的一些数据，就尽可能地缓存那些数据，以减少在该信息的后续调用中必须检索文件的次数。

