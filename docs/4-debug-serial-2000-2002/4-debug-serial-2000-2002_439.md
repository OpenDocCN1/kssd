
# PolyView 3.41

> 原文：[`www.pediy.com/kssd/pediy03/forum501.htm`](https://www.pediy.com/kssd/pediy03/forum501.htm)

PolyView 再破解

\---------------------------------------------------------------------

软件名称：`PolyView Version 3.41`

软件简介：看图工具，具有简单的图像处理功能。

软件下载：`http://www.polybytes.com`

破解人：阿郎

级别：处女（大笨鸟）

本人初学，如有不正确之处，请来信告知！

\---------------------------------------------------------------------

前两天破了个 `3.00 b5` 版，当时俺就说了版本有些老，用来学习就没关系啦。今日撒（上）网拉下一个 `3.41` 版，应该是最新版本吧。何不再试试身手，它升我也升，飞刀就是这样炼成的吗？Y! 废话少说。开工>>>>

使用工具：`w32dsm`、`UltraEdit`

首先还是老办法打开 `PolyView` 在注册处添入用户名贺密码，点击确定。这时跳出提示窗，请记住提示语。

再打开 `w32dsam`，装入 `PolyView`，查找错误提示语，查找结果如下：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00460B73(C)  //注意此处

|^^^^^^^^^^

* Possible Reference to String Resource ID=00141: "Unregistered"

                          |

:00460C08 688D000000                push 0000008D
:00460C0D 8BCF                      mov ecx, edi
:00460C0F E8655D0900                call 004F6979
:00460C14 53                        push ebx
:00460C15 53                        push ebx

* Possible StringData Ref from Data Obj ->"Registration unsuccessful. Please "
                                     ->"verify that you have entered the "
                                     ->"information exactly as shown on "
                                     ->"your registration letter."
  //错误提示处

                          |

:00460C16 6870755600                push 00567570
:00460C1B 899E04030000              mov dword ptr [esi+00000304], ebx
:00460C21 E8C8E10900                call 004FEDEE

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
```

看到 `00460B73(C)` 处了吧，老办法跳到 `00460B73` 处：

```
:00460B49 898608030000              mov dword ptr [esi+00000308], eax
:00460B4F 8D442478                  lea eax, dword ptr [esp+78]
:00460B53 50                        push eax
:00460B54 8BCF                      mov ecx, edi
:00460B56 E841500900                call 004F5B9C
:00460B5B 8B442474                  mov eax, dword ptr [esp+74]
:00460B5F 8B0F                      mov ecx, dword ptr [edi]
:00460B61 50                        push eax
:00460B62 51                        push ecx
:00460B63 898604030000              mov dword ptr [esi+00000304], eax
:00460B69 E82265FEFF                call 00447090
:00460B6E 83C408                    add esp, 00000008
:00460B71 85C0                      test eax, eax
:00460B73 0F848F000000              je 00460C08  //修改

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:00460B79 3BEB                      cmp ebp, ebx
:00460B7B 0F85A5000000              jne 00460C26
:00460B81 E835560900                call 004F61BB
:00460B86 3BC3                      cmp eax, ebx
:00460B88 740B                      je 00460B95
:00460B8A 8B10                      mov edx, dword ptr [eax]
:00460B8C 8BC8                      mov ecx, eax
:00460B8E FF5274                    call [edx+74]
:00460B91 8BF8                      mov edi, eax
:00460B93 EB02                      jmp 00460B97

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```

在 `00460B73` 处修改同 `3.00` 版本一样，但要吸取上次教训，万万不可拔腿走人，下面才是关键：

进入 `00460B69` 处：

```
* Referenced by a CALL at Addresses:

|:00407ECA  , :00408408  , :0040909A  , :00457ECB  , :0045D20F

|:0045DBC3  , :0045E7BF  , :00460AD5  , :00460B69  , :004625EB

|:00463204  , :00463609  , :004636B8  , :00483AB9  , :00483C71

|:0049FD98  , :0050F201

|

:00447090 64A100000000              mov eax, dword ptr fs:[00000000]

* Possible Reference to Menu: MenuID_00FF

                          |

* Possible Reference to Dialog: DialogID_011C, CONTROL_ID:00FF, ""

                          |

:00447096 6AFF                      push FFFFFFFF
:00447098 6841DE5100                push 0051DE41
:0044709D 50                        push eax
:0044709E 64892500000000            mov dword ptr fs:[00000000], esp
:004470A5 81EC24010000              sub esp, 00000124
:004470AB 53                        push ebx
:004470AC 55                        push ebp
:004470AD 56                        push esi
:004470AE 57                        push edi
:004470AF 8BBC2444010000            mov edi, dword ptr [esp+00000144]
:004470B6 68780B5700                push 00570B78
:004470BB 57                        push edi
:004470BC E89F020000                call 00447360
:004470C1 8B9C2450010000            mov ebx, dword ptr [esp+00000150]
:004470C8 83C408                    add esp, 00000008
:004470CB 3BC3                      cmp eax, ebx
:004470CD 0F84AD000000              je 00447180  //改

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Possible StringData Ref from Data Obj ->"1$1AA"

                          |

:004470D3 68A8565600                push 005656A8
:004470D8 57                        push edi
:004470D9 E882020000                call 00447360
:004470DE 83C408                    add esp, 00000008
:004470E1 3BC3                      cmp eax, ebx
:004470E3 0F8497000000              je 00447180  //改

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:004470E9 8D442418                  lea eax, dword ptr [esp+18]
:004470ED 50                        push eax
:004470EE E8EC6D0A00                call 004EDEDF
:004470F3 8B08                      mov ecx, dword ptr [eax]
:004470F5 33F6                      xor esi, esi
:004470F7 894C2424                  mov dword ptr [esp+24], ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```

看到 `004470CD` 和 `004470E3` 两处了？这两处检测之后写入注册表（不知俺说的对不对，若是说的不对请告诉案，谢谢！）所以一定要将 `je` 改成 `jmp`。

这回退出试试，输入你的中文姓名、数字号，再看看关于项。如何？你的大名是不是在上面。

ok，收工>>>>

阿郎

`langlirong@163.net`

`http://alang.shangdu.net`  `4dd466d0e1e0cc123afbe504da2462ca`
