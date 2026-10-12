
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
