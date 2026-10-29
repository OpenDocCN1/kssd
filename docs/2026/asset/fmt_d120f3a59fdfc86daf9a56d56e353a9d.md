
# [原创]《0day安全...（第二版）》第3章第4节开发通用的`shellcode`在`win10`系统下测试的问题

《0day安全软件漏洞分析技术（第二版）》第3章第4节开发通用的`shellcode`在`win10`系统下测试的问题

最近耐心的重学了一遍《汇编语言第二版》，为研究`0day`安全这本书打下一点基础，然后静心下来时隔几年开始重新学习。时过境迁，实验平台、实验环境、实验程序、开发平台都变化了。

使用`VS2010`编译`P97`的`shellcode`装载程序，生成控制台程序，并且配置关闭数据执行保护（`DEP`）。直接运行测试，可以看见对话框，但是退出时候崩溃了。这不是健康的`shellcode`，肯定是出问题了，具体是什么问题？

基本确定是`win10`系统和`shellcode`不兼容，即这个`shellcode`在`win10`下，不再是通用的了。现在开始动手找问题的原因所在。

首先，用`ollydbg`加载运行，停在图1界面：

![](https://bbs.kanxue.com/upload/attach/201808/238983_6XEFQWK8QRYH4Y7.jpg)

图1 `ollydbg`加载`shellcode`装载程序第一步

在图1中可以直接找到`xxx.main`的入口地址，`F2`下断点，`F9`执行到入口地址处，`F8`进入装载程序代码空间，如图2所示。

![](https://bbs.kanxue.com/upload/attach/201808/238983_6GFYK5J9QNVCGKB.jpg)

图2 装载程序代码区

一路`F8`直到`retn`后，跳入`shellcode`代码区，如图3所示。

![](https://bbs.kanxue.com/upload/attach/201808/238983_NCRBNN5JV35MPPX.jpg)

图3 `shellcode`代码

这里的代码与书本`P93`的代码就是相同的了。此后`F8`单步跟踪观察，三次`push`，把要搜索的`API`函数`HASH`值推入栈，如图4所示。

![](https://bbs.kanxue.com/upload/attach/201808/238983_EYJAGGTZQV497BU.jpg)

图4 入栈3个`HASH`值

因此先搜索的是最后入栈的`0x0C917432`，也就是`LoadLibraryA`函数。看图3中的第7行代码`lea edi,[esi-0xc]`，根据书本注释也能知道，就是图4堆栈位置再往上3个`DWORD`空间，用来保存三个`API`函数的入口地址的，`edi`指向这3个`DWORD`，因此后面只需要`call [edi-0x8]`、`call [edi-0x4]`即可调用相应的`API`函数了。当然要注意观察，`edi`值在`shellcode`运行过程中，并不是保持不变的，可以体会确保`call`调用时，`edi`指向正确的`API`地址位置。

在`ollydbg`地址观察窗显示`[edi]`地址，如图5所示。

![](https://bbs.kanxue.com/upload/attach/201808/238983_C9K64VSVH3NU655.jpg)

图5 `Address`观察窗口

对比图4和图5，依据代码，可以知道`esi`指向栈顶第一个`API`函数的`HASH`，`edi`指向比`HASH`值低3个`DWORD`的位置（目前这3个位置中的数据是无效数据），准备保存`API`函数入口地址。因此可以直接下内存读的断点，点击选中图5窗口的第一行地址，右击鼠标，在弹出菜单中选择`Break point -> Memory,on write`。

下好断点，按下`F9`，程序会先在`kernel32.dll`中根据`HASH`值搜索`LoadLibraryA`的函数名，找到之后确定函数入口地址，然后往`[edi]`的地址中写入入口地址，一旦写入触发我们的断点，就会停下，如图6所示。

![](https://bbs.kanxue.com/upload/attach/201808/238983_U579PXD7JN3A62D.jpg)

图6 函数`LoadLibraryA`的入口地址保存代码

中断位置发生在书本`P94`倒数第二行代码，`stosd`这句，也就是图6中被中断的位置。`F8`运行这句代码，立刻可以在`address`观察窗口看到果然记录下了第一个`API`入口地址，如图7所示。

![](https://bbs.kanxue.com/upload/attach/201808/238983_8Y8K673NWXFUEYZ.jpg)

图7 第一个`API`函数入口地址自动确定

对比图5和图7的区别，可以看到，这里果然保存了`LoadLibraryA`的地址了。

继续对后面两个`0x0113FB1C`和`0x0113FB20`下内存写断点，继续观察`API`函数入口地址是否正确，因为`shellcode`在退出时崩溃了。初步判断，就是`ExitProcess`函数入口地址应该是搜索出现问题了。这个问题是怎么出现的，是这篇文章的重点。

下好断点，按下`F9`，再次中断，这时候可以看到中断的代码位置还是相同的。按一下`F8`，发现问题了，本该保存`ExitProcess`入口地址的位置，变成保存的是一串字符串了，如图8所示。

![](https://bbs.kanxue.com/upload/attach/201808/238983_WBM45PVRTUUPKSQ.jpg)

图8 `ExitProcess`入口地址保存错误

很明显，这里保存的是`ASCII`字符串。看情况判断，`win10`可能是取消了`kernel32`中的`ExitProcess`函数了。因为系统的这个改变，所以`shellcode`运行到这里`call`的不是入口地址，而是一串字符串，肯定是崩溃了。崩溃的原因找到了。要解决的话，就是要改写`shellcode`，在`NTDLL`中搜索新的`ExitProcess`函数了，这个就是以后能力提高后再考虑的事情了。这次实验目的查找崩溃原因的目标顺利完成。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2018-8-29 11:28 被`fjrdwork`编辑，原因：
