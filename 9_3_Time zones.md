

# 时区

## 概览

当时区支持开启时，Django将时间用UTC格式存储到数据库中，在内部使用时区相关的对象，并且在模板(templates)与表单(forms)中将时间转换为终端用户所在时区的时间

当你的用户生活在多个时区，并且你希望根据他们所在的位置显示当地时间时很有用

即便你的网站仅能在一个时区内访问，在数据库中存储UTC 时间依然一种很好的做法。一个主要的原因是夏令时。 许多国家都拥有自己的一套夏令时系统，在这套系统里，春季的时间会提前，而秋季的时间便会后延。如果你只以当前时间为标准来开发，每年都会因为夏令时而引起两次错误（[pytz](http://pytz.sourceforge.net/)文档更详细地讨论了[这些问题](http://pytz.sourceforge.net/#problems-with-localtime)。）这个对于你的博客可能没有什么影响，但是如果涉及到按年，按月，按小时来收费的话，那么就会是一个问题，解决这个问题的方法便是在代码中使用UTC时间，仅在与最终用户进行交互的时候使用本地时间。

Django默认关闭时区支持，如欲开启时区支持，则需在settings中设置[`USE_TZ = True`](../../ref/settings.html#std:setting-USE_TZ) 。强烈推荐安装 [pytz](http://pytz.sourceforge.net/) , 但根据所使用的数据库引擎、操作系统及时区的不同并不强制安装。如果在查询日期或时间时出现异常，请在检查bug之前先尝试安装pytz。可执行下述命令安装pytz：

```
$ pip install pytz

```

注意：

为方便起见，在由[`django-admin startproject`](../../ref/django-admin.html#django-admin-startproject) 创建的`settings.py`文件中已设置 [`USE_TZ = True`](../../ref/settings.html#std:setting-USE_TZ) 。

注意：

另外在settings中，还有一个 [`USE_L10N`](../../ref/settings.html#std:setting-USE_L10N)设置选项，使用它可控制Django是否激活格式本地化。更多细节请参见[_格式本地化_](formatting.html)。

如果你正为某个与时区相关的特殊问题而纠结, 请阅读[_时区FAQ_](#time-zones-faq)。

## 概念

### Naive和aware类型的datetime对象

Python的 [`datetime.datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)") 对象有一个 `tzinfo` 属性，该属性是[`datetime.tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo "(in Python v3.4)")子类的一个实例，它被用来存储时区信息。 当某个datetime对象的tzinfo属性被设置并给出一个时间偏移量时，我们称该datetime对象是**aware（已知）**的。否则称其为 **naive（原生）**的。

可用[`is_aware()`](../../ref/utils.html#django.utils.timezone.is_aware "django.utils.timezone.is_aware") 和 [`is_naive()`](../../ref/utils.html#django.utils.timezone.is_naive "django.utils.timezone.is_naive") 函数来判断某个datetime对象是aware类型或naive类型。

当关闭时区支持时（USE_TZ=False）， Django使用原生的datetime对象保存本地时间。在许多应用中，这是最简单的方式，并且足以满足要求。在这种情况下，可使用下列代码获取当前时间：I

```
import datetime

now = datetime.datetime.now()

```

当开启时区支持时 ([`USE_TZ=True`](../../ref/settings.html#std:setting-USE_TZ)), Django使用已知（aware）的datetime对象存储本地时间。如果在代码中创建了datetime对象, 那么它们也应该是aware类型的datetime对象。在此情况下，上述例子变成：

```
from django.utils import timezone

now = timezone.now()

```

警告：

对aware类型的datetime对象的处理并非总是非常直观。例如：对DST时区来说，标准datetime对象构造器的`tzinfo` 参数并不能可靠地工作。此时，使用UTC一般来说是安全的；如果你的项目使用了其他的时区，那么应该认真阅读[pytz](http://pytz.sourceforge.net/)的文档。

注意：

Python的 [`datetime.time`](https://docs.python.org/3/library/datetime.html#datetime.time "(in Python v3.4)") 对象也具有包含一个 `tzinfo`属性的特点 , 并且在PostgreSQL数据库引擎中，有一个`time with time zone` 类型与该属性相匹配。但是，正如PostgreSQL的文档所指出的那样，该类型 “禁止使用那些可能导致出现问题的属性“。

Django仅支持naive(原生)类型的time对象， 并且，当试图保存一个aware类型的time对象时将会触发一个异常。这是因为，对于时区来说，如果保存的时间没有带有相关日期信息那就没有什么意义。.

###  naive datetime 对象的解释

当 [`USE_TZ`](../../ref/settings.html#std:setting-USE_TZ) 为 `True`, Django 为了保持向后兼容性依旧接受 naive datetime 对象 . 当数据库层收到一个数据库时，它会尝试通过在[_default time zone_](#default-current-time-zone)中解释它来提醒它，并发出警告。

不幸的是，在DST转换期间，一些数据时间不存在或不明确。在这种情况下，[pytz](http://pytz.sourceforge.net/)引发异常。其他[`tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo "(in Python v3.4)")实施（例如未安装[pytz](http://pytz.sourceforge.net/)时用作回退的本地时区）可能会引发异常或返回不准确的结果。这就是为什么当启用时区支持时，您应该始终创建感知datetime对象。

在实践中，这很少是一个问题。Django在模型和表单中给出了清晰的datetime对象，通常，通过[`timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta "(in Python v3.4)")算法从现有对象创建新的datetime对象。在应用程序代码中经常创建的唯一日期时间是当前时间，[`timezone.now()`](../../ref/utils.html#django.utils.timezone.now "django.utils.timezone.now")自动执行正确的操作。

### 默认时区和当前时区

**默认时区**是由[`TIME_ZONE`](../../ref/settings.html#std:setting-TIME_ZONE)设置定义的时区。

**当前时区**是用于呈现的时区。

您应该使用[`activate()`](../../ref/utils.html#django.utils.timezone.activate "django.utils.timezone.activate")将当前时区设置为最终用户的实际时区。否则，将使用默认时区。

注意

如在[`TIME_ZONE`](../../ref/settings.html#std:setting-TIME_ZONE)的文档中所解释的，Django设置环境变量，以使其进程在默认时区运行。无论[`USE_TZ`](../../ref/settings.html#std:setting-USE_TZ)的值和当前时区的值如何，都会发生这种情况。

当[`USE_TZ`](../../ref/settings.html#std:setting-USE_TZ)为`True`时，这对于仍然依赖于本地时间的应用程序保持向后兼容性很有用。但是，[_as explained above_](#naive-datetime-objects)，这不是完全可靠的，并且您应该始终在自己的代码中使用UTC中的感知数据时间。例如，使用[`utcfromtimestamp()`](https://docs.python.org/3/library/datetime.html#datetime.datetime.utcfromtimestamp "(in Python v3.4)")而不是[`fromtimestamp()`](https://docs.python.org/3/library/datetime.html#datetime.datetime.fromtimestamp "(in Python v3.4)") - 不要忘记将`tzinfo`设置为[`utc`](../../ref/utils.html#django.utils.timezone.utc "django.utils.timezone.utc")

### 选择当前时区

当前时区等效于当前[_locale_](index.html#term-locale-name)的翻译。但是，没有等效的Django可以用来自动确定用户的时区的`Accept-Language` HTTP标头。相反，Django提供[_time zone selection functions_](../../ref/utils.html#time-zone-selection-functions)。使用它们来构建对您有意义的时区选择逻辑。

大多数关心时区的网站只是询问用户居住的时区，并将此信息存储在用户的配置文件中。对于匿名用户，他们使用其主要受众群体或UTC的时区。[pytz](http://pytz.sourceforge.net/)提供[助手](http://pytz.sourceforge.net/#helpers)，例如每个国家/地区的时区列表，您可以使用它预先选择最可能的选择。

下面是一个将当前时区存储在会话中的示例。（为了简单起见，它完全跳过错误处理。）

将以下中间件添加到[`MIDDLEWARE_CLASSES`](../../ref/settings.html#std:setting-MIDDLEWARE_CLASSES)：

```
import pytz

from django.utils import timezone

class TimezoneMiddleware(object):
    def process_request(self, request):
        tzname = request.session.get('django_timezone')
        if tzname:
            timezone.activate(pytz.timezone(tzname))
        else:
            timezone.deactivate()

```

创建可以设置当前时区的视图：

```
from django.shortcuts import redirect, render

def set_timezone(request):
    if request.method == 'POST':
        request.session['django_timezone'] = request.POST['timezone']
        return redirect('/')
    else:
        return render(request, 'template.html', {'timezones': pytz.common_timezones})

```

在`template.html`中包含一个表单，它会`POST`到此视图：

```
{% load tz %}
{% get_current_timezone as TIME_ZONE %}
<form action="{% url 'set_timezone' %}" method="POST">
    {% csrf_token %}
    <label for="timezone">Time zone:</label>
    <select name="timezone">
        {% for tz in timezones %}
        <option value="{{ tz }}"{% if tz == TIME_ZONE %} selected="selected"{% endif %}>{{ tz }}</option>
        {% endfor %}
    </select>
    <input type="submit" value="Set" />
</form>

```

## 表单中的时区感知输入

启用时区支持时，Django解释在[_current time zone_](#default-current-time-zone)中以表单形式输入的数据时间，并返回`cleaned_data`中的已知datetime对象。

如果当前时区对于不存在或由于它们落在DST转换（由[pytz](http://pytz.sourceforge.net/)提供的时区）执行此操作而不存在或不明确的数据时间引发异常，则此类数据时间将报告为无效值。

## 模板中的时区感知输出

启用时区支持时，Django在模板中呈现时，将知道的datetime对象转换为[_current time zone_](#default-current-time-zone)。这与[_format localization_](formatting.html)。

警告

Django不转换天真的datetime对象，因为它们可能是模糊的，并且因为您的代码不应该产生幼稚的数据时间，当时区支持启用。但是，您可以使用下面描述的模板过滤器强制转换。

转换为本地时间并不总是适当的 - 您可能正在为计算机而不是为人类生成输出。以下由`tz`模板标记库提供的过滤器和标记允许您控制时区转换。

### 模板标签

#### 当地时间

启用或禁用将已知datetime对象转换为包含块中的当前时区。

对于模板引擎，此标记与[`USE_TZ`](../../ref/settings.html#std:setting-USE_TZ)设置具有完全相同的效果。它允许更细粒度的转换控制。

要激活或取消激活模板块的转换，请使用：

```
{% load tz %}

{% localtime on %}
    {{ value }}
{% endlocaltime %}

{% localtime off %}
    {{ value }}
{% endlocaltime %}

```

注意

在`{％ 本地时间 ％} t3内不遵守[`USE_TZ`](../../ref/settings.html#std:setting-USE_TZ)`

#### 时区

设置或取消所包含块中的当前时区。当前时区未设置时，将应用默认时区。

```
{% load tz %}

{% timezone "Europe/Paris" %}
    Paris time: {{ value }}
{% endtimezone %}

{% timezone None %}
    Server time: {{ value }}
{% endtimezone %}

```

#### get_current_timezone

您可以使用`get_current_timezone`标记获取当前时区的名称：

```
{% get_current_timezone as TIME_ZONE %}

```

如果您启用`django.template.context_processors.tz`上下文处理器，则每个[`RequestContext`](../../ref/templates/api.html#django.template.RequestContext "django.template.RequestContext")将包含一个`TIME_ZONE`变量，值为`get_current_timezone()`。

### 模板过滤器

这些过滤器接受意识和天真的数据时间。出于转换目的，他们假设幼稚的数据时间在默认时区。它们总是返回感知的数据时间。

#### 当地时间

强制将单个值转换为当前时区。

例如：

```
{% load tz %}

{{ value|localtime }}

```

#### 世界标准时间

强制将单个值转换为UTC。

例如：

```
{% load tz %}

{{ value|utc }}

```

#### 时区

强制将单个值转换为任意时区。

参数必须是[`tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo "(in Python v3.4)")子类的实例或时区名称。如果是时区名称，则需要[pytz](http://pytz.sourceforge.net/)。

例如：

```
{% load tz %}

{{ value|timezone:"Europe/Paris" }}

```

## 迁移指南

以下是如何迁移在Django支持的时区之前启动的项目。

### 数据库

#### PostgreSQL

PostgreSQL后端将数据时间存储为`timestamp 与 时间 区域`。实际上，这意味着它将数据时间从连接的时区转换为存储上的UTC，以及从UTC转换为检索时的连接的时区。

因此，如果您使用PostgreSQL，您可以在`USE_TZ = False`和`USE_TZ = True`。数据库连接的时区将分别设置为[`TIME_ZONE`](../../ref/settings.html#std:setting-TIME_ZONE)或`UTC`，以便Django在所有情况下都获得正确的数据时间。您不需要执行任何数据转换。

#### 其他数据库

其他后端存储没有时区信息的数据时间。如果您从`USE_TZ = False`切换到`USE_TZ = True`，您必须将您的数据从本地时间转换为UTC - 如果您的当地时间有DST，这是不确定的。

### 码

第一步是在您的设置文件中添加[`USE_TZ = True`](../../ref/settings.html#std:setting-USE_TZ)，然后安装[pytz t5 &gt;（如果可能）。](http://pytz.sourceforge.net/)在这一点上，事情应该主要工作。如果你在你的代码中创建天真的datetime对象，Django使他们知道在必要时。

但是，这些转换可能会在DST转换周围失败，这意味着您没有得到时区支持的全部好处。此外，你可能会遇到一些问题，因为它是不可能比较一个天真的datetime与意识datetime。由于Django现在给你知道的数据时间，你会得到异常，无论你比较来自一个模型或表单的日期时间与您在代码中创建的天真datetime。

所以第二步是重构你的代码，无论你实例化datetime对象，让他们知道。这可以递增地完成。[`django.utils.timezone`](../../ref/utils.html#module-django.utils.timezone "django.utils.timezone: Timezone support.") defines some handy helpers for compatibility code: [`now()`](../../ref/utils.html#django.utils.timezone.now "django.utils.timezone.now"), [`is_aware()`](../../ref/utils.html#django.utils.timezone.is_aware "django.utils.timezone.is_aware"), [`is_naive()`](../../ref/utils.html#django.utils.timezone.is_naive "django.utils.timezone.is_naive"), [`make_aware()`](../../ref/utils.html#django.utils.timezone.make_aware "django.utils.timezone.make_aware"), and [`make_naive()`](../../ref/utils.html#django.utils.timezone.make_naive "django.utils.timezone.make_naive").

最后，为了帮助您找到需要升级的代码，当您尝试将天真的datetime保存到数据库时，Django会发出警告：

```
RuntimeWarning: DateTimeField ModelName.field_name received a naive
datetime (2012-01-01 00:00:00) while time zone support is active.

```

在开发期间，您可以将此类警告转换为异常，并通过将以下内容添加到设置文件中获取回溯：

```
import warnings
warnings.filterwarnings(
        'error', r"DateTimeField .* received a naive datetime",
        RuntimeWarning, r'django\.db\.models\.fields')

```

### 夹具

当序列化意识datetime时，包括UTC偏移，像这样：

```
"2011-09-01T13:20:30+03:00"

```

对于天真的datetime，它显然不是：

```
"2011-09-01T13:20:30"

```

对于具有[`DateTimeField`](../../ref/models/fields.html#django.db.models.DateTimeField "django.db.models.DateTimeField")的模型，此差异使得无法编写支持和不支持时区支持的灯具。

使用`USE_TZ = False`或在Django 1.4之前生成的灯具使用“naive”格式。如果您的项目包含此类灯具，则在启用时区支持后，在加载时会看到[`RuntimeWarning`](https://docs.python.org/3/library/exceptions.html#RuntimeWarning "(in Python v3.4)")。要摆脱警告，你必须将你的灯具转换为“意识”的格式。

您可以使用[`loaddata`](../../ref/django-admin.html#django-admin-loaddata)，然后[`dumpdata`](../../ref/django-admin.html#django-admin-dumpdata)重新生成灯具。或者，如果它们足够小，您可以简单地编辑它们，以将与您的[`TIME_ZONE`](../../ref/settings.html#std:setting-TIME_ZONE)匹配的UTC偏移量添加到每个序列化的日期时间。

## 常问问题

### 建立

1.  **我不需要多个时区。** 我应该启用时区支持吗？

    是。当启用时区支持时，Django使用更准确的本地时间模型。这将屏蔽您在夏令时（DST）转换周围的微妙和不可再现的错误。

    在这方面，时区与Python中的`unicode`相当。起初很难。您得到编码和解码错误。然后你学习规则。并且一些问题消失 - 当您的应用程序接收到非ASCII输入时，您从不会再次遇到错误输出。

    当您启用时区支持时，您会遇到一些错误，因为您使用天真的数据时间，其中Django期望感知的数据时间。这样的错误显示在运行测试时，它们很容易修复。您将快速了解如何避免无效操作。

    另一方面，由于缺乏时区支持而导致的错误很难预防，诊断和修复。任何涉及计划任务或日期时间算法的事情都是一个微妙的bug，每年只会咬一次或一两次。

    因为这些原因，默认情况下在新项目中启用时区支持，并且您应该保留它，除非您有非常好的理由不要。

2.  **我已启用时区支援。** 我安全吗？

    也许。你更好地保护免受DST相关的错误，但你仍然可以通过不经意地将幼稚的数据时间转换为感知的数据时间来拍摄自己，反之亦然。

    如果您的应用程序连接到其他系统 - 例如，如果它查询Web服务 - 确保正确指定了数据时间。要安全地传输数据时间，它们的表示应该包括UTC偏移，或者它们的值应该是UTC（或两者都是！）。

    最后，我们的日历系统包含计算机的有趣陷阱：

    ```
    &gt;&gt;&gt; import datetime
    &gt;&gt;&gt; def one_year_before(value):       # DON'T DO THAT!
    ...     return value.replace(year=value.year - 1)
    &gt;&gt;&gt; one_year_before(datetime.datetime(2012, 3, 1, 10, 0))
    datetime.datetime(2011, 3, 1, 10, 0)
    &gt;&gt;&gt; one_year_before(datetime.datetime(2012, 2, 29, 10, 0))
    Traceback (most recent call last):
    ...
    ValueError: day is out of range for month

    ```

    （要实现此功能，您必须决定2012-02-29减去一年是2011-02-28还是2011-03-01，具体取决于您的业务需求。）

3.  **我应该安装pytz吗？**

    是。Django有一个不需要外部依赖的策略，因为[pytz](http://pytz.sourceforge.net/)是可选的。但是，安装它更安全。

    一旦激活时区支持，Django需要定义默认时区。当pytz可用时，Django从[tz数据库](http://en.wikipedia.org/wiki/Tz_database)加载此定义。这是最准确的解决方案。否则，它依赖于操作系统报告的本地时间和UTC之间的差异来计算转换。这不太可靠，特别是在DST转换周围。

    此外，如果你想支持用户在多个时区，pytz是时区定义的参考。

### 故障排除

1.  **我的应用程式与**发生当机`TypeError： 无法 比较 offset-naive` `和 偏移感知 数据时间` **- 出了什么问题？**

    让我们通过比较一个naive和一个意识datetime重现这个错误：

    ```
    &gt;&gt;&gt; import datetime
    &gt;&gt;&gt; from django.utils import timezone
    &gt;&gt;&gt; naive = datetime.datetime.utcnow()
    &gt;&gt;&gt; aware = timezone.now()
    &gt;&gt;&gt; naive == aware
    Traceback (most recent call last):
    ...
    TypeError: can't compare offset-naive and offset-aware datetimes

    ```

    如果你遇到这个错误，很可能你的代码是比较这两个东西：

    *   由Django提供的datetime - 例如，从表单或模型字段读取的值。由于您启用时区支持，它意识到。
    *   你的代码生成的日期时间，这是天真的（或者你不会读这个）。

    通常，正确的解决方案是更改您的代码，以使用意识到的datetime。

    如果您编写的可插拔应用程序预期独立于[`USE_TZ`](../../ref/settings.html#std:setting-USE_TZ)的值工作，您可能会发现[`django.utils.timezone.now()`](../../ref/utils.html#django.utils.timezone.now "django.utils.timezone.now")很实用。当`USE_TZ = False`时，此函数将当前日期和时间返回为原始日期时间， `USE_TZ = True`。您可以根据需要添加或减少[`datetime.timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta "(in Python v3.4)")。

2.  **我看到很多**`RuntimeWarning： DateTimeField 收到 a naive datetime ``（YYYY-MM-DD HH：MM：SS）``while time zone support is active` **- 是不是？**

    当启用时区支持时，数据库层希望从代码中仅接收知道的数据时间。当它收到一个天真的datetime时，会发生此警告。这表示您尚未完成移植您的代码以支持时区。有关此过程的提示，请参阅[_migration guide_](#time-zones-migration-guide)。

    在此期间，为了向后兼容，datetime被认为是在默认时区，这通常是你期望的。

3.  `now.date()`**是昨天！** （或明天）

    如果您始终使用天真的数据时间，那么您可能相信可以通过调用日期时间的[`date()`](https://docs.python.org/3/library/datetime.html#datetime.datetime.date "(in Python v3.4)")方法将日期时间转换为日期。您还认为[`date`](https://docs.python.org/3/library/datetime.html#datetime.date "(in Python v3.4)")很像[`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)")，除非它不太准确。

    在时区感知环境中这不是真的：

    ```
    &gt;&gt;&gt; import datetime
    &gt;&gt;&gt; import pytz
    &gt;&gt;&gt; paris_tz = pytz.timezone("Europe/Paris")
    &gt;&gt;&gt; new_york_tz = pytz.timezone("America/New_York")
    &gt;&gt;&gt; paris = paris_tz.localize(datetime.datetime(2012, 3, 3, 1, 30))
    # This is the correct way to convert between time zones with pytz.
    &gt;&gt;&gt; new_york = new_york_tz.normalize(paris.astimezone(new_york_tz))
    &gt;&gt;&gt; paris == new_york, paris.date() == new_york.date()
    (True, False)
    &gt;&gt;&gt; paris - new_york, paris.date() - new_york.date()
    (datetime.timedelta(0), datetime.timedelta(1))
    &gt;&gt;&gt; paris
    datetime.datetime(2012, 3, 3, 1, 30, tzinfo=&lt;DstTzInfo 'Europe/Paris' CET+1:00:00 STD&gt;)
    &gt;&gt;&gt; new_york
    datetime.datetime(2012, 3, 2, 19, 30, tzinfo=&lt;DstTzInfo 'America/New_York' EST-1 day, 19:00:00 STD&gt;)

    ```

    如此示例所示，相同的日期时间具有不同的日期，具体取决于表示时间的时区。但真正的问题是更根本的。

    datetime表示**时间点**。它是绝对的：它不依赖于任何东西。相反，日期是**日历概念**。这是一段时间，其范围取决于考虑日期的时区。可以看到，这两个概念根本不同，将日期时间转换为日期不是确定性操作。

    这在实践中意味着什么？

    一般来说，您应避免将[`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)")转换为[`date`](https://docs.python.org/3/library/datetime.html#datetime.date "(in Python v3.4)")。例如，您可以使用[`date`](../../ref/templates/builtins.html#std:templatefilter-date)模板过滤器仅显示日期时间的日期部分。此过滤器将格式化之前将datetime转换为当前时区，以确保结果正确显示。

    如果您真的需要自己进行转换，那么必须确保datetime首先转换为适当的时区。通常，这将是当前的时区：

    ```
    &gt;&gt;&gt; from django.utils import timezone
    &gt;&gt;&gt; timezone.activate(pytz.timezone("Asia/Singapore"))
    # For this example, we just set the time zone to Singapore, but here's how
    # you would obtain the current time zone in the general case.
    &gt;&gt;&gt; current_tz = timezone.get_current_timezone()
    # Again, this is the correct way to convert between time zones with pytz.
    &gt;&gt;&gt; local = current_tz.normalize(paris.astimezone(current_tz))
    &gt;&gt;&gt; local
    datetime.datetime(2012, 3, 3, 8, 30, tzinfo=&lt;DstTzInfo 'Asia/Singapore' SGT+8:00:00 STD&gt;)
    &gt;&gt;&gt; local.date()
    datetime.date(2012, 3, 3)

    ```

4.  **我收到错误**“`Are time zone definitions for your database and pytz installed?`” **pytz安装，所以我想问题是我的数据库？**

    如果您使用MySQL，请参阅MySQL注释的[_Time zone definitions_](../../ref/databases.html#mysql-time-zone-definitions)部分，了解加载时区定义的说明。

### 用法

1.  **我有一个字符串** `“2012-02-21 10:28:45”` **它位于** `"Europe/Helsinki"` **时区。我如何把它变成一个知道的datetime？**

    这正是[pytz](http://pytz.sourceforge.net/)的用途。

    ```
    &gt;&gt;&gt; from django.utils.dateparse import parse_datetime
    &gt;&gt;&gt; naive = parse_datetime("2012-02-21 10:28:45")
    &gt;&gt;&gt; import pytz
    &gt;&gt;&gt; pytz.timezone("Europe/Helsinki").localize(naive, is_dst=None)
    datetime.datetime(2012, 2, 21, 10, 28, 45, tzinfo=&lt;DstTzInfo 'Europe/Helsinki' EET+2:00:00 STD&gt;)

    ```

    请注意，`localize`是[`tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo "(in Python v3.4)") API的pytz扩展。此外，你可能想要捕捉`pytz.`InvalidTimeError。pytz的文档包含[更多示例](http://pytz.sourceforge.net/#example-usage)。您应该在尝试操作感知的数据时间之前查看它。

2.  **如何获取当前时区的本地时间？**

    嗯，第一个问题是，你真的需要吗？

    当您与人类互动时，您应该只使用本地时间，而且模板图层提供[_filters and tags_](#time-zones-in-templates)将数据时间转换为您选择的时区。

    此外，Python知道如何比较感知的数据时间，在必要时考虑UTC偏移。在UTC中编写所有模型和视图代码要容易得多（也可能更快）。因此，在大多数情况下，由[`django.utils.timezone.now()`](../../ref/utils.html#django.utils.timezone.now "django.utils.timezone.now")返回的UTC中的日期时间就足够了。

    然而，为了完整性，如果你真的想要当前时区的当地时间，你可以如何获得它：

    ```
    &gt;&gt;&gt; from django.utils import timezone
    &gt;&gt;&gt; timezone.localtime(timezone.now())
    datetime.datetime(2012, 3, 3, 20, 10, 53, 873365, tzinfo=&lt;DstTzInfo 'Europe/Paris' CET+1:00:00 STD&gt;)

    ```

    在此示例中，安装[pytz](http://pytz.sourceforge.net/)，当前时区为`"Europe/Paris"`。

3.  **如何查看所有可用的时区？**

    [pytz](http://pytz.sourceforge.net/)提供[帮助](http://pytz.sourceforge.net/#helpers)，包括当前时区列表和所有可用时区列表，其中一些只有历史价值。

