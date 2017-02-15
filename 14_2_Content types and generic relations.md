

# Contenttypes 框架

Django 包含一个[`contenttypes`](#module-django.contrib.contenttypes "django.contrib.contenttypes: Provides generic interface to installed models.") 应用，它可以追踪安装在你的Django 项目里的所有应用，并提供一个高层次的、通用的接口用于与你的模型进行交互。

## 概述

Contenttypes 的核心应用是[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 模型，存在于 `django.contrib.contenttypes.models.ContentType`。[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 的实例表示并存储你的项目当中安装的应用的信息，并且新的 [`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 实例每当新的模型安装时会自动创建。

[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")  实例具有返回它们表示的模型类的方法，以及从这些模型查询对象的方法。[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 还有一个[_自定义的管理器_](../../topics/db/managers.html#custom-managers)用于添加方法来与[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")工作，以及用于获得[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")实例的特定模型。

你的模型和[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 之间的关系还可以用于一个模型实例和任意一个已经安装的模型的实例建立“generic关联”。

## 安装Contenttypes 框架

Contenttypes 框架包含在`django-admin startproject` 创建的默认的[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 列表中，但如果你移除了它或者你手动创建 [`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 列表，你可以通过添加`'django.contrib.contenttypes'`到你的 [`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 设置中来启用它。

一般来说，安装Contenttypes 框架是个好主意。许多Django 的捆绑应用需要它：

*   Admin 应用使用它来记录通过Admin 界面添加或更改每个对象的历史。
*   Django 的[`authentication 框架`](../../topics/auth/index.html#module-django.contrib.auth "django.contrib.auth: Django's authentication framework.")用它来授用户权限给特殊的模型。

## 的`ContentType`

_class_ `ContentType`[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentType)

每一个[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 实例有两个字段，共同来唯一描述一个已经安装的模型。

`app_label`

模型所在的应用的名称。 这取自模型的[`app_label`](#django.contrib.contenttypes.models.ContentType.app_label "django.contrib.contenttypes.models.ContentType.app_label") 属性，并只包括应用的Python 导入路径的_最后_的部分。例如，"django.contrib.contenttypes"的[`app_label`](#django.contrib.contenttypes.models.ContentType.app_label "django.contrib.contenttypes.models.ContentType.app_label") 是"contenttypes"。

`model`

模型的类的名称。

此外，下面的属性是可用的︰

`name`[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentType.name)

Contenttype 的人类可读的的名称。它取之于模型的[`verbose_name`](../models/fields.html#django.db.models.Field.verbose_name "django.db.models.Field.verbose_name") 属性。

Changed in Django 1.8:

在Django 1.8 之前，`name` 属性是`ContentType` 模型的一个字段。

让我们看看一个例子，看看它如何工作。如果你已经安装[`contenttypes`](#module-django.contrib.contenttypes "django.contrib.contenttypes: Provides generic interface to installed models.") 应用，那么添加[`sites应用`](sites.html#module-django.contrib.sites "django.contrib.sites: Lets you operate multiple Web sites from the same database and Django project")到你的[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 设置并运行`manage.py migrate` 来安装它，模型[`django.contrib.sites.models.Site`](sites.html#django.contrib.sites.models.Site "django.contrib.sites.models.Site") 将安装到你的数据库中。同时将创建[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 的一个具有以下值的新实例︰

*   [`app_label`](#django.contrib.contenttypes.models.ContentType.app_label "django.contrib.contenttypes.models.ContentType.app_label") 将设置为`'sites'`（Python 路径"django.contrib.sites"的最后部分）。
*   [`model`](#django.contrib.contenttypes.models.ContentType.model "django.contrib.contenttypes.models.ContentType.model") 将设置为`'site'`。

## 方法`ContentType`

每一个 [`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 都有一些方法允许你用[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")实例来到达它所代表的model, 或者从model取出对象:

`ContentType.``get_object_for_this_type`(_**kwargs_)[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentType.get_object_for_this_type)

接收[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 表示的模型所接收的[_查询参数_](../../topics/db/queries.html#field-lookups-intro)，对该模型做[`一次get() 查询`](../models/querysets.html#django.db.models.query.QuerySet.get "django.db.models.query.QuerySet.get")，然后返回相应的对象。

`ContentType.``model_class`()[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentType.model_class)

返回此[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 实例所表示的模型类。

例如，我们可以查找[`User`](auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User") 模型的[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")︰

```
>>> from django.contrib.contenttypes.models import ContentType
>>> user_type = ContentType.objects.get(app_label="auth", model="user")
>>> user_type
<ContentType: user>

```

然后使用它来查询一个特定的[`User`](auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")，或者访问 `User` 模型类︰

```
>>> user_type.model_class()
<class 'django.contrib.auth.models.User'>
>>> user_type.get_object_for_this_type(username='Guido')
<User: Guido>

```

[`get_object_for_this_type()`](#django.contrib.contenttypes.models.ContentType.get_object_for_this_type "django.contrib.contenttypes.models.ContentType.get_object_for_this_type") 和[`model_class()`](#django.contrib.contenttypes.models.ContentType.model_class "django.contrib.contenttypes.models.ContentType.model_class") 一起使用可以实现两个极其重要的功能︰

1.  使用这些方法，你可以编写高级别的泛型代码，执行查询任何已安装的模型 —— 而不是导入和使用单一特定模型的类，可以通过`app_label` 和`model` 到[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 在运行时查找，然后使用这个模型类或从它获取对象。
2.  你可以关联另一个模型到[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 作为一种绑定它到特定模型类的方式，然后使用这些方法来获取对那些模型的访问。

几个Django 捆绑的应用利用后者的技术。例如，Django 的认证框架中的[`权限系统`](auth.html#django.contrib.auth.models.Permission "django.contrib.auth.models.Permission")使用的[`Permission`](auth.html#django.contrib.auth.models.Permission "django.contrib.auth.models.Permission") 模型具有一个外键到[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")；这允许[`Permission`](auth.html#django.contrib.auth.models.Permission "django.contrib.auth.models.Permission") 表示"可以添加博客条目"或"可以删除新闻故事"的概念。

### 的`ContentTypeManager`

_class_ `ContentTypeManager`[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentTypeManager)

[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 还具有自定义的管理器[`ContentTypeManager`](#django.contrib.contenttypes.models.ContentTypeManager "django.contrib.contenttypes.models.ContentTypeManager")，它增加了下列方法︰

`clear_cache`()[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentTypeManager.clear_cache)

清除[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 用于跟踪模型的内部缓存，它已为其创建[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 实例。你可能不需要自己调用此方法；Django 将在它需要的时候自动调用。

`get_for_id`(_id_)[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentTypeManager.get_for_id)

通过ID查找[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")。由于此方法使用与[`get_for_model()`](#django.contrib.contenttypes.models.ContentTypeManager.get_for_model "django.contrib.contenttypes.models.ContentTypeManager.get_for_model") 相同的共享缓存，建议使用这个方法而不是通常的 `ContentType.objects.get(pk=id)`。

`get_for_model`(_model_[, _for_concrete_model=True_])[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentTypeManager.get_for_model)

接收一个模型类或模型的实例，并返回表示该模型的[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 实例。`for_concrete_model=False` 允许获取代理模型的[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")。

`get_for_models`(_*models_[, _for_concrete_models=True_])[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentTypeManager.get_for_models)

接收可变数目的模型类，并返回一个字典，将模型类映射到表示它们的[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 实例。`for_concrete_model=False` 允许获取代理模型的[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")。

`get_by_natural_key`(_app_label_, _model_)[[source]](../../_modules/django/contrib/contenttypes/models.html#ContentTypeManager.get_by_natural_key)

返回由给定的应用标签和模型名称唯一标识的[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 实例。这种方法的主要目的是为允许[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 对象在反序列化期间通过[_自然键_](../../topics/serialization.html#topics-serialization-natural-keys)来引用。

[`get_for_model()`](#django.contrib.contenttypes.models.ContentTypeManager.get_for_model "django.contrib.contenttypes.models.ContentTypeManager.get_for_model") 方法特别有用，当你知道你需要与[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 交互但不想要去获取模型元数据以执行手动查找的麻烦︰

```
>>> from django.contrib.auth.models import User
>>> user_type = ContentType.objects.get_for_model(User)
>>> user_type
<ContentType: user>

```

## 通用关系

在你的model添加一个外键到 [`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")这将允许你更快捷的绑定自身到其他的model class，就像上述的 [`Permission`](auth.html#django.contrib.auth.models.Permission "django.contrib.auth.models.Permission") model 一样。但是它非常有可能进一步的利用 [`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 来实现真正的 generic (有时称之为多态) relationships 在models之间。

一个简单的例子是标记系统，它可能看起来像这样：

```
from django.db import models
from django.contrib.contenttypes.fields import GenericForeignKey
from django.contrib.contenttypes.models import ContentType

class TaggedItem(models.Model):
    tag = models.SlugField()
    content_type = models.ForeignKey(ContentType)
    object_id = models.PositiveIntegerField()
    content_object = GenericForeignKey('content_type', 'object_id')

    def __str__(self):              # __unicode__ on Python 2
        return self.tag

```

一个普通的 [`ForeignKey`](../models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey") 只能指向其他任意一个model，这是说 `TaggedItem`model用一个 [`ForeignKey`](../models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")只能一个，并且也只能有这么一个model粗存tags。contenttypes application提供了一个特殊的字段 (`GenericForeignKey`) 避免了这个问题并且允许你和任何一个model建立关联关系。 

_class_ `GenericForeignKey`[[source]](../../_modules/django/contrib/contenttypes/fields.html#GenericForeignKey)

三个步骤建立 [`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey"):

1.  给你的model设置一个 [`ForeignKey`](../models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey") 字段到[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType"). 一般命名为“content_type”.
2.  给你的model设置一个字段，用来储存你想要关联的model主键值。对于大多数model,，这是一个 [`PositiveIntegerField`](../models/fields.html#django.db.models.PositiveIntegerField "django.db.models.PositiveIntegerField")字段。并且通常命名为 “object_id”.
3.  给你的model一个 [`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey")字段， 把1,2点提到的那两个字段的名词传给他。如果这两个字段名字分别为“content_type” 和 “object_id”, 你就可以省略他们 – [`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey") 默认的会自动去查找这个两个命名的字段。

`for_concrete_model`

如果为`False`,那么字段将会涉及到proxy models（代理模型）。默认是`True`. 这映射了 `for_concrete_model` 的参数到 [`get_for_model()`](#django.contrib.contenttypes.models.ContentTypeManager.get_for_model "django.contrib.contenttypes.models.ContentTypeManager.get_for_model").

`allow_unsaved_instance_assignment`

New in Django 1.8.

与[`ForeignKey.allow_unsaved_instance_assignment`](../models/fields.html#django.db.models.ForeignKey.allow_unsaved_instance_assignment "django.db.models.ForeignKey.allow_unsaved_instance_assignment")类似。

自1.7版起已弃用：此类过去在`django.contrib.contenttypes.generic`中定义。将从Django 1.9中删除从此旧位置导入的支持。

主键类型的兼容性。

 “object_id” 字段并不总是相同的，这是由于储存在相关模型中的主键类型的关系,但是他们的主键值必须被转变为相同类型，这是通过 “object_id” 字段 的 [`get_db_prep_value()`](../models/fields.html#django.db.models.Field.get_db_prep_value "django.db.models.Field.get_db_prep_value")方法。

例如, 如果你想要简历generic 关系到一个[`IntegerField`](../models/fields.html#django.db.models.IntegerField "django.db.models.IntegerField")或者[`CharField`](../models/fields.html#django.db.models.CharField "django.db.models.CharField") 为主键的模型, 你可以使用 [`CharField`](../models/fields.html#django.db.models.CharField "django.db.models.CharField") 给 “object_id”字段，因为数字是可以被 [`get_db_prep_value()`](../models/fields.html#django.db.models.Field.get_db_prep_value "django.db.models.Field.get_db_prep_value")转化为字母的。

为了更大的灵活性你也可以用 [`TextField`](../models/fields.html#django.db.models.TextField "django.db.models.TextField") ，一个没有限制最大长度的字段,，然而这可能给你的数据库带来巨大的影响。

这里没有一个一刀切的解决办法来应对哪个字段类型最好的问题。你应该估摸一下。哪些models 你想要关联，并根据此决定一个最佳方案。（废话。）

序列化`ContentType` 的对象引用。

如果你想要序列化一个建立了 generic关系的model数据(for example, when generating [`fixtures`](../../topics/testing/tools.html#django.test.TransactionTestCase.fixtures "django.test.TransactionTestCase.fixtures")) ，你应该用一个自然键来唯一的标识相关的 [`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType") 对象。有关详细信息，请参阅[_natural keys_](../../topics/serialization.html#topics-serialization-natural-keys)和[`dumpdata --natural-foreign`](../django-admin.html#django-admin-option---natural-foreign)。

这允许你像平时用[`ForeignKey`](../models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")类似的API来工作。每一个 `TaggedItem`都将有一个`content_object`字段返回对象的关联，你也可以指定这个字段或者用它来创建一个 `TaggedItem`:

```
>>> from django.contrib.auth.models import User
>>> guido = User.objects.get(username='Guido')
>>> t = TaggedItem(content_object=guido, tag='bdfl')
>>> t.save()
>>> t.content_object
<User: Guido>

```

由于 [`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey") 完成的方式问题,，你没有办法用这个字段直接执行数据库API，filters的操作。比如 (`filter()` and `exclude()`, for example) 。因为一个 [`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey")不是一个普通的字段对象t, 这些例子是_不会_工作的:

```
# This will fail
>>> TaggedItem.objects.filter(content_object=guido)
# This will also fail
>>> TaggedItem.objects.get(content_object=guido)

```

同样的, [`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey")s 是不会出现在[`ModelForm`](../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")s.

### 反向通用关系

_class_ `GenericRelation`[[source]](../../_modules/django/contrib/contenttypes/fields.html#GenericRelation)

自1.7版起已弃用：此类过去在`django.contrib.contenttypes.generic`中定义。将从Django 1.9中删除从此旧位置导入的支持。

`related_query_name`

New in Django 1.7.

默认情况下，相关对象返回到该对象的关系不存在。设置 `related_query_name` 来创建一个对象从关联对象返回到对象自身。这允许查询和筛选相关的对象。

如果你知道你最经常使用哪种型号的，你还可以添加一个“反向”的通用关系，以使其能附加一个附加的API。 例如：

```
class Bookmark(models.Model):
    url = models.URLField()
    tags = GenericRelation(TaggedItem)

```

`Bookmark`的每个实例都会有一个`tags` 属性，可以用来获取相关的 `TaggedItems`:

```
>>> b = Bookmark(url='https://www.djangoproject.com/')
>>> b.save()
>>> t1 = TaggedItem(content_object=b, tag='django')
>>> t1.save()
>>> t2 = TaggedItem(content_object=b, tag='python')
>>> t2.save()
>>> b.tags.all()
[<TaggedItem: django>, <TaggedItem: python>]

```

New in Django 1.7.

定义一个[`GenericRelation`](#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation") 伴有`related_query_name`可以允许从相关联的对象中查询。

```
tags = GenericRelation(TaggedItem, related_query_name='bookmarks')

```

这允许你从`TaggedItem`执行过滤筛选, 排序, 和其他的查询操作`Bookmark`：

```
>>> # Get all tags belonging to books containing `django` in the url
>>> TaggedItem.objects.filter(bookmarks__url__contains='django')
[<TaggedItem: django>, <TaggedItem: python>]

```

就像[`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey") 接受 content-type 和 object-ID 字段的命名为参数,，[`GenericRelation`](#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation")也是一样的。如果一个model的 generic foreignkey 字段使用的不是默认的命名,当你创建一个[`GenericRelation`](#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation") 时一定要显示的传递这个字段的命名给它。例如，假如 `TaggedItem` model关联到上述所用的字段用 `content_type_fk` 和`object_primary_key` 两个名称来创建一个 generic foreign key,然后一个 [`GenericRelation`](#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation")需要这样定义:

```
tags = GenericRelation(TaggedItem,
                       content_type_field='content_type_fk',
                       object_id_field='object_primary_key')

```

当然，如果你没有添加一个反向的关系，你可以手动做相同类型的查找：

```
>>> b = Bookmark.objects.get(url='https://www.djangoproject.com/')
>>> bookmark_type = ContentType.objects.get_for_model(b)
>>> TaggedItem.objects.filter(content_type__pk=bookmark_type.id,
...                           object_id=b.id)
[<TaggedItem: django>, <TaggedItem: python>]

```

注意，如果一个[`GenericRelation`](#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation") model 在它的[`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey") 的 `ct_field` or `fk_field` 使用了非默认值，(for example, if you had a `Comment` model that uses `ct_field="object_pk"`), 你就要设置 [`GenericRelation`](#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation")中的`content_type_field`和/或 `object_id_field`  来分别匹配 [`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey")中的`ct_field`和 `fk_field`：

```
comments = fields.GenericRelation(Comment, object_id_field="object_pk")

```

同时请注意,如果你删除了一个具有[`GenericRelation`](#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation")的对象, 任何以 [`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey") 指向他的对象也会被删除. 在上面的例子中, 如果一个 `Bookmark` 对象被删除了,任何指向它的 `TaggedItem`对象也会被同时删除。.

不同于 [`ForeignKey`](../models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey"), [`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey") 并不接受一个 [`on_delete`](../models/fields.html#django.db.models.ForeignKey.on_delete "django.db.models.ForeignKey.on_delete") 参数来控制它的行为：如果你非常渴望这种可控制行为，你应该不使用 [`GenericRelation`](#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation")来避免这种级联删除，并且这种行为控制也可以通过 [`pre_delete`](../signals.html#django.db.models.signals.pre_delete "django.db.models.signals.pre_delete")信号来提供。

### 通用关系和聚合

[_Django’s database aggregation API_](../../topics/db/aggregation.html)不能与[`GenericRelation`](#django.contrib.contenttypes.fields.GenericRelation "django.contrib.contenttypes.fields.GenericRelation")配合使用。例如，您可能会试图尝试以下操作：

```
Bookmark.objects.aggregate(Count('tags'))

```

这将不会正确工作，然而generic relation添加了额外的查询集过滤来保证正确的内容类型, 但是 [`aggregate()`](../models/querysets.html#django.db.models.query.QuerySet.aggregate "django.db.models.query.QuerySet.aggregate")方法并没有被考虑进来。现在， 如果你需要在 generic relations使用聚合，你只能不通过聚合API来计算他们。

### Generic relation在表单中

[`django.contrib.contenttypes.forms`](#module-django.contrib.contenttypes.forms "django.contrib.contenttypes.forms")模块提供：

*   [BaseGenericInlineFormSet](#django.contrib.contenttypes.forms.BaseGenericInlineFormSet "django.contrib.contenttypes.forms.BaseGenericInlineFormSet")
*   用于[`GenericForeignKey`](#django.contrib.contenttypes.fields.GenericForeignKey "django.contrib.contenttypes.fields.GenericForeignKey")的表单工厂，[`generic_inlineformset_factory()`](#django.contrib.contenttypes.forms.generic_inlineformset_factory "django.contrib.contenttypes.forms.generic_inlineformset_factory")。

_class_ `BaseGenericInlineFormSet`[[source]](../../_modules/django/contrib/contenttypes/forms.html#BaseGenericInlineFormSet)

自1.7版起已弃用：此类过去在`django.contrib.contenttypes.generic`中定义。将从Django 1.9中删除从此旧位置导入的支持。

`generic_inlineformset_factory`(_model_, _form=ModelForm_, _formset=BaseGenericInlineFormSet_, _ct_field="content_type"_, _fk_field="object_id"_, _fields=None_, _exclude=None_, _extra=3_, _can_order=False_, _can_delete=True_, _max_num=None_, _formfield_callback=None_, _validate_max=False_, _for_concrete_model=True_, _min_num=None_, _validate_min=False_)[[source]](../../_modules/django/contrib/contenttypes/forms.html#generic_inlineformset_factory)

使用[`modelformset_factory()`](../forms/models.html#django.forms.models.modelformset_factory "django.forms.models.modelformset_factory")返回`GenericInlineFormSet`。

如果它们分别与默认值，`content_type`和`object_id`不同，则必须提供`ct_field`和`fk_field`。其他参数与[`modelformset_factory()`](../forms/models.html#django.forms.models.modelformset_factory "django.forms.models.modelformset_factory")和[`inlineformset_factory()`](../forms/models.html#django.forms.models.inlineformset_factory "django.forms.models.inlineformset_factory")中记录的参数类似。

`for_concrete_model`参数对应于`GenericForeignKey`上的[`for_concrete_model`](#django.contrib.contenttypes.fields.GenericForeignKey.for_concrete_model "django.contrib.contenttypes.fields.GenericForeignKey.for_concrete_model")参数。

自1.7版起已弃用：此函数用于在`django.contrib.contenttypes.generic`中定义。将从Django 1.9中删除从此旧位置导入的支持。

Changed in Django 1.7:

`min_num`和`validate_min`。

### 管理中的通用关系

[`django.contrib.contenttypes.admin`](#module-django.contrib.contenttypes.admin "django.contrib.contenttypes.admin")模块提供[`GenericTabularInline`](#django.contrib.contenttypes.admin.GenericTabularInline "django.contrib.contenttypes.admin.GenericTabularInline")和[`GenericStackedInline`](#django.contrib.contenttypes.admin.GenericStackedInline "django.contrib.contenttypes.admin.GenericStackedInline")（[`GenericInlineModelAdmin`](#django.contrib.contenttypes.admin.GenericInlineModelAdmin "django.contrib.contenttypes.admin.GenericInlineModelAdmin")的子类别）

这些类和函数确保了generic relations在forms 和 admin的使用。有关详细信息，请参阅[_model formset_](../../topics/forms/modelforms.html)和[_admin_](admin/index.html#using-generic-relations-as-an-inline)文档。

_class_ `GenericInlineModelAdmin`[[source]](../../_modules/django/contrib/contenttypes/admin.html#GenericInlineModelAdmin)

[`GenericInlineModelAdmin`](#django.contrib.contenttypes.admin.GenericInlineModelAdmin "django.contrib.contenttypes.admin.GenericInlineModelAdmin")类继承了来自[`InlineModelAdmin`](admin/index.html#django.contrib.admin.InlineModelAdmin "django.contrib.admin.InlineModelAdmin")类的所有属性。但是，它添加了一些自己的用于处理通用关系：

`ct_field`

模型上的[`ContentType`](#django.contrib.contenttypes.models.ContentType "django.contrib.contenttypes.models.ContentType")外键字段的名称。默认为`content_type`。

`ct_fk_field`

表示相关对象的ID的整数字段的名称。默认为`object_id`。

自1.7版起已弃用：此类过去在`django.contrib.contenttypes.generic`中定义。将从Django 1.9中删除从此旧位置导入的支持。

_class_ `GenericTabularInline`[[source]](../../_modules/django/contrib/contenttypes/admin.html#GenericTabularInline)

_class_ `GenericStackedInline`[[source]](../../_modules/django/contrib/contenttypes/admin.html#GenericStackedInline)

[`GenericInlineModelAdmin`](#django.contrib.contenttypes.admin.GenericInlineModelAdmin "django.contrib.contenttypes.admin.GenericInlineModelAdmin")的子类，分别具有堆叠和表格布局。

自1.7版起已弃用：这些类以前在`django.contrib.contenttypes.generic`中定义。将从Django 1.9中删除从此旧位置导入的支持。

