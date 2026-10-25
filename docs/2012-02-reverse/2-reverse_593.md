
# Rootkit Unhooker 驱动逆向分析

> 原文：[`www.pediy.com/kssd/pediy10/66839.html`](https://www.pediy.com/kssd/pediy10/66839.html)

这个驱动本来准备端午节搞的,但后来端午节去了趟北京玩了几天,也就搁置了...最近连续花了 4 个晚上,大致把它搞的差不多了,有些收获...初学 window 驱动,水平很菜,有些东西我也未必清明,加上 3 环的代码还没有吃透,期间也没有用调试器跟踪,全是 IDA(F5)静态分析的,谬误在所难免,高手请飘过,希望对菜鸟学习有点帮助.

## 1. SSDT 检测

这个功能有三个 `IoControlCode` 与之对应,他们分别是 `22000fh`(拷贝 `KeServiceDescriptorTable`);`220007h`(拷贝 `KeServiceDescriptorTable->ServiceTable`);`22000bh`(恢复一个 `ssdt hook`),这个功能目前还不具备检测和恢复 `inline hook` 的功能

## 2. Shadow SSDT 检测

这个功能实现和 `SSDT` 差不多,都是从 `KeSystemServiceDescriptorTableShadow` 入手,不过 3 环需要为每个平台配置一张函数名表(`ssdt` 可以不配置函数名表,可以从 `ntdll.dll` 中收集),详细文章可以参见 `zhuwg` 兄弟写的《《shadow ssdt 学习笔记》》一文

## 3. 进程

### 3.1 进程检测

`IoControlCode` 是 `22001Bh`,在检测之前会设置检测方式,然后根据检测方式进行检测.这个功能主要由 `irp_mj_device_control_dispatch_function` 函数的 `is_22001Bh_list_process` 分支完成,这里集成了 4 种检测方法,大致如下:

#### 3.1.1 基于 PspCidTalbe 的检测

这里 `rku` 对句柄表做了解析,对每个 `Object` 判定是不是 `EPROCESS`,是则插入到输出列表(`icesword` 也有这中检测,不过在 1.22 中没有解析句柄表,而是用 `ExEnumHandleTable` 搞定这些) 详细代码见 `list_process_in_2k_by_handle_table`/`list_process_in_not_2k_by_handle_table` 函数

#### 3.2.1 基于 KiWaitListHead/KiWaitOutListHead 的检测

这个检测方法很简单,关键是 `KiWaitListHead`/`KiWaitOutListHead` 两个链表的获取, 详细代码见 `list_process_by_list_head` 函数

#### 3.3.1 如果是 xp 系统则还会采用\*淫\*\*荡\*的内存暴力搜索进程进行检测

这段代码看 `debugman` 的 `yykingking` 发过, 详细代码在 `list_process_by_search_memory_in_xp` 函数

#### 3.4.1 rku 还会 inline hook SwapContext 函数

在这个函数里会记录当前的活动进程情况,经过上面三种方法的洗礼后,对在 `SwapContext` 的记录里的,但不在输出列表里的继续加入,当然是有判定条件的(条件就是 `EPROCESS` 不能在 `ntoskrnl` 模块内,即不为 `idle` 进程). `BTW`:这里有个逻辑我觉得有问题:

```
.text:000134A7           loc_134A7:                        ; CODE XREF: irp_mj_device_control_dispatch_function+233j

.text:000134A7 3B 05 34 4E 01 00             cmp     eax, g_nCurrentProcessCount

.text:000134AD 7D 5A                          jge     short loc_13509
```

这个检测方法的代码就在 `loc_134A7` 开始位置

上面获取的只是 `EPROCESS`,要获取进程路径信息,还需要 `IoControlCode(220047h)` 的代码, 详细功能代码在 `get_image_path_by_eprocess` 函数里,在这个函数里,我们可以看到 `rku` 先走 `peb` 获取进程全路径,然后对非 `2k` 系统再走一次我们更为熟悉的 `SectionObject` 获取路径

### 3.2 杀进程

`rku` 提供了两种杀进程的方法

#### 3.2.1 IoControlCode(22001Fh) ZwOpenProcess/ZwTerminateProcess/ZwClose 杀进程

这种杀法比较的弱

#### 3.2.2 ioControlCode(22004fh) 内存清 0 杀进程

具体代码是 `kill_process_by_fill_zero` 函数,这个代码看 `debugman` 的 `yykingking` 也发过

### 3.3 Dump All Process Memory

`IoControlCode` 是 `22008bh`, 没什么可说的, 详细代码在 `dump_all_process_memory` 函数

### 3.4 Dump Seleted

这个就是拷贝主程序的内存镜象, `IoControlCode` 是 `220083h`, 先获取文件大小(对 `xp` 及其以上系统,通过 `EPROCESS->SectionObject->segmentObject->SizeOfSegment` 获取主程序内存镜象大小,对 `2k` 系统,由于手头缺乏资料,那段代码什么意思不知道,详细代码在 `get_main_file_size_by_eprocess` 函数), 然后输出缓冲区足够大就进入 `dump_selected` 函数拷贝内存

## 4. 内核模块和 Stealth Code 检测

`IoControlCode` 是 `0x220023h`,在检测之前也会设置检测方式,然后根据检测方式进行检测.这个功能主要由 `list_kernel_module` 函数完成,这个函数里面集成了很多检测方法,很多方法都比较\*淫\*\*荡\*,大致的检测方法有:

### 4.1 用 IoDriverObjectType 指向的 OBJECT_TYPE.TypeList 链表检测

遍历这个链表中的每一项,并会取每个 `DriverObject` 上的 `DeviceObject`,并向上扫描这个设备所在的设备栈(注意这里没有向下扫描),尝试发现其它的 `DriverObject`  详细代码见 `list_module_by_IoDriverObjectType` 函数

### 4.2 用 IoDeviceObjectType 指向的 OBJECT_TYPE.TypeList 链表检测

遍历这个链表中的每一个设备,并也会向上扫描这个设备所在的设备栈(注意这里没有向下扫描),尝试发现其它的 `DriverObject` 详细代码见 `list_module_by_IoDeviceObjectType`

### 4.3 用目录对象树(ZwOpenDirectoryObject("\\\\"))检测

遍历这棵树,对叶子节点(具体对象)看其 `OBJECT_HEADER.Type` 字段是否指向 `IoDriverObjectType`/`IoDeviceObjectType`,如果是的话,则会采用 4.1/4.2 相似的措施;对目录对象子树则递归处理之(内核堆栈空间太小,递归搞始终给人一种不安全感,看 `SnipeSword` 里面处理注册表部分也大量递归,比较汗)  详细代码见 `list_module_by_OpenDirectoryObject` 函数

### 4.4 用 DriverSection(PsLoadedModuleList 链表)检测

这个方法大家都比较熟悉, 详细代码见 `list_module_by_DriverSection` 函数

经过上面 4 个方法,`DriverObject` 就获取完毕了,他们对应的都是有详细信息的内核模块(这项检测初步给人的感觉是比 `icesword` 可靠些,`icesword` 则是先尝试恢复 `NtQuerySystemInformation`,设置 `KernelMode`,然后调用 `NtQuerySystemInformation` 获取内核模块信息),下面的检测主要是对一些零星的能执行代码的内存块进行检测(`Stealth Code`)

### 4.5 用上面几种方法已经找到了一些 DriverObject,这里就对这些 DriverObject 下手

检测这些 `DriverObject` 的 `DriverStartIo`/`MajorFunction` 是否被 `hook`,如果被 `hook` 且新地址不在我们已经找到的模块里,则说明是那种分内存,写代码似的 `hook`,记录这块内存信息 详细代码见 `list_module_by_DriverStartIo_and_MajorFunction` 函数

### 4.6 暴力搜索内存

找到 `PE` 头部,并判定 `OptionalHeader.Subsystem` 是否是 `Native` 详细代码见 `list_module_by_scan_memory` 函数

### 4.7 利用句柄表检测

这个的具体原理我未能理解, 详细代码在 `list_module_use_suspicious_thread_by_handle_table_in_not_2k`/`list_module_use_suspicious_thread_by_handle_table_in_2k` 这两个函数,感觉跟内核模块检测无关......知道的请说一声

### 4.8 rku 会 inline hook 三个调用频度很高的函数(ExAllocatePool/ExAllocatePoolWithTag/KeDelayExecutionThread),在这三个函数里会记录它们的返回地址,并利用这些地址来检测一些模块,如果返回地址不在某具体模块范界则会记录其内存信息

详细代码见 `list_module_by_return_address` 函数

### 4.9 检测 KiSystemService/KiFastCallEntry 是否在某具体模块范界

如果不在也记录其内存信息

到这里检测就完成了,上面的检测方法太多,有些检测获取的信息不足,`rku` 会进行一些额外的修正处理,这些代码都在 `list_kernel_module` 的结尾部分,详细请参见源码,我都做了一些注释

## 5. File 检测

由于目前一直在用 `IDA` 反,并没有使用调试器,主程序目前还只反了一点,这个部分具体做什么的,我不太清楚,不过在驱动里看到一个 `IoControlCode(2200A7)` 是在做扇区级别的操作......

## 6. Code Hooks 检测

在驱动里并没有看到分析 `code hooks` 的代码, 只看到一些拷贝内存到 3 环的功能函数,把一些系统关键手工 `load` 展开后,进行内存对照比较就可以搞定这个了,当然这个里面还具备 `KiSystemService`/`KiFastCallEntry hook` 的检测和恢复代码,详细可以看 `IDB` 文件

## 7. rku 的进程保护

### 7.1 ssdt 上的保护

在 `ssdt` 上对三个函数(`NtOpenProcess`, `NtOpenThread`, `NtDuplicateObject`)进行了 `inline hook`

`NtOpenProcess` 防止打开 `rku` 进程

`NtOpenThread` 防止打开 `rku` 的线程

`NtDuplicateObject` 防止复制 `rku` 进程的句柄

### 7.2 shadow ssdt 上的保护

在 `shadow` 上对四个函数(`NtUserQueryWindow`, `NtUserFindWindowEx`, `NtUserWindowFromPoint`, `NtUserNotifyIMEStatus`)进行了 `inline hook`,防止其它程序通过 `FindWindow`, `WindowFromPoint` 等函数来枚举 `rku` 的窗口,你可以看到 `spy++` 对 `rku` 已经失效

## 8. 逆向这个的过程中,参考了网上很多大牛的文章

`IceSword`&`Rootkit Unhooker` 驱动简析

`zhuwg`  《《shadow ssdt 学习笔记》》

`prince` 翻译的《《侦测隐藏进程的强文》》

`yykingking` 发在 `debugman` 上的两个代码片段

还有些文章,无法一一列举,再次一并表示感谢

大致就是这样了.

`linxer` 2008-06-19 于哈尔滨

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `idb.rar`
|
