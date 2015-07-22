<!--
  译者：WrongWay [www.wrongway.me]
-->

# 元选项 #

## abstract ##

**Options.abstract**

为 True， 就表示 model 是 抽象基类 (abstract base class).

## app_label##

**Options.app_label**

如果你的 model 定义在默认的 models.py 之外(比如，你现在用的 model 在 myapp.models 子模块当中)，你必须告诉 Django 该 model 属于哪个应用：

```
app_label = 'myapp'
db_table
```

**Options.db_table**

该 model 所用的数据表的名称：

```
db_table = 'music_album'
```

## 数据表名称 ##

为了节省时间，Django 会自动指定数据表名称，规则是在应用名称的结尾加一个下划线，再加该 model 的名称。

举个例子， bookstore 应用(使用 manage.py startapp bookstore 创建)，里面有个名为 Book 的 model ，那数据表的名称就是 bookstore_book 。

数据表名称可以是 SQL 保留字，也可以包含不允许出现在 Python 变量中的特殊字符(比如连字符)，这是因为 Django 会自动给列名和表名添加引号。

## db_tablespace ##

**Options.db_tablespace**

这部分是在 Django 1.0 新增的： 请查看版本文档
该 model 所用的表空间。如果数据库不支持表空间，Django 就会忽略该选项。

## get_latest_by ##

**Options.get_latest_by**

model 中某个 DateField 或 DateTimeField 的名称，在 model 的 Manager 上使用 latest 方法时要用它作为排序字段。

例如：

```
get_latest_by = "order_date"
```

详见 latest() 。

## managed ##

**Options.managed**

这部分是在 Django 1.1 是新增的： 请查看版本文档
默认值为 True，这意味着该 model 可以用 syncdb 和 reset 命令来创建和移除对应的数据表，换句话说，Django 管理了数据表的生命周期。

如果是 False，Django 就不会为当前 model 创建和删除数据表。通常用在 model 要表示某个已存在的数据表或是视图的场合。使用非托管 model (managed = False)和普通 model 的区别在于：

如果你没有声明主键字段，就得在 model 中手动添加一个自增主键字段。为了让后来的程序员在阅读代码时不产生误解，建议您在托管 model 中对数据表中的所有的字段都进行定义。

如果两个非托管 model 之间 (managed=False) 使用 ManyToManyField 相关联，就不会创建存储多对多关系的中间表。但是普通 model 和非托管 model 之间的多对多关系仍然会创建中间表。

如果你要更改默认的行为，就得显式定义中介 model 来创建中间表(该 model 要启用managed=True)，然后在你的源 model 上令 ManyToManyField.through 指向中介 model ，就能实现定制的多对多关系。

如果你的测试包含非托管 model (managed=False)，那么在测试之前，要确保你已经创建了正确的数据表。

如果你想更改 model 类中某个 Python 层级的行为，你可以令 managed=False ，然后创建 model 的拷贝，在拷贝中定义新的行为。不过有个更好的办法，就是使用 代理 model (Proxy models).

## order_with_respect_to ##

**Options.order_with_respect_to**

根据给定的字段对 model 排序。在关联关系中，它经常用在根据目标对象对源对象排序的场合。举个例子，一个 Answer 只关联一个 Question 对象，而一个 question 对象却可以关联多个 answer对象。根据 question 对 answer 排序：

```
class Answer(models.Model):
    question = models.ForeignKey(Question)
    # ...

    class Meta:
        order_with_respect_to = 'question'
ordering
```

**Options.ordering**

该对象的默认排序项，它决定了对象列表的排序：

```
ordering = ['-order_date']
```

该项是一个字符串元组或列表。如果在字符串前面有一个"-"负号，就表示降序排列。没有"-"负号，就表示升序排列。有一个"?"问号，就表示随机排序。

> 注意
> 
> 无论 ordering 中有多少字段，Django 管理后台只使用第一个字符串。

举个例子，要根据 pub_date 字段做升序排列，使用：

```
ordering = ['pub_date']
```

根据 pub_date 字段做降序排列，使用：

```
ordering = ['-pub_date']
```

先根据 pub_date 降序，再根据 author 升序，使用：

```
ordering = ['-pub_date', 'author']
```

## permissions ##

**Options.permissions**

在创建对象时，添加到权限表当中的附加权限信息。Django 自动为每个设置了 admin 的对象创建了添加，删除和修改的权限。下面这个例子展示了如何添加一个附加的权限 can_deliver_pizzas ：

```
permissions = (("can_deliver_pizzas", "Can deliver pizzas"),)
```

该项可以是二元组，也可以是列表 (permission_code, human_readable_permission_name) 。

## proxy ##

**Options.proxy**

这部分是在 Django 1.1 中新增的： 请查看版本文档
如果设为 True，表示该 model 是父类的 代理 model (proxy model) 。

## unique_together ##

**Options.unique_together**

用来设置的不重复的字段组合：

```
unique_together = (("driver", "restaurant"),)
```

它是一个字段名称的列表，列表内的字段组合在数据库中是唯一，不重复的，也就是说不可以有两个对象，它们在列表中的字段值是完全相同的。它被用在 Django 的管理后台，在数据库层级约束数据。(比如，在 CREATE TABLE 语句中包含 UNIQUE 语句)

这部分是在 Django 1.0 中新增的： 请查看版本文档
为了使用方便，你可以赋给该项一个单独的字段列表，这表示列表内每个字段的值都必须是全表唯一的：

```
unique_together = ("driver", "restaurant")
```

## verbose_name ##

**Options.verbose_name**

易于理解和表述的对象名称--单数形式的自述名：

```
verbose_name = "pizza"
```

如果没有赋值，Django 会用该 model 名称的分词形式做为自述名： CamelCase 变成 camel case。

## verbose_name_plural ##

**Options.verbose_name_plural**

自述名的复数形式：

```
verbose_name_plural = "stories"
```

如果没有赋值，Django 使用 verbose_name + "s" 作为自述名的复数形式。