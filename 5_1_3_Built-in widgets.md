

# 表单字段

_class_ `Field`(_**kwargs_)

创建一个`表单`类时，最重要的部分是定义表单的字段。每个字段都可以有自定义的验证逻辑，以及一些其它的钩子。

`Field.``clean`(_value_)

虽然`字段`类主要使用在`表单`类中，但你也可以直接实例化它们来使用，以便更好地了解它们是如何工作的。每个`字段`实例都有一个`clean()`方法， 它接受一个参数，然后返回“清洁的”数据或者抛出一个`django.forms.ValidationError`异常：

```
>>> from django import forms
>>> f = forms.EmailField()
>>> f.clean('foo@example.com')
'foo@example.com'
>>> f.clean('invalid email address')
Traceback (most recent call last):
...
ValidationError: ['Enter a valid email address.']

```

## 字段的核心参数

每个`字段`类的构造函数至少接受这些参数。有些`字段`类接受额外的、字段特有的参数，但以下参数应该_总是_能接受：

### 需要

`Field.``required`

默认情况下，每个`字段` 类都假设必需有值，所以如果你传递一个空的值 —— 不管是`None` 还是空字符串(`""`) —— `clean()` 将引发一个`ValidationError` 异常：

```
>>> from django import forms
>>> f = forms.CharField()
>>> f.clean('foo')
'foo'
>>> f.clean('')
Traceback (most recent call last):
...
ValidationError: ['This field is required.']
>>> f.clean(None)
Traceback (most recent call last):
...
ValidationError: ['This field is required.']
>>> f.clean(' ')
' '
>>> f.clean(0)
'0'
>>> f.clean(True)
'True'
>>> f.clean(False)
'False'

```

若要表示一个字段_不_是必需的，请传递`required=False` 给`字段`的构造函数：

```
>>> f = forms.CharField(required=False)
>>> f.clean('foo')
'foo'
>>> f.clean('')
''
>>> f.clean(None)
''
>>> f.clean(0)
'0'
>>> f.clean(True)
'True'
>>> f.clean(False)
'False'

```

如果`字段` 具有`required=False`，而你传递给`clean()` 一个空值，`clean()` 将返回一个_转换后_的空值而不是引发`ValidationError`。例如`CharField`，它将是一个空的Unicode 字符串。对于其它`字段`类，它可能是`None`。（每个字段各不相同）。

### 标签

`Field.``label`

`label` 参数让你指定字段“对人类友好”的label。当`字段`在`表单`中显示时将用到它。

正如在前面“输出表单为HTML”中解释的，`字段`默认label 是通过将字段名中所有的下划线转换成空格并大写第一个字母生成的。如果默认的标签不合适，可以指定`label`。

下面是一个完整示例，`表单`为它的两个字段实现了`label`。我们指定`auto_id=False`来让输出简单一些：

```
>>> from django import forms
>>> class CommentForm(forms.Form):
...     name = forms.CharField(label='Your name')
...     url = forms.URLField(label='Your Web site', required=False)
...     comment = forms.CharField()
>>> f = CommentForm(auto_id=False)
>>> print(f)
<tr><th>Your name:</th><td><input type="text" name="name" /></td></tr>
<tr><th>Your Web site:</th><td><input type="url" name="url" /></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" /></td></tr>

```

### label_suffix

`Field.``label_suffix`

New in Django 1.8.

`label_suffix` 参数让你基于每个字段覆盖表单的[`label_suffix`](api.html#django.forms.Form.label_suffix "django.forms.Form.label_suffix")：

```
>>> class ContactForm(forms.Form):
...     age = forms.IntegerField()
...     nationality = forms.CharField()
...     captcha_answer = forms.IntegerField(label='2 + 2', label_suffix=' =')
>>> f = ContactForm(label_suffix='?')
>>> print(f.as_p())
<p><label for="id_age">Age?</label> <input id="id_age" name="age" type="number" /></p>
<p><label for="id_nationality">Nationality?</label> <input id="id_nationality" name="nationality" type="text" /></p>
<p><label for="id_captcha_answer">2 + 2 =</label> <input id="id_captcha_answer" name="captcha_answer" type="number" /></p>

```

### unindent does not match any outer indentation level

`Field.``initial`

`initial` 参数让你指定渲染未绑定的`表单`中的`字段`时使用的初始值。

若要指定动态的初始数据，参见[`Form.initial`](api.html#django.forms.Form.initial "django.forms.Form.initial") 参数。

这个参数的使用场景是当你想要显示一个“空”的表单，其某个字段初始化为一个特定的值。例如：

```
>>> from django import forms
>>> class CommentForm(forms.Form):
...     name = forms.CharField(initial='Your name')
...     url = forms.URLField(initial='http://')
...     comment = forms.CharField()
>>> f = CommentForm(auto_id=False)
>>> print(f)
<tr><th>Name:</th><td><input type="text" name="name" value="Your name" /></td></tr>
<tr><th>Url:</th><td><input type="url" name="url" value="http://" /></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" /></td></tr>

```

你可能正在想为什么不在显示表单的时候传递一个包含初始化值的字典？如果这么做，你将触发验证过程，此时HTML 输出将包含任何验证中产生的错误：

```
>>> class CommentForm(forms.Form):
...     name = forms.CharField()
...     url = forms.URLField()
...     comment = forms.CharField()
>>> default_data = {'name': 'Your name', 'url': 'http://'}
>>> f = CommentForm(default_data, auto_id=False)
>>> print(f)
<tr><th>Name:</th><td><input type="text" name="name" value="Your name" /></td></tr>
<tr><th>Url:</th><td><ul class="errorlist"><li>Enter a valid URL.</li></ul><input type="url" name="url" value="http://" /></td></tr>
<tr><th>Comment:</th><td><ul class="errorlist"><li>This field is required.</li></ul><input type="text" name="comment" /></td></tr>

```

这就是为什么`initial` 的值只在未绑定的表单中显示的原因。对于绑定的表单，HTML 输出将使用绑定的数据。

还要注意，如果某个字段的值没有给出，`initial` 值_不_会作为“后备”的数据。`initial` 值_只_用于原始表单的显示：

```
>>> class CommentForm(forms.Form):
...     name = forms.CharField(initial='Your name')
...     url = forms.URLField(initial='http://')
...     comment = forms.CharField()
>>> data = {'name': '', 'url': '', 'comment': 'Foo'}
>>> f = CommentForm(data)
>>> f.is_valid()
False
# The form does *not* fall back to using the initial values.
>>> f.errors
{'url': ['This field is required.'], 'name': ['This field is required.']}

```

除了常数之外，你还可以传递一个可调用的对象：

```
>>> import datetime
>>> class DateForm(forms.Form):
...     day = forms.DateField(initial=datetime.date.today)
>>> print(DateForm())
<tr><th>Day:</th><td><input type="text" name="day" value="12/23/2008" /><td></tr>

```

可调用对象在未绑定的表单显示的时候才计算，不是在定义的时候。

### 窗口小部件

`Field.``widget`

`widget` 参数让你指定渲染`表单`时使用的`Widget` 类。更多信息参见[_Widgets_](widgets.html)。

### help_text

`Field.``help_text`

`help_text` 参数让你指定`字段`的描述文本。如果提供`help_text`，在通过`表单`的便捷方法（例如，`as_ul()`）渲染`字段`时，它将紧接着`字段`显示。

下面是一个完整的示例，`表单`为它的两个字段实现了`help_text`。 我们指定`auto_id=False`来让输出简单一些：

```
>>> from django import forms
>>> class HelpTextContactForm(forms.Form):
...     subject = forms.CharField(max_length=100, help_text='100 characters max.')
...     message = forms.CharField()
...     sender = forms.EmailField(help_text='A valid email address, please.')
...     cc_myself = forms.BooleanField(required=False)
>>> f = HelpTextContactForm(auto_id=False)
>>> print(f.as_table())
<tr><th>Subject:</th><td><input type="text" name="subject" maxlength="100" /><br /><span class="helptext">100 characters max.</span></td></tr>
<tr><th>Message:</th><td><input type="text" name="message" /></td></tr>
<tr><th>Sender:</th><td><input type="email" name="sender" /><br />A valid email address, please.</td></tr>
<tr><th>Cc myself:</th><td><input type="checkbox" name="cc_myself" /></td></tr>
>>> print(f.as_ul()))
<li>Subject: <input type="text" name="subject" maxlength="100" /> <span class="helptext">100 characters max.</span></li>
<li>Message: <input type="text" name="message" /></li>
<li>Sender: <input type="email" name="sender" /> A valid email address, please.</li>
<li>Cc myself: <input type="checkbox" name="cc_myself" /></li>
>>> print(f.as_p())
<p>Subject: <input type="text" name="subject" maxlength="100" /> <span class="helptext">100 characters max.</span></p>
<p>Message: <input type="text" name="message" /></p>
<p>Sender: <input type="email" name="sender" /> A valid email address, please.</p>
<p>Cc myself: <input type="checkbox" name="cc_myself" /></p>

```

### error_messages

`Field.``error_messages`

`error_messages` 参数让你覆盖字段引发的异常中的默认信息。传递的是一个字典，其键为你想覆盖的错误信息。例如，下面是默认的错误信息：

```
>>> from django import forms
>>> generic = forms.CharField()
>>> generic.clean('')
Traceback (most recent call last):
  ...
ValidationError: ['This field is required.']

```

而下面是自定义的错误信息：

```
>>> name = forms.CharField(error_messages={'required': 'Please enter your name'})
>>> name.clean('')
Traceback (most recent call last):
  ...
ValidationError: ['Please enter your name']

```

在下面的[内建的字段](#built-in-field-classes)一节中，每个`字段`都定义了它自己的错误信息。

### 验证器

`Field.``validators`

`validators` 参数让你可以为字段提供一个验证函数的列表。

更多的信息，参见[_validators 文档_](../validators.html)。

### 本地化

`Field.``localize`

`localize` 参数启用表单数据的本地化，包括输入和输出。

更多信息，参见[_本地化格式_](../../topics/i18n/formatting.html#format-localization)。

## 检查字段数据是否已经改变

### 已经改变()

`Field.``has_changed`()

Changed in Django 1.8:

重命名`_has_changed()` 为该方法。

`has_changed()` 方法用于决定字段的值是否从初始值发生了改变。返回`True` 或`False`。

更多信息，参见[`Form.has_changed()`](api.html#django.forms.Form.has_changed "django.forms.Form.has_changed")。

## 内建的`字段`

自然，`表单`的库会带有一系列表示常见需求的`字段`。 这一节记录每个内建字段。

对于每个字段，我们描述默认的`widget`。我们还会指出提供空值时的返回值（参见上文的`required` 以理解它的含义）。

### BooleanField

_class_ `BooleanField`(_**kwargs_)

*   默认的Widget：[`CheckboxInput`](widgets.html#django.forms.CheckboxInput "django.forms.CheckboxInput")
*   空值：`False`
*   规范化为：Python 的`True` 或 `False`。
*   如果字段带有`required=True`，验证值是否为`True`（例如复选框被勾上）。 
*   错误信息的键：`required`

注

因为所有的`Field` 子类都默认带有`required=True`，这里的验证条件很重要。如果你希望表单中包含一个既可以为`True` 也可以为`False` 的布尔值（例如，复选框可以勾上也可以不勾上），你必须要记住在创建`BooleanField`时传递`required=False`。

### CharField

_class_ `CharField`(_**kwargs_)

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`''`（一个空字符串）
*   规范化为：一个Unicode 对象。
*   如果提供，验证`max_length` 或`min_length`。 否则，所有的输入都是合法的。
*   错误信息的键：`required`, `max_length`, `min_length`

有两个参数用于验证：

`max_length`

`min_length`

如果提供，这两个参数将确保字符串的最大和最小长度。

### 选择字段

_class_ `ChoiceField`(_**kwargs_)

*   默认的Widget：[`Select`](widgets.html#django.forms.Select "django.forms.Select")
*   空值：`''`（一个空字符串）
*   规范化为：一个Unicode 对象。
*   验证给定的值在选项列表中存在。
*   错误信息的键：`required`, `invalid_choice`

`invalid_choice` 错误消息可能包含`%(value)s`，它将被选择的选项替换掉。

接收一个额外的必选参数：

`choices`

用来作为该字段选项的一个二元组组成的可迭代对象（例如，列表或元组）或者一个可调用对象。参数的格式与模型字段的`choices` 参数相同。更多细节参见[_模型字段参考中关于选项的文档_](../models/fields.html#field-choices)。如果参数是可调用的，它在字段的表单初始化时求值。

Changed in Django 1.8:

添加传递一个可调用对象给`choices` 的功能。

### TypedChoiceField

_class_ `TypedChoiceField`(_**kwargs_)

与[`ChoiceField`](#django.forms.ChoiceField "django.forms.ChoiceField") 很像，只是[`TypedChoiceField`](#django.forms.TypedChoiceField "django.forms.TypedChoiceField") 接受两个额外的参数`coerce` 和`empty_value`。

*   默认的Widget：[`Select`](widgets.html#django.forms.Select "django.forms.Select")
*   空值：`empty_value`
*   规范化为：`coerce` 参数类型的值。
*   验证给定的值在选项列表中存在并且可以被强制转换。
*   错误信息的键：`required`, `invalid_choice`

接收的额外参数：

`coerce`

接收一个参数并返回强制转换后的值的一个函数。例如内建的`int`、`float`、`bool` 和其它类型。默认为id 函数。注意强制转换在输入验证结束后发生，所以它可能强制转换不在 `choices` 中的值。

`empty_value`

用于表示“空”的值。默认为空字符串；`None` 是另外一个常见的选项。注意这个值不会被`coerce` 参数中指定的函数强制转换，所以请根据情况进行选择。

### DateField

_class_ `DateField`(_**kwargs_)

*   默认的Widget：[`DateInput`](widgets.html#django.forms.DateInput "django.forms.DateInput")
*   空值：`None`
*   规范化为：一个Python `datetime.date` 对象。
*   验证给出的值是一个`datetime.date`、`datetime.datetime` 或指定日期格式的字符串。
*   错误信息的键：`required`, `invalid`

接收一个可选的参数：

`input_formats`

一个格式的列表，用于转换一个字符串为`datetime.date` 对象。

如果没有提供`input_formats`，默认的输入格式为：

```
['%Y-%m-%d',      # '2006-10-25'
'%m/%d/%Y',       # '10/25/2006'
'%m/%d/%y']       # '10/25/06'

```

另外，如果你在设置中指定[`USE_L10N=False`](../settings.html#std:setting-USE_L10N)，以下的格式也将包含在默认的输入格式中：

```
['%b %d %Y',      # 'Oct 25 2006'
'%b %d, %Y',      # 'Oct 25, 2006'
'%d %b %Y',       # '25 Oct 2006'
'%d %b, %Y',      # '25 Oct, 2006'
'%B %d %Y',       # 'October 25 2006'
'%B %d, %Y',      # 'October 25, 2006'
'%d %B %Y',       # '25 October 2006'
'%d %B, %Y']      # '25 October, 2006'

```

另见[_本地化格式_](../../topics/i18n/formatting.html#format-localization)。

### DateTimeField

_class_ `DateTimeField`(_**kwargs_)

*   默认的Widget：[`DateTimeInput`](widgets.html#django.forms.DateTimeInput "django.forms.DateTimeInput")
*   空值：`None`
*   规范化为：一个Python `datetime.datetime` 对象。
*   验证给出的值是一个`datetime.date`、`datetime.datetime` 或指定日期格式的字符串。
*   错误信息的键：`required`, `invalid`

接收一个可选的参数：

`input_formats`

一个格式的列表，用于转换一个字符串为`datetime.datetime` 对象。

如果没有提供`input_formats`，默认的输入格式为：

```
['%Y-%m-%d %H:%M:%S',    # '2006-10-25 14:30:59'
'%Y-%m-%d %H:%M',        # '2006-10-25 14:30'
'%Y-%m-%d',              # '2006-10-25'
'%m/%d/%Y %H:%M:%S',     # '10/25/2006 14:30:59'
'%m/%d/%Y %H:%M',        # '10/25/2006 14:30'
'%m/%d/%Y',              # '10/25/2006'
'%m/%d/%y %H:%M:%S',     # '10/25/06 14:30:59'
'%m/%d/%y %H:%M',        # '10/25/06 14:30'
'%m/%d/%y']              # '10/25/06'

```

另见[_本地化格式_](../../topics/i18n/formatting.html#format-localization)。

自1.7版起已弃用：`DateTimeField` 与[`SplitDateTimeWidget`](widgets.html#django.forms.SplitDateTimeWidget "django.forms.SplitDateTimeWidget") 一起使用的功能被废弃并将在Django 1.9 中删除。请改用[`SplitDateTimeField`](#django.forms.SplitDateTimeField "django.forms.SplitDateTimeField")。

### DecimalField

_class_ `DecimalField`(_**kwargs_)

*   默认的Widget：当[`Field.localize`](#django.forms.Field.localize "django.forms.Field.localize") 是`False` 时为[`NumberInput`](widgets.html#django.forms.NumberInput "django.forms.NumberInput")，否则为[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")。
*   空值：`None`
*   规范化为：一个Python `decimal`。
*   验证给定的值为一个十进制数。忽略前导和尾随的空白。
*   错误信息的键：`required`, `invalid`, `max_value`, `min_value`, `max_digits`, `max_decimal_places`, `max_whole_digits`

`max_value` 和`min_value` 错误信息可能包含`%(limit_value)s`，它们将被真正的限制值替换。类似地，`max_digits`、`max_decimal_places` 和 `max_whole_digits` 错误消息可能包含`%(max)s`。

接收四个可选的参数：

`max_value`

`min_value`

它们控制字段中允许的值的范围，应该以`decimal.Decimal` 值给出。

`max_digits`

值允许的最大位数（小数点之前和之后的数字总共的位数，前导的零将被删除）。

`decimal_places`

允许的最大小数位。

### DurationField

New in Django 1.8.

_class_ `DurationField`(_**kwargs_)

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`None`
*   规范化为：一个Python [`timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta "(in Python v3.4)")。
*   验证给出的值是一个字符串，而可以给转换为`timedelta`。
*   错误信息的键：`required`, `invalid`.

接收任何可以被[`parse_duration()`](../utils.html#django.utils.dateparse.parse_duration "django.utils.dateparse.parse_duration") 理解的格式。

### EmailField

_class_ `EmailField`(_**kwargs_)

*   默认的Widget：[`EmailInput`](widgets.html#django.forms.EmailInput "django.forms.EmailInput")
*   空值：`''`（一个空字符串）
*   规范化为：一个Unicode 对象。
*   验证给出的值是一个合法的邮件地址，使用一个适度复杂的正则表达式。
*   错误信息的键：`required`, `invalid`

具有两个可选的参数用于验证，`max_length` 和`min_length`。如果提供，这两个参数确保字符串的最大和最小长度。

### FileField

_class_ `FileField`(_**kwargs_)

*   默认的Widget：[`ClearableFileInput`](widgets.html#django.forms.ClearableFileInput "django.forms.ClearableFileInput")
*   空值：`None`
*   规范化为：一个`UploadedFile` 对象，它封装文件内容和文件名为一个单独的对象。
*   可以验证非空的文件数据已经绑定到表单。
*   错误信息的键：`required`, `invalid`, `missing`, `empty`, `max_length`

具有两个可选的参数用于验证，`max_length` 和 `allow_empty_file`。如果提供，这两个参数确保文件名的最大长度，而且即使文件内容为空时验证也会成功。

若要了解`UploadedFile` 对象的更多内容，参见[_文件上传的文档_](../../topics/http/file-uploads.html)。

当你在表单中使用`FileField` 时，必须要记住[_绑定文件数据到表单上_](api.html#binding-uploaded-files)。

`max_length` 错误信息表示文件名的长度。在错误信息中，`%(max)d` 将替换为文件的最大长度，`%(length)d` 将替换为当前文件名的长度。

### FilePathField

_class_ `FilePathField`(_**kwargs_)

*   默认的Widget：[`Select`](widgets.html#django.forms.Select "django.forms.Select")
*   空值：`None`
*   规范化为：一个Unicode 对象。
*   验证选择的选项在选项列表中存在。
*   错误信息的键：`required`, `invalid_choice`

这个字段允许从一个特定的目录选择文件。它接受三个额外的参数；只有`path` 是必需的：

`path`

你想要列出的目录的绝对路径。这个目录必须存在。

`recursive`

如果为`False`（默认值），只用直接位于`path` 下的文件或目录作为选项。如果为`True`，将递归访问这个目录，其所有的子目录和文件都将作为选项。

`match`

正则表达式表示的一个模式；只有匹配这个表达式的名称才允许作为选项。

`allow_files`

可选。为`True` 或`False`。默认为`True`。表示是否应该包含指定位置的文件。它和[`allow_folders`](#django.forms.FilePathField.allow_folders "django.forms.FilePathField.allow_folders") 必须有一个为`True`。

`allow_folders`

可选。为`True` 或`False`。 默认为`False`。表示是否应该包含指定位置的目录。 它和[`allow_files`](#django.forms.FilePathField.allow_files "django.forms.FilePathField.allow_files") 必须有一个为`True`。

### FloatField

_class_ `FloatField`(_**kwargs_)

*   默认的Widget：当 [`Field.localize`](#django.forms.Field.localize "django.forms.Field.localize") 是`False` 时为[`NumberInput`](widgets.html#django.forms.NumberInput "django.forms.NumberInput")，否则为[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")。
*   空值：`None`
*   规范化为：一个Float 对象。
*   验证给出的值是一个浮点数。和Python 的`float()` 函数一样，允许前导和尾随的空白符。
*   错误信息的键：`required`, `invalid`, `max_value`, `min_value`

接收两个可选的参数用于验证，`max_value` 和`min_value`。它们控制字段中允许的值的范围。

### ImageField

_class_ `ImageField`(_**kwargs_)

*   默认的Widget：[`ClearableFileInput`](widgets.html#django.forms.ClearableFileInput "django.forms.ClearableFileInput")
*   空值：`None`
*   规范化为: An `UploadedFile` object that wraps the file content and file name into a single object.
*   验证文件数据已绑定到表单，并且该文件具有Pillow理解的图像格式。
*   错误信息的键：`required`, `invalid`, `missing`, `empty`, `invalid_image`

使用`ImageField`需要安装[Pillow](http://pillow.readthedocs.org/en/latest/)并支持您使用的图像格式。如果在上传图片时遇到`损坏 图像`错误，通常意味着Pillow不了解其格式。要解决这个问题，请安装相应的库并重新安装Pillow。

在表单上使用`ImageField`时，您还必须记住[_bind the file data to the form_](api.html#binding-uploaded-files)。

Changed in Django 1.8:

在字段清理和验证后，`UploadedFile`对象将有一个额外的`image`属性，包含Pillow [图像](https://pillow.readthedocs.org/en/latest/reference/Image.html)实例，用于检查文件是一个有效的图像。`UploadedFile.content_type`也会更新为由Pillow确定的图片内容类型。

### IntegerField

_class_ `IntegerField`(_**kwargs_)

*   默认的Widget：当[`Field.localize`](#django.forms.Field.localize "django.forms.Field.localize") 是`False` 时为[`NumberInput`](widgets.html#django.forms.NumberInput "django.forms.NumberInput")，否则为[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")。
*   空值：`None`
*   规范化为：一个Python 整数或长整数。
*   验证给定值是一个整数。允许前导和尾随空格，如Python的`int()`函数。
*   错误信息的键：`required`, `invalid`, `max_value`, `min_value`

`max_value`和`min_value`错误消息可能包含`%(limit_value)s`，将替换为适当的限制。

采用两个可选参数进行验证：

`max_value`

`min_value`

它们控制字段中允许的值的范围。

### IPAddressField

_class_ `IPAddressField`(_**kwargs_)

自1.7版起已弃用：此字段已弃用，支持[`GenericIPAddressField`](#django.forms.GenericIPAddressField "django.forms.GenericIPAddressField")。

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`''`（一个空字符串）
*   规范化为：一个Unicode 对象。
*   使用正则表达式验证给定值是有效的IPv4地址。
*   错误信息的键：`required`, `invalid`

### GenericIPAddressField

_class_ `GenericIPAddressField`(_**kwargs_)

包含IPv4或IPv6地址的字段。

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`''`（一个空字符串）
*   规范化为：一个Unicode 对象。 IPv6地址如下所述进行归一化。
*   验证给定值是有效的IP地址。
*   错误信息的键：`required`, `invalid`

IPv6地址规范化遵循 [**RFC 4291**](http://tools.ietf.org/html/rfc4291.html#section-2.2)部分2.2，包括使用该部分第3段中建议的IPv4格式，如`::ffff:192.0.2.0`例如，`2001:0::0:01`将被标准化为`2001::1`和`::ffff:0a0a:0a0a` `::ffff:10.10.10.10`。所有字符都转换为小写。

有两个可选参数：

`protocol`

限制指定协议的有效输入。接受的值为`both`（默认值），`IPv4`或`IPv6`。匹配不区分大小写。

`unpack_ipv4`

解开IPv4映射地址，例如`::ffff:192.0.2.1`。如果启用此选项，则该地址将解包到`192.0.2.1`。默认为禁用。只能在`protocol`设置为`'both'`时使用。

### MultipleChoiceField

_class_ `MultipleChoiceField`(_**kwargs_)

*   默认的Widget：[`SelectMultiple`](widgets.html#django.forms.SelectMultiple "django.forms.SelectMultiple")
*   空值：`[]`（一个空列表）
*   规范化为：一个Unicode 对象列表。
*   验证给定值列表中的每个值都存在于选择列表中。
*   错误信息的键：`required`, `invalid_choice`, `invalid_list`

`invalid_choice`错误消息可能包含`%(value)s`，将替换为所选择的选项。

对于`ChoiceField`，需要一个额外的必需参数`choices`。

### TypedMultipleChoiceField

_class_ `TypedMultipleChoiceField`(_**kwargs_)

就像[`MultipleChoiceField`](#django.forms.MultipleChoiceField "django.forms.MultipleChoiceField")，除了[`TypedMultipleChoiceField`](#django.forms.TypedMultipleChoiceField "django.forms.TypedMultipleChoiceField")需要两个额外的参数，`coerce`和`empty_value`。

*   默认的Widget：[`SelectMultiple`](widgets.html#django.forms.SelectMultiple "django.forms.SelectMultiple")
*   空值：`empty_value`
*   规范化为：`coerce`参数提供的类型值列表。
*   验证给定值存在于选项列表中并且可以强制。
*   错误信息的键：`required`, `invalid_choice`

`invalid_choice`错误消息可能包含`%(value)s`，将替换为所选择的选项。

对于`TypedChoiceField`，需要两个额外的参数`coerce`和`empty_value`。

### NullBooleanField

_class_ `NullBooleanField`(_**kwargs_)

*   默认的Widget：[`NullBooleanSelect`](widgets.html#django.forms.NullBooleanSelect "django.forms.NullBooleanSelect")
*   空值：`None`
*   规范化为：一个Python `True`, `False` 或`None` 值。
*   不验证任何内容（即，它从不引发`ValidationError`）。

### RegexField

_class_ `RegexField`(_**kwargs_)

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`''`（一个空字符串）
*   规范化为：一个Unicode对象
*   验证给定值与某个正则表达式匹配。
*   错误信息的键：`required`, `invalid`

需要一个必需的参数：

`regex`

指定为字符串或编译的正则表达式对象的正则表达式。

也采用`max_length`和`min_length`，它们的作用与`CharField`相同。

自1.8版起已弃用：可接受的参数`error_message`也被接受用于向后兼容性，但会在Django 2.0中删除。提供错误消息的首选方法是使用[`error_messages`](#django.forms.Field.error_messages "django.forms.Field.error_messages")参数，传递具有`'invalid'`的字典作为键，并将错误消息作为值。

### SlugField

_class_ `SlugField`(_**kwargs_)

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`''`（一个空字符串）
*   规范化为：一个Unicode 对象。
*   验证给定的字符串只包括字母、数字、下划线及连字符。
*   错误信息的键：`required`, `invalid`

此字段用于在表单中表示模型[`SlugField`](../models/fields.html#django.db.models.SlugField "django.db.models.SlugField")。

### TimeField

_class_ `TimeField`(_**kwargs_)

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`None`
*   规范化为：一个Python 的`datetime.time` 对象。
*   验证给定值是`datetime.time`或以特定时间格式格式化的字符串。
*   错误信息的键：`required`, `invalid`

采用一个可选参数：

`input_formats`

用于尝试将字符串转换为有效的`datetime.time`对象的格式列表。

如果未提供`input_formats`参数，则默认输入格式为：

```
'%H:%M:%S',     # '14:30:59'
'%H:%M',        # '14:30'

```

### URLField

_class_ `URLField`(_**kwargs_)

*   默认的Widget：[`URLInput`](widgets.html#django.forms.URLInput "django.forms.URLInput")
*   空值：`''`（一个空字符串）
*   规范化为：一个Unicode 对象。
*   验证给定值是有效的URL。
*   错误信息的键：`required`, `invalid`

采用以下可选参数：

`max_length`

`min_length`

这些与`CharField.max_length`和`CharField.min_length`相同。

### UUIDField

New in Django 1.8.

_class_ `UUIDField`(_**kwargs_)

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`''`（一个空字符串）
*   规范化为：一个[`UUID`](https://docs.python.org/3/library/uuid.html#uuid.UUID "(in Python v3.4)") 对象。
*   错误信息的键：`required`, `invalid`

此字段将接受任何作为[`UUID`](https://docs.python.org/3/library/uuid.html#uuid.UUID "(in Python v3.4)")构造函数的`hex`参数接受的字符串格式。

## 稍微复杂一点的内建 `Field` 类

### ComboField

_class_ `ComboField`(_**kwargs_)

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`''`（一个空字符串）
*   规范化为：一个Unicode 对象。
*   根据指定为`ComboField`的参数的每个字段验证给定值。
*   错误信息的键：`required`, `invalid`

需要一个额外的必需参数：

`fields`

应用于验证字段值的字段列表（按提供它们的顺序）。

```
&gt;&gt;&gt; from django.forms import ComboField
&gt;&gt;&gt; f = ComboField(fields=[CharField(max_length=20), EmailField()])
&gt;&gt;&gt; f.clean('test@example.com')
'test@example.com'
&gt;&gt;&gt; f.clean('longemailaddress@example.com')
Traceback (most recent call last):
...
ValidationError: ['Ensure this value has at most 20 characters (it has 28).']

```

### MultiValueField

_class_ `MultiValueField`(_fields=()_, _**kwargs_)

*   默认的Widget：[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")
*   空值：`''`（一个空字符串）
*   规范化为：子类的`compress`方法返回的类型。
*   针对指定为`MultiValueField`的参数的每个字段验证给定值。
*   错误信息的键：`required`, `invalid`, `incomplete`

聚合共同产生单个值的多个字段的逻辑。

此字段是抽象的，必须是子类。与单值字段相反，[`MultiValueField`](#django.forms.MultiValueField "django.forms.MultiValueField")的子类不能实现[`clean()`](#django.forms.Field.clean "django.forms.Field.clean")，而是实现[`compress()`](#django.forms.MultiValueField.compress "django.forms.MultiValueField.compress")。

需要一个额外的必需参数：

`fields`

字段的元组，其值被清除并随后组合成单个值。每个字段的值由`字段中的相应字段清除` - 第一个值由第一个字段清除，第二个值由第二个字段清除等。清除所有字段后，通过[`compress()`](#django.forms.MultiValueField.compress "django.forms.MultiValueField.compress")将干净值列表合并为一个值。

还需要一个额外的可选参数：

`require_all_fields`

New in Django 1.7.

默认为`True`，在这种情况下，如果没有为任何字段提供值，则会出现`required`验证错误。

设置为`False`时，可以将[`Field.required`](#django.forms.Field.required "django.forms.Field.required")属性设置为`False`，以使其为可选字段。如果没有为必填字段提供值，则会出现`不完整`验证错误。

可以在[`MultiValueField`](#django.forms.MultiValueField "django.forms.MultiValueField")子类上定义默认`不完整`错误消息，或者可以在每个单独字段上定义不同的消息。例如：

```
from django.core.validators import RegexValidator

class PhoneField(MultiValueField):
    def __init__(self, *args, **kwargs):
        # Define one message for all fields.
        error_messages = {
            'incomplete': 'Enter a country calling code and a phone number.',
        }
        # Or define a different message for each field.
        fields = (
            CharField(error_messages={'incomplete': 'Enter a country calling code.'},
                      validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid country calling code.')]),
            CharField(error_messages={'incomplete': 'Enter a phone number.'},
                      validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid phone number.')]),
            CharField(validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid extension.')],
                      required=False),
        )
        super(PhoneField, self).__init__(
            error_messages=error_messages, fields=fields,
            require_all_fields=False, *args, **kwargs)

```

`widget`

必须是[`django.forms.MultiWidget`](widgets.html#django.forms.MultiWidget "django.forms.MultiWidget")的子类。默认值为[`TextInput`](widgets.html#django.forms.TextInput "django.forms.TextInput")，在这种情况下可能不是非常有用。

`compress`(_data_list_)

获取有效值的列表，并在单个值中返回这些值的“压缩”版本。例如，[`SplitDateTimeField`](#django.forms.SplitDateTimeField "django.forms.SplitDateTimeField")是将时间字段和日期字段合并为`datetime`对象的子类。

此方法必须在子类中实现。

### SplitDateTimeField

_class_ `SplitDateTimeField`(_**kwargs_)

*   默认的Widget：[`SplitDateTimeWidget`](widgets.html#django.forms.SplitDateTimeWidget "django.forms.SplitDateTimeWidget")
*   空值：`None`
*   规范化为：一个Python `datetime.datetime` 对象。
*   验证给定的值是`datetime.datetime`或以特定日期时间格式格式化的字符串。
*   错误信息的键：`required`, `invalid`, `invalid_date`, `invalid_time`

有两个可选参数：

`input_date_formats`

用于尝试将字符串转换为有效的`datetime.date`对象的格式列表。

如果未提供`input_date_formats`参数，则会使用`DateField`的默认输入格式。

`input_time_formats`

用于尝试将字符串转换为有效的`datetime.time`对象的格式列表。

如果未提供`input_time_formats`参数，则使用`TimeField`的默认输入格式。

## 处理关系的字段

两个字段可用于表示模型之间的关系：[`ModelChoiceField`](#django.forms.ModelChoiceField "django.forms.ModelChoiceField")和[`ModelMultipleChoiceField`](#django.forms.ModelMultipleChoiceField "django.forms.ModelMultipleChoiceField")。这两个字段都需要单个`queryset`参数，用于创建字段的选择。在表单验证时，这些字段将把一个模型对象（在`ModelChoiceField`的情况下）或多个模型对象（在`ModelMultipleChoiceField`的情况下）放置到`cleaned_data`表单的字典。

对于更复杂的用法，可以在声明表单字段时指定`queryset=None`，然后在窗体的`__init__()`方法中填充`queryset`

```
class FooMultipleChoiceForm(forms.Form):
    foo_select = forms.ModelMultipleChoiceField(queryset=None)

    def __init__(self, *args, **kwargs):
        super(FooMultipleChoiceForm, self).__init__(*args, **kwargs)
        self.fields['foo_select'].queryset = ...

```

### ModelChoiceField

_class_ `ModelChoiceField`(_**kwargs_)

*   默认的Widget：[`Select`](widgets.html#django.forms.Select "django.forms.Select")
*   空值：`None`
*   规范化为：一个模型实例。
*   验证给定的id存在于查询集中。
*   错误信息的键：`required`, `invalid_choice`

可以选择一个单独的模型对像，适用于表示一个外键字段。 `ModelChoiceField`默认widet不适用选择数量很大的情况，在大于100项时应该避免使用它。

需要单个参数：

`queryset`

将导出字段选择的模型对象的`QuerySet`，将用于验证用户的选择。

`ModelChoiceField`也有两个可选参数：

`empty_label`

默认情况下，`ModelChoiceField`使用的`&lt;select&gt;`小部件将在列表顶部有一个空选项。您可以使用`empty_label`属性更改此标签的文本（默认为`"---------"`），也可以禁用空白标签完全通过将`empty_label`设置为`None`：

```
# A custom empty label
field1 = forms.ModelChoiceField(queryset=..., empty_label="(Nothing)")

# No empty label
field2 = forms.ModelChoiceField(queryset=..., empty_label=None)

```

请注意，如果需要`ModelChoiceField`并且具有默认初始值，则不会创建空选项（不管`empty_label`的值）。

`to_field_name`

此可选参数用于指定要用作字段窗口小部件中选项的值的字段。确保它是模型的唯一字段，否则选定的值可以匹配多个对象。默认情况下，它设置为`None`，在这种情况下，将使用每个对象的主键。例如：

```
# No custom to_field_name
field1 = forms.ModelChoiceField(queryset=...)

```

将产生：

```
&lt;select id="id_field1" name="field1"&gt;
&lt;option value="obj1.pk"&gt;Object1&lt;/option&gt;
&lt;option value="obj2.pk"&gt;Object2&lt;/option&gt;
...
&lt;/select&gt;

```

和：

```
# to_field_name provided
field2 = forms.ModelChoiceField(queryset=..., to_field_name="name")

```

将产生：

```
&lt;select id="id_field2" name="field2"&gt;
&lt;option value="obj1.name"&gt;Object1&lt;/option&gt;
&lt;option value="obj2.name"&gt;Object2&lt;/option&gt;
...
&lt;/select&gt;

```

The `__str__` (`__unicode__` on Python 2) method of the model will be called to generate string representations of the objects for use in the field’s choices; 提供自定义表示，子类`ModelChoiceField`并覆盖`label_from_instance`。此方法将接收一个模型对象，并应返回一个适合表示它的字符串。例如：

```
from django.forms import ModelChoiceField

class MyModelChoiceField(ModelChoiceField):
    def label_from_instance(self, obj):
        return "My Object #%i" % obj.id

```

### ModelMultipleChoiceField

_class_ `ModelMultipleChoiceField`(_**kwargs_)

*   默认的Widget：[`SelectMultiple`](widgets.html#django.forms.SelectMultiple "django.forms.SelectMultiple")
*   空值：`QuerySet` (self.queryset.none())
*   规范化为： 模型实例的一个`QuerySet`。
*   验证在给定的值列表中的每个id存在于查询集中。
*   错误信息的键：`required`, `list`, `invalid_choice`, `invalid_pk_value`

`invalid_choice`消息可以包含`%(value)s`并且`invalid_pk_value`消息可以包含`%(pk)s`其将被适当的值代替。

允许选择适合于表示多对多关系的一个或多个模型对象。与[`ModelChoiceField`](#django.forms.ModelChoiceField "django.forms.ModelChoiceField")一样，您可以使用`label_from_instance`自定义对象表示，`queryset`是必需的参数：

`queryset`

将导出字段选择的模型对象的`QuerySet`，将用于验证用户的选择。

## 创建自定义的字段

如果内建的`字段`不能满足你的需求，你可以很容易地创建自定义的`字段`。你需要创建`django.forms.Field` 的一个子类。它只要求实现一个`clean()` 方法和接收上面核心参数的`__init__()` 方法(`required`, `label`, `initial`, `widget`, `help_text`)。

