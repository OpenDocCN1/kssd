
```asm
jne 004920CE          出错跳转  修改处6

 :00491ED1

A12C705500              mov eax, dword ptr

[0055702C]

 :00491ED6 8944242C

  mov dword ptr [esp+2C], eax

 :00491EDA 8D4C242C

          lea ecx, dword ptr [esp+2C]

 :00491EDE

C644245C07              mov [esp+5C], 07

:00491EE3 51

  push ecx

 \* Possible StringData Ref from Data Obj ->"key3"

          |

 :00491EE4 68AC625500

        push 005562AC

 :00491EE9 8D4C2440

            lea ecx, dword ptr [esp+40]

 :00491EED

E888450400              call 004D647A

:00491EF2 85C0

test eax, eax

 :00491EF4 0F84C1010000

je 004920BB        出错跳转  修改处7

 :00491EFA 8DBE94000000

          lea edi, dword ptr [esi+00000094]

 :00491F00

BD01000000              mov ebp, 00000001

:00491F05 8BCF

mov ecx, edi

 :00491F07 E808070400

  call 004D2614

 :00491F0C 8BCF

        mov ecx, edi

 :00491F0E E84D060400

          call 004D2560

 :00491F13 6A00

                push 00000000

 :00491F15

8D4C2430                lea ecx, dword

ptr [esp+30]

 :00491F19 E8789C0400

  call 004DBB96

 :00491F1E 50

          push eax

 :00491F1F E88CE6FFFF

          call 004905B0

 :00491F24 83C404

              add esp, 00000004

 :00491F27

85C0                    test

eax, eax

 :00491F29 0F847E010000

je 004920AD

 :00491F2F 6A00

      push 00000000

 :00491F31 8BCF

            mov ecx, edi

 :00491F33 E85E9C0400

            call 004DBB96

 :00491F38 50

                    push eax

 :00491F39 E802E6FFFF              call

00490540

 :00491F3E 83C404

    add esp, 00000004

 :00491F41 85C0

            test eax, eax

 :00491F43 0F8464010000

          je 004920AD

 :00491F49 E8F2E8FFFF

            call 00490840

 :00491F4E 85C0

                  test eax, eax

:00491F50 0F8457010000            je 004920AD

    出错跳转  修改处8

 \* Possible StringData Ref from Data Obj

->"RegUserName"

                |

 :00491F56 68FC135500

            push 005513FC

 :00491F5B 8D4C2438

              lea ecx, dword ptr [esp+38]

:00491F5F E870980400              call 004DB7D4

 :00491F64 68B49B5500              push

00559BB4

 :00491F69 8D4C2434

  lea ecx, dword ptr [esp+34]

 :00491F6D 885C2460

          mov byte ptr [esp+60], bl

 :00491F71 E85E980400

            call 004DB7D4

 :00491F76 8D9698000000

          lea edx, dword ptr [esi+00000098]

 :00491F7C

8D442434                lea eax, dword

ptr [esp+34]

 :00491F80 52

        push edx

 :00491F81 8D4C2434

          lea ecx, dword ptr [esp+34]

 :00491F85

50

push eax

 :00491F86 51

      push ecx

 :00491F87 C644246809

      mov [esp+68], 09

 :00491F8C E8DF23FFFF

        call 00484370

 :00491F91 83C40C

            add esp, 0000000C

 :00491F94 8D4C2430

              lea ecx, dword ptr [esp+30]

:00491F98 885C245C                mov

byte ptr [esp+5C], bl

 :00491F9C E8C5970400

    call 004DB766

 :00491FA1 8D4C2434

        lea ecx, dword ptr [esp+34]

 :00491FA5 C644245C07

            mov [esp+5C], 07

 :00491FAA E8B7970400

            call 004DB766

 :00491FAF 8B86AC000000

          mov eax, dword ptr [esi+000000AC]

 :00491FB5

85C0                    test

eax, eax

 :00491FB7 740B

    je 00491FC4    出错跳转  修改处9

 :00491FB9 E8A1450600

            call 004F655F

 :00491FBE 8B4004

                mov eax, dword ptr [eax+04]

 :00491FC1 55

    push ebp

 :00491FC2 EB0A

        jmp 00491FCE

 \* Referenced by a (U)nconditional

or (C)onditional Jump at Address:

 |:00491FB7(C)

 |

 :00491FC4 E896450600

            call 004F655F

 :00491FC9 8B4004

                mov eax, dword ptr [eax+04]

 :00491FCC 6A00

  push 00000000

 \* Referenced by a (U)nconditional or (C)onditional

Jump at Address:

 |:00491FC2(U)

 |

 \* Possible StringData Ref

from Data Obj ->"DoNotGrayRegMenu"

                      |

 :00491FCE 68BC5D5500              push

00555DBC

 \* Possible StringData Ref from Data Obj ->"CuteFTP"

        |

 :00491FD3 68F4135500

      push 005513F4

 :00491FD8 8BC8

            mov ecx, eax

 :00491FDA E836130500

            call 004E3315

 :00491FDF 8B152C705500

          mov edx, dword ptr [0055702C]

 :00491FE5

89542464                mov dword ptr

[esp+64], edx

 :00491FE9 8D442464

    lea eax, dword ptr [esp+64]

 :00491FED 8D4C2438

            lea ecx, dword ptr [esp+38]

 :00491FF1

50

push eax

 \* Possible StringData Ref from Data Obj ->"type"

        |

 :00491FF2 68A4625500

      push 005562A4

 :00491FF7 C64424640A

        mov [esp+64], 0A

 :00491FFC E879440400

          call 004D647A

 :00492001 8B4C2464

              mov ecx, dword ptr [esp+64]

 \* Possible StringData Ref from Data Obj ->"NEWREGISTR"

      |

 :00492005 6898625500

    push 00556298

 :0049200A 51

            push ecx

 :0049200B E879050300

            call 004C2589

 :00492010 83C408

                add esp, 00000008

:00492013 85C0

test eax, eax

 :00492015 7507

      jne 0049201E      出错跳转

 \* Possible

Reference to String Resource ID=61516: "m? CuteFTP o,

魉s桶/?CuteFTP"如果走到这一步，则注册成功

                    |

 :00492017

B84CF00000              mov eax, 0000F04C

:0049201C EB1D

jmp 0049203B

                    |

 整理一下：

 :004B7268 7573------------9090

 :004B72BC 7510------------9090

 :0043BCF6 7572------------EB72

 :00491581 756E------------EB6E

 :004A394D

745E------------EB5E

 :00491ECB 0F85FD010000----909090909090

 :00491EF4

0F84C1010000----909090909090

 :00491F50 0F8457010000----909090909090

:00491FB7 740B------------9090
```

任意名注册可通过验证~
