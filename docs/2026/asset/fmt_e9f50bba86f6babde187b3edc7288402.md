
从ESI数据中获得1个word的立即数，按照dword的方式压入EBPSTACK

### 0043D978 命名
`VM_PUSHw_IMMEDIATEb`

代码：
```
0043D979   .  0FB646 FF     MOVZX EAX,BYTE PTR DS:[ESI-1]            ; \*
0043D97E   .  30D8          XOR AL,BL                                ; \*
0043D1ED   .  FEC8          DEC AL                                   ; \*
0043D1F0   .  F6D8          NEG AL                                   ; \*
0043D1F7   .  F6D0          NOT AL                                   ; \*
0043D1FD   .  30C3          XOR BL,AL                                ; \*
0043CEE8   > /83ED 02       SUB EBP,2                                ; \*
0043DD79  |.  66:8945 00    MOV WORD PTR SS:[EBP],AX                 ; |\*
0043DD62  /$  4E            DEC ESI                                  ; \*
```

功能：
从ESI数据中获得1个byte的立即数，按照word的方式压入EBPSTACK

### 0043D3A5 命名
`VM_PUSHdw_IMMEDIATEb`

代码：
```
0043D3A7    0FB646 FF       MOVZX EAX,BYTE PTR DS:[ESI-1]            ; \*
0043D3AC    30D8            XOR AL,BL                                ; \*
0043D848    FEC8            DEC AL                                   ; \*
0043D855    F6D8            NEG AL                                   ; \*
0043D866    F6D0            NOT AL                                   ; \*
0043D86D    30C3            XOR BL,AL                                ; \*
0043ED8C    66:98           CBW                                      ; \*
0043CF7B    98              CWDE                                     ; \*
0043EC00    8D76 FF         LEA ESI,[ESI-1]                          ; \*
0043DB94    83ED 04         SUB EBP,4                                ; \*
0043DB9F    8945 00         MOV DWORD PTR SS:[EBP],EAX               ; \*
```

功能：
从ESI数据中获得1个byte的立即数，按照dword的方式压入EBPSTACK

### 0043CF24 命名
`VM_ADDdw_EBPSTACK`

代码：
```
0043CF2F  |.  8B45 00       MOV EAX,DWORD PTR SS:[EBP]               ; \*
0043EED3  |.  0145 04       ADD DWORD PTR SS:[EBP+4],EAX             ; \*
0043CE4F  |.  9C            PUSHFD                                   ; \*
0043CE50  |.  8F4424 3C     POP DWORD PTR SS:[ESP+3C]                ; \*
0043D1BF  /> \FF7424 3C     PUSH DWORD PTR SS:[ESP+3C]               ; \*
0043D1C3  |.  8F45 00       POP DWORD PTR SS:[EBP]                   ; \*
```

功能：
把EBPSTACK栈顶的2个dword数据相加，结果存储在`[EBP+4]`，EFLAGS标志存储在栈顶。

例：
```
        0013F97C   8021D2F0  !
        0013F980   00000000  ....

VM_ADDdw_EBPSTACK

        0013F97C   00000286  ..
        0013F980   8021D2F0  !
```

### 0043D43B 命名
`VM_PUSHdw_MEMORYdw`

代码：
```
0043D43F    8B45 00         MOV EAX,DWORD PTR SS:[EBP]               ; \*
0043D10A    8B00            MOV EAX,DWORD PTR DS:[EAX]               ; \*
0043D447    8945 00         MOV DWORD PTR SS:[EBP],EAX               ; \*
```

功能：
把EBPSTACK栈顶数据作为内存地址，从中读取1个dword的数据压入EBPSTACK

### 0043E9CA 命名
`VM_PUSHw_MEMORYw`

代码：
```
0043E9D0    8B45 00         MOV EAX,DWORD PTR SS:[EBP]               ; \*
0043E9D9    83C5 02         ADD EBP,2                                ; \*
0043DEBB    66:36:8B00      MOV AX,WORD PTR SS:[EAX]                 ; \*
0043DDC4    66:8945 00      MOV WORD PTR SS:[EBP],AX                 ; \*
```

功能：
把EBPSTACK栈顶数据作为内存地址，从中读取1个word的数据压入EBPSTACK

### 0043D8BD 命名
`VM_PUSHw_MEMORYb`

代码：
```
0043D57B  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]
0043D589  |.  83C5 02       ADD EBP,2                                ; \*
0043D591  |.  8A02          MOV AL,BYTE PTR DS:[EDX]                 ; \*
0043E70B  |.  66:8945 00    MOV WORD PTR SS:[EBP],AX                 ; \*
```

功能：
把EBPSTACK栈顶数据作为内存地址，从中读取1个byte的数据，按照word的方式压入EBPSTACK

### 0043DC56 命名
`VM_PUSHw_EDISTACKw`

代码：
```
0043DC5C    8A46 FF         MOV AL,BYTE PTR DS:[ESI-1]               ; \*
0043DC66    28D8            SUB AL,BL                                ; \*
0043DC6D    C0C8 05         ROR AL,5                                 ; \*
0043EADA    4E              DEC ESI                                  ; \*
0043EE2E   \F6D8            NEG AL                                   ; \*
0043EE34    34 0E           XOR AL,0E                                ; \*
0043E740    28C3            SUB BL,AL                                ; \*
0043E746    66:8B0438       MOV AX,WORD PTR DS:[EDI+EAX]             ; \*
0043D9E4    83ED 02         SUB EBP,2                                ; \*
0043EE44    66:8945 00      MOV WORD PTR SS:[EBP],AX                 ; \*
```

功能：
从EDISTACK中读取1个word数据压入EBPSTACK

### 0043CF81 命名
`VM_PUSHw_EDISTACKb`

代码：
```
0043CF84    8A46 FF         MOV AL,BYTE PTR DS:[ESI-1]               ; \*
0043CF8E    30D8            XOR AL,BL                                ; \*
0043EE0A   \FEC0            INC AL                                   ; \*
0043EE11    C0C8 07         ROR AL,7                                 ; \*
0043EE1E    FEC0            INC AL                                   ; \*
0043D59C    30C3            XOR BL,AL                                ; \*
0043D2CE    4E              DEC ESI                                  ; \*
0043D2D7    8A0438          MOV AL,BYTE PTR DS:[EDI+EAX]             ; \*
0043D2E6    83ED 02         SUB EBP,2                                ; \*
0043D075    66:8945 00      MOV WORD PTR SS:[EBP],AX                 ; \*
```

功能：
从EDISTACK中读取1个byte数据,按照word方式压入EBPSTACK

### 0043D72E 命名
`VM_PUSHdw_EBP`

代码：
```
0043D72F  /.  89E8          MOV EAX,EBP                              ; \*
0043E613  /$  83ED 04       SUB EBP,4                                ; \*
0043E61C  |.  8945 00       MOV DWORD PTR SS:[EBP],EAX               ; \*
```

功能：
复制EBP指针到EBPSTACK栈顶

例：
```
        EBP 0013F9AC
        0013F9AC   00000000  ....

VM_PUSHdw_EBP

        0013F9A8   0013F9AC  .
        0013F9AC   00000000  ....
```

### 0043EABE 命名
`VM_COPYdw_EBPSTACK`

代码：
```
0043EACC    8B45 00         MOV EAX,DWORD PTR SS:[EBP]               ; \*
0043DE1B    36:8B00         MOV EAX,DWORD PTR SS:[EAX]               ; \*
0043DE25    8945 00         MOV DWORD PTR SS:[EBP],EAX               ; \*
```

功能：
把EBPSTACK栈顶数据作为堆栈地址，从中读取一个dword的数据压入EBPSTACK

例：
```
        0013F998   F99E
        0013F99C   02460013  .F
        0013F9A0       0000  ...

VM_COPYdw_EBPSTACK

        0013F998   0246
        0013F99C   02460000  ..F
        0013F9A0       0000  ...
```

### 0043E790 命名
`VM_COPYw_EBPSTACK`

代码：
```
0043E79C  |.  8B55 00       MOV EDX,DWORD PTR SS:[EBP]               ; \*
0043E7A7  |.  83C5 02       ADD EBP,2                                ; \*
0043E7AE  |.  36:8A02       MOV AL,BYTE PTR SS:[EDX]                 ; \*
0043D01B  |.  66:8945 00    MOV WORD PTR SS:[EBP],AX                 ; \*
```

功能：
把EBPSTACK栈顶数据作为堆栈地址，从中读取一个byte的数据，按照word的方式压入EBPSTACK

例：
```
        0013F9A8   0013F9AC  .
        0013F9AC       0000  ....

VM_COPYw_EBPSTACK

        0013F9A8   0000
        0013F9AC       0000  ....
```

### 0043D198 命名
`VM_NANDdw`

代码：
```
0043D1A3    8B45 00         MOV EAX,DWORD PTR SS:[EBP]               ; \*
0043D1AD    8B55 04         MOV EDX,DWORD PTR SS:[EBP+4]             ; \*
0043DEAE    F7D0            NOT EAX                                  ; \*
0043DDE1   /F7D2            NOT EDX                                  ; \*
0043CDC2    21D0            AND EAX,EDX                              ; \*
0043E0F8    8945 04         MOV DWORD PTR SS:[EBP+4],EAX             ; \*
0043D0FB   /9C              PUSHFD                                   ; \*
0043D0FC    8F4424 2C       POP DWORD PTR SS:[ESP+2C]                ; \*
0043EC46    FF7424 34       PUSH DWORD PTR SS:[ESP+34]               ; \*
0043EC4A    8F45 00         POP DWORD PTR SS:[EBP]                   ; \*
```

功能：
dword版的与非门，从EBPSTACK的栈顶读取2个dword作为操作数，结果存储在第二个操作数位置，EFLAGS标志存储在栈顶。

例：
```
        0013F9A8   00000286  ..
        0013F9AC   00000286  ..

VM_NANDdw

        0013F9A8   00000282  ..
        0013F9AC   FFFFFD79  y
```

### 0043EB92 命名
`VM_NANDw`

代码：
```
0043EB94  |.  66:8B45 00    MOV AX,WORD PTR SS:[EBP]                 ; \*
0043EBA5  |.  66:8B55 02    MOV DX,WORD PTR SS:[EBP+2]               ; \*
0043EBB3  |.  F6D0          NOT AL                                   ; \*
```
