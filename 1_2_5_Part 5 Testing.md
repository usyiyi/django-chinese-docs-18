# 编写你的第一个Django应用，第5部分 #

本教程上接教程第4部分。 我们已经建立一个网页投票应用，现在我们将为它创建一些自动化测试。

## 自动化测试简介 ##

### 什么是自动化测试？ ###

测试是检查你的代码是否正常运行的简单程序。

测试可以划分为不同的级别。 一些测试可能专注于小细节（某一个模型的方法是否会返回预期的值？）， 其他的测试可能会检查软件的整体运行是否正常（用户在对网站进行了一系列的操作后，是否返回了正确的结果？）。这些其实和你早前在教程 1中做的差不多， 使用shell来检测一个方法的行为，或者运行程序并输入数据来检查它的行为方式。

自动化测试的不同之处就在于这些测试会由系统来帮你完成。你创建了一组测试程序，当你修改了你的应用，你就可以用这组测试程序来检查你的代码是否仍然同预期的那样运行，而无需执行耗时的手动测试。

### 为什么你需要创建测试 ###

那么，为什么要创建测试？而且为什么是现在？

你可能感觉学习Python/Django已经足够，再去学习其他的东西也许需要付出巨大的努力而且没有必要。 毕竟，我们的投票应用已经活蹦乱跳了； 将时间运用在自动化测试上还不如运用在改进我们的应用上。 如果你学习Django就是为了创建一个投票应用，那么创建自动化测试显然没有必要。 但如果不是这样，现在是一个很好的学习机会。

#### 测试将节省你的时间 ####

在某种程度上， ‘检查起来似乎正常工作’将是一种令人满意的测试。 在更复杂的应用中，你可能有几十种组件之间的复杂的相互作用。

这些组件的任何一个小的变化，都可能对应用的行为产生意想不到的影响。 检查起来‘似乎正常工作’可能意味着你需要运用二十种不同的测试数据来测试你代码的功能，仅仅是为了确保你没有搞砸某些事 —— 这不是对时间的有效利用。

尤其是当自动化测试只需要数秒就可以完成以上的任务时。 如果出现了错误，测试程序还能够帮助找出引发这个异常行为的代码。

有时候你可能会觉得编写测试程序将你从有价值的、创造性的编程工作里带出，带到了单调乏味、无趣的编写测试中，尤其是当你的代码工作正常时。

然而，比起用几个小时的时间来手动测试你的程序，或者试图找出代码中一个新引入的问题的原因，编写测试程序还是令人惬意的。

#### 测试不仅仅可以发现问题，它们还能防止问题 ####

将测试看做只是开发过程中消极的一面是错误的。

没有测试，应用的目的和意图将会变得相当模糊。 甚至在你查看自己的代码时，也不会发现这些代码真正干了些什么。

测试改变了这一切； 它们使你的代码内部变得明晰，当错误出现后，它们会明确地指出哪部分代码出了问题 —— 甚至你自己都不会料到问题会出现在那里。

#### 测试使你的代码更受欢迎 ####

你可能已经创建了一个堪称辉煌的软件，但是你会发现许多其他的开发者会由于它缺少测试程序而拒绝查看它一眼；没有测试程序，他们不会信任它。 Jacob Kaplan-Moss，Django最初的几个开发者之一，说过“不具有测试程序的代码是设计上的错误。”

你需要开始编写测试的另一个原因就是其他的开发者在他们认真研读你的代码前可能想要查看一下它有没有测试。

#### 测试有助于团队合作 ####

之前的观点是从单个开发人员来维护一个程序这个方向来阐述的。 复杂的应用将会被一个团队来维护。 测试能够减少同事在无意间破坏你的代码的机会（和你在不知情的情况下破坏别人的代码的机会）。 如果你想在团队中做一个好的Django开发者，你必须擅长测试！

## 基本的测试策略 ##

编写测试有很多种方法。

一些开发者遵循一种叫做“由测试驱动的开发”的规则；他们在编写代码前会先编好测试。 这似乎与直觉不符，尽管这种方法与大多数人经常的做法很相似：人们先描述一个问题，然后创建一些代码来解决这个问题。 由测试驱动的开发可以用Python测试用例将这个问题简单地形式化。

更常见的情况是，刚接触测试的人会先编写一些代码，然后才决定为这些代码创建一些测试。 也许在之前就编写一些测试会好一点，但什么时候开始都不算晚。

有时候很难解决从什么地方开始编写测试。 如果你已经编写了数千行Python代码，挑选它们中的一些来进行测试不会是太容易的。 这种情况下，在下次你对代码进行变更，或者添加一个新功能或者修复一个bug时，编写你的第一个测试，效果会非常好。

现在，让我们马上来编写一个测试。

## 编写我们的第一个测试  ##

### 我们找出一个错误 ###

幸运的是，polls应用中有一个小错误让我们可以马上来修复它：如果Question在最后一个天发布，Question.was_published_recently() 方法返回True（这是对的），但是如果Question的pub_date 字段是在未来，它还返回True（这肯定是不对的）。

你可以在管理站点中看到这一点； 创建一个发布时间在未来的一个Question； 你可以看到Question 的变更列表声称它是最近发布的。

你还可以使用shell看到这点：

```
>>> import datetime
>>> from django.utils import timezone
>>> from polls.models import Question
>>> # create a Question instance with pub_date 30 days in the future
>>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
>>> # was it published recently
>>> future_question.was_published_recently()
True
```

由于将来的事情并不能称之为‘最近’，这确实是一个错误。

### 创建一个测试来暴露这个错误 ###

我们需要在自动化测试里做的和刚才在shell里做的差不多，让我们来将它转换成一个自动化测试。

应用的测试用例安装惯例一般放在该应用的tests.py文件中；测试系统将自动在任何以test开头的文件中查找测试用例。

将下面的代码放入polls应用下的tests.py文件中：

```
polls/tests.py
import datetime

from django.utils import timezone
from django.test import TestCase

from .models import Question


class QuestionMethodTests(TestCase):

    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() should return False for questions whose
        pub_date is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertEqual(future_question.was_published_recently(), False)
```

我们在这里做的是创建一个django.test.TestCase子类，它具有一个方法可以创建一个pub_date在未来的Question实例。然后我们检查was_published_recently()的输出 —— 它应该是 False.

### 运行测试 ###

在终端中，我们可以运行我们的测试：

```
$ python manage.py test polls
```

你将看到类似下面的输出：

```
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionMethodTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/path/to/mysite/polls/tests.py", line 16, in test_was_published_recently_with_future_question
    self.assertEqual(future_question.was_published_recently(), False)
AssertionError: True != False

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
```

发生了如下这些事：

+ python manage.py test polls查找polls 应用下的测试用例
+ 它找到 django.test.TestCase 类的一个子类
+ 它为测试创建了一个特定的数据库
+ 它查找用于测试的方法 —— 名字以test开始
+ 它运行test_was_published_recently_with_future_question创建一个pub_date为未来30天的 Question实例
+ ... 然后利用assertEqual()方法，它发现was_published_recently() 返回True，尽管我们希望它返回False

这个测试通知我们哪个测试失败，甚至是错误出现在哪一行。

### 修复这个错误 ###

我们已经知道问题是什么：Question.was_published_recently() 应该返回 False，如果它的pub_date是在未来。在models.py中修复这个方法，让它只有当日期是在过去时才返回True ：

```
polls/models.py
def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

再次运行测试：

```
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
```

在找出一个错误之后，我们编写一个测试来暴露这个错误，然后在代码中更正这个错误让我们的测试通过。

未来，我们的应用可能会出许多其它的错误，但是我们可以保证我们不会无意中再次引入这个错误，因为简单地运行一下这个测试就会立即提醒我们。 我们可以认为这个应用的这一小部分会永远安全了。

### 更加综合的测试 ###

在这里，我们可以使was_published_recently() 方法更加稳定；事实上，在修复一个错误的时候引入一个新的错误将是一件很令人尴尬的事。

在同一个类中添加两个其它的测试方法，来更加综合地测试这个方法：

```
polls/tests.py
def test_was_published_recently_with_old_question(self):
    """
    was_published_recently() should return False for questions whose
    pub_date is older than 1 day.
    """
    time = timezone.now() - datetime.timedelta(days=30)
    old_question = Question(pub_date=time)
    self.assertEqual(old_question.was_published_recently(), False)

def test_was_published_recently_with_recent_question(self):
    """
    was_published_recently() should return True for questions whose
    pub_date is within the last day.
    """
    time = timezone.now() - datetime.timedelta(hours=1)
    recent_question = Question(pub_date=time)
    self.assertEqual(recent_question.was_published_recently(), True)
```

现在我们有三个测试来保证无论发布时间是在过去、现在还是未来 Question.was_published_recently()都将返回合理的数据。

再说一次，polls 应用虽然简单，但是无论它今后会变得多么复杂以及会和多少其它的应用产生相互作用，我们都能保证我们刚刚为它编写过测试的那个方法会按照预期的那样工作。

## 测试一个视图 ##

这个投票应用没有区分能力：它将会发布任何一个Question，包括 pub_date字段位于未来。我们应该改进这一点。 设定pub_date在未来应该表示Question在此刻发布，但是直到那个时间点才会变得可见。

### 视图的一个测试 ###

当我们修复上面的错误时，我们先写测试，然后修改代码来修复它。 事实上，这是由测试驱动的开发的一个简单的例子，但做的顺序并不真的重要。

在我们的第一个测试中，我们专注于代码内部的行为。 在这个测试中，我们想要通过浏览器从用户的角度来检查它的行为。

在我们试着修复任何事情之前，让我们先查看一下我们能用到的工具。

### Django测试客户端 ###

Django提供了一个测试客户端来模拟用户和代码的交互。我们可以在tests.py 甚至在shell 中使用它。

我们将再次以shell开始，但是我们需要做很多在tests.py中不必做的事。首先是在 shell中设置测试环境：

```
>>> from django.test.utils import setup_test_environment
>>> setup_test_environment()
```

setup_test_environment()安装一个模板渲染器，可以使我们来检查响应的一些额外属性比如response.context，否则是访问不到的。请注意，这种方法不会建立一个测试数据库，所以以下命令将运行在现有的数据库上，输出的内容也会根据你已经创建的Question不同而稍有不同。

下一步我们需要导入测试客户端类（在之后的tests.py 中，我们将使用django.test.TestCase类，它具有自己的客户端，将不需要导入这个类）：

```
>>> from django.test import Client
>>> # create an instance of the client for our use
>>> client = Client()
```

这些都做完之后，我们可以让这个客户端来为我们做一些事：

```
>>> # get a response from '/'
>>> response = client.get('/')
>>> # we should expect a 404 from that address
>>> response.status_code
404
>>> # on the other hand we should expect to find something at '/polls/'
>>> # we'll use 'reverse()' rather than a hardcoded URL
>>> from django.core.urlresolvers import reverse
>>> response = client.get(reverse('polls:index'))
>>> response.status_code
200
>>> response.content
'\n\n\n    <p>No polls are available.</p>\n\n'
>>> # note - you might get unexpected results if your ``TIME_ZONE``
>>> # in ``settings.py`` is not correct. If you need to change it,
>>> # you will also need to restart your shell session
>>> from polls.models import Question
>>> from django.utils import timezone
>>> # create a Question and save it
>>> q = Question(question_text="Who is your favorite Beatle?", pub_date=timezone.now())
>>> q.save()
>>> # check the response once again
>>> response = client.get('/polls/')
>>> response.content
'\n\n\n    <ul>\n    \n        <li><a href="/polls/1/">Who is your favorite Beatle?</a></li>\n    \n    </ul>\n\n'
>>> # If the following doesn't work, you probably omitted the call to
>>> # setup_test_environment() described above
>>> response.context['latest_question_list']
[<Question: Who is your favorite Beatle?>]
```

### 改进我们的视图 ###

投票的列表显示还没有发布的投票（即pub_date在未来的投票）。让我们来修复它。

在教程 4中，我们介绍了一个继承ListView的基于类的视图：

```
polls/views.py
class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]
```

response.context_data['latest_question_list'] 取出由视图放置在context 中的数据。

我们需要修改get_queryset方法并让它将日期与timezone.now()进行比较。首先我们需要添加一行导入：

```
polls/views.py
from django.utils import timezone
```

然后我们必须像这样修改get_queryset方法：

```
polls/views.py
def get_queryset(self):
    """
    Return the last five published questions (not including those set to be
    published in the future).
    """
    return Question.objects.filter(
        pub_date__lte=timezone.now()
    ).order_by('-pub_date')[:5]
```

Question.objects.filter(pub_date__lte=timezone.now()) 返回一个查询集，包含pub_date小于等于timezone.now的Question。

### 测试我们的新视图 ###

启动服务器、在浏览器中载入站点、创建一些发布时间在过去和将来的Questions ，然后检验只有已经发布的Question会展示出来，现在你可以对自己感到满意了。你不想每次修改可能与这相关的代码时都重复这样做 —— 所以让我们基于以上shell会话中的内容，再编写一个测试。

将下面的代码添加到polls/tests.py：

```
polls/tests.py
from django.core.urlresolvers import reverse
```

我们将创建一个快捷函数来创建Question，同时我们要创建一个新的测试类：

```
polls/tests.py
def create_question(question_text, days):
    """
    Creates a question with the given `question_text` published the given
    number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text,
                                   pub_date=time)


class QuestionViewTests(TestCase):
    def test_index_view_with_no_questions(self):
        """
        If no questions exist, an appropriate message should be displayed.
        """
        response = self.client.get(reverse('polls:index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_index_view_with_a_past_question(self):
        """
        Questions with a pub_date in the past should be displayed on the
        index page.
        """
        create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )

    def test_index_view_with_a_future_question(self):
        """
        Questions with a pub_date in the future should not be displayed on
        the index page.
        """
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertContains(response, "No polls are available.",
                            status_code=200)
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_index_view_with_future_question_and_past_question(self):
        """
        Even if both past and future questions exist, only past questions
        should be displayed.
        """
        create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )

    def test_index_view_with_two_past_questions(self):
        """
        The questions index page may display multiple questions.
        """
        create_question(question_text="Past question 1.", days=-30)
        create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question 2.>', '<Question: Past question 1.>']
        )
```

让我们更详细地看下以上这些内容。

第一个是Question的快捷函数create_question，将重复创建Question的过程封装在一起。

test_index_view_with_no_questions不创建任何Question，但会检查消息“No polls are available.” 并验证latest_question_list为空。注意django.test.TestCase类提供一些额外的断言方法。在这些例子中，我们使用assertContains() 和 assertQuerysetEqual()。

在test_index_view_with_a_past_question中，我们创建一个Question并验证它是否出现在列表中。

在test_index_view_with_a_future_question中，我们创建一个pub_date 在未来的Question。数据库会为每一个测试方法进行重置，所以第一个Question已经不在那里，因此首页面里不应该有任何Question。

等等。 事实上，我们是在用测试模拟站点上的管理员输入和用户体验，检查针对系统每一个状态和状态的新变化，发布的是预期的结果。

### 测试 DetailView ###

一切都运行得很好； 然而，即使未来发布的Question不会出现在index中，如果用户知道或者猜出正确的URL依然可以访问它们。所以我们需要给DetailView添加一个这样的约束：

```
polls/views.py
class DetailView(generic.DetailView):
    ...
    def get_queryset(self):
        """
        Excludes any questions that aren't published yet.
        """
        return Question.objects.filter(pub_date__lte=timezone.now())
```

当然，我们将增加一些测试来检验pub_date 在过去的Question 可以显示出来，而pub_date在未来的不可以：

```
polls/tests.py
class QuestionIndexDetailTests(TestCase):
    def test_detail_view_with_a_future_question(self):
        """
        The detail view of a question with a pub_date in the future should
        return a 404 not found.
        """
        future_question = create_question(question_text='Future question.',
                                          days=5)
        response = self.client.get(reverse('polls:detail',
                                   args=(future_question.id,)))
        self.assertEqual(response.status_code, 404)

    def test_detail_view_with_a_past_question(self):
        """
        The detail view of a question with a pub_date in the past should
        display the question's text.
        """
        past_question = create_question(question_text='Past Question.',
                                        days=-5)
        response = self.client.get(reverse('polls:detail',
                                   args=(past_question.id,)))
        self.assertContains(response, past_question.question_text,
                            status_code=200)
```

### 更多的测试思路 ###

我们应该添加一个类似get_queryset的方法到ResultsView并为该视图创建一个新的类。这将与我们刚刚创建的非常类似；实际上将会有许多重复。

我们还可以在其它方面改进我们的应用，并随之不断增加测试。例如，发布一个没有Choices的Questions就显得傻傻的。所以，我们的视图应该检查这点并排除这些 Questions。我们的测试应该创建一个不带Choices 的 Question然后测试它不会发布出来， 同时创建一个类似的带有 Choices的Question 并验证它会 发布出来。

也许登陆的用户应该被允许查看还没发布的 Questions，但普通游客不行。 再说一次：无论添加什么代码来完成这个要求，需要提供相应的测试代码，无论你是否是先编写测试然后让这些代码通过测试，还是先用代码解决其中的逻辑然后编写测试来证明它。

从某种程度上来说，你一定会查看你的测试，然后想知道是否你的测试程序过于臃肿，这将我们带向下面的内容：

## 测试越多越好 ##

看起来我们的测试代码的增长正在失去控制。 以这样的速度，测试的代码量将很快超过我们的应用，对比我们其它优美简洁的代码，重复毫无美感。

没关系。让它们继续增长。最重要的是，你可以写一个测试一次，然后忘了它。 当你继续开发你的程序时，它将继续执行有用的功能。

有时，测试需要更新。 假设我们修改我们的视图使得只有具有Choices的 Questions 才会发布。在这种情况下，我们许多已经存在的测试都将失败 —— 这会告诉我们哪些测试需要被修改来使得它们保持最新，所以从某种程度上讲，测试可以自己照顾自己。

在最坏的情况下，在你的开发过程中，你会发现许多测试现在变得冗余。 即使这样，也不是问题；对测试来说，冗余是一件好 事。

只要你的测试被合理地组织，它们就不会变得难以管理。 从经验上来说，好的做法是：

+ 每个模型或视图具有一个单独的TestClass
+ 为你想测试的每一种情况建立一个单独的测试方法
+ 测试方法的名字可以描述它们的功能

## 进一步的测试 ##

本教程只介绍了一些基本的测试。 还有很多你可以做，有许多非常有用的工具可以随便使用来你实现一些非常聪明的做法。

例如，虽然我们的测试覆盖了模型的内部逻辑和视图发布信息的方式，你可以使用一个“浏览器”框架例如Selenium来测试你的HTML文件在浏览器中真实渲染的样子。 这些工具不仅可以让你检查你的Django代码的行为，还能够检查你的JavaScript的行为。 它会启动一个浏览器，并开始与你的网站进行交互，就像有一个人在操纵一样，非常值得一看！ Django 包含一个LiveServerTestCase来帮助与Selenium 这样的工具集成。

如果你有一个复杂的应用，你可能为了实现continuous integration，想在每次提交代码后对代码进行自动化测试，让代码自动 —— 至少是部分自动 —— 地来控制它的质量。

发现你应用中未经测试的代码的一个好方法是检查测试代码的覆盖率。 这也有助于识别脆弱的甚至死代码。 如果你不能测试一段代码，这通常意味着这些代码需要被重构或者移除。 Coverage将帮助我们识别死代码。 查看与coverage.py 集成来了解更多细节。

Django 中的测试有关于测试更加全面的信息。

## 下一步？ ##

关于测试的完整细节，请查看Django 中的测试。

当你对Django 视图的测试感到满意后，请阅读本教程的第6部分来 了解静态文件的管理。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Part 5: Testing](https://docs.djangoproject.com/en/1.8/intro/tutorial05/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
