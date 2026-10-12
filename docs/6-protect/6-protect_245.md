
# LabView 7.1 FlexLM8.4a 破解

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-88.htm`](https://www.pediy.com/kssd/pediy09/pediy09-88.htm)

> 【软件名称】LabView 7.1

> 【应用平台】Win2000 SP4

> 【作者邮箱】chubing6143@sina.com

> 【使用工具】`OllyDbg1.10`，`FlexLM 7.2 SDK`

> 【软件限制】`FlexLM8.4a`

> 【软件简介】NI 的产品，做测试和虚拟仪器的大家可能都知道了。该软件其实在安装时提供的有不限制的 `Licence` 文件，但是作为练手，我将其删除掉了（这样我不用担心任何后果了）。

## 1. 确定 FlexLm 版本号 BEHAVIOR Version

- 用二进制编辑器,推荐使用 `HexWorkshop` 打开 `LabView.exe`,它的查找功能很强,特别是 `Find All Instances` 更是我最最常用的.

- 在查找对话框 `Type` 中选择类型 `Text String`,`Value` 中输入`"@(#) FLEXlm v"`,查找,`"@(#) FLEXlm v"`后面的就是版本号.

> 【以上是 `tulipfan[CCG]`大虾说的】

- `@(#) FLEXlm v8.4a (lmgr.lib)`可以知道 `FlexLm` 版本号 `BEHAVIOR Version` 是 `8.4a`

## 2. 找 FEATURE

- **目的**：确定 `FEATURE` 和产品版本号（不是 `FLEXLM` 的版本号）

- **方法**：在字符串参考中查找`'lm_ckout.c'`；总共也就出现几次，比较容易确定。

据 `newsearch` 讲“一般是挨着的两个才是；另外有四个挨着的不是。”

根据这一原则，在 `OD` 中 `BP C9EBFC`,然后 `F9` 运行，

程序被中断下来，此时察看堆栈情况如下所示：

```
0012E3E8    01B34178  |Arg1 = 01B34178
0012E3EC    00EE25E4  |Arg2 = 00EE25E4  ASCII "lm_ckout.c"
0012E3F0    00000093  \\Arg3 = 00000093
0012E3F4    00C99EF4  LabVIEW.00C99EF4
0012E3F8    01B34178
0012E3FC   /0012E46C
0012E400   |00C81BAD  返回到 LabVIEW.00C81BAD 来自 LabVIEW.00C9EBD0
0012E404   |01B34178
0012E408   |018BF908  ASCII "LabVIEW_LabVIEW"
0012E40C   |018BEF28  ASCII "7.0100"
```

可以确定

- `FEATURE` 版本 `v7.0100`
- `FEATURE` ID，`LABVIEW _LABVIEW`

## 3. 找 vendor 以及计算 seed 的关键数据

在反汇编以后，搜索“`87654321`”，在前面设断点

```
00C9393C  .  8D8D 80FDFFFF LEA ECX,DWORD PTR SS:[EBP-280]
00C93942  .  51           PUSH ECX                     ; /Arg3
00C93943  .  8B95 6CFDFFFF MOV EDX,DWORD PTR SS:[EBP-294]          ; ebp-294 地址是以后要用的！
00C93949  .  81C2 0C030000 ADD EDX,30C                  ; 计算得到 VENDOR 地址
00C9394F  .  52           PUSH EDX                     ; VENDOR 入栈！(ASCII "nilm")
00C93950  .  8B85 6CFDFFFF MOV EAX,DWORD PTR SS:[EBP-294]          ; |
00C93956  .  50           PUSH EAX                     ; |Arg1
00C93957  .  E8 07E80000  CALL LabVIEW.00CA2163            ; 这个 call 就是著名的所谓 7648B98E 标志 call！后面要进去的
00C9395C  .  83C4 0C       ADD ESP,0C
00C9395F  .  81BD 84FDFFFF>CMP DWORD PTR SS:[EBP-27C],87654321
00C93969  .  74 0C         JE SHORT LabVIEW.00C93977
00C9396B  .  81BD 88FDFFFF>CMP DWORD PTR SS:[EBP-278],12345678
```

我们在 `00C9393C` 停下 `d EBP-280`

```
0012E244  00000004   ...
0012E248  7E58C8CF  先 X~
0012E24C  9B7AB0C4  陌 z
0012E250  D7B636AA  ?蹲
0012E254  C9A6F893  ?ι
0012E258  B1CDE046  F 嗤
0012E25C  A5254D00  .M%
0012E260  00040008   . .
0012E264  38300061  a.08
0012E268  0000332E  .3..
```

我们得到 `BEHAVIOR Version  V8.3a`

`VENDOR` 为`"nilm"`

根据需要我们做一个假的 `LabVIEW_LabVIEW_070100.lic`，放在 `C:\Program Files\National Instruments\Shared\License Manager\Licenses` 目录里下:

```
FEATURE LabVIEW_PKG nilm 7.0100 permanent uncounted VENDOR_STRING="Cracked \
 by laowanghai" HOSTID=ANY SIGN=123456789012
```

然后重新加载程序，根据前面寻找 `Vendor` 的过程可以知道 `CALL LabVIEW.00CA2163` 这个 `call` 就是著名的所谓 `7648B98E` 标志 `call`！因此 `BP 00CA2163`，`F9` 运行

```
00CA2163  /$  55           PUSH EBP
00CA2164  .  8BEC         MOV EBP,ESP
00CA2166  .  83EC 24      SUB ESP,24
00CA2169  .  C745 F4 B8307>MOV DWORD PTR SS:[EBP-C],6F7330B8                 ; ====就是这个数在确定 l_sg 函数上很有用
00CA2170  .  C745 F0 03000>MOV DWORD PTR SS:[EBP-10],3
00CA2177  .  8B45 08      MOV EAX,DWORD PTR SS:[EBP+8]
00CA217A  .  8B48 6C      MOV ECX,DWORD PTR DS:[EAX+6C]
00CA217D  .  8B91 A8030000 MOV EDX,DWORD PTR DS:[ECX+3A8]
00CA2183  .  81E2 00800000 AND EDX,8000
00CA2189  .  85D2         TEST EDX,EDX
00CA218B  .  74 23        JE SHORT LabVIEW.00CA21B0                         ; ====这个跳第一次肯定会跳的！第二次再来！
00CA218D  .  833D 70EBF200>CMP DWORD PTR DS:[F2EB70],0
00CA2194  .  74 1A        JE SHORT LabVIEW.00CA21B0
00CA2196  .  8B45 10      MOV EAX,DWORD PTR SS:[EBP+10]
00CA2199  .  50           PUSH EAX
00CA219A  .  8B4D 0C      MOV ECX,DWORD PTR SS:[EBP+C]
00CA219D  .  51           PUSH ECX
00CA219E  .  8B55 08      MOV EDX,DWORD PTR SS:[EBP+8]
00CA21A1  .  52           PUSH EDX
00CA21A2  .  FF15 70EBF200 CALL DWORD PTR DS:[F2EB70]                       ; ====在这个地方下断，F8 之后就是我们要的！
00CA21A8  .  83C4 0C      ADD ESP,0C
00CA21AB  .  E9 13010000  JMP LabVIEW.00CA22C3
```

`F9` 执行，中断在 `00CA2163`。

这时第一次来到 `00CA218B` 时它是要跳过去的，注意要跳就让他跳吧。

若开始第一次时强制不跳，倒是能来到 `00CA21A2`，结果跟出了错误的 `seed1` 和 `seed2`，而且多少次结果还都一样。

再次 `F9`，我们又回来中断在 `00CA2163`，`F8` 单步执行，这次不跳了，直到过了 `00CA21A2`，暂停！

我们在命令栏下如下命令，可以看到：（可以纪录了多次的调试结果）

输入`d [esp+4]` 在内存窗口看到，【长型——ASCII 转存】

```
01B24484  6D6C696E  nilm
```

输入`d [esp+8]` 在内存窗口看到，【长型——ASCII 转存】

```
0012DCD0  00000004   ...
0012DCD4  8216F92E  .?   data[0]
0012DCD8  67348125  %?g  data[1]
0012DCDC  B266513C  <Qf
0012DCE0  C11AF607  ?
0012DCE4  B918EEBE  绢
0012DCE8  C0F52DF8  ?趵
0012DCEC  00040008   . .
0012DCF0  38300061  a.08
0012DCF4  0000332E  .3..
```

输入`d [esp]` 【长型——ASCII 转存】

```
01B24178  00000066  f...
01B2417C  002100E7  ?!.
01B24180  3C0DCF4B  K?<   job+08
01B24184  00B12ACE  ??    job+0c
01B24188  00D40000  ..?   job+10
```

这些数据就是我们计算 `seed1` 和 `seed2` 的关键！

## 4. 计算 ENCRYPTION_SEED

使用工具 `calcseed.exe`,输入 `data[0]`，`data[1]`，`job+08`，`job+0c`，`job+10`，`VENDORname` 为`"nilm"`上述信息,计算

```
ENCRYPTION_SEED1 0xa81b372e
ENCRYPTION_SEED2 0x4d394f25
```

至此,分析已经基本完成,可以使用 `SDK` 写 `license` 文件了。

## 5. 制作 license.dat

我们得到数据如下：

- `FEATURE` 版本 `v7.01`
- `FEATURE` id，`LABVIEW_LABVIEW`
- `vendor: nilm`
- `ENCRYPTION_SEED1 0xa81b372e`
- `ENCRYPTION_SEED2 0x4d394f25`
- `BEHAVIOR Version  V8.3a`

使用 `lmkg.exe`,选择 `version 7.0`,在 `vendor_id` 输入`"nilm"`得到:

```
/* Version 7 keys */
#define VENDOR_KEY1 0x6b55b25a
#define VENDOR_KEY2 0xe74d99f2
#define VENDOR_KEY3 0xaa7e7ee6
#define VENDOR_KEY4 0x93a6eec9
#define VENDOR_KEY5 0x1821d5fe
#define CRO_KEY1 0x6f75b052
#define CRO_KEY2 0xa16ffb74

#define VENDOR_NAME "nilm"
```

然后编辑 `lm_code.h` 如下所示：

```c
#ifndef LM_CODE_H
#define LM_CODE_H
#include "lm_cro.h"
#define LM_STRENGTH LM_STRENGTH_DEFAULT
#define ENCRYPTION_SEED1 0xa81b372e//计算出来的
#define ENCRYPTION_SEED2 0x4d394f25//计算出来的
#define ENCRYPTION_SEED3 0x11223344
#define ENCRYPTION_SEED4 0x55667788
#define VENDOR_KEY1 0x6b55b25a//计算出来的
#define VENDOR_KEY2 0xe74d99f2//计算出来的
#define VENDOR_KEY3 0xaa7e7ee6//计算出来的
#define VENDOR_KEY4 0x93a6eec9//计算出来的
#define VENDOR_KEY5 0x1821d5fe//计算出来的
#define CRO_KEY1 0x6f75b052//计算出来的
#define CRO_KEY2 0xa16ffb74//计算出来的
#define VENDOR_NAME "nilm"//找到的哦
#define LM_VER_BEHAVIOR LM_BEHAVIOR_V7_1
#endif /* LM_CODE_H */
```

打开`\flexlm\v7.2\i86_n3\genlic.exe` 开始我们的制作吧

1. `basic` 窗口内填好 `Feathure “LABVIEW_LABVIEW”`，选择 `permanent` 打勾，选择 `run anywhere`
2. 进入 `advance` 窗口，`version` 内填 `FEATURE` 的 `Version` 就是 `7.010`，`Vendor info` 填上 `Cracked by laowanghai`
3. 其余不改了，回到 `basic` 窗口，点击 `make license`，然后 `save` 你的 `license` 文件

大功告成！

生成的如下:

```
FEATURE LABVIEW_LABVIEW nilm 7.010 permanent uncounted \
  VENDOR_STRING="cracked by laowanghai" HOSTID=ANY \
  SIGN=80A999623E02
```

## 6. 明码比较（laoqian 文章中的方法）

通过搜索“`66D8B337`”可以定位到下面的代码：

```
00CA5A71   .  A2 290DF300          mov byte ptr ds:[F30D29],al
00CA5A76   .  C605 2F0DF300 00     mov byte ptr ds:[F30D2F],0
00CA5A7D   .  8A15 2F0DF300        mov dl,byte ptr ds:[F30D2F]
00CA5A83   .  8815 2E0DF300        mov byte ptr ds:[F30D2E],dl
```



```
> 00CA5A89   |>  C785 74FEFFFF 08000000  mov dword ptr ss:[ebp-18C],8

> 00CA5A93   |. 817D 18 37B3D866       cmp dword ptr ss:[ebp+18],66D8B337

> 00CA5A9A   |. 75 0F                  jnz short LabVIEW.00CA5AAB
```

在`CA5A9A`处下断点，运行到此处时，`d 0xf30d29`

可以看到：

```
> 00F30D25  80000000  ...€

> 00F30D29  3E6299A9  

> 00F30D2D  A2000002   .
```

因此正确的`SIGN`就是`80A999623E02`。

`4dd466d0e1e0cc123afbe504da2462ca`
