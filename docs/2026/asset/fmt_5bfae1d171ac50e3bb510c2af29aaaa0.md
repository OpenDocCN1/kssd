
# 修改进程代码区代码

```cpp
//修改进程代码区代码 参数: 进程句柄 修改代码起始地址 硬编码指针 代码字节数
BOOL WriteProcessCodeMemory(HANDLE hProcess, LPVOID lpStartAddress, LPCVOID lpBuffer, SIZE_T nSize) {
  DWORD dwOldProtect;
  //取消页保护
  if (!VirtualProtectEx(hProcess, lpStartAddress, nSize, PAGE_EXECUTE_READWRITE, &dwOldProtect)) {
    return FALSE;
  }
  BOOL bResult = WriteProcessMemory(hProcess, lpStartAddress, lpBuffer, nSize, NULL);
  //写入代码
  VirtualProtectEx(hProcess, lpStartAddress, nSize, dwOldProtect, &dwOldProtect);
  //开启页保护
  return bResult;
}
```

```cpp
//hook指定地址,申请新空间保存原始代码并写入hookcode,返回申请空间的地址
LPVOID SetHook(HANDLE hProcess, LPVOID desAddr, LPCVOID hookCode, SIZE_T hookCodeSize, SIZE_T origCodeSize) {
  BYTE origCode[10] = { 0 }, jmpCode[5] = { 0xE9,0,0,0,0 };
  //1. 读取并保存原始代码
  if (!ReadProcessMemory(hProcess, desAddr, origCode, origCodeSize, NULL))
    return NULL;

  //2. 申请空间用于存储原始代码,hook代码,jmp返回代码
  LPVOID allocAddr = VirtualAllocEx(hProcess, NULL, hookCodeSize + origCodeSize + 5, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
  if (!allocAddr)
    return NULL;

  //3. 向申请空间写入原始代码,hook代码,jmp返回代码  jmp xxx 偏移为目的地址-jmp下一条指令地址
  *(DWORD*)(jmpCode + 1) = (DWORD)desAddr + 5 - ((DWORD)allocAddr + hookCodeSize + origCodeSize + 5);
  //hook返回地址的偏移
  if (!WriteProcessCodeMemory(hProcess, allocAddr, origCode, origCodeSize)
    //写入原始代码
    || !WriteProcessCodeMemory(hProcess, (DWORD)allocAddr + origCodeSize, hookCode, hookCodeSize)
    //写入hook代码
    || !WriteProcessCodeMemory(hProcess, (DWORD)allocAddr + origCodeSize + hookCodeSize, jmpCode, 5))
    //写入jmpcode
  {
    VirtualFreeEx(hProcess, allocAddr, 0, MEM_RELEASE);
    //写入失败则释放空间
    return NULL;
  }

  //4. 修改目的地址处的代码  jmp xxx偏移 原始代码后才是需要执行的hook代码
  *(DWORD*)(jmpCode + 1) = ((DWORD)allocAddr + origCodeSize) - ((DWORD)desAddr + 5);
  WriteProcessCodeMemory(hProcess, desAddr, jmpCode, 5);
  //在源地址处写入跳转代码
  if (origCodeSize > 5)
    //原始代码长度大于5时nop多余字节
  {
    BYTE nopCode[5] = { 0x90,0x90,0x90,0x90,0x90 };
    if (!WriteProcessCodeMemory(hProcess, (DWORD)desAddr + 5, nopCode, origCodeSize - 5))
    {
      VirtualFreeEx(hProcess, allocAddr, 0, MEM_RELEASE);
      //写入nopcode失败则释放空间并返回
      return NULL;
    }
  }

  //5. hook成功则返回hookCode所在地址
  return allocAddr;
}
```

```cpp
//取消hook指定地址,写回原始代码并释放申请空间
BOOL UnHook(HANDLE hProcess, LPVOID desAddr, SIZE_T origCodeSize, LPVOID allocAddr) {
  BYTE origCode[10] = { 0 };
  //1. 从申请空间中读出原始代码
  if (!ReadProcessMemory(hProcess, allocAddr, origCode, origCodeSize, NULL))
    return FALSE;
  //2. 将原始代码写回目的地址
  if (!WriteProcessCodeMemory(hProcess, desAddr, origCode, origCodeSize))
    return FALSE;
  //3. 释放申请空间
  if (!VirtualFreeEx(hProcess, allocAddr, 0, MEM_RELEASE))
    return FALSE;
  return TRUE;
}
```

## 除雾代码

```cpp
//除雾 注意保留hook代码首地址
LPVOID DeFogByHook(HANDLE hProcess, LPVOID BaseAddr) {

  unsigned char hookCode[9] = {
    0xc7,0x01,0x00,0x00,0x00,0x00,  //mov [ecx],0
    0x83,0xc1,0x04                 //add ecx,0x4
  };
  //写入hook代码进行hook
  return SetHook(hProcess, (DWORD)BaseAddr + 0x26173, hookCode, sizeof(hookCode), 5);
}
```

```cpp
//恢复雾
BOOL RecoveryFogByUnHook(HANDLE hProcess, LPVOID BaseAddr, LPVOID allocAddr) {
  return UnHook(hProcess, (DWORD)BaseAddr + 0x26173, 5, allocAddr);
}
```

hook前 指令为`mov [ecx],edx` `add ecx,04`

![](https://bbs.kanxue.com/upload/attach/202308/968342_YN79FKXKBKBWASZ.png)

hook后 指令被修改为`jmp`

![](https://bbs.kanxue.com/upload/attach/202308/968342_S8SPKXF62AUM5T4.png)

hookcode 新分配空间前5个字节正是原始代码 之后是hook代码和jmp返回代码

![](https://bbs.kanxue.com/upload/attach/202308/968342_QH93FEUECHT33VN.png)

# 种植植物

## 基本原理

程序是执行种植植物的函数后再执行增加植物数量的功能

首先查找草坪上的植物数量,初值0,随着种植个数增加 基址`0x355E0C` 偏移+868 +D4

再查找是什么修改了植物数量,下断点之后再种植一个植物

断下后查看调用堆栈中的返回地址,即可找到种植函数

![](https://bbs.kanxue.com/upload/attach/202308/968342_ZWRS49XFX3CK4G9.png)

这个功能最初使用远程线程注入dll来实现,注入dll虽然比较简单但是却并不通用,在此仅做介绍,比较推荐使用远程代码注入的方式实现

### 远程线程注入dll函数

远程线程是当前进程在目标进程中创建一个线程并执行特定代码(这段代码必须在目标进程中而不是当前进程中)

注入dll是因为dll在被进程或线程加载时执行dll的`DllMain`函数,通过这一特点我们可以实现一些特殊功能

优点: 便于实现

缺点: dll注入容易被检测到

基本过程:

1. 打开进程获取进程句柄
2. 在目标进程中申请空间用于存储dll路径名
3. 将dll路径名写入申请的空间中
4. 创建远程线程,执行`LoadLibrary`函数(加载dll)
5. 目标进程加载dll后自动执行dll的`DllMain`函数

```cpp
//创建远程线程方式向指定进程注入dll
BOOL InjectDllByRemoteThread(DWORD desProcId, WCHAR* dllPath) {
  //打开进程获取进程句柄
  HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, desProcId);
  if (!hProcess)
    return FALSE;

  //申请空间
  DWORD pathSize = (wcslen(dllPath) + 1) * 2;
  LPVOID newMemAddr = VirtualAllocEx(hProcess, 0, pathSize, MEM_COMMIT, PAGE_READWRITE);
  if (!newMemAddr)
    return FALSE;

  //写入dll路径
  if (!WriteProcessMemory(hProcess, newMemAddr, dllPath, pathSize, NULL))
  {
    VirtualFreeEx(hProcess, newMemAddr, 0, MEM_RELEASE);
    return FALSE;
  }

  //创建远程线程
  HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, (LPTHREAD_START_ROUTINE)LoadLibraryW, newMemAddr, 0, NULL);
  if (!hThread)
  {
    VirtualFreeEx(hProcess, newMemAddr, 0, MEM_RELEASE);
    return FALSE;
  }

  WaitForSingleObject(hThread, INFINITE); //等待线程信号,保证成功注入

  //回收资源
  VirtualFreeEx(hProcess, newMemAddr, 0, MEM_RELEASE);
  CloseHandle(hThread);
  CloseHandle(hProcess);

  //返回成功
  return TRUE;
}
```

### 远程线程卸载dll函数

很多教程只给出了如何注入dll,没有演示如何卸载

如果只注入不卸载会导致下次再注入时不会执行特定函数(由于dll已经被加载过) 不方便实时调试更新dll等问题

基本过程:

1. 在目标进程申请内存,将需要卸载的dll模块名称写入该内存
2. 通过枚举模块来查找指定模块
3. 成功查找到dll模块则创建远程线程执行`FreeLibrary`函数卸载dll
