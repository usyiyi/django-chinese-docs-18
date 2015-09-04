# File对象 #

`django.core.files`模块及其子模块包含了一些用于基本文件处理的内建类。

## File类 ##

`class File(file_object)`

`File` 类是Python [file 对象](https://docs.python.org/3/glossary.html#term-file-object)的一个简单封装，并带有Django特定的附加功能。需要表示文件的时候，Django内部会使用这个类。

File对象拥有下列属性和方法：

`name`

含有`MEDIA_ROOT`相对路径的文件名称。

`size`

文件的字节数。

`file`

这个类所封装的，原生的[file 对象](https://docs.python.org/3/glossary.html#term-file-object)。

`mode`

文件的读写模式。

`open([mode=None])`

打开或者重新打开文件（同时会执行`File.seek(0)`）。 `mode`参数的值和Python内建的`open()`相同。

重新打开一个文件时，无论文件原先以什么模式打开，`mode`都会覆盖；`None`的意思是以原先的模式重新打开。

`read([num_bytes=None])`

读取文件内容。可选的`size`参数是要读的字节数；没有指定的话，文件会一直读到结尾。

`__iter__()`

迭代整个文件，并且每次生成一行。

```
Changed in Django 1.8:

File现在使用[通用的换行符](https://www.python.org/dev/peps/pep-0278)。以下字符会识别为换行符：Unix换行符'\n'，WIndows换行符'\r\n'，以及Macintosh旧式换行符'\r'。
```

`chunks([chunk_size=None])`

迭代整个文件，并生成指定大小的一部分内容。`chunk_size`默认为64 KB。

处理大文件时这会非常有用，因为这样可以把他们从磁盘中读取出来，而避免将整个文件存到内存中。

`multiple_chunks([chunk_size=None])`

如果文件足够大，需要按照提供的`chunk_size`切分成几个部分来访问到所有内容，则返回`True` 。

`write([content])`

将指定的内容字符串写到文件。取决于底层的储存系统，写入的内容在调用`close()`之前可能不会完全提交。

`close()`

关闭文件。

除了这些列出的方法，`File`暴露了 `file` 对象的以下属性和方法：`encoding`, `fileno`, `flush`, `isatty`, `newlines`, `read`, `readinto`, `readlines`, `seek`, `softspace`, `tell`, `truncate`, `writelines`, `xreadlines`。

## ContentFile类 ##

`class ContentFile(File)[source]`

`ContentFile`类继承自`File`，但是并不像`File`那样，它操作字符串的内容（也支持字节集），而不是一个实际的文件。例如：

```
from __future__ import unicode_literals
from django.core.files.base import ContentFile

f1 = ContentFile("esta sentencia está en español")
f2 = ContentFile(b"these are bytes")
```

## ImageFile类 ##

`class ImageFile(file_object)[source]`

Django特地为图像提供了这个内建类。`django.core.files.images.ImageFile`继承了 `File`的所有属性和方法，并且额外提供了以下的属性：

`width`

图像的像素单位宽度。

`height`

图像的像素单位高度。

## 附加到对象的文件的额外方法 ##

任何关联到一个对象（比如下面的`Car.photo`）的`File`都会有一些额外的方法：

`File.save(name, content[, save=True])`

以提供的文件名和内容保存一个新文件。这样不会替换已存在的文件，但是会创建新的文件，并且更新对象来指向它。如果`save`为`True`，模型的`save()`方法会在文件保存之后调用。这就是说，下面两行：

```
>>> car.photo.save('myphoto.jpg', content, save=False)
>>> car.save()
```

等价于：

```
>>> car.photo.save('myphoto.jpg', content, save=True)
```

要注意`content`参数必须是`File`或者 `File`的子类的实例，比如`ContentFile`。

`File.delete([save=True])`

从模型实例中移除文件，并且删除内部的文件。如果`save`是`True`，模型的`save()` 方法会在文件删除之后调用。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[File objects](https://docs.djangoproject.com/en/1.8/ref/files/file/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
