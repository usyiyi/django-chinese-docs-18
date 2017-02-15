# QuerySet API参考

本文档描述了`QuerySet` API的详细信息。它建立在[_模型_](../../topics/db/models.html)和[_数据库查询_](../../topics/db/queries.html)指南的基础上，所以在阅读本文档之前，你也许需要首先阅读这两部分的文档。

本文档将通篇使用在[_数据库查询指南_](../../topics/db/queries.html)中用到的[_Weblog 模型的例子_](../../topics/db/queries.html#queryset-model-example)。

## 何时对查询集求值(要写到程序里的字段麻烦不要自作聪明翻译谢谢)

在内部，可以创建、过滤、切片和传递`查询集`而不用真实操作数据库。在你对查询集做求值之前，不会发生任何实际的数据库操作。

你可以使用下列方法对`查询集`求值：

*   **迭代。**`queryset`是可迭代的，它在首次迭代查询集时执行实际的数据库查询。例如， 下面的语句会将数据库中所有Entry 的headline 打印出来：

    ```
    for e in Entry.objects.all():
        print(e.headline)

    ```

    注意：不要使用上面的语句来验证在数据库中是否至少存在一条记录。使用 [`exists()`](#django.db.models.query.QuerySet.exists "django.db.models.query.QuerySet.exists")方法更高效。

*   **切片。** 正如在[_限制查询集_](../../topics/db/queries.html#limiting-querysets)中解释的那样， 可以使用Python 的序列切片语法对一个`查询集`进行分片。一个未求值的`查询集`进行切片通常返回另一个未求值的`查询集`，但是如果你使用切片的”step“参数，Django 将执行数据库查询并返回一个列表。对一个已经求值的`查询集`进行切片将返回一个列表。

    还要注意，虽然对未求值的`查询集`进行切片返回另一个未求值的`查询集`，但是却不可以进一步修改它了（例如，添加更多的Filter，或者修改排序的方式），因为这将不太好翻译成SQL而且含义也不清晰。

*   **序列化/缓存。** [序列化查询集](#pickling-querysets)的细节参见下面一节。本节提到它的目的是强调序列化将读取数据库。

*   **repr()。** 当对`查询集`调用`repr()` 时，将对它求值。这是为了在Python 交互式解释器中使用的方便，这样你可以在交互式使用这个API 时立即看到结果。

*   **len()。** 当你对`查询集`调用`len()` 时， 将对它求值。正如你期望的那样，返回一个查询结果集的长度。

    注：如果你只需要知道集合中记录的个数（并不需要真实的对象），使用数据库层级的`SELECT COUNT(*)` 计数将更加高效。为此，Django 提供了 一个[`count()`](#django.db.models.query.QuerySet.count "django.db.models.query.QuerySet.count") 方法.

*   **list()。** 对`查询集`调用`list()` 将强制对它求值。例如：

    ```
    entry_list = list(Entry.objects.all())

    ```

*   **bool()。** 测试一个`查询集`的布尔值，例如使用`bool()`、`or`、`and` 或者`if` 语句将导致查询集的执行。如果至少有一个记录，则`查询集`为`True`，否则为`False`。例如:

    ```
    if Entry.objects.filter(headline="Test"):
       print("There is at least one Entry with the headline Test")

    ```

    注：如果你需要知道是否存在至少一条记录（而不需要真实的对象），使用 [`exists()`](#django.db.models.query.QuerySet.exists "django.db.models.query.QuerySet.exists") 将更加高效。

### Pickle 查询集

如果你[`Pickle`](https://docs.python.org/3/library/pickle.html#module-pickle "(in Python v3.4)")一个`查询集`，它将在Pickle 之前强制将所有的结果加载到内存中。Pickle 通常用于缓存之前，并且当缓存的查询集重新加载时，你希望结果已经存在随时准备使用（从数据库读取耗费时间，就失去了缓存的目的）。这意味着当你Unpickle`查询集`时，它包含Pickle 时的结果，而不是当前数据库中的结果。

如果此后你只想Pickle 必要的信息来从数据库重新创建`查询集`，可以Pickle`查询集`的`query` 属性。然后你可以使用类似下面的代码重新创建原始的`查询集`（不用加载任何结果）：

```
>>> import pickle
>>> query = pickle.loads(s)     # Assuming 's' is the pickled string.
>>> qs = MyModel.objects.all()
>>> qs.query = query            # Restore the original 'query'.

```

`query` 是一个不透明的对象。它表示查询的内部构造，不属于公开的API。然而，这里讲到的Pickle 和Unpickle 这个属性的内容是安全的（和完全支持的）。

不可以在不同版本之间共享Pickle 的结果

`查询集`的Pickle 只能用于生成它们的Django 版本中。如果你使用Django 的版本N 生成一个Pickle，不保证这个Pickle 在Django 的版本N+1 中可以读取。Pickle 不可用于归档的长期策略。

New in Django 1.8.

因为Pickle 兼容性的错误很难诊断例如产生损坏的对象，当你试图Unpickle 的查询集与Pickle 时的Django 版本不同，将引发一个`RuntimeWarning`。

## 查询集 API

下面是对于`查询集`的正式定义：

_class_ `QuerySet`([_model=None_, _query=None_, _using=None_])[[source]](../../_modules/django/db/models/query.html#QuerySet)

通常你在使用`QuerySet`时会以[_链式的filter_](../../topics/db/queries.html#chaining-filters) 来使用。为了让这个能工作，大部分`QuerySet` 方法返回新的QuerySet。这些方法在本节将详细讲述。

`QuerySet` 类具有两个公有属性用于内省：

`ordered`[[source]](../../_modules/django/db/models/query.html#QuerySet.ordered)

如果`QuerySet` 是排好序的则为`True` —— 例如有一个[`order_by()`](#django.db.models.query.QuerySet.order_by "django.db.models.query.QuerySet.order_by") 子句或者模型有默认的排序。否则为`False` .

`db`[[source]](../../_modules/django/db/models/query.html#QuerySet.db)

如果现在执行，则返回将使用的数据库。

注

[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet") 存在`query` 参数是为了让具有特殊查询用途的子类如[`GeoQuerySet`](../contrib/gis/geoquerysets.html#django.contrib.gis.db.models.GeoQuerySet "django.contrib.gis.db.models.GeoQuerySet") 可以重新构造内部的查询状态。这个参数的值是查询状态的不透明的表示，不是一个公开的API。简而言之：如果你有疑问，那么你实际上不需要使用它。

### 返回新的查询集 的方法(要写到程序里的字段麻烦不要自作聪明翻译谢谢)

Django 提供了一系列 的`QuerySet`筛选方法，用于改变 `QuerySet` 返回的结果类型或者SQL查询执行的方式。

#### filter

`filter`(_**kwargs_)

返回一个新的`QuerySet`，包含与给定的查询参数匹配的对象。

查找的参数（`**kwargs`）应该满足下文[字段查找](#id4)中的格式。在底层的SQL 语句中，多个参数通过`AND` 连接。

如果你需要执行更复杂的查询（例如，使用`OR` 语句查询），你可以使用[`Q 对象`](#django.db.models.Q "django.db.models.Q")。

#### exclude

`exclude`(_**kwargs_)

返回一个新的`QuerySet`，它包含_不_满足给定的查找参数的对象。

查找的参数（`**kwargs`）应该满足下文[字段查找](#id4)中的格式。 在底层的SQL 语句中，多个参数通过`AND` 连接，然后所有的内容放入`NOT()` 中。

下面的示例排除所有`pub_date` 晚于2005-1-3 且`headline` 为“Hello” 的记录：

```
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3), headline='Hello')

```

用SQL 语句，它等同于：

```
SELECT ...
WHERE NOT (pub_date > '2005-1-3' AND headline = 'Hello')

```

下面的示例排除所有`pub_date` 晚于2005-1-3 或者headline 为“Hello” 的记录：

```
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3)).exclude(headline='Hello')

```

用SQL 语句，它等同于：

```
SELECT ...
WHERE NOT pub_date > '2005-1-3'
AND NOT headline = 'Hello'

```

注意，第二个示例更严格。

如果你需要执行更复杂的查询（例如，使用`OR` 语句查询），你可以使用[`Q 对象`](#django.db.models.Q "django.db.models.Q")。

#### annotate

`annotate`(_*args_, _**kwargs_)

使用提供的[_查询表达式_](expressions.html)Annotate `查询集`中的每个对象。查询表达式可以是一个简单的值、模型（或关联模型）字段的一个引用或对`查询集`中的对象一个聚合函数（平均值、和等）。

New in Django 1.8:

之前版本的Django 值允许聚合函数用作Annotation。现在可以使用各种表达式annotate 一个模型。

`annotate()` 的每个参数都是一个annotation，它将添加到返回的`QuerySet` 中每个对象。

Django 提供的聚合函数在下文的[聚合函数](#id5)文档中讲述。

关键字参数指定的Annotation 将使用关键字作为Annotation 的别名。匿名的参数的别名将基于聚合函数的名称和模型的字段生成。只有引用单个字段的聚合表达式才可以使用匿名参数。其它所有形式都必须用关键字参数。

例如，如果你正在操作一个Blog 列表，你可能想知道每个Blog 有多少Entry：

```
>>> from django.db.models import Count
>>> q = Blog.objects.annotate(Count('entry'))
# The name of the first blog
>>> q[0].name
'Blogasaurus'
# The number of entries on the first blog
>>> q[0].entry__count
42

```

`Blog` 模型本身没有定义`entry__count` 属性，但是通过使用一个关键字参数来指定聚合函数，你可以控制Annotation 的名称：

```
>>> q = Blog.objects.annotate(number_of_entries=Count('entry'))
# The number of entries on the first blog, using the name provided
>>> q[0].number_of_entries
42

```

聚合的深入讨论，参见 [_聚合主题的指南_](../../topics/db/aggregation.html)。

#### order_by

`order_by`(_*fields_)

默认情况下，`QuerySet` 根据模型`Meta` 类的`ordering` 选项排序。你可以使用`order_by` 方法给每个`QuerySet` 指定特定的排序。

例如：

```
Entry.objects.filter(pub_date__year=2005).order_by('-pub_date', 'headline')

```

上面的结果将按照`pub_date` 降序排序，然后再按照`headline` 升序排序。`"-pub_date"` 前面的负号表示_降序_排序。隐式的是升序排序。若要随机排序，请使用`"?"`，像这样：

```
Entry.objects.order_by('?')

```

注：`order_by('?')` 查询可能耗费资源且很慢，这取决于使用的数据库。

若要按照另外一个模型中的字段排序，可以使用查询关联模型时的语法。即通过字段的名称后面跟上两个下划线（`__`），再跟上新模型中的字段的名称，直至你希望连接的模型。例如：

```
Entry.objects.order_by('blog__name', 'headline')

```

如果排序的字段与另外一个模型关联，Django 将使用关联的模型的默认排序，或者如果没有指定[`Meta.ordering`](options.html#django.db.models.Options.ordering "django.db.models.Options.ordering") 将通过关联的模型的主键排序。 例如，因为`Blog` 模型没有指定默认的排序：

```
Entry.objects.order_by('blog')

```

... 等同于：

```
Entry.objects.order_by('blog__id')

```

如果`Blog` 设置`ordering = ['name']`，那么第一个QuerySet 将等同于：

```
Entry.objects.order_by('blog__name')

```

通过关联字段排序QuerySet 还能够不用带来JOIN 产生的花费，方法是引用关联字段的`_id`：

```
# No Join
Entry.objects.order_by('blog_id')

# Join
Entry.objects.order_by('blog__id')

```

New in Django 1.7:

QuerySet 通过关联字段进行排序不用带来JOIN 产生的开销。

你还可以通过调用表达式的`asc()` 或者`desc()`，根据[_查询表达式_](expressions.html)排序：

```
Entry.objects.order_by(Coalesce('summary', 'headline').desc())

```

New in Django 1.8:

增加根据查询表达式排序。

如果你还用到[`distinct()`](#django.db.models.query.QuerySet.distinct "django.db.models.query.QuerySet.distinct")，在根据关联模型中的字段排序时要小心。[`distinct()`](#django.db.models.query.QuerySet.distinct "django.db.models.query.QuerySet.distinct") 中有一个备注讲述关联模型的排序如何对结果产生影响。

注

指定一个多值字段来排序结果（例如，一个[`ManyToManyField`](fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 字段或者[`ForeignKey`](fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey") 字段的反向关联）。

考虑下面的情况：

```
class Event(Model):
   parent = models.ForeignKey('self', related_name='children')
   date = models.DateField()

Event.objects.order_by('children__date')

```

这里，每个`Event` 可能有多个潜在的排序数据；each `Event` with multiple `children` will be returned multiple times into the new `QuerySet` that `order_by()` creates. 换句话说, 用 `order_by()`方法对 `QuerySet`对象进行操作会返回一个扩大版的新QuerySet对象——新增的条目也许并没有什么卵用，你也用不着它们。

因此，当你使用多值字段对结果进行排序时要格外小心。**如果**，您可以确保每个订单项只有一个订购数据，这种方法不会出现问题。如果不确定，请确保结果是你期望的。

没有方法指定排序是否考虑大小写。对于大小写的敏感性，Django 将根据数据库中的排序方式排序结果。

你可以通过[`Lower`](database-functions.html#django.db.models.functions.Lower "django.db.models.functions.Lower")将一个字段转换为小写来排序，它将达到大小写一致的排序：

```
Entry.objects.order_by(Lower('headline').desc())

```

New in Django 1.8:

新增根据表达式如`Lower` 来排序。

如果你不想对查询做任何排序，即使是默认的排序，可以不带参数调用[`order_by()`](#django.db.models.query.QuerySet.order_by "django.db.models.query.QuerySet.order_by")。

你可以通过检查[`QuerySet.ordered`](#django.db.models.query.QuerySet.ordered "django.db.models.query.QuerySet.ordered") 属性来知道查询是否是排序的，如果`QuerySet` 有任何方式的排序它将为`True`。

每个`order_by()` 都将清除前面的任何排序。例如，下面的查询将按照`pub_date` 排序，而不是`headline`：

```
Entry.objects.order_by('headline').order_by('pub_date')

```

警告

排序不是没有开销的操作。添加到排序中的每个字段都将带来数据库的开销。添加的每个外键也都将隐式包含进它的默认排序。

#### reverse

`reverse`()

`reverse()` 方法反向排序QuerySet 中返回的元素。第二次调用`reverse()` 将恢复到原有的排序。

如要获取QuerySet 中最后五个元素，你可以这样做：

```
my_queryset.reverse()[:5]

```

注意，这与Python 中从一个序列的末尾进行切片有点不一样。上面的例子将首先返回最后一个元素，然后是倒数第二个元素，以此类推。如果我们有一个Python 序列，当我们查看`seq[-5:]` 时，我们将一下子得到倒数五个元素。Django 不支持这种访问模型（从末尾进行切片），因为它不可能利用SQL 高效地实现。

同时还要注意，`reverse()` 应该只在一个已经定义排序的`QuerySet` 上调用（例如，在一个定义了默认排序的模型上，或者使用[`order_by()`](#django.db.models.query.QuerySet.order_by "django.db.models.query.QuerySet.order_by") 的时候）。如果`QuerySet` 没有定义排序，调用`reverse()` 将不会有任何效果（在调用`reverse()` 之前没有定义排序，那么调用之后仍保持没有定义）。

#### distinct

`distinct`([_*fields_])

返回一个在SQL 查询中使用`SELECT DISTINCT` 的新`QuerySet`。它将去除查询结果中重复的行。

默认情况下，`QuerySet` 不会去除重复的行。在实际应用中，这一般不是个问题，因为像`Blog.objects.all()` 这样的简单查询不会引入重复的行。但是，如果查询跨越多张表，当对`QuerySet` 求值时就可能得到重复的结果。这时候你应该使用`distinct()`。

注

[`order_by()`](#django.db.models.query.QuerySet.order_by "django.db.models.query.QuerySet.order_by") 调用中的任何字段都将包含在SQL 的 `SELECT` 列中。与`distinct()` 一起使用时可能导致预计不到的结果。如果你根据关联模型的字段排序，这些fields将添加到查询的字段中，它们可能产生本应该是唯一的重复的行。因为多余的列没有出现在返回的结果中（它们只是为了支持排序），有时候看上去像是返回了不明确的结果。

类似地，如果您使用[`values()`](#django.db.models.query.QuerySet.values "django.db.models.query.QuerySet.values")查询来限制所选择的列，则仍然会涉及任何[`order_by()`](#django.db.models.query.QuerySet.order_by "django.db.models.query.QuerySet.order_by")（或默认模型排序）影响结果的唯一性。

这里的道德是，如果你使用`distinct()`小心有关的模型排序。类似地，当一起使用`distinct()`和[`values()`](#django.db.models.query.QuerySet.values "django.db.models.query.QuerySet.values")时，请注意字段在不在[`values()`](#django.db.models.query.QuerySet.values "django.db.models.query.QuerySet.values")

在PostgreSQL上，您可以传递位置参数（`* fields`），以便指定`DISTINCT`应该应用的字段的名称。这转换为`SELECT DISTINCT ON` SQL查询。这里有区别。对于正常的`distinct()`调用，数据库在确定哪些行不同时比较每行中的_每个_字段。对于具有指定字段名称的`distinct()`调用，数据库将仅比较指定的字段名称。

注意

当你指定字段名称时，_必须_在`QuerySet`中提供`order_by()`，而且`order_by()`中的字段必须以`distinct()`中的字段相同开始并且顺序相同。

例如，`SELECT DISTINCT ON （a）`列`a`中的每个值。如果你没有指定一个顺序，你会得到一个任意的行。

示例（除第一个示例外，其他示例都只能在PostgreSQL 上工作）：

```
>>> Author.objects.distinct()
[...]

>>> Entry.objects.order_by('pub_date').distinct('pub_date')
[...]

>>> Entry.objects.order_by('blog').distinct('blog')
[...]

>>> Entry.objects.order_by('author', 'pub_date').distinct('author', 'pub_date')
[...]

>>> Entry.objects.order_by('blog__name', 'mod_date').distinct('blog__name', 'mod_date')
[...]

>>> Entry.objects.order_by('author', 'pub_date').distinct('author')
[...]

```

注释

请记住，[`order_by()`](#django.db.models.query.QuerySet.order_by "django.db.models.query.QuerySet.order_by")使用已定义的任何默认相关模型排序。您可能需要通过关系`_id`或引用字段显式排序，以确保`DISTINCT ON`在`ORDER BY`子句的开头。例如，如果`Blog`模型通过`name`定义[`排序`](options.html#django.db.models.Options.ordering "django.db.models.Options.ordering")：

```
Entry.objects.order_by('blog').distinct('blog')

```

...无法工作，因为查询将按`blog__name`排序，从而使`DISTINCT ON`表达式不匹配。你必须按照关系&lt;cite&gt;_id&lt;/cite&gt;字段（在这种情况下为`blog_id`）或引用的（`blog__pk`）显式排序来确保两个表达式都匹配。

#### values

`values`(_*fields_)

返回一个`ValuesQuerySet` —— `QuerySet` 的一个子类，迭代时返回字典而不是模型实例对象。

每个字典表示一个对象，键对应于模型对象的属性名称。

下面的例子将`values()` 与普通的模型对象进行比较：

```
# This list contains a Blog object.
>>> Blog.objects.filter(name__startswith='Beatles')
[<Blog: Beatles Blog>]

# This list contains a dictionary.
>>> Blog.objects.filter(name__startswith='Beatles').values()
[{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]

```

`values()` 接收可选的位置参数`*fields`，它指定`SELECT` 应该限制哪些字段。如果指定字段，每个字典将只包含指定的字段的键/值。如果没有指定字段，每个字典将包含数据库表中所有字段的键和值。

例如：

```
>>> Blog.objects.values()
[{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}],
>>> Blog.objects.values('id', 'name')
[{'id': 1, 'name': 'Beatles Blog'}]

```

值得注意的几点：

*   如果你有一个字段`foo` 是一个[`ForeignKey`](fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")，默认的`values()` 调用返回的字典将有一个叫做`foo_id` 的键，因为这是保存实际的值的那个隐藏的模型属性的名称（`foo` 属性引用关联的模型）。当你调用`values()` 并传递字段的名称，传递`foo` 或`foo_id` 都可以，得到的结果是相同的（字典的键会与你传递的字段名匹配）。

    例如：

    ```
    &gt;&gt;&gt; Entry.objects.values()
    [{'blog_id': 1, 'headline': 'First Entry', ...}, ...]

    &gt;&gt;&gt; Entry.objects.values('blog')
    [{'blog': 1}, ...]

    &gt;&gt;&gt; Entry.objects.values('blog_id')
    [{'blog_id': 1}, ...]

    ```

*   当`values()` 与[`distinct()`](#django.db.models.query.QuerySet.distinct "django.db.models.query.QuerySet.distinct") 一起使用时，注意排序可能影响最终的结果。详细信息参见[`distinct()`](#django.db.models.query.QuerySet.distinct "django.db.models.query.QuerySet.distinct") 中的备注。

*   如果`values()` 子句位于[`extra()`](#django.db.models.query.QuerySet.extra "django.db.models.query.QuerySet.extra") 调用之后，[`extra()`](#django.db.models.query.QuerySet.extra "django.db.models.query.QuerySet.extra") 中的`select` 参数定义的字段必须显式包含在`values()` 调用中。`values()` 调用后面的[`extra()`](#django.db.models.query.QuerySet.extra "django.db.models.query.QuerySet.extra") 调用将忽略选择的额外的字段。

*   在`values()` 之后调用[`only()`](#django.db.models.query.QuerySet.only "django.db.models.query.QuerySet.only") 和[`defer()`](#django.db.models.query.QuerySet.defer "django.db.models.query.QuerySet.defer") 不太合理，所以将引发一个`NotImplementedError`。

New in Django 1.7:

新增最后一点。以前，在`values()` 之后调用[`only()`](#django.db.models.query.QuerySet.only "django.db.models.query.QuerySet.only") 和[`defer()`](#django.db.models.query.QuerySet.defer "django.db.models.query.QuerySet.defer") 是允许的，但是它要么会崩溃要么返回错误的结果。

`ValuesQuerySet` 用于你知道你只需要字段的一小部分，而不需要用到模型实例对象的函数。只选择用到的字段当然更高效。

最后，要注意`ValuesQuerySet` 是`QuerySet` 的子类，它实现了大部分相同的方法。你可以对它调用`filter()`、`order_by()` 等等。这表示下面的两个调用完全相同：

```
Blog.objects.values().order_by('id')
Blog.objects.order_by('id').values()

```

Django 的作者喜欢将影响SQL 的方法放在前面，然后放置影响输出的方法（例如`values()`），但是实际上无所谓。这是卖弄你个性的好机会。

你可以通过`OneToOneField`、`ForeignKey` 和 `ManyToManyField` 属性反向引用关联的模型的字段：

```
Blog.objects.values('name', 'entry__headline')
[{'name': 'My blog', 'entry__headline': 'An entry'},
     {'name': 'My blog', 'entry__headline': 'Another entry'}, ...]

```

警告

因为[`ManyToManyField`](fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 字段和反向关联可能有多个关联的行，包含它们可能导致结果集的倍数放大。如果你在`values()` 查询中包含多个这样的字段将更加明显，这种情况下将返回所有可能的组合。

#### values_list

`values_list`(_*fields_, _flat=False_)

与`values()` 类似，只是在迭代时返回的是元组而不是字典。每个元组包含传递给`values_list()` 调用的字段的值 —— 所以第一个元素为第一个字段，以此类推。例如：

```
>>> Entry.objects.values_list('id', 'headline')
[(1, 'First entry'), ...]

```

如果只传递一个字段，你还可以传递`flat` 参数。如果为`True`，它表示返回的结果为单个值而不是元组。一个例子会让它们的区别更加清晰：

```
>>> Entry.objects.values_list('id').order_by('id')
[(1,), (2,), (3,), ...]

>>> Entry.objects.values_list('id', flat=True).order_by('id')
[1, 2, 3, ...]

```

如果有多个字段，传递`flat` 将发生错误。

如果你不传递任何值给`values_list()`，它将返回模型中的所有字段，以它们在模型中定义的顺序。

注意，这个方法返回`ValuesListQuerySet`。这个类的行为类似列表。大部分时候它足够用了，但是如果你需要一个真实的Python 列表对象，可以对它调用`list()`，这将会对查询集求值。

#### dates

`dates`(_field_, _kind_, _order='ASC'_)

返回`DateQuerySet` - `QuerySet`，其计算结果为[`datetime.date`](https://docs.python.org/3/library/datetime.html#datetime.date "(in Python v3.4)")对象列表，表示特定种类的所有可用日期`QuerySet`。

`field`应为模型的`DateField`的名称。 `kind`应为`"year"`、`"month"`或`"day"`。隐式的是升序排序。若要随机排序，请使用`"?"`，像这样：

*   `"year"` 返回对应该field的所有不同年份值的list。
*   `“month”`返回字段的所有不同年/月值的列表。
*   `“day”`返回字段的所有不同年/月/日值的列表。

`order`（默认为`“ASC”`）应为`'ASC'`或`'DESC'`。它t指定如何排序结果。

例子：

```
>>> Entry.objects.dates('pub_date', 'year')
[datetime.date(2005, 1, 1)]
>>> Entry.objects.dates('pub_date', 'month')
[datetime.date(2005, 2, 1), datetime.date(2005, 3, 1)]
>>> Entry.objects.dates('pub_date', 'day')
[datetime.date(2005, 2, 20), datetime.date(2005, 3, 20)]
>>> Entry.objects.dates('pub_date', 'day', order='DESC')
[datetime.date(2005, 3, 20), datetime.date(2005, 2, 20)]
>>> Entry.objects.filter(headline__contains='Lennon').dates('pub_date', 'day')
[datetime.date(2005, 3, 20)]

```

#### datetimes

`datetimes`(_field_name_, _kind_, _order='ASC'_, _tzinfo=None_)

返回`QuerySet`，其计算为[`datetime.datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)")对象的列表，表示`QuerySet`内容中特定种类的所有可用日期。

`field_name`应为模型的`DateTimeField`的名称。

`种`应为`“year”`，`“month”`，`“day”`，`“hour” `，`“分钟”`或`“秒”`。结果列表中的每个`datetime.datetime`对象被“截断”到给定的`类型`。

`order`, 默认为`'ASC'`, 可选项为`'ASC'` 或者 `'DESC'`. 这个选项指定了返回结果的排序方式。

`tzinfo`定义在截断之前将数据时间转换到的时区。实际上，给定的datetime具有不同的表示，这取决于使用的时区。此参数必须是[`datetime.tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo "(in Python v3.4)")对象。如果它`无`，Django使用[_当前时区_](../../topics/i18n/timezones.html#default-current-time-zone)。当[`USE_TZ`](../settings.html#std:setting-USE_TZ)为`False`时，它不起作用。

注意

此函数直接在数据库中执行时区转换。因此，您的数据库必须能够解释`tzinfo.tzname(None)`的值。这转化为以下要求：

*   SQLite：install [pytz](http://pytz.sourceforge.net/) - 转换实际上是在Python中执行的。
*   PostgreSQL：没有要求（见[时区](http://www.postgresql.org/docs/current/static/datatype-datetime.html#DATATYPE-TIMEZONES)）。
*   Oracle：无要求（请参阅[选择时区文件](http://docs.oracle.com/cd/B19306_01/server.102/b14225/ch4datetime.htm#i1006667)）。
*   MySQL：安装[pytz](http://pytz.sourceforge.net/)，并使用[mysql_tzinfo_to_sql](http://dev.mysql.com/doc/refman/5.6/en/mysql-tzinfo-to-sql.html)加载时区表。

#### none

`none`()

调用none()将创建一个从不返回任何对象的查询集，并且在访问结果时不会执行任何查询。qs.none()查询集是`EmptyQuerySet`的一个实例。

例子：

```
>>> Entry.objects.none()
[]
>>> from django.db.models.query import EmptyQuerySet
>>> isinstance(Entry.objects.none(), EmptyQuerySet)
True

```

#### all

`all`()

返回当前`QuerySet`（或`QuerySet` 子类） 的_副本_。它可以用于在你希望传递一个模型管理器或`QuerySet` 并对结果做进一步过滤的情况。不管对哪一种对象调用`all()`，你都将获得一个可以工作的`QuerySet`。

当对`QuerySet`进行[_求值_](#when-querysets-are-evaluated)时，它通常会缓存其结果。如果数据库中的数据在`QuerySet`求值之后可能已经改变，你可以通过在以前求值过的`QuerySet`上调用相同的`all()` 查询以获得更新后的结果。

#### select_related

`select_related`(_*fields_)

返回一个`QuerySet`，当执行它的查询时它沿着外键关系查询关联的对象的数据。它会生成一个复杂的查询并引起性能的损耗，但是在以后使用外键关系时将不需要数据库查询。

下面的例子解释了普通查询和`select_related()` 查询的区别。下面是一个标准的查询：

```
# Hits the database.
e = Entry.objects.get(id=5)

# Hits the database again to get the related Blog object.
b = e.blog

```

下面是一个`select_related` 查询：

```
# Hits the database.
e = Entry.objects.select_related('blog').get(id=5)

# Doesn't hit the database, because e.blog has been prepopulated
# in the previous query.
b = e.blog

```

`select_related()` 可用于任何对象的查询集：

```
from django.utils import timezone

# Find all the blogs with entries scheduled to be published in the future.
blogs = set()

for e in Entry.objects.filter(pub_date__gt=timezone.now()).select_related('blog'):
    # Without select_related(), this would make a database query for each
    # loop iteration in order to fetch the related blog for each entry.
    blogs.add(e.blog)

```

`filter()` 和`select_related()` 链的顺序不重要。下面的查询集是等同的：

```
Entry.objects.filter(pub_date__gt=timezone.now()).select_related('blog')
Entry.objects.select_related('blog').filter(pub_date__gt=timezone.now())

```

你可以沿着外键查询。如果你有以下模型：

```
from django.db import models

class City(models.Model):
    # ...
    pass

class Person(models.Model):
    # ...
    hometown = models.ForeignKey(City)

class Book(models.Model):
    # ...
    author = models.ForeignKey(Person)

```

... 那么`Book.objects.select_related('author__hometown').get(id=4)` 调用将缓存关联的`Person` _和_关联的 `City`：

```
b = Book.objects.select_related('author__hometown').get(id=4)
p = b.author         # Doesn't hit the database.
c = p.hometown       # Doesn't hit the database.

b = Book.objects.get(id=4) # No select_related() in this example.
p = b.author         # Hits the database.
c = p.hometown       # Hits the database.

```

在传递给`select_related()` 的字段中，你可以使用任何[`ForeignKey`](fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey") 和[`OneToOneField`](fields.html#django.db.models.OneToOneField "django.db.models.OneToOneField")。

在传递给`select_related` 的字段中，你还可以反向引用[`OneToOneField`](fields.html#django.db.models.OneToOneField "django.db.models.OneToOneField") —— 也就是说，你可以回溯到定义[`OneToOneField`](fields.html#django.db.models.OneToOneField "django.db.models.OneToOneField") 的字段。此时，可以使用关联对象字段的[`related_name`](fields.html#django.db.models.ForeignKey.related_name "django.db.models.ForeignKey.related_name")，而不要指定字段的名称。

有些情况下，你希望对很多对象调用`select_related()`，或者你不知道所有的关联关系。在这些情况下，可以调用不带参数的`select_related()`。它将查找能找到的所有不可为空外键 —— 可以为空的外键必须明确指定。大部分情况下不建议这样做，因为它会使得底层的查询非常复杂并且返回的很多数据都不是真实需要的。

如果你需要清除`QuerySet` 上以前的`select_related` 添加的关联字段，可以传递一个`None` 作为参数：

```
>>> without_relations = queryset.select_related(None)

```

链式调用`select_related` 的工作方式与其它方法类似 —— 也就是说，`select_related('foo', 'bar')` 等同于`select_related('foo').select_related('bar')`。

Changed in Django 1.7:

在以前，后者等同于`select_related('bar')`。

#### prefetch_related

`prefetch_related`(_*lookups_)

返回`QuerySet`，它将在单个批处理中自动检索每个指定查找的相关对象。

这具有与`select_related`类似的目的，两者都被设计为阻止由访问相关对象而导致的数据库查询的泛滥，但是策略是完全不同的。

`select_related`通过创建SQL连接并在`SELECT`语句中包括相关对象的字段来工作。因此，`select_related`在同一数据库查询中获取相关对象。然而，为了避免由于跨越“多个”关系而导致的大得多的结果集，`select_related`限于单值关系 - 外键和一对一关系。

`prefetch_related`，另一方面，为每个关系单独查找，并在Python中“加入”。这允许它预取多对多和多对一对象，除了外键和一对一关系，它们不能使用`select_related`来完成。 `select_related`。它还支持[`GenericRelation`](../contrib/contenttypes.html#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation")和[`GenericForeignKey`](../contrib/contenttypes.html#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey")的预取。

例如，假设您有这些模型：

```
from django.db import models

class Topping(models.Model):
    name = models.CharField(max_length=30)

class Pizza(models.Model):
    name = models.CharField(max_length=50)
    toppings = models.ManyToManyField(Topping)

    def __str__(self):              # __unicode__ on Python 2
        return "%s (%s)" % (self.name, ", ".join(topping.name
                                                 for topping in self.toppings.all()))

```

并运行：

```
>>> Pizza.objects.all()
["Hawaiian (ham, pineapple)", "Seafood (prawns, smoked salmon)"...

```

问题是每次`Pizza .__ str __()`要求`self.toppings.all()`它必须查询数据库，因此`Pizza.objects .all()`将在Pizza `QuerySet`中的**每个**项目的Toppings表上运行查询。

我们可以使用`prefetch_related`减少为只有两个查询：

```
>>> Pizza.objects.all().prefetch_related('toppings')

```

这意味着检索到的每个`Pizza`都会执行`self.toppings.all()`；现在每次调用`self.toppings.all()`，而不是去数据库的项目，它会在预取的`QuerySet`缓存中找到它们填充在单个查询中。

也就是说，所有相关的配料将在单个查询中提取，并用于使具有相关结果的预填充缓存的`QuerySets`；这些`QuerySets`然后在`self.toppings.all()`调用中使用。

`prefetch_related()`中的附加查询在`QuerySet`开始计算并且主查询已执行后执行。

请注意，主要`QuerySet`的结果缓存和所有指定的相关对象将被完全加载到内存中。这改变了`QuerySets`的典型行为，通常尽量避免在需要之前将所有对象加载到内存中，即使在数据库中执行了查询之后。

注意

请记住，与`QuerySets`一样，任何后续的链接方法隐含不同的数据库查询将忽略以前缓存的结果，并使用新的数据库查询检索数据。所以，如果你写下面的话：

```
>>> pizzas = Pizza.objects.prefetch_related('toppings')
>>> [list(pizza.toppings.filter(spicy=True)) for pizza in pizzas]

```

...然后事实，已经预取的`pizza.toppings.all()`不会帮助你。`prefetch_related（'toppings'）`隐含`pizza.toppings.all()`，但`pizza.toppings.filter()`是一个不同的查询。预取的缓存在这里不能帮助；实际上它伤害性能，因为你做了一个你没有使用的数据库查询。所以使用这个功能小心！

您还可以使用正常连接语法来执行相关字段的相关字段。假设我们有一个额外的模型上面的例子：

```
class Restaurant(models.Model):
    pizzas = models.ManyToMany(Pizza, related_name='restaurants')
    best_pizza = models.ForeignKey(Pizza, related_name='championed_by')

```

以下都是合法的：

```
>>> Restaurant.objects.prefetch_related('pizzas__toppings')

```

这将预取所有比萨饼属于餐厅，所有浇头属于那些比萨饼。这将导致总共3个数据库查询 - 一个用于餐馆，一个用于比萨饼，一个用于浇头。

```
>>> Restaurant.objects.prefetch_related('best_pizza__toppings')

```

这将获取最好的比萨饼和每个餐厅最好的披萨的所有浇头。这将在3个数据库查询 - 一个为餐厅，一个为“最佳比萨饼”，一个为一个为浇头。

当然，也可以使用`select_related`来获取`best_pizza`关系，以将查询计数减少为2：

```
>>> Restaurant.objects.select_related('best_pizza').prefetch_related('best_pizza__toppings')

```

由于预取在主查询（其包括`select_related`所需的连接）之后执行，因此它能够检测到`best_pizza`对象已经被提取，并且请跳过重新获取它们。

链接`prefetch_related`调用将累积预取的查找。要清除任何`prefetch_related`行为，请传递`None`作为参数：

```
>>> non_prefetched = qs.prefetch_related(None)

```

使用`prefetch_related`时需要注意的一点是，查询创建的对象可以在它们相关的不同对象之间共享，即单个Python模型实例可以出现在树中的多个点返回的对象。这通常会与外键关系发生。通常这种行为不会是一个问题，并且实际上会节省内存和CPU时间。

虽然`prefetch_related`支持预取`GenericForeignKey`关系，但查询的数量将取决于数据。由于`GenericForeignKey`可以引用多个表中的数据，因此需要对每个引用的表进行一次查询，而不是对所有项进行一次查询。如果尚未提取相关行，则可能会对`ContentType`表执行其他查询。

`prefetch_related`在大多数情况下将使用使用“IN”运算符的SQL查询来实现。这意味着对于一个大的`QuerySet`，可能会生成一个大的“IN”子句，根据数据库，在解析或执行SQL查询时可能会有性能问题。始终为您的使用情况配置文件！

请注意，如果您使用`iterator()`来运行查询，则会忽略`prefetch_related()`调用，因为这两个优化并没有意义。

New in Django 1.7.

您可以使用[`Prefetch`](#django.db.models.Prefetch "django.db.models.Prefetch")对象进一步控制预取操作。

在其最简单的形式中，`Prefetch`等效于传统的基于字符串的查找：

```
>>> Restaurant.objects.prefetch_related(Prefetch('pizzas__toppings'))

```

您可以使用可选的`queryset`参数提供自定义查询集。这可以用于更改查询集的默认顺序：

```
>>> Restaurant.objects.prefetch_related(
...     Prefetch('pizzas__toppings', queryset=Toppings.objects.order_by('name')))

```

或者在适当时调用[`select_related()`](#django.db.models.query.QuerySet.select_related "django.db.models.query.QuerySet.select_related")以进一步减少查询数量：

```
>>> Pizza.objects.prefetch_related(
...     Prefetch('restaurants', queryset=Restaurant.objects.select_related('best_pizza')))

```

您还可以使用可选的`to_attr`参数将预取结果分配给自定义属性。结果将直接存储在列表中。

这允许使用不同的`QuerySet`预取相同的关系多次；例如：

```
>>> vegetarian_pizzas = Pizza.objects.filter(vegetarian=True)
>>> Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', to_attr='menu'),
...     Prefetch('pizzas', queryset=vegetarian_pizzas, to_attr='vegetarian_menu'))

```

使用自定义`to_attr`创建的查找仍然可以像往常一样被其他查找遍历：

```
>>> vegetarian_pizzas = Pizza.objects.filter(vegetarian=True)
>>> Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', queryset=vegetarian_pizzas, to_attr='vegetarian_menu'),
...     'vegetarian_menu__toppings')

```

在过滤预取结果时，建议使用`to_attr`，因为它比在相关管理器的缓存中存储过滤的结果更不明确：

```
>>> queryset = Pizza.objects.filter(vegetarian=True)
>>>
>>> # Recommended:
>>> restaurants = Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', queryset=queryset, to_attr='vegetarian_pizzas'))
>>> vegetarian_pizzas = restaurants[0].vegetarian_pizzas
>>>
>>> # Not recommended:
>>> restaurants = Restaurant.objects.prefetch_related(
...     Prefetch('pizzas', queryset=queryset))
>>> vegetarian_pizzas = restaurants[0].pizzas.all()

```

自定义预取也适用于单个相关关系，如前`ForeignKey`或`OneToOneField`。一般来说，您希望对这些关系使用[`select_related()`](#django.db.models.query.QuerySet.select_related "django.db.models.query.QuerySet.select_related")，但有很多情况下使用自定义`QuerySet`进行预取是有用的：

*   您想要使用在相关模型上执行进一步预取的`QuerySet`。

*   您希望仅预取相关对象的子集。

*   You want to use performance optimization techniques like [`deferred fields`](#django.db.models.query.QuerySet.defer "django.db.models.query.QuerySet.defer"):

    ```
    &gt;&gt;&gt; queryset = Pizza.objects.only('name')
    &gt;&gt;&gt;
    &gt;&gt;&gt; restaurants = Restaurant.objects.prefetch_related(
    ...     Prefetch('best_pizza', queryset=queryset))

    ```

注意

查找的顺序很重要。

请看下面的例子：

```
>>> prefetch_related('pizzas__toppings', 'pizzas')

```

即使它是无序的，因为`'pizzas__toppings'`已经包含所有需要的信息，因此第二个参数`'pizzas'`实际上是多余的。

```
>>> prefetch_related('pizzas__toppings', Prefetch('pizzas', queryset=Pizza.objects.all()))

```

这将引发`ValueError`，因为尝试重新定义先前查看的查询的查询集。请注意，创建了隐式查询集，以作为`'pizzas__toppings'`查找的一部分遍历`'pizzas'`。

```
>>> prefetch_related('pizza_list__toppings', Prefetch('pizzas', to_attr='pizza_list'))

```

这会触发`AttributeError`，因为`'pizza_list'`在处理`'pizza_list__toppings'`时不存在。

这种考虑不限于使用`Prefetch`对象。一些高级技术可能需要以特定顺序执行查找以避免创建额外的查询；因此建议始终仔细订购`prefetch_related`参数。

#### extra

`extra`(_select=None_, _where=None_, _params=None_, _tables=None_, _order_by=None_, _select_params=None_)

有些情况下，Django的查询语法难以简单的表达复杂的 `WHERE` 子句，对于这种情况, Django 提供了 `extra()` `QuerySet` 修改机制 — 它能在 `QuerySet`生成的SQL从句中注入新子句

警告

无论何时你都需要非常小心的使用`extra()`. 每次使用它时，您都应该转义用户可以使用`params`控制的任何参数，以防止SQL注入攻击。请详细了解[_SQL injection protection_](../../topics/security.html#sql-injection-protection)。

由于产品差异的原因，这些自定义的查询难以保障在不同的数据库之间兼容(因为你手写 SQL 代码的原因)，而且违背了 DRY 原则，所以如非必要，还是尽量避免写 extra。

extra可以指定一个或多个 `参数`,例如 `select`, `where` or `tables`. 这些参数都不是必须的，但是你至少要使用一个

*   `select`

    The `select` 参数可以让你在 `SELECT` 从句中添加其他字段信息，它应该是一个字典，存放着属性名到 SQL 从句的映射。

    例：

    ```
    Entry.objects.extra(select={'is_recent': "pub_date &gt; '2006-01-01'"})

    ```

    结果集中每个 `Entry` 对象都有一个额外的属性`is_recent`, 它是一个布尔值，表示 Entry对象的`pub_date` 是否晚于 Jan. 1, 2006.

    Django 会直接在 `SELECT` 中加入对应的 SQL 片断，所以转换后的 SQL 如下：

    ```
    SELECT blog_entry.*, (pub_date &gt; '2006-01-01') AS is_recent
    FROM blog_entry;

    ```

    下面这个例子更复杂一些；它会在每个 `Blog`对象中添加一个 `entry_count` 属性，它会运行一个子查询，得到相关联的 `Entry` 对象的数量：

    ```
    Blog.objects.extra(
        select={
            'entry_count': 'SELECT COUNT(*) FROM blog_entry WHERE blog_entry.blog_id = blog_blog.id'
        },
    )

    ```

    在上面这个特例中，我们要了解这个事实，就是 `blog_blog` 表已经存在于`FROM`从句中

    上面例子的结果SQL将是：

    ```
    SELECT blog_blog.*, (SELECT COUNT(*) FROM blog_entry WHERE blog_entry.blog_id = blog_blog.id) AS entry_count
    FROM blog_blog;

    ```

    要注意的是，大多数数据库需要在子句两端添加括号，而在 Django 的`select`从句中却无须这样。 另请注意，某些数据库后端（如某些MySQL版本）不支持子查询。

    在少数情况下，您可能希望将参数传递到`extra（select = ...）`中的SQL片段。为此，请使用`select_params`参数。由于`select_params`是一个序列，并且`select`属性是字典，因此需要注意，以便参数与额外的选择片段正确匹配。在这种情况下，您应该使用[`collections.`](https://docs.python.org/3/library/collections.html#collections.OrderedDict "(in Python v3.4)")OrderedDict用于`select`值，而不仅仅是一个普通的Python字典。

    这将工作，例如：

    ```
    Blog.objects.extra(
        select=OrderedDict([('a', '%s'), ('b', '%s')]),
        select_params=('one', 'two'))

    ```

    如果您需要在选择字符串中使用文本`%s`，请使用序列`%%s`。

    Changed in Django 1.8:

    在1.8之前，您无法逃离文本`%s`。

*   `where` / `tables`

    您可以使用`其中`定义显式SQL `WHERE`子句 - 也许执行非显式连接。您可以使用`表`手动将表添加到SQL `FROM`子句。

    `其中`和`表`都接受字符串列表。所有`其中`参数均为“与”任何其他搜索条件。

    例：

    ```
    Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])

    ```

    ...翻译（大致）以下SQL：

    ```
    SELECT * FROM blog_entry WHERE (foo='a' OR bar='a') AND (baz='a')

    ```

    如果您要指定已在查询中使用的表，请在使用`tables`参数时小心。当您通过`tables`参数添加额外的表时，Django假定您希望该表包含额外的时间（如果已包括）。这会产生一个问题，因为表名将会被赋予一个别名。如果表在SQL语句中多次出现，则第二次和后续出现必须使用别名，以便数据库可以区分它们。如果您指的是在额外的`where`参数中添加的额外表，这将导致错误。

    通常，您只需添加尚未显示在查询中的额外表。然而，如果发生上述情况，则有几种解决方案。首先，看看你是否可以不包括额外的表，并使用已经在查询中的一个。如果不可能，请将`extra()`调用放在查询集结构的前面，以便您的表是该表的第一次使用。最后，如果所有其他失败，请查看生成的查询并重写`where`添加以使用给您的额外表的别名。每次以相同的方式构造查询集时，别名将是相同的，因此您可以依靠别名不更改。

*   `order_by`

    如果您需要使用通过`extra()`包含的一些新字段或表来对结果查询进行排序，请使用`order_by`参数`extra()`这些字符串应该是模型字段（如查询集上的正常[`order_by()`](#django.db.models.query.QuerySet.order_by "django.db.models.query.QuerySet.order_by")方法），形式为`table_name.column_name`或您在`select`参数到`extra()`。

    例如：

    ```
    q = Entry.objects.extra(select={'is_recent': "pub_date &gt; '2006-01-01'"})
    q = q.extra(order_by = ['-is_recent'])

    ```

    这会将`is_recent`的所有项目排序到结果集的前面（`True`在`False`之前按降序排序）。

    顺便说一句，你可以对`extra()`进行多次调用，它会按照你的期望（每次添加新的约束）运行。

*   `params`

    上述`where`参数可以使用标准Python数据库字符串占位符 - `'%s'`来指示数据库引擎应自动引用的参数。`params`参数是要替换的任何额外参数的列表。

    例：

    ```
    Entry.objects.extra(where=['headline=%s'], params=['Lennon'])

    ```

    始终使用`params`而不是将值直接嵌入`where`，因为`params`会确保根据您的特定后端正确引用值。例如，引号将被正确转义。

    坏：

    ```
    Entry.objects.extra(where=["headline='Lennon'"])

    ```

    好：

    ```
    Entry.objects.extra(where=['headline=%s'], params=['Lennon'])

    ```

警告

如果您正在对MySQL执行查询，请注意，MySQL的静默类型强制可能会在混合类型时导致意外的结果。If you query on a string type column, but with an integer value, MySQL will coerce the types of all values in the table to an integer before performing the comparison.例如，如果表包含值`'abc'`，`'def'`，并查询`WHERE mycolumn = 0`，两行都将匹配。为了防止这种情况，请在使用查询中的值之前执行正确的类型转换。

#### defer

`defer`(_*fields_)

在一些复杂的数据建模情况下，您的模型可能包含大量字段，其中一些可能包含大量数据（例如，文本字段），或者需要昂贵的处理来将它们转换为Python对象。如果您在某些情况下使用查询集的结果，当您最初获取数据时不知道是否需要这些特定字段，可以告诉Django不要从数据库中检索它们。

这是通过传递字段名称不加载到`defer()`：

```
Entry.objects.defer("headline", "body")

```

具有延迟字段的查询集仍将返回模型实例。如果您访问该字段（一次一个，而不是一次所有的延迟字段），将从数据库中检索每个延迟字段。

您可以多次调用`defer()`。每个调用都向延迟集添加新字段：

```
# Defers both the body and headline fields.
Entry.objects.defer("body").filter(rating=5).defer("headline")

```

字段添加到延迟集的顺序无关紧要。调用具有已延迟的字段名称的`defer()`是无害的（该字段仍将被延迟）。

您可以使用标准的双下划线符号来分隔相关字段，从而推迟相关模型中的字段加载（如果相关模型通过[`select_related()`](#django.db.models.query.QuerySet.select_related "django.db.models.query.QuerySet.select_related")加载）

```
Blog.objects.select_related().defer("entry__headline", "entry__body")

```

如果要清除延迟字段集，请将`None`作为参数传递到`defer()`：

```
# Load all fields immediately.
my_queryset.defer(None)

```

模型中的某些字段不会被延迟，即使您要求它们。你永远不能推迟加载主键。如果您使用[`select_related()`](#django.db.models.query.QuerySet.select_related "django.db.models.query.QuerySet.select_related")检索相关模型，则不应推迟从主模型连接到相关模型的字段的加载，否则将导致错误。

注意

`defer()`方法（及其表兄弟，[`only()`](#django.db.models.query.QuerySet.only "django.db.models.query.QuerySet.only")）仅适用于高级用例。They provide an optimization for when you have analyzed your queries closely and understand _exactly_ what information you need and have measured that the difference between returning the fields you need and the full set of fields for the model will be significant.

即使你认为你是在高级用例的情况下，**只使用defer()，当你不能，在查询集加载时，确定是否需要额外的字段或**。如果您经常加载和使用特定的数据子集，最好的选择是规范化模型，并将未加载的数据放入单独的模型（和数据库表）。如果列_必须_由于某种原因保留在一个表中，请创建一个具有`Meta.managed = / t4&gt;`（请参阅[`managed attribute`](options.html#django.db.models.Options.managed "django.db.models.Options.managed")文档），只包含您通常需要加载和使用的字段否则调用`defer()`。这使得你的代码对读者更加明确，稍微更快一些，并且在Python进程中消耗更少的内存。

例如，这两个模型使用相同的底层数据库表：

```
class CommonlyUsedModel(models.Model):
    f1 = models.CharField(max_length=10)

    class Meta:
        managed = False
        db_table = 'app_largetable'

class ManagedModel(models.Model):
    f1 = models.CharField(max_length=10)
    f2 = models.CharField(max_length=10)

    class Meta:
        db_table = 'app_largetable'

# Two equivalent QuerySets:
CommonlyUsedModel.objects.all()
ManagedModel.objects.all().defer('f2')

```

如果许多字段需要在非托管模型中复制，最好使用共享字段创建抽象模型，然后使非托管模型和托管模型从抽象模型继承。

注意

当对具有延迟字段的实例调用[`save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save")时，仅保存加载的字段。有关详细信息，请参见[`save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save")。

#### only

`only`(_*fields_)

`only()`方法或多或少与[`defer()`](#django.db.models.query.QuerySet.defer "django.db.models.query.QuerySet.defer")相反。您调用它时，应该在检索模型时延迟的字段。如果你有一个模型几乎所有的字段需要延迟，使用`only()`指定补充的字段集可以导致更简单的代码。

假设您有一个包含字段`名称`，`年龄`和`传记`的模型。以下两个查询集是相同的，就延迟字段而言：

```
Person.objects.defer("age", "biography")
Person.objects.only("name")

```

每当您调用`（`）时，_取代_立即加载的字段集。方法的名称是助记符：**只有**这些字段立即加载；其余的都被推迟。因此，对`only()`的连续调用仅导致所考虑的最后字段：

```
# This will defer all fields except the headline.
Entry.objects.only("body", "rating").only("headline")

```

由于`defer()`以递增方式动作（向延迟列表中添加字段），因此您可以将调用结合到`only()`和`defer()`

```
# Final result is that everything except "headline" is deferred.
Entry.objects.only("headline", "body").defer("body")

# Final result loads headline and body immediately (only() replaces any
# existing set of fields).
Entry.objects.defer("body").only("headline", "body")

```

[`defer()`](#django.db.models.query.QuerySet.defer "django.db.models.query.QuerySet.defer")文档注释中的所有注意事项也适用于`only()`。使用它谨慎，只有耗尽了你的其他选项。

使用[`only()`](#django.db.models.query.QuerySet.only "django.db.models.query.QuerySet.only")并省略使用[`select_related()`](#django.db.models.query.QuerySet.select_related "django.db.models.query.QuerySet.select_related")请求的字段也是错误。

注意

当对具有延迟字段的实例调用[`save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save")时，仅保存加载的字段。有关详细信息，请参见[`save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save")。

#### using

`using`(_alias_)

如果你使用多个数据库，这个方法用于控制`QuerySet` 将在哪个数据库上求值。这个方法的唯一参数是数据库的别名，定义在[`DATABASES`](../settings.html#std:setting-DATABASES)。

例如：

```
# queries the database with the 'default' alias.
>>> Entry.objects.all()

# queries the database with the 'backup' alias
>>> Entry.objects.using('backup')

```

#### select_for_update

`select_for_update`(_nowait=False_)

返回一个 queryset  ，会锁定相关行直到事务结束。在支持的数据库上面产生一个`SELECT ...` FORUPDATE语句

例如:

```
entries = Entry.objects.select_for_update().filter(author=request.user)

```

所有匹配的行将被锁定，直到事务结束。这意味着可以通过锁防止数据被其它事务修改。

一般情况下如果其他事务锁定了相关行，那么本查询将被阻塞，直到锁被释放。如果这不是你想要的行为，请使用`select_for_update(nowait=True)`. 这将使查询不阻塞。如果其它事务持有冲突的锁, 那么查询将引发 [`DatabaseError`](../exceptions.html#django.db.DatabaseError "django.db.DatabaseError") 异常.

目前  `postgresql_psycopg2`, `oracle` 和 `mysql` 数据库后端 `select_for_update()`. 但是 MySQL 不支持 `nowait` 参数。显然，用户应该检查后端的支持情况。

当在不支持`nowait`功能的数据库后端(例如 MySql) 使用`nowait=True` 参数调用 `select_for_update()`  时将引发 [`DatabaseError`](../exceptions.html#django.db.DatabaseError "django.db.DatabaseError") 异常. 这是防止意外造成代码被阻塞。

在自动提交模式下使用 `select_for_update()` 将引发 [`TransactionManagementError`](../exceptions.html#django.db.transaction.TransactionManagementError "django.db.transaction.TransactionManagementError") 异常，原因是自动提交模式下不支持锁定行。如果允许这个调用，那么可能造成数据损坏，而且这个功能很容易在事务外被调用。

对于不支持 `SELECT ...` FORUPDATE的后端 (例如SQLite)  select_for_update() 将没有效果。

Changed in Django 1.6.3:

在自动提交模式下，使用`select_for_update()`执行查询现在是一个错误。在1.6系列的早期版本中，它是一个无操作。

警告

Although `select_for_update()` normally fails in autocommit mode, since [`TestCase`](../../topics/testing/tools.html#django.test.TestCase "django.test.TestCase") automatically wraps each test in a transaction, calling `select_for_update()` in a `TestCase` even outside an [`atomic()`](../../topics/db/transactions.html#django.db.transaction.atomic "django.db.transaction.atomic") block will (perhaps unexpectedly) pass without raising a `TransactionManagementError`. 要正确测试`select_for_update()`，您应该使用[`TransactionTestCase`](../../topics/testing/tools.html#django.test.TransactionTestCase "django.test.TransactionTestCase")。

#### raw

`raw`(_raw_query_, _params=None_, _translations=None_)

Changed in Django 1.7:

`raw` 移动到`QuerySet` 类中。以前，它只位于[`Manager`](../../topics/db/managers.html#django.db.models.Manager "django.db.models.Manager") 中。

接收一个原始的SQL 查询，执行它并返回一个`django.db.models.query.RawQuerySet` 实例。这个`RawQuerySet` 实例可以迭代以提供实例对象，就像普通的`QuerySet` 一样。

更多信息参见[_执行原始的SQL 查询_](../../topics/db/sql.html)。

警告

`raw()` 永远触发一个新的查询，而与之前的filter 无关。因此，它通常应该从`Manager` 或一个全新的`QuerySet` 实例调用。

### 不会返回QuerySets的方法

以下`QuerySet`方法评估`QuerySet`并返回_而不是_ a `QuerySet`。

这些方法不使用高速缓存（请参阅[_Caching and QuerySets_](../../topics/db/queries.html#caching-and-querysets)）。这些方法每次被调用的时候都会查询数据库。

#### get

`get`(_**kwargs_)

返回按照查询参数匹配到的对象，参数的格式应该符合 [Field lookups](#id4)的要求.

如果匹配到的对象个数不只一个的话，`get()` 将会触发[`MultipleObjectsReturned`](../exceptions.html#django.core.exceptions.MultipleObjectsReturned "django.core.exceptions.MultipleObjectsReturned") 异常. [`MultipleObjectsReturned`](../exceptions.html#django.core.exceptions.MultipleObjectsReturned "django.core.exceptions.MultipleObjectsReturned") 异常是模型类的属性.

如果根据给出的参数匹配不到对象的话，`get()` 将触发[`DoesNotExist`](instances.html#django.db.models.Model.DoesNotExist "django.db.models.Model.DoesNotExist") 异常. 这个异常是模型类的属性. 例：

```
Entry.objects.get(id='foo') # raises Entry.DoesNotExist

```

[`DoesNotExist`](instances.html#django.db.models.Model.DoesNotExist "django.db.models.Model.DoesNotExist")异常从[`django.core.exceptions.ObjectDoesNotExist`](../exceptions.html#django.core.exceptions.ObjectDoesNotExist "django.core.exceptions.ObjectDoesNotExist")继承，因此您可以定位多个[`DoesNotExist`](instances.html#django.db.models.Model.DoesNotExist "django.db.models.Model.DoesNotExist")异常。例：

```
from django.core.exceptions import ObjectDoesNotExist
try:
    e = Entry.objects.get(id=3)
    b = Blog.objects.get(id=1)
except ObjectDoesNotExist:
    print("Either the entry or blog doesn't exist.")

```

#### create

`create`(_**kwargs_)

一个在一步操作中同时创建对象并且保存的便捷方法. 所以:

```
p = Person.objects.create(first_name="Bruce", last_name="Springsteen")

```

和:

```
p = Person(first_name="Bruce", last_name="Springsteen")
p.save(force_insert=True)

```

是等同的.

参数 [_force_insert_](instances.html#ref-models-force-insert) 在其他的文档中有介绍, 它意味着一个新的对象一定会被创建. 正常情况中，你不必要担心这点. 然而, 如果你的model中有一个你手动设置主键， 并且这个值已经存在于数据库中, 调用 `create()`将会失败并且触发 [`IntegrityError`](../exceptions.html#django.db.IntegrityError "django.db.IntegrityError") 因为主键必须是唯一的. 如果你手动设置了主键，做好异常处理的准备.

#### get_or_create

`get_or_create`(_defaults=None_, _**kwargs_)

一个通过给出的`kwargs` 来查询对象的便捷方法（如果你的模型中的所有字段都有默认值，可以为空），需要的话创建一个对象。

返回一个由`(object, created)`组成的元组，元组中的`object` 是一个查询到的或者是被创建的对象， `created` 是一个表示是否创建了新的对象的布尔值。

这主要用作样板代码的一种快捷方式。例如：

```
try:
    obj = Person.objects.get(first_name='John', last_name='Lennon')
except Person.DoesNotExist:
    obj = Person(first_name='John', last_name='Lennon', birthday=date(1940, 10, 9))
    obj.save()

```

如果模型的字段数量较大的话，这种模式就变的非常不易用了。上面的示例可以用`get_or_create()`重写  :

```
obj, created = Person.objects.get_or_create(first_name='John', last_name='Lennon',
                  defaults={'birthday': date(1940, 10, 9)})

```

任何传递给 `get_or_create()` 的关键字参数，_除了一个可选的_`defaults`，都将传递给[`get()`](#django.db.models.query.QuerySet.get "django.db.models.query.QuerySet.get") 调用。如果查找到一个对象，`get_or_create()` 返回一个包含匹配到的对象以及`False` 组成的元组。如果查找到的对象超过一个以上，`get_or_create` 将引发[`MultipleObjectsReturned`](../exceptions.html#django.core.exceptions.MultipleObjectsReturned "django.core.exceptions.MultipleObjectsReturned")。_如果查找不到对象_， `get_or_create()` 将会实例化并保存一个新的对象，返回一个由新的对象以及`True` 组成的元组。新的对象将会大概按照以下的逻辑创建:

```
params = {k: v for k, v in kwargs.items() if '__' not in k}
params.update(defaults)
obj = self.model(**params)
obj.save()

```

它表示从非`'defaults'` 且不包含双下划线的关键字参数开始（暗示这是一个不精确的查询）。然后将`defaults` 的内容添加进来，覆盖必要的键，并使用结果作为关键字参数传递给模型类。这是对用到的算法的简单描述，但它包含了所有的相关的细节。内部的实现有更多的错误检查并处理一些边缘条件；如果感兴趣，请阅读代码。

如果你有一个名为`defaults`的字段，并且想在`get_or_create()` 是用它作为精确查询，只需要使用`'defaults__exact'`，像这样：

```
Foo.objects.get_or_create(defaults__exact='bar', defaults={'defaults': 'baz'})

```

当你使用手动指定的主键时，`get_or_create()` 方法与[`create()`](#django.db.models.query.QuerySet.create "django.db.models.query.QuerySet.create")方法有相似的错误行为 。如果需要创建一个对象而该对象的主键早已存在于数据库中，[`IntegrityError`](../exceptions.html#django.db.IntegrityError "django.db.IntegrityError") 异常将会被触发。

这个方法假设正确使用原子操作，正确的数据库配置和底层数据库的正确行为。然而，如果数据库级别没有对`get_or_create` 中用到的`kwargs` 强制要求唯一性（参见[`unique`](fields.html#django.db.models.Field.unique "django.db.models.Field.unique") 和 [`unique_together`](options.html#django.db.models.Options.unique_together "django.db.models.Options.unique_together")），这个方法容易导致竞态条件可能会仍具有相同参数的多行同时插入。

如果你正在使用MySQL，请确保使用`READ COMMITTED` 隔离级别而不是默认的`REPEATABLE READ`，否则你将会遇到`get_or_create` 引发[`IntegrityError`](../exceptions.html#django.db.IntegrityError "django.db.IntegrityError") 但对象在接下来的[`get()`](#django.db.models.query.QuerySet.get "django.db.models.query.QuerySet.get") 调用中并不存在的情况。

最后讲一句`get_or_create()` 在Django 视图中的使用。请确保只在`POST` 请求中使用，除非你有充分的理由。`GET` 请求不应该对数据有任何影响。而`POST` 则用于对数据产生影响的请求。更多信息，参见HTTP 细则中的[安全的方法](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.1.1)。

警告

你可以通过[`ManyToManyField`](fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 属性和反向关联使用`get_or_create()`。在这种情况下，你应该限制查询在关联的上下文内部。如果你不一致地使用它，将可能导致完整性问题。

根据下面的模型：

```
class Chapter(models.Model):
    title = models.CharField(max_length=255, unique=True)

class Book(models.Model):
    title = models.CharField(max_length=256)
    chapters = models.ManyToManyField(Chapter)

```

你可以通过Book 的chapters 字段使用`get_or_create()`，但是它只会获取该Book 内部的上下文：

```
>>> book = Book.objects.create(title="Ulysses")
>>> book.chapters.get_or_create(title="Telemachus")
(<Chapter: Telemachus>, True)
>>> book.chapters.get_or_create(title="Telemachus")
(<Chapter: Telemachus>, False)
>>> Chapter.objects.create(title="Chapter 1")
<Chapter: Chapter 1>
>>> book.chapters.get_or_create(title="Chapter 1")
# Raises IntegrityError

```

发生这个错误时因为它尝试通过Book “Ulysses” 获取或者创建“Chapter 1”，但是它不能：关联关系不能获取这个chapter 因为它与这个book 不关联，但因为`title` 字段是唯一的它仍然不能创建。

#### update_or_create

`update_or_create`(_defaults=None_, _**kwargs_)

New in Django 1.7.

一个通过给出的`kwargs` 来更新对象的便捷方法， 如果需要的话创建一个新的对象。`defaults` 是一个由 (field, value) 对组成的字典，用于更新对象。

返回一个由 `(object, created)`组成的元组,元组中的`object` 是一个创建的或者是被更新的对象， `created` 是一个标示是否创建了新的对象的布尔值。

`update_or_create` 方法尝试通过给出的`kwargs` 去从数据库中获取匹配的对象。如果找到匹配的对象，它将会依据`defaults` 字典给出的值更新字段。

这用作样板代码的一种快捷方式。例如：

```
try:
    obj = Person.objects.get(first_name='John', last_name='Lennon')
    for key, value in updated_values.iteritems():
        setattr(obj, key, value)
    obj.save()
except Person.DoesNotExist:
    updated_values.update({'first_name': 'John', 'last_name': 'Lennon'})
    obj = Person(**updated_values)
    obj.save()

```

如果模型的字段数量较大的话，这种模式就变的非常不易用。上面的示例可以用 `update_or_create()` 重写:

```
obj, created = Person.objects.update_or_create(
    first_name='John', last_name='Lennon', defaults=updated_values)

```

`kwargs` 中的名称如何解析的详细描述可以参见[`get_or_create()`](#django.db.models.query.QuerySet.get_or_create "django.db.models.query.QuerySet.get_or_create")。

和上文描述的[`get_or_create()`](#django.db.models.query.QuerySet.get_or_create "django.db.models.query.QuerySet.get_or_create") 一样，这个方式容易导致竞态条件，如果数据库层级没有前置唯一性它会让多行同时插入。

#### bulk_create

`bulk_create`(_objs_, _batch_size=None_)

此方法以有效的方式（通常只有1个查询，无论有多少对象）将提供的对象列表插入到数据库中：

```
>>> Entry.objects.bulk_create([
...     Entry(headline="Django 1.0 Released"),
...     Entry(headline="Django 1.1 Announced"),
...     Entry(headline="Breaking: Django is awesome")
... ])

```

这有一些注意事项：

*   将不会调用模型的`save()`方法，并且不会发送`pre_save`和`post_save`信号。
*   它不适用于多表继承场景中的子模型。
*   如果模型的主键是[`AutoField`](fields.html#django.db.models.AutoField "django.db.models.AutoField")，它不会像`save()`那样检索和设置主键属性。
*   它不适用于多对多关系。

`batch_size`参数控制在单个查询中创建的对象数。默认值是在一个批处理中创建所有对象，除了SQLite，其中默认值为每个查询最多使用999个变量。

#### count

`count`()

返回在数据库中对应的 `QuerySet`.对象的个数。 `count()` 永远不会引发异常。

例：

```
# Returns the total number of entries in the database.
Entry.objects.count()

# Returns the number of entries whose headline contains 'Lennon'
Entry.objects.filter(headline__contains='Lennon').count()

```

`count()`在后台执行`SELECT COUNT（*）` `count()`，而不是将所有的记录加载到Python对象中并在结果上调用`len()`（除非你需要将对象加载到内存中， `len()`会更快）。

根据您使用的数据库（例如PostgreSQL vs. MySQL），`count()`可能返回一个长整型而不是普通的Python整数。这是一个潜在的实现方案，不应该引起任何真实世界的问题。

请注意，如果您想要`QuerySet`中的项目数量，并且还要从中检索模型实例（例如，通过迭代它），使用`len（查询集） `，这不会导致额外的数据库查询，如`count()`。

#### in_bulk

`in_bulk`(_id_list_)

获取主键值的列表，并返回将每个主键值映射到具有给定ID的对象的实例的字典。

例：

```
>>> Blog.objects.in_bulk([1])
{1: <Blog: Beatles Blog>}
>>> Blog.objects.in_bulk([1, 2])
{1: <Blog: Beatles Blog>, 2: <Blog: Cheddar Talk>}
>>> Blog.objects.in_bulk([])
{}

```

如果你传递`in_bulk()`一个空列表，你会得到一个空的字典。

#### iterator

`iterator`()

评估`QuerySet`（通过执行查询），并返回一个迭代器（参见 [**PEP 234**](http://www.python.org/dev/peps/pep-0234)）。`QuerySet`通常在内部缓存其结果，以便重复计算不会导致其他查询。相反，`iterator()`将直接读取结果，而不在`QuerySet`级别执行任何缓存（内部，默认迭代器调用`iterator()`并高速缓存返回值）。对于返回大量只需要访问一次的对象的`QuerySet`，这可以带来更好的性能和显着减少内存。

请注意，在已经评估的`QuerySet`上使用`iterator()`会强制它再次计算，重复查询。

此外，使用`iterator()`会导致先前的`prefetch_related()`调用被忽略，因为这两个优化一起没有意义。

警告

一些Python数据库驱动程序如`psycopg2`如果使用客户端游标（使用`connection.cursor()`实例化和Django的ORM使用）执行缓存。使用`iterator()`不会影响数据库驱动程序级别的缓存。要禁用此缓存，请查看[服务器端游标](http://initd.org/psycopg/docs/usage.html#server-side-cursors)。

#### latest

`latest`(_field_name=None_)

使用作为日期字段提供的`field_name`，按日期返回表中的最新对象。

此示例根据`pub_date`字段返回表中的最新`条目`：

```
Entry.objects.latest('pub_date')

```

如果模型的[_Meta_](../../topics/db/models.html#meta-options)指定[`get_latest_by`](options.html#django.db.models.Options.get_latest_by "django.db.models.Options.get_latest_by")，则可以将`field_name`参数留给`earliest()`或者` latest()`。默认情况下，Django将使用[`get_latest_by`](options.html#django.db.models.Options.get_latest_by "django.db.models.Options.get_latest_by")中指定的字段。

像[`get()`](#django.db.models.query.QuerySet.get "django.db.models.query.QuerySet.get")，`earliest()`和`latest()` raise [`DoesNotExist`](instances.html#django.db.models.Model.DoesNotExist "django.db.models.Model.DoesNotExist")参数。

请注意，`earliest()`和`latest()`仅仅是为了方便和可读性。

#### earliest

`earliest`(_field_name=None_)

除非方向更改，否则像[`latest()`](#django.db.models.query.QuerySet.latest "django.db.models.query.QuerySet.latest")。

#### first

`first`()

返回结果集的第一个对象, 当没有找到时返回`None`.如果 `QuerySet` 没有设置排序,则将会自动按主键进行排序

例：

```
p = Article.objects.order_by('title', 'pub_date').first()

```

笔记:`first()` 是一个简便方法 下面这个例子和上面的代码效果是一样

```
try:
    p = Article.objects.order_by('title', 'pub_date')[0]
except IndexError:
    p = None

```

#### last

`last`()

工作方式类似[`first()`](#django.db.models.query.QuerySet.first "django.db.models.query.QuerySet.first")，只是返回的是查询集中最后一个对象。

#### aggregate（聚合查询）

`aggregate`(_*args_, _**kwargs_)

返回一个字典，包含根据`QuerySet` 计算得到的聚合值（平均数、和等等）。`aggregate()` 的每个参数指定返回的字典中将要包含的值。

Django 提供的聚合函数在下文的[聚合函数](#id5)中讲述。因为聚合也是[_查询表达式_](expressions.html)，你可以组合多个聚合以及值来创建复杂的聚合。

使用关键字参数指定的聚合将使用关键字参数的名称作为Annotation 的名称。匿名的参数的名称将基于聚合函数的名称和模型字段生成。复杂的聚合不可以使用匿名参数，它们必须指定一个关键字参数作为别名。

例如，当你使用Blog Entry 时，你可能想知道对Author 贡献的Blog Entry 的数目：

```
>>> from django.db.models import Count
>>> q = Blog.objects.aggregate(Count('entry'))
{'entry__count': 16}

```

通过使用关键字参数来指定聚合函数，你可以控制返回的聚合的值的名称：

```
>>> q = Blog.objects.aggregate(number_of_entries=Count('entry'))
{'number_of_entries': 16}

```

聚合的深入讨论，参见[_聚合的指南_](../../topics/db/aggregation.html)。

#### exists

`exists`()

如果[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet") 包含任何结果，则返回`True`，否则返回`False`。它会试图用最简单和最快的方法完成查询，但它执行的方法与普通的_QuerySet_ 查询[`确实`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet")几乎相同。

[`exists()`](#django.db.models.query.QuerySet.exists "django.db.models.query.QuerySet.exists") 用于搜寻对象是否在[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet") 中以及[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet") 是否存在任何对象，特别是[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet") 比较大的时候。

查找具有唯一性字段（例如`primary_key`）的模型是否在一个[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet") 中的最高效的方法是：

```
entry = Entry.objects.get(pk=123)
if some_queryset.filter(pk=entry.pk).exists():
    print("Entry contained in queryset")

```

它将比下面的方法快很多，这个方法要求对QuerySet 求值并迭代整个QuerySet：

```
if entry in some_queryset:
   print("Entry contained in QuerySet")

```

若要查找一个QuerySet 是否包含任何元素：

```
if some_queryset.exists():
    print("There is at least one object in some_queryset")

```

将快于：

```
if some_queryset:
    print("There is at least one object in some_queryset")

```

... 但不会快很多（因为这需要很大的QuerySet 以获得效率的提升）。

另外，如果`some_queryset` 还没有求值，但你知道它将在某个时刻求值，那么使用`some_queryset.exists()` 将比简单地使用`bool(some_queryset)` 完成更多的工作（一个查询用于存在性检查，另外一个是后面的求值），后者将求值并检查是否有结果返回。

#### update

`update`(_**kwargs_)

对指定的字段执行SQL更新查询，并返回匹配的行数（如果某些行已具有新值，则可能不等于已更新的行数）。

例如，要对2010年发布的所有博客条目启用评论，您可以执行以下操作：

```
>>> Entry.objects.filter(pub_date__year=2010).update(comments_on=False)

```

（假设您的`输入`模型具有字段`pub_date`和`comments_on`。）

您可以更新多个字段 - 没有多少字段的限制。例如，在这里我们更新`comments_on`和`标题`字段：

```
>>> Entry.objects.filter(pub_date__year=2010).update(comments_on=False, headline='This is old')

```

`update()`方法立即应用，对更新的[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet")的唯一限制是它只能更新模型主表中的列，而不是相关模型。你不能这样做，例如：

```
>>> Entry.objects.update(blog__name='foo') # Won't work!

```

仍然可以根据相关字段进行过滤：

```
>>> Entry.objects.filter(blog__id=1).update(comments_on=True)

```

您不能在[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet")上调用`update()`，该查询已截取一个切片，或者无法再进行过滤。

`update()`方法返回受影响的行数：

```
>>> Entry.objects.filter(id=64).update(comments_on=True)
1

>>> Entry.objects.filter(slug='nonexistent-slug').update(comments_on=True)
0

>>> Entry.objects.filter(pub_date__year=2010).update(comments_on=False)
132

```

如果你只是更新一个记录，不需要对模型对象做任何事情，最有效的方法是调用`update()`，而不是将模型对象加载到内存中。例如，而不是这样做：

```
e = Entry.objects.get(id=10)
e.comments_on = False
e.save()

```

...做这个：

```
Entry.objects.filter(id=10).update(comments_on=False)

```

使用`update()`还可以防止在加载对象和调用`save()`之间的短时间内数据库中某些内容可能发生更改的竞争条件。

Finally, realize that `update()` does an update at the SQL level and, thus, does not call any `save()` methods on your models, nor does it emit the [`pre_save`](../signals.html#django.db.models.signals.pre_save "django.db.models.signals.pre_save") or [`post_save`](../signals.html#django.db.models.signals.post_save "django.db.models.signals.post_save") signals (which are a consequence of calling [`Model.save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save")). 如果你想更新一个具有自定义[`save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save")方法的模型的记录，请循环遍历它们并调用[`save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save")，如下所示：

```
for e in Entry.objects.filter(pub_date__year=2010):
    e.comments_on = False
    e.save()

```

#### delete

`delete`()

对[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet")中的所有行执行SQL删除查询。立即应用`delete()`。您不能在[`QuerySet`](#django.db.models.query.QuerySet "django.db.models.query.QuerySet")上调用`delete()`，该查询已采取切片或以其他方式无法过滤。

例如，要删除特定博客中的所有条目：

```
>>> b = Blog.objects.get(pk=1)

# Delete all the entries belonging to this Blog.
>>> Entry.objects.filter(blog=b).delete()

```

默认情况下，Django的[`ForeignKey`](fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")模拟SQL约束`ON DELETE CASCADE`字，任何具有指向要删除的对象的外键的对象将与它们一起被删除。例如：

```
blogs = Blog.objects.all()
# This will delete all Blogs and all of their Entry objects.
blogs.delete()

```

此级联行为可通过[`ForeignKey`](fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")的[`on_delete`](fields.html#django.db.models.ForeignKey.on_delete "django.db.models.ForeignKey.on_delete")参数自定义。

`delete()`方法执行批量删除，并且不会在模型上调用任何`delete()`方法。但它会为所有已删除的对象（包括级联删除）发出[`pre_delete`](../signals.html#django.db.models.signals.pre_delete "django.db.models.signals.pre_delete")和[`post_delete`](../signals.html#django.db.models.signals.post_delete "django.db.models.signals.post_delete")信号。

Django需要获取对象到内存中以发送信号和处理级联。然而，如果没有级联和没有信号，那么Django可以采取快速路径并删除对象而不提取到内存中。对于大型删除，这可以显着减少内存使用。执行的查询量也可以减少。

设置为[`on_delete`](fields.html#django.db.models.ForeignKey.on_delete "django.db.models.ForeignKey.on_delete") `DO_NOTHING`的外键不会阻止删除快速路径。

请注意，在对象删除中生成的查询是实施详细信息，可能会更改。

#### as_manager

_classmethod_ `as_manager`()

New in Django 1.7.

类方法，返回[`Manager`](../../topics/db/managers.html#django.db.models.Manager "django.db.models.Manager")的实例与`QuerySet`的方法的副本。有关详细信息，请参见[_Creating Manager with QuerySet methods_](../../topics/db/managers.html#create-manager-with-queryset-methods)。

### 字段查找

字段查询是指如何指定SQL `WHERE`子句的内容. 它们通过`查询集`的[`filter()`](#django.db.models.query.QuerySet.filter "django.db.models.query.QuerySet.filter"), [`exclude()`](#django.db.models.query.QuerySet.exclude "django.db.models.query.QuerySet.exclude") and [`get()`](#django.db.models.query.QuerySet.get "django.db.models.query.QuerySet.get")的关键字参数指定.

查阅简介, 请参考 [_模型与数据库查询_](../../topics/db/queries.html#field-lookups-intro).

Django的内置查找列在下面。也可以为模型字段写入[_custom lookups_](../../howto/custom-lookups.html)。

为了方便当没有提供查找类型时（例如`Entry.objects.get(id=14)`），假设查找类型为[`exact`](#std:fieldlookup-exact)。

#### exact

精确匹配。如果为比较提供的值为`None`，它将被解释为SQL `NULL`（有关详细信息，请参阅[`isnull`](#std:fieldlookup-isnull)）。

例子：

```
Entry.objects.get(id__exact=14)
Entry.objects.get(id__exact=None)

```

SQL等价物：

```
SELECT ... WHERE id = 14;
SELECT ... WHERE id IS NULL;

```

MySQL比较

在MySQL中，数据库表的“排序规则”设置确定`exact`比较是否区分大小写。这是一个数据库设置，_而不是_一个Django设置。可以配置MySQL表以使用区分大小写的比较，但涉及一些折衷。有关详细信息，请参阅[_databases_](../databases.html)文档中的[_collation section_](../databases.html#mysql-collation)。

#### iexact

不区分大小写的精确匹配

Changed in Django 1.7:

如果为比较提供的值为`None`，它将被解释为SQL `NULL`（有关详细信息，请参阅[`isnull`](#std:fieldlookup-isnull)）。

例：

```
Blog.objects.get(name__iexact='beatles blog')
Blog.objects.get(name__iexact=None)

```

SQL等价物：

```
SELECT ... WHERE name ILIKE 'beatles blog';
SELECT ... WHERE name IS NULL;

```

请注意，第一个查询将匹配 `'Beatles Blog'`, `'beatles blog'`, `'BeAtLes BLoG'`, etc.

SQLite用户

当使用SQLite后端和Unicode（非ASCII）字符串时，请记住关于字符串比较的[_数据库注释_](../databases.html#sqlite-string-matching)。SQLite不对Unicode字符串进行不区分大小写的匹配。

#### contains

区分敏感遏制试验。

例：

```
Entry.objects.get(headline__contains='Lennon')

```

SQL等效：

```
SELECT ... WHERE headline LIKE '%Lennon%';

```

请注意，这将匹配标题`'Lennon honored today'`，但不符合`'lennon honored today'`.

SQLite用户

SQLite不支持区分大小写的`LIKE`语句；`contains` 用于SQLite的`icontains`。有关详细信息，请参阅[_数据库注释_](../databases.html#sqlite-string-matching)。

#### icontains

不区分大小写的遏制试验。

例：

```
Entry.objects.get(headline__icontains='Lennon')

```

SQL等效：

```
SELECT ... WHERE headline ILIKE '%Lennon%';

```

SQLite用户

当使用SQLite后端和Unicode（非ASCII）字符串时，请记住关于字符串比较的[_数据库注释_](../databases.html#sqlite-string-matching)。

#### in

在给定的列表。

例：

```
Entry.objects.filter(id__in=[1, 3, 4])

```

SQL等效：

```
SELECT ... WHERE id IN (1, 3, 4);

```

您还可以使用查询集动态评估值列表，而不是提供文字值列表：

```
inner_qs = Blog.objects.filter(name__contains='Cheddar')
entries = Entry.objects.filter(blog__in=inner_qs)

```

此查询集将作为subselect语句求值：

```
SELECT ... WHERE blog.id IN (SELECT id FROM ... WHERE NAME LIKE '%Cheddar%')

```

如果您传入`ValuesQuerySet`或`ValuesListQuerySet`（调用`values()`或`values_list()`查询集）作为`__在`查找的值，您需要确保您只提取结果中的一个字段。例如，这将工作（过滤博客名称）：

```
inner_qs = Blog.objects.filter(name__contains='Ch').values('name')
entries = Entry.objects.filter(blog__name__in=inner_qs)

```

这个例子将产生一个异常，由于内查询试图提取两个字段的值，但是查询语句只期望提取一个字段的值：

```
# Bad code! Will raise a TypeError.
inner_qs = Blog.objects.filter(name__contains='Ch').values('name', 'id')
entries = Entry.objects.filter(blog__name__in=inner_qs)

```

性能注意事项

对于使用嵌套查询和了解数据库服务器的性能特征（如果有疑问，去做基准测试）要谨慎。一些数据库后端，最着名的是MySQL，不能很好地优化嵌套查询。在这些情况下，提取值列表然后将其传递到第二个查询中更有效。也就是说，执行两个查询，而不是一个：

```
values = Blog.objects.filter(
        name__contains='Cheddar').values_list('pk', flat=True)
entries = Entry.objects.filter(blog__in=list(values))

```

请注意`list()`调用Blog `QuerySet`以强制执行第一个查询。没有它，将执行嵌套查询，因为[_QuerySet是惰性的_](../../topics/db/queries.html#querysets-are-lazy)。

#### ）

大于

例子:

```
Entry.objects.filter(id__gt=4)

```

SQL语句相当于：

```
SELECT ... WHERE id > 4;

```

#### gte

大于或等于

#### 1

小于

#### lte

小于或等于

#### startswith

区分大小写，开始位置匹配

例：

```
Entry.objects.filter(headline__startswith='Will')

```

SQL等效：

```
SELECT ... WHERE headline LIKE 'Will%';

```

SQLite 不支持区分大小写 `LIKE` 语句; Sqlite 下`startswith` 等于 `istartswith` .

#### istartswith

不区分大小写，开始位置匹配

例：

```
Entry.objects.filter(headline__istartswith='will')

```

SQL等效：

```
SELECT ... WHERE headline ILIKE 'Will%';

```

SQLite用户

当使用SQLite后端和Unicode（非ASCII）字符串时，请记住关于字符串比较的[_数据库注释_](../databases.html#sqlite-string-matching)。

#### endswith

区分大小写。

例：

```
Entry.objects.filter(headline__endswith='cats')

```

SQL等效：

```
SELECT ... WHERE headline LIKE '%cats';

```

SQLite用户

SQLite不支持区分大小写的`LIKE`语句；`endswith`用作SQLite的`iendswith`。有关更多信息，请参阅[_数据库注释_](../databases.html#sqlite-string-matching)文档。

#### iendswith

不区分大小写。

例：

```
Entry.objects.filter(headline__iendswith='will')

```

SQL等效：

```
SELECT ... WHERE headline ILIKE '%will'

```

SQLite用户

当使用SQLite后端和Unicode（非ASCII）字符串时，请记住关于字符串比较的[_数据库注释_](../databases.html#sqlite-string-matching)。

#### range

范围测试（包含于之中）。

例：

```
import datetime
start_date = datetime.date(2005, 1, 1)
end_date = datetime.date(2005, 3, 31)
Entry.objects.filter(pub_date__range=(start_date, end_date))

```

SQL等效：

```
SELECT ... WHERE pub_date BETWEEN '2005-01-01' and '2005-03-31';

```

您可以在任何可以使用`BETWEEN`的SQL中使用`范围`（对于日期，数字和偶数字符）。

警告

过滤具有日期的`DateTimeField`不会包含最后一天的项目，因为边界被解释为“给定日期的0am”。如果`pub_date`是`DateTimeField`，上面的表达式将变成这个SQL：

```
SELECT ... WHERE pub_date BETWEEN '2005-01-01 00:00:00' and '2005-03-31 00:00:00';

```

一般来说，不能混合使用日期和数据时间。

#### year

对于日期和日期时间字段，确切的年匹配。整数年。

例：

```
Entry.objects.filter(pub_date__year=2005)

```

SQL等效：

```
SELECT ... WHERE pub_date BETWEEN '2005-01-01' AND '2005-12-31';

```

（确切的SQL语法因每个数据库引擎而异）。

当[`USE_TZ`](../settings.html#std:setting-USE_TZ)为`True`时，在过滤之前，datetime字段将转换为当前时区。

#### month

对于日期和日期时间字段，确切的月份匹配。取整数1（1月）至12（12月）。

例：

```
Entry.objects.filter(pub_date__month=12)

```

SQL等效：

```
SELECT ... WHERE EXTRACT('month' FROM pub_date) = '12';

```

（确切的SQL语法因每个数据库引擎而异）。

当[`USE_TZ`](../settings.html#std:setting-USE_TZ)为`True`时，在过滤之前，datetime字段将转换为当前时区。这需要数据库中的[_时区定义_](#database-time-zone-definitions)。

#### day(要写到程序里的字段麻烦不要自作聪明翻译谢谢)

对于日期和日期时间字段，具体到某一天的匹配。取一个整数的天数。

例：

```
Entry.objects.filter(pub_date__day=3)

```

SQL等效：

```
SELECT ... WHERE EXTRACT('day' FROM pub_date) = '3';

```

（确切的SQL语法因每个数据库引擎而异）。

请注意，这将匹配每月第三天（例如1月3日，7月3日等）的任何包含pub_date的记录。

当[`USE_TZ`](../settings.html#std:setting-USE_TZ)为`True`时，在过滤之前，datetime字段将转换为当前时区。这需要数据库中的[_time zone definitions in the database_](#database-time-zone-definitions)。

#### week_day

对于日期和日期时间字段，“星期几”匹配。

取整数值，表示星期几从1（星期日）到7（星期六）。

例：

```
Entry.objects.filter(pub_date__week_day=2)

```

（此查找不包括等效的SQL代码片段，因为相关查询的实现因不同数据库引擎而异）。

请注意，这将匹配落在星期一（星期二）的任何记录（`pub_date`），而不管其出现的月份或年份。周日被索引，第1天为星期天，第7天为星期六。

当[`USE_TZ`](../settings.html#std:setting-USE_TZ)为`True`时，在过滤之前，datetime字段将转换为当前时区。这需要数据库中的[_时区定义_](#database-time-zone-definitions)。

#### hour

对于日期时间字段，精确的小时匹配。取0和23之间的整数。

例：

```
Event.objects.filter(timestamp__hour=23)

```

SQL等效：

```
SELECT ... WHERE EXTRACT('hour' FROM timestamp) = '23';

```

（确切的SQL语法因每个数据库引擎而异）。

当[`USE_TZ`](../settings.html#std:setting-USE_TZ)为`True`时，在过滤之前将值转换为当前时区。

#### minute

对于日期时间字段，精确的分钟匹配。取0和59之间的整数。

例：

```
Event.objects.filter(timestamp__minute=29)

```

SQL等效：

```
SELECT ... WHERE EXTRACT('minute' FROM timestamp) = '29';

```

（确切的SQL语法因每个数据库引擎而异）。

当[`USE_TZ`](../settings.html#std:setting-USE_TZ)为`True`时，在过滤之前将值转换为当前时区。

#### second

对于datetime字段，精确的第二个匹配。取0和59之间的整数。

例：

```
Event.objects.filter(timestamp__second=31)

```

等同于SQL语句:

```
SELECT ... WHERE EXTRACT('second' FROM timestamp) = '31';

```

（确切的SQL语法因每个数据库引擎而异）。

当[`USE_TZ`](../settings.html#std:setting-USE_TZ)为`True`时，在过滤之前将值转换为当前时区。

#### isnull

值为 `True` 或 `False`, 相当于 SQL语句`IS NULL`和`IS NOT NULL`.

例：

```
Entry.objects.filter(pub_date__isnull=True)

```

SQL等效：

```
SELECT ... WHERE pub_date IS NULL;

```

#### search

一个Boolean类型的全文搜索，以全文搜索的优势。这个很像 [`contains`](#std:fieldlookup-contains) ，但是由于全文索引的优势，以使它更显著的快。

例：

```
Entry.objects.filter(headline__search="+Django -jazz Python")

```

SQL等效：

```
SELECT ... WHERE MATCH(tablename, headline) AGAINST (+Django -jazz Python IN BOOLEAN MODE);

```

注意，这仅在MySQL中可用，并且需要直接操作数据库以添加全文索引。默认情况下，Django使用BOOLEAN MODE进行全文搜索。有关其他详细信息，请参阅[MySQL文档](http://dev.mysql.com/doc/refman/5.6/en/fulltext-boolean.html)。

#### 正则表达式

区分大小写的正则表达式匹配。

正则表达式语法是正在使用的数据库后端的语法。在SQLite没有内置正则表达式支持的情况下，此功能由（Python）用户定义的REGEXP函数提供，因此正则表达式语法是Python的`re`模块。

例：

```
Entry.objects.get(title__regex=r'^(An?|The) +')

```

SQL等价物：

```
SELECT ... WHERE title REGEXP BINARY '^(An?|The) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(an?|the) +', 'c'); -- Oracle

SELECT ... WHERE title ~ '^(An?|The) +'; -- PostgreSQL

SELECT ... WHERE title REGEXP '^(An?|The) +'; -- SQLite

```

建议使用原始字符串（例如，`r'foo'`而不是`'foo'`）来传递正则表达式语法。

#### iregex

不区分大小写的正则表达式匹配。

例：

```
Entry.objects.get(title__iregex=r'^(an?|the) +')

```

SQL等价物：

```
SELECT ... WHERE title REGEXP '^(an?|the) +'; -- MySQL

SELECT ... WHERE REGEXP_LIKE(title, '^(an?|the) +', 'i'); -- Oracle

SELECT ... WHERE title ~* '^(an?|the) +'; -- PostgreSQL

SELECT ... WHERE title REGEXP '(?i)^(an?|the) +'; -- SQLite

```

### 聚合函数

Django 的`django.db.models` 模块提供以下聚合函数。关于如何使用这些聚合函数的细节，参见[_聚合函数的指南_](../../topics/db/aggregation.html)。关于如何创建聚合函数，参数[`聚合函数`](expressions.html#django.db.models.Aggregate "django.db.models.Aggregate") 的文档。

警告

SQLite 不能直接处理日期/时间字段的聚合。这是因为SQLite 中没有原生的日期/时间字段，Django 目前使用文本字段模拟它的功能。在SQLite 中对日期/时间字段使用聚合将引发`NotImplementedError`。

注

在`QuerySet` 为空时，聚合函数函数将返回`None`。 例如，如果`QuerySet` 中没有记录，`Sum` 聚合函数将返回`None` 而不是`0`。`Count` 是一个例外，如果`QuerySet` 为空，它将返回`0`。

所有聚合函数具有以下共同的参数：

#### 表达

引用模型字段的一个字符串，或者一个[_查询表达式_](expressions.html)。

New in Django 1.8:

现在在复杂的计算中，聚合函数可以引用多个字段。

#### output_field

用来表示返回值的[_模型字段_](fields.html)，它是一个可选的参数。

New in Django 1.8:

添加`output_field` 参数。

注

在组合多个类型的字段时，只有在所有的字段都是相同类型的情况下，Django 才能确定`output_field`。否则，你必须自己提供`output_field` 参数。

#### **额外

这些关键字参数可以给聚合函数生成的SQL 提供额外的信息。

#### avg

_class_ `Avg`(_expression_, _output_field=None_, _**extra_)

返回给定expression 的平均值，其中expression 必须为数值。

*   默认的别名：`&lt;field&gt;__avg`
*   返回类型：`float`

#### Count

_class_ `Count`(_expression_, _distinct=False_, _**extra_)

返回与expression 相关的对象的个数。

*   默认的别名：`&lt;field&gt;__count`
*   返回类型：`int`

有一个可选的参数：

`distinct`

如果`distinct=True`，Count 将只计算唯一的实例。它等同于`COUNT(DISTINCT &lt;field&gt;)` SQL 语句。默认值为`False`。

#### Max

_class_ `Max`(_expression_, _output_field=None_, _**extra_)

返回expression 的最大值。

*   默认的别名：`&lt;field&gt;__max`
*   返回类型：与输入字段的类型相同，如果提供则为 `output_field` 类型

#### Min

_class_ `Min`(_expression_, _output_field=None_, _**extra_)

返回expression 的最小值。

*   默认的别名：`&lt;field&gt;__min`
*   返回的类型：与输入字段的类型相同，如果提供则为 `output_field` 类型

#### StdDev

_class_ `StdDev`(_expression_, _sample=False_, _**extra_)

返回expression 的标准差。

*   默认的别名：`&lt;field&gt;__stddev`
*   返回类型：`float`

有一个可选的参数：

`sample`

默认情况下，`StdDev` 返回群体的标准差。但是，如果`sample=True`，返回的值将是样本的标准差。

SQLite

SQLite 没有直接提供`StdDev`。有一个可用的实现是SQLite 的一个扩展模块。参见[SQlite 的文档](http://www.sqlite.org/contrib) 中获取并安装这个扩展的指南。

#### Sum

_class_ `Sum`(_expression_, _output_field=None_, _**extra_)

计算expression 的所有值的和。

*   默认的别名：`&lt;field&gt;__sum`
*   返回类型：与输入的字段相同，如果提供则为`output_field` 的类型

#### Variance

_class_ `Variance`(_expression_, _sample=False_, _**extra_)

返回expression 的方差。

*   默认的别名：`&lt;field&gt;__variance`
*   返回的类型：`float`

有一个可选的参数：

`sample`

默认情况下，`Variance` 返回群体的方差。但是，如果`sample=True`，返回的值将是样本的方差。

SQLite

SQLite 没有直接提供`Variance`。有一个可用的实现是SQLite 的一个扩展模块。 参见[SQlite 的文档](http://www.sqlite.org/contrib) 中获取并安装这个扩展的指南。

## 查询相关的类

本节提供查询相关的工具的参考资料，它们其它地方没有文档。

### `Q()` 对象

_class_ `Q`

`Q()` 对象和[`F`](expressions.html#django.db.models.F "django.db.models.F") 对象类似，把一个SQL 表达式封装在Python 对象中，这个对象可以用于数据库相关的操作。

通常，`Q() 对象` 使得定义查询条件然后重用成为可能。这允许[_construction of complex database queries_](../../topics/db/queries.html#complex-lookups-with-q)使用`|` (`OR`) 和 `&` (`AND`) 操作符; 否则`QuerySets`中使用不了`OR`。

### `Prefetch()`对象

New in Django 1.7.

_class_ `Prefetch`(_lookup_, _queryset=None_, _to_attr=None_)

`Prefetch()`对象可用于控制[`prefetch_related()`](#django.db.models.query.QuerySet.prefetch_related "django.db.models.query.QuerySet.prefetch_related")的操作。

`lookup`参数描述了跟随的关系，并且工作方式与传递给[`prefetch_related()`](#django.db.models.query.QuerySet.prefetch_related "django.db.models.query.QuerySet.prefetch_related")的基于字符串的查找相同。例如：

```
>>> Question.objects.prefetch_related(Prefetch('choice_set')).get().choice_set.all()
[<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]
# This will only execute two queries regardless of the number of Question
# and Choice objects.
>>> Question.objects.prefetch_related(Prefetch('choice_set')).all()
[<Question: Question object>]

```

`查询集`参数为给定的查找提供基本`QuerySet`。这对于进一步过滤预取操作或从预取关系调用[`select_related()`](#django.db.models.query.QuerySet.select_related "django.db.models.query.QuerySet.select_related")很有用，因此进一步减少查询数量：

```
>>> voted_choices = Choice.objects.filter(votes__gt=0)
>>> voted_choices
[<Choice: The sky>]
>>> prefetch = Prefetch('choice_set', queryset=voted_choices)
>>> Question.objects.prefetch_related(prefetch).get().choice_set.all()
[<Choice: The sky>]

```

`to_attr`参数将预取操作的结果设置为自定义属性：

```
>>> prefetch = Prefetch('choice_set', queryset=voted_choices, to_attr='voted_choices')
>>> Question.objects.prefetch_related(prefetch).get().voted_choices
[<Choice: The sky>]
>>> Question.objects.prefetch_related(prefetch).get().choice_set.all()
[<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]

```

注意

当使用`to_attr`时，预取的结果存储在列表中。这可以提供比存储在`QuerySet`实例内的缓存结果的传统`prefetch_related`调用显着的速度改进。

