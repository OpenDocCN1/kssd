
# 想干什么干什么。分析 Atapi.sys 主干

> 原文：[`www.pediy.com/kssd/pediy11/112544.html`](https://www.pediy.com/kssd/pediy11/112544.html)

近来一直在做磁盘驱动的分析，于是对整个拓补图上所有的函数都逆了一遍。（估计有很多前辈逆过，但没人想发这个东西出来）

我这里讲的是原理，可能很多人会觉得没有用，但是其实这个是最有用的，懂了这个磁盘驱动的十三个函数，什么底层的硬件写入都可以写出来，什么高级的 HOOK 也都没有问题。

下面发些比较粗糙的分析总结，具体的分析就全记自己笔记本上了，毕竟是心血的结晶，不想全发出来，留点自己备用。

下面是磁盘驱动的拓补图：

```
00 f8ac63d8 f9802193 atapi!IdeReadWrite
01 f8ac6464 f9802bbc atapi!IdeSendCommand+0x3ab
02 f8ac64b0 f98050ad atapi!AtapiStartIo+0x23e
03 f8ac64dc 805414f7 atapi!IdeStartIoSynchronized+0x16f
04 f8ac650c f9805fe1 nt!KeSynchronizeExecution+0x17
05 f8ac6524 f9c1cd28 atapi!IdePortAllocateAccessToken+0x1b
06 f8ac6534 806d3998 PCIIDEX!BmReceiveScatterGatherList+0x24
07 f8ac6560 806d3b7c hal!HalBuildScatterGatherList+0x19e
08 f8ac6590 f9c1cddb hal!HalGetScatterGatherList+0x26
09 f8ac65c4 f98068b1 PCIIDEX!BmSetup+0x5f
0a f8ac65fc 804efe19 atapi!IdePortStartIo+0xeb
0b f8ac661c f9805c9a nt!IoStartPacket+0x7d
0c f8ac6648 804eedf9 atapi!IdePortDispatch+0x4e6
0d f8ac6658 f99dd061 nt!IopfCallDriver+0x31
0e f8ac666c f99dcd58 CLASSPNP!SubmitTransferPacket+0x82
0f f8ac669c f99dce49 CLASSPNP!ServiceTransferRequest+0xe4
10 f8ac66c0 804eedf9 CLASSPNP!ClassReadWrite+0xff
11 f8ac66d0 f9c2436c nt!IopfCallDriver+0x31
12 f8ac66e0 804eedf9 PartMgr!PmReadWrite+0x93
13 f8ac66f0 805cdc06 nt!IopfCallDriver+0x31
```

XP SP3 版可能会多一个 `CallIdeStartIoSynchronized`，夹在 `KeSynchronizeExecution` 和 `IdePortAllocateAccessToken` 之间

这是 `Atapi.sys` 的拓补图，从 `IdePortDispatch` 开始，一直到 `IdeReadWrite`，`IdeReadWrite` 以下就是 HAL 端口输入了。

`SubmitTransferPacket`，`ServiceTransferRequest`，`ClassReadWrite` 其实也可以看看，`WinDDK` 里提供了源码，对于理解还是有好处的。

这里讲讲 `Atapi` 各个主干函数的主要功能：

`IdePortDispatch` 人如其名：就是用来根据 `SRB` 功能号分发 `SRB`。

`IdePortStartIo`: 在 `IoStartPacket` 之后确保了每次只处理一个 `SRB`，里面对 `SRB` 的处理分为两个部分：其一是 DMA 传输模式，其二是原始的中断传输模式。DMA 传输模式下，他直接调用 `BmSetup`，以 `MdlAddress` 为参数。原始的传输模式下，他调用 `MmMapLockedPagesSpecifyCache` 先把 `DataBuffer` 之中的内容映射好(因为 `IoBuildMdl` 是不对 `DataBuffer` 映射到系统空间的，Windows 的设计方案是什么时候该用什么时候映射)，`DataBuffer` 有可能是用户空间的地址，而 `StartIo` 例程运行在 `DISPATCH_LEVEL`，就有可能会牛头不对马嘴，所以他这里其实不是根据 `DataBuffer` 建立映射，而是根据 `MDL` 结构后面的 `PFN`（物理页面号）数组进行映射。映射好了，会把得到的 `MappedSystemVa` 存入 `Srb` 的 `DataBuffer`，以供日后调用，最后调 `IdePortAllocateAccessToken`，直接略过中间的四个跟 DMA 有关的函数

`BmSetup`: 在 `[[IdePort1(我这里 IDE 设备的下层是 IdePort1,光驱的下层是 IdePort0)的设备扩展(以下都简称设备扩展)+AC]+8F8h]` (跟 DMA 有关的一个东西，有些 IO 内存也在里面)里存入一个数据结构：

```
+A0h:MdlAddress
+A4h:读或写标志号
+A8h:函数地址
+ACh:设备对象(IdePort1)
+90h:DataBuffer
+98h:DataTransferLength
```

然后调用 `HalGetScatterGatherList`

`HalGetScatterGatherList` 没什么好说的，就是调 `HalBuildScatterGatherList`。

`HalBuildScatterGatherList` 里面就比较重要了，

1. 先调 `HalCalculateScatterGatherListSize` 计算 `MDL` 结构紧跟着的 `PFN` 数量，返回 `PFN` 数量，和需分配的结构大小。然后对全局变量 `HalpOutStandingScatterGatherCount` (指明当前 SG 请求块数量)递增上 1，这个全局变量会在 DMA 中断服务完成之后，如果 SG 请求被完成，执行 DPC 函数时会被递减。

2. 对 `MDL` 结构进行解析，将解析后的数据存入新分配的结构中，结构解析如下：

```
+0h：离散的物理页面数量
+4:当 HalBuildScatterGatherList 的倒数第二个参数不为 0 时，为 1，为 0 时，为 0.具体意义不明
+8h 以后就是一个 16 字节大小的数组：
+0h：物理页面地址（加上页面内偏移后的）
+4h:页面大小
+8h:此页面有效的数据部分大小
```

3. `HalBuildScatterGatherList` 会对连续的物理块进行整合，整合到一个结构里

4. 然后调 `BmReceiveScatterGatherList`

`BmReceiveScatterGatherList` 里面调了三个函数：`BmRebuildScatterGatherList`，`BmPrepareController`，`IdePortAllocateAccessToken`。`BmRebuildScatterGatherList` 根据 `HalBuildScatterGatherList` 分配的结构，进行 DMA 传输时的内存基址设置。

具体的设置如下，对 `[[设备扩展+AC]+84h]` 表示的 IO 内存存入:

```
+0h:第一个物理页面地址
+4h:存入有效长度(USHORT)
+7h:结束标志位
```

对于有效长度大于 `10000` 的项次，会进行拆分处理，拆成两个结构

`BmPrepareController` 进行 DMA 控制器的初始化。

这中间四个函数是 DMA 相关

`IdePortAllocateAccessToken` 以下就是磁盘部分了。

`IdeStartIoSynchronized` 的负责范围是：

1. 对超时时间设置。
2. 对 `C7`，`C8` `SRB` 功能号调用重设总线函数。
3. 对 `ABORT_COMMAND` 获取当前 `SRB`
4. 调用 `AtapiStartIo`
5. 对所发现的错误情况用 `IdePortNotification` 完成掉，并通知下一 `SRB` 就绪，最后插入 `DPC` (没调过，可一猜就知道是调控 `SRB` 处理用的)

`AtapiStartIo` 根据 `SRB` 功能号进行选择处理，处理的大类有 7 个，`SRB_FUNCTION_IO_CONTROL` 里又有 4 个小分类，说起来比较多，就不说了。哈哈。总结几点跟这文章有关的：

1. 会判断当前 `CurrentSrb` 是否存在（`CurrentSrb` 标示着正在处理的 `SRB`，如果 `SRB` 请求完成，当中断返回时会清空，具体函数是 `AtapiInterrupt`）
2. 只有 `IRP_FUNCTION_EXECUTE_SCSI` 会调用 `IdeSendCommand`（这点对后面比较重要）

`IdeSendCommand`：根据 `Cdb` 里的 `OpeationCode` 进行分发，也就跟 `AtapiStartIo` 似的。只不过 `IdeSendCommand` 分发的是更细的功能号 `Cdb[0]`，而 `AtapiStartIo` 分发的是 `SrbFunction`

（对有些功能号直接在设备扩展里复制过来后返回，有些功能号就一定需要 DMA 了，所以得启动 DMA 传输）对 `SCSIOP_READ` 或者 `SCSIOP_WRITE` 会调用 `IdeReadWrite`

`IdeReadWrite`: 负责对磁盘设备进行启动。因为上面原始方式也好,DMA 方式也好，都调用了 `IdeReadWrite`，所以很自然 `IdeReadWrite` 里就分成了两种模式。原始模式下和 DMA 模式是用 `pSrb->SrbExtension` 作标志区分的。

1. 先判断设备是否忙

2. 先写扇区数量，再写磁盘扇区 `SectorBegin`，对其最高字节处理后，分成四个字节，依次写入 `[[设备扩展+ACh]+14h]`，`[[设备扩展+ACh]+18h]`，`[[设备扩展+ACh]+1Ch]`，`[[设备扩展+ACh]+20h]` （DMA 和无 DMA 处理情况略有不同）

3. 再判断是读是写，读则向 `[[设备扩展+ACh]+24h]` 端口号写入 `C8H`，写则输入 `CAH`，有 DMA 能力时，调用 `BmArm` 启动 DMA 传输后返回。

4. 无 DMA 能力时，直接用 `WRITE_PORT_BUFFER_USHORT` 写入 `DataBuffer` (其实是 `MappedSystemBufffer`，前面已经被覆盖掉了)

在 `IoStartPacket` 之后保证了 `SRB` 不重入，后面的函数都不会出现并发的情况。(所以 `BmRebuildScatterGatherList` 没有进行当前是否有繁忙的判断，因为只有在 `IdeReadWrite` 之后才有可能对另一个 `SRB` 中的物理地址进行设置)

在 `AtapiStartIo` 之中对 `CurrentSrb` 的判断保证了设备不同时执行两份任务。

那如果说要破还原，怎么办。直接模仿 `BmRebuildScatterGatherList`，和 `IdeReadWrite` 操作，就可以实现最底层的端口写入。或者你想不麻烦点，那就 `HOOK` `IdePortStartIo`，先提参数 `IRP`，看看是不是 `IRP_MJ_SCSI`，进而判断是不是写入.是写入则对里面的参数 `IRP` 参数中 `SRB` 进行替换，替换一下 `MDL` 地址，替换一下 `DataBuffer`，再替换一下 `TransferLength`，还有 `Srb` 里的 `cdb` 数组，那就想写哪里写哪里，

当然也可以自己 `Call` `IdePortStartIo`。(不过这样比较危险，可能会造成多线程问题,因为 `StartIo` 例程应该要保证只有一个 `SRB` 能进入，而直接 `Call` 的话，就造成了重入)。

其实不单单是 `IdePortStartIo`，整个拓补图所有的函数你都可以 `HOOK` 掉，替换参数，所以说拓补图上，凡目力所及，尽可 `HOOK`。

然后说说怎么防止写入，还原保护的问题。你可以直接这么干，`HOOK` 掉 `IdeReadWrite`，对于要处理的写入请求，进行检查，检查看看他有没有你想要的标识码（标识码可以在上层的还原驱动设置，至于设置在哪里呢，你可以试试看 `CDB[7]` `CDB[8]` `CDB[9`，`CDB[7]` 据说是扇区数量，可我在逆向中没有发现用到这个的地方，`IdeReadWrite` 是直接根据 `DataTransferLength` 来写入扇区数量的），当然这种方法解决不了破还原时 `HOOK` 只修改 `DataBuffer`，`MDL`，`TransferLength` 和 `CDB` 数组的问题。

那还有种方法，那就是对 `HOOK` 的 `IdeReadWrite` 这样处理，上层的还原驱动和 `IdeReadWrite` 进行通信，每次一有磁盘写入，就去查 `DataBuffer`，`MDL` 结构这些关键的地方是否被修改。

好了，就这么多。我还是大一学生，应该会有很多错误的地方。

有错误的地方，非常诚恳地请求指正。
