
```
005780E0 <Check>           > \6A FF         PUSH -0x1

005780E2                   .  68 324B7000   PUSH 010Edito.00704B32

005780E7                   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]

005780ED                   .  50            PUSH EAX

005780EE                   .  83EC 1C       SUB ESP,0x1C

005780F1                   .  53            PUSH EBX

005780F2                   .  55            PUSH EBP

005780F3                   .  56            PUSH ESI

005780F4                   .  57            PUSH EDI

005780F5                   .  A1 40488B00   MOV EAX,DWORD PTR DS:[0x8B4840]          ;  7ACAF566

005780FA                   .  33C4          XOR EAX,ESP

005780FC                   .  50            PUSH EAX

005780FD                   .  8D4424 30     LEA EAX,DWORD PTR SS:[ESP+0x30]

00578101                   .  64:A3 0000000>MOV DWORD PTR FS:[0],EAX                 ;  seh

00578107                   .  8BF1          MOV ESI,ECX

00578109                   .  33DB          XOR EBX,EBX

0057810B                   .  895C24 20     MOV DWORD PTR SS:[ESP+0x20],EBX

0057810F                   .  A1 C0978B00   MOV EAX,DWORD PTR DS:[<&QtCore4.?shared_>

00578114                   .  894424 18     MOV DWORD PTR SS:[ESP+0x18],EAX

00578118                   .  B9 01000000   MOV ECX,0x1

0057811D                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

00578121                   .  A1 C0978B00   MOV EAX,DWORD PTR DS:[<&QtCore4.?shared_>

00578126                   .  895C24 38     MOV DWORD PTR SS:[ESP+0x38],EBX

0057812A                   .  894424 1C     MOV DWORD PTR SS:[ESP+0x1C],EAX

0057812E                   .  BA 01000000   MOV EDX,0x1

00578133                   .  F0:0FC110     LOCK XADD DWORD PTR DS:[EAX],EDX

00578137                   .  8B4E 68       MOV ECX,DWORD PTR DS:[ESI+0x68]

0057813A                   .  8B3D F0AC8B00 MOV EDI,DWORD PTR DS:[<&QtGui4.?text@QLi>;  QtGui4.?text@QLineEdit@@QBE?AVQString@@XZ

00578140                   .  8D4424 24     LEA EAX,DWORD PTR SS:[ESP+0x24]

00578144                   .  50            PUSH EAX                                 ;  QString 对象 用户名

00578145                   .  C64424 3C 01  MOV BYTE PTR SS:[ESP+0x3C],0x1

0057814A                   .  FFD7          CALL EDI                                 ;  <&QtGui4.?text@QLineEdit@@QBE?AVQString@@XZ>

0057814C                   .  8B08          MOV ECX,DWORD PTR DS:[EAX]

0057814E                   .  3959 08       CMP DWORD PTR DS:[ECX+0x8],EBX           ;  取用户名

00578151                   .  8B5424 24     MOV EDX,DWORD PTR SS:[ESP+0x24]

00578155                   .  0F94C3        SETE BL

00578158                   .  83C8 FF       OR EAX,0xFFFFFFFF

0057815B                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

0057815F                   .  8B2D CC978B00 MOV EBP,DWORD PTR DS:[<&QtCore4.?free@QS>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

00578165                   .  75 0A         JNZ SHORT 010Edito.00578171

00578167                   .  8B4C24 24     MOV ECX,DWORD PTR SS:[ESP+0x24]

0057816B                   .  51            PUSH ECX

0057816C                   .  FFD5          CALL EBP                                 ;  <&QtCore4.?free@QString@@CAXPAUData@1@@Z>

0057816E                   .  83C4 04       ADD ESP,0x4

00578171                   >  84DB          TEST BL,BL                               ;  判断用户名是否为空

00578173                   .  0F84 8D000000 JE 010Edito.00578206

00578179                   .  6A FF         PUSH -0x1

0057817B                   .  68 54CC7400   PUSH 010Edito.0074CC54                   ;  please enter a name.

00578180                   .  FF15 84978B00 CALL DWORD PTR DS:[<&QtCore4.?fromAscii_>;  QtCore4.?fromAscii_helper@QString@@CAPAUData@1@PBDH@Z

00578186                   .  894424 28     MOV DWORD PTR SS:[ESP+0x28],EAX

0057818A                   .  8D5424 28     LEA EDX,DWORD PTR SS:[ESP+0x28]

0057818E                   .  52            PUSH EDX

0057818F                   .  C64424 44 02  MOV BYTE PTR SS:[ESP+0x44],0x2

00578194                   .  E8 7E95E8FF   CALL 010Edito.00401717

00578199                   .  8B4424 2C     MOV EAX,DWORD PTR SS:[ESP+0x2C]

0057819D                   .  83C4 0C       ADD ESP,0xC

005781A0                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

005781A5                   .  83C9 FF       OR ECX,0xFFFFFFFF

005781A8                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

005781AC                   .  75 0A         JNZ SHORT 010Edito.005781B8

005781AE                   .  8B5424 20     MOV EDX,DWORD PTR SS:[ESP+0x20]

005781B2                   .  52            PUSH EDX

005781B3                   .  FFD5          CALL EBP

005781B5                   .  83C4 04       ADD ESP,0x4

005781B8                   >  8B76 68       MOV ESI,DWORD PTR DS:[ESI+0x68]

005781BB                   .  6A 07         PUSH 0x7

005781BD                   .  8BCE          MOV ECX,ESI

005781BF                   .  FF15 B0B38B00 CALL DWORD PTR DS:[<&QtGui4.?setFocus@QW>;  QtGui4.?setFocus@QWidget@@QAEXW4FocusReason@Qt@@@Z

005781C5                   .  8B4424 1C     MOV EAX,DWORD PTR SS:[ESP+0x1C]

005781C9                   .  C64424 38 00  MOV BYTE PTR SS:[ESP+0x38],0x0

005781CE                   .  83C9 FF       OR ECX,0xFFFFFFFF

005781D1                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

005781D5                   .  75 0A         JNZ SHORT 010Edito.005781E1

005781D7                   .  8B5424 1C     MOV EDX,DWORD PTR SS:[ESP+0x1C]

005781DB                   .  52            PUSH EDX

005781DC                   .  FFD5          CALL EBP

005781DE                   .  83C4 04       ADD ESP,0x4

005781E1                   >  8B4424 18     MOV EAX,DWORD PTR SS:[ESP+0x18]

005781E5                   .  C74424 38 FFF>MOV DWORD PTR SS:[ESP+0x38],-0x1

005781ED                   .  83C9 FF       OR ECX,0xFFFFFFFF

005781F0                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

005781F4                   .  0F85 34080000 JNZ 010Edito.00578A2E

005781FA                   .  8B5424 18     MOV EDX,DWORD PTR SS:[ESP+0x18]

005781FE                   .  52            PUSH EDX

005781FF                   .  FFD5          CALL EBP

00578201                   .  E9 25080000   JMP 010Edito.00578A2B

00578206                   >  8B4E 70       MOV ECX,DWORD PTR DS:[ESI+0x70]

00578209                   .  8D4424 2C     LEA EAX,DWORD PTR SS:[ESP+0x2C]

0057820D                   .  50            PUSH EAX

0057820E                   .  FFD7          CALL EDI                                 ;  (QtGui4.?text@QLineEdit@@QBE?AVQString@@XZ)

00578210                   .  8B08          MOV ECX,DWORD PTR DS:[EAX]

00578212                   .  8379 08 00    CMP DWORD PTR DS:[ECX+0x8],0x0           ;  注册码长度

00578216                   .  8B2D 84978B00 MOV EBP,DWORD PTR DS:[<&QtCore4.?fromAsc>;  QtCore4.?fromAscii_helper@QString@@CAPAUData@1@PBDH@Z

0057821C                   .  BB 01000000   MOV EBX,0x1

00578221                   .  C64424 38 03  MOV BYTE PTR SS:[ESP+0x38],0x3

00578226                   .  895C24 20     MOV DWORD PTR SS:[ESP+0x20],EBX

0057822A                   .  74 56         JE SHORT 010Edito.00578282

0057822C                   .  6A FF         PUSH -0x1

0057822E                   .  68 F04F7300   PUSH 010Edito.00734FF0

00578233                   .  FFD5          CALL EBP                                 ;  <&QtCore4.?fromAscii_helper@QString@@CAPAUData@1@PBDH@Z>

00578235                   .  83C4 08       ADD ESP,0x8

00578238                   .  894424 24     MOV DWORD PTR SS:[ESP+0x24],EAX

0057823C                   .  8B4E 70       MOV ECX,DWORD PTR DS:[ESI+0x70]

0057823F                   .  8D5424 28     LEA EDX,DWORD PTR SS:[ESP+0x28]

00578243                   .  52            PUSH EDX

00578244                   .  C74424 3C 040>MOV DWORD PTR SS:[ESP+0x3C],0x4

0057824C                   .  C74424 24 030>MOV DWORD PTR SS:[ESP+0x24],0x3

00578254                   .  FFD7          CALL EDI                                 ;  还是注册码？

00578256                   .  6A 01         PUSH 0x1

00578258                   .  6A 00         PUSH 0x0

0057825A                   .  8D4C24 2C     LEA ECX,DWORD PTR SS:[ESP+0x2C]

0057825E                   .  51            PUSH ECX

0057825F                   .  BB 07000000   MOV EBX,0x7

00578264                   .  8BC8          MOV ECX,EAX

00578266                   .  C74424 44 050>MOV DWORD PTR SS:[ESP+0x44],0x5

0057826E                   .  895C24 2C     MOV DWORD PTR SS:[ESP+0x2C],EBX

00578272                   .  FF15 40978B00 CALL DWORD PTR DS:[<&QtCore4.?indexOf@QS>;  QtCore4.?indexOf@QString@@QBEHABV1@HW4CaseSensitivity@Qt@@@Z

00578278                   .  C64424 17 00  MOV BYTE PTR SS:[ESP+0x17],0x0

0057827D                   .  83F8 FF       CMP EAX,-0x1

00578280                   .  74 05         JE SHORT 010Edito.00578287

00578282                   >  C64424 17 01  MOV BYTE PTR SS:[ESP+0x17],0x1

00578287                   >  C74424 38 040>MOV DWORD PTR SS:[ESP+0x38],0x4

0057828F                   .  F6C3 04       TEST BL,0x4

00578292                   .  74 22         JE SHORT 010Edito.005782B6

00578294                   .  8B5424 28     MOV EDX,DWORD PTR SS:[ESP+0x28]          ;  Qstring_PASS 对象首地址

00578298                   .  83E3 FB       AND EBX,0xFFFFFFFB

0057829B                   .  895C24 20     MOV DWORD PTR SS:[ESP+0x20],EBX

0057829F                   .  83C8 FF       OR EAX,0xFFFFFFFF

005782A2                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

005782A6                   .  75 0E         JNZ SHORT 010Edito.005782B6

005782A8                   .  8B4C24 28     MOV ECX,DWORD PTR SS:[ESP+0x28]

005782AC                   .  51            PUSH ECX

005782AD                   .  FF15 CC978B00 CALL DWORD PTR DS:[<&QtCore4.?free@QStri>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

005782B3                   .  83C4 04       ADD ESP,0x4

005782B6                   >  C74424 38 030>MOV DWORD PTR SS:[ESP+0x38],0x3

005782BE                   .  F6C3 02       TEST BL,0x2

005782C1                   .  74 22         JE SHORT 010Edito.005782E5

005782C3                   .  8B5424 24     MOV EDX,DWORD PTR SS:[ESP+0x24]

005782C7                   .  83E3 FD       AND EBX,0xFFFFFFFD

005782CA                   .  895C24 20     MOV DWORD PTR SS:[ESP+0x20],EBX

005782CE                   .  83C8 FF       OR EAX,0xFFFFFFFF

005782D1                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

005782D5                   .  75 0E         JNZ SHORT 010Edito.005782E5

005782D7                   .  8B4C24 24     MOV ECX,DWORD PTR SS:[ESP+0x24]

005782DB                   .  51            PUSH ECX

005782DC                   .  FF15 CC978B00 CALL DWORD PTR DS:[<&QtCore4.?free@QStri>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

005782E2                   .  83C4 04       ADD ESP,0x4

005782E5                   >  C74424 38 010>MOV DWORD PTR SS:[ESP+0x38],0x1

005782ED                   .  F6C3 01       TEST BL,0x1

005782F0                   .  74 1B         JE SHORT 010Edito.0057830D

005782F2                   .  8B5424 2C     MOV EDX,DWORD PTR SS:[ESP+0x2C]

005782F6                   .  83C8 FF       OR EAX,0xFFFFFFFF

005782F9                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

005782FD                   .  75 0E         JNZ SHORT 010Edito.0057830D

005782FF                   .  8B4C24 2C     MOV ECX,DWORD PTR SS:[ESP+0x2C]

00578303                   .  51            PUSH ECX

00578304                   .  FF15 CC978B00 CALL DWORD PTR DS:[<&QtCore4.?free@QStri>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

0057830A                   .  83C4 04       ADD ESP,0x4

0057830D                   >  807C24 17 00  CMP BYTE PTR SS:[ESP+0x17],0x0

00578312                   .  0F84 8F000000 JE 010Edito.005783A7

00578318                   .  6A FF         PUSH -0x1

0057831A                   .  68 24CC7400   PUSH 010Edito.0074CC24                   ;  please completely enter your password.

0057831F                   .  FFD5          CALL EBP

00578321                   .  894424 2C     MOV DWORD PTR SS:[ESP+0x2C],EAX

00578325                   .  8D5424 2C     LEA EDX,DWORD PTR SS:[ESP+0x2C]

00578329                   .  52            PUSH EDX

0057832A                   .  C64424 44 06  MOV BYTE PTR SS:[ESP+0x44],0x6

0057832F                   .  E8 E393E8FF   CALL 010Edito.00401717

00578334                   .  8B4424 30     MOV EAX,DWORD PTR SS:[ESP+0x30]

00578338                   .  8B3D CC978B00 MOV EDI,DWORD PTR DS:[<&QtCore4.?free@QS>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

0057833E                   .  83C4 0C       ADD ESP,0xC

00578341                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

00578346                   .  83C9 FF       OR ECX,0xFFFFFFFF

00578349                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

0057834D                   .  75 0A         JNZ SHORT 010Edito.00578359

0057834F                   .  8B5424 24     MOV EDX,DWORD PTR SS:[ESP+0x24]

00578353                   .  52            PUSH EDX

00578354                   .  FFD7          CALL EDI

00578356                   .  83C4 04       ADD ESP,0x4

00578359                   >  8B76 68       MOV ESI,DWORD PTR DS:[ESI+0x68]

0057835C                   .  6A 07         PUSH 0x7

0057835E                   .  8BCE          MOV ECX,ESI

00578360                   .  FF15 B0B38B00 CALL DWORD PTR DS:[<&QtGui4.?setFocus@QW>;  QtGui4.?setFocus@QWidget@@QAEXW4FocusReason@Qt@@@Z

00578366                   .  8B4424 1C     MOV EAX,DWORD PTR SS:[ESP+0x1C]

0057836A                   .  C64424 38 00  MOV BYTE PTR SS:[ESP+0x38],0x0

0057836F                   .  83C9 FF       OR ECX,0xFFFFFFFF

00578372                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

00578376                   .  75 0A         JNZ SHORT 010Edito.00578382

00578378                   .  8B5424 1C     MOV EDX,DWORD PTR SS:[ESP+0x1C]

0057837C                   .  52            PUSH EDX

0057837D                   .  FFD7          CALL EDI

0057837F                   .  83C4 04       ADD ESP,0x4

00578382                   >  8B4424 18     MOV EAX,DWORD PTR SS:[ESP+0x18]

00578386                   .  C74424 38 FFF>MOV DWORD PTR SS:[ESP+0x38],-0x1

0057838E                   .  83C9 FF       OR ECX,0xFFFFFFFF

00578391                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

00578395                   .  0F85 93060000 JNZ 010Edito.00578A2E

0057839B                   .  8B5424 18     MOV EDX,DWORD PTR SS:[ESP+0x18]

0057839F                   .  52            PUSH EDX

005783A0                   .  FFD7          CALL EDI

005783A2                   .  E9 84060000   JMP 010Edito.00578A2B

005783A7                   >  8B4E 70       MOV ECX,DWORD PTR DS:[ESI+0x70]

005783AA                   .  8D4424 2C     LEA EAX,DWORD PTR SS:[ESP+0x2C]

005783AE                   .  50            PUSH EAX

005783AF                   .  FFD7          CALL EDI                                 ;  注册码

005783B1                   .  8B1D B8978B00 MOV EBX,DWORD PTR DS:[<&QtCore4.??4QStri>;  QtCore4.??4QString@@QAEAAV0@ABV0@@Z

005783B7                   .  50            PUSH EAX

005783B8                   .  8D4C24 20     LEA ECX,DWORD PTR SS:[ESP+0x20]

005783BC                   .  C64424 3C 07  MOV BYTE PTR SS:[ESP+0x3C],0x7

005783C1                   .  FFD3          CALL EBX                                 ;  <&QtCore4.??4QString@@QAEAAV0@ABV0@@Z>

005783C3                   .  8B4C24 2C     MOV ECX,DWORD PTR SS:[ESP+0x2C]

005783C7                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

005783CC                   .  83CA FF       OR EDX,0xFFFFFFFF

005783CF                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

005783D3                   .  75 0E         JNZ SHORT 010Edito.005783E3

005783D5                   .  8B4424 2C     MOV EAX,DWORD PTR SS:[ESP+0x2C]

005783D9                   .  50            PUSH EAX

005783DA                   .  FF15 CC978B00 CALL DWORD PTR DS:[<&QtCore4.?free@QStri>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

005783E0                   .  83C4 04       ADD ESP,0x4

005783E3                   >  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+0x1C]

005783E7                   .  51            PUSH ECX

005783E8                   .  8BCE          MOV ECX,ESI

005783EA                   .  E8 3918E9FF   CALL 010Edito.00409C28

005783EF                   .  85C0          TEST EAX,EAX

005783F1                   .  74 3D         JE SHORT 010Edito.00578430

005783F3                   .  6A 13         PUSH 0x13

005783F5                   .  8D5424 30     LEA EDX,DWORD PTR SS:[ESP+0x30]

005783F9                   .  52            PUSH EDX

005783FA                   .  8D4C24 24     LEA ECX,DWORD PTR SS:[ESP+0x24]

005783FE                   .  FF15 AC978B00 CALL DWORD PTR DS:[<&QtCore4.?left@QStri>;  QtCore4.?left@QString@@QBE?AV1@H@Z

00578404                   .  50            PUSH EAX

00578405                   .  8D4C24 20     LEA ECX,DWORD PTR SS:[ESP+0x20]

00578409                   .  C64424 3C 08  MOV BYTE PTR SS:[ESP+0x3C],0x8

0057840E                   .  FFD3          CALL EBX

00578410                   .  8B4424 2C     MOV EAX,DWORD PTR SS:[ESP+0x2C]

00578414                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

00578419                   .  83C9 FF       OR ECX,0xFFFFFFFF

0057841C                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

00578420                   .  75 0E         JNZ SHORT 010Edito.00578430

00578422                   .  8B5424 2C     MOV EDX,DWORD PTR SS:[ESP+0x2C]

00578426                   .  52            PUSH EDX

00578427                   .  FF15 CC978B00 CALL DWORD PTR DS:[<&QtCore4.?free@QStri>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

0057842D                   .  83C4 04       ADD ESP,0x4

00578430                   >  8B4E 68       MOV ECX,DWORD PTR DS:[ESI+0x68]

00578433                   .  8D4424 2C     LEA EAX,DWORD PTR SS:[ESP+0x2C]

00578437                   .  50            PUSH EAX                                 ;  用户名

00578438                   .  FFD7          CALL EDI

0057843A                   .  8B0D CC4D8B00 MOV ECX,DWORD PTR DS:[0x8B4DCC]

00578440                   .  50            PUSH EAX

00578441                   .  C64424 3C 09  MOV BYTE PTR SS:[ESP+0x3C],0x9

00578446                   .  E8 DF06E9FF   CALL 010Edito.00408B2A

0057844B                   .  8B4C24 2C     MOV ECX,DWORD PTR SS:[ESP+0x2C]

0057844F                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

00578454                   .  83CA FF       OR EDX,0xFFFFFFFF

00578457                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

0057845B                   .  75 0E         JNZ SHORT 010Edito.0057846B

0057845D                   .  8B4424 2C     MOV EAX,DWORD PTR SS:[ESP+0x2C]

00578461                   .  50            PUSH EAX

00578462                   .  FF15 CC978B00 CALL DWORD PTR DS:[<&QtCore4.?free@QStri>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

00578468                   .  83C4 04       ADD ESP,0x4

0057846B                   >  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+0x1C]

0057846F                   .  51            PUSH ECX                                 ;  注册码

00578470                   .  8B0D CC4D8B00 MOV ECX,DWORD PTR DS:[0x8B4DCC]

00578476                   .  E8 60A1E8FF   CALL 010Edito.004025DB

0057847B                   .  8B0D CC4D8B00 MOV ECX,DWORD PTR DS:[0x8B4DCC]

00578481                   .  68 23400000   PUSH 0x4023

00578486                   .  6A 07         PUSH 0x7

00578488                   .  E8 7E11E9FF   CALL 010Edito.0040960B                   ;  第一次验证

0057848D                   .  8B0D CC4D8B00 MOV ECX,DWORD PTR DS:[0x8B4DCC]

00578493                   .  68 23400000   PUSH 0x4023

00578498                   .  6A 07         PUSH 0x7

0057849A                   .  8BD8          MOV EBX,EAX

0057849C                   .  E8 D807E9FF   CALL 010Edito.00408C79                   ;  第二次验证

005784A1                   .  8B0D CC4D8B00 MOV ECX,DWORD PTR DS:[0x8B4DCC]          ;  结合下边看只有eax==0xDB才是正确的

005784A7                   .  33D2          XOR EDX,EDX

005784A9                   .  8BF8          MOV EDI,EAX                              ;  edi

005784AB                   .  8B41 34       MOV EAX,DWORD PTR DS:[ECX+0x34]

005784AE                   .  85C0          TEST EAX,EAX

005784B0                   .  0F94C2        SETE DL

005784B3                   .  895424 2C     MOV DWORD PTR SS:[ESP+0x2C],EDX

005784B7                   .  81FB E7000000 CMP EBX,0xE7                             ;  E7是老版本的

005784BD                   .  0F84 8F010000 JE 010Edito.00578652

005784C3                   .  8379 2C 00    CMP DWORD PTR DS:[ECX+0x2C],0x0

005784C7                   .  0F84 85010000 JE 010Edito.00578652

005784CD                   .  6A 00         PUSH 0x0

005784CF                   .  E8 22CCE8FF   CALL 010Edito.004050F6

005784D4                   .  85C0          TEST EAX,EAX

005784D6                   .  0F8D D6000000 JGE 010Edito.005785B2

005784DC                   .  6A 0A         PUSH 0xA

005784DE                   .  50            PUSH EAX

005784DF                   .  8D4424 28     LEA EAX,DWORD PTR SS:[ESP+0x28]

005784E3                   .  50            PUSH EAX

005784E4                   .  FF15 8C978B00 CALL DWORD PTR DS:[<&QtCore4.?number@QSt>;  QtCore4.?number@QString@@SA?AV1@JH@Z

005784EA                   .  8BF0          MOV ESI,EAX

005784EC                   .  6A FF         PUSH -0x1

005784EE                   .  68 18CB7400   PUSH 010Edito.0074CB18                   ;  could not contact the webserver. please check your internet connection. if your internet connection is currently working, the server may be down. if this problem persists, please visit 'dceK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4%4k6h3g2@1M7$3y4S2M7r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0r3i4K6t1%4 (

005784F3                   .  C64424 4C 0A  MOV BYTE PTR SS:[ESP+0x4C],0xA

005784F8                   .  FFD5          CALL EBP

005784FA                   .  894424 38     MOV DWORD PTR SS:[ESP+0x38],EAX

005784FE                   .  56            PUSH ESI

005784FF                   .  8D4C24 3C     LEA ECX,DWORD PTR SS:[ESP+0x3C]

00578503                   .  51            PUSH ECX

00578504                   .  8D5424 44     LEA EDX,DWORD PTR SS:[ESP+0x44]

00578508                   .  52            PUSH EDX

00578509                   .  C64424 58 0B  MOV BYTE PTR SS:[ESP+0x58],0xB

0057850E                   .  E8 AFCDE8FF   CALL 010Edito.004052C2

00578513                   .  68 DCD27300   PUSH 010Edito.0073D2DC                   ;  ).

00578518                   .  50            PUSH EAX

00578519                   .  8D4424 54     LEA EAX,DWORD PTR SS:[ESP+0x54]

0057851D                   .  B3 0C         MOV BL,0xC

0057851F                   .  50            PUSH EAX

00578520                   .  885C24 64     MOV BYTE PTR SS:[ESP+0x64],BL

00578524                   .  E8 F417E9FF   CALL 010Edito.00409D1D

00578529                   .  50            PUSH EAX

0057852A                   .  C64424 68 0D  MOV BYTE PTR SS:[ESP+0x68],0xD

0057852F                   .  E8 14E3E8FF   CALL 010Edito.00406848

00578534                   .  8B4C24 5C     MOV ECX,DWORD PTR SS:[ESP+0x5C]

00578538                   .  8B35 CC978B00 MOV ESI,DWORD PTR DS:[<&QtCore4.?free@QS>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

0057853E                   .  83C4 30       ADD ESP,0x30

00578541                   .  885C24 38     MOV BYTE PTR SS:[ESP+0x38],BL

00578545                   .  83CA FF       OR EDX,0xFFFFFFFF

00578548                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

0057854C                   .  75 0A         JNZ SHORT 010Edito.00578558

0057854E                   .  8B4424 2C     MOV EAX,DWORD PTR SS:[ESP+0x2C]

00578552                   .  50            PUSH EAX

00578553                   .  FFD6          CALL ESI

00578555                   .  83C4 04       ADD ESP,0x4

00578558                   >  8B4C24 28     MOV ECX,DWORD PTR SS:[ESP+0x28]

0057855C                   .  C64424 38 0B  MOV BYTE PTR SS:[ESP+0x38],0xB

00578561                   .  83CA FF       OR EDX,0xFFFFFFFF

00578564                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

00578568                   .  75 0A         JNZ SHORT 010Edito.00578574

0057856A                   .  8B4424 28     MOV EAX,DWORD PTR SS:[ESP+0x28]

0057856E                   .  50            PUSH EAX

0057856F                   .  FFD6          CALL ESI

00578571                   .  83C4 04       ADD ESP,0x4

00578574                   >  8B4C24 24     MOV ECX,DWORD PTR SS:[ESP+0x24]

00578578                   .  C64424 38 0A  MOV BYTE PTR SS:[ESP+0x38],0xA

0057857D                   .  83CA FF       OR EDX,0xFFFFFFFF

00578580                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

00578584                   .  75 0A         JNZ SHORT 010Edito.00578590

00578586                   .  8B4424 24     MOV EAX,DWORD PTR SS:[ESP+0x24]

0057858A                   .  50            PUSH EAX

0057858B                   .  FFD6          CALL ESI

0057858D                   .  83C4 04       ADD ESP,0x4

00578590                   >  8B4C24 20     MOV ECX,DWORD PTR SS:[ESP+0x20]

00578594                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

00578599                   .  83CA FF       OR EDX,0xFFFFFFFF

0057859C                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

005785A0                   .  0F85 51040000 JNZ 010Edito.005789F7

005785A6                   .  8B4424 20     MOV EAX,DWORD PTR SS:[ESP+0x20]

005785AA                   .  50            PUSH EAX

005785AB                   .  FFD6          CALL ESI

005785AD                   .  E9 42040000   JMP 010Edito.005789F4

005785B2                   >  0F85 80000000 JNZ 010Edito.00578638

005785B8                   .  6A FF         PUSH -0x1

005785BA                   .  68 10CA7400   PUSH 010Edito.0074CA10                   ;  010 editor has detected that you have entered an invalid license. please check to see if your license is entered correctly and try again. if this problem persists, please visit '3e7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4%4k6h3g2@1M7$3y4S2M7r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0r3i4K6t1%4i4K6u0W2

005785BF                   .  FFD5          CALL EBP

005785C1                   .  894424 2C     MOV DWORD PTR SS:[ESP+0x2C],EAX

005785C5                   .  8D4C24 2C     LEA ECX,DWORD PTR SS:[ESP+0x2C]

005785C9                   .  51            PUSH ECX

005785CA                   .  C64424 44 0E  MOV BYTE PTR SS:[ESP+0x44],0xE

005785CF                   .  E8 74E2E8FF   CALL 010Edito.00406848

005785D4                   .  8B5424 30     MOV EDX,DWORD PTR SS:[ESP+0x30]

005785D8                   .  8B35 CC978B00 MOV ESI,DWORD PTR DS:[<&QtCore4.?free@QS>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

005785DE                   .  83C4 0C       ADD ESP,0xC

005785E1                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

005785E6                   .  83C8 FF       OR EAX,0xFFFFFFFF

005785E9                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

005785ED                   .  75 0A         JNZ SHORT 010Edito.005785F9

005785EF                   .  8B4C24 24     MOV ECX,DWORD PTR SS:[ESP+0x24]

005785F3                   .  51            PUSH ECX

005785F4                   .  FFD6          CALL ESI

005785F6                   .  83C4 04       ADD ESP,0x4

005785F9                   >  8B5424 1C     MOV EDX,DWORD PTR SS:[ESP+0x1C]

005785FD                   .  C64424 38 00  MOV BYTE PTR SS:[ESP+0x38],0x0

00578602                   .  83C8 FF       OR EAX,0xFFFFFFFF

00578605                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

00578609                   .  75 0A         JNZ SHORT 010Edito.00578615

0057860B                   .  8B4C24 1C     MOV ECX,DWORD PTR SS:[ESP+0x1C]

0057860F                   .  51            PUSH ECX

00578610                   .  FFD6          CALL ESI

00578612                   .  83C4 04       ADD ESP,0x4

00578615                   >  8B5424 18     MOV EDX,DWORD PTR SS:[ESP+0x18]

00578619                   .  C74424 38 FFF>MOV DWORD PTR SS:[ESP+0x38],-0x1

00578621                   .  83C8 FF       OR EAX,0xFFFFFFFF

00578624                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

00578628                   .  0F85 00040000 JNZ 010Edito.00578A2E

0057862E                   .  8B4C24 18     MOV ECX,DWORD PTR SS:[ESP+0x18]

00578632                   .  51            PUSH ECX

00578633                   .  E9 F1030000   JMP 010Edito.00578A29

00578638                   >  8B0D CC4D8B00 MOV ECX,DWORD PTR DS:[0x8B4DCC]

0057863E                   .  68 23400000   PUSH 0x4023

00578643                   .  6A 07         PUSH 0x7

00578645                   .  E8 2F06E9FF   CALL 010Edito.00408C79

0057864A                   .  8B0D CC4D8B00 MOV ECX,DWORD PTR DS:[0x8B4DCC]

00578650                   .  8BF8          MOV EDI,EAX

00578652                   >  81FF DB000000 CMP EDI,0xDB                             ;  关键就是edi的值了

00578658                   .  75 38         JNZ SHORT 010Edito.00578692

0057865A                   .  6A FF         PUSH -0x1

0057865C                   .  68 CCC97400   PUSH 010Edito.0074C9CC                   ;  password accepted. thank you for purchasing 010 editor!

00578661                   .  FFD5          CALL EBP

00578663                   .  894424 2C     MOV DWORD PTR SS:[ESP+0x2C],EAX

00578667                   .  8D5424 2C     LEA EDX,DWORD PTR SS:[ESP+0x2C]

0057866B                   .  52            PUSH EDX

0057866C                   .  C64424 44 0F  MOV BYTE PTR SS:[ESP+0x44],0xF

00578671                   .  E8 E696E8FF   CALL 010Edito.00401D5C

00578676                   .  8B4424 30     MOV EAX,DWORD PTR SS:[ESP+0x30]

0057867A                   .  83C4 0C       ADD ESP,0xC

0057867D                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

00578682                   .  83C9 FF       OR ECX,0xFFFFFFFF

00578685                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

00578689                   .  75 73         JNZ SHORT 010Edito.005786FE

0057868B                   .  8B5424 24     MOV EDX,DWORD PTR SS:[ESP+0x24]

0057868F                   .  52            PUSH EDX

00578690                   .  EB 63         JMP SHORT 010Edito.005786F5

00578692                   >  81FF ED000000 CMP EDI,0xED

00578698                   .  0F84 44020000 JE 010Edito.005788E2

0057869E                   .  81FF 0C020000 CMP EDI,0x20C

005786A4                   .  0F84 38020000 JE 010Edito.005788E2

005786AA                   .  6A FF         PUSH -0x1

005786AC                   .  81FB 93000000 CMP EBX,0x93

005786B2                   .  0F85 AC010000 JNZ 010Edito.00578864                    ;  跳到错误

005786B8                   .  83FF 71       CMP EDI,0x71

005786BB                   .  0F85 25010000 JNZ 010Edito.005787E6

005786C1                   .  68 88C97400   PUSH 010Edito.0074C988                   ;  password accepted. your trial period has been extended.

005786C6                   .  FFD5          CALL EBP

005786C8                   .  894424 2C     MOV DWORD PTR SS:[ESP+0x2C],EAX

005786CC                   .  8D4424 2C     LEA EAX,DWORD PTR SS:[ESP+0x2C]

005786D0                   .  50            PUSH EAX

005786D1                   .  C64424 44 13  MOV BYTE PTR SS:[ESP+0x44],0x13

005786D6                   .  E8 8196E8FF   CALL 010Edito.00401D5C

005786DB                   .  8B4C24 30     MOV ECX,DWORD PTR SS:[ESP+0x30]

005786DF                   .  83C4 0C       ADD ESP,0xC

005786E2                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

005786E7                   .  83CA FF       OR EDX,0xFFFFFFFF

005786EA                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

005786EE                   .  75 0E         JNZ SHORT 010Edito.005786FE

005786F0                   .  8B4424 24     MOV EAX,DWORD PTR SS:[ESP+0x24]

005786F4                   .  50            PUSH EAX

005786F5                   >  FF15 CC978B00 CALL DWORD PTR DS:[<&QtCore4.?free@QStri>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

005786FB                   .  83C4 04       ADD ESP,0x4

005786FE                   >  8B0D CC4D8B00 MOV ECX,DWORD PTR DS:[0x8B4DCC]

00578704                   .  E8 C0B5E8FF   CALL 010Edito.00403CC9

00578709                   .  84C0          TEST AL,AL

0057870B                   .  0F85 80000000 JNZ 010Edito.00578791

00578711                   .  6A FF         PUSH -0x1

00578713                   .  68 10C97400   PUSH 010Edito.0074C910                   ;  could not change password. you may need to be logged in as an administrator to change the password.

00578718                   .  FFD5          CALL EBP

0057871A                   .  894424 2C     MOV DWORD PTR SS:[ESP+0x2C],EAX

0057871E                   .  8D4C24 2C     LEA ECX,DWORD PTR SS:[ESP+0x2C]

00578722                   .  51            PUSH ECX

00578723                   .  C64424 44 16  MOV BYTE PTR SS:[ESP+0x44],0x16

00578728                   .  E8 EA8FE8FF   CALL 010Edito.00401717

0057872D                   .  8B5424 30     MOV EDX,DWORD PTR SS:[ESP+0x30]

00578731                   .  8B35 CC978B00 MOV ESI,DWORD PTR DS:[<&QtCore4.?free@QS>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

00578737                   .  83C4 0C       ADD ESP,0xC

0057873A                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

0057873F                   .  83C8 FF       OR EAX,0xFFFFFFFF

00578742                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

00578746                   .  75 0A         JNZ SHORT 010Edito.00578752

00578748                   .  8B4C24 24     MOV ECX,DWORD PTR SS:[ESP+0x24]

0057874C                   .  51            PUSH ECX

0057874D                   .  FFD6          CALL ESI

0057874F                   .  83C4 04       ADD ESP,0x4

00578752                   >  8B5424 1C     MOV EDX,DWORD PTR SS:[ESP+0x1C]

00578756                   .  C64424 38 00  MOV BYTE PTR SS:[ESP+0x38],0x0

0057875B                   .  83C8 FF       OR EAX,0xFFFFFFFF

0057875E                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

00578762                   .  75 0A         JNZ SHORT 010Edito.0057876E

00578764                   .  8B4C24 1C     MOV ECX,DWORD PTR SS:[ESP+0x1C]

00578768                   .  51            PUSH ECX

00578769                   .  FFD6          CALL ESI

0057876B                   .  83C4 04       ADD ESP,0x4

0057876E                   >  8B5424 18     MOV EDX,DWORD PTR SS:[ESP+0x18]

00578772                   .  C74424 38 FFF>MOV DWORD PTR SS:[ESP+0x38],-0x1

0057877A                   .  83C8 FF       OR EAX,0xFFFFFFFF

0057877D                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

00578781                   .  0F85 A7020000 JNZ 010Edito.00578A2E

00578787                   .  8B4C24 18     MOV ECX,DWORD PTR SS:[ESP+0x18]

0057878B                   .  51            PUSH ECX

0057878C                   .  E9 98020000   JMP 010Edito.00578A29

00578791                   >  8B16          MOV EDX,DWORD PTR DS:[ESI]

00578793                   .  8B82 CC000000 MOV EAX,DWORD PTR DS:[EDX+0xCC]

00578799                   .  8BCE          MOV ECX,ESI

0057879B                   .  FFD0          CALL EAX

0057879D                   .  8B4C24 1C     MOV ECX,DWORD PTR SS:[ESP+0x1C]

005787A1                   .  C64424 38 00  MOV BYTE PTR SS:[ESP+0x38],0x0

005787A6                   .  83CA FF       OR EDX,0xFFFFFFFF

005787A9                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

005787AD                   .  75 0E         JNZ SHORT 010Edito.005787BD

005787AF                   .  8B4424 1C     MOV EAX,DWORD PTR SS:[ESP+0x1C]

005787B3                   .  50            PUSH EAX

005787B4                   .  FF15 CC978B00 CALL DWORD PTR DS:[<&QtCore4.?free@QStri>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

005787BA                   .  83C4 04       ADD ESP,0x4

005787BD                   >  8B4C24 18     MOV ECX,DWORD PTR SS:[ESP+0x18]

005787C1                   .  C74424 38 FFF>MOV DWORD PTR SS:[ESP+0x38],-0x1

005787C9                   .  83CA FF       OR EDX,0xFFFFFFFF

005787CC                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

005787D0                   .  0F85 58020000 JNZ 010Edito.00578A2E

005787D6                   .  8B4424 18     MOV EAX,DWORD PTR SS:[ESP+0x18]

005787DA                   .  50            PUSH EAX

005787DB                   .  FF15 CC978B00 CALL DWORD PTR DS:[<&QtCore4.?free@QStri>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

005787E1                   .  E9 45020000   JMP 010Edito.00578A2B

005787E6                   >  68 C8C87400   PUSH 010Edito.0074C8C8                   ;  password accepted but the trial period is already over.

005787EB                   .  FFD5          CALL EBP

005787ED                   .  894424 2C     MOV DWORD PTR SS:[ESP+0x2C],EAX

005787F1                   .  8D4C24 2C     LEA ECX,DWORD PTR SS:[ESP+0x2C]

005787F5                   .  51            PUSH ECX

005787F6                   .  C64424 44 14  MOV BYTE PTR SS:[ESP+0x44],0x14

005787FB                   .  E8 5C95E8FF   CALL 010Edito.00401D5C

00578800                   .  8B5424 30     MOV EDX,DWORD PTR SS:[ESP+0x30]

00578804                   .  8B35 CC978B00 MOV ESI,DWORD PTR DS:[<&QtCore4.?free@QS>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

0057880A                   .  83C4 0C       ADD ESP,0xC

0057880D                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

00578812                   .  83C8 FF       OR EAX,0xFFFFFFFF

00578815                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

00578819                   .  75 0A         JNZ SHORT 010Edito.00578825

0057881B                   .  8B4C24 24     MOV ECX,DWORD PTR SS:[ESP+0x24]

0057881F                   .  51            PUSH ECX

00578820                   .  FFD6          CALL ESI

00578822                   .  83C4 04       ADD ESP,0x4

00578825                   >  8B5424 1C     MOV EDX,DWORD PTR SS:[ESP+0x1C]

00578829                   .  C64424 38 00  MOV BYTE PTR SS:[ESP+0x38],0x0

0057882E                   .  83C8 FF       OR EAX,0xFFFFFFFF

00578831                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

00578835                   .  75 0A         JNZ SHORT 010Edito.00578841

00578837                   .  8B4C24 1C     MOV ECX,DWORD PTR SS:[ESP+0x1C]

0057883B                   .  51            PUSH ECX

0057883C                   .  FFD6          CALL ESI

0057883E                   .  83C4 04       ADD ESP,0x4

00578841                   >  8B5424 18     MOV EDX,DWORD PTR SS:[ESP+0x18]

00578845                   .  C74424 38 FFF>MOV DWORD PTR SS:[ESP+0x38],-0x1

0057884D                   .  83C8 FF       OR EAX,0xFFFFFFFF

00578850                   .  F0:0FC102     LOCK XADD DWORD PTR DS:[EDX],EAX

00578854                   .  0F85 D4010000 JNZ 010Edito.00578A2E

0057885A                   .  8B4C24 18     MOV ECX,DWORD PTR SS:[ESP+0x18]

0057885E                   .  51            PUSH ECX

0057885F                   .  E9 C5010000   JMP 010Edito.00578A29

00578864                   >  68 18C87400   PUSH 010Edito.0074C818                   ;  invalid name or password. please enter your name and password exactly as given when you purchased 010 editor (make sure no quotes are included).

00578869                   .  FFD5          CALL EBP

0057886B                   .  894424 2C     MOV DWORD PTR SS:[ESP+0x2C],EAX

0057886F                   .  8D5424 2C     LEA EDX,DWORD PTR SS:[ESP+0x2C]

00578873                   .  52            PUSH EDX

00578874                   .  C64424 44 15  MOV BYTE PTR SS:[ESP+0x44],0x15

00578879                   .  E8 DE94E8FF   CALL 010Edito.00401D5C

0057887E                   .  8B4424 30     MOV EAX,DWORD PTR SS:[ESP+0x30]

00578882                   .  8B35 CC978B00 MOV ESI,DWORD PTR DS:[<&QtCore4.?free@QS>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

00578888                   .  83C4 0C       ADD ESP,0xC

0057888B                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

00578890                   .  83C9 FF       OR ECX,0xFFFFFFFF

00578893                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

00578897                   .  75 0A         JNZ SHORT 010Edito.005788A3

00578899                   .  8B5424 24     MOV EDX,DWORD PTR SS:[ESP+0x24]

0057889D                   .  52            PUSH EDX

0057889E                   .  FFD6          CALL ESI

005788A0                   .  83C4 04       ADD ESP,0x4

005788A3                   >  8B4424 1C     MOV EAX,DWORD PTR SS:[ESP+0x1C]

005788A7                   .  C64424 38 00  MOV BYTE PTR SS:[ESP+0x38],0x0

005788AC                   .  83C9 FF       OR ECX,0xFFFFFFFF

005788AF                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

005788B3                   .  75 0A         JNZ SHORT 010Edito.005788BF

005788B5                   .  8B5424 1C     MOV EDX,DWORD PTR SS:[ESP+0x1C]

005788B9                   .  52            PUSH EDX

005788BA                   .  FFD6          CALL ESI

005788BC                   .  83C4 04       ADD ESP,0x4

005788BF                   >  8B4424 18     MOV EAX,DWORD PTR SS:[ESP+0x18]

005788C3                   .  C74424 38 FFF>MOV DWORD PTR SS:[ESP+0x38],-0x1

005788CB                   .  83C9 FF       OR ECX,0xFFFFFFFF

005788CE                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

005788D2                   .  0F85 56010000 JNZ 010Edito.00578A2E

005788D8                   .  8B5424 18     MOV EDX,DWORD PTR SS:[ESP+0x18]

005788DC                   .  52            PUSH EDX

005788DD                   .  E9 47010000   JMP 010Edito.00578A29

005788E2                   >  E8 E2B3E8FF   CALL 010Edito.00403CC9

005788E7                   .  6A FF         PUSH -0x1

005788E9                   .  8BCE          MOV ECX,ESI

005788EB                   .  E8 7AC7E8FF   CALL 010Edito.0040506A

005788F0                   .  68 C0C77400   PUSH 010Edito.0074C7C0                   ;  The password you entered is for an earlier version of this program.

005788F5                   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+0x1C]

005788F9                   .  FF15 3C978B00 CALL DWORD PTR DS:[<&QtCore4.??4QString@>;  QtCore4.??4QString@@QAEAAV0@PBD@Z

005788FF                   .  837C24 2C 00  CMP DWORD PTR SS:[ESP+0x2C],0x0

00578904                   .  8D4C24 18     LEA ECX,DWORD PTR SS:[ESP+0x18]

00578908                   .  74 07         JE SHORT 010Edito.00578911

0057890A                   .  68 B8C67400   PUSH 010Edito.0074C6B8                   ;  tNotchesVisible@QDial@@QAEX_N@Z

0057890F                   .  EB 05         JMP SHORT 010Edito.00578916

00578911                   >  68 60C67400   PUSH 010Edito.0074C660                   ;  You will have to purchase an upgrade to continue using this software.

00578916                   >  FF15 88978B00 CALL DWORD PTR DS:[<&QtCore4.??YQString@>;  QtCore4.?append@QString@@QAEAAV1@PBD@Z

0057891C                   .  81FF 0C020000 CMP EDI,0x20C

00578922                   .  75 0F         JNZ SHORT 010Edito.00578933

00578924                   .  68 14C67400   PUSH 010Edito.0074C614                   ;  You may continue to use this program as a trial version.

00578929                   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+0x1C]

0057892D                   .  FF15 88978B00 CALL DWORD PTR DS:[<&QtCore4.??YQString@>;  QtCore4.?append@QString@@QAEAAV1@PBD@Z

00578933                   >  68 A0C57400   PUSH 010Edito.0074C5A0                   ;  If you decide to purchase an upgrade, please click the "Purchase an upgrade" link below or you

00578938                   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+0x1C]

0057893C                   .  FF15 88978B00 CALL DWORD PTR DS:[<&QtCore4.??YQString@>;  QtCore4.?append@QString@@QAEAAV1@PBD@Z

00578942                   .  6A FF         PUSH -0x1

00578944                   .  68 20BA7400   PUSH 010Edito.0074BA20                   ;  d41K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4%4k6h3g2@1M7$3y4S2M7r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4m8J5k6i4k6A6L8%4g2K6i4K6u0r3

00578949                   .  FFD5          CALL EBP

0057894B                   .  894424 2C     MOV DWORD PTR SS:[ESP+0x2C],EAX

0057894F                   .  8D4424 2C     LEA EAX,DWORD PTR SS:[ESP+0x2C]

00578953                   .  50            PUSH EAX

00578954                   .  8D4C24 34     LEA ECX,DWORD PTR SS:[ESP+0x34]

00578958                   .  68 50C57400   PUSH 010Edito.0074C550                   ;  may download previous versions of 010 Editor from the website "

0057895D                   .  51            PUSH ECX

0057895E                   .  C64424 4C 10  MOV BYTE PTR SS:[ESP+0x4C],0x10

00578963                   .  E8 268FE8FF   CALL 010Edito.0040188E

00578968                   .  68 CCB97400   PUSH 010Edito.0074B9CC                   ;  ". Thank you!

0057896D                   .  50            PUSH EAX

0057896E                   .  8D5424 48     LEA EDX,DWORD PTR SS:[ESP+0x48]

00578972                   .  B3 11         MOV BL,0x11

00578974                   .  52            PUSH EDX

00578975                   .  885C24 58     MOV BYTE PTR SS:[ESP+0x58],BL

00578979                   .  E8 9F13E9FF   CALL 010Edito.00409D1D

0057897E                   .  83C4 20       ADD ESP,0x20

00578981                   .  50            PUSH EAX

00578982                   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+0x1C]

00578986                   .  C64424 3C 12  MOV BYTE PTR SS:[ESP+0x3C],0x12

0057898B                   .  FF15 D8978B00 CALL DWORD PTR DS:[<&QtCore4.?append@QSt>;  QtCore4.?append@QString@@QAEAAV1@ABV1@@Z

00578991                   .  8B4424 2C     MOV EAX,DWORD PTR SS:[ESP+0x2C]

00578995                   .  8B35 CC978B00 MOV ESI,DWORD PTR DS:[<&QtCore4.?free@QS>;  QtCore4.?free@QString@@CAXPAUData@1@@Z

0057899B                   .  885C24 38     MOV BYTE PTR SS:[ESP+0x38],BL

0057899F                   .  83C9 FF       OR ECX,0xFFFFFFFF

005789A2                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

005789A6                   .  75 0A         JNZ SHORT 010Edito.005789B2

005789A8                   .  8B5424 2C     MOV EDX,DWORD PTR SS:[ESP+0x2C]

005789AC                   .  52            PUSH EDX

005789AD                   .  FFD6          CALL ESI

005789AF                   .  83C4 04       ADD ESP,0x4

005789B2                   >  8B4424 28     MOV EAX,DWORD PTR SS:[ESP+0x28]

005789B6                   .  C64424 38 10  MOV BYTE PTR SS:[ESP+0x38],0x10

005789BB                   .  83C9 FF       OR ECX,0xFFFFFFFF

005789BE                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

005789C2                   .  75 0A         JNZ SHORT 010Edito.005789CE

005789C4                   .  8B5424 28     MOV EDX,DWORD PTR SS:[ESP+0x28]

005789C8                   .  52            PUSH EDX

005789C9                   .  FFD6          CALL ESI

005789CB                   .  83C4 04       ADD ESP,0x4

005789CE                   >  8B4424 24     MOV EAX,DWORD PTR SS:[ESP+0x24]

005789D2                   .  C64424 38 01  MOV BYTE PTR SS:[ESP+0x38],0x1

005789D7                   .  83C9 FF       OR ECX,0xFFFFFFFF

005789DA                   .  F0:0FC108     LOCK XADD DWORD PTR DS:[EAX],ECX

005789DE                   .  75 0A         JNZ SHORT 010Edito.005789EA

005789E0                   .  8B5424 24     MOV EDX,DWORD PTR SS:[ESP+0x24]

005789E4                   .  52            PUSH EDX

005789E5                   .  FFD6          CALL ESI

005789E7                   .  83C4 04       ADD ESP,0x4

005789EA                   >  8D4424 18     LEA EAX,DWORD PTR SS:[ESP+0x18]

005789EE                   .  50            PUSH EAX

005789EF                   .  E8 6893E8FF   CALL 010Edito.00401D5C

005789F4                   >  83C4 04       ADD ESP,0x4

005789F7                   >  8B4C24 1C     MOV ECX,DWORD PTR SS:[ESP+0x1C]

005789FB                   .  83CA FF       OR EDX,0xFFFFFFFF

005789FE                   .  C64424 38 00  MOV BYTE PTR SS:[ESP+0x38],0x0

00578A03                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

00578A07                   .  75 0A         JNZ SHORT 010Edito.00578A13

00578A09                   .  8B4424 1C     MOV EAX,DWORD PTR SS:[ESP+0x1C]

00578A0D                   .  50            PUSH EAX

00578A0E                   .  FFD6          CALL ESI

00578A10                   .  83C4 04       ADD ESP,0x4

00578A13                   >  8B4C24 18     MOV ECX,DWORD PTR SS:[ESP+0x18]

00578A17                   .  83CA FF       OR EDX,0xFFFFFFFF

00578A1A                   .  895424 38     MOV DWORD PTR SS:[ESP+0x38],EDX

00578A1E                   .  F0:0FC111     LOCK XADD DWORD PTR DS:[ECX],EDX

00578A22                   .  75 0A         JNZ SHORT 010Edito.00578A2E

00578A24                   .  8B4424 18     MOV EAX,DWORD PTR SS:[ESP+0x18]

00578A28                   .  50            PUSH EAX

00578A29                   >  FFD6          CALL ESI

00578A2B                   >  83C4 04       ADD ESP,0x4

00578A2E                   >  8B4C24 30     MOV ECX,DWORD PTR SS:[ESP+0x30]

00578A32                   .  64:890D 00000>MOV DWORD PTR FS:[0],ECX

00578A39                   .  59            POP ECX

00578A3A                   .  5F            POP EDI

00578A3B                   .  5E            POP ESI

00578A3C                   .  5D            POP EBP

00578A3D                   .  5B            POP EBX

00578A3E                   .  83C4 28       ADD ESP,0x28

00578A41                   .  C3            RETN

```
