
# [原创]CVE-2014-1776漏洞分析

```
/************************************************************************
/* 作者:kiwiHacker
/* 博客:[c60K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8&6M7X3q4F1k6r3f1H3x3o6m8Q4x3X3g2U0L8$3#2Q4x3V1j5`.](https://bbs.kanxue.com/elink@627K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8&6M7X3q4F1k6r3f1H3x3o6m8Q4x3X3g2U0L8$3#2Q4x3V1j5`.)
/* 日期:2015-01-25 16:10
/************************************************************************
```

## 0x00 实验环境

操作系统：VMWARE Windows 7 sp1

IE版本：8.0.7601.17514

漏洞DLL：`mshtml.dll`

调试器:`windbg`

## 0x01 简述

`CVE-2014-1776`属于use-after-free漏洞，影响范围不但涉及6/7/8/9，还影响IE10/11浏览器，这几个版本的浏览器被披露出可以远程执行代码的漏洞。漏洞出现在`mshtml!CMarkup::IsConnectedToPrimaryMarkup`函数。

![](https://bbs.kanxue.com/upload/attach/201501/661766_lp9152r044iazm0.png)

## 0x02 说明

为了避免黑产，共同维护互联网安全，漏洞中的关键代码段被打上了马赛克，本篇文章只是分析漏洞的起因及对突破WINDOWS保护机制的研究，希望码农们引以为戒，提高软件代码安全性！

## 0x03 漏洞原因

以下是能够触发漏洞并利用的三个文件：

![](https://bbs.kanxue.com/upload/attach/201501/661766_qqdtn4dxoe9d5dp.png)

`1.html`为入口文件,如图：

![](https://bbs.kanxue.com/upload/attach/201501/661766_dh30cudb23jal1k.png)

微软自从Windows XP SP2以后添加了很多安全保护机制，其中`DEP/ASLR`是我们在这个漏洞中需要突破的2项保护机制，SWF文件就是用来帮助我们突破保护的。

当swf加载时，会先进行Heap Spray，将绕过`DEP/ASLR`的代码和`SHELLCODE`放置在我们稍后可控的堆内：

![](https://bbs.kanxue.com/upload/attach/201501/661766_o6iflrbwfq0lqsq.png)

首先对`Vector.<uint>((0x1000 – 8) / 4)`做个说明，为了使每个`s[i]`的大小以0x1000字节对齐，我们需要分配`(0x1000 – 每个vector的头结构大小8字节) / sizeof(uint)=0x3FE`个`uint`变量，如下图：

![](https://bbs.kanxue.com/upload/attach/201501/661766_zjgg6h2rrzsbcgb.png)

在准备工作结束后，会调用`ExternalInterface.call("trigger");`来执行JS中的引发漏洞的代码：

![](https://bbs.kanxue.com/upload/attach/201501/661766_itfvdwahidrp32x.png)

现在让我们开启IE的页堆调试:

![](https://bbs.kanxue.com/upload/attach/201501/661766_dz1xx1litkxzz7n.png)

找到崩溃点如下：

![](https://bbs.kanxue.com/upload/attach/201501/661766_q744mun10zqlt6m.png)

对应汇编：

![](https://bbs.kanxue.com/upload/attach/201501/661766_cek80d6limnb2bg.png)

可以看到`EAX`的值来自`ESI`，推断`ESI`是一个`CMarkup`对象指针，现在我们深入剖析一下`ESI`的由来，我在`mshtml!CMarkup::IsConnectedToPrimaryMarkup`下断点：`bu mshtml!CMarkup::IsConnectedToPrimaryMarkup "!heap -p -a eax;g"`

重新运行程序：

![](https://bbs.kanxue.com/upload/attach/201501/661766_5o3d7pemq4a2dip.png)

![](https://bbs.kanxue.com/upload/attach/201501/661766_i8j8f1x5erotawp.png)

`!heap -p -a`只是截断了最近几条函数调用堆栈，我们需要更深层次的函数调用信息。

对`mshtml!CMarkup::~CMarkup`下断点，观察函数堆栈，如下图：

![](https://bbs.kanxue.com/upload/attach/201501/661766_93hc9jyxnocnet3.png)

![](https://bbs.kanxue.com/upload/attach/201501/661766_mfeqp5iea383wer.png)

结合下面崩溃时的堆栈信息，可以断定函数在`CStylement::OnPropertyChange`函数尚未返回前，又重新进入`CMarkup::IsConnectedToPrimaryMarkup`函数重复使用，造成了漏洞的产生，如下：

![](https://bbs.kanxue.com/upload/attach/201501/661766_ao6cy3002bstc9v.png)

## 0x04 突破WINDOWS安全保护

为了能够控制UAF对象，我们需要使用和UAF对象同样大小的内存对该段内存占位，以对齐内存，如下图：

![](https://bbs.kanxue.com/upload/attach/201501/661766_7li0tnww1d1fjqx.png)

再次运行IE：

![](https://bbs.kanxue.com/upload/attach/201501/661766_4vb8lkxiv60nobq.png)

这时候已经可以成功控制`EAX`，继续往下步进，看看后面有没有可以进一步与`EAX`或其他寄存器相关的利用代码，如下：

![](https://bbs.kanxue.com/upload/attach/201501/661766_wmiutpxoen3csge.png)

果不其然，居然找到了可以直接控制`EIP`的指令，而`EDX`同时还是我们可控的！

还记得我们最初在SWF中HEAP SPRAY了一大片内存吗，其中包含`SHELLCODE`，而因为`DEP`的存在，则需要通过`ROP`技术绕过这个限制。此刻`EAX`是指向可控堆的，因此我们如果能在内存空间中（需要利用SWF突破`ASLR`）找到类似`xchg eax, esp;ret;`这样的指令就可以控制堆栈，从而调用`zwProtectVirtualMemory`来修改指定的内存属性来绕过`DEP`，翻转堆栈后，堆栈布局如下：

![](https://bbs.kanxue.com/upload/attach/201501/661766_9p13qmxdbfkf51a.png)

![](https://bbs.kanxue.com/upload/attach/201501/661766_btazex37ql0a63z.png)

`Shellcode`部分我只用了`0x90909090`举个例子，来说明`DEP/ASLR`已经被击败了。

![](https://bbs.kanxue.com/upload/attach/201501/661766_hbnft176x2chy71.png)

## 0x05 总结

外国论坛中有些大牛通过在漏洞中修改`VECTOR`对象的`LENGTH`属性，获得对进程内任意地址的读写，通过覆盖一个`SOUND`对象的虚函数表，来触发`DEP`绕过，本文采用了另一种方法来叙述。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [图片1.png](javascript:void(0);)
（29.75kb，13次下载）
- [图片2.png](javascript:void(0);)
（7.02kb，8次下载）
- [图片3.png](javascript:void(0);)
（10.15kb，5次下载）
- [图片4.png](javascript:void(0);)
（38.67kb，8次下载）
- [图片5.png](javascript:void(0);)
（2.76kb，3次下载）
- [图片6.png](javascript:void(0);)
（10.75kb，4次下载）
- [图片7.png](javascript:void(0);)
（2.91kb，4次下载）
- [图片8.png](javascript:void(0);)
（5.12kb，8次下载）
- [图片9.png](javascript:void(0);)
（4.66kb，5次下载）
- [图片10.png](javascript:void(0);)
（20.61kb，6次下载）
- [图片11.png](javascript:void(0);)
（21.14kb，3次下载）
- [图片12.png](javascript:void(0);)
（36.00kb，6次下载）
- [图片13.png](javascript:void(0);)
（10.07kb，6次下载）
- [图片14.png](javascript:void(0);)
（11.99kb，6次下载）
- [图片15.png](javascript:void(0);)
（26.95kb，10次下载）
- [图片16.png](javascript:void(0);)
（5.92kb，5次下载）
- [图片17.png](javascript:void(0);)
（2.62kb，3次下载）
- [图片18.png](javascript:void(0);)
（2.34kb，4次下载）
- [图片19.png](javascript:void(0);)
（18.25kb，5次下载）
- [图片20.png](javascript:void(0);)
（4.47kb，10次下载）
