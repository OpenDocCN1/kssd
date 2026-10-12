
# BCWipe v2.34.2

> 原文：[`www.pediy.com/kssd/pediy03/forum780.htm`](https://www.pediy.com/kssd/pediy03/forum780.htm)

## 猜数记 - `BCWipe` 注册半破解

- **作者**：`Fpc`
- **目标**：取得注册码
- **软件名称**：`BCWipe v2.34.2`
- **主页**：`www.jetico.com`
- **简介**：外壳扩展程序，用于完全抹除磁盘上的敏感信息，使他人无法恢复（当然自己也不能，所以要小心呀。）

前几天贴过暴破注册的方法，但遇到有注册码输入而没追出，感觉实在是不舒服。今天又追了一次，没有完全成功，但已经可以注册了。‘猜数记’就来源于此，注册码是一点点试出来的，有点搞笑。

原因就在于，作者不是那么笨。`BW` 先判断输入的注册码（`Irc`）格式是否正确，接下来将 `Irc` 中各字符变为 32 进制数，作变换与两个给定的字串比较，若不等则注册失败。

它的格式是`"WP-####-####-####-####"`，`#`可以是数字或字母。

首先给出一个有效的注册码`"WP-1234-&*^%-81VH-7F21"`，最后的 8 位是关键，中间的`"&*^%"`可以用别的符号和数字‘0’以及字母‘I’、‘O’的任意组合来代替。

在本破解中，先看注册码计算的核心部分：

### 《1》

这个 `CALL` 对 `Irc` 作变换处理，如果出口是 `Eax=1`，则表示需进一步处理；若出口是 `Eax=0`，则表示不需要可以直接验证 `Irc` 是否正确（这个很重要）。

```asm
* Referenced by a CALL at Address:
|:10012C95
|
:10012CC0 8B442408                mov eax, dword ptr [esp+08]
:10012CC4 53                      push ebx
:10012CC5 8B5C2408                mov ebx, dword ptr [esp+08]
:10012CC9 56                      push esi
:10012CCA 57                      push edi
:10012CCB 33F6                    xor esi, esi
:10012CCD 8D7810                  lea edi, dword ptr [eax+10]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10012CF5(C)
|
:10012CD0 8A4FFF                  mov cl, byte ptr [edi-01]     <- 从 Irc 最后一位开始取一个字符
:10012CD3 4F                      dec edi
:10012CD4 51                      push ecx        <- 将这个字符压栈
:10012CD5 E8A6000000              call 10012D80   <- 《2》 将它换成 32 进制数，返回值在 Eax 中
:10012CDA 83C404                  add esp, 00000004
:10012CDD 83F8FF                  cmp eax, FFFFFFFF   <- Eax=-1 表示 Irc 中含有不在码表中的字符
:10012CE0 741E                    je 10012D00     <- 向下跳，准备返回。这个跳转在本破解中起到关键作用！！
:10012CE2 6A05                    push 00000005
:10012CE4 50                      push eax
:10012CE5 56                      push esi
:10012CE6 53                      push ebx
:10012CE7 E824000000              call 10012D10
:10012CEC 83C605                  add esi, 00000005
:10012CEF 83C410                  add esp, 00000010
:10012CF2 83FE50                  cmp esi, 00000050
:10012CF5 7CD9                    jl 10012CD0     <- 不到 16 次则继续
:10012CF7 5F                      pop edi
:10012CF8 5E                      pop esi
:10012CF9 B801000000              mov eax, 00000001 <- 表示 Irc 中的字符均可转换为 32 进制数
:10012CFE 5B                      pop ebx
:10012CFF C3                      ret
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10012CE0(C)
|
:10012D00 5F                      pop edi       <- 遇到“不正常”字符时跳到这里
:10012D01 5E                      pop esi
:10012D02 33C0                    xor eax, eax
:10012D04 5B                      pop ebx
:10012D05 C3                      ret
```

### 《2》

这个 `CALL` 将调用前的最后一次压栈字符换算为 32 进制数，返回值在 `Eax` 中，注意 32 进制数从 0 开始；若所查的字符不在其中则 `Eax=-1`。

例如：`1->0`，`3->2`，`9->8`，`W->A`，`K->17`，`M->1F`，`&->-1`，`0->-1`。

```asm
* Referenced by a CALL at Address:
|:10012CD5
|
:10012D80 8A0D5CC30210            mov cl, byte ptr [1002C35C]
```

这个地址是 32 进制数的码表（很重要）：`"12345678 9QWERTYU PASDFGHK LZXCVBNM"` `<- 注意其中没有 0、I、O`

```asm
:10012D86 33C0                    xor eax, eax
:10012D88 84C9                    test cl, cl
:10012D8A 7413                    je 10012D9F
:10012D8C 8A542404                mov dl, byte ptr [esp+04]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10012D9D(C)
|
:10012D90 3ACA                    cmp cl, dl
:10012D92 740E                    je 10012DA2
:10012D94 8A885DC30210            mov cl, byte ptr [eax+1002C35D]
:10012D9A 40                      inc eax
:10012D9B 84C9                    test cl, cl
:10012D9D 75F1                    jne 10012D90
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10012D8A(C)
|
:10012D9F 83C8FF                  or eax, FFFFFFFF    <- 如果字符不在其中则 Eax=-1
                                            <- 想不到这个也是有用的
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10012D92(C)
|
:10012DA2 C3                      ret
:10012DA3 90                      nop
```

### 《3》

这个 `CALL` 编写的实在是精彩，其作用是将得到的 32 进制数作移位变换，存放于目标地址。

```asm
* Referenced by a CALL at Address:
|:10012CE7
|
:10012D10 8B442408                mov eax, dword ptr [esp+08]
:10012D14 53                      push ebx
:10012D15 56                      push esi
:10012D16 57                      push edi
:10012D17 8BF8                    mov edi, eax
:10012D19 81E707000080            and edi, 80000007   <- Edi 值每次都在变，在下面用作移位的位数（很高明）
:10012D1F 7905                    jns 10012D26        <- Edi=0、5、2、7、4、1、8.....
:10012D21 4F                      dec edi             <-
:10012D22 83CFF8                  or edi, FFFFFFF8    <- 这三个好象不会被执行到
:10012D25 47                      inc edi             <-
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10012D1F(C)
|
:10012D26 8B74241C                mov esi, dword ptr [esp+1C]
:10012D2A 8B5C2410                mov ebx, dword ptr [esp+10]   <- 取得目标地址，内容一开始时为空
:10012D2E 99                      cdq
:10012D2F B908000000              mov ecx, 00000008
:10012D34 83E207                  and edx, 00000007
:10012D37 2BCE                    sub ecx, esi
:10012D39 BEFF000000              mov esi, 000000FF
:10012D3E 03C2                    add eax, edx
:10012D40 33D2                    xor edx, edx
:10012D42 D3FE                    sar esi, cl
:10012D44 C1F803                  sar eax, 03       <- 调整 Eax 的位置
```

`Eax` 的改变方式是随着这个 `CALL` 的调用 `Eax=0`、`0`、`1`、`1`、`2`、`3`、`3`、`4`、`4`、.......

```asm
:10012D47 03C3                    add eax, ebx      <- Eax 指向目标地址
:10012D49 8BCF                    mov ecx, edi
:10012D4B 8A7001                  mov dh, byte ptr [eax+01]
:10012D4E 8BDE                    mov ebx, esi
:10012D50 8A10                    mov dl, byte ptr [eax]
:10012D52 D3E3                    shl ebx, cl
:10012D54 8B4C2418                mov ecx, dword ptr [esp+18]   <- 取得一个 32 进制数
:10012D58 23F1                    and esi, ecx      <- 相当于 Esi=Ecx
:10012D5A 8BCF                    mov ecx, edi
:10012D5C D3E6                    shl esi, cl       <- Esi*2 的 N 次方
:10012D5E F7D3                    not ebx
:10012D60 23D3                    and edx, ebx
:10012D62 5F                      pop edi
:10012D63 0BD6                    or edx, esi       <- 结果送 Edx，注意它用 Or 的方式保留低位字节的值
:10012D65 5E                      pop esi
:10012D66 8BCA                    mov ecx, edx
:10012D68 5B                      pop ebx
:10012D69 8808                    mov byte ptr [eax], cl
:10012D6B C1F908                  sar ecx, 08
:10012D6E 884801                  mov byte ptr [eax+01], cl
:10012D71 C3                      ret
```

### 小结

综合《1》和《2》《3》，程序按从后到前的顺序取得 `Irc`，换成 32 进制数，移位得到注册码 1（`Rc1`）。

举例如下(可能有错误，只是表明计算过程是这样的)：

假设 `Irc`：`WP-1234-2345-4567-6789`，则(以下为 16 进制)

- `9->8`，`8*1=8`；
- `8->7`，`7*20=E0`。 `Rc1[0]=E0 OR 8=E8`；
- `7->6`，`6*4=18`；
- `6->5`，`5*80=280`。 `Rc1[1]=80 OR 18=98`；`Rc1[2]=2`；
- `7->6`，`6*10=60`。 `Rc1[2]=60 OR 2=62`；（注意这个位只计算一次）
- `6->5`，`5*2=A`；
- `5->4`，`4*40=100`。 `Rc1[3]=00 OR A=A`；`Rc1[4]=1`；
- .........................

生成的 `Rc1`：`E8 98 62 0A 01`

这个 `Rc1` 要与下面的字串作比较，因为我看不出算法，无法反推，只能用计算器来算（这个过程很麻烦）。

```
====================================================================
* Referenced by a CALL at Addresses:
|:1000728A , :1000771C , :10007BD4
|
:10007620 83EC0C                  sub esp, 0000000C
:10007623 8B0D00420310            mov ecx, dword ptr [10034200]
```
