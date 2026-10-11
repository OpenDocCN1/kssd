
# 详解 Windows 内存分页机制

> 原文：`www.pediy.com/kssd/pediy12/135274.html`  
> 博客：`http://hi.baidu.com/hu3167343`

昨天新买了两本书，看到了内存分页部分，特此记录下，没什么技术含量，错误之处还请大牛指点。

大多数现代的操作系统都支持虚存，这使得系统上的每个程序都拥有自己的地址空间。每当程序读取内存时，都必须指定一个地址。对于每个进程，该地址必须转换为实际的物理内存地址。

例如，若我们的 `MzfHips.exe` 程序需要的数据在虚拟内存地址的 `0x0041FF10` 处，则实际的物理地址可能被映射为 `0x01EE2F10`。

如下图：

![](img/f8f3e564512dfeb3f8996a2cb57a3559.jpg)

图 1: 虚拟地址转物理地址示意图（图中的数据仅仅是为了演示）

内存地址的转换主要通过一个称为页表目录的特殊表来完成。`Intel x86 CPU` 将页表目录的指针存储在特殊寄存器 `CR3` 中。该寄存器指向一个包含 1024 个 32 位值的数组，称为页目录。每个数组元素称为页目录项，它指定了页表在物理内存中的基地址，还通过状态位指示该页表当前是否存在于内存中。从页表中可以获得实际的物理地址。

![](img/d9a26540b1a626c57073d004066b62b6.jpg)

图 2: 在内存中寻找页面

下面我们来看下虚拟地址的组成：

![](img/0551b02232afd5a291fa3a042f067cbc.jpg)

由图可知，虚拟地址的前 10 位用来定位页目录项，中间 10 位用来定位页表项，最后 12 位得到具体物理地址的偏移。

到了这里，我们来总结下具体的步骤：

1. `CPU` 查询 `CR3` 寄存器以找到页表目录的基地址
2. 操作系统根据所请求的虚拟地址的前 10 位(如图 3)，来定位页目录项，从而在内存中找到相应的页表。
3. 页表根据中间的 10 位定位该页相应的物理内存首地址
4. 根据虚拟地址的后 12 位得到具体的物理地址相对于首地址的偏移量。
5. 最后得到的物理地址即包含我们要请求的数据

知道了前面的知识，下面我们来看下具体的页目录项和页表项中的内容。

由前面我们知道，`CR3` 寄存器指向页目录的首地址，而页目录是由最多 1024 个可能的页目录项组成。

下面我们看下页目录项的地址组成：

![](img/a8e03e629b865efbf0e2802ecb8c154c.jpg)

当访问页目录项时，要检查 `U` 位(第 2 位)，若 `U` 位为 0，则意味着正在处理的页表只能用于内核。

还要检查 `W` 位(第 1 位)，若 `W` 位为 0，则内存是只读的。

页目录项指向整个页表，即整个页面集合，因此，页目录项中的设置应用于整个内存页范围。

下面来看下页表项：

![](img/e3ddbc3119c0297240f03c66a0145d7d.jpg)

当访问页表项时，仍然首先要检查 `U` 位(第 2 位)，若 `U` 位为 0，则只有内核模式的程序才能够访问该内存页。

还要检查 `W` 位(第 1 位)，以判断读写访问权限。

最后还要判断 `P` 位(第 0 位)，若它设置为 0，则当前内存页被换出在磁盘上。若它设置为 1，则内存是常驻，并且当前是可用的。

在将内存页换出后，内存管理器在成功访问它之前必须将该页换入内存。

还有最后一个问题就是，系统上的大多数可执行程序的其实地址都是 `0x00400000`。多个进程如何能使用同一个虚拟地址，而不会在物理内存中发生冲突？那是因为系统上的每个进程都维护一个独立的页目录，都拥有自己私有的 `CR3` 寄存器的值。

当线程发生切换时，旧线程的状态会被保存起来。若当前调度运行的线程不属于刚才的进程，则当前进程的页目录地址会被加载到 `CR3` 寄存器中。页目录地址可以在进程的 `KPROCESS` 结构中找到。

分析下 `MmIsAddressValid` 函数，加深下对分页机制的了解。(参考 combojiang 大叔)

下面是 `ida` 看到的 `ntoskrnl.exe` 中导出的 `MmIsAddressValid`。

```
.text:0040C661 ; BOOLEAN __stdcall MmIsAddressValid(PVOID VirtualAddress)

.text:0040C661                 public MmIsAddressValid

.text:0040C661 MmIsAddressValid proc near               ; CODE XREF: sub_40D65E+Cp

.text:0040C661                 ; sub_415459:loc_415470p ...

.text:0040C661

.text:0040C661 VirtualAddress   = dword ptr  8

.text:0040C661

.text:0040C661 ; FUNCTION CHUNK AT .text:0041B84E SIZE 00000007 BYTES

.text:0040C661 ; FUNCTION CHUNK AT .text:0044A4F2 SIZE 00000019 BYTES

.text:0040C661

.text:0040C661                 mov     edi, edi

.text:0040C663                 push    ebp

.text:0040C664                 mov     ebp, esp

.text:0040C666                 mov     ecx, [ebp+VirtualAddress] ;取出虚拟地址

.text:0040C669                 mov     eax, ecx

.text:0040C66B                 shr     eax, 14h          ; 右移 20 位

.text:0040C66E                 mov     edx, 0FFCh        ; 取高 10 位

.text:0040C673                 and     eax, edx

.text:0040C675                 sub     eax, 3FD00000h    ; 加上 0xc0300000

.text:0040C675                 ; PDE = ((VA >> 22) << 2 )& 0xffc + 0xc0300000

.text:0040C67A                 mov     eax, [eax]

.text:0040C67C                 test    al, 1             ; 页目录项的第 0 位, 即 P 位

.text:0040C67E                 jz      loc_41B84E

.text:0040C684                 test    al, al

.text:0040C686                 js      short loc_40C6AC  ; 判断 page size 位

.text:0040C688                 shr     ecx, 0Ah          ; 右移 10 位

.text:0040C68B                 and     ecx, 3FFFFCh

.text:0040C691                 sub     ecx, 40000000h

.text:0040C697                 mov     eax, ecx          ; PTE = ((VA >> 12) << 2 ) & 0x3FFFC + 0xc0000000

.text:0040C699                 mov     ecx, [eax]        ; ecx = PTE Context

.text:0040C69B                 test    cl, 1             ; 判断 present 位

.text:0040C69E                 jz      loc_41B84E

.text:0040C6A4                 test    cl, cl

.text:0040C6A6                 js      loc_44A4F2        ; 判断 page size 位

.text:0040C6AC

.text:0040C6AC loc_40C6AC:                            ; CODE XREF: MmIsAddressValid+25j

.text:0040C6AC                 ; MmIsAddressValid+3DE9Fj

.text:0040C6AC                 mov     al, 1

.text:0040C6AE

.text:0040C6AE loc_40C6AE:                            ; CODE XREF: MmIsAddressValid+F1EFj

.text:0040C6AE                 pop     ebp

.text:0040C6AF                 retn    4

.text:0040C6AF MmIsAddressValid endp
```

Ps：上面说的理论是在未开启 `PAE` 模式下的一般情况，在开启了 `PAE` 的情况下，可能会有所不同。
