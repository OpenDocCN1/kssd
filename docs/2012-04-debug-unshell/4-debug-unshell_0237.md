
# 用 Arm3.75 加壳的 cc 版+iat 乱序主程序的脱壳

> 原文：[`www.pediy.com/kssd/pediy06/pediy6704.htm`](https://www.pediy.com/kssd/pediy06/pediy6704.htm)

用 Arm3.75 加壳的 ASProtect V1.31 build 06.14 主程序（1）

- **【破解作者】** `wangli_com`
- **【使用工具】** `WinXP`、`Ollydbg`、`PEiD`、`LordPE`、`ImportREC 1.6F`
- **【官方主页】** [`www.888cp.com/bbs/`](http://www.888cp.com/bbs/)
- **【破解平台】** `WinXP`
- **【软件名称】** `asprotect(v1.31 build 6.14) armadillo cc 版`
- **【下载地址】** [`bbs.pediy.com/showthread.php?s=&threadid=2453`](http://bbs.pediy.com/showthread.php?s=&threadid=2453)
- **【软件大小】** `1.14 MB`
- **【加壳方式】** `armadillo cc 版+iat 乱序+抽取代码`
- **【破解声明】** 我是一菜青虫，偶得一点心得，愿与大家分享 !!! 前面这几行模仿小虾，这种格式看起来比较舒服 !!!

## 1. 前言

初学脱壳，看着大虾们的文章，照猫画虎的破了几个，无意间一天被 armadillo 的一个单进程壳给难住了，就是在 `dfcg` 论坛上的“赤月 3（40515）脱壳笔记！|Armadillo 3.00a-3.61”，按照步骤一不不差，脱完一运行，初始化错误，代码 `00005`，为什么？重复了 `n` 遍，未果。于是激起了我学习 armadillo 的尽头，我到处查资料，看所有关于 armadillo 的文章，于是了解到双进程，cc 版+iat 乱序+抽取代码，一块一块的啃，一个多月时间，下了班就研究，好累啊！借鉴了很多大侠的文章，终于有了点收获，写出来与大家分享，也许方法比较笨拙，还望各位大侠指教。

## 2. 脱壳过程（分以下几步进行）

### （1）查找 OEP

用 `OD` 中载入程序，下 `he WaitForDebugEvent`，`F9` 运行，出现一个出错对话框，`shift+f9` 运行，中断如下：

```
77E93A07  k>  55             push   ebp          <====中断在此处，清除断点
77E93A08  k>  8BEC          mov    ebp,esp
77E93A0A  k>  83EC 68       sub    esp,68
77E93A0D  k>  56             push   esi
77E93A0E  k>  FF75 0C       push   dword ptr ss:[ebp+C]
```

查看堆栈窗口：

```
0012BC88    005140D5  /CALL 到 WaitForDebugEvent 来自 Aspr.005140CF
0012BC8C    0012CD60  |pDebugEvent = 0012CD60   <====注意这里
0012BC90    000003E8  \Timeout = 1000. ms
```

在 `0012CD60` 所在的行上点击右键选“转存中跟随”。

然后再下 `he WriteProcessMemory`，`F9` 运行：

```
77E41A90  k>  55             push   ebp<====中断在此处
77E41A91       8BEC          mov    ebp,esp
77E41A93       51             push   ecx
77E41A94       51             push   ecx
77E41A95       8B45 0C      mov    eax,dword ptr ss:[ebp+C]
```

看数据转存窗口：

```
0012CD60  01 00 00 00 BC 08 00 00  ...?..
0012CD68  20 09 00 00 01 00 00 80   .....€
0012CD70  00 00 00 00 00 00 00 00  ........
0012CD78  4C 87 4D 00 02 00 00 00  L 嘙....
0012CD80  00 00 00 00 4C 87 4D 00  ....L 嘙.
0012CD88  4C 87 4D 00 00 00 00 00  L 嘙.....
```

上面的 `004d874c` 就是 `OEP`.

### （2）dump 文件

现在我们重新用 `OD` 中载入程序，下硬件断点，`he WaitForDebugEvent`，`F9` 运行，中断如下：

```
77E93A07  k>  55 push ebp   <====中断在此处，删除断点
77E93A08  8BEC mov ebp,esp
77E93A0A  83EC 68 sub esp,68
77E93A0D  56 push esi
77E93A0E  FF75 0C push dword ptr ss:[ebp+C]
```

`Alt+F9` 返回，搜索常数 `FFFFFFFF`，来到（`0051467c` 处）：

```
00514630    83BD D0F5FFFF 00      cmp dword ptr ss:[ebp-A30],0<====参考文章里提到的关键代
码
00514637    0F8C A8020000         jl Aspr.005148E5
0051463D    8B8D D0F5FFFF        mov ecx,dword ptr ss:[ebp-A30]
00514643    3B0D E0075400        cmp ecx,dword ptr ds:[5407E0]
00514649    0F8D 96020000        jge Aspr.005148E5<====跳转到 0005148E5 处，因此在 05148E5
处下断点
0051464F    8B95 44F6FFFF        mov edx,dword ptr ss:[ebp-9BC]
00514655    81E2 FF000000       and edx,0FF
0051465B    85D2               test edx,edx
0051465D    0F84 AD000000       je Aspr.00514710
00514663    6A 00               push 0
00514665    8BB5 D0F5FFFF       mov esi,dword ptr ss:[ebp-A30]
0051466B    C1E6 04            shl esi,4
0051466E    8B85 D0F5FFFF       mov eax,dword ptr ss:[ebp-A30]
00514674    25 07000080         and eax,80000007
00514679    79 05              jns short Aspr.00514680
0051467B    48                 dec eax
0051467C    83C8 F8            or eax,FFFFFFF8<=====搜索来到这里，向上看在 0042438A 处
0051467F    40                 inc eax
00514680    33C9              xor ecx,ecx
00514682    8A88 2CE85300      mov cl,byte ptr ds:[eax+53E82C]
00514688    8B95 D0F5FFFF      mov edx,dword ptr ss:[ebp-A30]
0051468E    81E2 07000080      and edx,80000007
00514694    79 05             jns short Aspr.0051469B
00514696    4A                dec edx
00514697    83CA F8           or edx,FFFFFFF8
0051469A    42                inc edx
0051469B    33C0             xor eax,eax
0051469D    8A82 2DE85300     mov al,byte ptr ds:[edx+53E82D]
005146A3    8B3C8D B0A25300   mov edi,dword ptr ds:[ecx*4+53A2B0]
005146AA    333C85 B0A25300   xor edi,dword ptr ds:[eax*4+53A2B0]
005146B1    8B8D D0F5FFFF     mov ecx,dword ptr ss:[ebp-A30]
005146B7    81E1 07000080     and ecx,80000007
005146BD    79 05            jns short Aspr.005146C4
005146BF    49               dec ecx
005146C0    83C9 F8          or ecx,FFFFFFF8
005146C3    41               inc ecx
005146C4    33D2             xor edx,edx
005146C6    8A91 2EE85300    mov dl,byte ptr ds:[ecx+53E82E]
005146CC    333C95 B0A25300  xor edi,dword ptr ds:[edx*4+53A2B0]
005146D3    8B85 D0F5FFFF    mov eax,dword ptr ss:[ebp-A30]
005146D9    99               cdq
005146DA    B9 1C000000     mov ecx,1C
005146DF    F7F9            idiv ecx
005146E1    8BCA            mov ecx,edx
005146E3    D3EF            shr edi,cl
005146E5    83E7 0F         and edi,0F
005146E8    03F7            add esi,edi
005146EA    8B15 C4075400   mov edx,dword ptr ds:[5407C4]
005146F0    8D04B2          lea eax,dword ptr ds:[edx+esi*4]
005146F3    50             push eax
005146F4    8B8D D0F5FFFF   mov ecx,dword ptr ss:[ebp-A30]
005146FA    51             push ecx
005146FB    E8 1C210000     call Aspr.0051681C
00514700    83C4 0C          add esp,0C
00514703    25 FF000000      and eax,0FF<====从这里开始修改
00514708    85C0             test eax,eax
0051470A    0F84 D5010000    je Aspr.005148E5
```

在 `00514630` 处下硬件执行断点，`F9` 运行到 `00514630` 处，得到 `[ebp-A30]=12cd4c`，把 `12cd4c` 的“d7”改为 0，再修改上述的地方为：

```
00514703     FF05 4CCD1200        inc dword ptr ds:[12CD4C]
00514709     C705 E4075400 01000000 mov dword ptr ds:[5407E4],1
00514713   ^ E9 18FFFFFF         jmp Aspr.00514630
```

去掉所有断点，并在 `0005148E5` 处下断，`F9` 运行，断住。好了，所有代码都强制解压完成。

运行 `LordPE`，有 2 个关于 `Aspr.exe` 的进程，选择第 2 个，主程序可完全 dump 出来了。再选择区域脱壳选择 `10000000` 上面的保护属性为 `xrw`，大小为 `20000` 的代码段，用 `LordPE` 把这个区段装进脱出来的主文件中。

我的为 `Region033A0000-033C0000.dmp`，把虚拟地址改为 `033A0000-00400000=02fa0000`，这样偷走的代码被补了回来。每个人机子上都不一样，已自己机子为准。

用 `lordpe` 在主程序中加一个 section，`VOffset` 设置为 `02fa0000`，大小设置为 `20000`。用 `winhex` 把 dump 出来的 `033A0000` 处代码放回主程序。这样可以保证其中的跳转地址不会改变。

### （3）找 cc

在 `OD` 中重开程序 `Aspr.exe`，停在入口。下 `he GetThreadContext`，运行，第二次停住，按 `alt+f9` 回到用户空间。

```
005154C9     FF15 A4A05300        call dword ptr ds:[<&KERNEL32.GetThreadContext>]   ; kernel32.GetThreadContext
005154CF     50                   push eax//停在这，这条指令到 00515511 都是垃圾指令
005154D0     F7D0                 not eax\------>垃圾指令开始
005154D2     0FC8                 bswap eax
005154D4     58                   pop eax
005154D5     73 00                jnb short Aspr.005154D7
005154D7     9C                   pushfd
005154D8     60                   pushad
005154D9     EB 2B                jmp short Aspr.00515506
005154DB     D270 0E              sal byte ptr ds:[eax+E],cl
005154DE     EB 1D                jmp short Aspr.005154FD
005154E0     33C9                xor ecx,ecx
005154E2     74 00               je short Aspr.005154E4
005154E4     EB 10               jmp short Aspr.005154F6
005154E6     F9                  stc
005154E7     83F1 03             xor ecx,3
```



### 反汇编代码与注释

```
005154EA   ^ 74 90                     je short Aspr.0051547C
005154EC     EB 05                     jmp short Aspr.005154F3
005154EE     B9 EB1074EF               mov ecx, EF7410EB
005154F3   ^ 73 FA                     jnb short Aspr.005154EF
005154F5     F2:                       prefix repne:
005154F6     1ADB                      sbb bl, bl
005154F8     33DB                      xor ebx, ebx
005154FA   ^ EB EA                     jmp short Aspr.005154E6
005154FC   - E9 EBE1EBDD               jmp DE3D36EC
00515501     F2:                       prefix repne:
00515502     1C DB                     sbb al, 0DB
00515504     EB 03                     jmp short Aspr.00515509
00515506   ^ EB D2                     jmp short Aspr.005154DA
00515508     B8 619D0FC8               mov eax, C80F9D61
0051550D     F7D1                      not ecx
0051550F     0FC8                      bswap eax
00515511     F7D1                      not ecx  ------> 垃圾指令结束
00515513     C785 98EBFFFF 00000000    mov dword ptr ss:[ebp-1468], 0
```

### 修改后的代码

这部分是我自己改的：

```
005154CF     90                        nop
005154D0     90                        nop
005154D1     90                        nop
005154D2     90                        nop
005154D3     90                        nop
005154D4     90                        nop
005154D5     90                        nop
005154D6     90                        nop
005154D7     90                        nop
005154D8     90                        nop
005154D9     90                        nop
005154DA     90                        nop
005154DB     90                        nop
005154DC     8B0D F08F4D00             mov ecx, dword ptr ds:[4D8FF0]  ---> 计数器
005154E2     3E:8D348D 00104000        lea esi, dword ptr ds:[ecx*4+401000]
005154EA     BF 00104000               mov edi, Aspr.00401000
005154EF     8B85 54ECFFFF             mov eax, dword ptr ss:[ebp-13AC]
005154F5     F2:AF                     repne scas dword ptr es:[edi]
005154F7     74 16                     je short Aspr.0051550F  ----> 这段代码是排除重复的 cc 地址
005154F9     90                        nop
005154FA     90                        nop
005154FB     90                        nop
005154FC     90                        nop
005154FD     8906                      mov dword ptr ds:[esi], eax
005154FF     FF05 F08F4D00             inc dword ptr ds:[4D8FF0]
00515505     BF 0E000000               mov edi, 0E
0051550A     BE 10000000               mov esi, 10
0051550F     90                        nop
00515510     90                        nop
00515511     90                        nop
00515512     90                        nop
00515513     C785 98EBFFFF 00000000    mov dword ptr ss:[ebp-1468], 0
```

取消所有的断点，`f9` 运行，把所有的菜单都运行一下，不要漏掉什么，这是找全 `cc` 的关键，ok，所有的 `cc` 都乖乖的在 `401000` 处待着了，赶快把他复制下来。

### （3）修复 cc

关于这部分可参考【密界脱壳文集】第一版.chm 中 `pyzpyz` 的 `Blaze Media Pro5.05` 脱壳(armadillo)＋基本修复

`CC`(`int3`)＋破解，我也是照猫画虎。

在 `OD` 中重开程序 `Aspr.exe`，停在入口。下 `he GetThreadContext`，运行，第二次停住，按 `alt+f9` 回到用户空间。把上面找到的 `cc` 地址复制在 `40100` 处。

原代码如下：

```
005154C9     FF15 A4A05300             call dword ptr ds:[<&KERNEL32.GetThreadContext>]   ; kernel32.GetThreadContext
005154CF     50                        push eax // 停在这，这条指令到 00515511 都是垃圾指令
005154D0     F7D0                      not eax  ------> 垃圾指令开始
005154D2     0FC8                      bswap eax
005154D4     58                        pop eax
005154D5     73 00                     jnb short Aspr.005154D7
005154D7     9C                        pushfd
005154D8     60                        pushad
005154D9     EB 2B                     jmp short Aspr.00515506
005154DB     D270 0E                   sal byte ptr ds:[eax+E], cl
005154DE     EB 1D                     jmp short Aspr.005154FD
005154E0     33C9                      xor ecx, ecx
005154E2     74 00                     je short Aspr.005154E4
005154E4     EB 10                     jmp short Aspr.005154F6
005154E6     F9                        stc
005154E7     83F1 03                   xor ecx, 3
005154EA   ^ 74 90                     je short Aspr.0051547C
005154EC     EB 05                     jmp short Aspr.005154F3
005154EE     B9 EB1074EF               mov ecx, EF7410EB
005154F3   ^ 73 FA                     jnb short Aspr.005154EF
005154F5     F2:                       prefix repne:
005154F6     1ADB                      sbb bl, bl
005154F8     33DB                      xor ebx, ebx
005154FA   ^ EB EA                     jmp short Aspr.005154E6
005154FC   - E9 EBE1EBDD               jmp DE3D36EC
00515501     F2:                       prefix repne:
00515502     1C DB                     sbb al, 0DB
00515504     EB 03                     jmp short Aspr.00515509
00515506   ^ EB D2                     jmp short Aspr.005154DA
00515508     B8 619D0FC8               mov eax, C80F9D61
0051550D     F7D1                      not ecx
0051550F     0FC8                      bswap eax
00515511     F7D1                      not ecx   ------> 垃圾指令结束
00515513     C785 98EBFFFF 00000000    mov dword ptr ss:[ebp-1468], 0
0051551D     6A FF                     push -1
0051551F     6A 04                     push 4
00515521     8D95 54ECFFFF             lea edx, dword ptr ss:[ebp-13AC] ---> 子进程发生 int3 的地址 +1
00515527     52                        push edx
00515528     E8 638AFEFF               call Aspr.004FDF90  ----> 对 int3 地址运算
0051552D     83C4 0C                   add esp, 0C
00515530     8985 6CEEFFFF             mov dword ptr ss:[ebp-1194], eax   ---> 运算结果
00515536     8B85 6CEEFFFF             mov eax, dword ptr ss:[ebp-1194]
0051553C     33D2                      xor edx, edx
0051553E     B9 10000000               mov ecx, 10
00515543     F7F1                      div ecx           ---> 运算结果除以 10
00515545     8995 68EEFFFF             mov dword ptr ss:[ebp-1198], edx  ---> 取余数
0051554B     8B95 54ECFFFF             mov edx, dword ptr ss:[ebp-13AC] ---> int3 地址+1
00515551     52                        push edx
00515552     8B85 68EEFFFF             mov eax, dword ptr ss:[ebp-1198] ---> 前面得到的余数
00515558     FF1485 38E85300           call dword ptr ds:[eax*4+53E838] ---> 再次运算
0051555F     83C4 04                   add esp, 4
00515562     8985 98EBFFFF             mov dword ptr ss:[ebp-1468], eax ---> 运算结果，将用于在 table1（int3 地址计算结果表）中查找
00515568     C785 94EBFFFF 00000000    mov dword ptr ss:[ebp-146C], 0
00515572     8B8D 68EEFFFF             mov ecx, dword ptr ss:[ebp-1198]
00515578     8B148D E8065400           mov edx, dword ptr ds:[ecx*4+5406E8]
0051557F     8995 74EEFFFF             mov dword ptr ss:[ebp-118C], edx
00515585     8B85 94EBFFFF             mov eax, dword ptr ss:[ebp-146C]
0051558B     3B85 74EEFFFF             cmp eax, dword ptr ss:[ebp-118C]
00515591     7D 5C                     jge short Aspr.005155EF
00515593     8B85 74EEFFFF             mov eax, dword ptr ss:[ebp-118C]
00515599     2B85 94EBFFFF             sub eax, dword ptr ss:[ebp-146C]
0051559F     99                        cdq
005155A0     2BC2                      sub eax, edx
005155A2     D1F8                      sar eax, 1
005155A4     8B8D 94EBFFFF             mov ecx, dword ptr ss:[ebp-146C]
005155AA     03C8                      add ecx, eax
005155AC     898D 90EBFFFF             mov dword ptr ss:[ebp-1470], ecx
005155B2     8B95 68EEFFFF             mov edx, dword ptr ss:[ebp-1198] ---> 前面得到的余数
```



005155B8 `8B0495 88065400` `mov eax,dword ptr ds:[edx*4+540688]`--->[540688] 开始的是

`table1` 地址表，根据前面得到的余数找到 `table1` 的地址

005155BF `8B8D 90EBFFFF` `mov ecx,dword ptr ss:[ebp-1470]`

005155C5 `8B95 98EBFFFF` `mov edx,dword ptr ss:[ebp-1468]`

005155CB `3B1488` `cmp edx,dword ptr ds:[eax+ecx*4]`

005155CE `76 11` `jbe short Aspr.005155E1`

005155D0 `8B85 90EBFFFF` `mov eax,dword ptr ss:[ebp-1470]`

005155D6 `83C0 01` `add eax,1`

005155D9 `8985 94EBFFFF` `mov dword ptr ss:[ebp-146C],eax`

005155DF `EB 0C` `jmp short Aspr.005155ED`

005155E1 `8B8D 90EBFFFF` `mov ecx,dword ptr ss:[ebp-1470]`

005155E7 `898D 74EEFFFF` `mov dword ptr ss:[ebp-118C],ecx`

005155ED `^ EB 96` `jmp short Aspr.00515585`--->`eax`=查表结果(前面运算结果在

`table1` 中的序号)

005155EF `60` `pushad`------> 垃圾指令开始

005155F0 `33C0` `xor eax,eax`

005155F2 `75 02` `jnz short Aspr.005155F6`

005155F4 `EB 15` `jmp short Aspr.0051560B`

005155F6 `EB 33` `jmp short Aspr.0051562B`

005155F8 `C075 18 7A` `sal byte ptr ss:[ebp+18],7A`

005155FC `0C 70` `or al,70`

005155FE `0E` `push cs`

005155FF `EB 0D` `jmp short Aspr.0051560E`

00515601 `E8 720E79F1` `call F1CA6478`

00515606 `FF15 00790974` `call dword ptr ds:[74097900]`

0051560C `F0:EB 87` `lock jmp short Aspr.00515596` ; 锁定前缀是不允许的

0051560F `DB7A F0` `fstp tbyte ptr ds:[edx-10]`

00515612 `A0 33618B95` `mov al,byte ptr ds:[958B6133]`

00515617 `68 EEFFFF8B` `push 8BFFFFEE`

0051561C `04 95` `add al,95`

0051561E `8806` `mov byte ptr ds:[esi],al`

00515620 `54` `push esp`

00515621 `008B 8D94EBFF` `add byte ptr ds:[ebx+FFEB948D],cl`

00515627 `FF8B 14883B95` `dec dword ptr ds:[ebx+953B8814]`

0051562D `98` `cwde`

0051562E `EB FF` `jmp short Aspr.0051562F`

00515630 `FF0F` `dec dword ptr ds:[edi]`

00515632 `8512` `test dword ptr ds:[edx],edx`

00515634 `0300` `add eax,dword ptr ds:[eax]`

00515636 `0051 0F` `add byte ptr ds:[ecx+F],dl`

00515639 `C9` `leave`

0051563A `F7D1` `not ecx`

0051563C `50` `push eax`

0051563D `F7D0` `not eax`

0051563F `B8 6D69656C` `mov eax,6C65696D`

00515644 `91` `xchg eax,ecx`

00515645 `B9 DEC0ADDE` `mov ecx,DEADC0DE`

0051564A `91` `xchg eax,ecx`

0051564B `F7D0` `not eax`

0051564D `58` `pop eax`

0051564E `F7D1` `not ecx`

00515650 `59` `pop ecx`

00515651 `9C` `pushfd`

00515652 `60` `pushad`

00515653 `33DB` `xor ebx,ebx`

00515655 `74 03` `je short Aspr.0051565A`

00515657 `EB 22` `jmp short Aspr.0051567B`

00515659 `EB 33` `jmp short Aspr.0051568E`

0051565B `DB` `???` ; 未知命令

0051565C `74 00` `je short Aspr.0051565E`

0051565E `EB 0D` `jmp short Aspr.0051566D`

00515660 `B8 EB0FB987` `mov eax,87B90FEB`

00515665 `C9` `leave`

00515666 `F9` `stc`

00515667 `34 90` `xor al,90`

00515669 `F9` `stc`

0051566A `74 05` `je short Aspr.00515671`

0051566C `EB 33` `jmp short Aspr.005156A1`

0051566E `C074F2 B8 87` `sal byte ptr ds:[edx+esi*8-48],87`

00515673 `C9` `leave`

00515674 `40` `inc eax`

00515675 `48` `dec eax`

00515676 `85C0` `test eax,eax`

00515678 `^ 75 DD` `jnz short Aspr.00515657`

0051567A `- E9 619D6692` `jmp 92B7F3E0`

0051567F `66:92` `xchg ax,dx`

00515681 `8BC0` `mov eax,eax`

00515683 `70 07` `jo short Aspr.0051568C`

00515685 `7C 03` `jl short Aspr.0051568A`

00515687 `EB 05` `jmp short Aspr.0051568E`

00515689 `E8 74FBEBF9` `call FA3D5202`------> 垃圾指令结束

0051568E `8B85 68EEFFFF` `mov eax,dword ptr ss:[ebp-1198]`----> 前面的余数

00515694 `8B0C85 28075400` `mov ecx,dword ptr ds:[eax*4+540728]` ----> `table2`（跳转类型代号表）

0051569B `8B95 94EBFFFF` `mov edx,dword ptr ss:[ebp-146C]`

005156A1 `33C0` `xor eax,eax`

005156A3 `8A0411` `mov al,byte ptr ds:[ecx+edx]`----> 得到跳转类型代号

005156A6 `8985 78EBFFFF` `mov dword ptr ss:[ebp-1488],eax`

005156AC `8B85 78EBFFFF` `mov eax,dword ptr ss:[ebp-1488]`

005156B2 `99` `cdq`

005156B3 `83E2 0F` `and edx,0F`

005156B6 `03C2` `add eax,edx`

005156B8 `C1F8 04` `sar eax,4`

005156BB `8985 80EBFFFF` `mov dword ptr ss:[ebp-1480],eax`

005156C1 `8B8D 78EBFFFF` `mov ecx,dword ptr ss:[ebp-1488]`

005156C7 `81E1 0F000080` `and ecx,8000000F`

005156CD `79 05` `jns short Aspr.005156D4`

005156CF `49` `dec ecx`

005156D0 `83C9 F0` `or ecx,FFFFFFF0`

005156D3 `41` `inc ecx`

005156D4 `898D 7CEBFFFF` `mov dword ptr ss:[ebp-1484],ecx`

005156DA `8B95 80EBFFFF` `mov edx,dword ptr ss:[ebp-1480]`

005156E0 `3B95 7CEBFFFF` `cmp edx,dword ptr ss:[ebp-1484]`

005156E6 `75 1B` `jnz short Aspr.00515703`

005156E8 `8B85 7CEBFFFF` `mov eax,dword ptr ss:[ebp-1484]`

005156EE `83C0 01` `add eax,1`

005156F1 `25 0F000080` `and eax,8000000F`

005156F6 `79 05` `jns short Aspr.005156FD`

005156F8 `48` `dec eax`

005156F9 `83C8 F0` `or eax,FFFFFFF0`

005156FC `40` `inc eax`

005156FD `8985 7CEBFFFF` `mov dword ptr ss:[ebp-1484],eax`

00515703 `8B8D 78EBFFFF` `mov ecx,dword ptr ss:[ebp-1488]`

00515709 `8B95 80EBFFFF` `mov edx,dword ptr ss:[ebp-1480]`

0051570F `8B048D C8FE5300` `mov eax,dword ptr ds:[ecx*4+53FEC8]`

00515716 `330495 6CA25300` `xor eax,dword ptr ds:[edx*4+53A26C]`

0051571D `8B8D 7CEBFFFF` `mov ecx,dword ptr ss:[ebp-1484]`

00515723 `33048D 6CA25300` `xor eax,dword ptr ds:[ecx*4+53A26C]`

0051572A `8985 88EBFFFF` `mov dword ptr ss:[ebp-1478],eax`

00515730 `8B95 5CECFFFF` `mov edx,dword ptr ss:[ebp-13A4]`----> 子进程 `context` 的 `flag` 寄存器内容

00515736 `81E2 D70F0000` `and edx,0FD7`



```
0051573C     52                              push edx

0051573D     8B85 78EBFFFF                   mov eax,dword ptr ss:[ebp-1488] ----->跳转类型代号

00515743     0FBE88 30E75300                 movsx ecx,byte ptr ds:[eax+53E730]

0051574A     FF148D 38E85300                call dword ptr ds:[ecx*4+53E838]

00515751     83C4 04                          add esp,4

00515754     8985 8CEBFFFF                   mov dword ptr ss:[ebp-1474],eax

0051575A     8B95 48ECFFFF                   mov edx,dword ptr ss:[ebp-13B8]----> 子进程 context 的 ecx

内容

00515760     52                              push edx

00515761     8B85 8CEBFFFF                   mov eax,dword ptr ss:[ebp-1474]

00515767     50                              push eax

00515768     FF95 88EBFFFF                   call dword ptr ss:[ebp-1478]

0051576E     83C4 08                          add esp,8

00515771     50                              push eax

00515772     8B8D 78EBFFFF                   mov ecx,dword ptr ss:[ebp-1488]

00515778     0FBE91 30E75300                 movsx edx,byte ptr ds:[ecx+53E730]

0051577F     FF1495 78E85300                call dword ptr ds:[edx*4+53E878]

00515786     83C4 04                          add esp,4

00515789     8985 84EBFFFF                   mov dword ptr ss:[ebp-147C],eax

0051578F     8B85 84EBFFFF                   mov eax,dword ptr ss:[ebp-147C]

00515795     83E0 01                          and eax,1

00515798     85C0                             test eax,eax---->//经过若干次复杂计算最后得到子进程在 cc

处是跳呢(eax=1),还是不跳(eax=0)

0051579A     0F84 AE000000                    je Aspr.0051584E

005157A0     60                              pushad------>垃圾指令开始

005157A1     33C0                             xor eax,eax

005157A3     75 02                            jnz short Aspr.005157A7

005157A5     EB 15                            jmp short Aspr.005157BC

005157A7     EB 33                            jmp short Aspr.005157DC

005157A9     C075 18 7A                       sal byte ptr ss:[ebp+18],7A

005157AD     0C 70                            or al,70

005157AF     0E                              push cs

005157B0     EB 0D                            jmp short Aspr.005157BF

005157B2     E8 720E79F1                       call F1CA6629

005157B7     FF15 00790974                     call dword ptr ds:[74097900]

005157BD     F0:EB 87                         lock jmp short Aspr.00515747                        ; 锁

定前缀是不允许的

005157C0     DB7A F0                          fstp tbyte ptr ds:[edx-10]

005157C3     A0 33618B8D                       mov al,byte ptr ds:[8D8B6133]

005157C8     68 EEFFFF8B                       push 8BFFFFEE

005157CD     0C 8D                            or al,8D

005157CF     48                               dec eax

005157D0     06                               push es

005157D1     54                               push esp

005157D2     008B 8594EBFF                     add byte ptr ds:[ebx+FFEB9485],cl

005157D8     FF33                              push dword ptr ds:[ebx]

005157DA     D2BE 10000000                     sar byte ptr ds:[esi+10],cl

005157E0     F7F6                              div esi   ------>垃圾指令开始

005157E2     8B85 94EBFFFF                    mov eax,dword ptr ss:[ebp-146C]

005157E8     8B0C81                            mov ecx,dword ptr ds:[ecx+eax*4]

005157EB     338C95 90EEFFFF                  xor ecx,dword ptr ss:[ebp+edx*4-1170]    //得到跳转量

005157F2     8B95 54ECFFFF                    mov edx,dword ptr ss:[ebp-13AC]

005157F8     03D1                             add edx,ecx

005157FA     8995 54ECFFFF                    mov dword ptr ss:[ebp-13AC],edx     //重新设置 context

00515800     51                              push ecx------>垃圾指令开始

00515801     0FC9                              bswap ecx

00515803     F7D1                              not ecx

00515805     50                               push eax

00515806     F7D0                              not eax

00515808     B8 6D69656C                       mov eax,6C65696D

0051580D     91                               xchg eax,ecx

0051580E     B9 DEC0ADDE                        mov ecx,DEADC0DE

00515813     91                               xchg eax,ecx

00515814     F7D0                              not eax

00515816     58                               pop eax

00515817     F7D1                              not ecx

00515819     59                               pop ecx

0051581A     9C                               pushfd

0051581B     60                               pushad

0051581C     33DB                              xor ebx,ebx

0051581E     74 03                            je short Aspr.00515823

00515820     EB 22                            jmp short Aspr.00515844

00515822     EB 33                            jmp short Aspr.00515857

00515824     DB                               ???                                           ; 未

知命令

00515825     74 00                            je short Aspr.00515827

00515827     EB 0D                            jmp short Aspr.00515836

00515829     B8 EB0FB987                        mov eax,87B90FEB

0051582E     C9                               leave

0051582F     F9                               stc

00515830     34 90                            xor al,90

00515832     F9                               stc

00515833     74 05                            je short Aspr.0051583A

00515835     EB 33                            jmp short Aspr.0051586A

00515837     C074F2 B8 87                       sal byte ptr ds:[edx+esi*8-48],87

0051583C     C9                               leave

0051583D     40                               inc eax

0051583E     48                               dec eax

0051583F     85C0                             test eax,eax

00515841   ^ 75 DD                            jnz short Aspr.00515820

00515843   - E9 619D6692                        jmp 92B7F5A9

00515848     66:92                            xchg ax,dx

0051584A     8BC0                             mov eax,eax

0051584C     EB 75                            jmp short Aspr.005158C3

0051584E     70 07                            jo short Aspr.00515857

00515850     7C 03                            jl short Aspr.00515855

00515852     EB 05                            jmp short Aspr.00515859

00515854     E8 74FBEBF9                       call FA3D53CD------>垃圾指令结束

00515859     8B85 68EEFFFF                    mov eax,dword ptr ss:[ebp-1198]

0051585F     8B0C85 70075400                  mov ecx,dword ptr ds:[eax*4+540770]

00515866     8B95 94EBFFFF                    mov edx,dword ptr ss:[ebp-146C]

0051586C     33C0                             xor eax,eax

0051586E     8A0411                            mov al,byte ptr ds:[ecx+edx]    //得到下一条指令的距离

00515871     8B8D 54ECFFFF                    mov ecx,dword ptr ss:[ebp-13AC]

00515877     03C8                             add ecx,eax

00515879     898D 54ECFFFF                    mov dword ptr ss:[ebp-13AC],ecx  //重新设置 context

0051587F     50                              push eax------>垃圾指令开始

00515880     F7D0                              not eax

00515882     0FC8                              bswap eax

00515884     58                               pop eax

00515885     73 00                            jnb short Aspr.00515887

00515887     9C                               pushfd

00515888     60                               pushad

00515889     EB 2B                            jmp short Aspr.005158B6

0051588B     D270 0E                           sal byte ptr ds:[eax+E],cl

0051588E     EB 1D                            jmp short Aspr.005158AD

00515890     33C9                             xor ecx,ecx

00515892     74 00                            je short Aspr.00515894

00515894     EB 10                            jmp short Aspr.005158A6

00515896     F9                               stc

00515897     83F1 03                           xor ecx,3

0051589A   ^ 74 90                            je short Aspr.0051582C

0051589C     EB 05                            jmp short Aspr.005158A3

0051589E     B9 EB1074EF                        mov ecx,EF7410EB

005158A3   ^ 73 FA                            jnb short Aspr.0051589F
```



# 汇编代码段

```
005158A5     F2:                     prefix repne:  
005158A6     1ADB                    sbb bl,bl  
005158A8     33DB                    xor ebx,ebx  
005158AA   ^ EB EA                   jmp short Aspr.00515896  
005158AC   - E9 EBE1EBDD            jmp DE3D3A9C  
005158B1     F2:                     prefix repne:  
005158B2     1C DB                   sbb al,0DB  
005158B4     EB 03                   jmp short Aspr.005158B9  
005158B6   ^ EB D2                   jmp short Aspr.0051588A  
005158B8     B8 619D0FC8             mov eax,C80F9D61  
005158BD     F7D1                    not ecx  
005158BF     0FC8                    bswap eax  
005158C1     F7D1                    not ecx  
005158C3     51                      push ecx  
005158C4     0FC9                    bswap ecx  
005158C6     F7D1                    not ecx  
005158C8     50                      push eax  
005158C9     F7D0                    not eax  
005158CB     B8 6D69656C             mov eax,6C65696D  
005158D0     91                      xchg eax,ecx  
005158D1     B9 DEC0ADDE             mov ecx,DEADC0DE  
005158D6     91                      xchg eax,ecx  
005158D7     F7D0                    not eax  
005158D9     58                      pop eax  
005158DA     F7D1                    not ecx  
005158DC     59                      pop ecx  
005158DD     9C                      pushfd  
005158DE     60                      pushad  
005158DF     33DB                    xor ebx,ebx  
005158E1     74 03                   je short Aspr.005158E6  
005158E3     EB 22                   jmp short Aspr.00515907  
005158E5     EB 33                   jmp short Aspr.0051591A  
005158E7     DB                      ???                                       ; 未知命令  
005158E8     74 00                   je short Aspr.005158EA  
005158EA     EB 0D                   jmp short Aspr.005158F9  
005158EC     B8 EB0FB987             mov eax,87B90FEB  
005158F1     C9                      leave  
005158F2     F9                      stc  
005158F3     34 90                   xor al,90  
005158F5     F9                      stc  
005158F6     74 05                   je short Aspr.005158FD  
005158F8     EB 33                   jmp short Aspr.0051592D  
005158FA     C074F2 B8 87            sal byte ptr ds:\[edx+esi\*8-48\],87  
005158FF     C9                      leave  
00515900     40                      inc eax  
00515901     48                      dec eax  
00515902     85C0                    test eax,eax  
00515904   ^ 75 DD                   jnz short Aspr.005158E3  
00515906   - E9 619D6692             jmp 92B7F66C  
0051590B     66:92                   xchg ax,dx  
0051590D     8BC0                    mov eax,eax------>垃圾指令结束  
0051590F     8D95 9CEBFFFF           lea edx,dword ptr ss:\[ebp\-1464\]  
00515915     52                      push edx  
00515916     8B85 70EEFFFF           mov eax,dword ptr ss:\[ebp\-1190\]  
0051591C     50                      push eax  
0051591D     FF15 A0A05300           call dword ptr ds:\[<&KERNEL32.SetThreadContext>\]   ; kernel32.SetThreadContext  
00515923     60                      pushad  
00515924     33C0                    xor eax,eax  
00515926     75 02                   jnz short Aspr.0051592A  
00515928     EB 15                   jmp short Aspr.0051593F  
0051592A     EB 33                   jmp short Aspr.0051595F  
0051592C     C075 18 7A              sal byte ptr ss:\[ebp+18\],7A  
00515930     0C 70                   or al,70  
00515932     0E                      push cs  
00515933     EB 0D                   jmp short Aspr.00515942  
```

## 修改后的代码

**开始 1**

```
005154CF     90                      nop  
005154D0     90                      nop  
005154D1     90                      nop  
005154D2     90                      nop  
005154D3     90                      nop  
005154D4     90                      nop  
005154D5     90                      nop  
005154D6     90                      nop  
005154D7     90                      nop  
005154D8     90                      nop  
005154D9     90                      nop  
005154DA     90                      nop  
005154DB     90                      nop  
005154DC     90                      nop  
005154DD     90                      nop  
005154DE     90                      nop  
005154DF     90                      nop  
005154E0     90                      nop  
005154E1     A1 00404000             mov eax,dword ptr ds:\[404000\]   ---》\[404000\]中是原始 int3 表计数器  
005154E6     3E:8B0485 00104000      mov eax,dword ptr ds:\[eax\*4+401000\]  ---》401000 开始放置 int3 原始表  
005154EE     85C0                    test eax,eax  
005154F0     0F84 96030000           je Aspr.0051588C  ----》//int3 原始表到最后？是则结束。  
005154F6     8D95 54ECFFFF           lea edx,dword ptr ss:\[ebp\-13AC\]  
005154FC     90                      nop  
005154FD     8902                    mov dword ptr ds:\[edx\],eax  
005154FF     FF05 00404000           inc dword ptr ds:\[404000\]---》\[404000\]中是原始 int3 表计数器 ，加 1  
00515505     90                      nop  
00515506     90                      nop  
00515507     90                      nop  
00515508     90                      nop  
00515509     90                      nop  
0051550A     90                      nop  
0051550B     90                      nop  
0051550C     90                      nop  
0051550D     90                      nop  
0051550E     90                      nop  
0051550F     90                      nop  
00515510     90                      nop  
00515511     90                      nop  
00515512     90                      nop  
```

**结束 1**

```
00515513     C785 98EBFFFF 00000000    mov dword ptr ss:\[ebp\-1468\],0  
0051551D     6A FF                   push -1  
0051551F     6A 04                   push 4  
00515521     8D95 54ECFFFF           lea edx,dword ptr ss:\[ebp\-13AC\]  
00515527     52                      push edx  
00515528     E8 638AFEFF             call Aspr.004FDF90  
0051552D     83C4 0C                 add esp,0C  
00515530     8985 6CEEFFFF           mov dword ptr ss:\[ebp\-1194\],eax  
00515536     8B85 6CEEFFFF           mov eax,dword ptr ss:\[ebp\-1194\]  
0051553C     33D2                    xor edx,edx  
0051553E     B9 10000000             mov ecx,10  
00515543     F7F1                    div ecx  
00515545     8995 68EEFFFF           mov dword ptr ss:\[ebp\-1198\],edx  
0051554B     8B95 54ECFFFF           mov edx,dword ptr ss:\[ebp\-13AC\]  
00515551     52                      push edx  
00515552     8B85 68EEFFFF           mov eax,dword ptr ss:\[ebp\-1198\]  
00515558     FF1485 38E85300         call dword ptr ds:\[eax\*4+53E838\]  
0051555F     83C4 04                 add esp,4  
00515562     8985 98EBFFFF           mov dword ptr ss:\[ebp\-1468\],eax  
00515568     C785 94EBFFFF 00000000    mov dword ptr ss:\[ebp\-146C\],0  
00515572     8B8D 68EEFFFF           mov ecx,dword ptr ss:\[ebp\-1198\]  
00515578     8B148D E8065400         mov edx,dword ptr ds:\[ecx\*4+5406E8\]  
0051557F     8995 74EEFFFF           mov dword ptr ss:\[ebp\-118C\],edx  
00515585     8B85 94EBFFFF           mov eax,dword ptr ss:\[ebp\-146C\]  
0051558B     3B85 74EEFFFF           cmp eax,dword ptr ss:\[ebp\-118C\]  
```



```
00515591     7D 5C                       jge short Aspr.005155EF
00515593     8B85 74EEFFFF              mov eax,dword ptr ss:[ebp-118C]
00515599     2B85 94EBFFFF              sub eax,dword ptr ss:[ebp-146C]
0051559F     99                         cdq
005155A0     2BC2                       sub eax,edx
005155A2     D1F8                       sar eax,1
005155A4     8B8D 94EBFFFF              mov ecx,dword ptr ss:[ebp-146C]
005155AA     03C8                       add ecx,eax
005155AC     898D 90EBFFFF              mov dword ptr ss:[ebp-1470],ecx
005155B2     8B95 68EEFFFF              mov edx,dword ptr ss:[ebp-1198]
005155B8     8B0495 88065400            mov eax,dword ptr ds:[edx*4+540688]
005155BF     8B8D 90EBFFFF              mov ecx,dword ptr ss:[ebp-1470]
005155C5     8B95 98EBFFFF              mov edx,dword ptr ss:[ebp-1468]
005155CB     3B1488                     cmp edx,dword ptr ds:[eax+ecx*4]
005155CE     76 11                      jbe short Aspr.005155E1
005155D0     8B85 90EBFFFF              mov eax,dword ptr ss:[ebp-1470]
005155D6     83C0 01                    add eax,1
005155D9     8985 94EBFFFF              mov dword ptr ss:[ebp-146C],eax
005155DF     EB 0C                      jmp short Aspr.005155ED
005155E1     8B8D 90EBFFFF              mov ecx,dword ptr ss:[ebp-1470]
005155E7     898D 74EEFFFF              mov dword ptr ss:[ebp-118C],ecx
005155ED   ^ EB 96                      jmp short Aspr.00515585
```

！！！！！！！！！！！！！！！！！！！！！！！！！！开始 2

```
005155EF     90                         nop
005155F0     90                         nop
005155F1     8B95 68EEFFFF              mov edx,dword ptr ss:[ebp-1198]-----》//这段代码我把它
的位置朝前挪了，修改了跳转量
005155F7     8B0495 88065400            mov eax,dword ptr ds:[edx*4+540688]
005155FE     8B8D 94EBFFFF              mov ecx,dword ptr ss:[ebp-146C]
00515604     8B1488                     mov edx,dword ptr ds:[eax+ecx*4]
00515607     3B95 98EBFFFF              cmp edx,dword ptr ss:[ebp-1468]
0051560D   ^ 0F85 CEFEFFFF              jnz Aspr.005154E1
00515613     90                         nop
00515614     90                         nop
00515615     90                         nop
00515616     A1 04404000                mov eax,dword ptr ds:[404004]---》//符合条件的 int3 计数器
0051561B     8D95 54ECFFFF              lea edx,dword ptr ss:[ebp-13AC]
00515621     8B12                       mov edx,dword ptr ds:[edx]
00515623     3E:891485 00604000         mov dword ptr ds:[eax*4+406000],edx----》//符合条件的
int3 地址从 406000 开始放
0051562B     90                         nop
0051562C     90                         nop\----》//下面这段代码是为了得到跳转类型而增加的，目的
是得到 00515897 处自编函数需要的东西
0051562D     B8 0C404000                mov eax,Aspr.0040400C----》//0040400C 中是计数器
00515632     33C9                       xor ecx,ecx
00515634     8908                       mov dword ptr ds:[eax],ecx----》//计数器置 0
00515636     8948 04                    mov dword ptr ds:[eax+4],ecx----》//context 的 ecx=0
00515639     8948 0C                    mov dword ptr ds:[eax+C],ecx----》 //context 的 flag=0
0051563C     41                         inc ecx
0051563D     8948 08                    mov dword ptr ds:[eax+8],ecx----》//context 的 ecx=1
00515640     8948 10                    mov dword ptr ds:[eax+10],ecx----》//flag 的 cf=1
00515643     C740 14 04000000           mov dword ptr ds:[eax+14],4----》//pf=1
0051564A     C740 18 40000000           mov dword ptr ds:[eax+18],40----》//zf=1
00515651     C740 1C 80000000           mov dword ptr ds:[eax+1C],80----》//sf=1
00515658     C740 20 00080000           mov dword ptr ds:[eax+20],800----》//of=1
0051565F     8D85 9CEBFFFF              lea eax,dword ptr ss:[ebp-1464]
00515665     8B0D 0C404000              mov ecx,dword ptr ds:[40400C]
0051566B     3E:8B148D 10404000         mov edx,dword ptr ds:[ecx*4+404010]
00515673     83F9 01                    cmp ecx,1
00515676     7F 08                      jg short Aspr.00515680
00515678     8990 AC000000              mov dword ptr ds:[eax+AC],edx
0051567E     EB 06                      jmp short Aspr.00515686
00515680     8990 C0000000              mov dword ptr ds:[eax+C0],edx
00515686     90                         nop
00515687     90                         nop
00515688     90                         nop
00515689     90                         nop
0051568A     90                         nop
0051568B     90                         nop
0051568C     90                         nop
0051568D     90                         nop
```

！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！结束 2

```
0051568E     8B85 68EEFFFF              mov eax,dword ptr ss:[ebp-1198]
00515694     8B0C85 28075400            mov ecx,dword ptr ds:[eax*4+540728]
0051569B     8B95 94EBFFFF              mov edx,dword ptr ss:[ebp-146C]
005156A1     33C0                       xor eax,eax
005156A3     8A0411                     mov al,byte ptr ds:[ecx+edx]
005156A6     8985 78EBFFFF              mov dword ptr ss:[ebp-1488],eax
005156AC     8B85 78EBFFFF              mov eax,dword ptr ss:[ebp-1488]
005156B2     99                         cdq
005156B3     83E2 0F                    and edx,0F
005156B6     03C2                       add eax,edx
005156B8     C1F8 04                    sar eax,4
005156BB     8985 80EBFFFF              mov dword ptr ss:[ebp-1480],eax
005156C1     8B8D 78EBFFFF              mov ecx,dword ptr ss:[ebp-1488]
005156C7     81E1 0F000080              and ecx,8000000F
005156CD     79 05                      jns short Aspr.005156D4
005156CF     49                         dec ecx
005156D0     83C9 F0                    or ecx,FFFFFFF0
005156D3     41                         inc ecx
005156D4     898D 7CEBFFFF              mov dword ptr ss:[ebp-1484],ecx
005156DA     8B95 80EBFFFF              mov edx,dword ptr ss:[ebp-1480]
005156E0     3B95 7CEBFFFF              cmp edx,dword ptr ss:[ebp-1484]
005156E6     75 1B                      jnz short Aspr.00515703
005156E8     8B85 7CEBFFFF              mov eax,dword ptr ss:[ebp-1484]
005156EE     83C0 01                    add eax,1
005156F1     25 0F000080                and eax,8000000F
005156F6     79 05                      jns short Aspr.005156FD
005156F8     48                         dec eax
005156F9     83C8 F0                    or eax,FFFFFFF0
005156FC     40                         inc eax
005156FD     8985 7CEBFFFF              mov dword ptr ss:[ebp-1484],eax
00515703     8B8D 78EBFFFF              mov ecx,dword ptr ss:[ebp-1488]
00515709     8B95 80EBFFFF              mov edx,dword ptr ss:[ebp-1480]
0051570F     8B048D C8FE5300            mov eax,dword ptr ds:[ecx*4+53FEC8]
00515716     330495 6CA25300            xor eax,dword ptr ds:[edx*4+53A26C]
0051571D     8B8D 7CEBFFFF              mov ecx,dword ptr ss:[ebp-1484]
00515723     33048D 6CA25300            xor eax,dword ptr ds:[ecx*4+53A26C]
0051572A     8985 88EBFFFF              mov dword ptr ss:[ebp-1478],eax
00515730     8B95 5CECFFFF              mov edx,dword ptr ss:[ebp-13A4]
00515736     81E2 D70F0000              and edx,0FD7
0051573C     52                         push edx
0051573D     8B85 78EBFFFF              mov eax,dword ptr ss:[ebp-1488]
00515743     0FBE88 30E75300            movsx ecx,byte ptr ds:[eax+53E730]
0051574A     FF148D 38E85300            call dword ptr ds:[ecx*4+53E838]
00515751     83C4 04                    add esp,4
```



```
00515754      8985 8CEBFFFF           mov dword ptr ss:[ebp-1474],eax
0051575A      8B95 48ECFFFF           mov edx,dword ptr ss:[ebp-13B8]
00515760      52                      push edx
00515761      8B85 8CEBFFFF           mov eax,dword ptr ss:[ebp-1474]
00515767      50                      push eax
00515768      FF95 88EBFFFF           call dword ptr ss:[ebp-1478]
0051576E      83C4 08                 add esp,8
00515771      50                      push eax
00515772      8B8D 78EBFFFF           mov ecx,dword ptr ss:[ebp-1488]
00515778      0FBE91 30E75300         movsx edx,byte ptr ds:[ecx+53E730]
0051577F      FF1495 78E85300         call dword ptr ds:[edx*4+53E878]
00515786      83C4 04                 add esp,4
00515789      8985 84EBFFFF           mov dword ptr ss:[ebp-147C],eax
0051578F      8B85 84EBFFFF           mov eax,dword ptr ss:[ebp-147C]
00515795      83E0 01                 and eax,1
```

！！！！！！！！！！！！！！！！！！！！！！！！！！开始 3

```
00515798      90                      nop
00515799      90                      nop
0051579A      8B0D 0C404000           mov ecx,dword ptr ds:[40400C]
005157A0      3E:89048D 10404000      mov dword ptr ds:[ecx*4+404010],eax---》//得到的 eax 从
404010 开始放
005157A8      41                      inc ecx
005157A9      890D 0C404000           mov dword ptr ds:[40400C],ecx-----》//计数器+1
005157AF      83F9 07                 cmp ecx,7
005157B2     ^ 0F8E A7FEFFFF          jle Aspr.0051565F-----》 //循环 8 次
005157B8      90                      nop
005157B9      90                      nop
005157BA      60                      pushad
005157BB      9C                      pushfd
005157BC      E8 D6000000             call Aspr.00515897----》//pyzpyz 大侠写的判断跳转类型的
函数
005157C1      9D                      popfd
005157C2      61                      popad
005157C3      90                      nop
005157C4      90                      nop
005157C5      90                      nop
```

！！！！！！！！！！！！！！！！！！！！！！！！！！结束 3

```
005157C6      8B8D 68EEFFFF           mov ecx,dword ptr ss:[ebp-1198]
005157CC      8B0C8D 48065400         mov ecx,dword ptr ds:[ecx*4+540648]
005157D3      8B85 94EBFFFF           mov eax,dword ptr ss:[ebp-146C]
005157D9      33D2                    xor edx,edx
005157DB      BE 10000000             mov esi,10
005157E0      F7F6                    div esi
005157E2      8B85 94EBFFFF           mov eax,dword ptr ss:[ebp-146C]
005157E8      8B0C81                  mov ecx,dword ptr ds:[ecx+eax*4]
005157EB      338C95 90EEFFFF         xor ecx,dword ptr ss:[ebp+edx*4-1170]
```

！！！！！！！！！！！！！！！！！！！！！！！！！！开始 4

```
005157F2      90                      nop
005157F3      90                      nop
005157F4      A1 04404000             mov eax,dword ptr ds:[404004]
005157F9      3E:890C85 00624000      mov dword ptr ds:[eax*4+406200],ecx----》；跳转量从
406200 开始放
00515801      90                      nop
00515802      90                      nop
00515803      90                      nop
00515804      90                      nop
00515805      90                      nop
00515806      90                      nop
00515807      90                      nop
00515808      90                      nop
00515809      90                      nop
0051580A      90                      nop
0051580B      90                      nop
0051580C      90                      nop
0051580D      90                      nop
0051580E      90                      nop
0051580F      90                      nop
00515810      90                      nop
00515811      90                      nop
00515812      90                      nop
00515813      90                      nop
00515814      90                      nop
00515815      90                      nop
00515816      90                      nop
00515817      90                      nop
00515818      90                      nop
00515819      90                      nop
0051581A      90                      nop
0051581B      90                      nop
0051581C      90                      nop
0051581D      90                      nop
0051581E      90                      nop
0051581F      90                      nop
00515820      90                      nop
00515821      90                      nop
00515822      90                      nop
00515823      90                      nop
00515824      90                      nop
00515825      90                      nop
00515826      90                      nop
00515827      90                      nop
00515828      90                      nop
00515829      90                      nop
0051582A      90                      nop
0051582B      90                      nop
0051582C      90                      nop
0051582D      90                      nop
0051582E      90                      nop
0051582F      90                      nop
00515830      90                      nop
00515831      90                      nop
00515832      90                      nop
00515833      90                      nop
00515834      90                      nop
00515835      90                      nop
00515836      90                      nop
00515837      90                      nop
00515838      90                      nop
00515839      90                      nop
0051583A      90                      nop
0051583B      90                      nop
0051583C      90                      nop
0051583D      90                      nop
0051583E      90                      nop
0051583F      90                      nop
00515840      90                      nop
00515841      90                      nop
00515842      90                      nop
00515843      90                      nop
00515844      90                      nop
00515845      90                      nop
00515846      90                      nop
00515847      90                      nop
00515848      90                      nop
00515849      90                      nop
0051584A      90                      nop
0051584B      90                      nop
0051584C      90                      nop
0051584D      90                      nop
0051584E      90                      nop
0051584F      90                      nop
00515850      90                      nop
00515851      90                      nop
00515852      90                      nop
00515853      90                      nop
00515854      90                      nop
00515855      90                      nop
00515856      90                      nop
00515857      90                      nop
00515858      90                      nop
```

！！！！！！！！！！！！！！！！！！！！！！！！！！结束 4

```
00515859      8B85 68EEFFFF           mov eax,dword ptr ss:[ebp-1198]
0051585F      8B0C85 70075400         mov ecx,dword ptr ds:[eax*4+540770]
00515866      8B95 94EBFFFF           mov edx,dword ptr ss:[ebp-146C]
0051586C      33C0                    xor eax,eax
0051586E      8A0411                  mov al,byte ptr ds:[ecx+edx]
```

！！！！！！！！！！！！！！！！！！！！！！！！！！开始 5

```
00515871      90                      nop
00515872      90                      nop
00515873      8B0D 04404000           mov ecx,dword ptr ds:[404004]
```



### 5. 结束

```
00515879      8881 00634000                   mov byte ptr ds:[ecx+406300],al----》不跳，下条指令偏移
量从 406300 开始放       
0051587F      90                              nop
00515880      90                              nop
00515881      FF05 04404000                   inc dword ptr ds:[404004]
00515887      E9 55FCFFFF                     jmp Aspr.005154E1
0051588C      90                              nop
0051588D      90                              nop
0051588E      EB 1D                           jmp short Aspr.005158AD
00515890      33C9                            xor ecx,ecx
00515892      74 00                           je short Aspr.00515894
00515894      EB 10                           jmp short Aspr.005158A6
00515896      F9                              stc
```

### 6. 开始

```
00515897      8B3D 04404000                   mov edi,dword ptr ds:[404004]----》 //有效 int3 的计数器
0051589D      81C7 00614000                   add edi,Aspr.00406100 ---》//跳转类型从 00406100 开始放  
005158A3      BA 0C404000                     mov edx,Aspr.0040400C
005158A8      8B42 04                           mov eax,dword ptr ds:[edx+4]
005158AB      8B5A 08                           mov ebx,dword ptr ds:[edx+8]
005158AE      33D8                              xor ebx,eax
005158B0      74 0F                             je short Aspr.005158C1
005158B2      90                                nop
005158B3      90                                nop
005158B4      C607 E3                           mov byte ptr ds:[edi],0E3
005158B7      E9 20010000                       jmp Aspr.005159DC
005158BC      90                                nop
005158BD      90                                nop
005158BE      90                                nop
005158BF      90                                nop
005158C0      90                                nop
005158C1      8B42 0C                           mov eax,dword ptr ds:[edx+C]
005158C4      8B5A 14                           mov ebx,dword ptr ds:[edx+14]
005158C7      33D8                              xor ebx,eax
005158C9      74 1D                             je short Aspr.005158E8
005158CB      85C0                              test eax,eax
005158CD      74 0C                             je short Aspr.005158DB
005158CF      90                                nop
005158D0      90                                nop
005158D1      C607 7B                           mov byte ptr ds:[edi],7B
005158D4      E9 03010000                       jmp Aspr.005159DC
005158D9      90                                nop
005158DA      90                                nop
005158DB      C607 7A                           mov byte ptr ds:[edi],7A
005158DE      E9 F9000000                       jmp Aspr.005159DC
005158E3      90                                nop
005158E4      90                                nop
005158E5      90                                nop
005158E6      90                                nop
005158E7      90                                nop
005158E8      8B5A 18                           mov ebx,dword ptr ds:[edx+18]
005158EB      33D8                              xor ebx,eax
005158ED      74 61                             je short Aspr.00515950
005158EF      90                                nop
005158F0      90                                nop
005158F1      8B5A 10                           mov ebx,dword ptr ds:[edx+10]
005158F4      33D8                              xor ebx,eax
005158F6      74 1A                             je short Aspr.00515912
005158F8      85C0                              test eax,eax
005158FA      74 0C                             je short Aspr.00515908
005158FC      90                                nop
005158FD      90                                nop
005158FE      C607 77                           mov byte ptr ds:[edi],77
00515901      E9 D6000000                       jmp Aspr.005159DC
00515906      90                                nop
00515907      90                                nop
00515908      C607 76                           mov byte ptr ds:[edi],76
0051590B      E9 CC000000                       jmp Aspr.005159DC
00515910      90                                nop
00515911      90                                nop
00515912      8B5A 1C                           mov ebx,dword ptr ds:[edx+1C]
00515915      33D8                              xor ebx,eax
00515917      74 1A                             je short Aspr.00515933
00515919      85C0                              test eax,eax
0051591B      74 0C                             je short Aspr.00515929
0051591D      90                                nop
0051591E      90                                nop
0051591F      C607 7F                           mov byte ptr ds:[edi],7F
00515922      E9 B5000000                       jmp Aspr.005159DC
00515927      90                                nop
00515928      90                                nop
00515929      C607 7E                           mov byte ptr ds:[edi],7E
0051592C      E9 AB000000                       jmp Aspr.005159DC
00515931      90                                nop
00515932      90                                nop
00515933      85C0                              test eax,eax
00515935      74 0C                             je short Aspr.00515943
00515937      90                                nop
00515938      90                                nop
00515939      C607 75                           mov byte ptr ds:[edi],75
0051593C      E9 9B000000                       jmp Aspr.005159DC
00515941      90                                nop
00515942      90                                nop
00515943      C607 74                           mov byte ptr ds:[edi],74
00515946      E9 91000000                       jmp Aspr.005159DC
0051594B      90                                nop
0051594C      90                                nop
0051594D      90                                nop
0051594E      90                                nop
0051594F      90                                nop
00515950      8B5A 1C                           mov ebx,dword ptr ds:[edx+1C]
00515953      33D8                              xor ebx,eax
00515955      74 34                             je short Aspr.0051598B
00515957      90                                nop
00515958      90                                nop
00515959      8B5A 20                           mov ebx,dword ptr ds:[edx+20]
0051595C      33D8                              xor ebx,eax
0051595E      74 14                             je short Aspr.00515974
00515960      85C0                              test eax,eax
00515962      74 09                             je short Aspr.0051596D
00515964      90                                nop
00515965      90                                nop
00515966      C607 7D                           mov byte ptr ds:[edi],7D
00515969      EB 71                             jmp short Aspr.005159DC
0051596B      90                                nop
0051596C      90                                nop
0051596D      C607 7C                           mov byte ptr ds:[edi],7C
00515970      EB 6A                             jmp short Aspr.005159DC
00515972      90                                nop
00515973      90                                nop
00515974      85C0                              test eax,eax
00515976      74 09                             je short Aspr.00515981
00515978      90                                nop
00515979      90                                nop
0051597A      C607 79                           mov byte ptr ds:[edi],79
0051597D      EB 5D                             jmp short Aspr.005159DC
0051597F      90                                nop
00515980      90                                nop
00515981      C607 78                           mov byte ptr ds:[edi],78
00515984      EB 56                             jmp short Aspr.005159DC
00515986      90                                nop
00515987      90                                nop
00515988      90                                nop
00515989      90                                nop
0051598A      90                                nop
0051598B      8B5A 20                           mov ebx,dword ptr ds:[edx+20]
0051598E      33D8                              xor ebx,eax
00515990      74 17                             je short Aspr.005159A9
```



00515992 `85C0` `test eax,eax`  

00515994 `74 09` `je short Aspr.0051599F`  

00515996 `90` `nop`  

00515997 `90` `nop`  

00515998 `C607 71` `mov byte ptr ds:[edi],71`  

0051599B `EB 3F` `jmp short Aspr.005159DC`  

0051599D `90` `nop`  

0051599E `90` `nop`  

0051599F `C607 70` `mov byte ptr ds:[edi],70`  

005159A2 `EB 38` `jmp short Aspr.005159DC`  

005159A4 `90` `nop`  

005159A5 `90` `nop`  

005159A6 `90` `nop`  

005159A7 `90` `nop`  

005159A8 `90` `nop`  

005159A9 `8B5A 10` `mov ebx,dword ptr ds:[edx+10]`  

005159AC `33D8` `xor ebx,eax`  

005159AE `74 17` `je short Aspr.005159C7`  

005159B0 `85C0` `test eax,eax`  

005159B2 `74 09` `je short Aspr.005159BD`  

005159B4 `90` `nop`  

005159B5 `90` `nop`  

005159B6 `C607 73` `mov byte ptr ds:[edi],73`  

005159B9 `EB 21` `jmp short Aspr.005159DC`  

005159BB `90` `nop`  

005159BC `90` `nop`  

005159BD `C607 72` `mov byte ptr ds:[edi],72`  

005159C0 `EB 1A` `jmp short Aspr.005159DC`  

005159C2 `90` `nop`  

005159C3 `90` `nop`  

005159C4 `90` `nop`  

005159C5 `90` `nop`  

005159C6 `90` `nop`  

005159C7 `85C0` `test eax,eax`  

005159C9 `74 0C` `je short Aspr.005159D7`  

005159CB `90` `nop`  

005159CC `90` `nop`  

005159CD `C607 EB` `mov byte ptr ds:[edi],0EB`  

005159D0 `EB 0A` `jmp short Aspr.005159DC`  

005159D2 `90` `nop`  

005159D3 `90` `nop`  

005159D4 `90` `nop`  

005159D5 `90` `nop`  

005159D6 `90` `nop`  

005159D7 `C607 90` `mov byte ptr ds:[edi],90`  

005159DA `90` `nop`  

005159DB `90` `nop`  

005159DC `C3` `retn`  

！！！！！！！！！！！！！！！！！！！！！！！！！！结束 6  

005159DD `^ 74 EF` `je short Aspr.005159CE`  

在 `0051588c` 处下断，`f9` 运行，得到  

`406000～406100` `int3` 地址表  

`406100～406200` 跳转类型表  

`406200～406300` 跳转量表  

`406300～406400` 跳转命令长度表  

复制下来，已备修复 `cc` 用。把 `dump` 出来的主文件，入口点改成 `004fa000`，用 `od` 加载。  

修改前：  

```
004FA000 z>    55                 push ebp
004FA001       8BEC               mov ebp,esp
004FA003       83EC 0C            sub esp,0C
004FA006       8B45 10            mov eax,dword ptr ss:[ebp+10]
004FA009       50                 push eax
004FA00A       E8 608C0200        call zy1.00522C6F
004FA00F       83C4 04            add esp,4
004FA012       8945 F8            mov dword ptr ss:[ebp-8],eax
004FA015       8B4D F8            mov ecx,dword ptr ss:[ebp-8]
004FA018       894D FC            mov dword ptr ss:[ebp-4],ecx
004FA01B       8B55 0C            mov edx,dword ptr ss:[ebp+C]
004FA01E       52                 push edx                             ; 
/Arg3
004FA01F       8B45 FC            mov eax,dword ptr ss:[ebp-4]         ; |
004FA022       50                 push eax                             ; 
|Arg2
004FA023       8B4D 08            mov ecx,dword ptr ss:[ebp+8]         ; |
004FA026       51                 push ecx                             ; 
|Arg1
004FA027       E8 5D000000        call zy1.004FA089                    ; 
\zy1.004FA089
004FA02C       83C4 0C            add esp,0C
004FA02F       8B55 0C            mov edx,dword ptr ss:[ebp+C]
004FA032       52                 push edx                             ; 
/Arg3
004FA033       8B45 08            mov eax,dword ptr ss:[ebp+8]         ; |
004FA036       50                 push eax                             ; 
|Arg2
004FA037       8B4D FC            mov ecx,dword ptr ss:[ebp-4]         ; |
004FA03A       51                 push ecx                             ; 
|Arg1
004FA03B       E8 66020000        call zy1.004FA2A6                    ; 
\zy1.004FA2A6
004FA040       83C4 0C            add esp,0C
004FA043       8B55 0C            mov edx,dword ptr ss:[ebp+C]
004FA046       8B02               mov eax,dword ptr ds:[edx]
004FA048       50                 push eax                             ; 
/Arg3
004FA049       8B4D FC            mov ecx,dword ptr ss:[ebp-4]         ; |
004FA04C       51                 push ecx                             ; 
|Arg2
004FA04D       8B55 08            mov edx,dword ptr ss:[ebp+8]         ; |
004FA050       52                 push edx                             ; 
|Arg1
004FA051       E8 40030000        call zy1.004FA396                    ; 
\zy1.004FA396
004FA056       83C4 0C            add esp,0C
004FA059       8B45 10            mov eax,dword ptr ss:[ebp+10]
004FA05C       50                 push eax                             ; 
/Arg4
004FA05D       8B4D 0C            mov ecx,dword ptr ss:[ebp+C]         ; |
004FA060       51                 push ecx                             ; 
|Arg3
004FA061       8B55 08            mov edx,dword ptr ss:[ebp+8]         ; |
004FA064       52                 push edx                             ; 
|Arg2
004FA065       8B45 FC            mov eax,dword ptr ss:[ebp-4]         ; |
004FA068       50                 push eax                             ; 
|Arg1
004FA069       E8 521A0000        call zy1.004FBAC0                    ; 
\zy1.004FBAC0
004FA06E       83C4 10            add esp,10
004FA071       8B4D FC            mov ecx,dword ptr ss:[ebp-4]
004FA074       894D F4            mov dword ptr ss:[ebp-C],ecx
004FA077       8B55 F4            mov edx,dword ptr ss:[ebp-C]
004FA07A       52                 push edx
004FA07B       E8 E48B0200        call zy1.00522C64
004FA080       83C4 04            add esp,4
004FA083       B0 01              mov al,1
004FA085       8BE5               mov esp,ebp
004FA087       5D                 pop ebp
004FA088       C3                 retn
004FA089       55                 push ebp
004FA08A       8BEC               mov ebp,esp
004FA08C   |.  83EC 28            sub esp,28
004FA08F   |.  8B45 08            mov eax,dword ptr ss:[ebp+8]
```

修改后：  

```
004FA000 z>    90                 nop
004FA001       90                 nop
004FA002       90                 nop
```



```text
004FA003       90                      nop  

004FA004       90                      nop  

004FA005       90                      nop  

004FA006       90                      nop  

004FA007       90                      nop  

004FA008       90                      nop  

004FA009       90                      nop  

004FA00A       90                      nop  

004FA00B       90                      nop  

004FA00C       90                      nop  

004FA00D       90                      nop  

004FA00E       90                      nop  

004FA00F       90                      nop  

004FA010       90                      nop  

004FA011       90                      nop  

004FA012       33C9                    xor ecx,ecx  

004FA014       3E:8B048D 00905200      mov eax,dword ptr ds:[ecx*4+529000]  

004FA01C       83F8 00                 cmp eax,0  

004FA01F       74 69                   je short zy1.004FA08A                  ; |  

004FA021       0FB691 00915200         movzx edx,byte ptr ds:[ecx+529100]  

004FA028       3E:8B1C8D 00925200      mov ebx,dword ptr ds:[ecx*4+529200]  

004FA030       0FB6B9 00935200         movzx edi,byte ptr ds:[ecx+529300]  

004FA037       48                      dec eax                                 ; |  

004FA038       41                      inc ecx  

004FA039       8138 CC558BEC           cmp dword ptr ds:[eax],EC8B55CC  

004FA03F     ^ 74 D3                   je short zy1.004FA014  

004FA041       8078 FF CC              cmp byte ptr ds:[eax-1],0CC  

004FA045     ^ 74 CD                   je short zy1.004FA014  

004FA047       8078 01 CC              cmp byte ptr ds:[eax+1],0CC  

004FA04B     ^ 74 C7                   je short zy1.004FA014  

004FA04D       90                      nop                                     ; |  

004FA04E       83FF 01                 cmp edi,1  

004FA051       75 09                   jnz short zy1.004FA05C                  ;  

\zy1.004FA396  

004FA053       8810                    mov byte ptr ds:[eax],dl  

004FA055       FECB                    dec bl  

004FA057       8858 01                 mov byte ptr ds:[eax+1],bl  

004FA05A       90                      nop  

004FA05B       90                      nop  

004FA05C       83FF 05                 cmp edi,5                               ;  

/Arg4  

004FA05F       75 11                   jnz short zy1.004FA072  

004FA061       C600 0F                 mov byte ptr ds:[eax],0F                ; |  

004FA064       80C2 10                 add dl,10                               ;  

|Arg2  

004FA067       8850 01                 mov byte ptr ds:[eax+1],dl  

004FA06A       83EB 05                 sub ebx,5  

004FA06D       8958 02                 mov dword ptr ds:[eax+2],ebx  

004FA070       90                      nop  

004FA071       90                      nop  

004FA072       83FF 04                 cmp edi,4  

004FA075       75 10                   jnz short zy1.004FA087  

004FA077       80FA EB                 cmp dl,0EB  

004FA07A       75 0B                   jnz short zy1.004FA087  

004FA07C       C600 E9                 mov byte ptr ds:[eax],0E9  

004FA07F       83EB 04                 sub ebx,4  

004FA082       8958 01                 mov dword ptr ds:[eax+1],ebx  

004FA085       90                      nop  

004FA086       90                      nop  

004FA087     ^ EB 8B                   jmp short zy1.004FA014  

004FA089       90                      nop  

004FA08A       90                      nop  

004FA08B       90                      nop  

004FA08C   .  83EC 28                 sub esp,28  
```

把 `406000`-`406400` 处的代码复制到 `00529000` 处，在 `4fa08a` 处下断，`f9` 运行，断下，下面用 `lordpe` 把主程序 `dump` 出来吧。可能体积较大，用 `lordpe` 重建一下就小了。

未完待续，请见（2）

在（1）中如果先修复 `cc`，再把 `Region033A0000-033C0000.dmp` 附加到程序中，就不会出现很大的文件了。其实修复 `cc` 时可以把要改的代码写成二进制文件，用复制加粘贴的方法还是很快的，以下是我用到的文件。

`406000`～`406100` `int3` 地址表

`406100`～`406200` 跳转类型表

`406200`～`406300` 跳转量表

`406300`～`406400` 跳转命令长度表

得到 `cc` 地址的代码如下：`005154cf`：

```text
90 90 90 90 90 90 90 90 90 90 90 90 90 8B 0D F0 8F 4D 00 3E 8D 34 8D 00 10 40 00 BF 00 10 40 00 
8B 85 54 EC FF FF F2 AF 74 16 90 90 90 90 89 06 FF 05 F0 8F 4D 00 BF 0E 00 00 00 BE 10 00 00 00 

90 90 90 90 
```

1.`005154cf`: `90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 A1 00 40 40 00 3E 8B 04 85 00 10 40 00 85 C0 0F 84 96 03 00 00 8D 95 54 EC FF FF 90 89 02 FF 05 00 40 40 00 90 90 90 90 90 90 90 90 90 90 90 90 90`

2.`005155ef`: `90 90 8B 95 68 EE FF FF 8B 04 95 88 06 54 00 8B 8D 94 EB FF FF 8B 14 88 3B 95 98 EB FF FF 0F 85 CE FE FF FF 90`

3.`00515614`: `90 90 A1 04 40 40 00 8D 95 54 EC FF FF 8B 12 3E 89 14 85 00 60 40 00 90 90 B8 0C 40 40 00 33 C9 89 08 89 48 04 89 48 0C 41 89 48 08 89 48 10 C7 40 14 04 00 00 00 C7 40 18 40 00 00 00 C7 40 1C 80 00 00 00 C7 40 20 00 08 00 00 8D 85 9C EB FF FF 8B 0D 0C 40 40 00 3E 8B 14 8D 10 40 40 00 83 F9 01 7F 08 89 90 AC 00 00 00 EB 06 89 90 C0 00 00 00 90 90 90 90 90 90 90 90`

4.`00515798`: `90 90 8B 0D 0C 40 40 00 3E 89 04 8D 10 40 40 00 41 89 0D 0C 40 40 00 83 F9 07 0F 8E A7 FE FF FF 90 90 60 9C E8 D6 00 00 00 9D 61 90 90 90`

5.`005157F2`: `90 90 A1 04 40 40 00 3E 89 0C 85 00 62 40 00 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90`

6.`00515871`: `90 90 8B 0D 04 40 40 00 88 81 00 63 40 00 90 90 FF 05 04 40 40 00 E9 55 FC FF FF 90 90`

7.`00515897`: `8B 3D 04 40 40 00 81 C7 00 61 40 00 BA 0C 40 40 00 8B 42 04 8B 5A 08 33 D8 74 0F 90 90 C6 07 E3 E9 20 01 00 00 90 90 90 90 90 8B 42 0C 8B 5A 14 33 D8 74 1D 85 C0 74 0C 90 90 C6 07 7B E9 03 01 00 00 90 90 C6 07 7A E9 F9 00 00 00 90 90 90 90 90 8B 5A 18 33 D8 74 61 90 90 8B 5A 10 33 D8 74 1A 85 C0 74 0C 90 90 C6 07 77 E9 D6 00 00 00 90 90 C6 07 76 E9 CC 00 00 00 90 90 8B 5A 1C 33 D8 74 1A 85 C0 74 0C 90 90 C6 07 7F E9 B5 00 00 00 90 90 C6 07 7E E9 AB 00 00 00 90 90 85 C0 74 0C 90 90 C6 07 75 E9 9B 00 00 00 90 90 C6 07 74 E9 91 00 00 00 90 90 90 90 90 8B 5A 1C 33 D8 74 34 90 90 8B 5A 20 33 D8 74 14 85 C0 74 09 90 90 C6 07 7D EB 71 90 90 C6 07 7C EB 6A 90 90 85 C0 74 09 90 90 C6 07 79 EB 5D 90 90 C6 07 78 EB 56 90 90 90 90 90 8B 5A 20 33 D8 74 17 85 C0 74 09 90 90 C6 07 71 EB 3F 90 90 C6 07 70 EB 38 90 90 90 90 90 8B 5A 10 33 D8 74 17 85 C0 74 09 90 90 C6 07 73 EB 21 90 90 C6 07 72 EB 1A 90 90 90 90 90 85 C0 74 0C 90 90 C6 07 EB EB 0A 90 90 90 90 90 C6 07 90 90 90 C3`

8.修复 `cc` `004fa000`: `90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 33 C9 3E 8B 04 8D 00 90 52 00 83 F8 00 74 69 0F B6 91 00 91 52 00 3E 8B 1C 8D 00 92 52 00 0F B6 B9 00 93 52 00 48 41 81 38 CC 55 8B EC 74 D3 80 78 FF CC 74 CD 80 78 01 CC 74 C7 90 83 FF 01 75 09 88 10 FE CB 88 58 01 90 90 83 FF 05 75 11 C6 00 0F 80 C2 10 88 50 01 83 EB 05 89 58 02 90 90 83 FF 04 75 10 80 FA EB 75 0B C6 00 E9 83 EB 04 89 58 01 90 90 EB 8B 90 90 90`

### （4）找到正确排序的 `iat`

1.使程序把自己当成子进程运行

用 `OD` 再一次载入未脱壳的程序 `Aspr.exe`，`bp DebugActiveProcess`，`F9` 断下。看堆栈窗口：

`0012BC8C` `00513F28` /CALL 到 `DebugActiveProcess` 来自 `Aspr.00513F22`

`0012BC90` `000005C0` \ProcessId = 5C0<=====子进程句柄

`0012BC94` `0012FF04`

打开另一个 `OD` 附加 `5C0` 这个子进程。然后 `ALT+F9` 返回程序:

`0052A000` `A>-` `EB FE` `jmp short Aspr.<ModuleEntryPoint>><====中断在此处，还原`

`0052A002` `0000` `add byte ptr ds:[eax],al`

`0052A004` `0000` `add byte ptr ds:[eax],al`

`0052A006` `5D` `pop ebp`

还原 `0052A000` 处代码代码，`EB FE` 改为 `60 E8`，接下来就要按照 `FLY` 等大虾们的方法了。

使程序把自己当成子进程运行，取消以前断点，下断：`BP OpenMutexA`



```
77E6074A k>   55                     push ebp    //断在这，看看堆栈

77E6074B     8BEC                     mov ebp,esp

77E6074D     51                       push ecx

77E6074E     51                       push ecx

77E6074F     837D 10 00               cmp dword ptr ss:[ebp+10],0

77E60753     56                       push esi

77E60754     0F84 EA500200          je kernel32.77E85844
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

BP `OpenMutexA` 时的堆栈：

```
0012D778    0050FBBD  /CALL 到 OpenMutexA 来自 Aspr.0050FBB7
0012D77C    001F0001  |Access = 1F0001
0012D780    00000000  |Inheritable = FALSE
0012D784    0012DDB8  \MutexName = "7E8::DAD92F6E24" --->注意 0012DDB8
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

Ctrl+G：`401000` 键入以下代码

```
00401000     60                  pushad

00401001     9C                  pushfd

00401002     68 B4FB1200         push 12DDB8 ★ 堆栈里看到的值

00401007     33C0                xor eax,eax

00401009     50                  push eax

0040100A     50                  push eax

0040100B     E8 B4B2A577         call kernel32.CreateMutexA

00401010     9D                  popfd

00401011     61                  popad

00401012     E9 33F7A577         jmp kernel32.OpenMutexA
```

在 `401000` 处新建起源，F9 运行，再次中断在 `OpenMutexA` 处

### 2. Magic Jump，避开 IAT 加密

取消以前断点，下断：`BP GetModuleHandleA+5`，在这里中断十几次，中间会出现无法处理异常的对话框，用

`SHIFT+F9` 跳过就行了；`Alt+F9` 返回程序。判断返回程序的时机：

☆ ☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

注意看 `BP GetModuleHandleA` 时的堆栈变化：

```
00128CE4    00D404D2  返回到 00D404D2 来自 kernel32.GetModuleHandleA
00128CE8    00D52D40  ASCII "kernel32.dll"
00128CEC    00D53F30  ASCII "VirtualAlloc"

00128CE4    00D404EF  返回到 00D404EF 来自 kernel32.GetModuleHandleA
00128CE8    00D52D40  ASCII "kernel32.dll"
00128CEC    00D53F24  ASCII "VirtualFree"
```

//再 F9 下去就会出现无法处理异常的对话框 , `SHIFT+F9` 跳过

```
00128A5C    00D29900  返回到 00D29900 来自 kernel32.GetModuleHandleA
00128A60    00128B98  ASCII "kernel32.dll"
00128A64    00000000
```

到这里时就用 `alt+F9` 返回主程序：

```
00D29900     8B0D 0881D500             mov ecx,dword ptr ds:[D58108]

00D29906     89040E                    mov dword ptr ds:[esi+ecx],eax

00D29909     A1 0881D500              mov eax,dword ptr ds:[D58108]

00D2990E     393C06                    cmp dword ptr ds:[esi+eax],edi

00D29911     75 16                    jnz short 00D29929

00D29913     8D85 B4FEFFFF            lea eax,dword ptr ss:[ebp-14C]

00D29919     50                        push eax

00D2991A     FF15 D4D0D400            call dword ptr ds:[D4D0D4]                 ; 

kernel32.LoadLibraryA

00D29920     8B0D 0881D500             mov ecx,dword ptr ds:[D58108]

00D29926     89040E                    mov dword ptr ds:[esi+ecx],eax

00D29929     A1 0881D500              mov eax,dword ptr ds:[D58108]

00D2992E     393C06                    cmp dword ptr ds:[esi+eax],edi

00D29931     0F84 AD000000             je 00D299E4--->//这里就是我们常说的 Magic Jump，把它改为

JMP 00D299E4,代码改为 90e9AD000000

00D29937     33C9                      xor ecx,ecx

00D29939     8B03                      mov eax,dword ptr ds:[ebx]
```

再 f9 运行，观察堆栈变化：

☆ ☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

当出现以下代码时：

```
00128A5C    00D29900  返回到 00D29900 来自 kernel32.GetModuleHandleA
00128A60    00128B98  ASCII "advapi32.dll"
00128CE8    00D45306  返回到 00D45306 来自 kernel32.GetModuleHandleA
00128CEC    00000000
```

到这里时就用 `alt+F9` 返回主程序：

```
00D45306     3985 B4D4FFFF             cmp dword ptr ss:[ebp-2B4C],eax             ; 

Aspr.00400000

00D4530C     75 0F                     jnz short 00D4531D

00D4530E     C785 B0D4FFFF 301BD500     mov dword ptr ss:[ebp-2B50],0D51B30

00D45318     E9 C4000000               jmp 00D453E1
```

向下一直到这：

```
00D45803     8B85 18D8FFFF             mov eax,dword ptr ss:[ebp-27E8]

00D45809     3B85 68D8FFFF             cmp eax,dword ptr ss:[ebp-2798]

00D4580F     73 1D                    jnb short 00D4582E

00D45811     8B85 18D8FFFF             mov eax,dword ptr ss:[ebp-27E8]->在这设断得到 iat 的位置

003d7940

00D45817     8B8D 7CD2FFFF             mov ecx,dword ptr ss:[ebp-2D84]

00D4581D     8908                      mov dword ptr ds:[eax],ecx

00D4581F     8B85 18D8FFFF             mov eax,dword ptr ss:[ebp-27E8]

00D45825     83C0 04                   add eax,4

00D45828     8985 18D8FFFF             mov dword ptr ss:[ebp-27E8],eax

00D4582E   ^ E9 E8FCFFFF               jmp 00D4551B
```

在 `00D45811` 处设断得到 `iat` 的位置，我的是 `3e7940` 是 `iat` 的首地址，看到很多 `00d2????` 之类的就是 `iat` 的位置，计 算一下得之 `VA=3e7940`，`SIZA=990`，通过对 `3e7940` 下硬件写入断点，第一次中断时得到完整得 `iat`，第二次就能 找到壳是怎样把 `iat` 给搅和乱了得代码，为了讲解方便，我直接在加密 `iat` 得地方设断。取消原来得断点。

```
00D459A2    /EB 0D                     jmp short 00D459B1--------》在这地方设断，f9 运行------》

dump 出来的代码 1

00D459A4    |8B85 64D1FFFF             mov eax,dword ptr ss:[ebp-2E9C]

00D459AA    |40                        inc eax

00D459AB    |8985 64D1FFFF             mov eax,dword ptr ss:[ebp-2E9C],eax

00D459B1    \8B85 64D1FFFF             mov eax,dword ptr ss:[ebp-2E9C]

00D459B7     3B85 6CD1FFFF             cmp eax,dword ptr ss:[ebp-2E94]--》1 注意加密的数目 020B

00D459BD     0F83 9A000000             jnb 00D45A5D

00D459C3     68 0D5EDF01               push 1DF5E0D

00D459C8     FFB5 68D1FFFF             push dword ptr ss:[ebp-2E98]----》2 注意[ebp-2E98]的值

00D459CE     8D8D 68D1FFFF             lea ecx,dword ptr ss:[ebp-2E98]

00D459D4     E8 98B6FDFF               call 00D21071----》f7 进入

00D459D9     40                         inc eax

00D459DA     33D2                       xor edx,edx

00D459DC     B9 00E1F505               mov ecx,5F5E100

00D459E1     F7F1                       div ecx

00D459E3     8995 68D1FFFF             mov dword ptr ss:[ebp-2E98],edx

00D459E9     8B85 68D1FFFF             mov eax,dword ptr ss:[ebp-2E98]

00D459EF     33D2                       xor edx,edx

00D459F1     B9 10270000                mov ecx,2710

00D459F6     F7F1                       div ecx

00D459F8     0FAF85 48D8FFFF           imul eax,dword ptr ss:[ebp-27B8]----》3 注意数目 264

00D459FF     33D2                       xor edx,edx

00D45A01     B9 10270000                mov ecx,2710

00D45A06     F7F1                       div ecx

00D45A08     8985 5CD1FFFF             mov dword ptr ss:[ebp-2EA4],eax

00D45A0E     8B85 ECD6FFFF             mov eax,dword ptr ss:[ebp-2914]

00D45A14     8B00                      mov eax,dword ptr ds:[eax]

00D45A16     8985 60D1FFFF             mov dword ptr ss:[ebp-2EA0],eax

00D45A1C     8B85 5CD1FFFF             mov eax,dword ptr ss:[ebp-2EA4]

00D45A22     8D0485 04000000           lea eax,dword ptr ds:[eax*4+4]

00D45A29     50                         push eax

00D45A2A     8B85 ECD6FFFF             mov eax,dword ptr ss:[ebp-2914]

00D45A30     83C0 04                   add eax,4

00D45A33     50                         push eax

00D45A34     FFB5 ECD6FFFF             push dword ptr ss:[ebp-2914]

00D45A3A     FF15 ECD2D400             call dword ptr ds:[D4D2EC]                 ; 

msvcrt.memmove

00D45A40     83C4 0C                   add esp,0C

00D45A43     8B85 5CD1FFFF             mov eax,dword ptr ss:[ebp-2EA4]

00D45A49     8B8D ECD6FFFF             mov ecx,dword ptr ss:[ebp-2914]-->4 注意 iat 首地址

00D45A4F     8B95 60D1FFFF             mov edx,dword ptr ss:[ebp-2EA0]--》首地址的函数

00D45A55     891481                     mov dword ptr ds:[ecx+eax*4],edx

00D45A58   ^ E9 47FFFFFF               jmp 00D459A4
```

在 `00D459A2` 设断，f9 运行，断下,当所有 `IAT` 处理完毕后，就可以用 `ImportREC.exe` 得到所有正确 `IAT` 了。

`RVA=3e7940-400000=FFFE7940`，`SIZA=990`，然后即可点 `get imports.` ，CUT 掉无效函数，在按 `save tree` 把输入



### 1. 代码提取与脱壳准备

将表存成`txt`文件。使用`f8`单步执行，注意上述提到的四个值，它们在已壳脱壳时有用。

将`00D459A4`到`00D45A58`之间的代码用二进制复制的方法复制下来备用。

```assembly
00D459D4     E8 98B6FDFF          call 00D21071      ; f7 进入

00D21071     55                   push ebp           ; dump 出来的代码 2
00D21072     8BEC                 mov ebp, esp
00D21074     51                   push ecx
00D21075     8B45 08              mov eax, dword ptr ss:[ebp+8]
00D21078     53                   push ebx
00D21079     B9 10270000          mov ecx, 2710
00D2107E     56                   push esi
00D2107F     99                   cdq
00D21080     8BF1                 mov esi, ecx
00D21082     57                   push edi
00D21083     F7FE                 idiv esi
00D21085     8B45 0C              mov eax, dword ptr ss:[ebp+C]
00D21088     8BF9                 mov edi, ecx
00D2108A     8BD9                 mov ebx, ecx
00D2108C     8955 FC              mov dword ptr ss:[ebp-4], edx
00D2108F     99                   cdq
00D21090     F7FE                 idiv esi
00D21092     8B45 08              mov eax, dword ptr ss:[ebp+8]
00D21095     8BF2                 mov esi, edx
00D21097     99                   cdq
00D21098     F7FF                 idiv edi
00D2109A     8BF8                 mov edi, eax
00D2109C     8B45 0C              mov eax, dword ptr ss:[ebp+C]
00D2109F     99                   cdq
00D210A0     0FAFFE               imul edi, esi
00D210A3     F7FB                 idiv ebx
00D210A5     0FAF75 FC            imul esi, dword ptr ss:[ebp-4]
00D210A9     33D2                 xor edx, edx
00D210AB     0FAF45 FC            imul eax, dword ptr ss:[ebp-4]
00D210AF     03C7                 add eax, edi
00D210B1     5F                   pop edi
00D210B2     F7F1                 div ecx
00D210B4     B9 00E1F505          mov ecx, 5F5E100
00D210B9     8BC2                 mov eax, edx
00D210BB     33D2                 xor edx, edx
00D210BD     69C0 10270000        imul eax, eax, 2710
00D210C3     03C6                 add eax, esi
00D210C5     5E                   pop esi
00D210C6     F7F1                 div ecx
00D210C8     5B                   pop ebx
00D210C9     8BC2                 mov eax, edx
00D210CB     C9                   leave
00D210CC     C2 0800              retn 8
```

将`00D21071`到`00D210CC`之间的代码也用二进制复制的方法复制下来备用。

### 2. 修复 IAT

(5) 修复`iat`。

将`dump`出来的文件`oep`改为`004d874c`，用`od`加载，如下：

```assembly
004D874C z>   55                   push ebp
004D874D       8BEC                 mov ebp, esp
004D874F       83C4 F4              add esp, -0C
004D8752       53                   push ebx
004D8753       B8 4C814D00          mov eax, zy2.004D814C
004D8758       E8 3BE1F2FF          call zy2.00406898 ; f7 进入

00406898       50                   push eax
00406899       6A 00                push 0
0040689B       E8 F8FEFFFF          call zy2.00406798 ; f7 进入

004068A0   -   E9 B0C5F902          jmp zy2.033A2E55

00406798       FF25 60793E00        jmp dword ptr ds:[3E7960] ; 这些地址就是 iat, 但都在 400000 以内，我把它修改一下。

0040679E       8BC0                 mov eax, eax
004067A0       FF25 947B3E00        jmp dword ptr ds:[3E7B94]
004067A6       8BC0                 mov eax, eax
004067A8       FF25 E07A3E00        jmp dword ptr ds:[3E7AE0]
004067AE       8BC0                 mov eax, eax
004067B0       FF25 747E3E00        jmp dword ptr ds:[3E7E74]
```

用`winhex`打开修复了`cc`的文件，在`401000`---`4d9000`范围内，用查找和替换功能，用`FF25FFFF4E00`替换`FF25FFFF3E00`，中间的`ffff`是通配符，设通配符为`ff`。用`winhex`打开保存下来的`iat`原始文本文件，用查找和替换功能，用`000E`替换`FFFD`，`ok`，下一步运行`ImportREC 1.6`，提取修改过的`iat`原始文本文件，`FixDump`。

这中间可能遇到`iat`的首地址每次运行都不同的问题，多做两遍，或是用复制加粘贴的方法，用`ImportREC 1.6`在我们想要得地址处提取`iat`原始文件。

### 3. 以壳脱壳与最终修复

以壳脱壳，修复`iat`用`od`加载`dump`文件，如下：

**修改前：**

```assembly
004D874C z>   55                   push ebp
004D874D       8BEC                 mov ebp, esp
004D874F       83C4 F4              add esp, -0C
004D8752       53                   push ebx
004D8753       B8 4C814D00          mov eax, zy2.004D814C
004D8758       E8 3BE1F2FF          call zy2.00406898
```

**修改后：**

```assembly
004D874C u>   55                   push ebp
004D874D       8BEC                 mov ebp, esp
004D874F       83C4 F4              add esp, -0C
004D8752       53                   push ebx
004D8753   -   E9 A8290100          jmp un-Aspr.004EB100
004D8758       E8 3BE1F2FF          call un-Aspr.00406898
```

修改`004EB100`处的代码：

```assembly
004EB100       90                   nop
004EB101       90                   nop
004EB102       90                   nop
004EB103    .  C785 6CD1FFFF 0B020000   mov dword ptr ss:[ebp-2E94], 20B
004EB10D    .  C785 ECD6FFFF 40794E00   mov dword ptr ss:[ebp-2914], <&kernel32.CloseHandle>
004EB117    .  C785 48D8FFFF 64020000   mov dword ptr ss:[ebp-27B8], 264
004EB121    .  C785 68D1FFFF 731E4400   mov dword ptr ss:[ebp-2E98], un-Aspr.00441E73
004EB12B    .  EB 21                jmp short un-Aspr.004EB14E
004EB12D       90                   nop
004EB12E       90                   nop
004EB12F       90                   nop
004EB130       90                   nop
004EB131       90                   nop
004EB132       90                   nop
004EB133       90                   nop
004EB134       90                   nop
004EB135       90                   nop
004EB136       90                   nop
004EB137       90                   nop
004EB138       90                   nop
004EB139       90                   nop
004EB13A       90                   nop
004EB13B       90                   nop
004EB13C       90                   nop
004EB13D       90                   nop
004EB13E       90                   nop
004EB13F       90                   nop
004EB140       90                   nop
004EB141    >  8B85 64D1FFFF          mov eax, dword ptr ss:[ebp-2E9C] ; 前面 dump 出来的代码 1 开始
004EB147    .  40                   inc eax
004EB148    .  8985 64D1FFFF          mov dword ptr ss:[ebp-2E9C], eax
004EB14E    >  8B85 64D1FFFF          mov eax, dword ptr ss:[ebp-2E9C]
004EB154    .  3B85 6CD1FFFF          cmp eax, dword ptr ss:[ebp-2E94]
004EB15A    .  0F83 9A000000         jnb un-Aspr.004EB1FA
004EB160    .  68 0D5EDF01           push un-Aspr.01DF5E0D
004EB165    .  FFB5 68D1FFFF         push dword ptr ss:[ebp-2E98]
004EB16B    .  8D8D 68D1FFFF         lea ecx, dword ptr ss:[ebp-2E98]
004EB171    .  E8 AD000000           call un-Aspr.004EB223
004EB176    .  40                   inc eax
004EB177    .  33D2                 xor edx, edx
004EB179    .  B9 00E1F505          mov ecx, 5F5E100
004EB17E    .  F7F1                 div ecx
004EB180    .  8995 68D1FFFF         mov dword ptr ss:[ebp-2E98], edx
004EB186    .  8B85 68D1FFFF         mov eax, dword ptr ss:[ebp-2E98]
004EB18C    .  33D2                 xor edx, edx
004EB18E    .  B9 10270000          mov ecx, 2710
004EB193    .  F7F1                 div ecx
004EB195    .  0FAF85 48D8FFFF      imul eax, dword ptr ss:[ebp-27B8]
004EB19C    .  33D2                 xor edx, edx
004EB19E    .  B9 10270000          mov ecx, 2710
004EB1A3    .  F7F1                 div ecx
```



```
004EB1A5    .  8985 5CD1FFFF               mov dword ptr ss:[ebp-2EA4],eax  
004EB1AB    .  8B85 ECD6FFFF               mov eax,dword ptr ss:[ebp-2914]  
004EB1B1    .  8B00                         mov eax,dword ptr ds:[eax]  
004EB1B3    .  8985 60D1FFFF                mov dword ptr ss:[ebp-2EA0],eax  
004EB1B9    .  8B85 5CD1FFFF                mov eax,dword ptr ss:[ebp-2EA4]  
004EB1BF    .  8D0485 04000000              lea eax,dword ptr ds:[eax*4+4]  
004EB1C6    .  50                            push eax                                      ; /n  
004EB1C7    .  8B85 ECD6FFFF                mov eax,dword ptr ss:[ebp-2914]                ; |  
004EB1CD    .  83C0 04                      add eax,4                                      ; |  
004EB1D0    .  50                            push eax                                      ; |src  
004EB1D1    .  FFB5 ECD6FFFF                 push dword ptr ss:[ebp-2914]                 ; |dest  
004EB1D7    .  FF15 38794E00                call dword ptr ds:[<&msvcrt.memmove>]         ; \memmove  
004EB1DD    .  83C4 0C                      add esp,0C  
004EB1E0    .  8B85 5CD1FFFF                mov eax,dword ptr ss:[ebp-2EA4]  
004EB1E6    .  8B8D ECD6FFFF                mov ecx,dword ptr ss:[ebp-2914]  
004EB1EC    .  8B95 60D1FFFF                mov edx,dword ptr ss:[ebp-2EA0]  
004EB1F2    .  891481                      mov dword ptr ds:[ecx+eax*4],edx  
004EB1F5    .^ E9 47FFFFFF                   jmp un-Aspr.004EB141---》前面 dump 出来的代码 1 结束  
004EB1FA    >  90                            nop  
004EB1FB    .  90                            nop  
004EB1FC    .  90                            nop  
004EB1FD    .  90                            nop  
004EB1FE    .  90                            nop  
004EB1FF    .  90                            nop  
004EB200    .  90                            nop  
004EB201    .  90                            nop  
004EB202    .  90                            nop  
004EB203    .  90                            nop  
004EB204    .  90                            nop  
004EB205    .  B8 4C814D00                  mov eax,un-Aspr.004D814C  
004EB20A    .- E9 49D5FeFF                   jmp un-Aspr.004D8758  
004EB20F       90                            nop  
004EB210       90                            nop  
004EB211       90                            nop  
004EB212       90                            nop  
004EB213       90                            nop  
004EB214       90                            nop  
004EB215       90                            nop  
004EB216       90                            nop  
004EB217       90                            nop  
004EB218       90                            nop  
004EB219       90                            nop  
004EB21A       90                            nop  
004EB21B       90                            nop  
004EB21C       90                            nop  
004EB21D       90                            nop  
004EB21E       90                            nop  
004EB21F       90                            nop  
004EB220       90                            nop  
004EB221       90                            nop  
004EB222       90                            nop  
004EB223   /$  55                            push ebp\------》前面 dump 出来的代码 2 开始  
004EB224   |.  8BEC                       mov ebp,esp  
004EB226   |.  51                           push ecx  
004EB227   |.  8B45 08                      mov eax,dword ptr ss:[ebp+8]  
004EB22A   |.  53                           push ebx  
004EB22B   |.  B9 10270000                 mov ecx,2710  
004EB230   |.  56                           push esi  
004EB231   |.  99                           cdq  
004EB232   |.  8BF1                        mov esi,ecx  
004EB234   |.  57                           push edi  
004EB235   |.  F7FE                        idiv esi  
004EB237   |.  8B45 0C                     mov eax,dword ptr ss:[ebp+C]  
004EB23A   |.  8BF9                        mov edi,ecx  
004EB23C   |.  8BD9                        mov ebx,ecx  
004EB23E   |.  8955 FC                      mov dword ptr ss:[ebp-4],edx  
004EB241   |.  99                           cdq  
004EB242   |.  F7FE                        idiv esi  
004EB244   |.  8B45 08                     mov eax,dword ptr ss:[ebp+8]  
004EB247   |.  8BF2                        mov esi,edx  
004EB249   |.  99                           cdq  
004EB24A   |.  F7FF                        idiv edi  
004EB24C   |.  8BF8                        mov edi,eax  
004EB24E   |.  8B45 0C                     mov eax,dword ptr ss:[ebp+C]  
004EB251   |.  99                           cdq  
004EB252   |.  0FAFFE                      imul edi,esi  
004EB255   |.  F7FB                        idiv ebx  
004EB257   |.  0FAF75 FC                   imul esi,dword ptr ss:[ebp-4]  
004EB25B   |.  33D2                        xor edx,edx  
004EB25D   |.  0FAF45 FC                   imul eax,dword ptr ss:[ebp-4]  
004EB261   |.  03C7                        add eax,edi  
004EB263   |.  5F                           pop edi  
004EB264   |.  F7F1                        div ecx  
004EB266   |.  B9 00E1F505                mov ecx,5F5E100  
004EB26B   |.  8BC2                        mov eax,edx  
004EB26D   |.  33D2                        xor edx,edx  
004EB26F   |.  69C0 10270000             imul eax,eax,2710  
004EB275   |.  03C6                        add eax,esi  
004EB277   |.  5E                           pop esi  
004EB278   |.  F7F1                        div ecx  
004EB27A   |.  5B                           pop ebx  
004EB27B   |.  8BC2                        mov eax,edx  
004EB27D   |.  C9                           leave  
004EB27E   \.  C2 0800                     retn 8----》前面 dump 出来的代码 2 结束  
```

修改完毕，重新做了一遍，给文件减了一下肥，修改了一些地方，在 `w98` 和 `winxp` 下作了测试，运行正常，但如果在 `winxp` 下装了瑞星就会运行出错，不知为什么，还望各位朋友指教，谢谢您耐着性子看完，有些东西写的不是很清楚，还忘见谅，有些东西需要自己细细体会和琢磨。自己太菜，有不对的地方还望指教。

2004.10.3 下午 5.20

`4dd466d0e1e0cc123afbe504da2462ca`
