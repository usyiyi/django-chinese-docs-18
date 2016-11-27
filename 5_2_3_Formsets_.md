{% raw %}

# 表单集





_class_ `BaseFormSet`[[source]](../../_modules/django/forms/formsets.html#BaseFormSet)



表单集是同一个页面上多个表单的抽象。它非常类似于一个数据表格。假设有下述表单：





```
>>> from django import forms
>>> class ArticleForm(forms.Form):
...     title = forms.CharField()
...     pub_date = forms.DateField()

```





你可能希望允许用户一次创建多个Article。你可以根据`ArticleForm` 创建一个表单集：





```
>>> from django.forms.formsets import formset_factory
>>> ArticleFormSet = formset_factory(ArticleForm)

```





你已经创建一个命名为`ArticleFormSet` 的表单集。表单集让你能迭代表单集中的表单并显示它们，就和普通的表单一样：





```
>>> formset = ArticleFormSet()
>>> for form in formset:
...     print(form.as_table())
<tr><th><label for="id_form-0-title">Title:</label></th><td><input type="text" name="form-0-title" id="id_form-0-title" /></td></tr>
<tr><th><label for="id_form-0-pub_date">Pub date:</label></th><td><input type="text" name="form-0-pub_date" id="id_form-0-pub_date" /></td></tr>

```





正如你所看到的，这里仅显示一个空表单。显示的表单的数目通过`extra` 参数控制。默认情况下，[`formset_factory()`](../../ref/forms/formsets.html#django.forms.formsets.formset_factory "django.forms.formsets.formset_factory") 定义一个表单；下面的示例将显示两个空表单：





```
>>> ArticleFormSet = formset_factory(ArticleForm, extra=2)

```





对`formset` 的迭代将以它们创建时的顺序渲染表单。通过提供一个`__iter__()` 方法，可以改变这个顺序。

表单集还可以索引，它将返回对应的表单。如果覆盖`__iter__`，你还需要覆盖`__getitem__` 以获得一致的行为。



## 表单集的初始数据

初始数据体现着表单集的主要功能。如上所述，你可以定义表单的数目。它表示除了从初始数据生成的表单之外，还要生成多少个额外的表单。让我们看个例子：





```
>>> import datetime
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm
>>> ArticleFormSet = formset_factory(ArticleForm, extra=2)
>>> formset = ArticleFormSet(initial=[
...     {'title': 'Django is now open source',
...      'pub_date': datetime.date.today(),}
... ])

>>> for form in formset:
...     print(form.as_table())
<tr><th><label for="id_form-0-title">Title:</label></th><td><input type="text" name="form-0-title" value="Django is now open source" id="id_form-0-title" /></td></tr>
<tr><th><label for="id_form-0-pub_date">Pub date:</label></th><td><input type="text" name="form-0-pub_date" value="2008-05-12" id="id_form-0-pub_date" /></td></tr>
<tr><th><label for="id_form-1-title">Title:</label></th><td><input type="text" name="form-1-title" id="id_form-1-title" /></td></tr>
<tr><th><label for="id_form-1-pub_date">Pub date:</label></th><td><input type="text" name="form-1-pub_date" id="id_form-1-pub_date" /></td></tr>
<tr><th><label for="id_form-2-title">Title:</label></th><td><input type="text" name="form-2-title" id="id_form-2-title" /></td></tr>
<tr><th><label for="id_form-2-pub_date">Pub date:</label></th><td><input type="text" name="form-2-pub_date" id="id_form-2-pub_date" /></td></tr>

```





上面现在一共有三个表单。一个是初始数据生成的，还有两个是额外的表单。还要注意的是，我们传递的初始数据是一个由字典组成的列表。



另见

[_利用模型表单集从模型中创建表单集_](modelforms.html#model-formsets)。







## Limiting the maximum number of forms

?[`formset_factory()`](../../ref/forms/formsets.html#django.forms.formsets.formset_factory "django.forms.formsets.formset_factory")的?`max_num` 参数 ，给予你限制表单集展示表单个数的能力





```
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm
>>> ArticleFormSet = formset_factory(ArticleForm, extra=2, max_num=1)
>>> formset = ArticleFormSet()
>>> for form in formset:
...     print(form.as_table())
<tr><th><label for="id_form-0-title">Title:</label></th><td><input type="text" name="form-0-title" id="id_form-0-title" /></td></tr>
<tr><th><label for="id_form-0-pub_date">Pub date:</label></th><td><input type="text" name="form-0-pub_date" id="id_form-0-pub_date" /></td></tr>

```





假如 `max_num`的值 比已经在初始化数据中存在的条目数目多的话, `extra`对应个数的额外空表单将会被添加到表单集， 只要表单总数不超过 `max_num`. For example, if `extra=2` and `max_num=2` and the formset is initialized with one `initial` item, a form for the initial item and one blank form will be displayed.

假如初始化数据的条目超过 `max_num`的值, 所有初始化数据表单都会被展现并且忽视 `max_num`值的限定 ，而且不会有额外的表单被呈现。比如, 如果`extra=3` ，`max_num=1` 并且表单集由两个初始化条蜜，那么两个带有初始化数据的表单将被呈现。

`max_num` 的值为 `None` (默认值) 等同于限制了一个比较高的展现表单数目(1000个). 实际上就是等同于没限制.

默认的, `max_num` 只影响了表单的数目展示，但不影响验证. 假如 `validate_max=True` 传给了 [`formset_factory()`](../../ref/forms/formsets.html#django.forms.formsets.formset_factory "django.forms.formsets.formset_factory"), 然后 `max_num`才将会影响验证. See [_Validating the number of forms in a formset_](#validate-max).





## Formset validation

表单集的验证几乎和 一般的`Form`一样. 表单集里面有一个 `is_valid` 的方法来提供快捷的验证所有表单的功能。





```
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm
>>> ArticleFormSet = formset_factory(ArticleForm)
>>> data = {
...     'form-TOTAL_FORMS': '1',
...     'form-INITIAL_FORMS': '0',
...     'form-MAX_NUM_FORMS': '',
... }
>>> formset = ArticleFormSet(data)
>>> formset.is_valid()
True

```





We passed in no data to the formset which is resulting in a valid form. The formset is smart enough to ignore extra forms that were not changed. If we provide an invalid article:





```
>>> data = {
...     'form-TOTAL_FORMS': '2',
...     'form-INITIAL_FORMS': '0',
...     'form-MAX_NUM_FORMS': '',
...     'form-0-title': 'Test',
...     'form-0-pub_date': '1904-06-16',
...     'form-1-title': 'Test',
...     'form-1-pub_date': '', # <-- this date is missing but required
... }
>>> formset = ArticleFormSet(data)
>>> formset.is_valid()
False
>>> formset.errors
[{}, {'pub_date': ['This field is required.']}]

```





正如我们看见的, `formset.errors` 是一个列表， 他包含的错误信息正好与表单集内的表单一一对应错误检查会在两个表单中分别执行，被预见的错误出现错误列表的第二项



`BaseFormSet.total_error_count`()[[source]](../../_modules/django/forms/formsets.html#BaseFormSet.total_error_count)



想知道表单集内有多少个错误可以使用`total_error_count`方法?





```
>>> # Using the previous example
>>> formset.errors
[{}, {'pub_date': ['This field is required.']}]
>>> len(formset.errors)
2
>>> formset.total_error_count()
1

```





我们也可以检查表单数据是否从初始值发生了变化 (i.e. the form was sent without any data):





```
>>> data = {
...     'form-TOTAL_FORMS': '1',
...     'form-INITIAL_FORMS': '0',
...     'form-MAX_NUM_FORMS': '',
...     'form-0-title': '',
...     'form-0-pub_date': '',
... }
>>> formset = ArticleFormSet(data)
>>> formset.has_changed()
False

```







### Understanding the ManagementForm

你也许已经注意到了那些附加的数据 (`form-TOTAL_FORMS`, `form-INITIAL_FORMS` and `form-MAX_NUM_FORMS`) 他们是必要的，且必须位于表单集数据的最上方? 这些必须传递给`ManagementForm`. ManagementFormThis 用于管理表单集中的表单. 如果你不提供这些数据，将会触发异常





```
>>> data = {
...     'form-0-title': 'Test',
...     'form-0-pub_date': '',
... }
>>> formset = ArticleFormSet(data)
Traceback (most recent call last):
...
django.forms.utils.ValidationError: ['ManagementForm data is missing or has been tampered with']

```





也同样用于记录多少的表单实例将被展示If you are adding new forms via JavaScript, you should increment the count fields in this form as well. On the other hand, if you are using JavaScript to allow deletion of existing objects, then you need to ensure the ones being removed are properly marked for deletion by including `form-#-DELETE` in the `POST` data. It is expected that all forms are present in the `POST` data regardless.

The management form is available as an attribute of the formset itself. When rendering a formset in a template, you can include all the management data by rendering `{{ my_formset.management_form }}` (substituting the name of your formset as appropriate).





### total_form_count and

`BaseFormSet` has a couple of methods that are closely related to the `ManagementForm`, `total_form_count` and `initial_form_count`.

`total_form_count` returns the total number of forms in this formset. `initial_form_count` returns the number of forms in the formset that were pre-filled, and is also used to determine how many forms are required. You will probably never need to override either of these methods, so please be sure you understand what they do before doing so.





### empty_form

`BaseFormSet` provides an additional attribute `empty_form` which returns a form instance with a prefix of `__prefix__` for easier use in dynamic forms with JavaScript.





### Custom formset validation

A formset has a `clean` method similar to the one on a `Form` class. This is where you define your own validation that works at the formset level:





```
>>> from django.forms.formsets import BaseFormSet
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm

>>> class BaseArticleFormSet(BaseFormSet):
...     def clean(self):
...         """Checks that no two articles have the same title."""
...         if any(self.errors):
...             # Don't bother validating the formset unless each form is valid on its own
...             return
...         titles = []
...         for form in self.forms:
...             title = form.cleaned_data['title']
...             if title in titles:
...                 raise forms.ValidationError("Articles in a set must have distinct titles.")
...             titles.append(title)

>>> ArticleFormSet = formset_factory(ArticleForm, formset=BaseArticleFormSet)
>>> data = {
...     'form-TOTAL_FORMS': '2',
...     'form-INITIAL_FORMS': '0',
...     'form-MAX_NUM_FORMS': '',
...     'form-0-title': 'Test',
...     'form-0-pub_date': '1904-06-16',
...     'form-1-title': 'Test',
...     'form-1-pub_date': '1912-06-23',
... }
>>> formset = ArticleFormSet(data)
>>> formset.is_valid()
False
>>> formset.errors
[{}, {}]
>>> formset.non_form_errors()
['Articles in a set must have distinct titles.']

```





The formset `clean` method is called after all the `Form.clean` methods have been called. The errors will be found using the `non_form_errors()` method on the formset.







## Validating the number of forms in a formset

Django 提供了两种方法去检查表单能够提交的最大数和最小数，应用如果需要更多的关于提交数量的自定义验证逻辑，应该使用自定义表单击验证



### validate_max

I如果`validate_max=True` 被提交给 [`formset_factory()`](../../ref/forms/formsets.html#django.forms.formsets.formset_factory "django.forms.formsets.formset_factory"), validation 将在数据集中检查被提交表单的数量, 减去被标记删除的, 必须小于等于`max_num`.





```
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm
>>> ArticleFormSet = formset_factory(ArticleForm, max_num=1, validate_max=True)
>>> data = {
...     'form-TOTAL_FORMS': '2',
...     'form-INITIAL_FORMS': '0',
...     'form-MIN_NUM_FORMS': '',
...     'form-MAX_NUM_FORMS': '',
...     'form-0-title': 'Test',
...     'form-0-pub_date': '1904-06-16',
...     'form-1-title': 'Test 2',
...     'form-1-pub_date': '1912-06-23',
... }
>>> formset = ArticleFormSet(data)
>>> formset.is_valid()
False
>>> formset.errors
[{}, {}]
>>> formset.non_form_errors()
['Please submit 1 or fewer forms.']

```





`validate_max=True` validates 将会对`max_num` 严格限制，即使提供的初始数据超过 `max_num` 而导致其无效



Note

Regardless of `validate_max`, if the number of forms in a data set exceeds `max_num` by more than 1000, then the form will fail to validate as if `validate_max` were set, and additionally only the first 1000 forms above `max_num` will be validated. The remainder will be truncated entirely. This is to protect against memory exhaustion attacks using forged POST requests.







### validate_min

New in Django 1.7.

If `validate_min=True` is passed to [`formset_factory()`](../../ref/forms/formsets.html#django.forms.formsets.formset_factory "django.forms.formsets.formset_factory"), validation will also check that the number of forms in the data set, minus those marked for deletion, is greater than or equal to `min_num`.





```
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm
>>> ArticleFormSet = formset_factory(ArticleForm, min_num=3, validate_min=True)
>>> data = {
...     'form-TOTAL_FORMS': '2',
...     'form-INITIAL_FORMS': '0',
...     'form-MIN_NUM_FORMS': '',
...     'form-MAX_NUM_FORMS': '',
...     'form-0-title': 'Test',
...     'form-0-pub_date': '1904-06-16',
...     'form-1-title': 'Test 2',
...     'form-1-pub_date': '1912-06-23',
... }
>>> formset = ArticleFormSet(data)
>>> formset.is_valid()
False
>>> formset.errors
[{}, {}]
>>> formset.non_form_errors()
['Please submit 3 or more forms.']

```





Changed in Django 1.7:

The `min_num` and `validate_min` parameters were added to [`formset_factory()`](../../ref/forms/formsets.html#django.forms.formsets.formset_factory "django.forms.formsets.formset_factory").









## 表单的排序和删除行为

[`formset_factory()`](../../ref/forms/formsets.html#django.forms.formsets.formset_factory "django.forms.formsets.formset_factory")提供两个可选参数`can_order` 和`can_delete` 来实现表单集中表单的排序和删除。



### can_order



`BaseFormSet.can_order`



Default: `False`

使你创建能排序的表单集。





```
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm
>>> ArticleFormSet = formset_factory(ArticleForm, can_order=True)
>>> formset = ArticleFormSet(initial=[
...     {'title': 'Article #1', 'pub_date': datetime.date(2008, 5, 10)},
...     {'title': 'Article #2', 'pub_date': datetime.date(2008, 5, 11)},
... ])
>>> for form in formset:
...     print(form.as_table())
<tr><th><label for="id_form-0-title">Title:</label></th><td><input type="text" name="form-0-title" value="Article #1" id="id_form-0-title" /></td></tr>
<tr><th><label for="id_form-0-pub_date">Pub date:</label></th><td><input type="text" name="form-0-pub_date" value="2008-05-10" id="id_form-0-pub_date" /></td></tr>
<tr><th><label for="id_form-0-ORDER">Order:</label></th><td><input type="number" name="form-0-ORDER" value="1" id="id_form-0-ORDER" /></td></tr>
<tr><th><label for="id_form-1-title">Title:</label></th><td><input type="text" name="form-1-title" value="Article #2" id="id_form-1-title" /></td></tr>
<tr><th><label for="id_form-1-pub_date">Pub date:</label></th><td><input type="text" name="form-1-pub_date" value="2008-05-11" id="id_form-1-pub_date" /></td></tr>
<tr><th><label for="id_form-1-ORDER">Order:</label></th><td><input type="number" name="form-1-ORDER" value="2" id="id_form-1-ORDER" /></td></tr>
<tr><th><label for="id_form-2-title">Title:</label></th><td><input type="text" name="form-2-title" id="id_form-2-title" /></td></tr>
<tr><th><label for="id_form-2-pub_date">Pub date:</label></th><td><input type="text" name="form-2-pub_date" id="id_form-2-pub_date" /></td></tr>
<tr><th><label for="id_form-2-ORDER">Order:</label></th><td><input type="number" name="form-2-ORDER" id="id_form-2-ORDER" /></td></tr>

```





它会给每个表单添加一个字段，新字段命名 `ORDER`并且是 `forms.IntegerField`类型。它根据初始数据，为这些表单自动生成数值。下面让我们看一下，如果用户改变这个值会发生什么变化:





```
>>> data = {
...     'form-TOTAL_FORMS': '3',
...     'form-INITIAL_FORMS': '2',
...     'form-MAX_NUM_FORMS': '',
...     'form-0-title': 'Article #1',
...     'form-0-pub_date': '2008-05-10',
...     'form-0-ORDER': '2',
...     'form-1-title': 'Article #2',
...     'form-1-pub_date': '2008-05-11',
...     'form-1-ORDER': '1',
...     'form-2-title': 'Article #3',
...     'form-2-pub_date': '2008-05-01',
...     'form-2-ORDER': '0',
... }

>>> formset = ArticleFormSet(data, initial=[
...     {'title': 'Article #1', 'pub_date': datetime.date(2008, 5, 10)},
...     {'title': 'Article #2', 'pub_date': datetime.date(2008, 5, 11)},
... ])
>>> formset.is_valid()
True
>>> for form in formset.ordered_forms:
...     print(form.cleaned_data)
{'pub_date': datetime.date(2008, 5, 1), 'ORDER': 0, 'title': 'Article #3'}
{'pub_date': datetime.date(2008, 5, 11), 'ORDER': 1, 'title': 'Article #2'}
{'pub_date': datetime.date(2008, 5, 10), 'ORDER': 2, 'title': 'Article #1'}

```









### can_delete



`BaseFormSet.can_delete`



Default: `False`

使你创建一个表单集，可以选择删除一些表单。





```
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm
>>> ArticleFormSet = formset_factory(ArticleForm, can_delete=True)
>>> formset = ArticleFormSet(initial=[
...     {'title': 'Article #1', 'pub_date': datetime.date(2008, 5, 10)},
...     {'title': 'Article #2', 'pub_date': datetime.date(2008, 5, 11)},
... ])
>>> for form in formset:
....    print(form.as_table())
<input type="hidden" name="form-TOTAL_FORMS" value="3" id="id_form-TOTAL_FORMS" /><input type="hidden" name="form-INITIAL_FORMS" value="2" id="id_form-INITIAL_FORMS" /><input type="hidden" name="form-MAX_NUM_FORMS" id="id_form-MAX_NUM_FORMS" />
<tr><th><label for="id_form-0-title">Title:</label></th><td><input type="text" name="form-0-title" value="Article #1" id="id_form-0-title" /></td></tr>
<tr><th><label for="id_form-0-pub_date">Pub date:</label></th><td><input type="text" name="form-0-pub_date" value="2008-05-10" id="id_form-0-pub_date" /></td></tr>
<tr><th><label for="id_form-0-DELETE">Delete:</label></th><td><input type="checkbox" name="form-0-DELETE" id="id_form-0-DELETE" /></td></tr>
<tr><th><label for="id_form-1-title">Title:</label></th><td><input type="text" name="form-1-title" value="Article #2" id="id_form-1-title" /></td></tr>
<tr><th><label for="id_form-1-pub_date">Pub date:</label></th><td><input type="text" name="form-1-pub_date" value="2008-05-11" id="id_form-1-pub_date" /></td></tr>
<tr><th><label for="id_form-1-DELETE">Delete:</label></th><td><input type="checkbox" name="form-1-DELETE" id="id_form-1-DELETE" /></td></tr>
<tr><th><label for="id_form-2-title">Title:</label></th><td><input type="text" name="form-2-title" id="id_form-2-title" /></td></tr>
<tr><th><label for="id_form-2-pub_date">Pub date:</label></th><td><input type="text" name="form-2-pub_date" id="id_form-2-pub_date" /></td></tr>
<tr><th><label for="id_form-2-DELETE">Delete:</label></th><td><input type="checkbox" name="form-2-DELETE" id="id_form-2-DELETE" /></td></tr>

```





与`can_order`类似，它添加一个名为`DELETE` 的新字段，并且是`forms.BooleanField`类型。如下，你可以通过`deleted_forms`来获取标记删除字段的数据：





```
>>> data = {
...     'form-TOTAL_FORMS': '3',
...     'form-INITIAL_FORMS': '2',
...     'form-MAX_NUM_FORMS': '',
...     'form-0-title': 'Article #1',
...     'form-0-pub_date': '2008-05-10',
...     'form-0-DELETE': 'on',
...     'form-1-title': 'Article #2',
...     'form-1-pub_date': '2008-05-11',
...     'form-1-DELETE': '',
...     'form-2-title': '',
...     'form-2-pub_date': '',
...     'form-2-DELETE': '',
... }

>>> formset = ArticleFormSet(data, initial=[
...     {'title': 'Article #1', 'pub_date': datetime.date(2008, 5, 10)},
...     {'title': 'Article #2', 'pub_date': datetime.date(2008, 5, 11)},
... ])
>>> [form.cleaned_data for form in formset.deleted_forms]
[{'DELETE': True, 'pub_date': datetime.date(2008, 5, 10), 'title': 'Article #1'}]

```





如果你使用 [`ModelFormSet`](modelforms.html#django.forms.models.BaseModelFormSet "django.forms.models.BaseModelFormSet")，调用 `formset.save()` 将删除那些有删除标记的表单的模型实例。

Changed in Django 1.7:

如果你调用`formset.save(commit=False)`， 对像将不会被自动删除。你需要调用[`formset.deleted_objects`](modelforms.html#django.forms.models.BaseModelFormSet.deleted_objects "django.forms.models.BaseModelFormSet.deleted_objects")每个对像的 `delete()` 来真正删除他们。





```
>>> instances = formset.save(commit=False)
>>> for obj in formset.deleted_objects:
...     obj.delete()

```





如果你想保持向前兼容 Django 1.6 或更早的版本，你需要这样做：





```
>>> try:
>>>     # For Django 1.7+
>>>     for obj in formset.deleted_objects:
>>>         obj.delete()
>>> except AssertionError:
>>>     # Django 1.6 and earlier already deletes the objects, trying to
>>>     # delete them a second time raises an AssertionError.
>>>     pass

```







On the other hand, if you are using a plain `FormSet`, it’s up to you to handle `formset.deleted_forms`, perhaps in your formset’s `save()` method, as there’s no general notion of what it means to delete a form.







## 给表单集添加一个额外的字段

如果你想往表单集中添加额外的字段，是十分容易完成的，表单集的基类（BaseFormSet）提供了 `add_fields` 方法。可以简单的通过重写这个方法来添加你自己的字段，甚至重新定义order和deletion字段的方法和属性：





```
>>> from django.forms.formsets import BaseFormSet
>>> from django.forms.formsets import formset_factory
>>> from myapp.forms import ArticleForm
>>> class BaseArticleFormSet(BaseFormSet):
...     def add_fields(self, form, index):
...         super(BaseArticleFormSet, self).add_fields(form, index)
...         form.fields["my_field"] = forms.CharField()

>>> ArticleFormSet = formset_factory(ArticleForm, formset=BaseArticleFormSet)
>>> formset = ArticleFormSet()
>>> for form in formset:
...     print(form.as_table())
<tr><th><label for="id_form-0-title">Title:</label></th><td><input type="text" name="form-0-title" id="id_form-0-title" /></td></tr>
<tr><th><label for="id_form-0-pub_date">Pub date:</label></th><td><input type="text" name="form-0-pub_date" id="id_form-0-pub_date" /></td></tr>
<tr><th><label for="id_form-0-my_field">My field:</label></th><td><input type="text" name="form-0-my_field" id="id_form-0-my_field" /></td></tr>

```









## 在视图和模板中使用表单集

在视图中使用表单集就像使用标准的`Form` 类一样简单，唯一要做的就是确信你在模板中处理表单。让我们看一个简单视图：





```
from django.forms.formsets import formset_factory
from django.shortcuts import render_to_response
from myapp.forms import ArticleForm

def manage_articles(request):
    ArticleFormSet = formset_factory(ArticleForm)
    if request.method == 'POST':
        formset = ArticleFormSet(request.POST, request.FILES)
        if formset.is_valid():
            # do something with the formset.cleaned_data
            pass
    else:
        formset = ArticleFormSet()
    return render_to_response('manage_articles.html', {'formset': formset})

```





`manage_articles.html` 模板也可以像这样：





```
<form method="post" action="">
    {{ formset.management_form }}
    <table>
        {% for form in formset 
        {{ form }}
        {% endfor </table>
</form> 
```





不过，上面可以用一个快捷写法，让表单集来分发管理表单：





```
<form method="post" action="">
    <table>
        {{ formset }}
    </table>
</form>

```





上面表单集调用 `as_table` 方法。



### 手动渲染`can_delete`

如果你在模板中手工渲染字段，那么渲染 `can_delete` 参数用`{{ form.`DELETE }}:





```
<form method="post" action="">
    {{ formset.management_form }}
    {% for form in formset 
        <ul>
            <li>{{ form.title }}</li>
            <li>{{ form.pub_date }}</li>
            {% if formset.can_delete 
                <li>{{ form.DELETE }}</li>
            {% endif 
        </ul>
    {% endfor </form> 
```





类似的，如果表单集有排序功能(`can_order=True`)，可以使用`{{ form.ORDER` }}渲染。





### 在视图中使用多个表单集

可以在视图中使用多个表单集，表单集从表单中借鉴了很多方法你可以使用 `prefix` 给每个表单字段添加前缀，以允许多个字段传递给视图，而不发生命名冲突?让我们看看可以怎么做





```
from django.forms.formsets import formset_factory
from django.shortcuts import render_to_response
from myapp.forms import ArticleForm, BookForm

def manage_articles(request):
    ArticleFormSet = formset_factory(ArticleForm)
    BookFormSet = formset_factory(BookForm)
    if request.method == 'POST':
        article_formset = ArticleFormSet(request.POST, request.FILES, prefix='articles')
        book_formset = BookFormSet(request.POST, request.FILES, prefix='books')
        if article_formset.is_valid() and book_formset.is_valid():
            # do something with the cleaned_data on the formsets.
            pass
    else:
        article_formset = ArticleFormSet(prefix='articles')
        book_formset = BookFormSet(prefix='books')
    return render_to_response('manage_articles.html', {
        'article_formset': article_formset,
        'book_formset': book_formset,
    })

```





你可以以正常的方式渲染模板。记住 `prefix` 在POST请求和非POST 请求中均需设置，以便他能渲染和执行正确





{% endraw %}