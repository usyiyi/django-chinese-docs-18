# 快速安装指南 #

在你开始使用 Django 之前，你需要先安装它。我们有一个 *完整安装指南* 它涵盖了所有的安装步骤和可能遇到的问题；本指南将会给你一个最简单、简洁的安装指引。

## 安装 Python ##

作为一个 Web 框架，Django 需要使用 Python 。它适用 2.6.5 到 2.7 的所有 Python 版本。它还具有 3.2 和 3.3 版本的实验性支持。所有这些 Python 版本都包含一个轻量级的数据库名叫 SQLite 。因此你现在还不需要建立一个数据库。

在 http://www.python.org 获取 Python 。如果你使用 Linux 或者 Mac OS X，那很可能已经安装了 Python 。

> **在 Jython 使用 Django**
>
> 如果你使用 *Jython* (一个在 Java 平台上实现的 Python )，你需要遵循一些额外的步骤。查看 *在 Jyton 上运行 Python* 获取详细信息。
>

在你的终端命令行(shell)下输入 python 来验证是否已经安装 Python ; 你将看到如下类似的提示信息:

```
Python 3.3.3 (default, Nov 26 2013, 13:33:18)
[GCC 4.8.2] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

## 建立一个数据库 ##

若你需要一个“大”数据库引擎，例如：PostgreSQL ，MySQL ，或 Oracle ，那此步骤是需要的。 想要安装这样一个数据库，请参考 数据库安装信息.

## 删除旧版本的 Django ##

如果你是从旧版本的 Django 升级安装，你将需要 *在安装新版本之前先卸载旧版本的 Django*.

## 安装 Django ##

你可以使用下面这简单的三个方式来安装 Django:

+ 安装 *你的操作系统所提供供的发行包* 。对于操作系统提供了 Django 安装包的人来说，这是最快捷的安装方法。
+ *安装官方正式发布的版本* 。这是对于想要安装一个稳定版本而不介意运行一个稍旧版本的 Django 的人来说是最好的方式。
+ *安装最新的开发版本* 。这对于那些想要尝试最新最棒的特性而不担心运行崭新代码的用户来说是最好的。

> **总是参考你所使用的对应版本的 Django 文档！**
>
> 如果采用了前两种方式进行安装，你需要注意在文档中标明**在开发版中新增**的标记。这个标记表明这个特性仅适用开发版的 Django ，而他们可能不在官方版本发布。

## 验证安装 ##

为了验证 Django 被成功的安装到 Python 中，在你的终端命令行 (shell) 下输入 python 。 然后在 Python 提示符下，尝试导入 Django:

```
>>> import django
>>> print(django.get_version())
1.8
```

你可能已安装了其他版本的 Django 。

## 安装完成！ ##

安装完成 – 现在你可以 *学习入门教程*.

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Installation](https://docs.djangoproject.com/en/1.8/intro/install/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
