
# [分享]没想过的神奇的思路系列：跟某些线程过不去

我们都知道某些东西的根源是远程线程与DLL注入。如果才能把注入和谐呢？这很简单也很复杂~

在win8之前，论坛里某个人发表的某代码（`http://bbs.pediy.com/showthread.php?t=119649`）是可以的。

到了win8的时候用这个带就立刻亮了![](https://bbs.kanxue.com/view/img/face/33.gif)——会出现神奇的咔嚓，主要是因为`PEB`和`LdrLink`模型的变更（`PEB`不在`fs:[0x30]`了，和采用了该死的`RedBlackLink`...）![](https://bbs.kanxue.com/view/img/face/50.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)~

but 我们有新思路，不去获取`peb`直接找一个`LdrEntry`就好了，使用神秘api:`LdrFindEntryForAddress`![](https://bbs.kanxue.com/view/img/face/41.gif)![](https://bbs.kanxue.com/view/img/face/28.gif)

原型：

```
NTSYSAPI

NTSTATUS

NTAPI

LdrFindEntryForAddress(

    HMODULE hMod,

    LDR\_DATA\_TABLE\_ENTRY\*\* ppEntry

);
```

剩下的问题就很简单了

通过API直接找到`kernel32.dll`的Entry然后修改`Kernel32`的Entry的`EntryPoint`(替换式hook)

然后 轻松ok了~~

代码：

```cpp
#include "stdafx.h"
#include <winternl.h>

typedef NTSTATUS (NTAPI*pfnLdrFindEntryForAddress)(HMODULE hMod, LDR_DATA_TABLE_ENTRY** ppLdrData);
typedef BOOL (WINAPI *PDLLENTRY) (
	__in  HINSTANCE hinstDLL,
	__in  DWORD fdwReason,
	__in  LPVOID lpvReserved);

BOOL HookDllEntry(HMODULE hDll,PDLLENTRY NewEntry,PDLLENTRY *pOldEntry)
{
	HMODULE hNtdll = GetModuleHandleW(L"ntdll.dll");
	pfnLdrFindEntryForAddress LdrFindEntryForAddress = (pfnLdrFindEntryForAddress)GetProcAddress(hNtdll, "LdrFindEntryForAddress");
	NTSTATUS ns;
	PLDR_DATA_TABLE_ENTRY pLdrData;
	ns = LdrFindEntryForAddress(hDll,&pLdrData);
	if (NT_SUCCESS(ns))
	{
		if (pOldEntry)
		{
			*pOldEntry = (PDLLENTRY)pLdrData->Reserved3[0];
		}
		if (NewEntry)
		{
			pLdrData->Reserved3[0]=(PVOID)NewEntry;
		}
		return TRUE;
	}
	return FALSE;
}
PDLLENTRY OldK32Main=NULL;
BOOL WINAPI OnK32Main(
	__in  HINSTANCE hinstDLL,
	__in  DWORD fdwReason,
	__in  LPVOID lpvReserved)
{
	char *pszReason;

	switch (fdwReason)
	{
	case DLL_PROCESS_ATTACH:
		pszReason = "DLL_PROCESS_ATTACH";
		break;
	case DLL_PROCESS_DETACH:
		pszReason = "DLL_PROCESS_DETACH";
		break;
	case DLL_THREAD_ATTACH:
		pszReason = "DLL_THREAD_ATTACH";
		break;
	case DLL_THREAD_DETACH:
		pszReason = "DLL_THREAD_DETACH";
		break;
	default:
		pszReason = "*UNKNOWN*";
		break;
	}
	OutputDebugStringA(pszReason);
	if (NULL == OldK32Main)
	{
		return FALSE;
	}
	else
	{
		return OldK32Main(
			hinstDLL,
			fdwReason,
			lpvReserved);
	}
}
void Test()
{
	HookDllEntry(GetModuleHandleA("kernel32.dll"),OnK32Main,&OldK32Main);
}
```

核心代码就是这些了，其他怎么检测那个线程的那啥都是仁者见仁的智者见智的事情了![](https://bbs.kanxue.com/view/img/face/12.gif)![](https://bbs.kanxue.com/view/img/face/12.gif)

如果还是不知道那么我只能无语![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)

求捐助，求赞助，求投资，有意者欢迎联系QQ:86879759

欲购买 `AntiGameProtect`或`BypassGameProtect`系列代码与产品 也欢迎联系。

`AntiGameProtect`系列用于穿透驱动保护调试程序

`BypassGameProtect`系列用于穿透驱动保护进行各种操作（如读写内存，注入dll，模拟键盘鼠标，防内存检测，DLL隐藏，反CRC效验等）

`UltraGameProtect`项目强烈求投资。

qq技术扯淡群1：171797360

qq技术扯淡群2：1748876

yy技术扯淡频道：80252844

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
