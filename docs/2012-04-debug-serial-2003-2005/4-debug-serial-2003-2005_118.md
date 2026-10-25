
# explzh

> 原文：[`www.pediy.com/kssd/pediy06/pediy6231.htm`](https://www.pediy.com/kssd/pediy06/pediy6231.htm)

一个网友求助的软件的算法分析------`explzh` 好象是压缩与解压缩软件

说明：这个软件的破解是一个网友给我发了一封 `EMAIL`，求助这个软件的破解，原信如下：

发件人: "Chris Wahl" <smalltalk-80@tom.com>

收件人: 隐去我的邮件地址 抄送: (无)

发送时间: 2003-09-17 15:00:56 优先级: 普通

标题: help me to understand Float Computing

Hello , would you like to help me? ,

I am learning crack .

the progie I want to crack is `explzh`, a file compress software

like `WinRAR`.

I use `OllyDbg 1.09` too, but I found there are lots of floating

calculation in the code. I am confused.

fortunately , I found your article `http://www.jxlb.com/cgi-bin/view.cgi?forum=2&topic=2155`

and I think I can get some help from you.

the progie's site is , the updated version has only Japanese version.

I hope you will not reject

[`village.infoweb.ne.jp/~fwhv5283/mirror.htm`](http://village.infoweb.ne.jp/~fwhv5283/mirror.htm)

regrards.

Chris

这是一封英文信件，我的英语水平太菜，看了很久才明白，昨天晚上看到这封信，软件也发到了我的邮箱，

所以就下来看一看，这是一个日本人做的解压缩软件，软件界面的文字我又不认得，找了半天才找到注册处，弄

清了算法，就贴出来看一看，注册错误有信息，但反汇编找不到，所以只能动态调试找关键点。下载地址在：

[`village.infoweb.ne.jp/~fwhv5283/mirror.htm`](http://village.infoweb.ne.jp/~fwhv5283/mirror.htm) 这也是一个日本网站。

软件介绍：好象这是一个压缩与解压缩的软件，我自己也不太清楚。

破解作者：

`yzez[DFCG]`

破解工具：

`OLLYDBG1.09`

破解过程:

1、`OLLYDBG1.09` 载入要破的程序，点击菜单栏上的插件，选命令行，然后在出现的信息框里输入：`bpx hmemcpy,`

按 `Enter` 键，又会出现一个调用的 `CALL` 的框 ，在所有的 `CALL` 上都设置好断点，然后按 `F9`，反复按，直至出现注册

信息框，输入注册信息，用户名：`yzez[DFCG]`, `Email`: `yzez@163.com`, 试验码：`12345678-87654321`，点确定，程序

中断在 `004B9781` 处，代码如下，说明也在下面：

```
004B9774  |.  CALL    EXPLZH.004B96EC
004B9779  |.  CMP     DWORD PTR DS:[EDI], 0
004B977C  |.  MOV     ESI, EAX
004B977E  |.  JE      SHORT EXPLZH.004B97A8
004B9780  |.  PUSH    ESI
004B9781  |.  CALL    DWORD PTR DS:[<&USER32.GetWindow>]*****此 CALL 设断，F9 运行程序，输入任意注册信息
004B9787  |.  LEA     ECX, DWORD PTR DS:[EAX+1]*************点确定，程序被断，按 F8 走，这里连续按 F8 走
004B978A  |.  PUSH    ECX***********************************************会在中断处循环两次，不要管，继续按 F8，直
004B978B  |.  MOV     ECX, [ARG.3]*******************************到 00403088 处，才到核心段
004B978E  |.  PUSH    EAX
******************************************************************省略*********************************************************************
004B97B3 |>  POP     EDI
004B97B4  |.  POP     ESI
004B97B5  |.  POP     EBP
004B97B6  .  RETN    0C
******************************************************************程序核心段的代码如下！*********************************************************************
00403088   .  MOV     EAX, DWORD PTR DS:[503C30]*****进入核心处，我们停在这里！
0040308D   .  MOV     DWORD PTR SS:[ESP+10], EAX
00403091   .  MOV     EAX, DWORD PTR SS:[EBP+5C]
00403094   .  MOV     DWORD PTR SS:[ESP+22C], 0
0040309F   .  MOV     ECX, DWORD PTR DS:[EAX-8]
004030A2   .  TEST    ECX, ECX
******************************************************************省略 N 行代码!*********************************************************************
004031C6   .  E8 2DEE0A00   CALL    EXPLZH.004B1FF8
004031CB   .  8D4C24 14     LEA     ECX, DWORD PTR SS:[ESP+14]
004031CF   .  C68424 2C0200>MOV     BYTE PTR SS:[ESP+22C], 0
004031D7   .  E8 2FED0A00   CALL    EXPLZH.004B1F0B
004031DC   .  8B4424 18     MOV     EAX, DWORD PTR SS:[ESP+18]
004031E0   .  8B5424 1C     MOV     EDX, DWORD PTR SS:[ESP+1C]
004031E4   .  8B1B          MOV     EBX, DWORD PTR DS:[EBX]
004031E6   .  2BF0          SUB     ESI, EAX
004031E8   .  1BFA          SBB     EDI, EDX
004031EA   .  81EE D78A6E3C SUB     ESI, 3C6E8AD7
004031F0   .  53            PUSH    EBX
004031F1   .  MOV     ECX, EXPLZH.0050A328
004031F6   .  SBB     EDI, 0
004031F9   .  CALL    EXPLZH.0042AF60*********这里是关键 CALL，按 F7 跟进！
004031FE   .  TEXT    EAX, EAX***********测试 EAX 的值，注册成功 EAX 的值为 1，反之值为 0
00403200   .  JE      SHORT EXPLZH.00403214****相等就跳，跳就失败
00403202   .  PUSH    EDI
00403203   .  PUSH    ESI
00403204   .  MOV     ECX, EBP
00403206   .  CALL    EXPLZH.004032B0
0040320B   .  MOV     ECX, EBP
0040320D   .  CALL    EXPLZH.004AE153*********到了这里我们就是快乐的 CRACKER，注册成功!
00403212   .  JMP     SHORT EXPLZH.0040327F
00403214   >  PUSH    81C7***************************上面不等就会跳到这里!，向下走几步，就要你交钱!注册失败
00403219   .  LEA     ECX, DWORD PTR SS:[ESP+14]
0040321D   .  CALL    EXPLZH.004B24CE
00403222   .  ECX, DWORD PTR DS:[50A3A0]
00403228   .  MOV     EDX, DWORD PTR SS:[ESP+10]
0040322C   .  PUSH    10
0040322E   .  PUSH    ECX
0040322F   .  JMP     SHORT EXPLZH.00403259
00403231   >  MOV     EAX, DWORD PTR DS:[50A390]
00403236   .  LEA     EDX, DWORD PTR SS:[ESP+24]
0040323A   .  PUSH    200
0040323F   .  PUSH    EDX
00403240   .  PUSH    80E0*************************** "入力されたレジストコードは" 这是失败信息!
******************************************************************关键 CALL 的代码！*********************************************************************
0042AF60 /$  MOV     EAX, DWORD PTR FS:[0]****进入关键 CALL 后，我们就在这里！
0042AF66 |.  PUSH    -1
0042AF68 |.  PUSH    EXPLZH.004CEA68
0042AF6D |.  PUSH    EAX
0042AF6E |.  MOV     DWORD PTR FS:[0], ESP
0042AF75 |.  SUB     ESP, 14
0042AF78 |.  PUSH    EBX
0042AF79 |.  PUSH    EBP
0042AF7A |.  PUSH    ESI
0042AF7B |.  MOV     ESI, DWORD PTR SS:[ESP+30]
0042AF7F |.  PUSH    EDI
0042AF80 |.  MOV     EBX, ECX
0042AF82 |.  CMP     BYTE PTR DS:[ESI], 0
0042AF85 |.  JE      EXPLZH.0042B101
0042AF8B |.  PUSH    0
0042AF8D |.  PUSH    EXPLZH.004FD278
0042AF92 |.  LEA     EAX, DWORD PTR SS:[ESP+18]
0042AF96 |.  PUSH    EXPLZH.004FD290
0042AF9B |.  PUSH    EAX
0042AF9C |.  CALL    EXPLZH.004C57E7
0042AFA1 |.  MOV     EAX, DWORD PTR SS:[ESP+10]
0042AFA5 |.  MOV     DWORD PTR SS:[ESP+2C], 0
0042AFAD |.  MOV     ECX, DWORD PTR DS:[EAX-8]
0042AFB0 |.  TEST    ECX, ECX
0042AFB2 |.  JE      EXPLZH.0042B0F0
0042AFB8 |.  PUSH    0
0042AFBA |.  PUSH    0
0042AFBC |.  PUSH    EAX
0042AFBD |.  MOV     ECX, EBX
0042AFBF |.  CALL    EXPLZH.0042B130********************此 CALL 对用户名进行处理，处理过程是这样，取用户名各位的
********************ASCII 码值，然后把它们相加，我输入的用户名是 yzez[DFCG],其结果是这样：79+7A+65+7A+5B+44+46+43+
********************47+5D=39E，结果保存在 EAX 中，代码就不列了，有兴趣自己跟进去看一看!
0042AFC4 |.  PUSH    ESI*******************************输入的试验码入 ESI
0042AFC5 |.  LEA     ECX, DWORD PTR SS:[ESP+14]****保存用户名运算的结果!
0042AFC9 |.  MOV     DWORD PTR SS:[ESP+20], EAX
0042AFCD |.  MOV     DWORD PTR SS:[ESP+24], EDX
0042AFD1 |.  CALL    EXPLZH.004B2048
0042AFD6 |.  PUSH    2D****************************2D 入栈，2D 是“-”的 ASCII 码值，注册码中必须有“-”，故
******************************注册码的形式应是：XXXXXXXX-YYYYYYYY
0042AFD8 |.  LEA     ECX, DWORD PTR SS:[ESP+14]
```



# 注册码校验逻辑分析

## 1. 注册码格式验证

`0042AFDC |. CALL EXPLZH.004B23B9`

此 `CALL` 检验注册码中是否有`"-"`。如果有，就把第一组注册码的位数赋给`EAX`；如果没有，`EAX` 赋值为`FFFFFFFF`（即`-1`）。

- `0042AFE1 |. MOV EBP, EAX`：第一组注册码的位数（如 8）移入`EBP`中。
- `0042AFE3 |. CMP EBP, -1`：比较`EBP`是不是`-1`。
- `0042AFE6 |. JE EXPLZH.0042B0F0`：如果相等就跳转，这意味着流程结束（“送你去睡觉”）。因此注册码中必须包含`"-"`。
- `0042AFEC |. PUSH EBP`：如果不跳转，继续执行，将位数（如 8）入栈。

## 2. 字符串处理与初始化

以下指令用于设置局部变量和调用标准库函数进行字符串操作：

```
0042AFED  |.  LEA    ECX, DWORD PTR SS:[ESP+1C]
0042AFF1  |.  PUSH   0
0042AFF3  |.  PUSH   ECX
0042AFF4  |.  LEA    ECX, DWORD PTR SS:[ESP+1C]
0042AFF8  |.  CALL   EXPLZH.004AA18B
0042AFFD  |.  PUSH   EXPLZH.0050021C
0042B002  |.  LEA    EDX, DWORD PTR SS:[ESP+18]
0042B006  |.  PUSH   EAX
0042B007  |.  PUSH   EDX
0042B008  |.  MOV    BYTE PTR SS:[ESP+38], 1
0042B00D  |.  CALL   EXPLZH.004B2154
0042B012  |.  PUSH   EAX
0042B013  |.  LEA    ECX, DWORD PTR SS:[ESP+14]
0042B017  |.  MOV    BYTE PTR SS:[ESP+30], 2
0042B01C  |.  CALL   EXPLZH.004B1FF8
0042B021  |.  LEA    ECX, DWORD PTR SS:[ESP+14]
0042B025  |.  MOV    BYTE PTR SS:[ESP+2C], 1
0042B02A  |.  CALL   EXPLZH.004B1F0B
0042B02F  |.  LEA    ECX, DWORD PTR SS:[ESP+18]
0042B033  |.  MOV    BYTE PTR SS:[ESP+2C], 0
0042B038  |.  CALL   EXPLZH.004B1F0B
```

## 3. 第一组注册码数值计算

`0042B03D |. MOV EAX, DWORD PTR SS:[ESP+10]`

此处加载试验码的第一组（例如 `12345678`，注意后加 `H` 代表十六进制值）到`EAX`。

- `0042B041 |. PUSH EAX`：第一组试验码入栈，准备处理。
- `0042B042 |. CALL EXPLZH.00478D80`
- `0042B047 |. MOV ECX, EAX`
- `0042B049 |. CALL EXPLZH.00479060`
- `0042B04E |. MOV ECX, DWORD PTR SS:[ESP+1C]`：将用户名计算后的结果（如 `39E`）入`ECX`，开始计算第一组注册码。
- `0042B052 |. MOV ESI, EAX`：第一组试验码 `12345678` 移入`ESI`。
- `0042B054 |. MOV EDI, EDX`：此时`EDX` 值为 0，故`EDI` 值为 0。
- `0042B056 |. MOV EDX, DWORD PTR SS:[ESP+20]`：赋`EDX` 的值为 0。

### 核心减法运算

- `0042B05A |. SUB EAX, ECX`：执行 `EAX = EAX - ECX`，即 `12345678 - 39E = 123452DA`。
- `0042B05C |. MOV ECX, EDI`：将`EDI` 的值（0）入`ECX`。
- `0042B05E |. SBB ECX, EDX`：执行 `ECX = ECX - EDX - CF`，结果通常为 0。
- `0042B060 |. SUB EAX, 3C6E8AD7`：执行 `EAX = EAX - 3C6E8AD7`，即 `123452DA - 3C6E8AD7 = D5C5C803`。
- `0042B065 |. SBB ECX, 0`：执行 `ECX = ECX - 0`，结果为 0。

`0042B068 |. PUSH ECX`
`0042B069 |. PUSH EAX`：上述计算后的结果 `D5C5C803` 入栈。

## 4. 第一次比较与跳转

`0042B06A |. MOV ECX, EBX`
`0042B06C |. CALL EXPLZH.0042AE00`

在这里进行比较（可按 `F7` 进入该 `CALL` 查看）。

- `0042B071 |. TEST EAX, EAX`：检查上述 `CALL` 的比较结果。如果比较成功，`EAX` 的值为 `FFFFFFFE`；否则 `EAX` 的值为 0。
- `0042B073 |. MOV DWORD PTR SS:[ESP+1C], EAX`：保存 `EAX` 的值。
- `0042B077 |. JE SHORT EXPLZH.0042B0F0`：如果相等则跳转。此时 `EAX` 为 0 表示失败，跳转即失败。这是**爆破点 1**，建议改为 `NOP` 或逆向修改条件。
- `0042B079 |. CMP EAX, -1`：与 `-1` 比较。
- `0042B07C |. JE SHORT EXPLZH.0042B0F0`：如果相等则跳转，同样意味着失败。此处不必修改。

## 5. 第二组注册码数值计算

`0042B07E |. MOV ECX, DWORD PTR SS:[ESP+34]`：将试验码入`ECX`，准备获取第二组试验码。

- `0042B082 |. LEA EDX, DWORD PTR DS:[ECX+EBP+1]`：第二组试验码所在的地址赋给`EDX`。
- `0042B086 |. LEA ECX, DWORD PTR SS:[ESP+10]`：试验码的地址赋给`ECX`。
- `0042B08A |. PUSH EDX`
- `0042B08B |. CALL EXPLZH.004B2048`

返回后：
- `0042B090 |. MOV EBP, DWORD PTR SS:[ESP+1C]`：将 `FFFFFFFE`（即 `-2`）入`EBP`。
- `0042B094 |. MOV EAX, DWORD PTR DS:[EAX]`：第二组试验码的地址给`EAX`。
- `0042B096 |. CMP EBP, -2`：比较 `EBP` 是不是 `-2`。
- `0042B099 |. JNZ SHORT EXPLZH.0042B0AE`：如果不相等则跳转到下方（成功路径）。

### 第二组校验逻辑

以下指令用于校验第二组试验码的具体内容：

```
0042B09B  |.  CMP    BYTE PTR DS:[EAX], 31      ; 第二组试验码的第一位与 31 ('1') 比较
0042B09E  |.  JNZ    SHORT EXPLZH.0042B0AE      ; 不相等则跳转
0042B0A0  |.  MOV    DL, BYTE PTR DS:[EAX+1]
0042B0A3  |.  MOV    CL, 34
0042B0A5  |.  CMP    DL, CL
0042B0A7  |.  JNZ    SHORT EXPLZH.0042B0AE
0042B0A9  |.  CMP    BYTE PTR DS:[EAX+2], CL
0042B0AC  |.  JE     SHORT EXPLZH.0042B0E2      ; 如果相等，跳转到 0042B0E2
```

`0042B0AE |. PUSH EXPLZH.0050021C`

如果上述跳转发生（即跳转到这里），表示第二组校验逻辑中的特定分支被触发。

## 6. 第二次数值计算与比较

```
0042B0B3  |.  LEA    ECX, DWORD PTR SS:[ESP+14]
0042B0B7  |.  CALL   EXPLZH.004B229B
0042B0BC  |.  MOV    EAX, DWORD PTR SS:[ESP+10]
0042B0C0  |.  PUSH   EAX
0042B0C1  |.  CALL   EXPLZH.00478D80
0042B0C6  |.  MOV    ECX, EAX
0042B0C8  |.  CALL   EXPLZH.00479060
```

- `0042B0CD |. SUB ESI, EAX`：执行 `ESI = ESI - EAX`。例如 `12345678`（第一组试验码）减去 `87654321`（第二组试验码），结果为 `8ACF1357`。
- `0042B0CF |. MOV ECX, EBX`
- `0042B0D1 |. SBB EDI, EDX`
- `0042B0D3 |. PUSH EDI`
- `0042B0D4 |. PUSH ESI`：上述计算的值 `8ACF1357` 入栈。
- `0042B0D5 |. CALL EXPLZH.0042AE00`：进行第二次比较，调用同一个比较 `CALL`。

## 7. 最终判定与爆破点

- `0042B0DA |. TEST EAX, EAX`：如果比较的值不相等，`EAX` 会被赋值为 0。
- `0042B0DC |. JE SHORT EXPLZH.0042B0F0`：如果 `EAX` 为 0 则跳转，这意味着失败。这是**爆破点 2**，建议改为 `JNZ` 或直接 `NOP` 以防止跳转。
- `0042B0DE |. CMP EBP, EAX`：比较 `EBP` 与 `EAX`。
- `0042B0E0 |. JE SHORT EXPLZH.0042B118`：如果相等则跳转到成功路径。这是**爆破点 3**，建议将 `JE` 改为 `JMP` 以确保始终跳转到成功标志位设置处。

### 失败路径清理

如果跳转到 `0042B0E2` 或 `0042B0F0`：

```
0042B0E2  |.  PUSH   -1
0042B0E4  |.  PUSH   10
0042B0E6  |.  PUSH   81EB
0042B0EB  |.  CALL   EXPLZH.004B9A50
0042B0F0  |.  LEA    ECX, DWORD PTR SS:[ESP+10]
0042B0F4  |.  MOV    DWORD PTR SS:[ESP+2C], -1
0042B0FC  |.  CALL   EXPLZH.004B1F0B
```

后续指令为返回前清理堆栈和寄存器：

```
0042B101  |.  XOR    EAX, EAX
0042B103  |.  MOV    ECX, DWORD PTR SS:[ESP+24]
0042B107  |.  POP    EDI
0042B108  |.  POP    ESI
0042B109  |.  POP    EBP
0042B10A  |.  POP    EBX
0042B10B  |.  MOV    DWORD PTR FS:[0], ECX
0042B112  |.  ADD    ESP, 20
0042B115  |.  RETN   4
```

### 成功路径

`0042B118 |. LEA ECX, DWORD PTR SS:[ESP+10]`：跳转到这里表示校验成功。

```
0042B11C  |.  MOV    DWORD PTR SS:[ESP+2C], -1
0042B124  |.  CALL   EXPLZH.004B1F0B
0042B129  |.  MOV    EAX, 1          ; 在这里赋 EAX 的值为 1，这是注册成功的标志位
0042B12E  |.  JMP    SHORT EXPLZH.0042B103
```

## 8. 比较函数 (CALL) 实现细节

以下是 `EXPLZH.0042AE00` 处的比较 `CALL` 代码片段：

```
0042B130  /$  PUSH   EBX
0042B131  |.  PUSH   EBP
0042B132  |.  MOV    EBP, DWORD PTR SS:[ESP+C]
0042B136  |.  PUSH   ESI
0042B137  |.  PUSH   EDI
0042B138  |.  PUSH   EBP
0042B139  |.  XOR    EDI, EDI
0042B13B  |.  XOR    EBX, EBX
0042B13D  |.  CALL   DWORD PTR DS:[<&KERNEL32.lstrlenA>]  ; lstrlenA
0042B143  |.  MOV    ESI, EAX
0042B145  |.  TEST   ESI, ESI
0042B147  |.  JNZ    SHORT EXPLZH.0042B152
0042B149  |.  POP    EDI
0042B14A  |.  POP    EBP
0042B14C  |.  XOR    EDX, EDX
0042B14E  |.  POP    EBX
0042B14F  |.  0C

; 比较 CALL 的代码入口
0042AE00  /$  MOV    EAX, DWORD PTR SS:[ESP+8]  ; 进入比较 CALL 后我们停在这里! 0 入 EAX
```



# 反汇编代码分析

0042AE04 . MOV ECX, DWORD PTR SS:[ESP+4] 把上述运算过的值：`D5C5C803` 移入 `ECX` 中
0042AE08 . CMP EAX, 1 EAX 的值和 1 比较
0042AE0B . JG EXPLZH.0042AED0 如果大于就跳走，不进行后面的比较
0042AE11 . JL SHORT EXPLZH.0042AE1F 如果小于就跳到下面去进行比较
0042AE13 . CMP ECX, 6132023A
0042AE19 . JA EXPLZH.0042AED0
0042AE1F > CMP ECX, 6132023A 计算过的值与 `6132023A` 比较
0042AE25 . JNZ SHORT EXPLZH.0042AE30 不相等就跳，跳到下面继续进行比较!
0042AE27 . CMP EAX, 1 EAX 与 1 比较
0042AE2A . JE EXPLZH.0042AEC8 相等就跳!
0042AE30 > TEST EAX, EAX
0042AE32 . JG SHORT EXPLZH.0042AE96
0042AE34 . JL SHORT EXPLZH.0042AE3E 跳到 `0042AE96`
0042AE36 . CMP ECX, F87A404F
0042AE3C . JA SHORT EXPLZH.0042AE96
0042AE3E > CMP ECX, F87A404F 比较计算的值与 `F87A404F`
0042AE44 . JNZ SHORT EXPLZH.0042AE4A 不相等就跳，跳到下面!
0042AE46 . TEST EAX, EAX
0042AE48 . JE SHORT EXPLZH.0042AE8E
0042AE4A > CMP ECX, 87A30DE3 再与 `87A30DE3` 比较
0042AE50 . JNZ SHORT EXPLZH.0042AE56 不相等就跳
0042AE52 . TEST EAX, EAX
0042AE54 . JE SHORT EXPLZH.0042AE86
0042AE56 > CMP ECX, B64FBAC4 与 `B64FBAC4` 比较
0042AE5C . JNZ SHORT EXPLZH.0042AE62 不相等就跳
0042AE5E . TEST EAX, EAX
0042AE60 . JE SHORT EXPLZH.0042AE7E
0042AE62 > CMP ECX, BD507501 与 `BD507501` 比较
0042AE68 . JNZ EXPLZH.0042AF4D 不相等就跳，跳就失败
0042AE6E . TEST EAX, EAX 测试 `EAX` 的值
0042AE70 . JNZ EXPLZH.0042AF4D 不相等就跳，跳就失败
0042AE76 . MOV EAX, -2 到这里赋 `EAX` 的值为 `FFFFFFFE(-2)`
0042AE7B . RETN 8 子程序结束!

## 算法小结

注册码的形式是：`XXXXXXXX-YYYYYYYY`

- 第一组注册码减去用户名计算的值再减去 `3C6E8AD7` 必须等于 `BD507501`
- 第一组注册码减去第二组注册码也必须等于 `BD507501`

所以其算法如下：

计算用到用户名，用户名计算是这样，取用户名的各位 ASCII 码值相加，设计算的结果是：`T`。

设第一组注册码是 `S1`，`S1=T+3C6E8AD7+BD507501`

设第二组注册码是 `S2`，`S2=S1-BD507501`

## 验证算法

用户名为：`yzez[DFCG]`

用户名计算的值：`T=79+7A+65+7A+5B+44+46+43+47+5D=39E`

第一组注册码 `S1=39E+3C6E8AD7+BD507501=F9BF0376`

第二组注册码 `S2=F9BF0376-BD507501=3C6E8E75`

所以对应用户名 `yzez[DFCG]` 的注册码是：`F9BF0376-3C6E8E75`，输入进去我们看看，注册成功了吧!

## 爆破

- 点 1：`0042B077` . `JE` 改成 `NOP`
- 点 2：`0042B0DC` . `JE` 改成 `JNZ`
- 点 3：`0042B0E0` . `JE` 改成 `JMP`

上述修改，在输入的注册码中间必须有“-”，如果任意输入都成功，还要在 `0042AFE6` . `JE` 处改

- 点 4：`0042AFE6` . `JE` 改成 `NOP` 或者：`JNZ`

这样修改，输入任意信息都可成为注册版。

`4dd466d0e1e0cc123afbe504da2462ca`
