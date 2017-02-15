

# 从模型创建表单

## ModelForm

_class_ `ModelForm`

如果你正在构建一个数据库驱动的应用，那么你应该会有与Django 的模型紧密映射的表单。举个例子，你也许会有个`BlogComment` 模型，并且你还想创建一个表单让大家提交评论到这个模型中。 在这种情况下，在表单中定义字段将是冗余的，因为你已经在模型中定义了字段。

基于这个原因，Django 提供一个辅助类来让你可以从Django 的模型创建`表单`。

例如：

```
>>> from django.forms import ModelForm
>>> from myapp.models import Article

# Create the form class.
>>> class ArticleForm(ModelForm):
...     class Meta:
...         model = Article
...         fields = ['pub_date', 'headline', 'content', 'reporter']

# Creating a form to add an article.
>>> form = ArticleForm()

# Creating a form to change an existing article.
>>> article = Article.objects.get(pk=1)
>>> form = ArticleForm(instance=article)

```

### 字段类型

生成的`表单`类中将具有和指定的模型字段对应的表单字段，顺序为`fields` 属性中指定的顺序。

每个模型字段有一个对应的默认表单字段。比如，模型中的`CharField` 表现成表单中的`CharField`。模型中的`ManyToManyField` 字段会表现成`MultipleChoiceField` 字段。下面是一个完整的列表：


| Model field | Form field |
| --- | --- |
| [`AutoField`](../../ref/models/fields.html#django.db.models.AutoField "django.db.models.AutoField") | Not represented in the form |
| [`BigIntegerField`](../../ref/models/fields.html#django.db.models.BigIntegerField "django.db.models.BigIntegerField") | [`IntegerField`](../../ref/forms/fields.html#django.forms.IntegerField "django.forms.IntegerField") with `min_value` set to -9223372036854775808 and `max_value` set to 9223372036854775807. |
| [`BooleanField`](../../ref/models/fields.html#django.db.models.BooleanField "django.db.models.BooleanField") | [`BooleanField`](../../ref/forms/fields.html#django.forms.BooleanField "django.forms.BooleanField") |
| [`CharField`](../../ref/models/fields.html#django.db.models.CharField "django.db.models.CharField") | [`CharField`](../../ref/forms/fields.html#django.forms.CharField "django.forms.CharField") with `max_length` set to the model field’s `max_length` |
| [`CommaSeparatedIntegerField`](../../ref/models/fields.html#django.db.models.CommaSeparatedIntegerField "django.db.models.CommaSeparatedIntegerField") | [`CharField`](../../ref/forms/fields.html#django.forms.CharField "django.forms.CharField") |
| [`DateField`](../../ref/models/fields.html#django.db.models.DateField "django.db.models.DateField") | [`DateField`](../../ref/forms/fields.html#django.forms.DateField "django.forms.DateField") |
| [`DateTimeField`](../../ref/models/fields.html#django.db.models.DateTimeField "django.db.models.DateTimeField") | [`DateTimeField`](../../ref/forms/fields.html#django.forms.DateTimeField "django.forms.DateTimeField") |
| [`DecimalField`](../../ref/models/fields.html#django.db.models.DecimalField "django.db.models.DecimalField") | [`DecimalField`](../../ref/forms/fields.html#django.forms.DecimalField "django.forms.DecimalField") |
| [`EmailField`](../../ref/models/fields.html#django.db.models.EmailField "django.db.models.EmailField") | [`EmailField`](../../ref/forms/fields.html#django.forms.EmailField "django.forms.EmailField") |
| [`FileField`](../../ref/models/fields.html#django.db.models.FileField "django.db.models.FileField") | [`FileField`](../../ref/forms/fields.html#django.forms.FileField "django.forms.FileField") |
| [`FilePathField`](../../ref/models/fields.html#django.db.models.FilePathField "django.db.models.FilePathField") | [`FilePathField`](../../ref/forms/fields.html#django.forms.FilePathField "django.forms.FilePathField") |
| [`FloatField`](../../ref/models/fields.html#django.db.models.FloatField "django.db.models.FloatField") | [`FloatField`](../../ref/forms/fields.html#django.forms.FloatField "django.forms.FloatField") |
| [`ForeignKey`](../../ref/models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey") | [`ModelChoiceField`](../../ref/forms/fields.html#django.forms.ModelChoiceField "django.forms.ModelChoiceField") (see below) |
| `ImageField` | [`ImageField`](../../ref/forms/fields.html#django.forms.ImageField "django.forms.ImageField") |
| [`IntegerField`](../../ref/models/fields.html#django.db.models.IntegerField "django.db.models.IntegerField") | [`IntegerField`](../../ref/forms/fields.html#django.forms.IntegerField "django.forms.IntegerField") |
| `IPAddressField` | `IPAddressField` |
| [`GenericIPAddressField`](../../ref/models/fields.html#django.db.models.GenericIPAddressField "django.db.models.GenericIPAddressField") | [`GenericIPAddressField`](../../ref/forms/fields.html#django.forms.GenericIPAddressField "django.forms.GenericIPAddressField") |
| [`ManyToManyField`](../../ref/models/fields.html#django.db.models.ManyToManyField "django.db.models.ManyToManyField") | [`ModelMultipleChoiceField`](../../ref/forms/fields.html#django.forms.ModelMultipleChoiceField "django.forms.ModelMultipleChoiceField") (see below) |
| [`NullBooleanField`](../../ref/models/fields.html#django.db.models.NullBooleanField "django.db.models.NullBooleanField") | [`NullBooleanField`](../../ref/forms/fields.html#django.forms.NullBooleanField "django.forms.NullBooleanField") |
| [`PositiveIntegerField`](../../ref/models/fields.html#django.db.models.PositiveIntegerField "django.db.models.PositiveIntegerField") | [`IntegerField`](../../ref/forms/fields.html#django.forms.IntegerField "django.forms.IntegerField") |
| [`PositiveSmallIntegerField`](../../ref/models/fields.html#django.db.models.PositiveSmallIntegerField "django.db.models.PositiveSmallIntegerField") | [`IntegerField`](../../ref/forms/fields.html#django.forms.IntegerField "django.forms.IntegerField") |
| [`SlugField`](../../ref/models/fields.html#django.db.models.SlugField "django.db.models.SlugField") | [`SlugField`](../../ref/forms/fields.html#django.forms.SlugField "django.forms.SlugField") |
| [`SmallIntegerField`](../../ref/models/fields.html#django.db.models.SmallIntegerField "django.db.models.SmallIntegerField") | [`IntegerField`](../../ref/forms/fields.html#django.forms.IntegerField "django.forms.IntegerField") |
| [`TextField`](../../ref/models/fields.html#django.db.models.TextField "django.db.models.TextField") | [`CharField`](../../ref/forms/fields.html#django.forms.CharField "django.forms.CharField") with `widget=forms.Textarea` |
| [`TimeField`](../../ref/models/fields.html#django.db.models.TimeField "django.db.models.TimeField") | [`TimeField`](../../ref/forms/fields.html#django.forms.TimeField "django.forms.TimeField") |
| [`URLField`](../../ref/models/fields.html#django.db.models.URLField "django.db.models.URLField") | [`URLField`](../../ref/forms/fields.html#django.forms.URLField "django.forms.URLField") |

可能如你所料，`ForeignKey` 和 `ManyToManyField` 字段类型属于特殊情况：

*   `ForeignKey` 表示成`django.forms.ModelChoiceField`，它是一个`ChoiceField`，其选项是模型的`查询集`。
*   `ManyToManyField` 表示成`django.forms.ModelMultipleChoiceField`，它是一个`MultipleChoiceField`，其选项是模型的`查询集`。

此外，生成的每个表单字段都有以下属性集：

*   如果模型字段设置`blank=True`，那么表单字段的`required` 设置为`False`。否则，`required=True`。
*   表单字段的`label` 设置为模型字段的`verbose_name`，并将第一个字母大写。
*   表单字段的`help_text` 设置为模型字段的`help_text`。
*   如果模型字段设置了`choices`，那么表单字段的`Widget` 将设置成`Select`，其选项来自模型字段的`choices`。选项通常会包含空选项，并且会默认选择。如果字段是必选的，它会强制用户选择一个选项。如果模型字段的`blank=False` 且具有一个显示的`default` 值，将不会包含空选项（初始将选择`default` 值）。

最后，请注意你可以为给定的模型字段重新指定表单字段。参见下文[覆盖默认的字段](#overriding-the-default-fields)。

### 一个完整的例子

考虑下面的模型：

```
from django.db import models
from django.forms import ModelForm

TITLE_CHOICES = (
    ('MR', 'Mr.'),
    ('MRS', 'Mrs.'),
    ('MS', 'Ms.'),
)

class Author(models.Model):
    name = models.CharField(max_length=100)
    title = models.CharField(max_length=3, choices=TITLE_CHOICES)
    birth_date = models.DateField(blank=True, null=True)

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Book(models.Model):
    name = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author)

class AuthorForm(ModelForm):
    class Meta:
        model = Author
        fields = ['name', 'title', 'birth_date']

class BookForm(ModelForm):
    class Meta:
        model = Book
        fields = ['name', 'authors']

```

上面`ModelForm` 的子类大体等同于（唯一的不同是`save()` 方法，我们将稍后讨论）：

```
from django import forms

class AuthorForm(forms.Form):
    name = forms.CharField(max_length=100)
    title = forms.CharField(max_length=3,
                widget=forms.Select(choices=TITLE_CHOICES))
    birth_date = forms.DateField(required=False)

class BookForm(forms.Form):
    name = forms.CharField(max_length=100)
    authors = forms.ModelMultipleChoiceField(queryset=Author.objects.all())

```

### `模型表单`的验证

验证`模型表单`主要有两步：

1.  [验证表单](../../ref/forms/validation.html#form-and-field-validation)
2.  [验证模型实例](../../ref/models/instances.html#validating-objects)

与普通的表单验证类型类似，模型表单的验证在调用[`is_valid()`](../../ref/forms/api.html#django.forms.Form.is_valid "django.forms.Form.is_valid") 或访问[`errors`](../../ref/forms/api.html#django.forms.Form.errors "django.forms.Form.errors") 属性时隐式调用，或者通过`full_clean()` 显式调用，尽管在实际应用中你将很少使用后一种方法。

`模型`的验证（[`Model.full_clean()`](../../ref/models/instances.html#django.db.models.Model.full_clean "django.db.models.Model.full_clean")）在表单验证这一步的内部触发，紧跟在表单的`clean()` 方法调用之后。

警告

Clean 过程会以各种方式修改传递给`模型表单`构造函数的模型实例。例如，模型的日期字段将转换成日期对象。验证失败可能导致模型实例处于不一致的状态，所以不建议重新使用它。

#### 重写clean() 方法

可以重写模型表单的`clean()` 来提供额外的验证，方法和普通的表单一样。

模型表单实例包含一个`instance` 属性，表示与它绑定的模型实例。

警告

`ModelForm.clean()` 方法设置一个标识符， 使得[_模型验证_](../../ref/models/instances.html#validating-objects) 这一步验证标记为`unique`、 `unique_together` 或`unique_for_date|month|year` 的模型字段的唯一性。

如果你需要覆盖`clean()` 方法并维持这个验证行为，你必须调用父类的`clean()` 方法。

#### 与模型验证的交互

作为验证过程的一部分，`模型表单`将调用与表单字段对应的每个模型字段的`clean()` 方法。如果你已经排除某些模型字段，这些字段不会运行验证。关于字段clean 和验证是如何工作的，参见[_表单字段_](../../ref/forms/validation.html)的文档。

模型的`clean()` 方法在任何唯一性检查之前调用。关于模型`clean()` 钩子的更多信息，参见[_验证对象_](../../ref/models/instances.html#validating-objects) 。

#### 模型`error_messages` 的注意事项

[`表单字段`](../../ref/forms/fields.html#django.forms.Field.error_messages "django.forms.Field.error_messages")级别或[_表单_](#modelforms-overriding-default-fields)级别的错误信息永远比[`模型字段`](../../ref/models/fields.html#django.db.models.Field.error_messages "django.db.models.Field.error_messages")级别的错误信息优先。

[`模型字段`](../../ref/models/fields.html#django.db.models.Field.error_messages "django.db.models.Field.error_messages")的错误信息只用于[_模型验证_](../../ref/models/instances.html#validating-objects)步骤引发`ValidationError` 的时候，且不会有对应的表单级别的错误信息。

New in Django 1.7.

你可以根据模型验证引发的`NON_FIELD_ERRORS` 覆盖错误信息，方法是添加 [`NON_FIELD_ERRORS`](../../ref/exceptions.html#django.core.exceptions.NON_FIELD_ERRORS "django.core.exceptions.NON_FIELD_ERRORS") 键到`模型表单`内联`Meta` 类的`error_messages` 字典：

```
from django.forms import ModelForm
from django.core.exceptions import NON_FIELD_ERRORS

class ArticleForm(ModelForm):
    class Meta:
        error_messages = {
            NON_FIELD_ERRORS: {
                'unique_together': "%(model_name)s's %(field_labels)s are not unique.",
            }
        }

```

### `save()` 方法

每个`模型表单`还具有一个`save()` 方法。这个方法根据表单绑定的数据创建并保存数据库对象。`模型表单`的子类可以用关键字参数`instance` 接收一个已经存在的模型实例；如果提供，`save()` 将更新这个实例。如果没有提供，`save()` 将创建模型的一个新实例：

```
>>> from myapp.models import Article
>>> from myapp.forms import ArticleForm

# Create a form instance from POST data.
>>> f = ArticleForm(request.POST)

# Save a new Article object from the form's data.
>>> new_article = f.save()

# Create a form to edit an existing Article, but use
# POST data to populate the form.
>>> a = Article.objects.get(pk=1)
>>> f = ArticleForm(request.POST, instance=a)
>>> f.save()

```

注意，如果表单[_没有验证_](#validation-on-modelform)，`save()` 调用将通过检查`form.errors` 来进行验证。如果表单中的数据不合法，将引发`ValueError` —— 例如，如果`form.errors` 为`True`。

`save()` 接受一个可选的`commit` 关键字参数，其值为`True` 或`False`。如果`save()` 时`commit=False`，那么它将返回一个还没有保存到数据库的对象。这种情况下，你需要调用返回的模型实例的`save()`。 如果你想在保存之前自定义一些处理，或者你想使用特定的[_模型保存选项_](../../ref/models/instances.html#ref-models-force-insert)，可以这样使用。`commit` 默认为`True`。

使用`commit=False` 的另外一个副作用是在模型具有多对多关系的时候。如果模型具有多对多关系而且当你保存表单时指定`commit=False`，Django 不会立即为多对多关系保存表单数据。这是因为只有实例在数据库中存在时才可以保存实例的多对多数据。

为了解决这个问题，每当你使用`commit=False` 保存表单时，Django 将添加一个`save_m2m()` 方法到你的`模型表单`子类。在你手工保存有表单生成的实例之后，你可以调用`save_m2m()` 来保存多对多的表单数据。例如：

```
# Create a form instance with POST data.
>>> f = AuthorForm(request.POST)

# Create, but don't save the new author instance.
>>> new_author = f.save(commit=False)

# Modify the author in some way.
>>> new_author.some_field = 'some_value'

# Save the new instance.
>>> new_author.save()

# Now, save the many-to-many data for the form.
>>> f.save_m2m()

```

`save_m2m()` 只在你使用`save(commit=False)` 时才需要。当你直接使用`save()`，所有的数据 —— 包括多对多数据 —— 都将保存而不需要任何额外的方法调用。例如：

```
# Create a form instance with POST data.
>>> a = Author()
>>> f = AuthorForm(request.POST, instance=a)

# Create and save the new author instance. There's no need to do anything else.
>>> new_author = f.save()

```

除了`save()` 和`save_m2m()` 方法之外，`模型表单`与其它`表单`的工作方式完全一样。例如，`is_valid()`用于检查合法性，`is_multipart()` 方法用于决定表单是否需要multipart 的文件上传（以及这之后`request.FILES` 是否必须必须传递给表单）等等。更多信息，参见[_绑定上传的文件到表单_](../../ref/forms/api.html#binding-uploaded-files)。

### 选择用到的字段

强烈建议你使用`fields` 属性显式设置所有将要在表单中编辑的字段。如果不这样做，当表单不小心允许用户设置某些特定的字段，特别是有的字段添加到模型中的时候，将很容易导致安全问题。这些问题可能在网页上根本看不出来，它与表单的渲染方式有关。

另外一种方式是自动包含所有的字段，或者排除某些字段。这种基本方式的安全性要差很多，而且已经导致大型的网站受到严重的利用（例如 [GitHub](https://github.com/blog/1068-public-key-security-vulnerability-and-mitigation)）。

然而，有两种简单的方法保证你不会出现这些安全问题：

1.  设置`fields` 属性为特殊的值`'__all__'` 以表示需要使用模型的所有字段。例如：

    ```
    from django.forms import ModelForm

    class AuthorForm(ModelForm):
        class Meta:
            model = Author
            fields = '__all__'

    ```

2.  设置`ModelForm` 内联的`Meta` 类的`exclude` 属性为一个要从表单中排除的字段的列表。

    例如：

    ```
    class PartialAuthorForm(ModelForm):
        class Meta:
            model = Author
            exclude = ['title']

    ```

    因为`Author` 模型有3个字段`name`、`title` 和 `birth_date`，上面的例子会让`name` 和 `birth_date` 出现在表单中。

如果使用上面两种方法，表单中字段出现的顺序将和字段在模型中定义的顺序一致，其中`ManyToManyField` 出现在最后。

另外，Django 还将使用以下规则：如果设置模型字段的`editable=False`，那么使用`ModelForm` 从该模型创建的_任何_表单都不会包含该字段。

Changed in Django 1.8:

在旧的版本中，同时省略`fields` 和`exclude` 字段将导致模型的所有字段出现在表单中。现在这样做将引发一个[`ImproperlyConfigured`](../../ref/exceptions.html#django.core.exceptions.ImproperlyConfigured "django.core.exceptions.ImproperlyConfigured") 异常。

注

不会被上述逻辑包含进表单中的字段将不会被表单的`save()` 方法保存。另外，如果你手工添加排除的字段到表单中，它们也不会从模型实例初始化。

Django 将阻止保存不完全的模型，所以如果模型不允许缺失的字段为空且没有提供默认值，带有缺失字段的`ModelForm` 的`save()`将失败。为了避免这种失败，实例化模型时必须带有缺失的字段的初始值：

```
author = Author(title='Mr')
form = PartialAuthorForm(request.POST, instance=author)
form.save()

```

还有一种方法，你可以使用`save(commit=False)` 并手工设置额外需要的字段：

```
form = PartialAuthorForm(request.POST)
author = form.save(commit=False)
author.title = 'Mr'
author.save()

```

关于使用`save(commit=False)` 的更多细节参见[保存表单一节](#the-save-method)。

### 重写（覆盖）默认的字段

上文[字段类型](#field-types)表中默认的字段类型只是合理的默认值。如果你的模型中有一个`DateField`，你可能想在表单中也将它表示成`DateField`。但是`ModelForm` 还提供更多的灵活性，让你可以改变给定的模型字段对应的表单字段的类型和Widget。

使用内部类`Meta` 的`widgets` 属性可以指定一个字段的自定义Widget。它是映射字段名到Widget 类或实例的一个字典。

例如，`Author` 的`name` 属性为`CharField`，如果你希望它表示成一个`&lt;textarea&gt;` 而不是默认的`&lt;input type="text"&gt;`，你可以覆盖字段默认的Widget：

```
from django.forms import ModelForm, Textarea
from myapp.models import Author

class AuthorForm(ModelForm):
    class Meta:
        model = Author
        fields = ('name', 'title', 'birth_date')
        widgets = {
            'name': Textarea(attrs={'cols': 80, 'rows': 20}),
        }

```

不管是Widget 实例（`Textarea(...)`）还是Widget 类（`Textarea`），`widgets` 字典都可以接收。

类似地，如果你希望进一步自定义字段，你可以指定内部类`Meta` 的`labels`、`help_texts` 和`error_messages`。

例如，如果你希望自定义`name` 字段所有面向用户的字符串：

```
from django.utils.translation import ugettext_lazy as _

class AuthorForm(ModelForm):
    class Meta:
        model = Author
        fields = ('name', 'title', 'birth_date')
        labels = {
            'name': _('Writer'),
        }
        help_texts = {
            'name': _('Some useful help text.'),
        }
        error_messages = {
            'name': {
                'max_length': _("This writer's name is too long."),
            },
        }

```

最后，如果你希望完全控制字段 —— 包括它的类型、验证器等等，你可以像在普通的`表单`那样显式指定字段。

例如，如果你想为`slug` 字段使用`MySlugFormField` ，可以像下面这样：

```
from django.forms import ModelForm
from myapp.models import Article

class ArticleForm(ModelForm):
    slug = MySlugFormField()

    class Meta:
        model = Article
        fields = ['pub_date', 'headline', 'content', 'reporter', 'slug']

```

如果想要指定字段的验证器，可以显式定义字段并设置它的`validators` 参数：

```
from django.forms import ModelForm, CharField
from myapp.models import Article

class ArticleForm(ModelForm):
    slug = CharField(validators=[validate_slug])

    class Meta:
        model = Article
        fields = ['pub_date', 'headline', 'content', 'reporter', 'slug']

```

注

当你像这样显式实例化表单字段时，需要理解`ModelForm` 和普通的`Form` 的关系是怎样的。

`ModelForm`就是可以自动生产相应字段的`Form`.自动生成哪些字段取决于`Meta` 类的fields属性和在该ModelForm中显示声明的字段。`ModelForm` 基本上**只** 生成表单中**没有的**字段，换句话讲就是没有显式定义的字段。

显式定义的字段会保持原样，所以`Meta` 属性中任何自定义的属性例如 `widgets`、`labels`、`help_texts`或`error_messages` 都将忽略；它们只适用于自动生成的字段。

类似地，显式定义的字段不会从对应的模型中获取属性，例如 `max_length` 或`required`。 如果你希望保持模型中指定的行为，你必须设置在声明表单字段时显式设置相关的参数。

例如，如果`Article` 模型像下面这样：

```
class Article(models.Model):
    headline = models.CharField(max_length=200, null=True, blank=True,
                                help_text="Use puns liberally")
    content = models.TextField()

```

而你想为`headline` 做一些自定义的验证，在保持`blank` 和`help_text` 值的同时，你必须这样定义`ArticleForm`：

```
class ArticleForm(ModelForm):
    headline = MyFormField(max_length=200, required=False,
                           help_text="Use puns liberally")

    class Meta:
        model = Article
        fields = ['headline', 'content']

```

你必须保证表单字段的类型可以用于对应的模型字段。如果它们不兼容，因为不会有显示的转换你将会得到一个`ValueError`。

关于字段和它们的参数，参见[_表单字段的文档_](../../ref/forms/fields.html)。

### 启用字段的本地化功能

默认情况下，`ModelForm` 中的字段不会本地化它们的数据。你可以使用`Meta` 类的`localized_fields` 属性来启用字段的本地化功能。

```
>>> from django.forms import ModelForm
>>> from myapp.models import Author
>>> class AuthorForm(ModelForm):
...     class Meta:
...         model = Author
...         localized_fields = ('birth_date',)

```

如果`localized_fields` 设置为`'__all__'` 这个特殊的值，所有的字段都将本地化。

### 表单继承

在基本的表单里，你可以通过继承`ModelForms`来扩展和重用他们。当你的form是通过models生成的，而且需要在父类的基础上声明额外的field和method，这种继承是方便的。例如，使用以前的`ArticleForm` 类：

```
>>> class EnhancedArticleForm(ArticleForm):
...     def clean_pub_date(self):
...         ...

```

以上创建了一个与 `ArticleForm`非常类似的form，除了一些额外的验证和`pub_date` 的cleaning

你也可以在子类中继承父类的内部类 `Meta`来重写它的属性列表，比如 `Meta.fields` 或者`Meta.excludes` :

```
>>> class RestrictedArticleForm(EnhancedArticleForm):
...     class Meta(ArticleForm.Meta):
...         exclude = ('body',)

```

上例从父类`EnhancedArticleForm`继承后增加了额外的方法，并修改了 `ArticleForm.Meta` 排除了一个字段

当然，有一些注意事项

*   应用正常的Python名称解析规则。如果你有多个基类声明一个`Meta`内部类，只会使用第一个。这意味着孩子的`Meta`（如果存在），否则第一个父母的`Meta`等。

Changed in Django 1.7.

*   它可以同时继承`Form`和`ModelForm`，但是，必须确保`ModelForm`首先出现在MRO中。这是因为这些类依赖于不同的元类，而一个类只能有一个元类。

New in Django 1.7.

*   可以通过在子类上将名称设置为`None`，声明性地删除从父类继承的`Field`。

    您只能使用此技术选择退出由父类声明定义的字段；它不会阻止`ModelForm`元类生成默认字段。要退出默认字段，请参阅[_Selecting the fields to use_](#modelforms-selecting-fields)。

### 提供初始值

作为一个有参数的表单, 在实例化一个表单时可以通过指定`initial`字段来指定表单中数据的初始值. 这种方式指定的初始值将会同时替换掉表单中的字段和值. 例如:

```
>>> article = Article.objects.get(pk=1)
>>> article.headline
'My headline'
>>> form = ArticleForm(initial={'headline': 'Initial headline'}, instance=article)
>>> form['headline'].value()
'Initial headline'

```

### 模型表单的factory函数

你可以用单独的函数 [`modelform_factory()`](../../ref/forms/models.html#django.forms.models.modelform_factory "django.forms.models.modelform_factory") 来代替使用类定义来从模型直接创建表单。这在不需要很多自定义的情况下应该是更方便的。

```
>>> from django.forms.models import modelform_factory
>>> from myapp.models import Book
>>> BookForm = modelform_factory(Book, fields=("author", "title"))

```

这个函数还能对已有的表单类做简单的修改，比如，对给出的字段指定 widgets ：

```
>>> from django.forms import Textarea
>>> Form = modelform_factory(Book, form=BookForm,
...                          widgets={"title": Textarea()})

```

表单包含的字段可以用 `fields`或`exclude`关键字参数说明，或者用`ModelForm`内部`Meta`类的相应属性说明。请看 `ModelForm`文档： [_选择使用的字段_](#modelforms-selecting-fields)。

... 或者为指定的字段启用本地化功能。

```
>>> Form = modelform_factory(Author, form=AuthorForm, localized_fields=("birth_date",))

```

## 模型表单集

_class_ `models.``BaseModelFormSet`

与[_普通表单集_](formsets.html)一样, 它是Django提供的几个有力的表单集类来简化模型操作。. 让我们继续使用上面的`Author`模型：

```
>>> from django.forms.models import modelformset_factory
>>> from myapp.models import Author
>>> AuthorFormSet = modelformset_factory(Author, fields=('name', 'title'))

```

使用 `fields`限定表单集仅可以使用给出的字段，或者使用排除法，指定哪些字段被不被使用。

```
>>> AuthorFormSet = modelformset_factory(Author, exclude=('birth_date',))

```

Changed in Django 1.8:

在旧版本中，同时省略`fields` 和`exclude` 的结果是表单集使用模型的所有字段。现在这么做将引发[`ImproperlyConfigured`](../../ref/exceptions.html#django.core.exceptions.ImproperlyConfigured "django.core.exceptions.ImproperlyConfigured") 异常。

下面将创建一个与`Author` 模型数据相关联的功能强大的表单集，与普通表单集运行一样：

```
>>> formset = AuthorFormSet()
>>> print(formset)
<input type="hidden" name="form-TOTAL_FORMS" value="1" id="id_form-TOTAL_FORMS" /><input type="hidden" name="form-INITIAL_FORMS" value="0" id="id_form-INITIAL_FORMS" /><input type="hidden" name="form-MAX_NUM_FORMS" id="id_form-MAX_NUM_FORMS" />
<tr><th><label for="id_form-0-name">Name:</label></th><td><input id="id_form-0-name" type="text" name="form-0-name" maxlength="100" /></td></tr>
<tr><th><label for="id_form-0-title">Title:</label></th><td><select name="form-0-title" id="id_form-0-title">
<option value="" selected="selected">---------</option>
<option value="MR">Mr.</option>
<option value="MRS">Mrs.</option>
<option value="MS">Ms.</option>
</select><input type="hidden" name="form-0-id" id="id_form-0-id" /></td></tr>

```

注意

[`modelformset_factory()`](../../ref/forms/models.html#django.forms.models.modelformset_factory "django.forms.models.modelformset_factory")使用[`formset_factory()`](../../ref/forms/formsets.html#django.forms.formsets.formset_factory "django.forms.formsets.formset_factory") 生成表单集，这意味着模型表单集仅仅是扩展基本表单集，使其能处理模型的信息。

### 更改查询集

默认的, 如果你使用model生成formset，formset会使用一个包含模型全部对象的queryset(例如:`Author.objects.all()`). 你可以使用`queryset`参数重写这一行为：

```
>>> formset = AuthorFormSet(queryset=Author.objects.filter(name__startswith='O'))

```

或者，你可以创建子类设置 `self.queryset` in `__init__`:

```
from django.forms.models import BaseModelFormSet
from myapp.models import Author

class BaseAuthorFormSet(BaseModelFormSet):
    def __init__(self, *args, **kwargs):
        super(BaseAuthorFormSet, self).__init__(*args, **kwargs)
        self.queryset = Author.objects.filter(name__startswith='O')

```

然后，将`BaseAuthorFormSet` 类传给modelformset_factory函数：

```
>>> AuthorFormSet = modelformset_factory(
...     Author, fields=('name', 'title'), formset=BaseAuthorFormSet)

```

如果想返回不包含_任何_已存在模型实例的表单集，可以指定一个空的查询集（QuerySet）：

```
>>> AuthorFormSet(queryset=Author.objects.none())

```

### 更改`form`

默认情况下,当你使用`modelformset_factory`时, [`modelform_factory()`](../../ref/forms/models.html#django.forms.models.modelform_factory "django.forms.models.modelform_factory")将会创建一个模型 通常这有助于指定一个自定义模型表单. 例如,你可以创建一个自定义验证的表单模型

```
class AuthorForm(forms.ModelForm):
    class Meta:
        model = Author
        fields = ('name', 'title')

    def clean_name(self):
        # custom validation for the name field
        ...

```

然后,把你的模型作为参数传递过去

```
AuthorFormSet = modelformset_factory(Author, form=AuthorForm)

```

并不总是需要自定义一个模型表单， `modelformset_factory` 函数有几个参数，可以传给`modelform_factory`，他们的说明如下：

### 指定要在`widgets`中使用的小部件

使用`widgets` 参数，可以用字典值自定义`ModelForm`列出字段的widget类。这与 `widgets`字典在 `ModelForm` 的内部`Meta`类作用式一样。

```
>>> AuthorFormSet = modelformset_factory(
...     Author, fields=('name', 'title'),
...     widgets={'name': Textarea(attrs={'cols': 80, 'rows': 20})})

```

### 使用`localized_fields`为字段启用本地化

使用 `localized_fields`参数，可以使表单中字段启用本地化。

```
>>> AuthorFormSet = modelformset_factory(
...     Author, fields=('name', 'title', 'birth_date'),
...     localized_fields=('birth_date',))

```

如果`localized_fields`设置值为 `'__all__'`，将本地化所有字段。

### 提供初始化数据

与普通表单集一样，[`modelformset_factory()`](../../ref/forms/models.html#django.forms.models.modelformset_factory "django.forms.models.modelformset_factory")能返回初始化的模型表单集，`initial`参数能为表单集的中表单[_指定初始数据_](formsets.html#formsets-initial-data) 。但是，在模型表单集中，初始数据仅应用在增加的表单中，不会应用到已存在的模型实例。如果用户没有更改新增加表单中的初始数据，那他们也不会被校验和保存。

### 保存表单集中的对象

做为 `ModelForm`, 你可以保存数据到模型对象，以下就完成了表单集的 `save()`方法:

```
# Create a formset instance with POST data.
>>> formset = AuthorFormSet(request.POST)

# Assuming all is valid, save the data.
>>> instances = formset.save()

```

`save()`方法返回已保存到数据库的实例。如果给定实例的数据在绑定数据中没有更改，那么实例将不会保存到数据库，并且不会包含在返回值中（在上面的示例中为`instances`）。

当窗体中缺少字段（例如因为它们已被排除）时，这些字段不会由`save()`方法设置。您可以在[选择要使用的字段](#selecting-the-fields-to-use)中找到有关此限制的更多信息，这也适用于常规`ModelForms`。

传递`commit=False`返回未保存的模型实例：

```
# don't save to the database
>>> instances = formset.save(commit=False)
>>> for instance in instances:
...     # do something with instance
...     instance.save()

```

这使您能够在将数据保存到数据库之前将数据附加到实例。如果您的表单集包含`ManyToManyField`，您还需要调用`formset.save_m2m()`，以确保多对多关系正确保存。

调用`save()`之后，您的模型formset将有三个包含formset更改的新属性：

`models.BaseModelFormSet.``changed_objects`

`models.BaseModelFormSet.``deleted_objects`

`models.BaseModelFormSet.``new_objects`

### 操作表单对像的数量限制

与普通表单集一样，你可以用在[`modelformset_factory()`](../../ref/forms/models.html#django.forms.models.modelformset_factory "django.forms.models.modelformset_factory")中使用 `max_num` 和 `extra` 参数，来控制额外表单的显示数量。

`max_num` 不会限制已经存在的表单对像的显示:

```
>>> Author.objects.order_by('name')
[<Author: Charles Baudelaire>, <Author: Paul Verlaine>, <Author: Walt Whitman>]

>>> AuthorFormSet = modelformset_factory(Author, fields=('name',), max_num=1)
>>> formset = AuthorFormSet(queryset=Author.objects.order_by('name'))
>>> [x.name for x in formset.get_queryset()]
['Charles Baudelaire', 'Paul Verlaine', 'Walt Whitman']

```

如果 `max_num`大于存在的关联对像的数量，表单集将添加 `extra`个额外的空白表单，只要表单总数量不超过 `max_num`：

```
>>> AuthorFormSet = modelformset_factory(Author, fields=('name',), max_num=4, extra=2)
>>> formset = AuthorFormSet(queryset=Author.objects.order_by('name'))
>>> for form in formset:
...     print(form.as_table())
<tr><th><label for="id_form-0-name">Name:</label></th><td><input id="id_form-0-name" type="text" name="form-0-name" value="Charles Baudelaire" maxlength="100" /><input type="hidden" name="form-0-id" value="1" id="id_form-0-id" /></td></tr>
<tr><th><label for="id_form-1-name">Name:</label></th><td><input id="id_form-1-name" type="text" name="form-1-name" value="Paul Verlaine" maxlength="100" /><input type="hidden" name="form-1-id" value="3" id="id_form-1-id" /></td></tr>
<tr><th><label for="id_form-2-name">Name:</label></th><td><input id="id_form-2-name" type="text" name="form-2-name" value="Walt Whitman" maxlength="100" /><input type="hidden" name="form-2-id" value="2" id="id_form-2-id" /></td></tr>
<tr><th><label for="id_form-3-name">Name:</label></th><td><input id="id_form-3-name" type="text" name="form-3-name" maxlength="100" /><input type="hidden" name="form-3-id" id="id_form-3-id" /></td></tr>

```

 `max_num` 值为f `None` （缺省）设置一个较高的限制可显示1000个表单。实际上相当于没有限制。

### 在视图中使用模型表单集

模型表单集与表单集十分类似，假设我们想要提供一个表单集来编辑`Author`模型实例：

```
from django.forms.models import modelformset_factory
from django.shortcuts import render_to_response
from myapp.models import Author

def manage_authors(request):
    AuthorFormSet = modelformset_factory(Author, fields=('name', 'title'))
    if request.method == 'POST':
        formset = AuthorFormSet(request.POST, request.FILES)
        if formset.is_valid():
            formset.save()
            # do something.
    else:
        formset = AuthorFormSet()
    return render_to_response("manage_authors.html", {
        "formset": formset,
    })

```

可以看到，模型表单集的视图逻辑与“正常”表单集的视图逻辑没有显着不同。唯一的区别是我们调用`formset.save()`将数据保存到数据库中。（如上所述，在[_Saving objects in the formset_](#saving-objects-in-the-formset)中的对象）。

### 在`ModelFormSet`上覆盖`clean()`

与`ModelForms`一样，默认情况下，`ModelFormSet`的`clean()`方法将验证formset中没有项目违反唯一约束（`unique`，`unique_together`或`unique_for_date|month|year`）。如果要覆盖`ModelFormSet`上的`clean()`方法并维护此验证，则必须调用父类的`clean`方法：

```
from django.forms.models import BaseModelFormSet

class MyModelFormSet(BaseModelFormSet):
    def clean(self):
        super(MyModelFormSet, self).clean()
        # example custom validation across forms in the formset
        for form in self.forms:
            # your custom formset validation
            ...

```

另请注意，到达此步骤时，已为每个`Form`创建了各个模型实例。修改`form.cleaned_data`中的值不足以影响保存的值。如果您希望修改`ModelFormSet.clean()`中的值，则必须修改`form.instance`：

```
from django.forms.models import BaseModelFormSet

class MyModelFormSet(BaseModelFormSet):
    def clean(self):
        super(MyModelFormSet, self).clean()

        for form in self.forms:
            name = form.cleaned_data['name'].upper()
            form.cleaned_data['name'] = name
            # update the instance value.
            form.instance.name = name

```

### 使用自定义queryset

如前所述，您可以覆盖模型formset使用的默认查询集：

```
from django.forms.models import modelformset_factory
from django.shortcuts import render_to_response
from myapp.models import Author

def manage_authors(request):
    AuthorFormSet = modelformset_factory(Author, fields=('name', 'title'))
    if request.method == "POST":
        formset = AuthorFormSet(request.POST, request.FILES,
                                queryset=Author.objects.filter(name__startswith='O'))
        if formset.is_valid():
            formset.save()
            # Do something.
    else:
        formset = AuthorFormSet(queryset=Author.objects.filter(name__startswith='O'))
    return render_to_response("manage_authors.html", {
        "formset": formset,
    })

```

请注意，我们在此示例中的`POST`和`GET`中传递`queryset`参数。

### 在模板中使用表单集

在Django模板中有三种方式来渲染表单集。

第一种方式，你可以让表单集完成大部分的工作

```
<form method="post" action="">
    {{ formset }}
</form>

```

其次，你可以手动渲染formset，但让表单处理自己：

```
<form method="post" action="">
    {{ formset.management_form }}
    {% for form in formset %}
        {{ form }}
    {% endfor %}
</form>

```

当您自己手动呈现表单时，请确保呈现如上所示的管理表单。请参阅[_management form documentation_](formsets.html#understanding-the-managementform)。

第三，您可以手动呈现每个字段：

```
<form method="post" action="">
    {{ formset.management_form }}
    {% for form in formset %}
        {% for field in form %}
            {{ field.label_tag }} {{ field }}
        {% endfor %}
    {% endfor %}
</form>

```

如果您选择使用此第三种方法，并且不对`{％ for ％} t0&gt; loop，你需要渲染主键字段。`例如，如果您要渲染模型的`name`和`age`字段：

```
<form method="post" action="">
    {{ formset.management_form }}
    {% for form in formset %}
        {{ form.id }}
        <ul>
            <li>{{ form.name }}</li>
            <li>{{ form.age }}</li>
        </ul>
    {% endfor %}
</form>

```

注意我们需要如何显式渲染`{{ form.id }}`。这确保了在`POST`情况下的模型形式集将正常工作。（此示例假设名为`id`的主键。如果您明确定义了自己的主键（不是`id`），请确保其呈现）。

## 内联formets

_class_ `models.``BaseInlineFormSet`

内联formets是模型formets上的一个小的抽象层。这些简化了通过外键处理相关对象的情况。假设你有这两个模型：

```
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    author = models.ForeignKey(Author)
    title = models.CharField(max_length=100)

```

如果要创建允许您编辑属于特定作者的图书的表单集，您可以执行以下操作：

```
>>> from django.forms.models import inlineformset_factory
>>> BookFormSet = inlineformset_factory(Author, Book, fields=('title',))
>>> author = Author.objects.get(name='Mike Royko')
>>> formset = BookFormSet(instance=author)

```

注意

[`inlineformset_factory()`](../../ref/forms/models.html#django.forms.models.inlineformset_factory "django.forms.models.inlineformset_factory")使用[`modelformset_factory()`](../../ref/forms/models.html#django.forms.models.modelformset_factory "django.forms.models.modelformset_factory")并标记为`can_delete=True`。

也可以看看

[_Manually rendered can_delete and can_order_](formsets.html#manually-rendered-can-delete-and-can-order)。

### 覆盖`InlineFormSet`上的方法

当覆盖`InlineFormSet`上的方法时，您应该子类化[`BaseInlineFormSet`](#django.forms.models.BaseInlineFormSet "django.forms.models.BaseInlineFormSet")，而不是[`BaseModelFormSet`](#django.forms.models.BaseModelFormSet "django.forms.models.BaseModelFormSet")。

例如，如果要覆盖`clean()`：

```
from django.forms.models import BaseInlineFormSet

class CustomInlineFormSet(BaseInlineFormSet):
    def clean(self):
        super(CustomInlineFormSet, self).clean()
        # example custom validation across forms in the formset
        for form in self.forms:
            # your custom formset validation
            ...

```

另请参见[_Overriding clean() on a ModelFormSet_](#model-formsets-overriding-clean)上的clean()。

然后，在创建内联表单集时，传递可选参数`formset`：

```
>>> from django.forms.models import inlineformset_factory
>>> BookFormSet = inlineformset_factory(Author, Book, fields=('title',),
...     formset=CustomInlineFormSet)
>>> author = Author.objects.get(name='Mike Royko')
>>> formset = BookFormSet(instance=author)

```

### 多个外键对同一个模型

如果您的模型在同一个模型中包含多个外键，则需要使用`fk_name`手动解决歧义。例如，考虑以下模型：

```
class Friendship(models.Model):
    from_friend = models.ForeignKey(Friend, related_name='from_friends')
    to_friend = models.ForeignKey(Friend, related_name='friends')
    length_in_months = models.IntegerField()

```

要解决此问题，您可以使用`fk_name`到[`inlineformset_factory()`](../../ref/forms/models.html#django.forms.models.inlineformset_factory "django.forms.models.inlineformset_factory")：

```
>>> FriendshipFormSet = inlineformset_factory(Friend, Friendship, fk_name='from_friend',
...     fields=('to_friend', 'length_in_months'))

```

### 在视图中使用内联格式集

您可能需要提供一个视图，允许用户编辑模型的相关对象。以下是如何做到这一点：

```
def manage_books(request, author_id):
    author = Author.objects.get(pk=author_id)
    BookInlineFormSet = inlineformset_factory(Author, Book, fields=('title',))
    if request.method == "POST":
        formset = BookInlineFormSet(request.POST, request.FILES, instance=author)
        if formset.is_valid():
            formset.save()
            # Do something. Should generally end with a redirect. For example:
            return HttpResponseRedirect(author.get_absolute_url())
    else:
        formset = BookInlineFormSet(instance=author)
    return render_to_response("manage_books.html", {
        "formset": formset,
    })

```

注意我们如何在`POST`和`GET`例中传递`instance`。

### 指定要在内联表单中使用的窗口小部件

`inlineformset_factory`使用`modelformset_factory`并将其大部分参数传递给`modelformset_factory`。这意味着您可以使用`widgets`参数，将其传递到`modelformset_factory`。请参阅上面的[指定要在窗体中使用的窗口小部件](#specifying-widgets-to-use-in-the-form-with-widgets)的窗口小部件。
