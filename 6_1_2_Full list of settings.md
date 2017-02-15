{% raw %}

# 设置

*   [核心配置](#core-settings)
*   [权限](#auth)
*   [消息](#messages)
*   [会话](#sessions)
*   [站点](#sites)
*   [静态文件](#static-files)
*   [核心设置和主题索引](#core-settings-topical-index)

警告

当改变设置的时候你一定要小心，尤其当默认值是一个非空元组或者一个字典的时候，比如 [`MIDDLEWARE_CLASSES`](#std:setting-MIDDLEWARE_CLASSES) 和 [`STATICFILES_FINDERS`](#std:setting-STATICFILES_FINDERS). 确保组件符合Django的特性，你想使用的话。

## 核心配置

这里是一些Django的核心设置和它们的默认值。由contrib apps提供的设置，它的主题索引在下面列出。对于介绍材料， 请看 [_settings topic guide_](../topics/settings.html).

### ABSOLUTE_URL_OVERRIDES

Default: `{}` (默认为空的字典)

这个字典把`"app_label.model_name"`字符串映射到带着一个模型对象的函数上并返回一个URL。这是一种在每次安装的基础上插入或覆盖`get_absolute_url()`方法的方法。例：

```
ABSOLUTE_URL_OVERRIDES = {
    'blogs.weblog': lambda o: "/blogs/%s/" % o.slug,
    'news.story': lambda o: "/stories/%s/%s/" % (o.pub_year, o.slug),
}

```

注意这里设置中使用的模型对象名称一定要小写，与模型的类名的实际情况无关

Changed in Django 1.7.1:

`ABSOLUTE_URL_OVERRIDES`现在适用于未声明`get_absolute_url()`的模型。

### ADMINS

默认值：`()`（空元组）

一个错误码消息数组当 `DEBUG=False` ，并且一个视图引发了异常, Django 会给这些人发一封含有完整异常信息的电子邮件。元组的每个成员应该是一个(姓名全称，电子邮件地址)的元组 . 例：

```
(('John', 'john@example.com'), ('Mary', 'mary@example.com'))

```

注意无论何时发生错误Django都会向这里的所有人发送邮件_all_ 。查看[_Error reporting_](../howto/error-reporting.html) 了解更多信息。

### ALLOWED_HOSTS

默认值：`[]`（空列表）

一个可以被Django站点提供提供服务的主机/域名的字符串名单。这是一个防御攻击者的措施，攻击会来源于缓存中毒然后密码被重置，并通过提交一个伪造了`Host` header(主机头信息)的密码重置请求使得邮箱被链接到恶意主机，这是有可能发生的，即使在很多看似安全的web服务器配置中。

列表中的值要是完全合格的名称 (e.g. `'www.example.com'`), 这种情况下，他们将会正确地匹配请求的`Host` header (忽略大小写，不包括端口).。开始处的英文句号能够用于作为子域名的通配符: `'.example.com'` 会匹配`example.com`, `www.example.com`, 以及任何`example.com`. 的子域名。 `'*'`会匹配任何的值;在这种情况中，你务必要提供你自己的`Host` header的验证 (也可以是在中间件中，如果这样的话，中间件要首先被列入在[`MIDDLEWARE_CLASSES`](#std:setting-MIDDLEWARE_CLASSES)中).。

Changed in Django 1.7:

在先前的Django的版本中，如果你打算也要遵循[fully qualified domain name (FQDN)](http://en.wikipedia.org/wiki/Fully_qualified_domain_name)(完全合格的域名名称), 一些浏览器会发送在 `Host` header(头标信息)中, 你要显示地添加另一个英文句号结尾的`ALLOWED_HOSTS`.条目。这个条目也可以是个子域名通配符：

```
ALLOWED_HOSTS = [
    '.example.com',  # Allow domain and subdomains
    '.example.com.',  # Also allow FQDN and subdomains
]

```

在Django1.7中，末尾的点在执行主机验证的时候是被去掉的，因此一个条目没必要在末尾带点。

如果`Host` header(头信息)(或者是`X-Forwarded-Host` 如果[`USE_X_FORWARDED_HOST`](#std:setting-USE_X_FORWARDED_HOST)被启用的话) 不匹配这个列表中的任何值, 那么 [`django.http.HttpRequest.get_host()`](request-response.html#django.http.HttpRequest.get_host "django.http.HttpRequest.get_host") 函数将会抛出一个[`SuspiciousOperation`](exceptions.html#django.core.exceptions.SuspiciousOperation "django.core.exceptions.SuspiciousOperation")异常.

当 [`DEBUG`](#std:setting-DEBUG) 的值为 `True` 或者运行测试程序的时候，host validation(主机认证)会被停用；任何主机都会被接受。因此通常只在生产环境中有必要这么设置。

这个validation(验证)只应用于 [`get_host()`](request-response.html#django.http.HttpRequest.get_host "django.http.HttpRequest.get_host"); 如果你的代码是直接从`request.`META访问`Host` 头信息，那么你就绕过了安全保护措施。

### ALLOWED_INCLUDE_ROOTS

默认值：`()`（空元组）

自1.8版起已弃用：不赞成使用这个设置和[`ssi`](templates/builtins.html#std:templatetag-ssi) 模板标签，而且还会在Django2.0中被删除。

Changed in Django 1.8:

你可以在`DjangoTemplates` 后台的[`OPTIONS`](#std:setting-TEMPLATES-OPTIONS)中设置这个`'allowed_include_roots'` 选项来代替。

这tuple(元组)中的字符串允许在模板中使用的时候加个`{% ssi %}`模板标签前缀。这是一个安全措施，所以编写模板的人不能访问那些不应该被访问的文件。

例如, 如果 [`ALLOWED_INCLUDE_ROOTS`](#std:setting-ALLOWED_INCLUDE_ROOTS)的值为`('/home/html', '/var/www')`, 那么模板中这样写：`{% ssi /home/html/foo.txt %}`是没问题的 ，而像这样：`{% ssi /etc/passwd %}` 就不行了。（因为后者访问了/home/html目录以外的文件，这就是上面说的安全措施。译者注)

### APPEND_SLASH

默认值：`True`

当设定为`True` 时，如果请求的URL 没有匹配URLconf 里面的任何URL 并且没有以/（斜杠）结束，将重定向到以/ 结尾的URL。需要注意的是任何重定向都有可能导致post数据的丢失。

[`APPEND_SLASH`](#std:setting-APPEND_SLASH) 设置只有在安装了[`CommonMiddleware`](middleware.html#django.middleware.common.CommonMiddleware "django.middleware.common.CommonMiddleware") 时才用到（参见[_中间件_](../topics/http/middleware.html)）。另见[`PREPEND_WWW`](#std:setting-PREPEND_WWW)。

### 高速缓存

默认：

```
{
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    }
}

```

一个字典包含所有缓存要使用的设置它是一个嵌套字典，其内容将高速缓存别名映射到包含单个高速缓存的选项的字典中。

[`CACHES`](#std:setting-CACHES)设置必须配置`‘default’`缓存；还可以指定任何数量的附加高速缓存。如果您正在使用本地内存高速缓存之外的其他高速缓存后端，或者需要定义多个高速缓存，这就需要添加其他高速缓存项。以下高速缓存选项可用。

#### 后退

默认值：`''`（空字符串）

要使用的缓存后端。内置高速缓存后端是：

*   `'django.core.cache.backends.db.DatabaseCache'`
*   `'django.core.cache.backends.dummy.DummyCache'`
*   `'django.core.cache.backends.filebased.FileBasedCache'`
*   `'django.core.cache.backends.locmem.LocMemCache'`
*   `'django.core.cache.backends.memcached.MemcachedCache'`
*   `'django.core.cache.backends.memcached.PyLibMCCache'`

通过将[`BACKEND`](#std:setting-CACHES-BACKEND)设置为缓存后端类的完全限定路径（即`mypackage.backends.whatever.WhateverCache`），您可以使用未随Django提供的缓存后端。 ）。

#### KEY_FUNCTION

包含函数（或任何可调用）的虚线路径的字符串，定义如何将前缀，版本和键组成最终缓存键。默认实现等效于以下函数：

```
def make_key(key, key_prefix, version):
    return ':'.join([key_prefix, str(version), key])

```

你可以使用任何你想要的键功能，只要它有相同的参数签名。

有关详细信息，请参阅[_cache documentation_](../topics/cache.html#cache-key-transformation)。

#### KEY_PREFIX

默认值：`''`（空字符串）

一个字符串，将被自动包括（默认情况下预置）到Django服务器使用的所有缓存键。

有关详细信息，请参阅[_cache documentation_](../topics/cache.html#cache-key-prefixing)。

#### LOCATION

默认值：`''`（空字符串）

要使用的缓存的位置。这可能是文件系统缓存的目录，内存缓存服务器的主机和端口，或者只是本地内存缓存的标识名称。例如：

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
    }
}

```

#### OPTIONS

默认值：无

额外的参数传递到缓存后端。可用参数因缓存后端而异。

有关可用参数的一些信息，请参见[_Cache Backends_](../topics/cache.html)文档。有关详细信息，请参阅后端模块自己的文档。

#### TIMEOUT

默认值：300

高速缓存的有效时间。

New in Django 1.7.

如果此设置的值为`空`，则缓存将不会过期。

#### VERSION

默认值：`1`

Django服务器生成的缓存键的默认版本号。

有关详细信息，请参阅[_缓存文档_](../topics/cache.html#cache-versioning)。

### CACHE_MIDDLEWARE_ALIAS

默认值：`default`

用于[_cache middleware_](../topics/cache.html#the-per-site-cache)的高速缓存连接。

### CACHE_MIDDLEWARE_KEY_PREFIX

默认值：`''`（空字符串）

将由[_cache middleware_](../topics/cache.html#the-per-site-cache)生成的高速缓存键前缀的字符串。此前缀与[`KEY_PREFIX`](#std:setting-CACHES-KEY_PREFIX)设置组合；它不替代它。

请参阅[_Django’s cache framework_](../topics/cache.html)。

### CACHE_MIDDLEWARE_SECONDS

默认值：`600`

缓存[_cache middleware_](../topics/cache.html#the-per-site-cache)的页面的默认秒数。

请参阅[_Django’s cache framework_](../topics/cache.html)。

### CSRF_COOKIE_AGE

New in Django 1.7.

Default: `31449600` (默认时间为一年.单位为秒)

CSRF cookies有效期 ,单位为秒

设置长期过期时间的原因是为了避免在用户关闭浏览器或将页面加入书签，然后从浏览器缓存加载该页面的情况下出现问题。没有持久性cookie，在这种情况下表单提交将失败。

某些浏览器（特别是Internet Explorer）可能不允许使用持久性cookie，或者可能使cookie jar的索引在磁盘上损坏，从而导致CSRF保护检查失败（有时间歇性）。将此设置更改为`None`可使用基于会话的CSRF Coo​​kie，这些cookie将Cookie保存在内存中，而不是在持久存储上。

### CSRF_COOKIE_DOMAIN

默认值：`None`

设置CSRF cookie时要使用的域。This can be useful for easily allowing cross-subdomain requests to be excluded from the normal cross site request forgery protection.应将其设置为一个字符串，例如`".example.com"`，以允许来自一个子域的表单的POST请求被另一个子域提供的视图接受。

Please note that the presence of this setting does not imply that Django’s CSRF protection is safe from cross-subdomain attacks by default - please see the [_CSRF limitations_](csrf.html#csrf-limitations) section.

### CSRF_COOKIE_HTTPONLY

默认值：`False`

是否在CSRF Coo​​kie上使用`HttpOnly`标志。如果设置为`True`，客户端JavaScript将无法访问CSRF Coo​​kie。

这可以帮助防止恶意JavaScript绕过CSRF保护。如果启用此选项并需要使用Ajax请求发送CSRF令牌的值，那么JavaScript将需要从页面上的隐藏CSRF令牌表单输入中提取值，而不是从Cookie中提取值。

See [`SESSION_COOKIE_HTTPONLY`](#std:setting-SESSION_COOKIE_HTTPONLY) for details on `HttpOnly`.

### CSRF_COOKIE_NAME

默认值：`'csrftoken'`

要用于CSRF身份验证令牌的cookie的名称。这可以是任何你想要的。请参阅[_Cross Site Request Forgery protection_](csrf.html)。

### CSRF_COOKIE_PATH

默认值：`'/'`

在CSRF cookie上设置的路径。这应该匹配您的Django安装的URL路径，或者是该路径的父级。

如果您有多个运行在相同主机名下的Django实例，这将非常有用。他们可以使用不同的cookie路径，每个实例只会看到自己的CSRF cookie。

### CSRF_COOKIE_SECURE

默认值：`False`

是否为CSRF Coo​​kie使用安全Cookie。如果此设置为`True`，则Cookie将被标记为“安全”，这意味着浏览器可以确保该Cookie仅在HTTPS连接下发送。

### CSRF_FAILURE_VIEW

默认值：`'django.views.csrf.csrf_failure'`

当传入请求被CSRF保护拒绝时要使用的视图函数的虚线路径。该函数应该有这个签名：

```
def csrf_failure(request, reason="")

```

其中`reason`是指示请求被拒绝的原因的短消息（针对开发人员或记录，而不是最终用户）。请参阅[_Cross Site Request Forgery protection_](csrf.html)。

### DATABASES

默认：`{}` （空字典）

一个字典，包含Django 将使用的所有数据库的设置。它是一个嵌套的字典，其内容为数据库别名到包含数据库选项的字典的映射。

[`DATABASES`](#std:setting-DATABASES) 设置必须配置一个`default` 数据库；可以同时指定任何数目的额外数据库。

最简单的配置文件可能是使用SQLite 建立一个数据库。这可以使用以下配置：

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': 'mydatabase',
    }
}

```

当连接其他数据库后端，比如MySQL、Oracle 或PostgreSQL，必须提供更多的连接参数。关于如何指定其他的数据库类型，参见后面的[`ENGINE`](#std:setting-DATABASE-ENGINE) 设置。下面的例子用于PostgreSQL：

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'mydatabase',
        'USER': 'mydatabaseuser',
        'PASSWORD': 'mypassword',
        'HOST': '127.0.0.1',
        'PORT': '5432',
    }
}

```

下面是更复杂的配置可能需要的选项：

#### ATOMIC_REQUESTS

默认：`False`

当需要将每个HTTP 请求封装在一个数据库事务中时，设置它为`True`。参见[_将事务与HTTP 请求绑定_](../topics/db/transactions.html#tying-transactions-to-http-requests)。

#### AUTOCOMMIT

默认：`True`

如果你需要[_禁用Django 的事务管理_](../topics/db/transactions.html#deactivate-transaction-management)并自己实现，设置它为`False`。

#### ENGINE

默认：`''`（空字符串）

使用的数据库后端。内建的数据库后端有：

*   `'django.db.backends.postgresql_psycopg2'`
*   `'django.db.backends.mysql'`
*   `'django.db.backends.sqlite3'`
*   `'django.db.backends.oracle'`

你可以不使用Django 自带的数据库后端，通过设置`ENGINE` 为一个合法的路径即可（例如`mypackage.backends.whatever`）。

#### HOST

默认：`''`（空字符串）

连接数据库时使用哪个主机。空字符串意味着采用localhost 作为主机。 SQLite 不需要这个选项。

如果其值以斜杠（`'/'`）开始并且你使用的是MySQL，MySQL 将通过Unix socket 连接。例如：

```
"HOST": '/var/run/mysql'

```

如果你使用的是MySQL 并且该值_不是_以斜杠开头，那么将假设该值为主机。

如果你使用的是PostgreSQL，默认情况下（空[`HOST`](#std:setting-HOST)），数据库的连接通过UNIX domain sockets（`pg_hba.conf` 中的‘local’行）。如果你的UNIX domain socket 不在标准的路径，则使用`postgresql.conf` 中的`unix_socket_directory` 值。如果你想通过TCP sockets 连接，请设置[`HOST`](#std:setting-HOST) 为‘localhost’ 或 ‘127.0.0.1’（`pg_hba.conf` 中的‘host’行）。在Windows上，你应该始终定义[`HOST`](#std:setting-HOST)，因为其不可以使用UNIX domain sockets。

#### NAME

默认：`''`（空字符串）

使用的数据库名称。对于SQLite，它是数据库文件的完整路径。指定路径时，请始终使用前向的斜杠，即使在Windows 上（例如`C:/homes/user/mysite/sqlite3.db`）。

#### CONN_MAX_AGE

默认：`0`

数据库连接的存活时间，以秒为单位。`0` 表示在每个请求结束时关闭数据库连接 —— 这是Django 的历史遗留行为，`None` 表示无限的持久连接。

#### OPTIONS

默认：`{}（空字典）`

连接数据库时使用的额外参数。可用的参数与你的数据库后端有关。

在 [_数据库后端_](databases.html)的文档中可以找到可用的参数的一些信息。更多信息，参考后端模块自身的文档。

#### PASSWORD

默认：`''`（空字符串）

连接数据库时使用的密码。SQLite 不需要这个选项。

#### PORT (端口)

默认：`''`（空字符串）

连接数据库时使用的端口。空字符串表示默认的端口。SQLite 不需要这个选项。

#### USER

默认：`''`（空字符串）

连接数据库时使用的用户名。SQLite 不需要这个选项。

#### TEST

Changed in Django 1.7:

所有[`TEST`](#std:setting-DATABASE-TEST)子条目用作数据库设置字典中的独立条目，并具有`TEST_`前缀。为了向后兼容旧版本的Django，您可以定义两个版本的设置，只要它们匹配。Further, `TEST_CREATE`, `TEST_USER_CREATE` and `TEST_PASSWD` were changed to `CREATE_DB`, `CREATE_USER` and `PASSWORD` respectively.

Default: `{}`

用于测试数据库的一个设置字典；有关创建和使用测试数据库的更多详细信息，请参见[_测试数据库_](../topics/testing/overview.html#the-test-database)。以下条目可用：

##### CHARSET

Default: `None`

字符集设置用于指定数据库编码格式。该设置的值会直接传给数据库，所以它的格式是由指定的数据库来决定的。

该字段支持 [PostgreSQL](http://www.postgresql.org/docs/current/static/multibyte.html) (`postgresql_psycopg2`) 和[MySQL](http://dev.mysql.com/doc/refman/5.6/en/charset-database.html) (`mysql`) 数据库。

##### COLLATION

Default: `None`

创建测试数据库时要使用的归类顺序。此值直接传递到后端，因此其格式是后端特定的。

仅支持`mysql`后端（有关详细信息，请参阅[MySQL手册](http://dev.mysql.com/doc/refman/5.6/en/charset-database.html)）。

##### DEPENDENCIES

默认值：`['default']`，对于除了`default`之外的所有数据库，没有依赖关系。

数据库的创建顺序依赖性。有关详细信息，请参阅有关[_controlling the creation order of test databases_](../topics/testing/advanced.html#topics-testing-creation-dependencies)的文档。

##### MIRROR

默认值：`None`

此数据库在测试期间应映射的数据库的别名。

此设置存在以允许测试多个数据库的主/副本（由某些数据库称为主/从属）配置。有关详细信息，请参阅有关[_testing primary/replica configurations_](../topics/testing/advanced.html#topics-testing-primaryreplica)的文档。

##### NAME

默认值：`None`

运行测试套件时要使用的数据库的名称。

如果默认值（`None`）与SQLite数据库引擎一起使用，则测试将使用内存驻留数据库。对于所有其他数据库引擎，测试数据库将使用名称`'test _' + DATABASE_NAME`。

请参见[_The test database_](../topics/testing/overview.html#the-test-database)。

##### SERIALIZE

New in Django 1.7.1.

布尔值以控制在运行测试之前缺省测试运行器是否将数据库序列化为内存中的JSON字符串（用于在没有事务的情况下在测试之间恢复数据库状态）。如果您没有任何具有[_serialized_rollback=True_](../topics/testing/overview.html#test-case-serialized-rollback)的测试类，您可以将其设置为`False`以加快创建时间。

##### CREATE_DB

默认值：`True`

这是Oracle特定的设置。

如果设置为`False`，则测试表空间将不会在测试开始时自动创建，并在结束时删除。

##### CREATE_USER

默认值：`True`

这是一个 Oracle-specific 设置.

如果设置为`False`，测试用户将不会在测试开始时自动创建，并在最后删除。

##### USER

默认值：`None`

这是Oracle特定的设置。

连接到将在运行测试时使用的Oracle数据库时使用的用户名。如果没有提供，Django将使用`'test _' + USER`。

##### PASSWORD

默认值：`None`

这是Oracle特定的设置。

连接到运行测试时将使用的Oracle数据库时使用的密码。如果没有提供，Django将使用硬编码的默认值。

##### TBLSPACE

默认值：`None`

这是Oracle特定的设置。

运行测试时将使用的表空间的名称。如果没有提供，Django将使用`'test _' + USER`。

Changed in Django 1.8:

以前，如果未提供，Django使用了`'test _' + NAME`

##### TBLSPACE_TMP

默认值：`None`

这是Oracle特定的设置。

运行测试时将使用的临时表空间的名称。如果未提供，Django将使用`'test _' + USER + `。

Changed in Django 1.8:

之前Django使用了`'test _' + NAME + '_ temp' `（如果未提供）。

##### DATAFILE

New in Django 1.8.

默认值：`None`

这是Oracle特定的设置。

要用于TBLSPACE的数据文件的名称。如果没有提供，Django将使用`TBLSPACE + '。dbf'`。

##### DATAFILE_TMP

New in Django 1.8.

默认值：`None`

这是Oracle特定的设置。

要用于TBLSPACE_TMP的数据文件的名称。如果未提供，Django将使用`TBLSPACE_TMP + '。dbf'`。

##### DATAFILE_MAXSIZE

New in Django 1.8.

默认值：`'500M'`

Changed in Django 1.8:

上一个值为200M，不是用户可自定义的。

这是Oracle特定的设置。

允许DATAFILE增长到的最大大小。

##### DATAFILE_TMP_MAXSIZE

New in Django 1.8.

默认值：`'500M'`

Changed in Django 1.8:

上一个值为200M，不是用户可自定义的。

这是Oracle特定的设置。

允许DATAFILE_TMP增长到的最大大小。

#### TEST_CHARSET

自1.7版起已弃用：使用[`TEST`](#std:setting-DATABASE-TEST)字典中的[`CHARSET`](#std:setting-TEST_CHARSET)条目。

#### TEST_COLLATION

自1.7版起已弃用：使用[`TEST`](#std:setting-DATABASE-TEST)字典中的[`COLLATION`](#std:setting-TEST_COLLATION)条目。

#### TEST_DEPENDENCIES

自1.7版起已弃用：使用[`TEST`](#std:setting-DATABASE-TEST)字典中的[`DEPENDENCIES`](#std:setting-TEST_DEPENDENCIES)条目。

#### TEST_MIRROR

自1.7版起已弃用：Use the [`MIRROR`](#std:setting-TEST_MIRROR) entry in the [`TEST`](#std:setting-DATABASE-TEST) dictionary.

#### TEST_NAME

自1.7版起已弃用：使用[`TEST`](#std:setting-DATABASE-TEST)字典中的[`NAME`](#std:setting-TEST_NAME)条目。

#### TEST_CREATE

自1.7版起已弃用：使用[`TEST`](#std:setting-DATABASE-TEST)字典中的[`CREATE_DB`](#std:setting-TEST_CREATE)条目。

#### TEST_USER

自1.7版起已弃用：使用[`TEST`](#std:setting-DATABASE-TEST)字典中的[`USER`](#std:setting-TEST_USER)条目。

#### TEST_USER_CREATE

自1.7版起已弃用：Use the [`CREATE_USER`](#std:setting-TEST_USER_CREATE) entry in the [`TEST`](#std:setting-DATABASE-TEST) dictionary.

#### TEST_PASSWD

自1.7版起已弃用：使用[`TEST`](#std:setting-DATABASE-TEST)字典中的[`PASSWORD`](#std:setting-TEST_PASSWD)条目。

#### TEST_TBLSPACE

自1.7版起已弃用：Use the [`TBLSPACE`](#std:setting-TEST_TBLSPACE) entry in the [`TEST`](#std:setting-DATABASE-TEST) dictionary.

#### TEST_TBLSPACE_TMP

自1.7版起已弃用：Use the [`TBLSPACE_TMP`](#std:setting-TEST_TBLSPACE_TMP) entry in the [`TEST`](#std:setting-DATABASE-TEST) dictionary.

### DATABASE_ROUTERS

默认值：`[]`（空列表）

将用于确定在执行数据库查询时要使用哪个数据库的路由器列表。

请参阅有关[_automatic database routing in multi database configurations_](../topics/db/multi-db.html#topics-db-multi-db-routing)的文档。

### 日期格式

Default: `'N j, Y'` (e.g. `Feb. 4, 2003`)

用于在系统任何部分中显示日期字段的默认格式。请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则区域设置格式具有更高的优先级，将会应用。请参阅[`allowed date format strings`](templates/builtins.html#std:templatefilter-date)。

另请参阅[`DATETIME_FORMAT`](#std:setting-DATETIME_FORMAT)，[`TIME_FORMAT`](#std:setting-TIME_FORMAT)和[`SHORT_DATE_FORMAT`](#std:setting-SHORT_DATE_FORMAT)。

### DATE_INPUT_FORMATS

默认：

```
(
    '%Y-%m-%d', '%m/%d/%Y', '%m/%d/%y', # '2006-10-25', '10/25/2006', '10/25/06'
    '%b %d %Y', '%b %d, %Y',            # 'Oct 25 2006', 'Oct 25, 2006'
    '%d %b %Y', '%d %b, %Y',            # '25 Oct 2006', '25 Oct, 2006'
    '%B %d %Y', '%B %d, %Y',            # 'October 25 2006', 'October 25, 2006'
    '%d %B %Y', '%d %B, %Y',            # '25 October 2006', '25 October, 2006'
)

```

在日期字段上输入数据时将接受的格式的元组。格式将按顺序尝试，使用第一个有效的格式。请注意，这些格式字符串使用Python的[datetime](https://docs.python.org/library/datetime.html#strftime-strptime-behavior)模块语法，而不是来自`date` Django模板标记的格式字符串。

当[`USE_L10N`](#std:setting-USE_L10N)为`True`时，区域设置格式具有更高的优先级，将会应用。

另请参阅[`DATETIME_INPUT_FORMATS`](#std:setting-DATETIME_INPUT_FORMATS)和[`TIME_INPUT_FORMATS`](#std:setting-TIME_INPUT_FORMATS)。

### DATETIME_FORMAT

Default: `'N j, Y, P'` (e.g. `Feb. 4, 2003, 4 p.m.`)

用于在系统任何部分中显示datetime字段的默认格式。请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则区域设置格式具有更高的优先级，将会应用。请参阅[`allowed date format strings`](templates/builtins.html#std:templatefilter-date)。

另请参阅[`DATE_FORMAT`](#std:setting-DATE_FORMAT)，[`TIME_FORMAT`](#std:setting-TIME_FORMAT)和[`SHORT_DATETIME_FORMAT`](#std:setting-SHORT_DATETIME_FORMAT)。

### DATETIME_INPUT_FORMATS

默认：

```
(
    '%Y-%m-%d %H:%M:%S',     # '2006-10-25 14:30:59'
    '%Y-%m-%d %H:%M:%S.%f',  # '2006-10-25 14:30:59.000200'
    '%Y-%m-%d %H:%M',        # '2006-10-25 14:30'
    '%Y-%m-%d',              # '2006-10-25'
    '%m/%d/%Y %H:%M:%S',     # '10/25/2006 14:30:59'
    '%m/%d/%Y %H:%M:%S.%f',  # '10/25/2006 14:30:59.000200'
    '%m/%d/%Y %H:%M',        # '10/25/2006 14:30'
    '%m/%d/%Y',              # '10/25/2006'
    '%m/%d/%y %H:%M:%S',     # '10/25/06 14:30:59'
    '%m/%d/%y %H:%M:%S.%f',  # '10/25/06 14:30:59.000200'
    '%m/%d/%y %H:%M',        # '10/25/06 14:30'
    '%m/%d/%y',              # '10/25/06'
)

```

在日期时间字段上输入数据时将接受的格式的元组。格式将按顺序尝试，使用第一个有效的格式。请注意，这些格式字符串使用Python的[datetime](https://docs.python.org/library/datetime.html#strftime-strptime-behavior)模块语法，而不是来自`date` Django模板标记的格式字符串。

当[`USE_L10N`](#std:setting-USE_L10N)为`True`时，区域设置格式具有更高的优先级，将会应用。

另请参阅[`DATE_INPUT_FORMATS`](#std:setting-DATE_INPUT_FORMATS)和[`TIME_INPUT_FORMATS`](#std:setting-TIME_INPUT_FORMATS)。

### DEBUG

默认值：`False`

打开/关闭调试模式的布尔值。

部署网站的时候不要把[`DEBUG`](#std:setting-DEBUG) 打开.

你明白了吗？部署网站的时候一定不要把 [`DEBUG`](#std:setting-DEBUG) 打开.

调试模式的一个重要特性是显示错误页面的细节。当[`DEBUG`](#std:setting-DEBUG) 为 `True`的时候,若你的应用产生了一个异常，Django 会显示追溯细节,包括许多环境变量的元数据, 比如所有当前定义的Django设置(在`settings.py`中的).

作为安全措施, Django 将 _不会_ 包括敏感的 (或者可能会被攻击的)设置, 例如 [`SECRET_KEY`](#std:setting-SECRET_KEY). 特别是名字中包含下面这些单词的设置:

*   `'API'`
*   `'KEY'`
*   `'PASS'`
*   `'SECRET'`
*   `'SIGNATURE'`
*   `'TOKEN'`

注意，这里使用的是 _部分_ 匹配. `'PASS'`将匹配 PASSWORD, 另外 `'TOKEN'` 也将匹配 TOKENIZED 等等.

不过，总有一些调试的输出你不希望展现给公众的。文件路径， 配置信息和其他，将会提供信息给攻击者来攻击你的服务器。

另外，很重要的是要记住当你运行时 [`DEBUG`](#std:setting-DEBUG) 模式打开的话, Django 记住所有执行的 SQL 查询语句。 这在进行 DEBUG 调试时非常有用, 但这会消耗运行服务器的大量内存资源.

最后，如果[`DEBUG`](#std:setting-DEBUG) 为`False`，你还需要正确设置[`ALLOWED_HOSTS`](#std:setting-ALLOWED_HOSTS)。设置错误将导致对所有的请求返回“Bad Request (400)”。

### DEBUG_PROPAGATE_EXCEPTIONS

默认值：`False`

如果设置为True，Django的视图函数的正常异常处理将被抑制，异常将向上传播。这对于某些测试设置可能很有用，不应在实际站点上使用。

### DECIMAL_SEPARATOR

默认值：`'.'`（点）

格式化十进制数时使用的默认十进制分隔符。

请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则区域设置格式具有更高的优先级，将会应用。

另请参阅[`NUMBER_GROUPING`](#std:setting-NUMBER_GROUPING)，[`THOUSAND_SEPARATOR`](#std:setting-THOUSAND_SEPARATOR)和[`USE_THOUSAND_SEPARATOR`](#std:setting-USE_THOUSAND_SEPARATOR)。

### DEFAULT_CHARSET

默认值：`'utf-8'`

如果未手动指定MIME类型，则用于所有`HttpResponse`对象的默认字符集。与[`DEFAULT_CONTENT_TYPE`](#std:setting-DEFAULT_CONTENT_TYPE)配合使用以构造`Content-Type`头。

### DEFAULT_CONTENT_TYPE

默认值：`'text/html'`

如果未手动指定MIME类型，则用于所有`HttpResponse`对象的默认内容类型。与[`DEFAULT_CHARSET`](#std:setting-DEFAULT_CHARSET)配合使用以构造`Content-Type`头。

### DEFAULT_EXCEPTION_REPORTER_FILTER

默认值：[`django.views.debug.SafeExceptionReporterFilter`](../howto/error-reporting.html#django.views.debug.SafeExceptionReporterFilter "django.views.debug.SafeExceptionReporterFilter")

如果尚未将任何分配给[`HttpRequest`](request-response.html#django.http.HttpRequest "django.http.HttpRequest")实例，则使用缺省异常报告器过滤器类。请参阅[_Filtering error reports_](../howto/error-reporting.html#filtering-error-reports)。

### DEFAULT_FILE_STORAGE

默认：[`django.core.files.storage.FileSystemStorage`](files/storage.html#django.core.files.storage.FileSystemStorage "django.core.files.storage.FileSystemStorage")

默认的Storage 类，用于没有指定文件系统的任何和文件相关的操作。参见[_管理文件_](../topics/files.html)。

### DEFAULT_FROM_EMAIL

默认值：`'webmaster@localhost'`

用于来自站点管理员的各种自动通信的默认电子邮件地址。这不包括发送到[`ADMINS`](#std:setting-ADMINS)和[`MANAGERS`](#std:setting-MANAGERS)的错误消息；有关详细信息，请参阅[`SERVER_EMAIL`](#std:setting-SERVER_EMAIL)。

### DEFAULT_INDEX_TABLESPACE

默认值：`''`（空字符串）

用于不指定一个字段（如果后端支持它）的字段上的索引的默认表空间（请参阅[_Tablespaces_](../topics/db/tablespaces.html)）。

### DEFAULT_TABLESPACE

默认值：`''`（空字符串）

用于不指定后者的模型（如果后端支持它）的默认表空间（请参阅[_Tablespaces_](../topics/db/tablespaces.html)）。

### DISALLOWED_USER_AGENTS

默认值：`()`（空元组）

表示不允许访问系统范围内任何页面的User-Agent字符串的编译正则表达式对象列表。用于坏的漫游器/抓取工具。只有在安装`CommonMiddleware`时才会使用（请参阅[_Middleware_](../topics/http/middleware.html)）。

### EMAIL_BACKEND

默认：`'django.core.mail.backends.smtp.EmailBackend'`

用于发送邮件的后端。可选的后端参见[_发送邮件_](../topics/email.html)。

### EMAIL_FILE_PATH

默认：未指定

`file` 类型的邮件后端保存输出文件时使用的目录。

### EMAIL_HOST

默认：`'localhost'`

发送邮件使用的主机。

另见[`EMAIL_PORT`](#std:setting-EMAIL_PORT)。

### EMAIL_HOST_PASSWORD

默认：`''`（空字符串）

[`EMAIL_HOST`](#std:setting-EMAIL_HOST) 定义的SMTP 服务器使用的密码。这个设置与[`EMAIL_HOST_USER`](#std:setting-EMAIL_HOST_USER) 一起用于SMTP 服务器的认证。如果两个中有一个为空，Django 则不会尝试认证。

另见[`EMAIL_HOST_USER`](#std:setting-EMAIL_HOST_USER)。

### EMAIL_HOST_USER

默认：`''`（空字符串）

[`EMAIL_HOST`](#std:setting-EMAIL_HOST) 定义的SMTP 服务器使用的用户名。如果为空，Django 不会尝试认证。

另见[`EMAIL_HOST_PASSWORD`](#std:setting-EMAIL_HOST_PASSWORD)。

### EMAIL_PORT

默认：`25`

[`EMAIL_HOST`](#std:setting-EMAIL_HOST) 定义的SMTP 服务器使用的端口。

### EMAIL_SUBJECT_PREFIX

默认值：`'[Django] '`

使用`django.core.mail.mail_admins`或`django.core.mail.mail_managers`发送的电子邮件的主题行前缀。你可能想要包含结尾空格。

### EMAIL_USE_TLS

默认值：`False`

是否使用TLS(安全)当与SMTP服务器的连接。这是用于显式TLS连接,通常在端口587上。如果你正在经历挂连接,看到隐EMAIL_USE_SSL TLS设置。这用于显式TLS连接，通常在端口587上。如果您遇到挂起的连接，请参阅隐式TLS设置[`EMAIL_USE_SSL`](#std:setting-EMAIL_USE_SSL)。

### EMAIL_USE_SSL

New in Django 1.7.

默认值：`False`

在与SMTP服务器通信时是否使用隐式TLS（安全）连接。在大多数电子邮件文档中，此类型的TLS连接称为SSL。它通常在端口465上使用。如果您遇到问题，请参阅显式TLS设置[`EMAIL_USE_TLS`](#std:setting-EMAIL_USE_TLS)。

请注意，[`EMAIL_USE_TLS`](#std:setting-EMAIL_USE_TLS) / [`EMAIL_USE_SSL`](#std:setting-EMAIL_USE_SSL)是互斥的，因此只能将其中一个设置设置为`True`。

### EMAIL_SSL_CERTFILE

New in Django 1.8.

默认值：`None`

如果[`EMAIL_USE_SSL`](#std:setting-EMAIL_USE_SSL)或[`EMAIL_USE_TLS`](#std:setting-EMAIL_USE_TLS)为`True`，则可以选择指定要用于SSL连接的PEM格式的证书链文件的路径。

### EMAIL_SSL_KEYFILE

New in Django 1.8.

默认值：`None`

如果[`EMAIL_USE_SSL`](#std:setting-EMAIL_USE_SSL)或[`EMAIL_USE_TLS`](#std:setting-EMAIL_USE_TLS)为`True`，您可以选择指定要用于SSL连接的PEM格式的私钥文件的路径。

请注意，设置[`EMAIL_SSL_CERTFILE`](#std:setting-EMAIL_SSL_CERTFILE)和[`EMAIL_SSL_KEYFILE`](#std:setting-EMAIL_SSL_KEYFILE)不会导致任何证书检查。它们将传递到底层SSL连接。有关如何处理证书链文件和私钥文件的详细信息，请参阅Python的[`ssl.wrap_socket()`](https://docs.python.org/3/library/ssl.html#ssl.wrap_socket "(in Python v3.4)")函数的文档。

### EMAIL_TIMEOUT

New in Django 1.8.

默认值：`None`

指定阻止操作（如连接尝试）的超时（以秒为单位）。

### FILE_CHARSET

默认值：`'utf-8'`

用于解码从磁盘读取的任何文件的字符编码。这包括模板文件和初始SQL数据文件。

### FILE_UPLOAD_HANDLERS

默认：

```
("django.core.files.uploadhandler.MemoryFileUploadHandler",
 "django.core.files.uploadhandler.TemporaryFileUploadHandler")

```

用于上传的处理程序的元组。更改此设置允许完全自定义 - 甚至替换Django的上传过程。

有关详细信息，请参阅[_Managing files_](../topics/files.html)。

### FILE_UPLOAD_MAX_MEMORY_SIZE

默认值：`2621440`（即2.5 MB）。

在上传到文件系统之前上传的最大大小（以字节为单位）。有关详细信息，请参阅[_Managing files_](../topics/files.html)。

### FILE_UPLOAD_DIRECTORY_PERMISSIONS

New in Django 1.7.

默认值：`None`

应用于上传文件过程中创建的目录的数字模式。

此设置还会在使用[`collectstatic`](contrib/staticfiles.html#django-admin-collectstatic)管理命令时确定收集的静态目录的默认权限。有关覆盖它的详细信息，请参阅[`collectstatic`](contrib/staticfiles.html#django-admin-collectstatic)。

此值反映了[`FILE_UPLOAD_PERMISSIONS`](#std:setting-FILE_UPLOAD_PERMISSIONS)设置的功能和注意事项。

### FILE_UPLOAD_PERMISSIONS

默认值：`None`

将新上传的文件设置为的数字模式（即`0o644`）。有关这些模式意味着什么的更多信息，请参阅[`os.chmod()`](https://docs.python.org/3/library/os.html#os.chmod "(in Python v3.4)")的文档。

如果未给出或`None`，您将获得操作系统相关的行为。在大多数平台上，临时文件的模式为`0o600`，从内存保存的文件将使用系统的标准umask保存。

出于安全考虑，这些权限不会应用于存储在[`FILE_UPLOAD_TEMP_DIR`](#std:setting-FILE_UPLOAD_TEMP_DIR)中的临时文件。

此设置还会在使用[`collectstatic`](contrib/staticfiles.html#django-admin-collectstatic)管理命令时确定收集的静态文件的默认权限。有关覆盖它的详细信息，请参阅[`collectstatic`](contrib/staticfiles.html#django-admin-collectstatic)。

警告

**始终以模式前缀0.**

如果您不熟悉文件模式，请注意领先的`0`非常重要：它表示一个八进制数，这是模式必须指定的方式。如果您尝试使用`644`，您将得到完全不正确的行为。

### FILE_UPLOAD_TEMP_DIR

默认值：`None`

上传文件时临时存储数据的目录（通常大于[`FILE_UPLOAD_MAX_MEMORY_SIZE`](#std:setting-FILE_UPLOAD_MAX_MEMORY_SIZE)的文件）。如果`None`，Django将使用操作系统的标准临时目录。例如，在* nix风格的操作系统上，这将默认为`/tmp`。

有关详细信息，请参阅[_Managing files_](../topics/files.html)。

### FIRST_DAY_OF_WEEK

默认值：`0`（星期日）

表示一周中第一天的数字。这在显示日历时特别有用。此值仅在不使用格式国际化时或在找不到当前语言环境的格式时使用。

该值必须为0到6之间的整数，其中0表示星期日，1表示星期一，依此类推。

### FIXTURE_DIRS

默认值：`()`（空元组）

搜索夹具文件的目录列表，以及搜索顺序中每个应用程序的`fixtures`目录。

注意，这些路径应该使用Unix风格的正斜杠，即使在Windows上。

请参阅[_Providing initial data with fixtures_](../howto/initial-data.html#id1)和[_Fixture loading_](../topics/testing/tools.html#topics-testing-fixtures)。

### FORCE_SCRIPT_NAME

默认值: `None`

如果不是`None`，这将用作任何HTTP请求中`SCRIPT_NAME`环境变量的值。此设置可用于覆盖服务器提供的`SCRIPT_NAME`值，该值可能是首选值的重写版本，或者根本不提供。

### FORMAT_MODULE_PATH

默认值：`None`

Python包的完整Python路径，其中包含项目语言环境的格式定义。如果不是`None`，Django将在名为当前语言环境的目录下检查`formats.py`文件，并使用此文件中定义的格式。

例如，如果[`FORMAT_MODULE_PATH`](#std:setting-FORMAT_MODULE_PATH)设置为`mysite.formats`，并且当前语言为`en`（英语），Django将需要一个目录树，

```
mysite/
    formats/
        __init__.py
        en/
            __init__.py
            formats.py

```

Changed in Django 1.8:

您还可以将此设置设置为Python路径列表，例如：

```
FORMAT_MODULE_PATH = [
    'mysite.formats',
    'some_app.formats',
]

```

当Django搜索某种格式时，它将遍历所有给定的Python路径，直到找到一个实际定义给定格式的模块。这意味着在列表中更远的包中定义的格式将优先于在更远的包中的相同格式。

可用的格式有[`DATE_FORMAT`](#std:setting-DATE_FORMAT)，[`TIME_FORMAT`](#std:setting-TIME_FORMAT)，[`DATETIME_FORMAT`](#std:setting-DATETIME_FORMAT)，[`YEAR_MONTH_FORMAT`](#std:setting-YEAR_MONTH_FORMAT)，[`MONTH_DAY_FORMAT`](#std:setting-MONTH_DAY_FORMAT)，[`SHORT_DATE_FORMAT`](#std:setting-SHORT_DATE_FORMAT)，[`SHORT_DATETIME_FORMAT`](#std:setting-SHORT_DATETIME_FORMAT)，[`FIRST_DAY_OF_WEEK`](#std:setting-FIRST_DAY_OF_WEEK)，[`DECIMAL_SEPARATOR`](#std:setting-DECIMAL_SEPARATOR)，[`THOUSAND_SEPARATOR`](#std:setting-THOUSAND_SEPARATOR)和[`NUMBER_GROUPING`](#std:setting-NUMBER_GROUPING)

### IGNORABLE_404_URLS

默认值：`()`

通过电子邮件报告HTTP 404错误时应忽略的编译正则表达式对象列表（请参阅[_Error reporting_](../howto/error-reporting.html)）。正则表达式与[`request's full paths`](request-response.html#django.http.HttpRequest.get_full_path "django.http.HttpRequest.get_full_path")（包括查询字符串，如果有）匹配。如果您的网站没有提供通常要求的档案（例如`favicon.ico`或`robots.txt`），或是遭到脚本小孩的攻击，请使用此方法。

只有在启用[`BrokenLinkEmailsMiddleware`](middleware.html#django.middleware.common.BrokenLinkEmailsMiddleware "django.middleware.common.BrokenLinkEmailsMiddleware")（请参阅[_Middleware_](../topics/http/middleware.html)）时才会使用此选项。

### INSTALLED_APPS

默认值：`()`（空元组）

一个字符串元组，它标明了所有能在django安装的应用每个字符串应该是一个虚线的Python路径：

*   应用程序配置类或
*   包含应用程序的包。

[_Learn more about application configurations_](applications.html)。

Changed in Django 1.7:

[`INSTALLED_APPS`](#std:setting-INSTALLED_APPS)现在支持应用程序配置。

使用应用程序注册表进行自我检查

您的代码不应直接访问[`INSTALLED_APPS`](#std:setting-INSTALLED_APPS)。请改用[`django.apps.apps`](applications.html#django.apps.apps "django.apps.apps")。

应用程序名称和标签在[`INSTALLED_APPS`](#std:setting-INSTALLED_APPS)中必须是唯一的

应用程序[`names`](applications.html#django.apps.AppConfig.name "django.apps.AppConfig.name") - 应用程序包的点分Python路径必须是唯一的。没有办法包含相同的应用程序两次，没有重复其代码在另一个名称下。

应用程序[`标签`](applications.html#django.apps.AppConfig.label "django.apps.AppConfig.label") - 默认情况下，名称的最后一部分也必须是唯一的。例如，您不能同时包含`django.contrib.auth`和`myproject.auth`。但是，您可以使用定义不同[`标签`](applications.html#django.apps.AppConfig.label "django.apps.AppConfig.label")的自定义配置重新标记应用程序。

无论[`INSTALLED_APPS`](#std:setting-INSTALLED_APPS)是否引用应用程序包上的应用程序配置类，这些规则都适用。

当多个应用程序提供相同资源（模板，静态文件，管理命令，翻译）的不同版本时，[`INSTALLED_APPS`](#std:setting-INSTALLED_APPS)中首先列出的应用程序优先。

### INTERNAL_IPS

默认值：`()`（空元组）

一个IP地址的元组，作为字符串：

*   当[`DEBUG`](#std:setting-DEBUG)为`True`时，请参见调试注释
*   如果安装了`XViewMiddleware`，请在admindocs中接收X标头（请参阅[_The Django admin documentation generator_](contrib/admin/admindocs.html)）

### LANGUAGE_CODE

默认值：`'en-us'`

表示此安装的语言代码的字符串。这应该是标准的[_language ID format_](../topics/i18n/index.html#term-language-code)。例如，美国英语是`"en-us"`。另请参阅[语言标识符列表](http://www.i18nguy.com/unicode/language-identifiers.html)和[_Internationalization and localization_](../topics/i18n/index.html)。

[`USE_I18N`](#std:setting-USE_I18N)必须处于活动状态才能使此设置生效。

它有两个目的：

*   如果区域中间件未使用，则会决定向所有用户提供哪个翻译。
*   如果区域中间件处于活动状态，则它会提供后备语言，以防用户的首选语言无法确定或网站不支持。当用户的首选语言不存在给定文字的翻译时，它还提供后备翻译。

Changed in Django 1.8:

添加了翻译文字的回退。

有关详细信息，请参见[_How Django discovers language preference_](../topics/i18n/translation.html#how-django-discovers-language-preference)。

### LANGUAGE_COOKIE_AGE

New in Django 1.7.

默认值：`None`（在浏览器关闭时过期）

语言Cookie的年龄（以秒为单位）。

### LANGUAGE_COOKIE_DOMAIN

New in Django 1.7.

默认值：`None`

用于语言Cookie的域。将此字符串设置为跨域Cookie的字符串，例如`".example.com"`（请注意前导点！），或者对于标准域Cookie使用`None`。

在生产站点上更新此设置时要小心。如果您更新此设置以在以前使用标准域Cookie的网站上启用跨网域Cookie，则不会更新具有旧网域的现有用户Cookie。这将导致网站用户无法切换语言，只要这些Cookie持续存在。执行切换的唯一安全可靠的选项是永久更改语言cookie名称（通过[`LANGUAGE_COOKIE_NAME`](#std:setting-LANGUAGE_COOKIE_NAME)设置），并添加一个中间件，将旧值从一个新的cookie复制到一个新的，然后删除旧的。

### LANGUAGE_COOKIE_NAME

默认值：`'django_language'`

用于语言Cookie的Cookie的名称。这可以是您想要的（但应该不同于[`SESSION_COOKIE_NAME`](#std:setting-SESSION_COOKIE_NAME)）。请参阅[_Internationalization and localization_](../topics/i18n/index.html)。

### LANGUAGE_COOKIE_PATH

New in Django 1.7.

默认值：`/`

在语言cookie上设置的路径。这应该匹配您的Django安装的URL路径，或者是该路径的父级。

如果您有多个运行在相同主机名下的Django实例，这将非常有用。他们可以使用不同的cookie路径，每个实例只会看到自己的语言cookie。

在生产站点上更新此设置时要小心。如果您将此设置更新为使用比之前使用的更深的路径，则不会更新具有旧路径的现有用户Cookie。这将导致网站用户无法切换语言，只要这些Cookie持续存在。执行切换的唯一安全可靠的选项是永久更改语言Cookie名称（通过[`LANGUAGE_COOKIE_NAME`](#std:setting-LANGUAGE_COOKIE_NAME)设置），并添加中间件，将旧值从旧Cookie复制到新Cookie，然后删除一个。

### LANGUAGES

默认：所有可用语言的元组。这个列表不断增长，包括一个副本在这里将不可避免地变得迅速过时。您可以在`django/conf/global_settings.py`（或查看[在线源](https://github.com/django/django/blob/master/django/conf/global_settings.py)）中查看当前翻译语言列表。

该列表是格式（[_language code_](../topics/i18n/index.html#term-language-code)，`语言 名称`）的二元组的元组 - 例如，`（'ja'， '日语'）`。这指定了哪些语言可用于语言选择。请参阅[_Internationalization and localization_](../topics/i18n/index.html)。

一般来说，默认值应该足够了。如果您要将语言选择限制为Django提供的语言的一部分，请仅设置此设置。

如果您定义自定义[`LANGUAGES`](#std:setting-LANGUAGES)设置，则可以使用[`ugettext_lazy()`](utils.html#django.utils.translation.ugettext_lazy "django.utils.translation.ugettext_lazy")函数将语言名称标记为翻译字符串。

下面是一个示例设置文件：

```
from django.utils.translation import ugettext_lazy as _

LANGUAGES = (
    ('de', _('German')),
    ('en', _('English')),
)

```

### LOCALE_PATHS

默认值：`()`（空元组）

Django查找翻译文件的目录的一个元组。请参阅[_How Django discovers translations_](../topics/i18n/translation.html#how-django-discovers-translations)。

例：

```
LOCALE_PATHS = (
    '/home/www/project/common_files/locale',
    '/var/local/translations/locale',
)

```

Django将在这些路径中查找包含实际翻译文件的`&lt;locale_code&gt;/LC_MESSAGES`目录。

### LOGGING

默认值：日志配置字典。

包含配置信息的数据结构。此数据结构的内容将作为参数传递到[`LOGGING_CONFIG`](#std:setting-LOGGING_CONFIG)中描述的配置方法。

其中，默认日志配置会在[`DEBUG`](#std:setting-DEBUG)为`False`时将HTTP 500服务器错误传递到电子邮件日志处理程序。另请参见[_Configuring logging_](../topics/logging.html#configuring-logging)。

您可以在`django/utils/log.py`（或查看[在线源](https://github.com/django/django/blob/master/django/utils/log.py)）中查看默认日志配置。

### LOGGING_CONFIG

默认值：`'logging.config.dictConfig'`

将用于在Django项目中配置日志记录的可调用项的路径。默认情况下，Python的[dictConfig](https://docs.python.org/library/logging.config.html#configuration-dictionary-schema)配置方法实例的点。

如果将[`LOGGING_CONFIG`](#std:setting-LOGGING_CONFIG)设置为`None`，将跳过日志配置过程。

Changed in Django 1.7:

以前，默认值为`'django.utils.log.dictConfig'`。

### MANAGERS

默认值：`()`（空元组）

一个有 [`ADMINS`](#std:setting-ADMINS)相同格式的元组， 它指定了谁应该得到 broken link notifications 当[`BrokenLinkEmailsMiddleware`](middleware.html#django.middleware.common.BrokenLinkEmailsMiddleware "django.middleware.common.BrokenLinkEmailsMiddleware") 启用的时候.

### MEDIA_ROOT

缺省： `''` （空字符串）

指向存放[_用户上传文件_](../topics/files.html)所在目录的文件系统绝对路径。

例如： `"/var/www/example.com/media/"`

参见[`MEDIA_URL`](#std:setting-MEDIA_URL).

警告：

[`MEDIA_ROOT`](#std:setting-MEDIA_ROOT) 和 [`STATIC_ROOT`](#std:setting-STATIC_ROOT) 必须设置为不同的值。在引入（设置）[`STATIC_ROOT`](#std:setting-STATIC_ROOT) 之前,  静态文件的处理将依赖[`MEDIA_ROOT`](#std:setting-MEDIA_ROOT)。但是，由于这样做会导致产生隐藏的严重安全问题，所以必须进行有效的安全检查以避免这种情况发生。

### MEDIA_URL

缺省： `''` (空字符串)

MEDIA_URL指向[`MEDIA_ROOT`](#std:setting-MEDIA_ROOT)所指定的media文件，通过这个地址来[_管理所存储文件_](../topics/files.html)。该URL设置为非空值时，必须以斜杠“/”结束。你需要 [_配置这些文件用于_](../howto/static-files/index.html#serving-uploaded-files-in-development) 开发环境或线上环境。.

若你打算在模版中使用 `{{ MEDIA_URL }}` , 那么应在[`TEMPLATES`](#std:setting-TEMPLATES)的`'context_processors'`设置中添加`'django.template.context_processors.media'`.

例如： `"http://media.example.com/"`

警告

如果接受非授信用户上传的内容，将会给系统带来安全风险。关于迁移细节，请参见[_用户上传内容_](../topics/security.html#user-uploaded-content-security)中安全指南一节。

警告

[`MEDIA_URL`](#std:setting-MEDIA_URL) 和 [`STATIC_URL`](#std:setting-STATIC_URL) 必须设置为不同的值。更多细节，请参见 [`MEDIA_ROOT`](#std:setting-MEDIA_ROOT)。

### MIDDLEWARE_CLASSES

默认：

```
('django.middleware.common.CommonMiddleware',
 'django.middleware.csrf.CsrfViewMiddleware')

```

使用中间件类的元组 请参见[_Middleware_](../topics/http/middleware.html)。

Changed in Django 1.7:

已从此设置中移除[`SessionMiddleware`](middleware.html#django.contrib.sessions.middleware.SessionMiddleware "django.contrib.sessions.middleware.SessionMiddleware")，[`AuthenticationMiddleware`](middleware.html#django.contrib.auth.middleware.AuthenticationMiddleware "django.contrib.auth.middleware.AuthenticationMiddleware")和[`MessageMiddleware`](middleware.html#django.contrib.messages.middleware.MessageMiddleware "django.contrib.messages.middleware.MessageMiddleware")。

### MIGRATION_MODULES

默认：

```
{}  # empty dictionary

```

指定可在每个应用程序的基础上找到迁移模块的软件包的字典。此设置的默认值为空字典，但迁移模块的默认软件包名称为`migrations`。

例：

```
{'blog': 'blog.db_migrations'}

```

在这种情况下，与`blog`应用程序相关的迁移将包含在`blog.db_migrations`包中。

如果您提供`app_label`参数，则[`makemigrations`](django-admin.html#django-admin-makemigrations)将自动创建软件包（如果尚不存在）。

### MONTH_DAY_FORMAT

默认值：`'F j'`

用于Django管理更改列表页面上的日期字段（可能还包括系统的其他部分）的默认格式（仅显示月份和日期时）。

例如，当通过日期明细过滤Django管理更改列表页面时，给定日期的标题显示日期和月份。不同的区域设置具有不同的格式。例如，美国英语会说“1月1日”，而西班牙语可能会说“1 Enero”。

请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则相应的区域设置格式具有更高的优先级，并将应用。

请参阅[`allowed date format strings`](templates/builtins.html#std:templatefilter-date)。另请参阅[`DATE_FORMAT`](#std:setting-DATE_FORMAT)，[`DATETIME_FORMAT`](#std:setting-DATETIME_FORMAT)，[`TIME_FORMAT`](#std:setting-TIME_FORMAT)和[`YEAR_MONTH_FORMAT`](#std:setting-YEAR_MONTH_FORMAT)。

### NUMBER_GROUPING

默认值：`0`

在数字的整数部分上分组在一起的数字位数。

常用的是显示一千个分隔符。如果此设置为`0`，则不会对数字应用分组。如果此设置大于`0`，则[`THOUSAND_SEPARATOR`](#std:setting-THOUSAND_SEPARATOR)将用作这些组之间的分隔符。

请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则区域设置格式具有更高的优先级，将会应用。

另见[`DECIMAL_SEPARATOR`](#std:setting-DECIMAL_SEPARATOR)，[`THOUSAND_SEPARATOR`](#std:setting-THOUSAND_SEPARATOR)和[`USE_THOUSAND_SEPARATOR`](#std:setting-USE_THOUSAND_SEPARATOR)。

### PREPEND_WWW

默认值：`False`

是否将“www。”子网域添加到没有网址的网址。仅在安装[`CommonMiddleware`](middleware.html#django.middleware.common.CommonMiddleware "django.middleware.common.CommonMiddleware")时使用（请参阅[_Middleware_](../topics/http/middleware.html)）。另请参阅[`APPEND_SLASH`](#std:setting-APPEND_SLASH)。

### ROOT_URLCONF

默认值：没有定义

一个字符串，表示根URLconf 的完整Python 导入路径。例如：`"mydjangoapps.urls"`。每个请求可以覆盖它，方法是设置进来的`HttpRequest` 对象的`urlconf`属性。细节参见[_Django 如何处理一个请求_](../topics/http/urls.html#how-django-processes-a-request)。

### SECRET_KEY

默认值：`''`（空字符串）

特定Django安装的密钥。这用于提供[_cryptographic signing_](../topics/signing.html)，并应设置为唯一的不可预测的值。

[`django-admin startproject`](django-admin.html#django-admin-startproject)自动向每个新项目添加随机生成的`SECRET_KEY`

如果未设置[`SECRET_KEY`](#std:setting-SECRET_KEY)，Django将拒绝启动。

警告

**将此值保密。**

使用已知的[`SECRET_KEY`](#std:setting-SECRET_KEY)运行Django会导致许多Django的安全保护失效，并可能导致特权升级和远程代码执行漏洞。

密钥用于：

*   All [_sessions_](../topics/http/sessions.html) if you are using any other session backend than `django.contrib.sessions.backends.cache`, or if you use [`SessionAuthenticationMiddleware`](middleware.html#django.contrib.auth.middleware.SessionAuthenticationMiddleware "django.contrib.auth.middleware.SessionAuthenticationMiddleware") and are using the default [`get_session_auth_hash()`](../topics/auth/customizing.html#django.contrib.auth.models.AbstractBaseUser.get_session_auth_hash "django.contrib.auth.models.AbstractBaseUser.get_session_auth_hash").
*   如果您使用[`CookieStorage`](contrib/messages.html#django.contrib.messages.storage.cookie.CookieStorage "django.contrib.messages.storage.cookie.CookieStorage")或[`FallbackStorage`](contrib/messages.html#django.contrib.messages.storage.fallback.FallbackStorage "django.contrib.messages.storage.fallback.FallbackStorage")，则所有[_messages_](contrib/messages.html)。
*   [`formtools.wizard.views.CookieWizardView`](http://django-formtools.readthedocs.org/en/latest/wizard.html#formtools.wizard.views.CookieWizardView "(in django-formtools v1.0)")使用Cookie存储时，[`Form wizard`](http://django-formtools.readthedocs.org/en/latest/wizard.html#module-formtools.wizard.views "(in django-formtools v1.0)")
*   所有[`password_reset()`](../topics/auth/default.html#django.contrib.auth.views.password_reset "django.contrib.auth.views.password_reset")令牌。
*   正在进行[`form previews`](http://django-formtools.readthedocs.org/en/latest/preview.html#module-formtools.preview "(in django-formtools v1.0)")。
*   使用[_cryptographic signing_](../topics/signing.html)的任何用法，除非提供了不同的密钥。

如果您旋转密钥，上述所有内容都将失效。密钥不用于用户的密码，密钥轮换不会影响用户密码。

### SECURE_BROWSER_XSS_FILTER

New in Django 1.8.

默认值：`False`

如果`True`，[`SecurityMiddleware`](middleware.html#django.middleware.security.SecurityMiddleware "django.middleware.security.SecurityMiddleware")设置[_X-XSS-Protection: 1; mode=block_](middleware.html#x-xss-protection)标题中所有没有它的响应。

### SECURE_CONTENT_TYPE_NOSNIFF

New in Django 1.8.

默认值：`False`

如果`True`，则[`SecurityMiddleware`](middleware.html#django.middleware.security.SecurityMiddleware "django.middleware.security.SecurityMiddleware")在尚未拥有它的所有响应上设置[_X-Content-Type-Options: nosniff_](middleware.html#x-content-type-options)

### SECURE_HSTS_INCLUDE_SUBDOMAINS

New in Django 1.8.

默认值：`False`

如果`True`，[`SecurityMiddleware`](middleware.html#django.middleware.security.SecurityMiddleware "django.middleware.security.SecurityMiddleware")将`includeSubDomains`标记添加到[_HTTP Strict Transport Security_](middleware.html#http-strict-transport-security)除非[`SECURE_HSTS_SECONDS`](#std:setting-SECURE_HSTS_SECONDS)设置为非零值，否则它不起作用。

警告

设置不正确可能会不可逆转（一段时间）中断您的网站。首先阅读[_HTTP Strict Transport Security_](middleware.html#http-strict-transport-security)文档。

### SECURE_HSTS_SECONDS

New in Django 1.8.

默认值：`0`

如果设置为非零整数值，则[`SecurityMiddleware`](middleware.html#django.middleware.security.SecurityMiddleware "django.middleware.security.SecurityMiddleware")会在尚未拥有它的所有响应上设置[_HTTP Strict Transport Security_](middleware.html#http-strict-transport-security)标题。

警告

设置不正确可能会不可逆转（一段时间）中断您的网站。首先阅读[_HTTP Strict Transport Security_](middleware.html#http-strict-transport-security)文档。

### SECURE_PROXY_SSL_HEADER

默认值：`None`

表示表示请求的HTTP头/值组合的元组是安全的。这控制请求对象的`is_secure()`方法的行为。

这需要一些解释。默认情况下，`is_secure()`能够通过查看请求的网址是否使用“[https：//](https://)”来确定请求是否安全。这对于Django的CSRF保护很重要，可以由您自己的代码或第三方应用程序使用。

如果你的Django应用程序在代理后面，代理可能“吞下”一个请求是HTTPS的事实，使用代理和Django之间的非HTTPS连接。在这种情况下，`is_secure()`将始终返回`False` - 即使对于通过HTTPS由最终用户进行的请求。

在这种情况下，您需要配置代理以设置自定义HTTP标头，以通知Django请求是否通过HTTPS传入，并且您希望设置`SECURE_PROXY_SSL_HEADER`，以便Django知道哪个报头寻找。

您需要设置一个包含两个元素的元组：要查找的标题的名称和所需的值。例如：

```
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

```

在这里，我们告诉Django我们信任来自我们代理的`X-Forwarded-Proto`头部，并且任何时候它的值是`'https'`保证是安全的（即，它最初通过HTTPS进来）。显然，如果您控制您的代理或有其他保证它适当地设置/剥离此标头，您应该_仅_设置此设置。

请注意，标头必须采用`request.`META - 所有大写，可能从`HTTP_`开始。(Remember, Django automatically adds `'HTTP_'` to the start of x-header names before making the header available in `request.`META。）

警告

**如果你不知道你在做什么，你可能会在你的网站上打开安全漏洞。** 如果你没有设置它，当你应该。认真。

在设置之前，请确保以下所有条件成立（假设上述示例中的值）：

*   你的Django应用程序在代理后面。
*   您的代理会从所有传入的请求中取消`X-Forwarded-Proto`标头。换句话说，如果最终用户在其请求中包括该头，代理将丢弃它。
*   您的代理会设置`X-Forwarded-Proto`标头，并将其发送到Django，但仅适用于最初通过HTTPS进入的请求。

如果其中任何一个不正确，您应该将此设置设置为`None`，并找到另一种确定HTTPS的方法，也许通过自定义中间件。

### SECURE_REDIRECT_EXEMPT

New in Django 1.8.

默认值：`[]`

如果网址路径与此列表中的正则表达式匹配，则请求将不会重定向到HTTPS。如果[`SECURE_SSL_REDIRECT`](#std:setting-SECURE_SSL_REDIRECT)为`False`，则此设置无效。

### SECURE_SSL_HOST

New in Django 1.8.

默认值：`None`

如果字符串（例如`secure.example.com`），所有SSL重定向将被定向到此主机，而不是原始请求的主机（例如`www.example.com`） 。如果[`SECURE_SSL_REDIRECT`](#std:setting-SECURE_SSL_REDIRECT)为`False`，则此设置无效。

### SECURE_ SL_REDIRECT

New in Django 1.8.

默认值：`False`。

If `True`, the [`SecurityMiddleware`](middleware.html#django.middleware.security.SecurityMiddleware "django.middleware.security.SecurityMiddleware") [_redirects_](middleware.html#ssl-redirect) all non-HTTPS requests to HTTPS (except for those URLs matching a regular expression listed in [`SECURE_REDIRECT_EXEMPT`](#std:setting-SECURE_REDIRECT_EXEMPT)).

注意

如果将此设置为`True`会导致无限重定向，则可能意味着您的网站在代理后运行，无法分辨哪些请求是安全的，哪些请求不安全。您的代理可能设置标头来指示安全请求；您可以通过找出该标题并相应地配置[`SECURE_PROXY_SSL_HEADER`](#std:setting-SECURE_PROXY_SSL_HEADER)设置来更正该问题。

### SERIALIZATION_MODULES

默认值：未定义。

包含序列化程序定义（作为字符串提供）的模块字典，由该序列化类型的字符串标识符键入。例如，要定义YAML序列化程序，请使用：

```
SERIALIZATION_MODULES = {'yaml': 'path.to.yaml_serializer'}

```

### SERVER_EMAIL

默认值：`'root@localhost'`

错误消息来源的电子邮件地址，例如发送到[`ADMINS`](#std:setting-ADMINS)和[`MANAGERS`](#std:setting-MANAGERS)的电子邮件地址。

为什么我的电子邮件是从其他地址发送的？

此地址仅用于错误消息。_不是_与[`send_mail()`](../topics/email.html#django.core.mail.send_mail "django.core.mail.send_mail")一起发送的常规电子邮件的地址来自；有关详情，请参阅[`DEFAULT_FROM_EMAIL`](#std:setting-DEFAULT_FROM_EMAIL)。

### SHORT_DATE_FORMAT

默认值：`m/d/Y`（例如`12/31/2003`）

可用于在模板上显示日期字段的可用格式。请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则相应的区域设置格式具有更高的优先级，并将应用。请参阅[`allowed date format strings`](templates/builtins.html#std:templatefilter-date)。

另请参阅[`DATE_FORMAT`](#std:setting-DATE_FORMAT)和[`SHORT_DATETIME_FORMAT`](#std:setting-SHORT_DATETIME_FORMAT)。

### SHORT_DATETIME_FORMAT

预设值：`m / d / Y P`（例如`12/31/2003 4 pm`）

可用于在模板上显示datetime字段的可用格式。请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则相应的区域设置格式具有更高的优先级，并将应用。请参阅[`allowed date format strings`](templates/builtins.html#std:templatefilter-date)。

另请参阅[`DATE_FORMAT`](#std:setting-DATE_FORMAT)和[`SHORT_DATE_FORMAT`](#std:setting-SHORT_DATE_FORMAT)。

### SIGNING_BACKEND

默认值：`'django.core.signing.TimestampSigner'`

后端用于签名Cookie和其他数据。

另请参阅[_Cryptographic signing_](../topics/signing.html)文档。

### SILENCED_SYSTEM_CHECKS

New in Django 1.7.

默认值：`[]`

您希望永久确认和忽略的系统检查框架生成的消息的标识符列表（即`["models.W001"]`）。静音警告将不再输出到控制台；静默错误仍将被打印，但不会阻止管理命令运行。

另请参见[_System check framework_](checks.html)文档。

### TEMPLATES

New in Django 1.8.

默认:: `[]`（空列表）

Django的模板使用一个列表来进行配置。列表中每一项都是一个字典类型数据，可以配置模板不同的功能。

这里有一个简单的设置，告诉Django模板引擎从已安装的应用程序中的`模板`子目录加载模板：

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'APP_DIRS': True,
    },
]

```

以下选项适用于所有后端。

#### BACKEND

默认值：未定义

要使用的模板后端。内置模板后端是：

*   `'django.template.backends.django.DjangoTemplates'`
*   `'django.template.backends.jinja2.Jinja2'`

通过将`BACKEND`设置为完全限定路径（即`'mypackage.whatever.Backend'`），您可以使用未随Django提供的模板后端。

#### NAME

默认值：见下面

此特定模板引擎的别名。它是一个标识符，允许选择引擎进行渲染。别名在所有已配置的模板引擎中必须是唯一的。

它默认为定义引擎类的模块的名称，即[`BACKEND`](#std:setting-TEMPLATES-BACKEND)的下一段，当没有提供时。例如，如果后端是`'mypackage.whatever.Backend'`，则其默认名称为`'whatever'`。

#### DIRS

默认设置： `[]` (空列表)

包含搜索顺序的序列，搜索引擎会按照这个顺序查找template资源文件

#### APP_DIRS

默认:: `False`

Templates引擎是否应该在已安装的app中查找Template源文件

#### OPTIONS

默认:: `{}`（空dict）

传递给模板后端的额外参数。可用参数因模板后端而异。

### TEMPLATE_CONTEXT_PROCESSORS

默认：

```
("django.contrib.auth.context_processors.auth",
"django.template.context_processors.debug",
"django.template.context_processors.i18n",
"django.template.context_processors.media",
"django.template.context_processors.static",
"django.template.context_processors.tz",
"django.contrib.messages.context_processors.messages")

```

自1.8版起已弃用：在`DjangoTemplates`后端的[`OPTIONS`](#std:setting-TEMPLATES-OPTIONS)中设置`'context_processors'`选项。

用于填充`RequestContext`中上下文的可调用元组。这些callables接受请求对象作为它们的参数，并返回要合并到上下文中的项目的字典。

Changed in Django 1.8:

在Django 1.8中，内置模板上下文处理器已从`django.core.context_processors`移至`django.template.context_processors`。

### TEMPLATE_DEBUG

默认值：`False`

自1.8版起已弃用：在`DjangoTemplates`后端的[`OPTIONS`](#std:setting-TEMPLATES-OPTIONS)中设置`'debug'`选项。

打开/关闭模板调试模式的布尔值。如果这是`True`，花哨的错误页面将显示模板渲染期间引发的任何异常的详细报告。此报告包含模板的相关代码段，并突出显示相应的行。

请注意，如果[`DEBUG`](#std:setting-DEBUG)是`True`，Django只会显示精美的错误页面，因此您需要设置它以利用此设置。

另请参见[`DEBUG`](#std:setting-DEBUG)。

### TEMPLATE_DIRS

默认值：`()`（空元组）

自1.8版起已弃用：改为设置`DjangoTemplates`后端的[`DIRS`](#std:setting-TEMPLATES-DIRS)选项。

按照搜索顺序，由[`django.template.loaders.filesystem.Loader`](templates/api.html#django.template.loaders.filesystem.Loader "django.template.loaders.filesystem.Loader")搜索的模板源文件的位置列表。

注意，这些路径应该使用Unix风格的正斜杠，即使在Windows上。

请参阅[_The Django template language_](templates/language.html)。

### TEMPLATE_LOADERS

默认：

```
('django.template.loaders.filesystem.Loader',
 'django.template.loaders.app_directories.Loader')

```

自1.8版起已弃用：在`DjangoTemplates`后端的[`OPTIONS`](#std:setting-TEMPLATES-OPTIONS)中设置`'loaders'`选项。

一个模板加载器类的元组，指定为字符串。每个`Loader`类知道如何从特定源导入模板。可选地，可以使用元组而不是字符串。元组中的第一项应该是`Loader`的模块，后续项在初始化期间传递到`Loader`。请参阅[_The Django template language: for Python programmers_](templates/api.html)。

### TEMPLATE_STRING_IF_INVALID

默认值：`''`（空字符串）

自1.8版起已弃用：在`DjangoTemplates`后端的[`OPTIONS`](#std:setting-TEMPLATES-OPTIONS)中设置`'string_if_invalid'`选项。

输出为字符串，模板系统应该用于无效（例如拼写错误的）变量。请参见[_How invalid variables are handled_](templates/api.html#invalid-template-variables)。

### TEST_RUNNER

默认值：`'django.test.runner.DiscoverRunner'`

用于启动测试套件的类的名称。请参见[_Using different testing frameworks_](../topics/testing/advanced.html#other-testing-frameworks)。

### TEST_NON_SERIALIZED_APPS

New in Django 1.7.

默认值：`[]`

为了在`TransactionTestCase`的测试和没有事务的数据库后端之间恢复数据库状态，Django会在开始测试运行时[_serialize the contents of all apps with migrations_](../topics/testing/overview.html#test-case-serialized-rollback)然后可以在需要它的测试之前从该副本重新加载。

这减慢了测试运行器的启动时间；如果您拥有不需要此功能的应用，则可以在此处添加他们的全名（例如`'django.contrib.contenttypes'`），以将其从此序列化过程中排除。

### THOUSAND_SEPARATOR

默认值：`,`（逗号）

格式化数字时使用的默认千分位数。此设置仅在[`USE_THOUSAND_SEPARATOR`](#std:setting-USE_THOUSAND_SEPARATOR)为`True`且[`NUMBER_GROUPING`](#std:setting-NUMBER_GROUPING)大于`0`时使用。

请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则区域设置格式具有更高的优先级，将会应用。

另请参阅[`NUMBER_GROUPING`](#std:setting-NUMBER_GROUPING)，[`DECIMAL_SEPARATOR`](#std:setting-DECIMAL_SEPARATOR)和[`USE_THOUSAND_SEPARATOR`](#std:setting-USE_THOUSAND_SEPARATOR)。

### 时间格式

默认值：`'P'`（例如`4 p.m。`）

用于在系统任何部分中显示时间字段的默认格式。请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则区域设置格式具有更高的优先级，将会应用。请参阅[`allowed date format strings`](templates/builtins.html#std:templatefilter-date)。

另请参阅[`DATE_FORMAT`](#std:setting-DATE_FORMAT)和[`DATETIME_FORMAT`](#std:setting-DATETIME_FORMAT)。

### TIME_INPUT_FORMATS

默认：

```
(
    '%H:%M:%S',     # '14:30:59'
    '%H:%M:%S.%f',  # '14:30:59.000200'
    '%H:%M',        # '14:30'
)

```

在时间字段上输入数据时将接受的格式的元组。格式将按顺序尝试，使用第一个有效的格式。请注意，这些格式字符串使用Python的[datetime](https://docs.python.org/library/datetime.html#strftime-strptime-behavior)模块语法，而不是来自`date` Django模板标记的格式字符串。

当[`USE_L10N`](#std:setting-USE_L10N)为`True`时，区域设置格式具有更高的优先级，将会应用。

另请参阅[`DATE_INPUT_FORMATS`](#std:setting-DATE_INPUT_FORMATS)和[`DATETIME_INPUT_FORMATS`](#std:setting-DATETIME_INPUT_FORMATS)。

### 时区

默认：`'America/Chicago'`

一个字符串或者`None`，表示项目的时区。参见[时区列表](http://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。

注

因为Django 第一次发布时，[`TIME_ZONE`](#std:setting-TIME_ZONE) 设置为 `'America/Chicago'`，为了向前兼容，全局设置（ 在你的项目的`settings.py` 中没有定义任何内容时使用）仍然保留为`'America/Chicago'`。 新的项目模板默认为`'UTC'`。

注意，它不一定要和服务器的时区一致。例如，一个服务器可上可能有多个Django 站点，每个站点都有一个单独的时区设置。

当[`USE_TZ`](#std:setting-USE_TZ) 为`False` 时，它将成为Django 存储所有日期和时间时使用的时区。当[`USE_TZ`](#std:setting-USE_TZ) 为`True` 时，它是Django 显示模板中以及解释表单中的日期和时间默认使用的时区。

Django 设置`os.environ['TZ']` 变量为你在[`TIME_ZONE`](#std:setting-TIME_ZONE) 设置中指定的时区。所以，你的所有视图和模型都将自动在这个时区中运作。然而，在下面这些情况下，Django 不会设置`TZ` 环境变量：

*   如果你使用手工配置选项，参见[_手工配置设置_](../topics/settings.html#settings-without-django-settings-module)，或
*   如果你指定`TIME_ZONE = None`。这将导致Django 使用系统的时区。然而，当[`USE_TZ = True`](#std:setting-USE_TZ) 时不鼓励这样做，因为这使得本地时间和UTC 之间的转换不太可靠。

如果Django 没有设置`TZ` 环境变量，那么你需要自己确保你的进程在正确的环境中运行。

注

在Windows 环境中，Django 不能可靠地交替其它时区。如果你在Windows 上运行Django，[`TIME_ZONE`](#std:setting-TIME_ZONE) 必须设置为与系统时区一致。

### USE_ETAGS

默认值：`False`

这是一个布尔变量，它指定是否产生"Etag"头，这种方式会节省带宽但是会降低性能，这个标签在 `CommonMiddleware` (see [_Middleware_](../topics/http/middleware.html)) 和``Cache Framework`` 中使用(详情见[_Django’s cache framework_](../topics/cache.html)).

### USE_I18N

默认值：`True`

这是一个布尔值，它指定Django的翻译系统是否被启用。它提供了一种简单的方式去关闭翻译系统。如果设置为 `False`, Django 会做一些优化，不去加载翻译机制

另请参见[`LANGUAGE_CODE`](#std:setting-LANGUAGE_CODE)，[`USE_L10N`](#std:setting-USE_L10N)和[`USE_TZ`](#std:setting-USE_TZ)。

### USE_L10N

默认值：`False`

是一个布尔值，用于决定是否默认进行日期格式本地化。如果此设置为`True`，例如Django将使用当前语言环境的格式显示数字和日期。

另请参见[`LANGUAGE_CODE`](#std:setting-LANGUAGE_CODE)，[`USE_I18N`](#std:setting-USE_I18N)和[`USE_TZ`](#std:setting-USE_TZ)。

注意

[`django-admin startproject`](django-admin.html#django-admin-startproject)创建的默认`settings.py`文件包括`USE_L10N = True`。

### USE_THOUSAND_SEPARATOR

默认值：`False`

一个布尔值，指定是否使用千位分隔符显示数字。当[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，并且此值也设置为`True`时，Django将使用[`THOUSAND_SEPARATOR`](#std:setting-THOUSAND_SEPARATOR)和[`NUMBER_GROUPING`](#std:setting-NUMBER_GROUPING)以格式化数字。

另见[`DECIMAL_SEPARATOR`](#std:setting-DECIMAL_SEPARATOR)，[`NUMBER_GROUPING`](#std:setting-NUMBER_GROUPING)和[`THOUSAND_SEPARATOR`](#std:setting-THOUSAND_SEPARATOR)。

### USE_TZ

默认: `False`

这是一个布尔值,用来指定是否使用指定的时区(TIME_ZONE)的时间.若为 `True`, 则Django 会使用内建的时区的时间否则, Django 将会使用本地的时间

另请参阅[`TIME_ZONE`](#std:setting-TIME_ZONE)，[`USE_I18N`](#std:setting-USE_I18N)和[`USE_L10N`](#std:setting-USE_L10N)。

注意

使用django-admin startproject创建的项目中的 `settings.py` 文件中,[](django-admin.html#django-admin-startproject)为了方便将 `USE_TZ 设置为 True`

### USE_X_FORWARDED_HOST

默认值：`False`

一个布尔值，指定是否使用X-Forwarded-Host头优先于主机头。只有在使用设置此标头的代理时，才应启用此选项。

### WSGI_APPLICATION

默认值：`None`

Django的内置服务器（例如[`runserver`](django-admin.html#django-admin-runserver)）将使用的WSGI应用程序对象的完整Python路径。The [`django-admin startproject`](django-admin.html#django-admin-startproject) management command will create a simple `wsgi.py` file with an `application` callable in it, and point this setting to that `application`.

如果未设置，将使用`django.core.wsgi.get_wsgi_application()`的返回值。在这种情况下，[`runserver`](django-admin.html#django-admin-runserver)的行为将与以前的Django版本相同。

### YEAR_MONTH_FORMAT

默认值：`'F Y'`

用于Django管理更改列表页面上的日期字段（可能还包括系统的其他部分）的默认格式，以便仅显示年份和月份。

例如，当通过日期明细过滤Django管理更改列表页面时，给定月份的标题显示月份和年份。不同的区域设置具有不同的格式。例如，美国英语会说“2006年1月”，而另一个地区可能说“2006 / January”。

请注意，如果[`USE_L10N`](#std:setting-USE_L10N)设置为`True`，则相应的区域设置格式具有更高的优先级，并将应用。

请参阅[`allowed date format strings`](templates/builtins.html#std:templatefilter-date)。另请参阅[`DATE_FORMAT`](#std:setting-DATE_FORMAT)，[`DATETIME_FORMAT`](#std:setting-DATETIME_FORMAT)，[`TIME_FORMAT`](#std:setting-TIME_FORMAT)和[`MONTH_DAY_FORMAT`](#std:setting-MONTH_DAY_FORMAT)。

### X_FRAME_OPTIONS

默认值：`'SAMEORIGIN'`

[`XFrameOptionsMiddleware`](middleware.html#django.middleware.clickjacking.XFrameOptionsMiddleware "django.middleware.clickjacking.XFrameOptionsMiddleware")使用的X-Frame-Options标头的默认值。请参阅[_clickjacking protection_](clickjacking.html)文档。

## Auth

用于[`django.contrib.auth`](../topics/auth/index.html#module-django.contrib.auth "django.contrib.auth: Django's authentication framework.") 的设置。

### AUTHENTICATION_BACKENDS

默认：`('django.contrib.auth.backends.ModelBackend',)`

一个元组，包含认证后台的类（字符串形式），用于认证用户。详细信息参见[_认证后台的文档_](../topics/auth/customizing.html#authentication-backends)。

### AUTH_USER_MODEL

默认：‘auth.用户'

表示用户的模型。参见[_自定义用户模型_](../topics/auth/customizing.html#auth-custom-user)。

警告

在项目的生命周期内（即，一旦完成并迁移了依赖于它的模型），您不能在没有认真努力的情况下更改AUTH_USER_MODEL设置。它旨在在项目开始时设置，并且其引用的模型必须在其所驻留的应用程序的第一次迁移中可用。有关详细信息，请参阅[_Substituting a custom User model_](../topics/auth/customizing.html#auth-custom-user)。

### LOGIN_REDIRECT_URL

默认：`'/accounts/profile/'`

登录之后，`contrib.auth.login` 视图找不到`next` 参数时，请求被重定向到的URL。

例如，它被[`login_required()`](../topics/auth/default.html#django.contrib.auth.decorators.login_required "django.contrib.auth.decorators.login_required") 装饰器使用。

这个设置还接收视图函数名称和[_命名的URL 模式_](../topics/http/urls.html#naming-url-patterns)，它可以减少重复的配置，因为这样你就不需要在两个地方定义该URL（`settings` 和URLconf）。

### LOGIN_URL

默认：`'/accounts/login/'`

登录的URL，特别是使用[`login_required()`](../topics/auth/default.html#django.contrib.auth.decorators.login_required "django.contrib.auth.decorators.login_required") 装饰器的时候。

这个设置还接收视图函数名称和[_命名的URL 模式_](../topics/http/urls.html#naming-url-patterns)，它可以减少重复的配置，因为这样你就不需要在两个地方定义该URL（`settings` 和URLconf）。

### LOGOUT_URL

默认：`'/accounts/logout/'`

与LOGIN_URL 配对。

### PASSWORD_RESET_TIMEOUT_DAYS

默认：`3`

重置密码的链接有效的天数。用于[`django.contrib.auth`](../topics/auth/index.html#module-django.contrib.auth "django.contrib.auth: Django's authentication framework.") 的重置密码功能。

### PASSWORD_HASHERS

参见[_Django 如何存储密码_](../topics/auth/passwords.html#auth-password-storage)。

默认：

```
('django.contrib.auth.hashers.PBKDF2PasswordHasher',
 'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
 'django.contrib.auth.hashers.BCryptPasswordHasher',
 'django.contrib.auth.hashers.SHA1PasswordHasher',
 'django.contrib.auth.hashers.MD5PasswordHasher',
 'django.contrib.auth.hashers.UnsaltedMD5PasswordHasher',
 'django.contrib.auth.hashers.CryptPasswordHasher')

```

## 消息

[`django.contrib.messages`](contrib/messages.html#module-django.contrib.messages "django.contrib.messages: Provides cookie- and session-based temporary message storage.")的设置。

### MESSAGE_LEVEL

默认值：`messages.INFO`

设置消息框架将记录的最小消息级别。有关详细信息，请参阅[_message levels_](contrib/messages.html#message-level)。

重要

如果您在设置文件中替换`MESSAGE_LEVEL`并依赖于任何内置常量，则必须直接导入常数模块，以避免循环导入的可能性，例如：

```
from django.contrib.messages import constants as message_constants
MESSAGE_LEVEL = message_constants.DEBUG

```

如果需要，可以根据上述[_constants table_](contrib/messages.html#message-level-constants)中的值直接指定常数的数值。

### MESSAGE_STORAGE

默认值：`'django.contrib.messages.storage.fallback.FallbackStorage'`

控制Django在哪里存储消息数据。有效值为：

*   `'django.contrib.messages.storage.fallback.FallbackStorage'`
*   `'django.contrib.messages.storage.session.SessionStorage'`
*   `'django.contrib.messages.storage.cookie.CookieStorage'`

有关详细信息，请参阅[_message storage backends_](contrib/messages.html#message-storage-backends)。

使用Cookie的后端 - [`CookieStorage`](contrib/messages.html#django.contrib.messages.storage.cookie.CookieStorage "django.contrib.messages.storage.cookie.CookieStorage")和[`FallbackStorage`](contrib/messages.html#django.contrib.messages.storage.fallback.FallbackStorage "django.contrib.messages.storage.fallback.FallbackStorage") - 使用[`SESSION_COOKIE_DOMAIN`](#std:setting-SESSION_COOKIE_DOMAIN)，[`SESSION_COOKIE_SECURE`](#std:setting-SESSION_COOKIE_SECURE)和[`SESSION_COOKIE_HTTPONLY`](#std:setting-SESSION_COOKIE_HTTPONLY)。

### MESSAGE_TAGS

默认：

```
{messages.DEBUG: 'debug',
messages.INFO: 'info',
messages.SUCCESS: 'success',
messages.WARNING: 'warning',
messages.ERROR: 'error'}

```

这设置消息级别到消息标记的映射，通常呈现为HTML中的CSS类。如果指定一个值，它将扩展默认值。这意味着您只需要指定需要覆盖的那些值。有关详细信息，请参阅上面的[_Displaying messages_](contrib/messages.html#message-displaying)。

重要

如果您在设置文件中替换`MESSAGE_TAGS`并依赖任何内置常数，则必须直接导入`constants`模块，以避免循环导入的可能性，例如：

```
from django.contrib.messages import constants as message_constants
MESSAGE_TAGS = {message_constants.INFO: ''}

```

如果需要，可以根据上述[_constants table_](contrib/messages.html#message-level-constants)中的值直接指定常数的数值。

## 会话

[`django.contrib.sessions`](../topics/http/sessions.html#module-django.contrib.sessions "django.contrib.sessions: Provides session management for Django projects.")的设置。

### SESSION_CACHE_ALIAS

默认: `默认缓存设置`

使用 [_缓存存储会话时_](../topics/http/sessions.html#cached-sessions-backend), 使用何种缓存

### SESSION_COOKIE_AGE

默认：`1209600`（2个星期，以秒数为单位）

会话Cookie 的过期时间，以秒数为单位。

### SESSION_COOKIE_DOMAIN

默认: `无`

域名用于做会话的cookies.将类似于`".example.com"` (注意开头的点.) 这样的字符串设置为跨域的cookies, 或者使用 `None` 作为一个标准的域名cookie.

在生产站点上更新此设置时要小心。如果您更新此设置以在之前使用标准域Cookie的网站上启用跨域Cookie，则现有用户Cookie将设置为旧域。这可能会导致他们无法登录，只要这些cookie持续。

此设置还会影响由[`django.contrib.messages`](contrib/messages.html#module-django.contrib.messages "django.contrib.messages: Provides cookie- and session-based temporary message storage.")设置的Cookie。

### SESSION_COOKIE_HTTPONLY

默认值：`True`

是否在会话Cookie上使用`HTTPOnly`标志。如果设置为`True`，客户端JavaScript将无法访问会话Cookie。

[HTTPOnly](https://www.owasp.org/index.php/HTTPOnly)是Set-Cookie HTTP响应标头中包含的标志。它不是Cookie的 [**RFC 2109**](http://tools.ietf.org/html/rfc2109.html)然而，当它得到尊重时，它可以是一种有用的方式来减轻客户端脚本访问受保护的Cookie数据的风险。

启用它，使攻击者将跨站点脚本漏洞升级为完全劫持用户会话变得不那么简单。有没有什么借口离开这个，或者：如果你的代码依赖于阅读会话cookie从JavaScript，你可能是做错了。

New in Django 1.7.

此设置还会影响由[`django.contrib.messages`](contrib/messages.html#module-django.contrib.messages "django.contrib.messages: Provides cookie- and session-based temporary message storage.")设置的Cookie。

### SESSION_COOKIE_NAME

默认值：`'sessionid'`

要用于会话的Cookie的名称。这可以是您想要的（但应该不同于[`LANGUAGE_COOKIE_NAME`](#std:setting-LANGUAGE_COOKIE_NAME)）。

### SESSION_COOKIE_PATH

默认值：`'/'`

在会话cookie上设置的路径。这应该匹配您的Django安装的URL路径或该路径的父级。

如果您有多个运行在相同主机名下的Django实例，这将非常有用。他们可以使用不同的cookie路径，每个实例只会看到自己的会话cookie。

### SESSION_COOKIE_SECURE

默认值：`False`

是否对会话cookie使用安全cookie。如果此设置为`True`，则Cookie将被标记为“安全”，这意味着浏览器可以确保该Cookie仅在HTTPS连接下发送。

由于如果会话cookie未加密发送，数据包嗅探器（例如[Firesheep](http://codebutler.com/firesheep)）就劫持用户的会话，这是很平常的，真的没有什么好的借口离开这个。它会阻止你使用不安全的请求会话，这是一件好事。

New in Django 1.7.

此设置还会影响由[`django.contrib.messages`](contrib/messages.html#module-django.contrib.messages "django.contrib.messages: Provides cookie- and session-based temporary message storage.")设置的Cookie。

### SESSION_ENGINE

 默认：`django.contrib.sessions.backends.db`

控制Django 在哪里存储会话数据。包含的引擎有：

*   `'django.contrib.sessions.backends.db'`
*   `'django.contrib.sessions.backends.file'`
*   `'django.contrib.sessions.backends.cache'`
*   `'django.contrib.sessions.backends.cached_db'`
*   `'django.contrib.sessions.backends.signed_cookies'`

更多细节参见[_配置会话引擎_](../topics/http/sessions.html#configuring-sessions)。

### SESSION_EXPIRE_AT_BROWSER_CLOSE

默认值：`False`

是否在用户关闭浏览器时过期会话。请参阅[_Browser-length sessions vs. persistent sessions_](../topics/http/sessions.html#browser-length-vs-persistent-sessions)。

### SESSION_FILE_PATH

默认值：`None`

如果使用基于文件的会话存储，这将设置Django存储会话数据的目录。当使用默认值（`None`）时，Django将使用系统的标准临时目录。

### SESSION_SAVE_EVERY_REQUEST

默认值：`False`

是否保存每个请求的会话数据。如果这是`False`（默认值），那么会话数据只有在被修改时才被保存 - 也就是说，如果它的任何字典值被赋值或删除。

### SESSION_SERIALIZER

默认值：`'django.contrib.sessions.serializers.JSONSerializer'`

用于序列化会话数据的序列化类的完整导入路径。包括的序列化器有：

*   `'django.contrib.sessions.serializers.PickleSerializer'`
*   `'django.contrib.sessions.serializers.JSONSerializer'`

有关详细信息，请参阅[_Session serialization_](../topics/http/sessions.html#session-serialization)，包括使用[`PickleSerializer`](../topics/http/sessions.html#django.contrib.sessions.serializers.PickleSerializer "django.contrib.sessions.serializers.PickleSerializer")时可能的远程代码执行的警告。

## 网站

[`django.contrib.sites`](contrib/sites.html#module-django.contrib.sites "django.contrib.sites: Lets you operate multiple Web sites from the same database and Django project") 的设置。

### 网站ID

默认：未定义

当前站点在`django_site` 数据库表中的ID，为一个整数。这是用来让应用程序数据可以连接到特定的网站和一个单一的数据库可以管理多个站点的内容

## 静态文件

设置为 [`django.contrib.staticfiles`](contrib/staticfiles.html#module-django.contrib.staticfiles "django.contrib.staticfiles: An app for handling static files.").

### STATIC_ROOT

默认: `None`

[`collectstatic`](contrib/staticfiles.html#django-admin-collectstatic)用于部署而收集的静态文件的目录的绝对路径。

示例：`"/var/www/example.com/static/"`

如果 [_staticfiles_](contrib/staticfiles.html) 启用这个服务应用程序 (默认) [`collectstatic`](contrib/staticfiles.html#django-admin-collectstatic) 管理命令将收集的静态文件到这个目录查看如何在 [_managing static files_](../howto/static-files/index.html)有关使用的更多细节。

提醒

这应该是一个 (空目录) 的目录，用于从原始目录收集静态文件到这个目录，便于部署。它 **不是**永久存储静态文件的地方。你应该在[_staticfiles_](contrib/staticfiles.html)的[`finders`](#std:setting-STATICFILES_FINDERS)找到的目录中，它默认是`'static/'` app子目录以及您在[`STATICFILES_DIRS`](#std:setting-STATICFILES_DIRS)中包含的任何目录）。

### STATIC_URL

默认值: `None`

引用位于[`STATIC_ROOT`](#std:setting-STATIC_ROOT)中的静态文件时使用的网址。

示例：`"/static/"`或`"http://static.example.com/"`

如果不是`None`，则将用作[_asset definitions_](../topics/forms/media.html#form-asset-paths)（`Media`类）和[_staticfiles app_](contrib/staticfiles.html)

如果设置为非空值，它必须以斜杠结尾。

您可能需要[_configure these files to be served in development_](../howto/static-files/index.html#serving-static-files-in-development)，并且肯定需要在生产中执行[_in production_](../howto/static-files/deployment.html)

### STATICFILES_DIRS

默认值：`[]`

此设置定义了在启用`FileSystemFinder` finder时staticfiles应用程序将遍历的附加位置。如果您使用[`collectstatic`](contrib/staticfiles.html#django-admin-collectstatic)或[`findstatic`](contrib/staticfiles.html#django-admin-findstatic)管理命令或使用静态文件提供视图。

这应该设置为一个列表或元组的字符串，其中包含您的额外文件目录的完整路径例如：

```
STATICFILES_DIRS = (
    "/home/special.polls.com/polls/static",
    "/home/polls.com/polls/static",
    "/opt/webfiles/common",
)

```

请注意，即使在Windows上（例如`"C:/Users/user/mysite/extra_static_content"`），这些路径应使用Unix样式的正斜杠。

#### 前缀（可选）

如果要使用其他命名空间引用其中一个位置中的文件，可以**（可选）**提供前缀`（前缀， 路径） `元组，例如：

```
STATICFILES_DIRS = (
    # ...
    ("downloads", "/opt/webfiles/stats"),
)

```

For example, assuming you have [`STATIC_URL`](#std:setting-STATIC_URL) set to `'/static/'`, the [`collectstatic`](contrib/staticfiles.html#django-admin-collectstatic) management command would collect the “stats” files in a `'downloads'` subdirectory of [`STATIC_ROOT`](#std:setting-STATIC_ROOT).

这将允许您使用`'/static/downloads/polls_20101022.tar.gz'`来引用本地文件`'/opt/webfiles/stats/polls_20101022.tar.gz'`

```
<a href="{% static "downloads/polls_20101022.tar.gz" %}">

```

### STATICFILES_STORAGE

默认值：`'django.contrib.staticfiles.storage.StaticFilesStorage'`

使用[`collectstatic`](contrib/staticfiles.html#django-admin-collectstatic)管理命令收集静态文件时使用的文件存储引擎。

可在`django.contrib.staticfiles.storage.staticfiles_storage`中找到此设置中定义的存储后端的即时使用实例。

有关示例，请参阅[_Serving static files from a cloud service or CDN_](../howto/static-files/deployment.html#staticfiles-from-cdn)。

### STATICFILES_FINDERS

默认值:

```
("django.contrib.staticfiles.finders.FileSystemFinder",
 "django.contrib.staticfiles.finders.AppDirectoriesFinder")

```

finder后端列表，不同finder用来在不同的位置搜索静态文件。

默认设置是在 [`STATICFILES_DIRS`](#std:setting-STATICFILES_DIRS) (使用 `django.contrib.staticfiles.finders.FileSystemFinder`) 和每个应用的子目录 `static` (使用 `django.contrib.staticfiles.finders.AppDirectoriesFinder`)中搜索.如果存在多个具有相同名称的文件，则将使用找到的第一个文件。

查找器 `django.contrib.staticfiles.finders.DefaultStorageFinder`默认情况下是被禁用的.如果添加到您的[`STATICFILES_FINDERS`](#std:setting-STATICFILES_FINDERS)设置，它将在默认文件存储中查找由[`DEFAULT_FILE_STORAGE`](#std:setting-DEFAULT_FILE_STORAGE)设置定义的静态文件。

注意

使用`AppDirectoriesFinder`查找工具时，请确保您的应用可以通过静态文件找到。只需将应用程式新增至您网站的[`INSTALLED_APPS`](#std:setting-INSTALLED_APPS)设定即可。

静态文件查找器目前被认为是一个私有接口，因此这个接口是未被文档记录的。

## 核心设置主题索引

### 缓存

*   [高速缓存](#std:setting-CACHES)
*   [CACHE_MIDDLEWARE_ALIAS](#std:setting-CACHE_MIDDLEWARE_ALIAS)
*   [CACHE_MIDDLEWARE_KEY_PREFIX](#std:setting-CACHE_MIDDLEWARE_KEY_PREFIX)
*   [CACHE_MIDDLEWARE_SECONDS](#std:setting-CACHE_MIDDLEWARE_SECONDS)

### 数据库

*   [DATABASES](#std:setting-DATABASES)
*   [DATABASE_ROUTERS](#std:setting-DATABASE_ROUTERS)
*   [DEFAULT_INDEX_TABLESPACE](#std:setting-DEFAULT_INDEX_TABLESPACE)
*   [DEFAULT_TABLESPACE](#std:setting-DEFAULT_TABLESPACE)

### 调试

*   [调试](#std:setting-DEBUG)
*   [DEBUG_PROPAGATE_EXCEPTIONS](#std:setting-DEBUG_PROPAGATE_EXCEPTIONS)

### 电子邮件

*   [管理](#std:setting-ADMINS)
*   [DEFAULT_CHARSET](#std:setting-DEFAULT_CHARSET)
*   [DEFAULT_FROM_EMAIL](#std:setting-DEFAULT_FROM_EMAIL)
*   [EMAIL_BACKEND](#std:setting-EMAIL_BACKEND)
*   [EMAIL_FILE_PATH](#std:setting-EMAIL_FILE_PATH)
*   [EMAIL_HOST](#std:setting-EMAIL_HOST)
*   [EMAIL_HOST_PASSWORD](#std:setting-EMAIL_HOST_PASSWORD)
*   [EMAIL_HOST_USER](#std:setting-EMAIL_HOST_USER)
*   [EMAIL_PORT](#std:setting-EMAIL_PORT)
*   [EMAIL_SSL_CERTFILE](#std:setting-EMAIL_SSL_CERTFILE)
*   [EMAIL_SSL_KEYFILE](#std:setting-EMAIL_SSL_KEYFILE)
*   [EMAIL_SUBJECT_PREFIX](#std:setting-EMAIL_SUBJECT_PREFIX)
*   [EMAIL_TIMEOUT](#std:setting-EMAIL_TIMEOUT)
*   [EMAIL_USE_TLS](#std:setting-EMAIL_USE_TLS)
*   [经理](#std:setting-MANAGERS)
*   [SERVER_EMAIL](#std:setting-SERVER_EMAIL)

### 报告错误

*   [DEFAULT_EXCEPTION_REPORTER_FILTER](#std:setting-DEFAULT_EXCEPTION_REPORTER_FILTER)
*   [IGNORABLE_404_URLS](#std:setting-IGNORABLE_404_URLS)
*   [经理](#std:setting-MANAGERS)
*   [SILENCED_SYSTEM_CHECKS](#std:setting-SILENCED_SYSTEM_CHECKS)

### 文件上传

*   [DEFAULT_FILE_STORAGE](#std:setting-DEFAULT_FILE_STORAGE)
*   [FILE_CHARSET](#std:setting-FILE_CHARSET)
*   [FILE_UPLOAD_HANDLERS](#std:setting-FILE_UPLOAD_HANDLERS)
*   [FILE_UPLOAD_MAX_MEMORY_SIZE](#std:setting-FILE_UPLOAD_MAX_MEMORY_SIZE)
*   [FILE_UPLOAD_PERMISSIONS](#std:setting-FILE_UPLOAD_PERMISSIONS)
*   [FILE_UPLOAD_TEMP_DIR](#std:setting-FILE_UPLOAD_TEMP_DIR)
*   [MEDIA_ROOT](#std:setting-MEDIA_ROOT)
*   [MEDIA_URL](#std:setting-MEDIA_URL)

### 全球化（i18n / l10n）

*   [日期格式](#std:setting-DATE_FORMAT)
*   [DATE_INPUT_FORMATS](#std:setting-DATE_INPUT_FORMATS)
*   [DATETIME_FORMAT](#std:setting-DATETIME_FORMAT)
*   [DATETIME_INPUT_FORMATS](#std:setting-DATETIME_INPUT_FORMATS)
*   [DECIMAL_SEPARATOR](#std:setting-DECIMAL_SEPARATOR)
*   [FIRST_DAY_OF_WEEK](#std:setting-FIRST_DAY_OF_WEEK)
*   [FORMAT_MODULE_PATH](#std:setting-FORMAT_MODULE_PATH)
*   [LANGUAGE_CODE](#std:setting-LANGUAGE_CODE)
*   [LANGUAGE_COOKIE_AGE](#std:setting-LANGUAGE_COOKIE_AGE)
*   [LANGUAGE_COOKIE_DOMAIN](#std:setting-LANGUAGE_COOKIE_DOMAIN)
*   [LANGUAGE_COOKIE_NAME](#std:setting-LANGUAGE_COOKIE_NAME)
*   [LANGUAGE_COOKIE_PATH](#std:setting-LANGUAGE_COOKIE_PATH)
*   [语言](#std:setting-LANGUAGES)
*   [LOCALE_PATHS](#std:setting-LOCALE_PATHS)
*   [MONTH_DAY_FORMAT](#std:setting-MONTH_DAY_FORMAT)
*   [NUMBER_GROUPING](#std:setting-NUMBER_GROUPING)
*   [SHORT_DATE_FORMAT](#std:setting-SHORT_DATE_FORMAT)
*   [SHORT_DATETIME_FORMAT](#std:setting-SHORT_DATETIME_FORMAT)
*   [THOUSAND_SEPARATOR](#std:setting-THOUSAND_SEPARATOR)
*   [时间格式](#std:setting-TIME_FORMAT)
*   [TIME_INPUT_FORMATS](#std:setting-TIME_INPUT_FORMATS)
*   [时区](#std:setting-TIME_ZONE)
*   [USE_I18N](#std:setting-USE_I18N)
*   [USE_L10N](#std:setting-USE_L10N)
*   [USE_THOUSAND_SEPARATOR](#std:setting-USE_THOUSAND_SEPARATOR)
*   [USE_TZ](#std:setting-USE_TZ)
*   [YEAR_MONTH_FORMAT](#std:setting-YEAR_MONTH_FORMAT)

### HTTP

*   [DEFAULT_CHARSET](#std:setting-DEFAULT_CHARSET)
*   [DEFAULT_CONTENT_TYPE](#std:setting-DEFAULT_CONTENT_TYPE)
*   [DISALLOWED_USER_AGENTS](#std:setting-DISALLOWED_USER_AGENTS)
*   [FORCE_SCRIPT_NAME](#std:setting-FORCE_SCRIPT_NAME)
*   [INTERNAL_IPS](#std:setting-INTERNAL_IPS)
*   [MIDDLEWARE_CLASSES](#std:setting-MIDDLEWARE_CLASSES)
*   Security
    *   [{{s.1260}}](#std:setting-SECURE_BROWSER_XSS_FILTER)
    *   [SECURE_CONTENT_TYPE_NOSNIFF](#std:setting-SECURE_CONTENT_TYPE_NOSNIFF)
    *   [SECURE_HSTS_INCLUDE_SUBDOMAINS](#std:setting-SECURE_HSTS_INCLUDE_SUBDOMAINS)
    *   [SECURE_HSTS_SECONDS](#std:setting-SECURE_HSTS_SECONDS)
    *   [SECURE_PROXY_SSL_HEADER](#std:setting-SECURE_PROXY_SSL_HEADER)
    *   [SECURE_REDIRECT_EXEMPT](#std:setting-SECURE_REDIRECT_EXEMPT)
    *   [SECURE_SSL_HOST](#std:setting-SECURE_SSL_HOST)
    *   [SECURE_ SL_REDIRECT](#std:setting-SECURE_SSL_REDIRECT)
*   [SIGNING_BACKEND](#std:setting-SIGNING_BACKEND)
*   [USE_ETAGS](#std:setting-USE_ETAGS)
*   [USE_X_FORWARDED_HOST](#std:setting-USE_X_FORWARDED_HOST)
*   [WSGI_APPLICATION](#std:setting-WSGI_APPLICATION)

### 记录

*   [登录](#std:setting-LOGGING)
*   [LOGGING_CONFIG](#std:setting-LOGGING_CONFIG)

### 楷模

*   [ABSOLUTE_URL_OVERRIDES](#std:setting-ABSOLUTE_URL_OVERRIDES)
*   [FIXTURE_DIRS](#std:setting-FIXTURE_DIRS)
*   [INSTALLED_APPS](#std:setting-INSTALLED_APPS)

### 安全

*   Cross Site Request Forgery protection
    *   [{{s.1278}}](#std:setting-CSRF_COOKIE_DOMAIN)
    *   [CSRF_COOKIE_NAME](#std:setting-CSRF_COOKIE_NAME)
    *   [CSRF_COOKIE_PATH](#std:setting-CSRF_COOKIE_PATH)
    *   [CSRF_COOKIE_SECURE](#std:setting-CSRF_COOKIE_SECURE)
    *   [CSRF_FAILURE_VIEW](#std:setting-CSRF_FAILURE_VIEW)
*   [SECRET_KEY](#std:setting-SECRET_KEY)
*   [X_FRAME_OPTIONS](#std:setting-X_FRAME_OPTIONS)

### 序列化

*   [DEFAULT_CHARSET](#std:setting-DEFAULT_CHARSET)
*   [SERIALIZATION_MODULES](#std:setting-SERIALIZATION_MODULES)

### 模板

*   [ALLOWED_INCLUDE_ROOTS](#std:setting-ALLOWED_INCLUDE_ROOTS)
*   [模板](#std:setting-TEMPLATES)
*   [TEMPLATE_CONTEXT_PROCESSORS](#std:setting-TEMPLATE_CONTEXT_PROCESSORS)
*   [TEMPLATE_DEBUG](#std:setting-TEMPLATE_DEBUG)
*   [TEMPLATE_DIRS](#std:setting-TEMPLATE_DIRS)
*   [TEMPLATE_LOADERS](#std:setting-TEMPLATE_LOADERS)
*   [TEMPLATE_STRING_IF_INVALID](#std:setting-TEMPLATE_STRING_IF_INVALID)

### 测试

*   数据库：[`TEST`](#std:setting-DATABASE-TEST)
*   [TEST_NON_SERIALIZED_APPS](#std:setting-TEST_NON_SERIALIZED_APPS)
*   [TEST_RUNNER](#std:setting-TEST_RUNNER)

### 网址

*   [APPEND_SLASH](#std:setting-APPEND_SLASH)
*   [PREPEND_WWW](#std:setting-PREPEND_WWW)
*   [ROOT_URLCONF](#std:setting-ROOT_URLCONF)

{% endraw %}