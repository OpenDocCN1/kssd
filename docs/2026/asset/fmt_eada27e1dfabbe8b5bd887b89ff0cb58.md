
![](https://bbs.kanxue.com/upload/attach/201711/624619_27yqkhv8rrmqjlc.png)

图1-27

下面先在 VMware 中新建一个虚拟机，并分配不少于 4G 的内存，使用“NAT 网络地址转换”作为网络方面的配置。然后安装 Windows 系统，笔者这里安装的虚拟机系统版本是 Windows 10 14393 RS1。安装好系统后，关机，下面需要更改一些配置。

打开 VMware 的“虚拟机设置”面板，在“硬件”选项卡中，选择“处理器”选项，如图1-27。图1-27中，在“虚拟化引擎”中的“首选模式”选择“Intel VT-x/EPT 或 AMD-V/RVI”，并勾选上“虚拟化 Intel VT-x/EPT 或 AMD-V/RVI(V)”和“虚拟化 CPU 性能计数器”两项。点击确定保存配置并退出。

然后修改创建的虚拟机的配置文件，找到创建的虚拟机文件位置，在虚拟机目录下，找到`Windows 10 x64.vmx`文件（这里以笔者机器为例，你的机器上应该是：你的虚拟机名称.vmx）。然后用编辑器打开这个文件，在文件的末尾加上：

```
hypervisor.cpuid.v0 = “FALSE”

mce.enable = “TRUE”

vhv.enable = “TRUE”
```

在 vmx 文件中可能会存在上面中的配置语句，那么先要检查下原有的配置是否和上面语句相同，若不相同需要修改，然后把 vmx 文件中未出现的配置语句加到文件末尾，保存文件退出。

“hypervisor.cpuid.v0”设置成 FALSE 的意义为欺骗虚拟 Windows 系统没有运行在虚拟机中；“mce.enable”设置成 TRUE 意义为启用 Machine Check Exception（MCE）；“vhv.enable”设置成 TRUE 意义为启动嵌套虚拟化。

下面启动修改好配置的虚拟机，然后就可以按照 6.1.2 章节的方法安装和配置 Hyper-V 了。

## 第二步：配置虚拟 Windows 主机的调试环境

现在我们已经成功在 VMware 虚拟机中安装好 Windows 系统并且可以运行 Hyper-V 虚拟机了。下面还要对 Windows 虚拟机进行配置，实现网络双机调试。

由于我们在创建虚拟机时网络配置选择的是“NAT 网络地址转换”，所以虚拟机的 IP 是 192.168.230.175。那么宿主机便是 192.168.230.1。如图1-28。如果宿主机和 VMware 虚拟的 Windows 系统之间能互相 ping 通对方，说明网络配置正常，可以继续下面的配置。

![](https://bbs.kanxue.com/upload/attach/201711/624619_ibkq1dxrwwc5zgs.png)

图1-28

同样的，在打开的命令提示符中，我们输入如下的命令：

```
$bcdedit /set dbgtransport kdnet.dll

$bcdedit /dbgsettings NET HOSTIP:192.168.230.1 PORT:50000

$bcdedit /debug on

$bcdedit /hypervisorsettings NET HOSTIP:192.168.230.1 PORT:50001

$bcdedit /set hypervisordebug on

$bcdedit /set hypervisorlaunchtype auto
```

上面的命令已经在 3.2 章节中做过介绍了，不同的是第 2 个命令和第 4 个命令。这两个命令作用分别是：在被调试机系统启动过程中连接到 IP 为 192.168.230.1、端口为 50000 的调试机，用于调试被调试机的 Windows 内核代码；在被调试机系统启动过程中连接到 IP 为 192.168.230.1、端口为 50001 的调试机，用于调试被调试机的 Hypervisor 层代码。

以上命令执行完成后，查看配置信息。输入如下命令：

```
$bcdedit /dbgsettings

$bcdedit /hypervisorsettings
```

结果如图1-29所示，这两个命令分别显示出了刚才设置的配置。其中，“key”字段十分重要，它相当于调试的凭证，如果凭证不对是无法调试目标主机的，所以它需要我们记录下来，并且妥善保存。注意，其中的 key 值因不同的机器而异。

![](https://bbs.kanxue.com/upload/attach/201711/624619_6oswxgknoqslqqe.png)

图1-29

上面的配置见表1-5。

![](https://bbs.kanxue.com/upload/attach/201711/624619_1liti2zfs04cxj0.png)

这张表中的数据需要保存下来，因为在下面的配置中会用到。

## 第三步：配置调试机的调试环境

现在我们开始配置调试机端。还是和 3.2 章节中介绍的一样，新建一个快捷方式，在“请键入对象的位置”中填入`"C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\windbg.exe" -k net:port=50000,key=1alb1q6593i2s.1k6o4ts8tzv39.1w72krxwkrv11.3bqw9vhvl5uth`，并命名为“kernelroot”，用于调试 Windows 内核代码。同样的，再新建一个快捷方式，在“请键入对象的位置”中填入`"C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\windbg.exe" -k net:port=50001,key=n05t1fb0wyf0.137txzjv7e9ze.1srubjv5hkoo9.1iw56k4pkn6a3`，并命名为“hypervisor”，用于调试 Hypervisor 层代码。

和 3.2 章节不同的是，网络双机调试不需要运行`vmdemux`命令，不同的端口对应不同的调试目标。下面我们双击运行两个快捷方式，然后开启虚拟机电源，等待被调试机主动连上调试机的 WinDbg。

如图1-30，调试机中的 WinDbg 分别捕获到了 Hypervisor 层和 Windows 内核的调试信息，并且可以随时中断被调试机的运行。通过 WinDbg 对被调试机简单的调试操作，会明显地发现在调试速度上有了明显的提升，基本和调试用户态下的程序速度相同。这对我们对 Hyper-V 进行安全研究提供了非常有利的条件。

所以，综上所述，在硬件条件允许的情况下，还是优先选择使用 VMware 进行双机调试的办法去调试 Hyper-V，这样做确实很省时省力。笔者曾经使用串口进行双机调试，一条汇编指令运行 2-3 秒都是很常见的情况，在调试中浪费了大量的时间。

![](https://bbs.kanxue.com/upload/attach/201711/624619_ml2qb6qpsx6wujt.png)

图1-30

---

本文如需引用转载请联系本文作者，看雪 ID：ifyou

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [3.pdf](javascript:void(0);)（1.89MB，119次下载）
