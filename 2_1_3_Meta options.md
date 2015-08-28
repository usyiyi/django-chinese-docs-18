<!--
  译者：Github@wizardforcel
-->

# 模型元选项 #

这篇文档阐述了所有可用的元选项，你可以在你模型的Meta类中设置他们。

## 可用的元选项 ##

### abstract ###

**Options.abstract**

如果 abstract = True， 就表示模型是 抽象基类 (abstract base class).

### app_label ###

**Options.app_label**

如果你的模型定义在默认的 models.py 之外(比如，你现在用的模型在 myapp.models 子模块当中)，你必须告诉 Django 该模型属于哪个应用：

```
app_label = 'myapp'
```

```
Django 1.7中新增：

一个应用中，定义在models 模块以外的模型，不再需要app_label。
```

### db_table ###

**Options.db_table**

该模型所用的数据表的名称：

```
db_table = 'music_album'
```

#### 数据表名称 ####

为了节省时间，Django 会根据模型类的名称和包含它的app名称自动指定数据表名称，一个模型的数据表名称，由这个模型的“应用标签”（在 manage.py startapp中使用的名称）之间加上下划线组成。

举个例子，  bookstore应用(使用  manage.py startapp bookstore 创建)，里面有个名为 Book的模型，那数据表的名称就是  bookstore_book 。

使用 Meta类中的 db_table 参数来覆写数据表的名称。

数据表名称可以是 SQL 保留字，也可以包含不允许出现在 Python 变量中的特殊字符，这是因为 Django 会自动给列名和表名添加引号。

> 在 MySQL中使用小写字母为表命名
> 
> 当你通过db_table覆写表名称时，强烈推荐使用小写字母给表命名，特别是如果你用了MySQL作为后端。详见MySQL注意事项 。

> Oracle中表名称的引号处理
> 
> 为了遵从Oracle中30个字符的限制，以及一些常见的约定，Django会缩短表的名称，而且会把它全部转为大写。在db_table的值外面加上引号来避免这种情况：
> 
```
db_table = '"name_left_in_lowercase"'
```
> 
> 这种带引号的名称也可以用于Django所支持的其他数据库后端，但是除了Oracle，引号不起任何作用。详见 Oracle 注意事项 。

### db_tablespace ###

**Options.db_tablespace**

当前模型所使用的数据库表空间 的名字。默认值是项目设置中的DEFAULT_TABLESPACE，如果它存在的话。如果后端并不支持表空间，这个选项可以忽略。

### default_related_name ###

**Options.default_related_name**

```
Django 1.8中新增：
```

这个名字会默认被用于一个关联对象到当前对象的关系。默认为 <model_name>_set。

由于一个字段的反转名称应该是唯一的，当你给你的模型设计子类时，要格外小心。为了规避名称冲突，名称的一部分应该含有'%(app_label)s'和'%(model_name)s'，它们会被应用标签的名称和模型的名称替换，二者都是小写的。详见抽象模型的关联名称。

### get_latest_by ###

**Options.get_latest_by**

模型中某个可排序的字段的名称，比如DateField、DateTimeField或者IntegerField。它指定了Manager的latest()和earliest()中使用的默认字段。

例如：

```
get_latest_by = "order_date"
```

详见latest() 文档。

### managed ###

**Options.managed**

默认为True，意思是Django在migrate命令中创建合适的数据表，并且会在 flush 管理命令中移除它们。换句话说，Django会管理这些数据表的生命周期。

如果是False，Django 就不会为当前模型创建和删除数据表。如果当前模型表示一个已经存在的，通过其它方法建立的数据库视图或者数据表，这会相当有用。这是设置为managed=False时唯一的不同之处。. 模型处理的其它任何方面都和平常一样。这包括：

+ 如果你不声明它的话，会向你的模型中添加一个自增主键。为了避免给后面的代码读者带来混乱，强烈推荐你在使用未被管理的模型时，指定数据表中所有的列。
+ 如果一个带有managed=False的模型含有指向其他未被管理模型的ManyToManyField，那么多对多连接的中介表也不会被创建。但是，一个被管理模型和一个未被管理模型之间的中介表会被创建。

如果你需要修改这一默认行为，创建中介表作为显式的模型（设置为managed），并且使用ManyToManyField.through为你的自定义模型创建关联。

对于带有managed=False的模型的测试，你要确保在测试启动时建立正确的表。

如果你对修改模型类在Python层面的行为感兴趣，你可以设置 managed=False ，并且创建一个已经存在模型的部分。但是这种情况下使用代理模型才是更好的方法。

### order_with_respect_to ###

**Options.order_with_respect_to**

按照给定的字段把这个对象标记为”可排序的“。这一属性通常用到关联对象上面，使它在父对象中有序。比如，如果Answer和 Question相关联，一个问题有至少一个答案，并且答案的顺序非常重要，你可以这样做：

```
from django.db import models

class Question(models.Model):
    text = models.TextField()
    # ...

class Answer(models.Model):
    question = models.ForeignKey(Question)
    # ...

    class Meta:
        order_with_respect_to = 'question'
```

当order_with_respect_to 设置之后，模型会提供两个用于设置和获取关联对象顺序的方法：get_RELATED_order() 和set_RELATED_order()，其中RELATED是小写的模型名称。例如，假设一个 Question 对象有很多相关联的Answer对象，返回的列表中含有相关联Answer对象的主键：

```
>>> question = Question.objects.get(id=1)
>>> question.get_answer_order()
[1, 2, 3]
```

与Question对象相关联的Answer对象的顺序，可以通过传入一格包含Answer 主键的列表来设置：

```
>>> question.set_answer_order([3, 1, 2])
```

相关联的对象也有两个方法， get_next_in_order() 和get_previous_in_order()，用于按照合适的顺序访问它们。假设Answer对象按照 id来排序：

```
>>> answer = Answer.objects.get(id=2)
>>> answer.get_next_in_order()
<Answer: 3>
>>> answer.get_previous_in_order()
<Answer: 1>
```

> 修改 order_with_respect_to
> 
> order_with_respect_to属性会添加一个额外的字段（/数据表中的列）叫做_order，所以如果你在首次迁移之后添加或者修改了order_with_respect_to属性，要确保执行和应用了合适的迁移操作。

### ordering ###

**Options.ordering**

对象默认的顺序，获取一个对象的列表时使用：

```
ordering = ['-order_date']
```

它是一个字符串的列表或元组。每个字符串是一个字段名，前面带有可选的“-”前缀表示倒序。前面没有“-”的字段表示正序。使用"?"来表示随机排序。

例如，要按照pub_date字段的正序排序，这样写：

```
ordering = ['pub_date']
```

按照pub_date字段的倒序排序，这样写：

```
ordering = ['-pub_date']
```

先按照pub_date的倒序排序，再按照 author 的正序排序，这样写：

```
ordering = ['-pub_date', 'author']
```

> 警告
> 
> 排序并不是没有任何代价的操作。你向ordering属性添加的每个字段都会产生你数据库的开销。你添加的每个外键也会隐式包含它的默认顺序。

### permissions ###

**Options.permissions**

设置创建对象时权限表中额外的权限。增加、删除和修改权限会自动为每个模型创建。这个例子指定了一种额外的权限，can_deliver_pizzas：

```
permissions = (("can_deliver_pizzas", "Can deliver pizzas"),)
```

它是一个包含二元组的元组或者列表，格式为 (permission_code, human_readable_permission_name)。

### default_permissions ###

**Options.default_permissions**

```
Django 1.7中新增：
```

默认为('add', 'change', 'delete')。你可以自定义这个列表，比如，如果你的应用不需要默认权限中的任何一项，可以把它设置成空列表。在模型被migrate命令创建之前，这个属性必须被指定，以防一些遗漏的属性被创建。

### proxy ###

**Options.proxy**

如果proxy = True, 作为该模型子类的另一个模型会被视为代理模型。

### select_on_save ###

**Options.select_on_save**

该选项决定了Django是否采用1.6之前的 django.db.models.Model.save()算法。旧的算法使用SELECT来判断是否存在需要更新的行。而新式的算法直接尝试使用 UPDATE。在一些小概率的情况中，一个已存在的行的UPDATE操作并不对Django可见。比如PostgreSQL的ON UPDATE触发器会返回NULL。这种情况下，新式的算法会在最后执行 INSERT 操作，即使这一行已经在数据库中存在。

通常这个属性不需要设置。默认为False。

关于旧式和新式两种算法，请参见django.db.models.Model.save()。

### unique_together ###

**Options.unique_together**

用来设置的不重复的字段组合：

```
unique_together = (("driver", "restaurant"),)
```

它是一个元组的元组，组合起来的时候必须是唯一的。它在Django后台中被使用，在数据库层上约束数据(比如，在  CREATE TABLE  语句中包含  UNIQUE语句)。

为了方便起见，处理单一字段的集合时，unique_together 可以是一维的元组：

```
unique_together = ("driver", "restaurant")
```

ManyToManyField不能包含在unique_together中。（这意味着什么并不清楚！）如果你需要验证ManyToManyField关联的唯一性，试着使用信号或者显式的贯穿模型(explicit through model)。

```
Django 1.7中修改：
当unique_together的约束被违反时，模型校验期间会抛出ValidationError异常。
```

### index_together ###

**Options.index_together**

用来设置带有索引的字段组合：

```
index_together = [
    ["pub_date", "deadline"],
]
```

列表中的字段将会建立索引（例如，会在CREATE INDEX语句中被使用）。

```
Django 1.7中修改：
```

为了方便起见，处理单一字段的集合时，index_together可以是一个一维的列表。

```
index_together = ["pub_date", "deadline"]
```

### verbose_name ###

**Options.verbose_name**

对象的一个易于理解的名称，为单数：

```
verbose_name = "pizza"
```

如果此项没有设置，Django会把类名拆分开来作为自述名，比如CamelCase 会变成camel case，

### verbose_name_plural ###

**Options.verbose_name_plural**

该对象复数形式的名称：

```
verbose_name_plural = "stories"
```

如果此项没有设置，Django 会使用 verbose_name + "s"。