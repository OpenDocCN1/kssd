
## 汇编代码分析

```
008061D6    74 08               JE      SHORT ezcddax.008061E0            ; 如果是 ffffffff 表示修复结束。

008061D8  ^ E9 A6FCFFFF         JMP     ezcddax.00805E83

008061DD    90                  NOP
008061DE    90                  NOP
008061DF    90                  NOP
008061E0    90                  NOP
008061E1    90                  NOP
008061E2    90                  NOP
```

```
83 05 10 8F 82 00 04 8B 15 10 8F 82 00 8B 12 89 15 00 8F 82 00 83 FA 00 74 E6 83 FA FF 74 08 E9
A6 FC FF FF 90 90 90 90 90 90
```

经过调试，把处理代码修改为下面这样，实行的功能基本完成。

### 运行修改代码前必须做到

1. 把`dump`下的`.text`段二进制复制到，现在调试时的`.text`段中。
2. `Alt+M`在内存窗口中修改`.text`段的`[访问属性]`为`[完全访问]`。
3. 把收集的`CC`地址的二进制值复制到指定的内存中，我是放在`[00828000]`处，调试时发现不是`int3`的`CC`二进制修改为`00000000`，不停调试，不停的修改。
4. 在`CC`地址的最后放入`FFFFFFFF`用以表示`CC`修复结束。
5. 调试时要注意复制修改后的程序二进制数据，防止出现错误，完成后把`.text`中的二进制数据复制到`dump`的文件中保存。

### 修改后的代码

```
00805E39   .  50                 PUSH    EAX                               ; /pContext
00805E3A   .  8B8D 50EEFFFF      MOV     ECX, DWORD PTR SS:[EBP-11B0]       ; |
00805E40   .  51                 PUSH    ECX                               ; |hThread
00805E41   .  FF15 E0808300      CALL    DWORD PTR DS:[<&KERNEL32.GetThreadCo>; \GetThreadContext
00805E47   .  90                 NOP
00805E48   .  90                 NOP
00805E49   .  52                 PUSH    EDX
00805E4A   .  8B15 00808200      MOV     EDX, DWORD PTR DS:[828000]         ;  ezcddax.00439891
00805E50   .  8915 008F8200      MOV     DWORD PTR DS:[828F00], EDX
00805E56   .  C705 108F8200 0080>MOV     DWORD PTR DS:[828F10], ezcddax.00828>
00805E60   .  5A                 POP     EDX
00805E61   .  90                 NOP
00805E62   .  90                 NOP
00805E63   .  90                 NOP
00805E64   .  90                 NOP
00805E65   .  90                 NOP
00805E66   .  90                 NOP
00805E67   .  90                 NOP
00805E68   .  90                 NOP
00805E69   .  90                 NOP
00805E6A   .  90                 NOP
00805E6B   .  90                 NOP
00805E6C   .  90                 NOP
00805E6D   .  90                 NOP
00805E6E   .  90                 NOP
00805E6F   .  90                 NOP
00805E70   .  90                 NOP
00805E71   .  90                 NOP
00805E72   .  90                 NOP
00805E73   .  90                 NOP
00805E74   .  90                 NOP
00805E75   .  90                 NOP
00805E76   .  90                 NOP
00805E77   .  90                 NOP
00805E78   .  90                 NOP
00805E79   .  90                 NOP
00805E7A   .  90                 NOP
00805E7B   .  90                 NOP
00805E7C   .  90                 NOP
00805E7D   .  90                 NOP
00805E7E   .  90                 NOP
00805E7F   .  90                 NOP
00805E80   .  90                 NOP
00805E81   .  90                 NOP
00805E82   .  90                 NOP
00805E83   >  90                 NOP
00805E84   .  90                 NOP
00805E85   .  90                 NOP
00805E86   .  90                 NOP
00805E87   .  90                 NOP
00805E88   .  90                 NOP
00805E89   .  90                 NOP
00805E8A   .  60                 PUSHAD
00805E8B   .  C785 78EBFFFF 0000>MOV     DWORD PTR SS:[EBP-1488], 0
00805E95   .  6A FF              PUSH    -1                                ; /Arg3 = FFFFFFFF
00805E97   .  6A 04              PUSH    4                                 ; |Arg2 = 00000004
00805E99   .  8D95 34ECFFFF      LEA     EDX, DWORD PTR SS:[EBP-13CC]      ; |堆栈地址=0012E3D0
00805E9F   .  52                 PUSH    EDX                               ; |Arg1
00805EA0   .  E8 EB60FDFF        CALL    ezcddax.007DBF90                  ; \ezcddax.007DBF90
00805EA5   .  83C4 0C            ADD     ESP, 0C
00805EA8   .  8985 4CEEFFFF      MOV     DWORD PTR SS:[EBP-11B4], EAX
00805EAE   .  8B85 4CEEFFFF      MOV     EAX, DWORD PTR SS:[EBP-11B4]
00805EB4   .  33D2               XOR     EDX, EDX
00805EB6   .  B9 19000000        MOV     ECX, 19
00805EBB   .  F7F1               DIV     ECX
00805EBD   .  8995 48EEFFFF      MOV     DWORD PTR SS:[EBP-11B8], EDX
00805EC3   .  8B95 34ECFFFF      MOV     EDX, DWORD PTR SS:[EBP-13CC]
00805EC9   .  52                 PUSH    EDX
00805ECA   .  8B85 48EEFFFF      MOV     EAX, DWORD PTR SS:[EBP-11B8]
00805ED0   .  FF1485 98CD8300    CALL    DWORD PTR DS:[EAX*4+83CD98]
00805ED7   .  83C4 04            ADD     ESP, 4
00805EDA   .  8985 78EBFFFF      MOV     DWORD PTR SS:[EBP-1488], EAX
00805EE0   .  C785 74EBFFFF 0000>MOV     DWORD PTR SS:[EBP-148C], 0
00805EEA   .  8B8D 48EEFFFF      MOV     ECX, DWORD PTR SS:[EBP-11B8]
00805EF0   .  8B148D 00F38300    MOV     EDX, DWORD PTR DS:[ECX*4+83F300]
00805EF7   .  8995 54EEFFFF      MOV     DWORD PTR SS:[EBP-11AC], EDX
00805EFD   >  8B85 74EBFFFF      MOV     EAX, DWORD PTR SS:[EBP-148C]
00805F03   .  3B85 54EEFFFF      CMP     EAX, DWORD PTR SS:[EBP-11AC]
00805F09   .  7D 5C              JGE     SHORT ezcddax.00805F67
00805F0B   .  8B85 54EEFFFF      MOV     EAX, DWORD PTR SS:[EBP-11AC]
00805F11   .  2B85 74EBFFFF      SUB     EAX, DWORD PTR SS:[EBP-148C]
00805F17   .  99                 CDQ
00805F18   .  2BC2               SUB     EAX, EDX
00805F1A   .  D1F8               SAR     EAX, 1
00805F1C   .  8B8D 74EBFFFF      MOV     ECX, DWORD PTR SS:[EBP-148C]
00805F22   .  03C8               ADD     ECX, EAX
00805F24   .  898D 70EBFFFF      MOV     DWORD PTR SS:[EBP-1490], ECX
00805F2A   .  8B95 48EEFFFF      MOV     EDX, DWORD PTR SS:[EBP-11B8]
00805F30   .  8B0495 7CF28300    MOV     EAX, DWORD PTR DS:[EDX*4+83F27C]
00805F37   .  8B8D 70EBFFFF      MOV     ECX, DWORD PTR SS:[EBP-1490]
00805F3D   .  8B95 78EBFFFF      MOV     EDX, DWORD PTR SS:[EBP-1488]
00805F43   .  3B1488             CMP     EDX, DWORD PTR DS:[EAX+ECX*4]
00805F46   .  76 11              JBE     SHORT ezcddax.00805F59
00805F48   .  8B85 70EBFFFF      MOV     EAX, DWORD PTR SS:[EBP-1490]
00805F4E   .  83C0 01            ADD     EAX, 1
00805F51   .  8985 74EBFFFF      MOV     DWORD PTR SS:[EBP-148C], EAX
00805F57   .  EB 0C              JMP     SHORT ezcddax.00805F65
00805F59   >  8B8D 70EBFFFF      MOV     ECX, DWORD PTR SS:[EBP-1490]
00805F5F   .  898D 54EEFFFF      MOV     DWORD PTR SS:[EBP-11AC], ECX
00805F65   >^ EB 96              JMP     SHORT ezcddax.00805EFD
00805F67   >  90                 NOP
00805F68   .  90                 NOP
00805F69   .  90                 NOP
00805F6A   .  90                 NOP
00805F6B   .  90                 NOP
00805F6C   .  90                 NOP
00805F6D   .  90                 NOP
00805F6E   .  90                 NOP
00805F6F   .  90                 NOP
00805F70   .  90                 NOP
00805F71   .  90                 NOP
00805F72   .  90                 NOP
00805F73   .  90                 NOP
00805F74   .  90                 NOP
00805F75   .  90                 NOP
00805F76   .  90                 NOP
00805F77   .  90                 NOP
00805F78   .  90                 NOP
00805F79   .  90                 NOP
00805F7A   .  90                 NOP
00805F7B   .  90                 NOP
00805F7C   .  90                 NOP
00805F7D   .  90                 NOP
00805F7E   .  90                 NOP
00805F7F   .  90                 NOP
00805F80   .  90                 NOP
00805F81   .  90                 NOP
00805F82   .  90                 NOP
00805F83   .  90                 NOP
00805F84   .  90                 NOP
00805F85   .  90                 NOP
00805F86   .  90                 NOP
```
