
# chess explorer 6.0 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-283.htm`](https://www.pediy.com/kssd/pediy09/pediy09-283.htm)

> 使用工具：
>
> 反编译工具：`PEid`, `IDA Pro`, `Resource Hacker`
>
> 程序开发包：`MASM32v9`

### 0. 前言

`chess explorer` 是一个用于解答或创作国际象棋棋题的工具，`6.0` 版是其最新版。可以从网上搜到其主页及下载地址，这里就不给出了。

### 1. 反编译

当你拿到一个软件准备破解的时候，首先想到的工具是什么？（调试器，`OllyDBG`……）如果你是这么想的，那么你还处于刚入门的阶段，或者说，你的经验还浅。这也难怪，毕竟现在网络上流行的破解教程的第一课就是教读者如何使用调试器。然而要知道，当你在调试器中把程序跑起来的时候，风险也就随之而来。时至今日，共享软件中包含反调试代码是司空见惯的事。这些代码可能仅仅将你的调试器关闭，可能会将你的系统关闭，让你的未保存数据丢失。即便如此，这些都还算好的情况。万一碰到软件作者人品恶劣，在检测到调试器后释放一个病毒（或者反调试代码本身就是病毒），或者把你的硬盘格式化，造成的后果将不堪设想。

诚然，调试器的强大功能使得它们在软件分析中必不可少。但是在动用调试器之前，应当首先考虑静态分析的可行性。只有静态分析行不通时，才能动用这最后的武器。

为此，我们先从文件信息侦测开始。如果文件加了壳，就要先把壳脱掉才能进行静态分析。如果没办法脱掉壳，就只有带着壳在调试器里跑，承担由此带来的风险了。`PEid` 显示这个软件是用 `Borland C++` 编写的，没有加壳。

关闭一切与反编译相关的程序（包括反汇编器、调试器、`PEid` 等等），试运行待分析程序，以便了解其注册机制。我们发现，在程序主窗口菜单的 `"Info"` 菜单下有一项 `"Registering"`，单击将弹出注册对话框。

现在用 `Resource Hacker` 打开待分析程序，找到主菜单，我们看到 `"Registering"` 菜单项，其 ID 为 `143`。更进一步可以发现对话框资源中包含了“注册”对话框，其 ID 为 `600`。

这一来，我们至少有两种思路追踪到注册算法部分的代码。一是跟踪窗口过程的 `WM_COMMAND` 消息，找到 `wParam` 等于 `143` 的分支并跟进去。二是查找对 `DialogBoxParam` 或 `CreateDialogParam` 的引用（如果了解到“注册”对话框是模态的，则可以只查找对前者的引用），找到第二个参数等于 `600` 的那个调用，并且查看该调用的第四个参数，它就是“注册”对话框回调过程的地址。

由于第二种方法较为简便，我们采用此方法。用 `IDA` 加载程序，迅速定位到“注册”对话框回调过程，进入其中的 `WM_COMMAND` 消息分支，当 `"OK"` 按钮（`wParam == 1`）被按下时，来到：

```
代码:
```



```
loc_4079E2:        ; CODE XREF: _ProcDlgReg+D6j
    mov  ecx, glb_J
    mov  [ebp+ecx+@szBuffer1], 0
    inc  glb_J

loc_4079F3:        ; CODE XREF: _ProcDlgReg+BCj
    cmp  glb_J, 18h
    jl  short loc_4079E2
    lea  eax, [eax+ebx+4Dh]
    lea  edi, [edi+eax*2]
    mov  eax, edi
    movsx  eax, [ebp+@szBuffer1]
    mov  [ebp+var_4], eax ; var_4 = 0
    movsx  esi, [ebp+@szBuffer1+1]
    inc  esi
    mov  edi, esi
    shl  edi, 2    ; edi = 4
    add  esi, edi  ; esi = 5
    push  29h    ; nMaxCount
    lea  eax, [ebp+@lpszUserName]
    push  eax    ; lpString
    push  259h    ; nIDDlgItem
    push  [ebp+hDlg]  ; hDlg
    call  GetDlgItemTextA

    push  9    ; nMaxCount
    lea  edx, [ebp+@lpszSerial]
    push  edx    ; lpString
    push  25Ah    ; nIDDlgItem
    push  [ebp+hDlg]  ; hDlg
    call  GetDlgItemTextA

    lea  ecx, [ebp+@lpszUserName]
    push  ecx    ; lpString
    call  lstrlenA

    mov  [ebp+@uLenOfName], eax
    lea  eax, [ebp+@lpszSerial]
    push  eax    ; lpString
    call  lstrlenA

    cmp  eax, 8    ; 序列号必须是 8 个字符长
    jnz  short loc_407A64

    cmp  [ebp+@uLenOfName], 0Ch
    jge  short loc_407A82 ; 用户名不得少于 12 个字符？

loc_407A64:        ; CODE XREF: _ProcDlgReg+138j
    push  10h    ; uType
    push  offset Caption  ; "ChessExplorer"
    push  offset s_IncorrectPass ; "Incorrect password"
    push  [ebp+hDlg]  ; hWnd
    call  MessageBoxA

    mov  eax, 1
    jmp  loc_408267

; ---------------------------------------------------------------------------
loc_407A82:        ; CODE XREF: _ProcDlgReg+13Ej
    xor  edx, edx
    mov  glb_J, edx
    jmp  short loc_407A9F

; ---------------------------------------------------------------------------
loc_407A8C:        ; CODE XREF: _ProcDlgReg+182j
    mov  ecx, glb_J
    mov  szBuffer2[ecx],  84h
    inc  glb_J

loc_407A9F:        ; CODE XREF: _ProcDlgReg+166j
    cmp  glb_J, 27h
    jle  short loc_407A8C

    mov  eax, esi
    imul  edi
    cmp  eax, [ebp+@uLenOfName]
    jle  loc_407C1C  ; 用户名达到或超过 20 个字符时转移

    movsx  eax, [ebp+edi+@lpszUserName]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    movsx  eax, [ebp+edx+@lpszUserName+1]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+edi+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407AE8

    mov  edx, esi
    mov  [ebp+@szBuffer1], dl

loc_407AE8:        ; CODE XREF: _ProcDlgReg+1BDj
    mov  ecx, [ebp+@uLenOfName]
    movsx  eax, [ebp+ecx+var_A6+4]
    add  eax, [ebp+@uLenOfName]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial+1]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407B16

    mov  edx, edi
    mov  ecx, esi
    sub  ecx, edi
    mov  [ebp+ecx+@szBuffer1], dl

loc_407B16:        ; CODE XREF: _ProcDlgReg+1E6j
    mov  eax, [ebp+@uLenOfName]
    movsx  eax, [ebp+eax+var_A6+7]
    inc  eax
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+@szBuffer1]
    movsx  eax, [ebp+eax+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407B45

    mov  edx, esi
    mov  ecx, edi
    add  dl, cl
    mov  [ebp+@szBuffer1+2], dl

loc_407B45:        ; CODE XREF: _ProcDlgReg+216j
    movsx  eax, [ebp+@lpszUserName+10h]
    add  eax, 24h
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+@lpszSerial+7]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407B68

    mov  [ebp+@szBuffer1+3], 7

loc_407B68:        ; CODE XREF: _ProcDlgReg+23Ej
    movsx  eax, [ebp+@lpszUserName+2]
    mov  edx, [ebp+@uLenOfName]
    movsx  ecx, [ebp+edx+var_9E]
    add  eax, ecx
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+edi+var_18+2]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407B98

    mov  edx, edi
    mov  [ebp+edi+@szBuffer1], dl

loc_407B98:        ; CODE XREF: _ProcDlgReg+26Cj
    movsx  eax, [ebp+@lpszUserName+9]
    add  eax, 7
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    mov  eax, esi
    sub  eax, edi
    movsx  eax, [ebp+eax+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407BC2

    mov  edx, esi
    mov  [ebp+esi+@szBuffer1], dl

loc_407BC2:        ; CODE XREF: _ProcDlgReg+296j
    movsx  eax, [ebp+@lpszUserName+7]
    add  eax, 3
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+@lpszSerial+3]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407BE8

    mov  edx, esi
    inc  edx
    mov  [ebp+esi+@szBuffer1+1],  dl

loc_407BE8:        ; CODE XREF: _ProcDlgReg+2BBj
    movsx  eax, [ebp+esi*2+@lpszUserName]
    add  eax, 5
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  loc_40804C

    mov  edx, esi
    mov  ecx, edi
    sub  dl, cl
    mov  [ebp+esi+@szBuffer1+2],  dl
    jmp  loc_40804C

; ---------------------------------------------------------------------------
loc_407C1C:        ; CODE XREF: _ProcDlgReg+18Bj
    mov  eax, esi
    imul  edi
    add  eax, 6
    cmp  eax, [ebp+@uLenOfName]
    jle  loc_407D98

    movsx  eax, [ebp+@lpszUserName+11h]
    mov  edx, [ebp+@uLenOfName]
    movsx  ecx, [ebp+edx+var_A6+3]
    add  eax, ecx
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    mov  eax, esi
    sub  eax, edi
    movsx  eax, [ebp+eax+var_18+3]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407C5F

    mov  edx, esi
    mov  [ebp+@szBuffer1], dl

loc_407C5F:        ; CODE XREF: _ProcDlgReg+334j
    movsx  eax, [ebp+@lpszUserName]
    mov  edx, [ebp+@uLenOfName]
    movsx  ecx, [ebp+edx+var_9D]
    add  eax, ecx
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial+1]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407C93

    mov  edx, edi
    mov  ecx, esi
    sub  ecx, edi
    mov  [ebp+ecx+@szBuffer1], dl

loc_407C93:        ; CODE XREF: _ProcDlgReg+363j
    movsx  eax, [ebp+@lpszUserName+0Bh]
    add  eax, 2
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+edi+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407CBC

    mov  edx, esi
    mov  ecx, edi
    add  dl, cl
    mov  [ebp+@szBuffer1+2], dl

loc_407CBC:        ; CODE XREF: _ProcDlgReg+38Dj
    movsx  eax, [ebp+@lpszUserName+0Dh]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial+2]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407CDD

    mov  [ebp+@szBuffer1+3], 7

loc_407CDD:        ; CODE XREF: _ProcDlgReg+3B3j
    movsx  eax, [ebp+@lpszUserName+7]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    movsx  eax, [ebp+edx+@lpszUserName+3]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+edi+var_18+2]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407D10

    mov  edx, edi
    mov  [ebp+edi+@szBuffer1], dl

loc_407D10:        ; CODE XREF: _ProcDlgReg+3E4j
    mov  ecx, [ebp+@uLenOfName]
    movsx  eax, [ebp+ecx+var_A6]
    add  eax, 3
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    mov  eax, esi
    sub  eax, edi
    movsx  eax, [ebp+eax+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407D3E

    mov  edx, esi
    mov  [ebp+esi+@szBuffer1], dl

loc_407D3E:        ; CODE XREF: _ProcDlgReg+412j
    movsx  eax, [ebp+@lpszUserName+3]
    add  eax, 13h
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+@lpszSerial+3]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407D64

    mov  edx, esi
    inc  edx
    mov  [ebp+esi+@szBuffer1+1],  dl

loc_407D64:        ; CODE XREF: _ProcDlgReg+437j
    movsx  eax, [ebp+edi+@lpszUserName]
    add  eax, [ebp+@uLenOfName]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  loc_40804C

    mov  edx, esi
    mov  ecx, edi
    sub  dl, cl
    mov  [ebp+esi+@szBuffer1+2],  dl
    jmp  loc_40804C

; ---------------------------------------------------------------------------
loc_407D98:        ; CODE XREF: _ProcDlgReg+302j
    mov  eax, esi
    imul  edi
    mov  edx, esi
    add  edx, edx
    add  eax, edx
    add  eax, edi
    cmp  eax, [ebp+@uLenOfName]
    jle  loc_407EFB

    mov  ecx, [ebp+@uLenOfName]
    movsx  eax, [ebp+ecx+var_9D]
    add  eax, 2
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    mov  eax, esi
    sub  eax, edi
    movsx  eax, [ebp+eax+var_18+3]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407DDA

    mov  edx, esi
    mov  [ebp+@szBuffer1], dl

loc_407DDA:        ; CODE XREF: _ProcDlgReg+4AFj
    movsx  eax, [ebp+@lpszUserName+5]
    add  eax, [ebp+@uLenOfName]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial+1]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407E04

    mov  edx, edi
    mov  ecx, esi
    sub  ecx, edi
    mov  [ebp+ecx+@szBuffer1], dl

loc_407E04:        ; CODE XREF: _ProcDlgReg+4D4j
    movsx  eax, [ebp+@lpszUserName+0Dh]
    add  eax, [ebp+@uLenOfName]
    add  eax, -3
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+edi+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407E30

    mov  edx, esi
    mov  ecx, edi
    add  dl, cl
    mov  [ebp+@szBuffer1+2], dl

loc_407E30:        ; CODE XREF: _ProcDlgReg+501j
    movsx  eax, [ebp+@lpszUserName+10h]
    add  eax, -2
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial+2]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407E54

    mov  [ebp+@szBuffer1+3], 7

loc_407E54:        ; CODE XREF: _ProcDlgReg+52Aj
    mov  edx, [ebp+@uLenOfName]
    movsx  eax, byte ptr [ebp+edx-0A5h]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+edi+var_18+2]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407E7B

    mov  edx, edi
    mov  [ebp+edi+@szBuffer1], dl

loc_407E7B:        ; CODE XREF: _ProcDlgReg+54Fj
    movsx  eax, [ebp+@lpszUserName+8]
    add  eax, 2
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    mov  eax, esi
    sub  eax, edi
    movsx  eax, [ebp+eax+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407EA5

    mov  edx, esi
    mov  [ebp+esi+@szBuffer1], dl

loc_407EA5:        ; CODE XREF: _ProcDlgReg+579j
    movsx  eax, [ebp+@lpszUserName+1]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+@lpszSerial+3]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407EC8

    mov  edx, esi
    inc  edx
    mov  [ebp+esi+@szBuffer1+1],  dl

loc_407EC8:        ; CODE XREF: _ProcDlgReg+59Bj
    movsx  eax, [ebp+@lpszUserName+17h]
    add  eax, 7
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  loc_40804C

    mov  edx, esi
    mov  ecx, edi
    sub  dl, cl
    mov  [ebp+esi+@szBuffer1+2],  dl
    jmp  loc_40804C

; ---------------------------------------------------------------------------
loc_407EFB:        ; CODE XREF: _ProcDlgReg+483j
    mov  eax, [ebp+@uLenOfName]
    movsx  eax, byte ptr [ebp+eax-0A1h]
    add  eax, -2
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    mov  eax, esi
    sub  eax, edi
    movsx  eax, [ebp+eax+var_18+3]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407F28

    mov  edx, esi
    mov  [ebp+@szBuffer1], dl

loc_407F28:        ; CODE XREF: _ProcDlgReg+5FDj
    mov  ecx, [ebp+@uLenOfName]
    movsx  eax, [ebp+ecx+var_A6]
    add  eax, -3
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial+1]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407F56

    mov  edx, edi
    mov  ecx, esi
    sub  ecx, edi
    mov  [ebp+ecx+@szBuffer1], dl

loc_407F56:        ; CODE XREF: _ProcDlgReg+626j
    movsx  eax, [ebp+@lpszUserName+14h]
    add  eax, 6
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+edi+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407F7F

    mov  edx, esi
    mov  ecx, edi
    add  dl, cl
    mov  [ebp+@szBuffer1+2], dl

loc_407F7F:        ; CODE XREF: _ProcDlgReg+650j
    movsx  eax, [ebp+@lpszUserName+11h]
    add  eax, [ebp+@uLenOfName]
    inc  eax
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial+2]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407FA4

    mov  [ebp+@szBuffer1+3], 7

loc_407FA4:        ; CODE XREF: _ProcDlgReg+67Aj
    movsx  eax, [ebp+@lpszUserName]
    mov  edx, [ebp+@uLenOfName]
    movsx  ecx, [ebp+edx+var_9D]
    add  eax, ecx
    add  eax, -2
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+edi+var_18+2]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_407FD7

    mov  edx, edi
    mov  [ebp+edi+@szBuffer1], dl

loc_407FD7:        ; CODE XREF: _ProcDlgReg+6ABj
    movsx  eax, [ebp+@lpszUserName+7]
    add  eax, 2
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    mov  eax, esi
    sub  eax, edi
    movsx  eax, [ebp+eax+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_408001

    mov  edx, esi
    mov  [ebp+esi+@szBuffer1], dl

loc_408001:        ; CODE XREF: _ProcDlgReg+6D5j
    movsx  eax, [ebp+@lpszUserName+0Eh]
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+@lpszSerial+3]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_408024

    mov  edx, esi
    inc  edx
    mov  [ebp+esi+@szBuffer1+1],  dl

loc_408024:        ; CODE XREF: _ProcDlgReg+6F7j
    movsx  eax, [ebp+@lpszUserName+8]
    inc  eax
    mov  ecx, 0Ah
    cdq
    idiv  ecx
    mov  ebx, edx
    movsx  eax, [ebp+esi+@lpszSerial]
    add  eax, -30h
    cmp  eax, ebx
    jz  short loc_40804C

    mov  edx, esi
    mov  ecx, edi
    sub  dl, cl
    mov  [ebp+esi+@szBuffer1+2],  dl

loc_40804C:        ; CODE XREF: _ProcDlgReg+2E3j
          ; _ProcDlgReg+2F3j _ProcDlgReg+45Fj
          ; _ProcDlgReg+46Fj _ProcDlgReg+5C2j
          ; _ProcDlgReg+5D2j _ProcDlgReg+71Cj
    xor  esi, esi
    xor  edi, edi
    xor  ebx, ebx
    xor  eax, eax
    lea  edx, [edi+esi]
    mov  [ebp+var_4], edx
    mov  ecx, ebx
    shl  ecx, 2
    lea  edx, [edi+edi*2]
    add  ecx, edx
    mov  glb_J, ecx  ; 除 ebp 和 esp 以外的通用寄存器全清零
    movsx  edi, [ebp+@szBuffer1+3]
    movsx  edx, [ebp+@szBuffer1+1]
    add  edi, edx
    movsx  ecx, [ebp+@szBuffer1+5]
    add  edi, ecx
    lea  edx, [eax+ebx]
    mov  dword_4827D4, edx
    cmp  eax, esi
    jnz  short loc_40809E

    mov  eax, esi
    add  eax, eax
    movsx  ecx, [ebp+@szBuffer1]
    add  eax, ecx
    movsx  edx, [ebp+@szBuffer1+2]
    lea  ecx, [edx+edx*2]
    add  eax, ecx
    add  eax, ebx
    jmp  short loc_4080A3

; ---------------------------------------------------------------------------
loc_40809E:        ; CODE XREF: _ProcDlgReg+761j
    add  [ebp+var_4], esi
    jmp  short loc_4080C3

; ---------------------------------------------------------------------------
loc_4080A3:        ; CODE XREF: _ProcDlgReg+778j
    cmp  eax, [ebp+var_4]
    jnz  short loc_4080FD

    xor  edx, edx
    mov  glb_J, edx
    jmp  short loc_4080BA

; ---------------------------------------------------------------------------
loc_4080B2:        ; CODE XREF: _ProcDlgReg+79Dj
    add  ebx, esi
    inc  glb_J

loc_4080BA:        ; CODE XREF: _ProcDlgReg+78Cj
    cmp  glb_J, 40h
    jl  short loc_4080B2

loc_4080C3:        ; CODE XREF: _ProcDlgReg+77Dj
    mov  esi, ebx
    add  esi, esi
    sub  esi, [ebp+var_4]
    cmp  esi, [ebp+var_4]
    jz  short loc_4080D3

    jmp  short loc_4080D3

; ---------------------------------------------------------------------------
    jmp  short loc_40811B

; ---------------------------------------------------------------------------
loc_4080D3:        ; CODE XREF: _ProcDlgReg+7A9j
          ; _ProcDlgReg+7ABj
    add  edi, eax
    movsx  eax, [ebp+@szBuffer1+4]
    add  eax, eax
    movsx  edx, [ebp+@szBuffer1+5]
    lea  ecx, [edx+edx*2]
    add  eax, ecx
    movsx  edx, [ebp+@szBuffer1+6]
    shl  edx, 2
    add  eax, edx
    movsx  ecx, [ebp+@szBuffer1+7]
    add  eax, ecx
    add  eax, edi
    cmp  edi, dword_4827D4
    jz  short loc_40811B

loc_4080FD:        ; CODE XREF: _ProcDlgReg+782j
          ; _ProcDlgReg+7FDj
    push  10h    ; uType
    push  offset Caption  ; "ChessExplorer"
    push  offset s_IncorrectPass ; "Incorrect password"
    push  [ebp+hDlg]  ; hWnd
    call  MessageBoxA

    mov  eax, 1
    jmp  loc_408267

; ---------------------------------------------------------------------------
loc_40811B:        ; CODE XREF: _ProcDlgReg+7ADj
          ; _ProcDlgReg+7D7j
    cmp  eax, dword_4827D4
    jnz  short loc_4080FD

    xor  ebx, ebx

loc_408125:        ; CODE XREF: _ProcDlgReg+812j
    mov  al, [ebp+ebx+@lpszUserName]
    mov  szStatusOfReg[ebx], al
    inc  ebx
    cmp  ebx, 28h
    jle  short loc_408125

    xor  ebx, ebx
    jmp  short loc_408153

; ---------------------------------------------------------------------------
loc_40813C:        ; CODE XREF: _ProcDlgReg+83Ej
    mov  al, [ebp+ebx+@lpszUserName]
    add  al, 64h
    mov  edx, 27h
    sub  edx, ebx
    mov  szBuffer2[edx],  al
    inc  ebx

loc_408153:        ; CODE XREF: _ProcDlgReg+816j
    lea  ecx, [ebp+@lpszUserName]
    push  ecx    ; lpString
    call  lstrlenA

    dec  eax
    cmp  ebx, eax
    jle  short loc_40813C

    mov  dword_4EAC5C, offset szBuffer2
    push  FILE_ATTRIBUTE_NORMAL ;  dwFileAttributes
    push  offset Buffer  ; lpFileName
    call  SetFileAttributesA

    push  8001h
    push  offset Buffer
    call  j____open

    add  esp, 8
    mov  dword_4EAC70, eax
    inc  eax
    jnz  short loc_4081AF

    push  0    ; uType
    push  offset Caption  ; "ChessExplorer"
    push  offset s_ErrorOfRegist ; "Error  of registering"
    push  0    ; hWnd
    call  MessageBoxA

    jmp  loc_40823C

; ---------------------------------------------------------------------------
loc_4081AF:        ; CODE XREF: _ProcDlgReg+871j
    push  0
    push  0Ah
    push  dword_4EAC70
    call  j____lseek

    add  esp, 0Ch
    push  28h
    push  dword_4EAC5C
    push  dword_4EAC70
    call  j____write

    add  esp, 0Ch
    mov  [ebp+var_18+2],  4Dh
    mov  [ebp+var_18+3],  11h
    mov  [ebp+@lpszSerial], 0FFh
    lea  edx, [ebp+var_18+2]
    mov  dword_4EAC5C, edx
    push  0
    push  7
    push  dword_4EAC70
    call  j____lseek

    add  esp, 0Ch
    push  3
    push  dword_4EAC5C
    push  dword_4EAC70
    call  j____write

    add  esp, 0Ch
    push  dword_4EAC70
    call  j____close

    pop  ecx
    push  0    ; uType
    push  offset Caption  ; "ChessExplorer"
    push  offset s_Chessexplor_3 ; "ChessExplorer  has been registered"
    push  [ebp+hDlg]  ; hWnd
    call  MessageBoxA
```



可以看到，显示错误信息的地方一共有 3 处，其中最后一处是由于某个文件打开失败而造成的，与注册算法无关。经过反编译后，验证注册码的过程可书写如下：

代码：

```
BOOL VerifyKey (char* lpszUserName, char* lpszSerial) {
  char  loc_szBuffer1 [8];
  char  a1, b1, c1;
  BYTE   a [8], b [4], c [4];
  int   loc_iSum = 0;
  int  loc_uNameLen;
  int   i,j,k;
  for (i = 0; i < 8; i++)
    loc_szBuffer1 [i] = 0;
  if ((loc_uNameLen = lstrlen (lpszUserName)) < 12 || lstrlen (lpszSerial) != 8)
    return FALSE;
  for (i = 0; i < 40; i++)
    szBuffer2 [i] = 0x84;
  if (loc_uNameLen < 20)
  {
    if (lpszUserName [1 + loc_szUserName[4] % 10] % 10 != lpszSerial [4] - '0')
      loc_szBuffer1 [0] = 5;
    if ((lpszUserName [loc_uNameLen - 6] + loc_uNameLen) % 10 != lpszSerial [6] - '0')
      loc_szBuffer1 [1] = 4;
    if ((lpszUserName [loc_uNameLen - 3] + 1) % 10 != lpszSerial [loc_szBuffer1 [0]] - '0')
      loc_szBuffer1 [2] = 9;
    if ((lpszUserName [16] + 36) % 10 != lpszSerial [7] - '0')   //要求用户名长度至少为 17
      loc_szBuffer1 [3] = 7;
    if ((lpszUserName [2] + lpszUserName [loc_uNameLen - 2]) % 10 != lpszSerial [2] - '0')
      loc_szBuffer1 [4] = 4;
    if ((lpszUserName [9] + 7) % 10 != lpszSerial [1] - '0')
      loc_szBuffer1 [5] = 5;
    if ((lpszUserName [7] + 3) % 10 != lpszSerial [3] - '0')
      loc_szBuffer1 [6] = 6;
    if ((lpszUserName [10] + 5) % 10 != lpszSerial [5] - '0')
      loc_szBuffer1 [7] = 1;
  }
  else if (loc_uNameLen < 26)
  {
    if ((lpszUserName [17] + lpszUserName [loc_uNameLen - 7]) % 10 != lpszSerial [0] - '0')
      loc_szBuffer1 [0] = 5;
    if ((lpszUserName [0] + lpszUserName [loc_uNameLen - 1]) % 10 != lpszSerial [6] - '0')
      loc_szBuffer1 [1] = 4;
    if ((lpszUserName [11] + 2) % 10 != lpszSerial [4] - '0')
      loc_szBuffer1 [2] = 9;
    if (lpszUserName [13] % 10 != lpszSerial [7] - '0')
      loc_szBuffer1 [3] = 7;
    if (lpszUserName [3 + lpszUserName [7] % 10] % 10 != lpszSerial [2] - '0')
      loc_szBuffer1 [4] = 4;
    if ((lpszUserName [loc_uNameLen - 10] + 3) % 10 != lpszSerial [1] - '0')
      loc_szBuffer1 [5] = 5;
    if ((lpszUserName [3] + 19) % 10 != lpszSerial [3] - '0')
      loc_szBuffer1 [6] = 6;
    if ((lpszUserName [4] + loc_uNameLen) % 10 != lpszSerial [5] - '0')
      loc_szBuffer1 [7] = 1;
  }
  else if (loc_uNameLen < 34)
  {
    if ((lpszUserName [loc_uNameLen - 1] + 2) % 10 != lpszSerial [0] - '0')
      loc_szBuffer1 [0] = 5;
    if ((lpszUserName [5] + loc_uNameLen) % 10 != lpszSerial [6] - '0')
      loc_szBuffer1 [1] = 4;
    if ((lpszUserName [13] + loc_uNameLen - 3) % 10 != lpszSerial [4] - '0')
      loc_szBuffer1 [2] = 9;
    if ((lpszUserName [16] - 2) % 10 != lpszSerial [7] - '0')
      loc_szBuffer1 [3] = 7;
    if (lpszUserName [loc_uNameLen - 9] % 10 != lpszSerial [2] - '0')
      loc_szBuffer1 [4] = 4;
    if ((lpszUserName [8] + 2) % 10 != lpszSerial [1] - '0')
      loc_szBuffer1 [5] = 5;
    if (lpszUserName [1] % 10 != lpszSerial [3] - '0')
      loc_szBuffer1 [6] = 6;
    if ((lpszUserName [23] + 7) % 10 != lpszSerial [5] - '0')
      loc_szBuffer1 [7] = 1;
  }
  else
  {
    if ((lpszUserName [loc_uNameLen - 5] - 2) % 10 != lpszSerial [0] - '0')
      loc_szBuffer1 [0] = 5;
    if ((lpszUserName [loc_uNameLen - 10] - 3) % 10 != lpszSerial [6] - '0')
      loc_szBuffer1 [1] = 4;
    if ((lpszUserName [20] + 6) % 10 != lpszSerial [4] - '0')
      loc_szBuffer1 [2] = 9;
    if ((lpszUserName [17] + loc_uNameLen + 1) % 10 != lpszSerial [7] - '0')
      loc_szBuffer1 [3] = 7;
    if ((lpszUserName [0] + lpszUserName [loc_uNameLen - 1] - 2) % 10 != lpszSerial [2] - '0')
      loc_szBuffer1 [4] = 4;
    if ((lpszUserName [7] + 2) % 10 != lpszSerial [1] - '0')
      loc_szBuffer1 [5] = 5;
    if (lpszUserName [14] % 10 != lpszSerial [3] - '0')
      loc_szBuffer1 [6] = 6;
    if ((lpszUserName [8] + 1) % 10 != lpszSerial [5] - '0')
      loc_szBuffer1 [7] = 1;
  }
  loc_iSum = loc_szBuffer1 [0] + loc_szBuffer1 [2] * 3;
  if (loc_iSum != 0)
    return FALSE;
  loc_iSum += loc_szBuffer1 [3] + loc_szBuffer1 [1] + loc_szBuffer1 [5];
  if (loc_iSum != 0)
    return FALSE;
  loc_iSum += loc_szBuffer1 [4] * 2 + loc_szBuffer1 [5] * 3 + loc_szBuffer1 [6] * 4 + loc_szBuffer1 [7];
  if (loc_iSum != 0)
    return FALSE;
  return TRUE;
}
```

## 2. 算法总结

注册算法并不复杂，只是相当琐碎。首先它要求序列号一定是 8 个字符，用户名不得少于 12 个字符，否则不加进一步验证而否决。当用户名长度小于 20、大于或等于 20 而小于 26、大于或等于 26 而小于 34、大于或等于 34 时，分别做不同情况处理，抽取用户名中某些特定的字符来做一些特定的运算，然后看运算结果是否等于序列号中的特定字符。只有在上面反编译出的代码中，`if-elseif-elseif-else` 这个分支结构的每个分支中，8 个 `if` 条件都不成立，即不等号应换成等号时，序列号才是正确的。

由此可见，序列号完全由十进制数码字符组成。此外，用户名长度小于 20 的分支里，由于引用了 `lpszUserName[16]`，因此当用户名少于 17 个字符时，其行为是未定义的。编写注册机时建议对于 17 个字符以下的用户名不予生成序列号。

注册码生成函数可编写如下：

代码：



```asm
_KeyGen       proc            uses ebx esi edi _lpszUserName, _lpszSerial
                ; 生成序列号
                ; 入口参数: _lpszUserName 指向给定的用户名的指针
                ; 出口参数: _lpszSerial 指向所生成序列号的缓冲区指针
                ; 返回值: eax 等于 1 表示成功，0 表示失败

                mov           esi, _lpszUserName
                invoke        lstrlen, esi
                mov           ebx, eax
                mov           ecx, 10
                mov           edi, _lpszSerial

                .if           eax < 17
                    xor       eax, eax
                    jmp       locret_1

                .elseif       eax < 20
                    movsx     eax, byte ptr [esi+ebx-3]
                    inc       eax
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+9]
                    add       eax, 7
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+2]
                    movsx     edx, byte ptr [esi+ebx-2]
                    add       eax, edx
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+7]
                    add       eax, 3
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+4]
                    cdq
                    idiv      ecx
                    inc       edx
                    movsx     eax, byte ptr [esi+edx]
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+10]
                    add       eax, 5
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+ebx-6]
                    add       eax, ebx
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+16]
                    add       eax, 36
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                .elseif       eax < 26
                    movsx     eax, byte ptr [esi+17]
                    movsx     edx, byte ptr [esi+ebx-7]
                    add       eax, edx
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+ebx-10]
                    add       eax, 3
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+7]
                    cdq
                    idiv      ecx
                    movsx     eax, byte ptr [esi+edx+3]
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+3]
                    add       eax, 19
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+11]
                    add       eax, 2
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+4]
                    add       eax, ebx
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi]
                    movsx     edx, byte ptr [esi+ebx-1]
                    add       eax, edx
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+13]
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                .elseif       eax < 34
                    movsx     eax, byte ptr [esi+ebx-1]
                    add       eax, 2
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+8]
                    add       eax, 2
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+ebx-9]
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+1]
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+13]
                    lea       eax, [eax+ebx-3]
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+23]
                    add       eax, 7
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+5]
                    add       eax, ebx
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+16]
                    sub       eax, 2
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                .else
                    movsx     eax, byte ptr [esi+ebx-5]
                    sub       eax, 2
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+7]
                    add       eax, 2
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi]
                    movsx     edx, byte ptr [esi+ebx-1]
                    lea       eax, [eax+edx-2]
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+14]
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+20]
                    add       eax, 6
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+8]
                    inc       eax
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+ebx-10]
                    sub       eax, 3
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                    movsx     eax, byte ptr [esi+17]
                    lea       eax, [eax+ebx+1]
                    cdq
                    idiv      ecx
                    lea       eax, [edx+30h]
                    stosb

                .endif

                xor           eax, eax
                stosb
                inc           eax

locet_1:
                ret
_KeyGen       endp
```



`4dd466d0e1e0cc123afbe504da2462ca`
