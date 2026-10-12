
```
00346193    53                 PUSH EBX
00346194    E8 5D000000        CALL 003461F6
0012FF78    52                 PUSH EDX                                ; 对压缩的数据先做一变换
0012FF79    BA 2635B204        MOV EDX, 4B23526
0012FF7E    AC                 LODS BYTE PTR DS:[ESI]
0012FF7F    D2C8               ROR AL, CL
0012FF81    30C8               XOR AL, CL
0012FF83    04 65              ADD AL, 65
0012FF85    30E8               XOR AL, CH
0012FF87    00F0               ADD AL, DH
0012FF89    28D0               SUB AL, DL
0012FF8B    00C8               ADD AL, CL
0012FF8D    28E8               SUB AL, CH
0012FF8F    30D0               XOR AL, DL
0012FF91    04 23              ADD AL, 23
0012FF93    30F0               XOR AL, DH
0012FF95    F6D0               NOT AL
0012FF97    D2C8               ROR AL, CL
0012FF99    D3CA               ROR EDX, CL
0012FF9B    8846 FF            MOV BYTE PTR DS:[ESI-1], AL
0012FF9E    49                 DEC ECX
0012FF9F   ^ 75 DD             JNZ SHORT 0012FF7E
0012FFA1    5A                 POP EDX
0012FFA2    C3                 RETN

003462DD    53                 PUSH EBX                                  ; adalinks.0040FC9A
003462DE    6A 04              PUSH 4
003462E0    68 00100000        PUSH 1000
003462E5    FF342B             PUSH DWORD PTR DS:[EBX+EBP]               ; 8B000
003462E8    6A 00              PUSH 0
003462EA    8D85 96514000      LEA EAX, DWORD PTR SS:[EBP+405196]          ; 返回地址 3466DE (很远啊)
003462F0    50                 PUSH EAX
003462F1    8B85 32F44000      MOV EAX, DWORD PTR SS:[EBP+40F432]          ; KERNEL32.VirtualAlloc
003462F7    E9 C09F0000        JMP 003502BC
003466DE    5B                 POP EBX                                   ; adalinks.0040FC9A
0034682E    8BF0               MOV ESI, EAX                              ; 930000( 分配的空间)
00346830    8BC3               MOV EAX, EBX
00346849    03C5               ADD EAX, EBP                              ; 3511E2
0034684B    8B78 04            MOV EDI, DWORD PTR DS:[EAX+4]             ; 1000
0034684E    03BD 36F44000      ADD EDI, DWORD PTR SS:[EBP+40F436]        ; 40000
00346854    56                 PUSH ESI
00346855    57                 PUSH EDI
00346856    8D85 1E534000      LEA EAX, DWORD PTR SS:[EBP+40531E]        ; 返回地址 346866
0034685C    50                 PUSH EAX
0034685D    8B85 A2F44000      MOV EAX, DWORD PTR SS:[EBP+40F4A2]        ; 490151 解压用户代码到 930000
00346863    FFE0               JMP EAX                                   ; call 的变形
00346866    8B0C2B             MOV ECX, DWORD PTR DS:[EBX+EBP]           ; 8B000
00346869    56                 PUSH ESI
0034686A    51                 PUSH ECX
0034686B    C1E9 02            SHR ECX, 2
0034686E    F3:A5              REP MOVS DWORD PTR ES:[EDI], DWORD PTR DS:[ESI]  ; 930000->401000
00346870    59                 POP ECX
00346871    83E1 03            AND ECX, 3
00346874    F3:A4              REP MOVS BYTE PTR ES:[EDI], BYTE PTR DS:[ESI]    ; 代码解压完成了
00346876    5E                 POP ESI
00346877    53                 PUSH EBX                                  ; 40FC9A
00346878    68 00800000        PUSH 8000
0034687D    6A 00              PUSH 0
0034687F    56                 PUSH ESI                                  ; 930000
00346880    8D85 4B534000      LEA EAX, DWORD PTR SS:[EBP+40534B]        ; 返回地址 346893
00346886    50                 PUSH EAX
00346887    8B85 FAF64000      MOV EAX, DWORD PTR SS:[EBP+40F6FA]        ; KERNEL32.VirtualFree
0034688D    E9 2A9A0000        JMP 003502BC
00346893    5B                 POP EBX                                   ; adalinks.0040FC9A
00346894    83C3 0C            ADD EBX, 0C
00346897   ^ E9 A5F8FFFF      JMP 00346141                              ; 一段代码解压完成了, 循环下一段
```

### 七. 处理 IAT

```
;   003515D7   FF FF FF FF 0C 4D 53 56 42 56 4D 36 30 2E 44 4C   .MSVBVM60.DLL
;   003515E7   4C 00 C5 00 00 80 18 D5 F3 D5 F2 B3 10 93 52 F2
;   003515F7   92 10 15 D1 BF 52 B5 9F 12 F1 F2 51 DE D1 9F 00
;   00351607   0D 10 10 FF 71 51 F3 51 7F B3 9F BF 15 BF 00
```

到这一步，加密后的 IAT 在内存里如下存放：

```
; 1. FF FF FF FF    ----------- 00 00 00 00 表示 所有 DLL 结束
; 2. xx   ----- DLL Name　长度(不算 null)
; 3. DLL 名字, null 结尾 ( 明文 )
; 4. 80 yy yy yy    ---------- yy yy yy 表示　API 个数 ,   80 表示需要重定向
; 5. zz             ---------- ZZ<>0 表示 API Name　长度(不算 null),   ZZ==0, 后 4 byte 函数序号, 1 byte NULL
; 6. API Name, null 结尾 ( 密文, 解密代码见 12FF68 )
; 7. 重复 5,  6 结束 一个 DLL
; 重复 1,..,7 处理 所有 DLL
```

经过壳的 IAT 处理，形成了下面的一个调用过程，（引用 Window 的表示）：

```
;
; iat 中地址 --> `Hook_proc`:
; `Hook_proc`:
;             |PUSH DWORD PTR DS:[`Hook_proc`+1C]
;             |XOR DWORD PTR SS:[`ESP`], `key`
;             |ret; -> |`Stub_proc`:
;                       |`api_start_code`
;                       |`api_some_code`
;                       |push `api_next_code_addr`
;                       |ret
;
; `Hook_proc`+1C:
;   `Stub_proc_xor_key`
```

经过对壳代码 `348A32`、`34A2DC` 的修改，可以变成下面的形式：

```
;
; iat 中地址 --> `Hook_proc`:
; `Hook_proc`:
;             |PUSH DWORD PTR DS:[`Hook_proc`+1C]
;             |XOR DWORD PTR SS:[`ESP`], `key`
;             |ret; ->   `APIaddress`
;
; `Hook_proc`+1C:
;   `APIaddress_xor_key`
```

```
0034689C    8DB5 DD544000    LEA ESI, DWORD PTR SS:[EBP+4054DD]          ; 346A25
003469F1    87E6             XCHG ESI, ESP
003469F3    B9 43780000      MOV ECX, 7843
003469F8    58               POP EAX
003469F9    F6D0             NOT AL
003469FB    50               PUSH EAX
003469FC    44               INC ESP
003469FD   ^ E2 F9           LOOPD SHORT 003469F8                        ; 在 Stack 执行, 解密 346A25 开始的代码
003469FF    87E6             XCHG ESI, ESP                               ; 恢复 Stack
00346A01    6A 04            PUSH 4
00346A03    68 00100000      PUSH 1000
00346A08    68 00200000      PUSH 2000
00346A0D    6A 00            PUSH 0
00346A0F    FF95 32F44000    CALL DWORD PTR SS:[EBP+40F432]              ; KERNEL32.VirtualAlloc, 这一段空间是把 API 的开头部分搬到这里
00346A15    8985 22FD4000    MOV DWORD PTR SS:[EBP+40FD22], EAX          ; [35126A]=930000
00346A1B    C785 26FD4000 0> MOV DWORD PTR SS:[EBP+40FD26], 0
00346A25    8B85 66F44000    MOV EAX, DWORD PTR SS:[EBP+40F466]          ; [3509AE]=1 , IAT 被加密的标志
00346A58    0BC0             OR EAX, EAX
00346A5A    0F85 96090000    JNZ 003473F6
003473F6    8D95 C91A4000    LEA EDX, DWORD PTR SS:[EBP+401AC9]         ; 343011
00347429    0395 86F44000    ADD EDX, DWORD PTR SS:[EBP+40F486]         ; E5C6
0034742F    8B3A             MOV EDI, DWORD PTR DS:[EDX]                ; [3515D7]= FF FF FF FF, IAT 开始的地方
00347580    0BFF             OR EDI, EDI
00347582    75 05            JNZ SHORT 00347589
00347584    E9 6C340000      JMP 0034A9F5                                ; 所有 DLL 处理完了, 到这里
00347589    03BD 36F44000    ADD EDI, DWORD PTR SS:[EBP+40F436]       ; adalinks.00400000
003476DE    83C2 05          ADD EDX, 5                                ; EDX 指向 DLL 的名字
0034770E    8BF2             MOV ESI, EDX                              ; ESI 指向 DLL 的名字
00347710    56               PUSH ESI
00347711    8D85 0A624000    LEA EAX, DWORD PTR SS:[EBP+40620A]       ; 返回地址 347752
00347744    50               PUSH EAX
00347745    8B85 2AF44000    MOV EAX, DWORD PTR SS:[EBP+40F42A]       ; KERNEL32.GetModuleHandleA
0034774B    E9 6C8B0000      JMP 003502BC
00347752    0BC0             OR EAX, EAX
00347754    75 1E            JNZ SHORT 00347774                       ; DLL 已加载
00347756    56               PUSH ESI
00347757    8D85 23624000    LEA EAX, DWORD PTR SS:[EBP+406223]       ; 返回地址 34776B
0034775D    50               PUSH EAX
```
