
# 不需要依赖 `dllmain` 触发的 CE 注入代码

> 原文：[`www.pediy.com/kssd/pediy10/85899.html`](https://www.pediy.com/kssd/pediy10/85899.html)

首先感谢宇宙青年 `Yonsm` 提供的例子，源代码我也下来看了，不过多了点-\_-最后还是自己写了个简洁的。

代码同时参考了 `Yonsm` 和一篇国外的文章，已经实际测试能用。

以下代码演示将`test.dll`注入电话进程并触发`test.dll`里的导出函数`HelloWorld`

首先是注入的`exe`：

代码:

```
//取得 CProg.exe 进程句柄的 OpenProcess 就不贴了，大家都知道@@

//卸载钩子函数
bool UninstallHook(HANDLE hProcessDest,HINSTANCE hInst){
  CALLBACKINFO ci;
  ci.hProcess = hProcessDest;
  ci.pFunction = (FARPROC)MapPtrToProcess(GetProcAddress(GetModuleHandle(L"COREDLL"), L"FreeLibrary"), hProcessDest);
  ci.pvArg0 = hInst; //HINSTANCE returned by LoadLibrary
  DWORD dw = PerformCallBack4(&ci, 0,0,0); //returns 1 if correctly unloaded
  return (bool)dw;
}

//安装钩子
bool InstallHook( HANDLE hProcessDest )
{
  BOOL bMode = SetKMode(TRUE);
  DWORD dwPerm = SetProcPermissions(0xFFFFFFFF);

  CALLBACKINFO ci;

  ci.hProcess = hProcessDest;
  ci.pFunction = (FARPROC)GetProcAddress(GetModuleHandle( \_T("coredll.dll") ),\_T("LoadLibraryW") );
  ci.pvArg0 = MapPtrToProcess(\_T("test.dll"),GetCurrentProcess());    //先注入 dll

  HINSTANCE hInst = (HINSTANCE) PerformCallBack4(&ci,0,0,0);
  if ( 0 == GetLastError())
  {
    //MessageBox(NULL,TEXT("Success inje"),TEXT("success"),MB\_OK);// (NULL,TEXT("PerformCallBack4() run successful......\n",TEXT("test"),MB\_OK));
    //get the proc address
    FARPROC pHook = GetProcAddress(hInst, (LPCTSTR)L"HelloWorld"); //关键的地方！获取注入 dll 的函数地址
    ci.hProcess = hProcessDest;
    ci.pFunction = (FARPROC)MapPtrToProcess(pHook, hProcessDest);
    ci.pvArg0 = NULL; 
    DWORD dw = PerformCallBack4(&ci, 0, 0, 0); //再次注入！这次是函数地址！然后相关的导出函数就运作了
    //UninstallHook(hProcessDest,hInst);
    SetKMode(bMode);
    SetProcPermissions(dwPerm);
    return (bool)dw;
  }else{
    LPWSTR tt;
    wsprintf(tt,TEXT("GetLastError:%d"),GetLastError());
    MessageBox(NULL,tt,TEXT("fail"),MB\_OK);
  }
  SetKMode(bMode);
  SetProcPermissions(dwPerm);
  return false;
}

```

`DLL`的导出代码：

代码:

```
extern "C" \_\_declspec(dllexport) bool WINAPI HelloWorld()
{
  MessageBox(NULL,TEXT("Hello World by 小金"),TEXT("success"),MB\_OK);
  return true;
}

```
