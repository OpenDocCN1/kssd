
# CuteFTP最新版V4.2.5build3.7.1官方简体中文版 破解 (15千字)

标 题:CuteFTP最新版V4.2.5build3.7.1官方简体中文版 破解 (15千字)

发信人:sambarain

时 间:2002-4-9 22:03:32

详细信息:

工具：`ollydbg`.`exescope`.`w32dasm`.

保护：每次启动都弹出注册窗，提示上网注册; 30 天试用期；动态CRC校验。

使用`ollydbg`载入`cutftp32.exe`,查找标志`getwindowtexta`,共有6处，分别设断点，F9运行`cutftp`

,提示在线注册nag窗口弹出。

选择“输入产品序号”

产品序号填：012345678901234

用户名填：sambarain

点击注册

`trw`中断在

\* Reference To: USER32.GetWindowTextA, Ord:015Eh

          |

:004E3647 FF157CC55100

        Call dword ptr [0051C57C]

:004E364D 8B4D10

                mov ecx, dword ptr [ebp+10]

\* Possible Reference to String Resource ID=00255: "\*~0SM俟y??&?

            |

:004E3650 6AFF

              push FFFFFFFF

:004E3652

E88E85FFFF              call 004DBBE5

:004E3657 EB0B

jmp 004E3664

一路F10，直到

:00491570 50

              push eax

:00491571 8D442410

              lea eax, dword ptr [esp+10]

:00491575 50

  push eax

:00491576 E8D55C0200

  call 004B7250              所以要检查这个CALL

:0049157B 83C408

add esp, 00000008

:0049157E 6685C0

      test ax, ax

:00491581 756E

            jne 004915F1

              一定要让它跳走  修改4

:00491583

8D4C2414                lea ecx, dword

ptr [esp+14]

\* Possible Reference to String Resource ID=61533:

"1%"          注册失败  所以要跳过它

      |

:00491587 685DF00000

    push 0000F05D

:0049158C 51

            push ecx

:0049158D E87E40F9FF

            call 00425610

:00491592 8B38

                  mov edi, dword

ptr [eax]

:00491594 8D542418

  lea edx, dword ptr [esp+18]

　追踪`call 004B7250`　　　　　　　　　　　　　开始注册码比较

\* Referenced by a CALL at Addresses:

|:004908E7  , :00490B72

, :00491576

|

:004B7250 83EC20

        sub esp, 00000020　　　　　　　　　　　

:004B7253 83C9FF

                or ecx, FFFFFFFF

:004B7256 33C0

xor eax, eax

:004B7258 56

        push esi

:004B7259 8B74242C

          mov esi, dword ptr [esp+2C]

:004B725D

57

push edi

:004B725E 8BFE

    mov edi, esi

:004B7260 F2

            repnz

:004B7261 AE

                  scasb

:004B7262

F7D1                    not

ecx

:004B7264 49

    dec ecx

:004B7265 83F90E

        cmp ecx, 0000000E        检查序列号是否是14位,不对则跳

:004B7268 7573

  jne 004B72DD　　　　　　　----------------------修改处1

:004B726A 56

                  push esi

:004B726B

E823DD0000              call 004C4F93

\* Possible Reference to String Resource ID=00014: "? URL"

      |

:004B7270 6A0E

        push 0000000E

:004B7272 8D442420

            lea eax, dword ptr [esp+20]

:004B7276

56

push esi

:004B7277 50

      push eax

:004B7278 E823A70000

      call 004C19A0

:004B727D 8D4C2428

          lea ecx, dword ptr [esp+28]

:004B7281

C644243600              mov [esp+36], 00

:004B7286 51

  push ecx

:004B7287 E8D422FEFF

  call 00499560

:004B728C 8BF0

        mov esi, eax

:004B728E 56

                push esi

:004B728F

E8BC1FFEFF              call 00499250

:004B7294 8D542420                lea

edx, dword ptr [esp+20]

:004B7298 8BF8

          mov edi, eax

:004B729A 52

                  push edx

:004B729B

56

push esi

:004B729C C644242800

mov [esp+28], 00

:004B72A1 E83A21FEFF

  call 004993E0

:004B72A6 8D442438

      lea eax, dword ptr [esp+38]

查看EAX的值为012345678901234

\* Possible Reference to String Resource ID=00014:

"? URL"

              |

:004B72AA 6A0E

                push 0000000E

:004B72AC

8D4C242C                lea ecx, dword

ptr [esp+2C]          查看ECX的值为a22222222222222

:004B72B0

50

push eax

:004B72B1 51

      push ecx

:004B72B2 E8E9C50000

      call 004C38A0　　　　　具体验证CALL

:004B72B7 83C42C

              add esp, 0000002C

:004B72BA

85C0                    test

eax, eax          eax=1就over,=0正确

:004B72BC 7510

                  jne 004B72CE

        ----------------------修改处2

:004B72BE 8B54242C

      mov edx, dword ptr [esp+2C]

:004B72C2 660DFFFF

              or ax, FFFF

:004B72C6 893A

                  mov dword ptr [edx],

edi

:004B72C8 5F

    pop edi

:004B72C9 5E

          pop esi

:004B72CA 83C420

              add esp, 00000020

:004B72CD

C3

ret　　　　　　　　　　　　　　　

修改

:004B7268 7573－－－－－＞`9090`

:004B72BC 7510－－－－－＞`9090`

修改后，运行程序报告crc校验失败

使用`exescope`查看`cutftp.exe`资源,找到出错语句ID=00426

用`32dasm`, 反汇编`cutftp32.exe`,查找”ID=00426“

\* Possible Reference to Dialog:

DialogID\_0181

                |

:0043BCD4 6881010000

            push 00000181

:0043BCD9 57

                    push edi

:0043BCDA 57

    push edi

:0043BCDB 8B501C

        mov edx, dword ptr [eax+1C]

:0043BCDE 52

                    push edx

\* Reference To: USER32.RedrawWindow, Ord:01F1h

    |

:0043BCDF FF153CC65100

  Call dword ptr [0051C63C]

:0043BCE5 C7835006000001000000

mov dword ptr [ebx+00000650], 00000001

:0043BCEF E86C500500

          call 00490D60

:0043BCF4 85C0

                  test eax, eax

:0043BCF6 7572

jne 0043BD6A　　crc正确则跳转  --------------修改3

:0043BCF8 8D45EC

              lea eax, dword ptr [ebp-14]

\* Possible Reference to String Resource ID=00426: "CuteFTP ??%?s@c(("

            |

:0043BCFB 68AA010000

          push 000001AA

:0043BD00 50

                  push eax

:0043BD01

E80A99FEFF              call 004256100043BCEF

:0043BD06 83C408

add esp, 00000008

:0043BD09 8B00

        mov eax, dword ptr [eax]

:0043BD0B 57

                  push edi

修改:`0043BCF6 7572`--------------->`EB72`

重新运行程序，可是进行注册时程序自动连接服务器进行注册码验证,返回出错信息

在:`004B72BA`中断

跟踪到

\* Reference To: KERNEL32.LoadResource,

Ord:01C7h

                |

:00490BF9 FF15B0C25100

          Call dword ptr [0051C2B0]

:00490BFF 50

                    push eax

\* Reference To: KERNEL32.LockResource, Ord:01D5h

      |

:00490C00 FF15B4C25100

    Call dword ptr [0051C2B4]

:00490C06 8D8C2484080000

      lea ecx, dword ptr [esp+00000884]

:00490C0D 50

                    push eax

:00490C0E 51

    push ecx

:00490C0F 8D94245C100000

  lea edx, dword ptr [esp+0000105C]

:00490C16 56

                push esi

:00490C17

52

push edx

:00490C18 E8132C0100

call 004A3830                ＜－－－－－进入

:00490C1D 83C410

add esp, 00000010

:00490C20 8D4C240C

    lea ecx, dword ptr [esp+0C]

:00490C24 8BF0

              mov esi, eax

:00490C26 C784242C180000FFFFFFFF

mov dword ptr [esp+0000182C], FFFFFFFF

:00490C31 E830AB0400

          call 004DB766

:00490C36 8B8C2424180000

        mov ecx, dword ptr [esp+00001824]

:00490C3D 8BC6

                  mov eax, esi

:00490C3F 5E

  pop esi

:00490C40 5B

        pop ebx

:00490C41 64890D00000000

      mov dword ptr fs:[00000000], ecx

:00490C48 81C428180000

          add esp, 00001828

:00490C4E C3

                  ret

    |

            |

                    |

    |

\* Referenced by a CALL at Address:

 |:00490C18

 |

\* Possible Reference to String Resource ID=00255: "\*~0SM俟y??&?

            |

:004A3830 6AFF

              push FFFFFFFF

:004A3832

686D5B5100              push 00515B6D

:004A3837 64A100000000            mov eax, dword

ptr fs:[00000000]

:004A383D 50

        push eaxp

                    |

    |

            |

                    |

 \*

Possible Reference to String Resource ID=00008: "w?(桶/?CuteFTP KH"

        |

:004A393A B908000000

      mov ecx, 00000008

:004A393F 8D7C2430

          lea edi, dword ptr [esp+30]

:004A3943

8D742470                lea esi, dword

ptr [esp+70]

:004A3947 33C0

      xor eax, eax

:004A3949 F3

              repz

:004A394A A7

                    cmpsd

:004A394B 5F

  pop edi

:004A394C 5E

        pop esi

:004A394D 745E

            je 004A39AD

      一定要跳              修改处5

:004A394F 8D4C2418

lea ecx, dword ptr [esp+18]

还是在:`004B72BA`中断，F10跟踪到如下部分

\* Possible StringData Ref from Data Obj ->"reg"

        |

:00491EBB 68B4625500

      push 005562B4

:00491EC0 52

              push edx

:00491EC1 E8C3060300

            call 004C2589

:00491EC6 83C408

                add esp, 00000008

:00491EC9 85C0

test eax, eax

:00491ECB 0F85FD010000
