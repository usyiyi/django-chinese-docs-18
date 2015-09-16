# 加密签名 #

web应用安全的黄金法则是，永远不要相信来自不可信来源的数据。有时通过不可信的媒介来传递数据会非常方便。密码签名后的值可以通过不受信任的途径传递，这样是安全的，因为任何篡改都会检测的到。

Django提供了用于签名的底层API，以及用于设置和读取被签名cookie的上层API，它们是web应用中最常使用的签名工具之一。

你可能会发现，签名对于以下事情非常有用：

+ 生成用于“重置我的账户”的URL，并发送给丢失密码的用户。
+ 确保储存在隐藏表单字段的数据不被篡改，
+ 生成一次性的秘密URL，用于暂时性允许访问受保护的资源，例如用户付费的下载文件。

## 保护 SECRET_KEY ##

当你使用 startproject创建新的Django项目时，自动生成的 settings.py文件会得到一个随机的SECRET_KEY值。这个值是保护签名数据的密钥 -- 它至关重要，你必须妥善保管，否则攻击者会使用它来生成自己的签名值。

## 使用底层 API ##

Django的签名方法存放于`django.core.signing`模块。首先创建一个 `Signer` 的实例来对一个值签名：

```
>>> from django.core.signing import Signer
>>> signer = Signer()
>>> value = signer.sign('My string')
>>> value
'My string:GdMGD6HNQ_qdgxYP8yBZAdAIV1w'
```

这个签名会附加到字符串末尾，跟在冒号后面。你可以使用`unsign`方法来获取原始的值：

```
>>> original = signer.unsign(value)
>>> original
'My string'
```

如果签名或者值以任何方式改变，会抛出`django.core.signing.BadSignature` 异常：

```
>>> from django.core import signing
>>> value += 'm'
>>> try:
...    original = signer.unsign(value)
... except signing.BadSignature:
...    print("Tampering detected!")
```

通常，`Signer`类使用`SECRET_KEY`设置来生成签名。你可以通过向`Signer`构造器传递一个不同的密钥来使用它：

```
>>> signer = Signer('my-other-secret')
>>> value = signer.sign('My string')
>>> value
'My string:EkfQJafvGyiofrdGnuthdxImIJw'
```

`class Signer(key=None, sep=':', salt=None)[source]`

返回一个`signer`，它使用`key` 来生成签名，并且使用`sep`来分割值。`sep` 不能是 [URL安全的base64字母表(http://tools.ietf.org/html/rfc4648#section-5)]中的字符。字母表含有数字、字母、连字符和下划线。

### 使用salt参数 ###

如果你不希望对每个特定的字符串都生成一个相同的签名哈希值，你可以在`Signer`类中使用可选的`salt` 参数。使用`salt`参数会同时用它和`SECRET_KEY`初始化签名哈希函数：

```
>>> signer = Signer()
>>> signer.sign('My string')
'My string:GdMGD6HNQ_qdgxYP8yBZAdAIV1w'
>>> signer = Signer(salt='extra')
>>> signer.sign('My string')
'My string:Ee7vGi-ING6n02gkcJ-QLHg6vFw'
>>> signer.unsign('My string:Ee7vGi-ING6n02gkcJ-QLHg6vFw')
'My string'
```

以这种方法使用`salt`会把不同的签名放在不同的命名空间中。来自于单一命名空间（一个特定的`salt`值）的签名不能用于在不同的命名空间中验证相同的纯文本字符串。不同的命名空间使用不同的`salt`设置。这是为了防止攻击者使用在一个地方的代码中生成的签名后的字符串，作为使用不同`salt`来生成（和验证）签名的另一处代码的输入。

不像你的`SECRET_KEY`，你的`salt`参数可以不用保密。

### 验证带有时间戳的值 ###

`TimestampSigner`是 `Signer`的子类，它向值附加一个签名后的时间戳。这可以让你确认一个签名后的值是否在特定时间段之内被创建：

```
>>> from datetime import timedelta
>>> from django.core.signing import TimestampSigner
>>> signer = TimestampSigner()
>>> value = signer.sign('hello')
>>> value
'hello:1NMg5H:oPVuCqlJWmChm1rA2lyTUtelC-c'
>>> signer.unsign(value)
'hello'
>>> signer.unsign(value, max_age=10)
...
SignatureExpired: Signature age 15.5289158821 > 10 seconds
>>> signer.unsign(value, max_age=20)
'hello'
>>> signer.unsign(value, max_age=timedelta(seconds=20))
'hello'
```

`class TimestampSigner(key=None, sep=':', salt=None)[source]`

`sign(value)[source]`

签名`value`，并且附加当前的时间戳。

`unsign(value, max_age=None)[source]`

检查`value`是否在少于`max_age` 秒之前被签名，如果不是则抛出`SignatureExpired`异常。`max_age` 参数接受一个整数或者`datetime.timedelta`对象。

```
Changed in Django 1.8:

在此之前， max_age参数只接受整数。
```

### 保护复杂的数据结构 ###

如果你希望保护一个列表、元组或字典，你可以使用签名模块的`dumps` 和 `loads` 函数来实现。它们模仿了Python的`pickle`模块，但是在背后使用了`JSON`序列化。`JSON`可以确保即使你的`SECRET_KEY`被盗取，攻击者并不能利用`pickle`的格式来执行任意的命令：

```
>>> from django.core import signing
>>> value = signing.dumps({"foo": "bar"})
>>> value
'eyJmb28iOiJiYXIifQ:1NMg1b:zGcDE4-TCkaeGzLeW9UQwZesciI'
>>> signing.loads(value)
{'foo': 'bar'}
```

由于`JSON`的本质（列表和元组之间没有原生的区别），如果你传进来一个元组，你会从`signing.loads(object)`得到一个列表：

```
>>> from django.core import signing
>>> value = signing.dumps(('a','b','c'))
>>> signing.loads(value)
['a', 'b', 'c']
```

`dumps(obj, key=None, salt='django.core.signing', compress=False)[source]`

返回URL安全，sha1签名的base64压缩的JSON字符串。序列化的对象使用`TimestampSigner`来签名。

`loads(string, key=None, salt='django.core.signing', max_age=None)[source]`

`dumps()`的反转，如果签名失败则抛出`BadSignature`异常。如果提供了`max_age`则会检查它（以秒为单位）。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Cryptographic signing](https://docs.djangoproject.com/en/1.8/topics/signing/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
