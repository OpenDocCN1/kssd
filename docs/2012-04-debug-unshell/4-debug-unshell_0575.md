
# 脱壳教程：Crunch/PE -> BitArts

> 原文：[`www.pediy.com/kssd/pediy05/pediy50118.htm`](https://www.pediy.com/kssd/pediy05/pediy50118.htm)

## 概述

本文介绍如何脱除 Crunch/PE 到 BitArts 的壳。

这个壳并不困难，未发现有反跟踪功能。其复杂度远低于软件作者早期作品中添加的 PEcompact 修改版壳。

为遵守论坛约定，本文不提及具体软件名称。

该壳实现较为简单，以下是脱壳过程。

## 动态跟踪与分析

### 1. 启动跟踪

运行 `trw`，载入程序。

```assembly
BitArts:005A2000 start           proc near
BitArts:005A2000                 push    ebp
BitArts:005A2001                 call    5A2006 <--- 此处需按 F8，因为并非真正的 call，只是 jmp 的变形。在脱 aspr 壳时也常遇到此类情况。判断何时按 F8 何时按 F10？发现当 call 调用地址很近时（如上述 call 5A2006，目标地址紧随其后），应按 F8，否则程序会继续运行，无法达到跟踪目的。相反，若调用地址较远，可用 F10 跳过，但需视具体情况而定。
BitArts:005A2006                 pop     ebp
BitArts:005A2007                 sub     ebp, 6
BitArts:005A200A                 mov     eax, ebp
BitArts:005A200C                 push    ebp
BitArts:005A200D                 pusha
BitArts:005A200E                 mov     [ebp+348Ah], ebp
BitArts:005A2014                 sub     eax, [ebp+3465h]
BitArts:005A201A                 mov     [ebp+2519h], eax
BitArts:005A2020                 push    ebp
BitArts:005A2021                 mov     ebx, 1ECDh
BitArts:005A2026                 add     ebx, ebp
BitArts:005A2028                 push    ebx
BitArts:005A2029                 push    dword ptr fs:0
BitArts:005A202F                 mov     fs:0, esp
BitArts:005A2035                 cmp     byte ptr [ebp+3690h], 0
BitArts:005A203C                 jnz     short loc_5A2047
BitArts:005A203E                 mov     byte ptr [ebp+3690h], 1
BitArts:005A2045                 jmp     short loc_5A205C
```

### 2. 跳过解压过程

随后持续按 `F10`，但需密切关注 `ECX` 寄存器数据，因为 `ECX` 通常作为计数器使用。通过观察 `ECX` 并利用 `F7` 命令分段跳过解压流程。

此处有个小技巧：假设当前位置为：

`5A611C: jmp 005A6074` <----- 当前位置。

`5A6121: mov ecx, 13`

可见该 `jmp` 为向前跳转。此时按 `F6`，将光标移至 `5A6121` 处，再按 `F7`。通常情况下可安全着陆。在脱 `aspr` 壳时，此方法也可用于跳过重复步骤。

从 `5A6121` 起再跳过一个 `jmp`，即可快速到达程序真正入口，此时会出现 `jmp eax`。

### 3. Dump 与修复

执行命令 `makepe dump.exe` 即可生成 dump 文件，至此脱壳工作完成。

运行 `dump.exe`，发现 `logo` 显示后程序立即退出。使用 `trw` 分别载入 `dump.exe` 和未脱壳程序进行对比跟踪，发现以下代码段：

```assembly
:004C9C05 BA9CA64C00             mov edx, 004CA69C
:004C9C0A 8B45FC                mov eax, dword ptr [ebp-04]
:004C9C0D E8CAA10100            call 004E3DDC
:004C9C12 8B45A4                mov eax, dword ptr [ebp-5C]
:004C9C15 50                    push eax
:004C9C16 8D559C                lea edx, dword ptr [ebp-64]
:004C9C19 A1787C4E00            mov eax, dword ptr [004E7C78]
:004C9C1E 8B00                  mov eax, dword ptr [eax]
:004C9C20 E8A78BF8FF            call 004527CC
:004C9C25 8B459C                mov eax, dword ptr [ebp-64]
:004C9C28 8D55A0                lea edx, dword ptr [ebp-60]
:004C9C2B E878ECF3FF            call 004088A8
:004C9C30 8B55A0                mov edx, dword ptr [ebp-60]
:004C9C33 58                    pop eax
:004C9C34 E86BA5F3FF            call 004041A4
:004C9C39 85C0                  test eax, eax
:004C9C3B 7F05                  jg 004C9C42      <----- 此处需跳过。
:004C9C3D E81E9EF3FF            call 00403A60     <----- 此处会导致退出，未测试全 nop 是否可行。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C9C3B(C)

:004C9C42 6A00                  push 00000000
:004C9C44 6880000000            push 00000080
:004C9C49 6A03                  push 00000003
:004C9C4B 6A00                  push 00000000
:004C9C4D 6A03                  push 00000003
:004C9C4F 68000000C0            push C0000000
:004C9C54 8D4D98                lea ecx, dword ptr [ebp-68]
```

修复后再次运行，未发现异常，脱壳成功。

---
小球
2002.5.4
`4dd466d0e1e0cc123afbe504da2462ca`
