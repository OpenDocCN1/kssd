
```asm
:004B1E49 53                 push ebx
:004B1E4A 56                 push esi
:004B1E4B 57                 push edi
:004B1E4C 8B7D08             mov edi, dword ptr [ebp+08]
:004B1E4F 8D4DB0             lea ecx, dword ptr [ebp-50]
:004B1E52 8965F0             mov dword ptr [ebp-10], esp
:004B1E55 8B7704             mov esi, dword ptr [edi+04]
:004B1E58 8975EC             mov dword ptr [ebp-14], esi
:004B1E5B E8FFAFFFFF         call 004ACE5F
:004B1E60 33DB               xor ebx, ebx
:004B1E62 895DFC             mov dword ptr [ebp-04], ebx
:004B1E65 C645FC01           mov [ebp-04], 01
:004B1E69 E815710100         call 004C8F83
:004B1E6E 8B0F               mov ecx, dword ptr [edi]
:004B1E70 8B4904             mov ecx, dword ptr [ecx+04]
:004B1E73 894804             mov dword ptr [eax+04], ecx
:004B1E76 E835730100         call 004C91B0
:004B1E7B 6822924C00         push 004C9222
:004B1E80 8D8870100000       lea ecx, dword ptr [eax+00001070]
:004B1E86 E895910100         call 004CB020
:004B1E8B 897004             mov dword ptr [eax+04], esi
:004B1E8E E8B2010000         call 004B2045
:004B1E93 E818730100         call 004C91B0
:004B1E98 8B4004             mov eax, dword ptr [eax+04]
:004B1E9B 3BC3               cmp eax, ebx
:004B1E9D 7454               je 004B1EF3
:004B1E9F 395E1C             cmp dword ptr [esi+1C], ebx
:004B1EA2 754F               jne 004B1EF3
:004B1EA4 8B401C             mov eax, dword ptr [eax+1C]
:004B1EA7 3BC3               cmp eax, ebx
:004B1EA9 7448               je 004B1EF3
:004B1EAB 39581C             cmp dword ptr [eax+1C], ebx
:004B1EAE 7443               je 004B1EF3
:004B1EB0 FF701C             push [eax+1C]
:004B1EB3 8D4DB0             lea ecx, dword ptr [ebp-50]
:004B1EB6 E80AB4FFFF         call 004AD2C5
:004B1EBB 8D45B0             lea eax, dword ptr [ebp-50]
:004B1EBE 89461C             mov dword ptr [esi+1C], eax
:004B1EC1 EB30               jmp 004B1EF3
:004B1EC3 8D4DB0             lea ecx, dword ptr [ebp-50]
:004B1EC6 E833B4FFFF         call 004AD2FE
:004B1ECB 8B4508             mov eax, dword ptr [ebp+08]
:004B1ECE FF7010             push [eax+10]
:004B1ED1 C7401801000000     mov [eax+18], 00000001

* Reference To: kernel32.Ä‹Æ^Â, Ord:0000h

:004B1ED8 FF156C034E00       Call dword ptr [004E036C]
:004B1EDE 6A00               push 00000000

* Possible Reference to Dialog: DialogID_00A7, CONTROL_ID:00FF, ""

:004B1EE0 6AFF               push FFFFFFFF
:004B1EE2 E819010000         call 004B2000
:004B1EE7 B8ED1E4B00         mov eax, 004B1EED
:004B1EEC C3                 ret
:004B1EED 8B7D08             mov edi, dword ptr [ebp+08]
:004B1EF0 8B75EC             mov esi, dword ptr [ebp-14]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004B1E9D(C), :004B1EA2(C), :004B1EA9(C), :004B1EAE(C), :004B1EC1(U)

001B:004B1EED  8B7D08        MOV       EDI,[EBP+08]
001B:004B1EF0  8B75EC        MOV       ESI,[EBP-14]
001B:004B1EF3  FF7710        PUSH      DWORD PTR [EDI+10]
001B:004B1EF6  8365FC00      AND       DWORD PTR [EBP-04],00
001B:004B1EFA  8B5F14        MOV       EBX,[EDI+14]
001B:004B1EFD  FF156C034E00  CALL      [KERNEL32!SetEvent]
001B:004B1F03  6AFF          PUSH      FF
001B:004B1F05  53            PUSH      EBX
001B:004B1F06  FF1570034E00  CALL      [KERNEL32!WaitForSingleObject]
001B:004B1F0C  53            PUSH      EBX
001B:004B1F0D  FF1574034E00  CALL      [KERNEL32!CloseHandle]
001B:004B1F13  8B4650        MOV       EAX,[ESI+50]
001B:004B1F16  85C0          TEST      EAX,EAX
001B:004B1F18  7408          JZ        004B1F22
001B:004B1F1A  FF764C        PUSH      DWORD PTR [ESI+4C]
001B:004B1F1D  FFD0          CALL      EAX  //显示登录信息时 EAX= 00426FE0 ，显示 10 条限制信息时，EAX=00519060,见★E★！★★★
001B:004B1F1F  59            POP       ECX  //00426FF9 RET HERE
001B:004B1F20  EB17          JMP       004B1F39

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B1F18(C)

001B:004B1F22  8B06          MOV       EAX,[ESI]
001B:004B1F24  8BCE          MOV       ECX,ESI
001B:004B1F26  FF5050        CALL      [EAX+50]
001B:004B1F29  85C0          TEST      EAX,EAX
001B:004B1F2B  8B06          MOV       EAX,[ESI]
001B:004B1F2D  8BCE          MOV       ECX,ESI
001B:004B1F2F  7505          JNZ       004B1F36
001B:004B1F31  FF5068        CALL      [EAX+68]
001B:004B1F34  EB03          JMP       004B1F39

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004B1F2F(C)

001B:004B1F36  FF5054        CALL      [EAX+54]

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004B1F20(U), :004B1F34(U)

001B:004B1F39  8D4DB0        LEA       ECX,[EBP-50]
001B:004B1F3C  8BF0          MOV       ESI,EAX
001B:004B1F3E  E8BBB3FFFF    CALL      004AD2FE
001B:004B1F43  6A01          PUSH      01
001B:004B1F45  56            PUSH      ESI
001B:004B1F46  E8B5000000    CALL      004B2000  //此 CALL 将结束网上验证线程，返回登录线程（“用户登录”对话框），显示验证结果信息，见下面；
001B:004B1F4B  834DFCFF      OR        DWORD PTR [EBP-04],-01
001B:004B1F4F  8D4DB0        LEA       ECX,[EBP-50]
001B:004B1F52  E8E2B9FFFF    CALL      004AD939
001B:004B1F57  8B4DF4        MOV       ECX,[EBP-0C]
001B:004B1F5A  5F            POP       EDI
001B:004B1F5B  5E            POP       ESI
001B:004B1F5C  33C0          XOR       EAX,EAX
001B:004B1F5E  64890D00000000 MOV     FS:[00000000],ECX
001B:004B1F65  5B            POP       EBX
001B:004B1F66  C9            LEAVE
001B:004B1F67  C20400        RET       0004  //ret to

======================================================================

001B:004B1F46  CALL      004B2000  //此 CALL 将结束网上验证线程，返回登录线程（“用户登录”对话框），显示验证结果信息。

001B:004B2000  56            PUSH      ESI
001B:004B2001  57            PUSH      EDI
001B:004B2002  E8CF710100    CALL      004C91D6
001B:004B2007  8BF8          MOV       EDI,EAX
001B:004B2009  8B7704        MOV       ESI,[EDI+04]
001B:004B200C  85F6          TEST      ESI,ESI
001B:004B200E  741F          JZ        004B202F
001B:004B2010  8B4654        MOV       EAX,[ESI+54]
001B:004B2013  85C0          TEST      EAX,EAX
001B:004B2015  7406          JZ        004B201D
001B:004B2017  6A00          PUSH      00
001B:004B2019  6A01          PUSH      01
001B:004B201B  FFD0          CALL      EAX
001B:004B201D  837C241000    CMP       DWORD PTR [ESP+10],00
001B:004B2022  7407          JZ        004B202B
001B:004B2024  8B06          MOV       EAX,[ESI]
001B:004B2026  8BCE          MOV       ECX,ESI
001B:004B2028  FF5078        CALL      [EAX+78]
001B:004B202B  83670400      AND       DWORD PTR [EDI+04],00
001B:004B202F  6A00          PUSH      00
001B:004B2031  E86E000000    CALL      004B20A4
001B:004B2036  FF74240C      PUSH      DWORD PTR [ESP+0C]
```
