
# 如何在 NP 下读写游戏内存及如何进入 NP 进程

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-454.htm`](https://www.pediy.com/kssd/pediy09/pediy09-454.htm)

> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
>
> \*标题:【原创】如何在 NP 下读写游戏内存及如何进入 NP 进程 \*
>
> \*作者:堕落天才 \*
>
> \*日期:2007 年 1 月 4 号 \*
>
> \*版权声明:请保持文章的完整，转载请注明出处 \*
>
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

在上一篇文章《反 NP 监视原理》中说到要去掉 NP 的注入是很容易的事，但是去掉 `npggNT.des` 并不是说我们想对游戏怎么样都可以了，NP 还挂钩了很多内核函数，所以很多关键系统函数就算我们在用户层能用也对游戏没有什么效果。

如果我们想在不破解 NP 前提下读写游戏内存该怎么办呢，我想办法至少有两个

### 一、用驱动

在驱动下读写游戏内存是没问题，但是由于我不懂驱动，所以也没什么可说。

### 二、进入游戏进程

在用户层，如果我们想在不破解 NP 的前提下读写游戏内存的话，大概就只能进入游戏进程了。因为很简单，我们的程序无法对游戏使用 `OpenProcess`、`ReadProcessMemory` 及 `WriteProcessMemory` 这些函数（就算是去掉了 NP 监视模块 `npggNT.des`），而 NP 又不可能限制游戏自身使用这些函数，所以只要我们能够进入游戏进程就能够读写游戏的内存。怎么进入游戏呢？下面介绍两种方法：

1. **最简单的办法 — 全局消息钩子 (`WH_GETMESSAGE`)**

   看似很复杂的东西原来很简单就可以实现，大道至易啊。使用消息钩子进入游戏进程无疑是最简单的一种方法，具体编程大概象这样:一个消息钩子的 DLL，里面包含一个消息回调函数（什么都不用做），读写内存过程，跟主程序通讯过程或操作界面过程，当然在 `DLL_PROCESS_ATTACH` 要判断当前的进程是不是游戏的，是的话就做相应的处理；一个安装全局消息钩子的主程序。大概这样就可以了。使用全局消息钩子的好处是简单易用，但是不足之处是要在游戏完全启动（NP 当然也启动啦）后才能进入，如果想在 NP 启动前做些什么事的话是不可能的。

   另外也简单介绍一下防全局钩子的办法，Windows 是通过调用 `LoadLibraryExW` 来向目标进程注入钩子 DLL 的，所以只要我们在钩子安装前挂钩了这个函数，全局钩子就干扰不了了。

2. **更麻烦的办法 — 远程注入**

   知道远程注入方法和原理的人可能会说“有没有搞错，`OpenProcess`、`WriteProcessMemory` 这些必备函数都不能用，怎么注入?",当然啦，NP 启动后是不能干这些事情，所以我们要在 NP 启动前完成。这样一来，时机就很重要了。

   游戏启动的流程大概是这样:游戏 `Main` -> `GameGuard.des` -> `GameMon.des`（NP 进程）。这里的做法是这样：游戏 `Main` -> `GameGuard.des`(暂停)－> 注入 DLL －> `GameGuard.des`(继续)-> `GameMon.des`。关键点就是让 `GameGuard.des` 暂停，有什么办法？我想到一个是全局消息钩子(还是少不了它啊)。要实现大概需要做下面的工作：一个全局消息钩子 DLL,里面只要一个消息回调函数（什么都不用做），`DLL_PROCESS_ATTACH` 下进行当前进程判断找 `GameGuard.des`,找到的话就向主程序 `SendMessage`; 主程序，负责安装钩子，接收钩子 DLL 发来的消息，接收到消息就开始查找游戏进程，向游戏进程注入内存操作 DLL，返回给 `SendMessage` 让 `GameGuard.des` 继续，卸载钩子（免得它继续钩来钩去）；内存操作 DLL，负责对游戏内存进行操作。

   具体编写如下（有省略）:

   ```cpp
   ////////////////////////////////////////////////GameHook.cpp//////////////////////////////////////////////////////////////////

   BOOL IsGameGuard();

   //////////////////////////////////

   LRESULT CALLBACK GetMsgProc(int nCode, WPARAM wParam, LPARAM lParam)
   {
       return (CallNextHookEx(m_hHook, nCode, wParam, lParam)); //什么都不需要做
   }

   ///////////////////////////////////////

   BOOL WINAPI DllMain(HINSTANCE hInst, DWORD dwReason, LPVOID lp)
   {
       switch(dwReason){
           case DLL_PROCESS_ATTACH:
               if(IsGameGuard()) //判断当前进程是不是 GameGuard.des
                   SendMessage(m_hwndRecv, WM_HOOK_IN_GAMEGUARD, NULL, NULL); //向主窗体发送消息，SendMessage 是等待接受窗体处理完毕才返回的，
               break; //所以进程就暂停在这里，我们有足够的时间去做事情
           case DLL_PROCESS_DETACH:
               break;
       }
       return TRUE;
   }

   ///////////////////////////////////

   GAMEHOOKAPI BOOL SetGameHook(BOOL fInstall, HWND hwnd)
   {
       ...
   }

   ////////////////////////////////////////

   BOOL IsGameGuard()
   {
       TCHAR szFileName[256];
       GetModuleFileName(NULL, szFileName, 256);
       if(strstr(szFileName, "GameGuard.des") != NULL){ //这样的判断严格来说是有问题的，但实际操作也够用了。当然也可以进行更严格的判断，不过麻烦点
           return TRUE;
       }
       return FALSE;
   }

   //////////////////////////////////////////////////////Main////////////////////////////////////////////////////////////////////////

   void OnGameGuard(WPARAM wParam, LPARAM lParam) //处理消息钩子 DLL 发来的消息就是上面 SendMessage 的那个
   {
       DWORD dwProcessId = FindGameProcess(m_strGameName); //开始查找游戏进程
       if(dwProcessId == 0){
           MessageBox(m_hWnd, "没有找到游戏进程", "查找游戏进程", MB_OK);
           return;
       }

       if(!InjectDll(dwProcessId)){ //查找到就开始注入
           MessageBox(m_hWnd, "向游戏进程注入失败", "注入", MB_OK);
           return;
       }
   }

   /////////////////////////////////////////////////

   DWORD FindGameProcess(LPCSTR szGameName) //负责查找游戏进程
   {
       HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
       if(hSnapshot == INVALID_HANDLE_VALUE)
           return 0;

       PROCESSENTRY32 pe = {sizeof(pe)};
       DWORD dwProcessID = 0;

       for(BOOL fOK = Process32First(hSnapshot, &pe); fOK; fOK = Process32Next(hSnapshot, &pe)){
           if(lstrcmpi(szGameName, pe.szExeFile) == 0){
               dwProcessID = pe.th32ProcessID;
               break;
           }
       }

       CloseHandle(hSnapshot);
       return dwProcessID;
   }

   /////////////////////////////////////////////////

   BOOL InjectDll(DWORD dwProcessId) //负责注入，参考自 Jeffrey Richter《windows 核心编程》
   {
       CString strText;
       char* szLibFileRemote = NULL;

       HANDLE hProcess = OpenProcess(PROCESS_CREATE_THREAD | PROCESS_VM_OPERATION | PROCESS_VM_WRITE, FALSE, dwProcessId);
       if(hProcess == NULL){
           //  SetRecord("Open game process failed!");
           return FALSE;
       }

       int cch = lstrlen(szDll) + 1;
       int cb = cch * sizeof(char);
       szLibFileRemote = (char*) VirtualAllocEx(hProcess, NULL, cb, MEM_COMMIT, PAGE_READWRITE);
       if(szLibFileRemote == NULL){
           //  SetRecord("Alloc memory to game process failed!");
           CloseHandle(hProcess);
           return FALSE;
       }

       if(!WriteProcessMemory(hProcess, (LPVOID)szLibFileRemote, (LPVOID)szDll, cb, NULL)){
           //  SetRecord("Write game process memory failed!");
           CloseHandle(hProcess);
           return FALSE;
       }

       PTHREAD_START_ROUTINE pfnThreadRtn = (PTHREAD_START_ROUTINE)
           GetProcAddress(GetModuleHandle(TEXT("kernel32")), "LoadLibraryA");
       if(pfnThreadRtn == NULL){
           //  SetRecord("Alloc memory to game process failed!");
           CloseHandle(hProcess);
           return FALSE;
       }

       HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, pfnThreadRtn, szLibFileRemote, 0, NULL);
       if(!hThread)
       {
           //    SetRecord("Create remote thread failed!");
           CloseHandle(hProcess);
           return FALSE;
       }

       if(hThread != NULL)
           CloseHandle(hThread);

       CloseHandle(hProcess);
       return TRUE;
   }

   ///////////////////////////操作游戏内存的 DLL 就不贴了，大家根据不同的需要各显神通吧///////////////////////////////////////////////////
   ```

   这种方法比一个全局消息钩子麻烦一点，但是优点是显然易见的:可以在 NP 启动前做事情，比如 HOOK 游戏函数或做游戏内存补丁。下面进入 NP 进程还要用到这种方法。

### 三、进入 NP 进程

如果我们对 NP 有足够的了解，想对它内存补丁一下，来做一些事情，哪又怎样才可以进入 NP 的进程呢? 嗯，我们知道游戏启动流程是这样的游戏 `Main` -> `GameGuard.des` -> `GameMon.des`（NP 进程），其中 `GameGuard.des` 跟 `GameMon.des` 进程是游戏 `Main` 通过调用函数 `CreateProcessA` 来创建的，上面我们说到有办法在 NP 进程(`GameMon.des`) 启动前将我们的 DLL 注入到游戏进程里，因此我们可以在 `GameMon.des` 启动前挂钩（HOOK）`CreateProcessA`，游戏创建 NP 进程时让 NP 暂停，但是游戏本来创建 NP 进程时就是让它先暂停的，这步我们可以省了。下面是游戏启动 NP(版本 900)时传递的参数

*   `ApplicationName`: `C:\惊天动地 Cabal Online\GameGuard\GameMon.des`
*   `CommandLine`: `\x01\x58\x6d\xae\x99\x55\x57\x5d\x49\xbe\xe4\xe1\x9b\x14\xe6\x88\x57\x68\x6d\x11\xb9\x36\x73\x38\x71\x1e\x88\x46\xa9\x97\xd4\x3a\x20\x90\x62\xae\x15\xcd\x4b\xcd\x72\x82\xbd\x75\x0a\x54\xf0\xcc\x01\xad`
*   `CreationFlags`: `4`
*   `Directory`:

其中的 `CommandLine` 好长啊，它要传递的参数是: 一个被保护进程的 pid，两个 `Event` 的 `Handle`，以及当前 `timeGetTime` 的毫秒数 (感谢 JTR 分享)。

`CreationFlags: 4` 查查 `winbase.h` 头文件，发现 `#define CREATE_SUSPENDED 0x00000004`，所以 NP 进程创建时就是暂停的

在我们替换的 `CreateProcessA` 中，先让游戏创建 NP 进程(由于游戏创建时 NP 进程本来就是暂停的，所以不用担心 NP 的问题)，让游戏进程暂停(`SendMessage` 就可以了)，然后再向 NP 进程注入 DLL, 最后让游戏进程继续。这样我们的 DLL 就进入 NP 进程了。实现起来大概是这样子

```cpp
BOOL WINAPI MyCreateProcessA(//替换原来的 CreateProcessA
    LPCSTR lpApplicationName,
```



## 代码示例

```cpp
    LPSTR lpCommandLine,  
    LPSECURITY_ATTRIBUTES lpProcessAttributes,  
    LPSECURITY_ATTRIBUTES lpThreadAttributes,  
    BOOL bInheritHandles,  
    DWORD dwCreationFlags,  
    LPVOID lpEnvironment,  
    LPCSTR lpCurrentDirectory,  
    LPSTARTUPINFOA lpStartupInfo,  
    LPPROCESS_INFORMATION lpProcessInformation  
    )  
{  
    UnhookCreateProcessA();  
    BOOL fRet=CreateProcessA(lpApplicationName,lpCommandLine,lpProcessAttributes,lpThreadAttributes,bInheritHandles,dwCreationFlags,  
        lpEnvironment,lpCurrentDirectory,lpStartupInfo,lpProcessInformation);   
    RehookCreateProcessA();  
        SendMessage(hwndRecv,//负责注入的窗体句柄  
                      WM_HOOK_NP_CREATE,//自定义消息  
                      (WPARAM)lpProcessInformation->dwProcessId,//把 NP 进程 ID 传给负责注入的主窗体  
                      NULL);  
    return fRet;  
}  
```

## 四、注意问题

由于我们是在不破解 NP 的前提下对游戏内存进行操作，所以一不小心的话，很容易就死游戏。NP 保护了游戏进程的代码段，所以在 NP 启动后就不要再对其代码段进行修改，要补丁或 `HOOK` 系统函数这些都要在 NP 启动前完成。当然读写游戏的数据段是没问题的，因为游戏本身也不断进行这样的操作。

感谢楼主分享

### 以前分析的一点点有限笔记

打开 `gameguard.des`，`hgg=2ec`，校验后关闭

`CreateProcess` `gameguard.des` 参数为：

1. 时间 `GetTickTime` 的后 32 位，去掉后 20 位
2. `game` `name`
3. 互斥体句柄
4. 事件句柄
5. `game` 进程 `ID`
6. `0`

`CreateProcess` `GameDon.des` 参数

1=当前时间产生
2=
3=
4=标准输入句柄,管道二 `hReadPipe2`  `2c8`
5=标准输出句柄,管道一 `hWritePipe1`  `2c4`
6=游戏 `name`
7=`hEvent3`
8=`hEvent1`
9

"其中的 `CommandLine` 好长啊，它要传递的参数是:一个被保护进程的 `pid`，两个 `Event` 的 `Handle`，以及当前 `timeGetTime` 的毫秒数"

我想问问，如果把其中的参数 `PID` 改变为你的外挂的进程 `ID` 会有什么结果？

`CommandLine` 是经过二次加密后，然后 `format` 得到的一个串
在 `GameMon.des` 中，会判断 `game` `name`，有黑名单，也有服务用户
历史来看 NP 曾经出过无数版本..
从最早的驱动反 `HOOK`..到现在的 3 分钟检测..

一次比一个变态..一次比一个可怕..

所以做为技术人员..我们深入研究 NP 的各种技术..同时发现 NP 的技术大部分依赖驱动..但由于 `WIN32` 这个笼子...NP 本身开始走向一个衰败和极端...

首先..说一下 NP 本身的一些技术..和一些漏洞
(目前除了 3 分钟检测..我几乎都能挂掉 因为 3 分钟检测属于完全内置与游戏代码中...NP 的特征库不全..有时候无法区分版本..所以挂起来很困难)...

NP 可以说有众多版本..反外挂的等级也层次不起...
其中最容易也简单的..应该是 2001~2003 版本(部分 2003 版本带 3 分钟检测..例如精灵)

用年号区分...是因为 NP 本身的技术是没有太大的长进的..只是每年适当的更新

例如..众多版本都使用 NP 的驱动挂接 `API` 函数..这个和当初所谓的检测导入表差距比较大...
我个人也模仿 NP 的一些假想技术开发过一些反外挂程序..其中对于检测到处表..可以说是非常的占资源..属于不太可能实现的一种手段..因为你要去定位..但是这个对于 `CPU` 开销太大

我也在考虑 NP 是如何检测一个程序是否是非法程序的呢?

做几个试验..启动带 NP 的游戏..之后启动 `GM8` 或 `WPE`..你会发现这些程序自动关闭...

其实要做起来也简单..`JMP` `API` 函数....当程序要读写内存,注入进程时..这个时候 `API` 已经跳到了 NP 自己的地址空间..之后 NP 只需要结束目标程序即可..(如果是 `DLL`..就释放该 `DLL`)简单 `D` 很~!

那么如何破解呢?我们开发过驱动的应该知道..启动本身需要加载之后才可以起效....之后程序结束..驱动同时卸载...(如果需要可以参考 `WINIO` 的驱动加载部分)

这个过程大概需要 1 秒前后(好机器更快)..但是如果你的外挂是全局 `HOOK`..这个时候比 NP 要快..已经自动挂接到全部的系统进程当中(前提是再游戏启动 NP 前启动外挂)...着就是 NP 最大的局限..也是 NP 无法攻克的局限..

为了兼容 NP 只有 60%左右的驱动...

其实想挂掉 NP.只要挂掉 NP 加载即可..但是 3 分钟检测如果是再程序内部被封装..这个时候你即便挂掉 NP..3 分钟检测依然会启动..游戏本身没什么用..

NP 一般是在启动程序时的一个 `CALL`..之后加载 NP..只要耐心跟踪..首先确认哪个 `CALL LAOD NP`..跟进入..再确认哪个 `CALL LOAD NP`..依次..直到彻底 `NOP` 掉 NP 为止..

所以要看 NP 的版本来确定采用什么技术来挂掉 NP~!

同时需要你的汇编非常好~!

以上是一些经验和技术理论..

### 引用

*最初由 **堕落天才**发布* [![查看帖子](img/showthread.php?p=261961#post261961)] \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\*因为很简单，我们的程序无法对游戏使用 `OpenProcess`、`ReadProcessMemoery` 及 `WriteProcessMemory` 这些函数（就算是去掉了 NP 监视模块 `npggNT.des`），而 NP 又不可能限制游戏自身使用这些函数，所以只要我们能够进入游戏进程就能够读写游戏的内存...

现在不行了．．．就算进入游戏的进程，也不能读游戏内存．．但是我在测试的过程中，`dll` 进入游戏进程后，`OpenProcess` 打开游戏进程还是成功的．．`ReadProcessMemoery` 这个函数就不成功了，貌似是因为 NP 在 r0 下做了手脚．. `ReadVirtualMemory` 实际上调用 `NtReadVirtualMemory`：

```asm
.text:7C8021CC ReadProcessMemory proc near             ; CODE XREF: GetProcessVersion+2F2FDp
.text:7C8021CC                                         ; GetProcessVersion+2F31Cp
.text:7C8021CC                                         ; GetProcessVersion+2F343p
.text:7C8021CC                                         ; Toolhelp32ReadProcessMemory+26p
.text:7C8021CC
.text:7C8021CC hProcess        = dword ptr  8
.text:7C8021CC lpBaseAddress   = dword ptr  0Ch
.text:7C8021CC lpBuffer        = dword ptr  10h
.text:7C8021CC nSize           = dword ptr  14h
.text:7C8021CC lpNumberOfBytesRead= dword ptr  18h
.text:7C8021CC
.text:7C8021CC                 mov     edi, edi
.text:7C8021CE                 push    ebp
.text:7C8021CF                 mov     ebp, esp
.text:7C8021D1                 lea     eax, [ebp+nSize]
.text:7C8021D4                 push    eax
.text:7C8021D5                 push    [ebp+nSize]
.text:7C8021D8                 push    [ebp+lpBuffer]
.text:7C8021DB                 push    [ebp+lpBaseAddress]
.text:7C8021DE                 push    [ebp+hProcess]
.text:7C8021E1                 call    ds:NtReadVirtualMemory
```

`NtReadVirtualMemory`：

```asm
NtReadVirtualMemory proc near             ; CODE XREF: LdrFindCreateProcessManifest+1CCp
.text:7C92E2BB                                         ; LdrCreateOutOfProcessImage+7Cp
.text:7C92E2BB                                         ; RtlReadOutOfProcessMemoryStream+2Ep
.text:7C92E2BB                                         ; LdrFindCreateProcessManifest+1612Fp
.text:7C92E2BB                                         ; LdrFindCreateProcessManifest+1691Ep
.text:7C92E2BB                                         ; LdrFindCreateProcessManifest+1697Bp ...
.text:7C92E2BB                 mov     eax, 0BAh       ; NtReadVirtualMemory
.text:7C92E2C0                 mov     edx, 7FFE0300h
.text:7C92E2C5                 call    dword ptr [edx]
.text:7C92E2C7                 retn    14h
.text:7C92E2C7
.text:7C92E2C7 NtReadVirtualMemory endp
```

最后他还是要进 ring0．．．不知道恢复 `ssdt`，`ReadVirtualMemory` 能否读游戏内存？![恐怖!](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg) ![恐怖!](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg) ![恐怖!](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg)

