
# 打造对抗 CMPXCHG8G 的 OD

> 原文：[`www.pediy.com/kssd/pediy06/pediy6460.htm`](https://www.pediy.com/kssd/pediy06/pediy6460.htm)

【破文作者】 simonzh2000  
【使用工具】 `Ollydbg1.10B` 英文版  
【破解平台】 `Win2000SP4` English  
【软件名称】 `Ollydbg1.10B` 英文修改版  
【软件简介】 感谢 OD 的作者提供这么好的免费软件。  
【修改声明】 现在好多壳软件都使用反调试技术, 我们需要对 OD 动点手术,才能顺利调试。  
我们整天用 OD 调试其他软件, 今天也让他尝尝被调试的滋味. hehe.

## 1. 修改 `Window Caption` 和 `ClassName`

这一步就不多说了, 将 `Ollydbg.exe` 拷贝到 `CMD.EXE`, 用 `UltraEdit` 打开 `CMD.EXE`,  
搜索 `OllyDBG` 字符, 替换为个性化字符, 下面以 `ZSMOZSM` 为例

## 2. 复制 `Ollydgb.ini` 到 `ZSMOZSM.ini`

这样一来, 你原来的配置就生效了, 以后也不需要 `Ollydbg.exe` 了

## 3. 最近好多壳开始利用 `CMPXCHG8B` 非法指令, 弹出警告, 去掉他.

From Intel

\===============================================================================================
`CMPXCHG8B`—Compare and Exchange 8 Bytes
Opcode Instruction Description
`0F C7 /1 m64` `CMPXCHG8B m64` Compare `EDX:EAX` with `m64`. If equal, set `ZF` and load
`ECX:EBX` into `m64`. Else, clear `ZF` and load `m64` into `EDX:EAX`.
Description
Compares the 64-bit value in `EDX:EAX` with the operand `destination operand`. If the values
are equal, the 64-bit value in `ECX:EBX` is stored in the destination operand. Otherwise, the
value in the destination operand is loaded into `EDX:EAX`. The destination operand is an 8-byte
memory location. For the `EDX:EAX` and `ECX:EBX` register pairs, `EDX` and `ECX` contain the
high-order 32 bits and `EAX` and `EBX` contain the low-order 32 bits of a 64-bit value.
This instruction can be used with a `LOCK` prefix to allow the instruction to be executed atomically.
To simplify the interface to the processor's bus, the destination operand receives a write
cycle without regard to the result of the comparison. The destination operand is written back if
the comparison fails; otherwise, the source operand is written into the destination. `(The`
processor never produces a locked read without also producing a locked write.)
Intel Architecture Compatibility
This instruction is not supported on Intel processors earlier than the Pentium processors.
\================================================================================================

从上面最后一行, 我们可以看到, 这条指令不能在 Pentium 以前的 CPU 上使用,
所以 OD 有一个警告框. 其实是多此一举, 现在谁还用 486 啊?
让我们去掉他.

`Linson` 的新版仙剑刚刚出炉, 里面也有 `CMPXCHG8B`, 我们就用他吧, `Main.exe`.

运行 `Ollydbg.exe`, 打开 `CMD.EXE`, `F9`, 现在桌面有两个 OD,
`Ollydbg` 是调试器, `CMD.exe` 就是被调试的程序, 别搞错了.

在 `Ollydbg` 里 `Search For Name(label) in Current Module -- Name` 窗口, `MessageBoxA`, Enter -- `References` 窗口
有好多个对 `MessageBoxA` 的调用, `Set BreakPoint on Every Command`

切换到 `CMD.EXE`, 打开 `Main.EXE`, 被 `Ollydbg.exe` 断在这里

```
0045CC20  |.  6A 30         PUSH 30                                  ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
0045CC22  |.  68 6A9F4B00   PUSH CMD.004B9F6A                        ; |Title = "Entry Point Alert"
0045CC27  |.  50           PUSH EAX                                 ; |Text
0045CC28  |.  52           PUSH EDX                                 ; |hOwner => 000A00CC ('zsmozsm - Main.exe - [CPU]',class='zsmozsm')
0045CC29  |.  E8 84100500   CALL <JMP.&USER32.MessageBoxA>           ; MessageBoxA
```

这个就是 `"Entry Point Alert"` 对话框, `F2` 取消断点, `F9`, 切换到 `CMD.EXE`, 点 OK, 停在 `Main.exe` 入口点 `412060`, `F9`,

再次被 `OllyDBG.EXE` 断下, `4340B8`, 就是那个烦人的警告了, 去掉其他的断点.

```
0043402A  |.  6A 00         PUSH 0                                   ; /Arg7 = 00000000
0043402C  |.  8D8D 50F4FFFF LEA ECX,DWORD PTR SS:[EBP-BB0]           ; |
00434032  |.  6A 01         PUSH 1                                   ; |Arg6 = 00000001
00434034  |.  51           PUSH ECX                                 ; |Arg5 = 12E8B8
00434035  |.  6A 00         PUSH 0                                   ; |Arg4 = 00000000
00434037  |.  53           PUSH EBX                                 ; |Arg3 = 00412167 (Main.exe 中 CMPXCHG8B 的 VA)
00434038  |.  8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]            ; |
0043403B  |.  8D95 CCFDFFFF LEA EDX,DWORD PTR SS:[EBP-234]           ; |
00434041  |.  50           PUSH EAX                                 ; |Arg2 = 10h, 机器码长度最长 16 字节
00434042  |.  52           PUSH EDX                                 ; |Arg1 = 12F234, 放 Main.EXE 中 412167 开始的 16 字节
00434043  |.  E8 741EFEFF   CALL CMD._Disasm                         ; _Disasm
                           ;   // 这个 CALL 将分析 412167 开始的 16 字节,
                                                                   ;   // 找出最合理的一条机器指令, 返回机器指令长度
00434048  |.  83C4 1C       ADD ESP,1C
0043404B  |.  8945 E8       MOV DWORD PTR SS:[EBP-18],EAX            ;  // 指令长度
0043404E  |.  837D 10 00    CMP DWORD PTR SS:[EBP+10],0              ;  // 判断是不是危险指令, 见 _Disasm 原程序.
00434052  |.  74 76         JE SHORT CMD.004340CA                    ;  // 将这条语句改成 JMP 4340CA, (74->EB), 就可以了
00434054  |.  F685 8DFAFFFF>TEST BYTE PTR SS:[EBP-573],30
0043405B  |.  74 6D         JE SHORT CMD.004340CA
0043405D  |.  80BD 54F8FFFF>CMP BYTE PTR SS:[EBP-7AC],0
00434064  |.  75 04         JNZ SHORT CMD.0043406A
00434066  |.  33C0          XOR EAX,EAX
00434068  |.  EB 15         JMP SHORT CMD.0043407F
0043406A  |>  80BD 54F9FFFF>CMP BYTE PTR SS:[EBP-6AC],0
00434071  |.  75 07         JNZ SHORT CMD.0043407A
00434073  |.  B8 01000000   MOV EAX,1
00434078  |.  EB 05         JMP SHORT CMD.0043407F
0043407A  |>  B8 02000000   MOV EAX,2
0043407F  |>  C1E0 08       SHL EAX,8
00434082  |.  8D95 54F7FFFF LEA EDX,DWORD PTR SS:[EBP-8AC]
00434088  |.  03C2          ADD EAX,EDX
0043408A  |.  53           PUSH EBX                                 ; /Arg4
0043408B  |.  50           PUSH EAX                                 ; |Arg3
0043408C  |.  8D8D DCFDFFFF LEA ECX,DWORD PTR SS:[EBP-224]           ; |
00434092  |.  68 B4484B00   PUSH CMD.004B48B4                        ; |Arg2 = 004B48B4 ASCII "%s Do you REALLY want to execute this code at address %08X?"
00434097  |.  51           PUSH ECX                                 ; |Arg1
00434098  |.  E8 2B130700   CALL CMD.004A53C8                        ; CMD.004A53C8, sprintf()生成字符串
0043409D  |.  83C4 10       ADD ESP,10
004340A0  |.  8D85 DCFDFFFF LEA EAX,DWORD PTR SS:[EBP-224]
004340A6  |.  8B15 F4174D00 MOV EDX,DWORD PTR DS:[4D17F4]
004340AC  |.  68 34010000   PUSH 134                                 ; /Style = MB_YESNO|MB_ICONEXCLAMATION|MB_DEFBUTTON2|MB_APPLMODAL
004340B1  |.  68 F0484B00   PUSH CMD.004B48F0                        ; |Title = "Dangerous command"
004340B6  |.  50           PUSH EAX                                 ; |Text
004340B7  |.  52           PUSH EDX                                 ; |hOwner => 000A00CC ('zsmozsm - Main.exe - [CPU - m...',class='zsmozsm')
004340B8  |.  E8 F59B0700   CALL <JMP.&USER32.MessageBoxA>           ; MessageBoxA
004340BD  |.  83F8 06       CMP EAX,6                                ; // 用户点 Yes, 继续执行
004340C0      74 08         JE SHORT CMD.004340CA
004340C2  |.  83C8 FF       OR EAX,FFFFFFFF
004340C5  |.  E9 F8060000   JMP CMD.004347C2
004340CA  |>  8BBD 54FAFFFF MOV EDI,DWORD PTR SS:[EBP-5AC]           ; // 这里执行
004340D0  |.  81E7 F0000000 AND EDI,0F0
004340D6  |.  837D 10 02    CMP DWORD PTR SS:[EBP+10],2
```

修改 `434052` 处 `74` 为 `EB`, `Copy to Execute`, 保存文件 `CMD2.EXE`, 退出 `ollydbg.EXE`.

用 `CMD2.EXE` 打开 `Main.EXE`, 忽略所有异常, `F9`,

在 `412167` 处异常, `ExceptionCode = C000001E`

`Debug Option -- Exception -- Add last Exception -- Ignore custom Exception`,

`Shift + F9`, 这个异常要经过好多次, 等一会儿, 仙剑就运行了.

`4dd466d0e1e0cc123afbe504da2462ca`
