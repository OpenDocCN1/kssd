
# 简单说点对文件系统、分层驱动、文件读写的理解

> 原文：[`www.pediy.com/kssd/pediy10/85367.html`](https://www.pediy.com/kssd/pediy10/85367.html)

write by [`hi.baidu.com/weolar/blog/item...2dbb4bd4f.html`](http://hi.baidu.com/weolar/blog/item/34a8ff19be316572dbb4bd4f.html)

今天看到 sudami 同学学习起这些东西，好久没搞了，很生疏，所以重新学习了一下，有点小小心得（仅仅是心得，有不少错误，希望大家斧正![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") ）：

在我的理解中，设备对象（`drevobj`）相当于驱动对象（`drvobj`）创建的子对象，用来形成设备链，从而接受、处理数据的。设备对象挂到设备链中，接受到了被设备管理器派遣的 `IRP` 时（记得某本书上好像说，没有真正所谓的设备管理器，只是一组派遣例程，如 `IopfCallDriver` 就是将 `IRP` 派遣的。通过 hook 这个函数能得到很多我们想要的东西），这时设备对象的母对象--驱动对象组建的 `IRP` 派遣函数，（即 `DriverObject->MajorFunction[IRP_MJ_CREATE]         = DispatchCreate;`形式的函数）将会接受 `IRP` 并处理。而设备堆栈则是每次 `IRP` 下发时辅助 `IRP` 找到相应派遣函数并存储一些可重用参数的地方。

废话说了这么多，那么我们平时访问、删除文件是怎么回事呢？比如我们删除一个文件，那么系统的流程就是：

（应用层）`DeleteFileA` \-->

`DeleteFileW` \-->

`ntdll.ZwSetInformationFil` \->

（驱动层）`NtSetInformationFile` \->

`SrSetInformationFile` \->

`NtfsNtSetInformationFile` \->

`NtfsCommonSetInformationFile` \->

`NtfsSetDispositionInfo` \->

`MmFlushImageSection` \->

`MiCleanSection`

到了驱动层，windows 就是通过 `IRP` 来传递了。上面的流程中，`nt!NtSetInformationFile->SrSetInformationFile->NtfsNtSetInformationFile->NtfsCommonSetInformationFile` 几个此时虽然也有 `IRP` 的传送，但都还是直接调用，不是 `IopfCallDriver` 的形式。而 `IRP` 起到传递参数的作用。它体内保持着将要下发的设备对象。现在来看接下来某个 `IRP`（并非删除文件）在设备链中的传递：

`nt!NtFlushBuffersFile` \->

`nt!IopSynchronousServiceTail` \->

`sr!SrPassThrough` \->

`Ntfs!NtfsFsdFlushBuffers` \->

`Ntfs!LfsFlushToLsn` \->

`Ntfs!LfsFlushToLsnPriv` \->

`Ntfs!LfsFlushLbcb` \->

`Ntfs!LfsFlushLfcb` \->

`sr!SrWrite` \->

`Ntfs!NtfsCommonWrite` \->

`Ntfs!NtfsNonCachedIo` \->

`Ntfs!NtfsSingleAsync` \->

`VolSnap!VolSnapWrite` \->

`ftdisk!FtDiskReadWrite` \->

`CLASSPNP!ClassReadWrite` \->

`CLASSPNP!ServiceTransferRequest` \->

`CLASSPNP!SubmitTransferPacket` \->

`atapi!IdePortDispatch`(`\\Driver\\atapi` `\\` `IdeDeviceP0T0L0-3`)->

`atapi!IdePortInsertByKeyDeviceQueue` \->

`atapi!StartIo` `Packet` \->

`HAL`->

`Io` 端口……也就是说：`IRP` 是从 `Ntfs` ->`ftdisk` （卷设备） -> `class`(`classpnp` )-> `atapi` -> `hal` -> `IO` 的流向的。

那么`\\.\\Physical` `Drive%d` 啊，`\Device\Harddisk%d\Partition0` 啊~ 是表示什么呢？通过 `DeviceTree` 和 `winobj` 可以得知，

`\\.\\PhysicalDrive` 只是 `\Device\Harddisk%d\Partition0` 的符号连接，

而 `\Device\Harddisk0\Partition0` 是`\Device\Harddisk0\DR0` 的符号链接。

`\Device\Harddisk0\Partition1`、2……则是 `\Device\HarddiskVolume1`、2 的符号连接。

那么设备堆栈是怎么回事呢，原来从上到下是这样：

`FS` `DRIVER--->>`

`Volsnap--->>>`

`ftdisk`（`\Device\HarddiskVolume1`）--->>

`partmgr`（`Ipartmgr` ）--->>

`disk`（`\Device\Harddisk%d\Partition0\` `DR0`）--->>

`acpi--->>>`

`atapi`（ `IdeDeviceP0T0L0-3`）

每层都是 attached 到上一层，所以形成设备链的。括号内表示驱动对象创建的设备对象，用来形成设备堆栈并接受 `IRP` 的。

(所以过滤驱动挂载到任意一层时遍能拦截到数据，这也是微软提倡的“HOOK”方式)

这也就对应了微软的说法，`IRP` 是从文件系统->卷驱动 -> 磁盘驱动-> 类驱动-> 端口驱动-> 微端口驱动的流程的。

而 `\Device\Harddisk%d\Partition0\` `DR0` 则是 `disk` 驱动对象创建用来处理 `ClassReadWrite` 等 `IRP` 的设备对象了。我看了 `disk` 的 `IRP_MJ_READ` 确实是 `CLASSPNP!ClassReadWrite`，这就可以接受为什么 `IRP` 到了 `disk` 这层反而到了 `classpnp` 中去处理 `IRP` 了。

另外，`MJ` 前辈还说过：

1. 文件系统设备站和 storage 设备站使用同一个 `irp`，到了 `atapi`，这个 `MINIPORT`，就换成 `pkt` 中的另一个 `Irp`，
2. 如果中间没有人重新转发 `IRP`（例如 `DISKF`）的话，从 `FSD` 到 `ATAPI` 都是用的同一个 `IRP`。`IdePortDispatch` 没有源代码，而 `classpnp` 是开源的（`wdk` 就有），可参考 `classpnp` 中的 `ServiceTransferRequest` 和 `SetupReadWriteTransferPacket`。

`IRP` 再往下会到哪呢？原来通过在 `ATAPI` 中通过 `IdePortInsertByKeyDeviceQueue` 将 `IRP` 入队列，再往下就是 `ATAPI` 的 `StartIo` 了。`StartIo` 再往下就没有 `IRP` 的概念了，便到达 `HAL`（硬件抽象）层，再往下就是端口 `IO` 了！

下面是大米同学的图，画的很好，先引用一下^\_^：

```
IRP |

        |

        |           attach dev               attach dev

        |               |                       |

   ntfs/fat32.sys  --> dev --> dev --> ... --> dev

        |               |                       |

        |           attach dev               attach dev

        |               |

        |              ...

        |

        |

        |

        |              ...

        |               |

      partmgr.sys  --> dev --> dev --> ... --> dev

        |               |

        |              ...

        |

        |

        |                                             dev ( \\driver\\partmgr)

        |               |                       |

      disk.sys   --> dev --> dev --> ... --> dev (DR0)

        |              |

        |

        |                                             dev ( \\driver\\partmgr)

        |                                             dev (DR0)   (\\driver\\disk)

        |               |                       |

      atapi.sys  ->  dev --> dev --> ... --> dev

        |

        |

      ...
```

另外 `AZY` 前辈也说过：

读写 `IRP->IdePortDispatch->IoStartPacket->IdePortStartIo`，到这里分两支

有 `DMA` 能力的走: `BmSetup->BmReceiveScatterGatherList->IdePortAllocateAccessToken->xxx`

无 `DMA` 能力的走:`IdePortAllocateAccessToken->CallIdeStartIoSynchronized->IdeStartIoSynchronized->AtapiStartIo->IO` 端口 `xxx`
