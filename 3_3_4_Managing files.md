# 管理文件 #

这篇文档描述了Django为那些用户上传文件准备的文件访问API。底层的API足够通用，你可以使用为其它目的来使用它们。如果你想要处理静态文件（JS，CSS，以及其他），参见[管理静态文件（CSS和图像）](http://python.usyiyi.cn/django/howto/static-files/index.html)。

通常，Django使用`MEDIA_ROOT `和 `MEDIA_URL`设置在本地储存文件。下面的例子假设你使用这些默认值。

然而，Django提供了一些方法来编写自定义的 [文件储存系统](http://python.usyiyi.cn/django/topics/files.html#file-storage)，允许你完全自定义Django在哪里以及如何储存文件。这篇文档的另一部分描述了这些储存系统如何工作。

## 在模型中使用文件 ##

当你使用`FileField` 或者 `ImageField`的时候，Django为你提供了一系列的API用来处理文件。

考虑下面的模型，它使用`ImageField`来储存一张照片：

```
from django.db import models

class Car(models.Model):
    name = models.CharField(max_length=255)
    price = models.DecimalField(max_digits=5, decimal_places=2)
    photo = models.ImageField(upload_to='cars')
```

任何`Car`的实例都有一个 `photo`字段，你可以通过它来获取附加图片的详细信息：

```
>>> car = Car.objects.get(name="57 Chevy")
>>> car.photo
<ImageFieldFile: chevy.jpg>
>>> car.photo.name
'cars/chevy.jpg'
>>> car.photo.path
'/media/cars/chevy.jpg'
>>> car.photo.url
'http://media.example.com/cars/chevy.jpg'
```

例子中的`car.photo` 对象是一个`File` 对象，这意味着它拥有下面描述的所有方法和属性。

> 注意
>
> 文件保存是数据库模型保存的一部分，所以磁盘上真实的文件名在模型保存之前并不可靠。

例如，你可以通过设置文件的 `name`属性为一个和文件储存位置 （`MEDIA_ROOT`，如果你使用默认的`FileSystemStorage`）相关的路径，来修改文件名称。

```
>>> import os
>>> from django.conf import settings
>>> initial_path = car.photo.path
>>> car.photo.name = 'cars/chevy_ii.jpg'
>>> new_path = settings.MEDIA_ROOT + car.photo.name
>>> # Move the file on the filesystem
>>> os.rename(initial_path, new_path)
>>> car.save()
>>> car.photo.path
'/media/cars/chevy_ii.jpg'
>>> car.photo.path == new_path
True
```

## File ##

当Django需要表示一个文件的时候，它在内部使用`django.core.files.File`实例。这个对象是 Python [内建文件对象](https://docs.python.org/library/stdtypes.html#bltin-file-objects)的一个简单封装，并带有一些Django特定的附加功能。

大多数情况你可以简单地使用Django提供给你的`File`对象（例如像上面那样把文件附加到模型，或者是上传的文件）。

如果你需要自行构造一个`File`对象，最简单的方法是使用Python内建的`file` 对象来创建一个：

```
>>> from django.core.files import File

# Create a Python file object using open()
>>> f = open('/tmp/hello.world', 'w')
>>> myfile = File(f)
```

现在你可以使用 `File`类的任何文档中记录的属性和方法了。

注意这种方法创建的文件并不会自动关闭。以下步骤可以用于自动关闭文件：

```
>>> from django.core.files import File

# Create a Python file object using open() and the with statement
>>> with open('/tmp/hello.world', 'w') as f:
...     myfile = File(f)
...     myfile.write('Hello World')
...
>>> myfile.closed
True
>>> f.closed
True
```

在处理大量对象的循环中访问文件字段时，关闭文件极其重要。如果文件在访问之后没有手动关闭，会有消耗完文件描述符的风险。这可能导致如下错误：

```
IOError: [Errno 24] Too many open files
```

## 文件储存 ##

在背后，Django需要决定在哪里以及如何将文件储存到文件系统。这是一个对象，它实际上理解一些东西，比如文件系统，打开和读取文件，以及其他。

Django的默认文件储存由`DEFAULT_FILE_STORAGE`设置提供。如果你没有显式提供一个储存系统，就会使用它。

关于内建的默认文件储存系统的细节，请参见下面一节。另外，关于编写你自己的文件储存系统的一些信息，请见[编写自定义的文件系统](http://python.usyiyi.cn/django/howto/custom-file-storage.html)。

### 储存对象 ###

大多数情况你可能并不想使用`File`对象（它向文件提供适当的存储功能），你可以直接使用文件储存系统。你可以创建一些自定义文件储存类的实例，或者 – 大多数情况更加有用的 – 你可以使用全局的默认储存系统：

```
>>> from django.core.files.storage import default_storage
>>> from django.core.files.base import ContentFile

>>> path = default_storage.save('/path/to/file', ContentFile('new content'))
>>> path
'/path/to/file'

>>> default_storage.size(path)
11
>>> default_storage.open(path).read()
'new content'

>>> default_storage.delete(path)
>>> default_storage.exists(path)
False
```

关于文件储存API，参见 [文件储存API](http://python.usyiyi.cn/django/ref/files/storage.html)。

### 内建的文件系统储存类 ###

Django自带了`django.core.files.storage.FileSystemStorage` 类，它实现了基本的本地文件系统中的文件储存。

例如，下面的代码会在 `/media/photos` 目录下储存上传的文件，无论`MEDIA_ROOT`设置是什么：

```
from django.db import models
from django.core.files.storage import FileSystemStorage

fs = FileSystemStorage(location='/media/photos')

class Car(models.Model):
    ...
    photo = models.ImageField(storage=fs)
```

[自定义储存系统](http://python.usyiyi.cn/django/howto/custom-file-storage.html) 以相同方式工作：你可以把它们作为`storage`参数传递给`FileField`。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Managing files](https://docs.djangoproject.com/en/1.8/topics/files/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
