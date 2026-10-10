
# Android 应用破解之 SSHDroid 去广告

> 原文：[`www.pediy.com/kssd/pediy12/142708.html`](https://www.pediy.com/kssd/pediy12/142708.html)

SSHDroid 这个软件是用来在 android 手机上架设 ssh 服务器的，这个小应用有两个版本一个免费版，一个 pro 版。区别就是有无广告。

发现装了 Adfree android 这个应用之后，SSHDroid 就不干活了，说 ad 被 block 了，然后就只能选择“退出”或者“购买付费版”。

这个，多少让我有些不爽啊。

如图：提示广告被屏蔽，只能点“退出”或者使用收费版（adfree version）：

![](img/c7fcbd4b469789d0c94045ebae79ee3d.jpg)

有了想暴破它的想法了~~于是操刀把它菊花暴了，爽了一把。

在 linux 下面那个 `adb` 硬是没有运行起来，运行以后没有任何输出，不知何故。先在 win 下把这 app 搞定吧。

用到的暴菊工具：

引用: `adb.rar` （从网上下载的，里面有 `adb` 及其依赖的库，我懒得装 android sdk 了。。。）

`Auto-sign（WIN7 系统）.rar` 用于给 app 签名的，随便在网上搜索到的

`apktool1.4.1.tar.bz2` 在 google 项目：http://code.google.com/p/android-apktool/

`apktool-install-windows-r04-brut1.tar.bz2`

`EmEditor` （我机子上一直装着呢）

先说下 adfree android 这个软件屏蔽广告的原理，其实这个软件就是修改 andorid `/system/etc/hosts` 这个文件，把 ad 相关域名的 ip 指向 `127.0.0.1` 来达到屏蔽广告的目的。

爪机注意开启调试。

从机器里把程序拖出来（不知道 app 文件名就先用 `ls` 找一下）：

```
adb pull /data/app/berserker.android.apps.sshdroid-1.apk
```

然后用 `apktool` 对其进行解压缩和反汇编

```
apktool d berserker.android.apps.sshdroid-1.apk reversing
```

进入到 `reversing` 目录,里面有露出来的菊花(反汇编出来的东东)：

```
cd reversing
```

下面是正式暴菊了。

`zhiwei.li` 给出了三种方案来搞定 ad：

引用: 1. 对 `com.google.ads` 打补丁. 这将是一个比较通用的方法,对那些使用到 `com.google.ads` 的所有应用都有效.

2. 对 app 打补丁. 找到此 app 调用 `com.google.ads` 的地方

3. 对资源文件进行补丁,让广告资源不可见.

我这里采用的是 2 和 3 相结合的方法，算是比较彻底。

进入 `smali` 目录，用 `emeditor` 随便打开一个 `.smali` 文件，在此目录及其子目录搜索 `/etc/hosts`

共找到两个：

引用: `smali\berserker\android\a\a.smali(59)`

`smali\berserker\android\a\f.smali(227)`

打开反汇编的文件一看，前一个是定义的检测 hosts 的类。很快可以发现：

```
const-string v3, "admob"
```

这个程序是投放的 admob 广告。

```
const-string v3, "admob"
invoke-virtual {v2, v3}, Ljava/lang/String;->contains(Ljava/lang/CharSequence;)Z
move-result v2
if-eqz v2, :cond_0
sget-object v1, Ljava/lang/Boolean;->TRUE:Ljava/lang/Boolean;
sput-object v1, Lberserker/android/a/a;->b:Ljava/lang/Boolean;
```

从以上代码可以看出，它是检测 hosts 中是否有 `admob` 字符串，然后根据返回的结果要决定跳转还是不跳转.

这里可以修改常量，也可以修改 `if-eqz` 为 `if-nez`. 我是采用的修改常量，

把前面的代码:

```
const-string v3, "admob"
```

修改为：

```
const-string v3, "notexistsdomain"
```

这里的字串随便，要求就是在你的手机的 hosts 文件里所有条目中都不出现。

因为这样修改的话，不管有没有修改 hosts 文件的情况都不会有问题的。比较通用。

这里的修改是基于阅读反汇编代码的，并不是胡乱修改。

对于 android 的 JVM 指令集，可看查看这里，非常详细：

http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html

http://loda.hala01.com/dalvik-opcodes/

http://thinkinmylife.iteye.com/blog/443900

第二个文件看了下，没什么要修改的地方。

然后就是修改资源文件了。

到 `res/layout` 目录下面，一看到 `banner.xml` 和 `hosts_alert_dialog.xml` 这两个文件的名字你就大概知道它是干什么的了。

把其高度修改为 0 即可。

如：

```
android:layout_height="0.0dip"
```

完了之后开始收工：

对 apk 重新打包

```
apktools b reversing cracked_berserker.android.apps.sshdroid-1.apk
```

对它签名,以便能在大多数 android 设备上安装, 如果你已经安装了 SDK,那么可以使用 debug key 的证书

```
jarsigner -keystrore ~/.android/debug.keystore cracked_berserker.android.apps.sshdroid-1.apk androiddebugkey
```

然后输入 `android` 作为口令(password)

我这里图方便，直接用 `auto-sign` 啦，把破解后的文件托动到 `Sign.bat` 上即可生成代码:

```
cracked_berserker.android.apps.sshdroid-1.apk.apk
```

好了。

卸载原版软件，也可以在手机上操作，用命令：

```
adb uninstall berserker.android.apps.sshdroid-1
```

然后再安装

```
adb isntall berserker.android.apps.sshdroid-1.apk
```

破解完毕，经测试，使用近一个月，运行一切正常。
