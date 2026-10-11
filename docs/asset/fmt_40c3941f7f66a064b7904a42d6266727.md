
# PELock v1.0x 脱壳之完美风暴

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-468.htm`](https://www.pediy.com/kssd/pediy09/pediy09-468.htm)

【脱壳对象】：`PELock` `v1.06` 加壳的 `ACProtect` `v2.0` 自带 `ACP_Feedback.exe`

之所以选择这个对象的原因在于：

1. `PELock` 已经不再升级，其加壳程序的兼容性也随着操作系统的升级而下降，因此现在用 `PELock` 加壳的程序也难觅其踪；
2. `ACP_Feedback.exe` 是一个 Delphi 程序，`PELock` 对 Delphi 程序的一个地址表做了特殊处理，我们可以借此机会一觑这个特别的保护手段:)

【作者】：`jdpack`

【联系方式】：`redoc` at `citiz.net`

【脱壳工具】：`OllyDbg`，`LordPE`，`ImportRec` 等等

【脱壳过程】：

`OllyDbg` 载入 `ACP_Feedback.exe`，忽略除内存访问异常以外的所有异常，`Shift+F9` 12 次以后程序启动。

重新载入程序，这次我们 `Shift+F9` 11 次以后对代码段下内存访问断点，来到：

```asm
CALL    @ACP_Feed_00401DB0
JMP     @ACP_Feed_008806E9
PUSH    0
PUSH    0
CALL    @ACP_Feed_00401E74
MOV     EBX,EAX
CALL    @ACP_Feed_00401E94
JMP     @ACP_Feed_008806F4
JNZ     @ACP_Feed_0040219A
PUSH    EBX
CALL    @ACP_Feed_00401E64
JMP     @ACP_Feed_00880700
JMP     @ACP_Feed_0088070B
DB      0E8h
DB      022h                  ; CHAR '"'
```

很明显程序的开头几句指令被偷走了，不过这难不倒我们，随便找一个 Delphi 的程序就可以把入口补上：

```asm
push ebp
mov esp, ebp
add esp, -10
push ebx
mov eax, XXXXXXXX
```

Copyright by `redoc` (`redoc` at `citiz.net`)

任意找一个 API 调用确定 `IAT` 的位置和大小：

`@ACP_Feed_00401CEC:`

```asm
JMP     @ACP_Feed_00880500
DB      00
MOV     EAX,EAX
```

`@ACP_Feed_00880500:`

```asm
JMP     DWORD PTR DS:[00405108]      ; 定位 IAT
JMP     @ACP_Feed_00401CF2
OUT     0FF,EAX
```

```
dd 000370000h, 00037009Dh, 0003700D1h, 00037011Dh, 000370163h, 00037020Fh, 000370228h, 0003702D1h
dd 0003702EFh, 000370321h, 0003703ADh, 00037045Ah, 000370508h, 000370564h, 0003705E5h, 0F80EEB0Fh
dd 00037062Eh, 00037068Eh, 0A9D5AE40h, 0003706BFh, 000370715h, 00037076Bh, 05CAAD636h, 00037079Eh
dd 0003707EDh, 000370826h, 0003708C8h, 07D2BBBE9h, 0003708F5h, 00037091Dh, 00037093Dh, 00037098Ah
dd 0BCBE2098h, 0003709D4h, 000370A07h, 000370A30h, 000370A64h, 000370A7Ah, 000370AA1h, 000370AF0h
dd 000370B3Ah
```

这些 `00037XXXX` 的地址很容易识别，不过很显然它们被加密了。我们对其中任意一个地址下内存访问断点，就可以中断到外壳填写 `IAT` 的代码。

```asm
MOV     DWORD PTR DS:[ECX],EBX
JMP     @ACP_Feed_00354668
```

幸运的是，当外壳执行到这句指令时，`EAX` 恰好为正确的 API 地址，所以我们写一个类似下面的脚本就可以避开所有的 `IAT` 重定向：

```
mov tmpreg, ebx
mov ebx, eax
sti
mov ebx, tmpreg
```

需要注意的是，由于外壳的 `CRC` 校验，我们不能直接修改这句指令为 `"mov [ecx], eax"` 或者在这句指令上下 `bp` 断点，唯一的方法是采用硬件断点。

Copyright by `redoc` (`redoc` at `citiz.net`)

OK，`PELock` 对 `IAT` 的加密现在看了已经没有什么强度了，但是在当时看来还是比较出色的。下面才是 `PELock` 值得关注的重点——Code Redirection。

`PELock` 将原程序中的一些 6 字节指令重定向到外壳申请的空间去执行，这样脱壳以后程序就会出现访问异常。下面是 `PELock` 通常下毒手的一些指令：

**第一组：**

```
004011C6  JMP     00880187
004011CB  DB      00

00880187  CALL    DWORD PTR DS:[404008]      ; call dword ptr [XXXXXXXX]
0088018D  JMP     ACP_Feed.004011CC
```

**第二组：**

```
0040190A  JMP     00880341
0040190F  CDQ

00880341  LEA     EBX,DWORD PTR DS:[C5FCE76]
00880347  LEA     EBX,DWORD PTR DS:[EBX+22CDBDF0]
0088034D  LEA     EBX,DWORD PTR DS:[EBX+459B7BE1]
00880353  LEA     EBX,DWORD PTR DS:[EBX+8B36F7C3]  ; mov ebx, XXXXXXXX
00880359  JMP     ACP_Feed.0040190F
```

当然这里也有可能是 `eax`, `ecx`, `ebp` ... whatever

**第三组：**

```
004010D8  JMP     008800FD
004010DD  PUSH    DWORD PTR FS:[EDX]

008800FD  PUSH    40119A         ; push XXXXXXXX
00880102  JMP     ACP_Feed.004010DD
```

**第四组：**

```
00401293  JMP     008801B1
00401298  JE      SHORT 004012A1

008801B1  CMP     EAX,0D00         ; cmp eax, XXXXXXXX
008801B7  JMP     ACP_Feed.00401298
```

当然这里也有可能是 `ebx`, `ecx`, `ebp` ... whatever

Copyright by `redoc` (`redoc` at `citiz.net`)

指令的变形并不复杂，甚至可以说几乎没有变化，但是给转储带来困难。一种比较取巧的方法是把这些区段补到脱壳后的文件中，但是要完美还原的话还是要写代码来修复。`jingulong` 达人曾经给出一段修复代码，没能耐心看完，自己写了一个 `ODbgScript` 脚本做了一下类似反汇编器的工作如下：

```
mov  TmpOpcode, [TmpRemoteJmp], 1
cmp  TmpOpcode, FF        // call/jmp dword ptr [XXXXXXXX]
je  CallJmpDwordIns
cmp  TmpOpcode, 8D        // lea reg32, [XXXXXXXX]
je  AddRegIns
cmp  TmpOpcode, 68        // push XXXXXXXX
je  PushConstIns
cmp  TmpOpcode, 81        // cmp reg32, XXXXXXXX
je  CmpRegIns
msg  "new mode"        // more transforming modes to be revealed:)
ret

…………………………
```

完美还原所有的 Redirected Code 到原来的位置上，脚本比较长，不过理解了上述原理就很容易实现。

最后剩下需要处理的就是 Delphi 的那张表了，我们注意一下程序运行到入口点时 `eax` 的值：`00356C20`

`@ACP_Feed_00356C20:`

```
dd 000000007h, 000356C28h, 000401E24h, 000401DF4h, 000401C64h, 000401C10h, 000401E5Ch, 000401E2Ch
                  |           |
                  ~~~~~~~~~~~
dd 000401F2Ch, 000401EFCh, 000401F64h, 000401F34h, 000401FA4h, 000401F74h, 000000000h, 0004020E8h
```

这是一张地址表，既包含了数据又包含了代码指针，需要留意的是上面标出的这两个数据，第二个数据恰好是指向第三个数据的指针。

现在我们把这个表搬到脱壳以后废弃的区段中，把入口点处的 `"mov eax, XXXXXXXX"` 中的 `XXXXXXXX` 设定为这个表的指针。

今年的暖冬让人不禁战栗，也许在南太平洋的某块水域，一场完美的热带风暴正在紧张的酝酿之中……

`PELock`, u r TermiNated...

支持。

以下是建议：

1. `OEP` 混乱只是插入了一些 `junkcode`
2. Delphi 那个表可以找到原始的位置放回去
3. 那些 `jmp` 好像有个特殊的 `cmp eax, xxx` 要处理，太老了我记不清楚了

引用: *最初由 forgot 发布*

支持。

以下是建议：

1. `OEP` 混乱只是插入了一些 `junkcode`
2. Delphi 那个表可以找到原始的位置放回去
3. `cmp eax, XXXXXXXX` 有 2 种机器码，变形以后的是 6 个字节，但是还原的时候要处理成 5 个字节，否则会覆盖下面的代码；
4. 上传的脱壳文件好像有误－。－

谢谢～

脱壳机版主果然细心：)

1. 对 Delphi 程序来说，`OEP` `Stolen` `Code` 只有固定的 5 句，从堆栈和寄存器就可以恢复；
2. 同意，`OEP` 位置确定了就可以还原初始化表到原来的位置；
3. `cmp eax, XXXXXXXX` 有 2 种机器码，变形以后的是 6 个字节，但是还原的时候要处理成 5 个字节，否则会覆盖下面的代码；
4. 上传的脱壳文件好像有误－。－

`4dd466d0e1e0cc123afbe504da2462ca`
