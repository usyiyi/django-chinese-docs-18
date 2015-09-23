# Django异常

DJango会抛出一些它自己的异常，以及Python的标准异常。

## Django核心异常

Django核心异常类定义在`django.core.exceptions`中。

### ObjectDoesNotExist

_exception _`ObjectDoesNotExist`[[source]](../_modules/django/core/exceptions.html#ObjectDoesNotExist)

[`DoesNotExist`](models/instances.html#django.db.models.Model.DoesNotExist "django.db.models.Model.DoesNotExist")异常的基类；对`ObjectDoesNotExist`的`try/except`会为所有模型捕获到所有[`DoesNotExist`](models/instances.html#django.db.models.Model.DoesNotExist "django.db.models.Model.DoesNotExist") 异常。

[`ObjectDoesNotExist`](#django.core.exceptions.ObjectDoesNotExist "django.core.exceptions.ObjectDoesNotExist") 和 [`DoesNotExist`](models/instances.html#django.db.models.Model.DoesNotExist "django.db.models.Model.DoesNotExist")的更多信息请见&nbsp;[`get()`](models/querysets.html#django.db.models.query.QuerySet.get "django.db.models.query.QuerySet.get")。

### FieldDoesNotExist

_exception _`FieldDoesNotExist`[[source]](../_modules/django/core/exceptions.html#FieldDoesNotExist)

当被请求的字段在模型或模型的父类中不存在时，`FieldDoesNotExist`异常由模型的&nbsp;`_meta.get_field()`方法抛出。

Changed in Django 1.8:

之前的版本中，异常只在`django.db.models.fields`中定义，并不是公共API的一部分。

### MultipleObjectsReturned

_exception _`MultipleObjectsReturned`[[source]](../_modules/django/core/exceptions.html#MultipleObjectsReturned)

[`MultipleObjectsReturned`](#django.core.exceptions.MultipleObjectsReturned "django.core.exceptions.MultipleObjectsReturned")异常由查询产生，当预期只有一个对象，但是有多个对象返回的时候。这个异常的一个基础版本在[`django.core.exceptions`](#module-django.core.exceptions "django.core.exceptions: Django core exceptions")中提供。每个模型类都包含一个它的子类版本，它可以用于定义返回多个对象的特定的对象类型。

详见[`get()`](models/querysets.html#django.db.models.query.QuerySet.get "django.db.models.query.QuerySet.get")。

### SuspiciousOperation

_exception _`SuspiciousOperation`[[source]](../_modules/django/core/exceptions.html#SuspiciousOperation)

当用户进行的操作在安全方面可疑的时候，抛出[`SuspiciousOperation`](#django.core.exceptions.SuspiciousOperation "django.core.exceptions.SuspiciousOperation")异常，例如篡改会话cookie。`SuspiciousOperation`的子类包括：

*   `DisallowedHost`
*   `DisallowedModelAdminLookup`
*   `DisallowedModelAdminToField`
*   `DisallowedRedirect`
*   `InvalidSessionKey`
*   `SuspiciousFileOperation`
*   `SuspiciousMultipartForm`
*   `SuspiciousSession`

如果`SuspiciousOperation`异常到达了WSGI处理器层，它会在`Error`层记录，并导致[`HttpResponseBadRequest`](request-response.html#django.http.HttpResponseBadRequest "django.http.HttpResponseBadRequest")异常。
详见[_日志文档_](../topics/logging.html)。

### PermissionDenied

_exception _`PermissionDenied`[[source]](../_modules/django/core/exceptions.html#PermissionDenied)

[`PermissionDenied`](#django.core.exceptions.PermissionDenied "django.core.exceptions.PermissionDenied")异常当用户不被允许来执行请求的操作时产生。

### ViewDoesNotExist

_exception _`ViewDoesNotExist`[[source]](../_modules/django/core/exceptions.html#ViewDoesNotExist)

当所请求的视图不存在时，[`ViewDoesNotExist`](#django.core.exceptions.ViewDoesNotExist "django.core.exceptions.ViewDoesNotExist") 异常由&nbsp;[`django.core.urlresolvers`](urlresolvers.html#module-django.core.urlresolvers "django.core.urlresolvers")产生。

### MiddlewareNotUsed

_exception _`MiddlewareNotUsed`[[source]](../_modules/django/core/exceptions.html#MiddlewareNotUsed)

当中间件没有在服务器配置中出现时，产生[`MiddlewareNotUsed`](#django.core.exceptions.MiddlewareNotUsed "django.core.exceptions.MiddlewareNotUsed")异常。

### ImproperlyConfigured

_exception _`ImproperlyConfigured`[[source]](../_modules/django/core/exceptions.html#ImproperlyConfigured)

DJango配置不当时产生[`ImproperlyConfigured`](#django.core.exceptions.ImproperlyConfigured "django.core.exceptions.ImproperlyConfigured")异常 -- 例如，`settings.py`中的值不正确或者不可解析。

### FieldError

_exception _`FieldError`[[source]](../_modules/django/core/exceptions.html#FieldError)

[`FieldError`](#django.core.exceptions.FieldError "django.core.exceptions.FieldError")异常当模型字段上出现问题时产生。它会由以下原因造成：

*   模型中的字段与抽象基类中相同名称的字段冲突。
*   排序造成了一个死循环。
*   关键词不能由过滤器参数解析。
*   字段不能由查询参数中的关键词决定。
*   连接（join）不能在指定对象上使用。
*   字段名称不可用。
*   查询包含了无效的&nbsp;order_by参数。

### ValidationError

_exception _`ValidationError`[[source]](../_modules/django/core/exceptions.html#ValidationError)

当表单或模型字段验证失败时抛出[`ValidationError`](#django.core.exceptions.ValidationError "django.core.exceptions.ValidationError")异常。关于验证的更多信息，请见[_表单字段验证_](forms/validation.html), [_模型字段验证_](models/instances.html#validating-objects) 和 [_验证器参考_](validators.html)。

#### NON_FIELD_ERRORS

`NON_FIELD_ERRORS`

在表单或者模型中不属于特定字段的`ValidationError` 被归类为`NON_FIELD_ERRORS`。This constant is used as a key in dictionaries that otherwise map fields to their respective list of errors.

## URL解析器异常

URL解析器异常定义在`django.core.urlresolvers`中。

### Resolver404

_exception _`Resolver404`[[source]](../_modules/django/core/urlresolvers.html#Resolver404)

当向&nbsp;`resolve()` 传递的路径不映射到视图的时候，[`Resolver404`](#django.core.urlresolvers.Resolver404 "django.core.urlresolvers.Resolver404")异常由[`django.core.urlresolvers.resolve()`](urlresolvers.html#django.core.urlresolvers.resolve "django.core.urlresolvers.resolve")产生。 它是&nbsp;[`django.http.Http404`](../topics/http/views.html#django.http.Http404 "django.http.Http404")的子类。

### NoReverseMatch

_exception _`NoReverseMatch`[[source]](../_modules/django/core/urlresolvers.html#NoReverseMatch)

当你的URLconf中的一个匹配的URL不能基于提供的参数识别时，[`NoReverseMatch`](#django.core.urlresolvers.NoReverseMatch "django.core.urlresolvers.NoReverseMatch") 异常由 [`django.core.urlresolvers`](urlresolvers.html#module-django.core.urlresolvers "django.core.urlresolvers") 产生。

## Database Exceptions

数据库异常由`django.db`导入。

Django封装了标准的数据库异常，以便确保你的DJango代码拥有这些类的通用实现。

_exception _`Error`

_exception _`InterfaceError`

_exception _`DatabaseError`

_exception _`DataError`

_exception _`OperationalError`

_exception _`IntegrityError`

_exception _`InternalError`

_exception _`ProgrammingError`

_exception _`NotSupportedError`

Django数据库异常的包装器的行为和底层的数据库异常一样。详见[**PEP 249**](http://www.python.org/dev/peps/pep-0249)，Python 数据库 API 说明 v2.0。

按照 [**PEP 3134**](http://www.python.org/dev/peps/pep-3134)，`__cause__`属性会在原生（底层）的数据库异常中设置，允许访问所提供的任何附加信息。（注意这一属性在Python 2和 3下面都可用，虽然&nbsp;[**PEP 3134**](http://www.python.org/dev/peps/pep-3134)通常只用于Python 3。）

_exception _`models.``ProtectedError`

使用[`django.db.models.PROTECT`](models/fields.html#django.db.models.PROTECT "django.db.models.PROTECT")时，抛出异常来阻止所引用对象的删除。[`models.`](#django.db.models.ProtectedError "django.db.models.ProtectedError")ProtectedError is a subclass of [`IntegrityError`](#django.db.IntegrityError "django.db.IntegrityError").

## Http异常

HTTP异常由`django.http`导入。

### UnreadablePostError

_exception _`UnreadablePostError`

用户取消上传时抛出[`UnreadablePostError`](#django.http.UnreadablePostError "django.http.UnreadablePostError")异常。

## 事务异常

事务异常定义在`django.db.transaction`中。

### TransactionManagementError

_exception _`TransactionManagementError`[[source]](../_modules/django/db/transaction.html#TransactionManagementError)

对于数据库事务相关的任何问题，抛出[`TransactionManagementError`](#django.db.transaction.TransactionManagementError "django.db.transaction.TransactionManagementError")异常。

## 测试框架异常

由DJango&nbsp;`django.test` 包提供的异常。

### RedirectCycleError

_exception _`client.``RedirectCycleError`

New in Django 1.8\.

当测试客户端检测到重定向的循环或者过长的链时，抛出[`RedirectCycleError`](#django.test.client.RedirectCycleError "django.test.client.RedirectCycleError")异常。

## Python异常

Django在适当的时候也会抛出Python的内建异常。进一步的信息请见[_内建的异常_](https://docs.python.org/3/library/exceptions.html#bltin-exceptions "(in Python v3.4)")的Python文档。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Overview](https://docs.djangoproject.com/en/1.8/ref/exceptions/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
