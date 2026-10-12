
# 病毒分析报告

> 原文：[`www.pediy.com/kssd/pediy12/99991.html`](https://www.pediy.com/kssd/pediy12/99991.html)

学习病毒分析,目前自己只达到了这个水平，不知道能不能拿个邀请码..这个病毒的样本，加了注释的 `IDB` 文件,释放的 `DLL`,我打了一个包放了出来..释放的 `DLL` 不知道怎么回事,哪位大神可以给指点一下.

## 一、病毒标签

病毒名称：

病毒类型：`木马`

文件 `MD5`：`5CA7344508EC2EEF0A84E35150AC6FD25FC02C1C`

文件长度：脱壳前 276,480 字节，脱壳后 376,832 字节

受影响系统：

* `Microsoft Windows NT 4.0`
* `Microsoft Windows NT 4.0 Terminal Services Edition`
* `Microsoft Windows 2000`
* `Microsoft Windows XP`
* `Microsoft Windows Server 2003`

开发工具：未知

加壳类型：未知

## 二、病毒描述

修改注册表，自启动，注入 `explorer.exe` 执行远程代码。

## 三、行为分析

病毒首先判断操作系统版本，以及 `explorer.exe` 的大小和修改时间，并且创建事件对象 `"VBHSDDXCASDFERRBNM_BOB"` 来保证系统中只有一个实例在运行。

然后将自身文件复制到系统目录 `system32` 下，并释放 `cao220.txt` 与 `cao110.dll` 文件，设置隐藏属性。启动进程 `regsvr32.exe` 参数为 `/s cao220.txt`,安静模式执行命令。查找 `"AlertDialog"` 和 `"Product_Notification"` 窗口，模拟用户点击退出。过瑞星的实时监控，找到 `RavMon.exe` 进程，后枚举其中控制监视窗口的那个线程，模拟用户点击，关闭实时监控。填加注册表启动项 `SoftWare\Microsoft\Windows\CurrentVersion\Run`，键值为 `c:/windows/system32/zhido.exe`。向 `explorer.exe` 中注入病毒代码，并远程执行。如果写入失败，则加载起 `cao110.dll`，之后则进入消息循环中。

释放的 `DLL` 中，没有看到具体行为。

## 四、清除方案

1. 删除释放文件
2. 删除注册表 `SoftWare\Microsoft\Windows\CurrentVersion\Run` 中对应键值。

### 上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 1.rar |
| :--- | :--- |
