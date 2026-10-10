

# 发发俺的第一题解决方案

> 原文：[`www.pediy.com/kssd/pediy10/78990.html`](https://www.pediy.com/kssd/pediy10/78990.html)

mbrprot..sys 驱动建立了一个假 atapi 的 DriverObject 和一个假 DeviceObject，并以此对传过来的 IRP 进行处理，由于不得破坏 mbrpto.sys 自有流程（包括不得改写 mbrprot.sys 的代码，不得 HOOK mbrprot.sys 可能调用的系统函数等）；俺想出两种解决办法。  
1、由于已说明系统有且仅有一个 IDE 硬盘，必须是 ATA 接口，因此可以得到\\\\Device\\\\Ide\\\\IdePort0 Device 对象后直接向 atapi 驱动发 IRP。  
  
2、直接 IO 硬盘之。  
  
3.利用 SCSI\_PASS\_THROUGH\_DIRECT 写进去。  
  
4.直接对 DR0 发送 IRP，由于对 DR0 发送 IRP 位于的磁盘保护驱动的上端，因此会被磁盘保护驱动检查，但由于磁盘保护驱动检查的是 srb->QueueSortKey，所以可以通过修改 srb->QueueSortKey 来绕过磁盘保护驱动，这与直接向 atapi 驱动发送 IRP 的方法不同，不是得炒冷饭。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 第一题.rar  
 |