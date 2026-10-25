
# SafeDisc 不完全剖析

> 原文：[`www.pediy.com/kssd/pediy10/89030.html`](https://www.pediy.com/kssd/pediy10/89030.html)

【文章标题】: `SafeDisc` 不完全剖析  

【文章作者】: `leexuany`(小宝)  

【作者邮箱】: `leexuany@sina.com.cn`  

【软件名称】: 风来西林外传 For `Windows`  

【下载地址】: 很久以前买的 D 版光盘  

【保护方式】: `SafeDisc` 2.80.011  

【使用工具】: `OD`, `WinHex`, `VC6.0`  

【作者声明】: 为了促使该游戏的汉化重新启动，也可能什么作用都不起。  

--------------------------------------------------------------------------------

## 详细过程

### 前言

前几天看到 `DS` 版的风来西林汉化已经可以下载了，不禁让我想起胎死腹中的风来西林外传 For `Windows`。汉化者在开始 9 个月后发布消息，说因为加了当时最新的 `SD2.8`，无法破解，最终汉化被迫终止。时隔多年，再无音信……

### 一、准备工作

1. 安装游戏，打好最新的补丁，运行一下，一切正常，退出。

2. `OD` 载入，参考《[Unpack SafeDisc v2.90.40 (更新代码修复)](http://bbs.pediy.com/showthread.php?threadid=15985)》一文运行至 `OEP`，确定 `OD` 没问题。

3. 来了解一下 `SafeDisc` 的运行流程。

`SD` 加密的程序运行时，先将运行所需的动态链接库和调试器释放到临时文件夹，并创建调试进程。然后检测光盘，通过后解密程序代码，不过这些代码是加壳时预处理好的，存在很多问题代码。接下来，如果有加 Logo 就显示，之后与调试器通信，返回 `OEP`。

运行时，由事先安置在代码中的间谍负责与壳代码和调试进程交互，直至程序结束。

### 二、推翻 SafeDisc 的三座大山

#### 1、错误代码的修复

观察运行到 `OEP` 以后的程序代码，`int3`, `ud2`，还有大量的翻译不出来的指令随处可见。如果直接按 `F9`，系统还会向我们报告发生了错误，点击查看便看到错误发生的地址，`Offset: 0012268d`，在 `OD` 里查看一下，`int3` 映入眼帘，正常代码会是这样的吗？这样的代码怎么运行呢？

正常情况下，`SafeDisc` 会创建一个进程来调试程序本体。当程序执行到错误代码时会产生一个异常，执行中断。调试进程捕获这个异常，并根据异常发生的位置进行一系列的计算，求得正确代码，再根据计数器判断是否达到了频繁发生的水平，如果是则使用 `WriteProcessMemory` 还原代码，返回执行，如果不是，模拟代码，执行后返回。

正是这样，`OD` 无法附加正在运行的游戏，或者，由 `OD` 启动的游戏无法通过错误代码。

在这种情况下，像《Unpack SafeDisc v2.90.40 (更新代码修复)》的作者那样直接把正常运行时游戏的代码段 `Dump` 出来是一个不错的选择，但我似乎没有那么幸运，有些代码的计数器没有达到还原的水平，依然有错误代码存在。

我该怎么办？难道要盯着错误代码，然后大喊“还我漂漂拳”？秋香是不会这样就出来的！

正常运行游戏，附加 `~e5d141.tmp` 进程（也就是 `SD` 创建的调试进程），之后在 `WriteProcessMemory` 下断了。好了，去玩游戏吧，万一什么时候触动了还原机制，我们就赚到了。

哈哈，断下了，写完数据，一路返回，沿途做好标记，看看到底是什么关系。

啊，`WaitForDebugEvent`，原来如此，那么在下一条指令下断，一有 `Debug` 事件就断下。

断下后又是漫长的分析，唉，不废话了，直接说算法。

先定义一个运算

```
unsigned int get_v2(unsigned int offset_RVA)
{
    return (0xA0EDEC00 ^ 0x111EE638) * offset_RVA;
}
```

算法为：先对调用地址 `RVA` 和 `get_v2(调用地址 RVA)`共 8 字节计算 `MD5` 值，取第 1 个 `DWORD`，以字节为单位高低逆序为 `key`，在排序二叉树中查找，找到后使用第 2 个 `DWORD` 异或解密数据，解密后使用第 4 个 `DWORD` 校验。

最后编写程序对代码段穷举测试，150 条数据，每条数据 16 字节，例如

```
05 00 02 B8 01 00 00 00 5D 5F 00 00 12 34 56 78
```

- 第 1 字节代表修复的长度，最长不超过 7 字节
- 第 2、3 字节作用不明，有相关判断，但比较分散，暂时不能确定
- 第 4~10 字节修复的数据
- 第 11、12 字节总是为零
- 第 13~16 字节用于校验。

鉴于有两字节数据含义不明，不要将解密出的数据直接写回文件中。

比较明智的做法是先 `Dump` 正常运行时的代码段，然后参考解密数据手工修复剩余错误代码。

如果有高人分析出第 2、3 字节的含义，烦请告诉我一声。

最后要确定搜索用的排序二叉树中的数据是哪里来的。

`Ctrl+E` 双击 `~df394b.tmp` 模块，从代码开始出 `Ctrl+B` 搜索二进制串 `00 EC ED A0`，只有一处，代码如下：

```
100014BC  |>  \68 00ECEDA0   push    A0EDEC00
100014C1  |.   58           pop     eax
100014C2  |.   8945 FC      mov     dword ptr [ebp-4], eax
100014C5  |.   58           pop     eax
100014C6  |.   8B45 FC      mov     eax, dword ptr [ebp-4]
100014C9  |.   24 00       and     al, 0
100014CB  |.   0305 F8C10510 add   eax, dword ptr [1005C1F8] // 注意这里
100014D1  |.   C9           leave
100014D2  \\.   C3           retn
```

其中`[1005C1F8]`里面也是一个解密用的参数，不过似乎没用到，这暂且不管，我们要关心的是这个地址，因为这个地址`+4=1005C1FC` 就是我们要找的数据。

从 `1005C1FC` 开始 3000 字节就是我们要的解密信息，一共 150 条，每条 20 字节。

咦？不是 16 字节吗，怎么多了 4 字节？

别忘了我们在排序二叉树中搜索用的那个 `key` 不还 4 字节呢吗，不然比较什么啊。

可能有的朋友会问，你怎么知道就 150 条，怎么不是 180 条呢。

我写了个程序把 `SD` 用的二叉树遍历了一遍统计出来的，你也可以试试。不过要注意我们写二叉树叶子节点左右指针一般用 `NULL`，而 `SD` 叶子节点的左右指针指向的是一个固定的节点，这个节点左右指针是 `NULL`。

#### 2、IAT 争霸 混乱之治

同大多数加密壳一样，`SD` 也加密了 `IAT`，但这不算什么，最厉害的还是 `SD` 混乱调用的函数。虽然混乱，但不是胡乱，在计算机里一切都是有章法可循的。

随便找一个函数来看一下。

```
0051A282   .   6A 60        push    60
0051A284   .   68 30495300  push    00534930
0051A289   .   E8 DE160000 call    0051B96C
0051A28E   .   BF 94000000 mov     edi, 94
0051A293   .   8BC7        mov     eax, edi
0051A295   .   E8 36F6FFFF call    005198D0
0051A29A   .   8965 E8     mov     dword ptr [ebp-18], esp
0051A29D   .   8BF4        mov     esi, esp
0051A29F   .   893E        mov     dword ptr [esi], edi
0051A2A1   .   56          push    esi
0051A2A2   .   FF15 90805200 call  dword ptr [528090] // F7 进入
0051A2A8   .   8B4E 10     mov     ecx, dword ptr [esi+10]
```

到这里

```
01AA5043    68 F812EABF    push    BFEA12F8
01AA5048    9C             pushfd
01AA5049    60             pushad
01AA504A    54             push    esp
01AA504B    68 8350AA01    push    1AA5083     // 关键参数 1
01AA5050    E8 40935A0E    call    ~df394b.1004E395
01AA5055    83C4 08       add     esp, 8
01AA5058    6A 00         push    0
01AA505A    58             pop     eax
01AA505B    61             popad
01AA505C    9D             popfd
01AA505D    C3             retn
```

把数据窗口调到 `1AA5083`（关键参数 1）看一下

```
01AA5083|07 00 00 00|0C 00 00 00|00 00 00 00|00 00 00 00
```

可以很明显地看到两个数值，第 1 个 `DWROD` 是 `00000007`，我把它叫做目标模块编号，第 2 个 `DWORD` 是 `0000000C`，我把它叫做默认函数编号。

那这两个参数是什么意思呢？这些参数的意思就是告诉壳代码我要调用 `0x00000007` 号模块的一个函数，这个函数的默认编号为 `0x0000000C`. 注意，默认编号不一定就是最终调用的函数编号，符合某些条件时，还要重新计算。

下面就来看看函数调用的过程。

首先判断来自这个地址的 `CALL` 是不是已经调用过了。

```
1004D6A3  /$   55           push    ebp
1004D6A4  |.   8BEC        mov     ebp, esp
1004D6A6  |.   83EC 0C     sub     esp, 0C
1004D6A9  |.   8B45 10     mov     eax, dword ptr [ebp+10] // 参数 3，调用地址 004F2D4F
1004D6AC  |.   33D2        xor     edx, edx
1004D6AE  |.   6A 2F       push    2F
1004D6B0  |.   59          pop     ecx
1004D6B1  |.   F7F1        div     ecx       // 除以$2F
1004D6B3  |.   8955 F8     mov     dword ptr [ebp-8], edx // 保留余数，var_1 = offset % 0x2F
1004D6B6  |.   8B45 0C     mov     eax, dword ptr [ebp+C] // 参数 2，默认函数编号
1004D6B9  |.   69C0 C3040000 imul  eax, eax, 4C3
1004D6BF  |.   8B4D 08     mov     ecx, dword ptr [ebp+8] // 参数 1，0x015F1218，一个 IAT 相关的指针（每个模块单独一个），指向的结构用来存储很多信息，比如 N 号函数默认的 IAT 表项地址，某地址是否调用过等等
1004D6C2  |.   03C8        add     ecx, eax
1004D6C4  |.   8B45 F8     mov     eax, dword ptr [ebp-8]
1004D6C7  |.   6BC0 18     imul    eax, eax, 18
1004D6CA  |.   8D4401 03   lea     eax, dword ptr [ecx+eax+3] // 目的地址 1 = (offset % 0x2F) * 0x18 + 参数 1 +  默认函数编号 * 0x4C3
1004D6CE  |.   8945 FC     mov     dword ptr [ebp-4], eax
1004D6D1  |.   8365 F4 00  and     dword ptr [ebp-C], 0 // 清空计数器
1004D6D5  |.   EB 07       jmp     short 1004D6DE
1004D6D7  |>   8B45 F4     /mov     eax, dword ptr [ebp-C]
1004D6DA  |.   40          |inc     eax
1004D6DB  |.   8945 F4     |mov     dword ptr [ebp-C], eax
1004D6DE  |>   837D F4 03  cmp     dword ptr [ebp-C], 3 // 计数器比较
1004D6E2  |.   73 1C       |jnb     short 1004D700       // for (i=0; i<3; i++)
1004D6E4  |.   8B45 F4     |mov     eax, dword ptr [ebp-C]
1004D6E7  |.   8B4D FC     |mov     ecx, dword ptr [ebp-4] // 取出目的地址 1
1004D6EA  |.   8B04C1      |mov     eax, dword ptr [ecx+eax*8]
1004D6ED  |.   3B45 10     |cmp     eax, dword ptr [ebp+10] // 同调用地址比较
1004D6F0  |.   75 0C       |jnz     short 1004D6FE
1004D6F2  |.   8B45 F4     |mov     eax, dword ptr [ebp-C] // 如果相同，1004D6F0 不跳，到这里
1004D6F5  |.   8B4D FC     |mov     ecx, dword ptr [ebp-4]
1004D6F8  |.   8B44C1 04   |mov     eax, dword ptr [ecx+eax*8+4] // 取出目标函数的地址
1004D6FC  |.   EB 04       |jmp     short 1004D702
1004D6FE  |>^  EB D7       \jmp     short 1004D6D7
1004D700  |>   33C0        xor     eax, eax
1004D702  |>   C9          leave
1004D703  \\.   C3          retn
```

函数最后用到一个简单的结构体

```
struct func_map {
    unsigned int offset;       // 调用此函数地址
    unsigned int func_address; // 目标函数地址
};
```

首先判断是否是不是调用过了，如果是就直接根据保留下的信息进行调用。

先暂停一下，在继续前大家一起想想上面这个缓存下的信息对我们破解有什么用处。

恩，对啦，这些信息可以帮助我们快速的修复大量错误的函数调用。我们先打开游戏使劲玩上一会儿，把所有能做的操作都做一遍，然后写个程序把上面这些信息提取出来。就是这样，实际证明这个方法可以修复 70%左右的函数调用，极大的减轻了我们手动修复的负担。而且只用这 70%的函数，我们就可以让程序正确的运行起来，方便我们快速测试。

好了，继续，当壳代码发现这是一个新的函数调用时，来到如下代码：



### 1. 代码分析与校验逻辑

```
  1004E602   .  FF75 F0       push    dword ptr [ebp-10] // 从代码段开始计算的偏移量 0x004F2D4F - 0x00401000 = 0xF1D4F
  1004E605   .  E8 14070000   call    1004ED1E // 一个校验函数，用来初步判断这个地址是使用默认函数编号还是要重新计算。
  1004E60A   .  59            pop     ecx
  1004E60B   .  0FB7C0        movzx   eax, ax
  1004E60E   .  83F8 01       cmp     eax, 1 // eax=1 重新计算，其他时候使用默认函数编号
  1004E611   .  0F85 8E000000 jnz     1004E6A5
  1004E617   .  8B45 E4       mov     eax, dword ptr [ebp-1C]
  1004E61A   .  69C0 C3040000 imul    eax, eax, 4C3
  1004E620   .  8B4D FC       mov     ecx, dword ptr [ebp-4]
  1004E623   .  8B55 DC       mov     edx, dword ptr [ebp-24] 还是 0x015F1218，那个 IAT 相关的指针
  1004E626   .  8B49 02       mov     ecx, dword ptr [ecx+2]
  1004E629   .  3B8C02 AA0400>cmp     ecx, dword ptr [edx+eax+4AA] // 判断 1
  1004E630   .  75 73         jnz     short 1004E6A5
  1004E632   .  8B45 FC       mov     eax, dword ptr [ebp-4]
  1004E635   .  0FB600        movzx   eax, byte ptr [eax]
  1004E638   .  3D FF000000   cmp     eax, 0FF                // 判断 2
  1004E63D   .  75 66         jnz     short 1004E6A5
  1004E63F   .  8B45 FC       mov     eax, dword ptr [ebp-4]
  1004E642   .  0FB640 01     movzx   eax, byte ptr [eax+1]
  1004E646   .  83F8 15       cmp     eax, 15                  // 判断 3，这 3 处就是判断是不是 call [00528XXX]的形式
  1004E649   .  75 5A         jnz     short 1004E6A5           // 不是就跳走，使用默认函数编号，否则就重新计算编号
  1004E64B   .  8B45 E4       mov     eax, dword ptr [ebp-1C]
```

看过这段代码之后，大家有在想什么呢？

我想到了几点，说出来，大家看看有没有落下什么。

1. `call 1004ED1E` 一个校验函数，很重要，但是不难，简单模拟下，嵌入汇编或者 `LoadLibrary` 都可以。
2. 调用的时候要判断是不是 `call [00528XXX]`的形式，这一点用处很多。
   例如：`mov edi, dword ptr [00528010]`
   `call edi`
   我们不用劳神于类似这样的变形，对于那些经 `SD` 处理之后生成的代码也不担心，它们肯定都是调用默认编号指定的函数。
3. 注意判断 1 处，`[IAT 相关的指针 + 默认函数编号 * 0x4C3 + 0x4AA]`就是默认编号对应的 `IAT` 表项地址 `00528090`，换句话说就是我们可以推算出所有编号与 `IAT` 表项地址的对应关系，再确定编号与函数地址的关系就可以得到完美的 `IAT` 表了。

从 `$1004E64B~$1004E6A5` 之间就是重新计算编号的代码，只是简单的计算，这里就略过了，下面着重分析用模块编号和函数编号计算目标函数地址的代码。

在计算函数名和函数名长度时，`SD` 使用内部函数编号，用目标函数编号求内部函数编号参考后面的公式。

```
  1004E1D5  |.  8B45 08       mov     eax, dword ptr [ebp+8] // 取出模块编号，暂时用□标记
  1004E1D8  |.  69C0 8D000000 imul    eax, eax, 8D
  1004E1DE  |.  8B0D B8CD0610 mov     ecx, dword ptr [1006CDB8] // 这是一个固定指针，暂时用△标记
  1004E1E4  |.  8B55 0C       mov     edx, dword ptr [ebp+C] // 取出内部函数编号
  1004E1E7  |.  3B5401 58     cmp     edx, dword ptr [ecx+eax+58] // [△+□*0x8D+0x58]=模块内函数个数，判断是否超出函数总数
  1004E1EB  |.  0F83 E1000000 jnb     1004E2D2
  1004E1F1  |.  8B45 08       mov     eax, dword ptr [ebp+8]
  1004E1F4  |.  69C0 8D000000 imul    eax, eax, 8D
  1004E1FA  |.  8B0D B8CD0610 mov     ecx, dword ptr [1006CDB8]
  1004E200  |.  8B8401 B50000>mov     eax, dword ptr [ecx+eax+B5] // [△+□*0x8D+0xB5]=函数名字符串（加密的）地址列表首地址
  1004E207  |.  8B4D 0C       mov     ecx, dword ptr [ebp+C]
  1004E20A  |.  8B0488        mov     eax, dword ptr [eax+ecx*4] // 取出函数名字符串地址
  1004E20D  |.  8945 F4       mov     dword ptr [ebp-C], eax
  。。。。。。
  1004E244  |>  \8B45 08      mov     eax, dword ptr [ebp+8]
  1004E247  |.  69C0 8D000000 imul    eax, eax, 8D
  1004E24D  |.  8B0D B8CD0610 mov     ecx, dword ptr [1006CDB8]
  1004E253  |.  8B8401 BF0000>mov     eax, dword ptr [ecx+eax+BF] // [△+□*0x8D+0xBF]=函数名长度地址列表首地址
  1004E25A  |.  8B4D 0C       mov     ecx, dword ptr [ebp+C]
  1004E25D  |.  8B0488        mov     eax, dword ptr [eax+ecx*4]
  1004E260  |.  8B00          mov     eax, dword ptr [eax] // 取出函数名长度
  1004E262  |.  8945 E8       mov     dword ptr [ebp-18], eax
  1004E265  |.  A1 B8CD0610   mov     eax, dword ptr [1006CDB8]
  1004E26A  |.  83C0 26       add     eax, 26 // △+0x26 指向解密用的数据
  1004E26D  |.  50            push    eax // 解密数据指针 unsigned char * arg1
  1004E26E  |.  FF75 E8       push    dword ptr [ebp-18] // 函数名长度 int func_name_length
  1004E271  |.  FF75 F4       push    dword ptr [ebp-C] // 函数名（加密的）指针 char *func_name
  1004E274  |.  E8 437CFDFF   call    10025EBC // 解密用的函数，有点难，我翻译不出 C 代码，直接 LoadLibrary 调用
  1004E279  |.  83C4 0C       add     esp, 0C
```

至此我们已经看到了函数调用的曙光，`$1004E274 call 10025EBC` 返回之后，函数名就摆在我们面前了。

这里总结一下函数调用中常用的几个地址。

设 `△ = [1006CDB8]`
`□ = 模块编号`
`N = 目标函数编号`（不一定是默认函数编号！）
`○ = 内部函数编号`

那么：

- `[△ + 0xF] = 模块总个数`
- `[△ + 0x8D * □ + 0x58] = □模块内函数的总个数`
- `[△ + 0x8D * □ + 0xB5] = □模块中函数名字符串指针列表首地址`
- `[△ + 0x8D * □ + 0xBF] = □模块中函数名长度指针列表首地址`
- `[△ + 0x8D * □ + 0x52] = □模块的句柄，就是 GetProcAddress 的第一个参数`
- `[△ + 0x8D * □ + 0xC3] = □模块中那个与 IAT 相关的地址，暂用☆标记`
- `[☆ + 0x4C3 * N + 0x4AA] = □模块中 N 号函数对应的 IAT 表项地址，N 是未经过变换的。`
- `[△ + 0x8D * □ + 0x4C] = □模块中用于把目标编号转换成内部编号的表的首地址，暂用★标记`
- `○ = [★ + 4 * N]` // 查表求得内部标号，计算函数名地址和函数名长度时使用内部编号○

最后编写 2 个程序，一个获得 `IAT` 表，一个穷举代码段中的 `call [00528XXX]`并输出正确的地址。

注：《Unpack SafeDisc v2.90.40 (更新代码修复)》一文中通过修改代码强制写入原始 `IAT` 的方法与我写程序得到的 `IAT` 相同，那个方法不用自己写程序，推荐熟悉一下。

还有一部分函数调用伪装成了 `jmp XXXXXXXX` 的形式，这里提供一个快速修复的方法。先来看代码：

```
  地址       大小       属主    区段
  006E9000   00003000   AsfPc   stxt774
  006EC000   00004000   AsfPc   stxt371

  004012C9   .  /74 25        je      short 004012F0
  004012CB   .  |6A 64        push    64
  004012CD   . -|E9 FA862E00  jmp     006E99CC       // 跳到壳区段中去了，伪装的函数调用，跟进去
  004012D2   .  |8647 83       xchg    byte ptr [edi-7D], al
  004012D5   .  |FF08          dec     dword ptr [eax]
  004012D7   . .^|72 AE        jb      short 00401287

  006E99CC    53              push    ebx
  006E99CD    EB 06           jmp     short 006E99D5
  006E99D5    E8 CCFEFFFF     call    006E98A6
  006E99DA    B2 43           mov     dl, 43 // 记下这个地址 1

  006E98A6    870424          xchg    dword ptr [esp], eax
  006E98A9    9C              pushfd
  006E98AA    05 320F0000     add     eax, 0F32 // eax=地址 1+0x0F32
  006E98AF    8B18            mov     ebx, dword ptr [eax]
  006E98B1    6BDB 13         imul    ebx, ebx, 13
  006E98B4  ^  EB C7           jmp     short 006E987D
```

看下数据：

```
  006EA90C|C3 04 00 00|BF 04 62 01|.....
```

```
  006E987D    0358 04         add     ebx, dword ptr [eax+4] // ebx=0x4C3 * 0x13 + 0x016204BF=0x01625F38
  006E9880    9D              popfd
  006E9881    58              pop     eax
  006E9882    871C24          xchg    dword ptr [esp], ebx // 上面那个地址主要是用来计算调用的地址
  006E9885    C3              retn

  01625F38    68 D3124000     push    4012D3
  01625F3D    68 3F12EABF     push    BFEA123F
  01625F42    9C              pushfd
  01625F43    60              pushad
  01625F44    54              push    esp
  01625F45    68 785F6201     push    1625F78 // 注意这个参数，修复关键全在这里
  01625F4A    E8 4684A20E     call    ~df394b.1004E395 // 原来还是这个函数
  01625F4F    83C4 08         add     esp, 8
  01625F52    6A 00           push    0
  01625F54    58              pop     eax
  01625F55    61              popad
  01625F56    9D              popfd
  01625F57    C3              retn
```

再看下数据：

```
  01625F78|FF FF FF FF|FF FF FF FF|58 20 58 00|...
```

回忆一下正常调用时的参数是什么样的：

```
  01AA5083|07 00 00 00|0C 00 00 00|00 00 00 00|00 00 00 00
```

看出区别了吗？对了，`$00582058` 就是正确的函数调用。

实际上进入函数 `1004E395` 后，程序会根据模块数、模块内函数数量以及`[☆ + 0x4C3 * N + 0x4AA]`这三个数据查询 `00582058` 这个地址，找到后再把模块编号、默认函数编号写入 `01625F78` 处，最后按正常调用运算。

再根据 `jmp xxxxxxxx` 不是 `call [xxxxxxxx]`的形式，可推出 `00582058` 就是正确的函数调用。

好了，`SD` 的混乱统治终结了，一个新的时代到来了。

## 3. 伪装成 CALL 指令的跳转

`Dump` 数据，修复了 `IAT`，所有的函数调用，和大部分错误代码，应该可以运行了吧，`F9` 不久，程序又停了下来，内存访问异常。

跟踪下代码，确定错误的范围，如下：

```
  00402777   .  52            push    edx
  00402778   .  51            push    ecx
  00402779   .  FF15 D4005000 call    dword ptr [5000D4]
  0040277F   .  5F            pop     edi
  00402780   >  8B8C24 080400>mov     ecx, dword ptr [esp+408]
```



```
00402787   .  338C24 0C0400>xor     ecx, dword ptr [esp+40C]

0040278E   .  5E             pop     esi

0040278F   .  81C4 08040000 add     esp, 408

00402795   .  E8 FF010000    call    00402999 // 出错的函数

0040279A     90             nop

0040279B     90             nop

0040279C     90             nop

00402999 /$  51             push    ecx

0040299A |.  50             push    eax

0040299B |.  E8 D3F1FFFF    call    00401B73

004029A0 |$  8B4424 0C      mov     eax, dword ptr [esp+C]

00401B73   $  B8 7B310000    mov     eax, 317B

00401B78   .  59             pop     ecx

00401B79   .  8D0408         lea     eax, dword ptr [eax+ecx] // 还是计算一个关键地址 0x405B1B

00401B7C   .  8B00           mov     eax, dword ptr [eax] // 取出 0x10011387

00401B7E   .  FFE0          jmp     eax // jump 过去

10011387   .  58             pop     eax

10011388   .  59             pop     ecx

10011389   .  68 00004000    push    400000 // 算是一个小特征吧

1001138E   .  9C             pushfd

1001138F   .  60             pushad

10011390   .  54             push    esp

10011391   .  E8 D2FFFFFF    call    10011368

10011396   .  5C             pop     esp

10011397   .  61             popad

10011398   .  9D             popfd

10011399   .  C3             retn
```

跳来跳去，终于到了关键代码

```
100112BF |.  50             push    eax

100112C0 |.  FF75 08        push    dword ptr [ebp+8]

100112C3 |.  E8 E5FeFFFF    call    100111AD // 初始化 MD5 参数

100112C8 |.  8B85 E8FDFFFF mov     eax, dword ptr [ebp-218]

100112CE |.  B9 38CE0610    mov     ecx, 1006CE38

100112D3 |.  8BF8           mov     edi, eax

100112D5 |.  2B43 04        sub     eax, dword ptr [ebx+4]

100112D8 |.  50             push    eax // 调用返回地址的 RVA（0040279A-00400000=279A）

100112D9 |.  E8 DCED0300    call    100500BA // 计算 MD5

100112DE |.  50             push    eax // eax 是 MD5 的前 4 字节，就是关键的 key

100112DF |.  E8 F1010000    call    100114D5

100112E4 |.  8BC8           mov     ecx, eax

100112E6     E8 FC010000    call    100114E7 // 在一个 128 项的哈希表内查找 key，算法有兴趣的自己看下，没有冲突时，顺序查找也是一样。

100112EB |.  8BF0           mov     esi, eax

100112ED |.  85F6           test    esi, esi

100112EF |.  74 3F          je      short 10011330

100112F1 |.  66:837B 08 01 cmp      word ptr [ebx+8], 1

100112F6 |.  75 3D          jnz     short 10011335

100112F8 |.  8D85 30FDFFFF lea     eax, dword ptr [ebp-2D0]

100112FE |.  8BCE           mov     ecx, esi

10011300 |.  50             push    eax

10011301 |.  E8 E4580100    call    10026BEA  // 这里有模拟执行的代码，

10011306 |.  8BCB           mov     ecx, ebx

10011308 |.  E8 8AFEFFFF    call    10011197

1001130D |.  83F8 04       cmp     eax, 4    // 比较 1

10011310 |.  72 14          jb      short 10011326

10011312 |.  8BCE           mov     ecx, esi

10011314 |.  E8 B7570100    call    10026AD0

10011319 |.  83F8 04       cmp     eax, 4    // 比较 2（1 和 2 有一个调用的次数，另一个不清楚）

1001131C |.  72 08          jb      short 10011326

1001131E |.  57             push    edi       // 2 次比较都不跳，会到这里

1001131F |.  8BCE           mov     ecx, esi

10011321 |.  E8 FB570100    call    10026B21  // 内部有还原代码，如果前面跳走了，就不还原了
```

总结一下解密算法是先对调用返回地址的 `RVA`（4 Bytes）求 `MD5`，以结果的前 4 字节为 `Key`，在存放还原信息的哈希表中查找。

找到之后模拟运行，再根据调用计数器判断是否还原代码，返回。

快速确定还原数据的方法，`SafeDisc2.8` 中在还原信息前（`$1006CEB8`）有很大一片 `0x0001`，之后（`$1006CFB8`）紧跟的就是还原信息。

还原信息的结构为

```c
struct sd_info {
    unsigned int size;   // 使用前跟 0x9877D4A7 异或
    unsigned int id;     // 使用前跟 0x6F68D2EB 异或
    unsigned int code2;  // 使用前跟 0xE2536C94 异或
    unsigned int code1;  // 使用前跟 0x07AEECEA 异或
    unsigned int zero[8];// 占位用，全是零
};
```

最后根据 `size` 的大小，将 `code1` 和 `code2` 拼接成原始数据，例如：

原始数据 `size = 6`; `code1 = 000F8400`; `code2 = 00000175`

```
拼接前   00 00 00 00 00 00  一块空内存
code1    0F 84 00
code2          75 01 00 00
拼接后   0F 84 75 01 00 00
```

最后同样写程序对代码段穷举一下，结果很快就出来了，128 个刚刚好。

### 三、后记

写到最后反而没什么好写，那么删除 `stxt774` 和 `stxt371` 两个区段，重构一下文件吧。

14 天说多不多，说少不少。我与 `safedisc` 的第一次邂逅就这样结束了。

风来西林外传的汉化版还要等多久呢？

--------------------------------------------------------------------------------

2009 年 05 月 16 日 22:43:15
