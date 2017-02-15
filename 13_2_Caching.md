

# Django’s cache framework

一个动态网站的基本权衡点就是，它是动态的。 每次用户请求一个页面，Web服务器将进行所有涵盖数据库查询到模版渲染到业务逻辑的请求，用来创建浏览者需要的页面。从开销处理的角度来看，这比你读取一个现成的标准文件的代价要昂贵的多。

对于大多数网络应用程序，这个开销不是很大的问题。我们的应用不是`washingtonpost.com` or `slashdot.org`; 他们只是中小型网站，而且只有那么些流量而已。 但对于中等流量的网站来说，尽可能地减少开销是必要的。

这就是需要缓存的地方

缓存一些东西是为了保存那些需要很多计算资源的结果，这样的话就不必在下次重复消耗计算资源。 下面是一些伪代码，用来解释缓存怎样在动态生成的网页中工作的：

```
given a URL, try finding that page in the cache
if the page is in the cache:
    return the cached page
else:
    generate the page
    save the generated page in the cache (for next time)
    return the generated page

```

Django自带了一个健壮的缓存系统来让你保存动态页面这样避免对于每次请求都重新计算。方便起见，Django提供了不同级别的缓存粒度：你可以缓存特定视图的输出、你可以仅仅缓存那些很难生产出来的部分、或者你可以缓存你的整个网站。

Django也能很好的配合那些“下游”缓存, 比如 [Squid](http://www.squid-cache.org) 和基于浏览器的缓存。这里有一些缓存你不必要直接去控制但是你可以提供线索， (via HTTP headers)关于你的网站哪些部分需要缓存和如何缓存。

也可以看看

[_Cache Framework design philosophy_](../misc/design-philosophies.html#cache-design-philosophy)解释了框架的一些设计决策。

## 设置缓存

缓存系统需要一些设置才能使用。 也就是说，你必须告诉他你要把数据缓存在哪里- 是数据库中，文件系统或者直接在内存中。 这个决定很重要，因为它会影响你的缓存性能，是的，一些缓存类型要比其他的缓存类型更快速。

你的缓存配置是通过setting 文件的[`CACHES`](../ref/settings.html#std:setting-CACHES) 配置来实现的。 这里有[`CACHES`](../ref/settings.html#std:setting-CACHES)所有可配置的变量值。

### Memcached

Django支持的最快，最高效的缓存类型, [Memcached](http://memcached.org/) 是一个全部基于内存的缓存服务，起初是为了解决LiveJournal.com负载来开发的，后来是由Danga开源出来的。 它被类似Facebook 和 维基百科这种网站使用，用来减少数据库访问，显著的提高了网站的性能。

Memcached 是个守护进程，它被分配了单独的内存块。 它做的所有工作就是为缓存提供一个快速的添加，检索，删除的接口。 所有的数据直接存储在内存中，所以它不能取代数据库或者文件系统的使用。

在安装 Memcached 后， 还需要安装 Memcached 依赖模块。Python 有不少Memcache模块最为常用的是[python-memcached](ftp://ftp.tummy.com/pub/python-memcached/) and [pylibmc](http://sendapatch.se/projects/pylibmc/)两个模块.

需要在Django中使用Memcached时:

*   将 [`BACKEND`](../ref/settings.html#std:setting-CACHES-BACKEND) 设置为`django.core.cache.backends.memcached.MemcachedCache` 或者 `django.core.cache.backends.memcached.PyLibMCCache` (取决于你所选绑定memcached的方式)
*   将 [`LOCATION`](../ref/settings.html#std:setting-CACHES-LOCATION) 设置为 `ip:port` 值，`ip` 是 Memcached 守护进程的ip地址， `port` 是Memcached 运行的端口。或者设置为 `unix:path` 值，`path` 是 Memcached Unix socket file的路径.

在这个例子中，Memcached 运行再 本地 (127.0.0.1) 的11211端口，使用 `python-memcached`（也就是需要这么一个python插件） 绑定：

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
    }
}

```

这个例子中，Memcached 通过一个本地的Unix socket file`/tmp/memcached.sock` 来交互，也要使用`python-memcached`绑定：

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': 'unix:/tmp/memcached.sock',
    }
}

```

Memcached有一个非常好的特点就是可以让几个服务的缓存共享。 这就意味着你可以再几个物理机上运行Memcached服务，这些程序将会把这几个机器当做 _同一个_ 缓存，从而不需要复制每个缓存的值在每个机器上。为了使用这个特性，把所有的服务地址放在[`LOCATION`](../ref/settings.html#std:setting-CACHES-LOCATION)里面，用分号隔开或者当做一个list。

这个例子，缓存共享在2个Memcached 实例中，IP地址为172.19.26.240 和 172.19.26.242，端口同为11211：

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': [
            '172.19.26.240:11211',
            '172.19.26.242:11211',
        ]
    }
}

```

下面的这个例子，缓存通过下面几个  Memcached 实例共享，IP地址为172.19.26.240 (端口 11211), 172.19.26.242 (端口 11212), and 172.19.26.244 (端口 11213):

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': [
            '172.19.26.240:11211',
            '172.19.26.242:11212',
            '172.19.26.244:11213',
        ]
    }
}

```

关于Memcached最后要说一点，基于内存的缓存有一个缺点：因为缓存数据是存储在内存中的，所以如果你的服务器宕机数据就会丢失。还要明确， 内存不能替代常驻的数据存储，所以不要把基于内存的缓存当成你唯一的数据存储方式。毫无疑问的，_没有任何的_Django缓存后台应该被用来替代常驻存储--它们要做的是缓存解决方案，而不是存储方案--但是我们在这里指出这一点是因为基于内存的缓存真的是非常的临时。

### 数据库缓存

Django 可以把缓存保存在你的数据库里。如果你有一个快速的、专业的数据库服务器的话那这种方式是效果最好的。

为了把数据表用来当做你的缓存后台：

*   把[`BACKEND`](../ref/settings.html#std:setting-CACHES-BACKEND)设置为`django.core.cache.backends.db.DatabaseCache`
*   把 [`LOCATION`](../ref/settings.html#std:setting-CACHES-LOCATION) 设置为 `tablename`， 数据表的名称。这个名字可以是任何你想要的名字，只要它是一个合法的表名并且在你的数据库中没有被使用过。

在这个示例中，缓存表的名字是 `my_cache_table`:

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
    }
}

```

#### 创建缓存表

使用数据库缓存之前，你必须用这个命令来创建缓存表：

```
python manage.py createcachetable

```

这将在你的数据库中创建一个Django的基于数据库缓存系统预期的特定格式的数据表。表名会从 [`LOCATION`](../ref/settings.html#std:setting-CACHES-LOCATION)中获得。

如果你使用多数据库缓存， [`createcachetable`](../ref/django-admin.html#django-admin-createcachetable)会在每个缓存中创建一个表。

如果你使用多数据库，[`createcachetable`](../ref/django-admin.html#django-admin-createcachetable)会遵循你的数据库路由中的`allow_migrate()`方法(见下)。

像[`migrate`](../ref/django-admin.html#django-admin-migrate), [`createcachetable`](../ref/django-admin.html#django-admin-createcachetable) 这样的命令不会碰触现有的表。它只创建非现有的表。

Changed in Django 1.7:

在Django 1.7之前， [`createcachetable`](../ref/django-admin.html#django-admin-createcachetable)一次只创建一个表。您必须传递要创建的表的名称，如果您使用多个数据库，则必须使用[`--database`](../ref/django-admin.html#django-admin-option---database)选项。为了向后兼容，这仍然是可能的。

#### 多个数据库

如果你在多数据库的情况下使用数据库缓存，你还必须为你的数据库缓存表设置路由说明。出于路由的目的，数据库缓存表会在一个名为 `django_cache`的应用下的`CacheEntry`model中出现。 这个模型不会出现在模型缓存中，但这个模型的细节可以在路由中使用。

例如, 下面的路由会分配所有缓存读操作到 `cache_replica`， 和所有写操作到`cache_primary`. 缓存表只会同步到 `cache_primary`:

```
class CacheRouter(object):
    """A router to control all database cache operations"""

    def db_for_read(self, model, **hints):
        "All cache read operations go to the replica"
        if model._meta.app_label == 'django_cache':
            return 'cache_replica'
        return None

    def db_for_write(self, model, **hints):
        "All cache write operations go to primary"
        if model._meta.app_label == 'django_cache':
            return 'cache_primary'
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        "Only install the cache model on primary"
        if app_label == 'django_cache':
            return db == 'cache_primary'
        return None

```

如果你没有指定路由路径给数据库缓存model,那么缓存就会使用 `默认的` 数据库。

当然, 如果你不使用数据库做缓存，你就不需要担心提供路由结构给数据库缓存模型。

### 文件系统缓存

基于文件的缓存后端序列化和存储每个缓存值作为一个单独的文件。 为了使用这个文件缓存，你要设置[`BACKEND`](../ref/settings.html#std:setting-CACHES-BACKEND)为 `"django.core.cache.backends.filebased.FileBasedCache"` 并且 [`LOCATION`](../ref/settings.html#std:setting-CACHES-LOCATION) 设置为一个合适的目录。例如，把缓存储存在 `/var/tmp/django_cache`,就用这个设置:

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
    }
}

```

如果你在Windows上，将驱动器号放在路径的开头，如下所示：

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': 'c:/foo/bar',
    }
}

```

路径应该是绝对路径– 也就是说，要从你的系统路径开始算。你在末尾添加不添加斜杠都是无所谓的。

请确保，你的路径指向是存在的并且，这个路径下 你有系统用户的足够的读，写权限。继续上面的例子，如果你是一个 名叫`apache`用户,确保 `/var/tmp/django_cache`这个路径存在并且`apache`有读和写的权力。

### 本地内存缓存

这是默认的缓存，如果你不在指定其他的缓存设置。如果你想要具有高速这个有点的基于内存的缓存但是又没有能力带动 Memcached, 那就考虑一下本地缓存吧。这个缓存是per-process（见下文）和线程安全的。要使用它，请将[`BACKEND`](../ref/settings.html#std:setting-CACHES-BACKEND)设置为`"django.core.cache.backends.locmem.LocMemCache"`。例如：

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
    }
}

```

高速缓存[`LOCATION`](../ref/settings.html#std:setting-CACHES-LOCATION)用于标识各个存储器存储。如果您只有一个`locmem`快取，可以省略[`LOCATION`](../ref/settings.html#std:setting-CACHES-LOCATION)；然而，如果您有多个本地内存缓存，您将需要为其中至少一个分配一个名称，以保持它们分离。

注意每个进程都有自己的私有缓存实例，这意味着不可能有跨进程缓存。这显然也意味着本地内存缓存不是特别高效的内存，因此它可能不是生产环境的好选择。这是很好的发展。

### 虚拟缓存（用于开发）

最后, Django有一个 “dummy” 缓存，而且还不是真正的缓存– 它只是提供了一个缓存接口，但是什么也不做。

如果你有一个生产站点使用重型高速缓存在不同的地方，但一个开发/测试环境中，你不想缓存，不希望有你的代码更改为后面的特殊情况，这非常有用。 要激活虚拟缓存，请设置[`BACKEND`](../ref/settings.html#std:setting-CACHES-BACKEND)，如下所示：

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',
    }
}

```

### 使用自定义缓存后端

Django包含一定数量的外部缓存后端的支持，有时你可能想要使用一个自定义的缓存后端。 想要使用外部的缓存，就像python import那样在[`CACHES`](../ref/settings.html#std:setting-CACHES)的 [`BACKEND`](../ref/settings.html#std:setting-CACHES-BACKEND)设置中,就像这样

```
CACHES = {
    'default': {
        'BACKEND': 'path.to.backend',
    }
}

```

如果你建立自己的缓存后端，你可以使用标准的缓存后端作为参考实现。 您可以在Django源的`django/core/cache/backends/`目录中找到代码。

注意：没有真正令人信服的原因，例如不支持它们的主机，您应该坚持Django附带的缓存后端。它们经过了良好的测试，易于使用。

### Cache 参数

上述每一个缓存后台都可以给定一些额外的参数来控制缓存行为，这些参数在可以在[`CACHES`](../ref/settings.html#std:setting-CACHES) setting中以额外键值对的形式给定，可以设置的参数如下：

*   [`TIMEOUT`](../ref/settings.html#std:setting-CACHES-TIMEOUT):缓存的默认过期时间，以秒为单位， 这个参数默认是 `300` seconds (5 分钟).

    New in Django 1.7.

    你可以设置`TIMEOUT` 为 `None` 这样的话，缓存默认永远不会过期。值设置成`0`造成缓存立即失效(缓存就没有意义了)。

*   [`OPTIONS`](../ref/settings.html#std:setting-CACHES-OPTIONS): 这个参数应该被传到缓存后端。有效的可选项列表根据缓存的后端不同而不同，由第三方库所支持的缓存将会把这些选项直接配置到底层的缓存库。

    缓存的后端实现自己的选择策略 (i.e., the `locmem`, `filesystem` and `database` backends) 将会履行下面这些选项：

    *   `MAX_ENTRIES`:高速缓存允许的最大条目数，超出这个数则旧值将被删除. 这个参数默认是`300`.

    *   `CULL_FREQUENCY`:当达到`MAX_ENTRIES` 的时候,被删除的条目比率。 实际比率是 `1 / CULL_FREQUENCY`, 所以设置`CULL_FREQUENCY` 为`2`会在达到`MAX_ENTRIES` 所设置值时删去一半的缓存。这个参数应该是整数，默认为 `3`.

        把 `CULL_FREQUENCY`的值设置为 `0` 意味着当达到`MAX_ENTRIES`时,缓存将被清空。某些缓存后端 (`database`尤其)这将以很多缓存丢失为代价,大大_much_ 提高接受访问的速度。

*   [`KEY_PREFIX`](../ref/settings.html#std:setting-CACHES-KEY_PREFIX)：将自动包含（默认情况下预置为）Django服务器使用的所有缓存键的字符串。

    有关详细信息，请参阅[_cache documentation_](#cache-key-prefixing)。

*   [`VERSION`](../ref/settings.html#std:setting-CACHES-VERSION)：由Django服务器生成的缓存键的默认版本号。

    有关详细信息，请参阅[_cache documentation_](#cache-versioning)。

*   [`KEY_FUNCTION`](../ref/settings.html#std:setting-CACHES-KEY_FUNCTION)包含函数的虚线路径的字符串，定义如何将前缀，版本和键组成最终缓存键。

    有关详细信息，请参阅[_cache documentation_](#cache-key-transformation)。

在下面这个例子中，一个文件系统缓存后端，缓存过期时间被设置为60秒，最大条目为1000.

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
        'TIMEOUT': 60,
        'OPTIONS': {
            'MAX_ENTRIES': 1000
        }
    }
}

```

非法的参数会被忽略掉

## 站点级缓存

一旦高速缓存设置，最简单的方法是使用缓存缓存整个网站。 你需要把`'django.middleware.cache.UpdateCacheMiddleware'` 和 `'django.middleware.cache.FetchFromCacheMiddleware'` 添加到你的 [`MIDDLEWARE_CLASSES`](../ref/settings.html#std:setting-MIDDLEWARE_CLASSES) 设置里, 如例子所示:

```
MIDDLEWARE_CLASSES = (
    'django.middleware.cache.UpdateCacheMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware',
)

```

注意

不，这里并没有排版错误： 'update'中间件，必须放在列表的开始位置，而fectch中间件，必须放在最后。 细节有点费解，如果您想了解完整内幕请参看下面[MIDDLEWARE_CLASSES](#order-of-middleware-classes)顺序

然后，添加下面这些需要的参数到settings文件里:

*   [`CACHE_MIDDLEWARE_ALIAS`](../ref/settings.html#std:setting-CACHE_MIDDLEWARE_ALIAS) – 用于存储的缓存的别名
*   [`CACHE_MIDDLEWARE_SECONDS`](../ref/settings.html#std:setting-CACHE_MIDDLEWARE_SECONDS) –每个page需要被缓存多少秒.
*   [`CACHE_MIDDLEWARE_KEY_PREFIX`](../ref/settings.html#std:setting-CACHE_MIDDLEWARE_KEY_PREFIX) – 如果缓存被使用相同Django安装的多个网站所共享，那么把这个值设成当前网站名，或其他能代表这个Django实例的唯一字符串，以避免key发生冲突。 如果你不在意的话可以设成空字符串。

`FetchFromCacheMiddleware` 缓存GET和HEAD状态为200的回应，用不同的参数请求相同的url被视为独立的页面，缓存是分开的。这个中间件期望HEAD请求用与相应的GET请求相同的响应头来应答；在这种情况下，它可以返回HEAD请求的缓存的GET响应。

另外， `UpdateCacheMiddleware` 在每个[`HttpResponse`](../ref/request-response.html#django.http.HttpResponse "django.http.HttpResponse")里自动设置了一些头部信息

*   设置`Last-Modified` 当一个新(没缓存的)版本的页面被请求时，为当前日期/时间
*   设置 `Expires` 头 为当前日期/时间加上定义的[`CACHE_MIDDLEWARE_SECONDS`](../ref/settings.html#std:setting-CACHE_MIDDLEWARE_SECONDS).
*   设置 `Cache-Control`头部来给页面一个最长的有效期, 来自 [`CACHE_MIDDLEWARE_SECONDS`](../ref/settings.html#std:setting-CACHE_MIDDLEWARE_SECONDS) 的设置.

查看 [_Middleware_](http/middleware.html) 更多中间件.

如果视图设置其自己的缓存到期时间（即，在其`Cache-Control`头部中具有`max-age`部分），则页面将被缓存直到到期时间，而不是[`CACHE_MIDDLEWARE_SECONDS`](../ref/settings.html#std:setting-CACHE_MIDDLEWARE_SECONDS)。使用`django.views.decorators.cache`中的装饰器可以轻松设置视图的到期时间（使用`cache_control`装饰器）或禁用视图的缓存（使用`never_cache`装饰器）。有关这些装饰器的更多信息，请参阅[使用其他标头](#controlling-cache-using-other-headers)部分。

If [`USE_I18N`](../ref/settings.html#std:setting-USE_I18N) is set to `True` then the generated cache key will include the name of the active [_language_](i18n/index.html#term-language-code) – see also [_How Django discovers language preference_](i18n/translation.html#how-django-discovers-language-preference)). 这允许您轻松缓存多语言网站，而无需自己创建缓存密钥。

Cache keys also include the active [_language_](i18n/index.html#term-language-code) when [`USE_L10N`](../ref/settings.html#std:setting-USE_L10N) is set to `True` and the [_current time zone_](i18n/timezones.html#default-current-time-zone) when [`USE_TZ`](../ref/settings.html#std:setting-USE_TZ) is set to `True`.

## 单个view缓存

`django.views.decorators.cache.``cache_page`()

更加轻巧的缓存框架使用方法是对单个有效视图的输出进行缓存。 `django.views.decorators.cache` 定义了一个自动缓存视图响应的 `cache_page`装饰器，使用非常简单:

```
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)
def my_view(request):
    ...

```

`cache_page`接受一个参数：timeout，秒为单位。在前例中，“`my_view()`”视图的结果将被缓存 15 分钟 (注意为了提高可读性我们写了 `60 * 15` . `60 * 15` 等于 `900` –也就是说15分钟等于60秒乘15.)

和站点缓存一样，视图缓存与 URL 无关。如果多个 URL 指向同一视图，每个URL将会分别缓存。  继续 `my_view`范例，如果 URLconf 如下所示：

```
urlpatterns = [
    url(r'^foo/([0-9]{1,2})/$', my_view),
]

```

那么正如你所期待的那样，发送到 `/foo/1/` and `/foo/23/` 会被分别缓存。但是一旦一个明确的 URL (e.g., `/foo/23/`) 已经被请求过了, 之后再度发出的指向该 URL 的请求将使用缓存。

`cache_page` 也可以使用一些额外的参数, `cache`, 指示修饰符去具体使用缓存 (from your [`CACHES`](../ref/settings.html#std:setting-CACHES) setting) 当要缓存页面结果时。默认地, `default` cache 会被使用, 但是你可以特别指定你要用的缓存

```
@cache_page(60 * 15, cache="special_cache")
def my_view(request):
    ...

```

您还可以在每个视图的基础上覆盖缓存前缀。`cache_page`采用可选的关键字参数`key_prefix`，其工作方式与中间件的[`CACHE_MIDDLEWARE_KEY_PREFIX`](../ref/settings.html#std:setting-CACHE_MIDDLEWARE_KEY_PREFIX)设置相同。它可以这样使用：

```
@cache_page(60 * 15, key_prefix="site1")
def my_view(request):
    ...

```

`key_prefix` and `cache` 参数可以被一起指定。将连接`key_prefix`参数和[`CACHES`](../ref/settings.html#std:setting-CACHES)下指定的[`KEY_PREFIX`](../ref/settings.html#std:setting-CACHES-KEY_PREFIX)。

### 在URLconf中指定每个视图的缓存

前一节中的范例将视图硬编码为使用缓存，因为 `cache_page` 在适当的位置对 `my_view`函数进行了转换。 该方法将视图与缓存系统进行了耦合，从几个方面来说并不理想。例如，你可能想在某个无缓存的站点中重用该视图函数，或者不想通过缓存使用页面的人请求你的页面。  解决这些问题的方法是在 URLconf 中指定视图缓存，而不是在这些视图函数上来指定。

这样做很容易：在URLconf中引用它时，只需用`cache_page`包装视图函数。这是以前的URLconf：

```
urlpatterns = [
    url(r'^foo/([0-9]{1,2})/$', my_view),
]

```

这也是一样的，`my_view`包裹在`cache_page`：

```
from django.views.decorators.cache import cache_page

urlpatterns = [
    url(r'^foo/([0-9]{1,2})/$', cache_page(60 * 15)(my_view)),
]

```

## 模板片段缓存

如果想对缓存进行更多的控制，可以使用 `cache`模板标签来缓存模板的一个片段。 要让模板处理这个标签，把`{% load cache %}` 放在缓存片段的上面。

标签`{% cache %}`将按给定的时间缓存包含块中的内容。它最少需要两个参数：缓存时间（以秒为单位）；给缓存片段起的名称。 该名称将被视为是，不使用变量。例如：

```
{% load cache %}
{% cache 500 sidebar %}
    .. sidebar ..
{% endcache %}

```

有时，你可以依据这个片段内的动态内容缓存多个版本。如上个例子中，可以给站点的每个用户生成不同版本的sidebar缓存。只需要给 `{% cache %}`标签再传递一个参数来标识区分这个缓存片段。

```
{% load cache %}
{% cache 500 sidebar request.user.username %}
    .. sidebar for logged in user ..
{% endcache %}

```

指定一个以上的参数来识别片段是非常好的。 简单的尽可能的传递你需要的参数到 `{% cache %}` 。

如果[`USE_I18N`](../ref/settings.html#std:setting-USE_I18N)设置为`True`，则每个网站中间件缓存将[_respect the active language_](#i18n-cache-key)。对于`cache`模板标记，您可以使用模板中提供的[_translation-specific variables_](i18n/translation.html#template-translation-vars)之一来实现相同的结果：

```
{% load i18n %}
{% load cache %}

{% get_current_language as LANGUAGE_CODE %}

{% cache 600 welcome LANGUAGE_CODE %}
    {% trans "Welcome to example.com" %}
{% endcache %}

```

缓存超时可以是模板变量，只要模板变量解析为整数值即可。例如，如果模板变量`my_timeout`设置为值`600`，则以下两个示例是等效的：

```
{% cache 600 sidebar %} ... {% endcache %}
{% cache my_timeout sidebar %} ... {% endcache %}

```

此功能有助于避免模板中的重复。您可以在一个位置设置变量的超时，只需重复使用该值。

New in Django 1.7:

默认情况下，缓存标签将尝试使用名为“template_fragments”的缓存。如果不存在这样的缓存，则它将回退到使用默认缓存。您可以选择要与`using`关键字参数一起使用的备用高速缓存后端，该参数必须是标记的最后一个参数。

```
{% cache 300 local-thing ...  using="localcache" %}

```

指定未配置的缓存名称被视为错误。

`django.core.cache.utils.``make_template_fragment_key`(_fragment_name_, _vary_on=None_)

如果您想获得用于缓存片段缓存键，就可以使用 `make_template_fragment_key`. `fragment_name`就跟 `cache` template tag的第二参数是一样的; `vary_on` 是一个需要传递额外参数的列表。该功能可以用于无效或覆盖缓存项，例如：

```
>>> from django.core.cache import cache
>>> from django.core.cache.utils import make_template_fragment_key
# cache key for {% cache 500 sidebar username %}
>>> key = make_template_fragment_key('sidebar', [username])
>>> cache.delete(key) # invalidates cached template fragment

```

## 底层的缓存API

有时，缓存整个页面不会让你获益很多，事实上，过度的矫正原来的不方便，变得更加不方便。

也许，例如，您的网站包含的结果取决于几个昂贵的查询，其结果在不同的时间间隔都有所不同。 在这种情况下，使用每个站点或每个视图缓存策略提供的全页缓存是不理想的，因为你不想缓存整个结果（因为一些数据经常变化），但你仍然想要缓存很少变化的结果。

对于这个情况 Django提供了一个底层的 cache API. 你可以用这个 API来储存在缓存中的对象，并且控制粒度随你喜欢。您可以缓存可以安全pickle的任何Python对象：模型对象的字符串，字典，列表等等。 （最常见的Python对象可以Pickle;参考Python文档有关pickle更多信息。）

### 访问缓存

`django.core.cache.``caches`

New in Django 1.7.

你可以通过 类字典对象`django.core.cache.caches`.访问配置在[`CACHES`](../ref/settings.html#std:setting-CACHES) 设置中的字典类对象。对同一线程相同的别名重复请求将返回相同的对象。

```
&gt;&gt;&gt; from django.core.cache import caches
&gt;&gt;&gt; cache1 = caches['myalias']
&gt;&gt;&gt; cache2 = caches['myalias']
&gt;&gt;&gt; cache1 is cache2
True

```

如果key不存在，就会引发一个 `InvalidCacheBackendError` 。

为了提供线程安全， 不同的缓存实例将会返回给每一个线程。

`django.core.cache.``cache`

作为一个快捷方式，默认缓存可用为`django.core.cache.cache`：

```
&gt;&gt;&gt; from django.core.cache import cache

```

此对象等效于`caches['default']`。

`django.core.cache.``get_cache`(_backend_, _**kwargs_)

自1.7版起已弃用：此功能已弃用，支持[`caches`](#django.core.cache.caches "django.core.cache.caches")。

在Django 1.7之前，这个函数是获取缓存实例的规范方式。它也可以用于创建具有不同配置的新缓存实例。

```
&gt;&gt;&gt; from django.core.cache import get_cache
&gt;&gt;&gt; get_cache('default')
&gt;&gt;&gt; get_cache('django.core.cache.backends.memcached.MemcachedCache', LOCATION='127.0.0.2')
&gt;&gt;&gt; get_cache('default', TIMEOUT=300)

```

### 基本用法

The basic interface is `set(key, value, timeout)` and `get(key)`:

```
>>> cache.set('my_key', 'hello, world!', 30)
>>> cache.get('my_key')
'hello, world!'

```

`timeout`参数是可选的，默认为[`CACHES`](../ref/settings.html#std:setting-CACHES)设置中适当后端的`timeout`参数（如上所述）。它是值应该存储在缓存中的秒数。Passing in `None` for `timeout` will cache the value forever. `0`的`timeout`将不会缓存该值。

如果对象不存在于缓存中，则`cache.get()`返回`None`：

```
# Wait 30 seconds for 'my_key' to expire...

>>> cache.get('my_key')
None

```

我们建议不要在缓存中存储文本值`None`，因为您将无法区分您存储的`None`值和由返回值表示的缓存未命中`None`。

`cache.get()`可以采用`default`参数。如果对象不存在于缓存中，则指定返回哪个值：

```
>>> cache.get('my_key', 'has expired')
'has expired'

```

要添加键（如果它尚不存在），请使用`add()`方法。它使用与`set()`相同的参数，但如果指定的键已经存在，它不会尝试更新缓存：

```
>>> cache.set('add_key', 'Initial value')
>>> cache.add('add_key', 'New value')
>>> cache.get('add_key')
'Initial value'

```

如果你需要知道`add()`是否在缓存中存储了一个值，你可以检查返回值。如果值存储，则返回`True`，否则返回`False`。

还有一个`get_many()`接口，只会命中一次缓存。`get_many()`返回一个字典，其中包含您请求的所有实际存在于缓存中的键（并且未过期）：

```
>>> cache.set('a', 1)
>>> cache.set('b', 2)
>>> cache.set('c', 3)
>>> cache.get_many(['a', 'b', 'c'])
{'a': 1, 'b': 2, 'c': 3}

```

要更有效地设置多个值，请使用`set_many()`传递键值对的字典：

```
>>> cache.set_many({'a': 1, 'b': 2, 'c': 3})
>>> cache.get_many(['a', 'b', 'c'])
{'a': 1, 'b': 2, 'c': 3}

```

像`cache.set()`，`set_many()`采用可选的`timeout`参数。

您可以使用`delete()`显式删除键。这是清除特定对象的缓存的简单方法：

```
>>> cache.delete('a')

```

如果您想一次清除一堆键，`delete_many()`可以取得要清除的键列表：

```
>>> cache.delete_many(['a', 'b', 'c'])

```

最后，如果要删除缓存中的所有键，请使用`cache.clear()`。小心这个；`clear()`将从缓存中删除_所有_，而不仅仅是应用程序设置的键。

```
>>> cache.clear()

```

您还可以使用`incr()`或`decr()`方法分别递增或递减已存在的键。默认情况下，现有高速缓存值将递增或递减1。可以通过向增量/减量调用提供参数来指定其他增量/减量值。如果您尝试增加或减少不存在的缓存键，则会引发ValueError错误：

```
>>> cache.set('num', 1)
>>> cache.incr('num')
2
>>> cache.incr('num', 10)
12
>>> cache.decr('num')
11
>>> cache.decr('num', 5)
6

```

注意

`incr()` / `decr()`方法不能保证是原子的。在那些支持原子递增/递减（最明显的是memcached后端）的后端，递增和递减操作将是原子的。然而，如果后端本身不提供增量/减量操作，则它将使用两步检索/更新来实现。

如果由缓存后端实现，您可以使用`close()`关闭与缓存的连接。

```
>>> cache.close()

```

注意

对于不实现`close`方法的缓存，它是一个无操作。

### 高速缓存密钥前缀

如果您在服务器之间共享缓存实例，或者在您的生产和开发环境之间共享缓存的话，可以使用另一台服务器使用一台服务器缓存的数据。 如果缓存数据的格式是不同的服务器，这可能会导致一些很难诊断问题。

为了防止这种情况，Django提供了由服务器使用的所有前缀缓存键的能力 当保存或检索特定的缓存键时，Django将自动为缓存键加上[`KEY_PREFIX`](../ref/settings.html#std:setting-CACHES-KEY_PREFIX)缓存设置的值。

通过确保每个Django实例具有不同的[`KEY_PREFIX`](../ref/settings.html#std:setting-CACHES-KEY_PREFIX)，您可以确保缓存值中没有冲突。

### 缓存版本控制

当您更改使用缓存值的运行代码时，您可能需要清除任何现有的缓存值。最简单的方法是刷新整个缓存，但是这可能导致缓存值丢失，仍然有效和有用。

Django提供了一种更好的方法来定位各个缓存值。Django的缓存框架具有系统级版本标识符，使用[`VERSION`](../ref/settings.html#std:setting-CACHES-VERSION)缓存设置指定。此设置的值将自动与高速缓存前缀和用户提供的高速缓存密钥相结合，以获取最终的高速缓存密钥。

默认情况下，任何密钥请求都将自动包含站点默认缓存密钥版本。但是，原语缓存函数都包含`version`参数，因此可以指定要设置或获取的特定缓存键版本。例如：

```
# Set version 2 of a cache key
>>> cache.set('my_key', 'hello world!', version=2)
# Get the default version (assuming version=1)
>>> cache.get('my_key')
None
# Get version 2 of the same key
>>> cache.get('my_key', version=2)
'hello world!'

```

特定键的版本可以使用`incr_version()`和`decr_version()`方法递增和递减。这使得特定键可以被碰撞到新版本，而其他键不受影响。继续我们前面的例子：

```
# Increment the version of 'my_key'
>>> cache.incr_version('my_key')
# The default version still isn't available
>>> cache.get('my_key')
None
# Version 2 isn't available, either
>>> cache.get('my_key', version=2)
None
# But version 3 *is* available
>>> cache.get('my_key', version=3)
'hello world!'

```

### 缓存键转换

如前两节所述，用户提供的缓存键不是逐字使用的 - 它与缓存前缀和键版本结合以提供最终缓存键。默认情况下，使用冒号连接这三个部分以生成最终字符串：

```
def make_key(key, key_prefix, version):
    return ':'.join([key_prefix, str(version), key])

```

如果要以不同的方式组合这些部分，或对最终密钥应用其他处理（例如，获取关键部分的哈希摘要），则可以提供自定义键功能。

[`KEY_FUNCTION`](../ref/settings.html#std:setting-CACHES-KEY_FUNCTION)缓存设置指定与上述`make_key()`的原型匹配的函数的虚线路径。如果提供，将使用此自定义键功能，而不是默认键组合功能。

### 缓存关键警告

Memcached是最常用的生产缓存后端，它不允许超过250个字符或包含空格或控制字符的缓存键，并且使用这样的键将导致异常。为了鼓励缓存可移植代码并最小化令人不快的惊喜，其他内置缓存后端会发出警告（`django.core.cache.backends.base.CacheKeyWarning`），如果使用一个键， memcached上的错误。

If you are using a production backend that can accept a wider range of keys (a custom backend, or one of the non-memcached built-in backends), and want to use this wider range without warnings, you can silence `CacheKeyWarning` with this code in the `management` module of one of your [`INSTALLED_APPS`](../ref/settings.html#std:setting-INSTALLED_APPS):

```
import warnings

from django.core.cache import CacheKeyWarning

warnings.simplefilter("ignore", CacheKeyWarning)

```

如果您要为其中一个内置后端提供自定义键验证逻辑，则可以对其进行子类化，仅覆盖`validate_key`方法，并按照[的说明使用自定义缓存后端](#using-a-custom-cache-backend)。例如，要为`locmem`后端执行此操作，请将此代码放在模块中：

```
from django.core.cache.backends.locmem import LocMemCache

class CustomLocMemCache(LocMemCache):
    def validate_key(self, key):
        """Custom validation, raising exceptions or warnings as needed."""
        # ...

```

...并在[`CACHES`](../ref/settings.html#std:setting-CACHES)设置的[`BACKEND`](../ref/settings.html#std:setting-CACHES-BACKEND)部分使用此类别的Python路径。

## 下游缓存

到目前为止，本文档专注于缓存您的_自己的_数据。但是另一种类型的缓存也与Web开发相关：由“下游”缓存执行的缓存。这些是在请求到达您的网站之前为用户缓存页面的系统。

下面是一些下游缓存的例子：

*   您的ISP可能会缓存某些网页，因此如果您从[http://example.com/](http://example.com/)请求了某个网页，您的ISP会向您发送该网页，而无需直接访问example.com。example.com的维护者不知道这个缓存；该ISP位于example.com和您的Web浏览器之间，透明地处理所有的缓存。
*   您的Django网站可能位于_代理缓存_后面，例如Squid Web代理缓存（[http://www.squid-cache.org/](http://www.squid-cache.org/)），缓存页面性能。在这种情况下，每个请求将首先由代理处理，并且只有在需要时才会传递给您的应用程序。
*   您的Web浏览器也缓存页面。如果网页发出相应的标头，您的浏览器将使用本地缓存副本来处理该页面的后续请求，甚至无需再次联系网页以查看其是否已更改。

下游缓存是一个很好的效率提升，但是有一个危险：许多网页的内容基于身份验证和大量其他变量，缓存系统盲目保存纯粹基于URL的网页可能会暴露不正确或敏感数据到后续这些页面的访问者。

例如，假设您操作Web电子邮件系统，并且“收件箱”页面的内容显然取决于哪个用户登录。如果ISP盲目缓存您的网站，则通过该ISP登录的第一个用户将为其后的访问者缓存其用户特定的收件箱页面。这不酷。

幸运的是，HTTP提供了这个问题的解决方案。存在多个HTTP报头以指示下游高速缓存根据指定的变量来使其高速缓存内容不同，并且指示高速缓存机制不缓存特定页面。我们将在接下来的部分中看一些这些标题。

## 使用Vary头

`Vary`头定义了缓存机制在构建其缓存键时应考虑哪些请求头。例如，如果网页的内容取决于用户的语言偏好，则该页被称为“根据语言而变化”。

默认情况下，Django的缓存系统使用请求的完全限定网址（例如，`"http://www.example.com/stories/2005/?order_by=author"`）创建其缓存密钥。这意味着对该网址的每个请求都将使用相同的缓存版本，而不考虑用户代理的差异（例如Cookie或语言首选项）。但是，如果此网页根据请求标头（例如Cookie，语言或用户代理）的某些差异产生不同的内容，则需要使用`Vary`标头来指示缓存页面输出依赖于这些东西的机制。

Changed in Django 1.7:

缓存键使用请求的完全限定URL，而不仅仅是路径和查询字符串。

要在Django中执行此操作，请使用方便的[`django.views.decorators.vary.vary_on_headers()`](http/decorators.html#django.views.decorators.vary.vary_on_headers "django.views.decorators.vary.vary_on_headers")视图装饰器，如下所示：

```
from django.views.decorators.vary import vary_on_headers

@vary_on_headers('User-Agent')
def my_view(request):
    # ...

```

在这种情况下，缓存机制（例如Django自己的缓存中间件）将缓存每个唯一用户代理的页面的单独版本。

使用`vary_on_headers`装饰器而不是手动设置`Vary`标头的优点（使用类似`response ['Vary'] = 'user-agent'`）是装饰器_将_添加到`Vary`

您可以将多个标头传送至`vary_on_headers()`：

```
@vary_on_headers('User-Agent', 'Cookie')
def my_view(request):
    # ...

```

这告诉下游缓存在_两者上变化，这意味着用户代理和cookie的每个组合将获得自己的缓存值。_For example, a request with the user-agent `Mozilla` and the cookie value `foo=bar` will be considered different from a request with the user-agent `Mozilla` and the cookie value `foo=ham`.

因为cookie的变化很常见，所以有一个[`django.views.decorators.vary.vary_on_cookie()`](http/decorators.html#django.views.decorators.vary.vary_on_cookie "django.views.decorators.vary.vary_on_cookie")装饰器。这两个视图是等效的：

```
@vary_on_cookie
def my_view(request):
    # ...

@vary_on_headers('Cookie')
def my_view(request):
    # ...

```

您传递给`vary_on_headers`的标头不区分大小写； `"User-Agent"`与`"user-agent"`相同。

您也可以直接使用辅助函数[`django.utils.cache.patch_vary_headers()`](../ref/utils.html#django.utils.cache.patch_vary_headers "django.utils.cache.patch_vary_headers")。此函数设置或添加到`Vary 标题`。例如：

```
from django.utils.cache import patch_vary_headers

def my_view(request):
    # ...
    response = render_to_response('template_name', context)
    patch_vary_headers(response, ['Cookie'])
    return response

```

`patch_vary_headers`采用[`HttpResponse`](../ref/request-response.html#django.http.HttpResponse "django.http.HttpResponse")实例作为其第一个参数，以及不区分大小写的标题名称的列表/元组作为其第二个参数。

有关Vary头的更多信息，请参阅[官方Vary spec](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.44)。

## 控制缓存：使用其他头

缓存的其他问题是数据的隐私和数据应该存储在级联的缓存中的位置的问题。

用户通常面临两种缓存：它们自己的浏览器缓存（私有缓存）和它们的提供者的缓存（公共缓存）。公共缓存由多个用户使用并由其他人控制。这会导致敏感数据出现问题，例如您的银行帐号存储在公共缓存中。因此，Web应用程序需要一种方法来告诉缓存哪些数据是私有的，哪些是公共的。

解决方案是指示页面的缓存应为“私有”。要在Django中执行此操作，请使用`cache_control`视图装饰器。例：

```
from django.views.decorators.cache import cache_control

@cache_control(private=True)
def my_view(request):
    # ...

```

这个装饰器负责在幕后发送适当的HTTP标头。

注意，缓存控制设置“private”和“public”是互斥的。装饰器确保“public”指令被删除，如果“private”应该被设置（反之亦然）。两个指令的示例使用将是提供私人和公共条目的博客站点。公共条目可以在任何共享缓存上缓存。以下代码使用[`django.utils.cache.patch_cache_control()`](../ref/utils.html#django.utils.cache.patch_cache_control "django.utils.cache.patch_cache_control")，手动方式修改缓存控件头（它由`cache_control`装饰器内部调用）：

```
from django.views.decorators.cache import patch_cache_control
from django.views.decorators.vary import vary_on_cookie

@vary_on_cookie
def list_blog_entries_view(request):
    if request.user.is_anonymous():
        response = render_only_public_entries()
        patch_cache_control(response, public=True)
    else:
        response = render_private_and_public_entries(request.user)
        patch_cache_control(response, private=True)

    return response

```

还有一些其他方法来控制缓存参数。例如，HTTP允许应用程序执行以下操作：

*   定义页面应缓存的最大时间。
*   指定缓存是否应始终检查较新版本，仅在没有更改时传递缓存的内容。（即使服务器页面更改，一些缓存可能会提供缓存内容，因为缓存副本尚未到期。）

在Django中，使用`cache_control`视图装饰器来指定这些缓存参数。在此示例中，`cache_control`告诉缓存在每次访问时重新验证缓存，并将缓存的版本最多保存3,600秒：

```
from django.views.decorators.cache import cache_control

@cache_control(must_revalidate=True, max_age=3600)
def my_view(request):
    # ...

```

任何有效的`Cache-Control` HTTP指令在`cache_control()`中有效。这里有一个完整的列表：

*   `public=True`
*   `private=True`
*   `no_cache=True`
*   `no_transform=True`
*   `must_revalidate=True`
*   `proxy_revalidate=True`
*   `max_age=num_seconds`
*   `s_maxage=num_seconds`

有关缓存控制HTTP指令的说明，请参见[缓存控制规范](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)。

（请注意，缓存中间件已经将缓存头的max-age设置为[`CACHE_MIDDLEWARE_SECONDS`](../ref/settings.html#std:setting-CACHE_MIDDLEWARE_SECONDS)设置的值。如果您在`cache_control`装饰器中使用自定义`max_age`，装饰器将优先，并且标头值将正确合并。）

If you want to use headers to disable caching altogether, `django.views.decorators.cache.never_cache` is a view decorator that adds headers to ensure the response won’t be cached by browsers or other caches. 例：

```
from django.views.decorators.cache import never_cache

@never_cache
def myview(request):
    # ...

```

## MIDDLEWARE_CLASSES的顺序

如果您使用缓存中间件，请务必将每一半放在[`MIDDLEWARE_CLASSES`](../ref/settings.html#std:setting-MIDDLEWARE_CLASSES)设置中的正确位置。这是因为缓存中间件需要知道通过哪个头来改变缓存存储。中间件总是向`Vary`响应标头添加一些东西。

`UpdateCacheMiddleware`在响应阶段运行，其中中间件按相反顺序运行，因此列表顶部的项目在响应阶段运行_最后_。因此，您需要确保`UpdateCacheMiddleware`在_之前_出现可能会为`Vary`标题添加内容的任何其他中间件。以下中间件模块：

*   `SessionMiddleware`添加`Cookie`
*   `GZipMiddleware`添加`Accept-Encoding`
*   `LocaleMiddleware`添加`Accept-Language`

`FetchFromCacheMiddleware`, on the other hand, runs during the request phase, where middleware is applied first-to-last, so an item at the top of the list runs _first_ during the request phase. `FetchFromCacheMiddleware`也需要在其他中间件更新`Vary`标题后运行，因此`FetchFromCacheMiddleware`必须在_之后_这样做。

