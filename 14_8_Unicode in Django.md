

# Unicode数据

Django所有地方都原生地支持Unicode数据。只要你的数据库能存储数据，你就可以安全地把Unicode字符串传递到模板、模型和数据库中。

本文档告诉你如果当你写用到非ASCII的数据或者模板的应用时，你需要知道什么。

## 创建数据库

确认你的数据库配置可以存储任意字符串数据。一般来讲，这意味着给它一个UTF-8或者UTF-16的编码方式。如果你用了更具约束性的编码 – 例如latin1 (iso8859-1) – 你将无法存储某些特定的字符到数据库，并且这些信息也会丢失。

*   MySQL用户，有关如何设置或更改数据库字符集编码的详细信息，请参阅[MySQL手册](http://dev.mysql.com/doc/refman/5.6/en/charset-database.html)。
*   PostgreSQL用户，有关使用正确的编码创建数据库的详细信息，请参阅[PostgreSQL手册](http://www.postgresql.org/docs/current/static/multibyte.html)（PostgreSQL 9中的第22.3.2节）。
*   SQLite用户，没有什么你需要做的。SQLite总是使用UTF-8进行内部编码。

所有Django的数据库后端都自动将Unicode字符串转换为与数据库通信的适当编码。它们还自动将从数据库检索的字符串转换为Python Unicode字符串。你甚至不需要告诉Django你的数据库使用什么编码：这是透明地处理。

有关更多信息，请参阅下面的“数据库API”部分。

## 一般字符串处理

每当你使用Django字符串 - 例如，在数据库查找，模板渲染或其他地方 - 你有两个选择来编码这些字符串。您可以使用Unicode字符串，也可以使用使用UTF-8编码的普通字符串（有时称为“bytestrings”）。

在Python 3中，逻辑是相反的，这是正常的字符串是Unicode，当你想要专门创建一个bytestring，你必须在字符串前面加一个'b'。正如我们在1.5版本的Django代码中所做的，我们建议您从代码中的__future__库中导入`unicode_literals`。然后，当您特别要创建一个bytestring字面量时，在字符串前面加上'b'。

Python 2的遗产：

```
my_string = "This is a bytestring"
my_unicode = u"This is an Unicode string"

```

Python 2与unicode文字或Python 3：

```
from __future__ import unicode_literals

my_string = b"This is a bytestring"
my_unicode = "This is an Unicode string"

```

另请参见[_Python 3 compatibility_](../topics/python3.html)。

警告：

一个字节不包含关于它的编码的任何信息。为此，我们必须做一个假设，Django假设所有的bytestrings都是UTF-8。

如果你传递一个字符串到Django已经编码的其他格式，事情会出错的有趣的方式。通常，Django在某个时刻会引发`UnicodeDecodeError`。

如果您的代码只使用ASCII数据，那么可以安全地使用正常的字符串，随意传递它们，因为ASCII是UTF-8的子集。

不要被欺骗，认为如果您的[`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET)设置设置为`'utf-8'`以外的其他设置，您可以在您的bytestrings中使用其他编码！[`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET)仅适用于作为模板呈现（和电子邮件）的结果生成的字符串。Django将始终假定内部bytestrings的UTF-8编码。原因是[`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET)设置实际上不在您的控制之下（如果您是应用程序开发人员）。它是在安装和使用您的应用程序的人的控制下 - 如果该人选择不同的设置，您的代码必须仍然继续工作。Ergo，它不能依赖那个设置。

在大多数情况下，当Django处理字符串时，它会将它们转换为Unicode字符串，然后再做其他事情。所以，作为一般规则，如果你传递一个字节，准备在结果中接收一个Unicode字符串。

### 翻译字符串

除了Unicode字符串和bytestrings，有使用Django时可能遇到的第三种类型的字符串对象。框架的国际化特性引入了一个“延迟翻译”的概念 - 一个字符串，标记为已翻译，但实际的翻译结果直到对象在字符串中使用才被确定。在使用字符串之前翻译语言环境未知的情况下，此功能非常有用，即使字符串最初可能是在首次导入代码时创建的。

通常，你不必担心延迟翻译。只要注意，如果你检查一个对象，它声称是一个`django.utils.functional.__proxy__`对象，它是一个延迟的翻译。以延迟转换作为参数调用`unicode()`将在当前语言环境中生成一个Unicode字符串。

有关延迟翻译对象的更多详细信息，请参阅[_internationalization_](../topics/i18n/index.html)文档。

### 有用的实用功能

因为一些字符串操作一次又一次地出现，Django附带了一些有用的函数，应该使用Unicode和bytestring对象更容易一些。

#### 转换函数

`django.utils.encoding`模块包含一些方便在Unicode和bytestrings之间来回转换的函数。

*   `smart_text(s, encoding='utf-8', strings_only=False, errors='strict')` 将输入内容转换为Unicode编码，`encoding` 参数用来指定输入的编码. （例如，Django在处理表单输入数据时使用此内部数据，这可能不是UTF-8编码的。） `strings_only` 参数, 当设定为True, 输入的 numbers, booleans , `None` 并不会转换成字符类型 (保留原来的类型). `errors`参数使用Python的`unicode()`函数接受的任何值来处理它的错误。

    如果传递`smart_text()`对象具有`__unicode__`方法，它将使用该方法进行转换。

*   `force_text（s， encoding ='utf-8'， strings_only = False， errors ='strict'）`与`smart_text()`完全相同。区别在于第一个参数是[_lazy translation_](../topics/i18n/translation.html#lazy-translations)实例。当`smart_text()`保留延迟翻译时，`force_text()`将这些对象强制为Unicode字符串（导致翻译发生）。通常，您需要使用`smart_text()`。但是，`force_text()`在绝对_必须_具有要处理的字符串的模板标记和过滤器中很有用，而不仅仅是可以转换为字符串的内容。

*   `smart_bytes（s， encoding ='utf-8'， strings_only = False， errors ='strict'）`本质上与`smart_text()`相反。它强制第一个参数为一个字节。`strings_only`参数与`smart_text()`和`force_text()`具有相同的行为。这与Python的内置`str()`函数略有不同，但是在Django内部的几个地方需要区别。

通常，您只需要使用`smart_text()`。尽可能早地调用任何可能是Unicode或bytestring的输入数据，从那时起，可以将结果视为一直是Unicode。

#### URI和IRI处理

Web框架必须处理URL（这是一种[IRI](http://www.ietf.org/rfc/rfc3987.txt)）。URL的一个要求是它们仅使用ASCII字符编码。但是，在国际环境中，您可能需要从[IRI](http://www.ietf.org/rfc/rfc3987.txt)（非常宽松地说，[URI](http://www.ietf.org/rfc/rfc2396.txt)）构造一个可以包含Unicode字符的URL。引用和转换IRI到URI可能有点棘手，因此Django提供了一些帮助。

*   函数[`django.utils.encoding.iri_to_uri()`](utils.html#django.utils.encoding.iri_to_uri "django.utils.encoding.iri_to_uri")根据规范（ [**RFC 3987**](http://tools.ietf.org/html/rfc3987.html#section-3.1)）的要求实现从IRI到URI的转换。
*   The functions [`django.utils.http.urlquote()`](utils.html#django.utils.http.urlquote "django.utils.http.urlquote") and [`django.utils.http.urlquote_plus()`](utils.html#django.utils.http.urlquote_plus "django.utils.http.urlquote_plus") are versions of Python’s standard `urllib.quote()` and `urllib.quote_plus()` that work with non-ASCII characters. （数据在编码前转换为UTF-8.）

这两组功能的目的略有不同，重要的是保持它们的直线。通常，您可以在IRI或URI路径的各个部分使用`urlquote()`，以便正确编码任何保留字符，例如'＆'或'％'。然后，将`iri_to_uri()`应用于完整的IRI，并将任何非ASCII字符转换为正确的编码值。

注意

从技术上讲，`iri_to_uri()`在IRI规范中实现了完整的算法是不正确的。它没有（还）执行算法的国际域名编码部分。

`iri_to_uri()`函数不会更改URL中允许的ASCII字符。因此，例如，当传递给`iri_to_uri()`时，字符'％'不会进一步编码。这意味着你可以传递一个完整的URL到这个函数，它不会弄乱查询字符串或类似的东西。

一个例子可能会在这里澄清一下：

```
>>> urlquote('Paris & Orléans')
'Paris%20%26%20Orl%C3%A9ans'
>>> iri_to_uri('/favorites/François/%s' % urlquote('Paris & Orléans'))
'/favorites/Fran%C3%A7ois/Paris%20%26%20Orl%C3%A9ans'

```

如果仔细查看，可以看到第二个示例中由`urlquote()`生成的部分在传递到`iri_to_uri()`时没有双引号。这是一个非常重要和有用的功能。这意味着你可以构造你的IRI，而不必担心它是否包含非ASCII字符，然后，在结束，调用`iri_to_uri()`在结果。

类似地，Django提供[`django.utils.encoding.uri_to_iri()`](utils.html#django.utils.encoding.uri_to_iri "django.utils.encoding.uri_to_iri")，它实现了根据 [**RFC 3987**](http://tools.ietf.org/html/rfc3987.html#section-3.2)从URI到IRI的转换。它对除了不表示有效UTF-8序列的那些编码之外的所有百分比编码进行解码。

示例：

```
>>> uri_to_iri('/%E2%99%A5%E2%99%A5/?utf8=%E2%9C%93')
'/♥♥/?utf8=✓'
>>> uri_to_iri('%A9helloworld')
'%A9helloworld'

```

在第一个示例中，UTF-8字符和保留字符未引用。在第二个，百分比编码保持不变，因为它位于有效的UTF-8范围之外。

`iri_to_uri()`和`uri_to_iri()`函数是幂等的，这意味着以下内容总是为真：

```
iri_to_uri(iri_to_uri(some_string)) == iri_to_uri(some_string)
uri_to_iri(uri_to_iri(some_string)) == uri_to_iri(some_string)

```

因此，您可以安全地在同一URI / IRI上多次调用它，而不会冒双重引用问题。

## 楷模

因为所有字符串都作为Unicode字符串从数据库返回，所以当Django从数据库中检索数据时，基于字符（CharField，TextField，URLField等）的模型字段将包含Unicode值。这是_始终_的情况，即使数据可以适合ASCII测试。

您可以在创建模型或填充字段时传递bytestrings，Django会在需要时将其转换为Unicode。

### 选择之间`__str__()`

注意

如果你使用Python 3，可以跳过这一部分，因为你总是创建`__str__()`而不是`__unicode__()`。如果你想与Python 2兼容，你可以用[`python_2_unicode_compatible()`](utils.html#django.utils.encoding.python_2_unicode_compatible "django.utils.encoding.python_2_unicode_compatible")装饰你的模型类。

缺省情况下使用Unicode的一个结果是，在打印模型中的数据时需要小心。

特别是，我们建议您实现`__unicode__()`方法，而不是为您的模型提供`__str__()`在`__unicode__()`方法中，您可以非常安全地返回所有字段的值，而不必担心它们是否适合一个字节。（Python的工作方式，`__str__()`的结果是_总是_一个bytestring，即使你不小心尝试返回一个Unicode对象）。

你仍然可以在模型上创建一个`__str__()`方法，当然，除非你有很好的理由，否则你不应该这样做。Django的`Model`基类自动提供了调用`__unicode__()`的`__str__()`实现，并将结果编码为UTF-8。这意味着你通常只需要实现一个`__unicode__()`方法，并让Django在需要时处理强制的字节。

### 照顾`get_absolute_url()`

网址只能包含ASCII字符。如果要从可能是非ASCII的数据片段构造URL，请小心地以适合URL的方式对结果进行编码。[`reverse()`](urlresolvers.html#django.core.urlresolvers.reverse "django.core.urlresolvers.reverse")函数会自动处理此事件。

如果您手动构建网址（即_而不是_使用`reverse()`函数），则需要自己处理编码。在这种情况下，请使用[上面](#id1)中记录的`iri_to_uri()`和`urlquote()`函数。例如：

```
from django.utils.encoding import iri_to_uri
from django.utils.http import urlquote

def get_absolute_url(self):
    url = '/person/%s/?x=0&y=0' % urlquote(self.location)
    return iri_to_uri(url)

```

即使`self.location`类似于“Jack访问Paris＆Orléans”，此函数也会返回正确编码的网址。（实际上，在上面的例子中，`iri_to_uri()`调用不是绝对必要的，因为所有非ASCII字符在第一行的引号中都会被删除。）

## 数据库API

您可以将Unicode字符串或UTF-8 bytestrings作为参数传递给数据库API中的`filter()`方法等。以下两个查询集是相同的：

```
from __future__ import unicode_literals

qs = People.objects.filter(name__contains='Å')
qs = People.objects.filter(name__contains=b'\xc3\x85') # UTF-8 encoding of Å

```

## 模板

在手动创建模板时，可以使用Unicode或bytestrings：

```
from __future__ import unicode_literals
from django.template import Template
t1 = Template(b'This is a bytestring template.')
t2 = Template('This is a Unicode template.')

```

但是常见的情况是从文件系统读取模板，这产生了一个轻微的复杂性：并非所有文件系统都存储编码为UTF-8的数据。如果模板文件未以UTF-8编码存储，请将[`FILE_CHARSET`](settings.html#std:setting-FILE_CHARSET)设置设置为磁盘上文件的编码。当Django读取模板文件时，它会将数据从此编码转换为Unicode。（默认情况下，[`FILE_CHARSET`](settings.html#std:setting-FILE_CHARSET)设置为`'utf-8'`。）

[`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET)设置控制着色模板的编码。默认情况下，它设置为UTF-8。

### 模板标签和过滤器

在编写自己的模板代码和过滤器时要记住的几个提示：

*   始终从模板标签的`render()`方法和模板过滤器返回Unicode字符串。
*   在这些地方使用`force_text()`优先于`smart_text()`。标签渲染和过滤器调用在渲染模板时发生，因此延迟将延迟翻译对象转换为字符串没有任何优势。在这一点上更容易单独使用Unicode字符串。

## 电子邮件

Django的电子邮件框架（在`django.core.mail`）支持Unicode透明。您可以在邮件正文和任何标题中使用Unicode数据。但是，您仍然有义务遵守电子邮件规范的要求，因此，例如，电子邮件地址应该只使用ASCII字符。

以下代码示例演示除电子邮件地址之外的所有内容都可以是非ASCII：

```
from __future__ import unicode_literals
from django.core.mail import EmailMessage

subject = 'My visit to Sør-Trøndelag'
sender = 'Arnbjörg Ráðormsdóttir <arnbjorg@example.com>'
recipients = ['Fred <fred@example.com']
body = '...'
msg = EmailMessage(subject, body, sender, recipients)
msg.attach("Une pièce jointe.pdf", "%PDF-1.4.%...", mimetype="application/pdf")
msg.send()

```

## 表单提交

HTML表单提交是一个棘手的领域。不能保证提交将包括编码信息，这意味着框架可能必须猜测提交的数据的编码。

Django采用“惰性”方法来解码表单数据。`HttpRequest`对象中的数据只有在访问它时才会被解码。事实上，大多数数据根本没有被解码。只有`HttpRequest.GET`和`HttpRequest.POST`数据结构具有应用于它们的任何解码。这两个字段将返回其成员作为Unicode数据。`HttpRequest`的所有其他属性和方法都与客户端提交的完全相同。

默认情况下，[`DEFAULT_CHARSET`](settings.html#std:setting-DEFAULT_CHARSET)设置用作表单数据的假设编码。如果您需要针对特定​​表单更改此设置，可以在`HttpRequest`实例上设置`encoding`属性。例如：

```
def some_view(request):
    # We know that the data must be encoded as KOI8-R (for some reason).
    request.encoding = 'koi8-r'
    ...

```

您甚至可以在访问`request.`GET或`request.`POST，并且所有后续访问将使用新的编码。

大多数开发人员不需要担心更改表单编码，但这对于与不能控制其编码的传统系统交谈的应用程序是一个有用的功能。

Django不会解码文件上传的数据，因为该数据通常被视为字节集合，而不是字符串。任何自动解码都会改变字节流的含义。

