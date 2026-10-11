
# Awave Studio v7.0

> 原文：`[www.pediy.com/kssd/pediy03/forum428.htm](https://www.pediy.com/kssd/pediy03/forum428.htm)`

Awave Studio v7.0 的破解

工具：`TRW2000` 和 `Wdasm` `8.93` `Gold` 版 (`http://go.163.com/~wuhuashang/gold.exe`)

目标说明：一个能转换几乎所有音频格式的工具，用于聆听不同平台音频文件的最好工具之一，
你可以以各种方式使用它：音频文件格式转换器、音频编辑器、音频播放器、以及作为一个通用波表合成器格式的转换器和编辑器。`Awave` 可以读取差不多所有的音频文件
格式，同时它可以输出大部分流行的音频文件格式。

难度：中级

下载地址：`http://www.fmjsoft.com`

```
==================================================================================
```

`Awave` `Studio` `v7.0` 是用 `Asprotect` 加的壳，无法直接用 `Wdasm` 反汇编，只有先找到它的 `OEP`，再从内存
中 `Dump` 出脱离壳后的文件映像(无需重建 `Import` `Table`)，具体过程可参看《论坛精华 II》中的教程！

运行 `Wdasm` `8.93`(`Gold` 版)反汇编 `Dump` 出来的文件，在 `String` `Data` `References` 中会看到 `"Registered to:%s"`
的字样。通过 `RegMon` 监测，该软件启动时会校验注册表中的 `Username` 和 `Usercode` 两个键值，已确定是否已注册。

为了便于跟踪，可手动建立下列两个键及键值：

```
[HKEY_CURRENT_USER\Software\FMJ-Software\Awave Studio\]

"Username"="sUpErbOss"

"Usercode"="1122334455"
```

启动 `TRW2000`，加载 `Awave` `Studio`，点击 `"Load"` 按钮。进入到 `TRW2000` 环境，下指令 `"faults off"`，
再打 `"G 0046E22C"`，即可！

```
:0046E22C E82F140000                call 0046F660 <----重点！！(这是一个共用 Call！)

:0046E231 84C0                      test al, al  <----al=0，则为注册版！

:0046E233 7410                      je 0046E245

:0046E235 8B06                      mov eax, dword ptr [esi]

:0046E237 6A00                      push 00000000

* Possible StringData Ref from Data Obj ->"Unregistered copy"

                              |

:0046E239 689CD34C00              push 004CD39C

:0046E23E 8BCE                    mov ecx, esi

:0046E240 FF500C                  call [eax+0C]

:0046E243 EB44                    jmp 0046E289

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0046E233(C)

|

:0046E245 68ADF04D00              push 004DF0AD

:0046E24A 8D4C2444                lea ecx, dword ptr [esp+44]

* Possible StringData Ref from Data Obj ->"Registered to: %s"

                              |

:0046E24E 6888D34C00              push 004CD388

:0046E253 51                      push ecx

:0046E254 E826810300              call 004A637F

================================================================================
* Referenced by a CALL at Addresses:

|:0046E22C  , :0046EAC5  , :0046EB17

|

:0046F660 53                       push ebx

:0046F661 56                      push esi

:0046F662 68E8030000              push 000003E8

:0046F667 E82F6A0300              call 004A609B

:0046F66C 83C404                  add esp, 00000004

:0046F66F 8BF0                    mov esi, eax

:0046F671 E81A000000              call 0046F690  <--------核心 Call(1)！跟进去！！

:0046F676 8AD8                    mov bl, al

:0046F678 A05CF44D00              mov al, byte ptr [004DF45C]

:0046F67D 56                      push esi

:0046F67E 84C0                    test al, al

:0046F680 E80B6A0300              call 004A6090

:0046F685 83C404                  add esp, 00000004

:0046F688 8AC3                    mov al, bl

:0046F68A 5E                      pop esi

:0046F68B 5B                      pop ebx

:0046F68C C3                      ret

===============================================================================
* Referenced by a CALL at Address:

|:0046F671

|

:0046F690 E80B000000              call 0046F6A0  <-----继续跟进！

:0046F695 F6D8                    neg al

:0046F697 1BC0                    sbb eax, eax

:0046F699 40                      inc eax

:0046F69A C3                      ret

=============================================================================
* Referenced by a CALL at Address:

|:0046F690

|

:0046F6A0 B9D0E34D00              mov ecx, 004DE3D0

:0046F6A5 C6055CF44D0001           mov byte ptr [004DF45C], 01

:0046F6AC E81F95FFFF               call 00468BD0  <-----继续跟进！

:0046F6B1 F6D8                    neg al

:0046F6B3 1BC0                    sbb eax, eax

:0046F6B5 40                      inc eax

:0046F6B6 C3                      ret

=============================================================================
* Referenced by a CALL at Addresses:

|:0046445C  , :00466ECC  , :0046BB9C  , :0046F6AC  , :0047E6CC

|

:00468BD0 83EC1C                    sub esp, 0000001C

* Possible StringData Ref from Data Obj ->"裃 j?c 佀 4UN 塳桗勩菥?篤顤?'`鯚骸?"
                                          ->".啅 L 摚|菹 Q 赕暌??

                              |

:00468BD3 C705B8F14D00D8AC4C00     mov dword ptr [004DF1B8], 004CACD8

:00468BDD 53                      push ebx

:00468BDE 55                      push ebp

:00468BDF 8BE9                    mov ebp, ecx

:00468BE1 57                      push edi

:00468BE2 6A06                    push 00000006

* Possible StringData Ref from Data Obj ->"@DATE="

                              |

:00468BE4 68ACB54C00              push 004CB5AC

:00468BE9 8D9DDD0C0000             lea ebx, dword ptr [ebp+00000CDD]

:00468BEF 53                      push ebx

:00468BF0 E8AB930400              call 004B1FA0

:00468BF5 83C40C                  add esp, 0000000C

:00468BF8 85C0                    test eax, eax  <----eax=1

:00468BFA 0F85B1000000             jne 00468CB1

=============================================================================
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00468BFA(C), :00468CA6(C)

|

:00468CB1 8B0DB8F14D00             mov ecx, dword ptr [004DF1B8]

:00468CB7 8B95610D0000             mov edx, dword ptr [ebp+00000D61]

:00468CBD 51                      push ecx

:00468CBE 8BFB                    mov edi, ebx

:00468CC0 83C9FF                  or ecx, FFFFFFFF

:00468CC3 33C0                    xor eax, eax

:00468CC5 81C5210D0000            add ebp, 00000D21

:00468CCB 52                      push edx

:00468CCC F2                      repnz

:00468CCD AE                      scasb

:00468CCE F7D1                    not ecx

:00468CD0 49                      dec ecx

:00468CD1 55                      push ebp

:00468CD2 51                      push ecx

:00468CD3 53                      push ebx

:00468CD4 E807BB0300              call 004A47E0  <-----继续跟进！

:00468CD9 85C0                    test eax, eax

:00468CDB 5F                      pop edi

:00468CDC 5D                      pop ebp

:00468CDD 0F95C0                  setne al

:00468CE0 5B                      pop ebx

:00468CE1 83C41C                  add esp, 0000001C

:00468CE4 C3                      ret

=============================================================================
* Referenced by a CALL at Addresses:

|:00468CD4  , :00468DEE

|

:004A47E0 81ECD0000000             sub esp, 000000D0

:004A47E6 8D442438                lea eax, dword ptr [esp+38]

:004A47EA 56                      push esi

:004A47EB 50                      push eax

:004A47EC E84F010000              call 004A4940

:004A47F1 8D44243C                lea eax, dword ptr [esp+3C]

:004A47F5 8B8C24DC000000          mov ecx, dword ptr [esp+000000DC]

:004A47FC 8B9424D8000000          mov edx, dword ptr [esp+000000D8]

:004A4803 51                      push ecx

:004A4804 52                      push edx

:004A4805 50                      push eax

:004A4806 E865010000              call 004A4970

:004A480B 8BB424E4000000          mov esi, dword ptr [esp+000000E4]

:004A4812 83FE40                  cmp esi, 00000040

:004A4815 760F                    jbe 004A4826  <-----跳转！！

:004A4817 B899090000              mov eax, 00000999

:004A481C 5E                      pop esi

:004A481D 81C4D0000000            add esp, 000000D0
```
