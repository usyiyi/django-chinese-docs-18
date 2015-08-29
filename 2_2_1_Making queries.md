# 执行查询 #

一旦你建立好*数据模型*之后，django会自动生成一套数据库抽象的API，可以让你执行增删改查的操作。这篇文档阐述了如何使用这些API。关于所有模型检索选项的详细内容，请见*[数据模型参考](https://docs.djangoproject.com/en/1.8/ref/models/)*。

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

每次你筛选一个结果集，得到的都是全新的另一个结果集，它和之前的结果集之间没有任何绑定关系。每次筛选都会创建一个独立的结果集，可以被存储及反复使用。

例如：

```
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```

这三个 QuerySets 是不同的。 第一个 QuerySet 包含大标题以"What"开头的所有记录。第二个则是第一个的子集，用一个附加的条件排除了出版日期 pub_date 是今天的记录。 第三个也是第一个的子集，它只保留出版日期 pub_date 是今天的记录。 最初的 QuerySet (q1) 没有受到筛选的影响。

## 查询集是延迟的 ##

QuerySets 是惰性的 -- 创建 QuerySet 的动作不涉及任何数据库操作。你可以一直添加过滤器，在这个过程中，Django 不会执行任何数据库查询，除非 QuerySet 被执行. 看看下面这个例子:

```
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.now())
>>> q = q.exclude(body_text__icontains="food")
>>> print q
```

虽然上面的代码看上去象是三个数据库操作，但实际上只在最后一行 (print q) 执行了一次数据库操作，。一般情况下， QuerySet 不能从数据库中主动地获得数据，得被动地由你来请求。对 QuerySet 求值就意味着 Django 会访问数据库。想了解对查询集何时求值，请查看 何时对查询集求值 (When QuerySets are evaluated).

## 其他查询集方法  ##

大多数情况使用 all(), filter() 和 exclude() 就足够了。 但也有一些不常用的；请查看 查询API参考 (QuerySet API Reference) 中完整的 QuerySet 方法列表。

## 限制查询集范围 ##

可以用 python 的数组切片语法来限制你的 QuerySet 以得到一部分结果。它等价于SQL中的 LIMIT 和 OFFSET 。

例如，下面的这个例子返回前五个对象 (LIMIT 5):

```
>>> Entry.objects.all()[:5]
```

这个例子返回第六到第十之间的对象 (OFFSET 5 LIMIT 5):

```
>>> Entry.objects.all()[5:10]
```

Django 不支持对查询集做负数索引 (例如 Entry.objects.all()[-1]) 。

一般来说，对 QuerySet 切片会返回新的 QuerySet -- 这个过程中不会对运行查询。不过也有例外，如果你在切片时使用了 "step" 参数，查询集就会被求值，就在数据库中运行查询。举个例子，使用下面这个这个查询集返回前十个对象中的偶数次对象，就会运行数据库查询:

```
>>> Entry.objects.all()[:10:2]
```

要检索单独的对象，而非列表 (比如 SELECT foo FROM bar LIMIT 1)，可以直接使用索引来代替切片。举个例子，下面这段代码将返回大标题排序后的第一条记录 Entry:

```
>>> Entry.objects.order_by('headline')[0]
```

大约等价于：

```
>>> Entry.objects.order_by('headline')[0:1].get()
```
要注意的是：如果找不到符合条件的对象，第一种方法会抛出 IndexError ，而第二种方法会抛出 DoesNotExist。 详看 get() 。

## 字段筛选条件  ##

字段筛选条件就是 SQL 语句中的 WHERE 从句。就是 Django 中的 QuerySet 的 filter(), exclude() 和 get() 方法中的关键字参数。

筛选条件的形式是 field__lookuptype=value 。 (注意：这里是双下划线)。例如：

```
>>> Entry.objects.filter(pub_date__lte='2006-01-01')
```

大体可以翻译为如下的 SQL 语句:

```
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```

这是怎么办到的？

Python 允许函式接受任意多 name-value 形式的参数，并在运行时才确定name和value的值。详情请参阅官方Python教程中的 关键字参数(Keyword Arguments)。

如果你传递了一个无效的关键字参数，会抛出 TypeError 导常。

数据库 API 支持24种查询类型；可以在 字段筛选参考(field lookup reference) 查看详细的列表。为了给您一个直观的认识，这里我们列出一些常用的查询类型：

**exact**

"exact" 匹配。例如：

```
>>> Entry.objects.get(headline__exact="Man bites dog")
```

会生成如下的 SQL 语句：

```
SELECT ... WHERE headline = 'Man bites dog';
```

如果你没有提供查询类型 -- 也就是说关键字参数中没有双下划线，那么查询类型就会被指定为 exact。

举个例子，这两个语句是相等的：

```
>>> Blog.objects.get(id__exact=14)  # Explicit form
>>> Blog.objects.get(id=14)         # __exact is implied
```

这样做很方便，因为 exact 是最常用的。

**iexact**

忽略大小写的匹配。所以下面的这个查询:

```
>>> Blog.objects.get(name__iexact="beatles blog")
```

会匹配标题是 "Beatles Blog", "beatles blog", 甚至 "BeAtlES blOG" 的 Blog

**contains**

大小写敏感的模糊匹配。 例如：

```
Entry.objects.get(headline__contains='Lennon')
```

大体可以翻译为如下的 SQL：

```
SELECT ... WHERE headline LIKE '%Lennon%';
```

要注意这段代码匹配大标题 'Today Lennon honored' ，而不能匹配 'today lennon honored'。

它也有一个忽略大小写的版本，就是 icontains。

**startswith, endswith**

分别匹配开头和结尾，同样也有忽略大小写的版本 istartswith 和 iendswith。
再强调一次，这仅仅是简短介绍。完整的参考请参见 字段筛选条件参考(field lookup reference)。

## 跨关系查询 ##

Django 提供了一种直观而高效的方式在查询(lookups)中表示关联关系，它能自动确认 SQL JOIN 联系。要做跨关系查询，就使用两个下划线来链接模型(model)间关联字段的名称，直到最终链接到你想要的 model 为止。

这个例子检索所有关联 Blog 的 name 值为 'Beatles Blog' 的所有 Entry 对象：

```
>>> Entry.objects.filter(blog__name__exact='Beatles Blog')
```

跨关系的筛选条件可以一直延展。

关系也是可逆的。可以在目标 model 上使用源 model 名称的小写形式得到反向关联。

下面这个例子检索至少关联一个 Entry 且大标题 headline 包含 'Lennon' 的所有 Blog 对象：

```
>>> Blog.objects.filter(entry__headline__contains='Lennon')
```

如果在某个关联 model 中找不到符合过滤条件的对象，Django 将视它为一个空的 (所有的值都是 NULL), 但是可用的对象。这意味着不会有异常抛出，在这个例子中：

```
Blog.objects.filter(entry__author__name='Lennon')
```

(假设关联到 Author 类), 如果没有哪个 author 与 entry 相关联，Django 会认为它没有 name 属性，而不会因为不存在 author 抛出异常。通常来说，这正是你所希望的机制。唯一的例外是使用 isnull 的情况。如下:

```
Blog.objects.filter(entry__author__name__isnull=True)
```

这段代码会得到 author 的 name 为空的 Blog 或 entry 的 author为空的 Blog。 如果不嫌麻烦，可以这样写：

```
Blog.objects.filter (entry__author__isnull=False,
        entry__author__name__isnull=True)
```

跨一对多／多对多关系(Spanning multi-valued relationships)

这部分是Django 1.0中新增的： 请查看版本记录
如果你的过滤是基于 ManyToManyField 或是逆向 ForeignKeyField 的，你可能会对下面这两种情况感兴趣。回顾 Blog/Entry 的关系(Blog 到 Entry 是一对多关系)，如果要查找这样的 blog：它关联一个大标题包含"Lennon"，且在2008年出版的 entry ；或者要查找这样的 blogs：它关联一个大标题包含"Lennon"的 entry ，同时它又关联另外一个在2008年出版的 entry 。因为一个 Blog 会关联多个的Entry，所以上述两种情况在现实应用中是很有可能出现的。

同样的情形也出现在 ManyToManyField 上。例如，如果 Entry 有一个 ManyToManyField 字段，名字是 tags，我们想得到 tags 是"music"和"bands"的 entries，或者我们想得到包含名为"music" 的标签而状态是"public"的 entry。

针对这两种情况，Django 用一种很方便的方式来使用 filter() 和 exclude()。对于包含在同一个 filter() 中的筛选条件，查询集要同时满足所有筛选条件。而对于连续的 filter() ，查询集的范围是依次限定的。但对于跨一对多／多对多关系查询来说，在第二种情况下，筛选条件针对的是主 model 所有的关联对象，而不是被前面的 filter() 过滤后的关联对象。

这听起来会让人迷糊，举个例子会讲得更清楚。要检索这样的 blog：它要关系一个大标题中含有 "Lennon" 并且在2008年出版的 entry (这个 entry 同时满足这两个条件)，可以这样写：

```
Blog.objects.filter(entry__headline__contains='Lennon',
        entry__pub_date__year=2008)
```

要检索另外一种 blog：它关联一个大标题含有"Lennon"的 entry ，又关联一个在2008年出版的 entry （一个 entry 的大标题含有 Lennon，同一个或另一个 entry 是在2008年出版的）。可以这样写：

```
Blog.objects.filter(entry__headline__contains='Lennon').filter(
        entry__pub_date__year=2008)
```

在第二个例子中，第一个过滤器(filter)先检索与符合条件的 entry 的相关联的所有 blogs。第二个过滤器在此基础上从这些 blogs 中检索与第二种 entry 也相关联的 blog。第二个过滤器选择的 entry 可能与第一个过滤器所选择的完全相同，也可能不同。 因为过滤项过滤的是 Blog，而不是 Entry。

上述原则同样适用于 exclude()：一个单独 exclude() 中的所有筛选条件都是作用于同一个实例 (如果这些条件都是针对同一个一对多／多对多的关系)。连续的 filter() 或 exclude() 却根据同样的筛选条件，作用于不同的关联对象。

在过滤器中引用 model 中的字段(Filters can reference fields on the model)

这部分是 Django 1.1 新增的: 请查看版本记录
在上面所有的例子中，我们构造的过滤器都只是将字段值与某个常量做比较。如果我们要对两个字段的值做比较，那该怎么做呢？

Django 提供 F() 来做这样的比较。F() 的实例可以在查询中引用字段，来比较同一个 model 实例中两个不同字段的值。

例如：要查询回复数(comments)大于广播数(pingbacks)的博文(blog entries)，可以构造一个 F() 对象在查询中引用评论数量：

```
>>> from django.db.models import F
>>> Entry.objects.filter(n_pingbacks__lt=F('n_comments'))
```

Django 支持 F() 对象之间以及 F() 对象和常数之间的加减乘除和取模的操作。例如，要找到广播数等于评论数两倍的博文，可以这样修改查询语句：

```
>>> Entry.objects.filter(n_pingbacks__lt=F('n_comments') * 2)
```

要查找阅读数量小于评论数与广播数之和的博文，查询如下:

```
>>> Entry.objects.filter(rating__lt=F('n_comments') + F('n_pingbacks'))
```

你也可以在 F() 对象中使用两个下划线做跨关系查询。F() 对象使用两个下划线引入必要的关联对象。例如，要查询博客(blog)名称与作者(author)名称相同的博文(entry)，查询就可以这样写：

```
>>> Entry.objects.filter(author__name=F('blog__name'))
```

## 主键查询的简捷方式  ##

为使用方便考虑，Django 用 pk 代表主键"primary key"。

以 Blog 为例, 主键是 id 字段，所以下面三个语句都是等价的：

```
>>> Blog.objects.get(id__exact=14) # Explicit form
>>> Blog.objects.get(id=14) # __exact is implied
>>> Blog.objects.get(pk=14) # pk implies id__exact
```

pk 对 __exact 查询同样有效，任何查询项都可以用 pk 来构造基于主键的查询：

```
# Get blogs entries with id 1, 4 and 7
>>> Blog.objects.filter(pk__in=[1,4,7])

# Get all blog entries with id > 14
>>> Blog.objects.filter(pk__gt=14)
```

pk 查询也可以跨关系，下面三个语句是等价的：

```
>>> Entry.objects.filter(blog__id__exact=3) # Explicit form
>>> Entry.objects.filter(blog__id=3)        # __exact is implied
>>> Entry.objects.filter(blog__pk=3)        # __pk implies __id__exact
```

## 在LIKE语句中转义百分号%和下划线_ ##

字段筛选条件相当于 LIKE SQL 语句 (iexact, contains, icontains, startswith, istartswith, endswith 和 iendswith) ，它会自动转义两个特殊符号 -- 百分号%和下划线_。(在 LIKE 语句中，百分号%表示多字符匹配，而下划线_表示单字符匹配。)

这就意味着我们可以直接使用这两个字符，而不用考虑他们的 SQL 语义。例如，要查询大标题中含有一个百分号%的 entry：

```
>>> Entry.objects.filter(headline__contains='%')
```

Django 会处理转义；最终的 SQL 看起来会是这样：

```
SELECT ... WHERE headline LIKE '%\%%';
```

下划线_和百分号%的处理方式相同，Django 都会自动转义。

## 缓存和查询 ##

每个 QuerySet 都包含一个缓存，以减少对数据库的访问。要编写高效代码，就要理解缓存是如何工作的。

一个 QuerySet 时刚刚创建的时候，缓存是空的。 QuerySet 第一次运行时，会执行数据库查询，接下来 Django 就在 QuerySet 的缓存中保存查询的结果，并根据请求返回这些结果(比如，后面再次调用这个 QuerySet 的时候)。再次运行 QuerySet 时就会重用这些缓存结果。

要牢住上面所说的缓存行为，否则在使用 QuerySet 时可能会给你造成不小的麻烦。例如，创建下面两个 QuerySet ，并对它们求值，然后释放：

```
>>> print [e.headline for e in Entry.objects.all()]
>>> print [e.pub_date for e in Entry.objects.all()]
```

这就意味着相同的数据库查询将执行两次，事实上读取了两次数据库。而且，这两次读出来的列表可能并不完全相同，因为存在这种可能：在两次读取之间，某个 Entry 被添加到数据库中，或是被删除了。

要避免这个问题，只要简单地保存 QuerySet 然后重用即可：

```
>>> queryset = Poll.objects.all()
>>> print [p.headline for p in queryset] # Evaluate the query set.
>>> print [p.pub_date for p in queryset] # Re-use the cache from the evaluation.
```

用 Q 对象实现复杂查找 (Complex lookups with Q objects)

在 filter() 等函式中关键字参数彼此之间都是 "AND" 关系。如果你要执行更复杂的查询(比如，实现筛选条件的 OR 关系)，可以使用 Q 对象。

Q 对象(django.db.models.Q)是用来封装一组查询关键字的对象。这里提到的查询关键字请查看上面的 "Field lookups"。

例如，下面这个 Q 对象封装了一个单独的 LIKE 查询：

```
Q(question__startswith='What')
```

Q 对象可以用 & 和 | 运算符进行连接。当某个操作连接两个 Q 对象时，就会产生一个新的等价的 Q 对象。

例如，下面这段语句就产生了一个 Q ，这是用 "OR" 关系连接的两个 "question__startswith" 查询：

```
Q(question__startswith='Who') | Q(question__startswith='What')
```

上面的例子等价于下面的 SQL WHERE 从句：

```
WHERE question LIKE 'Who%' OR question LIKE 'What%'
```

你可以用 & 和 | 连接任意多的 Q 对象，而且可以用括号分组。Q 对象也可以用 ~ 操作取反，而且普通查询和取反查询(NOT)可以连接在一起使用：

```
Q(question__startswith='Who') | ~Q(pub_date__year=2005)
```

每种查询函式(比如 filter(), exclude(), get()) 除了能接收关键字参数以外，也能以位置参数的形式接受一个或多个 Q 对象。如果你给查询函式传递了多个 Q 对象，那么它们彼此间都是 "AND" 关系。例如：

```
Poll.objects.get(
    Q(question__startswith='Who'),
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
```

... 大体可以翻译为下面的 SQL：

```
SELECT * from polls WHERE question LIKE 'Who%'
    AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
```

查找函式可以混用 Q 对象和关键字参数。查询函式的所有参数(Q 关系和关键字参数) 都是 "AND" 关系。但是，如果参数中有 Q 对象，它必须排在所有的关键字参数之前。例如：

```
Poll.objects.get(
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
    question__startswith='Who')
```

... 是一个有效的查询。但下面这个查询虽然看上去和前者等价：

```
# INVALID QUERY
Poll.objects.get(
    question__startswith='Who',
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)))
```

... 但这个查询却是无效的。

> 参见
>
> 在 Django 的单元测试 OR查询实例(OR lookups examples) 中展示了 Q 的用例。

## 对象比较 ##

要比较两个对象，就和 Python 一样，使用双等号运算符：==。实际上比较的是两个 model 的主键值。

以上面的 Entry 为例，下面两个语句是等价的：

```
>>> some_entry == other_entry
>>> some_entry.id == other_entry.id
```

如果 model 的主键名称不是 id，也没关系。Django 会自动比较主键的值，而不管他们的名称是什么。例如，如果一个 model 的主键字段名称是 name，那么下面两个语句是等价的：

```
>>> some_obj == other_obj
>>> some_obj.name == other_obj.name
```

## 对象删除 ##

删除方法就是 delete()。它运行时立即删除对象而不返回任何值。例如：

```
e.delete()
```

你也可以一次性删除多个对象。每个 QuerySet 都有一个 delete() 方法，它一次性删除 QuerySet 中所有的对象。

例如，下面的代码将删除 pub_date 是2005年的 Entry 对象：

```
Entry.objects.filter(pub_date__year=2005).delete()
```

要牢记这一点：无论在什么情况下，QuerySet 中的 delete() 方法都只使用一条 SQL 语句一次性删除所有对象，而并不是分别删除每个对象。如果你想使用在 model 中自定义的 delete() 方法，就要自行调用每个对象的delete 方法。(例如，遍历 QuerySet，在每个对象上调用 delete()方法)，而不是使用 QuerySet 中的 delete()方法。

在 Django 删除对象时，会模仿 SQL 约束 ON DELETE CASCADE 的行为，换句话说，删除一个对象时也会删除与它相关联的外键对象。例如：

```
b = Blog.objects.get(pk=1)
# This will delete the Blog and all of its Entry objects.
b.delete()
```

要注意的是： delete() 方法是 QuerySet 上的方法，但并不适用于 Manager 本身。这是一种保护机制，是为了避免意外地调用 Entry.objects.delete() 方法导致 所有的 记录被误删除。如果你确认要删除所有的对象，那么你必须显式地调用：

```
Entry.objects.all().delete()
```

一次更新多个对象 (Updating multiple objects at once)

这部分是 Django 1.0 中新增的： 请查看版本文档
有时你想对 QuerySet 中的所有对象，一次更新某个字段的值。这个要求可以用 update() 方法完成。例如：

```
# Update all the headlines with pub_date in 2007.
Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')
```

这种方法仅适用于非关系字段和 ForeignKey 外键字段。更新非关系字段时，传入的值应该是一个常量。更新 ForeignKey 字段时，传入的值应该是你想关联的那个类的某个实例。例如：

```
>>> b = Blog.objects.get(pk=1)

# Change every Entry so that it belongs to this Blog.
>>> Entry.objects.all().update(blog=b)
```

update() 方法也是即时生效，不返回任何值的(与 delete() 相似)。 在 QuerySet 进行更新时，唯一的限制就是一次只能更新一个数据表，就是当前 model 的主表。所以不要尝试更新关联表和与此类似的操作，因为这是不可能运行的。

要小心的是： update() 方法是直接翻译成一条 SQL 语句的。因此它是直接地一次完成所有更新。它不会调用你的 model 中的 save() 方法，也不会发出 pre_save 和 post_save 信号(这些信号在调用 save() 方法时产生)。如果你想保存 QuerySet 中的每个对象，并且调用每个对象各自的 save() 方法，那么你不必另外多写一个函式。只要遍历这些对象，依次调用 save() 方法即可：

```
for item in my_queryset:
    item.save()
```

这部分是在 Django 1.1 中新增的： 请查看版本文档
在调用 update 时可以使用 F() 对象 来把某个字段的值更新为另一个字段的值。这对于自增记数器是非常有用的。例如，给所有的博文 (entry) 的广播数 (pingback) 加一：

```
>>> Entry.objects.all().update(n_pingbacks=F('n_pingbacks') + 1)
```

但是，与 F() 对象在查询时所不同的是，在filter 和 exclude子句中，你不能在 F() 对象中引入关联关系(NO-Join)，你只能引用当前 model 中要更新的字段。如果你在 F() 对象引入了Join 关系object，就会抛出 FieldError 异常：

```
# THIS WILL RAISE A FieldError
>>> Entry.objects.update(headline=F('blog__name'))
```

## 对象关联 ##

当你定义在 model 定义关系时 (例如， ForeignKey, OneToOneField, 或 ManyToManyField)，model 的实例自带一套很方便的API以获取关联的对象。

以最上面的 models 为例，一个 Entry 对象 e 能通过 blog 属性获得相关联的 Blog 对象： e.blog。

(在场景背后，这个功能是由 Python 的 descriptors 实现的。如果你对此感兴趣，可以了解一下。)

Django 也提供反向获取关联对象的 API，就是由从被关联的对象得到其定义关系的主对象。例如，一个 Blog 类的实例 b 对象通过 entry_set 属性得到所有相关联的 Entry 对象列表: b.entry_set.all()。

这一节所有的例子都使用本页顶部所列出的 Blog, Author 和 Entry model。

## 一对多关系 ##

### 正向 ###

如果一个 model 有一个 ForeignKey字段，我们只要通过使用关联 model 的名称就可以得到相关联的外键对象。

例如：

```
>>> e = Entry.objects.get(id=2)
>>> e.blog # Returns the related Blog object.
```

你可以设置和获得外键属性。正如你所期望的，改变外键的行为并不引发数据库操作，直到你调用 save()方法时，才会保存到数据库。例如：

```
>>> e = Entry.objects.get(id=2)
>>> e.blog = some_blog
>>> e.save()
```

如果外键字段 ForeignKey 有一个 null=True 的设置(它允许外键接受空值 NULL)，你可以赋给它空值 None 。例如：

```
>>> e = Entry.objects.get(id=2)
>>> e.blog = None
>>> e.save() # "UPDATE blog_entry SET blog_id = NULL ...;"
```

在一对多关系中，第一次正向获取关联对象时，关联对象会被缓存。其后根据外键访问时这个实例，就会从缓存中获得它。例如：

```
>>> e = Entry.objects.get(id=2)
>>> print e.blog  # Hits the database to retrieve the associated Blog.
>>> print e.blog  # Doesn't hit the database; uses cached version.
```

要注意的是，QuerySet 的 select_related() 方法提前将所有的一对多关系放入缓存中。例如：

```
>>> e = Entry.objects.select_related().get(id=2)
>>> print e.blog  # Doesn't hit the database; uses cached version.
>>> print e.blog  # Doesn't hit the database; uses cached version.
```

### 逆向关联 ###

如果 model 有一个 ForeignKey外键字段，那么外联 model 的实例可以通过访问 Manager 来得到所有相关联的源 model 的实例。默认情况下，这个 Manager 被命名为 FOO_set, 这里面的 FOO 就是源 model 的小写名称。这个 Manager 返回 QuerySets，它是可过滤和可操作的，在上面 "对象获取(Retrieving objects)" 有提及。

例如：

```
>>> b = Blog.objects.get(id=1)
>>> b.entry_set.all() # Returns all Entry objects related to Blog.

# b.entry_set is a Manager that returns QuerySets.
>>> b.entry_set.filter(headline__contains='Lennon')
>>> b.entry_set.count()
```

你可以通过在 ForeignKey() 的定义中设置 related_name 的值来覆写 FOO_set 的名称。例如，如果 Entry model 中做一下更改： blog = ForeignKey(Blog, related_name='entries')，那么接下来就会如我们看到这般：

```
>>> b = Blog.objects.get(id=1)
>>> b.entries.all() # Returns all Entry objects related to Blog.

# b.entries is a Manager that returns QuerySets.
>>> b.entries.filter(headline__contains='Lennon')
>>> b.entries.count()
```

你不能在一个类当中访问 ForeignKey Manager ；而必须通过类的实例来访问：

```
>>> Blog.entry_set
Traceback:
    ...
AttributeError: "Manager must be accessed via instance".
```

除了在上面 "对象获取Retrieving objects" 一节中提到的 QuerySet 方法之外，ForeignKey Manager 还有如下一些方法。下面仅仅对它们做一个简短介绍，详情请查看 related objects reference。

`add(obj1, obj2, ...)`

将某个特定的 model 对象添加到被关联对象集合中。

`create(**kwargs)`

创建并保存一个新对象，然后将这个对象加被关联对象的集合中，然后返回这个新对象。

`remove(obj1, obj2, ...)`

将某个特定的对象从被关联对象集合中去除。

`clear()`

清空被关联对象集合。
想一次指定关联集合的成员，那么只要给关联集合分配一个可迭代的对象即可。它可以包含对象的实例，也可以只包含主键的值。例如：

```
b = Blog.objects.get(id=1)
b.entry_set = [e1, e2]
```

在这个例子中，e1 和 e2 可以是完整的 Entry 实例，也可以是整型的主键值。

如果 clear() 方法是可用的，在迭代器(上例中就是一个列表)中的对象加入到 entry_set 之前，已存在于关联集合中的所有对象将被清空。如果 clear() 方法 不可用，原有的关联集合中的对象就不受影响，继续存在。

这一节提到的每一个 "reverse" 操作都是实时操作数据库的，每一个添加，创建，删除操作都会及时保存将结果保存到数据库中。

## 多对多关系 ##

在多对多关系的任何一方都可以使用 API 访问相关联的另一方。多对多的 API 用起来和上面提到的 "逆向" 一对多关系关系非常相象。

唯一的差虽就在于属性的命名： ManyToManyField 所在的 model (为了方便，我称之为源model A) 使用字段本身的名称来访问关联对象；而被关联的另一方则使用 A 的小写名称加上 '_set' 后缀(这与逆向的一对多关系非常相象)。

下面这个例子会让人更容易理解：

```
e = Entry.objects.get(id=3)
e.authors.all() # Returns all Author objects for this Entry.
e.authors.count()
e.authors.filter(name__contains='John')

a = Author.objects.get(id=5)
a.entry_set.all() # Returns all Entry objects for this Author.
```

与 ForeignKey 一样, ManyToManyField 也可以指定 related_name。在上面的例子中，如果 Entry 中的 ManyToManyField 指定 related_name='entries'，那么接下来每个 Author 实例的 entry_set 属性都被 entries 所代替。

### 一对一关系 ###

相对于多对一关系而言，一对一关系不是非常简单的。如果你在 model 中定义了一个 OneToOneField 关系，那么你就可以用这个字段的名称做为属性来访问其所关联的对象。

例如：

```
class EntryDetail(models.Model):
    entry = models.OneToOneField(Entry)
    details = models.TextField()

ed = EntryDetail.objects.get(id=2)
ed.entry # Returns the related Entry object.
```

与 "reverse" 查询不同的是，一对一关系的关联对象也可以访问 Manager 对象，但是这个 Manager 表现一个单独的对象，而不是一个列表：

```
e = Entry.objects.get(id=2)
e.entrydetail # returns the related EntryDetail object
```

如果一个空对象被赋予关联关系，Django 就会抛出一个 DoesNotExist 异常。

和你定义正向关联所用的方式一样，类的实例也可以赋予逆向关联方系：

```
e.entrydetail = ed
```

## 关系中的反向连接是如何做到的？ ##

其他对象关系的映射(ORM)需要你在关联双方都定义关系。而 Django 的开发者则认为这违背了 DRY 原则 (Don't Repeat Yourself)，所以 Django 只需要你在一方定义关系即可。

但仅由一个 model 类并不能知道其他 model 类是如何与它关联的，除非是其他 model 也被载入，那么这是如何办到的？

答案就在于 INSTALLED_APPS 设置中。任何一个 model 在第一次调用时，Django 就会遍历所有的 INSTALLED_APPS 的所有 models，并且在内存中创建中必要的反向连接。本质上来说，INSTALLED_APPS 的作用之一就是确认 Django 完整的 model 范围。

## 在关联对象上的查询 ##

包含关联对象的查询与包含普通字段值的查询都遵循相同的规则。为某个查询指定某个值的时候，你可以使用一个类实例，也可以使用对象的主键值。

例如，如果你有一个 Blog 对象 b ，它的 id=5, 下面三个查询是一样的：

```
Entry.objects.filter(blog=b) # Query using object instance
Entry.objects.filter(blog=b.id) # Query using id from instance
Entry.objects.filter(blog=5) # Query using id directly
```

## 直接使用SQL ##

如果你发现某个 SQL 查询用 Django 的数据库映射来处理会非常复杂的话，你可以使用直接写 SQL 来完成。

建议的方式是在你的 model 自定义方法或是自定义 model 的 manager 方法来运行查询。虽然 Django 不要求数据操作必须在 model 层中执行。但是把你的商业逻辑代码放在一个地方，从代码组织的角度来看，也是十分明智的。详情请查看 执行原生SQL查询(Performing raw SQL queries).

最后，要注意的是，Django的数据操作层仅仅是访问数据库的一个接口。你可以用其他的工具，编程语言，数据库框架来访问数据库。对你的数据库而言，没什么是非用 Django 不可的。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Executing queries](https://docs.djangoproject.com/en/1.8/topics/db/queries/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
