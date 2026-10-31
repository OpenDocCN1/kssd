
# [原创]重载内核 完爆SSDT hook

说明这是暑假期间弄得一个小东西。对驱动了解甚少。但是觉得SSDT HOOK比较容易破。于是自己也试了一下重载内核，注入我的DLL，通过我自己的函数进入内核完成功能。

附件内三个文件，一个说明以及测试文档。

加附件有驱动，加载驱动有风险，请虚拟机调试。否则。造成的损失小菜不负责。

一切的调试信息都用`dbgview`查看，对于被注入的软件 例如`CE`，`OD`要求`ntdll.dll`内的`nt`系列函数没有被HOOK。因为我是通过`ntdll.dll`内的值取得`ssdt`服务号的

还缺少的就是你电脑的一个内核驱动文件。也就是`ntxxxx.exe`系列的一个驱动。没有放进来是为了保证能在其他电脑尽量不会蓝屏。所以为了能使用，请先将自己电脑内的这个驱动复制一份重命名为`kernel.dll`然后和附件的三个文件放在一起

`HKKernelDbg.dll`

负责加载本地的内核（`kernel.dll`）作为COPY到内核的数据。以及R3 HOOK `nt`系列函数

`HKKernelDbg.ini`

配置 是否要hook的 `nt`函数（不能自己添加，我只写了一些常用的函数）

`HKKernelDbg.sys`

初始化数据，包括从R3复制一个内核的副本

内核副本数据的重定位。

以及内核函数的调用。与R3通信

使用注入器注入`hkkerneldbg.dll`到`OD` 或者`CE`以后，这个DLL先加载`kernel.dll`也就是把内核驱动当DLL加载，然后加载`sys`，然后`DeviceIoControl`发送信息通知`sys`复制副本到内核，进行数据重定位。最后 在R3 HOOK `ini`配置里的API。

接下来就是如何调用这些函数了。HOOK的R3函数会把内核中对应函数的偏移，以及函数参数，参数个数通过同一个控制码用

`DeviceIoControl`和`sys`通信，并把执行结果返回给R3。我并没有像其他人那样修改`Kifastcallentry`，也就是我并没有修改真正内核的一个字节，只是建立了一个自己的通道去内核执行函数。

后来虚拟机测试了用`CE`修改受保护的某游戏。修改数据成功了。呵呵

DLL 以及SYS 源码 在`HKDBG.ZIP` 上传了。写的比较烂。。各位莫笑。。毕竟还是菜鸟

`VS2010+DDK`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)

上传的附件：

- [hkkerneldbg.zip](javascript:void(0);)

  （550.20kb，849次下载）

- [HKDBG.zip](javascript:void(0);)

  （36.73kb，896次下载）
