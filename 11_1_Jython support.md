

# Running Django on Jython

[Jython](http://www.jython.org/) 是在 Java 平台 (JVM) 运行的 Python 实现。这个文档将让你在Jython之上运行Django。

## Installing Jython

Django使用Jython 2.7b2及更高版本。有关下载和安装说明，请参阅[Jython](http://www.jython.org/)网站。

## Creating a servlet container

如果你只是想试验Django，请跳到下一节； Django包含一个可以用于测试的轻量级Web服务器，因此在准备在生产环境中部署Django之前，您不需要设置任何其他内容。

如果要在生产站点上使用Django，请使用Java servlet容器，例如[Apache Tomcat](http://tomcat.apache.org/)。如果您需要其包含的额外功能，则完整的JavaEE应用程序服务器（如[GlassFish](https://glassfish.java.net/)或[JBoss](http://www.jboss.org/)）也可以。

## Installing Django

下一步是安装Django本身。这与在标准Python上安装Django完全相同，因此请参阅[_Remove any old versions of Django_](../topics/install.html#removing-old-versions-of-django)和[_Install the Django code_](../topics/install.html#install-django-code)以获取相关说明。

## Installing Jython platform support libraries

[django-jython](http://code.google.com/p/django-jython/)项目包含用于Django / Jython开发的数据库后端和管理命令。注意，内置的Django后端不会在Jython之上工作。

要安装它，请按照项目网站上详细的[安装说明](https://pythonhosted.org/django-jython/quickstart.html#install)操作。另外，阅读[数据库后端](https://pythonhosted.org/django-jython/database-backends.html)文档。

## Differences with Django on Jython

此时，Jython上的Django应该与运行在标准Python上的Django几乎相同。但是，有几点差异需要牢记：

*   请记住使用`jython`命令而不是`python`。文档使用`python`来保持一致性，但是如果你使用的是Jython，你会想每次发生时用`jython`来替代`python`。
*   同样，您需要使用`JYTHONPATH`环境变量，而不是`PYTHONPATH`。
*   Django中需要[枕头](http://pillow.readthedocs.org/en/latest/)的任何部分都将无法工作。

