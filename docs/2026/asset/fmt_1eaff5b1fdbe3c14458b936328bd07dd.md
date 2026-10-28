
# 0x04:与 `LoadLibrary`相对的`FreeLibrary`

既然一个东西有安装，那必然就有卸载，凡是都是相对的，不可能只有单方面（我什么时候把哲学搞这么明白了）

![](https://bbs.kanxue.com/upload/attach/202002/680946_UKV2M323GSK23H4.jpg)

话说回来，如果存在DLL注入，能安装进去，那也就必然能从上面卸载下来

安装是`LoadLibrary`，卸载就是`FreeLibrary`，为啥是呢，因为我是从MSDN上面看到的：

df7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3I4A6j5X3I4G2j5h3c8W2M7X3q4H3K9g2)9J5c8X3&6X3i4K6u0V1L8r3W2T1L8r3!0S2k6r3g2J5j5i4m8A6i4K6u0V1L8r3!0S2k6r3I4A6j5Y4u0S2M7Y4W2S2

68aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3I4A6j5X3I4G2j5h3c8W2M7X3q4H3K9g2)9J5c8X3&6X3i4K6u0V1L8r3W2T1L8r3!0S2k6r3g2J5j5i4m8A6i4K6u0V1k6Y4u0W2k6h3I4A6j5Y4u0S2M7Y4V1.

![](https://bbs.kanxue.com/upload/attach/202002/680946_WX3Y65AYG2Y25K2.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/680946_2YNWGQZYYPJ4KWS.jpg)

同样的道理，注入可以远程调用`LoadLibrary`注射进去，那么我们也可以远程调用`FreeLibrary`卸载吧，尝试一下：

![](https://bbs.kanxue.com/upload/attach/202002/680946_DFUZNY8ACFS4NS4.jpg)

看一下该函数都有什么参数：

![](https://bbs.kanxue.com/upload/attach/202002/680946_VCG3A7T2Y3XUKXD.jpg)

发现需要传一个句柄，那么我们直接放源码吧，可以远程获取进程模块的句柄：

```
//获取模块句柄

HMODULE GetProcessModuleHandleByName(DWORD pid, LPCSTR ModuleName)

{

	MODULEENTRY32 ModuleInfo;

	HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid);

	if (!hSnapshot)

	{

		return 0;

	}

	ZeroMemory(&ModuleInfo, sizeof(MODULEENTRY32));

	ModuleInfo.dwSize = sizeof(MODULEENTRY32);

	if (!Module32First(hSnapshot, &ModuleInfo))

	{

		return 0;

	}

	do

	{

		if (!lstrcmpi(ModuleInfo.szModule, ModuleName))

		{

			CloseHandle(hSnapshot);

			return ModuleInfo.hModule;

		}

	} while (Module32Next(hSnapshot, &ModuleInfo));

	CloseHandle(hSnapshot);

	return 0;

}

//获取进程id

DWORD GetProcessIDByName(const char* pName)

{

	HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

	if (INVALID_HANDLE_VALUE == hSnapshot) {

		return NULL;

	}

	PROCESSENTRY32 pe = { sizeof(pe) };

	for (BOOL ret = Process32First(hSnapshot, &pe); ret; ret = Process32Next(hSnapshot, &pe)) {

		if (strcmp(pe.szExeFile, pName) == 0) {

			CloseHandle(hSnapshot);

			return pe.th32ProcessID;

		}

		//printf("%-6d %s\n", pe.th32ProcessID, pe.szExeFile);

	}

	CloseHandle(hSnapshot);

	return 0;

}
```

简单的修改一下，然后去调用该函数，完整代码如下：

```
#include <Windows.h>

#include <stdio.h>

#include "tlhelp32.h"



void Inject(int pID, char* Path)

{

	//获取进程句柄

	HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pID);



	//申请一块内存给DLL路径

	LPVOID pReturnAddress = VirtualAllocEx(hProcess, NULL, strlen(Path) + 1, MEM_COMMIT, PAGE_READWRITE);



	//写入路径到上一行代码申请的内存中

	WriteProcessMemory(hProcess, pReturnAddress, Path, strlen(Path) + 1, NULL);





	//获取LoadLibraryA函数的地址

	HMODULE hModule = LoadLibrary("KERNEL32.DLL");

	LPTHREAD_START_ROUTINE lpStartAddress = (LPTHREAD_START_ROUTINE)GetProcAddress(hModule, "LoadLibraryA");





	//创建远程线程-并获取线程的句柄

	HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, lpStartAddress, pReturnAddress, 0, NULL);



	//等待线程事件

	WaitForSingleObject(hThread, 2000);





	//防止内存泄露

	CloseHandle(hThread);

	CloseHandle(hProcess);



}



HMODULE GetProcessModuleHandleByName(DWORD pid, LPCSTR ModuleName)

{

	MODULEENTRY32 ModuleInfo;

	HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid);

	if (!hSnapshot)

	{

		return 0;

	}

	ZeroMemory(&ModuleInfo, sizeof(MODULEENTRY32));

	ModuleInfo.dwSize = sizeof(MODULEENTRY32);

	if (!Module32First(hSnapshot, &ModuleInfo))

	{

		return 0;

	}

	do

	{

		if (!lstrcmpi(ModuleInfo.szModule, ModuleName))

		{

			CloseHandle(hSnapshot);

			return ModuleInfo.hModule;

		}

	} while (Module32Next(hSnapshot, &ModuleInfo));

	CloseHandle(hSnapshot);

	return 0;

}



DWORD GetProcessIDByName(const char* pName)

{

	HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

	if (INVALID_HANDLE_VALUE == hSnapshot) {

		return NULL;

	}

	PROCESSENTRY32 pe = { sizeof(pe) };

	for (BOOL ret = Process32First(hSnapshot, &pe); ret; ret = Process32Next(hSnapshot, &pe)) {

		if (strcmp(pe.szExeFile, pName) == 0) {

			CloseHandle(hSnapshot);

			return pe.th32ProcessID;

		}

		//printf("%-6d %s\n", pe.th32ProcessID, pe.szExeFile);

	}

	CloseHandle(hSnapshot);

	return 0;

}





void UnInject(int pID, char* Path)

{

	//获取进程句柄

	HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pID);



	LPVOID pReturnAddress = GetProcessModuleHandleByName(GetProcessIDByName("代码注入器.exe"), "mydll.dll");



	//获取LoadLibraryA函数的地址

	HMODULE hModule = LoadLibrary("KERNEL32.DLL");

	LPTHREAD_START_ROUTINE lpStartAddress = (LPTHREAD_START_ROUTINE)GetProcAddress(hModule, "FreeLibrary");





	//创建远程线程-并获取线程的句柄

	HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, lpStartAddress, pReturnAddress, 0, NULL);



	//等待线程事件

	WaitForSingleObject(hThread, 2000);





	//防止内存泄露

	CloseHandle(hThread);

	CloseHandle(hProcess);



}



int main()

{

	const char* a = "C:\\Users\\86186\\Desktop\\mydll.dll";



	UnInject(GetProcessIDByName("代码注入器.exe"), (char*)a);



	getchar();

	return 0;

}
```

上一秒，这个DLL还在程序内：

![](https://bbs.kanxue.com/upload/attach/202002/680946_J82HJ53SFEJTURN.jpg)

下一秒，我们运行`Uninject`程序，然后刷新一下，它就不见了:

![](https://bbs.kanxue.com/upload/attach/202002/680946_ZTZFQ6Q8M843U2V.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/680946_9537QV3SRPVTHUS.jpg)

# 0x05:`FreeLibrary`卸载DLL跟DLL隐藏有什么关系？

很多朋友都会这么问，你卸载了，就没了，你还隐藏个鸡儿？

![](https://bbs.kanxue.com/upload/attach/202002/680946_73RAWADP6ZTF75E.jpg)

按道理确实是这样，但是也正是如此，真正的隐藏就是卸载掉，让他真正的不存在！

我们因此可以从`Freelibrary`函数开刀，分析该函数的流程，然后下手

![](https://bbs.kanxue.com/upload/attach/202002/680946_JP57FNVCWNUQ5NC.jpg)

# 0x06:分析`FreeLibrary`

先从MSDN给的官方文档下手：

![](https://bbs.kanxue.com/upload/attach/202002/680946_FM4KR2NZT8E9DMG.jpg)

总结一下，大致就是四部分：

1. 判断DLL句柄是否有效，有效就说明该DLL存在于进程中
2. 递减模块的引用计数，且判断是否为0
3. 调用模块的`DllMain`函数响应`DLL_PROCESS_DETACH`消息
4. 从进程空间撤销对DLL的内存映射

我们不难看出，前三步只是假把式，最后一笔才是点睛之笔，才真正把DLL从内存清出去

因此，如果我们让系统帮我们清楚前面的痕迹，等到清楚内存映射的时候，直接阻止它，这样既清了痕迹，又保了我们的DLL

![](https://bbs.kanxue.com/upload/attach/202002/680946_KNUQGX6A8SM4XAJ.jpg)

# 0x07:逆向`FreeLibrary`

打开OD，附加进程，跳转到函数处：

![](https://bbs.kanxue.com/upload/attach/202002/680946_M473SFH8RTR2JZN.jpg)

此时我们可以看到这个函数在里面到底偷偷做了些什么：

![](https://bbs.kanxue.com/upload/attach/202002/680946_XWSJVBRZY8EQRBB.jpg)

我们会发现，其调用了很多来自`ntdll`的函数，而且最后，是通过调用下面的`call`来实现最后的内存清除工作：

```
76021DF7  |.  FF15 C0110176 call    dword ptr [<&ntdll.NtUnmapViewOfSection>]               ;  ntdll_12.ZwUnmapViewOfSection
```

![](https://bbs.kanxue.com/upload/attach/202002/680946_V45WKZ5QEXX5E69.jpg)

不懂就查：

640K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6V1k6r3W2Q4x3V1k6%4k6r3#2Q4x3V1k6F1k6W2)9J5k6s2N6V1L8g2)9J5k6s2A6%4N6h3&6E0j5i4m8$3K9h3g2%4L8$3k6K6k6h3y4@1K9h3!0F1

看一下这玩意到底是不是清除内存的：
