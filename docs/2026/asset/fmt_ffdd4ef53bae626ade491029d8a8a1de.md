
```
00414A74    90              NOP
00414A75    90              NOP
00414A76    90              NOP
00414A77    90              NOP
00414A78    90              NOP
00414A79    8A07            MOV AL,BYTE PTR DS:[EDI]
00414A7B    02C1            ADD AL,CL
00414A7D    C0C0 43         ROL AL,43                                ; Shift constant out of range 1..31
00414A80    FEC8            DEC AL
00414A82    04 40           ADD AL,40
00414A84    2C 39           SUB AL,39
00414A86    EB 01           JMP SHORT 00414A89
00414A88    90              NOP
00414A89    34 BB           XOR AL,0BB
00414A8B    0AC0            OR AL,AL
00414A8D    04 85           ADD AL,85
00414A8F    EB 01           JMP SHORT 00414A92
00414A91    90              NOP
00414A92    02C1            ADD AL,CL
00414A94    90              NOP
00414A95    F9              STC
00414A96    C0C8 53         ROR AL,53                                ; Shift constant out of range 1..31
00414A99    0AC0            OR AL,AL
00414A9B    04 C2           ADD AL,0C2
00414A9D    2AC1            SUB AL,CL
00414A9F    AA              STOS BYTE PTR ES:[EDI]
00414AA0    49              DEC ECX
00414AA1    9C              PUSHFD
00414AA2    C12C24 06       SHR DWORD PTR SS:[ESP],6
00414AA6    F71424          NOT DWORD PTR SS:[ESP]
00414AA9    832424 01       AND DWORD PTR SS:[ESP],1
00414AAD    50              PUSH EAX
00414AAE    52              PUSH EDX
00414AAF    B8 61B2DC12     MOV EAX,12DCB261
00414AB4    05 444D23ED     ADD EAX,ED234D44
00414AB9    F76424 08       MUL DWORD PTR SS:[ESP+8]
00414ABD    8D8428 D9564000 LEA EAX,DWORD PTR DS:[EAX+EBP+4056D9]
00414AC4    894424 08       MOV DWORD PTR SS:[ESP+8],EAX             ; PESpin.00414AD4
00414AC8    5A              POP EDX
00414AC9    58              POP EAX
00414ACA    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]
00414ACE    FF6424 FC       JMP DWORD PTR SS:[ESP-4]                 ; 如果没有解压完则继续
……
00412777    68 07000000     PUSH 7
0041277C    5B              POP EBX
0041277D    25 25382C37     AND EAX,372C3825
00412782    50              PUSH EAX
00412783    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]
00412787    F7D0            NOT EAX
00412789    234424 FC       AND EAX,DWORD PTR SS:[ESP-4]
0041278D    51              PUSH ECX                                 ; 从这里开始解密各段
0041278E    90              NOP
0041278F    90              NOP
00412790    90              NOP
00412791    90              NOP
00412792    90              NOP
00412793    90              NOP
00412794    90              NOP
00412795    90              NOP
00412796    90              NOP
00412797    90              NOP
00412798    90              NOP
00412799    90              NOP
0041279A    0FA3C3          BT EBX,EAX
0041279D    73 79           JNB SHORT 00412818                       ; 如果该段解压完则跳去解压下一段
0041279F    90              NOP
004127A0    90              NOP
004127A1    90              NOP
004127A2    90              NOP
004127A3    90              NOP
004127A4    90              NOP
004127A5    90              NOP
004127A6    90              NOP
004127A7    90              NOP
004127A8    90              NOP
004127A9    90              NOP
004127AA    90              NOP
004127AB    90              NOP
004127AC    90              NOP
004127AD    90              NOP
004127AE    90              NOP
004127AF    90              NOP
004127B0    90              NOP
004127B1    90              NOP
004127B2    90              NOP
004127B3    90              NOP
004127B4    90              NOP
004127B5    90              NOP
004127B6    90              NOP
004127B7    90              NOP
004127B8    90              NOP
004127B9    90              NOP
004127BA    90              NOP
004127BB    90              NOP
004127BC    90              NOP
004127BD    90              NOP
004127BE    90              NOP
004127BF    90              NOP
004127C0    90              NOP
004127C1    90              NOP
004127C2    90              NOP
004127C3    90              NOP
004127C4    90              NOP
004127C5    90              NOP
004127C6    90              NOP
004127C7    90              NOP
004127C8    90              NOP
004127C9    90              NOP
004127CA    90              NOP
004127CB    90              NOP
004127CC    90              NOP
004127CD    8B7A 0C         MOV EDI,DWORD PTR DS:[EDX+C]
004127D0    03BD C34B4000   ADD EDI,DWORD PTR SS:[EBP+404BC3]
004127D6    8B4A 10         MOV ECX,DWORD PTR DS:[EDX+10]            ; RSIZE = 6000
004127D9    50              PUSH EAX
004127DA    8A07            MOV AL,BYTE PTR DS:[EDI]                 ; 第一次 从401000处开始解密代码，size:6000
004127DC    2C 61           SUB AL,61
004127DE    F8              CLC
004127DF    F8              CLC
004127E0    C0C0 B1         ROL AL,0B1                               ; Shift constant out of range 1..31
004127E3    34 AF           XOR AL,0AF
004127E5    04 70           ADD AL,70
004127E7    FEC8            DEC AL
004127E9    EB 01           JMP SHORT 004127EC
004127EB    90              NOP
004127EC    F8              CLC
004127ED    32C1            XOR AL,CL
004127EF    C0C0 42         ROL AL,42                                ; Shift constant out of range 1..31
004127F2    EB 01           JMP SHORT 004127F5
004127F4    90              NOP
004127F5    02C1            ADD AL,CL
004127F7    2AC1            SUB AL,CL
004127F9    34 04           XOR AL,4
004127FB    C0C0 9B         ROL AL,9B                                ; Shift constant out of range 1..31
004127FE    FEC8            DEC AL
00412800    AA              STOS BYTE PTR ES:[EDI]
00412801    49              DEC ECX
00412802    90              NOP
00412803    90              NOP
00412804    90              NOP
00412805    90              NOP
00412806    90              NOP
00412807    90              NOP
00412808    90              NOP
00412809    90              NOP
0041280A    90              NOP
0041280B    90              NOP
0041280C    90              NOP
0041280D    90              NOP
0041280E    90              NOP
0041280F    90              NOP
00412810    90              NOP
00412811    90              NOP
00412812    90              NOP
00412813    0BC9            OR ECX,ECX
00412815  ^ 75 C3           JNZ SHORT 004127DA                       ; 该段没解压完该段则继续上去解密
00412817    58              POP EAX
00412818    40              INC EAX
00412819    83C2 28         ADD EDX,28
0041281C    90              NOP
0041281D    90              NOP
0041281E    90              NOP
0041281F    90              NOP
00412820    90              NOP
00412821    90              NOP
00412822    90              NOP
00412823    90              NOP
00412824    90              NOP
00412825    59              POP ECX
00412826    49              DEC ECX
00412827    9C              PUSHFD
00412828    C12C24 06       SHR DWORD PTR SS:[ESP],6
0041282C    F71424          NOT DWORD PTR SS:[ESP]
0041282F    832424 01       AND DWORD PTR SS:[ESP],1
00412833    50              PUSH EAX
00412834    52              PUSH EDX
00412835    B8 E979A6F5     MOV EAX,F5A679E9
0041283A    05 4985590A     ADD EAX,0A598549
0041283F    F76424 08       MUL DWORD PTR SS:[ESP+8]
00412843    8D8428 60344000 LEA EAX,DWORD PTR DS:[EAX+EBP+403460]
0041284A    894424 08       MOV DWORD PTR SS:[ESP+8],EAX
0041284E    5A              POP EDX
0041284F    58              POP EAX
00412850    8D6424 04       LEA ESP,DWORD PTR SS:[ESP+4]
00412854    FF6424 FC       JMP DWORD PTR SS:[ESP-4]                 ; 没有解压完则继续回去解密
……
0041286B    E8 BA1C0000     CALL 0041452A                            ; 这个CALL实际就是一个异常CALL
……
00415062    6A 04           PUSH 4                                         ; /Protect = PAGE_READWRITE
```
