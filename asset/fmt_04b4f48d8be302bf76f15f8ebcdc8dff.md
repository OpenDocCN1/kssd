
```asm
:0041C67A C78550FFFFFF147A4000     mov dword ptr [ebp+FFFFFF50], 00407A14
:0041C684 BE08000000              mov esi, 00000008
:0041C689 89B548FFFFFF            mov dword ptr [ebp+FFFFFF48], esi
:0041C68F 8D9548FFFFFF            lea edx, dword ptr [ebp+FFFFFF48]
:0041C695 8D4DB8                  lea ecx, dword ptr [ebp-48]
:0041C698 FFD7                    call edi
:0041C69A C78560FFFFFF04844000    mov dword ptr [ebp+FFFFFF60], 00408404
:0041C6A4 89B558FFFFFF            mov dword ptr [ebp+FFFFFF58], esi
:0041C6AA 8D9558FFFFFF            lea edx, dword ptr [ebp+FFFFFF58]
:0041C6B0 8D4DC8                  lea ecx, dword ptr [ebp-38]
:0041C6B3 FFD7                    call edi
:0041C6B5 8D4D98                  lea ecx, dword ptr [ebp-68]
:0041C6B8 51                      push ecx
:0041C6B9 8D55A8                  lea edx, dword ptr [ebp-58]
:0041C6BC 52                      push edx
:0041C6BD 8D45B8                  lea eax, dword ptr [ebp-48]
:0041C6C0 50                      push eax
:0041C6C1 6A40                    push 00000040
:0041C6C3 8D4DC8                  lea ecx, dword ptr [ebp-38]
:0041C6C6 51                      push ecx

* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h
|
:0041C6C7 FF15A0104000            Call dword ptr [004010A0]
                                  ====>BAD BOY！

:0041C6CD 8D5598                  lea edx, dword ptr [ebp-68]
:0041C6D0 52                      push edx
:0041C6D1 8D45A8                  lea eax, dword ptr [ebp-58]
:0041C6D4 50                      push eax
:0041C6D5 8D4DB8                  lea ecx, dword ptr [ebp-48]
:0041C6D8 51                      push ecx
:0041C6D9 8D55C8                  lea edx, dword ptr [ebp-38]
:0041C6DC 52                      push edx
:0041C6DD 6A04                    push 00000004

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
|
:0041C6DF FF1530104000            Call dword ptr [00401030]
:0041C6E5 83C414                  add esp, 00000014

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041C65F(U)
|

* Reference To: MSVBVM60.__vbaExitProc, Ord:0000h
|
:0041C6E8 FF1588104000            Call dword ptr [00401088]
:0041C6EE 6840C74100              push 0041C740
:0041C6F3 EB4A                    jmp 0041C73F
```

—————————————————————————————————

进入关键 `CALL`：`0041C5CC call dword ptr [edx+00000710]`

…… ……省 略…… ……

```asm
* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
|
:0041C841 8B3D40124000            mov edi, dword ptr [00401240]
:0041C847 FFD7                    call edi
:0041C849 6A01                    push 00000001
:0041C84B 6AFF                    push FFFFFFFF
:0041C84D 6A01                    push 00000001
:0041C84F 68286E4000              push 00406E28
:0041C854 6824844000              push 00408424
:0041C859 8B4E54                  mov ecx, dword ptr [esi+54]
:0041C85C 51                      push ecx
                                  ====>ECX=fly-13572468          试炼码

* Reference To: MSVBVM60.rtcReplace, Ord:02C8h
|
:0041C85D FF158C114000            Call dword ptr [0040118C]
                                  ===>去除试炼码中的-

:0041C863 8BD0                    mov edx, eax
                                  ===>EDX=fly13572468
:0041C865 8D4DCC                  lea ecx, dword ptr [ebp-34]
:0041C868 FFD7                    call edi
:0041C86A 8B55CC                  mov edx, dword ptr [ebp-34]
:0041C86D 52                      push edx
:0041C86E 8B4654                  mov eax, dword ptr [esi+54]
:0041C871 50                      push eax

* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h
|
:0041C872 FF1524114000            Call dword ptr [00401124]
                                  ===>其实这段就是检测试炼码中有无-

:0041C878 8BD8                    mov ebx, eax
                                  ===>返回值。 有 - 则 EAX=1
:0041C87A F7DB                    neg ebx
:0041C87C 1BDB                    sbb ebx, ebx
:0041C87E 43                      inc ebx
                                  ===>EBX=0

:0041C87F 8B4DCC                  mov ecx, dword ptr [ebp-34]
                                  ===>ECX=fly13572468
:0041C882 51                      push ecx

* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h
|
:0041C883 8B3D28104000            mov edi, dword ptr [00401028]
:0041C889 FFD7                    call edi
                                  ===>取 fly13572468 的长度 EAX=B
:0041C88B 33D2                    xor edx, edx
:0041C88D 83F808                  cmp eax, 00000008
                                  ===>和 8 位比较
:0041C890 0F9EC2                  setle dl
                                  ===>设置 DL 值
:0041C893 0BDA                    or ebx, edx
:0041C895 0F8540010000            jne 0041C9DB
                                  ===>跳则 OVER！爆破点①！

:0041C89B 8B45CC                  mov eax, dword ptr [ebp-34]
                                  ===>EAX=fly13572468
:0041C89E 50                      push eax
:0041C89F FFD7                    call edi
                                  ===>取 fly13572468 的长度

:0041C8A1 8985FCFEFFFF            mov dword ptr [ebp+FFFFFEFC], eax
                                  ===>[ebp+FFFFFEFC]=B
:0041C8A7 C7465801000000          mov [esi+58], 00000001

* Reference To: MSVBVM60.__vbaVarTstEq, Ord:0000h
|
:0041C8AE 8B1D2C114000            mov ebx, dword ptr [0040112C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041C996(U)
|

:0041C8B4 8B4658                  mov eax, dword ptr [esi+58]
:0041C8B7 3B85FCFEFFFF            cmp eax, dword ptr [ebp+FFFFFEFC]
:0041C8BD 0F8F18010000            jg 0041C9DB
:0041C8C3 C745BC08000000          mov [ebp-44], 00000008
:0041C8CA C745B402000000          mov [ebp-4C], 00000002
:0041C8D1 8D4DCC                  lea ecx, dword ptr [ebp-34]
:0041C8D4 898D4CFFFFFF            mov dword ptr [ebp+FFFFFF4C], ecx
:0041C8DA C78544FFFFFF08400000    mov dword ptr [ebp+FFFFFF44], 00004008
:0041C8E4 8D55B4                  lea edx, dword ptr [ebp-4C]
:0041C8E7 52                      push edx
:0041C8E8 50                      push eax
:0041C8E9 8D8544FFFFFF            lea eax, dword ptr [ebp+FFFFFF44]
:0041C8EF 50                      push eax
:0041C8F0 8D4DA4                  lea ecx, dword ptr [ebp-5C]
:0041C8F3 51                      push ecx

* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h
|
:0041C8F4 FF15FC104000            Call dword ptr [004010FC]
                                  ===>依次 8 位取 fly13572468
                                  ===>fly13572
                                  …… ……省略…… ……
                                  ===>13572468

:0041C8FA 8D55A4                  lea edx, dword ptr [ebp-5C]
:0041C8FD 52                      push edx
:0041C8FE 8D45C8                  lea eax, dword ptr [ebp-38]
:0041C901 50                      push eax

* Reference To: MSVBVM60.__vbaStrVarVal, Ord:0000h
|
:0041C902 FF15B0114000            Call dword ptr [004011B0]
:0041C908 50                      push eax

* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h
|
:0041C909 FF1578124000            Call dword ptr [00401278]
:0041C90F DC25581D4000            fsub qword ptr [00401D58]
                                  …… ……省略…… ……
```
