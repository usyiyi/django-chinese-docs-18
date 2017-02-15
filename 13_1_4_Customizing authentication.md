

# 在Django中自定义身份验证

Django 自带的认证系统足够应付大多数情况，但你或许不打算使用现成的认证系统。定制自己的项目的权限系统需要了解哪些一些关键点，即Django中哪些部分是能够扩展或替换的。这个文档提供了如何定制权限系统的细节。

[_“认证”后端_](#authentication-backends) 在以下情形时可被扩展:当一个 User 模型对象带有用户名和密码时，且需要有别于 Django 默认的认证功能。

你可为你的模型提供可通过 Django 权限系统检查的 [_定制的权限_](#custom-permissions)。

你能够[_扩展_](#extending-user) 默认的 User 模型，或[_实现_](#auth-custom-user) 一个完全定制的模型。

## 其他认证源

有时候你需要挂接到其他认证资源 -- 另一包含用户名，密码的数据源或者其他认证方法。

例如，您的公司可能已经有一个LDAP设置存储了每一位员工的用户名和密码。 对于一个在LDAP和Django网站都拥有账号的用户来说，如果他/她不能使用LDAP账号登录Django网站，对他/她以及网站管理员来说都是一件麻烦事。

为了解决类似情形，django的认证系统允许你添加其他认证方式。您可以覆盖Django的基于数据库的默认方案，也可以连接使用其他系统的认证服务。

有关Django附带的身份验证后端的信息，请参阅[_authentication backend reference_](../../ref/contrib/auth.html#authentication-backends-reference)。

### 指定认证后端

在底层，Django 维护一个“认证后台”的列表。当调用[`django.contrib.auth.authenticate()`](default.html#django.contrib.auth.authenticate "django.contrib.auth.authenticate") 时 —— [_如何登入一个用户_](default.html#how-to-log-a-user-in) 中所描述的 —— Django 会尝试所有的认证后台进行认证。如果第一个认证方法失败，Django 将尝试第二个，以此类推，直至试完所有的认证后台。

使用的认证后台通过[`AUTHENTICATION_BACKENDS`](../../ref/settings.html#std:setting-AUTHENTICATION_BACKENDS) 设置指定。它应该是一个包含Python 路径名称的元组，它们指向的Python 类知道如何进行验证。这些类可以位于Python 路径上任何地方。

默认情况下，[`AUTHENTICATION_BACKENDS`](../../ref/settings.html#std:setting-AUTHENTICATION_BACKENDS) 设置为：

```
('django.contrib.auth.backends.ModelBackend',)

```

这个基本的认证后台会检查Django 的用户数据库并查询内建的权限。它不会通过任何的速率限制机制防护暴力破解。你可以在自定义的认证后端中实现自己的速率控制机制，或者使用大部分Web 服务器提供的机制。

[`AUTHENTICATION_BACKENDS`](../../ref/settings.html#std:setting-AUTHENTICATION_BACKENDS) 的顺序很重要，所以如果用户名和密码在多个后台中都是合法的，Django 将在第一个匹配成功后停止处理。

如果后台引发[`PermissionDenied`](../../ref/exceptions.html#django.core.exceptions.PermissionDenied "django.core.exceptions.PermissionDenied") 异常，认证将立即失败。Django 不会检查后面的认证后台。

注

一旦用户被认证过，Django会在用户的session中存储他使用的认证后端，然后在session有效期中一直会为该用户提供此后端认证。这种高效意味着验证源被缓存基于per-session基础, 所以如果你改变 [`AUTHENTICATION_BACKENDS`](../../ref/settings.html#std:setting-AUTHENTICATION_BACKENDS), 如果你需要迫使用户重新认证，需要清除掉 session 数据.一个简单的方式是使用这个方法： `Session.objects.all().delete()`.

### 编写一个认证后端

一个认证后端是个实现两个方法的类: `get_user(user_id)` and `authenticate(**credentials)`, as well as a set of optional permission related [_authorization methods_](#authorization-methods).

`get_user` 方法要求一个参数 `user_id` –这个参数可以是用户名，数据库中的ID或其它标识 `User` 对象的主键– 方法返回一个 `User` 对象.

`身份验证` 方法使用凭据作为关键字参数。大多数情况下，代码如下︰

```
class MyBackend(object):
    def authenticate(self, username=None, password=None):
        # Check the username/password and return a User.
        ...

```

当然，它也可以接收token的方式作为参数，例如:

```
class MyBackend(object):
    def authenticate(self, token=None):
        # Check the token and return a User.
        ...

```

不管怎样, `authenticate` 至少应该检查凭证, 如果凭证合法，它应该返回一个匹配于登录信息的 `User` 实例。如果不合法，则返回 `None`.

正如文档开头所描述的，Django的 admin 与Django `User`  对象是紧耦合的。到目前为止, 最好的解决方法是给每一个在你后台的用户创建一个 `User` 对象 (e.g., in your LDAP directory, your external SQL database, etc.) 你可以先写一个脚本来做这件事, 或者用你的 `authenticate` 方法在用户登陆的时候完成这件事。

这里有一个例子，后台对你定义在 `settings.py` 文件里的用户和密码进行验证，并且在用第一次验证的时候创建一个 `User` 对象:

```
from django.conf import settings
from django.contrib.auth.models import User, check_password

class SettingsBackend(object):
    """
 Authenticate against the settings ADMIN_LOGIN and ADMIN_PASSWORD.

 Use the login name, and a hash of the password. For example:

 ADMIN_LOGIN = 'admin'
 ADMIN_PASSWORD = 'sha1$4e987$afbcf42e21bd417fb71db8c66b321e9fc33051de'
 """

    def authenticate(self, username=None, password=None):
        login_valid = (settings.ADMIN_LOGIN == username)
        pwd_valid = check_password(password, settings.ADMIN_PASSWORD)
        if login_valid and pwd_valid:
            try:
                user = User.objects.get(username=username)
            except User.DoesNotExist:
                # Create a new user. Note that we can set password
                # to anything, because it won't be checked; the password
                # from settings.py will.
                user = User(username=username, password='get from settings.py')
                user.is_staff = True
                user.is_superuser = True
                user.save()
            return user
        return None

    def get_user(self, user_id):
        try:
            return User.objects.get(pk=user_id)
        except User.DoesNotExist:
            return None

```

### 在定制后端中处理授权

自定义验证后端能提供自己的权限。

当认证后端完成了这些功能 ([`get_group_permissions()`](../../ref/contrib/auth.html#django.contrib.auth.models.User.get_group_permissions "django.contrib.auth.models.User.get_group_permissions"), [`get_all_permissions()`](../../ref/contrib/auth.html#django.contrib.auth.models.User.get_all_permissions "django.contrib.auth.models.User.get_all_permissions"), [`has_perm()`](../../ref/contrib/auth.html#django.contrib.auth.models.User.has_perm "django.contrib.auth.models.User.has_perm"), and [`has_module_perms()`](../../ref/contrib/auth.html#django.contrib.auth.models.User.has_module_perms "django.contrib.auth.models.User.has_module_perms")) 那么user model就会给它授予相对应的许可。

提供给用户的权限将是所有后端返回的所有权限的超集也就是说，只要任意一个backend授予了一个user权限，django就给这个user这个权限。

New in Django 1.8:

如果后端在[`has_perm()`](../../ref/contrib/auth.html#django.contrib.auth.models.User.has_perm "django.contrib.auth.models.User.has_perm")或[`has_module_perms()`](../../ref/contrib/auth.html#django.contrib.auth.models.User.has_module_perms "django.contrib.auth.models.User.has_module_perms")中引发[`PermissionDenied`](../../ref/exceptions.html#django.core.exceptions.PermissionDenied "django.core.exceptions.PermissionDenied")异常，授权将立即失败，Django不会检查后端接下来。

上述的简单backend可以相当容易的完成授予admin权限。

```
class SettingsBackend(object):
    ...
    def has_perm(self, user_obj, perm, obj=None):
        if user_obj.username == settings.ADMIN_LOGIN:
            return True
        else:
            return False

```

在上例中，授予了用户所有访问权限。注意， 由于[`django.contrib.auth.models.User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User") 同名函数将接收同样的参数，认证后台接收到的 user_obj，有可能是匿名用户 anonymous

一个完整的认证过程，可以参考 `ModelBackend`类，它位于[django/contrib/auth/backends.py](https://github.com/django/django/blob/master/django/contrib/auth/backends.py)，ModelBackend是默认的认证后台，并且大多数情况下会对`auth_permission`表进行查询。  如果你想对后台API提供自定义行为，你可以利用Python继承的优势，继承`ModelBackend`并自定义后台API

#### 授权匿名用户

匿名用户是指不经过身份验证即他们有没有提供有效的身份验证细节。然而，这并不一定意味着他们不被授权做任何事情。在最基本的层面上，大多数网站授权匿名用户浏览大部分网站，许多网站允许匿名张贴评论等。

Django 的权限框架没有一个地方来存储匿名用户的权限。然而，传递给身份验证后端的用户对象可能是 [`django.contrib.auth.models.AnonymousUser`](../../ref/contrib/auth.html#django.contrib.auth.models.AnonymousUser "django.contrib.auth.models.AnonymousUser") 对象，该对象允许后端指定匿名用户自定义的授权行为。这对可重用应用的作者是很有用的, 因为他可以委托所有的请求, 例如控制匿名用户访问,给这个认证后端, 而不需要设置它

#### 非活动用户的授权

非活动用户是经过身份验证但属性`is_active`设置为`False`的用户。然而这并不意味着他们无权做任何事情。例如他们可以被允许激活他们的帐户。

对权限系统中的匿名用户的支持允许匿名用户具有执行某些操作的权限的情况，而未被认证的用户不具有。

不要忘记在自己的后端权限方法中测试用户的`is_active`属性。

#### 操作对象权限

django的权限框架对对象权限有基础的支持, 尽管在它的核心没有实现它.这意味着对象权限检查将始终返回 `False` 或空列表 （取决于检查的行为）。一个认证后端将传递关键字参数`obj` 和 `user_obj` 给每一个对象相关的认证方法, 并且能够返回适当的对象级别的权限.

## 自定义权限

要为给定模型对象创建自定义权限，请使用`权限` [_模型元属性_](../db/models.html#meta-options)。

此示例任务模型创建三个自定义权限，即用户是否可以对您的应用程序任务实例执行操作：

```
class Task(models.Model):
    ...
    class Meta:
        permissions = (
            ("view_task", "Can see available tasks"),
            ("change_task_status", "Can change the status of tasks"),
            ("close_task", "Can remove a task by setting its status as closed"),
        )

```

唯一需要做的就是在运行[`manage.py migrate`](../../ref/django-admin.html#django-admin-migrate)时创建这些额外的权限。当用户尝试访问应用程序提供的功能（查看任务，更改任务状态，关闭任务）时，您的代码负责检查这些权限的值。继续上面的示例，以下检查用户是否可以查看任务：

```
user.has_perm('app.view_task')

```

## 扩展已有的用户模型

有两种方法来扩展默认的[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")模型，而不用替换你自己的模型。 如果你需要的只是行为上的改变，而不需要对数据库中存储的内容做任何改变，你可以创建基于[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User") 的[_代理模型_](../db/models.html#proxy-models)。代理模型提供的功能包括默认的排序、自定义管理器以及自定义模型方法。

如果你想存储新字段到已有的`User`里，那么你可以选择[_one-to-one relationship_](../../ref/models/fields.html#ref-onetoone)来扩展用户信息。这种 one-to-one 模型一般被称为资料模型(profile model)，它通常被用来存储一些有关网站用户的非验证性（ non-auth ）资料。例如，你可以创建一个员工模型 (Employee model)：

```
from django.contrib.auth.models import User

class Employee(models.Model):
    user = models.OneToOneField(User)
    department = models.CharField(max_length=100)

```

假设一个员工Fred Smith 既有User 模型又有Employee 模型，你可以使用Django 标准的关联模型访问相关联的信息：

```
>>> u = User.objects.get(username='fsmith')
>>> freds_department = u.employee.department

```

要将个人资料模型的字段添加到管理后台的用户页面中，请在应用程序的`admin.py`定义一个[`InlineModelAdmin`](../../ref/contrib/admin/index.html#django.contrib.admin.InlineModelAdmin "django.contrib.admin.InlineModelAdmin")（对于本示例，我们将使用[`StackedInline`](../../ref/contrib/admin/index.html#django.contrib.admin.StackedInline "django.contrib.admin.StackedInline") )并将其添加到`UserAdmin`类并向[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")类注册的：

```
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from django.contrib.auth.models import User

from my_user_profile_app.models import Employee

# Define an inline admin descriptor for Employee model
# which acts a bit like a singleton
class EmployeeInline(admin.StackedInline):
    model = Employee
    can_delete = False
    verbose_name_plural = 'employee'

# Define a new User admin
class UserAdmin(UserAdmin):
    inlines = (EmployeeInline, )

# Re-register UserAdmin
admin.site.unregister(User)
admin.site.register(User, UserAdmin)

```

这些Profile models在任何方面都不特殊，它们就是和User model多了一个一对一链接的普通Django models。 这种情形下，它们不会在一名用户创建时自动创建, 但是 [`django.db.models.signals.post_save`](../../ref/signals.html#django.db.models.signals.post_save "django.db.models.signals.post_save") 可以在适当的时候用于创建或更新相关模型。

注意使用相关模型的成果需另外的查询或者联结来获取相关数据，基于你的需求替换用户模型并添加相关字段可能是你更好的选择。但是，在你项目应用程序中，指向默认用户模型的链接可能带来额外的数据库负载。

## 重写用户模型

Django 内建的[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User") 模型可能不适合某些类型的项目。例如，在某些网站上使用邮件地址而不是用户名作为身份的标识可能更合理。

Django 允许你通过[`AUTH_USER_MODEL`](../../ref/settings.html#std:setting-AUTH_USER_MODEL) 设置覆盖默认的User 模型， 其值引用一个自定义的模型。

```
AUTH_USER_MODEL = 'myapp.MyUser'

```

上面的值表示Django 应用的名称（必须位于[`INSTALLED_APPS`](../../ref/settings.html#std:setting-INSTALLED_APPS) 中） 和你想使用的User 模型的名称。

注意

改变 [`AUTH_USER_MODEL`](../../ref/settings.html#std:setting-AUTH_USER_MODEL) 对你的数据库结构有很大的影响。它改变了一些会使用到的表格，并且会影响到一些外键和多对多关系的构造。如果你打算设置[`AUTH_USER_MODEL`](../../ref/settings.html#std:setting-AUTH_USER_MODEL), 你应该在创建任何迁移或者第一次运行 `manage.py migrate` 前设置它。

在你有表格被创建后更改此设置是不被[`makemigrations`](../../ref/django-admin.html#django-admin-makemigrations)支持的，并且会导致你需要手动修改数据库结构，从旧用户表中导出数据，可能重新应用一些迁移。

警告

由于Django的可交换模型的动态依赖特性的局限, 你必须确保 [`AUTH_USER_MODEL`](../../ref/settings.html#std:setting-AUTH_USER_MODEL)引用的模型在所属app中第一个迁移文件中被创建(通常命名为 `0001_initial`); 否则, 你会碰到错误.

此外，在运行迁移时可能会遇到CircularDependencyError，因为Django由于动态依赖性而无法自动断开依赖性循环。如果您看到此错误，您应该通过将依赖于您的用户模型的模型移动到第二个迁移中来打破循环（您可以尝试制作两个具有ForeignKey的正常模型，并查看`makemigrations`

可重复使用的应用程式和`AUTH_USER_MODEL`

可重复使用的应用不应实施自定义用户模型。项目可能使用许多应用程序，并且实施自定义用户模型的两个可重用应用程序不能一起使用。如果您需要在应用中存储每个用户的信息，请使用[`ForeignKey`](../../ref/models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")或[`OneToOneField`](../../ref/models/fields.html#django.db.models.OneToOneField "django.db.models.OneToOneField")设置`settings.`AUTH_USER_MODEL，如下所述。

### 引用User 模型

在[`AUTH_USER_MODEL`](../../ref/settings.html#std:setting-AUTH_USER_MODEL) 设置改成其它用户模型的项目中，如果你直接引用[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")（例如，通过一个外键引用它），你的代码将不能工作。

`get_user_model`()[[source]](../../_modules/django/contrib/auth.html#get_user_model)

你应该使用`django.contrib.auth.get_user_model()` 来引用用户模型，而不要直接引用[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")。 这个方法将返回当前正在使用的用户模型 —— 指定的自定义用户模型或者[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")。

当你定义一个外键或者到用户模型的多对多关系时，你应该使用 [`AUTH_USER_MODEL`](../../ref/settings.html#std:setting-AUTH_USER_MODEL) 设置来指定自定义的模型。例如：

```
from django.conf import settings
from django.db import models

class Article(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL)

```

New in Django 1.7:

连接用户模型发出的信号时， 应该使用[`AUTH_USER_MODEL`](../../ref/settings.html#std:setting-AUTH_USER_MODEL) 设置指定自定义的模型。例如：

```
from django.conf import settings
from django.db.models.signals import post_save

def post_save_receiver(sender, instance, created, **kwargs):
    pass

post_save.connect(post_save_receiver, sender=settings.AUTH_USER_MODEL)

```

一般来说，在导入时候执行的代码中，你应该使用[`AUTH_USER_MODEL`](../../ref/settings.html#std:setting-AUTH_USER_MODEL) 设置引用用户模型。`get_user_model()` 只在Django 已经导入所有的模型后才工作。

### 指定自定义的用户模型

模型设计考虑

处理不直接相关的认证在自定义用户模型信息之前，应仔细考虑。

这可能是更好的存储应用程序特定的用户信息在与用户模式的关系的典范。这使得每一个应用，而不用担心与其他应用程序冲突指定自己的用户数据需求.另一方面，查询来检索此相关的信息将涉及的数据库连接，这可能对性能有影响。

Django 期望你自定义的  User model 满足一些最低要求

1.  模型必须有一个唯一的字段可被用于识别目的。可以是一个用户名，电子邮件地址，或任何其它独特属性。
2.  你的模型必须提供一种方法可以在"short"and"long"form可以定位到用户。最普遍的方法是用用户的名来作为简称,用用户的全名来作为全称。然而，对这两种方式没有特定的要求,如果你想，他们可以返回完全相同的值。

Changed in Django 1.8:

Django的旧版本要求你的模型有一个整数主键也是如此。

创建一个规范的自定义模型最简单的方法是继承[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")提供`User`模型的核心实现，包括散列密码和令牌化密码重置。然后，您必须提供一些关键的实施细节：

_class_ `models.``CustomUser`

`USERNAME_FIELD`

描述User模型上用作唯一标识符的字段名称的字符串。这通常是某种用户名，但它也可以是电子邮件地址或任何其他唯一标识符。字段_必须_必须是唯一的（即在其定义中设置`unique=True`）。

在以下示例中，字段`identifier`用作标识字段：

```
class MyUser(AbstractBaseUser):
    identifier = models.CharField(max_length=40, unique=True)
    ...
    USERNAME_FIELD = 'identifier'

```

New in Django 1.8.

[`USERNAME_FIELD`](#django.contrib.auth.models.CustomUser.USERNAME_FIELD "django.contrib.auth.models.CustomUser.USERNAME_FIELD")现在支持[`ForeignKey`](../../ref/models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")。由于在[`createsuperuser`](../../ref/django-admin.html#django-admin-createsuperuser)提示期间没有办法传递模型实例，因此希望用户在默认情况下输入[`to_field`](../../ref/models/fields.html#django.db.models.ForeignKey.to_field "django.db.models.ForeignKey.to_field")值（[`primary_key`](../../ref/models/fields.html#django.db.models.Field.primary_key "django.db.models.Field.primary_key") ）的现有实例。

`REQUIRED_FIELDS`

当通过[`createsuperuser`](../../ref/django-admin.html#django-admin-createsuperuser)管理命令创建一个用户时，用于提示的一个字段名称列表.。将会提示给列表里面的每一个字段提供一个值。 它包含的必须是 为 `False` 或者[`blank`](../../ref/models/fields.html#django.db.models.Field.blank "django.db.models.Field.blank")未定义的字段， 也可包含你想要在交互地创建一个新的用户时想要展示的其他字段。`REQUIRED_FIELDS`在Django的其他部分没有任何影响，例如在管理员中创建用户。

例如，以下是定义两个必需字段（出生日期和身高）的`User`模型的部分定义：

```
class MyUser(AbstractBaseUser):
    ...
    date_of_birth = models.DateField()
    height = models.FloatField()
    ...
    REQUIRED_FIELDS = ['date_of_birth', 'height']

```

注意

`REQUIRED_FIELDS`必须包含`User`模型中的所有必填字段，但应_不应_包含`USERNAME_FIELD`或`password`，因为将始终提示输入这些字段。

New in Django 1.8.

[`REQUIRED_FIELDS`](#django.contrib.auth.models.CustomUser.REQUIRED_FIELDS "django.contrib.auth.models.CustomUser.REQUIRED_FIELDS")现在支持[`ForeignKey`](../../ref/models/fields.html#django.db.models.ForeignKey "django.db.models.ForeignKey")。由于在[`createsuperuser`](../../ref/django-admin.html#django-admin-createsuperuser)提示期间没有办法传递模型实例，因此希望用户在默认情况下输入[`to_field`](../../ref/models/fields.html#django.db.models.ForeignKey.to_field "django.db.models.ForeignKey.to_field")值（[`primary_key`](../../ref/models/fields.html#django.db.models.Field.primary_key "django.db.models.Field.primary_key") ）的现有实例。

`is_active`

指示用户是否被视为“活动”的布尔属性。此属性作为`AbstractBaseUser`上的属性提供，默认为`True`。如何选择实施它将取决于您选择的身份验证后端的详细信息。请参阅[`is_active attribute on the built-in user model`](../../ref/contrib/auth.html#django.contrib.auth.models.User.is_active "django.contrib.auth.models.User.is_active")。

`get_full_name`()

用户更长且正式的标识.常见的解释会是用户的完整名称，但它可以是任何字符串，用于标识用户。

`get_short_name`()

一个短的且非正式用户的标识符。常见的解释会是第一个用户的名称，但它可以是任意字符串，用于以非正式的方式标识用户。它也可能会返回与[`django.contrib.auth.models.User.get_full_name()`](../../ref/contrib/auth.html#django.contrib.auth.models.User.get_full_name "django.contrib.auth.models.User.get_full_name")相同的值。

以下方法适用于[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")的任何子类：

_class_ `models.``AbstractBaseUser`

`get_username`()

返回由`USERNAME_FIELD`指定的字段的值。

`is_anonymous`()

始终返回`False`。这是区分[`AnonymousUser`](../../ref/contrib/auth.html#django.contrib.auth.models.AnonymousUser "django.contrib.auth.models.AnonymousUser")对象的一种方法。通常，您应该优先使用[`is_authenticated()`](#django.contrib.auth.models.AbstractBaseUser.is_authenticated "django.contrib.auth.models.AbstractBaseUser.is_authenticated")到此方法。

`is_authenticated`()

始终返回`True`。这是一种判断用户是否已通过身份验证的方法。这并不意味着任何权限，并且不检查用户是否处于活动状态 - 它仅指示用户已提供有效的用户名和密码。

`set_password`(_raw_password_)

将用户的密码设置为给定的原始字符串，注意密码哈希。不保存[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")对象。

当raw_password为`None`时，密码将被设置为不可用的密码，如同使用[`set_unusable_password()`](#django.contrib.auth.models.AbstractBaseUser.set_unusable_password "django.contrib.auth.models.AbstractBaseUser.set_unusable_password")。

`check_password`(_raw_password_)

如果给定的原始字符串是用户的正确密码，则返回`True`。（这将在进行比较时处理密码散列。）

`set_unusable_password`()

将用户标记为没有设置密码。这与为密码使用空白字符串不同。[`check_password()`](#django.contrib.auth.models.AbstractBaseUser.check_password "django.contrib.auth.models.AbstractBaseUser.check_password")此用户将永远不会返回`True`。不保存[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")对象。

如果针对现有外部源（例如LDAP目录）进行应用程序的身份验证，则可能需要这样做。

`has_usable_password`()

如果[`set_unusable_password()`](#django.contrib.auth.models.AbstractBaseUser.set_unusable_password "django.contrib.auth.models.AbstractBaseUser.set_unusable_password")已为此用户调用，则返回`False`。

`get_session_auth_hash`()

New in Django 1.7.

返回密码字段的HMAC。用于[_Session invalidation on password change_](default.html#session-invalidation-on-password-change)。

你应该再为你的`User`模型自定义一个管理器。 如果你的`User`模型定义了这些字段：`username`, `email`, `is_staff`, `is_active`, `is_superuser`, `last_login`, and `date_joined`跟默认的`User`的字段是一样的话, 那么你就使用Django的[`UserManager`](../../ref/contrib/auth.html#django.contrib.auth.models.UserManager "django.contrib.auth.models.UserManager")就行了;总之, 如果你的`User`定义了不同的字段, 你就要去自定义一个管理器，它继承自[`BaseUserManager`](#django.contrib.auth.models.BaseUserManager "django.contrib.auth.models.BaseUserManager")并提供两个额外的方法:

_class_ `models.``CustomUserManager`

`create_user`(_*username_field*_, _password=None_, _**other_fields_)

 `create_user()` 原本接受username，以及其它所有必填字段作为参数。例如，如果你的user模型使用 `email` 作为username字段, 并且使用 `date_of_birth` 作为必填字段, 那么`create_user` 应该定义为：

```
def create_user(self, email, date_of_birth, password=None):
    # create user here
    ...

```

`create_superuser`(_*username_field*_, _password_, _**other_fields_)

`create_superuser()`的原型应该接受用户名字段，以及所有必需的字段作为参数。例如，如果您的用户模型使用`email`作为用户名字段，并且`date_of_birth`为必填字段，则`create_superuser`应定义为：

```
def create_superuser(self, email, date_of_birth, password):
    # create superuser here
    ...

```

与`create_user()`不同，`create_superuser()` _必须_要求调用方提供密码。

[`BaseUserManager`](#django.contrib.auth.models.BaseUserManager "django.contrib.auth.models.BaseUserManager")提供以下实用程序方法：

_class_ `models.``BaseUserManager`

`normalize_email`(_email_)

一个通过将部分电子邮箱地址转为小写来使其规范化的类方法 `classmethod` 

`get_by_natural_key`(_username_)

使用由`USERNAME_FIELD`指定的字段内容检索用户实例。

`make_random_password`(_length=10_, _allowed_chars='abcdefghjkmnpqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXYZ23456789'_)

返回具有给定长度和给定字符串的允许字符的随机密码。请注意，默认值`allowed_chars`不包含可能导致用户混淆的字母，包括：

*   `i`，`l`，`I`和`1`（小写字母i，小写字母L，大写字母i，第一号）
*   `o`，`O`和`0`（小写字母o，大写字母o和零）

### Extending Django’s default User

如果你完全满意Django 的 [`用户`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User") 模型且你只想添加一些额外的配置文件信息，您可以简单的继承 `django.contrib.auth.models.AbstractUser` 并添加您的自定义字段，尽管我们建议像"Model design considerations"中描述的 [_Specifying a custom User model_](#specifying-custom-user-model)那样, 使用一个单独的模型。`AbstractUser`作为一种 [_抽象模型_](../db/models.html#abstract-base-classes) 提供默认 [`用户`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User") 的完整实现。

### 自定义用户与内置身份验证表单

正如您所期望的，内置Django的[_forms_](default.html#built-in-auth-forms)和[_views_](default.html#built-in-auth-views)对他们正在使用的用户模型做出某些假设。

如果您的用户模型不遵循相同的假设，可能需要定义替换表单，并作为auth视图配置的一部分传递该表单。

*   [`UserCreationForm`](default.html#django.contrib.auth.forms.UserCreationForm "django.contrib.auth.forms.UserCreationForm")

    取决于[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")模型。必须为任何自定义用户模型重写。

*   [`UserChangeForm`](default.html#django.contrib.auth.forms.UserChangeForm "django.contrib.auth.forms.UserChangeForm")

    取决于[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")模型。必须为任何自定义用户模型重写。

*   [`AuthenticationForm`](default.html#django.contrib.auth.forms.AuthenticationForm "django.contrib.auth.forms.AuthenticationForm")

    与[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")的任何子类一起使用，并将适应使用`USERNAME_FIELD`中定义的字段。

*   [`PasswordResetForm`](default.html#django.contrib.auth.forms.PasswordResetForm "django.contrib.auth.forms.PasswordResetForm")

    假设用户模型具有可用于标识用户的`email`字段和名为`is_active`的布尔字段，以防止对非活动用户进行密码重置。

*   [`SetPasswordForm`](default.html#django.contrib.auth.forms.SetPasswordForm "django.contrib.auth.forms.SetPasswordForm")

    适用于[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")的任何子类

*   [`PasswordChangeForm`](default.html#django.contrib.auth.forms.PasswordChangeForm "django.contrib.auth.forms.PasswordChangeForm")

    适用于[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")的任何子类

*   [`AdminPasswordChangeForm`](default.html#django.contrib.auth.forms.AdminPasswordChangeForm "django.contrib.auth.forms.AdminPasswordChangeForm")

    适用于[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")的任何子类

### 自定义用户和[`django.contrib.admin`](../../ref/contrib/admin/index.html#module-django.contrib.admin "django.contrib.admin: Django's admin site.")

如果你想让你自定义的User模型也可以在站点管理上工作，那么你的模型应该再定义一些额外的属性和方法。 这些方法允许管理员去控制User到管理内容的访问:

_class_ `models.``CustomUser`

`is_staff`

如果允许用户访问管理网站，则返回`True`。

`is_active`

如果用户帐户当前处于活动状态，则返回`True`。

`has_perm(perm, obj=None):`

如果用户具有命名权限，则返回`True`。如果提供`obj`，则需要针对特定​​对象实例检查权限。

`has_module_perms(app_label):`

如果用户有权访问给定应用中的模型，则返回`True`。

您还需要向管理员注册您的自定义用户模型。如果您的自定义User模型扩展`django.contrib.auth.models.AbstractUser`，则可以使用Django现有的`django.contrib.auth.admin.UserAdmin`类。但是，如果您的User模型扩展[`AbstractBaseUser`](#django.contrib.auth.models.AbstractBaseUser "django.contrib.auth.models.AbstractBaseUser")，则需要定义一个自定义的`ModelAdmin`类。可以将默认`django.contrib.auth.admin.UserAdmin`；但是，您需要覆盖引用`django.contrib.auth.models.AbstractUser`上不在您的自定义User类上的字段的任何定义。

### 自定义用户和权限

为了方便将Django的权限框架包含到你自己的User类中，Django提供了[`PermissionsMixin`](#django.contrib.auth.models.PermissionsMixin "django.contrib.auth.models.PermissionsMixin")。这是一个抽象模型，您可以包含在用户模型的类层次结构中，为您提供支持Django权限模型所需的所有方法和数据库字段。

[`PermissionsMixin`](#django.contrib.auth.models.PermissionsMixin "django.contrib.auth.models.PermissionsMixin")提供了以下方法和属性：

_class_ `models.``PermissionsMixin`

`is_superuser`

布尔值。指定此用户具有所有权限，而不显式分配它们。

`get_group_permissions`(_obj=None_)

通过用户的组返回用户拥有的一组权限字符串。

如果传入`obj`，则仅返回此特定对象的组权限。

`get_all_permissions`(_obj=None_)

通过组和用户权限返回用户拥有的一组权限字符串。

如果传入`obj`，则仅返回此特定对象的权限。

`has_perm`(_perm_, _obj=None_)

如果用户具有指定的权限，则返回`True`，其中`perm`的格式为`“＆lt； app label＆ ＆lt； permission codename＆gt；“`（请参阅[_permissions_](default.html#topic-authorization)）。如果用户处于非活动状态，此方法将始终返回`False`。

如果传入`obj`，此方法将不会检查模型的权限，而是检查此特定对象。

`has_perms`(_perm_list_, _obj=None_)

如果用户具有每个指定的权限，则返回`True`，其中每个perm的格式为`“＆lt； app label＆gt；。＆lt； permission codename＆gt；“`。如果用户处于非活动状态，此方法将始终返回`False`。

如果传入`obj`，此方法将不会检查模型的权限，而是检查特定对象。

`has_module_perms`(_package_name_)

如果用户在给定的包（Django应用标签）中有任何权限，则返回`True`。如果用户处于非活动状态，此方法将始终返回`False`。

ModelBackend

如果您不包含[`PermissionsMixin`](#django.contrib.auth.models.PermissionsMixin "django.contrib.auth.models.PermissionsMixin")，则必须确保不要调用`ModelBackend`上的权限方法。`ModelBackend`假定某些字段在您的用户模型上可用。如果您的用户模型未提供这些字段，则在检查权限时将收到数据库错误。

### 自定义用户和代理模型

自定义用户模型的一个限制是，安装自定义用户模型将破坏扩展[`User`](../../ref/contrib/auth.html#django.contrib.auth.models.User "django.contrib.auth.models.User")的任何代理模型。代理模型必须基于具体的基类；通过定义自定义User模型，您可以删除Django可靠地识别基类的能力。

如果项目使用代理模型，则必须修改代理以扩展项目中当前使用的用户模型，或将代理的行为合并到用户子类中。

### 定制用户和测试/夹具

如果您正在编写与用户模型交互的应用程序，则必须采取一些预防措施，以确保测试套件将运行，而不管项目正在使用的用户模型。如果用户模型已换出，任何实例化User实例的测试都将失败。这包括使用夹具创建User实例的任何尝试。

为确保您的测试套件能够在任何项目配置中传递，`django.contrib.auth.tests.utils`定义了一个`@skipIfCustomUser`装饰器。如果正在使用除默认Django用户以外的任何用户模型，此装饰器将导致跳过测试用例。这个装饰器可以应用于单个测试或整个测试类。

根据您的应用程序，还可能需要添加测试，以确保应用程序与_任何_用户模型配合使用，而不仅仅是默认的用户模型。为了帮助这个，Django提供了两个可以在测试套件中使用的替代用户模型：

_class_ `tests.custom_user.``CustomUser`

使用`email`字段作为用户名的自定义用户模型，并且具有基本的管理员兼容的权限设置

_class_ `tests.custom_user.``ExtensionUser`

扩展`django.contrib.auth.models.AbstractUser`的自定义用户模型，添加了`date_of_birth`字段。

然后，您可以使用`@override_settings`装饰器使该测试与自定义User模型一起运行。例如，这里是一个测试的骨架，它将测试三个可能的用户模型 - 默认值，加上`auth` app提供的两个用户模型：

```
from django.contrib.auth.tests.utils import skipIfCustomUser
from django.contrib.auth.tests.custom_user import CustomUser, ExtensionUser
from django.test import TestCase, override_settings

class ApplicationTestCase(TestCase):
    @skipIfCustomUser
    def test_normal_user(self):
        "Run tests for the normal user model"
        self.assertSomething()

    @override_settings(AUTH_USER_MODEL='auth.CustomUser')
    def test_custom_user(self):
        "Run tests for a custom user model with email-based authentication"
        self.assertSomething()

    @override_settings(AUTH_USER_MODEL='auth.ExtensionUser')
    def test_extension_user(self):
        "Run tests for a simple extension of the built-in User."
        self.assertSomething()

```

### 一个完整例子

这是一个管理器允许的自定义user这个用户模型使用邮箱地址作为用户名，并且要求填写出生年月。它不提供任何权限检查，超出了用户帐户上的一个简单的`admin`标志。此模型将与所有内置的身份验证表单和视图兼容，但用户创建表单除外。此示例说明大多数组件如何协同工作，但不打算直接复制到项目以供生产使用。

此代码将全部位于自定义身份验证应用程序的`models.py`文件中：

```
from django.db import models
from django.contrib.auth.models import (
    BaseUserManager, AbstractBaseUser
)

class MyUserManager(BaseUserManager):
    def create_user(self, email, date_of_birth, password=None):
        """
 Creates and saves a User with the given email, date of
 birth and password.
 """
        if not email:
            raise ValueError('Users must have an email address')

        user = self.model(
            email=self.normalize_email(email),
            date_of_birth=date_of_birth,
        )

        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, date_of_birth, password):
        """
 Creates and saves a superuser with the given email, date of
 birth and password.
 """
        user = self.create_user(email,
            password=password,
            date_of_birth=date_of_birth
        )
        user.is_admin = True
        user.save(using=self._db)
        return user

class MyUser(AbstractBaseUser):
    email = models.EmailField(
        verbose_name='email address',
        max_length=255,
        unique=True,
    )
    date_of_birth = models.DateField()
    is_active = models.BooleanField(default=True)
    is_admin = models.BooleanField(default=False)

    objects = MyUserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['date_of_birth']

    def get_full_name(self):
        # The user is identified by their email address
        return self.email

    def get_short_name(self):
        # The user is identified by their email address
        return self.email

    def __str__(self):              # __unicode__ on Python 2
        return self.email

    def has_perm(self, perm, obj=None):
        "Does the user have a specific permission?"
        # Simplest possible answer: Yes, always
        return True

    def has_module_perms(self, app_label):
        "Does the user have permissions to view the app `app_label`?"
        # Simplest possible answer: Yes, always
        return True

    @property
    def is_staff(self):
        "Is the user a member of staff?"
        # Simplest possible answer: All admins are staff
        return self.is_admin

```

然后，要使用Django的管理员注册此自定义User模型，应用程序的`admin.py`文件中需要以下代码：

```
from django import forms
from django.contrib import admin
from django.contrib.auth.models import Group
from django.contrib.auth.admin import UserAdmin
from django.contrib.auth.forms import ReadOnlyPasswordHashField

from customauth.models import MyUser

class UserCreationForm(forms.ModelForm):
    """A form for creating new users. Includes all the required
 fields, plus a repeated password."""
    password1 = forms.CharField(label='Password', widget=forms.PasswordInput)
    password2 = forms.CharField(label='Password confirmation', widget=forms.PasswordInput)

    class Meta:
        model = MyUser
        fields = ('email', 'date_of_birth')

    def clean_password2(self):
        # Check that the two password entries match
        password1 = self.cleaned_data.get("password1")
        password2 = self.cleaned_data.get("password2")
        if password1 and password2 and password1 != password2:
            raise forms.ValidationError("Passwords don't match")
        return password2

    def save(self, commit=True):
        # Save the provided password in hashed format
        user = super(UserCreationForm, self).save(commit=False)
        user.set_password(self.cleaned_data["password1"])
        if commit:
            user.save()
        return user

class UserChangeForm(forms.ModelForm):
    """A form for updating users. Includes all the fields on
 the user, but replaces the password field with admin's
 password hash display field.
 """
    password = ReadOnlyPasswordHashField()

    class Meta:
        model = MyUser
        fields = ('email', 'password', 'date_of_birth', 'is_active', 'is_admin')

    def clean_password(self):
        # Regardless of what the user provides, return the initial value.
        # This is done here, rather than on the field, because the
        # field does not have access to the initial value
        return self.initial["password"]

class MyUserAdmin(UserAdmin):
    # The forms to add and change user instances
    form = UserChangeForm
    add_form = UserCreationForm

    # The fields to be used in displaying the User model.
    # These override the definitions on the base UserAdmin
    # that reference specific fields on auth.User.
    list_display = ('email', 'date_of_birth', 'is_admin')
    list_filter = ('is_admin',)
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        ('Personal info', {'fields': ('date_of_birth',)}),
        ('Permissions', {'fields': ('is_admin',)}),
    )
    # add_fieldsets is not a standard ModelAdmin attribute. UserAdmin
    # overrides get_fieldsets to use this attribute when creating a user.
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'date_of_birth', 'password1', 'password2')}
        ),
    )
    search_fields = ('email',)
    ordering = ('email',)
    filter_horizontal = ()

# Now register the new UserAdmin...
admin.site.register(MyUser, MyUserAdmin)
# ... and, since we're not using Django's built-in permissions,
# unregister the Group model from admin.
admin.site.unregister(Group)

```

最后，使用`settings.py`中的[`AUTH_USER_MODEL`](../../ref/settings.html#std:setting-AUTH_USER_MODEL)设置将自定义模型指定为项目的默认用户模型：

```
AUTH_USER_MODEL = 'customauth.MyUser'

```



# 匿名用户

_class_ `models.``AnonymousUser`

[`django.contrib.auth.models.AnonymousUser`](#django.contrib.auth.models.AnonymousUser "django.contrib.auth.models.AnonymousUser") 类实现了[`django.contrib.auth.models.User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 接口，但具有下面几个不同点：

*   [_id_](../../topics/db/models.html#automatic-primary-key-fields) 永远为`None`。
*   [`username`](#django.contrib.auth.models.User.username "django.contrib.auth.models.User.username") 永远为空字符串。
*   [`get_username()`](#django.contrib.auth.models.User.get_username "django.contrib.auth.models.User.get_username") 永远返回空字符串。
*   [`is_staff`](#django.contrib.auth.models.User.is_staff "django.contrib.auth.models.User.is_staff") 和[`is_superuser`](#django.contrib.auth.models.User.is_superuser "django.contrib.auth.models.User.is_superuser") 永远为`False`。
*   [`is_active`](#django.contrib.auth.models.User.is_active "django.contrib.auth.models.User.is_active") 永远为 `False`。
*   [`groups`](#django.contrib.auth.models.User.groups "django.contrib.auth.models.User.groups") 和[`user_permissions`](#django.contrib.auth.models.User.user_permissions "django.contrib.auth.models.User.user_permissions") 永远为空。
*   [`is_anonymous()`](#django.contrib.auth.models.User.is_anonymous "django.contrib.auth.models.User.is_anonymous") 返回`True` 而不是`False`。
*   [`is_authenticated()`](#django.contrib.auth.models.User.is_authenticated "django.contrib.auth.models.User.is_authenticated") 返回`False` 而不是`True`。
*   [`set_password()`](#django.contrib.auth.models.User.set_password "django.contrib.auth.models.User.set_password")、[`check_password()`](#django.contrib.auth.models.User.check_password "django.contrib.auth.models.User.check_password")、[`save()`](../models/instances.html#django.db.models.Model.save "django.db.models.Model.save") 和[`delete()`](../models/instances.html#django.db.models.Model.delete "django.db.models.Model.delete") 引发[`NotImplementedError`](https://docs.python.org/3/library/exceptions.html#NotImplementedError "(in Python v3.4)")。

New in Django 1.8:

新增`AnonymousUser.get_username()` 以更好地模拟 [`django.contrib.auth.models.User`](#django.contrib.auth.models.User "django.contrib.auth.models.User")。

在实际应用中，你自己可能不需要使用[`AnonymousUser`](#django.contrib.auth.models.AnonymousUser "django.contrib.auth.models.AnonymousUser") 对象，它们用于Web 请求，在下节会讲述。

