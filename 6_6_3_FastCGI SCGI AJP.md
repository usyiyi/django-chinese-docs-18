{% raw %}

# 如何使用Django与FastCGI，SCGI或AJP

自1.7版起已弃用：FastCGI支持已弃用，将在Django 1.9中删除。

虽然[_WSGI_](wsgi/index.html)是Django的首选部署平台，但许多人使用共享托管，其中诸如FastCGI，SCGI或AJP等协议是唯一可行的选项。

注意

本文档主要关注FastCGI。还通过`flup` Python包支持其他协议，如SCGI和AJP。有关SCGI和AJP的详细信息，请参阅下面的[协议](#protocols)部分。

本质上，FastCGI是一种有效的方式，让外部应用程序将页面提供给Web服务器。Web服务器将传入的Web请求（通过套接字）委派给FastCGI，FastCGI执行代码并将响应传递回Web服务器，Web服务器反过来将其传递回客户端的Web浏览器。

像WSGI一样，FastCGI允许代码留在内存中，允许在没有启动时间的情况下提供请求。而例如。 [_mod_wsgi_](wsgi/modwsgi.html)可以嵌入Apache Web服务器进程或作为单独的守护进程配置，FastCGI进程永远不会在Web服务器进程内运行，始终在一个单独的持久进程中。

为什么在单独的进程中运行代码？

Apache中的传统`mod_*`安排将各种脚本语言（最值得注意的是PHP，Python和Perl）嵌入到Web服务器的进程空间中。虽然这会降低启动时间 - 因为代码不必为每个请求读取磁盘 - 它是以内存使用为代价的。

由于FastCGI的性质，甚至可能有不同于Web服务器进程的用户帐户运行的进程。这对共享系统是一个很好的安全好处，因为它意味着你可以保护你的代码与其他用户。

## 先决条件：flup

在开始使用FastCGI和Django之前，您需要安装[flup](http://www.saddi.com/software/flup/)，一个用于处理FastCGI的Python库。版本0.5或更高版本应该工作正常。

## 启动FastCGI服务器

FastCGI在客户端 - 服务器模型上运行，在大多数情况下，您将自己启动FastCGI进程。当服务器需要加载动态页面时，您的Web服务器（无论是Apache，lighttpd还是其他）只会联系您的Django-FastCGI进程。因为守护程序已经在内存中运行代码，所以它能够很快地提供响应。

注意

如果你在共享托管系统上，你可能会被迫使用Web服务器管理的FastCGI进程。有关更多信息，请参阅下面关于使用Web服务器管理的进程运行Django的部分。

Web服务器可以通过以下两种方式之一连接到FastCGI服务器：它可以使用Unix域套接字（Win32系统上的“命名管道”），也可以使用TCP套接字。你选择的是一种偏好的方式；TCP套接字通常更容易由于权限问题。

要启动您的服务器，首先切换到项目的目录（[_manage.py_](../../ref/django-admin.html)），然后运行[`runfcgi`](../../ref/django-admin.html#django-admin-runfcgi)命令：

```
./manage.py runfcgi [options]

```

如果您在[`runfcgi`](../../ref/django-admin.html#django-admin-runfcgi)之后指定`help`作为唯一选项，它将显示所有可用选项的列表。

您需要指定[`socket`](../../ref/django-admin.html#django-admin-option-socket)，[`protocol`](../../ref/django-admin.html#django-admin-option-protocol)或[`host`](../../ref/django-admin.html#django-admin-option-host)和[`port`](../../ref/django-admin.html#django-admin-option-port)。然后，当您设置Web服务器时，您只需将其指向在启动FastCGI服务器时指定的主机/端口或套接字。请参阅下面的[示例](#examples)。

### 协议

Django支持[flup](http://www.saddi.com/software/flup/)的所有协议，即[fastcgi](http://www.fastcgi.com/)，[SCGI](http://python.ca/scgi/protocol.txt)和[AJP1.3](http://tomcat.apache.org/connectors-doc/ajp/ajpv13a.html)（Apache JServ协议，版本1.3）。通过使用[`protocol=&lt;protocol_name&gt;`](../../ref/django-admin.html#django-admin-option-protocol)选项和`./ manage.py runfcgi`选择您的首选协议 - 其中`&lt;protocol_name&gt;`可以是以下之一：`fcgi`（默认值），`scgi`或`ajp`。例如：

```
./manage.py runfcgi protocol=scgi

```

### 例子

在TCP端口上运行线程服务器：

```
./manage.py runfcgi method=threaded host=127.0.0.1 port=3033

```

在Unix域套接字上运行预分拣的服务器：

```
./manage.py runfcgi method=prefork socket=/home/user/mysite.sock pidfile=django.pid

```

套接字安全

Django的默认umask要求web服务器和Django fastcgi进程使用同一组**和**用户运行。为了增加安全性，您可以在同一组下运行它们，但作为不同的用户。如果这样做，您需要使用`runfcgi`的`umask`参数将umask设置为0002。

运行没有守护进程（后台）的进程（有利于调试）：

```
./manage.py runfcgi daemonize=false socket=/tmp/mysite.sock maxrequests=1

```

### 停止FastCGI守护程序

如果您的进程在前台运行，可以轻松停止它：只需点击`Ctrl-C`将停止并退出FastCGI服务器。但是，当您处理后台进程时，您需要使用Unix `kill`命令。

如果您为[`runfcgi`](../../ref/django-admin.html#django-admin-runfcgi)指定[`pidfile`](../../ref/django-admin.html#django-admin-option-pidfile)选项，则可以杀死正在运行的FastCGI守护程序，如下所示：

```
kill `cat $PIDFILE`

```

...其中`$PIDFILE`是您指定的`pidfile`。

```
#!/bin/bash

# Replace these three settings.
PROJDIR="/home/user/myproject"
PIDFILE="$PROJDIR/mysite.pid"
SOCKET="$PROJDIR/mysite.sock"

cd $PROJDIR
if [ -f $PIDFILE ]; then
    kill `cat -- $PIDFILE`
    rm -f -- $PIDFILE
fi

exec /usr/bin/env - \
  PYTHONPATH="../python:.." \
  ./manage.py runfcgi socket=$SOCKET pidfile=$PIDFILE

```

## Apache设置

要在Apache和FastCGI上使用Django，您需要安装并配置Apache，并安装并启用[mod_fastcgi](http://www.fastcgi.com/mod_fastcgi/docs/mod_fastcgi.html)。有关说明，请参阅Apache文档。

设置完成后，通过编辑`httpd.conf`（Apache配置）文件将Apache指向Django FastCGI实例。你需要做两件事情：

*   使用`FastCGIExternalServer`指令指定FastCGI服务器的位置。
*   使用`mod_rewrite`可根据需要将网址指向FastCGI。

### 指定FastCGI服务器的位置

`FastCGIExternalServer`指令告诉Apache如何找到您的FastCGI服务器。如[FastCGIExternalServer docs](http://www.fastcgi.com/mod_fastcgi/docs/mod_fastcgi.html#FastCgiExternalServer)说明，您可以指定`socket`或`host`。这里有两个例子：

```
# Connect to FastCGI via a socket / named pipe.
FastCGIExternalServer /home/user/public_html/mysite.fcgi -socket /home/user/mysite.sock

# Connect to FastCGI via a TCP host/port.
FastCGIExternalServer /home/user/public_html/mysite.fcgi -host 127.0.0.1:3033

```

在任一情况下，文件`/home/user/public_html/mysite.fcgi`实际上不必存在。（更多在此在下一节）。

### 使用mod_rewrite将URL指向FastCGI

第二步是告诉Apache对匹配某种模式的URL使用FastCGI。为此，请使用[mod_rewrite](http://httpd.apache.org/docs/2.0/mod/mod_rewrite.html)模块并将URL重写到`mysite.fcgi`（或您在`FastCGIExternalServer`指令中指定的任何内容，上一节）。

在本示例中，我们告诉Apache使用FastCGI来处理不表示文件系统上的文件并且不以`/media/`开头的任何请求。这可能是最常见的情况，如果你使用Django的管理网站：

```
<VirtualHost 12.34.56.78>
  ServerName example.com
  DocumentRoot /home/user/public_html
  Alias /media /home/user/python/django/contrib/admin/media
  RewriteEngine On
  RewriteRule ^/(media.*)$ /$1 [QSA,L,PT]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteRule ^/(.*)$ /mysite.fcgi/$1 [QSA,L]
</VirtualHost>

```

在使用[`{% url %}`](../../ref/templates/builtins.html#std:templatetag-url)构建网址时，Django会自动使用网址的预先重写版本。模板标签（和类似方法）。

### 使用mod_fcgid替代mod_fastcgi

通过FastCGI提供应用程序的另一种方法是使用Apache的[mod_fcgid](http://httpd.apache.org/mod_fcgid/)模块。与mod_fastcgi相比，mod_fcgid对FastCGI应用程序的处理方式不同，它管理工作进程本身的产生，不提供像`FastCGIExternalServer`这样的东西。这意味着配置看起来略有不同。

实际上，您必须像添加一个类似于稍后描述的在[_shared-hosting environment_](#apache-shared-hosting)中运行Django的脚本处理程序。有关详细信息，请参阅[mod_fcgid参考](http://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html)

## lighttpd设置

[lighttpd](http://www.lighttpd.net/)是一种通常用于提供静态文件的轻量级Web服务器。它本地支持FastCGI，因此，如果您的站点没有任何Apache特定的需求，它是服务静态页面和动态页面的不错选择。

请确保`mod_fastcgi`位于模块列表中`mod_rewrite`和`mod_access`之后的位置，而不是`mod_accesslog`之后。您可能也希望使用`mod_alias`来管理管理媒体。

将以下内容添加到lighttpd配置文件中：

```
server.document-root = "/home/user/public_html"
fastcgi.server = (
    "/mysite.fcgi" => (
        "main" => (
            # Use host / port instead of socket for TCP fastcgi
            # "host" => "127.0.0.1",
            # "port" => 3033,
            "socket" => "/home/user/mysite.sock",
            "check-local" => "disable",
        )
    ),
)
alias.url = (
    "/media" => "/home/user/django/contrib/admin/media/",
)

url.rewrite-once = (
    "^(/media.*)$" => "$1",
    "^/favicon\.ico$" => "/media/favicon.ico",
    "^(/.*)$" => "/mysite.fcgi$1",
)

```

### 在一个lighttpd上运行多个Django站点

lighttpd允许您使用“条件配置”允许每个主机自定义配置。要指定多个FastCGI站点，只需在每个站点的FastCGI配置周围添加一个条件块：

```
# If the hostname is 'www.example1.com'...
$HTTP["host"] == "www.example1.com" {
    server.document-root = "/foo/site1"
    fastcgi.server = (
       ...
    )
    ...
}

# If the hostname is 'www.example2.com'...
$HTTP["host"] == "www.example2.com" {
    server.document-root = "/foo/site2"
    fastcgi.server = (
       ...
    )
    ...
}

```

您还可以通过在`fastcgi.server`指令中指定多个条目，在同一网站上运行多个Django安装。为每个添加一个FastCGI主机。

## 切诺基设置

Cherokee是一个非常快速，灵活和易于配置的Web服务器。它支持现在广泛的技术：FastCGI，SCGI，PHP，CGI，SSI，TLS和SSL加密连接，虚拟主机，验证，即时编码，负载平衡，Apache兼容日志文件，数据库平衡器，反向HTTP代理和多更多。

Cherokee项目提供了一个文档，用于与Cherokee一起[设置Django](http://www.cherokee-project.com/doc/cookbook_django.html)。

## 在带有Apache的共享托管提供程序上运行Django

许多共享托管提供程序不允许您运行自己的服务器守护程序或编辑`httpd.conf`文件。在这些情况下，仍然可以使用Web服务器生成的进程运行Django。

注意

如果您正在使用Web服务器生成的进程，如本节所述，您无需自己启动FastCGI服务器。Apache将产生一些进程，按需扩展。

在Web根目录中，将其添加到名为`.htaccess`的文件：

```
AddHandler fastcgi-script .fcgi
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ mysite.fcgi/$1 [QSA,L]

```

然后，创建一个小脚本，告诉Apache如何生成您的FastCGI程序。创建一个文件`mysite.fcgi`并将其放在您的Web目录中，并确保使其可执行：

```
#!/usr/bin/python
import sys, os

# Add a custom Python path.
sys.path.insert(0, "/home/user/python")

# Switch to the directory of your project. (Optional.)
# os.chdir("/home/user/myproject")

# Set the DJANGO_SETTINGS_MODULE environment variable.
os.environ['DJANGO_SETTINGS_MODULE'] = "myproject.settings"

from django.core.servers.fastcgi import runfastcgi
runfastcgi(method="threaded", daemonize="false")

```

如果你的服务器使用mod_fastcgi这工作。另一方面，如果你正在使用mod_fcgid，除了在`.htaccess`文件中略有改动，设置基本相同。而不是添加一个fastcgi脚本处理程序，你必须添加一个fcgid-handler：

```
AddHandler fcgid-script .fcgi
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ mysite.fcgi/$1 [QSA,L]

```

### 重新启动生成的服务器

如果您更改网站上的任何Python代码，您需要告诉FastCGI代码已更改。但在这种情况下不需要重新启动Apache。相反，只需重新上传`mysite.fcgi`或编辑文件，以便文件上的时间戳将更改。当Apache看到文件已更新时，它会为您重新启动Django应用程序。

如果您可以访问Unix系统上的命令shell，可以使用`touch`命令轻松完成此操作：

```
touch mysite.fcgi

```

## 投放管理媒体文件

无论您最终决定使用的服务器和配置，您还需要考虑如何提供管理媒体文件。在[_mod_wsgi_](wsgi/modwsgi.html#serving-the-admin-files)文档中给出的建议也适用于上面详述的设置。

## 将URL前缀强制为特定值

因为许多这些基于fastcgi的解决方案需要在Web服务器的某个点重写URL，Django看到的路径信息可能不像传入的原始URL。这是一个问题，如果Django应用程序是从一个特定的前缀提供服务，你想要你的URL从[`{% url %}`](../../ref/templates/builtins.html#std:templatetag-url)标记看起来像前缀，而不是重写的版本，其可能包含例如`mysite.fcgi`。

Django做一个很好的尝试来弄出真正的脚本名称前缀应该是什么。特别是，如果Web服务器设置`SCRIPT_URL`（特定于Apache的mod_rewrite）或`REDIRECT_URL`（由一些服务器设置，包括Apache + mod_rewrite在某些情况下）将自动处理原始前缀。

如果Django无法正确计算出前缀，以及您希望在网址中使用原始值，则可以在主要的`settings`文件中设置[`FORCE_SCRIPT_NAME`](../../ref/settings.html#std:setting-FORCE_SCRIPT_NAME)设置。这将为通过该设置文件提供的每个网址统一设置脚本名称。因此，如果您希望不同的网址集在这种情况下具有不同的脚本名称，则需要使用不同的设置文件，但这是一种罕见的情况。

作为如何使用它的示例，如果您的Django配置为`'/'`下的所有URL提供服务，并且想要使用此设置，则可以设置`FORCE_SCRIPT_NAME = ''`。

{% endraw %}