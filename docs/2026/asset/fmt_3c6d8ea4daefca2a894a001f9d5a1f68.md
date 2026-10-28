
`Sync cache support`：该属性表明设备是否支持`SCSI SYNC CACHE`命令，或者其它总线上的与该命令功能相同的命令。

`Battery backup`：该属性值表明设备是否有备用电池以保护电源错误发生期间的数据的完整性。

# 存储类驱动介绍

无论mass storage device挂在什么类型的总线上，storage class driver都会利用已经建立好的SCSI class/port接口来控制它们。当然，系统会为挂载了mass storage device的总线提供storage port driver（目前有：SCSI、IDE、USB和IEEE 1394）。Mass storage device所挂载的总线到底是什么类型，对storage class driver来将是透明的。

所有storage class driver都需要处理来自APP或上层驱动的I/O请求，然后穿过下层过滤驱动，最后到达storage port driver。Storage class driver的上层驱动主要是创建包含`CDBs`的`SRBs`，然后作为`IRP`的一个参数，并将该`IRP`发送给storage class driver。

Storage class driver在`SRB`中不提供寻址信息，而port driver（或一个still-lower driver）负责处理所有的寻址请求。Storage port driver将`SRBs`翻译成下面的HBA所要求的格式的命令并发送到设备。这里下面的HBA可能是SCSI HBA、1394 HBA、IDE Controller或其他类似的硬件。在WDK中，“HBA”代表下层类似的适配器或控制器。

Storage class driver必须实现`DispatchSystemControl`例程，以处理system-control `IRPs`。

## (一般功能) Storage Class Driver's General Functionality

相对于storage port driver来讲，storage class driver是一个高层驱动，其实现了具体设备类型的功能。一般情况下，所有的storage class driver都需完成一下功能：

- 声称PnP管理器传递到其`AddDevice`例程中的所有`PDO`（由PnP管理器创建）所代表的设备。

- 为每一个`PDO`创建一个`FDO`，并挂到其设备堆栈上。

- 如果该storage class driver控制的是一个可分区的设备，则其还需为每个分区创建一个`PDO`，并相应枚举请求。

- 解析系统I/O请求（`IRPs`）。

- 将`IRPs`映射为SCSI class/port接口请求（带有SCSI `CDBs`的`SRBs`）。

- 为请求建立超时值（不同的设备类型，超时值可能不同）。

- 将数据传输大小限制到适合下层HBA的值。（将一个`IRP`分成多个`IRP`）

- 处理storage port driver没有处理的错误，比如check-condition状态或者总线重置。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`SCSIPhoto1.png`](javascript:void(0);) （75.45kb，22次下载）

- [`SCSIPhoto2.png`](javascript:void(0);) （138.21kb，15次下载）

- [`SCSIPhoto3.png`](javascript:void(0);) （86.94kb，12次下载）

- [`SCSIPhoto4.png`](javascript:void(0);) （128.14kb，8次下载）

- [`SCSIPhoto5.png`](javascript:void(0);) （142.89kb，10次下载）
