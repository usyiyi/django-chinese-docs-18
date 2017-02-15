

# 性能与优化

本文档提供的技术与工具概述，有助于使您的Django代码更高效，更快速，并使用更少系统资源。

## 简介

通常，人们首先关心的是编写的代码_起作用_，其逻辑函数根据需要产生预期输出。然而，有时，这将不足以使代码像我们希望的那样_有效地工作_。

Generally one’s first concern is to write code that works, whose logic functions as required to produce the expected output. Sometimes, however, this will not be enough to make the code work as efficiently as one would like.

## General approaches

### What are you optimizing _for_?

重要的是有一个清楚的想法你的意思是“性能”。它不只有一个指标。

改进的速度可能是程序最明显的目标，但有时可能会寻求其他性能改进，例如降低内存消耗或减少对数据库或网络的需求。

一个领域的改进往往会带来另一个领域的改进，但并不总是如此；有时甚至可以牺牲另一个。例如，程序速度的提高可能会导致程序使用更多的内存。更糟糕的是，它可以是自我毁灭 - 如果速度的改善是如此内存饥饿，系统开始耗尽内存，你会做更多的危害比好。

还有其他权衡要考虑。你自己的时间是宝贵的资源，比CPU时间更珍贵。一些改进可能太难以实现，或者可能影响代码的可移植性或可维护性。并非所有的性能改进都值得付出努力。

所以，你需要知道你的目标是什么性能改进，你还需要知道你有一个很好的理由，瞄准这个方向，因为你需要：

### Performance benchmarking

这是不好的只是猜测或假设在你的代码中低效率。

#### Django tools

[django-debug-toolbar](https://github.com/django-debug-toolbar/django-debug-toolbar/)是一个非常方便的工具，可以深入了解代码的工作以及它花费多少时间。特别是它可以显示你的页面生成的所有SQL查询，以及每个人花了多长时间。

第三方面板也可用于工具栏，可以（例如）报告缓存性能和模板呈现时间。

#### Third-party services

有一些免费服务将从远程HTTP客户端的角度分析和报告您的网站的页面的性能，实际上模拟实际用户的体验。

这些不能报告您的代码的内部，但可以提供有用的洞察您的网站的整体性能，包括不能从Django环境中充分测量的方面。示例包括：

*   [雅虎Yslow](http://yslow.org/)
*   [Google PageSpeed](https://developers.google.com/speed/pagespeed/)

还有一些付费服务执行类似的分析，包括一些是Django感知的，可以与您的代码库集成，以更广泛地分析其性能。

### Get things right from the start

一些优化工作涉及解决性能缺陷，但是有些工作可以简单地建立在你所做的工作中，作为你应该采取的良好做法的一部分，甚至在你开始考虑提高性能之前。

在这方面，Python是一个很好的语言，因为看起来优雅和感觉合适的解决方案通常是最好的表现。与大多数技能一样，学习什么“看起来正确”需要练习，但最有用的指南之一是：

#### Work at the appropriate level

Django提供了许多不同的方法来处理事情，但只是因为它可能以某种方式做某事并不意味着它是最合适的方式。例如，您可能会发现您可以计算同样的事情 - 集合中的项目数量，可能是在`QuerySet`中，在Python中，或在模板中。

但是，在较低级别而不是较高级别执行此工作几乎总是更快。在更高层次，系统必须通过多级抽象和多层机械来处理对象。

也就是说，数据库通常可以比Python更快地完成事情，这样做可以比模板语言更快：

```
# QuerySet operation on the database
# fast, because that's what databases are good at
my_bicycles.count()

# counting Python objects
# slower, because it requires a database query anyway, and processing
# of the Python objects
len(my_bicycles)

# Django template filter
# slower still, because it will have to count them in Python anyway,
# and because of template language overheads
{{ my_bicycles|length }}

```

一般来说，最适合的工作级别是最低级别的代码，它是舒适的。

注意

以上示例仅仅是说明性的。

首先，在现实情况下，您需要考虑在计数之前和之后发生的情况，以确定在特定上下文中执行_的最佳方式。_数据库优化文档描述了[_a case where counting in the template would be better_](db/optimization.html#overuse-of-count-and-exists)。

其次，还有其他选择要考虑：在现实生活中，`{{ my_bicycles.count }} t0 &gt;，它直接从模板调用`QuerySet` `count()`方法可能是最合适的选择。`

## Caching

通常，计算值是昂贵的（即资源匮乏和缓慢），因此将值保存到可快速访问的缓存中可以有巨大的好处，为下一次需要做好准备。

这是一个足够重要和强大的技术，Django包括一个综合的缓存框架，以及其他较小的缓存功能。

### [_The caching framework_](cache.html)

Django的[_caching framework_](cache.html)通过保存动态内容以便不需要为每个请求计算，从而为性能提升提供了非常重要的机会。

为了方便起见，Django提供了不同级别的缓存粒度：您可以缓存特定视图的输出，或仅缓存难以生成的片段，甚至整个网站。

实现缓存不应该被视为改进代码执行不良，因为它已被写得不好的替代方法。这是生产性能良好的代码的最后一步，而不是一个捷径。

### [`cached_property`](../ref/utils.html#django.utils.functional.cached_property "django.utils.functional.cached_property")

通常必须多次调用类实例的方法。如果这个功能是昂贵的，那么这样做可能是浪费。

使用`@cached_property`装饰器保存属性返回的值；下一次在该实例上调用函数时，它将返回保存的值，而不是重新计算它。请注意，这仅适用于将`self`作为其唯一参数的方法，并将方法更改为属性。

某些Django组件也有自己的缓存功能；这些在下面在与那些组件相关的部分中讨论。

## Understanding laziness

_懒惰_是对缓存的补充策略。缓存通过保存结果避免重新计算；延迟延迟计算，直到它实际需要。

懒惰允许我们在实例化之前，甚至在可能实例化它们之前引用它们。这有很多用途。

例如，可以在目标语言甚至已知之前使用[_lazy translation_](i18n/translation.html#lazy-translations)，因为在实际需要翻译的字符串之前不发生，例如在呈现的模板中。

懒惰也是一种通过试图避免工作来节省努力的方式。也就是说，懒惰的一个方面是在做任何事情之前不做任何事情，因为它可能不是必然的。因此，懒惰可能具有性能影响，并且相关工作越昂贵，通过懒惰获得的就越多。

Python提供了许多用于延迟评估的工具，特别是通过[_generator_](https://docs.python.org/3/glossary.html#term-generator "(in Python v3.4)")和[_generator expression_](https://docs.python.org/3/glossary.html#term-generator-expression "(in Python v3.4)")构造。值得一读的是Python中的懒惰，以便发现在代码中使用延迟模式的机会。

### Laziness in Django

Django本身相当懒惰。一个很好的例子可以在`QuerySets`的评估中找到。[_QuerySets are lazy_](db/queries.html#querysets-are-lazy)。因此，可以创建`QuerySet`，传递并与其他`QuerySets`组合，而不实际引发任何到数据库的访问以获取其描述的项目。传递的是`QuerySet`对象，而不是最终需要从数据库中获取的项目集合。

另一方面，[_certain operations will force the evaluation of a QuerySet_](../ref/models/querysets.html#when-querysets-are-evaluated)进行求值。避免对`QuerySet`的过早评估可以节省对数据库的昂贵且不必要的访问。

Django还提供了一个[`allow_lazy()`](../ref/utils.html#django.utils.functional.allow_lazy "django.utils.functional.allow_lazy")装饰器。这允许使用延迟参数调用的函数本身行为迟缓，只有在需要时才进行求值。因此，懒惰的论据 - 可能是一个昂贵的论点 - 不会被要求进行评估，直到它是严格要求。

## Databases

### [_Database optimization_](db/optimization.html)

Django的数据库层提供了各种方法来帮助开发人员从他们的数据库获得最佳性能。[_database optimization documentation_](db/optimization.html)汇集了相关文档的链接，并添加了各种提示，概述在尝试优化数据库使用情况时需要采取的步骤。

### Other database-related tips

启用[_Persistent connections_](../ref/databases.html#persistent-database-connections)可以加快连接到数据库帐户的请求处理时间的很大一部分。

这有助于在有限的网络性能的虚拟化主机上很多。

## HTTP performance

### Middleware

Django附带了一些有用的[_middleware_](../ref/middleware.html)，可以帮助优化您的网站的性能。他们包括：

#### [`ConditionalGetMiddleware`](../ref/middleware.html#django.middleware.http.ConditionalGetMiddleware "django.middleware.http.ConditionalGetMiddleware")

添加了对现代浏览器的支持，可根据`ETag`和`Last-Modified`标头有条件地获取响应。

#### [`GZipMiddleware`](../ref/middleware.html#django.middleware.gzip.GZipMiddleware "django.middleware.gzip.GZipMiddleware")

压缩所有现代浏览器的响应，节省带宽和传输时间。请注意，GZipMiddleware目前被认为是一种安全风险，并且容易受到由TLS / SSL提供的保护无效的攻击。有关详细信息，请参阅[`GZipMiddleware`](../ref/middleware.html#django.middleware.gzip.GZipMiddleware "django.middleware.gzip.GZipMiddleware")中的警告。

### Sessions

#### [_Using cached sessions_](http/sessions.html#cached-sessions-backend)

[_Using cached sessions_](http/sessions.html#cached-sessions-backend)可能是一种通过消除从像数据库这样较慢的存储源加载会话数据而改为将经常使用的会话数据存储在内存中来提高性能的方法。

### Static files

静态文件，根据定义是不动态的，使优化增益的一个优秀的目标。

#### [`CachedStaticFilesStorage`](../ref/contrib/staticfiles.html#django.contrib.staticfiles.storage.CachedStaticFilesStorage "django.contrib.staticfiles.storage.CachedStaticFilesStorage")

通过利用Web浏览器的缓存能力，您可以在初始下载后完全消除给定文件的网络匹配。

[`CachedStaticFilesStorage`](../ref/contrib/staticfiles.html#django.contrib.staticfiles.storage.CachedStaticFilesStorage "django.contrib.staticfiles.storage.CachedStaticFilesStorage")会将一个内容相关标记附加到[_static files_](../ref/contrib/staticfiles.html)的文件名中，以确保浏览器能够安全地对其进行长期缓存，而不会丢失未来的更改 - 当文件更改时，将标记，所以浏览器将自动重新加载资产。

#### “Minification”

一些第三方Django工具和包提供了“缩小”HTML，CSS和JavaScript的能力。它们删除不必要的空格，换行符和注释，缩短变量名，从而减少您的网站发布的文档的大小。

## Template performance

注意：

*   使用`{％ 阻止 ％}`比使用`{％ 包括 ％}`
*   从许多小块组装的重碎片模板可能会影响性能

### The cached template loader

启用[`cached template loader`](../ref/templates/api.html#django.template.loaders.cached.Loader "django.template.loaders.cached.Loader")通常会大幅提高性能，因为它避免每次需要时编译每个模板渲染。

## Using different versions of available software

有时可能需要检查您使用的软件的不同和性能更好的版本是否可用。

这些技术针对的是更高级的用户，他们希望提高已经优化的Django站点的性能边界。

然而，他们不是性能问题的魔法解决方案，他们不可能比没有以更正确的方式做更基本的事情的网站带来更好的边际收益。

注意

值得重复的是：**达到您已经使用的软件的替代品永远不是性能问题的第一个答案**。当您达到这种优化水平时，您需要一个正式的基准解决方案。

### Newer is often - but not always - better

很少有新版本的维护良好的软件效率较低，但是维护者无法预期每一种可能的使用情况 - 因此，虽然知道较新的版本可能性能更好，但不要简单地假设它们一直会。

这是Django本身的真实。连续版本已在系统中提供了许多改进，但您仍应检查应用程序的真实性能，因为在某些情况下，您可能会发现这些更改意味着性能更差，而不是更好。

较新的Python版本以及Python软件包，往往会表现得更好 - 但测量，而不是假设。

注意

除非您在特定版本中遇到了异常的性能问题，否则在新版本中通常会找到更好的功能，可靠性和安全性，这些优点远远胜过您可能获胜或失败的任何性能。

### Alternatives to Django’s template language

对于几乎所有情况，Django的内置模板语言是完全足够的。然而，如果你的Django项目的瓶颈似乎在于模板系统，你已经用尽了其他机会来弥补这一点，第三方替代品可能是答案。

[Jinja2](http://jinja.pocoo.org/docs/)可以提高性能，特别是在速度方面。

替代模板系统在它们共享Django的模板语言的程度上有所不同。

注意

_如果_在模板中遇到性能问题，首先要做的是明白为什么。使用替代模板系统可能会更快，但是也可以获得相同的收益，而不会遇到麻烦 - 例如，您的模板中的昂贵的处理和逻辑可以更有效地在您的视图中。

### Alternative software implementations

可能值得检查你使用的Python软件是否已在不同的实现中提供，可以更快地执行相同的代码。

然而：编写良好的Django站点中的大多数性能问题不是在Python执行级别，而是在于低效的数据库查询，缓存和模板。如果你依赖写得不好的Python代码，你的性能问题不可能通过更快地执行来解决。

使用替代实现可能引入兼容性，部署，可移植性或维护问题。不言而喻，在采用非标准实施之前，您应该确保它为您的应用程序提供足够的性能增益，超过潜在的风险。

记住这些注意事项，您应该注意：

#### [PyPy](http://pypy.org/)

[PyPy](http://pypy.org/)是Python中Python本身的实现（'标准'Python实现是在C中）。PyPy可以提供显着的性能提升，通常用于重量级应用。

PyPy项目的一个关键目标是与现有的Python API和库的[兼容性](http://pypy.org/compat.html)。Django是兼容的，但你需要检查你依赖的其他库的兼容性。

#### C implementations of Python libraries

一些Python库也在C中实现，可以更快。他们的目标是提供相同的API。注意兼容性问题和行为差异是未知的（并不总是立即明显）。

