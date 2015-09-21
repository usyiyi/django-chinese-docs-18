{% raw %}

# 使用表单 #

> 关于这页文档
>
> 这页文档简单介绍Web 表单的基本概念和它们在Django 中是如何处理的。关于表单API 某方面的细节，请参见[表单 API、表单的字段和表单和字段的检验](http://python.usyiyi.cn/django/ref/forms/fields.html)。

除非你计划构建的网站和应用只是发布内容而不接受访问者的输入，否则你将需要理解并使用表单。

Django 提供广泛的工具和库来帮助你构建表单来接收网站访问者的输入，然后处理以及响应输入。

## HTML 表单 ##

在HTML中，表单是位于`<form>...</form>` 之间的元素的集合，它们允许访问者输入文本、选择选项、操作对象和控制等等，然后将信息发送回服务器。

某些表单的元素 —— 文本输入和复选框 —— 非常简单而且内建于HTML 本身。其它的表单会复杂些；例如弹出一个日期选择对话框的界面、允许你移动滚动条的界面、使用JavaScript 和CSS 以及HTML 表单`<input>` 元素来实现操作控制的界面。

与`<input>` 元素一样，一个表单必须指定两样东西：

+ where：响应用户输入的URL
+ how：HTTP 方法

例如，Django Admin 站点的登录表单包含几个`<input>` 元素：`type="text"` 用于用户名，`type="password"` 用于密码，`type="submit"` 用于“Log in" 按钮。它还包含一些用户看不到的隐藏的文本字段，Django 使用它们来决定下一步的行为。

它还告诉浏览器表单数据应该发往`<form>` 的`action` 属性指定的URL —— `/admin/`，而且应该使用`method` 属性指定的HTTP 方法 —— `post`。

当触发`<input type="submit" value="Log in">` 元素时，数据将发送给`/admin/`。

### GET 和 POST ###

处理表单时候只会用到`GET `和` POST` 方法。

Django 的登录表单使用POST 方法，在这个方法中浏览器组合表单数据、对它们进行编码以用于传输、将它们发送到服务器然后接收它的响应。

相反，GET 组合提交的数据为一个字符串，然后使用它来生成一个URL。这个URL 将包含数据发送的地址以及数据的键和值。如果你在Django 文档中做一次搜索，你会立即看到这点，此时将生成一个`https://docs.djangoproject.com/search/?q=forms&release=1` 形式的URL。

`GET` 和`POST` 用于不同的目的。

用于改变系统状态的请求 —— 例如，给数据库带来变化的请求 —— 应该使用`POST`。`GET` 只应该用于不会影响系统状态的请求。

`GET` 还不适合密码表单，因为密码将出现在URL 中，以及浏览器的历史和服务器的日志中，而且都是以普通的文本格式。它还不适合数据量大的表单和二进制数据，例如一张图片。使用GET 请求作为管理站点的表单具有安全隐患：攻击者很容易模拟表单请求来取得系统的敏感数据。`POST`，如果与其它的保护措施结合将对访问提供更多的控制，例如Django 的[CSRF 保护](http://python.usyiyi.cn/django/ref/csrf.html)。

另一个方面，`GET` 适合网页搜索这样的表单，因为这种表示一个`GET` 请求的URL 可以很容易地作为书签、分享和重新提交。

Django 在表单中的角色

处理表单是一件很复杂的事情。考虑一下Django 的Admin 站点，不同类型的大量数据项需要在一个表单中准备好、渲染成HTML、使用一个方便的界面编辑、返回给服务器、验证并清除，然后保存或者向后继续处理。

Django 的表单功能可以简化并自动化大部分这些工作，而且还可以比大部分程序员自己所编写的代码更安全。

Django 会处理表单工作中的三个显著不同的部分：

+ 准备并重新构造数据
+ 为数据创建HTML 表单
+ 接收并处理客户端提交的表单和数据

可以手工编写代码来实现，但是Django 可以帮你完成所有这些工作。

## Django 中的表单 ##

我们已经简短讲述HTML 表单，但是HTML的`<form>` 只是其机制的一部分。

在一个Web 应用中，‘表单’可能指HTML `<form>`、或者生成它的Django 的`Form`、或者提交时发送的结构化数据、或者这些部分的总和。

### Django 的Form 类 ###

表单系统的核心部分是Django 的`Form` 类。Django 的模型描述一个对象的逻辑结构、行为以及展现给我们的方式，与此类似，`Form` 类描述一个表单并决定它如何工作和展现。

模型类的字典映射到数据库的字典，与此类似，表单类的字段映射到HTML 的表单`<input>` 元素。（`ModelForm `通过一个`Form` 映射模型类的字段到HTML 表单的`<input> `元素；Django 的Admin 站点就是基于这个）。

表单的字段本身也是类；它们管理表单的数据并在表单提交时进行验证。`DateField` 和`FileField `处理的数据类型差别很大，必须完成不同的事情。

表单字段在浏览器中呈现给用户的是一个HTML 的“widget”  —— 用户界面的一个片段。每个字段类型都有一个合适的默认[Widget 类](http://python.usyiyi.cn/django/ref/forms/widgets.html)，需要时可以覆盖。

### 实例化、处理和渲染表单 ###

在Django 中渲染一个对象时，我们通常：

1. 在视图中获得它（例如，从数据库中获取）
2. 将它传递给模板上下文
3. 使用模板变量将它扩展为HTML 标记

在模板中渲染表单和渲染其它类型的对象几乎一样，除了几个关键的差别。

在模型实例不包含数据的情况下，在模板中对它做处理很少有什么用处。但是渲染一个未填充的表单却非常有意义 —— 我们希望用户去填充它。

所以当我们在视图中处理模型实例时，我们一般从数据库中获取它。当我们处理表单时，我们一般在视图中实例化它。

当我们实例化表单时，我们可以选择让它为空还是预先填充它，例如使用：

+ 来自一个保存后的模型实例的数据（例如用于编辑的管理表单）
+ 我们从其它地方获得的数据
+ 从前面一个HTML 表单提交过来的数据

最后一种情况最令人关注，因为它使得用户可以不只是阅读一个网站，而且可以给网站返回信息。

## 构建一个表单 ##

### 需要完成的工作 ###

假设你想在你的网站上创建一个简单的表单，以获得用户的名字。你需要类似这样的模板：

```
<form action="/your-name/" method="post">
    <label for="your_name">Your name: </label>
    <input id="your_name" type="text" name="your_name" value="{{ current_name }}">
    <input type="submit" value="OK">
</form>
```

这告诉浏览器发送表单的数据到URL `/your-name/`，并使用`POST` 方法。它将显示一个标签为"Your name:"的文本字段，和一个"OK"按钮。如果模板上下文包含一个`current_name` 变量，它将用于预填充`your_name` 字段。

你将需要一个视图来渲染这个包含HTML 表单的模板，并提供合适的`current_name` 字段。

当表单提交时，发往服务器的`POST` 请求将包含表单数据。

现在你还需要一个对应`/your-name/` URL 的视图，它在请求中找到正确的键/值对，然后处理它们。

这是一个非常简单的表单。实际应用中，一个表单可能包含几十上百个字段，其中大部分需要预填充，而且我们预料到用户将来回编辑-提交几次才能完成操作。

我们可能需要在表单提交之前，在浏览器端作一些验证。我们可能想使用非常复杂的字段，以允许用户做类似从日历中挑选日期这样的事情，等等。

这个时候，让Django 来为我们完成大部分工作是很容易的。

### 在Django 中构建一个表单 ###

#### Form 类 ####

我们已经计划好了我们的 HTML 表单应该呈现的样子。在Django 中，我们的起始点是这里：

```
#forms.py

from django import forms

class NameForm(forms.Form):
    your_name = forms.CharField(label='Your name', max_length=100)
```

它定义一个`Form` 类，只带有一个字段（`your_name`）。我们已经对这个字段使用一个友好的标签，当渲染时它将出现在`<label>` 中（在这个例子中，即使我们省略它，我们指定的`label`还是会自动生成）。

字段允许的最大长度通过`max_length` 定义。它完成两件事情。首先，它在HTML 的`<input>` 上放置一个`maxlength="100"` （这样浏览器将在第一时间阻止用户输入多于这个数目的字符）。它还意味着当Django 收到浏览器发送过来的表单时，它将验证数据的长度。

`Form` 的实例具有一个`is_valid()` 方法，它为所有的字段运行验证的程序。当调用这个方法时，如果所有的字段都包含合法的数据，它将：

+ 返回`True`
+ 将表单的数据放到`cleaned_data `属性中。

完整的表单，第一次渲染时，看上去将像：

```
<label for="your_name">Your name: </label>
<input id="your_name" type="text" name="your_name" maxlength="100">
```

注意它不包含 `<form>` 标签和提交按钮。我们必须自己在模板中提供它们。

#### 视图 ####

发送给Django 网站的表单数据通过一个视图处理，一般和发布这个表单的是同一个视图。这允许我们重用一些相同的逻辑。

当处理表单时，我们需要在视图中实例化它：

```
#views.py

from django.shortcuts import render
from django.http import HttpResponseRedirect

from .forms import NameForm

def get_name(request):
    # if this is a POST request we need to process the form data
    if request.method == 'POST':
        # create a form instance and populate it with data from the request:
        form = NameForm(request.POST)
        # check whether it's valid:
        if form.is_valid():
            # process the data in form.cleaned_data as required
            # ...
            # redirect to a new URL:
            return HttpResponseRedirect('/thanks/')

    # if a GET (or any other method) we'll create a blank form
    else:
        form = NameForm()

    return render(request, 'name.html', {'form': form})
```

如果访问视图的是一个`GET` 请求，它将创建一个空的表单实例并将它放置到要渲染的模板的上下文中。这是我们在第一个访问该URL 时预期发生的情况。

如果表单的提交使用`POST` 请求，那么视图将再次创建一个表单实例并使用请求中的数据填充它：`form = NameForm(request.POST)`。这叫做”绑定数据至表单“（它现在是一个绑定的表单）。

我们调用表单的`is_valid() `方法；如果它不为`True`，我们将带着这个表单返回到模板。这时表单不再为空（未绑定），所以HTML 表单将用之前提交的数据填充，然后可以根据要求编辑并改正它。

如果`is_valid() `为`True`，我们将能够在`cleaned_data` 属性中找到所有合法的表单数据。在发送HTTP 重定向给浏览器告诉它下一步的去向之前，我们可以用这个数据来更新数据库或者做其它处理。

#### 模板 ####

我们不需要在name.html 模板中做很多工作。最简单的例子是：

```
<form action="/your-name/" method="post">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="Submit" />
</form>
```

根据`{{ form }}`，所有的表单字段和它们的属性将通过Django 的模板语言拆分成HTML 标记 。

> 表单和跨站请求伪造的防护
>
> Django 原生支持一个简单易用的[跨站请求伪造的防护](http://python.usyiyi.cn/django/ref/csrf.html)。当提交一个启用CSRF 防护的`POST` 表单时，你必须使用上面例子中的`csrf_token` 模板标签。然而，因为CSRF 防护在模板中不是与表单直接捆绑在一起的，这个标签在这篇文档的以下示例中将省略。

> HTML5 输入类型和浏览器验证
>
> 如果你的表单包含`URLField`、`EmailField` 和其它整数字段类似，Django 将使用`url`、`email`和 `number` 这样的HTML5 输入类型。默认情况下，浏览器可能会对这些字段进行它们自身的验证，这些验证可能比Django 的验证更严格。如果你想禁用这个行为，请设置`form` 标签的`novalidate` 属性，或者指定一个不同的字段，如`TextInput`。

现在我们有了一个可以工作的网页表单，它通过Django Form 描述、通过视图处理并渲染成一个HTML `<form>`。

这是你入门所需要知道的所有内容，但是表单框架为了提供了更多的内容。一旦你理解了上面描述的基本处理过程，你应该可以理解表单系统的其它功能并准备好学习更多的底层机制。

## Django Form 类详解 ##

所有的表单类都作为`django.forms.Form` 的子类创建，包括你在Django 管理站点中遇到的`ModelForm`。

> 模型和表单
>
> 实际上，如果你的表单打算直接用来添加和编辑Django 的模型，[ModelForm](http://python.usyiyi.cn/django/topics/forms/modelforms.html) 可以节省你的许多时间、精力和代码，因为它将根据`Model` 类构建一个表单以及适当的字段和属性。

### 绑定的和未绑定的表单实例 ###

绑定的和未绑定的表单 之间的区别非常重要：

+ 未绑定的表单没有关联的数据。当渲染给用户时，它将为空或包含默认的值。
+ 绑定的表单具有提交的数据，因此可以用来检验数据是否合法。如果渲染一个不合法的绑定的表单，它将包含内联的错误信息，告诉用户如何纠正数据。

表单的`is_bound` 属性将告诉你一个表单是否具有绑定的数据。

### 字段详解 ###

考虑一个比上面的迷你示例更有用的一个表单，我们可以用它来在一个个人网站上实现“联系我”功能：

```
#forms.py

from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField(widget=forms.Textarea)
    sender = forms.EmailField()
    cc_myself = forms.BooleanField(required=False)
```

我们前面的表单只使用一个字段`your_name`，它是一个`CharField`。在这个例子中，我们的表单具有四个字段：`subject`、`message`、`sender` 和`cc_myself`。共用到三种字段类型：`CharField`、`EmailField` 和 `BooleanField`；完整的字段类型列表可以在表单字段中找到。

#### Widgets ####

每个表单字段都有一个对应的`Widget` 类，它对应一个HTML 表单`Widget`，例如`<input type="text">`。

在大部分情况下，字段都具有一个合理的默认Widget。例如，默认情况下，`CharField` 具有一个`TextInput Widget`，它在HTML 中生成一个`<input type="text">`。如果你需要`<textarea>`，在定义表单字段时你应该指定一个合适的`Widget`，例如我们定义的`message` 字段。

#### 字段的数据 ####

不管表单提交的是什么数据，一旦通过调用`is_valid()` 成功验证（`is_valid()` 返回`True`），验证后的表单数据将位于`form.cleaned_data` 字典中。这些数据已经为你转换好为Python 的类型。

> 注
>
> 此时，你依然可以从`request.POST` 中直接访问到未验证的数据，但是访问验证后的数据更好一些。

在上面的联系表单示例中，`cc_myself` 将是一个布尔值。类似地，`IntegerField` 和`FloatField` 字段分别将值转换为Python 的`int` 和`float`。

下面是在视图中如何处理表单数据：

```
#views.py

from django.core.mail import send_mail

if form.is_valid():
    subject = form.cleaned_data['subject']
    message = form.cleaned_data['message']
    sender = form.cleaned_data['sender']
    cc_myself = form.cleaned_data['cc_myself']

    recipients = ['info@example.com']
    if cc_myself:
        recipients.append(sender)

    send_mail(subject, message, sender, recipients)
    return HttpResponseRedirect('/thanks/')
```

> 提示
>
> 关于Django 中如何发送邮件的更多信息，请参见发送邮件。

有些字段类型需要一些额外的处理。例如，使用表单上传的文件需要不同地处理（它们可以从`request.FILES` 获取，而不是`request.POST`）。如何使用表单处理文件上传的更多细节，请参见[绑定上传的文件到一个表单](http://python.usyiyi.cn/django/ref/forms/api.html#binding-uploaded-files)。

## 使用表单模板 ##

你需要做的就是将表单实例放进模板的上下文。如果你的表单在`Contex`t 中叫做`form`，那么` {{ form }} `将正确地渲染它的`<label>` 和 `<input>`元素。

### 表单渲染的选项 ###

> 表单模板的额外标签
>
> 不要忘记，表单的输出不 包含`<form> `标签，和表单的`submit` 按钮。你必须自己提供它们。

对于`<label>/<input>` 对，还有几个输出选项：

+ `{{ form.as_table }}` 以表格的形式将它们渲染在`<tr>` 标签中
+ `{{ form.as_p }} ` 将它们渲染在`<p>` 标签中
+ `{{ form.as_ul }}` 将它们渲染在`<li>` 标签中

注意，你必须自己提供`<table>` 或`<ul>` 元素。

下面是我们的`ContactForm` 实例的输出`{{ form.as_p }}`：

```
<p><label for="id_subject">Subject:</label>
    <input id="id_subject" type="text" name="subject" maxlength="100" /></p>
<p><label for="id_message">Message:</label>
    <input type="text" name="message" id="id_message" /></p>
<p><label for="id_sender">Sender:</label>
    <input type="email" name="sender" id="id_sender" /></p>
<p><label for="id_cc_myself">Cc myself:</label>
    <input type="checkbox" name="cc_myself" id="id_cc_myself" /></p>
```

注意，每个表单字段具有一个`ID `属性并设置为`id_<field-name>`，它被一起的`label` 标签引用。它对于确保屏幕阅读软件这类的辅助计算非常重要。你还可以[自定义label 和 id 生成的方式](http://python.usyiyi.cn/django/ref/forms/api.html#ref-forms-api-configuring-label)。

更多信息参见 [输出表单为HTML](http://python.usyiyi.cn/django/ref/forms/api.html#ref-forms-api-outputting-html)。

### 手工渲染字段 ###

我们没有必要非要让Django 来分拆表单的字段；如果我们喜欢，我们可以手工来做（例如，这样允许重新对字段排序）。每个字段都是表单的一个属性，可以使用`{{ form.name_of_field }}` 访问，并将在Django 模板中正确地渲染。例如：

```
{{ form.non_field_errors }}
<div class="fieldWrapper">
    {{ form.subject.errors }}
    <label for="{{ form.subject.id_for_label }}">Email subject:</label>
    {{ form.subject }}
</div>
<div class="fieldWrapper">
    {{ form.message.errors }}
    <label for="{{ form.message.id_for_label }}">Your message:</label>
    {{ form.message }}
</div>
<div class="fieldWrapper">
    {{ form.sender.errors }}
    <label for="{{ form.sender.id_for_label }}">Your email address:</label>
    {{ form.sender }}
</div>
<div class="fieldWrapper">
    {{ form.cc_myself.errors }}
    <label for="{{ form.cc_myself.id_for_label }}">CC yourself?</label>
    {{ form.cc_myself }}
</div>
```

完整的`<label>` 元素还可以使用`label_tag()` 生成。例如：

```
<div class="fieldWrapper">
    {{ form.subject.errors }}
    {{ form.subject.label_tag }}
    {{ form.subject }}
</div>
```

#### 渲染表单的错误信息 ####

当然，这个便利性的代价是更多的工作。直到现在，我们没有担心如何展示错误信息，因为Django 已经帮我们处理好。在下面的例子中，我们将自己处理每个字段的错误和表单整体的各种错误。注意，表单和模板顶部的`{{ form.non_field_errors }}` 查找每个字段的错误。

使用`{{ form.name_of_field.errors }}` 显示表单错误的一个清单，并渲染成一个`ul`。看上去可能像：

```
<ul class="errorlist">
    <li>Sender is required.</li>
</ul>
```

这个`ul` 有一个`errorlist` CSS 类型，你可以用它来定义外观。如果你希望进一步自定义错误信息的显示，你可以迭代它们来实现：

```
{% if form.subject.errors %}
    <ol>
    {% for error in form.subject.errors %}
        <li><strong>{{ error|escape }}</strong></li>
    {% endfor %}
    </ol>
{% endif %}
```

空字段错误（以及使用`form.as_p()` 时渲染的隐藏字段错误）将渲染成一个额外的CSS 类型`nonfield` 以帮助区分每个字段的错误信息。例如，`{{ form.non_field_errors }}` 看上去会像：

```
<ul class="errorlist nonfield">
    <li>Generic validation error</li>
</ul>
```

```
Changed in Django 1.8:

添加上面示例中提到的nonfield CSS 类型。
```

参见[Forms API](http://python.usyiyi.cn/django/ref/forms/api.html) 以获得关于错误、样式以及在模板中使用表单属性的更多内容。

### 迭代表单的字段 ###

如果你为你的表单使用相同的HTML，你可以使用`{% for %}` 循环迭代每个字段来减少重复的代码：

```
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
    </div>
{% endfor %}
```

`{{ field }}` 中有用的属性包括：

`{{ field.label }}`

字段的`label`，例如`Email address`。

`{{ field.label_tag }}`

包含在HTML `<label>` 标签中的字段`Label`。它包含表单的`label_suffix`。例如，默认的`label_suffix` 是一个冒号：

```
<label for="id_email">Email address:</label>
```

`{{ field.id_for_label }}`

用于这个字段的`ID`（在上面的例子中是`id_email`）。如果你正在手工构造`label`，你可能想使用它代替`label_tag`。如果你有一些内嵌的JavaScript 并且想避免硬编码字段的`ID`，这也是有用的。

`{{ field.value }}`

字段的值，例如`someone@example.com`。

`{{ field.html_name }}`

输入元素的`name` 属性中将使用的名称。它将考虑到表单的前缀。

`{{ field.help_text }}`

与该字段关联的帮助文档。

`{{ field.errors }}`

输出一个`<ul class="errorlist">`，包含这个字段的验证错误信息。你可以使用`{% for error in field.errors %}`自定义错误的显示。 这种情况下，循环中的每个对象只是一个包含错误信息的简单字符串。

`{{ field.is_hidden }}`

如果字段是隐藏字段，则为`True`，否则为`False`。作为模板变量，它不是很有用处，但是可以用于条件测试，例如：

```
{% if field.is_hidden %}

{% endif %}
```

`{{ field.field }}`

表单类中的`Field` 实例，通过`BoundField` 封装。你可以使用它来访问`Field` 属性，例如`{% char_field.field.max_length %}。`

#### 迭代隐藏和可见的字段 ####

如果你正在手工布局模板中的一个表单，而不是依赖Django 默认的表单布局，你可能希望将`<input type="hidden">` 字段与非隐藏的字段区别对待。例如，因为隐藏的字段不会显示，在该字段旁边放置错误信息可能让你的用户感到困惑 —— 所以这些字段的错误应该有区别地来处理。

Django 提供两个表单方法，它们允许你独立地在隐藏的和可见的字段上迭代：`hidden_fields()` 和`visible_fields()`。下面是使用这两个方法对前面一个例子的修改：

```
{% for hidden in form.hidden_fields %}
{{ hidden }}
{% endfor %}

{% for field in form.visible_fields %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
    </div>
{% endfor %}
```

这个示例没有处理隐藏字段中的任何错误信息。通常，隐藏字段中的错误意味着表单被篡改，因为正常的表单填写不会改变它们。然而，你也可以很容易地为这些表单错误插入一些错误信息显示出来。

### 可重用的表单模板 ###

如果你的网站在多个地方对表单使用相同的渲染逻辑，你可以保存表单的循环到一个单独的模板中来减少重复，然后在其它模板中使用`include` 标签来重用它：

```
# In your form template:
{% include "form_snippet.html" %}

# In form_snippet.html:
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
    </div>
{% endfor %}
```

如果传递到模板上下文中的表单对象具有一个不同的名称，你可以使用`include` 标签的`with` 参数来对它起个别名：

```
{% include "form_snippet.html" with form=comment_form %}
```

如果你发现自己经常这样做，你可能需要考虑一下创建一个自定义的` inclusion `标签。

## 更深入的主题 ##

这里只是基础，表单还可以完成更多的工作：

+ [表单集](http://python.usyiyi.cn/django/topics/forms/formsets.html)
  + [在表单集中使用初始化数据](http://python.usyiyi.cn/django/topics/forms/formsets.html#using-initial-data-with-a-formset)
  + [限制表单的最大数目](http://python.usyiyi.cn/django/topics/forms/formsets.html#limiting-the-maximum-number-of-forms)
  + [表单集的验证](http://python.usyiyi.cn/django/topics/forms/formsets.html#formset-validation)
  + [验证表单集中表单的数目](http://python.usyiyi.cn/django/topics/forms/formsets.html#validating-the-number-of-forms-in-a-formset)
  + [处理表单的排序和删除](http://python.usyiyi.cn/django/topics/forms/formsets.html#dealing-with-ordering-and-deletion-of-forms)
  + [添加额外的字段到表单中](http://python.usyiyi.cn/django/topics/forms/formsets.html#adding-additional-fields-to-a-formset)
  + [在视图和模板中视图表单集](http://python.usyiyi.cn/django/topics/forms/formsets.html#using-a-formset-in-views-and-templates)
+ [从模型中创建表单](http://python.usyiyi.cn/django/topics/forms/modelforms.html)
  + [ModelForm](http://python.usyiyi.cn/django/topics/forms/modelforms.html#modelform)
  + [模型表单集](http://python.usyiyi.cn/django/topics/forms/modelforms.html#model-formsets)
  + [Inline formsets](http://python.usyiyi.cn/django/topics/forms/modelforms.html#inline-formsets)
+ [表单集（`Media` 类）](http://python.usyiyi.cn/django/topics/forms/media.html)
  + [Assets as a static definition](http://python.usyiyi.cn/django/topics/forms/media.html#assets-as-a-static-definition)
  + [`Media` as a dynamic property](http://python.usyiyi.cn/django/topics/forms/media.html#media-as-a-dynamic-property)
  + [Paths in asset definitions](http://python.usyiyi.cn/django/topics/forms/media.html#paths-in-asset-definitions)
  + [`Media` 对象](http://python.usyiyi.cn/django/topics/forms/media.html#media-objects)
  + [表单中的`Media`](http://python.usyiyi.cn/django/topics/forms/media.html#media-on-forms)

> 另见
>
> [表单参考](http://python.usyiyi.cn/django/ref/forms/index.html)
> 覆盖完整的API 参考，包括表单字段、表单Widget 以及表单和字段的验证。

&zwj;

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Overview](https://docs.djangoproject.com/en/1.8/topics/forms/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。

{% endraw %}
