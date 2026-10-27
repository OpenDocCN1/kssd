
# [原创]Windows内核学习笔记之内存管理

## 一.Windows内存管理概述

Windows采用页式内存管理方案，在Intel x86处理器上，Windows不适用段来管理虚拟内存，但是，Intel x86处理器在访问内存时必须要通过段描述符，这意味着Windows将所有的段描述符都构造成了从基地址0开始，且段的大小根据段的用户和系统设置的不同，会设置为`0x80000000`，`0xC0000000`或`0xFFFFFFFF`。所以，Windows系统中的代码，包括操作系统本身的代码和应用程序的代码，所面对的地址空间都是线性地址空间。这种做法相当于屏蔽了处理器中的逻辑地址概念，段只被用于访问控制和内存保护。

Windows使用了两种特权级别：0和3，其中特权级0称为内核模式，特权级3称为用户模式。当处理器指向内核模式代码时，它们处于系统地址空间，位于`0x80000000~0xFFFFFFFF`，所有的进程共享此空间；当处理器执行用户模式代码时，它们处于进程地址空间，位于`0x00000000~0x7FFFFFFF`，这部分空间是进程私有的。用户模式代码只能访问进程自身的数据，而内核模式代码不仅可以访问当前进程的数据，也可也访问系统地址空间中的数据。所有的进程，一旦进入到内核模式，则共享同一的系统地址空间。

在Windows的每个地址空间中，虚拟地址的分配和回收都必须按照规定执行。Windows规定，应用程序在使用内存以前必须先申请，所以，操作系统内部可以根据应用程序的申请和释放操作来维护好整个虚拟地址空间的内存分配情况。而且，Windows也采用了按需分配的策略，也就是说，只有当一段虚拟地址空间真正被使用的时候，系统才会为它分配页表和物理页面。每个进程的虚拟地址空间的分配情况通过一组虚拟地址描述符（`VAD`）记录下来，这些描述符构成了有一颗平衡二叉树，以便于快速地定位到一个指定虚拟地址地描述符上。

## 二.Windows系统内存管理

### 1.概述

在系统地址空间中，有些部分提供一些特殊模块使用，比如会话空间是由会话管理器和Windows子系统使用的；而换页内存池和非换页内存池则是提供给系统内核模块和设备驱动程序使用的。在换页内存池中分配的内存有可能在物理内存紧缺的情况瞎被换出到外存中；而非换页内存池中分配的内存总是处于物理内存中。为了实现这两种池，Windows使用了两层内存管理。下层是基于页面的内存管理，仅限于执行体内部使用；上层建立在下层的内存管理功能基础之上，对外提供各种粒度的内存服务。两层结果如下图所示

![avatar](upload/attach/202112/835440\_K6GEF9ZPB6H7QBA.png)

系统换页内存池和非换页内存池是Windows系统提供的最基本动态内存管理手段，它以页面为基本粒度来管理系统中划定的地址范围。

页面粒度对于一般的代码逻辑而言太大了。所以，为了适应各种内核组件对于内存管理的需要，Windows内核在系统的非换页内存池和换页内存池的基础上，实现了灵活的，可适应各种大小内存需求的内存池，这便是执行体内存池。

执行体内存对象是由数据结构`POOL_DESCRIPTOR`来描述的，定义如下：

```php
kd> dt \_POOL\_DESCRIPTOR
nt!\_POOL\_DESCRIPTOR
+0x000 PoolType : \_POOL\_TYPE
+0x004 PoolIndex : Uint4B
+0x008 RunningAllocs : Uint4B
+0x00c RunningDeAllocs : Uint4B
+0x010 TotalPages : Uint4B
+0x014 TotalBigPages : Uint4B
+0x018 Threshold : Uint4B
+0x01c LockAddress : Ptr32 Void
+0x020 PendingFrees : Ptr32 Void
+0x024 PendingFreeDepth : Int4B
+0x028 ListHeads : [512] \_LIST\_ENTRY
```

| 偏移 | 名称 | 作用 |
|:------:|:---------:|:--------------------------------------------------------------:|
| `0x000` | `PoolType` | 枚举类型`POOL_TYPE`，其中的`NonPagedPool`代表了执行体非分页内存池，`PagedPool`代表执行体的分页内存池 |
| `0x004` | `PoolIndex` | 指当前对象在该类型的内存池数组中的索引，对于换页内存池，该域是它在`ExpPagedPoolDescriptor`数组中的下标值 |
| `0x028` | `ListHeads` | 执行体内存池对于小内存的分配和回收，是通过一组空间内存块链表实现的 |

以下是WRK中和内存管理有关的一部分全局变量:

```c
#if defined(NT\_UP)
#define NUMBER\_OF\_PAGED\_POOLS 2
#else
#define NUMBER\_OF\_PAGED\_POOLS 4
#endif
ULONG ExpNumberOfPagedPools = NUMBER\_OF\_PAGED\_POOLS;
ULONG ExpNumberOfNonPagedPools = 1;
POOL\_DESCRIPTOR NonPagedPoolDescriptor;
#define EXP\_MAXIMUM\_POOL\_NODES 16
PPOOL\_DESCRIPTOR ExpNonPagedPoolDescriptor[EXP\_MAXIMUM\_POOL\_NODES];
#define NUMBER\_OF\_POOLS 2
PPOOL\_DESCRIPTOR PoolVector[NUMBER\_OF\_POOLS];
PPOOL\_DESCRIPTOR ExpPagedPoolDescriptor[EXP\_MAXIMUM\_POOL\_NODES + 1];
volatile ULONG ExpPoolIndex = 1;
```

`ExpNumberOfNonPagePools`代表非换页内存池数量，只有一个；而换页内存池的数量，即变量`ExpNumberOfPagedPools`，在单处理器系统上是3个，在多处理器系统上有5个。`PoolVector`是一个包含两个数组的元素，第一个元素`PoolVector[0]`指向非换页内存池，即`NonPagedPoolDescriptor`的地址；第二个元素`PoolVector[1]`指向换页内存池，即指向第一个换页内存池。数组`ExpNonPagedPoolDescriptor`仅用于有多个非换页内存池的情形；数组`ExpPagedPoolDescriptor`中的每个元素分别指向一个换页内存池。

执行体内存池对象的初始化是按照换页内存池和非换页内存池分开进行的，在Windows系统内核中通过一些列内核函数来使用内存池对象。

### 2.初始化内存池

执行体内存池的初始化是在`InitializePool`函数中完成的，函数首先会判断要初始化的内存池是否是非分页内存池

```nasm
INIT:005EB2F9 ; int \_\_stdcall InitializePool(POOL\_TYPE PoolType, ULONG Threshold)
INIT:005EB2F9 \_InitializePool@8 proc near ; CODE XREF: MiInitMachineDependent(x)-1FE3↑p
INIT:005EB2F9 ; MiBuildPagedPool()+276↓p
INIT:005EB2F9
INIT:005EB2F9 PoolType = dword ptr 8
INIT:005EB2F9 Threshold = dword ptr 0Ch
INIT:005EB2F9
INIT:005EB2F9 mov edi, edi
INIT:005EB2FB push ebp
INIT:005EB2FC mov ebp, esp
INIT:005EB2FE push ebx
INIT:005EB2FF xor ebx, ebx ; ebx清0
INIT:005EB301 cmp [ebp+PoolType], ebx ; PoolTpe是否为NonPagedPool
INIT:005EB304 push esi
INIT:005EB305 push edi
INIT:005EB306 jnz loc\_5EB21C
```

如果要初始化非分页内存池，就会判断`KeNumberNodes`中的数据是否大于1，根据后面代码的内容可推测这个变量保存的是处理器的核数

```nasm
INIT:005EB319 loc\_5EB319: ; CODE XREF: InitializePool(x,x)+10BB9↓j
INIT:005EB319 mov al, \_KeNumberNodes
INIT:005EB31E cmp al, 1
INIT:005EB320 ja loc\_5FBEB7 ; al是否大于1
```

如果大于1，就会修改全局变量`ExpNumberOfNonPagedPools`保存的数据

```nasm
INIT:005FBEB7 loc\_5FBEB7: ; CODE XREF: InitializePool(x,x)+27↑j
INIT:005FBEB7 movzx eax, al
INIT:005FBEBA cmp eax, 7Fh
INIT:005FBEBD mov ds:\_ExpNumberOfNonPagedPools, eax
INIT:005FBEC2 jbe short loc\_5FBECC
INIT:005FBEC4 push 7Fh
INIT:005FBEC6 pop eax
INIT:005FBEC7 mov ds:\_ExpNumberOfNonPagedPools, eax
INIT:005FBECC
INIT:005FBECC loc\_5FBECC: ; CODE XREF: InitializePool(x,x)+10BC9↑j
INIT:005FBECC cmp eax, 10h
INIT:005FBECF jbe short loc\_5FBED9
INIT:005FBED1 push 10h
INIT:005FBED3 pop eax
INIT:005FBED4 mov ds:\_ExpNumberOfNonPagedPools, eax
```

调用`ExInitializePoolDescriptor`来初始化非分页内存池对象，该内存池对象的地址则是全局变量`NonPagedPoolDescriptor`，且在初始化之前会将地址保存到`PoolVector`数组中的第一个元素

```nasm
INIT:005EB326 loc\_5EB326: ; CODE XREF: InitializePool(x,x)+10BE4↓j
INIT:005EB326 ; InitializePool(x,x)+10C28↓j
INIT:005EB326 push offset \_ExpTaggedPoolLock ; SpinLock
INIT:005EB32B call \_KeInitializeSpinLock@4 ; KeInitializeSpinLock(x)
INIT:005EB330 push ebx ; PoolLock
INIT:005EB331 push [ebp+Threshold] ; Threshold
INIT:005EB334 mov eax, offset \_NonPagedPoolDescriptor ; 将NonPagedPoolDescriptor地址赋给eax
INIT:005EB339 push ebx ; PoolIndex
INIT:005EB33A push ebx ; PoolType
INIT:005EB33B push eax ; int
INIT:005EB33C mov \_PoolVector, eax ; 将NonPagedPoolDescriptor赋给PoolVector[0]
INIT:005EB341 call \_ExInitializePoolDescriptor@20 ; ExInitializePoolDescriptor(x,x,x,x,x)
INIT:005EB346
INIT:005EB346 loc\_5EB346: ; CODE XREF: InitializePool(x,x)-16↑j
INIT:005EB346 ; InitializePool(x,x)-7↑j ...
INIT:005EB346 pop edi
INIT:005EB347 pop esi
INIT:005EB348 pop ebx
INIT:005EB349 pop ebp
INIT:005EB34A retn 8
INIT:005EB34A \_InitializePool@8 endp
```

如果要初始化分页内存池，首先依然会判断`KeNumberNodes`中保存的数据是否大于1

```nasm
INIT:005EB21C loc\_5EB21C: ; CODE XREF: InitializePool(x,x)+D↓j
INIT:005EB21C mov cl, \_KeNumberNodes
INIT:005EB222 cmp cl, 1
INIT:005EB225 ja loc\_5FBF3E ; KeNumberNodes是否大于1
```

如果大于1，则修改`ExpNumberOfPagedPools`中保存的数据

```nasm
INIT:005FBF3E loc\_5FBF3E: ; CODE XREF: InitializePool(x,x)-D4↑j
INIT:005FBF3E movzx eax, cl
INIT:005FBF41 cmp eax, 7Fh
INIT:005FBF44 mov ds:\_ExpNumberOfPagedPools, eax
INIT:005FBF49 jbe loc\_5EB230
INIT:005FBF4F push 7Fh
INIT:005FBF51 pop eax
INIT:005FBF52 mov ds:\_ExpNumberOfPagedPools, eax
INIT:005FBF57 jmp loc\_5EB230
INIT:005FBF5C ; ---------------------------------------------------------------------------
INIT:005FBF5C
INIT:005FBF5C loc\_5FBF5C: ; CODE XREF: InitializePool(x,x)-C6↑j
INIT:005FBF5C push 10h
INIT:005FBF5E pop eax
INIT:005FBF5F mov ds:\_ExpNumberOfPagedPools, eax
INIT:005FBF64 jmp loc\_5EB239
```

调用函数来申请一块内存，由于此时`ebx`等于0，所以申请的是非分页内存，而申请的内存的大小则是`ExpNumberOfPagedPools`的大小加1乘以`0x1048`的大小

```nasm
INIT:005EB242 lea esi, [eax+1]
INIT:005EB245 imul esi, 1048h
INIT:005EB24B push 'looP' ; Tag
INIT:005EB250 push esi ; NumberOfBytes
INIT:005EB251 push ebx ; PoolType
INIT:005EB252 call \_ExAllocatePoolWithTag@12 ; ExAllocatePoolWithTag(x,x,x)
INIT:005EB257 mov edi, eax ; 要获取的内存赋值赋给esi
INIT:005EB259 cmp edi, ebx
INIT:005EB25B jz loc\_5FBFF2
```

这里申请的内存大小之所以要乘以`0x1048`，是因为要保存的结构体除了内存池对象的`0x1028`字节大小，还要申请`FAST_MUTEX`对象的`0x20`大小，该对象结构体如下：

```php
kd> dt \_FAST\_MUTEX
nt!\_FAST\_MUTEX
+0x000 Count : Int4B
+0x004 Owner : Ptr32 \_KTHREAD
+0x008 Contention : Uint4B
+0x00c Event : \_KEVENT
+0x01c OldIrql : Uint4B
```

将内存池对象地址赋给`PoolVector`数组下标为1的地址，将`FAST_MUTEX`对象地址赋给全局变量`ExpPagedPoolMutex`，并将参数`PoolType`修改为0，因为此时`ebx`等于0

```nasm
INIT:005EB261 mov eax, \_ExpNumberOfPagedPools
INIT:005EB266 lea esi, [eax+1]
INIT:005EB269 imul esi, 1028h ; 计算内存池对象占用的内存大小
INIT:005EB26F add esi, edi ; 将esi指向FAST\_MUTEXT对象地址
INIT:005EB271 inc eax
INIT:005EB272 mov \_PoolVector+4, edi
INIT:005EB278 mov \_ExpPagedPoolMutex, esi
INIT:005EB27E mov [ebp+PoolType], ebx ; 将PoolType赋值为1
INIT:005EB281 jz short loc\_5EB2CD
```

将填充申请到的`ExpNumberOfPagedPools+1`个的内存池对象和`FAST_MUTEX`对象
