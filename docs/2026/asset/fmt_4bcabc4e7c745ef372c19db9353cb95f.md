
`BOOL``UnLoadDllByRemoteThread(``DWORD``dwProcessId,``LPCWSTR``lpDllName)`

```BOOL UnLoadDllByRemoteThread(DWORD dwProcessId, LPCWSTR lpDllName)
{
HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessId);
if (hProcess == NULL)
return FALSE;

// 在目标进程中申请一块内存，并将需要卸载的DLL模块的名称写入该内存
LPVOID lpRemoteDllName = VirtualAllocEx(hProcess, NULL, (wcslen(lpDllName) + 1) * sizeof(WCHAR), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
if (lpRemoteDllName == NULL)
{
CloseHandle(hProcess);
return FALSE;
}
if (!WriteProcessMemory(hProcess, lpRemoteDllName, lpDllName, (wcslen(lpDllName) + 1) * sizeof(WCHAR), NULL))
{
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM_RELEASE);
CloseHandle(hProcess);
return FALSE;
}

//查找dll模块
HMODULE hModules[1024],DesModule=NULL;
DWORD dwSize = 0;
if (!EnumProcessModules(hProcess, hModules, sizeof(hModules), &dwSize))
{
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM_RELEASE);
CloseHandle(hProcess);
return FALSE;
}

// 遍历模块列表，查找需要卸载的DLL模块
for (DWORD i = 0; i < (dwSize / sizeof(HMODULE)); i++)
{
WCHAR szModuleName[MAX_PATH] = { 0 };
if (GetModuleFileNameExW(hProcess, hModules[i], szModuleName, MAX_PATH) > 0)
{
// 获取模块句柄
if (wcsicmp(szModuleName, lpDllName) == 0)
{
DesModule = hModules[i];
}
}
}
//没有查找到模块
if (!DesModule) {
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM_RELEASE);
CloseHandle(hProcess);
return FALSE;
}
// 在目标进程中创建远程线程，执行FreeLibrary函数
HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)FreeLibrary, DesModule, 0, NULL);
if (hThread == NULL)
{
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM_RELEASE);
CloseHandle(hProcess);
return FALSE;
}

// 等待线程执行完成
WaitForSingleObject(hThread, INFINITE);

// 关闭句柄
CloseHandle(hThread);
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM_RELEASE);
CloseHandle(hProcess);

return TRUE;
}
```

## 关键dll函数

这里使用了三种方法

注意: 不要将代码写入`switch(reason)`之外,否则可能会导致多次执行

```#include<windows.h>
#include<stdio.h>

//调用函数
BOOL GrowPlant(DWORD BaseAddr, DWORD x, DWORD y, DWORD TypePlant) {
LPVOID PlantFunc = BaseAddr + 0x18D70;
__asm {
pushad
push -1 //-1
push TypePlant //植物类型
mov eax, y //y
push x //x
mov ecx, BaseAddr
mov ecx, [ecx+0x355E0C]
mov ecx, [ecx + 0x868]
push ecx //植物种植ebp
call PlantFunc
popad
}
return TRUE;
}

BOOL WINAPI DllMain(HMODULE hInstance, DWORD fdwReason, LPVOID lpReserved) {
DWORD BaseAddr = GetModuleHandle(NULL);
DWORD pid = GetCurrentProcessId();
HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
LPVOID PlantFunc = BaseAddr + 0x18D70;
DWORD ebpAddr = BaseAddr+0x355E0C,num=0;
ReadProcessMemory(hProcess, ebpAddr, &ebpAddr, sizeof(DWORD), NULL);
ebpAddr += 0x868;
ReadProcessMemory(hProcess, ebpAddr, &ebpAddr, sizeof(DWORD), NULL);//必须使用带hProcess参数的才能正确读取到地址,NULL不可以
DWORD x = 1, y = 1, TypePlant = 16;

//注意不要写到switch外,否则可能会一次种多株植物,猜测是dll被多个线程加载导致的
switch (fdwReason)
{
case DLL_PROCESS_ATTACH: //当进程加载dll模块时执行
//MessageBoxW(0, L"ProcessAttach!", L"window2", 0);
//1.直接通过使用ReadProcessMemory函数读取内存获取ebp参数
__asm {
pushad
push - 1 //-1
push TypePlant //植物类型
mov eax, y //y
push x //x
push  ebpAddr //ebp
call PlantFunc
popad
}

//2.通过利用寄存器获取ebp(推荐)
x = 3, y = 2, TypePlant = 18;
__asm {
pushad
push - 1 //-1
push TypePlant //植物类型
mov eax, y //y
push x //x
mov ecx, BaseAddr
mov ecx, [ecx+0x355E0C]
mov ecx, [ecx + 0x868]
push ecx
call PlantFunc
popad
}
//3. 通过调用函数(推荐)
GrowPlant(BaseAddr,7,3,23);

break;
//case DLL_THREAD_ATTACH:
//    printf("ThreadAttach!\n");
//    break;
//case DLL_THREAD_DETACH:
//  if (lpReserved == NULL)
//    {
//        FreeLibrary(hInstance);
//    }
//break;
case DLL_PROCESS_DETACH: //当进程卸载dll模块时执行
MessageBoxW(0, L"ProcessDeTachDll!", L"window2", 0);
break;
}
return TRUE;
}
```

## 执行结果

![](https://bbs.kanxue.com/upload/attach/202308/968342_VMAJU5AURRJJ2T9.png)

## 失败代码

这是写dll函数时遇到的问题 如果直接用 `mov ecx,[BaseAddr+0x355E0C]`会导致代码执行失败,推测是这条指令访存过慢所以无效

建议`mov ecx,BaseAddr`之后通过对寄存器操作达到目的

```__asm {
pushad
push - 1 //-1
push TypePlant //植物类型
mov eax, y //y
push x //x
mov ecx,[BaseAddr+ 0x355E0C]//这样不行,推测是访存过慢
mov ecx,[ecx+0x868]
mov num, ecx
push ecx
call PlantFunc
popad
}
```

## 远程线程代码注入(推荐)

和远程线程dll注入类似,`CreateRemoteThread`函数要求的函数原型是

```DWORD WINAPI ThreadProc(
_In_ LPVOID lpParameter//使用CreateThread函数传递的参数 该参数是一个指向其他数据的指针,当然也可以强转为其他类型直接使用
);
```

基本过程:

1. 打开进程
2. 定义注入代码(函数)
3. 在目标进程中申请空间并写入注入代码
4. 创建远程线程执行注入代码(函数)
5. 执行完毕释放空间
