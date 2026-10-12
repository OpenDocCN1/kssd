
# FlaxFX v1.31

> 原文：[`www.pediy.com/kssd/pediy04/kanxue064.htm`](https://www.pediy.com/kssd/pediy04/kanxue064.htm)

`RegKey: ZLVKVI-The_Lamer-WEKHCDCWFDPHHM444M324KU`

`FlaxFX` 安装在 C 盘。

断点设在 `GetDiskFreeSpace` 上就可以了。

像 `flaxfx` 这种东西绕来绕去实在讨厌，但是分析出一个注册码的格式应该不难。

输入一个格式正确的注册号之后，在注册对话框里面按 `ok` 之前先运行某个能监控进程 `api` 调用的程序，比如 `apimon` 之类，看看它都干了什么。很容易发现它要取 `flaxfx` 所在磁盘的信息，这个大大可疑。赶紧把断点设上。之后就没什么难度了。

`4dd466d0e1e0cc123afbe504da2462ca`
