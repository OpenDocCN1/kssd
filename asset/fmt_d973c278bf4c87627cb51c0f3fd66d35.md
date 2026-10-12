
# 《如何在 NT 下获取进程的路径》

> 原文：`www.pediy.com/kssd/pediy07/pediy7-719.htm`

## 一：获取 NT 下进程路径的方法

在 `Win9X` 系统中，利用 `ToolHelp` API 中的相关函数可以很方便得得到进程的名称及其路径。但这种方法在 `NT` 系统中就不能奏效了，`szExeFile` 仅仅得到的是进程的名称，并没有包含进程的路径。

如何在 `NT` 下获取进程的路径呢？（由于 `WIN9X` 系统不在我们讨论的范围之内，所以我们选用 `PSAPI` 中的相关函数进行说明，这仅仅适用于 `NT` 系统。）其实也很简单——用 `OpenProcess()` 函数将进程打开后，再利用 `EnumProcessModules()` 函数枚举该进程的模块，最后利用 `GetModuleFileNameEx()` 函数就能取得该进程的路径了。

下面的这段程序将枚举 `NT` 系统中的进程，并将显示每个进程的路径。当然，这种方法对一些系统保护的进程而言或多或少会出现些问题，例如：`smss.exe`、`winlogon.exe`，`csrss.exe` 等。如果谁有更好的方法请赐教，不胜感激。

## 二：具体实践

```c
//////////////////////////////////////////////////////////////////////////////////
/*
* ShowProcessPath_PSAPI
* 版权所有 (C) 2004 赵春生
* 2004.08.08
* http://timw.yeah.net
* http://timw.126.com
* 本程序适用于:WinNT
* 代码在 Win2000P+SP4 + VC6+SP5 测试通过
*/

#include <stdio.h>
#include <windows.h>
#include "PSAPI.H"
#pragma comment( lib, "PSAPI.LIB" )

int main(void)
{
DWORD processid[1024],needed,processcount,i;
HANDLE hProcess;
HMODULE hModule;
char path[MAX_PATH] = "",temp[256];

printf("ShowProcessPath with [Process Status API]\n\n");
EnumProcesses(processid, sizeof(processid), &needed);
processcount=needed/sizeof(DWORD);
for (i=0;i<processcount;i++)
{
hProcess=OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ,false,processid[i]);
if (hProcess)
{
EnumProcessModules(hProcess, &hModule, sizeof(hModule), &needed);
GetModuleFileNameEx(hProcess, hModule, path, sizeof(path));
GetShortPathName(path,path,256);
itoa(processid[i],temp,10);
printf("%s --- %s\n",path,temp);
}
else
printf("Failed!!!\n");
}
CloseHandle(hProcess);
CloseHandle(hModule);
itoa(processcount,temp,10);
printf("\nProcess Count:%s\n\n",temp);
return 0;
}

//////////////////////////////////////////////////////////////////////////////////
```

## 三：相关测试

用 `ToolHelp` API 按上述方法也能达到获取进程路径的目的，但和 `PSAPI` 相比后者则更有优势，详见下表，测试环境为：`Win2000P+SP4`。

| 进程名 | PSAPI | THAPI |
| :--- | :---: | :---: |
| `smss.exe` | Y | N |
| `csrss.exe` | N | N |
| `winlogon.exe` | Y | N |
| `services.exe` | Y | N |
| `lsass.exe` | Y | N |
| `svchost.exe` | Y | N |
| `WinMgmt.exe` | Y | N |
| `Explorer.EXE` | Y | Y |
| `mobsync.exe` | Y | Y |
| `conime.exe` | Y | Y |

四：以上代码在 `Win2000P+SP4` + `VC6+SP5` 测试通过。

源码可从我的个人主页下载：
`http://timw.yeah.net`
`http://timw.126.com`

`20:09 2004-08-08`
`http://timw_web.home4u.china.com/myzhuowen/propath/showprocesspath.zip`

楼主的办法还有缺陷，最彻底的办法是用 `PsGetCurrentProcess` 函数得到当前进程的句柄，他是一个指向 `_EPROCESS` 结构的指针，再利用该结构中的 `*PEB(0x1B0)` 和 `ImageFileName(0x1FC)` 两成员就可以得到完整的路径了。示意图为：

`_EPROCESS->PEB->ProcessParamwters(进程路径)->ImageFile(进程名称)`

# 《如何在 NT 下获取进程的路径》增

——获取 NT 中系统进程的路径

## 一：程序说明

最近整理文档，发现以前写的《如何在 NT 下获取进程的路径》一文中还有个问题没有解决：原文中的程序无法获取系统进程的路径，如：`csrss.exe`。记得 `VCKBASE` 上有位网友说过一个方法：“给枚举的进程增加 `SE_DEBUG_NAME` 权限即可”，于是在网上找了些资料，解决了原文中的问题。这里要特别感谢那位名叫 `rovershen` 的网友！

我自定义了一个函数，用来赋予进程指定的权限（本例为 `SE_DEBUG_NAME`）：

```c
//////////////////////////////////////////////////////////////////////
BOOL EnablePrivilege(HANDLE hToken,LPCSTR szPrivName)
{
TOKEN_PRIVILEGES tkp;
LookupPrivilegeValue( NULL,szPrivName,&tkp.Privileges[0].Luid );//修改进程权限
tkp.PrivilegeCount=1;
tkp.Privileges[0].Attributes=SE_PRIVILEGE_ENABLED;
AdjustTokenPrivileges( hToken,FALSE,&tkp,sizeof tkp,NULL,NULL );//通知系统修改进程权限
return( (GetLastError()==ERROR_SUCCESS) );
}
//////////////////////////////////////////////////////////////////////
```

通过 `OpenProcessToken` 函数获得进程（本例为自身进程）访问令牌的句柄，然后调用此函数后就可以像原文那样打开目标进程获取路径了。可以看到：本方法已经成功获取了系统进程 `csrss.exe` 的路径。

## 二：具体实践

```c
//////////////////////////////////////////////////////////////////////////////////
/*
* ShowProcessPath 2.0
* 版权所有 (C) 2005 赵春生
* 2005.09.02
* http://timw.yeah.net
* http://timw.126.com
* 本程序适用于:WinNT
* 代码在 Win2000P+SP4 + VC6+SP6 测试通过
*/

#include <stdio.h>
#include <windows.h>
#include "PSAPI.H"
#pragma comment( lib, "PSAPI.LIB" )

//自定义函数：赋予指定特权。这里用来提升程序权限。
BOOL EnablePrivilege(HANDLE hToken,LPCSTR szPrivName);

int main(void)
{
DWORD processid[1024],needed,processcount,i;
HANDLE hProcess;
HMODULE hModule;
char path[MAX_PATH] = "",temp[256];
HANDLE hToken;

printf("ShowProcessPath 2.0 with [Process Status API]\n\n");
if ( OpenProcessToken(GetCurrentProcess(),TOKEN_ADJUST_PRIVILEGES,&hToken) )
{
if (EnablePrivilege(hToken,SE_DEBUG_NAME))
{
EnumProcesses(processid, sizeof(processid), &needed);
processcount=needed/sizeof(DWORD);
for (i=0;i<processcount;i++)
{
hProcess=OpenProcess(PROCESS_QUERY_INFORMATION | PROCESS_VM_READ,false,processid[i]);
if (hProcess)
{
EnumProcessModules(hProcess, &hModule, sizeof(hModule), &needed);
GetModuleFileNameEx(hProcess, hModule, path, sizeof(path));
GetShortPathName(path,path,256);
itoa(processid[i],temp,10);
printf("%s --- %s\n",path,temp);
}
else
printf("Failed!!!\n");
}
}
}
CloseHandle(hProcess);
CloseHandle(hModule);
itoa(processcount,temp,10);
printf("\nProcess Count:%s\n\n",temp);
return 0;
}

//////////////////////////////////////////////////////////////////////
BOOL EnablePrivilege(HANDLE hToken,LPCSTR szPrivName)
{
TOKEN_PRIVILEGES tkp;
LookupPrivilegeValue( NULL,szPrivName,&tkp.Privileges[0].Luid );//修改进程权限
tkp.PrivilegeCount=1;
tkp.Privileges[0].Attributes=SE_PRIVILEGE_ENABLED;
AdjustTokenPrivileges( hToken,FALSE,&tkp,sizeof tkp,NULL,NULL );//通知系统修改进程权限
return( (GetLastError()==ERROR_SUCCESS) );
}
//////////////////////////////////////////////////////////////////////

//////////////////////////////////////////////////////////////////////////////////
```

## 三：以上代码在 Win2000P+SP4 + VC6+SP6 测试通过

源码可从我的个人主页下载：
`http://timw.yeah.net`
`http://timw.126.com`

`20:33 2005-09-02`
`http://timw_web.home4u.china.com/myzhuowen/propath/showprocesspath2.zip`

`4dd466d0e1e0cc123afbe504da2462ca`
