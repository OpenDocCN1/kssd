
# [原创]Office EPS 文件解析漏洞分析

作者：Joey@天玄安全实验室

## 前言

第一次分析 EPS 类漏洞，对于 PostScript 格式十分陌生，通过查阅<a href="elink@039K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2k6r3!0T1k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6V1j5h3#2Q4x3V1k6S2j5$3!0E0i4K6u0r3k6h3&6Q4x3V1k6V1k6i4k6F1k6i4c8Q4x3V1k6S2j5%4c8A6L8$3&6K6j5%4u0A6M7s2c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8W2m8x3f1V1#2Q4x3X3g2H3k6r3j5`.">PostScript LANGUAGE REFERENCE</a>了解 PostScript 格式。调试 EXP 来自 kcufld 师傅的<a href="elink@733K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8j5%4g2X3d9h3c8Q4x3V1k6W2M7s2y4Q4x3X3c8o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3o6t1$3x3b7`.`.">eps-CVE-2017-0261</a>，EXP 在 Office 2007 可以正常运行，但在 Office 2010 以上版本需要配合提权漏洞逃逸沙箱后完成利用。

### 调试环境

调试是直接使用 kcufld 师傅的 EPS 加载器进行调试，`EPSIMP32.FLT` 版本信息如下：

```
OS: Win7 x64 SP1
Office: Ofiice 2007 x86
Image name: EPSIMP32.FLT
ImageSize: 0x0006E000
File version: 2006.1200.4518.1014
Product version: 2006.1200.4518.0
```

## 漏洞分析

### PostScript 格式简介

先介绍下 PostScript 基本的数据结构：

| SIMPLE OBJECTS | COMPOSITE OBJECTS |
| --- | --- |
| boolean | array |
| fontID | dictionary |
| integer | file |
| mark | gstate (LanguageLevel 2) |
| name | packedarray (LanguageLevel 2) |
| null | save |
| operator | string |
| real | |

左侧为简单对象，右侧为复合对象。简单对象都是原子实体，类型、属性和值不可逆转地结合在一起，不能改变。但复合对象的值与对象本身是分开的，对象本身存储于操作栈中，具体的结构如下：

```c
// PostScript Object
struct PostScript object
{
    dword type; //对象类型
    dword attr;
    dword value1; //指向对象所属变量名称
    dword value2; //若为简单对象，直接指向值；若为复合对象，指向存储的值的结构
} ps_obj;
```

其中部分 `type` 的值与类型的映射如下：

| type 值 | 数据类型 |
| --- | --- |
| `0x0` | nulltype |
| `0x3` | integertype |
| `0x5` | realtype |
| `0x8` | booleantype |
| `0x10` | operatortype |
| `0x20` | marktype |
| `0x40` | savetype |
| `0x300` | nametype |
| `0x500` | stringtype |
| `0x900` | filetype |
| `0x30000` | arraytype |
| `0x70000` | packedarraytype |
| `0x0B0000` | packedarraytype |
| `0x110000` | dicttype |
| `0x210000` | gstatetype |

接着介绍下漏洞中使用到的比较关键的操作符的意义：

| 操作符 | 示例 | 解析 |
| --- | --- | --- |
| `forall` | `array proc forall` | 枚举第一个操作数的元素，为每个元素执行过程 `proc`。如果第一个操作数是数组、压缩数组或字符串对象，则 `forall` 将一个元素压入操作数堆栈，并对对象中的每个元素执行 `proc`，从索引为 0 的元素开始并依次执行。 |
| `dup` | `any dup ---> any any` | 复制操作数堆栈上的顶部元素。`dup` 只复制对象；复合对象的值不是复制而是共享的。 |
| `putinterval` | `array1 index array2 putinterval` | 用第三个操作数的全部内容替换第一个操作数的元素的子序列。被替换的子序列从第一个操作数的 `index` 开始；它的长度与第三个操作数的长度相同。 |
| `put/get` | `array index any put/get` | 替换/获取第一个操作数的一个元素的值。如果第一个操作数是一个数组或一个字符串，`put/get` 将第二个操作数视为一个索引，并将第三个操作数存储在索引所确定的位置，从 0 开始计算。 |
| `save` | `/save save` | 保存当前 VM 状态快照，一个快照只能使用一次。 |
| `restore` | `save restore` | 丢弃本地 VM 中自相应保存以来创建的所有对象，并回收它们占用的内存；将本地 VM 中所有复合对象的值（字符串除外）重置为保存时的状态；关闭自相应保存以来打开的文件，只要这些文件在 local VM 分配模式有效时打开。 |

了解了上述背景后，开始分析漏洞。

### 漏洞成因

通过使用 `forall` 操作符获取创建的字符串对象，并在第一次循环时使用 `restore` 操作符释放字符串对象，随后创建新的字符串对象使得原本存储旧字符串对象的结构被新复合对象代替。若故意构造大小为 `0x27` 的字符串对象，则字符串被释放后会多出 `0x28` 的内存空间，此时立即创建新的字符串对象，则该内存会用来存储指向新字符串的 string 结构。随后通过改变 `forall` 的函数，获取指向新字符串的结构。

漏洞文件中一共触发了三次漏洞，第一次是获取了被释放的 string 的字符用于判断系统是 32 位还是 64 位。第二次触发故意构造大小为 `0x27` 的 string 对象，用于获取指向恶意 string 的结构。第三次则利用第二次构造的特殊 string 结构创造了一个起始地址为 `0x00000000`、大小为 `0x7fffffff` 的字符串，构造了能够读写任意地址内存的读写原语。接着利用读写原语搜索内存中函数地址构造 ROP 链。最终创建了一个文件对象，在调用 `closefile` 操作符时跳转执行 ROP 完成漏洞利用。

查看 `poc.eps` 文件，第一次调用 `forall` 如图所示：

![image-20210903105452944](upload/attach/202109/929902_MJXN8R7AX6M3ZGY.jpg)

在 IDA 中定位到 `forall` 操作符的代码：

![image-20210902173157063](upload/attach/202109/929902_ETPBP248HYZ6HZX.jpg)

使用 WinDbg 找到对应偏移后下断：

`sxe ld EPSIMP32;g;bp EPSIMP32+2b928;g;`

![image-20210831150138806](upload/attach/202109/929902_X7AVZQGKJ6XM84H.jpg)

运行到图中所示位置时查看 `edi` 的值，指向了操作栈，查看后发现有两个对象在栈中，第一个为 string `l63`，第二个为 array `l61`。

![image-20210910180510699](upload/attach/202109/929902_5AASQDDNCCYXSKZ.jpg)

继续分析，会获取 `l63` 和 `l61` 对象到栈中，并确认 `l63` 的类型为 string 后，跳转到获取 string 类型元素部分。

![image-20210831161042602](upload/attach/202109/929902_E8T55BBRNYESKDS.jpg)

获取值的过程会因为 `type` 的不同而有所变化，具体如图所示：

![image-20210901172253284](upload/attach/202109/929902_FHBKNN95A2JPHGR.jpg)

通过调试可以更加直观地看到通过 `value2` 获取 string 的方式：

![image-20210831171553055](upload/attach/202109/929902_3YEC2F9M6GKMJRN.jpg)

接着循环获取 string 中的每一个元素并执行函数：

![image-20210831192324185](upload/attach/202109/929902_7FC8N37N2QH8QYN.jpg)

此时传入 `deferred_exec` 的参数为 `eax`，查看传入参数：

```
0:000> bp EPSIMP32+2ba06 //call deferred_exec
0:000> g
Breakpoint 1 hit
eax=0018fd78 ebx=00000000 ecx=00291280 edx=00000001 esi=00425770 edi=00000000
eip=718fba06 esp=0018fd54 ebp=0018fdbc iopl=0 nv up ei pl nz na po nc
cs=0023 ss=002b ds=002b es=002b fs=0053 gs=002b efl=00000202
EPSIMP32!RegisterPercentCallback+0x4604:
718fba06 e8d8abffff call deferred_exec (718f65e3)
0:000> dd eax L4 //查看传入的参数为数组
0018fd78 00030000 00000000 0049ea98 0048f40c
0:000> dd poi(poi(poi(poi(poi( 0018fd78 +c))+24))+28) //查看数组中存储的内容
0049e2c0 00000500 00000100 00495408 0048ee98 //数组中存放着字符串对象
0049e2d0 12d85688 8000f194 00000020 00000100
0049e2e0 0049dc40 0048f198 12d8568f 80000000
0049e2f0 00490023 000007c8 00000300 00000100
0049e300 12d856b2 8000f19c 00000026 00000100
0049e310 0049dc60 0048f1a0 12d856b1 80000100
0049e320 00420029 0048f1a4 00000003 00000000
0049e330 12d856b4 8000f194 0000002c 00000100
0:000> db poi(poi(poi(poi(poi( 0049e2c0 +c))+24))+20) L10 //查看字符串的内容为l56 cvx exec
00495940 20 6c 35 36 20 63 76 78-20 65 78 65 63 20 00 00 l56 cvx exec ..
0:000> g //第二次执行deferred_exec
(5c8.144): C++ EH exception - code e06d7363 (first chance)
(5c8.144): C++ EH exception - code e06d7363 (first chance)
Breakpoint 1 hit
eax=0018fd78 ebx=00000000 ecx=00291280 edx=00000003 esi=00425770 edi=00000001
eip=718fba06 esp=0018fd54 ebp=0018fdbc iopl=0 nv up ei pl nz na pe nc
cs=0023 ss=002b ds=002b es=002b fs=0053 gs=002b efl=00000206
EPSIMP32!RegisterPercentCallback+0x4604:
718fba06 e8d8abffff call EPSIMP32+0x265e3 (718f65e3)
0:000> dd poi(poi(poi(poi(poi( 0018fd78 +c))+24))+28) //查看数组的内容
0049e2c0 00000500 00000100 00495438 0048eeac //数组中存放着字符串对象
0049e2d0 12d85688 8000f194 00000020 00000100
0049e2e0 0049dc40 0048f198 00000020 00000100
0049e2f0 00490023 000007c8 00000300 00000100
0049e300 12d856b2 8000f19c 00000026 00000100
0049e310 0049dc60 0048f1a0 12d856b1 80000100
0049e320 00420029 0048f1a4 00000003 00000000
0049e330 12d856b4 8000f194 0000002c 00000100
0:000> db poi(poi(poi(poi(poi( 0049e2c0 +c))+24))+20) L10 //查看字符串的内容为l53 cvx exec
00495958 20 6c 35 33 20 63 76 78-20 65 78 65 63 20 00 00 l53 cvx exec ..
```

从调试的结果可以得知，该函数执行的正是 `forall`。在第一次执行时，`l61` 中待执行的命令是 `l56 cvx exec`，在第二次执行时，`l61` 中的内容已经被换成了 `l53 cvx exec`，与调试结果相符。

接着深入函数分析，发现函数内部嵌套了 `deferred_exec`：

![image-20210901193647205](upload/attach/202109/929902_ZYXM5GFFSK3Q5DC.jpg)

于是重新调试，下断在此，分析参数：

![image-20210901201303745](upload/attach/202109/929902_PZM9KRPTATQM4KQ.jpg)

虽然 `type` 为 `0x10` 的操作符对象存储在 Systemdict 中无法查看，但是通过其他字符和数字还是能够确定该语句就是 `l50`。当执行该语句后，原本 `l63` 指向的 string 结构将被替换成存放 `l52` 内容的 string 结构：

![image-20210901211439785](upload/attach/202109/929902_7VCP2PXZP5SJG5N.jpg)

可以看到此时原本存放 `l63` 的 string 结构已经变成了 `l52`。

在 `get` 函数下断，跳转到 `forall` 下的 `/l64 l57 56 get def` 语句查看 `l57` 的值：

![image-20210903144034585](upload/attach/202109/929902_TJ4UR3FY597PPD5.jpg)

可以证实 `l57` 中存放的就是从 `l63` 中获取到的字符，该 `forall` 的作用就是泄露被释放的 string 结构指向的字符串。

接着获取 `l57` 中的值，并进行一些处理，通过 `ifelse` 判断系统位数，若 `l77` 等于 `l52` 的长度 + 1，那么 `l99` 的值为 1，代表系统为 64 位，否则 `l99` 为 0，代表系统为 32 位：

![image-20210903164816672](upload/attach/202109/929902_YZYCM3HEXTXWP2W.jpg)

可以看到在 32 位的调试环境下，`l77` 的值为 0，因此会将 5 个 0 压入操作栈中，并赋值给 `l95` 到 `l99`：

![image-20210903165138520](upload/attach/202109/929902_2HCG475UQ7CNPDC.jpg)

至此，漏洞原理部分分析完毕，接下来分析漏洞利用部分。

## 漏洞利用

第二次执行 `forall` 代码如下：

![image-20210903171728048](upload/attach/202109/929902_Y29RQVZD4Z94GZS.jpg)

和第一次执行十分类似，因此就不深入分析。查看执行完 `forall` 后 string `l63` 的变化：

![image-20210904155018524](upload/attach/202109/929902_3KMU8PWWYN8P4VT.jpg)

查看 `l63` 中的值，发现是一个 string 结构，于是查看字符串，内容正是 `l102` 中存储的 `l36` 的字符串。

![image-20210904155250267](upload/attach/202109/929902_8B2PT6W7R2AA694.jpg)

接着通过 `l90 0 l92 putinterval` 将 `l63` 中指向的第一个 4 字节的内容改为 `0x02b14ad4`，该值指向 `l36` 中四字节之后的内容。

![image-20210904161420028](upload/attach/202109/B95JGVDKGS53DQ4.jpg)

经过多次修改，字符串修改为如下状态，修改的值会在第三次漏洞触发时使用到：
