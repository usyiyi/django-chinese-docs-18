

# 序列化Django对象

通常情况下，这种形式是基于文本的，它被用来发送Django的数据，当然，序列化处理的形式也有例外（基于文本或者相反）。

参见

如果您只是想从表中获取一些数据到序列化形式，可以使用[`dumpdata`](../ref/django-admin.html#django-admin-dumpdata)管理命令。

## 序列化数据

从最高层面来看,序列化数据是一项非常简单的操作

```
from django.core import serializers
data = serializers.serialize("xml", SomeModel.objects.all())

```

传递给 `serialize` 方法的参数有二：一个序列化目标格式(参见 [Serialization formats](#id2)) ，另外一个是序列号的对象[`QuerySet`](../ref/models/querysets.html#django.db.models.query.QuerySet "django.db.models.query.QuerySet"). (事实上，第二个参数可以是任何可迭代的Django Model实例，但它很多情况下就是一个QuerySet).

`django.core.serializers.``get_serializer`(_format_)

你也可以直接序列化一个对象

```
XMLSerializer = serializers.get_serializer("xml")
xml_serializer = XMLSerializer()
xml_serializer.serialize(queryset)
data = xml_serializer.getvalue()

```

如果您想将数据直接序列化为类似文件的对象（其中包含[`HttpResponse`](../ref/request-response.html#django.http.HttpResponse "django.http.HttpResponse")），则此选项非常有用：

```
with open("file.xml", "w") as out:
    xml_serializer.serialize(SomeModel.objects.all(), stream=out)

```

注意

调用具有未知[_格式_](#serialization-formats)的[`get_serializer()`](#django.core.serializers.get_serializer "django.core.serializers.get_serializer")会产生`django.core.serializers.SerializerDoesNotExist`异常。

### 字段子集

如果只想要序列化一部分字段，可以为序列化程序指定`字段`参数：

```
from django.core import serializers
data = serializers.serialize('xml', SomeModel.objects.all(), fields=('name','size'))

```

在本示例中，只有每个模型的`名称`和`尺寸`属性都将被序列化。

注意

根据你定义的模型，您会发现不能反序列化一个只序列化其字段子集的模型。如果序列化对象未指定模型所必需的所有字段，则解序器将无法保存反序列化的实例。

### Inherited Models 继承模型

如果你定义的模型继承了[_abstract base class_](db/models.html#abstract-base-classes), 你不需要对它做任何特别的处理。只需对需要序列化的对象调用serializers，就能输出该对象完整的字段。

但是，如果您有使用[_multi-table inheritance_](db/models.html#multi-table-inheritance)的模型，则还需要序列化模型的所有基类。这是因为只有在模型上本地定义的字段才会被序列化。例如，考虑以下模型：

```
class Place(models.Model):
    name = models.CharField(max_length=50)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)

```

如果你只序列化餐厅模型：

```
data = serializers.serialize('xml', Restaurant.objects.all())

```

序列化输出上的字段将只包含`serves_hot_dogs`属性。基类的`name`属性将被忽略。

为了完全串行化您的`Restaurant`实例，您还需要将`Place`模型序列化：

```
all_objects = list(Restaurant.objects.all()) + list(Place.objects.all())
data = serializers.serialize('xml', all_objects)

```

## 反序列化数据

反序列化数据也是一个相当简单的操作：

```
for obj in serializers.deserialize("xml", data):
    do_something_with(obj)

```

如你所见，`deserialize`函数采用与`serialize`相同的格式参数，一个字符串或数据流，并返回一个迭代器。

然而，在这里它有点复杂。`deserialize`迭代器_返回的对象不是_简单的Django对象。相反，它们是包装已创建但未保存的对象和任何关联关系数据的特殊`DeserializedObject`实例。

调用`DeserializedObject.save()`将对象保存到数据库。

注意

如果序列化数据中的`pk`属性不存在或为null，则新实例将保存到数据库。

这确保了反序列化是一种非破坏性操作，即使序列化表示中的数据与数据库中当前的数据不匹配。通常，使用这些`DeserializedObject`实例看起来像：

```
for deserialized_object in serializers.deserialize("xml", data):
    if object_should_be_saved(deserialized_object):
        deserialized_object.save()

```

换句话说，通常的用法是检查反序列化的对象，以确保它们是“适当的”用于保存之前这样做。当然，如果你相信你的数据源，你可以保存对象，继续前进。

Django对象本身可以被检查为`deserialized_object.object`。如果模型中不存在序列化数据中的字段，则会出现`DeserializationError`，除非`ignorenonexistent`参数以`True`

```
serializers.deserialize("xml", data, ignorenonexistent=True)

```

## 序列化格式

Django支持多种序列化格式，其中一些格式要求您安装第三方Python模块：

<colgroup><col width="14%"> <col width="86%"></colgroup> 
| Identifier | Information |
| --- | --- |
| `xml` | Serializes to and from a simple XML dialect. |
| `json` | Serializes to and from [JSON](http://json.org/). |
| `yaml` | Serializes to YAML (YAML Ain’t a Markup Language). This serializer is only available if [PyYAML](http://www.pyyaml.org/) is installed. |

### XML

基本的XML序列化格式很简单：

```
<?xml version="1.0" encoding="utf-8"?>
<django-objects version="1.0">
    <object pk="123" model="sessions.session">
        <field type="DateTimeField" name="expire_date">2013-01-16T08:16:59.844560+00:00</field>
        <!-- ... -->
    </object>
</django-objects>

```

序列化或反序列化的对象的整个集合由包含多个`&lt;object&gt;`元素的`&lt;django-objects&gt;` -tag表示。每个这样的对象有两个属性：“pk”和“model”，后者由应用程序的名称（“sessions”）和用点分隔的模型的小写名称（“会话”）表示。

对象的每个字段被序列化为`&lt;field&gt;` - 运行字段“type”和“name”的元素。元素的文本内容表示应存储的值。

外键和其他关系字段的处理方式略有不同：

```
<object pk="27" model="auth.permission">
    <!-- ... -->
    <field to="contenttypes.contenttype" name="content_type" rel="ManyToOneRel">9</field>
    <!-- ... -->
</object>

```

在这个例子中，我们指定了auth。PK 27的权限对象具有内容类型的外键。ContentType实例与PK 9。

ManyToMany关系导出为绑定它们的模型。例如，auth。用户模型与auth有这样的关系。权限模型：

```
<object pk="1" model="auth.user">
    <!-- ... -->
    <field to="auth.permission" name="user_permissions" rel="ManyToManyRel">
        <object pk="46"></object>
        <object pk="47"></object>
    </field>
</object>

```

此示例将给定用户与具有PK 46和47的权限模型链接。

### JSON

当保持与之前相同的示例数据时，将以下列方式序列化为JSON：

```
[
    {
        "pk": "4b678b301dfd8a4e0dad910de3ae245b",
        "model": "sessions.session",
        "fields": {
            "expire_date": "2013-01-16T08:16:59.844Z",
            ...
        }
    }
]

```

这里的格式比使用XML简单一点。整个集合仅表示为数组，对象由具有三个属性的JSON对象表示：“pk”，“model”和“fields”。“fields”再次是一个对象，分别将每个字段的名称和值分别作为property和property-value。

外键只是将链接对象的PK作为属性值。ManyToMany关系被序列化为定义它们的模型，并被表示为PK列表。

日期和日期时间相关类型由JSON序列化程序以特殊方式处理，以使格式与[ECMA-262](http://www.ecma-international.org/ecma-262/5.1/#sec-15.9.1.15)兼容。

请注意，并非所有Django输出都可以未修改地传递给[`json`](https://docs.python.org/3/library/json.html#module-json "(in Python v3.4)")。特别地，[_lazy translation objects_](i18n/translation.html#lazy-translations)需要为它们写入[特殊编码器](https://docs.python.org/library/json.html#encoders-and-decoders)。这样的东西会工作：

```
from django.utils.functional import Promise
from django.utils.encoding import force_text
from django.core.serializers.json import DjangoJSONEncoder

class LazyEncoder(DjangoJSONEncoder):
    def default(self, obj):
        if isinstance(obj, Promise):
            return force_text(obj)
        return super(LazyEncoder, self).default(obj)

```

另请注意，GeoDjango提供了一个[_customized GeoJSON serializer_](../ref/contrib/gis/serializers.html)。

### YAML

YAML序列化看起来非常类似于JSON。对象列表被序列化为具有键“pk”，“model”和“fields”的序列映射。每个字段再次是一个映射，其中键是字段的名称和值的值：

```
-   fields: {expire_date: !!timestamp '2013-01-16 08:16:59.844560+00:00'}
    model: sessions.session
    pk: 4b678b301dfd8a4e0dad910de3ae245b

```

参考字段再次仅由PK或PK序列表示。

## 自然钥匙

外键和多对多关系的默认序列化策略是序列化关系中对象的主键的值。这个策略适用于大多数对象，但在某些情况下可能会导致困难。

考虑具有引用[`ContentType`](../ref/contrib/contenttypes.html#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")的外键的对象列表的情况。如果你要序列化一个引用内容类型的对象，那么你需要有一种方法来引用该内容类型。由于`ContentType`对象在数据库同步过程中由Django自动创建，所以给定内容类型的主键不容易预测；它将取决于执行[`migrate`](../ref/django-admin.html#django-admin-migrate)的方式和时间。这对于所有自动生成对象的模型都是如此，尤其包括[`Permission`](../ref/contrib/auth.html#django.contrib.auth.models.Permission "django.contrib.auth.models.Permission")，[`Group`](../ref/contrib/auth.html#django.contrib.auth.models.Group "django.contrib.auth.models.Group")和[`User`](../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")。

警告

您不应将自动生成的对象包括在夹具或其他序列化数据中。偶尔，夹具中的主键可能与数据库中的主键匹配，加载夹具将没有效果。在更可能的情况下，它们不匹配，夹具加载将失败，并出现[`IntegrityError`](../ref/exceptions.html#django.db.IntegrityError "django.db.IntegrityError")。

还有方便的事情。整数id并不总是最方便的引用对象的方法；有时，更自然的参考将是有帮助的。

正是由于这些原因，Django提供了_自然键_。自然键是可用于唯一地标识对象实例而不使用主键值的值的元组。

### 自然键的反序列化

考虑以下两个模型：

```
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    birthdate = models.DateField()

    class Meta:
        unique_together = (('first_name', 'last_name'),)

class Book(models.Model):
    name = models.CharField(max_length=100)
    author = models.ForeignKey(Person)

```

通常，`Book`的序列化数据将使用整数来引用作者。例如，在JSON中，一本书可能被序列化为：

```
...
{
    "pk": 1,
    "model": "store.book",
    "fields": {
        "name": "Mostly Harmless",
        "author": 42
    }
}
...

```

这不是一个特别自然的方式来引用作者。它要求你知道作者的主键值；它还要求这个主键值是稳定和可预测的。

然而，如果我们添加自然键处理到人，灯具变得更加人性化。要添加自然键处理，请使用`get_by_natural_key()`方法为人定义默认管理器。在Person的情况下，良好的自然键可能是名和姓：

```
from django.db import models

class PersonManager(models.Manager):
    def get_by_natural_key(self, first_name, last_name):
        return self.get(first_name=first_name, last_name=last_name)

class Person(models.Model):
    objects = PersonManager()

    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    birthdate = models.DateField()

    class Meta:
        unique_together = (('first_name', 'last_name'),)

```

现在书可以使用该自然键来引用`Person`对象：

```
...
{
    "pk": 1,
    "model": "store.book",
    "fields": {
        "name": "Mostly Harmless",
        "author": ["Douglas", "Adams"]
    }
}
...

```

当您尝试加载此序列化数据时，Django将使用`get_by_natural_key()`方法解析`[“Douglas”， “Adams”] `转换为实际`Person`对象的主键。

注意

无论用于自然键的任何字段都必须能够唯一地标识对象。这通常意味着您的模型将在自然键中的一个或多个字段中具有唯一性子句（单个字段上的unique = True，或多个字段中的`unique_together`）。但是，不需要在数据库级别强制实施唯一性。如果您确定一组字段将有效地唯一，您仍然可以将这些字段用作自然键。

New in Django 1.7.

没有主键的对象的反序列化将始终检查模型的管理器是否具有`get_by_natural_key()`方法，如果是，则使用它来填充反序列化对象的主键。

### 自然键的序列化

那么在序列化对象时，如何让Django发射一个自然的键呢？首先，你需要添加另一个方法 - 这一次到模型本身：

```
class Person(models.Model):
    objects = PersonManager()

    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    birthdate = models.DateField()

    def natural_key(self):
        return (self.first_name, self.last_name)

    class Meta:
        unique_together = (('first_name', 'last_name'),)

```

该方法应该总是返回一个自然键元组 - 在这个例子中，`（第一个 名称， 最后 t4&gt;`。然后，当您调用`serializers.serialize()`时，您提供`use_natural_foreign_keys=True`或`use_natural_primary_keys=True`

```
>>> serializers.serialize('json', [book1, book2], indent=2,
...      use_natural_foreign_keys=True, use_natural_primary_keys=True)

```

当指定`use_natural_foreign_keys=True`时，Django将使用`natural_key()`方法将任何外键引用序列化为定义该方法的类型的对象。

当指定`use_natural_primary_keys=True`时，Django不会在此对象的序列化数据中提供主键，因为它可以在反序列化期间计算：

```
...
{
    "model": "store.person",
    "fields": {
        "first_name": "Douglas",
        "last_name": "Adams",
        "birth_date": "1952-03-11",
    }
}
...

```

当您需要将序列化数据加载到现有数据库中，并且不能保证序列化主键值尚未使用时，这是非常有用的，并且不需要确保反序列化对象保留相同的主键。

如果您使用[`dumpdata`](../ref/django-admin.html#django-admin-dumpdata)生成序列化数据，请使用[`--natural-foreign`](../ref/django-admin.html#django-admin-option---natural-foreign)和[`--natural-primary`](../ref/django-admin.html#django-admin-option---natural-primary)命令行标志生成自然键。

注意

您不需要同时定义`natural_key()`和`get_by_natural_key()`。如果您不希望Django在序列化过程中输出自然键，但您希望保留加载自然键的能力，那么您可以选择不实现`natural_key()`方法。

相反，如果（由于某种奇怪的原因）你希望Django在序列化过程中输出自然键，但_不能_能够加载这些键值，只是不要定义`get_by_natural_key()`

Changed in Django 1.7:

以前，对于`serializers.serialize()`和&lt;cite&gt;-n&lt;/cite&gt;或 &lt;cite&gt;- 自然&lt;/cite&gt;命令行只有`use_natural_keys`这些已被弃用，支持`use_natural_foreign_keys`和`use_natural_primary_keys`参数和相应的[`--natural-foreign`](../ref/django-admin.html#django-admin-option---natural-foreign)和[`--natural-primary`](../ref/django-admin.html#django-admin-option---natural-primary)选项[`dumpdata`](../ref/django-admin.html#django-admin-dumpdata)。

原始参数和命令行标志保持向后兼容性，并映射到新的`use_natural_foreign_keys`参数和&lt;cite&gt;-natural-foreign&lt;/cite&gt;命令行标志。他们将在Django 1.9中删除。

### 序列化期间的依赖关系

由于自然键依赖于数据库查找来解析引用，因此重要的是数据在引用之前存在。您不能使用自然键创建“前向引用” - 您引用的数据必须存在，然后才能包含该数据的自然键引用。

为了适应此限制，使用[`--natural-foreign`](../ref/django-admin.html#django-admin-option---natural-foreign)选项的[`dumpdata`](../ref/django-admin.html#django-admin-dumpdata)调用将序列化之前使用`natural_key()`方法序列化任何模型标准主键对象。

然而，这可能并不总是足够。如果你的自然键引用另一个对象（通过使用外键或自然键对另一个对象作为自然键的一部分），那么你需要能够确保自然键所依赖的对象在序列化数据中出现在自然键之前需要它们。

要控制此顺序，可以在`natural_key()`方法上定义依赖关系。您可以通过在`natural_key()`方法本身设置`dependencies`属性来实现。

例如，让我们从上面的例子中添加一个自然键到`Book`模型：

```
class Book(models.Model):
    name = models.CharField(max_length=100)
    author = models.ForeignKey(Person)

    def natural_key(self):
        return (self.name,) + self.author.natural_key()

```

`Book`的自然键是其名称和作者的组合。这意味着`Person`必须在`Book`之前序列化。要定义这个依赖，我们添加一行：

```
def natural_key(self):
    return (self.name,) + self.author.natural_key()
natural_key.dependencies = ['example_app.person']

```

此定义确保所有`Person`对象在任何`Book`对象之前序列化。反过来，在`Person`和`Book`序列化之后，引用`Book`的任何对象将被序列化。

