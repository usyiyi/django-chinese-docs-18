# Django中的密码管理

密码管理在非必要情况下一般不会重新发明，Django致力于提供一套安全、灵活的工具集来管理用户密码。本文档描述Django存储密码和hash存储方法配置的方式，以及使用hash密码的一些实例。

另见

即使用户可能会使用强密码，攻击者也可能窃听到他们的连接。使用[_HTTPS_](../security.html#security-recommendation-ssl)来避免在HTTP连接上发送密码（或者任何敏感的数据），因为否则密码又被嗅探的风险。

## Django如何储存密码

Django通常使用PBKDF2来提供灵活的密码储存系统。

[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User") 对象的[`password`](../../ref/contrib/auth.html#django.contrib.auth.models.User.password "django.contrib.auth.models.User.password")属性是一个这种格式的字符串：

```
<algorithm>$<iterations>$<salt>$<hash>

```

那些就是用于储存用户密码的部分，以美元字符分分隔。它们由哈希算法、算法迭代次数（工作因数）、随机的salt、以及生成的密码哈希值组成。算法是Django可以使用的，单向哈希或者密码储存算法之一，请见下文。迭代描述了算法在哈希上执行的次数。salt是随机的种子值，哈希值是这个单向函数的结果。

通常，Django以SHA256的哈希值使用[PBKDF2](http://en.wikipedia.org/wiki/PBKDF2)算法，由[NIST](http://csrc.nist.gov/publications/nistpubs/800-132/nist-sp800-132.pdf)推荐的一种密码伸缩机制。这对于大多数用户都很有效：它非常安全，需要大量的计算来破解。

然而，取决于你的需求，你可以选择一个不同的算法，或者甚至使用自定义的算法来满足你的特定的安全环境。不过，大多数用户并不需要这样做 -- 如果你不确定，最好不要这样。如果你打算这样做，请继续阅读：

DJango通过访问[`PASSWORD_HASHERS`](../../ref/settings.html#std:setting-PASSWORD_HASHERS)设置来选择要使用的算法。这里有一个列表，列出了Django支持的哈希算法类。列表的第一个元素 (即`settings.`PASSWORD_HASHERS[0]) 会用于储存密码， 所有其它元素都是用于验证的哈希值，它们可以用于检查现有的密码。意思是如果你打算使用不同的算法，你需要修改[`PASSWORD_HASHERS`](../../ref/settings.html#std:setting-PASSWORD_HASHERS)，来将你最喜欢的算法在列表中放在首位。

[`PASSWORD_HASHERS`](../../ref/settings.html#std:setting-PASSWORD_HASHERS)默认为：

```
PASSWORD_HASHERS = (
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    'django.contrib.auth.hashers.BCryptPasswordHasher',
    'django.contrib.auth.hashers.SHA1PasswordHasher',
    'django.contrib.auth.hashers.MD5PasswordHasher',
    'django.contrib.auth.hashers.CryptPasswordHasher',
)

```

这意味着，Django会使用 [PBKDF2](http://en.wikipedia.org/wiki/PBKDF2) 储存所有密码，但是支持使用 PBKDF2SHA1, [bcrypt](http://en.wikipedia.org/wiki/Bcrypt), [SHA1](http://en.wikipedia.org/wiki/SHA1)等等算法来检查储存的密码。下一节会描述一些通用的方法，高级用户可能想通过它来修改这个设置。

### 在Django中使用bcrypt

[Bcrypt](http://en.wikipedia.org/wiki/Bcrypt)是一种流行的密码储存算法，它特意被设计用于长期的密码储存。Django并没有默认使用它，由于它需要使用三方的库，但是由于很多人都想使用它，Django会以最小的努力来支持。

执行以下步骤来作为你的默认储存算法来使用Bcrypt：

1.  安装[bcrypt 库](https://pypi.python.org/pypi/bcrypt/)。这可以通过运行`pip install django[bcrypt]`,，或者下载并运行 `python setup.py install`来实现。

2.  修改 [`PASSWORD_HASHERS`](../../ref/settings.html#std:setting-PASSWORD_HASHERS) ，将 `BCryptSHA256PasswordHasher`放在首位。也就是说，在你的设置文件中应该：

    ```
    PASSWORD_HASHERS = (
        'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
        'django.contrib.auth.hashers.BCryptPasswordHasher',
        'django.contrib.auth.hashers.PBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
        'django.contrib.auth.hashers.SHA1PasswordHasher',
        'django.contrib.auth.hashers.MD5PasswordHasher',
        'django.contrib.auth.hashers.CryptPasswordHasher',
    )

    ```

    (你应该将其它元素留在列表中，否则Django不能升级密码；见下文)。

配置完毕 -- 现在Django会使用Bcrypt作为默认的储存算法。

BCryptPasswordHasher的密码截断

bcrypt的设计者会在72个字符处截断所有的密码，这意味着`bcrypt(password_with_100_chars) == bcrypt(password_with_100_chars[:72])`。原生的 `BCryptPasswordHasher` 并不会做任何的特殊处理， 所以它也会受到这一隐藏密码长度限制的约束。`BCryptSHA256PasswordHasher` 通过事先使用&nbsp;sha256生成哈希来解决这一问题。这样就可以防止密码截断了，所以你还是应该优先考虑`BCryptPasswordHasher`。这个截断带来的实际效果很微不足道，因为大多数用户不会使用长度超过72的密码，并且即使在72个字符处截断，破解brypt所需的计算能力依然是天文数字。虽然如此，我们还是推荐使用`BCryptSHA256PasswordHasher` ，根据 “有备无患”的原则。

其它 bcrypt 的实现

有一些其它的bcrypt&nbsp;实现，可以让你在Django中使用它。Django的bcrypt 支持并不直接兼容这些实现。你需要修改数据库中的哈希值，改为&nbsp;`bcrypt$(raw bcrypt output)`的形式，来升级它们。例如： `bcrypt$$2a$12$NT0I31Sa7ihGEWpka9ASYrEFkhuTNeBQ2xfZskIiiJeyFXhRgS.Sy`。

### 增加工作因数

PBKDF2 和bcrypt 算法使用大量的哈希迭代或循环。这会有意拖慢攻击者，使对哈希密码的攻击更难以进行。然而，随着计算机能力的不断增加，迭代的次数也需要增加。我们选了一个合理的默认值（并且在Django的每个发行版会不断增加），但是你可能想要调高或者调低它，取决于你的安全需求和计算能力。要想这样做，你可以继承相应的算法，并且覆写`iterations`参数。例如，增加PBKDF2算法默认使用的迭代次数：

1.  创建`django.contrib.auth.hashers.PBKDF2PasswordHasher`的子类：

    ```
    from django.contrib.auth.hashers import PBKDF2PasswordHasher

    class MyPBKDF2PasswordHasher(PBKDF2PasswordHasher):
        """
        A subclass of PBKDF2PasswordHasher that uses 100 times more iterations.
        """
        iterations = PBKDF2PasswordHasher.iterations * 100

    ```

    把它保存在项目中的某个位置。例如，把它放在类似于`myproject/hashers.py`的文件中。

2.  将你的新的hasher作为第一个元素添加到[`PASSWORD_HASHERS`](../../ref/settings.html#std:setting-PASSWORD_HASHERS)：

    ```
    PASSWORD_HASHERS = (
        'myproject.hashers.MyPBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2PasswordHasher',
        'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
        'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
        'django.contrib.auth.hashers.BCryptPasswordHasher',
        'django.contrib.auth.hashers.SHA1PasswordHasher',
        'django.contrib.auth.hashers.MD5PasswordHasher',
        'django.contrib.auth.hashers.CryptPasswordHasher',
    )

    ```

配置完毕 -- 现在DJango在储存使用PBKDF2的密码时会使用更多的迭代次数。

### 密码升级

用户登录之后，如果他们的密码没有以首选的密码算法来储存，Django会自动将算法升级为首选的那个。这意味着Django中旧的安装会在用户登录时自动变得更加安全，并且你可以随意在新的（或者更好的）储存算法发明之后切换到它们。

然而，Django只会升级在&nbsp;[`PASSWORD_HASHERS`](../../ref/settings.html#std:setting-PASSWORD_HASHERS)中出现的算法，所以升级到新系统时，你应该确保不要&nbsp;_移除_列表中的元素。如果你移除了，使用列表中没有的算法的用户不会被升级。修改PBKDF2迭代次数之后，密码也会被升级。

## Manually managing a user’s password

[`django.contrib.auth.hashers`](#module-django.contrib.auth.hashers "django.contrib.auth.hashers")模块提供了一系列的函数来创建和验证哈希密码。你可以独立于`User`模型之外使用它们。

`check_password`(_password_, _encoded_)[[source]](../../_modules/django/contrib/auth/hashers.html#check_password)

如果你打算通过比较纯文本密码和数据库中哈希后的密码来手动验证用户，要使用[`check_password()`](#django.contrib.auth.hashers.check_password "django.contrib.auth.hashers.check_password")这一便捷的函数。它接收两个参数：要检查的纯文本密码，和数据库中用户的`password`字段的完整值。如果二者匹配，返回`True` ，否则返回`False` 。

`make_password`(_password_, _salt=None_, _hasher='default'_)[[source]](../../_modules/django/contrib/auth/hashers.html#make_password)

以当前应用所使用的格式创建哈希密码。它接受一个必需参数：纯文本密码。如果你不想使用默认值（`PASSWORD_HASHERS`设置的首选项），你可以提供salt值和要使用的哈希算法，它们是可选的。当前支持的算法是： `'pbkdf2_sha256'`, `'pbkdf2_sha1'`, `'bcrypt_sha256'` (参见[_在 Django中使用Bcrypt_](#bcrypt-usage)), `'bcrypt'`, `'sha1'`, `'md5'`, `'unsalted_md5'` (仅仅用于向后兼容) 和 `'crypt'` （如果你安装了 `crypt`库）。如果password参数是`None`，会返回一个不可用的密码（它永远不会被[`check_password()`](#django.contrib.auth.hashers.check_password "django.contrib.auth.hashers.check_password")接受）。

`is_password_usable`(_encoded_password_)[[source]](../../_modules/django/contrib/auth/hashers.html#is_password_usable)

检查提供的字符串是否是可以用[`check_password()`](#django.contrib.auth.hashers.check_password "django.contrib.auth.hashers.check_password")验证的哈希密码。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Password management](https://docs.djangoproject.com/en/1.8/topics/auth/passwords/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
