
# wordcard 3.10

> 原文：[`www.pediy.com/kssd/pediy06/pediy6573.htm`](https://www.pediy.com/kssd/pediy06/pediy6573.htm)

我的第二篇破文------wordcard 3.10 的破解明细

作者：`restxx`

破解目标：`wordcard` 3.10 这个东西是我一个月前下的，现在不知什么地方有但 3.20 版的很多，这个版本应该也不少

破解工具：`ollydbg1.10` (包括老罗的 `ULtra string ref`) `winxp`

破解分析：无壳，哇塞，爽！！这个 `VB` 写的破软件是用随机生成的机器码来生成序列号保护码的，

载入之后忽略所有的异常，并在 `OLLYDBG` 中用 `Shift+f9` 运行,

载入后看到的是如下代码，一看就知道是 `VB` 写的,由于是解释型语言，这样就只能在 `msvbvm60.DLL` 模块中下断了

________________________________

```
00404F98 > $  68 D0534000   PUSH wordcard.004053D0
00404F9D   .  E8 EEFFFFFF   CALL <JMP.&MSVBVM60.#100>
00404FA2   .  0000          ADD BYTE PTR DS:[EAX],AL
00404FA4   .  0000          ADD BYTE PTR DS:[EAX],AL
00404FA6   .  0000          ADD BYTE PTR DS:[EAX],AL
00404FA8   .  3000          XOR BYTE PTR DS:[EAX],AL
00404FAA   .  0000          ADD BYTE PTR DS:[EAX],AL
00404FAC   .  40            INC EAX
00404FAD   .  0000          ADD BYTE PTR DS:[EAX],AL
00404FAF   .  0000          ADD BYTE PTR DS:[EAX],AL
00404FB1   .  0000          ADD BYTE PTR DS:[EAX],AL
00404FB3   .  00C5          ADD CH,AL
00404FB5   .  1904A6        SBB DWORD PTR DS:[ESI],EAX
```

****************************************************************

选 查看\执行模块，找到 `msvbvm60.dll` 并 click 它，看到如下代码页

****************************************************************

```
66001000 >  53              PUSH EBX
66001001    BE E5778F4A     MOV ESI,4A8F77E5
66001006    E5 77           IN EAX,77                        ; I/O 命令
66001008 >  59              POP ECX
66001009    53              PUSH EBX
6600100A    E5 77           IN EAX,77                        ; I/O 命令
6600100C >  9E              SAHF
6600100D    FC              CLD
6600100E    E4 77           IN AL,77                         ; I/O 命令
66001010 >  ED              IN EAX,DX                        ; I/O 命令
66001011    95              XCHG EAX,EBP
66001012    E5 77           IN EAX,77                        ; I/O 命令
66001014 >  8B98 E4774A16   MOV EBX,DWORD PTR DS:[EAX+164A77E4]
6600101A    E5 77           IN EAX,77                        ; I/O 命令
6600101C >  A2 B4E477B1     MOV BYTE PTR DS:[B177E4B4],AL
66001021    C6              ???                              ; 未知命令
66001022    E4 77           IN AL,77                         ; I/O 命令
66001024 >  B8 16E47737     MOV EAX,3777E416
66001029    38E5            CMP CH,AH
6600102B    77 6B           JA SHORT <&KERNEL32.LeaveCriticalSection>
6600102D    90              NOP
6600102E    E4 77           IN AL,77                         ; I/O 命令
66001030 >  E1 60           LOOPDE SHORT MSVBVM60.66001092
66001032    E5 77           IN EAX,77                        ; I/O 命令
66001034 >  8BB0 E5779725   MOV ESI,DWORD PTR DS:[EAX+259777E5]
6600103A    E5 77           IN EAX,77                        ; I/O 命令
```

****************************************************************

点右键，搜索\当前模块中的名称，看看后找到 `vbaVarCopy` (当然有兴趣的朋友可以用上面的 `vbaSriComp` 或邻近的几个功能相近的试试，反正我也没用过) 这个时候不要下断，返回到 `CPU` 页（上面的代码处），用 `Shift+f9` 运行，进入注册码填写的地方，随手写入 `123456654987` 不要点确定（这个时候最好让这个破软件停下来，不要读词了）再回到 `vbaVarCopy` 这个地方下断点，这个时候才可以点确定，这之后会停在如下地方,这之后用 `F8` 走

****************************************************************

```
66106BF8 >  51              PUSH ECX
66106BF9    A1 7CEE1066     MOV EAX,DWORD PTR DS:[6610EE7C]
66106BFE    53              PUSH EBX
66106BFF    55              PUSH EBP
66106C00    56              PUSH ESI
66106C01    57              PUSH EDI
66106C02    8BDA            MOV EBX,EDX
66106C04    8BF1            MOV ESI,ECX
66106C06    50              PUSH EAX
66106C07    FF15 B8100066   CALL DWORD PTR DS:[<&KERNEL32.TlsGetValu>; kernel32.TlsGetValue
66106C0D    66:8B2B         MOV BP,WORD PTR DS:[EBX]
66106C10    8D78 50         LEA EDI,DWORD PTR DS:[EAX+50]
66106C13    66:8B06         MOV AX,WORD PTR DS:[ESI]
66106C16    66:83FD 08      CMP BP,8
66106C1A    73 26           JNB SHORT MSVBVM60.66106C42
66106C1C    66:3D 0800      CMP AX,8
66106C20    73 20           JNB SHORT MSVBVM60.66106C42
66106C22    8B13            MOV EDX,DWORD PTR DS:[EBX]
66106C24    8B43 04         MOV EAX,DWORD PTR DS:[EBX+4]
66106C27    8BCE            MOV ECX,ESI
66106C29    8911            MOV DWORD PTR DS:[ECX],EDX
66106C2B    8B53 08         MOV EDX,DWORD PTR DS:[EBX+8]
66106C2E    8941 04         MOV DWORD PTR DS:[ECX+4],EAX
66106C31    8B43 0C         MOV EAX,DWORD PTR DS:[EBX+C]
66106C34    8951 08         MOV DWORD PTR DS:[ECX+8],EDX
66106C37    8941 0C         MOV DWORD PTR DS:[ECX+C],EAX
66106C3A    8BC6            MOV EAX,ESI
66106C3C    5F              POP EDI
```

****************************************************************

```
66106CBF    3BDE            CMP EBX,ESI
66106CC1    0F84 04010000   JE MSVBVM60.66106DCB
66106CC7    8BCE            MOV ECX,ESI
66106CC9    E8 7E7DFFFF     CALL MSVBVM60.__vbaFreeVar
66106CCE    8B5B 08         MOV EBX,DWORD PTR DS:[EBX+8]
66106CD1    66:C706 0800    MOV WORD PTR DS:[ESI],8
66106CD6    85DB            TEST EBX,EBX
66106CD8    C746 08 0000000>MOV DWORD PTR DS:[ESI+8],0
66106CDF    0F84 E6000000   JE MSVBVM60.66106DCB
66106CE5    75 1C           JNZ SHORT MSVBVM60.66106D03
66106CE7    33C0            XOR EAX,EAX
66106CE9    50              PUSH EAX
66106CEA    53              PUSH EBX
66106CEB    FF15 F4190066   CALL DWORD PTR DS:[<&OLEAUT32.#150>]      ; OLEAUT32.SysAllocStringByteLen
```

到这里就会看到我刚才写入的“`123456987`”了   `66106CD8    C746 08 0000000>MOV DWORD PTR DS:[ESI+8],0`

不要管它坚持就是胜利，接用着 `F8` 走几步，又来到如下地方

```
00424C5C   .  C785 04FFFFFF>MOV DWORD PTR SS:[EBP-FC],wordcard.0040B>
00424C66   .  C785 FCFEFFFF>MOV DWORD PTR SS:[EBP-104],8
00424C70   .  8D95 FCFEFFFF LEA EDX,DWORD PTR SS:[EBP-104]
00424C76   .  8D8D 3CFFFFFF LEA ECX,DWORD PTR SS:[EBP-C4]
00424C7C   .  FF15 74124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarDu>;   MSVBVM60.__vbaVarDup
00424C82   .  8D95 0CFFFFFF LEA EDX,DWORD PTR SS:[EBP-F4]
00424C88   .  52              PUSH EDX
00424C89   .  8D85 1CFFFFFF LEA EAX,DWORD PTR SS:[EBP-E4]
```

到这一行就可以在数据窗口看到真正的注册码了

`00424C5C   .  C785 04FFFFFF>MOV DWORD PTR SS:[EBP-FC],wordcard.0040B>`

如下

`0040BC2C=wordcard.0040BC2C`

`Stack SS:[0012F534]=0018FA4C, (UNICODE "77p4mf81r8357")`

这个 `77p4mf81r8357` 就是注册码了，我的机器码是 `27090435875640`

你学会了吗？

我写这个的目的只是想给破解无门的初学者一点鼓舞，我也是一个初学者，`2004`，`6`，`14` 才拥有了第一台属于自己的电脑
。但请不要误会我学电脑却有 5 年多了，都是在网吧和书店学会的（包括 `JAVA`，`C` 之类），哥们没见过像我这样的网吧狂人吧，那个网吧老板也怕了我，呵呵！！经常搞得不能开机。顺便说一下我也是同济医学院的，不知看雪老大是不是在武汉读的，最后：请记住坚持就会成功！

网管，我以前申请的 `doa007` (也可能是 `doc007`), 不知为什么能进论坛但不能发帖，我比较喜欢以前的帐号

`4dd466d0e1e0cc123afbe504da2462ca`
