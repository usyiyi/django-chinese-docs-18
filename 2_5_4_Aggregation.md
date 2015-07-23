<!--
  来源：http://python.usyiyi.cn/
-->

# 聚合 #

Django数据库抽象API描述了使用Django查询来增删查改单个对象的方法。然而，你有时候会想要获取从一组对象导出的值或者是聚合一组对象。这份指南描述了通过Django查询来生成和返回聚合值的方法。

整篇指南我们都将引用以下模型。这些模型用来记录多个网上书店的库存。

```
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()

class Publisher(models.Model):
    name = models.CharField(max_length=300)
    num_awards = models.IntegerField()

class Book(models.Model):
    name = models.CharField(max_length=300)
    pages = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    rating = models.FloatField()
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    pubdate = models.DateField()

class Store(models.Model):
    name = models.CharField(max_length=300)
    books = models.ManyToManyField(Book)
    registered_users = models.PositiveIntegerField()
```

## 速查表 ##

急着用吗？以下是在上述模型的基础上，进行一般的聚合查询的方法:

```
# Total number of books.
>>> Book.objects.count()
2452

# Total number of books with publisher=BaloneyPress
>>> Book.objects.filter(publisher__name='BaloneyPress').count()
73

# Average price across all books.
>>> from django.db.models import Avg
>>> Book.objects.all().aggregate(Avg('price'))
{'price__avg': 34.35}

# Max price across all books.
>>> from django.db.models import Max
>>> Book.objects.all().aggregate(Max('price'))
{'price__max': Decimal('81.20')}

# Cost per page
>>> Book.objects.all().aggregate(
...    price_per_page=Sum(F('price')/F('pages'), output_field=FloatField()))
{'price_per_page': 0.4470664529184653}

# All the following queries involve traversing the Book<->Publisher
# many-to-many relationship backward

# Each publisher, each with a count of books as a "num_books" attribute.
>>> from django.db.models import Count
>>> pubs = Publisher.objects.annotate(num_books=Count('book'))
>>> pubs
[<Publisher BaloneyPress>, <Publisher SalamiPress>, ...]
>>> pubs[0].num_books
73

# The top 5 publishers, in order by number of books.
>>> pubs = Publisher.objects.annotate(num_books=Count('book')).order_by('-num_books')[:5]
>>> pubs[0].num_books
1323
```

## 在查询集上生成聚合 ##

Django提供了两种生成聚合的方法。第一种方法是从整个查询集生成统计值。比如，你想要计算所有在售书的平均价钱。Django的查询语法提供了一种方式描述所有图书的集合。

```
>>> Book.objects.all()
```

我们需要在QuerySet.对象上计算出总价格。这可以通过在QuerySet后面附加aggregate() 子句来完成。

```
>>> from django.db.models import Avg
>>> Book.objects.all().aggregate(Avg('price'))
{'price__avg': 34.35}
```

all()在这里是多余的，所以可以简化为：

```
>>> Book.objects.aggregate(Avg('price'))
{'price__avg': 34.35}
```

aggregate()子句的参数描述了我们想要计算的聚合值，在这个例子中，是Book 模型中price字段的平均值。查询集参考中列出了聚合函数的列表。

aggregate()是QuerySet 的一个终止子句，意思是说，它返回一个包含一些键值对的字典。键的名称是聚合值的标识符，值是计算出来的聚合值。键的名称是按照字段和聚合函数的名称自动生成出来的。如果你想要为聚合值指定一个名称，可以向聚合子句提供它。

```
>>> Book.objects.aggregate(average_price=Avg('price'))
{'average_price': 34.35}
```

如果你希望生成不止一个聚合，你可以向aggregate()子句中添加另一个参数。所以，如果你也想知道所有图书价格的最大值和最小值，可以这样查询：

```
>>> from django.db.models import Avg, Max, Min
>>> Book.objects.aggregate(Avg('price'), Max('price'), Min('price'))
{'price__avg': 34.35, 'price__max': Decimal('81.20'), 'price__min': Decimal('12.99')}
```

## 为查询集的每一项生成聚合 ##

生成汇总值的第二种方法，是为QuerySet中每一个对象都生成一个独立的汇总值。比如，如果你在检索一列图书，你可能想知道有多少作者写了每一本书。每本书和作者是多对多的关系。我们想要汇总QuerySet.中每本书里的这种关系。

逐个对象的汇总结果可以由annotate()子句生成。当annotate()子句被指定之后，QuerySet中的每个对象都会被注上特定的值。

这些注解的语法都和aggregate()子句所使用的相同。annotate()的每个参数都描述了将要被计算的聚合。比如，给图书添加作者数量的注解：

```
# Build an annotated queryset
>>> from django.db.models import Count
>>> q = Book.objects.annotate(Count('authors'))
# Interrogate the first object in the queryset
>>> q[0]
<Book: The Definitive Guide to Django>
>>> q[0].authors__count
2
# Interrogate the second object in the queryset
>>> q[1]
<Book: Practical Django Projects>
>>> q[1].authors__count
1
```

和使用 aggregate()一样，注解的名称也根据聚合函式的名称和聚合字段的名称得到的。你可以在指定注解时，为默认名称提供一个别名：

```
>>> q = Book.objects.annotate(num_authors=Count('authors'))
>>> q[0].num_authors
2
>>> q[1].num_authors
1
```

与 aggregate() 不同的是， annotate() 不是一个终止子句。annotate()子句的返回结果是一个查询集 (QuerySet)；这个 QuerySet可以用任何QuerySet方法进行修改，包括 filter(), order_by(), 甚至是再次应用annotate()。

> 有任何疑问的话，请检查 SQL query！
> 
> 要想弄清楚你的查询到底发生了什么，可以考虑检查你QuerySet的 query 属性。
> 
> 例如，在annotate() 中混入多个聚合将会得出错误的结果，因为多个表上做了交叉连接，导致了多余的行聚合。

## 连接和聚合 ##

至此，我们已经了解了作用于单种模型实例的聚合操作， 但是有时，你也想对所查询对象的关联对象进行聚合。

在聚合函式中指定聚合字段时，Django 允许你使用同样的 双下划线 表示关联关系，然后 Django 在就会处理要读取的关联表，并得到关联对象的聚合。

例如，要得到每个书店的价格区别，可以使用如下注解：

```
>>> from django.db.models import Max, Min
>>> Store.objects.annotate(min_price=Min('books__price'), max_price=Max('books__price'))
```

这段代码告诉 Django 获取书店模型，并连接(通过多对多关系)图书模型，然后对每本书的价格进行聚合，得出最小值和最大值。

同样的规则也用于  aggregate() 子句。如果你想知道所有书店中最便宜的书和最贵的书价格分别是多少：

```
>>> Store.objects.aggregate(min_price=Min('books__price'), max_price=Max('books__price'))
```

关系链可以按你的要求一直延伸。 例如，想得到所有作者当中最小的年龄是多少，就可以这样写：

```
>>> Store.objects.aggregate(youngest_age=Min('books__authors__age'))
```

## 遵循反向关系 ##

和 跨关系查找的方法类似，作用在你所查询的模型的关联模型或者字段上的聚合和注解可以遍历"反转"关系。关联模型的小写名称和双下划线也用在这里。

例如，我们可以查询所有出版商，并注上它们一共出了多少本书（注意我们如何用 'book'指定Publisher -> Book 的外键反转关系）：

```
>>> from django.db.models import Count, Min, Sum, Avg
>>> Publisher.objects.annotate(Count('book'))
```

QuerySet结果中的每一个Publisher都会包含一个额外的属性叫做book__count。

我们也可以按照每个出版商，查询所有图书中最旧的那本：

```
>>> Publisher.objects.aggregate(oldest_pubdate=Min('book__pubdate'))
```

（返回的字典会包含一个键叫做 'oldest_pubdate'。如果没有指定这样的别名，它会更长一些，像 'book__pubdate__min'。）

这不仅仅可以应用挂在外键上面。还可以用到多对多关系上。例如，我们可以查询每个作者，注上它写的所有书（以及合著的书）一共有多少页（注意我们如何使用 'book'来指定Author -> Book的多对多的反转关系）：

```
>>> Author.objects.annotate(total_pages=Sum('book__pages'))
```

（每个返回的QuerySet中的Author 都有一个额外的属性叫做total_pages。如果没有指定这样的别名，它会更长一些，像 book__pages__sum。）

或者查询所有图书的平均评分，这些图书由我们存档过的作者所写：

```
>>> Author.objects.aggregate(average_rating=Avg('book__rating'))
```

（返回的字典会包含一个键叫做'average__rating'。如果没有指定这样的别名，它会更长一些，像'book__rating__avg'。）

聚合和其他查询集子句

## filter() 和 exclude() ##

聚合也可以在过滤器中使用。 作用于普通模型字段的任何 filter()(或 exclude()) 都会对聚合涉及的对象进行限制。

使用annotate() 子句时，过滤器有限制注解对象的作用。例如，你想得到以 "Django" 为书名开头的图书作者的总数：

```
>>> from django.db.models import Count, Avg
>>> Book.objects.filter(name__startswith="Django").annotate(num_authors=Count('authors'))
```

使用aggregate()子句时，过滤器有限制聚合对象的作用。例如，你可以算出所有以 "Django" 为书名开头的图书平均价格：

```
>>> Book.objects.filter(name__startswith="Django").aggregate(Avg('price'))
```

## 对注解过滤 ##

注解值也可以被过滤。 像使用其他模型字段一样，注解也可以在filter()和exclude() 子句中使用别名。

例如，要得到不止一个作者的图书，可以用：

```
>>> Book.objects.annotate(num_authors=Count('authors')).filter(num_authors__gt=1)
```

这个查询首先生成一个注解结果，然后再生成一个作用于注解上的过滤器。

## annotate() 的顺序 ##

编写一个包含 annotate() 和 filter()  子句的复杂查询时，要特别注意作用于 QuerySet的子句的顺序。

当一个annotate() 子句作用于某个查询时，要根据查询的状态才能得出注解值，而状态由 annotate() 位置所决定。以这就导致filter() 和 annotate() 不能交换顺序，下面两个查询就是不同的：

```
>>> Publisher.objects.annotate(num_books=Count('book')).filter(book__rating__gt=3.0)
```

另一个查询：

```
>>> Publisher.objects.filter(book__rating__gt=3.0).annotate(num_books=Count('book'))
```

两个查询都返回了至少出版了一本好书(评分大于 3 分)的出版商。 但是第一个查询的注解包含其该出版商发行的所有图书的总数；而第二个查询的注解只包含出版过好书的出版商的所发行的图书总数。 在第一个查询中，注解在过滤器之前，所以过滤器对注解没有影响。 在第二个查询中，过滤器在注解之前，所以，在计算注解值时，过滤器就限制了参与运算的对象的范围。

## order_by() ##

注解可以用来做为排序项。 在你定义 order_by() 子句时，你提供的聚合可以引用定义的任何别名做为查询中 annotate()子句的一部分。

例如，根据一本图书作者数量的多少对查询集 QuerySet进行排序：

```
>>> Book.objects.annotate(num_authors=Count('authors')).order_by('num_authors')
```

## values() ##

通常，注解会添加到每个对象上 —— 一个被注解的QuerySet会为初始QuerySet的每个对象返回一个结果集。但是，如果使用了values()子句，它就会限制结果中列的范围，对注解赋值的方法就会完全不同。不是在原始的  QuerySet返回结果中对每个对象中添加注解，而是根据定义在values()  子句中的字段组合对先结果进行唯一的分组，再根据每个分组算出注解值， 这个注解值是根据分组中所有的成员计算而得的：

例如，考虑一个关于作者的查询，查询出每个作者所写的书的平均评分：

```
>>> Author.objects.annotate(average_rating=Avg('book__rating'))
```

这段代码返回的是数据库中所有的作者以及他们所著图书的平均评分。

但是如果你使用了values()子句，结果是完全不同的：

```
>>> Author.objects.values('name').annotate(average_rating=Avg('book__rating'))
```

在这个例子中，作者会按名称分组，所以你只能得到某个唯一的作者分组的注解值。 这意味着如果你有两个作者同名，那么他们原本各自的查询结果将被合并到同一个结果中；两个作者的所有评分都将被计算为一个平均分。

## annotate() 的顺序 ##

和使用 filter() 子句一样，作用于某个查询的annotate() 和  values() 子句的使用顺序是非常重要的。如果values() 子句在  annotate() 之前，就会根据 values()  子句产生的分组来计算注解。

但是，如果  annotate()  子句在 values()子句之前，就会根据整个查询集生成注解。在这种情况下，values() 子句只能限制输出的字段范围。

举个例子，如果我们互换了上个例子中 values()和 annotate() 子句的顺序：

```
>>> Author.objects.annotate(average_rating=Avg('book__rating')).values('name', 'average_rating')
```

这段代码将给每个作者添加一个唯一的字段，但只有作者名称和average_rating 注解会返回在输出结果中。

你也应该注意到 average_rating 显式地包含在返回的列表当中。之所以这么做的原因正是因为values()  和 annotate() 子句。

如果 values()  子句在 annotate() 子句之前，注解会被自动添加到结果集中；但是，如果 values() 子句作用于annotate() 子句之后，你需要显式地包含聚合列。

## 与默认排序或order_by()交互 ##

在查询集中的order_by() 部分(或是在模型中默认定义的排序项) 会在选择输出数据时被用到，即使这些字段没有在values() 调用中被指定。这些额外的字段可以将相似的数据行分在一起，也可以让相同的数据行相分离。在做计数时，就会表现地格外明显：

通过例子中的方法，假设有一个这样的模型：

```
from django.db import models

class Item(models.Model):
    name = models.CharField(max_length=10)
    data = models.IntegerField()

    class Meta:
        ordering = ["name"]
```

关键的部分就是在模型默认排序项中设置的name字段。如果你想知道每个非重复的data值出现的次数，可以这样写：

```
# Warning: not quite correct!
Item.objects.values("data").annotate(Count("id"))
```

...这部分代码想通过使用它们公共的 data 值来分组 Item对象，然后在每个分组中得到  id 值的总数。但是上面那样做是行不通的。这是因为默认排序项中的 name也是一个分组项，所以这个查询会根据非重复的 (data, name) 进行分组，而这并不是你本来想要的结果。所以，你应该这样改写：

```
Item.objects.values("data").annotate(Count("id")).order_by()
```

...这样就清空了查询中的所有排序项。 你也可以在其中使用 data ，这样并不会有副作用，这是因为查询分组中只有这么一个角色了。

这个行为与查询集文档中提到的 distinct() 一样，而且生成规则也一样：一般情况下，你不想在结果中由额外的字段扮演这个角色，那就清空排序项，或是至少保证它仅能访问 values()中的字段。

> 注意
> 
> 你可能想知道为什么 Django 不删除与你无关的列。主要原因就是要保证使用 distinct()和其他方法的一致性。Django  永远不会 删除你所指定的排序限制(我们不能改动那些方法的行为，因为这会违背 API stability 原则)。

## 聚合注解 ##

你也可以在注解的结果上生成聚合。 当你定义一个  aggregate() 子句时，你提供的聚合会引用定义的任何别名做为查询中 annotate() 子句的一部分。

例如，如果你想计算每本书平均有几个作者，你先用作者总数注解图书集，然后再聚合作者总数，引入注解字段：

```
>>> from django.db.models import Count, Avg
>>> Book.objects.annotate(num_authors=Count('authors')).aggregate(Avg('num_authors'))
{'num_authors__avg': 1.66}
```