
**`java/com/lxz/hookactivity.java`**

```java
package com.lxz;
import android.app.Activity;
import android.content.pm.PackageManager;
import androidx.appcompat.app.AppCompatActivity;
// 注意这里继承的 AppCompatActivity 类是目标应用 activity 继承的类
public class hookactivity extends AppCompatActivity {
@Override // android.content.ContextWrapper, android.content.Context
public PackageManager getPackageManager() {
PackageManager pm = new hook(super.getPackageManager());
return pm;
}
}
```

修改目标主启动类为 `hookactivity`，其实就是多了一层继承，中间修改了 PMS

```java
public class MainActivity extends hookactivity {
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
TextView textView = findViewById(R.id.mytext);
String str = getSignatuer(this);
if(checkSignatuer(str))
{
Log.d("lxz","签名正确");
textView.setText("签名正确");
}else{
Log.d("lxz","签名错误");
textView.setText("签名错误");
}
}
......
}
```

## smail 层实现

使用反编译工具将刚刚用 java 实现的 PMS 反编译，并取出其中的 `lxz` 文件夹

```
apktool.jar -r d .\app-debug.apk
```

再将 CheckSignature 中 刚刚添加的 psm 部分删掉，让其还原成一个可以检查 Signature 的应用，重新打包成 apk，再使用 apktool 将其解包

```
apktool.jar -r d .\app-debug.apk
```

将 `lxz` 文件夹拷贝到 `smali_classes3/com` 目录下，并修改 `smali_classes3\com\example\checksignatuer\MainActivity.smali` 的父类

`androidx.multidex.MultiDexApplication`

`com\babybus\app\App.smali`

```
.class public Lcom/example/checksignatuer/MainActivity;
#.super Landroidx/activity/ComponentActivity;
.super Lcom/lxz/hookactivity;
.source "MainActivity.java"
```

重新打包回 apk 文件，生成 `signature.apk`

```
apktool.jar b app-debug -o signature.apk
```

使用 `zipalign` 对生成的文件对齐，需要制作 v2 的签名，不然 android 11 及以后安装会失败

```
zipalign -v 4 signature.apk zipalign.apk
```

接下来我们需要进行签名，但我们现在还没有签名文件，所以还得做个签名文件，密码`123456`，问题答案随便填，这样我们就获得了签名文件 `demo.keystore`

```
keytool -genkey -alias demo.keystore -keyalg RSA -validity 40000 -keystore demo.keystore
```

使用生成的签名文件对目标应用进行签名，此时安装并启动 app 会发现强制返回正确签名的 log

```
apksigner.jar sign --ks demo.keystore --ks-key-alias demo.keystore --ks-pass pass:123456 --key-pass pass:123456 --out signature_final.apk zipalign.apk
```

# IO 重定向

简单介绍下 io 重定向，编写一个自定义的文件打开函数来替代、修改标准的文件打开函数。在这个自定义函数中，可以实现 IO 重定向的逻辑，比如将文件的读写重定向到其他文件，说白了就是把 `open` 函数中的路径参数换成我们想要的路径

## CheckSignature

老套路还是先制作一个通过 `base.apk` 检查 signature 的 app，相较于之前我们只需要改下 `getSignatuer` 这个函数，另外最好新建一个 带 native 的工程会方便些

```java
private String getSignatuer(Context context) {
try (ZipFile zipFile = new ZipFile(getPackageResourcePath())) {
Enumeration<? extends ZipEntry> entries = zipFile.entries();
// 遍历ZIP文件中的每个条目
while (entries.hasMoreElements()) {
ZipEntry entry = entries.nextElement();
// 检查条目名称是否匹配模式"META-INF/\*.RSA"、"META-INF/\*.DSA"或"META-INF/\*.EC"
if (entry.getName().matches("(META-INF/.\*)\\.(RSA|DSA|EC)")) {
// 从ZIP文件中获取当前条目的输入流
InputStream is = zipFile.getInputStream(entry);
// 创建一个CertificateFactory实例，使用"X509"证书类型
CertificateFactory certFactory = CertificateFactory.getInstance("X509");
// 通过输入流生成一个X509Certificate对象，表示证书
X509Certificate x509Cert = (X509Certificate) certFactory.generateCertificate(is);
// 将证书的编码字节转换为十六进制表示
return bytesToHex(x509Cert.getEncoded());
}
}
} catch (Exception e) {
e.printStackTrace();
}
// 如果找不到匹配的签名，返回空字符串
return "";
}
```

## Hook Open

此时竟有些不知如何提笔接着写，因为笔者在这里踩了太多的坑，以至于距离上次写这篇笔记已过去一个月有余，虽然可以像上边一样直接放出答案，但还是决定把我遇到的几个坑复现给大家，心急的也可以直接跳过看最后的答案

IO 重定向这里我参考了网上的几个帖子和 github 上的项目，无一例外，所有的矛头都指向了 `libc` 中的这四个函数 `open`、`open64`、`openat`、`openat64`，大致的 hook 伪代码大概是下面这样样子，只要找个 hook 框架就可以直接起飞

```cpp
int (*old_open)(const char *, int, mode_t);
static int new_open(const char *pathname, int flags, mode_t mode) {
if (strcmp(pathname, apkPath__) == 0){
return old_open(repPath__, flags, mode);
}
return old_open(pathname, flags, mode);
}
hook("libc.so", "open", new_open, (void **) &old_open);
```

但当我实际进行 hook 时却发现了一个非常致命的问题，我在 `libc.so` 的 `open` 可以 hook 到几乎所有被打开的文件，但唯独缺少最重要的目标文件 `base.apk`，一度我以为是我 hook 姿势的问题，甚至我还换了好几个 hook 框架，但大同小异，都无法 hook 到 `base.apk`，这里就卡了我好久，下面直接放分析思路

首先是找到读取 Zip 文件使用类为 `ZipFile`

![image-20230727234951039](upload/attach/202308/950902_GKCEAPPA42H64B8.jpg)

跟进去发现是调用 两个参数的 `ZipFile`，继续跟

![image-20230727235235734](upload/attach/202308/950902_P3SSJ35SW635TVX.jpg)

继续跟会发现最后调用了三个参数的 `ZipFile` 函数，在其中可以发现调用了一个 jni 的 open 函数，`ZipFile` 所在的类是 `package java.util.zip`，所以我们需要找到其对应的 Native 代码

![image-20230727235347536](upload/attach/202308/950902_TNHNUD7RADUT9CV.jpg)

在在线源码网站中搜索一番找到了下面这段代码，从文件名可以确定这就是我们要找的地方了，这里面有两个函数比较重要，一个是 `ZIP_Get_From_Catch` 另一个是 `JVM_Open`

![image-20230727235757098](upload/attach/202308/950902_6WV5K6S4KU5AQR9.jpg)

先看 `JVM_Open`，跟进去后发现调用了一个名为 `open` 的函数，说实话因为是在线源码，没有办法直接看这个 open 到底是啥，但在和群里大佬探讨的时候，大佬斩钉截铁的告诉我，这个 open 就是 `libc` 的 open ！事已至此，hook 不到 `bask.apk` 的原因只有一个！那就是 `ZIP_Get_From_Catch` ！

![image-20230728000430084](upload/attach/202308/950902_DK2YSNYC72AAM5T.jpg)

如何验证我们的猜想呢，因为 `ZIP_Get_From_Catch` 并不是一个导出函数，所以验证起来手段就少了许多，我选择了一个对我来讲比较简单，并且非常粗暴的方法，那就是在 art 源码里加 log，并且还发现了之前分析的一个问题，那就是 native 的文件找错了，它的真正路径是 **`libcore/ojluni/src/main/native/ZipFile.c`**

![image-20230728215308391](upload/attach/202308/950902_STEEMTKK73749MP.jpg)

刷机后重新启动 app 点击获取签名的按钮后，可以看到如下日志，很明显在还未获取签名时系统加载过一次 `base.apk`，而在我们点击获取签名时走的是 `ZIP_Get_From_Catch` 分支，这个地方我有尝试过将 hook 的时机提到 so init 的时机，但还是 hook 不到这个系统加载的这次打开，只能暂且认为系统加载这个 zip 的时候 app 还没有启动，并且这里还有一点其实卡了我挺久的问题，就是是否走 catch 的这种情况和ndk版本以及as版本似乎都有关系，目前发现的规律就是低版本ndk不会走catch，直接 hook `open` 就可以，高版本在创建项目时如果选择不带ndk的模板，会走 `jvm_open`，不会走catch，如果选择带 ndk 的模板，就会像我遇到这种情况一样，原因暂时未知，只能说android 碎片化过于严重，防守方还是可以通过很多版本机制进行抵抗的，进攻方则需要不断找到新路线的关键 api

![image-20230728215335148](upload/attach/202308/950902_XSPES6JGBFK7KAH.jpg)

## shadowhook

于是乎这里我们可以做的选择就比较有限了，hook `ZipFile_open` 这个导出的 jni 函数貌似是可行的，但事情真的会如此顺利么，这里我有尝试使用 `xhook` 和 `shadowhook` 去 hook 这个 `ZipFile_open` 函数，但不好意思，全部翻车，`xhook` 完全 hook 不到（不排除是我姿势不对，毕竟没怎么用过），`shadowhook` 则比较诡异，只能 hook 到一次，这里我还问了我们公司的大佬，只能说这个框架确实是多少有问题的

![image-20230728010250190](upload/attach/202308/950902_5SY3XC8UFZX252N.jpg)

## Find Symbol

一个 jni 函数又是走缓存又是不给 hook 的，我反手就是一个 diy inlinehook，倒要看看它还能不能继续这么傲娇，话不多说，继续参考网上各路大神的 inlinehook 帖子和项目，写代码嘛，主打的就是一个抄，肉丝大佬的代码更是得大抄特抄

首先是 so 解析部分，想要 hook 一个函数首先要拿到它在内存中的地址，本来这里准备直接照抄肉丝老师的代码的，但没想到肉丝老师的代码无法正确解析 `libopenjdk.so` 这个库（痛苦面具直接），最后经过几天的学习与分析，最终定位问题所在是符号表的大小计算有问题，肉丝大佬的代码中，直接以字符串表头的地址减符号表头的地址得到的大小作为整个符号表的大小，但这样其实是有问题的，有啥问题我也不知道，但我知道的是像他那样计算在访问字符串的时候会越界，GPT 说正确的判断符号表的大小的方式是从符号表头遍历到最后 `st_name` 为 0 时作为符号表的结束

![image-20230728012233796](upload/attach/202308/950902_FRHCJRXPX2EMKXG.jpg)

但不幸的是我使用 010 editor 中查看 `libopenjdk.so` 符号表的结尾并不是 0，而是 `00 00 02 00` 这种诡异的数据，不过通过观察发现一个正常的 `sym.st_info` 的值只为 `0x12 0x11 0x22`，如果不是这三个就结束了，这里就先这样判断吧（我太难了...）

![image-20230728012758427](upload/attach/202308/950902_4H7AYY9PM8J7CC8.jpg)

这里给出可以识别到 `ZipFile_open` 的代码（符号表的大小判断估计还是有问题的，所以找到就赶紧 break，如果想通用一点还是得多观察几个 lib，看看结尾的共同之处）
