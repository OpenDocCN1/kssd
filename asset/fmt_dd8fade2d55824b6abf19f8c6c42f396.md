
```asm
:0043A348 50                                  push eax
:0043A349 8D8500FEFFFF                        lea eax, dword ptr [ebp+FFFFFE00]
:0043A34F 50                                  push eax
:0043A350 E836FFFFFF                          call 0043A28B
:0043A355 59                                  pop ecx
:0043A356 59                                  pop ecx
:0043A357 50                                  push eax

* Reference To: CRTDLL.strcmp, Ord:01CFh
|
:0043A358 E8BD3B0000                         Call 0043DF1A
:0043A35D 59                                  pop ecx
:0043A35E 85C0                                test eax, eax
:0043A360 59                                  pop ecx
:0043A361 7478                                je 0043A3DB
:0043A363 8D8500FFFFFF                        lea eax, dword ptr [ebp+FFFFFF00]
:0043A369 50                                  push eax                                  ---d eax , we get our incorrect key:000000
:0043A36A 8D8754190617                        lea eax, dword ptr [edi+17061954]
:0043A370 50                                  push eax
:0043A371 8D8500FEFFFF                        lea eax, dword ptr [ebp+FFFFFE00]
:0043A377 50                                  push eax                                  -----d eax, get another key:144D21A0
:0043A378 E80EFFFFFF                          call 0043A28B
:0043A37D 59                                  pop ecx
:0043A37E 59                                  pop ecx
:0043A37F 50                                  push eax

* Reference To: CRTDLL.strcmp, Ord:01CFh
|
:0043A380 E8953B0000                         Call 0043DF1A
:0043A385 59                                  pop ecx
:0043A386 85C0                                test eax, eax
:0043A388 59                                  pop ecx
:0043A389 7450                                je 0043A3DB
:0043A38B 8D8500FFFFFF                        lea eax, dword ptr [ebp+FFFFFF00]
:0043A391 50                                  push eax
:0043A392 8D8781190510                        lea eax, dword ptr [edi+10051981]
:0043A398 50                                  push eax
:0043A399 8D8500FEFFFF                        lea eax, dword ptr [ebp+FFFFFE00]
:0043A39F 50                                  push eax
:0043A3A0 E8E6FEFFFF                          call 0043A28B
:0043A3A5 59                                  pop ecx
:0043A3A6 59                                  pop ecx
:0043A3A7 50                                  push eax

* Reference To: CRTDLL.strcmp, Ord:01CFh
|
:0043A3A8 E86D3B0000                         Call 0043DF1A
:0043A3AD 59                                  pop ecx
:0043A3AE 85C0                                test eax, eax
:0043A3B0 59                                  pop ecx
:0043A3B1 7455                                je 0043A408
:0043A3B3 8D8500FFFFFF                        lea eax, dword ptr [ebp+FFFFFF00]
:0043A3B9 50                                  push eax                                  ---d eax , we get our incorrect key:000000
:0043A3BA 8D8795190104                        lea eax, dword ptr [edi+04011995]
:0043A3C0 50                                  push eax
:0043A3C1 8D8500FEFFFF                        lea eax, dword ptr [ebp+FFFFFE00]
:0043A3C7 50                                  push eax                                  d eax, get another key:104D21D9
:0043A3C8 E8BEFEFFFF                          call 0043A28B
:0043A3CD 59                                  pop ecx
:0043A3CE 59                                  pop ecx
:0043A3CF 50                                  push eax

* Reference To: CRTDLL.strcmp, Ord:01CFh
|
:0043A3D0 E8453B0000                         Call 0043DF1A
:0043A3D5 59                                  pop ecx
:0043A3D6 85C0                                test eax, eax
:0043A3D8 59                                  pop ecx
:0043A3D9 7505                                jne 0043A3E0

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0043A335(C), :0043A361(C), :0043A389(C)
|
:0043A3DB 6A01                                push 00000001
:0043A3DD 58                                  pop eax
:0043A3DE EB35                                jmp 0043A415

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043A3D9(C)
|
:0043A3E0 8D8500FFFFFF                        lea eax, dword ptr [ebp+FFFFFF00]
:0043A3E6 81C797190602                         add edi, 02061997                       ---d eax , we get our incorrect key:000000
:0043A3EC 50                                  push eax
:0043A3ED 8D8500FEFFFF                        lea eax, dword ptr [ebp+FFFFFE00]
:0043A3F3 57                                  push edi                                  d eax, get another key:44921ED
:0043A3F4 50                                  push eax
:0043A3F5 E891FEFFFF                          call 0043A28B
:0043A3FA 59                                  pop ecx
:0043A3FB 59                                  pop ecx
:0043A3FC 50                                  push eax
```

To sum up, with the user's name: `wlq`, we get altogether four keys: `4B0B5B`, `144D21A0`, `104D21D9`, `44921ED`. `4dd466d0e1e0cc123afbe504da2462ca`
