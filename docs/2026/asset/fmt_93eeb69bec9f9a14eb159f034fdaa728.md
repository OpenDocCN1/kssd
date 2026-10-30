
CVE-2018-4990是Adobe在2018年5月修复的一个Adobe DC系列PDF阅读器的0day，原样本配合另一个windows本地提权漏洞CVE-2018-8120一起使用，后者用来进行沙箱逃逸，关于CVE-2018-8120漏洞的分析可以参考小刀师傅写的[《通过对比 5 月补丁分析 win32k 空指针解引用漏洞》](<https://bbs.kanxue.com/elink@1e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5K9h3q4G2k6r3q4G2P5X3S2A6i4K6u0W2j5$3!0E0i4K6u0r3k6i4S2H3L8r3!0A6N6q4)9J5c8U0p5@1z5g2)9J5k6h3S2@1L8h3H3>)一文。

官方公告说这是一个Double Free漏洞，但调试发现这是一个任意地址释放漏洞，原样本中，在32位环境下，攻击者可以通过这个漏洞实现对任意两个4字节地址的释放。原样本漏洞触发前用精准的堆喷射巧妙地布局内存，然后触发漏洞，释放可控的两块大小为0xfff8的相邻堆块。随后，Windows堆分配算法自动将两块空闲的堆块合并成一个大堆块，接着立即重新使用这个大堆块，并利用该堆块的读写能力改写一个`ArrayBuffer`对象的长度为0x66666666，从而实现任意地址读写。

## 影响版本

根据公告，这个漏洞并不影响Adobe Reader 11.x及之前的版本，根据ESET给出的列表，影响的版本如下：

- Acrobat DC (2018.011.20038 and earlier versions)
- Acrobat Reader DC (2018.011.20038 and earlier versions)
- Acrobat 2017 (011.30079 and earlier versions)
- Acrobat Reader DC 2017 (2017.011.30079 and earlier versions)
- Acrobat DC (Classic 2015) (2015.006.30417 and earlier versions)
- Acrobat Reader DC (Classic 2015) (2015.006.30417 and earlier versions)

## 分析环境

本次调试使用的是Adobe Acrobat Reader DC 18.11.20035，`JP2Klib.lib`版本为1.2.2.39492。操作系统为Windows 7 sp1 x86，调试器为Windbg。

## 全页堆下的crash分析

拿到样本，我们要做的第一件事就是定位漏洞触发点，先开启全页堆，打开样本后发现crash，现场如下：

```
First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

eax=d0d0d0b0 ebx=00000000 ecx=d0d0d000 edx=d0d0d0b0 esi=01930000 edi=01930000

eip=72676e88 esp=001ba2e0 ebp=001ba32c iopl=0         nv up ei ng nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010286

verifier!AVrfpDphFindBusyMemoryNoCheck+0xb8:

72676e88 813abbbbcdab    cmp     dword ptr [edx],0ABCDBBBBh ds:0023:d0d0d0b0=????????

0:000> kv 20

ChildEBP RetAddr  Args to Child

001ba32c 72676f95 01931000 d0d0d0d0 01930000 verifier!AVrfpDphFindBusyMemoryNoCheck+0xb8 (FPO: [SEH])

001ba350 72677240 01931000 d0d0d0d0 001ba3c0 verifier!AVrfpDphFindBusyMemory+0x15 (FPO: [2,5,0])

001ba36c 72679080 01931000 d0d0d0d0 0090d911 verifier!AVrfpDphFindBusyMemoryAndRemoveFromBusyList+0x20 (FPO: [2,3,0])

001ba388 779169cc 01930000 01000002 d0d0d0d0 verifier!AVrfDebugPageHeapFree+0x90 (FPO: [3,3,0])

001ba3d0 778d9e07 01930000 01000002 d0d0d0d0 ntdll!RtlDebugFreeHeap+0x2f (FPO: [SEH])

001ba4c4 778a63a6 00000000 d0d0d0d0 51b6cf98 ntdll!RtlpFreeHeap+0x5d (FPO: [SEH])

001ba4e4 7708c614 01930000 00000000 d0d0d0d0 ntdll!RtlFreeHeap+0x142 (FPO: [3,1,4])

001ba4f8 6f0cecfa 01930000 00000000 d0d0d0d0 kernel32!HeapFree+0x14 (FPO: [3,0,0])

001ba50c 67eb0574 d0d0d0d0 0ee62d74 52148fac MSVCR120!free+0x1a (FPO: [Non-Fpo]) (CONV: cdecl) [f:\dd\vctools\crt\crtw32\heap\free.c @ 51]

WARNING: Stack unwind information not available. Following frames may be wrong.

001ba62c 67ec6482 5383cfb8 51b6afd8 000000fd JP2KLib!JP2KCopyRect+0xbae6

001ba684 64e56cfc 5387ae88 53874fd0 51b6afd8 JP2KLib!JP2KImageInitDecoderEx+0x24

001ba70c 64e58696 53834fa8 52148fac 53834fa8 AcroRd32_64860000!AX_PDXlateToHostEx+0x261843

001ba76c 64e4d785 52148fac 001ba78c 64e56640 AcroRd32_64860000!AX_PDXlateToHostEx+0x2631dd

001ba778 64e56640 52148fac 53056f70 53050fc8 AcroRd32_64860000!AX_PDXlateToHostEx+0x2582cc

001ba78c 64a4030d 52148fac 53050fd0 53050fc8 AcroRd32_64860000!AX_PDXlateToHostEx+0x261187

001ba7c8 64a3f92b c0010000 00000016 53050fc8 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x7867d

001ba898 64a3ebc6 001bac40 00000000 1065768e AcroRd32_64860000!PDMediaQueriesGetCosObj+0x77c9b

001babe8 64a3eb88 001bac40 4ee49a50 1065717a AcroRd32_64860000!PDMediaQueriesGetCosObj+0x76f36

001bac1c 64a3ea71 53050e28 4ee49a50 001bacd4 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x76ef8

001bac88 64a3d949 c0010000 00000016 4ee49a50 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x76de1

001bb0f4 64a3ade9 001bb3e0 51ae2598 c0010000 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x75cb9

001bc8cc 64a3aa51 51ae2598 c0010000 00000016 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x73159

001bc9a4 64a24ab0 10651702 00000000 4ee49a50 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x72dc1

001bca64 64a844b4 00000000 00000000 00000000 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x5ce20

001bcac0 64a3d2d3 00000000 00000000 00000000 AcroRd32_64860000!CTJPEGDecoderReadNextTile+0x2dea4

001be28c 64a3aa51 51ae250c c0010000 00000015 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x75643

001be364 64a24ab0 10653942 5213ef78 00000000 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x72dc1

001be424 64a22e70 00000001 00000000 00000000 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x5ce20

001be46c 64a13d93 5213ef78 00000001 00000000 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x5b1e0

001be5cc 64a134fa 5360adbc 00000001 0000002d AcroRd32_64860000!PDMediaQueriesGetCosObj+0x4c103

001be634 64ac4a44 10653bd2 00000000 001be6d4 AcroRd32_64860000!PDMediaQueriesGetCosObj+0x4b86a

001be6b4 64ac4751 4e180ef8 c0010000 0000002d AcroRd32_64860000!CTJPEGDecoderReadNextTile+0x6e434

```

看过《Windows高级调试》第6章页堆部分的同学应该已经明白了，free地址为0xd0d0d0d0的原因是释放了全页堆的后置填充区域。

![](https://bbs.kanxue.com/upload/attach/201805/628056_BQGGM9CF3FPG3H8.png)

到这里，有经验的同学应该已经可以猜到是因为堆内存的越界访问导致释放了堆的后置填充数据，从上面的栈回溯我们可以看到这是`JP2Klib`动态库的一处调用导致的，如下：

```
001ba62c 67ec6482 5383cfb8 51b6afd8 000000fd JP2KLib!JP2KCopyRect+0xbae6
```

我们在IDA里面看一下相应偏移处的代码逻辑：

```
count1 = 0;
count2 = 0;

if ( *(v116 + 4) > 0 )
{
    do
    {
        if ( *(*(*(pObj + 0x48) + 0xC) + 4 * count1) )
        {
          sub_66FEA(*(*(*(pObj + 0x48) + 0xC) + 4 * count1)); // 从这里进入后面的free()
          count1 = count2;
          *(*(*(pObj + 0x48) + 0xC) + 4 * count2) = 0;
        }
        count2 = ++count1;
    }while ( count1 < *(*(pObj + 0x48) + 4) );
}
```

将上述代码整理一下后可以更清楚地看到逻辑：

```
count = 0;

if ( *(v116 + 4) > 0 )
{
    do
    {
        if ( *(mem_base + 4 * count) )
        {
            free(*(mem_base + 4 * count));
            *(mem_base + 4 * count) = 0;
        }
        count++;
    }while ( count < max_count );
}
```

## 漏洞的根本原因

我们来监控一下`max_count`和`while`循环里的`count`变化过程

```
bp JP2KLib+50588 "dd eax+4 l1; g;"  // 获取max_count值

bp JP2KLib+50567 "r eax; r ecx; g;" // 获取mem基地址和每次循环的count值

bp JP2KLib+5056e "r eax; g;"        // 获取每次free()的地址
```

![](https://bbs.kanxue.com/upload/attach/201805/628056_8U4T8THUDNMUNJD.png)

利用`PdfStreamDumper.exe`对pdf的修改能力，我们对内嵌的javascript代码稍作修改，放开原代码的一处注释，目的是在漏洞触发前弹个对话框，便于我们下断点：

![](https://bbs.kanxue.com/upload/attach/201805/628056_QDNN53C8KYQH4DW.png)

```
0:000> bp JP2KLib+50588 "dd eax+4 l1; g;"
0:000> bp JP2KLib+50567 "r eax; r ecx; g;"
0:000> bp JP2KLib+5056e "r eax; g;"

0:000> g
eax=073fa1a0
ecx=00000000
17bf44d4  000000ff

eax=073fa1a0
ecx=00000001
17bf44d4  000000ff

...略去无关输出...

eax=073fa1a0
ecx=000000fd
eax=0d0e0048
17bf44d4  000000ff

eax=073fa1a0
ecx=000000fe
eax=0d0f0048
17bf44d4  000000ff
```

可以看到`mem_base`为 073fa1a0，`max_count = 0xff`，当`count`变化到0xfd和0xfe时两个在原js中出现的堆地址 0x0d0e0048 和 0x0d0f0048 被释放。

此时我们对`mem_base`的分配大小很感兴趣，重启windbg，观察一下`mem_base`的大小，可以看到`mem_base`的大小为0x3f4

```
0:000> !heap -p -a 0aaf5060

    address 0aaf5060 found in

    _HEAP @ 880000

      HEAP_ENTRY Size Prev Flags    UserPtr UserSize - state

        0aaf5048 0089 0000  [00]   0aaf5060    003f4 - (busy)

        Trace: 1dd977d4

        7707dd6c ntdll!RtlAllocateHeap+0x00000274

        694ded63 MSVCR120!malloc+0x00000049

        674b6f34 JP2KLib!JP2KTileGeometryRegionIsTile+0x00000102

        674912cb JP2KLib!JP2KCodeStm::write+0x00017dcb

        67490978 JP2KLib!JP2KCodeStm::write+0x00017478

        6749f746 JP2KLib!JP2KCopyRect+0x0000acb8

        674b6482 JP2KLib!JP2KImageInitDecoderEx+0x00000024
```

到这里漏洞的成因已经很清楚了，漏洞根源在于`max_count = 0xff`，而`ff*4 = 3fc`，所以`while`循环可以访问到`mem_base ~ mem_base+3fc`区间的内存。但是我们看到`mem_base`对应处的内存大小只有0x3f4，两者的差值为8个字节（`3fc - 3f4 = 8`），于是可以借助上述`while`循环越界访问两个4字节地址并释放，来实现任意释放两个地址。

于是攻击者可以通过内存布局(例如堆喷射)提供的任意两个4字节地址，并实现任意释放，如下是样本中越界释放的两个堆地址，他们通过堆喷射被精确布控到相邻的内存。

```
0:000> dd 0aaf5060+3f4 l2

0aaf5454  0d0e0048 0d0f0048
```

所以这本质上不是一个double free漏洞，而是任意地址释放漏洞(原样本在32位下可以释放两个任意地址)。

## 堆喷射占坑+OOB释放
