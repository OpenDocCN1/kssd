
``` 
hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, dwPID);

//使用 Create Toolhelp32Snapshot0 API 可以获取加载到进程的模块（ DLL ）信息。

//将获取的 hSnapshot 句柄传递给Module32First(Module32NextO函数后，

//即可设置与MODULEENTRY32结构体相关的模块信息.
```

获取目标进程的句柄。

``` 
hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPID)

//使用pid获取目标进程的进程句柄
```

获取FreeLibrary API地址

``` 
 hModule = GetModuleHandle(L"kernel32.dll");

 pThreadProc = (LPTHREAD_START_ROUTINE)GetProcAddress(hModule, "FreeLibrary");

 //若要驱使 notepad 进程自己调用 FreeLibrary API ，需要先得到 FreeLibrary的地址。

 //然而代码获取的不是加载到 notepad.exe 进程中的Kernel32!FreeLibrary 地址，

// 而是加载到 EjectDl . exei 程中的Kernel32! FreeLibrary 地址。
```

在目标进程中运行线程

``` 
hThread = CreateRemoteThread(hProcess, NULL, 0, pThreadProc, me.modBaseAddr, 0, NULL);

//pThreadProc 参数是 FreeLibrary API 的地址,

//me.modBaseAddr 参数是要卸载的 DLL 的加载地址。

//将线程函数指定为 FreeLibrary 函数，并把 DLL 加载地址传递给线程参数，

//就在目称世中成功调用了 FreeLibraryO ) API 。
```

### 5、`AppInit_DLLs`

`计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows`

![](https://bbs.kanxue.com/upload/attach/202207/890008_N9YGDDKZ6ENJNPH.jpg)

填入dll文件路径

![](https://bbs.kanxue.com/upload/attach/202207/890008_6WTKH6HTTX8DS4M.jpg)

修改`LoadAppInit_DLLs`

![](https://bbs.kanxue.com/upload/attach/202207/890008_5APF4CF7EXGS5EW.jpg)

重启系统使修改生效，使用火绒剑，process explorer查看是否注入成功。可以看见已经被注入成功了。并且是注入了所有加载了`user32.dll`的进程。但是由于此dll的目标是`notepad.exe`进程，所以只要当运行这个exe之后才会有所动作。

![](https://bbs.kanxue.com/upload/attach/202207/890008_Y6W63MHXBDGQQNP.jpg)

![](https://bbs.kanxue.com/upload/attach/202207/890008_86UGH635PPXFUM2.jpg)

`myhack2.dll`的源码比较简单。主要目的为加载进程为`notepad.exe`的程序，然后隐藏并连接指定网站。

``` 
// myhack2.cpp

#include "windows.h"
#include "tchar.h"

#define DEF_CMD  L"c:\\Program Files\\Internet Explorer\\iexplore.exe" 
#define DEF_ADDR L"db2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6S2N6X3g2J5i4K6u0W2j5$3!0E0i4K6t1$3M7i4g2G2N6q4)9K6b7R3`.`.
#define DEF_DST_PROC L"notepad.exe"

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
    TCHAR szCmd[MAX_PATH]  = {0,};
    TCHAR szPath[MAX_PATH] = {0,};
    TCHAR *p = NULL;
    STARTUPINFO si = {0,};
    PROCESS_INFORMATION pi = {0,};

    si.cb = sizeof(STARTUPINFO);
    si.dwFlags = STARTF_USESHOWWINDOW;
    si.wShowWindow = SW_HIDE;

    switch( fdwReason )
    {
    case DLL_PROCESS_ATTACH : 
        if( !GetModuleFileName( NULL, szPath, MAX_PATH ) )
            break;

        if( !(p = _tcsrchr(szPath, '\\')) )
            break;

        if( _tcsicmp(p+1, DEF_DST_PROC) )
            break;

        wsprintf(szCmd, L"%s %s", DEF_CMD, DEF_ADDR);
        if( !CreateProcess(NULL, (LPTSTR)(LPCTSTR)szCmd, 
                            NULL, NULL, FALSE, 
                            NORMAL_PRIORITY_CLASS, 
                            NULL, NULL, &si, &pi) )
            break;

        if( pi.hProcess != NULL )
            CloseHandle(pi.hProcess);

        break;
    }

    return TRUE;
}
```

### 6、`SetWindowsHookEx()`

钩子过程（hook procedure）是系统调用的回调函数

安装钩子时，钩子过程需要在DLL内部，该DLL的实例句柄（instance handle）即`hMod`

线程ID如果为0，则钩子为“全局钩子”

用`SetWindowsHookEx()`设置好钩子后，在某个进程中生成指定消息时，操作系统会将相关DLL文件强制注入相应进程。

``` 
HHOOK SetWindowsHookEx(
    int idHook, // 钩子的类型，即它处理的消息类型
    HOOKPROC lpfn, //钩子子程的地址指针。如果dwThreadId参数为0
    // 或是由一个别的进程创建的线程的标识，
    // lpfn必须指向DLL中的钩子子程。
    // 除此以外，lpfn可以指向当前进程的一段钩子子程代码。
    //钩子函数的入口地址，当钩子钩到任何消息后便调用这个函数。
    HINSTANCE hMod,
    //应用程序实例的句柄。标识包含lpfn所指的子程的DLL。
    // 如果dwThreadId 标识当前进程创建的一个线程，
    // 而且子程代码位于当前进程，hMod必须为NULL。
    // 可以很简单的设定其为本应用程序的实例句柄。
    DWORD dwThreadId //与安装的钩子子程相关联的线程的标识符。
    // 如果为0，钩子子程与所有的线程关联，即为全局钩子。
);
```

#### 6.1、效果示意

首先运行`HookMain.exe`

![](https://bbs.kanxue.com/upload/attach/202207/890008_SJ68DJ5TRDS3RR2.jpg)

再运行`notepad.exe`，之后再使用查看，发现dll文件已经被注入

![](https://bbs.kanxue.com/upload/attach/202207/890008_CFTTTV7HJR7Q8Z8.jpg)

输入`q`，拆除钩子。拆除后，dll文件消失，可以正常输入

![](https://bbs.kanxue.com/upload/attach/202207/890008_QR9SUMKW4Y9M7Z5.jpg)

#### 6.2、分析源码

`HookMain.exe`主要过程为：首先加载`KeyHook.dll`文件，然后调用`HookStart()`函数开始钩取，用户输入`"q"`时，调用`HookStop()`函数终止钩取。

``` 
//HookMain.exe

#include "stdio.h"
#include "conio.h"
#include "windows.h"

#define	DEF_DLL_NAME		"KeyHook.dll"
#define	DEF_HOOKSTART		"HookStart"
#define	DEF_HOOKSTOP		"HookStop"

typedef void (*PFN_HOOKSTART)();
typedef void (*PFN_HOOKSTOP)();

void main()
{
	HMODULE			hDll = NULL;
	PFN_HOOKSTART	HookStart = NULL;
	PFN_HOOKSTOP	HookStop = NULL;
	char			ch = 0;

    // 加载KeyHook.dll
	hDll = LoadLibraryA(DEF_DLL_NAME);
    if( hDll == NULL )
    {
        printf("LoadLibrary(%s) failed!!! [%d]", DEF_DLL_NAME, GetLastError());
        return;
    }

    // 获取导出函数地址
	HookStart = (PFN_HOOKSTART)GetProcAddress(hDll, DEF_HOOKSTART);
	HookStop = (PFN_HOOKSTOP)GetProcAddress(hDll, DEF_HOOKSTOP);

    // 开始
	HookStart();

    // “q”退出
	printf("press 'q' to quit!\n");
	while( _getch() != 'q' )	;

    // 结束
	HookStop();
	
    // 卸载 KeyHook.dll 
	FreeLibrary(hDll);
}
```

`KeyHook.dll`在调用`HookStart()`时，`SetWindowsHookEx()`函数就会将`KeyboardProc()`添加到键盘钩链。

安装好键盘“钩子”后，无论哪个进程，只要发生键盘输入事件，OS 就会强制将`KeyHook.dll`注入相应进程。加载了`KeyHook.dll`的进程中，发生键盘事件时会首先调用执行`KeyboardProc`。

`KeyboardProc`函数中发生键盘输入事件时，就会比较当前进程的名称与`notepad.exe`字符串，若相同，则返回 1，终止`KeyboardProc()`函数，这意味着截获且删除消息。这样，键盘消息就不会传递到`notepad.exe`程序的消息队列。`notepad.exe`未能接收到任何键盘消息，故无法输出。

除此之外（即当前进程名称非`notepad.exe`时），执行`return CallNextHookEx(g_hHook, nCode, wParam, lParam)`，消息会被传递到另一个应用程序或钩链的另一个“钩子”函数。

``` 
// KeyHook.dll

#include "stdio.h"
#include "windows.h"

#define DEF_PROCESS_NAME		"notepad.exe"

HINSTANCE g_hInstance = NULL;
HHOOK g_hHook = NULL;
HWND g_hWnd = NULL;

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD dwReason, LPVOID lpvReserved)
{
	switch( dwReason )
	{
        case DLL_PROCESS_ATTACH:
			g_hInstance = hinstDLL;
			break;

        case DLL_PROCESS_DETACH:
			break;	
	}

	return TRUE;
}

LRESULT CALLBACK KeyboardProc(int nCode, WPARAM wParam, LPARAM lParam)
{
	char szPath[MAX_PATH] = {0,};
	char *p = NULL;

	if( nCode >= 0 )
	{
		// bit 31 : 0 => press, 1 => release
		if( !(lParam & 0x80000000) ) //释放键盘按键时
		{
			GetModuleFileNameA(NULL, szPath, MAX_PATH);
			p = strrchr(szPath, '\\');

            // 比较当前进程名称，如果是 notepad.exe 则消息不会传给应用程序
			if( !_stricmp(p + 1, DEF_PROCESS_NAME) )
				return 1;
		}
	}

    //反之，调用 CallNextHookEx() 消息传给应用程序
	return CallNextHookEx(g_hHook, nCode, wParam, lParam);
}

#ifdef __cplusplus
extern "C" {
#endif
	__declspec(dllexport) void HookStart()
	{
		g_hHook = SetWindowsHookEx(WH_KEYBOARD, KeyboardProc, g_hInstance, 0);
	}

	__declspec(dllexport) void HookStop()
	{
		if( g_hHook )
		{
			UnhookWindowsHookEx(g_hHook);
			g_hHook = NULL;
		}
	}
#ifdef __cplusplus
}
#endif
```

#### 6.3、调试方法

先调试`HookMain.exe`，OD打开该程序。

![](https://bbs.kanxue.com/upload/attach/202207/890008_N968B56743CEKE3.jpg)

查找核心代码

``` 
1、逐行跟踪调试

2、检索相关API

3、搜索相关字符串
```

由于之前打开过此程序，所以直接搜索`"press 'q' to quit!\n"`。

![](https://bbs.kanxue.com/upload/attach/202207/890008_QCSNHSFC2Y6W8S6.jpg)

![](https://bbs.kanxue.com/upload/attach/202207/890008_ZUZS6SNFW2MRGQZ.jpg)

点击后跳转到该位置，在`401000`地址处下断点。然后运行到此处。

先在`401006`地址处调用`LoadLibrary`,然后由`40104B`地址处的 CALL 指令调用`KeyHook.HookStart()`函数。

![](https://bbs.kanxue.com/upload/attach/202207/890008_YDR44NWQS6WKED6.jpg)

F7跟踪进入。

图中的代码是被加载到`HookMain.exe`进程中的`KeyHook.dll`的`HookStart()`函数。

在`100010EF`地址处可以看到 `CALL SetWindowsHookExW()` 指令，其上方`10010E8`与`100010ED`地址处的2条 `PUSH` 指令用于把 `SetWindowsHookExW()` API 的第1、2两个参数压入栈。

`SetWindowsHookExW()` API 的第一个参数（`idHook`）值为`WH_KEYBOARD(2)`，第二个参数（`lpfn`）值为`10001020`，该值即是钩子过程的地址。后面调试`KeyHook.dll`时再仔细看该地址。`HookMain.exe`的`main()`函数（`401000`）的其余代码接收到用户输入的“q”命令后终止钩取。
