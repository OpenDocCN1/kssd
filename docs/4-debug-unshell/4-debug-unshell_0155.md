
# ACProtect 脱壳与 Code Replace 修复指南

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-483.htm`](https://www.pediy.com/kssd/pediy07/pediy7-483.htm)

## 问题分析
`ACProtect V.1.3x-1.41` 的 `CodeReplace` 机制会导致代码在解密后被放置到特定内存块（如 `0x143270`）。然而，`Dump` 出来的程序通常不包含该内存块。因此，建议直接进行 `Patch` 操作。

## Patch 流程
例如，在 `0x009D0416` 处执行 `PUSHAD`，并跳转到 `0x009D05C1`。在 `0x009D05C1` 调用 `0x009DF4D2` 以获取所需的 `EBP` 值。

关键分支逻辑如下：
- `0x009D060A`：比较 `AL` 与 `20`。若相等，跳转至另一处解码。
- `0x009D060C`：修改 `JMP` 指令，使其跳转到固定地址以执行解码。
- `0x009D0612`：`MOV EDI, [EBP+40FC23]`，寻找合适的解码地址。
- `0x009D0673`：执行 `PUSHAD` 并跳转至 `0x009D06AE`，防止代码破坏。
- `0x0x009D06AE`：执行 `POPAD` 和 `RETN`。

## 还原 Code Replace
既然需要找到合适的位置进行解码，不如还原 `CodeReplace`。这里涉及两个 Table：
1. `Table1`：返回地址的 `RVA`。
2. `Table2`：相对于返回地址的编码，每个地址对应 `0Ah` 字节。

**操作步骤：**
1. 查看 `Table2` 的大小，例如 `0x6000`。
2. 在 `Dump` 出来的程序中增加一个新的 `Section`，大小为 `0x6000`。
3. 准备 4 个 `Buffer`：
   - `Buffer1`：存放返回地址。
   - `Buffer2`：存放解码后的地址。
   - `Buffer3`：存放错误发生时写入 `CALL` 地址的指针 (`lpErrorAddr`)。
   - `Buffer4`：计数器。

## Patch 代码示例
以下是原程式可 Patch 成的汇编代码逻辑：

```asm
; 009D05C1
CALL WinMain1.009DF4D2

; 009D05CA
XOR ECX, ECX

; 009D05CC
MOV EBX, [EBP+ECX*4+403269]
ADD EBX, 00400000
MOV [Buffer1], EBX

; 009D05E4
LEA ESI, [EBP+406149]

; 009D05EA
MOV EAX, 0A

; 009D05EF
MUL ECX

; 009D05F1
ADD ESI, EAX

; 009D05F3
LEA EDI, [EAX+YourNewSectionStartAddr]
MOV [Buffer2], EDI
MOV [Buffer4], ECX
JMP SHORT WinMain1.009D0621

; 009D0621
MOV BL, [EBP+402406]

; 009D0627
MOV ECX, 0A

; 009D062C
LODS BYTE PTR [ESI]

; 009D062D
XOR AL, BL

; 009D062F
STOS BYTE PTR [EDI]

; 009D0630
LOOPD SHORT WinMain1.009D062C
CALL WriteCallAddr

MOV ECX, [Buffer4]
INC ECX
CMP [EBP+ECX*4+403269], 0
JNZ SHORT WinMain1.009D05CC

WriteCallAddr:
PUSHAD
MOV EBX, [Buffer1]
SUB EBX, 5
CMP BYTE PTR [EBX], 0E8
JNZ WritErrorAddr
MOV EAX, [Buffer2]
SUB EAX, 5
SUB EAX, EBX
ADD EBX, 1
MOV [EBX], EAX
POPAD
RETN

WritErrorAddr:
MOV EDX, [Buffer3]
MOV [EDX], EBX
ADD [Buffer3], 4
POPAD
RETN
```

## 后续处理
如此处理后，你会看到 `CALL 009D0416` 变成了 `CALL XXXXXXXX`（你的新地址）。记得先用 `0D` 载入你的 dumped 程序，在 `0x009D0416` 处重建 `EIP`（可以通过 `da 009D0416` 或 `Ctrl+G` 输入 `009D0416` 后 `Patch`）。修复后再 `dump` 出来。

## 高级反混淆
`OEP obfuscation`、`Code Replace` 和 `API random redirection` 并非 `ACProtect` 最困难的部分。修复好 `CodeReplace` 后，你可能还要面对：
1. `HookMessageBoxA`
2. `Embedded Protector`

`HookMessageBoxA` 相对容易解决，最麻烦的是 `Embedded Protector`（嵌套加密）。所谓的 `Embedded Protector` 就是 `ACProtect` 用户能指定某段代码（如注册码比较、主要功能等）重新编码。你必须通过 `ACProtect` 的检验，这些检验包括：
- 文件的大小 (`SIZE`)
- 目前的 `OEP` 是否等于加壳后的 `OEP`
- 目前的 `OEP` 是否等于加壳前的 `OEP`
- 检查调试器的 `Driver` 是否存在
- 检查 `PE` 部首的数据等...

通过重重检验后才会给你正确的解码。麻烦的是这种检验可能不止一处，而是十几处。

至于第二个问题，`ACProtect` 会留下 `Import Table` 的完美指针，只清掉函数名（类似旧版 `ASProtect`）。丢掉的数据在 `IAT` 的编码处能找回。若你想脱壳并跨平台，修复原始的 `Import Table` 会省却不少麻烦。相关教程可以参考 `fly` 的教程。

祝好运!!
![image](img/0cf410da109ac47387dd713fcee2cd79.jpg)

4dd466d0e1e0cc123afbe504da2462ca
