# 部署静态文件 #

> 另见
>
> `django.contrib.staticfiles` 的用法简介，请参见[管理静态文件（CSS、images）](http://python.usyiyi.cn/django/howto/static-files/index.html)。

## 在线上环境部署静态文件 ##

放置静态文件到线上环境的基本步骤很简单：当静态文件改变时，运行`collectstatic` 命令，然后安排将收集好的静态文件的目录(`STATIC_ROOT`) 搬到静态文件服务器上。取决于`STATICFILES_STORAGE`，这些文件可能需要手工移动到一个新的位置或者`Storage` 类的`post_process` 方法可以帮你。

当然，与所有的部署任务一样，魔鬼隐藏在细节中。每个线上环境的建立都会有所不同，所以你需要调整基本的纲要以适应你的需求。下面是一些常见的方法，可能有所帮助。

### 网站和静态文件位于同一台服务器上 ###

如果你的静态文件和网站位于同一台服务器，流程可能像是这样：

+ 将你的代码推送到部署的服务器上。
+ 在这台服务器上，运行`collectstatic` 来收集所有的静态文件到`STATIC_ROOT`。
+ 配置Web 服务器来托管URL` STATIC_URL`下的`STATIC_ROOT`。 例如，这是[如何使用Apache 和mod_wsgi 来完成它](http://python.usyiyi.cn/django/howto/deployment/wsgi/modwsgi.html#serving-files)。

你可能想自动化这个过程，特别是如果你有多台Web 服务器。有许多种方法来完成这个自动化，但是许多Django 开发人员喜欢 [Fabric](http://fabfile.org/)。

在一下的小节中，我们将演示一些示例的Fabric 脚本来自动化不同选择的文件部署。Fabric 脚本的语法相当简单，但这里不会讲述；参见[Fabric 的文档](http://docs.fabfile.org/) 以获得其语法的完整解释。

所以，一个部署静态文件来多台Web 服务器上的Fabric 脚本大概会是：

```
from fabric.api import *

# Hosts to deploy onto
env.hosts = ['www1.example.com', 'www2.example.com']

# Where your project code lives on the server
env.project_root = '/home/www/myproject'

def deploy_static():
    with cd(env.project_root):
        run('./manage.py collectstatic -v0 --noinput')
```

### 静态文件位于一台专门的服务器上 ##

大部分大型的Django 站点都使用一台单独的Web 服务器来存放静态文件 —— 例如一台不运行Django 的服务器。这种服务器通常运行一种不同类型的服务器 —— 更快但是功能很少。一些常见的选择有：

+ [Nginx](http://wiki.nginx.org/Main)
+ 裁剪版的[Apache](http://httpd.apache.org/)

配置这些服务器在这篇文档范围之外；查看每种服务器各自的文档以获得说明。

既然你的静态文件服务器不会允许Django，你将需要修改的部署策略，大概会是这样：

+ 当静态文件改变时，在本地运行`collectstatic`。
+ 将你本地的`STATIC_ROOT` 推送到静态文件服务器相应的目录中。在这一步，常见的选择[rsync](https://rsync.samba.org/) ，因为它只传输静态文件改变的部分。

下面是Fabric 脚本大概的样子：

```
from fabric.api import *
from fabric.contrib import project

# Where the static files get collected locally. Your STATIC_ROOT setting.
env.local_static_root = '/tmp/static'

# Where the static files should go remotely
env.remote_static_root = '/home/www/static.example.com'

@roles('static')
def deploy_static():
    local('./manage.py collectstatic')
    project.rsync_project(
        remote_dir = env.remote_static_root,
        local_dir = env.local_static_root,
        delete = True
    )
```

### 静态文件位于一个云服务或CDN 上 ###

两位一个常见的策略是放置静态文档到一个云存储提供商比如亚马逊的S3 和/或一个CDN(Content Delivery Network)上。这让你可以忽略保存静态文件的问题，并且通常可以加快网页的加载（特别是使用CDN 的时候）。

当使用这些服务时，除了不是使用rsync 传输你的静态文件到服务器上而是到存储提供商或CDN 上之外，基本的工作流程和上面的差不多。

有许多方式可以实现它，但是如果提供商具有API，那么自定义的文件存储后端 将使得这个过程相当简单。如果你已经写好或者正在使用第三方的自定义存储后端，你可以通过设置`STATICFILES_STORAGE` 来告诉`collectstatic` 来使用它。

例如，如果你已经在`myproject.storage.S3Storage` 中写好一个S3 存储的后端，你可以这样使用它：

```
STATICFILES_STORAGE = 'myproject.storage.S3Storage'
```

一旦完成这个，你所要做的就是运行`collectstatic`，然后你的静态文件将被你的存储后端推送到S3 上。如果以后你需要切换到一个不同的存储提供商，你只需简单地修改你的`STATICFILES_STORAGE` 设置。

关于如何编写这些后端的细节，请参见[编写一个自定义的存储系统](http://python.usyiyi.cn/django/howto/custom-file-storage.html)。有第三方的应用提供存储后端，它们支持许多常见的文件存储API。一个不错的入口是[djangopackages.com 的概览](https://www.djangopackages.com/grids/g/storage-backends/)。

## 了解更多 ##

关于`django.contrib.staticfiles` 中包含的设置、命令、模板标签和其它细节，参见[staticfiles 参考](http://python.usyiyi.cn/django/ref/contrib/staticfiles.html)。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Deploying static files](https://docs.djangoproject.com/en/1.8/howto/static-files/deployment/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
