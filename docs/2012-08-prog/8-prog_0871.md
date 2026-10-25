
# 打造属于自己的任务管理器

> 原文：[`www.pediy.com/kssd/pediy10/102414.html`](https://www.pediy.com/kssd/pediy10/102414.html)

【文章标题】: 打造属于自己的任务管理器  
【文章作者】: SniperChan  
【作者邮箱】: SniperChan@126.com  
【编写语言】: VC  
【使用工具】: VS.NET 2008 SP1  
【操作平台】: XP SP3  
【作者声明】: 软件可以任意修改和传播

想想自己在看雪也混了有一些日子了,再看看自己的 ID 还是个临时会员,所以想写一些技术性的文章来申请个邀请码.本人无才,破解刚接触难以写出一篇能收买版主的文章.破文就写不来了,不过自己也学过编程一段时间了,那就写一些关于编程的吧,希望以下这段乱码能博君一笑.

任务管理器是在 windows 系统用得最频繁的一个软件之一吧.你是否会觉得任务管理器的功能过于简单,不能满足你的需要呢?那好,就让我们一起来动手打造一个更强大的任务管理器.

我就觉得任务管理器的网络项的功能不够,如果能显示实时的上传,下载速度就好了.这样我们就可以随时关注自己的网络状态.虽然显示实时速度的软件很多,但是每次查看的时候都要找出来打开,不够方便,加在任务管理器就方便调出来查看了,并且它本身已经有了曲线显示了.好,开始吧!

最终效果图:

![](img/4f46205a6c9f1ffe862c1e1732795c8c.jpg)

思路:

怎么样才能在任务管理器中插入两列速度呢?我认为有下面两种方法:

1. PEDIY 技术,对原来的任务管理器进行修改扩充,增加一定的代码以增加原程序的功能.
2. 通过 DLL 注入的方法,把已编写好的 DLL 注入到原程序中已实现想要的功能.

对于第一种方法,修改了原来的程序,不方便切换为没修改时的状态.第二种就可以很方便的进行加载和卸载 DLL,并且对原程序不需要做任何修改,不会破坏到原来的程序.下面就讨论第二种方法.

步骤:

一,远程注入 DLL;
二,编写 DLL 实现相关功能;

## 一,远程注入 DLL

在微软的 Windows 中,每个进程都能获得自身的私有地址空间.当使用指针引用内存时,指针值将引用自身进程地址空间中的一个内存地址.进程不能创建一个引用属于其他进程的内存指针.

进程不能创建一个引用属于其他进程的内存指针,否则就会出现内存的错误,该错误不会影响到其他进程所使用的内存.那如何才能将自己编写的 DLL 放到任务管理器的进程空间中呢?通过远程注入!远程注入技术需要满足一个条件就是要求目标进程中的一个线程调用 `LoadLibrary` 函数来加载所需要的 DLL.由于除了自己进程的线程中之外,我们不能简单的控制其他进程中的线程,因此该解决方法要求我们在目标进程中创建一个新线程,这样我们就可以控制它执行任何代码.Windows 提供了一个 `CreateRomoteThread` 的函数可以简单地在另一个进程中创建线程.

代码:

```cpp
HANDLE CreateRomoteThread(
                HANDLE hProcess                            //目标进程句柄
        PSECURITY_ATTRIBUTES psa,                     //指向 SECURITY_ATTRIBUTES 型态的结构的指针。在 Windows 98 中忽略该参数。在 Windows NT 中，它被设为 NULL，表示使用缺省值。
        DWORD dwStackSize,                          //线程堆栈大小，一般=0，在任何情况下，Windows 根据需要动态延长堆栈的大小。
        PTHREAD_START_ROUTINE pfnstartAddr,    //指向线程函数的指针，形式：@函数名，函数名称没有限制，但是必须以下列形式声明：DWORD WINAPI ThreadProc (LPVOID pParam) ,格式不正确将无法调用成功。 
        PVOID pvParam,                                   //向线程函数传递的参数，是一个指向结构的指针，不需传递参数时，为 NULL。
        DWORD fdwCreate,                                //线程标志
        PDWORD pdwThreadId                               //保存新线程的 id。
        );
```

返回值：

函数成功，返回线程句柄；函数失败返回 `false`。

现在已经知道了如何在另一个进程中创建线程了,但是如何让该线程来加载 DLL 呢?答案很简单:让线程去调用 `LoadLibrary` 函数:

代码:

```cpp
HANDLE LoadLibrary(PCTSTR pszLibFile)
```

`pszLibFile`:指定要载入的动态链接库的名称

返回值：

成功则返回库模块的句柄，零表示失败;

归结起来就应该是如下显示的一行代码:

代码:

```cpp
HANDLE hThreadd = CreateRemoteThread(hProcessRemote,NULL,0,LoadLibrary,"E:\\MyLib.dll",0,NULL);
```

`MyLib.dll` 就是下面要编写的 DLL.

可是还它不能达到预期的目的,里面还存在两个问题.

1. 如果目标程序并没有使用到 `LoadLibrary` 函数,生成的 exe 的导入表就没有该函数,如果直接这样使用一些不可以预测的东西,可能会导致访问为例等错误.为强制使用 `LoadLibrary` 函数,必须通过调用 `GetProcAddress` 得到该函数的确切内存位置.

`LoadLibrary` 函数在 `Kernel32.dll` 中,函数应该变成如下:

代码:

```cpp
PTHREAD_START_ROUTINE pfnThreadRtn =(PTHREAD_START_ROUTINE )GetProcAddress(GetModuleHandle(TEXT("Kernel32")),"LoadLibrary");

HANDLE hThread = CreateRemoteThread(hProcessRemote,NULL,0,pfnThreadRtn ,"E:\\MyLib.dll",0,NULL);
```

2. 由于字符串"E:\\MyLib.dll"是在调用者的进程空间里,当 `LoadLibrary` 时并不能使用其他进程空间的内存地址,将会出现上述的访问违例.所以还要将 DLL 的路径名字字符串放进远程的地址空间中.当调用 `CreateRemoteThread` 时,需要将所放置字符串的地址(相对远程进程)传递给它.同样 Windows 提供了一个函数,即 `VirtualAllocEx`,他允许一个进程分配另一个进程的地址空间中的内存:

代码:

```cpp
LPVOID VirtualAllocEx( 
    HANDLE hProcess, //申请内存所在的进程句柄。
    LPVOID lpAddress, //保留页面的内存地址；一般用 NULL 自动分配 。
    SIZE_T dwSize, //欲分配的内存大小，字节单位；注意实际分配的内存大小是页内存大小的整数倍
    DWORD flAllocationType, //可取下列值 MEM_PHYSICAL ,MEM_RESERVE,MEM_RESET ,MEM_TOP_DOWN,MEM_WRITE_WATCH
    DWORD flProtect //可取下列值 PAGE_READONLY,PAGE_EXECUTE,PAGE_EXECUTE_READ ,PAGE_EXECUTE_READWRITE,PAGE_GUARD,PAGE_NOACCESS,PAGE_NOCACHE
    ); 
```

另外一个函数允许释放该内存:

代码:

```cpp
BOOL VirtualFreeEx(
    HANDLE hProcess, // 要释放内存所在进程的句柄
    LPVOID lpAddress, // 区域地址
    DWORD dwSize, // 区域大小，字节
    DWORD dwFreeType //类型
    );
```

分配好了内存,现在需要从进程的地址空间复制到远程进程的地址空间中去.

代码:

```cpp
BOOL ReadProcessMemory( 
    HANDLE hProcess,// 远程进程句柄
    LPCVOID lpBaseAddress, //远程进程中内存地址
    LPVOID lpBuffer, //本地进程中内存地址. 函数将读取的内容写入此处
    DWORD nSize,// 要传送的字节数
    LPDWORD lpNumberOfBytesRead //实际传送的字节数. 函数返回时报告实际写入多少
    );

BOOL WriteProcessMemory( 
    HANDLE hProcess, 
    LPVOID lpBaseAddress, 
    LPVOID lpBuffer, 
    DWORD nSize, 
    LPDWORD lpNumberOfBytesWritten 
    );
```

到此已经解决了需要了解工作,下面来总结一下操作步骤:

1) 使用 `VirtualAllocEx` 函数分配远程的地址空间中的内存.
2) 使用 `WriteProcessMemory` 函数将 DLL 的路径名复制到步骤 1)所分配的内测中.
3) 使用 `GetProcAddress` 函数得到 `LoadLibrary` 函数的实际地址(在 `Kernel32.dll` 中).
4) 使用 `CreateRemoteThread` 函数在远程进程中创建一个线程,调用正确的 `LoadLibrary` 函数,将步骤 1)所分配的内存地址传递给他.

这时,DLL 已经注入到了远程进程的地址空间中,DLL 的 `DllMain` 函数接收到一个 `DLL_PROCESS_ATTACH` 通知,并且可以执行所需要执行的代码了.

具体代码如下:

```cpp
BOOL Inject(DWORD dwProcessId/*进程 ID*/, PCWSTR pszLibFile/*DLL 路径和名称*/)
{
  BOOL bOk = FALSE; 
  HANDLE hProcess = NULL, hThread = NULL;
  PWSTR pszLibFileRemote = NULL;

  __try {
    // 获取目标进程的句柄
    hProcess = OpenProcess(
      PROCESS_QUERY_INFORMATION |   // Required by Alpha
      PROCESS_CREATE_THREAD     |   // For CreateRemoteThread
      PROCESS_VM_OPERATION      |   // For VirtualAllocEx/VirtualFreeEx
      PROCESS_VM_WRITE,          // For WriteProcessMemory
      FALSE, dwProcessId);
    if (hProcess == NULL) __leave;

    // 计算 DLL 路径的长度
    int cch = 1 + lstrlenW(pszLibFile);
    int cb  = cch * sizeof(wchar_t);

    //在远程进程为 DLL 的名字和路径分配内存
    pszLibFileRemote = (PWSTR) 
      VirtualAllocEx(hProcess, NULL, cb, MEM_COMMIT, PAGE_READWRITE);
    if (pszLibFileRemote == NULL) __leave;

    //把路径复制的远程进程的内存中
    if (!WriteProcessMemory(hProcess, pszLibFileRemote, 
      (PVOID) pszLibFile, cb, NULL)) __leave;

    //获取 LoadLibraryW 在 Kernel32.dll 中的实际内存地址
    PTHREAD_START_ROUTINE pfnThreadRtn = (PTHREAD_START_ROUTINE)
      GetProcAddress(GetModuleHandle(TEXT("Kernel32")), "LoadLibraryW");
    if (pfnThreadRtn == NULL) __leave;


    //创建远程线程
    hThread = CreateRemoteThread(hProcess, NULL, 0, 
      pfnThreadRtn, pszLibFileRemote, 0, NULL);
    if (hThread == NULL) __leave;

    // 等待线程结束
    WaitForSingleObject(hThread, INFINITE);

    bOk = TRUE; 
  }
  __finally { 

    // 释放内存
    if (pszLibFileRemote != NULL) 
      VirtualFreeEx(hProcess, pszLibFileRemote, 0, MEM_RELEASE);

    if (hThread  != NULL) 
      CloseHandle(hThread);

    if (hProcess != NULL) 
      CloseHandle(hProcess);
  }

  return(bOk);
}
```

## 二,编写 DLL 实现相关功能

好了,DLL 就注入到了目标进程了,下面就开始编写 DLL 来实现想要的功能了.

我想要在任务管理器的这一个控件后面加上两列,该如何做呢?

![](img/0d399a5ac743d84277d728e29a6c2e39.jpg)

要想在这个控件中添加两列就必须要知道这个控件的句柄,通过 `EnumChildWindows` 函数可以枚举父窗口的所有子窗口.这个控件也在其中.

代码:

```cpp
BOOL EnumChildWindows(HWND hWndParent,WNDENUMPROC lpEnumFunc, LPARAM lParam);
    HWND hWndParent, //父窗口句柄
    WNDENUMPROC lpEnumFunc, // 回调函数的地址
    LPARAM lParam//你自已定义的参数
```

通过这个函数可以看出,也必须要知道父窗体的句柄,很简单,通过 `FindWindow` 就可以得到任务管理器的句柄了.

代码:

```cpp
HANDLE hwndTaskManager = FindWindow( (LPCTSTR)32770, L"Windows 任务管理器");
```

`FindWindow` 第一个参数是类名,第二个参数是窗体的标题.第一个参数 32770 是什么,从何而来的?通过 vs 自带的一个工具 SPY++可以查看到:

![](img/4db870be7a29f8cf01fd57fa18efd9ca.jpg)

看到了吧,32770 就是任务管理器的类名,也是一般对话框的类名.

前面说到,既然是枚举所有控件,我怎么知道那个控件的句柄才是我要找的控件句柄呢?也是通过 SPY++可以找到该控件的 ID:

![](img/25c9d1baffc5dad4a9b931054888e51f.jpg)

控件 ID: `00000A28` ;

`EnumChildWindows` 第二个参数是回调的函数,所以写一个`\_EnumChildProc` 函数来比较该控件是否是想要的控件,最后一个参数是一个自定义参数,所以我们需要定义一个结构来保存获取到句柄和需要查找的控件 ID,传递给 `EnumChildProc`;

代码:



### 查找任务管理器控件

```c
struct StructFindTaskManagerDlgItem
{   
  DWORD  itemID;//控件 ID
  HWND  hwnd;//该控件的句柄
};

BOOL CALLBACK _EnumChildProc( HWND hwnd, LPARAM lParam )
{
  StructFindTaskManagerDlgItem* pParam = (StructFindTaskManagerDlgItem*)lParam;

  if ( (DWORD)GetDlgCtrlID( hwnd ) == pParam->itemID )//判断是否为需要的控件
  {
    pParam->hwnd = hwnd;

    return FALSE;
  }

  return TRUE;
}


HWND FindTaskManagerDlgItem( DWORD itemID )
{
  StructFindTaskManagerDlgItem param;

  param.itemID = (DWORD)itemID;
  param.hwnd = NULL;

  EnumChildWindows(  hwndTaskManager, _EnumChildProc, (LPARAM)&param );

  return param.hwnd;
}
```

控件的句柄找到了，下一步就是在该控件中插入两个列头，具体代码如下：

### 获取列数与插入列

```c
//获取 listview 的数量
int GetListColmnCount(HWND hList)
{
  int count=0;

  WCHAR caption[MAX_PATH];

  LVCOLUMN lvc;
  lvc.mask=LVCF_TEXT;
  lvc.cchTextMax=MAX_PATH;
  lvc.pszText=caption;

  for (int i=0;i<50;i++)
  {
    ZeroMemory(caption,sizeof(caption));

    SendMessage(hList,LVM_GETCOLUMN,i,(long)&lvc);
    if (caption[0]==0&&caption[1]==0)
    {
      count=i;
      break;
    }   
  }
  return count;   
}

//向 listview 插入两列
void InsertColToNetworkList(HWND hNetworkList)//hNetworkList 即为该控件的句柄
{
  int n=GetListColmnCount(hNetworkList);//获取列数,加载所有列的后面
  // 添加列
  LV_COLUMN colmn;     // 列
  ZeroMemory(&colmn, sizeof(LV_COLUMN));

  colmn.mask = LVCF_TEXT|LVCF_WIDTH|LVCF_SUBITEM; // 风格
  colmn.fmt=LVCFMT_RIGHT;

  colmn.cx = 0x60;//宽度
  colmn.pszText = L"下载";//列名
  SendMessage(hNetworkList, LVM_INSERTCOLUMN, n, (LPARAM)&colmn);
   

  colmn.pszText = L"上传";//列名
  SendMessage(hNetworkList, LVM_INSERTCOLUMN, n+1, (LPARAM)&colmn);
}
```

列已经建好了，接下来要做的就是计算出当前的速度了。

`GetIfTable()` 可以从操作系统维护的 MIB 库中读出本机各个接口的当前信息，如接口数目、类型、速率、物理地址、接收/发送字节数、错语字节数等。

### 读取接口信息

```c
DWORD GetIfTable(
  __out    PMIB_IFTABLE pIfTable,//指向 PMIB_IFTABLE 的指针
  __inout  PULONG pdwSize,//pIfTable 的大小
  __in     BOOL bOrder//是否排序
);

typedef struct _MIB_IFTABLE {
  DWORD     dwNumEntries;
  MIB_IFROW table[ANY_SIZE];
}MIB_IFTABLE, *PMIB_IFTABLE;

typedef struct _MIB_IFROW {
  WCHAR wszName[MAX_INTERFACE_NAME_LEN];
  DWORD dwIndex;//端口索引号 
  DWORD dwType;//端口类型
  DWORD dwMtu;//最大传输包字节数
  DWORD dwSpeed;//端口速度
  DWORD dwPhysAddrLen;//物理地址长度
  BYTE  bPhysAddr[MAXLEN_PHYSADDR];//物理地址
  DWORD dwAdminStatus;//管理状态
  DWORD dwOperStatus;//操作状态
  DWORD dwLastChange;//上次状态更新时间 
  DWORD dwInOctets;//输入字节数
  DWORD dwInUcastPkts;//输入非广播包数
  DWORD dwInNUcastPkts;//输入广播包数
  DWORD dwInDiscards;//输入包丢弃数
  DWORD dwInErrors;//输入包错误数
  DWORD dwInUnknownProtos;//输入未知协议包数
  DWORD dwOutOctets;//输出字节数
  DWORD dwOutUcastPkts;//输出非广播包数 
  DWORD dwOutNUcastPkts;//输出广播包数
  DWORD dwOutDiscards;//输出包丢弃数
  DWORD dwOutErrors;//输出包错误数
  DWORD dwOutQLen;//输出队长   
  DWORD dwDescrLen;//端口描述长度
  BYTE  bDescr[MAXLEN_IFDESCR]; //端口描述
}MIB_IFROW, *PMIB_IFROW;
```

通过 `MIB_IFROW` 可以看出，里面已经包含了我们需要的信息 `dwInOctets`、`dwOutOctets` 就分别是下载和上传的字节数了，它是累积量，我们可以通过计算获得每秒的字节数。

当前下载速率 = (当次输入的字节数 - 上次输入的字节数) / 1 秒。

当前上传速率 = (当次输出的字节数 - 上次输出的字节数) / 1 秒。

具体代码如下：

### 计算网络速率

```c
int NetInformation() 
{  
  MIB_IFTABLE *pIfTable = NULL;  
  MIB_IFROW *pIfRow=NULL;  
  ULONG dwSize = 0;  

  DWORD dwRet; 

  dwRet = GetIfTable( pIfTable, &dwSize, TRUE ); //第一次调用获取结构大小
  if ( dwRet == ERROR_INSUFFICIENT_BUFFER ) 
  {  
    pIfTable = ( MIB_IFTABLE * ) new char[dwSize]; 

    if ( pIfTable != NULL ) 
    {  
      dwRet = GetIfTable( pIfTable, &dwSize, TRUE ); //获得相关信息
      if ( dwRet == NO_ERROR ) 
      {  
        pIfRow = (MIB_IFROW *) & pIfTable->table[1];
        CurrentInBytes=pIfRow->dwInOctets;           //保存当次的输入字节数
        CurrentOutBytes=pIfRow->dwOutOctets;         //保存当次的输出字节数
      }  
      else 
      {  
        printf( "Some error occured!\n" );  
        return FALSE;  
      }  
    }  
    else 
    {  
      printf( "Memory allocate failue\n" );  
      return FALSE;  
    }  
  }  
  else 
  {  
    printf( "Some error occured!\n" );  
    return FALSE;  
  }  
  return TRUE;  
}  
```

### 创建线程更新界面

下面是创建一个线程，每秒读取一次，然后更新到界面上。

```c
CreateThread(NULL,NULL,ThreadTimerProc,NULL,NULL,NULL);

DWORD WINAPI  ThreadTimerProc(PVOID param)
{
  NetInformation();

  lastInBytes=CurrentInBytes;
  lastOutBytes=CurrentOutBytes;

  while(TRUE)
  {
    if(NetInformation())
    {
      float InBps=(float(CurrentInBytes-lastInBytes))/1024;
      float OutBps=(float(CurrentOutBytes-lastOutBytes))/1024;

      lastInBytes=CurrentInBytes;
      lastOutBytes=CurrentOutBytes;

      item.mask = LVIF_TEXT;
      item.cchTextMax = MAX_PATH;

      int iSubItem=GetListItemIndex(hwndNetworkList,L"下载");

      if (iSubItem>0)
      {
        ZeroMemory(szBPS,sizeof(szBPS));
        swprintf(szBPS,L"%0.2f KB/s",InBps);
        //swprintf(szBPS,L"%ld KB/s",CurrentInBytes);
          item.pszText=szBPS;
        item.iSubItem =iSubItem;
        ListView_SetItem(hwndNetworkList,(LPARAM)&item);

        ZeroMemory(szBPS,sizeof(szBPS));
        swprintf(szBPS,L"%0.2f KB/s",OutBps);
        //swprintf(szBPS,L"%ld KB/s",CurrentOutBytes);
        item.pszText=szBPS;
        item.iSubItem =iSubItem+1;
        ListView_SetItem(hwndNetworkList,(LPARAM)&item);
      }
    }
    Sleep(1000);
  }
}
```

### 总结

到此为止，整个程序已经完成了 90% 了，还有些就是关于一些如何判断 DLL 已经加载了，如何卸载等请看源代码，这里就不一一介绍了。这个程序还可以做得更好，比如在每个进程那个列表加上一个实时速度，这样就可以看到每个进程网络情况等，有兴趣的朋友可以自己添加完善。

如果本文有什么不对之处，希望各位指出！

本程序使用的编译器是 `vs.net2008 sp1`。

### 附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | TaskMgrAddIn.rar |
|---|---|---|
