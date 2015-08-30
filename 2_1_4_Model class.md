# Model 类参考 #

这篇文档覆盖`Model` 类的特性。关于模型的更多信息，参见[Model 完全参考指南](http://python.usyiyi.cn/django/ref/models/index.html)。

## 属性 ##

### objects ###

`Model.objects`

每个非抽象的`Model` 类必须给自己添加一个`Manager`实例。Django 确保在你的模型类中至少有一个默认的`Manager`。如果你没有添加自己的`Manager`，Django 将添加一个属性`objects`，它包含默认的`Manager` 实例。如果你添加自己的`Manager`实例的属性，默认值则不会出现。思考一下下面的例子：

```
from django.db import models

class Person(models.Model):
    # Add manager with another name
    people = models.Manager()
```

关于模型管理器的更多信息，参见[Managers](http://python.usyiyi.cn/django/topics/db/managers.html) 和 [Retrieving objects](http://python.usyiyi.cn/django/topics/db/queries.html#retrieving-objects)。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Model class](https://docs.djangoproject.com/en/1.8/ref/models/class/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
