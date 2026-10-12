
# 深思Ⅲ加密狗 by 大老

> 原文：[`www.pediy.com/kssd/pediy05/pediy50161.htm`](https://www.pediy.com/kssd/pediy05/pediy50161.htm)

目标：`XX CAD`设计软件。

加密形式：深思Ⅲ加密狗。

破解工具：`Winice`, `Hiew`, `Wdasm893` 中文版。

作者：`sworm`

## 破解过程

### 1. 运行程序
运行程序，显示“Internal Error. 软件出现致命错误,请检查加密狗是否正确!”后退出。

### 2. 调试程序
在 `Wice` 中 `Bpx Messageboxa`，再运行程序，显示上述信息时弹出。按 `F12` 若干次回到调用处，可见是 `xxxxxrx` 调用 `ACAD.acrx_abort`。

### 3. 反汇编分析
反汇编 `xxxxxrx.arx` 文件，得：

```
Exported fn(): acrxEntryPoint - Ord:0002h

:1C05CF00 8B442404                 mov eax, dword ptr [esp+04]
:1C05CF04 48                       dec eax
:1C05CF05 83F804                   cmp eax, 00000004
:1C05CF08 0F878C000000             ja 1C05CF9A
:1C05CF0E FF2485A0CF051C           jmp dword ptr [4*eax+1C05CFA0]
:1C05CF15 8B442408                 mov eax, dword ptr [esp+08]
:1C05CF19 50                       push eax
* Reference To: ACAD.acrxUnlockApplication, Ord:0D5Bh
                                     |
:1C05CF1A E8BDC00800               Call 1C0E8FDC
:1C05CF1F 83C404                   add esp, 00000004
:1C05CF22 E8C9AAFEFF               call 1C0479F0
:1C05CF27 85C0                     test eax, eax
:1C05CF29 7505                     jne 1C05CF30
:1C05CF2B E8C0AAFEFF               call 1C0479F0
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1C05CF29(C)
|
:1C05CF30 E84BFEFFFF               call 1C05CD80
:1C05CF35 E8A6A8FEFF               call 1C0477E0
:1C05CF3A E8019CFEFF               call 1C046B40
:1C05CF3F A388E00F1C               mov dword ptr [1C0FE088], eax
:1C05CF44 85C0                     test eax, eax
:1C05CF46 7521                     jne 1C05CF69
:1C05CF48 6A00                     push 00000000
:1C05CF4A 6A04                     push 00000004
:1C05CF4C E82FAAFEFF               call 1C047980
:1C05CF51 83C408                   add esp, 00000008
:1C05CF54 E807ABFEFF               call 1C047A60
* Possible StringData Ref from Data Obj ->"
软件出现致命错误,请检查加密狗是否正确!"＝＝＝＝＝＝＝＝>就在这！
                                     |
:1C05CF59 6888560F1C               push 1C0F5688
* Reference To: ACAD.acrx_abort, Ord:0D5Dh
                                     |
:1C05CF5E E8CBBF0800               Call 1C0E8F2E
:1C05CF63 83C404                   add esp, 00000004
:1C05CF66 33C0                     xor eax, eax
:1C05CF68 C3                       ret
```

### 4. 进一步分析
在显示错误前，`ACAD.acrxUnlockApplication` 下面，有：

```
1C05CF22 E8C9AAFEFF               call 1C0479F0-------看 call 1C046B40 也可
```

查看该处指令，见：

```
:1C0479F0 83EC60                   sub esp, 00000060
:1C0479F3 E888FB0100               call 1C067580
:1C0479F8 85C0                     test eax, eax
:1C0479FA 7507                     jne 1C047A03-------------à是否为 TDMD 狗？
:1C0479FC B801000000               mov eax, 00000001
:1C047A01 EB31                     jmp 1C047A34
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1C0479FA(C)
|
:1C047A03 66C7442404AF07           mov [esp+04], 07AF―――>应用口令
:1C047A0A 66C74424060700           mov [esp+06], 0007―――>应用口令
:1C047A11 66C74424081A00           mov [esp+08], 001A―――>应用口令
:1C047A18 66C7442402FFFF           mov [esp+02], FFFF――――>开锁
:1C047A1F 8D442400                 lea eax, dword ptr [esp]
:1C047A23 50                       push eax
:1C047A24 E817170A00               call 1C0E9140―――――★
:1C047A29 66837C240001             cmp word ptr [esp], 0001
:1C047A2F 1BC0                     sbb eax, eax
:1C047A31 83E002                   and eax, 00000002
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1C047A01(U)
|
:1C047A34 83F801                   cmp eax, 00000001
:1C047A37 7509                     jne 1C047A42
:1C047A39 B801000000               mov eax, 00000001
:1C047A3E 83C460                   add esp, 00000060
:1C047A41 C3                       ret
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1C047A37(C)
|
:1C047A42 83F802                   cmp eax, 00000002―――――>是否 sense3 狗
:1C047A45 7511                     jne 1C047A58
:1C047A47 E874560100               call 1C05D0C0
:1C047A4C 663D0100                 cmp ax, 0001
:1C047A50 1BC0                     sbb eax, eax
:1C047A52 83C460                   add esp, 00000060
:1C047A55 F7D8                     neg eax
:1C047A57 C3                       ret
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1C047A45(C)
|
:1C047A58 33C0                     xor eax, eax
:1C047A5A 83C460                   add esp, 00000060
:1C047A5D C3                       ret
```

标志处即为 `sense3` 狗操作函数。该 `[esp]=0` 有狗。又是一个拿生日作口令的！

### 5. 查看 1C0E9140 处程序
见：

```
* Referenced by a CALL at Addresses
|:1C046774 , :1C0467EA , :1C046A26 , :1C046B75 , :1C046C96
|:1C046DD6 , :1C046F16 , :1C047065 , :1C047176 , :1C0472B6
|:1C047817 , :1C047A24 , :1C047A94 , :1C05D0E9 , :1C05D10E
|:1C05D1D6 , :1C05D2B0
|
:1C0E9140 8B442404                 mov eax, dword ptr [esp+04]
:1C0E9144 6A01                     push 00000001
:1C0E9146 50                       push eax
:1C0E9147 E864020000               call 1C0E93B0
:1C0E914C 83C408                   add esp, 00000008
:1C0E914F C20400                   ret 0004
```

由 reference 表，知有 17 处加密狗操作。

前 14 处均为开锁操作，第 15 处为关锁操作，最后两处为狗操作，必然在开锁操作之后，我们随便观察一处开锁：

```
* Referenced by a CALL at Addresses:
|:1C0101AA , :1C0156C0 , :1C026A29 , :1C05CF3A , :1C06D469
|:1C0720C7 , :1C0CC80A , :1C0D65DA
|
:1C046B40 83EC64                   sub esp, 00000064
:1C046B43 57                       push edi
:1C046B44 E8370A0200               call 1C067580
:1C046B49 85C0                     test eax, eax\-------------à是否为 TDMD 狗？
:1C046B4B 7507                     jne 1C046B54
:1C046B4D B801000000               mov eax, 00000001
:1C046B52 EB31                     jmp 1C046B85
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1C046B4B(C)
|
:1C046B54 66C7442408AF07           mov [esp+08], 07AF
:1C046B5B 66C744240A0700           mov [esp+0A], 0007
:1C046B62 66C744240C1A00           mov [esp+0C], 001A
:1C046B69 66C7442406FFFF           mov [esp+06], FFFF
:1C046B70 8D442404                 lea eax, dword ptr [esp+04]
:1C046B74 50                       push eax
:1C046B75 E8C6250A00               call 1C0E9140―――sense3 函数
:1C046B7A 66837C240401             cmp word ptr [esp+04], 0001
:1C046B80 1BC0                     sbb eax, eax
:1C046B82 83E002                   and eax, 00000002
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1C046B52(U)
|
:1C046B85 83F801                   cmp eax, 00000001
:1C046B88 0F8584000000             jne 1C046C12
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1C046B88(C)
|
:1C046C12 83F802                   cmp eax, 00000002―――sense3 狗？
:1C046C15 753F                     jne 1C046C56
:1C046C17 68164C0000               push 00004C16
:1C046C1C 684B110000               push 0000114B
:1C046C21 E84A660100               call 1C05D270――――干吗的？
:1C046C26 83C408                   add esp, 00000008
:1C046C29 8BC8                     mov ecx, eax
:1C046C2B 81E1FFFF0000             and ecx, 0000FFFF
:1C046C31 250000FFFF               and eax, FFFF0000
:1C046C36 3D0000DF00               cmp eax, 00DF0000
:1C046C3B 7512                     jne 1C046C4F
:1C046C3D 81F937220000             cmp ecx, 00002237
:1C046C43 750A                     jne 1C046C4F
:1C046C45 B801000000               mov eax, 00000001――好狗由此返回
:1C046C4A 5F                       pop edi
:1C046C4B 83C464                   add esp, 00000064
:1C046C4E C3                       ret
```
