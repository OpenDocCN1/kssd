
# DebugActiveProcess 杀进程

> 原文：[`www.pediy.com/kssd/pediy10/96973.html`](https://www.pediy.com/kssd/pediy10/96973.html)

发现个比较 WS 的东西。

代码:

```
#include <windows.h>
#include <iostream>
using namespace std;

//同归于尽杀进程
//By Fypher
//http://hi.baidu.com/nmn714

void RaisePrivileges(){
  HANDLE hToken;
  TOKEN_PRIVILEGES tkp;
  tkp.PrivilegeCount=1;
  tkp.Privileges[0].Attributes=SE_PRIVILEGE_ENABLED;

  if(OpenProcessToken(GetCurrentProcess(),TOKEN_ALL_ACCESS,&hToken)){
    if(LookupPrivilegeValue(NULL,SE_DEBUG_NAME,&tkp.Privileges[0].Luid)){
      AdjustTokenPrivileges(hToken,FALSE,&tkp,0x10,(PTOKEN_PRIVILEGES)NULL,0);
    }
  }
  if(hToken)
    CloseHandle(hToken);
}

int main(){
  RaisePrivileges();
  DWORD pid;
  cout<<"咱哥俩和谐谁去？PID：";
  cin>>pid;
  if(!DebugActiveProcess(pid)){
    cout<<"它太强大了，咱和谐不了"<<endl;
  }
  system("pause");
  return 0;
}
```
