{% raw %}

# 编写你的第一个Django应用，第6部分 #

本教程上接教程 5。 我们已经建立一个测试过的网页投票应用，现在我们将添加一张样式表和一张图片。

除了由服务器生成的HTML文件外，网页应用一般需要提供其它必要的文件 —— 比如图片文件、JavaScript脚本和CSS样式表 —— 来为用户呈现出一个完整的网站。 在Django中，我们将这些文件称为“静态文件”。

对于小型项目，这不是个大问题，因为你可以将它们放在你的网页服务器可以访问到的地方。 然而，在大一点的项目中 —— 尤其是那些由多个应用组成的项目 —— 处理每个应用提供的多个静态文件集合开始变得很难。

这正是django.contrib.staticfiles的用途：它收集每个应用（和任何你指定的地方）的静态文件到一个单独的位置，这个位置在线上可以很容易维护。

## 自定义你的应用的外观 ##

首先在你的polls中创建一个static目录。Django将在那里查找静态文件，与Django如何polls/templates/内部的模板类似。

Django 的 STATICFILES_FINDERS 设置包含一个查找器列表，它们知道如何从各种源找到静态文件。 其中默认的一个是AppDirectoriesFinder，它在每个INSTALLED_APPS下查找“static”子目录，就像刚刚在polls中创建的一样。管理站点也为它的静态文件使用相同的目录结构。

在你刚刚创建的static目录中，创建另外一个目录polls并在它下面创建一个文件style.css。换句话讲，你的样式表应该位于polls/static/polls/style.css。因为AppDirectoriesFinder 静态文件查找器的工作方式，你可以通过polls/style.css在Django中访问这个静态文件，与你如何访问模板的路径类似。

> 静态文件的命名空间
>
> 与模板类似，我们可以家那个我们的静态文件直接放在polls/static（而不是创建另外一个polls 子目录），但实际上这是一个坏主意。Django将使用它所找到的第一个文件名符合要求的静态文件，如果在你的不同应用中存在两个同名的静态文件，Django将无法区分它们。 我们需要告诉Django该使用其中的哪一个，最简单的方法就是为它们添加命名空间。 也就是说，将这些静态文件放进以它们所在的应用的名字命名的另外一个目录下。

将下面的代码放入样式表中 (polls/static/polls/style.css)：

```
polls/static/polls/style.css
li a {
    color: green;
}
```

下一步，在polls/templates/polls/index.html的顶端添加如下内容 ：

```
polls/templates/polls/index.html
{% load staticfiles %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}" />
```

`{% load staticfiles %}` 从staticfiles模板库加载`{% static %}` 模板标签。`{% static %}`模板标签会生成静态文件的绝对URL。

这就是你在开发过程中，所需要对静态文件做的所有处理。 重新加载 http://localhost:8000/polls/ ，你应该会看到Question的超链接变成了绿色（Django的风格！），这意味着你的样式表被成功导入。

## 添加一张背景图片 ##

下一步，我们将创建一个子目录来存放图片。 在polls/static/polls/目录中创建一个 images 子目录。在这个目录中，放入一张图片background.gif。换句话，将你的图片放在 polls/static/polls/images/background.gif。

然后，向你的样式表添加（polls/static/polls/style.css）：

```
polls/static/polls/style.css
body {
    background: white url("images/background.gif") no-repeat right bottom;
}
```

重新加载 http://localhost:8000/polls/ ，你应该在屏幕的右下方看到载入的背景图片。

> 警告：
>
> 当然，`{% static %}`模板标签不能用在静态文件（比如样式表）中，因为他们不是由Django生成的。 你应该永远使用相对路径来相互链接静态文件，因为这样你可以改变STATIC_URL （ static模板标签用它来生成URLs）而不用同时修改一大堆静态文件的路径。

这些知识基础。关于静态文件设置的更多细节和框架中包含的其它部分，参见静态文件 howto 和静态文件参考。部署静态文件讨论如何在真实的服务器上使用静态文件。

## 下一步？ ##

新手教程到此结束。 在这期间，你可能想要在如何查看文档中了解文档的结构和查找相关信息方法。

如果你熟悉Python 打包的技术，并且对如何将投票应用制作成一个“可重用的应用”感兴趣，请看高级教程：如何编写可重用的应用。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Part 6: Static files](https://docs.djangoproject.com/en/1.8/intro/tutorial06/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。

{% endraw %}
