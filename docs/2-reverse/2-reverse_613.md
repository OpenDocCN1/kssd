
# 珊瑚虫外挂原理分析【原创】

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-387.htm`](https://www.pediy.com/kssd/pediy07/pediy7-387.htm)

作者: RunJin  
runjin000@yahoo.com.cn

分析工具: `softice`, `OD`

我的操作系统: `win 2000 sp4`, `win 98`

分析对象: `QQ2005 贺岁版` 珊瑚虫外挂

为了方便分析,我先把 `CoralQQ.dll` 和 `coralqq.exe` 先脱壳,这两个文件加的都是超弱壳,脱壳过程不在这里多说.

由于 `win nt` 平台和 `win9x` 的差异,珊瑚虫外挂因系统的不同而分开两种工作原理.

我们都知道,珊瑚外挂是给 QQ 的主程序 `QQ.exe` 外挂了一个 `Coralqq.dll`,而完成这个工作的就是 `coralqq.exe`.要加载一个 dll 文件必须是 `qq.exe` 自己的某个线程执行 `LoadLibraryA`,并且以 `coralqq.dll` 为参数,但 `qq.exe` 本来根本没有这样的一段代码,怎么办?答案很简单,既然它没有,那就让我们帮它加上去,`coralqq.exe` 就是利用 `WriteProcessMemory` 帮 `QQ.exe` 加上了这样一段代码,并且改变程序流程,让 `qq.exe` 先加载 `coralqq.dll` 再跳到原来的 `oep` 执行.

想了一下,比较合理的方法有:

1. 在 `nt` 内核的系统上,利用远程线程的方法,首先用 `CreateProcess` 创建 `QQ.exe` 的进程,同时创建了主线程,再用 `VirtualAllocEx` 在 `qq.exe` 中申请一块内存,然后把我们的代码写进去,最后用 `CreateRemoteThread` 在 `qq` 的进程中创建另外一个线程,其开始执行的地方正是我们自己加入的代码首地址.
2. 先用 `CreateProcess` 创建 `QQ` 进程和主线程,在 `QQ` 的内存空间中找个合理的地方写入我们的代码,然后修改某个地方(例如 `OEP`)使 `QQ` 先跳去执行我们写入的代码,加载 `coralqq.dll`,修复原来改过的 `oep`,然后跳回 `oep` 让程序正常执行.

下面就让我们分析一下珊瑚虫的外挂是怎样做到的,首先分析 `nt` 平台上的运行过程.

用 `OD` 载入脱壳后的 `coralqq.exe`,停在下面:

```
00418E2C  >/$   55                  push ebp
00418E2D   |.   8BEC                mov ebp,esp
00418E2F   |.   83C4 F0             add esp,-10
00418E32   |.   B8 648D4100         mov eax,CoralQQ.00418D64
00418E37   |.   E8 A4BAFEFF         call CoralQQ.004048E0
00418E3C   |.   A1 F49D4100         mov eax,dword ptr ds:[419DF4]
00418E41   |.   33D2                xor edx,edx
00418E43   |.   E8 30F4FFFF         call CoralQQ.00418278
00418E48   |.   A1 F49D4100         mov eax,dword ptr ds:[419DF4]
00418E4D   |.   8B00                mov eax,dword ptr ds:[eax]
00418E4F   |.   83C0 18             add eax,18
00418E52   |.   E8 F9C6FEFF         call CoralQQ.00405550
00418E57   |.   E8 08A9FEFF         call CoralQQ.00403764
```

然后下断点: `bp CreateRemoteThread`,然后运行,结果发现,`OD` 根本没有断下,证明珊瑚虫的外挂没有使用远程线程的方法,难道它是用了第二种方法?我们再验证一下:

首先在 `OD` 中 `bp CreateProcess`,断下,反回后,用 `winhex` 打开 `QQ.exe` 的内存,把 `QQ.exe` 的 `OEP` 处改为 `CC`,接着在 `softice` 中 `bpint 3`,在 `OD` 中按 F9 运行,接着就是中断在 `softice` 中,把 `oep(464b58)` 处还原,在再 `softice` 下 `bpm 464b58`,然后结束程序,再运行 `coralqq.exe` 发现自始至终那个 `bpm` 断点还是没有断下,证明珊瑚虫外挂在 2000 下也没有修改 `QQ.exe` 的 `oep` 来改变程序流程,晕.没办法,只好在 `OD` 中下 `WriteProcessMemory` 断点,看看它究竟修改了哪里.

```
00231012    57             push edi
00231013    8B4424 24      mov eax,dword ptr ss:[esp+24]
00231017    A3 70302300    mov dword ptr ds:[233070],eax
0023101C    8305 70302300 04 add dword ptr ds:[233070],4
00231023    E8 5C000000    call <jmp.&kernel32.CreateProcessA>;创建 QQ 的进程
```

上面创建后的 QQ 主线程是暂停的,不会马上执行,在 `win9x` 下也是这样.

第一次中断时堆栈显示:

```
0012EBDC   0040C75C   /CALL 到 WriteProcessMemory 来自 CoralQQ.0040C756
0012EBE0   00000018   |hProcess = 00000018
0012EBE4   5F000000   |Address = 5F000000
0012EBE8   00CC0F44   |Buffer = 00CC0F44
0012EBEC   000000D2   |BytesToWrite = D2 (210.)
0012EBF0   0012EC68   \\pBytesWritten = 0012EC68
```

第二次:

```
0012EC70   00416231   /CALL 到 WriteProcessMemory 来自 CoralQQ.0041622C
0012EC74   00000018   |hProcess = 00000018
0012EC78   5F010000   |Address = 5F010000
0012EC7C   0012EE9F   |Buffer = 0012EE9F
0012EC80   00000025   |BytesToWrite = 25 (37.)
0012EC84   0012EFD0   \\pBytesWritten = 0012EFD0
```

第三次:

```
0012EC70   004162CB   /CALL 到 WriteProcessMemory 来自 CoralQQ.004162C6
0012EC74   00000018   |hProcess = 00000018
0012EC78   77F84BC0   |Address = 77F84BC0
0012EC7C   0012EFC3   |Buffer = 0012EFC3
0012EC80   00000005   |BytesToWrite = 5
0012EC84   0012EFD0   \\pBytesWritten = 0012EFD0
```

第四次:

```
0012EC70   0041632B   /CALL 到 WriteProcessMemory 来自 CoralQQ.00416326
0012EC74   00000018   |hProcess = 00000018
0012EC78   5F010025   |Address = 5F010025
0012EC7C   0012EC96   |Buffer = 0012EC96
0012EC80   00000208   |BytesToWrite = 208 (520.)
0012EC84   0012EFD0   \\pBytesWritten = 0012EFD0
```

第五次:

```
0012EC70   0041634B   /CALL 到 WriteProcessMemory 来自 CoralQQ.00416346
0012EC74   00000018   |hProcess = 00000018
0012EC78   5F010000   |Address = 5F010000
0012EC7C   0012EE9F   |Buffer = 0012EE9F
0012EC80   00000025   |BytesToWrite = 25 (37.)
0012EC84   0012EFD0   \\pBytesWritten = 0012EFD0
```

留意第三次中断: `77F84BC0` 对应着: `NtTestAlert` 所在库：`ntdll.dll`

程序每次执行前都要经过 `ntdll.dll` 的 `NtTestAlert` 函数,它使函数首句跳到某个地方,从而改变程序流程.

```
00417E7E    8B45 F4        mov eax,dword ptr ss:[ebp-C]
00417E81    50             push eax
00417E82    E8 DDCDFEFF    call <jmp.&kernel32.ResumeThread>;使得 QQ 的主线程继续执行
```

把 `0012efc3` 处的 `E9` 改为 `CC`,再在 `softice` 中 `bpint 3`,让其中断后跟踪:

```
001B:77F84BC0  E93BB408E7          JMP       5F010000
```

单步执行跟踪

```
001B:5F010000  B80000015F          MOV       EAX,5F010000
001B:5F010005  B90000005F          MOV       ECX,5F000000
001B:5F01000A  FFD1               CALL      ECX
```

此时 `ecx` 的值为 `5F000000`

```
001B:5F000000  55               PUSH      EBP
001B:5F000001  8BEC              MOV       EBP,ESP
001B:5F000003  83C4EC            ADD       ESP,-14
001B:5F000006  53               PUSH      EBX
001B:5F000007  56               PUSH      ESI
001B:5F000008  8BD8              MOV       EBX,EAX
001B:5F00000A  896DFC            MOV       [EBP-04],EBP
001B:5F00000D  8B431C            MOV       EAX,[EBX+1C]
001B:5F000010  8B55FC            MOV       EDX,[EBP-04]
001B:5F000013  83C204            ADD       EDX,04
001B:5F000016  8902              MOV       [EDX],EAX
001B:5F000018  8B431C            MOV       EAX,[EBX+1C]
001B:5F00001B  8945EC            MOV       [EBP-14],EAX
001B:5F00001E  C745FC05000000    MOV       DWORD PTR [EBP-04],00000005
001B:5F000025  8D45F8            LEA       EAX,[EBP-08]
001B:5F000028  50               PUSH      EAX
001B:5F000029  6A40              PUSH      40
001B:5F00002B  8D45FC            LEA       EAX,[EBP-04]
001B:5F00002E  50               PUSH      EAX
001B:5F00002F  8D45EC            LEA       EAX,[EBP-14]
001B:5F000032  50               PUSH      EAX
001B:5F000033  6AFF              PUSH      FF
001B:5F000035  FF15C600005F      CALL      [5F0000C6]
001B:5F00003B  8B431C            MOV       EAX,[EBX+1C]
001B:5F00003E  8B5320            MOV       EDX,[EBX+20]
001B:5F000041  8910              MOV       [EAX],EDX ;恢复 Ntdll.NtTestAlert 入口处
001B:5F000043  8A5324            MOV       DL,[EBX+24]
001B:5F000046  885004            MOV       [EAX+04],DL
001B:5F000049  C745FC05000000    MOV       DWORD PTR [EBP-04],00000005
001B:5F000050  8D45F8            LEA       EAX,[EBP-08]
001B:5F000053  50               PUSH      EAX
001B:5F000054  8B45F8            MOV       EAX,[EBP-08]
001B:5F000057  50               PUSH      EAX
001B:5F000058  8D45FC            LEA       EAX,[EBP-04]
001B:5F00005B  50               PUSH      EAX
001B:5F00005C  8D45EC            LEA       EAX,[EBP-14]
001B:5F00005F  50               PUSH      EAX
001B:5F000060  6AFF              PUSH      FF
001B:5F000062  FF15CA00005F      CALL      [5F0000CA]
001B:5F000068  8B7318            MOV       ESI,[EBX+18]
001B:5F00006B  4E               DEC       ESI
001B:5F00006C  85F6              TEST      ESI,ESI
001B:5F00006E  7C4C              JL        5F0000BC
001B:5F000070  46               INC       ESI
001B:5F000071  8D4325            LEA       EAX,[EBX+25]
001B:5F000074  8BD8              MOV       EBX,EAX
001B:5F000076  33D2              XOR       EDX,EDX
001B:5F000078  8BC3              MOV       EAX,EBX
001B:5F00007A  66833800          CMP       WORD PTR [EAX],00
001B:5F00007E  740C              JZ        5F00008C
001B:5F000080  42               INC       EDX
```



## 珊瑚虫 QQ 外挂逆向分析

### 一、内存映射与 LoadLibraryA 调用分析

以下是 `win9x` 环境下 `Coralqq.exe` 运行时的部分汇编代码片段，涉及模块加载逻辑：

```
001B:5F000081  83C002                ADD       EAX,02
001B:5F000084  81FA03010000        CMP       EDX,00000103
001B:5F00008A  75EE                JNZ       5F00007A
001B:5F00008C  8BC2                MOV       EAX,EDX
001B:5F00008E  03C0                ADD       EAX,EAX
001B:5F000090  668945F0            MOV       [EBP-10],AX
001B:5F000094  6683C002            ADD       AX,02
001B:5F000098  668945F2            MOV       [EBP-0E],AX
001B:5F00009C  8BC3                MOV       EAX,EBX
001B:5F00009E  8945F4              MOV       [EBP-0C],EAX
001B:5F0000A1  8D45FC              LEA       EAX,[EBP-04]
001B:5F0000A4  50                  PUSH      EAX
001B:5F0000A5  8D45F0              LEA       EAX,[EBP-10]
001B:5F0000A8  50                  PUSH      EAX
001B:5F0000A9  6A00                PUSH      00
001B:5F0000AB  6A00                PUSH      00
001B:5F0000AD  FF15CE00005F        CALL      [5F0000CE] ;call LoadLibraryA
001B:5F0000B3  81C308020000        ADD       EBX,00000208
001B:5F0000B9  4E                  DEC       ESI
001B:5F0000BA  75BA                JNZ       5F000076
001B:5F0000BC  5E                  POP       ESI
001B:5F0000BD  5B                  POP       EBX
001B:5F0000BE  8BE5                MOV       ESP,EBP
001B:5F0000C0  5D                  POP       EBP
001B:5F0000C1  C3                  RET
001B:5F0000C2  0000                ADD       [EAX],AL
001B:5F0000C4  0000                ADD       [EAX],AL
001B:5F0000C6  C4BFF877C4BF        LES       EDI,[EDI+BFC477F8]
001B:5F0000CC  F8                  CLC
001B:5F0000CD  7761                JA        5F000130
001B:5F0000CF  32F8                XOR       BH,AL
001B:5F0000D1  7700                JA        5F0000D3
```

我们再用 `OD` 载入 `Coralqq.exe` 进行深入分析：

```
00415C9D  mov dword ptr ss:[ebp-C],ecx
00415CA0  mov dword ptr ss:[ebp-4],edx
00415CA3  mov dword ptr ss:[ebp-8],eax
00415CA6  xor ebx,ebx
00415CA8  call <jmp.&kernel32.GetVersion> ;判断操作系统
00415CAD  test eax,80000000
00415CB2  je short CORALQQ.00415CFE
00415CB4  xor edi,edi
00415CB6  xor esi,esi
00415CB8  jmp short CORALQQ.00415CE7
00415CBA  /cmp esi,dword ptr ss:[ebp-24]
00415CBD  |je short CORALQQ.00415CE2
00415CBF  |mov eax,dword ptr ss:[ebp-4]
00415CC2  |mov edx,dword ptr ss:[ebp-24]
00415CC5  |mov dword ptr ds:[eax],edx
00415CC7  |cmp dword ptr ss:[ebp-18],1000
00415CCE  |jnz short CORALQQ.00415CDF
00415CD0  |push ebp                      ; /Arg1
00415CD1  |call CORALQQ.00415BD4         ; \CoralQQ.00415BD4
00415CD6  |pop ecx
00415CD7  |test al,al
00415CD9  |je short CORALQQ.00415CDF
00415CDB  |mov bl,1
00415CDD  |jmp short CORALQQ.00415D29
00415CDF  |mov esi,dword ptr ss:[ebp-24]
00415CE2  |mov eax,dword ptr ss:[ebp-1C]
00415CE5  |add edi,eax
00415CE7   push 1C                       ; /BufSize = 1C (28.)
00415CE9  |lea eax,dword ptr ss:[ebp-28] ; |
00415CEC  |push eax                      ; |Buffer
00415CED  |push edi                      ; |Address
00415CEE  |mov eax,dword ptr ss:[ebp-8]    ; |
00415CF1  |push eax                      ; |hProcess
00415CF2  |call <jmp.&kernel32.VirtualQueryEx> ; \VirtualQueryEx 获得内存业面信息
00415CF7  |cmp eax,1C
00415CFA  \je short CORALQQ.00415CBA
00415CFC  jmp short CORALQQ.00415D29
```

上述代码用于判断操作系统版本。如果是 `win9x`，则先跳转到 `415ce7`。接着使用 `VirtualQueryEx` 获取内存状态信息，并通过 `WriteProcessMemory` 写入数据。以下是相关代码：

```
00415A4B  |> \8>lea eax,dword ptr ss:[ebp-8]
00415A4E  |. 5>push eax                      ; /pBytesWritten
00415A4F  |. 5>push edi                      ; |BytesToWrite
00415A50  |. 8>mov eax,dword ptr ss:[ebp-4]      ; |
00415A53  |. 5>push eax                      ; |Buffer
00415A54  |. 5>push esi                      ; |Address
00415A55  |. 5>push ebx                      ; |hProcess
00415A56  |. E>call <jmp.&kernel32.WriteProcessMemory>  ; \WriteProcessMemory
```

此时的堆栈状态如下：

```
0067EA5C   0000000C  |hProcess = 0000000C
0067EA60   83138AAC  |Address = 83138AAC
0067EA64   0067EABC  |Buffer = 0067EABC
0067EA68   00000292  |BytesToWrite = 292 (658.)
0067EA6C   0067EA7C  \pBytesWritten = 0067EA7C
```

程序将缓冲区 `0067EABC` 的 658 字节数据写入 `QQ.exe` 的内存地址 `83138AAC` 处。在 `win98` 下，进程内存中的共享（`MMF`）分区范围是 `0x80000000` ~ `0xbfffffff`。所有的内存映射文件和系统共享 `DLL` 都将加载在此地址范围内。通常这些映射文件和系统共享 `DLL` 位于较高的地址，因此从 `80000000` 开始的一段地址空间往往比较“空闲”。

当然，这并不意味着这些地址可以随意读写。关于 `83138AAC` 这个具体地址值的计算方式尚未完全分析清楚，但存在另一种可行方法：以 `1000h` 大小为单位，从 `80000000` 开始使用 `WriteProcessMemory` 向上写入数据，直到写入成功，证明该段 `1000h` 大小的地址空间可用且足够容纳代码。之所以选择 `1000h` 作为单位大小，是考虑到内存块对齐的问题。

继续跟踪执行流：

```
00415A39  |. 8D45 F8        lea eax,dword ptr ss:[ebp-8]
00415A3C  |. 50             push eax                          ; /pOldProtect
00415A3D  |. 6A 40          push 40                           ; |NewProtect = PAGE_EXECUTE_READWRITE
00415A3F  |. 57             push edi                          ; |Size
00415A40  |. 56             push esi                          ; |Address
00415A41  |. 53             push ebx                          ; |hProcess
00415A42  |. E8 85F2FEFF    call <jmp.&kernel32.VirtualProtectEx>    ; \VirtualProtectEx
```

上述代码修改了 `QQ.exe` 的 `OEP`（Original Entry Point）处的属性，使其变为可读、可写、可执行，为改写 `OEP` 处的代码做准备。

随后，再次中断在 `WriteProcessMemory` 调用处。查看堆栈：

```
0067EA5C   0000000C  |hProcess = 0000000C
0067EA60   00464B58  |Address = 464B58
0067EA64   0067EE46  |Buffer = 0067EE46
0067EA68   00000005  |BytesToWrite = 5
0067EA6C   0067EA7C  \pBytesWritten = 0067EA7C
```

`464B58` 是 `QQ.exe` 的 `OEP`。程序显然通过改变 `OEP` 来重定向程序流程。

接着，使用调试器方法将 `0067EE46` 的第一个字节修改为 `CC`，并在 `SoftICE` 中设置 `bpint 3`，然后中断并跟踪。

在 `SoftICE` 下中断后的执行情况如下：

```
0167:00464B58  E91328CE82          JMP       83147370
0167:83147370  C705584B4600558BEC6A
              MOV       DWORD PTR [00464B58],6AEC8B55 ;马上恢复 OEP 处的代码
0167:8314737A  C6055C4B4600FF      MOV       BYTE PTR [00464B5C],FF
0167:83147381  68FA731483          PUSH      831473FA
0167:83147386  6838000000          PUSH      00000038
0167:8314738B  6A40                PUSH      40
0167:8314738D  FF15F6731483        CALL      [KERNEL32!GlobalAlloc] ;再申请内存
0167:83147393  C700B85077F7        MOV       DWORD PTR [EAX],F77750B8 ;从这里开始一直填入新数据
0167:83147399  C74004BFFFD068      MOV       DWORD PTR [EAX+04],68D0FFBF
0167:831473A0  C7400870731483      MOV       DWORD PTR [EAX+08],83147370
0167:831473A7  C7400CB89348E9      MOV       DWORD PTR [EAX+0C],E94893B8
0167:831473AE  C74010BFFFD0B8      MOV       DWORD PTR [EAX+10],B8D0FFBF
0167:831473B5  C74014584B4600      MOV       DWORD PTR [EAX+14],00464B58
0167:831473BC  C74018FFE0C705      MOV       DWORD PTR [EAX+18],05C7E0FF
0167:831473C3  C7401C584B4600      MOV       DWORD PTR [EAX+1C],00464B58
0167:831473CA  C74020558BEC6A      MOV       DWORD PTR [EAX+20],6AEC8B55
0167:831473D1  C74024C6055C4B      MOV       DWORD PTR [EAX+24],4B5C05C6
0167:831473D8  C740284600FF68      MOV       DWORD PTR [EAX+28],68FF0046
0167:831473DF  C7402CFA731483      MOV       DWORD PTR [EAX+2C],831473FA
0167:831473E6  C7403068380000      MOV       DWORD PTR [EAX+30],00003868
0167:831473ED  C74034006A40FF      MOV       DWORD PTR [EAX+34],FF406A00
0167:831473F4  FFE0                JMP       EAX ;此时 EAX 为 0063059C

0167:0063059C  B85077F7BF          MOV       EAX,KERNEL32!LoadLibraryA ;加载 Coralqq.dll
0167:006305A1  FFD0                CALL      EAX
0167:006305A3  6870731483          PUSH      83147370
0167:006305A8  B89348E9BF          MOV       EAX,COMCTL32!ORD_0049
0167:006305AD  FFD0                CALL      EAX
0167:006305AF  B8584B4600          MOV       EAX,00464B58
```



`0167:006305B4  FFE0                JMP       EAX ;跳回 QQ.exe 的入口点`

分析完毕.

到了这里,我们可以总结一下了,在 `win2000/xp` 下 `coralqq.exe` 先创建 `QQ` 的进程,同时也就创建了 `QQ` 的暂停的主线程,接着往 `QQ` 进程的内存写入代码,修改 `Ntdll.NtTestAlert` 的代码跳让程序跳到自己的代码处执行,在执行的过程中恢复 `Ntdll.NtTestAlert` 处被改了的代码,同时加载 `Coralqq.dll`.

在 `win9x` 下,`coralqq.exe` 先创建 `QQ` 的进程,同时也就创建了 `QQ` 的暂停的主线程,接着往 `QQ.exe` 的内存写数据,改写 `QQ.exe` 的 `oep` 从而达到改变程序流程的目的,让其先执行加入了的代码,加载 `Coralqq.dll`,加载完后再跳到原来的 `QQ.exe` 的 `oep` 继续执行.

其实在 `win2000/xp` 下面完全可以用远程线程的方法来实现加载 `dll` 文件,我后来自己编程实现了外挂这一步.

也许有读者读完这编文章后会问:"加载了 `coralqq.dll` 又有什么用?它是怎样显 `IP` 的?"

其实,要显 `IP` 当然要修改 `QQ` 的内存中的代码,而修改内存代码这一步是在加载 `coralqq.dll` 的时候由 `coralqq.dll` 完成的.也就是说,在执行 `LoadLibrary` 的过程中系统会执行 `corall.dll` 的 `LibMain` ,修改的过程就在 `LibMain` 中完成了!而说到显 `IP` 的原理,抱歉,不在我们这编文章的讨论范围之内,我们只是讨论外挂中"挂"这一步.

下面是我写的代码,可以做出一个顶替 `Coralqq.exe` 的程序,但原理上和 `Coralqq.exe` 有一点点不同.

```asm
.586

.model flat, stdcall

option casemap :none   ; case sensitive

include        windows.inc

include        kernel32.inc

includelib     kernel32.lib

.data

qq db ".\\QQ.exe",0

ikeyname db "qq",0

isecname db "main",0

szBuffer       dw  50 dup(0)

dllname db ".\\CoralQQ.dll",0

szkernel32 db "Kernel32.dll",0

dllin dd 0

szloadlibrary db "LoadLibraryA",0

oaddr dd 0

wriaddr dd 0

ininame db ".\\CoralQQ.ini",0

.data?

align dword

con CONTEXT <>

align dword

con2 CONTEXT <>

stStartUp    STARTUPINFO <?>

stProcInfo    PROCESS_INFORMATION <?>

.code

 start:

 main proc

    local wrisize:dword ;用来存放要写入的代码的大小

    local num:dword

    local oldpro:dword

    local lastwri:dword

    local wribase:dword ;用来存放写入代码的基址

    invoke GetPrivateProfileString,offset isecname,offset ikeyname,\

        offset szloadlibrary,offset szBuffer,\

        sizeof szBuffer,offset ininame    ;从 Coralqq.ini 中获取 QQ 程序的路径

    invoke GetModuleHandle,offset szkernel32

    invoke GetProcAddress,eax,offset szloadlibrary

    mov dllin,eax

    invoke GetStartupInfo,addr stStartUp

    invoke CreateProcess,NULL,offset szBuffer,NULL,NULL,FALSE,\

    CREATE_SUSPENDED,NULL,NULL,addr stStartUp,addr stProcInfo ;创建 QQ 进程

    cmp eax,0

    jz exit

    invoke GetVersion

    test eax,080000000h ;判断操作系统类型

    jnz win9x

    invoke VirtualAllocEx,stProcInfo.hProcess,NULL,01000h,MEM_COMMIT,\

    PAGE_EXECUTE_READWRITE ;在 QQ 内存中分配空间

    mov oaddr,eax

    invoke WriteProcessMemory,stProcInfo.hProcess,oaddr,offset dllname,\

    sizeof dllname,addr num ;写入数据

    invoke CreateRemoteThread,stProcInfo.hProcess,NULL,0,dllin,oaddr,0,\

    addr num ;创建远程线程

    invoke    CloseHandle,eax

    invoke ResumeThread,stProcInfo.hThread ;让 QQ 继续运行

    invoke ExitProcess,0

    ret

win9x:

    mov wribase,oepwrite

    mov wrisize,codeend-oepwrite

    invoke VirtualProtect,wribase,wrisize,\

    PAGE_EXECUTE_READWRITE,addr oldpro ;改变这个程序的要写入代码的地方的属性,其实可以在编译时加上开关

    mov wrisize,codeend-codewrite

    mov wriaddr,080000000h

    sub wriaddr,1000h

again:

    add wriaddr,1000h

    mov wribase,codewrite

    invoke WriteProcessMemory,stProcInfo.hProcess,wriaddr,\

    wribase,wrisize,addr num ;写入代码

    mov eax,wrisize

    cmp eax,num

    jnz again

    mov eax,wriaddr

    mov ebx,chan0-4

    mov [ebx],eax

    invoke VirtualProtectEx,stProcInfo.hProcess,0464b58h,7,\

    PAGE_EXECUTE_READWRITE,addr oldpro ;改变 QQ.exe 的 oep 处的属性

    mov wribase,oepwrite

    invoke WriteProcessMemory,stProcInfo.hProcess,0464b58h, wribase,7,\

    addr num ;写入数据

    mov eax,dllin

    mov ebx,chan2-4

    mov [ebx],eax

    mov eax,codeend-codewrite

    add eax,wriaddr

    mov ebx,chan1-4

    mov [ebx],eax

    mov wribase,codewrite

    mov wrisize,codeend-codewrite

    invoke WriteProcessMemory,stProcInfo.hProcess,wriaddr,\

    wribase,wrisize,addr num

    mov eax,codeend-codewrite

    add eax,wriaddr

    mov lastwri,eax

    invoke WriteProcessMemory,stProcInfo.hProcess,lastwri,offset dllname,\

    sizeof dllname,addr num

    invoke ResumeThread,stProcInfo.hThread ;让 QQ 继续执行

    exit:

    invoke ExitProcess,0

    ret

main endp

;下面的都是要写入到 QQ.exe 的内存的附加代码,但有些数值在写入前要实时修正

oepwrite:

    mov eax,0

chan0:    jmp eax

codewrite: 

mov eax,0464b58h

push  6aec8b55h

pop [eax]

mov eax,0464b5ch

push  101868ffh

pop [eax]

push 080000000

chan1: 

push 464b58h

mov eax,0

chan2: 

jmp eax

codeend:

end start
```

`4dd466d0e1e0cc123afbe504da2462ca`
