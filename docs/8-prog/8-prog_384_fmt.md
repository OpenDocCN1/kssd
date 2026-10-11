
# Rootkit 直接访问硬件之 [二]

> 原文：`www.pediy.com/kssd/pediy10/62061.html`

关于 `ring3` 访问硬件，今天我们将采用另外的方法来进行 I/O 访问。为了找出方法，我们还是要继续看看 I/O 保护的八条规则：呵呵，正所谓“知己知彼，百战不殆。“  

(1) 若 `CPL` <= `IOPL`，则直接转步骤(8)；  

(2) 取得 I/O 位图开始偏移；  

(3) 计算 I/O 地址对应位所在字节在 I/O 许可位图内的偏移；  

(4) 计算位偏移以形成屏蔽码值，即计算 I/O 地址对应位在字节中的第几位；  

(5) 把字节偏移加上位图开始偏移，再加 1，所得值与 `TSS` 界限比较，若越界，则产生出错码为 0 的通用保护故障；  

(6) 若不越界，则从位图中读对应字节及下一个字节；  

(7) 把读出的两个字节与屏蔽码进行与运算，若结果不为 0 表示检查未通过，则产生出错码为 0 的通用保护故障；  

(8) 进行 I/O 访问。  

上一篇中我们已经介绍了修改 `IOPL = 3`，使其顺利通过保护规则。因此八条规则中的第一条我们已经用过了，今天我们从第二条开始找办法。  

我们先看看 I/O 许可位图，它位于任务状态段 `TSS` 中。要找出 `tss` 段所在的线性地址，需要先从 `tr` 寄存器中，找出任务状态段的选择子，根据这个选择子我们可以在 `GDT` 中找到任务状态段描述符，根据这个描述符，我们可以找出 `TSS` 在内存中的位置。`TSS` 是保存一个任务重要信息的特殊段。我们先看看其内存结构。  

```
lkd> dt _ktss

nt!_KTSS

   +0x000 Backlink          : Uint2B
   +0x002 Reserved0         : Uint2B
   +0x004 Esp0              : Uint4B
   +0x008 Ss0               : Uint2B
   +0x00a Reserved1         : Uint2B
   +0x00c NotUsed1          : [4] Uint4B
   +0x01c CR3               : Uint4B
   +0x020 Eip               : Uint4B
   +0x024 EFlags            : Uint4B
   +0x028 Eax               : Uint4B
   +0x02c Ecx               : Uint4B
   +0x030 Edx               : Uint4B
   +0x034 Ebx               : Uint4B
   +0x038 Esp               : Uint4B
   +0x03c Ebp               : Uint4B
   +0x040 Esi               : Uint4B
   +0x044 Edi               : Uint4B
   +0x048 Es                : Uint2B
   +0x04a Reserved2         : Uint2B
   +0x04c Cs                : Uint2B
   +0x04e Reserved3         : Uint2B
   +0x050 Ss                : Uint2B
   +0x052 Reserved4         : Uint2B
   +0x054 Ds                : Uint2B
   +0x056 Reserved5         : Uint2B
   +0x058 Fs                : Uint2B
   +0x05a Reserved6         : Uint2B
   +0x05c Gs                : Uint2B
   +0x05e Reserved7         : Uint2B
   +0x060 LDT               : Uint2B
   +0x062 Reserved8         : Uint2B
   +0x064 Flags             : Uint2B
   +0x066 IoMapBase         : Uint2B
   +0x068 IoMaps            : [1] _KiIoAccessMap
   +0x208c IntDirectionMap  : [32] UChar

lkd> dt _KiIoAccessMap

nt!_KiIoAccessMap

   +0x000 DirectionMap      : [32] UChar
   +0x020 IoMap            : [8196] UChar
```

根据这个结构我们可以计算出其长度为：`0x208c + 32 - 1 = 0x20ab`

其中的 `IoMapBase` 就是 I/O 许可位图在 `TSS` 段中的偏移位置。也就是说在 `TSS` 段中从这个位置开始就是 I/O 许可位图了。  

对于本机操作系统，我们可以通过 `WINDBG` 察看下该结构的值。  

首先我们找到 `TSS` 的内存位置，我们可以通过如下指令得到 `TSS`: `80042000`

```
lkd> !PCR

KPCR for Processor 0 at ffdff000:

    Major 1 Minor 1
  NtTib.ExceptionList: b2616c7c
      NtTib.StackBase: b2616df0
     NtTib.StackLimit: b2614000
   NtTib.SubSystemTib: 00000000
        NtTib.Version: 00000000
    NtTib.UserPointer: 00000000
        NtTib.SelfTib: 7ffde000
              SelfPcr: ffdff000
                 Prcb: ffdff120
                 Irql: 00000000
                  IRR: 00000000
                  IDR: ffffffff
        InterruptMode: 00000000
                  IDT: 8003f400
                  GDT: 8003f000
                  TSS: 80042000
        CurrentThread: 88fb6da8
           NextThread: 00000000
           IdleThread: 80552d20
            DpcQueue: 
```

接下来，我们看看这个结构中每一项的实际值。  

```
lkd> dt _ktss 80042000

nt!_KTSS

   +0x000 Backlink          : 0
   +0x002 Reserved0         : 0x838d
   +0x004 Esp0              : 0xb1da5de0
   +0x008 Ss0               : 0x10
   +0x00a Reserved1         : 0xb9fc
   +0x00c NotUsed1          : [4] 0x1000
   +0x01c CR3               : 0x12640740
   +0x020 Eip               : 0x14453b02
   +0x024 EFlags            : 0x8a870f
   +0x028 Eax               : 0xc18b0000
   +0x02c Ecx               : 0x8d02e9c1
   +0x030 Edx               : 0xfb8b0272
   +0x034 Ebx               : 0xc88ba5f3
   +0x038 Esp               : 0xf303e183
   +0x03c Ebp               : 0xc5d03a4
   +0x040 Esi               : 0x3bf8558b
   +0x044 Edi               : 0x7174fc5d
   +0x048 Es                : 0x458b
   +0x04a Reserved2         : 0x8314
   +0x04c Cs                : 0xfec0
   +0x04e Reserved3         : 0xd03b
   +0x050 Ss                : 0x6777
   +0x052 Reserved4         : 0xc2f6
   +0x054 Ds                : 0x7401
   +0x056 Reserved5         : 0x8a0d
   +0x058 Fs                : 0x8802
   +0x05a Reserved6         : 0x1045
   +0x05c Gs                : 0x428a
   +0x05e Reserved7         : 0x8801
   +0x060 LDT               : 0
   +0x062 Reserved8         : 0x7eb
   +0x064 Flags             : 0
   +0x066 IoMapBase         : 0x20ac
   +0x068 IoMaps            : [1] _KiIoAccessMap
   +0x208c IntDirectionMap  : [32]  "???"
```

我们看 `IoMapBase` 值为 `0x20ac`。而在上面我们看到 `TSS` 的大小是 `0x20ab`，说明已经越界，根据前面我们列出的检测规则第五条，产生出错码为 0 的通用保护故障；因此，看到这里，我们可以考虑一种办法，就是扩充下这个结构的大小，让 `tss` 的结构大小大于 `0x20ac`，  

也就是说在 `tss` 中给 I/O 许可位图留出一块空间来。考虑到增加部分不能增加物理分页，因此我们可以设置的结构体最大值就是 `0x2fff`。这样，我们扩充的大小是 `0x2fff - 0x20ab = 0xf54`；  

因为 I/O 许可位图中每一 `bit` 代表一个端口，因此，`0xf54` 字节对应的可访问的最大端口是：  

`0xf54 * 8 = 0x7aa0`，对应于 10 进制，就是 `31392`。因此我们可访问的 i/o 口范围是 `0 ~ 31392`。  

因为每个进程都有自己的 I/O 许可位图，每个单独的 I/O 端口的访问权限都可以对每个进程进行单独授权，如果相关的位被设置的话，对对应端口的访问就是被禁止的，如果相关的位被清除，那么进程就可以访问对应的端口。  

`TSS` 的设计意图是为了在任务切换的时候保存处理器状态，从执行效率的考虑出发，`Windows NT` 并没有使用这个特征，它只维护一个 `TSS` 供多个进程共享，因此本篇中我们扩展了 `tss` 结构大小，扩展部分会影响到所有进程。  

其实这种做法，早在 1995 年 `Dale Roberts` 写了一个名叫 `totalio` 的例子影响较广，后来在 2002 年，`sinister` 大牛在此基础上改写了一个。虽然做法早就有了，但是网上并没有过多地介绍其中的原理。  

今天我们贴出的代码就是 `sinister` 大牛的。大家一起来学习。我在其代码上作了一个小小的调整，就是原来其扩展到 `0x2fab`，我把这个扩展范围扩大到了 `0x2fff`。反正分页是 `0x1000` 大小对齐，剩余的空间闲着也是浪费。

测试代码：

```c
#include "stdio.h"

void main()
{
     _asm
    {
           mov dx,0x64
           mov al,0xfe
           out  dx,al
    }
}
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `drv.rar`  

 |
