
```
 005369A9   .  FFD7           CALL    EDI                      ; 取用户名第 1 位;<&MSVBVM60.rtcMidCharBstr>  
 005369AB   .  8BD0           MOV     EDX, EAX  
 005369AD   .  8D4D A0        LEA     ECX, DWORD PTR SS:[EBP-60]  
 005369B0   .  FFD6           CALL    ESI  
 005369B2   .  50             PUSH    EAX  
 005369B3   .  FF15 50104000  CALL    DWORD PTR DS:[<&MSVBVM60.rtcAnsiVa>;  取用户名字符 ASCII 码  
 005369B9   .  8985 0CFFFFFF  MOV     DWORD PTR SS:[EBP-F4], EAX  
 005369BF   .  6A 01          PUSH    1  
 005369C1   .  8B55 A8        MOV     EDX, DWORD PTR SS:[EBP-58]  
 005369C4   .  52             PUSH    EDX  
 005369C5   .  FF15 E4124000  CALL    DWORD PTR DS:[<&MSVBVM60.rtcLeftCh>;  取注册码的左边第一位  
 005369CB   .  8BD0           MOV     EDX, EAX  
 005369CD   .  8D4D 98        LEA     ECX, DWORD PTR SS:[EBP-68]  
 005369D0   .  FFD6           CALL    ESI  
 005369D2   .  50             PUSH    EAX  
 005369D3   .  6A 01          PUSH    1  
 005369D5   .  8B85 0CFFFFFF  MOV     EAX, DWORD PTR SS:[EBP-F4]  
 005369DB   .  50             PUSH    EAX  
 005369DC   .  FF15 08104000  CALL    DWORD PTR DS:[<&MSVBVM60.__vbaStrI>;  用户名第 1 位的 ASCII 码转为十进制字串  
 005369E2   .  8BD0           MOV     EDX, EAX  
 005369E4   .  8D4D 9C        LEA     ECX, DWORD PTR SS:[EBP-64]  
 005369E7   .  FFD6           CALL    ESI  
 005369E9   .  50             PUSH    EAX  
 005369EA   .  FF15 08134000  CALL    DWORD PTR DS:[<&MSVBVM60.rtcRightC>;  取十进制字串的右边的 1 位字符  
 005369F0   .  8BD0           MOV     EDX, EAX  
 005369F2   .  8D4D 94        LEA     ECX, DWORD PTR SS:[EBP-6C]  
 005369F5   .  FFD6           CALL    ESI  
 005369F7   .  50             PUSH    EAX  
 005369F8   .  FF15 68114000  CALL    DWORD PTR DS:[<&MSVBVM60.__vbaStrC>;  与注册码左边第 1 位进行比较  
 005369FE   .  8BD8           MOV     EBX, EAX  
 00536A00   .  F7DB           NEG     EBX  
 00536A02   .  1BDB           SBB     EBX, EBX  
 00536A04   .  F7DB           NEG     EBX  
 00536A06   .  F7DB           NEG     EBX  
 00536A08   .  8D4D 94        LEA     ECX, DWORD PTR SS:[EBP-6C]  
 00536A0B   .  51             PUSH    ECX  
 00536A0C   .  8D55 98        LEA     EDX, DWORD PTR SS:[EBP-68]  
 00536A0F   .  52             PUSH    EDX  
 00536A10   .  8D45 9C        LEA     EAX, DWORD PTR SS:[EBP-64]  
 00536A13   .  50             PUSH    EAX  
 00536A14   .  8D4D A0        LEA     ECX, DWORD PTR SS:[EBP-60]  
 00536A17   .  51             PUSH    ECX  
 00536A18   .  6A 04          PUSH    4  
 00536A1A   .  FF15 84124000  CALL    DWORD PTR DS:[<&MSVBVM60.__vbaFree>;  MSVBVM60.__vbaFreeStrList  
 00536A20   .  83C4 14        ADD     ESP, 14  
 00536A23   .  8D4D 80        LEA     ECX, DWORD PTR SS:[EBP-80]  
 00536A26   .  FF15 20104000  CALL    DWORD PTR DS:[<&MSVBVM60.__vbaFree>;  MSVBVM60.__vbaFreeVar  
 00536A2C   .  66:85DB       TEST    BX, BX  
 00536A2F   .  74 5B         JE      SHORT 00536A8C                     ;  这里要跳（要爆就要改为 JMP）  
 00536A31   .  8B45 A4        MOV     EAX, DWORD PTR SS:[EBP-5C]  
 00536A34   .  85C0           TEST    EAX, EAX  
 00536A36   .  75 12          JNZ     SHORT 00536A4A  
 00536A38   .  8D55 A4        LEA     EDX, DWORD PTR SS:[EBP-5C]  
 00536A3B   .  52             PUSH    EDX  
 00536A3C   .  68 88774000    PUSH    00407788  
 00536A41   .  FF15 58124000  CALL    DWORD PTR DS:[<&MSVBVM60.__vbaNew2>;  MSVBVM60.__vbaNew2  
 00536A47   .  8B45 A4        MOV     EAX, DWORD PTR SS:[EBP-5C]  
 00536A4A   >  8BF0           MOV     ESI, EAX  
 00536A4C   .  C785 0CFFFFFF>MOV     DWORD PTR SS:[EBP-F4], 0  
 00536A56   .  8B08           MOV     ECX, DWORD PTR DS:[EAX]  
 00536A58   .  8D95 08FFFFFF  LEA     EDX, DWORD PTR SS:[EBP-F8]  
 00536A5E   .  52             PUSH    EDX  
 00536A5F   .  8D95 0CFFFFFF  LEA     EDX, DWORD PTR SS:[EBP-F4]  
 00536A65   .  52             PUSH    EDX  
 00536A66   .  8D55 DC        LEA     EDX, DWORD PTR SS:[EBP-24]  
 00536A69   .  52             PUSH    EDX  
 00536A6A   .  50             PUSH    EAX  
 00536A6B   .  FF51 24        CALL    DWORD PTR DS:[ECX+24]  
 00536A6E   .  DBE2           FCLEX  
 00536A70   .  85C0           TEST    EAX, EAX  
 00536A72   .  0F8D 51080000  JGE     005372C9  
 00536A78   .  6A 24          PUSH    24  
 00536A7A   .  68 F0684100    PUSH    004168F0  
 00536A7F   .  56             PUSH    ESI  
 00536A80   .  50             PUSH    EAX  
 00536A81   .  FF15 A8104000  CALL    DWORD PTR DS:[<&MSVBVM60.__vbaHres>;  MSVBVM60.__vbaHresultCheckObj  
 00536A87   .  E9 3D080000    JMP     005372C9  
 00536A8C   >  BB 01000000    MOV     EBX, 1  
 00536A91   .  899D 78FFFFFF  MOV     DWORD PTR SS:[EBP-88], EBX  
 00536A97   .  C785 70FFFFFF>MOV     DWORD PTR SS:[EBP-90], 2  
 00536AA1   .  8D85 70FFFFFF  LEA     EAX, DWORD PTR SS:[EBP-90]  
 00536AA7   .  50             PUSH    EAX  
 00536AA8   .  6A 02          PUSH    2  
 00536AAA   .  8B4D E0        MOV     ECX, DWORD PTR SS:[EBP-20]  
 00536AAD   .  51             PUSH    ECX  
 00536AAE   .  FFD7           CALL    EDI                      ;  取用户名第 2 位  
 00536AB0   .  8BD0           MOV     EDX, EAX  
 00536AB2   .  8D4D A0        LEA     ECX, DWORD PTR SS:[EBP-60]  
 00536AB5   .  FFD6           CALL    ESI  
 00536AB7   .  50             PUSH    EAX  
 00536AB8   .  FF15 50104000  CALL    DWORD PTR DS:[<&MSVBVM60.rtcAnsiVa>;  取第 2 位 ASCII 码  
 00536ABE   .  8985 0CFFFFFF  MOV     DWORD PTR SS:[EBP-F4], EAX  
 00536AC4   .  895D 88        MOV     DWORD PTR SS:[EBP-78], EBX  
 00536AC7   .  C745 80 02000>MOV     DWORD PTR SS:[EBP-80], 2  
 00536ACE   .  8D55 80        LEA     EDX, DWORD PTR SS:[EBP-80]  
 00536AD1   .  52             PUSH    EDX  
 00536AD2   .  6A 02          PUSH    2  
 00536AD4   .  8B45 A8        MOV     EAX, DWORD PTR SS:[EBP-58]  
 00536AD7   .  50             PUSH    EAX  
 00536AD8   .  FFD7           CALL    EDI                      ;  取注册码的第 2 位  
 00536ADA   .  8BD0           MOV     EDX, EAX  
 00536ADC   .  8D4D 98        LEA     ECX, DWORD PTR SS:[EBP-68]  
 00536ADF   .  FFD6           CALL    ESI  
 00536AE1   .  50             PUSH    EAX  
 00536AE2   .  53             PUSH    EBX  
 00536AE3   .  8B8D 0CFFFFFF  MOV     ECX, DWORD PTR SS:[EBP-F4]  
 00536AE9   .  51             PUSH    ECX  
 00536AEA   .  FF15 08104000  CALL    DWORD PTR DS:[<&MSVBVM60.__vbaStrI>;  第 2 位用户名的 ASCII 码转为十进制串  
 00536AF0   .  8BD0           MOV     EDX, EAX  
 00536AF2   .  8D4D 9C        LEA     ECX, DWORD PTR SS:[EBP-64]  
 00536AF5   .  FFD6           CALL    ESI  
 00536AF7   .  50             PUSH    EAX  
 00536AF8   .  FF15 08134000  CALL    DWORD PTR DS:[<&MSVBVM60.rtcRightC>;  取十进制串的右边一位  
 00536AFE   .  8BD0           MOV     EDX, EAX  
 00536B00   .  8D4D 94        LEA     ECX, DWORD PTR SS:[EBP-6C]  
 00536B03   .  FFD6           CALL    ESI  
 00536B05   .  50             PUSH    EAX  
 00536B06   .  FF15 68114000  CALL    DWORD PTR DS:[<&MSVBVM60.__vbaStrC>;  与注册码第 2 位进行比较  
 00536B0C   .  8BD8           MOV     EBX, EAX  
 00536B0E   .  F7DB           NEG     EBX  
 00536B10   .  1BDB           SBB     EBX, EBX  
 00536B12   .  F7DB           NEG     EBX  
 00536B14   .  F7DB           NEG     EBX  
 00536B16   .  8D55 94        LEA     EDX, DWORD PTR SS:[EBP-6C]  
 00536B19   .  52             PUSH    EDX  
 00536B1A   .  8D45 98        LEA     EAX, DWORD PTR SS:[EBP-68]  
 00536B1D   .  50             PUSH    EAX  
 00536B1E   .  8D4D 9C        LEA     ECX, DWORD PTR SS:[EBP-64]  
```
