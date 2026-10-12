
```
:00424DF2 85C0                 test eax, eax
:00424DF4 8975E0               mov dword ptr [ebp-20], esi
:00424DF7 7541                 jne 00424E3A
:00424DF9 8B4DEC               mov ecx, dword ptr [ebp-14]
:00424DFC 8D5508               lea edx, dword ptr [ebp+08]
:00424DFF 51                   push ecx

* Possible StringData Ref from Data Obj -> "Referer: http://%s

"
                                    |
:00424E00 684C025100           push 0051024C
:00424E05 52                   push edx
:00424E06 E8DC490800           call 004A97E7
:00424E0B 83C40C               add esp, 0000000C
:00424E0E 8D4508               lea eax, dword ptr [ebp+08]
:00424E11 8BCE                 mov ecx, esi
:00424E13 6A00                 push 00000000
:00424E15 6A00                 push 00000000
:00424E17 50                   push eax
:00424E18 E884370A00           call 004C85A1
:00424E1D EB77                 jmp 00424E96

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00424DDC(C)
|
:00424E1F 8B4DE8               mov ecx, dword ptr [ebp-18]
:00424E22 51                   push ecx
:00424E23 8B0E                 mov ecx, dword ptr [esi]
:00424E25 6A00                 push 00000000
:00424E27 E8B8350A00           call 004C83E4
:00424E2C 8BD0                 mov edx, eax
:00424E2E A128A05100           mov eax, dword ptr [0051A028]
:00424E33 85C0                 test eax, eax
:00424E35 8955E0               mov dword ptr [ebp-20], edx
:00424E38 7446                 je 00424E80

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00424DF7(C)
|
:00424E3A 8D4D08               lea ecx, dword ptr [ebp+08]
:00424E3D C645FC05             mov [ebp-04], 05
:00424E41 E853C80800           call 004B1699

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00424D5C(C), :00424D86(C)
|
:00424E46 8D4DE8               lea ecx, dword ptr [ebp-18]
:00424E49 C645FC02             mov [ebp-04], 02
:00424E4D E847C80800           call 004B1699
:00424E52 8D4DEC               lea ecx, dword ptr [ebp-14]
:00424E55 C645FC00             mov [ebp-04], 00
:00424E59 E83BC80800           call 004B1699
:00424E5E 8D4D14               lea ecx, dword ptr [ebp+14]
:00424E61 C745FCFFFFFFFF       mov [ebp-04], FFFFFFFF
:00424E68 E82CC80800           call 004B1699
:00424E6D 33C0                 xor eax, eax
:00424E6F 8B4DF4               mov ecx, dword ptr [ebp-0C]
:00424E72 64890D00000000       mov dword ptr fs:[00000000], ecx
:00424E79 5F                   pop edi
:00424E7A 5E                   pop esi
:00424E7B 5B                   pop ebx
:00424E7C 8BE5                 mov esp, ebp
:00424E7E 5D                   pop ebp
:00424E7F C3                   ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00424E38(C)
|
001B:00424E80  8B4514           MOV       EAX,[EBP+14]
001B:00424E83  8B48F8           MOV       ECX,[EAX-08]
001B:00424E86  51               PUSH      ECX
001B:00424E87  50               PUSH      EAX
001B:00424E88  8D4508           LEA       EAX,[EBP+08]
001B:00424E8B  8BCA             MOV       ECX,EDX
001B:00424E8D  50               PUSH      EAX
001B:00424E8E  E80E370A00       CALL      004C85A1  ★★SendRequestA★★
001B:00424E93  8B75E0           MOV       ESI,[EBP-20]  ★★
001B:00424E96  A128A05100       MOV       EAX,[0051A028]  ★★
001B:00424E9B  85C0             TEST      EAX,EAX
001B:00424E9D  0F854E010000     JNZ       00424FF1
001B:00424EA3  85F6             TEST      ESI,ESI
001B:00424EA5  0F8446010000     JZ        00424FF1
001B:00424EAB  8B0DA4105100     MOV       ECX,[005110A4]
001B:00424EB1  894DDC           MOV       [EBP-24],ECX
001B:00424EB4  6844025100       PUSH      00510244
001B:00424EB9  8D4D1C           LEA       ECX,[EBP+1C]
001B:00424EBC  C645FC07          MOV       BYTE PTR [EBP-04],07
001B:00424EC0  E842C80800       CALL      004B1707
001B:00424EC5  A128A05100       MOV       EAX,[0051A028]
001B:00424ECA  C645FC08          MOV       BYTE PTR [EBP-04],08
001B:00424ECE  85C0             TEST      EAX,EAX
001B:00424ED0  0F8503010000     JNZ       00424FD9
001B:00424ED6  6800500000       PUSH      00005000
001B:00424EDB  8D4DAC           LEA       ECX,[EBP-54]
001B:00424EDE  E8916E0900       CALL      004BBD74
001B:00424EE3  8B5510           MOV       EDX,[EBP+10]
001B:00424EE6  8B5D18           MOV       EBX,[EBP+18]
001B:00424EE9  C645FC09          MOV       BYTE PTR [EBP-04],09
001B:00424EED  BE00500000       MOV       ESI,00005000
001B:00424EF2  C70200000000      MOV       DWORD PTR [EDX],00000000
001B:00424EF8  A128A05100       MOV       EAX,[0051A028]
001B:00424EFD  85F6             TEST      ESI,ESI
001B:00424EFF  0F8E8B000000     JLE       00424F90
001B:00424F05  85C0             TEST      EAX,EAX
001B:00424F07  0F85B7000000     JNZ       00424FC4
001B:00424F0D  B900140000       MOV       ECX,00001400
001B:00424F12  33C0             XOR       EAX,EAX
001B:00424F14  8DBDACADFFFF     LEA       EDI,[EBP+FFFFADAC]
001B:00424F1A  8D95ACADFFFF     LEA       EDX,[EBP+FFFFADAC]
001B:00424F20  F3AB             REPZ STOSD
001B:00424F22  8B4DE0           MOV       ECX,[EBP-20]
001B:00424F25  6800500000       PUSH      00005000
001B:00424F2A  52               PUSH      EDX
001B:00424F2B  8B01             MOV       EAX,[ECX]
001B:00424F2D  FF5034           CALL      [EAX+34]  //★★ 004C7C70 ★★该 CALL 取得服务器的应答信息于 【ESP+C】
001B:00424F30  8BF0             MOV       ESI,EAX
001B:00424F32  8D85ACADFFFF     LEA       EAX,[EBP+FFFFADAC]
001B:00424F38  56               PUSH      ESI
001B:00424F39  50               PUSH      EAX
001B:00424F3A  8D4DAC           LEA       ECX,[EBP-54]
001B:00424F3D  E855700900       CALL      004BBF97
001B:00424F42  8B4510           MOV       EAX,[EBP+10]
001B:00424F45  8B08             MOV       ECX,[EAX]
001B:00424F47  03CE             ADD       ECX,ESI
001B:00424F49  85DB             TEST      EBX,EBX
001B:00424F4B  8908             MOV       [EAX],ECX
001B:00424F4D  74A9             JZ        00424EF8
001B:00424F4F  B8CDCCCCCC       MOV       EAX,CCCCCCCD
001B:00424F54  BF64000000       MOV       EDI,00000064
001B:00424F59  F7E1             MUL       ECX
001B:00424F5B  8BC2             MOV       EAX,EDX
001B:00424F5D  33D2             XOR       EDX,EDX
001B:00424F5F  C1E80E           SHR       EAX,0E
001B:00424F62  F7F7             DIV       EDI
001B:00424F64  33FF             XOR       EDI,EDI
001B:00424F66  85FF             TEST      EDI,EDI
001B:00424F68  8BC2             MOV       EAX,EDX
001B:00424F6A  7E0D             JLE       00424F79
001B:00424F6C  8D0489           LEA       EAX,[ECX*4+ECX]
001B:00424F6F  33D2             XOR       EDX,EDX
001B:00424F71  8D0480           LEA       EAX,[EAX*4+EAX]
001B:00424F74  C1E002           SHL       EAX,02
001B:00424F77  F7F7             DIV       EDI
001B:00424F79  8B4B1C           MOV       ECX,[EBX+1C]
001B:00424F7C  6A00             PUSH      00    -->lPara
001B:00424F7E  50               PUSH      EAX   -->wPara
001B:00424F7F  6871040000       PUSH      00000471  -->uMsg
001B:00424F84  51               PUSH      ECX   -->hwnd
```
