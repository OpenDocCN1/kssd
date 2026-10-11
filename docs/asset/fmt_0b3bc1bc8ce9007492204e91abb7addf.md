
#### 逆向工程分析

**003B0FC4** `E8 34140000` **call** `003B23FD` ; 当然要进去了

**003B242F** `60` **pushad**

**003B2474** `FF95 1BCF4100` **call** `dword ptr ss:[ebp+41CF1B]` ; `kernel32.GetCurrentProcessId`

**003B24A8** `8985 20FA4100` **mov** `dword ptr ss:[ebp+41FA20], eax` ; save process Id

**003B24B3** `8B85 1BCF4100` **mov** `eax, dword ptr ss:[ebp+41CF1B]` ; `kernel32.GetCurrentProcessId`

**003B24D0** `8985 24FA4100` **mov** `dword ptr ss:[ebp+41FA24], eax` ; `kernel32.GetCurrentProcessId`

; 不知道搞什么名堂

**003B2508** `B8 B8FA4100` **mov** `eax, 41FAB8`

**003B2512** `03C5` **add** `eax, ebp`

**003B2519** `8985 ACDC4100` **mov** `dword ptr ss:[ebp+41DCAC], eax`

**003B254C** `8B9D 08D64100` **mov** `ebx, dword ptr ss:[ebp+41D608]` ; Import Table RVA

-------------------

`0041F023 78750000`

`0041F027 00000000`

`0041F02B 00000000`

`0041F02F 82750000`

`0041F033 7D910000`

`0041F037 25350000`

`............`

-------------------

**003B2557** `83FB 00` **cmp** `ebx, 0` ; 没有 import 么？

**003B2587** `/0F84 1F0A0000` **je** `003B2FAC` ; 当然有，不跳

**003B25D1** `039D DCD34100` **add** `ebx, dword ptr ss:[ebp+41D3DC]` ; +ImageBase,得到 IT VA

; `ebx` -> `IID(s)`

**003B25E6** `8B43 0C` **mov** `eax, dword ptr ds:[ebx+C]` ; pointer to DLL asciz name

**003B25EE** `83F8 00` **cmp** `eax, 0`

**003B25F6** `/0F84 B0090000` **je** `003B2FAC` ; Game Over?

**003B2601** `53` **push** `ebx` ; Try.0041F023

**003B262F** `51` **push** `ecx`

**003B2647** `52` **push** `edx`

**003B2648** `33D2` **xor** `edx, edx`

**003B264F** `B9 20000000` **mov** `ecx, 20`

**003B2654** `33DB` **xor** `ebx, ebx` ; Try.0041F023

**003B2656** `D1F8` **sar** `eax, 1`

**003B265D** `0F92C3` **setb** `bl`

**003B2660** `D3E3` **shl** `ebx, cl`

**003B2667** `03D3` **add** `edx, ebx`

**003B2669** `^\\E2 E9` **loopd** `short 003B2654`

; DLL Name 的 RVA 解码,结果在 `edx` 输出

**003B266B** `8BC2` **mov** `eax, edx` ; `eax = edx = dll name rva`

**003B2684** `5A` **pop** `edx`

**003B2685** `59` **pop** `ecx`

**003B268B** `5B` **pop** `ebx`

**003B2696** `0385 DCD34100` **add** `eax, dword ptr ss:[ebp+41D3DC]` ; +ImageBase = VA

**003B26C9** `8BF0` **mov** `esi, eax`

**003B26D5** `C685 78894100 0>` **mov** `byte ptr ss:[ebp+418978], 0`

; -------------- 是否是特殊 DLL, 包括 Windows `Kernel32` & `User32`, 与 VB 的 `MSVB` ------------------------

**003B2709** `50` **push** `eax`

**003B270F** `8B00` **mov** `eax, dword ptr ds:[eax]`; 取 dll name 开头 4 个字节

**003B2716** `25 DFDFDFDF` **and** `eax, DFDFDFDF`; 转换为大写

**003B2720** `3D 4B45524E` **cmp** `eax, 4E52454K` ; `KERN****`

**003B2725** `74 07` **je** `short 003B272E`

**003B2727** `3D 55534552` **cmp** `eax, 52455355` ; `USER****`

**003B272C** `75 11` **jnz** `short 003B273F`

**003B2733** `C685 78894100 0>` **mov** `byte ptr ss:[ebp+418978], 1` ; 特殊 dll 标记

**003B2744** `58` **pop** `eax`

**003B2777** `50` **push** `eax` ; Try.00415C82

**003B277D** `8B00` **mov** `eax, dword ptr ds:[eax]`

**003B2784** `25 DFDFDFDF` **and** `eax, DFDFDFDF`

**003B278E** `3D 4D535642` **cmp** `eax, 4256534D` ; `MSVB****`

**003B2793** `75 11` **jnz** `short 003B27A6`

**003B279A** `C685 78894100 0>` **mov** `byte ptr ss:[ebp+418978], 1` ; 特殊 dll 标记

**003B27D3** `58` **pop** `eax` ; Try.00415C82

; 为何不一起判断? D.boy 大概写到这里喝多了:0

; ---------------------------------------------------------------------------

**003B27DE** `50` **push** `eax`

**003B27DF** `FF95 C13D4200` **call** `dword ptr ss:[ebp+423DC1]` ; `kernel32.LoadLibraryA`

; 载入 dll

**003B27EA** `8985 7FCC4100` **mov** `dword ptr ss:[ebp+41CC7F], eax` ; 保存 dll module base

**003B2822** `33C0` **xor** `eax, eax`

**003B2851** `8703` **xchg** `dword ptr ds:[ebx], eax` ; 清掉 IID 的 OriginalFirstThunk,然后把它读到 `eax`

; 如果改为 `mov eax, dword ptr [ebx]`可以避免...我偷懒,不改了,下面也是这样

**003B2858** `53` **push** `ebx`

**003B2886** `51` **push** `ecx`

**003B289E** `52` **push** `edx`

**003B289F** `33D2` **xor** `edx, edx`

**003B28A6** `B9 20000000` **mov** `ecx, 20`

**003B28AB** `33DB` **xor** `ebx, ebx`

**003B28AD** `D1F8` **sar** `eax, 1`

**003B28B4** `0F92C3` **setb** `bl`

**003B28B7** `D3E3` **shl** `ebx, cl`

**003B28BE** `03D3` **add** `edx, ebx`

**003B28C0** `^\\E2 E9` **loopd** `short 003B28AB`

**003B28C2** `8BC2` **mov** `eax, edx`

**003B28DB** `5A` **pop** `edx`

**003B28DC** `59` **pop** `ecx`

**003B28E2** `5B` **pop** `ebx`

**003B28E8** `8BF0` **mov** `esi, eax`

; OriginalFirstThunk 解码 -> `esi`

**003B292E** `33C0` **xor** `eax, eax`

**003B295D** `8743 10` **xchg** `dword ptr ds:[ebx+10], eax` ; `eax -> FirstThunk, & erase rva`

**003B2965** `53` **push** `ebx`

**003B2993** `51` **push** `ecx`

**003B29AB** `52` **push** `edx`

**003B29AC** `33D2` **xor** `edx, edx`

**003B29B3** `B9 20000000` **mov** `ecx, 20`

**003B29B8** `33DB` **xor** `ebx, ebx`

**003B29BA** `D1F8` **sar** `eax, 1`

**003B29C1** `0F92C3` **setb** `bl`

**003B29C4** `D3E3` **shl** `ebx, cl`

**003B29CB** `03D3` **add** `edx, ebx`

**003B29CD** `^\\E2 E9` **loopd** `short 003B29B8`

**003B29CF** `8BC2` **mov** `eax, edx`

**003B29E8** `5A` **pop** `edx`

**003B29E9** `59` **pop** `ecx`

**003B29EF** `5B` **pop** `ebx`

**003B29F5** `8BF8` **mov** `edi, eax`

; FirstThunk 解码 -> `edi`

**003B2A01** `83FE 00` **cmp** `esi, 0` ; OriginalFirstThunk 不能用?

**003B2A04** `/75 34` **jnz** `short 003B2A3A`; 能就走

**003B2A33** `8BF7` **mov** `esi, edi` ; 它不行就使用第二个表 FirstThunk

**003B2A67** `03B5 DCD34100` **add** `esi, dword ptr ss:[ebp+41D3DC]` ; +ImageBase,Get VA
