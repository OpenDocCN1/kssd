
```asm
00404F2F   83F8 01               CMP     EAX, 1
00404F32  /7D 45                 JGE     SHORT 00404F79
00404F34  |8B55 08               MOV     EDX, [EBP+8]
00404F37  |C702 9C010000         MOV     DWORD PTR [EDX], 19C
00404F3D  |8D4D E4               LEA     ECX, [EBP-1C]
00404F40  |C645 FC 04            MOV     BYTE PTR [EBP-4], 4
00404F44  |E8 378B0000           CALL    0040DA80
00404F49  |8D4D EC               LEA     ECX, [EBP-14]
00404F4C  |C645 FC 03            MOV     BYTE PTR [EBP-4], 3
00404F50  |E8 2B8B0000           CALL    0040DA80
00404F55  |8D4D E0               LEA     ECX, [EBP-20]
00404F58  |C645 FC 02            MOV     BYTE PTR [EBP-4], 2
00404F5C  |E8 1F8B0000           CALL    0040DA80
00404F61  |8D4D F0               LEA     ECX, [EBP-10]
00404F64  |C645 FC 01            MOV     BYTE PTR [EBP-4], 1
00404F68  |E8 138B0000           CALL    0040DA80
00404F6D  |C745 FC FFFFFFFF      MOV     DWORD PTR [EBP-4], -1
00404F74  |E9 B6000000           JMP     0040502F
00404F79  \8B45 08               MOV     EAX, [EBP+8]
00404F7C   C700 9A010000         MOV     DWORD PTR [EAX], 19A
00404F82  /EB 10                 JMP     SHORT 00404F94
00404F84  |90                    NOP
00404F85  |90                    NOP
00404F86  |90                    NOP
00404F87  |90                    NOP
00404F88  |90                    NOP
00404F89  |90                    NOP
00404F8A  |90                    NOP
00404F8B  |90                    NOP
00404F8C  |90                    NOP
00404F8D  |90                    NOP
00404F8E  |90                    NOP
00404F8F  |90                    NOP
00404F90  |90                    NOP
00404F91  |90                    NOP
00404F92  |90                    NOP
00404F93  |90                    NOP
00404F94  \8D4D E4               LEA     ECX, [EBP-1C]          ; clean up
00404F97   C645 FC 04            MOV     BYTE PTR [EBP-4], 4
00404F9B   E8 E08A0000           CALL    0040DA80
00404FA0   8D4D EC               LEA     ECX, [EBP-14]
00404FA3   C645 FC 03            MOV     BYTE PTR [EBP-4], 3
00404FA7   E8 D48A0000           CALL    0040DA80
00404FAC   8D4D E0               LEA     ECX, [EBP-20]
00404FAF   C645 FC 02            MOV     BYTE PTR [EBP-4], 2
00404FB3   E8 C88A0000           CALL    0040DA80
00404FB8   8D4D F0               LEA     ECX, [EBP-10]
00404FBB   C645 FC 01            MOV     BYTE PTR [EBP-4], 1
00404FBF   E8 BC8A0000           CALL    0040DA80
00404FC4   8D4D E8               LEA     ECX, [EBP-18]
00404FC7   C745 FC FFFFFFFF      MOV     DWORD PTR [EBP-4], -1
00404FCE   E8 AD8A0000           CALL    0040DA80
00404FD3   5F                    POP     EDI
00404FD4   5E                    POP     ESI
00404FD5   B8 01000000           MOV     EAX, 1                 ; return true
00404FDA   5B                    POP     EBX
00404FDB   8B4D F4               MOV     ECX, [EBP-C]           ; 0012FEC0
00404FDE   64:890D 00000000      MOV     FS:[0], ECX            ; reset SE Handler
00404FE5   8BE5                  MOV     ESP, EBP
00404FE7   5D                    POP     EBP
00404FE8   C3                    RETN                           ; return to 005E1C79
00404FE9   8B4D 08               MOV     ECX, [EBP+8]
00404FEC   C701 96010000         MOV     DWORD PTR [ECX], 196
00404FF2   8D4D EC               LEA     ECX, [EBP-14]
00404FF5   C645 FC 03            MOV     BYTE PTR [EBP-4], 3
00404FF9   E8 828A0000           CALL    0040DA80
00404FFE   8D4D E0               LEA     ECX, [EBP-20]
00405001   C645 FC 02            MOV     BYTE PTR [EBP-4], 2
00405005   E8 768A0000           CALL    0040DA80
0040500A   EB 14                 JMP     SHORT 00405020
0040500C   8B55 08               MOV     EDX, [EBP+8]
0040500F   C702 95010000         MOV     DWORD PTR [EDX], 195
00405015   EB 09                 JMP     SHORT 00405020
00405017   8B45 08               MOV     EAX, [EBP+8]
0040501A   C700 94010000         MOV     DWORD PTR [EAX], 194
00405020   8D4D F0               LEA     ECX, [EBP-10]
00405023   C645 FC 01            MOV     BYTE PTR [EBP-4], 1
00405027   E8 548A0000           CALL    0040DA80
0040502C   897D FC               MOV     [EBP-4], EDI
0040502F   8D4D E8               LEA     ECX, [EBP-18]
00405032   E8 498A0000           CALL    0040DA80
00405037   8B4D F4               MOV     ECX, [EBP-C]
0040503A   5F                    POP     EDI
0040503B   5E                    POP     ESI
0040503C   33C0                  XOR     EAX, EAX               ; return false
0040503E   5B                    POP     EBX
0040503F   64:890D 00000000      MOV     FS:[0], ECX
00405046   8BE5                  MOV     ESP, EBP
00405048   5D                    POP     EBP
00405049   C3                    RETN
```

总结一下，“机关”用MACRO把细节保护起来，配合几个Winlicense SDK函数再次验证许可证的有效性。

采用在用户自定义数据里写入与许可证相同的用户名及`HWID`的方式进行判断，而且在"License Restrictions"里面必须同时设置"Days Expiration"和"Date Expiration"，否则程序将不能运行。

无疑增加了Keygen的难度，下图为最终Keygen时的设置，当然这个过程重复了几次，逐步接近真相。Keygen程序为Oreans在"WinLicense 2.2.1.0 demo"中“免费”提供的。

![](https://bbs.kanxue.com/upload/attach/201212/522307_yxxkn0h9620n9we.png)

附件`GetDuxiuStr.RePacked.by.MistHill.with.Key.7z`为原目标替换两个`RSAPublicKey`后重新打包，带一个我调试虚拟机上的有效许可证。

<http://bbs.pediy.com/attachment.php?attachmentid=74389&d=1355298773>

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`Fig01.png`](javascript:void(0);)（26.10kb，19次下载）
- [`Fig02.png`](javascript:void(0);)（31.27kb，9次下载）
- [`Fig03.png`](javascript:void(0);)（24.05kb，8次下载）
- [`Fig04.png`](javascript:void(0);)（24.95kb，7次下载）
- [`Fig05.png`](javascript:void(0);)（24.56kb，8次下载）
- [`Fig06.png`](javascript:void(0);)（12.94kb，8次下载）
