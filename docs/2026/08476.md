
# [原创]常见反调试手段及其规避方法（二）-检测进程名

所谓检测进程名，指软件会获取所有当前正在运行的进程的名字，初步筛选出可疑进程，接着配合其他手段确认可疑进程是否真的是调试器的进程，如果是就将它关闭。

调试器：`OllyDbg`

环境：`win7 64位真机`

首先配置`OD`：在主界面点击选项-调试选项-异常

![](https://bbs.kanxue.com/upload/attach/201805/803510_YBCEQAQM3Z9DMS6.jpg)

忽略所有异常，如图示。

来到“安全”选项卡-勾上：前三个和允许单步步入到未知命令，点确定。如下图

![](https://bbs.kanxue.com/upload/attach/201805/803510_DZUVW8QBWZPW65E.jpg)

载入后，`ctrl+n`打开函数列表，发现很多很多函数，我们找到`GetProcAddress`，使用`MSDN`搜索，可知是获取另一个函数的地址。如果函数列表里没有可以的函数，说明他被隐藏，就是通过这个函数获取隐藏函数的地址的。

![](https://bbs.kanxue.com/upload/attach/201805/803510_KAVR8AYTA9SXSM6.jpg)

从`MSDN`上查阅函数的功能。百度百科上的内容可能不太准确。

翻译为：“从特定的动态链接库文件（`DLL`）中抽取出输出函数或者变量的地址”

什么意思呢？比如，函数列表里没有`MessageBoxA`，使用`bp MessageBoxA`是无法下段的。因为系统找不到。除非你用`GetProcAddress`获取这个函数的地址才能下断。

我们在`GetProcAddress`段首下断，观察堆栈窗口，`hModule=0400000`，`ProcNameOr="_CPPdebugHook`

![](https://bbs.kanxue.com/upload/attach/201805/803510_X9BN5BWHSDXQV5V.jpg)

这个是该函数的基址以及该函数的名字，当`ctrl+f9`后才能获取真正的地址。地址会返回给`eax`，注意`95%`的函数都是返回给`eax`。（还记得前面讲到的条件记录断点吗？这里留一个小作业给大家：用条件记录断点记录`GetProcAddress`函数的`ProcNameOriginal`参数）

知道函数的名字，还知道了他的首地址，就能在我们感兴趣的隐藏函数那下断了 。

一直`f9`，在堆栈窗口找可疑函数，如果你不知道是啥函数，就用`MSDN`查找。这是每一个初学者必须经过的过程。

`F9`，直到`EnumProcess`获取当前进程列表里边所有的进程的`PID`（如下图），就是任务管理器里所有进程的`PID`，`ctrl+f9`后直接`bp eax`，因为该地址在`eax`里面。这时记住要到`B`窗口里注释一下函数名称，不然以后肯定忘。

![](https://bbs.kanxue.com/upload/attach/201805/803510_84DMASYFMJB8XTX.jpg)

`PID`就是系统分配给当前每一个正在运行的进程的编号或者叫标识符。只要这个进程还存在，不被关闭，那么它的`PID`就不会变。当他再次打开以后，会获得一个新的编号，新的`PID`，虽然程序还是那个程序。任务管理器里的`PID`里是十进制数，`2668`（抱歉这里图上没有显示出来），那么是十六进制.`0xA6C`

我们还对`EnumPeocessMOdules`感兴趣，获得指定进程中所有模块的句柄。也下段，加注释。

我们对`GetModuleBaseName`感兴趣，通过进程句柄，进而获取进程名字，也下段，也加注释。

然后会执行到`EnumProcess`这里，断在段首时，`ESP`指向的一定是返回地址，`ESP+4`一定是函数的第一参数，这里的`ESP+4`是一个堆栈地址，我们在数据窗口跟随这个堆栈地址，这里就是存放进程列表的`PID`的，以这个地址为开头存储，每四字节存储一个`PID`。我们找到到 `6C 0A 00 00`，就是`A6C`，这里是`PID`为`2668`的`OD`调试器的`PID`。然后就选中这四字节，设置一个内存访问断点，既然我们获取了这个`PID`，我看看谁在获取这个值，接下来又执行了什么。

稍等一会儿，不要连续按，让子弹飞一会。

来到`OpenProcess`，我们来观察堆栈：

`ProcessID=A6C`     显然程序动了这块数据，把它提取出来压入堆栈了

`Inheritable=FALSE`

`Access：VM_READ|QUERY_INFORMATION`

来看看`OpenProecess`能干嘛：

![](https://bbs.kanxue.com/upload/attach/201805/803510_VD6CUKHE4MYFWW9.jpg)

翻译：“打开一个已经存在的本地进程”。

如果一个程序获取了另一个程序的句柄，那他就可以操作另一个程序，比如关闭。比如外挂可以操作游戏。

这个函数的作用是获取当前`PID`的句柄，会把句柄返回到`EAX`中，请看`MSDN`中关于该函数返回值的陈述：

![](https://bbs.kanxue.com/upload/attach/201805/803510_PWKQWVGSBVAFAV2.jpg)

翻译为，“如果函数成功运行，返回值就是一个特定进程的句柄，如果函数执行失败，返回值为空。使用`GetLastError`函数获取扩展错误信息”

执行完后`EAX`是`88`，注意是十六进制数。

`F9`,执行`EnumProcessModules`，这个函数的作用是通过句柄，来获取这个进程的基址。断在在段首时我们看到`ESP`的下一行，就是第一参数，是`00000088`，`0x88`正是`OD`的句柄。

`ctrl+f9`，看堆栈，第一行是返回地址，第二行是我们的句柄`88`，这是`OD`的句柄。第三行是一个堆栈地址，这个地址在数据窗口跟随后，就是该函数的基址了。

然后`F9`，就是`GetModuleBaseNameA`，这个就是通过句柄来获取对应进程的名字了。就是任务管理器里的映像名称，第一列。再执行这个函数，`ctrl+f9`后，堆栈第一行是返回地址，第二行是`88`句柄，第三行是`4000000`这是基址，这两个参数刚刚被获取。第四行就是进程的名称了`Ollydbg.exe`。看来我们获取了`OD`的名字了

返回程序领空后，来到`CloseHandle`这里了

`hObject=000088`

`CloseHNDLE`

就是关闭句柄为`88`的进程，这就是要关闭`OD`的节奏吗？

来到`H`窗口，发现的确没有`0x88`这个句柄了。

继续`F8`，来到下一个函数，进入以后发现这里是在比较字符串。显然是在根据这个进程的其他特征在确认是不是调试器本身。

确认过眼神，是想干掉的人。`OD`已经暴露了，接着程序调用`OpenProcess`，根据`PID`获取其句柄（

因为它刚把句柄关了）。

`Call <JMP.&kernal32.OpenProcess>`

`Mov [ebp-24],eax`                                          把获取到的句柄给出去

`Cmp [ebp-24],0`                                             看看这个`PID`对应的句柄是不是为`0`

`Je short 00401e0f`                                         句柄显然不是`0`，不跳出去

接着就开始执行`TerminateProcess`了，这个句柄作为参数传入，将调试器进程关闭。

`Push 0`                                 `ExitCode=0`

`Pusn [ebp-24]`                      `hProcess`

`Call TerminateProcess`        `TerminateProcess`

## 总结：

因为有的函数被隐藏，或在函数列表里未找到可疑函数，所以利用`getprocaddress`捕捉它是否间接调用其他隐藏函数，它获取了几个关键函数

`EnumProcess`

`EnumProcessModule`

`GetmodulebasenameA`

这三个函数`EnumProcess`获取所有进程`PID`

接着调用`openprocess`获取对应`PID`的句柄

利用`EnumProcessModule`利用句柄获取该函数模块的基址

再调用`GetmodulebasenameA`获取当前符合两参数的模块名称（就是进程名）

获取完成了就立即关闭这个句柄，不再占用资源

将进程名与指定字符串比较。

如果相等，利用`openprocess`重新获取句柄，调用结束进程函数

如果不等，由于之前关闭了句柄，所以不会造成无用句柄的大量堆积

以上就是检测进程名的完整过程

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2018-5-9 11:56

被`r0Cat`编辑，原因：

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
