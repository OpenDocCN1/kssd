
# [原创]Sfilter中的那些绑定(学习笔记)

## 1. 首先是挂载Sfilter创建的设备到文件系统上。

### （1）在`DriverEntry`中`IoRegisterFsRegistrationChange`注册一个回调函数`SfFsNotification`，当一个文件系统注册或者注销时，从而触发回调函数`SfFsNotification`。

{
当文件系统注册的时候会调用`IoRegisterFileSystem`，
创建一个文件系统控制设备对象（file system control device object，FSCDO）到文件系统对象的队列中。注销时会调用`IoUnregisterFileSystem`，
把前面创建的FSCDO从队列中移除
}

### （2）在回调函数`SfFsNotification`中，会判断这是激活还是撤销一个文件系统

![](https://bbs.kanxue.com/upload/attach/200906/117003_m2k0cqgqlfag0fh.jpg)

### （3）首先看文件系统注册的处理， `SfAttachToFileSystemDevice`

在`SfAttachToFileSystemDevice`中，我们只绑定
`FILE_DEVICE_DISK_FILE_SYSTEM`、
`FILE_DEVICE_CD_ROM_FILE_SYSTE`、
`FILE_DEVICE_NETWORK_FILE_SYSTEM`，这三种类型的文件系统。
所以先要用`IS_DESIRED_DEVICE_TYPE`判断是不是这三种类型的文件系统，然后再把文件系统识别器排除。接着调用`SfAttachDeviceToDeviceStack`，绑定我们的设备对象到文件系统的控制设备对象的设备栈上。

然后调用`SfEnumerateFileSystemVolumes`枚举当前文件系统下，所有的当前安装的设备，并绑定它们。

{
`SfEnumerateFileSystemVolumes`枚举的过程是这样的：
调用`IoEnumerateDeviceObjectList`枚举这个驱动下的device object list。
`IoEnumerateDeviceObjectList`在这个函数要调用了两次。
第一次调用是获取list中device objects的数量，
然后根据大小开辟存放device objects的空间，第二次调用才得到设备链。
之后就是遍历已经得到的device objects list，排除CDO、不符合的类型的、已经绑过的这三种设备，不绑定。
绑定时调用`SfAttachToMountedDevice`，在`SfAttachToMountedDevice`设置我们创建的设备的Flag，然后调用`SfAttachDeviceToDeviceStack`。
在`SfAttachDeviceToDeviceStack`中，终于是最终的绑定了，调用`IoAttachDeviceToDeviceStackSafe`，到这里就完成了绑定。
}

流程图表示就是：

![](https://bbs.kanxue.com/upload/attach/200906/117003_46xsosgbj9ycrvo.jpg)

## 2. 就像WINDOWS是用消息驱动一样，驱动是用IRP驱动的。

当Sfilter接收到`IRP_MJ_FILE_SYSTEM_CONTROL`这个IRP，会调用`SfFsControl`去处理。

{
先简单说下`IRP_MJ_FILE_SYSTEM_CONTROL`：
一个新的存储媒质被系统发现,并在文件系统中生成一个卷(Volume)的过程称为Mounting, 这时I/O管理器会发送`IRP_MJ_FILE_SYSTEM_CONTROL`.
他的副功能号有：
`IRP_MN_KERNEL_CALL`
`IRP_MN_MOUNT_VOLUME`
`IRP_MN_USER_FS_REQUEST`
`IRP_MN_VERIFY_VOLUME`
`IRP_MN_LOAD_FILE_SYSTEM`
Sfilter中处理了以下三个：
`IRP_MN_MOUNT_VOLUME`
这是一个卷的mount请求
`IRP_MN_LOAD_FILE_SYSTEM`
这是文件系统识别器的副功能号，是一个加载文件系统的请求
当一个文件识别器决定加载真正的文件系统的时候，会产生一个这样的irp
调用`SfFsControlLoadFileSystem`处理。
`IRP_MN_USER_FS_REQUEST`
这是一个文件系统操作的请求
}

当确认了一个卷的mount请求，也就是`IRP_MN_MOUNT_VOLUME`，
Sfilter调用`SfFsControlMountVolume`处理。

{
`SfFsControlMountVolume`这里面的处理大约就是判断下是不是卷影和调用`SfAttachToMountedDevice`实现对卷的绑定。
}

当确认了一个加载文件系统的请求，也就是`IRP_MN_LOAD_FILE_SYSTEM`，Sfilter调用`SfFsControlLoadFileSystem`，在`SfFsControlLoadFileSystem`的完成函数`SfFsControlLoadFileSystemComplete`中调用`SfAttachDeviceToDeviceStack`对文件系统识别器进行绑定。

当确认了一个文件系统操作的请求，也就是`IRP_MN_USER_FS_REQUEST`
，查看`irpSp->Parameters.FileSystemControl.FsControlCode`中的值是否为`FSCTL_DISMOUNT_VOLUME`。如果是`FSCTL_DISMOUNT_VOLUME`则标志着这里是要Dismounts a volume。

就先写到这里吧。
因为学驱动没多久，文中应该有不少错误，希望大家不吝指出。
脸丢了，东西就学到了 ^\_^

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [1.jpg](javascript:void(0);)
  （26.01kb，288次下载）
- [2.jpg](javascript:void(0);)
  （33.41kb，286次下载）
