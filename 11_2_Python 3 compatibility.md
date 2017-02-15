

# Porting to Python 3

第一个支持Python 3的版本是Django 1.5。感谢[six](http://pythonhosted.org/six/)的兼容层，无需对代码做出任何改动，就可以让你的代码同时在Python 2 (≥ 2.6.5)和Python 3 (≥ 3.2)上运行。

本文档主要针对希望支持Python 2和3的可插拔应用程序的作者。它还描述了适用于Django代码的指南。

## Philosophy

本文档假定您熟悉Python 2和Python 3之间的更改。如果你不是，请先阅读[Python的官方移植指南](https://docs.python.org/3/howto/pyporting.html)。刷新你对Python 2和3的unicode处理的知识将有所帮助； [Pragmatic Unicode](http://nedbatchelder.com/text/unipain.html)演示文稿是一个很好的资源。

Django使用_Python 2/3兼容源_策略。当然，你可以为自己的代码自由选择另一个策略，特别是如果你不需要保持与Python 2兼容。但是可插拔应用程序的作者鼓励使用与Django本身相同的移植策略。

如果您定位Python≥2.6，编写兼容的代码要容易得多。Django 1.5引入了诸如[`django.utils.six`](#module-django.utils.six "django.utils.six")之类的兼容性工具，它是[`six module`](http://pythonhosted.org/six/index.html#module-six "(in six v1.9)")的定制版本， 。为了方便起见，在Django 1.4.2中引入了向前兼容的别名。如果您的应用程序利用这些工具，它将需要Django≥1.4.2。

显然，编写兼容的源代码会增加一些开销，这会导致沮丧。Django的开发人员发现，尝试编写与Python 2兼容的Python 3代码比相反的更有意义。这不仅使你的代码更加面向未来，但Python 3的优势（如saner字符串处理）开始迅速发光。处理Python 2成为向后兼容性的要求，我们作为开发人员来处理这样的约束。

Django提供的移植工具受这种理念的启发，并且贯穿本指南。

## Porting tips

### Unicode literals

此步骤包括：

*   在Python模块顶部从 t&gt; __未来__ 导入 unicode_literals添加`把它放在每个模块中，否则你会继续检查文件的顶部，看看哪个模式是有效的；`
*   在unicode字符串之前删除`u`前缀；
*   在bytestrings之前添加`b`前缀。

执行这些更改系统地保证向后兼容性。

然而，Django应用程序通常不需要bytestrings，因为Django只将unicode接口暴露给程序员。Python 3不鼓励使用bytestrings，二进制数据或面向字节的接口除外。Python 2使得bytestrings和unicode字符串可以有效地互换，只要它们只包含ASCII数据。利用这一点，尽可能使用unicode字符串，并避免`b`前缀。

注意

Python 2的`u`前缀是Python 3.2中的一个语法错误，但是由于 [**PEP 414**](http://www.python.org/dev/peps/pep-0414)，它将再次允许在Python 3.3中。因此，如果您定位Python≥3.3，则此转换是可选的。它仍然推荐，按照“编写Python 3代码”的哲学。

### String handling

Python 2’s [unicode](https://docs.python.org/2/library/functions.html#unicode) type was renamed [`str`](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.4)") in Python 3, `str()` was renamed [`bytes`](https://docs.python.org/3/library/functions.html#bytes "(in Python v3.4)"), and [basestring](https://docs.python.org/2/library/functions.html#basestring) disappeared. [六](http://pythonhosted.org/six/)提供[_tools_](#string-handling-with-six)来处理这些更改。

Django在[`django.utils.encoding`](../ref/utils.html#module-django.utils.encoding "django.utils.encoding: A series of helper functions to manage character encoding.")和[`django.utils.safestring`](../ref/utils.html#module-django.utils.safestring "django.utils.safestring: Functions and classes for working with strings that can be displayed safely without further escaping in HTML.")模块中还包含多个与字符串相关的类和函数。他们的名字使用了`str`这个词，这在Python 2和Python 3以及`unicode`中并不是相同的，它在Python 3中不存在。为了避免歧义和混淆，这些概念重命名为`bytes`和`text`。

以下是[`django.utils.encoding`](../ref/utils.html#module-django.utils.encoding "django.utils.encoding: A series of helper functions to manage character encoding.")中的名称更改：

| 旧名称 | 新名称 |
| --- | --- |
| `smart_str` | `smart_bytes` |
| `smart_unicode` | `smart_text` |
| `force_unicode` | `force_text` |

为了向后兼容，旧名称仍然适用于Python 2。在Python 3下，`smart_str`是`smart_text`的别名。

为了向前兼容性，新名称的工作原理与Django 1.4.2相同。

注意

[`django.utils.encoding`](../ref/utils.html#module-django.utils.encoding "django.utils.encoding: A series of helper functions to manage character encoding.")在Django 1.5中被重构，以提供更一致的API。检查其文档以获取更多信息。

[`django.utils.safestring`](../ref/utils.html#module-django.utils.safestring "django.utils.safestring: Functions and classes for working with strings that can be displayed safely without further escaping in HTML.")主要通过[`mark_safe()`](../ref/utils.html#django.utils.safestring.mark_safe "django.utils.safestring.mark_safe")和[`mark_for_escaping()`](../ref/utils.html#django.utils.safestring.mark_for_escaping "django.utils.safestring.mark_for_escaping")函数使用，但没有更改。如果你使用的内部，这里是名称更改：

| 旧名称 | 新名称 |
| --- | --- |
| `EscapeString` | `EscapeBytes` |
| `EscapeUnicode` | `EscapeText` |
| `SafeString` | `SafeBytes` |
| `SafeUnicode` | `SafeText` |

为了向后兼容，旧名称仍然适用于Python 2。在Python 3下，`EscapeString`和`SafeString`分别是`EscapeText`和`SafeText`的别名。

为了向前兼容性，新名称的工作原理与Django 1.4.2相同。

### [`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)") and [__unicode__()](https://docs.python.org/2/reference/datamodel.html#object.__unicode__) methods

在Python 2中，对象模型指定[`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)")和[__unicode __()](https://docs.python.org/2/reference/datamodel.html#object.__unicode__)方法。如果这些方法存在，它们必须分别返回`str`（bytes）和`unicode`（text）。

`print`语句和[`str`](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.4)")内置调用[`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)")来确定对象的人类可读表示。`unicode`内置调用[__unicode __()](https://docs.python.org/2/reference/datamodel.html#object.__unicode__)如果存在，否则返回[`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)")，并使用系统编码。相反，[`Model`](../ref/models/instances.html#django.db.models.Model "django.db.models.Model")基类通过编码为UTF-8自动从[__unicode __()](https://docs.python.org/2/reference/datamodel.html#object.__unicode__)中导出[`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)")

在Python 3中，只有[`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)")，它必须返回`str`（text）。

（也可以定义[`__bytes__()`](https://docs.python.org/3/reference/datamodel.html#object.__bytes__ "(in Python v3.4)")，但Django应用程序对该方法几乎没有用处，因为它们几乎不处理`bytes`。）

Django provides a simple way to define [`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)") and [__unicode__()](https://docs.python.org/2/reference/datamodel.html#object.__unicode__) methods that work on Python 2 and 3: you must define a [`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)") method returning text and to apply the [`python_2_unicode_compatible()`](../ref/utils.html#django.utils.encoding.python_2_unicode_compatible "django.utils.encoding.python_2_unicode_compatible") decorator.

在Python 3上，装饰器是一个无操作。在Python 2中，它定义了适当的[__unicode __()](https://docs.python.org/2/reference/datamodel.html#object.__unicode__)和[`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)")方法（替换过程中的原始[`__str__()`](https://docs.python.org/3/reference/datamodel.html#object.__str__ "(in Python v3.4)")方法）。这里有一个例子：

```
from __future__ import unicode_literals
from django.utils.encoding import python_2_unicode_compatible

@python_2_unicode_compatible
class MyClass(object):
    def __str__(self):
        return "Instance of my class"

```

这种技术是Django移植理念的最佳匹配。

对于向前兼容性，此装饰器可用于Django 1.4.2。

最后，请注意，[`__repr__()`](https://docs.python.org/3/reference/datamodel.html#object.__repr__ "(in Python v3.4)")必须在所有版本的Python上返回`str`。

### [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.4)") and [`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.4)")-like classes

[`dict.keys()`](https://docs.python.org/3/library/stdtypes.html#dict.keys "(in Python v3.4)")，[`dict.items()`](https://docs.python.org/3/library/stdtypes.html#dict.items "(in Python v3.4)")和[`dict.values()`](https://docs.python.org/3/library/stdtypes.html#dict.values "(in Python v3.4)")返回列表在Python 2和迭代器在Python 3。[`QueryDict`](../ref/request-response.html#django.http.QueryDict "django.http.QueryDict")和在[`django.utils.datastructures`](../ref/utils.html#module-django.utils.datastructures "django.utils.datastructures: Data structures that aren't in Python's standard library.")中定义的[`dict`](https://docs.python.org/3/library/stdtypes.html#dict "(in Python v3.4)")

[six](http://pythonhosted.org/six/) provides compatibility functions to work around this change: [`iterkeys()`](http://pythonhosted.org/six/index.html#six.iterkeys "(in six v1.9)"), [`iteritems()`](http://pythonhosted.org/six/index.html#six.iteritems "(in six v1.9)"), and [`itervalues()`](http://pythonhosted.org/six/index.html#six.itervalues "(in six v1.9)"). 它还包含未正式记录的`iterlists`函数，适用于`django.utils.datastructures.MultiValueDict`及其子类。

### [`HttpRequest`](../ref/request-response.html#django.http.HttpRequest "django.http.HttpRequest") and [`HttpResponse`](../ref/request-response.html#django.http.HttpResponse "django.http.HttpResponse") objects

根据 [**PEP 3333**](http://www.python.org/dev/peps/pep-3333)：

*   头总是`str`对象，
*   输入和输出流始终为`bytes`对象。

具体来说，[`HttpResponse.content`](../ref/request-response.html#django.http.HttpResponse.content "django.http.HttpResponse.content")包含`bytes`，如果您在测试中将其与`str`进行比较，则可能会出现问题。首选解决方案是依靠[`assertContains()`](testing/tools.html#django.test.SimpleTestCase.assertContains "django.test.SimpleTestCase.assertContains")和[`assertNotContains()`](testing/tools.html#django.test.SimpleTestCase.assertNotContains "django.test.SimpleTestCase.assertNotContains")。这些方法接受响应和unicode字符串作为参数。

## Coding guidelines

以下准则在Django的源代码中强制实施。它们也推荐用于遵循相同移植策略的第三方应用程序。

### Syntax requirements

#### Unicode

在Python 3中，默认情况下所有字符串都被视为Unicode。来自Python 2的`unicode`类型在Python 3中称为`str`，`str`变为`bytes`。

您不能在unicode字符串字面量之前使用`u`前缀，因为它在Python 3.2中是语法错误。必须使用`b`前缀字节字符串。

为了在Python 2中启用相同的行为，每个模块必须从`__future__`导入`unicode_literals`：

```
from __future__ import unicode_literals

my_string = "This is an unicode literal"
my_bytestring = b"This is a bytestring"

```

如果您需要Python 2下的字节字符串字符串和Python 3下的Unicode字符串字符串，请使用[`str`](https://docs.python.org/3/library/stdtypes.html#str "(in Python v3.4)") builtin：

```
str('my string')

```

在Python 3中，`str`和`bytes`之间没有任何自动转换，而且[`codecs`](https://docs.python.org/3/library/codecs.html#module-codecs "(in Python v3.4)")模块变得更加严格。[`str.encode()`](https://docs.python.org/3/library/stdtypes.html#str.encode "(in Python v3.4)")始终返回`bytes`，`bytes.decode`始终返回`str`。因此，有时需要以下模式：

```
value = value.encode('ascii', 'ignore').decode('ascii')

```

如果您必须[索引bytestrings](https://docs.python.org/3/howto/pyporting.html#indexing-bytes-objects)，请谨慎。

#### Exceptions

捕获异常时，使用`as`关键字：

```
try:
    ...
except MyException as exc:
    ...

```

这个旧的语法在Python 3中被删除：

```
try:
    ...
except MyException, exc:    # Don't do that!
    ...

```

使用不同跟踪重新处理异常的语法也发生了更改。使用[`six.reraise()`](http://pythonhosted.org/six/index.html#six.reraise "(in six v1.9)")。

### Magic methods

使用下面的模式来处理在Python 3中重命名的魔法方法。

#### Iterators

```
class MyIterator(six.Iterator):
    def __iter__(self):
        return self             # implement some logic here

    def __next__(self):
        raise StopIteration     # implement some logic here

```

#### Boolean evaluation

```
class MyBoolean(object):

    def __bool__(self):
        return True             # implement some logic here

    def __nonzero__(self):      # Python 2 compatibility
        return type(self).__bool__(self)

```

#### Division

```
class MyDivisible(object):

    def __truediv__(self, other):
        return self / other     # implement some logic here

    def __div__(self, other):   # Python 2 compatibility
        return type(self).__truediv__(self, other)

    def __itruediv__(self, other):
        return self // other    # implement some logic here

    def __idiv__(self, other):  # Python 2 compatibility
        return type(self).__itruediv__(self, other)

```

在类上而不是在实例上查找特殊的方法来反映Python解释器的行为。

### Writing compatible code with six

[six](http://pythonhosted.org/six/)是在单个代码库中支持Python 2和3的规范兼容性库。阅读它的文档！

A [`customized version of six`](#module-django.utils.six "django.utils.six") is bundled with Django as of version 1.4.2\. 您可以将其导入为`django.utils.six`。

以下是编写兼容代码所需的最常见更改。

#### String handling

在Python 3中删除了`basestring`和`unicode`类型，并改变了`str`的含义。要测试这些类型，请使用以下习语：

```
isinstance(myvalue, six.string_types)       # replacement for basestring
isinstance(myvalue, six.text_type)          # replacement for unicode
isinstance(myvalue, bytes)                  # replacement for str

```

Python≥2.6提供`bytes`作为`str`的别名，因此您不需要[`six.binary_type`](http://pythonhosted.org/six/index.html#six.binary_type "(in six v1.9)")。

#### `long`

`long`类型在Python 3中不再存在。`1L`是语法错误。使用[`six.integer_types`](http://pythonhosted.org/six/index.html#six.integer_types "(in six v1.9)")检查值是整数还是长整型：

```
isinstance(myvalue, six.integer_types)      # replacement for (int, long)

```

#### `xrange`

如果在Python 2上使用`xrange`，请导入`six.moves.range`并使用它。您还可以导入`six.moves.xrange`（它等同于`six.moves.range`），但第一种方法允许您在删除对Python 2的支持时。

#### Moved modules

一些模块在Python 3中重命名。`django.utils.six.moves`模块（基于[`six.moves module`](http://pythonhosted.org/six/index.html#module-six.moves "(in six v1.9)")）提供了兼容的位置以导入它们。

#### PY2

如果您需要在Python 2和Python 3中使用不同的代码，请检查[`six.PY2`](http://pythonhosted.org/six/index.html#six.PY2 "(in six v1.9)")：

```
if six.PY2:
    # compatibility code for Python 2

```

这是最后的解决方案，当[`six`](http://pythonhosted.org/six/index.html#module-six "(in six v1.9)")不提供适当的功能。

### Django customized version of six

与Django捆绑在一起的六个版本（`django.utils.six`）包含一些仅供内部使用的自定义项。

