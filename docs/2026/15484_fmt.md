
# [原创]AD Blocker Trial 破解小记

【文章标题】: 【原创】AD Blocker Trial 破解小记

【文章作者】: 漏网之鱼

【作者邮箱】: lwzy-crack@163.com

【作者主页】: 06cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4%4P5Y4W2Q4x3X3c8U0M7X3q4U0K9#2)9J5k6h3u0D9L8$3N6Q4x3X3f1I4y4U0y4Q4x3X3g2U0L8$3@1`.

【作者 QQ】: 530747686

【软件名称】: AD Blocker Trial

【下载地址】: 自己搜索下载

【操作平台】: Windows 7

【软件介绍】: 一款Android平台的广告拦截软件。

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 【软件介绍】

软件简介:英文名字叫 "AD Blocker Trial",中文名有点长——"广告拦截器及净触发器的审判" 。

该软件无需手机用户获得root权限，因为该软件拦截广告原理不是修改hosts文件 127.0.0.1 来的，并且可以单独设置拦截哪一款软件的广告非常人性化值得试用。

使用方法:启动软件后 找到你想要拦截广告的软件名称单击，选择“Block” 就可以了

取消拦截：找到你不想要拦截广告的软件名称单击 选择“Unblock”就可以了 简单吧！

注:有些应用程序如果你第一次没有阻止的话下次再阻止就不行了，因为第一次已经下载好了，解决方法就清理下缓存。

“AD Blocker Trial ”优点无需用户获取root 可以自己选择拦截某款软件的广告 值得下载！

但是它是共享软件，需要注册，软件有时间限制。过期后，软件不能正常使用，并且弹出'NAG'如图所示：

![](https://bbs.kanxue.com/upload/attach/201202/155099_fts6ty3n4in2gcr.jpg)

因为国行手机Root以后就不给保修了，所以我至今没有Root,从"安卓市场"里面下载的很多程序都带有广告，很烦人，而且还浪费流量

于是我就找到了这个软件，用了３天以后不能用了，提示过期，看来只好自己动手"PJ"它了。

--------------------------------------------------------------------------------

## 【详细过程】

准备工作：安装好Java运行环境，没有的朋友去这里下载 9dfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3A6S2N6X3q4Q4x3X3g2U0L8$3#2Q4x3V1k6\*7K9q4)9#2k6V1y4z5i4K6u0r3

用到的工具：ApkTool\_GUI\_1.3.5内测.exe(众多Android程序的GUI版本)，　Notepad++　(非常强大的文本编辑工具，推荐使用  下载地址：7faK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6G2N6r3g2H3j5h3c8Q4x3X3c8H3L8s2g2K6i4K6u0V1M7r3I4#2M7#2)9J5k6h3!0J5k6#2)9J5c8R3`.`. )

### 1、反编译apk：

![](https://bbs.kanxue.com/upload/attach/201202/155099_18rnjix9kd192ns.png)

反编译后生成了ADBlocker\_trial 文件夹，浏览目录发现 `ADBlocker_trial/smali/net/xdevelop/util` 里面 有一个可疑 文件 `RegUtil.smali` ，`*.smali`是什么文件？你问我啊！去看这篇文件，有详细介绍：http://bbs.pediy.com/showthread.php?t=135323

`RegUtil.smali` 内部结构如下：

![](https://bbs.kanxue.com/upload/attach/201202/155099_8u7p91ma3k3m8c7.png)

通过这几个方法的名称，我们大致可以猜测出每个方法的用途：

`check` ，检查是否注册成功，

`getProdCode`  ，生成本手机（`getLocalPhone`）的机器码

用 `Notepad++`打开`ADBlocker_trial/smali/net/xdevelop/adblocker_t`目录下的所有文件，搜索过期提示 `"This trial version has expired, if you like it and want to continue using it, please buy it."`

来到这个文件 `ADBlocker.smali`的 这里

``` 
.method protected onResume()V

    .locals 2

    .prologue

    .line 141

    invoke-super {p0}, Landroid/app/Activity;->onResume()V

    .line 142

    const-string v0, "ap"

    const-string v1, "onResume"

    invoke-static {v0, v1}, Landroid/util/Log;->v(Ljava/lang/String;Ljava/lang/String;)I

    .line 143

    new-instance v0, Lnet/xdevelop/adblocker_t/ADBlocker$2;

    invoke-direct {v0, p0}, Lnet/xdevelop/adblocker_t/ADBlocker$2;-><init>(Lnet/xdevelop/adblocker_t/ADBlocker;)V

    .line 149

    invoke-virtual {v0}, Lnet/xdevelop/adblocker_t/ADBlocker$2;->start()V

    # 这里判断是否过期

    .line 151

    invoke-static {p0}, Lnet/xdevelop/adblocker_t/ProtectorPreferences;->isExpired(Landroid/content/Context;)Z

    move-result v0

    if-eqz v0, :cond_0

    .line 152

    const-string v0, "Expired!"

    const-string v1, "This trial version has expired, if you like it and want to continue using it, please buy it."

    invoke-direct {p0, v0, v1}, Lnet/xdevelop/adblocker_t/ADBlocker;->showDialog(Ljava/lang/String;Ljava/lang/CharSequence;)V

    .line 155

    :cond_0

    return-void

.end method
```

--------------------------------------------------------------------

通过阅读这个文件，我们发现一个非常可疑的 方法![](https://bbs.kanxue.com/view/img/face/41.gif) ：`.method private final reg(Landroid/content/Context;Ljava/lang/String;)V`

仔细阅读后发现：

``` 
.method private final reg(Landroid/content/Context;Ljava/lang/String;)V

    .locals 3

    .parameter "context"

    .parameter "key"

    .prologue

    .line 477

    # 调用 check 函数 判断是否注册

    invoke-static {p1, p2}, Lnet/xdevelop/util/RegUtil;->check(Landroid/content/Context;Ljava/lang/String;)Z

    move-result v1

    # 如果没有注册成功，就跳到  cond_0

    if-eqz v1, :cond_0

    .line 478

    invoke-static {p0}, Lnet/xdevelop/adblocker_t/ProtectorPreferences;->getPref(Landroid/content/Context;)Lnet/xdevelop/adblocker_t/ProtectorPrefModel;

    move-result-object v0

    .line 479

    .local v0, m:Lnet/xdevelop/adblocker_t/ProtectorPrefModel;

    const/4 v1, 0x1

    iput-boolean v1, v0, Lnet/xdevelop/adblocker_t/ProtectorPrefModel;->donated:Z

    .line 480

    invoke-static {p0, v0}, Lnet/xdevelop/adblocker_t/ProtectorPreferences;->savePref(Landroid/content/Context;Lnet/xdevelop/adblocker_t/ProtectorPrefModel;)V

    .line 481

    const v1, 0x7f070011

    invoke-virtual {p0, v1}, Lnet/xdevelop/adblocker_t/ADBlocker;->getText(I)Ljava/lang/CharSequence;

    move-result-object v1

    invoke-interface {v1}, Ljava/lang/CharSequence;->toString()Ljava/lang/String;

    move-result-object v1

    const v2, 0x7f070013

    # 注册成功

    invoke-virtual {p0, v2}, Lnet/xdevelop/adblocker_t/ADBlocker;->getText(I)Ljava/lang/CharSequence;

    move-result-object v2

    invoke-interface {v2}, Ljava/lang/CharSequence;->toString()Ljava/lang/String;

    move-result-object v2

    invoke-direct {p0, v1, v2}, Lnet/xdevelop/adblocker_t/ADBlocker;->showMsg(Ljava/lang/String;Ljava/lang/String;)V

    .line 486

    .end local v0           #m:Lnet/xdevelop/adblocker_t/ProtectorPrefModel;

    :goto_0

    return-void

    .line 484

    :cond_0

    const v1, 0x7f070012

    invoke-virtual {p0, v1}, Lnet/xdevelop/adblocker_t/ADBlocker;->getText(I)Ljava/lang/CharSequence;

    move-result-object v1

    invoke-interface {v1}, Ljava/lang/CharSequence;->toString()Ljava/lang/String;

    move-result-object v1

    const v2, 0x7f070014

    # 注册失败

    invoke-virtual {p0, v2}, Lnet/xdevelop/adblocker_t/ADBlocker;->getText(I)Ljava/lang/CharSequence;

    move-result-object v2

    invoke-interface {v2}, Ljava/lang/CharSequence;->toString()Ljava/lang/String;

    move-result-object v2

    invoke-direct {p0, v1, v2}, Lnet/xdevelop/adblocker_t/ADBlocker;->showMsg(Ljava/lang/String;Ljava/lang/String;)V

    goto :goto_0

.end method
```

------------------------------------------------------------------------------------

``` 
.field public static final upgrade_failed:I = 0x7f070014

.field public static final upgrade_success:I = 0x7f070013
```

------------------------------------------------------------------------------------

破解方法： `if-eqz v1, :cond_0`  这一句话 删掉，保存，用`ApkTool_GUI_1.3.5内测.exe`  打包，OK，搞定，

效果如图：

![](https://bbs.kanxue.com/upload/attach/201202/155099_9kr9dbhs1iiwuii.jpg)

未破解的apk 文件， [ADBlocker_trial.rar](https://bbs.kanxue.com/attach-download-65107.htm)

破解好的apk文件： [ADBlocker_trial(已签名).rar](https://bbs.kanxue.com/attach-download-65120.htm)

--------------------------------------------------------------------------------

【版权声明】: 转载请注明作者并保持文章的完整, 谢谢!

                                                       2012年02月28日 19:55:59

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [过期提示.jpg](javascript:void(0);)

  （905.57kb，22次下载）

- [1.png](javascript:void(0);)

  （205.23kb，29次下载）

- [fun.png](javascript:void(0);)

  （27.49kb，41次下载）

- [IMG0011.jpg](javascript:void(0);)

  （28.88kb，455次下载）

- [ADBlocker_trial.rar](javascript:void(0);)

  （66.21kb，130次下载）

- [ADBlocker_trial(已签名).rar](javascript:void(0);)

  （67.26kb，122次下载）
