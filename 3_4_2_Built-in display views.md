{% raw %}

<!--
  译者：wrongwat.cn
  1.8更新：Github@wizardforcel
-->

# 基于类的内建通用视图 #

编写Web应用可能是单调的，因为你需要不断的重复某一种模式。 Django尝试从model和 template层移除一些单调的情况，但是Web开发者依然会在view（视图）层经历这种厌烦。

Django的通用视图被开发用来消除这一痛苦。它们采用某些常见的习语和在开发过 程中发现的模式然后把它们抽象出来，以便你能够写更少的代码快速的实现基础的视图。

我们能够识别一些基础的任务，比如展示对象的列表，以及编写代码来展示任何对象的 列表。此外，有问题的模型可以作为一个额外的参数传递到URLconf中。

Django通过通用视图来完成下面一些功能：

+ 为单一的对象展示列表和一个详细页面。 如果我们创建一个应用来管理会议，那么   一个 TalkListView (讨论列表视图)和一个 RegisteredUserListView （   注册用户列表视图）就是列表视图的一个例子。一个单独的讨论信息页面就是我们称   之为 "详细" 视图的例子。
+ 在年/月/日归档页面，以及详细页面和“最后发表”页面中，展示以数据库为基础的对象。
允许用户创建，更新和删除对象 -- 以授权或者无需授权的方式。

总的来说，这些视图提供了一些简单的接口来完成开发者遇到的大多数的常见任务。

## 扩展通用视图 ##

使用通用视图可以极大的提高开发速度，是毫无疑问的。 然而在大多数工程中， 总会遇到通用视图无法满足需求的时候。的确，大多数来自Django开发新手 的问题是如何能使得通用视图的使用范围更广。

这是通用视图在1.3发布中被重新设计的原因之一 - 之前，它们仅仅是一些函数视图加上 一列令人疑惑的选项；现在，比起传递大量的配置到URLconf中，更推荐的扩展通用视图的 方法是子类化它们，并且重写它们的属性或者方法。

这就是说，通用视图有一些限制。如果你将你的视图实现为通用视图的子类，你就会发现这样能够更有效地编写你想要的代码，使用你自己的基于类或功能的视图。

在一些三方的应用中，有更多通用视图的示例，或者你可以自己按需编写。

## 对象的通用视图 ##

TemplateView确实很有用，但是当你需要 呈现你数据库中的内容时Django的通用视图才真的会脱颖而出。因为这是如此常见 的任务，Django提供了一大把内置的通用视图，使生成对象的展示列表和详细视图 的变得极其容易。

让我们来看一下这些通用视图中的"对象列表"视图。

我们将使用下面的模型：

```
# models.py
from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=30)
    address = models.CharField(max_length=50)
    city = models.CharField(max_length=60)
    state_province = models.CharField(max_length=30)
    country = models.CharField(max_length=50)
    website = models.URLField()

    class Meta:
        ordering = ["-name"]

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Author(models.Model):
    salutation = models.CharField(max_length=10)
    name = models.CharField(max_length=200)
    email = models.EmailField()
    headshot = models.ImageField(upload_to='author_headshots')

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField('Author')
    publisher = models.ForeignKey(Publisher)
    publication_date = models.DateField()
```

现在我们需要定义一个视图：

```
# views.py
from django.views.generic import ListView
from books.models import Publisher

class PublisherList(ListView):
    model = Publisher
```

最后将视图解析到你的url上：

```
# urls.py
from django.conf.urls import url
from books.views import PublisherList

urlpatterns = [
    url(r'^publishers/$', PublisherList.as_view()),
]
```

上面就是所有我们需要写的Python代码了。

> 注意
>
> 所以，当（例如）DjangoTemplates后端的APP_DIRS选项在TEMPLATES中设置为True时，模板的位置应该为：/path/to/project/books/templates/books/publisher_list.html。

这个模板将会依据于一个上下文(context)来渲染，这个context包含一个名为object_list 包含所有publisher对象的变量。一个非常简单的模板可能看起来像下面这样：

```
{% extends "base.html" %}

{% block content %}
    <h2>Publishers</h2>
    <ul>
        {% for publisher in object_list %}
            <li>{{ publisher.name }}</li>
        {% endfor %}
    </ul>
{% endblock %}
```

这确实就是全部代码了。 所有通用视图中有趣的特性来自于修改被传递到通用视图中的"信息" 字典。generic views reference文档详细 介绍了通用视图以及它的选项；本篇文档剩余的部分将会介绍自定义以及扩展通用 视图的常见方法。


## 编写“友好的”模板上下文 ##

你可能已经注意到了，我们在publisher列表的例子中把所有的publisher对象 放到 object_list 变量中。虽然这能正常工作，但这对模板作者并不是 "友好的"。他们只需要知道在这里要处理publishers就行了。

因此，如果你在处理一个模型(model)对象，这对你来说已经足够了。 当你处理 一个object或者queryset时，Django能够使用你定义对象显示用的自述名（verbose name，或者复数的自述名，对于对象列表）来填充上下文(context)。提供添加到默认的 object_list 实体中，但是包含完全相同的数据，例如publisher_list。

如果自述名(或者复数的自述名) 仍然不能很好的符合要求，你 可以手动的设置上下文(context)变量的名字。在一个通用视图上的context_object_name属性指定了要使用的定了上下文变量：

```
# views.py
from django.views.generic import ListView
from books.models import Publisher

class PublisherList(ListView):
    model = Publisher
    context_object_name = 'my_favorite_publishers'
```

提供一个有用的context_object_name总是个好主意。和你一起工作的设计 模板的同事会感谢你的。

## 添加额外的上下文 ##

多数时候，你只是需要展示一些额外的信息而不是提供一些通用视图。 比如，考虑到每个publisher 详细页面上的图书列表的展示。DetailView通用视图提供了一个publisher对象给context，但是我们如何在模板中添加附加信息呢？

答案是派生DetailView，并且在get_context_data方法中提供你自己的实现。默认的实现只是简单的 给模板添加了要展示的对象，但是你这可以这样覆写来展示更多信息：

```
from django.views.generic import DetailView
from books.models import Publisher, Book

class PublisherDetail(DetailView):

    model = Publisher

    def get_context_data(self, **kwargs):
        # Call the base implementation first to get a context
        context = super(PublisherDetail, self).get_context_data(**kwargs)
        # Add in a QuerySet of all the books
        context['book_list'] = Book.objects.all()
        return context
```

> 注意
>
> 通常来说，get_context_data会将当前类中的上下文数据，合并到所有超类中的上下文数据。要在你自己想要改变上下文的类中保持这一行为，你应该确保在超类中调用了get_context_data。如果没有任意两个类尝试定义相同的键，会返回异常的结果。然而，如果任何一个类尝试在超类持有一个键的情况下覆写它（在调用超类之后），这个类的任何子类都需要显式于超类之后设置它，如果你想要确保他们覆写了所有超类的话。如果你有这个麻烦，复查你视图中的方法调用顺序。

## 查看对象的子集 ##

现在让我们来近距离查看下我们一直在用的 model参数。model参数指定了视图在哪个数据库模型之上进行操作，这适用于所有的需要 操作一个单独的对象或者一个对象集合的通用视图。然而，model参数并不是唯一能够指明视图要基于哪个对象进行操作的方法 --  你同样可以使用queryset参数来指定一个对象列表：

```
from django.views.generic import DetailView
from books.models import Publisher

class PublisherDetail(DetailView):

    context_object_name = 'publisher'
    queryset = Publisher.objects.all()
```

指定model = Publisher等价于快速声明的queryset = Publisher.objects.all()。然而，通过使用queryset来定义一个过滤的对象列表，你可以更加详细 的了解哪些对象将会被显示的视图中(参见执行查询来获取更多关于查询集对象的更对信息，以及参见 基于类的视图参考来获取全部 细节)。

我们可能想要对图书列表按照出版日期进行排序来选择一个简单的例子，并且把 最近的放到前面：

```
from django.views.generic import ListView
from books.models import Book

class BookList(ListView):
    queryset = Book.objects.order_by('-publication_date')
    context_object_name = 'book_list'
```

这是个非常简单的列子，但是它很好的诠释了处理思路。 当然，你通常想做的不仅仅只是 对对象列表进行排序。如果你想要展现某个出版商的所有图书列表，你可以使用 同样的手法：

```
from django.views.generic import ListView
from books.models import Book

class AcmeBookList(ListView):

    context_object_name = 'book_list'
    queryset = Book.objects.filter(publisher__name='Acme Publishing')
    template_name = 'books/acme_list.html'
```

注意，除了经过过滤之后的查询集，一起定义的还有我们自定义的模板名称。如果我们不这么做，通过视图会使用和 "vanilla" 对象列表名称一样的模板，这可 能不是我们想要的。

另外需要注意，这并不是处理特定出版商的图书的非常优雅的方法。 如果我们  要创建另外一个出版商页面，我们需要添加另外几行代码到URLconf中，并且再多几个 出版商就会觉得这么做不合理。我们会在下一个章节处理这个问题。

> 注意
>
> 如果你在访问 /books/acme/时出现404错误，检查确保你确实有一个名字为“ACME Publishing”的出版商。通用视图在这种情况下拥有一个allow_empty 的参数。详见基于类的视图参考。

## 动态过滤 ##

另一个普遍的需求是在给定的列表页面中根据URL中的关键字来过滤对象。 前面我们把出版 商的名字硬编码到URLconf中，但是如果我们想要编写一个视图来展示任何publisher的所有 图书，应该如何处理？

相当方便的是， ListView 有一个get_queryset() 方法来供我们重写。在之前，它只是返回一个queryset属性值，但是现在我们可以添加更多的逻辑。

让这种方式能够工作的关键点，在于当类视图被调用时，各种有用的对象被存储在self上；同request()(self.request)一样，其中包含了从URLconf中获取到的位置参数 (self.args)和基于名字的参数(self.kwargs)(关键字参数)。

这里，我们拥有一个带有一组供捕获的参数的URLconf：

```
# urls.py
from django.conf.urls import url
from books.views import PublisherBookList

urlpatterns = [
    url(r'^books/([\w-]+)/$', PublisherBookList.as_view()),
]
```

接着，我们编写了PublisherBookList视图::

```
# views.py
from django.shortcuts import get_object_or_404
from django.views.generic import ListView
from books.models import Book, Publisher

class PublisherBookList(ListView):

    template_name = 'books/books_by_publisher.html'

    def get_queryset(self):
        self.publisher = get_object_or_404(Publisher, name=self.args[0])
        return Book.objects.filter(publisher=self.publisher)
```

如你所见，在queryset区域添加更多的逻辑非常容易；如果我们想的话，我们可以 使用self.request.user来过滤当前用户，或者添加其他更复杂的逻辑。

同时我们可以把出版商添加到上下文中，这样我们就可以在模板中使用它：

```
# ...

def get_context_data(self, **kwargs):
    # Call the base implementation first to get a context
    context = super(PublisherBookList, self).get_context_data(**kwargs)
    # Add in the publisher
    context['publisher'] = self.publisher
    return context
```

## 执行额外的工作 ##

我们需要考虑的最后的共同模式在调用通用视图之前或者之后会引起额外的开销。

想象一下，在我们的Author对象上有一个last_accessed字段，这个字段用来 跟踪某人最后一次查看了这个作者的时间。

```
# models.py
from django.db import models

class Author(models.Model):
    salutation = models.CharField(max_length=10)
    name = models.CharField(max_length=200)
    email = models.EmailField()
    headshot = models.ImageField(upload_to='author_headshots')
    last_accessed = models.DateTimeField()
```

通用的DetailView类，当然不知道关于这个字段的事情，但我们可以很容易 再次编写一个自定义的视图，来保持这个字段的更新。

首先，我们需要添加作者详情页的代码配置到URLconf中，指向自定义的视图：

```
from django.conf.urls import url
from books.views import AuthorDetailView

urlpatterns = [
    #...
    url(r'^authors/(?P<pk>[0-9]+)/$', AuthorDetailView.as_view(), name='author-detail'),
]
```

然后，编写我们新的视图 -- get_object是用来获取对象的方法 -- 因此我们简单的 重写它并封装调用：

```
from django.views.generic import DetailView
from django.utils import timezone
from books.models import Author

class AuthorDetailView(DetailView):

    queryset = Author.objects.all()

    def get_object(self):
        # Call the superclass
        object = super(AuthorDetailView, self).get_object()
        # Record the last accessed date
        object.last_accessed = timezone.now()
        object.save()
        # Return the object
        return object
```

> 注意
>
> 这里URLconf使用参数组的名字pk - 这个名字是DetailView用来查找主键的值的默认名称，其中主键用于过滤查询集。
>
> 如果你想要调用参数组的其它方法，你可以在视图上设置pk_url_kwarg。详见 DetailView参考。

{% endraw %}
