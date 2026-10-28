
# [原创]最适合新手的病毒分析——彩虹猫病毒

## 前言

彩虹猫是一款很有意思的病毒，结构也很简单，适合小白分析练手。这篇分析也是我的一个练习项目，不但展示了分析结果，也重点阐述了分析过程中遇到问题时的解决思路，尽量做到详细、多图、无死角，希望能对同样是新手的读者一点帮助。

> 样本名称：`MEMZ.exe`
> 
> MD5：     19DBEC50735B5F2A72D4199C4E184960
> 
> SHA1：    6FED7732F7CB6F59743795B2AB154A3676F4C822
> 
> 实验平台：Windows 7 Ultimate With Service Pack 1
> 
> 分析工具：PEiD v0.95、StudyPE x64 v1.11、IDA x32 v7.4、Bochs v2.6.9、VMWare WorkStation v15.56、Olly ICE v1.10

## 一、初步观察和信息收集

拿到样本不要一来就拖进IDA和OllyDbg，在没有任何参考信息的情况下很容易迷失方向，应该先做最基本的观察，为之后的分析提供思路。特别是这次的彩虹猫样本，有很明显的用户能直接观察的特征。让我们先把病毒在虚拟机中运行起来。

双击病毒样本后接连弹出两个警告弹窗，希望用户明白运行病毒的风险。点击确定后，桌面会慢慢出现一些诡异的现象，比如：

- 自动弹出多个浏览器搜索窗口
- 鼠标异常晃动
- 窗口颜色怪异
- 反复出现系统提示音
- 出现6个`MEMZ`进程

这6个名为`MEMZ`的进程，用户尝试关闭任意一个，或者手动关闭计算机，都会遭至大量弹窗随后蓝屏，接着Windows无法正常启动，只会循环播放一只彩虹猫附带欢快的背景音乐，无论你重启多少次都是如此。运行观察结束后再用一些基本的可执行文件信息查看工具进行勘察。

首先上场的是PEiD，监测到该样本可能使用了ASProtect v1.32进行加壳或加密，可能会给本次分析带来一点小困难

![](https://bbs.kanxue.com/upload/attach/202011/873189_K3YR29C2A87WRY8.jpg)

接着使用StudyPE+查看样本的导入表，发现了很多有意思的函数，这些导入函数能给我们提供大量的信息，我把这些函数大致分为三类：

![](https://bbs.kanxue.com/upload/attach/202011/873189_NAJAHQFHVE9UT7Q.jpg)

第一类：很明显能和观察到的现象直接联系

- 自动弹出多个浏览器搜索窗口----------------------`ShellExecute`
- 鼠标异常晃动--------------------------------------`SetCursorPos`、`GetCursorPos`
- 桌面出现奇怪图标---------------------------------`DrawIcon`
- 窗口颜色怪异--------------------------------------`BitBlt`、`StretchBlt`
- 反复出现系统提示音-------------------------------`PlaySoundA`
- 出现6个`MEMZ`进程--------------------------------`ShellExecute`

第二类：不能直接关联，但常成对出现完成某些功能

- `OpenProcessToken`、`AdjustTokenPrivilege`、`LookUpPrivilegeValue`    给进程提权
- `GetMessage`、`TranslateMessage`、`DispatchMessage`                           建立消息循环
- `CreateToolhelp32Snapshot`、`Process32First`、`Process32Next`              遍历进程
- `SetWindowHookEx`、`UnhookWindowHookEx`、`CallNextHookEx`         给窗口下钩子
- `LoadLibrary`、`GetProcAddress`                                                               加载库并导入函数

第三类：不确定干了什么事或者我根本不熟悉

- `SendInput`、`GetSystemMetrics`、`DefWindowProc`……

对于复杂的样本，还可以使用监控类工具看看样本对文件、进程、注册表、网络进行了哪些操作，不过这次样本比较简单，以上信息已经足够指导接下来的分析工作了。

## 二、入口部分

使用IDA载入样本，定位到入口函数`Start`并按下F5获得伪代码，该功能极大地提升了样本分析的效率，毕竟看高级语言比看汇编还是快得多。但是这些伪代码只是IDA根据现有信息猜测出来的，**并不能保证100%准确**，有的时候甚至会对我们的分析工作产生误导，一会你就能看到。现在让我们继续回到IDA生成的伪代码

![](https://bbs.kanxue.com/upload/attach/202011/873189_MGYQ3RHJEKG57HS.jpg)

暂时别着急深入细节，先从整体把握走向。在`start`函数的起始部分调用`GetCommandLineW`、`CommandLineToArgvW`获取进程参数，之后整个`Start`函数就根据进程参数分成了几大块，整理后其结构如下

![](https://bbs.kanxue.com/upload/attach/202011/873189_C64WK8YXTNSZS5B.jpg)

注意，这段体现`start`函数结构的伪代码并不是直接照搬自IDA，而是为了整体逻辑清晰而特意整理修改过的。接下来就要依次分析这三大部分，为了方便表述，这三个部分我称为“`watchdog`部分”、“`/main`部分”和“无参数部分” 。我们第一次双击样本时自然是不带参数的，所以我们先来看`start`函数的无参数部分。

## 三、无参数部分

此处先是用`MessagexBox`依次弹出两个警告信息提示框，这也正是我们运行病毒时所看到的。若两个弹窗都被用户确认，使用`GetModuleFileNameW`获取当前进程的路径并保存，当然所保存的空间由`LocalAlloc`提前申请。接着用`do while`执行5次循环，每次循环都调用一次`ShellExecuteW`，参数正是刚才得到的样本进程路径，以及字符串`"/watchdog"`，即以`"/watchdog"`为参数，生成5个`MEMZ.exe`进程。

![](https://bbs.kanxue.com/upload/attach/202011/873189_6XCGG43K3ZR8BQJ.jpg)

在最后调用了`ShellExecuteExW`以`"/main"`为参数生成了一个`MEMZ.exe`进程，一般带Ex后缀是加强版函数，表示在普通版函数基础上进行了功能的扩展，这里特意用了一个功能更强的函数一定有特别的目的。该函数接收一个`SHELLEXECUTEINFOA`结构体，结构体中设定了进程路径和执行参数，跟前面普通版函数传递的参数其实没什么区别。关键是紧接着调用了`SetPriorityClass`，当中对`hProcess`成员复制为`0x80`，查看MSDN得知该值意味着创建的进程将拥有最高的响应优先级，最后调用`ExitProcess`使当前进程结束。如果你不会使用MSDN我在文末备注有介绍。

![](https://bbs.kanxue.com/upload/attach/202011/873189_F7Q9ZX8B7494V4Z.jpg)

现在我们知道在观察阶段看到的6个`MEMZ.exe`进程是怎么来的了：最开始双击生成了一个原始进程，原始进程生成了5个`watchdog`进程和1个`/main`进程，然后原始进程迅速结束了自己，等我们打开任务管理器查看时，还存在6个`MEMZ.exe`

IDA全称是交互式反汇编器（Interactive Disassembler），重点在于“交互式”，我们可以把手动分析后的任何想法和有用的信息都记录在IDA里，比如给IDA自动生成的变量和函数起一个有意义的名字，这些名字会在所有用到该变量或函数的地方生效。完成后应该像下图这样。

![](https://bbs.kanxue.com/upload/attach/202011/873189_V2ZJB3E4S85BGMT.jpg)

在后面的分析中，请大家自行完成注释的添加，为了后面的文章不剧透，我贴出来的图大部分还是未注释的状态。至此无参数部分分析完毕。

## 四、“`watchdog`”部分

### 1. 主体

该部分的主体代码量很少，我一张图就全展示清楚了，但里面的坑可一点都不少。

![](https://bbs.kanxue.com/upload/attach/202011/873189_TAYKUEAU9WWDG2R.jpg)

首先是`CreateThread`创建一个线程，IDA给自动命名为`sub_40114A`。在这里给大家提一个醒，以`watchdog`为参数运行的`MEMZ.exe`进程是有5个的，也就是说这里看到的创建一个线程是“某个进程“的行为，5个进程一共应该创建了5个线程，在后面的分析中大家也要注意这一点。我们进入`sub_40114A`看看这个线程干了些什么事。

### 2. 子函数`sub_40114A`

同样代码很少一张图概括完。前三个函数用来获取当前进程路径，顺便说一下这种多个函数配合起来做事的情况有很多，常常是最后一个函数完成关键功能，前面几个都是为它提供必要的参数，比如`LocalAlloc`申请空间用来存路径字符串，`GetCurrentProcess`获取当前进程句柄，这都是第三个函数要用到的。

接着一个大的`while`循环把后面的代码都包了进去，注意该`while`循环的条件永远为真，说明是个死循环。在循环内部使用`CreateToolhelp32Snapshot`给进程拍快照，再用`Process32FirstW`和`Process32NextW`进行遍历，这都是非常常见的操作，我们在观察阶段查看导入表时也发现了这三个函数，这里就用上了。

当然光做进程遍历是没意义的，肯定要干点活。每当遍历到一个进程时都要获取它的路径，并和我们之前获取的路径对比，如果相同就让计数器加1，很明显这就是在看当前有多少个`MEMZ.exe`的进程。当遍历完所有进程后，我们就统计出了`MEMZ.exe`的进程个数。然后最关键的地方来了，由于`Sleep`函数的存在，死循环每隔一段时间就能统计出当前`MEMEZ.exe`进程的个数并存放在`v4`变量中，然后先判断`v4`和`v7`的大小，再让`v7`保存`v4`的值。

如此就形成了一个监测机制，`v7`永远只保存`v4`最大的值，一旦`v4`的值小于了`v7`就会被`if`语句监测到，并进入`sub_401021`子函数。这不正好和我们观察阶段的发现一致吗？只要`MEMZ.exe`进程数量减少，系统就会蓝屏重启，这就是它的监测原理。反过来我们可以推测`sub_401021`就是完成系统蓝屏重启功能的。

![](https://bbs.kanxue.com/upload/attach/202011/873189_RB6RVDVDQSGK2A2.jpg)

### 3. 子函数`sub_401021`

前面进展得很顺利，到这里就碰到麻烦了。在这个子函数的开头是一个`do while`循环，循环次数为20次，用`CreateThread`创建线程，这没什么特别的，像往常一样我们双击`StartAddress`准备去看这个线程做了什么事，但是双击过后却没有任何反应，这其实是IDA的代码识别出现错误，相关问题我在第七节“IDA的一些问题”中做详细解释。

![](https://bbs.kanxue.com/upload/attach/202011/873189_TTWA39KM4YNN2FR.jpg)

在这里为了不影响我们的分析主线，直接键盘按G输入地址`4010FE`就可以跳到`StartAddress`的代码。该处的`MessageBoxA`用于弹出消息，有26条消息保存在`lpText`所指向的地址中。使用`sub_401A55`获取随机数保存在`v3`，`v3`对`0x1A`取余以实现在26条消息中随机选取一条并显示。

![](https://bbs.kanxue.com/upload/attach/202011/873189_3CQCRRAS387EPSZ.jpg)
![](https://bbs.kanxue.com/upload/attach/202011/873189_JEETS3FMQ2ZX9DJ.jpg)

而`SetWindowsHookEx`和`UnhookWindowsHookEx`用于给窗口下钩子，干了什么事要到回调函数`fn`里去看。`code==3`表明目标窗口即将被创建，此时`lParam`表示该窗口的基本信息（坐标、大小等），修改这些信息可以在窗口真正创建之前生效，下方正是在随机修改窗口的位置。这些信息都可以在MSDN中查到。

![](https://bbs.kanxue.com/upload/attach/202011/873189_A93BTXN9VEHBUYY.jpg)

`sub_401021`的后半部分是两种关机的方式：先主动诱发蓝屏关机，如果不成功则主动退出Windows。

前一种方式使用`LoadLibrary`加载`ntdll`库，再连续两次调用`GetProcAddress`获取两个函数地址，这两个函数是未公开的Windows API，只能用这种方式来隐式调用。随后依次调用这两个函数，主动引发蓝屏。顺便提一下，很多恶意程序在隐式调用时会对函数名字符串先加密，要用的时候再解密，或者干脆不使用函数名而用函数编号，显然我们分析的样本没有做这种“隐蔽处理”。

后一种方式先用一系列函数给当前进程提权，然后调用`ExitWindowsEx`主动关闭Windows。为什么要先提权？调用`ExitWindowsEx`关闭Windows需要相应的权限，详情见MSDN。

![](https://bbs.kanxue.com/upload/attach/202011/873189_W2SXDFJK3P5NWDT.jpg)

总结一下，我们确认了之前的猜想，`sub_401021`确实是一个强制关机的函数，先创建线程用于弹出大量位置和内容都随机的窗口，再使用蓝屏或退出Windows的方式强制关闭计算机。

### 4. 注册并创建窗口

我们依次分析了函数开头->`sub_40114A`->`sub_401021`，已经陷入了太多细节，是时候回到`watchdog`的主体部分。这张图之前出现过，为了大家方便看我再次贴出来。在创建`sub_40114A`后，调用`RegisterClassExA`注册了一个名为“`hax`”的用户自定义窗口类型，并用`CreateWindowExA`将其创建。此处有个小问题，传递给`RegisterClassExA`函数的结构体变量`pExecInfo`定义为了`SHELLEXECUTEINFOW`类型，而不是该函数所需要的`WNDCLASSEXA`类型，这是IDA的识别出现了错误，为了不干扰主线分析，其讲解安排在了第七节“IDA的一些问题”。

![](https://bbs.kanxue.com/upload/attach/202011/873189_PHJP72C9938JE2W.jpg)

### 5. 回调函数`sub_401000`

创建的窗口有一个回调函数`sub_401000`。经查询MSDN，常量值16和22分别对应窗口消息`WM_CLOSE`和`WM_ENDSESSION`，那么整体含义就很明了了：该窗口回调函数会对窗口消息进行过滤，若消息为`WM_CLOSE`或者`WM_ENDSESSION`，则调用`sub_401021`强制关机（这个函数前面刚分析过）。而`WM_CLOSE`或`WM_ENDSESSION`消息是在系统关机时，由操作系统发送给各个窗口。

若是其他消息，则并不做任何处理，丢给系统默认处理函数`DefWindowProcW`。

![](https://bbs.kanxue.com/upload/attach/202011/873189_46HRZW2FF7CSKWY.jpg)

如此一来，`sub_401021`这个强制关机函数在两处被调用。第一个是监测`watchdog`进程数量，如有减少就调用。第二个是监测用户是否主动关机，如有也调用。这和我们观察阶段看到的完全一致。

### 6. 消息循环

紧接着的代码`GetMessage`、`TranslateMessage`、`DispatchMessage`被包进一个大的`while`循环，这是常见的操作，叫做“消息循环”。由于在前面创建了窗口，并且还对发送给窗口的消息进行了过滤，意味着我们必须自己写消息循环完成收取消息和派发消息的工作，否则创建的窗口是收不到消息的。Windows并不会自动帮我们完成消息循环，这在MSDN里有明确解释，给出两条参考链接：

一篇关于消息循环的博客

[ae6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5Y4S2B7j5i4W2Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3U0l9H3z5g2)9J5c8U0l9$3i4K6u0r3x3U0N6Q4x3V1j5I4y4e0p5J5x3K6M7J5i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@1ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5Y4S2B7j5i4W2Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3U0l9H3z5g2)9J5c8U0l9$3i4K6u0r3x3U0N6Q4x3V1j5I4y4e0p5J5x3K6M7J5i4K6u0W2K9s2c8E0L8l9`.`.)

MSDN中关于创建消息循环的官方回答
