
# DebugApiSpy v3.2.1.56

> 原文：[`www.pediy.com/kssd/pediy06/pediy6478.htm`](https://www.pediy.com/kssd/pediy06/pediy6478.htm)

破解 `DebugApiSpy` `v3.2.1.56`

上次 `trw2000` 兄的贴子里提到这个软件，我研究了一阵，现把结果呈给大家，请大家指正。我先扔一块石头，有玉的尽管朝我砸来。

原贴：`http://bbs2.pediy.com/viewtopic.php?t=5531&highlight=%CB%AB%B2%E3%BF%C7`

这是一个双层壳，比较简单。虽然可以一步一步跟，但 `trw2000` 兄的方法简洁实用，所以这儿用他的方法(谢谢 `trw2000` 兄)

【转】

## 一、脱壳

一运行，要注册。很久没搞破解啦，就拿它开刀温习一下功课吧。

因为在 `od` 中找不到可用的中文字符串，所以考虑用 `w32dasm` 反汇编，但要求先脱壳。

用 `FI2.5` 查看，是用 `AsPack2.1` 加的壳，很久没有手工跟啦，手工跟一把吧。用 `od` 刚一载入，停在：

```
00469001  PUSHAD
```

`Ctrl-F` 查找 `popad` 指令，去掉“整个区段”选项，找到这里：

```
004694F3  POPAD
004694F4  JNZ SHORT 004694FE
004694F6  MOV EAX,1
004694FB  RETN 0C
004694FE  PUSH 00464000
00469503  RETN
```

让光标停在上面一段的最后一行，按 `F4` 一次，再 `F8` 一次，即可到达第一层壳的 OEP。这时如果转存进程的话，用 `fi` 查看还有一层 `PeCompact` 的壳，并且不能运行（用工具自动脱也不能运行，不知怎样修改）。因此继续脱第二层壳，起始一段如下，有特征命令 `pushfd` 和 `pushad`，是壳的开始：

```
00464000  JMP SHORT 00464008
00464002  PUSH 2E9E8    这就是程序的原始 OEP 的 RVA 地址
00464007  RETN
00464008  PUSHFD
00464009  PUSHAD
0046400A  CALL 00464011
```

跟到下面一段，有特征命令 `popad` 和 `popfd`，是壳的结束：

```
0046554E  POPAD
0046554F  POPFD
00465550  PUSH EAX
00465551  PUSH 0042E9E8
00465556  RETN 4
```

到达 `00465556` 一行，再 `F8` 一次，返回到真正入口点：

```
0042E9E8  PUSH EBP
0042E9E9  MOV EBP,ESP
```

光标停在入口点后转存该进程，再用 `fi` 查看无壳，是用 `Visual C++ 6.0` 编写的。但是用 `od` 直接转存的程序不能运行。重跟到入口点，用别的方法转存。重跟就没有必要细跟啦，只要在下面几个关键点设断即可：

1.  停在入口点时，在 `00469503` 处设断，中断后单步一下，即到达第二层壳的入口点。
2.  在第二层壳的入口点时，壳的结尾 `00465556` 处还没有代码，因此无法把断点设在这里，要分几步到才能到达这里。第一步可以断点可以设在 `0046407B`，执行到这儿后，向结尾处写代码的代码才出现：

    ```
    00464077  REP MOVS DWORD PTR ES:[EDI],DWORD PTR [ESI]
    00464079  MOV EDI,EBX
    0046407B  RETN
    ```

3.  第二步，在 `0046407B` 处中断后，断点可以设在 `004651A7`，执行到这儿以后壳结尾代码才形成：

    ```
    004651A5  REP MOVS BYTE PTR ES:[EDI],BYTE PTR [ESI]
    004651A7  JMP SHORT 004651CF
    ```

4.  第三步，在 `004651A7` 处中断后，在壳结尾代码 `00465556` 处设断，中断后单步走到真正的 OEP。

用 `PEditor` 来 dump 进程，并修正入口点，测试，程序不能运行，用别的方法 dump，也不能运行。

【结束】

当然在最后需要用 `ImportREC` 来修复输入表（未加密）。

下面我们来修复 dump 出来的程序。

## 二、修复

### 1.去掉 ZwSetInformationThread

```
00409BD1   PUSH DebugApi.0044DD34           ; ASCII "ZwSetInformationThread"
00409BD6   PUSH DebugApi.0044DD28           ; ASCII "ntdll.dll"
00409BDB   CALL DWORD PTR DS:[4361D8]       ; kernel32.GetModuleHandleA
00409BE1   PUSH EAX
00409BE2   CALL DWORD PTR DS:[4361E8]       ; kernel32.GetProcAddress
00409BE8   MOV ESI, EAX
00409BEA   CMP ESI, EDI
00409BEC   JE SHORT DebugApi.00409BFB
00409BEE   PUSH EDI
00409BEF   PUSH EDI
00409BF0   PUSH 11
00409BF2   CALL DWORD PTR DS:[436150]       ; kernel32.GetCurrentThread
00409BF8   PUSH EAX                        //nop
00409BF9   CALL ESI                        //nop
```

脱壳程序在 `00409BF9` 处调用 `ZwSetInformationThread` 函数来反调试，这个函数除了反调试以外没其他用途，因此可以把 `00409BF8` 和 `00409BF9` 处 nop 掉。

### 2.解决动态解码

运行程序到 `424d0f` 处出错，上一句为 `CALL 0042407C`，进去看看，发现代码显然不对，这是程序动态自解码，脱壳后解码出错，因此运行也出错。

在 `OD` 中打开原程序在 `00409BF8` 处下一个硬件执行断点，运行停住，在 `00409BFB` 处 new origin here。在 `0042407C` 处下硬件执行断点，运行停住，`0042407C` 处已正确解码，`0042407C`～`0042447B`。可以直接用 `OD` 的 `Binary copy` 和 `Binary paste` 拷贝到脱壳程序中。但要注意，这样拷贝过去后，程序还是要对该段代码进行解码，因此要对解码函数处理。

下面寻找解码函数，运行到 `00409BFB` 后，在 `0042407C` 处下一硬件访问断点，再运行，中断：

```
0042CEA3      REP MOVS DWORD PTR ES:[EDI], DWORD PTR DS:[ESI]  //断下
```

这是把 `0042407C` 开始处的代码复制到一内存空间，准备解码。假设内存空间为 `0fc0570`，则在 `0fc0570` 处再下一硬件访问断点，运行。

```
0040542D       XOR EAX, EAX
0040542F       CMP DWORD PTR SS:[EBP-8], EAX
00405432       JE SHORT DebugApi.00405449
00405434       PUSH DWORD PTR SS:[EBP+8]
00405437       MOV ECX, DWORD PTR SS:[EBP-4]
0040543A       PUSH ESI
0040543B       CALL DebugApi.00404E68            //解码函数
00405440       ADD ESI, 8
00405443       ADD DWORD PTR SS:[EBP+8], 8
00405447       JMP SHORT DebugApi.0040546E
00405449       MOV CL, BYTE PTR DS:[EAX+ESI]
0040544C       MOV BYTE PTR DS:[EAX+EDI], CL     //断在这
0040544F       INC EAX
00405450       CMP EAX, 8
00405453       JL SHORT DebugApi.00405449
00405455       PUSH DWORD PTR SS:[EBP+C]
00405458       MOV ECX, DWORD PTR SS:[EBP-4]
0040545B       PUSH EDI
0040545C       CALL DebugApi.00404E68            //解码函数
00405461       PUSH 8
00405463       POP EAX
00405464       ADD DWORD PTR SS:[EBP+8], EAX
00405467       ADD ESI, EAX
00405469       ADD EDI, EAX
0040546B       ADD DWORD PTR SS:[EBP+C], EAX
0040546E       DEC EBX
0040546F       JNZ SHORT DebugApi.0040542D
```

进入 `CALL DebugApi.00404E68` 看看，好长一段代码，没去研究解码算法。改成在 `00404E68` 直接返回，即改成:

```
00404E68  RETN 8
```

修改完成。我们回到调用 `0042CEA3` 复制语句的上一层函数看看：

```
00424CCC     PUSH ESI
00424CCD     PUSH DWORD PTR SS:[EBP+C]
00424CD0     PUSH EDI
00424CD1     CALL DebugApi.0042CE70               //复制
00424CD6     ADD ESP, 0C
00424CD9     MOV ECX, EBX
00424CDB     PUSH ESI
00424CDC     PUSH DWORD PTR SS:[EBP-10]
00424CDF     PUSH EDI
00424CE0     CALL DebugApi.004053F9               //解码
00424CE5     TEST EBX, EBX
00424CE7     JE SHORT DebugApi.00424CF7
00424CE9     MOV ECX, EBX
00424CEB     CALL DebugApi.00404AE1
00424CF0     PUSH EBX
00424CF1     CALL DebugApi.0042D1A5
00424CF6     POP ECX
00424CF7     PUSH ESI
00424CF8     PUSH DWORD PTR SS:[EBP-10]
00424CFB     PUSH DWORD PTR SS:[EBP+C]
00424CFE     CALL DebugApi.0042CE70               //把正确代码复制到 42407c
00424D03     PUSH DWORD PTR SS:[EBP-10]
00424D06     PUSH EDI
00424D07     PUSH DWORD PTR SS:[EBP+10]
00424D0A     CALL DebugApi.0042407C               //运行 42407c
00424D0F     PUSH ESI
00424D10     PUSH EDI
00424D11     PUSH DWORD PTR SS:[EBP+C]
00424D14     CALL DebugApi.0042CE70               //运行完后，把错误代码复制回 42407c
00424D19     PUSH DWORD PTR SS:[EBP-10]
```

另外 `4144e2`～`4158e1` 也是类似的动态解码。程序中有两处解码函数：

```
0042429E    CALL dumped_6.004053F9
00424CE0    CALL dumped_6.004053F9
```

就是这两个地方的解码。也有可能还有其他地方是动态解码，我没有再找。

### 3.解决自效验

这个问题一开始我的方法是正面去找，即运行脱壳程序一处一处找，找到了一些地方，但太麻烦，后来看到程序中有这样的语句：

```
CALL DWORD PTR DS:[<&kernel32.ExitProces]; ExitProcess
```

这不就是退出函数吗，因此查找所有命令：`CALL DWORD PTR DS:[4361F4]`，然后在每处都设断，运行，停住：

(1)

```
00414589    CMP EAX, DWORD PTR SS:[ESP+14]
0041458D    JE SHORT dumped_4.00414597     //JE 改成 JMP
0041458F    PUSH 0                        ; /ExitCode = 0
00414591    CALL DWORD PTR DS:[<&kernel32.ExitProces]; ExitProcess   //停在这
```

`00414589` 处就是一个自效验，把 `0041458D` 处的 je 改成 jmp。好，其他的也类似修改：

(2)

```
004167E3    CMP EAX, DWORD PTR SS:[ESP+14]
004167E7    JE SHORT dumped_4.004167F1     //JE 改成 JMP
004167E9    PUSH 0                        ; /ExitCode = 0
004167EB    CALL DWORD PTR DS:[<&kernel32.ExitProces>; ExitProcess
```

修改好这两处后，程序可以运行了，但运行一段时间后还会退出，因此还有自效验：

(3)

```
004168D3    CMP ECX, EDX
004168D5    MOV DWORD PTR SS:[EBP+8], ECX
004168D8    JE SHORT dumped_4.004168E2     //JE 改成 JMP
004168DA    PUSH 0                        ; /ExitCode = 0
004168DC    CALL DWORD PTR DS:[<&kernel32.ExitProces>; ExitProcess
```

(4)

```
00416907    CMP EAX, DWORD PTR SS:[EBP-4]
0041690A    MOV DWORD PTR SS:[EBP+8], EAX
0041690D    JE SHORT dumped_4.0041691A     //JE 改成 JMP
0041690F    PUSH 0                        ; /ExitCode = 0
00416911    CALL DWORD PTR DS:[<&kernel32.ExitProces>; ExitProcess
```

(5)

```
00416928    CMP EAX, DWORD PTR DS:[ECX+8]
0041692B    JE SHORT dumped_4.00416935     //JE 改成 JMP
0041692D    PUSH 0                        ; /ExitCode = 0
0041692F    CALL DWORD PTR DS:[<&kernel32.ExitProces>; ExitProcess
```

修改好这些地方以后，程序正常运行。如果使用时还会退出，用同样的方法解决。

## 三、破解

(1)爆破：

从注册表中取假注册码：



```
0042497C     PUSH EAX                 //注册码字符长度保存地址
0042497D     PUSH EBX                 //注册码保存地址
0042497E     PUSH dumped_6.00452818           ;  ASCII "RegInfoBin"
00424983     CALL dumped_6.00424A5E

........
```

**假注册码计算：**

```
004249B1    PUSH EAX                                   ; /Arg6
004249B2    PUSH 0                                     ; |Arg5 = 00000000
004249B4    PUSH 10                                    ; |Arg4 = 00000010
004249B6    PUSH DWORD PTR DS:[ESI+A4]                 ; |Arg3
004249BC    PUSH 10                                    ; |Arg2 = 00000010
004249BE    PUSH EBX                                   ; |Arg1
004249BF    CALL dumped_6.0041EF76                     ; dumped_6.0041EF76
```

**比较：**

```
004249CE E8 58000000   CALL dumped_6.00424A2B             //比较 call
004249D3 85C0           TEST EAX, EAX                      //=0 错，=1 对
004249D5 74 1B          JE SHORT dumped_6.004249F2
```

因此要爆破，只要把 `004249D5` 处改成 `JMP SHORT dumped_6.004249E2` 即可。这段代码中没出现真注册码的明码，是计算后的结果比较。

**(1) 寻找注册码：**

这个软件注册算法很复杂，我没看明白，也不清楚找到的是不是真的注册码，但好像可以用，有兴趣的可以自己试试：

```
004244BE   PUSH EBX                               ; /Arg6
004244BF   PUSH ECX                               ; |Arg5
004244C0   OR DWORD PTR SS:[EBP-4], FFFFFFFFFFFF   ; |
004244C4   PUSH 10                                ; |
004244C6   LEA ECX, DWORD PTR DS:[ESI+90]          ; |
004244CC   POP EDI                                ; |
004244CD   LEA EAX, DWORD PTR DS:[ESI+24]          ; |
004244D0   PUSH EDI                               ; |Arg4 => 00000010
004244D1   PUSH ECX                               ; |Arg3
004244D2   PUSH 8                                 ; |Arg2 = 00000008
004244D4   PUSH EAX                               ; |Arg1
004244D5   CALL dumped_6.0041F14C                  ; dumped_6.0041F14C
```

关键 call：`04244D5 CALL dumped_6.0041F14C`，`Arg5` 为保存注册码的地址。

运行过这个 call，得到注册码：`AACBF24A6CE25E89AE6643DDC496502B`

对应的注册 ID：`A7E4EB4B4A683C9E10ED6CF866F5188E`

pyzpyz

2004.3.31

`4dd466d0e1e0cc123afbe504da2462ca`
