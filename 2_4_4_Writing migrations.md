<!--
  译者：Github@wizardforcel
-->

# 编写数据库迁移 #

这一节介绍你可能遇到的在不同情况下如何分析和编写数据库迁移. 有关迁移的入门资料，请查看 the topic guide.

## 数据迁移和多数据库 ##

在使用多个数据库时，需要解决是否针对某个特定数据库运行迁移。例如，你可能 只 想在某个特定数据库上运行迁移。

为此你可以在RunPython中通过查看schema_editor.connection.alias 属性来检查数据库连接别名：

```
from django.db import migrations

def forwards(apps, schema_editor):
    if not schema_editor.connection.alias == 'default':
        return
    # Your migration code goes here

class Migration(migrations.Migration):

    dependencies = [
        # Dependencies to other migrations
    ]

    operations = [
        migrations.RunPython(forwards),
    ]
```

```
Django 1.8 中新增。
```

你也可以提供一个提示作为 **hints参数传递到数据库路由的allow_migrate() 方法：

```
myapp/dbrouters.py
class MyRouter(object):

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if 'target_db' in hints:
            return db == hints['target_db']
        return True
```

然后，要在你的迁移中利用，执行以下操作：

```
from django.db import migrations

def forwards(apps, schema_editor):
    # Your migration code goes here

class Migration(migrations.Migration):

    dependencies = [
        # Dependencies to other migrations
    ]

    operations = [
        migrations.RunPython(forwards, hints={'target_db': 'default'}),
    ]
```

如果你的RunPython或者RunSQL操作只对一个模型有影响，最佳实践是将model_name作为提示传递，使其尽可能对路由可见。这对可复用的和第三方应用极其重要。

## 添加唯一字段的迁移 ##

如果你应用了一个“朴素”的迁移，向表中一个已存在的行中添加了一个唯一的非空字段，会产生错误，因为位于已存在行中的值只会生成一次。所以需要移除唯一性的约束。

所以，应该执行下面的步骤。在这个例子中，我们会以默认值添加一个非空的UUIDField字段。你可以根据你的需要修改各个字段。

+ 把default=...和unique=True参数添加到你模型的字段中。在这个例子中，我们默认使用uuid.uuid4。
+ 运行 makemigrations 命令。
+ 编辑创建的迁移文件。

生成的迁移类看上去像这样：

```
class Migration(migrations.Migration):

    dependencies = [
        ('myapp', '0003_auto_20150129_1705'),
    ]

    operations = [
        migrations.AddField(
            model_name='mymodel',
            name='uuid',
            field=models.UUIDField(max_length=32, unique=True, default=uuid.uuid4),
        ),
    ]
```

你需要做三处更改：

+ 从已生成的迁移类中复制，添加第二个AddField操作，并改为AlterField。
+ 在第一个AddField操作中，把unique=True改为 null=True，这会创建一个中间的null字段。
+ 在两个操作之间，添加一个RunPython或RunSQL操作为每个已存在的行生成一个唯一值（例如UUID）。

最终的迁移类应该看起来是这样：

```
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.db import migrations, models
import uuid

def gen_uuid(apps, schema_editor):
    MyModel = apps.get_model('myapp', 'MyModel')
    for row in MyModel.objects.all():
        row.uuid = uuid.uuid4()
        row.save()

class Migration(migrations.Migration):

    dependencies = [
        ('myapp', '0003_auto_20150129_1705'),
    ]

    operations = [
        migrations.AddField(
            model_name='mymodel',
            name='uuid',
            field=models.UUIDField(default=uuid.uuid4, null=True),
        ),
        # omit reverse_code=... if you don't want the migration to be reversible.
        migrations.RunPython(gen_uuid, reverse_code=migrations.RunPython.noop),
        migrations.AlterField(
            model_name='mymodel',
            name='uuid',
            field=models.UUIDField(default=uuid.uuid4, unique=True),
        ),
    ]
```

现在你可以像平常一样使用migrate命令应用迁移。

注意如果你在这个迁移运行时让对象被创建，就会产生竞争条件(race condition)。在AddField之后， RunPython之前创建的对象会覆写他们原始的uuid。