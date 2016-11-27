

# 数据库

Django试图尽可能多的支持所有数据库后端的特性。然而，并不是所有数据库都一样，所以我们必须在支持哪些特性和做出哪些安全的假定上做出设计决策。

本文描述了一些Django使用数据库的有关特性。当然，它并不想成为各服务器指定的文档或者参考手册的替代品。



## 综合说明



### 持续连接特性

持续连接的特性避免了每一次重新建立与数据库的连接的请求中所增加的压力。这些连接通过 [`CONN_MAX_AGE`](settings.html#std:setting-CONN_MAX_AGE) 参数(控制一个连接的最长存活时间)来控制。它可以被单独的设置在每一个数据库中。

参数的默认值为 `0` ，对每次请求结束时终止数据库连接的历史行为提供保护。当启用持续连接时，设置 [`CONN_MAX_AGE`](settings.html#std:setting-CONN_MAX_AGE) 的值(正数，单位为秒)为每个连接的最大存活时间。对于无限制的连接，请设置为 `None`。



#### 连接管理

Django在它第一次建立数据库查询的时候打开与数据库的连接。它保持着连接的通畅使得在随后的请求中可以再利用。一旦这个连接超过参数 [`CONN_MAX_AGE`](settings.html#std:setting-CONN_MAX_AGE) 的最长存活时间的限制或者不能更长久的存活时，Django就会终止连接。

详细地说，当Django需要的时或者当前不存在数据库连接时，它会自动的与数据库之间建立连接————要么是因为这是第一次连接，要么是因为先前连接被关闭。

在每个请求开始时后，如果连接到了设置的最长存活时间时，Django就会关闭它。如果你的数据库在之后一段时间终止了闲置的连接，你应该设置[`CONN_MAX_AGE`](settings.html#std:setting-CONN_MAX_AGE) 为一个更低的值，以至于Django不会尝试去连接数据库服务器上已经终止的连接。（这个问题可能仅仅影响流量比较低的站点。）

在每个请求结束，如果它已经到达了最长存活时间或者它处在一个不可恢复的错误状态时，Django就会关闭连接。在处理请求中如果有任何数据库错误产生，Django就会检查是否连接仍然工作，不工作就关闭它。因此，数据库错误最多影响一个请求。如果连接不可用，下一个 请求会打开一个新的连接。





#### 注意事项

因为每个线程都保持着自己的连接，你的数据库至少必须在你的工作线程中支持尽可能多的并发线程。

有的时候数据库并不会在你的视图中有太多的访问，例如，因为数据库是一个外部系统，或者因为缓存的作用。在这种情况下，你应该设置[`CONN_MAX_AGE`](settings.html#std:setting-CONN_MAX_AGE)为一个低的值或者为`0`，因为这对保持一个连接并没有意义，而且连接不太可能被重复调用。这会有助于保持数据库的并发性连接数为一个小的值。

开发服务器对每个需要处理的请求创建一个新的线程，用来否定持续连接的影响。不要在开发过程中启用他们。

当Django建立与数据库的连接时，它会设置相应的参数，这取决于后台的使用情况。如果你启用了持续性连接，那么程序将不再重复每个请求。如果你修改了参数，例如连接的隔离级别或者是时区，你也应该恢复Django原有的默认值在每个请求结束时，强制使用一个合适的值在每个请求开始时，或者禁用持续性连接。







### 编码

Django假定所有的数据库使用UTF-8编码。使用其他的编码有可能会导致不可预知的行为发生，例如在Django中你的数据库里有效的数据可能会出现“value too long”的错误。在下面的信息中你可以查看数据库的具体说明来正确的设置你的数据库。







## PostgreSQL 说明

Django支持PostgreSQL 9.0和更高版本。它需要使用[psycopg2](http://initd.org/psycopg/) 2.4.5或更高版本 (或者 2.5+ 如果你想使用[`django.contrib.postgres`](contrib/postgres/index.html#module-django.contrib.postgres "django.contrib.postgres: PostgreSQL-specific fields and features")).

如果你使用的是Windows操作系统，请查看一下我们非官方psycopg2的[compiled Windows version](http://stickpeople.com/projects/python/win-psycopg/)



### PostgreSQL 连接设置

参考[`HOST`](settings.html#std:setting-HOST)来了解详细信息。





### Optimizing PostgreSQL’s configuration

Django规定它的数据库连接需要下列参数：

*   `client_encoding`: `'UTF8'`,
*   `default_transaction_isolation`: `'read committed'`为默认值，或者是连接选项中设置的值（参见下文），
*   `timezone`: `'UTC'` when [`USE_TZ`](settings.html#std:setting-USE_TZ) is `True`, value of [`TIME_ZONE`](settings.html#std:setting-TIME_ZONE) otherwise.

如果这些参数已经有了正确的值，Django将不会为每一个新的连接设置它们，这略微提高了性能。你可以直接在`postgresql.conf`中配置它们或者更方便的使用[ALTER ROLE](http://www.postgresql.org/docs/current/interactive/sql-alterrole.html)来为每一个database user(...设置?).

没有这个优化，Django也会工作得很好，但每一个新的连接会做一些额外的查询来设置这些参数。





### 隔离级别

像 PostgreSQL 本身，Django默认`READ COMMITTED` [隔离级别isolation level](http://www.postgresql.org/docs/current/static/transaction-iso.html). 如果你需要一个更高的隔离级别，例如 `REPEATABLE READ` 或者 `SERIALIZABLE`, 在 [`数据库databases`](settings.html#std:setting-DATABASES) [`OPTIONS`](settings.html#std:setting-OPTIONS) 设置部分的数据库配置 :





```
import psycopg2.extensions

DATABASES = {
    # ...
    'OPTIONS': {
        'isolation_level': psycopg2.extensions.ISOLATION_LEVEL_SERIALIZABLE,
    },
}

```







说明

在较高的隔离级别中，你的application应该做好准备去处理序列化失败中引发的异常(？不确定)。这个选项被设计为高级的用法。







### `varchar` 和 `text` 列的索引

当你在你的模块字段中指定了`db_index=True`, Django通常会输出一个单一的`CREATE INDEX`语句。但是，如果数据库类型对应的字段是`varchar`或者`text` (e.g., used by `CharField`, `FileField`, and `TextField`), 针对该列，Django会使用适当的[PostgreSQL operator class](http://www.postgresql.org/docs/current/static/indexes-opclass.html) 创建一个additional index.使用`LIKE`操作在SQL中,额外的索引是执行正确的查找所必需的，即用`contains` and `startswith`查找类型。







## MySQL 说明



### 版本支持 

Django 支持MySQL 5.5 和更高版本。

Django的`inspectdb`功能使用了`information_schema` database,它在所有的database schemas包含了详细的数据。

Django希望数据库支持Unicode (UTF-8编码),并且代理它去执transactions and referential integrity的任务。当你使用MyISAM储存引擎时，一个你需要注意到的事情是在MYSQL，后两个实际上是不去执行的，请参阅下一节。





### 存储引擎

MySQL有数种[存储引擎](http://dev.mysql.com/doc/refman/5.6/en/storage-engines.html). 你可以改变默认的存储引擎在不同的配置中。

直到MySQL 5.5.4版本, 默认存数引擎是[MyISAM](http://dev.mysql.com/doc/refman/5.6/en/myisam-storage-engine.html) [[1]](#id6). MyISAM数据的主要缺点是，它不支 transactions或者执行foreign-key约束。从好的方面来看，直到MySQL 5.6.4，它是唯一一个支持全文索引和搜索的引擎。

自从MySQL 5.5.5,默认的储存引擎变为了 [InnoDB](http://dev.mysql.com/doc/refman/5.6/en/innodb-storage-engine.html). 这个引擎是全事务和支持foreign key引用的。这可能是目前最好的选择。但是，请注意由于它不能够记取`AUTO_INCREMENT`的值，而不是重新创建像 “max(id)+1”这样，导致InnoDB的autoincrement counter在MySQL中丢失了。这可能导致一个无意重用的[`AutoField`](models/fields.html#django.db.models.AutoField "django.db.models.AutoField")

如果将现有项目升级到MySQL5.5.5，随后添加一些表，确保您的表可以使用相同的存储引擎（如MyISAM数据与InnoDB的）。特别注意，如果你的表中存在`ForeignKey`，比较他们在不同的存储引擎中,你可能会看到如下的错误当运行`migrate`的时候:





```
_mysql_exceptions.OperationalError: (
    1005, "Can't create table '\\db_name\\.#sql-4a8_ab' (errno: 150)"
)

```





<colgroup><col class="label"><col></colgroup>
| [[1]](#id4) | Unless this was changed by the packager of your MySQL package. We’ve had reports that the Windows Community Server installer sets up InnoDB as the default storage engine, for example. |





### MySQL DB API 驱动

Python的Database API 被描述为[**PEP 249**](http://www.python.org/dev/peps/pep-0249). MySQL拥有三种很棒的能够实现API的驱动：

*   [MySQLdb](https://pypi.python.org/pypi/MySQL-python/1.2.4)是一个由Andy Dustman开发，已经发展并支持十多年的一个本地驱动。
*   [mysqlclient](https://pypi.python.org/pypi/mysqlclient)是`MySQLdb`的一个分支，它与python3有着特别好的契合并且可以作为MySQLdb的直接替代。在书写这篇的时候，这是在Django使用MySQL的**推荐的选择。**
*   [MySQL Connector/Python](http://dev.mysql.com/downloads/connector/python)是一个来自Oracle的纯python驱动，它不需要MySQL client库或在标准库之外的任何Python模块。

所有这些驱动都是线程安全的，并提供连接池。`MySQLdb`是当前唯一一个不支持python3的。

除了一个DB API驱动，Django需要一个适配器来通过它的ORM访问数据库驱动。Django对 MySQLdb/mysqlclient提供了适配器直到MySQL Connector/Python包含了[its own](http://dev.mysql.com/doc/refman/5.6/en/connector-python-info.html)。



#### MySQLdb

Django需要MySQLdb version 1.2.1p2或更高版本。

在写文档的时候，最新版本MySQLdb、(1.2.5)还没有支持python3为了在Python 3使用MySQLdb, 你需要安装`mysqlclient`来替代它 。



说明

关于将date strings转化为 datetime objects，MySQLdb有已经存在的问题。具体来说，值为`0000-00-00`的date strings对于MySQL是有效的，但是会被MySQLdb转换成`None`。

这意味着当你在列中使用值可能为`0000-00-00`的[`loaddata`](django-admin.html#django-admin-loaddata)和 [`dumpdata`](django-admin.html#django-admin-dumpdata)时，你需要注意，因为它们会被转换为`None`。







#### mysqlclient

Django需要[mysqlclient](https://pypi.python.org/pypi/mysqlclient) 1.3.3或更高版本。需要注意，Python 3.2 不支持。除了 Python 3.3+ 支持, mysqlclient在其他版本表现的更相似于 MySQLDB。





#### MySQL Connector/Python

MySQL Connector/Python可从[download page](http://dev.mysql.com/downloads/connector/python/)下载。 Django适配器在1.1.X或更高版本可用。它可能不支持最新的Django的版本。







### 时区规定

如果您计划使用Django的[_timezone support，使用_](../topics/i18n/timezones.html)_ [mysql_tzinfo_to_sql加载时区表到MySQL数据库。](http://dev.mysql.com/doc/refman/5.6/en/mysql-tzinfo-to-sql.html)_这需要在你的MySQL服务器部署一次就好，而不是在每个数据库上。





### 创建数据库

你可以使用命令行工具运行这条SQL语句来[create your database:](http://dev.mysql.com/doc/refman/5.6/en/create-database.html)





```
CREATE DATABASE <dbname> CHARACTER SET utf8;

```





这可以确保所有表和列默认使用UTF-8。



#### 排序设置

一列的排序规则设置控制了数据排序的顺序，以及字符串的比较。它可以被设置在数据库的水平，也可以在每一个表和每一列这个是在MySQL文档中[详细记载的](http://dev.mysql.com/doc/refman/5.6/en/charset.html)。在所有情况下，你可以直接操作数据库中的表来设置排序。Django不提供在模型定义上设置它的方式。

默认情况下，在一个UTF8编码的数据库中，MySQL将会使用`utf8_general_ci`排序规则。这会导致所有的字符串比较是否相等的结果是以_不区分大小写_ 的方式完成的。就像这样，`"Fred"` and `"freD"` 被认为是相等的在数据库级别上。如果你在字段里有一个特殊的约束，如果试图将 `"aa"` 和`"AA"`插入到同一列会是非法的，因为他们在默认的排序(and, hence, non-unique) 中比较相等。

在许多情况下，这种默认情况将不会成为一个问题。然而,如果你真的想要区分大小写比较在一个特定的列或表,你要把这个列或表应用`utf8_bin` 排序。主要的在这种情况下需要注意的是,如果您正在使用MySQLdb 1.2.2 Django的数据库后端会返回bytestrings(而非unicode字符串)对于从数据库中获得的任何字符字段。Django平常的做法_总是_返回unicode字符串,与之相比这是一个显著的变化。这取决于你，开发者们，事实中你将会得到bytestrings的返回如果你配置你的表使用`utf8_bin`排序时。通常Django自身应当顺利地工作使用这样的列（除了`contrib.sessions` `Session` and `contrib.admin` `LogEntry`表描述的），如果你想要在你的工作中是数据保持一致，你的代码必须时刻准备调用`django.utils.encoding.smart_text()` ---Django不会为你做这些（数据库后曾和群体模块层在内部被分离以至于数据库层不知道它需要在这种特殊情况下作出转换）。

如果你使用MySQLdb 1.2.1p2，Django的标准[`CharField`](models/fields.html#django.db.models.CharField "django.db.models.CharField")类型将会返回unicode字符串即使存在`utf8_bin`序列。然而, [`TextField`](models/fields.html#django.db.models.TextField "django.db.models.TextField")字段将会像一个`array.array`实例返回（来自Python的标准 `array` 模块）。Django关于这方面不能做出很多，因为，下一次，当从数据库读取数据时，这些信息需要作出必要转换的行为将不可用。这个问题[固定在 MySQLdb 1.2.2](http://sourceforge.net/tracker/index.php?func=detail&aid=1495765&group_id=22307&atid=374932)，所以如果你想使用使用`utf8_bin`序列的[`TextField`](models/fields.html#django.db.models.TextField "django.db.models.TextField")，升级到1.2.2版本然后结合bytestrings处理（这不会太难）如上所述是推荐的解决方案。

你应该决定在 MySQLdb 1.2.1p2 or 1.2.2对你的表使用 `utf8_bin`排序规则，对于`django.contrib.sessions.models.Session`表(通常称为`django_session`)和`django.contrib.admin.models.LogEntry` 表 (通常称为`django_admin_log`)，你仍然应该使用`utf8_general_ci` (默认)排序规则。Those are the two standard tables that use [`TextField`](models/fields.html#django.db.models.TextField "django.db.models.TextField") internally.

请注意，根据 [MySQL Unicode字符集](http://dev.mysql.com/doc/refman/5.7/en/charset-unicode-sets.html), 比较下`utf8_general_ci`排序规则更加快速，但是与`utf8_unicode_ci`比较明显正确率略低。如果在你的应用中这是可接受的,你应该使用`utf8_general_ci`因为它更快。如果这是不能接受的 (例如,如果您需要德国字典顺序), 使用`utf8_unicode_ci` ，因为它更准确。



警告

模块层组以区分大小写的方式验证唯一的字段。因此当使用一个不区分大小写的序列，一个带有唯一字段值的层组，只有通过不同层组（？）才可通过验证，但是依据调用`save()`，`IntegrityError`错误将会被指出。









### 连接数据库

请参阅 [_settings documentation_](settings.html).

连接设置将被用在这些命令上：

1.  [`OPTIONS`](settings.html#std:setting-OPTIONS).
2.  [`NAME`](settings.html#std:setting-NAME), [`USER`](settings.html#std:setting-USER), [`PASSWORD`](settings.html#std:setting-PASSWORD), [`HOST`](settings.html#std:setting-HOST), [`PORT`](settings.html#std:setting-PORT)
3.  MySQL option files.

换句话说,如果你设置数据库的名称在[`OPTIONS`](settings.html#std:setting-OPTIONS), 这将优先于[`NAME`](settings.html#std:setting-NAME), 这将覆盖任何在[MySQL option file](http://dev.mysql.com/doc/refman/5.6/en/option-files.html)中的东西.

下面是使用一个MySQL选择文件一个示例配置:





```
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'OPTIONS': {
            'read_default_file': '/path/to/my.cnf',
        },
    }
}

# my.cnf
[client]
database = NAME
user = USER
password = PASSWORD
default-character-set = utf8

```





其他几个MySQLdb连接选项可能是有用的,例如`ssl`, `init_command`, and `sql_mode`. 查阅 [MySQLdb documentation](http://mysql-python.sourceforge.net/)了解更多细节。





### 创建你的表

当Django生成数据库得schema（数据库对象的集合）时，它没有指定一个储存引擎，所以你的表将被创建成你服务器配置的默认储存引擎。最简单的解决方案是设置你的数据库服务器的默认存储引擎为你所需求的引擎。

如果你使用托管服务,并且不能改变你的服务器的默认存储引擎,你有两种选择。

*   在表创建之后, 执行 `ALTER TABLE`语句来转换表为一个新的储存引擎 (例如 InnoDB):

    

    

```
ALTER TABLE <tablename> ENGINE=INNODB;
    
```

    

    

    如果你有很多表，这会很繁琐。

*   另一个选择是对于MySQLdb使用`init_command`选项在你创建表之前。

    

    

```
'OPTIONS': {
       'init_command': 'SET storage_engine=INNODB',
    }
    
```

    

    

    这设置默认的储存引擎在连接数据库之上。创建了表之后,您应该删除这个选项,因为它增加了一个只需要在每个表创建数据库连接时的查询。





### 表名

甚至在最新版本的MySQL中，这里存在[已知的问题](http://bugs.mysql.com/bug.php?id=48875)导致当某些SQL语句在一定条件下被执行下，表的名称可以被改变。建议您使用小写的表名，如果可能的话，避免任何可能从这个行为产生的问题。当Django从模型中自动生成表名时，它使用小写的表名，这主要考虑到如果你不顾问题让表名通过[` db_table `](models/options.html#django.db.models.Options.db_table "django.db.models.Options.db_table")参数。





### 保存点

Django ORM和MySQL (当使用InnoDB[_存储引擎_](#mysql-storage-engines)) 支持数据库[_保存点_](../topics/db/transactions.html#topics-db-transactions-savepoints)。

如果你使用MyISAM存储引擎请注意，你将会收到数据库生成的错误如果你尝试去使用[_savepoint-related methods of the transactions API_](../topics/db/transactions.html#topics-db-transactions-savepoints)。这个原因是检测一个MySQL数据库或表的存储引擎是一项expensive operation，所以它认定为在这些操作结果中无操作的情况下，它不值得去动态转换这些方法。





### 特定字段说明



#### Character fields

任何字段都被存储在 `VARCHAR`列类型的`max_length` ，它被限制在255个字符之内，如果您对这些字段使用`unique=True`的话。 这个影响到[`CharField`](models/fields.html#django.db.models.CharField "django.db.models.CharField"), [`SlugField`](models/fields.html#django.db.models.SlugField "django.db.models.SlugField") 和[`CommaSeparatedIntegerField`](models/fields.html#django.db.models.CommaSeparatedIntegerField "django.db.models.CommaSeparatedIntegerField")。





#### Time and DateTime fields的小数秒的支持

MySQL 5.6.4和更高版本可以存储小数秒，只需要列定义包含一个小数指示 (例如`DATETIME(6)`).。早期版本不支持它们。此外，MySQLdb的1.2.5以上版本存在一个[bug](https://github.com/farcepest/MySQLdb1/issues/24)，它会阻止MySQL使用小数秒。

如果数据库服务器支持的话，Django不会升级现有的列去包含小数秒。如果你想确保它们在一个当前的数据库中，这决定由你自己在目标数据库中手动去升级列，通过执行这样的一个命令：





```
ALTER TABLE `your_table` MODIFY `your_datetime_column` DATETIME(6)

```





或者在[_数据迁移_](../topics/migrations.html#data-migrations)中使用[`RunSQL`](migration-operations.html#django.db.migrations.operations.RunSQL "django.db.migrations.operations.RunSQL") 选项。

Changed in Django 1.8:

以前，Django在使用MySQL后端时删除（truncated）了来自`datetime` 和`time`值的小数秒。现在它让数据库决定他是否应该去删除（drop）部分的值。默认情况下, 新的`DateTimeField`或者`TimeField`列现在创建小数秒支持在MySQL5.6.4或更高版本和每个mysqlclient或MySQLdb 1.2.5或更高版本。







#### `TIMESTAMP`列

如果你使用了包含`TIMESTAMP` 列的遗留数据库,你必须设置[`USE_TZ = False`](settings.html#std:setting-USE_TZ)来避免数据丢失。[`inspectdb`](django-admin.html#django-admin-inspectdb) 映射这些列在[`DateTimeField`](models/fields.html#django.db.models.DateTimeField "django.db.models.DateTimeField")并且如果你启用时区支持,MySQL和Django将尝试从UTC的值转换为本地时间。







### Row locking with `QuerySet.select_for_update()`

MySQL不支持`NOWAIT`选项来 `SELECT ...` FOR UPDATE statement. 如果`select_for_update()`和`nowait=True`一起被使用，会导致一个`DatabaseError（数据库错误）` 产生。





### 自动类型转换会导致意想不到的结果

当你执行一个字符串类型的查询时，存在整型值时，MySQL会强制转换表里的所有值变为整型在你进行比较之前。如果你的表里包含`'abc'`, `'def'`和你查询`WHERE mycolumn=0`, 两行相匹配。同样, `WHERE mycolumn=1` 将会匹配`'abc1'`.。因此，Django中的字符串类型将会总是转换这个值为一个字符串当你查询中使用它之前。

如果你执行直接继承自[`Field`](models/fields.html#django.db.models.Field "django.db.models.Field")的自定义模型字段，会覆盖[`get_prep_value()`](models/fields.html#django.db.models.Field.get_prep_value "django.db.models.Field.get_prep_value")，或者使用[`extra()`](models/querysets.html#django.db.models.query.QuerySet.extra "django.db.models.query.QuerySet.extra")或者[`raw()`](../topics/db/sql.html#django.db.models.Manager.raw "django.db.models.Manager.raw"), 你应该确保执行了适当的类型。







## SQLite说明

[SQLite](http://www.sqlite.org/) 提供了一个极好的开发替代品对于主要为只读或者安装一个小的程序的应用程序。你应该知道的是，如同所有的数据库服务一样，SQLLite有一些不同于其他数据库的差异。



### 子串匹配和大小写敏感性

对于所有的SQLite版本，存在一些细微的违反语感的行为当你试图匹配某些类型的字符串。当你在Querysets中使用 [`iexact`](models/querysets.html#std:fieldlookup-iexact)或者 [`contains`](models/querysets.html#std:fieldlookup-contains) 过滤器，这些行为会被触发。 这种行为分为两个情况:

1\. 对于子串匹配，所有的匹配都会不区分大小写。这是一个过滤器例如 `filter(name__contains="aa")`将会匹配一个名称为`"Aabb"`。

2\. 对于字符串包含ASCII范围之外的字符，所有准确的字符串匹配都会被执行区分大小写的操作，甚至当不区分大小写的选项在查询中通过。所以在这些情况下，[`iexact`](models/querysets.html#std:fieldlookup-iexact)过滤器将会表现与[`exact`](models/querysets.html#std:fieldlookup-exact)过滤器一模一样。

一些可能的解决办法在 [documented at sqlite.org](http://www.sqlite.org/faq.html#q18)，但是他们不能利用默认的SQLite后端在Django中，强行将他们合并将会非常的困难。因此，Django显示默认的SQLite行为，当你使用不区分大小写或者子串过滤操作时你应该意识到这点。





### 旧版SQLite和`CASE`表达式

SQLite 3.6.23.1和更老版本包含了一个bug 当[handling query parameters](https://code.djangoproject.com/ticket/24148) 在一个 `CASE`表达式中包含了一个`ELSE`和算法。

2010年3月SQLite 3.6.23.1发布，目前大多数不同平台的二进制发行版包括一个新版本的SQLite，值得注意的是python 2.7 安装在windows平台上。

在撰写本文时,最新版本Windows - Python 2.7.9包括SQLite 3.6.21。你可以安装`pysqlite2`或者替换`sqlite3.dll` (默认情况下安装在`C:Python27DLLs`)通过来自[http://www.sqlite.org/](http://www.sqlite.org/) 的新版本来补救这个问题。





### 使用新版本的 SQLite DB-API 2.0驱动

Django将会使用一个`pysqlite2`模块优先于附带的`sqlite3`如果你发现一个是可用的在Python的标准库中。

这能够升级 DB-API 2.0接口或SQLite 3本身比那些包含在您的特定版本Python二进制发行版更新，如果你需要的话。





### “Database is locked” errors

SQLite是一个轻量级的数据库,因此不能支持一个高水平的并发性。`OperationalError: database is locked`错误表明你的应用正在经受更高的并发相较于`sqlite`能在默认配置中处理的那样。这个错误意味着一个线程或进程在数据库连接上存在一个交互型锁,另一个超时线程等待独自锁被释放。

Python的SQLite封装有一个默认的超时时间值决定了在超时之前第二个线程被允许等待多长时间在lock上并且会反馈`OperationalError: database is locked` error。

如果你出现了这个错误，你可以这样解决：

*   切换到另一个数据库端。在某种程度上SQLite变得太“lite”对于现实世界的应用程序,并且这些并发错误表明你已经达到了这一点。

*   重写代码,以减少并发性和确保数据库事务是短期的。

*   增加默认超时时间值通过设置`timeout`数据库选项：

    

    

```
'OPTIONS': {
        # ...
        'timeout': 20,
        # ...
    }
    
```

    

    

    这只会使SQLite等一段时间才会抛出“database is locked”错误；它不会真的做一些事情去解决他们。





### `QuerySet.select_for_update()` 不被支持

SQLite不支持 `SELECT ...` FOR UPDATE syntax. 它将没有影响。





### “pyformat” parameter style in raw queries not supported

对于大多数的数据库后端, 原始查询(`Manager.raw()`或者`cursor.execute()`) 可以使用 “pyformat”参数风格，在查询中placeholders作为 `'%(name)s'`并且被传递的参数是一个字典而不是列表。SQLite 不支持这个。





### 在 `connection.queries`参数不被引用

`sqlite3` 没有提供一种方式来检索SQL引用后和替换的参数。相反，SQL在 `connection.queries`被一个简单的字符串插值重建。它可能是不正确的。确保你在必要时添加引用钱拷贝一个引用在SQLite shell中。







## Oracle说明

Django 支持 [Oracle Database Server](http://www.oracle.com/) 11.1和更高版本 。4.3.1或更高版本的[cx_Oracle](http://cx-oracle.sourceforge.net/) Python驱动程序是必需的,尽管我们建议5.1.3或更高的支持Python 3的版本。

注意由于一个Unicode-corruption bug 在 `cx_Oracle` 5.0, 这个版本的驱动程序 **不能** 支持Django使用； `cx_Oracle` 5.0.1解决了这个问题,所以如果你希望使用一个更近时间发布的`cx_Oracle`, 使用5.0.1版本。

`cx_Oracle` 5.0.1或更高版本支持被随意编辑通过`WITH_UNICODE` 环境变量。这个建议不是必须的。

为了使`python manage.py migrate` 命令工作, 您的Oracle数据库用户必须拥有特权运行以下命令:

*   CREATE TABLE
*   CREATE SEQUENCE
*   CREATE PROCEDURE
*   CREATE TRIGGER

运行一个项目的测试套件， 用户通常需要这些_additional_ 特权：

*   CREATE USER
*   DROP USER
*   CREATE TABLESPACE
*   DROP TABLESPACE
*   CREATE SESSION WITH ADMIN OPTION
*   CREATE TABLE WITH ADMIN OPTION
*   CREATE SEQUENCE WITH ADMIN OPTION
*   CREATE PROCEDURE WITH ADMIN OPTION
*   CREATE TRIGGER WITH ADMIN OPTION

注意, 虽然 RESOURCE role 具有有需求 CREATE TABLE, CREATE SEQUENCE, CREATE PROCEDURE 和 CREATE TRIGGER 特权, 并且一个用户被允许RESOURCE当ADMIN OPTION能够授权 RESOURCE这样一个用户不能授予个人特权(例如 CREATE TABLE)，因此RESOURCE伴随ADMIN OPTION通畅不满足运行测试。（这段没翻译好。。大家理解一下）

一些测试套件也创建视图;运行这些,用户还需要 CREATE VIEW WITH ADMIN OPTION 特权。特别的是，这是Django自己的测试套件所需要的。

Changed in Django 1.8:

Django 1.8之前,测试用户被允许CONNECT and RESOURCE roles,所以所需的额外的特权运行测试套件是不同的。



所有这些特权都包含在DBA role,这适合使用在一个私人开发人员的数据库。

Oracle数据库后端使用 `SYS.`DBMS_LOB包,所以用户需要执行权限。通常是在默认情况下所有用户都可以访问的,但如果不是,你需要授予权限如下:





```
GRANT EXECUTE ON SYS.DBMS_LOB TO user;

```







### 连接到数据库

为了连接使用你的Oracle数据库的服务名, 你的 `settings.py` 文件应该像这样：





```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.oracle',
        'NAME': 'xe',
        'USER': 'a_user',
        'PASSWORD': 'a_password',
        'HOST': '',
        'PORT': '',
    }
}

```





在这种情况下, 你应该让 [`HOST`](settings.html#std:setting-HOST) and [`PORT`](settings.html#std:setting-PORT) 置空。 然而，如果你不是使用一个 `tnsnames.ora`文件或者一个类似的命名方法 并且想要连接使用 SID (“xe” 在本例),填写[`HOST`](settings.html#std:setting-HOST) 和[`PORT`](settings.html#std:setting-PORT) 像如下：





```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.oracle',
        'NAME': 'xe',
        'USER': 'a_user',
        'PASSWORD': 'a_password',
        'HOST': 'dbprod01ned.mycompany.com',
        'PORT': '1540',
    }
}

```





你应该同时填写 [`HOST`](settings.html#std:setting-HOST) and [`PORT`](settings.html#std:setting-PORT), 或者全部置空。Django将会取决你的选择使用一个不同的连接描述符。





### 线程的选项

如果你计划运行Django在多线程环境中(例如Apache使用默认MPM模块在任何现代的操作系统)， 在你的Oracle数据库配置中你 **必须** 设置 `threaded`选项为True:





```
'OPTIONS': {
    'threaded': True,
},

```





未能这样做可能会导致崩溃和其他奇怪的行为。





### INSERT ... RETURNING INTO

默认情况下,当插入新的行时， Oracle 后端使用 `RETURNING INTO` 去检索一个有效的`AutoField`。这个行为可能会导致一个`DatabaseError` 在某些不寻常设置中, 例如当插入到一个远程表,或者在一个视图中随着一个`INSTEAD OF` 触发。`RETURNING INTO` 子句可以禁用通过`use_returning_into`选项在数据库设置中为Flase:





```
'OPTIONS': {
    'use_returning_into': False,
},

```





在这种情况, Oracle后端将会使用一个单独的 `SELECT` 查询来检索AutoField值。





### 命名问题

Oracle强加了一个30个字符的名称限制。 为了适应, 后端缩短了数据库表示符为最适长度，取代带着一个可重复的MD5散列值的缩短的名称的最后四个字符。此外,后端数据库标识符变成大写。

为了防止这些转换(这通常是只需要在处理遗留数据库或访问表属于其他用户),使用一个引用名称像`db_table`值这样:





```
class LegacyModel(models.Model):
    class Meta:
        db_table = '"name_left_in_lowercase"'

class ForeignModel(models.Model):
    class Meta:
        db_table = '"OTHER_USER"."NAME_ONLY_SEEMS_OVER_30"'

```





引用名称也可以被使用在Django的其他受支持的数据库后端;除了r Oracle, 然而,引用没有影响。

当运行 `migrate`，一个 `ORA-06552`错误可能出现，如果某些Oracle的keywords被使用作为模型字段名称或一个`db_column`选项的值时。Django引用在查询使用的所有的标示符来防止大多数这样的问你，但是这个错误仍然可以发生当Oracle数据类型被用为一个列名。特别是，避免使用这样的名称`date`, `timestamp`, `number` or `float` 作为一个字段名。





### NULL和空字符串

Django通常更喜欢使用空字符串(“”)而不是NULL,但是Oracle将两个相同对待。为了解决这个问题，Oracle后端忽略了字段中一个explict的`null`选项，让空字符串作为一个可能的值并生成DDL就像`null=True`。当从数据库获取时,它假定一个`NULL`值在这些字段之一意味一个空字符串，并且数据默认转换去反映这种假设。





### `TextField`限制

The Oracle后端存储 `TextFields`像`NCLOB` 列.。通常使用LOB列时，Oracle 强加了一些限制：

*   LOB 列不被像主键一样使用。
*   LOB列不得用于索引。
*   LOB列不被使用在一个`SELECT DISTINCT` 列表。 这意味着当试图去使用 `QuerySet.distinct`方法在一个包含 `TextField`列的模型中将会导致一个错误当违反Oracle时。一个解决方案是, 使用 `QuerySet.defer`方法连同`distinct()`来防止将要包含在 `SELECT DISTINCT`列表中的 `TextField`列。







## 使用第三方数据库后端

除了官方支持的数据库,这些第三方提供的后端也允许您在Django中使用其他数据库：

*   [SAP SQL Anywhere](https://github.com/sqlanywhere/sqlany-django)
*   [IBM DB2](http://code.google.com/p/ibm-db/)
*   [Microsoft SQL Server](http://django-mssql.readthedocs.org/en/latest/)
*   [Firebird](https://github.com/maxirobaina/django-firebird)
*   [ODBC](https://github.com/lionheart/django-pyodbc/)
*   [ADSDB](http://code.google.com/p/adsdb-django/)

Django版本和ORM特性被支持通过这些非官方的后端相当大的变化。查询这些非官方的后端有关的特定功能，以及任何支持的查询，应该针对每个第三方项目提供的支持项目。



