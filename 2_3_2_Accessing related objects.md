<!--
  译者：Github@wizardforcel
-->

# 关联对象参考 #

**class RelatedManager**

"关联管理器"是在一对多或者多对多的关联上下文中使用的管理器。它存在于下面两种情况：

ForeignKey关系的“另一边”。像这样：

```
from django.db import models

class Reporter(models.Model):
    # ...
    pass

class Article(models.Model):
    reporter = models.ForeignKey(Reporter)
```

在上面的例子中，管理器reporter.article_set拥有下面的方法。

ManyToManyField关系的两边：

```
class Topping(models.Model):
    # ...
    pass

class Pizza(models.Model):
    toppings = models.ManyToManyField(Topping)
```

这个例子中，topping.pizza_set 和pizza.toppings都拥有下面的方法。

**add(obj1[, obj2, ...])**

把指定的模型对象添加到关联对象集中。

例如：

```
>>> b = Blog.objects.get(id=1)
>>> e = Entry.objects.get(id=234)
>>> b.entry_set.add(e) # Associates Entry e with Blog b.
```

在上面的例子中，对于ForeignKey关系，e.save()由关联管理器调用，执行更新操作。然而，在多对多关系中使用add()并不会调用任何 save()方法，而是由QuerySet.bulk_create()创建关系。如果你需要在关系被创建时执行一些自定义的逻辑，请监听m2m_changed信号。

**create(\*\*kwargs)**

创建一个新的对象，保存对象，并将它添加到关联对象集之中。返回新创建的对象：

```
>>> b = Blog.objects.get(id=1)
>>> e = b.entry_set.create(
...     headline='Hello',
...     body_text='Hi',
...     pub_date=datetime.date(2005, 1, 1)
... )

# No need to call e.save() at this point -- it's already been saved.
```

这完全等价于（不过更加简洁于）：

```
>>> b = Blog.objects.get(id=1)
>>> e = Entry(
...     blog=b,
...     headline='Hello',
...     body_text='Hi',
...     pub_date=datetime.date(2005, 1, 1)
... )
>>> e.save(force_insert=True)
```

要注意我们并不需要指定模型中用于定义关系的关键词参数。在上面的例子中，我们并没有传入blog参数给create()。Django会明白新的 Entry对象blog 应该添加到b中。

**remove(obj1[, obj2, ...])**

从关联对象集中移除执行的模型对象：

```
>>> b = Blog.objects.get(id=1)
>>> e = Entry.objects.get(id=234)
>>> b.entry_set.remove(e) # Disassociates Entry e from Blog b.
```

和add()相似，上面的例子中，e.save()可会执行更新操作。但是，多对多关系上的remove()，会使用QuerySet.delete()删除关系，意思是并不会有任何模型调用save()方法：如果你想在一个关系被删除时执行自定义的代码，请监听m2m_changed信号。

对于ForeignKey对象，这个方法仅在null=True时存在。如果关联的字段不能设置为None (NULL)，则这个对象在添加到另一个关联之前不能移除关联。在上面的例子中，从b.entry_set()移除e等价于让e.blog = None，由于blog的ForeignKey没有设置null=True，这个操作是无效的。

对于ForeignKey对象，该方法接受一个bulk参数来控制它如果执行操作。如果为True（默认值），QuerySet.update()会被使用。而如果bulk=False，会在每个单独的模型实例上调用save()方法。这会触发pre_save和post_save，它们会消耗一定的性能。

**clear()**

从关联对象集中移除一切对象。

```
>>> b = Blog.objects.get(id=1)
>>> b.entry_set.clear()
```

注意这样不会删除对象 —— 只会删除他们之间的关联。

就像 remove() 方法一样，clear()只能在 null=True的ForeignKey上被调用，也可以接受bulk关键词参数。

> 注意
> 
> 注意对于所有类型的关联字段，add()、create()、remove()和clear()都会马上更新数据库。换句话说，在关联的任何一端，都不需要再调用save()方法。
> 
> 同样，如果你再多对多关系中使用了中间模型，一些关联管理的方法会被禁用。

## 直接赋值 ##

通过赋值一个新的可迭代的对象，关联对象集可以被整体替换掉。

```
>>> new_list = [obj1, obj2, obj3]
>>> e.related_set = new_list
```

如果外键关系满足null=True，关联管理器会在添加new_list中的内容之前，首先调用clear()方法来解除关联集中一切已存在对象的关联。否则， new_list中的对象会在已存在的关联的基础上被添加。