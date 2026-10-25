
# Ring3 下 Inline Hook MessageBox(演示)

> 原文：[`www.pediy.com/kssd/pediy10/69666.html`](https://www.pediy.com/kssd/pediy10/69666.html)

这是学习 Ring3 下 Inline Hook 的代码，写的很菜，部分代码参考了邪 8 中“我非我”的代码，在此表示感谢。

原文中有二处错误，后面好多人都说运行不成功，不知道原作者是怎样编译成功的，原代码参见[ring3 inline hook demo](https://forum.eviloctal.com/thread-28859-1-1.html)

由于在邪 8 中等级太低，不能发贴，借用宝地说明下

提前声明下，我调试的 `VC++` 为 6.0 的，

在 `VC++` 编译器生成 `EXE` 中，函数也像 API 一样进行了中间跳转

如下

```
0040100F   $ /E9 CC020000   jmp     MyFunc

00401014   $ |E9 37000000   jmp     main

00401019   $ |E9 B2000000   jmp     Hook
```

在 `40100F` 中，原文中的意思是此处即为 `MyFunc` 的入口地址，但此处只为一中转跳转，要计算真正函数的入口地址，还要再次进行重定位。另外一处，原文开头为替换为 5 个字节，在这里正好将一条完整的指令拆开了，这是在调试过程中发现的，正确的在原文中应替换 6 个字节，这就不详说了，有不对的地方，希望指正

代码:

```c
#include "stdafx.h"
#include <windows.h>
#include <stdio.h>
#include <iostream.h>
//using namespace std;

DWORD head;//保存 API 返回地址
int nRet;
BYTE orig_code[5] = {0x90, 0x90, 0x90, 0x90, 0x90};//存放原始的指令
BYTE hook_code[5] = {0xe9, 0, 0, 0, 0};//存放跳转到 MyMessageBoxA 的指令
BYTE jmp_org_code[5] = {0xe9, 0, 0, 0, 0};//存放跳转到原起始地址后 5 字节的指令

int MyMessageBoxA(
        HWND hWnd,          // handle to owner window
        LPCTSTR lpText,     // text in message box
        LPCTSTR lpCaption,  // message box title
        UINT uType          // message box style
        );
int MyMessageBoxAA(
          HWND hWnd,          // handle to owner window
          LPCTSTR lpText,     // text in message box
          LPCTSTR lpCaption,  // message box title
          UINT uType          // message box style
        );
int MyFunc();
void Hook();
int jmp_back();

ULONG OldFuncAddr;
ULONG MyFuncAddr;
ULONG jmp_backAddr;

//在修改前几个字节时，注意：取出的指令为完整的

int main()
{
  Hook();

  int rt = MessageBoxA(NULL, "Hello World", "Title", MB_OK);
//  cout << rt << endl;//查看返回值是否已修改成功

  system("pause");

  return 0;
}

void Hook()
{
  DWORD dwOldProtect;   
  OldFuncAddr = (ULONG)MessageBoxA;

  // MyFuncAddr ＝ MyMessageBoxA 的实际地址
  MyFuncAddr = *(ULONG *)((BYTE *)MyMessageBoxA+1) + (ULONG)MyMessageBoxA + 5;
  // jmp_backAddr = jmp_back 的实际地址
  jmp_backAddr = *(ULONG *)((BYTE *)jmp_back+1) + (ULONG)jmp_back + 5;
    //修改内存为 PAGE_EXECUTE_READWRITE
  VirtualProtect((LPVOID)jmp_backAddr, 10, PAGE_EXECUTE_READWRITE, &dwOldProtect);

  VirtualProtect((LPVOID)OldFuncAddr, 5, PAGE_EXECUTE_READWRITE, &dwOldProtect);
  //计算跳转地址
  *((ULONG*)(hook_code+1)) = (ULONG)MyFuncAddr - (ULONG)OldFuncAddr - 5;
  
  memcpy(orig_code,(BYTE *)OldFuncAddr, 5);

  memcpy((BYTE*)OldFuncAddr, hook_code, 5);
  //计算返回地址
  *((ULONG*)(jmp_org_code+1)) = (ULONG)OldFuncAddr - (ULONG)jmp_backAddr - 5;

  memcpy((BYTE *)jmp_backAddr, orig_code, 5);

  memcpy((BYTE *)jmp_backAddr + 5, jmp_org_code, 5);
}

__declspec(naked) int jmp_back()
{
  __asm
  {
    _emit 0x90
    _emit 0x90
    _emit 0x90
    _emit 0x90
    _emit 0x90
    _emit 0x90
    _emit 0x90
    _emit 0x90
    _emit 0x90
    _emit 0x90
  }
}

//MyMessageBoxA：在函数执行前进行自己的处理
__declspec(naked) int MyMessageBoxA(
                     HWND hWnd,          // handle to owner window
                     LPCTSTR lpText,     // text in message box
                     LPCTSTR lpCaption,  // message box title
                     UINT uType          // message box style
                   )
{    
  printf("MyMessageBoxA is called\r\n"); 
   __asm
  {
    pop head
    pop hWnd
    pop lpText
    pop lpCaption
    pop uType
  }
  MyFunc();////可以加入函数过程
  __asm
  {
    //压栈过程
    push uType
    push lpCaption
    push lpText
    push hWnd
    push head
    //跳回 MessageBoxA 入口点
    jmp jmp_back;
    ret;
  }
}


//MyMessageBoxA：在函数执行后进行自己的处理
__declspec(naked) int MyMessageBoxAA(
                 HWND hWnd,          // handle to owner window
                 LPCTSTR lpText,     // text in message box
                 LPCTSTR lpCaption,  // message box title
                 UINT uType          // message box style
               )
{    
  printf("MyMessageBoxAA is called\r\n"); 
  __asm
  {
    pop head
    push offset s1;//返回地址为 S1:
    //跳回 MessageBoxA 入口点
    jmp jmp_back;
s1:    nop   
  }

  MyFunc();
  
  __asm
  {
    ;//将原返回地址压栈
    mov eax, 0;////演示：将返回结果改为 0，也可由 MyFunc 返回
    push head
    ret;
  }
}

int MyFunc()
{
  printf("Hello World\r\n");
  return 1;
}
```
