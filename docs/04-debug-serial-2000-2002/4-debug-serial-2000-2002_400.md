
# Mouse Odometer v2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum355.htm`](https://www.pediy.com/kssd/pediy03/forum355.htm)

## Mouse Odometer v2.0 破解（入门）

**软件名称**：Mouse Odometer v2.0（30 天试用且鼠标到 5 英里后，要求注册）

**简介**：一个非常好玩的软件，它能记录你每天鼠标在屏幕上的跑的公里数，呵呵

**作者**：`2000yeah`

**级别**：卖菜的

**日期**：2000 年 10 月 1 日

**工具**：`W32dsm`

```
:00423BCD 90                   nop
:00423BCE 90                   nop
:00423BCF 90                   nop
:00423BD0 0400                 add al, 00
:00423BD2 0000                 add byte ptr [eax], al
:00423BD4 90                   nop
:00423BD5 000C00               add byte ptr [eax+eax], cl
:00423BD8 94                   xchg eax,esp
:00423BD9 3D42005452           cmp eax, 52540042
:00423BDE 65675F               pop edi
:00423BE1 4E                   dec esi
:00423BE2 756D                 jne 00423C51
:00423BE4 202A                 and byte ptr [edx], ch
:00423BE6 0090558BEC83         add byte ptr [eax+83EC8B55], dl
:00423BEC C4D4                 les edx, esp
:00423BEE 53                   push ebx
:00423BEF 8BD8                 mov ebx, eax
:00423BF1 B8F8734A00           mov eax, 004A73F8
:00423BF6 E8B5820700           call 0049BEB0
:00423BFB 66C745E40800         mov [ebp-1C], 0008
:00423C01 33D2                 xor edx, edx
:00423C03 8955FC               mov dword ptr [ebp-04], edx
:00423C06 8D55FC               lea edx, dword ptr [ebp-04]
:00423C09 FF45F0               inc [ebp-10]
:00423C0C 8B83C4020000         mov eax, dword ptr [ebx+000002C4]
:00423C12 E809210400           call 00465D20
:00423C17 8D4DFC               lea ecx, dword ptr [ebp-04]
:00423C1A 51                   push ecx

* Possible StringData Ref from Data Obj ->"277130a" ---------------“注册码”

                                  |

:00423C1B BAF0724A00           mov edx, 004A72F0
:00423C20 8D45F8               lea eax, dword ptr [ebp-08]
:00423C23 E814840700           call 0049C03C
:00423C28 FF45F0               inc [ebp-10]
:00423C2B 8D55F8               lea edx, dword ptr [ebp-08]
:00423C2E 58                   pop eax
:00423C2F E8F0850700           call 0049C224
:00423C34 50                   push eax
:00423C35 FF4DF0               dec [ebp-10]
:00423C38 8D45F8               lea eax, dword ptr [ebp-08]
:00423C3B BA02000000           mov edx, 00000002
:00423C40 E8FB840700           call 0049C140
:00423C45 FF4DF0               dec [ebp-10]
:00423C48 8D45FC               lea eax, dword ptr [ebp-04]
:00423C4B BA02000000           mov edx, 00000002
:00423C50 E8EB840700           call 0049C140
:00423C55 59                   pop ecx
:00423C56 84C9                 test cl, cl
:00423C58 7467                 je 00423CC1 ----------------"一跳就死了”－－－－把 `je` 改为 `jne`
:00423C5A A1B0AC4A00           mov eax, dword ptr [004AACB0]
:00423C5F 6A00                 push 00000000

* Possible StringData Ref from Data Obj ->"Mouse Odometer"

                                  |

:00423C61 B913734A00           mov ecx, 004A7313

* Possible StringData Ref from Data Obj ->"Thank you for registering!" －－－－－－“注册搞定”

                                  |

:00423C66 BAF8724A00           mov edx, 004A72F8
:00423C6B 8B00                 mov eax, dword ptr [eax]
:00423C6D E81E830700           call 0049BF90
:00423C72 E84159FEFF           call 004095B8

* Reference To: `MyTimer._Form2`

............

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00423C8E(C)

|

:00423CAE 8BC3                 mov eax, ebx
:00423CB0 E843840300           call 0045C0F8
:00423CB5 8B55D4               mov edx, dword ptr [ebp-2C]
:00423CB8 64891500000000       mov dword ptr fs:[00000000], edx
:00423CBF EB3F                 jmp 00423D00

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00423C58(C)

|

:00423CC1 8B83C4020000         mov eax, dword ptr [ebx+000002C4]
:00423CC7 8B10                 mov edx, dword ptr [eax]
:00423CC9 FF5250               call [edx+50]
:00423CCC 3C01                 cmp al, 01        －－－－－－－－－－1 为注册
:00423CCE 7526                 jne 00423CF6
:00423CD0 A1B0AC4A00           mov eax, dword ptr [004AACB0]
:00423CD5 6A00                 push 00000000

* Possible StringData Ref from Data Obj ->"Mouse Odometer"

                                  |

:00423CD7 B962734A00           mov ecx, 004A7362

* Possible StringData Ref from Data Obj ->"The registration code you entered " “死翘翘..."
                                       ->"is invalid, please try again."

                                  |

:00423CDC BA22734A00           mov edx, 004A7322
:00423CE1 8B00                 mov eax, dword ptr [eax]
:00423CE3 E8A8820700           call 0049BF90
:00423CE8 8B83C4020000         mov eax, dword ptr [ebx+000002C4]
:00423CEE 8B10                 mov edx, dword ptr [eax]
:00423CF0 FF92B4000000         call dword ptr [edx+000000B4]
```

－－－－－－－－－－－－－－初学破解，错误难免，请各位大侠指点，谢谢。－－－－－－2000.10.1－`2000yeah`－－－

**注册码**：`277130a`   `4dd466d0e1e0cc123afbe504da2462ca`
