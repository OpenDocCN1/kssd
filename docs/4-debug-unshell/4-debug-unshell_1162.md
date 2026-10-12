
# easypdf1.5.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1053.htm`](https://www.pediy.com/kssd/pediy03/forum1053.htm)

**标题**：Visual Protect3.0 脱壳讨论

**所用工具**：`trw2000`, `peditor1.7`

**研究对象**：`easypdf1.5.2`

### 前言

Visual Protect3.0 是一款不错的加密软件，它可以给你的共享软件提供如使用总时间限制、使用总日期限制，使用次数限制、过期限制等，还可以定制试用版提示框。如果用户注册的话只需要利用网络传送一个小于 1K 的 `keyfile` 即可去除所有限制。在它的帮助里说：*Once a file or program has been protected with Visual Protect and distributed to your customers, its protection CANNOT BE REMOVED*。在这点上它好象有些太自信了。下面我们就来看看如何去除它的保护。

### 保护原理

利用 Visual Protect 保护一个软件时，它会在被保护软件同目录下放一个 `vp.dll`，再对被保护软件加一个外壳，利用这个外壳调用 `vp.dll`，利用 `vp.dll` 对授权文件 `*.vpl` 进行验证，验证通过后对程序解密运行。可见破解的方法可以有两种，一种是对被保护程序进行脱壳，一般脱壳后就不再会有使用日期等限制，而且可以脱离 `vp.dll` 运行。第二种是对 `vp.dll` 的破解。利用同一版本 Visual Protect 加密的软件的 `vp.dll` 是相同的，一旦破解了某个版本的 `vp.dll`，那所有利用这个版本的 Visual Protect 保护的软件的加密都会失效。

### 破解对象简介

`easypdf` 据说是一款不错的所见即所得的 `pdf` 文件制作工具（很遗憾它似乎不支持中文，用它制作的中文文档会显示乱码，我还没找到解决办法，修改过 `W32DASM` 的同志可以看看）。它与 Visual Protect 是同一家软件公司编写的，所以它加密用的 Visual Protect 应该是最典型的。安装后观察有 3 个文件是用 Visual Protect 加密的，分别是 `easypdf.exe`、`vspdf.dll` 和 `vp.dll`（他们好象真的很自信，这么重要的文件都用它来加密），这三个文件的脱壳都不完全相同，下面我们就分别讨论它们的脱壳方法。

### 脱壳实战

#### 一、easypdf.exe 的脱壳

运行 `trw2000`，运行命令 `faults off`，用 `trw2000` 载入 `easypdf.exe`，先出现一个试用提示框，按下'try'后会中断在 `easypdf.exe` 的入口 `6B5BC0`，往下走有 4 个 `CALL`，在第四个 `CALL` 按 F8 进入，进入后按 F10 一直往下走，直到：

```
0167:006B6C4A 8D4C2424        LEA       ECX,[ESP+24]

0167:006B6C4E 50              PUSH      EAX

0167:006B6C4F 8D54242C        LEA       EDX,[ESP+2C]

0167:006B6C53 51              PUSH      ECX

0167:006B6C54 52              PUSH      EDX

0167:006B6C55 FF1574AB6B00    CALL      `VP![NONAME]`    <-注意

0167:006B6C5B 8B442414        MOV       EAX,[ESP+14]

0167:006B6C5F 85C0            TEST      EAX,EAX

0167:006B6C61 740F            JZ        006B6C72

0167:006B6C63 8B0DD8AB6B00    MOV       ECX,[006BABD8]

0167:006B6C69 03C1            ADD       EAX,ECX

0167:006B6C6B 81C464030000    ADD       ESP,0364

0167:006B6C71 C3              RET
```

在 `6B6C55` 处按 F8 进入，运行命令 `D 401000`，你将会看到很多的"?"，不管它，一直用 F10 往下走，直到：

```
0167:008AD84D  MOV       EAX,[008B35F8]

0167:008AD852  CALL      0088CDDC

0167:008AD857  MOV       EAX,[EBP+08]

0167:008AD85A  CALL      NEAR [EAX]       <-注意

0167:008AD85C  MOV       EAX,[008B35AC]

0167:008AD861  MOV       EAX,[EAX]
```

经过 `8AD85A` 处的 `CALL` 后，你会发现原来显示"?"的地方可以看到内容了，所以这个 `CALL` 应该是对程序进行了解码，再往下走，直到：

```
0167:008AD909  MOV       EAX,008ACEC8

0167:008AD90E  CALL      008AE198

0167:008AD913  CMP       EAX,[008B66B0]

0167:008AD919  JZ        008AD92D

0167:008AD91B  MOV       EAX,[008B357C]

0167:008AD920  MOV       BYTE [EAX],00

0167:008AD923  MOV       EAX,[008B35F8]

0167:008AD928  CALL      0088CDDC

0167:008AD92D  JMP       NEAR [EBP-04]       <-注意

0167:008AD930  PUSH      BYTE +00

0167:008AD932  CALL      00836788

0167:008AD937  JMP       008ADD31

0167:008AD93C  MOV       EAX,[008B3824]

0167:008AD941  MOV       EAX,[EAX+41]
```

注意，程序将由上面的 `JMP` 跳到原始的入口 `67B9E0`，现在你可以把它 `DUMP` 下来，`DUMP` 完后用 F8 慢慢走，可到如下地方：

```
0167:00407DF4  JMP       NEAR [0068C304]

0167:00407DFA  MOV       EAX,EAX

0167:00407DFC  JMP       NEAR [0068C300]

0167:00407E02  MOV       EAX,EAX

0167:00407E04  JMP       NEAR [0068C2FC]

0167:00407E0A  MOV       EAX,EAX

0167:00407E0C  JMP       NEAR [0068C2F8]
```

根据经验，可以判断 `68C300` 前后应该是输入表的所在，再上下看看，看到如下：

```
0030:0068C1E0 00 00 00 00 00 00 00 00-00 00 00 00 EA F4 28 00 ............牯(.
0030:0068C1F0 84 CB 28 00 00 00 00 00-00 00 00 00 00 00 00 00 勊(.............
0030:0068C200 00 00 00 00 00 00 00 00-88 F7 6E 01 A0 AE 6E 01 ........堶 n.牣 n.
0030:0068C210 DC B4 6E 01 FC 7E 73 01-08 7F 73 01 5C 7F 73 01 艽 n.鼅 s..\s.
……………………
0030:0068CBD0 6C 76 7E 00 54 76 7E 00-38 76 7E 00 44 74 7E 00 lv~.Tv~.8v~.Dt~.
0030:0068CBE0 50 70 7E 00 74 70 7E 00-28 72 7E 00 F4 8D 7E 00 Pp~.tp~.(r~.魨~.
0030:0068CBF0 00 00 00 00 6B 65 72 6E-65 6C 33 32 2E 64 6C 6C ....kernel32.dll
0030:0068CC00 00 00 00 00 12 EA A3 E1-C7 13 22 14 43 51 31 25 .....辏崆.".CQ1%
```

根据经验判断输入地址表是从 `68C200` 到 `68CBF0`，其中的 `016EF788` 等都是被加密的函数入口地址，例如：程序通过 `JMP NEAR [0068C208]` 调用某个函数，`[0068C208]` 中应存放有这个函数的入口地址，现在被加密放入了 `016EF788`，程序跳到 `16EF788` 处的代码为：

```
0167:016EF788  MOV       EAX,009311E8
0167:016EF78D  JMP       EAX
```

程序将跳到 `9311E8`，`9311E8` 处的代码为：

```
0167:009311E8  MOV       EAX,BFF816F3
0167:009311ED  JMP       EAX
```

程序将跳到 `BFF816F3`，`BFF816F3` 处的代码为：

```
KERNEL32!GetCurrentThreadId
0167:BFF816F3  MOV       EAX,[BFFCADE0]
0167:BFF816F8  PUSH      DWORD [EAX]
0167:BFF816FA  CALL      BFF80455
```

所以 `JMP NEAR [0068C208]` 调用的应该是 `KERNEL32!GetCurrentThreadId` 函数，`[0068C208]` 中放的本应是 `BFF816F3`。原理很简单，下面就是如何修复了。可是我很遗憾的发现常用的修复输入表的工具对它都失效了。看样子我们得另想办法。一种是我们先象刚才一样通过手动将正确的地址逐个写入，写入以后再利用工具完全修复，但是这样太费时间了，估计一天都可能干不完。看来得另想方法。另一个方法是考虑如何让它不加密。刚才我们已经讲了 `008AD85A` 处的 `CALL [EAX]` 是对程序进行解码，那输入表的加密肯定是再这个之后，所以再次运行，通过这个 `CALL` 后我们下命令：

`bpx GetProcAddress`

然后 `G`，停下来后用 F12 走几次返回，看到如下代码：

```
0167:008C5BEA  PUSH      DWORD [EBP+0C]
0167:008C5BED  PUSH      ECX
0167:008C5BEE  CALL      `KERNEL32!GetProcAddress`
0167:008C5BF4  PUSH      BYTE +08       <-返回到这
0167:008C5BF6  PUSH      BYTE +00
0167:008C5BF8  MOV       [EBP+0C],EAX
0167:008C5BFB  CALL      `KERNEL32!LocalAlloc`
0167:008C5C01  MOV       ESI,EAX
0167:008C5C03  LEA       EAX,[EBP+0C]
0167:008C5C06  PUSH      BYTE +04
0167:008C5C08  PUSH      EAX
0167:008C5C09  LEA       EAX,[ESI+01]
0167:008C5C0C  MOV       BYTE [ESI],B8
0167:008C5C0F  PUSH      EAX
0167:008C5C10  CALL      008C6320
0167:008C5C15  ADD       ESP,BYTE +0C
0167:008C5C18  OR        BYTE [ESI+05],FF
0167:008C5C1C  MOV       BYTE [ESI+06],E0
0167:008C5C20  MOV       EAX,ESI
0167:008C5C22  POP       ESI
0167:008C5C23  POP       EBP
0167:008C5C24  RET
```

返回时 `EAX` 中放的就是正确的入口地址。按 F12 两次后走到：

```
0167:0089EEB2  CALL      00836858
0167:0089EEB7  CALL      0089EDF0       <-到这，注意这个 CALL
0167:0089EEBC  POP       ECX
0167:0089EEBD  MOV       EAX,[EBP+08]
0167:0089EEC0  MOV       EBX,[EAX-04]   <-注意此时的 EBX 的值
0167:0089EEC3  MOV       EAX,[EBP+08]
0167:0089EEC6  MOV       ECX,[EAX-04]
0167:0089EEC9  MOV       EDX,[EBP-04]
0167:0089EECC  MOV       EAX,EDI
0167:0089EECE  MOV       ESI,[EAX]
0167:0089EED0  CALL      NEAR [ESI+38]
0167:0089EED3  XOR       EAX,EAX
```

走几圈后你会发现规律，就是上面取出的 `EBX` 的值就是最后放入输入表的值，那这个值是如何产生的呢？进入上面的 `CALL` 看个究竟：

```
0167:0089EDF0  PUSH      EBP
0167:0089EDF1  MOV       EBP,ESP
0167:0089EDF3  PUSH      ECX            <-入栈，此时 ECX 的值就是正确的人口地址
0167:0089EDF4  MOV       [EBP-04],EAX   <-破坏堆栈内的正确人口地址
0167:0089EDF7  MOV       EAX,07
0167:0089EDFC  CALL      0083279C
0167:0089EE01  MOV       EDX,[EBP+08]
0167:0089EE04  MOV       [EDX-04],EAX   <-保存加密后的入口地址
0167:0089EE07  MOV       EAX,[EBP+08]
0167:0089EE0A  MOV       EAX,[EAX-04]
0167:0089EE0D  MOV       BYTE [EAX],B8
0167:0089EE10  MOV       EAX,[EBP+08]
0167:0089EE13  MOV       EAX,[EAX-04]
0167:0089EE16  LEA       EDX,[EAX+01]
0167:0089EE19  LEA       EAX,[EBP-04]
0167:0089EE1C  MOV       ECX,04
0167:0089EE21  CALL      008328B4
0167:0089EE26  MOV       EAX,[EBP+08]
0167:0089EE29  MOV       EAX,[EAX-04]
0167:0089EE2C  MOV       BYTE [EAX+05],FF
0167:0089EE30  MOV       EAX,[EBP+08]
0167:0089EE33  MOV       EAX,[EAX-04]
0167:0089EE36  MOV       BYTE [EAX+06],E0
0167:0089EE3A  POP       ECX
0167:0089EE3B  POP       EBP
0167:0089EE3C  RET
```

我们可以在内存中对这些代码稍做修改，如下：

```
0167:0089EDF0 55              PUSH      EBP
0167:0089EDF1 8BEC            MOV       EBP,ESP
0167:0089EDF3 51              PUSH      ECX            <-入栈，此时 ECX 的值就是正确的人口地址
0167:0089EDF4 90              NOP                    <-去掉了破坏刚入栈的正确地址的代码
0167:0089EDF5 90              NOP
0167:0089EDF6 90              NOP
0167:0089EDF7 B807000000      MOV       EAX,07
```



# 一、`easypdf.exe` 的脱壳（续）

```
0167:0089EDFC E89B39F9FF     CALL    0083279C

0167:0089EE01 8B5508         MOV     EDX,[EBP+08]

0167:0089EE04 8942FC         MOV     [EDX-04],EAX

0167:0089EE07 8B4508         MOV     EAX,[EBP+08]

0167:0089EE0A 8B40FC         MOV     EAX,[EAX-04]

0167:0089EE0D C600B8         MOV     BYTE [EAX],B8

0167:0089EE10 8B4508         MOV     EAX,[EBP+08]

0167:0089EE13 8B40FC         MOV     EAX,[EAX-04]

0167:0089EE16 8D5001         LEA     EDX,[EAX+01]

0167:0089EE19 8D45FC         LEA     EAX,[EBP-04]

0167:0089EE1C B904000000     MOV     ECX,04

0167:0089EE21 E88E3AF9FF     CALL    008328B4

0167:0089EE26 8B4508         MOV     EAX,[EBP+08]

0167:0089EE29 59             POP     ECX                 <-取出正确的地址

0167:0089EE2A 8948FC         MOV     [EAX-04],ECX        <-替换掉被加密的入口地址

0167:0089EE2D 5D             POP     EBP

0167:0089EE2E C3             RET
```

这样修改后放入输入表的就将是未被加密的函数入口地址。

运行起来后我们就可以用 `Import REConstructor` 来进行输入表的完全修复，具体操作可看相关文章。须注意的是修复时发现还有两个函数的入口地址不对。仔细研究发现它们对应的都是函数 `KERNEL32!GetProcAddress`，需手工选择一下，再将程序的入口修正为正确入口就可以了。至此，`easypdf.exe` 脱壳完成。

## 二、`vspdf.dll` 的脱壳

`dll` 文件的脱壳一般较少被提到，开始我也没有注意，只是因为发现在 `easypdf.exe` 运行前已经出现一个试用框，然后寻找原因才发现是 `vspdf.dll` 被加壳了。下面我们就来看如何脱它。

要脱壳先得找到它的入口点，但是 `dll` 文件被载入的基址并不是我们静态观察文件头看到的基址，那如何找到他的入口呢？我们可以用 `peditor1.7` 来找，先按"browse"按钮打开 `vspdf.dll`，取得它的入口偏移为 `6CBC0`，再运行 `easypdf`，运行好后按"tasks"按钮，在上半窗口中点 `easypdf.exe`，然后可在下半窗口中找到 `vspdf.dll` 的基址为 `7B0000`，所以正确入口应该是 `81CBC0`。

运行 `trw2000`，下命令 `faults off` 和 `bp 81CBC0`。运行 `easypdf`，中断后我们可看到如下代码：

```
0167:0081CBC0  MOV     EAX,[ESP+08]
0167:0081CBC4  CMP     EAX,BYTE +01
0167:0081CBC7  JNZ     0081CC04
0167:0081CBC9  MOV     [00821B9C],EAX
0167:0081CBCE  MOV     EAX,[ESP+04]
0167:0081CBD2  PUSH    EAX
0167:0081CBD3  MOV     [00821BD8],EAX
0167:0081CBD8  CALL    0081D540            <-进入
0167:0081CBDD  ADD     ESP,BYTE +04
0167:0081CBE0  MOV     [00820704],EAX
0167:0081CBE5  TEST    EAX,EAX
0167:0081CBE7  JNZ     0081CBEC
0167:0081CBE9  RET     0C
0167:0081CBEC  MOV     ECX,[ESP+0C]
0167:0081CBF0  MOV     EDX,[00821BD8]
0167:0081CBF6  PUSH    ECX
0167:0081CBF7  PUSH    BYTE +01
0167:0081CBF9  PUSH    EDX
0167:0081CBFA  MOV     [00820700],EAX
0167:0081CBFF  CALL    EAX                 <-注意此时 EAX 的值
0167:0081CC01  RET     0C
0167:0081CC04  MOV     ECX,[ESP+0C]
0167:0081CC08  MOV     EDX,[00821BD8]
0167:0081CC0E  PUSH    ECX
0167:0081CC0F  PUSH    EAX
0167:0081CC10  PUSH    EDX
0167:0081CC11  CALL    NEAR [00820700]
0167:0081CC17  RET     0C
```

第一个 `JNZ` 是不跳的，我们进入 `81CBD8` 的 `CALL`，然后就跟 `easypdf.exe` 查不多了。不同的是对输入表加密完后继续走会回到 `81CBDD`，那哪是正确的入口呢？我们继续往下走，走到 `81CBFF` 的 `CALL EAX`，然后发现此时 `EAX` 的值为 `7B1000`，这不是正确入口吗？好了在入口处我们可以 `DUMP` 它，不过发现用 `Procdump` 无法 `DUMP`，但是 `peditor` 可以，`DUMP` 下来后再按上面的方法修复输入表，修复时发现一个奇怪的现象，就是 `vspdf.dll` 的输入地址表有两份，交错地放在一起，不知道为什么？其中一份已经初始化好了，我们只要把这一份修复就可以了。还有一个问题就是 `Import REConstructor` 修复的 `vspdf.dll` 的输入表变得让我看不懂了。我竟然找不到函数名，但是却可以初始化，谁能帮我解释一下。

## 三、`vp.dll` 的脱壳

可运用上面相同的方法找到它的现在的人口地址和原来的入口地址。不同的是第一个 `CALL` 进入后的流程与前面不同，当然了，因为这时候 `vp.dll` 还没有完全运行。可以看到如下代码：

```
0167:008C62CE  MOV     EAX,[ESP+04]
0167:008C62D2  MOV     [008CAB5C],EAX
0167:008C62D7  CALL    008C5E2D
0167:008C62DC  CALL    008C6085
0167:008C62E1  CALL    008C5A47
0167:008C62E6  MOV     EAX,[008CAB5C]
0167:008C62EB  CMP     EAX,[008CA7C5]
0167:008C62F1  JZ      008C6301
0167:008C62F3  CMP     DWORD [008CA7BD],BYTE +00
0167:008C62FA  JZ      008C6301
0167:008C62FC  CALL    008C5AA5
0167:008C6301  CALL    008C5CAF
0167:008C6306  MOV     EAX,[008CAB5C]
0167:008C630B  MOV     ECX,[008CA7B9]
0167:008C6311  ADD     EAX,ECX
0167:008C6313  RET
```

我们可以走到 `8C6301` 后下 `bpx GetProcAddress` 找到关键的输入表加密处。走几圈你会找到规律。关键代码如下：

```
0167:008C5E03  PUSH    EAX
0167:008C5E04  PUSH    EBP
0167:008C5E05  CALL    008C5C25            <-关键的 CALL
0167:008C5E0A  POP     ECX
0167:008C5E0B  POP     ECX
0167:008C5E0C  CMP     EAX,EDI
0167:008C5E0E  MOV     [ESI],EAX           <-将被加密的入口地址放入
0167:008C5E10  JNZ     008C5E18
0167:008C5E12  MOV     DWORD [ESI],DEADBEEF <-将被加密的入口地址放入
0167:008C5E18  ADD     ESI,BYTE +04
0167:008C5E1B  JMP     SHORT 008C5DDA
```

我们可以参照前面的改法修改，但是这儿我们可以尝试另一种方法。你可能注意到了，在现在的输入表里，函数名部分还是被加密的，我们能否完全修复它呢？

进入上面的 `CALL` 看一下：

> 我把它改成了这样：
>
> ```
> 0167:008C5C25 55             PUSH    EBP
> 0167:008C5C26 8BEC           MOV     EBP,ESP
> 0167:008C5C28 81EC00010000   SUB     ESP,0100
> 0167:008C5C2E 8B450C         MOV     EAX,[EBP+0C]
> 0167:008C5C31 56             PUSH    ESI
> 0167:008C5C32 0FB630         MOVZX   ESI,BYTE [EAX]
> 0167:008C5C35 40             INC     EAX
> 0167:008C5C36 50             PUSH    EAX
> 0167:008C5C37 56             PUSH    ESI
> 0167:008C5C38 50             PUSH    EAX
> 0167:008C5C39 48             DEC     EAX
> 0167:008C5C3A 90             NOP
> 0167:008C5C3B 90             NOP
> 0167:008C5C3C 90             NOP
> 0167:008C5C3D 90             NOP
> 0167:008C5C3E 50             PUSH    EAX
> 0167:008C5C3F E8DC060000     CALL    008C6320
> 0167:008C5C44 80243000       AND     BYTE [EAX+ESI],00
> 0167:008C5C48 90             NOP
> 0167:008C5C49 90             NOP
> 0167:008C5C4A 90             NOP
> 0167:008C5C4B 90             NOP
> 0167:008C5C4C 6A00           PUSH    BYTE +00
> 0167:008C5C4E 90             NOP
> 0167:008C5C4F 90             NOP
> 0167:008C5C50 90             NOP
> 0167:008C5C51 90             NOP
> 0167:008C5C52 90             NOP
> 0167:008C5C53 90             NOP
> 0167:008C5C54 56             PUSH    ESI
> 0167:008C5C55 50             PUSH    EAX
> 0167:008C5C56 E815000000     CALL    008C5C70
> 0167:008C5C5B 83C418         ADD     ESP,BYTE +18
> 0167:008C5C5E 58             POP     EAX
> 0167:008C5C5F 48             DEC     EAX
> 0167:008C5C60 90             NOP
> 0167:008C5C61 90             NOP
> 0167:008C5C62 90             NOP
> 0167:008C5C63 90             NOP
> 0167:008C5C64 50             PUSH    EAX
> 0167:008C5C65 FF7508         PUSH    DWORD [EBP+08]
> 0167:008C5C68 E8DCFEFFFF     CALL    008C5B49
> 0167:008C5C6D 5E             POP     ESI
> 0167:008C5C6E C9             LEAVE
> 0167:008C5C6F C3             RET
> ```
>
> 你对照着走一偏就能明白它的意思。最后还要把 `8C5E0E` 和 `8C5E12` 处的两句 `NOP` 掉，让它不初始化。然后返回到程序的真正入口。这时候的输入表就是完全被解密的未初始化的输入表，接下来就可以 `DUMP` 了，`DUMP` 完后就把这个进程杀掉，不然由于输入表未初始化，再运行就是非法操作。将 `DUMP` 下来的程序的入口和输入表地址修改一下就可以使用了。

> 问题：上面的方法在本机上经过试验都成功了。对于 `vp.dll` 的输入表的修复方法是否可以运用在前两个文件上面呢？应该也是可以的，但修改会比较复杂一些，各位可以自己试一下。还有所有脱壳都是在我的 98 上进行的，脱壳后的 `vp.dll` 在 `NT4.0` 上运行会产生初始化错误，估计应该不是输入表的问题，哪位可以找出原因。前面两个脱壳文件未试。

> 由于本人水平有限，如有错误之处请指正。

> `hying[CCG]`

> 2001 年 6 月 10 日 `4dd466d0e1e0cc123afbe504da2462ca`
