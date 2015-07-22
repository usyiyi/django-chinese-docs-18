<!--
  译者：Github@wizardforcel
-->

# 管理器 #

**class Manager**

管理器是一个接口，数据库查询操作通过它提供给django的模型。django应用的每个模型至少拥有一个 管理器。

管理器类的工作方式在 执行查询文档中阐述，而这篇文档涉及了自定义管理器行为的模型选项。

## 管理器的名字 ##

通常，django为每个模型类添加一个名为objects的管理器。然而，如果你想将objects用于字段名称，或者你想使用其它名称而不是objects访问管理器，你可以在每个模型类中重命名它。在模型中定义一个值为models.Manager()的属性，来重命名管理器。例如：

```
from django.db import models

class Person(models.Model):
    #...
    people = models.Manager()
```

使用例子中的模型， Person.objects会抛出AttributeError异常，而Person.people.all()会返回一个包含所有Person对象的列表。

## 自定义管理器 ##

在一个特定的模型中，你可以通过继承管理器类来构建一个自定义的管理器，以及实例化你的自定义管理器。

你有两个原因可能会自己定义管理器：向器类中添加额外的方法，或者修改管理器最初返回的查询集。

### 添加额外的管理器方法 ###

为你的模型添加表级(table-level)功能时，采用添加额外的管理器方法是更好的处理方式。如果要添加行级功能－－就是说该功能只对某个模型的实例对象起作用。在这种情况下，使用 模型方法 比使用自定义的管理器方法要更好。）

自定义的管理器 方法可以返回你想要的任何数据，而不只是查询集。

例如，下面这个自定义的 管理器提供了一个 with_counts() 方法，它返回所有 OpinionPoll 对象的列表，而且列表中的每个对象都多了一个名为 num_responses的属性，这个属性保存一个聚合查询(COUNT*)的结果：

```
from django.db import models

class PollManager(models.Manager):
    def with_counts(self):
        from django.db import connection
        cursor = connection.cursor()
        cursor.execute("""
            SELECT p.id, p.question, p.poll_date, COUNT(*)
            FROM polls_opinionpoll p, polls_response r
            WHERE p.id = r.poll_id
            GROUP BY p.id, p.question, p.poll_date
            ORDER BY p.poll_date DESC""")
        result_list = []
        for row in cursor.fetchall():
            p = self.model(id=row[0], question=row[1], poll_date=row[2])
            p.num_responses = row[3]
            result_list.append(p)
        return result_list

class OpinionPoll(models.Model):
    question = models.CharField(max_length=200)
    poll_date = models.DateField()
    objects = PollManager()

class Response(models.Model):
    poll = models.ForeignKey(OpinionPoll)
    person_name = models.CharField(max_length=50)
    response = models.TextField()
```

在这个例子中，你已经可以使用 OpinionPoll.objects.with_counts()  得到所有含有  num_responses属性的 OpinionPoll对象。

这个例子要注意的一点是： 管理器方法可以访问 self.model来得到它所用到的模型类。

### 修改管理器初始的查询集 ###

管理器自带的 查询集返回系统中所有的对象。例如，使用下面这个模型：

```
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=50)
```

... Book.objects.all() 语句将返回数据库中所有的 Book 对象。

你可以通过重写 Manager.get_queryset() 的方法来覆盖 管理器自带的 查询集。get_queryset() 会根据你所需要的属性返回  查询集。

例如，下面的模型有两个  管理器，一个返回所有的对象，另一个则只返回作者是 Roald Dahl 的对象：

```
# First, define the Manager subclass.
class DahlBookManager(models.Manager):
    def get_queryset(self):
        return super(DahlBookManager, self).get_queryset().filter(author='Roald Dahl')

# Then hook it into the Book model explicitly.
class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=50)

    objects = models.Manager() # The default manager.
    dahl_objects = DahlBookManager() # The Dahl-specific manager.
```

在这个简单的例子中，Book.objects.all()将返回数据库中所有的图书。而 Book.dahl_objects.all() 只返回作者是 Roald Dahl 的图书。

由于 get_queryset() 返回的是一个  查询集 对象，所以你仍可以对它使用  filter(), exclude()和其他 查询集的方法。所以下面这些例子都是可用的：

```
Book.dahl_objects.all()
Book.dahl_objects.filter(title='Matilda')
Book.dahl_objects.count()
```

这个例子还展示了另外一个很有意思的技巧：在同一个模型中使用多个管理器。你可以随你所意在一个模型里面添加多个 Manager() 实例。下面就用很简单的方法，给模型添加通用过滤器：

例如：

```
class AuthorManager(models.Manager):
    def get_queryset(self):
        return super(AuthorManager, self).get_queryset().filter(role='A')

class EditorManager(models.Manager):
    def get_queryset(self):
        return super(EditorManager, self).get_queryset().filter(role='E')

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    role = models.CharField(max_length=1, choices=(('A', _('Author')), ('E', _('Editor'))))
    people = models.Manager()
    authors = AuthorManager()
    editors = EditorManager()
```

在这个例子中，你使用 Person.authors.all(), Person.editors.all(),以及 Person.people.all(), 都会得到和名称相符的结果。

#### 默认管理器 ####

如果你使用了自定义  管理器对象，要注意 Django 中的第一个 管理器 (按照模型中出现的顺序而定) 拥有特殊的地位。Django 会将模型中定义的管理器解释为默认的 管理器，并且 Django 中的一部分应用(包括数据备份)会使用默认的管理器，除了前面那个模型。因此，要决定默认的管理器时，要小心谨慎，仔细考量，这样才能避免重写 get_queryset()  导致无法正确地获得数据。

#### 使用管理器访问关联对象 ####

默认情况下，在访问相关对象时（例如choice.poll），Django 并不使用相关对象的默认管理器，而是使用一个"朴素"管理器类的实例来访问。这是因为 Django 要能从关联对象中获得数据，但这些数据有可能被默认管理器过滤掉，或是无法进行访问。

如果普通的朴素管理器类(django.db.models.Manager)并不适用于你的应用，那么你可以通过在管理器类中设置 use_for_related_fields ，强制 Django 在你的模型中使用默认的管理器。这部分内容在 下面有 详细介绍。

### 调用自定义的查询集 ###

虽然大多数标准查询集的方法可以从管理器中直接访问到，但是这是一个例子，访问了定义在自定义 查询集上的额外方法，如果你也在管理器上面实现了它们：

```
class PersonQuerySet(models.QuerySet):
    def authors(self):
        return self.filter(role='A')

    def editors(self):
        return self.filter(role='E')

class PersonManager(models.Manager):
    def get_queryset(self):
        return PersonQuerySet(self.model, using=self._db)

    def authors(self):
        return self.get_queryset().authors()

    def editors(self):
        return self.get_queryset().editors()

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    role = models.CharField(max_length=1, choices=(('A', _('Author')), ('E', _('Editor'))))
    people = PersonManager()
```

这个例子展示了如何直接从管理器 Person.people调用authors() 和 editors()。

### 创建管理器 ###

** django 1.7 中新增 **

对于上面的例子，同一个方法需要在查询集 和 管理器上创建两份副本，作为替代，QuerySet.as_manager()可以创建一个管理器的实例，它拥有自定义查询集的方法：

```
class Person(models.Model):
    ...
    people = PersonQuerySet.as_manager()
```

通过QuerySet.as_manager()创建的管理器 实例，实际上等价于上面例子中的PersonManager。

并不是每个查询集的方法都在管理器层面上有意义。比如 QuerySet.delete()，我们有意防止它复制到管理器 中。

方法按照以下规则进行复制：

+ 公共方法默认被复制。
+ 私有方法（前面带一个下划线）默认不被复制。
+ 带queryset_only 属性，并且值为False的方法总是被复制。
+ 带 queryset_only 属性，并且值为True 的方法不会被复制。

例如：

```
class CustomQuerySet(models.QuerySet):
    # Available on both Manager and QuerySet.
    def public_method(self):
        return

    # Available only on QuerySet.
    def _private_method(self):
        return

    # Available only on QuerySet.
    def opted_out_public_method(self):
        return
    opted_out_public_method.queryset_only = True

    # Available on both Manager and QuerySet.
    def _opted_in_private_method(self):
        return
    _opted_in_private_method.queryset_only = False
```

#### from_queryset ####

```
classmethod from_queryset(queryset_class)
```

在进一步的使用中，你可能想创建一个自定义管理器和一个自定义查询集。你可以调用Manager.from_queryset()，它会返回管理器的一个子类，带有自定义查询集所有方法的副本：

```
class BaseManager(models.Manager):
    def manager_only_method(self):
        return

class CustomQuerySet(models.QuerySet):
    def manager_and_queryset_method(self):
        return

class MyModel(models.Model):
    objects = BaseManager.from_queryset(CustomQueryset)()
```

你也可以在一个变量中储存生成的类：

```
CustomManager = BaseManager.from_queryset(CustomQueryset)

class MyModel(models.Model):
    objects = CustomManager()
```

### 自定义管理器和模型继承 ###

类继承和模型管理器两者之间配合得并不是很好。  管理器一般只对其定义所在的类起作用，在子类中对其继承绝对不是一个好主意。 而且，因为第一个 管理器会被 Djange 声明为默认的管理器，所以对默认的管理器 进行控制是非常必要的。下面就是 Django 如何处理自定义管理器和模型继承(model inheritance)的：

+ 定义在非抽象基类中的管理器是 不会 被子类继承的。如果你想从一个非抽象基类中重用管理器，只能在子类中重定义管理器。 这是因为这种管理器与定义它的模型 绑定得非常紧密，所以继承它们经常会导致异常的结果（特别是默认管理器运行的时候）。 因此，它们不应继承给子类。
+ 定义在抽象基类中的管理器总是被子类继续的，是按 Python 的命名解析顺序解析的（首先是子类中的命名覆盖所有的，然后是第一个父类的，以此类推）。 抽象类用来提取子类中的公共信息和行为，定义公共管理器也是公共信息的一部分。
+ 如果类当中显示定义了默认管理器，Django 就会以此做为默认管理器；否则就会从第一个抽象基类中继承默认管理器； 如果没有显式声明默认管理器，那么 Django 就会自动添加默认管理器。

如果你想在一组模型上安装一系列自定义管理器，上面提到的这些规则就已经为你的实现提供了必要的灵活性。你可以继承一个抽象基类，但仍要自定义默认的管理器。例如，假设你的基类是这样的：

```
class AbstractBase(models.Model):
    # ...
    objects = CustomManager()

    class Meta:
        abstract = True
```

如果你在基类中没有定义管理器，直接使用上面的代码，默认管理器就是从基类中继承的 objects：

```
class ChildA(AbstractBase):
    # ...
    # This class has CustomManager as the default manager.
    pass
```

如果你想从 AbstractBase继承，却又想提供另一个默认管理器，那么你可以在子类中定义默认管理器：

```
class ChildB(AbstractBase):
    # ...
    # An explicit default manager.
    default_manager = OtherManager()
```

在这个例子中， default_manager就是默认的 管理器。从基类中继承的 objects 管理器仍是可用的。只不过它不再是默认管理器罢了。

最后再举个例子，假设你想在子类中再添加一个额外的管理器，但是很想使用从 AbstractBase继承的管理器做为默认管理器。那么，你不在直接在子类中添加新的管理器，否则就会覆盖掉默认管理器，而且你必须对派生自这个基类的所有子类都显示指定管理器。 解决办法就是在另一个基类中添加新的管理器，然后继承时将其放在默认管理器所在的基类 之后。例如：

```
class ExtraManager(models.Model):
    extra_manager = OtherManager()

    class Meta:
        abstract = True

class ChildC(AbstractBase, ExtraManager):
    # ...
    # Default manager is CustomManager, but OtherManager is
    # also available via the "extra_manager" attribute.
    pass
```

注意在抽象模型上面定义一个自定义管理器的时候，不能调用任何使用这个抽象模型的方法。就像：

```
ClassA.objects.do_something()
```

是可以的，但是：

```
AbstractBase.objects.do_something()
```

会抛出一个异常。这是因为，管理器被设计用来封装对象集合管理的逻辑。由于抽象的对象中并没有一个集合，管理它们是毫无意义的。如果你写了应用在抽象模型上的功能，你应该把功能放到抽象模型的静态方法，或者类的方法中。

### 实现上的注意事项 ###

无论你向自定义管理器中添加了什么功能，都必须可以得到 管理器实例的一个浅表副本：例如，下面的代码必须正常运行：

```
>>> import copy
>>> manager = MyManager()
>>> my_copy = copy.copy(manager)
```

Django 在一些查询中会创建管理器的浅表副本；如果你的管理器不能被复制，查询就会失败。

这对于大多数自定义管理器不是什么大问题。如果你只是添加一些简单的方法到你的管理器中，不太可能会把你的管理器实例变为不可复制的。但是，如果你覆盖了\_\_getattr\_\_，或者其它管理器中控制对象状态的私有方法，你应该确保不会影响到管理器的复制。

## 控制自动管理器的类型 ##

这篇文档已经提到了Django创建管理器类的一些位置：默认管理器和用于访问关联对象的“朴素” 管理器。在 Django 的实现中也有很多地方用到了临时的朴素管理器。 正常情况下，django.db.models.Manager 类的实例会自动创建管理器。

在整个这一节中，我们将那种由 Django 为你创建的管理器称之为 "自动管理器"，既有因为没有管理器而被 Django 自动添加的默认管理器， 也包括在访问关联模型时使用的临时管理器。

有时，默认管理器也并非是自动管理器。 一个例子就是 Django 自带的django.contrib.gis 应用，所有 gis模型都必须使用一个特殊的管理器类(GeoManager)，因为它们需要运行特殊的查询集(GeoQuerySet)与数据库进行交互。这表明无论自动管理器是否被创建，那些要使用特殊的管理器的模型仍要使用这个特殊的管理器类。

Django 为自定义管理器的开发者提供了一种方式：无论开发的管理器类是不是默认的管理器，它都应该可以用做自动管理器。 可以通过在管理器类中设置 use_for_related_fields 属性来做到这点：

```
class MyManager(models.Manager):
    use_for_related_fields = True
    # ...
```

如果在模型中的默认 管理器(在这些情况中仅考虑默认管理器)中设置了这个属性，那么无论它是否需要被自动创建，Django 都会自动使用它。否则 Django 就会使用 django.db.models.Manager.

> 历史回顾
> 
> 从它使用目的来看，这个属性的名称(use_for_related_fields)好象有点古怪。原本，这个属性仅仅是用来控制访问关联字段的管理器的类型，这就是它名字的由来。 后来它的作用更加拓宽了，但是名称一直未变。 因为要保证现在的代码在 Django 以后的版本中仍可以正常工作(continue to work)，这就是它名称不变的原因。

### 在自动管理器实例中编写正确的管理器 ###

在上面的django.contrib.gis 已经提到了， use_for_related_fields这个特性是在需要返回一个自定义查询集子类的管理器中使用的。要在你的管理器中提供这个功能，要注意以下几点。

#### 不要在这种类型的管理器子类中过滤掉任何结果 ####

一个原因是自动管理器是用来访问关联模型 的对象。 在这种情况下，Django 必须要能看到相关模型的所有对象，所以才能根据关联关系得到任何数据 。

如果你重写了 get_queryset() 方法并且过滤掉了一些行数据，Django 将返回不正确的结果。不要这么做！ 在 get_queryset()方法中过滤掉数据，会使得它所在的管理器不适于用做自动管理器。

#### 设置 use_for_related_fields ####

use_for_related_fields属性必须在管理器类中设置，而不是在类的 实例中设置。上面已经有例子展示如何正确地设置，下面这个例子就是一个错误的示范：

```
# BAD: Incorrect code
class MyManager(models.Manager):
    # ...
    pass

# Sets the attribute on an instance of MyManager. Django will
# ignore this setting.
mgr = MyManager()
mgr.use_for_related_fields = True

class MyModel(models.Model):
    # ...
    objects = mgr

# End of incorrect code.
```

你也不应该在模型中使用这个属性之后，在类上改变它。这是因为在模型类被创建时，这个属性值马上就会被处理，而且随后不会再读取这个属性值。 这节的第一个例子就是在第一次定义的时候在管理器上设置use_for_related_fields属性，所有的代码就工作得很好。