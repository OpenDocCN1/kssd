
# [原创]某南极动物厂新版XX分析——系统线程部分（新瓶旧酒）

这里分析的是6月18号的版本，由于某些原因这里隐去敏感内容

目前静态分析跟踪到7个系统线程创建，线程创建过程如下：

先判断`hv`是否存在

![](https://bbs.kanxue.com/upload/attach/202006/619065_SSWNEETESAYGPGR.jpg)

如果`hv`不存在则搜索`SeSetAuditParameter`中的`FF E1`特征码

![](https://bbs.kanxue.com/upload/attach/202006/619065_YVWVPZ9PJZYZJC2.jpg)

根据如果成功搜索到上述`fake thread entry`则把线程入口指向上述`entry`，线程参数指向原真实线程入口（也就是`rcx`=真实线程入口，并且`jmp rcx`）

然后调用`pscreatesysthrd`创建系统线程

![](https://bbs.kanxue.com/upload/attach/202006/619065_YVFPBJ6MTCFTQHU.jpg)

目前分析到的7个线程分别是：

`CheckKSharedUserDataKdDebuggerEnabledThread`

`AntiKdDebuggerThread`

`CheckGameProcessDebuggedThread`

`CheckKeTimeAcceleratedThread`

`GameCommonCheckThread`

`sub_xxxxxxxx`（作用不明，可能代码被注释或功能暂未实装）

## CheckKSharedUserDataKdDebuggerEnabledThread：

判断`KSHARED_USER_DATA->KdDebuggerEnabled` & 1或 & 2 ?

每5秒检测一次，如果判断为`true`则蓝屏

![](https://bbs.kanxue.com/upload/attach/202006/619065_SN25395STF4J9WP.jpg)

## AntiKdDebuggerThread：

如果系统版本大于等于`win7`则执行`int2D`指令触发`SEH`（由于静态分析无法分析`SEH`所以`exception`之后不知道发生了什么）

否则调用`KdDisableDebugger`

每5秒执行一次上述功能

## CheckGameProcessDebuggedThread：

`ExecuteForGameProcess(AntiProcessDebug)`;对所有受XX保护的`GameProcess`执行`AntiProcessDebug`回调

![](https://bbs.kanxue.com/upload/attach/202006/619065_MFG7YFNQEWCD6A3.jpg)

![](https://bbs.kanxue.com/upload/attach/202006/619065_DXE3RPZDYBGYU9F.jpg)

![](https://bbs.kanxue.com/upload/attach/202006/619065_7VEXJDHGENP95GZ.jpg)

如果`EPROCESS.DebugPort != 0`则触发蓝屏

上述检测每10秒执行一次

## CheckKeTimeAcceleratedThread：

检测系统时间是否被加速，导致`KeDelayExecutionThread`的等待时间结果跟`HalQueryRealTimeClock`时间差不一致

![](https://bbs.kanxue.com/upload/attach/202006/619065_2USQGU84NBEXS2E.jpg)

如果时间不一致：

如果某开关开启，则上报服务器，否则蓝屏

上述检测每1秒执行一次

## GameCommonCheckThread：

![](https://bbs.kanxue.com/upload/attach/202006/619065_94TSECPKAQ7KUS6.jpg)

检测`cr4.tsd` `bit`是否为1

如果为1：

如果某开关开启，则上报服务器，否则强制结束游戏（对`g_game_process`数组里的所有进程`ZwTerminateProcess`）

除此之外还对所有游戏进程检测`SHARED_USER_DATA`

`ExecuteForGameProcess(CheckSharedUserData)`;

检测如下：

![](https://bbs.kanxue.com/upload/attach/202006/619065_BNA6QAS67UPV45W.jpg)

判断`R3`和`R0`的`KSHARED_USER_DATA`的物理地址是否一致

如果物理地址不一致：

如果某开关开启，则上报服务器，否则强制结束游戏

上述检测每10秒执行一次

## AllProcessCommonCheckThread：

对`PID`为16~0x40000的所有进程枚举句柄表

![](https://bbs.kanxue.com/upload/attach/202006/619065_9NDW4SBCBRHS8PA.jpg)

![](https://bbs.kanxue.com/upload/attach/202006/619065_FY67AAU4TXC3D8Q.jpg)

如果句柄`ObjectType==*PsProcessType`且 拥有该句柄的进程不是受保护的游戏进程 且 进程句柄指向的进程是受保护的游戏进程

则进行句柄降权

遍历`PID`为16~0x40000的所有进程，遍历其内部的从8~0x2000的所有句柄，如果是`SECTION`类型且名称包含

`L"\\TX_SSO_SHARE_INFO_SIZE"`

`L"\\TENPROTECT3_SHARE_DATA_"`

`L"\\TENPROTECT6_SHARE_DATA_"`

则

如果该进程是受保护的游戏进程，则什么都不做

如果该进程不是受保护的游戏进程，且该进程仍然存活（未退出），则上报服务器

如果上述条件都不满足，则强制蓝屏

（猜测：此举可能是为了防止有人强制清空受保护的游戏数组`g_game_process`）

上述检测每10秒执行一次

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
