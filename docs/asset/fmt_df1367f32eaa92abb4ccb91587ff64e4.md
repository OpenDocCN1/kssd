
# 第二阶段第二题代码(比较弱)(看雪金山 2007 逆向分析挑战赛)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-259.htm`](https://www.pediy.com/kssd/pediy09/pediy09-259.htm)

> 第二阶段第二题代码(比较弱)(看雪金山 2007 逆向分析挑战赛)

by `aker`

说老实话,第二题要想做的话，是个费时间的题目,不想动手去做很多,都是拷贝的代码,大家

随便看看吧.

使用 `IDA` 载入那个 `sys`，发现没有做什么其他的，就是常规的工作，然后 `hook` 了 `NtOpenProcess`

，这样你 `OpenProcess` 的时候，他判断是不是打开的自己，如果是就返回 `Deny`。也就是说，只

是限制了你不能打开进程，获取句柄。原理这个帖子里面有，大家可以先看看那个;)

`http://bbs.pediy.com/showthread.php?t=40832`

既然只是 `HOOK` 了，那么发送 `WM_CLOSE` 等可以关闭，修复 `ssdt` 也可以关闭

下面是我的一些代码。

来个最简单的吧

代码:

```
#include <windows.h>
int main(int argc, char *argv[])
{
HWND hwin = FindWindow(NULL,"crackmeapp");
SendMessage(hwin,WM_CLOSE,0,0);
return 0;
}
```

--------------------------------------------------------------------------------

代码:

```
// 窗口置前，发送 alt f4
#include <windows.h>
#pragma comment(lib,"user32")
int main(int argc, char *argv[])
{
HWND hwin = FindWindow(NULL,"crackmeapp");
SetForegroundWindow(hwin);
Sleep(20);
keybd_event(18,MapVirtualKey(18,0),0,0);
keybd_event(115,MapVirtualKey(115,0),0,0);
keybd_event(115,MapVirtualKey(115,0),KEYEVENTF_KEYUP,0);
keybd_event(115,MapVirtualKey(115,0),0,0);
keybd_event(115,MapVirtualKey(115,0),KEYEVENTF_KEYUP,0);
keybd_event(18,MapVirtualKey(18,0),KEYEVENTF_KEYUP,0);
return 0;
}
```

--------------------------------------------------------------------------------

关闭所有线程，呵呵

代码:

```
////////////头文件////////////////////////////////////////////////////////////
#include <stdio.h>
#include <windows.h>
#include <tlhelp32.h>

////////////宏定义////////////////////////////////////////////////////////////
////////////全局变量//////////////////////////////////////////////////////////
////////////函数定义//////////////////////////////////////////////////////////
DWORD WINAPI GetPIDbyName(LPTSTR lpName)
{
HANDLE m_Snap = INVALID_HANDLE_VALUE;
PROCESSENTRY32 pe = {sizeof(pe)};
DWORD ret = NULL;

{
m_Snap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,NULL);
if (m_Snap == INVALID_HANDLE_VALUE) goto finally;
if (!Process32First(m_Snap, &pe)) goto finally;
do if(!lstrcmpi(pe.szExeFile,lpName))
{
ret = pe.th32ProcessID;
goto finally;
}
while (Process32Next(m_Snap, &pe));
}

finally: {
if (m_Snap != INVALID_HANDLE_VALUE)
CloseHandle(m_Snap);
}
return ret;
}

bool TerminateAProcess(DWORD dwPid)
{
HANDLE hThreadSnap = NULL;
THREADENTRY32 te32;
BOOL bThreadFind = FALSE;
hThreadSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD,NULL);
if(hThreadSnap)
{
te32.dwSize = sizeof(THREADENTRY32);
bThreadFind = Thread32First(hThreadSnap,&te32);
while(bThreadFind)
{
if(te32.th32OwnerProcessID == dwPid)
{
HANDLE hThread = NULL;
hThread = OpenThread(THREAD_ALL_ACCESS,FALSE,te32.th32ThreadID);
if(hThread)
{
TerminateThread(hThread,0);
CloseHandle(hThread);
}
}
te32.dwSize = sizeof(THREADENTRY32);
bThreadFind = Thread32Next(hThreadSnap,&te32);
}
CloseHandle(hThreadSnap);
}
return TRUE;
}

int main(int argc, char *argv[])
{
TerminateAProcess(GetPIDbyName("crackmeapp.exe"));
return 0;
}
```

--------------------------------------------------------------------------------

采用的 `www.security.org.sg\code\SDTrestore-0.2.zip`

甚至代码都没有改什么，然后你就知道了吧，呵呵，就添加了这个

代码:

```
if (SetDebugPrivileges() == 0) puts("Unable to grant debug privileges !");
DWORD dwPID;
HANDLE hProcess;
dwPID = GetPIDbyName("CrackMeapp.exe");
if ((hProcess = OpenProcess(PROCESS_TERMINATE, FALSE, dwPID)) != NULL) {
if (TerminateProcess(hProcess, 0) == 0) printf("Unable to kill process %lu !\n", dwPID);
else printf("kill process OK!\n");
CloseHandle(hProcess);
} else printf("Unable to access process %lu !\n", dwPID);
```

--------------------------------------------------------------------------------

采用的 `www.security.org.sg\code\SDTrestore-0.2.zip`

`ring3` 下恢复被 `hook` 的 `ssdt` `Createremotethread` ,`exitprocess`，插入远程线程

代码:

```
if (SetDebugPrivileges() == 0) puts("Unable to grant debug privileges !");
DWORD dwPID;
HANDLE hProcess;
dwPID = GetPIDbyName("CrackMeapp.exe");
char lpDllFullPathName[MAX_PATH];
WCHAR pszLibFileName[MAX_PATH]={0};
PWSTR pszLibFileRemote=NULL;
int iReturnCode;

if ((hProcess = OpenProcess(PROCESS_TERMINATE | PROCESS_CREATE_THREAD |PROCESS_VM_OPERATION |PROCESS_VM_WRITE , FALSE, dwPID)) != NULL)
{
GetCurrentDirectory(MAX_PATH, lpDllFullPathName);
strcat(lpDllFullPathName, "\\");
strcat(lpDllFullPathName, "testdll.dll");
if(!(iReturnCode=(int)_lopen(lpDllFullPathName, OF_READ)))
printf("%s not exist~\n",lpDllFullPathName);
iReturnCode = MultiByteToWideChar(CP_ACP, MB_ERR_INVALID_CHARS,
lpDllFullPathName, strlen(lpDllFullPathName),pszLibFileName, MAX_PATH);

int cb = (1 + lstrlenW(pszLibFileName)) * sizeof(WCHAR);
pszLibFileRemote = (PWSTR) VirtualAllocEx( hProcess, NULL, cb, MEM_COMMIT, PAGE_READWRITE);

iReturnCode = WriteProcessMemory(hProcess, pszLibFileRemote, (PVOID) pszLibFileName, cb, NULL);
PTHREAD_START_ROUTINE pfnStartAddr = (PTHREAD_START_ROUTINE) GetProcAddress(GetModuleHandle(TEXT("Kernel32")), "LoadLibraryW");

HANDLE hRemoteThread = CreateRemoteThread( hProcess, NULL, 0, pfnStartAddr, pszLibFileRemote, 0, NULL);

WaitForSingleObject(hRemoteThread, INFINITE);
if (pszLibFileRemote != NULL)
VirtualFreeEx(hProcess, pszLibFileRemote, 0, MEM_RELEASE);
if (hRemoteThread != NULL) CloseHandle(hRemoteThread );
if (hProcess!= NULL) CloseHandle(hProcess);
} else printf("Unable to access process %lu !\n", dwPID);
```

`dll` 的代码

代码:

```
#include <windows.h>
BOOL APIENTRY DllMain(HANDLE hModule, DWORD reason, LPVOID lpReserved)
{
switch (reason){
case DLL_PROCESS_ATTACH:{
ExitProcess(0);
}
default:
return TRUE;
}
}
```

--------------------------------------------------------------------------------

1 个附件

第三个 `ring3` 下恢复被 `hook` 的 `ssdt` -->`create remote thread`,破坏该进程,最省事

代码:

```
if (SetDebugPrivileges() == 0) puts("Unable to grant debug privileges !");
DWORD dwPID;
HANDLE hProcess;
dwPID = GetPIDbyName("CrackMeapp.exe");

if ((hProcess = OpenProcess(PROCESS_TERMINATE | PROCESS_CREATE_THREAD |PROCESS_VM_OPERATION |PROCESS_VM_WRITE , FALSE, dwPID)) != NULL)
{
HANDLE hRemoteThread = CreateRemoteThread(hProcess,0,0,(DWORD (__stdcall *)(void *))0x20,0,0,0);
WaitForSingleObject(hRemoteThread, INFINITE);
if (hRemoteThread != NULL) CloseHandle(hRemoteThread );
if (hProcess!= NULL) CloseHandle(hProcess);
} else printf("Unable to access process %lu !\n", dwPID);
```

--------------------------------------------------------------------------------

上面都是应用层的,驱动的没有环境做,其实很多都差不多,说下思路吧,

一种上面一样,恢复 `ssdt` 中的序号为 `7A` 的 `NtOpenProcess`,然后该怎么杀就怎么杀,

这个代码也是一堆,可以搜索 `icesword` 看到.

`{openprocess,terminate}`

`{createremotethread,exitprocess}`

另外采用 `EPROCESS` 结构,获取每个线程,然后用 `PspTerminateThreadByPointer` 结束每个线程.

这些网上代码都很多的.

另外进程创建以后,windows 对象管理器中有 `handle`,但是这样做实在是太得不偿失了,
