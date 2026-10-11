
# 一个 Java 反汇编器的修改

> 原文：[`www.pediy.com/kssd/pediy06/pediy6772.htm`](https://www.pediy.com/kssd/pediy06/pediy6772.htm)

不知大家是否有用 Java 的，现在 Java 的反编译器很多（当然 Jad 最牛，而且是用 C 写的），而反汇编的不多，JCD version 2A 是一个挺老的反汇编程序，不过我自己觉得挺好用（有的时候被扰码后的 `.class` 不能反编译，所以必须反汇编）。不过 Jcd 有一个问题，就是不能显示字节代码，并且在代码中的偏移也是相对的，而不是文件偏移，这使得修改前还的自己计算，所以我们需要改进这些地方：

首先我们要找到反汇编 `Method` 的代码，用 W32dasm 的 `StringRef` 可以看到以下的代码：

```
* Possible StringData Ref from Data Obj ->"%8d%8caconst_null"

                          |

:004144B2 68DE0B4200               push 00420BDE
:004144B7 E875D3FFFF               call 00411831
:004144BC 83C40C                   add esp, 0000000C
:004144BF E9F9250000               jmp 00416ABD
:004144C4 6A20                     push 00000020
:004144C6 8BC3                     mov eax, ebx
:004144C8 43                       inc ebx
:004144C9 50                       push eax
```

再向前看，可以看到：

```
* Referenced by a CALL at Address:

|:00413395

|

:00414063 55                       push ebp
:00414064 8BEC                     mov ebp, esp
:00414066 81C4E0F3FFFF             add esp, FFFFF3E0
:0041406C 53                       push ebx
:0041406D 56                       push esi
:0041406E 57                       push edi
:0041406F 8B7D10                   mov edi, dword ptr [ebp+10] ; 代码偏移指针
:00414072 33F6                     xor esi, esi
:00414074 8B1F                     mov ebx, dword ptr [edi]
:00414076 8B4508                   mov eax, dword ptr [ebp+08] ；当前文件
:00414079 8B550C                   mov edx, dword ptr [ebp+0C] ；当前方法
:0041407C 03C2                     add eax, edx
:0041407E 0FB60418                  movzx eax, byte ptr [eax+ebx]
:00414082 3DFF000000               cmp eax, 000000FF
:00414087 0F871D2A0000             ja 00416AAA
:0041408D FF248594404100           jmp dword ptr [4*eax+00414094]
:00414094 94444100                 DWORD 00414494
:00414098 AC444100                 DWORD 004144AC
```

下面还有许多 `DWORD`，明显这是一个根据代码第一字节的跳转表。因而 JCD 的单条指令的反编译代码就是 `00414063`。再看代码 `413395` 处：

```
:0041338D 8D45BC                   lea eax, dword ptr [ebp-44]
:00413390 50                       push eax ；代码偏移指针
:00413391 53                       push ebx ；当前方法
:00413392 57                       push edi ；当前文件
:00413393 E8CB0C0000               call 00414063
:00413398 83C40C                   add esp, 0000000C
:0041339B 85C0                     test eax, eax ；代码正确
:0041339D 7522                     jne 004133C1
:0041339F 6A00                     push 00000000

* Possible StringData Ref from Data Obj ->"JCD"

                          |

:004133A1 6870004200               push 00420070
:004133A6 8D8631080000             lea eax, dword ptr [esi+00000831]
:004133AC 50                       push eax
:004133AD FF7508                   push [ebp+08]

* Reference To: USER32.MessageBoxA, Ord:0000h

                          |

:004133B0 E8EA9E0000               Call 0041D29F
:004133B5 33C0                     xor eax, eax
:004133B7 A374004200               mov dword ptr [00420074], eax
:004133BC E97F0C0000               jmp 00414040

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041339D(C)

|

:004133C1 8B45BC                   mov eax, dword ptr [ebp-44]
:004133C4 3B45D0                   cmp eax, dword ptr [ebp-30] ；当前方法的长度
:004133C7 7CC4                     jl 0041338D
```

这是一段循环直到改方法反汇编结束。再跟踪进入方法 `414063`，发现它将反编译完成的字符串放在 `[42008C]` 所指的 `DWORD` 所指的指针中。格式为 `"%8d%8c%s"`。现在，一切都清楚了，以下是我们所做的。

```
:00413390 FF30                     push dword ptr [eax] ；我们需要保存上次的偏移
:00413392 50                       push eax
:00413393 53                       push ebx
:00413394 57                       push edi
:00413395 E8C90C0000               call 00414063
:0041339A 83C40C                   add esp, 0000000C
:0041339D 85C0                     test eax, eax
:0041339F 0F849B0C0000             je 00414040
:004133A5 58                       pop eax ；上次的偏移
:004133A6 60                       pushad ；保存寄存器
:004133A7 8B4DBC                   mov ecx, dword ptr [ebp-44] ；这次的偏移
:004133AA BEF4D34100               mov esi, 0041D3F9 ；Format
:004133AF 2BC8                     sub ecx, eax ；这条指令的字节数
:004133B1 83F903                   cmp ecx, 00000003 ；我们只显示三个字节
:004133B4 7E05                     jle 004133BB
:004133B6 B903000000               mov ecx, 00000003

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004133B4(C)

|

:004133BB E99F9F0000               jmp 0041D35F ；代码在这里写不下
:004133C0 90                       nop
```

上面的代码跳到这里

```
:0041D35F 03FB                     add edi, ebx
:0041D361 03F8                     add edi, eax ；EDI 指向该指令的第一个字节
:0041D363 33D2                     xor edx, edx
:0041D365 890DE3D34100             mov dword ptr [0041D3E3], ecx ；保存 ECX 的值，在调用函数 wsprintf 后清栈用

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D37B(U)

|

:0041D36B 49                       dec ecx
:0041D36C 740F                     je 0041D37D
:0041D36E 8A140F                   mov dl, byte ptr [edi+ecx]
:0041D371 52                       push edx ；将指令字节推入栈
:0041D372 C70625303258             mov dword ptr [esi], 58323025 ；Format 字符串增加"%02X"
:0041D378 83C604                   add esi, 00000004
:0041D37B EBEE                     jmp 0041D36B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D36C(C)

|

:0041D37D 8A17                     mov dl, byte ptr [edi] ；该指令的第一个字节入栈
:0041D37F 52                       push edx
:0041D380 90                       nop
:0041D381 90                       nop
:0041D382 90                       nop
:0041D383 50                       push eax
:0041D384 03C3                     add eax, ebx ；当前指令偏移
:0041D386 50                       push eax
:0041D387 A18C004200               mov eax, dword ptr [0042008C]
:0041D38C 68EBD34100               push 0041D3EB ；Format 字符串（目前应为“%08X %02X[%02X...]”）
:0041D391 FF30                     push dword ptr [eax] ；目的字符串
:0041D393 C60600                   mov byte ptr [esi], 00 ；Format 字符串最后一位为 0

* Reference To: USER32.wsprintfA, Ord:0000h

                          |

:0041D396 E8AAFEFFFF               Call 0041D245
:0041D39B 83C410                   add esp, 00000014 ；先清 4 个 DWORD（buff，format，offset，offset, first byte）
:0041D39E 8B0DE3D34100             mov ecx, dword ptr [0041D3E3]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D3A8(U)

|

:0041D3A4 49                       dec ecx
:0041D3A5 7403                     je 0041D3AA
:0041D3A7 58                       pop eax ；根据 ECX 清除其他的栈中的数据
:0041D3A8 EBFA                     jmp 0041D3A4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041D3A5(C)

|

:0041D3AA A18C004200               mov eax, dword ptr [0042008C]
:0041D3AF 8B10                     mov edx, dword ptr [eax]
:0041D3B1 52                       push edx
:0041D3B2 52                       push edx

* Reference To: KERNEL32.lstrlenA, Ord:0000h

  |

:0041D3B3 E84FFDFFFF               Call 0041D107
:0041D3B8 5A                       pop edx
:0041D3B9 C6041020                 mov byte ptr [eax+edx], 20 ；我们必须将空格改回来
:0041D3BD 61                       popad
:0041D3BE E9FE5FFFFF               jmp 004133C ；一切 Over！
```

Format 数据 `"%04X:%03d %02X"`

```
:0041D3EB 253034583A               and eax, 3A583430
:0041D3F0 2530336420               and eax, 20643330
:0041D3F5 2530325800               and eax, 00583230
```

`4dd466d0e1e0cc123afbe504da2462ca`
