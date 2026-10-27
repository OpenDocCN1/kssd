
```php
kd> dt \_MMVAD\_FLAGS 0x819d2ca8 + 0x14
nt!\_MMVAD\_FLAGS
+0x000 CommitCharge : 0y0000000000000000001 (0x1)
+0x000 PhysicalMapping : 0y0
+0x000 ImageMap : 0y0
+0x000 UserPhysicalPages : 0y0
+0x000 NoChange : 0y0
+0x000 WriteWatch : 0y0
+0x000 Protection : 0y00100 (0x4)
+0x000 LargePages : 0y0
+0x000 MemCommit : 0y1
+0x000 PrivateMemory : 0y1
```

为了方便查看一个进程的内存使用情况，WinDbg提供了`vad`命令来解析这颗二叉树，如下就是对记事本进程的内存使用情况

```php
kd> !vad 0x81d14f48
VAD level start end commit
81d13148 ( 3) 10 10 1 Private READWRITE
819d2ca8 ( 2) 20 20 1 Private READWRITE
81aa1220 ( 5) 30 3f 5 Private READWRITE
81dca1e0 ( 4) 40 7f 18 Private READWRITE
81a54570 ( 3) 80 82 0 Mapped READONLY Pagefile-backed section
81d22888 ( 4) 90 91 0 Mapped READONLY Pagefile-backed section
81e7af58 ( 1) a0 19f 20 Private READWRITE
819598e0 ( 4) 1a0 1af 6 Private READWRITE
81eac848 ( 3) 1b0 1bf 0 Mapped READWRITE Pagefile-backed section
81a503b8 ( 4) 1c0 1d5 0 Mapped READONLY \WINDOWS\system32\unicode.nls
81a503e8 ( 2) 1e0 220 0 Mapped READONLY \WINDOWS\system32\locale.nls
81d14f78 ( 4) 230 270 0 Mapped READONLY \WINDOWS\system32\sortkey.nls
819d3a18 ( 3) 280 285 0 Mapped READONLY \WINDOWS\system32\sorttbls.nls
81d14f48 ( 0) 290 2d0 0 Mapped READONLY Pagefile-backed section
81d170f8 ( 5) 2e0 3a7 0 Mapped EXECUTE\_READ Pagefile-backed section
81e6e220 ( 6) 3b0 3bf 8 Private READWRITE
81d146b0 ( 7) 3c0 3c0 1 Private READWRITE
81b2edc8 ( 8) 3d0 3d0 1 Private READWRITE
81ad5368 (10) 3e0 3e1 0 Mapped READONLY Pagefile-backed section
81ad5338 ( 9) 3f0 3f1 0 Mapped READONLY Pagefile-backed section
81ad5318 (10) 400 40f 3 Private READWRITE
81950eb8 ( 4) 410 41f 8 Private READWRITE
81dd0388 ( 3) 420 42f 4 Private READWRITE
81dd0358 ( 4) 430 432 0 Mapped READONLY \WINDOWS\system32\ctype.nls
81aa4190 ( 2) 440 47f 3 Private READWRITE
81e6e240 ( 5) 480 582 0 Mapped READONLY Pagefile-backed section
81960680 ( 4) 590 88f 0 Mapped EXECUTE\_READ Pagefile-backed section
81aa8ae0 ( 3) 890 90f 1 Private READWRITE
81aba778 ( 5) 910 910 0 Mapped READWRITE Pagefile-backed section
81b1b2e8 ( 4) 920 95f 0 Mapped READWRITE Pagefile-backed section
81e7ba20 ( 6) 960 96d 0 Mapped READWRITE Pagefile-backed section
81d158b8 ( 5) 970 a6f 123 Private READWRITE
81a55f98 ( 7) a70 a73 0 Mapped READWRITE Pagefile-backed section
81e6e1e0 ( 6) a90 b0f 0 Mapped READWRITE Pagefile-backed section
81e2dab8 ( 7) b10 b8f 0 Mapped READWRITE Pagefile-backed section
81b2ed88 ( 1) 1000 1012 3 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\notepad.exe
81952cc0 ( 7) 58fb0 59179 10 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\AppPatch\AcGenral.dll
81d84a88 ( 8) 5adc0 5adf6 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\uxtheme.dll
81d170c8 ( 6) 5cc30 5cc55 21 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\shimeng.dll
81960650 ( 7) 62c20 62c28 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\lpk.dll
81e72fd8 ( 5) 72f70 72f95 3 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\winspool.drv
81e7b9f0 ( 8) 73640 7366d 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\MSCTFIME.IME
81d226c8 ( 7) 73fa0 7400a 17 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\usp10.dll
8196a608 ( 8) 74680 746cb 3 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\MSCTF.dll
81d126e8 ( 6) 759d0 75a7e 3 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\userenv.dll
81aa4160 ( 7) 76300 7631c 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\imm32.dll
819d32b8 ( 4) 76320 76366 5 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\comdlg32.dll
81d14ee8 ( 8) 76990 76acd 8 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\ole32.dll
81e6d480 ( 7) 76b10 76b39 3 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\winmm.dll
819d46c0 ( 8) 770f0 7717a 4 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\oleaut32.dll
81d14eb8 ( 6) 77180 77282 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\WinSxS\x86\_Microsoft.Windows.Common-Controls\_6595b64144ccf1df\_6.0.2600.6028\_x-ww\_61e65202\comctl32.dll
81e2d8d8 ( 8) 77bb0 77bc4 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\msacm32.dll
81dc9e08 ( 9) 77bd0 77bd7 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\version.dll
81d14f18 ( 7) 77be0 77c37 8 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\msvcrt.dll
81d22688 ( 8) 77d10 77d9f 3 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\user32.dll
81dcc1b8 ( 5) 77da0 77e48 6 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\advapi32.dll
81dcbca0 ( 6) 77e50 77ee2 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\rpcrt4.dll
81e2c630 ( 8) 77ef0 77f39 3 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\gdi32.dll
81e72f78 ( 9) 77f40 77fb5 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\shlwapi.dll
81aa9120 ( 7) 77fc0 77fd0 2 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\secur32.dll
81dccc68 ( 3) 7c800 7c91d 6 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\kernel32.dll
81aac6f0 ( 2) 7c920 7c9b2 5 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\ntdll.dll
81e72fa8 ( 5) 7d590 7dd83 31 Mapped Exe EXECUTE\_WRITECOPY \WINDOWS\system32\shell32.dll
81d212d8 ( 4) 7f6f0 7f7ef 0 Mapped EXECUTE\_READ Pagefile-backed section
81a559a0 ( 3) 7ffa0 7ffd2 0 Mapped READONLY Pagefile-backed section
81aa2538 ( 4) 7ffda 7ffda 1 Private READWRITE
81aa79d0 ( 5) 7ffdf 7ffdf 1 Private READWRITE
Total VADs: 67 average level: 6 maximum depth: 10
```

# 四.Windows物理内存管理

在系统地址空间，有一个区域称为PFN数据库，用于管理系统的物理内存。PFN数据库是一个阵列，或者说是一个数组，每一项都描述了物理页面的状态。全局变量`MnPfnDatabase`是一个指向该数组的指针，以下是该全局变量的输出，可看到该数组的地址是0x80C00000

```php
kd> dd MmPfnDatabase
805630c8 80c00000 ffffffff 00000006 0000003f
805630d8 00000000 00018597 00000005 00000000
805630e8 00000000 00000000 00000004 00000000
805630f8 00000000 00000000 00000000 00000000
80563108 00000000 00000000 00000000 0000001e
80563118 000000fa 0001de80 0001ffff 0001ffff
80563128 0001ff6a 00000040 00000000 7fff0000
80563138 80000000 7ffeffff 00000000 00000000
```

数组中的每一个元素都是一个`MMPFN`结构体，该结构体定义如下：

```php
kd> dt \_MMPFN
nt!\_MMPFN
+0x000 u1 : \_\_unnamed
+0x004 PteAddress : Ptr32 \_MMPTE
+0x008 u2 : \_\_unnamed
+0x00c u3 : \_\_unnamed
+0x010 OriginalPte : \_MMPTE
+0x018 u4 : \_\_unnamed
```

在WRK中的定义如下：

```c
typedef struct \_MMPFN {
union {
PFN\_NUMBER Flink;
WSLE\_NUMBER WsIndex;
PKEVENT Event;
NTSTATUS ReadStatus;
SINGLE\_LIST\_ENTRY NextStackPfn;
} u1;
PMMPTE PteAddress;
union {
PFN\_NUMBER Blink;
ULONG\_PTR ShareCount;
} u2;
union {
struct {
USHORT ReferenceCount;
MMPFNENTRY e1;
};
struct {
USHORT ReferenceCount;
USHORT ShortFlags;
} e2;
} u3;
union {
MMPTE OriginalPte;
LONG AweReferenceCount;
};
union {
ULONG\_PTR EntireFrame;
struct {
ULONG\_PTR PteFrame: 25;
ULONG\_PTR InPageError : 1;
ULONG\_PTR VerifierAllocation : 1;
ULONG\_PTR AweAllocation : 1;
ULONG\_PTR Priority : MI\_PFN\_PRIORITY\_BITS;
ULONG\_PTR MustBeCached : 1;
};
} u4;
} MMPFN, \*PMMPFN;
```

| 名称 | 含义 |
|:---:|:---:|
| `WsIndex` | 包含了该页面在所属工作集链表中的索引。如果这个页面只被一个进程映射，也就是说它是一个私有页面，则`WsIndex`等于该页面在进程工作集链表中的索引。如果该页面被多个进程共享，则该域只保证第一个使该页面变成有效的进程是正确的，通过此域，可用快速地在工作集中定位到它地工作集链表项 |
| `PteAddress` | 指向此页面的PTE的虚拟地址 |
| `ShareCount` | 指向该页面的PET数量，而对于页表页面，该域表示页表中有效PTE和转移PTE的数量。只要`ShareCount`大于0，它就不能从内存中移除 |
| `ReferenceCount` | 代表这个页面必须保留在内存中的引用计数，包括该页面被加入工作集或者I/O的需要而被锁定的次数 |
| `e1` | `MMPFNENTRY`结构，包含了各种标志位 |
| `OriginalPte` | 包含了指向此页面的原始内容，可能是一个PTE。它的用意是，当将一个物理页面分配给一个PTE时，它记录了原来的PTE；以后当该物理页面不再为它使用时，可用恢复其原来的PTE |
| `u4` | 指向该页面的PTE所在的页表页面的物理页帧编号，以及一些标志位 |

其中`MMPFNENTRY`结构定义如下：

```php
kd> dt \_MMPFNENTRY
nt!\_MMPFNENTRY
+0x000 Modified : Pos 0, 1 Bit
+0x000 ReadInProgress : Pos 1, 1 Bit
+0x000 WriteInProgress : Pos 2, 1 Bit
+0x000 PrototypePte : Pos 3, 1 Bit
+0x000 PageColor : Pos 4, 3 Bits
+0x000 ParityError : Pos 7, 1 Bit
+0x000 PageLocation : Pos 8, 3 Bits
+0x000 RemovalRequested : Pos 11, 1 Bit
+0x000 CacheAttribute : Pos 12, 2 Bits
+0x000 Rom : Pos 14, 1 Bit
+0x000 LockCharged : Pos 15, 1 Bit
+0x000 DontUse : Pos 16, 16 Bits
```
