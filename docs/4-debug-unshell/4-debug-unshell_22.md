
# 脱壳小技巧

> 原文：[`www.pediy.com/kssd/pediy04/kanxue493.htm`](https://www.pediy.com/kssd/pediy04/kanxue493.htm)

有的加壳软件会检查你有没有在一些常用中断上设下断点，比如：`loadlibrarya`, `getprocaddress`, etc. 你可能在跟踪时发现这样的语句。特征是设下断点后程序会崩溃或退出。

解决起来很简单，这样：`bpx loadlibrarya+1`, `bpx getprocaddress+1`. 注意要保证在语句上，不要设到语句中，否则 `4dd466d0e1e0cc123afbe504da2462ca`
