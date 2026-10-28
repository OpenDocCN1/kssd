
# [原创]关于Windows消息钩子的理解与测试项目（附源码）

## 前奏

近来一直在自学Windows Hook相关的知识，已经尝试多种注入方式。尤其对消息钩子方式很感兴趣，因为看到Spy++能够截获系统中绝大多数应用的消息流，就很想知道它的工作原理，打算制作属于自己的Spy++。

## 消息钩子简介

消息钩子简言之就是Windows处理消息的一个平台，用户可以在此平台获取或过滤所需应用的消息（例如某鼠标位置，键盘击下等），这些被监控的消息都会在目标窗口处理函数之前被截获。系统在拿到消息后会问你：这是不是你想要的消息？如果是，则恭喜你，你可以在回调函数里做相应操作了。

## 消息钩子函数背景

消息钩子主要由三部分组成：

1. 钩子安装函数
2. 钩子回调函数
3. 钩子卸载函数

### 钩子安装函数原型

```
HHOOK WINAPI SetWindowsHookEx(
  _In_  int idHook,
  _In_  HOOKPROC lpfn,
  _In_  HINSTANCE hMod,
  _In_  DWORD dwThreadId
);
```

参数1-`idHook`：这个函数代表了你要安装钩子的种类，比如键盘钩子，鼠标钩子，窗体钩子等等。以下我列了一张表，结合MSDN供大家参考。

![](https://bbs.kanxue.com/upload/attach/201710/654109_5utof66puvivc2s.jpg)

参数2-`HOOKPROC lpfn`：此参数是钩子回调函数的地址，对应上述不同种类的钩子类型，其钩子回调函数原型基本是一致的。请见下（需要注意的是，`wParam`和`lParam`针对不同类型的钩子，传递参数所代表意义不同，因种类繁多，请各位查阅MSDN）：

```
LRESULT CALLBACK HookProc(
  int nCode, 
  WPARAM wParam, 
  LPARAM lParam
)
{
   // process event
   ...

   return CallNextHookEx(NULL, nCode, wParam, lParam);
}
```

`nCode`：`int`，此参数指示`Hook`例程是否需要处理消息，如果`nCode`为`HC_ACTION`，则需要处理；如果小于0，不予处理，须调用`CallNextHookEx`函数返回。

`wParam`与`lParam`：针对不同类型的钩子，代表的意义不同，但总体是针对当前类型钩子的消息。后面以例子做出验证。

参数3-`HINSTANCE hMod`：包含`Hook`例程的`Dll`所在模块基址。

参数4-`DWORD dwThreadId`：所要注入程序的主线程ID。如果此项填0，则代表钩子为系统钩子，基本所有在运行的进程都会被注入。如果此项指定线程ID，则是有针对性的线程钩子。

返回值：`HHOOK`类型的句柄。

### 钩子回调函数

请见上

### 钩子卸载函数

```
BOOL WINAPI UnhookWindowsHookEx(
  _In_  HHOOK hhk
);
```

参数：`SetWindowsHookEx`的返回值。

## 键盘消息钩子及CBT钩子程序代码

首先我们要明确下，钩子从安装到回显的流程

1. 编写`Dll`，在`Dll`中实现我们所需要的钩子回调函数及安装函数。
2. `Dll`中除了要编写与钩子相关的处理函数，也要编写与显示程序通信的模块，以便验证钩子消息的正确性。
3. `Dll`需要导出钩子启动函数与卸载函数。
4. 回显程序处理已安装钩子发送的信息。

至此，框架已经明了，我们分步拆解完成这个小项目。（整体代码请见附件，VS2013编译）

### 结构体及枚举的定义

```
typedef struct COMMPACK
{
	int nType;//Hook类型
	WPARAM wParam;//参数1
	LPARAM lParam;//参数2
	TCHAR szMsg[16];//额外的字符串信息，取决于具体钩子
} COMMPACK, *PCOMMPACK;//与回显程序通讯的包结构

typedef struct HOOKSTC
{
	int nType;//Hook类型
	HOOKPROC hkproc;//Hook回调
	HHOOK hhook;//Hook句柄
} HOOKSTC, *PHOOKSTC;

HOOKSTC m_Array4Hooks[NHOOKNUMS] = { 0 };//创建多钩子类型的结构体数组

enum { M_CALLWNDPROC, M_CBT, M_DEBUG, M_GETMESSAGE, M_KEYBOARD, M_MOUSE, M_MSGFILTER };//钩子类型枚举

enum { IDHCBT_ACTIVATE, IDHCBT_CLICKSKIPPED, IDHCBT_CREATEWND, IDHCBT_DESTROYWND, IDHCBT_KEYSKIPPED,
	IDHCBT_MINMAX, IDHCBT_MOVESIZE, IDHCBT_QS, IDHCBT_SETFOCUS, IDHCBT_SYSCOMMAND, IDUnknown
};//CBT钩子的消息群
```

### CBT钩子的回调函数

```
LRESULT WINAPI m_CBTProc(int nCode, WPARAM wParam, LPARAM lParam)
{
	if (nCode < 0)return CallNextHookEx(m_Array4Hooks[M_CBT].hhook, nCode, wParam, lParam);

	CHAR szCode[16] = { 0 };
	PCOMMPACK pCP = new COMMPACK;
	pCP->nType = M_CBT;
	switch (nCode)
	{
	case HCBT_ACTIVATE://系统要激活一个窗口
		pCP->lParam = (LPARAM)IDHCBT_ACTIVATE;
		pCP->wParam = IDHCBT_ACTIVATE;
		break;
	case HCBT_CLICKSKIPPED://系统已经从系统消息队列中删除了一个鼠标消息
		pCP->lParam = (LPARAM)IDHCBT_CLICKSKIPPED;
		pCP->wParam = IDHCBT_CLICKSKIPPED;
		break;
	case HCBT_CREATEWND://一个窗口将要被创建
		pCP->lParam = (LPARAM)IDHCBT_CREATEWND;
		pCP->wParam = IDHCBT_CREATEWND;
		break;
	case HCBT_DESTROYWND://一个窗口将要被销毁
		pCP->lParam = (LPARAM)IDHCBT_DESTROYWND;
		pCP->wParam = IDHCBT_DESTROYWND;
		break;
	case HCBT_KEYSKIPPED://系统已从系统消息队列中删除了一个键盘消息
		pCP->lParam = (LPARAM)IDHCBT_KEYSKIPPED;
		pCP->wParam = IDHCBT_KEYSKIPPED;
		break;
	case HCBT_MINMAX://一个窗口将被最小化或最大化
		pCP->lParam = (LPARAM)IDHCBT_MINMAX;
		pCP->wParam = IDHCBT_MINMAX;
		break;
	case HCBT_MOVESIZE://一个窗口将被移动或改变尺寸
		pCP->lParam = (LPARAM)IDHCBT_MOVESIZE;
		pCP->wParam = IDHCBT_MOVESIZE;
		break;
	case HCBT_QS://系统已从系统消息队列中取到一个WM_QUEUESYNC 消息
		pCP->lParam = (LPARAM)IDHCBT_QS;
		pCP->wParam = IDHCBT_QS;
		break;
	case HCBT_SETFOCUS://一个窗口将要获得键盘焦点
		pCP->lParam = (LPARAM)IDHCBT_SETFOCUS;
		pCP->wParam = IDHCBT_SETFOCUS;
		break;
	case HCBT_SYSCOMMAND://一个系统命令将被执行
		pCP->lParam = (LPARAM)IDHCBT_SYSCOMMAND;
		pCP->wParam = IDHCBT_SYSCOMMAND;
		break;
	default://其他
		pCP->lParam = (LPARAM)IDUnknown;
		pCP->wParam = IDUnknown;
		break;
	}
	m_Com(pCP);//与回显程序通信
	delete pCP;
	return CallNextHookEx(m_Array4Hooks[M_CBT].hhook, nCode, wParam, lParam);
}
```

### 键盘钩子的回调函数

```
LRESULT WINAPI m_KeyboardProc(int nCode, WPARAM wParam, LPARAM lParam)
{
	if ((nCode < 0 || !((DWORD)lParam & 0x40000000)))return CallNextHookEx(m_Array4Hooks[M_KEYBOARD].hhook, nCode, wParam, lParam);

	PCOMMPACK pCP = new COMMPACK;
	pCP->lParam = lParam;
	pCP->wParam = wParam;//这个值就是按键的VirtualKey
	pCP->nType = M_KEYBOARD;
	m_Com(pCP);//与回显程序通讯
	delete pCP;
	return CallNextHookEx(m_Array4Hooks[M_KEYBOARD].hhook, nCode, wParam, lParam);
}
```

### 与回显程序通讯的函数

```
void m_Com(PCOMMPACK& pCP, int addi)
{
	hTarget = ::FindWindow(NULL, L"HookApp");//寻找回显程序窗口标题
	COPYDATASTRUCT stcCDS;//采用WM_COPYDATA消息方式进程间通讯
	stcCDS.cbData = sizeof(COMMPACK);
	stcCDS.dwData = addi;
	stcCDS.lpData = pCP;//打包消息结构体
	::SendMessage(hTarget, WM_COPYDATA, 0, (LPARAM)&stcCDS);
}
```

### 钩子初始化函数

```
void InitHooks()
{
	hTarget = ::FindWindow(NULL, L"HookApp");
	m_Array4Hooks[M_CALLWNDPROC].hkproc = m_CallWndProc;
	m_Array4Hooks[M_CALLWNDPROC].nType = WH_CALLWNDPROC;
	m_Array4Hooks[M_CBT].hkproc = m_CBTProc;
	m_Array4Hooks[M_CBT].nType = WH_CBT;
	//m_Array4Hooks[M_DEBUG]->hkproc = m_DebugProc;
	//m_Array4Hooks[M_DEBUG]->nType = WH_DEBUG;
	//m_Array4Hooks[M_GETMESSAGE]->hkproc = m_GetMsgProc;
	//m_Array4Hooks[M_GETMESSAGE]->nType = WH_GETMESSAGE;
	m_Array4Hooks[M_KEYBOARD].hkproc = m_KeyboardProc;
	m_Array4Hooks[M_KEYBOARD].nType = WH_KEYBOARD;
	m_Array4Hooks[M_MOUSE].hkproc = m_MouseProc;
	m_Array4Hooks[M_MOUSE].nType = WH_MOUSE;
	//m_Array4Hooks[M_MSGFILTER]->hkproc = m_MessageFilterProc;
	//m_Array4Hooks[M_MSGFILTER]->nType = WH_MSGFILTER;
	for (int i = 0; i < NHOOKNUMS; ++i)
	{
		m_Array4Hooks[i].hhook = 0;
	}
}
```

### 钩子安装及卸载函数

```
void InstallHook(DWORD dwProcessID)
{
	g_dwThreadID = GetThreadInfo(dwProcessID);//由注入器传入指定注入进程ID，遍历获得主线程ID

	InitHooks();//初始化钩子

	for (int i = 0; i < NHOOKNUMS; ++i)
	{
		if (m_Array4Hooks[i].hkproc == NULL)continue;
		m_Array4Hooks[i].hhook = SetWindowsHookEx(m_Array4Hooks[i].nType, m_Array4Hooks[i].hkproc, g_Module, g_dwThreadID);
	}
}

void UnInstallHook()
{
	for (int i = 0; i < NHOOKNUMS; ++i)
	{
		UnhookWindowsHookEx(m_Array4Hooks[i].hhook);
	}
}
```

### 线程遍历函数

```
DWORD GetThreadInfo(DWORD dwProcessID)
{
	HANDLE hSnThread = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);
	if (hSnThread == INVALID_HANDLE_VALUE) {
		return 0;
	}

	// 开始遍历线程
	THREADENTRY32 threadEntry = { sizeof(THREADENTRY32) };

	// 获取快照中的第一个线程的信息
	Thread32First(hSnThread, &threadEntry);

	DWORD dwSuspendCount = 0;
	do {
		// 判断遍历到的线程是否属于这个进程的.
		if (threadEntry.th32OwnerProcessID == dwProcessID) {
			return threadEntry.th32ThreadID;
		}
		// 获取快照中的下一个线程信息
	} while (Thread32Next(hSnThread, &threadEntry));
	return 1;
}
```

### 函数导出

```
EXTERN_C _declspec(dllexport) void InstallHook(DWORD dwProcessID);
EXTERN_C _declspec(dllexport) void UnInstallHook();
```

至此DLL部分结束，开始回显程序的消息处理
