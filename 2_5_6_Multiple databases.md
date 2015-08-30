# 多数据库 #

这篇主题描述Django 对多个数据库的支持。大部分Django 文档假设你只和一个数据库打交道。如果你想与多个数据库打交道，你将需要一些额外的步骤。

## 定义你的数据库 #

在Django中使用多个数据库的第一步是告诉Django 你将要使用的数据库服务器。这通过使用`DATABASES` 设置完成。该设置映射数据库别名到一个数据库连接设置的字典，这是整个Django 中引用一个数据库的方式。字典中的设置在 `DATABASES` 文档中有完整描述。

你可以为数据库选择任何别名。然而，`default`这个别名具有特殊的含义。当没有选择其它数据库时，Django 使用具有`default` 别名的数据库。

下面是`settings.py`的一个示例片段，它定义两个数据库 —— 一个默认的PostgreSQL 数据库和一个叫做`users`的MySQL 数据库：

```
DATABASES = {
    'default': {
        'NAME': 'app_data',
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'USER': 'postgres_user',
        'PASSWORD': 's3krit'
    },
    'users': {
        'NAME': 'user_data',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'priv4te'
    }
}
```

如果`default` 数据库在你的项目中不合适，你需要小心地永远指定是想使用的数据库。Django 要求`default` 数据库必须定义，但是其参数字典可以保留为空如果不使用它。若要这样做，你必须为你的所有的应用的模型建立`DATABASE_ROUTERS`，包括正在使用的`contrib` 中的应用和第三方应用，以使得不会有查询被路由到默认的数据库。下面是`settings.py` 的一个示例片段，它定义两个非默认的数据库，其中`default` 有意保留为空：

```
DATABASES = {
    'default': {},
    'users': {
        'NAME': 'user_data',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'superS3cret'
    },
    'customers': {
        'NAME': 'customer_data',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_cust',
        'PASSWORD': 'veryPriv@ate'
    }
}
```

如果你试图访问在`DATABASES` 设置中没有定义的数据库，Django 将抛出一个`django.db.utils.ConnectionDoesNotExist`异常。

## 同步你的数据库 ##

`migrate` 管理命令一次操作一个数据库。默认情况下，它在`default` 数据库上操作，但是通过提供一个`--database` 参数，你可以告诉`migrate` 同步一个不同的数据库。因此，为了同步所有模型到我们示例中的所有数据库，你将需要调用：

```
$ ./manage.py migrate
$ ./manage.py migrate --database=users
```

如果你不想每个应用都被同步到同一台数据库上，你可以定义一个数据库路由，它实现一个策略来控制特定模型的访问性。

### 使用其它管理命令 ###

其它`django-admin` 命令与数据库交互的方式与`migrate`相同 —— 它们都一次只操作一个数据库，并使用`--database`来控制使用的数据库。

## 数据库自动路由 ##

使用多数据库最简单的方法是建立一个数据库路由模式。默认的路由模式确保对象’粘滞‘在它们原始的数据库上（例如，从`foo` 数据库中获取的对象将保存在同一个数据库中）。默认的路由模式还确保如果没有指明数据库，所有的查询都回归到`default`数据库中。

你不需要做任何事情来激活默认的路由模式 —— 它在每个Django项目上’直接‘提供。然而，如果你想实现更有趣的数据库分配行为，你可以定义并安装你自己的数据库路由。

### 数据库路由 ###

数据库路由是一个类，它提供4个方法：

`db_for_read(model, **hints)`

建议model类型的对象的读操作应该使用的数据库。

如果一个数据库操作能够提供其它额外的信息可以帮助选择一个数据库，它将在`hints`字典中提供。合法的`hints` 的详细信息在下文给出。

如果没有建议，则返回`None`。

`db_for_write(model, **hints)`

建议Model 类型的对象的写操作应该使用的数据库。

如果一个数据库操作能够提供其它额外的信息可以帮助选择一个数据库，它将在`hints`字典中提供。 合法的`hints` 的详细信息在下文给出。

如果没有建议，则返回None。

`allow_relation(obj1, obj2, **hints)`

如果`obj1` 和`obj2` 之间应该允许关联则返回`True`，如果应该防止关联则返回`False`，如果路由无法判断则返回`None`。这是纯粹的验证操作，外键和多对多操作使用它来决定两个对象之间是否应该允许一个关联。

`allow_migrate(db, app_label, model_name=None, **hints)`

定义迁移操作是否允许在别名为`db`的数据库上运行。如果操作应该运行则返回`True` ，如果不应该运行则返回`False`，如果路由无法判断则返回`None`。

位置参数`app_label `是正在迁移的应用的标签。

大部分迁移操作设置`model_name`的值为正在迁移的模型的`model._meta.model_name`（模型的`__name__` 的小写）。对于RunPython和RunSQL 操作它的值为`None`，除非这两个操作使用hint 提供它。

`hints` 用于某些操作来传递额外的信息给路由。

当设置了`model_name`时，`hints` 通常通过键'`model`'包含该模型的类。注意，它可能是一个历史模型，因此不会有自定的属性、方法或管理器。你应该只依赖`_meta`。

这个方法还可以用来决定一个给定数据库上某个模型的可用性。

注意，如果这个方法返回`False`，迁移将默默地不会在模型上做任何操作。这可能导致你应用某些操作之后出现损坏的外键、表多余或者缺失。

```
Changed in Django 1.8:

The signature of allow_migrate has changed significantly from previous versions. See the deprecation notes for more details.
```

路由不必提供所有这些方法 —— 它可以省略一个或多个。如果某个方法缺失，在做相应的检查时Django 将忽略该路由。

#### Hints ####

`Hint` 由数据库路由接收，用于决定哪个数据库应该接收一个给定的请求。

目前，唯一一个提供的hint 是instance，它是一个对象实例，与正在进行的读或者写操作关联。This might be the instance that is being saved, or it might be an instance that is being added in a many-to-many relation. In some cases, no instance hint will be provided at all. The router checks for the existence of an instance hint, and determine if that hint should be used to alter routing behavior.

### 使用路由 ###

数据库路由使用`DATABASE_ROUTERS` 设置安装。这个设置定义一个类名的列表，其中每个类表示一个路由，它们将被主路由（`django.db.router`）使用。

Django 的数据库操作使用主路由来分配数据库的使用。每当一个查询需要知道使用哪一个数据库时，它将调用主路由，并提供一个模型和一个`Hint` （可选）。Django 然后依次测试每个路由直至找到一个数据库的建议。如果找不到建议，它将尝试`Hint` 实例的当前`_state.db`。如果没有提供Hint 实例，或者该实例当前没有数据库状态，主路由将分配`default` 数据库。

### 一个例子 ###

> 只是为了示例！
>
> 这个例子的目的是演示如何使用路由这个基本结构来改变数据库的使用。它有意忽略一些复杂的问题，目的是为了演示如何使用路由。
>
> 如果`myapp `中的任何一个模型包含与其它 数据库之外的模型的关联，这个例子将不能工作。跨数据的关联引入引用完整性问题，Django目前还无法处理。
>
> `Primary/replica`（在某些数据库中叫做`master/slave`）配置也是有缺陷的 —— 它不提供任何处理Replication lag 的解决办法（例如，因为写入同步到replica 需要一定的时间，这会引入查询的不一致）。It also doesn’t consider the interaction of transactions with the database utilization strategy.


那么 —— 在实际应用中这以为着什么？让我们看一下另外一个配置的例子。这个配置将有几个数据库：一个用于`auth` 应用，所有其它应用使用一个具有两个读`replica` 的 `primary/replica`。下面是表示这些数据库的设置：

```
DATABASES = {
    'auth_db': {
        'NAME': 'auth_db',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'swordfish',
    },
    'primary': {
        'NAME': 'primary',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'spam',
    },
    'replica1': {
        'NAME': 'replica1',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'eggs',
    },
    'replica2': {
        'NAME': 'replica2',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'bacon',
    },
}
```

现在我们将需要处理路由。首先，我们需要一个路由，它知道发送`auth` 应用的查询到`auth_db`：

```
class AuthRouter(object):
    """
    A router to control all database operations on models in the
    auth application.
    """
    def db_for_read(self, model, **hints):
        """
        Attempts to read auth models go to auth_db.
        """
        if model._meta.app_label == 'auth':
            return 'auth_db'
        return None

    def db_for_write(self, model, **hints):
        """
        Attempts to write auth models go to auth_db.
        """
        if model._meta.app_label == 'auth':
            return 'auth_db'
        return None

    def allow_relation(self, obj1, obj2, **hints):
        """
        Allow relations if a model in the auth app is involved.
        """
        if obj1._meta.app_label == 'auth' or \
           obj2._meta.app_label == 'auth':
           return True
        return None

    def allow_migrate(self, db, app_label, model=None, **hints):
        """
        Make sure the auth app only appears in the 'auth_db'
        database.
        """
        if app_label == 'auth':
            return db == 'auth_db'
        return None
```

我们还需要一个路由，它发送所有其它应用的查询到`primary/replica` 配置，并随机选择一个`replica` 来读取：

```
import random

class PrimaryReplicaRouter(object):
    def db_for_read(self, model, **hints):
        """
        Reads go to a randomly-chosen replica.
        """
        return random.choice(['replica1', 'replica2'])

    def db_for_write(self, model, **hints):
        """
        Writes always go to primary.
        """
        return 'primary'

    def allow_relation(self, obj1, obj2, **hints):
        """
        Relations between objects are allowed if both objects are
        in the primary/replica pool.
        """
        db_list = ('primary', 'replica1', 'replica2')
        if obj1._state.db in db_list and obj2._state.db in db_list:
            return True
        return None

    def allow_migrate(self, db, app_label, model=None, **hints):
        """
        All non-auth models end up in this pool.
        """
        return True
```

最后，在设置文件中，我们添加如下内容（替换`path.to.`为该路由定义所在的真正路径）：

```
DATABASE_ROUTERS = ['path.to.AuthRouter', 'path.to.PrimaryReplicaRouter']
```

路由处理的顺序非常重要。路由的查询将按照DATABASE_ROUTERS设置中列出的顺序进行。在这个例子中，`AuthRouter`在`PrimaryReplicaRouter`之前处理，因此`auth`中的模型的查询处理在其它模型之前。如果`DATABASE_ROUTERS`设置按其它顺序列出这两个路由，`PrimaryReplicaRouter.allow_migrate()` 将先处理。`PrimaryReplicaRouter` 中实现的捕获所有的查询，这意味着所有的模型可以位于所有的数据库中。

建立这个配置后，让我们运行一些Django 代码：

```
>>> # This retrieval will be performed on the 'auth_db' database
>>> fred = User.objects.get(username='fred')
>>> fred.first_name = 'Frederick'

>>> # This save will also be directed to 'auth_db'
>>> fred.save()

>>> # These retrieval will be randomly allocated to a replica database
>>> dna = Person.objects.get(name='Douglas Adams')

>>> # A new object has no database allocation when created
>>> mh = Book(title='Mostly Harmless')

>>> # This assignment will consult the router, and set mh onto
>>> # the same database as the author object
>>> mh.author = dna

>>> # This save will force the 'mh' instance onto the primary database...
>>> mh.save()

>>> # ... but if we re-retrieve the object, it will come back on a replica
>>> mh = Book.objects.get(title='Mostly Harmless')
```

## 手动选择一个数据库 ##

Django 还提供一个API，允许你在你的代码中完全控制数据库的使用。人工指定的数据库的优先级高于路由分配的数据库。

### 为QuerySet手动选择一个数据库 ###

你可以在`QuerySet`“链”的任意节点上为`QuerySet`选择数据库 。只需要`在QuerySet`上调用`using()`就可以让`QuerySet`使用一个指定的数据库。

`using()` 接收单个参数：你的查询想要运行的数据库的别名。例如：

```
>>> # This will run on the 'default' database.
>>> Author.objects.all()

>>> # So will this.
>>> Author.objects.using('default').all()

>>> # This will run on the 'other' database.
>>> Author.objects.using('other').all()
```

### 为save() 选择一个数据库 ###

对`Model.save()`使用`using` 关键字来指定数据应该保存在哪个数据库。

例如，若要保存一个对象到legacy_users 数据库，你应该使用：

```
>>> my_object.save(using='legacy_users')
```

如果你不指定`using`，`save()`方法将保存到路由分配的默认数据库中。

#### 将对象从一个数据库移动到另一个数据库 ####

如果你已经保存一个实例到一个数据库中，你可能很想使用`save(using=...)` 来迁移该实例到一个新的数据库中。然而，如果你不使用正确的步骤，这可能导致意外的结果。

考虑下面的例子：

```
>>> p = Person(name='Fred')
>>> p.save(using='first')  # (statement 1)
>>> p.save(using='second') # (statement 2)
```

在statement 1中，一个新的`Person` 对象被保存到 `first` 数据库中。此时`p `没有主键，所以Django 发出一个`SQL INSERT `语句。这会创建一个主键，且Django 将此主键赋值给`p`。

当保存在statement 2中发生时，`p`已经具有一个主键，Django 将尝试在新的数据库上使用该主键。如果该主键值在`second` 数据库中没有使用，那么你不会遇到问题 —— 该对象将被复制到新的数据库中。

然而，如果`p` 的主键在`second`数据库上已经在使用`second` 数据库中的已经存在的对象将在`p`保存时被覆盖。

你可以用两种方法避免这种情况。首先，你可以清除实例的主键。如果一个对象没有主键，Django 将把它当做一个新的对象，这将避免`second`数据库上数据的丢失：

```
>>> p = Person(name='Fred')
>>> p.save(using='first')
>>> p.pk = None # Clear the primary key.
>>> p.save(using='second') # Write a completely new object.
```

第二种方法是使用`force_insert` 选项来`save()`以确保Django 使用一个`INSERT SQL`：

```
>>> p = Person(name='Fred')
>>> p.save(using='first')
>>> p.save(using='second', force_insert=True)
```

这将确保名称为`Fred` 的`Person`在两个数据库上具有相同的主键。在你试图保存到`second`数据库，如果主键已经在使用，将会引抛出发一个错误。

### 选择一个数据库用于删除表单 ###

默认情况下，删除一个已存在对象的调用将在与获取对象时使用的相同数据库上执行：

```
>>> u = User.objects.using('legacy_users').get(username='fred')
>>> u.delete() # will delete from the `legacy_users` database
```

要指定删除一个模型时使用的数据库，可以对`Model.delete()`方法使用`using` 关键字参数。这个参数的工作方式与`save()`的`using`关键字参数一样。

例如，你正在从`legacy_users` 数据库到`new_users` 数据库迁移一个`User` ，你可以使用这些命令：

```
>>> user_obj.save(using='new_users')
>>> user_obj.delete(using='legacy_users')
```

### 多个数据库上使用管理器 ###

在管理器上使用`db_manager()`方法来让管理器访问非默认的数据库。

例如，你有一个自定义的管理器方法，它访问数据库时候用 ——`User.objects.create_user()`。因为`create_user()`是一个管理器方法，不是一个`QuerySet`方法，你不可以使用`User.objects.using('new_users').create_user()`。（`create_user()` 方法只能在`User.objects`上使用，而不能在从管理器得到的`QuerySet`上使用）。解决办法是使用`db_manager()`，像这样：

```
User.objects.db_manager('new_users').create_user(...)
```

`db_manager()` 返回一个绑定在你指定的数据上的一个管理器。

#### 多数据库上使用get_queryset() ####

如果你正在覆盖你的管理器上的`get_queryset()`，请确保在其父类上调用方法（使用`super()`）或者正确处理管理器上的`_db`属性（一个包含将要使用的数据库名称的字符串）。

例如，如果你想从`get_queryset` 方法返回一个自定义的 `QuerySet` 类，你可以这样做：

```
class MyManager(models.Manager):
    def get_queryset(self):
        qs = CustomQuerySet(self.model)
        if self._db is not None:
            qs = qs.using(self._db)
        return qs
```

## Django 的管理站点中使用多数据库 ##

Django 的管理站点没有对多数据库的任何显式的支持。如果你给数据库上某个模型提供的管理站点不想通过你的路由链指定，你将需要编写自定义的`ModelAdmin`类用来将管理站点导向一个特殊的数据库。

`ModelAdmin` 对象具有5个方法，它们需要定制以支持多数据库：

```
class MultiDBModelAdmin(admin.ModelAdmin):
    # A handy constant for the name of the alternate database.
    using = 'other'

    def save_model(self, request, obj, form, change):
        # Tell Django to save objects to the 'other' database.
        obj.save(using=self.using)

    def delete_model(self, request, obj):
        # Tell Django to delete objects from the 'other' database
        obj.delete(using=self.using)

    def get_queryset(self, request):
        # Tell Django to look for objects on the 'other' database.
        return super(MultiDBModelAdmin, self).get_queryset(request).using(self.using)

    def formfield_for_foreignkey(self, db_field, request=None, **kwargs):
        # Tell Django to populate ForeignKey widgets using a query
        # on the 'other' database.
        return super(MultiDBModelAdmin, self).formfield_for_foreignkey(db_field, request=request, using=self.using, **kwargs)

    def formfield_for_manytomany(self, db_field, request=None, **kwargs):
        # Tell Django to populate ManyToMany widgets using a query
        # on the 'other' database.
        return super(MultiDBModelAdmin, self).formfield_for_manytomany(db_field, request=request, using=self.using, **kwargs)
```

这里提供的实现实现了一个多数据库策略，其中一个给定类型的所有对象都将保存在一个特定的数据库上（例如，所有的`User`保存在`other` 数据库中）。如果你的多数据库的用法更加复杂，你的`ModelAdmin`将需要反映相应的策略。

`Inlines` 可以用相似的方式处理。它们需要3个自定义的方法：

```
class MultiDBTabularInline(admin.TabularInline):
    using = 'other'

    def get_queryset(self, request):
        # Tell Django to look for inline objects on the 'other' database.
        return super(MultiDBTabularInline, self).get_queryset(request).using(self.using)

    def formfield_for_foreignkey(self, db_field, request=None, **kwargs):
        # Tell Django to populate ForeignKey widgets using a query
        # on the 'other' database.
        return super(MultiDBTabularInline, self).formfield_for_foreignkey(db_field, request=request, using=self.using, **kwargs)

    def formfield_for_manytomany(self, db_field, request=None, **kwargs):
        # Tell Django to populate ManyToMany widgets using a query
        # on the 'other' database.
        return super(MultiDBTabularInline, self).formfield_for_manytomany(db_field, request=request, using=self.using, **kwargs)
```

一旦你写好你的模型管理站点的定义，它们就可以使用任何`Admin`实例来注册：

```
from django.contrib import admin

# Specialize the multi-db admin objects for use with specific models.
class BookInline(MultiDBTabularInline):
    model = Book

class PublisherAdmin(MultiDBModelAdmin):
    inlines = [BookInline]

admin.site.register(Author, MultiDBModelAdmin)
admin.site.register(Publisher, PublisherAdmin)

othersite = admin.AdminSite('othersite')
othersite.register(Publisher, MultiDBModelAdmin)
```

这个例子建立两个管理站点。在第一个站点上，`Author` 和 `Publisher` 对象被暴露出来；`Publisher` 对象具有一个表格的内联，显示该出版社出版的书籍。第二个站点只暴露`Publishers`，而没有内联。

## 多数据库上使用原始游标 ##

如果你正在使用多个数据库，你可以使用`django.db.connections`来获取特定数据库的连接（和游标）：`django.db.connections`是一个类字典对象，它允许你使用别名来获取一个特定的连接：

```
from django.db import connections
cursor = connections['my_db_alias'].cursor()
```

## 多数据库的局限 ##

### 跨数据库关联 ###

Django 目前不提供跨多个数据库的外键或多对多关系的支持。如果你使用一个路由来路由分离到不同的数据库上，这些模型定义的任何外键和多对多关联必须在单个数据库的内部。

这是因为引用完整性的原因。为了保持两个对象之间的关联，Django 需要知道关联对象的主键是合法的。如果主键存储在另外一个数据库上，判断一个主键的合法性不是很容易。

如果你正在使用Postgres、Oracle或者MySQ 的InnoDB，这是数据库完整性级别的强制要求 —— 数据库级别的主键约束防止创建不能验证合法性的关联。

然而，如果你正在使用SQLite 或MySQL的MyISAM 表，则没有强制性的引用完整性；结果是你可以‘伪造’跨数据库的外键。但是Django 官方不支持这种配置。

### Contrib 应用的行为 ###

有几个Contrib 应用包含模型，其中一些应用相互依赖。因为跨数据库的关联是不可能的，这对你如何在数据库之间划分这些模型带来一些限制：

+ `contenttypes.ContentType`、`sessions.Session`和`sites.Site` 可以存储在分开存储在不同的数据库中，只要给出合适的路由
+ `auth`模型 —— `User`、`Group`和`Permission` —— 关联在一起并与`ContentType`关联，所以它们必须与`ContentType`存储在相同的数据库中。
+ `admin`依赖`auth`，所以它们的模型必须与`auth`在同一个数据库中。
+ `flatpages`和`redirects`依赖`sites`，所以它们必须与`sites`在同一个数据库中。

另外，一些对象在migrate在数据库中创建一张表后自动创建：

+ 一个默认的`Site`，
+ 为每个模型创建一个`ContentType`（包括没有存储在同一个数据库中的模型），
+ 为每个模型创建3个`Permission` （包括不是存储在同一个数据库中的模型）。

对于常见的多数据库架构，将这些对象放在多个数据库中没有什么用处。常见的数据库架构包括`primary/replica` 和连接到外部的数据库。因此，建议写一个数据库路由，它只允许同步这3个模型到一个数据中。对于不需要将表放在多个数据库中的Contrib 应用和第三方应用，可以使用同样的方法。

> 警告
>
> 如果你将`Content Types` 同步到多个数据库中，注意它们的主键在数据库之间可能不一致。这可能导致数据损坏或数据丢失。

&zwj;

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Multiple databases](https://docs.djangoproject.com/en/1.8/topics/db/multi-db/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
