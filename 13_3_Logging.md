# 日志

## 日志快速入门

Django 使用Python 内建的[`logging`](https://docs.python.org/3/library/logging.html#module-logging) 模块打印日志。该模块的用法在Python 本身的文档中有详细的讨论。如果你从来没有使用过Python 的logging 框架（或者即使使用过），请参见下面的快速导论。

### logging 的组成

Python 的logging 配置由四个部分组成：

*   [Loggers](#topic-logging-parts-loggers)
*   [Handlers](#topic-logging-parts-handlers)
*   [Filters](#topic-logging-parts-filters)
*   [Formatters](#topic-logging-parts-formatters)

#### Loggers

Logger 为日志系统的入口。每个logger 是一个具名的容器，可以向它写入需要处理的消息。

每个logger 都有一个_日志级别_。日志级别表示该logger 将要处理的消息的严重性。Python 定义以下几种日志级别：

*   `DEBUG`：用于调试目的的底层系统信息
*   `INFO`：普通的系统信息
*   `WARNING`：表示出现一个较小的问题。
*   `ERROR`：表示出现一个较大的问题。
*   `CRITICAL`：表示出现一个致命的问题。

写入logger 的每条消息都是一个_日志记录_。每个日志记录也具有一个_日志级别_，它表示对应的消息的严重性。每个日志记录还可以包含描述正在打印的事件的有用元信息。这些元信息可以包含很多细节，例如回溯栈或错误码。

当给一条消息给logger 时，会将消息的日志级别与logger 的日志级别进行比较。如果消息的日志级别大于等于logger 的日志级别，该消息将会往下继续处理。如果小于，该消息将被忽略。

Logger 一旦决定消息需要处理，它将传递该消息给一个_Handler_。

#### Handlers

Handler 决定如何处理logger 中的每条消息。它表示一个特定的日志行为，例如将消息写到屏幕上、写到文件中或者写到网络socket。

与logger 一样，handler 也有一个日志级别。如果消息的日志级别小于handler 的级别，handler 将忽略该消息。

Logger 可以有多个handler，而每个handler 可以有不同的日志级别。利用这种方式，可以根据消息的重要性提供不同形式的处理。例如，你可以用一个handler 将`ERROR` 和 `CRITICAL` 消息发送给一个页面服务，而用另外一个hander 将所有的消息（包括 `ERROR` 和`CRITICAL` 消息）记录到一个文件中用于以后进行分析。

#### Filters

Filter 用于对从logger 传递给handler 的日志记录进行额外的控制。

默认情况下，满足日志级别的任何消息都将被处理。通过安装一个filter，你可以对日志处理添加额外的条件。例如，你可以安装一个filter，只允许处理来自特定源的`ERROR` 消息。

Filters 还可以用于修改将要处理的日志记录的优先级。例如，如果日志记录满足特定的条件，你可以编写一个filter 将日志记录从`ERROR` 降为`WARNING`。

Filters 可以安装在logger 上或者handler 上；多个filter 可以串联起来实现多层filter 行为。

#### Formatters

最后，日志记录需要转换成文本。Formatter 表示文本的格式。Fomatter 通常由包含[_日志记录属性_](https://docs.python.org/3/library/logging.html#logrecord-attributes)的Python 格式字符串组成；你也可以编写自定义的fomatter 来实现自己的格式。

## 使用logging

配置好logger、handler、filter 和formatter 之后，你需要在代码中放入logging 调用。使用logging 框架非常简单。下面是个例子：

```
# import the logging library
import logging

# Get an instance of a logger
logger = logging.getLogger(__name__)

def my_view(request, arg1, arg):
    ...
    if bad_mojo:
        # Log an error message
        logger.error('Something went wrong!')

```

就是这样！每次满足`bad_mojo` 条件，将写入一条错误日志记录。

### 命名logger

[`logging.getLogger()`](https://docs.python.org/3/library/logging.html#logging.getLogger) 调用获取（如有必要则创建）一个logger 的实例。Logger 实例通过名字标识。Logger 使用名称的目的是用于标识其配置。

Logger 的名称习惯上通常使用`__name__`，即包含该logger 的Python 模块的名字。这允许你基于模块filter 和handle 日志调用。如果你想使用其它方式组织日志消息，可以提供点号分隔的名称来标识你的logger：

```
# Get an instance of a specific named logger
logger = logging.getLogger('project.interesting.stuff')

```

点号分隔的logger 名称定义一个层级。`project.interesting` logger 被认为是 `project.interesting.stuff` logger 的上一级；`project` logger 是`project.interesting` logger 的上一级。

层级为何如此重要？因为可以设置logger _传播_它们的logging 调用给它们的上一级。利用这种方式，你可以在根logger 上定义一系列的handler，并捕获子logger 中的所有logging 调用。在`project`命名空间中定义的handler 将捕获`project.interesting` 和`project.interesting.stuff` logger 上的所有日志消息。

这种传播行为可以基于每个logger 进行控制。如果你不想让某个logger 传播消息给它的上一级，你可以关闭这个行为。

### logging 调用

Logger 实例为每个默认的日志级别提供一个入口方法：

*   `logger.debug()`
*   `logger.info()`
*   `logger.warning()`
*   `logger.error()`
*   `logger.critical()`

还有另外两个调用：

*   `logger.log()`：打印消息时手工指定日志级别。
*   `logger.exception()`：创建一个`ERROR` 级别日志消息，它封装当前异常栈的帧。

## 配置logging

当然，只是将logging 调用放入你的代码中还是不够的。你还需要配置logger、handler、filter 和formatter 来确保日志的输出是有意义的。

Python 的logging 库提供几种配置logging 的技术，从程序接口到配置文件。默认情况下，Django 使用[dictConfig 格式](https://docs.python.org/library/logging.config.html#configuration-dictionary-schema)。

为了配置logging，你需要使用[`LOGGING`](../ref/settings.html#std:setting-LOGGING) 来定义字典形式的logging 设置。这些设置描述你的logging 设置的logger、handler、filter 和formatter，以及它们的日志等级和其它属性。

默认情况下，[`LOGGING`](../ref/settings.html#std:setting-LOGGING) 设置与[_Django 的默认logging 配置_](#default-logging-configuration)进行合并。

如果[`LOGGING`](../ref/settings.html#std:setting-LOGGING) 中的`disable_existing_loggers` 键为`True`（默认值），那么默认配置中的所有logger 都将禁用。Logger 的禁用与删除不同；logger 仍然存在，但是将默默丢弃任何传递给它的信息，也不会传播给上一级logger。所以，你应该非常小心使用`'disable_existing_loggers': True`；它可能不是你想要的。你可以设置`disable_existing_loggers` 为`False`，并重新定义部分或所有的默认loggers；或者你可以设置[`LOGGING_CONFIG`](../ref/settings.html#std:setting-LOGGING_CONFIG) 为 `None`，并 [_自己处理logging 配置_](#disabling-logging-configuration)。

Logging 的配置属于Django `setup()` 函数的一部分。所以，你可以肯定在你的项目代码中logger 是永远可用的。

### 示例

[dictConfig 格式](https://docs.python.org/library/logging.config.html#configuration-dictionary-schema)的完整文档是logging 字典配置最好的信息源。但是为了让你尝尝，下面是几个例子。

首先，下面是一个简单的配置，它将来自[_django.request_](#django-request-logger) logger 的所有日志请求写入到一个本地文件：

```
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'DEBUG',
            'class': 'logging.FileHandler',
            'filename': '/path/to/django/debug.log',
        },
    },
    'loggers': {
        'django.request': {
            'handlers': ['file'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}

```

如果你使用这个示例，请确保修改`'filename'` 路径为运行Django 应用的用户有权限写入的一个位置。

其次，下面这个示例演示如何让日志系统将Django 的日志打印到控制台。`django.request` 和`django.security` 不会传播日志给上一级。它在本地开发期间可能有用。

默认情况下，这个配置只会将`INFO` 和更高级别的日志发送到控制台。Django 中这样的日志信息不多。可以设置环境变量`DJANGO_LOG_LEVEL=DEBUG` 来看看Django 的debug 日志，它包含所有的数据库查询所以非常详尽。

```
import os

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['console'],
            'level': os.getenv('DJANGO_LOG_LEVEL', 'INFO'),
        },
    },
}

```

最后，下面是相当复杂的一个logging 设置：

```
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(process)d %(thread)d %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(message)s'
        },
    },
    'filters': {
        'special': {
            '()': 'project.logging.SpecialFilter',
            'foo': 'bar',
        }
    },
    'handlers': {
        'null': {
            'level': 'DEBUG',
            'class': 'logging.NullHandler',
        },
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
            'filters': ['special']
        }
    },
    'loggers': {
        'django': {
            'handlers': ['null'],
            'propagate': True,
            'level': 'INFO',
        },
        'django.request': {
            'handlers': ['mail_admins'],
            'level': 'ERROR',
            'propagate': False,
        },
        'myproject.custom': {
            'handlers': ['console', 'mail_admins'],
            'level': 'INFO',
            'filters': ['special']
        }
    }
}

```

这个logging 配置完成以下事情：

*   以‘dictConfig version 1’格式解析配置。目前为止，这是dictConfig 格式唯一的版本。

*   定义两个formatter：

    *   `simple`，它只输出日志的级别（例如，`DEBUG`）和日志消息。

        `format` 字符串是一个普通的Python 格式化字符串，描述每行日志的细节。输出的完整细节可以在[formatter 文档](https://docs.python.org/library/logging.html#formatter-objects)中找到。

    *   `verbose`，它输出日志级别、日志消息，以及时间、进程、线程和生成日志消息的模块。

*   定义filter —— `project.logging.SpecialFilter`，并使用别名`special`。如果filter 在构造时要求额外的参数，可以在filter 的配置字段中用额外的键提供。在这个例子中，在实例化`SpecialFilter` 时，`foo` 参数的值将使用`bar`。

*   定义三个handler：

    *   `null`，一个NullHandler，它传递`DEBUG`（和更高级）的消息给`/dev/null`。
    *   `console`，一个StreamHandler，它将打印`DEBUG`（和更高级）的消息到stderr。这个handler 使用`simple` 输出格式。
    *   `mail_admins`，一个AdminEmailHandler，它将用邮件发送`ERROR`（和更高级）的消息到站点管理员。这个handler 使用`special` filter。

*   配置三个logger：

    *   `django`，它传递所有`INFO` 和更高级的消息给`null` handler。
    *   `django.request`，它传递所有`ERROR` 消息给`mail_admins` handler。另外，标记这个logger _不_ 向上传播消息。这表示写入`django.request` 的日志信息将不会被`django` logger 处理。
    *   `myproject.custom`，它传递所有`INFO` 和更高级的消息并通过`special` filter 的消息给两个handler —— `console`和`mail_admins`。这表示所有`INFO`（和更高级）的消息将打印到控制台上；`ERROR` 和`CRITICAL` 消息还会通过邮件发送出来。

### 自定义logging 配置

如果你不想使用Python 的dictConfig 格式配置logger，你可以指定你自己的配置模式。

[`LOGGING_CONFIG`](../ref/settings.html#std:setting-LOGGING_CONFIG) 设置定义一个可调用对象，将它用来配置Django 的logger。默认情况下，它指向Python 的[`logging.config.dictConfig()`](https://docs.python.org/3/library/logging.config.html#logging.config.dictConfig) 函数。但是，如果你想使用不同的配置过程，你可以使用其它只接受一个参数的可调用对象。配置logging 时，将使用[`LOGGING`](../ref/settings.html#std:setting-LOGGING) 的内容作为参数的值。

### 禁用logging 配置

如果你完全不想配置logging（或者你想使用自己的方法手工配置logging），你可以设置[`LOGGING_CONFIG`](../ref/settings.html#std:setting-LOGGING_CONFIG) 为`None`。这将禁用[_Django 默认logging_](#default-logging-configuration) 的配置过程。下面的示例禁用Django 的logging 配置，然后手工配置logging：

settings.py
```
LOGGING_CONFIG = None

import logging.config
logging.config.dictConfig(...)

```

设置[`LOGGING_CONFIG`](../ref/settings.html#std:setting-LOGGING_CONFIG) 为`None` 只表示禁用自动配置过程，而不是禁用logging 本身。如果你禁用配置过程，Django 仍然执行logging 调用，只是调用的是默认定义的logging 行为。

## Django’s logging extensions

Django 提供许多工具用于处理在网站服务器环境中独特的日志需求。

### Loggers

Django 提供几个内建的logger。

#### django

`django` 是一个捕获所有信息的logger。消息不会直接提交给这个logger。

#### django.request

记录与处理请求相关的消息。5XX 响应作为`ERROR` 消息；4XX 响应作为`WARNING` 消息。

这个logger 的消息具有以下额外的上下文：

*   `status_code`：请求的HTTP 响应码。
*   `request`：生成日志信息的请求对象。

#### django.db.backends

与数据库交互的代码相关的消息。例如，HTTP请求执行应用级别的SQL 语句将以`DEBUG` 级别记录到该logger。

这个logger 的消息具有以下额外的上下文：

*   `duration`：执行SQL 语句花费的时间。
*   `sql`：执行的SQL 语句。
*   `params`：SQL 调用中用到的参数。

由于性能原因，SQL的日志只在`设置`之后开启。DEBUG 设置为`True`，无论日志级别或者安装的处理器是什么。

这里的日志不包含框架级别的的初始化（例如，`SET TIMEZONE`）和事务管理查询（例如，`BEGIN`、`COMMIT` 和`ROLLBACK`）。如果你希望看到所有的数据库查询，可以打开数据库中的查询日志。

#### django.security.*

Security logger 将收到任何出现[`SuspiciousOperation`](../ref/exceptions.html#django.core.exceptions.SuspiciousOperation "django.core.exceptions.SuspiciousOperation") 的消息。SuspiciousOperation 的每个子类型都有一个子logger。日志的级别取决于异常处理的位置。大部分情况是一个warning 日志，而如果`SuspiciousOperation` 到达WSGI handler 则记录为一个error。例如，如果请求中包含的HTTP `Host` 头部与[`ALLOWED_HOSTS`](../ref/settings.html#std:setting-ALLOWED_HOSTS) 不匹配，Django 将返回400 响应，同时将记录一个error 消息到`django.security.DisallowedHost` logger。

默认情况下只会配置`django.security` logger，其它所有的子logger 都将传播给上一级logger。`django.security` logger 的配置与`django.request` logger 相同，任何error 消息将用邮件发送给站点管理员。由于`SuspiciousOperation` 导致400 响应的请求不会在`django.request` logger 中记录日志，而只在`django.security` logger 中记录日志。

若要默默丢弃某种类型的SuspiciousOperation，你可以按照下面的示例覆盖其logger：

```
'loggers': {
    'django.security.DisallowedHost': {
        'handlers': ['null'],
        'propagate': False,
    },
},

```

#### django.db.backends.schema

New in Django 1.7\.

当[_迁移框架_](migrations.html)执行的SQL 查询会改变数据库的模式时，则记录这些SQL 查询。注意，它不会记录[`RunPython`](../ref/migration-operations.html#django.db.migrations.operations.RunPython "django.db.migrations.operations.RunPython") 执行的查询。

### Handlers

在Python logging 模块提供的handler 基础之上，Django 还提供另外一个handler。

_class _`AdminEmailHandler`(_include_html=False_, _email_backend=None_)[[source]](../_modules/django/utils/log.html#AdminEmailHandler)

这个handler 将它收到的每个日志信息用邮件发送给站点管理员。

如果日志记录包含`request` 属性，该请求的完整细节都将包含在邮件中。

如果日志记录包含栈回溯信息，该栈回溯也将包含在邮件中。

`AdminEmailHandler` 的`include_html` 参数用于控制邮件中是否包含HTML 附件，这个附件包含[`DEBUG`](../ref/settings.html#std:setting-DEBUG) 为`True` 时的完整网页。若要在配置中设置这个值，可以将它包含在`django.utils.log.AdminEmailHandler` handler 的定义中，像下面这样：

```
'handlers': {
    'mail_admins': {
        'level': 'ERROR',
        'class': 'django.utils.log.AdminEmailHandler',
        'include_html': True,
    }
},

```

注意，邮件中的HTML 包含完整的回溯栈，包括栈每个层级局部变量的名称和值以及你的Django 设置。这些信息可能非常敏感，你也许不想通过邮件发送它们。此时可以考虑使用类似[Sentry](https://pypi.python.org/pypi/sentry) 这样的东西，回溯栈的完整信息和安全信息_不会_ 通过邮件发送。你还可以从错误报告中显式过滤掉特定的敏感信息 —— 更多信息参见[_过滤错误报告_](../howto/error-reporting.html#filtering-error-reports)。

通过设置`AdminEmailHandler` 的`email_backend` 参数，可以覆盖handler 使用的[_email backend_](email.html#topic-email-backends)，像这样：

```
'handlers': {
    'mail_admins': {
        'level': 'ERROR',
        'class': 'django.utils.log.AdminEmailHandler',
        'email_backend': 'django.core.mail.backends.filebased.EmailBackend',
    }
},

```

默认情况下，将使用[`EMAIL_BACKEND`](../ref/settings.html#std:setting-EMAIL_BACKEND) 中指定的邮件后端。

`send_mail`(_subject_, _message_, _*args_, _**kwargs_)[[source]](../_modules/django/utils/log.html#AdminEmailHandler.send_mail)

New in Django 1.8\.

发送邮件给管理员用户。若要自定它的行为，可以子类化[`AdminEmailHandler`](#django.utils.log.AdminEmailHandler "django.utils.log.AdminEmailHandler") 类并覆盖这个方法。

### Filters

在Python logging 模块提供的过滤器的基础之上，Django 还提供两个过滤器。

_class _`CallbackFilter`(_callback_)[[source]](../_modules/django/utils/log.html#CallbackFilter)

这个过滤器接受一个回调函数（它接受一个单一参数，也就是要记录的东西），并且对每个传递给过滤器的记录调用它。如果回调函数返回False，将不会进行记录的处理。

例如，要从admin邮件中过滤掉[`UnreadablePostError`](../ref/exceptions.html#django.http.UnreadablePostError "django.http.UnreadablePostError")（只在用户取消上传时产生），你可以创建一个过滤器函数：

```
from django.http import UnreadablePostError

def skip_unreadable_post(record):
    if record.exc_info:
        exc_type, exc_value = record.exc_info[:2]
        if isinstance(exc_value, UnreadablePostError):
            return False
    return True

```

然后把它添加到logger的配置中：

```
'filters': {
    'skip_unreadable_posts': {
        '()': 'django.utils.log.CallbackFilter',
        'callback': skip_unreadable_post,
    }
},
'handlers': {
    'mail_admins': {
        'level': 'ERROR',
        'filters': ['skip_unreadable_posts'],
        'class': 'django.utils.log.AdminEmailHandler'
    }
},

```

_class _`RequireDebugFalse`[[source]](../_modules/django/utils/log.html#RequireDebugFalse)

这个过滤器只在设置后传递记录。DEBUG 为 False。

这个过滤器遵循[`LOGGING`](../ref/settings.html#std:setting-LOGGING) 默认的配置，以确保[`AdminEmailHandler`](#django.utils.log.AdminEmailHandler "django.utils.log.AdminEmailHandler")只在[`DEBUG`](../ref/settings.html#std:setting-DEBUG)为`False`的时候发送错误邮件。

```
'filters': {
    'require_debug_false': {
        '()': 'django.utils.log.RequireDebugFalse',
    }
},
'handlers': {
    'mail_admins': {
        'level': 'ERROR',
        'filters': ['require_debug_false'],
        'class': 'django.utils.log.AdminEmailHandler'
    }
},

```

_class _`RequireDebugTrue`[[source]](../_modules/django/utils/log.html#RequireDebugTrue)

这个过滤器类似于[`RequireDebugFalse`](#django.utils.log.RequireDebugFalse "django.utils.log.RequireDebugFalse")，除了记录只在[`DEBUG`](../ref/settings.html#std:setting-DEBUG) 为 `True`时传递的情况。

## Django’s default logging configuration

默认情况下，Django 的logging 配置如下：

当[`DEBUG`](../ref/settings.html#std:setting-DEBUG) 为`True` 时：

*   `django`的全局logger会向控制台发送级别等于或高级`INFO`的所有消息。Django在这个时候并不会做任何日志调用（所有在`DEBUG`级别上的日志，或者被`django.request` 和 `django.security`处理的日志）。
*   `py.warnings` logger，它处理来自`warnings.warn()`的消息，会向控制台发送消息。

当[`DEBUG`](../ref/settings.html#std:setting-DEBUG) 为`False` 时：

*   `django.request` 和`django.security` loggers 向[`AdminEmailHandler`](#django.utils.log.AdminEmailHandler "django.utils.log.AdminEmailHandler")发送带有`ERROR` 或 `CRITICAL`级别的消息。这些logger 会忽略任何级别等于或小于`WARNING`的信息，被记录的日志不会传递给其他logger（它们不会传递给`django`的全局 logger，即使`DEBUG` 为 `True`)。

另见[_配置日志_](#configuring-logging)来了解如何补充或者替换默认的日志配置。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Logging](https://docs.djangoproject.com/en/1.8/topics/logging/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
