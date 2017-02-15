

# 查询表达式

查询表达式可以作为过滤，分组，注解或者是聚合的一个值或者是计算。这里（文档中）有很多内置表达式可以帮助你完成自己的查询。表达式可以组合，甚至是嵌套，来完成更加复杂的计算

## 支持的算术

Django支持在查询表达式使用加减乘除，求模，幂运算，Python常量，变量甚至是其它表达式。

New in Django 1.7:

增加了对指数运算符`**`的支持。

## 一些例子

Changed in Django 1.8:

一些例子依赖于Django1.8中的新功能

```
from django.db.models import F, Count
from django.db.models.functions import Length

# Find companies that have more employees than chairs.
Company.objects.filter(num_employees__gt=F('num_chairs'))

# Find companies that have at least twice as many employees
# as chairs. Both the querysets below are equivalent.
Company.objects.filter(num_employees__gt=F('num_chairs') * 2)
Company.objects.filter(
    num_employees__gt=F('num_chairs') + F('num_chairs'))

# How many chairs are needed for each company to seat all employees?
>>> company = Company.objects.filter(
...    num_employees__gt=F('num_chairs')).annotate(
...    chairs_needed=F('num_employees') - F('num_chairs')).first()
>>> company.num_employees
120
>>> company.num_chairs
50
>>> company.chairs_needed
70

# Annotate models with an aggregated value. Both forms
# below are equivalent.
Company.objects.annotate(num_products=Count('products'))
Company.objects.annotate(num_products=Count(F('products')))

# Aggregates can contain complex computations also
Company.objects.annotate(num_offerings=Count(F('products') + F('services')))

# Expressions can also be used in order_by()
Company.objects.order_by(Length('name').asc())
Company.objects.order_by(Length('name').desc())

```

## 内置表达式

说明

这些表达式定义在`django.db.models.expressions` 和 `django.db.models.aggregates`中, 但为了方便，通常可以直接从[`django.db.models`](../../topics/db/models.html#module-django.db.models "django.db.models")导入.

### `F()` 表达式

_class_ `F`

一个 `F()`对象代表了一个model的字段值或注释列。使用它就可以直接参考model的field和执行数据库操作而不用再把它们（model field）查询出来放到python内存中。

作为代替，Django使用 `F()`对象生成一个SQL表达式，来描述数据库层级所需要的操作

这些通过一个例子可以很容易的理解。往常，我们会这样做：

```
# Tintin filed a news story!
reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed += 1
reporter.save()

```

这里呢，我们把 `reporter.stories_filed` 的值从数据库取出放到内存中并用我们熟悉的python运算符操作它，最后再把它保存到数据库。然而，我们还可以这样做：

```
from django.db.models import F

reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed = F('stories_filed') + 1
reporter.save()

```

虽然`reporter.stories_filed = F('stories_filed') + 1`看起来像一个正常的Python分配值赋给一个实例属性，事实上这是一个描述数据库操作的SQL概念

当Django遇到 `F()`实例，它覆盖了标准的Python运算符创建一个封装的SQL表达式。在这个例子中，`reporter.stories_filed`就代表了一个指示数据库对该字段进行增量的命令。

无论`reporter.stories_filed`的值是或曾是什么，Python一无所知--这完全是由数据库去处理的。所有的Python，通过Django的`F()` 类，只是去创建SQL语法参考字段和描述操作。

注

为了获得用这种方法保存的新值，此对象应重新加载：

```
reporter = Reporters.objects.get(pk=reporter.pk)

```

.和上面单独实例的操作一样， `F()`配合 `update()可以应用于对象实例的 `QuerySets``这减少了我们上面使用的两个查询 - `get()`和[`save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save") - 只有一个：

```
reporter = Reporters.objects.filter(name='Tintin')
reporter.update(stories_filed=F('stories_filed') + 1)

```

我们可以使用[`update()`](querysets.html#django.db.models.query.QuerySet.update "django.db.models.query.QuerySet.update")方法批量地增加多个对象的字段值。这比先从数据库查询后，通过循环一个个增加，并一个个保存要快的很多。

```
Reporter.objects.all().update(stories_filed=F('stories_filed') + 1)

```

`F()`表达式的效率上的优点主要体现在

*   直接通过数据库操作而不是Python
*   减少数据库查询次数

#### 使用 `F()` 避免竞态条件

使用 `F()` 的另一个好处是通过数据库而不是Python来更新字段值以避免_竞态条件_.

如果两个Python线程执行上面第一个例子中的代码，一个线程可能在另一个线程刚从数据库中获取完字段值后获取、增加、保存该字段值。第二个线程保存的值将会基于原始字段值；第一个线程的工作将会丢失。

如果让数据库对更新字段负责，这个过程将变得更稳健：它将只会在 [`save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save") 或 `update()`执行时根据数据库中该字段值来更新字段，而不是基于实例之前获取的字段值。

#### 在过滤器中使用 `F()`

`F()`在`查询集` 过滤器中也十分有用，它使得使用条件通过字段值而不是Python值过滤一组对象变得可能。

这在 [_在查询中使用F()表达式_](../../topics/db/queries.html#using-f-expressions-in-filters) 中被记录。

#### 使用`F()`和注释

`F()`可用于通过将不同字段与算术相结合来在模型上创建动态字段：

```
company = Company.objects.annotate(
    chairs_needed=F('num_employees') - F('num_chairs'))

```

如果你组合的字段是不同类型，你需要告诉Django将返回什么类型的字段。由于`F()`不直接支持`output_field`，您需要使用[`ExpressionWrapper`](#django.db.models.ExpressionWrapper "django.db.models.ExpressionWrapper")

```
from django.db.models import DateTimeField, ExpressionWrapper, F

Ticket.objects.annotate(
    expires=ExpressionWrapper(
        F('active_at') + F('duration'), output_field=DateTimeField()))

```

### `Func()`表达式

New in Django 1.8.

`Func()` 表达式是所有表达式的基础类型，包括数据库函数如 `COALESCE` 和 `LOWER`, 或者 `SUM`聚合.用下面方式可以直接使用:

```
from django.db.models import Func, F

queryset.annotate(field_lower=Func(F('field'), function='LOWER'))

```

或者它们可以用于构建数据库函数库：

```
class Lower(Func):
    function = 'LOWER'

queryset.annotate(field_lower=Lower(F('field')))

```

但是这两种情况都将导致查询集，其中每个模型用从以下SQL大致生成的额外属性`field_lower`注释：

```
SELECT
    ...
    LOWER("app_label"."field") as "field_lower"

```

有关内置数据库函数的列表，请参见[_数据库函数_](database-functions.html)。

`Func` API如下：

_class_ `Func`(_*expressions_, _**extra_)

`function`

描述将生成的函数的类属性。具体来说，`函数`将被插入为[`模板`](#django.db.models.Func.template "django.db.models.Func.template")中的`函数`占位符。默认为`无`。

`template`

类属性，作为格式字符串，描述为此函数生成的SQL。默认为`'％（function）s（％（expressions）s）'`。

`arg_joiner`

类属性，表示用于连接`表达式`列表的字符。默认为`'， '`。

`*expressions`参数是函数将要应用与表达式的位置参数列表。表达式将转换为字符串，与`arg_joiner`连接在一起，然后作为`expressions`占位符插入`template`。

位置参数可以是表达式或Python值。字符串假定为列引用，并且将包装在`F()`表达式中，而其他值将包裹在`Value()`表达式中。

`**extra` kwargs是可以插入到`template`属性中的`key=value`对。请注意，关键字`function`和`template`可用于分别替换`function`和`template`属性，而无需定义自己的类。`output_field`可用于定义预期的返回类型。

### `Aggregate()`表达式

聚合表达式是[_Func() expression_](#func-expressions)的一种特殊情况，它通知查询：`GROUP BY`子句是必须的。所有[_aggregate functions_](querysets.html#aggregation-functions)，如`Sum()`和`Count()`，继承自`Aggregate()`。

由于`Aggregate`是表达式和​​换行表达式，因此您可以表示一些复杂的计算：

```
from django.db.models import Count

Company.objects.annotate(
    managers_required=(Count('num_employees') / 4) + Count('num_managers'))

```

`Aggregate` API如下：

_class_ `Aggregate`(_expression_, _output_field=None_, _**extra_)

`template`

类属性，作为格式字符串，描述为此聚合生成的SQL。默认为`'％（函数）s（ ％（表达式）s ）`。

`function`

描述将生成的聚合函数的类属性。具体来说，`function`将被插入为[`template`](#django.db.models.Aggregate.template "django.db.models.Aggregate.template")中的`function`占位符。默认为`None`。

`expression`参数可以是模型上的字段的名称，也可以是另一个表达式。它将转换为字符串，并用作`template`中的`expressions`占位符。

`output_field`参数需要一个模型字段实例，如`IntegerField()`或`BooleanField()`，Django将在检索后数据库。通常在实例化模型字段时不需要任何参数，因为与数据验证相关的任何参数（`max_length`，`max_digits`等）不会对表达式的输出值强制执行。

注意，只有当Django无法确定结果应该是什么字段类型时，才需要`output_field`。混合字段类型的复杂表达式应定义所需的`output_field`。例如，将`IntegerField()`和`FloatField()`添加在一起应该可以有`output_field=FloatField()`

Changed in Django 1.8:

`output_field`是一个新参数。

`**extra` kwargs是可以插入到`template`属性中的`key=value`对。

New in Django 1.8:

聚合函数现在可以在单个函数中使用算术和参考多个模型字段。

### 创建自己的聚合函数

创建自己的聚合是非常容易的。至少，您需要定义`function`，但也可以完全自定义生成的SQL。这里有一个简单的例子：

```
from django.db.models import Aggregate

class Count(Aggregate):
    # supports COUNT(distinct field)
    function = 'COUNT'
    template = '%(function)s(%(distinct)s%(expressions)s)'

    def __init__(self, expression, distinct=False, **extra):
        super(Count, self).__init__(
            expression,
            distinct='DISTINCT ' if distinct else '',
            output_field=IntegerField(),
            **extra)

```

### `Value()`表达式

_class_ `Value`(_value_, _output_field=None_)

`Value()`对象表示表达式的最小可能组件：简单值。当您需要在表达式中表示整数，布尔或字符串的值时，可以在`Value()`中包装该值。

您很少需要直接使用`Value()`。当您编写表达式`F（'field'） + 1`时，Django隐式包装`1`在`Value()`中，允许在更复杂的表达式中使用简单的值。

`value`参数描述要包括在表达式中的值，例如`1`，`True`或`None`。Django知道如何将这些Python值转换为相应的数据库类型。

`output_field`参数应为模型字段实例，如`IntegerField()`或`BooleanField()`，Django将在检索后从数据库。通常在实例化模型字段时不需要任何参数，因为与数据验证相关的任何参数（`max_length`，`max_digits`等）不会对表达式的输出值强制执行。

### `ExpressionWrapper()`表达式

_class_ `ExpressionWrapper`(_expression_, _output_field_)

New in Django 1.8.

`ExpressionWrapper`简单地包围另一个表达式，并提供对其他表达式可能不可用的属性（例如`output_field`）的访问。当对[_Using F() with annotations_](#using-f-with-annotations)中描述的不同类型的`F()`表达式使用算术时，必须使用`ExpressionWrapper`。

### 条件表达式

New in Django 1.8.

条件表达式允许您使用[`if`](https://docs.python.org/3/reference/compound_stmts.html#if "(in Python v3.4)") ...[`elif`](https://docs.python.org/3/reference/compound_stmts.html#elif "(in Python v3.4)") ...[`else`](https://docs.python.org/3/reference/compound_stmts.html#else "(in Python v3.4)")查询中的逻辑。Django本地支持SQL `CASE`表达式。有关更多详细信息，请参阅[_Conditional Expressions_](conditional-expressions.html)。

## 技术信息

下面您将找到对图书馆作者有用的技术实施细节。下面的技术API和示例将有助于创建可扩展Django提供的内置功能的通用查询表达式。

### 表达式API

查询表达式实现了[_query expression API_](lookups.html#query-expression)，但也暴露了下面列出的一些额外的方法和属性。所有查询表达式必须从`Expression()`或相关子类继承。

当查询表达式包装另一个表达式时，它负责调用包装表达式上的相应方法。

_class_ `Expression`

`contains_aggregate`

告诉Django此表达式包含聚合，并且需要将`GROUP BY`子句添加到查询中。

`resolve_expression`(_query=None_, _allow_joins=True_, _reuse=None_, _summarize=False_)

提供在将表达式添加到查询之前对表达式执行任何预处理或验证的机会。还必须在任何嵌套表达式上调用`resolve_expression()`。应该返回具有任何必要变换的`self`的`copy()`。

`query`是后端查询实现。

`allow_joins`是一个布尔值，允许或拒绝在查询中使用联接。

`reuse`是用于多连接场景的一组可重用连接。

`summarize`是一个布尔值，当`True`时，表示正在计算的查询是终端聚合查询。

`get_source_expressions`()

返回内部表达式的有序列表。例如：

```
&gt;&gt;&gt; Sum(F('foo')).get_source_expressions()
[F('foo')]

```

`set_source_expressions`(_expressions_)

获取表达式列表并存储它们，以便`get_source_expressions()`可以返回它们。

`relabeled_clone`(_change_map_)

返回`self`的克隆（副本），并重新标记任何列别名。创建子查询时，将重命名列别名。`relabeled_clone()`也应该在任何嵌套表达式上调用并分配给克隆。

`change_map`是将旧别名映射到新别名的字典。

例：

```
def relabeled_clone(self, change_map):
    clone = copy.copy(self)
    clone.expression = self.expression.relabeled_clone(change_map)
    return clone

```

`convert_value`(_self_, _value_, _expression_, _connection_, _context_)

允许表达式将`value`强制为更适当类型的钩子。

`refs_aggregate`(_existing_aggregates_)

Returns a tuple containing the `(aggregate, lookup_path)` of the first aggregate that this expression (or any nested expression) references, or `(False, ())` if no aggregate is referenced. 例如：

```
queryset.filter(num_chairs__gt=F('sum__employees'))

```

`F()`表达式此处引用前一个`Sum()`计算，这意味着此过滤器表达式应添加到`HAVING` `WHERE`子句。

在大多数情况下，对任何嵌套表达式返回`refs_aggregate`的结果应该是合适的，因为必要的内置表达式将返回正确的值。

`get_group_by_cols`()

负责返回此表达式的列引用列表。`get_group_by_cols()`应在任何嵌套表达式上调用。`F()`对象，特别是保存对列的引用。

`asc`()

返回准备好以升序排序的表达式。

`desc`()

返回准备好以降序排序的表达式。

`reverse_ordering`()

通过在`order_by`调用中反转排序顺序所需的任何修改，返回`self`。例如，执行`NULLS LAST`的表达式将其值更改为`NULLS FIRST`。仅对实现类似`OrderBy`的排序顺序的表达式需要修改。当在查询集上调用[`reverse()`](querysets.html#django.db.models.query.QuerySet.reverse "django.db.models.query.QuerySet.reverse")时，调用此方法。

### 编写自己的查询表达式

您可以编写自己的查询表达式类，这些类使用其他查询表达式，并可以与其集成。让我们通过编写一个`COALESCE` SQL函数的实现，而不使用内置的[_Func() expressions_](#func-expressions)来演示一个例子。

`COALESCE` SQL函数定义为获取列或值的列表。它将返回不是`NULL`的第一列或值。

我们将首先定义用于生成SQL的模板，然后使用`__init__()`方法来设置一些属性：

```
import copy
from django.db.models import Expression

class Coalesce(Expression):
    template = 'COALESCE( %(expressions)s )'

    def __init__(self, expressions, output_field, **extra):
      super(Coalesce, self).__init__(output_field=output_field)
      if len(expressions) < 2:
          raise ValueError('expressions must have at least 2 elements')
      for expression in expressions:
          if not hasattr(expression, 'resolve_expression'):
              raise TypeError('%r is not an Expression' % expression)
      self.expressions = expressions
      self.extra = extra

```

我们对参数进行一些基本验证，包括至少需要2列或值，并确保它们是表达式。我们在这里需要`output_field`，以便Django知道要将最终结果分配给什么样的模型字段。

现在我们实现预处理和验证。由于我们现在没有任何自己的验证，我们只是委托给嵌套表达式：

```
def resolve_expression(self, query=None, allow_joins=True, reuse=None, summarize=False):
    c = self.copy()
    c.is_summary = summarize
    for pos, expression in enumerate(self.expressions):
        c.expressions[pos] = expression.resolve_expression(query, allow_joins, reuse, summarize)
    return c

```

接下来，我们编写负责生成SQL的方法：

```
def as_sql(self, compiler, connection):
    sql_expressions, sql_params = [], []
    for expression in self.expressions:
        sql, params = compiler.compile(expression)
        sql_expressions.append(sql)
        sql_params.extend(params)
    self.extra['expressions'] = ','.join(sql_expressions)
    return self.template % self.extra, sql_params

def as_oracle(self, compiler, connection):
    """
 Example of vendor specific handling (Oracle in this case).
 Let's make the function name lowercase.
 """
    self.template = 'coalesce( %(expressions)s )'
    return self.as_sql(compiler, connection)

```

我们使用`compiler.compile()`方法为每个`expressions`生成SQL，并用逗号连接结果。然后使用我们的数据填充模板，并返回SQL和参数。

我们还定义了一个特定于Oracle后端的自定义实现。如果Oracle后端正在使用，则将调用`as_oracle()`函数，而不是`as_sql()`。

最后，我们实现允许我们的查询表达式与其他查询表达式一起播放的其他方法：

```
def get_source_expressions(self):
    return self.expressions

def set_source_expressions(self, expressions):
    self.expressions = expressions

```

让我们看看它是如何工作的：

```
>>> from django.db.models import F, Value, CharField
>>> qs = Company.objects.annotate(
...    tagline=Coalesce([
...        F('motto'),
...        F('ticker_name'),
...        F('description'),
...        Value('No Tagline')
...        ], output_field=CharField()))
>>> for c in qs:
...     print("%s: %s" % (c.name, c.tagline))
...
Google: Do No Evil
Apple: AAPL
Yahoo: Internet Company
Django Software Foundation: No Tagline

```

