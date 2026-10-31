
# [原创]杀死那个名为360安全的软件

## 前言

出去练习了两年半，合同到期闲出了一段时间，重新发个文章活跃保号。先整点铺垫

![](upload/attach/202403/749276_PW3KNNDDWG3EUCX.webp)

## 背景

2023年底，闲来没事想起了`xjun`师傅2021年发的`procexp`驱动利用帖子时在群里讨论的，通过`procexp`驱动突破`PPL`后注入到`csrss`进程中，再通过`csrss`来结束那些个安全防护软件。于是在当时就有了如下成果：

![](upload/attach/202403/749276_G8N623J7S4HTH3F.webp)

文章链接（在职的时候发的）

<mark class="encrypted">b02K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8V1q4j5K9W2k6B7k6%4g2t1j5g2N6V1x3%4u0t1M7g2W2c8y4s2N6X3L8i4M7`.</mark>

`xjun`师傅的`PPL`文章

<mark class="encrypted">f9bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7I4y4e0b7$3x3K6x3$3i4K6u0V1x3g2)9J5k6o6q4Q4x3X3g2Z5N6r3#2D9</mark>

这些弄完之后，觉得通过`hook` `csrss`中的`CsrCreateProcess`来达到主动`kill`指定进程并不优雅。于是对360的`360FsFlt`和`360Hvm64`进行了分析，并从中找到了他的防护缺陷。导致可以用简单的办法结束`ZhuDongFangYu.exe`和`360try.exe`等进程。

## ObjectCallback的降权突破

在`360FsFlt`的回调中，存在一个白名单，而这个白名单为360安装目录下的`exe`，每当运行后都会记录对应的`pid`。

![](upload/attach/202403/749276_6CXEF4BJMZAUXY4.webp)

![](upload/attach/202403/749276_V3K4R37VAG6EW2N.webp)

但是，当你想当然的去启动它的`exe`以此获得白名单时，你会发现启动的`exe`不能注入，不能写。所以他仍然还有手段禁止白名单被利用。不要灰心，在这个阶段我们知道了白名单可以不被降权。那么接下来的目的就是想办法获得一个内存修改的权限。

## 内存修改的康庄大道

在上面的启动白名单的`exe`获得的句柄权限是最高权限，但并没有操作的空间。显然是对`api`做了手脚，于是目光转到了晶核上，在`360Hvm64`中发现了这么个表

![](upload/attach/202403/749276_EACRED8RKTP4ZKF.webp)

![](upload/attach/202403/749276_ZENKZA38RXRE4RJ.webp)

从下图可以发现，在调用`NtWriteVirtualMemory`会有校验

![](upload/attach/202403/749276_WTTTHC9JJBQSZPN.webp)

![](upload/attach/202403/749276_8KYNC36AAN9453X.webp)

到这里可以确定，通过下图的部分`exe`中可以对白名单进行读写注入。

![](upload/attach/202403/749276_P49ZNKYQ7HEQ7YG.webp)

```
可以用来创建白名单进程的系统进程
\\SystemRoot\\System32\\Lsass.exe
\\SystemRoot\\System32\\Svchost.exe
\\SystemRoot\\System32\\Services.exe
\\SystemRoot\\System32\\Csrss.exe
\\SystemRoot\\System32\\Consent.exe
```

## 冲锋前的总结

通过分析，我们得到如下几个点

```
Obj的降权可以通过白名单绕过
白名单进程可以随便创建但不能做其他操作
在晶核的防护中允许了部分系统进程可以对白名单进程操作
```

那么我们的利用链就可以是

```
启动并注入晶核中的白名单（Lsass.exe、Svchost.exe、Services.exe、Csrss.exe、Consent.exe）程序，比如svchost。
通过svchost去启动并注入360安装目录下的exe，如360ShellPro.exe
再通过360ShellPro.exe对ZhuDongFangYu.exe或360try.exe之类的进程进行关闭
最后完成冲锋。
```

## 开冲

根据上面的一顿操作，得到下图的中的一坨

![](upload/attach/202403/749276_94ZRQPSQ9BRC45W.webp)

![](upload/attach/202403/749276_DN6RRRG8EQ9CPQX.webp)

# 动图展示

![](upload/attach/202403/749276_F3QFBR7BBX9PA8U.webp)

## 前言

出去练习了两年半，合同到期闲出了一段时间，重新发个文章活跃保号。先整点铺垫

![](https://bbs.kanxue.com/upload/attach/202403/749276_PW3KNNDDWG3EUCX.webp)

## 背景

2023年底，闲来没事想起了`xjun`师傅2021年发的`procexp`驱动利用帖子时在群里讨论的，通过`procexp`驱动突破`PPL`后注入到`csrss`进程中，再通过`csrss`来结束那些个安全防护软件。于是在当时就有了如下成果：

![](https://bbs.kanxue.com/upload/attach/202403/749276_G8N623J7S4HTH3F.webp)

文章链接（在职的时候发的）

[a4aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8V1q4j5K9W2k6B7k6%4g2t1j5g2N6V1x3%4u0t1M7g2W2c8y4s2N6X3L8i4M7`.](https://bbs.kanxue.com/elink@324K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8V1q4j5K9W2k6B7k6%4g2t1j5g2N6V1x3%4u0t1M7g2W2c8y4s2N6X3L8i4M7`.)

`xjun`师傅的`PPL`文章

[985K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7I4y4e0b7$3x3K6x3$3i4K6u0V1x3g2)9J5k6o6q4Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@0f2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7I4y4e0b7$3x3K6x3$3i4K6u0V1x3g2)9J5k6o6q4Q4x3X3g2Z5N6r3#2D9)

这些弄完之后，觉得通过`hook` `csrss`中的`CsrCreateProcess`来达到主动`kill`指定进程并不优雅。于是对360的`360FsFlt`和`360Hvm64`进行了分析，并从中找到了他的防护缺陷。导致可以用简单的办法结束`ZhuDongFangYu.exe`和`360try.exe`等进程。

## ObjectCallback的降权突破

在`360FsFlt`的回调中，存在一个白名单，而这个白名单为360安装目录下的`exe`，每当运行后都会记录对应的`pid`。

![](https://bbs.kanxue.com/upload/attach/202403/749276_6CXEF4BJMZAUXY4.webp)

![](https://bbs.kanxue.com/upload/attach/202403/749276_V3K4R37VAG6EW2N.webp)

但是，当你想当然的去启动它的`exe`以此获得白名单时，你会发现启动的`exe`不能注入，不能写。所以他仍然还有手段禁止白名单被利用。不要灰心，在这个阶段我们知道了白名单可以不被降权。那么接下来的目的就是想办法获得一个内存修改的权限。

## 内存修改的康庄大道

在上面的启动白名单的`exe`获得的句柄权限是最高权限，但并没有操作的空间。显然是对`api`做了手脚，于是目光转到了晶核上，在`360Hvm64`中发现了这么个表

![](https://bbs.kanxue.com/upload/attach/202403/749276_EACRED8RKTP4ZKF.webp)

![](https://bbs.kanxue.com/upload/attach/202403/749276_ZENKZA38RXRE4RJ.webp)

从下图可以发现，在调用`NtWriteVirtualMemory`会有校验

![](https://bbs.kanxue.com/upload/attach/202403/749276_WTTTHC9JJBQSZPN.webp)

![](https://bbs.kanxue.com/upload/attach/202403/749276_8KYNC36AAN9453X.webp)

到这里可以确定，通过下图的部分`exe`中可以对白名单进行读写注入。

![](https://bbs.kanxue.com/upload/attach/202403/749276_P49ZNKYQ7HEQ7YG.webp)

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `可以用来创建白名单进程的系统进程`  `\\SystemRoot\\System32\\Lsass.exe`  `\\SystemRoot\\System32\\Svchost.exe`  `\\SystemRoot\\System32\\Services.exe`  `\\SystemRoot\\System32\\Csrss.exe`  `\\SystemRoot\\System32\\Consent.exe` |

## 冲锋前的总结

通过分析，我们得到如下几个点

|  |  |
| --- | --- |
| 1  2  3 | `Obj的降权可以通过白名单绕过`  `白名单进程可以随便创建但不能做其他操作`  `在晶核的防护中允许了部分系统进程可以对白名单进程操作` |

那么我们的利用链就可以是

|  |  |
| --- | --- |
| 1  2  3  4 | `启动并注入晶核中的白名单（Lsass.exe、Svchost.exe、Services.exe、Csrss.exe、Consent.exe）程序，比如svchost。`  `通过svchost去启动并注入``360``安装目录下的exe，如``360ShellPro``.exe`  `再通过``360ShellPro``.exe对ZhuDongFangYu.exe或``360try``.exe之类的进程进行关闭`  `最后完成冲锋。` |

## 开冲

根据上面的一顿操作，得到下图的中的一坨

![](https://bbs.kanxue.com/upload/attach/202403/749276_94ZRQPSQ9BRC45W.webp)

![](https://bbs.kanxue.com/upload/attach/202403/749276_DN6RRRG8EQ9CPQX.webp)

# 动图展示

![](https://bbs.kanxue.com/upload/attach/202403/749276_F3QFBR7BBX9PA8U.webp)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)
