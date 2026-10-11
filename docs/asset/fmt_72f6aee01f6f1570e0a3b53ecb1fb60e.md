
```
0048EA16  PUSH unpacked.0048EAA4
0048EA1B  LEA EAX,DWORD PTR SS:[EBP-1C]
0048EA1E  PUSH EAX
0048EA1F  MOV EAX,DWORD PTR DS:[EBX]
0048EA21  MOV ECX,5
0048EA26  MOV EDX,6
0048EA2B  CALL unpacked.00404BF0                  ; 第 6 位开始，取 5 位
0048EA30  PUSH DWORD PTR SS:[EBP-1C]
0048EA33  PUSH unpacked.0048EAA4
0048EA38  LEA EAX,DWORD PTR SS:[EBP-20]
0048EA3B  PUSH EAX
0048EA3C  MOV EAX,DWORD PTR DS:[EBX]
0048EA3E  MOV ECX,5
0048EA43  MOV EDX,0B
0048EA48  CALL unpacked.00404BF0                  ; 从第 11 位开始，取 5 位
0048EA4D  PUSH DWORD PTR SS:[EBP-20]
0048EA50  MOV EAX,EBX
0048EA52  MOV EDX,5
0048EA57  CALL unpacked.00404A58                  ;
0048EA5C  XOR EAX,EAX                             ; unpacked.0048EA5C
0048EA5E  POP EDX
0048EA5F  POP ECX
0048EA60  POP ECX
0048EA61  MOV DWORD PTR FS:[EAX],EDX
0048EA64  PUSH unpacked.0048EA7E
0048EA69  LEA EAX,DWORD PTR SS:[EBP-20]
0048EA6C  MOV EDX,8
0048EA71  CALL unpacked.00404704
0048EA76  RETN
0048EA77  JMP unpacked.00404064
```

再次进入 `0048E9F9 CALL unpacked.0048DA74` 处的 `call`

```
0048DA9E  PUSH unpacked.0048DB90
0048DAA3  PUSH DWORD PTR FS:[EAX]
0048DAA6  MOV DWORD PTR FS:[EAX],ESP
0048DAA9  MOV EAX,DWORD PTR SS:[EBP-8]
0048DAAC  CALL unpacked.00404998
0048DAB1  MOV DWORD PTR SS:[EBP-C],EAX
0048DAB4  CMP DWORD PTR SS:[EBP-C],0
0048DAB8  JNZ SHORT unpacked.0048DAC7
0048DABA  LEA EAX,DWORD PTR SS:[EBP-8]
0048DABD  MOV EDX,unpacked.0048DBA8                ; ASCII "Think Space"
0048DAC2  CALL unpacked.00404778                  ; 长度
0048DAC7  XOR ESI,ESI
0048DAC9  MOV EBX,100
0048DACE  LEA EAX,DWORD PTR SS:[EBP-10]
0048DAD1  PUSH EAX                                ; /Arg1
0048DAD2  MOV DWORD PTR SS:[EBP-1C],100           ; |
0048DAD9  MOV BYTE PTR SS:[EBP-18],0              ; |
0048DADD  LEA EDX,DWORD PTR SS:[EBP-1C]           ; |
0048DAE0  XOR ECX,ECX                             ; |
0048DAE2  MOV EAX,unpacked.0048DBBC               ; |ASCII "%1.2x"
0048DAE7  CALL unpacked.00409E4C                  ; \unpacked.00409E4C
0048DAEC  MOV EAX,DWORD PTR SS:[EBP-4]            ; 上面产生 100(H)
0048DAEF  CALL unpacked.00404998
0048DAF4  MOV EDI,EAX
0048DAF6  TEST EDI,EDI
0048DAF8  JLE SHORT unpacked.0048DB5A
0048DAFA  MOV DWORD PTR SS:[EBP-14],1
0048DB01  /MOV EAX,DWORD PTR SS:[EBP-4]           ; 串 1 入 eax
0048DB04  |MOV EDX,DWORD PTR SS:[EBP-14]          ; 计数器
0048DB07  |MOVZX EAX,BYTE PTR DS:[EAX+EDX-1]     ; 串 1 各位依次入 eax
0048DB0C  |ADD EAX,EBX                            ; eax=eax+ebx
0048DB0E  |MOV ECX,0FF
0048DB13  |CDQ
0048DB14  |IDIV ECX                               ; 即为取 FF 的模
0048DB16  |MOV EBX,EDX                            ; 余数入 ebx
0048DB18  |CMP ESI,DWORD PTR SS:[EBP-C]
0048DB1B  |JGE SHORT unpacked.0048DB20
0048DB1D  |INC ESI
0048DB1E  |JMP SHORT unpacked.0048DB25
0048DB20  |MOV ESI,1
0048DB25  |MOV EAX,DWORD PTR SS:[EBP-8]           ; 串 2 入 eax
0048DB28  |MOVZX EAX,BYTE PTR DS:[EAX+ESI-1]      ; 串 2 位次入 eax
0048DB2D  |XOR EBX,EAX                            ; ebx=ebx xor eax
0048DB2F  |LEA EAX,DWORD PTR SS:[EBP-20]
0048DB32  |PUSH EAX                               ; /Arg1
0048DB33  |MOV DWORD PTR SS:[EBP-1C],EBX          ; |
0048DB36  |MOV BYTE PTR SS:[EBP-18],0             ; |
0048DB3A  |LEA EDX,DWORD PTR SS:[EBP-1C]          ; |
0048DB3D  |XOR ECX,ECX                            ; |
0048DB3F  |MOV EAX,unpacked.0048DBBC              ; |ASCII "%1.2x"
0048DB44  |CALL unpacked.00409E4C                ; \unpacked.00409E4C
0048DB49  |MOV EDX,DWORD PTR SS:[EBP-20]          ; 把 ebx 转换为十六进制
0048DB4C  |LEA EAX,DWORD PTR SS:[EBP-10]
0048DB4F  |CALL unpacked.004049A0                ; 各位接起来
0048DB54  |INC DWORD PTR SS:[EBP-14]
0048DB57  |DEC EDI
0048DB58  \JNZ SHORT unpacked.0048DB01
0048DB5A  MOV EAX,DWORD PTR SS:[EBP+8]
0048DB5D  MOV EDX,DWORD PTR SS:[EBP-10]
0048DB60  CALL unpacked.00404734
0048DB65  XOR EAX,EAX
0048DB67  POP EDX
0048DB68  POP ECX
0048DB69  POP ECX
0048DB6A  MOV DWORD PTR FS:[EAX],EDX
```

### 总结

- 用户名加在串“`T7e6xtS9p2e`”前面，形成串 `S`：`lordorT7e6xtS9p2e`
- 串 `S` 按其长度除 2 分为两子串：
  - 串 `S1`：`lordorT7`
  - 串 `S2`：`e6xtS9p2e`
- 然后串 `S2` 放在前面，串 `S1` 放在后面，形成 `e6xtS9p2elordorT7`
- 对其串再取两子串：
  - 第 1 位开始取 10 位，即为串 1：`e6xtS9p2el`
  - 第 6 位开始，取余下位，即为串 2：`9p2elordorT7`
- 先设 `ebx=100(H)`，取串 1 的第 1 位 `HEX` 值，取 `(100+第 1 位 HEX)` 的 `FF` 模，得 `A` 值，设为 `ebx=A`
- 取对应位的串 2 的 `HEX` 值 `B`，`ebx=ebx xor B`
- 此 `ebx(H)` 转换为字符串，即为两位注册码，如此继续
- 最后从生成的串中取前面 15 位，形成 `XXXXX-XXXXX-XXXXX`，此即为注册码

比较简单的程序，有空再补上注册机（`keymake` 的就不做了）。下一篇，将介绍 `ollyDbg` 高级断点技术。

一个可用的注册码：

- 用户名：`lordor`
- 注册码：`1005F-E56C8-5B482`

`cracked by lordor`

`03.8.22`

`4dd466d0e1e0cc123afbe504da2462ca`
