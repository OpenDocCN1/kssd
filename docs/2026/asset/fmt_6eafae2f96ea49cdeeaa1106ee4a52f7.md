
```
004141BF    . 50                         PUSH EAX                                                  ; /pDisposition

004141C0    . 8B1D 00104200              MOV EBX,DWORD PTR DS:[<&ADVAPI32.RegCreateKeyExA>]        ; |ADVAPI32.RegCreateKeyExA           建立注册表键值

004141C6    . 51                         PUSH ECX                                                  ; |pHandle

004141C7    . 6A 00                      PUSH 0                                                    ; |pSecurity = NULL

004141C9    . 68 06000200                PUSH 20006                                                ; |Access = KEY\_WRITE

004141CE    . 6A 00                      PUSH 0                                                    ; |Options = REG\_OPTION\_NON\_VOLATILE

004141D0    . 68 C8A14200                PUSH hostscan.0042A1C8                                    ; |Class = ""

004141D5    . 6A 00                      PUSH 0                                                    ; |Reserved = 0

004141D7    . 68 B06B4200                PUSH hostscan.00426BB0                                    ; |Subkey = "SoftwareNetSeek\"        注册表键值

004141DC    . 68 01000080                PUSH 80000001                                             ; |hKey = HKEY\_CURRENT\_USER

004141E1    . FFD3                       CALL EBX                                                  ; RegCreateKeyExA

004141E3    . 85C0                       TEST EAX,EAX

004141E5    . 74 0C                      JE SHORT hostscan.004141F3

004141E7    . 5F                         POP EDI

004141E8    . 5E                         POP ESI

004141E9    . 5D                         POP EBP

004141EA    . 33C0                       XOR EAX,EAX

004141EC    . 5B                         POP EBX

004141ED    . 83C4 64                    ADD ESP,64

004141F0    . C2 1000                    RETN 10

004141F3    > 8B35 BC104200              MOV ESI,DWORD PTR DS:[<&KERNEL32.lstrlenA>]               ;  KERNEL32.lstrlenA

004141F9    . 68 AC9D4300                PUSH hostscan.00439DAC                                    ; /String = "coldeye"

004141FE    . FFD6                       CALL ESI                                                  ; lstrlenA

00414200    . 40                         INC EAX

00414201    . 8B5424 10                  MOV EDX,DWORD PTR SS:[ESP+10]

00414205    . 50                         PUSH EAX                                                  ; /BufSize

00414206    . 68 AC9D4300                PUSH hostscan.00439DAC                                    ; |Buffer = hostscan.00439DAC

0041420B    . 6A 01                      PUSH 1                                                    ; |ValueType = REG\_SZ

0041420D    . 6A 00                      PUSH 0                                                    ; |Reserved = 0

0041420F    . 68 8C8A4200                PUSH hostscan.00428A8C                                    ; |ValueName = "Username"

00414214    . 52                         PUSH EDX                                                  ; |hKey

00414215    . FF15 14104200              CALL DWORD PTR DS:[<&ADVAPI32.RegSetValueExA>]            ; RegSetValueExA

0041421B    . 85C0                       TEST EAX,EAX

0041421D    . 74 0C                      JE SHORT hostscan.0041422B

0041421F    . 5F                         POP EDI

00414220    . 5E                         POP ESI

00414221    . 5D                         POP EBP

00414222    . 33C0                       XOR EAX,EAX

00414224    . 5B                         POP EBX

00414225    . 83C4 64                    ADD ESP,64

00414228    . C2 1000                    RETN 10

0041422B    > 8B4424 10                  MOV EAX,DWORD PTR SS:[ESP+10]

0041422F    . 8B3D 1C104200              MOV EDI,DWORD PTR DS:[<&ADVAPI32.RegCloseKey>]            ;  ADVAPI32.RegCloseKey

00414235    . 50                         PUSH EAX                                                  ; /hKey

00414236    . FFD7                       CALL EDI                                                  ; RegCloseKey

00414238    . 8D4C24 14                  LEA ECX,DWORD PTR SS:[ESP+14]

0041423C    . 8D5424 10                  LEA EDX,DWORD PTR SS:[ESP+10]

00414240    . 51                         PUSH ECX

00414241    . 52                         PUSH EDX

00414242    . 6A 00                      PUSH 0

00414244    . 68 06000200                PUSH 20006

00414249    . 6A 00                      PUSH 0

0041424B    . 68 C8A14200                PUSH hostscan.0042A1C8

00414250    . 6A 00                      PUSH 0

00414252    . 68 748A4200                PUSH hostscan.00428A74                                    ;  ASCII "hostscan.fileshellEx\"      注册表键值

00414257    . 68 00000080                PUSH 80000000

0041425C    . FFD3                       CALL EBX

0041425E    . 85C0                       TEST EAX,EAX

00414260    . 74 0C                      JE SHORT hostscan.0041426E

00414262    . 5F                         POP EDI

00414263    . 5E                         POP ESI

00414264    . 5D                         POP EBP

00414265    . 33C0                       XOR EAX,EAX

00414267    . 5B                         POP EBX

00414268    . 83C4 64                    ADD ESP,64

0041426B    . C2 1000                    RETN 10

0041426E    > 8D4424 18                  LEA EAX,DWORD PTR SS:[ESP+18]

00414272    . 33DB                       XOR EBX,EBX

00414274    . 50                         PUSH EAX

00414275    . FFD6                       CALL ESI

00414277    . 85C0                       TEST EAX,EAX

00414279    . 7E 16                      JLE SHORT hostscan.00414291

0041427B    > 8A4C1C 18                  MOV CL,BYTE PTR SS:[ESP+EBX+18]

0041427F    . 02CB                       ADD CL,BL

00414281    . 884C1C 18                  MOV BYTE PTR SS:[ESP+EBX+18],CL

00414285    . 8D4C24 18                  LEA ECX,DWORD PTR SS:[ESP+18]

00414289    . 51                         PUSH ECX

0041428A    . 43                         INC EBX

0041428B    . FFD6                       CALL ESI

0041428D    . 3BD8                       CMP EBX,EAX

0041428F    .^7C EA                      JL SHORT hostscan.0041427B

00414291    > 8D5424 18                  LEA EDX,DWORD PTR SS:[ESP+18]

00414295    . 52                         PUSH EDX

00414296    . FFD6                       CALL ESI

00414298    . 40                         INC EAX

00414299    . 8B4C24 10                  MOV ECX,DWORD PTR SS:[ESP+10]

0041429D    . 50                         PUSH EAX                                                  ; /BufSize

0041429E    . 8D4424 1C                  LEA EAX,DWORD PTR SS:[ESP+1C]                             ; |

004142A2    . 50                         PUSH EAX                                                  ; |Buffer

004142A3    . 6A 01                      PUSH 1                                                    ; |ValueType = REG\_SZ

004142A5    . 6A 00                      PUSH 0                                                    ; |Reserved = 0

004142A7    . 68 688A4200                PUSH hostscan.00428A68                                    ; |ValueName = "DropHandler"

004142AC    . 51                         PUSH ECX                                                  ; |hKey

004142AD    . FF15 14104200              CALL DWORD PTR DS:[<&ADVAPI32.RegSetValueExA>]            ; RegSetValueExA

004142B3    . 85C0                       TEST EAX,EAX

004142B5    . 74 0C                      JE SHORT hostscan.004142C3

004142B7    . 5F                         POP EDI

004142B8    . 5E                         POP ESI

004142B9    . 5D                         POP EBP

004142BA    . 33C0                       XOR EAX,EAX

004142BC    . 5B                         POP EBX

004142BD    . 83C4 64                    ADD ESP,64

004142C0    . C2 1000                    RETN 10

004142C3    > 8B5424 10                  MOV EDX,DWORD PTR SS:[ESP+10]
```
