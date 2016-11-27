

# 表单验证和字段验证

表单验证发生在数据验证之后。如果你需要定制化这个过程，有几个不同的地方可以修改，每个地方的目的不一样。表单处理过程中要运行三种类别的验证方法。它们通常在你调用表单的`is_valid()` 方法时执行。还有其它方法可以触发验证过程（访问`errors` 属性或直接调用`full_clean()` ），但是通用情况下不需要。

一般情况下，如果处理的数据有问题，每个类别的验证方法都会引发`ValidationError`，并将相关信息传递给`ValidationError`。 [_参见下文_](#raising-validation-error)中引发`ValidationError` 的最佳实践。如果没有引发`ValidationError`，这些方法应该返回验证后的（规整化的）数据的Python 对象。

大部分应该可以使用[validators](#validators) 完成，它们可以很容易地重用。Validators 是简单的函数（或可调用对象），它们接收一个参数并对非法的输入抛出`ValidationError`。 Validators 在字段的`to_python` 和`validate` 方法调用之后运行。

表单的验证划分成几个步骤，它们可以定制或覆盖：

*   字段的`to_python()` 方法是验证的第一步。它将值强制转换为正确的数据类型，如果不能转换则引发`ValidationError`。 这个方法从Widget 接收原始的值并返回转换后的值。例如，FloatField 将数据转换为Python 的`float` 或引发`ValidationError`。

*   字段的`validate()` 方法处理字段特异性的验证，这种验证不适合位于validator 中。它接收一个已经转换成正确数据类型的值， 并在发现错误时引发`ValidationError`。这个方法不返回任何东西且不应该改变任何值。当你遇到不可以或不想放在validator 中的验证逻辑时，应该覆盖它来处理验证。

*   字段的`run_validators()` 方法运行字段的所有Validator，并将所有的错误信息聚合成一个单一的`ValidationError`。你应该不需要覆盖这个方法。

*   Field子类的`clean()` 方法。它负责以正确的顺序运行`to_python`、`validate` 和 `run_validators` 并传播它们的错误。如果任何时刻、任何方法引发`ValidationError`，验证将停止并引发这个错误。这个方法返回验证后的数据，这个数据在后面将插入到表单的 `cleaned_data` 字典中。

*   表单子类中的`clean_<fieldname>()` 方法 —— `<fieldname>` 通过表单中的字段名称替换。这个方法完成于特定属性相关的验证，这个验证与字段的类型无关。这个方法没有任何传入的参数。你需要查找`self.cleaned_data` 中该字段的值，记住此时它已经是一个Python 对象而不是表单中提交的原始字符串（它位于`cleaned_data` 中是因为字段的`clean()` 方法已经验证过一次数据）。

    例如，如果你想验证名为`serialnumber` 的`CharField` 的内容是否唯一， `clean_serialnumber()` 将是实现这个功能的理想之处。你需要的不是一个特别的字段（它只是一个`CharField`），而是一个特定于表单字段特定验证，并规整化数据。

    这个方法返回从cleaned_data 中获取的值，无论它是否修改过。

*   表单子类的`clean()` 方法。这个方法可以实现需要同时访问表单多个字段的验证。这里你可以验证如果提供字段`A`，那么字段`B` 必须包含一个合法的邮件地址以及类似的功能。 这个方法可以返回一个完全不同的字典，该字典将用作`cleaned_data`。

    因为字段的验证方法在调用`clean()` 时会运行，你还可以访问表单的`errors` 属性，它包含验证每个字段时的所有错误。

    注意，你覆盖的[`Form.clean()`](api.html#django.forms.Form.clean "django.forms.Form.clean") 引发的任何错误将不会与任何特定的字段关联。它们位于一个特定的“字段”（叫做`__all__`）中，如果需要可以通过 [`non_field_errors()`](api.html#django.forms.Form.non_field_errors "django.forms.Form.non_field_errors") 方法访问。如果你想添加一个特定字段的错误到表单中，需要调用 [`add_error()`](api.html#django.forms.Form.add_error "django.forms.Form.add_error")。

    还要注意，覆盖`ModelForm` 子类的`clean()` 方法需要特殊的考虑。（更多信息参见[_ModelForm 文档_](../../topics/forms/modelforms.html#overriding-modelform-clean-method)）。

这些方法按以上给出的顺序执行，一次验证一个字段。也就是说，对于表单中的每个字段（按它们在表单定义中出现的顺序），先运行`Field.clean()` ，然后运行`clean_<fieldname>()`。每个字段的这两个方法都执行完之后，最后运行[`Form.clean()`](api.html#django.forms.Form.clean "django.forms.Form.clean") 方法，无论前面的方法是否抛出过异常。

下面有上面每个方法的示例。

我们已经提到过，所有这些方法都可以抛出`ValidationError`。对于每个字段，如果`Field.clean()` 方法抛出 `ValidationError`，那么将不会调用该字段对应的clean_<fieldname>()方法。 但是，剩余的字段的验证方法仍然会执行。



## 抛出`ValidationError`

为了让错误信息更加灵活或容易重写，请考虑下面的准则：

*   给构造函数提供一个富有描述性的错误码`code`：

    

    

```
# Good
    ValidationError(_('Invalid value'), code='invalid')

    # Bad
    ValidationError(_('Invalid value'))
    
```

    

    

*   不要预先将变量转换成消息字符串；使用占位符和构造函数的`params` 参数：

    

    

```
# Good
    ValidationError(
        _('Invalid value: %(value)s'),
        params={'value': '42'},
    )

    # Bad
    ValidationError(_('Invalid value: %s') % value)
    
```

    

    

*   使用字典参数而不要用位置参数。这使得重写错误信息时不用考虑变量的顺序或者完全省略它们：

    

    

```
# Good
    ValidationError(
        _('Invalid value: %(value)s'),
        params={'value': '42'},
    )

    # Bad
    ValidationError(
        _('Invalid value: %s'),
        params=('42',),
    )
    
```

    

    

*   用`gettext` 封装错误消息使得它可以翻译：

    

    

```
# Good
    ValidationError(_('Invalid value'))

    # Bad
    ValidationError('Invalid value')
    
```

    

    

所有的准则放在一起就是：





```
raise ValidationError(
    _('Invalid value: %(value)s'),
    code='invalid',
    params={'value': '42'},
)

```





如果你想编写可重用的表单、表单字段和模型字段，遵守这些准则是非常必要的。

如果你在验证的最后（例如，表单的`clean()` 方法）且知道_永远_ 不需要重新错误信息，虽然不提倡但你仍然可以选择重写不详细的信息：





```
ValidationError(_('Invalid value: %s') % value)

```





New in Django 1.7.

[`Form.errors.as_data()`](api.html#django.forms.Form.errors.as_data "django.forms.Form.errors.as_data") 和[`Form.errors.as_json()`](api.html#django.forms.Form.errors.as_json "django.forms.Form.errors.as_json") 方法很大程度上受益于`ValidationError`（利用`code` 名和`params` 字典）。



### 抛出多个错误

如果在一个验证方法中检查到多个错误并且希望将它们都反馈给表单的提交者，可以传递一个错误的列表给`ValidationError` 构造函数。

和上面一样，建议传递的列表中的`ValidationError` 实例都带有 `code` 和`params`，但是传递一个字符串列表也可以工作：





```
# Good
raise ValidationError([
    ValidationError(_('Error 1'), code='error1'),
    ValidationError(_('Error 2'), code='error2'),
])

# Bad
raise ValidationError([
    _('Error 1'),
    _('Error 2'),
])

```











## 实践验证

前面几节解释在一般情况下表单的验证是如何工作的。因为有时直接看功能在实际中的应用会更容易掌握，下面是一些列小例子，它们用到前面的每个功能。



### 使用Validator

Django 的表单（以及模型）字段支持使用简单的函数和类用于验证，它们叫做Validator。Validator 是可调用对象或函数，它接收一个值，如果该值合法则什么也不返回，否则抛出[`ValidationError`](../exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")。它们可以通过字段的`validators` 参数传递给字段的构造函数，或者定义在[`Field`](fields.html#django.forms.Field "django.forms.Field") 类的`default_validators` 属性中。

简单的Validator 可以用于在字段内部验证值，让我们看下Django 的`SlugField`：





```
from django.forms import CharField
from django.core import validators

class SlugField(CharField):
    default_validators = [validators.validate_slug]

```





正如你所看到的，`SlugField` 只是一个带有自定义Validator 的`CharField`，它们验证提交的文本符合某些字符规则。这也可以在字段定义时实现，所以：





```
slug = forms.SlugField()

```





等同于：





```
slug = forms.CharField(validators=[validators.validate_slug])

```





常见的情形，例如验证邮件地址和正则表达式，可以使用Django 中已经存在的Validator 类处理。例如，`validators.validate_slug` 是[`RegexValidator`](../validators.html#django.core.validators.RegexValidator "django.core.validators.RegexValidator") 的一个实例，它构造时的第一个参数为：`^[-a-zA-Z0-9_]+$`。[_编写Validator_](../validators.html) 一节可以查到已经存在的Validator 以及如何编写Validator 的一个示例。





### 表单字段的默认验证

让我们首先创建一个自定义的表单字段，它验证其输入是一个由逗号分隔的邮件地址组成的字符串。完整的类像这样：





```
from django import forms
from django.core.validators import validate_email

class MultiEmailField(forms.Field):
    def to_python(self, value):
        "Normalize data to a list of strings."

        # Return an empty list if no input was given.
        if not value:
            return []
        return value.split(',')

    def validate(self, value):
        "Check if value consists only of valid emails."

        # Use the parent's handling of required fields, etc.
        super(MultiEmailField, self).validate(value)

        for email in value:
            validate_email(email)

```





使用这个字段的每个表单都将在处理该字段数据之前运行这些方法。这个验证特定于该类型的字段，与后面如何使用它无关。

让我们来创建一个简单的`ContactForm` 来向你演示如何使用这个字段：





```
class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField()
    sender = forms.EmailField()
    recipients = MultiEmailField()
    cc_myself = forms.BooleanField(required=False)

```





只需要简单地使用`MultiEmailField`，就和其它表单字段一样。当调用表单的`is_valid()` 方法时，`MultiEmailField.clean()` 方法将作为验证过程的一部分运行，它将调用自定义的`to_python()` 和`validate()` 方法。





### 验证特定字段属性

继续上面的例子，假设在我们的`ContactForm` 中，我们想确保`recipients` 字段始终包含`"fred@example.com"`。这是特定于我们这个表单的验证，所以我们不打算将它放在通用的`MultiEmailField` 类中。我们将编写一个运行在`recipients` 字段上的验证方法，像这样：





```
from django import forms

class ContactForm(forms.Form):
    # Everything as before.
    ...

    def clean_recipients(self):
        data = self.cleaned_data['recipients']
        if "fred@example.com" not in data:
            raise forms.ValidationError("You have forgotten about Fred!")

        # Always return the cleaned data, whether you have changed it or
        # not.
        return data

```









### 验证相互依赖的字段

假设我们添加另外一个需求到我们的联系人表单中：如果`cc_myself` 字段为`True`，那么`subject` 必须包含单词`"help"`。我们的这个验证包含多个字段，所以表单的[`clean()`](api.html#django.forms.Form.clean "django.forms.Form.clean") 方法是个不错的地方。注意，我们这里讨论的是表单的`clean()` 方法，之前我们编写的字段的`clean()` 方法。区别字段和表单之间的差别非常重要。字段是单个数据，表单是字段的集合。

在调用表单`clean()` 方法的时候，所有字段的验证方法已经执行完（前两节），所以`self.cleaned_data` 填充的是目前为止已经合法的数据。所以你需要记住这个事实，你需要验证的字段可能没有通过初试的字段检查。

在这一步，有两种方法报告错误。最简单的方法是在表单的顶端显示错误。你可以在`clean()` 方法中抛出`ValidationError` 来创建错误。例如：





```
from django import forms

class ContactForm(forms.Form):
    # Everything as before.
    ...

    def clean(self):
        cleaned_data = super(ContactForm, self).clean()
        cc_myself = cleaned_data.get("cc_myself")
        subject = cleaned_data.get("subject")

        if cc_myself and subject:
            # Only do something if both fields are valid so far.
            if "help" not in subject:
                raise forms.ValidationError("Did not send for 'help' in "
                        "the subject despite CC'ing yourself.")

```





Changed in Django 1.7:

在以前版本的Django 中，要求`form.clean()` 返回`cleaned_data` 的一个字典。现在，这个方法仍然可以返回将要用到的数据的字典，但是不再是强制的。



在这段代码中，如果抛出验证错误，表单将在表单的顶部显示（通常是）描述该问题的一个错误信息。

注意，示例代码中`super(ContactForm, self).clean()` 的调用时为了保证维持父类中的验证逻辑。

第二种方法涉及将错误消息关联到某个字段。在这种情况下，让我们在表单的显示中分别关联一个错误信息到“subject” 和“cc_myself” 行。在实际应用中要小心，因为它可能导致表单的输出变得令人困惑。我们只是向你展示这里可以怎么做，在特定的情况下，需要你和你的设计人员确定什么是好的方法。我们的新代码（代替前面的示例）像这样：





```
from django import forms

class ContactForm(forms.Form):
    # Everything as before.
    ...

    def clean(self):
        cleaned_data = super(ContactForm, self).clean()
        cc_myself = cleaned_data.get("cc_myself")
        subject = cleaned_data.get("subject")

        if cc_myself and subject and "help" not in subject:
            msg = "Must put 'help' in subject when cc'ing yourself."
            self.add_error('cc_myself', msg)
            self.add_error('subject', msg)

```





`add_error()` 的第二个参数可以是一个简单的字符串，但更倾向是`ValidationError` 的一个实例。更多细节参见[_抛出ValidationError_](#raising-validation-error)。注意，`add_error()` 将从`cleaned_data` 中删除相应的字段。





