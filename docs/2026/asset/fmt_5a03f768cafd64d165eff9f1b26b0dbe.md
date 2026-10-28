
程序首先创建了一个`vlog.log`日志文件，接着调用`SetFilePointer`、`GetForegroundWindow`和`GetWindowTextA`函数获取按键按下时活动的窗口，然后获得窗口的标题，接着将其写入文件中。

![](https://bbs.kanxue.com/upload/attach/202011/830337_EKHMYDEUM8VST84.png)

![](https://bbs.kanxue.com/upload/attach/202011/830337_9QKQ8AKNX2MBFZ7.png)

接下来是一个跳转表。

![](https://bbs.kanxue.com/upload/attach/202011/830337_6FT5EGWAFJN9JGN.png)

`var_C`包含接收的虚拟按键码，该按键码作为`switch`表的索引，该表位于`off_401441`处，见图中光标处；`byte_40148d`处是一个数组。

![](https://bbs.kanxue.com/upload/attach/202011/830337_FV94BBQXHKNRTA2.png)

在`401251`处我们看到`[SHIFT]`被写入文件，下面是一系列的按键。

![](https://bbs.kanxue.com/upload/attach/202011/830337_ZS7PRG9GASK9WH4.png)

`[BACKSPACE]`、`[TAB]`、`[CTRL]`，下面都是一系列的按键。

![](https://bbs.kanxue.com/upload/attach/202011/830337_2GFJZKYF22M468D.png)

![](https://bbs.kanxue.com/upload/attach/202011/830337_X2E2B4CWM6JVMRA.png)

这里已确认资源节解密出的可执行文件的功能就是一个键盘记录器，记录到创建的`vlog`文件中。

# 总结

这个病毒骚的地方不仅是资源节隐藏加密的可执行文件，这样就导致很难识别，`Krypto ANALyzer`插件也识别不出加密算法，最后只能在`IDA`中找解密例程。

最骚的还是R3下隐藏进程的手法。动态分析工具识别不到进程名的PID，只能看到正常运行的进程。该病毒利用的是`svchost.exe`，关键技术是利用`CreateProcessA`创建挂起的线程，`EBX`寄存器指向`PEB`，`EAX`指向`EIP`。因为大部分人知道，如果线程没被挂起，`FS`指向`TEB`，然后加上`0x30h`偏移获取`PEB`的指针。这里把`svchost.exe`掏空了，把恶意可执行文件射入正常进程的地址空间，好一个偷梁换柱、借窝生蛋。最牛逼的是用各种工具都很难发现。这个R3下的隐身技术就很叼，别具一格，很有创意，艺术啊。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
