# 部署 Django #

虽然Django 满满的便捷性让Web 开发人员活得轻松一些，但是如果不能轻松地部署你的网站，这些工具还是没有什么用处。Django 起初，易于部署就是一个主要的目标。有许多优秀的方法可以轻松地来部署Django：

+ [如何使用WSGI 部署](http://python.usyiyi.cn/django/howto/deployment/wsgi/index.html)
+ [部署的检查清单](http://python.usyiyi.cn/django/howto/deployment/checklist.html)

FastCGI  的支持已经废弃并将在Django 1.9 中删除。

+ [如何使用FastCGI、SCGI 和AJP 部署Django](http://python.usyiyi.cn/django/howto/deployment/fastcgi.html)

如果你是部署Django 和/或 Python 的新手，我们建议你先试试 [mod_wsgi](http://python.usyiyi.cn/django/howto/deployment/wsgi/modwsgi.html)。 在大部分情况下，这将是最简单、最迅速和最稳当的部署选择。

> 另见
>
> [Django Book（第二版）的第12 章](http://www.djangobook.com/en/2.0/chapter12.html) 更详细地讨论了部署，尤其是可扩展性。但是请注意，这个版本是基于Django 1.1 版本编写，而且在`mod_python` 废弃并于Django 1.5 中删除之后一直没有更新。

&zwj;

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Overview](https://docs.djangoproject.com/en/1.8/howto/deployment/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
