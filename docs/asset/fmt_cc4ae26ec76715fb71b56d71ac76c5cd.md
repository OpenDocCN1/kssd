
# 优化 C++ 程序编译效率的实例

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-811.htm`](https://www.pediy.com/kssd/pediy07/pediy7-811.htm)

1. 自定义程序的入口
2. 合并区段
3. 不用调 `malloc`, `free` 等函数
4. 不用 `cout` 来输出, `cin` 来输入
5. 如果调用了很多库函数如 `memset`, `stycpy` 等等的话，请导入 `msvcrt.lib`, 不然编译器会在程序里面导入静态库，这样的话程序就会大很多。
6. `window` 编程不用 `mfc`...

减少程序运行内存占用量可以调用 `SetProcessWorkingSetSize(GetCurrentProcess(),-1,-1);`

```cpp
//----------sample.cpp---------------------

#include <windows.h>
#include "sample.h"
#include "mydll.h"
#include "resource.h"

// 这下面自定义函数入口
#pragma comment(linker, "/ENTRY:EntryPoint")
#pragma comment(linker,"/ALIGN:0x400")

// 设置区段属性, 跟区段在内存起始地址
// 这里面要加写入的权限, 不然程序就运行不了了
// E 为执行，R 为可读, W 为可写
// 更多的说明请参见 msdn
#pragma comment(linker,"/SECTION:.text,ERW /ALIGN:0x1000")

// 下面合并区段
#pragma comment(linker,"/merge:.data=.text")
#pragma comment(linker,"/merge:.rdata=.text")

// 下面导入函数
#pragma comment(lib,"mydll.lib")

// 下面是函数的入口
// 得到 WinMain 里面的几个参数
// HINSTANCE hInstance=GetModuleHandle(NULL)
// LPSTR lpCmdLine= GetCommandLine()
// int nCmdShow 这个可以自己填

void EntryPoint()
{
  HINSTANCE hInstance;
  hInstance=GetModuleHandle(NULL);
  Sample pro(hInstance);
  DialogBoxParam(hInstance, MAKEINTRESOURCE(IDD_DIALOG), NULL, (DLGPROC)Sample::DialogProc, (LPARAM)&pro);
}

INT_PTR CALLBACK Sample::DialogProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
  static Sample *pro;
  switch(message)
  {
  case WM_INITDIALOG:
    pro=(Sample *)lParam;
    pro->hwnd=hwnd;
    pro->Init();
    break;
  case WM_COMMAND:
    pro->WndPro_Command(wParam, lParam);
    break;
  case WM_CLOSE:
    pro->WndPro_Close();
    break;
  case WM_TIMER:
    // 这里每秒钟触发一次
    // 来减少内存占用量
    // 试着在任务管理器里面看一下，sample.exe 内存占用量是不是只有 200 多 k，在我这里是这样的
    if(wParam==100)
    {
      SetProcessWorkingSetSize(GetCurrentProcess(),
        -1, -1);
    }
    break;
  }
  return 0;
}

Sample::Sample(HINSTANCE hi)
{
  this->hInstance=hi;
}

int Sample::WndPro_Command(WPARAM wParam, LPARAM lParam)
{
  switch(wParam)
  {
  case IDOK:
    this->LoadDll();
    break;
  case IDCANCEL:
    this->WndPro_Close();
    break;
  }
  return 0;
}

int Sample::WndPro_Close()
{
  EndDialog(hwnd, 0);
  return 0;
}

int Sample::Init()
{
  SetTimer(hwnd, 100, 100, NULL);
  return 0;
}

int Sample::LoadDll()
{
  test(hwnd);
  return 0;
}

UINT malloc_(size_t num)
{
  return (UINT)VirtualAlloc(NULL, num, MEM_RESERVE |MEM_COMMIT, PAGE_EXECUTE_READWRITE);
}

void free_(void * p)
{
  VirtualFree(p, NULL, MEM_RELEASE);
}

void ZeroMem(char * mem, int len)
{
  for(int i=0; i<len; i++)
    mem[i]=0;
}
```

```cpp
//-------------------------------
//----------sample.h--------

// 下面是主函数的类
class Sample
{
public:
  static INT_PTR CALLBACK DialogProc(HWND, UINT, WPARAM, LPARAM);
  Sample(HINSTANCE);
private:
  HINSTANCE hInstance;
  HWND hwnd;
  int WndPro_Command(WPARAM, LPARAM);
  int WndPro_Close();
  int Init();
  int LoadDll();
};

///--> 下为精简的库函数

// 分配内存最好不用库函数 malloc,
// 不要用 cout 来输出
// 如果你调用了很多库函数的话
// 请在导入 msvcrt.lib

UINT malloc_(size_t);// 分配内存
void free_(void *);// 释放内存
void ZeroMem(char *, int);// 清零

//-------------------------------
```

```cpp
//---------mydll.cpp-----------

#include <windows.h>
#include "mydll.h"

#pragma comment(linker, "/ENTRY:DllEntry")
#pragma comment(linker,"/ALIGN:0x400")
#pragma comment(linker,"/SECTION:.text,ERW /ALIGN:0x1000")
#pragma comment(linker,"/merge:.data=.text")
#pragma comment(linker,"/merge:.rdata=.text")

BOOL APIENTRY DllEntry()
{
  return 1;
}

INT APIENTRY test(HWND hwnd)
{
  MessageBox(hwnd, "调用 dll", " ", 0);
  return 0;
}

//-----------------------
```

下面是程序完整的代码。

附件: `sample.rar`

还能加些

```cpp
#pragma comment(linker, "/Filealign:0x200")
#pragma comment(linker, "/OPT:REF")
#pragma comment(linker, "/OPT:ICF")
#pragma optimize("gsy", on)

#pragma comment(linker, "/Filealign:0x200")
```

在 `cl` 里面，没这个选项。

用下面这个，但是加了没用的，除非你的区段只有一个

```cpp
#pragma comment(linker, "/align:0x200")
```

下面的选项我还没用过, `![](img/6d0f69ca3b0526abc5f42199a1f1f0e8.jpg)`

```cpp
#pragma comment(linker, "/OPT:REF")
#pragma comment(linker, "/OPT:ICF")
#pragma optimize("gsy", on)
```

我编译一般用命令行的，在 `.net 2003` 跟 `.net 2005` 里面的有好多选项连去都去不掉

```bash
cl   mydll.cpp mydll.def /link /dll /out:mydll.dll /machine:x86 /subsystem:windows user32.lib kernel32.lib
rc sample.rc
cl sample.cpp /link /machine:x86 user32.lib kernel32.lib /subsystem:windows sample.res
del sample.obj sample.res mydll.obj mydll.exp
pause
```

> 引用:
>
> * * *
>
> *最初由 FlyToTheSpace 发布*
>
> **(1) 自定义程序的入口
> (2) 合并区段
> (3) 不用调 malloc, free 等函数
> (3) 不用 cout 来输出, cin 来输入
> (4) 如果调用了很多库函数如 memset, stycpy 等等
> ........**
>
> * * *

`SetProcessWorkingSetSize` 只是给你感观上的内存变小而已, 如果你弄个 hook 进入 `photoshop` 这样的程序去做这件事, 你就会马上发现为什么别人并不怎么用 `SetProcessWorkingSetSize`。

`SetProcessWorkingSetSize` 只是把内存写回磁盘, 当用到时又要从磁盘加载回来, 程序一大, 内存量本身占用大, 你次读写磁盘那么慢, 你就会发现虽然内存占用量下去了, 但是硬盘一真转个不停, 程序基本上动不了...

所以如何节内存, 不应从 `SetProcessWorkingSetSize` 下手, 当然现在你写的小程序可以例外. 其实如果有个界面, 占用个 1M 的内存实在是小事, 不必要去计较.

`SetProcessWorkingSetSize` 这个我是跟踪 `realplay` 里面一个程序 `realsched.exe` 发现的，其它减少内存用量的方法，还没找到 `![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg)`。

对于那些大一点的程序，就不能用 `settimer` 每隔段时间调用，当程序的界面最小化时，调用一下就可以了，比如 `potoshop` 在显示界面的时候，占用内存会很大，但只要你将它最小化内存占用量马上就减下来了，很多大一点的程序都是这样的。

`cl.exe` 有许多关于程序代码优化的开关：

```text
/////////////////////////////////////////////////////////
                   -OPTIMIZATION-
/O1 minimize space      /Op[-] improve floating-pt consistency
/O2 maximize speed      /Os favor code space
/Oa assume no aliasing  /Ot favor code speed
/Ob<n> inline expansion (default n=0)    /Ow assume cross-function aliasing
/Od disable optimizations (default)      /Ox maximum opts. (/Ogityb1 /Gs)
/Og enable global optimization           /Oy[-] enable frame pointer omission
/Oi enable intrinsic functions

                  -CODE GENERATION-
/G3 optimize for 80386               /Gy separate functions for linker
/G4 optimize for 80486               /Ge force stack checking for all funcs
/G5 optimize for Pentium             /Gs[num] disable stack checking calls
/G6 optimize for Pentium Pro         /Gh enable hook function call
/GB optimize for blended model (default) /GR[-] enable C++ RTTI
/Gd __cdecl calling convention       /GX[-] enable C++ EH (same as /EHsc)
/Gr __fastcall calling convention    /Gi[-] enable incremental compilation
/Gz __stdcall calling convention     /Gm[-] enable minimal rebuild
/GA optimize for Windows Application /EHs enable synchronous C++ EH
/GD optimize for Windows DLL         /EHa enable asynchronous C++ EH
/Gf enable string pooling            /EHc extern "C" defaults to nothrow
/GF enable read-only string pooling  /QIfdiv[-] enable Pentium FDIV fix
/GZ enable runtime debug checks      /QI0f[-] enable Pentium 0x0f fix
```

我有中文版的编译器: `![](img/27037aad577b8dcca17d45fd7d02bd99.jpg)`

```text
Microsoft (R) 32-bit C/C++ Optimizing Compiler Version 13.10.3077 for 80x86
Copyright (C) Microsoft Corporation 1984-2002. All rights reserved.

                    C/C++ 编译器选项
                       -优化-
/O1 最小化空间                  /Op[-] 改善浮点数一致性
/O2 最大化速度                  /Os 优选代码空间
/Oa 假设没有别名                /Ot 优选代码速度
/Ob<n> 内联展开(默认 n=0)       /Ow 假设交叉函数别名
/Od 禁用优化(默认值)            /Ox 最大化选项。(/Ogityb2 /Gs)
/Og 启用全局优化                /Oy[-] 启用框架指针省略
/Oi 启用内部函数

                  -代码生成-
```
