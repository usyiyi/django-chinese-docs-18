<!--
  译者：Github@wizardforcel
-->

# 将遗留数据库整合到Django #

虽然Django最适合用来开发新的应用，但也可以将它整合到遗留的数据库中。Django包含了很多工具，尽可能自动化解决这类问题。

这篇文章假设你了解Django的基础部分，它们在教程中提及。

一旦你的Django环境建立好之后，你可以按照这个大致的流程，整合你的现有数据库。

## 向Django提供你的数据库参数 ##

你需要告诉Django你的数据库连接参数，以及数据库的名称。请修改DATABASES设置，为'默认' 连接的以下键赋值：

+ NAME
+ ENGINE
+ USER
+ PASSWORD
+ HOST
+ PORT

## 自动生成模型 ##

Django自带叫做inspectdb的工具，可以按照现有的数据库创建模型。你可以运行以下命令，并查看输出：

```
$ python manage.py inspectdb
```

通过重定向Unix标准输出流来保存文件：

```
$ python manage.py inspectdb > models.py
```

这个特性是一个快捷方式，并不是一个确定的模型生成器。详见inspectdb文档 。

一旦你创建好了你的模型，把文件命名为models.py，然后把它放到你应用的Python包中。然后把应用添加到你的INSTALLED_APPS 设置中。

默认情况下，inspectdb创建未被管理的模型。这就是说，模型的Meta类中的managed = False告诉Django不要管理每个表的创建、修改和删除：

```
class Person(models.Model):
    id = models.IntegerField(primary_key=True)
    first_name = models.CharField(max_length=70)
    class Meta:
       managed = False
       db_table = 'CENSUS_PERSONS'
```

如果你希望Django管理表的生命周期，你需要把managed选项改为 True（或者简单地把它移除，因为True是默认值）。

## 安装Django核心表 ##

接下来，运行migrate命令来安装所有所需的额外的数据库记录，比如后台权限和内容类型：

```
$ python manage.py migrate
```

## 测试和调整 ##

上面就是所有基本的步骤了 —— 到目前为止你会想要调整Django自动生成的模型，直到他们按照你想要的方式工作。尝试通过Django数据库API访问你的数据，并且尝试使用Django后台页面编辑对象，以及相应地编辑模型文件。