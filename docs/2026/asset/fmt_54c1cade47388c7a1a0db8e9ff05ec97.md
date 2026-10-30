
![](https://bbs.kanxue.com/upload/attach/202009/762319_TEBXXB7EFTWKUT4.png)

此时触发异常之后，调用异常处理函数为`0x0C0C0C0C`处的数据如下所示：

![](https://bbs.kanxue.com/upload/attach/202009/762319_3ECH24PVD8G3WGY.png)

是一大段的`nop`，POC表现形式为弹出一个对话框，所以此时可以在`MessageBoxA`下断点，分析`shellcode`，可以先忽略程序异常，设置如下：

![](https://bbs.kanxue.com/upload/attach/202009/762319_FKPVYAUF7B8C6D6.png)

然后按`F9`运行程序，如下所示：

![](https://bbs.kanxue.com/upload/attach/202009/762319_C25YZG8HXXKPV28.png)

查看`0x0C100008`处代码：

![](https://bbs.kanxue.com/upload/attach/202009/762319_BZRHYKS945NXKE9.png)

上一条指令为`call [edi-4]`，可以看到左下角数据区为此时调用的函数为`MessageBoxA`，然后调用`call [edi-8]`是`ExitProcess`函数退出程序。`shellcode`如下：

![](https://bbs.kanxue.com/upload/attach/202009/762319_3KAH2VXMKD559UB.png)

![](https://bbs.kanxue.com/upload/attach/202009/762319_8XYU9TR85XF642W.png)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班，第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-9-15 23:16 被kanxue编辑，原因：

上传的附件：

- [poc.pdf](javascript:void(0);)

  （1.80kb，27次下载）
