<!--
  译者：WrongWay [www.wrongway.me]
  1.8更新：Github@wizardforcel
-->

# 模型 #

模型是有关你的数据的，简单、确定的信息源。它包含了你所储存数据的一些必要的字段和行为。通常来说，每个模型都对应数据库中的一张表。

基础：

+ 每个模型都是django.db.models.Model类的子类。
+ 模型的每个属性都表示数据库中的一个字段。
+ Django 会提供一套自动生成的用于数据库访问的API；详见执行查询。

## 简短的例子 ##

这个例子定义了一个Person模型，它有 first_name和last_name两个属性

```
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
```

first_name和last_name是模型的两个字段。每个字段都被指定成一个类属性，每个属性 都映射一个数据库的列。

上面的Person模型会在数据库中创建这样一张表:

```
CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY,
    "first_name" varchar(30) NOT NULL,
    "last_name" varchar(30) NOT NULL
);
```

一些技术上的注意事项:字段类型

这个表的名称myapp_person，是根据 模型中的元数据自动生成的，也可以覆写为别的名称，详见Table names。
id 字段是自动添加的，但这个行为可以被重写。详见Automatic primary key fields。
这个例子使用 PostgreSQL 语法格式化CREATE TABLESQL 语句，要注意的是 Django 是根据settings file配置中指定的数据库类型来生成相应的 SQL 语句。

## 使用模型 ##

一旦你定义了模型，就要通知Django启用这些模型，你要做的就是修改配置文件中的INSTALLED_APPS 设置，在其中添加models.py所在应用的名称。

例如，假设你的 model 定义在 mysite.myapp.models 中 ( mysite 这个包是由 manage.py startapp 脚本创建的)，那么 INSTALLED_APPS 就应该包含下面这行：

```
INSTALLED_APPS = (
    #...
    'mysite.myapp',
    #...
)
```

在 INSTALLED_APPS 中添加新应用之后，要运行 manage.py syncdb 同步数据库。

## 字段 ##

模型 中不可或缺且最为重要的，就是字段集，它是一组数据库字段的列表。字段被指定为类属性。要注意选择字段名称的时候不要和models API 冲突，比如clean, save, 或者delete。

例如：

```
class Musician(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    instrument = models.CharField(max_length=100)

class Album(models.Model):
    artist = models.ForeignKey(Musician)
    name = models.CharField(max_length=100)
    release_date = models.DateField()
    num_stars = models.IntegerField()
```

## 字段类型 ##

model 中的每个字段都是 Field 子类的某个实例。Django 根据字段类的类型确定以下信息：

数据库当中的列类型 (比如，INTEGER, VARCHAR)。
Django 的用户管理界面所使用的部件(widget)。当然，前提是你启用了 Django 的管理后台 (例如， `<input type="text">`， `<select>`)。
最低限度的验证需求。它被用在 Django 管理后台和自动生成的表单中。
Django 自带数十种内置的字段类型；详见 model 字段参考(model field reference)。如果内置类型仍不能满足你的要求，你可以自由地编写符合你要求的字段类型；详见 编写自定义 model 字段(Writing custom model fields)。

## 字段选项 ##

每个字段都有一些特有的参数，详见 model 字段参考(model field reference)。例如， CharField (还有它的派生类) 都需要 max_length 参数来指定存储数据的 VARCHAR 数据库字段的大小。

还有一些适用于所有字段的可选的通用参数，这些参数在 参考(reference) 中有详细定义，这里我们只简单介绍一些最常用的：

**null**

如果为 True， Django 在数据库中会将空值(empty)存储为 NULL 。 默认为 False。

**blank**

如果为 True，该字段允许不填(blank)。默认为 False。

要注意，这与 null 不同。 null 纯粹是数据库范畴的，而 blank 是数据验证范畴的。如果一个字段的 blank=True，Django 的管理后台在做数据验证时，会允许该字段是空值。如果字段的 blank=False，该字段就是必填的。

**choices**

它是一个可迭代的二元组(例如，列表或是元组)，用来给字段提供选择项。如果设置了 choices ，Django 的管理后台就会显示选择框，而不是标准的文本框，而且这个选择框的选项就是 choices 中的元组。

这是一个关于 choices 列表的例子：

```
YEAR_IN_SCHOOL_CHOICES = (
    (u'FR', u'Freshman'),
    (u'SO', u'Sophomore'),
    (u'JR', u'Junior'),
    (u'SR', u'Senior'),
    (u'GR', u'Graduate'),
)
```

每个元组中的第一个元素，是存储在数据库中的值；第二个元素是在管理界面或 ModelChoiceField 中用作显示的内容。在一个给定的 model 类的实例中，想得到某个 choices 字段的显示值，就调用 get_FOO_display 方法(这里的 FOO 就是 choices 字段的名称 )。例如：

```
from django.db import models

class Person(models.Model):
    GENDER_CHOICES = (
        (u'M', u'Male'),
        (u'F', u'Female'),
    )
    name = models.CharField(max_length=60)
    gender = models.CharField(max_length=2, choices=GENDER_CHOICES)
>>> p = Person(name="Fred Flinstone", gender="M")
>>> p.save()
>>> p.gender
u'M'
>>> p.get_gender_display()
u'Male'
```

**default**

字段的默认值。它可以是一个值，也可以是一个可调用的对象(这里称之为对象C)。若是后者，那么每次创建一个新对象时，对象C都将被调用。

**help_text**

附加的帮助信息。在管理后台编辑该对象的表单中，它显示在字段下面。即使你的对象无须在后台进行管理，它对于文档化也是很有用的。

**primary_key**

如果为 True，那么这个字段就是 model 的主键。

如果你没有指定任何一个字段的 primary_key=True，Django 就会自动添加一个 IntegerField 字段做为主键。所以除非你想重写默认的主键方法，否则没必要在任何字段上设置 primary_key=True 。详见 自增主键字段(Automatic primary key fields).

主键字段是只读的。如果你在一个已存在的对象上面更改主键的值并且保存，一个新的对象将会在原有对象之外创建出来。例如：

```
from django.db import models

class Fruit(models.Model):
    name = models.CharField(max_length=100, primary_key=True)

>>> fruit = Fruit.objects.create(name='Apple')
>>> fruit.name = 'Pear'
>>> fruit.save()
>>> Fruit.objects.values_list('name', flat=True)
['Apple', 'Pear']
```

**unique**

如果为 True，那么字段值就必须是全表唯一的。
再说一次，这些仅仅是常用字段的简短介绍，要了解详细内容，请查看 通用 model 字段选项参考(common model field option reference).

## 自增主键字段 ##

默认情况下，Django 会给每个 model 添加下面这个字段：

```
id = models.AutoField(primary_key=True)
```

这是一个自增主键字段。

如果你想指定一个自定义主键字段，只要在某个字段上指定 primary_key=True 即可。如果 Django 看到你显式地设置了 Field.primary_key，就不会自动添加 id 列。

每个 model 只要有一个字段指定 primary_key=True 就可以了。（无论是显式声明还是自动添加的。）

## 字段的自述名 ##

除了 ForeignKey, ManyToManyField 和 OneToOneField 之外，其余每个字段类型都接受一个排在首位的可选的位置参数--这就是字段的自述名。如果没有给定自述名，Django 将根据字段的属性名称自动创建自述名--就是将属性名称的空格替换成下划线。

在这个例子中，自述名是 "Person's first name"：

```
first_name = models.CharField("Person's first name", max_length=30)
```

在这个例子中，自述名是 "first name"：

```
first_name = models.CharField(max_length=30)
```

ForeignKey, ManyToManyField 和 OneToOneField 都要求排在首位的参数得是一个 model 类，所以要使用 verbose_name 关键字参数才能指定自述名：

```
poll = models.ForeignKey(Poll, verbose_name="the related poll")
sites = models.ManyToManyField(Site, verbose_name="list of sites")
place = models.OneToOneField(Place, verbose_name="related place")
```

verbose_name 首字母是不用大写的，这是因为 Django 在必要的时候会自动大写首字母的。

## 关系 ##

显然，关系数据库的威力体现在表之间的相互关联。Django 提供了三种最常见的数据库关系：多对一(many-to-one)，多对多(many-to-many)，一对一(one-to-one)。

### 多对一关系 ###

Django 使用 ForeignKey 定义多对一关系。 和使用其他 字段(Field) 类型一样：在 model 当中把它做为一个类属性包含进来。

ForeignKey 需要一个位置参数：与该 model 关联的类。

比如，如果每个 汽车(Car) model 都有一个 生产商(Manufacturer) model -- 也就是说，一个 Manufacturer 可以生产出很多 Car ；但是每一辆 Car 却只能有一个 Manufacturer -- 使用下面的定义：

```
class Manufacturer(models.Model):
    # ...

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer)
    # ...
```

你还可以创建 递归的关联关系(recursive relationships) (对象和自己进行多对一关联) 和 关联至尚未定义关系的 model (relationships to models not yet defined); 详见 model 字段参考(the model field reference) 。

建议你用被关联 model 的小写名称做为 ForeignKey 字段的命名 (上例中，我们就是以manufacturer 的小写做为命名的)。当然，你也可以起别的名字，例如：

```
class Car(models.Model):
    company_that_makes_it = models.ForeignKey(Manufacturer)
    # ...
```

> 另见
> 
> ForeignKey 字段还可以接受别的参数，它们都是可选的，在 model 字段参考(the model field reference) 有详细介绍。这些选项定义了关系是如何工作的。
> 
> 访问反向关联对象的细节，请见Following relationships backward example。
> 
> 示例代码请见多对一关系的模型例子( Many-to-one relationship model example)。

## 多对多关系 ##

ManyToManyField 用来定义多对多关系，用法和其他 Field 字段类型一样：在 model 中做为一个类属性包含进来。

ManyToManyField 需要一个位置参数：和该 model 关联的类。

例如，一个 匹萨(Pizza) 可以有多种不同口味的 浇头(Topping) -- 也就是说，一种 Topping 可以浇在多个 Pizza 上，而每个 Pizza 也可以浇上多种 topping -- 如下：

```
class Topping(models.Model):
    # ...

class Pizza(models.Model):
    # ...
    toppings = models.ManyToManyField(Topping)
```

和使用 ForeignKey 一样，你也可以创建 递归的关联关系(recursive relationships) (对象和自己做多对多关联)和 关联至尚未定义关系的 model (relationships to models not yet defined)；详见 the model field reference 。

建议你以被关联 model 名称的复数形式做为 ManyToManyField 的命名 (例如上例中的 toppings )。

在哪个 model 中设置 ManyToManyField 并不重要，在两个 model 中任选一个即可。

通常来说，如果启用了 Django 管理后台，你就可以在后台将 ManyToManyField 实例添加到关联对象中。在上面的例子中，在 Pizza 里面设置 toppings (而不是在 Topping 里面设置 pizzas ManyToManyField)。 这么设置的原因是因为一个 pizza 有多个 topping 相比于一个 topping 浇在多个 pizza 上要更加自然。这样，在 Pizza 的管理后台中，就会允许用户选择不同的 toppings。

> 另见
> 
> 在 多对多关系 model 实例(Many-to-many relationship model example) 有一个完整例子。

ManyToManyField 字段还可以接受别的参数，它们都是可选的，在 model 字段参考(the model field reference) 中有详细介绍。这些选项定义了关系是如何工作的。

### 多对多关系中的其他字段 ###

处理类似搭配 pizza 和 topping 这样简单的多对多关系时，使用标准的 ManyToManyField 就可以了。但是有时，我们需要在两个 model 之间关联其他的数据。

例如，有这样一个应用：关注某个音乐小组，它拥有多个音乐家成员。我们可以用一个标准的 ManyToManyField 表示小组和成员之间的多对多关系。但是，有时你可能想知道更多成员关系的细节，比如成员是何时加入小组的。

在这种情况下，Django 允许你指定一个 model 来定义多对多关系（我们称之为中介 model ）。你可以将其他字段放在中介 model 里面，而主 model 的 ManyToManyField 使用 through 参数来指向中介 model 。对于上面的音乐小组的例子来说，代码如下：

```
class Person(models.Model):
    name = models.CharField(max_length=128)

    def __unicode__(self):
        return self.name

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

    def __unicode__(self):
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person)
    group = models.ForeignKey(Group)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)
```

在设置中介 model 时，要显式地定义一个外键，它与包含多对多关系的 model 相关联。这个显式的声明定义了两个 model 之间中如何关联的。

在使用中介 model 时要注意以下限制：

+ 有且只有一个外键指向目标 model (例中目标 model 就是 Person )；否则就会抛出验证异常。
+ 有且只有一个外键指向源 model (例中源 model 就是 Group )；否则就会抛出验证异常。
+ 但存在唯一的一种特殊情况：利用中介 model 实现递归的多对多关系。这种情况下，两个外键指向同一个 model 是允许的；但这个 model 会被视为多对多关系中不同的双方进行处理。
+ 定义递归的多对多关系时，你必须设置 symmetrical=False (详见 model 字段参考(the model field reference))。

现在你已经设置了 ManyToManyField 来使用中介 model (在这个例子中就是 Membership)，接下来你要开始创建多对多关系。你要做的就是创建中介 model 的实例：

```
>>> ringo = Person.objects.create(name="Ringo Starr")
>>> paul = Person.objects.create(name="Paul McCartney")
>>> beatles = Group.objects.create(name="The Beatles")
>>> m1 = Membership(person=ringo, group=beatles,
...     date_joined=date(1962, 8, 16),
...     invite_reason= "Needed a new drummer.")
>>> m1.save()
>>> beatles.members.all()
[<Person: Ringo Starr>]
>>> ringo.group_set.all()
[<Group: The Beatles>]
>>> m2 = Membership.objects.create(person=paul, group=beatles,
...     date_joined=date(1960, 8, 1),
...     invite_reason= "Wanted to form a band.")
>>> beatles.members.all()
[<Person: Ringo Starr>, <Person: Paul McCartney>]
```

与普通的多对多字段不同，你不能使用 add, create, 和赋值语句 (比如，beatles.members = [...]) 来创建关系：

```
# THIS WILL NOT WORK
>>> beatles.members.add(john)
# NEITHER WILL THIS
>>> beatles.members.create(name="George Harrison")
# AND NEITHER WILL THIS
>>> beatles.members = [john, paul, ringo, george]
```

为什么不能这样做? 这是因为你不能只创建 Person and a Group 之间的关联关系，你还要指定 Membership model 中所需要的所有信息；而简单的 add, create 和赋值语句是做不到这一点的。所以它们不能在这种情况下使用。此时，唯一的办法就是创建中介 model 的实例。

remove 方法被禁用也是出于同样的原因。但是 clear() 方法却是可用的。它可以清空某个实例所有的多对多关系：

```
# Beatles have broken up
>>> beatles.members.clear()
```

在创建了中介 model 的实例，完成了对多对多关系的定义之后，你就可以执行查询了。和普通的多对多字段一样，你可以直接使用被关联 model 的属性进行查询：

```
# Find all the groups with a member whose name starts with 'Paul'
>>> Group.objects.filter(members__name__startswith='Paul')
[<Group: The Beatles>]
```

如果你使用了中介 model ，你也可以利用中介 model 的其他属性进行查询：

```
# Find all the members of the Beatles that joined after 1 Jan 1961
>>> Person.objects.filter(
...     group__name='The Beatles',
...     membership__date_joined__gt=date(1961,1,1))
[<Person: Ringo Starr]
```

### 一对一关系 ###

OneToOneField 用来定义一对一关系。用法和其他 Field 字段类型一样：在 model 里面做为类属性包含进来。

当某个对象想扩展自另一个对象时，最常用的方式就是在这个对象的主键上添加一对一关系。

OneToOneField 需要一个位置参数：与 model 关联的类。

例如，你想建一个 "places" 数据库，里面有一些常用的字段，比如 address, phone number, 等等。接下来，如果你想在 Place 数据库的基础上建立一个 饭店(Restaurant) 数据库，而不想将已有的字段复制到 Restaurant model ，那你可以在 Restaurant 添加一个 OneToOneField 字段，这个字段指向 Place (因为饭店(restaurant)本身就是一个地点(place)，事实上，在处理这个问题的时候，你已经使用了一个典型的 继承(inheritance)，它隐含了一个一对一关系)。

和使用 ForeignKey 一样，你可以定义 递归的关联关系(recursive relationship) 和 引用尚未定义关系的 model (references to as-yet undefined models) 。详见 model 字段参考(the model field reference) 。

> 参见
> 
> 在 一对一关系的 model 例子(One-to-one relationship model example) 有一套完整的例子。
> 
> 这部分是在 Django 1.0 中新增的： 请查看版本文档

OneToOneField 字段还有其他一些参数，它们都是可选的，在 model 字段参考(model field reference) 中有详细介绍。

在以前的版本中，OneToOneField 字段会自动变成 model 的主键。不过现在已经不这么做了(不过要是你愿意的话，你仍可以传递 primary_key 参数来创建主键字段)。所以一个 model 中可以有多个 OneToOneField 字段。

## 跨文件访问 model ##

访问其他应用的 model 是非常容易的。在使用 model 之前将它导入到当前程序即可。例如：

```
from mysite.geography.models import ZipCode

class Restaurant(models.Model):
    # ...
    zip_code = models.ForeignKey(ZipCode)
```

## 字段命名的限制 ##

Django 对字段的命名只有两个限制：

字段名不可以是 Python 的保留字，否则会导致 Python 语法错误。例如：

```
class Example(models.Model):
    pass = models.IntegerField() # 'pass' is a reserved word!
```

字段名称不可以包含连续多个下划线，因为这与 Django 查询时所用的筛选条件语法相冲突。例如：

```
class Example(models.Model):
    foo__bar = models.IntegerField() # 'foo__bar' has two underscores!
```

但是，只要你的字段名称与数据库中的列名不同，就可以绕过这些限制。详见 db_column 选项。

SQL 保留字，如 join, where 和 select, 可以做为 model 中字段的名称。这是因为 Django 会对每个 SQL 查询的数据库名称和列名称做重编码，至于如何编码视你所用的数据库而定。

## 自定义字段类型 ##

如果 Django 自带的字段类型不能满足你的应用，或者你希望使用一些不常见的数据库列类型，那你可以创建自定义的字段类型。详见 编写自定义 model 字段(Writing custom model fields)。

## Meta 选项 ##

通过使用一个内含的 class Meta 来为你的model 添加元数据，例如：

```
class Ox(models.Model):
    horn_length = models.IntegerField()

    class Meta:
        ordering = ["horn_length"]
        verbose_name_plural = "oxen"
```

在 model 里面，除了字段就是元数据，比如排序项(ordering)，数据库名称(db_table)，和自述名(verbose_name 和 verbose_name_plural)。对于 model 来说，这些都不是必需的，甚至就连 class Meta 本身都不是必需的。

Meta 选项的完整列表可以在 model 选项参考(model option reference) 中找到。

## Model 方法 ##

自定义 model 的方法，就是为你的对象添加自定义的行级功能(row-level)，而 Manager 方法却喜欢做表级的事情(table-wide)。所以，model 方法应该作用于 model 类的实例(也就是说，在实例对象上使用 model 方法，而不是在类上直接使用)。

最好是只在一个地方(就是在 model 中)保存商业逻辑。

例如，在下面这个 model 中自定义方法：

```
from django.contrib.localflavor.us.models import USStateField

class Person(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    birth_date = models.DateField()
    address = models.CharField(max_length=100)
    city = models.CharField(max_length=50)
    state = USStateField() # Yes, this is America-centric...

    def baby_boomer_status(self):
        "Returns the person's baby-boomer status."
        import datetime
        if datetime.date(1945, 8, 1) <= self.birth_date <= datetime.date(1964, 12, 31):
            return "Baby boomer"
        if self.birth_date < datetime.date(1945, 8, 1):
            return "Pre-boomer"
        return "Post-boomer"

    def is_midwestern(self):
        "Returns True if this person is from the Midwest."
        return self.state in ('IL', 'WI', 'MI', 'IN', 'OH', 'IA', 'MO')

    def _get_full_name(self):
        "Returns the person's full name."
        return '%s %s' % (self.first_name, self.last_name)
    full_name = property(_get_full_name)
```

本例中最后一个方法是一个 属性(property). 了解属性详见这里。

在 model 实例参考(model instance reference) 中一个完整的方法列表 自动添加到每个 model 中的方法(methods automatically given to each model)。 你可以重写里面的大部分方法 -- 详见下面的 重写已定义的 model 方法(overriding predefined model methods)，-- 但是有两个方法是经常要重写的：

**__unicode__()**

这是一个 Python 的魔术方法 ("magic method")，它返回对象的 Unicode 表示。当某个对象被要强制转换成字符串，或是要做为字符串显示时，Python 和 Django 就会调用该方法。最典型的，在命令行或管理后台中显示对象，就会用到 __unicode__() 方法。

你应该总是自定义这个方法；该方法默认的实现没有什么用。

**get_absolute_url()**

Django 使用这个方法算出某个对象的网址(URL)。Django 在管理后台和任何需要得到对象网址的地方使用该方法。

如果对象有一个唯一的网址，那么你就应该定义这个方法。

## 重写已定义的方法 ##

还有另外一组 model 方法(model methods) 封装了你想定制的数据库的操作。有些情况下，你可能经常会改变 save() 和 delete() 的实现。

你可以自由地重写这些方法 (以及任何其他的 model 方法) 来改变默认的实现。

一个典型的重写内置方法的案例就是：在你保存对象时，触发某些操作。例如 (详见 save() 的参数说明)：

```
class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, force_insert=False, force_update=False):
        do_something()
        super(Blog, self).save(force_insert, force_update) # Call the "real" save() method.
        do_something_else()
```

你也可以阻止保存：

```
class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, force_insert=False, force_update=False):
        if self.name == "Yoko Ono's blog":
            return # Yoko shall never have her own blog!
        else:
            super(Blog, self).save(force_insert, force_update) # Call the "real" save() method.
```

别忘记调用父类的方法，这很重要 -- 上例中的父类方法是 super(Blog, self).save() ，它要做的就是确保将对象保存到数据库。如果忘记调用父类的方法，默认的行为就不会发生，也就不会对数据库进行操作。

## 运行定制的 SQL ##

另外一种常见的模式就是在 model 方法或是模块级(module-level)的方法中使用定制的 SQL 语句。想了解使用原始 SQL 的更多细节，请查看 使用原始 SQL （using raw SQL） 。

## Model 继承 ##

这部分是在 Django 1.0 中新增的： 请注意版本文档
Django 中的 model 继承和 Python 中的类继承非常相似，只不过你要选择具体的实现方式：让父 model 拥有独立的数据库；还是让父 model 只包含基本的公共信息，由子 model 呈现公共信息。

在 Django 中有三种继承方式：

+ 通常，你只是想用父 model 来保存那些你不想在子 model 中重复录入的信息，父类并不单独使用。 抽象基类(Abstract base classes) 适用于这种情况。
+ 如果你继承了某个已有的 model (可能是直接从其他应用中拿来的)，并想让每个 model 都有自己的数据库。多表继承(Multi-table inheritance) 适用于这种情况。
+ 最后，如果你只想在 model 中修改 Python-level 级的行为，而不涉及字段改变。 代理 model (Proxy models) 适用于这种场合。

## 抽象基类 ##

如果你想把某些公共信息添加到很多 model 中，抽象基类就显得非常有用。你编写完基类之后，在 Meta 内嵌类中设置 abstract=True ，该类就不能创建任何数据表。然而如果将它做为其他 model 的基类，那么该类的字段就会被添加到子类中。抽象基类和子类如果含有同名字段，就会导致错误(Django 将抛出异常)。

举个例子：

```
class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```

学生(Student) model 会有三个字段： 姓名(name), 年龄(age) 和 分组(home_group)。 CommonInfo model 不能做为普通的 Django model 使用，因为它是一个抽象基类。他即不生成数据表，也没有 manager ，更不能直接被实例化和保存。

对很多应用来说，这种继承方式正是你想要的。它提供一种在 Python 语言层级上提取公共信息的方式，但在数据库层级上，各个子类仍然只创建一个数据库。

## Meta 继承 ##

创建抽象基类的时候，Django 会将你在基类中所声明的有效的 Meta 内嵌类做为一个属性。如果子类没有声明它自己的 Meta 内嵌类，它就会继承父类的 Meta 。子类的 Meta 也可以直接继承父类的 Meta 内嵌类，对其进行扩展。例如：

```
class CommonInfo(models.Model):
    ...
    class Meta:
        abstract = True
        ordering = ['name']

class Student(CommonInfo):
    ...
    class Meta(CommonInfo.Meta):
        db_table = 'student_info'
```

继承时，Django 会对基类的 Meta 内嵌类做一个调整：在安装 Meta 属性之前，Django 会设置 abstract=False。 这意味着抽象基类的子类不会自动变成抽象类。当然，你可以让一个抽象类继承另一个抽象基类，不过每次都要显式地设置 abstract=True 。

对于抽象基类而言，有些属性放在 Meta 内嵌类里面是没有意义的。例如，包含 db_table 将意味着所有的子类(是指那些没有指定自己的 Meta 内嵌类的子类)都使用同一张数据表，一般来说，这并不是我们想要的。

## 小心使用 related_name  ##

如果你在 ForeignKey 或 ManyToManyField 字段上使用 related_name 属性，你必须总是为该字段指定一个唯一的反向名称。但在抽象基类上这样做就会引发一个很严重的问题。因为 Django 会将基类字段添加到每个子类当中，而每个子类的字段属性值都完全相同 (这里面就包括 related_name)。注：这样每个子类的关联字段都会指向同一个字段。

当你在(且仅在)抽象基类中使用 related_name 时，如果想绕过这个问题，就要在属性值中包含 '%(class)s' 字符串。这个字符串会替换成字段所在子类的小写名称。因为每个子类的命名都不同,所以 related_name 也会不一样。例如：

```
class Base(models.Model):
    m2m = models.ManyToManyField(OtherModel, related_name="%(class)s_related")

    class Meta:
        abstract = True

class ChildA(Base):
    pass

class ChildB(Base):
    pass
```

ChildA.m2m 字段的反向名称是 childa_related，而 ChildB.m2m 字段的反向名称是 childb_related。这取决于你如何使用 '%(class)s' 来构造你的反向名称。如果你没有这样做，Django 就会在验证 model (或运行 syncdb) 时抛出错误。

如果你没有在抽象基类中为某个关联字段定义 related_name 属性，那么默认的反向名称就是子类名称加上 '_set'，它能否正常工作取决于你是否在子类中定义了同名字段。例如，在上面的代码中，如果去掉 related_name 属性，在 ChildA 中，m2m 字段的反向名称就是 childa_set；而 ChildB 的 m2m 字段的反向名称就是 childb_set 。

## 多表继承 ##

这是 Django 支持的第二种继承方式。使用这种继承方式时，同一层级下的每个子 model 都是一个真正意义上完整的 model 。每个子 model 都有专属的数据表，都可以查询和创建数据表。继承关系在子 model 和它的每个父类之间都添加一个链接 (通过一个自动创建的 OneToOneField 来实现)。 例如：

```
class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField()
    serves_pizza = models.BooleanField()
```

Place 里面的所有字段在 Restaurant 中也是有效的，只不过数据保存在另外一张数据表当中。所以下面两个语句都是可以运行的：

```
>>> Place.objects.filter(name="Bob's Cafe")
>>> Restaurant.objects.filter(name="Bob's Cafe")
```
如果你有一个 Place，那么它同时也是一个 Restaurant， 那么你可以使用子 model 的小写形式从 Place 对象中获得与其对应的 Restaurant 对象：

```
>>> p = Place.objects.filter(name="Bob's Cafe")
# If Bob's Cafe is a Restaurant object, this will give the child class:
>>> p.restaurant
<Restaurant: ...>
```

但是，如果上例中的 p 并不是 Restaurant (比如它仅仅只是 Place 对象，或者它是其他类的父类)，那么在引用 p.restaurant 就会抛开Restaurant.DoesNotExist 异常。

## 多表继承中的Meta ##

在多表继承中，子类继承父类的 Meta 内嵌类是没什么意见的。所有的 Meta 选项已经对父类起了作用，再次使用只会起反作用。(这与使用抽象基类的情况正好相反，因为抽象基类并没有属于它自己的内容)

所以子 model 并不能访问它父类的 Meta 内嵌类。但是在某些受限的情况下，子类可以从父类继承某些 Meta ：如果子类没有指定 django.db.models.Options.ordering 属性或 django.db.models.Options.get_latest_by 属性，它就会从父类中继承这些属性。

如果父类有了排序设置，而你并不想让子类有任何排序设置，你就可以显式地禁用排序：

```
class ChildModel(ParentModel):
    ...
    class Meta:
        # Remove parent's ordering effect
        ordering = []
```

## 继承与反向关联 ##

因为多表继承使用了一个隐含的 OneToOneField 来链接子类与父类，所以象上例那样，你可以用父类来指代子类。但是这个 OnetoOneField 字段默认的 related_name 值与 django.db.models.fields.ForeignKey 和 django.db.models.fields.ManyToManyField 默认的反向名称相同。如果你与其他 model 的子类做多对一或是多对多关系，你就必须在每个多对一和多对多字段上强制指定 related_name 。如果你没这么做，Django 就会在你运行 验证(validate) 或 同步数据库(syncdb) 时抛出异常。

例如，仍以上面 Place 类为例，我们创建一个带有 ManyToManyField 字段的子类：

```
class Supplier(Place):
    # Must specify related_name on all relations.
    customers = models.ManyToManyField(Restaurant, related_name='provider')
```

## 指定链接父类的字段 ##

之前我们提到，Django 会自动创建一个 OneToOneField 字段将子类链接至非抽象的父 model 。如果你想指定链接父类的属性名称，你可以创建你自己的 OneToOneField 字段并设置 parent_link=True ，从而使用该字段链接父类。

## 代理model  ##

这部分是在 Django 1.1 中新增的： 请查看版本文档
使用 多表继承(multi-table inheritance) 时，model 的每个子类都会创建一张新数据表，通常情况下，这正是我们想要的操作。这是因为子类需要一个空间来存储不包含在基类中的字段数据。但有时，你可能只想更改 model 在 Python 层的行为实现。比如：更改默认的 manager ，或是添加一个新方法。

而这，正是代理 model 继承方式要做的：为原始 model 创建一个代理(proxy)。你可以创建，删除，更新代理 model 的实例，而且所有的数据都可以象使用原始 model 一样被保存。不同之处在于：你可以在代理 model 中改变默认的排序设置和默认的 manager ，更不会对原始 model 产生影响。

声明代理 model 和声明普通 model 没有什么不同。设置Meta 内置类中 proxy 的值为 True，就完成了对代理 model 的声明。

举个例子，假设你想给 Django 自带的标准 User model (它被用在你的模板中)添加一个方法：

```
from django.contrib.auth.models import User

class MyUser(User):
    class Meta:
        proxy = True

    def do_something(self):
        ...
```

MyUser 类和它的父类 User 操作同一个数据表。特别的是，User 的任何实例也可以通过 MyUser 访问，反之亦然：

```
>>> u = User.objects.create(username="foobar")
>>> MyUser.objects.get(username="foobar")
<MyUser: foobar>
```

你也可以使用代理 model 给 model 定义不同的默认排序设置。Django 自带的 User model 没有定义排序设置(这是故意为之，是因为排序开销极大，我们不想在获取用户时浪费额外资源)。你可以利用代理对 username 属性进行排序，这很简单：

```
class OrderedUser(User):
    class Meta:
        ordering = ["username"]
        proxy = True
```

普通的 User 查询，其结果是无序的；而 OrderedUser 查询的结果是按 username 排序。

查询集只返回请求时所使用的 model (Querysets still return the model that was requested)

无论你何时查询 User 对象，Django 都不会返回 MyUser 对象。针对 User 对象的查询集只返回 User 对象。代理对象的精要就在于依赖原始 User 的代码仅对它自己有效，而你自己的代码就使用你扩展的内容。不管你怎么改动，都不会在查询 User 时得到 MyUser。

## 基类的限制 ##

代理 model 必须继承自一个非抽象基类。你不能继承自多个非抽象基类，这是因为一个代理 model 不能连接不同的数据表。代理 model 也可以继承任意多个抽象基类，但前提是它们没有定义任何 model 字段。

代理 model 从非抽象基类中继承那些未在代理 model 定义的 Meta 选项。

## 代理 model 的 manager ##

如果你没有在代理 model 中定义任何 manager ，代理 model 就会从父类中继承 manager 。如果你在代理 model 中定义了一个 manager ，它就会变成默认的 manager ，不过定义在父类中的 manager 仍是有效的。

继续上面的例子，你可以改变默认 manager，例如：

```
class NewManager(models.Manager):
    ...

class MyUser(User):
    objects = NewManager()

    class Meta:
        proxy = True
```

如果你想给代理添加一个新的 manager ，却不想替换已有的默认 manager ，那么你可以参考 自定义 manager (custom manager) 中提到的方法：创建一个包含新 manager 的基类，然后放在主基类后面继承：

```
# Create an abstract class for the new manager.
class ExtraManagers(models.Model):
    secondary = NewManager()

    class Meta:
        abstract = True

class MyUser(User, ExtraManagers):
    class Meta:
        proxy = True
```

你可能不需要经常这样做，但这样做是可行的。

## 代理 model 与非托管 model 之间的差异 ##

代理 model 继承看上去和使用 Meta 内嵌类中的 managed 属性的非托管 model 非常相似。但两者并不相同，你应当考虑选用哪种方案。

一个不同之处是你可以在 Meta.managed=False 的 model 中定义字段(事实上，是必须指定，除非你真的想得到一个空 model )。在创建非托管 model 时要谨慎设置 Meta.db_table ，这是因为创建的非托管 model 映射某个已存在的 model ，并且有自己的方法。因此，如果你要保证这两个 model 同步并对程序进行改动，那么就会变得繁冗而脆弱。

另一个不同之处是两者对 manager 的处理方式不同。这对于代理 model 非常重要。代理 model 要与它所代理的 model 行为相似，所以代理 model 要继承父 model 的 managers ，包括它的默认 manager 。但在普通的多表继承中，子类不能继承父类的 manager ，这是因为在处理非基类字段时，父类的 manager 未必适用。在 manager documentation 有详细介绍。

我们实现了这两种特性(Meta.proxy和Meta.unmanaged)之后，曾尝试把两者结合到一起。结果证明，宏观的继承关系和微观的 manager 揉在一起，不仅导致 API 复杂难用，而且还难以理解。由于任何场合下都可能需要这两个选项，所以目前二者仍是各自独立使用的。

所以，一般规则是：

如果你要镜像一个已有的 model 或数据表，且不想涉及所有的原始数据表的列，那就令 Meta.managed=False。通常情况下，对数据库视图创建 model 或是数据表不需要由 Django 控制时，就使用这个选项。
如果你想对 model 做 Python 层级的改动，又想保留字段不变，那就令 Meta.proxy=True。因此在数据保存时，代理 model 相当于完全复制了原始 model 的存储结构。
多重继承(Multiple inheritance)

和 Python 一样，Django 的 model 也可以做多重继承。这里要记住 Python 的名称解析规则。如果某个特定名称 (例如，Meta) 出现在第一个基类当中，那么子类就会使用第一个基类的该特定名称。例如，如果多重父类都包含 Meta 内嵌类，只有第一个基类的 Meta 才会被使用，其他的都被会忽略。

一般来说，你没必要使用多重继承。多重继承常见是用在 "mix-in" (对Mixin不了解的，请参阅赖勇浩的文章http://blog.csdn.net/lanphaday/archive/2007/06/18/1656969.aspx)：给继承自 mix-in的每个类添加某个特定的字段或方法。尽可能让继承结构简单直接，这样你就不必关注特定信息的来源。(注：这是说你不必花精力去穷究某个字段，属性，方法是从哪个父类继承的)

## 不允许"隐藏"字段 ##

普通的 Python 类继承允许子类覆盖父类的任何属性。但在 Django 中，重写 Field 实例是不允许的(至少现在还不行)。如果基类中有一个 author 字段，你就不能在子类中创建任何名为 author 的字段。

重写父类的字段会导致很多麻烦，比如：初始化实例(指定在 Model.__init__ 中被实例化的字段) 和序列化。而普通的 Python 类继承机制并不能处理好这些特性。所以 Django 的继承机制被设计成与 Python 有所不同，这样做并不是随意而为的。

这些限制仅仅针对做为属性使用的 Field 实例，并不是针对 Python 属性，Python 属性仍是可以被重写的。在 Python 看来，上面的限制仅仅针对字段实例的名称：如果你手动指定了数据库的列名称，那么在多重继承中，你就可以在子类和某个祖先类当中使用同一个列名称。(因为它们使用的是两个不同数据表的字段)。

如果你在任何一个祖先类中重写了某个 model 字段，Django 都会抛出 FieldError 异常。