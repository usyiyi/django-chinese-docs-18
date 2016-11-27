

# 为Django编写首个补丁



## 介绍

有兴趣为社区做出点贡献吗？也许你会在Django中发现你想要修复的漏洞，或者你希望为它添加一个小功能。

为Django作贡献这件事本身就是使你的顾虑得到解决的最好方式。一开始这可能会使你怯步，但事实上是很简单的。整个过程中我们会一步一步为你解说，所以你可以通过例子学习。



### 本页教程面向的读者

使用教程前，我们希望你至少对于Django的运行方式有基础的了解。这意味着你可以自如地在[_写你自己的Django app_](tutorial01.html)时使用教程。 除此之外，你应该对于Python本身有很好的了解。如果您并不太了解， 我们为您推荐[Dive Into Python](http://www.diveintopython3.net/)，对于初次使用Python的程序员来说这是一本很棒（而且免费）的在线电子书。

对于版本控制系统及Trac不熟悉的人来说，这份教程及其中的链接所包含的信息足以满足你们开始学习的需求。然而，如果你希望定期为Django贡献，你可能会希望阅读更多关于这些不同工具的信息。

当然对于其中的大部分内容，Django会尽可能做出解释以帮助广大的读者。



何处获得帮助:

如果你在使用本教程时遇到困难，你可以发送信息给[_django开发者_](../internals/mailing-lists.html#django-developers-mailing-list) 或者登陆 [#django-dev on irc.freenode.net](irc://irc.freenode.net/django-dev) 向其他Django使用者需求帮助。







### 教程包含的内容

一开始我们会帮助你为Django编写补丁，在教程结束时，你将具备对于工具和所包含过程的基本了解。准确来说，我们的教程将包含以下几点：

*   安装Git。
*   如何下载Django的开发副本
*   运行Django的测试组件
*   为你的补丁编写一个测试
*   为你的补丁编码。
*   测试你的补丁。
*   为你所做的改变写一个补丁文件。
*   去哪里寻找更多的信息。

一旦你完成了这份教程，你可以浏览剩下的[_Django’s documentation on contributing_](../internals/contributing/index.html). 它包含了大量信息。任何想成为Django的正式贡献者必须去阅读它。如果你有问题，它也许会给你答案







## 安装Git

使用教程前，你需要安装好Git，下载Django的最新开发版本并且为你作出的改变生成补丁文件

为了确认你是否已经安装了Git, 输入 `git` 进入命令行。如果信息提示命令无法找到, 你就需要下载并安装Git, 详情阅读 [Git’s download page](http://git-scm.com/download).

如果你还不熟悉 Git, 你可以在命令行下输入 `git help` 了解更多关于它的命令（确认已安装）。





## 获取Django 开发版的副本

为Django贡献的第一步就是获取源代码复本。在命令行里， 使用 `cd` 命令进入你想要保存Django的目录

使用下面的命令来下载Django的源码库





```
git clone https://github.com/django/django.git

```







注意

对那些希望使用 [virtualenv](http://www.virtualenv.org)的人，你可以用:





```
pip install -e /path/to/your/local/clone/django/

```





(你clone的`django` 目录包含 `setup.py`) ，它可以链接到你的cloned确认一个虚拟环境。这是一个伟大的选择，你开发的 Django 副本从您的系统的其余部分隔离，避免了潜在冲突的包。







## 回滚到更早的Django版本

这个教程中，我们使用 [#17549](https://code.djangoproject.com/ticket/17549)问题来作为学习用例，所以我们要把git中Django的版本回滚到这个问题的补丁没有提交之前。这样的话我们就可以参与到从草稿到补丁的所有过程，包括运行Django的测试套件。

**请记住，我们将用Django的老版本来到达学习的目的，通常情况下你应当使用当前最新的开发版本来提交补丁。**



注意

这个补丁由 Ulrich Petri 开发， Git  提交到 Django 源码 [提交id为ac2052ebc84c45709ab5f0f25e685bf656ce79bc](https://github.com/django/django/commit/ac2052ebc84c45709ab5f0f25e685bf656ce79bc). 因此，我们要回到补丁提交之前的版本号 [提交ID： 39f5bc7fc3a4bb43ed8a1358b17fe0521a1a63ac](https://github.com/django/django/commit/39f5bc7fc3a4bb43ed8a1358b17fe0521a1a63ac).



首先打开Django源码的根目录（这个目录包含了  `django`, `docs`, `tests`, `AUTHORS`, 等） 然后你你可以根据下面的教程check out老版本的Django：





```
git checkout 39f5bc7fc3a4bb43ed8a1358b17fe0521a1a63ac

```









## 首先运行Django 的测试套件

当你贡献代码给Django的时候，一个非常重要的问题就是你修改的代码不要给其他部分引入新的bug。 有个办法可以在你更改代码之后检查Django是否能正常工作，就是运行Django的测试套件。如果所有的测试用例都通过，你就有理由相信你的改动完全没有破坏Django。如果你从来没有运行过Django的测试套件，那么比较好的做法是事先运行一遍，熟悉下正常情况下应该输出什么结果。

你可以简单的通过`cd`到Django  `tests/` 目录下执行测试，如果你是用GNU/Linux, Mac OS X或者其你喜欢的其他Unix系统，执行：





```
PYTHONPATH=.. python runtests.py --settings=test_sqlite

```





如果你使用 Windows，安装 Git 后默认生成 Git 命令行环境： “Git Bash” ，在命令行中环境中执行上面的测试命令。GitHub提供了一个[很好的教程](https://help.github.com/articles/set-up-git#platform-windows)。



Note

如果你使用了 `virtualenv`，你可以在执行测试时省略 `PYTHONPATH=..`。表示在 `测试`.目录的上一层目录寻找 Django 。`virtualenv` 自动把 Django 放在 `PYTHONPATH` 目录下。



现在坐下来放松一下。Django的整个测试套件有超过4800种不同的测试,所以它运行时间需要5到15分钟,这取决于你的电脑的速度。

Django的测试套件运行时,您将看到一个字符流代表每个测试的运行的状态。 `E` 表示测试中出现异常 和 `F` 表示断言失败。这两种情况都被认为测试失败。同时，`X` 和 `S` 分别表示与期望结果不同和跳过测试。 点表示测试通过。

跳过测试主要由缺少测试所需的外部库引起；查看 [_Running all the tests_](../internals/contributing/writing-code/unit-tests.html#running-unit-tests-dependencies) 获取测所需依赖包，并确保安装由于代码修改造成的新依赖包（这篇教程不需要额外安装依赖包）。

当测试执行完毕后，得到反馈信息显示测试已通过，或者测试失败。因为还没有对 Django 的源码做任何修改，所有的测试用例**应该**测试通过。如果测试失败或出现错误，回头确认以上执行操作是否正确。查看 [_Running the unit tests_](../internals/contributing/writing-code/unit-tests.html#running-unit-tests) 获取更多信息。

注意最新版本 Django 分支不总稳定。当在分支上开发时，你可以查看代码持续集成构建页面的信息 [Django’s continuous integration builds](http://djangoci.com) 来判断测试错误只在你指定的电脑上发生，还是官方版本中也存在该错误。如果点击某个构建信息，可以通过配置列表信息查看错误发生时 Python 以及后端数据库的信息。



Note

在本教程以及所用分支中，测试使用数据库 SQLite 即可， 然而在某些情况下需要 [_测试更多不同的数据库_](../internals/contributing/writing-code/unit-tests.html#running-unit-tests-settings)。







## 为你的ticket写一些测试用例

大多数情况下，Django 的补丁必需包含测试。Bug 修复补丁的测试是一个回归测试，确保该 Bug 不会再次在 Django 中出现。该测试应该在 Bug 存在时测试失败，在 Bug 已经修复后通过测试。新功能补丁的测试必须验证新功能是否正常运行。新功能的测试将在功能正常时通过测试，功能未执行时测试失败。

最好的方式是在修改代码之前写测试单元代码。这种开发风格叫做 [测试驱动开发](http://en.wikipedia.org/wiki/Test-driven_development) 被应用在项目开发和单一补丁开发过程中。测试单元编写完毕后，执行测试单元，此时测试失败（因为目前还没有修复 BuG 或 添加新功能），如果测试成功通过，你需要重新修改测试单元保证测试失败。然而测试单元并没有阻止 BUG 发生的作用。

现在我们的操作示例。



### 为分支 #17549 写测试

分支 [#17549](https://code.djangoproject.com/ticket/17549) 描述了以下的额外功能。

> It’s useful for URLField to give you a way to open the URL; otherwise you might as well use a CharField.

为了解决这个问题，我们将添加一个 `render` 方法到 `AdminURLFieldWidget` ，通过表单显示一个可点击的链接。在更改代码之前，我们需要一组测试来验证将添加的功能现在以及未来都能正常工作。

进入 Django 下 `tests/regressiontests/admin_widgets/` 目录打开文件  `tests.py` 。在第 269行类`AdminFileWidgetTest` 之前添加以下内容：





```
class AdminURLWidgetTest(DjangoTestCase):
    def test_render(self):
        w = widgets.AdminURLFieldWidget()
        self.assertHTMLEqual(
            conditional_escape(w.render('test', '')),
            '<input class="vURLField" name="test" type="text" />'
        )
        self.assertHTMLEqual(
            conditional_escape(w.render('test', 'http://example.com')),
            '<p class="url">Currently:<a href="http://example.com">http://example.com</a><br />Change:<input class="vURLField" name="test" type="text" value="http://example.com" /></p>'
        )

    def test_render_idn(self):
        w = widgets.AdminURLFieldWidget()
        self.assertHTMLEqual(
            conditional_escape(w.render('test', 'http://example-äüö.com')),
            '<p class="url">Currently:<a href="http://xn--example--7za4pnc.com">http://example-äüö.com</a><br />Change:<input class="vURLField" name="test" type="text" value="http://example-äüö.com" /></p>'
        )

    def test_render_quoting(self):
        w = widgets.AdminURLFieldWidget()
        self.assertHTMLEqual(
            conditional_escape(w.render('test', 'http://example.com/<sometag>some text</sometag>')),
            '<p class="url">Currently:<a href="http://example.com/%3Csometag%3Esome%20text%3C/sometag%3E">http://example.com/&lt;sometag&gt;some text&lt;/sometag&gt;</a><br />Change:<input class="vURLField" name="test" type="text" value="http://example.com/<sometag>some text</sometag>" /></p>'
        )
        self.assertHTMLEqual(
            conditional_escape(w.render('test', 'http://example-äüö.com/<sometag>some text</sometag>')),
            '<p class="url">Currently:<a href="http://xn--example--7za4pnc.com/%3Csometag%3Esome%20text%3C/sometag%3E">http://example-äüö.com/&lt;sometag&gt;some text&lt;/sometag&gt;</a><br />Change:<input class="vURLField" name="test" type="text" value="http://example-äüö.com/<sometag>some text</sometag>" /></p>'
        )

```





该测试会验证我们新添加的方法 `render` 在不同情况下工作正常。



但是这个测试内容看起来比较难...

如果你没有写过测试，第一眼看上去测试代码会有点难。幸运的是测试在编程里是一个 _非常_ 重要的部分， 因此下面有更多的相关信息：

*   为 Django 添加测试代码浏览官网文档： [_编写和运行测试单元_](../topics/testing/overview.html).
*   深入 Python (一个在线免费的 Python 初学者教程) 包含了非常棒的 [测试单元介绍](http://www.diveintopython.net/unit_testing/index.html).
*   阅读以上文档后，如果想更深入了解测试内容，参考:  [Python 测试单元文档](https://docs.python.org/library/unittest.html).







### 编写新的测试

因为我们还没有对`AdminURLFieldWidget`做任何修改，所以我们的测试会失败。 我们在`model_forms_regress` 目录中运行所有测试，确保测试会失败。 在命令行中 `进入` Django 的 `tests/` 目录并执行：





```
PYTHONPATH=.. python runtests.py --settings=test_sqlite admin_widgets

```





如果测试方法运行正常，会出现三个测试失败信息，每个信息对应一个我们刚刚添加的新测试方法。如果所有测试方法都正常通过，请检查上面的测试方法是否添加到了正确的文件位置。







## 修改 Django 源码

我们在 Django 仓储的标签[#17549](https://code.djangoproject.com/ticket/17549)中添加新功能描述。



### 为标签#17549编写代码

进入目录 `django/django/contrib/admin/` 并打开文件 `widgets.py`。 在第302行找到类 `AdminURLFieldWidget`，在`__init__`方法后面添加新方法 `render` 内容如下：





```
def render(self, name, value, attrs=None):
    html = super(AdminURLFieldWidget, self).render(name, value, attrs)
    if value:
        value = force_text(self._format_value(value))
        final_attrs = {'href': mark_safe(smart_urlquote(value))}
        html = format_html(
            '<p class="url">{} <a {}>{}</a><br />{} {}</p>',
            _('Currently:'), flatatt(final_attrs), value,
            _('Change:'), html
        )
    return html

```









### 确保测试通过

修改 Django 源码后，我们通过之前编写的测试方法来验证源码修改是否工作正常。运行 `admin_widgets` 目录下所有的测试方法， `进入`  Django 的 `tests/` 目录然后运行：





```
PYTHONPATH=.. python runtests.py --settings=test_sqlite admin_widgets

```





哦,好事是我们写了这些测试! 但仍然收到三个测试异常：





```
NameError: global name 'smart_urlquote' is not defined

```





我们忘记导入这些方法。在 `smart_urlquote` 第 13 行的末尾添加`django/contrib/admin/widgets.py`，结果如下：





```
from django.utils.html import escape, format_html, format_html_join, smart_urlquote

```





重新运行测试方法正常会通过测试。如果没有通过测试，请重新确认上面提到的类 `AdminURLFieldWidget` 以及新添加的测试方法是否被正确复制到指定位置。







## 再次运行Django 的测试套件

如果已经确认补丁以及测试结果都正常，现在是时候运行 Django 完整的测试用例，验证你的修改是否对 Django 的其他部分造成新的 Bug。 虽然测试用例帮助识别容易被人忽略的错误，但测试通过并不能保证完全没有 Bug 存在。

运行 Django 完整的测试用例， `进入` Django 下  `tests/` 目录并执行：





```
PYTHONPATH=.. python runtests.py --settings=test_sqlite

```





只要没有看到测试异常，你可以继续下一步骤。注意这个修复会产生一个[小的 CSS 变动](https://github.com/django/django/commit/ac2052ebc84c45709ab5f0f25e685bf656ce79bc#diff-0) 来格式化新的组件。如果你喜欢，你可以修改它，但是目前我们不做任何修改。





## 编写文档

这个新功能信息应该被记录到文档。找到文件 `django/docs/ref/models/fields.txt` 第 925 行，在已存在的 `URLField` 文档条目下添加以下内容：





```
.. versionadded:: 1.5

    The current value of the field will be displayed as a clickable link above the
    input widget.

```





关于 `versionadded` 的解释以及文档编写的更多信息，请参考 [_文档编写_](../internals/contributing/writing-documentation.html)。 这个页面还介绍了怎么在本地重新生成一份文档，你可以查看新生成的 HTML 文档页面.





## 为你的修改生成补丁

现在是时候生成一个补丁文件，这个补丁文件可以上传到 Trac 或者更新到其他 Django。 运行下面这个命令来查看你的补丁内容：





```
git diff

```





这里显示的内容为当前代码与 check out 时候的代码变化，即之前对代码所做修改前后的变化。

在浏览补丁内容后按 `q` 键退出命令行。如果你的补丁内容看起来正常，运行下面这个命令，在当前目录生成补丁文件：





```
git diff > 17549.diff

```





在 Django 的根目录生成补丁文件 `17549.diff`。这个补丁文件包含所有的代码变动信息，看起来如下：





```
diff --git a/django/contrib/admin/widgets.py b/django/contrib/admin/widgets.py
index 1e0bc2d..9e43a10 100644
--- a/django/contrib/admin/widgets.py
+++ b/django/contrib/admin/widgets.py
@@ -10,7 +10,7 @@ from django.contrib.admin.templatetags.admin_static import static
 from django.core.urlresolvers import reverse
 from django.forms.widgets import RadioFieldRenderer
 from django.forms.util import flatatt
-from django.utils.html import escape, format_html, format_html_join
+from django.utils.html import escape, format_html, format_html_join, smart_urlquote
 from django.utils.text import Truncator
 from django.utils.translation import ugettext as _
 from django.utils.safestring import mark_safe
@@ -306,6 +306,18 @@ class AdminURLFieldWidget(forms.TextInput):
             final_attrs.update(attrs)
         super(AdminURLFieldWidget, self).__init__(attrs=final_attrs)

+    def render(self, name, value, attrs=None):
+        html = super(AdminURLFieldWidget, self).render(name, value, attrs)
+        if value:
+            value = force_text(self._format_value(value))
+            final_attrs = {'href': mark_safe(smart_urlquote(value))}
+            html = format_html(
+                '<p class="url">{} <a {}>{}</a><br />{} {}</p>',
+                _('Currently:'), flatatt(final_attrs), value,
+                _('Change:'), html
+            )
+        return html
+
 class AdminIntegerFieldWidget(forms.TextInput):
     class_name = 'vIntegerField'

diff --git a/docs/ref/models/fields.txt b/docs/ref/models/fields.txt
index 809d56e..d44f85f 100644
--- a/docs/ref/models/fields.txt
+++ b/docs/ref/models/fields.txt
@@ -922,6 +922,10 @@ Like all :class:`CharField` subclasses, :class:`URLField` takes the optional
 :attr:`~CharField.max_length`argument. If you don't specify
 :attr:`~CharField.max_length`, a default of 200 is used.

+.. versionadded:: 1.5
+
+The current value of the field will be displayed as a clickable link above the
+input widget.

 Relationship fields
 ===================
diff --git a/tests/regressiontests/admin_widgets/tests.py b/tests/regressiontests/admin_widgets/tests.py
index 4b11543..94acc6d 100644
--- a/tests/regressiontests/admin_widgets/tests.py
+++ b/tests/regressiontests/admin_widgets/tests.py

@@ -265,6 +265,35 @@ class AdminSplitDateTimeWidgetTest(DjangoTestCase):
                     '<p class="datetime">Datum: <input value="01.12.2007" type="text" class="vDateField" name="test_0" size="10" /><br />Zeit: <input value="09:30:00" type="text" class="vTimeField" name="test_1" size="8" /></p>',
                 )

+class AdminURLWidgetTest(DjangoTestCase):
+    def test_render(self):
+        w = widgets.AdminURLFieldWidget()
+        self.assertHTMLEqual(
+            conditional_escape(w.render('test', '')),
+            '<input class="vURLField" name="test" type="text" />'
+        )
+        self.assertHTMLEqual(
+            conditional_escape(w.render('test', 'http://example.com')),
+            '<p class="url">Currently:<a href="http://example.com">http://example.com</a><br />Change:<input class="vURLField" name="test" type="text" value="http://example.com" /></p>'
+        )
+
+    def test_render_idn(self):
+        w = widgets.AdminURLFieldWidget()
+        self.assertHTMLEqual(
+            conditional_escape(w.render('test', 'http://example-äüö.com')),
+            '<p class="url">Currently:<a href="http://xn--example--7za4pnc.com">http://example-äüö.com</a><br />Change:<input class="vURLField" name="test" type="text" value="http://example-äüö.com" /></p>'
+        )
+
+    def test_render_quoting(self):
+        w = widgets.AdminURLFieldWidget()
+        self.assertHTMLEqual(
+            conditional_escape(w.render('test', 'http://example.com/<sometag>some text</sometag>')),
+            '<p class="url">Currently:<a href="http://example.com/%3Csometag%3Esome%20text%3C/sometag%3E">http://example.com/&lt;sometag&gt;some text&lt;/sometag&gt;</a><br />Change:<input class="vURLField" name="test" type="text" value="http://example.com/<sometag>some text</sometag>" /></p>'
+        )
+        self.assertHTMLEqual(
+            conditional_escape(w.render('test', 'http://example-äüö.com/<sometag>some text</sometag>')),
+            '<p class="url">Currently:<a href="http://xn--example--7za4pnc.com/%3Csometag%3Esome%20text%3C/sometag%3E">http://example-äüö.com/&lt;sometag&gt;some text&lt;/sometag&gt;</a><br />Change:<input class="vURLField" name="test" type="text" value="http://example-äüö.com/<sometag>some text</sometag>" /></p>'
+        )

 class AdminFileWidgetTest(DjangoTestCase):
     def test_render(self):

```









## 接下来做什么？

恭喜，你已经生成了你的第一个 Django 补丁 ！现在你已经明白了整个过程，你可以好好利用这些技能帮助改善Django的代码库。 生成补丁和发送到 Trac 上是有用的，然而我们推荐使用 [_面向 git 的工作流_](../internals/contributing/writing-code/working-with-git.html)。

目前我们没有在本地对仓储做提交操作，我们可以通过下面这个命令放弃修改并回到最原始 Django 代码状态。





```
git reset --hard HEAD
git checkout master

```







### 关于新手贡献值的注意事项

在你开始为 Django 编写补丁时，这里有些信息，你应该看一看：

*   你应该阅读了 Django 的参考文档 [_claiming tickets and submitting patches_](../internals/contributing/writing-code/submitting-patches.html). 它涵盖了Trac 规则，如何声称自己的 tickets，补丁的编码风格和其他一些重要信息。
*   第一次提交补丁额外应该阅读  [_documentation for first time contributors_](../internals/contributing/new-contributors.html). 这里有很多对新手贡献值的建议。
*   接下来，如果你想对源码贡献有更深入了解，可以阅读接下来的 Django 文档 [_Django’s documentation on contributing_](../internals/contributing/index.html)。 它包含了大量的有用信息，这里可以解决你可能遇到的所有问题。





### 寻找你的第一个真正的标签

一旦你看过了之前那些信息，你便已经具备了走出困境，为自己编写补丁寻找门票的能力。对于那些有着“容易获得”标准的门票要尤其注意。这些门票实际上常常很简单而且对于第一次撰写补丁的人很有帮助。一旦你熟悉了给Django写补丁，你就可以进一步为更难且更复杂的门票写补丁。

如果你只是想要简单的了解(没人会因此责备你!), 那么你可以尝试着查看这个[需要补丁的简单标签](https://code.djangoproject.com/query?status=new&status=reopened&has_patch=0&easy=1&col=id&col=summary&col=status&col=owner&col=type&col=milestone&order=priority)列表和[已有补丁但需要提升的简单标签](https://code.djangoproject.com/query?status=new&status=reopened&needs_better_patch=1&easy=1&col=id&col=summary&col=status&col=owner&col=type&col=milestone&order=priority)列表. 如果你比较擅长写测试，那么你也可以看看这个 [需要测试的简单标签](https://code.djangoproject.com/query?status=new&status=reopened&needs_tests=1&easy=1&col=id&col=summary&col=status&col=owner&col=type&col=milestone&order=priority)列表. 一定要记得遵循在Django的文档[_声明标签和递交补丁_](../internals/contributing/writing-code/submitting-patches.html)中提到的关于声明标签的指导规则.





### 接下来要做什么？

一旦一个标签有了补丁，那么它就需要其他人来重审。上传了一个补丁或递交了一个pull request之后，一定记得更新标签的元数据，比如设置标签的标志状态为“has patch”，“doesn’t need tests”等。只有这样，其他人才能找到并重审这个标签。从零开始写补丁并不是做贡献的唯一方式。重审一些已经存在的补丁也是一种非常有用的做贡献方式。点击[_标签鉴别_](../internals/contributing/triaging-tickets.html) 查看更多详细信息.





