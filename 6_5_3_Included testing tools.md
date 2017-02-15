

# 测试工具

Django提供了一组小工具，在写测试时派上用场。

## 测试客户端

测试客户端是一个Python类，作为一个虚拟的Web浏览器，允许您测试您的视图，并与您的Django供电的应用程序以编程方式交互。

你可以用测试客户端做的一些事情是：

*   模拟对URL的GET和POST请求，并观察响应 - 从低级HTTP（结果头和状态代码）到页面内容的一切。
*   查看重定向链（如果有），并在每个步骤中检查网址和状态代码。
*   测试给定的请求是否由给定的Django模板呈现，其中模板上下文包含某些值。

请注意，测试客户端不是要替代[Selenium](http://seleniumhq.org/)或其他“浏览器内”框架。Django的测试客户端有不同的焦点。简而言之：

*   使用Django的测试客户端来确定正在渲染正确的模板，并且传递正确的上下文数据。
*   使用[Selenium](http://seleniumhq.org/)等浏览器框架测试网页的_呈现的_ HTML和_行为_，即JavaScript功能。Django还为这些框架提供特殊支持；有关详细信息，请参阅[`LiveServerTestCase`](#django.test.LiveServerTestCase "django.test.LiveServerTestCase")一节。

一个全面的测试套件应该使用两种测试类型的组合。

### 概述和一个快速示例

要使用测试客户端，请实例化`django.test.Client`并检索网页：

```
>>> from django.test import Client
>>> c = Client()
>>> response = c.post('/login/', {'username': 'john', 'password': 'smith'})
>>> response.status_code
200
>>> response = c.get('/customer/details/')
>>> response.content
'<!DOCTYPE html...'

```

正如此示例所建议的，您可以在Python交互式解释器的会话中实例化`Client`。

请注意测试客户端如何工作的几个重要的事情：

*   测试客户端_不_要求Web服务器正在运行。事实上，它将运行很好，没有Web服务器运行在所有！这是因为它避免了HTTP的开销，直接处理Django框架。这有助于使单元测试快速运行。

*   检索网页时，请记住指定网址的_路径_，而不是整个网域。例如，这是正确的：

    ```
    &gt;&gt;&gt; c.get('/login/')

    ```

    这是不正确的：

    ```
    &gt;&gt;&gt; c.get('http://www.example.com/login/')

    ```

    测试客户端无法检索不受您的Django项目驱动的网页。如果需要检索其他网页，请使用Python标准库模块，例如[`urllib`](https://docs.python.org/3/library/urllib.html#module-urllib "(in Python v3.4)")。

*   要解析网址，测试客户端将使用您的[`ROOT_URLCONF`](../../ref/settings.html#std:setting-ROOT_URLCONF)设置指向的任何URLconf。

*   虽然上面的例子可以在Python交互式解释器中工作，但是测试客户端的一些功能，特别是与模板相关的功能，只有在测试运行时_才可用_。

    这样做的原因是，Django的测试运行器执行一些黑魔法，以确定哪个模板由给定的视图加载。这个黑魔法（本质上是Django的模板系统在内存中的修补）只发生在测试运行期间。

*   默认情况下，测试客户端将禁用由您的站点执行的任何CSRF检查。

    如果由于某种原因，您_希望_测试客户端执行CSRF检查，您可以创建实施CSRF检查的测试客户端的实例。为此，在构建客户端时传递`enforce_csrf_checks`参数：

    ```
    &gt;&gt;&gt; from django.test import Client
    &gt;&gt;&gt; csrf_client = Client(enforce_csrf_checks=True)

    ```

### 提出请求

使用`django.test.Client`类发出请求。

_class_ `Client`(_enforce_csrf_checks=False_, _**defaults_)

它在建设时不需要论证。但是，您可以使用关键字参数指定一些默认标头。例如，这会在每个请求中发送`User-Agent` HTTP标头：

```
&gt;&gt;&gt; c = Client(HTTP_USER_AGENT='Mozilla/5.0')

```

传递给[`get()`](#django.test.Client.get "django.test.Client.get")，[`post()`](#django.test.Client.post "django.test.Client.post")等的`extra`关键字参数的值优先于传递给类构造函数的默认值。

`enforce_csrf_checks`参数可用于测试CSRF保护（参见上文）。

一旦您有一个`Client`实例，您就可以调用以下任何方法：

`get`(_path_, _data=None_, _follow=False_, _secure=False_, _**extra_)

New in Django 1.7:

已添加`secure`参数。

在提供的`path`上发出GET请求，并返回`Response`对象，下面将对此进行说明。

`data`字典中的键值对用于创建GET数据有效内容。例如：

```
&gt;&gt;&gt; c = Client()
&gt;&gt;&gt; c.get('/customers/details/', {'name': 'fred', 'age': 7})

```

...将导致GET请求的求值等效于：

```
/customers/details/?name=fred&age=7

```

`extra`关键字arguments参数可用于指定要在请求中发送的标头。例如：

```
&gt;&gt;&gt; c = Client()
&gt;&gt;&gt; c.get('/customers/details/', {'name': 'fred', 'age': 7},
...       HTTP_X_REQUESTED_WITH='XMLHttpRequest')

```

...将HTTP标头`HTTP_X_REQUESTED_WITH`发送到详细信息视图，这是一个测试使用[`django.http.HttpRequest.is_ajax()`](../../ref/request-response.html#django.http.HttpRequest.is_ajax "django.http.HttpRequest.is_ajax")方法的代码路径的好方法。

CGI规范

通过`**extra`发送的标头应遵循[CGI](http://www.w3.org/CGI/)规范。例如，模拟从浏览器到服务器的HTTP请求中发送的不同“主机”标头应作为`HTTP_HOST`传递。

如果您已经有以URL编码形式的GET参数，则可以使用该编码，而不是使用data参数。例如，先前的GET请求也可以被提出为：

```
&gt;&gt;&gt; c = Client()
&gt;&gt;&gt; c.get('/customers/details/?name=fred&age=7')

```

如果您提供的网址包含编码的GET数据和数据参数，则数据参数优先。

如果您将`follow`设置为`True`，则客户端将跟踪任何重定向，并且将在包含中间网址元组的响应对象中设置`redirect_chain`属性和状态代码。

如果您有重定向到`/next/`的网址`/redirect_me/`，则重定向到`/final/`，这是您会看到的：

```
&gt;&gt;&gt; response = c.get('/redirect_me/', follow=True)
&gt;&gt;&gt; response.redirect_chain
[('http://testserver/next/', 302), ('http://testserver/final/', 302)]

```

如果您将`secure`设置为`True`，则客户端将模拟HTTPS请求。

`post`(_path_, _data=None_, _content_type=MULTIPART_CONTENT_, _follow=False_, _secure=False_, _**extra_)

在提供的`path`上发出POST请求，并返回`Response`对象，这在下面进行了说明。

`data`字典中的键值对用于提交POST数据。例如：

```
&gt;&gt;&gt; c = Client()
&gt;&gt;&gt; c.post('/login/', {'name': 'fred', 'passwd': 'secret'})

```

...将导致对此URL的POST请求的评估：

```
/login/

```

...用这个POST数据：

```
name=fred&passwd=secret

```

如果您为XML有效内容提供`content_type`（例如_text / xml_），则`data`的内容将作为POST请求，使用HTTP `Content-Type`标头中的`content_type`。

如果不为`content_type`提供值，则`data`中的值将以内容类型_multipart / form-data_传输。在这种情况下，`data`中的键值对将被编码为一个多部分消息，并用于创建POST数据有效负载。

要为给定键提交多个值 - 例如，为`＆lt； select multiple&gt;`指定选择，列表或元组。例如，`data`的此值将为名为`choices`的字段提交三个选定值：

```
{'choices': ('a', 'b', 'd')}

```

提交文件是一种特殊情况。要发布文件，您只需要提供文件字段名作为键，以及要作为值上传的文件的文件句柄。例如：

```
&gt;&gt;&gt; c = Client()
&gt;&gt;&gt; with open('wishlist.doc') as fp:
...     c.post('/customers/wishes/', {'name': 'fred', 'attachment': fp})

```

（名称`attachment`这里不相关；使用您的文件处理代码所期望的任何名称。）

您还可以提供任何类似文件的对象（例如，[`StringIO`](https://docs.python.org/3/library/io.html#io.StringIO "(in Python v3.4)")或[`BytesIO`](https://docs.python.org/3/library/io.html#io.BytesIO "(in Python v3.4)")）作为文件句柄。

New in Django 1.8:

添加了使用类文件对象的能力。

请注意，如果您希望对多个`post()`调用使用相同的文件句柄，则需要在文章之间手动重置文件指针。最简单的方法是在文件提供给`post()`之后手动关闭文件，如上所示。

您还应确保以允许读取数据的方式打开文件。如果您的文件包含二进制数据，如图像，这意味着您需要以`rb`（读取二进制）模式打开该文件。

`extra`参数的作用与[`Client.get()`](#django.test.Client.get "django.test.Client.get")相同。

如果您通过POST请求的URL包含已编码的参数，则这些参数将在请求中可用。GET数据。例如，如果您提出请求：

```
&gt;&gt;&gt; c.post('/login/?visitor=true', {'name': 'fred', 'passwd': 'secret'})

```

...处理此请求的视图可以询问请求。POST以检索用户名和密码，并且可以询问请求。GET以确定用户是否是访问者。

如果您将`follow`设置为`True`，则客户端将跟踪任何重定向，并且将在包含中间网址元组的响应对象中设置`redirect_chain`属性和状态代码。

如果您将`secure`设置为`True`，则客户端将模拟HTTPS请求。

`head`(_path_, _data=None_, _follow=False_, _secure=False_, _**extra_)

在提供的`path`上执行HEAD请求，并返回`Response`对象。此方法的工作方式与[`Client.get()`](#django.test.Client.get "django.test.Client.get")一样，除了它之外，包括`follow`，`secure`和`extra`

`options`(_path_, _data=''_, _content_type='application/octet-stream'_, _follow=False_, _secure=False_, _**extra_)

在提供的`path`上执行OPTIONS请求，并返回`Response`对象。用于测试RESTful接口。

当提供`data`时，它用作请求主体，并且`Content-Type`头设置为`content_type`。

`follow`，`secure`和`extra`参数的作用与[`Client.get()`](#django.test.Client.get "django.test.Client.get")相同。

`put`(_path_, _data=''_, _content_type='application/octet-stream'_, _follow=False_, _secure=False_, _**extra_)

在提供的`path`上发出PUT请求，并返回`Response`对象。用于测试RESTful接口。

当提供`data`时，它用作请求主体，并且`Content-Type`头设置为`content_type`。

`follow`，`secure`和`extra`参数的作用与[`Client.get()`](#django.test.Client.get "django.test.Client.get")相同。

`patch`(_path_, _data=''_, _content_type='application/octet-stream'_, _follow=False_, _secure=False_, _**extra_)

在提供的`path`上发出PATCH请求，并返回`Response`对象。用于测试RESTful接口。

`follow`，`secure`和`extra`参数的作用与[`Client.get()`](#django.test.Client.get "django.test.Client.get")相同。

`delete`(_path_, _data=''_, _content_type='application/octet-stream'_, _follow=False_, _secure=False_, _**extra_)

在提供的`path`上发出DELETE请求，并返回`Response`对象。用于测试RESTful接口。

当提供`data`时，它用作请求主体，并且`Content-Type`头设置为`content_type`。

`follow`，`secure`和`extra`参数的作用与[`Client.get()`](#django.test.Client.get "django.test.Client.get")相同。

`trace`(_path_, _follow=False_, _secure=False_, _**extra_)

New in Django 1.8.

在提供的`path`上执行TRACE请求，并返回`Response`对象。用于模拟诊断探头。

与其他请求方法不同，为了符合 [**RFC 2616**](http://tools.ietf.org/html/rfc2616.html)，`data`不作为关键字参数提供，这要求TRACE请求不应有一个实体体。

`follow`，`secure`和`extra`参数的作用与[`Client.get()`](#django.test.Client.get "django.test.Client.get")相同。

`login`(_**credentials_)

如果您的网站使用Django的[_authentication system_](../auth/index.html)，并且您处理用户登录，则可以使用测试客户端的`login()`方法来模拟用户登录网站的效果。

调用此方法后，测试客户端将拥有通过任何可能构成视图一部分的基于登录的测试所需的所有Cookie和会话数据。

`credentials`参数的格式取决于您使用的[_authentication backend_](../auth/customizing.html#authentication-backends)（由您的[`AUTHENTICATION_BACKENDS`](../../ref/settings.html#std:setting-AUTHENTICATION_BACKENDS)设置配置）。如果您使用的是由Django提供的标准认证后端（`ModelBackend`），则`credentials`应该是用户的用户名和密码，

```
&gt;&gt;&gt; c = Client()
&gt;&gt;&gt; c.login(username='fred', password='secret')

# Now you can access a view that's only available to logged-in users.

```

如果您使用的是其他身份验证后端，则此方法可能需要不同的凭据。它需要您的后端的`authenticate()`方法需要的凭据。

`login()` returns `True` if it the credentials were accepted and login was successful.

最后，您需要记住创建用户帐户，然后才能使用此方法。如上所述，测试运行器是使用测试数据库执行的，默认情况下不包含用户。因此，在生产站点上有效的用户帐户将无法在测试条件下工作。您需要创建用户作为测试套件的一部分 - 手动（使用Django模型API）或测试夹具。请记住，如果您希望测试用户拥有密码，则不能通过直接设置password属性来设置用户的密码 - 您必须使用[`set_password()`](../../ref/contrib/auth.html#django.contrib.auth.models.User.set_password "django.contrib.auth.models.User.set_password")函数来存储正确的散列密码。或者，您可以使用[`create_user()`](../../ref/contrib/auth.html#django.contrib.auth.models.UserManager.create_user "django.contrib.auth.models.UserManager.create_user")助手方法创建具有正确散列密码的新用户。

`logout`()

如果您的网站使用Django的[_authentication system_](../auth/index.html)，则可以使用`logout()`方法模拟用户从您的网站注销的效果。

调用此方法后，测试客户端将所有Cookie和会话数据清除为默认值。后续请求将显示为来自[`AnonymousUser`](../../ref/contrib/auth.html#django.contrib.auth.models.AnonymousUser "django.contrib.auth.models.AnonymousUser")。

### 测试响应

`get()`和`post()`方法都会返回`Response`对象。此`Response`对象_不是_与Django视图返回的`HttpResponse`对象相同；测试响应对象具有一些对于测试代码验证有用的附加数据。

具体来说，`Response`对象具有以下属性：

_class_ `Response`

`client`

用于生成导致响应的请求的测试客户端。

`content`

响应的主体，作为字符串。这是视图呈现的最终页面内容，或任何错误消息。

`context`

用于呈现产生响应内容的模板的模板`Context`实例。

如果呈现的页面使用多个模板，则`context`将是`Context`对象的列表，按照它们的呈现顺序。

无论渲染期间使用的模板数量如何，都可以使用`[]`运算符检索上下文值。例如，可以使用以下方式检索上下文变量`name`：

```
&gt;&gt;&gt; response = client.get('/foo/')
&gt;&gt;&gt; response.context['name']
'Arthur'

```

`request`

刺激响应的请求数据。

`wsgi_request`

New in Django 1.7.

由生成响应的测试处理程序生成的`WSGIRequest`实例。

`status_code`

响应的HTTP状态，作为整数。有关HTTP状态代码的完整列表，请参见 [**RFC 2616**](http://tools.ietf.org/html/rfc2616.html#section-10)。

`templates`

用于渲染最终内容的`Template`实例列表，按渲染顺序排列。对于列表中的每个模板，如果从文件加载模板，请使用`template.name`获取模板的文件名。（名称是一个字符串，例如`'admin/index.html'`。）

`resolver_match`

New in Django 1.8:

响应的实例[`ResolverMatch`](../../ref/urlresolvers.html#django.core.urlresolvers.ResolverMatch "django.core.urlresolvers.ResolverMatch")。例如，您可以使用[`func`](../../ref/urlresolvers.html#django.core.urlresolvers.ResolverMatch.func "django.core.urlresolvers.ResolverMatch.func")属性验证提供响应的视图：

```
# my_view here is a function based view
self.assertEqual(response.resolver_match.func, my_view)

# class based views need to be compared by name, as the functions
# generated by as_view() won't be equal
self.assertEqual(response.resolver_match.func.__name__, MyView.as_view().__name__)

```

如果找不到给定的URL，访问此属性将引发[`Resolver404`](../../ref/exceptions.html#django.core.urlresolvers.Resolver404 "django.core.urlresolvers.Resolver404")异常。

您还可以在响应对象上使用字典语法查询HTTP标头中的任何设置的值。例如，您可以使用`response['Content-Type']`确定响应的内容类型。

### 例外

如果将测试客户端指向引发异常的视图，那么该异常将在测试用例中可见。然后，您可以使用标准`尝试 ...`除了块或[`assertRaises()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertRaises "(in Python v3.4)")来测试异常。

对测试客户端不可见的唯一例外是[`Http404`](../http/views.html#django.http.Http404 "django.http.Http404")，[`PermissionDenied`](../../ref/exceptions.html#django.core.exceptions.PermissionDenied "django.core.exceptions.PermissionDenied")，[`SystemExit`](https://docs.python.org/3/library/exceptions.html#SystemExit "(in Python v3.4)")和[`SuspiciousOperation`](../../ref/exceptions.html#django.core.exceptions.SuspiciousOperation "django.core.exceptions.SuspiciousOperation")。Django在内部捕获这些异常并将它们转换为适当的HTTP响应代码。在这些情况下，您可以在测试中检查`response.status_code`。

### 持久状态

测试客户端是有状态的。如果响应返回cookie，那么该cookie将存储在测试客户端中，并与所有后续的`get()`和`post()`请求一起发送。

不遵循这些cookie的过期政策。如果您希望Cookie过期，请手动删除或创建新的`Client`实例（这将有效删除所有Cookie）。

测试客户机具有存储持久状态信息的两个属性。您可以作为测试条件的一部分访问这些属性。

`Client.``cookies`

Python [`SimpleCookie`](https://docs.python.org/3/library/http.cookies.html#http.cookies.SimpleCookie "(in Python v3.4)")对象，包含所有客户端Cookie的当前值。有关更多信息，请参阅[`http.cookies`](https://docs.python.org/3/library/http.cookies.html#module-http.cookies "(in Python v3.4)")模块的文档。

`Client.``session`

包含会话信息的类字典对象。有关详细信息，请参阅[_session documentation_](../http/sessions.html)。

要修改会话然后保存它，它必须首先存储在变量中（因为每次访问此属性时都会创建一个新的`SessionStore`）：

```
def test_something(self):
    session = self.client.session
    session['somekey'] = 'test'
    session.save()

```

### 例

以下是使用测试客户端的简单单元测试：

```
import unittest
from django.test import Client

class SimpleTest(unittest.TestCase):
    def setUp(self):
        # Every test needs a client.
        self.client = Client()

    def test_details(self):
        # Issue a GET request.
        response = self.client.get('/customer/details/')

        # Check that the response is 200 OK.
        self.assertEqual(response.status_code, 200)

        # Check that the rendered context contains 5 customers.
        self.assertEqual(len(response.context['customers']), 5)

```

也可以看看

[`django.test.RequestFactory`](advanced.html#django.test.RequestFactory "django.test.RequestFactory")

## 提供测试用例类

正常的Python单元测试类扩展了[`unittest.`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "(in Python v3.4)")测试用例。Django提供了这个基类的一些扩展：

[![Hierarchy of Django unit testing classes (TestCase subclasses)](https://docs.djangoproject.com/en/1.8/_images/django_unittest_classes_hierarchy.svg)](https://docs.djangoproject.com/en/1.8/_images/django_unittest_classes_hierarchy.svg)

Django单元测试类的层次结构

### SimpleTestCase

_class_ `SimpleTestCase`

[`unittest.`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "(in Python v3.4)")TestCase，它扩展它与一些基本功能，如：

*   保存和恢复Python警告机制状态。
*   Some useful assertions like:
    *   Checking that a callable [](#django.test.SimpleTestCase.assertRaisesMessage "django.test.SimpleTestCase.assertRaisesMessage").
    *   测试表单字段[`rendering and error treatment`](#django.test.SimpleTestCase.assertFieldOutput "django.test.SimpleTestCase.assertFieldOutput")。
    *   测试[`HTML responses for the presence/lack of a given fragment`](#django.test.SimpleTestCase.assertContains "django.test.SimpleTestCase.assertContains")。
    *   验证模板[`has/hasn't been used to generate a given response content`](#django.test.SimpleTestCase.assertTemplateUsed "django.test.SimpleTestCase.assertTemplateUsed")。
    *   验证HTTP [`redirect`](#django.test.SimpleTestCase.assertRedirects "django.test.SimpleTestCase.assertRedirects")是由应用执行的。
    *   稳健地测试两个[`HTML fragments`](#django.test.SimpleTestCase.assertHTMLEqual "django.test.SimpleTestCase.assertHTMLEqual")的等式/不等式或[`containment`](#django.test.SimpleTestCase.assertInHTML "django.test.SimpleTestCase.assertInHTML")。
    *   稳健地测试两个[`XML fragments`](#django.test.SimpleTestCase.assertXMLEqual "django.test.SimpleTestCase.assertXMLEqual")的相等/不等。
    *   稳健地测试两个[`JSON fragments`](#django.test.SimpleTestCase.assertJSONEqual "django.test.SimpleTestCase.assertJSONEqual")的相等性。
*   使用[_modified settings_](#overriding-settings)运行测试的能力。
*   使用[`client`](#django.test.SimpleTestCase.client "django.test.SimpleTestCase.client") [`Client`](#django.test.Client "django.test.Client")。
*   自定义测试时间[`URL maps`](#django.test.SimpleTestCase.urls "django.test.SimpleTestCase.urls")。

如果您需要任何其他更复杂和重量级的Django特定功能，如：

*   测试或使用ORM。
*   数据库[`fixtures`](#django.test.TransactionTestCase.fixtures "django.test.TransactionTestCase.fixtures")。
*   根据数据库后端功能测试[_skipping based on database backend features_](#skipping-tests)。
*   其余的专用[`assert*`](#django.test.TransactionTestCase.assertQuerysetEqual "django.test.TransactionTestCase.assertQuerysetEqual")方法。

那么您应该改用[`TransactionTestCase`](#django.test.TransactionTestCase "django.test.TransactionTestCase")或[`TestCase`](#django.test.TestCase "django.test.TestCase")。

`SimpleTestCase`继承自`unittest.`测试用例。

警告

`SimpleTestCase`及其子类（例如`TestCase`，...）依赖于`setUpClass()`和`tearDownClass()`执行一些类的初始化（例如覆盖设置）。如果需要重写这些方法，不要忘记调用`super`实现：

```
class MyTestCase(TestCase):

    @classmethod
    def setUpClass(cls):
        super(MyTestCase, cls).setUpClass()     # Call parent first
        ...

    @classmethod
    def tearDownClass(cls):
        ...
        super(MyTestCase, cls).tearDownClass()  # Call parent last

```

### TransactionTestCase

_class_ `TransactionTestCase`

Django的`TestCase`类（如下所述）利用数据库事务工具来加快在每个测试开始时将数据库重置为已知状态的过程。然而，这样做的一个后果是，一些数据库行为不能在Django `TestCase`类中测试。例如，您不能测试一个代码块是否在事务中执行，如使用[`select_for_update()`](../../ref/models/querysets.html#django.db.models.query.QuerySet.select_for_update "django.db.models.query.QuerySet.select_for_update")时所需。在这些情况下，您应该使用`TransactionTestCase`。

Changed in Django 1.8:

在旧版本的Django中，事务提交和回滚的影响无法在`TestCase`中测试。随着Django 1.8中旧式事务管理的弃用循环的完成，在`TestCase`中不再禁用事务管理命令（例如`transaction.commit()`）。

`TransactionTestCase`和`TestCase`是完全相同的，除了数据库重置为已知状态的方式以及测试代码测试提交和回滚效果的能力：

*   通过截断所有表，测试运行后，`TransactionTestCase`会重置数据库。`TransactionTestCase`可以调用提交和回滚，并观察这些调用对数据库的影响。
*   另一方面，`TestCase`在测试后不会截断表。相反，它将测试代码包含在测试结束时回滚的数据库事务中。这保证在测试结束时的回滚将数据库恢复到其初始状态。

警告

在不支持回滚的数据库上运行的`TestCase`（例如，具有MyISAM存储引擎的MySQL）以及`TransactionTestCase`的所有实例将在测试结束时回滚从测试数据库中删除所有数据，并重新加载应用程序的初始数据而不进行迁移。

迁移[_will not see their data reloaded_](overview.html#test-case-serialized-rollback)；如果您需要此功能（例如，第三方应用应启用此功能），您可以设置`serialized_rollback = True t0 &gt;在`TestCase`内。`

`TransactionTestCase`继承自[`SimpleTestCase`](#django.test.SimpleTestCase "django.test.SimpleTestCase")。

### 测试用例

_class_ `TestCase`

这个类提供了一些额外的功能，可以用来测试网站

正常转换[`unittest.`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "(in Python v3.4)")TestCase到Django [`TestCase`](#django.test.TestCase "django.test.TestCase")很简单：只需从`'unittest.`TestCase'to `'django.test.TestCase'`。所有标准的Python单元测试功能将继续可用，但将增加一些有用的补充，包括：

*   自动加载夹具。
*   将测试包含在两个嵌套的`atomic`块中：一个用于整个类，一个用于每个测试。
*   创建一个TestClient实例。
*   Django特定的断言用于测试重定向和形式错误。

_classmethod_ `TestCase.``setUpTestData`()

New in Django 1.8.

上述类级别`atomic`块允许在类级别创建初始数据，对于整个`TestCase`一次。与使用`setUp()`相比，此技术允许更快的测试。

例如：

```
from django.test import TestCase

class MyTests(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Set up data for the whole TestCase
        cls.foo = Foo.objects.create(bar="Test")
        ...

    def test1(self):
        # Some test using self.foo
        ...

    def test2(self):
        # Some other test using self.foo
        ...

```

注意，如果测试在没有事务支持的数据库上运行（例如，使用MyISAM引擎的MySQL），则在每次测试之前将调用`setUpTestData()`，否定速度优势。

警告

如果要测试一些特定的数据库事务行为，应该使用`TransactionTestCase`作为`TestCase`在[`atomic()`](../db/transactions.html#django.db.transaction.atomic "django.db.transaction.atomic")块中测试执行。

`TestCase`继承自[`TransactionTestCase`](#django.test.TransactionTestCase "django.test.TransactionTestCase")。

### LiveServerTestCase

_class_ `LiveServerTestCase`

`LiveServerTestCase`基本上与[`TransactionTestCase`](#django.test.TransactionTestCase "django.test.TransactionTestCase")相同，具有一个额外的功能：它在设置的后台启动一个活动的Django服务器，并在拆卸时将其关闭。这允许使用除了[_Django dummy client_](#test-client)（例如，[Selenium](http://seleniumhq.org/)客户端）之外的自动测试客户端，在浏览器中执行一系列功能测试，并模拟真实用户的操作。

默认情况下，活动服务器的地址为`'localhost:8081'`，在测试期间可以使用`self.live_server_url`访问完整的URL。If you’d like to change the default address (in the case, for example, where the 8081 port is already taken) then you may pass a different one to the [`test`](../../ref/django-admin.html#django-admin-test) command via the [`--liveserver`](../../ref/django-admin.html#django-admin-option---liveserver) option, for example:

```
$ ./manage.py test --liveserver=localhost:8082

```

更改默认服务器地址的另一种方法是在代码中某处设置&lt;cite&gt;DJANGO_LIVE_TEST_SERVER_ADDRESS&lt;/cite&gt;环境变量（例如，在[_custom test runner_](advanced.html#topics-testing-test-runner)中）：

```
import os
os.environ['DJANGO_LIVE_TEST_SERVER_ADDRESS'] = 'localhost:8082'

```

在测试由多个进程并行运行的情况下（例如，在几个并发的[连续集成](http://en.wikipedia.org/wiki/Continuous_integration)构建的上下文中），进程将竞争同一个地址，因此您的测试可能随机失败并显示“地址已在使用中”错误。为了避免此问题，您可以传递逗号分隔的端口列表或端口范围（至少与潜在的并行进程数一样多）。例如：

```
$ ./manage.py test --liveserver=localhost:8082,8090-8100,9000-9200,7041

```

然后，在测试执行期间，每个新的活测试服务器将尝试每个指定的端口，直到找到一个可用的并且接受它。

为了演示如何使用,让我们编写一个简单的Selenium测试。 首先呢,你需要用pip命令安装 [selenium package](https://pypi.python.org/pypi/selenium) 到你的Python路径里面:

```
$ pip install selenium

```

然后，向应用程序的测试模块添加`LiveServerTestCase`测试（例如：`myapp/tests.py`）。此测试的代码可能如下所示：

```
from django.test import LiveServerTestCase
from selenium.webdriver.firefox.webdriver import WebDriver

class MySeleniumTests(LiveServerTestCase):
    fixtures = ['user-data.json']

    @classmethod
    def setUpClass(cls):
        super(MySeleniumTests, cls).setUpClass()
        cls.selenium = WebDriver()

    @classmethod
    def tearDownClass(cls):
        cls.selenium.quit()
        super(MySeleniumTests, cls).tearDownClass()

    def test_login(self):
        self.selenium.get('%s%s' % (self.live_server_url, '/login/'))
        username_input = self.selenium.find_element_by_name("username")
        username_input.send_keys('myuser')
        password_input = self.selenium.find_element_by_name("password")
        password_input.send_keys('secret')
        self.selenium.find_element_by_xpath('//input[@value="Log in"]').click()

```

最后，您可以运行测试如下：

```
$ ./manage.py test myapp.tests.MySeleniumTests.test_login

```

此示例将自动打开Firefox，然后转到登录页面，输入凭据并按“登录”按钮。Selenium提供其他驱动程序，以防您没有安装Firefox或希望使用其他浏览器。上面的例子只是Selenium客户端可以做的一小部分；有关详细信息，请参阅[完整参考](http://selenium-python.readthedocs.org/en/latest/api.html)。

Changed in Django 1.7:

在旧版本中，`LiveServerTestCase`依赖[_staticfiles contrib app_](../../howto/static-files/index.html)在测试执行期间透明地提供静态文件。此功能已移至[`StaticLiveServerTestCase`](../../ref/contrib/staticfiles.html#django.contrib.staticfiles.testing.StaticLiveServerTestCase "django.contrib.staticfiles.testing.StaticLiveServerTestCase")子类，因此如果您需要[_the original behavior_](../../howto/static-files/index.html#staticfiles-testing-support)，请使用该子类。

`LiveServerTestCase`现在只需在[`STATIC_ROOT`](../../ref/settings.html#std:setting-STATIC_ROOT)下的[`STATIC_URL`](../../ref/settings.html#std:setting-STATIC_URL)发布文件系统的内容。

注意

当使用内存中的SQLite数据库来运行测试时，同一数据库连接将由两个并行线程共享：运行活动服务器的线程和运行测试用例的线程。重要的是防止两个线程通过这个共享连接同时进行数据库查询，因为这可能会随机导致测试失败。所以你需要确保这两个线程不会同时访问数据库。特别是，这意味着在某些情况下（例如，在单击链接或提交表单之后），您可能需要检查Selenium是否收到响应，并且在继续执行进一步的测试之前加载下一页。例如，通过使Selenium等待，直到在响应中找到`&lt;body&gt;` HTML标记（需要Selenium&gt; 2.13）：

```
def test_login(self):
    from selenium.webdriver.support.wait import WebDriverWait
    timeout = 2
    ...
    self.selenium.find_element_by_xpath('//input[@value="Log in"]').click()
    # Wait until the response is received
    WebDriverWait(self.selenium, timeout).until(
        lambda driver: driver.find_element_by_tag_name('body'))

```

这里的棘手的事情是，真的没有一个“页面加载”，尤其是在现代Web应用程序中，在服务器生成初始文档后动态生成HTML。因此，简单地检查响应中是否存在`&lt;body&gt;`可能不一定适用于所有用例。有关详细信息，请参阅[Selenium常见问题](http://code.google.com/p/selenium/wiki/FrequentlyAskedQuestions#Q:_WebDriver_fails_to_find_elements_/_Does_not_block_on_page_loa)和[Selenium文档](http://seleniumhq.org/docs/04_webdriver_advanced.html#explicit-waits)。

## 测试用例特性

### 默认测试客户端

`SimpleTestCase.``client`

`django.test.*TestCase`实例中的每个测试用例都可以访问Django测试客户端的实例。此客户端可以作为`self.client`访问。每个测试都重新创建此客户端，因此您不必担心从一个测试到另一个测试的状态（例如Cookie）。

这意味着，不是在每个测试中实例化`Client`：

```
import unittest
from django.test import Client

class SimpleTest(unittest.TestCase):
    def test_details(self):
        client = Client()
        response = client.get('/customer/details/')
        self.assertEqual(response.status_code, 200)

    def test_index(self):
        client = Client()
        response = client.get('/customer/index/')
        self.assertEqual(response.status_code, 200)

```

...你可以参考`self.client`，像这样：

```
from django.test import TestCase

class SimpleTest(TestCase):
    def test_details(self):
        response = self.client.get('/customer/details/')
        self.assertEqual(response.status_code, 200)

    def test_index(self):
        response = self.client.get('/customer/index/')
        self.assertEqual(response.status_code, 200)

```

### 自定义测试客户端

`SimpleTestCase.``client_class`

如果要使用不同的`Client`类（例如，具有自定义行为的子类），请使用[`client_class`](#django.test.SimpleTestCase.client_class "django.test.SimpleTestCase.client_class")类属性：

```
from django.test import TestCase, Client

class MyTestClient(Client):
    # Specialized methods for your environment
    ...

class MyTest(TestCase):
    client_class = MyTestClient

    def test_my_stuff(self):
        # Here self.client is an instance of MyTestClient...
        call_some_test_code()

```

### 夹具装载

`TransactionTestCase.``fixtures`

如果数据库中没有任何数据，则对于数据库支持的Web站点的测试用例没有多大用处。为了方便将测试数据放入数据库，Django的自定义`TransactionTestCase`类提供了加载**fixtures**的方法。

fixture是Django知道如何导入到数据库中的数据集合。例如，如果您的网站有用户帐户，您可能会设置一个假的用户帐户，以便在测试期间填充您的数据库。

创建fixture的最直接的方法是使用[`manage.py dumpdata`](../../ref/django-admin.html#django-admin-dumpdata)命令。这假定您的数据库中已经有一些数据。有关详细信息，请参阅[`dumpdata documentation`](../../ref/django-admin.html#django-admin-dumpdata)。

注意

如果您曾经运行过[`manage.py migrate`](../../ref/django-admin.html#django-admin-migrate)，您已经使用了一个甚至不知道它的灯具！当您第一次在数据库中调用[`migrate`](../../ref/django-admin.html#django-admin-migrate)时，Django会安装一个名为`initial_data`的夹具。这为您提供了一种使用任何初始数据填充新数据库的方法，例如默认的一组类别。

可以使用[`manage.py loaddata`](../../ref/django-admin.html#django-admin-loaddata)命令手动安装具有其他名称的装置。

初始SQL数据和测试

Django提供了将初始数据插入模型的第二种方法 - [_custom SQL hook_](../../howto/initial-data.html#initial-sql)。然而，该技术_不能用于提供用于测试目的的初始数据。_Django的测试框架在每次测试后刷新测试数据库的内容；因此，使用自定义SQL钩​​子添加的任何数据都将丢失。

Once you’ve created a fixture and placed it in a `fixtures` directory in one of your [`INSTALLED_APPS`](../../ref/settings.html#std:setting-INSTALLED_APPS), you can use it in your unit tests by specifying a `fixtures` class attribute on your [`django.test.TestCase`](#django.test.TestCase "django.test.TestCase") subclass:

```
from django.test import TestCase
from myapp.models import Animal

class AnimalTestCase(TestCase):
    fixtures = ['mammals.json', 'birds']

    def setUp(self):
        # Test definitions as before.
        call_setup_methods()

    def testFluffyAnimals(self):
        # A test that uses the fixtures.
        call_some_test_code()

```

这里具体是什么会发生：

*   在每个测试用例开始时，在运行`setUp()`之前，Django将刷新数据库，将数据库返回到调用[`migrate`](../../ref/django-admin.html#django-admin-migrate)之后的状态。
*   然后，安装所有命名的夹具。在这个例子中，Django将安装任何名为`mammals`的JSON夹具，然后安装任何名为`birds`的夹具。有关定义和安装灯具的更多详细信息，请参阅[`loaddata`](../../ref/django-admin.html#django-admin-loaddata)文档。

对测试用例中的每个测试重复这个刷新/装载过程，因此您可以确定测试的结果不会受到另一个测试或测试执行的顺序的影响。

默认情况下，fixture仅加载到`default`数据库中。如果您使用多个数据库并设置[`multi_db=True`](#django.test.TransactionTestCase.multi_db "django.test.TransactionTestCase.multi_db")，fixture将被加载到所有数据库。

### URLconf配置

`SimpleTestCase.``urls`

自1.8版起已弃用：请改用URLconf配置`@override_settings(ROOT_URLCONF=...)`。

如果您的应用程序提供了视图，您可能需要包括使用测试客户端来执行这些视图的测试。但是，最终用户可以在自己选择的任何URL上自由地在应用程序中部署视图。

为了为测试提供可靠的URL空间，`django.test.*TestCase`类提供了在测试套件执行期间自定义URLconf配置的功能。如果您的`*TestCase`实例定义了`urls`属性，则`*TestCase`将使用该属性的值作为[`ROOT_URLCONF`](../../ref/settings.html#std:setting-ROOT_URLCONF)

例如：

```
from django.test import TestCase

class TestMyViews(TestCase):
    urls = 'myapp.test_urls'

    def testIndexPageView(self):
        # Here you'd test your view using ``Client``.
        call_some_test_code()

```

此测试用例将使用`myapp.test_urls`的内容作为测试用例持续时间的URLconf。

### 多数据库支持

`TransactionTestCase.``multi_db`

Django设置与设置文件中的[`DATABASES`](../../ref/settings.html#std:setting-DATABASES)定义中定义的每个数据库对应的测试数据库。但是，运行Django TestCase所需的大部分时间由调用`flush`消耗，确保在每次测试运行开始时都有一个干净的数据库。如果您有多个数据库，则需要多次刷新（每个数据库一次），这可能是一个耗时的活动 - 特别是如果您的测试不需要测试多数据库活动。

作为优化，Django仅在每次测试运行开始时刷新`default`数据库。如果您的设置包含多个数据库，并且测试需要每个数据库都是干净的，则可以使用测试套件上的`multi_db`属性请求完全刷新。

例如：

```
class TestMyViews(TestCase):
    multi_db = True

    def testIndexPageView(self):
        call_some_test_code()

```

此测试用例将在运行`testIndexPageView`之前刷新_所有_测试数据库。

`multi_db`标志还影响attr：&lt;cite&gt;TransactionTestCase.fixtures&lt;/cite&gt;加载到哪些数据库中。默认情况下（当`multi_db=False`时），fixture仅加载到`default`数据库中。如果`multi_db=True`，fixture将加载到所有数据库中。

### 覆盖设置

警告

使用以下功能临时更改测试中的设置值。不要直接操作`django.conf.settings`，因为Django在这种操作后不会恢复原始值。

`SimpleTestCase.``settings`()

为了测试目的，在运行测试代码之后临时更改设置并恢复为原始值通常很有用。对于这种用例，Django提供了一个标准的Python上下文管理器（参见 [**PEP 343**](http://www.python.org/dev/peps/pep-0343)），名为[`settings()`](#django.test.SimpleTestCase.settings "django.test.SimpleTestCase.settings")

```
from django.test import TestCase

class LoginTestCase(TestCase):

    def test_login(self):

        # First check for the default behavior
        response = self.client.get('/sekrit/')
        self.assertRedirects(response, '/accounts/login/?next=/sekrit/')

        # Then override the LOGIN_URL setting
        with self.settings(LOGIN_URL='/other/login/'):
            response = self.client.get('/sekrit/')
            self.assertRedirects(response, '/other/login/?next=/sekrit/')

```

此示例将覆盖`with`块中的代码的[`LOGIN_URL`](../../ref/settings.html#std:setting-LOGIN_URL)设置，然后将其值重置为之前的状态。

`SimpleTestCase.``modify_settings`()

New in Django 1.7.

它可以证明难以重新定义包含值列表的设置。在实践中，添加或删除值通常就足够了。[`modify_settings()`](#django.test.SimpleTestCase.modify_settings "django.test.SimpleTestCase.modify_settings")上下文管理器使其变得容易：

```
from django.test import TestCase

class MiddlewareTestCase(TestCase):

    def test_cache_middleware(self):
        with self.modify_settings(MIDDLEWARE_CLASSES={
            'append': 'django.middleware.cache.FetchFromCacheMiddleware',
            'prepend': 'django.middleware.cache.UpdateCacheMiddleware',
            'remove': [
                'django.contrib.sessions.middleware.SessionMiddleware',
                'django.contrib.auth.middleware.AuthenticationMiddleware',
                'django.contrib.messages.middleware.MessageMiddleware',
            ],
        }):
            response = self.client.get('/')
            # ...

```

对于每个操作，您可以提供值列表或字符串。当该值已经存在于列表中时，`append`和`prepend`不起作用；当值不存在时，`remove`。

`override_settings`()

如果要覆盖测试方法的设置，Django提供[`override_settings()`](#django.test.override_settings "django.test.override_settings")装饰器（请参阅 [**PEP 318**](http://www.python.org/dev/peps/pep-0318)）。它的使用像这样：

```
from django.test import TestCase, override_settings

class LoginTestCase(TestCase):

    @override_settings(LOGIN_URL='/other/login/')
    def test_login(self):
        response = self.client.get('/sekrit/')
        self.assertRedirects(response, '/other/login/?next=/sekrit/')

```

装饰器也可以应用于[`TestCase`](#django.test.TestCase "django.test.TestCase")类：

```
from django.test import TestCase, override_settings

@override_settings(LOGIN_URL='/other/login/')
class LoginTestCase(TestCase):

    def test_login(self):
        response = self.client.get('/sekrit/')
        self.assertRedirects(response, '/other/login/?next=/sekrit/')

```

Changed in Django 1.7:

以前，`override_settings`是从`django.test.utils`导入的。

`modify_settings`()

New in Django 1.7.

同样，Django提供[`modify_settings()`](#django.test.modify_settings "django.test.modify_settings")装饰器：

```
from django.test import TestCase, modify_settings

class MiddlewareTestCase(TestCase):

    @modify_settings(MIDDLEWARE_CLASSES={
        'append': 'django.middleware.cache.FetchFromCacheMiddleware',
        'prepend': 'django.middleware.cache.UpdateCacheMiddleware',
    })
    def test_cache_middleware(self):
        response = self.client.get('/')
        # ...

```

装饰器也可以应用于测试用例类：

```
from django.test import TestCase, modify_settings

@modify_settings(MIDDLEWARE_CLASSES={
    'append': 'django.middleware.cache.FetchFromCacheMiddleware',
    'prepend': 'django.middleware.cache.UpdateCacheMiddleware',
})
class MiddlewareTestCase(TestCase):

    def test_cache_middleware(self):
        response = self.client.get('/')
        # ...

```

注意

当给定一个类时，这些装饰器直接修改类并返回它；他们不创建并返回它的修改副本。因此，如果您尝试调整上述示例以将返回值分配给不同于`LoginTestCase`或`MiddlewareTestCase`的名称，您可能会惊讶地发现原始测试用例类仍然同样受到装饰师的影响。对于给定类，[`modify_settings()`](#django.test.modify_settings "django.test.modify_settings")始终应用于[`override_settings()`](#django.test.override_settings "django.test.override_settings")之后。

警告

设置文件包含一些仅在Django内部初始化期间参考的设置。如果使用`override_settings`更改这些设置，如果通过`django.conf.settings`模块访问它，设置会更改，但是Django的内部访问方式不同。有效地，使用这些设置使用[`override_settings()`](#django.test.override_settings "django.test.override_settings")或[`modify_settings()`](#django.test.modify_settings "django.test.modify_settings")可能不会做你期望做的。

我们不建议更改[`DATABASES`](../../ref/settings.html#std:setting-DATABASES)设置。改变[`CACHES`](../../ref/settings.html#std:setting-CACHES)设置是可能的，但如果你使用使用缓存的内部，如[`django.contrib.sessions`](../http/sessions.html#module-django.contrib.sessions "django.contrib.sessions: Provides session management for Django projects.")，有点棘手。例如，您必须在使用缓存会话并覆盖[`CACHES`](../../ref/settings.html#std:setting-CACHES)的测试中重新初始化会话后端。

最后，避免将您的设置作为模块级常量别名，因为`override_settings()`将不适用于这些值，因为它们仅在首次导入模块时进行评估。

您还可以在设置被覆盖后通过删除设置来模拟缺少设置，如下所示：

```
@override_settings()
def test_something(self):
    del settings.LOGIN_URL
    ...

```

Changed in Django 1.7:

以前，您只能模拟删除显式覆盖的设置。

覆盖设置时，请确保处理您的应用代码使用缓存或类似功能的情况，即使设置更改也保留状态。Django提供了[`django.test.signals.setting_changed`](../../ref/signals.html#django.test.signals.setting_changed "django.test.signals.setting_changed")信号，允许您注册回调以清除设置，否则在更改设置时重置状态。

Django本身使用这个信号来重置各种数据：


| Overridden settings | Data reset |
| --- | --- |
| USE_TZ, TIME_ZONE | Databases timezone |
| TEMPLATES | Template engines |
| SERIALIZATION_MODULES | Serializers cache |
| LOCALE_PATHS, LANGUAGE_CODE | Default translation and loaded translations |
| MEDIA_ROOT, DEFAULT_FILE_STORAGE | Default file storage |

### 清空测试发件箱

如果您使用任何Django的自定义`TestCase`类，测试运行器将在每个测试用例开始时清除测试电子邮件发件箱的内容。

有关测试期间电子邮件服务的详细信息，请参阅下面的[电子邮件服务](#email-services)。

### 断言

作为Python的正常[`unittest.`](https://docs.python.org/3/library/unittest.html#unittest.TestCase "(in Python v3.4)")TestCase类实现诸如[`assertTrue()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertTrue "(in Python v3.4)")和[`assertEqual()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertEqual "(in Python v3.4)")之类的断言方法，Django的自定义[`TestCase`](#django.test.TestCase "django.test.TestCase")类提供了许多有用的自定义断言方法用于测试Web应用程序：

大多数断言方法给出的失败消息可以使用`msg_prefix`参数定制。此字符串将作为断言生成的任何失败消息的前缀。这允许您提供其他详细信息，以帮助您确定测试套件中的故障位置和原因。

`SimpleTestCase.``assertRaisesMessage`(_expected_exception_, _expected_message_, _callable_obj=None_, _*args_, _**kwargs_)

认为执行可调用`callable_obj`引发了`expected_exception`异常，并且此异常具有`expected_message`表示。任何其他结果报告为失败。类似于unittest的[`assertRaisesRegex()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertRaisesRegex "(in Python v3.4)")，区别在于`expected_message`不是正则表达式。

`SimpleTestCase.``assertFieldOutput`(_fieldclass_, _valid_, _invalid_, _field_args=None_, _field_kwargs=None_, _empty_value=''_)

断言表单字段在各种输入中正确运行。

Parameters:

*   **fieldclass** - 要测试的字段的类。
*   **valid** - 将有效输入映射到其预期清除值的字典。
*   **invalid** - 将无效输入映射到一个或多个引发的错误消息的字典。
*   **field_args** - 传递给实例化字段的arg。
*   **field_kwargs** - 传递给实例化字段的kwargs。
*   **empty_value** - `empty_values`中输入的预期干净输出。


例如，以下代码测试`EmailField`接受`a@a.com`作为有效的电子邮件地址，但拒绝具有合理错误的`aaa`信息：

```
self.assertFieldOutput(EmailField, {'a@a.com': 'a@a.com'}, {'aaa': ['Enter a valid email address.']})

```

`SimpleTestCase.``assertFormError`(_response_, _form_, _field_, _errors_, _msg_prefix=''_)

断言窗体上的字段在表单上呈现时会提供所提供的错误列表。

`form`是在模板上下文中给出的`Form`实例的名称。

`field`是要检查的表单上的字段的名称。如果`field`的值为`None`，则将检查非字段错误（可通过[`form.non_field_errors()`](../../ref/forms/api.html#django.forms.Form.non_field_errors "django.forms.Form.non_field_errors")访问的错误）。

`errors`是预期为表单验证结果的错误字符串或错误字符串列表。

`SimpleTestCase.``assertFormsetError`(_response_, _formset_, _form_index_, _field_, _errors_, _msg_prefix=''_)

断言`formset`在呈现时引发提供的错误列表。

`formset`是在模板上下文中给出的`Formset`实例的名称。

`form_index`是`Formset`内的表单编号。如果`form_index`的值为`None`，则将检查非格式错误（可通过`formset.non_form_errors()`访问的错误）。

`field`是要检查的表单上的字段的名称。如果`field`的值为`None`，则将检查非字段错误（可通过[`form.non_field_errors()`](../../ref/forms/api.html#django.forms.Form.non_field_errors "django.forms.Form.non_field_errors")访问的错误）。

`errors`是预期为表单验证结果的错误字符串或错误字符串列表。

`SimpleTestCase.``assertContains`(_response_, _text_, _count=None_, _status_code=200_, _msg_prefix=''_, _html=False_)

断言`Response`实例生成给定的`status_code`，并且该`文本`显示在响应的内容中。如果提供了`count`，则`text`必须在响应中出现正好`count`次数。

将`html`设置为`True`，将`text`作为HTML。与响应内容的比较将基于HTML语义，而不是逐个字符的等同。在大多数情况下忽略空白，属性排序不重要。有关详细信息，请参见[`assertHTMLEqual()`](#django.test.SimpleTestCase.assertHTMLEqual "django.test.SimpleTestCase.assertHTMLEqual")。

`SimpleTestCase.``assertNotContains`(_response_, _text_, _status_code=200_, _msg_prefix=''_, _html=False_)

认为`Response`实例产生给定的`status_code`，并且`text`不会_不会出现在响应的内容中。_

将`html`设置为`True`，将`text`作为HTML。与响应内容的比较将基于HTML语义，而不是逐个字符的等同。在大多数情况下忽略空白，属性排序不重要。有关详细信息，请参见[`assertHTMLEqual()`](#django.test.SimpleTestCase.assertHTMLEqual "django.test.SimpleTestCase.assertHTMLEqual")。

`SimpleTestCase.``assertTemplateUsed`(_response_, _template_name_, _msg_prefix=''_, _count=None_)

断言具有给定名称的模板用于呈现响应。

名称是一个字符串，例如`'admin/index.html'`。

New in Django 1.8:

count参数是一个整数，表示模板应该渲染的次数。默认值为`None`，表示模板应该呈现一次或多次。

你可以使用它作为上下文管理器，像这样：

```
with self.assertTemplateUsed('index.html'):
    render_to_string('index.html')
with self.assertTemplateUsed(template_name='index.html'):
    render_to_string('index.html')

```

`SimpleTestCase.``assertTemplateNotUsed`(_response_, _template_name_, _msg_prefix=''_)

Asserts that the template with the given name was _not_ used in rendering the response.

您可以使用与[`assertTemplateUsed()`](#django.test.SimpleTestCase.assertTemplateUsed "django.test.SimpleTestCase.assertTemplateUsed")相同的方式作为上下文管理器。

`SimpleTestCase.``assertRedirects`(_response_, _expected_url_, _status_code=302_, _target_status_code=200_, _host=None_, _msg_prefix=''_, _fetch_redirect_response=True_)

Asserts that the response returned a `status_code` redirect status, redirected to `expected_url` (including any `GET` data), and that the final page was received with `target_status_code`.

如果您的请求使用`follow`参数，则`expected_url`和`target_status_code`将是重定向链最后一点的网址和状态代码。

如果`expected_url`不包含一个（例如`"/bar/"`），`host`如果`expected_url`是包含主机的绝对网址（例如`"http://testhost/bar/"`），则`host`忽略。请注意，测试客户端不支持获取外部URL，但是如果您使用自定义HTTP主机进行测试（例如，使用`Client(HTTP_HOST="testhost")`

New in Django 1.7.

如果`fetch_redirect_response`为`False`，则不会加载最后一页。由于测试客户端无法获取外部网址，因此如果`expected_url`不是您的Django应用程序的一部分，则此功能特别有用。

New in Django 1.7.

在两个URL之间进行比较时，会正确处理Scheme。如果在我们重定向到的位置中没有指定任何方案，则使用原始请求的方案。如果存在，`expected_url`中的方案是用于进行比较的方案。

`SimpleTestCase.``assertHTMLEqual`(_html1_, _html2_, _msg=None_)

断言字符串`html1`和`html2`是相等的。比较基于HTML语义。比较需要考虑以下因素：

*   忽略HTML标记之前和之后的空格。
*   所有类型的空格都被视为等同。
*   所有打开的标签被隐式地关闭，例如。当周围标签关闭或HTML文档结束时。
*   空标记等同于其自我关闭版本。
*   HTML元素的属性顺序并不重要。
*   没有参数的属性等于名称和值相等的属性（参见示例）。

以下示例是有效的测试，不会引发任何`AssertionError`：

```
self.assertHTMLEqual('&lt;p&gt;Hello &lt;b&gt;world!&lt;/p&gt;',
    '''&lt;p&gt;
        Hello   &lt;b&gt;world! &lt;b/&gt;
    &lt;/p&gt;''')
self.assertHTMLEqual(
    '&lt;input type="checkbox" checked="checked" id="id_accept_terms" /&gt;',
    '&lt;input id="id_accept_terms" type='checkbox' checked&gt;')

```

`html1`和`html2`必须是有效的HTML。如果其中一个无法解析，则会引发`AssertionError`。

出错时的输出可以用`msg`参数定制。

`SimpleTestCase.``assertHTMLNotEqual`(_html1_, _html2_, _msg=None_)

认为字符串`html1`和`html2`的_不是_相等。比较基于HTML语义。有关详细信息，请参见[`assertHTMLEqual()`](#django.test.SimpleTestCase.assertHTMLEqual "django.test.SimpleTestCase.assertHTMLEqual")。

`html1`和`html2`必须是有效的HTML。如果其中一个无法解析，则会引发`AssertionError`。

出错时的输出可以用`msg`参数定制。

`SimpleTestCase.``assertXMLEqual`(_xml1_, _xml2_, _msg=None_)

断言字符串`xml1`和`xml2`相等。比较基于XML语义。与[`assertHTMLEqual()`](#django.test.SimpleTestCase.assertHTMLEqual "django.test.SimpleTestCase.assertHTMLEqual")类似，对解析的内容进行比较，因此仅考虑语义差异，而不考虑语法差异。当在任何参数中传递无效的XML时，始终会出现`AssertionError`，即使两个字符串都相同。

出错时的输出可以用`msg`参数定制。

`SimpleTestCase.``assertXMLNotEqual`(_xml1_, _xml2_, _msg=None_)

断言字符串`xml1`和`xml2`的_不是_相等。比较基于XML语义。有关详细信息，请参见[`assertXMLEqual()`](#django.test.SimpleTestCase.assertXMLEqual "django.test.SimpleTestCase.assertXMLEqual")。

出错时的输出可以用`msg`参数定制。

`SimpleTestCase.``assertInHTML`(_needle_, _haystack_, _count=None_, _msg_prefix=''_)

断言HTML片段`needle`包含在`haystack`中。

如果指定`count`整数参数，则将严格验证`needle`出现的次数。

在大多数情况下，空白被忽略，属性排序不重要。传入的参数必须是有效的HTML。

`SimpleTestCase.``assertJSONEqual`(_raw_, _expected_data_, _msg=None_)

断言JSON片段`raw`和`expected_data`是相等的。通常的JSON非重要空格规则适用于将重量级委派给[`json`](https://docs.python.org/3/library/json.html#module-json "(in Python v3.4)")库。

出错时的输出可以用`msg`参数定制。

`SimpleTestCase.``assertJSONNotEqual`(_raw_, _expected_data_, _msg=None_)

New in Django 1.8.

断言JSON片段`raw`和`expected_data`的_不是_相等。有关详细信息，请参见[`assertJSONEqual()`](#django.test.SimpleTestCase.assertJSONEqual "django.test.SimpleTestCase.assertJSONEqual")。

出错时的输出可以用`msg`参数定制。

`TransactionTestCase.``assertQuerysetEqual`(_qs_, _values_, _transform=repr_, _ordered=True_, _msg=None_)

断言查询集`qs`返回值`values`。

使用函数`transform`执行`qs`和`values`的内容的比较；默认情况下，这意味着比较每个值的`repr()`。如果`repr()`未提供唯一或有帮助的比较，则可以使用任何其他可调用项。

默认情况下，比较也依赖于顺序。如果`qs`不提供隐式排序，可以将`ordered`参数设置为`False`，将比较结果转换为`collections.`计数器比较。如果顺序未定义（如果给定的`qs`不是有序的，并且比较是针对多个有序值），则会引发`ValueError`。

出错时的输出可以用`msg`参数定制。

Changed in Django 1.7:

该方法现在接受`msg`

`TransactionTestCase.``assertNumQueries`(_num_, _func_, _*args_, _**kwargs_)

断言当使用`*args`和`**kwargs`调用`func`时，将执行`num`数据库查询。

如果`kwargs`中存在`"using"`键，它将用作要检查查询数的数据库别名。如果你想用`using`参数调用一个函数，你可以通过用`lambda`包装调用来添加一个额外的参数：

```
self.assertNumQueries(7, lambda: my_function(using=7))

```

您还可以将其用作上下文管理器：

```
with self.assertNumQueries(2):
    Person.objects.create(name="Aaron")
    Person.objects.create(name="Daniel")

```

## 电子邮件服务

如果您的任何Django视图使用[_Django’s email functionality_](../email.html)发送电子邮件，您可能不希望在每次使用该视图运行测试时发送电子邮件。因此，Django的测试运行器会自动将所有Django发送的电子邮件重定向到一个虚拟发件箱。这使您可以测试发送电子邮件的每个方面 - 从发送到每封邮件内容的邮件数量，而不实际发送邮件。

测试运行器通过用测试后端透明地替换普通电子邮件后端来实现这一点。（不要担心，这对Django之外的任何其他电子邮件发件人（例如您计算机的邮件服务器）（如果您正在运行）没有任何影响。）

`django.core.mail.``outbox`

在测试运行期间，每封发出的电子邮件都保存在`django.core.mail.outbox`中。这是所有已发送的[`EmailMessage`](../email.html#django.core.mail.EmailMessage "django.core.mail.EmailMessage")实例的简单列表。`outbox`属性是在使用`locmem`电子邮件后端时仅在创建的特殊属性。它通常不作为[`django.core.mail`](../email.html#module-django.core.mail "django.core.mail: Helpers to easily send email.")模块的一部分存在，您不能直接导入它。下面的代码显示了如何正确访问此属性。

下面是一个示例测试，检查`django.core.mail.outbox`的长度和内容：

```
from django.core import mail
from django.test import TestCase

class EmailTest(TestCase):
    def test_send_email(self):
        # Send message.
        mail.send_mail('Subject here', 'Here is the message.',
            'from@example.com', ['to@example.com'],
            fail_silently=False)

        # Test that one message has been sent.
        self.assertEqual(len(mail.outbox), 1)

        # Verify that the subject of the first message is correct.
        self.assertEqual(mail.outbox[0].subject, 'Subject here')

```

如前所述[_previously_](#emptying-test-outbox)，测试发件箱在Django `*TestCase`中的每个测试开始时都被清空。要手动清空发件箱，请将空列表分配给`mail.outbox`：

```
from django.core import mail

# Empty the test outbox
mail.outbox = []

```

## 管理命令

可以使用[`call_command()`](../../ref/django-admin.html#django.core.management.call_command "django.core.management.call_command")函数测试管理命令。输出可以重定向到`StringIO`实例：

```
from django.core.management import call_command
from django.test import TestCase
from django.utils.six import StringIO

class ClosepollTest(TestCase):
    def test_command_output(self):
        out = StringIO()
        call_command('closepoll', stdout=out)
        self.assertIn('Expected output', out.getvalue())

```

## 跳过测试

unittest库提供[`@skipIf`](https://docs.python.org/3/library/unittest.html#unittest.skipIf "(in Python v3.4)")和[`@skipUnless`](https://docs.python.org/3/library/unittest.html#unittest.skipUnless "(in Python v3.4)")装饰器，如果提前知道这些测试在某些条件下会失败，您可以跳过测试。

例如，如果您的测试需要特定的可选库来成功，您可以使用[`@skipIf`](https://docs.python.org/3/library/unittest.html#unittest.skipIf "(in Python v3.4)")来装饰测试用例。然后，测试运行器将报告测试没有被执行以及为什么，而不是失败测试或完全省略测试。

为了补充这些测试跳过行为，Django提供了两个额外的跳过装饰器。这些装饰器不是测试通用布尔值，而是检查数据库的功能，如果数据库不支持特定的命名特性，则跳过测试。

装饰器使用字符串标识符来描述数据库特征。此字符串对应于数据库连接要素类的属性。有关可用作跳过测试的基础的数据库功能的完整列表，请参见`django.db.backends.BaseDatabaseFeatures`类。

`skipIfDBFeature`(_*feature_name_strings_)

如果支持所有命名的数据库功能，请跳过装饰测试或`TestCase`。

例如，如果数据库支持事务（例如，_不_在PostgreSQL下运行，但在MySQL with MyISAM表下），则不会执行以下测试：

```
class MyTests(TestCase):
    @skipIfDBFeature('supports_transactions')
    def test_transaction_behavior(self):
        # ... conditional test code

```

Changed in Django 1.7:

`skipIfDBFeature`现在可以用来装饰`TestCase`类。

Changed in Django 1.8:

`skipIfDBFeature`可以接受多个要素字符串。

`skipUnlessDBFeature`(_*feature_name_strings_)

如果任何命名的数据库功能_不支持_，请跳过装饰测试或`TestCase`。

例如，如果数据库支持事务（例如，它将在PostgreSQL下运行，但在MySQL和MyISAM表下的_不是_），则将执行以下测试：

```
class MyTests(TestCase):
    @skipUnlessDBFeature('supports_transactions')
    def test_transaction_behavior(self):
        # ... conditional test code

```

Changed in Django 1.7:

`skipUnlessDBFeature`现在可用于装饰`TestCase`类。

Changed in Django 1.8:

`skipUnlessDBFeature`可以接受多个要素字符串。
