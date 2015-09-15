# Widgets

Widget 是Django 对HTML 输入元素的表示。Widget 负责渲染HTML和提取GET/POST 字典中的数据。

小贴士

不要将Widget 与[_表单字段_](fields.html)搞混淆。表单字段负责验证输入并直接在模板中使用。Widget 负责渲染网页上HTML 表单的输入元素和提取提交的原始数据。但是，Widget 需要[_赋值_](#widget-to-field)给表单的字段。

## 指定Widget

每当你指定表单的一个字段的时候，Django 将使用适合其数据类型的默认Widget。若要查找每个字段使用的Widget，参见[_内建的字段_](fields.html#built-in-fields)文档。

然而，如果你想要使用一个不同的Widget，你可以在定义字段时使用[`widget`](fields.html#django.forms.Field.widget "django.forms.Field.widget") 参数。例如：

```
from django import forms

class CommentForm(forms.Form):
    name = forms.CharField()
    url = forms.URLField()
    comment = forms.CharField(widget=forms.Textarea)

```

这将使用一个[`Textarea`](#django.forms.Textarea "django.forms.Textarea") Widget来设置表单的评论&nbsp;，而不是默认的[`TextInput`](#django.forms.TextInput "django.forms.TextInput") Widget。

## 设置Widget 的参数

很多Widget 都有可选的参数；它们可以在定义字段的Widget 时设置。在下面的示例中，设置了[`SelectDateWidget`](#django.forms.extras.widgets.SelectDateWidget "django.forms.extras.widgets.SelectDateWidget") 的[`years`](#django.forms.extras.widgets.SelectDateWidget.years "django.forms.extras.widgets.SelectDateWidget.years") 属性：

```
from django import forms
from django.forms.extras.widgets import SelectDateWidget

BIRTH_YEAR_CHOICES = ('1980', '1981', '1982')
FAVORITE_COLORS_CHOICES = (('blue', 'Blue'),
                            ('green', 'Green'),
                            ('black', 'Black'))

class SimpleForm(forms.Form):
    birth_year = forms.DateField(widget=SelectDateWidget(years=BIRTH_YEAR_CHOICES))
    favorite_colors = forms.MultipleChoiceField(required=False,
        widget=forms.CheckboxSelectMultiple, choices=FAVORITE_COLORS_CHOICES)

```

可用的Widget 以及它们接收的参数，参见[_内建的Widget_](#built-in-widgets)。

## 继承自Select 的Widget

继承自[`Select`](#django.forms.Select "django.forms.Select") 的Widget 负责处理HTML 选项。它们呈现给用户一个可以选择的选项列表。不同的Widget 以不同的方式呈现选项；[`Select`](#django.forms.Select "django.forms.Select") 使用HTML 的列表形式`<select>`，而[`RadioSelect`](#django.forms.RadioSelect "django.forms.RadioSelect") 使用单选按钮。

[`ChoiceField`](fields.html#django.forms.ChoiceField "django.forms.ChoiceField") 字段默认使用[`Select`](#django.forms.Select "django.forms.Select")。Widget 上显示的选项来自[`ChoiceField`](fields.html#django.forms.ChoiceField "django.forms.ChoiceField")，对[`ChoiceField.choices`](fields.html#django.forms.ChoiceField.choices "django.forms.ChoiceField.choices") 的改变将更新[`Select.choices`](#django.forms.Select.choices "django.forms.Select.choices")。例如：

```
>>> from django import forms
>>> CHOICES = (('1', 'First',), ('2', 'Second',))
>>> choice_field = forms.ChoiceField(widget=forms.RadioSelect, choices=CHOICES)
>>> choice_field.choices
[('1', 'First'), ('2', 'Second')]
>>> choice_field.widget.choices
[('1', 'First'), ('2', 'Second')]
>>> choice_field.widget.choices = ()
>>> choice_field.choices = (('1', 'First and only',),)
>>> choice_field.widget.choices
[('1', 'First and only')]

```

提供[`choices`](#django.forms.Select.choices "django.forms.Select.choices") 属性的Widget 也可以用于不是基于选项的字段 ， 例如[`CharField`](fields.html#django.forms.CharField "django.forms.CharField") —— 当选项与模型有关而不只是Widget 时，建议使用基于[`ChoiceField`](fields.html#django.forms.ChoiceField "django.forms.ChoiceField") 的字段。

## 自定义Widget 的实例

当Django 渲染Widget 成HTML 时，它只渲染最少的标记 —— Django 不会添加class 的名称和特定于Widget 的其它属性。这表示，网页上所有[`TextInput`](#django.forms.TextInput "django.forms.TextInput") 的外观是一样的。

有两种自定义Widget 的方式：基于每个[_Widget 实例_](#styling-widget-instances)和基于每个[_Widget 类_](#styling-widget-classes)。

### 设置Widget 实例的样式

如果你想让某个Widget 实例与其它Widget 看上去不一样，你需要在Widget 对象实例化并赋值给一个表单字段时指定额外的属性（以及可能需要在你的CSS 文件中添加一些规则）。

例如下面这个简单的表单：

```
from django import forms

class CommentForm(forms.Form):
    name = forms.CharField()
    url = forms.URLField()
    comment = forms.CharField()

```

这个表单包含三个默认的[`TextInput`](#django.forms.TextInput "django.forms.TextInput") Widget，以默认的方式渲染 —— 没有CSS 类、没有额外的属性。这表示每个Widget 的输入框将渲染得一模一样：

```
>>> f = CommentForm(auto_id=False)
>>> f.as_table()
<tr><th>Name:</th><td><input type="text" name="name" /></td></tr>
<tr><th>Url:</th><td><input type="url" name="url"/></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" /></td></tr>

```

在真正得网页中，你可能不想让每个Widget 看上去都一样。你可能想要给comment 一个更大的输入元素，你可能想让‘name’ Widget 具有一些特殊的CSS 类。可以指定‘type’ 属性来利用新式的HTML5 输入类型。在创建Widget 时使用[`Widget.attrs`](#django.forms.Widget.attrs "django.forms.Widget.attrs") 参数可以实现：

```
class CommentForm(forms.Form):
    name = forms.CharField(widget=forms.TextInput(attrs={'class': 'special'}))
    url = forms.URLField()
    comment = forms.CharField(widget=forms.TextInput(attrs={'size': '40'}))

```

Django 将在渲染的输出中包含额外的属性：

```
>>> f = CommentForm(auto_id=False)
>>> f.as_table()
<tr><th>Name:</th><td><input type="text" name="name" class="special"/></td></tr>
<tr><th>Url:</th><td><input type="url" name="url"/></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" size="40"/></td></tr>

```

你还可以使用[`attrs`](#django.forms.Widget.attrs "django.forms.Widget.attrs") 设置HTML `id`。参见[`BoundField.id_for_label`](api.html#django.forms.BoundField.id_for_label "django.forms.BoundField.id_for_label") 示例。

### 设置Widget 类的样式

可以添加（`css` 和`javascript`）给Widget，以及深度定制它们的外观和行为。

概况来讲，你需要子类化Widget 并[_定义一个“Media” 内联类_](../../topics/forms/media.html#assets-as-a-static-definition) 或 [_创建一个“media” 属性_](../../topics/forms/media.html#dynamic-property)。

这些方法涉及到Python 高级编程，详细细节在[_表单Assets_](../../topics/forms/media.html) 主题中讲述。

## Widget 的基类

[`Widget`](#django.forms.Widget "django.forms.Widget") 和[`MultiWidget`](#django.forms.MultiWidget "django.forms.MultiWidget") 是所有[_内建Widget_](#built-in-widgets) 的基类，并可用于自定义Widget 的基类。

_class _`Widget`(_attrs=None_)

这是个抽象类，它不可以渲染，但是提供基本的属性[`attrs`](#django.forms.Widget.attrs "django.forms.Widget.attrs")。你可以在自定义的Widget 中实现或覆盖[`render()`](#django.forms.Widget.render "django.forms.Widget.render") 方法。

`attrs`

包含渲染后的Widget 将要设置的HTML 属性。

```
>>> from django import forms
>>> name = forms.TextInput(attrs={'size': 10, 'title': 'Your name',})
>>> name.render('name', 'A name')
'<input title="Your name" type="text" name="name" value="A name" size="10" />'

```

Changed in Django 1.8:

如果你给一个属性赋值`True` 或`False`，它将渲染成一个HTML5 风格的布尔属性：

```
>>> name = forms.TextInput(attrs={'required': True})
>>> name.render('name', 'A name')
'<input name="name" type="text" value="A name" required />'
>>>
>>> name = forms.TextInput(attrs={'required': False})
>>> name.render('name', 'A name')
'<input name="name" type="text" value="A name" />'

```

`render`(_name_, _value_, _attrs=None_)

返回Widget 的HTML，为一个Unicode 字符串。子类必须实现这个方法，否则将引发`NotImplementedError`。

它不会确保给出的‘value’ 是一个合法的输入，因此子类的实现应该防卫式地编程。

`value_from_datadict`(_data_, _files_, _name_)

根据一个字典和该Widget 的名称，返回该Widget 的值。`files` may contain data coming from [`request.`](../request-response.html#django.http.HttpRequest.FILES "django.http.HttpRequest.FILES")FILES. 如果没有提供value，则返回`None`。 在处理表单数据的过程中，`value_from_datadict` 可能调用多次，所以如果你自定义并添加额外的耗时处理时，你应该自己实现一些缓存机制。

_class _`MultiWidget`(_widgets_, _attrs=None_)

由多个Widget 组合而成的Widget。[`MultiWidget`](#django.forms.MultiWidget "django.forms.MultiWidget") 始终与[`MultiValueField`](fields.html#django.forms.MultiValueField "django.forms.MultiValueField") 联合使用。

[`MultiWidget`](#django.forms.MultiWidget "django.forms.MultiWidget") 具有一个必选参数：

`widgets`

一个包含需要的Widget 的可迭代对象。

以及一个必需的方法：

`decompress`(_value_)

这个方法接受来自字段的一个“压缩”的值，并返回“解压”的值的一个列表。可以假设输入的值是合法的，但不一定是非空的。

子类**必须实现** 这个方法，而且因为值可能为空，实现必须要防卫这点。

“解压”的基本原理是需要“分离”组合的表单字段的值为每个Widget 的值。

有个例子是，[`SplitDateTimeWidget`](#django.forms.SplitDateTimeWidget "django.forms.SplitDateTimeWidget") 将[`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)") 值分离成两个独立的值分别表示日期和时间：

```
from django.forms import MultiWidget

class SplitDateTimeWidget(MultiWidget):

    # ...

    def decompress(self, value):
        if value:
            return [value.date(), value.time().replace(microsecond=0)]
        return [None, None]

```

小贴士

注意，[`MultiValueField`](fields.html#django.forms.MultiValueField "django.forms.MultiValueField") 有一个[`compress()`](fields.html#django.forms.MultiValueField.compress "django.forms.MultiValueField.compress") 方法用于相反的工作 —— 将所有字段的值组合成一个值。

其它可能需要覆盖的方法：

`render`(_name_, _value_, _attrs=None_)

这个方法中的&nbsp;`value`参数的处理方式与[`Widget`](#django.forms.Widget "django.forms.Widget")子类不同，因为需要弄清楚如何为了在不同widget中展示分割单一值。

渲染中使用的`value`参数可以是二者之一：

*   一个`列表`。
*   一个单一值（比如字符串），它是`列表`的“压缩”表现形式。

如果`value`是个列表，[`render()`](#django.forms.MultiWidget.render "django.forms.MultiWidget.render")的输出会是一系列渲染后的子widget。如果`value`不是一个列表，首先会通过[`decompress()`](#django.forms.MultiWidget.decompress "django.forms.MultiWidget.decompress")方法来预处理，创建列表，之后再渲染。

`render()`方法执行HTML渲染时，列表中的每个值都使用相应的widget来渲染 -- 第一个值在第一个widget中渲染，第二个值在第二个widget中渲染，以此类推。

不像单一值的widget，[`render()`](#django.forms.MultiWidget.render "django.forms.MultiWidget.render") 方法并不需要在子类中实现。

`format_output`(_rendered_widgets_)

接受选然后的widget（以字符串形式）的一个列表，返回表示全部HTML的Unicode字符串。

这个钩子允许你以任何你想要的方式，格式化widget的HTML设计。

下面示例中的Widget 继承[`MultiWidget`](#django.forms.MultiWidget "django.forms.MultiWidget") 以在不同的选择框中显示年、月、日。这个Widget 主要想用于[`DateField`](fields.html#django.forms.DateField "django.forms.DateField") 而不是[`MultiValueField`](fields.html#django.forms.MultiValueField "django.forms.MultiValueField")，所以我们实现了[`value_from_datadict()`](#django.forms.Widget.value_from_datadict "django.forms.Widget.value_from_datadict")：

```
from datetime import date
from django.forms import widgets

class DateSelectorWidget(widgets.MultiWidget):
    def __init__(self, attrs=None):
        # create choices for days, months, years
        # example below, the rest snipped for brevity.
        years = [(year, year) for year in (2011, 2012, 2013)]
        _widgets = (
            widgets.Select(attrs=attrs, choices=days),
            widgets.Select(attrs=attrs, choices=months),
            widgets.Select(attrs=attrs, choices=years),
        )
        super(DateSelectorWidget, self).__init__(_widgets, attrs)

    def decompress(self, value):
        if value:
            return [value.day, value.month, value.year]
        return [None, None, None]

    def format_output(self, rendered_widgets):
        return ''.join(rendered_widgets)

    def value_from_datadict(self, data, files, name):
        datelist = [
            widget.value_from_datadict(data, files, name + '_%s' % i)
            for i, widget in enumerate(self.widgets)]
        try:
            D = date(day=int(datelist[0]), month=int(datelist[1]),
                    year=int(datelist[2]))
        except ValueError:
            return ''
        else:
            return str(D)

```

构造器在一个元组中创建了多个[`Select`](#django.forms.Select "django.forms.Select") widget。`超`类使用这个元组来启动widget。

[`format_output()`](#django.forms.MultiWidget.format_output "django.forms.MultiWidget.format_output")方法相当于在这里没有干什么新的事情（实际上，它和`MultiWidget`中默认实现的东西相同），但是这个想法是，你可以以自己的方式在widget之间添加自定义的HTML。

必需的[`decompress()`](#django.forms.MultiWidget.decompress "django.forms.MultiWidget.decompress")方法将`datetime.date` 值拆成年、月和日的值，对应每个widget。注意这个方法如何处理`value`为`None`的情况。

[`value_from_datadict()`](#django.forms.Widget.value_from_datadict "django.forms.Widget.value_from_datadict")的默认实现会返回一个列表，对应每一个`Widget`。当和[`MultiValueField`](fields.html#django.forms.MultiValueField "django.forms.MultiValueField")一起使用`MultiWidget`的时候，这样会非常合理，但是由于我们想要和拥有单一值得[`DateField`](fields.html#django.forms.DateField "django.forms.DateField")一起使用这个widget，我们必须覆写这一方法，将所有子widget的数据组装成`datetime.date`。这个方法从`POST` 字典中获取数据，并且构造和验证日期。如果日期有效，会返回它的字符串，否则会返回一个空字符串，它会使`form.is_valid`返回`False`。

## 内建的Widget

Django 提供所有基本的HTML Widget，并在`django.forms.widgets` 模块中提供一些常见的Widget 组，包括[_文本的输入_](#text-widgets)、[_各种选择框_](#selector-widgets)、[_文件上传_](#file-upload-widgets)和[_多值输入_](#composite-widgets)。

### 处理文本输入的Widget

这些Widget 使用HTML 元素`input` 和 `textarea`。

#### TextInput

_class _`TextInput`

文本输入：`<input type="text" ...>`

#### NumberInput

_class _`NumberInput`

文本输入：`<input type="number" ...>`

注意，不是所有浏览器的`number`输入类型都支持输入本地化的数字。Django 将字段的[`localize`](fields.html#django.forms.Field.localize "django.forms.Field.localize") 属性设置为`True` 以避免字段使用它们。

#### EmailInput

_class _`EmailInput`

文本输入：`<input type="email" ...>`

#### URLInput

_class _`URLInput`

文本输入：`<input type="url" ...>`

#### PasswordInput

_class _`PasswordInput`

密码输入：`<input type='password' ...>`

接收一个可选的参数：

`render_value`

决定在验证错误后重新显示表单时，Widget 是否填充（默认为`False`）。

#### HiddenInput

_class _`HiddenInput`

隐藏的输入：`<input type='hidden' ...>`

注意，还有一个[`MultipleHiddenInput`](#django.forms.MultipleHiddenInput "django.forms.MultipleHiddenInput") Widget，它封装一组隐藏的输入元素。

#### DateInput

_class _`DateInput`

日期以普通的文本框输入：`<input type='text' ...>`

接收的参数与[`TextInput`](#django.forms.TextInput "django.forms.TextInput") 相同，但是带有一些可选的参数：

`format`

字段的初始值应该显示的格式。

如果没有提供`format` 参数，默认的格式为参考[_本地化格式_](../../topics/i18n/formatting.html#format-localization)在[`DATE_INPUT_FORMATS`](../settings.html#std:setting-DATE_INPUT_FORMATS) 中找到的第一个格式。

#### DateTimeInput

_class _`DateTimeInput`

日期/时间以普通的文本框输入：`<input type='text' ...>`

接收的参数与[`TextInput`](#django.forms.TextInput "django.forms.TextInput") 相同，但是带有一些可选的参数：

`format`

字段的初始值应该显示的格式。

如果没有提供`format` 参数，默认的格式为参考[_本地化格式_](../../topics/i18n/formatting.html#format-localization)在[`DATETIME_INPUT_FORMATS`](../settings.html#std:setting-DATETIME_INPUT_FORMATS) 中找到的第一个格式。

#### TimeInput

_class _`TimeInput`

时间以普通的文本框输入：`<input type='text' ...>`

接收的参数与[`TextInput`](#django.forms.TextInput "django.forms.TextInput") 相同，但是带有一些可选的参数：

`format`

字段的初始值应该显示的格式。

如果没有提供`format` 参数，默认的格式为参考[_本地化格式_](../../topics/i18n/formatting.html#format-localization)在[`TIME_INPUT_FORMATS`](../settings.html#std:setting-TIME_INPUT_FORMATS) 中找到的第一个格式。

#### Textarea

_class _`Textarea`

文本区域：`<textarea>...</textarea>`

### 选择和复选框Widget

#### CheckboxInput

_class _`CheckboxInput`

复选框：`<input type='checkbox' ...>`

接受一个可选的参数：

`check_test`

一个可调用的对象，接收`CheckboxInput` 的值并如果复选框应该勾上返回`True`。

#### Select

_class _`Select`

Select widget：`<select><option ...>...</select>`

`choices`

当表单字段没有`choices` 属性时，该属性是随意的。如果字段有choice 属性，当[`字段`](fields.html#django.forms.Field "django.forms.Field")的该属性更新时，它将覆盖你在这里的任何设置。

#### NullBooleanSelect

_class _`NullBooleanSelect`

Select Widget，选项为‘Unknown’、‘Yes’ 和‘No’。

#### SelectMultiple

_class _`SelectMultiple`

类似[`Select`](#django.forms.Select "django.forms.Select")，但是允许多个选择：`<select multiple='multiple'>...</select>`

#### RadioSelect

_class _`RadioSelect`

类似[`Select`](#django.forms.Select "django.forms.Select")，但是渲染成`<li>` 标签中的一个单选按钮列表：

```
<ul>
  <li><input type='radio' name='...'></li>
  ...
</ul>

```

你可以迭代模板中的单选按钮来更细致地控制生成的HTML。假设表单`myform` 具有一个字段`beatles`，它使用`RadioSelect` 作为Widget：

```
{% for radio in myform.beatles %}
<div class="myradio">
    {{ radio }}
</div>
{% endfor %}

```

它将生成以下HTML：

```
<div class="myradio">
    <label for="id_beatles_0"><input id="id_beatles_0" name="beatles" type="radio" value="john" /> John</label>
</div>
<div class="myradio">
    <label for="id_beatles_1"><input id="id_beatles_1" name="beatles" type="radio" value="paul" /> Paul</label>
</div>
<div class="myradio">
    <label for="id_beatles_2"><input id="id_beatles_2" name="beatles" type="radio" value="george" /> George</label>
</div>
<div class="myradio">
    <label for="id_beatles_3"><input id="id_beatles_3" name="beatles" type="radio" value="ringo" /> Ringo</label>
</div>

```

这包括`<label>` 标签。你可以使用单选按钮的`tag`、`choice_label` 和 `id_for_label` 属性进行更细的控制。例如，这个模板：

```
{% for radio in myform.beatles %}
    <label for="{{ radio.id_for_label }}">
        {{ radio.choice_label }}
        <span class="radio">{{ radio.tag }}</span>
    </label>
{% endfor %}

```

... 将生成下面的HTML：

```
<label for="id_beatles_0">
    John
    <span class="radio"><input id="id_beatles_0" name="beatles" type="radio" value="john" /></span>
</label>

<label for="id_beatles_1">
    Paul
    <span class="radio"><input id="id_beatles_1" name="beatles" type="radio" value="paul" /></span>
</label>

<label for="id_beatles_2">
    George
    <span class="radio"><input id="id_beatles_2" name="beatles" type="radio" value="george" /></span>
</label>

<label for="id_beatles_3">
    Ringo
    <span class="radio"><input id="id_beatles_3" name="beatles" type="radio" value="ringo" /></span>
</label>

```

如果你不迭代单选按钮 —— 例如，你的模板只是简单地包含`{{ myform.beatles }}` —— 它们将以`<ul>` 中的`<li>` 标签输出，就像上面一样。

外层的`<ul>` 将带有定义在Widget 上的`id` 属性。

Changed in Django 1.7:

当迭代单选按钮时，`label` 和`input` 标签分别包含`for` 和`id` 属性。每个单项按钮具有一个`id_for_label` 属性来输出元素的ID。

#### CheckboxSelectMultiple

_class _`CheckboxSelectMultiple`

类似[`SelectMultiple`](#django.forms.SelectMultiple "django.forms.SelectMultiple")，但是渲染成一个复选框列表：

```
<ul>
  <li><input type='checkbox' name='...' ></li>
  ...
</ul>

```

外层的`<ul>` 具有定义在Widget 上的`id` 属性。

类似[`RadioSelect`](#django.forms.RadioSelect "django.forms.RadioSelect")，你可以迭代列表的每个复选框。更多细节参见[`RadioSelect`](#django.forms.RadioSelect "django.forms.RadioSelect") 的文档。

Changed in Django 1.7:

当迭代单选按钮时，`label` 和`input` 标签分别包含`for` 和`id` 属性。 每个单项按钮具有一个`id_for_label` 属性来输出元素的ID。

### 文件上传Widget

#### FileInput

_class _`FileInput`

文件上传输入：`<input type='file' ...>`

#### ClearableFileInput

_class _`ClearableFileInput`

文件上传输入：`<input type='file' ...>`，带有一个额外的复选框，如果该字段不是必选的且有初始的数据，可以清除字段的值。

### 复合Widget

#### MultipleHiddenInput

_class _`MultipleHiddenInput`

多个`<input type='hidden' ...>` Widget。

一个处理多个隐藏的Widget 的Widget，用于值为一个列表的字段。

`choices`

当表单字段没有`choices` 属性时，这个属性是可选的。如果字段有choice 属性，当[`字段`](fields.html#django.forms.Field "django.forms.Field")的该属性更新时，它将覆盖你在这里的任何设置。

#### SplitDateTimeWidget

_class _`SplitDateTimeWidget`

封装（使用[`MultiWidget`](#django.forms.MultiWidget "django.forms.MultiWidget")）两个Widget：[`DateInput`](#django.forms.DateInput "django.forms.DateInput") 用于日期，[`TimeInput`](#django.forms.TimeInput "django.forms.TimeInput") 用于时间。

`SplitDateTimeWidget` 有两个可选的属性：

`date_format`

类似[`DateInput.format`](#django.forms.DateInput.format "django.forms.DateInput.format")

`time_format`

类似[`TimeInput.format`](#django.forms.TimeInput.format "django.forms.TimeInput.format")

#### SplitHiddenDateTimeWidget

_class _`SplitHiddenDateTimeWidget`

类似[`SplitDateTimeWidget`](#django.forms.SplitDateTimeWidget "django.forms.SplitDateTimeWidget")，但是日期和时间都使用[`HiddenInput`](#django.forms.HiddenInput "django.forms.HiddenInput")。

#### SelectDateWidget

_class _`SelectDateWidget`[[source]](../../_modules/django/forms/extras/widgets.html#SelectDateWidget)

封装三个[`Select`](#django.forms.Select "django.forms.Select") Widget：分别用于年、月、日。注意，这个Widget 与标准的Widget 位于不同的文件中。

接收一个可选的参数：

`years`

一个可选的列表/元组，用于”年“选择框。默认为包含当前年份和未来9年的一个列表。

`months`

New in Django 1.7\.

一个可选的字典，用于”月“选择框。

字典的键对应于月份的数字（从1开始），值为显示出来的月份：

```
MONTHS = {
    1:_('jan'), 2:_('feb'), 3:_('mar'), 4:_('apr'),
    5:_('may'), 6:_('jun'), 7:_('jul'), 8:_('aug'),
    9:_('sep'), 10:_('oct'), 11:_('nov'), 12:_('dec')
}

```

`empty_label`

New in Django 1.8\.

如果[`DateField`](fields.html#django.forms.DateField "django.forms.DateField") 不是必选的，[`SelectDateWidget`](#django.forms.extras.widgets.SelectDateWidget "django.forms.extras.widgets.SelectDateWidget") 将有一个空的选项位于选项的顶部（默认为`---`）。你可以通过`empty_label` 属性修改这个文本。`empty_label` 可以是一个`字符串`、`列表` 或`元组`。当使用字符串时，所有的选择框都带有这个空选项。如果`empty_label` 为具有3个字符串元素的`列表` 或`元组`，每个选择框将具有它们自定义的空选项。空选项应该按这个顺序`('year_label', 'month_label', 'day_label')`。

```
# A custom empty label with string
field1 = forms.DateField(widget=SelectDateWidget(empty_label="Nothing"))

# A custom empty label with tuple
field1 = forms.DateField(widget=SelectDateWidget(
    empty_label=("Choose Year", "Choose Month", "Choose Day"))

```

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Built-in widgets](https://docs.djangoproject.com/en/1.8/ref/forms/widgets/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
