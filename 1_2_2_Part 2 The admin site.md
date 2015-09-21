{% raw %}

# 编写你的第一个 Django 程序 第2部分 #

本教程上接 教程 第1部分 。 我们将继续开发 Web-poll 应用，并且专注在 Django 的 自动生成的管理网站上。

> 哲理
>
> 为你的员工或客户生成添加、修改和删除内容的管理性网站是个单调乏味的工作。 出于这个原因，Django 根据模型完全自动化创建管理界面。
>
> Django 是在新闻编辑室环境下编写的，“内容发表者”和“公共”网站之间有 非常明显的界线。网站管理员使用这个系统来添加新闻、事件、体育成绩等等， 而这些内容会在公共网站上显示出来。Django 解决了为网站管理员创建统一 的管理界面用以编辑内容的问题。
>
> 管理界面不是让网站访问者使用的。它是为网站管理员准备的。

## 启用管理网站 ##

+ 默认情况下 Django 管理网站是不启用的 – 它是可选的。 要启用管理网站，需要做三件事：
+ 在 INSTALLED_APPS 设置中取消 "django.contrib.admin" 的注释。
+ 运行 python manage.py syncdb 命令。既然你添加了新应用到 INSTALLED_APPS 中，数据库表就需要更新。
+ 编辑你的 mysite/urls.py 文件并且将有关管理的行取消注释 – 共有三行取消了注释。该文件是 URLconf ；我们将在下一个教程中深入探讨 URLconfs 。现在，你需要知道的是它将 URL 映射到应用。最后你拥有的 urls.py 文件看起来像这样:

```
from django.conf.urls import patterns, include, url

# Uncomment the next two lines to enable the admin:
from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    # Examples:
    # url(r'^$', '{{ project_name }}.views.home', name='home'),
    # url(r'^{{ project_name }}/', include('{{ project_name }}.foo.urls')),

    # Uncomment the admin/doc line below to enable admin documentation:
    # url(r'^admin/doc/', include('django.contrib.admindocs.urls')),

    # Uncomment the next line to enable the admin:
    url(r'^admin/', include(admin.site.urls)),
)
```

( 粗体显示的行就是那些需要取消注释的行。)

## 启动开发服务器 ##

让我们启动开发服务器并浏览管理网站。

回想下教程的第一部分，像如下所示启动你的开发服务器：

```
python manage.py runserver
```

现在，打开一个浏览器并在本地域名上访问 “/admin/” – 例如 http://127.0.0.1:8000/admin/ 。你将看到管理员的登录界面：

![](https://docs.djangoproject.com/en/1.8/_images/admin01.png)

> 和你看到的不一样？
>
> 如果看到这，而不是上面的登录界面，那你应该得到一个类似如下所示的错误页面报告：
>
```
ImportError at /admin/ cannot import name patterns ...
```
>
> 那么你很可能使用的 Django 版本不符合本教程的版本。 你可以切换到对应的旧版本教程去或者更新到较新的 Django 版本。

## 进入管理网站 ##

现在尝试登录进去。（还记得吗？在本教程的第一部分时你创建过一个超级用户的帐号。如果你没有创建或忘记了密码，你可以 另外创建一个 。） 你将看到 Djaong 的管理索引页：

![](https://docs.djangoproject.com/en/1.8/_images/admin02.png)

你将看到一些可编辑的内容，包括 groups ，users 和 sites 。这些都是 Django 默认情况下自带的核心功能。

## 使 poll 应用的数据在管理网站中可编辑 ##

但是 poll 应用在哪？ 它可是没有在管理网站的首页上显示啊。

只需要做一件事：我们需要告诉管理网站 Poll 对象要有一个管理界面。为此，我们在你的 polls 目录下创建一个名为 admin.py 的文件，并添加如下内容：:

```
from django.contrib import admin
from polls.models import Poll
admin.site.register(Poll)
```

你需要重启开发服务器才能看到变化。通常情况下，你每次修改过一个文件后开发 服务器都会自动载入，但是创建一个新文件却不会触发自动载入的逻辑。

## 探索管理功能 ##

现在我们已经注册了 Poll ，那 Django 就知道了要在管理网站的首页上显示出来：

![](https://docs.djangoproject.com/en/1.8/_images/admin03t.png)

点击 “Polls” 。现在你在 polls 的 “更改列表” 页。该页 显示了数据库中所有的 polls 可让你选中一个进行编辑。 有个 “What’s up?” poll 是我们在第一个教程中创建的：

![](https://docs.djangoproject.com/en/1.8/_images/admin04t.png)

点击这个”What’s up?” 的 poll 进行编辑：

![](https://docs.djangoproject.com/en/1.8/_images/admin05t.png)

这有些注意事项：

+ 这的表单是根据 Poll 模型自动生成的。
+ 不同模型的字段类型 (DateTimeField, CharField) 会对应的相应的 HTML 输入控件。 每一种类型的字段 Djaong 管理网站都知道如何显示它们。
+ 每个 DateTimeField 都会有个方便的 JavaScript 快捷方式。日期有一个 “Today” 快捷方式和弹出式日历，而时间有个 “Now” 快捷方式和一个列出了常用时间选项的弹出式窗口。

在页面的底部还为你提供了几个选项：

+ Save – 保存更改并返回到当前类型的对象的更改列表页面。
+ Save and continue editing – 保存更改并重新载入当前对象的管理界面。
+ Save and add another – 保存更改并载入当前对象类型的新的空白表单。
+ Delete – 显示删除确认页。

如果 “Date published” 的值与你在第一部分教程时创建的 poll 的时间不符，这可能 意味着你忘记了将 TIME_ZONE 设置成正确的值了。修改正确后再重启载入页面 来检查值是否正确。

分别点击 “Today” 和 “Now” 快捷方式来修改 “Date published” 的值。 然后点击 “Save and continue editing” 。最后点击右上角的 “History” 。 你将看到一页列出了通过 Django 管理界面对此对象所做的全部更改的清单的页面， 包含有时间戳和修改人的姓名等信息：

![](https://docs.djangoproject.com/en/1.8/_images/admin06t.png)

## 自定义管理表单 ##

花些时间感叹一下吧，你没写什么代码就拥有了这一切。通过 admin.site.register(Poll) 注册了 Poll 模型，Django 就能构造一个默认的 表单。通常情况下，你将要自定义管理表单的外观和功能。这样的话你就需要在注册对象 时告诉 Django 对应的配置。

让我们来看看如何在编辑表单上给字段重新排序。将 admin.site.register(Poll) 这行替换成：:

```
class PollAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question']

admin.site.register(Poll, PollAdmin)
```

你将遵循这个模式 – 创建一个模型的管理对象，将它作为 admin.site.register() 方法的第二个参数传入 – 当你需要为一个对象做管理界面配置的时候。

上面那特定的更改使得 “Publication date” 字段在 “Question” 字段之前:

![](https://docs.djangoproject.com/en/1.8/_images/admin07.png)

仅有两个字段不会令你印象深刻，但是对于有许多字段的管理表单时，选择一个直观 的排序方式是一个重要的实用细节。

刚才所说的有许多字段的表单，你可能想将表单中的字段分割成 fieldsets ：:

```
class PollAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question']}),
        ('Date information', {'fields': ['pub_date']}),
    ]

admin.site.register(Poll, PollAdmin)
```

在 fieldsets 中每一个 tuple 的第一个元素就是 fieldset 的标题。 下面是我们表单现在的样子：

![](https://docs.djangoproject.com/en/1.8/_images/admin08t.png)

你可以为每个 fieldset 指定 THML 样式类。Django 提供了一个 "collapse" 样式类用于显示初始时是收缩的 fieldset 。 当你有一个包含一些不常用的长窗体时这是非常有用的

```
class PollAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
Fieldset is initially collapsed
```

## 添加关联对象 ##

Ok，现在我们有了 Poll 的管理页面。但是一个 Poll 拥有多个 Choices ，而 该管理页面并没有显示对应的 choices 。

是的。

我们有两种方法来解决这个问题。第一种就像刚才 Poll 那样在管理网站上 注册 Choice 。这很简单：

```
from polls.models import Choice

admin.site.register(Choice)
```

现在 “Choices” 在 Django 管理网站上是一个可用的选项了。”Add choice” 表单 看起来像这样：

![](https://docs.djangoproject.com/en/1.8/_images/admin10.png)

该表单中，``Poll`` 字段是一个包含了数据库中每个 poll 的选择框。 Django 知道 ForeignKey 在管理网站中以 `<select>` 框显示。在本例中，选择框中仅存在一个 poll 。

另外请注意 Poll 旁边的 “Add Another” 链接。每个有 ForeignKey 的对象关联到其他对象都会得到这个链接。 当点击 “Add Another” 时，你将会获得一个 “Add poll” 表单的弹出窗口。 如果你在窗口中添加了一 poll 并点击了 “Save” 按钮， Django 会将 poll 保存至数据库中并且动态的添加为你正在查看的 “Add choice” 表单中的 已选择项。

但是，这真是一个低效的将 Choice 对象添加进系统的方式。 如果在创建 Poll 对象时能够直接添加一批 Choices 那会更好。 让我们这样做吧。

移除对 Choice 模型的 register() 方法调用 。然后，将 Poll 的注册代码 编辑为如下所示：

```
from django.contrib import admin
from polls.models import Choice, Poll

class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 3

class PollAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    inlines = [ChoiceInline]

admin.site.register(Poll, PollAdmin)
```

这将告诉 Django: “Choice 对象在 Poll 管理页面中被编辑。 默认情况下，提供 3 个 choices 的字段空间。

载入 “Add poll” 页面来看看，你可能需要重启你的开发服务器：

![](https://docs.djangoproject.com/en/1.8/_images/admin11t.png)

它看起来像这样：多了三个为关联 Choices 提供的输入插槽 – 由 extra 指定 – 并且每次你在 “Change” 页修改已经创建的对象时，都会另外获得三个额外插槽。

在现有的三个插槽的底部，你会发现一个 “Add another Choice” 链接。 如果你点击它，一个新的插槽会被添加。如果想移除添加的插槽， 你可以点击所添加的插槽的右上方的 X 。注意你不能移除原有的三个插槽。 此图片中显示了新增的插槽：

![](https://docs.djangoproject.com/en/1.8/_images/admin15t.png)

还有个小问题。为了显示所有关联 Choice 对象的字段需要占用大量的 屏幕空间。为此，Django 提供了一个以表格方式显示内嵌有关联对象的方式； 你只需要将 ChoiceInline 声明改为如下所示：

```
class ChoiceInline(admin.TabularInline):
    #...
```

使用了 TabularInline 后(而不是 StackedInline) ，基于表的格式下相关 对象被显示的更紧凑了：

![](https://docs.djangoproject.com/en/1.8/_images/admin12t.png)


需要注意的是有个额外的 “Delete?” 列允许保存时移除已保存过的行。

## 自定义管理界面的变更列表 ##

现在 Poll 的管理界面看起来不错了，让我们给 “chang list” 页面做些调整 – 显示系统中所有 polls 的页面。

下面是现在的样子：

![](https://docs.djangoproject.com/en/1.8/_images/admin04t.png)

默认情况下， Django 显示的是每个对象 str() 的结果。但是若是我们能够 显示每个字段的话有时会更有帮助的。要做到这一点，需要使用 list_display 管理选项，这是一个 tuple ，包含了要显示的字段名， 将会以列的形式在该对象的 chang lsit 页上列出来：:

```
class PollAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question', 'pub_date')
```

效果再好的点话，让我们把在第一部分教程中自定义的方法 was_published_recently 也包括进来:

```
class PollAdmin(admin.ModelAdmin):
    # ...
    list_display = ('question', 'pub_date', 'was_published_recently')
```

现在 poll 的变更列表页看起来像这样：

![](https://docs.djangoproject.com/en/1.8/_images/admin13t.png)

你可以点击列的标题对这些值进行排序 – 除了 was_published_recently 这一列，因为不支持根据方法输出的内容的排序。还要注意的是默认情况下列的标题是 was_published_recently ，就是方法名（将下划线替换为空格），并且每一行以字符串形式输出。

你可以通过给该方法 （在 models.py 内 ） 添加一些属性来改善显示效果，如下所示：:

```
class Poll(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
    was_published_recently.admin_order_field = 'pub_date'
    was_published_recently.boolean = True
    was_published_recently.short_description = 'Published recently?'
```

再次编辑你的 admin.py 文件并添加一个改进 Poll 的 change list 页面效果的功能： 筛选 ( Filters ) 。在 PollAdmin 内添加一行如下所示的代码：:

```
list_filter = ['pub_date']
```

这就增加了一个 “筛选” 的侧边栏，让人们通过 pub_date 字段的值来筛选 change list 显示的内容：

![](https://docs.djangoproject.com/en/1.8/_images/admin14t.png)

显示筛选的类型取决于你需要筛选的字段类型。 因为 pub_date 是一个 DateTimeField 的实例，Django 知道提供对应的筛选选项：”Any date,” “Today,” “Past 7 days,” “This month,” “This year.”

为了效果更好。让我们来加上搜索功能：:

```
search_fields = ['question']
```

在 chang list 页的顶部增加了一个搜索框。当有人输入了搜索条件， Django 将搜索 question 字段。 虽然你可以使用任意数量的字段，如你希望的那样 – 但是因为它在后台用 LIKE 查询，为了保持数据库的性能请合理使用。

最后，因为 Poll 对象有日期字段，根据日期来向下钻取记录将会很方便。 添加下面这一行代码：:

```
date_hierarchy = 'pub_date'
```

这会在 change list 页的顶部增加了基于日期的分层导航功能。 在最顶层，显示所有可用年份。然后可钻取到月份，最终到天。

现在又是一个好时机，请注意 change lists 页面提供了分页功能。默认情况下每一页显示 100 条记录。 Change-list 分页，搜索框，筛选，日期分层和列标题排序如你所原地在一起运行了。

## 自定义管理界面的外观 ##

显而易见，在每一个管理页面顶部有 “Django administration” 是无语的。虽然它仅仅是个占位符。

不过使用 Django 的模板系统是很容易改变的。Django 管理网站有 Django 框架自身的功能，可以通过 Django 自身的模板系统来修改界面。

### 自定义你的 项目 模板 ###

在你的项目目录下创建一个 templates 目录。模板可以放在你的文件系统的任何地方，Diango 都能访问。 (Django 能以任何用户身份在你的服务器上运行。) 然后，在你的项目中保存模板是一个好习惯。

默认情况下，TEMPLATE_DIRS 值是空的。因此，让我们添加一行代码，来告诉 Django 我们的模板在哪里：:

```
TEMPLATE_DIRS = (
    '/path/to/mysite/templates', # 将此处改为你的目录。
)
```

现在从 Django 源代码中自带的默认 Django 管理模板的目录 (django/contrib/admin/templates) 下复制 admin/base_site.html 模板到你正在使用的 TEMPLATE_DIRS 中任何目录的子目录 admin 下。例如：如果你的 TEMPLATE_DIRS 中包含 '/path/to/mysite/templates' 目录， 如上所述，复制 django/contrib/admin/templates/admin/base_site.html 模板到 /path/to/mysite/templates/admin/base_site.html 。不要忘了是 admin 子目录。

> Django 的源代码在哪里？
>
> 如果在你的文件系统中很难找到 Django 源代码，可以运行如下命令：
>
```
python -c "
import sys
sys.path = sys.path[1:]
import django
print(django.__path__)"
```

然后，只需要编辑该文件并将通用的 Djangot 文字替换为你认为适合的属于你自己的网站名。

该模板包含了大量的文字，比如 `{% block branding %}` 和 `{{ title }}`。`{%` 和 `{{` 标记是 Django 模板语言的一部分。 当 Django 呈现 admin/base_site.html 时，根据模板语言生成最终的 HTML 页面。 Don’t worry if you can’t make any sense of the template right now – 如果你现在不能理解模板的含义先不用担心 – 我们将在教程 3 中深入探讨 Django’ 的模板语言。

请注意 Django 默认的管理网站中的任何模板都是可覆盖的。 要覆盖一个模板，只需要像刚才处理 base_site.html 一样 – 从默认的目录下复制到你的自定义目录下，并修改它。

### 自定义你的 应用 模板 ###

细心的读者会问：如果 TEMPLATE_DIRS 默认的情况下是空值， 那 Django 是如何找到默认的管理网站的模板的？ 答案就是在默认情况下， Django 会自动在每一个应用的包内查找 templates/ 目录，作为备用使用。 （不要忘记 django.contrib.admin 是一个应用）。

我们的 poll 应用不是很复杂并不需要自定义管理模板。但是如果它变得更复杂 而且为了一些功能需要修改 Django 的标准管理模板，修改*应用*模板将是更 明智的选择，而不是修改*项目*模板。通过这种方式，你可以在任何新项目包括 polls 应用中自定义模板并且放心会找到需要的自定义的模板的。

有关 Django 怎样找到它的模板的更多信息，请参考 模板加载文档 。

## 自定义管理网站的首页 ##

于此类似，你可能还想自定义 Django 管理网站的首页。

默认情况下，首页会显示在 INSTALLED_APPS 中所有注册了管理功能的应用， 并按字母排序。你可能想在页面布局上做大修改。总之，首页可能是管理网站中最重要的页面， 因此它应该很容易使用。

你需要自定义的模板是 admin/index.html 。 （同先前处理 admin/base_site.html 一样 – 从默认目录下复制到你自定义的模板目录下。） 编辑这个文件，你将看到一个名为 app_list 的模板变量。这个变量包含了每一个 已安装的 Django 应用。你可以通过你认为最好的方法硬编码链接到特定对象的管理页面，而不是使用默认模板。 再次强调，如果你不能理解模板语言的话不用担心 – 我们将在教程 3 中详细介绍。

当你熟悉了管理网站的功能后，阅读 教程 第3部分 开始开发公共 poll 界面。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Part 2: The admin site](https://docs.djangoproject.com/en/1.8/intro/tutorial02/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。

{% endraw %}
