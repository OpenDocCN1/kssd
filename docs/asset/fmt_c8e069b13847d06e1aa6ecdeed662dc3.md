
```assembly
001B:004C8324   FF7510             PUSH     DWORD PTR [EBP+10]
001B:004C8327   FF750C             PUSH     DWORD PTR [EBP+0C]
001B:004C832A   57                 PUSH     EDI
001B:004C832B   FF151C084E00       CALL     [WININET!InternetConnectA] //D-2
001B:004C8331   85C0               TEST     EAX,EAX
001B:004C8333   894604             MOV      [ESI+04],EAX
001B:004C8336   750B               JNZ      004C8343
001B:004C8338   50                 PUSH     EAX
001B:004C8339   FF7608             PUSH     DWORD PTR [ESI+08]
001B:004C833C   E8E6020000         CALL     004C8627  //联机错误信息处理
001B:004C8341   EB0E               JMP      004C8351
001B:004C8343   FF760C             PUSH     DWORD PTR [ESI+0C]
001B:004C8346   B930BF5100         MOV      ECX,0051BF30
001B:004C834B   50                 PUSH     EAX
001B:004C834C   E882F4FFFF         CALL     004C77D3
001B:004C8351   8B4DF4             MOV      ECX,[EBP-0C]
001B:004C8354   8BC6               MOV      EAX,ESI
001B:004C8356   5F                 POP      EDI
001B:004C8357   5E                 POP      ESI
001B:004C8358   64890D00000000     MOV      FS:[00000000],ECX
001B:004C835F   C9                 LEAVE
001B:004C8360   C21800             RET      0018
001B:004C8363   E981FDFFFF         JMP      004C80E9
-----------------------------------------------------------
```

### 第四步 创建一个到 HTTP 的请求信息的句柄

```assembly
CALL     [WININET!HttpOpenRequestA]  //D-4

001B:004C8368  B8017E4D00         MOV      EAX,004D7E01
001B:004C836D  E8D6F7F6FF         CALL     00437B48
001B:004C8372  53                 PUSH     EBX
001B:004C8373  8B5D14             MOV      EBX,[EBP+14]
001B:004C8376  56                 PUSH     ESI
001B:004C8377  83FB01             CMP      EBX,01
001B:004C837A  57                 PUSH     EDI
001B:004C837B  8BF1               MOV      ESI,ECX
001B:004C837D  7503               JNZ      004C8382
001B:004C837F  8B5E08             MOV      EBX,[ESI+08]
001B:004C8382  837D1C00           CMP      DWORD PTR [EBP+1C],00
001B:004C8386  7507               JNZ      004C838F
001B:004C8388  C7451CD0714E00     MOV      DWORD PTR [EBP+1C],004E71D0
001B:004C838F  53                 PUSH     EBX
001B:004C8390  FF7520             PUSH     DWORD PTR [EBP+20]
001B:004C8393  FF7518             PUSH     DWORD PTR [EBP+18]
001B:004C8396  FF7510             PUSH     DWORD PTR [EBP+10]
001B:004C8399  FF751C             PUSH     DWORD PTR [EBP+1C]
001B:004C839C  FF750C             PUSH     DWORD PTR [EBP+0C]
001B:004C839F  FF7508             PUSH     DWORD PTR [EBP+08]
001B:004C83A2  FF7604             PUSH     DWORD PTR [ESI+04]
001B:004C83A5  FF1524084E00       CALL     [WININET!HttpOpenRequestA] //D-3
001B:004C83AB  6A4C               PUSH     4C
001B:004C83AD  8BF8               MOV      EDI,EAX
001B:004C83AF  E89B86FEFF         CALL     004B0A4F
001B:004C83B4  59                 POP      ECX
001B:004C83B5  8BC8               MOV      ECX,EAX
001B:004C83B7  894D1C             MOV      [EBP+1C],ECX
001B:004C83BA  33C0               XOR      EAX,EAX
001B:004C83BC  3BC8               CMP      ECX,EAX
001B:004C83BE  8945FC             MOV      [EBP-04],EAX
001B:004C83C1  740D               JZ       004C83D0
001B:004C83C3  56                 PUSH     ESI
001B:004C83C4  FF750C             PUSH     DWORD PTR [EBP+0C]
001B:004C83C7  FF7508             PUSH     DWORD PTR [EBP+08]
001B:004C83CA  57                 PUSH     EDI
001B:004C83CB  E869000000         CALL     004C8439
001B:004C83D0  8B4DF4             MOV      ECX,[EBP-0C]
001B:004C83D3  5F                 POP      EDI
001B:004C83D4  89581C             MOV      [EAX+1C],EBX
001B:004C83D7  5E                 POP      ESI
001B:004C83D8  5B                 POP      EBX
001B:004C83D9  64890D00000000     MOV      FS:[00000000],ECX
001B:004C83E0  C9                 LEAVE
001B:004C83E1  C21C00             RET      001C
-------------------------------------------------
```

### 第五步 向 HTTP 发送信息

```assembly
CALL     [WININET!HttpSendRequestA]     //D-5

* Referenced by a CALL at Addresses:
|:0040713A   , :00424119    
|
```

```assembly
:00424BC0 55                    push ebp
:00424BC1 8BEC                  mov ebp, esp

* Possible Reference to Dialog: DialogID_00A7, CONTROL_ID:00FF, ""
                                |
:00424BC3 6AFF                  push FFFFFFFF
:00424BC5 6842644D00            push 004D6442
:00424BCA 64A100000000          mov eax, dword ptr fs:[00000000]
:00424BD0 50                    push eax
:00424BD1 64892500000000        mov dword ptr fs:[00000000], esp
:00424BD8 51                    push ecx
:00424BD9 B844520000            mov eax, 00005244
:00424BDE E80D2C0100            call 004377F0
:00424BE3 53                    push ebx
:00424BE4 56                    push esi
:00424BE5 57                    push edi
:00424BE6 8965F0                mov dword ptr [ebp-10], esp
:00424BE9 8B4510                mov eax, dword ptr [ebp+10]
:00424BEC 8B5D20                mov ebx, dword ptr [ebp+20]
:00424BEF 85DB                  test ebx, ebx
:00424BF1 C745FC00000000        mov [ebp-04], 00000000
:00424BF8 C70000000000          mov dword ptr [eax], 00000000
:00424BFE 0F85AF000000          jne 00424CB3
:00424C04 A12CA05100            mov eax, dword ptr [0051A02C]
:00424C09 85C0                  test eax, eax
:00424C0B 0F859C000000          jne 00424CAD
:00424C11 6A18                  push 00000018
:00424C13 E837BE0800            call 004B0A4F
:00424C18 8BF0                  mov esi, eax
:00424C1A 83C404                add esp, 00000004
:00424C1D 8975DC                mov dword ptr [ebp-24], esi
:00424C20 85F6                  test esi, esi
:00424C22 C645FC01              mov [ebp-04], 01
:00424C26 7424                  je 00424C4C
:00424C28 8B3D24A05100          mov edi, dword ptr [0051A024]
:00424C2E 53                    push ebx
:00424C2F 53                    push ebx
:00424C30 53                    push ebx
:00424C31 53                    push ebx

* Reference To: version.SV‹ñW‰t$Ç, Ord:0000h
                                |
:00424C32 FF153C084E00           Call dword ptr [004E083C]
:00424C38 33D2                  xor edx, edx
:00424C3A B9E8030000             mov ecx, 000003E8
:00424C3F F7F1                  div ecx
:00424C41 8BCE                  mov ecx, esi
:00424C43 52                    push edx
:00424C44 57                    push edi
:00424C45 E88620FFFF            call 00416CD0
:00424C4A EB02                  jmp 00424C4E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424C26(C)
|
:00424C4C 33C0                  xor eax, eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424C4A(U)
|
:00424C4E 8B15E4005100          mov edx, dword ptr [005100E4]
:00424C54 6A00                  push 00000000
:00424C56 8D4DE4               lea ecx, dword ptr [ebp-1C]
:00424C59 6A04                  push 00000004
:00424C5B 51                    push ecx
:00424C5C 6A06                  push 00000006
:00424C5E 8BC8                  mov ecx, eax
:00424C60 C645FC00              mov [ebp-04], 00
:00424C64 A32CA05100            mov dword ptr [0051A02C], eax
:00424C69 8955E4                mov dword ptr [ebp-1C], edx
:00424C6C E8762C0A00            call 004C78E7
:00424C71 8B15E4005100          mov edx, dword ptr [005100E4]
```
