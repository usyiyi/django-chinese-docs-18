# 使用基于类的视图处理表单 #

表单的处理通常有3 个步骤：

+ 初始的的GET （空白或预填充的表单）
+ 带有非法数据的POST（通常重新显示表单和错误信息）
+ 带有合法数据的POST（处理数据并重定向）

你自己实现这些功能经常导致许多重复的样本代码（参见在视图中使用表单）。为了避免这点，Django 提供一系列的通用的基于类的视图用于表单的处理。

## 基本的表单 ##

根据一个简单的联系人表单：

```
#forms.py

from django import forms

class ContactForm(forms.Form):
    name = forms.CharField()
    message = forms.CharField(widget=forms.Textarea)

    def send_email(self):
        # send email using the self.cleaned_data dictionary
        pass
```

可以使用`FormView`来构造其视图：

```
#views.py

from myapp.forms import ContactForm
from django.views.generic.edit import FormView

class ContactView(FormView):
    template_name = 'contact.html'
    form_class = ContactForm
    success_url = '/thanks/'

    def form_valid(self, form):
        # This method is called when valid form data has been POSTed.
        # It should return an HttpResponse.
        form.send_email()
        return super(ContactView, self).form_valid(form)
```

注：

+ `FormView`继承`TemplateResponseMixin`所以这里可以使用`template_name`。
+ `form_valid()`的默认实现只是简单地重定向到`success_url`。

## 模型的表单 ##

通用视图在于模型一起工作时会真正光芒四射。这些通用的视图将自动创建一个ModelForm，只要它们能知道使用哪一个模型类：

+ 如果给出`model`属性，则使用该模型类。
+ 如果`get_object()` 返回一个对象，则使用该对象的类。
+ 如果给出`queryset`，则使用该查询集的模型。

模型表单提供一个`form_valid()` 的实现，它自动保存模型。如果你有特殊的需求，可以覆盖它；参见下面的例子。

你甚至不需要为`CreateView` 和`UpdateView`提供`success_url` —— 如果存在它们将使用模型对象的`get_absolute_url()`。

如果你想使用一个自定义的`ModelForm`（例如添加额外的验证），只需简单地在你的视图上设置`form_class`。

> 注
>
>
当指定一个自定义的表单类时，你必须指定模型，即使`form_class` 可能是一个`ModelForm`。

首先我们需要添加`get_absolute_url()` 到我们的`Author` 类中：

```
#models.py

from django.core.urlresolvers import reverse
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=200)

    def get_absolute_url(self):
        return reverse('author-detail', kwargs={'pk': self.pk})
```

然后我们可以使用`CreateView` 机器伙伴来做实际的工作。注意这里我们是如何配置通用的基于类的视图的；我们自己没有写任何逻辑：

```
#views.py

from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.core.urlresolvers import reverse_lazy
from myapp.models import Author

class AuthorCreate(CreateView):
    model = Author
    fields = ['name']

class AuthorUpdate(UpdateView):
    model = Author
    fields = ['name']

class AuthorDelete(DeleteView):
    model = Author
    success_url = reverse_lazy('author-list')
```

> 注
>
> 这里我们必须使用`reverse_lazy()` 而不是`reverse`，因为在该文件导入时URL 还没有加载。

`fields` 属性的工作方式与`ModelForm` 的内部`Meta`类的`fields` 属性相同。除非你用另外一种方式定义表单类，该属性是必须的，如果没有将引发一个`ImproperlyConfigured` 异常。

如果你同时指定`fields` 和`form_class` 属性，将引发一个`ImproperlyConfigured` 异常。

```
Changed in Django 1.8:

省略fields 属性在以前是允许的，但是导致表单带有模型的所有字段。
```

```
Changed in Django 1.8:

以前，如果fields 和form_class 两个都指定，会默默地忽略 fields。
```

最后，我我们来将这些新的视图放到URLconf 中：

```
#urls.py

from django.conf.urls import url
from myapp.views import AuthorCreate, AuthorUpdate, AuthorDelete

urlpatterns = [
    # ...
    url(r'author/add/$', AuthorCreate.as_view(), name='author_add'),
    url(r'author/(?P<pk>[0-9]+)/$', AuthorUpdate.as_view(), name='author_update'),
    url(r'author/(?P<pk>[0-9]+)/delete/$', AuthorDelete.as_view(), name='author_delete'),
]
```

> 注
>
> 这些表单继承`SingleObjectTemplateResponseMixin`，它使用`template_name_suffix `并基于模型来构造`template_name`。
>
> 在这个例子中：
>
> + `CreateView` 和`UpdateView` 使用 `myapp/author_form.html`
> + `DeleteView` 使用 `myapp/author_confirm_delete.html`
>
> 如果你希望分开`CreateView` 和`UpdateView` 的模板，你可以设置你的视图类的`template_name` 或`template_name_suffix`。

## 模型和request.user ##

为了跟踪使用CreateView 创建一个对象的用户，你可以使用一个自定义的ModelForm 来实现这点。首先，向模型添加外键关联：

```
#models.py

from django.contrib.auth.models import User
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=200)
    created_by = models.ForeignKey(User)

    # ...
```

在这个视图中，请确保你没有将`created_by` 包含进要编辑的字段列表，并覆盖`form_valid()` 来添加这个用户：

```
#views.py

from django.views.generic.edit import CreateView
from myapp.models import Author

class AuthorCreate(CreateView):
    model = Author
    fields = ['name']

    def form_valid(self, form):
        form.instance.created_by = self.request.user
        return super(AuthorCreate, self).form_valid(form)
```

注意，你需要使用`login_required()` 来装饰这个视图，或者在`form_valid()` 中处理未认证的用户。

## AJAX 示例 ##

下面是一个简单的实例，展示你可以如何实现一个表单，使它可以同时为AJAX 请求和‘普通的’表单POST 工作：

```
from django.http import JsonResponse
from django.views.generic.edit import CreateView
from myapp.models import Author

class AjaxableResponseMixin(object):
    """
    Mixin to add AJAX support to a form.
    Must be used with an object-based FormView (e.g. CreateView)
    """
    def form_invalid(self, form):
        response = super(AjaxableResponseMixin, self).form_invalid(form)
        if self.request.is_ajax():
            return JsonResponse(form.errors, status=400)
        else:
            return response

    def form_valid(self, form):
        # We make sure to call the parent's form_valid() method because
        # it might do some processing (in the case of CreateView, it will
        # call form.save() for example).
        response = super(AjaxableResponseMixin, self).form_valid(form)
        if self.request.is_ajax():
            data = {
                'pk': self.object.pk,
            }
            return JsonResponse(data)
        else:
            return response

class AuthorCreate(AjaxableResponseMixin, CreateView):
    model = Author
    fields = ['name']
```

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Built-in editing views](https://docs.djangoproject.com/en/1.8/topics/class-based-views/generic-editing/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
