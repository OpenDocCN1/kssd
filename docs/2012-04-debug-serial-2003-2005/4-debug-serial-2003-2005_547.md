
# 英语词汇逻辑记忆 v3.20

> 原文：[`www.pediy.com/kssd/pediy05/pediy50638.htm`](https://www.pediy.com/kssd/pediy05/pediy50638.htm)

英语词汇逻辑记忆 `v3.20` 共享版

发布时间：2003-1-5

发布网站：`http://www.cu2008.com`

下载连接：`http://www.cu2008.com/download/lgm3.zip`

联系方式:

电话：`0765-8607074`

支持：`support@cu2008.com`

连接：`links@cu2008.com`

`OEM`：`sales@cu2008.com`

## 软件简介

运用对比、联想、词义辨析、构词法、同义词和反义词等记忆方法，以词（不是词根）为单位，代替以字母为单位作为突破口，从而根本上改变了人们习以字母为单位的传统记忆方法，进而更新了人们关于“生词”、“常用词”、“词类”等一系列旧有的思维模式。最有效地防止遗忘。

它还是国内第一款倒列英汉词典，提供正列和倒列查找功能，并可以限制查找单词的长度，并可以将查找到的单词直接倒入用户词库。

序列号：`98125` `58109930`

注册码：`654321`

## 一、软件用 `ASPack2.1` 加的壳

脱壳后，用 `W32DASM` 载入，找出错信息。

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CCF9D(C)
|
:004CD00F 4B                 dec ebx
:004CD010 3B5AFC             cmp ebx, dword ptr [edx-04]
:004CD013 7205               jb 004CD01A
:004CD015 E8725EF3FF         call 00402E8C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CD013(C)
|
:004CD01A 43                 inc ebx
:004CD01B 807C1AFF30         cmp byte ptr [edx+ebx-01], 30
:004CD020 7216               jb 004CD038
:004CD022 8B55FC             mov edx, dword ptr [ebp-04]
:004CD025 4B                 dec ebx
:004CD026 3B5AFC             cmp ebx, dword ptr [edx-04]
:004CD029 7205               jb 004CD030====＞
:004CD02B E85C5EF3FF         call 00402E8C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CD029(C)
|
:004CD030 43                 inc ebx
:004CD031 807C1AFF39         cmp byte ptr [edx+ebx-01], 39
:004CD036 760F               jbe 004CD047

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004CD020(C)
|

* Possible StringData Ref from Code Obj ->"无效的注册码!"
                                |
:004CD038 B878D14C00         mov eax, 004CD178
```

## 二、用 `ollydbg` 载入程序

关键算法如下：

```
004CCFC8  |.  68 60D14C00   PUSH unpacked.004CD160
004CCFCD  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004CCFD0  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004CCFD3  |.  8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]
004CCFD6  |.  8B86 D8020000 MOV EAX,DWORD PTR DS:[ESI+2D8]
004CCFDC  |.  E8 3B6DF6FF   CALL unpacked.00433D1C                     ;  取输入注册码位数
004CCFE1  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]               ;  注册码入 eax
004CCFE4  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
004CCFE7  |.  E8 90BAF3FF   CALL unpacked.00408A7C                     ;  取注册码
004CCFEC  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]               ;  注册码入 eax
004CCFEF  |.  E8 BC6EF3FF   CALL unpacked.00403EB0                     ;  位数 L
004CCFF4  |.  48            DEC EAX                                    ;  以上判断输入是否为 0，如是则跳走
004CCFF5  |.  0F8C 28010000 JL unpacked.004CD123
004CCFFB  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]               ;  注册码入 eax
004CCFFE  |.  E8 AD6EF3FF   CALL unpacked.00403EB0                     ;  位数
004CD003  |.  85C0          TEST EAX,EAX
004CD005  |.  7E 44         JLE SHORT unpacked.004CD04B                ;  是否为 0
004CD007  |.  BB 01000000   MOV EBX,1                                  ;  计数开始
004CD00C  |>  8B55 FC       /MOV EDX,DWORD PTR SS:[EBP-4]             ;  注册码入 edx
004CD00F  |.  4B            |DEC EBX
004CD010  |.  3B5A FC       |CMP EBX,DWORD PTR DS:[EDX-4]
004CD013  |.  72 05         |JB SHORT unpacked.004CD01A
004CD015  |.  E8 725EF3FF   |CALL unpacked.00402E8C
004CD01A  |>  43            |INC EBX
004CD01B  |.  807C1A FF 30  |CMP BYTE PTR DS:[EDX+EBX-1],30           ;  注册码第一位是否小于 0
004CD020  |.  72 16         |JB SHORT unpacked.004CD038
004CD022  |.  8B55 FC       |MOV EDX,DWORD PTR SS:[EBP-4]             ;  注册码入 edx
004CD025  |.  4B            |DEC EBX
004CD026  |.  3B5A FC       |CMP EBX,DWORD PTR DS:[EDX-4]
004CD029  |.  72 05         |JB SHORT unpacked.004CD030
004CD02B  |.  E8 5C5EF3FF   |CALL unpacked.00402E8C
004CD030  |>  43            |INC EBX
004CD031  |.  807C1A FF 39  |CMP BYTE PTR DS:[EDX+EBX-1],39           ;  注册码是否大于 9
004CD036  |.  76 0F         |JBE SHORT unpacked.004CD047
004CD038  |>  B8 78D14C00   |MOV EAX,unpacked.004CD178
004CD03D  |.  E8 96BAF8FF   |CALL unpacked.00458AD8
004CD042  |.  E9 DC000000   |JMP unpacked.004CD123
004CD047  |>  43            |INC EBX
004CD048  |.  48            |DEC EAX
004CD049  |.^ 75 C1         \JNZ SHORT unpacked.004CD00C              ;  以上判断输入的注册码是否为数字
004CD04B  |>  8D4D FC       LEA ECX,DWORD PTR SS:[EBP-4]              ;  ecx 为注册码在 stack 的位置 SS:[EBP
004CD04E  |.  8B15 40024E00 MOV EDX,DWORD PTR DS:[4E0240]             ;  unpacked.004E1B84
004CD054  |.  8B12          MOV EDX,DWORD PTR DS:[EDX]                ;  65843767 值入 edxPT
004CD056  |.  A1 F8014E00   MOV EAX,DWORD PTR DS:[4E01F8]
004CD05B  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]                ;  583818E7012F5957B080 入 eax]
004CD05D  |.  E8 4E87FEFF   CALL unpacked.004B57B0
004CD062  |.  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]             ;  583818E7012F5957B080 入 edx-
004CD065  |.  8B86 D8020000 MOV EAX,DWORD PTR DS:[ESI+2D8]
004CD06B  |.  E8 AC6CF6FF   CALL unpacked.00433D1C
004CD070  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]             ;  注册码入 eax
004CD073  |.  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
004CD076  |.  E8 01BAF3FF   CALL unpacked.00408A7C                     ;  判断注册码的第一位与最后一位是否为"-"
004CD07B  |.  BB 03000000   MOV EBX,3
004CD080  |>  8D45 EC       /LEA EAX,DWORD PTR SS:[EBP-14]
004CD083  |.  50            |PUSH EAX
004CD084  |.  B9 67760000   |MOV ECX,7667                              ;  7667 入 ecx
004CD089  |.  BA 91100000   |MOV EDX,1091                              ;  1091 入 edx
004CD08E  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]             ;  注册码入 eax
004CD091  |.  E8 1EA6FEFF   |CALL unpacked.004B76B4                   ;  进入　,产生 22956ed
004CD096  |.  8B55 EC       |MOV EDX,DWORD PTR SS:[EBP-14]           ;  依次产生 22956，9549 并入 edx
004CD099  |.  8D45 F8       |LEA EAX,DWORD PTR SS:[EBP-8]
004CD09C  |.  E8 276CF3FF   |CALL unpacked.00403CC8                   ;  依次把输入的注册码换为 22956，95498
004CD0A1  |.  4B            |DEC EBX
004CD0A2  |.^ 75 DC         \JNZ SHORT unpacked.004CD080
004CD0A4  |.  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
004CD0A7  |.  50            PUSH EAX
004CD0A8  |.  B9 C7850000   MOV ECX,85C7
004CD0AD  |.  BA E5480000   MOV EDX,48E5
004CD0B2  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004CD0B5  |.  E8 FAA5FEFF   CALL unpacked.004B76B4
004CD0BA  |.  8B55 E8       MOV EDX,DWORD PTR SS:[EBP-18]
004CD0BD  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]
004CD0C0  |.  E8 036CF3FF   CALL unpacked.00403CC8
004CD0C5  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004CD0C8  |.  8B15 B0034E00 MOV EDX,DWORD PTR DS:[4E03B0]             ;  unpacked.004E1B78
004CD0CE  |.  8B12          MOV EDX,DWORD PTR DS:[EDX]
004CD0D0  |.  E8 EB6EF3FF   CALL unpacked.00403FC0                   ;  此 call 会用到 eax 的值
004CD0D5      75 42         JNZ SHORT unpacked.004CD119              ;  关键跳
004CD0D7  |.  6A 00         PUSH 0
004CD0D9  |.  B9 88D14C00   MOV ECX,unpacked.004CD188
004CD0DE  |.  BA A4D14C00   MOV EDX,unpacked.004CD1A4
004CD0E3  |.  A1 44024E00   MOV EAX,DWORD PTR DS:[4E0244]
004CD0E8  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
004CD0EA  |.  E8 B551F8FF   CALL unpacked.004522A4
004CD0EF  |.  8D55 E0       LEA EDX,DWORD PTR SS:[EBP-20]
004CD0F2  |.  8B86 D8020000 MOV EAX,DWORD PTR DS:[ESI+2D8]
004CD0F8  |.  E8 1F6CF6FF   CALL unpacked.00433D1C
004CD0FD  |.  8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]
```



```
004CD100  |.  8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]

004CD103  |.  E8 74B9F3FF   CALL unpacked.00408A7C

004CD108  |.  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]

004CD10B  |.  E8 A4ABFEFF   CALL unpacked.004B7CB4

004CD110  |.  8BC6          MOV EAX,ESI

004CD112  |.  E8 491DF8FF   CALL unpacked.0044EE60

004CD117  |.  EB 0A         JMP SHORT unpacked.004CD123

004CD119  |>  B8 78D14C00   MOV EAX,unpacked.004CD178

004CD11E  |.  E8 B5B9F8FF   CALL unpacked.00458AD8               ; 出错框

004CD123  |>  33C0          XOR EAX,EAX

004CD125  |.  5A            POP EDX
```

### 三、去掉可恶的 NAG 窗口

使用`ollydbg`如下功能：
- 菜单：`Debug/animate over` 功能
- 在跳出`nag`窗口这句会停在一句`call`
- 向上看有没有跳转，没有的话，按`F7`进入，继续单步调试（Step Over/Trace），直到找到能跳过`NAG`的`CALL`。

代码如下：

```
004D059A     /7D 5D         JGE SHORT unpacked.004D05F9  ; 把这句改为 jmp SHORT unpacked.004D05F9

004D059C    . |33C0         XOR EAX,EAX

004D059E    . |55           PUSH EBP

004D059F    . |68 F2054D00  PUSH unpacked.004D05F2

004D05A4    . |64:FF30      PUSH DWORD PTR FS:[EAX]

004D05A7    . |64:8920      MOV DWORD PTR FS:[EAX],ESP

004D05AA    . |A1 E8004E00  MOV EAX,DWORD PTR DS:[4E00E8]

004D05AF    . |8338 00      CMP DWORD PTR DS:[EAX],0

004D05B2    . |75 17        JNZ SHORT unpacked.004D05CB

004D05B4    . |8B4D FC      MOV ECX,DWORD PTR SS:[EBP-4]

004D05B7    . |B2 01        MOV DL,1

004D05B9    . |A1 24D34C00  MOV EAX,DWORD PTR DS:[4CD324]

004D05BE    . |E8 BDAAF7FF  CALL unpacked.0044B080

004D05C3    . |8B15 E8004E00 MOV EDX,DWORD PTR DS:[4E00E8]      ; unpacked.004E1B60

004D05C9    . |8902         MOV DWORD PTR DS:[EDX],EAX

004D05CB    > |A1 E8004E00   MOV EAX,DWORD PTR DS:[4E00E8]

004D05D0    . |8B00         MOV EAX,DWORD PTR DS:[EAX]

004D05D2    . |8B10         MOV EDX,DWORD PTR DS:[EAX]

004D05D4    . |FF92 D8000000 CALL DWORD PTR DS:[EDX+D8]         ; => 跳出 nag

004D05DA    . |33C0         XOR EAX,EAX
```

#### 修改

将`004D059A /7D 5D JGE SHORT unpacked.004D05F9`这句改为`jmp SHORT unpacked.004D05F9`，会跳过`NaG`。

#### 备注

没法找到注册码，那位兄弟看看。

`4dd466d0e1e0cc123afbe504da2462ca`
