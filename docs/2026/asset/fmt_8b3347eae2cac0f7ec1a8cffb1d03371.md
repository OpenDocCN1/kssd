
# [原创]发一个完全禁止 ring3 访问网络的代码

在 `ring3` 调用的那些 `WSASend`、`WSARecv` 等 socket 的函数，他们最终都是调用 `ZwDeviceIoControlFile` 来和驱动程序通讯的，所以只要 hook 了这个函数，判断一下句柄的类型，如果和网络有关的就直接返回，这样 `ring3` 的程序就不能访问网络了，本例子中还 hook 了 `ZwCreateFile` `ZwOpenFile` 等函数，因为之前只是想禁止打开这些对象，但比如 qq 等程序会一直打开着 socket，这样只 hook `ZwCreateFile`、`ZwOpenFile` 就不能完全的禁止 qq 这类程序访问网络了，所以把 `ZwDeviceIoControlFile` 都 hook 了

![](https://bbs.kanxue.com/view/img/face/16.gif)

代码写的很烂，可以在 `windows 7` 运行
