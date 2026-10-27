
# 年经贴 SSDT与Shadow SSDT的遍历与恢复

## 前言

首发于[debugwar.com](https://bbs.kanxue.com/elink@82eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8W2j5Y4g2Y4N6$3q4J5i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3g2F1N6h3#2W2M7X3q4@1k6g2)9J5k6r3q4F1k6q4)9J5k6s2u0W2M7%4c8G2M7X3g2Q4x3X3c8K6M7$3c8@1i4K6u0V1L8%4u0Q4x3X3c8K6K9r3q4V1L8%4N6K6M7$3c8@1i4K6u0V1N6r3q4T1L8r3f1`.)，授权转载。

本文是[这篇](https://bbs.pediy.com/thread-274979.htm)求助帖子的后续，可能有人会问， 都2202年了， x64占主流的情况下为什么还在研究SSDT相关的东西。原因其实很简单，别人有我也要有 /doge。

不废话，请看正文。

--------------下面是正文，而我是分割线--------------

## SSDT与ShadowSSDT表的结构

两张表使用了同一数据结构，只是名字不同而已，具体结构C语言表示如下：

```
typedef struct _KSYSTEM_SERVICE_TABLE {
PUCHAR ServiceTableBase;
PULONG Count;
ULONG_PTR TableSize;
PUCHAR ArgumentTable;
} KSYSTEM_SERVICE_TABLE, *PKSYSTEM_SERVICE_TABLE;
```

```
typedef struct _KSERVICE_TABLE_DESCRIPTOR
{
KSYSTEM_SERVICE_TABLE   ntoskrnl;
KSYSTEM_SERVICE_TABLE   win32k;
KSYSTEM_SERVICE_TABLE   notUsed1;
KSYSTEM_SERVICE_TABLE   notUsed2;
} KSERVICE_TABLE_DESCRIPTOR, *PKSERVICE_TABLE_DESCRIPTOR;
```

其中`_KSERVICE_TABLE_DESCRIPTOR`对应`nt!KeServiceDescriptorTable`和`nt!KeServiceDescriptorTableShadow`。而`nt!KiServiceTable`一般指向`_KSYSTEM_SERVICE_TABLE`的首地址，即`ServiceTableBase`。

这么说还是太抽象了，来看几个实际的例子。

```
kd> .process /i fffffa80060a7b30
You need to continue execution (press 'g' <enter>) for the context
to be switched. When the debugger breaks in again, you will be in
the new process context.
kd> g
Break instruction exception - code 80000003 (first chance)
nt!RtlpBreakWithStatusInstruction:
fffff800`03cd1490 cc              int     3
kd> !process fffffa80060a7b30 0
PROCESS fffffa80060a7b30
SessionId: 0  Cid: 0138    Peb: 7fffffde000  ParentCid: 0130
DirBase: 9e3fa000  ObjectTable: fffff8a006353010  HandleCount: 408.
Image: csrss.exe
```

```
kd> dps nt!KeServiceDescriptorTable L10
// typedef struct _KSERVICE_TABLE_DESCRIPTOR {
fffff800`03f0a840  fffff800`03cda300 nt!KiServiceTable             //        KSYSTEM_SERVICE_TABLE   ntoskrnl.ServiceTableBase;
fffff800`03f0a848  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   ntoskrnl.Count;
fffff800`03f0a850  00000000`00000191                                     //        KSYSTEM_SERVICE_TABLE   ntoskrnl.TableSize;
fffff800`03f0a858  fffff800`03cdaf8c nt!KiArgumentTable           //        KSYSTEM_SERVICE_TABLE   ntoskrnl.ArgumentTable;
fffff800`03f0a860  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   win32k.ServiceTableBase;
fffff800`03f0a868  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   win32k.Count;
fffff800`03f0a870  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   win32k.TableSize;
fffff800`03f0a878  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   win32k.ArgumentTable;
fffff800`03f0a880  fffff800`03cda300 nt!KiServiceTable             //        KSYSTEM_SERVICE_TABLE   notUsed1.ServiceTableBase;
fffff800`03f0a888  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   notUsed1.Count;
fffff800`03f0a890  00000000`00000191                                     //        KSYSTEM_SERVICE_TABLE   notUsed1.TableSize;
fffff800`03f0a898  fffff800`03cdaf8c nt!KiArgumentTable           //        KSYSTEM_SERVICE_TABLE   notUsed1.ArgumentTable;
fffff800`03f0a8a0  fffff960`00161f00                                           //        KSYSTEM_SERVICE_TABLE   notUsed2.ServiceTableBase;
fffff800`03f0a8a8  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   notUsed2.Count;
fffff800`03f0a8b0  00000000`0000033b                                     //        KSYSTEM_SERVICE_TABLE   notUsed2.TableSize;
fffff800`03f0a8b8  fffff960`00163c1c                                          //        KSYSTEM_SERVICE_TABLE   notUsed2.ArgumentTable;
// } KSERVICE_TABLE_DESCRIPTOR, *PKSERVICE_TABLE_DESCRIPTOR;
```

```
kd> dps nt!KeServiceDescriptorTableShadow L10
// typedef struct _KSERVICE_TABLE_DESCRIPTOR {
fffff800`03f0a880  fffff800`03cda300 nt!KiServiceTable             //        KSYSTEM_SERVICE_TABLE   ntoskrnl.ServiceTableBase;
fffff800`03f0a888  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   ntoskrnl.Count;
fffff800`03f0a890  00000000`00000191                                     //        KSYSTEM_SERVICE_TABLE   ntoskrnl.TableSize;
fffff800`03f0a898  fffff800`03cdaf8c nt!KiArgumentTable           //        KSYSTEM_SERVICE_TABLE   ntoskrnl.ArgumentTable;
fffff800`03f0a8a0  fffff960`00161f00                                           //        KSYSTEM_SERVICE_TABLE   win32k.ServiceTableBase;
fffff800`03f0a8a8  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   win32k.Count;
fffff800`03f0a8b0  00000000`0000033b                                     //        KSYSTEM_SERVICE_TABLE   win32k.TableSize;
fffff800`03f0a8b8  fffff960`00163c1c                                          //        KSYSTEM_SERVICE_TABLE   win32k.ArgumentTable;
fffff800`03f0a8c0  00000000`77c51206                                     //        KSYSTEM_SERVICE_TABLE   notUsed1.ServiceTableBase;
fffff800`03f0a8c8  00000000`00000000                                     //        KSYSTEM_SERVICE_TABLE   notUsed1.Count;
fffff800`03f0a8d0  fffff800`00a06428                                          //        KSYSTEM_SERVICE_TABLE   notUsed1.TableSize;
fffff800`03f0a8d8  fffff800`00a063d8                                          //        KSYSTEM_SERVICE_TABLE   notUsed1.ArgumentTable;
fffff800`03f0a8e0  00000000`00000002                                     //        KSYSTEM_SERVICE_TABLE   notUsed2.ServiceTableBase;
fffff800`03f0a8e8  00000000`00008600                                     //        KSYSTEM_SERVICE_TABLE   notUsed2.Count;
fffff800`03f0a8f0  00000000`000d79f0                                       //        KSYSTEM_SERVICE_TABLE   notUsed2.TableSize;
fffff800`03f0a8f8  00000000`00000000                                      //        KSYSTEM_SERVICE_TABLE   notUsed2.ArgumentTable;
//
```

由于我们要查看`ShadowSSDT`，因此上面先用`.process`切换到了`fffffa80060a7b30`这个进程上下文（有`SessionID`的程序），不然看到的`nt!KeServiceDescriptorTableShadow`可能不是有效数据。

上文中，我们根据表结构，已经可以得到`ServiceTableBase`地址了，那么如何获取这个`Base`下各个函数的实际地址呢？

## 计算表中各个函数地址

如果此时你是`x64`的环境，而且你迫不及待的去看`ServiceTableBase`处内容的话，你会发现和真实的函数地址对不上，产生这个现象的原因本文后面会提到。现在让我们先从最简单的情况开始讨论。

### x86环境

`x86`环境下情况最为简单，你会发现`ServiceTableBase`处的地址就是对应函数的地址，无论是静态看还是动态看：

![](https://bbs.kanxue.com/upload/attach/202211/156241_Q2RUTSFTF73PAPU.jpg)

调试器中观察相同位置的数值及符号如下：

![](https://bbs.kanxue.com/upload/attach/202211/156241_HPSPUAX4CCW5WDH.jpg)

由于基址不同，以`NtAccessCheck`函数为例。静态环境的值为`0x00471b6c`, 加载在基址`0x00400000`处。动态环境的值为`0x820d5b6c`，加载在基址`0x82064000`处。

两个环境的值分别换算成`RVA`，可知：

```
0x00471b6c - 0x00400000 = 0x820d5b6c - 0x08206400 = 0x71b6c
```

虽然静态与动态环境下`KiServiceTable`具体的值不一样（这是由于重定位造成的），但是通过上述`RVA`计算可知，其内存处的值是可以对上的。最终我们可以得出结论，`x86`下 `KiServiceTable`直接存放的就是目标函数地址。

### x64环境

`x64`下微软做了调整，静态环境下，在低版本内核中（`Windows7`、`8`、`8.1`）和`x86`一样直接存放函数地址。而高版本内核（`Windows 10`以上），`KiServiceTable`中将不再直接存放函数地址，而且数据长度也由原来的`4`字节变成了`8`字节。

调整较大的是动态环境，当用调试器进行内核调试时，会发现无论高版本还是低版本，`KiServiceTable`处的值无论如何也和静态文件中的不一样，这主要是因为内核会对`KiServiceTable`处的值做“压缩”，压缩的动作由`nt!KeCompactServiceTable`函数完成的，后文中我们会有讨论。

下面我们分开看一下高低版本各环境的情况。

#### 低版本内核

下图是`Windows 7 x64`下`KiServiceTable`的截图，可见依然是存储的函数地址，此时的加载基址为`0x00000001`40000000`。

![](https://bbs.kanxue.com/upload/attach/202211/156241_UZ3WRAZCYCEQHRK.jpg)

然后通过调试器观察系统跑起来后`KiServiceTable`处的动态值：

![](https://bbs.kanxue.com/upload/attach/202211/156241_4R7HB4H46HYKQZ6.jpg)
