
# recover 4 all 1.04

> 原文：[`www.pediy.com/kssd/pediy04/kanxue098.htm`](https://www.pediy.com/kssd/pediy04/kanxue098.htm)

破解者: 我不说

难度: 容易

程序: `Recover 4 all v1.04`

工具：`TRW2000 v1.22`, `W32Dasm`, `eXeScope`.

过程：

运行程序，出现 `NAG` 窗口。退出程序，运行 `eXeScope`，查找 `NAG` 窗口，是 `dialog: 163`, 退出 `eXeScope`。用 `W32Dasm` 打开 `Rec4all.exe`, 查找 `DialogID_00A3`, 为什么？因为 `dec(163)=hex(A3).` 见到：

`* Referenced by a CALL at Address:`

`:0040F583`

```
:00411440 8B442404              mov eax, dword ptr [esp+04]
:00411444 56                    push esi
:00411445 50                    push eax
:00411446 8BF1                  mov esi, ecx
* Possible Reference to Dialog: DialogID_00A3
:00411448 68A3000000            push 000000A3
:0041144D E8EB780100            call 00428D3D
:00411452 C706A03E4400          mov dword ptr [esi], 00443EA0
:00411458 8BC6                  mov eax, esi
:0041145A 5E                    pop esi
:0041145B C20400                ret 0004
```

向上寻找 `0040F583`，

`* Referenced by a CALL at Address:`

`:0040AE7E`

```
:0040F560 6AFF                  push FFFFFFFF
:0040F562 68D8144400            push 004414D8
:0040F567 64A100000000          mov eax, dword ptr fs:[00000000]
:0040F56D 50                    push eax
:0040F56E 64892500000000        mov dword ptr fs:[00000000], esp
:0040F575 83EC6C                sub esp, 0000006C
:0040F578 E893F4FFFF            call 0040EA10
:0040F57D 6A00                  push 00000000
:0040F57F 8D4C2404              lea ecx, dword ptr [esp+04]
:0040F583 E8B81E0000            call 00411440
:0040F588 8D4C2400              lea ecx, dword ptr [esp]
:0040F58C C744247400000000      mov [esp+74], 00000000
:0040F594 E859980100            call 00428DF2
:0040F599 8D4C2400              lea ecx, dword ptr [esp]
:0040F59D C7442474FFFFFFFF      mov [esp+74], FFFFFFFF
:0040F5A5 E889940100            call 00428A33
:0040F5AA 8B4C246C              mov ecx, dword ptr [esp+6C]
:0040F5AE 64890D00000000        mov dword ptr fs:[00000000], ecx
:0040F5B5 83C478                add esp, 00000078
:0040F5B8 C3                    ret
```

再向上找 `0040AE7E`，

```
:0040AE6D 8BCD                  mov ecx, ebp
:0040AE6F E89C3B0000            call 0040EA10
:0040AE74 399DD0050000          cmp dword ptr [ebp+000005D0], ebx
:0040AE7A 7522                  jne 0040AE9E
:0040AE7C 8BCD                  mov ecx, ebp
:0040AE7E E8DD460000            call 0040F560
:0040AE83 391DB84E4600          cmp dword ptr [00464EB8], ebx
:0040AE89 0F858A000000          jne 0040AF19
:0040AE8F C7853002000001000000  mov dword ptr [ebp+00000230], 00000001
:0040AE99 E90A010000            jmp 0040AFA8
```

运行 `trw2000` 设置断点在 `0040AE74` 处，编辑 `ebp+5D0` 处的内存数值为非0，`F5` 后，程序显示注册成功。

重新在 `ebp+5D0` 内存处设断后运行程序，拦截如下

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0040ACE3(C)`

```
:0040AD6D 8BCD                  mov ecx, ebp
:0040AD6F 895D58                mov dword ptr [ebp+58], ebx
:0040AD72 89B5E4430600          mov dword ptr [ebp+000643E4], esi
:0040AD78 899D30020000          mov dword ptr [ebp+00000230], ebx
:0040AD7E 899D38020000          mov dword ptr [ebp+00000238], ebx
:0040AD84 899D3C020000          mov dword ptr [ebp+0000023C], ebx
:0040AD8A 899DD0050000          mov dword ptr [ebp+000005D0], ebx　《－－
:0040AD90 C7455CFFFFFFFF        mov [ebp+5C], FFFFFFFF
:0040AD97 899D88000000          mov dword ptr [ebp+00000088], ebx
:0040AD9D 899D84000000          mov dword ptr [ebp+00000084], ebx
:0040ADA3 899D80000000          mov dword ptr [ebp+00000080], ebx
:0040ADA9 895D7C                mov dword ptr [ebp+7C], ebx
:0040ADAC 895D78                mov dword ptr [ebp+78], ebx
:0040ADAF 895D74                mov dword ptr [ebp+74], ebx
:0040ADB2 895D70                mov dword ptr [ebp+70], ebx
:0040ADB5 895D6C                mov dword ptr [ebp+6C], ebx
:0040ADB8 895D68                mov dword ptr [ebp+68], ebx
:0040ADBB 895D64                mov dword ptr [ebp+64], ebx
```

修改 `0040AD8A` 处的代码为 `89B5D0050000` 后，运行程序，爆破成功。

我研究了很长时间，也没把它的注册算法搞清楚，希望网上的各位高手，能给予指点，先谢过啦！

`superboss`

## `Recover4All v1.03` 的破解

工具: `TRW2000` 和 `Wdasm v8.93`

目标说明: 每个人一定都有曾不小心误删文件与目录的经验，通常要恢复误删文件有时是相当困难的，不妨先试试 `Recover 4 all` 看看，它能将刚刚所删除的文件找出来，但必须在 `Win95/98` 下所删除的文件，它才能找出并救回来.

难度：中级?

\=======================================================

引子：这是一篇破解讨论性的文章，仅作为大家深入讨论的依据。

运行程序，输入 `Registration Name:sUpErbOss` `Registration Code:1122334455`

按 `Ctrl-N` 切换到 `TRW2000` 下，打"BPX HMEMCPY"，点击“OK”按钮，被拦下, 打"BD \*"。

按几次 `F12` 键，回到 `Rec4All` 模组下，再一直按 `F10` 键，直到下面所指的重点(2)处：

```
:00411DE0 51                    push ecx
:00411DE1 8BCD                  mov ecx, ebp
:00411DE3 E898A9FFFF            call 0040C780
:00411DE8 50                    push eax
:00411DE9 8D8C247C030000        lea ecx, dword ptr [esp+0000037C]
:00411DF0 C684243413000001      mov byte ptr [esp+00001334], 01
:00411DF8 E8B3220000            call 004140B0
:00411DFD 8D8C24500B0000        lea ecx, dword ptr [esp+00000B50]
:00411E04 C684243013000000      mov byte ptr [esp+00001330], 00
:00411E0C E8EF220000            call 00414100
:00411E11 8D83E8030000          lea eax, dword ptr [ebx+000003E8]
:00411E17 8D9424500B0000        lea edx, dword ptr [esp+00000B50]
:00411E1E 50                    push eax
:00411E1F 52                    push edx
:00411E20 8BCD                  mov ecx, ebp
:00411E22 E839B8FFFF            call 0040D660
:00411E27 8DB340080000          lea esi, dword ptr [ebx+00000840]
:00411E2D 50                    push eax
:00411E2E 8BCE                  mov ecx, esi
:00411E30 C684243413000002      mov byte ptr [esp+00001334], 02
:00411E38 E873220000            call 004140B0
:00411E3D 8D8C24500B0000        lea ecx, dword ptr [esp+00000B50]
:00411E44 C684243013000000      mov byte ptr [esp+00001330], 00
:00411E4C E8AF220000            call 00414100
:00411E51 81ECD8070000          sub esp, 000007D8
:00411E57 8BCC                  mov ecx, esp
:00411E59 89A424F4070000        mov dword ptr [esp+000007F4], esp
:00411E60 56                    push esi
:00411E61 E82A220000            call 00414090
:00411E66 8D8C24500B0000        lea ecx, dword ptr [esp+00000B50]
:00411E6D E8FE200000            call 00413F70  <---重点(1)!!(比对升级码)
:00411E72 85C0                  test eax, eax
:00411E74 0F858C000000          jne 00411F06
.
.
.
:00411F93 E8E8A7FFFF            call 0040C780 <----此 Call 根据注册码经过变形算出一个数
:00411F98 50                    push eax
:00411F99 8D8C247C030000        lea ecx, dword ptr [esp+0000037C]
:00411FA0 C684243413000003      mov byte ptr [esp+00001334], 03
:00411FA8 E803210000            call 004140B0
:00411FAD 8D8C24500B0000        lea ecx, dword ptr [esp+00000B50]
:00411FB4 C684243013000000      mov byte ptr [esp+00001330], 00
:00411FBC E83F210000            call 00414100
:00411FC1 8D83E8030000          lea eax, dword ptr [ebx+000003E8]
:00411FC7 8D8C24500B0000        lea ecx, dword ptr [esp+00000B50]
:00411FCE 50                    push eax
:00411FCF 51                    push ecx
:00411FD0 8BCD                  mov ecx, ebp
:00411FD2 E889B6FFFF            call 0040D660 <----此 Call 根据 Name 经过变形算出一个数
```
