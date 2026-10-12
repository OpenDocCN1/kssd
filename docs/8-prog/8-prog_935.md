
# 在异常处理函数中同步线程的问题

> 原文：[`www.pediy.com/kssd/pediy12/139634.html`](https://www.pediy.com/kssd/pediy12/139634.html)

```c
#include<stdio.h>
#define _WIN32_WINNT 0x0500
#include<windows.h>
#include<tchar.h>
#include<Tlhelp32.h>

HANDLE hevent,hevent2;

/*
ZwMapViewOfSection
76E44ED0 Z>  B8 A8000000           MOV EAX,0A8
76E44ED5   BA 0003FE7F           MOV EDX,7FFE0300
76E44EDA   FF12               CALL DWORD PTR DS:[EDX]
76E44EDC   C2 2800             RETN 28
*/

//注意---硬编码！
PVOID addr = 0x76E44EDC;//ZwMapViewOfSection 中 retn 28 的地址

DWORD myThread( PVOID context )
{
    while( 1 ){
        WaitForSingleObjectEx( hevent,INFINITE,TRUE );
        printf("ok\n");
        MessageBox(NULL,_T("thread exit!"),_T("text"),MB_OK );
    }
}

int main()
{
    HMODULE hMod;
    HANDLE hThread;
    DWORD oldpt;

    hevent = CreateEvent( NULL,FALSE,FALSE,NULL );
    hevent2 = CreateEvent( NULL,FALSE,FALSE,NULL );

    hThread = CreateThread( NULL,0,(LPTHREAD_START_ROUTINE)myThread,0,0,NULL);

    MessageBox( NULL,_T("1"),_T("OK"),MB_OK );

    //在 ZwMapViewOfSection 的 retn 28 地址处写入 int 3 断点
    VirtualProtect( addr,1,PAGE_EXECUTE_READWRITE,&oldpt );
    *(PBYTE)addr = 0xcc;

    getch();

    __try{
    LoadLibrary(_T("123.dll"));     //触发断点异常,任意加载一个 dll 即可
    }
    __except(
        *(PBYTE)addr = 0xc2,            //恢复原始代码
        SetEvent( hevent ),            //设置 myThread 中等待的事件
        WaitForSingleObject( hevent2,INFINITE),//使当前线程处于等待状态
    EXCEPTION_CONTINUE_EXECUTION ){;}

    getch();
    return 0;
}

/*
myThread 中的 WaitForSingleObject 不返回，不知为何？
我做了些实验，好像那个断点位置有些特殊。具体什么原因导致出现上述情况就不明白了。
还有，要是去掉异常处理中那条等待语句，使异常线程恢复运行的话，一切工作正常。
*/
```

给你改了改:

1. 选择 `ZwMapViewOfSection` 不合适, `LdrLoadDll` 开始就有 SEH, 你的`__except` block 得不到执行，被内部的 SEH filter 给吃掉了。
2. 处理 int3 的重新执行要调节 EIP。

```c
#include "stdafx.h"
#include<stdio.h>
#include <conio.h>
#include<windows.h>
#include<tchar.h>

HANDLE hevent,hevent2;
HMODULE DllHandle;
PUCHAR Address;

DWORD myThread( PVOID context )
{
    WaitForSingleObjectEx(hevent,INFINITE,TRUE);
    printf("myThread is signaled! \n");
    printf("singal main thread to run ... \n");
    SetEvent(hevent2);
    return 0;
}

LONG WINAPI ExceptionFilter(
  __in struct _EXCEPTION_POINTERS* ExceptionInfo
  )
{
  //
  // don't care about other exception conditions.
  //
  printf("Enter ExceptionFilter \n");
  if (*Address == (UCHAR)0x8b) {
    return EXCEPTION_CONTINUE_SEARCH;
  }
  if (*Address != (UCHAR)0xcc) {
    return EXCEPTION_CONTINUE_SEARCH;
  }
  //
  // recover old opcode
  //
    *Address = 0x8b,
  //
  // adjust EIP to old value to re-execute
  //
  ExceptionInfo->ContextRecord->Eip = (DWORD)(Address - 1);
    SetEvent(hevent);
    WaitForSingleObject(hevent2, INFINITE);
  printf("Exit ExceptionFilter \n");
  return EXCEPTION_CONTINUE_EXECUTION;
}

int main(int argc, TCHAR *argv)
{
    HANDLE hThread;
  ULONG OldProtect;
    hevent = CreateEvent( NULL,FALSE,FALSE,NULL );
    hevent2 = CreateEvent( NULL,FALSE,FALSE,NULL );
    hThread = CreateThread( NULL,0,(LPTHREAD_START_ROUTINE)myThread,0,0,NULL);
  DllHandle = LoadLibrary(_T("kernel32.dll"));
  Address = (PUCHAR)GetProcAddress(DllHandle, "LoadLibraryW");
  printf("LoadLibrary address: 0x%08x \n", Address);
    VirtualProtect(Address, 1, PAGE_EXECUTE_READWRITE, &OldProtect);
    *(PUCHAR)Address = (UCHAR)0xcc;
    putch(getch());
  printf("\n");
    __try{
      DllHandle = LoadLibrary(_T("mshtml.dll"));
    printf("LoadLibrary is called : 0x%08x \n", DllHandle);
    }
  __except(ExceptionFilter(GetExceptionInformation())) {
  }
  getch();
    return 0;
}
```
