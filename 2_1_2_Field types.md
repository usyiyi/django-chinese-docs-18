<!--
  译者：WrongWay [www.wrongway.me]
  1.8更新：Github@wizardforcel
-->

# Model 字段参考 #

本文档包含所有 字段选项 (field options) 的内部细节和 Django 已经提供的 field types 。

> 参见
> 
> 如果内置的字段不能满足你的应用，你可以很容易地If the built-in fields don’t do the trick, you can easily 编写自定义 model 字段 (write your own custom model fields)。

> 注意
> 
> 从技术上讲， model 是定义在 django.db.models.fields 里面，但为了使用方便，它们被导入到 django.db.models 中；标准上，我们导入 from django.db import models ，然后使用 models.<Foo>Field 的形式使用字段。

## 字段选项 ##

下列参数对所有字段类型都是有效的，同时这些参数也是可选的。

### null ###

**Field.null**

如果为 True ，Django 就会将空值(empty)存储为数据库中的 NULL 。默认值是 False。

要注意空字符串(empty string)总是被存储为空字符串，而不是 NULL。 null=True 只对非字符串字段有意义，比如整数(integer)，布尔值(boolean)，日期(dates)。如果你允许表单提交空值，无论是哪种字段，你还要再设置 blank=True ，这是因为 null 仅仅影响数据库存储 (详见 blank)。

除非你有很充分的理由，否则不要在字符串字段(比如CharField 和 TextField)上使用 null 。在字符串字段上声明 null=True ，就意味着有两种意义的空值：NULL，和空字符串(empty string)。大多数情况下，存在两种空值是多余的。Django 按惯例是使用空字符串，而非 NULL 。

> 注意
> 
> 在使用 Oracle 数据库时，null=True 选项将被强加到有可能是空值的字符串字段，而且会在数据库中保存 NULL ，来表示空字符串。

### blank ###

** Field.blank **

如果为 True，字段允许不填。默认为 False 。

要注意该选项与 null 不同。 null 纯粹是数据库范畴的概念，而 blank 是数据验证范畴的。如果某个字段声明了 blank=True，那么 Django 的管理后台就允许该字段填写空值；否则，如果声明为 blank=False，该字段就是必填的。

### choices ###

**Field.choices**

它是一个可迭代的二元组(比如，列表或是元组)，用来给字段提供选择项。

如果设置了 choices ，Django 的管理后台就会显示选择框，而不是标准的文本框，而且这个选择框的选项就是 choices 中的元组。

这是一个 choices 列表的例子：

```
YEAR_IN_SCHOOL_CHOICES = (
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
    ('GR', 'Graduate'),
)
```

每个元组中的第一个元素，是存储在数据库中的值；第二个元素是该选项更易理解的描述。

choices 列表可以定义为 model 类的一部分：

```
class Foo(models.Model):
    GENDER_CHOICES = (
        ('M', 'Male'),
        ('F', 'Female'),
    )
    gender = models.CharField(max_length=1, choices=GENDER_CHOICES)
```

也可以定义在 model 类之外：

```
GENDER_CHOICES = (
    ('M', 'Male'),
    ('F', 'Female'),
)
class Foo(models.Model):
    gender = models.CharField(max_length=1, choices=GENDER_CHOICES)
```

你也可以将 choices 整理成命名组，这样代码更有条理，结构更清晰：

```
MEDIA_CHOICES = (
    ('Audio', (
            ('vinyl', 'Vinyl'),
            ('cd', 'CD'),
        )
    ),
    ('Video', (
            ('vhs', 'VHS Tape'),
            ('dvd', 'DVD'),
        )
    ),
    ('unknown', 'Unknown'),
)
```

每个元组中的第一个元素会用做组的命名。第二个元素是一个可迭代的二元组，每个二元组都包含一个一个值和一个易于理解的描述。分组选项可以和未分组选项组合在一个单独的列表当中(比如上例中的 unknown 项)。

Djanog 会对设置了 choices 的字段添加一个方法。这个方法根据该字段的当前值获取它易于理解的描述。详见数据库 API 文档中的 get_FOO_display() 。

最后注意一点，choices 可以是任何可迭代的对象，并不仅仅只是列表或是元组。这意味着你可以构造动态的 choices ，不过你最好通过 ForeignKey 使用一张适合的数据表来构造动态 choices。毕竟 choices 适用于变动不大的静态数据。

### db_column ###

**Field.db_column**

该字段在数据库中所使用的列名称。如果没有声明，Django 就会以该字段的名称做为列名。

列名称可以是 SQL 保留字，也可以包含不允许出现在 Python 变量名的特殊字符，比如连字符。这是因为 Django 会在幕后给列名和表名加上引号。

### db_index ###

**Field.db_index**

如果为 True，运行 django-admin.py sqlindexes <sqlindexes> 会为该字段输出一条 CREATE INDEX 语句。

### db_tablespace ###

**Field.db_tablespace**

这部分是在 Django 1.0 新增的： 请查看版本文档
如果该字段是索引字段，db_tablespace 就表示该索引所在的数据库表空间的名称。如果项目配置文件中设定了 DEFAULT_INDEX_TABLESPACE ，那么默认值就是配置项的值；如果你指定了该 model 中 Meta 内嵌类的 db_tablespace ，那么默认值就是 Meta 中 db_tablespace 的值。如果数据库不支持表空间，就会忽略该选项。

## default ##

**Field.default**

该字段的默认值。它可以是一个值，也可以是一个可调用的对象(这里称之为对象C)。若是后者，那么每次创建一个新对象时，对象C都将被调用。

### editable ###

**Field.editable**

如果为 False ，那么在 Django 管理后台中就不会编辑该字段；同样，在 Django 自动生成的表单中也不会编辑该字段。默认值是 True 。

### help_text ###

**Field.help_text**

附加的提示信息。在管理后台编辑该对象的表单中，它显示在该字段下面。即使你的对象无须在后台管理，对于文档化也是很用的。

注意，在管理后台显示该提示信息时，并不会对其转义。所以你可以在 help_text 中包含 HTML 。例如：

```
help_text="Please use the following format: <em>YYYY-MM-DD</em>."
```

你也可以使用纯文本，还可以用 django.utils.html.escape() 转义任何 HTML 字符。

### primary_key ###

**Field.primary_key**

如果为 True，那么该字段就是 model 的主键。

如果你没有指定任何一个字段的 primary_key=True ，Django 就会自动添加一个 IntegerField 做为主键字段。所以除非你想重写默认的主键行为，否则没必要在任何字段上设置 primary_key=True 。详见 自增主键字段 (Automatic primary key fields) 。

primary_key=True 意味着 null=False 和 unique=True 。一个对象只能有一个主键。

### unique ###

**Field.unique**

如果为 True，该字段值就必须是全表唯一的。

它同时作用于数据库层级以及 Django 的管理后台和表单层级。如果你保存 model 时，某个 unique 字段的值是重复的，那么 save() 方法就会抛出 django.db.IntegrityError 异常。

这个选项在所有的字段上都是可用的，除了 ManyToManyField 和 FileField 以外。

### unique_for_date ###

**Field.unique_for_date**

如果要求在某个日期内，该字段值在数据表中是唯一的(就不存在时期和字段值都相同的记录)，那就可以将 unique_for_date 设置为某个 DateField 或 DateTimeField 的名称。

例如，假设你有一个声明了 unique_for_date="pub_date" 的 title 字段，那么 Django 就不会允许出现 title 和 pub_date 相同的两条记录。

它作用于 Django 的管理后台和表单层级，而非数据库层级。

### unique_for_month ###

**Field.unique_for_month**

和 unique_for_date 相似，只不过限定的是月分。

### unique_for_year ###

**Field.unique_for_year**

和 unique_for_date， unique_for_month 相似，只不过限定的是年分。

### verbose_name ###

**Field.verbose_name**

该字段易于理解的名称。如果没有提供自述名，Django 会根据字段的属性名自动创建自述名--就是将属性名称中的空格替换成下划线。详见 字段的自述名(Verbose field names).

## 字段类型 ##

### AutoField ###

**class AutoField(\*\*options)**

它是一个根据 ID 自增长的 IntegerField 字段。通常，你不必直接使用该字段。如果你没在别的字段上指定主键，Django 就会自动添加主键字段。详见 自增主键字段(Automatic primary key fields)。

### BooleanField ###


**class BooleanField(\*\*options)**

一个布尔值(true/false)字段。

Django 的管理后台用checkbox来表现该字段类型。

MySQL 用户请注意：

布尔字段在 MySQL 中被存储为 TINYINT 列。它的值只能是 0 或 1 (其他大多数据库都有适用的 BOOLEAN 类型)。所以仅在使用 MySQL 的情况下，从数据库中检索 BooleanField 并保存为 model 的属性，这时属性值是 1 或是 0 ，而不是 True 或 False 。正常情况下，这不会引起问题，因为 Python 保证令 1 == True 和 0 == False 都是有效的。只是在你编写类似 obj is True 语句时，如果 obj 正是 model 的一个布尔属性值，那你就要留心注意了。因为在使用 mysql 数据库时，"is" 操作是无效的。在这种场合下，使用相等判断更好(使用 "==")。

### CharField ###

**class CharField(max_length=None[, \*\*options])**

它是一个字符串字段，对小字符串和大字符串都适用。

对于更大的文本，应该使用 TextField 。

Django 的管理后台用 `<input type="text">` (单行输入框) 来表示这种字段。

CharField 有一个必须传入的参数：

**CharField.max_length**

字段的最大字符数。它作用于数据库层级和 Django 的数据验证层级。

> 注意
> 
> 如果你正在编写的应用要求满足多种数据库，那么你就要注意不同数据库对 max_length 有不同的限制。详见 数据库 (database backend notes) 。

MySQL 用户请注意：

如果你使用 MySQLdb 1.2.2 和 utf8_bin 字符集(非默认设置)，有几点注意事项要格外留意。详见 MySQL 数据库 (MySQL database notes) 。

### CommaSeparatedIntegerField ###

**class CommaSeparatedIntegerField(max_length=None[, \*\*options])**

它用来存放以逗号间隔的整数序列。和 CharField 一样，必须为它提供 max_length 参数。而且要注意不同数据库对 max_length 的限制。

### DateField ###

**class DateField([auto_now=False, auto_now_add=False, \*\*options])**

该字段利用 Python 的 datetime.date 实例来表示日期。下面是它额外的可选参数：

**DateField.auto_now**

每一次保存对象时，Django 都会自动将该字段的值设置为当前时间。一般用来表示 "最后修改" 时间。要注意使用的是当前日期，而并非默认值，所以你不能通过重写默认值的办法来改变保存时间。

**DateField.auto_now_add**

在第一次创建对象时，Django 自动将该字段的值设置为当前时间，一般用来表示对象创建时间。它使用的同样是当前日期，而非默认值。
Django 管理后台使用一个带有 Javascript 日历的 <input type="text"> 来表示该字段，它带有一个当前日期的快捷选择。那个 JavaScript 日历总是以星期天做为一个星期的第一天。

### DateTimeField ###

**class DateTimeField([auto_now=False, auto_now_add=False, \*\*options])**

该字段利用 datetime.datetime 实例表示日期和时间。该字段所按受的参数和 DateField 一样。

Django 的管理后台使用两个 <input type="text"> 分别表示日期和时间，同样也带有 JavaScript 快捷选项。

### DecimalField ###

这部分是在 Django 1.0 中新增的： 请查看版本文档

**class DecimalField(max_digits=None, decimal_places=None[, \*\*options])**

它是使用 Decimal 实例表示固定精度的十进制数的字段。它有两个必须的参数：

**DecimalField.max_digits**

数字允许的最大位数

**DecimalField.decimal_places**

小数的最大位数
例如，要存储的数字最大值是999，而带有两个小数位，你可以使用：

```
models.DecimalField(..., max_digits=5, decimal_places=2)
```

要存储大约是十亿级且带有10个小数位的数字，就这样写：

```
models.DecimalField(..., max_digits=19, decimal_places=10)
```

Django 管理后台使用 `<input type="text">` (单行输入框) 表示该字段。

### EmailField ###

**class EmailField([max_length=75, \*\*options])**

它是带有 email 合法性检测的A CharField 。

### FileField ###

**class FileField(upload_to=None[, max_length=100, \*\*options])**

文件上传字段

> 注意
> 
> 该字段不支持 primary_key 和 unique 参数，否则会抛出 TypeError 异常。

它有一个必须的参数：

**FileField.upload_to**

用于保存文件的本地文件系统。它根据 MEDIA_ROOT 设置确定该文件的 url 属性。

该路径可以包含 时间格式串 (strftime formatting)，可以在上传文件的时候替换成当时日期／时间(这样，就不会出现在上传文件把某个目录塞满的情况了)。

在 Django 1.0 已改动： 请查看版本文档
该参数也可以是一个可调用项，比如是一个函式，可以调用函式获得包含文件名的上传路径。这个可调用项必须要接受两个参数，并且返回一个保存文件用的 Unix-Style 的路径(用 / 斜杠)。两个参数分别是：

参数：描述
instance：
定义了当前 FileField 的 model 实例。更准确地说，就是以该文件为附件的 model 实例。

大多数情况下，在保存该文件时， model 实例对象还并没有保存到数据库，这是因为它很有可能使用默认的 AutoField，而此时它还没有从数据库中获得主键值。(用法见oteam的http://oteam.cn/2008/10/4/dynamic-upload-paths-in-django/)

filename：上传文件的原始名称。在生成最终路径的时候，有可能会用到它。
还有一个可选的参数：

**FileField.storage**

这部分是在 Django 1.0 中新增的： 请查看版本文档
负责保存和获取文件的对象。详见 Managing files。

Django 管理后台使用 `<input type="file">` (一个文件上传的部件) 来表示这个对象。

在 model 中使用 FileField 或 ImageField (稍后会提到) 要按照以下的步骤：

+ 在项目配置文件中，你要定义 MEDIA_ROOT ，将它的值设为用来存放上传文件的目录的完整路径。(基于性能的考虑，Django 没有将文件保存在数据库中。) ，然后定义 MEDIA_URL ，将它的值设为表示该目录的网址。要确保 web 服务器所用的帐号拥有对该目录的写权限。
+ 在 model 里面添加 FileField 或 ImageField ，并且确认已定义了 upload_to 项，让 Django 知道应该用 MEDIA_ROOT 的哪个子目录来保存文件。
+ 存储在数据库当中的仅仅只是文件的路径(而且是相对于 MEDIA_ROOT 的相对路径)。你可能已经想到利用 Django 提供的 url 这个方便的函式。举个例子，如果你的 ImageField 名称是 mug_shot，那么你可以在模板中使用 {{ object.mug_shot.url }} ，就能得到图片的完整网址。

例如，假设你的 MEDIA_ROOT 被设为 '/home/media'，upload_to 被设为 'photos/%Y/%m/%d'。 upload_to 中的 '%Y/%m/%d' 是一个 时间格式字符串 (strftime formatting)； '%Y' 是四位的年分，'%m' 是两位的月分， '%d' 是两位的日子。如果你在2007年01月15号上传了一个文件，那么这个文件就保存在 /home/media/photos/2007/01/15 目录下。

如果你想得到上传文件的本地文件名称，文件网址，或是文件的大小，你可以使用 name, url 和 size 属性；详见 管理文件 (Managing files)。

注意：在上传文件时，要警惕保存文件的位置和文件的类型，这么做的原因是为了避免安全漏洞。对每一个上传文件都要验证，这样你才能确保上传的文件是你想要的文件。举个例子，如果你盲目地让别人上传文件，而没有对上传文件进行验证，如果保存文件的目录处于 web 服务器的根目录下，万一有人上传了一个 CGI 或是 PHP 脚本，然后通过访问脚本网址来运行上传的脚本，那可就太危险了。千万不要让这样的事情发生！

这部分是在 Django 1.0 中新增的： 在这个版本中，新加了 max_length 参数。
默认情况下，FileField 实例在数据库中的对应列是 varchar(100) ，和其他字段一样，你可以利用 max_length 参数改变字段的最大长度。

### FilePathField ###

**class FilePathField(path=None[, match=None, recursive=False, max_length=100, \*\*options])**

它是一个 CharField ，它用来选择文件系统下某个目录里面的某些文件。它有三个专有的参数，只有第一个参数是必须的：

**FilePathField.path**

这个参数是必需的。它是一个目录的绝对路径，而这个目录就是 FilePathField 用来选择文件的那个目录。比如： "/home/images".
**FilePathField.match**

可选参数。它是一个正则表达式字符串， FilePathField 用它来过滤文件名称，只有符合条件的文件才出现在文件选择列表中。要注意正则表达式只匹配文件名，而不是匹配文件路径。例如： "foo.*\.txt$" 只匹配名为 foo23.txt 而不匹配 bar.txt 和 foo23.gif。
**FilePathField.recursive**

可选参数。它的值是 True 或 False。默认值是 False。它指定是否包含 path 下的子目录。
当然，这三个参数可以同时使用。

前面已经提到了 match 只匹配文件名称，而不是文件路径。所以下面这个例子：

```
FilePathField(path="/home/images", match="foo.*", recursive=True)
```

将匹配 /home/images/foo.gif ，而不匹配 /home/images/foo/bar.gif。这是因为 match 只匹配文件名 (foo.gif 和 bar.gif).

这部分是在 Django 1.0 中新增的： 在这个版本中，新加了 max_length 参数。
默认情况下， FilePathField 实例在数据库中的对应列是s varchar(100) 。和其他字段一样，你可以利用 max_length 参数改变字段的最大升序。

### FloatField ###

**class FloatField([\*\*options])**

在 Django 1.0 在已改动： 请查看版本文档
该字段在 Python 中使用by a float 实例来表示一个浮点数。

Django 管理后台用 `<input type="text">` (一个单行输入框) 表示该字段。

### ImageField ###

**class ImageField(upload_to=None[, height_field=None, width_field=None, max_length=100, \*\*options])**

和 FileField 一样，只是会验证上传对象是不是一个合法的图象文件。它有两个可选参数：

**ImageField.height_field**

保存图片高度的字段名称。在保存对象时，会根据该字段设定的高度，对图片文件进行缩放转换。

**ImageField.width_field**

保存图片宽度的字段名称。在保存对象时，会根据该字段设定的宽度，对图片文件进行缩放转换。
除了那些在 FileField 中有效的参数之外， ImageField 还可以使用 File.height and File.width 两个属性。详见管理文件 (Managing files)(但我个人wrongway并没有找到这两个参数的介绍)。

使用该字段要求安装 Python Imaging Library(PIL).

这部分是在 Django 1.0 中新增的： 在新版本中新加了 max_length 参数。
默认情况下， ImageField 实例对应着数据库中的 created as varchar(100) 列。和其他字段一样，你可以使用 max_length 参数来改变字段的最大长度。

### IntegerField ###

**class IntegerField([\*\*options])**

整数字段。Django 管理后台用 `<input type="text">` (一个单行输入框) 表示该字段。

### IPAddressField ###

**class IPAddressField([\*\*options])**

以字符串形式(比如 192.0.2.30)表示 IP 地址字段。Django 管理后台使用 `<input type="text">` (一个单行输入框) 表示该字段。

### NullBooleanField ###

**class NullBooleanField([\*\*options])**

与 BooleanField 相似，但多了一个 NULL 选项。建议用该字段代替使用 null=True 选项的 BooleanField 。Django 管理后台使用 `<select>` 选择框来表示该字段，选择框有三个选项，分别是 "Unknown", "Yes" 和 "No" 。

### PositiveIntegerField ###

**class PositiveIntegerField([\*\*options])**

和 IntegerField 相似，但字段值必须是非负数。

### PositiveSmallIntegerField ###

**class PositiveSmallIntegerField([\*\*options])**

和 PositiveIntegerField 类似，但数值的取值范围较小，受限于数据库设置。

### SlugField ###

**class SlugField([max_length=50, \*\*options])**

Slug 是一个新闻术语，是指某个事件的短标签。它只能由字母，数字，下划线或连字符组成。通赏情况下，它被用做网址的一部分。

和 CharField 类似，你可以指定 max_length (要注意数据库兼容性和本节提到的 max_length )。如果没有指定 max_length ，Django 会默认字段长度为50。

该字段会自动设置 Field.db_index to True。

基于其他字段的值来自动填充 Slug 字段是很有用的。你可以在 Django 的管理后台中使用 prepopulated_fields 来做到这一点。

### SmallIntegerField ###

**class SmallIntegerField([\*\*options])**

和 IntegerField 类似，但数值的取值范围较小，受限于数据库的限制。

### TextField ###

**class TextField([\*\*options])**

大文本字段。Django 的管理后台使用 `<textarea>` (一个多行文本框) 表示该字段。

MySQL 用户请注意

如果你正在使用 MySQLdb 1.2.1p2 和 utf8_bin 字符集(非默认设置)，有几点注意事项要格外留意，详见 MySQL database notes 。

### TimeField ###

**class TimeField([auto_now=False, auto_now_add=False, \*\*options])**

该字段使用 Python 的 datetime.time 实例来表示时间。它和 DateField 接受同样的自动填充的参数。

Django 管理后台使用一个带 Javascript 快捷链接 的 `<input type="text">` 表示该字段。

### URLField ###

**class URLField([verify_exists=True, max_length=200, \*\*options])**

保存 URL 的 CharField 。它有一个额外的可选参数：

**URLField.verify_exists**

如果为 True (默认值)，Django 在保存对象时会检测该 URL 是否可访问(比如，网址可以正常访问，不返回404错误)。值得注意的是，如果你使用的是一个单线程开发服务器，那么验证网址会挂起当前线程。当然，对于生产用的多线程服务器来说，这就不是一个问题了。
Django 管理后台使用 `<input type="text">` (一个单行输入框) 表示该字段。

和所有 CharField 子类一样，URLField 接受可选的 max_length 参数，该参数默认值是200。

### XMLField ###

**class XMLField(schema_path=None[, \*\*options])**

这是一个根据给定的 schema 验证所输文本是否是合法 XML 的 TextField 字段。它有一个必需的参数：

**schema_path**

用来验证 XML 的 RelaxNG schema 的文件路径。
关联关系字段 (Relationship fields)

Django 也定义了一组用来表示关联关系的字段。

### ForeignKey ###

**class ForeignKey(othermodel[, \*\*options])**

这是一个多对一关系。必须为它提供一个位置参数：被关联的 model 类。

要创建递归关联时--与对象自己做多对一关系，那就使用 models.ForeignKey('self') 。

如果你要与某个尚未定义的 model 建立关联 ，就使用 model 的名称，而不是使用 model 对象本身：

```
class Car(models.Model):
    manufacturer = models.ForeignKey('Manufacturer')
    # ...

class Manufacturer(models.Model):
    # ...
```

这部分是在 Django 1.0 中新建的： 请查看版本文档
要与其他应用中的 model 相关联，你要用完整的应用标签来显式地定义关联。例如，如果上面的 Manufacturer model 定义在另外一个名为 production 的应用中，你只要用：

```
class Car(models.Model):
    manufacturer = models.ForeignKey('production.Manufacturer')
```

在解决两个应用双向依赖时，这种引用方法非常有用。

#### 数据库表现 ####

Django 使用该字段名称＋ "_id" 做为数据库中的列名称。在上面的例子中， Car model 对应的数据表中会有一个 manufacturer_id 列。(你可以通过显式地指定 db_column 来改变该字段的列名称)。不过，除非你想自定义 SQL ，否则没必要更改数据库的列名称。

#### 参数 ####

ForeignKey 接受下列这些可选参数，这些参数定义了关系是如何运行的。

**ForeignKey.limit_choices_to**

它是一个包含筛选条件和对应值的字典 (详见see 制作查询(Making queries))，用来在 Django 管理后台筛选关联对象。例如，利用 Python 的 datetime 模块，过滤掉不符合筛选条件关联对象：

```
limit_choices_to = {'pub_date__lte': datetime.now}
```

只有 pub_date 在当前日期之前的关联对象才允许被选。

也可以使用 Q 对象来代替字典，从而实现更复杂的筛选，详见 复杂查询 (complex queries)。

limit_choices_to 对于在管理后台显示为 inline 的关联对象不起作用。

**ForeignKey.related_name**

反向名称，用来从被关联字段指向关联字段。在 被关联对象文档 (related objects documentation) 中有详细介绍和范例。注意，在你定义 抽象 model (abstract models) 时，你必须显式指定反向名称; 只有在你这么做了之后， 某些特别语法 (some special syntax) 才能正常使用。
**ForeignKey.to_field**

指定当前关系与被关联对象中的哪个字段关联。默认情况下，to_field 指向被关联对象的主键。

### ManyToManyField ###

**class ManyToManyField(othermodel[, \*\*options])**

用来定义多对多关系。必须给它一个位置参数：被关联的 model 类。工作方式和 ForeignKey 一样, 连 递归关联 (recursive) and 延后关联 (lazy) 都一样。

#### 数据库表示  ####

Django 创建一个中间表来表示多对多关系。默认情况下，中间表的名称由两个关系表名结合而成。由于某些数据库对表名的长度有限制，所以中间表的名称会自动限制在64个字符以内，并包含一个不重复的哈希字符串。这意味着，你可能看到类似 author_books_9cdf4 这样的表名称；这是很正常的。你可以使用 db_table 选项手动指定中间表名称。

#### 参数  ####

ManyToManyField 接受下列可选参数，这些参数定义了关系是如何运行的。

ManyToManyField.related_name
和 ForeignKey.related_name 用法一样。
ManyToManyField.limit_choices_to
和 ForeignKey.limit_choices_to 用法一样。

limit_choices_to 对于通过 through 参数指定了中介表的 ManyToManyField 不起作用。

**ManyToManyField.symmetrical**

只要定义递归的多对多关系时起作用。假调有下面这样一个 model :

```
class Person(models.Model):
    friends = models.ManyToManyField("self")
```

Django 处理该 model 时，Django 会发现这个一个关联自己的递归 ManyToManyField ，所以 Django 不会给该字段添加一个指向 Person 类的 person_set 属性，而是把 ManyToManyField 视为对称的--也就是说，如果我是你的朋友，那么你自然也就是我的朋友。

如果不想将递归的多对多关系设为对称的，那你就指定 symmetrical 为 False。这样就会强迫 Django 添加反向名称，从而将该 ManyToManyField 关联改为非对称的。

**ManyToManyField.through**

Django 会自动生成一张表来管理多对多关系。但是，如果你想手动指定中间表，你可以用 through 选项来指定 model 使用另外某个 model 来管理多对多关系。而这个 model 就是中间表所对应的 model 。(我将through所指定的中间表称为中介表)。

当你想使用 多对多关系中的其他数据 (extra data with a many-to-many relationship) 时，一般要用到这个选项。

**ManyToManyField.db_table**

指定数据库中保存多对多关系数据的表名称。如果没有提供该选项，Django 就会根据两个关系表的名称生成一个新的表名，做为中间表的名称。

### OneToOneField ###

**class OneToOneField(othermodel[, parent_link=False, \*\*options])**

用来定义一对一关系。笼统地讲，它与声明了 unique=True 的 ForeignKey 非常相似，不同的是使用反向关联的时候，得到的不是一个对象列表，而是一个单独的对象。

在某个 model 扩展自另一个 model 时，这个字段是非常有用的；例如： 多表继承 (Multi-table inheritance) 就是通过在子 model 中添加一个指向父 model 的一对一关联而实现的。

必须给该字段一个参数：被关联的 model 类。工作方式和 ForeignKey 一样，连 递归关联 (recursive) 和 延后关联 (lazy) 都一样。

此外，OneToOneField 接受 ForeignKey 可接受的参数，只有一个参数是 OnetoOneField 专有的：

**OneToOneField.parent_link**

如果为 True ，并且作用于继承自某个父 model 的子 model 上(这里不能是延后继承，父 model 必须真实存在)，那么该字段就会变成指向父类实例的引用(或者叫链接)，而不是象其他 OneToOneField 那样用于扩展父类并继承父类属性。