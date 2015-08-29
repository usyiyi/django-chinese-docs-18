# 高级教程：如何编写可重用的应用 #

本高级教程上接教程 6。我们将把我们的网页投票转换成一个独立的Python包，这样你可以在其它项目中重用或者分享给其它人。

如果你最近没有完成教程1–6，我们建议你阅读它们使得你的示例项目与下面描述的相匹配。

## 可重用很重要 ##

设计、构建、测试和维护一个网页应用有许多工作要做。许多Python 和 Django 项目都有常见的共同问题。如果我们可以节省一些这些重复的工作会不会很棒？

可重用性是Python 中一种生活的态度。Python包索引 (PyPI) 具有广泛的包，你可以在你自己的Python程序中使用。调查一下Django Packages中已经存在的可重用的应用，你可以结合它们到你的项目。Django 自身也只是一个Python 包。这意味着你可以获取已经存在的Python包和Django应用并将它们融合到你自己的网页项目。你只需要编写你项目的独特的部分。

比如说，你正在开始一个新的项目，需要一个像我们正在编写的投票应用。你如何让该应用可重用？幸运的是，你已经在正确的道路上。在教程 3中，我们看到我们可以如何使用include将投票应用从项目级别的URLconf 解耦。在本教程中，我们将更进一步，让你的应用在新的项目中容易地使用并随时可以发布给其它人安装和使用。

> 包？应用？
>
> Python 包 提供的方式是分组相关的Python 代码以容易地重用。一个包包含一个或多个Python代码（也叫做“模块”）。
>
> 包可以通过import foo.bar 或from foo import bar 导入。如果一个目录（例如polls）想要形成一个包，它必须包含一个特殊的文件__init__.py，即使这个文件为空。
>
> 一个Django 应用 只是一个Python包，它特意用于Django项目中。一个应用可以使用常见的Django 约定，例如具有models、tests、urls和views 子模块。
>
> 后面我们使用打包这个词来描述将一个Python包变得让其他人易于安装的过程。我们知道，这可能有点绕人。

## 你的项目和你的可重用的应用 ##

经过前面的教程之后，我们的项目应该看上去像这样：

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
    polls/
        __init__.py
        admin.py
        migrations/
            __init__.py
            0001_initial.py
        models.py
        static/
            polls/
                images/
                    background.gif
                style.css
        templates/
            polls/
                detail.html
                index.html
                results.html
        tests.py
        urls.py
        views.py
    templates/
        admin/
            base_site.html
```

你在教程 2中创建了mysite/templates ，在教程 3中创建了polls/templates。 现在你可能更加清晰为什么我们为项目和应用选择单独的模板目录：属于投票应用的部分全部在polls中。它使得该应用自包含且更加容易丢到一个新的项目中。

现在可以拷贝polls目录到一个新的Django项目并立即使用。然后它还不能充分准备好到可以立即发布。由于这点，我们需要打包这个应用来让它对其他人易于安装。

## 安装一些前提条件 ##

Python 打包的目前状态因为有多种工具而混乱不堪。对于本教程，我们打算使用setuptools来构建我们的包。它是推荐的打包工具（已经与distribute 分支合并）。我们还将使用pip来安装和卸载它。现在你应该安装这两个包。如果你需要帮助，你可以参考如何使用pip安装Django。你可以使用同样的方法安装setuptools。

## 打包你的应用 ##

Python packaging refers to preparing your app in a specific format that can be easily installed and used. Django 自己是以非常相似的方式打包起来的。对于一个像polls这样的小应用，这个过程不是太难。

首先，在你的Django项目之外，为polls创建一个父目录。称这个目录为django-polls。

> 为你的应用选择一个名字
>
> 让为你的包选择一个名字时，检查一下PyPI中的资源以避免与已经存在的包有名字冲突。当创建一个要发布的包时，在你的模块名字前面加上django-通常很有用。 这有助于其他正在查找Django应用的人区分你的应用是专门用于Django的。
>
> 应用的标签（应用的包的点分路径的最后部分）在INSTALLED_APPS中必须唯一。避免使用与Django的contrib 包 中任何一个使用相同的标签，例如auth、admin和messages。

将polls 目录移动到django-polls目录。

创建一个包含一些内容的文件django-polls/README.rst：

```
django-polls/README.rst
=====
Polls
=====

Polls is a simple Django app to conduct Web-based polls. For each
question, visitors can choose between a fixed number of answers.

Detailed documentation is in the "docs" directory.

Quick start
-----------

1. Add "polls" to your INSTALLED_APPS setting like this::

    INSTALLED_APPS = (
        ...
        'polls',
    )

2. Include the polls URLconf in your project urls.py like this::

    url(r'^polls/', include('polls.urls')),

3. Run `python manage.py migrate` to create the polls models.

4. Start the development server and visit http://127.0.0.1:8000/admin/
   to create a poll (you'll need the Admin app enabled).

5. Visit http://127.0.0.1:8000/polls/ to participate in the poll.
```

创建一个django-polls/LICENSE文件。选择License超出本教程的范围，但值得一说的是公开发布的代码如果没有License是毫无用处的。Django和许多与Django兼容的应用以BSD License 发布；然而，你可以随便挑选自己的License。只需要知道你的License的选则将影响谁能够使用你的代码。

下一步我们将创建一个setup.py 文件，它提供如何构建和安装该应用的详细信息。该文件完整的解释超出本教程的范围，setuptools 文档 有很好的解释。创建一个文件django-polls/setup.py，其内容如下：

```
django-polls/setup.py
import os
from setuptools import setup

with open(os.path.join(os.path.dirname(__file__), 'README.rst')) as readme:
    README = readme.read()

# allow setup.py to be run from any path
os.chdir(os.path.normpath(os.path.join(os.path.abspath(__file__), os.pardir)))

setup(
    name='django-polls',
    version='0.1',
    packages=['polls'],
    include_package_data=True,
    license='BSD License',  # example license
    description='A simple Django app to conduct Web-based polls.',
    long_description=README,
    url='http://www.example.com/',
    author='Your Name',
    author_email='yourname@example.com',
    classifiers=[
        'Environment :: Web Environment',
        'Framework :: Django',
        'Intended Audience :: Developers',
        'License :: OSI Approved :: BSD License', # example license
        'Operating System :: OS Independent',
        'Programming Language :: Python',
        # Replace these appropriately if you are stuck on Python 2.
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3.2',
        'Programming Language :: Python :: 3.3',
        'Topic :: Internet :: WWW/HTTP',
        'Topic :: Internet :: WWW/HTTP :: Dynamic Content',
    ],
)
```

默认只有Python模块和包会包含进包中。如果需要包含额外的文件，我们需要创建一个MANIFEST.in文件。上一步提到的setuptools 文档对这个文件有更详细的讨论。如果要包含模板、README.rst和我们的LICENSE 文件，创建一个文件django-polls/MANIFEST.in，其内容如下：

```
django-polls/MANIFEST.in
include LICENSE
include README.rst
recursive-include polls/static *
recursive-include polls/templates *
```

将详细的文档包含进你的应用中，它是可选的，但建议你这样做。创建一个空的目录django-polls/docs用于将来存放文档。向django-polls/MANIFEST.in添加另外一行：

```
recursive-include docs *
```

注意docs不会包含进你的包中除非你添加一些文件到它下面。许多Django应用还通过类似readthedocs.org这样的站点提供它们的在线文档.

试着通过python setup.py sdist 构建你的包（从django-polls的内部运行）。这创建一个dist目录并构建一个新包django-polls-0.1.tar.gz。

更多关于打包的信息，参见Python 的 打包和分发项目的教程。

## 使用你自己的包 ##

因为，我们将polls 目录移到项目的目录之外，它不再工作了。我们将通过安装我们的新的django-polls包来修复它。

> 安装成某个用户的库
>
> 以下的步骤将安装django-polls 成某个用户的库。根据用户安装相比系统范围的安装具有许多优点，例如用于没有管理员权限的系统上以及防止你的包影响系统的服务和机器上的其它用户。
>
> 注意根据用户的安装仍然可以影响以该用户身份运行的系统工具，所以virtualenv 是更健壮的解决办法（见下文）。

安装这个包，使用pip（你已经安装好它了，对吧？）：

```
pip install --user django-polls/dist/django-polls-0.1.tar.gz
```

如果幸运，你的Django 项目现在应该可以再次正确工作。请重新运行服务器以证实这点。

若要卸载这个包，使用pip：

```
pip uninstall django-polls
```

## 发布你的应用： ##

既然我们已经打包并测试过django-polls，是时候与世界共享它了！要不是它仅仅是个例子，你现在可以：

+ 将这个包用邮件发送给朋友。
+ 上传这个包到你的网站上。
+ 上传这个包到一个公开的仓库，例如Python 包索引 (PyPI)。packaging.python.org has a good tutorial for doing this.

## 使用 virtualenv 安装Python 包 ##

前面，我们将poll 安装成一个用户的库。它有一些缺点：

+ 修改这个用户的库可能影响你的系统上的其它Python 软件。
+ 你将不可以运行这个包的多个版本（或者具有相同名字的其它包）。

特别是一旦你维护几个Django项目，这些情况就会出现。如果确实出现，最好的解决办法是使用virtualenv。这个工具允许你维护多个分离的Python环境，每个都具有它自己的库和包的命名空间。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[How to write reusable apps](https://docs.djangoproject.com/en/1.8/intro/reusable-apps/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
