
# 实战 DES 算法注册机（破解教程）

> 原文：[`www.pediy.com/kssd/pediy10/59285.html`](https://www.pediy.com/kssd/pediy10/59285.html)

从看雪下了不少有用的工具，一直想回报一下，可惜没有拿得出手的作品，最近破了某连连看助手的的注册码算法（其实也是练手）。

在这里写一些破解关键点，希望对大家有用。

这款软件似乎已经成了 `crackme`，爆破和内存注册机的教程到处都是，只是算法注册机还没有见过。

我只是以练手为目的，却发现它是典型的 `DES` 算法，适合非常好的样本（实际的软件总比 `crackme` 真实），于是拿出来剖析一下。

我已经把算法抠下来写了个 `c` 的，又用这个 `c` 的写了个 `MFC` 的注册机，需要者可直接联系我，因为没法上传附件，请见谅。

由于算法代码太多，就不整段贴了，只贴关键的部分。

先用 `peid` 探壳，`"Borland Delphi 6.0 - 7.0"`，省事了。

开始破解，第一反应是搜索注册失败的字符串，打开 `W32dasm`，找到 `Nothing`。

一般情况下，可能是因为中文被 `W32dasm` 显示成乱码，所以直接打开 `eXeScope`，还是没有...暴迷惑。

怒！直接上 `DeDe`，发现 `TForm_Reg`（一般 `Form` 的名字都很直白），`"确认"` 按钮的事件是 `"btn_ConfirmClick"`，地址 `00470138`。

开 `Ollydbg`，开始调试。

在 `00470138` 下断点，按下 `"确认"` 键进入...这个函数里有 `12` 个 `call`，头直接变成 `6` 个大...

**注：预备知识。**

1. `call` 是调用函数（废话），参数一般在前面紧跟着 `call`，`Borland Delphi` 的参数常用寄存器传递。
2. `EBP` 很重要，指向堆栈中函数与调用者的分解，`EBP-n` 代表函数内的变量，`EBP+n` 一般是通过堆栈传递的参数（寄存器不够用等情况会用堆栈传递）。

一个个 `call` 看吧...

```
00470153  .  B8 C8000000   MOV EAX,0C8
00470158  .  E8 EB68FFFF   CALL llkzs.00466A48
```

不知道...跟进去看看。

幸好 `OD` 有注释 `"GetTickCount"`，估计是去注册次数，因为这个程序有注册 `3` 次的限制（纯属猜测，我没验证）。

```
0047015D  .  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
00470160  .  8B83 10030000 MOV EAX,DWORD PTR DS:[EBX+310]
00470166  .  E8 3111FDFF   CALL llkzs.0044129C
```

`EBX+310` 里太远了，监视一下 `EBP-8` 吧

运行 `call` 后的结果

```
0013F258  00D8EFC0  ASCII "12345678"
```

是输入的假注册码，那就可能是读界面输入了。

```
0047016B  .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
0047016E  .  8D4D FC       LEA ECX,DWORD PTR SS:[EBP-4]
00470171  .  BA 34024700   MOV EDX,llkzs.00470234             ;  ASCII "kc"
00470176     E8 F985FFFF   CALL llkzs.00468774
```

观察寄存器

`EAX` `00D8EFC0` `ASCII "12345678"`

`ECX` `0013F25C`

`EDX` `00470234` `ASCII "kc"`

`EDX` 不知道干吗用，监视 `0013F25C` 吧

```
0013F25C  00000000
```

不是吧，先放着。

```
0047017F  .  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
00470182  .  8B83 0C030000 MOV EAX,DWORD PTR DS:[EBX+30C]
00470188  .  E8 0F11FDFF   CALL llkzs.0044129C
```

又一个 `CALL llkzs.0044129C` 读界面到 `EBP-10`。（这时候暴想念 `IDA`，哪个 `call` 一改名所有的都改了，可惜是静态工具）

结果 `0013F250` `00D927F8` `ASCII "0200508"`，读到机器码。

```
0047018D  .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]
00470190  .  B9 64000000   MOV ECX,64
00470195  .  BA 03000000   MOV EDX,3
0047019A  .  E8 1943F9FF   CALL llkzs.004044B8
```

监视 `EBP-10` 得到 `0013F254` `00D8BF70` `ASCII "00507"`

到现在为止，这 `4` 个函数运行的结果是

```
0013F250  00D927F8  ASCII "0200507"
0013F254  00D8BF70  ASCII "00507"
0013F258  00D8EFC0  ASCII "12345678"
0013F25C  00000000
```

而下一个函数

```
0047019F  .  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]
004701A2  .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004701A5  .  E8 FA41F9FF   CALL llkzs.004043A4
004701AA  .  75 31         JNZ SHORT llkzs.004701DD
```

其中

`EAX` `00000000`

`EDX` `00D8BF70` `ASCII "00507"`

估计是比较了。

可以调试两边，更改 `"JNZ"`，看跳与不跳有什么不同。

结果发现这就是区别注册成功与否的比较。要是爆破的话，到这里就结束了，改成 `NOP`。

下面的 `call` 不看了。关键是 `CALL llkzs.00468774`，`CALL llkzs.004044B8` 和 `CALL llkzs.004043A4`（因为一共就 `4` 个 `call`，那个还确定是取界面句柄了）。

先进最后一个 `CALL llkzs.004043A4` 吧，`"比较"` 可能会简单一些。

这个实在没的可看，一大堆判断，进行循环比较，看两个参数是否完全相等。

**注：很多比较时的技巧。**

画流程图，根据汇编语言一条条的话称流程图就一目了然了...不过就这破函数，画了一张纸。

接下来进 `CALL llkzs.004044B8` 吧，知道结果是 `"00507"`，应该容易一些。

一片比较，判断是否位空字符串后调用了两个不知是什么的 `call`。

还用老办法跟踪

```
004044D5 |> /01C2          ADD EDX,EAX
004044D7 |. |8B4424 08     MOV EAX,DWORD PTR SS:[ESP+8]
004044DB |. |E8 A8FBFFFF   CALL llkzs.00404088
```

`EAX` `0013F254`

`EDX` `00D92D46` `ASCII "00507"`

发现它只是写入堆栈中...无语，竟然只是去除机器码的前两位，简单得流泪。

看来只剩最后一个了，也是复杂的出乎我意料的一个 `CALL llkzs.00468774`。

```
0046878C  .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0046878F  .  E8 B4BCF9FF   CALL llkzs.00404448
00468794  .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
00468797  .  E8 ACBCF9FF   CALL llkzs.00404448
```

连调了两个 `CALL llkzs.00404448`，我是没看出干什么用（我不会 `Delphi`），不过 `DeDe` 的提示是

`* Reference to: System.@LStrAddRef(void;void):Pointer;`

估计和分配空间有关，也就不管了。

```
004687AA  .  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
004687AD  .  E8 E6B7F9FF   CALL llkzs.00403F98
004687B2  .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004687B5  .  E8 9EBAF9FF   CALL llkzs.00404258
```

依然不知道是什么。`DeDe` 给的是

`* Reference to: System.@LStrClr(void;void);`
`* Reference to: System.@LStrLen(String):Integer;`

想想 `CALL llkzs.00404258` 确实返回了 `EAX` `00000008`

不过无所谓了，一般和系统有关的都和我没什么关系，我在意的是作者自己写的部分（注册码计算部分）。

下面是一个循环

还是通过调试

```
004687D3  .  42            |INC EDX
004687D4  .  B9 02000000   |MOV ECX,2
004687D9  .  8B45 FC       |MOV EAX,DWORD PTR SS:[EBP-4]
004687DC  .  E8 D7BCF9FF   |CALL llkzs.004044B8
```

`call` 前

`EAX` `00D838D4` `ASCII "12345678"`

`ECX` `00000002`

`EDX` `00000001`

`call` 后

```
0013F224  00D81C58  ASCII "12"
```

```
004687E1  .  8B45 F0       |MOV EAX,DWORD PTR SS:[EBP-10]
004687E4  .  E8 D3FEFFFF   |CALL llkzs.004686BC
```

`call` 前 `EAX` `00D8D9D0` `ASCII "12"`

`call` 后 `EAX` `00000012`

```
004687E9  .  8BD0          |MOV EDX,EAX
004687EB  .  8D45 EC       |LEA EAX,DWORD PTR SS:[EBP-14]
004687EE  .  E8 8DB9F9FF   |CALL llkzs.00404180
```

`call` 前

`EAX` `0013F220`

`EDX` `00000012`

```
0013F220  00000000
```

`call` 后

`EAX` `00000012`

```
0013F220  00D930CC
```

```
004687F3  .  8B55 EC       |MOV EDX,DWORD PTR SS:[EBP-14]
004687F6  .  8D45 F4       |LEA EAX,DWORD PTR SS:[EBP-C]
004687F9  .  E8 62BAF9FF   |CALL llkzs.00404260
```

`call` 前

`EAX` `0013F228`

`EDX` `00D930CC`

```
0013F228  00000000
00D930CC  00000012
```

`call` 后

```
0013F228  00D930CC
00D930CC  00000012
```

跟一会儿这个循环就会发现，它是将 `string` 转为 `num`，结果是

```
00D92D44  78563412
```

这个函数还剩两个 `call` 了，而且最后一个

```
0046881C  |> /8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]
0046881F  |. |BA 05000000   MOV EDX,5
00468824  |. |E8 93B7F9FF   CALL llkzs.00403FBC
```

`DeDe` 给的是`* Reference to: System.@LStrArrayClr(void;void;Integer);`

跟踪也发现它的作用是清除内存。

所以真正的算法在

```
00468802  |> \\8BCF         MOV ECX,EDI
00468804  |.  8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]
00468807  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
0046880A  |.  E8 19FDFFFF   CALL llkzs.00468528
```

这时

`EAX` `00D81C58`

`ECX` `0013F25C`

`EDX` `00470234` `ASCII "kc"`

```
00D81C58  78563412
0013F25C  00000000
```

看来是将计算结果写入 `0013F25C`

进入函数

`CALL llkzs.00404448`
`CALL llkzs.00404258`
`CALL llkzs.00404260`
`CALL llkzs.00403F98`

等前面出现过的，就不再讨论。

跟踪发现第一个循环的作用是在 `EBP-8` 的 `0013F1F8` `00D92CA4` `ASCII "kc"`填充

```
00D92CA4  0000636B
00D92CA8  00000000
```

`636B` 就是 `"kc"`

紧接着又是一个循环将 `EBP-8` 的 `64` 位拷到 `EBP-24`

```
0013F1DC  0000636B
0013F1E0  00000000
```

**注：汇编有些冗余是很正常的，而且很多循环都是用作 `copy`。**

```
0046859E  .  B9 2C4C4700   MOV ECX,llkzs.00474C2C
004685A3  .  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]
004685A6  .  BA 07000000   MOV EDX,7
004685AB  .  E8 64F9FFFF   CALL llkzs.00467F14
```

`EAX` `0013F1DC` `ASCII "kc"`

`ECX` `00474C2C` `llkzs.00474C2C`

`EDX` `00000007`

暂时不知道作什么用，跟入。

第一个小循环，把 `EAX` 中的 `"kc"` 压入栈中。

接着

```
00467F37  .  8BCE          MOV ECX,ESI
00467F39  .  E8 46FEFFFF   CALL llkzs.00467D84
```

此时 `ECX` `0013F1A1`，`EAX` 指向 `"kc"`。

运行结果

```
0013F1A1  00030300
0013F1A5  XX100030
```

第一个算法函数终于出现了，跟入。

第一个循环还是将 `"kc"` 压栈

```
00467DA4  .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
00467DA7  .  33C9          XOR ECX,ECX
00467DA9  .  BA 07000000   MOV EDX,7
00467DAE  .  E8 B1AEF9FF   CALL llkzs.00402C64
```

`DeDe` 的提示是`* Reference to: System.@FillChar(void;void;Integer;Char);`

在 `EAX` 指向的地址清出 `7` 个 `0x00`。

```
0013F1A1  00000000
0013F1A5  XX000000
```

到此准备工作就绪，开始进行算法了

```
00467DB5  .  B8 E4324700   MOV EAX,llkzs.004732E4
```

`EAX` 指向了一个莫名其妙的东西

```
004732E4  20283038
004732E8  00081018
004732EC  21293139
004732F0  01091119
004732F4  222A323A
004732F8  020A121A
004732FC  232B333B
00473300  262E363E
00473304  060E161E
00473308  252D353D
0047330C  050D151D
00473310  242C343C
00473314  040C141C
00473318  030B131B
0047331C  170A100D
00473320  1B020400
```

开始循环

1. 取出 `EAX` 的第一个 `BYTE` 位 `0x38`；
2. 计算 `ESI=1<<(7-(0x38 & 7))`；
3. 将 `"kc"` 的第 `0x38/8` 个 `BYTE` 位赋给 `ECX`；
4. 如果 `ECX & ESI` 不为 `0` 就写入 `EBP-8` 指向的地址的第 `7` 位；

循环 `0x38` 次



# 反汇编代码分析：DES 算法逆向

## 1. 汇编代码逻辑解析

### 1.1 位操作与移位

`ESI = 1 << (7 - (0x38 & 7))`

`ECX = (&"kc")[0x38/8]` （"kc"的第 `0x38/8` 个 `BYTE`）

这两个合起来考虑 `ESI` 就成了 `"kc"` 的第 `0x38` 位。而这个循环的作用也就是将 `"kc"` 的第 `0x38` 位写到 `[EBP-8]` 的第 `7` 位。

`0x38 = 56 = 7 * 8`，也就填充了 `CALL llkzs.00402C64` 清空的那 `BYTE`

### 1.2 Code1 生成

`0013F1A1` `00030300`

`0013F1A5` `XX100030`

称之为 `code1`。

**注**：这是典型的换位操作，换位表在 `EAX` 中，这种操作在这个程序中很常见。遇到 `MOV EAX, llkzs.004732E4`，就要注意这可能是换位表。

### 1.3 Code2 生成

出了 `CALL llkzs.00467D84`，继续跟踪，看到了一片移位操作产生 `code2`。

`code2[0] = code1[0] >> 4`

`code2[1] = (code1[0] << 4) | (code1[1] >> 4)`

`code2[2] = (code1[1] << 4) | (code1[2] >> 4)`

`code2[3] = (code1[2] << 4) | (code1[3] >> 4)`

`code2[4] = (code1[3] & 0x0F)`

`code2[5] = code1[4]`

`code2[6] = code1[5]`

`code2[7] = code1[6]`

得到 `code2`

`0013F199` `10003000`

`0013F19D` `30300000`

### 1.4 循环移位与 Code3 生成

下面

```
00467F9D |.  BB 4C334700  MOV EBX, llkzs.0047334C
0047334C  02020101
00473350  02020202
00473354  02020201
00473358  01020202
```

循环 16(0x10)次

先调

```
00467FA5 |> /8D45 F1       /LEA EAX, DWORD PTR SS:[EBP-F]
00467FA8 |. |8A0B         |MOV CL, BYTE PTR DS:[EBX]
00467FAA |. |BA 03000000  |MOV EDX, 3
00467FAF |. |E8 00FFFFFF  |CALL llkzs.00467EB4
```

`EAX = &code2[0]`

`CL = 1`

跟入，发现是对 `code2[0-3]` 作循环左移 `CL` 位得到 `code2*[0-3]`。

又调

```
00467FB4 |.  8D45 ED      |LEA EAX, DWORD PTR SS:[EBP-13]
00467FB7 |.  8A0B         |MOV CL, BYTE PTR DS:[EBX]
00467FB9 |.  BA 03000000  |MOV EDX, 3
00467FBE |.  E8 F1FEFFFF  |CALL llkzs.00467EB4
```

看来是将 `code2[4-7]` 循环左移 `CL` 位得到 `code2*[4-7]`。

之后又是一片移位得到 `code3`

`code3[0] = (code2*[0] << 4) | (code2*[1] >> 4)`

`code3[1] = (code2*[1] << 4) | (code2*[2] >> 4)`

`code3[2] = (code2*[2] << 4) | (code2*[3] >> 4)`

`code3[3] = (code2*[3] << 4) | code2*[4]`

`code3[4] = code2*[5]`

`code3[5] = code2*[6]`

`code3[6] = code2*[7]`

得到 `code3`

`0013F192` `00060600`

`0013F196` `XX200060`

### 1.5 调用与运行结果

然后调用

```
0046801A |.  8BCE        |MOV ECX, ESI
0046801C |.  8D45 E6     |LEA EAX, DWORD PTR SS:[EBP-1A]
0046801F |.  BA 06000000 |MOV EDX, 6
00468024 |.  E8 F3FDFFFF |CALL llkzs.00467E1C
```

`EAX 0013F192`

`ECX 00474C2C llkzs.00474C2C`

`EDX 00000006`

`EAX = code3`

跟入，发现和 `CALL llkzs.00467D84` 一样的换位操作。

换位表在 `EAX` 中

`0047331C` `170A100D`

`00473320` `1B020400`

`00473324` `0914050E`

`00473328` `030B1216`

`0047332C` `060F0719`

`00473330` `010C131A`

`00473334` `241E3328`

`00473338` `271D362E`

`0047333C` `2F202C32`

`00473340` `3726302B`

`00473344` `292D3421`

`00473348` `1F1C2331`

流程很清楚，可以直接跳出 `CALL llkzs.00467F14` 了。

运行结果

`00474C2C` `00008880`

`00474C30` `20808080`

`00474C34` `88200002`

`00474C38` `20040620`

`00474C3C` `10400110`

`00474C40` `22000200`

`00474C44` `04708000`

`00474C48` `40840009`

`00474C4C` `50000002`

`00474C50` `41000322`

`00474C54` `10080080`

`00474C58` `08040001`

`00474C5C` `10400009`

`00474C60` `40000014`

`00474C64` `20000898`

`00474C68` `00000114`

`00474C6C` `08020068`

`00474C70` `10002001`

`00474C74` `81042009`

`00474C78` `04000200`

`00474C7C` `00020488`

`00474C80` `10200050`

`00474C84` `88104400`

`00474C88` `04004020`

暂时不知道有什么用。

### 1.6 大循环与 SN 处理

下面有个判断，如果 `SN` 位数小于 `8` 直接 `over`，看来刚才输入的 `sn`：`78563412` 只相当于 `4` 位。下断点，运行，重新输入 `sn`：`1234567812345678`，再运行到这个位置，通过。

大循环一次取 `8` 个 `BYTE` 的 `SN`

循环复制就不分析了，直接进入

```
004685FC |.  8D55 EC      |LEA EDX, DWORD PTR SS:[EBP-14]
004685FF |.  B9 07000000  |MOV ECX, 7
00468604 |.  B0 01        |MOV AL, 1
00468606 |.  E8 61FBFFFF  |CALL llkzs.0046816C
```

`EDX` 指向 `SN`

`0013F1EC` `78563412`

`0013F1F0` `78563412`

到

```
004681A4 |.  8B45 0C      MOV EAX, DWORD PTR SS:[EBP+C]
004681A7 |.  8B55 08      MOV EDX, DWORD PTR SS:[EBP+8]
004681AA |.  E8 81F9FFFF  CALL llkzs.00467B30
```

`EAX` 指向 `SN`

又是一个换位操作得到 `code1`

之后循环 16 次调用

```
004681F8 |.  8BC8        |MOV ECX, EAX
004681FA |.  8B45 0C     |MOV EAX, DWORD PTR SS:[EBP+C]
004681FD |.  8B55 08     |MOV EDX, DWORD PTR SS:[EBP+8]
00468200 |.  E8 3FFEFFFF |CALL llkzs.00468044
```

`EAX 0013FDD4`

`ECX 00474C2C llkzs.00474C2C`

`EDX 00000007`

`EAX` 存的是 `code1`

注意 `ECX`，存的是 `CALL llkzs.00467F14` 的结果

除了 `copy`，有些 `call` 还起到 `code[0-3]` 和 `code[4-7]` 交换的作用，也不分析了。

直接进入 `CALL llkzs.00468044`

一直运行到

```
00468076 |.  8D4D F6     LEA ECX, DWORD PTR SS:[EBP-A]
00468079 |.  E8 BAFBFFFF CALL llkzs.00467C38
```

又是换位得到 `code2`

下面的循环是将 `code2` 与 `CALL llkzs.00467F14` 的结果异或得到 `code3`

之后是一片移位得到 `code4`（到这里，这种简单的操作读者应该已经可以自己解决了）

### 1.7 循环差异与查表操作

后面这个循环有些不同了

```
0046810E |>  /8BC3       /MOV EAX, EBX
00468110 |.  8A16        |MOV DL, BYTE PTR DS:[ESI]
00468112 |.  E8 31FCFFFF |CALL llkzs.00467D48
00468117 |.  8806        |MOV BYTE PTR DS:[ESI], AL
```

`DL` 带入，返回 `AL`。

进入看看

函数不长，先用 `DL` 计算出一个值 `d = (DL & 0x20) | ((DL & 0x1E) >> 1) | ((DL & 1) << 4)`

然后在 `DS:[EAX*64+4730E4]` 中找到第 `d` 个值赋给 `AL`

注意 `EAX` 是循环次数，也就是说，每次循环都在不同的表中查找，每个表的大小是 `64` `BYTE`

得到 `code5`

### 1.8 Code6, Code7, Code8 生成

之后的循环将 `code5` 的 `8` 个 `BYTE` 合成 `4` 个 `BYTE` 的 `code6`

`code6[i] = (code5[i*2] << 4) + code5[i*2+1]`

之后又是换位

```
0046813C |.  8D45 F6     LEA EAX, DWORD PTR SS:[EBP-A]
0046813F |.  BA 05000000 MOV EDX, 5
00468144 |.  E8 7FFBFFFF CALL llkzs.00467CC8
```

`EAX` 是 `code6`

得到 `code7`

`CALL llkzs.00468044` 结束

在经过一些循环进行 `code[0-3]` 和 `code[4-7]` 的异或和交换后得到 `code8` 循环结束。

### 1.9 最后置换与 Code9

来到最后一个置换

```
00468327 |>  \8B45 0C     MOV EAX, DWORD PTR SS:[EBP+C]
0046832A |.  8B55 08     MOV EDX, DWORD PTR SS:[EBP+8]
0046832D |.  E8 82F8FFFF CALL llkzs.00467BB4
```

最后得到 `code9`，`CALL llkzs.0046816C` 结束。

其他的 `call` 都是无关紧要的了，`DeDe` 中也都有提示。

## 2. DES 算法逆向

至此 `SN` 的计算就算完成了，也可以根据汇编改写成其他语言了，但破解工作刚完成了一半。

如果是用机器码计算注册码，那么把这段汇编改写成注册机就行了，但这个程序显然不是，它是通过注册码计算机器码，我们还需要写出逆过程。

### 2.1 算法识别

第一直觉，这么复杂的加密过程，又换位又查表，还循环 16 次，显然是某种现有的加密算法。尤其是换位，用了这么多换位很可能是 `DES` 或其改进算法，于是我急忙上网上搜了篇 `DES` 算法（这么复杂的东西谁记得住）

`http://www.vckbase.com/document/viewdoc/?id=352`

比对以下，不仅加密过程完全一样，就连换位矩阵都差不多（差别在于，算法中的 `1` 代表数据的 `0` 位）。

### 2.2 对应关系映射

`CALL llkzs.00467F14` 显然就是 `Key` 的生成

1. `CALL llkzs.00402C64` 是 1.1.2 等分密钥
2. `CALL llkzs.00467EB4` 是 1.1.3 密钥移位；`MOV EBX, llkzs.0047334C` 是移位表
3. `CALL llkzs.00467E1C` 是 1.1.4 密钥的选取
4. 最后得 16 组 `Key` 在 `00474C2C`

`CALL llkzs.0046816C` 就是 `DES` 加密的过程

1. `CALL llkzs.00467B30` 是 1.2.2 初始换位
2. `CALL llkzs.00467C38` 是 1.2.3 数据扩展
3. 计算 `code3` 的过程就是与 `Key` 的异或
4. 计算 `code5` 的循环 + 计算 `code6` 的循环就是 1.2.4 数据压缩；`CALL llkzs.00467D48` 中的 8 个 `64` `BYTE` 的表就是表 7.1-7.8
5. `CALL llkzs.00467CC8` 是 1.2.5 数据换位
6. `code8` 就是 1.2.6 交换数据 的结果
7. `CALL llkzs.00467BB4` 是 1.2.8 数据整理

既然确定了这是 `DES` 算法，那就简单了，只要用同样的算法（与密钥异或顺序颠倒）和同样的密钥，对机器码计算结果 `"00508"` 进行解密就能得到注册码了。

将算法式先后，就算大功告成了。

我把算法抠下来写了个 `c` 的，又用这个 `c` 的写了个 `MFC` 的注册机，需要者可直接联系我，因为没法上传附件，请见谅。
