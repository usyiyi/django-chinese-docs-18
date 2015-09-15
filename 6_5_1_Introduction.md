# Django中的测试 #

自动化测试对于现代web开发者来说，是非常实用的除错工具。你可以使用一系列测试-- 测试套件 -- 来解决或者避免大量问题：

+ 当你编写新代码的时候，你可以使用测试来验证你的代码是否像预期一样工作。
+ 当你重构或者修改旧代码的时候，你可以使用测试来确保你的修改不会在意料之外影响到你的应用的应为。

测试web应用是个复杂的任务，因为web应用由很多的逻辑层组成 -- 从HTTP层面的请求处理，到表单验证和处理，到模板渲染。使用Django的测试执行框架和各种各样的工具，你可以模拟请求，插入测试数据，检查你的应用的输出，以及大体上检查你的代码是否做了它应该做的事情。

最好的一点是，它非常简单。

在Django中编写测试的最佳方法是，使用构建于Python标准库的unittest模块。这在[编写和运行测试](http://python.usyiyi.cn/django/topics/testing/overview.html) 文档中会详细介绍。

你也可以使用任何其它 Python 的测试框架；Django为整合它们提供了API和工具。这在[高级测试话题](http://python.usyiyi.cn/django/topics/testing/advanced.html)的[使用不同的测试框架](http://python.usyiyi.cn/django/topics/testing/advanced.html#other-testing-frameworks) 一节中描述。

+ [编写和运行测试](http://python.usyiyi.cn/django/topics/testing/overview.html)
+ [测试工具](http://python.usyiyi.cn/django/topics/testing/tools.html)
+ [高级测试话题](http://python.usyiyi.cn/django/topics/testing/advanced.html)

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Introduction](https://docs.djangoproject.com/en/1.8/topics/testing/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
