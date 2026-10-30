
# [原创]Hyper-V安全从0到1(5)

## 5. Cve-2016-0088的分析，触发，调试

本节介绍的是由Google project zero发现的Hyper-V漏洞，这枚漏洞于2016年被发现，是`vmswitch.sys`组件产生的问题，可以导致宿主机内核崩溃，造成宿主机DOS。这个漏洞是在Windows Server2012 R2版本的操作系统版本上触发的，所以下面的介绍使用的Windows版本为：Microsoft Windows Server 2012 R2 Standard x64且未安装任何补丁。

### 5.1. 漏洞分析

这个漏洞发生在`vmswitch.sys`，用于Hyper-V的虚拟网络设备服务。在`vmswitch!VmsMpCommonPvtHandleMulticastOids`函数中，有一处越界写内存的问题，可以造成相邻堆块的5字节数据的写入，破坏了下个堆块的`POOL_HEADER`，在free下个堆块时内核便会抛出异常，导致宿主机蓝屏。

下面我们通过IDA逆向分析`vmswitch!VmsMpCommonPvtHandleMulticastOids`函数，代码如下。
