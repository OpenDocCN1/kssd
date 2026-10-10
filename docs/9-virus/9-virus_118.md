

# Stuxnet 蠕虫(超级工厂病毒)驱动分析

> 原文：[`www.pediy.com/kssd/pediy11/125160.html`](https://www.pediy.com/kssd/pediy11/125160.html)

Stuxnet 蠕虫驱动分析  
  
  
这里只是分析文字, 贴图影响速度, 就不贴了, 具体看附件 pdf 文档, 难免有错误之处, 欢迎指正!  
  
一. Stuxnet 蠕虫(超级工厂病毒)简单说明  
  
能够利用对 windows 系统和西门子 SIMATIC WinCC 系统的 7 个漏洞进行攻击,被称为是新时代网络战争开始的标志,  
也有人宣称是"政府发动的网路战争、带有圣经讯息、最高机密等",呵呵,有点雷人!  
  
windows 下最主要的传播途径是攻击快捷方式自动执行漏洞(MS10-046), WindowsServer 服务的远程溢出漏洞  
(MS08-067)以及打印后台程序服务中的远程代码执行漏洞(MS10-061)。  
  
病毒成功攻击了伊朗核电站,造成伊朗核电站推迟发电.由于能够对西门子公司的 SIMATIC WinCC 监控与数据采集   
(SCADA) 系统进行攻击,因此也能攻击我国的钢铁、电力、能源、化工等重要行业,  
  
Stuxnet 超级工厂病毒直到 2010-09-25，才在传入国内网络.我是在近两个月前才收到的,当时只需要简单分析用户态的部分样本,因此没有对驱动进行分析.后来闲着没事时,就分析了驱动部分.不过,此病毒强大之处大部分都在用户态实现.  
  
  
二.Stuxnet 蠕虫(超级工厂病毒)驱动(mrxnet.sys)分析  
  
从 Stuxnet 蠕虫病毒样本中提取出了两个驱动文件,mrxnet.sys(17k),mrxcls.sys(26k),其中驱动 mrxnet.sys 在蠕  
虫通过 u 盘驱动器传播时被用来隐藏特定文件,这个驱动是一个文件系统过滤驱动,支持三种文件系统 ntfs,  
fastfat 和 cdfs 文件系统.驱动文件 mrxcls.sys 则被用于向用户空间中注入代码,被注入的模块被放在一个配  
置文件中,这应该就是最初让卡巴斯基头痛的"父进程注入"技术,轻松地绕过卡巴.这里单就只分析驱动 mrxnet.sys.  
  
入口处开始分析:  
  
1.此驱动首先创建一个设备类型为 FILE\_DEVICE\_DISK\_FILE\_SYSTEM,设备特征为 FILE\_DEVICE\_SECURE\_OPEN,  
设备扩展为 8 字节的无名设备对象.这个驱动没有控制设备,因此不会跟用户态交换信息.  
  
其中设备扩展的类似如下定义:  
DeviceExtension  
{  
PDEVICE\_OBJECT LowerDevice;  
PDEVICE\_OBJECT RealDevice;  
}  
其中 LowerDevice 为本驱动的下层驱动,RealDevice 为 VPB 中的 RealDevice 设备,其实也就是文件系统的卷设备.  
  
2.接下来注册 MajorFunction:  
  
IRP\_MJ\_FILE\_SYSTEM\_CONTROL  
IRP\_MJ\_DIRECTORY\_CONTROL  
  
这两个派遣例程,这个驱动主要就处理这两例程,其他的用一个例程处理,其实是直接 pass 了,不做任何处理.  
两个例程分别是 FileSystemControl,DirControl.是本驱动核心功能的实现,留到最后阐述.  
  
3.接着就是填充 FastIoRoutines,驱动并没填充所有的 FastIoRoutines,这里填充了如下:  
FastIoCheckIfPossible;  
FastIoRead;  
FastIoWrite;  
FastIoQueryBasicInfo;  
FastIoQueryStandardInfo;  
FastIoLock;  
FastIoUnlockSingle;  
FastIoUnlockAll;  
FastIoUnlockAllByKey;  
FastIoDeviceControl;  
FastIoDetachDevice;  
FastIoQueryNetworkOpenInfo;  
MdlRead;  
MdlReadComplete;  
PrepareMdlWrite;  
MdlWriteComplete;  
FastIoReadCompressed;  
FastIoWriteCompressed;  
MdlReadCompleteCompressed;  
MdlWriteCompleteCompressed;  
FastIoQueryOpen;  
这些 FastIoRoutines 没有做什么实质性的工作,只是把简单判断一下,然后调用下层驱动.  
  
4.然后就是 attach 到文件系统了,函数:AttatchTargetDrivers.  
先获取通过 MmGetSystemRoutineAddress 动态获取 ObReferenceObjectByName 函数的地址,将其地址作为参数去  
获取文件的驱动对象,文件系统名为:  
\\\\FileSystem\\\\ntfs  
\\\\FileSystem\\\\fastfat  
\\\\FileSystem\\\\cdfs  
支持的文件系统有 ntfs,fastfat,cdfs.  
这三个文件系统名的地址被放在一个指针数组中,这个指针数组以后会被通过 ObReferenceObjectByName 获取的  
驱动对象覆盖;  
  
这里有个循环,循环 3 次,分别去 attach 这三种文件系统.  
这个 attach 的函数是 GetTargetDriverAndAttach(),函数有两个参数,第一个参数是函数 ObReferenceObjectBy  
Name 的地址,第二个参数是文件系统文件名指针,这个参数即用来传入文件系统名地址,也用于获取所得的驱动  
对象指针.进入这个函数内部,函数先获取文件系统驱动对象,然后遍历驱动对象的每个设备对象进行 attach,  
attach 每个设备对象的函数是 TryToAttachDevice()  
  
TryToAttachDevice()  
这个函数有两个参数,第一个是设备对象(既是文件系统的设备对象),第二个是 BOOLEAN 型(这里调用时传入的为  
true),用于判定这个文件系统是被注册(激活),还是卸下.如是被注册,则调用函数 CreateFilterDeviceAndAtta  
ch()否则 Detach 本驱动在文件系统设备栈中的设备.  
  
CreateFilterDeviceAndAttach()这个设备有一个参数,参数为文件系统的设备.  
这个函数只 attach 设备类型为磁盘文件类设备,CD\_ROM 文件类设备,网络文件类设备.  
FILE\_DEVICE\_DISK\_FILE\_SYSTEM,  
FILE\_DEVICE\_CD\_ROM\_FILE\_SYSTEM  
FILE\_DEVICE\_NETWORK\_FILE\_SYSTEM  
先判断设备类型满足以上条件,如果这个文件系统设备还没有被 attach,则创建一个本驱动的设备 attach 到文  
件系统设备栈中,Attach 是通过函数  
AttachToFsDevie 来实现的,这个函数很简单,只是把 attach 设备后,把下层设备保存于设备扩展中.  
  
这样就完成了 attach 工作.  
  
  
5.调用函数 IoRegisterFsRegistrationChange 注册文件系统变动回调例程.这个例程也就是 4 讲到的  
TryToAttachDevice()函数,用于完成对动态激活的文件系统进行 attach.  
  
  
  
IRP\_MJ\_FILE\_SYSTEM\_CONTROL 和 IRP\_MJ\_DIRECTORY\_CONTROL 的派遣例程处理,这两个例程都只处理  
IRP\_MN\_MOUNT\_VOLUME 类型的次功能码,主要是为了能在新的文件系统被激活时,做相应的处理.其他的  
直接 pass,不做处理.  
  
1.FileSystemControlRoutine 例程  
首先获取 MinorFunction,判断是否是 IRP\_MN\_MOUNT\_VOLUME,是则调用 MnMountVolumeForFSC,否则 pass;  
  
2.MnMountVolumeForFSC()  
函数先创建了一个设备对象,然后初始化设备扩展,在设备扩展的第二成员里保存 VPB 中 realDevice.然后是  
自己实现了将当前设备的 Stack Location 拷贝到下层设备的 Stack Location 处,并且设置好完成例程,  
Context 和控制域.完成例程名为  
CompleteRoutineForFSC.  
  
3.完成例程 CompleteRoutineForFSC.  
完成例程的 context 参数传进来是我们创建的过滤设备对象,通过这个设备对象得到原先保存的 vpb->RealDevice,  
再根据这个设备得到 vpb 然后在获得 VPB->DeviceObject,这个设备则是文件系统的卷设备,为什么要这么做,  
因为在 mount 之前我们创建的设备的 VPB 可能在 mount 完成后就不同了,而完成后的 VPB 才是有效的.故这里  
先将文件系统的卷设备保存在设备扩展中,后在完成例程里重新获取.得到文件系统的卷设备对象后就可以  
进行 attach 了.attach 完成例程函数的主要工作也就算是完成了.  
  
1.DirControlRoutine  
其实重头戏在函数 DirControlRoutine 中,这个函数主要实现文件隐藏功能.  
同样也只处理 IRP\_MN\_MOUNT\_VOLUME,判断后直接调用函数 MnQueryDirectory.  
  
2.MnQueryDirectory  
这个函数先获取查询目录(文件)名,  
如果文件名存在并且文件名长度 76,且文件名前 19 个字符全是'{'的话,就放过(这里有点不解,为什么要设置这个例外呢.  
),否则复制当前栈道下层栈,并设置完成例程 IoCompleteRoutineForQueryDir,将 irp 直接下发,这里栈的 COPY 和完  
成例程的设置前面一样,是自己实现的.  
  
3 完成例程 IoCompleteRoutineForQueryDir  
这个函数比较大,主要是做文件过滤隐藏的.  
先判断 IRP->IoStatus.Status 是否成功,如果不成功则释放相关资源并退出,然后就是获取一些文件相关的  
偏移,从函数 GetFileInforOffset 中获取 EndOfFile Offset,FileName Offset,FileNameLength Offset.因为  
这里会因为文件信息类 FileInformationClass 的不同而会有不同的偏移.  
  
得到偏移后就可以这里涉及的文件信息类有  
FileBothDirectoryInformation  
FileDirectoryInformation  
FileFullDirectoryInformation  
FileIdBothDirectoryInformation  
FileIdFullDirectoryInformation  
FileNamesInformation  
  
接着就是得到查询信息的缓冲,然后调用 HideFile 来隐藏文件.  
  
4.隐藏文件函数 HideFile  
此函数中,从前面获取的几个偏移从缓冲中定位文件名地址,文件长度,文件结束位置.  
然后就是调用两个函数文件名过滤函数(CheckPostfix,CheckNameInvlidForTMP)来筛选病毒关心的文件.  
在函数 CheckPostfix 中判断文件名是否是一".LNK"后缀的文件名,如是则符合隐藏规则,在函数 CheckNameInvlid 中,  
如果文件大小在 0x1000~ 0x800000,文件名长度为 12,文件后缀名为".TMP",文件名第一个字符为"~",  
并且文件名从第 5 个字节到第 8 个字节为数字,并且这些数字之和必须等于 10,便符合规则.  
符合以上两种规则的文件实施隐藏,隐藏处理是,如果条件符合,这把后面文件记录向前移动一个文件记录,  
也就是覆盖病毒关心的文件记录.

上传的附件
| ![](img/668feaf58c8e7ddb6fbdbf839a5a19db.jpg) || Stuxnet 蠕虫驱动分析.pdf  
 |