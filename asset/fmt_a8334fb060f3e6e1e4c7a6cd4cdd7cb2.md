
```
:100092AD 8D4C240C                  lea ecx, dword ptr [esp+0C]

:100092B1 E89A9BFFFF                call 10002E50

:100092B6 8B742458                  mov esi, dword ptr [esp+58]

:100092BA C744245000000000          mov [esp+50], 00000000

:100092C2 8B4602                    mov eax, dword ptr [esi+02]

:100092C5 85C0                      test eax, eax

:100092C7 7471                      je 1000933A

:100092C9 8D7E14                    lea edi, dword ptr [esi+14]

:100092CC 6A09                      push 00000009

* Possible StringData Ref from Data Obj ->"stock.dll"

                                      |
:100092CE 683C570410                push 1004573C

:100092D3 57                        push edi

:100092D4 E877200100                call 1001B350

:100092D9 83C40C                    add esp, 0000000C

:100092DC 85C0                      test eax, eax

:100092DE 752F                      jne 1000930F

:100092E0 56                        push esi

:100092E1 8BCB                      mov ecx, ebx

:100092E3 E888000000                call 10009370            <----继续向上

:100092E8 8D4C240C                  lea ecx, dword ptr [esp+0C]

:100092EC C7442450FFFFFFFF          mov [esp+50], FFFFFFFF

:100092F4 E8179DFFFF                call 10003010

:100092F9 5F                        pop edi

:100092FA 5E                        pop esi

:100092FB 33C0                      xor eax, eax

:100092FD 5B                        pop ebx

:100092FE 8B4C243C                  mov ecx, dword ptr [esp+3C]

:10009302 64890D00000000            mov dword ptr fs:[00000000], ecx

:10009309 83C448                    add esp, 00000048

:1000930C C20400                    ret 0004

* Referenced by a CALL at Address:

|:100092E3                                          <----CALL，右键，看看是哪里来的
|

:10009370 81EC00030000            sub esp, 00000300

:10009376 8D842400020000          lea eax, dword ptr [esp+00000200]

:1000937D 53                      push ebx

:1000937E 55                      push ebp

:1000937F 56                      push esi

:10009380 57                      push edi

:10009381 6880000000              push 00000080

:10009386 50                      push eax

* Reference To: KERNEL32.GetSystemDirectoryA, Ord:0159h     <----获得系统目录，继续向上

                                      |

:10009387 FF15E4710310            Call dword ptr [100371E4]

:1000938D 83C9FF                    or ecx, FFFFFFFF

* Possible StringData Ref from Data Obj ->"\\"

                                      |
:10009390 BFE4520410                mov edi, 100452E4

:10009395 33C0                      xor eax, eax

:10009397 8D942410020000          lea edx, dword ptr [esp+00000210]

:1000939E F2                        repnz

:1000939F AE                        scasb

:100093A0 F7D1                      not ecx

:100093A2 2BF9                      sub edi, ecx

:100093A4 8BD9                      mov ebx, ecx

:100093A6 8BF7                      mov esi, edi

:100093A8 83C9FF                    or ecx, FFFFFFFF

:100093AB 8BFA                      mov edi, edx

:100093AD F2                        repnz

:100093AE AE                        scasb

:100093AF 8BCB                      mov ecx, ebx

:100093B1 4F                        dec edi

:100093B2 C1E902                    shr ecx, 02

:100093B5 F3                        repz

:100093B6 A5                        movsd

:100093B7 8BCB                      mov ecx, ebx

:100093B9 8D542410                  lea edx, dword ptr [esp+10]

:100093BD 83E103                    and ecx, 00000003

:100093C0 8B9C2414030000          mov ebx, dword ptr [esp+00000314]

:100093C7 F3                        repz

:100093C8 A4                        movsb

:100093C9 83C9FF                    or ecx, FFFFFFFF

:100093CC 8DBC2410020000          lea edi, dword ptr [esp+00000210]

:100093D3 F2                        repnz

:100093D4 AE                        scasb

:100093D5 F7D1                      not ecx

:100093D7 2BF9                      sub edi, ecx

:100093D9 8BC1                      mov eax, ecx

:100093DB 8BF7                      mov esi, edi

:100093DD 8BFA                      mov edi, edx

:100093DF C1E902                    shr ecx, 02

:100093E2 F3                        repz

:100093E3 A5                        movsd

:100093E4 8BC8                      mov ecx, eax

:100093E6 33C0                      xor eax, eax

:100093E8 83E103                    and ecx, 00000003

:100093EB F3                        repz

:100093EC A4                        movsb

:100093ED 8D7C2410                  lea edi, dword ptr [esp+10]

:100093F1 83C9FF                    or ecx, FFFFFFFF

:100093F4 F2                        repnz

:100093F5 AE                        scasb

:100093F6 0FBF530E                  movsx edx, word ptr [ebx+0E]

:100093FA F7D1                      not ecx

:100093FC 49                        dec ecx

:100093FD 8D7314                    lea esi, dword ptr [ebx+14]

:10009400 8D7C0C10                  lea edi, dword ptr [esp+ecx+10]

:10009404 8BCA                      mov ecx, edx

:10009406 8BC1                      mov eax, ecx

:10009408 C1E902                    shr ecx, 02

:1000940B F3                        repz

:1000940C A5                        movsd

:1000940D 8BC8                      mov ecx, eax

:1000940F 33C0                      xor eax, eax

:10009411 83E103                    and ecx, 00000003

:10009414 F3                        repz

:10009415 A4                        movsb

:10009416 8D7C2410                  lea edi, dword ptr [esp+10]

:1000941A 83C9FF                    or ecx, FFFFFFFF

:1000941D F2                        repnz

:1000941E AE                        scasb

:1000941F F7D1                      not ecx

:10009421 49                        dec ecx

:10009422 8DBC2410020000          lea edi, dword ptr [esp+00000210]

:10009429 03CA                      add ecx, edx

:1000942B 8D942410010000          lea edx, dword ptr [esp+00000110]

:10009432 88440C10                  mov byte ptr [esp+ecx+10], al

:10009436 83C9FF                    or ecx, FFFFFFFF

:10009439 F2                        repnz

:1000943A AE                        scasb

:1000943B F7D1                      not ecx

:1000943D 2BF9                      sub edi, ecx

:1000943F 8BC1                      mov eax, ecx

:10009441 8BF7                      mov esi, edi

:10009443 8BFA                      mov edi, edx

:10009445 C1E902                    shr ecx, 02

:10009448 F3                        repz

:10009449 A5                        movsd

:1000944A 8BC8                      mov ecx, eax

:1000944C 83E103                    and ecx, 00000003

:1000944F F3                        repz

:10009450 A4                        movsb

* Possible StringData Ref from Data Obj ->"stock000.dll"    <----来到这里，从这里向上找

                                      |
:10009451 BF40590410                mov edi, 10045940

:10009456 83C9FF                    or ecx, FFFFFFFF

:10009459 33C0                      xor eax, eax

:1000945B 8D942410010000          lea edx, dword ptr [esp+00000110]

:10009462 F2                        repnz

:10009463 AE                        scasb

:10009464 F7D1                      not ecx

:10009466 2BF9                      sub edi, ecx

:10009468 6880000000              push 00000080

:1000946D 8BF7                      mov esi, edi

:1000946F 8BE9                      mov ebp, ecx

:10009471 8BFA                      mov edi, edx

:10009473 83C9FF                    or ecx, FFFFFFFF

:10009476 F2                        repnz

:10009477 AE                        scasb

:10009478 8BCD                      mov ecx, ebp

:1000947A 4F                        dec edi

:1000947B C1E902                    shr ecx, 02

:1000947E F3                        repz

:1000947F A5                        movsd

:10009480 8BCD                      mov ecx, ebp
```
