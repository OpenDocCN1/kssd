
存储端口驱动为所有Windows存储类驱动（如磁盘、磁带、DVD驱动）定义统一接口，提供硬件抽象，隔离Storage Driver与HBA（主机总线适配器）的硬件特性差异，同步管理同一HBA上所有设备的访问。

## 协议转换

对SCSI设备：将`SRB`（SCSI请求块）直接传递至`Storport`微端口驱动（硬件专属）

对传统IDE/ATAPI/IEEE 1394总线：将`SRB`转换为适配器所需格式（如重组`CDB`/协议包）。

### 挂钩的设置

这是Windows存储设备栈的底部，也是我们拦截的地方

![](https://bbs.kanxue.com/upload/attach/202507/982720_K666X4ZV4DU5ZMF.webp)

我们想在Storage Port Driver上挂钩，设置Filter Driver。

（这里画的有点问题，Capture Driver应该在Storage Driver和Storage Port Driver之间，Capture还打错了=￣ω￣=）

看一下windbg的输出：

![](https://bbs.kanxue.com/upload/attach/202507/982720_BDYUMUQT65PWFY4.webp)

`storahci.sys`便是Storage Port Driver，也是我们挂钩的位置。

那么如何挂钩呢

答案在`DriverObject->MajorFunction`中

![](https://bbs.kanxue.com/upload/attach/202507/982720_HFDUS5TA6DQFTVB.webp)

`IRP_MJ_INTERNAL_DEVICE_CONTROL`

Storage Driver和Storage Port Driver之间通过`DriverDispatchXxx`联系，所以只要修改`MajorFunction`的地址我们就可以截获`SRB`请求。

经过调试得知，二者通过`IRP_MJ_INTERNAL_DEVICE_CONTROL`联系，所以只需修改`MajorFunction`的`IRP_MJ_INTERNAL_DEVICE_CONTROL`即可。

## WinDbg调试

首先打个断点看一下调用上下文，这是Call到`MajorFunction`的`IRP_MJ_INTERNAL_DEVICE_CONTROL`的上下文：

![](https://bbs.kanxue.com/upload/attach/202507/982720_VJNJGNW62ADG6EQ.webp)

![](https://bbs.kanxue.com/upload/attach/202507/982720_RZAVUTU4AV8YEEP.webp)

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `NTSTATUS __fastcall DispatchXxx(`  `PDEVICE_OBJECT DeviceObject,`  `PIRP           Irp`  `)`  `{`  `RCX:DeviceObject`  `RDX:Irp`  `}` |

根据`fastcall`调用协议，`rdx`是指向`IRP`的指针，所以我们解析`IRP`即可。

根据Windows文档，`SRB`块的指针位于`CurrentStackLoaction`中，而`CurrentStackLoaction`在`DeviceObject`中，具体见下（也可看后面的代码）：

|  |  |
| --- | --- |
