
```c
BOOL UnLoadDllByRemoteThread(DWORD dwProcessId, LPCWSTR lpDllName)
{
HANDLE hProcess = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, dwProcessId);
if (hProcess == NULL)
return FALSE;
// 在目标进程中申请一块内存，并将需要卸载的DLL模块的名称写入该内存
LPVOID lpRemoteDllName = VirtualAllocEx(hProcess, NULL, (wcslen(lpDllName) + 1) \* sizeof(WCHAR), MEM\_COMMIT | MEM\_RESERVE, PAGE\_READWRITE);
if (lpRemoteDllName == NULL)
{
CloseHandle(hProcess);
return FALSE;
}
if (!WriteProcessMemory(hProcess, lpRemoteDllName, lpDllName, (wcslen(lpDllName) + 1) \* sizeof(WCHAR), NULL))
{
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM\_RELEASE);
CloseHandle(hProcess);
return FALSE;
}
//查找dll模块
HMODULE hModules[1024],DesModule=NULL;
DWORD dwSize = 0;
if (!EnumProcessModules(hProcess, hModules, sizeof(hModules), &dwSize))
{
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM\_RELEASE);
CloseHandle(hProcess);
return FALSE;
}
// 遍历模块列表，查找需要卸载的DLL模块
for (DWORD i = 0; i < (dwSize / sizeof(HMODULE)); i++)
{
WCHAR szModuleName[MAX\_PATH] = { 0 };
if (GetModuleFileNameExW(hProcess, hModules[i], szModuleName, MAX\_PATH) > 0)
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
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM\_RELEASE);
CloseHandle(hProcess);
return FALSE;
}
// 在目标进程中创建远程线程，执行FreeLibrary函数
HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD\_START\_ROUTINE)FreeLibrary, DesModule, 0, NULL);
if (hThread == NULL)
{
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM\_RELEASE);
CloseHandle(hProcess);
return FALSE;
}
// 等待线程执行完成
WaitForSingleObject(hThread, INFINITE);
// 关闭句柄
CloseHandle(hThread);
VirtualFreeEx(hProcess, lpRemoteDllName, 0, MEM\_RELEASE);
CloseHandle(hProcess);
return TRUE;
}
```

## 关键dll函数

这里使用了三种方法

注意: 不要将代码写入`switch(reason)`之外,否则可能会导致多次执行

```c
#include<windows.h>
#include<stdio.h>
//调用函数
BOOL GrowPlant(DWORD BaseAddr, DWORD x, DWORD y, DWORD TypePlant) {
LPVOID PlantFunc = BaseAddr + 0x18D70;
\_\_asm {
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
HANDLE hProcess = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, pid);
LPVOID PlantFunc = BaseAddr + 0x18D70;
DWORD ebpAddr = BaseAddr+0x355E0C,num=0;
ReadProcessMemory(hProcess, ebpAddr, &ebpAddr, sizeof(DWORD), NULL);
ebpAddr += 0x868;
ReadProcessMemory(hProcess, ebpAddr, &ebpAddr, sizeof(DWORD), NULL);//必须使用带hProcess参数的才能正确读取到地址,NULL不可以
DWORD x = 1, y = 1, TypePlant = 16;
//注意不要写到switch外,否则可能会一次种多株植物,猜测是dll被多个线程加载导致的
switch (fdwReason)
{
case DLL\_PROCESS\_ATTACH: //当进程加载dll模块时执行
//MessageBoxW(0, L"ProcessAttach!", L"window2", 0);
//1.直接通过使用ReadProcessMemory函数读取内存获取ebp参数
\_\_asm {
pushad
push - 1 //-1
push TypePlant //植物类型
mov eax, y //y
push x //x
push ebpAddr //ebp
call PlantFunc
popad
}
//2.通过利用寄存器获取ebp(推荐)
x = 3, y = 2, TypePlant = 18;
\_\_asm {
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
//case DLL\_THREAD\_ATTACH:
// printf("ThreadAttach!\n");
// break;
//case DLL\_THREAD\_DETACH:
// if (lpReserved == NULL)
// {
// FreeLibrary(hInstance);
// }
break;
case DLL\_PROCESS\_DETACH: //当进程卸载dll模块时执行
MessageBoxW(0, L"ProcessDeTachDll!", L"window2", 0);
break;
}
return TRUE;
}
```

执行结果

![](upload/attach/202308/968342\_VMAJU5AURRJJ2T9.png)

## 失败代码

这是写dll函数时遇到的问题 如果直接用 `mov ecx,[BaseAddr+0x355E0C]` 会导致代码执行失败,推测是这条指令访存过慢所以无效

建议`mov ecx,BaseAddr`之后通过对寄存器操作达到目的

```c
\_\_asm {
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

```c
DWORD WINAPI ThreadProc(
\_In\_ LPVOID lpParameter//使用CreateThread函数传递的参数 该参数是一个指向其他数据的指针,当然也可以强转为其他类型直接使用
);
```

基本过程:

1. 打开进程

2. 定义注入代码(函数)

3. 在目标进程中申请空间并写入注入代码

4. 创建远程线程执行注入代码(函数)

5. 执行完毕释放空间

```c
//以创建远程线程方式种植植物
BOOL GrowPlantByInjectCode(DWORD dwProcessId,DWORD BaseAddr,DWORD x,DWORD y,DWORD PlantType)
{
BOOL bSuccess = FALSE;
//1. 打开进程
HANDLE hProcess = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, dwProcessId);
if (hProcess != NULL)
{
//2. 定义注入代码(函数)
BYTE InjectCode[50] = { //汇编指令 //修正点偏移
0x55, //0 push ebp
0x89, 0xE5, //1 mov ebp,esp
0x60, //3 pushad
0x68, 0xFF, 0xFF, 0xFF, 0xFF, //4 push -1
0x68, 0x00, 0x00, 0x00, 0x00, //9 push PlantType //10
0xB8, 0x00, 0x00, 0x00, 0x00, //14 mov eax,y //15
0x68, 0x00, 0x00, 0x00, 0x00, //19 push x //20
0xB9, 0x00, 0x00, 0x00, 0x00, //24 mov ecx,BaseAddr //25
0x8B, 0x89, 0x0C, 0x5E, 0x35, 0x00, //29 mov ecx,[ecx+0x355E0C]
0x8B, 0x89, 0x68, 0x08, 0x00, 0x00, //35 mov ecx,[ecx+0x868]
0x51, //41 push ecx
0xE8, 0x00, 0x00, 0x00, 0x00, //42 call PlantFunc //43 //被调方平栈
0x61, //47 popad
0xC9, //48 leave
0xC3 //49 ret
};
//3. 申请空间用于存储代码
DWORD dwCodeSize = 50, desFunc = BaseAddr + 0x18D70;
LPVOID lpRemoteCodeMem = VirtualAllocEx(hProcess, NULL, dwCodeSize, MEM\_COMMIT | MEM\_RESERVE, PAGE\_EXECUTE\_READWRITE);
//4. 修正参数
\*(DWORD\*)&InjectCode[10] = PlantType;
\*(DWORD\*)&InjectCode[15] = y;
\*(DWORD\*)&InjectCode[20] = x;
\*(DWORD\*)&InjectCode[25] = BaseAddr;
\*(DWORD\*)&InjectCode[43] = desFunc-((DWORD)lpRemoteCodeMem+42+5) ;
//call指令与jmp类似,相对于当前指令的下一条指令计算偏移,offset=des-(source+5),减去call自身长度5
if (lpRemoteCodeMem != NULL)
{
SIZE\_T dwBytesWritten = 0;
//5. 注入代码
if (WriteProcessMemory(hProcess, lpRemoteCodeMem, InjectCode, dwCodeSize, &dwBytesWritten) &&
dwBytesWritten == dwCodeSize)
{
//6. 创建远程线程执行代码
HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD\_START\_ROUTINE)lpRemoteCodeMem,NULL, 0, NULL);
if (hThread != NULL)
{
//7. 等待线程信号
WaitForSingleObject(hThread, INFINITE);
CloseHandle(hThread);
bSuccess = TRUE;
}
}
//8. 执行完后释放空间
VirtualFreeEx(hProcess, lpRemoteCodeMem, 0, MEM\_RELEASE);
}
CloseHandle(hProcess);
}
return bSuccess;
}
```

# 种植僵尸

## 基本原理

与种植植物思路类似

首先在头脑风暴中通过种植僵尸来找到僵尸数量地址

然后找到僵尸数量增加代码

再通过查看调用堆栈和参数找到种植僵尸`call`

参数应该也是`x` `y` `type` `ebp` (注意没有-1)

僵尸种植函数的`x`值在一个`call`上方,这个`call`是个`switch`结构,没有参数,所以`x`值也没被修改

![](upload/attach/202308/968342\_REFW2C9J6PVJFJQ.png)

## 种植僵尸函数--dll注入版

```c
BOOL GrowZombie(DWORD BaseAddr, DWORD x, DWORD y, DWORD ZombieType) {
LPVOID PlantZombieFunc = BaseAddr + 0x35390;
\_\_asm {
pushad
push x
push ZombieType
mov eax,y
mov ecx,BaseAddr
mov ecx,[ecx+0x355E0C]
mov ecx,[ecx+0x868]
mov ecx,[ecx+0x178] //ebp
call PlantZombieFunc
popad
}
return TRUE;
}
```

## 远程代码注入版
