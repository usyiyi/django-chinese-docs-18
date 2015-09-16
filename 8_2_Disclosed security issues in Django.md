# 安全问题归档

Django的开发小组坚定地承诺，为报告和公开安全相关问题负责，这在[_Django的安全问题_](../internals/security.html)中列出。

作为承诺的一部分，我们保留了下面的问题的历史列表，这些问题已经被解决和公开。对于每个问题，下面的列表包含日期、简短的描述、[CVE 标识符](http://en.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures)、受影响的版本列表、完整的页面链接以及相应补丁的连接。

有一些重要的附加说明：

*   列出的受影响版本只包含了在漏洞公开时期的Django稳定的安全支持发行版。这意味着，老的版本（安全支持已经过期），以及预发行版本（alpha/beta/RC）在漏洞公开的时期也可能会受影响，但是没有列出。
*   Django项目偶尔会发布安全公告，指出潜在的安全问题，可能会由不合理的配置或其他Django本身以外的问题产生。这些公告中有一些收到了CVE；这种情况下，它们会在这里列出来，但是没有任何附加的补丁或者发行版，只有描述、公开信息和CVE。

## Issues prior to Django’s security process

一些安全问题在Django具有规范化的安全处理流程之前被修复。对于这些问题，可能不会发布新的发行版，也不会分配CVE。

### August 16, 2006 - CVE-2007-0404

[CVE-2007-0404](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2007-0404&cid=3): 翻译框架中的文件名验证问题。[Full description](https://www.djangoproject.com/weblog/2006/aug/16/compilemessages/)

#### Versions affected

*   Django 0.90 [(patch)](https://github.com/django/django/commit/518d406e53)
*   Django 0.91 [(patch)](https://github.com/django/django/commit/518d406e53)
*   Django 0.95 [(patch)](https://github.com/django/django/commit/a132d411c6) (released January 21 2007)

### January 21, 2007 - CVE-2007-0405

[CVE-2007-0405](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2007-0405&cid=3): 已认证用户的可见“缓存”。[Full description](https://www.djangoproject.com/weblog/2007/jan/21/0951/)

#### Versions affected

*   Django 0.95 [(patch)](https://github.com/django/django/commit/e89f0a6558)

## Issues under Django’s security process

所有其它的安全问题都已经在Django安全处理流程下的版本中解决。下面会列出来：

### October 26, 2007 - CVE-2007-5712

[CVE-2007-5712](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2007-5712&cid=3): 通过任意大尺寸`Accept-Language`协议头的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2007/oct/26/security-fix/)

#### Versions affected

*   Django 0.91 [(patch)](https://github.com/django/django/commit/8bc36e726c9e8c75c681d3ad232df8e882aaac81)
*   Django 0.95 [(patch)](https://github.com/django/django/commit/412ed22502e11c50dbfee854627594f0e7e2c234)
*   Django 0.96 [(patch)](https://github.com/django/django/commit/7dd2dd08a79e388732ce00e2b5514f15bd6d0f6f)

### May 14, 2008 - CVE-2008-2302

[CVE-2008-2302](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2008-2302&cid=3): 通过admin登录重定向的XSS。[Full description](https://www.djangoproject.com/weblog/2008/may/14/security/)

#### Versions affected

*   Django 0.91 [(patch)](https://github.com/django/django/commit/50ce7fb57d)
*   Django 0.95 [(patch)](https://github.com/django/django/commit/50ce7fb57d)
*   Django 0.96 [(patch)](https://github.com/django/django/commit/7791e5c050)

### September 2, 2008 - CVE-2008-3909

[CVE-2008-3909](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2008-3909&cid=3): 通过在admin登录状态下保存POST数据的CSRF。[Full description](https://www.djangoproject.com/weblog/2008/sep/02/security/)

#### Versions affected

*   Django 0.91 [(patch)](https://github.com/django/django/commit/44debfeaa4473bd28872c735dd3d9afde6886752)
*   Django 0.95 [(patch)](https://github.com/django/django/commit/aee48854a164382c655acb9f18b3c06c3d238e81)
*   Django 0.96 [(patch)](https://github.com/django/django/commit/7e0972bded362bc4b851c109df2c8a6548481a8e)

### July 28, 2009 - CVE-2009-2659

[CVE-2009-2659](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2009-2659&cid=3): 开发服务器的媒体处理器上的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2009/jul/28/security/)

#### Versions affected

*   Django 0.96 [(patch)](https://github.com/django/django/commit/da85d76fd6)
*   Django 1.0 [(patch)](https://github.com/django/django/commit/df7f917b7f)

### October 9, 2009 - CVE-2009-3965

[CVE-2009-3965](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2009-3695&cid=3): 通过执行异常正则表达式的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2009/oct/09/security/)

#### Versions affected

*   Django 1.0 [(patch)](https://github.com/django/django/commit/594a28a904)
*   Django 1.1 [(patch)](https://github.com/django/django/commit/e3e992e18b)

### September 8, 2010 - CVE-2010-3082

[CVE-2010-3082](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2010-3082&cid=3): 通过不安全cookie值的XSS。[Full description](https://www.djangoproject.com/weblog/2010/sep/08/security-release/)

#### Versions affected

*   Django 1.2 [(patch)](https://github.com/django/django/commit/7f84657b6b)

### December 22, 2010 - CVE-2010-4534

[CVE-2010-4534](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2010-4534&cid=3): 管理界面上的信息泄露。[Full description](https://www.djangoproject.com/weblog/2010/dec/22/security/)

#### Versions affected

*   Django 1.1 [(patch)](https://github.com/django/django/commit/17084839fd)
*   Django 1.2 [(patch)](https://github.com/django/django/commit/85207a245b)

### December 22, 2010 - CVE-2010-4535

[CVE-2010-4535](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2010-4535&cid=2): 密码重置机制上的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2010/dec/22/security/)

#### Versions affected

*   Django 1.1 [(patch)](https://github.com/django/django/commit/7f8dd9cbac)
*   Django 1.2 [(patch)](https://github.com/django/django/commit/d5d8942a16)

### February 8, 2011 - CVE-2011-0696

[CVE-2011-0696](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-0696&cid=2): 通过伪造HTTP协议头的XSS。[Full description](https://www.djangoproject.com/weblog/2011/feb/08/security/)

#### Versions affected

*   Django 1.1 [(patch)](https://github.com/django/django/commit/408c5c873c)
*   Django 1.2 [(patch)](https://github.com/django/django/commit/818e70344e)

### February 8, 2011 - CVE-2011-0697

[CVE-2011-0697](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-0697&cid=2): 通过未检查的名称或者上传文件的XSS。[Full description](https://www.djangoproject.com/weblog/2011/feb/08/security/)

#### Versions affected

*   Django 1.1 [(patch)](https://github.com/django/django/commit/1966786d2d)
*   Django 1.2 [(patch)](https://github.com/django/django/commit/1f814a9547)

### February 8, 2011 - CVE-2011-0698

[CVE-2011-0698](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-0698&cid=2): Windows上通过不正确的目录分隔符处理的目录遍历。[Full description](https://www.djangoproject.com/weblog/2011/feb/08/security/)

#### Versions affected

*   Django 1.1 [(patch)](https://github.com/django/django/commit/570a32a047)
*   Django 1.2 [(patch)](https://github.com/django/django/commit/194566480b)

### September 9, 2011 - CVE-2011-4136

[CVE-2011-4136](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-4136&cid=2):使用memory-cache-backed会话时的会话操纵。[Full description](https://www.djangoproject.com/weblog/2011/sep/09/security-releases-issued/)

#### Versions affected

*   Django 1.2 [(patch)](https://github.com/django/django/commit/ac7c3a110f)
*   Django 1.3 [(patch)](https://github.com/django/django/commit/fbe2eead2f)

### September 9, 2011 - CVE-2011-4137

[CVE-2011-4137](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-4137&cid=2): 通过`URLField.verify_exists`的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2011/sep/09/security-releases-issued/)

#### Versions affected

*   Django 1.2 [(patch)](https://github.com/django/django/commit/7268f8af86)
*   Django 1.3 [(patch)](https://github.com/django/django/commit/1a76dbefdf)

### September 9, 2011 - CVE-2011-4138

[CVE-2011-4138](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-4138&cid=2): 通过`URLField.verify_exists`的信息泄露/任何请求发布。[Full description](https://www.djangoproject.com/weblog/2011/sep/09/security-releases-issued/)

#### Versions affected

*   Django 1.2: [(patch)](https://github.com/django/django/commit/7268f8af86)
*   Django 1.3: [(patch)](https://github.com/django/django/commit/1a76dbefdf)

### September 9, 2011 - CVE-2011-4139

[CVE-2011-4139](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-4139&cid=2): `Host`协议头缓存污染。 [Full description](https://www.djangoproject.com/weblog/2011/sep/09/security-releases-issued/)

#### Versions affected

*   Django 1.2 [(patch)](https://github.com/django/django/commit/c613af4d64)
*   Django 1.3 [(patch)](https://github.com/django/django/commit/2f7fadc38e)

### September 9, 2011 - CVE-2011-4140

[CVE-2011-4140](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2011-4140&cid=2):通过`Host`协议头的潜在CSRF威胁。[Full description](https://www.djangoproject.com/weblog/2011/sep/09/security-releases-issued/)

#### Versions affected

这个通知只是一个公告，没有任何补丁发布。

*   Django 1.2
*   Django 1.3

### July 30, 2012 - CVE-2012-3442

[CVE-2012-3442](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2012-3442&cid=2): 通过验证重定向模式失败的XSS。[Full description](https://www.djangoproject.com/weblog/2012/jul/30/security-releases-issued/)

#### Versions affected

*   Django 1.3: [(patch)](https://github.com/django/django/commit/4dea4883e6c50d75f215a6b9bcbd95273f57c72d)
*   Django 1.4: [(patch)](https://github.com/django/django/commit/e34685034b60be1112160e76091e5aee60149fa1)

### July 30, 2012 - CVE-2012-3443

[CVE-2012-3443](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2012-3443&cid=2): 通过压缩的图像文件的拒绝服务u攻击。[Full description](https://www.djangoproject.com/weblog/2012/jul/30/security-releases-issued/)

#### Versions affected

*   Django 1.3: [(patch)](https://github.com/django/django/commit/b2eb4787a0fff9c9993b78be5c698e85108f3446)
*   Django 1.4: [(patch)](https://github.com/django/django/commit/c14f325c4eef628bc7bfd8873c3a72aeb0219141)

### July 30, 2012 - CVE-2012-3444

[CVE-2012-3444](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2012-3444&cid=2):通过大尺寸图像文件的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2012/jul/30/security-releases-issued/)

#### Versions affected

*   Django 1.3 [(patch)](https://github.com/django/django/commit/9ca0ff6268eeff92d0d0ac2c315d4b6a8e229155)
*   Django 1.4 [(patch)](https://github.com/django/django/commit/da33d67181b53fe6cc737ac1220153814a1509f6)

### October 17, 2012 - CVE-2012-4520

[CVE-2012-4520](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2012-4520&cid=2): `Host`协议头污染。[Full description](https://www.djangoproject.com/weblog/2012/oct/17/security/)

#### Versions affected

*   Django 1.3 [(patch)](https://github.com/django/django/commit/b45c377f8f488955e0c7069cad3f3dd21910b071)
*   Django 1.4 [(patch)](https://github.com/django/django/commit/92d3430f12171f16f566c9050c40feefb830a4a3)

### December 10, 2012 - No CVE 1

对`Host`协议头处理的额外加固。[Full description](https://www.djangoproject.com/weblog/2012/dec/10/security/)

#### Versions affected

*   Django 1.3 [(patch)](https://github.com/django/django/commit/2da4ace0bc1bc1d79bf43b368cb857f6f0cd6b1b)
*   Django 1.4 [(patch)](https://github.com/django/django/commit/319627c184e71ae267d6b7f000e293168c7b6e09)

### December 10, 2012 - No CVE 2

对重定向验证的额外加固。[Full description](https://www.djangoproject.com/weblog/2012/dec/10/security/)

#### Versions affected

*   Django 1.3: [(patch)](https://github.com/django/django/commit/1515eb46daa0897ba5ad5f0a2db8969255f1b343)
*   Django 1.4: [(patch)](https://github.com/django/django/commit/b2ae0a63aeec741f1e51bac9a95a27fd635f9652)

### February 19, 2013 - No CVE

对`Host`协议头处理的额外加固。[Full description](https://www.djangoproject.com/weblog/2013/feb/19/security/)

#### Versions affected

*   Django 1.3 [(patch)](https://github.com/django/django/commit/27cd872e6e36a81d0bb6f5b8765a1705fecfc253)
*   Django 1.4 [(patch)](https://github.com/django/django/commit/9936fdb11d0bbf0bd242f259bfb97bbf849d16f8)

### February 19, 2013 - CVE-2013-1664/1665

[CVE-2013-1664](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-1664&cid=2) and [CVE-2013-1665](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-1665&cid=2): 对Python XML库的基于实体的攻击。[Full description](https://www.djangoproject.com/weblog/2013/feb/19/security/)

#### Versions affected

*   Django 1.3 [(patch)](https://github.com/django/django/commit/d19a27066b2247102e65412aa66917aff0091112)
*   Django 1.4 [(patch)](https://github.com/django/django/commit/1c60d07ba23e0350351c278ad28d0bd5aa410b40)

### February 19, 2013 - CVE-2013-0305

[CVE-2013-0305](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-0305&cid=2): 通过admin历史记录的信息泄露。[Full description](https://www.djangoproject.com/weblog/2013/feb/19/security/)

#### Versions affected

*   Django 1.3 [(patch)](https://github.com/django/django/commit/d3a45e10c8ac8268899999129daa27652ec0da35)
*   Django 1.4 [(patch)](https://github.com/django/django/commit/0e7861aec73702f7933ce2a93056f7983939f0d6)

### February 19, 2013 - CVE-2013-0306

[CVE-2013-0306](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-0306&cid=2): 通过表单集`max_num` 的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2013/feb/19/security/)

#### Versions affected

*   Django 1.3 [(patch)](https://github.com/django/django/commit/d7094bbce8cb838f3b40f504f198c098ff1cf727)
*   Django 1.4 [(patch)](https://github.com/django/django/commit/0cc350a896f70ace18280410eb616a9197d862b0)

### August 13, 2013 - Awaiting CVE 1

(CVE not yet issued): 通过admin受信任的`URLField`值的XSS。[Full description](https://www.djangoproject.com/weblog/2013/aug/13/security-releases-issued/)

#### Versions affected

*   Django 1.5 [(patch)](https://github.com/django/django/commit/90363e388c61874add3f3557ee654a996ec75d78)

### August 13, 2013 - Awaiting CVE 2

(CVE not yet issued):可能的XSS漏洞，通过未验证的URL重定向模式。[Full description](https://www.djangoproject.com/weblog/2013/aug/13/security-releases-issued/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/ec67af0bd609c412b76eaa4cc89968a2a8e5ad6a)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/1a274ccd6bc1afbdac80344c9b6e5810c1162b5f)

### September 10, 2013 - CVE-2013-4315

[CVE-2013-4315](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2013-4315&cid=2) 通过`ssi`模板标签的目录遍历。[Full description](https://www.djangoproject.com/weblog/2013/sep/10/security-releases-issued/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/87d2750b39f6f2d54b7047225521a44dcd37e896)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/988b61c550d798f9a66d17ee0511fb7a9a7f33ca)

### September 14, 2013 - CVE-2013-1443

CVE-2013-1443: 通过长密码的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2013/sep/15/security/)

#### Versions affected

*   Django 1.4 [(patch](https://github.com/django/django/commit/3f3d887a6844ec2db743fee64c9e53e04d39a368) and [Python compatibility fix)](https://github.com/django/django/commit/6903d1690a92aa040adfb0c8eb37cf62e4206714)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/22b74fa09d7ccbc8c52270d648a0da7f3f0fa2bc)

### April 21, 2014 - CVE-2014-0472

[CVE-2014-0472](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-0472&cid=2): 使用`reverse()`的非预期代码执行。[Full description](https://www.djangoproject.com/weblog/2014/apr/21/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/c1a8c420fe4b27fb2caf5e46d23b5712fc0ac535)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/2a5bcb69f42b84464b24b5c835dca6467b6aa7f1)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/4352a50871e239ebcdf64eee6f0b88e714015c1b)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/546740544d7f69254a67b06a3fc7fa0c43512958)

### April 21, 2014 - CVE-2014-0473

[CVE-2014-0473](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-0473&cid=2): 匿名页面的缓存可能会泄露CSRF标识。[Full description](https://www.djangoproject.com/weblog/2014/apr/21/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/1170f285ddd6a94a65f911a27788ba49ca08c0b0)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/6872f42757d7ef6a97e0b6ec5db4d2615d8a2bd8)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/d63e20942f3024f24cb8cd85a49461ba8a9b6736)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/380545bf85cbf17fc698d136815b7691f8d023ca)

### April 21, 2014 - CVE-2014-0474

[CVE-2014-0474](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-0474&cid=2): MySQL类型转换产生非预期的查询结果。[Full description](https://www.djangoproject.com/weblog/2014/apr/21/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/aa80f498de6d687e613860933ac58433ab71ea4b)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/985434fb1d6bf2335bf96c6ebf91c3674f1f399f)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/5f0829a27e85d89ad8c433f5c6a7a7d17c9e9292)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/34526c2f56b863c2103655a0893ac801667e86ea)

### May 18, 2014 - CVE-2014-1418

[CVE-2014-1418](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-1418&cid=2): 缓存可能允许存储和处理私人数据。[Full description](https://www.djangoproject.com/weblog/2014/may/14/security-releases-issued/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/28e23306aa53bbbb8fb87db85f99d970b051026c)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/4001ec8698f577b973c5a540801d8a0bbea1205b)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/1abcf3a808b35abae5d425ed4d44cb6e886dc769)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/7fef18ba9e5a8b47bc24b5bb259c8bf3d3879f2a)

### May 18, 2014 - CVE-2014-3730

[CVE-2014-3730](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-3730&cid=2): 来源于用户输入的错误格式URL的不正确验证。[Full description](https://www.djangoproject.com/weblog/2014/may/14/security-releases-issued/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/7feb54bbae3f637ab3c4dd4831d4385964f574df)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/ad32c218850ad40972dcef57beb460f8c979dd6d)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/601107524523bca02376a0ddc1a06c6fdb8f22f3)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/e7b0cace455c2da24492660636bfd48c45a19cdf)

### August 20, 2014 - CVE-2014-0480

[CVE-2014-0480](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-0480&cid=2): reverse() 可能会生成指向其它域名的URL。[Full description](https://www.djangoproject.com/weblog/2014/aug/20/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/c2fe73133b62a1d9e8f7a6b43966570b14618d7e)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/45ac9d4fb087d21902469fc22643f5201d41a0cd)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/da051da8df5e69944745072611351d4cfc6435d5)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/bf650a2ee78c6d1f4544a875dcc777cf27fe93e9)

### August 20, 2014 - CVE-2014-0481

[CVE-2014-0481](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-0481&cid=2): 文件上传的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2014/aug/20/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/30042d475bf084c6723c6217a21598d9247a9c41)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/26cd48e166ac4d84317c8ee6d63ac52a87e8da99)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/dd0c3f4ee1a30c1a1e6055061c6ba6e58c6b54d1)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/3123f8452cf49071be9110e277eea60ba0032216)

### August 20, 2014 - CVE-2014-0482

[CVE-2014-0482](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-0482&cid=2): RemoteUserMiddleware会话劫持。[Full description](https://www.djangoproject.com/weblog/2014/aug/20/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/c9e3b9949cd55f090591fbdc4a114fcb8368b6d9)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/dd68f319b365f6cb38c5a6c106faf4f6142d7d88)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/0268b855f9eab3377f2821164ef3e66037789e09)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/1a45d059c70385fcd6f4a3955f3b4e4cc96d0150)

### August 20, 2014 - CVE-2014-0483

[CVE-2014-0483](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-0483&cid=2):&nbsp;admin中查询集操作产生的数据泄露。[Full description](https://www.djangoproject.com/weblog/2014/aug/20/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/027bd348642007617518379f8b02546abacaa6e0)
*   Django 1.5 [(patch)](https://github.com/django/django/commit/2a446c896e7c814661fb9c4f212b071b2a7fa446)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/f7c494f2506250b8cb5923714360a3642ed63e0f)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/2b31342cdf14fc20e07c43d258f1e7334ad664a6)

### January 13, 2015 - CVE-2015-0219

[CVE-2015-0219](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2015-0219&cid=2): 通过下划线或者破折号合并产生的WSGI协议头欺骗。[Full description](https://www.djangoproject.com/weblog/2015/jan/13/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/4f6fffc1dc429f1ad428ecf8e6620739e8837450)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/d7597b31d5c03106eeba4be14a33b32a5e25f4ee)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/41b4bc73ee0da7b2e09f4af47fc1fd21144c710f)

### January 13, 2015 - CVE-2015-0220

[CVE-2015-0220](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2015-0220&cid=2):&nbsp;
通过用户提供的重定向URL的可能的XSS攻击。[Full description](https://www.djangoproject.com/weblog/2015/jan/13/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/4c241f1b710da6419d9dca160e80b23b82db7758)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/72e0b033662faa11bb7f516f18a132728aa0ae28)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/de67dedc771ad2edec15c1d00c083a1a084e1e89)

### January 13, 2015 - CVE-2015-0221

[CVE-2015-0221](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2015-0221&cid=2): `django.views.static.serve()`上的拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2015/jan/13/security/)

#### Versions affected

*   Django 1.4 [(patch)](https://github.com/django/django/commit/d020da6646c5142bc092247d218a3d1ce3e993f7)
*   Django 1.6 [(patch)](https://github.com/django/django/commit/553779c4055e8742cc832ed525b9ee34b174934f)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/818e59a3f0fbadf6c447754d202d88df025f8f2a)

### January 13, 2015 - CVE-2015-0222

[CVE-2015-0222](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2015-0222&cid=2): 使用`ModelMultipleChoiceField`的数据库拒绝服务攻击。[Full description](https://www.djangoproject.com/weblog/2015/jan/13/security/)

#### Versions affected

*   Django 1.6 [(patch)](https://github.com/django/django/commit/d7a06ee7e571b6dad07c0f5b519b1db02e2a476c)
*   Django 1.7 [(patch)](https://github.com/django/django/commit/bcfb47780ce7caecb409a9e9c1c314266e41d392)

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[Disclosed security issues in Django](https://docs.djangoproject.com/en/1.8/releases/security/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
