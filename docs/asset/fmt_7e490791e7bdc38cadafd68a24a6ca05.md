
//后面介绍

```c
if (0 == InitPerfInfo())
{
    retval = FALSE;
    goto cleanup;
}
```

//创建窗口

```c
g_hMainWnd = CreateDialog(hInstance,
                          MAKEINTRESOURCE(IDD_MAINWND),
                          NULL,
                          MainWindowProc);
```

//显示窗口

```c
SetWindowPos(g_hMainWnd, NULL,
             g_Options.m_rcWindow.left,
             g_Options.m_rcWindow.top,
             cx,
             cy,
             SWP_NOZORDER);
MyShowWindow(g_hMainWnd, nCmdShow);
```

--------------------------------------------------------

下面是消息循环和恢复现场~~这里算是主函数基本上结束了~~~~~，MY GOD！两个函数写这么多，如果真要这么一步步分析下去，不死人了~~~~我以后会把主要的重点的东西贴出来给大家，源代码我已经放出来了，详细点的大家根据代码自己研究了，我不说了，打字都打的心烦起来了。

--------------------------------------------------------------------------------

研究成果的实践

--------------------------------------------------------------------------------

我们研究了不应用不就白研究了嘛！！不管木马还是病毒都要对付任务管理器，所以任务管理器实际蛮可怜的，我今天就把我的任务管理器整理一下，这样我不怕病毒感染了呵呵~~~~

我们首先为了让任务管理能够正常显示，我们要把注册表中的那一项跳掉，其次是我们把入口做处理，最起码我们知道我们的任务管理器是否被感染了。

好我们开始手动来做我们的要求：

### 第一、注册表项的处理

我现在的任务管理是如图这样，我们都知道这个是 `MS` 给保留的功能我们就把它处理掉。

![](img/fc20d0f738b8b9a3b96391c06e5d3345.jpg)

--------------------------------------------------------------------------------

```asm
//OD 载入：
01005944 > $  E8 1EC70000   call    01012067
01005949   .^ E9 0AFFFFFF   jmp     01005858
0100594E      CC            int3
//下面不就是我们的入口代码的第一个函数吗？？
01005860  |.  56            push    esi
01005861  |.  57            push    edi
01005862  |.  68 80000000   push    80                      ; /Priority = HIGH_PRIORITY_CLASS
01005867  |.  FF15 14110001 call    dword ptr [<&KERNEL32.GetCurrent>; |[GetCurrentProcess
0100586D  |.  50            push    eax                     ; |hProcess
0100586E  |.  FF15 18110001 call    dword ptr [<&KERNEL32.SetPriorit>; \\SetPriorityClass
01005922  |> \\6A 0A         push    0A
01005924  |.  58            pop     eax
01005925  |>  50            push    eax                     ; /Arg4
01005926  |.  56            push    esi                     ; |Arg3
01005927  |.  6A 00         push    0                       ; |Arg2 = 00000000
01005929  |.  6A 00         push    0                       ; |/pModule = NULL
0100592B  |.  FF15 24110001 call    dword ptr [<&KERNEL32.GetModuleH>; |\\GetModuleHandleW
01005931  |.  50            push    eax                     ; |Arg1 = 01000000
//从这里进去，就是到了 WinMain 函数了
01005932  |.  E8 57FAFFFF   call    0100538E                ; \\taskmgrX.0100538E
//看来 XP 和 2000 没什么太多的变化！
010053A9  |.  68 4C1B0001   push    01001B4C                ; /MsgName = "TaskbarCreated"
010053AE  |.  8945 FC       mov     dword ptr [ebp-4], eax  ; |
010053B1  |.  89B5 28FCFFFF mov     dword ptr [ebp-3D8], esi ; |
010053B7  |.  8935 285E0101 mov     dword ptr [1015E28], esi ; |
010053BD  |.  89BD 2CFCFFFF mov     dword ptr [ebp-3D4], edi ; |
010053C3  |.  899D 20FCFFFF mov     dword ptr [ebp-3E0], ebx ; |
010053C9  |.  FF15 A0120001 call    dword ptr [<&USER32.RegisterWind>; \\RegisterWindowMessageW
//下面这个也是没变的
//HANDLE g_hStartupMutex = NULL; //互斥句柄
//const TCHAR cszStartupMutex[] = TEXT("NTShell Taskman Startup Mutex"); //互斥标识
//上面的是 C 定义，我们要处理掉，想想如果要是有个病毒或木马有个这样的标识，那么我们的任务管理器是不是就不出来了~~~~，一会再处理它！下个 F2 断点记下地方。
010053CF  |.  68 C0140001   push    010014C0                ; /MutexName = "NTShell Taskman Startup Mutex"
010053D4  |.  57            push    edi                     ; |InitialOwner => TRUE
010053D5  |.  53            push    ebx                     ; |pSecurity => NULL
010053D6  |.  A3 105E0101   mov     dword ptr [1015E10], eax ; |
010053DB  |.  FF15 2C110001 call    dword ptr [<&KERNEL32.CreateMute>; \\CreateMutexW
//如果我们的任务管理，是一直这个名字，那么病毒或木马不就很容易找到我们的任务管理的窗口吗？
//一会把它改了。
01005437  |> \\68 04010000   push    104                     ; /Count = 104 (260.)
0100543C  |.  8D85 30FCFFFF lea     eax, dword ptr [ebp-3D0] ; |
01005442  |.  50            push    eax                     ; |Buffer
01005443  |.  68 13270000   push    2713                    ; |RsrcID = STRING "Windows 任务管理器"
01005448  |.  56            push    esi                     ; |hInst
01005449  |.  8B35 BC130001 mov     esi, dword ptr [<&USER32.LoadStr>; |USER32.LoadStringW
0100544F  |.  FFD6          call    esi                     ; \\LoadStringW
//我们到这里，发现了我们要改的地方，也就是那个保留的设置。
010054BF  |> \\8D85 24FCFFFF lea     eax, dword ptr [ebp-3DC]
010054C5  |.  50            push    eax                     ; /pHandle
010054C6  |.  68 19000200   push    20019                   ; |Access = KEY_READ
010054CB  |.  53            push    ebx                     ; |Reserved
010054CC  |.  68 D81A0001   push    01001AD8                ; |Subkey = "Software\\Microsoft\\Windows\\CurrentVersion\\Policies\\System"
010054D1  |.  68 01000080   push    80000001                ; |hKey = HKEY_CURRENT_USER
010054D6  |.  FF15 08100001 call    dword ptr [<&ADVAPI32.RegOpenKey>; \\RegOpenKeyExW
//下面是检查~~~是否为 0 如果为 9 就正常运行，如果不是就弹出图中的对话框
01005521  |.  399D 20FCFFFF cmp     dword ptr [ebp-3E0], ebx
01005527  |.  74 4B         je      short 01005574      //这里我们修改为 JMP，无论值是多少都给我正常运行~~实际这样做不好，最好把[EBP-3E0]的值改了，我们已经知道了它的工作机制，对后面的影响实际没什么，我们就可以这样跳了。熟悉的任务管理器就出现了呵呵~~~~~~~~
```

`CTRL-ALT-DEL`，怎么还是如图啊，呵呵！~~~是我修改的不是系统中的，等我修改好就把它放进系统目录去，这样我就再也不怕病毒欺负它了，无论怎么改我按这三个键都会照常蹦出来哈哈~~~

--------------------------------------------------------------------------------

### 二、互斥修改

--------------------------------------------------------------------------------

//这个简单了，我们把我们的互斥标识改进去就可以了
//再次 `OD` 我们到这里了。

```asm
010053CF  |.  68 C0140001   push    010014C0                ; /MutexName = "NTShell Taskman Startup Mutex"
010053D4  |.  57            push    edi                     ; |InitialOwner => TRUE
010053D5  |.  53            push    ebx                     ; |pSecurity => NULL
010053D6  |.  A3 105E0101   mov     dword ptr [1015E10], eax ; |
010053DB  |.  FF15 2C110001 call    dword ptr [<&KERNEL32.CreateMute>; \\CreateMutexW
```

//改成什么呢？？就改:`"<* NTShell Taskman Startup *>\"`
//这里改的时候要注意了，这里的字符是 `UN` 不是 `A` 的哦！

### 三、窗口标题的修改

--------------------------------------------------------------------------------

我们可以用 `exescope` 来修改，也可以用其它的由于它的标题在 `string table` 中，我们方便点不拿 `OD` 做了，我这里拿 `exescope` 来修改。修改为“`10003,^\_^Windows 任务管理器^\_^`”

### 四、入口修改

如果碰到像熊猫那样的破性的病毒怎么办呢！！至少我们要知道我们的任务管理器是否被感染~~

为了方便加代码~~，我们必须把入口点改掉！

----------------------------------------------------------------------------------

偏移地址：`00005944`

基地址 ：`01000000`

如果我一直这样加汇编来实现，会让程序的稳定变的差点，系统嘛！稳定很重要的！我还是写个 `DLL` 来保护我的任务管理吧！我实在是不想再修改手动加代码了~~太麻烦了！我又懒~~~~

在这期间我们可以修改 `OEP`，也可以修改入口代码总之就是让病毒没办法破就是了，我写了个简单的 `DLL`，把入口代码改了，也就是说，只要病毒改了入口代码~~我都会发现！，

我把我的保护 `DLL` 代码贴一下，简单但是实用，我没有把它做的很难，因为够用就可以了。

----------------------------------------------------------------------------------

```c
void Init()
{
  //我们是要做为模块加进去的所以我们事先就能知道我们的程序基地址:
  //获取 Taskmgr 的基地址:
  hModule =::GetModuleHandle(NULL);
  DWORD dwBaseAddr=(LONG)hModule;
  m_pData =(PDWORD)dwBaseAddr;
  _dosHeader =(PIMAGE_DOS_HEADER)m_pData;
    _ntHeader  =(PIMAGE_NT_HEADERS)((LONG)_dosHeader+(LONG)_dosHeader->e_lfanew);
  //检测入口点,
  if(_ntHeader->OptionalHeader.AddressOfEntryPoint!=0x00005944){
    TCHAR   tcCaption[MAX_PATH]={0};
```
