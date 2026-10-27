
![图片描述](upload/attach/202010/747296_FFBFFHDWUHDPXF6.png)

我是怎么找到他的呢？很简单，还是看的`MiInitializeNonPagedPool`，发现用了这个数组的0号成员，很明显这个数组就是存放各种区域的地址范围的，并且Nt初始化时这个数组也早已被初始化了

![图片描述](upload/attach/202010/747296_Q27YW3UTQMKFCM5.png)

通过一些猜测与符号的查找，发现这个数组结构体以及宏定义如下

```
enum _MI_ASSIGNED_REGION_TYPES
{
AssignedRegionNonPagedPool = 0,
AssignedRegionPagedPool = 1,
AssignedRegionSystemCache = 2,
AssignedRegionSystemPtes = 3,
AssignedRegionUltraZero = 4,
AssignedRegionPfnDatabase = 5,
AssignedRegionCfg = 6,
AssignedRegionHyperSpace = 7,
AssignedRegionKernelStacks = 8,
AssignedRegionPageTables = 9,
AssignedRegionSession = 10,
AssignedRegionSecureNonPagedPool = 11,
AssignedRegionSystemImages = 12,
AssignedRegionMaximum = 13
};
typedef struct _MI_SYSTEM_VA_ASSIGNMENT
{
VOID* BaseAddress; //0x0
ULONGLONG NumberOfBytes; //0x8
} MI_SYSTEM_VA_ASSIGNMENT, * PMI_SYSTEM_VA_ASSIGNMENT;
```

在NT内核初始化时，会将`Assignment`数组里的地址范围全部应用到系统中。

那么找这个数组就用`MiQuerySytemBase`的特征码就行了。

```
MmsScanSection(".text",
(PCUCHAR)"\x48\x63\xC1\x48\x8D\x0D\xCC\xCC\xCC\xCC\x48\x03\xC0\x48\x8B\x04\xC1\xC3",
0xCC,
18,
(PVOID)&lpTargetAddr);
if (!lpTargetAddr)
{
DbgPrint("MemScanner: %s: MmsScanSection Failed\n", __FUNCTION__);
return STATUS_NOT_FOUND;
}
lpMiSystemVaAssignment = (PMI_SYSTEM_VA_ASSIGNMENT)((PUCHAR)lpTargetAddr + *(PULONG)((PUCHAR)lpTargetAddr + 6) + 10);
for (ulIndex = 0; ulIndex < AssignedRegionMaximum; ulIndex++)
{
DbgPrint("MemScanner: %s: Names:%s, BaseAddr:%I64x, Size:%I64x\n",
__FUNCTION__,
g_szAssignedRegionNames[ulIndex],
lpMiSystemVaAssignment[ulIndex].BaseAddress,
lpMiSystemVaAssignment[ulIndex].NumberOfBytes);
}
```

不过...

最让人头疼的事情是...

```
Win10 RS1
enum _MI_ASSIGNED_REGION_TYPES
{
AssignedRegionNonPagedPool = 0,
AssignedRegionPagedPool = 1,
AssignedRegionSystemCache = 2,
AssignedRegionSystemPtes = 3,
AssignedRegionUltraZero = 4,
AssignedRegionPfnDatabase = 5,
AssignedRegionCfg = 6,
AssignedRegionHyperSpace = 7,
AssignedRegionPageTables = 8,
AssignedRegionSpecialPool = 9,
AssignedRegionSession = 10,
AssignedRegionWorkingSetPagedPool = 11,
AssignedRegionWorkingSetSystemCache = 12,
AssignedRegionSystemImages = 13,
AssignedRegionMaximum = 14
};
Win10 RS2
enum _MI_ASSIGNED_REGION_TYPES
{
AssignedRegionNonPagedPool = 0,
AssignedRegionPagedPool = 1,
AssignedRegionSystemCache = 2,
AssignedRegionSystemPtes = 3,
AssignedRegionUltraZero = 4,
AssignedRegionPfnDatabase = 5,
AssignedRegionCfg = 6,
AssignedRegionHyperSpace = 7,
AssignedRegionPageTables = 8,
AssignedRegionSpecialPoolPaged = 9,
AssignedRegionSpecialPoolNonPaged = 10,
AssignedRegionSession = 11,
AssignedRegionSystemImages = 12,
AssignedRegionMaximum = 13
};
Win10 RS3
enum _MI_ASSIGNED_REGION_TYPES
{
AssignedRegionNonPagedPool = 0,
AssignedRegionPagedPool = 1,
AssignedRegionSystemCache = 2,
AssignedRegionSystemPtes = 3,
AssignedRegionUltraZero = 4,
AssignedRegionPfnDatabase = 5,
AssignedRegionCfg = 6,
AssignedRegionHyperSpace = 7,
AssignedRegionKernelStacks = 8,
AssignedRegionPageTables = 9,
AssignedRegionSpecialPoolPaged = 10,
AssignedRegionSpecialPoolNonPaged = 11,
AssignedRegionSession = 12,
AssignedRegionSystemImages = 13,
AssignedRegionMaximum = 14
};
Win10 RS4
enum _MI_ASSIGNED_REGION_TYPES
{
AssignedRegionNonPagedPool = 0,
AssignedRegionPagedPool = 1,
AssignedRegionSystemCache = 2,
AssignedRegionSystemPtes = 3,
AssignedRegionUltraZero = 4,
AssignedRegionPfnDatabase = 5,
AssignedRegionCfg = 6,
AssignedRegionHyperSpace = 7,
AssignedRegionKernelStacks = 8,
AssignedRegionPageTables = 9,
AssignedRegionSpecialPoolPaged = 10,
AssignedRegionSpecialPoolNonPaged = 11,
AssignedRegionSession = 12,
AssignedRegionSystemImages = 13,
AssignedRegionMaximum = 14
};
Win10 RS5
enum _MI_ASSIGNED_REGION_TYPES
{
AssignedRegionNonPagedPool = 0,
AssignedRegionPagedPool = 1,
AssignedRegionSystemCache = 2,
AssignedRegionSystemPtes = 3,
AssignedRegionUltraZero = 4,
AssignedRegionPfnDatabase = 5,
AssignedRegionCfg = 6,
AssignedRegionHyperSpace = 7,
AssignedRegionKernelStacks = 8,
AssignedRegionPageTables = 9,
AssignedRegionSpecialPoolPaged = 10,
AssignedRegionSpecialPoolNonPaged = 11,
AssignedRegionSession = 12,
AssignedRegionSystemImages = 13,
AssignedRegionMaximum = 14
};
Win10 19H1
enum _MI_ASSIGNED_REGION_TYPES
{
AssignedRegionNonPagedPool = 0,
AssignedRegionPagedPool = 1,
AssignedRegionSystemCache = 2,
AssignedRegionSystemPtes = 3,
AssignedRegionUltraZero = 4,
AssignedRegionPfnDatabase = 5,
AssignedRegionCfg = 6,
AssignedRegionHyperSpace = 7,
AssignedRegionKernelStacks = 8,
AssignedRegionPageTables = 9,
AssignedRegionSession = 10,
AssignedRegionSystemImages = 11,
AssignedRegionMaximum = 12
};
Win10 19H2
enum _MI_ASSIGNED_REGION_TYPES
{
AssignedRegionNonPagedPool = 0,
AssignedRegionPagedPool = 1,
AssignedRegionSystemCache = 2,
AssignedRegionSystemPtes = 3,
AssignedRegionUltraZero = 4,
AssignedRegionPfnDatabase = 5,
AssignedRegionCfg = 6,
AssignedRegionHyperSpace = 7,
AssignedRegionKernelStacks = 8,
AssignedRegionPageTables = 9,
AssignedRegionSession = 10,
AssignedRegionSystemImages = 11,
AssignedRegionMaximum = 12
};
Win10 20H1
enum _MI_ASSIGNED_REGION_TYPES
{
AssignedRegionNonPagedPool = 0,
AssignedRegionPagedPool = 1,
AssignedRegionSystemCache = 2,
AssignedRegionSystemPtes = 3,
AssignedRegionUltraZero = 4,
AssignedRegionPfnDatabase = 5,
AssignedRegionCfg = 6,
AssignedRegionHyperSpace = 7,
AssignedRegionKernelStacks = 8,
AssignedRegionPageTables = 9,
AssignedRegionSession = 10,
AssignedRegionSecureNonPagedPool = 11,
AssignedRegionSystemImages = 12,
AssignedRegionMaximum = 13
};
```

尽管8号之前没什么变化，但是后面序号一直在变，也不知道微软程序员抽了什么风...

最后看下Win10的测试结果吧，由于序号变得快，调试信息有不准的地方我也没做修改了...

![图片描述](upload/attach/202010/747296_AJTG9GPDE33VFJV.png)

结论还是与Win8.1一致，`MmMapViewInSystemSpace`映射在`DriverImage`区域，MDL与连续物理页的虚拟页映射在`SystemPtes`区域。

五点了，睡觉睡觉!

# 前言

---

内存扫描一直都是比较根本的威胁检测手法，对于顽固型木马外挂而言，依赖于操作系统API相关的传统检测方式，模块扫描，窗口扫描，进程扫描以及文件扫描等都显得较为吃力，应用层而言如此，到内核层亦是如此。对于x86内核而言，`0x80000000~0xFFFFFFFF` 2GB内存扫描检测不是什么难事，因为整个区域就这么大。但是对于x64内核空间而言，就没有这么简单，尽管64位地址高16位一直都是保留字节`0xffff`，48位地址虚拟内存总量也大的有些离谱。或许有人会说物理内存就这么大，现在见的比较大的是16GB、32GB内存条，直接扫描物理内存不就好了。起初我也这么想，但是最主要的是要访问物理内存本质上需要通过虚拟内存经过页表转换到MMU单元才能完成对物理页的访问，而且抛开这个层面，尽管可以直接访问物理内存最终我们也是需要转为虚拟内存地址。那么本篇就来探索下从Win7开始到Win10 x64内核的虚拟地址空间布局以及经历了哪些更新变化。

---

# 项目传送门

[黑暗之门](https://bbs.kanxue.com/elink@958K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3j5f1g2J5P5f1W2o6c8g2)9J5c8V1#2W2L8g2y4U0j5h3&6F1k6i4t1`.)

---

# 一. WIN7/WIN8 x64固定布局

---

WIN7相关资料也比较多，WIN8总体上与WIN7很像，就放在一起介绍了。推荐下面两篇文章

原文版本[Kernel Virtual Address Layout](https://bbs.kanxue.com/elink@f52K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2L8h3q4U0K9r3W2F1k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4y4h3k6^5y4U0c8C8N6X3q4K6i4K6u0W2K9s2c8E0L8l9`.`.)

翻译版本[内核虚拟地址空间布局](https://bbs.kanxue.com/elink@b04K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2y4e0q4U0N6r3!0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3q4&6K9e0p5&6z5o6y4Q4x3V1j5I4y4K6x3@1z5o6t1J5)

引用下这张图

![](https://bbs.kanxue.com/upload/attach/202010/747296_9MCE4ARFC4F2SWS.png)

- PTE Space 顾名思义**页表所在区域**

- Hyper Space 姑且称为**超区域**，主要是操作系统做中转用的，不同页表场景下需要用这块区域临时映射；

- Shared System Page **共享内存空间**，会映射到每一个进程空间，应用层地址即可访问内核层数据；

- System Cache Working Set **系统缓存工作集**，没怎么研究过，还有个`SystemCache`区域，老容易搞混...

- Initial Loader Mappings **最初的加载器映射区域**，由加载器初始化。在系统引导阶段，`winload.exe` 将 `NTOSKRNL.EXE`，`HAL.DLL`，以及内核调试器 DLL（`KDCOM`, `KD1394`, `KDUSB`）加载到此区域。该区域还包含了空闲线程的栈，DPC（延迟过程调用）的栈，以及 KPCR（内核处理器控制区，Kernel Processor Control Region）和 Idle 线程的数据结构。

- Sys PTEs **系统页表条目区域** 重点哦，***MDL映射的虚拟内存，驱动映像都在这里***。

- Paged Pool Area **分页内存区域** 注意区分`SpecialPagedPool`。

- Dynamic Kernel VA Spacce **动态内存区域** `MiVaSystemCache`(注意前面还有一个缓存工作集), `MiVaSpecialPoolPaged`(特殊池，特殊池还会细分`PagedPool`与`NonpagedPool`)

- PFN Database **PFN 数据库** 别的我不知道，`MmGetVirtualForPhysical`找虚拟地址的时候访问PFN数据库容易蓝屏

- Non-paged Pool **非分页内存区域** 不会被倒换到磁盘空间，只要有物理内存映射，只要不被申请者释放一般都很稳定，注意这个地址需要获取!

那我们看看怎么获取`MmNonPagedPoolStart`与`MmNonpagedPoolEnd`吧：

固定思路先从`MiInitializeNonPagedPool`函数找到未导出变量`MmNonPagedPoolStart`，发现他在`MiCreatePfnDatabase`里初始化
