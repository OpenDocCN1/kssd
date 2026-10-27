

# 如何在NP下读写游戏内存及如何进入NP进程

---

标题：【原创】如何在NP下读写游戏内存及如何进入NP进程
作者：堕落天才
日期：2007年1月4号
版权声明：请保持文章的完整，转载请注明出处

---

在上一篇文章《反NP监视原理》中说到要去掉NP的注入是很容易的事，但是去掉`npggNT.des`并不是说我们想对游戏怎么样都可以了，NP还挂钩了很多内核函数，所以很多关键系统函数就算我们在用户层能用也对游戏没有什么效果。

如果我们想在不破解NP前提下读写游戏内存该怎么办呢，我想办法至少有两个

## 一、用驱动

在驱动下读写游戏内存是没问题，但是由于我不懂驱动，所以也没什么可说。

## 二、进入游戏进程

在用户层，如果我们想在不破解NP的前提下读写游戏内存的话，大概就只能进入游戏进程了。因为很简单，我们的程序无法对游戏使用`OpenProcess`、`ReadProcessMemory`及`WriteProcessMemory`这些函数（就算是去掉了NP监视模块`npggNT.des`），而NP又不可能限制游戏自身使用这些函数，所以只要我们能够进入游戏进程就能够读写游戏的内存。怎么进入游戏呢？下面介绍两种方法：

### 1. 最简单的办法 — 全局消息钩子(`WH_GETMESSAGE`)

看似很复杂的东西原来很简单就可以实现，大道至易啊。使用消息钩子进入游戏进程无疑是最简单的一种方法，具体编程大概象这样：一个消息钩子的DLL，里面包含一个消息回调函数（什么都不用做），读写内存过程，跟主程序通讯过程或操作界面过程，当然在`DLL_PROCESS_ATTACH`要判断当前的进程是不是游戏的，是的话就做相应的处理；一个安装全局消息钩子的主程序。大概这样就可以了。使用全局消息钩子的好处是简单易用，但是不足之处是要在游戏完全启动（NP当然也启动啦）后才能进入，如果想在NP启动前做一些什么事的话是不可能的。

另外也简单介绍一下防全局钩子的办法，Windows是通过调用`LoadLibraryExW`来向目标进程注入钩子DLL的，所以只要我们在钩子安装前挂钩了这个函数，全局钩子就干扰不了了。

### 2. 更麻烦的办法 — 远程注入

知道远程注入方法和原理的人可能会说“有没有搞错，`OpenProcess`、`WriteProcessMemory`这些必备函数都不能用，怎么注入？”，当然啦，NP启动后是不能干这些事情，所以我们要在NP启动前完成。这样一来，时机就很重要了。

游戏启动的流程大概是这样：游戏`Main`->`GameGuard.des`->`GameMon.des`（NP进程）。这里的做法是这样：游戏`Main`->`GameGuard.des`(暂停)－>注入DLL－>`GameGuard.des`(继续)->`GameMon.des`。关键点就是让`GameGuard.des`暂停，有什么办法？我想到一个是全局消息钩子(还是少不了它啊)。要实现大概需要做下面的工作：一个全局消息钩子DLL,里面只要一个消息回调函数（什么都不用做），`DLL_PROCESS_ATTACH`下进行当前进程判断找`GameGuard.des`,找到的话就向主程序`SendMessage`;主程序，负责安装钩子，接收钩子DLL发来的消息，接收到消息就开始查找游戏进程，向游戏进程注入内存操作DLL，返回给`SendMessage`让`GameGuard.des`继续，卸载钩子（免得它继续钩来钩去）；内存操作DLL，负责对游戏内存进行操作。

具体编写如下（有省略）:

```
////////////////////////////////////////////////GameHook.cpp//////////////////////////////////////////////////////////////////

BOOL IsGameGuard();

//////////////////////////////////

LRESULT CALLBACK GetMsgProc(int nCode,WPARAM wParam,LPARAM lParam)
{
        return (CallNextHookEx(m_hHook,nCode,wParam,lParam));//什么都不需要做
}

///////////////////////////////////////
BOOL WINAPI DllMain(HINSTANCE hInst,DWORD dwReason,LPVOID lp)
{
        switch(dwReason){
        case DLL_PROCESS_ATTACH:
                if(IsGameGuard())//判断当前进程是不是GameGuard.des
                   SendMessage(m_hwndRecv,WM_HOOK_IN_GAMEGUARD,NULL,NULL);//向主窗体发送消息，SendMessage是等待接受窗体处理完毕才返回的，
                break;                               //所以进程就暂停在这里，我们有足够的时间去做事情
        case DLL_PROCESS_DETACH:
                break;
        }
        return TRUE;
}

///////////////////////////////////
GAMEHOOKAPI BOOL SetGameHook(BOOL fInstall,HWND hwnd)
{
        ...
}

////////////////////////////////////////
BOOL IsGameGuard()
{
       TCHAR  szFileName[256];
       GetModuleFileName(NULL,szFileName,256);
       if(strstr(szFileName,"GameGuard.des")!=NULL){//这样的判断严格来说是有问题的，但实际操作也够用了。当然也可以进行更严格的判断，不过麻烦点
          return TRUE;
       }
        return FALSE;
}

//////////////////////////////////////////////////////Main////////////////////////////////////////////////////////////////////////
void OnGameGuard(WPARAM wParam,LPARAM lParam)//处理消息钩子DLL发来的消息就是上面SendMessage的那个
{
        DWORD dwProcessId=FindGameProcess(m_strGameName);//开始查找游戏进程
        if(dwProcessId==0){
                MessageBox(m_hWnd,"没有找到游戏进程","查找游戏进程",MB_OK);
                return;
        }
    if(!InjectDll(dwProcessId)){//查找到就开始注入
          MessageBox(m_hWnd,"向游戏进程注入失败",注入",MB_OK);
                return;
     }
}

/////////////////////////////////////////////////
DWORD FindGameProcess(LPCSTR szGameName)//负责查找游戏进程
{
        HANDLE hSnapshot=CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,0);
        if(hSnapshot==INVALID_HANDLE_VALUE)
                return 0;
        PROCESSENTRY32 pe={sizeof(pe)};
        DWORD dwProcessID=0;
        for(BOOL fOK=Process32First(hSnapshot,&pe);fOK;fOK=Process32Next(hSnapshot,&pe)){
                if(lstrcmpi(szGameName,pe.szExeFile)==0){
                        dwProcessID=pe.th32ProcessID;
                        break;
                }
        }
        CloseHandle(hSnapshot);
        return dwProcessID;
}

/////////////////////////////////////////////////
BOOL InjectDll(DWORD dwProcessId)//负责注入，参考自Jeffrey Richter《windows核心编程》
{
        CString strText;
        char* szLibFileRemote=NULL;
        HANDLE hProcess=OpenProcess(PROCESS_CREATE_THREAD|PROCESS_VM_OPERATION|PROCESS_VM_WRITE,FALSE,dwProcessId);
        if(hProcess==NULL){
        //        SetRecord("Open game process failed!");
                return FALSE;
        }
        int cch=lstrlen(szDll)+1;
        int cb=cch*sizeof(char);
        szLibFileRemote=(char*)VirtualAllocEx(hProcess,NULL,cb,MEM_COMMIT,PAGE_READWRITE);
        if(szLibFileRemote==NULL){
        //        SetRecord("Alloc memory to game process failed!");
                CloseHandle(hProcess);
                return FALSE;
        }
        if(!WriteProcessMemory(hProcess,(LPVOID)szLibFileRemote,(LPVOID)szDll,cb,NULL)){
        //        SetRecord("Write game process memory failed!");
                CloseHandle(hProcess);
                return FALSE;
        }
        PTHREAD_START_ROUTINE pfnThreadRtn=(PTHREAD_START_ROUTINE)
                 GetProcAddress(GetModuleHandle(TEXT("kernel32")),"LoadLibraryA");
        if(pfnThreadRtn==NULL){
        //        SetRecord("Alloc memory to game process failed!");
                CloseHandle(hProcess);
                return FALSE;
        }
        HANDLE hThread=CreateRemoteThread(hProcess,NULL,0,pfnThreadRtn, szLibFileRemote,0,NULL);
   if(!hThread)
   {
    //          SetRecord("Create remote thread failed!");
          CloseHandle(hProcess);
         return FALSE;
   }
   if(hThread!=NULL)
           CloseHandle(hThread);
   CloseHandle(hProcess);
    return TRUE;
}

///////////////////////////操作游戏内存的DLL就不贴了，大家根据不同的需要各显神通吧///////////////////////////////////////////////////
```

这种方法比一个全局消息钩子麻烦一点，但是优点是显然易见的：可以在NP启动前做事情，比如HOOK游戏函数或做游戏内存补丁。下面进入NP进程还要用到这种方法。

## 三、进入NP进程

如果我们对NP有足够的了解，想对它内存补丁一下，来做一些事情，哪又怎样才可以进入NP的进程呢?嗯，我们知道游戏启动流程是这样的游戏`Main`->`GameGuard.des`->`GameMon.des`（NP进程），其中`GameGuard.des`跟`GameMon.des`进程是游戏`Main`通过调用函数`CreateProcessA`来创建的，上面我们说到有办法在NP进程(`GameMon.des`)启动前将我们的DLL注入到游戏进程里，因此我们可以在`GameMon.des`启动前挂钩（HOOK）`CreateProcessA`，游戏创建NP进程时让NP暂停，但是游戏本来创建NP进程时就是让它先暂停的，这步我们可以省了。下面是游戏启动NP(版本900)时传递的参数

```
ApplicationName:C:\惊天动地Cabal Online\GameGuard\GameMon.des
CommandLine:\x01\x58\x6d\xae\x99\x55\x57\x5d\x49\xbe\xe4\xe1\x9b\x14\xe6\x88\x57\x68\x6d\x11\xb9\x36\x73\x38\x71\x1e\x88\x46\xa9\x97\xd4\x3a\x20\x90\x62\xae\x15\xcd\x4b\xcd\x72\x82\xbd\x75\x0a\x54\xf0\xcc\x01\xad
CreationFlags:4
Directory:
```

其中的`CommandLine`好长啊，它要传递的参数是：一个被保护进程的`pid`，两个`Event`的`Handle`，以及当前`timeGetTime`的毫秒数(感谢JTR分享)。

`CreationFlags:4`查查`winbase.h`头文件，发现`#define CREATE_SUSPENDED 0x00000004`，所以NP进程创建时就是暂停的

在我们替换的`CreateProcessA`中，先让游戏创建NP进程(由于游戏创建时NP进程本来就是暂停的，所以不用担心NP的问题)，让游戏进程暂停(`SendMessage`就可以了)，然后再向NP进程注入DLL，最后让游戏进程继续。这样我们的DLL就进入NP进程了。实现起来大概是这样子

```
BOOL
WINAPI
MyCreateProcessA(//替换原来的CreateProcessA
    LPCSTR lpApplicationName,
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
```

