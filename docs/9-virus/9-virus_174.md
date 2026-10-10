

# 通过 IDA 分析一个病毒

> 原文：[`www.pediy.com/kssd/pediy12/99645.html`](https://www.pediy.com/kssd/pediy12/99645.html)

在看雪注册了好久了，一直处于潜水阶段，很多时候看到好贴子，想下载附件学习。苦于临时二字。  
这周末刚好闲着，就用一个菜鸟的眼光，分析一个病毒好了。希望能借此转正。如果没有成功，或许是我的能力还没有达到看雪的最低要求，继续努力。抑或是总潜水，对论坛的贡献不够。以后会改正的。  
  
  
不废话了。  
  
前两天上数据库课，由于家中的事情，耽误了很多课程，索性找到老师拷了课件，准备回去看看，课打开 U 盘，傻眼了，4 个系统、只读属性的陌生程序， wsctf.exe   EXPLORER.exe auto.exe  SysAnti.exe 看名字一个比一个邪恶，估计都不是什么好东西。  
  刚好到周末了，拿出一个 auto.exe 稍微分析一下，由于本人，也是才入看雪，菜鸟一个。有什么问题，也请您谅解，多多指出。  
  
先用 PEID 查壳。ASPack 2.12 -> Alexey Solodovnikov \[Overlay\]  
  
如果没有变形，这个 ASPack2.12 还是很好脱得。  
在 OD 里面 ESP 定律尝试一下，到达 OEP。  
PEID 再看一下，这次看到病毒是用 VC++写的。  
  
下面就先载入 IDA 大概了解一下病毒程序的流程。  
因为是新手，所以我把最近学习到的东西，都写上一点，也给自己加深一下记忆。  
  
.text:00401000 ; Input MD5   : 31166CBED8689B31956DC95FE02C8333  
.text:00401000 ; File Name   : C:\\Documents and Settings\\Administrator\\桌面\\unpack\_virus.exe  
.text:00401000 ; Format      : Portable executable for 80386 (PE)  
.text:00401000 ; Imagebase   : 400000  
.text:00401000 ; Virtual size   : 00002000 (8192.)  
.text:00401000 ; OS type         :  MS Windows  
.text:00401000 ; Application type:  Executable 32bit  
.text:00401000  
.text:00401000                 .686p  
.text:00401000                 .mmx  
.text:00401000                 .model flat  
  
.text:00401000 ; ======================================================================  
.text:00401000 ; Segment permissions: Read/Write  
.text:00401000                 assume cs:\_text  
.text:00401000             assume es:nothing, ss:nothing, ds:\_data, fs:nothing, gs:nothing  
    
  首先这是个 80386 下的 PE 格式文件，映像基址为 0x00400000，一般在 Win NT 中，这个基址是默认的。系统为 MS Windows，应用程序类型 32bit，CPU 为 686P 的指令系统，4GB 的平坦模式。这些都是最基本的，大体上，没什么说的。  
  
  下面，是汇编中，对段寄存器的设置只定义了代码段，数据段。  
  
OK 下面就是对程序部分开始进行一些简单的分析了。  
  
首先通过 GetSystemTime 得到系统时间，之后修改系统时间。（估计是为了废掉卡巴，可能是个老病毒了，毕竟学校机器都是 N 久不动得了）  
.text:0040104F                 lea     eax, \[ebp+SystemTime\]  
.text:00401052                 push    eax             ; lpSystemTime  
.text:00401053                 call    ds:GetSystemTime  
.text:00401059                 cmp     \[ebp+SystemTime.wYear\], 7D5h  
.text:0040105F                 jbe     short loc\_401071  
.text:00401061                 lea     eax, \[ebp+SystemTime\]  
.text:00401064                 mov     \[ebp+SystemTime.wYear\], 7D5h  
.text:0040106A                 push    eax             ; lpSystemTime  
.text:0040106B                 call    ds:SetSystemTime  
  
   
  
  之后是一段时间的 Sleep  
  下面发现了一个 FindWindowExA 函数，查找一个名为卡巴斯？的进行匹配。(目的昭然若揭，在这个下面还调用了一个 PostMessageA，将结束 avp.exe 进程的消息传递到消息队列中，还是为了卡巴而生)  
.text:004010A2                 push    offset szWindow ; "卡巴斯?  
.text:004010A7                 push    ebx             ; lpszClass  
.text:004010A8                 push    ebx             ; hWndChildAfter  
.text:004010A9                 push    ebx             ; hWndParent  
.text:004010AA                 call    ds:FindWindowExA  
    
    
  
  
下面又开始 Sleep  
.text:004010C2                 push    edi             ; dwMilliseconds  
.text:004010C3                 call    esi ; Sleep  
  
  
  通过 GetModuleFileNameA 获得当前文件路径，通过 GetSystemDirectoryA 获得系统路径。再往下看，会释放一些东西。 ProcName ; "LoadLibraryA"     ModuleName ; "kernel32"  
  
.text:00401100                 push    0FFh            ; nSize  
.text:00401105                 push    eax             ; lpFilename  
.text:00401106                 push    0               ; hModule  
.text:00401108                 call    ds:GetModuleFileNameA  
.text:0040110E                 lea     eax, \[ebp+sz\]  
.text:00401114                 push    0FEh            ; uSize  
.text:00401119                 push    eax             ; lpBuffer  
.text:0040111A                 call    ds:GetSystemDirectoryA  
  
  
  后面通过 lstrcpy   lstrcat   lstrlen   lstrcmp 来释放一个 del.bat 来达到运行 kowin.exe 和删除自身的目的。  
.text:0040115B                 push    offset aDel\_bat ; "del.bat"  
.text:0040116D                 push    offset aKowin\_exe ; "kowin.exe"  
  
.text:004011C5                 push    40000000h       ; dwDesiredAccess  
.text:004011CA                 push    eax             ; lpFileName  
.text:004011CB                 call    ds:CreateFileA  
  
  
.text:00401236                 push    eax             ; lpBuffer  
.text:00401237                 push    \[ebp+hObject\]   ; hFile  
.text:0040123A                 call    edi ; WriteFile  
  
  
.text:004013A2                 push    \[ebp+hObject\]   ; hObject  
.text:004013A5                 call    ds:CloseHandle  
.text:004013AB                 lea     eax, \[ebp+CmdLine\]  
.text:004013B1                 push    0               ; uCmdShow  
.text:004013B3                 push    eax             ; lpCmdLine  
.text:004013B4                 call    ds:WinExec  
  
  
把这个进程名字压入堆栈，之后怎么调用我不太知道，但根据病毒的特征，应当是插入进程，来达到，跟随系统启动自身的目的。  
.text:004014C6                 push    offset aWinlogon\_exe ; "winlogon.exe"  
  
  下面开始的创建自身服务。  
.text:0040151E                 push    offset ServiceStatus ; lpServiceStatus  
.text:00401523                 xor     eax, eax  
.text:00401525                 push    hServiceStatus  ; hServiceStatus  
.text:0040152B                 mov     ServiceStatus.dwWin32ExitCode, eax  
.text:00401530                 mov     ServiceStatus.dwCurrentState, 1  
.text:0040153A                 mov     ServiceStatus.dwCheckPoint, eax  
.text:0040153F                 mov     ServiceStatus.dwWaitHint, eax  
.text:00401544                 call    ds:SetServiceStatus  
.text:0040154A                 test    eax, eax  
.text:0040154C                 jnz     short locret\_401554  
.text:0040154E                 jmp     ds:GetLastError  
  
  
.text:00401561                 push    offset ServiceName ; "kowin"  
.text:00401566                 mov     ServiceStatus.dwServiceType, 30h  
.text:00401570                 mov     ServiceStatus.dwCurrentState, 2  
.text:0040157A                 mov     ServiceStatus.dwControlsAccepted, 3  
.text:00401584                 mov     ServiceStatus.dwWin32ExitCode, edi  
.text:0040158A                 mov     ServiceStatus.dwServiceSpecificExitCode, edi  
.text:00401590                 mov     ServiceStatus.dwCheckPoint, edi  
.text:00401596                 mov     ServiceStatus.dwWaitHint, edi  
.text:0040159C                 call    ds:RegisterServiceCtrlHandlerA  
  
  在这段代码之前，病毒还根据之前获得系统路径释放两个文件 exe 和 dll。  
  并在各个盘符下释放一个 auto.exe 和 auotrun.inf，达到感染 U 盘的目的。  
  
  下面这段，是通过 ShellExecute 来启动资源管理器，explorer.exe 进而插入其中。  
.text:00401AEC                 push    esi             ; nShowCmd  
.text:00401AED                 push    0               ; lpDirectory  
.text:00401AEF                 push    eax             ; lpParameters  
.text:00401AF0                 push    offset File     ; "explorer.exe"  
.text:00401AF5                 push    offset Operation ; "open"  
.text:00401AFA                 push    0               ; hwnd  
.text:00401AFC                 call    ds:ShellExecuteA  
  
  
  这一小段显示了在服务中，启动病毒程序的参数， -k  
.text:00401B42                 push    offset SubStr   ; "-k"  
.text:00401B47                 push    \[ebp+Str\]       ; Str  
.text:00401B4A                 call    strstr  
  
  
后面还是一大段的 Sleep，这个病毒中 Sleep 用的很多，以后遇到就不说。  
  
  
  下面使用 RegCreateKeyExA 来对注册表进行操作，由于 IDA 是静态的，所以，只能做到这样，等下用 OD，看能不能跟出来注册表的键位。原因还是自己才学习，很多东西，不懂。望见谅。  
.text:00401CD8                 push    esi             ; lpdwDisposition  
.text:00401CD9                 push    eax             ; phkResult  
.text:00401CDA                 push    esi             ; lpSecurityAttributes  
.text:00401CDB                 push    0F003Fh         ; samDesired  
.text:00401CE0                 mov     edi, ds:RegCreateKeyExA  
…..  
…..  
.text:00401D12                 push    \[ebp+hKey\]      ; hKey  
.text:00401D15                 call    ds:RegCloseKey  
  
  
  先将 eax 清零，之后通过 GetVolumeInformation 获得 C 盘的盘区信息。由于是静态的，所以这应当是个子程序，在之前的某个时刻就调用了。  
.text:00401DEA                 xor     eax, eax  
.text:00401DEC                 push    0Ah             ; nFileSystemNameSize  
.text:00401DEE                 push    eax             ; lpFileSystemNameBuffer  
.text:00401DEF                 push    eax             ; lpFileSystemFlags  
.text:00401DF0                 lea     ecx, \[ebp+VolumeSerialNumber\]  
.text:00401DF3                 push    eax             ; lpMaximumComponentLength  
.text:00401DF4                 push    ecx             ; lpVolumeSerialNumber  
.text:00401DF5                 push    0Ch             ; nVolumeNameSize  
.text:00401DF7                 push    eax             ; lpVolumeNameBuffer  
.text:00401DF8                 push    offset RootPathName ; "c:\\\\"  
.text:00401DFD                 call    ds:GetVolumeInformationA  
  
  
WriteProcessMemory  
GetProcAddress  
CreateRemoteThread  
GetProcAddress  
FreeLibrary  
之后就是这几个病毒特有函数的的使用，基本完毕。  
  
  
这样通过 IDA 走了一遍，大概知道这个病毒的流程还有动作。下面在虚拟机里面实体测试一下。  
  
  
  
通过上面的截图，基本验证之前在 IDA 里的判断，还有 System Repair Engineer 的扫描日志。  
（我个人两个浏览器 chrome，还有世界之窗，用 chrome 编辑的图片，别人看到是红叉，这里就不上图了，我把图片链接放最底下）  
  
下面就写下病毒的基本动作：  
获取 C 盘的磁盘信息，获取系统盘路径，进而向 system32 下写入一个 dll 和 一个 exe 这两个的名字是一个随机的 8 位数，至于通过磁盘信息算出的 8 位数的算法， 由于个人是新手，分析不出来。之后 dll 插入系统的 winlogon.exe 进程。  
而后，检测卡巴斯基，如果检测到了，通过修改时间过卡巴。（这里我通过 C 写了一个窗口名为卡巴斯基，进程名为 avp.exe 的东西，但病毒貌似不管这个，估计我的小程序，还是没达到卡巴的要求，呵呵）  
下面开始注入 explorer 进程，并启动这个进程。并向各个盘符下写入 auto.exe 还有 autorun.inf  
  
\[AutoRun\]  
open=auto.exe  
shellexecute=auto.exe  
shell\\Auto\\command=auto.exe  
这个是提取出来的，呵呵，很经典了。  
  
剩下的就是修该注册表，创建服务。基本过程就是这样了。  
text:00401F39 push  offset aSystemCurrentc ; "SYSTEM\\\\CurrentControlSet\\\\Services\\\\"  
注册表是这个地方遭到修改。  
  
  
下面在用 OD 动态的走一遍。由于是动态的，很多东西，我也不是很熟悉，只能说也是大概的，有一个新手菜鸟的眼光看一下，希望得到大大们的指导。  
  
004025A6  |.  53            push    ebx        ; /pModule  
004025A7  |.  FF15 50304000 call    dword ptr \[<&kernel32.GetMod>; \\GetModuleHandleA  
004025AD  |.  50            push    eax  
004025AE  |.  E8 A3F0FFFF   call    00401656     //F7 跟进程序  
004025B3  |.  8945 98       mov     dword ptr \[ebp-68\], eax  
004025B6  |.  50            push    eax                          ; /status  
004025B7  |.  FF15 E4304000 call    dword ptr \[<&msvcrt.exit>\]   ; \\exit  
  
头有点晕，估计很多细节都没有把握好，不过这个病毒特征很明显，而且动作也不复杂，流程都出来了，就不用 OD 再走了，我估计剩下的每个 call 都跟进去，应当会很神伤。  
休息一下。休息一下。![](img/e06ce445d64470eaf3496ba167dfb4ac.jpg "碌眼")  
  
还希望能获得一个邀请码，能更深入的学习。  
  
用到的 API 帮助（由于个人还处于初级菜鸟阶段，很多函数需要百度）  
FindWindowExA     [`baike.baidu.com/view/1080313.htm`](http://baike.baidu.com/view/1080313.htm)  
  
GetModuleFileNameA        [`baike.baidu.com/view/1285912.html`](http://baike.baidu.com/view/1285912.html)  
  
GetSystemDirectoryA         [`baike.baidu.com/view/1290410.htm`](http://baike.baidu.com/view/1290410.htm)  
  
lstrcpy        [`baike.baidu.com/view/1906471.htm`](http://baike.baidu.com/view/1906471.htm)  
  
PostMessageA      [`baike.baidu.com/view/1080179.htm`](http://baike.baidu.com/view/1080179.htm)  
  
lstrcat       [`baike.baidu.com/view/1906745.htm`](http://baike.baidu.com/view/1906745.htm)  
  
lstrlen      [`baike.baidu.com/view/1907117.htm`](http://baike.baidu.com/view/1907117.htm)  
  
lstrcmp   The lstrcmp function compares two character strings.   
  
CreateFileA     [`baike.baidu.com/view/1288759.htm`](http://baike.baidu.com/view/1288759.htm)  
  
WriteFile           [`baike.baidu.com/view/1295782.htm`](http://baike.baidu.com/view/1295782.htm)  
  
CloseHandle       [`baike.baidu.com/view/1288756.htm`](http://baike.baidu.com/view/1288756.htm)  
  
WinExec      [`baike.baidu.com/view/1286882.htm`](http://baike.baidu.com/view/1286882.htm)  
  
SetServiceStatus      [`msdn.microsoft.com/en-us/libr...41(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ms686241\(VS.85\).aspx)  
  
GetLastError       [`baike.baidu.com/view/1730168.htm`](http://baike.baidu.com/view/1730168.htm)  
  
RegisterServiceCtrlHandlerA         [`msdn.microsoft.com/en-us/libr...54(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ms685054\(VS.85\).aspx)  
  
CreateThread     [`baike.baidu.com/view/1191444.htm`](http://baike.baidu.com/view/1191444.htm)  
  
ShellExecuteA          [`baike.baidu.com/view/1044533.htm`](http://baike.baidu.com/view/1044533.htm)  
  
StartServiceCtrlDispatcherA       [`msdn.microsoft.com/en-us/libr...24(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ms686324\(VS.85\).aspx)  
  
RegCreateKeyExA        [`baike.baidu.com/view/1293831.htm`](http://baike.baidu.com/view/1293831.htm)  
  
RegSetValueExA   [`baike.baidu.com/view/1297625.htm`](http://baike.baidu.com/view/1297625.htm)  
  
RegCloseKey         [`baike.baidu.com/view/1293825.htm`](http://baike.baidu.com/view/1293825.htm)  
  
GetVersionExA       [`baike.baidu.com/view/2267844.html`](http://baike.baidu.com/view/2267844.html)  
  
GetVolumeInformationA   [`baike.baidu.com/view/1292120.htm`](http://baike.baidu.com/view/1292120.htm)  
  
OpenSCManagerA        [`msdn.microsoft.com/en-us/libr...23(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ms684323\(VS.85\).aspx)  
  
OpenServiceA         [`msdn.microsoft.com/en-us/libr...30(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ms684330\(VS.85\).aspx)  
  
StartServiceA      [`msdn.microsoft.com/en-us/libr...21(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ms686321\(VS.85\).aspx)  
  
CloseServiceHandle   [`msdn.microsoft.com/en-us/libr...28(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ms682028\(VS.85\).aspx)  
  
LocalAlloc        [`msdn.microsoft.com/en-us/libr...23(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/aa366723\(VS.85\).aspx)  
  
QueryServiceConfigA   [`msdn.microsoft.com/en-us/libr...32(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ms684932\(VS.85\).aspx)  
  
ChangeServiceConfigA    [`msdn.microsoft.com/en-us/libr...87(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ms681987\(VS.85\).aspx)  
  
WriteProcessMemory     [`baike.baidu.com/view/1495693.htm`](http://baike.baidu.com/view/1495693.htm)  
  
GetProcAddress     [`baike.baidu.com/view/1523523.htm`](http://baike.baidu.com/view/1523523.htm)  
  
CreateRemoteThread   [`baike.baidu.com/view/697167.html`](http://baike.baidu.com/view/697167.html)  
  
GetProcAddress    [`baike.baidu.com/view/1523523.htm`](http://baike.baidu.com/view/1523523.htm)  
  
FreeLibrary       [`baike.baidu.com/view/1285888.htm`](http://baike.baidu.com/view/1285888.htm)  
  
  
  
  
本文有关图片链接的地  
http://photo2.bababian.com/upload1/20091018/5369DFA0EC2EBA674FFD990CA8426C20.jpg  
  
http://photo2.bababian.com/upload1/20091018/85D251CCF73F8A4E29D3D7BD251B2683.jpg  
  
http://photo2.bababian.com/upload1/20091018/DDA6BB772765683156E57CCC60A2E225.jpg  
  
http://photo2.bababian.com/upload1/20091018/C22BDC2D2CC95160BD7BA58A96B7A6AC.jpg  
  
http://photo2.bababian.com/upload1/20091018/289D6E7B661162FC1B980442BA2B8AAE.jpg  
  
http://photo2.bababian.com/upload1/20091018/3782C4892A4D2FA5C858BBD466ED4284.jpg  
  
http://photo2.bababian.com/upload1/20091018/3F4F583DDAF64AA5F3FEB6003855053C.jpg  
  
  
如果帖子哪里有不合规范，请见谅，并指出，我会在看到的第一时间修改不合规范的地方的。 

从第一个病毒分析，深切的发现逆向/破解 确实是一个很需要耐心的东西。  
从昨天晚上开始，IDA，OD，VMWARE，还有 ingshi 手杀的各种工具全拿出来，对付一个在大家看来其实很普通的一个病毒。  
不过一直到今天中午才把全部的弄完，本想在用 OD 动态跟一遍的，发现，眼睛有点涨，额。  
  
下次有机会，用 OD 来分析，学校中的另外的几个病毒好了。  
躺一会先。