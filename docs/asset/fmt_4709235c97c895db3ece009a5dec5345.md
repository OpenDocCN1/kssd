
# NTFSI--the NT Cache Manager 相关(1)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-336.htm`](https://www.pediy.com/kssd/pediy09/pediy09-336.htm)

> \[/b\]\[note\]NTFSI--the NT Cache Manager 相关(1)

> +++++++++++++++++++++++++++++++++++

> +   sudami \[xiao\_rui\_119@163.com\] +

> +   2007.12.02 +

> +   英文翻译&学习笔记 +

> +++++++++++++++++++++++++++++++++++

> IDA 还没有去熟悉,连分析 IDB 都不会.看到那些大牛们逆向的代码,很是羡慕.于是先得打好基础.切入主题---

> NTFSI 这书的相当不错\[可惜只有英文电子版\].偶先看第 6、7、8 章,纪录一些关于 cache Manager 的学习笔记.共勉之

> \------------------------------------------------------------------------------------------------------------------------------------

> `the NT cache Manager` 和 `Virtual Memory Manager`、`I/O Manager` 、`File System Driver`联系密切.

> 它主要是加速数据的读出和写入.将 2 级存储设备上的（硬盘什么的）的部分数据缓存在内存中\[**System Cache**\]----预先将磁盘的部分内容读进内存,避免用户等待; 延迟写入磁盘.

> (1)文件流 \[File Streams\]

> 任何打开的文件都一个相对应的文件对象\[**File Object**\],

> 任何线状的、细长的与文件对象相关的数据流被定义为文件流(理解的可能不准确). 对每个文件流的存储通过内存映射\[**memory mapping**\],文件系统提供**缓存**和**非缓存**的权限.

> (2)虚拟块存储 \[Virtual Block Caching\]

> 说的主要是文件内存映射, MS"操作系统"还有"WINDOWS 核心编程"的书都有详细的介绍.

> 原来的一些操作系统直接是操作物理地址----在进行文件操作前首先会将虚拟地址转换成物理地址\[内存中的实际地址\],如果有多的物理地址,操作才会继续进行; 而 `the NT cache Manager` 会利用内存映射,不会在意实际的物理内存是否够能够拥有,而是

> only use virtual byte offsets in a file to keep track of cached information

> ,一份数据保存在真正的物理内存中,映射多份到进程的地址空间,利于同步操作. 对于某个进程修改的数据,它会单独复制一个页出来,保存修改的数据,到最后才延迟写入磁盘。

> (3)读操作 \[Cached Read Operation\]

> 用户进程发出一个"读"请求被 `kernel` 层的**I/O Manager**知晓---->它产生 IRP 发送至 `File System Driver`， 用户进程的一些数据有 3 种方式被保存：

> 映射到系统的高 2GB 内存空间、运用 `Memory Descriptor List`（`MDL`）、直接传下去,不用拷贝数据

> \---->**File System Driver**接收到 IRP，意识到此 IRP 有文件的打开读取权限.于是检查此文件**"缓存"\[caching\]**是否已经初始化，如果没有,调用**the cache Manager**来初始化，而它会调用**Virtual Memory Manager**来创建文件映射\[**section object**\]来缓存文件---->`File System Driver` 调用 `CcCopyRead()` 函数把 IRP 传递给 the cache Manager,，现在它有责任把磁盘上的数据拷贝到用户的 buffer 中---->the cache Manager 检查包含用户请求的文件数据是否已经映射了，如果没有则创建之,然后仅仅是把映射的文件数据拷贝到用户的 buffer 中---->如果产生了**缺页中断\[page fault\]**，权限移交给 `Virtual Memory Manager`，它产生一个**noncachedpaging IRP**，此 IRP 经过 NT I/O Manager 到达 `File System Driver`，然后它产生一个相应的 IRP 向底层的设备驱动传递，底层的设备从磁盘获得数据后完成请求---->然后 the cache Manager 继续拷贝数据到用户的 buffer. copy 完后把控制权限交给 `File System Driver`---->`File System Driver` 完成此 IRP 请求,返回到 `I/O Manager`----> I/O Manager 完成操作后返回到用户进程.结束

> ![名称: Snap2.gif> 查看次数: 136> 文件大小: 128.5 KB](img/08a0b786a6b773078c501366b4f812e7.jpg)

> (4) 写操作 \[Cached Write Operation\]

> 和上面的差不多,只不过是用户的 buffer 中开始就包含着大量的数据,Cache Manager 从 buffer 中拷贝数据到映射中,产生缺页中断时,虚拟内存管理器不会向别处发 IRP 了,而是释放掉部分的物理内存,用于此.

> ![名称: Snap3.gif> 查看次数: 134> 文件大小: 194.3 KB](img/311ca854320e19dc056b25298b3204e0.jpg)

> (5) 存储管理器的交互 \[Interfaces\] \翻译有误,偶也不知道翻译成什么![ \]

> 系统组建可以调用 Cache Manager 的许多交互接口：

> ┎文件流控制函数 \[file stream manipulation functions\]

> ┎复制接口 \[the copy inerface\]

> ┎MDL 接口 \[the MDL interface\]

> ┎锁定操作 \[the pinning inerface\]

> the copy inerface 是最简单的一个操作,可以在内存和 the cache file stream 之间任意复制数据

> the MDL interface 是一种不透明的方式, 它通过**DMA**得到**the system cache**的直接访问权限

> the pinning inerface 可以完成 2 个任务 ----

> ┎ Map data into the system cache for direct access using a buffer pointer

> ┎ Pin (or lock) the physical pages that back the mapped data

> 如果文件过滤驱动或者系统的其他组件需要频繁性的直接使用物理内存中的数据结构,the pinning inerface 可以派上用场；如果要保证物理内存中的数据常驻,不可移出.the pinning inerface 同样可以运用

> (6) 一些重要的数据结构

> ┎ FsContext

> ┎ SectionObjectPointer

> ┎ PrivateCacheMap

> ① 当缓存需要被初始化时,**FsContext**必须被初始化,它指向一个结构体 **FSRTL\_COMMAN\_FCB\_HEADER **

> ![名称: Snap4.gif> 查看次数: 133> 文件大小: 14.8 KB](img/cf4d7819a741bf0c5312f906360a0e2b.jpg)

> 一个文件流在内存中对应一个**File Control Block(FCB), **参数**Flags**用来指定用 ERESOURCE 中的哪个；参数**Flags2**用来特指 ----> 一般情况下, FSD 可能指定对缓存文件流不用延迟写入,但如果 `Flags2` 被设置为`FSRTL\_FLAG2\_DO\_MODIFIED\_WRITE,`则缓存管理器会忽略 FSD 的指令,而运用延迟写入；2 个 ERESOURCE 通过`ExInitializeResourceLite()`来初始化；

> 剩下的 3 个关于 Size 的很简单：**AllocationSize**是在硬盘上为文件流分配的存储空间，紧接着，缓存管理器需要时刻标记这个参数的变化；**FileSize**是面向用户的,包含了文件流数据的大小.任何越界都会返回`STATUS\_END\_OF\_FILE`；**ValidDataLength**则是有效的文件长度,任何读操作如果超过了这个范围,都会返回 0.这有助于防止对磁盘的不必要的 I/O 操作,也是为了数据安全; \[NTFS、HPFS 都支持这个操作\]

> \==>>the NT Cache Manager 通过`SetFilelnformation IRP`来通知`ValidDataLength`的改变

> \==>>举个例子来说明`ValidDataLength`的必要性(理解的可能有偏差)

> \[ 当用户进程要扩展一个文件长度,这个请求被送到 Cache Manager,它首先会检查是否有多个页可用,接着产生缺页中断,page fault 最终会被 the file system driver 处理,此时它如果保留有 `ValidDataLength` 指定的可用长度,便意识到不需要进行读草组，返回 0 来完成缺页中断即可\]

> PS: (A.)改变`AllocationSize`或者`FileSize`必须和其他读/写请求同步,而且缓存管理器必须立即响应任何的改变,要不然就会出现以下现象----用户扩展了文件的长度,但 the cache Manager 没有标明新的文件大小,于是用户的读请求的数据地址在旧的数据结束地址以后时,就会被返回`STATUS\_END\_OF\_FILE`错误；

> (B.)如果是`Paging I/O read/write`请求,**FileSize**的改变不需要同步处理

> ② `SectionObjectPointer` 指向`SECTION\_OBJECT\_POINTERS`这个结构体\[需要分配到非分页的 kernel memory 中\],它保存了与文件流相关的文件映射和缓存相关信息. 这让偶联想起来 MJ0011 的 360 粉碎机.呵呵,众多猥亵的手法中的一种便是设置`ImageSectionObject`、`DataSectionObject `为空,发 IRP 删除之.具体可参考

> [`hi.baidu.com/sudami/blog/item/7760f61990997e4643a9adf3.html`](http://hi.baidu.com/sudami/blog/item/7760f61990997e4643a9adf3.html)

> 或者

> [`bbs.pediy.com/showthread.php`](http://bbs.pediy.com/showthread.php?t=45233)

> 引用:typedef struct \_SECTION\_OBJECT\_POINTERS {
> PVOID `DataSectionObject;`
> PVOID `SharedCacheMap;`
> PVOID `ImageSectionObject;`
> } `SECTION\_OBJECT\_POINTERS;`

> \[only one shared cache map structure is allo-cated by the Cache Manager when caching is first initiated for a file stream viasome file object. This shared cache map is used by all open instances for the file
> stream. \]

> ③ `PrivateCacheMap` 值得注意：

> 文件系统驱动检查 `PrivateCacheMap` 是否为空而且\[唯一的方法\],来判断**缓存是否**通过给定的文件流对应的文件对象而**被初始化**(相当饶口,没解释好-.=)

> "缓存是否被初始化"这个信息不能被保存到任何其他的地方,因为缓存管理器保留有通过暴力方式关闭缓存,这些缓存经过一些或者全部的与文件流想关联的文件对象. \[理解的不透彻,嘿嘿,大家可以看看原文\]

> (7) 数据控制块 \[Buffer Control Blocks\]

> 要用锁定操作 \[the pinning inerface\]，就必须用到**Buffer Control Blocks(BCB)**,它分为公共 BCB、私有 BCBpublic BCB 运用在"数据的锁定"和紧接着的"解锁"上

> (8) `Virtual Address Control Block` (`VACB`)

> 所有指向同一个文件流的`VACB`组成一个数组. 而`SharedCacheMap`保存有指向这个数组首地址的指针.每一个 `VACB` 保存有一块数据的虚拟地址和在文件流中的偏移值. the NT Cache Manager 通过这个可以决定一个已经映射的 view 是否包含用户请求的数据,没有则映射之；`VACB` 结构被分配在固定的全局内存池中，所以当一个 view 需要被创建时,缓存管理器可能找不到空闲的 `VACBs` 来分配新的文件流.所以 Cache Manager 需要 unmap 以前的一个 mapped view,从数组中摘除一个 `VACB`,再分配 `VACB` 到新的文件流上去.

> ![名称: Snap5.gif> 查看次数: 134> 文件大小: 125.4 KB](img/78c500fc402bc8426e8ed60dd9b22a84.jpg)
> ![名称: Snap6.gif> 查看次数: 132> 文件大小: 135.0 KB](img/cac4c091ee1d1b288c02ee4e56405751.jpg)

> 看完这个图,偶明晰了不少,特别是关于 `FCB` 中的 `PrivateCacheMap` 和 `SECTION\_OBJECT\_POINTERS` 中的`SharedCacheMap`

> (9) 资源的获取 \[Resource Acquisition\]

> 有 2 个 `ERESOURCE`： `MainResource` 和 `PagingloResource`

> 待续。。。。

> \-------------------------------------------------------------------------------------------------------

> [`hi.baidu.com/sudami/blog/item/1e1227d0550d428ca0ec9c2c.html`](http://hi.baidu.com/sudami/blog/item/1e1227d0550d428ca0ec9c2c.html)

4dd466d0e1e0cc123afbe504da2462ca
