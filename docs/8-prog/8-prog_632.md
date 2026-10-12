
# 去除驱动安装数字签名的终极解决方案------适用 win2000, winxp, win2003

> 原文：[`www.pediy.com/kssd/pediy10/81521.html`](https://www.pediy.com/kssd/pediy10/81521.html)

主要有两种方法：

一、向"签署驱动程序"对话框发送模拟消息，使之置于"忽略"位置。

本代码修正了 LIUTANG 大牛代码"2003-01-28_ 如何去掉 xp 下的数字签名.rar"

1. `uDriverSigningButtonID` 在 XP、2003 中与 2000 中并不一致
2. 上述代码使用 MFC 库，造成执行文件过大的问题，修正代码中消息模拟全部用 Windows API 实现

二、修改注册表的方法

1. windows 2000

   - `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Driver Signing\Policy` （键值改为 0）
   - `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Non-Driver Signing\Policy` （键值改为 0）
   - `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Driver Signing\Policy` （键值改为 0）

2. windows XP 及 Windows 2003

   除修改上述三个键值外，还需要修改 `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\PrivateHash`，而该值是由种子值 `HKLM\SYSTEM\WPA\PnP\seed` 哈希散列得来，每个机器均不同；如果该值未改或改得不对，windows 的注册表监控机制会将上述修改恢复，造成失败。

鸣谢：[http://blogmal.42.org/rev-eng/](http://blogmal.42.org/rev-eng/) LIUTANG 大牛等

其他方法：

1. 同上述一，只是钩子钩住 "继续安装"那个对话框
2. 有人说，可以修改 `sigtab.dll` 中的某个函数，经 windbg 调试，这种方法可能有效，未试验。

源代码测试通过 2000, xp, 2003！

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `RemoveDriverSign.rar` |
| :--- | :--- |
