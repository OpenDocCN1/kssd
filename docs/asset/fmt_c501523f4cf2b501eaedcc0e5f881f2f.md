
### `sub_110EB` 与 `sub_111E8` 函数分析

`sub_110EB` 函数用来定位 `ntoskrnl.exe` 在内存中的位置；

`sub_111E8` 函数开始修改 SDT，hook 自己关心的函数，并返回原始函数地址，留到后面给自己用：

```assembly
.text:000111E8 sub_111E8       proc near            ; CODE XREF: sub_10883+4Cp
.text:000111E8                             ; sub_10883+57p
.text:000111E8                             ; sub_10883+62p
.text:000111E8                             ; sub_10883+6Dp
.text:000111E8                             ; sub_10883+7Dp
.text:000111E8
.text:000111E8 arg_0           = dword ptr  4
.text:000111E8
.text:000111E8                 mov     eax, BaseAddress
.text:000111ED                 test    eax, eax
.text:000111EF                 jz      short locret_1121F ; 基址为空则返回
.text:000111EF
.text:000111F1                 mov     ecx, [esp+arg_0] ; ecx = 压栈的函数地址
.text:000111F5                 mov     ecx, [ecx+1]    ; 取该函数的在 SDT 中的索引号(magic 
.text:000111F5                             ; number)
.text:000111F5                             ; 根据反汇编 Zw**可以得知，函数入口处
.text:000111F8                 test    ch, 30h
.text:000111FB                 push    esi
.text:000111FC                 jnz     short loc_1121C
.text:000111FC
.text:000111FE                 push    ecx          ; ecx = 入栈函数在 SDT 中的索引号
.text:000111FF                 push    eax          ; eax = BaseAddress
.text:00011200                 push    dword_11904  ; dword_11904 为 ntoskrnl.exe 的基址
.text:00011206                 call    sub_10C07    ; 修改 SDT，替换函数入口，返回原始函数
.text:00011206                             ; 地址。
.text:00011206
.text:0001120B                 mov     esi, eax     ; 保存返回的地址
.text:0001120D                 push    esi          ; VirtualAddress
.text:0001120E                 call    ds:MmIsAddressValid ; 检查该地址的有效性
.text:00011214                 test    al, al
.text:00011216                 jz      short loc_1121C ; 如果地址不可用则跳走
.text:00011216
.text:00011218                 mov     eax, esi     ; 否则保留该地址为此函数的返回值
.text:0001121A                 jmp     short loc_1121E
.text:0001121A
.text:0001121C ; ---------------------------------------------------------------
.text:0001121C
.text:0001121C loc_1121C:                   ; CODE XREF: sub_111E8+14j
.text:0001121C                             ; sub_111E8+2Ej
.text:0001121C                 xor     eax, eax     ; 返回值置零，表示失败
.text:0001121C
.text:0001121E
.text:0001121E loc_1121E:                   ; CODE XREF: sub_111E8+32j
.text:0001121E                 pop     esi
.text:0001121E
.text:0001121F
.text:0001121F locret_1121F:                 ; CODE XREF: sub_111E8+7j
.text:0001121F                 retn    4
.text:0001121F
.text:0001121F sub_111E8       endp
```

### SDT Hook 原理推测

Hook SDT 的代码，网上多得很，我这里没有仔细分析。简单用 `Windbg` 跟了一下，察看 `sub_10C07` 函数返回的地址，在 `Windbg` 中用 `uf` 命令来反汇编压栈的函数，看看地址是否吻合；或者用 `ln` 命令直接察看该地址也可以。所以可以猜测 hook 函数原型是：`NTPROC* HookFuncBySdt(NTPROC *Zw****);`

### `sub_10EFF` 线程创建与分析

一路返回到我们开始的地方，继续看下面的代码：

`sub_10EFF` 创建了一个线程，用来写注册表注册驱动，跟进之后来到：

...

```assembly
.text:00010F35                 add     esp, 18h
.text:00010F38                 push    offset sub_104D0 ; StartContext （回调函数的参数）
.text:00010F3D                 push    edi          ; StartRoutine
.text:00010F3E                 push    esi          ; ClientId
.text:00010F3F                 push    esi          ; ProcessHandle
.text:00010F40                 push    esi          ; ObjectAttributes
.text:00010F41                 push    esi          ; DesiredAccess
.text:00010F42                 lea     eax, [ebp+Handle]
.text:00010F45                 push    eax          ; ThreadHandle
.text:00010F46                 call    ds:PsCreateSystemThread ; 创建线程 sub_10EEF
```

...

线程回调函数的参数 `sub_104D0` 也是一个函数地址，用来写注册表来注册这个驱动；

`StartRoutine` 主要是打开 2 个文件，保存他们的句柄，使得用户不能关闭或删除他们：

```assembly
.text:00010DC5 ; void __stdcall StartRoutine(PVOID)
```

### `sub_10883` 函数尾部分析

在 `sub_10883` 的尾部，代码通过比较各个全局变量（`dword_11698`, `dword_116E4`, `dword_116E0`, `dword_116E8`, `dword_116EC`）是否为零来判断 Hook 是否成功。如果全部非零，则返回 1；否则跳转到 `loc_1093D`，将原始函数地址写回对应变量并返回 0。

```assembly
.text:000108D4                 push    ebx          ; ZwSetValueKey
.text:000108D5                 mov     dword_11698, eax ; dword_11698 = 原始 NtOpenKey 函数地
.text:000108DA                 call    sub_111E8
.text:000108DA
.text:000108DF                 push    ebp          ; ZwEnumerateKey
.text:000108E0                 mov     dword_116E4, eax ; dword_116E4 = 原始 NtSetValueKey 函
.text:000108E5                 call    sub_111E8
.text:000108E5
.text:000108EA                 push    edi          ; ZwClose
.text:000108EB                 mov     dword_116E0, eax ; dword_116E0 = 原始 NtEnumerateKey 函
.text:000108F0                 call    sub_111E8
.text:000108F0
.text:000108F5                 push    ds:ZwCreateFile
.text:000108FB                 mov     dword_116E8, eax ; dword_116E8 = 原始 NtClose 函数地址
.text:00010900                 call    sub_111E8
.text:00010900
.text:00010905                 mov     dword_116EC, eax ; dword_116EC = 原始 NtCreateFile 函数
.text:0001090A                 call    sub_11222    ; 取消映射，关闭 section、file 等句柄
.text:0001090A
.text:0001090F                 xor     eax, eax
.text:00010911                 cmp     dword_11698, eax ; 判断 dword_11698(NtOpenKey())是否为
.text:00010917                 jz      short loc_1093D ; 为零则跳
.text:00010917
.text:00010919                 cmp     dword_116E4, eax ; 判断 dword_116E4(NtSetValueKey())是
.text:0001091F                 jz      short loc_1093D
.text:0001091F
.text:00010921                 cmp     dword_116E0, eax ; 判断 dword_116E0(NtEnumerateKey())
.text:00010927                 jz      short loc_1093D
.text:00010927
.text:00010929                 cmp     dword_116E8, eax ; 判断 dword_116E8(NtClose)是否为空
.text:0001092F                 jz      short loc_1093D
.text:0001092F
.text:00010931                 cmp     dword_116EC, eax ; 判断 dword_116EC(NtCreateFile)是否
.text:00010937                 jz      short loc_1093D
.text:00010937
.text:00010939                 mov     al, 1
.text:0001093B                 jmp     short loc_10961 ; 否则返回 1
.text:0001093B
.text:0001093D ; ---------------------------------------------------------------
.text:0001093D
.text:0001093D loc_1093D:                   ; CODE XREF: sub_10883+94j
.text:0001093D                             ; sub_10883+9Cj
.text:0001093D                             ; sub_10883+A4j
.text:0001093D                             ; sub_10883+ACj
.text:0001093D                             ; sub_10883+B4j
.text:0001093D                 mov     eax, ds:ZwCreateFile
.text:00010942                 mov     dword_11698, esi ; esi = ZwOpenKey
.text:00010948                 mov     dword_116E8, edi ; edi = ZwClose
.text:0001094E                 mov     dword_116E4, ebx ; ebx = ZwSetValueKey
.text:00010954                 mov     dword_116E0, ebp ; ebp = ZwEnumerateKey
.text:0001095A                 mov     dword_116EC, eax ; eax = ZwCreateFile
.text:0001095A
.text:0001095F
.text:0001095F loc_1095F:                   ; CODE XREF: sub_10883+45j
.text:0001095F                 xor     al, al       ; 返回 0
.text:0001095F
.text:00010961
.text:00010961 loc_10961:                   ; CODE XREF: sub_10883+B8j
.text:00010961                 pop     edi
.text:00010962                 pop     esi
.text:00010963                 pop     ebp
.text:00010964                 pop     ebx
.text:00010965                 retn
.text:00010965
.text:00010965 sub_10883       endp
```
