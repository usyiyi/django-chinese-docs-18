# Django 中的用户认证 #

Django从开始就带有一个用户认证系统。它处理用户账号、组、权限以及基于cookie的用户会话。本节文档解释默认的实现如何直接使用，以及如何[扩展和定制](http://python.usyiyi.cn/django/topics/auth/customizing.html)它以适合你项目的需要。

## 概览 ##

Django认证系统同时处理认证和授权。简单地讲，认证验证一个用户是它们声称的那个人，授权决定一个认证通过的用户允许做什么。这里的词语认证同时指代这两项任务。

认证系统包含：

+ 用户
+ 权限：二元（是/否）标志指示一个用户是否可以做一个特定的任务。
+ 组：对多个用户运用标签和权限的一种通用的方式。
+ 一个可配置的密码哈希系统
+ 用于登录用户或限制内容的表单和视图
+ 一个可插拔的后台系统

Django中的认证系统的目标是非常通用且不提供在web认证系统中某些常见的功能。某些常见问题的解决方法已经在第三方包中实现：

+ 密码强度检查
+ 登录尝试的制约
+ 第三方认证（例如OAuth）

## 安装 ##

认证的支持作为Django的一个contrib模块，打包于`django.contrib.auth`中。默认情况下，要求的配置已经包含在`django-admin startproject`生成的`settings.py`中，它们的组成包括`INSTALLED_APPS`设置中的两个选项：

1. '`django.contrib.auth`'包含认证框架的核心和默认的模型。
2. '`django.contrib.contenttypes`'是Django内容类型系统，它允许权限与你创建的模型关联。
和`MIDDLEWARE_CLASSES`设置中的两个选项：

1. `SessionMiddleware`管理请求之间的会话。
2. `AuthenticationMiddleware`使用会话将用户与请求管理起来。

有了这些设置，运行`manage.py migrate`命令将为认证相关的模型创建必要的数据库表并为你的应用中定义的任意模型创建权限。

## 使用 ##

[使用Django默认的实现](http://python.usyiyi.cn/django/topics/auth/default.html)

+ [使用User对象](http://python.usyiyi.cn/django/topics/auth/default.html#user-objects)
+ [权限和授权](http://python.usyiyi.cn/django/topics/auth/default.html#topic-authorization)
+ [Web 请求中的认证](http://python.usyiyi.cn/django/topics/auth/default.html#auth-web-requests)
+ [ 在admin 中管理用户](http://python.usyiyi.cn/django/topics/auth/default.html#auth-admin)

[默认实现的API参考](http://python.usyiyi.cn/django/ref/contrib/auth.html)

[自定义Users和认证](http://python.usyiyi.cn/django/topics/auth/customizing.html)

[Django中的密码管理](http://python.usyiyi.cn/django/topics/auth/passwords.html)

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Overview](https://docs.djangoproject.com/en/1.8/topics/auth/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
