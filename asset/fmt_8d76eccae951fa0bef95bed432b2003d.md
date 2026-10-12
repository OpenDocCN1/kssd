
# ActiveSkin 3.62

> 原文：[`www.pediy.com/kssd/pediy04/kanxue212.htm`](https://www.pediy.com/kssd/pediy04/kanxue212.htm)

`ActiveSkin` 是一个很出色的 skin（“皮肤”）`AcitveX` 控件。支持 `vb`，`vc`，`dephi` 等，功能十分强大。

下载地址：`http://www.softshape.com/download/activeskin.zip` (`v3.62` 版)

安装后，将在 `windows` 的 `system` 下产生 `ActiveSkin.ocx` 文件。

它的加密方式比较特别，如果直接修改原文件的话，修改后的 `ActiveSkin.ocx` 将不能再次注册成为控件。（即：用“`Regsvr32.exe` `ActiveSkin.ocx`”命令注册控件会失败，导致你开发的程序不能用在其他电脑上）。

因为是未注册的，所以当你的程序调用 `ActiveSkin` 时，将弹出 `Unregistered` 的对话框。

用 `w32asm` 反汇编 `ActiveSkin.ocx`，查找字符“`Unregistered control`”，可看到下面代码：

```
:12121503 E879350000              call 12124A81

:12121508 33DB                    xor ebx, ebx

:1212150A 395DF8                  cmp dword ptr [ebp-08], ebx

:1212150D 7441                    je 12121550

:1212150F 389E44010000            cmp byte ptr [esi+00000144], bl

:12121515 7548                    jne 1212155F

* Reference To: KERNEL32.GetTickCount, Ord:016Dh

                              |

:12121517 8B3DB4211512          mov edi, dword ptr [121521B4]

:1212151D FFD7                  call edi

:1212151F 8945F4                mov dword ptr [ebp-0C], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:12121549(C)

|

:12121522 6A10                    push 00000010

* Possible StringData Ref from Data Obj ->"Unregistered control"

                              |

:12121524 68F0851512              push 121585F0

* Possible StringData Ref from Data Obj ->"Warning! This application was "

                              ->"created with trial version of "

                              ->"ActiveSkin control."

                              |

:12121529 685C851512              push 1215855C

:1212152E 53                      push ebx

* Reference To: USER32.MessageBoxA, Ord:01BEh

                              |

:1212152F FF155C231512            Call dword ptr [1215235C]

============================================================================
```

很显然，修改 `12121515` 处，将 `jne` 改为 `jmp` 就可跳过对话框。为什么不改 `[esi+00000144]` 的初始化代码呢？因为有好几个地方将 `[esi+00000144]` 置 `0`，其中包括 `mfc42.dll`。改好后拿到别人电脑上注册控件，命令行方式下输入“`Regsvr32.exe` `ActiveSkin.ocx`”，失败！看来还有暗桩。将 `ActiveSkin.ocx` 恢复回原来的文件。再用 `w32asm` 反汇编。我们知道 `AcitveX` 控件注册时都要提供一个引出函数“`DllRegisterServer`”供 `Regsvr32.exe` 调用。可能 `ActiveSkin.ocx` 在函数 `DllRegisterServer` 检查自身是否被修改过。如果被修改过将注册失败。

现在看看 `DllRegisterServer` 函数，选菜单 `Functions`->`Exports`，查 `DllRegisterServer` 函数，知道它的地址在 `12125386` 处。

好！先运行 `trw2000`，`control-n`，下断点 `bpx loadlibraryexa do "d *(esp+4)"`，这个命令当 `loadlibraryexa` 被执行是中断，并显示出参数。然后在开始菜单的运行窗口输入“`Regsvr32.exe` 路径\`ActiveSkin.ocx`”

中断后，看一下参数，`f5`，直到参数为 `ActiveSkin.ocx`，`f12` 数次，回到 `Regsvr32.exe` 领空，此时下命令：`bpx 12125386;f5`，来到 `ActiveSkin` 的领空：

```
==============================================================================================

Exported fn(): DllRegisterServer - Ord:0003h

:12125386 833D94DB151200          cmp dword ptr [1215DB94], 00000000 <==解码标志

:1212538D 7523                    jne 121253B2

:1212538F 68534F1312              push 12134F53

* Possible StringData Ref from Code Obj ->"?+?"

                              |

:12125394 68744E1312              push 12134E74

:12125399 BA5F161212              mov edx, 1212165F

:1212539E B9C1141212              mov ecx, 121214C1

:121253A3 E8E7900100              call 1213E48F      <=========注意此 call；进入

:121253A8 C70594DB151201000000    mov dword ptr [1215DB94], 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1212538D(C)

|

:121253B2 6A00                    push 00000000

:121253B4 6A01                    push 00000001

:121253B6 6810DB1512              push 1215DB10

:121253BB E83C150000              call 121268FC

:121253C0 C3                      ret

===============================================================================================

* Referenced by a CALL at Addresses:

|:121210DC  , :121253A3  , :121253DE  , :1214792D  , :1214AC65

|

:1213E48F 55                      push ebp

:1213E490 8BEC                    mov ebp, esp

:1213E492 83EC24                  sub esp, 00000024

:1213E495 53                      push ebx

:1213E496 56                      push esi

:1213E497 33DB                    xor ebx, ebx

:1213E499 57                      push edi

:1213E49A 8955E0                  mov dword ptr [ebp-20], edx

:1213E49D 894DDC                  mov dword ptr [ebp-24], ecx

:1213E4A0 894DF4                  mov dword ptr [ebp-0C], ecx

:1213E4A3 885DFE                  mov byte ptr [ebp-02], bl

:1213E4A6 885DFD                  mov byte ptr [ebp-03], bl

:1213E4A9 885DFF                  mov byte ptr [ebp-01], bl

:1213E4AC 885DFC                  mov byte ptr [ebp-04], bl

:1213E4AF 895DF8                  mov dword ptr [ebp-08], ebx

:1213E4B2 895DE4                  mov dword ptr [ebp-1C], ebx

* Reference To: KERNEL32.GetCurrentProcess, Ord:00F7h

                              |

:1213E4B5 FF15CC211512            Call dword ptr [121521CC]

:1213E4BB BE30DC1512              mov esi, 1215DC30

:1213E4C0 8945E8                  mov dword ptr [ebp-18], eax

:1213E4C3 6804010000              push 00000104

:1213E4C8 56                      push esi

:1213E4C9 FF3514DB1512            push dword ptr [1215DB14]

* Reference To: KERNEL32.GetModuleFileNameA, Ord:0124h

                              |

:1213E4CF FF154C211512            Call dword ptr [1215214C]

:1213E4D5 53                      push ebx

:1213E4D6 6880000000              push 00000080

:1213E4DB 6A03                    push 00000003

:1213E4DD 53                      push ebx

:1213E4DE 6A01                    push 00000001

:1213E4E0 6800000080              push 80000000

:1213E4E5 56                      push esi          <=====打开"ActiveSkin.ocx"

* Reference To: KERNEL32.CreateFileA, Ord:0034h

                              |

:1213E4E6 FF15D8211512            Call dword ptr [121521D8]

:1213E4EC BE00101212              mov esi, 12121000

:1213E4F1 8945F0                  mov dword ptr [ebp-10], eax

:1213E4F4 8BC6                    mov eax, esi

:1213E4F6 2B054CA51512            sub eax, dword ptr [1215A54C]

:1213E4FC 8945EC                  mov dword ptr [ebp-14], eax

:1213E4FF 740E                    je 1213E50F

:1213E501 3AC3                    cmp al, bl

:1213E503 750A                    jne 1213E50F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1213E50A(C)

|

:1213E505 C1F808                  sar eax, 08

:1213E508 3AC3                    cmp al, bl

:1213E50A 74F9                    je 1213E505

:1213E50C 8945EC                  mov dword ptr [ebp-14], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:1213E4FF(C), :1213E503(C)

|

* Reference To: KERNEL32.SetFilePointer, Ord:026Ah

                              |

:1213E50F 8B3DDC201512            mov edi, dword ptr [121520DC]

:1213E515 8945F8                  mov dword ptr [ebp-08], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1213E5F3(C)

|

:1213E518 53                      push ebx

:1213E519 8D45FE                  lea eax, dword ptr [ebp-02]

:1213E51C 6A01                    push 00000001

:1213E51E 50                      push eax

:1213E51F FF7508                  push [ebp+08]        〈=======初始地址为 12134e74

:1213E522 FF75E8                  push [ebp-18]

* Reference To: KERNEL32.ReadProcessMemory, Ord:021Ch

                              |

:1213E525 FF15D8201512            Call dword ptr [121520D8]
```
