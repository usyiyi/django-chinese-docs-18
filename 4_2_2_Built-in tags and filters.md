{% raw %}

# 内置标签与过滤器

本文档描述的是django 内置模板标签和过滤器.我们推荐尽可能使用 [_自动文档_](../contrib/admin/admindocs.html)，同时也可以自行编辑任何已安装的自定义标签或过滤器的文档。

## 内置标签参考指南

### autoescape 自动转义

控制自动转义是否可用.这种标签带有任何 `on` 或 `off` 作为参数的话，他将决定转义块内效果。该标签会以一个`endautoescape`作为结束标签.

当自动转义生效时，所有变量内容会被转义成HTML输出（在所有过滤器生效后）这等同与手动将[`escape`](#std:templatefilter-escape)筛选器应用于每个变量。

唯一一个例外是，变量或者通过渲染变量的代码，或者因为它已经应用了 [`safe`](#std:templatefilter-safe)或[`escape`](#std:templatefilter-escape)过滤器，已经被标记为“safe”。

样品用量：

```
{% autoescape on %}
    {{ body }}
{% endautoescape %}

```

### 块

block标签可以被子模板覆盖.查看 [_Template inheritance_](language.html#template-inheritance) 可以获得更多信息.

### 评论

在 `{% comment %}` 和 `{% endcomment %}`，之间的内容会被忽略，作为注释。在第一个标签可以插入一个可选的记录。 比如，当要注释掉一些代码时，可以用此来记录代码被注释掉的原因。

简单实例：

```
<p>Rendered text with {{ pub_date|date:"c" }}</p>
{% comment "Optional note" %}
    <p>Commented out text with {{ create_date|date:"c" }}</p>
{% endcomment %}

```

`comment`标签不能嵌套使用。

### csrf_token

这个标签用于跨站请求伪造保护, 具体可以参考[_Cross Site Request Forgeries_](../csrf.html)中的描述。

### cycle

每当这个标签被访问,则传出一个它的可迭代参数的元素。第一次访问返回第一个元素,第二次访问返回第二个参数,以此类推.一旦所有的变量都被访问过了，就会回到最开始的地方，重复下去

这个标签在循环中特别有用:

```
{% for o in some_list %}
    <tr class="{% cycle 'row1' 'row2' %}">
        ...
    </tr>
{% endfor %}

```

第一次迭代产生的HTML引用了 `row1`类，第二次则是`row2`类，第三次 又是`row1` 类，如此类推。

你也可以使用变量，例如，如果你有两个模版变量, `rowvalue1`和`rowvalue2`, 你可以让他们的值像这样替换:

```
{% for o in some_list %}
    <tr class="{% cycle rowvalue1 rowvalue2 %}">
        ...
    </tr>
{% endfor %}

```

被包含在cycle中的变量将会被转义。你可以禁止自动转义:

```
{% for o in some_list %}
    <tr class="{% autoescape off %}{% cycle rowvalue1 rowvalue2 %}{% endautoescape %}">
        ...
    </tr>
{% endfor %}

```

你能混合使用变量和字符串：

```
{% for o in some_list %}
    <tr class="{% cycle 'row1' rowvalue2 'row3' %}">
        ...
    </tr>
{% endfor %}

```

In some cases you might want to refer to the current value of a cycle without advancing to the next value.To do this, just give the `{% cycle %}` tag a name, using “as”, like this:

```
{% cycle 'row1' 'row2' as rowcolors %}

```

From then on, you can insert the current value of the cycle wherever you’d like in your template by referencing the cycle name as a context variable.If you want to move the cycle to the next value independently of the original `cycle` tag, you can use another `cycle` tag and specify the name of the variable.所以，下面的模板：

```
<tr>
    <td class="{% cycle 'row1' 'row2' as rowcolors %}">...</td>
    <td class="{{ rowcolors }}">...</td>
</tr>
<tr>
    <td class="{% cycle rowcolors %}">...</td>
    <td class="{{ rowcolors }}">...</td>
</tr>

```

将输出：

```
<tr>
    <td class="row1">...</td>
    <td class="row1">...</td>
</tr>
<tr>
    <td class="row2">...</td>
    <td class="row2">...</td>
</tr>

```

 `cycle` 标签中，通过空格分割，你可以使用任意数量的值。被包含在单引号 (`'`)或者双引号 (`"`) 中的值被认为是可迭代字符串，相反，没有被引号包围的值被当作模版变量。

默认情况下，当你在cycle标签中使用`as` 关键字时，关于`{% cycle %}`的使用，会启动cycle并且直接产生第一个值。如果你想要在嵌套循环中或者included模版中使用这个值，那么将会遇到困难。如果你只是想要声明cycle，但是不产生第一个值，你可以添加一个`silent`关键字来作为cycle标签的最后一个关键字。例如:

```
{% for obj in some_list %}
    {% cycle 'row1' 'row2' as rowcolors silent %}
    <tr class="{{ rowcolors }}">{% include "subtemplate.html" %}</tr>
{% endfor %}

```

This will output a list of `&lt;tr&gt;` elements with `class` alternating between `row1` and `row2`.The subtemplate will have access to `rowcolors` in its context and the value will match the class of the `&lt;tr&gt;` that encloses it.If the `silent` keyword were to be omitted, `row1` and `row2` would be emitted as normal text, outside the `&lt;tr&gt;` element.

When the silent keyword is used on a cycle definition, the silence automatically applies to all subsequent uses of that specific cycle tag.The following template would output _nothing_, even though the second call to `{% cycle %}` doesn’t specify `silent`:

```
{% cycle 'row1' 'row2' as rowcolors silent %}
{% cycle rowcolors %}

```

For backward compatibility, the `{% cycle %}` tag supports the much inferior old syntax from previous Django versions.You shouldn’t use this in any new projects, but for the sake of the people who are still using it, here’s what it looks like:

```
{% cycle row1,row2,row3 %}

```

In this syntax, each value gets interpreted as a literal string, and there’s no way to specify variable values.Or literal commas.Or spaces.Did we mention you shouldn’t use this syntax in any new projects?

### debug

输出整个调试信息，包括当前上下文和导入的模块。

### extends

表示当前模板继承自一个父模板

这个标签可以有两种用法:

*   `{% extends "base.html" %}` (要有引号).继承名为`"base.html"`的父模板
*   `{% extends variable %}` 使用`variable`的值. 如果变量被计算成一个字符串，Django将会把它看成是父模版的名字。如果变量被计算到一个`Template`对象，Django将会使用那个对象作为一个父模版。

阅读 [_Template inheritance_](language.html#template-inheritance) 获取更多信息

### filter

通过一个或多个过滤器对内容过滤。 作为灵活可变的语法，多个过滤器被管道符号相连接，且过滤器可以有参数。

注意块中_所有的_内容都应该包括在`filter` 和`endfilter` 标签中。

简单用例：

```
{% filter force_escape|lower %}
    This text will be HTML-escaped, and will appear in all lowercase.
{% endfilter %}

```

注意

[`escape`](#std:templatefilter-escape)和[`safe`](#std:templatefilter-safe)过滤器不是可接受的参数。而应使用[`autoescape`](#std:templatetag-autoescape)标记来管理模板代码块的自动转义。

### firstof

输出第一个不为`False`参数。如果传入的所有变量都为`False`，就什么也不输出。

简单用例：

```
{% firstof var1 var2 var3 %}

```

它等价于：

```
{% if var1 %}
    {{ var1 }}
{% elif var2 %}
    {{ var2 }}
{% elif var3 %}
    {{ var3 }}
{% endif %}

```

当然你也可以用一个默认字符串作为输出以防传入的所有变量都是False：

```
{% firstof var1 var2 var3 "fallback value" %}

```

标签auto-escapes是开启的， 你可以这样关闭auto-escaping:

```
{% autoescape off %}
    {% firstof var1 var2 var3 "<strong>fallback value</strong>" %}
{% endautoescape %}

```

如果只想要部分变量被规避，可以这样使用：

```
{% firstof var1 var2|safe var3 "<strong>fallback value</strong>"|safe %}

```

### for

循环组中的每一个项目，并让这些项目在上下文可用。 举个例子，展示`athlete_list`中的每个成员：

```
<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% endfor %}
</ul>

```

可以利用`{% for obj in list reversed %}`反向完成循环。

如果你需要循环一个包含列表的列表，可以通过拆分每一个二级列表为一个独立变量来达到目的。 举个例子，如果你的内容包括一个叫做`points`的(x,y) 列表，你可以像以下例子一样输出points列表：

```
{% for x, y in points %}
    There is a point at {{ x }},{{ y }}
{% endfor %}

```

如果你想访问一个字典中的项目，这个方法同样有用。举个例子：如果你的内容包含一个叫做`data`的字典，下面的方式可以输出这个字典的键和值：

```
{% for key, value in data.items %}
    {{ key }}: {{ value }}
{% endfor %}

```

for循环设置了一系列在循环中可用的变量：

<colgroup><col width="36%"> <col width="64%"></colgroup> 
| Variable | Description |
| --- | --- |
| `forloop.counter` | The current iteration of the loop (1-indexed) |
| `forloop.counter0` | The current iteration of the loop (0-indexed) |
| `forloop.revcounter` | The number of iterations from the end of the loop (1-indexed) |
| `forloop.revcounter0` | The number of iterations from the end of the loop (0-indexed) |
| `forloop.first` | True if this is the first time through the loop |
| `forloop.last` | True if this is the last time through the loop |
| `forloop.parentloop` | For nested loops, this is the loop surrounding the current one |

### for ... empty

`for` 标签带有一个可选的`{% empty %}` 从句，以便在给出的组是空的或者没有被找到时，可以有所操作。

```
<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% empty %}
    <li>Sorry, no athletes in this list.</li>
{% endfor %}
</ul>

```

它和下面的例子作用相等，但是更简洁、更清晰甚至可能运行起来更快：

```
<ul>
  {% if athlete_list %}
    {% for athlete in athlete_list %}
      <li>{{ athlete.name }}</li>
    {% endfor %}
  {% else %}
    <li>Sorry, no athletes in this list.</li>
  {% endif %}
</ul>

```

### if

`{% if %}`会对一个变量求值，如果它的值是“True”（存在、不为空、且不是boolean类型的false值），这个内容块会输出：

```
{% if athlete_list %}
    Number of athletes: {{ athlete_list|length }}
{% elif athlete_in_locker_room_list %}
    Athletes should be out of the locker room soon!
{% else %}
    No athletes.
{% endif %}

```

上述例子中，如果`athlete_list`不为空，就会通过使用`{{ athlete_list|length }}`过滤器展示出运动员的数量。

正如你所见，`if`标签之后可以带有一个或者多个`{% elif %}` 从句，也可以带有一个`{% else %}`从句以便在之前的所有条件不成立的情况下完成执行。这些从句都是可选的。

### 布尔运算符

[`if`](#std:templatetag-if)标签可以使用`and`，`or`或`not`来测试多个变量或取消给定变量：

```
{% if athlete_list and coach_list %}
    Both athletes and coaches are available.
{% endif %}

{% if not athlete_list %}
    There are no athletes.
{% endif %}

{% if athlete_list or coach_list %}
    There are some athletes or some coaches.
{% endif %}

{% if not athlete_list or coach_list %}
    There are no athletes or there are some coaches.
{% endif %}

{% if athlete_list and not coach_list %}
    There are some athletes and absolutely no coaches.
{% endif %}

```

允许同时使用`and`和`or`子句，`and`的优先级高于`or` ：

```
{% if athlete_list and coach_list or cheerleader_list %}

```

将解释如下：

```
if (athlete_list and coach_list) or cheerleader_list

```

在[`if`](#std:templatetag-if)标记中使用实际括号是无效的语法。如果您需要它们指示优先级，则应使用嵌套的[`if`](#std:templatetag-if)标记。

[`if`](#std:templatetag-if) 标签还可能使用 `==`, `!=`, `&lt;`, `&gt;`, `&lt;=`, `&gt;=` 和 `in` ，他们作用如下：

### `==`运算符

相等。例：

```
{% if somevar == "x" %}
  This appears if variable somevar equals the string "x"
{% endif %}

```

### `!=`运算符

不相等。例：

```
{% if somevar != "x" %}
  This appears if variable somevar does not equal the string "x",
  or if somevar is not found in the context
{% endif %}

```

### `&lt;`运算符

小于。例：

```
{% if somevar < 100 %}
  This appears if variable somevar is less than 100.
{% endif %}

```

### `&gt;`运算符

大于。例：

```
{% if somevar > 0 %}
  This appears if variable somevar is greater than 0.
{% endif %}

```

### `&lt;=`运算符

小于或等于。例：

```
{% if somevar <= 100 %}
  This appears if variable somevar is less than 100 or equal to 100.
{% endif %}

```

### `&gt;=`运算符

大于或等于。例：

```
{% if somevar >= 1 %}
  This appears if variable somevar is greater than 1 or equal to 1.
{% endif %}

```

### `in`运算符

包含在内。许多Python容器支持此运算符，以测试给定值是否在容器中。以下是在 t&gt; y中如何解释`x 的一些示例：`

```
{% if "bc" in "abcdef" %}
  This appears since "bc" is a substring of "abcdef"
{% endif %}

{% if "hello" in greetings %}
  If greetings is a list or set, one element of which is the string
  "hello", this will appear.
{% endif %}

{% if user in users %}
  If users is a QuerySet, this will appear if user is an
  instance that belongs to the QuerySet.
{% endif %}

```

### `not in` 运算符

不包含在内。这是`in`运算符的否定操作。

比较运算符不能像Python或数学符号中那样“链接”。例如，不能使用：

```
{% if a > b > c %}  (WRONG)

```

你应该使用：

```
{% if a > b and b > c %}

```

### 过滤器

你也可以在 [`if`](#std:templatetag-if)表达式中使用过滤器。举个例子：

```
{% if messages|length >= 100 %}
   You have lots of messages today!
{% endif %}

```

### 复合表达式

所有上述操作符可以组合以形成复杂表达式。对于这样的表达式，重要的是要知道在表达式求值时如何对运算符进行分组 - 即优先级规则。操作符的优先级从低至高如下：

*   `or`
*   `and`
*   `not`
*   `in`中
*   `==`, `!=`, `&lt;`, `&gt;`, `&lt;=`, `&gt;=`

（这完全依据Python）。所以，例如，下面的复杂[`if`](#std:templatetag-if)标签：

```
{% if a == b or c == d and e %}

```

...将被解释为：

```
(a == b) or ((c == d) and e)

```

如果你想要不同的优先级，那么你需要使用嵌套的[`if`](#std:templatetag-if) 标签。有时，为了清楚起见，更好的是为了那些不知道优先规则的人。

### ifchanged

检查一个值是否在上一次的迭代中改变。

`{% ifchanged %}` 块标签用在循环里。它可能有两个用处：

1.  检查它已经渲染过的内容中的先前状态。并且只会显示发生改变的内容。例如， 以下的代码是输出days的列表项，不过它只会输出被修改过月份的项:

    ```
    &lt;h1&gt;Archive for {{ year }}&lt;/h1&gt;

    {% for date in days %}
        {% ifchanged %}&lt;h3&gt;{{ date|date:"F" }}&lt;/h3&gt;{% endifchanged %}
        &lt;a href="{{ date|date:"M/d"|lower }}/"&gt;{{ date|date:"j" }}&lt;/a&gt;
    {% endfor %}

    ```

2.  如果标签内被给予多个值时,则会比较每一个值是否与上一次不同。例如，以下显示每次更改时的日期，如果小时或日期已更改，则显示小时：

    ```
    {% for date in days %}
        {% ifchanged date.date %} {{ date.date }} {% endifchanged %}
        {% ifchanged date.hour date.date %}
            {{ date.hour }}
        {% endifchanged %}
    {% endfor %}

    ```

`ifchanged`标记也可以采用可选的`{％ else ％} 将显示如果值没有改变：`

```
{% for match in matches %}
    <div style="background-color:
  {% ifchanged match.ballot_id %}
  {% cycle "red" "blue" %}
  {% else %}
 gray
  {% endifchanged %}
 ">{{ match }}</div>
{% endfor %}

```

### ifequal

如果给定的两个参数是相等的,则显示被标签包含的内容.

举个例子:

```
{% ifequal user.pk comment.user_id %}
    ...
{% endifequal %}

```

在 [`if`](#std:templatetag-if) 标签里面, 也可以包含 `{% else %}` 标签选项.

参数可以是一个硬编码的字符串,所以也可以这样:

```
{% ifequal user.username "adrian" %}
    ...
{% endifequal %}

```

`ifequal`标记的替代方法是使用[`if`](#std:templatetag-if)标记和`==`运算符。

### ifnotequal

就像[`ifequal`](#std:templatetag-ifequal)，不过它测试两个参数不相等。

`ifnotequal`标记的替代方法是使用[`if`](#std:templatetag-if)标记和`!=`运算符。

### include

加载模板并以标签内的参数渲染。这是一种可以引入别的模板的方法。

模板名可以是变量或者是硬编码的字符串，可以用单引号也可以是双引号.

下面这个示例包括模板`“foo/bar.html”`的内容：

```
{% include "foo/bar.html" %}

```

此示例包括其名称包含在变量`template_name`中的模板的内容：

```
{% include template_name %}

```

Changed in Django 1.7:

变量也可以是任何实现了`render()` 方法接口的对象，这个对象要可以接收上下文（context）。这就允许你在context中引用一个已经被编译过的`Template`。

被包含的模板在包含它的模板的上下文中渲染。下面这个示例生成输出`“Hello, John!”`：

*   上下文：变量`person`设置为`“John”`，变量`greeting`设置为`“Hello”`。

*   模板：

    ```
    {% include "name_snippet.html" %}

    ```

*   `name_snippet.html`模板：

    ```
    {{ greeting }}, {{ person|default:"friend" }}!

    ```

你可以使用关键字参数将额外的上下文传递到模板：

```
{% include "name_snippet.html" with person="Jane" greeting="Hello" %}

```

如果要仅使用提供的变量（或根本不使用变量）来渲染上下文，请使用`only`选项。所包含的模板没有其他变量可用：

```
{% include "name_snippet.html" with greeting="Hi" only %}

```

注意

 [`include`](#std:templatetag-include) 标签应该被理解为是一种"将子模版渲染并嵌入HTML中"的变种方法,而不是认为是"解析子模版并在被父模版包含的情况下展现其被父模版定义的内容".这意味着在不同的被包含的子模版之间并不共享父模版的状态,每一个子包含都是完全独立的渲染过程.

Block模块在被包含 _之前_ 就已经被执行. 这意味着模版在被包含之前就已经从另一个block扩展并 _已经被执行并完成渲染_ - 没有block模块会被include引入并执行,即使父模版中的扩展模版.

### load

加载自定义模板标签集。

举个例子, 下面这模板将会从`package`包中载入所有`somelibrary` 和`otherlibrary` 中已经注册的标签和过滤器:

```
{% load somelibrary package.otherlibrary %}

```

你还可以使用`from`参数从库中选择性加载单个过滤器或标记。在下面这个示例中，名为`foo`和`bar`的模板标签/过滤器将从`somelibrary`加载：

```
{% load foo bar from somelibrary %}

```

有关详细信息，请参阅[_自定义标记和过滤器库_](../../howto/custom-template-tags.html)。

### lorem

New in Django 1.8:

该标签之前位于[`django.contrib.webdesign`](../contrib/webdesign.html#module-django.contrib.webdesign "django.contrib.webdesign: Helpers and utilities targeted primarily at Web *designers* rather than Web *developers*.")中。

展示随机的“lorem ipsum”拉丁文本. 这个标签是用来在模版中提供文字样本以供测试用的.

用法：

```
{% lorem [count] [method] [random] %}

```

可以使用零个，一个，两个或三个参数使用`{％ lorem ％}` 。这些参数是：

<colgroup><col width="15%"> <col width="85%"></colgroup> 
| Argument | Description |
| --- | --- |
| `count` | A number (or variable) containing the number of paragraphs or words to generate (default is 1). |
| `method` | Either `w` for words, `p` for HTML paragraphs or `b` for plain-text paragraph blocks (default is `b`). |
| `random` | The word `random`, which if given, does not use the common paragraph (“Lorem ipsum dolor sit amet...”) when generating text. |

例子：

*   `{％ lorem ％}`将输出常见的“lorem ipsum”段落。
*   `{％ lorem 3 p ％} 输出常见的“lorem ipsum”段落和两个随机段落，每个段落包含在HTML `&lt;p&gt;`标签中。`
*   `{％ lorem 2 w 随机 ％} / t6&gt;`将输出两个随机拉丁字。

### now

显示最近的日期或事件,可以通过给定的字符串格式显示。此类字符串可以包含格式说明符字符，如[`date`](#std:templatefilter-date)过滤器部分中所述。

例：

```
It is {% now "jS F Y H:i" %}

```

注意！，如果你想要使用“raw”值，你能够反斜杠转义一个格式化字符串。在这个例子中，“o”和“f”都是反斜杠转义，因为如果不这样，会分别显示年和时间：

```
It is the {% now "jS \o\f F" %}

```

这将显示为“这是9月4日”。

注意

传递的格式也可以是预定义的[`DATE_FORMAT`](../settings.html#std:setting-DATE_FORMAT)，[`DATETIME_FORMAT`](../settings.html#std:setting-DATETIME_FORMAT)，[`SHORT_DATE_FORMAT`](../settings.html#std:setting-SHORT_DATE_FORMAT)或[`SHORT_DATETIME_FORMAT`](../settings.html#std:setting-SHORT_DATETIME_FORMAT)之一。预定义的格式可能会因当前语言环境和[_格式本地化_](../../topics/i18n/formatting.html#format-localization)的启用而有所不同，例如：

```
It is {% now "SHORT_DATETIME_FORMAT" %}

```

You can also use the syntax `{% now "Y" as current_year %}` to store the output inside a variable.This is useful if you want to use `{% now %}` inside a template tag like [`blocktrans`](../../topics/i18n/translation.html#std:templatetag-blocktrans) for example:

```
{% now "Y" as current_year %}
{% blocktrans %}Copyright {{ current_year }}{% endblocktrans %}

```

New in Django 1.8.

添加了使用“as”语法的能力。

### regroup

用相似对象间共有的属性重组列表.

用一个例子来解释这种复杂的标签是最好的方法:say that “places” is a list of cities represented by dictionaries containing `"name"`, `"population"`, and `"country"` keys:

```
cities = [
    {'name': 'Mumbai', 'population': '19,000,000', 'country': 'India'},
    {'name': 'Calcutta', 'population': '15,000,000', 'country': 'India'},
    {'name': 'New York', 'population': '20,000,000', 'country': 'USA'},
    {'name': 'Chicago', 'population': '7,000,000', 'country': 'USA'},
    {'name': 'Tokyo', 'population': '33,000,000', 'country': 'Japan'},
]

```

你会希望用下面这种方式来展示国家和城市的信息

*   India
    *   孟买：19,000,000
    *   加尔各答：15,000,000
*   USA
    *   纽约：20,000,000
    *   芝加哥：7,000,000
*   Japan
    *   东京：33,000,000

你可以使用`{% regroup %}`标签来给每个国家的城市分组。以下模板代码片段将实现这一点：

```
{% regroup cities by country as country_list %}

<ul>
{% for country in country_list %}
    <li>{{ country.grouper }}
    <ul>
        {% for item in country.list %}
          <li>{{ item.name }}: {{ item.population }}</li>
        {% endfor %}
    </ul>
    </li>
{% endfor %}
</ul>

```

让我们来看看这个例子。`{% regroup %}`有三个参数： 你想要重组的列表, 被分组的属性, 还有结果列表的名字. 在这里，我们通过`country`属性重新分组`cities`列表，并调用结果`country_list`。

`{％ regroup ％}`产生一个清单（在本例中为`country_list`的**组对象**。每个组对象有两个属性：

*   `grouper` - 按分组的项目（例如，字符串“India”或“Japan”）。
*   `list` - 此群组中所有项目的列表（例如，所有城市的列表，其中country ='India'）。

请注意，`{％ regroup ％}`不会对其输入进行排序！我们的例子依赖于事实：`cities`列表首先由`country`排序。如果`cities`列表_不_通过`country`对其成员进行排序，则重新分组将天真显示单个国家/地区的多个组。例如，假设`cities`列表已设置为此（请注意，国家/地区未分组在一起）：

```
cities = [
    {'name': 'Mumbai', 'population': '19,000,000', 'country': 'India'},
    {'name': 'New York', 'population': '20,000,000', 'country': 'USA'},
    {'name': 'Calcutta', 'population': '15,000,000', 'country': 'India'},
    {'name': 'Chicago', 'population': '7,000,000', 'country': 'USA'},
    {'name': 'Tokyo', 'population': '33,000,000', 'country': 'Japan'},
]

```

对于`cities`的输入，示例`{％ regroup ％}`以上将导致以下输出：

*   India
    *   孟买：19,000,000
*   USA
    *   纽约：20,000,000
*   India
    *   加尔各答：15,000,000
*   USA
    *   芝加哥：7,000,000
*   Japan
    *   东京：33,000,000

这个问题的最简单的解决方案是确保在你的视图代码中，数据是根据你想要显示的顺序排序。

另一个解决方案是使用[`dictsort`](#std:templatefilter-dictsort)过滤器对模板中的数据进行排序，如果您的数据在字典列表中：

```
{% regroup cities|dictsort:"country" by country as country_list %}

```

### Grouping on other properties

一个有效的模版查找是一个regroup标签的合法的分组属性。包括方法，属性，字典健和列表项。例如，如果“country”字段是具有属性“description”的类的外键，则可以使用：

```
{% regroup cities by country.description as country_list %}

```

或者，如果`country`是具有`choices`的字段，则它将具有作为属性的[`get_FOO_display()`](../models/instances.html#django.db.models.Model.get_FOO_display "django.db.models.Model.get_FOO_display")方法，显示字符串而不是`choices`键：

```
{% regroup cities by get_country_display as country_list %}

```

`{{ country.grouper }}`现在会显示`choices`

### spaceless

删除HTML标签之间的空白格.包括制表符和换行.

用法示例：

```
{% spaceless %}
    <p>
        <a href="foo/">Foo</a>
    </p>
{% endspaceless %}

```

这个示例将返回下面的HTML：

```
<p><a href="foo/">Foo</a></p>

```

仅删除 _tags_ 之间的空格 – 而不是标签和文本之间的。在此示例中，`Hello`周围的空格不会被删除：

```
{% spaceless %}
    <strong>
        Hello
    </strong>
{% endspaceless %}

```

### ssi

从1.8以后不建议使用这标签建议不再使用，将会在Django 2.0以后会被删除。用 [`include`](#std:templatetag-include)标签代替。

将给定文件的内容输出到页面。

像一个简单的[`include`](#std:templatetag-include)标签，`{％ ssi ％}`在当前页面中的另一个文件 - 必须使用绝对路径指定：

```
{% ssi '/home/html/ljworld.com/includes/right_generic.html' %}

```

`ssi`的第一个参数可以是引用的文字或任何其他上下文变量。

如果给出了可选的`parsed`参数，则在当前上下文中，将包含文件的内容作为模板代码进行评估：

```
{% ssi '/home/html/ljworld.com/includes/right_generic.html' parsed %}

```

Note that if you use `{% ssi %}`, you’ll need to define `'allowed_include_roots'` in the [`OPTIONS`](../settings.html#std:setting-TEMPLATES-OPTIONS) of your template engine, as a security measure.

注意

使用[`ssi`](#std:templatetag-ssi)标记和`parsed`参数，文件之间没有共享状态 - 每个包含是一个完全独立的呈现过程。这意味着例如不可能使用包含的文件来定义块或改变当前页面中的上下文。

另请参阅：[`{% include %}`](#std:templatetag-include)。

### templatetag

输出用于构成模板标记的语法字符之一。

由于模板系统没有“转义”的概念，为了显示模板标签中使用的一个位，必须使用`{％ templatetag ％}`标记。

参数指定要输出哪个模板位：

<colgroup><col width="72%"> <col width="28%"></colgroup> 
| Argument | Outputs |
| --- | --- |
| `openblock` | `{%` |
| `closeblock` | `%}` |
| `openvariable` | `{{` |
| `closevariable` | `}}` |
| `openbrace` | `{` |
| `closebrace` | `}` |
| `opencomment` | `{#` |
| `closecomment` | `#}` |

样品用量：

```
{% templatetag openblock %} url 'entry_list' {% templatetag closeblock %}

```

### url

返回一个绝对路径的引用(不包含域名的URL)，该引用匹配一个给定的视图函数和一些可选的参数。在解析后返回的结果路径字符串中，每个特殊字符将使用[`iri_to_uri()`](../utils.html#django.utils.encoding.iri_to_uri "django.utils.encoding.iri_to_uri")编码。

这是一种不违反DRY原则的输出链接的方式，它可以避免在模板中硬编码链接路径。

```
{% url 'some-url-name' v1 v2 %}

```

第一个参数是视图函数中包名.模块名.函数名这样的路径`package.package.module.function`. 它可以是一个被引号引起来的字符串或者其他的上下文变量. 其他参数是可选的并且应该以空格隔开，这些值会在URL中以参数的形式传递. 上面的例子展示了如何传递位置参数.当然你也可以使用关键字参数.

```
{% url 'some-url-name' arg1=v1 arg2=v2 %}

```

不要把位置参数和关键字参数混在一起使用。URLconf所需的所有参数都应该存在。

例如，假设您有一个视图`app_views.client`，其URLconf接受客户端ID（此处`client()`是视图文件`app_views .py`）。URLconf行可能如下所示：

```
('^client/([0-9]+)/$', 'app_views.client', name='app-views-client')

```

如果你的应用中的URLconf 已经被包含到项目 URLconf 中，比如下面这样

```
('^clients/', include('project_name.app_name.urls'))

```

...那么, 在模板文件中, 你可以很方便的创建一个接指该视图的超链接，示例如下

```
{% url 'app-views-client' client.id %}

```

模板标签会输出如下的字符串 `/clients/client/123/`.

如果您使用[_命名的网址格式_](../../topics/http/urls.html#naming-url-patterns)，则可以参考`网址`标记中的模式名称，而不是使用视图的路径。

请注意，如果您要撤消的网址不存在，您会收到[`NoReverseMatch`](../exceptions.html#django.core.urlresolvers.NoReverseMatch "django.core.urlresolvers.NoReverseMatch")异常，这会导致您的网站显示错误网页。

如果您希望在不显示网址的情况下检索网址，则可以使用略有不同的调用：

```
{% url 'some-url-name' arg arg2 as the_url %}

<a href="{{ the_url }}">I'm linking to {{ the_url }}</a>

```

The scope of the variable created by the `as var` syntax is the `{% block %}` in which the `{% url %}` tag appears.

此`{％ url ...`为var％}语法将_不_导致错误，如果视图丢失。实际上，您将使用此链接来链接到可选的视图：

```
{% url 'some-url-name' as the_url %}
{% if the_url %}
  <a href="{{ the_url }}">Link to optional stuff</a>
{% endif %}

```

如果您要检索名称空间网址，请指定完全限定名称：

```
{% url 'myapp:view-name' %}

```

这将遵循正常的[_命名空间URL解析策略_](../../topics/http/urls.html#topics-http-reversing-url-namespaces)，包括使用上下文对当前应用程序提供的任何提示。

自1.8版起已弃用：点状Python路径语法已弃用，将在Django 2.0中删除：

```
{% url 'path.to.some_view' v1 v2 %}

```

警告

不要忘记在函数路径或模式名称周围加引号，否则值将被解释为上下文变量！

### verbatim

停止模版引擎在该标签中的渲染/

常见的用法是允许与Django语法冲突的JavaScript模板图层。例如：

```
{% verbatim %}
    {{if dying}}Still alive.{{/if}}
{% endverbatim %}

```

You can also designate a specific closing tag, allowing the use of `{% endverbatim %}` as part of the unrendered contents:

```
{% verbatim myblock %}
    Avoid template rendering via the {% verbatim %}{% endverbatim %} block.
{% endverbatim myblock %}

```

### widthratio

为了创建条形图等，此标签计算给定值与最大值的比率，然后将该比率应用于常量。

例如：

```
<img src="bar.png" alt="Bar"
     height="10" width="{% widthratio this_value max_value max_width %}" />

```

如果`this_value`是175，`max_value`是200，并且`max_width`是100，则上述示例中的图像将是88像素宽（因为175 / 200 = .875； .875 * 100 = 87.5，上舍入为88）。

在某些情况下，您可能想要捕获变量中的`widthratio`的结果。它可以是有用的，例如，在[`blocktrans`](../../topics/i18n/translation.html#std:templatetag-blocktrans)像这样：

```
{% widthratio this_value max_value max_width as width %}
{% blocktrans %}The width is: {{ width }}{% endblocktrans %}

```

Changed in Django 1.7:

添加了使用“as”与此标记一样的能力，如上例所示。

### 与

使用一个简单地名字缓存一个复杂的变量，当你需要使用一个“昂贵的”方法（比如访问数据库）很多次的时候是非常有用的

例如：

```
{% with total=business.employees.count %}
    {{ total }} employee{{ total|pluralize }}
{% endwith %}

```

`{％ 与 ％}之间可用填充变量（上例中`total` t5&gt;`和`{％ endwith ％}`

你可以分配多个上下文变量：

```
{% with alpha=1 beta=2 %}
    ...
{% endwith %}

```

注意

The previous more verbose format is still supported: `{% with business.employees.count as total %}`

## 内置过滤器参考 

### 加

把add后的参数加给value

例如:

```
{{ value|add:"2" }}

```

如果 `value` 为 `4`,则会输出 `6`.

过滤器首先会强制把两个值转换成Int类型。如果强制转换失败, 它会试图使用各种方式吧两个值相加。它会使用一些数据类型 (字符串, 列表, 等等.) 其他类型则会失败. 如果转换失败，结果会变成一个空字符串

例如，我们使用下面的值

```
{{ first|add:second }}

```

 `first` 是 `[1, 2, 3]`  ，`second` 是 `[4, 5, 6]`, 将会输出 `[1, 2, 3, 4, 5, 6]`.

警告

如果字符串可以被强制转换成int类型则会 **summed**，无法被转换，则和上面的第一个例子一样

### addslashes

在引号前面加上斜杆。例如，用于在CSV中转义字符串。

例如：

```
{{ value|addslashes }}

```

如果`value` 是 `"I'm using Django"`, 输出将变成 `"I\'m using Django"`.

### capfirst

大写变量的第一个字母。如果第一个字符不是字母，该过滤器将不会生效。

例如：

```
{{ value|capfirst }}

```

如果 `value` 是 `"django"`, 输出将变成 `"Django"`.

### 中央

使"value"在给定的宽度范围内居中.

例如:

```
"{{ value|center:"15" }}"

```

如果`value`是`"Django"`，输出将是`“ Django t7&gt;`。

### 切

移除value中所有的与给出的变量相同的字符串

例如：

```
{{ value|cut:" " }}

```

如果`value`为`“String 与 空格”`，输出将为`"Stringwithspaces"`。

### 日期

根据给定格式对一个date变量格式化

格式类似于 PHP 的 `date()` 函数 ([http://php.net/date](http://php.net/date)) ，在一些细节上有不同.

注意

这些格式字符不在模板外的Django中使用。它们被设计为与PHP兼容，以便为设计者轻松过渡。

可用的格式字符串：

<colgroup><col width="12%"> <col width="31%"> <col width="57%"></colgroup> 
| Format character | Description | Example output |
| --- | --- | --- |
| a | `'a.m.'` or `'p.m.'` (Note that this is slightly different than PHP’s output, because this includes periods to match Associated Press style.) | `'a.m.'` |
| A | `'AM'` or `'PM'`. | `'AM'` |
| b | Month, textual, 3 letters, lowercase. | `'jan'` |
| B | Not implemented. |   |
| c | ISO 8601 format. (Note: unlike others formatters, such as “Z”, “O” or “r”, the “c” formatter will not add timezone offset if value is a naive datetime (see [`datetime.tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo "(in Python v3.4)")). | `2008-01-02T10:30:00.000123+02:00`, or `2008-01-02T10:30:00.000123` if the datetime is naive |
| d | Day of the month, 2 digits with leading zeros. | `'01'` to `'31'` |
| D | Day of the week, textual, 3 letters. | `'Fri'` |
| e | Timezone name. Could be in any format, or might return an empty string, depending on the datetime. | `''`, `'GMT'`, `'-500'`, `'US/Eastern'`, etc. |
| E | Month, locale specific alternative representation usually used for long date representation. | `'listopada'` (for Polish locale, as opposed to `'Listopad'`) |
| f | Time, in 12-hour hours and minutes, with minutes left off if they’re zero. Proprietary extension. | `'1'`, `'1:30'` |
| F | Month, textual, long. | `'January'` |
| g | Hour, 12-hour format without leading zeros. | `'1'` to `'12'` |
| G | Hour, 24-hour format without leading zeros. | `'0'` to `'23'` |
| h | Hour, 12-hour format. | `'01'` to `'12'` |
| H | Hour, 24-hour format. | `'00'` to `'23'` |
| i | Minutes. | `'00'` to `'59'` |
| I | Daylight Savings Time, whether it’s in effect or not. | `'1'` or `'0'` |
| j | Day of the month without leading zeros. | `'1'` to `'31'` |
| l | Day of the week, textual, long. | `'Friday'` |
| L | Boolean for whether it’s a leap year. | `True` or `False` |
| m | Month, 2 digits with leading zeros. | `'01'` to `'12'` |
| M | Month, textual, 3 letters. | `'Jan'` |
| n | Month without leading zeros. | `'1'` to `'12'` |
| N | Month abbreviation in Associated Press style. Proprietary extension. | `'Jan.'`, `'Feb.'`, `'March'`, `'May'` |
| o | ISO-8601 week-numbering year, corresponding to the ISO-8601 week number (W) | `'1999'` |
| O | Difference to Greenwich time in hours. | `'+0200'` |
| P | Time, in 12-hour hours, minutes and ‘a.m.’/’p.m.’, with minutes left off if they’re zero and the special-case strings ‘midnight’ and ‘noon’ if appropriate. Proprietary extension. | `'1 a.m.'`, `'1:30 p.m.'`, `'midnight'`, `'noon'`, `'12:30 p.m.'` |
| r | [**RFC 2822**](http://tools.ietf.org/html/rfc2822.html) formatted date. | `'Thu, 21 Dec 2000 16:01:07 +0200'` |
| s | Seconds, 2 digits with leading zeros. | `'00'` to `'59'` |
| S | English ordinal suffix for day of the month, 2 characters. | `'st'`, `'nd'`, `'rd'` or `'th'` |
| t | Number of days in the given month. | `28` to `31` |
| T | Time zone of this machine. | `'EST'`, `'MDT'` |
| u | Microseconds. | `000000` to `999999` |
| U | Seconds since the Unix Epoch (January 1 1970 00:00:00 UTC). |   |
| w | Day of the week, digits without leading zeros. | `'0'` (Sunday) to `'6'` (Saturday) |
| W | ISO-8601 week number of year, with weeks starting on Monday. | `1`, `53` |
| y | Year, 2 digits. | `'99'` |
| Y | Year, 4 digits. | `'1999'` |
| z | Day of the year. | `0` to `365` |
| Z | Time zone offset in seconds. The offset for timezones west of UTC is always negative, and for those east of UTC is always positive. | `-43200` to `43200` |

例如：

```
{{ value|date:"D d M Y" }}

```

如果`value`是[`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)")对象（例如，`datetime.datetime.now()`的结果），输出将是字符串 `'Wed 09 Jan 2008'`。

传递的格式可以是预定义的格式[`DATE_FORMAT`](../settings.html#std:setting-DATE_FORMAT)，[`DATETIME_FORMAT`](../settings.html#std:setting-DATETIME_FORMAT)，[`SHORT_DATE_FORMAT`](../settings.html#std:setting-SHORT_DATE_FORMAT)或[`SHORT_DATETIME_FORMAT`](../settings.html#std:setting-SHORT_DATETIME_FORMAT)使用上表中显示的格式说明符。请注意，预定义的格式可能会根据当前语言环境而有所不同。

假设[`USE_L10N`](../settings.html#std:setting-USE_L10N)为`True`和[`LANGUAGE_CODE`](../settings.html#std:setting-LANGUAGE_CODE)为例如`"es"`

```
{{ value|date:"SHORT_DATE_FORMAT" }}

```

the output would be the string `"09/01/2008"` (the `"SHORT_DATE_FORMAT"` format specifier for the `es` locale as shipped with Django is `"d/m/Y"`).

不使用格式字符串时使用：

```
{{ value|date }}

```

...将使用[`DATE_FORMAT`](../settings.html#std:setting-DATE_FORMAT)设置中定义的格式化字符串，而不应用任何本地化。

您可以将`date`与[`time`](#std:templatefilter-time)过滤器结合使用，以呈现`datetime`值的完整表示形式。例如。：

```
{{ value|date:"D d M Y" }} {{ value|time:"H:i" }}

```

### 默认

如果value的计算结果为`False`，则使用给定的默认值。否则，使用该value。

例如：

```
{{ value|default:"nothing" }}

```

如果`value`为`""`（空字符串），则输出将为`nothing`。

### default_if_none

如果（且仅当）value为`None`，则使用给定的默认值。否则，使用该value。

注意，如果给出一个空字符串，默认值将_不_被使用。如果要回退空字符串，请使用[`default`](#std:templatefilter-default)过滤器。

例如：

```
{{ value|default_if_none:"nothing" }}

```

如果`value`为`None`，则输出将为字符串`“nothing”`。

### dictsort

接受一个字典列表，并返回按参数中给出的键排序后的列表。

例如：

```
{{ value|dictsort:"name" }}

```

如果`value`为：

```
[
    {'name': 'zed', 'age': 19},
    {'name': 'amy', 'age': 22},
    {'name': 'joe', 'age': 31},
]

```

那么输出将是：

```
[
    {'name': 'amy', 'age': 22},
    {'name': 'joe', 'age': 31},
    {'name': 'zed', 'age': 19},
]

```

你也可以做更复杂的事情，如：

```
{% for book in books|dictsort:"author.age" %}
    * {{ book.title }} ({{ book.author.name }})
{% endfor %}

```

如果`books`是：

```
[
    {'title': '1984', 'author': {'name': 'George', 'age': 45}},
    {'title': 'Timequake', 'author': {'name': 'Kurt', 'age': 75}},
    {'title': 'Alice', 'author': {'name': 'Lewis', 'age': 33}},
]

```

那么输出将是：

```
* Alice (Lewis)
* 1984 (George)
* Timequake (Kurt)

```

### dictsort翻转

获取字典列表，并返回按照参数中给出的键按相反顺序排序的列表。这与上面的过滤器完全相同，但返回的值将是相反的顺序。

### 可分割

如果value可以被给出的参数整除，则返回 `True`

例如：

```
{{ value|divisibleby:"3" }}

```

如果`value`是`21`，则输出将为`True`。

### 逃逸

转义字符串的HTML。具体来说，它使这些替换：

*   `&lt;`转换为`&lt;`
*   `&gt;`转换为`&gt;`
*   `'`（单引号）转换为`&#39;`
*   `"`（双引号）转换为`&quot;`
*   `&`转换为`&amp;`

转义仅在字符串输出时应用，因此在连接的过滤器序列中`escape`的位置无关紧要：它将始终应用，就像它是最后一个过滤器。如果要立即应用转义，请使用[`force_escape`](#std:templatefilter-force_escape)过滤器。

将`转义`应用于通常会对结果应用自动转义的变量只会导致一轮转义完成。因此，即使在自动逃逸环境中使用此功能也是安全的。如果要应用多个转义通过，请使用[`force_escape`](#std:templatefilter-force_escape)过滤器。

例如，您可以在[`autoescape`](#std:templatetag-autoescape)关闭时将`escape`应用于字段：

```
{% autoescape off %}
    {{ title|escape }}
{% endautoescape %}

```

### escapejs

转义用于JavaScript字符串的字符。这使_不_使字符串安全用于HTML，但确保在使用模板生成JavaScript / JSON时避免语法错误。

例如：

```
{{ value|escapejs }}

```

如果`value`为`“testing \ r \ njavascript \'string” ＆lt； b＆gt； escaping＆lt； / b＆ `，输出将为`“testing \\ u000D \\ u000Ajavascript \\ u0027string \\ u0022 \\ u003Cb \\ u003Eescaping \\ u003C / b \\ u003E“`。

### filesizeformat

格式化数值为“人类可读”的文件大小（例如`'13 KB'`, `'4.1 MB'`, `'102 bytes'`等）。

例如：

```
{{ value|filesizeformat }}

```

如果`value` 为123456789，输出将是`117.7 MB`。

文件大小和国际系统单位

严格地讲，`filesizeformat` 没有遵守国际单位系统建议的KiB、MiB、GiB等，它们使用1024 为幂（虽然这里使用的也是）。相反，Django 使用传统的更常用的单位名称（KB、MB、GB等）。

### 第一

返回列表中的第一项。

例如：

```
{{ value|first }}

```

如果`值`是列表`['a'， 'b'， 'c']` ，输出将为`'a'`。

### floatformat

当不使用参数时，将浮点数舍入到小数点后一位，但前提是要显示小数部分。例如：

<colgroup><col width="26%"> <col width="57%"> <col width="17%"></colgroup> 
| `value` | Template | Output |
| --- | --- | --- |
| `34.23234` | `{{ value&#124;floatformat }}` | `34.2` |
| `34.00000` | `{{ value&#124;floatformat }}` | `34` |
| `34.26000` | `{{ value&#124;floatformat }}` | `34.3` |

如果与数字整数参数一起使用，`floatformat`将数字四舍五入为小数位数。例如：

<colgroup><col width="24%"> <col width="57%"> <col width="20%"></colgroup> 
| `value` | Template | Output |
| --- | --- | --- |
| `34.23234` | `{{ value&#124;floatformat:3 }}` | `34.232` |
| `34.00000` | `{{ value&#124;floatformat:3 }}` | `34.000` |
| `34.26000` | `{{ value&#124;floatformat:3 }}` | `34.260` |

特别有用的是传递0（零）作为参数，它将使float浮动到最接近的整数。

<colgroup><col width="22%"> <col width="59%"> <col width="19%"></colgroup> 
| `value` | Template | Output |
| --- | --- | --- |
| `34.23234` | `{{ value&#124;floatformat:"0" }}` | `34` |
| `34.00000` | `{{ value&#124;floatformat:"0" }}` | `34` |
| `39.56000` | `{{ value&#124;floatformat:"0" }}` | `40` |

如果传递给`floatformat`的参数为负，则它会将一个数字四舍五入到小数点后的位置，但前提是要显示一个小数部分。例如：

<colgroup><col width="22%"> <col width="59%"> <col width="19%"></colgroup> 
| `value` | Template | Output |
| --- | --- | --- |
| `34.23234` | `{{ value&#124;floatformat:"-3" }}` | `34.232` |
| `34.00000` | `{{ value&#124;floatformat:"-3" }}` | `34` |
| `34.26000` | `{{ value&#124;floatformat:"-3" }}` | `34.260` |

使用没有参数的`floatformat`等效于使用具有`-1`的参数的`floatformat`。

### force_escape

将HTML转义应用于字符串（有关详细信息，请参阅[`escape`](#std:templatefilter-escape)过滤器）。此过滤器立即应用于，并返回一个新的转义字符串。这在需要多次转义或想要对转义结果应用其他过滤器的罕见情况下非常有用。通常，您要使用[`escape`](#std:templatefilter-escape)过滤器。

例如，如果您要捕获由[`linebreaks`](#std:templatefilter-linebreaks)过滤器创建的`&lt;p&gt;` HTML元素：

```
{% autoescape off %}
    {{ body|linebreaks|force_escape }}
{% endautoescape %}

```

### get_digit

给定一个整数，返回所请求的数字，其中1是最右边的数字，2是第二个最右边的数字等。返回无效输入的原始值（如果输入或参数不是整数，或参数小于1）。否则，输出总是一个整数。

例如：

```
{{ value|get_digit:"2" }}

```

如果`value`为`123456789`，则输出将为`8`。

### iriencode

将IRI（国际化资源标识符）转换为适合包含在URL中的字符串。如果您尝试在网址中使用包含非ASCII字符的字符串，这是必要的。

在已经通过[`urlencode`](#std:templatefilter-urlencode)过滤器的字符串上使用此过滤器是安全的。

例如：

```
{{ value|iriencode }}

```

如果`value`为`"?test=1&me=2"`，输出将为`"?test=1&amp;me=2"`

### 加入

使用字符串连接列表，例如Python的`str.join(list)`

例如：

```
{{ value|join:" // " }}

```

如果`value`是列表`['a'， 'b'， 'c'] / t2&gt;，输出将是字符串`“a // b // c“`。`

### 持续

返回列表中的最后一个项目。

例如：

```
{{ value|last }}

```

If `value` is the list `['a', 'b', 'c', 'd']`, the output will be the string `"d"`.

### 长度

返回值的长度。这适用于字符串和列表。

例如：

```
{{ value|length }}

```

如果`value`是`['a'， 'b'， 'c'， 'd']`或`"abcd"`，输出将为`4`。

Changed in Django 1.8:

对于未定义的变量，过滤器返回`0`。以前，它返回一个空字符串。

### length_is

如果值的长度是参数，则返回`True`，否则返回`False`。

例如：

```
{{ value|length_is:"4" }}

```

如果`value`是`['a'， 'b'， 'c'， 'd']`或`"abcd"`，输出将为`True`。

### linebreaks

用适当的HTML替换纯文本中的换行符；单个换行符变为HTML换行符（`＆lt； br /＆gt；`），新行后跟空行将成为段落（`&lt;/p&gt;`）。

例如：

```
{{ value|linebreaks }}

```

如果`value`为`Joel为 a slug`，输出将为`＆lt； p＆gt； Joel＆lt； br /＆gt；是 a slug＆lt； / p＆gt；`

### linebreaksbr

将纯文字中的所有换行符转换为HTML换行符（`＆lt； br /＆gt；`）。

例如：

```
{{ value|linebreaksbr }}

```

如果`value`为`Joel为 a slug`，输出将为`Joel＆lt； br /＆gt；是 a slug`

### 亚麻布

显示带行号的文本。

例如：

```
{{ value|linenumbers }}

```

如果`value`为：

```
one
two
three

```

输出将是：

```
1\. one
2\. two
3\. three

```

### ljust

将给定宽度的字段中的值左对齐。

**参数：**字段大小

例如：

```
"{{ value|ljust:"10" }}"

```

如果`value`为`Django`，则输出将为`“Django ”`。

### 降低

将字符串转换为全部小写。

例如：

```
{{ value|lower }}

```

如果`value`为`仍然 MAD 在 Yoko 输出将在`仍然 mad 在 yoko`。`

### make_list

返回转换为列表的值。对于字符串，它是一个字符列表。对于整数，在创建列表之前将参数强制转换为unicode字符串。

例如：

```
{{ value|make_list }}

```

如果`value`是字符串`"Joel"`，输出将是列表`['J'， 'o' ， 'e'， 'l']`。如果`value`是`123`，则输出将是列表`['1'， '2'， t15 &gt; '3']`。

### phone2numeric

将电话号码（可能包含字母）转换为其等效数字。

输入不必是有效的电话号码。这将很乐意转换任何字符串。

例如：

```
{{ value|phone2numeric }}

```

如果`value`为`800-COLLECT`，输出将为`800-2655328`。

### 复数

如果值不是1则返回一个复数形式通常用 `'s'`表示.

例：

```
You have {{ num_messages }} message{{ num_messages|pluralize }}.

```

如果`num_messages` 的值是 `1`, 那么将会输出`You have 1 message.` 如果`num_messages` 的值是 `2` 那么将会输出`You have 2 messages.`

另外如果你需要的不是 `'s'`后缀的话, 你可以提供一个备选的参数给过滤器

例：

```
You have {{ num_walruses }} walrus{{ num_walruses|pluralize:"es" }}.

```

对于非一般形式的复数,你可以同时指定 单复数形式，用逗号隔开.

例：

```
You have {{ num_cherries }} cherr{{ num_cherries|pluralize:"y,ies" }}.

```

注意

使用[`blocktrans`](../../topics/i18n/translation.html#std:templatetag-blocktrans)来翻译复数形式的字符串

### 打印

包装器[`pprint.pprint()`](https://docs.python.org/3/library/pprint.html#pprint.pprint "(in Python v3.4)") - 用于调试，真的。

### 随机

返回给定列表中的随机项。

例如：

```
{{ value|random }}

```

If `value` is the list `['a', 'b', 'c', 'd']`, the output could be `"b"`.

### removetags

自1.8版起已弃用：`removetags`无法保证HTML安全输出，因安全问题而被弃用。请考虑使用[漂白](http://bleach.readthedocs.org/en/latest/)。

从输出中删除[X] HTML标签的空格分隔列表。

例如：

```
{{ value|removetags:"b span" }}

```

If `value` is `"&lt;b&gt;Joel&lt;/b&gt; &lt;button&gt;is&lt;/button&gt; a &lt;span&gt;slug&lt;/span&gt;"` the unescaped output will be `"Joel &lt;button&gt;is&lt;/button&gt; a slug"`.

请注意，此过滤器区分大小写。

If `value` is `"&lt;B&gt;Joel&lt;/B&gt; &lt;button&gt;is&lt;/button&gt; a &lt;span&gt;slug&lt;/span&gt;"` the unescaped output will be `"&lt;B&gt;Joel&lt;/B&gt; &lt;button&gt;is&lt;/button&gt; a slug"`.

无安全保证

请注意，`removetags`不会保证其输出是HTML安全的。特别地，它不递归地工作，因此像`"&lt;sc&lt;script&gt;ript&gt;alert('XSS')&lt;/sc&lt;/script&gt;ript&gt;"`即使您应用`|removetags:"script"`也是安全的。因此，如果输入是用户提供的，则**NEVER**将`safe`过滤器应用于`removetags`输出。如果您正在寻找更强大的功能，可以使用`bleach` Python库，特别是其[清洁](http://bleach.readthedocs.org/en/latest/clean.html)方法。

### rjust

右对齐给定宽度字段中的值。

**参数：**字段大小

例如：

```
"{{ value|rjust:"10" }}"

```

如果`value`为`Django`，则输出将为`“ Django”`。

### 安全

将字符串标记为在输出之前不需要进一步的HTML转义。当自动转义关闭时，此过滤器不起作用。

注意

如果您要链接过滤器，在`safe`后应用的过滤器可能会使内容再次不安全。例如，以下代码按原样打印变量：

```
{{ var|safe|escape }}

```

### safeseq

将[`safe`](#std:templatefilter-safe)过滤器应用于序列的每个元素。与对序列进行操作的其他过滤器（例如[`join`](#std:templatefilter-join)）一起使用非常有用。例如：

```
{{ some_list|safeseq|join:", " }}

```

在这种情况下，不能直接使用[`safe`](#std:templatefilter-safe)过滤器，因为它首先将变量转换为字符串，而不是使用序列的各个元素。

### 片

返回列表的一部分。

使用与Python的列表切片相同的语法。有关介绍，请参见[http://www.diveintopython3.net/native-datatypes.html#slicinglists](http://www.diveintopython3.net/native-datatypes.html#slicinglists)。

例：

```
{{ some_list|slice:":2" }}

```

如果`some_list`是`['a'， 'b'， 'c'] &gt;，输出将为`['a'， 'b']`。`

### slugify

转换为ASCII。将空格转换为连字符。删除不是字母数字，下划线或连字符的字符。转换为小写。还剥离前导和尾随空格。

例如：

```
{{ value|slugify }}

```

如果`value`是`“Joel 是 a &gt;，输出将为`"joel-is-a-slug"`。`

### stringformat

根据参数格式化变量，一个字符串格式化说明符。此说明符使用Python字符串格式化语法，除了前导“％”被删除。

有关Python字符串格式的文档，请参见[https://docs.python.org/library/stdtypes.html#string-formatting-operations](https://docs.python.org/library/stdtypes.html#string-formatting-operations)

例如：

```
{{ value|stringformat:"E" }}

```

如果`value`为`10`，输出将为`1.000000E+01`。

### striptags

尽一切可能努力剥离所有[X] HTML标签。

例如：

```
{{ value|striptags }}

```

If `value` is `"&lt;b&gt;Joel&lt;/b&gt; &lt;button&gt;is&lt;/button&gt; a &lt;span&gt;slug&lt;/span&gt;"`, the output will be `"Joel is a slug"`.

无安全保证

请注意，`striptags`不会保证其输出是HTML安全的，尤其是对于无效的HTML输入。因此，**NEVER**将`safe`过滤器应用于`striptags`输出。如果您正在寻找更强大的功能，可以使用`bleach` Python库，特别是其[清洁](http://bleach.readthedocs.org/en/latest/clean.html)方法。

### 时间

根据给定的格式格式化时间。

给定格式可以是预定义的[`TIME_FORMAT`](../settings.html#std:setting-TIME_FORMAT)，也可以是与[`date`](#std:templatefilter-date)过滤器相同的自定义格式。请注意，预定义的格式是与区域设置相关的。

例如：

```
{{ value|time:"H:i" }}

```

如果`value`等效于`datetime.datetime.now()`，则输出将为字符串`"01:23"`。

另一个例子：

假设[`USE_L10N`](../settings.html#std:setting-USE_L10N)为`True`且[`LANGUAGE_CODE`](../settings.html#std:setting-LANGUAGE_CODE)为例如`"de"`

```
{{ value|time:"TIME_FORMAT" }}

```

输出将是字符串`"01:23:00"`（与Django一起提供的`de`语言环境的`"TIME_FORMAT"`格式说明`"H:i:s"`）。

`time`过滤器只接受格式字符串中与时间相关的参数，而不是日期（由于显而易见的原因）。If you need to format a `date` value, use the [`date`](#std:templatefilter-date) filter instead (or along `time` if you need to render a full [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime "(in Python v3.4)") value).

There is one exception the above rule: When passed a `datetime` value with attached timezone information (a [_time-zone-aware_](../../topics/i18n/timezones.html#naive-vs-aware-datetimes) `datetime` instance) the `time` filter will accept the timezone-related [_format specifiers_](#date-and-time-formatting-specifiers) `'e'`, `'O'` , `'T'` and `'Z'`.

不使用格式字符串时使用：

```
{{ value|time }}

```

...将使用[`TIME_FORMAT`](../settings.html#std:setting-TIME_FORMAT)设置中定义的格式化字符串，而不应用任何本地化。

Changed in Django 1.7:

在Django 1.7中添加了接收和操作带有时区信息的值的能力。

### 时光

将日期格式设为自该日期起的时间（例如，“4天，6小时”）。

采用一个可选参数，它是一个包含用作比较点的日期的变量（不带参数，比较点为_现在_）。例如，如果`blog_date`是表示2006年6月1日午夜的日期实例，并且`comment_date`是2006年6月1日08:00的日期实例，则以下将返回“8小时”：

```
{{ blog_date|timesince:comment_date }}

```

比较offset-naive和offset-aware datatimes将返回一个空字符串。

分钟是所使用的最小单位，对于相对于比较点的未来的任何日期，将返回“0分钟”。

### 时间

类似于`timesince`，除了它测量从现在开始直到给定日期或日期时间的时间。例如，如果今天是2006年6月1日，而`conference_date`是保留2006年6月29日的日期实例，则`{{ conference_date | timeuntil }}`将返回“4周”。

使用可选参数，它是一个包含用作比较点的日期（而不是_现在_）的变量。如果`from_date`包含2006年6月22日，则以下内容将返回“1周”：

```
{{ conference_date|timeuntil:from_date }}

```

比较offset-naive和offset-aware datatimes将返回一个空字符串。

分钟是使用的最小单位，对于过去的任何相对于比较点的日期，将返回“0分钟”。

### 标题

使字符以大写字符开头，其余字符小写，将字符串转换为titlecase。此标记不会努力保持“小写字”小写。

例如：

```
{{ value|title }}

```

如果`value`为`“我 FIRST post”`，输出将为 `“我的 第一 帖子”`。

### 截短体

如果字符串字符多于指定的字符数量，那么会被截断。截断的字符串将以可翻译的省略号序列（“...”）结尾。

**参数：**要截断的字符数

例如：

```
{{ value|truncatechars:9 }}

```

如果`value`是`“Joel 是 a &gt;，输出将为`“Joel i ...”`。`

### truncatechars_html

New in Django 1.7.

类似于[`truncatechars`](#std:templatefilter-truncatechars)，除了它知道HTML标记。在字符串中打开并且在截断点之前未关闭的任何标记在截断后立即关闭。

例如：

```
{{ value|truncatechars_html:9 }}

```

If `value` is `"&lt;p&gt;Joel is a slug&lt;/p&gt;"`, the output will be `"&lt;p&gt;Joel i...&lt;/p&gt;"`.

HTML内容中的换行符将保留。

### 截断字

在一定数量的字后截断字符串。

**参数：**要截断的字数

例如：

```
{{ value|truncatewords:2 }}

```

如果`value`是`“Joel 是 a &gt;，输出将是`“Joel 是 ...”`。`

字符串中的换行符将被删除。

### truncatewords_html

类似于[`truncatewords`](#std:templatefilter-truncatewords)，除了它知道HTML标记。在字符串中打开并且在截断点之前未关闭的任何标记在截断后立即关闭。

这比[`truncatewords`](#std:templatefilter-truncatewords)效率较低，因此只应在传递HTML文本时使用。

例如：

```
{{ value|truncatewords_html:2 }}

```

If `value` is `"&lt;p&gt;Joel is a slug&lt;/p&gt;"`, the output will be `"&lt;p&gt;Joel is ...&lt;/p&gt;"`.

HTML内容中的换行符将保留。

### unordered_list

接收一个嵌套的列表，返回一个HTML 的列表 —— 不包含开始和结束的&lt;ul&gt; 标签。

列表假设成具有合法的格式。例如，如果`var` 包含`['States', ['Kansas', ['Lawrence', 'Topeka'], 'Illinois']]`， 那么`{{ var|unordered_list }}` 将返回：

```
<li>States
<ul>
        <li>Kansas
        <ul>
                <li>Lawrence</li>
                <li>Topeka</li>
        </ul>
        </li>
        <li>Illinois</li>
</ul>
</li>

```

自1.8版起已弃用：还支持旧的，更限制性和详细的输入格式：`['States'， [['Kansas'， t&gt; []]， ['Topeka'， []]]]， t7&gt; []]]]`。这种语法在Django 2.0 中将不再支持。

### 上

将字符串转换为大写形式：

例如：

```
{{ value|upper }}

```

如果`value`是`“Joel 是 a &gt;，输出将为`“JOEL IS A SLUG”`。`

### urlencode

转义要在URL中使用的值。

例如：

```
{{ value|urlencode }}

```

If `value` is `"http://www.example.org/foo?a=b&c=d"`, the output will be `"http%3A//www.example.org/foo%3Fa%3Db%26c%3Dd"`.

可以提供包含不应该转义的字符的可选参数。

如果未提供，则'/'字符被假定为安全的。当_所有_字符应该转义时，可以提供空字符串。例如：

```
{{ value|urlencode:"" }}

```

如果`value`为`"http://www.example.org/"`，输出将为`"http%3A%2F%2Fwww.example.org%2F"`。

### urlize

将文字中的网址和电子邮件地址转换为可点击的链接。

此模板代码适用于以`http://`，`https://`或`www.`为前缀的链接。例如，`http://goo.gl/aia1t`会得到转换，但`goo.gl/aia1t`不会。

它还支持以原始顶级域（`.com`，`.edu`，`.gov`，`.int`，`.mil`，`.net`和`.org`）。例如，`djangoproject.com`被转换。

Changed in Django 1.8:

支持添加包含顶级域名后面的字符（例如，`djangoproject.com/`和`djangoproject.com/download/`）的纯域链接。

链接可以具有结尾标点符号（句点，逗号，近括号）和前导标点符号（开头括号），`urlize`仍然可以做正确的事。

由`urlize`生成的链接会向其中添加`rel="nofollow"`属性。

例如：

```
{{ value|urlize }}

```

如果`value`是`“检查 出 www.djangoproject.com” 将`“检查 出 ＆lt； a href =”http://www.djangoproject.com“ t10&gt; rel =“nofollow”＆gt； www.djangoproject.com＆lt； / a＆gt；“`。`

除了网络链接，`urlize`还会将电子邮件地址转换为`mailto:`链接。如果`value`是`“发送 问题 到 foo@example.com” t6 &gt;`，输出将是`“发送 问题 到 ＆lt； a href =“mailto：foo@example.com”＆gt； foo@example.com&lt； / a＆gt；“`。

`urlize`过滤器还采用可选参数`autoescape`。如果`autoescape`是`True`，则使用Django的内置[`escape`](#std:templatefilter-escape)过滤器转义链接文字和网址。`autoescape`的默认值为`True`。

注意

如果`urlize`应用于已经包含HTML标记的文本，则会无法正常工作。仅将此过滤器应用于纯文本。

### urlizetrunc

将网址和电子邮件地址转换为可点击的链接，就像[urlize](#urlize)，但截断长度超过给定字符数限制的网址。

**参数：**链接文本的字符数应截短为，包括如果截断是必要的，添加的省略号。

例如：

```
{{ value|urlizetrunc:15 }}

```

If `value` is `"Check out www.djangoproject.com"`, the output would be `'Check out &lt;a href="http://www.djangoproject.com" rel="nofollow"&gt;www.djangopr...&lt;/a&gt;'`.

与[urlize](#urlize)一样，此过滤器应仅应用于纯文本。

### 字数

返回字数。

例如：

```
{{ value|wordcount }}

```

如果`value`是`“Joel 是 a &gt;，输出将为`4`。`

### wordwrap

以指定的行长度换行单词。

**参数：**用于换行文本的字符数

例如：

```
{{ value|wordwrap:5 }}

```

如果`value`是`Joel 是 a slug 输出将是：`

```
Joel
is a
slug

```

### yesno

将值“`True`，`False`和（可选）`None`映射到字符串”yes“，”no“，”maybe“自定义映射作为逗号分隔列表传递，并根据值返回其中一个字符串：

例如:

```
{{ value|yesno:"yeah,no,maybe" }}

```

<colgroup><col width="13%"> <col width="29%"> <col width="57%"></colgroup> 
| Value | Argument | Outputs |
| --- | --- | --- |
| `True` |   | `yes` |
| `True` | `"yeah,no,maybe"` | `yeah` |
| `False` | `"yeah,no,maybe"` | `no` |
| `None` | `"yeah,no,maybe"` | `maybe` |
| `None` | `"yeah,no"` | `no` (converts `None` to `False` if no mapping for `None` is given) |

## 国际化标签和过滤器

Django提供模板标记和过滤器，以控制模板中[_internationalization_](../../topics/i18n/index.html)的每个方面。它们允许对翻译，格式化和时区转换进行粒度控制。

### i18n

此库允许在模板中指定可翻译文本。要启用它，请将[`USE_I18N`](../settings.html#std:setting-USE_I18N)设置为`True`，然后加载`{％ 加载 i18n ％}`。

请参阅[_Internationalization: in template code_](../../topics/i18n/translation.html#specifying-translation-strings-in-template-code)中。

### l10n

此库提供对模板中值的本地化的控制。您只需要使用`{％ 加载 l10n ％}`但您通常会将[`USE_L10N`](../settings.html#std:setting-USE_L10N)设置为`True`，以便本地化默认处于活动状态。

请参阅[_Controlling localization in templates_](../../topics/i18n/formatting.html#topic-l10n-templates)。

### tz

此库提供对模板中时区转换的控制。像`l10n`，您只需要使用`{％ 加载 tz }`，但通常还会将[`USE_TZ`](../settings.html#std:setting-USE_TZ)设置为`True`，以便默认情况下会转换为本地时间。

请参阅[_Time zone aware output in templates_](../../topics/i18n/timezones.html#time-zones-in-templates)。

## 其他标签和过滤器库

Django附带了一些其他模板标记库，您必须在[`INSTALLED_APPS`](../settings.html#std:setting-INSTALLED_APPS)设置中显式启用，并在您的模板中启用[`{% load %}`](#std:templatetag-load)标记。

### django.contrib.humanize

一组Django模板过滤器，用于向数据添加“人性化”。请参阅[_django.contrib.humanize_](../contrib/humanize.html)。

### django.contrib.webdesign

在设计网站时有用的模板标签集合，例如Lorem Ipsum文本生成器。请参阅[_django.contrib.webdesign_](../contrib/webdesign.html)。

### 静态的

#### 静态的

要链接保存在[`STATIC_ROOT`](../settings.html#std:setting-STATIC_ROOT)中的静态文件，Django附带了[`static`](#std:templatetag-static)模板标记。无论是否使用[`RequestContext`](api.html#django.template.RequestContext "django.template.RequestContext")，您都可以使用此方法。

```
{% load static %}
<img src="{% static "images/hi.jpg" %}" alt="Hi!" />

```

它还能够消耗标准上下文变量，例如。假设将`user_stylesheet`变量​​传递给模板：

```
{% load static %}
<link rel="stylesheet" href="{% static user_stylesheet %}" type="text/css" media="screen" />

```

如果您希望在不显示静态网址的情况下检索静态网址，则可以使用略有不同的调用：

```
{% load static %}
{% static "images/hi.jpg" as myphoto %}
<img src="{{ myphoto }}"></img>

```

注意

The [`staticfiles`](../contrib/staticfiles.html#module-django.contrib.staticfiles "django.contrib.staticfiles: An app for handling static files.") contrib app also ships with a [`static template tag`](../contrib/staticfiles.html#std:templatetag-staticfiles-static) which uses `staticfiles'` [`STATICFILES_STORAGE`](../settings.html#std:setting-STATICFILES_STORAGE) to build the URL of the given path (rather than simply using [`urllib.parse.urljoin()`](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.urljoin "(in Python v3.4)") with the [`STATIC_URL`](../settings.html#std:setting-STATIC_URL) setting and the given path). 如果您有高级用例（例如[_using a cloud service to serve static files_](../../howto/static-files/deployment.html#staticfiles-from-cdn)），请改用此方法：

```
{% load static from staticfiles %}
<img src="{% static "images/hi.jpg" %}" alt="Hi!" />

```

#### get_static_prefix

You should prefer the [`static`](#std:templatetag-static) template tag, but if you need more control over exactly where and how [`STATIC_URL`](../settings.html#std:setting-STATIC_URL) is injected into the template, you can use the [`get_static_prefix`](#std:templatetag-get_static_prefix) template tag:

```
{% load static %}
<img src="{% get_static_prefix %}images/hi.jpg" alt="Hi!" />

```

还有一个第二种形式，你可以使用，以避免额外的处理，如果你需要多次的价值：

```
{% load static %}
{% get_static_prefix as STATIC_PREFIX %}

<img src="{{ STATIC_PREFIX }}images/hi.jpg" alt="Hi!" />
<img src="{{ STATIC_PREFIX }}images/hi2.jpg" alt="Hello!" />

```

#### get_media_prefix

类似于[`get_static_prefix`](#std:templatetag-get_static_prefix)，`get_media_prefix`填充媒体前缀为[`MEDIA_URL`](../settings.html#std:setting-MEDIA_URL)的模板变量，例如：

```
{% load static %}
<body data-media-url="{% get_media_prefix %}">

```

通过将值存储在数据属性中，如果我们想在JavaScript上下文中使用它，我们确保它适当地转义。

{% endraw %}