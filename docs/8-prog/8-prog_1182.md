
# [翻译] Entering the kernel without a driver-无驱动进入内核态与通过 APIC 获得中断信息

> 原文：[`www.pediy.com/kssd/pediy12/132597.html`](https://www.pediy.com/kssd/pediy12/132597.html)

《Entering the kernel without a driver and getting interrupt information from APIC》

------Anton Bassov

《无驱动进入内核态与通过 APIC 获得中断信息》

先唠叨两句-----整理移动硬盘时，发现一篇两年前翻译了一半的文章....唉....很多烦心的事情涌了上来。比较老的一篇文章了，先贴上来，可能已经有朋友翻译过了，见谅....

------- SHK

## 第一部分：免驱动进入内核态

### 【简介】

虽然让用户模式下的应用程序进入内核模式是激动人心的尝试，但实现方法却闻所未闻。第一个实现该想法的人是 Matt Pietrek(多年以前在 Windows 95 环境下)。后来 Prasad Dabak, Sandeep Phadke 和 Milind Borate 在 Windows NT 下改进了他的技术。为了进入内核模式，不得不在全局描述符表(GDT)中创建中断门描述符，以便应用程序通过中断门进入内核模式。但是，用户模式下的代码是不允许访问 GDT 的，上述方法中使用了内核模式下的驱动来创建中断门描述符。当然，一个逻辑上的问题产生了 - 如果需要驱动来实现，那么无驱动进入内核态是指什么？？？二者自相矛盾，不是么？

本文将描述用户态程序如何访问内核地址空间，以及不依靠驱动在 GDT 中创建中断门描述符。文章讲解了 32 位处理器上如何完成虚拟地址到物理地址的转换，以及用户态程序如何查找指定虚拟地址代表的物理地址。所有方法思路完全是我独立设计 - 你不可能在其它任何地方找到相似的文章。文章也说明了 Windows NT 如何保护内核地址空间，基于 x86 系统的特权级转换和无驱动进入内核态。

补充一点，文中介绍了高级可编程中断控制器(APIC)以及如何通过它获取中断信息。该知识点在 Windows 界似乎并不为人所知，虽然 Mark Russinovich 和 David Solomon 在《Windows Internals(第四版)》中简单提到了 APIC。不管怎样，该书并没有说明怎样进行 APIC 编程。我在其它以 Windows 为主题的文章中也从未遇到关于 APIC 的介绍 - 我不得不根据 Intel 的参考手册来弄清这一切。因此，我相信这些内容能使 Windows 开发人员产生极大的兴趣。

总而言之，如果你想学到更多关于系统内部的东西，本文正合你胃口。

### 【访问内核地址空间】

假定我们想在用户模式下运行一个程序来访问内核空间的一些地址。这可能吗？可能，也不可能。若我们直接当作一个虚拟地址来访问，将引发非法访问(ACCESS VIOLATION)异常。不过，可以换个思路 - 物理内存可以当作一个名为“`\\Device\\PhysicalMemory`”的区域用函数 `NtOpenSection()` 打开，然后通过内核 API 函数 `NtMapViewOfSection` 进行映射。通过该技术我们可以访问内存的任何页面。如果我们知道目标虚拟地址所代表的物理地址，工作将变得非常简单 - 只需要用 `NtMapViewOfSection()` 映射这块物理内存，函数返回的指针指向内核中目标地址对应相同的物理地址，但地址在数值上小于 `0x80000000`。我们能够在用户模式下访问物理地址之后，若要写入数据，则映射前需要做些额外工作。问题就在于非系统级进程，进程若不是在用户态 SYSTEM 等级启动，就无法对“`\\Device\\PhysicalMemory`”进行写入访问。因此，你需要授予程序写入“`\\Device\\PhysicalMemory`”的权限。代码如下：

```
EXPLICIT_ACCESS Access;
PACL OldDacl=NULL, NewDacl=NULL;
PVOID security;
HANDLE Section;
INIT_UNICODE_STRING(name,L"\\Device\\PhysicalMemory");
OBJECT_ATTRIBUTES oa ={sizeof(oa),0,&name,0,0,0};
memset(&Access, 0, sizeof(EXPLICIT_ACCESS));
NtOpenSection(&Section, WRITE_DAC | READ_CONTROL, &oa);
GetSecurityInfo(Section, SE_KERNEL_OBJECT, DACL_SECURITY_INFORMATION, NULL, NULL, &OldDacl,NULL, &security);
Access.grfAccessPermissions = SECTION_ALL_ACCESS;
Access.grfAccessMode        = GRANT_ACCESS;
Access.grfInheritance       = NO_INHERITANCE;
Access.Trustee.MultipleTrusteeOperation = NO_MULTIPLE_TRUSTEE;
Access.Trustee.TrusteeForm  = TRUSTEE_IS_NAME;
Access.Trustee.TrusteeType  = TRUSTEE_IS_USER;
Access.Trustee.ptstrName = "CURRENT_USER";
SetEntriesInAcl(1, &Access, OldDacl, &NewDacl);
SetSecurityInfo(Section, SE_KERNEL_OBJECT, DACL_SECURITY_INFORMATION,NULL, NULL, NewDacl, NULL);
CloseHandle(Section);
```

为了让代码运行，你必须以 Administrator 身份登录系统。根据我的经验，受限用户无法打开“`\\Device\\PhysicalMemory`”进行任何访问 - 即使你请求对“`\\Device\\PhysicalMemory`”进行只读访问， `NtOpenSection()` 也总是返回出错代码。

因此，一旦我们拥有 Administrator 权限，就可以访问内存的任何页面，并且是在用户模式下 - 即使页面发生错误。换句话说，我们可以间接访问内核地址空间。你不觉得激动么？为了实用，我们还要找出目标地址在内核空间中对应的物理地址。内核模式的驱动程序可以调用 `MmGetPhysicalAddress()`，但用户模式下绝对没机会使用这个函数。进一步说，用户态的 API 函数可能都毫无帮助。因此，只能依靠我们自己去弄清一切，这就是为什么我们首先要学会 32 位处理器下的虚拟-物理地址转换。

在 x86 系统上页面大小可以是 4KB 或 4MB。如果页面大小是 4KB，32 位虚拟地址包含 3 段 CPU 所需的信息，用于定位物理地址。虚拟地址的低 12 位表示在物理内存页面中偏移量。12 至 21 位是页面表中项目的索引号，共有 1024 个物理页面入口。每个进程理论上最多有 1024 个页面表。那么，`1024×1024×4096` 产生了 4GB 可编址空间。进程中所有页面表的地址存储在一个页面目录中。第 22 到 31 位用于在页面目录中查找合适的页面表。各个进程维护着自己的页面目录。当前进程页面目录的物理地址存放在 `CR3` 寄存器中。这个寄存器只在任务切换时被修改，所以不希望系统中其它任何东西来访问它。通过任务切换，系统在 `CR3` 载入不同的页面目录地址。因此，先前关联到物理页面 X 的虚拟地址现在可能仍然表示同一个页面 X，也可能是其他页面 Y，或者没有对应的物理页面。这就是为什么对于任何虚拟地址，在进程 A 中有效的地址空间可能在进程 B 中变得毫无意义，除非它们关联到同一个物理地址。驱动程序被一次性加载到内存中，并映射到所有进程的相同虚拟地址上，因此改变 `CR3` 的值不会对驱动有影响。

每个页面目录和页面表入口用 32 位结构体描述如下：

```
struct PageDirectoryOrTableEntry{
  DWORD Present:1;
  DWORD ReadWrite:1;
  DWORD UserSupervisor:1;
  DWORD WriteThrough:1;
  DWORD CacheDisabled:1;
  DWORD Accessed:1;
  DWORD Reserved:1;
  DWORD Size:1;
  DWORD Global:1;
  DWORD Unused:3;
  DWORD PhysicalAddress:20;
};
```

如果页面大小是 4KB，地址转换方式如下：

1. CPU 从 `CR3` 寄存器获取当前进程的页面目录。虚拟地址的高 10 位表示为索引号(i)，CPU 用它来查找给定虚拟地址在该页面目录中对应的页面表的入口地址。若页面目录中第 i 个入口的 `Present` 标志没有设置(为 1)，CPU 将产生一个页面错误异常(`INT 0xE`)。导致页面错误的原因很多，比如无效地址、向只读单元写入数据等等。因此，系统首先检查页面错误的原因。若这个异常仅仅是由于 `Present` 标志未设置造成的，系统就会得知存在问题的页面已经交换到了磁盘上。那么，它会将该页面装载到内存，并设置相应页面目录入口的 `Present` 标志，CPU 也会再次尝试访问该虚拟地址。以上所有操作对用户代码来说是不可见的。
2. 确定页面表之后，CPU 在该表中进一步查找指定虚拟地址对应的页面位置。第 12 至 21 位表示为索引号(i)，CPU 用它来确定页面中目标页面的地址。如果页面表中第 i 个入口的 `Present` 标志未设置，CPU 将产生一个页面错误异常，系统按照前面提到的方式处理。
3. 最后，已经确定了目标页面，CPU 就是用虚拟地址的低 12 位作为页面内的偏移地址。

这就是 4KB 页面大小的地址转换过程。如果页面为 4MB，则地址的高 10 位表示页面目录中页面的索引号，剩余 22 位表示该页面内的偏移量(1024*4MB 的偏移量，4GB 的可编址空间)。

64 位处理器采用更高级的地址变换方式。例如，Itanium 处理器允许所谓的硬件级的数据执行保护(DEP)。很多人误以为 DEP 是 Windows XP SP2 的功能。并非如此 - DEP 是 CPU 的功能，可以被 Windows XP SP2 利用。如果 Windows XP SP2 运行在一个不支持 DEP 的 CPU 上，DEP 就不会发挥作用 - 在没有 DEP 功能的处理器上，没有办法去手动阻止机器指令的执行。我们的讨论是基于 32 位处理器和 4KB 页面大小进行的。

在 Windows NT 下，当前进程的页面目录被映射到虚拟地址 `0xC0300000`。结合我们已知的地址转换知识，得出以下两条结论：

1. 在 Windows NT 中，页面目录的第 `0x300` 个入口保存着页面目录自己的物理地址。
2. 虚拟地址对应的页面表可以按表达式 `0xC0000000+((address>>10)&0x3FF000)` 来访问。通过这种转换，可以发现页面表地址 `0xC0300000` 就是 `0xC0300000` 本身。换而言之，页面目录本身也位于页面目录的虚拟地址对应的页面表中。

现在，我们来实战一下。试想如下内核模式的代码：

```
_asm{
  mov ebx,0xfec00000
  or ebx, 0x13
  mov eax,0xc0000000
  mov dword ptr[eax],ebx
}//我们想干什么???难道疯了吗???
PULONG array=NULL;
array[0]=1;
```

运行这段代码将发生什么？答案似乎显而易见，但那是错的 - 程序并不会崩溃。我们对单元 `0xfec00013` 进行写操作(高 20 位指明内存页面 `0xfec00`，低 12 位设置了 `Present`、`ReadWrite` 和 `CacheDisabled` 标志)，它正是第一个页面表的入口，该地址存放在页面目录的第一个入口中。下面看看 CPU 如何转换虚拟地址 0，你将会明白，由于我们的修改，0 会被转换为物理地址 `0xfec00000`。从现在起，虚拟地址范围 0 到 `0x1000` 变成进程地址空间内的一个有效范围。`NULL` 指针生效，并指向物理地址 `0xfec00000`!!!很有趣，对吧？稍后你会明白这个技巧在某些情况下相当有用。

言归正传 - 你可能还记得，我们当前的任务是找到目标虚拟地址所代表的物理地址。更重要的是，在用户模式下完成这项工作。如果已经知道物理内存中进程的页面目录的地址，我们的工作就不值一提了。假设目标虚拟地址是 V。通过 `NtMapViewOfSection()` 函数映射页面目录到虚拟地址 D，把 D 看作是由 1024 个 `DWORD` 类型数据组成的数组。D 中第 `(V>>22)` 个入口数据的高 20 位描述的物理页面就是一个页面表，与虚拟地址 V 相对应。所以，我们将映射该页面到虚拟地址 T，再次将其看作一个包含 1024 个 `DWORD` 元素的数组 - T 中第 `((V>>12)&0x3FF)` 个入口数据的高 20 位对应着虚拟地址 V 所表示的物理页面。这样描述非常简洁。只有一个问题 - `CR3` 寄存器，通过它才能使我们进程的页面目录的物理地址有效，却无法在用户模式下访问。那么，首先我们必须要找到进程页面目录所在的物理地址。

我们将要做一次内存扫描，把每个内存页面映射到进程的地址空间，那样做迟早会找到进程的页面目录。但怎样识别它呢？假定我们检查的物理页面是 P，通过 `NtMapViewOfSection()` 映射到虚拟地址 V。V 是包含 1024 个 `DWORD` 元素的数组。若 P 是包含进程页面目录的物理页面，那么：

1. V 中第 `0x300` 个入口数据的高 20 位一定等于 P，因为页面目录的第 `0x300` 个入口数据保存了目录自身的物理地址。
2. V 中第 `0x300` 个入口数据的最低位一定被置 1，因为它表明了内存中页面的存在。
3. 若 P 是一个页面目录，则 V 中第 `(V>>22)` 个入口数据描述了一个页面表，它对应于虚拟地址 V 自身，并且该页面表已载入内存。因此，V 中第 `(V>>22)` 个入口数据最低位一定被置为 1。

若不满足以上任意一条，我们就可以断定 P 不是页面目录，那么接着扫描下一个页面。否则，就映射 V 中第 `(V>>22)` 个入口数据高 20 位所描述的页面(把这个虚拟地址叫做 T 吧)，并假定 T 是对应于虚拟地址 V 的一张页面表。若假设成立，则：

1. T 中第 `((V>>12)&0x3FF)` 个入口数据的高 20 位一定等于 P。
2. T 中第 `((V>>12)&0x3FF)` 个入口数据的最低位一定被置 1。

满足以上两个条件，就可以认为 P 确实是页面目录，所以它可以通过虚拟地址 V 来访问。我们凭什么相信这个结论？再看看虚拟-物理地址转换的相关内容，仔细想想 V 到物理地址 P 的转换，然后，我希望你将会理解这一切。参看以下代码：

```
//检查以下我们获得了多少内存
MEMORYSTATUS meminfo;
GlobalMemoryStatus(&meminfo);
//获取内存句柄
status = NtOpenSection(&Section, SECTION_MAP_READ|SECTION_MAP_WRITE, &oa);
DWORD found=0,MappedSize,x;
LARGE_INTEGER phys;
DWORD* entry;
PVOID DirectoryMappedAddress,TableMappedAddress;
DWORD DirectoryOffset,TableOffset;
for(x=0;x<meminfo.dwTotalPhys;x+=0x1000){
  //映射内存中的当前页面
  MappedSize = 4096;
  phys.QuadPart = x;
  DirectoryMappedAddress = 0;
  status = NtMapViewOfSection(Section, (HANDLE) -1, &DirectoryMappedAddress, 0L, MappedSize, &phys, &MappedSize, ViewShare, 0, PAGE_READONLY);
  if(status)
    continue;
  entry = (DWORD*)DirectoryMappedAddress;
  //获取偏移量
  DirectoryOffset = (DWORD)DirectoryMappedAddress;
```



```
TableOffset = (DWORD)DirectoryMappedAddress;      
DirectoryOffset>>=22;      
TableOffset = (TableOffset>>12)&0x3ff;  

//检查该页面是否为页面目录       
//第 0x300 个入口数据的高 20 位一定等于 P      
//并且第 0x300 个入口和第(V>>22)个入口的数据 Present 位一定被设置 1      
//不满足就扫描下一个页面      
if((entry[0x300]&0xfffff000)!=x ||(entry[0x300]&1)!=1 || (entry[DirectoryOffset]&1)!=1){  
  NtUnmapViewOfSection((HANDLE) -1, DirectoryMappedAddress);  
  continue;  
}  
//看样子差不多了      
//接下来试着映射一下页面表      
MappedSize = 4096;       
phys.QuadPart = (entry[DirectoryOffset]&0xfffff000);       
TableMappedAddress = 0;      
status = NtMapViewOfSection(Section, (HANDLE) -1, &TableMappedAddress, 0L,MappedSize, &phys, &MappedSize, ViewShare, 0, PAGE_READONLY);      
if(status){  
  NtUnmapViewOfSection((HANDLE) -1, DirectoryMappedAddress);  
  continue;  
}  
//现在来检查它是否是一个真正的页面表       
//如果是，第((V>>12)&0x3ff)个入口数据的高 20 位一定等于 P      
//并且该入口的 Present 位一定被设置为 1  
//上述为真，则 P 确实是一个页面目录      
entry = (DWORD*)TableMappedAddress;      
if((entry[TableOffset]&1)==1 && (entry[TableOffset]&0xfffff000)==x)  
  found++;  
NtUnmapViewOfSection((HANDLE) -1, TableMappedAddress);      
//目录找到了 - 没必要继续扫描下去      
if(found)  
  break;  
NtUnmapViewOfSection((HANDLE) -1, DirectoryMappedAddress);  
}  
```

这段代码可信么？会不会误判了一些页面为页面目录？这种错误要用 `42` 个随机位同时发生来表示。因此，发生该错误的概率为 `2` 的 `42` 次方分之一，也就是说，实践中可以忽略不计。那么会不会丢掉了某些页面目录？我们进行连续扫描可以保证找到它，除非页面目录在代码运行时被移走了。理论上而言，内存管理器可能会移动物理内存中包括页面目录在内的任何页面，但这只发生在页面故障时 - 某些页面被交换到磁盘，之后又加载到内存的其它位置。一旦频繁交换访问页面到磁盘导致性能显著降低，系统就不再频繁交换访问中的页面。在页面目录交换到磁盘之前，进程不得不暂停一段时间 - 代码运行时，页面目录正在被执行中的指令访问，所以它不可能成为分页内存。因而我们可以放心地推测代码执行过程中，页面目录位于内存中某个固定的地址上。也就是说，我们的方法在实践中是非常可靠的 - 我用过很多、很多、很多次了，屡试不爽(!!!)。

现在，一旦我们知道页面目录的物理地址，就可以轻松获取任何我们感兴趣的虚拟地址所对应的物理地址 - 解决“方案”前面已叙述过。下面我们来获取保存了全局描述符表(`GDT`)的页面的物理地址。

```
//获取 GDT 的基地址  
BYTE gdtr[8];  
DWORD gdtbase,physgdtbase;  
_asm{     
  sgdt gdtr     
  lea eax,gdtr     
  mov ebx,dword ptr[eax+2]     
  mov gdtbase,ebx  
}  
//获取目录和表的偏移量  
DirectoryOffset = gdtbase;  
TableOffset = gdtbase;  
DirectoryOffset>>=22;  
TableOffset = (TableOffset>>12)&0x3ff;  
entry = (DWORD*)DirectoryMappedAddress;  
//映射页面表 - 物理地址是页面目录中第(V>>22)个入口数据的高 20 位  
MappedSize = 4096;  
phys.QuadPart = (entry[DirectoryOffset]&0xfffff000);  
TableMappedAddress = 0;  
status = NtMapViewOfSection(Section, (HANDLE) -1, &TableMappedAddress, 0L, MappedSize, &phys, &MappedSize, ViewShare, 0, PAGE_READONLY);  
//物理页面是页面表中第((V>>12)&0x3ff)个入口数据的高 20 位  
//这就是我们需要的  
entry = (DWORD*)TableMappedAddress;  
physgdtbase = (entry[TableOffset]&0xfffff000);  
//取消一切映射  
NtUnmapViewOfSection((HANDLE) -1, TableMappedAddress);  
NtUnmapViewOfSection((HANDLE) -1, DirectoryMappedAddress);  
```

`GDT` 在用户模式下不可访问，但是，如前所说，这个限制对我们不再有效 - 我们已经找到了页面所在的物理地址，对不？为了访问 `GDT`，首先要利用 `NtMapViewOfSection()`映射这个页面到虚拟地址。

`GDT` 不一定位于页面的起始处，所以我们要使用它虚拟地址的低 `12` 位作为页面内的偏移量。因此，要给 `V` 加上(`gdtbase&0xFFF`)。至此我们可以在虚拟地址 `V` 中读写 `GDT`。那么，我们就能在用户态程序中对内核地址空间进行读写访问。若还能执行代码岂不更好？这就是我们所期待的事情，也就是前面一系列讲解的目的，也就是我们为何要在内核地址空间中得到 `GDT` 的物理地址而不是其它地址的原因 - `GDT` 有助于我们的程序在不依靠驱动的情况下进入内核模式。

### 【代码特权级和保护属性】

`GDT` 中保存了段描述符、局部描述符表(`LDT`)描述符、中断门描述符和任务状态段(`TSS`)描述符。虽然各自结构不同，但所有这些描述符大小都是 `8` 字节。段描述符和中断门描述符的构成用以下 `8` 字节结构体表示：

```
struct  SegmentDescriptor{  
  WORD LimitLow;      
  WORD BaseLow;      
  DWORD BaseMid : 8;      
  DWORD Type : 5;      
  DWORD Dpl : 2;      
  DWORD Pres : 1;      
  DWORD LimitHi : 4;      
  DWORD Sys : 1;      
  DWORD Reserved_0 : 1;      
  DWORD Default_Big : 1;      
  DWORD Granularity : 1;      
  DWORD BaseHi : 8;       
};  

struct CallGateDescriptor{    
  WORD offset_low;    
  WORD selector;    
  BYTE param_count : 5;    
  BYTE   unused : 3;    
  BYTE  type : 5;    
  BYTE  dpl : 2;    
  BYTE present : 1;    
  WORD offset_high;  
};  
```

`Windows NT` 中不使用 `LDT` 和中断门。尽管考虑到性能的因素，`Windows NT` 下所有用户进程运行在一个单任务的环境中，`GDT` 中没有 `TSS` 描述符。它们主要被保留用来做“异常环境”，也就是系统崩溃 - 它们的任务是确保 `CPU` 重启前系统有足够长的时间运行来抛出一个蓝屏。我们对它们不感兴趣。那段描述符如何？你可能会认为，一旦 `Windows` 使用平坦内存模式，我们就不必关心段描述符。

实际上，事情没那么简单。`Windows` 平坦内存模式的实现是通过设置 `GDT` 中代码、数据、堆栈和附加段描述符的 `BaseLow`、`BaseMid` 和 `BaseHi` 域值为 `0`，所以内存编址时不需要指明段基址和偏移量。可是，段仍然隐藏在幕后，因为在基于 `x86` 的机器上不使用段分割，就无法实现保护模式下运行的系统。严格地讲，问题在于 `x86` 系统中不存在内核或用户操作模式这类概念。相反，执行特权级指令和访问超级权限页面的功能是受代码段描述符中的描述符特权级(`DPL`)域控制的。因此，为了区分特权级代码和非特权级代码，需要两个代码段 - 特权级代码段 `DPL=0`，非特权级代码 `DPL=3`。尽管它们可能被映射到同一虚拟地址 `0` 中，但处理器会将它们区别对待。`CS` 寄存器用于存放距离当前运行的代码段描述符表头的偏移字节数，与该代码段的 `DPL` 进行或运算。这就是 `CPU` 如何知道当前代码特权级的方法。在 `Windows NT` 中，`CS` 寄存器的值可以为 `0x8`(执行特权级代码时)或 `0x1B`(执行非特权级代码时)。

所以，特权级定义在 `CS` 寄存器中比定义在代码地址中更好。如果某个函数在执行时 `CS` 等于 `0x8`，它就是特权级代码；但同一函数在 `CS` 等于 `0x1B` 时执行，它就是非特权级代码。我知道你的疑惑 - 毕竟非特权级代码没有机会访问 `0x80000000` 以上的地址单元。那不就有些矛盾么？不要紧，再看看 `PageDirectoryOrTableEntry` 结构体，注意 `UserSupervisor` 标志位，问题就迎刃而解 - `Windows` 只是把映射到 `0x80000000` 以上地址的页面在页面表中标记为仅限超级权限访问。非特权级代码尝试访问这些页面，就会发生非法访问异常。任何位于该页面的函数不能在 `CS` 等于 `0x1B` 时运行 - 否则直接给出非法访问异常。因此，内核地址空间的保护是通过带页面权限保护的组合段分割方式来实现的。 

代码段从非特权级转变到特权级可以用下列方式实现：

1.通过 `INT n` 指令 - 该指令将用户模式下的 `SS` 寄存器、用户模式下的 `ESP` 寄存器、`EFLAGS` 寄存器、用户模式下的 `CS` 寄存器和返回地址(按该顺序压栈)压入内核栈，并转移控制权到 `INT n` 中断处理。`Windows NT` 将中断描述符表入口的 `DPL` 位设置为用户态代码只允许在中断 `0x3`、`0x4`、`0x2A`、`0x2B`、`0x2C`、`0x2D` 和 `0x2E` 中执行。

2.`SYSENTER` 指令 - 该指令会设置调用线程的 `ESP` 为特殊寄存器 `SYSENTER_ESP_MSR` 结构所指定的值，设置 `CS` 为特殊寄存器 `SYSENTER_CS_MSR` 结构所指定的值，并转移执行到特殊寄存器 `SYSENTER_EIP_MSR` 结构所指定的地址处(用户模式下不能访问这些寄存器)。标志位和返回地址不会被保存。`Windows NT/2000` 不使用 `SYSENTER` 指令，`Windows XP` 下该指令转移执行到系统服务分配器中。

3.通过中断门进行远调用 - 当通过中断门设定内核入口时，`CPU` 从用户栈中复制大约 `32` 个(实际数目由中断门描述符的 `param_count` 域指定)`DWORD` 单元到内核栈，接着将用户模式的 `CS` 寄存器值和返回地址压入内核栈，然后设置 `CS` 为中断门描述符 `selector` 域所指定的值，最后转移到中断门描述符 `offset_low` 和 `offset_high` 域所指定的地址处执行代码。`Windows` 不使用中断门。

前两个方式用于 `Windows` 中，它们转移执行到一些系统定义的地址处。因此，我们不能使用它们来转移执行到我们想要的任意一个地址。中断门与之不同 - 一旦 `Windows` 不使用它，我们就可以随心所欲地操作中断门。如果在 `GDT` 中设置一个中断门，我们就可以将代码执行权转移到中断门描述符中指定的任意地址。当然这个地址必须在我们进程地址空间中有效，不过不必位于内核地址空间中。特权级代码可以访问进程空间中的任何有效地址，另外你应当明白，代码的权限定义在 `CS` 中，而不是寄存器 `EIP` 中。所以，如果我们通过程序在中断门描述符中指定了函数的入口地址，那么通过中断门调用该函数，它就会被 `CPU` 当作特权级代码，即使它位于进程的用户地址空间中。接下来这个函数可以访问任何内存地址、`IO` 端口、产生中断、调用 `ntoskrnl.exe` 中的导出函数，也就是说，几乎可以完成一切内核模式驱动程序才能做的事情。与此同时，该函数不应去尝试调用用户模式下 `DLL` 中的 `API` 函数。为什么呢？因为这些 `API` 函数可能调用了内核 `API`，并且内核 `API` 函数动用了系统服务，换句话说，通过系统服务分配器进入内核 - 在我们已经处于特权级模式下的时候！！！这不会给我们带来任何好处。一个基本原则是 `Windows` 对我们的手法了解越少，对我们就越有利。那我们怎样返回用户模式呢？可以用 `IRETD`、`SYSEXIT` 或 `RETF` 指令实现。虽然返回的方式和调用的方式本应该匹配，但这也不是绝对的。例如，在 `Windows XP` 下，从 `INT 0x2B` 中断处理程序返回用的是 `SYSEXIT`，并非 `IRETD` 指令 - 只要进入中断程序和中断返回二者能很好地平衡内核和用户栈，就万事 `OK`。显然，在我们讨论的情况下，`RETF` 指令是离开内核模式最合理的方法，所以我们利用 `RETF` 来回到用户模式。

### 【在 GDT 中设置中断门】

现在万事俱备，可以通过我们的程序在 `GDT` 中设置中断门并进入内核模式了。请看下列代码：

```
//先映射 GDT  
PBYTE GdtMappedAddress = 0;  
phys.QuadPart = physgdtbase;  
MappedSize = 4096;  
NtMapViewOfSection(Section, (HANDLE) -1,(PVOID*)&GdtMappedAddress, 0L, MappedSize, &phys, &MappedSize, ViewShare, 0, PAGE_READWRITE);  
gdtbase& = 0xfff;  
GdtMappedAddress += gdtbase;  
CallGateDescriptor * gate = (CallGateDescriptor * )GdtMappedAddress;  
//在 GDT 中找一个没有被占用的入口.   
//当前 GDT 入口的类型无关紧要 - 所有类型的描述符第 48 位都是 Present 位， 
//所以我们把全部描述符都当作中断门  
selector = 1;  
while(1){         
  if(!gate[selector].present)  break;     
  selector++;  
}  
//设置一个中断门  
gate[selector].offset_low  = (WORD)((DWORD)kernelfunction & 0xFFFF);  
gate[selector].selector = 8;  
//将要传递一个参数  
gate[selector].param_count  = 1; 
gate[selector].unused = 0;  
//32 位中断门  
gate[selector].type = 0xc;           
//一定是 3  
gate[selector].dpl = 3;       
gate[selector].present = 1;  
gate[selector].offset_high = (WORD)((DWORD)kernelfunction >> 16);       
//不再需要物理内存  
NtUnmapViewOfSection((HANDLE) -1, GdtMappedAddress);  
CloseHandle(Section);  
```



### 映射 GDT 与设置中断门

首先，我们按照前面所说的方法映射 `GDT`，并查找一个未被使用入口。`GDT` 中所有描述符长度为 8 字节，第 48 位表示 `Present` 位。这对任何类型的描述符都是一样。因此，当我们查找空闲的入口时，可以将所有 `GDT` 入口当作中断门描述符，忽略它们的实际类型。找到入口之后，设置一个中断门描述符。我们把它的类型设为 `0xC` 来描述 32 位中断门，它的 `DPL` 为 3，也就是可以在用户模式下被访问。它的 `Selector` 域为 `0x8`，`offset_low` 和 `offset_high` 域分别为中断调用的函数的低 16 位和高 16 位地址。参数怎么办？通过指定 `param_count` 域？我想是这样，本文所涉及的行为都是非常规的，可以很好地利用它使我们的内核代码做些鲜为人知的事情。有一篇关于代码工程的文章讲述了如何从注册表中获得中断信息。我也将把这一切展示给你，当然，是通过更激动人心的方法，而不是靠解释中断源代码或搜索注册表来得到中断信息。因此，我们将传递 `IRQ` 作为函数的参数，而函数将返回这个特殊的 `IRQ` 所对应的中断向量。

### 高级可编程中断控制器

系统如何映射 `IRQs` 到中断向量并定义它们的优先级？这取决于你的机器是否支持高级可编程中断控制器（`APIC`）。这个可以通过 `CPUID` 指令，然后从指定寄存器 - `APIC_BASE_MSR` 结构读取信息来知道。

\====这里附上这篇文章后半部分关于 APIC 的原文，没有继续翻译了，SORRY====

#### Advanced Programmable Interrupt Controller

How does the system map `IRQs` to interrupt vectors and define their priority? It depends on whether your machine supports Advanced Programmable Interrupt Controller (`APIC`). This can be discovered by `CPUID` instruction and read from `APIC_BASE_MSR` model-specific register. If `APIC` is present and you make `CPUID` instruction with 1 in `EAX`, bit 9 of `EDX` register will be set by this instruction. In order to find out whether `APIC` is enabled, you have to read the `APIC_BASE_MSR` model-specific register - bit 11 of it must be set if `APIC` is enabled. Unless your computer is completely outdated, I am 99.9% sure that `APIC` is present and enabled on your machine. If it is not, then the interrupt vector, corresponding to some given `IRQ`, equals `0x30+IRQ`, so that timer (`IRQ0`) interrupt vector is `0x30`, keyboard (`IRQ1`) interrupt vector is `0x31`, etc. This how how Windows NT maps hardware interrupts if `APIC` is not present or disabled. In such cases interrupt priority is implied by `IRQ` - there is nothing than can be done here.

If `APIC` is present and enabled, things become much more interesting to program. Every CPU in the system has its own local `APIC`, physical address of which is specified by `APIC_BASE_MSR` model-specific register. Local `APIC` can be programmed by reading from and writing to its registers. For example, processor's `IRQL` can be manipulated via `Task Priority` register, which is located at the offset of `0x80` from the local `APIC`'s base address - this is what `KeRaiseIrql()` and `KeLowerIrql()` do. If you want to raise an interrupt, you can do it via `Interrupt Command` register, which is located at the offset of `0x300` from the local `APIC`'s base address - this is what `HalRequestSoftwareInterrupt()` does. You can also specify whether you want the CPU to interrupt itself or whether you want interrupt to be dispensed to all CPUs in the system. Local `APIC` programming is quite an extensive topic, so it is well beyond the scope of this article. If you need more information, I would strongly advise you to read Volume 3 of Intel Developer's Manual.

All local `APIC`s communicate with `IO APIC`, which is located on the motherboard, via `APIC` bus. `IO APIC` maps `IRQs` to interrupt vectors, and it is able to map up to 24 interrupts. `IO APIC` can be programmed by reading from and writing to its registers. These are 32-bit `ID Register` (located at the offset of `0`), 32-bit `Version Register` (located at the offset of `0X1`), 32-bit `Arbitration Register` (located at the offset of `0X2`), and 24 64-bit `Redirection Table` Registers, with every `Redirection Table` Register corresponding to some given `IRQ`. The location of `Redirection Table` Register, corresponding to any given `IRQ`, can be calculated as `0X10+2*IRQ`. If you want to know the binary layout of `Redirection Table`, I suggest you should read Intel `IOAPIC` manual - we are interested only in 8 low-order bits of `Redirection Table`, because they indicate interrupt vector that corresponds to the given `IRQ`. Interrupt priority can be calculated as `vector/16`, and, once operating system designers can map `IRQs` to interrupt vectors in any way they wish, they can assign any interrupt priority level to any given `IRQ`.

`IO APIC` uses indirect addressing scheme, which means all the above mentioned registers cannot be accessed directly. How can they be accessed then??? `IO APIC` provides 2 direct access registers for this purpose. These are `IOREGSEL` and `IOWIN` registers, located at the offsets of respectively `0` and `0X10` from `IO APIC`'s base address. `IO APIC` is mapped to the physical memory at the address `0XFEC00000`. Although Intel allows operating system designers to relocate `IO APIC` to some other physical address, Windows NT does not relocate it. Therefore, we will make a bold assumption that `IO APIC` is located at the physical address `0XFEC00000` on your machine, so that physical addresses of `IOREGSEL` and `IOWIN` registers are respectively `0XFEC00000` and `0XFEC00010`. In order to access these registers, you have to map them to non-cached memory. In order to read any indirect access register, you have to write its offset to `IOREGSEL` register - subsequent read of `IOWIN` register will return the value of the target indirect access register. All reads are 32-bit. If you want to read 32 low-order or 32 high-order bits of `Redirection Table` Register that corresponds to some given `IRQ`, you have to write respectively `0X10+2*IRQ` or `0X10+2*IRQ+1` to `IOREGSEL` register, and then read `IOWIN` register in order to get the sought information.

How are we going to map `IO APIC` to the virtual memory? If we used a regular driver, we would call `MmMapIoSpace()`. However, in our case things are slightly different. If CPU treats our code as a privileged one, it does not necessarily imply that Windows always shares its opinion on the subject - everything depends on what you want to do. Some `ntoskrnl.exe`'s exports ( for example, `ExAllocatePool()`) can be called by our code without a slightest problem, but `MmMapIoSpace()` is not among them - if our code calls `MmMapIoSpace()`, we will get a blue screen with `IRQL_NOT_LESS_OR_EQUAL` error code. What are we going to do then? This is when our trick with mapping some page to the virtual address 0 comes handy, so we are going to use it.

The code below maps `IO APIC` to the virtual address 0, and obtains interrupt vector that corresponds to some given `IRQ`:

```
//map ioapic - make sure that we map
//it to non-cached memory.
_asm{
mov ebx,0xfec00000
or ebx,0x13
mov eax,0xc0000000
mov dword ptr[eax],ebx
}//now we are about to get
//interrupt vector
PULONG array=NULL;
//write 0x10+2*irq to IOREGSEL
array[0]=0x10+2*irq;
// subsequent read from IOWIN returns 32
// low-order bits of Redirection Table
//that corresponds to our IRQ.
// 8 low-order bits are interrupt vector,
// corresponding to our IRQ
DWORD vector=(array[4]&0xff);
```

As you can see, `IO APIC` programming is among those things that are easily done than explained - so much explanation and only few simple lines of code. But why did we choose to map `IO APIC` to 0, rather than to some more conventional address? Just because the address 0 is guaranteed to be unused, so mapping `IO APIC` to this address is the very first thing that gets into the head.

#### Putting it all together

Now let's put it all together. Look at the code below - it calls the kernel function:

```
// now we will get interrupt vectors
DWORD res;
DWORD resultarray[24];
ZeroMemory(resultarray,sizeof(resultarray));
for (x=0;x<25;x++){
//let's call the function via the
//call gate. Are you ready???
WORD   farcall[3];
farcall[2] = (selector<<3);
_asm    {
mov ebx,x
push ebx
call fword ptr [farcall]
mov res,eax
}
if(x==24)break;
//if the return value is 500 and this
//was not the final invocation,
//apic is not present. Inform the user
//about it, and that't it
if(res==500)    {
MessageBox(GetDesktopWindow(),           "APIC is not supported",            "IRQs",MB_OK);
break;    }
resultarray[x]=res;
}
```



## Remote Call and Kernel Function Implementation

There is no way to make a far call via the call gate in C, so we have no option other than calling the kernel function from an ASM block. The client code in itself is straightforward - it pushes the value of `IRQ` on the stack, calls the kernel function via the call gate, and saves the result in the array. It does so for `IRQs` 0 to 23, plus makes a final invocation with non-existent `IRQ24`. Upon the receipt of 24 as a parameter, in order to make sure that no traces of our experiments are left anywhere, the kernel function cleans up the call gate in `GDT`. After having obtained all the information about all `IRQs`, we will inform the user about each `IRQ` with `MessageBox()`. I hope there is no need to list this code here.

Now let's look at our kernel function:

```c
void kernelfunction(DWORD usercs, DWORD irq) {    
    DWORD absent = 0;     
    BYTE gdtr[8];          

    //check if ioapic is     
    //present and enabled         

    if(irq <= 23) {       
    __asm {                 
        mov eax, 1            
        cpuid                 
        and edx, 0x00000200             
        cmp edx, 0                 
        jne skip1                 
        mov absent, 1             
        skip1: mov ecx, 0x1b             
        rdmsr                 
        and eax, 0x00000800             
        cmp eax, 0             
        jne skip2             
        mov absent, 1       
    }                 

    //if APIC is enabled, get vector         
    //from it and return        
    skip2: if(!absent) {          
        //map ioapic - make sure that we         
        //map it to non-cached memory.          
        //Certainly, we have to do it only upon the          
        //function's very first invocation,          
        //i.e. when irq is 0                      

        if(!irq) {          
        __asm {                         
            mov ebx, 0xfec00000             
            or ebx, 0x13             
            mov eax, 0xc0000000             
            mov dword ptr[eax], ebx           }           
        }                                 

        //now we are about to get           
        //interrupt vector         

        PULONG array = NULL;                

        //write 0x10+2*irq to IOREGSEL         

        array[0] = 0x10 + 2*irq;                

        // subsequent read from IOWIN returns         

        // 32 low-order bits of Redirection Table         

        //that corresponds to our IRQ.          

        // 8 low-order bits are interrupt vector,          

        // corresponding to our IRQ          

        DWORD vector = (array[4] & 0xff);                   

        // return interrupt vector. Dont forget         

        // that we must return with RETF,         

        // and pop 4 bytes off the stack                    

        __asm {                        
            //                     

            mov eax, vector             

            mov esp, ebp             

            pop ebp             

            retf 4           }       
    }    
    }         

    //either apic is not supported, or irq is     

    //above 23, i.e. this is the last invocation     

    //therefore, clean up gdt and return 500     

    __asm {        
        //clean up gdt         

        sgdt gdtr         

        lea eax, gdtr         

        mov ebx, dword ptr[eax + 2]         

        mov eax, 0         

        mov ax, selector         

        shl eax, 3         

        add ebx, eax         

        mov dword ptr[ebx], 0         

        mov dword ptr[ebx + 4], 0         

        // adjust stack and return         

        mov eax, 500         

        mov esp, ebp         

        pop ebp         

        retf 4    
    }
}
```

Once our kernel function declares the local variables, and, hence, needs a standard function prolog, it does not make sense to write it as a naked routine. Our function is supposed to take only 1 parameter, but, once it is going to get invoked via the call gate, the CPU will push the value of user-mode `CS` on the stack below the return address. Do you know a way of explaining it to the compiler? Me neither. Therefore, to make sure that the compiler generates the code properly, we present this extra value on the stack as a function parameter - we are going to ignore it anyway. If `IRQ` parameter is 24, i.e. this is the function's final invocation, or if `APIC` is disabled, `kernelfunction()` cleans up `GDT` and returns with the error code 500. If everything is OK, it maps `IO APIC` to the virtual address 0, obtains interrupt vector, corresponding to `IRQ` parameter, and returns this vector. There is nothing special here. The only thing worth mentioning is that we have to restore `EBP` and `ESP` registers before we return - this is very important. It is understandable that we have to return with `RETF` instruction, and to pop 4 bytes off the stack.

### SMP Compatibility and Thread Affinity

There is one more thing left to deal with - we have to make sure that our code is suitable for running on both uni-processor and `SMP` machines. With the advent of hyperthreading technology (`HT`), we should always make an assumption that our code may run on an `SMP` machine - CPU that supports `HT` is treated as two independent processors by Windows, and, as far as I am concerned, Intel does not produce CPUs without support for `HT` any more. Under Windows running on an `SMP` machine, every CPU has its own `GDT`, and any thread may run on any CPU in the system by default. I hope you can imagine the mess we are guaranteed to create by allowing our code to be executed by different processors - we may set up a call gate while running on CPU A and try to enter the kernel while running on CPU B. Therefore, we have to prevent our code from running on more than one processor. This can be done in the following way (`go()` is the function that runs all the user-mode code that you've seen in this article):

```c
DWORD dw;

HANDLE thread = CreateThread(0, 0,            
    (LPTHREAD_START_ROUTINE)go,            
    0, CREATE_SUSPENDED, &dw);

SetThreadAffinityMask(thread, 1);

ResumeThread(thread);
```

If the target machine has more than one processor, calling `SetThreadAffinityMask()` makes sure that our code is allowed to run only on the processor that we have specified. Calling `SetThreadAffinityMask()` on a uni-processor machine does not result in error - this call just has no effect. Therefore, the above adjustment is suitable for both uni-processor and `SMP` machines.

## Conclusion

In conclusion I want to say a few words of warning. First of all, the functionality of our privileged code will always be limited, compared to that of a conventional kernel-mode driver. You already know that not all `ntoskrnl.exe`'s exports may be safely called by our code (`MmMapIoSpace()` is just one example). Therefore, you should use this approach sparingly. Second, I would not advise you to use any of these tricks in production applications - they are intended to be used only in the development of analysis tools, intrusion detection systems and other "unsupported" software. The problem with all unsupported tricks is that they may be system-specific. To make things even worse, they may be hardware-specific - the code that does not pose even a slightest problem on machine A can crash machine B, even if they both run the same version of Windows. Therefore, if your code works perfectly well on your development machine, it is too early to celebrate victory - you never know how it may behave on some other platform.



The sample application has been thoroughly tested on my machine, which runs Windows XP SP2 - it works perfectly well and does not seem to pose even a slightest problem. However, I really don't know how it is going to behave on your system - it is your task to find it out. If something goes wrong, don't hesitate to inform me about it. In such cases, it would be great if you could provide me with some info about your machine (CPU, motherboard, `OS` version, etc.), as well as the description of the problem - who knows, maybe there are some more hidden bugs that are to be fixed. In order to run the sample, the only thing you have to do is to click on its bitmap, and then wait for message boxes - it may take a few seconds before they pop up, so you have to be patient.

I would highly appreciate if you send me an e-mail with your comments and suggestions.

Anton Bassov
