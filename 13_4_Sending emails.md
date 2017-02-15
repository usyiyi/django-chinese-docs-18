

# 发送邮件 

尽管Python 通过[`smtplib`](https://docs.python.org/3/library/smtplib.html#module-smtplib "(in Python v3.4)") 模块使得发送邮件很简单，Django 仍然在此基础上提供了几个轻量的封装包。这些封装包使得发送邮件非常快速、让开发中测试发送邮件变得很简单、并且支持不使用SMTP 的平台。

这些代码包含在`django.core.mail`模块中。

## 简单例子

两行代码实现：

```
from django.core.mail import send_mail

send_mail('Subject here', 'Here is the message.', 'from@example.com',
    ['to@example.com'], fail_silently=False)

```

邮件使用[`EMAIL_HOST`](../ref/settings.html#std:setting-EMAIL_HOST) 和 [`EMAIL_PORT`](../ref/settings.html#std:setting-EMAIL_PORT) 设置指定的SMTP 主机和端口发送。如果settings中设置了 [`EMAIL_HOST_USER`](../ref/settings.html#std:setting-EMAIL_HOST_USER) 和 [`EMAIL_HOST_PASSWORD`](../ref/settings.html#std:setting-EMAIL_HOST_PASSWORD) 它们将被用来验证SMTP主机， 并且如果设置了 [`EMAIL_USE_TLS`](../ref/settings.html#std:setting-EMAIL_USE_TLS) 和 [`EMAIL_USE_SSL`](../ref/settings.html#std:setting-EMAIL_USE_SSL) 它们将控制是否使用相应的加密链接。

注

`django.core.mail`发送邮件时使用的字符集将按照你在settings中的 [`DEFAULT_CHARSET`](../ref/settings.html#std:setting-DEFAULT_CHARSET) 项来设置。

## 发送邮件()

`send_mail`(_subject_, _message_, _from_email_, _recipient_list_, _fail_silently=False_, _auth_user=None_, _auth_password=None_, _connection=None_, _html_message=None_)[[source]](../_modules/django/core/mail.html#send_mail)

发送邮件最简单的方法是使用`django.core.mail.send_mail()`。

`subject`、`message`、`from_email`和 `recipient_list` 参数是必须的。

*   `subject`：一个字符串。
*   `message`：一个字符串。
*   `from_email`：一个字符串。
*   `recipient_list`：一个由邮箱地址组成的字符串列表。`recipient_list` 中的每一个成员都会在邮件信息的“To:”区域看到其它成员。
*   `fail_silently`: 一个布尔值。如果设置为 `False`, `send_mail` 将引发一个 [`smtplib.`](https://docs.python.org/3/library/smtplib.html#smtplib.SMTPException "(in Python v3.4)")SMTPException异常. 查看 [`smtplib`](https://docs.python.org/3/library/smtplib.html#module-smtplib "(in Python v3.4)") 文档中列车出的所有可能的异常， 它们都是 [`SMTPException`](https://docs.python.org/3/library/smtplib.html#smtplib.SMTPException "(in Python v3.4)")的子类。
*   `auth_user`: 可选的用户名用来验证SMTP服务器。如果没有提供这个值， Django 将会使用settings中 [`EMAIL_HOST_USER`](../ref/settings.html#std:setting-EMAIL_HOST_USER) 的值。
*   `auth_password`: 可选的密码用来验证 SMTP 服务器。如果没有提供这个值， Django 将会使用settings中 [`EMAIL_HOST_PASSWORD`](../ref/settings.html#std:setting-EMAIL_HOST_PASSWORD) 的值。
*   `connection`: 可选的用来发送邮件的电子邮件后端。如果没有指定，将使用缺省的后端实例。查看 [_Email backends_](#topic-email-backends) 文档来获取更多细节。
*   `html_message`: 如果提供了 `html_message` ，会导致邮件变成 _multipart/alternative_ ， `message` 格式变成 _text/plain_ ， `html_message` 格式变成 _text/html_ 。

返回值将是成功传递的消息的数量（可以是`0`或`1`，因为它只能发送一个消息）。

New in Django 1.7:

已添加`html_message`参数。

## send_mass_mail()

`send_mass_mail`(_datatuple_, _fail_silently=False_, _auth_user=None_, _auth_password=None_, _connection=None_)[[source]](../_modules/django/core/mail.html#send_mass_mail)

`django.core.mail.send_mass_mail()`用来处理大批量邮件任务

`datatuple`是一个元组，其中元素的格式如下

```
(subject, message, from_email, recipient_list)

```

`fail_silently`, `auth_user` , `auth_password` 与 [`send_mail()`](#django.core.mail.send_mail "django.core.mail.send_mail")中的方法相同.

`datatuple`的每个单独元素产生单独的电子邮件。在[`send_mail()`](#django.core.mail.send_mail "django.core.mail.send_mail")中，同一`recipient_list`中的收件人将看到电子邮件的“收件人：”字段中的其他地址。

例如，以下代码将向两个不同的收件人集发送两个不同的消息；然而，只有一个到邮件服务器的连接将被打开：

```
message1 = ('Subject here', 'Here is the message', 'from@example.com', ['first@example.com', 'other@example.com'])
message2 = ('Another Subject', 'Here is another message', 'from@example.com', ['second@test.com'])
send_mass_mail((message1, message2), fail_silently=False)

```

返回值将是已成功传递的消息数。

### send_mass_mail()vs. send_mail()

 [`send_mass_mail()`](#django.core.mail.send_mass_mail "django.core.mail.send_mass_mail")和 [`send_mail()`](#django.core.mail.send_mail "django.core.mail.send_mail")的主要差别是 [`send_mail()`](#django.core.mail.send_mail "django.core.mail.send_mail") 每次运行时打开一个到邮箱服务器的连接，而 [`send_mass_mail()`](#django.core.mail.send_mass_mail "django.core.mail.send_mass_mail") 对于所有的信息都只使用一个连接。这使得 [`send_mass_mail()`](#django.core.mail.send_mass_mail "django.core.mail.send_mass_mail") 更高效点.

## mail_admins()

`mail_admins`(_subject_, _message_, _fail_silently=False_, _connection=None_, _html_message=None_)[[source]](../_modules/django/core/mail.html#mail_admins)

`django.core.mail.mail_admins()`是向[`ADMINS`](../ref/settings.html#std:setting-ADMINS)设置中定义的网站管理员发送电子邮件的快捷方式。

`mail_admins()`以[`EMAIL_SUBJECT_PREFIX`](../ref/settings.html#std:setting-EMAIL_SUBJECT_PREFIX)设置的值为主题添加前缀，即`“[Django] t7&gt;`。

电子邮件的“From：”标头将是[`SERVER_EMAIL`](../ref/settings.html#std:setting-SERVER_EMAIL)设置的值。

此方法的存在是为了方便和可读性。

如果提供`html_message`，则生成的电子邮件将是具有`message`作为_text / plain_的_multipart /内容类型和`html_message`作为_text / html_内容类型。_

## mail_managers()

`mail_managers`(_subject_, _message_, _fail_silently=False_, _connection=None_, _html_message=None_)[[source]](../_modules/django/core/mail.html#mail_managers)

`django.core.mail.mail_managers()` is just like `mail_admins()`, except it sends an email to the site managers, as defined in the [`MANAGERS`](../ref/settings.html#std:setting-MANAGERS) setting.

## 例子

这会向[john @示例发送一封电子邮件。](mailto:john%40example.com)com和[jane @示例。](mailto:jane%40example.com)com，他们都出现在“To：”：

```
send_mail('Subject', 'Message.', 'from@example.com',
    ['john@example.com', 'jane@example.com'])

```

这会向[john @示例发送消息。](mailto:john%40example.com)com和[jane @示例。](mailto:jane%40example.com)com，他们都收到一个单独的电子邮件：

```
datatuple = (
    ('Subject', 'Message.', 'from@example.com', ['john@example.com']),
    ('Subject', 'Message.', 'from@example.com', ['jane@example.com']),
)
send_mass_mail(datatuple)

```

## 防止标题注入

[Header injection](http://www.nyphp.org/phundamentals/8_Preventing-Email-Header-Injection)是一个安全漏洞，攻击者插入额外的电子邮件标题控制”TO：“和”FROM：“在你的脚本生成的电子邮件

Django的电子邮件上述的功能概述都是通过标头值禁止换行来防止头注入。 如果任意的 `subject`, `from_email` 或者`recipient_list` 包含一个新行 (in either Unix, Windows or Mac style), 则email功能函数 (e.g. [`send_mail()`](#django.core.mail.send_mail "django.core.mail.send_mail")) 将会引起 `django.core.mail.BadHeaderError` (`ValueError`的子类) 因此，并不会发送邮件。你有责任在把数据发送到email功能函数之前进行验证。

如果一条 `message`字符串包含了一个header开头，这个header将会被简单的打印为这条email的第一个字节。

这里有个简单的例子， `subject`, `message`和 `from_email` 来自于 request’s POST 数据, 发送数据到 [admin@example.](mailto:admin%40example.com)com并在完成后重定向到 “/contact/thanks/” ：

```
from django.core.mail import send_mail, BadHeaderError
from django.http import HttpResponse, HttpResponseRedirect

def send_email(request):
    subject = request.POST.get('subject', '')
    message = request.POST.get('message', '')
    from_email = request.POST.get('from_email', '')
    if subject and message and from_email:
        try:
            send_mail(subject, message, from_email, ['admin@example.com'])
        except BadHeaderError:
            return HttpResponse('Invalid header found.')
        return HttpResponseRedirect('/contact/thanks/')
    else:
        # In reality we'd use a form class
        # to get proper validation errors.
        return HttpResponse('Make sure all fields are entered and valid.')

```

## EmailMessage类

Django的 [`send_mail()`](#django.core.mail.send_mail "django.core.mail.send_mail") 和[`send_mass_mail()`](#django.core.mail.send_mass_mail "django.core.mail.send_mass_mail") 是利用了[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage") class.的简单封装。

并不是所有的 [`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage")类 的特性都可以通过 [`send_mail()`](#django.core.mail.send_mail "django.core.mail.send_mail") 和相关的封装函数来实现的。如果你想用更高级的特性， 比如 BCC’ed recipients, 附件上传, 多部分邮件, 你需要直接创建 [`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage") 实例。

注意

这是一个设计特点。 [`send_mail()`](#django.core.mail.send_mail "django.core.mail.send_mail") 和相关的函数是django最初提供的接口。然而，.参数列表随着时间的推移不断增长。转向面向对象设计的邮件信息，并且保留最初的功能以向后兼任是明智之举。

[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage") 负责创建电子邮件本身。 [_email backend_](#topic-email-backends) 才是负责发送email的。

方便起见， [`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage") 提供了一个简单的 `send()` 方法来发送单一邮件。. 如果你需要发送多份邮件, email的后端 API [_provides an alternative_](#topics-sending-multiple-emails).

### EmailMessage对象

_class_ `EmailMessage`

[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage") 这个类由一下这些参数来实例化 (in the given order, if positional arguments are used). 所有参数都是可选的并且可以在 `send()` 方法之前的任意时间设置。

*   `subject`：电子邮件的主题行。
*   `body`：正文文本。这应该是纯文本消息。
*   `from_email`：发件人的地址。表单合法，`fred@example.com`和`Fred ＆lt； fred@example.com&gt；`如果省略，则使用[`DEFAULT_FROM_EMAIL`](../ref/settings.html#std:setting-DEFAULT_FROM_EMAIL)设置。
*   `to`：收件人地址的列表或元组。
*   `bcc`：发送电子邮件时在“密件抄送”标头中使用的地址列表或元组。
*   `connection`：电子邮件后端实例。如果要对多个消息使用相同的连接，请使用此参数。如果省略，则在调用`send()`时创建新连接。
*   `attachments`：要放在邮件上的附件列表。这些可以是`email.`MIMEBase.MIMEBase实例或`（文件名， 内容， mimetype）`
*   `headers`：一个额外标题的字典放在消息上。键是标题名称，值是标题值。它取决于调用者确保电子邮件消息的头名称和值的格式正确。相应的属性为`extra_headers`。
*   `cc`：发送电子邮件时在“Cc”标头中使用的收件人地址的列表或元组。
*   `reply_to`：发送电子邮件时在“回复”标题中使用的收件人地址的列表或元组。

Changed in Django 1.8:

已添加`reply_to`参数。

例如：

```
email = EmailMessage('Hello', 'Body goes here', 'from@example.com',
            ['to1@example.com', 'to2@example.com'], ['bcc@example.com'],
            reply_to=['another@example.com'], headers={'Message-ID': 'foo'})

```

该类有以下方法：

*   `send(fail_silently=False)`发送消息。如果在构建电子邮件时指定了连接，则将使用该连接。否则，将实例化并使用默认后端的实例。如果关键字参数`fail_silently`是`True`，则发送消息时抛出的异常将被取消。如果接受者为空，这并不会引起异常！

*   `message()`构造一个`django.core.mail.SafeMIMEText`对象（Python的`email.`MIMEText.MIMEText类）或一个`django.core.mail.SafeMIMEMultipart`对象保存要发送的消息。如果您需要扩展[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage")类，那么您可能想要覆盖此方法，将所需的内容放入MIME对象中。

*   `recipients()`返回邮件的所有收件人列表，无论这些收件人是记录在`to`，`cc`或`bcc`属性。这是您在子类化时可能需要覆盖的另一种方法，因为在发送消息时，需要告知SMTP服务器收件人的完整列表。如果您添加另一种方式来指定类中的收件人，则还需要从此方法返回。

*   `attach()`创建一个新文件附件并将其添加到消息中。有两种方法调用`attach()`：

    *   您可以向其传递一个为`email.`MIMEBase.MIMEBase实例。这将直接插入到生成的消息中。

    *   或者，您可以传递`attach()`三个参数：`filename`，`content`和`mimetype`。`filename`是电子邮件中显示的文件附件的名称，`content`是附件中包含的数据，`mimetype`是附件的可选MIME类型。如果你忽略了 `mimetype`,  MIME content type将会从你的文件名来猜测。。

        例如：

        ```
        message.attach('design.png', img_data, 'image/png')

        ```

        Changed in Django 1.7:

        如果您指定`message/rfc822`的`mimetype`，它也会接受[`django.core.mail.EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage")和[`email.message.Message`](https://docs.python.org/3/library/email.message.html#email.message.Message "(in Python v3.4)")。

        此外，`message/rfc822`附件将不再以违反 [**RFC 2046**](http://tools.ietf.org/html/rfc2046.html#section-5.2.1)的base64编码，这可能会导致显示附件的问题[进化](https://bugzilla.gnome.org/show_bug.cgi?id=651197)和[Thunderbird](https://bugzilla.mozilla.org/show_bug.cgi?id=333880)。

*   `attach_file()`使用文件系统中的文件创建新附件。使用要附加的文件的路径和可选的用于附件的MIME类型调用它。如果省略MIME类型，则将从文件名中猜出。最简单的用法是：

    ```
    message.attach_file('/images/weather_map.png')

    ```

#### 发送替代内容类型

在你要包含各种乱七八糟的内容在邮件内容时，这非常的有用; 最经典的案例就是发送text和HTML的信息版本。用Django的 email库, 你可以用 `EmailMultiAlternatives` 这个类来完成。这个类是 [`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage") 的子类，它有 `attach_alternative()` 来包含额外的邮件信息版本。所有其他方法（包括类初始化）直接从[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage")继承。

要发送文本和HTML组合，您可以写：

```
from django.core.mail import EmailMultiAlternatives

subject, from_email, to = 'hello', 'from@example.com', 'to@example.com'
text_content = 'This is an important message.'
html_content = '<p>This is an <strong>important</strong> message.</p>'
msg = EmailMultiAlternatives(subject, text_content, from_email, [to])
msg.attach_alternative(html_content, "text/html")
msg.send()

```

默认情况下，[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage")中`body`参数的MIME类型为`"text/plain"`。

```
msg = EmailMessage(subject, html_content, from_email, [to])
msg.content_subtype = "html"  # Main content is now text/html
msg.send()

```

## 邮件后端

邮件的真正发送是通过邮件后端处理的。

邮件后端类具有以下方法：

*   `open()` 实例化一个用于发送邮件的长连接。
*   `close()` 关闭当前的邮件发送连接。
*   `send_messages(email_messages)` 发送一个[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage") 对象的列表。如果连接没有打开，该调用将隐式打开连接，并在之后关闭连接。如果连接已经打开，它在邮件发送之后会保持打开状态。

它还可以用作上下文管理器，在需要的时候自动调用`open()` 和 `close()`：

```
from django.core import mail

with mail.get_connection() as connection:
    mail.EmailMessage(subject1, body1, from1, [to1],
                      connection=connection).send()
    mail.EmailMessage(subject2, body2, from2, [to2],
                      connection=connection).send()

```

New in Django 1.8:

新增上下文管理器协议。

### 获取邮件后端的一个实例

位于`django.core.mail` 中的[`get_connection()`](#django.core.mail.get_connection "django.core.mail.get_connection") 函数返回邮件后端的一个实例。

`get_connection`(_backend=None_, _fail_silently=False_, _*args_, _**kwargs_)[[source]](../_modules/django/core/mail.html#get_connection)

默认情况下，对`get_connection()` 的调用将返回[`EMAIL_BACKEND`](../ref/settings.html#std:setting-EMAIL_BACKEND) 指定的邮件后端实例。如果你指定`backend` 参数，则返回相应的后端的一个实例。

`fail_silently` 参数控制后端如何处理错误。如果`fail_silently` 为True，邮件发送过程中的异常将会默默忽略。

所有其它的参数都直接传递给邮件后端的构造函数。

Django 自带几个邮件发送的后端。除了SMTP 后端（默认），其它后端只用于测试和开发阶段。如果发送邮件有特殊的需求，你可以[_编写自己的邮件后端_](#topic-custom-email-backend)。

#### SMTP 后端

_class_ `backends.smtp.``EmailBackend`([_host=None_, _port=None_, _username=None_, _password=None_, _use_tls=None_, _fail_silently=False_, _use_ssl=None_, _timeout=None_, _ssl_keyfile=None_, _ssl_certfile=None_, _**kwargs_])

这是默认的后端。邮件将通过SMTP 服务器发送。

每个参数的值如果为`None`，则从settings 中获取对应的设置：

*   `host`：[`EMAIL_HOST`](../ref/settings.html#std:setting-EMAIL_HOST)
*   `port`：[`EMAIL_PORT`](../ref/settings.html#std:setting-EMAIL_PORT)
*   `username`：[`EMAIL_HOST_USER`](../ref/settings.html#std:setting-EMAIL_HOST_USER)
*   `password`：[`EMAIL_HOST_PASSWORD`](../ref/settings.html#std:setting-EMAIL_HOST_PASSWORD)
*   `use_tls`：[`EMAIL_USE_TLS`](../ref/settings.html#std:setting-EMAIL_USE_TLS)
*   `use_ssl`：[`EMAIL_USE_SSL`](../ref/settings.html#std:setting-EMAIL_USE_SSL)
*   `timeout`：[`EMAIL_TIMEOUT`](../ref/settings.html#std:setting-EMAIL_TIMEOUT)
*   `ssl_keyfile`：[`EMAIL_SSL_KEYFILE`](../ref/settings.html#std:setting-EMAIL_SSL_KEYFILE)
*   `ssl_certfile`：[`EMAIL_SSL_CERTFILE`](../ref/settings.html#std:setting-EMAIL_SSL_CERTFILE)

SMTP 后端是Django 内在的默认配置。如果你想显示指定，可以将下面这行放到settings 中：

```
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'

```

New in Django 1.7:

添加`timeout` 参数。如果没有指定，默认的`timeout` 为[`socket.getdefaulttimeout()`](https://docs.python.org/3/library/socket.html#socket.getdefaulttimeout "(in Python v3.4)") 提供的值，它默认是`None`（不会超时）。

Changed in Django 1.8:

添加`ssl_keyfile` 和`ssl_certfile` 参数以及对应的settings。添加([`EMAIL_TIMEOUT`](../ref/settings.html#std:setting-EMAIL_TIMEOUT)) 设置以使得可以自定义`timeout`。

#### Console 后端

Console 后端不真正发送邮件，而只是向标准输出打印出邮件。默认情况下，console 后端打印到`stdout`。你可以通过在构造connection 时提供`stream` 参数，来使用一个不同的流对象。

如要指定这个后端，可以将下面这行放入你的settings 中：

```
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

```

这个后端不能用于生产环境 —— 它只是为了开发方便。

#### File 后端

File 后端将邮件写到文件中。该后端的每个会话将创建一个新的文件。文件的目录从[`EMAIL_FILE_PATH`](../ref/settings.html#std:setting-EMAIL_FILE_PATH) 设置或者[`get_connection()`](#django.core.mail.get_connection "django.core.mail.get_connection") 的`file_path` 关键字参数获取。

如要指定这个后端，可以将下面这行放入你的settings 中：

```
EMAIL_BACKEND = 'django.core.mail.backends.filebased.EmailBackend'
EMAIL_FILE_PATH = '/tmp/app-messages' # change this to a proper location

```

这个后端不能用于生成环境 —— 它只是为了开发方便。

#### In-memory 后端

`'locmem'` 后端将邮件保存在`django.core.mail` 模块的一个特殊属性中。当一封邮件发送时将创建`outbox` 属性。它是[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage") 实例的一个列表，表示每个将要发送的邮件。

如要指定这个后端，可以将下面这行放入你的settings 中：

```
EMAIL_BACKEND = 'django.core.mail.backends.locmem.EmailBackend'

```

这个后端不能用于生成环境 —— 它只是为了开发和测试方便。

#### Dummy 后端

和名字一样，dummy 后端什么也不做。如要指定这个后端，可以将下面这行放入你的settings 中：

```
EMAIL_BACKEND = 'django.core.mail.backends.dummy.EmailBackend'

```

这个后端不能用于生成环境 —— 它只是为了开发方便。

### 定义自定义电子邮件后端

如果您需要更改电子邮件的发送方式，您可以编写自己的电子邮件后端。您的设置文件中的[`EMAIL_BACKEND`](../ref/settings.html#std:setting-EMAIL_BACKEND)设置是您的后端类的Python导入路径。

自定义电子邮件后端应该位于`django.core.mail.backends.base`模块中的`BaseEmailBackend`。自定义电子邮件后端必须实施`send_messages(email_messages)`方法。此方法接收[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage")实例的列表，并返回已成功传递的邮件数。如果您的后端有任何持续会话或连接的概念，您还应该实现`open()`和`close()`方法。请参阅`smtp.`EmailBackend用于参考实现。

### 正在发送多封电子邮件

建立和关闭SMTP连接（或任何其他网络连接，这方面）是一个昂贵的过程。如果您有大量电子邮件要发送，则重用SMTP连接是有意义的，而不是每次要发送电子邮件时创建和销毁连接。

有两种方法可以让电子邮件后端重复使用连接。

首先，您可以使用`send_messages()`方法。`send_messages()`获取[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage")实例（或子类）的列表，并使用单个连接发送它们。

例如，如果您有一个名为`get_notification_email()`的函数，该函数返回表示您希望发送的某些周期性电子邮件的[`EmailMessage`](#django.core.mail.EmailMessage "django.core.mail.EmailMessage")对象列表，则可以使用单次调用send_messages：

```
from django.core import mail
connection = mail.get_connection()   # Use default email connection
messages = get_notification_email()
connection.send_messages(messages)

```

在此示例中，对`send_messages()`的调用在后端打开一个连接，发送消息列表，然后再次关闭连接。

第二种方法是使用电子邮件后端上的`open()`和`close()`方法手动控制连接。`send_messages()`将不会手动打开或关闭已打开的连接，因此如果您手动打开连接，则可以控制它何时关闭。例如：

```
from django.core import mail
connection = mail.get_connection()

# Manually open the connection
connection.open()

# Construct an email message that uses the connection
email1 = mail.EmailMessage('Hello', 'Body goes here', 'from@example.com',
                          ['to1@example.com'], connection=connection)
email1.send() # Send the email

# Construct two more messages
email2 = mail.EmailMessage('Hello', 'Body goes here', 'from@example.com',
                          ['to2@example.com'])
email3 = mail.EmailMessage('Hello', 'Body goes here', 'from@example.com',
                          ['to3@example.com'])

# Send the two emails in a single call -
connection.send_messages([email2, email3])
# The connection was already open so send_messages() doesn't close it.
# We need to manually close the connection.
connection.close()

```

## 配置用于开发的电子邮件

有时候你不想让Django发送电子邮件。例如，在开发网站时，您可能不想发送数千封电子邮件，但您可能需要验证电子邮件是否会在正确的条件下发送给正确的人员，并且这些电子邮件将包含正确的内容。

为本地开发配置电子邮件的最简单方法是使用[_console_](#topic-email-console-backend)电子邮件后端。此后端将所有电子邮件重定向到stdout，允许您检查邮件的内容。

[_file_](#topic-email-file-backend)电子邮件后端在开发过程中也可以是有用的 - 这个后端将每个SMTP连接的内容转储到可以随时检查的文件。

另一种方法是使用“哑”SMTP服务器，在本地接收电子邮件并将其显示到终端，但实际上不发送任何内容。Python有一个内置的方法来完成这个使用一个单一的命令：

```
python -m smtpd -n -c DebuggingServer localhost:1025

```

此命令将启动一个简单的SMTP服务器侦听localhost的端口1025。此服务器只打印标准输出所有电子邮件标头和电子邮件正文。然后，您只需相应地设置[`EMAIL_HOST`](../ref/settings.html#std:setting-EMAIL_HOST)和[`EMAIL_PORT`](../ref/settings.html#std:setting-EMAIL_PORT)。有关SMTP服务器选项的更详细的讨论，请参阅[`smtpd`](https://docs.python.org/3/library/smtpd.html#module-smtpd "(in Python v3.4)")模块的Python文档。

有关在应用程序中单元测试电子邮件发送的信息，请参阅测试文档的[_Email services_](testing/tools.html#topics-testing-email)部分。

