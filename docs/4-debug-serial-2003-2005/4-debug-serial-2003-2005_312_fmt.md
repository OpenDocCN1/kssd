
# XNview v1.65

> 原文：[`www.pediy.com/kssd/pediy06/pediy6361.htm`](https://www.pediy.com/kssd/pediy06/pediy6361.htm)

XNview v1.65 算法分析

## 【软件简介】
支持多达 70 种格式的图形浏览、转换、编辑软件，还可制作 Slide Show。是否嫌 ACDSEE 太大功能太少？这个软件能很好地解决问题，他具有抓图、编辑图象、增加特效的功能，支持你所知道的所有格式及你不知道的格式（包括电影、MP3）。支持简体中文语言。

## 【声 明】
破解旨在学习技术，无其它目的。失误之处敬请诸位大侠赐教!

## 【程 序 名】
`XNview.exe`

## 【版 本】
`v1.65` <-不知道它现在的版本是多少，管它的先看看再说。:)

## 【大 小】
`730KB`

## 【语 言】
Microsoft Visual C++ 6.0

## 【运行平台】
`W9x/NT/W2K/WXP`

## 【保护方式】
`ASPack` / `ASProtect` `x.xx` 壳/注册码

## 【分析方式】
追注册码/分析注册算法

## 【难 度】
简单

## 【工 具】
`AspackDie141`/`PEid0.9`/`W32Dasm8.93+`/`TRW2000` `v1.23`

## 【程序下载】
`http://www.xnview.com`

## 【作 者】
`xbb_NCG`

## 【分 析】

用 `PEid0.9` 查看程序被 `ASPack` 加壳，`AspackDie141` 脱壳(本想尝试手动脱壳，不过功能不够，还需要学习)。查看脱壳后的程序是用 Microsoft Visual C++ 6.0 编的，点击软件菜单 帮助->注册 会弹出一个注册对话框，输入 `xbb-NCG` 和假注册码 `123456789` 提示"非法注册"。于是用 `ResHacker3.4.0.79` 版打开它。在对话框资源中我们可以找到以 `NVIEWREGISTRATION` 名的对话框资源，点击它左边的+号，点击展开的 `1033` 项会显示一个一程序中一样的对话框，并且右边窗口中可以找到 `CAPTION"Registration"` 字样。OK，复制 `Registration`。用 `W32Dasm8.93+` 反汇编程序，打开串式参考，在 `Search` 栏粘贴 `Registration` 然后搜索,我们会发现第一个字符是`"Invalid registration"`（非法注册<---:)HOHO 就是它）。这个字串的中文含义和我们前面在程序中输入假注册码后的提示信息是一样的:)。现在双击它我们会看到如下代码。

如果我们要找出软件的注册算法的话就要在 `TRW2000` 中下 `bpx 466E40` 断点，因为在 `466E52` 下面我们可清楚地看到 `GetDlgItemTextA` 这个 API，很明显程序调用了它来获取注册对话框里的字符。当然直接下 `bpx getdlgitemtexta` 断点也行，随个人喜好。因为我要跟踪注册码的算法，所以从前面一步步地跟下来要好些。:)

OK，开工了(对了，不要忘了准备一些纸和笔，随时在纸上对一些语句做注释，这样不会退出 `TRW2000` 后就忘了，以后写手记也要靠它的。做为新手这点一定要做到，不要怕累，做多了，你会对一些汇编语句非常熟悉的，一看就知道是干什么的。遇到不懂的汇编语句就去看汇编的书找找，这样你会记得更牢些。这是我个人的一些做法，你不一定要像我这样。:)）

```
*******************************************************************

:00466E3C 90                 nop
:00466E3D 90                 nop
:00466E3E 90                 nop
:00466E3F 90                 nop
:00466E40 81EC68010000       sub esp, 00000168
:00466E46 8D442468           lea eax, dword ptr [esp+68]
:00466E4A 56                 push esi
:00466E4B 8BB42470010000     mov esi, dword ptr [esp+00000170]
:00466E52 57                 push edi
* Reference To: USER32.GetDlgItemTextA, Ord:0104h
                                |
:00466E53 8B3D40A55300       mov edi, dword ptr [0053A540]
* Possible Ref to Menu: BROWSERMENU, Item: "Create Panorama..."
                                |
:00466E59 6800010000         push 00000100
:00466E5E 50                 push eax
* Possible Reference to Dialog: NVIEWAUTOCROP, CONTROL_ID:07D0, ""
                                |
:00466E5F 68D0070000         push 000007D0
:00466E64 56                 push esi
:00466E65 FFD7               call edi
:00466E67 8D4C2410           lea ecx, dword ptr [esp+10]
* Possible Ref to Menu: NVIEWMENU, Item: "Crop    Ctrl+Y"
                                |
:00466E6B 6A20               push 00000020
:00466E6D 51                 push ecx
* Possible Reference to Dialog: NVIEWAUTOCROP, CONTROL_ID:07D1, ""
                                |
:00466E6E 68D1070000         push 000007D1
:00466E73 56                 push esi
:00466E74 FFD7               call edi
:00466E76 8A442470           mov al, byte ptr [esp+70]   <-检测用户名是否为 0
:00466E7A 84C0               test al, al
:00466E7C 0F843A010000       je 00466FBC                <-等于 0 则跳
:00466E82 8A442410           mov al, byte ptr [esp+10]   <-检测注册码是否为 0
:00466E86 84C0               test al, al
:00466E88 0F842E010000       je 00466FBC                <-等于 0 测跳
:00466E8E 8D542408           lea edx, dword ptr [esp+08]
:00466E92 8D442470           lea eax, dword ptr [esp+70]
:00466E96 52                 push edx
:00466E97 50                 push eax
:00466E98 E8A340FAFF         call 0040AF40                <-注册码计算 CALL，后面跟入分析...
:00466E9D 8D4C2418           lea ecx, dword ptr [esp+18]    <-ECX=假注册码
:00466EA1 51                 push ecx
:00466EA2 E8BC9F0000         call 00470E63                <-将假注册码转换成 16 进制数
:00466EA7 8B4C2414           mov ecx, dword ptr [esp+14]    <-ECX 为真注册码的 16 进制，在此处键入? ecx 即可见到真注册码
:00466EAB 83C40C             add esp, 0000000C              问题：为什么用？而不用 D。答：因为注册码是以 16 进制显示的。:)
:00466EAE 3BC8               cmp ecx, eax                   <-真假注册码比较
:00466EB0 745D               je 00466F0F                    <-真假注册码相等则跳到注册成功处
:00466EB2 A1288C5800         mov eax, dword ptr [00588C28]
:00466EB7 8D542430           lea edx, dword ptr [esp+30]
* Possible Ref to Menu: NVIEWMENU, Item: "16 Grey scale (Dither)"
                                |
:00466EBB 6A40               push 00000040
:00466EBD 52                 push edx
* Possible Reference to String Resource ID=05011: "Invalid registration"  <-双击后我们来到这里
                                |
:00466EBE 6893130000         push 00001393
:00466EC3 50                 push eax
* Reference To: USER32.LoadStringA, Ord:01ABh
                                |
:00466EC4 FF15F8A55300       Call dword ptr [0053A5F8]
* Possible Ref to Menu: NVIEWDEFAULTME, Item: "Slide Show...   Ctrl+L"
                                |
:00466ECA 6A10               push 00000010
:00466ECC 8D4C2434           lea ecx, dword ptr [esp+34]
:00466ED0 689C325800         push 0058329C
:00466ED5 51                 push ecx
:00466ED6 56                 push esi
* Reference To: USER32.MessageBoxA, Ord:01BEh
                                |
:00466ED7 FF157CA55300       Call dword ptr [0053A57C]        <-非法注册提示
* Possible Reference to Dialog: NVIEWAUTOCROP, CONTROL_ID:07D0, ""
                                |
:00466EDD 68D0070000         push 000007D0
:00466EE2 56                 push esi
* Reference To: USER32.GetDlgItem, Ord:0102h
                                |
:00466EE3 FF151CA65300       Call dword ptr [0053A61C]
:00466EE9 50                 push eax
* Reference To: USER32.SetFocus, Ord:022Fh
                                |
:00466EEA FF1558A55300       Call dword ptr [0053A558]
:00466EF0 689C325800         push 0058329C
* Possible Reference to Dialog: NVIEWAUTOCROP, CONTROL_ID:07D1, ""
                                |
:00466EF5 68D1070000         push 000007D1
:00466EFA 56                 push esi
* Reference To: USER32.SetDlgItemTextA, Ord:022Ch
                                |
:00466EFB FF15FCA55300       Call dword ptr [0053A5FC]
:00466F01 5F                 pop edi
* Possible Ref to Menu: BROWSERMENU, Item: "Open..."
                                |
:00466F02 B801000000         mov eax, 00000001
:00466F07 5E                 pop esi
:00466F08 81C468010000       add esp, 00000168
:00466F0E C3                 ret
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00466EB0(C)                                 <--------注册码比对正确后跳来此处
|
:00466F0F 8D542470           lea edx, dword ptr [esp+70]
* Possible Ref to Menu: BROWSERMENU, Item: "Create Panorama..."
                                |
:00466F13 6800010000         push 00000100
:00466F18 52                 push edx
* Possible Reference to Dialog: NVIEWAUTOCROP, CONTROL_ID:07D0, ""
                                |
:00466F19 68D0070000         push 000007D0
:00466F1E 56                 push esi
:00466F1F FFD7               call edi
```



```
:00466F21 8D442410             lea eax, dword ptr [esp+10]

* Possible Ref to Menu: NVIEWMENU, Item: "Crop   Ctrl+Y"

                                 |

:00466F25 6A20                 push 00000020

:00466F27 50                   push eax

* Possible Reference to Dialog: NVIEWAUTOCROP, CONTROL\_ID:07D1, ""

                                 |

:00466F28 68D1070000           push 000007D1

:00466F2D 56                   push esi

:00466F2E FFD7                 call edi

:00466F30 8D4C2470             lea ecx, dword ptr [esp+70]

:00466F34 51                   push ecx

* Possible StringData Ref from Data Obj ->"LicenseName"

                                 |

:00466F35 6828365600           push 00563628

:00466F3A 6A00                 push 00000000

:00466F3C E8FF5DFDFF           call 0043CD40           <-用户名写入注册表

:00466F41 8D54241C             lea edx, dword ptr [esp+1C]

:00466F45 52                   push edx

* Possible StringData Ref from Data Obj ->"LicenseNumber"

                                 |

:00466F46 6818365600           push 00563618

:00466F4B 6A00                 push 00000000

:00466F4D E8EE5DFDFF           call 0043CD40           <-注册码写入注册表

:00466F52 A12C8C5800           mov eax, dword ptr [00588C2C]

:00466F57 83C418               add esp, 00000018      问题：你怎么知道上面是写注册表？答：因为我用 `Regmon` 这个软件

                                                       对软件启动时的动作进行了监测的。:)

* Possible Ref to Menu: BROWSERMENU, Item: "Open..."

                                 |

:00466F5A C705488C580001000000    mov dword ptr [00588C48], 00000001

* Possible Ref to Menu: BROWSERMENU, Item: "Open..."

                                 |

:00466F64 6A01                 push 00000001

* Possible Ref to Menu: BROWSERMENU, Item: "Registration"

                                 |

:00466F66 68DE000000           push 000000DE

:00466F6B 50                   push eax

* Reference To: USER32.GetMenu, Ord:011Ch

                                 |

:00466F6C FF154CA55300         Call dword ptr [0053A54C]

:00466F72 50                   push eax

* Reference To: USER32.EnableMenuItem, Ord:00B5h

                                 |

:00466F73 FF1530A55300         Call dword ptr [0053A530]    <-将帮助菜单中注册选项禁用(显示为灰色)

:00466F79 8B15288C5800         mov edx, dword ptr [00588C28]    问题：你怎么知道这里是对菜单的禁用？答：因为 `EnableMenuItem`，

:00466F7F 8D4C2430             lea ecx, dword ptr [esp+30]        这在编程中是对一个控件的可用或不可能进行控制的，当它的

                                                       为 TRUE 时，可用；为 FALSE 时，则不可用。:)

* Possible Ref to Menu: NVIEWMENU, Item: "16 Grey scale (Dither)"

                                 |

:00466F83 6A40                 push 00000040

:00466F85 51                   push ecx

* Possible Reference to String Resource ID=05012: "Registration succesfull"

                                 |

:00466F86 6894130000           push 00001394

:00466F8B 52                   push edx

* Reference To: USER32.LoadStringA, Ord:01ABh

                                 |

:00466F8C FF15F8A55300         Call dword ptr [0053A5F8]

* Possible Ref to Menu: NVIEWMENU, Item: "16 Grey scale (Dither)"

                                 |

:00466F92 6A40                 push 00000040

:00466F94 8D442434             lea eax, dword ptr [esp+34]

:00466F98 689C325800           push 0058329C

:00466F9D 50                   push eax

:00466F9E 56                   push esi

* Reference To: USER32.MessageBoxA, Ord:01BEh

                                 |

:00466F9F FF157CA55300         Call dword ptr [0053A57C]    <-注册成功提示

:00466FA5 6A00                 push 00000000

:00466FA7 56                   push esi

* Reference To: USER32.EndDialog, Ord:00B9h

                                 |

:00466FA8 FF15F4A55300         Call dword ptr [0053A5F4]

:00466FAE 5F                   pop edi

* Possible Ref to Menu: BROWSERMENU, Item: "Open..."

                                 |

:00466FAF B801000000           mov eax, 00000001

:00466FB4 5E                   pop esi

:00466FB5 81C468010000         add esp, 00000168

:00466FBB C3                   ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00466E7C(C), :00466E88(C)                                      <-用户名和注册码为空跳到这里

|

:00466FBC 8B15288C5800         mov edx, dword ptr [00588C28]

:00466FC2 8D4C2430             lea ecx, dword ptr [esp+30]

* Possible Ref to Menu: NVIEWMENU, Item: "16 Grey scale (Dither)"

                                 |

:00466FC6 6A40                 push 00000040

:00466FC8 51                   push ecx

* Possible Reference to String Resource ID=05011: "Invalid registration"

                                 |

:00466FC9 6893130000           push 00001393

:00466FCE 52                   push edx

* Reference To: USER32.LoadStringA, Ord:01ABh

                                 |

:00466FCF FF15F8A55300         Call dword ptr [0053A5F8]

* Possible Ref to Menu: NVIEWDEFAULTME, Item: "Slide Show...   Ctrl+L"

                                 |

:00466FD5 6A10                 push 00000010

:00466FD7 8D442434             lea eax, dword ptr [esp+34]

:00466FDB 689C325800           push 0058329C

:00466FE0 50                   push eax

:00466FE1 56                   push esi

* Reference To: USER32.MessageBoxA, Ord:01BEh

                                 |

:00466FE2 FF157CA55300         Call dword ptr [0053A57C]      <-非法注册提示

:00466FE8 5F                   pop edi

* Possible Ref to Menu: BROWSERMENU, Item: "Open..."

                                 |

:00466FE9 B801000000           mov eax, 00000001

:00466FEE 5E                   pop esi

:00466FEF 81C468010000         add esp, 00000168

:00466FF5 C3                   ret

******************************************************************************

    跟入注册码算法 `call 0040AF40`...

.............

* Referenced by a CALL at Addresses:

|:0040B0FC   , :00466E98       <-程序有两处调用此 CALL 检查注册码的正确性，40B0FC 显然为程序启动时的调用。

|

:0040AF40 8B542404             mov edx, dword ptr [esp+04]        <-EDX 为用户名

:0040AF44 53                   push ebx

:0040AF45 55                   push ebp

:0040AF46 56                   push esi

:0040AF47 57                   push edi

:0040AF48 8BFA                 mov edi, edx                        <-EDI=用户名

:0040AF4A 83C9FF               or ecx, FFFFFFFF

:0040AF4D 33C0                 xor eax, eax                        <-EAX=0

:0040AF4F F2                   repnz                                 <\_ 取用户名

:0040AF50 AE                   scasb                                 的位数

:0040AF51 F7D1                 not ecx

:0040AF53 49                   dec ecx

* Possible StringData Ref from Data Obj ->"獕宁 Fx 鹦琪绻?蒔 s"

                                 |

:0040AF54 BE80355600           mov esi, 00563580                   <-字符表道地址入 ESI

:0040AF59 8BE9                 mov ebp, ecx                        <-用户名位数入 EBP

* Possible Ref to Menu: NVIEWMENU, Item: "Reopen   Ctrl+R"

                                 |

:0040AF5B B905000000           mov ecx, 00000005                   <-ECX=5
```



`mov edi, 005832A8` <-EDI=5832A8 这个地址

`repz` <\_ 将 563580 处的字符表移到 `movsd` < 5832A8 处，共 20 位

`mov esi, eax` <-ESI=0

`je 0040AF8C` +---------------------+ |字符表： |

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |4347622D4E 78F0D003E7|

|:0040AF8A(C) |F7FDF4E7B9 B51BC95073|

+---------------------+

`mov cl, byte ptr [esi+edx]`---- <-CL 为用户名的 ASCII 码

`mov bl, cl`

`xor bl, byte ptr [eax+005832A8]` <-BL 与 EAX+5832A8(字符表 1-5 位)进行异或运算

`inc eax` <-EAX+1

`cmp eax, 00000005` <-EAX=5 吗？ |此循环将用户名逐位与字符表中 1-5

`mov byte ptr [esi+edx], bl` |位进行异或运算。循环算出的结果

`mov byte ptr [eax+005832A7], cl` |放入 ESI+EDX 中。

`jne 0040AF87` <-EAX 不等于 5 则跳 |

`xor eax, eax` <-EAX=5 则 EAX=0 |结果：D2EBA6D3083B25 <---中间数 1

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |第一次循环

|:0040AF83(C) / /

`inc esi` <-计数器 /

`cmp esi, ebp` / <-计数器是否等于用户名位数

`jb 0040AF6B`_____/

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040AF69(C) |

`xor edi, edi`

`xor ecx, ecx`

`test ebp, ebp`

`jbe 0040AFBA`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040AFB8(C) |

`mov bl, byte ptr [edi+005832AD]`- <-BL 指向字符表第 6 位

`mov esi, ebp` <-ESI=用户名位数

`sub esi, ecx`

`dec esi` <-ESI-1 |

`mov al, byte ptr [esi+edx]` |<-ESI+EDX=中间数 1

`xor bl, al` |此循环将第一次循环的结果从右向左依次

`inc edi` <-计数器 |与字符表中 6-10 位进行异或运算，结果保

`mov byte ptr [esi+edx], bl` |存在 ESI+EDX 中。

`mov byte ptr [edi+005832AC], al` |

`cmp edi, 00000005` <-EDI 与 5 比较 |结果：E9CE41D0D8CB5D <---中间数 2

`jne 0040AFB5` |

`xor edi, edi` |

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |第二次循环

|:0040AFB1(C) / /

`inc ecx` /

`cmp ecx, ebp` / <-循环结束了吗？

`jb 0040AF94`_____/ <-没结束就跳

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040AF92(C) |

`xor esi, esi`

`xor edi, edi`

`test ebp, ebp`

`jbe 0040AFE3`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040AFE1(C) |

`mov al, byte ptr [edi+edx]`---- <-EDI+EDX=中间数 2

`mov cl, byte ptr [esi+005832B2]`

`xor cl, al` <-CL 与 AL 做异或运算

`inc esi`

`mov byte ptr [edi+edx], cl` |

`mov byte ptr [esi+005832B1], al` |

`cmp esi, 00000005` |此循环将第二次循环的结果从左向右依次

`jne 0040AFDE` |与字符表中 11-15 位进行异或运算，结果保

`xor esi, esi` |存在 EDI+EDX 中。

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |结果：1E33B537612293 <---中间数 3

|:0040AFDA(C) |第三次循环

/

`inc edi` / <-计数器

`cmp edi, ebp` /

`jb 0040AFC2`_________/

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040AFC0(C) |

`xor edi, edi`

`xor ecx, ecx`

`test ebp, ebp`

`jbe 0040B011`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040B00F(C) |

`mov bl, byte ptr [edi+005832B7]`

`mov esi, ebp`

`sub esi, ecx`

`dec esi`

`mov al, byte ptr [esi+edx]` | <-ESI+EDX=1E33B537612293

`xor bl, al` | <-BL 与 AL 异或

`inc edi` |此循环将第三次循环的结果从右向左依次

`mov byte ptr [esi+edx], bl` |与字符表中 16-20 位进行异或运算，结果保

`mov byte ptr [edi+005832B6], al` |存在 ESI+EDX 中。

`cmp edi, 00000005` |

`jne 0040B00C` |

`xor edi, edi` |结果：3CA0C667A83926 <---中间数 4

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |第四次循环

|:0040B008(C) / /

`inc ecx` / <-计数器

`cmp ecx, ebp` /

`jb 0040AFEB`_________/

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040AFE9(C) |



```
:0040B011 8B7C2418                mov edi, dword ptr [esp+18]         <-EDI=假注册码   

:0040B015 33C0                    xor eax, eax   

:0040B017 85ED                    test ebp, ebp   

:0040B019 C70700000000            mov dword ptr [edi], 00000000       <-将[EDI]所指向的地址按 DW 格式清零   

:0040B01F 7617                    jbe 0040B038   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040B036(C)   

|                                                                   第五次循环   

:0040B021 8BC8                    mov ecx, eax---------------   

:0040B023 83E103                  and ecx, 00000003                 <-ECX=4 时，EAX=0   

:0040B026 8A1C39                  mov bl, byte ptr [ecx+edi]    

:0040B029 8D3439                  lea esi, dword ptr [ecx+edi]   
    
:0040B02C 8A0C10                  mov cl, byte ptr [eax+edx]      |此循环将中间数 4 前 4 位依次放入 EDI 所指向的   

:0040B02F 02D9                    add bl, cl                       |地址中，第 5 位数继续放入地址的第一位，但   

:0040B031 40                      inc eax     <-EAX 加 1，计数器  /  要加上此地址中原来存在的数；第 6 位放入地址   

:0040B032 3BC5                    cmp eax, ebp                    /  的第二位并加上原来存在的数，依此类推，直   

:0040B034 881E                    mov byte ptr [esi], bl          /   至中间数取完。得到的 4 位数的十进制就是注册码。   

:0040B036 72E9                    jb 0040B021_________________/ 结果：E4D9EC67=67ECD9E4=174375524 <-注册码:)   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040B01F(C)   

|   

:0040B038 5F                      pop edi   

:0040B039 5E                      pop esi   

:0040B03A 5D                      pop ebp   

:0040B03B 5B                      pop ebx   

:0040B03C C3                      ret                      <-返回 ESI=67ECD9E4 到主程序   
```

---

### 【算法总结】

通过上面对算法 `CALL` 的逐句分析我们可以看出，软件是通过我们输入的用户名与给定的字符表中的字符进行四次循环计算后得出一个中间数，然后把这个中间数进行按 4 位拆分进行相加计算，即如果中间数是七位，则把第一位与第五位相加，结果放入第一位地址；第二位与第六位相加，结果放入第二位地址，依此类推，直到中间数取完为止。最后得到一个 4 位数（16 进制的），这个数由算法 `CALL` 返回到主程序，与我们输入的假注册码的 16 进制进行比对。正确就写注册表，错误就提示注册码错误。

另，用户名超过 10 位注册码会是个负数，不过一样注册码，没事的。:)

---

### 【注册信息】

用 `Regmon` 可以监测到软件在注册成功后在注册里写入注册信息：

`HKCUSoftwareXnViewLicenseName`  <-用户名"xbb-NCG"

`HKCUSoftwareXnViewLicenseNumber`  <-注册码"1743575524"

取注册注册只需把 `LicenseName` 和 `LicenseNumber` 两个键值删除即可。

---

### 【注册机】

我的编程非常差，虽然能找追算法，可还不够能力用代码还原它，正在学习 C，争取能在以后能写出注册机。

xbb-NCG
2003.12.15 上午

`4dd466d0e1e0cc123afbe504da2462ca`
