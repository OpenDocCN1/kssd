
# 实战 API 函数 －－ API 绝密档案系列之五

> 原文：[`www.pediy.com/kssd/pediy10/23595.html`](https://www.pediy.com/kssd/pediy10/23595.html)

## 背景介绍

实战 API 函数 －－ API 绝密档案系列之五。

本来没有这个续篇，因为两个原因，催生了这篇不该出现的东西。上面第四篇谈的都是自己打造的函数，我觉得有必要给各位欣赏一下，现在流行猛壳是如何打造这个函数的。

我将选择顶级猛壳 `Themida` 的 `GetFunctionAddress` 来展示在实际中是如何实现获取函数调用地址的。

代码很长，大多是垃圾代码，我对代码做了详细的解释，对有用的代码都加了注解，并加了编号，没有编号的就是垃圾代码了，你可以完全不看，如果你觉得垃圾代码干扰你的思维，可以将凡是没有注解的地方都删除，余下的就是真正的程序了。

这段代码是从 `Themida` 加密的 `notepad.exe` 壳中挖出来的。我在 `IDA` 中进行了整理，你所看到的和你自己在 `IDA` 或 `Debug` 中看到的可能在感觉上完全不同。在回答我的前几篇帖子中，我看到有朋友希望我能讲讲怎样使用 `IDA`，不是谦虚，我对 `IDA` 所知甚少，离著作还差的远，我想大家不仿要求在论坛注册名为 `bpx` 的高手来阐述，`IDA` 对于分割线不支持中文操作系统的一个 `Plugin` 就出自这位大侠之手。或者请看雪老大亲自出手，也比我强的多。

分析这些代码对于新手来说可能是非常困难的一件事，这个你不用自卑，我干这个工作已经超过 20 年，如果你也干了这么长的时间，非常坦白的说，你肯定比我强。

论坛上有许多朋友对破解彼有心得和建树，但许多朋友并不擅长分析，所谓“知己知彼，百战不殆”，了解一下壳是怎样具体工作的，对你一定有帮助。

这篇文章是以非常严肃的态度来处理的，原因是面对创造 `Themida` 这样强壳的作者，应该给予尊敬。

## 技术分析

`Themida` 的 `GetFunctionAddress` 入口参数有三个，其中一个是通过变量 `APIFristChar` 来传递 `API` 函数的第一个字母，目的可能是为了加快运行速度。

代码:



```asm
010D505D ; int __stdcall GetFunctionAddress(int BaseAddress,int CryptCode)
010D505D GetFunctionAddress proc near
010D505D
010D505D FunctionAddress= dword ptr -4
010D505D BaseAddress= dword ptr 4
010D505D CryptCode= dword ptr 8
010D505D
010D505D         pusha                                   ; 1
010D505E         push    eax
010D505F         push    edx
010D5060         push    ecx
010D5061         push    ecx
010D5062         pusha
010D5063         popa
010D5064         sidt    qword ptr [esp-2]
010D5069         pop     ecx
010D506A         jmp     loc_10D507E
010D506F         db 11h, 0AAh, 58h, 8Eh, 9Bh, 0E4h, 0CFh, 16h, 67h, 0C0h, 0DDh
010D506F         db 0F7h, 0F8h, 13h, 0C5h
010D507E
010D507E loc_10D507E:
010D507E         pop     ecx
010D507F         rdtsc
010D5081         jnb     loc_10D5089
010D5087         pusha
010D5088         popa
010D5089
010D5089 loc_10D5089:
010D5089         pop     edx
010D508A         pop     eax
010D508B         mov     eax, 0                          ; 2
010D5090         pusha
010D5091         movsx   ecx, cx
010D5094         add     [ebp+153E2D91h], edi
010D509A         popa
010D509B         mov     edx, [esp+20h+CryptCode]        ; 3  CryptCode 函数名加密代码
010D509F         stc
010D50A0         mov     ss:(dwFunctionsCount+1531F38Ah - ThemidaSeg)[ebp], eax ; 4
010D50A6         pusha
010D50A7         mov     edi, [ebp+153E23DDh]
010D50AD         mov     edx, [ebp+153E2DB9h]
010D50B3         popa
010D50B4         push    edx
010D50B5         push    edx
010D50B6         mov     [ebp+153E11CDh], edx
010D50BC         sidt    qword ptr [esp-2]
010D50C1         pop     edx
010D50C2         push    eax
010D50C3         push    edx
010D50C4         stc
010D50C5         rdtsc
010D50C7
010D50C7 loc_10D50C7:
010D50C7         sub     [ebp+153E04B5h], edx
010D50CD         pop     edx
010D50CE         pop     eax
010D50CF         pop     edx
010D50D0         mov     esi, IMAGE_DOS_HEADER.e_lfanew ; 5 PE 文件头的偏移量
010D50D5         push    eax
010D50D6         push    edx
010D50D7         jle     loc_10D50E3
010D50DD         jle     $+6
010D50E3
010D50E3 loc_10D50E3:
010D50E3         rdtsc
010D50E5         jmp     loc_10D50FB
010D50EA         db 6Fh, 95h, 0D0h, 51h, 69h, 31h, 0ECh, 0CEh, 9Fh, 65h, 0FEh, 2Bh
010D50EA         db 0B8h, 0Ch, 0DEh, 40h, 18h
010D50FB
010D50FB loc_10D50FB:
010D50FB         pop     edx
010D50FC         pop     eax
010D50FD         add     esi, [esp+20h+BaseAddress]       ; 6 DosHeader.e_lfanew + BaseAddress
010D5101         mov     [ebp+153E0021h], edi
010D5107         pusha
010D5108         call    loc_10D5119
010D510D         db 2 dup(53h), 0AFh, 7Ah, 0B8h, 0A5h, 8Ah, 20h, 58h, 84h, 8Fh
010D510D         db 91h
010D5119
010D5119 loc_10D5119:
010D5119         jb      $+6
010D511F         pop     eax
010D5120         mov     [ebp+153E029Dh], ebx
010D5126         popa
010D5127         lodsw                           ; 7 获取 PE 头的偏移量
010D5129         sub     [ebp+153E271Dh], ebx
010D512F         add     eax, [esp+20h+BaseAddress]       ; 8 加上基地址得到 PE 头的起始位置
010D5133         jmp     loc_10D5142
010D5138         db 3Dh, 0BCh, 69h, 0B6h, 4Bh, 0ADh, 0AEh, 0E2h, 79h, 2Bh
010D5142
010D5142 loc_10D5142:
010D5142         nop
010D5143         jle     loc_10D5157
010D5149         pusha
010D514A         jg      $+6
010D5150         jl      $+6
010D5156         popa
010D5157
010D5157 loc_10D5157:
010D5157         push    eax                     ; 9
010D5158         jnp     loc_10D5164
010D515E         pusha
010D515F         mov     esi, edx
010D5161         mov     ah, dl
010D5163         popa
010D5164
010D5164 loc_10D5164:
010D5164         nop
010D5165         pusha
010D5166         clc
010D5167         call    loc_10D5171
010D516C         db 0D2h, 1, 94h, 0ECh, 0E3h
010D5171
010D5171 loc_10D5171:
010D5171         xor     eax, [ebp+153E2071h]
010D5177         pop     ecx
010D5178         popa
010D5179         mov     eax, [eax+IMAGE_NT_HEADERS.OptionalHeader.DirectoryExport.VirtualAddress] ; 10
010D517C         jnb     loc_10D5190
010D5182         pusha
010D5183         push    eax
010D5184         push    edx
010D5185         rdtsc
010D5187         pop     edx
010D5188         pop     eax
010D5189         or      [ebp+153E1569h], edx
010D518F         popa
010D5190
010D5190 loc_10D5190:
010D5190         stc
010D5191         add     eax, [esp+24h+BaseAddress]       ; 11, Export.VirtualAddress + BaseAddress
010D5195         sub     [ebp+153E1965h], edi
010D519B         mov     eax, [eax+IMAGE_EXPORT_DIRECTORY.NumberOfNames] ; 12
010D519E         jb      loc_10D51AA
010D51A4         mov     [ebp+153E0D35h], ecx
010D51AA
010D51AA loc_10D51AA:
010D51AA         inc     eax                     ; 13
010D51AB         jbe     loc_10D51BE
010D51B1         push    eax
010D51B2         push    edx
010D51B3         cmc
010D51B4         rdtsc
010D51B6         push    eax
010D51B7         push    edx
010D51B8         rdtsc
010D51BA         pop     edx
010D51BB         pop     eax
010D51BC         pop     edx
010D51BD         pop     eax
010D51BE
010D51BE loc_10D51BE:                              ; 14
010D51BE         mov     ss:(dwNumeberOfName+1531F38Ah - ThemidaSeg)[ebp], eax
010D51C4         sub     [ebp+153E199Dh], esi
010D51CA         pop     eax                     ; 15
010D51CB         pusha
010D51CC         mov     eax, [ebp+153E0181h]
010D51D2         jmp     loc_10D51E2
010D51D7         db 1Fh, 5Ch, 94h, 99h, 29h, 91h, 49h, 0Dh, 0B6h, 97h, 8Eh
010D51E2
010D51E2 loc_10D51E2:
010D51E2         popa
010D51E3         mov     esi, [eax+IMAGE_NT_HEADERS.OptionalHeader.DirectoryExport.VirtualAddress] ; 16
010D51E6         jb      loc_10D51F2
010D51EC         or      [ebp+153E2ED9h], ebx
010D51F2
010D51F2 loc_10D51F2:
010D51F2         add     esi, IMAGE_EXPORT_DIRECTORY.AddressOfFunctions ; 17
010D51F5         jnp     loc_10D5210
010D51FB         push    eax
010D51FC         push    edx
010D51FD         push    esi
010D51FE         push    esi
010D51FF         sidt    qword ptr [esp-2]
010D5204         pop     esi
010D5205         pop     esi
010D5206         rdtsc
010D5208         sub     [ebp+153E2869h], edi
010D520E         pop     edx
010D520F         pop     eax
010D5210
010D5210 loc_10D5210:                              ; 18, AddressOfFunctions + BaseAddress
010D5210         add     esi, [esp+20h+BaseAddress]
010D5214         jmp     loc_10D521F
010D5219         db 88h, 0C1h, 0A4h, 6Ah, 85h, 17h
010D521F
010D521F loc_10D521F:
010D521F         clc
010D5220         lea     edi, (dwAddressOfFunctions+1531F38Ah - ThemidaSeg)[ebp] ; 19
010D5220                                             ; 取 AddressOfFunctions 变量地址
010D5226         pusha
010D5227         and     eax, [ebp+153E2411h]
010D522D         mov     esi, [ebp+153E2285h]
010D5233         popa
010D5234         lodsd                           ; 20 取 AddressOfFunctions 的偏移量
010D5235         mov     [ebp+153E15F9h], eax
010D523B         add     eax, [esp+20h+BaseAddress]       ; 21 + BaseAddress
010D523F         jmp     loc_10D524F
010D5244         db 3Ch, 69h, 36h, 4, 0A3h, 0C9h, 88h, 19h, 0CBh, 3Fh, 7Fh
010D524F
010D524F loc_10D524F:
010D524F         stosd                           ; 22 保存 AddressOfFunctions
010D5250         jmp     loc_10D5267
010D5255         db 0ABh, 37h, 2 dup(50h), 53h, 9Ah, 68h, 0C4h, 36h, 69h, 0B8h
010D5255         db 2Ah, 7Ah, 0F5h, 52h, 0E6h, 33h, 0E7h
010D5267
010D5267 loc_10D5267:
010D5267         jmp     loc_10D5271
010D526C         db 62h, 0CCh, 5Dh, 5Ah, 0BAh
010D5271
010D5271 loc_10D5271:
010D5271         lea     edi, (dwAddressOfNames+1531F38Ah - ThemidaSeg)[ebp] ; 23
010D5271                                             ; 取保存 AddressOfNames 变量地址
010D5277         mov     [ebp+153E2219h], edi
010D527D         lodsd                           ; 24 取 AddressOfNames 的偏移量
010D527E         pusha
010D527F         mov     ebx, [ebp+153E044Dh]
010D5285         mov     ecx, [ebp+153E1A51h]
010D528B         popa
010D528C         add     eax, [esp+20h+BaseAddress]       ; 25 + BaseAddress
010D5290         pusha
010D5291         or      [ebp+153E133Dh], ecx
010D5297         cmc
010D5298         popa
010D5299         push    eax                     ; 26
010D529A         push    eax
010D529B         push    edx
010D529C         jmp     loc_10D52B4
010D52A1         db 4Dh, 0AEh, 42h, 0B7h, 0AEh, 0E4h, 0B0h, 3Ah, 0CCh, 15h, 0EEh
010D52A1         db 31h, 6Bh, 0F2h, 4Eh, 96h, 0E6h, 3Fh, 0A0h
010D52B4
010D52B4 loc_10D52B4:
010D52B4         rdtsc
010D52B6         push    eax
010D52B7         push    edx
010D52B8         jmp     loc_10D52C8
010D52BD         db 0B8h, 21h, 0F6h, 3Ah, 0D2h, 0D3h, 0ADh, 66h, 0D6h, 0F6h, 0D1h
010D52C8
010D52C8 loc_10D52C8:
010D52C8         rdtsc
010D52CA         pusha
010D52CB         popa
010D52CC         pop     edx
010D52CD         pop     eax
010D52CE         pop     edx
010D52CF         pop     eax
010D52D0         mov     [esp], eax
010D52D3         mov     [ebp+153E1895h], esi
010D52D9         stosd                           ; 27 保存 AddressOfNames 地址
010D52DA         jnz     loc_10D52F9
010D52E0         jmp     loc_10D52F9
010D52E5         db 9Eh, 3Ah, 2Ah, 0E4h, 0BCh, 46h, 8Bh, 5, 7Dh, 0C5h, 9, 0C0h
010D52E5         db 0AFh, 61h, 0A4h, 0Ah, 0FAh, 84h, 0A1h, 0D4h
010D52F9
010D52F9 loc_10D52F9:
010D52F9         lea     edi, (dwAddressOfNamesOrdinals+1531F38Ah - ThemidaSeg)[ebp] ; 28
010D52F9                                             ; 取 AddressOfNameOrdinals 变量地址
010D52FF         stc
010D5300         cld                             ; 29
010D5301         lodsd                           ; 30 读 AddressNameOrdinals 的偏移量
010D5302         push    eax
010D5303         push    edx
010D5304         jnz     loc_10D5320
010D530A         jmp     loc_10D5320
010D530F         db 6Ch, 29h, 0AEh, 0E6h, 87h, 64h, 49h, 7Ah, 92h, 0Ch, 38h, 5
010D530F         db 0CCh, 0CEh, 3Fh, 18h, 0F2h
010D5320
010D5320 loc_10D5320:
010D5320         rdtsc
010D5322         cld
010D5323         pop     edx
010D5324         pop     eax
010D5325         pusha
010D5326
```
