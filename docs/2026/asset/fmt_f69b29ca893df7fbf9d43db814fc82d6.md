
# [原创]对XP盾甲ASLR实现机制的逆向分析

注意：由于原文是用markdown写的，发在论坛里格式比较乱而且没图片，可以去我博客查看格式正常的版本，地址：[266K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2$3K9i4u0@1j5h3!0Q4x3X3g2G2M7X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8U0t1I4x3#2)9J5k6h3S2@1L8h3H3`](https://bbs.kanxue.com/elink@25fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2$3K9i4u0@1j5h3!0Q4x3X3g2G2M7X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8U0t1I4x3#2)9J5k6h3S2@1L8h3H3`.)

这篇文章是去年为某公司的面试准备的，虽然最终没有被录用，但是在这过程中收获很多，总结出这篇文章与大家分享。对于逆向分析我纯属于爱好者水平，而且之前主要以Android smali为主，对Win32经验并不多，文章中大部分内容都是现学现卖，最终没有被录用的原因是还有一个重要的地方我没有分析出来。不过以我有限的经验，无法再继续，在此也希望有大神指点一二，不胜感激！！

360出的XP盾甲，为Windows XP实现了Vista以上版本才有的ASLR机制，这个机制会使绝大多数溢出类漏洞失效或难以利用。这也是为什么360这么自信的宣传360盾甲。下面是我对此机制实现过程的逆向分析，看看360用了什么黑科技搞定这件事情的。

## 什么是ASLR？

对溢出类漏洞的攻击，会用到许多系统共享库，在XP及之前的系统，这些共享库的内存地址是固定的。构造溢出代码时，可以很方便的调用这些共享库的API。例如：`LoadLibrary`函数在XP下地址是`0x7C80AEEB`，直接调用即可。当然还有其它的漏洞利用方式，但几乎都必须依靠固定的地址实现。ASLR全称是Address Space Layout Randomization（地址空间布局随机化）。它通过随机化共享库的加载地址，使以上攻击失效或难以利用。

## ASLR的效果

这里我写了一小段程序`ASLRTest`，用来输出各个库的加载地址。输出结果如下：

未使用XP盾甲：

Ntdll的加载地址：7C920000

Kernel32的加载地址：7C800000

`LoadLibrary`函数的地址：7C80AEEB

MSVCR120.dll的加载地址：10000000

system函数的地址：100808C2

本程序customFunc函数的地址：00401000

按任意键继续。

使用XP盾甲后：

Ntdll的加载地址：77040000

Kernel32的加载地址：76CD0000

`LoadLibrary`函数的地址：76CDAEEB

MSVCR120.dll的加载地址：6BEB0000

system函数的地址：6BF308C2

本程序customFunc函数的地址：00401000

按任意键继续。

## 寻找关键点

ASLR的机制涉及到系统底层的东西，估计360会在驱动上做文章，因此首要目标就是360的驱动文件。经过排除，发现只需要保留以下两个驱动，就可以实现ASLR。

`C:\WINDOWS\system32\drivers\hookport.sys`

`C:\WINDOWS\system32\drivers\360rosdrv.sys`

对于ASLR来说，应该涉及到Windows的内存分配功能。我猜测360使用了HOOK技术，来改变Windows默认的内存分配功能。由于`hookport.sys`是最先加载的驱动，名字里也带有hook，因此刚开始我花了大量时间在`hookport.sys`上，后来发现其仅仅实现一些初始化的操作，并没有具体的实现代码。HOOK活动也没有在`hookport.sys`中体现出来（从后期分析看，仅HOOK了一个函数）。

转变思路，使用XueTr工具查看内核钩子，发现了一个重要的线索：

![](https://bbs.kanxue.com/upload/attach/201501/662791_3t8hlaey2tpahvq.png)

360的内核HOOK列表

原来`360rosdrv.sys`才是关键。360使用了Inline Hook，回调函数都指向`360rosdrv.sys`中的代码。在这些被HOOK的函数中，发现`MmMapViewOfSection`函数与内存分配有关系，因此重点对这个函数进行跟踪。

## 验证思路

首先验证一下是不是这个函数。根据XueTr检测结果，360修改了`MmMapViewOfSection`入口处的2个字节（其实不止这些，后面会说到）。内核加载过后，暂停运行。找到内核`MmMapViewOfSection`入口处，添加硬件写入断点，继续运行。一会就会找到做Inline Hook的代码，在`0xBA5C2EA2`处。

注意：这里使用`360rosdrv.sys`的内存映射地址定位，以下相同。其内存映射基址是`0xBA5B6000`。

以下是对做Inline Hook代码的分析：

```
.text:BA5C2E50
.text:BA5C2E50 ; =============== S U B R O U T I N E =======================================
.text:BA5C2E50
.text:BA5C2E50 ; 对MmMapViewOfSection做Inline Hook
.text:BA5C2E50 ; Attributes: bp-based frame
.text:BA5C2E50
.text:BA5C2E50 sub_BA5C2E50 proc near                  ; CODE XREF: sub_BA5B8518+11A
.text:BA5C2E50                                         ; sub_BA5B8518+20C
.text:BA5C2E50
.text:BA5C2E50 MemoryDescriptorList= dword ptr -10h
.text:BA5C2E50 var_C= byte ptr -0Ch
.text:BA5C2E50 var_B= dword ptr -0Bh
.text:BA5C2E50 var_4= dword ptr -4
.text:BA5C2E50 arg_0= dword ptr  8
.text:BA5C2E50 arg_4= dword ptr  0Ch
.text:BA5C2E50 arg_8= dword ptr  10h
.text:BA5C2E50
.text:BA5C2E50 mov     edi, edi
.text:BA5C2E52 push    ebp
.text:BA5C2E53 mov     ebp, esp
.text:BA5C2E55 sub     esp, 10h
.text:BA5C2E58 mov     eax, ___security_cookie
.text:BA5C2E5D xor     eax, ebp
.text:BA5C2E5F mov     [ebp+var_4], eax
.text:BA5C2E62 mov     eax, [ebp+arg_0]                ; MmMapViewOfSection入口地址到eax
.text:BA5C2E65 mov     ecx, [ebp+arg_8]                ; 取得保存Mm新入口地址的变量的地址。这里是0xBA5FC3E0
.text:BA5C2E68 lea     edx, [eax+2]
.text:BA5C2E6B mov     [ecx], edx                      ; 将Inline Hook后的MmMapViewOfSection的入口地址保存在360rosdrv.sys的0xBA5FC3E0处，这应该是个全局变量。由于原始Mm函数入口的2个字节已被修改，这里的新入口是旧入口加2个字节，即DAE2C（原入口地址为DAE2A，相对内核基址偏移，下同），经过后期分析，360还需要调用原始的Mm函数，因此需要这个地址。
.text:BA5C2E6D mov     ecx, [ebp+arg_4]                ; 回调函数地址，这里是0xBA5BA3DA
.text:BA5C2E70 push    ebx
.text:BA5C2E71 mov     ebx, [eax]                      ; 获取DAE2A处的4个字节（8B FF 55 8B）。
.text:BA5C2E73 sub     ecx, eax                        ; 计算回调函数地址与原MmMapViewOfSection地址的差，下面做Inline Hook跳转时使用。
.text:BA5C2E75 mov     [ebp+var_B], ecx
.text:BA5C2E78 lea     ecx, [ebp+MemoryDescriptorList]
.text:BA5C2E7B push    ecx                             ; int
.text:BA5C2E7C push    9                               ; Length
.text:BA5C2E7E add     eax, 0FFFFFFFBh                 ; 计算得到地址[MmMapViewOfSection-5]，对Mm函数的Inline Hook是从其地址之前5个字节开始的。不知为啥，微软在Mm函数之前留了5个字节的空位置，被0xCC填充，正好够一个长jmp跳转。
.text:BA5C2E81 and     ebx, 0FFFFF9EBh
.text:BA5C2E87 push    eax                             ; VirtualAddress
.text:BA5C2E88 mov     [ebp+var_C], 0E9h ;
.text:BA5C2E8C or      ebx, 0F9EBh                     ; 这是DAE2A~DAE2B的目标值
.text:BA5C2E92 call    sub_BA5B7380                    ; 锁住内存
.text:BA5C2E97 test    eax, eax
.text:BA5C2E99 jz      short loc_BA5C2EAB
.text:BA5C2E9B push    esi
.text:BA5C2E9C push    edi
.text:BA5C2E9D mov     edi, eax
.text:BA5C2E9F lea     esi, [ebp+var_C]
.text:BA5C2EA2 movsd                                   ; 执行MmMapViewOfSction Inline Hook的地方，将DAE25~DAE28（相对内核基址）CC CC CC CC修改为E9 B0 F5 F4
.text:BA5C2EA3 movsb                                   ; 将DAE29的0xCC修改为0x39
.text:BA5C2EA3                                         ; 以上两句其实是jmp指令，跳转到360rosdrv的回调函数中。其中的跳转地址在上面代码中已经计算出来。
.text:BA5C2EA4 add     eax, 5                          ; 指针前进5个字节，到DAE2A
.text:BA5C2EA7 xchg    ebx, [eax]                      ; 将DAE2A~DAE2B的8B FF修改为EB F9
.text:BA5C2E9A pop     edi
.text:BA5C2EAA pop     esi
```

以下是Inline Hook前后`MmMapViewOfSection`的代码变化：

修改前：

```
MEMORY:8066AE25 db 0CCh ;
MEMORY:8066AE26 db 0CCh ;
MEMORY:8066AE27 db 0CCh ;
MEMORY:8066AE28 db 0CCh ;
MEMORY:8066AE29 db 0CCh ;
MEMORY:8066AE2A ; ---------------------------------------------------------------------------
MEMORY:8066AE2A
MEMORY:8066AE2A ; __stdcall MmMapViewOfSection(x, x, x, x, x, x, x, x, x, x)
MEMORY:8066AE2A _MmMapViewOfSection@40:                 ; CODE XREF: MEMORY:805CBD62
MEMORY:8066AE2A mov     edi, edi
MEMORY:8066AE2C push    ebp
MEMORY:8066AE2D mov     ebp, esp
MEMORY:8066AE2F sub     esp, 20h
MEMORY:8066AE32 mov     eax, [ebp+20h]
MEMORY:8066AE35 mov     ecx, [eax]
MEMORY:8066AE37 and     dword ptr [ebp-4], 0
```

修改前

修改后：

![](https://bbs.kanxue.com/upload/attach/201501/662791_3oiqe3pp460sytc.png)

```
MEMORY:8066AE25 jmp     sub_BA5BA3DA ; 5个0xCC修改为了一句jmp指令
MEMORY:8066AE2A
MEMORY:8066AE2A ; =============== S U B R O U T I N E =======================================
MEMORY:8066AE2A
MEMORY:8066AE2A ; Attributes: bp-based frame
MEMORY:8066AE2A
MEMORY:8066AE2A ; __stdcall MmMapViewOfSection(x, x, x, x, x, x, x, x, x, x)
MEMORY:8066AE2A _MmMapViewOfSection@40 proc near        ; CODE XREF: MmInitializeProcessAddressSpace(x,x,x,x)+3D2
MEMORY:8066AE2A                                         ; NtSecureConnectPort(x,x,x,x,x,x,x,x,x)+3CE
MEMORY:8066AE2A
MEMORY:8066AE2A var_20= byte ptr -20h
MEMORY:8066AE2A var_8= dword ptr -8
MEMORY:8066AE2A var_4= dword ptr -4
MEMORY:8066AE2A arg_0= dword ptr  8
MEMORY:8066AE2A arg_4= dword ptr  0Ch
MEMORY:8066AE2A arg_8= dword ptr  10h
MEMORY:8066AE2A arg_C= dword ptr  14h
MEMORY:8066AE2A arg_10= dword ptr  18h
MEMORY:8066AE2A arg_14= dword ptr  1Ch
MEMORY:8066AE2A arg_18= dword ptr  20h
MEMORY:8066AE2A arg_1C= dword ptr  24h
MEMORY:8066AE2A arg_20= dword ptr  28h
MEMORY:8066AE2A arg_24= dword ptr  2Ch
MEMORY:8066AE2A
MEMORY:8066AE2A jmp     short loc_8066AE25 ; 修改为了一句短跳转，跳转到上面一句长跳转指令
MEMORY:8066AE2C push    ebp
MEMORY:8066AE2D mov     ebp, esp
MEMORY:8066AE2F sub     esp, 20h
MEMORY:8066AE32 mov     eax, [ebp+arg_18]
MEMORY:8066AE35 mov     ecx, [eax]
MEMORY:8066AE37 and     [ebp+var_4], 0
```

修改后

![](https://bbs.kanxue.com/upload/attach/201501/662791_3oiqe3pp460sytc.png)

修改后的代码为两个跳转，通过这两个跳转最终跳到360rosdrv中的处理代码。如下图：

![](https://bbs.kanxue.com/upload/attach/201501/662791_3t8hlaey2tpahvq.png)

跳转指令

通过上面的分析，我使用的验证思路是，在360的Inline Hook代码处设置断点，待360的代码做完Inline Hook后，将修改后的代码还原，看看ASLR是否还有效。经过验证，代码还原后ASLR机制就失效了。这说明很有可能这里就是实现ASLR的关键地方。
