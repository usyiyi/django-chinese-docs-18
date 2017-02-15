

# Django Admin 站点

Django 最强大的部分之一是自动生成的Admin 界面。它读取模型中的元数据来提供一个强大的、生产环境就绪的界面，使内容提供者能立即用它向站点中添加内容。在这篇文档中，我们讨论如何去激活、使用和自定义Django 的Admin 界面。

## 概述

通过使用[`startproject`](../../django-admin.html#django-admin-startproject) 创建的默认项目模版中，Admin 已启用。

下面的一些要求作为参考：

1.  添加 `'django.contrib.admin'`到[`INSTALLED_APPS`](../../settings.html#std:setting-INSTALLED_APPS) 设置中.
2.  admin有四个依赖 - [`django.contrib.auth`](../../../topics/auth/index.html#module-django.contrib.auth "django.contrib.auth: Django's authentication framework."), [`django.contrib.contenttypes`](../contenttypes.html#module-django.contrib.contenttypes "django.contrib.contenttypes: Provides generic interface to installed models."), [`django.contrib.messages`](../messages.html#module-django.contrib.messages "django.contrib.messages: Provides cookie- and session-based temporary message storage.") 和[`django.contrib.sessions`](../../../topics/http/sessions.html#module-django.contrib.sessions "django.contrib.sessions: Provides session management for Django projects."). 如果这些应用没有在 [`INSTALLED_APPS`](../../settings.html#std:setting-INSTALLED_APPS) 列表中, 那你要把它们添加到该列表中.
3.  把`django.contrib.messages.context_processors.messages` 添加到[`TEMPLATES`](../../settings.html#std:setting-TEMPLATES) 中`DjangoTemplates`后台的`'context_processors'`选项中，同样把[`django.contrib.auth.middleware.AuthenticationMiddleware`](../../middleware.html#django.contrib.auth.middleware.AuthenticationMiddleware "django.contrib.auth.middleware.AuthenticationMiddleware") 和 [`django.contrib.messages.middleware.MessageMiddleware`](../../middleware.html#django.contrib.messages.middleware.MessageMiddleware "django.contrib.messages.middleware.MessageMiddleware") 添加到 [`MIDDLEWARE_CLASSES`](../../settings.html#std:setting-MIDDLEWARE_CLASSES). (这些默认都是激活的，所以如果你手工操作过的话就需要按照以上方法进行设置..)
4.  确定应用中的哪些模型应该在Admin 界面中可以编辑。
5.  给每个模型创建一个`ModelAdmin` 类，封装模型自定义的Admin 功能和选项。
6.  实例化`AdminSite` 并且告诉它你的每一个模块和`ModelAdmin` 类.
7.  将`AdminSite` 实例绑定到URLconf。

做了这些步骤之后, 你将能够通过你已经绑定的URL来访问Django管理站点(默认是`/admin/`).

### 其他话题

*   [Admin 行为](actions.html)
*   [ Django admin 文档生成](admindocs.html)

注意

如何在产品中使用admin相关的静态文件(图片,JavaScript和CSS)的办法, 请参阅[_文件服务_](../../../howto/deployment/wsgi/modwsgi.html#serving-files)

还有什么问题? 试试  [_FAQ: The admin_](../../../faq/admin.html).

## ModelAdmin objects

_class_ `ModelAdmin`

`ModelAdmin` 类是模型在Admin 界面中的表示形式。通常，将它们在你的应用中的名为`admin.py`的文件里。让我们来看一个关于`ModelAdmin`类非常简单的例子:

```
from django.contrib import admin
from myproject.myapp.models import Author

class AuthorAdmin(admin.ModelAdmin):
    pass
admin.site.register(Author, AuthorAdmin)

```

你真的需要一个`ModelAdmin` 对象吗?

在上面的例子中，`ModelAdmin`并没有定义任何自定义的值。因此, 系统将使用默认的Admin 界面。如果对于默认的Admin 界面足够满意，那你根本不需要自己定义`ModelAdmin` 对象, 你可以直接注册模型类而无需提供`ModelAdmin` 的描述。那么上面的例子可以简化成：

```
from django.contrib import admin
from myproject.myapp.models import Author

admin.site.register(Author)

```

### 注册装饰器

`register`(_*models_[, _site=django.admin.sites.site_])

New in Django 1.7.

还可以用一个装饰来注册您的`ModelAdmin`类（这里有关装饰器的详细信息,请参考python中的相关说明）:

```
from django.contrib import admin
from .models import Author

@admin.register(Author)
class AuthorAdmin(admin.ModelAdmin):
    pass

```

如果不使用默认的`AdminSite`，可以提供一个或多个模块类来注册`ModelAdmin` 并且一个选择性关键参数 `site`（这里使用装饰器来注册需要注册的类和模块的，请特别留意紧跟装饰器后面关于ModelAdmin的声明，前面是Author，后面是PersonAdmin，我的理解是后一种情况 下注册的类都可以用PersonAdmin来作为接口）：

```
from django.contrib import admin
from .models import Author, Reader, Editor
from myproject.admin_site import custom_admin_site

@admin.register(Author, Reader, Editor, site=custom_admin_site)
class PersonAdmin(admin.ModelAdmin):
    pass

```

### 发现admin 文件

当你将 `'django.contrib.admin'`加入到[`INSTALLED_APPS`](../../settings.html#std:setting-INSTALLED_APPS) 设置中, Django就会自动搜索每个应用的`admin`模块并将其导入。

_class_ `apps.``AdminConfig`

New in Django 1.7.

这是 admin的默认[`AppConfig`](../../applications.html#django.apps.AppConfig "django.apps.AppConfig") 类. 它在 Django 启动时调用[`autodiscover()`](#django.contrib.admin.autodiscover "django.contrib.admin.autodiscover") .

_class_ `apps.``SimpleAdminConfig`

New in Django 1.7.

这个类和 [`AdminConfig`](#django.contrib.admin.apps.AdminConfig "django.contrib.admin.apps.AdminConfig")的作用一样,除了它不调用[`autodiscover()`](#django.contrib.admin.autodiscover "django.contrib.admin.autodiscover").

`autodiscover`()[[source]](../../../_modules/django/contrib/admin.html#autodiscover)

这个函数尝试导入每个安装的应用中的`admin` 模块。这些模块用于注册模型到Admin 中。

Changed in Django 1.7:

以前的Django版本推荐直接在URLconf中调用这个函数. Django 1.7不再需要这样. [`AdminConfig`](#django.contrib.admin.apps.AdminConfig "django.contrib.admin.apps.AdminConfig") 能够自动的运行 auto-discovery.

如果你使用自定义的`AdminSite`, 一般是导入所有的`ModelAdmin` 子类到你的代码中并将其注册到自定义的`AdminSite`中. 在这种情况下， 为了禁用auto-discovery,在你的[`INSTALLED_APPS`](../../settings.html#std:setting-INSTALLED_APPS) 设置中，应该用 `'django.contrib.admin.apps.SimpleAdminConfig'`代替`'django.contrib.admin'` 。

Changed in Django 1.7:

在以前的版本中,admin需要被指示寻找 `admin.py` 文件通过 [`autodiscover()`](#django.contrib.admin.autodiscover "django.contrib.admin.autodiscover"). 在Django 1.7, auto-discovery默认可用的，必须明确的使它失效当不需要时.

### ModelAdmin options

`ModelAdmin` 非常灵活。 它有几个选项来处理自定义界面。 所有的选项都在 `ModelAdmin` 子类中定义：

```
from django.contrib import admin

class AuthorAdmin(admin.ModelAdmin):
    date_hierarchy = 'pub_date'

```

`ModelAdmin.``actions`

在修改列表页面可用的操作列表。详细信息请查看[_Admin actions_](actions.html) .

`ModelAdmin.``actions_on_top`

`ModelAdmin.``actions_on_bottom`

控制actions bar 出现在页面的位置。默认情况下，admin的更改列表将操作显示在页面的顶部(`actions_on_top = True;`actions_on_bottom=False）。

`ModelAdmin.``actions_selection_counter`

控制选择计数器是否紧挨着下拉菜单action默认的admin 更改列表将会显示它 (`actions_selection_counter = True`).

`ModelAdmin.``date_hierarchy`

把 `date_hierarchy` 设置为在你的model 中的`DateField`或`DateTimeField`的字段名，然后更改列表将包含一个依据这个字段基于日期的下拉导航。

例如:

```
date_hierarchy = 'pub_date'

```

这将根据现有数据智能地填充自己，例如，如果所有的数据都是一个月里的, 它将只显示天级别的数据.

注意

`date_hierarchy` 在内部使用[`QuerySet.datetimes()`](../../models/querysets.html#django.db.models.query.QuerySet.datetimes "django.db.models.query.QuerySet.datetimes"). 当时区支持启用时，请参考它的一些文档说明。([`USE_TZ = True`](../../settings.html#std:setting-USE_TZ)).

`ModelAdmin.``exclude`

如果设置了这个属性，它表示应该从表单中去掉的字段列表。

例如，让我们来考虑下面的模型：

```
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    title = models.CharField(max_length=3)
    birth_date = models.DateField(blank=True, null=True)

```

如果你希望`Author` 模型的表单只包含`name` 和`title` 字段, 你应该显式说明`fields` 或`exclude`，像这样：

```
from django.contrib import admin

class AuthorAdmin(admin.ModelAdmin):
    fields = ('name', 'title')

class AuthorAdmin(admin.ModelAdmin):
    exclude = ('birth_date',)

```

由于Author 模型只有三个字段，`name`、`title`和 `birth_date`，上述声明产生的表单将包含完全相同的字段。

`ModelAdmin.``fields`

如果需要实现字段的布局中的“添加” 和 “更改”，“更改”网页形式的简单改变像只显示可用字段的一个子集，你可以使用 `fields`选项修改他们的顺序或者行内分组(需要复杂布局的请参阅[`fieldsets`](#django.contrib.admin.ModelAdmin.fieldsets "django.contrib.admin.ModelAdmin.fieldsets") 选项将在下一段讲到). 例如,可以定义一个简单的管理表单的版本使用[`django.contrib.flatpages.models.FlatPage`](../flatpages.html#django.contrib.flatpages.models.FlatPage "django.contrib.flatpages.models.FlatPage") 模块像下面这样:

```
class FlatPageAdmin(admin.ModelAdmin):
    fields = ('url', 'title', 'content')

```

在上面的例子中, 只有字段`url`, `title` 和 `content` 将会在表单中顺序的显示. `fields`能够包含在 [`ModelAdmin.readonly_fields`](#django.contrib.admin.ModelAdmin.readonly_fields "django.contrib.admin.ModelAdmin.readonly_fields") 中定义的作为只读显示的值

不同于 [`list_display`](#django.contrib.admin.ModelAdmin.list_display "django.contrib.admin.ModelAdmin.list_display")，`fields` 选项 只包含model中的字段名或者通过[`form`](#django.contrib.admin.ModelAdmin.form "django.contrib.admin.ModelAdmin.form")指定的表单。只有当它们列在[`readonly_fields`](#django.contrib.admin.ModelAdmin.readonly_fields "django.contrib.admin.ModelAdmin.readonly_fields")中，它才能包含callables

要在同一行显示多个字段， 就把那些字段打包在一个元组里。例子中， `url` 和 `title` 字段 会显示在同一行， `content` 字段将会显示在他们的下一行里：

```
class FlatPageAdmin(admin.ModelAdmin):
    fields = (('url', 'title'), 'content')

```

注意

此`字段`选项不应与[`fieldsets`](#django.contrib.admin.ModelAdmin.fieldsets "django.contrib.admin.ModelAdmin.fieldsets")选项中的` fields `字典键混淆，如下一节所述。

如果`fields`和[`fieldsets`](#django.contrib.admin.ModelAdmin.fieldsets "django.contrib.admin.ModelAdmin.fieldsets") 选项都不存在, Django将会默认显示每一个不是 `AutoField` 并且 `editable=True`的字段, 在单一的字段集，和在模块中定义的字段有相同的顺序

`ModelAdmin.``fieldsets`

设置`fieldsets` 控制管理“添加”和 “更改” 页面的布局.

`fieldsets` 是一个以二元元组为元素的列表, 每一个二元元组代表一个在管理表单的 `&lt;fieldset&gt;`( `&lt;fieldset&gt;` 是表单的一部分.)

二元元组的格式是 `(name, field_options)`, 其中 `name` 是一个字符串相当于 fieldset的标题， `field_options` 是一个关于 fieldset的字典信息,一个字段列表包含在里面。

一个完整的例子, 来自于[`django.contrib.flatpages.models.FlatPage`](../flatpages.html#django.contrib.flatpages.models.FlatPage "django.contrib.flatpages.models.FlatPage") 模块:

```
from django.contrib import admin

class FlatPageAdmin(admin.ModelAdmin):
    fieldsets = (
        (None, {
            'fields': ('url', 'title', 'content', 'sites')
        }),
        ('Advanced options', {
            'classes': ('collapse',),
            'fields': ('enable_comments', 'registration_required', 'template_name')
        }),
    )

```

在管理界面的结果看起来像这样:

![../../../_images/flatfiles_admin.png](../../../_images/flatfiles_admin.png)

如果`fields`和[`fieldsets`](#django.contrib.admin.ModelAdmin.fields "django.contrib.admin.ModelAdmin.fields") 选项都不存在, Django将会默认显示每一个不是 `AutoField` 并且 `editable=True`的字段, 在单一的字段集，和在模块中定义的字段有相同的顺序。

`field_options` 字典有以下关键字:

*   `fields`

    字段名元组将显示在该fieldset. 此键必选.

    例如:

    ```
    {
    'fields': ('first_name', 'last_name', 'address', 'city', 'state'),
    }

    ```

    就像[`fields`](#django.contrib.admin.ModelAdmin.fields "django.contrib.admin.ModelAdmin.fields") 选项, 显示多个字段在同一行, 包裹这些字段在一个元组. 在这个例子中,  `first_name` 和 `last_name` 字段将显示在同一行:

    ```
    {
    'fields': (('first_name', 'last_name'), 'address', 'city', 'state'),
    }

    ```

    `fields` 能够包含定义在[`readonly_fields`](#django.contrib.admin.ModelAdmin.readonly_fields "django.contrib.admin.ModelAdmin.readonly_fields") 中显示的值作为只读.

    如果添加可调用的名称到`fields`中,相同的规则适用于[`fields`](#django.contrib.admin.ModelAdmin.fields "django.contrib.admin.ModelAdmin.fields")选项: 可调用的必须在 [`readonly_fields`](#django.contrib.admin.ModelAdmin.readonly_fields "django.contrib.admin.ModelAdmin.readonly_fields")列表中.

*   `classes`

    一个列表包含额外的CSS classes 应用到 fieldset.

    例如:

    ```
    {
    'classes': ('wide', 'extrapretty'),
    }

    ```

    通过默认的管理站点样式表定义的两个有用的classes 是 `collapse` 和 `wide`. Fieldsets 使用 `collapse` 样式将会在初始化时展开并且替换掉一个 “click to expand” 链接. Fieldsets 使用 `wide` 样式将会有额外的水平空格.

*   `description`

    一个可选择额外文本的字符串显示在每一个fieldset的顶部,在fieldset头部的底下. 字符串没有被[`TabularInline`](#django.contrib.admin.TabularInline "django.contrib.admin.TabularInline") 渲染由于它的布局.

    记住这个值_不是_ HTML-escaped 当它显示在管理接口中时. 如果你愿意，这允许你包括HTML。另外，你可以使用纯文本和 `django.utils.html.escape()` 避免任何HTML特殊字符。

`ModelAdmin.``filter_horizontal`

默认的, [`ManyToManyField`](../../models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 会在管理站点上显示一个`&lt;select multiple&gt;`.（多选框）．但是，当选择多个时多选框非常难用. 添加一个 [`ManyToManyField`](../../models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField")到该列表将使用一个漂亮的低调的JavaScript中的“过滤器”界面,允许搜索选项。选和不选选项框并排出现。参考[`filter_vertical`](#django.contrib.admin.ModelAdmin.filter_vertical "django.contrib.admin.ModelAdmin.filter_vertical") 使用垂直界面。

`ModelAdmin.``filter_vertical`

与[`filter_horizontal`](#django.contrib.admin.ModelAdmin.filter_horizontal "django.contrib.admin.ModelAdmin.filter_horizontal")相同，但使用过滤器界面的垂直显示，其中出现在所选选项框上方的未选定选项框。

`ModelAdmin.``form`

默认情况下， 会根据你的模型动态创建一个`ModelForm`。 它被用来创建呈现在添加/更改页面上的表单。你可以很容易的提供自己的`ModelForm` 来重写表单默认的添加/修改行为。或者，你可以使用[`ModelAdmin.get_form()`](#django.contrib.admin.ModelAdmin.get_form "django.contrib.admin.ModelAdmin.get_form") 方法自定义默认的表单，而不用指定一个全新的表单。

例子见[_添加自定义验证到Admin 中_](#admin-custom-validation)部分。

注

如果你在[`ModelForm`](../../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")中定义 `Meta.model`属性，那么也必须定义 `Meta.fields`或`Meta.exclude`属性。然而，当admin本身定义了fields，则`Meta.fields`属性将被忽略。

如果`ModelForm` 仅仅只是给Admin 使用，那么最简单的解决方法就是忽略`Meta.model` 属性，因为`ModelAdmin` 将自动选择应该使用的模型。或者，你也可以设置在 `Meta` 类中的 `fields = []`  来满足 `ModelForm` 的合法性。

提示

如果 `ModelForm` 和 `ModelAdmin` 同时定义了一个 `exclude` 选项，那么 `ModelAdmin` 具有更高的优先级：

```
from django import forms
from django.contrib import admin
from myapp.models import Person

class PersonForm(forms.ModelForm):

    class Meta:
        model = Person
        exclude = ['name']

class PersonAdmin(admin.ModelAdmin):
    exclude = ['age']
    form = PersonForm

```

在上例中， “age” 字段将被排除而 “name” 字段将被包含在最终产生的表单中。

`ModelAdmin.``formfield_overrides`

这个属性通过一种临时的方案来覆盖现有的模型中[`Field`](../../forms/fields.html#django.forms.Field "django.forms.Field") （字段）类型在admin site中的显示类型。`formfield_overrides` 在类初始化的时候通过一个字典类型的变量来对应模型字段类型与实际重载类型的关系。

因为概念有点抽象，所以让我们来举一个具体的例子。`formfield_overrides` 常被用于让一个已有的字段显示为自定义控件。所以，试想一下我们写了一个 `RichTextEditorWidget（富文本控件）` 然后我们想用它来代替`&lt;textarea&gt;（文本域控件）`用于输入大段文字。下面就是我们如何做到这样的替换。

```
from django.db import models
from django.contrib import admin

# Import our custom widget and our model from where they're defined
from myapp.widgets import RichTextEditorWidget
from myapp.models import MyModel

class MyModelAdmin(admin.ModelAdmin):
    formfield_overrides = {
        models.TextField: {'widget': RichTextEditorWidget},
    }

```

注意字典的键是一个实际的字段类型，而_不是_一个具体的字符。  字典的值是另外一个字典结构的数据；这个参数会传递到表单字段 `__init__()（初始化方法）` 中。有关详细信息，请参见[_The Forms API_](../../forms/api.html)。

警告

如果你想用一个关系字段的自定义界面 (即 [`ForeignKey`](../../models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey") 或者 [`ManyToManyField`](../../models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField"))， 确保你没有在`raw_id_fields` or `radio_fields`中included那个字段名。

`formfield_overrides`不会让您更改`raw_id_fields`或`radio_fields`设置的关系字段上的窗口小部件。这是因为`raw_id_fields`和`radio_fields`暗示自己的自定义小部件。

`ModelAdmin.``inlines`

请参见下面的[`InlineModelAdmin`](#django.contrib.admin.InlineModelAdmin "django.contrib.admin.InlineModelAdmin")对象以及[`ModelAdmin.get_formsets_with_inlines()`](#django.contrib.admin.ModelAdmin.get_formsets_with_inlines "django.contrib.admin.ModelAdmin.get_formsets_with_inlines")。

`ModelAdmin.``list_display`

使用`list_display` 去控制哪些字段会显示在Admin 的修改列表页面中。

示例︰

```
list_display = ('first_name', 'last_name')

```

如果你没有设置`list_display`，Admin 站点将只显示一列表示每个对象的`__str__()` （Python 2 中是`__unicode__()`）。

在`list_display`中，你有4种赋值方式可以使用：

*   模型的字段。 例如:

    ```
    class PersonAdmin(admin.ModelAdmin):
        list_display = ('first_name', 'last_name')

    ```

*   一个接受对象实例作为参数的可调用对象。例子：

    ```
    def upper_case_name(obj):
        return ("%s  %s" % (obj.first_name, obj.last_name)).upper()
    upper_case_name.short_description = 'Name'

    class PersonAdmin(admin.ModelAdmin):
        list_display = (upper_case_name,)

    ```

*   一个表示`ModelAdmin` 中某个属性的字符串。行为与可调用对象相同。 例如︰

    ```
    class PersonAdmin(admin.ModelAdmin):
        list_display = ('upper_case_name',)

        def upper_case_name(self, obj):
            return ("%s  %s" % (obj.first_name, obj.last_name)).upper()
        upper_case_name.short_description = 'Name'

    ```

*   表示模型中某个属性的字符串。它的行为与可调用对象几乎相同，但这时的`self` 是模型实例。这里是一个完整的模型示例︰

    ```
    from django.db import models
    from django.contrib import admin

    class Person(models.Model):
        name = models.CharField(max_length=50)
        birthday = models.DateField()

        def decade_born_in(self):
            return self.birthday.strftime('%Y')[:3] + "0's"
        decade_born_in.short_description = 'Birth decade'

    class PersonAdmin(admin.ModelAdmin):
        list_display = ('name', 'decade_born_in')

    ```

关于`list_display` 要注意的几个特殊情况︰

*   如果字段是一个`ForeignKey`，Django 将展示相关对象的`__str__()` （Python 2 上是`__unicode__()`）。

*   不支持`ManyToManyField` 字段， 因为这将意味着对表中的每一行执行单独的SQL 语句。如果尽管如此你仍然想要这样做，请给你的模型一个自定义的方法，并将该方法名称添加到 `list_display`。（`list_display` 的更多自定义方法请参见下文）。

*   如果该字段为`BooleanField` 或`NullBooleanField`，Django 会显示漂亮的"on"或"off"图标而不是`True` 或`False`。

*   如果给出的字符串是模型、`ModelAdmin` 的一个方法或可调用对象，Django 将默认转义HTML输出。如果你不希望转义方法的输出，可以给方法一个`allow_tags` 属性，其值为`True`。然而，为了避免XSS 漏洞，应该使用[`format_html()`](../../utils.html#django.utils.html.format_html "django.utils.html.format_html") 转义用户提供的输入。

    下面是一个完整的示例模型︰

    ```
    from django.db import models
    from django.contrib import admin
    from django.utils.html import format_html

    class Person(models.Model):
        first_name = models.CharField(max_length=50)
        last_name = models.CharField(max_length=50)
        color_code = models.CharField(max_length=6)

        def colored_name(self):
            return format_html('&lt;span style="color: #{};"&gt;{} {}&lt;/span&gt;',
                               self.color_code,
                               self.first_name,
                               self.last_name)

        colored_name.allow_tags = True

    class PersonAdmin(admin.ModelAdmin):
        list_display = ('first_name', 'last_name', 'colored_name')

    ```

*   如果给出的字符串是模型、`ModelAdmin` 的一个方法或一个返回 True 或False 的可调用的方法，然后赋值给方法的`boolean` 属性一个`True`值， Django 将显示漂亮的"on"或"off"图标，。

    下面是一个完整的示例模型︰

    ```
    from django.db import models
    from django.contrib import admin

    class Person(models.Model):
        first_name = models.CharField(max_length=50)
        birthday = models.DateField()

        def born_in_fifties(self):
            return self.birthday.strftime('%Y')[:3] == '195'
        born_in_fifties.boolean = True

    class PersonAdmin(admin.ModelAdmin):
        list_display = ('name', 'born_in_fifties')

    ```

*   `__str__()`（Python 2 上是`__unicode__()`）方法在`list_display` 中同样合法，就和任何其他模型方法一样，所以下面这样写完全OK︰

    ```
    list_display = ('__str__', 'some_other_field')

    ```

*   通常情况下，`list_display` 的元素如果不是实际的数据库字段不能用于排序（因为 Django 所有的排序都在数据库级别）。

    然而，如果`list_display` 元素表示数据库的一个特定字段，你可以通过设置 元素的`admin_order_field` 属性表示这一事实。

    例如︰

    ```
    from django.db import models
    from django.contrib import admin
    from django.utils.html import format_html

    class Person(models.Model):
        first_name = models.CharField(max_length=50)
        color_code = models.CharField(max_length=6)

        def colored_first_name(self):
            return format_html('&lt;span style="color: #{};"&gt;{}&lt;/span&gt;',
                               self.color_code,
                               self.first_name)

        colored_first_name.allow_tags = True
        colored_first_name.admin_order_field = 'first_name'

    class PersonAdmin(admin.ModelAdmin):
        list_display = ('first_name', 'colored_first_name')

    ```

    上面的示例告诉Django 在Admin 中按照按`colored_first_name` 排序时依据`first_name` 字段。

    New in Django 1.7.

    要表示按照`admin_order_field` 降序排序，你可以在该字段名称前面使用一个连字符前缀。使用上面的示例，这会看起来像︰

    ```
    colored_first_name.admin_order_field = '-first_name'

    ```

*   `list_display` 的元素也可以是属性。不过请注意，由于方式属性在Python 中的工作方式，在属性上设置`short_description` 只能使用 `property()` 函数，**不** 能使用`@property` 装饰器。

    例如：

    ```
    class Person(models.Model):
        first_name = models.CharField(max_length=50)
        last_name = models.CharField(max_length=50)

        def my_property(self):
            return self.first_name + ' ' + self.last_name
        my_property.short_description = "Full name of the person"

        full_name = property(my_property)

    class PersonAdmin(admin.ModelAdmin):
        list_display = ('full_name',)

    ```

*   `list_display` 中的字段名称还将作为HTML 输出的CSS 类， 形式为每个`&lt;th&gt;` 元素上具有`column-&lt;field_name&gt;`。例如这可以用于在CSS 文件中设置列的宽度。

*   Django 会尝试以下面的顺序解释`list_display` 的每个元素︰

    *   模型的字段。
    *   可调用对象。
    *   表示`ModelAdmin` 属性的字符串。
    *   表示模型属性的字符串。

    例如，如果`first_name` 既是模型的一个字段又是`ModelAdmin` 的一个属性，使用的将是模型字段。

`ModelAdmin.``list_display_links`

使用`list_display_links`可以控制[`list_display`](#django.contrib.admin.ModelAdmin.list_display "django.contrib.admin.ModelAdmin.list_display")中的字段是否应该链接到对象的“更改”页面。

默认情况下，更改列表页将链接第一列 - `list_display`中指定的第一个字段 - 到每个项目的更改页面。但是`list_display_links`可让您更改此设置：

*   将其设置为`None`，根本不会获得任何链接。

*   将其设置为要将其列转换为链接的字段列表或元组（格式与`list_display`相同）。

    您可以指定一个或多个字段。只要这些字段出现在`list_display`中，Django不会关心多少（或多少）字段被链接。唯一的要求是，如果要以这种方式使用`list_display_links`，则必须定义`list_display`。

在此示例中，`first_name`和`last_name`字段将链接到更改列表页面上：

```
class PersonAdmin(admin.ModelAdmin):
    list_display = ('first_name', 'last_name', 'birthday')
    list_display_links = ('first_name', 'last_name')

```

在此示例中，更改列表页面网格将没有链接：

```
class AuditEntryAdmin(admin.ModelAdmin):
    list_display = ('timestamp', 'message')
    list_display_links = None

```

Changed in Django 1.7:

`None`作为有效的`list_display_links`值添加。

`ModelAdmin.``list_editable`

将`list_editable`设置为模型上的字段名称列表，这将允许在更改列表页面上进行编辑。也就是说，`list_editable`中列出的字段将在更改列表页面上显示为表单小部件，允许用户一次编辑和保存多行。

注意

`list_editable`以特定方式与其他几个选项进行交互；您应该注意以下规则：

*   `list_editable`中的任何字段也必须位于`list_display`中。您无法编辑未显示的字段！
*   同一字段不能在`list_editable`和`list_display_links`中列出 - 字段不能同时是表单和链接。

如果这些规则中的任一个损坏，您将收到验证错误。

`ModelAdmin.``list_filter`

`list_filter` 设置激活激活Admin 修改列表页面右侧栏中的过滤器，如下面的屏幕快照所示︰

![../../../_images/users_changelist.png](../../../_images/users_changelist.png)

`list_filter` 应该是一个列表或元组，其每个元素应该是下面类型中的一种：

*   字段名称，其指定的字段应该是`BooleanField`、`CharField`、`DateField`、`DateTimeField`、`IntegerField`、`ForeignKey` 或`ManyToManyField`，例如︰

    ```
    class PersonAdmin(admin.ModelAdmin):
        list_filter = ('is_staff', 'company')

    ```

    `list_filter` 中的字段名称也可以使用`__` 查找跨关联关系，例如︰

    ```
    class PersonAdmin(admin.UserAdmin):
        list_filter = ('company__name',)

    ```

*   一个继承自`django.contrib.admin.SimpleListFilter` 的类，你需要给它提供`title` 和 `parameter_name` 属性来重写`lookups` 和`queryset` 方法，例如︰

    ```
    from datetime import date

    from django.contrib import admin
    from django.utils.translation import ugettext_lazy as _

    class DecadeBornListFilter(admin.SimpleListFilter):
        # Human-readable title which will be displayed in the
        # right admin sidebar just above the filter options.
        title = _('decade born')

        # Parameter for the filter that will be used in the URL query.
        parameter_name = 'decade'

        def lookups(self, request, model_admin):
            """
     Returns a list of tuples. The first element in each
     tuple is the coded value for the option that will
     appear in the URL query. The second element is the
     human-readable name for the option that will appear
     in the right sidebar.
     """
            return (
                ('80s', _('in the eighties')),
                ('90s', _('in the nineties')),
            )

        def queryset(self, request, queryset):
            """
     Returns the filtered queryset based on the value
     provided in the query string and retrievable via
     `self.value()`.
     """
            # Compare the requested value (either '80s' or '90s')
            # to decide how to filter the queryset.
            if self.value() == '80s':
                return queryset.filter(birthday__gte=date(1980, 1, 1),
                                        birthday__lte=date(1989, 12, 31))
            if self.value() == '90s':
                return queryset.filter(birthday__gte=date(1990, 1, 1),
                                        birthday__lte=date(1999, 12, 31))

    class PersonAdmin(admin.ModelAdmin):
        list_filter = (DecadeBornListFilter,)

    ```

    注

    作为一种方便，`HttpRequest` 对象将传递给`lookups` 和`queryset` 方法，例如︰

    ```
    class AuthDecadeBornListFilter(DecadeBornListFilter):

        def lookups(self, request, model_admin):
            if request.user.is_superuser:
                return super(AuthDecadeBornListFilter,
                    self).lookups(request, model_admin)

        def queryset(self, request, queryset):
            if request.user.is_superuser:
                return super(AuthDecadeBornListFilter,
                    self).queryset(request, queryset)

    ```

    也作为一种方便，`ModelAdmin` 对象将传递给`lookups` 方法，例如如果你想要基于现有的数据查找︰

    ```
    class AdvancedDecadeBornListFilter(DecadeBornListFilter):

        def lookups(self, request, model_admin):
            """
     Only show the lookups if there actually is
     anyone born in the corresponding decades.
     """
            qs = model_admin.get_queryset(request)
            if qs.filter(birthday__gte=date(1980, 1, 1),
                          birthday__lte=date(1989, 12, 31)).exists():
                yield ('80s', _('in the eighties'))
            if qs.filter(birthday__gte=date(1990, 1, 1),
                          birthday__lte=date(1999, 12, 31)).exists():
                yield ('90s', _('in the nineties'))

    ```

*   一个元组，第一个元素是字段名称，第二个元素是从继承自`django.contrib.admin.FieldListFilter` 的一个类，例如︰

    ```
    class PersonAdmin(admin.ModelAdmin):
        list_filter = (
            ('is_staff', admin.BooleanFieldListFilter),
        )

    ```

    New in Django 1.8.

    你可以使用`RelatedOnlyFieldListFilter` 限制与该对象关联的模型的选项：

    ```
    class BookAdmin(admin.ModelAdmin):
        list_filter = (
            ('author', admin.RelatedOnlyFieldListFilter),
        )

    ```

    假设`author` 是`User` 模型的一个`ForeignKey`，这将限制`list_filter` 的选项为编写过书籍的用户，而不是所有用户。

    注意

    `FieldListFilter` API 被视为内部的，可能会改变。

也可以指定自定义模板用于渲染列表筛选器︰

```
class FilterWithCustomTemplate(admin.SimpleListFilter):
    template = "custom_template.html"

```

具体的例子请参见Django 提供的默认模板（`admin/filter.html`）。

`ModelAdmin.``list_max_show_all`

设置`list_max_show_all`以控制在“显示所有”管理更改列表页面上可以显示的项目数。只有当总结果计数小于或等于此设置时，管理员才会在更改列表上显示“显示全部”链接。默认情况下，设置为`200`。

`ModelAdmin.``list_per_page`

`list_per_page` 设置控制Admin 修改列表页面每页中显示多少项。默认设置为`100`。

`ModelAdmin.``list_select_related`

设置`list_select_related`以告诉Django在检索管理更改列表页面上的对象列表时使用[`select_related()`](../../models/querysets.html#django.db.models.query.QuerySet.select_related "django.db.models.query.QuerySet.select_related")。这可以节省大量的数据库查询。

该值应该是布尔值，列表或元组。默认值为`False`。

当值为`True`时，将始终调用`select_related()`。When value is set to `False`, Django will look at `list_display` and call `select_related()` if any `ForeignKey` is present.

如果您需要更细粒度的控制，请使用元组（或列表）作为`list_select_related`的值。空元组将阻止Django调用`select_related`。任何其他元组将直接传递到`select_related`作为参数。例如：

```
class ArticleAdmin(admin.ModelAdmin):
    list_select_related = ('author', 'category')

```

将会调用`select_related('author', 'category')`.

`ModelAdmin.``ordering`

设置`ordering`以指定如何在Django管理视图中对对象列表进行排序。这应该是与模型的[`ordering`](../../models/options.html#django.db.models.Options.ordering "django.db.models.Options.ordering")参数格式相同的列表或元组。

如果没有提供，Django管理员将使用模型的默认排序。

如果您需要指定动态顺序（例如，根据用户或语言），您可以实施[`get_ordering()`](#django.contrib.admin.ModelAdmin.get_ordering "django.contrib.admin.ModelAdmin.get_ordering")方法。

`ModelAdmin.``paginator`

paginator类用于分页。默认情况下，使用[`django.core.paginator.Paginator`](../../../topics/pagination.html#django.core.paginator.Paginator "django.core.paginator.Paginator")。如果自定义paginator类没有与[`django.core.paginator.Paginator`](../../../topics/pagination.html#django.core.paginator.Paginator "django.core.paginator.Paginator")相同的构造函数接口，则还需要为[`ModelAdmin.get_paginator()`](#django.contrib.admin.ModelAdmin.get_paginator "django.contrib.admin.ModelAdmin.get_paginator") 。

`ModelAdmin.``prepopulated_fields`

将`prepopulated_fields`设置为将字段名称映射到其应预先填充的字段的字典：

```
class ArticleAdmin(admin.ModelAdmin):
    prepopulated_fields = {"slug": ("title",)}

```

设置时，给定字段将使用一些JavaScript来从分配的字段填充。此功能的主要用途是自动从一个或多个其他字段生成`SlugField`字段的值。生成的值是通过连接源字段的值，然后将该结果转换为有效的字节（例如用空格替换破折号）来生成的。

`prepopulated_fields`不接受`DateTimeField`，`ForeignKey`或`ManyToManyField`字段。

`ModelAdmin.``preserve_filters`

管理员现在在创建，编辑或删除对象后保留列表视图中的过滤器。您可以将此属性设置为`False`，以恢复之前清除过滤器的行为。

`ModelAdmin.``radio_fields`

By default, Django’s admin uses a select-box interface (&lt;select&gt;) for fields that are `ForeignKey` or have `choices` set. 如果`radio_fields`中存在字段，Django将使用单选按钮接口。假设`group`是`Person`模型上的`ForeignKey`

```
class PersonAdmin(admin.ModelAdmin):
    radio_fields = {"group": admin.VERTICAL}

```

您可以选择使用`django.contrib.admin`模块中的`HORIZONTAL`或`VERTICAL`。

除非是`ForeignKey`或设置了`choices`，否则不要在`radio_fields`中包含字段。

`ModelAdmin.``raw_id_fields`

默认情况下，Django 的Admin 对`ForeignKey` 字段使用选择框表示 (&lt;select&gt;) 。有时候你不想在下拉菜单中显示所有相关实例产生的开销。

`raw_id_fields` 是一个字段列表，你希望将`ForeignKey` 或`ManyToManyField` 转换成`Input` Widget：

```
class ArticleAdmin(admin.ModelAdmin):
    raw_id_fields = ("newspaper",)

```

如果该字段是一个`ForeignKey`，`raw_id_fields` `Input` Widget 应该包含一个外键，或者如果字段是一个`ManyToManyField` 则应该是一个逗号分隔的值的列表。 `raw_id_fields` Widget 在字段旁边显示一个放大镜按钮，允许用户搜索并选择一个值︰

![../../../_images/raw_id_fields.png](../../../_images/raw_id_fields.png)

`ModelAdmin.``readonly_fields`

默认情况下，管理员将所有字段显示为可编辑。此选项中的任何字段（应为`list`或`tuple`）将按原样显示其数据，且不可编辑；它们也会从用于创建和编辑的[`ModelForm`](../../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")中排除。请注意，指定[`ModelAdmin.fields`](#django.contrib.admin.ModelAdmin.fields "django.contrib.admin.ModelAdmin.fields")或[`ModelAdmin.fieldsets`](#django.contrib.admin.ModelAdmin.fieldsets "django.contrib.admin.ModelAdmin.fieldsets")时，只读字段必须显示才能显示（否则将被忽略）。

如果在未通过[`ModelAdmin.fields`](#django.contrib.admin.ModelAdmin.fields "django.contrib.admin.ModelAdmin.fields")或[`ModelAdmin.fieldsets`](#django.contrib.admin.ModelAdmin.fieldsets "django.contrib.admin.ModelAdmin.fieldsets")定义显式排序的情况下使用`readonly_fields`，则它们将在所有可编辑字段之后添加。

只读字段不仅可以显示模型字段中的数据，还可以显示模型方法的输出或`ModelAdmin`类本身的方法。这与[`ModelAdmin.list_display`](#django.contrib.admin.ModelAdmin.list_display "django.contrib.admin.ModelAdmin.list_display")的行为非常相似。这提供了一种使用管理界面提供对正在编辑的对象的状态的反馈的简单方法，例如：

```
from django.contrib import admin
from django.utils.html import format_html_join
from django.utils.safestring import mark_safe

class PersonAdmin(admin.ModelAdmin):
    readonly_fields = ('address_report',)

    def address_report(self, instance):
        # assuming get_full_address() returns a list of strings
        # for each line of the address and you want to separate each
        # line by a linebreak
        return format_html_join(
            mark_safe('&lt;br/&gt;'),
            '{}',
            ((line,) for line in instance.get_full_address()),
        ) or "&lt;span class='errors'&gt;I can't determine this address.&lt;/span&gt;"

    # short_description functions like a model field's verbose_name
    address_report.short_description = "Address"
    # in this example, we have used HTML tags in the output
    address_report.allow_tags = True

```

`ModelAdmin.``save_as`

`save_as` 设置启用Admin 更改表单上的“save as”功能。

通常情况下，对象有三个保存选项："保存"、"保存并继续编辑"和"保存并添加另一个"。如果`save_as` 为`True`，"保存并添加另一个"将由"另存为"按钮取代。

"另存为"表示对象将被保存为一个新的对象 （带有一个新的 ID)，而不是旧的对象。

默认情况下，`save_as` 设置为`False`。

`ModelAdmin.``save_on_top`

设置`save_on_top`可在表单顶部添加保存按钮。

通常，保存按钮仅出现在表单的底部。如果您设置`save_on_top`，则按钮将同时显示在顶部和底部。

默认情况下，`save_on_top`设置为`False`。

`ModelAdmin.``search_fields`

`search_fields` 设置启用Admin 更改列表页面上的搜索框。此属性应设置为每当有人在该文本框中提交搜索查询将搜索的字段名称的列表。

这些字段应该是某种文本字段，如`CharField` 或`TextField`。你还可以通过查询API 的"跟随"符号进行`ForeignKey` 或`ManyToManyField` 上的关联查找：

```
search_fields = ['foreign_key__related_fieldname']

```

例如，如果你有一个具有作者的博客，下面的定义将启用通过作者的电子邮件地址搜索博客条目︰

```
search_fields = ['user__email']

```

如果有人在Admin 搜索框中进行搜索，Django 拆分搜索查询为单词并返回包含每个单词的所有对象，不区分大小写，其中每个单词必须在至少一个`search_fields`。例如，如果`search_fields` 设置为`['first_name', 'last_name']`，用户搜索`john lennon`，Django 的行为将相当于下面的这个`WHERE` SQL 子句︰

```
WHERE (first_name ILIKE '%john%' OR last_name ILIKE '%john%')
AND (first_name ILIKE '%lennon%' OR last_name ILIKE '%lennon%')

```

若要更快和/或更严格的搜索，请在字典名称前面加上前缀︰

`^`

匹配字段的开始。例如，如果`search_fields` 设置为`['^first_name', '^last_name']`，用户搜索`john lennon` 时，Django 的行为将等同于下面这个`WHERE` SQL 字句：

```
WHERE (first_name ILIKE 'john%' OR last_name ILIKE 'john%')
AND (first_name ILIKE 'lennon%' OR last_name ILIKE 'lennon%')

```

此查询比正常`'%john%'` 查询效率高，因为数据库只需要检查某一列数据的开始，而不用寻找整列数据。另外，如果列上有索引，有些数据库可能能够对于此查询使用索引，即使它是`like` 查询。

`=`

精确匹配，不区分大小写。例如，如果`search_fields` 设置为`['=first_name', '=last_name']`，用户搜索`john lennon` 时，Django 的行为将等同于下面这个`WHERE` SQL 字句：

```
WHERE (first_name ILIKE 'john' OR last_name ILIKE 'john')
AND (first_name ILIKE 'lennon' OR last_name ILIKE 'lennon')

```

注意，该查询输入通过空格分隔，所以根据这个示例，目前不能够搜索`first_name` 精确匹配`'john winston'`（包含空格）的所有记录。

`@`

Performs a full-text match. This is like the default search method but uses an index. Currently this is only available for MySQL.

如果你需要自定义搜索，你可以使用[`ModelAdmin.get_search_results()`](#django.contrib.admin.ModelAdmin.get_search_results "django.contrib.admin.ModelAdmin.get_search_results") 来提供附件的或另外一种搜索行为。

`ModelAdmin.``show_full_result_count`

New in Django 1.8.

设置`show_full_result_count`以控制是否应在过滤的管理页面上显示对象的完整计数（例如`99 结果 103 total）`）。如果此选项设置为`False`，则像`99 结果 （显示 ）`。

默认情况下，`show_full_result_count=True`生成一个查询，对表执行完全计数，如果表包含大量行，这可能很昂贵。

`ModelAdmin.``view_on_site`

New in Django 1.7.

设置`view_on_site`以控制是否显示“在网站上查看”链接。此链接将带您到一个URL，您可以在其中显示已保存的对象。

此值可以是布尔标志或可调用的。如果`True`（默认值），对象的[`get_absolute_url()`](../../models/instances.html#django.db.models.Model.get_absolute_url "django.db.models.Model.get_absolute_url")方法将用于生成网址。

如果您的模型有[`get_absolute_url()`](../../models/instances.html#django.db.models.Model.get_absolute_url "django.db.models.Model.get_absolute_url")方法，但您不想显示“在网站上查看”按钮，则只需将`view_on_site`设置为`False`：

```
from django.contrib import admin

class PersonAdmin(admin.ModelAdmin):
    view_on_site = False

```

如果它是可调用的，它接受模型实例作为参数。例如：

```
from django.contrib import admin
from django.core.urlresolvers import reverse

class PersonAdmin(admin.ModelAdmin):
    def view_on_site(self, obj):
        return 'http://example.com' + reverse('person-detail',
                                              kwargs={'slug': obj.slug})

```

#### 自定义模板的选项

[_重写Admin模板_](#admin-overriding-templates) 一节描述如何重写或扩展默认Admin 模板。使用以下选项来重写[`ModelAdmin`](#django.contrib.admin.ModelAdmin "django.contrib.admin.ModelAdmin") 视图使用的默认模板︰

`ModelAdmin.``add_form_template`

[`add_view()`](#django.contrib.admin.ModelAdmin.add_view "django.contrib.admin.ModelAdmin.add_view") 使用的自定义模板的路径。

`ModelAdmin.``change_form_template`

[`change_view()`](#django.contrib.admin.ModelAdmin.change_view "django.contrib.admin.ModelAdmin.change_view") 使用的自定义模板的路径。

`ModelAdmin.``change_list_template`

[`changelist_view()`](#django.contrib.admin.ModelAdmin.changelist_view "django.contrib.admin.ModelAdmin.changelist_view") 使用的自定义模板的路径。

`ModelAdmin.``delete_confirmation_template`

[`delete_view()`](#django.contrib.admin.ModelAdmin.delete_view "django.contrib.admin.ModelAdmin.delete_view") 使用的自定义模板，用于删除一个或多个对象时显示一个确认页。

`ModelAdmin.``delete_selected_confirmation_template`

`delete_selected()` 使用的自定义模板，用于删除一个或多个对象时显示一个确认页。参见[_Action 的文档_](actions.html)。

`ModelAdmin.``object_history_template`

[`history_view()`](#django.contrib.admin.ModelAdmin.history_view "django.contrib.admin.ModelAdmin.history_view") 使用的自定义模板的路径。

### ModelAdmin methods

警告

[`ModelAdmin.save_model()`](#django.contrib.admin.ModelAdmin.save_model "django.contrib.admin.ModelAdmin.save_model") 以及 [`ModelAdmin.delete_model()`](#django.contrib.admin.ModelAdmin.delete_model "django.contrib.admin.ModelAdmin.delete_model") must save/delete the object, they are not for veto purposes, rather they allow you to perform extra operations.

`ModelAdmin.``save_model`(_request_, _obj_, _form_, _change_)

`save_model`方法被赋予`HttpRequest`，模型实例，`ModelForm`实例和布尔值，基于它是添加还是更改对象。在这里您可以执行任何预保存或后保存操作。

例如，在保存之前将`request.user`附加到对象：

```
from django.contrib import admin

class ArticleAdmin(admin.ModelAdmin):
    def save_model(self, request, obj, form, change):
        obj.user = request.user
        obj.save()

```

`ModelAdmin.``delete_model`(_request_, _obj_)

`delete_model`方法给出了`HttpRequest`和模型实例。使用此方法执行预删除或后删除操作。

`ModelAdmin.``save_formset`(_request_, _form_, _formset_, _change_)

`save_formset`方法是给予`HttpRequest`，父`ModelForm`实例和基于是否添加或更改父对象的布尔值。

例如，要将`request.user`附加到每个已更改的formset模型实例：

```
class ArticleAdmin(admin.ModelAdmin):
    def save_formset(self, request, form, formset, change):
        instances = formset.save(commit=False)
        for obj in formset.deleted_objects:
            obj.delete()
        for instance in instances:
            instance.user = request.user
            instance.save()
        formset.save_m2m()

```

另请参见[_Saving objects in the formset_](../../../topics/forms/modelforms.html#saving-objects-in-the-formset)。

`ModelAdmin.``get_ordering`(_request_)

The `get_ordering` method takes a``request`` as parameter and is expected to return a `list` or `tuple` for ordering similar to the [`ordering`](#django.contrib.admin.ModelAdmin.ordering "django.contrib.admin.ModelAdmin.ordering") attribute. 例如：

```
class PersonAdmin(admin.ModelAdmin):

    def get_ordering(self, request):
        if request.user.is_superuser:
            return ['name', 'rank']
        else:
            return ['name']

```

`ModelAdmin.``get_search_results`(_request_, _queryset_, _search_term_)

`get_search_results`方法将显示的对象列表修改为与提供的搜索项匹配的对象。它接受请求，应用当前过滤器的查询集以及用户提供的搜索项。它返回一个包含被修改以实现搜索的查询集的元组，以及一个指示结果是否可能包含重复项的布尔值。

默认实现搜索在[`ModelAdmin.search_fields`](#django.contrib.admin.ModelAdmin.search_fields "django.contrib.admin.ModelAdmin.search_fields")中命名的字段。

此方法可以用您自己的自定义搜索方法覆盖。例如，您可能希望通过整数字段搜索，或使用外部工具（如Solr或Haystack）。您必须确定通过搜索方法实现的查询集更改是否可能在结果中引入重复项，并在返回值的第二个元素中返回`True`。

例如，要启用按整数字段搜索，您可以使用：

```
class PersonAdmin(admin.ModelAdmin):
    list_display = ('name', 'age')
    search_fields = ('name',)

    def get_search_results(self, request, queryset, search_term):
        queryset, use_distinct = super(PersonAdmin, self).get_search_results(request, queryset, search_term)
        try:
            search_term_as_int = int(search_term)
        except ValueError:
            pass
        else:
            queryset |= self.model.objects.filter(age=search_term_as_int)
        return queryset, use_distinct

```

`ModelAdmin.``save_related`(_request_, _form_, _formsets_, _change_)

`save_related`方法给出了`HttpRequest`，父`ModelForm`实例，内联表单列表和一个布尔值，添加或更改。在这里，您可以对与父级相关的对象执行任何预保存或后保存操作。请注意，此时父对象及其形式已保存。

`ModelAdmin.``get_readonly_fields`(_request_, _obj=None_)

`get_readonly_fields`方法在添加表单上给予`HttpRequest`和`obj`（或`None`），希望返回将以只读形式显示的字段名称的`list`或`tuple`，如上面在[`ModelAdmin.readonly_fields`](#django.contrib.admin.ModelAdmin.readonly_fields "django.contrib.admin.ModelAdmin.readonly_fields")部分中所述。

`ModelAdmin.``get_prepopulated_fields`(_request_, _obj=None_)

`get_prepopulated_fields`方法在添加表单上给予`HttpRequest`和`obj`（或`None`），预期返回`dictionary`，如上面在[`ModelAdmin.prepopulated_fields`](#django.contrib.admin.ModelAdmin.prepopulated_fields "django.contrib.admin.ModelAdmin.prepopulated_fields")部分中所述。

`ModelAdmin.``get_list_display`(_request_)

`get_list_display`方法被赋予`HttpRequest`，并且希望返回字段名称的`list`或`tuple`显示在如上所述的[`ModelAdmin.list_display`](#django.contrib.admin.ModelAdmin.list_display "django.contrib.admin.ModelAdmin.list_display")部分中的changelist视图上。

`ModelAdmin.``get_list_display_links`(_request_, _list_display_)

The `get_list_display_links` method is given the `HttpRequest` and the `list` or `tuple` returned by [`ModelAdmin.get_list_display()`](#django.contrib.admin.ModelAdmin.get_list_display "django.contrib.admin.ModelAdmin.get_list_display"). 预期将返回更改列表上将链接到更改视图的字段名称的`None`或`list`或`tuple`，如上所述在[`ModelAdmin.list_display_links`](#django.contrib.admin.ModelAdmin.list_display_links "django.contrib.admin.ModelAdmin.list_display_links")部分中。

Changed in Django 1.7:

`None`作为有效的`get_list_display_links()`返回值添加。

`ModelAdmin.``get_fields`(_request_, _obj=None_)

New in Django 1.7.

`get_fields`方法被赋予`HttpRequest`和`obj`被编辑（或在添加表单上`None`），希望返回字段列表，如上面在[`ModelAdmin.fields`](#django.contrib.admin.ModelAdmin.fields "django.contrib.admin.ModelAdmin.fields")部分中所述。

`ModelAdmin.``get_fieldsets`(_request_, _obj=None_)

`get_fieldsets`方法是在添加表单上给予`HttpRequest`和`obj`（或`None`），期望返回二元组列表，其中每个二元组在管理表单页面上表示`&lt;fieldset&gt;`，如上面在[`ModelAdmin.fieldsets`](#django.contrib.admin.ModelAdmin.fieldsets "django.contrib.admin.ModelAdmin.fieldsets")部分。

`ModelAdmin.``get_list_filter`(_request_)

`get_list_filter`方法被赋予`HttpRequest`，并且期望返回与[`list_filter`](#django.contrib.admin.ModelAdmin.list_filter "django.contrib.admin.ModelAdmin.list_filter")属性相同类型的序列类型。

`ModelAdmin.``get_search_fields`(_request_)

New in Django 1.7.

`get_search_fields`方法被赋予`HttpRequest`，并且期望返回与[`search_fields`](#django.contrib.admin.ModelAdmin.search_fields "django.contrib.admin.ModelAdmin.search_fields")属性相同类型的序列类型。

`ModelAdmin.``get_inline_instances`(_request_, _obj=None_)

`get_inline_instances`方法在添加表单上给予`HttpRequest`和`obj`（或`None`），预期会返回`list`或`tuple`的[`InlineModelAdmin`](#django.contrib.admin.InlineModelAdmin "django.contrib.admin.InlineModelAdmin")对象，如下面的[`InlineModelAdmin`](#django.contrib.admin.InlineModelAdmin "django.contrib.admin.InlineModelAdmin")部分所述。例如，以下内容将返回内联，而不进行基于添加，更改和删除权限的默认过滤：

```
class MyModelAdmin(admin.ModelAdmin):
    inlines = (MyInline,)

    def get_inline_instances(self, request, obj=None):
        return [inline(self.model, self.admin_site) for inline in self.inlines]

```

如果覆盖此方法，请确保返回的内联是[`inlines`](#django.contrib.admin.ModelAdmin.inlines "django.contrib.admin.ModelAdmin.inlines")中定义的类的实例，或者在添加相关对象时可能会遇到“错误请求”错误。

`ModelAdmin.``get_urls`()

`ModelAdmin` 的`get_urls` 方法返回ModelAdmin 将要用到的URLs，方式与URLconf 相同。因此，你可以用[_URL 调度器_](../../../topics/http/urls.html) 中所述的方式扩展它们︰

```
class MyModelAdmin(admin.ModelAdmin):
    def get_urls(self):
        urls = super(MyModelAdmin, self).get_urls()
        my_urls = [
            url(r'^my_view/$', self.my_view),
        ]
        return my_urls + urls

    def my_view(self, request):
        # ...
        context = dict(
           # Include common variables for rendering the admin template.
           self.admin_site.each_context(request),
           # Anything else you want in the context...
           key=value,
        )
        return TemplateResponse(request, "sometemplate.html", context)

```

如果你想要使用Admin 的布局，可以从`admin/base_site.html` 扩展︰

```
{% extends "admin/base_site.html" %}
{% block content %}
...
{% endblock %}

```

注

请注意，自定义的模式包含在正常的Admin URLs_之前_：Admin URL 模式非常宽松，将匹配几乎任何内容，因此你通常要追加自定义的URLs 到内置的URLs 前面。

在此示例中，`my_view` 的访问点将是`/admin/myapp/mymodel/my_view/`（假设Admin URLs 包含在`/admin/` 下）。

但是,  上述定义的函数`self.my_view`  将遇到两个问题：

*   它_不_ 执行任何权限检查，所以会向一般公众开放。
*   它_不_提供任何HTTP头的详细信息以防止缓存。这意味着，如果页面从数据库检索数据，而且缓存中间件处于活动状态，页面可能显示过时的信息。

因为这通常不是你想要的，Django 提供一个方便的封装函数来检查权限并标记视图为不可缓存的。这个封装函数就是`AdminSite.admin_view()`（例如位于`ModelAdmin` 实例中的`self.admin_site.admin_view`）；就像这样使用它︰

```
class MyModelAdmin(admin.ModelAdmin):
    def get_urls(self):
        urls = super(MyModelAdmin, self).get_urls()
        my_urls = [
            url(r'^my_view/$', self.admin_site.admin_view(self.my_view))
        ]
        return my_urls + urls

```

请注意上述第5行中的被封装的视图︰

```
url(r'^my_view/$', self.admin_site.admin_view(self.my_view))

```

这个封装将保护`self.my_view` 免受未经授权的访问，并将运用`django.views.decorators.cache.never_cache` 装饰器以确保它不会被缓存，即使缓存中间件是活跃的。

如果该页面是可缓存的，但你仍然想要执行权限检查，你可以传递`AdminSite.admin_view()` 的`cacheable=True` 参数︰

```
url(r'^my_view/$', self.admin_site.admin_view(self.my_view, cacheable=True))

```

`ModelAdmin.``get_form`(_request_, _obj=None_, _**kwargs_)

返回Admin中添加和更改视图使用的[`ModelForm`](../../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm") 类，请参阅[`add_view()`](#django.contrib.admin.ModelAdmin.add_view "django.contrib.admin.ModelAdmin.add_view") 和 [`change_view()`](#django.contrib.admin.ModelAdmin.change_view "django.contrib.admin.ModelAdmin.change_view")。

其基本的实现是使用[`modelform_factory()`](../../forms/models.html#django.forms.models.modelform_factory "django.forms.models.modelform_factory") 来子类化[`form`](#django.contrib.admin.ModelAdmin.form "django.contrib.admin.ModelAdmin.form")，修改如[`fields`](#django.contrib.admin.ModelAdmin.fields "django.contrib.admin.ModelAdmin.fields") 和[`exclude`](#django.contrib.admin.ModelAdmin.exclude "django.contrib.admin.ModelAdmin.exclude")属性。所以，举个例子，如果你想要为超级用户提供额外的字段，你可以换成不同的基类表单，就像这样︰

```
class MyModelAdmin(admin.ModelAdmin):
    def get_form(self, request, obj=None, **kwargs):
        if request.user.is_superuser:
            kwargs['form'] = MySuperuserForm
        return super(MyModelAdmin, self).get_form(request, obj, **kwargs)

```

你也可以简单地直接返回一个自定义的[`ModelForm`](../../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm") 类。

`ModelAdmin.``get_formsets`(_request_, _obj=None_)

自1.7版起已弃用：请改用[`get_formsets_with_inlines()`](#django.contrib.admin.ModelAdmin.get_formsets_with_inlines "django.contrib.admin.ModelAdmin.get_formsets_with_inlines")。

产生[`InlineModelAdmin`](#django.contrib.admin.InlineModelAdmin "django.contrib.admin.InlineModelAdmin")用于管理员添加和更改视图。

例如，如果您只想在更改视图中显示特定的内联，则可以覆盖`get_formsets`，如下所示：

```
class MyModelAdmin(admin.ModelAdmin):
    inlines = [MyInline, SomeOtherInline]

    def get_formsets(self, request, obj=None):
        for inline in self.get_inline_instances(request, obj):
            # hide MyInline in the add view
            if isinstance(inline, MyInline) and obj is None:
                continue
            yield inline.get_formset(request, obj)

```

`ModelAdmin.``get_formsets_with_inlines`(_request_, _obj=None_)

New in Django 1.7.

产量（`FormSet`，[`InlineModelAdmin`](#django.contrib.admin.InlineModelAdmin "django.contrib.admin.InlineModelAdmin")）对用于管理添加和更改视图。

例如，如果您只想在更改视图中显示特定的内联，则可以覆盖`get_formsets_with_inlines`，如下所示：

```
class MyModelAdmin(admin.ModelAdmin):
    inlines = [MyInline, SomeOtherInline]

    def get_formsets_with_inlines(self, request, obj=None):
        for inline in self.get_inline_instances(request, obj):
            # hide MyInline in the add view
            if isinstance(inline, MyInline) and obj is None:
                continue
            yield inline.get_formset(request, obj), inline

```

`ModelAdmin.``formfield_for_foreignkey`(_db_field_, _request_, _**kwargs_)

`ModelAdmin`上的`formfield_for_foreignkey`方法允许覆盖外键字段的默认窗体字段。例如，要根据用户返回此外键字段的对象子集：

```
class MyModelAdmin(admin.ModelAdmin):
    def formfield_for_foreignkey(self, db_field, request, **kwargs):
        if db_field.name == "car":
            kwargs["queryset"] = Car.objects.filter(owner=request.user)
        return super(MyModelAdmin, self).formfield_for_foreignkey(db_field, request, **kwargs)

```

这使用`HttpRequest`实例过滤`Car`外键字段，只显示由`User`实例拥有的汽车。

`ModelAdmin.``formfield_for_manytomany`(_db_field_, _request_, _**kwargs_)

与`formfield_for_foreignkey`方法类似，可以覆盖`formfield_for_manytomany`方法来更改多对多字段的默认窗体字段。例如，如果所有者可以拥有多个汽车，并且汽车可以属于多个所有者 - 多对多关系，则您可以过滤`Car`外键字段，仅显示由`User`：

```
class MyModelAdmin(admin.ModelAdmin):
    def formfield_for_manytomany(self, db_field, request, **kwargs):
        if db_field.name == "cars":
            kwargs["queryset"] = Car.objects.filter(owner=request.user)
        return super(MyModelAdmin, self).formfield_for_manytomany(db_field, request, **kwargs)

```

`ModelAdmin.``formfield_for_choice_field`(_db_field_, _request_, _**kwargs_)

与`formfield_for_foreignkey`和`formfield_for_manytomany`方法类似，可以覆盖`formfield_for_choice_field`方法更改已声明选择的字段的默认窗体字段。例如，如果超级用户可用的选择应与正式工作人员可用的选项不同，则可按以下步骤操作：

```
class MyModelAdmin(admin.ModelAdmin):
    def formfield_for_choice_field(self, db_field, request, **kwargs):
        if db_field.name == "status":
            kwargs['choices'] = (
                ('accepted', 'Accepted'),
                ('denied', 'Denied'),
            )
            if request.user.is_superuser:
                kwargs['choices'] += (('ready', 'Ready for deployment'),)
        return super(MyModelAdmin, self).formfield_for_choice_field(db_field, request, **kwargs)

```

注意

在表单字段上设置的任何`choices`属性将仅限于表单字段。如果模型上的相应字段有选择集，则提供给表单的选项必须是这些选择的有效子集，否则，在保存模型本身之前验证模型本身时，表单提交将失败并显示[`ValidationError`](../../exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError") 。

`ModelAdmin.``get_changelist`(_request_, _**kwargs_)

返回要用于列表的`Changelist`类。默认情况下，使用`django.contrib.admin.views.main.ChangeList`。通过继承此类，您可以更改列表的行为。

`ModelAdmin.``get_changelist_form`(_request_, _**kwargs_)

返回[`ModelForm`](../../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")类以用于更改列表页面上的`Formset`。要使用自定义窗体，例如：

```
from django import forms

class MyForm(forms.ModelForm):
    pass

class MyModelAdmin(admin.ModelAdmin):
    def get_changelist_form(self, request, **kwargs):
        return MyForm

```

注意

如果您在[`ModelForm`](../../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")上定义`Meta.model`属性，则还必须定义`Meta.fields`属性（或`Meta.exclude`属性）。但是，`ModelAdmin`会忽略此值，并使用[`ModelAdmin.list_editable`](#django.contrib.admin.ModelAdmin.list_editable "django.contrib.admin.ModelAdmin.list_editable")属性覆盖该值。最简单的解决方案是省略`Meta.model`属性，因为`ModelAdmin`将提供要使用的正确模型。

`ModelAdmin.``get_changelist_formset`(_request_, _**kwargs_)

如果使用[`list_editable`](#django.contrib.admin.ModelAdmin.list_editable "django.contrib.admin.ModelAdmin.list_editable")，则返回[_ModelFormSet_](../../../topics/forms/modelforms.html#model-formsets)类以在更改列表页上使用。要使用自定义表单集，例如：

```
from django.forms.models import BaseModelFormSet

class MyAdminFormSet(BaseModelFormSet):
    pass

class MyModelAdmin(admin.ModelAdmin):
    def get_changelist_formset(self, request, **kwargs):
        kwargs['formset'] = MyAdminFormSet
        return super(MyModelAdmin, self).get_changelist_formset(request, **kwargs)

```

`ModelAdmin.``has_add_permission`(_request_)

如果允许添加对象，则应返回`True`，否则返回`False`。

`ModelAdmin.``has_change_permission`(_request_, _obj=None_)

如果允许编辑obj，则应返回`True`，否则返回`False`。如果obj为`None`，则应返回`True`或`False`以指示是否允许对此类对象进行编辑（例如，`False`将被解释为意味着当前用户不允许编辑此类型的任何对象）。

`ModelAdmin.``has_delete_permission`(_request_, _obj=None_)

如果允许删除obj，则应返回`True`，否则返回`False`。If obj is `None`, should return `True` or `False` to indicate whether deleting objects of this type is permitted in general (e.g., `False` will be interpreted as meaning that the current user is not permitted to delete any object of this type).

`ModelAdmin.``has_module_permission`(_request_)

New in Django 1.8.

如果在管理索引页上显示模块并允许访问模块的索引页，则应返回`True`，否则`False`。默认情况下使用[`User.has_module_perms()`](../auth.html#django.contrib.auth.models.User.has_module_perms "django.contrib.auth.models.User.has_module_perms")。覆盖它不会限制对添加，更改或删除视图的访问，[`has_add_permission()`](#django.contrib.admin.ModelAdmin.has_add_permission "django.contrib.admin.ModelAdmin.has_add_permission")，[`has_change_permission()`](#django.contrib.admin.ModelAdmin.has_change_permission "django.contrib.admin.ModelAdmin.has_change_permission")和[`has_delete_permission()`](#django.contrib.admin.ModelAdmin.has_delete_permission "django.contrib.admin.ModelAdmin.has_delete_permission")用于那。

`ModelAdmin.``get_queryset`(_request_)

`ModelAdmin`上的`get_queryset`方法会返回管理网站可以编辑的所有模型实例的[`QuerySet`](../../models/querysets.html#django.db.models.query.QuerySet "django.db.models.query.QuerySet")。覆盖此方法的一个用例是显示由登录用户拥有的对象：

```
class MyModelAdmin(admin.ModelAdmin):
    def get_queryset(self, request):
        qs = super(MyModelAdmin, self).get_queryset(request)
        if request.user.is_superuser:
            return qs
        return qs.filter(author=request.user)

```

`ModelAdmin.``message_user`(_request_, _message_, _level=messages.INFO_, _extra_tags=''_, _fail_silently=False_)

使用[`django.contrib.messages`](../messages.html#module-django.contrib.messages "django.contrib.messages: Provides cookie- and session-based temporary message storage.") 向用户发送消息。参见[_自定义ModelAdmin 示例_](actions.html#custom-admin-action)。

关键字参数运行你修改消息的级别、添加CSS 标签，如果`contrib.messages` 框架没有安装则默默的失败。关键字参数与[`django.contrib.messages.add_message()`](../messages.html#django.contrib.messages.add_message "django.contrib.messages.add_message") 的参数相匹配，更多细节请参见这个函数的文档。有一个不同点是级别除了使用整数/常数传递之外还以使用字符串。

`ModelAdmin.``get_paginator`(_queryset_, _per_page_, _orphans=0_, _allow_empty_first_page=True_)

返回要用于此视图的分页器的实例。默认情况下，实例化[`paginator`](#django.contrib.admin.ModelAdmin.paginator "django.contrib.admin.ModelAdmin.paginator")的实例。

`ModelAdmin.``response_add`(_request_, _obj_, _post_url_continue=None_)

为[`add_view()`](#django.contrib.admin.ModelAdmin.add_view "django.contrib.admin.ModelAdmin.add_view")阶段确定[`HttpResponse`](../../request-response.html#django.http.HttpResponse "django.http.HttpResponse")。

`response_add`在管理表单提交后，在对象和所有相关实例已创建并保存之后调用。您可以覆盖它以在对象创建后更改默认行为。

`ModelAdmin.``response_change`(_request_, _obj_)

确定[`change_view()`](#django.contrib.admin.ModelAdmin.change_view "django.contrib.admin.ModelAdmin.change_view") 阶段的[`HttpResponse`](../../request-response.html#django.http.HttpResponse "django.http.HttpResponse")。

`response_change` 在Admin 表单提交并保存该对象和所有相关的实例之后调用。您可以重写它来更改对象修改之后的默认行为。

`ModelAdmin.``response_delete`(_request_, _obj_display_, _obj_id_)

New in Django 1.7.

为[`delete_view()`](#django.contrib.admin.ModelAdmin.delete_view "django.contrib.admin.ModelAdmin.delete_view")阶段确定[`HttpResponse`](../../request-response.html#django.http.HttpResponse "django.http.HttpResponse")。

在对象已删除后调用`response_delete`。您可以覆盖它以在对象被删除后更改默认行为。

`obj_display`是具有已删除对象名称的字符串。

`obj_id`是用于检索要删除的对象的序列化标识符。

New in Django 1.8:

已添加`obj_id`参数。

`ModelAdmin.``get_changeform_initial_data`(_request_)

New in Django 1.7.

用于管理员更改表单上的初始数据的挂钩。默认情况下，字段从`GET`参数给出初始值。例如，`?name=initial_value`会将`name`字段的初始值设置为`initial_value`。

此方法应返回`{'fieldname'： 'fieldval'}`形式的字典：

```
def get_changeform_initial_data(self, request):
    return {'name': 'custom_initial_value'}

```

#### 其他方法

`ModelAdmin.``add_view`(_request_, _form_url=''_, _extra_context=None_)

Django视图为模型实例添加页面。见下面的注释。

`ModelAdmin.``change_view`(_request_, _object_id_, _form_url=''_, _extra_context=None_)

Django视图为模型实例版本页。见下面的注释。

`ModelAdmin.``changelist_view`(_request_, _extra_context=None_)

Django视图为模型实例更改列表/操作页面。见下面的注释。

`ModelAdmin.``delete_view`(_request_, _object_id_, _extra_context=None_)

模型实例删除确认页面的Django 视图。请参阅下面的注释。

`ModelAdmin.``history_view`(_request_, _object_id_, _extra_context=None_)

显示给定模型实例的修改历史的页面的Django视图。

与上一节中详述的钩型`ModelAdmin`方法不同，这五个方法实际上被设计为从管理应用程序URL调度处理程序调用为Django视图，以呈现处理模型实例的页面CRUD操作。因此，完全覆盖这些方法将显着改变管理应用程序的行为。

覆盖这些方法的一个常见原因是增加提供给呈现视图的模板的上下文数据。在以下示例中，覆盖更改视图，以便为渲染的模板提供一些额外的映射数据，否则这些数据将不可用：

```
class MyModelAdmin(admin.ModelAdmin):

    # A template for a very customized change view:
    change_form_template = 'admin/myapp/extras/openstreetmap_change_form.html'

    def get_osm_info(self):
        # ...
        pass

    def change_view(self, request, object_id, form_url='', extra_context=None):
        extra_context = extra_context or {}
        extra_context['osm_data'] = self.get_osm_info()
        return super(MyModelAdmin, self).change_view(request, object_id,
            form_url, extra_context=extra_context)

```

这些视图返回[`TemplateResponse`](../../template-response.html#django.template.response.TemplateResponse "django.template.response.TemplateResponse")实例，允许您在渲染之前轻松自定义响应数据。有关详细信息，请参阅[_TemplateResponse documentation_](../../template-response.html)。

### ModelAdmin asset definitions

有时候你想添加一些CSS和/或JavaScript到添加/更改视图。这可以通过在`ModelAdmin`上使用`Media`内部类来实现：

```
class ArticleAdmin(admin.ModelAdmin):
    class Media:
        css = {
            "all": ("my_styles.css",)
        }
        js = ("my_code.js",)

```

[_staticfiles app_](../staticfiles.html)将[`STATIC_URL`](../../settings.html#std:setting-STATIC_URL)（或[`MEDIA_URL`](../../settings.html#std:setting-MEDIA_URL)如果[`STATIC_URL`](../../settings.html#std:setting-STATIC_URL)为`None`资产路径。相同的规则适用于表单上的[_regular asset definitions on forms_](../../../topics/forms/media.html#form-asset-paths)。

#### jQuery

Django管理JavaScript使用[jQuery](http://jquery.com)库。

为了避免与用户提供的脚本或库冲突，Django的jQuery（版本1.11.2）命名为`django.jQuery`。如果您想在自己的管理JavaScript中使用jQuery而不包含第二个副本，则可以使用更改列表上的`django.jQuery`对象和添加/编辑视图。

Changed in Django 1.8:

嵌入式jQuery已经从1.9.1升级到1.11.2。

默认情况下，[`ModelAdmin`](#django.contrib.admin.ModelAdmin "django.contrib.admin.ModelAdmin")类需要jQuery，因此除非有特定需要，否则不需要向您的`ModelAdmin`的媒体资源列表添加jQuery。例如，如果您需要将jQuery库放在全局命名空间中（例如使用第三方jQuery插件时）或者如果您需要更新的jQuery版本，则必须包含自己的副本。

Django提供了jQuery的未压缩和“缩小”版本，分别是`jquery.js`和`jquery.min.js`。

[`ModelAdmin`](#django.contrib.admin.ModelAdmin "django.contrib.admin.ModelAdmin")和[`InlineModelAdmin`](#django.contrib.admin.InlineModelAdmin "django.contrib.admin.InlineModelAdmin")具有`media`属性，可返回存储到JavaScript文件的路径的`Media`对象列表形式和/或格式。如果[`DEBUG`](../../settings.html#std:setting-DEBUG)是`True`，它将返回各种JavaScript文件的未压缩版本，包括`jquery.js`；如果没有，它将返回“minified”版本。

### 向管理员添加自定义验证

在管理员中添加数据的自定义验证是很容易的。自动管理界面重用[`django.forms`](../../forms/api.html#module-django.forms "django.forms")，并且`ModelAdmin`类可以定义您自己的形式：

```
class ArticleAdmin(admin.ModelAdmin):
    form = MyArticleAdminForm

```

`MyArticleAdminForm`可以在任何位置定义，只要在需要的地方导入即可。现在，您可以在表单中为任何字段添加自己的自定义验证：

```
class MyArticleAdminForm(forms.ModelForm):
    def clean_name(self):
        # do something that validates your data
        return self.cleaned_data["name"]

```

重要的是你在这里使用`ModelForm`否则会破坏。有关详细信息，请参阅[_custom validation_](../../forms/validation.html)上的[_forms_](../../forms/index.html)文档，更具体地说，[_model form validation notes_](../../../topics/forms/modelforms.html#overriding-modelform-clean-method)。

## InlineModelAdmin objects

_class_ `InlineModelAdmin`

_class_ `TabularInline`

_class_ `StackedInline`

此管理界面能够在一个界面编辑多个Model。这些称为内联。假设你有这两个模型：

```
from django.db import models

class Author(models.Model):
   name = models.CharField(max_length=100)

class Book(models.Model):
   author = models.ForeignKey(Author)
   title = models.CharField(max_length=100)

```

The first step in displaying this intermediate model in the admin is to define an inline class for the Membership model:您可以通过在`ModelAdmin.inlines`中指定模型来为模型添加内联：

```
from django.contrib import admin

class BookInline(admin.TabularInline):
    model = Book

class AuthorAdmin(admin.ModelAdmin):
    inlines = [
        BookInline,
    ]

```

Django提供了两个`InlineModelAdmin`的子类如下:

*   [TabularInline](#django.contrib.admin.TabularInline "django.contrib.admin.TabularInline")
*   [StackedInline](#django.contrib.admin.StackedInline "django.contrib.admin.StackedInline")

这两者之间仅仅是在用于呈现他们的模板上有区别。

### InlineModelAdmin options

`InlineModelAdmin`与`ModelAdmin`具有许多相同的功能，并添加了一些自己的功能（共享功能实际上是在`BaseModelAdmin`超类中定义的）。共享功能包括：

*   [形成](#django.contrib.admin.InlineModelAdmin.form "django.contrib.admin.InlineModelAdmin.form")
*   [fieldets](#django.contrib.admin.ModelAdmin.fieldsets "django.contrib.admin.ModelAdmin.fieldsets")
*   [字段](#django.contrib.admin.ModelAdmin.fields "django.contrib.admin.ModelAdmin.fields")
*   [formfield_overrides](#django.contrib.admin.ModelAdmin.formfield_overrides "django.contrib.admin.ModelAdmin.formfield_overrides")
*   [排除](#django.contrib.admin.ModelAdmin.exclude "django.contrib.admin.ModelAdmin.exclude")
*   [filter_horizo​​ntal](#django.contrib.admin.ModelAdmin.filter_horizontal "django.contrib.admin.ModelAdmin.filter_horizontal")
*   [filter_vertical](#django.contrib.admin.ModelAdmin.filter_vertical "django.contrib.admin.ModelAdmin.filter_vertical")
*   [订购](#django.contrib.admin.ModelAdmin.ordering "django.contrib.admin.ModelAdmin.ordering")
*   [prepopulated_fields](#django.contrib.admin.ModelAdmin.prepopulated_fields "django.contrib.admin.ModelAdmin.prepopulated_fields")
*   [get_queryset()](#django.contrib.admin.ModelAdmin.get_queryset "django.contrib.admin.ModelAdmin.get_queryset")
*   [radio_fields](#django.contrib.admin.ModelAdmin.radio_fields "django.contrib.admin.ModelAdmin.radio_fields")
*   [readonly_fields](#django.contrib.admin.ModelAdmin.readonly_fields "django.contrib.admin.ModelAdmin.readonly_fields")
*   [raw_id_fields](#django.contrib.admin.InlineModelAdmin.raw_id_fields "django.contrib.admin.InlineModelAdmin.raw_id_fields")
*   [formfield_for_choice_field()](#django.contrib.admin.ModelAdmin.formfield_for_choice_field "django.contrib.admin.ModelAdmin.formfield_for_choice_field")
*   [formfield_for_foreignkey()](#django.contrib.admin.ModelAdmin.formfield_for_foreignkey "django.contrib.admin.ModelAdmin.formfield_for_foreignkey")
*   [formfield_for_manytomany()](#django.contrib.admin.ModelAdmin.formfield_for_manytomany "django.contrib.admin.ModelAdmin.formfield_for_manytomany")
*   [has_add_permission()](#django.contrib.admin.ModelAdmin.has_add_permission "django.contrib.admin.ModelAdmin.has_add_permission")
*   [has_change_permission()](#django.contrib.admin.ModelAdmin.has_change_permission "django.contrib.admin.ModelAdmin.has_change_permission")
*   [has_delete_permission()](#django.contrib.admin.ModelAdmin.has_delete_permission "django.contrib.admin.ModelAdmin.has_delete_permission")
*   [has_module_permission()](#django.contrib.admin.ModelAdmin.has_module_permission "django.contrib.admin.ModelAdmin.has_module_permission")

`InlineModelAdmin`类添加：

`InlineModelAdmin.``model`

内联正在使用的模型。这是必需的。

`InlineModelAdmin.``fk_name`

模型上的外键的名称。在大多数情况下，这将自动处理，但如果同一父模型有多个外键，则必须显式指定`fk_name`。

`InlineModelAdmin.``formset`

默认为[`BaseInlineFormSet`](../../../topics/forms/modelforms.html#django.forms.models.BaseInlineFormSet "django.forms.models.BaseInlineFormSet")。使用自己的表单可以给你很多自定义的可能性。内联围绕[_model formsets_](../../../topics/forms/modelforms.html#model-formsets)构建。

`InlineModelAdmin.``form`

`form`的值默认为`ModelForm`。这是在为此内联创建表单集时传递到[`inlineformset_factory()`](../../forms/models.html#django.forms.models.inlineformset_factory "django.forms.models.inlineformset_factory")的内容。

警告

在为`InlineModelAdmin`表单编写自定义验证时，请谨慎编写依赖于父模型功能的验证。如果父模型无法验证，则可能会处于不一致状态，如[_Validation on a ModelForm_](../../../topics/forms/modelforms.html#validation-on-modelform)中的警告中所述。

`InlineModelAdmin.``extra`

这控制除初始形式外，表单集将显示的额外表单的数量。有关详细信息，请参阅[_formsets documentation_](../../../topics/forms/formsets.html)。

对于具有启用JavaScript的浏览器的用户，提供了“添加另一个”链接，以允许除了由于`extra`参数提供的内容之外添加任意数量的其他内联。

如果当前显示的表单数量超过`max_num`，或者用户未启用JavaScript，则不会显示动态链接。

[`InlineModelAdmin.get_extra()`](#django.contrib.admin.InlineModelAdmin.get_extra "django.contrib.admin.InlineModelAdmin.get_extra")还允许您自定义额外表单的数量。

`InlineModelAdmin.``max_num`

这控制在内联中显示的表单的最大数量。这不直接与对象的数量相关，但如果值足够小，可以。有关详细信息，请参阅[_Limiting the number of editable objects_](../../../topics/forms/modelforms.html#model-formsets-max-num)。

[`InlineModelAdmin.get_max_num()`](#django.contrib.admin.InlineModelAdmin.get_max_num "django.contrib.admin.InlineModelAdmin.get_max_num")还允许您自定义最大数量的额外表单。

`InlineModelAdmin.``min_num`

New in Django 1.7.

这控制在内联中显示的表单的最小数量。有关详细信息，请参阅[`modelformset_factory()`](../../forms/models.html#django.forms.models.modelformset_factory "django.forms.models.modelformset_factory")。

[`InlineModelAdmin.get_min_num()`](#django.contrib.admin.InlineModelAdmin.get_min_num "django.contrib.admin.InlineModelAdmin.get_min_num")还允许您自定义显示的表单的最小数量。

`InlineModelAdmin.``raw_id_fields`

By default, Django’s admin uses a select-box interface (&lt;select&gt;) for fields that are `ForeignKey`. 有时，您不希望产生必须选择要在下拉列表中显示的所有相关实例的开销。

`raw_id_fields`是您希望更改为`ForeignKey`或`ManyToManyField`的`Input`窗口小部件的字段列表：

```
class BookInline(admin.TabularInline):
    model = Book
    raw_id_fields = ("pages",)

```

`InlineModelAdmin.``template`

用于在页面上呈现内联的模板。

`InlineModelAdmin.``verbose_name`

覆盖模型的内部`Meta`类中找到的`verbose_name`。

`InlineModelAdmin.``verbose_name_plural`

覆盖模型的内部`Meta`类中的`verbose_name_plural`。

`InlineModelAdmin.``can_delete`

指定是否可以在内联中删除内联对象。默认为`True`。

`InlineModelAdmin.``show_change_link`

New in Django 1.8.

指定是否可以在admin中更改的内联对象具有指向更改表单的链接。默认为`False`。

`InlineModelAdmin.``get_formset`(_request_, _obj=None_, _**kwargs_)

返回[`BaseInlineFormSet`](../../../topics/forms/modelforms.html#django.forms.models.BaseInlineFormSet "django.forms.models.BaseInlineFormSet")类，以在管理员添加/更改视图中使用。请参阅[`ModelAdmin.get_formsets_with_inlines`](#django.contrib.admin.ModelAdmin.get_formsets_with_inlines "django.contrib.admin.ModelAdmin.get_formsets_with_inlines")的示例。

`InlineModelAdmin.``get_extra`(_request_, _obj=None_, _**kwargs_)

返回要使用的其他内联表单的数量。默认情况下，返回[`InlineModelAdmin.extra`](#django.contrib.admin.InlineModelAdmin.extra "django.contrib.admin.InlineModelAdmin.extra")属性。

覆盖此方法以编程方式确定额外的内联表单的数量。例如，这可以基于模型实例（作为关键字参数`obj`传递）：

```
class BinaryTreeAdmin(admin.TabularInline):
    model = BinaryTree

    def get_extra(self, request, obj=None, **kwargs):
        extra = 2
        if obj:
            return extra - obj.binarytree_set.count()
        return extra

```

`InlineModelAdmin.``get_max_num`(_request_, _obj=None_, _**kwargs_)

返回要使用的额外内联表单的最大数量。默认情况下，返回[`InlineModelAdmin.max_num`](#django.contrib.admin.InlineModelAdmin.max_num "django.contrib.admin.InlineModelAdmin.max_num")属性。

覆盖此方法以编程方式确定内联表单的最大数量。例如，这可以基于模型实例（作为关键字参数`obj`传递）：

```
class BinaryTreeAdmin(admin.TabularInline):
    model = BinaryTree

    def get_max_num(self, request, obj=None, **kwargs):
        max_num = 10
        if obj.parent:
            return max_num - 5
        return max_num

```

`InlineModelAdmin.``get_min_num`(_request_, _obj=None_, _**kwargs_)

New in Django 1.7.

返回要使用的内联表单的最小数量。默认情况下，返回[`InlineModelAdmin.min_num`](#django.contrib.admin.InlineModelAdmin.min_num "django.contrib.admin.InlineModelAdmin.min_num")属性。

覆盖此方法以编程方式确定最小内联表单数。例如，这可以基于模型实例（作为关键字参数`obj`传递）。

### 使用具有两个或多个外键的模型到同一个父模型

有时可能有多个外键到同一个模型。以这个模型为例：

```
from django.db import models

class Friendship(models.Model):
    to_person = models.ForeignKey(Person, related_name="friends")
    from_person = models.ForeignKey(Person, related_name="from_friends")

```

如果您想在`Person`管理员添加/更改页面上显示内联，则需要明确定义外键，因为它无法自动执行：

```
from django.contrib import admin
from myapp.models import Friendship

class FriendshipInline(admin.TabularInline):
    model = Friendship
    fk_name = "to_person"

class PersonAdmin(admin.ModelAdmin):
    inlines = [
        FriendshipInline,
    ]

```

### 使用多对多模型

默认情况下，多对多关系的管理窗口小部件将显示在包含[`ManyToManyField`](../../models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField")的实际引用的任何模型上。根据您的`ModelAdmin`定义，模型中的每个多对多字段将由标准HTML `＆lt； select multiple&gt; t4&gt;`，水平或垂直过滤器或`raw_id_admin`小部件。但是，也可以用内联替换这些小部件。

假设我们有以下模型：

```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, related_name='groups')

```

如果要使用内联显示多对多关系，可以通过为关系定义`InlineModelAdmin`对象来实现：

```
from django.contrib import admin

class MembershipInline(admin.TabularInline):
    model = Group.members.through

class PersonAdmin(admin.ModelAdmin):
    inlines = [
        MembershipInline,
    ]

class GroupAdmin(admin.ModelAdmin):
    inlines = [
        MembershipInline,
    ]
    exclude = ('members',)

```

在这个例子中有两个值得注意的特征。

首先 - `MembershipInline`类引用`Group.members.through`。`through`属性是对管理多对多关系的模型的引用。在定义多对多字段时，此模型由Django自动创建。

其次，`GroupAdmin`必须手动排除`members`字段。Django在定义关系（在这种情况下，`Group`）的模型上显示多对多字段的管理窗口小部件。如果要使用内联模型来表示多对多关系，则必须告知Django的管理员_而不是_显示此窗口小部件 - 否则您最终会在管理页面上看到两个窗口小部件，用于管理关系。

在所有其他方面，`InlineModelAdmin`与任何其他方面完全相同。您可以使用任何正常的`ModelAdmin`属性自定义外观。

### 使用多对多中介模型

当您使用[`ManyToManyField`](../../models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField")的`through`参数指定中介模型时，管理员将不会默认显示窗口小部件。这是因为该中间模型的每个实例需要比可以在单个小部件中显示的更多的信息，并且多个小部件所需的布局将根据中间模型而变化。

但是，我们仍然希望能够内联编辑该信息。幸运的是，这很容易与内联管理模型。假设我们有以下模型：

```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')

class Membership(models.Model):
    person = models.ForeignKey(Person)
    group = models.ForeignKey(Group)
    date_joined = models.DateField()
    invite_reason = models.CharField(max_length=64)

```

在管理中显示此中间模型的第一步是为`Membership`模型定义一个内联类：

```
class MembershipInline(admin.TabularInline):
    model = Membership
    extra = 1

```

此简单示例使用`Membership`模型的默认`InlineModelAdmin`值，并将额外添加表单限制为一个。这可以使用`InlineModelAdmin`类可用的任何选项进行自定义。

现在为`Person`和`Group`模型创建管理视图：

```
class PersonAdmin(admin.ModelAdmin):
    inlines = (MembershipInline,)

class GroupAdmin(admin.ModelAdmin):
    inlines = (MembershipInline,)

```

最后，向管理网站注册您的`Person`和`Group`模型：

```
admin.site.register(Person, PersonAdmin)
admin.site.register(Group, GroupAdmin)

```

现在，您的管理网站已设置为从`Person`或`Group`详细信息页面内联编辑`Membership`对象。

### 使用泛型关系作为内联

可以使用内联与一般相关的对象。假设您有以下型号：

```
from django.db import models
from django.contrib.contenttypes.fields import GenericForeignKey

class Image(models.Model):
    image = models.ImageField(upload_to="images")
    content_type = models.ForeignKey(ContentType)
    object_id = models.PositiveIntegerField()
    content_object = GenericForeignKey("content_type", "object_id")

class Product(models.Model):
    name = models.CharField(max_length=100)

```

If you want to allow editing and creating `Image` instance on the `Product` add/change views you can use [`GenericTabularInline`](../contenttypes.html#django.contrib.contenttypes.admin.GenericTabularInline "django.contrib.contenttypes.admin.GenericTabularInline") or [`GenericStackedInline`](../contenttypes.html#django.contrib.contenttypes.admin.GenericStackedInline "django.contrib.contenttypes.admin.GenericStackedInline") (both subclasses of [`GenericInlineModelAdmin`](../contenttypes.html#django.contrib.contenttypes.admin.GenericInlineModelAdmin "django.contrib.contenttypes.admin.GenericInlineModelAdmin")) provided by [`admin`](../contenttypes.html#module-django.contrib.contenttypes.admin "django.contrib.contenttypes.admin"), they implement tabular and stacked visual layouts for the forms representing the inline objects respectively just like their non-generic counterparts and behave just like any other inline. 在此示例应用的`admin.py`中：

```
from django.contrib import admin
from django.contrib.contenttypes.admin import GenericTabularInline

from myproject.myapp.models import Image, Product

class ImageInline(GenericTabularInline):
    model = Image

class ProductAdmin(admin.ModelAdmin):
    inlines = [
        ImageInline,
    ]

admin.site.register(Product, ProductAdmin)

```

有关更多具体信息，请参阅[_contenttypes documentation_](../contenttypes.html)。

## 重写 admin 模板

相对重写一个admin站点的各类页面，直接在admin站点默认templates上直接进行修改是件相对简单的事。你甚至可以为特定的应用或一个特定的模型覆盖少量的这些模板。

### 设置项目的Admin模板目录

Admin模板文件位于`contrib/admin/templates/admin` 目录中。

如要覆盖一个或多个模板，首先在你的项目的`templates` 目录中创建一个`admin` 目录。它可以是你在[`TEMPLATES`](../../settings.html#std:setting-TEMPLATES) 设置的`DjangoTemplates` 后端的[`DIRS`](../../settings.html#std:setting-TEMPLATES-DIRS) 选项中指定的任何目录。如果你已经自定义`'loaders'` 选项，请确保`'django.template.loaders.filesystem.Loader'` 出现在 `'django.template.loaders.app_directories` 之前。Loader'，以便在包含[`django.contrib.admin`](#module-django.contrib.admin "django.contrib.admin: Django's admin site.")的模板之前，模板加载系统可以找到您的自定义模板。

在 `admin` 目录下, 以你的应用名创建子目录. 在应用名的目录下，以你模型层的名字创建子目录. 注意：admin应用会以小写名的形式在目录下查找模型, 如果你想在大小写敏感的文件系统上运行app，请确保以小写形式命名目录.

为一个特定的app重写admin模板, 需要拷贝`django/contrib/admin/templates/admin` 目录到你刚才创建的目录下, 并且修改它们.

For example, if we wanted to add a tool to the change list view for all the models in an app named `my_app`, we would copy `contrib/admin/templates/admin/change_list.html` to the `templates/admin/my_app/` directory of our project, and make any necessary changes.

如果我们只想为名为“Page”的特定模型添加一个工具到更改列表视图，我们将把同一个文件复制到我们项目的`templates/admin/my_app/page`目录。

### 覆盖与替换管理模板

由于管理模板的模块化设计，通常既不必要也不建议替换整个模板。最好只覆盖模板中需要更改的部分。

要继续上述示例，我们要为`Page`模型的`History`工具旁边添加一个新链接。查看`change_form.html`后，我们确定我们只需要覆盖`object-tools-items`块。因此，这里是我们的新`change_form.html`：

```
{% extends "admin/change_form.html" %}
{% load i18n admin_urls %}
{% block object-tools-items %}
    <li>
        <a href="{% url opts|admin_urlname:'history' original.pk|admin_urlquote %}" class="historylink">{% trans "History" %}</a>
    </li>
    <li>
        <a href="mylink/" class="historylink">My Link</a>
    </li>
    {% if has_absolute_url %}
        <li>
            <a href="{% url 'admin:view_on_site' content_type_id original.pk %}" class="viewsitelink">{% trans "View on site" %}</a>
        </li>
    {% endif %}
{% endblock %}

```

就是这样！如果我们将此文件放在`templates/admin/my_app`目录中，我们的链接将出现在my_app中所有模型的更改表单上。

### 每个应用或模型中可以被重写的模板

不是`contrib/admin/templates/admin` 中的每个模板都可以在每个应用或每个模型中覆盖。以下可以 ︰

*   `app_index.html`
*   `change_form.html`
*   `change_list.html`
*   `delete_confirmation.html`
*   `object_history.html`

对于那些不能以这种方式重写的模板，你可能仍然为您的整个项目重写它们。只需要将新版本放在你的`templates/admin` 目录下。这对于要创建自定义的404 和500 页面特别有用。

注意

一些Admin的模板，例如`change_list_results.html` 用于呈现自定义包含标签。这些可能会被覆盖，但在这种情况下你可能最好是创建您自己的版本Tag，并给它一个不同的名称。这样你可以有选择地使用它。

### Root and login 模板

如果你想要更改主页、 登录或登出页面的模板，你最后创建你自己的`AdminSite` 实例（见下文），并更改[`AdminSite.index_template`](#django.contrib.admin.AdminSite.index_template "django.contrib.admin.AdminSite.index_template")、[`AdminSite.login_template`](#django.contrib.admin.AdminSite.login_template "django.contrib.admin.AdminSite.login_template") 和[`AdminSite.logout_template`](#django.contrib.admin.AdminSite.logout_template "django.contrib.admin.AdminSite.logout_template") 属性。

## AdminSite objects

_class_ `AdminSite`(_name='admin'_)

Django 的一个Admin 站点通过`django.contrib.admin.sites.AdminSite` 的一个实例表示；默认创建的这个类实例是`django.contrib.admin.site`，你可以通过它注册自己的模型和`ModelAdmin` 实例。

当构造`AdminSite` 的实例时，你可以使用`name` 参数给构造函数提供一个唯一的实例名称。这个实例名称用于标识实例，尤其是[_反向解析Admin URLs_](#admin-reverse-urls) 的时候。如果没有提供实例的名称，将使用默认的实例名称`admin`。有关自定义[`AdminSite`](#django.contrib.admin.AdminSite "django.contrib.admin.AdminSite") 类的示例，请参见[_自定义AdminSite 类_](#customizing-adminsite)。

### AdminSite attributes

如[_覆盖Admin 模板_](#admin-overriding-templates)中所述，模板可以覆盖或扩展基础的Admin 模板。

`AdminSite.``site_header`

New in Django 1.7.

每个Admin 页面顶部的文本，形式为`&lt;h1&gt;`（字符串）。默认为 “Django administration”。

`AdminSite.``site_title`

New in Django 1.7.

每个Admin 页面底部的文本，形式为`&lt;title&gt;`（字符串）。默认为“Django site admin”。

`AdminSite.``site_url`

New in Django 1.8.

每个Admin 页面顶部"View site" 链接的URL。默认情况下，`site_url` 为`/`。设置为`None` 可以删除这个链接。

`AdminSite.``index_title`

New in Django 1.7.

Admin 主页顶部的文本（一个字符串）。默认为 “Site administration”。

`AdminSite.``index_template`

Admin 站点主页的视图使用的自定义模板的路径。

`AdminSite.``app_index_template`

Admin 站点app index 的视图使用的自定义模板的路径。

`AdminSite.``login_template`

Admin 站点登录视图使用的自定义模板的路径。

`AdminSite.``login_form`

Admin 站点登录视图使用的[`AuthenticationForm`](../../../topics/auth/default.html#django.contrib.auth.forms.AuthenticationForm "django.contrib.auth.forms.AuthenticationForm") 的子类。

`AdminSite.``logout_template`

Admin 站点登出视图使用的自定义模板的路径。

`AdminSite.``password_change_template`

Admin 站点密码修改视图使用的自定义模板的路径。

`AdminSite.``password_change_done_template`

Admin 站点密码修改完成视图使用的自定义模板的路径。

### AdminSite methods

`AdminSite.``each_context`(_request_)

New in Django 1.7.

返回一个字典，包含将放置在Admin 站点每个页面的模板上下文中的变量。

包含以下变量和默认值：

*   `site_header`：[`AdminSite.site_header`](#django.contrib.admin.AdminSite.site_header "django.contrib.admin.AdminSite.site_header")
*   `site_title`：[`AdminSite.site_title`](#django.contrib.admin.AdminSite.site_title "django.contrib.admin.AdminSite.site_title")
*   `site_url`：[`AdminSite.site_url`](#django.contrib.admin.AdminSite.site_url "django.contrib.admin.AdminSite.site_url")
*   `has_permission`：[`AdminSite.has_permission()`](#django.contrib.admin.AdminSite.has_permission "django.contrib.admin.AdminSite.has_permission")

Changed in Django 1.8:

添加`request` 参数和`has_permission` 变量。

`AdminSite.``has_permission`(_request_)

对于给定的`HttpRequest`，如果用户有权查看Admin 网站中的至少一个页面，则返回 `True`。默认要求[`User.is_active`](../auth.html#django.contrib.auth.models.User.is_active "django.contrib.auth.models.User.is_active") 和[`User.is_staff`](../auth.html#django.contrib.auth.models.User.is_staff "django.contrib.auth.models.User.is_staff") 都为`True`。

### 绑定`AdminSite`

设置Django Admin 的最后一步是放置你的`AdminSite` 到你的URLconf 中。通过指向给定的URL 到`AdminSite.urls` 方法来执行此操作。

在下面的示例中，我们注册默认的`AdminSite` 实例`django.contrib.admin.site` 到 URL `/admin/`。

```
# urls.py
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
]

```

### 自定义[`AdminSite`](#django.contrib.admin.AdminSite "django.contrib.admin.AdminSite")

如果你想要建立你自己的具有自定义行为Admin 站点，你可以自由地子类化`AdminSite` 并重写或添加任何你喜欢的东西。你只需创建`AdminSite` 子类的实例（方式与你会实例化任何其它Python 类相同） 并注册你的模型和`ModelAdmin` 子类与它而不是默认的站点。最后，更新`myproject/urls.py` 来引用你的[`AdminSite`](#django.contrib.admin.AdminSite "django.contrib.admin.AdminSite") 子类。

myapp/admin.py

```
from django.contrib.admin import AdminSite

from .models import MyModel

class MyAdminSite(AdminSite):
    site_header = 'Monty Python administration'

admin_site = MyAdminSite(name='myadmin')
admin_site.register(MyModel)

```

myproject/urls.py

```
from django.conf.urls import include, url

from myapp.admin import admin_site

urlpatterns = [
    url(r'^myadmin/', include(admin_site.urls)),
]

```

注意，当使用你自己的`AdminSite` 实例时，你可能不希望自动发现`admin` 模块，因为这将导入`admin` 模块到你的每个`myproject.admin` 模块中 。这时，你需要将`'django.contrib.admin.apps.SimpleAdminConfig'` 而不是`'django.contrib.admin'` 放置在你的[`INSTALLED_APPS`](../../settings.html#std:setting-INSTALLED_APPS) 设置中。

### 相同URLconf 中有多个Admin 站点

在Django 构建的同一Web 站点上创建Admin 站点的多个实例非常容易。只需要创建`AdminSite` 的多个实例并将每个实例放置在不同的URL 下。

在下面的示例中，`/basic-admin/` 和`/advanced-admin/` 分别使用`AdminSite` 的`myproject.admin.basic_site` 实例和`myproject.admin.advanced_site` 实例表示不同版本的Admin 站点：

```
# urls.py
from django.conf.urls import include, url
from myproject.admin import basic_site, advanced_site

urlpatterns = [
    url(r'^basic-admin/', include(basic_site.urls)),
    url(r'^advanced-admin/', include(advanced_site.urls)),
]

```

`AdminSite` 实例的构造函数中接受一个单一参数用做它们的名字，可以是任何你喜欢的东西。此参数将成为[_反向解析它们_](#admin-reverse-urls) 时URL 名称的前缀。只有在你使用多个`AdminSite` 时它才是必要的。

### 向管理网站添加视图

与[`ModelAdmin`](#django.contrib.admin.ModelAdmin "django.contrib.admin.ModelAdmin")一样，[`AdminSite`](#django.contrib.admin.AdminSite "django.contrib.admin.AdminSite")提供了一个[`get_urls()`](#django.contrib.admin.ModelAdmin.get_urls "django.contrib.admin.ModelAdmin.get_urls")方法，可以重写该方法以定义网站的其他视图。要向您的管理网站添加新视图，请扩展基本[`get_urls()`](#django.contrib.admin.ModelAdmin.get_urls "django.contrib.admin.ModelAdmin.get_urls")方法，为新视图添加模式。

注意

您呈现的任何使用管理模板的视图或扩展基本管理模板，应在渲染模板之前设置`request.current_app`。It should be set to either `self.name` if your view is on an `AdminSite` or `self.admin_site.name` if your view is on a `ModelAdmin`.

Changed in Django 1.8:

在以前的Django版本中，在呈现模板时，您必须向[`RequestContext`](../../templates/api.html#django.template.RequestContext "django.template.RequestContext")或[`Context`](../../templates/api.html#django.template.Context "django.template.Context")提供`current_app`参数。

### 加入一个密码重置的特性

想admin site加入密码重置功能只需要在url配置文件中简单加入几行代码即可。具体操作就是加入下面四个正则规则。

```
from django.contrib.auth import views as auth_views

url(r'^admin/password_reset/$', auth_views.password_reset, name='admin_password_reset'),
url(r'^admin/password_reset/done/$', auth_views.password_reset_done, name='password_reset_done'),
url(r'^reset/(?P<uidb64>[0-9A-Za-z_\-]+)/(?P<token>.+)/$', auth_views.password_reset_confirm, name='password_reset_confirm'),
url(r'^reset/done/$', auth_views.password_reset_complete, name='password_reset_complete'),

```

（假设您已在`admin/`添加了管理员，并要求您在包含管理应用程序的行之前将`^admin/`开头的网址）。

如果存在`admin_password_reset`命名的URL，则会在密码框下的默认管理登录页面上显示“忘记了您的密码？”链接。

## 反向解析Admin 的URL

[`AdminSite`](#django.contrib.admin.AdminSite "django.contrib.admin.AdminSite") 部署后，该站点所提供的视图都可以使用Django 的[_URL 反向解析系统_](../../../topics/http/urls.html#naming-url-patterns)访问。

[`AdminSite`](#django.contrib.admin.AdminSite "django.contrib.admin.AdminSite") 提供以下命名URL：

<colgroup><col width="30%"> <col width="29%"> <col width="41%"></colgroup> 
| Page | URL name | Parameters |
| --- | --- | --- |
| Index | `index` |   |
| Logout | `logout` |   |
| Password change | `password_change` |   |
| Password change done | `password_change_done` |   |
| i18n JavaScript | `jsi18n` |   |
| Application index page | `app_list` | `app_label` |
| Redirect to object’s page | `view_on_site` | `content_type_id`, `object_id` |

每个[`ModelAdmin`](#django.contrib.admin.ModelAdmin "django.contrib.admin.ModelAdmin") 实例还将提供额外的命名URL：

<colgroup><col width="27%"> <col width="57%"> <col width="16%"></colgroup> 
| Page | URL name | Parameters |
| --- | --- | --- |
| Changelist | `{{ app_label }}_{{ model_name }}_changelist` |   |
| Add | `{{ app_label }}_{{ model_name }}_add` |   |
| History | `{{ app_label }}_{{ model_name }}_history` | `object_id` |
| Delete | `{{ app_label }}_{{ model_name }}_delete` | `object_id` |
| Change | `{{ app_label }}_{{ model_name }}_change` | `object_id` |

这些命名URL 注册的应用命名空间为`admin`，实例命名空间为对应的AdminSite 实例的名称。

所以，如果你想要获取默认Admin 中，（polls 应用的） 一个特定的`Choice` 对象的更改视图的引用，你可以调用︰

```
>>> from django.core import urlresolvers
>>> c = Choice.objects.get(...)
>>> change_url = urlresolvers.reverse('admin:polls_choice_change', args=(c.id,))

```

这将找到管理应用程序的第一个注册实例（无论实例名称），并解析到视图以更改`poll.`该实例中的选择实例。

如果你想要查找一个特定的Admin 实例中URL，请提供实例的名称作为`current_app` 给反向解析的调用 。例如，如果你希望得到名为`custom` 的Admin 实例中的视图，你将需要调用︰

```
>>> change_url = urlresolvers.reverse('admin:polls_choice_change',
...                                   args=(c.id,), current_app='custom')

```

有关更多详细信息，请参阅[_反向解析名称空间URL_](../../../topics/http/urls.html#topics-http-reversing-url-namespaces) 的文档。

为了让模板中反向解析Admin URL 更加容易，Django 提供一个`admin_urlname` 过滤器，它以Action 作为参数︰

```
{% load admin_urls %}
<a href="{% url opts|admin_urlname:'add' %}">Add user</a>
<a href="{% url opts|admin_urlname:'delete' user.pk %}">Delete this user</a>

```

在上面的例子中Action 将匹配上文所述的[`ModelAdmin`](#django.contrib.admin.ModelAdmin "django.contrib.admin.ModelAdmin") 实例的URL 名称的最后部分。`opts` 变量可以是任何具有`app_label` 和`model_name` 属性的对象，通常由Admin 视图为当前的模型提供。

