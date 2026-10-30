

# [原创] CVE-2014-6332 学习笔记

## 0x00 漏洞简介

CVE-2014-6332 是微软于 2014 年 11 月 11 日公布的一个 IE 浏览器漏洞（同时公布的还有 CVE-2014-6352），影响 IE3—IE11。根据漏洞利用方法，官方定义为远程代码执行漏洞。

究其原因，是因为 VBScript 引擎在执行 `Redim array(nNum)`（即重定义数组）时，会调用 `OLEAUT32.dll` 模块中的 `SafeArrayRedim` 函数。该函数内部处理逻辑不严谨，使用了错误的条件跳转指令；而且当传入的 `nNum` 足够大时，函数内部的数组空间申请会失败，`SafeArrayRedim` 函数不做任何处理直接返回，导致返回时已经将 `nNum` 写入数组结构，后续便可以对该数组进行任意“越界”访问。

## 0x01 漏洞分析

### 1.1 基础知识

#### 1.1.1 VBScript 中的 VARIANT 结构

`VARIANT` 结构是脚本语言中常用的一种结构，用于封装对象结构。参见官方说明：

`787K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3j5i4m8A6i4K6u0r3L8$3q4A6k6r3I4Q4x3V1j5.ns-oaidl-tagvariant`

简化一下该结构，这是一个 `0x10` 大小的结构：

```c
typedef struct _myVARIANT {
    VARTYPE vt;       // 变量类型，占 2 字节
    WORD    wReserved1;
    WORD    wReserved2;
    WORD    wReserved3; // 保留位，共 6 字节
    DWORD   pHighData;  // 指向变量数据的指针，一般用于基本数据类型
    DWORD   pLowData;   // 指向变量数据的指针，一般用于复合数据类型
} myVARIANT;
```

`VARTYPE` 结构参考：

`cf2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4m8J5k6i4k6A6L8%4g2K6i4K6u0V1N6X3g2J5M7$3W2G2L8Y4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3K9h3&6@1k6i4u0F1k6i4c8Q4x3X3c8W2P5s2m8D9L8%4u0W2M7W2)9J5c8X3W2W2i4K6u0V1k6r3g2$3k6h3I4G2M7r3g2J5i4K6u0r3M7$3y4J5K9i4m8@1K9h3&6Y4i4K6u0V1j5i4u0@1K9h3y4D9k6i4y4Q4x3V1j5K6K9$3k6\*7x3e0f1%4K9q4)9J5z5s2k6Q4x3@1c8$3M7#2)9J5k6e0R3@1i4K6t1&6`

列出常用的几种类型如下表所示：

| Constant | Value (DEC) | Description |
| --- | ---: | --- |
| `vbEmpty` | 0 | Empty (uninitialized) |
| `vbNull` | 1 | Null (no valid data) |
| `vbInteger` | 2 | Integer |
| `vbLong` | 3 | Long integer |
| `vbSingle` | 4 | Single-precision floating-point number |
| `vbDouble` | 5 | Double-precision floating-point number |
| `vbString` | 8 | String |
| `vbVariant` | 12 | Variant (used only with arrays of Variants) |
| `Array` | 8192 | `vbArray` |

接下来看一下 VBScript 中的数组结构，参见：

`d52K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3j5i4m8A6i4K6u0r3L8$3q4A6k6r3I4Q4x3V1k6F1M7#2)9J5k6r3!0S2K9h3c8D9i4K6u0V1N6r3q4Y4M7$3q4X3k6h3q4J5M7X3q4&6`

```c
typedef struct tagSAFEARRAY {
    USHORT         cDims;       // 数组维度
    USHORT         fFeatures;
    ULONG          cbElements;  // 每个元素的大小
    ULONG          cLocks;
    PVOID          pvData;      // 指向数组数据的指针
    SAFEARRAYBOUND rgsabound[1];
} SAFEARRAY;

typedef struct tagSAFEARRAYBOUND {
    ULONG cElements;             // 数组元素个数
    LONG  lLbound;               // 起始索引
} SAFEARRAYBOUND, *LPSAFEARRAYBOUND;
```

我们测试一下这些结构在内存中的存储情况：

![](https://bbs.kanxue.com/upload/attach/201812/808521_BWS7TYQ3SEGS7KN.png)

在实际调试中发现，数组元素的类型可以不一致；使用 `redim` 重定义数组时，数组实际的元素个数为传入的 `nNum+1`。

![](https://bbs.kanxue.com/upload/attach/201812/808521_M857FM7BG57CTDJ.png)

#### 1.1.2 VBS 中将函数指针赋值给变量

VBS 中将函数指针赋值给变量时，先将函数类型的 `VARIANT` 结构压入求值栈，然后将求值栈中的结构再传递给变量。但当检测到求值栈中的数据类型为 `0x400C`（即函数类型）时，并不会传递此类型，从而引发相关异常。

如果当前代码流设置了 `On Error Resume Next`，则会忽略当前异常，执行下面的代码。

![](https://bbs.kanxue.com/upload/attach/201812/808521_9PZEWHN2DH6XFVJ.png)

#### 1.1.3 SafeMode

`SafeMode` 是 Windows 操作系统中针对安全的一种特殊模式，即安全模式。在 VBS 引擎中同样存在这样一个安全属性值，正常情况下该属性值为 `0xE`。

默认情况下，VBScript 脚本执行权限是非常低的，正是因为 `SafeMode` 安全属性的限制。如果能通过一定方法修改掉此属性值（改为 `0`），即可绕过安全权限检查，为所欲为，即进入上帝模式。

经过调试，我们发现在 `COleScript` 对象偏移 `0x174` 的位置正是 Safe Mode 标志。

![](https://bbs.kanxue.com/upload/attach/201812/808521_59NNXUJN6ZTHJZV.png)

所以，对该漏洞利用的核心就是找到 `SafeMode` 标志的位置并修改它。

### 1.2 漏洞分析

我们选用附件中一个弹出计算器的 Exp 进行分析（`stage2.html`）。

分享一篇帖子中的调试方法（博客链接见文末）：由于 VBScript 脚本由 VBS 引擎解释执行，如果盲目跟踪解析流程，效率会比较低。调试的目的是复现漏洞位置的代码 bug，所以定位到漏洞模块的解析位置是关键，跟踪关键数据的解析过程即可。

前面为了查看测试变量的内存结构，手动加入了 `IsEmPty()` 函数的调用。在 WinDbg 中模糊搜索该函数，可找到其符号为 `Vbscript.dll!VbsIsEmpty`。将想查看的对象作为参数传给 `VbsIsEmpty()` 函数，即可断在该函数进行对象结构的查看。

为了方便判断脚本执行流程，可以将自定义的日志信息字符串传给 `VbsIsEmpty()` 函数，然后下条件断点：

```text
bp vbscript!VbsIsEmpty "r $t1=poi(@esp+c);r $t2=poi($t1)&0xF;.if ($t2=0x8) {.printf \"%mu\",poi($t1+8);.echo;gc;}.else{}"
```

上述指令的意思是：在 `VbsIsEmpty` 函数处下断点；如果传入参数为字符串（即自定义的日志信息），则打印字符串，然后继续执行；否则就断下来，此时可以查看特定的变量结构。

![](https://bbs.kanxue.com/upload/attach/201812/808521_JSRKZMTM8XPZMBR.png)

我们重点跟踪一下 `redim` 操作，在 `redim Preserve arrX(ofnumele)` 之前跟进 `redim` 流程。搜索 `vbscript.dll` 中与 `redim` 相关的符号，定位到一个可疑函数 `Vbscript.dll!RedimPreserveArray`，在该函数处下断点并结合 IDA 进行动态跟踪验证，程序成功断下。

我们发现在 `RedimPreserveArray` 内部会调用 `SafeArrayRedim` 函数，传入的两个参数（下图中的 `psa`、`&psaboundNew`）分别为原数组对象指针和新的 `SAFEARRAYBOUND` 结构指针。

注意到 `SafeArrayRedim` 函数位于 `OLEAUT32.dll` 模块内。此后将通过 `SafeArrayRedim` 函数的返回值（注意图中的 `v4` 是一个有符号数）判断是否 `redim` 成功；若小于零，则抛出异常并结束 `RedimPreserveArray` 函数。

![](https://bbs.kanxue.com/upload/attach/201812/808521_4M9DT35FZ9QRT6V.png)

继续跟进 `OLEAUT32.dll` 中的 `SafeArrayRedim` 函数。

我们直接反编译 `SafeArrayRedim` 函数。该模块自带符号，简单分析一下该函数的伪代码，即可快速定位其中的逻辑 bug（如下图所示），结合动态验证也正是如此。

![](https://bbs.kanxue.com/upload/attach/201812/808521_MUEFWWKQ9VEM5MM.png)

![](https://bbs.kanxue.com/upload/attach/201812/808521_NMRRZH35SRT54R2.png)

总结一下该漏洞成因，主要有两点：

**A、** `SAFEARRAYBOUND` 结构中的 `cElements` 字段（即数组元素个数）是一个 `ULONG` 类型，即无符号长整型。但是在 `SafeArrayRedim` 函数中处理新旧数组操作时，条件跳转判断选择了 `jge` 有符号跳转指令（源代码中应该是定义错了数据类型）。

这里所说的处理，是指上两图中对于变动的数组空间开辟新的缓冲区进行保存备份操作。

这样就存在一种可能：如果 `psaboundNew->cElements - psa->rgsabound.cElements >= 0x08000000`，即数组元素个数增加的值大于或等于 `0x08000000`，其变动的大小就会大于等于 `0x80000000`。该数值会占用有符号数的符号位，使图中的 `v5` 恒小于 `0`，随后便会产生错误的跳转。

**B、** `SafeArrayRedim` 函数中的处理流程是先将 `psaboundNew` 结构赋给 `psa->rgsabound`，随后按照数组空间变动的大小去申请缓冲区。如果申请失败，则会返回 `0x8007000`，然后外层的 `Vbscript.dll!RedimPreserveArray` 函数将会抛出异常退出。

这里的处理存在一个非常严重的问题：`RedimPreserveArray` 函数 `redim` 失败之后却并未还原 `psa->rgsabound.cElements`，这让后续对 `psa` 数组的“越界”访问合法化。

至此，漏洞成因已分析完毕。

## 0x02 Exploit 分析

接下来，我们分析一下附件中的漏洞利用文件 `stage2.html`，学习该 Exp 的编写技巧及思路。

前面的分析中提到，该漏洞的常规利用方法是修改 VBS 引擎中的 `SafeMode` 标志，开启上帝模式；而且我们知道 `COleScript` 对象偏移 `0x174` 位置正是 Safe Mode 标志。

所以，分析的重点是如何找到 Safe Mode 标志的具体位置，并合法地修改该标志。

笔者在调试 `stage2.html` 时，简单总结了一下代码流程：

1. 初始化数组 `arrX()`、`arrY()`，即 `Init()` 函数。
2. 通过循环 `redim`，尝试使 `arrX` 和 `arrY` 的空间“连续”（准确来说，是使原始数组 `arrX` 和 `arrY` 之间只间隔 8 字节），即 `Exploit()` 函数。
3. 触发漏洞，修改 `arrX` 的 `cElements` 字段，使 `arrX` 数组可以“越界”访问 `arrY` 数组元素，让两个数组的后续空间重叠，即 `Trigger()` 函数。
4. 通过前面提到的将自定义函数指针赋给变量的方法，获取 `CScriptEntryPoint` 对象地址，为获取 Safe Mode 标志位置做准备，即 `LeakFnAddr()` 函数。
5. 获取 `COleScript` 对象地址，并通过数组重叠后的特性构造特定的数组空间布局以及伪造的数组结构 `myarray`，成功访问 Safe Mode 并将其属性值修改为 `0`。此时已成功开启上帝模式，即 `EnableGodMode()` 函数。
6. 执行自定义的 shell，即 `runshell()` 函数。

详细调试注释见附件中的 `z.html` 文件。这里重点分析第 4 步和第 5 步。依然选用前面提到的通过打印日志快速定位关键代码的方法。

![](https://bbs.kanxue.com/upload/attach/201812/808521_QAFFN6M4SX5NTTJ.png)

![](https://bbs.kanxue.com/upload/attach/201812/808521_95J3A9RE6GHJQXG.png)

![](https://bbs.kanxue.com/upload/attach/201812/808521_93EN67F5X987XCN.png)

随后，将函数 `arrX(olapPos)` 赋给 `LeakFnAddr` 返回。注意，此时是将 `CScriptEntryPoint` 对象地址作为长整型赋给 `LeakFnAddr`。

然后调用 `ReadMenInt` 函数获取 `COleScript` 对象地址。前面已经知道，`poi(pCScriptEntryPoint+8)+0x10` 即可获取 `COleScript` 对象地址。

这里需要注意两个细节：

**A、** 通过 `arrY(0)=8`，间接地将 `arrX(olapPos)` 的类型修改为 `8`（即 `v_str` 字符串类型）。

**B、** `Lenb` 函数的作用是获取字符串的字节总数。传入 `v_str` 类型参数时，该函数会按照 `v_str` 结构，默认将字符串指针前四字节的 `DWORD` 值作为结果返回。这也是前面执行 `arrX(olapPos)=addr+4` 的原因。

![](https://bbs.kanxue.com/upload/attach/201812/808521_HK8K46K6DQBHBEW.png)

获取到 `COleScript` 对象地址之后，接下来要解决的问题便是如何合规地修改 `COleScript+0x174` 位置的 `SafeMode` 标志。

我们来分析一下这里的流程：

1. 构造一个自定义的数据类型，即 `myarray`。其数据内容为：

   `08800001 00000001 00000000 00000000 7fffffff 00000000`

   这里 `0x18` 字节的数据刚好是一个数组的结构：一个单元素字节长度为 `1`、总共 `7fffffff` 个元素的一维数组，即 `byte array[0x7fffffff]`。`myarray` 是指向这个数组的自定义结构。

2. 将 `myarray` 对象赋给 `arrX(olapPos+2)`。此时，`arrX(olapPos+2)` 类型为 `0x4a`，值为 `0x00676584`，即 `myarray` 对象地址。

3. `arrY(2)=8192+12`，间接地将 `arrX(olapPos+2)` 类型修改为 `0x200c`，即 `vbArray|vbVariant` 类型（数组 | 变体类型）。

4. 此时，`arrX(olapPos+2)` 已成功修改为数组类型，现在可以正常访问该数组空间。该数组空间从 `0x00000000` 到 `0x7ffffffe`，即整个用户空间，`SafeMode` 肯定位于整个空间中。

5. 前面已经获取 `COleScript` 对象地址并传递给 `i` 保存，因此直接通过 `arrX(olapPos+2)(i+0x174)` 便可以访问 `SafeMode`，并将其修改为 `0`。

![](https://bbs.kanxue.com/upload/attach/201812/808521_69FW446PKJBK6DH.png)

![](https://bbs.kanxue.com/upload/attach/201812/808521_9QY34ETG8HXBBJM.png)

至此，利用分析已完毕。

## 0x03 填个坑

笔者因为较少编写 VBScript 文件，在分析该 Exp 时随手写了较多注释，然后调试就出现问题了……

