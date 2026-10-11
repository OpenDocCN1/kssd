
# 进程保护的逆向，附源码

> 原文：[`www.pediy.com/kssd/pediy12/141902.html`](https://www.pediy.com/kssd/pediy12/141902.html)

驱动是来自 `MJ0011` 在 08 年放出来的驱动，无聊的时候逆了一下。

`http://www.debugman.com/thread/1122/1/1`

引用: 看到有人诟病 360 的自我保护，又见江民、微点等一堆国内挫人使用一大堆 `HOOK` 对自己的进程做保护

导致系统不稳定还保护不住

于是放点东西出来给大家玩玩，欢迎挑战

测试程序没有使用任何 `HOOK` 技术，没有使用任何 `DKOM` 技术，完全使用微软标准函数、标准接口，却让无数号称强大的 `ARK`、结束进程工具望之兴叹！

以下是无法结束本测试程序的程序列表，欢迎各位测试自己的工具，逐步更新此表

`Icesword` 1.22 --- failed

`RKU` 3.7 --- failed

`gmer` 1.14 --- failed

`SnipeSword` 20080225 --- failed

`Wsyscheck` 20080223 --- failed

墨者安全专家进程管理 3.05 --- failed

`DarkSpy` 1.0.5 --- failed

`SysProt` 1.0.0.5 --- failed

### 挑战规则

让进程退出

### 禁止

- 不允许攻击窗口
- 不允许恢复驱动使用标准函数建立的接口等
- 不允许破坏驱动内部数据或代码流程（例如保护 `PID` 值）
- 不允许类似上面的技巧

测试程序见压缩包:

### 测试方法

使用 `Driver Monitor` 或 `DrvLoad` 等工具加载 `ppxx.sys`

运行 `ppxx.exe`

其实我很菜，练手而已。

### 上传的附件

| 附件 | 说明 |
| :--- | :--- |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `ida_ppxx.rar` |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `src_ppxx.rar` |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `原东西_ppxx.rar` |
