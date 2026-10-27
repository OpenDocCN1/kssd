
```
00404C31  /EB 10                 JMP     SHORT 00404C43
00404C33  |90                    NOP
00404C34  |90                    NOP
00404C35  |90                    NOP
00404C36  |90                    NOP
00404C37  |90                    NOP
00404C38  |90                    NOP
00404C39  |90                    NOP
00404C3A  |90                    NOP
00404C3B  |90                    NOP
00404C3C  |90                    NOP
00404C3D  |90                    NOP
00404C3E  |90                    NOP
00404C3F  |90                    NOP
00404C40  |90                    NOP
00404C41  |90                    NOP
00404C42  |90                    NOP
00404C43  \8D45 DC               LEA     EAX, [EBP-24]
00404C46   50                    PUSH    EAX
00404C47   E8 94F8FFFF           CALL    004044E0               ; Get CurrentFolder to [EBP-24], "C:\Temp"
00404C4C   83C4 04               ADD     ESP, 4
00404C4F   68 E8444100           PUSH    004144E8               ; ASCII "\regkey.dat"
00404C54   8D4D D8               LEA     ECX, [EBP-28]
00404C57   50                    PUSH    EAX
00404C58   51                    PUSH    ECX
00404C59   C745 FC 00000000      MOV     DWORD PTR [EBP-4], 0
00404C60   E8 5F8F0000           CALL    0040DBC4               ; mfc42.#924, string cat, "C:\Temp\regkey.dat"
00404C65   8B00                  MOV     EAX, [EAX]
00404C67   50                    PUSH    EAX
00404C68   FF15 34004100         CALL    NEAR [410034]          ; [00410034]=02B30000, kernel32.GetFileAttributesW, ret EAX=00000020
00404C6E   83CF FF               OR      EDI, FFFFFFFF
00404C71   8D4D D8               LEA     ECX, [EBP-28]
00404C74   39F8                  CMP     EAX, EDI
00404C76   0F94C3                SETE    BL
00404C79   E8 028E0000           CALL    0040DA80
00404C7E   8D4D DC               LEA     ECX, [EBP-24]
00404C81   897D FC               MOV     [EBP-4], EDI
00404C84   E8 F78D0000           CALL    0040DA80
00404C89   84DB                  TEST    BL, BL
00404C8B  /74 1C                 JE      SHORT 00404CA9
00404C8D  |8B55 08               MOV     EDX, [EBP+8]           ; 0012FEAC, [0012FEAC]=001605F0
00404C90  |5F                    POP     EDI
00404C91  |5E                    POP     ESI
00404C92  |31C0                  XOR     EAX, EAX
00404C94  |C702 91010000         MOV     DWORD PTR [EDX], 191
00404C9A  |5B                    POP     EBX
00404C9B  |8B4D F4               MOV     ECX, [EBP-C]
00404C9E  |64:890D 00000000      MOV     FS:[0], ECX
00404CA5  |89EC                  MOV     ESP, EBP
00404CA7  |5D                    POP     EBP
00404CA8  |C3                    RETN
00404CA9  \E8 70930000           CALL    0040E01E               ; VM1_137B4E2A, ret eax=00000001
```

调用`0040E01E`实际上就是虚拟化后的SDK函数：`bool WLRegCheckMachineLocked(void);`

> The `WLRegCheckMachineLocked` function determines if the current license is locked to a specific machine.

> If the current license key is locked to a specific machine, the return value is True.

> If the application is not licensed or the current license is not locked to a specific machine, the return value is False.

函数`WLRegCheckMachineLocked`用来检查当前许可证是否锁定到特定机器。是返回true，否则返回false。

```
00404CAE   84C0                  TEST    AL, AL                 ; Is locked to a specific machine?
00404CB0  /75 1C                 JNZ     SHORT 00404CCE         ; Yes
00404CB2  |8B45 08               MOV     EAX, [EBP+8]
00404CB5  |5F                    POP     EDI
00404CB6  |5E                    POP     ESI
00404CB7  |5B                    POP     EBX
00404CB8  |C700 92010000         MOV     DWORD PTR [EAX], 192
00404CBE  |31C0                  XOR     EAX, EAX
00404CC0  |8B4D F4               MOV     ECX, [EBP-C]
00404CC3  |64:890D 00000000      MOV     FS:[0], ECX
00404CCA  |89EC                  MOV     ESP, EBP
00404CCC  |5D                    POP     EBP
00404CCD  |C3                    RETN
00404CCE  \8D8D CCF8FFFF         LEA     ECX, [EBP-734]         ; 0012F5B0, pCustomData
00404CD4   8D95 CCFCFFFF         LEA     EDX, [EBP-334]         ; 0012F9B0, pCompanyName
00404CDA   51                    PUSH    ECX
00404CDB   8D85 CCFDFFFF         LEA     EAX, [EBP-234]         ; 0012FAB0, pName
00404CE1   52                    PUSH    EDX
00404CE2   50                    PUSH    EAX
00404CE3   E8 18930000           CALL    0040E000               ; VM1_137A376D, ret eax=00000001
```

调用`0040E000`为虚拟化后的SDK函数：`bool WLRegGetLicenseInfo(char* pName, char* pCompanyName, char* pCustomData);`

> The `WLRegGetLicenseInfo` function retrieves the registration information for the current license.

> If the function succeeds, the return value is True.

> If the function fails, the return value is False.

函数`WLRegGetLicenseInfo`用来获取当前许可证的注册信息：用户名称、组织名称及用户自定义数据。成功返回true，失败返回false。

```
00404CE8   84C0                  TEST    AL, AL                 ; Is succeeded?
00404CEA  /75 1C                 JNZ     SHORT 00404D08         ; Yes
00404CEC  |8B4D 08               MOV     ECX, [EBP+8]
00404CEF  |5F                    POP     EDI
00404CF0  |5E                    POP     ESI
00404CF1  |31C0                  XOR     EAX, EAX
00404CF3  |C701 93010000         MOV     DWORD PTR [ECX], 193
00404CF9  |5B                    POP     EBX
00404CFA  |8B4D F4               MOV     ECX, [EBP-C]
00404CFD  |64:890D 00000000      MOV     FS:[0], ECX
00404D04  |89EC                  MOV     ESP, EBP
00404D06  |5D                    POP     EBP
00404D07  |C3                    RETN
00404D08  \8D95 CCFDFFFF         LEA     EDX, [EBP-234]         ; pName
00404D0E   8D4D E8               LEA     ECX, [EBP-18]          ; 0012FCCC
00404D11   52                    PUSH    EDX
00404D12   E8 898E0000           CALL    0040DBA0               ; mfc42.#537, copy to [EBP-18]
00404D17   8D85 CCF8FFFF         LEA     EAX, [EBP-734]         ; pCustomData
00404D1D   8D4D F0               LEA     ECX, [EBP-10]
00404D20   50                    PUSH    EAX
00404D21   C745 FC 01000000      MOV     DWORD PTR [EBP-4], 1
00404D28   E8 738E0000           CALL    0040DBA0               ; mfc42.#537, copy to [EBP-10]
00404D2D   8B4D E8               MOV     ECX, [EBP-18]
00404D30   C645 FC 02            MOV     BYTE PTR [EBP-4], 2
00404D34   8B41 F8               MOV     EAX, [ECX-8]           ; length of Name
00404D37   83F8 02               CMP     EAX, 2
00404D3A   0F8C D7020000         JL      00405017
00404D40   83F8 0A               CMP     EAX, 0A                ; 0x02 <= length of Name <= 0x0A
00404D43   0F8F CE020000         JG      00405017
```

用户名字符串长度限制：大于等于2，小于等于10。

```
00404D49   8B55 F0               MOV     EDX, [EBP-10]
00404D4C   8B42 F8               MOV     EAX, [EDX-8]           ; length of CustomData
00404D4F   83F8 31               CMP     EAX, 31
00404D52   0F8F B4020000         JG      0040500C
00404D58   83F8 27               CMP     EAX, 27
00404D5B   0F8C AB020000         JL      0040500C               ; 0x27 <= length of CustomData <= 0x31
```

自定义字符串长度限制：大于等于39，小于等于49。

```
00404D61   68 E4444100           PUSH    004144E4               ; pointer to delimiter char: 0x7C('|')
00404D66   8D4D F0               LEA     ECX, [EBP-10]
00404D69   E8 4C8F0000           CALL    0040DCBA               ; mfc42.#2764, search '|' in CustomData string
```

查找自定义字符串中的分隔符`'|'`。
