
# 壳之旅 FSG2.0+脱壳(含 OD 脚本源码对输入表的处理)及其一个 BUG

> 原文：[`www.pediy.com/kssd/pediy10/63417.html`](https://www.pediy.com/kssd/pediy10/63417.html)

## 1. FSG2.0+ 壳入口处的代码

这个 FSG2.0+的壳对输入表有一点小小的处理。

FSG2.0+壳入口处的代码:

```
00400154  >  8725 2C794100   xchg    dword ptr [41792C], esp
0040015A    61             popad
0040015B    94             xchg    eax, esp
0040015C    55             push    ebp
0040015D    A4             movs    byte ptr es:[edi], byte ptr [esi]          ;Save
0040015E    B6 80           mov     dh, 80
00400160    FF13           call    dword ptr [ebx]                        ; call Decode
00400162  >^ 73 F9         jnb     short 0040015D                        ; goto Save …..
00400164    33C9           xor     ecx, ecx
00400166    FF13           call    dword ptr [ebx]
00400168    73 16          jnb     short 00400180
0040016A    33C0           xor     eax, eax
0040016C    FF13           call    dword ptr [ebx]
0040016E    73 1F          jnb     short 0040018F
00400170    B6 80          mov     dh, 80
00400172    41             inc     ecx
00400173    B0 10          mov     al, 10
00400175    FF13           call    dword ptr [ebx]
00400177    12C0           adc     al, al
00400179  ^ 73 FA          jnb     short 00400175
0040017B    75 3A          jnz     short 004001B7
0040017D    AA             stos    byte ptr es:[edi]
0040017E  ^ EB E0          jmp     short 00400160
00400180    FF53 08        call    dword ptr [ebx+8]
00400183    02F6           add     dh, dh
00400185    83D9 01        sbb     ecx, 1
00400188    75 0E          jnz     short 00400198
0040018A    FF53 04        call    dword ptr [ebx+4]
0040018D    EB 24          jmp     short 004001B3
0040018F    AC             lods    byte ptr [esi]
00400190    D1E8           shr     eax, 1
00400192    74 2D          je      short 004001C1
00400194    13C9           adc     ecx, ecx
00400196    EB 18          jmp     short 004001B0
00400198    91             xchg    eax, ecx
00400199    48             dec     eax
0040019A    C1E0 08        shl     eax, 8
0040019D    AC             lods    byte ptr [esi]
0040019E    FF53 04        call    dword ptr [ebx+4]
004001A1    3B43 F8       cmp     eax, dword ptr [ebx-8]
004001A4    73 0A          jnb     short 004001B0
004001A6    80FC 05       cmp     ah, 5
004001A9    73 06          jnb     short 004001B1
004001AB    83F8 7F       cmp     eax, 7F
004001AE    77 02          ja      short 004001B2
004001B0    41             inc     ecx
004001B1    41             inc     ecx
004001B2    95             xchg    eax, ebp
004001B3    8BC5           mov     eax, ebp
004001B5    B6 00          mov     dh, 0
004001B7    56             push    esi
004001B8    8BF7           mov     esi, edi
004001BA    2BF0           sub     esi, eax
004001BC    F3:A4         rep     movs byte ptr es:[edi], byte ptr [esi]
004001BE    5E             pop     esi
004001BF  ^ EB 9F          jmp     short 00400160                        ; goto Decode
004001C1    5E             pop     esi
004001C2    AD             lods    dword ptr [esi]                    ; While
004001C3    97             xchg    eax, edi
004001C4    AD             lods    dword ptr [esi]
004001C5    50             push    eax                              ; strLibraryName
004001C6  >  FF53 10       call    dword ptr [ebx+10]                    ; kernel32.LoadLibraryA
004001C9    95             xchg    eax, ebp
004001CA    8B07           mov     eax, dword ptr [edi]
004001CC    40             inc     eax
004001CD  ^ 78 F3          js      short 004001C2
004001CF    75 03          jnz     short 004001D4
004001D1  -  FF63 0C       jmp     dword ptr [ebx+C]                        ; GoToOEP
004001D4    50             push    eax
004001D5    55             push    ebp
004001D6    FF53 14       call    dword ptr [ebx+14]                    ; kernel32.GetProcAddress
004001D9    AB             stos    dword ptr es:[edi]
004001DA  ^ EB EE          jmp     short 004001CA                    ; endwhile
004001DC    33C9           xor     ecx, ecx
004001DE    41             inc     ecx
004001DF    FF13           call    dword ptr [ebx]
004001E1    13C9           adc     ecx, ecx
004001E3    FF13           call    dword ptr [ebx]
004001E5  ^ 72 F8          jb      short 004001DF
004001E7    C3             retn
004001E8    02D2           add     dl, dl
004001EA    75 05          jnz     short 004001F1
004001EC    8A16           mov     dl, byte ptr [esi]
004001EE    46             inc     esi
004001EF    12D2           adc     dl, dl
004001F1    C3             retn
```

`0040015D` 和 `004001BF` 之间的是解码过程，不用理会。

动态跟一下就会发现 `004001D1 - FF63 0C jmp dword ptr [ebx+C]` 就是跳到 OEP 的。

浏览一下代码发现 `ebx` 在整个过程中的作用还是很大的。当前 `ebx = 00417930`，数据跟随一下：

```
00417930  004001E8  ASCII 02,""
00417934  004001DC  记事本.004001DC
00417938  004001DE  记事本.004001DE
0041793C  0040010CC 记事本.004010CC
00417940 >7C801D77  kernel32.LoadLibraryA
00417944 >7C80ADA0  kernel32.GetProcAddress
……
```

看见没有？这里存储了一些重要的数据，其中 `[ebx+c]` 就是程序 OEP。

我们看看 `ebx` 从什么时候开始就确定的，一直向上看，直到：

```
00400154  >  8725 2C794100   xchg    dword ptr [41792C], esp
0040015A    61             popad
0040015B    94             xchg    eax, esp                        ; 记事本.00417930
```

即 `popad` 执行过后 `ebx` 就确定了。

如果要脱壳的话就在 `0040015B` 处断掉，直接取 `[ebx+c]` 的值就 OK 了。

入口跟随到 OEP 后直接 dump 后的程序是不能运行的，还得处理输入表。

`004001C2 AD lods dword ptr [esi]` 一句开始处理输入表的，先定义一下数据结构吧，这样容易描述。

```
TYPE TImp = Record
   Pfunction;             //要存从某 dll 中导入的函数的地址
   LpLibraryName;        //dll 名字
End;
```

则 `esi` 为：`array[0..0] of TImp`，最后以一个为 0 的 `TImp` 结构结束。如：

```
0040F000  004062E0  记事本.004062E0
0040F004  0040F078  ASCII "ADVAPI32.dll"
0040F008  004062F8  记事本.004062F8
0040F00C  0040F0D5  ASCII "GDI32.dll"
0040F010  00406358  记事本.00406358
0040F014  0040F22A  ASCII "kernel32.dll"
0040F018  004063F0  记事本.004063F0
0040F01C  0040F424  ASCII "SHELL32.dll"
0040F020  0040640C  记事本.0040640C
0040F024  0040F497  ASCII "USER32.dll"
0040F028  00000000
0040F02C  00000000
```

输入函数地址的存放形式是：

```
004062E0  77DAEBE7  ADVAPI32.RegSetValueExA
004062E4  77DA7883  ADVAPI32.RegQueryValueExA
004062E8  77DA6BF0  ADVAPI32.RegCloseKey
004062EC  77DCC41B  ADVAPI32.RegOpenKeyA
004062F0  77DCD5BB  ADVAPI32.RegCreateKeyA
004062F4  7FFFFFFF
004062F8  77EF61C1  GDI32.GetStockObject
………
00406350  77EFED13  GDI32.CreateFontIndirectA
00406354  7FFFFFFF
00406358  7C831EAB  kernel32.DeleteFileA
………
004063E8  7C80FF19  kernel32.GlobalLock
004063EC  7FFFFFFF
004063F0  7D610EE0  SHELL32.ShellExecuteA
………
00406404  7D6470E8  SHELL32.DragFinish
00406408  7FFFFFFF
0040640C  77D1A8AD  USER32.wsprintfA
………
004064F8  77D2F52B  USER32.SetWindowTextA
004064FC  FFFFFFFF
```

即从不同的 dll 中导入的函数地址表以 `7FFFFFFF` 分隔，最后以 `FFFFFFFF` 结束。

## 2. 脱壳过程

为了处理输入表，我们还得关注一下 `ESI` 的值从哪里得到的。`004001C1 5E pop esi` 对应 `0040015C 55 push ebp`。

所以只要在 `0040015B 94 xchg eax, esp` 处下断获取两个寄存器中的重要值就 OK 了，一个是 `EBX`，一个是 `EBP`。

- 其中 `[ebx+c] = OEP`
- `EBP` 与输入表有关

脱壳吧。来到 OEP 后，直接 dump，发现程序不能运行，主要是输入表有问题。

使用 `ImportREC.exe` 修复也不行，再配合使用 `LordPe` 也不行。

我也不明白为什么大家都说 FSG2.0 的壳很容易就手脱了，而到了我的手里却不行呢？？？？？

我一直在怀疑，呵呵，不过 FSG 是公平的，它不会偏倚谁的，后来猜测可能是 FSG20+。

经过小小的分析就会发现输入表里的奥秘了。

如上图，每个 dll 中的函数分隔靠的是 `7FFFFFFF`。



最后以一个`FFFFFF`结束. 这就不是标准型的输入表结构, 应该是以`0`分隔和结束. 也就是说当来到`OEP`时, 将`7FFFFFFF`和`FFFFFF`修改为`0`, 再`dump`就`OK`了.

### 脱壳机的编写

单步执行前两条指令, 保存`EBP`中的值作为`pIAT`, 保存`[EBX+C]`为`OEP`, 直接在`OEP`处下硬件执行断点, `RUN`! 便中断在了`OEP`, 然后修复输入表, `dump`.

不过我不想编写脱壳机, 因为那样很花费时间. 而且在我发觉脚本的强大功能时, 我就更加不想再编写代码了, 我想体验一下脚本的强大魔力. 呵呵, 我的`OD`脚本处女作. 第一次写`OD`脚本, 相当不熟练, 写出来感觉很憋足.

```assembly
//文件名:FSG 脱壳.osc
//功能:脱 FSG2.0+的壳,主要是针对输入表不能修复的情况.

var   OEP
var   pIAT

Start:
       sto
       sto
       add       ebx, c
       mov      OEP, [ebx]
       sub       ebx, c
       mov      pIAT, ebp
       bphws   OEP, "x"
       run           //goooo  to oep

//来到 oep,修复输入表
@nextdll:
       mov eax,  pIAT
       mov eax, [eax]
       cmp eax, 0
       je  @over

@nextfunc:
       cmp      [eax], 7FFFFFFF
       je   @equ_7FFFFFFF
       cmp      [eax], FFFFFFFF
       je   @equ_FFFFFFFF
       add       eax, 4
       jmp @nextfunc

@equ_7FFFFFFF:
@equ_FFFFFFFF:
       mov      [eax], 0
       add       pIAT, 8
       jmp       @nextdll

@over:
       bphwc   OEP
ret

///dpe "c:\fsg_dumped.exe",OEP
```

//本来想用脚本直接 dump 到文件中, 可是用脚本 dump 出的文件不能运行, 只好运行到 OEP 再手动 Dump 了....
//最后到达 OEP 时若代码很乱, 就取消分析.

运行脚本后, 很快就能到`OEP`了, 然后`DUMP`处程序就`OK`了. 感觉很爽. 我一连测试了`5`个被`FSG2.0+`加过壳的程序(编写语言有: 汇编, `vc`, `delphi`, `VB`), 全部脱之全部`OK`.

然而相当不满意的是脱壳之后的程序要远远大于加壳前的程序, 例如一个`asm`编写的程序在加壳前只有`2.5Kb`, 加壳之后为`893`字节, 脱壳之后居然`25KB`, 使用`LordPE.EXE`重建一下还有`9.61kb`. 这一点相当有问题.

另: 发现`FSG 2.0`存在一个`BUG`: 有一个测试程序在加密后为`6.18k`(压缩比`95%`), 的确很强啊, 不过运行加密后的程序`CPU`就一直`100%`了, 程序却运行不起来了. 脱壳后的程序也是`CPU`一直`100%`.

我把这个原程序和加壳后的程序放在附件里了, 有兴趣的好手可以分析下, 告诉俺原因啊.

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `FSG 2.0+.rar` \[解压密码：`PEDIY`\] | |
|---|---|---|---|
