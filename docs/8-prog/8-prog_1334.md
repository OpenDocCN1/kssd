
# RING 3 无驱动进入 RING 0

> 原文：[`www.pediy.com/kssd/pediy10/65906.html`](https://www.pediy.com/kssd/pediy10/65906.html)

可能这些已经没有什么，但对俺菜鸟来说学习一下应该还可以的。

MY BLOG：[`hi.baidu.com/hljleo`](http://hi.baidu.com/hljleo)

MY QQ：554920269

下面就是 RING 3 无驱动怎么进入 RING 0 的知识要点

## 1. 通过工作区 `\\Device\\PhysicalMemory` 操作内存

通过工作区 `\\Device\\PhysicalMemory` 操作内存，用到的函数流程：`NtOpenSection()`，`NtMapViewOfSection()`。所以依靠这种方法我们可以进入 RAM 的任何地址。

然而在使用这个工作区之前必须保证我们有权限使用它，可以通过一些函数来获得管理员权限：`GetSecurityInfo()`，`SetEntriesInAcl()`，`SetSecurityInfo()`。

## 2. 虚拟地址到物理地址的映射

在进入 RING 0 之前我们还必须知道目标虚拟地址映射后的物理地址的准确位置，所以我们必须找到进程的页目录。这里涉及到虚拟地址到物理地址的映射，简单解释一下：

* **A** CPU 从当前运行进程的 `CR3` 寄存器中得到页目录。在这个目录中，虚拟地址的高 10 位让 CPU 找到页表的地址。
* **B** 当找到了页表之后，在这张表中，CPU 根据相应的虚拟地址进入下一步定位页。
* **C** 当定位好了页之后，CPU 使用虚拟地址的低 12 位作为页的偏移量。

所以如果我们知道了页目录的位置，我们也就知道了虚拟地址映射后的物理位置。页目录地址存在 `CR3` 寄存器中，我们是没有权限在 user-mode 直接获得的。

以下就是如何获得页目录的一些知识：

现在假设我们知道我们进程的页目录在物理 RAM 中的地址。假使我们的目标虚拟地址是 `V`，映射页目录到虚拟地址 `D`，把 `D` 认为是 1024 个 `DWORD`。因此 `D` 中的 `(V>>22)` 项的高 20 位是物理页，一张页表，对应的虚拟地址 `V`。现在映射这张页表到虚拟地址 `T`，把 `T` 想象成一个 1024 `DWORD` 的数组，`T` 中第 `((V>>12)&0x3FF)` 项的高 20 位是虚拟地址 `V` 所代表的一个物理页，也就是我们目标虚拟地址映射后的物理地址。

在 X86，进程的页目录是从虚拟地址 `0XC0300000` 开始，内存管理器从 `0XC0000000` 开始映射页表的。

问题的关键在于我们不知道进程页目录的物理地址，为找到页目录，必须做如下工作：

将 RAM 中的每一页都映射到我们进程的地址空间中，扫描一下内存，总之我们应该会找到页目录的。

假设我们检查的物理页是 `P`，并且 `P` 被映射进入虚拟地址 `V` 中，我们把 `V` 想象成一个 1024 `DWORD` 的数组。如果 `P` 是我们进程中含有页目录的一个物理页的话，有如下结论：

* **A** `V` 的第 `0x300` 项的高 20 位肯定等于 `P`。
* **B** `V` 中第 `0x300` 项的最低位肯定被设置了，因为它表明了此页存在于 RAM 中。
* **C** 如果 `P` 是一个页目录，那么 `V` 中的第 `(V>>22)` 项表示是一个页表——对应于虚拟地址 `V` 它的本身。同时这个页表毫无疑问的被装载进入 RAM 中。因此，`V` 中第 `(V>>22)` 项的最低位肯定被设置了。

如果找不到就换下一页。

## 3. 获取 GDT 物理地址并设置调用门

知道了页目录的物理地址，我们就可以找到我们所感兴趣的虚拟地址到物理地址的映射，我们是得到 `GDT` 的物理地址。

有了 `GDT` 的物理地址我们就可以无驱动进入内核了。因为我们可以通过 `GDT` 中的调用门来实现不同特权级的代码段之间进行控制访问，每一种门描述符中都有 `present` 位代表该描述符是否存在。它在每个描述符中的位置都是一样的，因此我们可以根据这个 `present` 位来在 `GDT` 中找出一个空白位置，在这个位置里面添加我们自己定义的描述符。

我们可以将描述符的 `selector` 域设置为 `0X8`（表明将会执行特权指令），同时它的 `offset_low` 和 `offse_high` 域分别对应于我们即将要调用的函数的低 16 位和高 16 位。

至于要调用的函数实现的功能这就随个人不同的兴趣而定，比如说可以获得中断信息等等。

下面是主要程序代码的实现：

### （1）寻找页目录

```c
for(x=0;x<meminfo.dwTotalPhys;x+=0x1000)//经过修改后运行这些代码的进程，其虚拟地址的范围从 0--0x1000 将成为进程地址空间中有效的虚拟地址范围
{
    //映射当前页进入 RAM
    MappedSize=4096; phys.QuadPart=x; DirectoryMappedAddress=0;

    status = NtMapViewOfSection(Section, (HANDLE) -1, &DirectoryMappedAddress, 0L, MappedSize, &phys,
    &MappedSize, ViewShare, 0, PAGE_READONLY);

    if(status)continue;

    entry=(DWORD*)DirectoryMappedAddress;

    //得到偏移量
    DirectoryOffset=(DWORD)DirectoryMappedAddress;
    TableOffset=(DWORD)DirectoryMappedAddress;
    DirectoryOffset>>=22;//第>>22 项高 20 位是页表的物理地址
    TableOffset=(TableOffset>>12)&0x3ff;//第>>12)&0x3FF 项的高 20 位是虚拟地址的物理页的位置，虚拟地址中 第（（V>>12）&0x3FF）项的低位必定被设置

    //为了找到页目录，我们需要作出判断，即页目录的第 0x300 项的高 20 位对应的是 我们要找页目录物理地址自身，如果是找到页目录的位置，如果不是进入下一页
    // 当然该项的存在位必须被设置
    if((entry[0x300]&0xfffff000)!=x || (entry[0x300]&1)!=1 || (entry[DirectoryOffset]&1)!=1)
    {
        NtUnmapViewOfSection((HANDLE) -1, DirectoryMappedAddress);
        continue;
    }

    //映射页表
    MappedSize=4096;
    phys.QuadPart=(entry[DirectoryOffset]&0xfffff000);
    TableMappedAddress=0;

    status = NtMapViewOfSection(Section, (HANDLE) -1, &TableMappedAddress, 0L, MappedSize, &phys,
    &MappedSize, ViewShare, 0, PAGE_READONLY);

    if(status){NtUnmapViewOfSection((HANDLE) -1, DirectoryMappedAddress);continue;}

    //如果找到页表---虚拟地址第(V>>12)&0x3ff 页表项的高 20 位必须是页目录的地址
    //当然存在位同样应该被设置，，现在找到真正页目录的位置
    entry=(DWORD*)TableMappedAddress;

    if((entry[TableOffset]&1)==1 && (entry[TableOffset]&0xfffff000)==x)found++;

    NtUnmapViewOfSection((HANDLE) -1, TableMappedAddress);

    //页目录已经找到，退出循环
    if(found)break;

    NtUnmapViewOfSection((HANDLE) -1, DirectoryMappedAddress);
}
```

### （2）获取 GDT 物理地址并设置调用门

知道了页目录的物理地址，我们就可以找到我们所感兴趣的虚拟地址到物理地址的映射，下面是得到 `GDT` 的物理地址，并设置自己的调用门，现在我们就可以无驱动进入内核了。

```asm
_asm
{
    sgdt gdtr   //得到 gdtr 寄存器的内容保存于内存单元中，即得到 GDT 基地址与界限
    lea eax, gdtr
    mov ebx, dword ptr[eax+2] //高 32 位指出 GDT 在物理存取器中存放的基地址
    mov gdtbase, ebx
}
```

```c
DirectoryOffset=gdtbase;
TableOffset=gdtbase;
DirectoryOffset>>=22;
TableOffset=(TableOffset>>12)&0x3ff;//保证 GDT 从一个页开始
entry=(DWORD*)DirectoryMappedAddress;

//映射页表- 第(V-22)页目录项的高 20 位是物理地址
MappedSize=4096;
phys.QuadPart=(entry[DirectoryOffset]&0xfffff000);
TableMappedAddress=0;

status = NtMapViewOfSection(Section, (HANDLE) -1, &TableMappedAddress, 0L, MappedSize, &phys,
&MappedSize, ViewShare, 0, PAGE_READONLY);

//物理页是 第 (V>>12)&0x3ff 页表项的高 20 位 - -这就是我们所想要的
entry=(DWORD*)TableMappedAddress;
physgdtbase=(entry[TableOffset]&0xfffff000);

NtUnmapViewOfSection((HANDLE) -1, TableMappedAddress);
NtUnmapViewOfSection((HANDLE) -1, DirectoryMappedAddress);

// 映射 gdt
phys.QuadPart=physgdtbase;
MappedSize=4096;
NtMapViewOfSection(Section, (HANDLE) -1, (PVOID*)&GdtMappedAddress, L, MappedSize, &phys, &MappedSize,
ViewShare, 0, PAGE_READWRITE);

gdtbase&=0xfff;
GdtMappedAddress+=gdtbase;

gate=(CallGateDescriptor *)GdtMappedAddress;

//在 GDT 中寻找空闲入口，即 Present 位为 0 的
selector=1;
while(1)
{
    if(!gate[selector].present)break;
    selector++;
}

// 设置 call gate
gate[selector].offset_low  = (WORD) ((DWORD)kernelfunction & 0xFFFF);//将会调用任意函数的地址，这里是
                                                                    //kernelfunction 这个函数
gate[selector].selector     = 8;
gate[selector].param_count  = 1; //we will pass a parameter
gate[selector].unused    = 0;
gate[selector].type      = 0xc;   //0xc 是 32-bit callgate 标志
gate[selector].dpl       = 3;     // 必须是 3
gate[selector].present   = 1;
gate[selector].offset_high = (WORD) ((DWORD)kernelfunction >> 16);

NtUnmapViewOfSection((HANDLE) -1, GdtMappedAddress);
CloseHandle(Section);
```

...........这里可以调用一些自己实现的函数。

很多细节的东西应该讲得不大好，希望大牛们能指出来，让俺菜鸟学习学习。

昨天弄的时候忘记上传源码了，，不好意思

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | 进入 RING0.rar |
|---|---|
