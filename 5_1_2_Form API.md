# 表单 API

关于这篇文档

这篇文档讲述Django 表单API 的详细细节。你应该先阅读[_表单简介_](../../topics/forms/index.html)。

## 绑定的表单和未绑定的表单

[`表单`](#django.forms.Form "django.forms.Form")要么是**绑定的**，要么是**未绑定的**。

*   如果是**绑定的**，那么它能够验证数据，并渲染表单及其数据成HTML。
*   如果是**未绑定的**，那么它不能够完成验证（因为没有可验证的数据！），但是仍然能渲染空白的表单成HTML。

_class _`Form`

若要创建一个未绑定的[`表单`](#django.forms.Form "django.forms.Form")实例，只需简单地实例化该类：

```
>>> f = ContactForm()

```

若要绑定数据到表单，可以将数据以字典的形式传递给[`表单`](#django.forms.Form "django.forms.Form")类的构造函数的第一个参数：

```
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data)

```

在这个字典中，键为字段的名称，它们对应于[`表单`](#django.forms.Form "django.forms.Form")类中的属性。值为需要验证的数据。它们通常为字符串，但是没有强制要求必须是字符串；传递的数据类型取决于[`字段`](fields.html#django.forms.Field "django.forms.Field")，我们稍后会看到。

`Form.``is_bound`

如果运行时刻你需要区分绑定的表单和未绑定的表单，可以检查下表单[`is_bound`](#django.forms.Form.is_bound "django.forms.Form.is_bound") 属性的值：

```
>>> f = ContactForm()
>>> f.is_bound
False
>>> f = ContactForm({'subject': 'hello'})
>>> f.is_bound
True

```

注意，传递一个空的字典将创建一个带有空数据的_绑定的_表单：

```
>>> f = ContactForm({})
>>> f.is_bound
True

```

如果你有一个绑定的[`表单`](#django.forms.Form "django.forms.Form")实例但是想改下数据，或者你想绑定一个未绑定的[`表单`](#django.forms.Form "django.forms.Form")表单到某些数据，你需要创建另外一个[`表单`](#django.forms.Form "django.forms.Form")实例。[`Form`](#django.forms.Form "django.forms.Form") 实例的数据没有办法修改。[`表单`](#django.forms.Form "django.forms.Form")实例一旦创建，你应该将它的数据视为不可变的，无论它有没有数据。

## 使用表单来验证数据

`Form.``clean`()

当你需要为相互依赖的字段添加自定义的验证时，你可以实现`表单`的`clean()`方法。示例用法参见[_Cleaning and validating fields that depend on each other_](validation.html#validating-fields-with-clean)。

`Form.``is_valid`()

[`表单`](#django.forms.Form "django.forms.Form")对象的首要任务就是验证数据。对于绑定的[`表单`](#django.forms.Form "django.forms.Form")实例，可以调用[`is_valid()`](#django.forms.Form.is_valid "django.forms.Form.is_valid")方法来执行验证并返回一个表示数据是否合法的布尔值。

```
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> f.is_valid()
True

```

让我们试下非法的数据。下面的情形中，`subject` 为空（默认所有字段都是必需的）且`sender` 是一个不合法的邮件地址：

```
>>> data = {'subject': '',
...         'message': 'Hi there',
...         'sender': 'invalid email address',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> f.is_valid()
False

```

`Form.``errors`

访问[`errors`](#django.forms.Form.errors "django.forms.Form.errors") 属性可以获得错误信息的一个字典：

```
>>> f.errors
{'sender': ['Enter a valid email address.'], 'subject': ['This field is required.']}

```

在这个字典中，键为字段的名称，值为表示错误信息的Unicode 字符串组成的列表。错误信息保存在列表中是因为字段可能有多个错误信息。

你可以在调用[`is_valid()`](#django.forms.Form.is_valid "django.forms.Form.is_valid")&nbsp;之前访问[`errors`](#django.forms.Form.errors "django.forms.Form.errors")。表单的数据将在第一次调用[`is_valid()`](#django.forms.Form.is_valid "django.forms.Form.is_valid") 或者访问[`errors`](#django.forms.Form.errors "django.forms.Form.errors") 时验证。

验证将值调用一次，无论你访问[`errors`](#django.forms.Form.errors "django.forms.Form.errors") 或者调用[`is_valid()`](#django.forms.Form.is_valid "django.forms.Form.is_valid") 多少次。这意味着，如果验证过程有副作用，这些副作用将只触发一次。

`Form.errors.``as_data`()

New in Django 1.7\.

返回一个`字典`，它映射字段到原始的`ValidationError` 实例。

```
>>> f.errors.as_data()
{'sender': [ValidationError(['Enter a valid email address.'])],
'subject': [ValidationError(['This field is required.'])]}

```

每当你需要根据错误的`code` 来识别错误时，可以调用这个方法。它可以用来重写错误信息或者根据特定的错误编写自定义的逻辑。它还可以用来序列化错误为一个自定义的格式（例如，XML）；[`as_json()`](#django.forms.Form.errors.as_json "django.forms.Form.errors.as_json") 就依赖于`as_data()`。

需要`as_data()` 方法是为了向后兼容。以前，`ValidationError` 实例在它们**渲染后**&nbsp;的错误消息一旦添加到`Form.errors` 字典就立即被丢弃。理想情况下，`Form.errors` 应该已经保存`ValidationError` 实例而带有`as_` 前缀的方法可以渲染它们，但是为了不破坏直接使用`Form.errors` 中的错误消息的代码，必须使用其它方法来实现。

`Form.errors.``as_json`(_escape_html=False_)

New in Django 1.7\.

返回JSON 序列化后的错误。

```
>>> f.errors.as_json()
{"sender": [{"message": "Enter a valid email address.", "code": "invalid"}],
"subject": [{"message": "This field is required.", "code": "required"}]}

```

默认情况下，`as_json()` 不会转义它的输出。如果你正在使用AJAX 请求表单视图，而客户端会解析响应并将错误插入到页面中，你必须在客户端对结果进行转义以避免可能的跨站脚本攻击。使用一个JavaScript 库比如jQuery 来做这件事很简单 —— 只要使用`$(el).text(errorText)` 而不是`.html()` 就可以。

如果由于某种原因你不想使用客户端的转义，你还可以设置`escape_html=True`，这样错误消息将被转义而你可以直接在HTML 中使用它们。

`Form.``add_error`(_field_, _error_)

New in Django 1.7\.

这个方法允许在`Form.clean()` 方法内部或从表单的外部一起给字段添加错误信息；例如从一个视图中。

`field` 参数为字段的名称。如果值为`None`，error 将作为[`Form.non_field_errors()`](#django.forms.Form.non_field_errors "django.forms.Form.non_field_errors") 返回的一个非字段错误。

`error` 参数可以是一个简单的字符串，或者最好是一个`ValidationError` 实例。[_引发ValidationError_](validation.html#raising-validation-error) 中可以看到定义表单错误时的最佳实践。

注意，`Form.add_error()` 会自动删除`cleaned_data` 中的相关字段。

`Form.``has_error`(_field_, _code=None_)

New in Django 1.8\.

这个方法返回一个布尔值，指示一个字段是否具有指定错误`code` 的错误。当`code` 为`None` 时，如果字段有任何错误它都将返回`True`。

若要检查非字段错误，使用[`NON_FIELD_ERRORS`](../exceptions.html#django.core.exceptions.NON_FIELD_ERRORS "django.core.exceptions.NON_FIELD_ERRORS") 作为`field` 参数。

`Form.``non_field_errors`()

这个方法返回[`Form.errors`](#django.forms.Form.errors "django.forms.Form.errors") 中不是与特定字段相关联的错误。它包含在[`Form.clean()`](#django.forms.Form.clean "django.forms.Form.clean") 中引发的`ValidationError` 和使用[`Form.add_error(None, "...")`](#django.forms.Form.add_error "django.forms.Form.add_error") 添加的错误。

### 未绑定表单的行为

验证没有绑定数据的表单是没有意义的，下面的例子展示了这种情况：

```
>>> f = ContactForm()
>>> f.is_valid()
False
>>> f.errors
{}

```

## 动态的初始值

`Form.``initial`

表单字段的初始值使用[`initial`](#django.forms.Form.initial "django.forms.Form.initial")声明。例如，你可能希望使用当前会话的用户名填充`username`字段。

使用[`Form`](#django.forms.Form "django.forms.Form")的[`initial`](#django.forms.Form.initial "django.forms.Form.initial")参数可以实现。该参数是字段名到初始值的一个字典。只需要包含你期望给出初始值的字段；不需要包含表单中的所有字段。例如：

```
>>> f = ContactForm(initial={'subject': 'Hi there!'})

```

这些值只显示在没有绑定的表单中，即使没有提供特定值它们也不会作为后备的值。

注意，如果[`字段`](fields.html#django.forms.Field "django.forms.Field")有定义[`initial`](#django.forms.Form.initial "django.forms.Form.initial")， _而_实例化`表单`时也提供`initial`，那么后面的`initial` 将优先。在下面的例子中，`initial` 在字段和表单实例化中都有定义，此时后者具有优先权：

```
>>> from django import forms
>>> class CommentForm(forms.Form):
...     name = forms.CharField(initial='class')
...     url = forms.URLField()
...     comment = forms.CharField()
>>> f = CommentForm(initial={'name': 'instance'}, auto_id=False)
>>> print(f)
<tr><th>Name:</th><td><input type="text" name="name" value="instance" /></td></tr>
<tr><th>Url:</th><td><input type="url" name="url" /></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" /></td></tr>

```

## 检查表单数据是否改变

`Form.``has_changed`()

当你需要检查表单的数据是否从初始数据发生改变时，可以使用`表单`的`has_changed()` 方法。

```
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data, initial=data)
>>> f.has_changed()
False

```

当提交表单时，我们可以重新构建表单并提供初始值，这样可以实现比较：

```
>>> f = ContactForm(request.POST, initial=data)
>>> f.has_changed()

```

如果`request.POST` 中的数据与[`initial`](#django.forms.Form.initial "django.forms.Form.initial") 中的不同，`has_changed()` 将为`True`，否则为`False`。 计算的结果是通过调用表单每个字段的[`Field.has_changed()`](fields.html#django.forms.Field.has_changed "django.forms.Field.has_changed") 得到的。

## 从表单中访问字段

`Form.``fields`

你可以从[`表单`](#django.forms.Form "django.forms.Form")实例的`fields`属性访问字段：

```
>>> for row in f.fields.values(): print(row)
...
<django.forms.fields.CharField object at 0x7ffaac632510>
<django.forms.fields.URLField object at 0x7ffaac632f90>
<django.forms.fields.CharField object at 0x7ffaac3aa050>
>>> f.fields['name']
<django.forms.fields.CharField object at 0x7ffaac6324d0>

```

可你可以修改[`表单`](#django.forms.Form "django.forms.Form")实例的字段来改变字段在表单中的表示：

```
>>> f.as_table().split('\n')[0]
'<tr><th>Name:</th><td><input name="name" type="text" value="instance" /></td></tr>'
>>> f.fields['name'].label = "Username"
>>> f.as_table().split('\n')[0]
'<tr><th>Username:</th><td><input name="name" type="text" value="instance" /></td></tr>'

```

注意不要改变`base_fields` 属性，因为一旦修改将影响同一个Python 进程中接下来所有的`ContactForm` 实例：

```
>>> f.base_fields['name'].label = "Username"
>>> another_f = CommentForm(auto_id=False)
>>> another_f.as_table().split('\n')[0]
'<tr><th>Username:</th><td><input name="name" type="text" value="class" /></td></tr>'

```

## 访问“清洁”的数据

`Form.``cleaned_data`

[`表单`](#django.forms.Form "django.forms.Form")类中的每个字段不仅负责验证数据，还负责“清洁”它们 —— 将它们转换为正确的格式。这是个非常好用的功能，因为它允许字段以多种方式输入数据，并总能得到一致的输出。

例如，[`DateField`](fields.html#django.forms.DateField "django.forms.DateField") 将输入转换为Python 的 `datetime.date` 对象。无论你传递的是`'1994-07-15'` 格式的字符串、`datetime.date` 对象、还是其它格式的数字，`DateField` 将始终将它们转换成`datetime.date` 对象，只要它们是合法的。

一旦你创建一个[`表单`](#django.forms.Form "django.forms.Form")实例并通过验证后，你就可以通过它的`cleaned_data` 属性访问清洁的数据：

```
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> f.is_valid()
True
>>> f.cleaned_data
{'cc_myself': True, 'message': 'Hi there', 'sender': 'foo@example.com', 'subject': 'hello'}

```

注意，文本字段 —— 例如，`CharField` 和`EmailField` —— 始终将输入转换为Unicode 字符串。我们将在这篇文档的后面将是编码的影响。

如果你的数据_没有_ 通过验证，`cleaned_data` 字典中只包含合法的字段：

```
>>> data = {'subject': '',
...         'message': 'Hi there',
...         'sender': 'invalid email address',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> f.is_valid()
False
>>> f.cleaned_data
{'cc_myself': True, 'message': 'Hi there'}

```

`cleaned_data` 始终_只_ 包含`表单`中定义的字段，即使你在构建`表单` 时传递了额外的数据。在下面的例子中，我们传递一组额外的字段给`ContactForm` 构造函数，但是`cleaned_data` 将只包含表单的字段：

```
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True,
...         'extra_field_1': 'foo',
...         'extra_field_2': 'bar',
...         'extra_field_3': 'baz'}
>>> f = ContactForm(data)
>>> f.is_valid()
True
>>> f.cleaned_data # Doesn't contain extra_field_1, etc.
{'cc_myself': True, 'message': 'Hi there', 'sender': 'foo@example.com', 'subject': 'hello'}

```

当`表单`合法时，`cleaned_data` 将包含_所有_字段的键和值，即使传递的数据不包含某些可选字段的值。在下面的例子中，传递的数据字典不包含`nick_name` 字段的值，但是`cleaned_data` 任然包含它，只是值为空：

```
>>> from django.forms import Form
>>> class OptionalPersonForm(Form):
...     first_name = CharField()
...     last_name = CharField()
...     nick_name = CharField(required=False)
>>> data = {'first_name': 'John', 'last_name': 'Lennon'}
>>> f = OptionalPersonForm(data)
>>> f.is_valid()
True
>>> f.cleaned_data
{'nick_name': '', 'first_name': 'John', 'last_name': 'Lennon'}

```

在上面的例子中，`cleaned_data` 中`nick_name` 设置为一个空字符串，这是因为`nick_name` 是`CharField`而 `CharField` 将空值作为一个空字符串。每个字段都知道自己的“空”值 —— 例如，`DateField` 的空值是`None` 而不是一个空字符串。关于每个字段空值的完整细节，参见“内建的`Field` 类”一节中每个字段的“空值”提示。

你可以自己编写代码来对特定的字段（根据它们的名字）或者表单整体（考虑到不同字段的组合）进行验证。更多信息参见[_表单和字段验证_](validation.html)。

## 输出表单为HTML

`表单`对象的第二个任务是将它渲染成HTML。很简单，`print` 它：

```
>>> f = ContactForm()
>>> print(f)
<tr><th><label for="id_subject">Subject:</label></th><td><input id="id_subject" type="text" name="subject" maxlength="100" /></td></tr>
<tr><th><label for="id_message">Message:</label></th><td><input type="text" name="message" id="id_message" /></td></tr>
<tr><th><label for="id_sender">Sender:</label></th><td><input type="email" name="sender" id="id_sender" /></td></tr>
<tr><th><label for="id_cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="id_cc_myself" /></td></tr>

```

如果表单是绑定的，输出的HTML 将包含数据。例如，如果字段是`<input type="text">` 的形式，其数据将位于`value` 属性中。如果字段是`<input type="checkbox">` 的形式，HTML 将包含`checked="checked"`：

```
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> print(f)
<tr><th><label for="id_subject">Subject:</label></th><td><input id="id_subject" type="text" name="subject" maxlength="100" value="hello" /></td></tr>
<tr><th><label for="id_message">Message:</label></th><td><input type="text" name="message" id="id_message" value="Hi there" /></td></tr>
<tr><th><label for="id_sender">Sender:</label></th><td><input type="email" name="sender" id="id_sender" value="foo@example.com" /></td></tr>
<tr><th><label for="id_cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="id_cc_myself" checked="checked" /></td></tr>

```

默认的输出时具有两个列的HTML 表格，每个字段对应一个`<tr>`。注意事项：

*   为了灵活性，输出_不_包含`<table>` 和`</table>`、`<form>` 和`</form>`&nbsp;以及`<input type="submit">` 标签。你需要添加它们。
*   每个字段类型有一个默认的HTML 表示。`CharField` 表示为一个`<input type="text">`，`EmailField` 表示为一个`<input type="email">`。`BooleanField` 表示为一个`<input type="checkbox">`。注意，这些只是默认的表示；你可以使用Widget 指定字段使用哪种HTML，我们将稍后解释。
*   每个标签的HTML `name` 直接从`ContactForm` 类中获取。
*   每个字段的文本标签 —— 例如`'Subject:'`、`'Message:'` 和`'Cc myself:'` 通过将所有的下划线转换成空格并大写第一个字母生成。再次提醒，这些只是默认的表示；你可以手工指定标签。
*   每个文本标签周围有一个HTML `<label>` 标签，它指向表单字段的`id`。这个`id`，是通过在字段名称前面加上`'id_'` 前缀生成。`id` 属性和`<label>` 标签默认包含在输出中，但你可以改变这一行为。

虽然`print` 表单时`<table>` 是默认的输出格式，但是还有其它格式可用。每个格式对应于表单对象的一个方法，每个方法都返回一个Unicode 对象。

### as_p()

`Form.``as_p`()

`as_p()` 渲染表单为一系列的`<p>` 标签，每个`<p>` 标签包含一个字段：

```
>>> f = ContactForm()
>>> f.as_p()
'<p><label for="id_subject">Subject:</label> <input id="id_subject" type="text" name="subject" maxlength="100" /></p>\n<p><label for="id_message">Message:</label> <input type="text" name="message" id="id_message" /></p>\n<p><label for="id_sender">Sender:</label> <input type="text" name="sender" id="id_sender" /></p>\n<p><label for="id_cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="id_cc_myself" /></p>'
>>> print(f.as_p())
<p><label for="id_subject">Subject:</label> <input id="id_subject" type="text" name="subject" maxlength="100" /></p>
<p><label for="id_message">Message:</label> <input type="text" name="message" id="id_message" /></p>
<p><label for="id_sender">Sender:</label> <input type="email" name="sender" id="id_sender" /></p>
<p><label for="id_cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="id_cc_myself" /></p>

```

### as_ul()

`Form.``as_ul`()

`as_ul()` 渲染表单为一系列的`<li>`标签，每个`<li>` 标签包含一个字段。它_不_包含`<ul>` 和`</ul>`，所以你可以自己指定`<ul>` 的任何HTML 属性：

```
>>> f = ContactForm()
>>> f.as_ul()
'<li><label for="id_subject">Subject:</label> <input id="id_subject" type="text" name="subject" maxlength="100" /></li>\n<li><label for="id_message">Message:</label> <input type="text" name="message" id="id_message" /></li>\n<li><label for="id_sender">Sender:</label> <input type="email" name="sender" id="id_sender" /></li>\n<li><label for="id_cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="id_cc_myself" /></li>'
>>> print(f.as_ul())
<li><label for="id_subject">Subject:</label> <input id="id_subject" type="text" name="subject" maxlength="100" /></li>
<li><label for="id_message">Message:</label> <input type="text" name="message" id="id_message" /></li>
<li><label for="id_sender">Sender:</label> <input type="email" name="sender" id="id_sender" /></li>
<li><label for="id_cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="id_cc_myself" /></li>

```

### as_table()

`Form.``as_table`()

最后，`as_table()`输出表单为一个HTML `<table>`。它与`print` 完全相同。事实上，当你`print` 一个表单对象时，在后台调用的就是`as_table()` 方法：

```
>>> f = ContactForm()
>>> f.as_table()
'<tr><th><label for="id_subject">Subject:</label></th><td><input id="id_subject" type="text" name="subject" maxlength="100" /></td></tr>\n<tr><th><label for="id_message">Message:</label></th><td><input type="text" name="message" id="id_message" /></td></tr>\n<tr><th><label for="id_sender">Sender:</label></th><td><input type="email" name="sender" id="id_sender" /></td></tr>\n<tr><th><label for="id_cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="id_cc_myself" /></td></tr>'
>>> print(f.as_table())
<tr><th><label for="id_subject">Subject:</label></th><td><input id="id_subject" type="text" name="subject" maxlength="100" /></td></tr>
<tr><th><label for="id_message">Message:</label></th><td><input type="text" name="message" id="id_message" /></td></tr>
<tr><th><label for="id_sender">Sender:</label></th><td><input type="email" name="sender" id="id_sender" /></td></tr>
<tr><th><label for="id_cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="id_cc_myself" /></td></tr>

```

### 表单必填行和错误行的样式

`Form.``error_css_class`

`Form.``required_css_class`

将必填的表单行和有错误的表单行定义不同的样式特别常见。例如，你想将必填的表单行以粗体显示、将错误以红色显示。

[`表单`](#django.forms.Form "django.forms.Form")类具有一对钩子，可以使用它们来添加`class` 属性给必填的行或有错误的行：只需简单地设置[`Form.error_css_class`](#django.forms.Form.error_css_class "django.forms.Form.error_css_class") 和/或 [`Form.required_css_class`](#django.forms.Form.required_css_class "django.forms.Form.required_css_class") 属性：

```
from django.forms import Form

class ContactForm(Form):
    error_css_class = 'error'
    required_css_class = 'required'

    # ... and the rest of your fields here

```

一旦你设置好，将根据需要设置行的`"error"` 和/或`"required"` CSS 类型。 其HTML 看上去将类似：

```
>>> f = ContactForm(data)
>>> print(f.as_table())
<tr class="required"><th><label class="required" for="id_subject">Subject:</label>    ...
<tr class="required"><th><label class="required" for="id_message">Message:</label>    ...
<tr class="required error"><th><label class="required" for="id_sender">Sender:</label>      ...
<tr><th><label for="id_cc_myself">Cc myself:<label> ...
>>> f['subject'].label_tag()
<label class="required" for="id_subject">Subject:</label>
>>> f['subject'].label_tag(attrs={'class': 'foo'})
<label for="id_subject" class="foo required">Subject:</label>

```

Changed in Django 1.8:

`required_css_class` 添加到`<label>` 标签，如上面所看到的。

### 配置表单元素的HTML `id` 属性和 `<label>` 标签

`Form.``auto_id`

默认情况下，表单的渲染方法包含：

*   表单元素的HTML `id` 属性
*   对应的`<label>` 标签。HTML `<label>` 标签指示标签文本关联的表单元素。这个小小的改进让表单在辅助设备上具有更高的可用性。使用`<label>` 标签始终是个好想法。

`id` 属性值通过在表单字段名称的前面加上`id_` 生成。但是如果你想改变`id` 的生成方式或者完全删除 HTML `id` 属性和`<label>`标签，这个行为是可配置的。

`id` 和label 的行为使用`表单`构造函数的`auto_id` 参数控制。这个参数必须为`True`、`False` 或者一个字符串。

如果`auto_id` 为`False`，那么表单的输出将不包含`<label>` 标签和`id` 属性：

```
>>> f = ContactForm(auto_id=False)
>>> print(f.as_table())
<tr><th>Subject:</th><td><input type="text" name="subject" maxlength="100" /></td></tr>
<tr><th>Message:</th><td><input type="text" name="message" /></td></tr>
<tr><th>Sender:</th><td><input type="email" name="sender" /></td></tr>
<tr><th>Cc myself:</th><td><input type="checkbox" name="cc_myself" /></td></tr>
>>> print(f.as_ul())
<li>Subject: <input type="text" name="subject" maxlength="100" /></li>
<li>Message: <input type="text" name="message" /></li>
<li>Sender: <input type="email" name="sender" /></li>
<li>Cc myself: <input type="checkbox" name="cc_myself" /></li>
>>> print(f.as_p())
<p>Subject: <input type="text" name="subject" maxlength="100" /></p>
<p>Message: <input type="text" name="message" /></p>
<p>Sender: <input type="email" name="sender" /></p>
<p>Cc myself: <input type="checkbox" name="cc_myself" /></p>

```

如果`auto_id` 设置为`True`，那么输出的表示_将_ 包含`<label>` 标签并简单地使用字典名称作为每个表单字段的`id`：

```
>>> f = ContactForm(auto_id=True)
>>> print(f.as_table())
<tr><th><label for="subject">Subject:</label></th><td><input id="subject" type="text" name="subject" maxlength="100" /></td></tr>
<tr><th><label for="message">Message:</label></th><td><input type="text" name="message" id="message" /></td></tr>
<tr><th><label for="sender">Sender:</label></th><td><input type="email" name="sender" id="sender" /></td></tr>
<tr><th><label for="cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="cc_myself" /></td></tr>
>>> print(f.as_ul())
<li><label for="subject">Subject:</label> <input id="subject" type="text" name="subject" maxlength="100" /></li>
<li><label for="message">Message:</label> <input type="text" name="message" id="message" /></li>
<li><label for="sender">Sender:</label> <input type="email" name="sender" id="sender" /></li>
<li><label for="cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="cc_myself" /></li>
>>> print(f.as_p())
<p><label for="subject">Subject:</label> <input id="subject" type="text" name="subject" maxlength="100" /></p>
<p><label for="message">Message:</label> <input type="text" name="message" id="message" /></p>
<p><label for="sender">Sender:</label> <input type="email" name="sender" id="sender" /></p>
<p><label for="cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="cc_myself" /></p>

```

如果`auto_id` 设置为包含格式字符`'%s'` 的字符串，那么表单的输出将包含`<label>` 标签，并将根据格式字符串生成`id` 属性。例如，对于格式字符串`'field_%s'`，名为`subject` 的字段的`id` 值将是`'field_subject'`。继续我们的例子：

```
>>> f = ContactForm(auto_id='id_for_%s')
>>> print(f.as_table())
<tr><th><label for="id_for_subject">Subject:</label></th><td><input id="id_for_subject" type="text" name="subject" maxlength="100" /></td></tr>
<tr><th><label for="id_for_message">Message:</label></th><td><input type="text" name="message" id="id_for_message" /></td></tr>
<tr><th><label for="id_for_sender">Sender:</label></th><td><input type="email" name="sender" id="id_for_sender" /></td></tr>
<tr><th><label for="id_for_cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="id_for_cc_myself" /></td></tr>
>>> print(f.as_ul())
<li><label for="id_for_subject">Subject:</label> <input id="id_for_subject" type="text" name="subject" maxlength="100" /></li>
<li><label for="id_for_message">Message:</label> <input type="text" name="message" id="id_for_message" /></li>
<li><label for="id_for_sender">Sender:</label> <input type="email" name="sender" id="id_for_sender" /></li>
<li><label for="id_for_cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="id_for_cc_myself" /></li>
>>> print(f.as_p())
<p><label for="id_for_subject">Subject:</label> <input id="id_for_subject" type="text" name="subject" maxlength="100" /></p>
<p><label for="id_for_message">Message:</label> <input type="text" name="message" id="id_for_message" /></p>
<p><label for="id_for_sender">Sender:</label> <input type="email" name="sender" id="id_for_sender" /></p>
<p><label for="id_for_cc_myself">Cc myself:</label> <input type="checkbox" name="cc_myself" id="id_for_cc_myself" /></p>

```

如果`auto_id` 设置为任何其它的真值 —— 例如不包含`%s` 的字符串 —— 那么其行为将类似`auto_id` 等于`True`。

默认情况下，`auto_id` 设置为`'id_%s'`。

`Form.``label_suffix`

一个字符串（默认为英文的`:`），表单渲染时将附加在每个label 名称的后面。

使用`label_suffix` 参数可以自定义这个字符，或者完全删除它：

```
>>> f = ContactForm(auto_id='id_for_%s', label_suffix='')
>>> print(f.as_ul())
<li><label for="id_for_subject">Subject</label> <input id="id_for_subject" type="text" name="subject" maxlength="100" /></li>
<li><label for="id_for_message">Message</label> <input type="text" name="message" id="id_for_message" /></li>
<li><label for="id_for_sender">Sender</label> <input type="email" name="sender" id="id_for_sender" /></li>
<li><label for="id_for_cc_myself">Cc myself</label> <input type="checkbox" name="cc_myself" id="id_for_cc_myself" /></li>
>>> f = ContactForm(auto_id='id_for_%s', label_suffix=' ->')
>>> print(f.as_ul())
<li><label for="id_for_subject">Subject -></label> <input id="id_for_subject" type="text" name="subject" maxlength="100" /></li>
<li><label for="id_for_message">Message -></label> <input type="text" name="message" id="id_for_message" /></li>
<li><label for="id_for_sender">Sender -></label> <input type="email" name="sender" id="id_for_sender" /></li>
<li><label for="id_for_cc_myself">Cc myself -></label> <input type="checkbox" name="cc_myself" id="id_for_cc_myself" /></li>

```

注意，该标签后缀只有当label 的最后一个字符不是表单符号（`.`, `!`, `?` 和`:`）时才添加。

New in Django 1.8\.

字段可以定义自己的[`label_suffix`](fields.html#django.forms.Field.label_suffix "django.forms.Field.label_suffix")。而且将优先于[`Form.label_suffix`](#django.forms.Form.label_suffix "django.forms.Form.label_suffix")。在运行时刻，后缀可以使用[`label_tag()`](#django.forms.BoundField.label_tag "django.forms.BoundField.label_tag") 的`label_suffix` 参数覆盖。

### 字段的顺序

在`as_p()`、`as_ul()` 和`as_table()` 中，字段以表单类中定义的顺序显示。例如，在`ContactForm` 示例中，字段定义的顺序为`subject`, `message`, `sender`, `cc_myself`。若要重新排序HTML 中的输出，只需改变字段在类中列出的顺序。

### 错误如何显示

如果你渲染一个绑定的`表单`对象，渲染时将自动运行表单的验证，HTML 输出将在出错字段的附近以`<ul class="errorlist">` 形式包含验证的错误。错误信息的位置与你使用的输出方法有关：

```
>>> data = {'subject': '',
...         'message': 'Hi there',
...         'sender': 'invalid email address',
...         'cc_myself': True}
>>> f = ContactForm(data, auto_id=False)
>>> print(f.as_table())
<tr><th>Subject:</th><td><ul class="errorlist"><li>This field is required.</li></ul><input type="text" name="subject" maxlength="100" /></td></tr>
<tr><th>Message:</th><td><input type="text" name="message" value="Hi there" /></td></tr>
<tr><th>Sender:</th><td><ul class="errorlist"><li>Enter a valid email address.</li></ul><input type="email" name="sender" value="invalid email address" /></td></tr>
<tr><th>Cc myself:</th><td><input checked="checked" type="checkbox" name="cc_myself" /></td></tr>
>>> print(f.as_ul())
<li><ul class="errorlist"><li>This field is required.</li></ul>Subject: <input type="text" name="subject" maxlength="100" /></li>
<li>Message: <input type="text" name="message" value="Hi there" /></li>
<li><ul class="errorlist"><li>Enter a valid email address.</li></ul>Sender: <input type="email" name="sender" value="invalid email address" /></li>
<li>Cc myself: <input checked="checked" type="checkbox" name="cc_myself" /></li>
>>> print(f.as_p())
<p><ul class="errorlist"><li>This field is required.</li></ul></p>
<p>Subject: <input type="text" name="subject" maxlength="100" /></p>
<p>Message: <input type="text" name="message" value="Hi there" /></p>
<p><ul class="errorlist"><li>Enter a valid email address.</li></ul></p>
<p>Sender: <input type="email" name="sender" value="invalid email address" /></p>
<p>Cc myself: <input checked="checked" type="checkbox" name="cc_myself" /></p>

```

### 自定义错误清单的格式

默认情况下，表单使用`django.forms.utils.ErrorList` 来格式化验证时的错误。如果你希望使用另外一种类来显示错误，可以在构造时传递（在Python 2 中将 `__str__` 替换为`__unicode__`）：

```
>>> from django.forms.utils import ErrorList
>>> class DivErrorList(ErrorList):
...     def __str__(self):              # __unicode__ on Python 2
...         return self.as_divs()
...     def as_divs(self):
...         if not self: return ''
...         return '<div class="errorlist">%s</div>' % ''.join(['<div class="error">%s</div>' % e for e in self])
>>> f = ContactForm(data, auto_id=False, error_class=DivErrorList)
>>> f.as_p()
<div class="errorlist"><div class="error">This field is required.</div></div>
<p>Subject: <input type="text" name="subject" maxlength="100" /></p>
<p>Message: <input type="text" name="message" value="Hi there" /></p>
<div class="errorlist"><div class="error">Enter a valid email address.</div></div>
<p>Sender: <input type="email" name="sender" value="invalid email address" /></p>
<p>Cc myself: <input checked="checked" type="checkbox" name="cc_myself" /></p>

```

Changed in Django 1.7:

`django.forms.util` 重命名为`django.forms.utils`。

### 更细粒度的输出

`as_p()`、`as_ul()` 和`as_table()` 方法是为懒惰的程序员准备的简单快捷方法 —— 它们不是显示表单的唯一方式。

_class _`BoundField`

用于显示HTML 表单或者访问[`表单`](#django.forms.Form "django.forms.Form")实例的一个属性。

其`__str__()`（Python 2 上为`__unicode__`）方法显示该字段的HTML。

以字段的名称为键，用字典查询语法查询表单，可以获取一个 `BoundField`：

```
>>> form = ContactForm()
>>> print(form['subject'])
<input id="id_subject" type="text" name="subject" maxlength="100" />

```

迭代表单可以获取所有的`BoundField`：

```
>>> form = ContactForm()
>>> for boundfield in form: print(boundfield)
<input id="id_subject" type="text" name="subject" maxlength="100" />
<input type="text" name="message" id="id_message" />
<input type="email" name="sender" id="id_sender" />
<input type="checkbox" name="cc_myself" id="id_cc_myself" />

```

字段的输出与表单的`auto_id` 设置有关：

```
>>> f = ContactForm(auto_id=False)
>>> print(f['message'])
<input type="text" name="message" />
>>> f = ContactForm(auto_id='id_%s')
>>> print(f['message'])
<input type="text" name="message" id="id_message" />

```

若要获取字段的错误列表，可以访问字段的`errors` 属性。

`BoundField.``errors`

一个类列表对象，打印时以HTML `<ul class="errorlist">` 形式显示：

```
>>> data = {'subject': 'hi', 'message': '', 'sender': '', 'cc_myself': ''}
>>> f = ContactForm(data, auto_id=False)
>>> print(f['message'])
<input type="text" name="message" />
>>> f['message'].errors
['This field is required.']
>>> print(f['message'].errors)
<ul class="errorlist"><li>This field is required.</li></ul>
>>> f['subject'].errors
[]
>>> print(f['subject'].errors)

>>> str(f['subject'].errors)
''

```

`BoundField.``label_tag`(_contents=None_, _attrs=None_, _label_suffix=None_)

可以调用`label_tag` 方法单独渲染表单字段的label 标签：

```
>>> f = ContactForm(data)
>>> print(f['message'].label_tag())
<label for="id_message">Message:</label>

```

如果你提供一个可选的`contents` 参数，它将替换自动生成的label 标签。另外一个可选的`attrs` &nbsp;参数可以包含`<label>` 标签额外的属性。

生成的HTML 包含表单的[`label_suffix`](#django.forms.Form.label_suffix "django.forms.Form.label_suffix")（默认为一个冒号），或者当前字段的[`label_suffix`](fields.html#django.forms.Field.label_suffix "django.forms.Field.label_suffix")。可选的`label_suffix` 参数允许你覆盖之前设置的后缀。例如，你可以使用一个空字符串来隐藏已选择字段的label。如果在模板中需要这样做，你可以编写一个自定义的过滤器来允许传递参数给`label_tag`。

Changed in Django 1.8:

如果可用，label 将包含[`required_css_class`](#django.forms.Form.required_css_class "django.forms.Form.required_css_class")。

`BoundField.``css_classes`()

当你使用Django 的快捷的渲染方法时，习惯使用CSS &nbsp;类型来表示必填的表单字段和有错误的字段。如果你是手工渲染一个表单，你可以使用`css_classes` 方法访问这些CSS 类型：

```
>>> f = ContactForm(data)
>>> f['message'].css_classes()
'required'

```

除了错误和必填的类型之外，如果你还想提供额外的类型，你可以用参数传递它们：

```
>>> f = ContactForm(data)
>>> f['message'].css_classes('foo bar')
'foo bar required'

```

`BoundField.``value`()

这个方法用于渲染字段的原始值，与用`Widget` 渲染的值相同：

```
>>> initial = {'subject': 'welcome'}
>>> unbound_form = ContactForm(initial=initial)
>>> bound_form = ContactForm(data, initial=initial)
>>> print(unbound_form['subject'].value())
welcome
>>> print(bound_form['subject'].value())
hi

```

`BoundField.``id_for_label`

使用这个属性渲染字段的ID。例如，如果你在模板中手工构造一个`<label>`（尽管 [`label_tag()`](#django.forms.BoundField.label_tag "django.forms.BoundField.label_tag") 将为你这么做）：

```
<label for="{{ form.my_field.id_for_label }}">...</label>{{ my_field }}

```

默认情况下，它是在字段名称的前面加上`id_` （上面的例子中将是“`id_my_field`”）。你可以通过设置字段Widget 的[`attrs`](widgets.html#django.forms.Widget.attrs "django.forms.Widget.attrs") 来修改ID。例如，像这样声明一个字段：

```
my_field = forms.CharField(widget=forms.TextInput(attrs={'id': 'myFIELD'}))

```

使用上面的模板，将渲染成：

```
<label for="myFIELD">...</label><input id="myFIELD" type="text" name="my_field" />

```

## 绑定上传的文件到表单

处理带有`FileField` 和`ImageField` 字段的表单比普通的表单要稍微复杂一点。

首先，为了上传文件，你需要确保你的`<form>` 元素正确定义`enctype` 为`"multipart/form-data"`：

```
<form enctype="multipart/form-data" method="post" action="/foo/">

```

其次，当你使用表单时，你需要绑定文件数据。文件数据的处理与普通的表单数据是分开的，所以如果表单包含`FileField` 和`ImageField`，绑定表单时你需要指定第二个参数。所以，如果我们扩展ContactForm 并包含一个名为`mugshot` 的`ImageField`，我们需要绑定包含mugshot 图片的文件数据：

```
# Bound form with an image field
>>> from django.core.files.uploadedfile import SimpleUploadedFile
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> file_data = {'mugshot': SimpleUploadedFile('face.jpg', <file data>)}
>>> f = ContactFormWithMugshot(data, file_data)

```

实际上，你一般将使用`request.FILES` 作为文件数据的源（和使用`request.POST` 作为表单数据的源一样）：

```
# Bound form with an image field, data from the request
>>> f = ContactFormWithMugshot(request.POST, request.FILES)

```

构造一个未绑定的表单和往常一样 —— 将表单数据_和_文件数据同时省略：

```
# Unbound form with an image field
>>> f = ContactFormWithMugshot()

```

### 测试multipart 表单

`Form.``is_multipart`()

如果你正在编写可重用的视图或模板，你可能事先不知道你的表单是否是一个multipart 表单。`is_multipart()` 方法告诉你表单提交时是否要求multipart：

```
>>> f = ContactFormWithMugshot()
>>> f.is_multipart()
True

```

下面是如何在模板中使用它的一个示例：

```
{% if form.is_multipart %}
    <form enctype="multipart/form-data" method="post" action="/foo/">
{% else %}
    <form method="post" action="/foo/">
{% endif %}
{{ form }}
</form>

```

## 子类化表单

如果你有多个`表单`类共享相同的字段，你可以使用子类化来减少冗余。

当你子类化一个自定义的`表单`类时，生成的子类将包含父类中的所有字段，以及在子类中定义的字段。

在下面的例子中，`ContactFormWithPriority` 包含`ContactForm` 中的所有字段，以及另外一个字段`priority`。排在前面的是`ContactForm` 中的字段：

```
>>> class ContactFormWithPriority(ContactForm):
...     priority = forms.CharField()
>>> f = ContactFormWithPriority(auto_id=False)
>>> print(f.as_ul())
<li>Subject: <input type="text" name="subject" maxlength="100" /></li>
<li>Message: <input type="text" name="message" /></li>
<li>Sender: <input type="email" name="sender" /></li>
<li>Cc myself: <input type="checkbox" name="cc_myself" /></li>
<li>Priority: <input type="text" name="priority" /></li>

```

可以子类化多个表单，将表单作为“mix-ins”。在下面的例子中，`BeatleForm` 子类化`PersonForm` 和 `InstrumentForm` ，所以它的字段列表包含两个父类的所有字段：

```
>>> from django.forms import Form
>>> class PersonForm(Form):
...     first_name = CharField()
...     last_name = CharField()
>>> class InstrumentForm(Form):
...     instrument = CharField()
>>> class BeatleForm(PersonForm, InstrumentForm):
...     haircut_type = CharField()
>>> b = BeatleForm(auto_id=False)
>>> print(b.as_ul())
<li>First name: <input type="text" name="first_name" /></li>
<li>Last name: <input type="text" name="last_name" /></li>
<li>Instrument: <input type="text" name="instrument" /></li>
<li>Haircut type: <input type="text" name="haircut_type" /></li>

```

New in Django 1.7\.
*   在子类中，可以通过设置名字为`None` 来删除从父类中继承的`字段`。例如：

    ```
    >>> from django import forms

    >>> class ParentForm(forms.Form):
    ...     name = forms.CharField()
    ...     age = forms.IntegerField()

    >>> class ChildForm(ParentForm):
    ...     name = None

    >>> ChildForm().fields.keys()
    ... ['age']

    ```

## 表单前缀

`Form.``prefix`

你可以将几个Django 表单放在一个`<form>` 标签中。为了给每个`表单`一个自己的命名空间，可以使用`prefix` 关键字参数：

```
>>> mother = PersonForm(prefix="mother")
>>> father = PersonForm(prefix="father")
>>> print(mother.as_ul())
<li><label for="id_mother-first_name">First name:</label> <input type="text" name="mother-first_name" id="id_mother-first_name" /></li>
<li><label for="id_mother-last_name">Last name:</label> <input type="text" name="mother-last_name" id="id_mother-last_name" /></li>
>>> print(father.as_ul())
<li><label for="id_father-first_name">First name:</label> <input type="text" name="father-first_name" id="id_father-first_name" /></li>
<li><label for="id_father-last_name">Last name:</label> <input type="text" name="father-last_name" id="id_father-last_name" /></li>

```

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Form API](https://docs.djangoproject.com/en/1.8/ref/forms/api/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
