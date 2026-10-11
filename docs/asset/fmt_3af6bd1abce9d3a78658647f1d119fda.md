
# 一个 Themida 加壳的程序 LOADER 破解

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-458.htm`](https://www.pediy.com/kssd/pediy08/pediy8-458.htm)

**【文章标题】** 一个 Themida 加壳的程序 LOADER 破解  
**【文章作者】** rockhard  
**【作者邮箱】** `wnh1@sohu.com`  
**【软件名称】** 附件下载  
**【加壳方式】** Themida  
**【保护方式】** Themida  
**【使用工具】** `ollydbg`, VC  
**【作者声明】** 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

### 详细过程

遇到一个程序，在 `Ollydbg` 修改某些关键点可以运行了，但由于用 `THEMIDA` 加壳了，水平菜，脱不掉壳，每次只能在 `OLLYDBG` 中运行，想给他写个 LOADER 加载，然后修改内存。发现用 `CreateProcess`、`ShellExecute` 和 `WinExec` 运行起来就报错，看来是加了 LOADER 的 ANTI 了。不能脱壳，只能在 LOADER 上做文章了。

最直接的想法就是模拟 `explorer` 启动它，然后再修改。整个思路是这样：

1. 创建一个 `Event`，用于进程间同步。
2. 将 `DLL` 注入到 `explorer` 中。
3. Loader 调用 `WaitForObject` 进入等待状态。
4. `DLL` 的 `DLL_PROCESS_ATTACH` 中创建一线程，用 `CreateProcess` 启动目标程序，这地方用 `CREATE_SUSPENDED` 标志，免得跑飞了。
5. `DLL` 中用 `OpenEvent` 找到上面 `Event`，通知一次，激活 LOADER 线程。
6. LOADER 查找目录程序，然后不断的 `SUSPEND` 目标进程，查看要修改的地方有没有已解码，如果已解码，PATCH，没有，将目标进程恢复运行一小段时间。

源代码中没有什么注释，这儿帖出部分写上注释：

#### PATCH 代码段如下

```c
//////////////////////////////////////////////////////////////////////////
/**
 *  dwProcessID : Process ID which wanted to patch
 *
 *  return      : return TRUE if patch successfully ,otherwise return FALSE
 */
#define PATCH_ADDRESS   0x401019      //目标进程中要 PATCH 的地址
#define PATCH_SIZE      16           //从目标进程中读取的字节数
#define KEY_CODE        ("\\x72\\x12\\x68\\x3C\\x60\\x40\\x00\\x6A\\x00\\xFF\\x15\\xA0\\x50\\x40\\x00\\x33") //这个是壳解码后没有改动的程序代码
#define NEW_CODE        ("\\xEB\\x12\\x68\\x3C\\x60\\x40\\x00\\x6A\\x00\\xFF\\x15\\xA0\\x50\\x40\\x00\\x33") //要修改后的代码
#define TRY_TIMES       400          //只试搜索 N 次，如果还没出现，就不再查找了。

BOOL CrackIt(DWORD dwProcessID)
{
    BOOL bContinueRun=TRUE;
    BOOL bPatchSucess =FALSE;
    DWORD dwTryTimes=0;
    DWORD dwOldProtection,dwDummy;
    HANDLE hProcess =OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessID);
    VirtualProtectEx (hProcess, (LPVOID)PATCH_ADDRESS, PATCH_SIZE,PAGE_EXECUTE_READWRITE, &dwOldProtection);
    //如果 PATCH 成功，设置 bContinue 为假，或尝试了 N 次后还不能找到要修改的代码，放弃了。
    while(bContinueRun && ++dwTryTimes<TRY_TIMES)
    {
      BYTE OldKeyCode[17];
      //让目标程序运行 3ms 然后停下来，此时读目标程序的进程，查看是否已解码了
      SuspendResumeProcess(dwProcessID,FALSE);
      Sleep(3);
      SuspendResumeProcess(dwProcessID,TRUE);
      //读目标进程要 PATCH 地址处的数据
      ReadProcessMemory(hProcess, (LPVOID)PATCH_ADDRESS, OldKeyCode, PATCH_SIZE,&dwDummy);
      //判断是不是完全解码出来了
      if( !memcmp(OldKeyCode,(BYTE \*)KEY_CODE, 16) ){
        //相同，说明已解码了，写入我们的新值
        WriteProcessMemory(hProcess, (LPVOID)PATCH_ADDRESS, NEW_CODE, PATCH_SIZE,&dwDummy);
        //we have patched ,stop check :-)
        bPatchSucess=TRUE;
        bContinueRun=false;
      }
    }
    VirtualProtectEx(hProcess, (LPVOID)PATCH_ADDRESS, PATCH_SIZE,dwOldProtection, &dwDummy);
    //for debug
    if(dwTryTimes >= TRY_TIMES)
      OutputDebugString("Cannot Match Code In Program !\\n");
    //恢复目标程序执行
    SuspendResumeProcess(dwProcessID,FALSE);
    return bPatchSucess;
}
```

另外，在创建目标进程时不能写入绝对路径，不知道什么原因，后来通过 `SetCurrentDirectory` 改变工作目录解决了这个问题：

```c
#define WORK_DIRECTORY          "C:\\\\test"
#define CRACK_PROGRAM_NAME      "test.exe"  //这地方如果写成 c:\\\\test\\\\test.exe 传给 CreateProcess 尽管目标程序运行，但 explorer 会出错.不明白

DWORD WINAPI StartProcess(LPVOID lpParam)
{
    STARTUPINFO si = { sizeof(si) };
    PROCESS_INFORMATION pi;
    si.dwFlags = STARTF_USESHOWWINDOW;
    si.wShowWindow = TRUE;
    SetCurrentDirectory(WORK_DIRECTORY); //IMPORTANT!!
    BOOL bRet = CreateProcess(NULL, CRACK_PROGRAM_NAME,
      NULL, NULL, FALSE,  CREATE_SUSPENDED,
      NULL, NULL, &si,  &pi);
    if(bRet){
      CloseHandle (pi.hThread);
      CloseHandle (pi.hProcess);
    }
    HANDLE hEvent = OpenEvent(EVENT_ALL_ACCESS,FALSE,EVENT_OBJECT_NAME);
    SetEvent(hEvent);
    return 0;
}
```

注入到 `explorer` 的代码(感谢 CSDN 上的 `kesummer` 帮助)：

```c
BOOL CInjector::InjectModuleInto(DWORD dwProcessId)
{
    //不要给自己注入
    if(GetCurrentProcessId() == dwProcessId)
      return FALSE;
    //下面一段代码用于在 explorer 中查找 dll 是不是已经注入过了。
    BOOL bFound = FALSE;
    MODULEENTRY32 me32 = { 0 };
    me32.dwSize = sizeof(MODULEENTRY32);
    HANDLE hModuleSnap = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, dwProcessId);
    if(Module32First(hModuleSnap, &me32)) {
      do{
        if(lstrcmpiA(me32.szExePath, m_szDllName) == 0) {
          bFound = TRUE;
          break;
        }
      }
      while(Module32Next(hModuleSnap, &me32));
    }
    CloseHandle(hModuleSnap);
    if(bFound)  return FALSE;
    HANDLE hProcess = OpenProcess(
      PROCESS_VM_WRITE|PROCESS_CREATE_THREAD|PROCESS_VM_OPERATION,
      FALSE, dwProcessId);
    if(hProcess == NULL) return FALSE;
    int cbSize = (strlen(m_szDllName) + 1);
    LPVOID lpRemoteDllName = VirtualAllocEx(hProcess, NULL, cbSize, MEM_COMMIT, PAGE_READWRITE);
    WriteProcessMemory(hProcess, lpRemoteDllName, m_szDllName, cbSize, NULL);
    HMODULE hModule=GetModuleHandle("kernel32.dll");
    LPTHREAD_START_ROUTINE pfnStartRoutine =
        (LPTHREAD_START_ROUTINE)GetProcAddress(hModule, "LoadLibraryA");
    //用远程进程的 LoadLibraryA 函数的入口点做为远程线程的入口点，线程运行就相当于运行了 LoadLibraryA
    HANDLE hRemoteThread = CreateRemoteThread(hProcess, NULL, 0, pfnStartRoutine, lpRemoteDllName, 0, NULL);
    if(hRemoteThread == NULL)
    {
      CloseHandle(hProcess);
      return FALSE;
    }
    WaitForSingleObject(hRemoteThread, INFINITE);
    CloseHandle(hRemoteThread);
    CloseHandle(hProcess);
    m_dwProcessId=dwProcessId;
    return TRUE;
}
```

附件为全部源代码

---

### 经验总结

如果没有 Loader 检测，不需要注入到 `explorer` 了，直接用代码 `CreateProcess` 然后用上面的 `CrackIt` 函数就可以解决问题了。

有检测只好找办法绕过去了。不知道有没有更简单的方法。

本文是写给跟我一样菜鸟的，搞不定壳的，高手就别笑了。

---

【版权声明】 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 11 月 24 日 14:47:43

好像 `heXer` 曾经说过的

貌似对 `DEBUG` 方式的无效

附件下载

>

>
