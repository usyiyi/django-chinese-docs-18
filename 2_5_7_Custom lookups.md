# 自定义查找 #

```
New in Django 1.7.
```

Django为过滤提供了大量的内建的查找（例如，`exact`和`icontains`）。这篇文档阐述了如何编写自定义查找，以及如何修改现存查找的功能。关于查找的API参考，详见查找API参考。

## 一个简单的查找示例 ##

让我们从一个简单的自定义查找开始。我们会编写一个自定义查找`ne`，提供和`exact`相反的功能。`Author.objects.filter(name__ne='Jack')`会转换成下面的SQL：

```
"author"."name" <> 'Jack'
```

这条SQL是后端独立的，所以我们并不需要担心不同的数据库。

实现它需要两个步骤。首先我们需要实现这个查找，然后我们需要告诉Django它的信息。实现是十分简单直接的：

```
from django.db.models import Lookup

class NotEqual(Lookup):
    lookup_name = 'ne'

    def as_sql(self, compiler, connection):
        lhs, lhs_params = self.process_lhs(compiler, connection)
        rhs, rhs_params = self.process_rhs(compiler, connection)
        params = lhs_params + rhs_params
        return '%s <> %s' % (lhs, rhs), params
```

我们只需要在我们想让查找应用的字段上调用`register_lookup`，来注册`NotEqual`查找。这种情况下，查找在所有`Field`的子类都起作用，所以我们直接使用`Field`注册它。

```
from django.db.models.fields import Field
Field.register_lookup(NotEqual)
```

也可以使用装饰器模式来注册查找：

```
from django.db.models.fields import Field

@Field.register_lookup
class NotEqualLookup(Lookup):
    # ...
```

```
Changed in Django 1.8:

新增了使用装饰器模式的能力。
```

我们现在可以为任何`foo`字段使用 `foo__ne`。你需要确保在你尝试创建使用它的任何查询集之前完成注册。你应该把实现放在`models.py`文件中，或者在`AppConfig`的`ready()`方法中注册查找。

现在让我们深入观察这个实现，首先需要的属性是`lookup_name`。这需要让ORM理解如何去解释`name__ne`，以及如何使用`NotEqual`来生成SQL。按照惯例，这些名字一般是只包含字母的小写字符串，但是唯一硬性的要求是不能够包含字符串`__`。

然后我们需要定义`as_sql`方法。这个方法需要传入一个`SQLCompiler`对象，叫做 `compiler`，以及活动的数据库连接。`SQLCompiler`对象并没有记录，但是我们需要知道的唯一一件事就是他们拥有`compile()`方法，这个方法返回一个元组，含有SQL字符串和要向字符串插入的参数。在多数情况下，你并不需要世界使用它，并且可以把它传递给`process_lhs()` 和 `process_rhs()`。

`Lookup`作用于两个值，lhs和rhs，分别是左边和右边。左边的值一般是个字段的引用，但是它可以是任何实现了查询表达式API的对象。右边的值由用户提供。在例子`Author.objects.filter(name__ne='Jack')`中，左边的值是`Author`模型的`name` 字段的引用，右边的值是`'Jack'`。

我们可以调用 `process_lhs` 和`process_rhs` 来将它们转换为我们需要的SQL值，使用之前我们描述的`compiler` 对象。

最后我们用`<>`将这些部分组合成SQL表达式，然后将所有参数用在查询中。然后我们返回一个元组，包含生成的SQL字符串以及参数。

## 一个简单的转换器示例 ##

上面的自定义转换器是极好的，但是一些情况下你可能想要把查找放在一起。例如，假设我们构建一个应用，想要利用`abs()` 操作符。我们有用一个`Experiment`模型，它记录了起始值，终止值，以及变化量（起始值 - 终止值）。我们想要寻找所有变化量等于一个特定值的实验（`Experiment.objects.filter(change__abs=27)`），或者没有达到指定值的实验（`Experiment.objects.filter(change__abs__lt=27)`）。

> 注意
>
> 这个例子一定程度上很不自然，但是很好地展示了数据库后端独立的功能范围，并且没有重复实现Django中已有的功能。

我们从编写`AbsoluteValue`转换器来开始。这会用到SQL函数`ABS()`，来在比较之前转换值。

```
from django.db.models import Transform

class AbsoluteValue(Transform):
    lookup_name = 'abs'

    def as_sql(self, compiler, connection):
        lhs, params = compiler.compile(self.lhs)
        return "ABS(%s)" % lhs, params
```

接下来，为`IntegerField`注册它：

```
from django.db.models import IntegerField
IntegerField.register_lookup(AbsoluteValue)
```

我们现在可以执行之前的查询。`Experiment.objects.filter(change__abs=27)`会生成下面的SQL：

```
SELECT ... WHERE ABS("experiments"."change") = 27
```

通过使用`Transform`来替代`Lookup`，这说明了我们能够把以后更多的查找放到一起。所以`Experiment.objects.filter(change__abs__lt=27)`会生成以下的SQL：

```
SELECT ... WHERE ABS("experiments"."change") < 27
```

注意在没有指定其他查找的情况中，Django会将 `change__abs=27` 解释为`change__abs__exact=27`。

当寻找在 `Transform`之后，哪个查找可以使用的时候，Django使用`output_field`属性。因为它并没有修改，我们在这里并不指定，但是假设我们在一些字段上应用AbsoluteValue，这些字段代表了一个更复杂的类型（比如说与原点（origin）相关的一个点，或者一个复数（complex number））。之后我们可能想指定，转换要为进一步的查找返回`FloatField`类型。这可以通过向转换添加`output_field` 属性来实现：

```
from django.db.models import FloatField, Transform

class AbsoluteValue(Transform):
    lookup_name = 'abs'

    def as_sql(self, compiler, connection):
        lhs, params = compiler.compile(self.lhs)
        return "ABS(%s)" % lhs, params

    @property
    def output_field(self):
        return FloatField()
```

这确保了更进一步的查找，像`abs__lte`的行为和对`FloatField`表现的一样。

## 编写高效的 `abs__lt` 查找 ##

当我们使用上面编写的`abs`查找的时候，在一些情况下，生成的SQL并不会高效使用索引。尤其是我们使用`change__abs__lt=27`的时候，这等价于`change__gt=-27 AND change__lt=27`。（对于`lte` 的情况，我们可以使用 SQL子句`BETWEEN`）。

所以我们想让`Experiment.objects.filter(change__abs__lt=27)`生成以下SQL:

```
SELECT .. WHERE "experiments"."change" < 27 AND "experiments"."change" > -27
```

它的实现为：

```
from django.db.models import Lookup

class AbsoluteValueLessThan(Lookup):
    lookup_name = 'lt'

    def as_sql(self, compiler, connection):
        lhs, lhs_params = compiler.compile(self.lhs.lhs)
        rhs, rhs_params = self.process_rhs(compiler, connection)
        params = lhs_params + rhs_params + lhs_params + rhs_params
        return '%s < %s AND %s > -%s' % (lhs, rhs, lhs, rhs), params

AbsoluteValue.register_lookup(AbsoluteValueLessThan)
```

有一些值得注意的事情。首先，`AbsoluteValueLessThan`并不调用`process_lhs()`。而是它跳过了由`AbsoluteValue`完成的`lhs`，并且使用原始的`lhs`。这就是说，我们想要得到`27` 而不是`ABS(27)`。直接引用`self.lhs.lhs`是安全的，因为 `AbsoluteValueLessThan`只能够通过` AbsoluteValue `查找来访问，这就是说 `lhs`始终是`AbsoluteValue`的实例。

也要注意，就像两边都要在查询中使用多次一样，参数也需要多次包含`lhs_params` 和`rhs_params`。

最终的实现直接在数据库中执行了反转 (27变为 -27) 。这样做的原因是如果`self.rhs`不是一个普通的整数值（比如是一个`F()`引用），我们在Python中不能执行这一转换。

> 注意
>
> 实际上，大多数带有__abs的查找都实现为这种范围查询，并且在大多数数据库后端中它更可能执行成这样，就像你可以利用索引一样。然而在PostgreSQL中，你可能想要向abs(change) 中添加索引，这会使查询更高效。

## 一个双向转换器的示例 ##

我们之前讨论的，`AbsoluteValue`的例子是一个只应用在查找左侧的转换。可能有一些情况，你想要把转换同时应用在左侧和右侧。比如，你想过滤一个基于左右侧相等比较操作的查询集，在执行一些SQL函数之后它们是大小写不敏感的。

让我们测试一下这一大小写不敏感的转换的简单示例。这个转换在实践中并不是十分有用，因为Django已经自带了一些自建的大小写不敏感的查找，但是它是一个很好的，数据库无关的双向转换示例。

我们定义使用SQL 函数`UPPER()`的`UpperCase `转换器，来在比较前转换这些值。我们定义了`bilateral = True`来表明转换同时作用在`lhs` 和`rhs`上面：

```
from django.db.models import Transform

class UpperCase(Transform):
    lookup_name = 'upper'
    bilateral = True

    def as_sql(self, compiler, connection):
        lhs, params = compiler.compile(self.lhs)
        return "UPPER(%s)" % lhs, params
```

接下来，让我们注册它：

```
from django.db.models import CharField, TextField
CharField.register_lookup(UpperCase)
TextField.register_lookup(UpperCase)
```

现在，查询集`Author.objects.filter(name__upper="doe")`会生成像这样的大小写不敏感查询：

```
SELECT ... WHERE UPPER("author"."name") = UPPER('doe')
```

## 为现存查找编写自动的实现 ##

有时不同的数据库供应商对于相同的操作需要不同的SQL。对于这个例子，我们会为MySQL重新编写一个自定义的，`NotEqual`操作的实现。我们会使用 `!=` 而不是 `<>`操作符。（注意实际上几乎所有数据库都支持这两个，包括所有Django支持的官方数据库）。

我们可以通过创建带有`as_mysql`方法的`NotEqual`的子类来修改特定后端上的行为。

```
class MySQLNotEqual(NotEqual):
    def as_mysql(self, compiler, connection):
        lhs, lhs_params = self.process_lhs(compiler, connection)
        rhs, rhs_params = self.process_rhs(compiler, connection)
        params = lhs_params + rhs_params
        return '%s != %s' % (lhs, rhs), params

Field.register_lookup(MySQLNotEqual)
```

我们可以在`Field`中注册它。它取代了原始的`NotEqual`类，由于它具有相同的`lookup_name`。

当编译一个查询的时候，Django首先寻找`as_%s % connection.vendor`方法，然后回退到 `as_sql`。内建后端的供应商名称是 sqlite，postgresql， oracle 和mysql。

## Django如何决定使用查找还是转换 ##

有些情况下，你可能想要动态修改基于传递进来的名称， `Transform` 或者 `Lookup`哪个会返回，而不是固定它。比如，你拥有可以储存搭配（ coordinate）或者任意一个维度（dimension）的字段，并且想让类似于`.filter(coords__x7=4)`的语法返回第七个搭配值为4的对象。为了这样做，你可以用一些东西覆写`get_lookup`，比如：

```
class CoordinatesField(Field):
    def get_lookup(self, lookup_name):
        if lookup_name.startswith('x'):
            try:
                dimension = int(lookup_name[1:])
            except ValueError:
                pass
            finally:
                return get_coordinate_lookup(dimension)
        return super(CoordinatesField, self).get_lookup(lookup_name)
```

之后你应该合理定义`get_coordinate_lookup`。来返回一个 `Lookup`的子类，它处理`dimension`的相关值。

有一个名称相似的方法叫做`get_transform()`。`get_lookup()`应该始终返回 `Lookup` 的子类，而`get_transform()` 返回`Transform` 的子类。记住`Transform` 对象可以进一步过滤，而 `Lookup` 对象不可以，这非常重要。

过滤的时候，如果还剩下只有一个查找名称要处理，它会寻找`Lookup`。如果有多个名称，它会寻找`Transform`。在只有一个名称并且 Lookup找不到的情况下，会寻找`Transform`，之后寻找在`Transform`上面的`exact`查找。所有调用的语句都以一个`Lookup`结尾。解释一下：

+ `.filter(myfield__mylookup)`会调用 `myfield.get_lookup('mylookup')`。
+ `.filter(myfield__mytransform__mylookup)` 会调用 `myfield.get_transform('mytransform')`，然后调用`mytransform.get_lookup('mylookup')`。
+ `.filter(myfield__mytransform)` 会首先调用 `myfield.get_lookup('mytransform')`，这样会失败，所以它会回退来调用 `myfield.get_transform('mytransform')` ，之后是 `mytransform.get_lookup('exact')`。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Custom lookups](https://docs.djangoproject.com/en/1.8/howto/custom-lookups/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
