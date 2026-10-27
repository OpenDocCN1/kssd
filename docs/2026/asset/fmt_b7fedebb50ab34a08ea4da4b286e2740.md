
至此，`_LdrpInitializeProcess`已经接近返回，Shim Engine的Ldr初始化也结束了，剩下就是进程运行过程中的Dll Load and Unload，`apphelp`和`AcLayers`会实时接到通知，进行Hook处理。

附带一下，`LdrInitializeThunk`返回后：

`ZwContinue->RtlUserThreadStart->RtlInitializeExceptionChain(TOP SEH)->ntdll_offset_000368A3(var_Kernel32ThreadInitThunkFunction)->kernel32!BaseThreadInitThunk->exe!ModuleEntryPoint->RtlExitUserThread`

## 2）`LdrInitShimEngineDynamic`

你可以在`ntdll`还没有初始化SE的Dll的时候（比如启动时ShellCode改`Eip`注入Dll），执行`LdrInitShimEngineDynamic`函数，可以让你的Dll接收到Shim Engine的通知，前提是你的Dll导出上面列表中的函数，并且`LdrInitShimEngineDynamic`这个`ntdll`导出函数在Win7和Win8下竟然参数不同：

```
NTSTATUS NTAPI LdrInitShimEngineDynamic(HMODULE hModule,PLDR_DATA_TABLE_ENTRY pLdrEntry) //win8
NTSTATUS NTAPI LdrInitShimEngineDynamic(HMODULE hModule) //win7
```

其的实现也很简单，仅仅是判断有没有SE的Dll已经载入，没有就执行一下上面说到的`GetInterface`函数：

![](https://bbs.kanxue.com/upload/attach/201307/402913_vjl9lf8x9gv69d4.gif)

## 3）利用Shim Engine来Dll注入

这个我相信有不少人已经在用了，其实就是在`pShimData`中写入我们的Dll文件，并且模拟成一个SE的Dll，`ntdll`会跟普通载入dll那样载入。（Win8下你不导出函数也行，`DllMain`一样得到执行）

Dll源代码在下面，把`Win32Protect6.dll`放到C盘下，执行exe即可。（仅在win8系统有效，我现在没win7了）

![](https://bbs.kanxue.com/upload/attach/201307/402913_pc3dws3jes9deyu.jpg)

exe源代码（工程删了）：

```
typedef struct _PROCESS_BASIC_INFORMATION{
  NTSTATUS ExitStatus;
  PVOID PebBaseAddress;
  ULONG_PTR AffinityMask;
  LONG BasePriority;
  HANDLE UniqueProcessId;
  HANDLE InheritedFromUniqueProcessId;
}PROCESS_BASIC_INFORMATION,*PPROCESS_BASIC_INFORMATION;

void main()
{
  STARTUPINFO si = {};
  si.cb = sizeof(STARTUPINFO);
  PROCESS_INFORMATION pi;
  CreateProcessW(L"C:\\Windows\\notepad.exe",NULL,NULL,NULL,FALSE,CREATE_SUSPENDED,NULL,NULL,&si,&pi);
  typedef NTSTATUS (NTAPI* fnNtQueryInformationProcess)(HANDLE,ULONG,PVOID,ULONG,PULONG);
  PROCESS_BASIC_INFORMATION pbi;
  ((fnNtQueryInformationProcess)GetProcAddress(GetModuleHandleA("ntdll.dll"),"NtQueryInformationProcess"))(pi.hProcess,0,&pbi,sizeof(PROCESS_BASIC_INFORMATION),NULL);
  LPVOID lpShimData = (LPVOID)((ULONG_PTR)pbi.PebBaseAddress + 0x1E8);
  PVOID pShimData = NULL;
  ReadProcessMemory(pi.hProcess,lpShimData,&pShimData,sizeof(PVOID),NULL);
  LPWSTR lpszDllFile = L"C:\\Win32Project6.dll";
  if (pShimData) WriteProcessMemory(pi.hProcess,pShimData,lpszDllFile,lstrlenW(lpszDllFile) * sizeof(wchar_t),NULL);
  ResumeThread(pi.hThread);
  CloseHandle(pi.hProcess);
  CloseHandle(pi.hThread);
  ExitProcess(0);
}
```

## 4）利用Shim Engine来Dll劫持（win8测试通过）

打开某个exe的文件属性，选择兼容性选项卡，勾选使用兼容性模式运行，下拉框随便选一个系统，确定保存。

进入`C:\Windows\AppPatch`下面找到`AcLayers.dll`，改名，比如改成`AcLayers.dl_`。(需要管理员取得所有权)

自己写一个Dll，名称就是`AcLayers.dll`，放到AppPatch下面，Dll需要导出下面这2个名称的函数：

```
PVOID WINAPI GetHookAPIs(LPCSTR lpszVerb,LPCWSTR lpszTagId,PVOID lpHookAPIs)
PVOID WINAPI NotifyShims(DWORD dwNotifyType,PVOID pvParameter)
```

`DllMain`里面判断载入我们的Dll的进程是`***.exe`，就进行`LoadLibrary`核心Dll，balalalala...，然后把`GetHookAPIs`和`NotifyShims`这2个导出函数直接`return 0`。（不使用兼容性Hook，不过`NotifyShims`还是可以接到Dll和Load和Unload事件，附加福利啊haha~）

如果不是，`LoadLibrary("AcLayers.dl_")`，把`GetHookAPIs`和`NotifyShims`这2个导出函数`JMP`到`AcLayers.dl_`的上面。（实现其他程序的兼容性模式设置）

顺便附带上32和64的win8 `apphelp.dll`的导出函数lib文件。

小弟最近失眠得厉害，也就暑假回家了才有时间整理这篇文章出来，如果哪里写得不好，希望大家海涵~

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

## 上传的附件：

- [LdrInitShimEngineDynamic.gif](javascript:void(0);)（11.94kb，10次下载）
- [LdrpInitShimEngine.gif](javascript:void(0);)（6.20kb，19次下载）
- [LdrpInitShimEngine2.gif](javascript:void(0);)（3.17kb，19次下载）
- [LdrpInitShimEngine3.gif](javascript:void(0);)（11.12kb，19次下载）
- [LdrpInitShimEngine4.gif](javascript:void(0);)（12.69kb，19次下载）
- [LdrpInitShimEngine5.gif](javascript:void(0);)（11.68kb，18次下载）
- [LdrpInitShimEngine6.gif](javascript:void(0);)（10.53kb，12次下载）
- [LdrpInitShimEngine7.gif](javascript:void(0);)（19.95kb，13次下载）
- [LdrpInitShimEngine8.gif](javascript:void(0);)（12.20kb，6次下载）
- [LdrpInitShimEngine9.gif](javascript:void(0);)（12.34kb，6次下载）
- [LdrpInitShimEngine10.gif](javascript:void(0);)（8.82kb，7次下载）
- [LdrpInitShimEngine11.gif](javascript:void(0);)（9.04kb，6次下载）
- [未命名.jpg](javascript:void(0);)（117.96kb，20次下载）
- [Win32Project6.rar](javascript:void(0);)（5.97kb，176次下载）
- [apphelp_lib.zip](javascript:void(0);)（18.98kb，142次下载）
- [LdrpInitShimEngine12.gif](javascript:void(0);)（4.87kb，7次下载）
- [2013-07-15_131929.gif](javascript:void(0);)（6.83kb，7次下载）
