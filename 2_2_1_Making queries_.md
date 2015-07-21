<!--
  译者：Github@wizardforcel
-->

# 执行查询 #

一但你建立好*数据模型*之后，django会自动生成一套数据库抽象的API，可以让你执行增删改查的操作。这篇文档阐述了如何使用这些API。关于所有模型检索选项的详细内容，请见*[数据模型参考](https://docs.djangoproject.com/en/1.8/ref/models/)*。

在整个文档（以及参考）中，我们会大量使用下面的模型，它构成了一个博客应用。

```
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=50)
    email = models.EmailField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):              # __unicode__ on Python 2
        return self.headline
```

## 创建对象 ##

为了把数据库表中的数据表示成python对象，django使用一种直观的方式：一个模型类代表数据库的一个表，一个模型的实例代表数据库表中的一条特定的记录。

使用关键词参数实例化一个对象来创建它，然后调用**save()**把它保存到数据库中。

假设模型存放于文件**mysite/blog/models.py**中，下面是一个例子：

```
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

上面的代码在背后执行了sql的**INSERT**操作。在你显式调用**save()**之前，django不会访问数据库。

**save()**方法没有返回值。

> 请参见
> 
> **save()**方法带有一些高级选项，它们没有在这里给出，完整的细节请见**save()**文档。
> 
> 如果你想只用一条语句创建并保存一个对象，使用**create()**方法。

## 保存对象的改动 ##

调用**save()**方法，来保存已经存在于数据库中的对象的改动。

假设一个**Blog**的实例**b5**已经被保存在数据库中，这个例子更改了它的名字，并且在数据库中更新它的记录：

```
>>> b5.name = 'New name'
>>> b5.save()
```

上面的代码在背后执行了sql的**UPDATE**操作。在你显式调用**save()**之前，django不会访问数据库。

### 保存**ForeignKey**和**ManyToManyField**字段 ###

更新**ForeignKey**字段的方式和保存普通字段相同--只是简单地把一个类型正确的对象赋值到字段中。下面的例子更新了**Entry**类的实例**entry**的**blog**属性，假设**Entry**的一个合适的实例以及**Blog**已经保存在数据库中（我们可以像下面那样获取他们）：

```
>>> from blog.models import Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```

更新**ManyToManyField**的方式有一些不同--使用字段的**add()**方法来增加关系的记录。这个例子向**entry**对象添加**Author**类的实例**joe**：

```
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

为了在一条语句中，向**ManyToManyField**添加多条记录，可以在调用**add()**方法时传入多个参数，像这样：

```
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

Django将会在你添加错误类型的对象时抛出异常。

## 获取对象 ##

通过模型中的**Manager**构造一个**QuertSet**，来从你的数据库中获取对象。

**QuerySet**表示你数据库中取出来的一个对象的集合。它可以含有零个、一个或者多个过滤器，过滤器根据所给的参数限制查询结果的范围。在sql的角度，**QuerySet**和**SELECT**命令等价，过滤器是像**WHERE**和**LIMIT**一样的限制子句。

你可以从模型的**Manager**那里取得**QuerySet**。每个模型都至少有一个**Manager**，它通常命名为**objects**。通过模型类直接访问它，像这样：

```
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```

> **注意**
> 
> 管理器通常只可以通过模型类来访问，不可以通过模型实例来访问。这是为了强制区分表级别和记录级别的操作。

对于一个模型来说，**Manager**是**QuerySet**的主要来源。例如，** Blog.objects.all() **会返回持有数据库中所有**Blog**对象的一个**QuerySet**。

### 获取所有对象 ###

获取一个表中所有对象的最简单的方式是全部获取。使用**Manager**的**all()**方法：

```
>>> all_entries = Entry.objects.all()
```

**all()**方法返回包含数据库中所有对象的**QuerySet**。

### 使用过滤器获取特定对象 ###

**all()**方法返回的结果集中包含全部对象，但是更普遍的情况是你需要获取完整集合的一个子集。

要创建这样一个子集，需要精炼上面的结果集，增加一些过滤器作为条件。两个最普遍的途径是：

**filter(\*\*kwargs)**
返回一个包含对象的集合，它们满足参数中所给的条件。

**exclude(\*\*kwargs)**
返回一个包含对象的集合，它们***不***满足参数中所给的条件。

查询参数（上面函数定义中的**\*\*kwargs**）需要满足特定的格式，*字段检索*一节中会提到。

举个例子，要获取年份为2006的所有文章的结果集，可以这样使用**filter()**方法：

```
Entry.objects.filter(pub_date__year=2006)
```

在默认的管理器类中，它相当于：

```
Entry.objects.all().filter(pub_date__year=2006)
```

#### 链式过滤 ####

**QuerySet**的精炼结果还是**QuerySet**，所以你可以把精炼用的语句组合到一起，像这样：

```
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime(2005, 1, 30)
... )
```

最开始的**QuerySet**包含数据库中的所有对象，之后增加一个过滤器去掉一部分，在之后又是另外一个过滤器。最后的结果的一个**QuerySet**，包含所有标题以”word“开头的记录，并且日期是2005年一月，日为当天的值。

#### 过滤后的结果集是独立的 ####

每次你精炼一个结果集，得到的都是全新的另一个结果集，它和之前的结果集之间没有任何绑定关系。每次精炼都会创建一个独立的结果集，可以被存储及反复使用。

例如：

```
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```

