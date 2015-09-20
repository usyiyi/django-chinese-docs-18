# 验证器

## 编写验证器

验证器是一个可调用的对象，它接受一个值，并在不符合一些规则时抛出[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")异常。验证器有助于在不同类型的字段之间重复使用验证逻辑。

例如，这个验证器只允许偶数：

```
from django.core.exceptions import ValidationError

def validate_even(value):
    if value % 2 != 0:
        raise ValidationError('%s is not an even number' % value)

```

你可以通过字段的[`validators`](models/fields.html#django.db.models.Field.validators "django.db.models.Field.validators")参数将它添加到模型字段中：

```
from django.db import models

class MyModel(models.Model):
    even_field = models.IntegerField(validators=[validate_even])

```

由于值在验证器运行之前会转化为Python，你可以在表单上使用相同的验证器：

```
from django import forms

class MyForm(forms.Form):
    even_field = forms.IntegerField(validators=[validate_even])

```

你也可以使用带有&nbsp;`__call__()`方法的类，来实现更复杂或可配置的验证器。例如，[`RegexValidator`](#django.core.validators.RegexValidator "django.core.validators.RegexValidator")就用了这种技巧。如果一个基于类的验证器用于[`validators`](models/fields.html#django.db.models.Field.validators "django.db.models.Field.validators")模型字段的选项，你应该通过添加[_deconstruct()_](../topics/migrations.html#custom-deconstruct-method) 和`__eq__()` 方法确保它可以[_被迁移框架序列化_](../topics/migrations.html#migration-serializing)。

## 验证器如何运行

关于验证器如何在表单中运行，详见[_表单验证_](forms/validation.html) 。关于它们如何在模型中运行，详见&nbsp;[_验证对象_](models/instances.html#validating-objects)。要注意验证器不会在你保存模型时自动运行，但是如果你使用[`ModelForm`](../topics/forms/modelforms.html#django.forms.ModelForm "django.forms.ModelForm")，它会在任何你表单包含的字段上运行你的验证器。关于模型验证器如何和表单交互，详见[_ModelForm 文档_](../topics/forms/modelforms.html)。

## 内建的验证器

[`django.core.validators`](#module-django.core.validators "django.core.validators: Validation utilities and base classes")模块包含了一系列的可调用验证器，用于模型和表单字段。它们在内部使用，但是也可以用在你自己的字段上。它们可以用在`field.clean()` 方法之外，或者代替它。

### RegexValidator

_class _`RegexValidator`([_regex=None_, _message=None_, _code=None_, _inverse_match=None_, _flags=0_])[[source]](../_modules/django/core/validators.html#RegexValidator)

<table class="docutils field-list" frame="void" rules="none"><colgroup><col class="field-name"><col class="field-body"></colgroup><tbody valign="top"><tr class="field-odd field"><th class="field-name">Parameters:</th><td class="field-body">*   **regex** – 如果不是`None`则覆写 [`regex`](#django.core.validators.RegexValidator.regex "django.core.validators.RegexValidator.regex")。可以是一个正则表达式字符串，或者预编译的正则表达式对象。
*   **message** – 如果不是`None`，则覆写 [`message`](#django.core.validators.RegexValidator.message "django.core.validators.RegexValidator.message")。
*   **code** – 如果不是`None`，则覆写[`code`](#django.core.validators.RegexValidator.code "django.core.validators.RegexValidator.code")。
*   **inverse_match** – 如果不是`None`，则覆写[`inverse_match`](#django.core.validators.RegexValidator.inverse_match "django.core.validators.RegexValidator.inverse_match")。
*   **flags** – 如果不是`None`，则覆写 [`flags`](#django.core.validators.RegexValidator.flags "django.core.validators.RegexValidator.flags")。这种情况下，[`regex`](#django.core.validators.RegexValidator.regex "django.core.validators.RegexValidator.regex") ，必须是正则表达式字符串，否则抛出[`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError) 异常。

</td></tr></tbody></table>

`regex`

用于搜索提供的`value`的正则表达式，或者是预编译的正则表达式对象。通常在找不到匹配时抛出带有 [`message`](#django.core.validators.RegexValidator.message "django.core.validators.RegexValidator.message") 和[`code`](#django.core.validators.RegexValidator.code "django.core.validators.RegexValidator.code")的&nbsp;[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")异常。这一标准行为可以通过设置[`inverse_match`](#django.core.validators.RegexValidator.inverse_match "django.core.validators.RegexValidator.inverse_match") 为`True`来反转，这种情况下，如果**找到**匹配则抛出&nbsp;[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")异常。通常它会匹配任何字符串（包括空字符串）。

`message`

验证失败时[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")所使用的错误信息。默认为`"Enter a valid value"`。

`code`

验证失败时[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")所使用的错误代码。默认为`"invalid"`。

`inverse_match`

New in Django 1.7\.

[`regex`](#django.core.validators.RegexValidator.regex "django.core.validators.RegexValidator.regex")的匹配模式。默认为`False`。

`flags`

New in Django 1.7\.

编译正则表达式字符串[`regex`](#django.core.validators.RegexValidator.regex "django.core.validators.RegexValidator.regex")时所用的标识。如果[`regex`](#django.core.validators.RegexValidator.regex "django.core.validators.RegexValidator.regex")是预编译的正则表达式，并且覆写了[`flags`](#django.core.validators.RegexValidator.flags "django.core.validators.RegexValidator.flags")，会产生[`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError)异常。默认为 <cite>0</cite>。

### EmailValidator

_class _`EmailValidator`([_message=None_, _code=None_, _whitelist=None_])[[source]](../_modules/django/core/validators.html#EmailValidator)

<table class="docutils field-list" frame="void" rules="none"><colgroup><col class="field-name"><col class="field-body"></colgroup><tbody valign="top"><tr class="field-odd field"><th class="field-name">Parameters:</th><td class="field-body">*   **message** – 如果不是 `None`，则覆写[`message`](#django.core.validators.EmailValidator.message "django.core.validators.EmailValidator.message")。
*   **code** – 如果不是 `None`，则覆写[`code`](#django.core.validators.EmailValidator.code "django.core.validators.EmailValidator.code")。
*   **whitelist** – 如果不是`None`，则覆写 [`whitelist`](#django.core.validators.EmailValidator.whitelist "django.core.validators.EmailValidator.whitelist")。

</td></tr></tbody></table>

`message`

验证失败时[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")所使用的错误信息。默认为`"Enter a valid email address"`。

`code`

验证失败时[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")所使用的错误代码。默认为`"invalid"`。

`whitelist`

所允许的邮件域名的白名单。通常，正则表达式(`domain_regex` 属性)&nbsp;用于验证&nbsp;@ 符号后面的任何东西。但是，如果这个字符串在白名单里，就可以通过验证。如果没有提供，默认的白名单是 `['localhost']`。其它不包含点符号的域名不能通过验证，所以你需要按需将它们添加进白名单。

### URLValidator

_class _`URLValidator`([_schemes=None_, _regex=None_, _message=None_, _code=None_])[[source]](../_modules/django/core/validators.html#URLValidator)

[`RegexValidator`](#django.core.validators.RegexValidator "django.core.validators.RegexValidator")确保一个值看起来像是URL，并且如果不是的话产生`'invalid'`错误代码。

回送地址以及保留的IP空间被视为有效。同时也支持字面的IPv6地址 ([**RFC 2732**](http://tools.ietf.org/html/rfc2732.html)) 以及unicode域名。

除了父类[`RegexValidator`](#django.core.validators.RegexValidator "django.core.validators.RegexValidator")的可选参数之外，`URLValidator`接受一个额外的可选属性：

`schemes`

需要验证的URL/URI模式列表。如果没有提供，默认为 `['http', 'https', 'ftp', 'ftps']`。IANA 网站提供了 [有效的URI模式](https://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml)的完整列表作为参考。

Changed in Django 1.7:

添加了可选的`schemes` 属性。

Changed in Django 1.8:

添加了对IPv6 地址, unicode 域名, 以及含有验证信息的URL的支持。

### validate_email

`validate_email`

一个不带有任何自定义的[`EmailValidator`](#django.core.validators.EmailValidator "django.core.validators.EmailValidator")实例。

### validate_slug

`validate_slug`

一个&nbsp;[`RegexValidator`](#django.core.validators.RegexValidator "django.core.validators.RegexValidator")实例，确保值只含有字母、数字、下划线和连字符。

### validate_ipv4_address

`validate_ipv4_address`

一个[`RegexValidator`](#django.core.validators.RegexValidator "django.core.validators.RegexValidator")的实例，确保值是IPv4地址。

### validate_ipv6_address

`validate_ipv6_address`[[source]](../_modules/django/core/validators.html#validate_ipv6_address)

使用`django.utils.ipv6` 来检查是否是 IPv6 地址。

### validate_ipv46_address

`validate_ipv46_address`[[source]](../_modules/django/core/validators.html#validate_ipv46_address)

使用`validate_ipv4_address` 和 `validate_ipv6_address` 值是有效的 IPv4 或 IPv6 地址。

### validate_comma_separated_integer_list

`validate_comma_separated_integer_list`

一个[`RegexValidator`](#django.core.validators.RegexValidator "django.core.validators.RegexValidator")的实例，确保值是整数的逗号分隔列表。

### MaxValueValidator

_class _`MaxValueValidator`(_max_value_, _message=None_)[[source]](../_modules/django/core/validators.html#MaxValueValidator)

如果`value` 大于&nbsp;`max_value`，抛出带有`'max_value'`代码的[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError") 异常。

Changed in Django 1.8:

添加了`message`参数。

### MinValueValidator

_class _`MinValueValidator`(_min_value_, _message=None_)[[source]](../_modules/django/core/validators.html#MinValueValidator)

如果`value`小于`min_value`，抛出带有`'min_value'`代码的[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")异常。

Changed in Django 1.8:

添加了`message` 参数。

### MaxLengthValidator

_class _`MaxLengthValidator`(_max_length_, _message=None_)[[source]](../_modules/django/core/validators.html#MaxLengthValidator)

如果`value`的长度大于`max_length`，抛出带有`'max_length'`代码的[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError") 异常。

Changed in Django 1.8:

添加了`message`参数。

### MinLengthValidator

_class _`MinLengthValidator`(_min_length_, _message=None_)[[source]](../_modules/django/core/validators.html#MinLengthValidator)

如果`value`的长度小于`min_length`，抛出带有`'min_length'`代码的[`ValidationError`](exceptions.html#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")异常。

Changed in Django 1.8:

添加了`message` 参数。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Data validation](https://docs.djangoproject.com/en/1.8/ref/validators/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
