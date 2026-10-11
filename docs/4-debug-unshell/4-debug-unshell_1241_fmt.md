
# 外挂克星脱壳——江湖 OnLine V1.05 中秋版

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-662.htm`](https://www.pediy.com/kssd/pediy07/pediy7-662.htm)

**下载页面**： `http://jianghuol.88joy.com.cn`  
**软件大小**： `884M`  
**软件简介**： 游戏。那是一个被历史杜撰者刻意遗忘的年代，如果有谁刻意去追问个究竟，他会被反问，当生命没有价值，那么真相还有什么意义！所以，因为曾经有过那么的一个时代，武林就失去了它原来历史。

**【作者声明】**： 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

**【调试环境】**： `WinXP`、`OllyDBD`、`Themida`、`PEiD`、`LordPE`、`ImportREC`

---

## 【脱壳过程】

逝者如斯，2005 年中秋居然也过了。

很久没写东西了，有朋友说江湖 `OnLine` 的壳没见过，所以看看，游戏是没时间玩的。

江湖 `OnLine` 的客户端应该是采用乐宇科技的外挂克星保护的。

外挂克星算是比较低调，似乎专门为网游开发。和某个壳应该有关系。

`[LY_WGKX -> www.szleyu.com]`

`signature = 4D 79 46 75 6E 00 62 73`

`ep_only = false`

---

### 一、以壳克壳

外挂克星对 `OllyDBD` 的检测比较繁琐，这部分我就不详细说明了，可以让其注入到记事本中或者用 `SoftICE` 来调试。最简单的解决方法是，用 `Themida` 对原版 `OllyDBG.exe` 加壳，这样就可以避开那些反调试了，算是 `Themida` 也为 `UnPacKer` 做点好事吧。强调一下，`Themida` 有可能会造成系统崩溃，另外试用版有 20 分钟的限制，正好强制休息一下啦。

加壳后的 `JiangHu.exe` 输入表只有一个函数，`JiangHu.dll` 的 `MyFun`，先调用 `JiangHu.dll` 对 `BS_StopApiM.DB` 解密，得到 `BS_StopApiM.dll`，然后把 `BS_StopApiM.dll` 注入到 `Explorer.exe` 进程中，大部分反跟踪检测都在此注入代码中进行。检测黑名单、检测 `OllyDBG` 第一个区段数据、检测特征码等等，看来作者对 `OllyDBD` 研究了不少，不过对 `SoftICE+IceExt` 却不作抵抗，可惜。

`JiangHu.exe` 通过 `Loader` 运行，可以 `BP CreateProcessA` 获得参数，直接调试也行。

```
0012F830     00402984   /CALL 到 CreateProcessA 来自 Launcher.0040297E
0012F834     004584D8   |ModuleFileName = "JiangHu.exe"
0012F838     0012F8B8   |CommandLine = "JiangHu.exe 619290899"
0012F83C     00000000   |pProcessSecurity = NULL
0012F840     00000000   |pThreadSecurity = NULL
0012F844     00000000   |InheritHandles = FALSE
0012F848     00000000   |CreationFlags = 0
0012F84C     00000000   |pEnvironment = NULL
0012F850     00000000   |CurrentDir = NULL
0012F854     0012F874   |pStartupInfo = 0012F874
0012F858     0012F864   \pProcessInfo = 0012F864
```

---

### 二、HeapAlloc 后门检测

在 `OllyDBG` 等三环 Debugger 调试状态下，`LocalAlloc`、`HeapAlloc` 申请内存的初始填充值非 0,某些壳可以通过这个来检测调试器了。

```c
LPVOID HeapAlloc(
  HANDLE hHeap,   // handle to private heap block
  DWORD dwFlags,  // heap allocation control
  SIZE_T dwBytes  // number of bytes to allocate
);
```

`dwFlags`：

```
HEAP_NO_SERIALIZE equ 00000001h     
HEAP_GROWABLE equ 00000002h     
HEAP_GENERATE_EXCEPTIONS equ 00000004h     
HEAP_ZERO_MEMORY equ 00000008h     
HEAP_REALLOC_IN_PLACE_ONLY equ 00000010h     
HEAP_TAIL_CHECKING_ENABLED equ 00000020h     
HEAP_FREE_CHECKING_ENABLED equ 00000040h     
HEAP_DISABLE_COALESCE_ON_FREE equ 00000080h     
HEAP_CREATE_ALIGN_16 equ 00010000h     
HEAP_CREATE_ENABLE_TRACING equ 00020000h     
HEAP_MAXIMUM_TAG equ 0FFFh           
HEAP_PSEUDO_TAG_FLAG equ 8000h           
HEAP_TAG_SHIFT equ 18            
```

`dwFlags＝00000008`，则申请的内存处以 00 填充

---

设置 `OllyDBD` 暂停在系统断点，忽略所有异常选项。使用 `IsDebug` 插件。

```
7C921231    C3           retn
//进入 OllyDbg 后暂停在这
```

`HeapAlloc` 在 `XP` 上输出转向为 `ntdll` 的 `RtlAllocateHeap`

下断：`BP RtlAllocateHeap [ESP]<20000000`

其实这里我们可以先运行，让其异常，就能快速定位到检测处了。

`Shift+F9` 中断约 8 次后看堆栈：

```
0012FEB4    100034C7  返回到 JiangH_1.100034C7 来自 ntdll.RtlAllocateHeap
0012FEB8    003C0000
0012FEBC    00000000
0012FEC0    00019000
0012FEC4    00019000
0012FEC8    1000AEA8  JiangH_1.1000AEA8
```

修改堆栈中 `0012FEBC＝00000008`

取消以前的断点，在 `100034C7` 处下断

```
100034B5    83E6 F0       and esi,FFFFFFF0
100034B8    56           push esi
100034B9    6A 00        push 0
100034BB    FF35 9CC50010 push dword ptr ds:[1000C59C]
100034C1    FF15 6C900010 call dword ptr ds:[1000906C] ; ntdll.RtlAllocateHeap
100034C7    5F           pop edi
//RtlAllocateHeap 后中断这里
100034C8    5E           pop esi
100034C9    C3           retn
```

`Shift+F9`，`EAX=011526D4`，可以看到 `[011526D4]` 处都已经填充 00 了。

取消所有断点，继续 `F7` 走，看看哪里检测

```
10001F37    8B56 0C      mov edx,dword ptr ds:[esi+C]
10001F3A    8D4C24 08    lea ecx,dword ptr ss:[esp+8]
10001F3E    52           push edx
10001F3F    E8 8C050000  call 100024D0
10001F44    8B4E 0C      mov ecx,dword ptr ds:[esi+C]
10001F47    6A 00        push 0
10001F49    6A 01        push 1
10001F4B    51           push ecx
10001F4C    FFD0         call eax        ;011004DB
//进入
10001F4E    8B56 0C      mov edx,dword ptr ds:[esi+C]
10001F51    68 78A00010  push 1000A078          ; ASCII "BS_New_Proc"
10001F56    52           push edx
10001F57    8D4C24 10    lea ecx,dword ptr ss:[esp+10]
10001F5B    E8 80050000  call 100024E0
```

进入 `10001F4C call eax` ，   就能看见 `HeapAlloc` 后门检测了

```
011404DB    55           push ebp
011404DC    8BEC         mov ebp,esp
011404DE    53           push ebx
011404DF    8B5D 08      mov ebx,dword ptr ss:[ebp+8]
011404E2    56           push esi
011404E3    8B75 0C      mov esi,dword ptr ss:[ebp+C]
011404E6    57           push edi
011404E7    8B7D 10      mov edi,dword ptr ss:[ebp+10]
011404EA    85F6         test esi,esi
011404EC    75 09        jnz short 011404F7
011404EE    833D E40F1501 0>cmp dword ptr ds:[1150FE4],0
011404F5    EB 26        jmp short 0114051D
011404F7    83FE 01      cmp esi,1
011404FA    74 05        je short 01140501
011404FC    83FE 02      cmp esi,2
011404FF    75 22        jnz short 01140523
01140501    A1 D4261501  mov eax,dword ptr ds:[11526D4]
//检测[11526D4]是否为 00000000
01140506    85C0         test eax,eax
01140508    74 09        je short 01140513
0114050A    57           push edi
0114050B    56           push esi
0114050C    53           push ebx
0114050D    FFD0         call eax
//不为 00000000,则运行至此处则异常崩溃
```

---

### 三、搞定输入表

外挂克星对输入表的处理不敢恭维，至少作者没有考虑效率，使用代码入壳和大量的重复冗余希冀脱壳者止步。

`BP LoadLibraryA`      `F9` 中断

```
0158FC14    0113EC06   /CALL 到 LoadLibraryA 来自 0113EC00
0158FC18    0158FC5C   \FileName = "KERNEL32.dll"
```

取消断点，返回 `0113EC06` 处分析

```
0113EB68    8B4424 30    mov eax,dword ptr ss:[esp+30]
0113EB6C    C74424 20 00000>mov dword ptr ss:[esp+20],0
0113EB74    85C0         test eax,eax
0113EB76    0F8E 05020000 jle 0113ED81
//输入表处理完毕则跳转，在 0113ED81 处下断  ★
0113EB7C    EB 0B        jmp short 0113EB89
0113EB7E    8B9C24 44020000 mov ebx,dword ptr ss:[esp+244]
0113EB85    8B7C24 34    mov edi,dword ptr ss:[esp+34]
0113EB89    8B85 0C010000 mov eax,dword ptr ss:[ebp+10C]
0113EB8F    8B4C24 20    mov ecx,dword ptr ss:[esp+20]
0113EB93    50           push eax
0113EB94    8D45 0C      lea eax,dword ptr ss:[ebp+C]
0113EB97    50           push eax
0113EB98    57           push edi
0113EB99    56           push esi
0113EB9A    53           push ebx
0113EB9B    51           push ecx
0113EB9C    8BCD         mov ecx,ebp
0113EB9E    E8 BD020000  call 0113EE60
0113EBA3    85C0         test eax,eax
0113EBA5    0F8C EE010000 jl 0113ED99
0113EBAB    56           push esi
0113EBAC    8BCD         mov ecx,ebp
0113EBAE    E8 6D020000  call 0113EE20
0113EBB3    56           push esi
0113EBB4    8BCD         mov ecx,ebp
0113EBB6    894424 2C    mov dword ptr ss:[esp+2C],eax
0113EBBA    E8 71020000  call 0113EE30
0113EBBF    8BD8         mov ebx,eax
0113EBC1    83C3 64      add ebx,64
0113EBC4    53           push ebx
0113EBC5    895C24 40    mov dword ptr ss:[esp+40],ebx
0113EBC9    E8 CB120000  call 0113FE99
0113EBCE    8BF8         mov edi,eax
0113EBD0    83C4 04      add esp,4
0113EBD3    85FF         test edi,edi
0113EBD5    897C24 38    mov dword ptr ss:[esp+38],edi
0113EBD9    0F84 DA010000 je 0113EDB9
0113EBDF    8D5424 40    lea edx,dword ptr ss:[esp+40]
```



## 汇编代码片段

```
0113EBE3    68 00010000     push 100
0113EBE8    52              push edx
0113EBE9    56              push esi
0113EBEA    6A 00           push 0
0113EBEC    8BCD            mov ecx,ebp
0113EBEE    E8 5D040000     call 0113F050
0113EBF3    85C0            test eax,eax
0113EBF5    0F8C F4010000   jl 0113EDEF
0113EBFB    8D4424 40       lea eax,dword ptr ss:[esp+40]
0113EBFF    50              push eax
0113EC00    FF15 94611401   call dword ptr ds:[1146194]       ; kernel32.LoadLibraryA
0113EC06    85C0            test eax,eax
```

在`00401000`段“设置内存写入断点”。`F9`运行，中断后取消断点

```
0113F18C    C606 E9         mov byte ptr ds:[esi],0E9
```

//中断在这里，我们要在这里 `Patch`，否则壳会把输入表搞的乱七八糟
//修改为：`jmp 114607D`  ★    跳到 `Patch` 代码处

```
0113F18F    2BC6            sub eax,esi
0113F191    83C0 FB         add eax,-5
0113F194    8946 01         mov dword ptr ds:[esi+1],eax
0113F197    8B5424 14       mov edx,dword ptr ss:[esp+14]
0113F19B    8B4424 20       mov eax,dword ptr ss:[esp+20]
0113F19F    3D 2D8B0000     cmp eax,8B2D
0113F1A4    893A            mov dword ptr ds:[edx],edi
0113F1A6    0F8F 9C000000  jg 0113F248
0113F1AC    0F84 86000000  je 0113F238
0113F1B2    3D 158B0000     cmp eax,8B15
0113F1B7    7F 68           jg short 0113F221
0113F1B9    74 38           je short 0113F1F3
0113F1BB    3D A1000000     cmp eax,0A1
0113F1C0    74 1B           je short 0113F1DD
0113F1C2    3D 0D8B0000     cmp eax,8B0D
0113F1C7    0F85 D9000000   jnz 0113F2A6
0113F1CD    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0113F1D1    8D46 06         lea eax,dword ptr ds:[esi+6]
0113F1D4    C601 8B         mov byte ptr ds:[ecx],8B
0113F1D7    C641 01 0D      mov byte ptr ds:[ecx+1],0D
0113F1DB    EB 24           jmp short 0113F201
0113F1DD    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0113F1E1    8D46 05         lea eax,dword ptr ds:[esi+5]
0113F1E4    C601 A1         mov byte ptr ds:[ecx],0A1
0113F1E7    8B5424 14       mov edx,dword ptr ss:[esp+14]
0113F1EB    8951 01         mov dword ptr ds:[ecx+1],edx
0113F1EE    83C1 05         add ecx,5
0113F1F1    EB 18           jmp short 0113F20B
0113F1F3    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0113F1F7    8D46 06         lea eax,dword ptr ds:[esi+6]
0113F1FA    C601 8B         mov byte ptr ds:[ecx],8B
0113F1FD    C641 01 15      mov byte ptr ds:[ecx+1],15
0113F201    8B5424 14       mov edx,dword ptr ss:[esp+14]
0113F205    8951 02         mov dword ptr ds:[ecx+2],edx
0113F208    83C1 06         add ecx,6
0113F20B    2BC1            sub eax,ecx
0113F20D    C601 E9         mov byte ptr ds:[ecx],0E9
0113F210    83E8 05         sub eax,5
0113F213    5F              pop edi
0113F214    8941 01         mov dword ptr ds:[ecx+1],eax
0113F217    5E              pop esi
0113F218    33C0            xor eax,eax
0113F21A    5B              pop ebx
0113F21B    83C4 10         add esp,10
0113F21E    C2 0C00         retn 0C
0113F221    3D 1D8B0000     cmp eax,8B1D
0113F226    75 7E           jnz short 0113F2A6
0113F228    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0113F22C    8D46 06         lea eax,dword ptr ds:[esi+6]
0113F22F    C601 8B         mov byte ptr ds:[ecx],8B
0113F232    C641 01 1D      mov byte ptr ds:[ecx+1],1D
0113F236    EB C9           jmp short 0113F201
0113F238    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0113F23C    8D46 06         lea eax,dword ptr ds:[esi+6]
0113F23F    C601 8B         mov byte ptr ds:[ecx],8B
0113F242    C641 01 2D      mov byte ptr ds:[ecx+1],2D
0113F246    EB B9           jmp short 0113F201
0113F248    3D 15FF0000     cmp eax,0FF15
0113F24D    7F 3D           jg short 0113F28C
0113F24F    74 2C           je short 0113F27D
0113F251    2D 358B0000     sub eax,8B35
0113F256    74 15           je short 0113F26D
0113F258    83E8 08         sub eax,8
0113F25B    75 49           jnz short 0113F2A6
0113F25D    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0113F261    8D46 06         lea eax,dword ptr ds:[esi+6]
0113F264    C601 8B         mov byte ptr ds:[ecx],8B
0113F267    C641 01 3D      mov byte ptr ds:[ecx+1],3D
0113F26B    EB 94           jmp short 0113F201
0113F26D    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0113F271    8D46 06         lea eax,dword ptr ds:[esi+6]
0113F274    C601 8B         mov byte ptr ds:[ecx],8B
0113F277    C641 01 35      mov byte ptr ds:[ecx+1],35
0113F27B    EB 84           jmp short 0113F201
0113F27D    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0113F281    8D46 06         lea eax,dword ptr ds:[esi+6]
0113F284    C601 FF         mov byte ptr ds:[ecx],0FF
0113F287    E9 71FFFFFF     jmp 0113F1FD
0113F28C    3D 25FF0000     cmp eax,0FF25
0113F291    75 13           jnz short 0113F2A6
0113F293    8B4C24 10       mov ecx,dword ptr ss:[esp+10]
0113F297    8D46 06         lea eax,dword ptr ds:[esi+6]
0113F29A    C601 FF         mov byte ptr ds:[ecx],0FF
0113F29D    C641 01 25      mov byte ptr ds:[ecx+1],25
0113F2A1    E9 5BFFFFFF     jmp 0113F201
0113F2A6    5F              pop edi
0113F2A7    C743 08 152700> mov dword ptr ds:[ebx+8],2715
0113F2AE    5E              pop esi
0113F2AF    83C8 FF         or eax,FFFFFFFF
0113F2B2    5B              pop ebx
0113F2B3    83C4 10         add esp,10
0113F2B6    C2 0C00         retn 0C
```

看上面的输入表处理流程，应该明白为何说和某壳很像了吧？

外挂克星相当于在比泰软件防盗版战士上增加了诸多反调试。

## Patch 处理，避开输入表加密

现在我们要找两个地方：

1. `Patch`代码存放地址。选择`Patch`代码存放的地址时候要小心，小心异常，我选择`0114607D`处
2. 跳转表存放地址。这里的地址要保证是程序中不被占用的空白处，我选择放在`004E9000`处。

```
0114607D    53              push ebx
0114607E    3E:8B5C24 7C    mov ebx,dword ptr ds:[esp+7C]
```

//`[esp+7C]`处保存的是处理 `DLL` 的基址 ★

```
01146083    3B1D 00B44E00   cmp ebx,dword ptr ds:[4EB400]
```

//提前在`[4EB400]`处写入当前第一次处理 `DLL` 的基址 ★

//这样比较前后 2 次处理的函数是否是同一个 `DLL` 的

```
01146089    74 0D           je short 01146098
0114608B    891D 00B44E00   mov dword ptr ds:[4EB400],ebx
01146091    8305 04B44E00 0> add dword ptr ds:[4EB404],4
```

//提前在`[4EB404]`处写入准备放跳转地址的地址`004E9000` ★

```
01146098    3B3D 08B44E00   cmp edi,dword ptr ds:[4EB408]
```

//第一个函数过后在`[4EB408]`处写入第一个函数的地址，判断函数是否重复

```
0114609E    74 0D           je short 011460AD
011460A0    893D 08B44E00   mov dword ptr ds:[4EB408],edi
011460A6    8305 04B44E00 0> add dword ptr ds:[4EB404],4
011460AD    3E:8B5C24 24    mov ebx,dword ptr ds:[esp+24]
```

//`[esp+24]`是壳放置的命令类型代码！ ★

```
011460B2    81FB A1000000   cmp ebx,0A1
```

//有一个单字节的，所以需要单独提出来

```
011460B8    74 11           je short 011460CB
011460BA    86FB            xchg bl,bh
```

//前后位调换

```
011460BC    891E            mov dword ptr ds:[esi],ebx
```

//命令类型写入

```
011460BE    8B1D 04B44E00   mov ebx,dword ptr ds:[4EB404]
011460C4    893B            mov dword ptr ds:[ebx],edi
```

//正确函数写入 ★

```
011460C6    895E 02         mov dword ptr ds:[esi+2],ebx
```

//跳转地址写入 ★

```
011460C9    EB 0D           jmp short 011460D8
011460CB    891E            mov dword ptr ds:[esi],ebx
011460CD    8B1D 04B44E00   mov ebx,dword ptr ds:[4EB404]
011460D3    893B            mov dword ptr ds:[ebx],edi
011460D5    895E 01         mov dword ptr ds:[esi+1],ebx
011460D8    5B              pop ebx
011460D9    E9 B990FFFF     jmp 0113F197
```

//跳回去继续流程

从 `Ollydbg` 中“二进制复制”如下：

```
53 3E 8B 5C 24 7C 3B 1D 00 B4 4E 00 74 0D 89 1D 00 B4 4E 00 83 05 04 B4 4E 00 04 3B 3D 08 B4 4E
00 74 0D 89 3D 08 B4 4E 00 83 05 04 B4 4E 00 04 3E 8B 5C 24 24 81 FB A1 00 00 00 74 11 86 FB 89
1E 8B 1D 04 B4 4E 00 89 3B 89 5E 02 EB 0D 89 1E 8B 1D 04 B4 4E 00 89 3B 89 5E 01 5B E9 B9 90 FF
FF
```

这样处理后，所有的函数不再重复累赘，可以用 `ImportRec` 顺利获取了。

外挂克星主程序的 `Patch` 处理一样，也可以取得壳对应表后写代码修复。



# 四、飞向光明之颠

`Shift+F9`，中断在 `0113ED81` 处，输入表处理结束。

选择 `EP` 处的几个字节，右键->设置内存访问断点。`F9` 运行，中断下来

```
004EA410  00 15 A4 AE
```

//此处设置内存访问断点

```
0113F590    8B4C24 04       mov ecx,dword ptr ss:[esp+4]   ; JiangHu.00400000

0113F594    8039 4D         cmp byte ptr ds:[ecx],4D

0113F597    74 08           je short 0113F5A1

0113F599    B8 FFFFFFFF     mov eax,-2

0113F59E    C2 0800        retn 8

0113F5A1    8079 01 5A      cmp byte ptr ds:[ecx+1],5A

0113F5A5    74 08           je short 0113F5AF

0113F5A7    B8 FFFFFFFF     mov eax,-2

0113F5AC    C2 0800        retn 8

0113F5AF    8B41 3C         mov eax,dword ptr ds:[ecx+3C]

0113F5B2    03C1           add eax,ecx                     ; JiangHu.00400000

0113F5B4    8038 50        cmp byte ptr ds:[eax],50

0113F5B7    74 08           je short 0113F5C1

0113F5B9    B8 FFFFFFFF     mov eax,-2

0113F5BE    C2 0800        retn 8

0113F5C1    8078 01 45      cmp byte ptr ds:[eax+1],45

0113F5C5    74 08           je short 0113F5CF

0113F5C7    B8 FFFFFFFF     mov eax,-2

0113F5CC    C2 0800        retn 8

//上面几处是检测 PE 标记信息

0113F5CF    8B40 28         mov eax,dword ptr ds:[eax+28]

0113F5D2    03C1           add eax,ecx                     ; JiangHu.00400000

0113F5D4    8038 FF        cmp byte ptr ds:[eax],0FF

//中断在这里，检测 EP 第 1 个字节

0113F5D7    74 1C           je short 0113F5F5

0113F5D9    6A 00           push 0

0113F5DB    68 A0021501     push 11502A0                   ; ASCII "Error"

0113F5E0    68 A0021501     push 11502A0                   ; ASCII "Error"

0113F5E5    6A 00           push 0

0113F5E7    FF15 A0621401   call dword ptr ds:[11462A0]    ; USER32.MessageBoxA

0113F5ED    B8 FDFFFFFF     mov eax,-3

0113F5F2    C2 0800        retn 8

0113F5F5    8078 01 15      cmp byte ptr ds:[eax+1],15

//检测 EP 第 2 个字节

0113F5F9    74 1C           je short 0113F617

0113F5FB    6A 00           push 0

0113F5FD    68 A0021501     push 11502A0                   ; ASCII "Error"

0113F602    68 A0021501     push 11502A0                   ; ASCII "Error"

0113F607    6A 00           push 0

0113F609    FF15 A0621401   call dword ptr ds:[11462A0]    ; USER32.MessageBoxA

0113F60F    B8 FDFFFFFF     mov eax,-3

0113F614    C2 0800        retn 8

0113F617    8B5424 08       mov edx,dword ptr ss:[esp+8]

0113F61B    83C0 06         add eax,6

0113F61E    2BC8           sub ecx,eax

0113F620    C600 E9        mov byte ptr ds:[eax],0E9

0113F623    8D4C11 FB      lea ecx,dword ptr ds:[ecx+edx-5]

0113F627    8948 01        mov dword ptr ds:[eax+1],ecx

//修改 EP 处代码

0113F62A    33C0           xor eax,eax

0113F62C    C2 0800        retn 8
```

我们可以在 `004EA416` 处设置断点，或者使用 `Second` 段内存断点大法去 `OEP`

```
004EA416    E9 432CFFFF     jmp 004DD05E

//飞向光明之颠

004DD05E    55             push ebp

//OEP

004DD05F    8BEC           mov ebp,esp

004DD061    6A FF          push -1

004DD063    68 08964E00    push 4E9608

004DD068    68 48D54D00    push 4DD548

004DD06D    64:A1 00000000 mov eax,dword ptr fs:[0]

004DD073    50             push eax

004DD074    64:8925 0000000> mov dword ptr fs:[0],esp

004DD07B    83EC 58        sub esp,58

004DD07E    53             push ebx

004DD07F    56             push esi

004DD080    57             push edi

004DD081    8965 E8        mov dword ptr ss:[ebp-18],esp

004DD084    FF15 FC904E00  call dword ptr ds:[4E90FC]    ; kernel32.GetVersion
```

可以用 `OllyDump` 插件来 `dump`。如果此时用 `LordPE`，会提示“Couldn't dump module”。这是外挂克星注入 `Explorer.exe` 的代码在作怪，`Ctrl+Alt+Delete` 打开任务管理程器，结束 `Explorer` 进程，再“新建任务”打开 `Explorer.exe`，此时外挂克星注入的代码已经不存在，可以运行 `LordPE` 和 `ImportRec` 来正常修复啦。

`Game Over`

桃李春风一杯酒 江湖夜雨十年灯

```
          ,     \_/          
        /| \_.-~/            \\_      ,        青春都一晌
       ( /~   /               \~-.\_ |\\
       \`\\   \_/                \    ~\ )            忍把浮名
   \_-~~~-.)  )\_\_/;;,.         \\_  //'
  /'\_,\   --~   \ ~~~-  ,;;\_\_\_(  (.-~~~-.        换了脱壳轻狂
 \`~ \_( ,\_..--\ (     ,;''/    ~--   /.\_\`\\
  /~~//'   /' \`~\          ) /--..\_, )\_  \`~
  "  \`~"  "       \`"       /~'\`\\     \`\\\\~~\\    
                "      "   "~'  ""

             UnPacKed By :  fly

              2005-09-24 00:00

4dd466d0e1e0cc123afbe504da2462ca
```
