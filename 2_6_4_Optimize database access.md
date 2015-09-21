{% raw %}

<!--
  译者：Github@wizardforcel
-->

# 数据库访问优化 #

Django的数据库层提供了很多方法来帮助开发者充分的利用他们的数据库。这篇文档收集了相关文档的一些链接，添加了大量提示，并且按照优化数据库使用的步骤的概要来组织。

## 性能优先 ##

作为通用的编程实践，性能的重要性不用多说。弄清楚你在执行什么查询以及你的开销花在哪里。你也可能想使用外部的项目，像django-debug-toolbar，或者直接监控数据库的工具。

记住你可以优化速度、内存占用，甚至二者一起，这取决于你的需求。一些针对其中一个的优化会对另一个不利，但有时会对二者都有帮助。另外，数据库进程做的工作，可能和你在Python代码中做的相同工作不具有相同的开销。决定你的优先级是什么，是你自己的事情，你必须要权衡利弊，按需使用它们，因为这取决于你的应用和服务器。

对于下面提到的任何事情，要记住在任何修改后验证一下，确保修改是有利的，并且足够有利，能超过你代码中可读性的下降。下面的所有建议都带有警告，在你的环境中大体原则可能并不适用，或者会起到相反的效果。

## 使用标准数据库优化技巧 ##

...包括：

+ 索引。在你决定哪些索引应该添加 之后，这一条具有最高优先级。使用Field.db_index或者Meta.index_together在Dhango中添加它们。考虑在你经常使用filter()、exclude()、order_by()和其它方法查询的字段上面添加索引，因为索引有助于加速查找。注意，设计最好的索引方案是一个复杂的、数据库相关的话题，它取决于你应用的细节。持有索引的副作用可能会超过查询速度上的任何收益。
+ 合理使用字段类型。

我们假设你已经完成了上面这些显而易见的事情。这篇文档剩下的部分，着重于讲解如何以不做无用功的方式使用Django。这篇文档也没有强调用在开销大的操作上其它的优化技巧，像general purpose caching。

## 理解查询集 ##

理解查询集(QuerySets) 是通过简单的代码获取较好性能至关重要的一步。特别是：

### 理解查询集计算 ###

要避免性能问题，理解以下几点非常重要：

+ QuerySets是延迟的。
+ 什么时候它们被计算出来。
+ 数据在内存中如何存储。

### 理解缓存属性 ###

和整个QuerySet的缓存相同，ORM对象的属性的结果中也存在缓存。通常来说，不可调用的属性会被缓存。例如下面的博客模型示例：

```
>>> entry = Entry.objects.get(id=1)
>>> entry.blog   # Blog object is retrieved at this point
>>> entry.blog   # cached version, no DB access
```

但是通常来讲，可调用的属性每一次都会访问数据库。

```
>>> entry = Entry.objects.get(id=1)
>>> entry.authors.all()   # query performed
>>> entry.authors.all()   # query performed again
```

要小心当你阅读模板代码的时候 —— 模板系统不允许使用圆括号，但是会自动调用callable对象，会隐藏上述区别。

要小心使用你自定义的属性 —— 实现所需的缓存取决于你，例如使用cached_property装饰符。

### 使用with模板标签 ###

要利用QuerySet的缓存行为，你或许需要使用with模板标签。

### 使用iterator() ###

当你有很多对象时，QuerySet的缓存行为会占用大量的内存。这种情况下，采用iterator()解决。

## 在数据库中而不是Python中做数据库的工作 ##

比如：

+ 在最基础的层面上，使用过滤器和反向过滤器对数据库进行过滤。
+ 使用F 表达式在相同模型中基于其他字段进行过滤。
+ 使用数据库中的注解和聚合。

如果上面那些都不够用，你可以自己生成SQL语句：

### 使用QuerySet.extra() ###

extra()是一个移植性更差，但是功能更强的方法，它允许一些SQL语句显式添加到查询中。如果这些还不够强大：

### 使用原始的SQL ###

编写你自己的自定义SQL语句，来获取数据或者填充模型。使用django.db.connection.queries来了解Django为你编写了什么，以及从这里开始。

## 用唯一的被或索引的列来检索独立对象 ##

有两个原因在get()中，用带有unique或者db_index的列检索独立对象。首先，由于查询经过了数据库的索引，所以会更快。其次，如果很多对象匹配查询，查询会更慢一些；列上的唯一性约束确保这种情况永远不会发生。

所以，使用博客模型的例子：

```
>>> entry = Entry.objects.get(id=10)
```

会快于：

```
>>> entry = Entry.object.get(headline="News Item Title")
```

因为id被数据库索引，而且是唯一的。

下面这样做会十分缓慢：

```
>>> entry = Entry.objects.get(headline__startswith="News")
```

首先， headline没有被索引，它会使查询变得很慢：

其次，这次查找并不确保返回唯一的对象。如果查询匹配到多于一个对象，它会在数据库中遍历和检索所有这些对象。如果记录中返回了成百上千个对象，代价是非常大的。如果数据库运行在分布式服务器上，网络开销和延迟也是一大因素，代价会是它们的组合。

## 一次性检索你需要的任何东西 ##

在不同的位置多次访问数据库，一次获取一个数据集，通常来说不如在一次查询中获取它们更高效。如果你在一个循环中执行查询，这尤其重要。有可能你会做很多次数据库查询，但只需要一次就够了。所以：

### 使用QuerySet.select_related()和prefetch_related() ###

充分了解并使用select_related()和prefetch_related()：

+ 在视图的代码中，
+ 以及在适当的管理器和默认管理器中。要意识到你的管理器什么时候被使用和不被使用；有时这很复杂，所以不要有任何假设。

## 不要获取你不需要的东西 ##

### 使用QuerySet.values()和values_list() ###

当你仅仅想要一个带有值的字典或者列表，并不需要使用ORM模型对象时，可以适当使用values()。对于在模板代码中替换模型对象，这样会非常有用 —— 只要字典中带有的属性和模板中使用的一致，就没问题。

### 使用QuerySet.defer()和only() ###

如果一些数据库的列你并不需要（或者大多数情况下并不需要），使用defer()和only()来避免加载它们。注意如果你确实要用到它们，ORM会在另外的查询之中获取它们。如果你不能够合理地使用这些函数，不如不用。

另外，当建立起一个带有延迟字段的模型时，要意识到一些（小的、额外的）消耗会在Django内部产生。不要不分析数据库就盲目使用延迟字段，因为数据库必须从磁盘中读取大多数非text和VARCHAR数据，在结果中作为单独的一行，即使其中的列很少。 defer()和only()方法在你可以避免加载大量文本数据，或者可能要花大量时间处理而返回给Python的字段时，特别有帮助。像往常一样，应该先写出个大概，之后再优化。

### 使用QuerySet.count() ###

...如果你想要获取大小，不要使用 len(queryset)。

### 使用QuerySet.exists() ###

...如果你想要知道是否存在至少一个结果，不要使用if queryset。

但是：

### 不要过度使用 count() 和 exists() ###

如果你需要查询集中的其他数据，就把它加载出来。

例如，假设Email模型有一个body属性，并且和User有多对多的关联，下面的的模板代码是最优的：

```
{% if display_inbox %}
  {% with emails=user.emails.all %}
    {% if emails %}
      <p>You have {{ emails|length }} email(s)</p>
      {% for email in emails %}
        <p>{{ email.body }}</p>
      {% endfor %}
    {% else %}
      <p>No messages today.</p>
    {% endif %}
  {% endwith %}
{% endif %}
```

这是因为：

+ 因为查询集是延迟加载的，如果‘display_inbox’为False，不会查询数据库。
+ 使用with意味着我们为了以后的使用，把user.emails.all储存在一个变量中，允许它的缓存被复用。
+ {% if emails %}的那一行调用了QuerySet.__bool__()，它导致user.emails.all()查询在数据库上执行，并且至少在第一行以一个ORM对象的形式返回。如果没有任何结果，会返回False，反之为True。
+ {{ emails|length }}调用了QuerySet.__len__()方法，填充了缓存的剩余部分，而且并没有执行另一次查询。
+ for循环的迭代器访问了已经缓存的数据。

总之，这段代码做了零或一次查询。唯一一个慎重的优化就是with标签的使用。在任何位置使用QuerySet.exists()或者QuerySet.count()都会导致额外的查询。

### 使用QuerySet.update()和delete() ###

通过QuerySet.update()使用批量的SQL UPDATE语句，而不是获取大量对象，设置一些值再单独保存。与此相似，在可能的地方使用批量deletes。

但是要注意，这些批量的更新方法不会在单独的实例上面调用save()或者delete()方法，意思是任何你向这些方法添加的自定义行为都不会被执行，包括由普通数据库对象的信号驱动的任何方法。

### 直接使用外键的值 ###

如果你仅仅需要外键当中的一个值，要使用对象上你已经取得的外键的值，而不是获取整个关联对象再得到它的主键。例如，执行：

```
entry.blog_id
```

而不是：

```
entry.blog.id
```

### 不要做无谓的排序 ###

排序并不是没有代价的；每个需要排序的字段都是数据库必须执行的操作。如果一个模型具有默认的顺序（Meta.ordering），并且你并不需要它，通过在查询集上无参调用order_by() 来移除它。

向你的数据库添加索引可能有助于提升排序性能。

## 整体插入 ##

创建对象时，尽可能使用bulk_create()来减少SQL查询的数量。例如：

```
Entry.objects.bulk_create([
    Entry(headline="Python 3.0 Released"),
    Entry(headline="Python 3.1 Planned")
])
```

...更优于：

```
Entry.objects.create(headline="Python 3.0 Released")
Entry.objects.create(headline="Python 3.1 Planned")
```

注意该方法有很多注意事项，所以确保它适用于你的情况。

这也可以用在ManyToManyFields中，所以：

```
my_band.members.add(me, my_friend)
```

...更优于：

```
my_band.members.add(me)
my_band.members.add(my_friend)
```

...其中Bands和Artists具有多对多关联。

{% endraw %}
