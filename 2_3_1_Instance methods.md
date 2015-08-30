# 模型实例参考 #

该文档详细描述模型 的API。它建立在模型 和执行查询 的资料之上， 所以在阅读这篇文档之前，你可能会想要先阅读并理解那两篇文档。

我们将用执行查询中所展现的 博客应用模型 来贯穿这篇参考文献。

## 创建对象 ##

要创建模型的一个新实例，只需要像其它Python 类一样实例化它：

`class Model(**kwargs)`

关键字参数就是在你的模型中定义的字段的名字。注意，实例化一个模型不会访问数据库；若要保存，你需要save() 一下。

> 注
>
> 也许你会想通过重写 `__init__` 方法来自定义模型。无论如何，如果你这么做了，小心不要改变了调用签名——任何改变都可能阻碍模型实例被保存。尝试使用下面这些方法之一，而不是重写__init__：
>
> 1\. 在模型类中增加一个类方法：

```
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=100)

    @classmethod
    def create(cls, title):
        book = cls(title=title)
        # do something with the book
        return book

book = Book.create("Pride and Prejudice")
```

> 2\. 在自定义管理器中添加一个方法（推荐）：

```
class BookManager(models.Manager):
    def create_book(self, title):
        book = self.create(title=title)
        # do something with the book
        return book

class Book(models.Model):
    title = models.CharField(max_length=100)

    objects = BookManager()

book = Book.objects.create_book("Pride and Prejudice")
```

### 自定义模型加载 ###

`classmethod Model.from_db(db, field_names, values)`

```
New in Django 1.8.
```

`from_db()` 方法用于自定义从数据库加载时模型实例的创建。

`db` 参数包含数据库的别名，`field_names` 包含所有加载的字段的名称，`values` 包含`field_names` 中每个字段加载的值。`field_names` 与`values` 的顺序相同，所以可以使用`cls(**(zip(field_names, values)))` 来实例化对象。如果模型的所有字段都提供，会保证` values` 的顺序与`__init__()` 所期望的一致。这表示此时实例可以通过`cls(*values)` 创建。可以通过`cls._deferred `来检查是否提供所有的字段 —— 如果为 `False`，那么所有的字段都已经从数据库中加载。

除了创建新模型之前，`from_db()` 必须设置新实例`_state` 属性中的`adding` 和 `db` 标志位。

下面的示例演示如何保存从数据库中加载进来的字段原始值：

```
@classmethod
def from_db(cls, db, field_names, values):
    # default implementation of from_db() (could be replaced
    # with super())
    if cls._deferred:
        instance = cls(**zip(field_names, values))
    else:
        instance = cls(*values)
    instance._state.adding = False
    instance._state.db = db
    # customization to store the original field values on the instance
    instance._loaded_values = zip(field_names, values)
    return instance

def save(self, *args, **kwargs):
    # Check how the current values differ from ._loaded_values. For example,
    # prevent changing the creator_id of the model. (This example doesn't
    # support cases where 'creator_id' is deferred).
    if not self._state.adding and (
            self.creator_id != self._loaded_values['creator_id']):
        raise ValueError("Updating the value of creator isn't allowed")
    super(...).save(*args, **kwargs)
```

上面的示例演示`from_db() `的完整实现。当然在这里的`from_db() `中完全可以只用`super()` 调用。

## 从数据库更新对象 ##

`Model.refresh_from_db(using=None, fields=None, **kwargs)`

```
New in Django 1.8.
```

如果你需要从数据库重新加载模型的一个值，你可以使用 `refresh_from_db()` 方法。当不带参数调用这个方法时，将完成以下的动作：

模型的所有非延迟字段都更新成数据库中的当前值。
之前加载的关联实例，如果关联的值不再合法，将从重新加载的实例中删除。例如，如果重新加载的实例有一个外键到另外一个模型`Author`，那么如果 `obj.author_id != obj.author.id`，`obj.author` 将被扔掉并在下次访问它时根据`obj.author_id` 的值重新加载。
注意，只有本模型的字段会从数据库重新加载。其它依赖数据库的值不会重新加载，例如聚合的结果。

重新加载使用的数据库与实例加载时使用的数据库相同，如果实例不是从数据库加载的则使用默认的数据库。可以使用`using` 参数来强制指定重新加载的数据库。

可以回使用`fields` 参数强制设置加载的字段。

例如，要测试`update()` 调用是否得到预期的更新，可以编写类似下面的测试：

```
def test_update_result(self):
    obj = MyModel.objects.create(val=1)
    MyModel.objects.filter(pk=obj.pk).update(val=F('val') + 1)
    # At this point obj.val is still 1, but the value in the database
    # was updated to 2. The object's updated value needs to be reloaded
    # from the database.
    obj.refresh_from_db()
    self.assertEqual(obj.val, 2)
```

注意，当访问延迟的字段时，延迟字段的加载会通过这个方法加载。所以可以自定义延迟加载的行为。下面的实例演示如何在重新加载一个延迟字段时重新加载所有的实例字段：

```
class ExampleModel(models.Model):
    def refresh_from_db(self, using=None, fields=None, **kwargs):
        # fields contains the name of the deferred field to be
        # loaded.
        if fields is not None:
            fields = set(fields)
            deferred_fields = self.get_deferred_fields()
            # If any deferred field is going to be loaded
            if fields.intersection(deferred_fields):
                # then load all of them
                fields = fields.union(deferred_fields)
        super(ExampleModel, self).refresh_from_db(using, fields, **kwargs)
```

`Model.get_deferred_fields()`

```
New in Django 1.8.
```

一个辅助方法，它返回一个集合，包含模型当前所有延迟字段的属性名称。

## 验证对象 ##

验证一个模型涉及三个步骤：

1. 验证模型的字段 —— `Model.clean_fields()`
2. 验证模型的完整性 —— `Model.clean()`
3. 验证模型的唯一性 —— `Model.validate_unique()`

当你调用模型的`full_clean()` 方法时，这三个方法都将执行。

当你使用`ModelForm`时，`is_valid()` 将为表单中的所有字段执行这些验证。更多信息参见`ModelForm` 文档。 如果你计划自己处理验证出现的错误，或者你已经将需要验证的字段从`ModelForm` 中去除掉，你只需调用模型的`full_clean()` 方法。

`Model.full_clean(exclude=None, validate_unique=True)`

该方法按顺序调用`Model.clean_fields()`、`Model.clean()` 和`Model.validate_unique()`（如果`validate_unique` 为`True`），并引发一个`ValidationError`，该异常的`message_dict` 属性包含三个步骤的所有错误。

可选的`exclude` 参数用来提供一个可以从验证和清除中排除的字段名称的列表。`ModelForm` 使用这个参数来排除表单中没有出现的字段，使它们不需要验证，因为用户无法修正这些字段的错误。

注意，当你调用模型的`save()` 方法时，`full_clean() `不会 自动调用。如果你想一步就可以为你手工创建的模型运行验证，你需要手工调用它。例如：

```
from django.core.exceptions import ValidationError
try:
    article.full_clean()
except ValidationError as e:
    # Do something based on the errors contained in e.message_dict.
    # Display them to a user, or handle them programmatically.
    pass
```

`full_clean()` 第一步执行的是验证每个字段。

`Model.clean_fields(exclude=None)`

这个方法将验证模型的所有字段。可选的`exclude` 参数让你提供一个字段名称列表来从验证中排除。如果有字段验证失败，它将引发一个`ValidationError`。

`full_clean()` 第二步执行的是调用`Model.clean()`。如要实现模型自定义的验证，应该覆盖这个方法。

`Model.clean()`

应该用这个方法来提供自定义的模型验证，以及修改模型的属性。例如，你可以使用它来给一个字段自动提供值，或者用于多个字段需要一起验证的情形：

```
import datetime
from django.core.exceptions import ValidationError
from django.db import models

class Article(models.Model):
    ...
    def clean(self):
        # Don't allow draft entries to have a pub_date.
        if self.status == 'draft' and self.pub_date is not None:
            raise ValidationError('Draft entries may not have a publication date.')
        # Set the pub_date for published items if it hasn't been set already.
        if self.status == 'published' and self.pub_date is None:
            self.pub_date = datetime.date.today()
```

然而请注意，和`Model.full_clean()` 类似，调用模型的`save()` 方法时不会引起`clean()` 方法的调用。

在上面的示例中，`Model.clean()` 引发的`ValidationError` 异常通过一个字符串实例化，所以它将被保存在一个特殊的错误字典键`NON_FIELD_ERRORS`中。这个键用于整个模型出现的错误而不是一个特定字段出现的错误：

```
from django.core.exceptions import ValidationError, NON_FIELD_ERRORS
try:
    article.full_clean()
except ValidationError as e:
    non_field_errors = e.message_dict[NON_FIELD_ERRORS]
```

若要引发一个特定字段的异常，可以使用一个字典实例化`ValidationError`，其中字典的键为字段的名称。我们可以更新前面的例子，只引发`pub_date` 字段上的异常：

```
class Article(models.Model):
    ...
    def clean(self):
        # Don't allow draft entries to have a pub_date.
        if self.status == 'draft' and self.pub_date is not None:
            raise ValidationError({'pub_date': 'Draft entries may not have a publication date.'})
        ...
```

最后，`full_clean()` 将检查模型的唯一性约束。

`Model.validate_unique(exclude=None)`

该方法与`clean_fields()` 类似，只是验证的是模型的所有唯一性约束而不是单个字段的值。可选的`exclude` 参数允许你提供一个字段名称的列表来从验证中排除。如果有字段验证失败，将引发一个 `ValidationError`。

注意，如果你提供一个`exclude` 参数给`validate_unique()`，任何涉及到其中一个字段的`unique_together` 约束将不检查。

## 对象保存 ##

将一个对象保存到数据库，需要调用 `save()`方法：

`Model.save([force_insert=False, force_update=False, using=DEFAULT_DB_ALIAS, update_fields=None])`
如果你想要自定义保存的动作，你可以重写 `save()` 方法。请看 重写预定义的模型方法 了解更多细节。

模型保存过程还有一些细节的地方要注意；请看下面的章节。

### 自增的主键 ###

如果模型具有一个`AutoField` —— 一个自增的主键 —— 那么该自增的值将在第一次调用对象的`save()` 时计算并保存：

```
>>> b2 = Blog(name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b2.id     # Returns None, because b doesn't have an ID yet.
>>> b2.save()
>>> b2.id     # Returns the ID of your new object.
```

在调用`save()` 之前无法知道ID 的值，因为这个值是通过数据库而不是Django 计算。

为了方便，默认情况下每个模型都有一个`AutoField` 叫做`id`，除非你显式指定模型某个字段的 `primary_key=True`。更多细节参见`AutoField` 的文档。

#### pk 属性 ####

`Model.pk`

无论你是自己定义还是让Django 为你提供一个主键字段， 每个模型都将具有一个属性叫做`pk`。它的行为类似模型的一个普通属性，但实际上是模型主键字段属性的别名。你可以读取并设置它的值，就和其它属性一样，它会更新模型中正确的值。

#### 显式指定自增主键的值 ####

如果模型具有一个`AutoField`，但是你想在保存时显式定义一个新的对象`ID`，你只需要在保存之前显式指定它而不用依赖`ID` 自动分配的值：

```
>>> b3 = Blog(id=3, name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b3.id     # Returns 3.
>>> b3.save()
>>> b3.id     # Returns 3.
```

如果你手工赋值一个自增主键的值，请确保不要使用一个已经存在的主键值！如果你使用数据库中已经存在的主键值创建一个新的对象，Django 将假设你正在修改这个已存在的记录而不是创建一个新的记录。

接着上面的'`Cheddar Talk`' 博客示例，下面这个例子将覆盖数据库中之前的记录：

```
b4 = Blog(id=3, name='Not Cheddar', tagline='Anything but cheese.')
b4.save()  # Overrides the previous blog with ID=3!
```

出现这种情况的原因，请参见下面的[Django 如何知道是UPDATE 还是INSERT](http://python.usyiyi.cn/django/ref/models/instances.html#how-django-knows-to-update-vs-insert)。

显式指定自增主键的值对于批量保存对象最有用，但你必须有信心不会有主键冲突。

### 当你保存时，发生了什么？ ###

当你保存一个对象时，Django 执行以下步骤：

1\. 发出一个`pre-save` 信号。 发送一个`django.db.models.signals.pre_save` 信号，以允许监听该信号的函数完成一些自定义的动作。

2\. 预处理数据。 如果需要，对对象的每个字段进行自动转换。

大部分字段不需要预处理 —— 字段的数据将保持原样。预处理只用于具有特殊行为的字段。例如，如果你的模型具有一个`auto_now=True` 的`DateField`，那么预处理阶段将修改对象中的数据以确保该日期字段包含当前的时间戳。（我们的文档还没有所有具有这种“特殊行为”字段的一个列表。）

3\. 准备数据库数据。 要求每个字段提供的当前值是能够写入到数据库中的类型。

大部分字段不需要数据准备。简单的数据类型，例如整数和字符串，是可以直接写入的Python 对象。但是，复杂的数据类型通常需要一些改动。

例如，`DateField` 字段使用Python 的 `datetime` 对象来保存数据。数据库保存的不是`datetime` 对象，所以该字段的值必须转换成ISO兼容的日期字符串才能插入到数据库中。

4\. 插入数据到数据库中。 将预处理过、准备好的数据组织成一个SQL 语句用于插入数据库。

5\. 发出一个post-save 信号。 发送一个`django.db.models.signals.post_save` 信号，以允许监听听信号的函数完成一些自定义的动作。

### Django 如何知道是UPDATE 还是INSERT ###

你可能已经注意到Django 数据库对象使用同一个`save()` 方法来创建和改变对象。Django 对`INSERT` 和`UPDATE` SQL 语句的使用进行抽象。当你调用`save()` 时，Django 使用下面的算法：

+ 如果对象的主键属性为一个求值为`True` 的值（例如，非`None` 值或非空字符串），Django 将执行`UPDATE`。
+ 如果对象的主键属性没有设置或者`UPDATE` 没有更新任何记录，Django 将执行`INSERT`。

现在应该明白了，当保存一个新的对象时，如果不能保证主键的值没有使用，你应该注意不要显式指定主键值。关于这个细微差别的更多信息，参见上文的显示指定主键的值 和下文的强制使用`INSERT` 或`UPDATE`。

在Django 1.5 和更早的版本中，在设置主键的值时，Django 会作一个 `SELECT`。如果`SELECT` 找到一行，那么Django 执行`UPDATE`，否则执行`INSERT`。旧的算法导致`UPDATE` 情况下多一次查询。有极少数的情况，数据库不会报告有一行被更新，即使数据库包含该对象的主键值。有个例子是PostgreSQL 的`ON UPDATE` 触发器，它返回NULL。在这些情况下，可能要通过将`select_on_save` 选项设置为`True` 以启用旧的算法。

#### 强制使用INSERT 或UPDATE ####
在一些很少见的场景中，需要强制`save()` 方法执行SQL 的 `INSERT` 而不能执行`UPDATE`。或者相反：更新一行而不是插入一个新行。在这些情况下，你可以传递`force_insert=True` 或 `force_update=True` 参数给`save()` 方法。显然，两个参数都传递是错误的：你不可能同时插入和更新！

你应该极少需要使用这些参数。Django 几乎始终会完成正确的事情，覆盖它将导致错误难以跟踪。这个功能只用于高级用法。

使用`update_fields` 将强制使用类似`force_update` 的更新操作。

### 基于已存在字段值的属性更新 ###

有时候你需要在一个字段上执行简单的算法操作，例如增加或者减少当前值。实现这点的简单方法是像下面这样：

```
>>> product = Product.objects.get(name='Venezuelan Beaver Cheese')
>>> product.number_sold += 1
>>> product.save()
```

如果从数据库中读取的旧的`number_sold` 值为10，那么写回到数据库中的值将为11。

通过将更新基于原始字段的值而不是显式赋予一个新值，这个过程可以[避免竞态条件](http://python.usyiyi.cn/django/ref/models/expressions.html#avoiding-race-conditions-using-f)而且更快。Django 提供`F 表达式` 用于这种类型的相对更新。利用`F 表达式`，前面的示例可以表示成：

```
>>> from django.db.models import F
>>> product = Product.objects.get(name='Venezuelan Beaver Cheese')
>>> product.number_sold = F('number_sold') + 1
>>> product.save()
```

更多细节，请参见`F 表达式` 和它们[在更新查询中的用法](http://python.usyiyi.cn/django/topics/db/queries.html#topics-db-queries-update)。

### 指定要保存的字段 ###

如果传递给`save()` 的`update_fields` 关键字参数一个字段名称列表，那么将只有该列表中的字段会被更新。如果你想更新对象的一个或几个字段，这可能是你想要的。不让模型的所有字段都更新将会带来一些轻微的性能提升。例如：

```
product.name = 'Name changed again'
product.save(update_fields=['name'])
```

`update_fields` 参数可以是任何包含字符串的可迭代对象。空的`update_fields` 可迭代对象将会忽略保存。如果为`None` 值，将执行所有字段上的更新。

指定`update_fields` 将强制使用更新操作。

当保存通过延迟模型加载（`only()` 或`defer()`）进行访问的模型时，只有从数据库中加载的字段才会得到更新。这种情况下，有个自动的`update_fields`。如果你赋值或者改变延迟字段的值，该字段将会添加到更新的字段中。

## 删除对象 ##

`Model.delete([using=DEFAULT_DB_ALIAS])`

发出一个SQL `DELETE` 操作。它只在数据库中删除这个对象；其Python 实例仍将存在并持有各个字段的数据。

更多细节，包括如何批量删除对象，请参见[删除对象](http://python.usyiyi.cn/django/topics/db/queries.html#topics-db-queries-delete)。

如果你想自定义删除的行为，你可以覆盖`delete()` 方法。详见[覆盖预定义的模型方法](http://python.usyiyi.cn/django/topics/db/models.html#overriding-model-methods)。

## Pickling 对象 ##

当你`pickle` 一个模型时，它的当前状态是pickled。当你unpickle 它时，它将包含pickle 时模型的实例，而不是数据库中的当前数据。

> 你不可以在不同版本之间共享pickles
>
> 模型的Pickles 只对于产生它们的Django 版本有效。如果你使用Django 版本N pickle，不能保证Django 版本N+1 可以读取这个pickle。Pickles 不应该作为长期的归档策略。
>
> `New in Django 1.8.`
>
> 因为pickle 兼容性的错误很难诊断例如一个悄无声息损坏的对象，当你unpickle 模型使用的Django 版本与pickle 时的不同将引发一个`RuntimeWarning`。

## 其它的模型实例方法 ##

有几个实例方法具有特殊的目的。

> 注
>
> 在Python 3 上，因为所有的字段都原生被认为是Unicode，只需使用`__str__()` 方法（`__unicode__()` 方法被废弃）。如果你想与Python 2 兼容，你可以使用`python_2_unicode_compatible()` 装饰你的模型类。

### \_\_unicode\_\_ ###

`Model.__unicode__()`

`__unicode__()` 方法在每当你对一个对象调用unicode() 时调用。Django 在许多地方都使用`unicode(obj)`（或者相关的函数 `str(obj)`）。最明显的是在Django 的Admin 站点显示一个对象和在模板中插入对象的值的时候。所以，你应该始终让`__unicode__()` 方法返回模型的一个友好的、人类可读的形式。

例如：

```
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)

    def __unicode__(self):
        return u'%s %s' % (self.first_name, self.last_name)
```

如果你定义了模型的`__unicode__()` 方法且没有定义`__str__()` 方法，Django 将自动提供一个 `__str__()`，它调用`__unicode__()` 并转换结果为一个UTF-8 编码的字符串。下面是一个建议的开发实践：只定义`__unicode__()` 并让Django 在需要时负责字符串的转换。

### \_\_str\_\_ ###

`Model.__str__()`

`__str__()` 方法在每当你对一个对象调用`str()` 时调用。在Python 3 中，Django 在许多地方使用`str(obj)`。 最明显的是在Django 的Admin 站点显示一个对象和在模板中插入对象的值的时候。 所以，你应该始终让`__str__()` 方法返回模型的一个友好的、人类可读的形式。

例如：

```
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)

    def __str__(self):
        return '%s %s' % (self.first_name, self.last_name)
```

在Python 2 中，Django 内部对`__str__` 的直接使用主要在随处可见的模型的`repr()` 输出中（例如，调试时的输出）。如果已经有合适的`__unicode__()` 方法就不需要`__str__()` 了。

前面`__unicode__()` 的示例可以使用`__str__()` 这样类似地编写：

```
from django.db import models
from django.utils.encoding import force_bytes

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)

    def __str__(self):
        # Note use of django.utils.encoding.force_bytes() here because
        # first_name and last_name will be unicode strings.
        return force_bytes('%s %s' % (self.first_name, self.last_name))
```

### \_\_eq\_\_ ###

`Model.__eq__()`

定义这个方法是为了让具有相同主键的相同实类的实例是相等的。对于代理模型，实类是模型第一个非代理父类；对于其它模型，它的实类就是模型类自己。

例如：

```
from django.db import models

class MyModel(models.Model):
    id = models.AutoField(primary_key=True)

class MyProxyModel(MyModel):
    class Meta:
        proxy = True

class MultitableInherited(MyModel):
    pass

MyModel(id=1) == MyModel(id=1)
MyModel(id=1) == MyProxyModel(id=1)
MyModel(id=1) != MultitableInherited(id=1)
MyModel(id=1) != MyModel(id=2)
```

```
Changed in Django 1.7:

在之前的版本中，只有类和主键都完全相同的实例才是相等的。
```

### \_\_hash\_\_ ###

`Model.__hash__()`

`__hash__` 方法基于实例主键的值。它等同于hash(obj.pk)。如果实例的主键还没有值，将引发一个`TypeError`（否则，`__hash__` 方法在实例保存的前后将返回不同的值，而改变一个实例的`__hash__` 值在Python 中是禁止的）。

```
Changed in Django 1.7:

在之前的版本中，主键没有值的实例是可以哈希的。
```

### get_absolute_url ###

`Model.get_absolute_url()`

`get_absolute_url()` 方法告诉Django 如何计算对象的标准URL。对于调用者，该方法返回的字符串应该可以通过HTTP 引用到这个对象。

例如：

```
def get_absolute_url(self):
    return "/people/%i/" % self.id
```

（虽然这段代码正确又简单，这并不是编写这个方法可移植性最好的方式。通常使用`reverse()` 函数是最好的方式。）

例如：

```
def get_absolute_url(self):
    from django.core.urlresolvers import reverse
    return reverse('people.views.details', args=[str(self.id)])
```

Django 使用`get_absolute_url()` 的一个地方是在Admin 应用中。如果对象定义该方法，对象编辑页面将具有一个“View on site”链接，可以将你直接导入由`get_absolute_url()` 提供的对象公开视图。

类似地，Django 的另外一些小功能，例如[syndication feed 框架](http://python.usyiyi.cn/django/ref/contrib/syndication.html) 也使用`get_absolute_url()`。 如果模型的每个实例都具有一个唯一的URL 是合理的，你应该定义`get_absolute_url()`。

> 警告
>
> 你应该避免从没有验证过的用户输入构建URL，以减少有害的链接和重定向：

```
def get_absolute_url(self):
    return '/%s/' % self.name
```

> 如果`self.name` 为'`/example.com`'，将返回 '`//example.com/`'， 而它是一个合法的相对URL而不是期望的'`/%2Fexample.com/`'。

在模板中使用`get_absolute_url()` 而不是硬编码对象的URL 是很好的实践。例如，下面的模板代码很糟糕：

```
<!-- BAD template code. Avoid! -->
<a href="/people/{{ object.id }}/">{{ object.name }}</a>
```

下面的模板代码要好多了：

```
<a href="{{ object.get_absolute_url }}">{{ object.name }}</a>
```

如果你改变了对象的URL 结构，即使是一些简单的拼写错误，你不需要检查每个可能创建该URL 的地方。在`get_absolute_url()` 中定义一次，然后在其它代码调用它。

> 注
>
> `get_absolute_url()` 返回的字符串必须只包含ASCII 字符（URI 规范[RFC 2396](http://tools.ietf.org/html/rfc2396.html) 的要求），并且如需要必须要URL-encoded。
>
> 代码和模板中对`get_absolute_url()` 的调用应该可以直接使用而不用做进一步处理。你可能想使用`django.utils.encoding.iri_to_uri()` 函数来帮助你解决这个问题，如果你正在使用ASCII 范围之外的Unicode 字符串。

## 额外的实例方法 ##

除了`save()`、`delete()`之外，模型的对象还可能具有以下一些方法：

`Model.get_FOO_display()`

对于每个具有`choices` 的字段，每个对象将具有一个`get_FOO_display()` 方法，其中`FOO` 为该字段的名称。这个方法返回该字段对“人类可读”的值。

例如：

```
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=2, choices=SHIRT_SIZES)
>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```

`Model.get_next_by_FOO(**kwargs)`

`Model.get_previous_by_FOO(**kwargs)`

如果`DateField` 和`DateTimeField`没有设置 `null=True`，那么该对象将具有`get_next_by_FOO()` 和`get_previous_by_FOO()` 方法，其中`FOO` 为字段的名称。它根据日期字段返回下一个和上一个对象，并适时引发一个`DoesNotExist`。

这两个方法都将使用模型默认的管理器来执行查询。如果你需要使用自定义的管理器或者你需要自定义的筛选，这个两个方法还接受可选的参数，它们应该用字段查询 中提到的格式。

注意，对于完全相同的日期，这些方法还将利用主键来进行查找。这保证不会有记录遗漏或重复。这还意味着你不可以在未保存的对象上使用这些方法。

## 其它属性 ##

### DoesNotExist ###

exception `Model.DoesNotExist`

ORM 在好几个地方会引发这个异常，例如`QuerySet.get()` 根据给定的查询参数找不到对象时。

Django 为每个类提供一个`DoesNotExist` 异常属性是为了区别找不到的对象所属的类，并让你可以利用`try/except`捕获一个特定模型的类。这个异常是`django.core.exceptions.ObjectDoesNotExist` 的子类。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Instance methods](https://docs.djangoproject.com/en/1.8/ref/models/instances/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
