
# [原创]SPIP（Scanning Probe Image Proce

课题：SPIP（Scanning Probe Image Processor）个人版软件注册码学习

作者：HaIlDuZ

程序来源：c47K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2E0j5h3N6W2L8h3g2@1i4K6u0W2j5$3!0E0i4K6u0r3 丹麦，下载名称：`SPIPinstall.exe`

软件名称：SPIP 3.3.9.0 个人版 更新时间：2005.4

评价：该软件不错，用于扫描探针图像分析类软件。

使用工具：`OllyDbg` v1.09，`Ultraedit-32`，`W32Dasm`汉化修改版10.0

使用平台：winXP/98/2k（win98下需要`GDIplus.Dll`）

作者申明：纯技术交流，无任何商业目的，转贴请保持完整。

## 开始

运行显示演示版仅可128乘128像素操作，一段时间后变为64乘64。打开文件显示对话框警告。`W32Dasm`反汇编察看无警告字符信息，从对话框入手，用`OllyDbg`装载，跟踪最终观察到该软件采用keyfile方式，文件名：`SPIP.LIC`，到程序中打开其文件处如下：

```asm
:0049D4EF 68D8CB5A00              push 005ACBD8                       /\*注册文件名压栈，”spip.lic“\*/

:0049D4F4 8BC8                    mov ecx, eax

:0049D4F6 C68424A000000006        mov byte ptr [esp+000000A0], 06

:0049D4FE FF15C48E5900            Call dword ptr [00598EC4]

:0049D504 50                      push eax

:0049D505 E8B6D50600              call 0050AAC0

:0049D50A 83C408                  add esp, 00000008

:0049D50D 8D4C2424                lea ecx, dword ptr [esp+24]

:0049D511 A3FC686000              mov dword ptr [006068FC], eax

:0049D516 C684249C00000005        mov byte ptr [esp+0000009C], 05

:0049D51E FF15C88E5900            Call dword ptr [00598EC8]

:0049D524 A1FC686000              mov eax, dword ptr [006068FC]

:0049D529 6868DA5900              push 0059DA68

:0049D52E 50                      push eax

:0049D52F FF1530905900            Call dword ptr [00599030]             /\*注册文件打开读写方式\*/

:0049D535 8BF0                    mov esi, eax

:0049D537 83C408                  add esp, 00000008

:0049D53A 85F6                    test esi, esi

:0049D53C 0F8489000000            je 0049D5CB                           /\*无注册文件调到演示版\*/

:0049D542 8B0DFC686000            mov ecx, dword ptr [006068FC]

:0049D548 51                      push ecx

:0049D549 E8A2470600              call 00501CF0

:0049D54E A3F4686000              mov dword ptr [006068F4], eax

:0049D553 83C002                  add eax, 00000002

:0049D556 50                      push eax

\* Reference To: MFC71.Ordinal:0109, Ord:0109h

                                  |

:0049D557 E85E010E00              Call 0057D6BA

:0049D55C 8B0DF4686000            mov ecx, dword ptr [006068F4]

:0049D562 83C102                  add ecx, 00000002

:0049D565 8BD1                    mov edx, ecx

:0049D567 8BF8                    mov edi, eax

:0049D569 C1E902                  shr ecx, 02

:0049D56C 893DF0686000            mov dword ptr [006068F0], edi

:0049D572 33C0                    xor eax, eax

:0049D574 F3                      repz

:0049D575 AB                      stosd

:0049D576 8BCA                    mov ecx, edx

:0049D578 83E103                  and ecx, 00000003

:0049D57B F3                      repz

:0049D57C AA                      stosb

:0049D57D A1F4686000              mov eax, dword ptr [006068F4]

:0049D582 8B0DF0686000            mov ecx, dword ptr [006068F0]

:0049D588 56                      push esi

:0049D589 50                      push eax

:0049D58A 6A01                    push 00000001

:0049D58C 51                      push ecx

\* Reference To: MSVCR71.fread, Ord:02ABh

                                  |

:0049D58D FF1554905900            Call dword ptr [00599054]

:0049D593 56                      push esi

\* Reference To: MSVCR71.fclose, Ord:029Ah

                                  |

:0049D594 FF152C905900            Call dword ptr [0059902C]

:0049D59A 8D54243C                lea edx, dword ptr [esp+3C]

:0049D59E 52                      push edx

:0049D59F 8B15F0686000            mov edx, dword ptr [006068F0]

:0049D5A5 8D44243C                lea eax, dword ptr [esp+3C]

:0049D5A9 50                      push eax

:0049D5AA 8D4C243C                lea ecx, dword ptr [esp+3C]

:0049D5AE 51                      push ecx

:0049D5AF 52                      push edx

:0049D5B0 E8EBFFF6FF              call 0040D5A0                           /\*关键调用见后\*/

:0049D5B5 83C42C                  add esp, 0000002C

:0049D5B8 F6D8                    neg al

:0049D5BA 1BC0                    sbb eax, eax

:0049D5BC 25C01F0000              and eax, 00001FC0

:0049D5C1 83C040                  add eax, 00000040

:0049D5C4 A314995F00              mov dword ptr [005F9914], eax

:0049D5C9 EB19                    jmp 0049D5E4

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0049D53C(C)

|

:0049D5CB E8D072FDFF              call 004748A0

:0049D5D0 33C9                    xor ecx, ecx

:0049D5D2 85C0                    test eax, eax

:0049D5D4 0F9CC1                  setl cl

:0049D5D7 49                      dec ecx

:0049D5D8 83E140                  and ecx, 00000040

:0049D5DB 83C140                  add ecx, 00000040

:0049D5DE 890D14995F00            mov dword ptr [005F9914], ecx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0049D5C9(U)

|

:0049D5E4 E8C7D5FFFF              call 0049ABB0                        //程序功能限制函数调用见后
```

...................

....................

## 关键调用

```asm
:0040D5A0 6AFF                    push FFFFFFFF

:0040D5A2 68BBFC5700              push 0057FCBB

:0040D5A7 64A100000000            mov eax, dword ptr fs:[00000000]

:0040D5AD 50                      push eax

:0040D5AE 64892500000000          mov dword ptr fs:[00000000], esp

:0040D5B5 81EC7C010000            sub esp, 0000017C

:0040D5BB A1B0AB5F00              mov eax, dword ptr [005FABB0]

:0040D5C0 89842478010000          mov dword ptr [esp+00000178], eax

:0040D5C7 F60520BF5F0001          test byte ptr [005FBF20], 01

:0040D5CE 53                      push ebx

:0040D5CF 56                      push esi

:0040D5D0 751D                    jne 0040D5EF

:0040D5D2 A120BF5F00              mov eax, dword ptr [005FBF20]

:0040D5D7 83C801                  or eax, 00000001

:0040D5DA 33DB                    xor ebx, ebx

:0040D5DC A320BF5F00              mov dword ptr [005FBF20], eax

:0040D5E1 891D18BF5F00            mov dword ptr [005FBF18], ebx

:0040D5E7 891D1CBF5F00            mov dword ptr [005FBF1C], ebx

:0040D5ED EB02                    jmp 0040D5F1

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040D5D0(C)

|

:0040D5EF 33DB                    xor ebx, ebx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040D5ED(U)

|

\* Reference To: MSVCR71.\_time64, Ord:0211h

                                  |

:0040D5F1 8B3540905900            mov esi, dword ptr [00599040]

:0040D5F7 53                      push ebx

:0040D5F8 FFD6                    call esi

:0040D5FA A318BF5F00              mov dword ptr [005FBF18], eax

:0040D5FF A0D5675F00              mov al, byte ptr [005F67D5]

:0040D604 83C404                  add esp, 00000004

:0040D607 84C0                    test al, al

:0040D609 89151CBF5F00            mov dword ptr [005FBF1C], edx

:0040D60F 742A                    je 0040D63B

:0040D611 8D442408                lea eax, dword ptr [esp+08]

:0040D615 6840E05900              push 0059E040

:0040D61A 50                      push eax

:0040D61B E8B0FDFFFF              call 0040D3D0

:0040D620 8B08                    mov ecx, dword ptr [eax]

:0040D622 890D18BF5F00            mov dword ptr [005FBF18], ecx

:0040D628 8B5004                  mov edx, dword ptr [eax+04]

:0040D62B 83C408                  add esp, 00000008

:0040D62E 89151CBF5F00            mov dword ptr [005FBF1C], edx

:0040D634 C605D5675F0000          mov byte ptr [005F67D5], 00

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040D60F(C)

|

:0040D63B 53                      push ebx

:0040D63C FFD6                    call esi

:0040D63E 8B0D1CBF5F00            mov ecx, dword ptr [005FBF1C]

:0040D644 8B3518BF5F00            mov esi, dword ptr [005FBF18]

:0040D64A 83C404                  add esp, 00000004
```
