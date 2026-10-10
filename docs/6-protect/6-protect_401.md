

# 检测 OllyDbg 的另类方法 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6866.htm`](https://www.pediy.com/kssd/pediy06/pediy6866.htm)

原文链接：[`bbs.pediy.com/showthread.php?s=&threadid=4013`](http://bbs.pediy.com/showthread.php?s=&threadid=4013)

这是 linhanshi 兄放在他的网站上的，我觉得不错就翻译了下，水平很烂，多多包涵！  
  
The other ways to detect OllyDbg 检测 OllyDbg 的另类方法  
  
Pumqara 作/RoBa\[TT\]译  
  
前言  
  
现在是 2004 年了，RING-3 级调试器被越来越多地使用，因为它们有图形界面并且比 RING-0 级调试器（比如 SOFTICE）更加方便。在这篇文章中我将讲述如何检测最好的 RING-3 调试器之一——OllyDbg。许多人都听说过 IsDebugerPresent 和 fs:\[20\]检测手段，但是有没有其他的新方法呢？下面我向你介绍我自己的一些检测手段。我会给出详细的解释，因此你可以用你自己的想象力来完善它们。  
  
方法一：FindWindow  
  
这个方法是基于 FindWindow 函数。像所有的对话框一样，OllyDbg 的主对话框（窗口？）也有它的标题和类名。使用这个 API 函数我们可以判断 OllyDbg 的主窗口是否打开。Microsoft 这样写道：  
\------------------------------------------------------------------------------------------------  
FindWindow 函数能够获得窗口类名或标题为特定字符串的顶层窗口的句柄。该函数不搜索子窗口。  
  
HWND FindWindow(  
  
LPCTSTR lpClassName, //窗口类名的地址  
LPCTSTR lpWindowName //窗口标题的地址  
);  
  
参数说明  
  
lpClassName  
指向一个以 NULL 结尾的表示窗口类名的字符串的指针，或者是一个标识窗口类名字符串的 atom。如果该参数是一个 atom，它必须是被函数 GlobalAddAtom 预先建立的一个全局 atom。这个 16 位的 atom 必须放在 lpClassName 的低 8 位，lpClassName 的高 8 位必须为 0。  
  
lpWindowName  
指向一个以 NULL 结尾的表示窗口名称（即标题）的字符串的指针。如果这个参数为 NULL，所有的窗口都被认为是符合条件的。  
  
返回值  
  
如果搜索成功，返回找到的符合条件的窗口句柄。  
如果搜索失败，返回值为 NULL。要得到详细的错误信息可以调用 GetLastError。  
\------------------------------------------------------------------------------------------------  
我的程序片断：  

> ```
> 代码:
> .data
> strOllyClsName db "OLLYDBG",0
> 
> .code
> invoke FindWindow, ADDR strOllyClsName, NULL
> cmp eax, 00000000h
> jne Olly\_Detected/
> 
> ```
> 
>   

  
方法二：CreateToolhelp32Snapshot, Process32First/Next  
  
这是一个有趣的方法。它基于四个 API 函数(CreateToolhelp32Snapshot, Process32First, Process32Next, GetCurrentProcessId)和一个结构( PROCESSENTRY32 )。先看看 MSDN 怎么说：  
\------------------------------------------------------------------------------------------------  
\[CreateToolhelp32Snapshot\]  
  
CreateToolhelp32Snapshot 函数为指定的进程（可以包括进程使用的堆\[HEAP\]，模块\[MODULE\]和线程\[THREAD\]）建立一个快照\[snapshot\]。  
  
HANDLE WINAPI CreateToolhelp32Snapshot(  
DWORD dwFlags,  
DWORD th32ProcessID  
);  
  
参数说明：  
  
dwFlags  
  
\[输入\]TH32CS\_INHERIT - 声明快照句柄是可继承的。  
TH32CS\_SNAPALL - 在快照中包含系统中所有的进程和线程，还包括在 th32ProcessID 中指定的进程的堆和模块 TH32CS\_SNAPHEAPLIST - 在快照中包含在 th32ProcessID 中指定的进程的所有的堆。要列举出进程的堆，查看 Heap32ListFirst。  
TH32CS\_SNAPMODULE - 在快照中包含在 th32ProcessID 中指定的进程的所有的模块。要列举出进程的模块，查看 Module32ListFirst。  
TH32CS\_SNAPPROCESS - 在快照中包含系统中所有的进程。要列举出所有进程，查看 Process32First.  
TH32CS\_SNAPTHREAD - 在快照中包含系统中所有的线程。要列举出所有线程，查看 Thread32First.  
  
th32ProcessID  
\[输入\]指定将要抓取的进程 ID。如果该参数为 0 表示抓取当前进程。该参数只有在设置了 TH32CS\_SNAPHEAPLIST,TH32CS\_SNAPMOUDLE 或 TH32CS\_SNAPALL 后才有效，在其他情况下该参数被忽略，所有的进程都会被抓取。   
  
Return Values  
  
如果调用成功，返回快照的句柄。  
如果调用失败，返回 INVAID\_HANDLE\_VALUE。  
  
Remarks  
  
The snapshot taken by this function is examined by the other tool help functions to provide their results. Access to the snapshot is read only. The snapshot handle acts like an object handle and is subject to the same rules regarding which processes and threads it is valid in.  
  
要列举出所有进程的堆和模块状态，指定 TH32CS\_SNAPALL 并且把 th32ProcessID 设为 0。然后，对于快照中每个新增的进程，再次调用 CreateToolhelp32Snapshot，设置要抓取的新进程 ID 和 TH32CS\_SNAPHEAPLIST 或 TH32CS\_SNAPMOULE.  
  
要删除快照，使用 CloseHandle 函数。  
\------------------------------------------------------------------------------------------------  
\[Process32First\]  
  
Process32First 得到一个系统快照里第一个进程的信息。  
  
BOOL WINAPI Process32First(  
HANDLE hSnapshot,  
LPPROCESSENTRY32 lppe  
);  
  
参数说明  
  
hSnapshot   
\[输入\] 先前调用 CreateToolhelp32Snapshot 函数时返回的指向系统快照的句柄  
  
lppe   
\[输入，输出\] 指向一个 PROCESSENTRY32 结构的指针  
  
返回值  
  
如果进程列表的第一个入口已经被复制到缓冲区中则返回 TRUE，否则返回 FALSE。如果快照中不包含进程信息，通过 GetLastError 会返回一个 ERROR\_NO\_MORE\_FILES 错误。  
  
备注  
  
调用该函数的程序必须把 PROCESSENTRY32 中的成员 dwSize 设定为该结构的大小（用字节数表示）。Process32First 会把 dwSize 改为写入该结构的字节数。这个值永远不会比 dwSize 的初始值更大，但是有可能更小。如果这个值变小了，任何偏移量大于这个值的成员都是不可靠的。  
  
要得到同一快照中其它进程的信息，使用 Process32Next 函数。  
\------------------------------------------------------------------------------------------------  
\[Process32Next\]  
  
Process32Next 可以得到在一个系统快照中下一个进程的信息。  
  
BOOL WINAPI Process32Next(  
HANDLE hSnapshot,  
LPPROCESSENTRY32 lppe  
);  
  
参数说明  
  
hSnapshot   
\[输入\] 先前调用 CreateToolhelp32Snapshot 函数时返回的指向系统快照的句柄  
  
lppe   
\[输出\] 指向一个 PROCESSENTRY32 结构的指针   
  
返回值  
  
如果进程列表的下一个入口已经被复制到缓冲区中则返回 TRUE，否则返回 FALSE。如果快照中不包含进程信息，通过 GetLastError 会返回一个 ERROR\_NO\_MORE\_FILES 错误。  
  
备注  
  
要得到快照中第一个进程的信息，使用 Process32First 函数。  
\------------------------------------------------------------------------------------------------  
\[PROCESSENTRY32\]  
  
当一个快照建立后，PROCESSENTRY32 描述了在系统地址空间中一系列进程中的一条。  
  
typedef struct tagPROCESSENTRY32 {  
DWORD dwSize;  
DWORD cntUsage;  
DWORD th32ProcessID;  
ULONG\_PTR th32DefaultHeapID;  
DWORD th32ModuleID;  
DWORD cntThreads;  
DWORD th32ParentProcessID;  
LONG pcPriClassBase;  
DWORD dwFlags;  
TCHAR szExeFile\[MAX\_PATH\];   
  
} PROCESSENTRY32, \*PPROCESSENTRY32;  
  
成员变量  
  
dwSize   
用字节数表示的结构大小。在调用 Process32First 前，把这个成员设为 sizeof(PROCESSENTRY32)。如果你不对 dwSize 进行初始化，Process32First 会调用失败。  
  
cntUsage   
该进程被引用的次数。只有一个进程的引用次数不为 0 时这个进程才存在。一旦它的引用次数为 0，进程就终止了。  
  
th32ProcessID   
该进程的标识。(ID)  
  
th32DefaultHeapID   
Identifier of the default heap for the process. The contents of this member has meaning only to the tool help functions. It is not a handle, nor is it usable by functions other than the ToolHelp functions.   
  
th32ModuleID   
Module identifier of the process. The contents of this member has meaning only to the tool help functions. It is not a handle, nor is it usable by functions other than the ToolHelp functions.   
  
  
cntThreads   
该进程启动的线程数目。   
  
th32ParentProcessID   
建立该进程的父进程的标识。  
  
pcPriClassBase   
Base priority of any threads created by this process.  
   
dwFlags   
未使用，保留  
  
szExeFile   
指向一个以 NULL 结尾的字符串的指针，该字符串说明了进程所属的可执行文件。  
  
在 Windows Me/98/95 中，这个文件名包含路径。  
\------------------------------------------------------------------------------------------------  
\[GetCurrentProcessId\]  
  
GetCurrentProcessId 返回调用该函数的进程的标识。  
  
DWORD GetCurrentProcessId(VOID)  
  
参数说明：  
  
该函数没有参数。   
  
返回值  
  
返回值为调用该函数的进程的标识。  
  
备注  
  
这个进程标识（ID）在系统中确定唯一的进程，直到进程终止。  
\------------------------------------------------------------------------------------------------  
我们的目标是检测我们的程序的父进程是不是 OllyDbg.  
  
计划：  
1.)用 GetCurrentProcessId 得到我们程序的进程 ID。  
2.)用 CreateToolhelp32Snapshot 和 Process32First/Next 依次比较每一个 PROCESSENTRY32 结构中的 th32ProcessID 成员是不是我们的进程 ID，直到找出为止。  
3.)得到 PROCESSENTRY32 结构中的 th32ParentProcessID。（译注：即父进程 ID）  
4.)用 CreateToolhelp32Snapshot 开始一轮新的比较，但这次比较的是每个 th32ProcessID 成员是不是我们在第 3 步得到的父进程的 ID，直到找出为止。  
5.)得到 PROCESSENTRY32 中的 szExeFile 成员，看是不是"Ollydbg.exe"  
6.)如果是的话就知道我们的程序正在 OllyDbg 控制下运行了。  
  
示例代码：  

> ```
> 代码:
> .586
> .model flat, stdcall
> option casemap:none
> include d:\\masm32\\INCLUDE\\Windows.inc
> include d:\\masm32\\INCLUDE\\user32.inc
> include d:\\masm32\\INCLUDE\\kernel32.inc
> includelib d:\\masm32\\lib\\user32.lib
> includelib d:\\masm32\\lib\\kernel32.lib
> 
> .data
> strCaption db "OllyDbg Detector!",0
> strFound db "OllyDbg found!",0
> strNotFound db "OllyDbg NOT found!",0
> strOllyDbg db "OLLYDBG.EXE",0h
> valCurrentPiD dd 0 
> valParentPiD dd 0
> hSnapShot dd 0
> 
> .data?
> proces PROCESSENTRY32 <>
> 
> .code
> start: 
> ; 建立快照
> invoke CreateToolhelp32Snapshot, TH32CS\_SNAPPROCESS,NULL
> mov hSnapShot,eax
> 
> ; 得到当前进程 ID
> invoke GetCurrentProcessId
> mov valCurrentPiD,eax
> 
> lea esi,offset proces
> assume esi:ptr PROCESSENTRY32
> mov \[esi\].dwSize,sizeof PROCESSENTRY32
> 
> ; 开始第一轮搜索
> ; 用 valCurrentPiD 查找当前进程
> invoke Process32First,hSnapShot,addr proces
> 
> lea esi,offset proces
> assume esi:ptr PROCESSENTRY32
> mov ebx,valCurrentPiD
> 
> cmp ebx,\[esi\].th32ProcessID
> jne nope1
> 
> nope1:
> invoke Process32Next,hSnapShot,addr proces
> 
> lea esi,offset proces
> assume esi:ptr PROCESSENTRY32
> 
> mov ebx,valCurrentPiD
> cmp ebx,\[esi\].th32ProcessID
> jne nope1
> 
> push \[esi\].th32ParentProcessID
> pop valParentPiD
> invoke CloseHandle,hSnapShot
> 
> ; 再次建立快照
> invoke CreateToolhelp32Snapshot, TH32CS\_SNAPPROCESS,NULL
> mov hSnapShot,eax
> 
> mov \[esi\].dwSize,sizeof PROCESSENTRY32
> 
> ; 开始第二轮搜索
> ; 使用 valParentPiD 查找当前进程的父进程
> invoke Process32First,hSnapShot,addr proces
> 
> lea esi,offset proces
> assume esi:ptr PROCESSENTRY32
> mov ebx,valParentPiD
> 
> cmp ebx,\[esi\].th32ProcessID
> jne nope2
> 
> nope2:
> 
> invoke Process32Next,hSnapShot,addr proces
> lea esi,offset proces
> assume esi:ptr PROCESSENTRY32
> mov ebx,valParentPiD
> cmp ebx,\[esi\].th32ProcessID
> jne nope2
> 
> ; 从完整路径中提取出文件名
> lea eax, \[esi\].szExeFile
> push eax
> invoke lstrlen,eax
> sub eax,11
> pop ebx
> add ebx,eax
> 
> ; 把文件名变成大写，与"OLLYDBG.EXE"比较 
> invoke CharUpper,ebx
> invoke lstrcmp,ebx,addr strOllyDbg
> 
> .IF eax==0
> invoke MessageBox,0,addr strFound,addr strCaption,0
> .ELSE
> invoke MessageBox,0,addr strNotFound,addr strCaption,0
> .ENDIF
> 
> invoke CloseHandle,hSnapShot
> invoke ExitProcess,0
> 
> end start
> 
> ```
> 
>   

  
方法三：SetUnhandledExceptionFilter  
  
（译注：关于 SEH 在 Hume 的<<SEH in Asm>>一文中有经典论述，在此省略，仅列出代码，具体细节请参考英文原文和<<SEH in Asm>>一文）  

> ```
> 代码:
> .586
> .model flat, stdcall
> option casemap:none
> 
> include d:\\masm32\\INCLUDE\\Windows.inc
> include d:\\masm32\\INCLUDE\\user32.inc
> include d:\\masm32\\INCLUDE\\kernel32.inc
> includelib d:\\masm32\\lib\\user32.lib
> includelib d:\\masm32\\lib\\kernel32.lib
> 
> .data
> strCaption db "OllyDbg Detector",0
> strNotFound db "OllyDbg NOT found!",0
> 
> .code
> ExcpHandler proc
> mov eax, dword ptr \[esp+4\] ; eax = EXCEPTION\_POINTERS 
> mov eax, \[eax+4\] ; eax = CONTEXT 
> 
> assume eax:ptr CONTEXT
> mov \[eax\].regEip, offset safe\_address ; Change regEip
> pushad 
> invoke MessageBox,0,addr strNotFound,addr strCaption,0
> popad
> xor eax, eax ;\\
> ; ) Set EXCEPTION\_CONTINUE\_EXECUTION
> dec eax ;/
> 
> retn 4 ; Normalize stack and return
> ExcpHandler endp
> 
> start: 
> invoke SetUnhandledExceptionFilter,offset ExcpHandler
> 
> mov ebx,dword ptr \[0FFFFFFFFh\] ;Exception is here!
> 
> safe\_address:
> invoke ExitProcess,0
> 
> end start 
> 
> ```
> 
>   

  
方法四：API 重定位  
  
这是我自己发现的一种方法，它基于 OllyDbg 调用 API 函数时的处理方法。当被调试的程序调用 API 函数时，Oleh Yuschuk（OllyDbg 的作者）在他的调试器中使用 API 重定位方法来处理。下面看一个例子：  
  
0401000 >PUSH ASD.00403033 ; /FileName = "kernel32.dll"  
00401005 CALL ; \\LoadLibraryA  
|  
|  
'--> 0040105C JMP DWORD PTR DS:\[402004\] ; JMP DWORD PTR DS:\[<&KERNEL32.LoadLibraryA>\]  
|  
|  
'--> 87FF4120 PUSH BFF776D0 ; PUSH KERNEL32.LoadLibraryA  
87FF4125 JMP KERNEL32.BFF957CA   
  
看上去他手动装载了输入表并且手动填充了 IAT，所有的 API 地址都被重定到位一个分配好的缓冲区里。在这个缓冲区中他用一种奇怪的方式调用函数。但是对我们来说这里有一个更重要的问题：他同样模仿了 GetProcAddress 函数，因此这个函数返回的是重定位以后的地址。举个例子来说，IsDebuggerPresent 在内存中的实际地址为 BFF946F6，但是这个函数返回的是重定位以后的地址 87FF4110。我们怎样用这个来实现 OllyDbg 检测呢？嗯，有很多方法，下面是最简短的一种：  
  
1.)加载 kernel32.dll 库，这时将会返回它的基地址（它加载到内存的位置）  
2.)调用 GetProcAddress 函数得到 ExitProcess 的地址  
3.)将上面的返回值（用 GetProcAddress 得到的 ExitProcess 的地址）和 kernel32.dll 的基地址比较  
4.)如果这个返回值（第 2 步所得）大于基地址（第 1 步所得），我们可以肯定这个 API 函数是被直接调用的，否则这个 API 就是被间接调用。  
  
示例程序：  

> ```
> 代码:
> .586
> .model flat, stdcall
> option casemap:none
> 
> include d:\\masm32\\INCLUDE\\Windows.inc
> include d:\\masm32\\INCLUDE\\user32.inc
> include d:\\masm32\\INCLUDE\\kernel32.inc
> includelib d:\\masm32\\lib\\user32.lib
> includelib d:\\masm32\\lib\\kernel32.lib
> 
> .data
> strCaption db "OllyDbg Detector",0
> strFound db "OllyDbg found!",0
> strNotFound db "OllyDbg NOT found!",0
> strLibrary db "kernel32.dll",0
> strFunction db "ExitProcess",0
> 
> .code
> start: 
> invoke LoadLibrary,addr strLibrary
> push eax ; EAX 为 kernel32.dll 的基地址
> 
> invoke GetProcAddress,eax,addr strFunction
> ; eax 为 ExitProcess 的地址或者重定位以后的地址
> 
> pop ebx ; EBX 为 kernel32.dll 的基地址 
> cmp eax,ebx ; 是否 ExitProcess 的地址 < Kernel32.dll 的基地址 
>  
> jl Olly\_Detected
> invoke MessageBox,0,addr strNotFound,addr strCaption,NULL
> invoke ExitProcess,0
> 
> Olly\_Detected:
> invoke MessageBox,0,addr strFound,addr strCaption,NULL
> invoke ExitProcess,0
> 
> end start 
> 
> ```
> 
>   

  
结束语：  
  
希望您喜欢这篇文章。祝您好运，Pumqara。  
  
译者菜评：  
  
这篇文章提出了四种检测 OllyDbg 的方法，其中第一种应该是最简单最常见的，第三种方法用 SEH 在看雪的书中也有涉及，但第二种和第四种方法非常新颖，而且只有当程序正被调试的时候才会检测到，这样也更“人性化”一点，不像 FindWindow 或者传统的遍历进程一样，发现有调试器不管人家在干什么一律拉出去毙了:D。  
本来挺通顺的文章，经我折腾后怎么读都别扭，那些进程线程什么的本来我就搅不清，一翻译更乱了，哪位能帮忙改下。看来以后还要好好学习，提高水平。感谢 linhanshi 兄提供这篇文章，感谢您耐心把它读完。:)  

  
 4dd466d0e1e0cc123afbe504da2462ca