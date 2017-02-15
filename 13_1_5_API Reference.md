

# django.contrib.auth

这份文档提供Django 认证系统组件的API 参考资料。对于这些组件的用法以及如何自定义认证和授权请参照[_认证主题的相关指南_](../../topics/auth/index.html)。



# 用户

## 字段

_class_ `models.``User`

[`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 对象具有如下字段：

`username`

必选。少于等于30个字符。 用户名可以包含字母、数字、`_`、`@`、`+`、`.`和`-` 字符。

`first_name`

可选。 少于等于30个字符。

`last_name`

可选。少于30个字符。

`email`

可选。邮箱地址。

`password`

必选。 密码的哈希及元数据。（Django 不保存原始密码）。原始密码可以无限长而且可以包含任意字符。参见[_密码相关的文档_](../../topics/auth/passwords.html)。

`groups`

与[`Group`](#django.contrib.auth.models.Group "django.contrib.auth.models.Group") 之间的多对多关系。

`user_permissions`

与[`Permission`](#django.contrib.auth.models.Permission "django.contrib.auth.models.Permission") 之间的多对多关系。

`is_staff`

布尔值。指示用户是否可以访问Admin 站点。

`is_active`

布尔值。指示用户的账号是否激活。我们建议把这个标记设置为`False` 来代替删除账号；这样的话，如果你的应用和User 之间有外键关联，外键就不会失效。

它不是用来控制用户是否能够登录。认证的后端没有要求检查`is_active` 标记，而且默认的后端不会检查。如果你想在`is_active` 为`False` 时拒绝用户登录，你需要在你自己的视图或自定义的认证后端中作检查。但是，默认的[`login()`](../../topics/auth/default.html#django.contrib.auth.views.login "django.contrib.auth.views.login") 视图使用的[`AuthenticationForm`](../../topics/auth/default.html#django.contrib.auth.forms.AuthenticationForm "django.contrib.auth.forms.AuthenticationForm") 却_会_ 作这个检查，正如在Django 的Admin 站点中所做的权限检查方法如[`has_perm()`](#django.contrib.auth.models.User.has_perm "django.contrib.auth.models.User.has_perm") 和认证一样。对于未激活的用户，所有这些函数/方法都返回`False`。

`is_superuser`

布尔值。指定这个用户拥有所有的权限而不需要给他们分配明确的权限。

`last_login`

用户最后一次登录的时间。

Changed in Django 1.8:

如果这个用户没有登录过，这个字段将会是`null`。以前默认设置成当前的date/time。

`date_joined`

账户创建的时间。当账号创建时，默认设置为当前的date/time。

## 方法

_class_ `models.``User`

`get_username`()

返回这个User 的username。因为User 模型可以置换，你应该使用这个方法而不要直接访问username 属性。

`is_anonymous`()

永远返回`False`。这是区别[`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 和[`AnonymousUser`](#django.contrib.auth.models.AnonymousUser "django.contrib.auth.models.AnonymousUser") 对象的一种方法。一般情况下，相比这个方法更建议你使用[`is_authenticated()`](#django.contrib.auth.models.User.is_authenticated "django.contrib.auth.models.User.is_authenticated")。

`is_authenticated`()

永远返回`True`（与`AnonymousUser.is_authenticated()` 永远返回`False` 相反）。这是区分用户是否已经认证的一种方法。它不检查权限、用户是否激活以及是否具有一个合法的会话。即使通常你将在`request.user`上面 调用这个方法来确认用户是否已经被[`AuthenticationMiddleware`](../middleware.html#django.contrib.auth.middleware.AuthenticationMiddleware "django.contrib.auth.middleware.AuthenticationMiddleware") 填充（表示当前登录的用户），你应该明白这个方法对于任何[`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 实例都返回`True`。

`get_full_name`()

返回[`first_name`](#django.contrib.auth.models.User.first_name "django.contrib.auth.models.User.first_name") 和[`last_name`](#django.contrib.auth.models.User.last_name "django.contrib.auth.models.User.last_name")，之间带有一个空格。

`get_short_name`()

返回[`first_name`](#django.contrib.auth.models.User.first_name "django.contrib.auth.models.User.first_name")。

`set_password`(_raw_password_)

设置用户的密码为给定的原始字符串，并负责密码的哈希。不会保存[`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 对象。

当`raw_password` 为`None` 时，密码将设置为一个不可用的密码，和使用[`set_unusable_password()`](#django.contrib.auth.models.User.set_unusable_password "django.contrib.auth.models.User.set_unusable_password") 的效果一样。

`check_password`(_raw_password_)

如果给出的原始字符串是用户正确的密码，则返回`True`。 （它负责在比较时密码的哈希）。

`set_unusable_password`()

标记用户为没有设置密码。它与密码为空的字符串不一样。[`check_password()`](#django.contrib.auth.models.User.check_password "django.contrib.auth.models.User.check_password") 对这种用户永远不会返回`True`。不会保存[`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 对象。

如果你的认证发生在外部例如LDAP 目录时，可能需要这个函数。

`has_usable_password`()

如果对这个用户调用过[`set_unusable_password()`](#django.contrib.auth.models.User.set_unusable_password "django.contrib.auth.models.User.set_unusable_password")，则返回`False`。

`get_group_permissions`(_obj=None_)

返回一个用户当前拥有的权限的set，通过用户组

如果传入`obj`，则仅返回此特定对象的组权限。http://python.usyiyi.cn/translate/django_182/ref/contrib/auth.html#

`get_all_permissions`(_obj=None_)

通过组和用户权限返回用户拥有的一组权限字符串。

如果传入`obj`，则仅返回此特定对象的权限。

`has_perm`(_perm_, _obj=None_)

如果用户具有指定的权限，则返回`True`，其中perm的格式为`“＆lt； app label＆gt；。＆lt； permission codename＆gt；“`。（请参阅有关[_permissions_](../../topics/auth/default.html#topic-authorization)）。如果用户处于非活动状态，此方法将始终返回`False`。

如果传入`obj`，此方法将不会检查模型的权限，而是检查此特定对象。

`has_perms`(_perm_list_, _obj=None_)

如果用户具有每个指定的权限，则返回`True`，其中每个perm的格式为`“＆lt； app label＆gt；。＆lt； permission codename＆gt；“`。如果用户处于非活动状态，此方法将始终返回`False`。

如果传入`obj`，此方法将不会检查模型的权限，而是检查特定对象。

`has_module_perms`(_package_name_)

如果用户具有给出的package（Django 的应用标签）中的权限，则返回`True`。如果用户没有激活，这个方法将永远返回 `False`。

`email_user`(_subject_, _message_, _from_email=None_, _**kwargs_)

发生邮件给这个用户。如果`from_email` 为`None`，Django 将使用[`DEFAULT_FROM_EMAIL`](../settings.html#std:setting-DEFAULT_FROM_EMAIL)。

Changed in Django 1.7:

任何`**kwargs` 都将传递给底层的[`send_mail()`](../../topics/email.html#django.core.mail.send_mail "django.core.mail.send_mail") 调用。

## 管理器方法

_class_ `models.``UserManager`

[`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 模型有一个自定义的管理器，它具有以下辅助方法（除了[`BaseUserManager`](../../topics/auth/customizing.html#django.contrib.auth.models.BaseUserManager "django.contrib.auth.models.BaseUserManager") 提供的方法之外）：

`create_user`(_username_, _email=None_, _password=None_, _**extra_fields_)

创建、保存并返回一个[`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User")。

[`username`](#django.contrib.auth.models.User.username "django.contrib.auth.models.User.username") 和[`password`](#django.contrib.auth.models.User.password "django.contrib.auth.models.User.password") 设置为给出的值。[`email`](#django.contrib.auth.models.User.email "django.contrib.auth.models.User.email") 的域名部分将自动转换成小写，返回的[`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 对象将设置[`is_active`](#django.contrib.auth.models.User.is_active "django.contrib.auth.models.User.is_active") 为`True`。

如果没有提供password，将调用 [`set_unusable_password()`](#django.contrib.auth.models.User.set_unusable_password "django.contrib.auth.models.User.set_unusable_password")。

`extra_fields` 关键字参数将传递给[`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 的`__init__` 方法，以允许设置[_自定义User 模型_](../../topics/auth/customizing.html#auth-custom-user) 的字段。

参见[_创建用户_](../../topics/auth/default.html#topics-auth-creating-users) 中的示例用法。

`create_superuser`(_username_, _email_, _password_, _**extra_fields_)

与[`create_user()`](#django.contrib.auth.models.UserManager.create_user "django.contrib.auth.models.UserManager.create_user") 相同，但是设置[`is_staff`](#django.contrib.auth.models.User.is_staff "django.contrib.auth.models.User.is_staff") 和[`is_superuser`](#django.contrib.auth.models.User.is_superuser "django.contrib.auth.models.User.is_superuser") 为`True`。



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




# 权限

_class_ `models.``Permission`

## 字段

[`Permission`](#django.contrib.auth.models.Permission "django.contrib.auth.models.Permission") 对象有以下字段:

_class_ `models.``Permission`

`name`

必填项. 255个字符或者更少. 例如: `'Can vote'`.

Changed in Django 1.8:

`max_length` 属性从50个字符增加至255个字符

`content_type`

必填项.对`django_content_type`数据库表的引用，其中包含每个已安装模型的记录。

`codename`

必须项.小于等于是100个字符.例如: `'can_vote'`.

## 方法

[`Permission`](#django.contrib.auth.models.Permission "django.contrib.auth.models.Permission")对象具有类似任何其他[_Django model_](../models/instances.html)的标准数据访问方法。




# 用户群组 Group

_class_ `models.``Group`

## 字段

[`Group`](#django.contrib.auth.models.Group "django.contrib.auth.models.Group") 对象有以下字段:

_class_ `models.``Group`

`name`

必填项，80个字符以内。允许任何字符. 例如: `'Awesome Users'`.

`permissions`

多对多字段到[`Permission`](#django.contrib.auth.models.Permission "django.contrib.auth.models.Permission")：

```
group.permissions = [permission_list]
group.permissions.add(permission, permission, ...)
group.permissions.remove(permission, permission, ...)
group.permissions.clear()

```




# 登陆和注销标识

auth框架使用以下[_signals_](../../topics/signals.html)，可用于在用户登录或注销时通知。

`user_logged_in`()

当用户成功登录时发送。

与此信号一起发送的参数：

`sender`

The class of the user that just logged in.

`request`

The current [`HttpRequest`](../request-response.html#django.http.HttpRequest "django.http.HttpRequest") instance.

`user`

The user instance that just logged in.

`user_logged_out`()

在调用logout方法时发送。

`sender`

As above: the class of the user that just logged out or `None` if the user was not authenticated.

`request`

The current [`HttpRequest`](../request-response.html#django.http.HttpRequest "django.http.HttpRequest") instance.

`user`

The user instance that just logged out or `None` if the user was not authenticated.

`user_login_failed`()

当用户登录失败时发送

`sender`

The name of the module used for authentication.

`credentials`

A dictionary of keyword arguments containing the user credentials that were passed to [`authenticate()`](../../topics/auth/default.html#django.contrib.auth.authenticate "django.contrib.auth.authenticate") or your own custom authentication backend. Credentials matching a set of ‘sensitive’ patterns, (including password) will not be sent in the clear as part of the signal.



# 认证使用的后台

这一节详细讲述Django 自带的认证后台。关于如何使用它们以及如何编写你自己的认证后台，参见[_用户认证指南_](../../topics/auth/index.html) 中的[_其它认证源一节_](../../topics/auth/customizing.html#authentication-backends)。

## 可用的认证后台

以下是[`django.contrib.auth.backends`](#module-django.contrib.auth.backends "django.contrib.auth.backends: Django's built-in authentication backend classes.") 中可以使用的后台：

_class_ `ModelBackend`[[source]](../../_modules/django/contrib/auth/backends.html#ModelBackend)

这是Django使用的默认认证后台。它使用由用户标识和密码组成的凭据进行认证。对于Django的默认用户模型，用户的标识是用户名，对于自定义的用户模型，它通过USERNAME_FIELD 字段表示（参见[_自定义Users 和认证_](../../topics/auth/customizing.html)）。

它还处理 [`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 和[`PermissionsMixin`](../../topics/auth/customizing.html#django.contrib.auth.models.PermissionsMixin "django.contrib.auth.models.PermissionsMixin") 定义的权限模型。

[`has_perm()`](#django.contrib.auth.backends.ModelBackend.has_perm "django.contrib.auth.backends.ModelBackend.has_perm"), [`get_all_permissions()`](#django.contrib.auth.backends.ModelBackend.get_all_permissions "django.contrib.auth.backends.ModelBackend.get_all_permissions"), [`get_user_permissions()`](#django.contrib.auth.backends.ModelBackend.get_user_permissions "django.contrib.auth.backends.ModelBackend.get_user_permissions"), 和[`get_group_permissions()`](#django.contrib.auth.backends.ModelBackend.get_group_permissions "django.contrib.auth.backends.ModelBackend.get_group_permissions") 允许一个对象作为特定权限参数来传递, 如果条件是 if `obj is not None`. 后端除了返回一个空的permissions 外，并不会去完成他们。

`authenticate`(_username=None_, _password=None_, _**kwargs_)[[source]](../../_modules/django/contrib/auth/backends.html#ModelBackend.authenticate)

通过调用[`User.check_password`](#django.contrib.auth.models.User.check_password "django.contrib.auth.models.User.check_password") 验证`username` 和`password`。如果`username` 没有提供，它会使用[`CustomUser.USERNAME_FIELD`](../../topics/auth/customizing.html#django.contrib.auth.models.CustomUser.USERNAME_FIELD "django.contrib.auth.models.CustomUser.USERNAME_FIELD") 关键字从`kwargs` 中获取username。返回一个认证过的User 或`None`。

`get_user_permissions`(_user_obj_, _obj=None_)[[source]](../../_modules/django/contrib/auth/backends.html#ModelBackend.get_user_permissions)

New in Django 1.8.

返回`user_obj`具有的自己用户权限的权限字符串集合。如果[`is_anonymous()`](../../topics/auth/customizing.html#django.contrib.auth.models.AbstractBaseUser.is_anonymous "django.contrib.auth.models.AbstractBaseUser.is_anonymous")或[`is_active`](../../topics/auth/customizing.html#django.contrib.auth.models.CustomUser.is_active "django.contrib.auth.models.CustomUser.is_active")为`False`，则返回空集。

`get_group_permissions`(_user_obj_, _obj=None_)[[source]](../../_modules/django/contrib/auth/backends.html#ModelBackend.get_group_permissions)

返回`user_obj`从其所属组的权限中获取的权限字符集。如果[`is_anonymous()`](../../topics/auth/customizing.html#django.contrib.auth.models.AbstractBaseUser.is_anonymous "django.contrib.auth.models.AbstractBaseUser.is_anonymous")或[`is_active`](../../topics/auth/customizing.html#django.contrib.auth.models.CustomUser.is_active "django.contrib.auth.models.CustomUser.is_active")为`False`，则返回空集。

`get_all_permissions`(_user_obj_, _obj=None_)[[source]](../../_modules/django/contrib/auth/backends.html#ModelBackend.get_all_permissions)

返回`user_obj`的权限字符串集，包括用户权限和组权限。如果[`is_anonymous()`](../../topics/auth/customizing.html#django.contrib.auth.models.AbstractBaseUser.is_anonymous "django.contrib.auth.models.AbstractBaseUser.is_anonymous")或[`is_active`](../../topics/auth/customizing.html#django.contrib.auth.models.CustomUser.is_active "django.contrib.auth.models.CustomUser.is_active")为`False`，则返回空集。

`has_perm`(_user_obj_, _perm_, _obj=None_)[[source]](../../_modules/django/contrib/auth/backends.html#ModelBackend.has_perm)

使用[`get_all_permissions()`](#django.contrib.auth.backends.ModelBackend.get_all_permissions "django.contrib.auth.backends.ModelBackend.get_all_permissions")检查`user_obj`是否具有权限字符串`perm`。如果用户不是[`is_active`](../../topics/auth/customizing.html#django.contrib.auth.models.CustomUser.is_active "django.contrib.auth.models.CustomUser.is_active")，则返回`False`。

`has_module_perms`(_self_, _user_obj_, _app_label_)[[source]](../../_modules/django/contrib/auth/backends.html#ModelBackend.has_module_perms)

返回`user_obj`是否对应用`app_label`有任何权限。

_class_ `RemoteUserBackend`[[source]](../../_modules/django/contrib/auth/backends.html#RemoteUserBackend)

使用这个后台来处理Django的外部认证。. 它使用 [`request.`](../request-response.html#django.http.HttpRequest.META "django.http.HttpRequest.META")里面的usernames来进行验证。META ['REMOTE_USER']。请参阅[_Authenticating against REMOTE_USER_](../../howto/auth-remote-user.html)文档。

如果你需要更多的控制，你可以创建你自己的验证后端，继承这个类，并重写这些属性或方法：

`RemoteUserBackend.``create_unknown_user`

`True`或`False`。决定是否有一个 [`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") 对象已经在数据库中创建。默认为`True`。

`RemoteUserBackend.``authenticate`(_remote_user_)[[source]](../../_modules/django/contrib/auth/backends.html#RemoteUserBackend.authenticate)

作为`remote_user`传递的用户名被认为是可信的。此方法仅返回给定用户名的`User`对象，如果[`create_unknown_user`](#django.contrib.auth.backends.RemoteUserBackend.create_unknown_user "django.contrib.auth.backends.RemoteUserBackend.create_unknown_user")为`True`，则创建新的`User`对象。

如果[`create_unknown_user`](#django.contrib.auth.backends.RemoteUserBackend.create_unknown_user "django.contrib.auth.backends.RemoteUserBackend.create_unknown_user")是`False`，并且在数据库中找不到具有给定用户名的`User`对象，则返回`None`。

`RemoteUserBackend.``clean_username`(_username_)[[source]](../../_modules/django/contrib/auth/backends.html#RemoteUserBackend.clean_username)

Performs any cleaning on the `username` (e.g. stripping LDAP DN information) prior to using it to get or create a [`User`](#django.contrib.auth.models.User "django.contrib.auth.models.User") object. 返回已清除的用户名。

`RemoteUserBackend.``configure_user`(_user_)[[source]](../../_modules/django/contrib/auth/backends.html#RemoteUserBackend.configure_user)

配置新创建的用户。此方法在创建新用户后立即调用，并可用于执行自定义设置操作，例如根据LDAP目录中的属性设置用户的组。返回用户对象。

