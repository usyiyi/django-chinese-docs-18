# 文件储存API #

## 获取当前的储存类 ##

Django提供了两个便捷的方法来获取当前的储存类：

`class DefaultStorage[source]`

`DefaultStorage` 提供对当前的默认储存系统的延迟访问，像`DEFAULT_FILE_STORAGE`中定义的那样。`DefaultStorage` 内部使用了`get_storage_class()`。

`get_storage_class([import_path=None])[source]`

返回实现储存API的类或者模块。

当没有带着`import_path` 参数调用的时候， `get_storage_class` 会返回当前默认的储存系统，像`DEFAULT_FILE_STORAGE`中定义的那样。如果提供了`import_path`， `get_storage_class`会尝试从提供的路径导入类或者模块，并且如果成功的话返回它。如果导入不成功会抛出异常。

## FileSystemStorage类 ##

`class FileSystemStorage([location=None, base_url=None, file_permissions_mode=None, directory_permissions_mode=None])[source]`

`FileSystemStorage`类在本地文件系统上实现了基本的文件存储功能。它继承自`Storage` ，并且提供父类的所有公共方法的实现。

`location`

储存文件的目录的绝对路径。默认为`MEDIA_ROOT`设置的值。

`base_url`

在当前位置提供文件储存的URL。默认为`MEDIA_URL`设置的值。

`file_permissions_mode`

文件系统的许可，当文件保存时会接收到它。默认为`FILE_UPLOAD_PERMISSIONS`。

```
New in Django 1.7:

新增了file_permissions_mode属性。之前，文件总是会接收到FILE_UPLOAD_PERMISSIONS许可。
```

`directory_permissions_mode`

文件系统的许可，当目录保存时会接收到它。默认为`FILE_UPLOAD_DIRECTORY_PERMISSIONS`。

```
New in Django 1.7:

新增了directory_permissions_mode属性。之前，目录总是会接收到FILE_UPLOAD_DIRECTORY_PERMISSIONS许可。
```

> 注意
>
> `FileSystemStorage.delete()`在提供的文件名称不存在的时候并不会抛出任何异常。

## Storage类 ##

`class Storage[source]`

`Storage`类为文件的存储提供了标准化的API，并带有一系列默认行为，所有其它的文件存储系统可以按需继承或者复写它们。

> 注意
>
> 对于返回原生`datetime`对象的方法，所使用的有效时区为`os.environ['TZ']`的当前值。要注意它总是可以通过Django的`TIME_ZONE`来设置。

`accessed_time(name)[source]`

返回包含文件的最后访问时间的原生`datetime`对象。对于不能够返回最后访问时间的储存系统，会抛出`NotImplementedError`异常。

`created_time(name)[source]`

返回包含文件创建时间的原生`datetime`对象。对于不能够返回创建时间的储存系统，会抛出`NotImplementedError`异常。

`delete(name)[source]`

删除`name`引用的文件。如果目标储存系统不支持删除操作，会抛出`NotImplementedError`异常。

`exists(name)[source]`

如果提供的名称所引用的文件在文件系统中存在，则返回`True`，否则如果这个名称可用于新文件，返回`False`。

`get_available_name(name, max_length=None)[source]`

返回基于`name`参数的文件名称，它在目标储存系统中可用于写入新的内容。

如果提供了`max_length`，文件名称长度不会超过它。如果不能找到可用的、唯一的文件名称，会抛出`SuspiciousFileOperation` 异常。

如果`name`命名的文件已存在，一个下划线加上随机7个数字或字母的字符串会添加到文件名称的末尾，扩展名之前。

```
Changed in Django 1.7:

之前，下划线和一位数字（比如"_1"，"_2"，以及其他）会添加到文件名称的末尾，直到目标目录中发现了可用的名称。一些恶意的用户会利用这一确定性的算法来进行dos攻击。这一变化也在1.6.6， 1.5.9， 和 1.4.14中出现。
```

```
Changed in Django 1.8:

新增了max_length参数。
```

`get_valid_name(name)[source]`

返回基于`name`参数的文件名称，它适用于目标储存系统。

`listdir(path)[source]`

列出特定目录的所有内容，返回一个包含2元组的列表；第一个元素是目录，第二个是文件。对于不能够提供列表功能的储存系统，抛出`NotImplementedError`异常。

`modified_time(name)[source]`

返回包含最后修改时间的原生`datetime`对象。对于不能够返回最后修改时间的储存系统，抛出`NotImplementedError`异常。

`open(name, mode='rb')[source]`

通过提供的`name`打开文件。注意虽然返回的文件确保为`File`对象，但可能实际上是它的子类。在远程文件储存的情况下，这意味着读写操作会非常慢，所以警告一下。

`path(name)[source]`

本地文件系统的路径，文件可以用Python标准的`open()`在里面打开。对于不能从本地文件系统访问的储存系统，抛出`NotImplementedError`异常。

`save(name, content, max_length=None)[source]`

使用储存系统来保存一个新文件，最好带有特定的名称。如果名称为 `name`的文件已存在，储存系统会按需修改文件名称来获取一个唯一的名称。返回被储存文件的实际名称。

max_length参数会传递给`get_available_name()`。

`content`参数必须为`django.core.files.File`或者`File`子类的实例。

```
Changed in Django 1.8:

新增了max_length参数。
```

`size(name)[source]`

返回`name`所引用的文件的总大小，以字节为单位。对于不能够返回文件大小的储存系统，抛出`NotImplementedError`异常。

`url(name)[source]`

返回URL，通过它可以访问到`name`所引用的文件。对于不支持通过URL访问的储存系统，抛出`NotImplementedError`异常。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Storage API](https://docs.djangoproject.com/en/1.8/ref/files/storage/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
