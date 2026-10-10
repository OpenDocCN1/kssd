
# CVE-2010-2734 提权漏洞分析(附 POC)

> 原文：[`www.pediy.com/kssd/pediy11/122652.html`](https://www.pediy.com/kssd/pediy11/122652.html)

这个漏洞是在分析 Stuxnet 途中发现的，但是应为当时微软没有发布补丁，所以没有第一时间贴出来，上周补丁已出，所以发出来和大家分享一下。

## 1 漏洞描述

这是一个提权漏洞，是没有正确处理 Keyboard Layout 文件导致的

CVE 描述(`CVE-2010-2743`):

[`cve.mitre.org/cgi-bin/cvename...=CVE-2010-2743`](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2010-2743)

微软公告(`MS10-073`) Microsoft Windows Win32K Keyboard Layout (981957)

[`www.microsoft.com/technet/sec...letin/ms10-073`](http://www.microsoft.com/technet/security/bulletin/ms10-073)

我最早看到相关描述是在 Symantec 的 Blog 里:

[`www.symantec.com/connect/blog...ulnerabilities`](http://www.symantec.com/connect/blogs/stuxnet-using-three-additional-zero-day-vulnerabilities)

## 2 keyboard layout 文件

Keyboard Layout 文件在注册表 `HKLM\SYSTEM\CURRENT CONTROL SET\CONTROL\KEYBOARD LAYOUTS` 下可以找到, 它们都是 PE 文件

![](img/b3f60f491098885d1b3cc126e8f5220e.jpg)

`Win32k!ReadLayoutFile` 函数负加载 Keyboard layout 文件，该函数会将 layout 文件中的信息读入内存中, 其中一个步骤是设置 `win32K.sys!gpKbdNlsTbl`, 指向当前的 `NLSTable` (这里的名字是我自己给的，不是官方名字)

``` 
Struct NLSTable{
  
  +0x4  dwNumEntries
  
  +0x8  pNLSData
  
}
```

`pNLSData` 是一个 buffer, 每一个成员大小为 `0x84` 字节，所以总大小为 `dwNumEntries*0x84`，里面的数据都来自 keyboard layout 文件。下面是一个 keyboard layout 文件的示例：这里 `dwNumEntries` 为一，`pNLSData` 的起始为 `0x1C2`

![](img/fd05e4a0d324e505012affc2e3a375a3.jpg)

## 3 漏洞触发

当用户敲击键盘的某个键时，`Win32k.sys` 里的 `xxxKENLSProcs` 会被调用，传入按键的扫描码，然后程序会查找前面的 `NLSTable` 里的每个成员(前面提到过，每个成员大小为 `0x84` 自节), 直到找到某个成员，该成员的第一个字节等于传入的扫描码，然后取该成员的第二个字节作为 index 来调用 `_aNLSVKFProc`(一个函数数组)中的函数:

![](img/8ec11bf5ba6557394ad71e2b70480e88.jpg)

这里的问题在于，`_aNLSVKFProc` 中只有三个有效的函数，因此我们可以构造一个特殊的 keyboard layout 文件，让这个函数 `call` 越界

![](img/bd5ac853091b24c9b94900187383f7d3.jpg)

比如，我们设计 `NLSTable` 里的一个成员，他的头俩个字节是 `00 05`, 然后发送一个扫描码为 `0` 的按键消息，于是 `_aNLSVKFProc[5]` 会被调用，在我的机子上是 `0x60636261`

接下来大家都知道怎么做了![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

keyboard layout.zip
