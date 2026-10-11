
```
:00500168 FF7508                push [ebp+08]
:0050016B FF75E4                push [ebp-1C]
:0050016E FF75E0                push [ebp-20]
:00500171 8B45F0                mov eax, dword ptr [ebp-10]
                                ====>EAX=[ebp-10]=0A1CE262
:00500174 8B55F4                mov edx, dword ptr [ebp-0C]
:00500177 E84459F0FF           call 00405AC0
                                ====>子运算 CALL 2
:0050017C E8DF59F0FF           call 00405B60
                                ====>子运算 CALL 3
:00500181 8945E0                mov dword ptr [ebp-20], eax
                                ===>[ebp-20]=EAX=0A1CE262
                                ===>循环结束后 EAX=5D7B911B    呵呵,这就是注册码的前 8 位了!
:00500184 8955E4                mov dword ptr [ebp-1C], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:005000FC(U), :00500151(U)
|
:00500187 837DEC00              cmp dword ptr [ebp-14], 00000000
:0050018B 0F8570FFFFFF          jne 00500101
:00500191 837DE800              cmp dword ptr [ebp-18], 00000000
                                ===>[ebp-18]=002C86F9
:00500195 0F8566FFFFFF          jne 00500101
                                ===>呵呵，疯狂循环直至 002C86F9 为 0！
:0050019B 8B45E0                mov eax, dword ptr [ebp-20]
:0050019E 8945F8                mov dword ptr [ebp-08], eax
:005001A1 8B45E4                mov eax, dword ptr [ebp-1C]
:005001A4 8945FC                mov dword ptr [ebp-04], eax
:005001A7 8B45F8                mov eax, dword ptr [ebp-08]
:005001AA 8B55FC                mov edx, dword ptr [ebp-04]
:005001AD 8BE5                  mov esp, ebp
:005001AF 5D                    pop ebp
:005001B0 C21800                ret 0018

---------------------------------------------------
进入子运算 CALL 1: `:00500124` `call 00405AE4`

* Referenced by a CALL at Addresses:
|:00405C6F   , :0041636F   , :00500124
|
:00405AE4 55                   push ebp
:00405AE5 53                   push ebx
:00405AE6 56                   push esi
:00405AE7 57                   push edi
:00405AE8 31FF                 xor edi, edi
:00405AEA 8B5C2414             mov ebx, dword ptr [esp+14]
:00405AEE 8B4C2418             mov ecx, dword ptr [esp+18]
:00405AF2 09C9                 or ecx, ecx
:00405AF4 7508                 jne 00405AFE
:00405AF6 09D2                 or edx, edx
:00405AF8 745C                 je 00405B56
:00405AFA 09DB                 or ebx, ebx
:00405AFC 7458                 je 00405B56

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405AF4(C)
|
:00405AFE 09D2                 or edx, edx
:00405B00 790A                 jns 00405B0C
:00405B02 F7DA                 neg edx
:00405B04 F7D8                 neg eax
:00405B06 83DA00               sbb edx, 00000000
:00405B09 83CF01               or edi, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405B00(C)
|
:00405B0C 09C9                 or ecx, ecx
:00405B0E 790A                 jns 00405B1A
:00405B10 F7D9                 neg ecx
:00405B12 F7DB                 neg ebx
:00405B14 83D900               sbb ecx, 00000000
:00405B17 83F701               xor edi, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405B0E(C)
|
:00405B1A 89CD                 mov ebp, ecx
:00405B1C B940000000           mov ecx, 00000040
:00405B21 57                   push edi
:00405B22 31FF                 xor edi, edi
:00405B24 31F6                 xor esi, esi
:00405B26 D1E0                 shl eax, 1
:00405B28 D1D2                 rcl edx, 1
:00405B2A D1D6                 rcl esi, 1
:00405B2C D1D7                 rcl edi, 1
:00405B2E 39EF                 cmp edi, ebp
:00405B30 720B                 jb 00405B3D
:00405B32 7704                 ja 00405B38
:00405B34 39DE                 cmp esi, ebx
:00405B36 7205                 jb 00405B3D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405B32(C)
|
:00405B38 29DE                 sub esi, ebx
:00405B3A 19EF                 sbb edi, ebp
:00405B3C 40                   inc eax

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00405B30(C), :00405B36(C)
|
:00405B3D E2E7                 loop 00405B26
:00405B3F 5B                   pop ebx
:00405B40 F7C301000000         test ebx, 00000001
:00405B46 7407                 je 00405B4F
:00405B48 F7DA                 neg edx
:00405B4A F7D8                 neg eax
:00405B4C 83DA00               sbb edx, 00000000

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00405B46(C), :00405B5A(U)
|
:00405B4F 5F                   pop edi
:00405B50 5E                   pop esi
:00405B51 5B                   pop ebx
:00405B52 5D                   pop ebp
:00405B53 C20800               ret 0008

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00405AF8(C), :00405AFC(C)
|
:00405B56 F7F3                 div ebx
:00405B58 31D2                 xor edx, edx
:00405B5A EBF3                 jmp 00405B4F
:00405B5C C3                   ret

---------------------------------------------------
进入子运算 CALL 2: `call 00405AC0`

* Referenced by a CALL at Addresses:
|:00405E95   , :00409F6F   , :00409F89   , :00416318   , :00440B51
|:00500141   , :00500177
|
:00405AC0 52                   push edx
:00405AC1 50                   push eax
:00405AC2 8B442410             mov eax, dword ptr [esp+10]
:00405AC6 F72424               mul dword ptr [esp]
:00405AC9 89C1                 mov ecx, eax
:00405ACB 8B442404             mov eax, dword ptr [esp+04]
:00405ACF F764240C             mul [esp+0C]
:00405AD3 01C1                 add ecx, eax
:00405AD5 8B0424               mov eax, dword ptr [esp]
:00405AD8 F764240C             mul [esp+0C]
:00405ADC 01CA                 add edx, ecx
:00405ADE 59                   pop ecx
:00405ADF 59                   pop ecx
:00405AE0 C20800               ret 0008

---------------------------------------------------
进入子运算 CALL 3: `call 00405B60`

* Referenced by a CALL at Addresses:
|:00405C4A   , :0041638B   , :0050010B   , :00500146   , :0050017C
|
:00405B60 55                   push ebp
:00405B61 53                   push ebx
:00405B62 56                   push esi
:00405B63 57                   push edi
:00405B64 31FF                 xor edi, edi
:00405B66 8B5C2414             mov ebx, dword ptr [esp+14]
:00405B6A 8B4C2418             mov ecx, dword ptr [esp+18]
:00405B6E 09C9                 or ecx, ecx
:00405B70 7508                 jne 00405B7A
:00405B72 09D2                 or edx, edx
:00405B74 745D                 je 00405BD3
:00405B76 09DB                 or ebx, ebx
:00405B78 7459                 je 00405BD3

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405B70(C)
|
:00405B7A 09D2                 or edx, edx
:00405B7C 790A                 jns 00405B88
:00405B7E F7DA                 neg edx
:00405B80 F7D8                 neg eax
:00405B82 83DA00               sbb edx, 00000000
:00405B85 83CF01               or edi, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405B7C(C)
|
:00405B88 09C9                 or ecx, ecx
```
