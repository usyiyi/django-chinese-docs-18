<!--
  译者：WrongWay [www.wrongway.me]
  1.8更新：Github@wizardforcel
-->

# 查询集 API 参考 #

该文档详细介绍了 QuerySet 的 API。这里面的内容是建立在 model 和 database query 文档的基础上，所以建议您在看该文档之前先读一下这两个文档。

贯穿该文档，我们仍使用database query guide 文档中的 example weblog models 为例：

## 在查询时发生了什么 ##

从内部讲，QuerySet 可以被构造，过滤，切片，做为参数传递，这些行为都不会对数据库进行操作。只要你查询的时候才真正的操作数据库。

下面的 QuerySet 行为会导致执行查询的操作：

+ 循环(Iteration)：QuerySet 是可迭代的，在你遍历对象时就会执行数据库操作。例如，打印出所有博文的大标题：

```
for e in Entry.objects.all():
    print e.headline
```

+ 切片(Slicing)：在 查询限定(Limiting QuerySets) 中提到, a QuerySet 是可以用 Python 的数组切片语法完成切片。一般来说对一个 QuerySet 切片就返回另一个 QuerySet (新 QuerySet 不会被执行)。不过如果你在切片时使用了 "step" 参数，Django 仍会执行数据库操作。

+ 序列化／缓存化(Pickling/Caching)： 详情请查看 pickling QuerySets。 这一节所强调的一点是查询结果是从数据库中读取的。

+ repr(). 调用 QuerySet 的 repr() 方法时，查询就会被运行。这对于 Python 命令行来说非常方便，你可以使用 API 立即看到查询结果。

+ len(). 调用 QuerySet 的 len() 方法，查询就会被运行。这正如你所料，会返回查询结果列表的长度。

注意：如果你想得到集合中记录的数量，就不要使用 QuerySet 的 len() 方法。因为直接在数据库层面使用 SQL 的 SELECT COUNT(*) 会更加高效，Django 提供了 count() 方法就是这个原因。详情参阅下面的 count() 方法。

+ list(). 对 QuerySet 应用 list() 方法，就会运行查询。例如：

```
entry_list = list(Entry.objects.all())
```

要注意地是：使用这个方法会占用大量内存，因为 Django 将列表内容都载入到内存中。做为对比，遍历 QuerySet 是从数据库读取数据，仅在使用某个对象时才将其载入到内容中。

## 序列化查询集 ##

如果你要 序列化(pickle) 一个 QuerySet，Django 首先就会将查询对象载入到内存中以完成序列化，这样你就可以第一时间使用对象(直接从数据库中读取数据需要一定时间，这正是缓存所想避免的)。而序列化是缓存化的先行工作，所以在缓存查询时，首先就会进行序列化工作。这意味着当你反序列化 QuerySet 时，第一时间就会从内存中获得查询的结果，而不是从数据库中查找。

如果你只是想序列化部分必要的信息以便晚些时候可以从数据库中重建 Queryset ，那只序列化 QuerySet 的 query 属性即可。接下来你就可以使用下面的代码重建原来的 QuerySet （这当中没有数据库读取）：

```
>>> import pickle
>>> query = pickle.loads(s)     # Assuming 's' is the pickled string.
>>> qs = MyModel.objects.all()
>>> qs.query = query            # Restore the original 'query'.
```

query 属性是一个不透明的对象。这就意味着它的内部结构并不是公开的。即便如此，对于本节提到的序列化和反序列化来说，它仍是安全和被完全支持的。

## 查询API ##

一般情况下，我们都是直接使用而不是手动创建一个 Manager ，但在这里我们仍要介绍一下 QuerySet 的形式。

`class QuerySet([model=None])`

通常你想修改某个 QuerySet 时，你会使用 过滤链(chaining filters)。所以为了让它们能正常工作，大多数 QuerySet 方法都返回新的查询(querysets)。

## 返回新查询的方法 ##

Django 为 QuerySet 提供了一套优雅的方法，用来修改 QuerySet 查询结果的类型和运行 SQL 查询的方式。

`filter(**kwargs)`

返回一个新的 QuerySet ，它包含了与所给的筛选条件相匹配的对象。

这些筛选条件(\*\*kwargs)在下面的字段筛选(Field lookups) 中有详细介绍。多个条件之间在 SQL 语句中是 AND 关系。

`exclude(**kwargs)`

返回一个新的 QuerySet，它包含那些与所给筛选条件不匹配的对象。

这些筛选条件(**kwargs)也在下面的 字段筛选(Field lookups) 中有详细描述。多个条件之间在 SQL 语句中也是 AND 关系，但是整体又是一个 NOT() 关系。

这个例子剔除了出版日期 pub_date 晚于 2005-1-3 并且大标题 headline 是 "Hello" 的所有博文(entry)：

```
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3), headline='Hello')
```

在 SQL 语句中，这等价于：

```
SELECT ...
WHERE NOT (pub_date > '2005-1-3' AND headline = 'Hello')
```

这个例子剔除出版日期 pub_date 晚于 2005-1-3 或者大标题是 "Hello" 的所有博文：

```
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3)).exclude(headline='Hello')
```

在 SQL 语句中，这等价于：

```
SELECT ...
WHERE NOT pub_date > '2005-1-3'
OR NOT headline = 'Hello'
```

要注意第二个例子是有很多限制的。

## 注解annotate(\*args, \*\*kwargs) ##

这部分是 Django 1.1 新增的： 请查看版本文档。
我们可以为 QuerySet 中的每个对象添加注解。可以通过计算查询结果中每个对象所关联的对象集合，从而得出总计值(也可以是平均值或总和，等等)，做为 QuerySet 中对象的注解。annotate() 中的每个参数都会被做为注解添加到 QuerySet 中返回的对象。

Django 提供的注解函式在下面的 (注解函式Aggregation Functions) 有详细介绍。

注解会使用关键字参数来做为注解的别名。其中任何参数都会生成一个别名，它与注解函式的名称和被注解的 model 相关。

例如，你正在操作一个博客列表，你想知道一个博客究竟有多少篇博文：

>>> q = Blog.objects.annotate(Count('entry'))
# The name of the first blog
>>> q[0].name
'Blogasaurus'
# The number of entries on the first blog
>>> q[0].entry__count
42
Blog model 类本身并没有定义 entry__count 属性，但可以使用注解函式的关系字参数，从而改变注解的命名：

>>> q = Blog.objects.annotate(number_of_entries=Count('entry'))
# The number of entries on the first blog, using the name provided
>>> q[0].number_of_entries
42
要深入了解注解，请参阅 注解指南(the topic guide on Aggregation)。

order_by(*fields)

默认情况下， QuerySet 返回的查询结果是根据 model 类的 Meta 设置所提供的 ordering 项中定义的排序元组来进行对象排序的。你可以使用 order_by 方法覆盖之前 QuerySet 中的排序设置。

例如：

Entry.objects.filter(pub_date__year=2005).order_by('-pub_date', 'headline')
返回结果就会先按照 pub_date 进行升序排序，再按照 headline 进行降序排序。 "-pub_date" 前面的负号"?"表示降序排序。默认是采用升序排序。要随机排序，就使用 "?"，例如：

Entry.objects.order_by('?')
注意：order_by('?') 可能会非常缓慢和消耗过多资源，这取决于你所使用的数据库。

要根据其他 model 字段排序，所用语法和跨关系查询的语法相同。就是说，用两个连续的下划线(__)连接关联 model 和 要排序的字段名称, 而且可以一直延伸。例如：

Entry.objects.order_by('blog__name', 'headline')
如果你想对关联字段排序，在没有指定 Meta.ordering 的情况下，Django 会采用默认排序设置，就是按照关联 model 的主键进行排序。例如：

Entry.objects.order_by('blog')
...等价于：

Entry.objects.order_by('blog__id')
...这是因为 Blog model 没有声明排序项的原故。

如果你使用了 distinct() 方法，那么在对关联字段排序时要格外谨慎。详情请查看 distinct() 一节了解在使用 distinct() 时，是如何对预期的排序结果产生意想不到的影响。

在 Django 当中是可以按照多值字段（例如 ManyToMany 字段）进行排序的。不过，这个特性虽然先进，但是并不实用。除非是你已经很清楚过滤结果或可用数据中的每个对象，都只有一个相关联的对象时（就是相当于只是一对一关系时），排序才会符合你预期的结果，所以对多值字段排序时要格外注意。

这部分是在 Django 1.0 中新增的： 请查看版本文档
如果你不想对任何字段排序，也不想使用 model 中原有的排序设置，那么可以调用无参数的 order_by() 方法。

这部分是在 Django 1.0 中新增的： 请查看版本文档
跨关系排序的语法已经改变，与 Django 0.96 有所不同，旧版本请查看 Django 0.96 documentation。

对于排序项是否应该大小写敏感，Django 并没有提供设置方法，这完全取决于后端的数据库对排序大小写如何处理。

这部分是在 Django 1.1 中新增的： 请查看版本文档
你可以令某个查询结果是可排序的，也可以是不可排序的，这取决于 QuerySet.ordered 属性。如果它的值是 True ，那么 QuerySet 就是可排序的。

reverse()

这部分是在 Django 1.0 中新增的： 请查看版本文档
使用 reverse() 方法会对查询结果进行反向排序。调用两次 reverse() 方法相当于排序没发生改过。

要得到查询结果中最后五个对象，可以这样写：

my_queryset.reverse()[:5]
要注意这种方式与 Python 语法中的从尾部切片是完全不一样的。在上面的例子中，是先得到最后一个元素，然后是倒数第二个，依次处理。但是如果我们有一个 Python 队列，使用 seq[-5:]时，却是先得到第五个元素。Django 之所以采用 reverse 来获取倒数的记录，而不支持切片的方法，原因就是后者在 SQL 中难以做好。

还有一点要注意，就是 reverse() 方法应该只作用于已定义了排序项 QuerySet (例如，在查询时使用了order_by()方法，或是在 model 类当中直接定义了排序项). 如果并没有明确定义排序项，那么调用 QuerySet, calling reverse() 就没什么实际意义（因为在调用 reverse() 之前，数据没有定义排序，所以在这之后也不会进行排序。)

distinct()

返回一个新的 QuerySet ，它会在执行 SQL 查询时使用 SELECT DISTINCT。这意味着返回结果中的重复记录将被剔除。

默认情况下， QuerySet 并会剔除重复的记录。在实际当中，这不是什么问题，因为象 Blog.objects.all() 这样的查询并不会产生重复的记录。但是，如果你使用 QuerySet 做多表查询时，就很可能会产生重复记录。这时，就可以使用 distinct() 方法。

Note

在 order_by(*fields) 中出现的字段也会包含在 SQL SELECT 列中。如果和 distinct() 同时使用，有时返回的结果却与预想的不同。这是因为：如果你对跨关系的关联字段进行排序，这些字段就会被添加到被选取的列中，这就可能产生重复数据（比如，其他的列数据都相同，只是关联字段的值不同）。但由于 order_by 中的关联字段并不会出现在返回结果中（他们仅仅是用来实现order），所以有时返回的数据看上去就象是并没有进行过 distinct 处理一样。

同样的原因，如果你用 values() 方法获得被选取的列，就会发现包含在 order_by() (或是 model 类的 Meta 中设置的排序项)中的字段也包含在里面，就会对返回的结果产生影响。

本节强调的就是在你使用 distinct() 时，要谨慎对待关联字段排序。同样的，在同时使用 distinct() 和 values() 时，如果排序字段并没有出现在 values() 返回的结果中，那么也要引起注意。

values(*fields)

返回一个 ValuesQuerySet -- 它是一个特殊的 QuerySet ，运行后得到的并不是一系列 model 的实例化对象，而是一个字典列表。

每个字典都表示一个对象，而键名就是 model 对象的属性名称。

下面的例子就对 values() 得到的字典与传统的 model 对象进行了对比：

# This list contains a Blog object.
>>> Blog.objects.filter(name__startswith='Beatles')
[<Blog: Beatles Blog>]

# This list contains a dictionary.
>>> Blog.objects.filter(name__startswith='Beatles').values()
[{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]
values() 可以接收可选的位置参数，*fields，就是字段的名称，用来限制 SELECT 选取的数据。如果你指定了字段参数，每个字典就会以 Key-Value 的形式保存你所指定的字段信息；如果没有指定，每个字典就会包含当前数据表当中的所有字段信息。

例如：

>>> Blog.objects.values()
[{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}],
>>> Blog.objects.values('id', 'name')
[{'id': 1, 'name': 'Beatles Blog'}]
下面这些细节值得注意：

values() 方法不返回 ManyToManyField 属性的任何信息，如果你把 ManyToManyField 的名称做为参数传递给 values()，就会抛出异常。

如果你有一个名为 foo 的ForeignKey 字段，默认情况下调用 values() 返回的字典中包含键名为 foo_id 的字典项，因为它是一个隐含的 model 字段，用来保存关联对象的主键值( foo 属性用来联系相关联的 model )。当你使用 values() 并传递字段名称时， 传递foo 或 foo_id 都会得到相同的结果 (字典中的键名会自动换成你传递的字段名称)。

例如：

>>> Entry.objects.values()
[{'blog_id: 1, 'headline': u'First Entry', ...}, ...]

>>> Entry.objects.values('blog')
[{'blog': 1}, ...]

>>> Entry.objects.values('blog_id')
[{'blog_id': 1}, ...]
在 values() 和 distinct() 同时使用时，要注意排序项会影响返回的结果，详情请查看上面 distinct() 一节。

这部分是在 Django 1.0 中新增的： 详情请查阅版本文档
前面所提到的，在 values() 中使用 blog_id 是无效的，只能用 blog。这是 Django 1.0 及后续版本的用法。

ValuesQuerySet 是非常有用的。利用它，你就可以只获得你所需的那部分数据，而不必同时读取其他的无用数据。

最后，要提醒的是，ValuesQuerySet 是 QuerySet 的一个子类，所以它拥有 QuerySet 所有的方法。你可以对它调用 filter() 或是 order_by() 以及其他方法。所以下面俩种写法是等价的：

Blog.objects.values().order_by('id')
Blog.objects.order_by('id').values()
Django 的编写者们更喜欢第二种写法，就是先写影响 SQL 的方法，再写影响输出的方法（比如例中先写 order，再写values ），但这些都无关紧要，完全视你个人喜好而定。

values_list(*fields)

这部分是在 Django 1.0 中新增的： 请查阅版本文档
它与 values() 非常相似，只不过后者返回的结果是字典列表，而 values() 返回的结果是元组列表。每个元组都包含传递给 values_list() 的字段名称和内容。比如第一项就对应着第一个字段，例如：

>>> Entry.objects.values_list('id', 'headline')
[(1, u'First entry'), ...]
如果你传递了一个字段做为参数，那么你可以使用 flat 参数。如果它的值是 True，就意味着返回结果都是单独的值，而不是元组。下面的例子会讲得更清楚：

>>> Entry.objects.values_list('id').order_by('id')
[(1,), (2,), (3,), ...]

>>> Entry.objects.values_list('id', flat=True).order_by('id')
[1, 2, 3, ...]
如果传递的字段不止一个，使用 flat 就会导致错误。

如果你没给 values_list() 传递参数，它就会按照字段在 model 类中定义的顺序返回所有的字段。

dates(field, kind, order='ASC')

返回一个 DateQuerySet ，就是提取 QuerySet 查询中所包含的日期，将其组成一个新的 datetime.datetime 对象的列表。

field 是你的 model 中的 DateField 或 DateTimeField 字段名称。

kind 是 "year", "month" 或 "day" 之一。 每个 datetime.datetime 对象都会根据所给的 type 进行截减。

"year" 返回所有时间值中非重复的年分列表。
"month" 返回所有时间值中非重复的年／月列表。
"day" 返回所有时间值中非重复的年／月／日列表。
order, 默认是 'ASC'，只有两个取值 'ASC' 或 'DESC'。它决定结果如何排序。

例子：

>>> Entry.objects.dates('pub_date', 'year')
[datetime.datetime(2005, 1, 1)]
>>> Entry.objects.dates('pub_date', 'month')
[datetime.datetime(2005, 2, 1), datetime.datetime(2005, 3, 1)]
>>> Entry.objects.dates('pub_date', 'day')
[datetime.datetime(2005, 2, 20), datetime.datetime(2005, 3, 20)]
>>> Entry.objects.dates('pub_date', 'day', order='DESC')
[datetime.datetime(2005, 3, 20), datetime.datetime(2005, 2, 20)]
>>> Entry.objects.filter(headline__contains='Lennon').dates('pub_date', 'day')
[datetime.datetime(2005, 3, 20)]
none()

这部分是在 Django 1.0 中新增的： 请查看版本文档
返回一个 EmptyQuerySet -- 它是一个运行时只返回空列表的 QuerySet。它经常用在这种场合：你要返回一个空列表，但是调用者却需要接收一个 QuerySet 对象。（这时，就可以用它代替空列表）

例如：

>>> Entry.objects.none()
[]
all()

这部分是在 Django 1.0 中新增的： 请查看版本文档
返回当前 QuerySet (或者是传递的 QuerySet 子类)的一分拷贝。 这在某些场合是很用的，比如，你想对一个 model manager 或是一个 QuerySet 的查询结果做进一步的过滤。你就可以调用 all() 获得一分拷贝以继续操作，从而保证原 QuerySet 的安全。

select_related()

返回一个 QuerySet ，它会在执行查询时自动跟踪外键关系，从而选取所关联的对象数据。它是一个增效器，虽然会导致较大的数据查询（有时会非常大），但是接下来再使用外键关系获得关联对象时，就会不再次读取数据库了。

下面的例子展示在获得关联对象时，使用 select_related() 和不使用的区别，首先是不使用的例子：

# Hits the database.
e = Entry.objects.get(id=5)

# Hits the database again to get the related Blog object.
b = e.blog
接下来是使用 select_related 的例子：

# Hits the database.
e = Entry.objects.select_related().get(id=5)

# Doesn't hit the database, because e.blog has been prepopulated
# in the previous query.
b = e.blog
select_related() 会尽可能地深入遍历外键连接。例如：

class City(models.Model):
    # ...

class Person(models.Model):
    # ...
    hometown = models.ForeignKey(City)

class Book(models.Model):
    # ...
    author = models.ForeignKey(Person)
...接下来调用 Book.objects.select_related().get(id=4) 将缓存关联的 Person 和 City：

b = Book.objects.select_related().get(id=4)
p = b.author         # Doesn't hit the database.
c = p.hometown       # Doesn't hit the database.

b = Book.objects.get(id=4) # No select_related() in this example.
p = b.author         # Hits the database.
c = p.hometown       # Hits the database.
要注意的是，默认情况下，select_related() 并不跟踪 null=True 的外键关系。

一般情况下，使用 select_related() 会极大的改善性能，因为它可以避免过多的数据库操作。但是，在关系嵌套深度过多的情况下，select_related() 有时会结束对关系的跟踪，而且因为生成的查询会非常大大，会导致操作变慢。

在这种情况下，你可以使用 depth 参数，添加到 select_related()中从而控制要跟踪的关系层数：

b = Book.objects.select_related(depth=1).get(id=4)
p = b.author         # Doesn't hit the database.
c = p.hometown       # Requires a database call.
有时，你的 model 中有多个关联字段，而你只想一性访问其中的某几个，这时你就可以将这些字段的名称做为参数传递给 select_related() ，然后就只有它们所关联的关系被跟踪。你甚至可以象使用过滤器那样，在其中使用双下划线来表示 model 之间的关联，例如：

class Room(models.Model):
    # ...
    building = models.ForeignKey(...)

class Group(models.Model):
    # ...
    teacher = models.ForeignKey(...)
    room = models.ForeignKey(Room)
    subject = models.ForeignKey(...)
...如果你只想跟踪 room 和 subject 属性，可以这样写：

g = Group.objects.select_related('room', 'subject')
这样写也是正确的：

g = Group.objects.select_related('room__building', 'subject')
...这样也会将 building 关系加入到跟踪中。

你只能将外键字段 ForeignKey 传递给 select_related。你可以用传递名称做为参数，用来连接 null=True 的外链关系 (与默认的 select_related() 不同)。如果你在同一个 select_related() 中同时使用字段名称和 depth 参数，就会导致错误，因为他们是矛盾的两个选项。

这部分是在 Django 1.0 中新增的： 请查看版本文档
在 select_related() 中同时使用 depth 参数和指定关联字段名称在 Django 1.0 及其后续版本中是可用的。

extra(select=None, where=None, params=None, tables=None, order_by=None, select_params=None)

有些情况下，Django 的查询语法难以简练地表达复杂的 WHERE 子句。对于这种情况，Django 提供了 extra() QuerySet 修改机制，它能在QuerySet 生成的 SQL 从句中注入新子句。

由于产品差异的原因，这些自定义的查询难以保障在不同的数据库之间兼容(因为你手写 SQL 代码的原因)，而且违背了 DRY 原则，所以如非必要，还是尽量避免写 extra。

在 extra 可以指定一个或多个 params 参数，如 select，where 或 tables。所有参数都是可选的，但你至少要使用一个。

select
select 参数可以让你在 SELECT 从句中添加其他字段信息。它应该是一个字典，存放着属性名到 SQL 从句的映射。

例如：

Entry.objects.extra(select={'is_recent': "pub_date > '2006-01-01'"})
结果中每个 Entry 对象都有一个额外的 is_recent 属性，它是一个布尔值，表示 pub_date 是否晚于2006年1月1号。

Django 会直接在 SELECT 中加入对应的 SQL 片断，所以转换后的 SQL 如下：

SELECT blog_entry.*, (pub_date > '2006-01-01')
FROM blog_entry;
下面这个例子更复杂一些；它会在每个 Blog 对象中添加一个 entry_count 属性，它会运行一个子查询，得到相关联的 Entry 对象的数量：

Blog.objects.extra(
    select={
        'entry_count': 'SELECT COUNT(*) FROM blog_entry WHERE blog_entry.blog_id = blog_blog.id'
    },
)
(在上面这个特例中，我们要了解这个事实，就是 blog_blog 表已经存在于 FROM 从句中。)

翻译成 SQL 如下：

SELECT blog_blog.*, (SELECT COUNT(*) FROM blog_entry WHERE blog_entry.blog_id = blog_blog.id) AS entry_count
FROM blog_blog;
要注意的是，大多数数据库需要在子句两端添加括号，而在 Django 的 select 从句中却无须这样。同样要引起注意的是，在某些数据库中，比如某些 MySQL 版本，是不支持子查询的。

这部分是在 Django 1.0 中新增的： 请查看版本文档
某些时候，你可能想给 extra(select=...) 中的 SQL 语句传递参数，这时就可以使用 select_params 参数。因为 select_params 是一个队列，而 select 属性是一个字典，所以两者在匹配时应正确地一一对应。在这种情况下中，你应该使用 django.utils.datastructures.SortedDict 匹配 select 的值，而不是使用一般的 Python 队列。

例如：

Blog.objects.extra(
    select=SortedDict([('a', '%s'), ('b', '%s')]),
    select_params=('one', 'two'))
在使用 extra() 时要避免在 select 字串含有 "%%s" 子串， 这是因为在 Django 中，处理 select 字串时查找的是 %s 而并非转义后的 % 字符。所以如果对 % 进行了转义，反而得不到正确的结果。

where / tables
你可以使用 where 参数显示定义 SQL 中的 WHERE 从句，有时也可以运行非显式地连接。你还可以使用 tables 手动地给 SQL FROM 从句添加其他表。

where 和 tables 都接受字符串列表做为参数。所有的 where 参数彼此之间都是 "AND" 关系。

例如：

Entry.objects.extra(where=['id IN (3, 4, 5, 20)'])
...大致可以翻译为如下的 SQL:

SELECT * FROM blog_entry WHERE id IN (3, 4, 5, 20);
下面这个是 tables 的例子:

queryset.extra(tables=['(select * from table) as k'])
翻译的 SQL 如下：

select ........ from `self_table`, `(select * from table) as k`
这展示了如何运行子查询。

在使用 tables 时，如果你指定的表在查询中已出现过，那么要格外小心。当你通过 tables 参数添加其他数据表时，如果这个表已经被包含在查询中，那么 Django 就会认为你想再一次包含这个表。这就导致了一个问题：由于重复出现多次的表会被赋予一个别名，所以除了第一次之外，每个重复的表名都会分别由 Django 分配一个别名。所以，如果你同时使用了 where 参数，在其中用到了某个重复表，却不知它的别名，那么就会导致错误。

一般情况下，你只会添加一个未在查询中出现的新表。但是如果上面所提到的特殊情况发生了，那么可以采用如下措施解决。首先，判断是否有必要要出现重复的表，能否将重复的表去掉。如果这点行不通，就试着把 extra() 调用放在查询结构的起始处，因为首次出现的表名不会被重命名，所以可能能解决问题。如果这也不行，那就查看生成的 SQL 语句，从中找出各个数据库的别名，然后依此重写 where 参数，因为只要你每次都用同样的方式调用查询(queryset)，表的别名都不会发生变化。所以你可以直接使用表的别名来构造 where。

order_by
如果你已通过 extra() 添加了新字段或是数据库，此时若想对新字段进行排序，就可以给 extra() 中的 order_by 参数传递一个排序字符串序列。字符串可以是 model 原生的字段名(与使用普通的 order_by() 方法一样)，也可以是 table_name.column_name 这种形式，或者是你在 extra() 的 select 中所定义的字段。

例如：

q = Entry.objects.extra(select={'is_recent': "pub_date > '2006-01-01'"})
q = q.extra(order_by = ['-is_recent'])
这段代码按照 is_recent 对记录进行排序，字段值是 True 的排在前面，False 的排在后面。(True 在降序排序时是排在 False 的前面)。

顺便说一下，上面这段代码同时也展示出，可以依你所愿的那样多次调用 extra() 操作(每次添加新的语句结构即可)。

params
上面提到的 where 参数还可以用标准的 Python 占位符 -- '%s' ，它可以根据数据库引擎自动决定是否添加引号。 params 参数是用来替换占位符的字符串列表。

例如：

Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
使用 params 替换 where 的中嵌入值是一个非常好的做法，这是因为 params 可以根据你的数据库判断要不要给传入值添加引号（例如，传入值中的引号会被自动转义）。

不好的用法：

Entry.objects.extra(where=["headline='Lennon'"])
优雅的用法：

Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
defer(*fields)

这部分是在 Django 1.1 中新增的： 请查看版本文档
在某些数据复杂的环境下，你的 model 可能包含非常多的字段，可能某些字段包含非常多的数据(比如，文档字段)，或者将其转化为 Python 对象会消耗非常多的资源。在这种情况下，有时你可能并不需要这种字段的信息，那么你可以让 Django 不读取它们的数据。

将不想载入的字段的名称传给 defer() 方法，就可以做到延后载入：

Entry.objects.defer("lede", "body")
延后截入字段的查询返回的仍是 model 类的实例。在你访问延后载入字段时，你仍可以获得字段的内容，所不同的是，内容是在你访问延后字段时才读取数据库的，而普通字段是在运行查询(queryset)时就一次性从数据库中读取数据的。

你可以多次调用 defer() 方法。每个调用都可以添加新的延后载入字段：

# Defers both the body and lede fields.
Entry.objects.defer("body").filter(headline="Lennon").defer("lede")
对延后载入字段进行排序是不会起作用的；重复添加延后载入字段也不会有何不良影响。

你也可以延后载入关联 model 中的字段(前提是你使用 select_related() 载入了关联 model)，用法也是用双下划线连接关联字段：

Blog.objects.select_related().defer("entry__lede", "entry__body")
如果你想清除延后载入的设置，只要使用将 None 做为参数传给 defer() 即可：

# Load all fields immediately.
my_queryset.defer(None)
有些字段无论你如何指定，都不会被延后加载。比如，你永远不能延后加载主键字段。如果你使用 select_related() 获得关联 model 字段信息，那么你就不能延后载入关联 model 的主键。（如果这样做了，虽然不会抛出错误，事实上却不完成延后加载）

注意

defer() 方法(和随后提到的 only() 方法) 都只适用于特定情况下的高级属性。它们可以提供性能上的优化，不过前提是你已经对你用到的查询有过很深入细致的分析，非常清楚你需要的究竟是哪些信息，而且已经对你所需要的数据和默认情况下返回的所有数据进行比对，清楚两者之间的差异。这完成了上述工作之后，再使用这两种方法进行优化才是有意义的。所以当你刚开始构建你的应用时，先不要急着使用 defer() 方法，等你已经写完查询并且分析成哪些方面是热点应用以后，再用也不迟。

only(*fields)

这部分是在 Django 1.1 中新增的： 请查看版本文档
only() 方法或多或少与 defer() 的作用相反。如果你在提取数据时希望某个字段不应该被延后载入，而应该立即载入，那么你就可以做使用 only() 方法。如果你一个 model ，你希望它所有的字段都延后加载，只有某几个字段是立即载入的，那么你就应该使用 only() 方法。

如果你有一个 model，它有 name, age 和 biography 三个字段，那么下面两种写法效果一样的：

Person.objects.defer("age", "biography")
Person.objects.only("name")
你无论何时调用 only()，它都会立刻更改载入设置。这与它的命名非常相符：只有 only 中的字段会立即载入，而其他的则都是延后载入的。因此，连续调用 only() 时，只有最后一个 only 方法才会生效：

# This will defer all fields except the headline.
Entry.objects.only("body", "lede").only("headline")
由于 defer() 可以递增（每次都添加字段到延后载入的列表中），所以你可以将 only() 和 defer() 结合在一起使用，请注意使用顺序，先 only 而后 defer：

# Final result is that everything except "headline" is deferred.
Entry.objects.only("headline", "body").defer("body")

# Final result loads headline and body immediately (only() replaces any
# existing set of fields).
Entry.objects.defer("body").only("headline", "body")
不返回查询的方法(QuerySet methods that do not return QuerySets)

下面所列的 QuerySet 方法作用于 QuerySet，却并不返回 other than a QuerySet。

这些方法并不使用缓存(请查看 缓存与查询(Caching and QuerySets))。所以它们在运行时是立即读取数据库的。

get(**kwargs)

返回与所给的筛选条件相匹配的对象，筛选条件在 字段筛选条件(Field lookups) 一节中有详细介绍。

在使用 get() 时，如果符合筛选条件的对象超过一个，就会抛出 MultipleObjectsReturned 异常。MultipleObjectsReturned 是 model 类的一个属性。

在使用 get() 时，如果没有找到符合筛选条件的对象，就会抛出 DoesNotExist 异常。这个异常也是 model 对象的一个属性。例如：

Entry.objects.get(id='foo') # raises Entry.DoesNotExist
DoesNotExist 异常继承自 django.core.exceptions.ObjectDoesNotExist，所以你可以直接截获 DoesNotExist 异常。例如：

from django.core.exceptions import ObjectDoesNotExist
try:
    e = Entry.objects.get(id=3)
    b = Blog.objects.get(id=1)
except ObjectDoesNotExist:
    print "Either the entry or blog doesn't exist."
create(**kwargs)

创建对象并同时保存对象的快捷方法：

p = Person.objects.create(first_name="Bruce", last_name="Springsteen")
和

p = Person(first_name="Bruce", last_name="Springsteen")
p.save(force_insert=True)
是相同的。

force_insert 参数在别处有详细介绍，它表示把当前 model 当成一个新对象来创建。一般情况下，你不必担心这一点，但是如果你的 model 的主键是你手动指定的，而且它的值已经在数据库中存在，那么调用 create() 就会失败，并抛出 IntegrityError。这是因为主键值必须是唯一的。所以当你手动指定主键时，记得要做好处理异常的准备。

get_or_create(**kwargs)

这是一个方便实际应用的方法，它根据所给的筛选条件查询对象，如果对象不存在就创建一个新对象。

它返回的是一个 (object, created) 元组，其中的 object 是所读取或是创建的对象，而 created 则是一个布尔值，它表示前面提到的 object 是否是新创建的。

这意味着它可以有效地减少代码，并且对编写数据导入脚本非常有用。例如：

try:
    obj = Person.objects.get(first_name='John', last_name='Lennon')
except Person.DoesNotExist:
    obj = Person(first_name='John', last_name='Lennon', birthday=date(1940, 10, 9))
    obj.save()
上面的代码会随着 model 中字段数量的激增而变得愈发庸肿。接下来用 get_or_create() 重写：

obj, created = Person.objects.get_or_create(first_name='John', last_name='Lennon',
                  defaults={'birthday': date(1940, 10, 9)})
在这里要注意 defaults 是一个字典，它仅适用于创建对象时为字段赋值，而并不适用于查找已存在的对象。 get_or_create() 所接收的关键字参数都会在调用 get() 时被使用，有一个参数例外，就是 defaults。在使用get_or_create() 时如果找到了对象，就会返回这个对象和 False。如果没有找到，就会实例化一个新对象，并将其保存；同时返回这个新对象和 True。创建新对象的步骤大致如下：

defaults = kwargs.pop('defaults', {})
params = dict([(k, v) for k, v in kwargs.items() if '__' not in k])
params.update(defaults)
obj = self.model(**params)
obj.save()
用自然语言描述：从非 'defaults' 关键字参数中排除含有双下划线的参数（因为双下划线表示非精确查询），然后再添加 defaults 字典的内容，如果键名与已有的关键字参数重复，就以 defaults 中的内容为准, 然后将整理后的关键字参数传递给 model 类。当然，这只是算法的简化描述，实际上对很多细节没有提及，比如对异常和边界条件的处理。如果你对此感兴趣，不妨看一下原代码。

如果你的 model 恰巧有一个字段，名称正是 defaults，而且你想在 get_or_create() 中用它做为精确查询的条件, 就得使用 'defaults__exact' (之前提过 defaults 只能在创建时对对象赋值，而不能进行查询)，象下面这样：

Foo.objects.get_or_create(defaults__exact='bar', defaults={'defaults': 'baz'})
如果你手动指定了主键，那么使用 get_or_create() 方法时也会象 create() 一样，抛出类似的异常。当你手动指定了主键，若主键值已经在数据库中存在，就会抛出一个 IntegrityError 异常。

最后提一下在 Django 视图(views)中使用 get_or_create() 时要注意的一点。如上所说，对于在脚本中分析数据和添加新数据而言，get_or_create() 是非常有用的。但是如果你是在视图中使用 get_or_create() ，那么就要格外留意，要确认是在 POST 请求中使用，除非你有很必要和很充分的理由才不这么做。而在 GET 请求中使用的话，不会对数据产生任何作用。而使用 POST 的话，每个发往页面的请求都会对数据有一定的副作用。要了解更多，请查看 HTTP 规范中的 安全方法(Safe methods)。

count()

返回数据库中匹配查询(QuerySet)的对象数量。 count() 不会抛出任何异常。

例如：

# Returns the total number of entries in the database.
Entry.objects.count()

# Returns the number of entries whose headline contains 'Lennon'
Entry.objects.filter(headline__contains='Lennon').count()
count() 会在后端执行 SELECT COUNT(*) 操作，所以你应该尽量使用 count() 而不是对返回的查询结果使用 len() 。

根据你所使用的数据库(例如 PostgreSQL 和 MySQL)，count() 可能会返回长整型，而不是普通的 Python 整数。这确实是一个很古怪的举措，没有什么实际意义。

in_bulk(id_list)

接收一个主键值列表，然后根据每个主键值所其对应的对象，返回一个主键值与对象的映射字典。

Example:

>>> Blog.objects.in_bulk([1])
{1: <Blog: Beatles Blog>}
>>> Blog.objects.in_bulk([1, 2])
{1: <Blog: Beatles Blog>, 2: <Blog: Cheddar Talk>}
>>> Blog.objects.in_bulk([])
{}
如果你给 in_bulk() 传递的是一个空列表明，得到就是一个空字典。

iterator()

运行查询(QuerySet)，然后根据结果返回一个 迭代器(iterator。 做为比较，使用 QuerySet 时，从数据库中读取所有记录后，一次性将所有记录实例化为对应的对象；而 iterator() 则是读取记录后，是分多次对数据实例化，用到哪个对象才实例化哪个对象。相对于一次性返回很多对象的 QuerySet，使用迭代器不仅效率更高，而且更节省内存。

要注意的是，如果将 iterator() 作用于 QuerySet，那就意味着会再一次运行查询，就是说会运行两次查询。

latest(field_name=None)

根据时间字段 field_name 得到最新的对象。

下面这个例子根据 pub_date 字段得到数据表中最新的 Entry 对象：

Entry.objects.latest('pub_date')
如果你在 model 中 Meta 定义了 get_latest_by 项, 那么你可以略去 field_name 参数。Django 会将 get_latest_by 做为默认设置。

和 get(), latest() 一样，如果根据所给条件没有找到匹配的对象，就会抛出 DoesNotExist 异常。

注意 latest() 是纯粹为了易用易读而存在的方法。

aggregate(*args, **kwargs)

这部分是在 Django 1.1 中新增的： 请查看版本文档
通过对 QuerySet 进行计算，返回一个聚合值的字典。 aggregate() 中每个参数都指定一个包含在字典中的返回值。

在下面的 聚合函式(Aggregation Functions) 中有对聚合函式的详细描述。

聚合使用关键字参数做为注解的名称。每个参数都有一个为其订做的名称，这个名称取决于聚合函式的函数名和聚合字段的名称。

例如，你正在处理博文，你想知道博客中一共有多少篇博文：

>>> q = Blog.objects.aggregate(Count('entry'))
{'entry__count': 16}
通过在 aggregate 指定关键字参数，你可以控制返回的聚合名称：

>>> q = Blog.objects.aggregate(number_of_entries=Count('entry'))
{'number_of_entries': 16}
要更深入的了解 aggregation，请查看 聚合指南(the topic guide on Aggregation).

exists()

这是在 Django 开发版中新增的。
如果 QuerySet 包含有数据，就返回 True 否则就返回 False。这可能是最快最简单的查询方法了，但它的确会运行查询。某种程度上这意味着使用 QuerySet.exists() 比 bool(some_query_set) 更快。

字段筛选条件(Field lookups)

字段筛选条件决定了你如何构造 SQL 语句中的 WHERE 从句。它们被指定为 QuerySet 中 filter()，exclude() 和 get() 方法的关键字参数。

要了解这部分内容，可以查看 字段查找条件(Field lookups) 一节。

exact

精确匹配。如果指定的值是 None，就会翻译成 SQL 中的 NULL (详情请查看 isnull )。

例如：

Entry.objects.get(id__exact=14)
Entry.objects.get(id__exact=None)
等价的 SQL：

SELECT ... WHERE id = 14;
SELECT ... WHERE id IS NULL;
在 Django 1.0 有所改动： id__exact=None 的语义在 Django 1.0 中已经有所改变。之前，它可以在 SQL 中翻译为 WHERE id = NULL，就是不匹配任何数据。但现在它的作用却和 id__isnull=True 一样。
MySQL 提示

在 MySQL 中，数据表的 "collation" 设置决定了 exact 比较是不是大小写敏感的。这是一个数据库设置，而不是一个 Django 设置。所以更改 MySQL 数据库的设置就能决定是否对大小写敏感，但是具体要不要这么做，仍需要根据实际情况进行权衡考虑。关这方面细节，请查看 数据库(databases) 文档中 collation section 一节。

iexact

忽略大小写的匹配。

例如：

Blog.objects.get(name__iexact='beatles blog')
等价于如下 SQL ：

SELECT ... WHERE name ILIKE 'beatles blog';
要注意它能匹配 'Beatles Blog', 'beatles blog', 'BeAtLes BLoG'，等等。

SQLite 用户要注意

在使用 SQLite 作为数据库，并且应用 Unicode (non-ASCII) 字符串时，请先查看 database note 中关于字符串比对那一节内容。SQLite 对 Unicode 字符串，无法做忽略大小写的匹配。

contains

大小写敏感的包含匹配。

例如：

Entry.objects.get(headline__contains='Lennon')
等价于 SQL ：

SELECT ... WHERE headline LIKE '%Lennon%';
要注意，上述语句将匹配大标题 'Today Lennon honored' ，但不能匹配 'today lennon honored'。

SQLite 不支持大小写敏感的 LIKE 语句；所以对 SQLite 使用 contains 时就和使用 icontains 一样。

icontains

忽略大小写的包含匹配。

例如：

Entry.objects.get(headline__icontains='Lennon')
等价于 SQL：

SELECT ... WHERE headline ILIKE '%Lennon%';
SQLite 用户请注意

使用 SQLite 数据库并应用 Unicode (non-ASCII) 字符串时，请先查看 database note 文档中关于字符串比对那一节内容。

in

是否在一个给定的列表中。

例如：

Entry.objects.filter(id__in=[1, 3, 4])
等价于 SQL：

SELECT ... WHERE id IN (1, 3, 4);
你也可以把查询(queryset)结果当做动态的列表，从而代替固定的列表：

inner_qs = Blog.objects.filter(name__contains='Cheddar')
entries = Entry.objects.filter(blog__in=inner_qs)
做动态列表的 queryset 运行时就会被做为一个子查询：

SELECT ... WHERE blog.id IN (SELECT id FROM ... WHERE NAME LIKE '%Cheddar%')
上面的代码也可以这样写：

inner_q = Blog.objects.filter(name__contains='Cheddar').values('pk').query
entries = Entry.objects.filter(blog__in=inner_q)
在 Django 1.1 中已有发生改变： 在 Django 1.0 中，只有最后一例代码是可用的。
Django 1.0 这种代码形式并不太自然，稍稍有点难读，由于它访问了内部的 query 属性，并且要用到 ValuesQuerySet。如果你的代码不需要兼容 Django 1.0，那么使用第一种形式，直接传递一个 queryset 就好。

如果你传递了一个 ValuesQuerySet 或 ValuesListQuerySet (它们是调用查询集上 values() 和 values_list() 方法的返回结果) 做为 __in 条件的值，那么你要确认只匹配返回结果中的一个字段。例如，下面的代码能正常的工作(对博客名称进行过滤)：

inner_qs = Blog.objects.filter(name__contains='Ch').values('name')
entries = Entry.objects.filter(blog__name__in=inner_qs)
下面的代码却会抛出异常，原因是内部的查询会尝试匹配两个字段值，但只有一个是有用的：

# Bad code! Will raise a TypeError.
inner_qs = Blog.objects.filter(name__contains='Ch').values('name', 'id')
entries = Entry.objects.filter(blog__name__in=inner_qs)
警告

query 属性本是一个不公开的内部属性，虽然他在上面的代码中工作得很好，但是它的API很可能会在不同的 Django 版本中经常变动。

性能考虑

要谨慎使用嵌套查询，并且要对你所采用的数据库性能有所了解(如果不了解，就去做一下性能测试)。有些数据库，比如著名的MySQL，就不能很好地优化嵌套查询。所以在上面的案例中，先在第一个查询中提取值列表，然后再将其传递给第二个查询，会对性能有较高的提升。说白了，就是用两个高效的查询替换掉一个低效的查询：

values = Blog.objects.filter(
        name__contains='Cheddar').values_list('pk', flat=True)
entries = Entry.objects.filter(blog__in=values)
gt

大于。

例如：

Entry.objects.filter(id__gt=4)
等价于 SQL：

SELECT ... WHERE id > 4;
gte

大于等于。

lt

小于。

lte

小于等于。

startswith

大小写敏感的以....开头。

例如：

Entry.objects.filter(headline__startswith='Will')
等价于 SQL：

SELECT ... WHERE headline LIKE 'Will%';
SQLite 不支持大小写敏感的 LIKE 语句；所以在 SQLite 中使用 startswith 就和使用 istartswith 一样。

istartswith

忽略大小写的以....开头。

例如：

Entry.objects.filter(headline__istartswith='will')
等价于 SQL：

SELECT ... WHERE headline ILIKE 'Will%';
SQLite 用户请注意

在使用 SQLite 数据库并应用 Unicode (non-ASCII) 字符串时，请先查看 database note 文档中有关字符串对比那一切的内容。

endswith

大小写敏感的以....结尾。

例如：

Entry.objects.filter(headline__endswith='cats')
等价于 SQL：

SELECT ... WHERE headline LIKE '%cats';
SQLite 不支持大小写敏感的 LIKE 语句；所以在 SQLite 中使用 endswith 就象使用 iendswith 一样。

iendswith

忽略大小写的以....结尾。

例如：

Entry.objects.filter(headline__iendswith='will')
等价于 SQL：

SELECT ... WHERE headline ILIKE '%will'
SQLite users

在使用 SQLite 数据库和应用 Unicode (non-ASCII) 字符串时，请先查看 database note 文档中有关字符串对比那一节内容。

range

包含的范围。

例如：

start_date = datetime.date(2005, 1, 1)
end_date = datetime.date(2005, 3, 31)
Entry.objects.filter(pub_date__range=(start_date, end_date))
等价于 SQL：

SELECT ... WHERE pub_date BETWEEN '2005-01-01' and '2005-03-31';
你可以把 range 当成 SQL 中的 BETWEEN 来用，比如日期，数字，甚至是字符。

year

对日期／时间字段精确匹配年分，年分用四位数字表示。

例如：

Entry.objects.filter(pub_date__year=2005)
等价于 SQL：

SELECT ... WHERE EXTRACT('year' FROM pub_date) = '2005';
(不同的数据库引擎中，翻译得到的 SQL 也不尽相同。)

month

对日期／时间字段精确匹配月分，用整数表示月分，比如 1 表示一月，12 表示十二月。

例如：

Entry.objects.filter(pub_date__month=12)
等价于 SQL：

SELECT ... WHERE EXTRACT('month' FROM pub_date) = '12';
(不同的数据库引擎中，翻译得到的 SQL 也不尽相同。)

day

对日期／时间字段精确匹配日期。

例如：

Entry.objects.filter(pub_date__day=3)
等价于 SQL：

SELECT ... WHERE EXTRACT('day' FROM pub_date) = '3';
(不同的数据库引擎中，翻译得到的 SQL 也不尽相同。)

要注意的是，这个匹配只会得到所有 pub_date 字段内容是表示 某月的第三天 的记录，如一月三号，六月三号。而十月二十三号就不在此列。

week_day

这部分是在 Django 1.1 中新增的： 请查看版本文档
对日期／时间字段匹配星期几

例如：

Entry.objects.filter(pub_date__week_day=2)
等价于 SQL：

SELECT ... WHERE EXTRACT('dow' FROM pub_date) = '2';
(不同的数据库引擎中，翻译得到的 SQL 也不尽相同。)

要注意的是，这段代码将得到 pub_date 字段是星期一的所有记录 (西方习惯于将星期一看做一周的第二天)，与它的年月信息无关。星期以星期天做为第一天，以星期六做为最后一天。

isnull

根据 SQL 查询是空 IS NULL 还是非空 IS NOT NULL，返回相应的 True 或 False。

例如：

Entry.objects.filter(pub_date__isnull=True)
等价于 SQL：

SELECT ... WHERE pub_date IS NULL;
search

利用全文索引做全文搜索。它与 contains 相似，但使用全文索引做搜索会更快一些。

例如：

Entry.objects.filter(headline__search="+Django -jazz Python")
等价于：

SELECT ... WHERE MATCH(tablename, headline) AGAINST (+Django -jazz Python IN BOOLEAN MODE);
要注意这个方法仅适用于 MySQL ，并且要求设置全文索引。默认情况下 Django 使用 BOOLEAN MODE 模式。详见 Please check MySQL documentation for additional details.

regex

这部分是在 Django 1.0 中新增的： 请查看版本文档
大小写敏感的正则表达式匹配。

它要求数据库支持正则表达式语法，而 SQLite 却没有内建正则表达式支持，因此 SQLite 的这个特性是由一个名为 REGEXP 的 Python 方法实现的，所以要用到 Python 的正则库 re.

例如：

Entry.objects.get(title__regex=r'^(An?|The) +')
等价于 SQL：

SELECT ... WHERE title REGEXP BINARY '^(An?|The) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(an?|the) +', 'c'); -- Oracle

SELECT ... WHERE title ~ '^(An?|The) +'; -- PostgreSQL

SELECT ... WHERE title REGEXP '^(An?|The) +'; -- SQLite
建议使用原生字符串 (例如，用 r'foo' 替换 'foo') 做为正则表达式。

iregex

这部分是在 Django 1.0 中新增的： 请查看版本文档
忽略大小写的正则表达式匹配。

匹配：

Entry.objects.get(title__iregex=r'^(an?|the) +')
等价于SQL ：

SELECT ... WHERE title REGEXP '^(an?|the) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(an?|the) +', 'i'); -- Oracle

SELECT ... WHERE title ~* '^(an?|the) +'; -- PostgreSQL

SELECT ... WHERE title REGEXP '(?i)^(an?|the) +'; -- SQLite
聚合函式(Aggregation Functions)

这部分是在 Django 1.1 中新增的： 请查看版本文档
Django 为继续自 django.db.models 的 module 中提供了一系列的聚合函式。要详细了解如何使用这些聚合函式，请查看 the topic guide on aggregation。

Avg

class Avg(field)
返回所给字段的平均值。

默认别名：<field>__avg
返回类型： float
Count

class Count(field, distinct=False)
根据所给的关联字段返回被关联 model 的数量。

默认别名： <field>__count
返回类型： integer
它有一个可选参数：

distinct
如果 distinct=True，那么只返回不重复的实例数量，相当于 SQL 中的 COUNT(DISTINCT field)。默认值是 False。
Max

class Max(field)
返回所给字段的最大值。

默认别名： <field>__max
返回类型： 与所给字段值相同
Min

class Min(field)
返回所给字段的最小值。

默认别名： <field>__min
返回类型： 与所给字段相同
StdDev

class StdDev(field, sample=False)
返回所给字段值的标准差。

默认别名： <field>__stddev
返回类型： float
它有一个可选参数：

sample
默认情况下， StdDev 返回一个总体偏差值，但是如果 sample=True，则返回一个样本偏差值。
SQLite

SQLite 本身并不提供 StdDev 支持，可以使用 SQLite 的外置模块实现这个功能。详情请查看相应的 SQLite 文档，了解如何获得和安装扩展。

Sum

class Sum(field)
计算所给字段值的总和

默认别名： <field>__sum
返回类型： 与所给字段相同
Variance

class Variance(field, sample=False)
返回所给字段值的标准方差。

默认别名： <field>__variance
返回类型： float
它有一个可选参数：

sample
默认情况下， Variance 返回的是总体方差；如果 sample=True，返回的则是样式方差。
SQLite

SQLite 本身并不提供 Variance 支持，可以使用 SQLite 的外置模块实现这个功能。详情请查看相应的 SQLite 文档，了解如何获得和安装扩展。