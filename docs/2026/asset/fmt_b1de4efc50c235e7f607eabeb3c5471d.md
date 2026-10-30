
# [原创]MS11-011漏洞分析

![](https://bbs.kanxue.com/view/img/face/16.gif)

刚发的写得太潦草了，特地修改一下，如果还是有问题的话，欢迎继续提出～～

早就想分析一下传说中的长老漏洞，不过因为开学的原因，各种事情，一直没有时间，这两天抽时间把漏洞的分析做完了，本来还计划研究一下利用代码和补丁，但是现在不得不放弃了，现把自己的分析过程贴出来，逆向和语文能力都不怎么样，有描述不清之处，请见谅，同时欢迎各位大牛拍砖，但请尽量温柔！

以下内容全部是个人理解，无法保证其正确性，仅供参考！！！

测试环境：Windows 7

测试工具：VMWare + Windbg + IDA

首先下载漏洞的利用程序，在虚拟机中测试一下，发现程序可以正常运行：

```
C:\>whoami

q-test\qever

C:\>poc

C:\>whoami

nt authority\system
```

下面开始寻找漏洞的成因。

为了减少工作量，应该尽可能地确定漏洞产生的准确位置，在这里采用的方法是在`poc`的源代码的shellcode起始位置添加`0xcc`，对应的是`int 3`指令，这样shellcode一运行，就会立刻断下来，然后就可以进行分析了。

修改过程就不废话了，直接运行修改后的程序，当windbg断下来之后，查看调用堆栈，却发现调用堆栈被破坏了

```
kd> k

ChildEBP RetAddr  

WARNING: Frame IP not in any known module. Following frames may be wrong.

00000000 00000000 0x3d0000
```

此时可以通过对当前栈进行分析粗略估计出错位置。当然，还有一种比较好的方法，考虑到程序可以正常执行，故shellcode运行完毕后必定会返回调用位置，那么在shellcode返回位置下断，运行再次断下来之后查看堆栈可以得到

```
kd> k

ChildEBP RetAddr  

WARNING: Frame IP not in any known module. Following frames may be wrong.

92639d14 994bb8d0 0x3d0096

92639d28 83c7f42a win32k!GreEnableEUDC+0x7c

92639d28 772964f4 nt!KiFastCallEntry+0x12a
```

看到上面的信息，再结合`poc`源码中的漏洞触发代码，就能确定问题就是出现在`win32k!GreEnableEUDC`函数里面了，那么就从它开始分析查找漏洞成因吧！

为了减少工作量，这里采用了一种比较笨的方法，多次尝试来准确判定出错原因及位置。过程如下：

在`win32k!GreEnableEUDC`处下断点，运行修改后的程序，然后单步跟踪，直至触发shellcode中`int 3`指令，多次尝试之后可以确定如下调用信息：

```
win32k!GreEnableEUDC+0x77 : call win32k!BuildAndLoadLinkedFontRoutine+0xeb

	win32k!BuildAndLoadLinkedFontRoutine+0x19d : call win32k!bAppendSysDirectory+0x209

		win32k!bAppendSysDirectory+0x334 : ret 8 //出错！
```

即在调用`win32k!bAppendSysDirectory+0x209`函数之后返回时跳入shellcode，同时在返回时查看堆栈可以发现返回地址被修改了，也就是说在该函数运行过程中栈空间被修改，即栈溢出。

结合网上的信息，多次尝试后可以确定整个漏洞的产生流程：

```
nt!KiFastCallEntry+0x128 : call ebx (win32k!GreEnableEUDC)

	win32k!GreEnableEUDC+0x77 : call win32k!BuildAndLoadLinkedFontRoutine+0xeb

		win32k!BuildAndLoadLinkedFontRoutine+0x19d : call win32k!bAppendSysDirectory+0x209

			win32k!bAppendSysDirectory+0x2de : call dword ptr [win32k!_imp__RtlQueryRegistryValues (9972f104)]

				nt!RtlQueryRegistryValues+0x318 : call nt!RtlpCallQueryRegistryRoutine (83e2ab81)

					nt!RtlpCallQueryRegistryRoutine+0x290 : call nt!RtlpQueryRegistryDirect (83e33997)	

						nt!RtlpQueryRegistryDirect+3D : call nt!memcpy (83c797a0)			//溢出

			win32k!bAppendSysDirectory+0x334 : ret 8 //出错！
```

那么只要分析上面所列出函数的行为，就应该能确定漏洞的成因了。

接下来我们挨个来分析以上函数，实际分析过程中是Windbg动态调试和IDA静态分析同时进行的，在此为了书写方便，仅对Windbg代码进行分析。

对于`win32k!GreEnableEUDC`，涉及到一些不知道什么意思的变量，不过并不影响分析过程。

```
win32k!GreEnableEUDC+0x37:

9956b88b 33f6            xor     esi,esi					//esi清零

9956b88d 46              inc     esi						//esi == 1

……

win32k!GreEnableEUDC+0x75:

9956b8c9 56              push    esi							//参数压栈

9956b8ca 56              push    esi							//参数压栈

9956b8cb e8b8faffff      call    win32k!BuildAndLoadLinkedFontRoutine+0xeb (9956b388)
```

在此只简单提一下，调用参数均为1，毕竟漏洞产生的原因并不在此。

这个函数应该是对应`poc`源码中的`EnableEUDC(TRUE);`一句，EUDC就不解释了，因为没找到好的中文资料，附上微软的介绍：

994K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6\*7K9q4)9J5k6r3y4F1i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3#2K6z5e0l9H3y4K6x3%4i4K6u0W2j5i4y4H3P5l9`.`.

根据前面的分析，要关注`win32k!BuildAndLoadLinkedFontRoutine+0xeb`函数，所以这里重点列出该函数的两个参数，均为1。

下面来看`BuildAndLoadLinkedFontRoutine+0xeb`函数，这个函数中也没有什么重点内容，依旧是列出参数，和调用语句。

```
995cb393 be08020000      mov     esi,208h		

995cb398 56              push    esi

995cb399 8d4dfc          lea     ecx,[ebp-4]		

995cb39c e8a6030000      call    win32k!MALLOCOBJ::MALLOCOBJ (995cb747)	//生成MALLOCOBJ对象

……

995cb3aa 8b5dfc          mov     ebx,dword ptr [ebp-4]

……

995cb434 6804010000      push    104h

995cb439 53              push    ebx

995cb43a e8a0050000      call    win32k!bAppendSysDirectory+0x209 (995cb9df)
```

`win32k!bAppendSysDirectory+0x209` 函数的两个参数，依次是`MALLOCOBJ`对象指针和`0x104`。

关于`win32k!MALLOCOBJ`，个人感觉就像一个字符串的类，而在整个漏洞分析过程中，该对象也是扮演了一个字符串的角色，同时在内存起始位置记录了字符串信息。

下面这个`win32k!bAppendSysDirectory+0x209`函数就比较关键了，最终出错也是在这个函数当中，先看代码：

```
win32k!bAppendSysDirectory+0x209:

9955b9df 8bff            mov     edi,edi

9955b9e1 55              push    ebp

9955b9e2 8bec            mov     ebp,esp

9955b9e4 83ec20          sub     esp,20h

9955b9e7 53              push    ebx

9955b9e8 56              push    esi

9955b9e9 57              push    edi

9955b9ea be08020000      mov     esi,208h

9955b9ef 56              push    esi

9955b9f0 8d4df4          lea     ecx,[ebp-0Ch]					//ebp-0Ch为MALLOCOBJ对象指针

9955b9f3 e84ffdffff      call    win32k!MALLOCOBJ::MALLOCOBJ (9955b747)

9955b9f8 56              push    esi

9955b9f9 8d4dfc          lea     ecx,[ebp-4]					//ebp-4为MALLOCOBJ对象指针

9955b9fc e846fdffff      call    win32k!MALLOCOBJ::MALLOCOBJ (9955b747)

9955ba01 8b4df4          mov     ecx,dword ptr [ebp-0Ch]

9955ba04 33f6            xor     esi,esi					//esi == 0

9955ba06 3bce            cmp     ecx,esi

9955ba08 0f84e6000000    je      win32k!bAppendSysDirectory+0x31e (9955baf4)



win32k!bAppendSysDirectory+0x238:

9955ba0e 8b7dfc          mov     edi,dword ptr [ebp-4]

9955ba11 3bfe            cmp     edi,esi

9955ba13 0f84db000000    je      win32k!bAppendSysDirectory+0x31e (9955baf4)



win32k!bAppendSysDirectory+0x243:

9955ba19 33c0            xor     eax,eax

9955ba1b 8975f0          mov     dword ptr [ebp-10h],esi

9955ba1e 8975ec          mov     dword ptr [ebp-14h],esi

9955ba21 668901          mov     word ptr [ecx],ax

9955ba24 668907          mov     word ptr [edi],ax

9955ba27 668945e0        mov     word ptr [ebp-20h],ax

9955ba2b b804010000      mov     eax,104h

9955ba30 50              push    eax

9955ba31 8bd0            mov     edx,eax

9955ba33 57              push    edi

9955ba34 8975e8          mov     dword ptr [ebp-18h],esi

9955ba37 668955e2        mov     word ptr [ebp-1Eh],dx

9955ba3b 894de4          mov     dword ptr [ebp-1Ch],ecx

9955ba3e e8d7feffff      call    win32k!GrePolyPolyline+0xa2 (9955b91a)		//返回"\REGISTRY\USER\S-1-5-18\EUDC\936"

9955ba43 3bc6            cmp     eax,esi

9955ba45 8945f8          mov     dword ptr [ebp-8],eax

9955ba48 7c7c            jl      win32k!bAppendSysDirectory+0x2f0 (9955bac6)
```

上面这段代码，首先生成`MALLOCOBJ`对象，通过调用`win32k!GrePolyPolyline+0xa2`函数，返回`\REGISTRY\USER\S-1-5-18\EUDC\936`，即需要查询的注册表项。

```
win32k!bAppendSysDirectory+0x274:

9955ba4a 8d45e8          lea     eax,[ebp-18h]

9955ba4d 50              push    eax

9955ba4e 8d45ec          lea     eax,[ebp-14h]

9955ba51 50              push    eax

9955ba52 8d45f0          lea     eax,[ebp-10h]

9955ba55 50              push    eax

9955ba56 57              push    edi

9955ba57 e850010000      call    win32k!AutoResource<&ExFreePool>::~AutoResource<&ExFreePool>+0x177 (9955bbac)

9955ba5c 85c0            test    eax,eax

9955ba5e 745f            je      win32k!bAppendSysDirectory+0x2e9 (9955babf)



win32k!bAppendSysDirectory+0x28a:

9955ba60 3975e8          cmp     dword ptr [ebp-18h],esi

9955ba63 745a            je      win32k!bAppendSysDirectory+0x2e9 (9955babf)
```

这段代码是打酱油的～～
