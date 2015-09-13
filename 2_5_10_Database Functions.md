# 数据库函数 #

```
New in Django 1.8.
```

下面记述的类为用户提供了一些方法，来在Django中使用底层数据库提供的函数用于注解、聚合或者过滤器等操作。函数也是表达式，所以可以像聚合函数一样混合使用它们。

我们会在每个函数的实例中使用下面的模型：

```
class Author(models.Model):
    name = models.CharField(max_length=50)
    age = models.PositiveIntegerField(null=True, blank=True)
    alias = models.CharField(max_length=50, null=True, blank=True)
    goes_by = models.CharField(max_length=50, null=True, blank=True)
```

我们并不推荐在CharField上允许null=True，以后那位这会允许字段有两个“空值”，但是对于下面的Coalesce示例来说它很重要。

## Coalesce ##

`class Coalesce(*expressions, **extra)[source]`

接受一个含有至少两个字段名称或表达式的列表，返回第一个非空的值（注意空字符串不被认为是一个空值）。每个参与都必须是相似的类型，所以掺杂了文本和数字的列表会导致数据库错误。

使用范例：

```
>>> # Get a screen name from least to most public
>>> from django.db.models import Sum, Value as V
>>> from django.db.models.functions import Coalesce
>>> Author.objects.create(name='Margaret Smith', goes_by='Maggie')
>>> author = Author.objects.annotate(
...    screen_name=Coalesce('alias', 'goes_by', 'name')).get()
>>> print(author.screen_name)
Maggie

>>> # Prevent an aggregate Sum() from returning None
>>> aggregated = Author.objects.aggregate(
...    combined_age=Coalesce(Sum('age'), V(0)),
...    combined_age_default=Sum('age'))
>>> print(aggregated['combined_age'])
0
>>> print(aggregated['combined_age_default'])
None
```

## Concat ##

`class Concat(*expressions, **extra)[source]`

接受一个含有至少两个文本字段的或表达式的列表，返回连接后的文本。每个参数都必须是文本或者字符类型。如果你想把一个`TextField()`和一个`CharField()`连接， 一定要告诉Django`output_field`应该为`TextField()`类型。在下面连接`Value`的例子中，这也是必需的。

这个函数不会返回`null`。在后端中，如果一个`null`参数导致了整个表达式都是`null`，Django会确保把每个`null`的部分转换成一个空字符串。

使用范例：

```
>>> # Get the display name as "name (goes_by)"
>>> from django.db.models import CharField, Value as V
>>> from django.db.models.functions import Concat
>>> Author.objects.create(name='Margaret Smith', goes_by='Maggie')
>>> author = Author.objects.annotate(
...    screen_name=Concat('name', V(' ('), 'goes_by', V(')'),
...    output_field=CharField())).get()
>>> print(author.screen_name)
Margaret Smith (Maggie)
```

## Length ##

`class Length(expression, **extra)[source]`

接受一个文本字段或表达式，返回值的字符个数。如果表达式是`null`，长度也会是`null`。

使用范例：

```
>>> # Get the length of the name and goes_by fields
>>> from django.db.models.functions import Length
>>> Author.objects.create(name='Margaret Smith')
>>> author = Author.objects.annotate(
...    name_length=Length('name'),
...    goes_by_length=Length('goes_by')).get()
>>> print(author.name_length, author.goes_by_length)
(14, None)
```

## Lower ##

`class Lower(expression, **extra)[source]`

接受一个文本字符串或表达式，返回它的小写表示形式。

使用范例：

```
>>> from django.db.models.functions import Lower
>>> Author.objects.create(name='Margaret Smith')
>>> author = Author.objects.annotate(name_lower=Lower('name')).get()
>>> print(author.name_lower)
margaret smith
```

## Substr ##

`class Substr(expression, pos, length=None, **extra)[source]`

返回这个字段或者表达式的，以`pos`位置开始，长度为`length`的子字符串。位置从下标为1开始，所以必须大于0。如果`length`是`None`，会返回剩余的字符串。

使用范例：

```
>>> # Set the alias to the first 5 characters of the name as lowercase
>>> from django.db.models.functions import Substr, Lower
>>> Author.objects.create(name='Margaret Smith')
>>> Author.objects.update(alias=Lower(Substr('name', 1, 5)))
1
>>> print(Author.objects.get(name='Margaret Smith').alias)
marga
```

## Upper ##

`class Upper(expression, **extra)[source]`

接受一个文本字符串或表达式，返回它的大写表示形式。

使用范例：

```
>>> from django.db.models.functions import Upper
>>> Author.objects.create(name='Margaret Smith')
>>> author = Author.objects.annotate(name_upper=Upper('name')).get()
>>> print(author.name_upper)
MARGARET SMITH
```

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Database Functions](https://docs.djangoproject.com/en/1.8/ref/models/database-functions/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
