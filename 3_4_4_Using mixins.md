

# Mixin



注意

这是一个进阶的话题。需要建立在了解 [_基于类的视图_](index.html)的基础上。



Django的基于类的视图提供了许多功能，但是你可能只想使用其中的一部分。例如，你想编写一个视图，它渲染模板来响应HTTP，但是你用不了[`TemplateView`](../../ref/class-based-views/base.html#django.views.generic.base.TemplateView "django.views.generic.base.TemplateView")；或者你只需要对`POST` 请求渲染一个模板，而`GET` 请求做一些其它的事情。 虽然你可以直接使用[`TemplateResponse`](../../ref/template-response.html#django.template.response.TemplateResponse "django.template.response.TemplateResponse")，但是这将导致重复的代码。

由于这些原因，Django 提供许多Mixin，它们提供更细致的功能。例如，渲染模板封装在[`TemplateResponseMixin`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin "django.views.generic.base.TemplateResponseMixin") 中。Django 参考手册包含[_所有Mixin 的完整文档_](../../ref/class-based-views/mixins.html)。



## Context 和TemplateResponse

在基于类的视图中使用模板具有一致的接口，有两个Mixin 起了核心的作用。



[`TemplateResponseMixin`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin "django.views.generic.base.TemplateResponseMixin")



返回[`TemplateResponse`](../../ref/template-response.html#django.template.response.TemplateResponse "django.template.response.TemplateResponse") 的每个视图都将调用[`render_to_response()`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin.render_to_response "django.views.generic.base.TemplateResponseMixin.render_to_response") 方法，这个方法由 `TemplateResponseMixin` 提供。大部分时候，这个方法会隐式调用（例如，它会被[`TemplateView`](../../ref/class-based-views/base.html#django.views.generic.base.TemplateView "django.views.generic.base.TemplateView")和[`DetailView`](../../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView") 中实现的`get()` 方法调用）；如果你不想通过Django 的模板渲染响应，那么你可以覆盖它，虽然通常不需要这样。其示例用法请参见[_JSONResponseMixin 示例_](#jsonresponsemixin-example)。

`render_to_response()` 本身调用[`get_template_names()`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin.get_template_names "django.views.generic.base.TemplateResponseMixin.get_template_names")，它默认查找类视图的[`template_name`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin.template_name "django.views.generic.base.TemplateResponseMixin.template_name")； 其它两个Mixin（[`SingleObjectTemplateResponseMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectTemplateResponseMixin "django.views.generic.detail.SingleObjectTemplateResponseMixin") 和 [`MultipleObjectTemplateResponseMixin`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectTemplateResponseMixin "django.views.generic.list.MultipleObjectTemplateResponseMixin")）覆盖了这个方法，以在处理实际的对象时能提供更灵活的默认行为。



[`ContextMixin`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.ContextMixin "django.views.generic.base.ContextMixin")

需要Context 数据的每个内建视图，例如渲染模板的视图（包括`TemplateResponseMixin` ），都应该以关键字参数调用[`get_context_data()`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.ContextMixin.get_context_data "django.views.generic.base.ContextMixin.get_context_data")，以确保它们想要的数据在里面。`get_context_data()` 返回一个字典；在`ContextMixin` 中，它只是简单地返回它的关键字参数，通常会覆盖这个方法来向字典中添加更多的成员。







## 构建Django 的基于类的通用视图函数

让我们看下Django 的两个通用的基于类的视图是如何通过互不相关的Mixin 构建的。我们将考虑[`DetailView`](../../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView")，它渲染一个对象的“详细”视图，和[`ListView`](../../ref/class-based-views/generic-display.html#django.views.generic.list.ListView "django.views.generic.list.ListView")，它渲染一个对象列表，通常来自一个查询集，需要时还会分页。这将会向我们接收四个Mixin，这些Mixin 在用到单个或多个Django对象时非常有用。

在通用的编辑视图（[`FormView`](../../ref/class-based-views/generic-editing.html#django.views.generic.edit.FormView "django.views.generic.edit.FormView") 和模型相关的视图[`CreateView`](../../ref/class-based-views/generic-editing.html#django.views.generic.edit.CreateView "django.views.generic.edit.CreateView")、[`UpdateView`](../../ref/class-based-views/generic-editing.html#django.views.generic.edit.UpdateView "django.views.generic.edit.UpdateView") 和[`DeleteView`](../../ref/class-based-views/generic-editing.html#django.views.generic.edit.DeleteView "django.views.generic.edit.DeleteView")）和基于日期的通用视图中都会涉及到Minxin。它们在[_Mixin 参考文档_](../../ref/class-based-views/mixins.html)中讲述。



### DetailView：用于单个Django 对象

为了显示一个对象的详细信息，我们通常需要做两件事情：查询对象然后利用合适的模板和包含该对象的Context 生成[`TemplateResponse`](../../ref/template-response.html#django.template.response.TemplateResponse "django.template.response.TemplateResponse")。

为了获得对象，[`DetailView`](../../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView") 依赖[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin")，它提供一个[`get_object()`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin.get_object "django.views.generic.detail.SingleObjectMixin.get_object") 方法，这个方法基于请求的URL 获取对象（它查找URLconf 中声明的`pk` 和`slug` 关键字参数，然后从视图的[`model`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin.model "django.views.generic.detail.SingleObjectMixin.model") 属性或[`queryset`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin.queryset "django.views.generic.detail.SingleObjectMixin.queryset") 属性查询对象）。`SingleObjectMixin` 还覆盖[`get_context_data()`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.ContextMixin.get_context_data "django.views.generic.base.ContextMixin.get_context_data")，这个方法在Django 所有的内建的基于类的视图中都有用到，用来给模板的渲染提供Context 数据。

然后，为了生成[`TemplateResponse`](../../ref/template-response.html#django.template.response.TemplateResponse "django.template.response.TemplateResponse")，[`DetailView`](../../ref/class-based-views/flattened-index.html#DetailView "DetailView") 使用[`SingleObjectTemplateResponseMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectTemplateResponseMixin "django.views.generic.detail.SingleObjectTemplateResponseMixin")，它扩展自[`TemplateResponseMixin`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin "django.views.generic.base.TemplateResponseMixin")并覆盖上文讨论过的[`get_template_names()`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin.get_template_names "django.views.generic.base.TemplateResponseMixin.get_template_names")。实际上，它提供比较复杂的选项集合，但是大部分人用到的主要的一个是 `<app_label>/<model_name>_detail.html`。`_detail` 部分可以通过设置子类的[`template_name_suffix`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectTemplateResponseMixin.template_name_suffix "django.views.generic.detail.SingleObjectTemplateResponseMixin.template_name_suffix") 来改变。（例如，[_通用的编辑视图_](generic-editing.html) 使用`_form` 来创建和更新视图，用`_confirm_delete` 来删除视图）。





### ListView：用于多个Django 对象

显示对象的列表和上面的步骤大体相同：我们需要一个对象的列表（可能是分页形式的），这通常是一个[`QuerySet`](../../ref/models/querysets.html#django.db.models.query.QuerySet "django.db.models.query.QuerySet")，然后我们需要利用合适的模板和对象列表生成一个[`TemplateResponse`](../../ref/template-response.html#django.template.response.TemplateResponse "django.template.response.TemplateResponse")。

为了获取对象，[`ListView`](../../ref/class-based-views/generic-display.html#django.views.generic.list.ListView "django.views.generic.list.ListView") 使用[`MultipleObjectMixin`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectMixin "django.views.generic.list.MultipleObjectMixin")，它提供[`get_queryset()`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectMixin.get_queryset "django.views.generic.list.MultipleObjectMixin.get_queryset") 和[`paginate_queryset()`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectMixin.paginate_queryset "django.views.generic.list.MultipleObjectMixin.paginate_queryset") 两种方法。与[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin") 不同，不需要根据URL 中关键字参数来获得查询集，默认将使用视图类的[`queryset`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectMixin.queryset "django.views.generic.list.MultipleObjectMixin.queryset") 或[`model`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectMixin.model "django.views.generic.list.MultipleObjectMixin.model") 属性。通常需要覆盖[`get_queryset()`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectMixin.get_queryset "django.views.generic.list.MultipleObjectMixin.get_queryset") 以动态获取不同的对象，例如根据当前的用户或排除打算在将来提交的博客。

[`MultipleObjectMixin`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectMixin "django.views.generic.list.MultipleObjectMixin") 还覆盖[`get_context_data()`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.ContextMixin.get_context_data "django.views.generic.base.ContextMixin.get_context_data") 以包含合适的Context 变量用于分页（如果禁止分页，则提供一些假的）。这个方法依赖传递给它的关键字参数`object_list`，[`ListView`](../../ref/class-based-views/flattened-index.html#ListView "ListView") 会负责准备好这个参数。

为了生成[`TemplateResponse`](../../ref/template-response.html#django.template.response.TemplateResponse "django.template.response.TemplateResponse")，[`ListView`](../../ref/class-based-views/flattened-index.html#ListView "ListView") 然后使用[`MultipleObjectTemplateResponseMixin`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectTemplateResponseMixin "django.views.generic.list.MultipleObjectTemplateResponseMixin")；与上面的[`SingleObjectTemplateResponseMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectTemplateResponseMixin "django.views.generic.detail.SingleObjectTemplateResponseMixin") 类似，它覆盖`get_template_names()` 来提供[`一系列的选项`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectTemplateResponseMixin "django.views.generic.list.MultipleObjectTemplateResponseMixin")，而最常用到的是`<app_label>/<model_name>_list.html`，其中`_list` 部分同样由[`template_name_suffix`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectTemplateResponseMixin.template_name_suffix "django.views.generic.list.MultipleObjectTemplateResponseMixin.template_name_suffix") 属性设置。（基于日期的通用视图使用`_archive`、`_archive_year` 等等这样的后缀来针对各种基于日期的列表视图使用不同的模板）。







## 使用Django 的基于类的视图的Mixin

既然我们已经看到Django 通用的基于类的视图时如何使用Mixin，让我们在看看其它组合它们的方式。当然，我们仍将它们与内建的基于类的视图或其它通用的基于类的视图组合，但是对于Django 提供的便利性你将解决一些更加罕见的问题。



警告

不是所有的Mixin 都可以一起使用，也不是所有的基于类的视图都可以与其它Mixin 一起使用。这里我们展示的是可以工作的几个例子；如果你需要其它功能，那么你必须考虑不同类之间的属性和方法的相互作用，以及[方法解析顺序](https://www.python.org/download/releases/2.3/mro/)将如何影响方法调用的版本和顺序。

Django 的[_基于类的视图_](../../ref/class-based-views/index.html) 和[_基于类的视图的Mixin_](../../ref/class-based-views/mixins.html) 的文档将帮助你理解在不同的类和Mixin 之间那些属性和方法可能引起冲突。

如果有担心，通常最好退避并基于[`View`](../../ref/class-based-views/flattened-index.html#View "View") 或[`TemplateView`](../../ref/class-based-views/flattened-index.html#TemplateView "TemplateView")，或者可能的话加上[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin") 和 [`MultipleObjectMixin`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectMixin "django.views.generic.list.MultipleObjectMixin")。虽然你可能最终会编写更多的代码，但是对于后来的人更容易理解，而且你自己也少了份担心。（当然，你始终可以深入探究Django 中基于类的通用视图的具体实现以获取如何处理出现的问题的灵感）。





### SingleObjectMixin 与View 一起使用

如果你想编写一个简单的基于类的视图，它只响应`POST`，我们将子类化[`View`](../../ref/class-based-views/base.html#django.views.generic.base.View "django.views.generic.base.View") 并在子类中只编写一个`post()` 方法。但是，如果我们想处理一个由URL 标识的特定对象，我们将需要[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin") 提供的功能。

我们将使用在[_通用的基于类的视图简介_](generic-display.html) 中用到的`Author` 模型做演示。



views.py



```
from django.http import HttpResponseForbidden, HttpResponseRedirect
from django.core.urlresolvers import reverse
from django.views.generic import View
from django.views.generic.detail import SingleObjectMixin
from books.models import Author

class RecordInterest(SingleObjectMixin, View):
    """Records the current user's interest in an author."""
    model = Author

    def post(self, request, *args, **kwargs):
        if not request.user.is_authenticated():
            return HttpResponseForbidden()

        # Look up the author we're interested in.
        self.object = self.get_object()
        # Actually record interest somehow here!

        return HttpResponseRedirect(reverse('author-detail', kwargs={'pk': self.object.pk}))

```





实际应用中，你的对象可能以键-值的方式保存而不是保存在关系数据库中，所以我们不考虑这点。使用[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin") 的视图唯一需要担心的是在哪里查询我们感兴趣的Author，而它会用一个简单的`self.get_object()` 调用实现。其它的所有事情都有该Mixin 帮我们处理。

我们可以将它这样放入URL 中，非常简单：



urls.py



```
from django.conf.urls import url
from books.views import RecordInterest

urlpatterns = [
    #...
    url(r'^author/(?P<pk>[0-9]+)/interest//span>, RecordInterest.as_view(), name='author-interest'),
]

```





注意`pk` 命名组，[`get_object()`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin.get_object "django.views.generic.detail.SingleObjectMixin.get_object") 将用它来查询`Author` 实例。你还可以使用slug，或者[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin") 的其它功能。





### SingleObjectMixin 与ListView 一起使用

[`ListView`](../../ref/class-based-views/generic-display.html#django.views.generic.list.ListView "django.views.generic.list.ListView") 提供内建的分页，但是可能你分页的列表中每个对象都与另外一个对象（通过一个外键）关联。在我们的Publishing 例子中，你可能想通过一个特定的Publisher 分页所有的Book。

一种方法是组合[`ListView`](../../ref/class-based-views/flattened-index.html#ListView "ListView") 和[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin")，这样分页的Book 列表的查询集能够与找到的单个Publisher 对象关联。为了实现这点，我们需要两个不同的查询集：



`Book` queryset for use by [`ListView`](../../ref/class-based-views/generic-display.html#django.views.generic.list.ListView "django.views.generic.list.ListView")

Since we have access to the `Publisher` whose books we want to list, we simply override `get_queryset()` and use the `Publisher`’s [_reverse foreign key manager_](../db/queries.html#backwards-related-objects).

`Publisher` queryset for use in [`get_object()`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin.get_object "django.views.generic.detail.SingleObjectMixin.get_object")

We’ll rely on the default implementation of `get_object()` to fetch the correct `Publisher` object. However, we need to explicitly pass a `queryset` argument because otherwise the default implementation of `get_object()` would call `get_queryset()` which we have overridden to return `Book` objects instead of `Publisher` ones.





注

我们必须仔细考虑`get_context_data()`。因为[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin") 和[`ListView`](../../ref/class-based-views/flattened-index.html#ListView "ListView") 都会将Context 数据的`context_object_name` 下，我们必须显式确保`Publisher` 位于Context 数据中。[`ListView`](../../ref/class-based-views/flattened-index.html#ListView "ListView") 将为我们添加合适的`page_obj` 和 `paginator` ，只要我们记住调用`super()`。



现在，我们可以编写一个新的`PublisherDetail`：





```
from django.views.generic import ListView
from django.views.generic.detail import SingleObjectMixin
from books.models import Publisher

class PublisherDetail(SingleObjectMixin, ListView):
    paginate_by = 2
    template_name = "books/publisher_detail.html"

    def get(self, request, *args, **kwargs):
        self.object = self.get_object(queryset=Publisher.objects.all())
        return super(PublisherDetail, self).get(request, *args, **kwargs)

    def get_context_data(self, **kwargs):
        context = super(PublisherDetail, self).get_context_data(**kwargs)
        context['publisher'] = self.object
        return context

    def get_queryset(self):
        return self.object.book_set.all()

```





注意我们 在 `get()`方法里设置了`self.object` ，这样我们就可以在后面的 `get_context_data()` 和`get_queryset()`方法里再次用到它. 如果不设置 `template_name`, 那模板会指向默认的 [`ListView`](../../ref/class-based-views/flattened-index.html#ListView "ListView") 所选择的模板, 也就是 `"books/book_list.html"`，因为这个模板是书目的一个列表; 但[`ListView`](../../ref/class-based-views/flattened-index.html#ListView "ListView") 对于该类继承了 [`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin")这个类是一无所知的,所以不会对使用`Publisher`来查看视图有任何反应.

`paginate_by`是每页显示几条数据的意思，这里设的比较小，是因为这样你就不用造一堆数据才能看到分页的效果了！下面是你想要的模板:





```
{% extends "base.html" %}

{% block content %}
    <h2>Publisher {{ publisher.name }}</h2>

    <ol>
      {% for book in page_obj %}
        <li>{{ book.title }}</li>
      {% endfor %}
    </ol>

     class="pagination">
         class="step-links">
            {% if page_obj.has_previous %}
                <a href="?page={{ page_obj.previous_page_number }}">previous</a>
            {% endif %}

             class="current">
                Page {{ page_obj.number }} of {{ paginator.num_pages }}.
            

            {% if page_obj.has_next %}
                <a href="?page={{ page_obj.next_page_number }}">next</a>
            {% endif %}
        
    
{% endblock %}

```











## 避免让事情复杂化

通常情况下你只在需要相关功能时才会使用 [`TemplateResponseMixin`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin "django.views.generic.base.TemplateResponseMixin")和[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin")这两个类。如上所示，只要加点儿小心，你甚至可以把`SingleObjectMixin`和[`ListView`](../../ref/class-based-views/generic-display.html#django.views.generic.list.ListView "django.views.generic.list.ListView")结合在一起来使用. 但是这么搞可能会让事情变得有点复杂，作为一个好的原则：



提示：

你的视图扩展应该仅仅使用那些来自于同一组通用基类的view或者mixins。如: [_detail, list_](generic-display.html), [_editing_](generic-editing.html) 和 date. 例如：把 [`TemplateView`](../../ref/class-based-views/flattened-index.html#TemplateView "TemplateView") (内建视图)和 [`MultipleObjectMixin`](../../ref/class-based-views/mixins-multiple-object.html#django.views.generic.list.MultipleObjectMixin "django.views.generic.list.MultipleObjectMixin") (通用列表)整合在一起是极好的, 但是若想把`SingleObjectMixin` (generic detail) 和 `MultipleObjectMixin` (generic list)整合在一起就有麻烦啦！



To show what happens when you try to get more sophisticated, we show an example that sacrifices readability and maintainability when there is a simpler solution. 首先，我们来看一下如何把[`DetailView`](../../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView") 和 [`FormMixin`](../../ref/class-based-views/mixins-editing.html#django.views.generic.edit.FormMixin "django.views.generic.edit.FormMixin")结合起来，实现 `POST` 一个 Django [`表单`](../../ref/forms/api.html#django.forms.Form "django.forms.Form") 到相同URL，这样我们就可以用[`DetailView`](../../ref/class-based-views/flattened-index.html#DetailView "DetailView")来显示具体对象了.



### 使用 FormMixin 与 DetailView

想想我们之前合用 [`View`](../../ref/class-based-views/flattened-index.html#View "View") 和[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin") 的例子. 我们想要记录用户对哪些作者感兴趣; 也就是说我们想让用户发表说为什么喜欢这些作者的信息。同样的，我们假设这些数据并没有存放在关系数据库里，而是存在另外一个奥妙之地（其实这里不用关心具体存放到了哪里）。

要实现这一点，自然而然就要设计一个?[`Form`](../../ref/forms/api.html#django.forms.Form "django.forms.Form")，让用户把相关信息通过浏览器发送到Django后台。 另外，我们要巧用[REST](http://en.wikipedia.org/wiki/Representational_state_transfer)方法,这样我们就可以用相同的URL来显示作者和捕捉来自用户的消息了。 让我们重写 `AuthorDetailView` 来实现它。

We’ll keep the `GET` handling from [`DetailView`](../../ref/class-based-views/flattened-index.html#DetailView "DetailView"), although we’ll have to add a [`Form`](../../ref/forms/api.html#django.forms.Form "django.forms.Form") into the context data so we can render it in the template. We’ll also want to pull in form processing from [`FormMixin`](../../ref/class-based-views/mixins-editing.html#django.views.generic.edit.FormMixin "django.views.generic.edit.FormMixin"), and write a bit of code so that on `POST` the form gets called appropriately.



Note

We use [`FormMixin`](../../ref/class-based-views/mixins-editing.html#django.views.generic.edit.FormMixin "django.views.generic.edit.FormMixin") and implement `post()` ourselves rather than try to mix [`DetailView`](../../ref/class-based-views/flattened-index.html#DetailView "DetailView") with [`FormView`](../../ref/class-based-views/flattened-index.html#FormView "FormView") (which provides a suitable `post()` already) because both of the views implement `get()`, and things would get much more confusing.



Our new `AuthorDetail` looks like this:





```
# CAUTION: you almost certainly do not want to do this.
# It is provided as part of a discussion of problems you can
# run into when combining different generic class-based view
# functionality that is not designed to be used together.

from django import forms
from django.http import HttpResponseForbidden
from django.core.urlresolvers import reverse
from django.views.generic import DetailView
from django.views.generic.edit import FormMixin
from books.models import Author

class AuthorInterestForm(forms.Form):
    message = forms.CharField()

class AuthorDetail(FormMixin, DetailView):
    model = Author
    form_class = AuthorInterestForm

    def get_success_url(self):
        return reverse('author-detail', kwargs={'pk': self.object.pk})

    def get_context_data(self, **kwargs):
        context = super(AuthorDetail, self).get_context_data(**kwargs)
        context['form'] = self.get_form()
        return context

    def post(self, request, *args, **kwargs):
        if not request.user.is_authenticated():
            return HttpResponseForbidden()
        self.object = self.get_object()
        form = self.get_form()
        if form.is_valid():
            return self.form_valid(form)
        else:
            return self.form_invalid(form)

    def form_valid(self, form):
        # Here, we would record the user's interest using the message
        # passed in form.cleaned_data['message']
        return super(AuthorDetail, self).form_valid(form)

```





`get_success_url()` is just providing somewhere to redirect to, which gets used in the default implementation of `form_valid()`. We have to provide our own `post()` as noted earlier, and override `get_context_data()` to make the [`Form`](../../ref/forms/api.html#django.forms.Form "django.forms.Form") available in the context data.





### 优化方案

It should be obvious that the number of subtle interactions between [`FormMixin`](../../ref/class-based-views/mixins-editing.html#django.views.generic.edit.FormMixin "django.views.generic.edit.FormMixin") and [`DetailView`](../../ref/class-based-views/flattened-index.html#DetailView "DetailView") is already testing our ability to manage things. 你不太可能会去想自己写这种类的。

In this case, it would be fairly easy to just write the `post()` method yourself, keeping [`DetailView`](../../ref/class-based-views/flattened-index.html#DetailView "DetailView") as the only generic functionality, although writing [`Form`](../../ref/forms/api.html#django.forms.Form "django.forms.Form") handling code involves a lot of duplication.

Alternatively, it would still be easier than the above approach to have a separate view for processing the form, which could use [`FormView`](../../ref/class-based-views/generic-editing.html#django.views.generic.edit.FormView "django.views.generic.edit.FormView") distinct from [`DetailView`](../../ref/class-based-views/flattened-index.html#DetailView "DetailView") without concerns.





### 其他可选的方案

What we’re really trying to do here is to use two different class based views from the same URL. So why not do just that? We have a very clear division here: `GET` requests should get the [`DetailView`](../../ref/class-based-views/flattened-index.html#DetailView "DetailView") (with the [`Form`](../../ref/forms/api.html#django.forms.Form "django.forms.Form") added to the context data), and `POST` requests should get the [`FormView`](../../ref/class-based-views/flattened-index.html#FormView "FormView"). Let’s set up those views first.

The `AuthorDisplay` view is almost the same as [_when we first introduced AuthorDetail_](generic-display.html#generic-views-extra-work); we have to write our own `get_context_data()` to make the `AuthorInterestForm` available to the template. We’ll skip the `get_object()` override from before for clarity:





```
from django.views.generic import DetailView
from django import forms
from books.models import Author

class AuthorInterestForm(forms.Form):
    message = forms.CharField()

class AuthorDisplay(DetailView):
    model = Author

    def get_context_data(self, **kwargs):
        context = super(AuthorDisplay, self).get_context_data(**kwargs)
        context['form'] = AuthorInterestForm()
        return context

```





?`AuthorInterest`是一个简单的 [`FormView`](../../ref/class-based-views/flattened-index.html#FormView "FormView"), 但是我们不得不把[`SingleObjectMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectMixin "django.views.generic.detail.SingleObjectMixin")引入进来，这样我们才能定位我们评论的作者，并且我们还要记得设置`template_name`来确保form出错时使用?`GET`会渲染到?`AuthorDisplay`相同的模板 :





```
from django.core.urlresolvers import reverse
from django.http import HttpResponseForbidden
from django.views.generic import FormView
from django.views.generic.detail import SingleObjectMixin

class AuthorInterest(SingleObjectMixin, FormView):
    template_name = 'books/author_detail.html'
    form_class = AuthorInterestForm
    model = Author

    def post(self, request, *args, **kwargs):
        if not request.user.is_authenticated():
            return HttpResponseForbidden()
        self.object = self.get_object()
        return super(AuthorInterest, self).post(request, *args, **kwargs)

    def get_success_url(self):
        return reverse('author-detail', kwargs={'pk': self.object.pk})

```





Finally we bring this together in a new `AuthorDetail` view. We already know that calling [`as_view()`](../../ref/class-based-views/base.html#django.views.generic.base.View.as_view "django.views.generic.base.View.as_view") on a class-based view gives us something that behaves exactly like a function based view, so we can do that at the point we choose between the two subviews.

You can of course pass through keyword arguments to [`as_view()`](../../ref/class-based-views/base.html#django.views.generic.base.View.as_view "django.views.generic.base.View.as_view") in the same way you would in your URLconf, such as if you wanted the `AuthorInterest` behavior to also appear at another URL but using a different template:





```
from django.views.generic import View

class AuthorDetail(View):

    def get(self, request, *args, **kwargs):
        view = AuthorDisplay.as_view()
        return view(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        view = AuthorInterest.as_view()
        return view(request, *args, **kwargs)

```





This approach can also be used with any other generic class-based views or your own class-based views inheriting directly from [`View`](../../ref/class-based-views/flattened-index.html#View "View") or [`TemplateView`](../../ref/class-based-views/flattened-index.html#TemplateView "TemplateView"), as it keeps the different views as separate as possible.







## 返回HTML 以外的内容

基于类的视图在同一件事需要实现多次的时候非常有优势。假设你正在编写API，每个视图应该返回JSON 而不是渲染后的HTML。

我们可以创建一个Mixin 类来处理JSON 的转换，并将它用于所有的视图。

例如，一个简单的JSON Mixin 可能像这样：





```
from django.http import JsonResponse

class JSONResponseMixin(object):
    """
 A mixin that can be used to render a JSON response.
 """
    def render_to_json_response(self, context, **response_kwargs):
        """
 Returns a JSON response, transforming 'context' to make the payload.
 """
        return JsonResponse(
            self.get_data(context),
            **response_kwargs
        )

    def get_data(self, context):
        """
 Returns an object that will be serialized as JSON by json.dumps().
 """
        # Note: This is *EXTREMELY* naive; in reality, you'll need
        # to do much more complex handling to ensure that arbitrary
        # objects -- such as Django model instances or querysets
        # -- can be serialized as JSON.
        return context

```







注

查看[_序列化Django 对象_](../serialization.html) 的文档，其中有如何正确转换Django 模型和查询集到JSON 的更多信息。



该Mixin 提供一个`render_to_json_response()` 方法，它与 [`render_to_response()`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin.render_to_response "django.views.generic.base.TemplateResponseMixin.render_to_response") 的参数相同。要使用它，我们只需要将它与`TemplateView` 组合，并覆盖`render_to_response()` 来调用`render_to_json_response()`：





```
from django.views.generic import TemplateView

class JSONView(JSONResponseMixin, TemplateView):
    def render_to_response(self, context, **response_kwargs):
        return self.render_to_json_response(context, **response_kwargs)

```





同样地，我们可以将我们的Mixin 与某个通用的视图一起使用。我们可以实现自己的[`DetailView`](../../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView") 版本，将`JSONResponseMixin` 和`django.views.generic.detail.BaseDetailView` 组合– (the [`DetailView`](../../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView") before template rendering behavior has been mixed in):





```
from django.views.generic.detail import BaseDetailView

class JSONDetailView(JSONResponseMixin, BaseDetailView):
    def render_to_response(self, context, **response_kwargs):
        return self.render_to_json_response(context, **response_kwargs)

```





这个视图可以和其它[`DetailView`](../../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView") 一样使用，它们的行为完全相同 —— 除了响应的格式之外。

如果你想更进一步，你可以组合[`DetailView`](../../ref/class-based-views/generic-display.html#django.views.generic.detail.DetailView "django.views.generic.detail.DetailView") 的子类，它根据HTTP 请求的某个属性_既_能够返回HTML 又能够返回JSON 内容，例如查询参数或HTTP 头部。这只需将`JSONResponseMixin` 和[`SingleObjectTemplateResponseMixin`](../../ref/class-based-views/mixins-single-object.html#django.views.generic.detail.SingleObjectTemplateResponseMixin "django.views.generic.detail.SingleObjectTemplateResponseMixin") 组合，并覆盖[`render_to_response()`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin.render_to_response "django.views.generic.base.TemplateResponseMixin.render_to_response") 的实现以根据用户请求的响应类型进行正确的渲染：





```
from django.views.generic.detail import SingleObjectTemplateResponseMixin

class HybridDetailView(JSONResponseMixin, SingleObjectTemplateResponseMixin, BaseDetailView):
    def render_to_response(self, context):
        # Look for a 'format=json' GET argument
        if self.request.GET.get('format') == 'json':
            return self.render_to_json_response(context)
        else:
            return super(HybridDetailView, self).render_to_response(context)

```





由于Python 解析方法重载的方式，`super(HybridDetailView, self).render_to_response(context)` 调用将以调用 [`TemplateResponseMixin`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin "django.views.generic.base.TemplateResponseMixin") 的[`render_to_response()`](../../ref/class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin.render_to_response "django.views.generic.base.TemplateResponseMixin.render_to_response") 实现结束。



