
# [原创]ShellCode编写之hash式函数调用及相关

前段时间的Exploit Me挑战赛，使我深刻的感到ShellCode编写完全不同与病毒，对于栈空间得按字节来省着用，ShellCode中hash式函数调用势在必行。另外，在unicode环境下的shell处理，也得过关才行。参照《The Shellcoder's Handbook》 中关于“百叶窗”法，及相关网文资料，自己也写了拆分ShellCode字符串模块，在此一并放出![](https://bbs.kanxue.com/view/img/face/50.gif)

`CreateWinAPINameHash`功能是获取指定模块中函数名称串的HASH值，并返回Hash的Key值，要计算实际中函数名称串的HASH值只需要调用次过程就可以；`ShellCodeToStdByteChar`功能是对于已经写好的ShellCode，将其中的非纯数字和字母字节进行拆分，使之转换为字母字节。另外，`GetBaseAddressOfKernel32ByTEB`与`GetKernel32BaseAddressBySearch`是两种不同方法获取`Kernel32.dll`的基址的子过程，以及`GetStringSize`用于获取字符串大小的子程序，是顺便送给大家公用子程序，省的自己写![](https://bbs.kanxue.com/view/img/face/12.gif)
