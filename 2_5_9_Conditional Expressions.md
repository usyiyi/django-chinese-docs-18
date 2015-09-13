# 条件表达式 #

```
New in Django 1.8.
```

条件表达式允许你在过滤器、注解、聚合和更新操作中使用 `if ... elif ... else`的逻辑。条件[表达式](http://python.usyiyi.cn/django/ref/models/expressions.html)为表中的每一行计算一系列的条件，并且返回匹配到的结果表达式。条件表达式也可以像其它 表达式一样混合和嵌套。

## 条件表达式类 ##

我们会在后面的例子中使用下面的模型：

```
from django.db import models

class Client(models.Model):
    REGULAR = 'R'
    GOLD = 'G'
    PLATINUM = 'P'
    ACCOUNT_TYPE_CHOICES = (
        (REGULAR, 'Regular'),
        (GOLD, 'Gold'),
        (PLATINUM, 'Platinum'),
    )
    name = models.CharField(max_length=50)
    registered_on = models.DateField()
    account_type = models.CharField(
        max_length=1,
        choices=ACCOUNT_TYPE_CHOICES,
        default=REGULAR,
    )
```

### When ###

`class When(condition=None, then=None, **lookups)[source]`

`When()`对象用于封装条件和它的结果，为了在条件表达式中使用。使用`When()`对象和使用`filter()` 方法类似。条件可以使用[字段查找](http://python.usyiyi.cn/django/ref/models/querysets.html#field-lookups) 或者 `Q` 来指定。结果通过使用`then`关键字来提供。

一些例子：

```
>>> from django.db.models import When, F, Q
>>> # String arguments refer to fields; the following two examples are equivalent:
>>> When(account_type=Client.GOLD, then='name')
>>> When(account_type=Client.GOLD, then=F('name'))
>>> # You can use field lookups in the condition
>>> from datetime import date
>>> When(registered_on__gt=date(2014, 1, 1),
...      registered_on__lt=date(2015, 1, 1),
...      then='account_type')
>>> # Complex conditions can be created using Q objects
>>> When(Q(name__startswith="John") | Q(name__startswith="Paul"),
...      then='name')
```

要注意这些值中的每一个都可以是表达式。

> 注意
>
> 由于then 关键字参数为 When()的结果而保留，如果Model有名称为 then的字段，会有潜在的冲突。这可以用以下两种办法解决：

```
>>> from django.db.models import Value
>>> When(then__exact=0, then=1)
>>> When(Q(then=0), then=1)
```

### Case ###

`class Case(*cases, **extra)[source]`

`Case()`表达式就像是Python中的`if ... elif ... else`语句。每个提供的`When()`中的`condition` 按照顺序计算，直到得到一个真值。返回匹配`When() `对象的`result`表达式。

一个简单的例子：

```
>>>
>>> from datetime import date, timedelta
>>> from django.db.models import CharField, Case, Value, When
>>> Client.objects.create(
...     name='Jane Doe',
...     account_type=Client.REGULAR,
...     registered_on=date.today() - timedelta(days=36))
>>> Client.objects.create(
...     name='James Smith',
...     account_type=Client.GOLD,
...     registered_on=date.today() - timedelta(days=5))
>>> Client.objects.create(
...     name='Jack Black',
...     account_type=Client.PLATINUM,
...     registered_on=date.today() - timedelta(days=10 * 365))
>>> # Get the discount for each Client based on the account type
>>> Client.objects.annotate(
...     discount=Case(
...         When(account_type=Client.GOLD, then=Value('5%')),
...         When(account_type=Client.PLATINUM, then=Value('10%')),
...         default=Value('0%'),
...         output_field=CharField(),
...     ),
... ).values_list('name', 'discount')
[('Jane Doe', '0%'), ('James Smith', '5%'), ('Jack Black', '10%')]
```

`Case()` 接受任意数量的`When()`对象作为独立的参数。其它选项使用关键字参数提供。如果没有条件为`TRUE`，表达式会返回提供的`default`关键字参数。如果没有提供`default`参数，会使用`Value(None)`。

如果我们想要修改之前的查询，来获取基于`Client`跟着我们多长时间的折扣，我们应该这样使用查找：

```
>>> a_month_ago = date.today() - timedelta(days=30)
>>> a_year_ago = date.today() - timedelta(days=365)
>>> # Get the discount for each Client based on the registration date
>>> Client.objects.annotate(
...     discount=Case(
...         When(registered_on__lte=a_year_ago, then=Value('10%')),
...         When(registered_on__lte=a_month_ago, then=Value('5%')),
...         default=Value('0%'),
...         output_field=CharField(),
...     )
... ).values_list('name', 'discount')
[('Jane Doe', '5%'), ('James Smith', '0%'), ('Jack Black', '10%')]
```

> 注意
>
> 记住条件按照顺序来计算，所以上面的例子中，即使第二个条件匹配到了 Jane Doe 和 Jack Black，我们也得到了正确的结果。这就像Python中的if ... elif ... else语句一样。

## 高级查询 ##

条件表达式可以用于注解、聚合、查找和更新。它们也可以和其它表达式混合和嵌套。这可以让你构造更强大的条件查询。

### 条件更新 ###

假设我们想要为客户端修改`account_type`来匹配它们的注册日期。我们可以使用条件表达式和`update()`放啊来实现：

```
>>> a_month_ago = date.today() - timedelta(days=30)
>>> a_year_ago = date.today() - timedelta(days=365)
>>> # Update the account_type for each Client from the registration date
>>> Client.objects.update(
...     account_type=Case(
...         When(registered_on__lte=a_year_ago,
...              then=Value(Client.PLATINUM)),
...         When(registered_on__lte=a_month_ago,
...              then=Value(Client.GOLD)),
...         default=Value(Client.REGULAR)
...     ),
... )
>>> Client.objects.values_list('name', 'account_type')
[('Jane Doe', 'G'), ('James Smith', 'R'), ('Jack Black', 'P')]
```

### 条件聚合 ###

如果我们想要弄清楚每个`account_type`有多少客户端，要怎么做呢？我们可以在[聚合函数](http://python.usyiyi.cn/django/ref/models/querysets.html#aggregation-functions)中嵌套条件表达式来实现：

```
>>> # Create some more Clients first so we can have something to count
>>> Client.objects.create(
...     name='Jean Grey',
...     account_type=Client.REGULAR,
...     registered_on=date.today())
>>> Client.objects.create(
...     name='James Bond',
...     account_type=Client.PLATINUM,
...     registered_on=date.today())
>>> Client.objects.create(
...     name='Jane Porter',
...     account_type=Client.PLATINUM,
...     registered_on=date.today())
>>> # Get counts for each value of account_type
>>> from django.db.models import IntegerField, Sum
>>> Client.objects.aggregate(
...     regular=Sum(
...         Case(When(account_type=Client.REGULAR, then=1),
...              output_field=IntegerField())
...     ),
...     gold=Sum(
...         Case(When(account_type=Client.GOLD, then=1),
...              output_field=IntegerField())
...     ),
...     platinum=Sum(
...         Case(When(account_type=Client.PLATINUM, then=1),
...              output_field=IntegerField())
...     )
... )
{'regular': 2, 'gold': 1, 'platinum': 3}
```

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Conditional Expressions](https://docs.djangoproject.com/en/1.8/ref/models/conditional-expressions/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
