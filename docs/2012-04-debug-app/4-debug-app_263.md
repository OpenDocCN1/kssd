
# 《游戏修改器 DIY》

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-64.htm`](https://www.pediy.com/kssd/pediy07/pediy7-64.htm)

## 《潜艇大战》v1.0 修改器及源码下载（内含游戏）

近日在整理电脑光盘时，在其中发现了一款名叫《潜艇大战》的小游戏，做得挺有趣的，可怎么也打不通关……郁闷中！于是找来《金山游侠》开始作弊，嘿嘿……结果嘛——当然是爆机喽！

每次玩都用《游侠》修改岂不是很不爽？！那就让我们一起来做个游戏修改器吧！

### 我们的目标是——
《潜艇大战》（版本：`1.0`）

### 所需工具——
*   《金山游侠 III》（玩家们都应该知道吧？版本：`III`）
*   `SOFTICE`（一款强大的程序调试工具，版本：`4.0.5 Build334+IceDump`）
*   `Microsoft Visual C++`（编程利器！版本：`6.0+SP5`）

### 预备知识——
游戏修改器的原理很简单，就是修改内存中数据。那么就要用到下面这个 API 函数：`WriteProcessMemory`，其函数原型为：

```c
BOOL WriteProcessMemory(
    HANDLE hProcess,              // handle to process whose memory is written to
    LPVOID lpBaseAddress,        // address to start writing to
    LPVOID lpBuffer,             // pointer to buffer to write data to
    DWORD nSize,                 // number of bytes to write
    LPDWORD lpNumberOfBytesWritten // actual number of bytes written
);
```

有了这个 API 函数我们就可以修改内存中的数据了，让我们一步一步来，把这个游戏做成一个不死版的游戏——

### 第一部分：获得游戏中关键的数据——这是制作游戏修改器的前提。

首先运行《潜艇大战》，在游戏开始后你只有 5 次机会，《金山游侠》的使用方法我就不在赘述了，很简单，相信众玩家都会使用。通过一系列的操作并测试后，发现用于存放生命的内存地址在：`0x007B0BE8`。OK！让我们调出 `SOFTICE`，输入下面的命令（其作用是：在对内存 `0x007B0BE8` 处有写操作时中断程序）：

```
bpm 007B0BE8 w
```

回车返回游戏后故意使潜艇中弹使生命数减 1，这时 `SOFTICE` 会自动跳出并停留在：

```
0167:00405686 3BCD CMP ECX,EBP
```
（图一）

![](img/ba50a7dcba9f4ec7ba87e203dd8b4e62.jpg)

往上看，大家看到了什么？对，是：

```
0167:0040567D 4A DEC EDX
```

而 `EDX` 存放的就是潜艇中弹之前的生命数，`DEC` 是什么我就不用解释了吧（减一）？到了这里想必大家也就弄清楚了制作不死版游戏的关键，那就是把 `0167:0040567D` 处的代码改成什么都不做——

```
0167:0040567D 90 NOP
```

也就是——即使潜艇中弹了也不会减少生命数！

### 第二部分：用 VC 制作游戏修改器。

我们要制作的这个游戏修改器很简单，不需要 MFC，SDK 即可完全搞定！

启动 `VC`，依次——`[File]-[New]-[Projects]-[Win32 Application]-[输入-Project name]`后`[OK]`，选择`[An empty project]-[OK]`。然后`[File]-[New]-[Files]-[选择-C++Source File]`，起名后`[OK]`。输入以下代码——

```cpp
//////////////////////////////////////////////////////////////////////////////////////////
/*
--《潜艇大战》v1.0 修改器--
作者：赵春生
制作时间：17:18 04-1-15
主页：HTTP://TIMW.YEAH.NET
*/

#include<windows.h>

int APIENTRY WinMain(HINSTANCE hInstance,
HINSTANCE hPrevInstance,
LPSTR lpCmdLine,
int nCmdShow)
{
    //OBJIP
    unsigned int OBJ_ip=0x0040567D;//修改潜艇生命数代码的偏移地址
    unsigned int OBJ_patch[1]={0x89CA8B90};//其中的 90 则是 NOP 的十六进制
    HWND hwndOBJ=FindWindow(NULL,"潜艇大战");//获得游戏的窗口句柄
    DWORD PID;
    HANDLE hProcess;

    if (hwndOBJ!=0)
    {
        SetForegroundWindow(hwndOBJ);//找到句柄后将游戏设置为当前窗口
        GetWindowThreadProcessId(hwndOBJ,&PID);//得到游戏的进程 ID
        hProcess=OpenProcess(PROCESS_ALL_ACCESS,false,PID);//打开游戏进程
        WriteProcessMemory(hProcess,(void *)OBJ_ip,OBJ_patch,1,0);//写入修改后的代码
        CloseHandle(hwndOBJ);
        CloseHandle(hProcess);
    }
    else
        MessageBox(NULL,"游戏还没有启动吧？","错误！",NULL);

    return 0;
}
//////////////////////////////////////////////////////////////////////////////////////////
```

编译后运行程序，看看效果吧？！怎么样？游戏成为不死版了吧？

在这里我只是给大家介绍了制作游戏修改器的一种思路，本程序还可以做得更加完美，例如：直接使用游戏修改器来加载游戏进行修改，而不是像现在这样：要先运行游戏后才能使用游戏修改器；再有，游戏还有一处修改潜艇生命数的地方，这处在：`0167:00404BEE`，大家有兴趣的话可以自己尝试……欢迎大家和我一起进行交流！本游戏的修改器及其源码可从我的主页下载。

17:18 04-1-15

#### 还有一个地方不明白

这句：

```c
long OBJ_patch[1] = {0x7D5};
flag = WriteProcessMemory(HProcess,(LPVOID)OBJ_ip,OBJ_patch,1,0);
```

像上面那样写就行\`\`但换一吓\`

```c
long OBJ_patch1 = 0x7D5;  //这里
flag = WriteProcessMemory(HProcess,(LPVOID)OBJ_ip,(LPVOID)OBJ_patch,1,0);
```

像上面那样写就不行\`\`\`不明白为什么写成数组就不用把缓冲区转类型？

```c
BOOL WriteProcessMemory(
    HANDLE hProcess,  // handle to process whose memory is written to
    LPVOID lpBaseAddress,  // address to start writing to
    LPVOID lpBuffer,  // pointer to buffer to write data to
    DWORD nSize,  // number of bytes to write
    LPDWORD lpNumberOfBytesWritten   // actual number of bytes written
);
```

`lpBuffer` 要的是地址存放(`0x7D5`)的地址。

```c
long OBJ_patch[1] = {0x7D5};
flag = WriteProcessMemory(HProcess,(LPVOID)OBJ_ip,OBJ_patch,1,0);
```

`OBJ_patch` 就是 `0x7D5`(`OBJ_patch[0]`)的地址。

```c
long OBJ_patch1 = 0x7D5;  //这里
flag = WriteProcessMemory(HProcess,(LPVOID)OBJ_ip,(LPVOID)OBJ_patch1,1,0);
```

`OBJ_patch1` 是 `0x7D5`，这样写编译应该无错，但程序就错了，她写的内容不是 `D5`，而是内存 `0x7D5` 位置上的一个字节。

# 如何编写游戏修改器 第二版（SE）昆明 李维

## 前言

本文乃作者多年研究游戏修改器经验的结晶，通过浅显易懂的语言，并晓之以原理，示之以代码，将为您揭开游戏修改器的技术奥秘。本文包含了当今 `Win9x/Me/2000/xp` 平台下游戏修改器的大部分流行甚至可以说“先进”的技术，相信您通过本文的指引，定能写出一款属于自己的游戏修改器。（SE: 本文所述的技术适用于 `Windows 95/97/98/Me/2000/Xp`）

在正式开始前，我将假定您已经具备了如下的阅读条件：

1.  具备在 32 位 `Windows` 下使用 `C/C++` 编程的基本知识(SE:最好是 `VC`)，大致了解 `Windows` 的结构及基本原理。
2.  可供查询的 `Windows` 编程资料，如 `MSDN` 和著名的《Windows 核心编程》等。
3.  使用过游戏修改器，开个玩笑:)

## 目录

*   如何编写游戏修改器... 1
*   前言... 1
*   目录... 1
*   游戏修改器的基本工作原理... 1
*   如何访问游戏程序的内存... 2
*   如何实现热键... 3
*   如何实现暂停游戏... 3
*   如何在游戏中弹出自己的界面... 3
*   如何调整游戏速度... 4
*   抓图功能... 4

## 游戏修改器的基本工作原理

所谓游戏修改器，主要是通过修改游戏程序的内存数据或存盘文件来修改游戏中的相关数据，使之达到“无敌”等效果。

游戏修改器主要分为两类：单一游戏的修改器和通用游戏修改器。顾名思义，前者只能修改特定的游戏，此类修改器也叫“无敌引导程序”或“游戏作弊器”；而后者则能够以“不变应万变”，可以修改大多数的游戏。本文主要讨论后者，相比之下，前者是后者只留下修改功能的“简化版”。

总的来说，游戏修改器主要的功能便是反复搜索并筛选某一特定内存地址并将其按照固定的周期修改为特定的值（所谓的“锁定”），当然，将“内存”换为“文件”便可以以同样的方式搜索文件了。

搜索和筛选是如何实现的呢？假如我们已经可以访问游戏的内存，我们将游戏的内存分块读入一个缓冲区，然后借由下面的函数搜索：

```cpp
//在一个内存块中搜索内容
//成功返回位置
//失败返回-1
//(SE:现在我发现这个顺序查找函数完全可以用 STL 的 std::find() 函数取代，原因是写文章的时候我还不太了解 STL:)

int MemFind(int iStartPosition, LPBYTE pDestBuffer, int iDestBufferLength, LPBYTE pPatternBuffer, int iPatternBufferLength)
{
    signed int iFoundPosition, i;
    iFoundPosition = -1;
    if(iStartPosition > iDestBufferLength)return -1;
    for(i = iStartPosition; i < (iDestBufferLength + 1); i++)
    {
        if(memcmp(&pDestBuffer[i], pPatternBuffer, iPatternBufferLength) == 0)
        {
            iFoundPosition = i;
            break;
        }
    }
    return iFoundPosition;
}
```

由于游戏在内存或文件中保存的数据是二进制格式，因此，当我们搜索一个为 123 的整数时应用如下的格式：

```cpp
int nPattern = 123；
dwOffset = MemFind(nStartPosition, pDestBuffer, nDestBufferLength, (LPBYTE)(&nPattern), sizeof(nPattern))
```

同样，可以这样搜索浮点类型：

```cpp
float rPattern = 123；
dwOffset = MemFind(nStartPosition, pDestBuffer, nDestBufferLength, (LPBYTE)(&rPattern), sizeof(rPattern))
```

由于内存中肯定会存在许多相同的数据，所以第一次我们肯定会搜索到许多地址，而真正我们要找的地址一定包含在其中。所以，我们建立一个临时文件将这些地址保存起来，并设置“有效”标志，如下代码所示：

```cpp
struct CTempItem
{
    DWORD dwAddress;
    BOOL bEnable;
};

CTempItem item;
item.dwAddress = ?????????;
item.bEnable = TRUE;
fwrite(&item, sizeof(item), 1, fp);
```

接下来，当用户进行第二次搜索时，将这些保存在临时文件中的数据取出来，先看 `item.bEnable` 若等于 `FALSE` 则跳过，否则读取 `item.dwAddress` 所指示的游戏内存，并于用户第二次输入的数值相比较，若发现相同，则设置 `item.bEnable = TRUE`；若不同设置 `item.bEnable=FALSE`，表示废弃。完成之后，再次把 `item` 写回文件，当所有 `item` 分析完之后，我们就完成了第二次搜索。再接下来，`bEnable=TRUE` 的地址还有很多，则仍然用第二次搜索的方式反复搜索，直到剩下 `1-2` 个地址为止。（SE:应用读写缓冲区，即成批地读写，可以大大加快速度）

由以上介绍可以看出，游戏修改器的搜索分为 2 个阶段：第一次搜索和第 2、3、4……次搜索，游戏修改器在第一次搜索出的很多地址中分析出与用户输入的数据始终相同的地址。当我们有了目标地址，就可以按照用户的意愿定时或手动方式写入用户指定的数据，这便是游戏修改器的基本原理。

当然，这只是基本原理，当具体编写修改器将遇到许多具体的技术困难，以下章节将为您一一解答。

## 如何访问游戏程序的内存

当我们的修改器运行于 `Windows` 时，首先遇到的问题便是如何访问游戏的内存。

首先，在访问游戏的内存前我们还必须获得游戏进程的句柄，这可以通过 `ToolHelp` 函数获取系统中当前运行的所有进程的列表和各进程的 ID，经由用户选择之后通过 `OpenProcess` 函数来获取。若您的修改器运行于后台，而前台是游戏的话，可以在用户按下“弹出”热键时使用 `GetForegroundWindow` 函数获取游戏窗口的 `HWND`，再使用 `GetWindowThreadProcessId` 转换成游戏进程的 ID，再使用 `OpenProcess` 函数获取游戏进程的句柄。

有了游戏进程的句柄之后，便可以使用 `Windows` 提供的 `ReadProcessMemory` 和 `WriteProcessMemory` 这两个 API 来读写游戏的内存了。但是，在 `Windows9x` 中每个进程均拥有各自独立的 `1GB` 虚拟地址空间，而在 `Win2000/XP` 下更是达到了 `2GB`。显然，搜索这样大的地址空间是不现实的，而且游戏也仅仅用了其中的几十到几百 MB。所以，我们需要使用 `VirtualQueryEx` 这个函数来查询哪些是已经分配的地址，哪些是未用的地址。以下查询与搜索相结合的示范代码：

```cpp
DWORD dwBaseAddress;
SYSTEM_INFO si;
GetSystemInfo(si);
dwBaseAddress = si.lpMinimumApplicationAddress;

while(dwBaseAddress < si.lpMaximumApplicationAddress)
{
    mbi.BaseAddress = (LPVOID)dwBaseAddress;
    ProcessMem.Query((PVOID)dwBaseAddress, &mbi);
    VirtualQueryEx(hProcess, (LPVOID)dwAddress), mbi, sizeof(mbi)
    dwBaseAddress = (DWORD)mbi.BaseAddress + mbi.RegionSize;
    if(mbi.State != MEM_COMMIT || mbi.AllocationProtect != PAGE_READWRITE); //跳过未分配或不可读写的区域
    {
       continue;
    }
    //搜索这块内存区域
}
```



## 资源
请到 `http://alphasun.betajin.com` 下载我写的一个很“简陋”的游戏修改器——`GameProbe` 的源程序。

## 如何实现热键
热键的原理很简单，使用全局键盘 `HOOK` 就可以了。鉴于这方面的资料较多，具体的 `Hook` 使用方法请参阅 `MSDN` 或相关资料。

## 如何实现暂停游戏
这是游戏修改器必须具备的一项功能了，若不暂停游戏，搜索之前数值很可能会改变，从而造成找不到数据的现象。暂停游戏的办法有很多种，主要有：

1. `Debug` 法，这种方法利用 `DebugActiveProcess` 这个 `Windows` API 将游戏修改器作为游戏的调试程序，游戏修改器可通过 `Windows` 提供的调试事件（`DebugEvents`）来获取游戏的各个线程的句柄。但此法有一个缺点就是不能在关闭游戏前关闭修改器，否则 `Windows` 将会自动终止游戏的运行，运用此方法的典型是 `FPE2000`。
2. 通过使用 `ToolHelp` 系列函数获取游戏进程的所有线程，并使用公开的 `OpenThread` 这个 API 来获得各线程的句柄并使用 `SuspendThread` 和 `ResumeThread` 来暂停或恢复游戏的运行，目前大部分程序运用此法。（BTW: 著名的 `GameMaster` 使用的是 `CreateProcess` 并通过 `Suspend` 游戏主线程的方法暂停游戏的，很显然，若游戏采用了多线程，此方法是欠妥的。）

但恼人的是 `Win9x` 下并没有 `OpenThread` 这个 API，不过我们可以通过一个未公开的使用 `TCB` 的方法在 `Win9x` 下替代 `OpenThread` 而获取线程的句柄，从而达到了暂停游戏运行的目的。您可以到 `http://www.windrun.com/` 下载 `Ligtest` 前辈编写的 `PauseProcess` 程序的源代码来学习，或者下载我的 `TestPopup`。

## 如何在游戏中弹出自己的界面
这个问题可以和热键问题一并解决：众所周知，`Windows` 是一个消息驱动的 32 位操作系统。在 `Windows` 中，所有正在运的进程都有一个独立的 2GB 的虚拟地址空间，进程之间相互不可见。`Windows` 的绝大多数 API 与消息是不能跨越进程的。

`Hook` 在 `Windows` 中主要是用来截取消息的，形象说，就是用来“钩” 消息的。`Hook` 实际上是一个处理消息的程序段，每当特定的消息发出，在没有到达目的窗口前，`Hook` 函数就先捕获该消息，即 `Hook` 函数先得到处理消息的控制权。而且如果你把 `Hook` 实现在 `DLL` 文件中，那么 `Hook` 函数将会自动被系统映射到会处理那个特定消息的窗口所在的进程虚拟地址空间中。例如，你可以用 `Hook` 来捕获系统中所有的键盘输入消息（`WM_KEYDOWN`）来实现对电脑使用者的输入进行记录（关于 `Windows` 进程管理与 `Hook` 的详细用法，请参阅 `MSDN` 与相关资料）。

微软的 `DirectX` 为 `Windows` 下的游戏带来了华丽的声光效果。但是由于 `DirectX` 采用直接访问硬件的方法提高多媒体与游戏程序的速度，因此导致了人们误以为在 `DirectX`（确切地说是 `DirectDraw`）下不能显示普通的 `Windows` 对话框。

幸运的是，`DirectX` 是支持 `GDI` 的，也就是说游戏程序可以用常规的方法在 `DirectX` 下显示对话框（在微软 `DirectX` 8 SDK 中有名为“`FullScreenDialog`”的例子）。所以现在我们的问题变为：如上所述，如何让我们的程序进入游戏的程序的内部并显示对话框。

这似乎是一个很棘手的问题。但是，有了上面所讲的 `Hook` 情况就大为不同了。我们知道既然 `Hook` 可以映射到别的进程内部，那么只要将显示对话框的函数以及对话框资源包括在 `Hook` DLL 中不就可以调用 `DialogBox()` 了吗？完全正确！我们用 `SetWindowsHookEx()` 为系统设置一个键盘消息 `Hook`，系统会自动将这个 DLL 映射到游戏的进程中。每当有键盘消息，我们只要判断是不是我们所设定的热键，如果是就调用 `DialogBox()` 显示对话框即可。(SE: 还有一种方法可以把 DLL 插入别的进程，那就是利用 `RemoteThread` 远线程，具体程序请参阅上文提到的 `PauseProcess` 程序。)

您可以参阅我的 `TestPopup` 程序，就演示了在 `DirectX` 下弹出界面、暂停游戏、热键等功能。

## 如何调整游戏速度
游戏通常是通过 `timeGetTime`、`GetTickCount` 等几个与时间相关的 API 来控制速度的。因此，我们只要抢在游戏调用这些 API 之前调用它们，并修改返回值，便可以调整游戏的速度了。具体的程序可以参阅我的 `SpeedMan`，您可以免费下载它的源程序。`SpeedMan` 使用了《`Windows` 核心编程》所提供的 `CApiHook` 类来拦截 `timeGetTime` 等 `Windows` API。

## 抓图功能
抓图功能很好实现。具体就是先暂停游戏，再通过使用 `GetForegroundWindow` 函数获取游戏窗口的 `HWND`，最后 DC 就可以获得 Bitmap 格式的屏幕图像了，至于如何保存成文件，或转换为 Jpeg（可以使用 `IJLib`）等，则不在本文讨论范围里了。

## 结束语
相信您通过阅读本文，已经大致了解了游戏修改器的关键技术了吧？希望本文能助您写出更新、更好的游戏修改器。

有任何问题欢迎您来信讨论：`malloc@km169.net`

若要转载（包括传统媒体），必须提前通知作者，征得作者同意。 此文仅供研究和学习，作者不对文章造成的后果负责。

版权所有 李维 (`CSDN` 帐号: `xenix`) 保留所有权力

`4dd466d0e1e0cc123afbe504da2462ca`
