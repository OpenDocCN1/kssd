
```
                                  |

:00455179 BB01000000              mov ebx, 00000001

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00455177(C)

|

:0045517E 8DBC245C060000          lea edi, dword ptr [esp+0000065C]

:00455185 83C9FF                  or ecx, FFFFFFFF

:00455188 33C0                    xor eax, eax

:0045518A 42                      inc edx

:0045518B F2                      repnz

:0045518C AE                      scasb

:0045518D F7D1                    not ecx

:0045518F 49                      dec ecx

:00455190 3BD1                    cmp edx, ecx

:00455192 7CD6                    jl 0045516A

:00455194 85DB                    test ebx, ebx

:00455196 7447                    je 004551DF-------------------------------------------->不跳则到到错误

:00455198 8B0D4C394F00            mov ecx, dword ptr [004F394C]

:0045519E 6804010000              push 00000104

:004551A3 6860C84F00              push 004FC860

\* Possible Reference to String Resource ID=01238: "Incorrect registration !"------------->错误

                                  |

:004551A8 68D6040000              push 000004D6

:004551AD 51                      push ecx

\* Reference To: USER32.LoadStringA, Ord:01ABh

                                  |

:004551AE FF15A0834C00            Call dword ptr [004C83A0]

:004551B4 8B1588CB4F00            mov edx, dword ptr [004FCB88]

:004551BA 6830200000              push 00002030

:004551BF 6800DF4F00              push 004FDF00

:004551C4 6860C84F00              push 004FC860

:004551C9 52                      push edx

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0045522C(U), :00455294(U)

|

\* Reference To: USER32.MessageBoxA, Ord:01BEh

                                  |

:004551CA FF15A4834C00            Call dword ptr [004C83A4]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0045511D(C)

|

:004551D0 33C0                    xor eax, eax

:004551D2 5F                      pop edi

:004551D3 5E                      pop esi

:004551D4 5D                      pop ebp

:004551D5 5B                      pop ebx

:004551D6 81C4580B0000            add esp, 00000B58

:004551DC C21000                  ret 0010

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00455168(C), :00455196(C)

|

:004551DF 8D84245C060000          lea eax, dword ptr [esp+0000065C]

:004551E6 8D8C245C050000          lea ecx, dword ptr [esp+0000055C]

:004551ED 50                      push eax

:004551EE 51                      push ecx

:004551EF E8CC16FEFF              call 004368C0------------------------------------------>关键

:004551F4 83C408                  add esp, 00000008

:004551F7 85C0                    test eax, eax

:004551F9 7533                    jne 0045522E------------------------------------------->关键跳!!!

:004551FB 8B154C394F00            mov edx, dword ptr [004F394C]

:00455201 6804010000              push 00000104

:00455206 6860C84F00              push 004FC860

\* Possible Reference to String Resource ID=01238: "Incorrect registration !"------------->错误

                                  |

:0045520B 68D6040000              push 000004D6

:00455210 52                      push edx

\* Reference To: USER32.LoadStringA, Ord:01ABh

                                  |

:00455211 FF15A0834C00            Call dword ptr [004C83A0]

:00455217 A188CB4F00              mov eax, dword ptr [004FCB88]

:0045521C 6830200000              push 00002030

:00455221 6800DF4F00              push 004FDF00

:00455226 6860C84F00              push 004FC860

:0045522B 50                      push eax

:0045522C EB9C                    jmp 004551CA

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004551F9(C)

|

\* Reference To: KERNEL32.WritePrivateProfileStringA, Ord:02E5h

                                  |

:0045522E 8B35C8814C00            mov esi, dword ptr [004C81C8]

:00455234 8D8C245C050000          lea ecx, dword ptr [esp+0000055C]

:0045523B 68C0DC4F00              push 004FDCC0

:00455240 51                      push ecx

:00455241 6864814E00              push 004E8164

:00455246 6854814E00              push 004E8154

:0045524B FFD6                    call esi

:0045524D 8D94245C060000          lea edx, dword ptr [esp+0000065C]

:00455254 68C0DC4F00              push 004FDCC0

:00455259 52                      push edx

:0045525A 684C814E00              push 004E814C

:0045525F 6854814E00              push 004E8154

:00455264 FFD6                    call esi

:00455266 A14C394F00              mov eax, dword ptr [004F394C]

:0045526B 6804010000              push 00000104

:00455270 6860C84F00              push 004FC860

\* Possible Reference to String Resource ID=01239: "Registration successful !------------->注册成功

Thanks."

                                  |

:00455275 68D7040000              push 000004D7

:0045527A 50                      push eax

\* Reference To: USER32.LoadStringA, Ord:01ABh

                                  |

:0045527B FF15A0834C00            Call dword ptr [004C83A0]

:00455281 8B0D88CB4F00            mov ecx, dword ptr [004FCB88]

:00455287 6A40                    push 00000040

:00455289 6800DF4F00              push 004FDF00

:0045528E 6860C84F00              push 004FC860

:00455293 51                      push ecx

:00455294 E931FFFFFF              jmp 004551CA

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00455137(C), :0045514F(C)

|

:00455299 8B154C394F00            mov edx, dword ptr [004F394C]

:0045529F 6804010000              push 00000104

:004552A4 6860C84F00              push 004FC860

\* Possible Reference to String Resource ID=01238: "Incorrect registration !"------------->错误

                                  |

:004552A9 68D6040000              push 000004D6

:004552AE 52                      push edx

\* Reference To: USER32.LoadStringA, Ord:01ABh

                                  |

:004552AF FF15A0834C00            Call dword ptr [004C83A0]

:004552B5 A188CB4F00              mov eax, dword ptr [004FCB88]

:004552BA 6830200000              push 00002030

:004552BF 6800DF4F00              push 004FDF00

:004552C4 6860C84F00              push 004FC860

:004552C9 50                      push eax

\* Reference To: USER32.MessageBoxA, Ord:01BEh

                                  |

:004552CA FF15A4834C00            Call dword ptr [004C83A4]

:004552D0 33C0                    xor eax, eax

:004552D2 5F                      pop edi

:004552D3 5E                      pop esi

:004552D4 5D                      pop ebp

:004552D5 5B                      pop ebx

:004552D6 81C4580B0000            add esp, 00000B58

:004552DC C21000                  ret 0010

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
```

## OD跟踪生成注册码的过程

```
004368C0  /$  8B4424 08     MOV EAX,DWORD PTR SS:[ESP+8]---------------------->注册码

004368C4  |.  83EC 14       SUB ESP,14

004368C7  |.  53            PUSH EBX

004368C8  |.  55            PUSH EBP

004368C9  |.  56            PUSH ESI

004368CA  |.  57            PUSH EDI

004368CB  |.  50            PUSH EAX

004368CC  |.  33DB          XOR EBX,EBX

004368CE  |.  E8 2AFF0700   CALL UNIRFANV.004B67FD-------------------------->转化注册码为16进制,存于EAX

004368D3  |.  8B7424 2C     MOV ESI,DWORD PTR SS:[ESP+2C]------------------->注册名(字符)

004368D7  |.  8BE8          MOV EBP,EAX

004368D9  |.  8BFE          MOV EDI,ESI

004368DB  |.  83C9 FF       OR ECX,FFFFFFFF-----------------------------------

004368DE  |.  33C0          XOR EAX,EAX                                       |
```
