
# EmEditor 3.28

> 原文：[`www.pediy.com/kssd/pediy05/pediy50475.htm`](https://www.pediy.com/kssd/pediy05/pediy50475.htm)

目标软件：`EmEditor 3.28` 简体中文版

文件大小：1086 KB

使用平台：`Win9x/NT/2000/XP`

软件授权：共享软件

使用限制：功能限制

文件大小：1086 KB

下载地址：[`www.emurasoft.com/pub/eme328cs.exe`](http://www.emurasoft.com/pub/eme328cs.exe)

软件简介：大名鼎鼎的 `EmEditor` 是一款优秀的文字处理软件，与同类软件相比，它具有轻便快速的优点。它的功能非常强大，不管你的文章是否含有换行符、即使容量为几 MB 的文件也可以很快地打开编辑；通过插件导入支持，几乎可实现所有的特殊功能；键盘、工具列、菜单、字型和色彩等的自订功能，实现轻松编辑。`EmEditor` 还获得了“Designed for Windows”徽标认证，可在各操作系统下获得最佳的性能体验。网页设计者用它来编辑 `HTML` 或者 `Perl` 程序文件，程序员用它来编辑源程序，普通用户用它作为记事本——这正是它的出众之处。

新版本更新内容：在检索，替换以及在文档中检索的时候，增加了利用正规表现的方法，增加了“终了后关闭”的勾选栏；解决了在 `tag jump` 时，如果文件不存在，会出现的异常结束的问题；勾选“以空格代替 `tab` 插入”后，如果输入 `tab`，就会自动将 `tab` 变为空格；在命令栏中增加了“不增加最近使用过的文档的快速路径”选项。

使用工具：`TRW2000 v1.22`

### 破解过程

输入序列号：`1000-2000-3000-4000`，记为 `SN(1)`-`SN(2)`-`SN(3)`-`SN(4)`。

`Hmemcpy` 设断，来到：

```asm
CMP     EAX,03F1
JNZ     004196DE
PUSH    BYTE +01
CALL    00418CF1
MOV     ESI,EAX
CMP     ESI,EBX
JZ      004196C7
PUSH    ESI
CALL    004194AA             / * Shit！竟然不符合人体工程学！追进！* /
CMP     EAX,EBX
JNG     004196A4             / * 跳走就 Game Over ！* /
XOR     ECX,ECX
CMP     EAX,BYTE +02
SETZ    CL
MOV     [00447B74],EAX
PUSH    BYTE +40
ADD     ECX,0455
JMP     SHORT 004196BA
JZ      004196C0
XOR     ECX,ECX
CMP     EAX,BYTE -02
SETNZ   CL
PUSH    BYTE +30
DEC     ECX
AND     ECX,BYTE +03
ADD     ECX,0454
PUSH    ECX
CALL    0041ABEA             / * Call 对话框 * /
PUSH    ESI
CALL    `ADVAPI32!RegCloseKey`
LEA     ECX,[EBP+08]
CALL    0040C0AA
XOR     EAX,EAX
MOV     [004499A4],BL
INC     EAX
POP     ESI
POP     EBX
LEAVE
RET     04
```

```asm
PUSH    EBP
MOV     EBP,ESP
SUB     ESP,BYTE +30
PUSH    ESI
PUSH    BYTE +02
PUSH    DWORD 004464F8
LEA     EAX,[EBP-30]
PUSH    DWORD 0043FBB8
PUSH    EAX
CALL    `USER32!wsprintfA`
MOV     ESI,[0043F004]
AND     DWORD [EBP-04],BYTE +00
ADD     ESP,BYTE +10
LEA     EAX,[EBP-04]
PUSH    EAX
PUSH    BYTE +00
LEA     EAX,[EBP-08]
PUSH    EAX
PUSH    BYTE +00
LEA     EAX,[EBP-30]
PUSH    EAX
PUSH    DWORD [EBP+08]
CALL    ESI
TEST    EAX,EAX
JNZ     0041951F
CMP     DWORD [EBP-08],BYTE +03
JNZ     0041951F
CMP     DWORD [EBP-04],BYTE +08
JNZ     0041951F
LEA     EAX,[EBP-04]
PUSH    EAX
LEA     EAX,[EBP-10]
PUSH    EAX
LEA     EAX,[EBP-08]
PUSH    EAX
PUSH    BYTE +00
LEA     EAX,[EBP-30]
PUSH    EAX
PUSH    DWORD [EBP+08]
CALL    ESI
TEST    EAX,EAX
JNZ     0041951F
LEA     EAX,[EBP-10]
PUSH    EAX
CALL    00423EDA            / * 兵来将挡，“你”来我访！* /
JMP     SHORT 00419521
XOR     EAX,EAX
POP     ESI
LEAVE
RET     04
```

```asm
PUSH    ESI
MOV     ESI,[ESP+08]
MOVZX   EAX,WORD [ESI]    / * SN（1）的十六进制值送入 EAX * /
PUSH    BYTE +0A          / * 0xA 入栈 * /
CDQ
POP     ECX               / * ECX 出栈 * /
IDIV    ECX               / * 除以 ECX * /
CMP     EAX,AB            / * 比较商是否等于 0xAB * /
JZ      00423EF4          / * 是就进入下一步计算 * /
OR      EAX,BYTE -01
JMP     SHORT 00423F12    / * 否则返回 * /
```

从上面我们知道：`SN(1)/0xA=0xAB`，即 `SN(1)=0xAB*0xA=0x6AE`。

哎，等等先~！我以前校队球衣号码是 7 号，那就：`SN(1)=0x6AE+0x7=0x6B5` 吧 ：）！

重新填入注册信息继续：

```asm
PUSH    EDI
MOV     DI,[ESI+06]
PUSH    ESI
CALL    00423DD1          / * 可疑呀~~~ * /
CMP     EAX,BYTE +01      / * EAX 必须等于 1 哦 * /
JNZ     00423F11          / * 跳走就 Game Over ！
```

```asm
PUSH    ECX
PUSH    ESI
MOV     ESI,[ESP+0C]
MOV     AX,[ESI+02]       / * SN(2) 送入 AX * /
AND     WORD [ESI+06],BYTE +00
CMP     AX,270F
JA      NEAR 00423ED2     / * 大于 0x270F 就 Game Over ！
```

```asm
MOV     DX,[ESI+04]       / * SN（3） 送入 DX * /
CMP     DX,270F
JA      NEAR 00423ED2     / * 大于 0x270F 就跳向 423ED2 * /
TEST    AX,AX             / * 测试 SN（2） * /
JZ      NEAR 00423ECE     / * 为空就跳向 423ECE * /
CMP     AX,08AE           / * 比较 AX 是否等于 0x8AE * /
JZ      NEAR 00423ECE     / * 为空就跳向 423ECE * /
CMP     AX,162E           / * 比较 AX 是否等于 0x162E * /
JZ      NEAR 00423ECE     / * 为空就跳向 423ECE * /
CMP     AX,2516           / * 比较 AX 是否等于 0x2516 * /
JZ      NEAR 00423ECE     / * 为空就跳向 423ECE * /
XOR     ECX,ECX           / * ECX 清零 * /
MOV     CX,[ESI]          / * SN（1）送入 CX * /
CMP     CX,06AE           / * 比较是否等于 0x6AE * /
MOV     [ESP+04],ECX      / * ECX 的值送入[ESP+4]处 * /
JZ      NEAR 00423ECE     / * 是就送你回姥姥家！
```

```asm
CMP     AX,0B2C           / * 比较 SN（2）是否等于 0xB2C  * /
JZ      NEAR 00423ECE     / * 是就 Game Over ！
```

```asm
CMP     AX,1F80           / * 比较 SN（2）是否等于 0x1F80  * /
JZ      NEAR 00423ECE     / * 是就 Game Over ！
```

```asm
PUSH    EBX
MOVZX   EAX,AX            / * SN（2）的值送入 EAX * /
PUSH    EBP
PUSH    EDI
MOVZX   EDI,DX            / * SN（3）的值送入 EDI * /
MOV     [ESP+18],EAX      / * EAX 的值送入 [ESP+18]处 * /
PUSH    BYTE +64          / * 0x64 入栈 * /
POP     EBX               / * EBX 出栈 * /
MOV     EAX,EDI           / * EDI 送入 EAX * /
CDQ
IDIV    EBX               / * 除以 EBX * /
PUSH    BYTE +0A          / * 0xA 入栈 * /
POP     EBP               / * EBP 出栈 * /
MOVZX   ECX,CX            / * SN（1）送入 ECX * /
PUSH    BYTE +64
MOV     EBX,EAX           / * EAX 送入 EBX * /
MOV     EAX,ECX           / * ECX 送入 EAX * /
CDQ
IDIV    EBP               / * 除以 EBP * /
MOV     EDX,[ESP+1C]      / * SN（2）送入 EDX * /
ADD     EDX,EBX           / * EDX + EBX * /
ADD     EAX,EDX           / * EAX + EDX * /
ADD     EAX,EDI           / * EAX + EDI * /
CDQ
POP     EDI               / * SN（3）出栈 * /
IDIV    EDI               / * 除以 EDI * /
MOV     EAX,[ESP+18]      / * SN（2）送入 EAX * /
PUSH    BYTE +64          / * 0x64 入栈 * /
POP     EBX               / * EBX 出栈 * /
PUSH    BYTE +64
POP     EBP               / * EBP 出栈 * /
PUSH    EBP
MOV     DI,[EDX*4+00446B10]   / * 查表取值 * /
CDQ
IMUL    DI,DI,BYTE +64    / * 乘以 0x64 * /
IDIV    EBX               / * 除以 EBX * /
MOV     EBX,EAX           / * EAX 送入 EBX * /
MOV     EAX,ECX           / * ECX 送入 EAX * /
CDQ
IDIV    EBP               / * 除以 EBP * /
ADD     ECX,EBX           / * ECX + EBX * /
ADD     EAX,ECX           / * EAX + ECX * /
POP     ECX               / * ECX 出栈 * /
CDQ
IDIV    ECX               / * 除以 ECX * /
ADD     DI,[EDX*4+00446B10]   / * 查表取值 * /
CMP     WORD [ESP+10],06B3    / * 比较 SN（1） 是否等于 0x6B3 * /
MOV     [ESI+06],DI       / * DI 的值 送入 ESI+06 处 * /
POP     EDI               / * SN（4）出栈 * /
POP     EBP
POP     EBX
JZ      00423EC9          / * 是就跳向教学注册版本 * /
```

```asm
CMP     WORD [ESP+04],06B2    / * 比较 SN（1）是否等于 0x6B2 * /
JZ      00423EC9          / * 是就跳向教学注册版本 * /
XOR     EAX,EAX           / * 否则将 EAX 清零 * /
INC     EAX               / * EAX + 1 * /
JMP     SHORT 00423ED5    / * 成功返回 * /
PUSH    BYTE +02          / * 0x2 入栈 * /
POP     EAX               / * EAX 出栈 * /
JMP     SHORT 00423ED5
```

```asm
PUSH    BYTE -02          / * 0xFFFFFFFE 入栈 * /
JMP     SHORT 00423ECB
OR      EAX,BYTE -01      / * EAX 与 0xFFFFFFFF 做与运算 * /
POP     ESI
POP     ECX
RET     04                / * 返回 * /
```

```asm
XOR     EAX,EAX           / * EAX 清零 * /
CMP     DI,[ESI+06]       / * 关键比较 * /
SETZ    AL                / * 相等就将 AL 置 1 * /
LEA     EAX,[EAX+EAX-01]  / * 将 EAX+EAX-1 的值保存到 EAX * /
POP     EDI
POP     ESI
RET     04                / * 返回 * /
```

附表：
