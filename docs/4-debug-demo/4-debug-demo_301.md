
# 超级解霸 2000

> 原文：`www.pediy.com/kssd/pediy03/forum542.htm`

`超级解霸 2000`试用版的使用限制解除：

梁肇新的`超级解霸 2000`试用版只能使用 100 次，他将安装信息存放在两个地方：

1. 注册表中：
   `HKEY-LOCAL-MACHINE\SOFTWARE\MICROSOFT\WINDOWS\CURRENTVERSION\SETUP`
   `RUNTIME` 的值

2. `ini` 文件中：
   `C:\WINDOWS\STHVCD.INI`
   `SETTING`
   `BERUN`

上次写的还是有点问题，我自己在使用的过程中发现：上面两处只是校验使用次数的，程序在别的地方还有校验时间的代码。不过，如果你把我上篇文章里面提到的两个地方的值恢复到安装时的状态的话，如果还不能使用，则请把系统时间往前调一点应该就可以使用了（我自己一直在用这个试用版，按这种方法改没问题的）。如果嫌每次用的时候改时间麻烦，可以用`AnyDay`或者类似软件！

`4dd466d0e1e0cc123afbe504da2462ca`
