

# 另辟蹊径，VMProtect 分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-556.htm`](https://www.pediy.com/kssd/pediy09/pediy09-556.htm)

> VMProtect 分析  
>   
> 看了 forgot，bughoho，somuch 的分析文章，获益菲浅，不过这都是理论基础，要实际分析一个真正的程序时，这么做的话要累死人的。我分析了一个程序，主 switch 跑了至少几万次啊，看的我头晕眼花，于是想实际分析程序应该不是这样的，就找 API，断到到自己怀疑的地方的时候，再慢慢分析，可也跟的很辛苦，从网上看到这篇文章，觉得非常好，省了很大的劲，幸亏比较短，所以翻译了下，要不就放弃了。  
>   
> 与大家分享， 交流是目的。  
>   
> 概述  
> 这个有害软件是个木马下载器，今天的分析重点不在程序本身的逻辑，而在看这个文件是怎么针对逆向工程被保护的。这个文件是被 VMProtect 保护的，这个保护把汇编码变成伪码。本二进制文件中嵌入了一个 VM 来解释新创建的代码，反汇编器就没有用了，因为她不支持 Virtual CPU 指令。VMProtect 用了个非常复杂的 VM，分析她很费时间。VM 在每个被保护的文件中是不同的，以至于在分析这个文件得出的结果在以后别的分析用不上。因为输入表没有被搞乱，所以我用另外一个方法来分析代码。你将发现我用一个 OD 的脚本来在每个输入的 API 设断。简单分析 IAT。然后就会发现很容易搞清楚程序流程和她到底干了什么。  
>   
> 技术细节  
> 如果我们用个 PE Editor 来查看文件，就会看到一个带 vmp 的段，这就提示了我们的文件是被 VMProtect 保护了的。用 IDA Pro 来反汇编，我们看到程序开始的是汇编码，但又不是纯汇编，显然是故意搞乱的。不过能看到一个重要信息，本可执行文件的 ImageBase 是 0x13140000。  
>   
> 程序入口的汇编：  
>   
>   
> 稍微懂点汇编的人都能看出程序不是干净的，就象我前面说过的，我要用另外个技术来跟这个例子程序背后的流程和理解代码逻辑。看一下 IAT，看到几个感兴趣的 API，这个程序很有可能干了些什么，她们给了我们很多信息。  
>   
> CreateRemoteThread, WriteProcessMemory 。。。等，我们不知道哪个进程或 dll 被插入了，或在汇编的基础上加了一块，或是用什么其他别的什么技术在远程进程加了代码。为了找出来，我做了个 OD 脚本来扫描 IAT，并且在每个输入的 API 上设断。IAT 从 13142000 开始。  
>   
> ////////////////////////////////////////////////////  
> var IATstart  
> var IATslots  
>   
> mov IATstart, 13142000 // IAT start address  
> mov IATslots, 15 // how many apis to handle.  
>   
> breakpoint\_loop:  
>  cmp \[IATstart\],0 // don't set breakpoint at nulls between IID.  
>  je skip\_breakpoint  
>  bp \[IATstart\]  
> skip\_breakpoint:  
>  add IATstart, 4  
>  dec IATslots  
>  cmp IATslots, 0  
>  jne breakpoint\_loop  
> /////////////////////////////////////////////////////  
>   
> 现在，我们用 OD 运行程序，到入口时，跑这个脚本，他会给我们设断。  
>   
> 我们可能已经在 GetProcAddress 和 LoadLibraryA 加了断点。（可是有害软件可能会模拟这 2 个 API，请尽量在 VMWare 上或一个空的机器上调试有害软件）  
>   
> F9，看看程序是怎么断的，得到：  
> 0012FFBC   13145D45  /CALL to GetModuleHandleA  
> 0012FFC0   00000000  \\pModule = NULL  
> 0012FFC4   7C816D4F  RETURN to kernel32.7C816D4F  
>   
> 程序调了几次 GetModuleHandleA，没发现什么感兴趣的。  
>   
> 0012FFB8   13145DB7   /CALL to FindWindowA  
> 0012FFBC   13141152  |Class = "shell\_traywnd"  
> 0012FFC0   00000000  \\Title = NULL  
> 0012FFC4   7C816D4F  RETURN to kernel32.7C816D4F  
> 0012FFC8   7C920738  ntdll.7C920738  
> 然后在类 Shell\_traywnd 里调了 FindWindow，类 Shell\_traywnd 属于 explorer.exe，提示：这个文件可能往 explorer.exe 里插入代码。  
>   
> 0012FFB8   13145F29  /CALL to GetWindowThreadProcessId  
> 0012FFBC   00050054  |hWnd = 00050054 (class='Shell\_TrayWnd')  
> 0012FFC0   13141350  \\pProcessID = show.13141350  
> 0012FFC4   7C816D4F  RETURN to kernel32.7C816D4F  
>   
> 然后用 GetWindowThreadProcessID 得到一些进程信息，handle 是前面 FindWindowA 返回的。  
>   
> 0012FFB4   1314610B  /CALL to OpenProcess  
> 0012FFB8   001F0FFF  |Access = PROCESS\_ALL\_ACCESS  
> 0012FFBC   00000000  |Inheritable = FALSE  
> 0012FFC0   000001CC  \\ProcessId = 1CC  
>   
> 打开进程，权限 PROCESS\_ALL\_ACCESS，然后你会看到 VirtualAllocEx 分配了一块内存，地址为 13140000  
>   
> 0012FFAC   13145E23  call to VirtualAllocEx  
> 0012FFB0   00000048  
> 0012FFB4   13140000  show.13140000 // Address of allocation.  
> 0012FFB8   00008000  
> 0012FFBC   00003000  
> 0012FFC0   00000040  
>   
> 注意：  
> 必须注意，我们程序想在 explorer.exe 里分配内存，用他自己的 ImageBase 做 VirtualAllocEx 的参数。我们程序有个故意弄的不标准的 ImageBase，这么做的背后目的是如果插入程序能分配同样地址的内存，他就能把自己直接拷贝到远程进程。什么意思？简单说就是被插入代码不需要在原地址上加上个增量来加代码，不需要重定位。输入表在插入代码 IAT 已经解决了，这样当被插入到远程进程时，输入表还是对的。终于，这次内存分配后，调 WriteProcessMemory 写入 8000h 到 explorer.exe 的地址 13140000。  
>   
> 0012FFAC   13145EB6  /CALL to WriteProcessMemory  
> 0012FFB0   00000048  |hProcess = 00000048 (window)  
> 0012FFB4   13140000  |Address = 13140000  
> 0012FFB8   13140000  |Buffer = show.13140000  
> 0012FFBC   00008000  |BytesToWrite = 8000 (32768.) ; writes 8000 bytes  
> 0012FFC0   13141354  \\pBytesWritten = show.13141354  
> 0012FFC4   7C816D4F  RETURN to kernel32.7C816D4F  
>   
> 最后，调 CreateRemoteThread 来执行被插入代码。CreateRemoteThread 后的地址是 13141168。  
>   
> 0012FFA4   13145D39  show.13145D39  
> 0012FFA8   00000048  
> 0012FFAC   00000000  
> 0012FFB0   00000000  
> 0012FFB4   13141168  show.13141168 // Start address of the remote  
>                                       thread==> the interesting code.  
> 0012FFB8   13140000  show.13140000  
> 0012FFBC   00000000  
> 0012FFC0   13141358  show.13141358  
>   
> 看下内存空间，  
>   
>   
> 能看到程序逻辑很简单，被插入代码从删除插入原程序开始，为的是毁灭罪证。  
>   
> 然后，他从网上下了另外一个可执行程序，放到 Windows 目录下，这个新建的程序就执行了。  
>   
> Outro（这个实在不知道是什么地方的英语土话，就当是最后吼 2 嗓子吧）:  
> 我希望这个小文章能给大家些分析复杂的被搞乱的程序的主意，这个例子比较简单，但这些技术会被用到不同的场合的。  
>   
> 原文：  
> http://www.websense.com/securitylabs/blog/blog.php?BlogID=77 

 4dd466d0e1e0cc123afbe504da2462ca