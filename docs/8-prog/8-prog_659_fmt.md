
# 抢先 DriverStudio 夺取机器控制权(修改 x1)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-649.htm`](https://www.pediy.com/kssd/pediy09/pediy09-649.htm)

\[原创\] 抢先 DriverStudio 夺取机器控制权

（这是以前早些时候研究 `ring0` 代码时随笔写的，现整理发表出来，请各位指教。）

废话不谈，言归正传！

大家都知道，装了 `DriverStudio` 软件(我使用的是 `v3.2` 版)的系统在启动时会显示其配置画面，(如图 0 所示)

![](img/084cd72ccc345b5c779839ac2b6bfb58.jpg)

图 0

这时操作系统的其他部件还没有运行。那么显示的原理是什么？能不能早于 `DriverStudio` 而先执行其他代码呢？答案是肯定的。下面先谈谈原理吧。

原理比较简单，详细的大家可以看网文《《Inside NT boot》》，我这里简单说说。当引导扇区被引导后，会查找 `NTLDR`, `NTLDR` 做的一件非常重要的事就是将 `CPU` 从实模式转换到保护模式下。在加载完 `NTDECT.COM` 后接下来就会将 `NT` 的核心装入内存，它们是 `HAL.DLL` 和 `NTOSKRNL.EXE`，加载完毕后，`NTLDR` 再加载所有引导必须的驱动程序。接下来就是我们所关心的地方: 它会将 `HKLM\SYSTEM\Services` 中值为 `SERVICE_BOOT_START` 的 `DRIVER` 装入，但此时不初始化。

(补充附注:

`NtLdr` 第 3 步动作: 扫描内存中 `SYSTEM` 注册表 hive 文件找到所有引导设备驱动程序，这些 `Driver` 在注册表中通过 `SERVICE-BOOT-START` 启动值标记。

`NtLdr` 第 5 步动作: 加载引导驱动程序，在启动画面 `Starting Windows` 下显示出不断更新的进度栏，该进度栏随每个驱动程序的加载而变化。

`Kernel` 第 23 步动作: `I/O` 管理器初始化……所有前面加载的 `Boot-Start` 型驱动程序被调用以完成驱动程序相关初始化，这时才轮到 `System-Start` 型的驱动程序被加载并初始化。

更详细的信息请参考《《Windows 2000 内部揭秘》》)

那么 `DriverStudio` 是不是也是使用了这个特性呢？

运行 `regedit.exe` ,在 `Services` 目录中找到 `bootcfg` 这个注册项，可以看到它是一个内核驱动，并且 `Start` 类型为 `0`。如图 1 所示:

![](img/9daaa1152b5ed254854b9ab87b71cdb5.jpg)

图 1

为了证实这个 `Driver` 就是提供 `DriverStudio` 开机时配置功能的驱动程序，我用一个会引起系统崩溃驱动的名字来替换它，然后重新启动。原本该显示配置画面的地方果然发生了系统崩溃。

(如图 2 所示)

![](img/a99c3f035afe6b6dabdbe6ead9fea838.jpg)

图 2

既然知道了原理，再来看看操作。实际上 `windows` 自身提供了这个实现。细心的朋友可能早就知道 `Win32 API` `CreateService` 的 `dwStartType` 形参有个选项为: `SERVICE_BOOT_START` 。

```
SERVICE_BOOT_START -- Specifies a device driver started by the operating system loader. This value is valid only if the service type is SERVICE_KERNEL_DRIVER or SERVICE_FILE_SYSTEM_DRIVER.
```

好像看起来和普通的诸如 `SERVICE_DEMAND_START` 的用法没什么不同嘛，结果手动编制尝试结果返回非法参数。

为什么会这样呢？经过一番 thinking 之后，我认为是由于使用了 `SERVICE_BOOT_START` 时，是在系统引导的早期发生 load Driver 事件，这时可能除了少数几个 `OS` 必须的路径以外，还无法访问其他 `windows` 目录。

为了证实，我将自己的 Driver 拷贝到系统目录：

```
%root%\system32\drivers\
```

下，再次运行结果成功！

下面再引出注册表中的一个与驱动程序加载相关的 KEY：`ServiceGroupOrder` ，如图 3 所示:

![](img/085bbfc8fc2e136c225e1505aa14d64a.jpg)

图 3

这个 KEY 决定了所属改组的驱动程序的加载顺序。

再看一下图 1 中 `DriverStudio` 引导驱动的组名，是 `Boot Bus Extender`，它排在启动顺序的第 2 位。现在我新加一个 Group 名: `hopy`。同时还要修改 `ServiceGroupOrder` 在其中添加新组 `hopy`, 要注意的是要以 `UNICODE` 的格式添加字符串 `"hopy"`。如图 4:

![](img/e096e412e1c7d322b46d54dbf2a2dfd3.jpg)

图 4

然后将代码改写如下:

```asm
;BTmain.asm 节选
.const
namesvr db 'TryBootSvr',0
notesvr db 'Test Drv Start at System Boot',0
namefile db 'BTdrv.sys',0
szLOG db 'hopy',0
szSSN db 'TBSvr2007',0 ;ServiceStartName
szpath db 'C:\WINNT\system32\drivers\BTdrv.sys',0

.code

invoke CreateService,hSCM,addr namesvr,addr notesvr,\
SERVICE_ALL_ACCESS,SERVICE_KERNEL_DRIVER,\
SERVICE_BOOT_START,SERVICE_ERROR_NORMAL,\
addr szpath,addr szLOG,addr tagid,NULL,\
NULL,NULL
```

驱动代码如下:

```asm
;BTdrv.asm 节选
local status:NTSTATUS
local pDeviceObject:PVOID

mov status, STATUS_DEVICE_CONFIGURATION_ERROR
xor edi,edi
mov [edi],eax
jmp $
mov eax,status
ret
```

结果如我们所预料，该 Driver 抢在 `DriverStudio` 之前发生蓝屏，如果将这个 Driver 换成带有特定功能的代码的驱动，则可以实现超前的目的。这个留给大家去想象吧，呵呵。

（图看不完整的话可以点击察看完整视图）

(唉，格式怎么也弄不好，诸位抱歉啦。)

`hopy` | 侯佩
2007.09.16 整理于中国 女足惨败于巴西之后
