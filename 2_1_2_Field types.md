



# 模型字段参考

本文档包含了Django提供的全部模型[`字段`](#django.db.models.Field "django.db.models.Field")的[字段选项](#field-options) 和 [字段类型](#field-types)的API参考。



请看:

如果内建的字段不能满足你的需要，你可以尝试包含对特定国家和文化有帮助的配套代码的 [django-localflavor](https://django-localflavor.readthedocs.org/)。当然，你也可以很容易的编写你[_自定义的字段_](../../howto/custom-model-fields.html)。





注意

严格意义上来讲， Model 是定义在[`django.db.models.fields`](#module-django.db.models.fields "django.db.models.fields: Built-in field types.")里面，但为了使用方便，它们被导入到 [`django.db.models`](../../topics/db/models.html#module-django.db.models "django.db.models")中；标准上，我们导入`from django.db import models`，然后使用?`models.<Foo>Field`的形式使用字段。




## 字段选项(Field options)

下列参数是全部字段类型都可用的，而且都是可选择的。



### null



`Field.null`



如果为`True`，Django 将空值以`NULL` 存储到数据库中。默认值是 `False`。

字符串字段例如[`CharField`](#django.db.models.CharField "django.db.models.CharField") 和[`TextField`](#django.db.models.TextField "django.db.models.TextField") 要避免使用[`null`](#django.db.models.Field.null "django.db.models.Field.null")，因为空字符串值将始终储存为空字符串而不是`NULL`。如果字符串字段的`null=True`，那意味着对于“无数据”有两个可能的值：`NULL` 和空字符串。在大多数情况下，对于“无数据”声明两个值是赘余的，Django 的惯例是使用空字符串而不是`NULL`。

无论是字符串字段还是非字符串字段，如果你希望在表单中允许空值，你将还需要设置`blank=True`，因为[`null`](#django.db.models.Field.null "django.db.models.Field.null") 仅仅影响数据库存储（参见[`blank`](#django.db.models.Field.blank "django.db.models.Field.blank")）。



注意

在使用Oracle 数据库时，数据库将存储`NULL` 来表示空字符串，而与这个属性无关。



如果你希望[`BooleanField`](#django.db.models.BooleanField "django.db.models.BooleanField") 接受[`null`](#django.db.models.Field.null "django.db.models.Field.null") 值，请用 [`NullBooleanField`](#django.db.models.NullBooleanField "django.db.models.NullBooleanField") 代替。





### blank



`Field.blank`



如果为`True`，则该字段允许为空白。 默认值是 `False`。

注意它与[`null`](#django.db.models.Field.null "django.db.models.Field.null")不同。[`null`](#django.db.models.Field.null "django.db.models.Field.null") 纯粹是数据库范畴的概念，而[`blank`](#django.db.models.Field.blank "django.db.models.Field.blank") 是数据验证范畴的。如果字段设置`blank=True`，表单验证时将允许输入空值。如果字段设置`blank=False`，则该字段为必填。





### choices



`Field.choices`



它是一个可迭代的结构(比如，列表或是元组)，由可迭代的二元组组成(比如`[(A, B), (A, B) ...]`)，用来给这个字段提供选择项。如果设置了 choices ，默认表格样式就会显示选择框，而不是标准的文本框，而且这个选择框的选项就是 choices 中的元组。

每个元组中的第一个元素，是存储在数据库中的值；第二个元素是该选项更易理解的描述。 比如:





```
YEAR_IN_SCHOOL_CHOICES = (
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
)

```





一般来说，最好在模型类内部定义choices，然后再给每个值定义一个合适名字的常量。





```
from django.db import models

class Student(models.Model):
    FRESHMAN = 'FR'
    SOPHOMORE = 'SO'
    JUNIOR = 'JR'
    SENIOR = 'SR'
    YEAR_IN_SCHOOL_CHOICES = (
        (FRESHMAN, 'Freshman'),
        (SOPHOMORE, 'Sophomore'),
        (JUNIOR, 'Junior'),
        (SENIOR, 'Senior'),
    )
    year_in_school = models.CharField(max_length=2,
                                      choices=YEAR_IN_SCHOOL_CHOICES,
                                      default=FRESHMAN)

    def is_upperclass(self):
        return self.year_in_school in (self.JUNIOR, self.SENIOR)

```





尽管你可以在模型类的外部定义choices然后引用它，但是在模型类中定义choices和其每个choice的name(即元组的第二个元素)可以保存所有使用choices的类的信息， 也使得choices更容易被应用（例如， `Student.SOPHOMORE` 可以在任何引入`Student` 模型的位置生效)。

你也可以归类可选的choices到已命名的组中用来达成组织整理的目的:





```
MEDIA_CHOICES = (
    ('Audio', (
            ('vinyl', 'Vinyl'),
            ('cd', 'CD'),
        )
    ),
    ('Video', (
            ('vhs', 'VHS Tape'),
            ('dvd', 'DVD'),
        )
    ),
    ('unknown', 'Unknown'),
)

```





每个元组的第一个元素是组的名字。第二个元素是一组可迭代的二元元组，每一个二元元组包含一个值和一个给人看的名字构成一个选项。分组的选项可能会和未分组的选项合在同一个list中。 （就像例中的<cite>unknown</cite>选项）。

对于有[`choices`](#django.db.models.Field.choices "django.db.models.Field.choices") set的模型字段, Django 将会加入一个方法来获取当前字段值的易于理解的名称(即元组的第二个值)参见数据库API文档中的[`get_FOO_display()`](instances.html#django.db.models.Model.get_FOO_display "django.db.models.Model.get_FOO_display")。

请注意choices可以是任何可迭代的对象 – 不是必须是列表或者元组。这一点使你可以动态的构建choices。但是如果你发现你自己搞不定动态的[`choices`](#django.db.models.Field.choices "django.db.models.Field.choices")，你最好还是使用[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey")来构建一个合适的数据库表。如果有数据变动的话，[`choices`](#django.db.models.Field.choices "django.db.models.Field.choices")意味着那些变动不多的静态数据。

New in Django 1.7.

除非[`blank=False`](#django.db.models.Field.blank "django.db.models.Field.blank") 和[`default`](#django.db.models.Field.default "django.db.models.Field.default")一起在字段中被设置，否则，可选择菜单将会有`"---------"` 的标签。要重写这个行为, 需要加入一个包含`None`的元组到 `choices`里面; 例如 `(None, 'Your String For Display')`. 或者, 你可以在操作有意义的地方用一个空字符串代替`None` ?- 比如在一个 [`CharField`](#django.db.models.CharField "django.db.models.CharField").





### db_column



`Field.db_column`



数据库中用来表示该字段的名称。如果未指定，那么Django将会使用Field名作为字段名.

如果你的数据库列名为SQL语句的保留字，或者是包含不能作为Python 变量名的字符，如连字符，没问题。Django 会在后台给列名和表名加上双引号。





### db_index



`Field.db_index`



若值为 `True`, 则 [`django-admin sqlindexes`](../django-admin.html#django-admin-sqlindexes) 将会为此字段输出 `CREATE INDEX` 语句。（译注：为此字段创建索引）





### db_tablespace



`Field.db_tablespace`



?如果该字段有索引的话，[_database tablespace_](../../topics/db/tablespaces.html)的名称将作为该字段的索引名。 如果[`DEFAULT_INDEX_TABLESPACE`](../settings.html#std:setting-DEFAULT_INDEX_TABLESPACE) 已经设置，则默认值是由DEFAULT_INDEX_TABLESPACE指定, 如果没有设置则由 [`db_tablespace`](options.html#django.db.models.Options.db_tablespace "django.db.models.Options.db_tablespace") 指定，如果后台数据库不支持表空间，或者索引，则该选项被忽略





### default



`Field.default`



该字段的默认值. 它可以是一个值或者一个可调用对象. 如果是一个可调用对象，那么在每一次创建新对象的时候，它将会调用一次.

这个默认值不可以是一个可变对象（如字典，列表，等等）,因为对于所有模型的一个新的实例来说，它们指向同一个引用。或者，把他们包装为一个可调用的对象。例如，你有一个自定义的`JSONField`，并且想指定一个特定的字典值，可以如下使用：





```
def contact_default():
    return {"email": "to1@example.com"}

contact_info = JSONField("ContactInfo", default=contact_default)

```





请注意`lambda`s 函数不可作为如 `default` 这类可选参数的值.因为它们无法被 [_migrations命令序列化_](../../topics/migrations.html#migration-serializing). 请参见文档其他部分。

默认值会在新实例创建并且没有给该字段提供值时使用。如果字段为主键，默认值也会在设置为`None`时使用。

Changed in Django 1.8:

之前的版本不会使用`None`作为主键?







### editable



`Field.editable`



如果设为`False`, 这个字段将不会出现在 admin 或者其他 [`ModelForm`](../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm"). 他们也会跳过 [_模型验证_](instances.html#validating-objects). 默认是`True`.





### error_messages



`Field.error_messages`



`error_messages` 参数能够让你重写默认抛出的错误信息。通过指定 key 来确认你要重写的错误信息。

error_messages 的 key 值包括 `null`, `blank`, `invalid`, `invalid_choice`, `unique`, 和 `unique_for_date`. 其余的 error_messages 的 keys 是不一样的在不同的章节下 [Field types](#field-types) 。

New in Django 1.7.

这个 `unique_for_date` 的 error_messages 的key 是在 1.7 中加的。





### help_text



`Field.help_text`



额外的 ‘help' 文本将被显示在表单控件form中。即便你的字段没有应用到一个form里面，这样的操作对文档化也很有帮助。

注意这 _不_ 会自动添加 HTML 标签。需要你在 [`help_text`](#django.db.models.Field.help_text "django.db.models.Field.help_text") 包含自己需要的格式。例如:





```
help_text="Please use the following format: <em>YYYY-MM-DD</em>."

```





另外, 你可以使用简单文本和`django.utils.html.escape()`以避免任何HTML特定的字符.请确保你所使用的help text能够避免那些由不受信任的用户进行的跨站点脚本攻击。





### primary_key



`Field.primary_key`



若为 `True`, 则该字段会成为模型的主键字段。

如果你没有在模型的任何字段上指定 `primary_key=True`, Django会自动添加一个 [`AutoField`](#django.db.models.AutoField "django.db.models.AutoField") 字段来充当主键。 所以除非你想要覆盖默认的主键行为，否则不需要在任何字段上设定`primary_key=True` 。更多内容 请参考 [_Automatic primary key fields_](../../topics/db/models.html#automatic-primary-key-fields).

`primary_key=True` 暗含着[`null=False`](#django.db.models.Field.null "django.db.models.Field.null") 和[`unique=True`](#django.db.models.Field.unique "django.db.models.Field.unique"). 一个对象上只能拥有一个主键.

主键字段是只读的。如果你改变了一个已存在对象上的主键并且保存的话，会创建一个新的对象，而不是覆盖旧的.





### unique



`Field.unique`



如果为?`True`, 这个字段在表中必须有唯一值.

这是一个在数据库级别的强制性动作,并且通过模型来验证。如果你试图用一个重复的值来保存[`unique`](#django.db.models.Field.unique "django.db.models.Field.unique") 字段，那么一个[`django.db.IntegrityError`](../exceptions.html#django.db.IntegrityError "django.db.IntegrityError")就会通过模型的[`save()`](instances.html#django.db.models.Model.save "django.db.models.Model.save") 函数抛出来。

除了[`ManyToManyField`](#django.db.models.ManyToManyField "django.db.models.ManyToManyField")、[`OneToOneField`](#django.db.models.OneToOneField "django.db.models.OneToOneField")和[`FileField`](#django.db.models.FileField "django.db.models.FileField") 以外的其他字段类型都可以使用这个设置。

注意当设置`unique` 为`True` 时，你不需要再指定 [`db_index`](#django.db.models.Field.db_index "django.db.models.Field.db_index")，因为`unique` 本身就意味着一个索引的创建。





### unique_for_date



`Field.unique_for_date`



当设置它为[`DateField`](#django.db.models.DateField "django.db.models.DateField") 和[`DateTimeField`](#django.db.models.DateTimeField "django.db.models.DateTimeField") 字段的名称时，表示要求该字段对于相应的日期字段值是唯一的。

例如，你有一个`title` 字段设置`unique_for_date="pub_date"`，那么Django 将不允许两个记录具有相同的`title` 和`pub_date`。

注意，如果你将它设置为[`DateTimeField`](#django.db.models.DateTimeField "django.db.models.DateTimeField")，只会考虑其日期部分。此外，如果[`USE_TZ`](../settings.html#std:setting-USE_TZ) 为`True`，检查将以对象保存时的[_当前的时区_](../../topics/i18n/timezones.html#default-current-time-zone) 进行。

这是在模型验证期间通过[`Model.validate_unique()`](instances.html#django.db.models.Model.validate_unique "django.db.models.Model.validate_unique") 强制执行的，而不是在数据库层级进行验证。如果[`unique_for_date`](#django.db.models.Field.unique_for_date "django.db.models.Field.unique_for_date") 约束涉及的字段不是[`ModelForm`](../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")中的字段（例如，`exclude`中列出的字段或者设置了[`editable=False`](#django.db.models.Field.editable "django.db.models.Field.editable")），[`Model.validate_unique()`](instances.html#django.db.models.Model.validate_unique "django.db.models.Model.validate_unique") 将忽略该特殊的约束。





### unique_for_month



`Field.unique_for_month`



类似[`unique_for_date`](#django.db.models.Field.unique_for_date "django.db.models.Field.unique_for_date")，只是要求字段对于月份是唯一的。





### unique_for_year



`Field.unique_for_year`



类似[`unique_for_date`](#django.db.models.Field.unique_for_date "django.db.models.Field.unique_for_date") 和 [`unique_for_month`](#django.db.models.Field.unique_for_month "django.db.models.Field.unique_for_month")。





### verbose_name



`Field.verbose_name`



一个字段的可读性更高的名称。如果用户没有设定冗余名称字段，Django会自动将该字段属性名中的下划线转换为空格，并用它来创建冗余名称。可以参照 [_Verbose field names_](../../topics/db/models.html#verbose-field-names).





### validators



`Field.validators`



该字段将要运行的一个Validator 的列表。更多信息参见[_Validators 的文档_](../validators.html)。



#### 注册和获取查询

`Field` 实现了[_查询注册API_](lookups.html#lookup-registration-api)。该API 可以用于自定义一个字段类型的可用的查询，以及如何从一个字段获取查询。









## 字段类型 （Field types）



### 自增字段



_class_ `AutoField`(_**options_)



一个根据实际ID自动增长的[`IntegerField`](#django.db.models.IntegerField "django.db.models.IntegerField") . 你通常不需要直接使用;如果不指定,一个主键字段将自动添加到你创建的模型中.详细查看?[_主键字段_](../../topics/db/models.html#automatic-primary-key-fields).





### BigIntegerField



_class_ `BigIntegerField`([_**options_])



一个64位整数, 类似于一个?[`IntegerField`](#django.db.models.IntegerField "django.db.models.IntegerField") ,它的值的范围是?`-9223372036854775808` 到`9223372036854775807`之间. 这个字段默认的表单组件是一个[`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput").





### BinaryField



_class_ `BinaryField`([_**options_])



这是一个用来存储原始二进制码的Field. 只支持`bytes` 赋值，注意这个Field只有很有限的功能。例如，不大可能在一个`BinaryField` 值的数据上进行查询



Abusing `BinaryField`

尽管你可能想使用数据库来存储你的文件，但是99%的情况下这都是不好的设计。这个字段 _不是_替代[_static files_](../../howto/static-files/index.html) 的合理操作.







### BooleanField



_class_ `BooleanField`(_**options_)



true/false 字段。

此字段的默认表单挂件是一个[`CheckboxInput`](../forms/widgets.html#django.forms.CheckboxInput "django.forms.CheckboxInput").

如果你需要设置[`null`](#django.db.models.Field.null "django.db.models.Field.null") 值，则使用[`NullBooleanField`](#django.db.models.NullBooleanField "django.db.models.NullBooleanField") 来代替BooleanField。

如果[`Field.default`](#django.db.models.Field.default "django.db.models.Field.default")没有指定的话， `BooleanField` 的默认值是 `None`。





### CharField



_class_ `CharField`(_max_length=None_[, _**options_])



一个用来存储从小到很大各种长度的字符串的地方

如果是巨大的文本类型, 可以用 [`TextField`](#django.db.models.TextField "django.db.models.TextField").

这个字段默认的表单样式是 [`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput").

[`CharField`](#django.db.models.CharField "django.db.models.CharField")必须接收一个额外的参数:



`CharField.max_length`



字段的最大字符长度.max_length将在数据库层和Django表单验证中起作用, 用来限定字段的长度.?







Note

如果你在写一个需要导出到多个不同数据库后端的应用，你需要注意某些后端对`max_length`有一些限制，查看[_数据库后端注意事项_](../databases.html)来获取更多的细节





MySQL的使用者们:

如果你在使用该字段的同时使用MySQLdb 1.2.2以及 `utf8_bin` 校对 其一般_不是_ 默认情况), 你必须了解一些问题. 详细请参考 [_MySQL database notes_](../databases.html#mysql-collation) .







### CommaSeparatedIntegerField



_class_ `CommaSeparatedIntegerField`(_max_length=None_[, _**options_])



一个逗号分隔的整数字段。像 [`CharField`](#django.db.models.CharField "django.db.models.CharField")一样， 需要一个[`max_length`](#django.db.models.CharField.max_length "django.db.models.CharField.max_length") 参数， 同时数据库移植时也需要注意。





### DateField



_class_ `DateField`([_auto_now=False_, _auto_now_add=False_, _**options_])



这是一个使用Python的`datetime.date`实例表示的日期. 有几个额外的设置参数:



`DateField.auto_now`



每次保存对象时，自动设置该字段为当前时间。用于"最后一次修改"的时间戳。注意，它_总是_使用当前日期；和你可以覆盖的那种默认值不一样。







`DateField.auto_now_add`



当对象第一次被创建时自动设置当前时间。用于创建时间的时间戳. 它_总是_使用当前日期；和你可以覆盖的那种默认值不一样。





该字段默认对应的表单控件是一个[`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput"). 在管理员站点添加了一个JavaScript写的日历控件，和一个“Today"的快捷按钮.包含了一个额外的`invalid_date`错误消息键.

`auto_now_add`, `auto_now`, and `default` 这些设置是相互排斥的. 他们之间的任何组合将会发生错误的结果.



Note

在目前的实现中，设置`auto_now`或者`auto_now_add`为`True`将为让这个字段同时得到`editable=False`和`blank=True`这两个设置.





Note

`auto_now` and `auto_now_add`这两个设置会在对象创建或更新的时刻,总是使用[_default timezone_](../../topics/i18n/timezones.html#default-current-time-zone)(默认时区)的日期. 如果你不想这样，你可以考虑一下简单地使用你自己的默认调用或者重写`save()`(在save()函数里自己添加保存时间的机制.译者注)而不是使用`auto_now` or `auto_now_add`; 或者使用`DateTimeField`字段类来替换`DateField` 并且在给用户呈现时间的时候,决定如何处理从datetime到date的转换.







### DateTimeField



_class_ `DateTimeField`([_auto_now=False_, _auto_now_add=False_, _**options_])



它是通过Python `datetime.datetime`实例表示的日期和时间. 携带了跟[`DateField`](#django.db.models.DateField "django.db.models.DateField")一样的额外参数.

该字段默认对应的表单控件是一个单个的[`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput")(单文本输入框). 管理界面是使用两个带有 JavaScript控件的?[`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput")?文本框.





### DecimalField



_class_ `DecimalField`(_max_digits=None_, _decimal_places=None_[, _**options_])



用python中 [`Decimal`](https://docs.python.org/3/library/decimal.html#decimal.Decimal "(in Python v3.4)") 的一个实例来表示十进制浮点数. 有两个 **必须的**参数:



`DecimalField.max_digits`



位数总数，包括小数点后的位数。该值必须大于等于`decimal_places`.







`DecimalField.decimal_places`



小数点后的数字数量





例如,要保存最大为 `999` 并有两位小数的数字,你应该使用:





```
models.DecimalField(..., max_digits=5, decimal_places=2)

```





而要存储那些将近10亿，并且要求达到小数点后十位精度的数字:





```
models.DecimalField(..., max_digits=19, decimal_places=10)

```





该字段默认的窗体组件是 [`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput").



Note

想获取更多关于 [`FloatField`](#django.db.models.FloatField "django.db.models.FloatField") 和 [`DecimalField`](#django.db.models.DecimalField "django.db.models.DecimalField") 差异, 请参照 [_FloatField vs. DecimalField_](#floatfield-vs-decimalfield).







### DurationField

New in Django 1.8.



_class_ `DurationField`([_**options_])



用作存储一段时间的字段类型 - 类似Python中的[`timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta "(in Python v3.4)"). 当数据库使用的是PostgreSQL, 该数据类型使用的是一个 `interval` 而在Oracle上，则使用的是 `INTERVAL DAY(9) TO SECOND(6)`. Otherwise a `bigint` of microseconds is used.



注意

`DurationField` 的算数运算在多数情况下可以很好的工作。?然而，在除了PostgreSQL之外的其他数据库中, 将 `DurationField` 与 `DateTimeField` 的实例比较则不会得到正确的结果。







### EmailField



_class_ `EmailField`([_max_length=254_, _**options_])



一个 [`CharField`](#django.db.models.CharField "django.db.models.CharField") 用来检查输入的email地址是否合法。它使用 [`EmailValidator`](../validators.html#django.core.validators.EmailValidator "django.core.validators.EmailValidator") 来验证输入合法性。

Changed in Django 1.8:

默认最大长度 `max_length` 从75增加到254以符合RFC3696/5321标准。







### FileField



_class_ `FileField`([_upload_to=None_, _max_length=100_, _**options_])



一个上传文件的字段。



注意

FileField字段不支持`primary_key` 和`unique`参数，如果使用会生成?`TypeError`错误



有两个可选参数：



`FileField.upload_to`



Changed in Django 1.7:

在旧版本Django中，`upload_to`?属性是必须要有的;



一个本地文件系统的路径，它将附加到[`MEDIA_ROOT`](../settings.html#std:setting-MEDIA_ROOT) 设置的后面来确定[`url`](#django.db.models.fields.files.FieldFile.url "django.db.models.fields.files.FieldFile.url") 属性的值。

这个路径可能会包含一个 [`strftime()`](https://docs.python.org/3/library/time.html#time.strftime "(in Python v3.4)") 格式串,并且会在文件上传时被替换为 实际的date/time作为文件路径 (这样上传的文件就不会塞满你指定的文件夹了).

它还可以是一个可调用对象如函数，将调用它来获取上传路径，包括文件名。这个可调用对象必须接受两个参数，并且返回一个Unix 风格的路径（带有前向/）给存储系统。将传递的两个参数为：

| Argument | Description |
| --- | --- |
| `instance` | `FileField` 定义所在的模型的实例。更准确地说，就是当前文件的所在的那个实例。大部分情况下，这个实例将还没有保存到数据库中，若它用到默认的`AutoField` 字段，_它的主键字段还可能没有值_。 |
| `filename` | The filename that was originally given to the file. This may or may not be taken into account when determining the final destination path. |







`FileField.storage`



一个Storage 对象，用于你的文件的存取。参见[_管理文件_](../../topics/files.html) 获取如何提供这个对象的细节。





这个字段的默认表单Widget 是[`ClearableFileInput`](../forms/widgets.html#django.forms.ClearableFileInput "django.forms.ClearableFileInput")。

在模型中调用[`FileField`](#django.db.models.FileField "django.db.models.FileField") 或 [`ImageField`](#django.db.models.ImageField "django.db.models.ImageField") (见下方) 需如下几步：

1.  在你的settings文件中, 你必须要定义 [`MEDIA_ROOT`](../settings.html#std:setting-MEDIA_ROOT) 作为Django存储上传文件的路径(从性能上考虑，这些文件不能存在数据库中。) 定义一个 [`MEDIA_URL`](../settings.html#std:setting-MEDIA_URL) 作为基础的URL或者目录。确保这个目录可以被web server使用的账户写入。
2.  在模型中添加[`FileField`](#django.db.models.FileField "django.db.models.FileField") 或 [`ImageField`](#django.db.models.ImageField "django.db.models.ImageField") 字段, 定义 [`upload_to`](#django.db.models.FileField.upload_to "django.db.models.FileField.upload_to")参数，内容是 [`MEDIA_ROOT`](../settings.html#std:setting-MEDIA_ROOT) 的子目录，用来存放上传的文件。
3.  数据库中存放的仅是这个文件的路径 （相对于[`MEDIA_ROOT`](../settings.html#std:setting-MEDIA_ROOT)). 你很可能会想用由Django提供的便利的[`url`](#django.db.models.fields.files.FieldFile.url "django.db.models.fields.files.FieldFile.url") 属性。比如说, 如果你的[`ImageField`](#django.db.models.ImageField "django.db.models.ImageField") 命名为 `mug_shot`, 你可以在template中用 `{{ object.mug_shot.url }}`获得你照片的绝对路径。

例如，如果你的 [`MEDIA_ROOT`](../settings.html#std:setting-MEDIA_ROOT)设定为 `'/home/media'`，并且 [`upload_to`](#django.db.models.FileField.upload_to "django.db.models.FileField.upload_to")设定为 `'photos/%Y/%m/%d'`。 [`upload_to`](#django.db.models.FileField.upload_to "django.db.models.FileField.upload_to")的`'%Y/%m/%d'`被[`strftime()`](https://docs.python.org/3/library/time.html#time.strftime "(in Python v3.4)")所格式化；`'%Y'` 将会被格式化为一个四位数的年份, `'%m'` 被格式化为一个两位数的月份`'%d'`是两位数日份。如果你在Jan.15.2007上传了一个文件，它将被保存在`/home/media/photos/2007/01/15`目录下.

如果你想获得上传文件的存盘文件名，或者是文件大小，你可以分别使用 [`name`](../files/file.html#django.core.files.File.name "django.core.files.File.name") 和 [`size`](../files/file.html#django.core.files.File.size "django.core.files.File.size") 属性； 更多可用属性及方法信息，请参见 [`File`](../files/file.html#django.core.files.File "django.core.files.File") 类索引 和 [_Managing files_](../../topics/files.html) 主题指导.



Note

保存的文件作为模型存储在数据库中的一部分，所以在磁盘上使用的实际的文件名在模型保存完毕之前是不可靠的。



上传的文件对应的URL可以通过使用 [`url`](#django.db.models.fields.files.FieldFile.url "django.db.models.fields.files.FieldFile.url") 属性获得. 在内部，它会调用 ?[`Storage`](../files/storage.html#django.core.files.storage.Storage "django.core.files.storage.Storage") 类下的[`url()`](../files/storage.html#django.core.files.storage.Storage.url "django.core.files.storage.Storage.url")方法.

值得注意的是，无论你在任何时候处理上传文件的需求，你都应该密切关注你的文件将被上传到哪里，上传的文件类型，以避免安全漏洞。_认证所有上传文件_ 以确保那些上传的文件是你所认为的文件。例如，如果你盲目的允许其他人在无需认证的情况下上传文件至你的web服务器的root目录中，那么别人可以上传一个CGI或者PHP脚本然后通过访问一个你网站的URL来执行这个脚本。所以，不要允许这种事情发生。

甚至是上传HTML文件也值得注意，它可以通过浏览器（虽然不是服务器）执行，也可以引发相当于是XSS或者CSRF攻击的安全威胁。

[`FileField`](#django.db.models.FileField "django.db.models.FileField") 实例将会在你的数据库中创建一个默认最大长度为100字符的`varchar` 列。就像其他的fields一样, 你可以用 [`max_length`](#django.db.models.CharField.max_length "django.db.models.CharField.max_length") 参数改变最大长度的值.



#### FileField 和FieldFile



_class_ `FieldFile`[[source]](../../_modules/django/db/models/fields/files.html#FieldFile)



当你添加 [`FileField`](#django.db.models.FileField "django.db.models.FileField") 到你的模型中时, 你实际上会获得一个 [`FieldFile`](#django.db.models.fields.files.FieldFile "django.db.models.fields.files.FieldFile")的实例来替代将要访问的文件。 除了继承至 [`django.core.files.File`](../files/file.html#django.core.files.File "django.core.files.File")的功能外, 这个类还有其他属性和方法可以用于访问文件:



`FieldFile.url`



通过潜在[`Storage`](../files/storage.html#django.core.files.storage.Storage "django.core.files.storage.Storage") 类的[`url()`](../files/storage.html#django.core.files.storage.Storage.url "django.core.files.storage.Storage.url")方法可以只读地访问文件的URL。



`FieldFile.open`(_mode='rb'_)[[source]](../../_modules/django/db/models/fields/files.html#FieldFile.open)



该方法像标准的Python `open()` 方法,并可通过?`mode`参数设置打开模式.



`FieldFile.close`()[[source]](../../_modules/django/db/models/fields/files.html#FieldFile.close)



该方法像标准的Python`file.close()` 方法,并关闭相关文件.



`FieldFile.save`(_name_, _content_, _save=True_)[[source]](../../_modules/django/db/models/fields/files.html#FieldFile.save)



这个方法会将文件名以及文件内容传递到字段的storage类中,并将模型字段与保存好的文件关联. 如果想要手动关联文件数据到你的模型中的 [`FileField`](#django.db.models.FileField "django.db.models.FileField")实例, 则`save()` 方法总是用来保存该数据.

方法接受两个必选参数: `name` 文件名, 和 `content` 文件内容.可选参数`save` 控制模型实例在关联的文件被修改时是否保存.默认为 `True`.

注意参数 `content` 应该是 [`django.core.files.File`](../files/file.html#django.core.files.File "django.core.files.File")的一个实例, 而不是Python内建的File对象.你可以用如下方法从一个已经存在的Python文件对象来构建 [`File`](../files/file.html#django.core.files.File "django.core.files.File") :





```
from django.core.files import File
# Open an existing file using Python's built-in open()
f = open('/tmp/hello.world')
myfile = File(f)

```





或者，你可以像下面的一样从一个python字符串中构建





```
from django.core.files.base import ContentFile
myfile = ContentFile("hello world")

```





更多信息, 请参见 [_Managing files_](../../topics/files.html).



`FieldFile.delete`(_save=True_)[[source]](../../_modules/django/db/models/fields/files.html#FieldFile.delete)



删除与此实例关联的文件，并清除该字段的所有属性。注意︰ 如果它碰巧是开放的调用 `delete()` 方法 时，此方法将关闭该文件。

模型实例`save`的文件与此字段关联的可选 保存 参数控件已被删除。默认值为 `True`。

注意，model删除的时候，与之关联的文件并不会被删除。如果你要把文件也清理掉，你需要自己处理。







### FilePathField



_class_ `FilePathField`(_path=None_[, _match=None_, _recursive=False_, _max_length=100_, _**options_])



一个?[`CharField`](#django.db.models.CharField "django.db.models.CharField") ，内容只限于文件系统内特定目录下的文件名。有三个参数, 其中第一个是 **必需的**:



`FilePathField.path`



必填。这个[`FilePathField`](#django.db.models.FilePathField "django.db.models.FilePathField") 应该得到其选择的目录的绝对文件系统路径。例如: `"/home/images"`.







`FilePathField.match`



可选的.[`FilePathField`](#django.db.models.FilePathField "django.db.models.FilePathField") 将会作为一个正则表达式来匹配文件名。但请注意正则表达式将将被作用于基本文件名，而不是完整路径。例如: `"foo.*.txt$"`, 将会匹配到一个名叫 `foo23.txt` 的文件，但不匹配到 `bar.txt` 或者 `foo23.png`.







`FilePathField.recursive`



可选的.`True` 或 `False`.默认是`False`.声明是否包含所有子目录的[`路径`](#django.db.models.FilePathField.path "django.db.models.FilePathField.path")







`FilePathField.allow_files`



可选的.`True` 或 `False`.默认是`True`.声明是否包含指定位置的文件。该参数或[`allow_folders`](#django.db.models.FilePathField.allow_folders "django.db.models.FilePathField.allow_folders") 中必须有一个为 `True`.







`FilePathField.allow_folders`



是可选的.输入 `True` 或者 `False`.默认值为 `False`.声明是否包含指定位置的文件夹。该参数或 [`allow_files`](#django.db.models.FilePathField.allow_files "django.db.models.FilePathField.allow_files") 中必须有一个为 `True`.





当然，这些参数可以同时使用。

有一点需要提醒的是 [`match`](#django.db.models.FilePathField.match "django.db.models.FilePathField.match")只匹配基本文件名（base filename）, 而不是整个文件路径（full path）. 例如:





```
FilePathField(path="/home/images", match="foo.*", recursive=True)

```





...将匹配`/home/images/foo.png`而不是`/home/images/foo/bar.png` 因为只允许[`匹配`](#django.db.models.FilePathField.match "django.db.models.FilePathField.match") 基本文件名(`foo.png` 和 `bar.png`).

[`FilePathField`](#django.db.models.FilePathField "django.db.models.FilePathField")实例被创建在您的数据库为`varchar`列默认最大长度为 100 个字符。作为与其他字段，您可以更改使用的[`max_length`](#django.db.models.CharField.max_length "django.db.models.CharField.max_length")最大长度。





### FloatField



_class_ `FloatField`([_**options_])



用Python的一个`float` 实例来表示一个浮点数.

该字段的默认组件是一个 [`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput").



`FloatField` 与`DecimalField`

有时候[`FloatField`](#django.db.models.FloatField "django.db.models.FloatField") 类会和[`DecimalField`](#django.db.models.DecimalField "django.db.models.DecimalField") 类发生混淆. 虽然它们表示都表示实数，但是二者表示数字的方式不一样。`FloatField` 使用的是Python内部的 `float` 类型, 而`DecimalField` 使用的是Python的 `Decimal` 类型. 想要了解更多二者的差别, 可以查看Python文档中的 [`decimal`](https://docs.python.org/3/library/decimal.html#module-decimal "(in Python v3.4)") 模块.







### ImageField



_class_ `ImageField`([_upload_to=None_, _height_field=None_, _width_field=None_, _max_length=100_, _**options_])



继承了 [`FileField`](#django.db.models.FileField "django.db.models.FileField")的所有属性和方法, 但还对上传的对象进行校验，确保它是个有效的image.

除了从[`FileField`](#django.db.models.FileField "django.db.models.FileField")继承来的属性外，[`ImageField`](#django.db.models.ImageField "django.db.models.ImageField") 还有`宽`和 `高`属性。

为了更便捷的去用那些属性值, [`ImageField`](#django.db.models.ImageField "django.db.models.ImageField") 有两个额外的可选参数



`ImageField.height_field`



该属性的设定会在模型实例保存时,自动填充图片的高度.







`ImageField.width_field`



该属性的设定会在模型实例保存时,自动填充图片的宽度.





ImageField字段需要调用[Pillow](http://pillow.readthedocs.org/en/latest/) 库.

[`ImageField`](#django.db.models.ImageField "django.db.models.ImageField")会创建在你的数据库中 和 `varchar` 一样,默认最大长度为100和其他字段一样, 你可以使用[`max_length`](#django.db.models.CharField.max_length "django.db.models.CharField.max_length") 参数来设置默认文件最大值.

此字段的默认表单工具是[`ClearableFileInput`](../forms/widgets.html#django.forms.ClearableFileInput "django.forms.ClearableFileInput").





### IntegerField



_class_ `IntegerField`([_**options_])



一个整数。在Django所支持的所有数据库中，从 `-2147483648` 到 `2147483647` 范围内的值是合法的。默认的表单输入工具是[`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput").





### IPAddressField



_class_ `IPAddressField`([_**options_])





Deprecated since version 1.7: 该字段已废弃，从1.7开始支持[`GenericIPAddressField`](#django.db.models.GenericIPAddressField "django.db.models.GenericIPAddressField").



IP地址，会自动格式化（例如：“192.0.2.30”）。默认表单控件为 [`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput").





### GenericIPAddressField



_class_ `GenericIPAddressField`([_protocol=both_, _unpack_ipv4=False_, _**options_])



一个 IPv4 或 IPv6 地址, 字符串格式 (例如 `192.0.2.30` 或 `2a02:42fe::4`). 这个字段的默认表单小部件是一个[`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput").

IPv6 地址会根据 [**RFC 4291**](http://tools.ietf.org/html/rfc4291.html#section-2.2) 章节 2.2所规范, 包括该章节中第三段的的IPv4格式建议, 就像 `::ffff:192.0.2.0`这样. 例如, `2001:0::0:01` 将会被规范成 `2001::1`, `::ffff:0a0a:0a0a` 被规范成 `::ffff:10.10.10.10`. 所有字符都会被转换成小写.



`GenericIPAddressField.protocol`



限制有效输入的协议类型. 允许的值是 `'both'` (默认值), `'IPv4'` 或 `'IPv6'`. 匹配不区分大小写.







`GenericIPAddressField.unpack_ipv4`



解析IPv4映射地址如 `::ffff:192.0.2.1`.如果启用该选项，则地址将被解析到`192.0.2.1`.默认为禁用。只有当`协议` 设置为`'both'`时才可以使用。





如果允许空白值，则必须允许null值，因为空白值存储为null。





### NullBooleanField



_class_ `NullBooleanField`([_**options_])



类似[`BooleanField`](#django.db.models.BooleanField "django.db.models.BooleanField"), 但是允许 `NULL` 作为一个选项.使用此代替`null=True`的[`BooleanField`](#django.db.models.BooleanField "django.db.models.BooleanField")。此字段的默认表单widget为[`NullBooleanSelect`](../forms/widgets.html#django.forms.NullBooleanSelect "django.forms.NullBooleanSelect")。





### PositiveIntegerField（正整数字段）



_class_ `PositiveIntegerField`([_**options_])



类似 [`IntegerField`](#django.db.models.IntegerField "django.db.models.IntegerField"), 但值必须是正数或者零(`0`). 从`0`到`2147483647`的值在Django支持的所有数据库中都是安全的。由于向后兼容性原因，接受值`0`。





### PositiveSmallIntegerField



_class_ `PositiveSmallIntegerField`([_**options_])



该模型字段类似 [`PositiveIntegerField`](#django.db.models.PositiveIntegerField "django.db.models.PositiveIntegerField"), 但是只允许小于某一特定值（依据数据库类型而定）。从`0` 到 `32767` 这个区间，对于Django所支持的所有数据库而言都是安全的。





### SlugField



_class_ `SlugField`([_max_length=50_, _**options_])



[_Slug_](../../glossary.html#term-slug) 是一个新闻术语（通常叫做短标题）。一个slug只能包含字母、数字、下划线或者是连字符，通常用来作为短标签。通常它们是用来放在URL里的。

像CharField一样，你可以指定[`max_length`](#django.db.models.CharField.max_length "django.db.models.CharField.max_length")（也请参阅该部分中的有关数据库可移植性的说明和[`max_length`](#django.db.models.CharField.max_length "django.db.models.CharField.max_length")）。如果没有指定 [`max_length`](#django.db.models.CharField.max_length "django.db.models.CharField.max_length"), Django将会默认长度为50。

Implies setting [`Field.db_index`](#django.db.models.Field.db_index "django.db.models.Field.db_index") to `True`.

根据某些其他值的值自动预填充SlugField通常很有用。你可以在admin中使用[`prepopulated_fields`](../contrib/admin/index.html#django.contrib.admin.ModelAdmin.prepopulated_fields "django.contrib.admin.ModelAdmin.prepopulated_fields")自动执行此操作。





### SmallIntegerField



_class_ `SmallIntegerField`([_**options_])



与 [`IntegerField`](#django.db.models.IntegerField "django.db.models.IntegerField")这个字段类型很类似,不同的是SmallInteger类型只能在一个确定的范围内(数据库依赖)。对于django来讲，该字段值在 `-32768` 至 `32767`这个范围内对所有可支持的数据库都是安全的。





### TextField



_class_ `TextField`([_**options_])



大文本字段。该模型默认的表单组件是[`Textarea`](../forms/widgets.html#django.forms.Textarea "django.forms.Textarea")。

Changed in Django 1.7:

如果你在这个字段类型中使用了`max_length`属性，它将会在渲染页面表单元素[`Textarea`](../forms/widgets.html#django.forms.Textarea "django.forms.Textarea") 时候体现出来。但是并不会在model或者数据库级别强制性的限定字段长度。请使用[`CharField`](#django.db.models.CharField "django.db.models.CharField")。





MySQL 用户

如果你将此字段用于MySQLdb 1.2.1p2和`utf8_bin`排序规则（这_不是_默认值），则需要注意一些问题。有关详细信息，请参阅[_MySQL数据库注释_](../databases.html#mysql-collation)。







### TimeField



_class_ `TimeField`([_auto_now=False_, _auto_now_add=False_, _**options_])



时间字段，和Python中 `datetime.time` 一样。接受与[`DateField`](#django.db.models.DateField "django.db.models.DateField")相同的自动填充选项。

表单默认为 [`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput").输入框。Admin添加一些JavaScript快捷方式。





### URLField



_class_ `URLField`([_max_length=200_, _**options_])



一个[`CharField`](#django.db.models.CharField "django.db.models.CharField") 类型的URL

此字段的默认表单widget为[`TextInput`](../forms/widgets.html#django.forms.TextInput "django.forms.TextInput")。

与所有[`CharField`](#django.db.models.CharField "django.db.models.CharField")子类一样，[`URLField`](#django.db.models.URLField "django.db.models.URLField")接受可选的[`max_length`](#django.db.models.CharField.max_length "django.db.models.CharField.max_length")参数。如果不指定[`max_length`](#django.db.models.CharField.max_length "django.db.models.CharField.max_length")，则使用默认值200。





### UUIDField

New in Django 1.8.



_class_ `UUIDField`([_**options_])



一个用来存储UUID的字段。使用Python的[`UUID`](https://docs.python.org/3/library/uuid.html#uuid.UUID "(in Python v3.4)")类。 当使用PostgreSQL数据库时，该字段类型对应的数据库中的数据类型是`uuid`，使用其他数据库时，数据库对应的是`char(32)`类型。

使用UUID类型相对于使用具有[`primary_key`](#django.db.models.Field.primary_key "django.db.models.Field.primary_key")参数的[`AutoField`](#django.db.models.AutoField "django.db.models.AutoField")类型是一个更好的解决方案。 数据库不会自动生成UUID，所以推荐使用[`default`](#django.db.models.Field.default "django.db.models.Field.default")参数：





```
import uuid
from django.db import models

class MyUUIDModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    # other fields

```





注意：这里传递给default是一个可调用的对象（即一个省略了括号的方法），而不是传递一个`UUID`实例给`default`







## 关系字段

Django 同样定义了一系列的字段来描述数据库之间的关联。



### ForeignKey



_class_ `ForeignKey`(_othermodel_[, _**options_])



多对一关系。需要一个位置参数：与该模型关联的类。

若要创建一个递归的关联 —— 对象与自己具有多对一的关系 —— 请使用`models.ForeignKey('self')`。

如果你需要关联到一个还没有定义的模型，你可以使用模型的名字而不用模型对象本身：





```
from django.db import models

class Car(models.Model):
    manufacturer = models.ForeignKey('Manufacturer')
    # ...

class Manufacturer(models.Model):
    # ...
    pass

```





若要引用在其它应用中定义的模型，你可以用带有完整标签名的模型来显式指定。例如，如果上面提到的`Manufacturer` 模型是在一个名为`production` 的应用中定义的，你应该这样使用它：





```
class Car(models.Model):
    manufacturer = models.ForeignKey('production.Manufacturer')

```





在解析两个应用之间具有相互依赖的导入时，这种引用将会很有帮助。

`ForeignKey` 会自动创建数据库索引。你可以通过设置[`db_index`](#django.db.models.Field.db_index "django.db.models.Field.db_index") 为`False` 来取消。如果你创建外键是为了一致性而不是用来Join，或者如果你将创建其它索引例如部分或多列索引，你也许想要避免索引的开销。



警告

不建议从一个没有迁移的应用中创建一个`ForeignKey` 到一个具有迁移的应用。更多详细信息，请参阅[_依赖性文档_](../../topics/migrations.html#unmigrated-dependencies)。





#### 数据库中的表示

在幕后，Django 会在字段名上添加`"_id"` 来创建数据库中的列名。在上面的例子中，`Car` 模型的数据库表将会拥有一个`manufacturer_id` 列。（你可以通过显式指定[`db_column`](#django.db.models.Field.db_column "django.db.models.Field.db_column")?改变它）。但是，你的代码永远不应该处理数据库中的列名称，除非你需要编写自定义的SQL。你应该永远只处理你的模型对象中的字段名称。





#### 参数

[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 接受额外的参数集 —— 全都是可选的 —— 它们定义关联关系如何工作的细节。



`ForeignKey.limit_choices_to`



当这个字段使用`模型表单`或者Admin 渲染时（默认情况下，查询集中的所有对象都可以使用），为这个字段设置一个可用的选项。它可以是一个字典、一个[`Q`](querysets.html#django.db.models.Q "django.db.models.Q") 对象或者一个返回字典或[`Q`](querysets.html#django.db.models.Q "django.db.models.Q")对象的可调用对象。

例如：





```
staff_member = models.ForeignKey(User, limit_choices_to={'is_staff': True})

```





将使得`模型表单` 中对应的字段只列出`is_staff=True` 的`Users`。 这在Django 的Admin 中也可能有用处。

可调用对象的形式同样非常有用，比如与Python 的`datetime`模块一起使用来限制选择的时间范围。例如：





```
def limit_pub_date_choices():
    return {'pub_date__lte': datetime.date.utcnow()}

limit_choices_to = limit_pub_date_choices

```





如果`limit_choices_to` 自己本身是或者返回一个用于[_复杂查询_](../../topics/db/queries.html#complex-lookups-with-q)的[`Q 对象`](querysets.html#django.db.models.Q "django.db.models.Q")，当字段没有在模型的`ModelAdmin`中的[`raw_id_fields`](../contrib/admin/index.html#django.contrib.admin.ModelAdmin.raw_id_fields "django.contrib.admin.ModelAdmin.raw_id_fields") 列出时，它将只会影响Admin中的可用的选项。

Changed in Django 1.7:

以前的Django 版本不允许传递一个可调用的对象给`limit_choices_to`。





注

如果`limit_choices_to` 使用可调用对象，这个可调用对象将在每次创建一个新表单的时候都调用。它还可能在一个模型校验的时候调用，例如被管理命令或者Admin。Admin 多次构造查询集来验证表单在各种边缘情况下的输入，所以你的可调用对象可能调用多次。









`ForeignKey.related_name`



这个名称用于让关联的对象反查到源对象。它还是[`related_query_name`](#django.db.models.ForeignKey.related_query_name "django.db.models.ForeignKey.related_query_name") 的默认值（关联的模型进行反向过滤时使用的名称）。完整的解释和示例参见[_关联对象的文档_](../../topics/db/queries.html#backwards-related-objects)。注意，当你为[_抽象模型_](../../topics/db/models.html#abstract-base-classes)定义关联关系的时，必须设置这个参数的值；而且当你这么做的时候需要用到[_一些特殊语法_](../../topics/db/models.html#abstract-related-name)。

如果你不想让Django 创建一个反向关联，请设置`related_name` 为 `'+'` 或者以`'+'` 结尾。 例如，下面这行将确定`User` 模型将不会有到这个模型的返回关联：





```
user = models.ForeignKey(User, related_name='+')

```











`ForeignKey.related_query_name`



这个名称用于目标模型的反向过滤。如果设置了[`related_name`](#django.db.models.ForeignKey.related_name "django.db.models.ForeignKey.related_name")，则默认为它的值，否则默认值为模型的名称：





```
# Declare the ForeignKey with related_query_name
class Tag(models.Model):
    article = models.ForeignKey(Article, related_name="tags", related_query_name="tag")
    name = models.CharField(max_length=255)

# That's now the name of the reverse filter
Article.objects.filter(tag__name="important")

```











`ForeignKey.to_field`



关联到的关联对象的字段名称。默认地，Django 使用关联对象的主键。







`ForeignKey.db_constraint`



控制是否在数据库中为这个外键创建约束。默认值为`True`，而且这几乎一定是你想要的效果；设置成`False` 对数据的完整性来说是很糟糕的。即便如此，有一些场景你也许想要这么设置：

*   你有遗留的无效数据。
*   你正在对数据库缩容。

如果被设置成`False`，访问一个不存在的关联对象将抛出 `DoesNotExist` 异常。







`ForeignKey.on_delete`



当一个[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 引用的对象被删除时，Django 默认模拟SQL 的`ON DELETE CASCADE` 的约束行为，并且删除包含该`ForeignKey`的对象。这种行为可以通过设置[`on_delete`](#django.db.models.ForeignKey.on_delete "django.db.models.ForeignKey.on_delete") 参数来改变。例如，如果你有一个可以为空的[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey")，在其引用的对象被删除的时你想把这个ForeignKey 设置为空：





```
user = models.ForeignKey(User, blank=True, null=True, on_delete=models.SET_NULL)

```









[`on_delete`](#django.db.models.ForeignKey.on_delete "django.db.models.ForeignKey.on_delete") 在[`django.db.models`](../../topics/db/models.html#module-django.db.models "django.db.models")中可以找到的值有：

*   `CASCADE`

    级联删除；默认值。

*   `PROTECT`

    抛出[`ProtectedError`](../exceptions.html#django.db.models.ProtectedError "django.db.models.ProtectedError") 以阻止被引用对象的删除，它是[`django.db.IntegrityError`](../exceptions.html#django.db.IntegrityError "django.db.IntegrityError") 的一个子类。

*   `SET_NULL`

    把[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 设置为null； [`null`](#django.db.models.Field.null "django.db.models.Field.null") 参数为`True` 时才可以这样做。

*   `SET_DEFAULT`

    [`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 值设置成它的默认值；此时必须设置[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 的default 参数。

*   `SET`()

    设置[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 为传递给[`SET()`](#django.db.models.SET "django.db.models.SET") 的值，如果传递的是一个可调用对象，则为调用后的结果。在大部分情形下，传递一个可调用对象用于避免models.py 在导入时执行查询：

    ```
    from django.conf import settings
        from django.contrib.auth import get_user_model
        from django.db import models

        def get_sentinel_user():
            return get_user_model().objects.get_or_create(username='deleted')[0]

        class MyModel(models.Model):
            user = models.ForeignKey(settings.AUTH_USER_MODEL,
                                     on_delete=models.SET(get_sentinel_user))
        
    ```

*   `DO_NOTHING`

    不采取任何动作。如果你的数据库后端强制引用完整性，它将引发一个[`IntegrityError`](../exceptions.html#django.db.IntegrityError "django.db.IntegrityError") ，除非你手动添加一个`ON DELETE` 约束给数据库自动（可能要用到[_初始化的SQL_](../../howto/initial-data.html#initial-sql)）。



`ForeignKey.swappable`



New in Django 1.7.

控制迁移框架的的重复行为如果该[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 指向一个可切换的模型。如果它是默认值`True`，那么如果[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 指向的模型与`settings.AUTH_USER_MODEL` 匹配（或其它可切换的模型），则保存在迁移中的关联关系将使用对setting 中引用而不是直接对模型的引用。

只有当你确定你的模型将永远指向切换后的模型 —— 例如如果它是专门为你的自定义用户模型设计的模型时，你才会想将它设置成`False`。

设置为`False` 并不表示你可以引用可切换的模型即使在它被切换出去之后 —— `False` 只是表示生成的迁移中ForeignKey 将始终引用你指定的准确模型（所以，如果用户试图允许一个你不支持的User 模型时将会失败）。

如果有疑问，请保留它的默认值`True`。







`ForeignKey.allow_unsaved_instance_assignment`



New in Django 1.8:

添加这个标志是为了向后兼容，因为老版本的Django 始终允许赋值未保存的模型实例。



Django 阻止未保存的模型实例被分配给一个`ForeignKey` 字段以防止意味的数据丢失（当保存一个模型实例时，未保存的外键将默默忽略）。

如果你需要允许赋值未保存的实例且不关心数据的丢失（例如你不会保存对象到数据库），你可以通过创建这个字段的子类并设置其`allow_unsaved_instance_assignment` 属性为`True` 来关闭这个检查。例如：





```
class UnsavedForeignKey(models.ForeignKey):
    # A ForeignKey which can point to an unsaved object
    allow_unsaved_instance_assignment = True

class Book(models.Model):
    author = UnsavedForeignKey(Author)

```















### ManyToManyField



_class_ `ManyToManyField`(_othermodel_[, _**options_])



一个多对多关联。要求一个关键字参数：与该模型关联的类，与[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 的工作方式完全一样，包括[_递归关系_](#recursive-relationships) 和[_惰性关系_](#lazy-relationships)。

关联的对象可以通过字段的[`RelatedManager`](relations.html#django.db.models.fields.related.RelatedManager "django.db.models.fields.related.RelatedManager") 添加、删除和创建。



警告

不建议从一个没有迁移的应用中创建一个`ManyToManyField`到一个具有迁移的应用。 更多细节参见[_依赖性文档_](../../topics/migrations.html#unmigrated-dependencies)。





#### 数据库中的表示

在幕后，Django 创建一个中间表来表示多对多关系。默认情况下，这张中间表的名称使用多对多字段的名称和包含这张表的模型的名称生成。因为某些数据库支持的表的名字的长度有限制，这些标的名称将自动截短到64 个字符并加上一个唯一性的哈希值。这意味着，你看的表的名称可能类似 `author_books_9cdf4`；这再正常不过了。你可以使用[`db_table`](#django.db.models.ManyToManyField.db_table "django.db.models.ManyToManyField.db_table") 选项手工提供中间表的名称。





#### 参数

[`ManyToManyField`](#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 接收一个参数集来控制关联关系的功能 —— 它们都是可选的。



`ManyToManyField.related_name`



与[`ForeignKey.related_name`](#django.db.models.ForeignKey.related_name "django.db.models.ForeignKey.related_name") 相同。







`ManyToManyField.related_query_name`



与[`ForeignKey.related_query_name`](#django.db.models.ForeignKey.related_query_name "django.db.models.ForeignKey.related_query_name") 相同。







`ManyToManyField.limit_choices_to`



与[`ForeignKey.limit_choices_to`](#django.db.models.ForeignKey.limit_choices_to "django.db.models.ForeignKey.limit_choices_to") 相同。

`limit_choices_to` 对于使用[`through`](#django.db.models.ManyToManyField.through "django.db.models.ManyToManyField.through") 参数自定义中间表的`ManyToManyField` 不生效。







`ManyToManyField.symmetrical`



只用于与自身进行关联的ManyToManyField。例如下面的模型：





```
from django.db import models

class Person(models.Model):
    friends = models.ManyToManyField("self")

```





当Django 处理这个模型的时候，它定义该模型具有一个与自身具有多对多关联的[`ManyToManyField`](#django.db.models.ManyToManyField "django.db.models.ManyToManyField")，因此它不会向`Person` 类添加`person_set` 属性。Django 将假定这个[`ManyToManyField`](#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 字段是对称的 —— 如果我是你的朋友，那么你也是我的朋友。

如果你希望与`self` 进行多对多关联的关系不具有对称性，可以设置[`symmetrical`](#django.db.models.ManyToManyField.symmetrical "django.db.models.ManyToManyField.symmetrical") 为`False`。这会强制让Django 添加一个描述器给反向的关联关系，以使得[`ManyToManyField`](#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 的关联关系不是对称的。







`ManyToManyField.through`



Django 会自动创建一个表来管理多对多关系。不过，如果你希望手动指定中介表，可以使用[`through`](#django.db.models.ManyToManyField.through "django.db.models.ManyToManyField.through") 选项来指定Django 模型来表示你想要使用的中介表。

这个选项最常见的使用场景是当你想要关联[_额外的数据到多对多关联关系_](../../topics/db/models.html#intermediary-manytomany)的时候。

如果你没有显式指定`through` 的模型，仍然会有一个隐式的`through` 模型类，你可以用它来直接访问对应的表示关联关系的数据库表。它由三个字段来链接模型。

如果源模型和目标不同，则生成以下字段：

*   `id`：关系的主键。
*   `<containing_model>_id`：声明`ManyToManyField` 字段的模型的`id`。
*   `<other_model>_id`：`ManyToManyField` 字段指向的模型的`id`。

如果`ManyToManyField` 的源模型和目标模型相同，则生成以下字段：

*   `id`：关系的主键。
*   `from_<model>_id`：源模型实例的`id`。
*   `to_<model>_id`：目标模型实例的`id`。

这个类可以让一个给定的模型像普通的模型那样查询与之相关联的记录。







`ManyToManyField.through_fields`



New in Django 1.7.

只能在指定了自定义中间模型的时候使用。 Django 一般情况会自动决定使用中间模型的哪些字段来建立多对多关联。但是，考虑如下模型：





```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=50)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership', through_fields=('group', 'person'))

class Membership(models.Model):
    group = models.ForeignKey(Group)
    person = models.ForeignKey(Person)
    inviter = models.ForeignKey(Person, related_name="membership_invites")
    invite_reason = models.CharField(max_length=64)

```





`Membership` 有_两个_ 外键指向`Person` （`person` 和`inviter`），这使得关联关系含混不清并让Django 不知道使用哪一个。在这种情况下，你必须使用`through_fields` 明确指定Django 应该使用哪些外键，就像上面例子一样。

`through_fields` 接收一个二元组`('field1', 'field2')`，其中`field1` 为指向定义[`ManyToManyField`](#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 字段的模型的外键名称（本例中为`group`），`field2` 为指向目标模型的外键的名称（本例中为`person`）。

当中间模型具有多个外键指向多对多关联关系模型中的任何一个（或两个），你_必须_ 指定`through_fields`。这通用适用于[_递归的关联关系_](#recursive-relationships)，当用到中间模型而有多个外键指向该模型时，或当你想显式指定Django 应该用到的两个字段时。

递归的关联关系使用的中间模型始终定义为非对称的，也就是[`symmetrical=False`](#django.db.models.ManyToManyField.symmetrical "django.db.models.ManyToManyField.symmetrical") —— 所以具有源和目标的概念。这种情况下，`'field1'` 将作为管理关系的源，而`'field2'` 作为目标。







`ManyToManyField.db_table`



为存储多对多数据而创建的表的名称。如果没有提供，Django 将基于定义关联关系的模型和字段假设一个默认的名称。







`ManyToManyField.db_constraint`



控制中间表中的外键是否创建约束。默认为`True`，而且这是几乎就是你想要的；设置为`False` 对数据完整性将非常糟糕。下面是你可能需要这样设置的一些场景：

*   你具有不合法的遗留数据。
*   你正在对数据库缩容。

不可以同时传递`db_constraint` 和 `through`。







`ManyToManyField.swappable`



New in Django 1.7.

控制[`ManyToManyField`](#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 指向一个可切换的模型时迁移框架的行为。如果它是默认值`True`，那么如果[`ManyToManyField`](#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 指向的模型与`settings.AUTH_USER_MODEL` 匹配（或其它可切换的模型），则保存在迁移中的关联关系将使用对setting 中引用而不是直接对模型的引用。

只有当你确定你的模型将永远指向切换后的模型 —— 例如如果它是专门为你的自定义用户模型设计的模型时，你才会想将它设置成`False`。

如果不确定，请将它保留为`True`。







`ManyToManyField.allow_unsaved_instance_assignment`



New in Django 1.8.

与[`ForeignKey.allow_unsaved_instance_assignment`](#django.db.models.ForeignKey.allow_unsaved_instance_assignment "django.db.models.ForeignKey.allow_unsaved_instance_assignment") 的工作方式类似。





[`ManyToManyField`](#django.db.models.ManyToManyField "django.db.models.ManyToManyField") 不支持[`validators`](#django.db.models.Field.validators "django.db.models.Field.validators")。

[`null`](#django.db.models.Field.null "django.db.models.Field.null") 不生效，因为无法在数据库层次要求关联关系。







### OneToOneField



_class_ `OneToOneField`(_othermodel_[, _parent_link=False_, _**options_])



一对一关联关系。概念上讲，这个字段很像是[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 设置了[`unique=True`](#django.db.models.Field.unique "django.db.models.Field.unique")，不同的是它会直接返回关系另一边的单个对象。

它最主要的用途是作为扩展自另外一个模型的主键；例如，[_多表继承_](../../topics/db/models.html#multi-table-inheritance)就是通过对子模型添加一个隐式的一对一关联关系到父模型实现的。

需要一个位置参数：与该模型关联的类。 它的工作方式与[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 完全一致，包括所有与[_递归关系_](#recursive-relationships)和[_惰性关系_](#lazy-relationships)相关的选项。

如果你没有指定`OneToOneField` 的[`related_name`](#django.db.models.ForeignKey.related_name "django.db.models.ForeignKey.related_name") 参数，Django 将使用当前模型的小写的名称作为默认值。

例如下面的例子：





```
from django.conf import settings
from django.db import models

class MySpecialUser(models.Model):
    user = models.OneToOneField(settings.AUTH_USER_MODEL)
    supervisor = models.OneToOneField(settings.AUTH_USER_MODEL, related_name='supervisor_of')

```





你将使得`User` 模型具有以下属性：





```
>>> user = User.objects.get(pk=1)
>>> hasattr(user, 'myspecialuser')
True
>>> hasattr(user, 'supervisor_of')
True

```





当反向访问关联关系时，如果关联的对象不存在对应的实例，则抛出`DoesNotExist` 异常。例如，如果一个User 没有`MySpecialUser` 指定的supervisor：





```
>>> user.supervisor_of
Traceback (most recent call last):
    ...
DoesNotExist: User matching query does not exist.

```





另外，`OneToOneField` 除了接收[`ForeignKey`](#django.db.models.ForeignKey "django.db.models.ForeignKey") 接收的所有额外的参数之外，还有另外一个参数：



`OneToOneField.parent_link`



当它为`True` 并在继承自另一个[_具体模型_](../../glossary.html#term-concrete-model) 的模型中使用时，表示该字段应该用于反查的父类的链接，而不是在子类化时隐式创建的`OneToOneField`。





`OneToOneField` 的使用示例参见[_One-to-one 关联关系_](../../topics/db/examples/one_to_one.html)。







## Field API 参考



_class_ `Field`



`Field` 是一个抽象的类, 用来代表数据库中的表的一列. Django 使用这些fields 去创建表 ([`db_type()`](#django.db.models.Field.db_type "django.db.models.Field.db_type")), 去建立Python中的类型和数据库中类型的映射关系 ([`get_prep_value()`](#django.db.models.Field.get_prep_value "django.db.models.Field.get_prep_value")) 反之亦然 ([`from_db_value()`](#django.db.models.Field.from_db_value "django.db.models.Field.from_db_value")), 并且实现[_Lookup API reference_](lookups.html) ([`get_prep_lookup()`](#django.db.models.Field.get_prep_lookup "django.db.models.Field.get_prep_lookup")).

field 是不同Django版本API中最根本的部分，尤其是[`models`](instances.html#django.db.models.Model "django.db.models.Model") and [`querysets`](querysets.html#django.db.models.query.QuerySet "django.db.models.query.QuerySet").

在模型中，一个字段被实例化为类的属性，并表现为一个特定的表的列，详情查看[_Models_](../../topics/db/models.html). 它具有[`null`](#django.db.models.Field.null "django.db.models.Field.null")和[`唯一`](#django.db.models.Field.unique "django.db.models.Field.unique")等属性，以及Django用于将字段值映射到数据库特定值的方法。

`字段`是[`RegisterLookupMixin`](lookups.html#django.db.models.lookups.RegisterLookupMixin "django.db.models.lookups.RegisterLookupMixin")的子类，因此可以在其上注册[`Transform`](lookups.html#django.db.models.Transform "django.db.models.Transform")和[`Lookup`](lookups.html#django.db.models.Lookup "django.db.models.Lookup") `QuerySet` s（例如`field_name__exact =“foo”`）。默认情况下，所有[_内置查找_](querysets.html#field-lookups)都已注册。

Django的所有内建字段，如[`CharField`](#django.db.models.CharField "django.db.models.CharField")都是`Field`的特定实现。如果您需要自定义字段，则可以将任何内置字段子类化，也可以从头开始写入`字段`。无论哪种情况，请参阅[_编写自定义模型字段_](../../howto/custom-model-fields.html)。



`description`



字段的详细说明，例如用于[`django.contrib.admindocs`](../contrib/admin/admindocs.html#module-django.contrib.admindocs "django.contrib.admindocs: Django's admin documentation generator.")应用程序。

描述可以是以下形式：





```
description = _("String (up to %(max_length)s)")

```





其中参数从字段的`__ dict __`插入。





To map a `Field` to a database-specific type, Django exposes two methods:



`get_internal_type`()



返回一个字符串，命名此字段以用于后端特定用途。默认情况下，它返回类名。

有关自定义字段中的用法，请参见[_模拟内置字段类型_](../../howto/custom-model-fields.html#emulating-built-in-field-types)。







`db_type`(_connection_)



返回[`字段`](#django.db.models.Field "django.db.models.Field")的数据库列数据类型，同时考虑`连接`。

有关自定义字段中的用法，请参见[_自定义数据库类型_](../../howto/custom-model-fields.html#custom-database-types)。





有三种主要情况，Django需要与数据库后端和字段交互：

*   当它查询数据库（Python值 转为 数据库后端值）
*   当它从数据库加载数据（数据库后端值 转为 Python值）
*   当它保存到数据库（Python值 转为 数据库后端值）

查询时，使用[`get_db_prep_value()`](#django.db.models.Field.get_db_prep_value "django.db.models.Field.get_db_prep_value")和[`get_prep_value()`](#django.db.models.Field.get_prep_value "django.db.models.Field.get_prep_value")：



`get_prep_value`(_value_)



`value`是模型属性的当前值，方法应以已准备好用作查询中的参数的格式返回数据。

有关使用方式，请参阅[_将Python对象转换为查询值_](../../howto/custom-model-fields.html#converting-python-objects-to-query-values)。







`get_db_prep_value`(_value_, _connection_, _prepared=False_)



将`值`转换为后端特定值。如果`prepared = True`和[`get_prep_value()`](#django.db.models.Field.get_prep_value "django.db.models.Field.get_prep_value") if为`False`，则默认返回`值`。

有关用法，请参见[_将查询值转换为数据库值_](../../howto/custom-model-fields.html#converting-query-values-to-database-values)。





加载数据时，使用[`from_db_value()`](#django.db.models.Field.from_db_value "django.db.models.Field.from_db_value")：



`from_db_value`(_value_, _expression_, _connection_, _context_)



New in Django 1.8.

将数据库返回的值转换为Python对象。它与[`get_prep_value()`](#django.db.models.Field.get_prep_value "django.db.models.Field.get_prep_value")相反。

此方法不用于大多数内置字段，因为数据库后端已返回正确的Python类型，或后端本身执行转换。

有关用法，请参见[_将值转换为Python对象_](../../howto/custom-model-fields.html#converting-values-to-python-objects)。



注意

出于性能原因，`from_db_value`在不需要它的字段（所有Django字段）上不实现为无操作。因此，您不能在定义中调用`super`。







保存时，使用[`pre_save()`](#django.db.models.Field.pre_save "django.db.models.Field.pre_save")和[`get_db_prep_save()`](#django.db.models.Field.get_db_prep_save "django.db.models.Field.get_db_prep_save")



`get_db_prep_save`(_value_, _connection_)



与[`get_db_prep_value()`](#django.db.models.Field.get_db_prep_value "django.db.models.Field.get_db_prep_value")相同，但在字段值必须_保存到数据库_时调用。默认返回[`get_db_prep_value()`](#django.db.models.Field.get_db_prep_value "django.db.models.Field.get_db_prep_value")。







`pre_save`(_model_instance_, _add_)



在[`get_db_prep_save()`](#django.db.models.Field.get_db_prep_save "django.db.models.Field.get_db_prep_save")之前调用方法以在保存之前准备值（例如，对于[`DateField.auto_now`](#django.db.models.DateField.auto_now "django.db.models.DateField.auto_now")）。

`model_instance`是此字段所属的实例，`add`是实例是否第一次保存到数据库。

它应该返回此字段的`model_instance`适当属性的值。属性名称位于`self.attname`（这是由[`Field`](#django.db.models.Field "django.db.models.Field")设置）。

有关使用情况，请参阅[_保存前预处理值_](../../howto/custom-model-fields.html#preprocessing-values-before-saving)。





当在字段上使用查找时，该值可能需要“准备”。Django公开了两种方法：



`get_prep_lookup`(_lookup_type_, _value_)



在用于查找之前，准备`values`到数据库。The `lookup_type` will be one of the valid Django filter lookups: `"exact"`, `"iexact"`, `"contains"`, `"icontains"`, `"gt"`, `"gte"`, `"lt"`, `"lte"`, `"in"`, `"startswith"`, `"istartswith"`, `"endswith"`, `"iendswith"`, `"range"`, `"year"`, `"month"`, `"day"`, `"isnull"`, `"search"`, `"regex"`, and `"iregex"`.

New in Django 1.7.

如果你使用[_自定义查找_](lookups.html)，则`lookup_type`可以是在字段中注册的任何`lookup_name`。

有关用法，请参见[_准备在数据库查找中使用的值_](../../howto/custom-model-fields.html#preparing-values-for-use-in-database-lookups)。







`get_db_prep_lookup`(_lookup_type_, _value_, _connection_, _prepared=False_)



类似于[`get_db_prep_value()`](#django.db.models.Field.get_db_prep_value "django.db.models.Field.get_db_prep_value")，但用于执行查找。

与[`get_db_prep_value()`](#django.db.models.Field.get_db_prep_value "django.db.models.Field.get_db_prep_value")一样，将用于查询的特定连接作为`connection`传递。此外，`prepared`描述该值是否已经用[`get_prep_lookup()`](#django.db.models.Field.get_prep_lookup "django.db.models.Field.get_prep_lookup")准备好。





字段经常从不同的类型接收它们的值，从序列化或从表单。



`to_python`(_value_)



改变这个值为正确的python对象。它作为[`value_to_string()`](#django.db.models.Field.value_to_string "django.db.models.Field.value_to_string")的反向操作，也在[`clean()`](instances.html#django.db.models.Model.clean "django.db.models.Model.clean")中调用。

有关用法，请参见[_将值转换为Python对象_](../../howto/custom-model-fields.html#converting-values-to-python-objects)。





除了保存到数据库，该字段还需要知道如何序列化其值：



`value_to_string`(_obj_)



将`obj`转换为字符串。用于序列化字段的值。

有关用法，请参见[_转换字段数据以进行序列化_](../../howto/custom-model-fields.html#converting-model-field-to-serialization)。





当使用[`模型 表单`](../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")时，`Field`需要知道应由哪个表单字段表示：



`formfield`(_form_class=None_, _choices_form_class=None_, _**kwargs_)



返回[`ModelForm`](../../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")的此字段的默认[`django.forms.Field`](../forms/fields.html#django.forms.Field "django.forms.Field")。

By default, if both `form_class` and `choices_form_class` are `None`, it uses [`CharField`](../forms/fields.html#django.forms.CharField "django.forms.CharField"); if `choices_form_class` is given, it returns [`TypedChoiceField`](../forms/fields.html#django.forms.TypedChoiceField "django.forms.TypedChoiceField").

有关用法，请参见[_为模型字段指定表单字段_](../../howto/custom-model-fields.html#specifying-form-field-for-model-field)。







`deconstruct`()



New in Django 1.7.

返回具有足够信息的4元组，以重新创建字段：

1.  模型上字段的名称。
2.  字段的导入路径（例如`“django.db.models.IntegerField”`）。这应该是兼容的版本，所以不要太具体可能会更好。
3.  位置参数列表。
4.  关键字参数的字典。

必须将此方法添加到1.7之前的字段，才能使用[_迁移_](../../topics/migrations.html)迁移其数据。















# Field属性参考

New in Django 1.8.

每个`字段`实例包含几个允许内省其行为的属性。Use these attributes instead of `isinstance` checks when you need to write code that depends on a field’s functionality. 这些属性可与[_Model._meta API_](meta.html#model-meta-field-api)一起使用，以缩小特定字段类型的搜索范围。自定义模型字段应实现这些标志。



## Attributes for fields



`Field.auto_created`



布尔标志，指示是否自动创建字段，例如模型继承使用的`OneToOneField`。







`Field.concrete`



布尔标志，指示字段是否具有与其相关联的数据库列。







`Field.hidden`



Boolean flag that indicates if a field is used to back another non-hidden field’s functionality (e.g. the `content_type` and `object_id` fields that make up a `GenericForeignKey`). The `hidden` flag is used to distinguish what constitutes the public subset of fields on the model from all the fields on the model.



Note

[`Options.get_fields()`](meta.html#django.db.models.options.Options.get_fields "django.db.models.options.Options.get_fields")默认排除隐藏字段。传入`include_hidden = True`可返回结果中的隐藏字段。









`Field.is_relation`



布尔标志，指示字段是否包含对一个或多个其他模型的功能的引用（例如`ForeignKey`，`ManyToManyField`，`OneToOneField`等） 。







`Field.model`



返回定义字段的模型。如果在模型的超类上定义了字段，则`模型`将引用超类，而不是实例的类。









## 具有关系的字段的属性

这些属性用于查询关系的基数和其他详细信息。这些属性存在于所有字段上；但是，如果字段是关系类型（[`Field.is_relation=True`](#django.db.models.Field.is_relation "django.db.models.Field.is_relation")），则它们只有布尔值（而不是`None`）。



`Field.many_to_many`



如果字段具有多对多关系，则布尔标志为`True`；否则为`False`。The only field included with Django where this is `True` is `ManyToManyField`.







`Field.many_to_one`



如果字段具有多对一关系，例如`ForeignKey`，则布尔标志为`True`；否则为`False`。







`Field.one_to_many`



如果字段具有一对多关系（例如`GenericRelation`或`ForeignKey`的反向），则`True`的布尔标志；否则为`False`。







`Field.one_to_one`



如果字段具有一对一关系，例如`OneToOneField`，则布尔标志为`True`；否则为`False`。







`Field.related_model`



指向字段涉及的模型。例如，`ForeignKey(Author)`中的`Author`。如果字段具有通用关系（例如`GenericForeignKey`或`GenericRelation`），则`related_model`将为`None`。



> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Field types](https://docs.djangoproject.com/en/1.8/ref/models/fields)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。






