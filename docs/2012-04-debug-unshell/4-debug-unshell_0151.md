
# ACProtect1.41 保护技术分析

> 原文：[`www.pediy.com/kssd/pediy12/139995.html`](https://www.pediy.com/kssd/pediy12/139995.html)

这个壳很老了，但是还是来分析下它所使用的保护技术。这个壳是使用加密解密三的 SMC 那章的保护技术，依次循环解码，分几个段，由当前段解码加密后的字节码，放到下一个执行段。当前段解码执行完成后，执行下一段解出来的代码。每段解码代码中包含了花指令、代码变形、迷惑等等。当解压到最后一段代码的时候，这个壳就直接一个 `JMP` 跳到 OEP 了。我们脱这个壳分为两个步骤：

1. 清除花指令
2. 查找迷惑变形代码

我们首先清除花指令，花指令的清除比较简单，比如下面这段代码：

```
00405000 >   60               PUSHAD
00405001     EB 01            JMP SHORT 00405004                     ; JMP SHORT 00405004
00405003     E8 0F810100      CALL 0041D117
00405008     0000             ADD BYTE PTR DS:[EAX],AL
```

那个 `JMP` 那里跳到 `405004`，这里没有 `405004`，有 `405003`，所以 `405003` 就是一个花指令了。我们直接二进制编辑 `NOP` 掉，这样依次循环下去，我们就能清除花指令了。实际上那个 `JMP` 指令也是无用的。我们第一段解码代码清除后总结下花指令的组合方式。

花指令清除后，变形代码的清除。

下面这段代码模拟 `JMP` 指令代码：

```
00405016   /7E 03            JLE SHORT 0040501B
00405018   |7F 01            JG SHORT 0040501B
```

下面这段代码清除花指令后可以直接 `NOP` 掉，因为它什么作用都没有，用来迷惑我们的代码：

```
0040502A     50               PUSH EAX
0040502B     E8 01000000      CALL 00405031                          ; 花指令，下指令的开始字节 NOP 掉
00405030   - 76 83            JBE SHORT 00404FB5
花指令后的代码
0040502A     50               PUSH EAX                               ; 保存 EAX 的值
0040502B     E8 01000000      CALL 00405031                          ; Call 会压入下个地址到堆栈，esp-4
00405030     90               NOP
00405031     83C4 04          ADD ESP,4                              ; 恢复堆栈，ESP+4
00405034     58               POP EAX                                ; 恢复 EAX 的值
```

下面这段代码也是什么作用都没有，也是用来迷惑我们的：

```
00405036     E8 01000000      CALL 0040503C
0040503B   - 7C 83            JL SHORT 00404FC0                      ; 花指令
0040503D     04 24            ADD AL,24
清除后的代码
00405036     E8 01000000      CALL 0040503C                          ; Call 会压入下个地址到堆栈，esp-4
0040503B     90               NOP                                    ; 花指令
0040503C     830424 06        ADD DWORD PTR SS:[ESP],6               ; 下条指令的地址加上 6，CALL 的下一条指令到 RETN 刚好 6 字节，所以目标地址就是 RETN 后一条指令
00405040     C3               RETN
```

这种情况下的 `JMP` 指令也是可以 `NOP` 掉的：

```
00405043   /EB 01             JMP SHORT 00405046                     ; 直接跳到 XXXX 地址，其下的代码到目标地址之间的代码是无效的
00405045   |90                NOP                                    ; 花指令
```

无用代码：

```
0040508B     E8 01000000      CALL 00405091
00405090     90               NOP
00405091     83C4 04          ADD ESP,4
```

不可以 `NOP` 的代码：

```
004050BD   /0F83 05000000     JNB 004050C8                           ; 不可 NOP
004050C3   |66:81E2 73A2      AND DX,0A273
004050C8   \\EB 01            JMP SHORT 004050CB                     ; 可以 NOP
```

另一种情况代码：

```
00405101   /7C 03             JL SHORT 00405106
00405103   |7D 01             JGE SHORT 00405106                     ; 模拟 JMP，跳到下面的 jmp
00405105   |90                NOP
00405106   \\E9 06000000      JMP 00405111                           ; 跳到另一个地址，其模拟 JMP 指令开始的地方到目标地址的上个指令都是无效
0040510B     46               INC ESI                                ; 无效
0040510C     B8 6D9231A7      MOV EAX,A731926D                       ; 无效
```

这是我第一段的解码指令清除后的代码：



```
00405000 >  60                PUSHAD
00405001   90                 NOP                             ; JMP SHORT 00405004
00405002   90                 NOP
00405003   90                 NOP
00405004   0F81 01000000      JNO 0040500B
0040500A   41                 INC ECX
0040500B   4A                 DEC EDX
0040500C   F9                 STC
0040500D   F8                 CLC
0040500E   FC                 CLD
0040500F   66:8BF7            MOV SI,DI
00405012   46                 INC ESI
00405013   66:8BF0            MOV SI,AX
00405016   7E 03              JLE SHORT 0040501B
00405018   7F 01              JG SHORT 0040501B
0040501A   90                 NOP
0040501B   4A                 DEC EDX
0040501C   90                 NOP
0040501D   90                 NOP
0040501E   90                 NOP
0040501F   0F8A 05000000      JPE 0040502A
00405025   BE 004176CF        MOV ESI,CF764100
0040502A   90                 NOP                             ; 保存 EAX 的值
0040502B   90                 NOP                             ; Call 会压入下个地址到堆栈，esp-4
0040502C   90                 NOP
0040502D   90                 NOP
0040502E   90                 NOP
0040502F   90                 NOP
00405030   90                 NOP
00405031   90                 NOP                             ; 恢复堆栈，ESP+4
00405032   90                 NOP
00405033   90                 NOP
00405034   90                 NOP                             ; 恢复 EAX 的值
00405035   46                 INC ESI
00405036   90                 NOP                             ; Call 会压入下个地址到堆栈，esp-4
00405037   90                 NOP
00405038   90                 NOP
00405039   90                 NOP
0040503A   90                 NOP
0040503B   90                 NOP                             ; 花指令
0040503C   90                 NOP                             ; 下条指令的地址加上 6，CALL 的下一条指令到 RETN 刚好 6 字节，所以目标地址就是 RETN 后一条指令
0040503D   90                 NOP
0040503E   90                 NOP
0040503F   90                 NOP
00405040   90                 NOP
00405041   8BF0               MOV ESI,EAX
00405043   90                 NOP                             ; 直接跳到 XXXX 地址，其下的代码到目标地址之间的代码是无效的
00405044   90                 NOP
00405045   90                 NOP                             ; 花指令
00405046   87D6               XCHG ESI,EDX
00405048   90                 NOP
00405049   90                 NOP
0040504A   90                 NOP
0040504B   90                 NOP
0040504C   90                 NOP
0040504D   90                 NOP
0040504E   90                 NOP
0040504F   90                 NOP
00405050   90                 NOP
00405051   90                 NOP
00405052   90                 NOP
00405053   33D7               XOR EDX,EDI
00405055   90                 NOP
00405056   90                 NOP
00405057   90                 NOP
00405058   90                 NOP
00405059   90                 NOP
0040505A   90                 NOP
0040505B   90                 NOP
0040505C   90                 NOP
0040505D   90                 NOP
0040505E   90                 NOP
0040505F   90                 NOP
00405060   FC                 CLD
00405061   90                 NOP
00405062   90                 NOP
00405063   90                 NOP
00405064   85EA               TEST EDX,EBP
00405066   90                 NOP                             ; JMP，直接跳到目标地址，其现在的代码到目标之间的代码直接 NOP
00405067   90                 NOP
00405068   90                 NOP
00405069   90                 NOP
0040506A   90                 NOP                             ; 花指令
0040506B   77 02              JA SHORT 0040506F
0040506D   1BEB               SBB EBP,EBX
0040506F   90                 NOP
00405070   90                 NOP
00405071   90                 NOP
00405072   90                 NOP
00405073   90                 NOP
00405074   46                 INC ESI
00405075   90                 NOP
00405076   90                 NOP
00405077   90                 NOP
00405078   D3D6               RCL ESI,CL
0040507A   90                 NOP
0040507B   90                 NOP
0040507C   90                 NOP
0040507D   90                 NOP
0040507E   90                 NOP
0040507F   90                 NOP
00405080   90                 NOP
00405081   90                 NOP
00405082   90                 NOP
00405083   90                 NOP
00405084   90                 NOP
00405085   81F5 E0CDFA12      XOR EBP,12FACDE0
0040508B   90                 NOP
0040508C   90                 NOP
0040508D   90                 NOP
0040508E   90                 NOP
0040508F   90                 NOP
00405090   90                 NOP
00405091   90                 NOP
00405092   90                 NOP
00405093   90                 NOP
00405094   46                 INC ESI
00405095   90                 NOP
00405096   90                 NOP
00405097   90                 NOP
00405098   90                 NOP
00405099   90                 NOP
0040509A   90                 NOP
0040509B   90                 NOP
0040509C   90                 NOP
0040509D   90                 NOP
0040509E   90                 NOP
0040509F   90                 NOP
004050A0   87EA               XCHG EDX,EBP
004050A2   90                 NOP
004050A3   90                 NOP
004050A4   90                 NOP
004050A5   90                 NOP
004050A6   90                 NOP
004050A7   F9                 STC
004050A8   90                 NOP
004050A9   90                 NOP
004050AA   90                 NOP
004050AB   46                 INC ESI
004050AC   90                 NOP
004050AD   90                 NOP
004050AE   90                 NOP
004050AF   90                 NOP
004050B0   90                 NOP
004050B1   C1FA 75            SAR EDX,75
004050B4   90                 NOP
004050B5   90                 NOP
004050B6   90                 NOP
004050B7   90                 NOP
004050B8   90                 NOP
004050B9   90                 NOP
004050BA   90                 NOP
004050BB   90                 NOP
004050BC   90                 NOP
004050BD   0F83 05000000      JNB 004050C8                     ; 不可 NOP
004050C3   66:81E2 73A2       AND DX,0A273
004050C8   90                 NOP                             ; 可以 NOP
004050C9   90                 NOP
004050CA   90                 NOP
004050CB   66:8BD7            MOV DX,DI
004050CE   90                 NOP
004050CF   90                 NOP
004050D0   90                 NOP
004050D1   90                 NOP
004050D2   90                 NOP
004050D3   90                 NOP
004050D4   90                 NOP
004050D5   90                 NOP
004050D6   90                 NOP
004050D7   66:8BD3            MOV DX,BX
004050DA   90                 NOP
004050DB   90                 NOP
004050DC   90                 NOP
004050DD   66:D3CE            ROR SI,CL
004050E0   90                 NOP
004050E1   90                 NOP
004050E2   90                 NOP
004050E3   F9                 STC
004050E4   68 C1514000        PUSH 004051C1
004050E9   7A 01              JPE SHORT 004050EC
004050EB   40                 INC EAX
004050EC   5A                 POP EDX
004050ED   78 03              JS SHORT 004050F2
004050EF   79 01              JNS SHORT 004050F2
004050F1   90                 NOP
004050F2   40                 INC EAX
004050F3   BF 4047AA83        MOV EDI,83AA4740
004050F8   66:13F3            ADC SI,BX
004050FB   81EF 4AB300E0      SUB EDI,E000B34A
00405101   90                 NOP
00405102   90                 NOP
00405103   90                 NOP                             ; 模拟 JMP，跳到下面的 jmp
00405104   90                 NOP
00405105   90                 NOP
00405106   90                 NOP                             ; 跳到另一个地址，其模拟 JMP 指令开始的地方到目标地址的上个指令都是无效
00405107   90                 NOP
00405108   90                 NOP
00405109   90                 NOP
0040510A   90                 NOP
0040510B   90                 NOP                             ; 无效
0040510C   90                 NOP                             ; 无效
0040510D   90                 NOP
0040510E   90                 NOP
0040510F   90                 NOP
00405110   90                 NOP
00405111   BD F1933A56        MOV EBP,563A93F1
00405116   8BDF               MOV EBX,EDI
00405118   81C5 576CC5A9      ADD EBP,A9C56C57
0040511E   90                 NOP
0040511F   90                 NOP
00405120   90                 NOP
00405121   90                 NOP
00405122   90                 NOP
00405123   90                 NOP
00405124   90                 NOP
00405125   90                 NOP
00405126   90                 NOP
00405127   90                 NOP
00405128   90                 NOP
00405129   85F3               TEST EBX,ESI
0040512B   87C3               XCHG EBX,EAX
0040512D   8B0A               MOV ECX,DWORD PTR DS:[EDX]
0040512F   90                 NOP
00405130   90                 NOP
00405131   90                 NOP
00405132   90                 NOP
00405133   90                 NOP
00405134   66:8BDA            MOV BX,DX
00405137   66:D3C6            ROL SI,CL
0040513A   03CF               ADD ECX,EDI
0040513C   90                 NOP
0040513D   90                 NOP
0040513E   90                 NOP
0040513F   90                 NOP
00405140   90                 NOP
00405141   90                 NOP
00405142   90                 NOP
00405143   90                 NOP
00405144   90                 NOP
00405145   90                 NOP
00405146   90                 NOP
00405147   43                 INC EBX
00405148   66:D3CE            ROR SI,CL
0040514B   C1C1 08            ROL ECX,8
0040514E   90                 NOP
0040514F   90                 NOP
00405150   90                 NOP
00405151   90                 NOP
00405152   90                 NOP
00405153   90                 NOP
00405154   90                 NOP
00405155   90                 NOP
00405156   90                 NOP
00405157   90                 NOP
00405158   90                 NOP
00405159   90                 NOP
0040515A   90                 NOP
0040515B   90                 NOP
0040515C   90                 NOP
0040515D   90                 NOP
0040515E   90                 NOP
0040515F   90                 NOP
00405160   90                 NOP
00405161   90                 NOP
00405162   90                 NOP
00405163   90                 NOP
00405164   90                 NOP
00405165   83EA FC            SUB EDX,-4
00405168   030A               ADD ECX,DWORD PTR DS:[EDX]
0040516A   83EA 04            SUB EDX,4
0040516D   90                 NOP
0040516E   90                 NOP
0040516F   90                 NOP
00405170   90                 NOP
00405171   90                 NOP
00405172   90                 NOP
00405173   90                 NOP
00405174   90                 NOP
00405175   90                 NOP
00405176   46                 INC ESI
00405177   890A               MOV DWORD PTR DS:[EDX],ECX
00405179   90                 NOP
0040517A   90                 NOP
0040517B   90                 NOP
0040517C   0F8A 04000000      JPE 00405186
00405182   66:B8 8BA0         MOV AX,0A08B
00405186   81EF 737F09E2      SUB EDI,E2097F73
0040518C   90                 NOP
0040518D   90                 NOP
0040518E   90                 NOP
0040518F   90                 NOP
00405190   90                 NOP
00405191   66:BE 3699         MOV SI,9936
00405195   83C2 04            ADD EDX,4
00405198   90                 NOP
00405199   90                 NOP
0040519A   90                 NOP
0040519B   90                 NOP
0040519C   90                 NOP
0040519D   90                 NOP
0040519E   90                 NOP
0040519F   90                 NOP
004051A0   90                 NOP
004051A1   90                 NOP
004051A2   83C5 FF            ADD EBP,-1
004051A5   ^ 0F85 82FFFFFF    JNZ 0040512D                     ; 向上跳转
```



下面总结花指令的方式，当有一个 `JMP` 指令跳到一个 `XXXX` 地址的时候，这个地址不管是不是花指令，其 `JMP` 指令到目标地址的上一条指令都是无效的，有模拟 `JMP` 指令的时候也是如此，比如 `Jg Jle` 指令这样的代码到目标地址的上一条指令之间的代码都是无效的，我们这样依次清除下去就能看到全部外壳段的代码了，这段代码比较长，是依次循环解码的，就不依依给出了，下面说下脱壳的方法，当装载进程序后，下一个硬件访问 (`hr`) 断点，其地址指向 `ESP`，这里是 `hr 0012FFA4`，当我们断点下好以后，按 13 次 `F9` 就间接跳到 `OEP` 了。

跳向 `OEP` 的代码:

```
0041C105    61               POPAD
0041C106    EB 01            JMP SHORT 0041C109
0041C108    90               NOP
0041C109   - FF25 4BC14100    JMP DWORD PTR DS:\[41C14B\]             ; GUI1.00401000
```

来到 `Oep` 后我们发现输入表被加密了，看下代码：

代码:

```
00401000    6A 00            PUSH 0
00401002    E8 27000000      CALL 0040102E                             ; GetModuleHandleA
00401007    6A 00            PUSH 0
00401009    68 09304000      PUSH 00403009                             ; ASCII "Test"
0040100E    68 00304000      PUSH 00403000                             ; ASCII "UnpackMe"
00401013    6A 00            PUSH 0
00401015    E8 08000000      CALL 00401022                             ; MessageBoxA
0040101A    6A 00            PUSH 0
0040101C    E8 07000000      CALL 00401028                             ; ExitProcess
00401021    CC               INT3
00401022   - FF25 0C204000    JMP DWORD PTR DS:\[40200C\]             ; JMP 函数地址，这里是外壳添加的代码的地址
00401028   - FF25 04204000    JMP DWORD PTR DS:\[402004\]             ; JMP 函数地址，这里是外壳添加的代码的地址
0040102E   - FF25 00204000    JMP DWORD PTR DS:\[402000\]             ; JMP 函数地址，这里是外壳添加的代码的地址
```

代码:

```
0040501D    68 09C50F67      PUSH 670FC509
00405022    813424 48728F1B  XOR DWORD PTR SS:\[ESP\],1B8F7248
00405029    C3               RETN                                    ; 返回到 7C80B741 (kernel32.GetModuleHandleA)
```

现在修复输入表，我们直接把 `JMP` 那个地址的内容改为函数的地址，示例代码:

```
00401000   .  6A 00           PUSH 0                                  ; /pModule = NULL
00401002   .  E8 27000000     CALL 0040102E                           ; \GetModuleHandleA
00401007   .  6A 00           PUSH 0                                  ; /Style = MB_OK|MB_APPLMODAL
00401009   .  68 09304000     PUSH 00403009                           ; |Title = "Test"
0040100E   .  68 00304000     PUSH 00403000                           ; |Text = "UnpackMe"
00401013   .  6A 00           PUSH 0                                  ; |hOwner = NULL
00401015   .  E8 08000000     CALL 00401022                           ; \MessageBoxA
0040101A   .  6A 00           PUSH 0                                  ; /ExitCode = 0
0040101C   .  E8 07000000     CALL 00401028                           ; \ExitProcess
00401021     CC              INT3
00401022   $- FF25 0C204000  JMP DWORD PTR DS:\[40200C\]             ; DS:\[0040200C\]=77D507EA (user32.MessageBoxA)
00401028   .- FF25 04204000  JMP DWORD PTR DS:\[402004\]             ; DS:\[00402004\]=7C81CB12 (kernel32.ExitProcess)
0040102E   $- FF25 00204000  JMP DWORD PTR DS:\[402000\]             ; DS:\[00402004\]=7C81CB12 (kernel32.ExitProcess)
```

然后使用 `LoadPE` 重建 `PE`，修复成功，运行程序 OK, 分析完毕

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) | | `GUI1.zip` |
|---|---|---|
