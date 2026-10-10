

# 将 VMware 与 SoftICE 基于网络的远程调试功能相结合

> 原文：[`www.pediy.com/kssd/pediy05/pediy50510.htm`](https://www.pediy.com/kssd/pediy05/pediy50510.htm)

作者：scz（小四)   
  
2.0 将 VMware 与 SoftICE 基于网络的远程调试功能相结合   
  
Q: 在 VMware 上安装 SoftICE，总是无法正确配置显卡驱动，选择   
  
  Universal Video Driver(SoftICE appears in a "window")   
  
  测试失败。因此无法在 VMware 中 Ctrl-D 呼叫出 SoftICE 的屏幕。能否对 VMware 机进   
  行基于网络的远程调试。   
  
A: 小四 <scz@nsfocus.com> 2003-03-20 09:04   
  
经过四个小时测试、调整，答案是肯定的。我使用 VMware Workstation 3.0，其上安   
装了英文版 Windows XP SP1 以及 DriverStudio 2.7 所携带的 SoftICE，版本如下:   
  
SoftICE (R) - DriverStudio (tm) 4.2.7 (Build 562)   
  
VMWare 外部机器也是英文版 Windows XP SP1。下面未提与远程调试无关的 SoftICE 配   
置。   
  
\--------------------------------------------------------------------------   
1) 确认在 VMware 设置中只虚拟出一块物理网卡。在 VMware 中安装 XP，是否打 SP1 与远   
  程调试无关。   
  
  安装操作系统完成后，系统属性->硬件->设备管理器，确认只有一块物理网卡，   
  并且正常工作中(无黄色问号、无红色叉号)。   
  
  进入网络属性，将与现有物理网卡关联的原有驱动程序删除。禁用(不建议删除)   
  已安装协议、客户端软件、服务方软件，就是将前面复选框中的对号全部勾去不   
  要。   
  
  如果不是新装 XP，假设有与上述不相符的配置，比如已经虚拟出两块物理网卡，   
  已经进行过 TCP/IP 配置，最好能重新配置一下，删除一块物理网卡、取消现有的   
  TCP/IP 配置。当然，你可以不改动原有配置，一般来说不会出什么问题，真出了   
  问题再按我说的做也来得及。   
  
  对于真实的远程主机，如果有两块物理网卡，必须先从插槽拔下一块来，只保留   
  一块物理网卡在插槽中。   
  
2) SoftICE 在\\Program Files\\Compuware\\DriverStudio\\SoftICE\\Network\\下提供了   
  三种网卡驱动(3COM/3C90X、Intel/E100、Novell/NE2000)，如果你的网卡在这三   
  种驱动支持范围内，可以立即完装或更新网卡驱动，选择从磁盘安装，指定上述   
  安装目录。安装驱动安成后，可以配置 TCP/IP 协议等等。   
  
  这个版本的 VMware 虚拟出来的网卡是 AMD PCNET Family PCI Ethernet Adapter。   
  
  按照以前 BBS 流行说法，如果你的网卡不在上述三种驱动支持范围内，就无法进行   
  基于网络的远程调试。总不能为了远程调试而专门换网卡吧，某些便携机的网卡   
  还不是那么容易更换的。幸运的是有办法让几乎所有类型的网卡都支持基于网络   
  的远程调试。执行:   
  
  \\Program Files\\Compuware\\DriverStudio\\SoftICE\\Network\\UND\\UNDSetup.exe   
  
  在 Available Device 中可以看到当前物理网卡，选中它，选择收音机按钮   
  
  Use Universal Network Driver   
  
  一路确定并重启 VMware 中的 OS。重启之后，你会发现网络属性里没有任何物理网   
  卡了。事实上，无论你以前有多少块物理网卡、做过多少网络配置，都会"丢失"。   
  不要慌，如果想恢复，再次执行:   
  
  \\Program Files\\Compuware\\DriverStudio\\SoftICE\\Network\\UND\\UNDSetup.exe   
  
  在 Available Device 中可以看到"SoftICE network transport"，选中它，选择收   
  音机按钮   
  
  None   
  
  一路确定并重启 VMware 中的 OS。重启之后，你以前设置就基本恢复了。前面我的   
  建议部分出于完美主义倾向，可能不是必须的，现在你理解了么。   
  
3) 当网络属性里没有任何物理网卡的时候，已经可以进行基于网络的远程调试。与   
  "Remote Access"设置无关，将那里所有复选框中的对号全部勾掉。不要使用   
  "Network Debugging"设置，据 yuange 说，这里处理有问题，他修改了 ntice.sys。   
  我没有修改 ntice.sys，而是使用 General->Initialization，内容如下:   
  
  faults off;set font 3;lines 43;net start 192.168.7.153 mask=255.255.255.0 gateway=192.168.7.254;net allow 192.168.7.2 auto password=123456;X;   
  
  VMware 外部的机器 IP 为 192.168.7.2，所以有如上设置。只设置了 lines 而没有设   
  置 width，因为后者需要"Universal Video Driver"模式，否则不可调。   
  
  虽然在 VMware 中 Ctrl-D 呼叫不出 SoftICE 屏幕，但实际上已经呼叫成功，可以盲打。   
  因此，如果没有做如上设置或基于其它原因需要动态修改、测试时，可以在盲打   
  输入:   
  
  Ctrl-D   
  net start 192.168.7.153 mask=255.255.255.0 gateway=192.168.7.254   
  net allow any auto   
  X   
  
  其中 192.168.7.153 就是远程 SoftICE 所使用的 IP 地址，"net allow any auto"表   
  示对操作方无任何 IP、口令限制，对于初次测试远程调试功能的人来说，最好使   
  用这样的设置，不需要重启机器，即刻生效。   
  
  留心盲打，碰上键盘、鼠标无响应时，很可能是 SoftICE 被呼出来了，输入 X 退出   
  试试，不要急于按电源重启。   
  
4) 如果远端设置是:   
  
  net allow any auto   
  
  在 VMware 外部机器上执行如下命令:   
  
  \\Program Files\\Compuware\\DriverStudio\\SoftICE\\siremote.exe 192.168.7.153   
  
  成功的话就会看到你熟悉的 SoftICE 窗口。如果远端设置了口令:   
  
  net allow 192.168.7.2 auto password=123456   
  
  在 VMware 外部机器上执行如下命令:   
  
  \\Program Files\\Compuware\\DriverStudio\\SoftICE\\siremote.exe 192.168.7.153 21321 123456   
  
  这种远程调试使用了 21321/UDP 端口，123456 是口令。siremote.exe 有 BUG，为了   
  指定口令，必须指定目标端口，可我在文档中没有找到相关说明，还好老夫是折   
  磨 Sniffer Pro 出身，对"net allow any auto/siremote.exe 192.168.7.153"通   
  信过程捕包一观，才确定目标端口是 21321/UDP。不过，我不敢确定你们那里也使   
  用同样的端口，假设有问题，请立即捕包确定。   
  
  net start、net allow 这些命令不能在远程窗口中使用，只能在本地窗口中使用，   
  因此有时动态修改、测试时，需要在 VMware 中盲打。   
  
  为了进行基于网络的远程调试，不要求操作方启动 SoftICE，siremote.exe 只作为   
  普通网络客户端软件出现。   
  
5) 可以在 VMware 中设置如下注册表项:   
  
  HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\NTice   
  
  NullVGA REG\_DWORD 1(缺省为 0)   
  
  其本意是，如果远程呼叫 SoftICE，在被调试端看不到 SoftICE 窗口，否则可以看   
  到。由于 VMware 中显卡驱动的缘故，有无这个设置都一样效果，我还是设置上了。   
  
6) 至第 5 步为止，结合 VMware 的远程调试已经搞定。可 VMware 中的 XP 也丧失了正常的   
  网络通信功能，我们需要远程调试 SMB 协议处理时，就完蛋了。此时有两种选择，   
  第一种是通过添加新硬件安装如下虚拟设备:   
  
  \\Program Files\\Compuware\\DriverStudio\\SoftICE\\Network\\UND\\VNIC\\sivnic.inf   
  
  在重启过程中如果因 VNIC 出现故障，当 UND 驱动提示时，可按 ESC，这将禁止加载   
  UND、VNIC。很可怜，我还没等到重启，只是在安装过程中就出现 BSOD 了，连续试   
  了几次都如此，不清楚是使用 VMware 的缘故，还是 sivnic.inf 本身的缘故，我没   
  有试真实远程主机的情形。   
  
  理论上，如果此时安装了 VNIC，就可以看到一块网卡，然后可进行 TCP/IP 协议配   
  置等等。但是网络性能很低，这与是否使用 VMware 无关，即便真实的远程机器也   
  如此。   
  
  另一种办法是真实添加一块物理网卡。对于 VMware 来说，需要关闭当前 XP，关闭   
  电源，回到初始界面，然后在配置中增加一块物理网卡。对于真实的远程主机，   
  就需要打开机箱插入新网卡了。注意，安装 UND 时会导致以前所有配置"丢失"，包   
  括物理网卡，因此一些操作顺序尤其重要。VMware 以前有两块物理网卡，安装 UND   
  后丢失，不太影响什么，可以继续增加物理网卡。真实远程主机就不同了，不大   
  可能有第三个插槽给你插第三块物理网卡，现在明白步骤 1 中所说了吧。   
  
  现在在网络属性里可以看到惟一一块网卡了，就是新增加上来的那块，然后进行   
  正常的 TCP/IP 协议配置，不要使用 192.168.7.153，这是 SoftICE 使用的 IP 地址，   
  比如可以使用 192.168.7.152。有些网络设置是全局设置，比如 TCP/IP 筛选，如果   
  修改，可能会影响包括"那些丢失的网卡"在内的所有网卡，结果未知，所以不建   
  议修改这些全局设置。此外还有一处古怪，即使没有选择"自动获得 IP 地址"，而   
  是配置静态固定 IP，重启后在网络属性里看到的还是"自动获得 IP 地址"，不过这   
  是假像，最好在 CMD 中执行"ipconfig /all"命令，可以看到:   
  
  Dhcp Enabled. . . . . . . . . . . : No   
  IP Address. . . . . . . . . . . . : 192.168.7.152   
  Subnet Mask . . . . . . . . . . . : 255.255.255.0   
  Default Gateway . . . . . . . . . : 192.168.7.254   
  
  在注册表中也只看到一个网络接口，并且配置了静态固定 IP:   
  
  HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\Tcpip\\Parameters\\Interfaces   
  
  虽然"自动获得 IP 地址"是假像，但你要修改什么的话，还是需要从头走一遍，除   
  非你直接去注册表中修改。无论如何，都需要重启，而本来 XP 下修改这些东西不   
  需要重启的。   
  
  现在我们的 VMware 即具有正常的未降低性能(与 VNIC 相比)的网络功能，又支持基   
  于网络的远程调试功能，爽。   
  
7) UND、VNIC 的卸载顺序很重要，一定要先通过删除设备卸载 VNIC，再执行   
  UNDSetup.exe 卸载 UND，和安装顺序正好相反，不要混了。   
  
  如果安装 VNIC 过程中出现 BSOD，也去设备管理器中查看一下，删除半安装状态的   
  VNIC，以免影响 UND。   
  
  从上述文字看出，有两块物理网卡的真实远程主机无论如何都可以支持基于网络   
  的远程调试。影响性能的方式只有一种，就是 VNIC，其它两种方式都不影响性能。   
  不再局限于 3COM/3C90X、Intel/E100、Novell/NE2000。   
  
  如果使用 VMware，更方便。   
  
8) 在设备管理器中，System devices->SoftICE network transport，该设备占用了   
  中断请求 15。我的测试环境中，Secondary IDE Channel 也占用中断请求 15，前者   
  会抢占成功，于是后者出现黄色问号，无法启动。如果 VMware 的光驱的接在   
  IDE 1:0 上，就无法使用光驱，此时可以将光驱换接在 IDE 0:1 上，即 Primary IDE   
  Channel 的从盘，与主盘一起使用中断请求 14，光驱恢复正常。   
\--------------------------------------------------------------------------   
  
上述就是 VMware+SoftICE 完美解决方案的配置过程，中间省略了一些与远程调试无关   
的配置步骤。  4dd466d0e1e0cc123afbe504da2462ca