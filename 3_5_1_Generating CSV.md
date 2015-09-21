{% raw %}

# 使用Django输出CSV #
这篇文档阐述了如何通过使用Django视图动态输出CSV (Comma Separated Values)。 你可以使用Python CSV 库或者Django的模板系统来达到目的。

## 使用Python CSV库 ##

Python自带了CSV库，`csv`。在Django中使用它的关键是，`csv`模块的CSV创建功能作用于类似于文件的对象，并且Django的`HttpResponse`对象就是类似于文件的对象。

这里是个例子：

```
import csv
from django.http import HttpResponse

def some_view(request):
    # Create the HttpResponse object with the appropriate CSV header.
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'

    writer = csv.writer(response)
    writer.writerow(['First row', 'Foo', 'Bar', 'Baz'])
    writer.writerow(['Second row', 'A', 'B', 'C', '"Testing"', "Here's a quote"])

    return response
```

代码和注释是不用多说的，但是一些事情需要提醒一下：

+ 响应对象获得了一个特殊的MIME类型，`text/csv`。这会告诉浏览器，文档是个CSV文件而不是HTML文件。如果你把它去掉，浏览器可能会把输出解释为HTML，会在浏览器窗口中显示一篇丑陋的、可怕的官样文章。
+ 响应对象获取了附加的`Content-Disposition`协议头，它含有CSV文件的名称。文件名可以是任意的；你想把它叫做什么都可以。浏览器会在”另存为“对话框中使用它，或者其它。
+ 钩住CSV生成API非常简单：只需要把`response`作为第一个参数传递给`csv.writer`。`csv.writer` 函数接受一个类似于文件的对象，而`HttpResponse` 对象正好合适。
+ 对于你CSV文件的每一行，调用`writer.writerow`，向它传递一个可迭代的对象比如列表或者元组。
+ CSV模板会为你处理引用，所以你不用担心没有转义字符串中的引号或者逗号。只需要向`writerow()`传递你的原始字符串，它就会执行正确的操作。

> 在Python 2中处理Unicode
>
> Python2的`csv`模块不支持Unicode输入。由于Django在内部使用Unicode，这意味着从一些来源比如`HttpRequest`读出来的字符串可能导致潜在的问题。有一些选项用于处理它：
>
> + 手动将所有Unicode对象编码为兼容的编码。
> + 使用[csv模块示例章节](https://docs.python.org/library/csv.html#examples)中提供的`UnicodeWriter`类。
> + 使用[python-unicodecsv 模块](https://github.com/jdunck/python-unicodecsv)，它作为`csv`模块随时可用的替代方案，能够优雅地处理Unicode。
>
> 更多信息请见`csv`模块的Python文档。

### 流式传输大尺寸CSV文件 ###

当处理生成大尺寸响应的视图时，你可能想要使用Django的`StreamingHttpResponse`类。例如，通过流式传输需要长时间来生成的文件，可以避免负载均衡器在服务器生成响应的时候断掉连接。

在这个例子中，我们利用Python的生成器来有效处理大尺寸CSV文件的拼接和传输：

```
import csv

from django.utils.six.moves import range
from django.http import StreamingHttpResponse

class Echo(object):
    """An object that implements just the write method of the file-like
    interface.
    """
    def write(self, value):
        """Write the value by returning it, instead of storing in a buffer."""
        return value

def some_streaming_csv_view(request):
    """A view that streams a large CSV file."""
    # Generate a sequence of rows. The range is based on the maximum number of
    # rows that can be handled by a single sheet in most spreadsheet
    # applications.
    rows = (["Row {}".format(idx), str(idx)] for idx in range(65536))
    pseudo_buffer = Echo()
    writer = csv.writer(pseudo_buffer)
    response = StreamingHttpResponse((writer.writerow(row) for row in rows),
                                     content_type="text/csv")
    response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'
    return response
```

## 使用模板系统 ##

或者，你可以使用[Django模板系统](http://python.usyiyi.cn/django/topics/templates.html)来生成CSV。比起便捷的Python `csv`模板来说，这样比较低级，但是为了完整性，这个解决方案还是在这里展示一下。

它的想法是，传递一个项目的列表给你的模板，并且让模板在`for`循环中输出逗号。

这里是一个例子，它像上面一样生成相同的CSV文件：

```
from django.http import HttpResponse
from django.template import loader, Context

def some_view(request):
    # Create the HttpResponse object with the appropriate CSV header.
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'

    # The data is hard-coded here, but you could load it from a database or
    # some other source.
    csv_data = (
        ('First row', 'Foo', 'Bar', 'Baz'),
        ('Second row', 'A', 'B', 'C', '"Testing"', "Here's a quote"),
    )

    t = loader.get_template('my_template_name.txt')
    c = Context({
        'data': csv_data,
    })
    response.write(t.render(c))
    return response
```

这个例子和上一个例子之间唯一的不同就是，这个例子使用模板来加载，而不是CSV模块。代码的结果 -- 比如`content_type='text/csv'` -- 都是相同的。

然后，创建模板`my_template_name.txt`，带有以下模板代码：

```
{% for row in data %}"{{ row.0|addslashes }}", "{{ row.1|addslashes }}", "{{ row.2|addslashes }}", "{{ row.3|addslashes }}", "{{ row.4|addslashes }}"
{% endfor %}
```

这个模板十分基础。它仅仅遍历了提供的数据，并且对于每一行都展示了一行CSV。它使用了`addslashes`模板过滤器来确保没有任何引用上的问题。

## 其它基于文本的格式 ##

要注意对于 CSV来说，这里并没有什么特别之处 -- 只是特定了输出格式。你可以使用这些技巧中的任何一个，来输出任何你想要的，基于文本的格式。你也可以使用相似的技巧来生成任意的二进制数据。例子请参见[在Django中输出PDF](http://python.usyiyi.cn/django/howto/outputting-pdf.html)。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Generating CSV](https://docs.djangoproject.com/en/1.8/howto/outputting-csv/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。

{% endraw %}
