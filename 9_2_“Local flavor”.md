# "本地特色"附加功能

由于历史因素，Django自带了`django.contrib.localflavor` -- 各种各样的代码片段，有助于在特定的国家地区或文化中使用。为了便于维护以及减少Django代码库的体积，这些代码现在在Django之外单独发布。

详见官方文档：

> [https://django-localflavor.readthedocs.org/](https://django-localflavor.readthedocs.org/)

这些代码托管在GIthub上面，[https://github.com/django/django-localflavor](https://github.com/django/django-localflavor)。

## 如何迁移

如果你使用了老版本的`django.contrib.localflavor`包，或者 `django-localflavor-*` 的模板之一，执行这两个简单的步骤就可以更新你的代码：

*   在PyPI中安装第三方的`django-localflavor` 包。

*  修改你应用的导入语句来引用新的包。

    例如，将：

    ```
    from django.contrib.localflavor.fr.forms import FRPhoneNumberField

    ```

    ...改为：

    ```
    from localflavor.fr.forms import FRPhoneNumberField

    ```

新的包中的代码和以前一样(它是直接从Django中复制出来的)，所以你并不用担心功能上的向后兼容问题。只需要修改导入语句。

## 弃用政策

在 Django 1.5中，导入`django.contrib.localflavor`会产生 `DeprecationWarning`异常。也就是说你的代码还可以继续工作，但是你应该尽快修改它。

在Django 1.6中，导入 `django.contrib.localflavor`将不会继续工作。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[“Local flavor”](https://docs.djangoproject.com/en/1.8/topics/localflavor/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
