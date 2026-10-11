
# [原创] 理解 VM：一个 CraCkMe 的 VM 简单分析

> 原文：[`www.pediy.com/kssd/pediy10/81642.html`](https://www.pediy.com/kssd/pediy10/81642.html)

**【文章标题】**: 理解 VM 一个 CraCkMe 的 VM 简单分析
**【文章作者】**: eASYVm
**【作者主页】**: `http://blog.sina.com.cn/77muyulong`
**【软件名称】**: `crackme.exe_BY_KuNgBiM`
**【下载地址】**: `http://bbs.pediy.com/showthread.php?threadid=40679`
**【编写语言】**: `asm`
**【使用工具】**: `OD`
**【软件介绍】**: 一个带有微小完整虚拟机引擎 crackme
**【作者声明】**: 仅是感兴趣...

--------------------------------------------------------------------------------

### 详细过程

最近的日子实在是太无聊了...

看不到一点曙光，于是决定学习一下虚拟机运作原理了，搜索了大量的网上文章并拜读之。

深感受益匪浅，故成此文，仅作菜虫学习过程之心得~无他...

各位大侠不要笑我炒冷饭，这确是我第一个分析的 VM。

所以选择这个程序来练习：
1. 是因为 `asm` 编写结构清晰；
2. 二是伪指令并不多。据某大侠说，该虚拟机配备了一个基于堆栈、单操作数的伪指令系统。虽比不上商业保护的 VM 复杂程度，但是也算样样俱全~

学习 VM，首先你要对 VM 有个理解。

VM 就是虚拟机的意思。但是这里的虚拟机并不同于 `VMware` `Workstation` 中的那种虚拟机。在这里的虚拟机只是说一个环境，这个环境可以使你自己的代码脱离计算机 CPU 的寄存器、堆栈等等运行，而其使用的是你自己定义的一套环境。其中包括所要用到的寄存器、堆栈，甚至还有可能是数据区。而这些可以在你的代码段里，可以在你的数据段里，也可以在 CPU 的堆栈里分配空间进行使用...

今天分析的这个程序就是一个典型的堆栈机，它将自己的环境以临时变量的形式分配在了子函数的临时变量空间...

代码:

```
00401060 <> $ 55 push ebp ; VMproc
00401061 . 8BEC mov ebp,esp
00401063 . 81C4 D0FEFFFF add esp,-130 ; 分配临时变量空间储存虚拟机环境配置
```

可以看出，在调用这个虚拟机的时候，子函数分配了 0x130 个字节的空间来存放 VM 的环境，其中包括一些自定义的通用寄存器、自定义标志寄存器，还有一些其他指针。

代码:

```
00401069 . C745 E4 00000000 mov dword ptr ss:[ebp-1C],0 ; \VMedi
00401070 . C745 E8 00000000 mov dword ptr ss:[ebp-18],0 ; |VMecx
00401077 . C745 F1 00000000 mov dword ptr ss:[ebp-F],0 ; |VMesp
0040107E . C645 FD 00 mov byte ptr ss:[ebp-3],0 ; |标志寄存器 S
00401082 . C645 FE 00 mov byte ptr ss:[ebp-2],0 ; |标志寄存器 Z
00401086 . C745 F5 00000000 mov dword ptr ss:[ebp-B],0 ; |VMeax
0040108D . 8D85 D0FEFFFF lea eax,dword ptr ss:[ebp-130] ; |lpVMstack 指针
00401093 . 8945 F1 mov dword ptr ss:[ebp-F],eax ; |虚拟机环境初始化;lpVMstack 入 VMesp
00401096 . 8B45 14 mov eax,dword ptr ss:[ebp+14] ; |
00401099 . 8945 E0 mov dword ptr ss:[ebp-20],eax ; |
0040109C . 8B45 08 mov eax,dword ptr ss:[ebp+8] ; |
0040109F . 8945 D0 mov dword ptr ss:[ebp-30],eax ; |
004010A2 . 8B45 0C mov eax,dword ptr ss:[ebp+C] ; |
004010A5 . 8945 D8 mov dword ptr ss:[ebp-28],eax ; |
004010A8 . C745 DC 00000000 mov dword ptr ss:[ebp-24],0 ; |
004010AF . C745 D4 00000000 mov dword ptr ss:[ebp-2C],0 ; /
```

以上是该虚拟机环境的初始化过程。后面的注释是我自己加的，你也可以不叫这个名字。但是因为大家都比较熟悉现有的计算机机制，为了后面便于分析，所以用了这些记号。我们可以看出来，这些代码把通用寄存器和标志寄存器都置 0，还把一些指针放入了对应的寄存器里。因为编写 VM 的大侠也是在用现在的计算机机制，所以我猜测，这样分析起来会比较轻松~这也告诉我们，假如那天自己要编写 VM，在能够完成我们自己需要的功能前提下，离现有机制越远应该越不容易被分析~

环境准备完毕，下面一步就要开始虚拟机循环了。循环第一步就是取出 VM 的伪指令。

代码:

```
004010B6 . 8B45 10 mov eax,dword ptr ss:[ebp+10] ; \读取伪指令指针
004010B9 . 8945 EC mov dword ptr ss:[ebp-14],eax ; |伪指令指针入 VMeip
004010BC > FF45 EC inc dword ptr ss:[ebp-14] ; |VMeip +1 ;VMentry
004010BF . 8B45 EC mov eax,dword ptr ss:[ebp-14] ; |
004010C2 . 8A00 mov al,byte ptr ds:[eax] ; |读取伪指令 置 al
004010C4 . 8845 F0 mov byte ptr ss:[ebp-10],al ; |传 al 置 VMcode(VM 环境)
004010C7 . B8 00204000 mov eax,crackme.00402000 ; |传 VMJMPbase 置 eax
004010CC . 0FB65D F0 movzx ebx,byte ptr ss:[ebp-10] ; |VMcode 拓展
004010D0 . C1E3 02 shl ebx,2 ; |VMcode*4
004010D3 . 03C3 add eax,ebx ; |VMcode*4+VMJMPbase
004010D5 . FF20 jmp dword ptr ds:[eax] ; /跳转至 VM 函数
```

以上代码完成了这个过程。我们来看一下：
首先读取指向伪指令区域的指针，之后把这个指针放到了 VM 环境的 `VMeip` 里。
在读取这个指针所指的“字节码”（我们也可以叫做伪指令），这个伪指令通过运算变换计算出她所代表的过程偏移。
该偏移加上 VM 过程跳转表基址 `VMJMPbase` 就得到了该 VM 过程的偏移地址。
之后 `JMP` 过去执行………等到执行该过程结束后，再由一条指令跳回。
伪指令指针加 1 后，继续循环。

该指令如下

代码:

```
jmp short crackme.004010BC ; JMPtoVMentry
```

这便是一个 VM 循环过程。VM 过程跳转表里每一个地址都代表了一个过程，即可以完成一条或多条指令。分析这些过程，及对应过程所对应的字节码，才是分析虚拟机的重头戏。等所有过程的功能及对应字节码都成熟的搞清楚后，剩下的伪指令就基本可以还原为功能类似的程序了~

以下试举例分析

代码:

```
指令 VMnop 004010D7

004010D7 .^\\EB E3 jmp short crackme.004010BC ; JMPtoVMentry ;VMnop
```

代码:

```
指令 VMpushImm32 004010D7

004010D9 . 8345 F1 04 add dword ptr ss:[ebp-F],4 ; \\add VMesp,4 ;VMpushImm32
004010DD . 8B45 F1 mov eax,dword ptr ss:[ebp-F] ; |mov eax,VMesp
004010E0 . 8945 E4 mov dword ptr ss:[ebp-1C],eax ; |mov VMedi,eax
004010E3 . FF45 EC inc dword ptr ss:[ebp-14] ; |inc VMeip
004010E6 . 8B45 EC mov eax,dword ptr ss:[ebp-14] ; |mov eax,VMeip
004010E9 . 8A00 mov al,byte ptr ds:[eax] ; |mov al,[eax]
004010EB . 8845 F0 mov byte ptr ss:[ebp-10],al ; |mov VMcode,al
004010EE . 0FB645 F0 movzx eax,byte ptr ss:[ebp-10] ; |movzx eax,VMcode
004010F2 . 8945 F5 mov dword ptr ss:[ebp-B],eax ; |mov VMeax,eax
004010F5 . 8945 E8 mov dword ptr ss:[ebp-18],eax ; |mov VMecx,eax
004010F8 . 8B5D E4 mov ebx,dword ptr ss:[ebp-1C] ; |mov ebx,VMedi
004010FB . 8903 mov dword ptr ds:[ebx],eax ; |mov [ebx],eax
004010FD .^ EB BD jmp short crackme.004010BC ; /JMPtoVMentry
```

代码:

```
指令 VMLoadMem32 004010D7

004010FF . FF45 EC inc dword ptr ss:[ebp-14] ; \\inc VMeip VMLoadMem32
00401102 . 8B45 EC mov eax,dword ptr ss:[ebp-14] ; |mov eax,VMeip
00401105 . 0FB600 movzx eax,byte ptr ds:[eax] ; |movzx eax,byte[eax]
00401108 . 8845 F0 mov byte ptr ss:[ebp-10],al ; |mov VMcode,al
0040110B . 8B45 E0 mov eax,dword ptr ss:[ebp-20] ; |mov eax,VMesi
0040110E . 0FB65D F0 movzx ebx,byte ptr ss:[ebp-10] ; |movzx ebx,byte[VMcode]
00401112 . C1E3 02 shl ebx,2 ; |
00401115 . 03C3 add eax,ebx ; |
00401117 . 8945 E4 mov dword ptr ss:[ebp-1C],eax ; |mov VMedi,eax
0040111A . 8B18 mov ebx,dword ptr ds:[eax] ; |mov ebx,[eax]
0040111C . 895D E8 mov dword ptr ss:[ebp-18],ebx ; |mov VMecx,ebx
0040111F . 8345 F1 04 add dword ptr ss:[ebp-F],4 ; |add VMesp,4
00401123 . 8B45 F1 mov eax,dword ptr ss:[ebp-F] ; |mov eax,VMesp
00401126 . 8945 E4 mov dword ptr ss:[ebp-1C],eax ; |mov VMedi,eax
00401129 . 8B5D E8 mov ebx,dword ptr ss:[ebp-18] ; |mov ebx,VMecx
0040112C . 8918 mov dword ptr ds:[eax],ebx ; |mov [eax],ebx
0040112E . 895D F5 mov dword ptr ss:[ebp-B],ebx ; |mov VMeax,ebx
00401131 .^ EB 89 jmp short crackme.004010BC ; /JMPtoVMentry
```

简单分析了这 3 个过程，我注释的应该还算清楚，所以就不过多解释了...

另外，我们看代码 004013FC 处的 `VMretn 10`

代码:

```
指令 VMretn 10 004013FC

004013FC . C9 leave
004013FD . C2 1000 retn 10
```

这里是返回的过程，也就意味着 VM 的返回。在 VM 过程跳转表里查询

代码:



```
  00402000  004010D7       crackme.004010D7
  00402004  004010D9       crackme.004010D9
  00402008  004010FF       crackme.004010FF
  0040200C  00401133       crackme.00401133
  00402010  00401169       crackme.00401169
  00402014  00401196       crackme.00401196
  00402018  004013FC       crackme.004013FC     ;<<---这里
  0040201C  004012B8       crackme.004012B8
  00402020  004011C6       crackme.004011C6
  00402024  00401203       crackme.00401203
  00402028  00401240       crackme.00401240
  0040202C  00401268       crackme.00401268
  00402030  00401290       crackme.00401290
  00402034  004012CF       crackme.004012CF
  00402038  004012E6       crackme.004012E6
  0040203C  00401309       crackme.00401309
  00402040  0040132E       crackme.0040132E
  00402044  0040135D       crackme.0040135D
  00402048  00401392       crackme.00401392
  0040204C  004013C7       crackme.004013C7
```

该过程基址在 `VMJMPbase+6*0x4` 处 所以其对应字节码为 `0x06` 于是微指令中只要是 `0x06` 的就是退出 `VM` 返回到程序的时候 方便分析每一个被 `VM` 过的函数~  

不多说了 多了我也不会 总之感觉通过此次学习对 `VM` 小有认识 对以后的学习算是一个积淀吧~  

最后附上 `cyclotron` 分析的 `VM` 过程跳转表(原来前人一分析得如此透彻 还好没有过于班门弄斧)  

`cyclotron` 对该 `VM` 的完整分析详见 [`bbs.pediy.com/showthread.php?t=40854&viewgoodnees=1`](http://bbs.pediy.com/showthread.php?t=40854&viewgoodnees=1)  

代码:

```
  00402000   dd offset VNop
  00402004                 dd offset VPushImm32
  00402008                 dd offset VLoadMem32
  0040200C                 dd offset VSaveMem32
  00402010                 dd offset VAdd
  00402014                 dd offset VSub
  00402018                 dd offset VRet10
  0040201C                 dd offset VPushRegEax
  00402020                 dd offset VMul
  00402024                 dd offset VDiv
  00402028                 dd offset VAnd
  0040202C                 dd offset Vor
  00402030                 dd offset VXor
  00402034                 dd offset VPop2RegEax
  00402038                 dd offset VPushByteVArg
  0040203C                 dd offset VSaveRegEax2ByteVRetVal
  00402040                 dd offset VJmp
  00402044                 dd offset VJz
  00402048                 dd offset VJnz
  0040204C                 dd offset VJs
```

笑~到此结束吧 `VM` 我还会继续玩弄的 知道研究清楚为止~  

另:现在 `nooby` 老师的 `NPr` 都已经如此成熟了 以后关于 `VM` 和猥琐引擎 还望诸大侠不吝赐教~  

最后感谢很多人 包括看雪和一蓑烟雨的很多大大 我列举不全 感谢你们的文献资料 没有他们我们不会取得进步  

还要感谢观看此文的人 感谢你们看到这里...  

--------------------------------------------------------------------------------

### 【经验总结】

善于运用 `GG` 等搜索引擎查阅资料 会学习到很多知识...

--------------------------------------------------------------------------------

### 【版权声明】: 本文首发于 `PEDIY(hIMcrACk)` `一蓑烟雨(i++)` 无版权 欢迎转载 欢迎大侠指点~

2009 年 02 月 09 日 1:04:20

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `crackme.rar`  

 |

老 `k` 给的程序似乎被谁改过了？！  

<http://www.hackerboard.de/attachment.php?attachmentid=2025>  

<http://www.hackerboard.de/attachment.php?attachmentid=2029>  

这是 `Lesco` 的 `keygen`  

<http://www.hackerboard.de/attachment.php?attachmentid=2032>  

上面还有他的教程，德语的。  

<http://www.hackerboard.de/attachment.php?attachmentid=2034>  

to `xzchina`:  

`SetLayeredWindowAttributes` 在这里：  

```
00401693   . 68 58204000    PUSH vm.00402058                  ; /FileName = "User32.dll"
00401698   . E8 11020000    CALL <JMP.&kernel32.LoadLibraryA>        ; \LoadLibraryA
0040169D   . 68 63204000    PUSH vm.00402063                  ; /ProcNameOrOrdinal = "SetLayeredWindowAttributes"
004016A2   . 50             PUSH EAX                            ; |hModule
004016A3   . E8 00020000    CALL <JMP.&kernel32.GetProcAddress>      ; \GetProcAddress
004016A8   . A3 82204000    MOV DWORD PTR DS:[402082],EAX
004016AD   . 6A 02          PUSH 2
004016AF   . 68 C8000000    PUSH 0C8
004016B4   . 6A 00          PUSH 0
004016B6   . FF35 54204000  PUSH DWORD PTR DS:[402054]
004016BC   . FF15 82204000  CALL DWORD PTR DS:[402082]
```

`;简单分析`

`;004020A9 处的 vm 子程序-计算 name 的 hash`

```
     lea esi,szName
     mov eax,1
     mov ebx,2
   _loop:
     mul ebx
     mov ebx,eax
     movzx eax,byte ptr[esi]
     inc esi
     mov edx,eax
     xor ecx,ecx
     sub ecx,edx
     js _loop
     add ebx,07fh;hash
```

`;4020cf 处的 vm 子程序-将输入的 key 转换成十进制数值`

```
     ;buff1 dd 0
             ;buff2 dd 1
             lea esi,szKey
     movzx eax,byte ptr[esi]
     inc esi
   _loopkey:
     sub eax,30h
     mov ecx,buff2
     mul ecx
     mov ecx,buff1
     add eax,ecx
     mov buff1,eax
     mov ecx,buff2
     mov eax,0ah   ;radix 10.
     mul ecx
     mov buff2,eax
     movzx eax,byte ptr[esi]
     inc esi
     xor ecx,ecx
     sub ecx,eax
     js _loopkey
     mov eax,buff1
             sub eax,0ebh
```

`;004020F7vm 子程序 - 关键比较`

```
     sub eax,5
     sub ebx,eax
     jz _goodboy
```

很需要耐心![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬") 我也是刚刚接触，希望能引出玉来![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑") ![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")
