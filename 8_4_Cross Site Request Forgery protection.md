

# 跨站请求伪造保护

CSRF 中间件和模板标签提供对[跨站请求伪造](http://www.squarefree.com/securitytips/web-developers.html#CSRF)简单易用的防护。某些恶意网站上包含链接、表单按钮或者JavaScript ，它们会利用登录过的用户在浏览器中的认证信息试图在你的网站上完成某些操作，这就是跨站攻击。还有另外一种相关的攻击叫做“登录CSRF”，攻击站点触发用户浏览器用其它人的认证信息登录到其它站点。

防护CSRF 攻击的第一道防线是保证GET 请求（以及在9.1.1 Safe Methods, HTTP 1.1, [**RFC 2616**](http://tools.ietf.org/html/rfc2616.html#section-9.1.1) 中定义的其它安全的方法）不会产生副作用。通过不安全的请求方法例如POST、PUT 和DELETE  可以通过以下步骤进行防护。

## 如何使用

想在你的视图中使用CSRF 防护，请遵循以下步骤：

1.  CSRF 中间件在[`MIDDLEWARE_CLASSES`](settings.html#std:setting-MIDDLEWARE_CLASSES) 设置中默认启用。如果你要覆盖这个设置，请记住 `'django.middleware.csrf.CsrfViewMiddleware'` 应该位于其它任何假设CSRF 已经处理过的视图中间件之前。

    如果你关闭了它，虽然不建议，你可以在你想要保护的视图上使用[`csrf_protect()`](#django.views.decorators.csrf.csrf_protect "django.views.decorators.csrf.csrf_protect")（见下文）。

2.  在使用POST 表单的模板中，对于内部的URL请在`&lt;form&gt;` 元素中使用[`csrf_token`](templates/builtins.html#std:templatetag-csrf_token) 标签：

    ```
    &lt;form action="." method="post"&gt;

    ```

    它不应该用于目标是外部URL 的POST 表单，因为这将引起CSRF 信息泄露而导致出现漏洞。

3.  在对应的视图函数中，确保使用 `'django.template.context_processors.csrf'` Context 处理器。通常可以用两种方法实现：

    1.  使用RequestContext，它会始终使用`'django.template.context_processors.csrf'`（无论 [`TEMPLATES`](settings.html#std:setting-TEMPLATES) 设置中配置的是什么模板上下文处理器）。如果你正在使用通用视图或Contrib 中的应用，你就不用担心了，因为这些应用通篇都使用RequestContext。

    2.  手工导入并使用处理器来生成CSRF token，并将它添加到模板上下文中。例如：

        ```
        from django.shortcuts import render_to_response
        from django.template.context_processors import csrf

        def my_view(request):
            c = {}
            c.update(csrf(request))
            # ... view code here
            return render_to_response("a_template.html", c)

        ```

        你可能想要编写你自己的[`render_to_response()`](../topics/http/shortcuts.html#django.shortcuts.render_to_response "django.shortcuts.render_to_response") 来处理这个步骤。

### AJAX

虽然上面的方法可以用于AJAX POST 请求，但是它不太方便：你必须记住在每个POST 请求的数据中传递CSRF token。由于这个原因，还有另外一种方法：在每个XMLHttpRequest 上设置一个自定义的`X-CSRFToken` 头部，其值为CSRF token。这非常容易，因为许多JavaScript 框架都提供在每个请求上设置头部的方法。

第一步，你必须获得CSRF token。建议从`csrftoken` Cookie 中获取，如果你在视图中启用CSRF 防护它就会设置。

注

CSRF token 的Cookie 默认叫做`csrftoken`，你可以通过[`CSRF_COOKIE_NAME`](settings.html#std:setting-CSRF_COOKIE_NAME) 设置自定义它的名字。

获取token 非常简单：

```
// using jQuery
function getCookie(name) {
    var cookieValue = null;
    if (document.cookie && document.cookie != '') {
        var cookies = document.cookie.split(';');
        for (var i = 0; i < cookies.length; i++) {
            var cookie = jQuery.trim(cookies[i]);
            // Does this cookie string begin with the name we want?
            if (cookie.substring(0, name.length + 1) == (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}
var csrftoken = getCookie('csrftoken');

```

以上代码可以使用[jQuery cookie plugin](http://plugins.jquery.com/cookie/) 来替换`getCookie`：

```
var csrftoken = $.cookie('csrftoken');

```

注

CSRF token 也存在于DOM 中，但只有你在模板中明确使用[`csrf_token`](templates/builtins.html#std:templatetag-csrf_token) 标签时才有。Cookie 中包含标准的token；`CsrfViewMiddleware` 倾向于使用Cookie 而不是DOM 中的token。无论如何，如果DOM 中具有token，Cookie 中将保证会有，所以你应该使用Cookie。

警告

如果你的视图渲染的模板没有包含[`csrf_token`](templates/builtins.html#std:templatetag-csrf_token) 标签，Django 可能不会再Cookie 中设置CSRF token。这常见于表单是动态的方式添加到网页中的。为了解决这个问题，Django 提供一个视图装饰器[`ensure_csrf_cookie()`](#django.views.decorators.csrf.ensure_csrf_cookie "django.views.decorators.csrf.ensure_csrf_cookie")，它将强制设置这个Cookie。

最后，你不想在AJAX 请求中设置头部，使用jQuery 已经更新版本的 [settings.crossDomain](http://api.jquery.com/jQuery.ajax) 可以包含CSRF token 不会发送给其它域：

```
function csrfSafeMethod(method) {
    // these HTTP methods do not require CSRF protection
    return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
}
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
            xhr.setRequestHeader("X-CSRFToken", csrftoken);
        }
    }
});

```

### 其它模版引擎

当使用非Django自带的模板引擎，你可以手动设置token，但要确保它在template context中可用

例如，在采用Jinja2模板引擎时，你可以在表单中包含以下内容：

```
<div style="display:none">
    <input type="hidden" name="csrfmiddlewaretoken" value="nK3w9RcmP6lHX8mUKQxmFRL49osJDl5X">
</div>

```

您可以使用与上述[_AJAX code_](#csrf-ajax)类似的JavaScript来获取CSRF令牌的值。

### 装饰器方法

您可以对需要保护的特定视图使用`csrf_protect`装饰器，而不是添加`CsrfViewMiddleware`作为整体保护。对于在输出中插入CSRF令牌的视图以及接受POST表单数据的视图，必须使用**both**。（这些通常是相同的视图函数，但不总是）。

使用装饰器本身是**不推荐**的，因为如果你忘记使用它，你会有一个安全漏洞。使用两者的“belt and braces”策略是很好的，并且将产生最小的开销。

`csrf_protect`(_view_)

装饰器为视图提供`CsrfViewMiddleware`的保护。

用法：

```
from django.views.decorators.csrf import csrf_protect
from django.shortcuts import render

@csrf_protect
def my_view(request):
    c = {}
    # ...
    return render(request, "a_template.html", c)

```

如果您使用的是基于类的视图，则可以参考[_装饰基于类的视图_](../topics/class-based-views/intro.html#id2)。

## 已拒绝的请求

默认情况下，如果传入请求未能通过`CsrfViewMiddleware`执行的检查，则向用户发送“403禁止”响应。这通常只有在有一个真正的跨站点请求伪造，或由于编程错误，CSRF令牌未包括在POST表单中的情况下才会看到。

错误页面，但是，不是很友好，所以你可能想提供自己的视图来处理这种情况。为此，只需设置[`CSRF_FAILURE_VIEW`](settings.html#std:setting-CSRF_FAILURE_VIEW)设置即可。

## 它是如何工作的

跨站伪造保护基于以下几点：

1.  设置为随机值（与session无关的随机数）的CSRF Cookie，其他网站将无法访问。

    此Cookie由`CsrfViewMiddleware`设置。它是永久的，但由于没有办法设置一个永远不会过期的cookie，它会与每个响应一起发送，该响应调用`django.middleware.csrf.get_token()`（使用的函数内部检索CSRF令牌）。

2.  所有传出POST表单中都有一个名为“csrfmiddlewaretoken”的隐藏表单字段。此字段的值是CSRF cookie的值。

    此部分由模板标记完成。

3.  对于所有未使用HTTP GET，HEAD，OPTIONS或TRACE的传入请求，必须存在CSRF cookie，并且“csrfmiddlewaretoken”字段必须存在且正确。如果不是，用户将得到403错误。

    此检查由`CsrfViewMiddleware`完成。

4.  此外，对于HTTPS请求，`CsrfViewMiddleware`会执行严格的引用程序检查。这是必要的，以解决在使用独立于会话的现时在HTTPS下是可能的中间人中间人攻击，由于HTTP'Set-Cookie'头被（不幸）被接受由与站点。（由于Referer头的存在在HTTP下不够可靠，因此不会对HTTP请求执行Referer检查。）

这确保只有源自您的网站的表单才能用于POST数据。

它故意忽略GET请求（以及由 [**RFC 2616**](http://tools.ietf.org/html/rfc2616.html)定义为“安全”的其他请求）。这些请求不应该有任何潜在的危险副作用，因此具有GET请求的CSRF攻击应该是无害的。 [**RFC 2616**](http://tools.ietf.org/html/rfc2616.html)将POST，PUT和DELETE定义为“不安全”，并且假定所有其他方法都不安全，以获得最大保护。

## Caching

如果模板使用了[`csrf_token`](templates/builtins.html#std:templatetag-csrf_token)模板标签（或者`get_token`函数被称为某种其他方式），则`CsrfViewMiddleware`将添加一个Cookie和`Vary： Cookie`标头。这意味着如果按照指示使用中间件（`UpdateCacheMiddleware`在所有其他中间件之前），则中间件将与高速缓存中间件良好匹配。

但是，如果在单个视图上使用缓存装饰器，CSRF中间件将无法设置Vary头或CSRF cookie，并且响应将被缓存而没有任何一个。在这种情况下，在任何需要插入CSRF令牌的视图中，您应该首先使用[`django.views.decorators.csrf.csrf_protect()`](#django.views.decorators.csrf.csrf_protect "django.views.decorators.csrf.csrf_protect")装饰器：

```
from django.views.decorators.cache import cache_page
from django.views.decorators.csrf import csrf_protect

@cache_page(60 * 15)
@csrf_protect
def my_view(request):
    ...

```

如果您使用的是基于类的视图，则可以参考[_装饰基于类的视图_](../topics/class-based-views/intro.html#id2)。

## 测试

`CsrfViewMiddleware`通常会阻碍测试视图函数，因为需要每次POST请求都必须发送CSRF令牌。出于这个原因，Django的HTTP客户端测试已被修改，以设置一个标志，请求放松中间件和`csrf_protect`装饰器，以便他们不再拒绝请求。在每个其他方面（例如发送cookie等），它们的行为相同。

如果由于某种原因，您_希望_测试客户端执行CSRF检查，您可以创建实施CSRF检查的测试客户端的实例：

```
>>> from django.test import Client
>>> csrf_client = Client(enforce_csrf_checks=True)

```

## 限制

网站中的子网域可以在整个网域的客户端上设置Cookie。通过设置cookie并使用相应的令牌，子域将能够绕过CSRF保护。避免这种情况的唯一方法是确保子域由受信任的用户控制（或至少无法设置Cookie）。请注意，即使没有CSRF，也有其他漏洞，如会话固定，使得给不受信任的方的子域一个坏主意，这些漏洞不能轻易地用当前浏览器修复。

## 边框

某些视图可能有不寻常的要求，这意味着它们不适合这里设想的正常模式。在这些情况下，许多实用程序可能很有用。以下部分描述了可能需要的方案。

### 实用程序

下面的示例假设您使用基于函数的视图。如果您使用基于类的视图，则可以参考[_装饰基于类的视图_](../topics/class-based-views/intro.html#id2)。

`csrf_exempt`(_view_)[[source]](../_modules/django/views/decorators/csrf.html#csrf_exempt)

这个装饰器将视图标记为不受中间件保护的保护。例：

```
from django.views.decorators.csrf import csrf_exempt
from django.http import HttpResponse

@csrf_exempt
def my_view(request):
    return HttpResponse('Hello world')

```

`requires_csrf_token`(_view_)

通常，如果`CsrfViewMiddleware.process_view`或类似`csrf_protect`的等效项未运行，则[`csrf_token`](templates/builtins.html#std:templatetag-csrf_token)模板标记将无法正常工作。视图装饰器`requires_csrf_token`可用于确保模板标记正常工作。此装饰器与`csrf_protect`类似，但不会拒绝传入的请求。

例：

```
from django.views.decorators.csrf import requires_csrf_token
from django.shortcuts import render

@requires_csrf_token
def my_view(request):
    c = {}
    # ...
    return render(request, "a_template.html", c)

```

`ensure_csrf_cookie`(_view_)

此装饰器强制视图发送CSRF cookie。

### 场景

#### CSRF保护应该禁用只有几个视图

大多数视图需要CSRF保护，但有几个不需要。

解决方案：而不是禁用中间件，并对所有需要它的视图应用`csrf_protect`，启用中间件并使用[`csrf_exempt()`](#django.views.decorators.csrf.csrf_exempt "django.views.decorators.csrf.csrf_exempt")。

#### CsrfViewMiddleware.process_view未使用

有些情况下，如果`CsrfViewMiddleware.process_view`可能在您的视图运行之前没有运行 - 例如404和500处理程序，但是您仍然需要一个表单中的CSRF令牌。

解决方案：使用[`requires_csrf_token()`](#django.views.decorators.csrf.requires_csrf_token "django.views.decorators.csrf.requires_csrf_token")

#### 不受保护的视图需要CSRF令牌

可能有一些视图未受保护，并且已被`csrf_exempt`豁免，但仍需要包括CSRF令牌。

解决方案：使用[`csrf_exempt()`](#django.views.decorators.csrf.csrf_exempt "django.views.decorators.csrf.csrf_exempt")，后跟[`requires_csrf_token()`](#django.views.decorators.csrf.requires_csrf_token "django.views.decorators.csrf.requires_csrf_token")。（即`requires_csrf_token`应该是最内部的装饰器）。

#### View需要保护一个路径

视图仅需要一组条件下的CSRF保护，并且在其余时间内不能拥有它。

解决方案：对于需要保护的路径，使用[`csrf_exempt()`](#django.views.decorators.csrf.csrf_exempt "django.views.decorators.csrf.csrf_exempt")作为整个视图函数，[`csrf_protect()`](#django.views.decorators.csrf.csrf_protect "django.views.decorators.csrf.csrf_protect")例：

```
from django.views.decorators.csrf import csrf_exempt, csrf_protect

@csrf_exempt
def my_view(request):

    @csrf_protect
    def protected_path(request):
        do_something()

    if some_condition():
       return protected_path(request)
    else:
       do_something_else()

```

#### 页面使用AJAX，没有任何HTML表单

网页通过ajax发出post请求，并且该网页没有带有[`csrf_token`](templates/builtins.html#std:templatetag-csrf_token)的HTML表单，就会导致所需的CSRF Cookie被发送。

解决方案：在发送页面的视图上使用[`ensure_csrf_cookie()`](#django.views.decorators.csrf.ensure_csrf_cookie "django.views.decorators.csrf.ensure_csrf_cookie")。

## Contrib和可重复使用的应用程序

因为开发人员可以关闭`CsrfViewMiddleware`，所以contrib应用程序中的所有相关视图都使用`csrf_protect`装饰器，以确保这些应用程序的安全性不受CSRF的影响。建议其他需要相同保证的可重用应用程序的开发人员也在其视图上使用`csrf_protect`装饰器。

## 设置

许多设置可用于控制Django的CSRF行为：

*   [CSRF_COOKIE_AGE](settings.html#std:setting-CSRF_COOKIE_AGE)
*   [CSRF_COOKIE_DOMAIN](settings.html#std:setting-CSRF_COOKIE_DOMAIN)
*   [CSRF_COOKIE_HTTPONLY](settings.html#std:setting-CSRF_COOKIE_HTTPONLY)
*   [CSRF_COOKIE_NAME](settings.html#std:setting-CSRF_COOKIE_NAME)
*   [CSRF_COOKIE_PATH](settings.html#std:setting-CSRF_COOKIE_PATH)
*   [CSRF_COOKIE_SECURE](settings.html#std:setting-CSRF_COOKIE_SECURE)
*   [CSRF_FAILURE_VIEW](settings.html#std:setting-CSRF_FAILURE_VIEW)

