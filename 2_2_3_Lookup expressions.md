# 查找 API 参考 #

```
New in Django 1.7.
```

这篇文档是查找 API 的参考，Django 用这些API 构建数据库查询的`WHERE` 子句。若要学习如何使用 查找，参见[执行查询](http://python.usyiyi.cn/django/topics/db/queries.html)；若要了解如何创建 新的查找，参见[自定义查找](http://python.usyiyi.cn/django/howto/custom-lookups.html)。

查找 API 由两个部分组成：`RegisterLookupMixin` 类，它用于注册查找；[查询表达式API](http://python.usyiyi.cn/django/ref/models/lookups.html#query-expression)，它是一个方法集，类必须实现它们才可以注册成一个查找。

Django 有两个类遵循查询表达式API，且Django 所有内建的查找都继承自它们：

+ `Lookup`：用于查找一个字段（例如`field_name__exact` 中的`exact`）
+ `Transform`：用于转换一个字段

查找表达式由三部分组成：

+ 字段部分（例如， `Book.objects.filter(author__best_friends__first_name...)`；
+ 转换部分（可以省略）（例如， `__lower__first3chars__reversed`）；
+ 查找部分（例如，`__icontains`），如果省略则默认为`__exact`。

## 注册 API ##

Django 使用`RegisterLookupMixin` 来为类提供接口，注册它自己的查找。两个最突出的例子是`Field`（所有模型字段的基类）和 `Aggregate`（Django 所有聚合函数的基类）。

`class lookups.RegisterLookupMixin`

一个mixin，实现一个类上的查找API。

`classmethod register_lookup(lookup)`

在类中注册一个新的查找。例如，`DateField.register_lookup(YearExact)` 将在`DateField `上注册一个 `YearExact`查找。它会覆盖已存在的同名查找。

`get_lookup(lookup_name)`

返回类中注册的名为`lookup_name` 的 `Lookup`。默认的实现会递归查询所有的父类，并检查它们中的任何一个是否具有名称为`lookup_name`的查找，并返回第一个匹配。

`get_transform(transform_name)`

返回一个名为`transform_name` 的`Transform`。默认的实现会递归查找所有的父类，并检查它们中的任何一个是否具有名称为`transform_name`的查找，并返回第一个匹配。

一个类如果想要成为查找，它必须实现查询表达式API。`Lookup` 和`Transform`一开始就遵循这个API。

## 查询表达式API ##

查询表达式API是一个通用的方法集，在查询表达式中可以使用定义了这些方法的类，来将它们自身转换为SQL表达式。直接的字段引用，聚合，以及`Transform`类都是遵循这个API的示例。当一个对象实现以下方法时，就被称为遵循查询表达式API：

`as_sql(self, compiler, connection)`

负责从表达式中产生查询字符串和参数。`compiler`是一个`SQLCompiler`对象，它拥有可以编译其它表达式的`compile()`方法。`connection`是用于执行查询的连接。

调用`expression.as_sql()`一般是不对的 -- 而是应该调用`compiler.compile(expression)`。 `compiler.compile()`方法应该在调用表达式的供应商特定方法时格外小心。

`as_vendorname(self, compiler, connection)`

和`as_sql()`的工作方式类似。当一个表达式经过`compiler.compile()`编译之后， Django会首先尝试调用`as_vendorname()`，其中`vendorname`是用于执行查询的后端供应商。对于Django内建的后端，`vendorname`是`postgresql`，`oracle`，`sqlite`，或者`mysql`之一。

`get_lookup(lookup_name)`

必须返回名称为`lookup_name`的查找。例如，通过返回`self.output_field.get_lookup(lookup_name)`来实现。

`get_transform(transform_name)`

必须返回名称为`transform_name的`查找。例如，通过返回`self.output_field.get_transform(transform_name)`来实现。

`output_field`

定义`get_lookup()`方法所返回的类的类型。必须为`Field`的实例。

## Transform 类参考 ##

`class Transform`

`Transform`是用于实现字段转换的通用类。一个显然的例子是`__year`会把`DateField`转换为`IntegerField`。

在表达式中执行查找的标记是`Transform<expression>__<transformation>` (例如 `date__year`)。

这个类遵循查询表达式API，也就是说你可以使用 `<expression>__<transform1>__<transform2>`。

`bilateral`

```
New in Django 1.8.
```

一个布尔值，表明是否对`lhs`和 `rhs`都应用这个转换。如果对两侧都应用转换，应用在`rhs`的顺序和在查找表达式中的出现顺序相同。默认这个属性为`False`。使用方法的实例请见自定义查找。

`lhs`

在左边，也就是被转换的东西。必须遵循查询表达式API。

`lookup_name`

查找的名称，用于在解析查询表达式的时候识别它。

`output_field`

为这个类定义转换后的输出。必须为`Field`的实例。默认情况下和`lhs.output_field`相同。

`as_sql()`

需要被覆写；否则抛出`NotImplementedError`异常。

`get_lookup(lookup_name)`

和`get_lookup()`相同。

`get_transform(transform_name)`

和`get_transform()`相同。

## Lookup 类参考 ##

`class Lookup`

`Lookup`是实现查找的通用的类。查找是一个查询表达式，它的左边是`lhs`，右边是`rhs`；`lookup_name`用于构造`lhs`和`rhs`之间的比较，来产生布尔值，例如`lhs in rhs`或者`lhs > rhs`。

在表达式中执行查找的标记是`<lhs>__<lookup_name>=<rhs>`。

这个类并不遵循查询表达式API，因为在它构造的时候出现了`=<rhs>`：查找总是在查找表达式的最后。

`lhs`

在左边，也就是被查找的东西。这个对象必须遵循查询表达式API。

`rhs`

在右边，也就是用来和`lhs`比较的东西。它可以是个简单的值，也可以是在SQL中编译的一些东西，比如 `F()` 对象或者`QuerySet`。

`lookup_name`

查找的名称，用于在解析查询表达式的时候识别它。

`process_lhs(compiler, connection[, lhs=None])`

返回元组`(lhs_string, lhs_params)`，和`compiler.compile(lhs)`所返回的一样。这个方法可以被覆写，来调整`lhs`的处理方式。

`compiler`是一个`SQLCompiler`对象，可以像 `compiler.compile(lhs)`这样使用来编译`lhs`。`connection`可以用于编译供应商特定的SQL语句。`lhs`如果不为`None`, 会代替`self.lhs`作为处理后的`lhs`使用。

`process_rhs(compiler, connection)`

对于右边的东西，和`process_lhs()`的行为相同。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Lookup expressions](https://docs.djangoproject.com/en/1.8/ref/models/lookups/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
