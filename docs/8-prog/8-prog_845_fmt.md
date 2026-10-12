
# CPUID 详解（增加 TLB 与 Cache）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-158.htm`](https://www.pediy.com/kssd/pediy08/pediy8-158.htm)

这是文章最后一次更新，加入了 `TLB` 与 `Cache` 信息等资料。

前言：论坛上面有人不明白 `CPUID` 指令的用法，于是就萌生写这篇文章的想法。

若有错误请大侠指出，谢谢了 ^^

论坛的式样貌似有问题，若式样问题导致阅读问题的话，可以在文章尾下载 `txt` 文档阅读。

`CPUID` 是 `Intel Pentium` 以上级 `CPU` 内置的一个指令（`486` 级及以下的 `CPU` 不支持），它用于识别某一类型的 `CPU`，它能返回 `CPU` 的级别（`family`），型号（`model`），`CPU` 步进（`Stepping ID`）及 `CPU` 字串等信息，从此命令也可以得到 `CPU` 的缓存与 `TLB` 信息。

`CPUID` 返回数据类型是在 `EAX` 寄存器里面定义的，而指令返回的数值则存储在 `EAX`、`EBX`、`ECX` 和 `EDX` 寄存器里面。

返回的信息分两部分：基本信息与扩展信息。在 `EAX` 输入 `0-3` 参数时，它返回的 `CPU` 的基本信息；而在 `EAX` 输入 `0x8000000` 至 `0x800000x` 时，它返回的是 `CPU` 的扩展信息（`extended function information`）。扩展信息只包含在 `Pentium 4` 及以后的 `CPU` 上，`Pentium 4` 以前的 `CPU` 无法取得它的扩展信息。

如下面的表：

| CPU 级别 | 基本信息 | 扩展信息 |
| :--- | :--- | :--- |
| `486` 及以前的 `CPU` | 不可用 | 不可用 |
| `Pentium` | `0x1` | 不可用 |
| `Pentium Pro`, `Pentium 2` | `0x2` | 不可用 |
| `Pentium 3` | `0x3` | 不可用 |
| `Pentium 4` | `0x2` | `0x80000004` |
| `Xeon` (至强) | `0x2` | `0x80000004` |

假若输入高于该处理器的值时，`CPUID` 指令返回的是该 `CPU` 的输入最高值的返回值（这一句不知道怎么说才好）。

比如在在 `Pentium 4` 上输入 `0x4`，则 `CPU` 返回值与输入 `0x2` 的返回值一样。

下面的表是输入值与返回值的关系：

| 输入值 | 返回值 |
| :--- | :--- |
| `0x0` | `EAX` `CPU` 基本参数的输入值<br>`EBX` "Genu"<br>`ECX` "Intel"<br>`EDX` "inel" |
| `0x1` | `EAX` `CPU` 的级别，型号及步进<br>`EBX` 信息很多，下面介绍<br>`ECX` 保留<br>`EDX` 特征信息 (`Feature Information`) |
| `0x2` | `EAX` 到 `EDX` 返回的都是缓存和 `TLB` 的信息 |
| `0x3` | `EAX` 保留<br>`EBX` 保留<br>`ECX` `CPU` 序列号 (`0 - 31bit`) (只是在 `Pentium 3` 中才有效)<br>`EDX` `CPU` 序列号 (`32 - 63bit`) (只是在 `Pentium 3` 中才有效) |
| `0x80000000` | `EAX` 扩展信息输入数最大值 (具有扩展信息的 `CPU` 才能返回)<br>`EBX` 保留<br>`ECX` 保留<br>`EDX` 保留 |
| `0x80000001` | `EAX` `CPU` 特征 (`Signature`) 和扩展特征位 (`Extended Feature Bits`)<br>`EBX` 到 `ECX` 保留 |
| `0x80000002` | `EAX` 处理器字串 (`Processor Brand String`)<br>`EBX` 处理器字串 (续)<br>`ECX` 处理器字串 (续)<br>`EDX` 处理器字串 (续) |
| `0x80000003` | `EAX` 处理器字串 (续)<br>`EBX` 处理器字串 (续)<br>`ECX` 处理器字串 (续)<br>`EDX` 处理器字串 (续) |
| `0x80000004` | `EAX` 处理器字串 (续)<br>`EBX` 处理器字串 (续)<br>`ECX` 处理器字串 (续)<br>`EDX` 处理器字串 (续) |

当输入 `0x1` 时，`EBX` 返回值是：

- 第 `0` - `7` 位：`CPU` 字串索引 (`Brand Index`)
- 第 `8` - `15` 位：`CLFLUSH` 线大小 (`CLFLUSH line size`) (`返回值 * 8 = cache line size`)
- 第 `16` - `23` 位：保留
- 第 `24` - `31` 位：处理器 `APIC` 物理标号 (`Processor local APIC physical ID`)

当输入 `0x1` 时，`EDX` 返回的扩展信息解释如下：

| 位 | 标号 | 解释 |
| :--- | :--- | :--- |
| `0` | `FPU` | `Floating Point Unit On-Chip`. `CPU` 是否内置浮点计算单元 |
| `1` | `VME` | `Virtual 8086 Mode Enhancements`. 是否支持虚拟 `8086` 模式 |
| `2` | `DE` | `Debugging Extensions`. 是否支持调试功能. |
| `3` | `PSE` | `Page Size Extension`. 是否支持大于 `4MB` 的分页. |
| `4` | `TSC` | `Time Stamp Counter`. 是否支持 `RDTSC` 指令. (注：`RDTSC` 指令可以计算出 `CPU` 的频率) |
| `5` | `MSR` | `Module Specific Registers RDMSR and WRMSR Instructions`. 是否支持 `RDMSR` 与 `WRMSR` (\*注 1) |
| `6` | `PAE` | `Physical Address Extension`. 是否支持大于 `32bit` 的物理地址. |
| `7` | `MCE` | `Machine Check Exception`. (\*注 2) |
| `8` | `CX8` | `CMPXCHG8B Instruction`. 是否支持 `8bytes` (`64bit`) 数的比较与交换指令. |
| `9` | `APIC` | `APIC On-Chip`. 是否支持 `APIC` (`Advanced Programmable Interrupt Controller`) |
| `10` | 保留 | |
| `11` | `SEP` | `SYSENTER and SYSEXIT Instructions`. 是否支持 `SYSENTER` 与 `SYSEXIT` 指令. (\*注 3) |
| `12` | `MTRR` | `Memory Type Range Registers`. 是否支持 `MTTR` (\*注 4) |
| `13` | `PGE` | `PTE Global Bit`. 是否支持全局页面目录入口标志位 (`global bit in page directory entries`) |
| `14` | `MCA` | `Machine Check Architecture`. 是否支持 `MCA`，`MCA` 是 `Pentium4`, `Xeon`, `P6` 级处理器的一个错误报告机制 |
| `15` | `CMOV` | `Conditional Move Instructions`. `CMOV` 指令是否可用. (请问谁可以解释一下 `CMOV` 是什么命令?) |
| `16` | `PAT` | `Page Attribute Table`. 是否支持 `PAT`，`PAT` 允许操作系统指定 `4K` 大小的线性内存空间 |
| `17` | `PSE-36` | `32-bit Page Size Extension`. 是否支持 `4GB` 的扩展内存 |
| `18` | `PSN` | `Processor Serial Number`. 是否支持处理器序列号. (`P3` 有效) |
| `19` | `CLFSH` | `CLFLUSH Instruction`. 是否支持 `CLFLUSH`. (\*注 5) |
| `20` | 保留 | |
| `21` | `DS` | `Debug Store`. 是否支持把调试信息写入缓存, |
| `22` | `ACPI` | `ACPI Processor Performance Modulation Registers`. 处理器使用特别的寄存器以允许软件控制处理器的运行周期. |
| `23` | `MMX` | `Inter MMX Technology`. 是否支持 `MMX` |
| `24` | `FXSR` | `FXSAVE and FXRSTOR Instructions`. `FXSAVE` 与 `FXRSTOR` 指令是否可用 (\*注 6) |
| `25` | `SSE` | `SSE`. 是否支持 `SSE`. |
| `26` | `SSE2` | 是否支持 `SSE2`. |
| `27` | `SS` | `Self Snoop`. 处理器是否支持总线监视，以防止储存器冲突. |
| `28` | 保留 | |
| `29` | `TM` | `Thermal Monitor`. `CPU` 是否支持温度控制. |
| `30` & `31` | 保留 | |

注 1: `RDMSR`: `Load MSR specified by ECX into EDX:EAX`<br>`WRMSR`: `Write the value in EDX:EAX to MSR specified by ECX`

注 2:
原文是 `Exception 18 is defined for Machine Checks, including CR4.MCE for controlling the feature. This feature does not define the model-specific implementations of machine-check error logging, reporting, and processor shutdowns. Machine Check exception handlers may have to depend on processor version to do model specific processing of the exception, or test for the presence of the Machine Check feature.`

注 3: `SYSENTER`: `Fast call to privilege level 0 system procedures`<br>`SYSEXIT`: `Fast return to privilege level 3 user code.`

注 4:
原文是 `The MTRRcap MSR contains feature bits that describe what memory types are supported, how many variable MTRRs are supported, and whether fixed MTRRs are supported.`

注 5: `CLFLUSH`: `Flushes cache line containing m8.`

注 6: `FXSAVE`: `Save the x87 FPU, MMX, XMM, and MXCSR register`<br>`FXSTOR`: `Restore the x87 FPU, MMX, XMM, and MXCSR register`

按照这个，就可以自己写一个 `CPU` 检测程序了；

```c
#include <stdio.h>

void main()
{
  unsigned long DBaseIndex, DFeInfo, DFeInfo2, DCPUBaseInfo;
  unsigned long DFeIndex, DCPUExInfo, i;
  unsigned long DOther[4], DTLB[4], DProceSN[2];
  char cCom[13];
  char cProStr[49];
  unsigned int j;

  _asm
  {
    xor eax, eax
    cpuid
    mov DBaseIndex      ,eax
    mov dword ptr cCom    ,ebx
    mov dword ptr cCom+4  ,ecx //AMD CPU 要把 ecx 改为 edx
    mov dword ptr cCom+8  ,edx //AMD CPU 要把 edx 改为 ecx
    mov eax, 1
    cpuid
    mov DCPUBaseInfo, eax
    mov DFeInfo, ebx
    mov DFeInfo2, edx
    mov eax, 0x80000000
    cpuid
    mov DFeIndex, eax
    mov eax, 0x80000001
    cpuid
    mov DCPUExInfo, eax
    mov eax, 0x80000002
    cpuid
    mov dword ptr cProStr    , eax
    mov dword ptr cProStr + 4  , ebx
    mov dword ptr cProStr + 8  , ecx
    mov dword ptr cProStr + 12  ,edx
    mov eax, 0x80000003
    cpuid
    mov dword ptr cProStr + 16  , eax
    mov dword ptr cProStr + 20  , ebx
    mov dword ptr cProStr + 24  , ecx
    mov dword ptr cProStr + 28  , edx
    mov eax, 0x80000004
    cpuid
    mov dword ptr cProStr + 32  , eax
    mov dword ptr cProStr + 36  , ebx
    mov dword ptr cProStr + 40  , ecx
    mov dword ptr cProStr + 44  , edx
  }

  if( DBaseIndex >= 2 )
  {
    _asm
    {
      mov eax, 2
      cpuid
      mov DTLB[0], eax
      mov DTLB[2], ebx
      mov DTLB[3], ecx
      mov DTLB[4], edx
    }
  }

  if(DBaseIndex == 3)
  {
    _asm
    {
      mov eax, 3
      cpuid
      mov DProceSN[0], ecx
      mov DProceSN[1], edx
    }
  }

  cCom[12] = '\0'; //加一个结尾符
  printf( "CPU 厂商:  %s\n", cCom );
  printf( "CPU 字串:  %s\n", cProStr );
  printf( "CPU 基本参数: Family:%X  Model:%X  Stepping ID:%X\n", (DCPUBaseInfo & 0x0F00) >> 8,
    (DCPUBaseInfo & 0xF0) >> 4, DCPUBaseInfo & 0xF );
  printf( "CPU 扩展参数: Family:%X  Model:%X  Stepping ID:%X\n", (DCPUExInfo & 0x0F00) >> 8,
    (DCPUExInfo & 0xF0) >> 4, DCPUExInfo & 0xF );
}
```



```c
  printf( "CPU 字串索引: 0x%X\n", DFeInfo & 0xFF );  

  printf( "CLFLUSH 线大小: 0x%X\n", ( DFeInfo & 0xFF00 ) >> 8 );  

  printf ( "处理器 APIC 物理标号:0x%X\n", ( DFeInfo & 0xF000 ) >> 24 );  

  if( DBaseIndex >= 2)  

  {  

    printf( "CPU Cache & TLB Information: " );  

    for(j = 0; j < 4; j++)  

    {  

      if( !(DTLB[j] & 0xFF) ) printf( "%.2X ", DTLB[j] & 0xFF );  

      if( !((DTLB[j] & 0xFF) >> 8) ) printf( "%.2X ", (DTLB[j] & 0xFF00) >> 8 );  

      if( !((DTLB[j] & 0xFF0000) >> 16) ) printf( "%.2X ",( DTLB[j] & 0xFF0000) >> 16);  

      if( !((DTLB[j] & 0xFF000000) >> 24) ) printf( "%.2X ",( DTLB[j] & 0xFF000000) >> 24);  

    }  

    printf("\n");  

  }  

  if( DBaseIndex == 3 )  

  {  

    printf( "CPU 序列号是:%X%X\n", DProceSN[0], DProceSN[1] );  

  }  

    printf( "FPU:  %d\t\t", DFeInfo2 & 0x00000001 ); //下面是调用某 BLOG 上面的代码,懒得写了 ^^  

    printf( "VME:  %d\t\t", (DFeInfo2 & 0x00000002 ) >> 1 );  

    printf( "DE:  %d\n", (DFeInfo2 & 0x00000004 ) >> 2 );  

    printf( "PSE:  %d\t\t", (DFeInfo2 & 0x00000008 ) >> 3 );  

    printf( "TSC:  %d\t\t", (DFeInfo2 & 0x00000010 ) >> 4 );  

    printf( "MSR:  %d\n", (DFeInfo2 & 0x00000020 ) >> 5 );  

    printf( "PAE:  %d\t\t", (DFeInfo2 & 0x00000040 ) >> 6 );  

    printf( "MCE:  %d\t\t", (DFeInfo2 & 0x00000080 ) >> 7 );  

    printf( "CX8:  %d\n", (DFeInfo2 & 0x00000100 ) >> 8 );  

    printf( "APIC:  %d\t", (DFeInfo2 & 0x00000200 ) >> 9 );  

    printf( "SEP:  %d\t\t", (DFeInfo2 & 0x00000800 ) >> 11 );  

    printf( "MTRR:  %d\n", (DFeInfo2 & 0x00001000 ) >> 12 );  

    printf( "PGE:  %d\t\t", (DFeInfo2 & 0x00002000 ) >> 13 );  

    printf( "MCA:  %d\t\t", (DFeInfo2 & 0x00004000 ) >> 14 );  

    printf( "CMOV:  %d\n", (DFeInfo2 & 0x00008000 ) >> 15 );  

    printf( "PAT:  %d\t\t", (DFeInfo2 & 0x00010000 ) >> 16 );  

    printf( "PSE-36:  %d\t", (DFeInfo2 & 0x00020000 ) >> 17 );  

    printf( "PSN:  %d\n", (DFeInfo2 & 0x00040000 ) >> 18 );  

    printf( "CLFSN:  %d\t", (DFeInfo2 & 0x00080000 ) >> 19 );  

    printf( "DS:  %d\t\t", (DFeInfo2 & 0x00200000 ) >> 21 );  

    printf( "ACPI:  %d\n", (DFeInfo2 & 0x00400000 ) >> 22 );  

    printf( "MMX:  %d\t\t", (DFeInfo2 & 0x00800000 ) >> 23 );  

    printf( "FXSR:  %d\t", (DFeInfo2 & 0x01000000 ) >> 24 );  

    printf( "SSE:  %d\n", (DFeInfo2 & 0x02000000 ) >> 25 );  

    printf( "SSE2:  %d\t", (DFeInfo2 & 0x04000000 ) >> 26 );  

    printf( "SS:  %d\t\t", (DFeInfo2 & 0x08000000 ) >> 27 );  

    printf( "TM:  %d\n", (DFeInfo2 & 0x20000000 ) >> 29 );  

  printf("\n 其它信息:\n");  

  printf("----------------------------------------\n");  

  printf("In \t\tEAX \t\tEBX \t\tECX \t\tEDX");  

  for( i = 0x80000004; i <= DFeIndex; ++i )  

  {  

    DOther[0] = DOther[1] = DOther[2] = DOther[3] = 0;  

    _asm  

    {  

      mov eax, i  

      cpuid  

      mov DOther[0], eax  

      mov DOther[1], ebx  

      mov DOther[2], ecx  

      mov DOther[3], edx  

    }  

    printf( "\n0x%.8X\t0x%.8X\t0x%.8X\t0x%.8X\t0x%.8X", i, DOther[0], DOther[1], DOther[2], DOther[3] );  

  }  

  printf( "\n" );  

  system( "pause" );  

}
```

### TLB 与 Cache 信息详解

*   `0x00` Null descriptor
*   `0x01` Instruction TLB: 4K-Byte Pages, 4-way set associative, 32 entries
*   `0x02` Instruction TLB: 4M-Byte Pages, 4-way set associative, 2 entries
*   `0x03` Data TLB: 4K-Byte Pages, 4-way set associative, 64 entries
*   `0x04` Data TLB: 4M-Byte Pages, 4-way set associative, 8 entries
*   `0x06` 1st-level instruction cache: 8K Bytes, 4-way set associative, 32 byte line size
*   `0x08` 1st-level instruction cache: 16K Bytes, 4-way set associative, 32 byte line size
*   `0x0A` 1st-level data cache: 8K Bytes, 2-way set associative, 32 byte line size
*   `0x0C` 1st-level data cache: 16K Bytes, 4-way set associative, 32 byte line size
*   `0x22` 3rd-level cache: 512K Bytes, 4-way set associative, 64 byte line size
*   `0x23` 3rd-level cache: 1M Bytes, 8-way set associative, 64 byte line size
*   `0x25` 3rd-level cache: 2M Bytes, 8-way set associative, 64 byte line size
*   `0x29` 3rd-level cache: 4M Bytes, 8-way set associative, 64 byte line size
*   `0x40` No 2nd-level cache or, if processor contains a valid 2nd-level cache, no 3rd-level cache
*   `0x41` 2nd-level cache: 128K Bytes, 4-way set associative, 32 byte line size
*   `0x42` 2nd-level cache: 256K Bytes, 4-way set associative, 32 byte line size
*   `0x43` 2nd-level cache: 512K Bytes, 4-way set associative, 32 byte line size
*   `0x44` 2nd-level cache: 1M Byte, 4-way set associative, 32 byte line size
*   `0x45` 2nd-level cache: 2M Byte, 4-way set associative, 32 byte line size
*   `0x50` Instruction TLB: 4-KByte and 2-MByte or 4-MByte pages, 64 entries
*   `0x51` Instruction TLB: 4-KByte and 2-MByte or 4-MByte pages, 128 entries
*   `0x52` Instruction TLB: 4-KByte and 2-MByte or 4-MByte pages, 256 entries
*   `0x5B` Data TLB: 4-KByte and 4-MByte pages, 64 entries
*   `0x5C` Data TLB: 4-KByte and 4-MByte pages, 128 entries
*   `0x5D` Data TLB: 4-KByte and 4-MByte pages, 256 entries
*   `0x66` 1st-level data cache: 8KB, 4-way set associative, 64 byte line size
*   `0x67` 1st-level data cache: 16KB, 4-way set associative, 64 byte line size
*   `0x68` 1st-level data cache: 32KB, 4-way set associative, 64 byte line size
*   `0x70` Trace cache: 12K-μop, 8-way set associative
*   `0x71` Trace cache: 16K-μop, 8-way set associative
*   `0x72` Trace cache: 32K-μop, 8-way set associative
*   `0x79` 2nd-level cache: 128KB, 8-way set associative, sectored, 64 byte line size
*   `0x7A` 2nd-level cache: 256KB, 8-way set associative, sectored, 64 byte line size
*   `0x7B` 2nd-level cache: 512KB, 8-way set associative, sectored, 64 byte line size
*   `0x7C` 2nd-level cache: 1MB, 8-way set associative, sectored, 64 byte line size
*   `0x82` 2nd-level cache: 256K Bytes, 8-way set associative, 32 byte line size
*   `0x84` 2nd-level cache: 1M Byte, 8-way set associative, 32 byte line size
*   `0x85` 2nd-level cache: 2M Byte, 8-way set associative, 32 byte line size

### 示例

举一个例子，一个 CPU 执行：

```asm
mov eax, 2
cpuid
```

后，返回值如下：

```
EAX 0x665B5001
EBX 0x0
ECX 0x0
EDX 0x007A7000
```

就代表了这个 CPU 的 Cache 信息是：

1. `0x66`: 1st-level data cache: 8KB, 4-way set associative, 64 byte line size
2. `0x5B`: Data TLB: 4-KByte and 4-MByte pages, 64 entries
3. `0x50`: Instruction TLB: 4-KByte and 2-MByte or 4-MByte pages, 64 entries
4. `0x01`: Instruction TLB: 4K-Byte Pages, 4-way set associative, 32 entries
5. `0x7A`: 2nd-level cache: 256KB, 8-way set associative, sectored, 64 byte line size
6. `0x70`: Trace cache: 12K-μop, 8-way set associative

### CPU 字串索引 (Brand Index) 详解

*   `0x0`: CPU 不支持 `Brand Index`
*   `0x1`: `Celeron` CPU
*   `0x2`: `Pentium 3`
*   `0x3`: `Pentium 3 Xeon`
*   `0x4`-`0x7`: 未用
*   `0x8`: `Pentium 4`

可惜我是用 `AMD` CPU 的，有好些功能都不能亲自试验

参考资料：`IA-32 Intel Architecture Software Developer’s Manual`

`4dd466d0e1e0cc123afbe504da2462ca`
