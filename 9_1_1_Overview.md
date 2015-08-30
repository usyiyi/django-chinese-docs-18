# 国际化和本地化 #

## 概述 ##

国际化和本地化的目的就是让一个网站应用能做到根据用户语种和指定格式的不同而提供不同的内容。

Django 对文本翻译, 日期、时间和数字的格式化，以及时区提供了完善的支持。

实际上，Django做了两件事：

+ 由开发者和模板作者指定应用的哪些部分应该翻译，或是根据本地语种和文化进行相应的格式化。
+ 根据用户的偏好设置，使用钩子将web应用本地化。

很显然，翻译取决于用户所选语言，而格式化通常取决于用户所在国家。 这些信息由浏览器通过`Accept-Language` 协议头提供。不过确定时区就不是这么简单了。

## 定义 ##

国际化和本地化通常会被混淆，这里我们对其进行简单的定义和区分：

**国际化**

让软件支持本地化的准备工作，通常由开发者完成。

**本地化**

编写翻译和本地格式，通常由翻译者完成。

更多细节详见[W3C Web Internationalization FAQ](http://www.w3.org/International/questions/qa-i18n)、[Wikipedia article](http://en.wikipedia.org/wiki/Internationalization_and_localization)和[GNU gettext documentation](http://www.gnu.org/software/gettext/manual/gettext.html#Concepts)。

> 警告
>
> 是否启用翻译和格式化分别由配置项`USE_I18N`和 `USE_L10N` 决定。 但是，这两个配置项都同时影响国际化和本地化。 这种情况是Django的历史因素所致。

下面几项可帮助我们更好地处理某种语言：

**本地化名称**

表示地域文化的名称，可以是 `ll` 格式的语种代码，也可以是 `ll_CC` 格式的语种和国家组合代码。例如：`it`, `de_AT`, `es`, `pt_BR` 。语种部分总是小写而国家部分则应是大写，中间以下划线(\_)连接。

**语言代码**

表示语言的名称。浏览器会发送带有语言代码的 ``Accept-Language`` HTTP报头给服务器。例如：`it`, `de-at`, `es`, `pt-br`. 语种和国家部分都是小写，中间以破折线(-)连接。

**消息文件**

消息文件是纯文本文件，包含某种语言下所有可用的[翻译字符串](http://python.usyiyi.cn/django/topics/i18n/index.html#term-translation-string)及其对应的翻译结果。消息文件以 `.po` 做为文件扩展名。

**翻译字符串**

可以被翻译的文字。

**格式文件**

针对某个地域定义数据格式的Python模块。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Overview](https://docs.djangoproject.com/en/1.8/topics/i18n/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
