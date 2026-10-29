
经过搜索，发现`MmMapViewOfSection`函数的资料很少，只在第三方开源内核reactos上找到了定义：

```
NTSTATUS NTAPI MmMapViewOfSection(
    _In_ PVOID SectionObject,
    _In_ PEPROCESS Process,
    _Inout_ PVOID * BaseAddress,
    _In_ ULONG_PTR  ZeroBits,
    _In_ SIZE_T CommitSize,
    _Inout_opt_ PLARGE_INTEGER SectionOffset,
    _Inout_ PSIZE_T ViewSize,
    _In_ SECTION_INHERIT InheritDisposition,
    _In_ ULONG AllocationType,
    _In_ ULONG Protect
)
```

不过，经过跟踪发现`NtMapViewOfSection`最终调用了`MmMapViewOfSection`，而MSDN上说`NtMapViewOfSection`与`ZwMapViewOfSection`是一样的，下面是`ZwMapViewOfSection`：

```
NTSTATUS ZwMapViewOfSection(
    _In_         HANDLE SectionHandle,
    _In_         HANDLE ProcessHandle,
    _Inout_      PVOID *BaseAddress,
    _In_         ULONG_PTR ZeroBits,
    _In_         SIZE_T CommitSize,
    _Inout_opt_  PLARGE_INTEGER SectionOffset,
    _Inout_      PSIZE_T ViewSize,
    _In_         SECTION_INHERIT InheritDisposition,
    _In_         ULONG AllocationType,
    _In_         ULONG Win32Protect
);
```

很多资料都指向了`BaseAddress`参数，这个参数默认是0，但是修改后，能够修改加载地址。因此重点对这个参数进行跟踪。

注意：由于之前我的分析记录将`BaseAddress`写成了`ImageBase`，因此`BaseAddress`名称在下文中会用`ImageBase`代替。

## 对Hook函数的分析

经过以上的Inline Hook后，对`MmMapViewOfSection`的调用会跳转到`0xBA5BA3DA`。看来`0xBA5BA3DA`处的函数是关键了。

注意：这里我把360的`0xBA5BA3DA`处的Hook函数命名为`MmHook`，下面会以这个名称代替。

### MMHOOK函数流程

`MmHook`的流程比较绕，不过还是能看的出其大体的框架。刚开始，做了一些初始化工作。然后重点判断了当前内存域是内核域还是用户域，如果是内核域，则继续判断模块是否是符合修改基址的要求。如果是用户域，则判断模块是否已经被处理过。如果被处理过，则直接调用原始的`MmMapViewOfSection`函数。否则，进入基址的处理过程。代码分析如下：

```
.text:BA5BA3DA mov     edi, edi
.text:BA5BA3DC push    ebp
.text:BA5BA3DD mov     ebp, esp
.text:BA5BA3DF sub     esp, 30h
.text:BA5BA3E2 or      [ebp+var_14], 0FFFFFFFFh
.text:BA5BA3E6 push    ebx
.text:BA5BA3E7 push    esi
.text:BA5BA3E8 mov     esi, ds:ExGetPreviousMode
.text:BA5BA3EE push    edi
.text:BA5BA3EF xor     edi, edi
.text:BA5BA3F1 mov     [ebp+var_1], 0
.text:BA5BA3F5 mov     [ebp+var_3], 0
.text:BA5BA3F9 mov     [ebp+var_5], 0
.text:BA5BA3FD mov     [ebp+var_2], 0
.text:BA5BA401 mov     [ebp+var_4], 0
.text:BA5BA405 mov     [ebp+var_10], edi
.text:BA5BA408 mov     [ebp+ImageBase], edi            ; 以上是做初始化工作，将局部变量清零
.text:BA5BA40B call    esi ; ExGetPreviousMode         ; 查询当前的内存域是内核进程还是用户进程。
.text:BA5BA40D mov     ebx, [ebp+arg_8]                ; 取得模块基址的指针保存到ebx
.text:BA5BA410 cmp     al, 1
.text:BA5BA412 jnz     short loc_BA5BA44A              ; 如果是内核域，则跳转
.text:BA5BA414 mov     eax, [ebx]                      ; 判断当前为用户域进程（如果为内核域进程，则需要额外判断是否跳过修改基址的操作），此句将模块基址保存到eax
.text:BA5BA416 cmp     eax, edi                        ; 判断eax是否为0，为0则跳转到基址处理分支。eax保存了模块基址指针，未作处理的模块，基址为0，需要做基址处理。否则eax保存处理过后的基址地址。
.text:BA5BA418 jz      short loc_BA5BA452              ; 这个分支貌似是内存基址处理分支，大部分用户态模块和少数内核态模块都经此分支，内核态貌似有ntdll.dll进行了特殊处理，将7C920000基址改为了77920000，并将此地址保存到了全局变量BA5FBB7C处。
.text:BA5BA41A cmp     eax, offset off_10000           ; 比较模块基址与0x10000的大小
.text:BA5BA41F jnb     short loc_BA5BA44A              ; 如果模块基址大于0x10000，则跳转，之后会不做任何处理，调用原始Mm函数
......
```

对于符合要求的内核模块和用户模块，则会跳转到`0xBA5BA452`分支，从这个分支起，开始做基址的处理，期间还做了大量的判断工作，以区分不同的情况。

接着往下分析，我找到了四个调用原始`MmMapViewOfSection`函数的地方：

- `0xBA5BA5C4`
- `0xBA5BA7D7`
- `0xBA5BA8C6`
- `0xBA5BA92F`

其中只有`0xBA5BA8C6`处的调用，修改了`MmMapViewOfSection`的`ImageBase`参数。而且`ImageBase`与模块加载基址吻合，也就是说，之前的猜测是正确的，`ImageBase`的修改确实引起了模块加载基址的变化。并且，我猜测这个分支上面有计算地址的代码。从`0xBA5BA8C6`往上溯源，可以在`0xBA5BA854`处找到计算函数`0xBA5C32DC`。在分析这个函数之前，先得准备些基础的内容。

## 内存的位图管理方式

地址的计算，涉及到Windows内核的一种内存分配管理方式。这个方式网上资料很少（可能我搜索姿势不对），手头也没有相关的书籍。在看了好多零零散散的文章后，大概摸清了这种内存分配管理方式。

系统把内存分为很多区块，每个区块有固定的大小（具体到本文是`0x10000`）。然后开辟一个位图空间（Bitmap），这个空间的每一位，都代表相应的区块是否被占用。我们假设区块大小为`0x10000`，那么一个4字节（32位）位图空间，就可以管理`0x4 * 0x8 * 0x10000 = 0x200000`——也就是2KB大小的内存。这个位图空间如下图：

![](https://bbs.kanxue.com/upload/attach/201501/662791_xex6lv2i2rze9zt.png)

位图

上图标识为1的区块表示已被占用。如果我们需要3个区块的内存空间，从0索引位开始搜索，那么我们会得到8~10区块。

当然，我们还可以从指定索引位置搜索。比如，我们还是需要3个区块的内存空间，但我们不从0索引位开始搜索，我们指定从第20索引位开始搜索，这样我们得到的区块是20~22。这种方式是实现地址随机的关键，下面会用到。

我们再来看一个实例，假设我们需要管理640MB（`0x280`）的内存空间，每个内存块是64KB（`0x10000`）字节大小。那么我们需要`640 * 1024 / 64 = 10240`位，即`10240 / 8 = 1280 (0x500)`字节的空间，来表示这640MB空间那些块用过，哪些没有用过。这也是360的做法，具体会在下文说到。

Windows内核提供了以下管理函数来实现这种内存管理方式：

```
VOID RtlInitializeBitMap(
    _Out_  PRTL_BITMAP BitMapHeader,
    _In_   PULONG BitMapBuffer,
    _In_   ULONG SizeOfBitMap
);
```

初始化一个位图空间。`PRTL_BITMAP`结构体是返回值，保存了位图空间的大小以及地址。

```
ULONG RtlFindClearBits(
    _In_  PRTL_BITMAP BitMapHeader,
    _In_  ULONG NumberToFind,
    _In_  ULONG HintIndex
);
```

寻找连续的指定大小的未使用的内存索引。`BitMapHeader`是使用`RtlInitializeBitMap`初始化的位图空间结构体。`NumberToFind`代表需要寻找的连续区块的个数。`HintIndex`代表起始的搜索位置，默认是从位图空间第0位开始搜索。之前我们说到过自定义搜索起始位置，这个参数就是用来实现这个功能的。

```
ULONG RtlFindClearBitsAndSet(
    _In_  PRTL_BITMAP BitMapHeader,
    _In_  ULONG NumberToFind,
    _In_  ULONG HintIndex
);
```

寻找并分配连续的指定大小的未使用的内存索引。比上面的函数多了一个分配的动作，如果能找到连续的区块，则将这些区块位置为1，表示已占用。

当然还有其它的，不过360主要使用了上面三个函数。

## 360内存分配的流程及地址随机化的实现

### 初始化工作，创建位图空间及单字节随机值

`360rosdrv`在初始化时，创建了一个可管理640MB大小内存的位图空间，用于模块内存加载地址的分配管理。同时，还创建了一个单字节的随机值`HintIndex`，这个随机值用于之后的地址随机化。以下是代码分析：

```
.text:BA5C326A
.text:BA5C326A ; =============== S U B R O U T I N E =======================================
.text:BA5C326A
.text:BA5C326A ; 生成RTL_BITMAP中Buffer所需的存储空间（0x500(1280)字节，0x2800(10240)位），同时产生HintIndex这个决定基址模块的随机值
.text:BA5C326A
.text:BA5C326A sub_BA5C326A    proc near               ; CODE XREF: sub_BA5BC8FA+53
.text:BA5C326A                 call    sub_BA5C27B8    ; 生成随机值
.text:BA5C326F                 push    4D49424Dh       ; Tag
.text:BA5C3274                 push    500h            ; NumberOfBytes
.text:BA5C3279                 and     eax, 0FFh       ; 取生成的随机数的末尾字节（HintIndex只需一个字节）
.text:BA5C327E                 push    0               ; PoolType
.text:BA5C3280                 mov     HintIndex, eax  ; 全局变量，内存块起始搜索位置。通过修改这个数值，可以达到模块基址随机变化的目的。
.text:BA5C3285                 call    ds:ExAllocatePoolWithTag ; 分配0x500(1280)字节的内存，用于存储位图空间数据
.text:BA5C328B                 test    eax, eax
.text:BA5C328D                 jnz     short loc_BA5C3290
.text:BA5C328F                 retn
.text:BA5C3290 ; ---------------------------------------------------------------------------
.text:BA5C3290
.text:BA5C3290 loc_BA5C3290:                           ; CODE XREF: sub_BA5C326A+23
.text:BA5C3290                 push    esi
.text:BA5C3291 位图大小为0x2800(10240)位
.text:BA5C3291                 push    2800h           ; SizeOfBitMap
.text:BA5C3296 位图空间数据的保存位置，就是上面分配的内存
.text:BA5C3296                 push    eax             ; BitMapBuffer
.text:BA5C3297                 mov     esi, offset BitMapHeader
.text:BA5C329C RTL_BITMAP结构体保存位置
.text:BA5C329C                 push    esi             ; BitMapHeader
.text:BA5C329D                 call    ds:RtlInitializeBitMap
.text:BA5C32A3                 push    esi             ; BitMapHeader
.text:BA5C32A4 将位图空间所有的位全部清零
.text:BA5C32A4                 call    ds:RtlClearAllBits
.text:BA5C32AA                 and     dword_BA5FC028, 0
.text:BA5C32B1                 xor     eax, eax
.text:BA5C32B3                 inc     eax
.text:BA5C32B4                 pop     esi
.text:BA5C32B5                 retn
.text:BA5C32B5 sub_BA5C326A    endp
.text:BA5C32B5
.text:BA5C32B5 ; ---------------------------------------------------------------------------
```

### 计算`IMAGEBASE`

我们首先通过一个例子，来了解整个的计算过程，然后通过代码分析，看看具体的代码实现。

#### 计算过程

计算最终的地址，需要3个参数：

- `0x7800`: 起始内存地址的高16位，是个常量，写死在程序里的。换句话说，360将所有需要ASLR的模块加载到`0x78000000`以下的地址。
- `HintIndex`: 位图空间的起始搜索位，是个8位的随机值，`RtlFindClearBits`中的参数之一。我们说过，位图空间的搜索不一定非得从第0位开始。
- `ModuleBlockSize`: 模块所需内存区块的数量，每个区块的大小为`0x10000`。使用模块所占内存大小计算而来。

我们通过一个例子来说明整个地址的计算过程：

```
# 初始值，设
HintIndex = 0x7E
ModuleSize = 0x96000 # 模块所需内存大小
```

首先，由于内存的分配是以`0x10000`为单位分配的，那我们需要得到模块加载到内存需要多少个内存区块：

```
0x96000 >> 12 = 0x96
0x96 + 0xF = 0xA5
0xA5 >> 4 = 0xA
```

因此`ModuleBlockSize = 0xA`，模块至少需要`0xA`个内存区块。

然后，在10240个位的位图空间中，从`HintIndex`（也就是第`0x7E`位）开始，搜索具有`0xA`个连续空位（即置为0的位）的位置，这里假设搜索结果为`BlockIndex = 0x1F0`，表示在第`0x1F0`位处找到了`0xA`个连续的为0的位。换句话说，从第`0x1F0`个内存区块起有连续`0xA`个空着的内存区块。

最后，计算加载模块的起始内存地址：

```
0x7800 - BlockIndex = 0x7800 - 0x1F0 = 0x7610
0x7610 - ModuleBlockSize = 0x7610 - 0xA = 0x7606
0x7606 << 16 = 0x76060000
```

