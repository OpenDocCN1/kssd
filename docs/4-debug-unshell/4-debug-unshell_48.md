
# `attach` 法脱壳

> 原文：[`www.pediy.com/kssd/pediy06/pediy6682.htm`](https://www.pediy.com/kssd/pediy06/pediy6682.htm)

`attach` 法脱壳

`kongfoo/2004.5.10-12`

现在的壳 `anti` 都不弱，如果从壳入口一步步跟到 `OEP` 的话会很累，这里介绍一种比较舒服的方法，不过有小小要求：掌握壳的一般知识，如 `IAT` 加密，`stolen code` 等。掌握 `PE` 文件知识，如一般地址表会放在什么地方，各种编译器生成的代码入口点一般在什么地方等。掌握汇编语言的编写，因为我们面对的是加密后的 `IAT`，修复时通常都要写程序修复。

快速脱壳方法有很多种，`Memory Map` 断点法（我最先见到是二哥/`David` 的文章介绍这种方法），`ESP` 定律法（`fly` 大侠总结出来的方法）等。

现在这种方法姑且叫 `attach` 法吧，当然这个方法不是我首创的，我只是总结一下。相信很多大侠都用过这种方法脱壳![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)`attach` 法有局限性：目标有 `anti attach` 的话就很麻烦了，比如双进程，进程逃逸等。

`DFCG` 上 `Volx` 大侠提供的 `asp1.4` 壳目标程序。用这个程序做 `attach` 法脱壳例子不很合适，因为这个程序在 `OD` 下可以运行起来，而 `attach` 法脱壳比较适用于对付在 `OD` 下不能直接运行起来的壳。不管了![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg) 这里只介绍有这样一种办法，当然只靠这个方法是不行的，几种方法结合起来才有效率。

## 壳入口

（`OD` 设置：忽略全部异常，`IsDebuggerPresent` `Hide`）

```
00401000  >  68 01D07300      PUSH dskchkr.0073D001
00401005    E8 01000000      CALL dskchkr.0040100B
0040100A    C3               RETN
0040100B    C3               RETN
```

用 `attach` 上的程序和壳比较一下就可以发现，壳的代码在程序运行后换成原程序的代码了。我们要做的是恢复一份 `IAT`。在进程空间 `bpx j`，观察一下函数调用情况，都 `CALL` 到动态申请的空间了，有部分 `API` 没有处理。

看看一个地址项目：`call [cdca0c]` `cdca0c` 的内容是 `cdca18`，去 `cdca18` 分析一下代码就可以找出原 `API` 地址。（伪 `API` 代码 `cdca18` 紧接着 `cdca0c` 后面）

```
00CDCA18     33C0             XOR EAX,EAX
00CDCA1A     50              PUSH EAX
00CDCA1B     50              PUSH EAX
00CDCA1C     50              PUSH EAX
00CDCA1D     FF7424 10       PUSH DWORD PTR SS:[ESP+10]
00CDCA21     50              PUSH EAX
00CDCA22     F2:             PREFIX REPNE:                       ; Superfluous prefix
00CDCA23     EB 01          JMP SHORT 00CDCA26
00CDCA25     90              NOP   ==处理过花指令
00CDCA26     68 BF20C477     PUSH 77C420BF   ==根据这个地址很容易就找出原 API 地址了
```

去 `77c420bf` 向上一看，`API` 地址是 `77c420b0`。去 `cdca0c` 写入地址（反序），原来是 `gdi32.CreateSlidBrush`。

分析一下其对 `API` 的处理，有几种情况：

1. 摘取 `API` 从开头到 `CALL` 之前的代码，到 `CALL` 时压入 `CALL` 之后的地址，后面接着摘取 `CALL` 里面的代码过来。
2. 整个 `API` 搬了过来。
3. 摘取 `API` 从开头到 `CALL` 之前的代码，再压入 `CALL` 的地址返回。
4. 分析的 `API` 不多，可能还有其它处理方式。

伪地址表在 `cb0000` 段开始。

`bp VirtualAlloc`，几次之后申请 `cb0000`（连续申请 2 次，第 2 次才是），可以跟了。申请 `cb0000` 后回到这里：

```
00C815CE     85C0             TEST EAX,EAX

00C815DE     81FE 5024CA00    CMP ESI,0CA2450   ==本来想去 ca2450 看看的，眼花了看成是 ca4250，去到一看，乖乖，是正确的 API 地址表！
```

立即复制一份下来（用 `binary copy`）。`ca4100` 开始。看看 `401000` 处，代码还没解码。还是 `bp VirtualAlloc`，在第 2 次申请 `cc0000` 时原程序解码了（用 `dump` 窗口监视）。所有 `ff15` 的 `CALL` 的目的地址都用随机数改掉了。找第一个 `CALL` 下内存写断点，看看哪里进行处理。

```
00402676     FF15 6034CD00    CALL DWORD PTR DS:[672B6EAE]   ==在 402678 下内存写断

00C95755     8906             MOV DWORD PTR DS:[ESI],EAX   ==断下来，知道处理代码的地址就好办了。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　 ==重来，bp VirtualAlloc 原程序解码后去 c95755 写修复代码就行了
```

此时的 `ebp` 是 `api` 地址。在原程序找一块地方，用来放刚才无意中得到的 `API` 地址表，再用 `ebp` 放着的地址比较地址表里面的地址，就可以确定写回 `CALL` 目的地址的地址了。

\=======这里到“代码修复 `API` 地址的小结”一段只是记录当时处理的过程，实际操作只需按小结的说明进行。

## 代码修复 API 地址

地址表放到 `500000` 处（用 `binary paste`）。将 `mov [esi],eax` 一句改成 `jmp 0ca3650`。

代码：执行前先将 `400000` 段的 `access` 设为 `full access` 并把地址表放到 `500000`。

```
00CA3650     50               PUSH EAX
00CA3651     B8 00044000     MOV EAX,500000
00CA3656     3B28            CMP EBP,DWORD PTR DS:[EAX]
00CA3658     74 0C          JE SHORT 00CA3666
00CA365A     83C0 04         ADD EAX,4
00CA365D     3D 04064000     CMP EAX,500204
00CA3662     77 0F          JA SHORT 00CA3673
00CA3664  ^  EB F0          JMP SHORT 00CA3656
00CA3666     8906            MOV DWORD PTR DS:[ESI],EAX
00CA3668     58               POP EAX
00CA3669     0FB74424 04     MOVZX EAX,WORD PTR SS:[ESP+4]
00CA366E  ^\ E9 E920FFFF      JMP 00C9575C
00CA3673     8928            MOV DWORD PTR DS:[EAX],EBP
00CA3675     8305 5E36CA00 04    ADD DWORD PTR DS:[CA365E],4
00CA367C  ^  EB E8            JMP SHORT 00CA3666
```

在 `0c95900` 处按 `F4` 就处理完了。好了，第一步，`API` 地址修复完了，看看程序中还有很多东西要修复。

### 调用壳代码

```
00401268     E8 8FA18D00      CALL 00CDCB18   ==地址是动态的。跟去分析一下，原来是把 FF15 的 CALL 变成 E8 的 CALL，
　　　　　　　　　　　　　　　　　　　　　 ==最后一字节随机，在修复 API 地址前在 401268 下内存写断，用代码修复。

00C9369A     C603 E8          MOV BYTE PTR DS:[EBX],0E8   ==这里断下，ebp 是原 API 地址
```

注意，这段代码运行前要做的工作有：
1. 设置 `full access`。
2. 写好 `0CA3650` 的代码。
3. `API` 地址表放到 `500000`。

代码：（把 `0c9369a` 的指令改成 `jmp 0ca3700`）

```
00CA3700     50               PUSH EAX
00CA3701     B8 00044000     MOV EAX,500000
00CA3706     3B28            CMP EBP,DWORD PTR DS:[EAX]
00CA3708     74 0C          JE SHORT 00CA3716
00CA370A     83C0 04         ADD EAX,4
00CA370D     3D 04064000     CMP EAX,500204
00CA3712     77 0F          JA SHORT 00CA3723
00CA3714  ^  EB F0          JMP SHORT 00CA3706
00CA3716     66:C703 FF15    MOV WORD PTR DS:[EBX],15FF
00CA371B     8906            MOV DWORD PTR DS:[ESI],EAX
00CA371D     58               POP EAX
00CA371E  ^  E9 7DFFFEFF     JMP 00C936A0
00CA3723     8928            MOV DWORD PTR DS:[EAX],EBP
00CA3725     8305 5E36CA00 0>ADD DWORD PTR DS:[CA365E],4
00CA372C     8305 0E37CA00 0>ADD DWORD PTR DS:[CA370E],4
00CA3733  ^  EB E1           JMP SHORT 00CA3716
```

在 `0c956a2` 按 `F4` 执行完修复代码之后大部分 `API` 调用都正常了。（在 `0c956a2` 按 `F4` 后 `0CA3650` 的修复代码也执行过了）

再来检查一下，`bpx j`，还有几个没修复。手动搞一下。

前 4 个都是调用 `c93f78` 函数，去看一下，发现有一个 `CALL`：

```
00C93F94     E8 6717FFFF      CALL 00C85700                     ; JMP to kernel32.GetProcAddress
```

去 `c85700` 一看，`DELPHI` 格式的跳转表。哪么 `OEP` 在代码的尾部啦。先不管这个，看看 `Intermodular calls` 窗口的调用的函数表，还有很多 `API` 要修复。

### DELPHI 格式的跳转表

```
00C85700  -  FF25 7042CA00    JMP DWORD PTR DS:[CA4270]              ; kernel32.GetProcAddress
00C85706     8BC0             MOV EAX,EAX
```

最前面的 4 个是 `GetProcAddress` 啦，注意 `ca4270` 这个地址，我们无意中得到的地址表是从 `ca4100` 开始的，搬到 `500000` 之后就是 `500170` 啦。把 4 个 `CALL` 都改成 `call [500170]` 就行了。

另外的一些 `CALL` 到壳里的也用同样的方法修复：下内存写断找出哪里写入，再改代码修复。

在 `402ace` 下内存写断。

```
00C9583A     8906              MOV DWORD PTR DS:[ESI],EAX   ==断下，和 0C95755 处的代码是一样的，所以代码很好写。
```

### 代码修复 API 地址的小结

1. `bp VirtualAlloc` 一直观察 `401000`，直到解码完成。
2. 把 `0ca3646` 改为 `040250`（反序的 `500204` 地址表尾）。
3. 改 `0C95755` 为 `jmp 0ca3650`，去 `0ca3650` 写代码；改 `0c9583a` 为 `jmp 0ca3690`，去 `0ca3690` 写代码；改 `0c9369a` 为 `jmp 0ca3700`，去 `0ca3700` 写代码。（所有代码用后面的代码，有修改）
4. 设置 `400000` 段为 `full access`。
5. 把 `API` 地址表写到 `500000` 处。
6. 到 `401268` 下内存写断。
7. 断下后去除中断再到 `0c956a2`、`0c95900` 按 `F4` 执行修复代码。
8. 手动修复 4 个 `GetProcAddress` 项目及其余项目。

### 0ca3650 的代码

```
00CA3650     50                PUSH EAX
00CA3651     B8 00044000       MOV EAX,500000
00CA3656     3B28             CMP EBP,DWORD PTR DS:[EAX]
00CA3658     74 0D           JE SHORT 00CA3667
00CA365A     83C0 04         ADD EAX,4
00CA365D     3B05 4636CA00    CMP EAX,DWORD PTR DS:[CA3646]        ; dskchkr.00500000
00CA3663     77 0F           JA SHORT 00CA3674
00CA3665  ^  EB EF          JMP SHORT 00CA3656
00CA3667     8906            MOV DWORD PTR DS:[ESI],EAX
00CA3669     58                POP EAX
00CA366A     0FB74424 04     MOVZX EAX,WORD PTR SS:[ESP+4]
00CA366F  ^  E9 E820FFFF      JMP 00C9575C
00CA3674     8928            MOV DWORD PTR DS:[EAX],EBP
00CA3676     8305 4636CA00 04    ADD DWORD PTR DS:[CA3646],4
00CA367D  ^  EB E8           JMP SHORT 00CA3667
```

### 0ca3690 的代码



### `00CA3690` 代码
```
00CA3690    50                   PUSH EAX
00CA3691    B8 00044000          MOV EAX,500000
00CA3696    3B28                 CMP EBP,DWORD PTR DS:[EAX]
00CA3698    74 0D                JE SHORT 00CA36A7
00CA369A    83C0 04              ADD EAX,4
00CA369D    3B05 4636CA00        CMP EAX,DWORD PTR DS:[CA3646]              ; dskchkr.00500000
00CA36A3    77 0F                JA SHORT 00CA36B4
00CA36A5   ^ EB EF              JMP SHORT 00CA3696
00CA36A7    8906                 MOV DWORD PTR DS:[ESI],EAX
00CA36A9    58                   POP EAX
00CA36AA    0FB74424 04          MOVZX EAX,WORD PTR SS:[ESP+4]
00CA36AF   ^ E9 8D21FFFF         JMP 00C95841
00CA36B4    8928                 MOV DWORD PTR DS:[EAX],EBP
00CA36B6    8305 4636CA00 04     ADD DWORD PTR DS:[CA3646],4
00CA36BD   ^ EB E8              JMP SHORT 00CA36A7
```

### `00CA3700` 代码
```
00CA3700    50                   PUSH EAX
00CA3701    B8 00044000          MOV EAX,500000
00CA3706    3B28                 CMP EBP,DWORD PTR DS:[EAX]
00CA3708    74 0D                JE SHORT 00CA3717
00CA370A    83C0 04              ADD EAX,4
00CA370D    3B05 4636CA00        CMP EAX,DWORD PTR DS:[CA3646]              ; dskchkr.00500000
00CA3713    77 0F                JA SHORT 00CA3724
00CA3715   ^ EB EF              JMP SHORT 00CA3706
00CA3717    66:C703 FF15        MOV WORD PTR DS:[EBX],15FF
00CA371C    8906                 MOV DWORD PTR DS:[ESI],EAX
00CA371E    58                   POP EAX
00CA371F   ^ E9 7CFFFEFF        JMP 00C936A0
00CA3724    8928                 MOV DWORD PTR DS:[EAX],EBP
00CA3726    8305 4636CA00 04     ADD DWORD PTR DS:[CA3646],4
00CA372D   ^ EB E8              JMP SHORT 00CA3717
```

### `00CA3646` 全部修复代码
```
04 02 50 00 00 90 90 90 90 90 50 B8 00 00 50 00 3B 28 74 0D 83 C0 04 3B 05 46 36 CA 00 77 0F EB

EF 89 06 58 0F B7 44 24 04 E9 E8 20 FF FF 89 28 83 05 46 36 CA 00 04 EB E8 00 FF FF FF FF 01 00

00 00 FF FF FF FF 00 90 90 90 50 B8 00 00 50 00 3B 28 74 0D 83 C0 04 3B 05 46 36 CA 00 77 0F EB

EF 89 06 58 0F B7 44 24 04 E9 8D 21 FF FF 89 28 83 05 46 36 CA 00 04 EB E8 00 00 00 00 00 00 00

00 00 00 50 B8 00 00 50 00 3B 28 74 0D 83 C0 04 3B 05 46 36 CA 00 77 0E EB EF 89 06 58 8A 4C 24

06 E9 A0 21 FF FF 89 28 83 05 46 36 CA 00 04 EB E9 00 00 00 00 00 00 90 90 90 50 B8 00 00 50 00

3B 28 74 0D 83 C0 04 3B 05 46 36 CA 00 77 0F EB EF 66 C7 03 FF 15 89 06 58 E9 7C FF FE FF 89 28

83 05 46 36 CA 00 04 EB E8

04 02 50 00 00 90 90 90 90 90 50 B8 00 00 50 00 3B 28 74 0D 83 C0 04 3B 05 46 36 CA 00 77 0F EB

EF 89 06 58 0F B7 44 24 04 E9 E8 20 FF FF 89 28 83 05 46 36 CA 00 04 EB E8 00 FF FF FF FF 01 00

00 00 FF FF FF FF 00 90 90 90 50 B8 00 00 50 00 3B 28 74 0D 83 C0 04 3B 05 46 36 CA 00 77 0F EB

EF 89 06 58 0F B7 44 24 04 E9 8D 21 FF FF 89 28 83 05 46 36 CA 00 04 EB E8 00 00 00 00 00 00 00

00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 90 90 90 50 B8 00 00 50 00

3B 28 74 0D 83 C0 04 3B 05 46 36 CA 00 77 0F EB EF 66 C7 03 FF 15 89 06 58 E9 7C FF FE FF 89 28

83 05 46 36 CA 00 04 EB E8
```

修复代码运行后（在 `0c95900` 按 `F4`）还有些没修复，这些 `CALL` 是由动态生成的代码处理的，不好用代码修复，反正也不多而且都有现成的地址参考，手动修复一下。比如：

```
415ed1 call [cf865c] ;CloseHandle  ==CloseHandle 地址在 5000a4，把 call [cf865c]改成 call [5000a4]
                          ==（Intermodular calls 窗口双击项目即可定位到程序中）
```

一轮修复后还有 3 个 API 没修复，`comdlg32` 的 `GetOpenFileNameA` 和 `GetSaveFileNameA` 以及 `comctl32` 的 `ImageList_GetImageCount`，马上更新一下 API 地址专家，增加这 2 个库，得到地址后去地址表尾部写入地址，再把 `call [xxxxx]` 改一下 `OK`。

用 API 地址专家取得的 `ImageList_GetImageCount` 地址与程序的不同，直接把程序的地址写到地址表尾部。

最后还有一个：

```
429ad8 call f042dd77  ==看样子是被破坏了，先不管。
```

### 找 OEP
虽然我们看到了 DELPHI 格式的跳转表，但并不代表这个就是 DELPHI 写的程序，因为跳转表是在壳里面的，并不程序里面。从程序空隙用 `CC` 填充的情况来看，程序应该是 `VC` 写的。

先不动修复好 API 地址的程序，再开一个 OD，`bp VirtualAlloc`，监视 `401000`，解码后去 `402ace` 下内存写断，再到 `0c95900` 按 `F4`，此时壳已经完成 API 处理。对照修复了 API 地址的程序，`GetModuleHandleA` 的地址有 3 个，分别下断。

`0042AFC8    E8 373B9400      CALL 00D6EB04  ==断下`

回到修复了 API 地址的程序，向上看，很快我们就找到 OEP 了：

```
0042AEF4    FA                   CLI     ==已经被破坏啦
0042AEF5    58                   POP EAX
0042AEF6    92                   XCHG EAX,EDX
0042AEF7    8F                   ???                                ; Unknown command
0042AEF8    76 34                JBE SHORT dskchkr.0042AF2E
0042AEFA    DDC1                 FFREE ST(1)
0042AEFC    D26F 64              SHR BYTE PTR DS:[EDI+64],CL
0042AEFF   ^ E3 84              JECXZ SHORT dskchkr.0042AE85
0042AF01    8F80 33E49FED        POP DWORD PTR DS:[EAX+ED9FE433]
0042AF07    D3CF                 ROR EDI,CL
0042AF09    6A B5                PUSH -4B
0042AF0B    E5 35                IN EAX,35                          ; I/O command
0042AF0D    18D3                 SBB BL,DL
0042AF0F    15 35C4E186          ADC EAX,86E1C435
0042AF14    DB07                 FILD DWORD PTR DS:[EDI]
0042AF16    79 1C                JNS SHORT dskchkr.0042AF34
0042AF18    A8 9A                TEST AL,9A
0042AF1A    FF15 1C064000        CALL DWORD PTR DS:[40061C]           ; kernel32.GetVersion
```

在第二个 OD 里面我们可以看到 OEP 处的情况：

```
0042AEF4  - E9 EE1E8900            JMP 00CBCDE7  ==对照样板分析一下 cbcde7（动态地址，每次不同）
                                  ==里面的代码就可以找到 stolen code 了
```

### 样板
`ezip` 主程序，以前做 `pmview` 收集来的资料，有各种入口 `![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)`

```
0041FFAF    55                   PUSH EBP
0041FFB0    8BEC                 MOV EBP,ESP
0041FFB2    6A FF                PUSH -1
0041FFB4    68 70C44200          PUSH ezip.0042C470
0041FFB9    68 A2014200          PUSH ezip.004201A2                       ; JMP to MSVCRT._except_handler3
0041FFBE    64:A1 00000000       MOV EAX,DWORD PTR FS:[0]
0041FFC4    50                   PUSH EAX
0041FFC5    64:8925 00000000     MOV DWORD PTR FS:[0],ESP
0041FFCC    83EC 20              SUB ESP,20
0041FFCF    53                   PUSH EBX
0041FFD0    56                   PUSH ESI
0041FFD1    57                   PUSH EDI
0041FFD2    8965 E8              MOV DWORD PTR SS:[EBP-18],ESP
0041FFD5    8365 FC 00           AND DWORD PTR SS:[EBP-4],0
0041FFD9    6A 01                PUSH 1
0041FFDB    FF15 ACF64400        CALL DWORD PTR DS:[44F6AC]             ; MSVCRT.__set_app_type
```

### 恢复的 OEP stolen code
```
0042AEF4    55                   PUSH EBP
0042AEF5    8BE5                 MOV EBP,ESP
0042AEF7    6A FF                PUSH -1
0042AEF9    68 90444300          PUSH dskchkr.00434490
0042AEFE    68 3C054300          PUSH dskchkr.0043053C
0042AF03    64:A1 00000000       MOV EAX,DWORD PTR FS:[0]
0042AF09    50                   PUSH EAX
0042AF0A    64:8925 00000000     MOV DWORD PTR FS:[0],ESP
0042AF11    83EC 58              SUB ESP,58
0042AF14    53                   PUSH EBX
0042AF15    56                   PUSH ESI
0042AF16    57                   PUSH EDI
0042AF17    8965 E8              MOV DWORD PTR SS:[EBP-18],ESP
```

`55 8B EC 6A FF 68 90 44 43 00 68 3C 05 43 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 EC 58`

`53 56 57 89 65 E8`

### API 地址表
```
00 00 00 00 C4 7C E5 77 75 32 F5 77 00 E3 F7 77 1F E2 F7 77 08 99 E5 77 34 9E E5 77 0A 98 E5 77

45 9A E5 77 81 98 E5 77 44 F0 E5 77 24 99 E5 77 CE 7C E5 77 72 46 E5 77 EF 3B E5 77 67 31 E5 77

B8 05 E6 77 21 7F E5 77 7A 17 E4 77 FD A5 E5 77 93 9F E5 77 99 A0 E5 77 3C 51 E5 77 7D 15 F5 77

38 C9 E5 77 18 06 E6 77 9E 5D E5 77 AA 8E E5 77 B5 5C E5 77 8C 9D E5 77 84 9A E9 77 81 8C E5 77

92 01 E5 77 3E 18 F6 77 82 8B E5 77 06 D7 E4 77 3D 9C E5 77 EF 93 E5 77 08 16 E4 77 06 84 E5 77

37 A8 E5 77 63 79 E5 77 00 00 00 00 D6 E1 D3 77 50 72 D1 77 D7 AD D3 77 EC 72 D1 77 00 00 00 00
```



```
D7 23 DA 77 EA 22 DA 77 9A 18 DA 77 00 00 00 00 25 18 0F 77 E8 29 10 77 1D 15 0F 77 62 36 0F 77

E8 14 0F 77 A4 16 0F 77 00 00 00 00 39 9B E5 77 61 8B E5 77 29 2B E5 77 B4 C5 E5 77 45 9A E5 77

81 98 E5 77 99 A0 E5 77 00 00 00 00 F0 59 DA 77 54 91 DB 77 D7 23 DA 77 23 AE DA 77 EA 22 DA 77

2A 84 DA 77 9A 18 DA 77 00 00 00 00 44 F0 E5 77 84 9A E9 77 F7 15 E5 77 5B D7 E4 77 FC 02 E6 77

D8 05 E6 77 14 1B E5 77 99 1B E5 77 6F 16 E5 77 03 38 E5 77 A3 36 E5 77 EF 81 E4 77 57 C6 E5 77

87 30 E5 77 21 7F E5 77 08 16 E4 77 A5 C3 E5 77 FD A5 E5 77 93 9F E5 77 99 A0 E5 77 3C 51 E5 77

89 0F E5 77 E3 C0 E4 77 9B 86 E4 77 63 31 E5 77 9F 84 E5 77 B5 5C E5 77 62 92 EA 77 37 A8 E5 77

63 79 E5 77 00 00 00 00 7D 16 BD 77 E3 15 BD 77 2D 16 BD 77 00 00 00 00 6C BB D2 77 25 95 D1 77

2D 27 D3 77 4C BB D3 77 10 E3 D2 77 D7 AD D3 77 50 72 D1 77 09 D2 D2 77 C0 77 D1 77 4F 27 D3 77

46 81 D1 77 CB F5 D1 77 30 BB D3 77 06 B4 D3 77 03 E3 D2 77 00 00 00 00 75 16 19 77 00 00 00 00

57 C6 E5 77 00 00 00 00
```

代码修复后手动补上的 `API` 地址 `500490`

```
B3 22 32 76 D6 8B 32 76 00 00 00 00 7A E5 32 77
```

`LordPE` `full dump`，`ImpREC` `fix dump`（`OEP` `2aef4`），运行 `OK`。 `4dd466d0e1e0cc123afbe504da2462ca`
