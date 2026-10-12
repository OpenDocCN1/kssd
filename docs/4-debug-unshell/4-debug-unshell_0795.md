
# Obsidium 外壳学习手记

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-776.htm`](https://www.pediy.com/kssd/pediy07/pediy7-776.htm)

**第 2 篇 Obsidium 1.3.0.4 学习手记**

前面那篇文章，实例程序是没重定位表的，因此 `Obsidium` 对外壳没有重定位。将实例程序 `TraceMe.exe` 重新编译一下，加上重定位表，再用 `Obsidium 1.304` 加壳保护。（由于重新编译过了，因此这个实例与上篇是 2 个不同的程序，请勿参照上篇的而误导）。

加壳时，将一些常用的保护都选上了，这篇教学没有 `SDK` 部分，有关 `SDK` 的处理，请参考 `heXer` 的笔记：`Obsidium1.2.5.0` 主程序脱壳记录点滴

附件:`obsidium1.304` 实例下载

## 一. OD 反跟踪

`Obsidium` 在 `1.304` 加上了许多反跟踪代码，可以用隐藏 `OD` 插件 `HideOD` 躲过这些反跟踪。插件下载：

`http://bbs.pediy.com/showthread.php?s=&threadid=19170`

只需要将 `HideNtDebgBit`, `CheckRemoteDebuggerPresent`, `Process32Next` 三项勾上即可隐藏 `OD`。

除了用插件隐藏外，还必须将 `OD` 类标识改掉，下面的红字：

```
000B5FE0 CFC2 B5F7 CAD4 A1A3 0000 0000 004F 6C6C 7944 4247 ...........OllyDBG
000B5FF4 0000 6F6C 6C79 6462 6700 2E69 6E69 0049 434F 5F41 ..ollydbg..ini.ICO_A
000B6008 4141 4D41 494E 004D 4149 4E4D 454E 5500 4F4C 4C59 AAMAIN.MAINMENU.OLLY
000B601C 4442 4700 CEDE B7A8 B4B4 BDA8 D6F7 D2AA B4B0 BFDA DBG .................
```

这段是 `Obsidium 1.3.0.4` 利用 `CheckRemoteDebuggerPresent` 检测调试器代码，是 `goldenegg` 以前整理的。

> 引用:
>
> 解码时会有长长的一列除零错误。大约有两百多个除 0。
>
> 运行后会启动一个线程，伪代码是这样子的：
>
> ```
> while(1)
> {
>     Sleep(500);
>     if(CheckRemoteDebuggerPresent()) ExitProcess();
>     if(IsDebugPresent()) ExitProcess();
>     DebugBreak(); //异常时中 seh 自动把 eip 指向循环开始。
>     ExitProcess();
> }
> ```
>
> 当然，这些 `API` 都是以加密形式运行的。
>
> 这样运行起来后, `od` 的 `log` 中就可以看到长长的一大列 `int3` 中断在 `ntdll.DebugBreak`,且一直不停的增多下去。

上面说的“`API` 都是以加密形式运行的”是怎么回事呢？

如果用 `PE` 编辑工具查看加壳后的文件，会发现未加壳的文件和加壳后的文件的输入表不一样，加壳后的输入表一般所引入的 `DLL` 和 `API` 函数很少，甚至只有 `Kernel32.dll` 以及 `GetProcAddress` 这个 `API` 函数，有些连这个函数都没有。壳实际上还需要其他的 `API` 函数来完成它的工作，为了隐藏这些 `API`，它一般只在壳的代码中用显式链接方式动态加载这些 `API` 函数。

一般的壳是通过 `LoadLibrary` 加载 `DLL`，`GetProcAddress` 加载函数的：

```c
HMODULE GetModuleHandle(
    LPCTSTR lpModuleName   // DLL 文件名地址
);

FARPROC GetProcAddress(
    HMODULE hModule,    // DLL 模块句柄
    LPCSTR lpProcName   // 函数名
);
```

如果外壳输入表里没有 `LoadLibrary`，可能是通过暴力搜索获得 `Kernel32.dll` 基址的，相关文章请参考看雪论坛精华集。外壳调用函数，一般是通过 `GetProcAddress` 函数实现的，因此对这设断能很快找到感兴趣的东西。但现在的加密壳为了更好了隐藏自己操作，己不调用系统提供的 `GetProcAddress` 函数实现了，而是自己专门写一段代码来实现 `GetProcAddress` 功能。`Obsidium 1.3.0.4` 就是这情况。为了让大家更好的理解，我们先来寻找 `Obsidium` 自己的 `GetProcAddress` 函数。用 `VirtualAlloc` 来做为入点，因为外壳都用这函数分配临时空间放代码，因此用的比较频繁。

记住，不要将断点设在 `VirtualAlloc` 第一字节，因为外壳程序会检测第一字节是否有断点的，我一般喜欢将断点设在函数尾部。按 `Ctrl＋G` 跳到 `VirtualAlloc`：

```asm
7C809A81 k> 8BFF             mov edi,edi
7C809A83     55             push ebp
7C809A84     8BEC           mov ebp,esp
7C809A86     FF75 14        push dword ptr ss:[ebp+14]
7C809A89     FF75 10        push dword ptr ss:[ebp+10]
7C809A8C     FF75 0C        push dword ptr ss:[ebp+C]
7C809A8F     FF75 08        push dword ptr ss:[ebp+8]
7C809A92     6A FF          push -1
7C809A94     E8 09000000    call kernel32.VirtualAllocEx
7C809A99     5D             pop ebp
7C809A9A     C2 1000        retn 10      //在这按 F2 设个断点
```

设置内存访问异常，2 次后会调用 `VirtualAlloc` 函数。返回到如下代码，取消花指令干扰后，得到的干净代码如下：

```asm
00403688                 6A 00            push    0
0040368A                 6A 56            push    56
0040368C                 6A 00            push    0
0040368E                 68 4A0DCE09      push    9CE0D4A
00403693                 FFB6 98000000    push    dword ptr [esi+98]
00403699                 FF56 54          call    [esi+54]
0040369C                 90               nop
0040369D                 90               nop
0040369E                 90               nop
0040369F                 90               nop
004036A0                 8B55 0C          mov     edx, [ebp+C]
004036A3                 90               nop
004036A4                 90               nop
004036A5                 90               nop
004036A6                 90               nop
004036A7                 81C2 00040000    add     edx, 400
004036AD                 90               nop
004036AE                 90               nop
004036AF                 90               nop
004036B0                 90               nop
004036B1                 6A 40            push    40
004036B3                 68 00300000      push    3000
004036B8                 52               push    edx
004036B9                 6A 00            push    0
004036BB                 50               push    eax
004036BC                 FF96 84000000    call    [esi+84] //这里调用 VirtualAlloc，我们从这里 CALL 返回
```

向上看，估计 `00403699` 就是 `Obsidium` 自己实现的 `GetProcAddress` 代码。

重新加载程序，通过 2 个内存异常后，`bp 00403688` 设断，会中断：

```asm
00403688                 6A 00            push    0
0040368A                 6A 56            push    56
0040368C                 6A 00            push    0
0040368E                 68 4A0DCE09      push    9CE0D4A
00403693                 FFB6 98000000    push    dword ptr [esi+98]
00403699                 FF56 54          call    [esi+54]
```

取消断点，按 `F7` 进入 `00403699` 这个 `CALL`：

```asm
0040AC80                 55               push    ebp
0040AC81                 8BEC             mov     ebp, esp
0040AC83                 81EC 18010000   sub     esp, 118
...
0040ADA9                 8B0483           mov     eax, [ebx+eax*4]
0040ADAC                 03C1             add     eax, ecx
0040ADAE                 EB 57            jmp     short 0040AE07
...
0040AE88                 5F               pop     edi
0040AE89                 5E               pop     esi
0040AE8A                 5B               pop     ebx
0040AE8B                 8BE5             mov     esp, ebp
0040AE8D                 5D               pop     ebp
0040AE8E                 C2 1400          retn    14   //函数地址通过 EAX 返回
```

上面这段代码就是 `Obsidium` 自己实现的 `GetProcAddress` 代码，整个外壳程序调用的 `APi` 函数几乎都是通过这个函数得到。

可以将断点设在 `0040AE8E` 这一行，直接运行外壳程序，观察 `EAX` 指向的字符串，这些就是外壳将要调用的 `API` 函数。

## 二.寻找 OEP

设置内存访问异常，加载程序从头跟踪：

第 1 次异常：`00403055 8B00 mov eax, [eax]`

第 11 次异常 `00404D73 8B00 mov eax, [eax]`

为了能少跟踪些代码，我们尽量找到离 `OEP` 最近的异常，再设置整除异常，一次后来到：

（因为两次整除异常后程序界面己出来，因此一次就行了，此时再将整除异常取消）

`00405354 F7F2 div edx //整除除异常`

查看堆栈窗口：

```
0012FF88 0012FFE0
0012FF8C 00405391 //注意这里
```

在命令行设断：`bp 00405391`

按 `Shift＋F9` 跳过异常来到：

```asm
00405391 55            push    ebp
00405392 8BEC          mov     ebp, esp
00405394 90            nop
00405395 90            nop
00405396 90            nop
00405397 90            nop
00405398 8B4D 08       mov     ecx, [ebp+8]
0040539B 90            nop
0040539C 90            nop
0040539D 90            nop
0040539E 90            nop
0040539F 8B01          mov     eax, [ecx]
004053A1 90            nop
004053A2 90            nop
004053A3 90            nop
004053A4 90            nop
004053A5 90            nop
004053A6 90            nop
004053A7 90            nop
004053A8 90            nop
...
00405431 8D940A EFAB0CFF lea    edx, [edx+ecx+FF0CABEF]
00405438 90            nop
00405439 90            nop
0040543A 90            nop
0040543B 90            nop
0040543C 90            nop
0040543D 90            nop
0040543E 8990 B8000000 mov    [eax+B8], edx  //记下 EDX 值，设断跟进
```

`0040543E` 这句中，`EDX` 中就是 `CONTEXT.EIP` 的值，记 `EDX` 的值 `00405563`，对其设断，来到：

`00405563 E8 CB000000 call 00405633 ; 这是一个加密 CALL，按 F7 进去`

```asm
{
...
00405692 8BEC          mov     ebp, esp
00405694 49            dec     ecx
00405695 C061 F9 72    shl     byte ptr [ecx-7], 72
}
```



```
00405699    0224FE          add     ah, [esi+edi*8]
0040569C    C3             retn                    //按 F4,走出这段循环代码
```

走出上面的代码，重新来到 `00405563` 这个地址处，代码已被 `SMC` 处理成一个跳转：

```
00405563   /EB 04          jmp     short 00405569 //所以开始这也可按一下 F4 即可
```

……………… （按 `F8` 单步走，这段有花指令,不要去除，会有自检验）

```
00405612    FFE7          jmp     edi   //来到此处，跟进
```

来到：

```
008FB40D    E8 00000000   call    008FB412
008FB412    EB 01         jmp     short 008FB415
008FB414    2F            das
008FB415    5D            pop     ebp
008FB416    EB 03         jmp     short 008FB41B
```

…………

//1.304 已没有将 `Stole code` 放在这里

```
008FB480  - E9 4A5FAFFF   jmp     009513CF //来到这里，跳到伪 OEP
```

```
009513CF    64:A1 00000000 mov     eax, fs:[0] //伪 OEP
009513D5    50             push    eax
009513D6    64:8925 0000000>mov     fs:[0], esp
009513DD    83EC 58        sub     esp, 58
009513E0    53             push    ebx
009513E1    56             push    esi
009513E2    57             push    edi
009513E3    8965 E8        mov     dword ptr ss:[ebp-18],esp
009513E6    FF15 44503F00  call    dword ptr ds:[955044]
```

到 `OEP` 后，此时的基址已不是加壳前的 `00400000`，`Obsidium` 已将程序代码重定位了，这里的程序基址是 `950000`，不同的系统此值不同，这个值是 `Obsidium` 外壳调用 `VirtualAlloc` 函数分配的一空间。但同一系统，重复运行实例程序，其基址都是相同的。这就给我们跟踪调试带来方便了。

停到 `009513CF` 伪 `OEP` 后，查看堆栈：

```
0012FFB4    0095208C
0012FFB8    009550D8
0012FFBC    FFFFFFFF
```

此时只要根据 `VC6` 文件的头部特征就可还原出被抽取的代码：

```
push    ebp
mov     ebp, esp
push    -1
push    009550D8
push    0095208C
```

接下来分析 `IAT` 的位置，这句"`009513E6     call dword ptr ds:[955044]`"就是调用系统的某个 `API`，在数据窗口下命令：

```
d 955044
```

```
00954FF4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 B6 00  ................................................
00955004  0C 00 B6 00 18 00 B6 00 24 00 B6 00 30 00 B6 00  ..?.?$.?0.?
00955014  3C 00 B6 00 48 00 B6 00 54 00 B6 00 60 00 B6 00  <.?H.?T.?\`.?
00955024  6C 00 B6 00 78 00 B6 00 84 00 B6 00 90 00 B6 00  l.?x.?????
00955034  9C 00 B6 00 A8 00 B6 00 B4 00 B6 00 18 02 B6 00  ?????
00955044  22 02 B6 00 D8 00 B6 00 E4 00 B6 00 2C 02 B6 00  "?????,?
00955054  FC 00 B6 00 08 01 B6 00 14 01 B6 00 20 01 B6 00  ??????
00955064  2C 01 B6 00 38 01 B6 00 44 01 B6 00 50 01 B6 00  ,?8?D?P?
00955074  5C 01 B6 00 68 01 B6 00 74 01 B6 00 80 01 B6 00  \\?h?t?€?
00955084  8C 01 B6 00 98 01 B6 00 A4 01 B6 00 B0 01 B6 00  ????????
00955094  BC 01 B6 00 C8 01 B6 00 D4 01 B6 00 E0 01 B6 00  ????????
009550A4  36 02 B6 00 42 02 B6 00 4E 02 B6 00 5A 02 B6 00  6?B?N?Z?
009550B4  66 02 B6 00 72 02 B6 00 7E 02 B6 00 8A 02 B6 00  f?r?~???
009550C4  96 02 B6 00 A2 02 B6 00 AE 02 B6 00 BA 02 B6 00  ????????
009550D4  C6 02 B6 00  FF FF FF FF 97 14 95 00 AB 14 95 00  ??
```

初步猜测 `IAT` 的范围就在 `955000`～`9550D8` 这个范围。

修复 `OEP` 后，这时可以 `Dump` 取文件，然后重建输入表后即可脱壳完毕，由于是在基址为 `955000` 这个基址 `Dump` 取的，脱壳后的文件基址也得为这个值。因此总感觉不完美，程序原来基址是 `400000` 这个值，我们可以对外壳动点手脚，让其代码不重定位，这样就可得到基址是 `400000` 的文件，下面我们就来跳过外壳的重定位。

### 三.寻找 Stole code

由于上一步已构造好 `OEP` 处的代码了，这一步可以省略，但为了教学完整性，在这将 `Obsidium` 1.304 处理 `Stole code` 的过程列出。

前面说过，到达伪 `OEP` 时，堆栈代码：

```
0012FFB4    0095208C
0012FFB8    009550D8
0012FFBC    FFFFFFFF
```

定位处理 `Stole code` 方法就是根据外壳程序是何时生成这三个堆栈数据而跟踪的。经过跟踪，是在第 10 次异常后开始处理 `Stole code` 代码的。

设置内存访问异常，重新加载程序，10 次内存访问异常：

```
008F6F91    8B02          mov     eax, [edx] //第 10 次异常中断此处
```

查看堆栈数据：

```
0012FE40   0012FE60  指针到下一个 SEH 记录
0012FE44   008F7034  SE 句柄            //异常后将跳到 008F7034 地址处
```

因此，跳到 `008F7034` 地址，按 `F2` 设断，再按 `Shift F9` 运行程序，中断这里：（下面代码花指令已取消）

```
008F7034    55             push    ebp
008F7035    8BEC          mov     ebp, esp
008F7037    90            nop
008F7038    90            nop
008F7039    90            nop
008F703A    90            nop
008F703B    90            nop
008F703C    90            nop
008F703D    E8 00000000   call    008F7042
008F7042    90            nop
008F7043    90            nop
008F7044    90            nop
008F7045    90            nop
008F7046    58            pop     eax
008F7047    90            nop
008F7048    90            nop
008F7049    90            nop
008F704A    90            nop
008F704B    8B55 10       mov     edx, [ebp+10]
008F704E    90            nop
008F704F    90            nop
008F7050    90            nop
008F7051    90            nop
008F7052    90            nop
008F7053    8D40 87       lea     eax, [eax-79]
008F7056    90            nop
008F7057    90            nop
008F7058    90            nop
008F7059    8982 B8000000 mov     [edx+B8], eax //contex.eip 关键点,对 EAX 设断
```

运行到 `008F7059` 一行，对 `EAX` 地址设断，中断后到这里：

```
008F6FC9   /EB 04          jmp     short 008F6FCF
```

…………

```
008F6FE4    5B             pop     ebx
008F6FE5    5E            pop     esi
008F6FE6    8BE5          mov     esp, ebp
008F6FE8    5D            pop     ebp
008F6FE9    C3            retn    //从走这出这个 CALL
```

来到：（以下代码花指令已去除）

```
00404A7B    90            nop
00404A7C    90            nop
00404A7D    90            nop
00404A7E    F747 0C 04000>test    dword ptr [edi+C], 4
00404A85    90            nop
00404A86    90            nop
00404A87    90            nop
00404A88    74 26         je      short 00404AB0
00404A8A    90            nop
00404A8B    90            nop
```

…………

```
00404AE7    8D7C07 14     lea     edi, [edi+eax+14]
00404AEB    90            nop
00404AEC    90            nop
00404AED    90            nop
00404AEE    90            nop
00404AEF    90            nop
00404AF0    ^ E9 75FBFFFF  jmp     0040466A //跟进
```

来到：（以下代码花指令已去除）

```
0040466A    90            nop
0040466B    90            nop
0040466C    90            nop
0040466D    90            nop
0040466E    90            nop
0040466F    90            nop
00404670    90            nop
00404671    90            nop
00404672    90            nop
00404673    90            nop
00404674    90            nop
00404675    90            nop
00404676    8B45 0C       mov     eax, [ebp+C]
00404679    90            nop
0040467A    90            nop
0040467B    90            nop
0040467C    90            nop
0040467D    90            nop
0040467E    90            nop
0040467F    8B80 A02AEE00 mov     eax, [eax+EE2AA0]
```

…………

```
00404A6E    /74 40        je      short 00404AB0
00404A70    |90           nop
00404A71    |90           nop
00404A72    |90           nop
00404A73    |90           nop
00404A74    |90           nop
00404A75    |90           nop
00404A76    |90           nop
00404A77    |90           nop
00404A78    |FF55 FC      call    [ebp-4]                     ; 关键，跟进
```

```
//这段代码不得改动，有自校验，按 F8 跟
008F6F28    E8 1F040000   call    008F734C
008F6F2D    6B99 161A9156 A>imul    ebx, [ecx+56911A16], -5B
008F6F34    4F            dec     edi
……
008F73DA    ^\0F85 AFFFFFF  jnz     008F738A
008F73E0    EB 03         jmp     short 008F73E5
008F73E2    5B            pop     ebx
008F73E3    216F 61      and     [edi+61], ebp
008F73E6    C3            retn   //如果看到这里，按 F4 设断，可来到这里
```

`008F73E6` 返回后，来到：（以下代码花指令已去除）

```
008F6F28    90            nop
008F6F29    90            nop
008F6F2A    90            nop
008F6F2B    90            nop
008F6F2C    90            nop
008F6F2D    E8 00000000   call    008F6F32
008F6F32    90            nop
008F6F33    90            nop
008F6F34    90            nop
008F6F35    90            nop
008F6F36    90            nop
```

………………

```
008F701B    61            popad
008F701C    90            nop
008F701D    90            nop
008F701E    90            nop
```



## 008F701F 处反汇编代码

```
008F701F    9D                   popfd  
008F7020    90                   nop  
008F7021    90                   nop  
008F7022    90                   nop  
008F7023    90                   nop  
008F7024    90                   nop  
008F7025    90                   nop  
008F7026    90                   nop  
008F7027    90                   nop  
008F7028    90                   nop  
008F7029    90                   nop  
008F702A    55                   push    ebp      //stolen code 第一句  
008F702B    90                   nop  
008F702C    90                   nop  
008F702D    90                   nop  
008F702E    90                   nop  
008F702F    90                   nop  
008F7030    90                   nop  
008F7031    8BEC                 mov     ebp, esp  //stolen code 第二 句  
008F7033    90                   nop  
008F7034    90                   nop  
008F7035    90                   nop  
008F7036    90                   nop  
008F7037    6A FF                push    -1         //stolen code 第三句  
008F7039    90                   nop  
008F703A    90                   nop  
008F703B    90                   nop  
008F703C    90                   nop  
008F703D    68 D8503F00          push    9550D8     //stolen code 第四句  
008F7042    90                   nop  
008F7043    90                   nop  
008F7044    90                   nop  
008F7045    68 8C203F00          push    95208C     //stolen code 第五句  
008F704A    90                   nop  
008F704B    90                   nop  
008F704C    90                   nop  
008F704D    90                   nop  
008F704E    90                   nop  
008F704F    9C                   pushfd  
008F7050    60                   pushad  
```

## 四. 跳过基址重定位，修复内存映像文件

重新加载程序，按 `Ctrl+G` 跳到 `VirtualAlloc`：

```
7C809A81 ker>   8BFF               mov edi,edi  
7C809A83        55                 push ebp  
7C809A84        8BEC               mov ebp,esp  
7C809A86        FF75 14            push dword ptr ss:[ebp+14]  
7C809A89        FF75 10            push dword ptr ss:[ebp+10]  
7C809A8C        FF75 0C            push dword ptr ss:[ebp+C]  
7C809A8F        FF75 08            push dword ptr ss:[ebp+8]  
7C809A92        6A FF              push -1  
7C809A94        E8 09000000        call kernel32.VirtualAllocEx  
7C809A99        5D                 pop ebp  
7C809A9A        C2 1000            retn 10  //这里按 F2 设断  
```

同一系统多次运行实例程序，外壳申请的映像空间的地址是一样的，笔者机子当时基址是：`9500000`，设好 `VirtualAlloc` 断点后，

运行程序，当第三次调用 `VirtualAlloc` 函数时，返回的值是 `9500000`。

```
00404EEF    FF93 84000000   call    [ebx+84]         //VirtualAlloc  
00404EF5    90              nop                     返回这里，eax=950000  
00404EF6    90              nop  
00404EF7    90              nop  
00404EF8    90              nop  
00404EF9    90              nop  
00404EFA    90              nop  
00404EFB    85C0            test    eax, eax          
00404EFD    90              nop  
00404EFE    90              nop  
00404EFF    90              nop  
00404F00    90              nop  
00404F01    90              nop  
00404F02    90              nop  
00404F03    0F84 B2030000   je      004052BB  
```

下面开始解码过程。

解码后这里处理 `reloc`：

```
00405234    8B43 10         mov     eax, [ebx+10]  ; 新 imagebase  
00405237    90              nop  
00405238    90              nop  
00405239    90              nop  
0040523A    90              nop  
0040523B    90              nop  
0040523C    2B43 40         sub     eax, [ebx+40]  ; 求与原来的基址差值，以重定位相关代码要，改成  xor eax,eax      
0040523F    90              nop  
00405240    90              nop  
00405241    90              nop  
00405242    90              nop  
00405243    FF77 08         push    dword ptr [edi+8]  
00405246    FF77 04         push    dword ptr [edi+4]  
00405249    FF73 7C         push    dword ptr [ebx+7C]  
0040524C    50              push    eax  
0040524D    FF73 10         push    dword ptr [ebx+10]  
00405250    FF53 74         call    [ebx+74]  
```

为了脱壳后使 `base` 定位在 `400000h`, 将 `0040523C` 这句改成：

```
0040523C      33C0           xor eax,eax  
```

接下来就到 `OEP` 处，修复 `Stole code` 再 `Dump` 取程序。

此时取消 `VirtualAlloc` 断点，再经过 7 次内存访问异常，再单步跟踪就可到伪 `OEP` 了：

```
009513CF      64:A1 00000000     mov eax,dword ptr fs:[0]  
009513D5      50                 push eax  
009513D6      64:8925 00000000   mov dword ptr fs:[0],esp  
009513DD      83EC 58            sub esp,58  
009513E0      53                 push ebx  
009513E1      56                 push esi  
009513E2      57                 push edi  
009513E3      8965 E8            mov dword ptr ss:[ebp-18],esp  
009513E6      FF15 44504000      call dword ptr ds:[405044]  
```

此时查看上面的红字`[405044]`，已不是开始的`[955044]`了，外壳没有再重定位相关代码与数据。

在修复键入 `OEP` 代码前，先按 `Alt+M` 打开内存映射窗口，找到基址这个地址（我当前的基址 `950000`），同时记下映像的大小 `0xB000`，点击右键/设置权限/完整权限，这样操作后，就可在 `OD` 里键入代码了。

运行 `LordPE`，选中实例进程，先用 `Dump Full` 功能抓取映像，存为 `dumpedfull.exe`。别忘设置 `LordPE` 的 `Options/Full dump:paste header from disk` 这项，从磁盘文件里取 `PE` 的各项信息。不然 `dumped.exe` 的程序一些 `PE` 头数据是错误的，例如没图标等。

接下来再抓取 `950000` 这个基址，大小是 `0xB000` 的映像，这个映像就是程序的代码段、数据段。同样用 `LordPE`，在右键选 `Dump partial`，将你程序 `OEP` 处的基址填进，大小是 `0xB000`。保存为 `dumppartial.dmp` 文件。

下面就利用 `dumpedfull.exe` 和 `dumppartial.dmp` 构造一个完整的 `dumped.exe`。

### 1). 修正 PE 头

用 `LordPE` 打开 `dumpedfull.exe`，查看区块：

```
═Number  Name   VirtSize   RVA    PhysSize  Offset    Flag═  
    1      00001000 00001000 00001000 00001000 C0000040  
    2 .rsrc   00001000 00002000 00001000 00002000 C0000040  
    3      0000C000 00003000 0000C000 00003000 E0000060  
```

我们只需要 `PE` 头这部分的数据，就是 `0x1000` 以前的数据，用 `Hex Workshop` 十六进制工具打开 `dumpedfull.exe`，将光标定在文件头，菜单执行 `Edit/Select Block`，输入 `1000`，选中 `PE` 头部分，点击复制。

`![](img/b2cf2df9c0768153bdd90e9a8b68eed6.jpg)` 

再用 `Hex Workshop` 接着打开 `dumppartial.dmp` 文件，这个文件前 `0x1000` 字节已清空，我们要做的就是将 `dumpedfull.exe` 文件头部分复制过来。

同样的操作，将光标定在文件头，菜单执行 `Edit/Select Block`，输入 `1000`，选中 `PE` 头部分，点击粘贴，将刚才复制的头部数据粘贴过来。

将 `dumppartial.dmp` 改名为 `dumped.exe`。

用 `LordPE` 打开修正过的 `dumped.exe`，查看区块，点击右键 `wipe section header`，将第 2、3 项删除。

同时将第 1 项修正，大小改为 `0xb000`, 属性选上“执行代码”，`E0000040`。

```
═Number  Name   VirtSize   RVA    PhysSize  Offset    Flag═  
    1 .text  0000B000 00001000 0000B000 00001000 E0000040  
```

### 2). 资源修复

用 `dREAMtHEATER` 的 `DT_FixRes` 工具打开 `dumpedfull.exe`，点击 `Dump` 标签，`NewRVA` 填上 `C000`, `FileAlignment` 填上 `1000`, 点击 `Dump Resource`，将资源文件提取出来 `rsrc.bin`。

再用 `LordPE` 打开 `dumped.exe`，查看区块，右键执行 `Load section from disk`，将资源 `rsrc.bin` 导入，同时在数据目录表里修正资源项的 `RVA` 为 `C000`。

```
═Number  Name   VirtSize   RVA    PhysSize  Offset    Flag═  
     1 .text  0000B000 00001000 0000B000 00001000 E0000040  
     2 rsrc.bin 00002000 0000C000 00002000 0000C000 E00000E0  
```

将修复好的 `dumped.exe` 先放一边，等获得正确的输入表再修复就可运行了。

## 五. 输入表的修复

重新加载程序，仅设置内存访问异常。当外壳程序 `VirtualAlloc` 调用为映像申请内存后，就可以用命令：

`D 955000` 查看 `IAT` 的数据了。

注意：不同机子，`VirtualAlloc` 申请的值会不同，根据前面的分析得知，基址＋`5000` 就是 `IAT` 的起始地址。

当第 11 次异常后，`D 955000` 查看的数据已有值，因此重新来过，当第 10 次异常后，停下：

```
008F6F29    8B02            mov     eax, [edx]  
```

中断后数据窗口：

```
00955000  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00955010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
```

在 `00955000` 选中几个字节，点击鼠标右键，设置“断点/内在写入”

再按 `Shift+F9` 通过异常继续执行程序（请不要按 `F9`, 否则以后程序可能会出错的），停在如下：

```
008F76E2    893E            mov     [esi], edi  
```

别忘取消内存断点。为了让大家看的明白，用花指令去除了垃圾代码，并且将多余的 `NOP` 也省略，剩下的代码为：

```
008F76E2    893E            mov     [esi], edi //不断这里，将 IAT 里填充数据  
008F76E7    83C7 0C         add     edi, 0C  
008F76EE    83C6 04         add     esi, 4  
008F76F7    41              inc     ecx  
008F76FD    3B4D 0C         cmp     ecx, [ebp+C]  
008F7704    0F82 68FFFFFF   jb      008F7672  
008F7712    833E 00         cmp     dword ptr [esi], 0  
008F771B    0F85 7D000000   jnz     008F779E  
008F7726    C607 60         mov     byte ptr [edi], 60  
008F772D    66:895F 01      mov     [edi+1], bx  
008F7734    8BC1            mov     eax, ecx  
008F773C    3345 FC         xor     eax, [ebp-4]  
008F7743    66:8947 03      mov     [edi+3], ax  
008F774A    C1CB 10         ror     ebx, 10  
008F7753    8857 06         mov     [edi+6], dl  
008F775B    885F 05         mov     [edi+5], bl  
```



# 汇编代码分析与修复

## 原始代码段

```
008F7762    C647 07 E9      mov     byte ptr [edi+7], 0E9  
008F776A    C1CB 10         ror     ebx, 10  
008F7772    8B45 14         mov     eax, [ebp+14]  
008F777A    2BC7            sub     eax, edi  
008F7781    83E8 0C         sub     eax, 0C  
008F7787    8947 08         mov     [edi+8], eax  
008F7790    893E            mov     [esi], edi  
008F7798    83C7 0C         add     edi, 0C  
008F77A2    8BC7            mov     eax, edi  
008F77A8    2B45 18         sub     eax, [ebp+18]  
008F77AF    5F              pop     edi  
008F77B0    5E              pop     esi  
008F77B1    5B              pop     ebx  
008F77B2    8BE5            mov     esp, ebp // 下面这几句没有花指令，中断可以向下翻，直接 F4 走出这段代码  
008F77B4    5D              pop     ebp  
008F77B5    C2 1400         retn    14  
```

## 清理后的代码段（花指令已去除，代码重新排版整理）

```
008F738D    FF73 34         push    dword ptr [ebx+34]  
008F7390    50              push    eax  
008F7391    FF76 0C         push    dword ptr [esi+C]  
008F7394    FF75 F8         push    dword ptr [ebp-8]  
008F7397    E8 09040000     call    008F77A5 // 我们从这个 CALL 出来  
008F739F    0145 EC         add     [ebp-14], eax  
008F73AF    8B46 10         mov     eax, [esi+10]  
008F73B5    8B56 14         mov     edx, [esi+14]  
008F73BE    0343 10         add     eax, [ebx+10]  
008F73C4    0353 48         add     edx, [ebx+48]  
008F73CB    FF36            push    dword ptr [esi]  
008F73CD    53              push    ebx  
008F73CE    52              push    edx  
008F73CF    50              push    eax  
008F73D0    FF76 0C         push    dword ptr [esi+C]  
008F73D3    E8 E4010000     call    008F75BC // 普通加密函数  
008F73DE    85C0            test    eax, eax  
008F73E3    0F84 18010000   je      008F7501  
008F73F2    837D F0 00      cmp     dword ptr [ebp-10], 0  
008F73F9    74 43           je      short 008F743E // 如跳，则这个 DLL 里没有特殊函数  
008F7405    8B46 10         mov     eax, [esi+10]  
008F740D    8B56 14         mov     edx, [esi+14]  
008F7414    0343 10         add     eax, [ebx+10]  
008F741D    0353 48         add     edx, [ebx+48]  
008F7426    FF75 EC         push    dword ptr [ebp-14]  
008F7429    53              push    ebx  
008F742A    52              push    edx  
008F742B    50              push    eax  
008F742C    FF76 0C         push    dword ptr [esi+C]  
008F742F    E8 A5020000     call    008F76D9 // 特殊加密函数  
008F7438    0145 EC         add     [ebp-14], eax  
008F7445    33C0            xor     eax, eax  
008F744D    8946 0C         mov     [esi+C], eax  
008F7455    8946 10         mov     [esi+10], eax  
008F745B    83C6 18         add     esi, 18  
008F7461    FF45 F8         inc     dword ptr [ebp-8]  
008F7467    FF4D FC         dec     dword ptr [ebp-4]  
008F746F    ^ 0F85 05FDFFFF jnz     008F717A  
008F747B    33C0            xor     eax, eax  
008F7481    5F              pop     edi  
008F7482    5E              pop     esi  
008F7483    5B              pop     ebx  
008F7484    8BE5            mov     esp, ebp  
008F7486    5D              pop     ebp  
008F7487    C3              retn  
```

## 1. 处理普通加密函数

按 `F7` 进入 `008F73D3 call 008F75BC`，这段代码就是外壳为了防止某些函数指针加密出错，这里将加密的指针还原的。

按照以前版本的通用修改方法如下：（花指令己除及中间的 `NOP` 指令己忽略）

```
008F75BC    55              push    ebp  
008F75BD    8BEC            mov     ebp, esp  
008F75BF    53              push    ebx  
008F75C0    56              push    esi  
008F75C1    57              push    edi  
008F75C7    8B75 10         mov     esi, [ebp+10]  
008F75D0    8B7D 0C         mov     edi, [ebp+C]  
008F75D7    8B5D 14         mov     ebx, [ebp+14]  
008F75E4            test    word ptr [esi], 20 // test [esi],8  
008F75EC            je      008F7685           // jnz 008F7685  
008F75F8    66:F706 0200   test    word ptr [esi], 2  
008F7602    75 47           jnz     short 008F764B  
008F760E    33C0            xor     eax, eax  
008F7613    66:C706 0400   mov     word ptr [esi], 4  
008F761E    6A 01           push    1  
008F7620    50              push    eax  
008F7621    FF76 04         push    dword ptr [esi+4]  
008F7624    50              push    eax  
008F7625    FF75 18         push    dword ptr [ebp+18]  
008F7628    FF53 54         call    [ebx+54]  
008F762F    85C0            test    eax, eax  
008F7635    0F84 86000000   je      008F76C1           // je 008F7685  
008F7641    8907            mov     [edi], eax  
008F7646    EB 3D           jmp     short 008F7685  
008F7650    66:C706 0400   mov     word ptr [esi], 4  
008F7659    0FB756 02       movzx   edx, word ptr [esi+2]  
008F7661    6A 01           push    1  
008F7663    52              push    edx  
008F7664    6A 00           push    0  
008F7666    FF76 04         push    dword ptr [esi+4]  
008F7669    FF75 18         push    dword ptr [ebp+18]  
008F766C    FF53 54         call    [ebx+54]  
008F7674    85C0            test    eax, eax  
008F7679    74 0A           je      short 008F76C1       // je 008F7685  
008F767E    8907            mov     [edi], eax  
008F7688    83C6 08         add     esi, 8  
008F768F    83C7 04         add     edi, 4  
008F7698    FF4D 08         dec     dword ptr [ebp+8]  
008F769E    ^ 0F85 3CFFFFFF jnz     008F75E0  
```

这种方法在以前版本有效，但在这个版本就失效了，生成的 `IAT` 中有部分函数出错：(在数据窗口，右键/长型/地址显示：)

```
00405000 >7C80C729  kernel32.lstrcpyA  
00405004 >7C81EE79  kernel32.lstrcmpA  
00405008 >7C838CB9  kernel32.GetStringTypeA  
0040500C >7C80CEC4  kernel32.LCMapStringW  
00405010 >7C832E2B  kernel32.LCMapStringA  
00405014 >7C809CAD  kernel32.MultiByteToWideChar  
00405018 >00B60048  <====这里放上的是错误函数  
0040501C >00B60054  <====这里放上的是错误函数  
00405020 >7C9379FD  ntdll.RtlReAllocateHeap  
00405024 >7C809A81  kernel32.VirtualAlloc  
00405028 >7C9305D4  ntdll.RtlAllocateHeap  
0040502C >7C81E82A  kernel32.GetOEMCP  
00405030 >7C809943  kernel32.GetACP  
00405034 >7C812BE6  kernel32.GetCPInfo  
00405038 >7C80B529  kernel32.GetModuleHandleA  
0040503C >7C801EEE  kernel32.GetStartupInfoA  
00955040  00B6021B  <====特殊函数，上面的 008F729F call 008F7549 这个函数处理  
00955044  00B60225  <====特殊函数  
00405048 >00B600D8  <====这里放上的是错误函数  
0040504C >7C801E16  kernel32.TerminateProcess  
00955050  00B6022F  <====特殊函数  
00405054 >7C862B8A  kernel32.UnhandledExceptionFilter  
00405058 >7C80B357  kernel32.GetModuleFileNameA  
0040505C >00B60114  <====这里放上的是错误函数  
00405060 >00B60120  <====这里放上的是错误函数  
00405064 >7C80A0C7  kernel32.WideCharToMultiByte  
00405068 >00B60138  <====这里放上的是错误函数  
0040506C >00B60144  <====这里放上的是错误函数  
00405070 >7C80C6CF  kernel32.SetHandleCount  
00405074 >7C812CA9  kernel32.GetStdHandle  
00405078 >7C811069  kernel32.GetFileType  
0040507C >00B60174  
00405080 >7C812851  kernel32.GetVersionExA  
00405084 >7C811110  kernel32.HeapDestroy  
00405088 >7C812929  kernel32.HeapCreate  
0040508C >7C809B14  kernel32.VirtualFree  
00405090 >7C93043D  ntdll.RtlFreeHeap  
00405094 >7C957A40  ntdll.RtlUnwind  
00405098 >00B601C8  <====这里放上的是错误函数 s  
0040509C >7C80A480  kernel32.GetStringTypeW  
```

经过分析，问题出在这：

```
008F75F8    66:F706 0200   test    word ptr [esi], 2  
008F7602    75 47           jnz     short 008F764B  
```

查看 `ESI` 指向的数据如下：

```
008F0984  02 00 6C 00 10 E4 37 3D 02 00 6C 00 16 68 FD 1F  l栖>  
008F0994  02 00 47 00 FA FD D2 A4 02 00 4C 00 2A 39 97 BA  G??L??  
```



008F09A4 02 00 4C 00 89 69 EB 25 02 00 4D 00 D6 68 87 83 L 榉?M 棖>

008F09B4 80 00 00 00 04 00 00 00 80 00 00 00 0A 00 00 00 ?..?...

008F09C4 02 00 48 00 25 3C 50 91 02 00 56 00 28 A7 07 B5 H?酐 V??

008F09D4 02 00 48 00 5A 56 8D 4C 02 00 47 00 4B AB 0D 19 H 噚?G??

008F09E4 02 00 47 00 6D 2F CD E2 02 00 47 00 BA 5B 49 4E G?G 宺>

008F09F4 02 00 47 00 ED 52 B7 AF 02 00 47 00 8C 68 8D A0 G 勭?G 梌>

008F0A04 08 00 00 00 03 00 00 00 08 00 00 00 00 00 00 00 .....

008F0A14 40 00 00 00 03 00 00 00 02 00 54 00 11 04 59 C8 @..TБ?

`02`：普通函数标志，用 `008F73D3 call 008F75BC` 函数解密

`08`：特殊函数标志，用 `008F742F call 008F76D9` 函数恢复

`80`：特殊函数标志

对于 `80` 这个标志，用“`008F7498 call [ebx+54]`”这个函数也处理，但得到的结果是错误，猜测作者在 `Obsidium1.3` 里新增了一种处理函数方法。

在这，我们还是用以前的方法，将几处代码修改好：

```
008F75E4             test [esi],8
008F75EC             jnz 008F7685
008F75F8 66:F706 0200 test word ptr [esi], 2
008F7602 /75 47      jnz short 008F764B
008F760E 33C0        xor eax, eax //设断，让程序执行，查看数据面板窗口
008F7613 66:C706 0400 mov word ptr [esi], 4
008F761E 6A 01       push 1
008F7620 50         push eax
008F7621 FF76 04    push dword ptr [esi+4]
008F7624 50         push eax
008F7625 FF75 18    push dword ptr [ebp+18]
008F7628 FF53 54    call [ebx+54]
008F762F 85C0       test eax, eax
008F7635           je 008F7685
008F7641 8907       mov [edi], eax
008F7646 EB 3D      jmp short 008F7685
008F7650 66:C706 0400 mov word ptr [esi], 4
008F7659 0FB756 02  movzx edx, word ptr [esi+2]
008F7661 6A 01      push 1
008F7663 52         push edx
008F7664 6A 00      push 0
008F7666 FF76 04    push dword ptr [esi+4]
008F7669 FF75 18    push dword ptr [ebp+18]
008F766C FF53 54    call [ebx+54]
008F7674 85C0       test eax, eax
008F7679 74 0A      je 008F7685
```

`008F760E` 中断后，查看数据面板窗口：

```
00955000 7C80C729 kernel32.lstrcpyA
00955004 7C81EE79 kernel32.lstrcmpA
00955008 7C838CB9 kernel32.GetStringTypeA
0095500C 7C80CEC4 kernel32.LCMapStringW
00955010 7C832E2B kernel32.LCMapStringA
00955014 7C809CAD kernel32.MultiByteToWideChar
00955018 00B60048                   //这个是 80 标志的函数
```

……

为了了解代码是如何处理 `80` 标志的函数，我们跟进 `00955018` 这个 `IAT` 地址所指的 `00B60048` 函数。按 `Ctrl` ＋ `G` 打开地址框，输入 `0B60048`（注意：地址首位如果是字母，别忘加个 `0`）：

```
00B60048 60       pushad //在这按右键新建 EIP 源，将当前 EIP 指针指到此处
00B60049 66:BF B55B mov di, 5BB5
00B6004D B2 80    mov dl, 80
00B6004F - E9 E350D9FF jmp 008F5137
```

除去花指令后的代码：

```
008F52C7 90       nop
008F52C8 90       nop
008F52C9 90       nop
008F52CA 90       nop
008F52CB 90       nop
008F52CC 0FB6D2   movzx edx, dl
008F52CF 90       nop
008F52D0 90       nop
008F52D1 90       nop
008F52D2 0FB7C7   movzx eax, di
```

……

```
008F5399 8D34C6   lea esi, [esi+eax*8] //来到这里，查看 ESI
008F53A0 0FB706   movzx eax, word ptr [esi]
008F53AC 83F8 04  cmp eax, 4
008F53B5 0F84 78010000 je 008F5533
008F53C4 83F8 01  cmp eax, 1
008F53CA 0F84 11010000 je 008F54E1
008F53DA 3D 80000000 cmp eax, 80
008F53E3 /0F84 AE070000 je 008F5B97//这里会跳
```

到 `008F5209` 这句查看 `ESI`：

```
008F09B4 80 00 00 00 04 00 00 00 80 00 00 00 0A 00 00 00 ?..?...
```

这个就是上面的那个数据表，`80` 就是特殊函数标志，后面的 `04` 就是与函数名有关的参数。

```
008F5BBB 8BD5     mov edx, ebp
008F5BBD 90       nop
008F5BBE 90       nop
008F5BBF 90       nop
008F5BC0 90       nop
008F5BC1 90       nop
008F5BC2 90       nop
008F5BC3 039485 9401EC00 add edx, [ebp+eax*4+EC0194]
008F5BCA 90       nop
008F5BCB 90       nop
008F5BCC 90       nop
008F5BCD 90       nop
008F5BCE FFE2     jmp edx
```

继续走，来到如下：

```
008F5D23 90       nop
008F5D24 90       nop
008F5D25 90       nop
008F5D26 8B83 C4010000 mov eax, [ebx+1C4]
008F5D2C 90       nop
008F5D2D 90       nop
008F5D2E 90       nop
008F5D2F 90       nop
008F5D30 90       nop
008F5D31 90       nop
008F5D32 ^ E9 E7FDFFFF  jmp 008F5B1E
```

查看 `ebx+1C4`，会发现一组代码的地址，原来程序是根据不同的函数参数确定函数名，你可以 `Ctrl` ＋ `G` 输入下面的地址查看代码：

```
008F0390 008F223D 008F22C7 008F23AB 008F232D
008F03A0 008F2423 008F2489 008F24EF 008F2555
008F03B0 008F2633 008F269F 008F2705 008F27E9
008F03C0 008F285B 008F25C1 008F2777 008F28C7
008F03D0 008F2159 008F21CB 008F20A1 008F20FD
008F03E0 008F1FD5 008F203B 008F1E73 008F1EF1
008F03F0 008F1F63 008F2951 008F29C3 008F2A47
008F0400 008F2AC5 008F2B2B 008F1C98 00000000
```

上面每个地址是一段代码的入口，处理某个 `API` 函数，下面要做的就是找到这些代码共同出口点，为补丁程序做准备。继续：

```
008F5B26 8038 CC  cmp byte ptr [eax], 0CC
008F5B2E 74 1B    je short 008F5B4B
008F5B38 894424 1C mov [esp+1C], eax
008F5B3C 90       nop
008F5B3D 90       nop
008F5B3E 90       nop
008F5B3F 90       nop
008F5B40 61       popad
008F5B41 90       nop
008F5B42 90       nop
008F5B43 90       nop
008F5B44 90       nop
008F5B45 FFE0     jmp eax //根据参数不同，跳到不同代码里获取函数地址
```

这次跟进 `eax` 为 `008F2423` 值，`jmp 008F2423` 地址，来到：

```
008F2423 55       push ebp
008F2424 8BEC     mov ebp, esp
008F2426 83EC 08  sub esp, 8
008F2429 53       push ebx
008F242A 56       push esi
008F242B 57       push edi
008F242C E8 00000000 call 008F2431
008F2431 5B       pop ebx
008F2432 8BF3     mov esi, ebx
008F2434 8B9B 4FF4FFFF mov ebx, [ebx-BB1]
008F243A 8975 F8  mov [ebp-8], esi
008F243D 8DBE A7070000 lea edi, [esi+7A7]
008F2443 BE 04000000 mov esi, 4
008F2448 8B07     mov eax, [edi]
008F244A 85C0     test eax, eax
008F244C 74 14    je short 008F2462
008F244E 8D55 FC  lea edx, [ebp-4]
008F2451 FF75 08  push dword ptr [ebp+8]
008F2454 52       push edx
008F2455 FFD0     call eax
008F2457 83F8 01  cmp eax, 1
008F245A 74 21    je short 008F247D
008F245C 83C7 04  add edi, 4
008F245F 4E       dec esi
008F2460 ^ 75 E6  jnz short 008F2448
008F2462 8B75 F8  mov esi, [ebp-8]
008F2465 FF75 08  push dword ptr [ebp+8]
008F2468 FFB6 57090000 push dword ptr [esi+957] //esi+957 指向的就是正确的函数名，按 F7 跟进
008F246E FF93 84000000 call [ebx+84]
008F2474 5F       pop edi
008F2475 5E       pop esi
008F2476 5B       pop ebx
008F2477 8BE5     mov esp, ebp
008F2479 5D       pop ebp
008F247A C2 0400  retn 4
```

`008F2468` 这个 `CALL` 进入会来到外壳自己的领空，其地址是 `4xxxxx`：

```
0040C633 /EB 01   jmp short 0040C636 //在这可以用 mov eax, [esp+4] 指令取得函数的地址
0040C635 |5D      pop ebp
```



`0040C636` `\\EB 01` `jmp short 0040C639`  
`0040C638` `04 60` `add al, 60`  
`0040C63A` `EB 04` `jmp short 0040C640`  

这就意味着不同的函数，最终都会来到这里，这为补丁程序取得函数名地址提供了可能。

### 补丁思路

1. 处理标志为 `80` 函数时，跳到 `IAT` 指定的地址处执行。
2. 补丁 `0040C633`，让其跳回 `IAT` 普通函数处理代码里。

当然如果特殊函数不多，你不怕麻烦可以不用这方法来处理，手动跟踪那些没被识别出的函数，找到正确的函数指针。

按这思路构造的代码如下，红色部分是处理标志为 `80` 的函数：

（由于每次跟踪，外壳这段代码地址不同，下面这段代码是第二次运行抓取的，因此同一指令和上文提供的地址不同，大家以汇编代码来识别）

```
008F7564        test word ptr ds:[esi],8
008F756C        jnz 008F7605
008F7578        test word ptr ds:[esi],2
008F7582        jnz short 008F75CB
008F7584        test word ptr ds:[esi],80    //如是 80 标志的函数就处理
008F7589        je short 008F7605
008F758B        mov eax,dword ptr ds:[esi+4]
008F758E        pushad
008F758F        mov edx,954FF8              //954FF8 地址是内存中的一空白，这里是 IAT 前面部分，用以保存临时变量
008F7594        mov dword ptr ds:[edx],esp  //将 esp 的值保存在 954FF8 这个地址变量里
008F7596        jmp dword ptr ds:[edi]      //跳到 IAT 指定的地址运行
008F7598        nop
008F7599        nop
008F759A        nop                       //IAT 指定地址运行结束后，让它返回到这里
008F759B        mov eax,dword ptr ss:[esp+4] //取得函数地址
008F759F        mov esp,dword ptr ds:[954FF8] //恢复堆栈
008F75A5        mov dword ptr ss:[esp+1C],eax //eax 的结果放进堆栈里，以便 popad 时，会放到 eax 寄存器里
008F75A9        popad
008F75AA        mov word ptr ds:[esi],4
008F75AF        mov dword ptr ds:[edi],eax  //将得到的函数地址放进 IAT
008F75B1        nop
008F75B2        nop
……（中间全是 NOP 指令）
008F75C6       EB 3D             jmp short 008F7605  //继续处理下一个函数
008F75D0       66:C706 0400      mov word ptr ds:[esi],4
008F75D9       0FB756 02         movzx edx,word ptr ds:[esi+2]
008F75E1       6A 01             push 1
008F75E3       52                push edx
008F75E4       6A 00             push 0
008F75E6       FF76 04           push dword ptr ds:[esi+4]
008F75E9       FF75 18           push dword ptr ss:[ebp+18]
008F75EC       FF53 54           call dword ptr ds:[ebx+54]
008F75F4       85C0              test eax,eax
008F75F9       74 0A             je short 008F7605
008F75FE       8907              mov dword ptr ds:[edi],eax
008F7608       83C6 08           add esi,8
008F760F       83C7 04           add edi,4
008F7618       FF4D 08           dec dword ptr ss:[ebp+8]
008F761E      ^ 0F85 3CFFFFFF    jnz 008F7560
008F7629       33C0              xor eax,eax   //在这设个断点，执行完上述代码，80 标志的函数就还原了
008F762F       40                inc eax
008F7636       5F                pop edi
008F7637       5E                pop esi
008F7638       5B                pop ebx
008F7639       5D                pop ebp
008F763A       C2 1400           retn 14
```

上面这段代码有几点要注意的：

第一个就是 `954FF8` 地址，我这取的是 `IAT`（`IAT` 起始地址是 `955000`）前面那个空间，由于不同系统分配的内存地址可能不同，所以你得根据你的情况将这个值定好。

第二个就是 `008F7582`～`008F75C6` 之间代码别忘全部 `NOP` 掉，用新增的代码替换。

第三就是处理函数的每个代码入口参数不一样，因此必须将 `ESP` 保存到一变量了。

上面修改代码二进制如下，实际操作时，可以用 `OD` 的二进制粘贴功能将代码复制过去，别忘了还要根据你的情况修正 `954FF8` 变量地址。

`66 F7 06 80 00 74 7A 8B 46 04 60 BA F8 4F 95 00 89 22 FF 27 90 90 90 8B 44 24 04 8B 25 F8 4F 95 00 89 44 24 1C 61 66 C7 06 04 00 89 07`

另外，再按 `Ctrl+G` 跳到 `0040C633`，这是所有代码的出口，将其改成：

`0040C633 - E9 62AF4E00 jmp 008F759 //别忘设个断，所有函数处理完毕回调用这里的代码，你再撤消选择将其还原`

### 2. 处理特殊加密函数

走出上面的普通函数，进入特殊函数处理的 `CALL:`:

```
008F7424    90               nop
008F7425    90               nop
008F7426    FF75 EC          push    dword ptr [ebp-14]
008F7429    53               push    ebx
008F742A    52               push    edx
008F742B    50               push    eax
008F742C    FF76 0C          push    dword ptr [esi+C]
008F742F    E8 A5020000      call    008F76D9//进入这个处理特殊函数的 CALL
```

`F7` 走进：

```
008F76D9    55               push    ebp
008F76DA    8BEC             mov     ebp, esp
008F76DC    53               push    ebx
008F76DD    56               push    esi
008F76DE    57               push    edi
008F76DF    8B75 10          mov     esi, [ebp+10]
…………
008F771A    83F8 03          cmp     eax, 3
008F771D    74 12            je      short 008F7731
008F771F    83F8 04          cmp     eax, 4
008F7722   ^ 75 CA           jnz     short 008F76EE
008F7724    8B45 14          mov     eax, [ebp+14]
008F7727    8B90 E8000000    mov     edx, [eax+E8]
008F772D    8917             mov     [edi], edx
008F772F   ^ EB BD           jmp     short 008F76EE
008F7731    8B45 14          mov     eax, [ebp+14]
008F7734    68 C5B1662D      push    2D66B1C5
008F7739    6A 00            push    0
008F773B    FF50 18          call    [eax+18]
008F773E    50               push    eax
008F773F    53               push    ebx
008F7740    E8 98020000      call    008F79DD
008F7745    53               push    ebx
008F7746    E8 19020000      call    008F7964
008F774B    8BCB             mov     ecx, ebx
008F774D    8D5C03 01        lea     ebx, [ebx+eax+1]
008F7751    8BC1             mov     eax, ecx
008F7753    EB 2B            jmp     short 008F7780
008F7755    8B45 14          mov     eax, [ebp+14]
008F7758    68 0F1ACF4C      push    4CCF1A0F
008F775D    6A 00            push    0
008F775F    FF50 18          call    [eax+18]   //解密函数，按 F7 进入
```

```
{
  0040CE93    90               nop
  0040CE94    90               nop
  0040CE95    90               nop
  0040CE96    60               pushad
  0040CE9B    83EC 04          sub     esp, 4
  0040CEA3    E8 00000000      call    0040CEA8
  0040CEAC    5B               pop     ebx
  0040CEB1    8BEB             mov     ebp, ebx
  0040CEB7    8B5B E7          mov     ebx, [ebx-19]
  0040CEC4    8B4424 28        mov     eax, [esp+28]
  0040CED4    33C9             xor     ecx, ecx
  0040CEDC    8B8483 98000000  mov     eax, [ebx+eax*4+98]
  0040CEE9    8B5424 2C        mov     edx, [esp+2C]
  0040CEF3    51               push    ecx
  0040CEF4    51               push    ecx
  0040CEF5    51               push    ecx
  0040CEF6    52               push    edx
  0040CEF7    50               push    eax
  0040CEF8    FF53 54          call    [ebx+54] //Obsidium 自己实现的 GetProcAddress
  0040CF01    85C0             test    eax, eax
  0040CF06    0F84 56020000    je      0040D162
  0040CF0C    90               nop    //改成 MOV [EDI], EAX  同时设断
  0040CF0D    90               nop
  0040CF11    8BF0             mov     esi, eax
}
```

```
008F7762   /EB 1C            jmp     short 008F7780
008F7764   |8B45 14           mov     eax, [ebp+14]
008F7767   |68 A41A86D0       push    D0861AA4
008F776C   |6A 00             push    0
008F776E   |FF50 18           call    [eax+18]
008F7771   |EB 0D             jmp     short 008F7780
008F7773   |8B45 14           mov     eax, [ebp+14]
008F7776   |68 E313B41D       push    1DB413E3
008F777B   |6A 00             push    0
008F777D   |FF50 18           call    [eax+18]
008F7780   \C603 B8           mov     byte ptr [ebx], 0B8
008F7783    8943 01           mov     [ebx+1], eax
008F7786    8B45 14           mov     eax, [ebp+14]
008F7789    8B90 A4010000     mov     edx, [eax+1A4]
008F778F    8D43 0A           lea     eax, [ebx+A]
008F7792    2BD0              sub     edx, eax
008F7794    C643 05 E9        mov     byte ptr [ebx+5], 0E9
008F7798    8953 06           mov     [ebx+6], edx
008F779B    891F              mov     [edi], ebx       //这句 NOP 掉，因为处理完，会用其地址覆盖\edi
```

在：`0040CF0C 90 nop //改成 MOV [EDI], EAX //同时设断`

这一行，会中断三次，得到三个特殊函数：

`GetCommandLineA`  
`GetVersion`  
`GetCurrentProcess`

所有的特殊函数处理完毕后，再执行，程序返回的 `EDI` 值己不指向 `IAT` 了，因此必须将 `MOV [EDI], EAX` 这句删除，还原代码。

处理完 `kernel32.dll` 后将处理 `user32.dll`，程序会再循环回到那个解密普通函数的 `CALL` 程序再次执行的。`user32.dll` 没有特殊函数，出来后：



### 汇编代码分析

```
008F73D3    E8 E4010000     call    008F75BC         //普通加密函数  

008F73DE    85C0            test    eax, eax  

008F73E3    0F84 18010000   je      008F7501  

008F73F2    837D F0 00      cmp     dword ptr [ebp-10], 0  

008F73F9    74 43           je      short 008F743E    //如跳，则这个 DLL 里没有特殊函数  

008F7405    8B46 10         mov     eax, [esi+10]  

008F740D    8B56 14         mov     edx, [esi+14]  

008F7414    0343 10         add     eax, [ebx+10]  

008F741D    0353 48         add     edx, [ebx+48]  

008F7426    FF75 EC         push    dword ptr [ebp-14]  

008F7429    53              push    ebx  

008F742A    52              push    edx  

008F742B    50              push    eax  

008F742C    FF76 0C         push    dword ptr [esi+C]  

008F742F    E8 A5020000     call    008F76D9          //特殊加密函数  

008F7438    0145 EC         add     [ebp-14], eax  

008F7445    33C0            xor     eax, eax  

008F744D    8946 0C         mov     [esi+C], eax  

008F7455    8946 10         mov     [esi+10], eax  

008F745B    83C6 18         add     esi, 18  

008F7461    FF45 F8         inc     dword ptr [ebp-8]  

008F7467    FF4D FC         dec     dword ptr [ebp-4]  

008F746F    0F85 05FDFFFF   jnz     008F717A  

008F747B    33C0            xor     eax, eax          //将断点设在，处理所有的 DLL 就会中断在这  

008F7481    5F              pop     edi  

008F7482    5E              pop     esi  

008F7483    5B              pop     ebx  

008F7484    8BE5            mov     esp, ebp  

008F7486    5D              pop     ebp  

008F7487    C3              retn  
```

### IAT 查看

所有的 DLL 处理完毕查看一下 IAT：

```
00955000  7C80C729  kernel32.lstrcpyA  
00955004  7C81EE79  kernel32.lstrcmpA  
00955008  7C838CB9  kernel32.GetStringTypeA  
0095500C  7C80CEC4  kernel32.LCMapStringW  
00955010  7C832E2B  kernel32.LCMapStringA  
00955014  7C809CAD  kernel32.MultiByteToWideChar  
00955018  7C801D77  kernel32.LoadLibraryA  
0095501C  7C80AC28  kernel32.GetProcAddress  
00955020  7C9379FD  ntdll.RtlReAllocateHeap  
00955024  7C809A81  kernel32.VirtualAlloc  
00955028  7C9305D4  ntdll.RtlAllocateHeap  
0095502C  7C81E82A  kernel32.GetOEMCP  
00955030  7C809943  kernel32.GetACP  
00955034  7C812BE6  kernel32.GetCPInfo  
00955038  7C80B529  kernel32.GetModuleHandleA  
0095503C  7C801EEE  kernel32.GetStartupInfoA  
00955040  7C812C8D  kernel32.GetCommandLineA  
00955044  7C8114AB  kernel32.GetVersion  
00955048  00B600D8                                  //这个没找出，这个函数的特殊标志是 40  
0095504C  7C801E16  kernel32.TerminateProcess  
00955050  7C80E00D  kernel32.GetCurrentProcess  
00955054  7C862B8A  kernel32.UnhandledExceptionFilter  
00955058  7C80B357  kernel32.GetModuleFileNameA  
0095505C  7C81DC3F  kernel32.FreeEnvironmentStringsA  
00955060  7C81485F  kernel32.FreeEnvironmentStringsW  
00955064  7C80A0C7  kernel32.WideCharToMultiByte  
00955068  7C81CC23  kernel32.GetEnvironmentStringsA  
0095506C  7C812C78  kernel32.GetEnvironmentStringsW  
00955070  7C80C6CF  kernel32.SetHandleCount  
00955074  7C812CA9  kernel32.GetStdHandle  
00955078  7C811069  kernel32.GetFileType  
0095507C  7C81486A  kernel32.GetEnvironmentVariableA  
00955080  7C812851  kernel32.GetVersionExA  
00955084  7C811110  kernel32.HeapDestroy  
00955088  7C812929  kernel32.HeapCreate  
0095508C  7C809B14  kernel32.VirtualFree  
00955090  7C93043D  ntdll.RtlFreeHeap  
00955094  7C957A40  ntdll.RtlUnwind  
00955098  7C810F9F  kernel32.WriteFile  
0095509C  7C80A480  kernel32.GetStringTypeW  
009550A0  00B601E0  
009550A4  77D1A8AD  USER32.wsprintfA  
009550A8  77D6AC1E  USER32.GetDlgItemTextA  
009550AC  77D1BE71  USER32.EnableWindow  
009550B0  77D1DA60  USER32.SetFocus  
009550B4  77D31F4C  USER32.MessageBeep  
009550B8  77D21324  USER32.LoadIconA  
009550BC  77D3C2BF  USER32.SendDlgItemMessageA  
009550C0  77D1DAEA  USER32.DestroyWindow  
009550C4  77D24816  USER32.GetDlgItem  
009550C8  77D2F39A  USER32.SendMessageA  
009550CC  77D26250  USER32.EndDialog  
009550D0  77D3B11C  USER32.DialogBoxParamA  
```

### 手工跟踪未获取的函数

最后还有一个函数没获得：`00955048` `00B600D8`。

因此手工跟。在 `00B600D8` 新建 EIP：

```
00B600D8        60              pushad  
00B600D9        66:BF A15B      mov     di, 5BA1  
00B600DD        B2 80           mov     dl, 80  
00B600DF        E9 C351D9FF     jmp     008F52A7  
```

……

```
008F53D2        83F8 40         cmp     eax, 40  
008F53D5        90              nop  
008F53D6        90              nop  
008F53D7        90              nop  
008F53D8        0F84 AB010000   je      008F5589  
008F57E9        6A 00           push    0  
008F57EB        6A 45           push    45  
008F57ED        6A 00           push    0  
008F57EF        68 CC971025     push    251097CC  
008F57F4        FF37            push    dword ptr [edi]  
008F57F6        FF53 54         call    [ebx+54]      //经过这个 CALL，获得 EAX 7C81CAA2 kernel32.ExitProcess  
```

执行到 `008F747B    33C0            xor     eax, eax` 这句后，就可不执行了，接下来运行 `ImportREC` 修复输入表。

### ImportREC 配置与修复

将 IAT 的地址 `RVA: 555000`，大小 `D4` 填进，`Get Imports` 后就能得到正确的输入表。

`Obsidium` 这种重定位的壳，`ImportREC` 计算是按脱壳前文件的基址，例如这个实例脱壳前的基址是 `00400000`，IAT 的 VA 地址是 `955000`，则 `ImportREC` 里的 RVA 填 `955000 - 400000 = 555000`。

如果外壳重定位的基址低于外壳基址，计算方法还是一样的。例假设这个实例重定位后的映像基址是 `3F0000`，则 IAT 的 VA 为 `3F5000`，`ImportREC` 里的 RVA 填 `3F5000 - 00400000 = FFFF5000`（多余的 F 舍去）。

这样 `ImportREC` 就能工作了，但最后 `Fix Dump` 会出错，提示“Invalid dump file!Can't match RVA to Offset in the dump file”。

解决办法很简单，此时点击 `Save Tree`，将 IAT 保存到文件里，打开这个文本文件：

```
FThunk: FFFF5000  NbFunc: 00000028  

1  FFFF5000  kernel32.dll  03AD  lstrcpy  

1  FFFF5004  kernel32.dll  03A7  lstrcmp  

1  FFFF5008  kernel32.dll  01B0  GetStringTypeA  

1  FFFF500C  kernel32.dll  0235  LCMapStringW  
```

将 `FFFF` 用 `0000` 全部替换：

```
FThunk: 00005000  NbFunc: 00000028  

1  00005000  kernel32.dll  03AD  lstrcpy  

1  00005004  kernel32.dll  03A7  lstrcmp  

1  00005008  kernel32.dll  01B0  GetStringTypeA  
```

最后 `ImportREC` 打开进程，点击 `Load Tree`，这次 `Fix Dump` 就能成功修复文件了。

---

kanxue

看雪技术论坛 `www.pediy.com`

2005.12.18

`4dd466d0e1e0cc123afbe504da2462ca`
