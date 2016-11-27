

# 数据库事务

Django 为你提供几种方法来控制如何管理数据库事务。



## 管理数据库事务



### Django’s default transaction behavior

Django 的默认行为是运行在自动提交模式下。任何一个查询都立即被提交到数据库中，除非激活一个事务。[_具体细节看下面_](#autocommit-details).

Django 用事务或者保存点去自动的保证复杂ORM各种查询操作的统一性,尤其是 [_delete()_](queries.html#topics-db-queries-delete) 和[_update()_](queries.html#topics-db-queries-update) 查询.

Django’s [`测试用例`](../testing/tools.html#django.test.TestCase "django.test.TestCase") 也包装了事务性能原因的测试类





### 把事务绑定到HTTP 请求上

在web上一种简单处理事务的方式是把每个请求用事务包装起来.在每个你想保存这种行为的数据库的配置文件中，设置 [`ATOMIC_REQUESTS`](../../ref/settings.html#std:setting-DATABASE-ATOMIC_REQUESTS)值为 `True`，

它是这样工作的。在调用一个view里面的方法之前，django开始一个事务如果发出的响应没有问题,Django就会提交这个事务。如果在view这里产生一个异常，Django就会回滚这次事务

你可能会在你的视图代码中执行一部分提交并且回滚，通常使用[`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")context管理器.但是最后你的视图，要么是所有改变都提交执行，要么是都不提交。



警告

虽然这种简洁的事物模型看上去很吸引人, 但要注意当流量增长时它会表现出较差的效率。对每个视图开启一个事务是有所耗费的。其对性能的影响依赖于应用程序对数据库的查询语句效率和数据库当前的锁竞争情况。





预请求事务和流式响应

当一个视图返回一个 [`StreamingHttpResponse`](../../ref/request-response.html#django.http.StreamingHttpResponse "django.http.StreamingHttpResponse")时, 其读取的内容是由执行代码来产生的。因为视图调用已经返回，这样代码在事务的外部运行。

一般而言，在产生一个流式响应时，不建议再进行写数据库的操作，因为没有明智的方式在开始发送响应之后来处理错误。



在实际操作时，可以通过如下 [`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")装饰器把这一功能简单地加载到视图函数上。?

表示事务仅仅是在当前视图中有效，诸如模板响应之类的中间件(Middleware)操作是运行在事务之外的。

当 [`ATOMIC_REQUESTS`](../../ref/settings.html#std:setting-DATABASE-ATOMIC_REQUESTS)被启用后，仍然有办法来阻止视图运行一个事务操作。



`non_atomic_requests`(_using=None_)[[source]](../../_modules/django/db/transaction.html#non_atomic_requests)



这个装饰器会否定一个由 [`ATOMIC_REQUESTS`](../../ref/settings.html#std:setting-DATABASE-ATOMIC_REQUESTS)设定的视图:





```
from django.db import transaction

@transaction.non_atomic_requests
def my_view(request):
    do_stuff()

@transaction.non_atomic_requests(using='other')
def my_other_view(request):
    do_stuff_on_the_other_database()

```





它将仅工作在设定了此装饰器的视图上。









### 更加明确地控制事务

Django提供了单一的API来控制数据库事务。



`atomic`(_using=None_, _savepoint=True_)[[source]](../../_modules/django/db/transaction.html#atomic)



原子性是由数据库的事务操作来界定的。 `atomic`允许我们在执行代码块时，在数据库层面提供原子性保证。 如果代码块成功完成， 相应的变化会被提交到数据库进行commit；如果执行期间遇到异常，则会将该段代码所涉及的所有更改回滚。.

`atomic`块可以嵌套。 在下面的例子里，使用with语句，当一个内部块完成后，如果某个异常在外部块被抛出，内部块上的操作仍然可以回滚(前提是外部块也被atomic装饰过)。

`atomic` 被用作[_装饰器_](https://docs.python.org/3/glossary.html#term-decorator "(in Python v3.4)"):





```
from django.db import transaction

@transaction.atomic
def viewfunc(request):
    # This code executes inside a transaction.
    do_stuff()

```





atomic被用作 [_上下文管理器_](https://docs.python.org/3/glossary.html#term-context-manager "(in Python v3.4)"):





```
from django.db import transaction

def viewfunc(request):
    # This code executes in autocommit mode (Django's default).
    do_stuff()

    with transaction.atomic():
        # This code executes inside a transaction.
        do_more_stuff()

```





经过 `atomic`装饰的代码在一个 try/except 块内允许使用常见的完整性错误检测语法:





```
from django.db import IntegrityError, transaction

@transaction.atomic
def viewfunc(request):
    create_parent()

    try:
        with transaction.atomic():
            generate_relationships()
    except IntegrityError:
        handle_exception()

    add_children()

```





在这个例子中，即使`generate_relationships()` 违反完整性约束导致了数据库错误， 你仍可以进行 `add_children()`的操作, 并且`create_parent()`的变化仍然存在。注意，当?`handle_exception()`被触发时，在`generate_relationships()`上的尝试操作已经被安全回滚，所以若有必要，这个异常的句柄也能够操作数据库。



避免在 `atomic`里捕获异常!

当一个`原子块`执行完退出时，Django会审查是正常提交还是回滚。如果你在 `原子`块中捕获了异常的句柄, 你可能就向Django隐藏了问题的发生。这可能会导致意想不到的后果。

这主要是考虑到 [`DatabaseError`](../../ref/exceptions.html#django.db.DatabaseError "django.db.DatabaseError")和其诸如[`IntegrityError`](../../ref/exceptions.html#django.db.IntegrityError "django.db.IntegrityError")这样的子类。 若是遇到这样的错误，事务的原子性会被打破，Django会在`原子`代码块上执行回滚操作。如果你试图在回滚发生前运行数据库查询，Django会产生一个[`TransactionManagementError`](../../ref/exceptions.html#django.db.transaction.TransactionManagementError "django.db.transaction.TransactionManagementError")的异常。当一个ORM-相关的信号句柄操作异常时，你可能也会遇到类似的情形。

正确捕捉数据库异常应该是类似上文所讲 ，基于`atomic` 代码块来做。若有必要,可以额外增加一层`atomic`代码来用于此目的。这种模式还有另一个优势：它明确了当一个异常发生时，哪些操作将回滚。

如果你是从原始的SQL查询语句中捕获异常，则Django的行为是不明确的，而且是依赖于数据库的。



为了确保原子性达成, `atomic`会 disables一些 APIs. 在`atomic`代码块内试图 commit, roll back,或者更改数据库autocommit的状态都会导致异常。

`atomic`使用的 `using` 参数必须是数据库的名字. 如果这个参数没提供, Django默认使用 `"default"` 数据库。

在底层，Django的事务管理代码：

*   当进入到最外层的 `atomic` 代码块时会打开一个事务;
*   当进入到内层`atomic`代码块时会创建一个保存点;
*   当退出内部块时会释放或回滚保存点;
*   当退出外部块时提交或回退事物。

你可以通过设置`savepoint` 参数为 `False`来使对内层的保存点失效。如果异常发生，若设置了savepoint，Django会在退出第一层代码块时执行回滚，否则会在最外层的代码块上执行回滚。 原子性始终会在外层事物上得到保证。这个选项仅仅用在设置保存点开销很明显时的情况下。它的缺点是打破了上述错误处理的原则。

在autocommit关闭的情况下，你可以使用`atomic`. 这只使用savepoints功能，即使是对于最外层的块。如果在最外层的块上声明`savepoint=False`，这将会产生一个错误。







性能考虑

所有打开的事务会对数据库带来性能成本。要尽量减少这种开销，尽量保持您的交易尽可能短。 在Django的请求/响应周期，如果你使用?[`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")来执行长运行的进程，这尤其重要。









## Autocommit



### 为什么 Django会使用autocommit

在SQL标准中, 每个SQL语句在执行时都会启动一个事务，除非已经存在一个事务了。这样事务必须明确是提交还是回滚。

对应用程序开发者而言，这样非常不方便。为了避免这个问题，大多数数据库提供了一个autocommit模式。当 autocommit被打开并且事物处于活动状态时，每个SQL查询都可以看成是一个事物。也就是说, 不但每个查询是每个事物的开始，而且每个事物会自动提交或回滚，这取决于该查询是否成功执行。

[**PEP 249**](http://www.python.org/dev/peps/pep-0249), Python数据库API 规范v2.0, 需要将autocommit初试设置为关闭状态。Django覆盖了这个默认规范并且将autocommit设置为 on.

要想避免这样, 你可以[_关闭事务管理器_](#deactivate-transaction-management), 但不建议这样做。





### 关闭事务管理器

你可以在配置文件里通过设置[`AUTOCOMMIT`](../../ref/settings.html#std:setting-DATABASE-AUTOCOMMIT)为?`False` 完全关闭Django的事物管理。如果这样做了，Django将不能启用autocommit,也不能执行任何 commits. 你只能遵照数据库层面的规则行为。

这就需要你对每个事物执行明确的commit操作，即使由Django或第三方库创建的。因此，这最好只用于你自定义的事物控制中间件或者是一些比较奇特的场景。







## 更低级别的API



警告

如果可能的话，尽量优先选择[`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")来控制事物 ，它遵守数据库的相关特性并且防止了非法操作。

低级别 API仅仅用于你自定义的事物管理场景。





### Autocommit

在 [`django.db.transaction`](#module-django.db.transaction "django.db.transaction")模块里，Django提供了一个简单的API， 用于管理每个数据库的自提交状态。



`get_autocommit`(_using=None_)[[source]](../../_modules/django/db/transaction.html#get_autocommit)





`set_autocommit`(_autocommit_, _using=None_)[[source]](../../_modules/django/db/transaction.html#set_autocommit)



这些函数使用了一个 `using`参数，参数的值是数据库的名字。 如果参数没有提供, Django使用 `"default"` 数据库。

Autocommit初始是打开的。如果你把它关掉了,那么你有义务恢复它。

一旦你把autocommit 关掉了, 那么你得到就是数据库的默认行为， Django 不会帮你做任何事。虽然在 [**PEP 249**](http://www.python.org/dev/peps/pep-0249)有描述此规范,但数据库适配器的实现并不总与规范是一致的。请仔细检查你当前正在使用的数据库适配器文档。

在把autocommit改回on之前，你必须确保所有的SQL事物处于非活跃状态，通常是使用[`commit()`](#django.db.transaction.commit "django.db.transaction.commit")或者[`rollback()`](#django.db.transaction.rollback "django.db.transaction.rollback")这样的语法操作。

当 [`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")代码块处于活跃状态时，Django会拒绝将autocommit从on的状态调整为off，因为这样会破坏原子性。?





### 事务

事务是一系列数据库语句的原子集。即使程序在运行时崩溃了,数据库可以确保事物集中的所有变更要么都被提交，要么都被放弃。

Django 并没有提供一个API来开启一个事务。因为开始事务的预期方式是将[`set_autocommit()`](#django.db.transaction.set_autocommit "django.db.transaction.set_autocommit")设置为disable状态。

一旦你处于一个事物之中，你可以选择要么apply所有变更?[`commit()`](#django.db.transaction.commit "django.db.transaction.commit")提交它，要么取消所有变化 [`rollback()`](#django.db.transaction.rollback "django.db.transaction.rollback"). 这个函数功能是在 [`django.db.transaction`](#module-django.db.transaction "django.db.transaction")定义的。



`commit`(_using=None_)[[source]](../../_modules/django/db/transaction.html#commit)





`rollback`(_using=None_)[[source]](../../_modules/django/db/transaction.html#rollback)



这个函数通过 `using`指定数据库的名字作为参数。如果没提供, Django 使用`"default"` 数据库.

当一个?[`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")程序块在运行状态，Django会拒绝 commit 或rollback操作，因为这些操作是自动的。





### Savepoints

保存点是在事物执行过程中的一个标记，它可以让你执行回滚事物的一部分 ，而不是整个事物。 保存点在 SQLite (≥ 3.6.8), PostgreSQL, Oracle和MySQL (当使用nnoDB存储引擎时)是有效的。在其他的数据库后端虽然也提供保存点的函数，但其实它们是空操作，实际不起任何作用。

如果你开启了autocommit，Savepoints并没有太多用处，因为这是Django的默认行为。然而, 一旦你使用 [`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")开启了一个事物, 那么你所建立的一系列数据库操作将被视为一个整体，等待同时提交或回滚。 如果你触发了一个回滚,那么整个事物就要进行回滚。Savepoints提供了更细粒度的回滚，而不是用 `transaction.rollback()`对整个事物进行回滚.

当嵌套使用 [`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")装饰器时, 它会创建 savepoint以允许部分提交或回滚。 强烈建议你使用 [`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")而不是下面描述的函数功能，当然他们也是公开API的一部分，并且现在也没有废除它们的计划。

每个函数都带一个 `using`参数，这个参数是你要操作的数据库的名字。 如果没有 `using`参数，则会使用 `"default"` 数据库。

Savepoints 是由[`django.db.transaction`](#module-django.db.transaction "django.db.transaction")里的三个函数来控制的：



`savepoint`(_using=None_)[[source]](../../_modules/django/db/transaction.html#savepoint)



创建一个新的保存点。这将实现在事物里对“好”的状态做一个标记点。返回值是 savepoint ID (`sid`).







`savepoint_commit`(_sid_, _using=None_)[[source]](../../_modules/django/db/transaction.html#savepoint_commit)



释放保存点`sid`. 自创建保存点进行的更改将成为事物的一部分。







`savepoint_rollback`(_sid_, _using=None_)[[source]](../../_modules/django/db/transaction.html#savepoint_rollback)



回滚事物保存点`sid`.





如果不支持保存点或者数据库未处于autocommit模式，这些函数将什么也不做。

此外，还有一个实用的功能：



`clean_savepoints`(_using=None_)[[source]](../../_modules/django/db/transaction.html#clean_savepoints)



重置用来生成唯一保存点ID的计数器：





下面的例子演示了如何使用保存点：





```
from django.db import transaction

# open a transaction
@transaction.atomic
def viewfunc(request):

    a.save()
    # transaction now contains a.save()

    sid = transaction.savepoint()

    b.save()
    # transaction now contains a.save() and b.save()

    if want_to_keep_b:
        transaction.savepoint_commit(sid)
        # open transaction still contains a.save() and b.save()
    else:
        transaction.savepoint_rollback(sid)
        # open transaction now contains only a.save()

```





保存点通过实现部分回滚实现对数据库报错的恢复。如果你是在一个 [`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic")块中这么干的话, 那整个块都会被回滚，因为Django并不知你在下一层做此处理操作。为了避免这样，你可以在下面函数中控制回滚行为。



`get_rollback`(_using=None_)[[source]](../../_modules/django/db/transaction.html#get_rollback)





`set_rollback`(_rollback_, _using=None_)[[source]](../../_modules/django/db/transaction.html#set_rollback)



当退出最内层atomic块时设置回滚标记为 `True`以实现强制回滚。这对在没有抛出异常的情况下触发一个回滚操作是很有用的。

将标志设为 `False` 阻止这样一个回滚。 在此之前，请确保你已经把事物回滚到了该原子块内一个已知良好的保存点。否则，你打破原子性，并且数据损坏可能会发生。







## 具体数据库的相关说明



### Savepoints in SQLite

?SQLite ≥ 3.6.8之后开始支持savepoints,但由于[`sqlite3`](https://docs.python.org/3/library/sqlite3.html#module-sqlite3 "(in Python v3.4)")模块的设计缺陷导致其很难使用。

当自动提交被启用，保存点是没有意义的. 当被禁用时， [`sqlite3`](https://docs.python.org/3/library/sqlite3.html#module-sqlite3 "(in Python v3.4)")在savepoint之前已经进行了的隐式的提交。(实际上, 在任何诸如`SELECT`, `INSERT`, `UPDATE`, `DELETE`和 `REPLACE`等操作语句之前都会进行提交.) 这个问题有两个后果：

*   savepoint的低级别API只能用于内部事物。在一个[`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic") 块之内。
*   当autocommit处于关闭状态时，是不可能使用[`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic") 的。





### MySQL中的事务

如果你正在使用 MySQL, 你的表也许支持事务，也许不支持。这具体依赖于你的MySQL版本和你正在使用的表的engine类型。(这里的表engine类型是指“InnoDB” 或 “MyISAM”等.) MySQL的事物特性不在本文讨论的范围之内，你可以从MySQL的官方站点获取[MySQL事物的相关信息](http://dev.mysql.com/doc/refman/5.6/en/sql-syntax-transactions.html).

如果你安装的MySQL _不_支持事务, 那么Django会一直工作在自动提交模式 : 语句一旦被调用就会被执行和提交。 如果你安装的MySQL_确定_支持事物, Django会遵循本文所介绍的关于事务的处理原则。





### 处理PostgreSQL的交易中的异常



注意

本节内容只有当你实现你自己的事务管理时才相关。 这个问题不会出现在 Django默认模式和 [`atomic()`](#django.db.transaction.atomic "django.db.transaction.atomic") 自动控制的情况下。



在一个事物内部, 当调用一个PostgreSQL光标抛出一个异常 (通常是 `IntegrityError`), 后续所有在此同一个事物中的SQL将失败并报以下错误“current transaction is aborted, queries ignored until end of transaction block”. 虽然有时简单用`save()`是不太可能导致PostgreSQL异常的, 但仍然有更高级模式用法的可能, 例如在一个有唯一约束的字段保存对象, 保存使用force_insert/force_update 标志,或者调用一些定制化的SQL。

有几种方法可以从这种错误中恢复过来。



#### 事务回滚

第一个选择是回滚整个事物。例如：





```
a.save() # Succeeds, but may be undone by transaction rollback
try:
    b.save() # Could throw exception
except IntegrityError:
    transaction.rollback()
c.save() # Succeeds, but a.save() may have been undone

```





调用 `transaction.rollback()` 回滚整个事物。任何未提交的数据库操作都会丢失。在此例中, 由 `a.save()`所保存的变更将会丢失,即使这个操作自身没有产生错误。





#### 保存点回滚

你可以使用 [_savepoints_](#topics-db-transactions-savepoints)来控制一个回滚的扩展。 在执行数据库操作可能失败之前，你可以设置或更新保存点； 这样，如果操作失败，您可以回滚该单违规操作，而不是整个事务。例如：





```
a.save() # Succeeds, and never undone by savepoint rollback
sid = transaction.savepoint()
try:
    b.save() # Could throw exception
    transaction.savepoint_commit(sid)
except IntegrityError:
    transaction.savepoint_rollback(sid)
c.save() # Succeeds, and a.save() is never undone

```





在此例中, 当`b.save()`抛出异常的情况下，`a.save()` 所做的更改将不会丢失。







