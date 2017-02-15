

# 消息框架

在网页应用中，你经常需要在处理完表单或其它类型的用户输入后，显示一个通知消息（也叫做“flash message”）给用户。

对于这个功能，Django 提供基于Cookie 和会话的消息，无论是匿名用户还是认证的用户。其消息框架允许你临时将消息存储在请求中，并在接下来的请求（通常就是下一个请求）中提取它们并显示。每个消息都带有一个特定`level` 标签，表示其优先级（例如`info`、`warning` 或`error`）。

## 启用消息框架

消息框架的实现通过一个[_中间件_](../middleware.html) 类和对应的[_context processor_](../templates/api.html)。

`django-admin startproject` 创建的默认`settings.py`  已经包含启用消息框架功能需要的所有的设置：

*   [`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 中的`'django.contrib.messages'`。

*   [`MIDDLEWARE_CLASSES`](../settings.html#std:setting-MIDDLEWARE_CLASSES) 中的`'django.contrib.sessions.middleware.SessionMiddleware'` 和 `'django.contrib.messages.middleware.MessageMiddleware'`。

    默认的[_后端存储_](#message-storage-backends) 依赖[_sessions_](../../topics/http/sessions.html)。所以[`MIDDLEWARE_CLASSES`](../settings.html#std:setting-MIDDLEWARE_CLASSES) 中必须启用`SessionMiddleware` 并出现在`MessageMiddleware` 之前。

*   [`TEMPLATES`](../settings.html#std:setting-TEMPLATES) 设置中定义的`DjangoTemplates` 的`'context_processors'` 选项包含 `'django.contrib.messages.context_processors.messages'`。

如果你不想使用消息框架，你可以删除[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS) 中的 `'django.contrib.messages'`、[`MIDDLEWARE_CLASSES`](../settings.html#std:setting-MIDDLEWARE_CLASSES) 中的`MessageMiddleware` 和[`TEMPLATES`](../settings.html#std:setting-TEMPLATES) 中的`messages` context processor。

## 配置消息框架引擎

### 后台存储

消息框架可以使用不同的后台存储临时消息。

Django 在[`django.contrib.messages`](#module-django.contrib.messages "django.contrib.messages: Provides cookie- and session-based temporary message storage.") 中提供三个内建的存储类：

_class_ `storage.session.``SessionStorage`

这个类存储所有的消息于请求的会话中。因此，它要求启用Django 的`contrib.sessions` 应用。

_class_ `storage.cookie.``CookieStorage`

这个类存储消息数据于与Cookie 中（已经用一个安全的哈希进行签名以防止篡改）以在请求之间传递消息。如果Cookie 数据的大小将超过2048 字节，将丢弃旧的消息。

_class_ `storage.fallback.``FallbackStorage`

这个类首先使用`CookieStorage`，如果消息塞不进一个Cookie 中则使用`SessionStorage`。 它同样要求启用Django 的`contrib.sessions` 应用。

这个行为避免每次都写会话。在通常情况下，它提供的性能应该是最好的。

[`FallbackStorage`](#django.contrib.messages.storage.fallback.FallbackStorage "django.contrib.messages.storage.fallback.FallbackStorage") 是默认的存储类。如果它不适合你的需要，你可以通过设置 [`MESSAGE_STORAGE`](../settings.html#std:setting-MESSAGE_STORAGE) 为它的完整导入路径选择另外一个存储类，例如：

```
MESSAGE_STORAGE = 'django.contrib.messages.storage.cookie.CookieStorage'

```

_class_ `storage.base.``BaseStorage`

如果想编写你自己的存储类，子类化`django.contrib.messages.storage.base` 中的`BaseStorage` 类并实现`_get` 和 `_store` 方法。

### 消息级别

消息框架的级别是可配置的，与Python logging 模块类似。消息的级别可以让你根据类型进行分组，这样它们能够在不同的视图和模板中过滤或显示出来。

可以直接从`django.contrib.messages` 导入的内建级别有：


| Constant | Purpose |
| --- | --- |
| `DEBUG` | Development-related messages that will be ignored (or removed) in a production deployment |
| `INFO` | Informational messages for the user |
| `SUCCESS` | An action was successful, e.g. “Your profile was updated successfully” |
| `WARNING` | A failure did not occur but may be imminent |
| `ERROR` | An action was **not** successful or some other failure occurred |

[`MESSAGE_LEVEL`](../settings.html#std:setting-MESSAGE_LEVEL) 设置可以用来改变记录的最小级别（它还可以[在每个请求中修改](#changing-the-minimum-recorded-level-per-request)）。小于这个级别的消息将被忽略。

### 消息的标签

消息的标签是一个字符串，表示消息的级别以及在视图中添加的其它标签（参见下文[添加额外的消息标签](#adding-extra-message-tags)）。标签存储在字符串中并通过空格分隔。通常情况下，消息的标签用于作为CSS 类来根据消息的类型定制消息的风格。默认情况下，每个级别具有一个标签，为其级别的字符串常量的小写：


| Level Constant | Tag |
| --- | --- |
| `DEBUG` | `debug` |
| `INFO` | `info` |
| `SUCCESS` | `success` |
| `WARNING` | `warning` |
| `ERROR` | `error` |

若要修改消息级别的默认标签，设置[`MESSAGE_TAGS`](../settings.html#std:setting-MESSAGE_TAGS)为包含你想要修改的级别的字典。由于这扩展了默认标记，您只需要为要覆盖的级别提供标记：

```
from django.contrib.messages import constants as messages
MESSAGE_TAGS = {
    messages.INFO: '',
    50: 'critical',
}

```

## 在视图和模板中使用消息

`add_message`(_request_, _level_, _message_, _extra_tags=''_, _fail_silently=False_)

### 新增一条消息

新增一条消息，调用：

```
from django.contrib import messages
messages.add_message(request, messages.INFO, 'Hello world.')

```

有几个快捷方法提供标准的方式来新增消息并带有常见的标签（这些标签通常表示消息的HTML 类型）：

```
messages.debug(request, '%s SQL statements were executed.' % count)
messages.info(request, 'Three credits remain in your account.')
messages.success(request, 'Profile details updated.')
messages.warning(request, 'Your account expires in three days.')
messages.error(request, 'Document deleted.')

```

### 显示消息

`get_messages`(_request_)

**在你的模板中**，像下面这样使用：

```
{% if messages %}
<ul class="messages">
    {% for message in messages %}
    <li{% if message.tags %} class="{{ message.tags }}"{% endif %}>{{ message }}</li>
    {% endfor %}
</ul>
{% endif %}

```

如果你正在使用context processor，你的模板应该通过 `RequestContext` 渲染。否则，需要确保`messages` 在模板的Context 中可以访问。

即使你知道只有一条消息，你也应该仍然迭代`messages` 序列，否则下个请求中的消息不会被清除。

New in Django 1.7.

Context processor 还提供一个`DEFAULT_MESSAGE_LEVELS` 变量，它映射消息级别的名称到它们的数值：

```
{% if messages %}
<ul class="messages">
    {% for message in messages %}
    <li{% if message.tags %} class="{{ message.tags }}"{% endif %}>
        {% if message.level == DEFAULT_MESSAGE_LEVELS.ERROR %}Important: {% endif %}
        {{ message }}
    </li>
    {% endfor %}
</ul>
{% endif %}

```

**在模板的外面**，你可以使用[`get_messages()`](#django.contrib.messages.get_messages "django.contrib.messages.get_messages")：

```
from django.contrib.messages import get_messages

storage = get_messages(request)
for message in storage:
    do_something_with_the_message(message)

```

例如，你可以获取所有的消息并在[_JSONResponseMixin_](../../topics/class-based-views/mixins.html#jsonresponsemixin-example) 而不是[`TemplateResponseMixin`](../class-based-views/mixins-simple.html#django.views.generic.base.TemplateResponseMixin "django.views.generic.base.TemplateResponseMixin") 中返回它们。

[`get_messages()`](#django.contrib.messages.get_messages "django.contrib.messages.get_messages") 将返回配置的存储后台的一个实例。

### 的`Message`

_class_ `storage.base.``Message`

当浏览某个模板的消息列表时，你得到的其实是`Message` 类的实例。它只是一个非常简单、只带很少属性的对象：

*   `message`: 消息的实际内容文本。
*   `level`: 一个整数，它描述了消息的类型 (请参阅上面 [message levels](#message-levels)一节).
*   `tags`: 一个字符串，它由该消息的所有标签 (`extra_tags` 和`level_tag`)组合而成，组合时用空格分割开这些标签。
*   `extra_tags`: 一个字符串，它由该消息的定制标签组合而成，并用空格分割。缺省为空。

New in Django 1.7.

*   `level_tag`: 代表该消息级别的字符串。该属性缺省由小写的关联常数名组成， 但当设置[`MESSAGE_TAGS`](../settings.html#std:setting-MESSAGE_TAGS)参数时，可改变该规则。

### 创建自定义消息级别

消息级别只是整数，因此您可以定义自己的级别常量，并使用它们创建更多自定义的用户反馈，例如：

```
CRITICAL = 50

def my_view(request):
    messages.add_message(request, CRITICAL, 'A serious error occurred.')

```

在创建自定义消息级别时，应小心避免重载现有级别。内置级别的值为：


| Level Constant | Value |
| --- | --- |
| `DEBUG` | 10 |
| `INFO` | 20 |
| `SUCCESS` | 25 |
| `WARNING` | 30 |
| `ERROR` | 40 |

如果您需要识别HTML或CSS中的自定义级别，则需要通过[`MESSAGE_TAGS`](../settings.html#std:setting-MESSAGE_TAGS)设置提供映射。

注意

如果要创建可重复使用的应用程序，建议仅使用内置的[消息级别](#message-levels)，而不依赖于任何自定义级别。

### 在每个请求中修改最小的记录级别

每个请求都可以通过 `set_level`方法设置最小记录级别:

```
from django.contrib import messages

# Change the messages level to ensure the debug message is added.
messages.set_level(request, messages.DEBUG)
messages.debug(request, 'Test message...')

# In another request, record only messages with a level of WARNING and higher
messages.set_level(request, messages.WARNING)
messages.success(request, 'Your profile was updated.') # ignored
messages.warning(request, 'Your account is about to expire.') # recorded

# Set the messages level back to default.
messages.set_level(request, None)

```

与此相似，当前有效的记录级别可以用`get_level`方法获取:

```
from django.contrib import messages
current_level = messages.get_level(request)

```

有关最小记录级别相关的函数信息，请参阅上面 [Message levels](#message-levels) 一节.

### 添加额外的消息标签

要更直接地控制消息标签，您可以选择为任何添加方法提供包含额外标签的字符串：

```
messages.add_message(request, messages.INFO, 'Over 9000!',
                     extra_tags='dragonball')
messages.error(request, 'Email box full', extra_tags='email')

```

在该级别的默认标记之前添加额外的标记，并以空格分隔。

### 当消息框架被禁止时，失败静悄悄

如果您撰写的是可重复使用的应用程式（或其他碎片代码），但想要包含讯息功能，但又不想要您的使用者启用（如果他们不想使用），您可以传送额外的关键字参数` fail_silently = True`指向任何`add_message`方法系列。举个例子

```
messages.add_message(request, messages.SUCCESS, 'Profile details updated.',
                     fail_silently=True)
messages.info(request, 'Hello world.', fail_silently=True)

```

注意

设置`fail_silently=True`只会隐藏消息框架禁用时会出现的`MessageFailure`，并尝试使用`add_message` 。它不会隐藏可能由于其他原因发生的故障。

### 在基于类的视图中添加消息

_class_ `views.``SuccessMessageMixin`

向基于[`FormView`](../class-based-views/generic-editing.html#django.views.generic.edit.FormView "django.views.generic.edit.FormView") 的类添加一条成功的消息

`get_success_message`(_cleaned_data_)

`cleaned_data` 是表单中的清洁数据，用于字符串格式化

**示例 views.py**：

```
from django.contrib.messages.views import SuccessMessageMixin
from django.views.generic.edit import CreateView
from myapp.models import Author

class AuthorCreate(SuccessMessageMixin, CreateView):
    model = Author
    success_url = '/success/'
    success_message = "%(name)s was created successfully"

```

字符串插值可以使用`%(field_name)s` 语法访问`form` 中的清洁数据。对于ModelForms，如果你需要访问保存的`object` 中的字段，可以覆盖[`get_success_message()`](#django.contrib.messages.views.SuccessMessageMixin.get_success_message "django.contrib.messages.views.SuccessMessageMixin.get_success_message") 方法。

**ModelForms 的示例views.py**：

```
from django.contrib.messages.views import SuccessMessageMixin
from django.views.generic.edit import CreateView
from myapp.models import ComplicatedModel

class ComplicatedCreate(SuccessMessageMixin, CreateView):
    model = ComplicatedModel
    success_url = '/success/'
    success_message = "%(calculated_field)s was created successfully"

    def get_success_message(self, cleaned_data):
        return self.success_message % dict(cleaned_data,
                                           calculated_field=self.object.calculated_field)

```

## 消息过期

消息被标记为在存储实例被迭代时被清除（并且当响应被处理时被清除）。

为了避免消息被清除，您可以在迭代后将消息存储设置为`False`：

```
storage = messages.get_messages(request)
for message in storage:
    do_something_with(message)
storage.used = False

```

## 并行请求的行为

由于Cookie（以及会话）的工作方式，**使用Cookie或会话的任何后端的行为在同一客户端发出并行设置或获取消息的多个请求时未定义**。例如，如果客户端在第一窗口重定向之前发起在一个窗口（或标签）中创建消息并且然后在另一个窗口中获取另一个单元消息的请求，则该消息可以出现在第二窗口中而不是第一窗口中窗口，它可能是预期的。

简而言之，当涉及来自相同客户端的多个同时请求时，不能保证消息被传递到创建它们的相同窗口，在某些情况下根本不传递。注意，这在大多数应用中通常不是问题，并且在HTML5中将成为非问题，其中每个窗口/选项卡将具有其自己的浏览上下文。

## 设置

几个[_settings_](../settings.html#settings-messages)可让您控制邮件行为：

*   [MESSAGE_LEVEL](../settings.html#std:setting-MESSAGE_LEVEL)
*   [MESSAGE_STORAGE](../settings.html#std:setting-MESSAGE_STORAGE)
*   [MESSAGE_TAGS](../settings.html#std:setting-MESSAGE_TAGS)

New in Django 1.7.

对于使用Cookie的后端，Cookie的设置取自会话Cookie设置：

*   [SESSION_COOKIE_DOMAIN](../settings.html#std:setting-SESSION_COOKIE_DOMAIN)
*   [SESSION_COOKIE_SECURE](../settings.html#std:setting-SESSION_COOKIE_SECURE)
*   [SESSION_COOKIE_HTTPONLY](../settings.html#std:setting-SESSION_COOKIE_HTTPONLY)
