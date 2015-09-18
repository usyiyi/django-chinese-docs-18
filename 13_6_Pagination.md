# 分页

Django提供了一些类来帮助你管理分页的数据 -- 也就是说，数据被分在不同页面中，并带有“上一页/下一页”标签。这些类位于`django/core/paginator.py`中。

## 示例

向[`Paginator`](#django.core.paginator.Paginator "django.core.paginator.Paginator")提供对象的列表，以及你想为每一页分配的元素数量，它就会为你提供访问每一页上对象的方法：

```
>>> from django.core.paginator import Paginator
>>> objects = ['john', 'paul', 'george', 'ringo']
>>> p = Paginator(objects, 2)

>>> p.count
4
>>> p.num_pages
2
>>> p.page_range
[1, 2]

>>> page1 = p.page(1)
>>> page1
<Page 1 of 2>
>>> page1.object_list
['john', 'paul']

>>> page2 = p.page(2)
>>> page2.object_list
['george', 'ringo']
>>> page2.has_next()
False
>>> page2.has_previous()
True
>>> page2.has_other_pages()
True
>>> page2.next_page_number()
Traceback (most recent call last):
...
EmptyPage: That page contains no results
>>> page2.previous_page_number()
1
>>> page2.start_index() # The 1-based index of the first item on this page
3
>>> page2.end_index() # The 1-based index of the last item on this page
4

>>> p.page(0)
Traceback (most recent call last):
...
EmptyPage: That page number is less than 1
>>> p.page(3)
Traceback (most recent call last):
...
EmptyPage: That page contains no results

```

注意

注意你可以向`Paginator`提供一个列表或元组，Django的`QuerySet`，或者任何带有`count()`或`__len__()`方法的对象。当计算传入的对象所含对象的数量时，`Paginator`会首先尝试调用`count()`，接着如果传入的对象没有`count()`方法则回退调用&nbsp;`len()`。这样会使类似于Django的`QuerySet`的对象使用更加高效的 `count()`方法，如果存在的话。

## 使用 `Paginator`

这里有一些复杂一点的例子，它们在视图中使用&nbsp;[`Paginator`](#django.core.paginator.Paginator "django.core.paginator.Paginator") 来为查询集分页。我们提供视图以及相关的模板来展示如何展示这些结果。这个例子假设你拥有一个已经导入的`Contacts`模型。

视图函数看起来像是这样：

```
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger

def listing(request):
    contact_list = Contacts.objects.all()
    paginator = Paginator(contact_list, 25) # Show 25 contacts per page

    page = request.GET.get('page')
    try:
        contacts = paginator.page(page)
    except PageNotAnInteger:
        # If page is not an integer, deliver first page.
        contacts = paginator.page(1)
    except EmptyPage:
        # If page is out of range (e.g. 9999), deliver last page of results.
        contacts = paginator.page(paginator.num_pages)

    return render_to_response('list.html', {"contacts": contacts})

```

在`list.html`模板中，你会想要包含页面之间的导航，以及来自对象本身的任何有趣的信息：

```

<div class="pagination">
    <span class="step-links">

        <span class="current">
            Page  of .
        </span>

    </span>
</div>

```

## Paginator objects

[`Paginator`](#django.core.paginator.Paginator "django.core.paginator.Paginator")类拥有以下构造器：

_class _`Paginator`(_object_list_, _per_page_, _orphans=0_, _allow_empty_first_page=True_)[[source]](../_modules/django/core/paginator.html#Paginator)

### 所需参数

`object_list`

A list, tuple, Django `QuerySet`, or other sliceable object with a
`count()` or `__len__()` method.

`per_page`

The maximum number of items to include on a page, not including orphans
(see the `orphans` optional argument below).

### 可选参数

`orphans`

The minimum number of items allowed on the last page, defaults to zero.
Use this when you don’t want to have a last page with very few items.
If the last page would normally have a number of items less than or equal
to `orphans`, then those items will be added to the previous page (which
becomes the last page) instead of leaving the items on a page by
themselves. For example, with 23 items, `per_page=10`, and
`orphans=3`, there will be two pages; the first page with 10 items and
the  second (and last) page with 13 items.

`allow_empty_first_page`

Whether or not the first page is allowed to be empty.  If `False` and
`object_list` is  empty, then an `EmptyPage` error will be raised.

### 方法

`Paginator.``page`(_number_)[[source]](../_modules/django/core/paginator.html#Paginator.page)

返回在提供的下标处的[`Page`](#django.core.paginator.Page "django.core.paginator.Page")对象，下标以1开始。如果提供的页码不存在，抛出[`InvalidPage`](#django.core.paginator.InvalidPage "django.core.paginator.InvalidPage")异常。

### 属性

`Paginator.``count`

所有页面的对象总数。

注意

当计算`object_list`所含对象的数量时， `Paginator`会首先尝试调用`object_list.count()`。如果`object_list`没有 `count()` 方法，`Paginator` 接着会回退使用`len(object_list)`。这样会使类似于Django’s `QuerySet`的对象使用更加便捷的`count()`方法，如果存在的话。

`Paginator.``num_pages`

页面总数。

`Paginator.``page_range`

页码的范围，从1开始，例如`[1, 2, 3, 4]`。

## InvalidPage exceptions

_exception _`InvalidPage`[[source]](../_modules/django/core/paginator.html#InvalidPage)

异常的基类，当paginator传入一个无效的页码时抛出。

[`Paginator.page()`](#django.core.paginator.Paginator.page "django.core.paginator.Paginator.page")放回在所请求的页面无效（比如不是一个整数）时，或者不包含任何对象时抛出异常。通常，捕获`InvalidPage`异常就够了，但是如果你想更加精细一些，可以捕获以下两个异常之一：

_exception _`PageNotAnInteger`[[source]](../_modules/django/core/paginator.html#PageNotAnInteger)

当向`page()`提供一个不是整数的值时抛出。

_exception _`EmptyPage`[[source]](../_modules/django/core/paginator.html#EmptyPage)

当向`page()`提供一个有效值，但是那个页面上没有任何对象时抛出。

这两个异常都是[`InvalidPage`](#django.core.paginator.InvalidPage "django.core.paginator.InvalidPage")的子类，所以你可以通过简单的`except InvalidPage`来处理它们。

## Page objects

你通常不需要手动构建 `Page`对象 -- 你可以从[`Paginator.page()`](#django.core.paginator.Paginator.page "django.core.paginator.Paginator.page")来获得它们。

_class _`Page`(_object_list_, _number_, _paginator_)[[source]](../_modules/django/core/paginator.html#Page)

当调用`len()`或者直接迭代一个页面的时候，它的行为类似于 [`Page.object_list`](#django.core.paginator.Page.object_list "django.core.paginator.Page.object_list") 的序列。

### 方法

`Page.``has_next`()[[source]](../_modules/django/core/paginator.html#Page.has_next)

Returns `True` if there’s a next page.

`Page.``has_previous`()[[source]](../_modules/django/core/paginator.html#Page.has_previous)

如果有上一页，返回&nbsp;`True`。

`Page.``has_other_pages`()[[source]](../_modules/django/core/paginator.html#Page.has_other_pages)

如果有上一页_或_下一页，返回`True`。

`Page.``next_page_number`()[[source]](../_modules/django/core/paginator.html#Page.next_page_number)

返回下一页的页码。如果下一页不存在，抛出[`InvalidPage`](#django.core.paginator.InvalidPage "django.core.paginator.InvalidPage")异常。

`Page.``previous_page_number`()[[source]](../_modules/django/core/paginator.html#Page.previous_page_number)

返回上一页的页码。如果上一页不存在，抛出[`InvalidPage`](#django.core.paginator.InvalidPage "django.core.paginator.InvalidPage")异常。

`Page.``start_index`()[[source]](../_modules/django/core/paginator.html#Page.start_index)

返回当前页上的第一个对象，相对于分页列表的所有对象的序号，从1开始。比如，将五个对象的列表分为每页两个对象，第二页的[`start_index()`](#django.core.paginator.Page.start_index "django.core.paginator.Page.start_index")会返回`3`。

`Page.``end_index`()[[source]](../_modules/django/core/paginator.html#Page.end_index)

返回当前页上的最后一个对象，相对于分页列表的所有对象的序号，从1开始。 比如，将五个对象的列表分为每页两个对象，第二页的[`end_index()`](#django.core.paginator.Page.end_index "django.core.paginator.Page.end_index") 会返回&nbsp;`4`。

### 属性

`Page.``object_list`

当前页上所有对象的列表。

`Page.``number`

当前页的序号，从1开始。

`Page.``paginator`

相关的[`Paginator`](#django.core.paginator.Paginator "django.core.paginator.Paginator")对象。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Pagination](https://docs.djangoproject.com/en/1.8/topics/pagination/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
