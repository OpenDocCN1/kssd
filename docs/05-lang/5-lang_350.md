
# OverNimble Localize Plus 1.04

> 原文：[`www.pediy.com/kssd/pediy05/pediy50012.htm`](https://www.pediy.com/kssd/pediy05/pediy50012.htm)

软件名称：OverNimble Localize Plus 1.04
中文名称：字串替换器 1.04

软件说明：该软件是一个本地化工具，主要用于非资源格式的本地化工作，支持的种类包括非资源格式的 C 编译的程序中的 ASCII 字符串和 UniCode 字符串、非资源格式的 Delphi（C++ Builder）编译的程序的字符串、VB 编译的程序的字符串、文本格式的字符串等的提取及替换。同时它还拥有方便的版本升级功能、字典处理功能，使您在翻译新版本时事半功倍。

破解撰写：leeyam

首先用 `TRW2000` 脱壳

载入 `LocPlus.exe`，`F10` 慢慢走，见到有向上的跳就按 `F7` 闪过去，经过一段崎岖的道路，我们来到这里：

```
:00443708 7407                  je 00443711

:0044370A 8903                  mov dword ptr [ebx], eax

:0044370C 83C304               add ebx, 00000004

:0044370F EBD8                  jmp 004436E9

:00443711 FF96443D0400          call dword ptr [esi+00043D44]

:00443717 61                    popad

:00443718 E9CBEFFBFF           jmp 004026E8…………………………看到入口，心中一阵喜悦！

:0044371D 00000000000000000000  BYTE 10 DUP(0)

:00443727 00000000000000000000  BYTE 10 DUP(0)

:00443731 00000000000000000000  BYTE 10 DUP(0)

:0044373B 00000000000000000000  BYTE 10 DUP(0)

:00443745 00000000000000000000  BYTE 10 DUP(0)

:0044374F 00000000000000000000  BYTE 10 DUP(0)

:00443759 00000000000000000000  BYTE 10 DUP(0)
```

于是下 `g 4026e8` 到：

```
:004026E8 688C584000             push 0040588C…………………………到了，终于进来了！

:004026ED E8F0FFFFFF             Call 004026E2

:004026F2 000000000000           BYTE  6 DUP(0)

:004026F8 3000                   xor byte ptr [eax], al

:004026FA 0000                   add byte ptr [eax], al

:004026FC 48                     dec eax

:004026FD 00000000000000         BYTE  7 DUP(0)
```

光标停在 `004026E8` 处马上下 `makepe` `路径\保存文件名.exe` 脱壳成功！试试运行没问题！下面开始破解。

`VB` 的程序用 `W32Dasm` 看不到中文字串，该软件是多语言版，注册提示是 `e` 文的，呵呵方便多了！

看看字串发现注册码正确的提示信息 `"TThank you register "` 双击进入：

```
* Possible StringData Ref from Data Obj ->"TThank you register "

                                  |

:0042F22B 6838E14000             push 0040E138

:0042F230 50                    push eax

:0042F231 FFD7                   call edi

:0042F233 8BD0                   mov edx, eax

:0042F235 8D4DDC                 lea ecx, dword ptr [ebp-24]
```

向下看发现注册码错误的提示信息：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F1B9(C)…………………………从这里跳过来的！

|

:0042F2E3 8D5588                 lea edx, dword ptr [ebp-78]

:0042F2E6 8D4DC8                 lea ecx, dword ptr [ebp-38]

* Possible StringData Ref from Data Obj ->"YYour RegCode is inefficacy, please "

                                  ->"re-input it!"

                                  |

:0042F2E9 C7459080E14000         mov [ebp-70], 0040E180

:0042F2F0 C7458808000000         mov [ebp-78], 00000008
```

想必该程序在 `0042F1B9` 的位置是判断关键点，上去看看：

```
* Reference To: MSVBVM50.__vbaFreeStr, Ord:0000h

                                  |

:0042F18B FF153C974300           Call dword ptr [0043973C]

:0042F191 E81AF8FFFF             call 0042E9B0…………………………关键跳就在这个 Call 的下面，这个 Call 当然是最可疑的，F8 进入看看。

:0042F196 66391D2C614300         cmp word ptr [0043612C], bx

:0042F19D B904000280             mov ecx, 80020004

:0042F1A2 B80A000000             mov eax, 0000000A

:0042F1A7 894DA0                 mov dword ptr [ebp-60], ecx

:0042F1AA 894598                 mov dword ptr [ebp-68], eax

:0042F1AD 894DB0                 mov dword ptr [ebp-50], ecx

:0042F1B0 8945A8                 mov dword ptr [ebp-58], eax

:0042F1B3 894DC0                 mov dword ptr [ebp-40], ecx

:0042F1B6 8945B8                 mov dword ptr [ebp-48], eax

:0042F1B9 0F8424010000           je 0042F2E3…………………………这里是关键跳！

:0042F1BF 391D706D4300           cmp dword ptr [00436D70], ebx

:0042F1C5 7510                   jne 0042F1D7

:0042F1C7 68706D4300             push 00436D70

:0042F1CC 68C8C54000             push 0040C5C8
```

这是上面的 `Call`：

```
* Referenced by a CALL at Addresses:

|:00413AD6  , :0042F191

|

:0042E9B0 55                     push ebp

:0042E9B1 8BEC                   mov ebp, esp

:0042E9B3 83EC14                 sub esp, 00000014

* Possible StringData Ref from Data Obj ->"％%饡 C"

                                  |

:0042E9B6 68C6214000             push 004021C6

:0042E9BB 64A100000000           mov eax, dword ptr fs:[00000000]

:0042E9C1 50                    push eax

:0042E9C2 64892500000000         mov dword ptr fs:[00000000], esp

:0042E9C9 83EC44                 sub esp, 00000044

:0042E9CC 53                    push ebx

:0042E9CD 56                    push esi

:0042E9CE 57                    push edi

:0042E9CF 8965EC                 mov dword ptr [ebp-14], esp

* Possible StringData Ref from Data Obj ->""

                                  |

:0042E9D2 C745F0781F4000         mov [ebp-10], 00401F78

:0042E9D9 33F6                   xor esi, esi

:0042E9DB 8975F4                 mov dword ptr [ebp-0C], esi

:0042E9DE 8975F8                 mov dword ptr [ebp-08], esi

:0042E9E1 8975DC                 mov dword ptr [ebp-24], esi

:0042E9E4 8975D8                 mov dword ptr [ebp-28], esi

:0042E9E7 8975C8                 mov dword ptr [ebp-38], esi

:0042E9EA 8975B8                 mov dword ptr [ebp-48], esi

* Possible Reference to String Resource ID=00001: "ChineseTraditional"

                                  |

:0042E9ED 6A01                   push 00000001

* Reference To: MSVBVM50.__vbaOnError, Ord:0000h

                                  |

:0042E9EF FF15E4944300           Call dword ptr [004394E4]

:0042E9F5 6689352C614300         mov word ptr [0043612C], si

* Possible StringData Ref from Data Obj ->"HHKEY_LOCAL_MACHINE\Software\OverNimble\LocPlu"

                                  ->"s\RegUserName"…………………………用户名

                                  |

:0042E9FC BAC0B64000             mov edx, 0040B6C0

:0042EA01 8D4DD8                 lea ecx, dword ptr [ebp-28]

* Reference To: MSVBVM50.__vbaStrCopy, Ord:0000h

                                  |

:0042EA04 8B3D7C964300           mov edi, dword ptr [0043967C]

:0042EA0A FFD7                   call edi

:0042EA0C 8D45D8                 lea eax, dword ptr [ebp-28]

:0042EA0F 50                    push eax

:0042EA10 8D4DC8                 lea ecx, dword ptr [ebp-38]

:0042EA13 51                    push ecx

:0042EA14 E887100000             call 0042FAA0

:0042EA19 8D55C8                 lea edx, dword ptr [ebp-38]

:0042EA1C 52                    push edx

* Reference To: MSVBVM50.__vbaStrVarMove, Ord:0000h

                                  |

:0042EA1D 8B1D24944300           mov ebx, dword ptr [00439424]

:0042EA23 FFD3                   call ebx

:0042EA25 8BD0                   mov edx, eax

:0042EA27 B928614300             mov ecx, 00436128

* Reference To: MSVBVM50.__vbaStrMove, Ord:0000h

                                  |

:0042EA2C 8B35FC964300           mov esi, dword ptr [004396FC]

:0042EA32 FFD6                   call esi

:0042EA34 8D4DD8                 lea ecx, dword ptr [ebp-28]

* Reference To: MSVBVM50.__vbaFreeStr, Ord:0000h

                                  |

:0042EA37 FF153C974300           Call dword ptr [0043973C]

:0042EA3D 8D4DC8                 lea ecx, dword ptr [ebp-38]

* Reference To: MSVBVM50.__vbaFreeVar, Ord:0000h

                                  |

:0042EA40 FF1518944300           Call dword ptr [00439418]

* Possible StringData Ref from Data Obj ->"HHKEY_LOCAL_MACHINE\Software\OverNimble\LocPlu"
```



`->"s\RegCode"`…………………………注册码，要警惕下面的每一个动作！

|

```
:0042EA46 BA58B74000           mov edx, 0040B758

:0042EA4B 8D4DD8              lea ecx, dword ptr [ebp-28]

:0042EA4E FFD7                call edi

:0042EA50 8D45D8              lea eax, dword ptr [ebp-28]

:0042EA53 50                  push eax

:0042EA54 8D4DC8              lea ecx, dword ptr [ebp-38]

:0042EA57 51                  push ecx

:0042EA58 E843100000          call 0042FAA0

:0042EA5D 8D55C8              lea edx, dword ptr [ebp-38]

:0042EA60 52                  push edx

:0042EA61 FFD3                call ebx

:0042EA63 8BD0                mov edx, eax

:0042EA65 8D4DDC              lea ecx, dword ptr [ebp-24]

:0042EA68 FFD6                call esi

:0042EA6A 8D4DD8              lea ecx, dword ptr [ebp-28]
```

* Reference To: `MSVBVM50.__vbaFreeStr`, Ord:`0000h`

|

```
:0042EA6D FF153C974300        Call dword ptr [0043973C]

:0042EA73 8D4DC8              lea ecx, dword ptr [ebp-38]
```

* Reference To: `MSVBVM50.__vbaFreeVar`, Ord:`0000h`

|

```
:0042EA76 FF1518944300        Call dword ptr [00439418]

:0042EA7C 8D45DC              lea eax, dword ptr [ebp-24]

:0042EA7F 50                  push eax

:0042EA80 E8DBFAFFFF          call 0042E560

:0042EA85 8BD0                mov edx, eax

:0042EA87 8D4DDC              lea ecx, dword ptr [ebp-24]

:0042EA8A FFD6                call esi

:0042EA8C 8B4DDC              mov ecx, dword ptr [ebp-24]

:0042EA8F 51                  push ecx
```

* Reference To: `MSVBVM50.__vbaLenBstr`, Ord:`0000h`

|

```
:0042EA90 FF151C944300        Call dword ptr [0043941C]

:0042EA96 83F80C              cmp eax, 0000000C…………………………判断注册码位数

:0042EA99 756A                jne 0042EB05…………………………不能跳！

:0042EA9B 8D55DC              lea edx, dword ptr [ebp-24]

:0042EA9E 8955C0              mov dword ptr [ebp-40], edx

:0042EAA1 C745B808400000      mov [ebp-48], 00004008

:0042EAA8 8D45B8              lea eax, dword ptr [ebp-48]

:0042EAAB 50                  push eax
```

* Reference To: `MSVBVM50.rtcIsNumeric`, Ord:`0000h`

|

```
:0042EAAC FF1578954300        Call dword ptr [00439578]

:0042EAB2 6685C0              test ax, ax

:0042EAB5 744E                je 0042EB05…………………………不跳！

:0042EAB7 8B4DDC              mov ecx, dword ptr [ebp-24]

:0042EABA 51                  push ecx
```

* Reference To: `MSVBVM50.rtcR8ValFromBstr`, Ord:`0000h`

|

```
:0042EABB FF1540974300        Call dword ptr [00439740]
```

* Reference To: `MSVBVM50.__vbaFpI4`, Ord:`0000h`

|

```
:0042EAC1 FF15E4964300        Call dword ptr [004396E4]

:0042EAC7 8945E0              mov dword ptr [ebp-20], eax

:0042EACA 83F801              cmp eax, 00000001

:0042EACD 7C36                jl 0042EB05…………………………俺的水平有限，看不懂，反正和上面的跳一样，不跳就对！

:0042EACF DB45E0              fild dword ptr [ebp-20]

:0042EAD2 DD5DA0              fstp qword ptr [ebp-60]

:0042EAD5 DD45A0              fld qword ptr [ebp-60]

:0042EAD8 DC1D501F4000        fcomp qword ptr [00401F50]

:0042EADE DFE0                fstsw ax

:0042EAE0 F6C441              test ah, 41

:0042EAE3 7420                je 0042EB05

:0042EAE5 66C7052C614300FFFF mov word ptr [0043612C], FFFF
```

* Reference To: `MSVBVM50.__vbaExitProc`, Ord:`0000h`

|

```
:0042EAEE FF15C8944300        Call dword ptr [004394C8]

:0042EAF4 9B                  wait

:0042EAF5 6830EB4200          push 0042EB30

:0042EAFA EB2A                jmp 0042EB26

:0042EAFC 66C7052C6143000000 mov word ptr [0043612C], 0000
```

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Addresses:

| `:0042EA99(C)`, `:0042EAB5(C)`, `:0042EACD(C)`, `:0042EAE3(C)`

|

* Reference To: `MSVBVM50.__vbaExitProc`, Ord:`0000h`

|

```
:0042EB05 FF15C8944300        Call dword ptr [004394C8]

:0042EB0B 9B                  wait

:0042EB0C 6830EB4200          push 0042EB30

:0042EB11 EB13                jmp 0042EB26

:0042EB13 8D4DD8              lea ecx, dword ptr [ebp-28]
```

* Reference To: `MSVBVM50.__vbaFreeStr`, Ord:`0000h`

|

```
:0042EB16 FF153C974300        Call dword ptr [0043973C]

:0042EB1C 8D4DC8              lea ecx, dword ptr [ebp-38]
```

* Reference To: `MSVBVM50.__vbaFreeVar`, Ord:`0000h`

|

```
:0042EB1F FF1518944300        Call dword ptr [00439418]

:0042EB25 C3                  ret
```

* Referenced by a `(U)nconditional` or `(C)onditional` Jump at Addresses:

| `:0042EAFA(U)`, `:0042EB11(U)`

|

```
:0042EB26 8D4DDC              lea ecx, dword ptr [ebp-24]
```

* Reference To: `MSVBVM50.__vbaFreeStr`, Ord:`0000h`

|

```
:0042EB29 FF253C974300        Jmp dword ptr [0043973C]

:0042EB2F C3                  ret

:0042EB30 8B4DE4              mov ecx, dword ptr [ebp-1C]

:0042EB33 64890D00000000      mov dword ptr fs:[00000000], ecx

:0042EB3A 5F                  pop edi

:0042EB3B 5E                  pop esi

:0042EB3C 5B                  pop ebx

:0042EB3D 8BE5                mov esp, ebp

:0042EB3F 5D                  pop ebp

:0042EB40 C3                  ret…………………………返回
```

现在用 `HIEW` `nop` 掉上面的三个跳，运行程序试试。随意输入用户名（注册码不填都可以）注册成功！

多多指教！

谢谢看完！ `4dd466d0e1e0cc123afbe504da2462ca`
