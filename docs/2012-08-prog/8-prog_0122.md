
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



# MSVC 编译器选项参考

## 优化与代码生成

* `/G3` 为 80386 进行优化
* `/G4` 为 80486 进行优化
* `/G5` 为 Pentium 进行优化
* `/G6` 对 PPro、P-II、P-III 进行优化
* `/G7` 对 Pentium 4 或 Athlon 进行优化
* `/GB` 为混合模型进行优化 (默认)
* `/Gr` `__fastcall` 调用约定
* `/Gz` `__stdcall` 调用约定
* `/GA` 为 Windows 应用程序进行优化
* `/Gf` 启用字符串池
* `/GF` 启用只读字符串池
* `/Gy` 分隔链接器函数
* `/GZ` 启用堆栈检查 (`/RTCs`)
* `/Ge` 对所有函数强制堆栈检查
* `/Gs[num]` 控制堆栈检查调用
* `/GS` 启用安全检查
* `/clr[:noAssembly]` 为公共语言运行库编译
    * `noAssembly` - 不产生程序集

## 架构与修复

* `/arch:<SSE|SSE2>` CPU 结构的最低要求，以下内容之一:
    * `SSE` - 启用支持 SSE 的 CPU 可用的指令
    * `SSE2` - 启用支持 SSE2 的 CPU 可用的指令
* `/QIfdiv[-]` 启用 Pentium FDIV 修复
* `/QI0f[-]` 启用 Pentium 0x0f 修复
* `/QIfist[-]` 使用 FIST 而不是 `ftol()`

## 输出文件

* `/Fa[file]` 命名程序集列表文件
* `/FA[sc]` 配置程序集列表
* `/Fd[file]` 命名 .PDB 文件
* `/Fe<file>` 命名可执行文件
* `/Fm[file]` 命名映射文件
* `/Fo<file>` 命名对象文件
* `/Fp<file>` 命名预编译头文件
* `/Fr[file]` 命名源浏览器文件
* `/FR[file]` 命名扩展 .SBR 文件

## 预处理器

* `/AI<dir>` 添加到程序集搜索路径
* `/FU<file>` 强制使用程序集/模块
* `/C` 不抽出注释
* `/D<name>{=|#}<text>` 定义宏
* `/E` 预处理到 stdout
* `/EP` 预处理到 stdout，没有 `#line`
* `/P` 预处理到文件
* `/FI<file>` 命名强制包含文件
* `/U<name>` 移除预定义宏
* `/u` 移除所有预定义宏
* `/I<dir>` 添加到包含搜索路径
* `/X` 忽略“标准位置”
* `/Fx` 将插入的代码合并到文件

## 语言

* `/Zi` 启用调试信息
* `/ZI` 启用“编辑并继续”调试信息
* `/Z7` 启用旧式调试信息
* `/Zd` 仅有行号调试信息
* `/Zp[n]` 在 n 字节边界上包装结构
* `/Za` 禁用扩展 (暗指 `/Op`)
* `/Zc:arg1[,arg2]` C++ 语言一致性，这里的参数可以是:
    * `forScope` - 对范围规则强制使用标准 C++
    * `wchar_t` - `wchar_t` 是本机类型，不是 `typedef`
* `/Ze` 启用扩展 (默认)
* `/Zl` 省略 .OBJ 中的默认库名
* `/Zg` 生成函数原型
* `/Zs` 只进行语法检查
* `/vd{0|1}` 禁用/启用 `vtordisp`
* `/vm<x>` 指向成员的指针类型

## 杂项

* `@<file>` 选项响应文件
* `/?`, `/help` 打印此帮助消息
* `/c` 只编译，不链接
* `/H<num>` 最大外部名称长度
* `/J` 默认 `char` 类型是 `unsigned`
* `/nologo` 取消显示版权消息
* `/showIncludes` 显示包含文件名
* `/Tc<source file>` 将文件编译为 .c
* `/Tp<source file>` 将文件编译为 .cpp
* `/TC` 将所有文件编译为 .c
* `/TP` 将所有文件编译为 .cpp
* `/V<string>` 设置版本字符串
* `/w` 禁用所有警告
* `/wd<n>` 禁用警告 n
* `/we<n>` 将警告 n 视为错误
* `/wo<n>` 发出一次警告 n
* `/w<l><n>` 为 n 设置警告等级 1-4
* `/W<n>` 设置警告等级 (默认 `n=1`)
* `/Wall` 启用所有警告
* `/Wp64` 启用 64 位端口定位警告
* `/WX` 将警告视为错误
* `/WL` 启用单行诊断
* `/Yc[file]` 创建 .PCH 文件
* `/Yd` 将调试信息放在每个 .OBJ 中
* `/Yl[sym]` 为调试库插入 .PCH 引用
* `/Yu[file]` 使用 .PCH 文件
* `/YX[file]` 自动 .PCH
* `/Y-` 禁用所有 PCH 选项
* `/Zm<n>` 最大内存分配 (默认为 %)

## 运行时库与链接

* `/MD` 与 `MSVCRT.LIB` 链接
* `/ML` 与 `LIBC.LIB` 链接
* `/MT` 与 `LIBCMT.LIB` 链接
* `/LD` 创建 .DLL
* `/LDd` 创建 .DLL 调试库
* `/MDd` 与 `MSVCRTD.LIB` 调试库链接
* `/MLd` 与 `LIBCD.LIB` 调试库链接
* `/MTd` 与 `LIBCMTD.LIB` 调试库链接
* `/F<num>` 设置堆栈大小
* `/link [链接器选项和库]`

## 编译策略讨论

楼主的文章写的不错!

但标题写的有点疑虑。

因为优化有几种意义: 速度优化, 代码空间优化, 编译速度优化和内存占用优化。

一般来说需要速度优化的地方, 代码空间优化和内存占用优化就要牺牲一些。

编译速度优化可能意义不太大, 不过我还是补充一下:

例如添加以下宏可以提高一些编译速度, 在硬盘和 CPU 速度较慢的机器上尤为明显。

```c
#define VC_EXTRALEAN
#define WIN32_LEAN_AND_MEAN
#define WIN32_EXTRA_LEAN
```

另外如果关闭预编译头文件的功能, 可以大大提高第一次编译的速度, 并减少硬盘占用的空间。

强烈建议编译小程序的时候关闭这个功能, 因为它默认是启动的。

下面评价一下楼主的一些优化, 仅供参考。

### (1) 自定义程序的入口

一般不建议使用自定义入口, 虽然只有使用动态库时只节省不到 1KB。

主要是考虑到 C++ 的全局对象的构造, 如果自定义就会被忽略掉。

不过如果确实想使代码精简到极限, 可以考虑。

此优化对速度几乎无影响。

### (2) 合并区段

这一项对减小 PE 文件大小很有帮助, 但一定要注意**正确设置好节(段)属性**, 否则后果不可知。

表面上此优化对速度无影响, 但可能会导致数据 cache 与代码 cache 重叠, 不利于 CPU 的执行。

如果需要加压缩壳, 此优化可能无效, 如果合并不当可能还会使压缩能力降低 (例如代码被混合在数据当中)。

### (3) 不用调 `malloc`、`free` 等函数

分配内存不用 `malloc`、`free` 等函数用什么? `new` 和 `delete` 吗? 后者当然更好 :)

Win32 API 有众多分配内存的函数, `malloc` 等 C/C++ 标准库当然使用的是 API 中的

堆分配 (`HeapAlloc`), 而堆分配调用虚拟分配 (`VirtualAlloc`)。效率当然是后者高一些。

但实际内存分配一般在初始化过程中使用, 而且使用次数也不应过多 (以避免内存碎片)。

所以速度影响几乎可忽略。

代码量应该不会有太大影响。

### (4) 不用 `cout` 来输出, `cin` 来输入

这个确实可以使用 C 标准库来替代, 而且使用灵活, 速度更快一些。

如果使用动态库, 代码量差距不大。

### (5) 如果调用了很多库函数如 `memset`、`strcpy` 等等的话，请导入 `msvcrt.lib`

不然编译器会在程序里面导入静态库，这样的话程序就会大很多。。

这点不用担心了。原因如下:

如果使用最快速度优化, `memset`、`strcpy`、`sin` 等某些内存/字符串/数学函数会被自动内联到程序中,

无论是否使用动态库还是静态库, 速度要快的多, 我也推荐这么做。

使用次数过多代码量会大一些。但如果加压缩壳的话影响不会太大。

如果使用最小代码优化, 自动内联的函数不会内联, 此时可使用动态库以减小代码量。

如果只使用 1~2 次某个可自动内联的函数的话代码量可能反而更多 (还要考虑到导入表)

### (6) Windows 编程不用 MFC

呵呵, 这就不一定了。目前如果没有还在使用 Win95 系统的话, MFC 的动态库就可以大胆使用了。

但要注意一定要使用 `MFC42.DLL` 的库, 也就是 VC6.0 自带的库, 高版本的可能连 WinXP 也不会自带。

### 减少程序运行内存占用量

可以调用 `SetProcessWorkingSetSize(GetCurrentProcess(),-1,-1);`

又学到一招!

但我不得不说一下: 这是个骗人的招数。拿来骗一骗初级用户还可以 -\_-

因为任务管理器只显示进程当前使用的物理内存大小, 所以使用这个招数虽然可以降低物理内存占用。

但虚拟内存并无减少, 所以这个技巧只建议使用在后台或很少工作的进程。

经常使用这个函数会造成 CPU 负担过重, 磁盘交换文件的读写也会更频繁。这样会很影响进程的执行效率。

其实不需要在意物理内存占用, 如果物理内存不够用, 系统会自动降低不常工作的进程的物理内存。

### 补充链接器选项

#### 代码对齐

```cpp
#pragma comment (linker, "/align:0x200")
```

这招我也经常用, 不过我经常这么写 (只用于 VC6.0):

```cpp
#pragma comment(linker,"/OPT:NOWIN98")
```

但是如果要加压缩壳的话, 这个 PE 文件空间优化没有一点作用,

因为虚拟地址没有改变 (仍然是 4KB, 而且无法改变)。

#### 16 字节对齐

```cpp
#pragma comment (linker, "/ALIGN:16")
```

对齐到 16 字节可能有兼容性问题, 可能不能在 Win9x 中运行。

#### 自定义入口点

```cpp
#pragma comment(linker, "/ENTRY:EntryPoint")
```

如果没有不加入默认库的话可以不用自定义入口。

而直接用 `void mainCRTStartup()` 或 `void WinMainCRTStartup()` 即可。

但不使用默认库可能某些浮点数的转换会出现问题, 虽然可以使用其他方法解决。

#### 引用优化与合并

```cpp
#pragma comment (linker, "/OPT:REF")
#pragma comment (linker, "/OPT:ICF")
```

这一项其实在默认的优化编译设置中已经有了, 不用写出来。

如果启动 `"/OPT:REF"` 的话, `"/OPT:ICF"` 默认会自动启动。

#### 合并资源段

```cpp
#pragma comment (linker, "/merge:.rsrc=.text")
```

这项一定不要加, 有时候如果资源不在某个节 (段) 的开始, 程序的图标可能不能显示出来。

### 关于内存优化函数的误解

**SetProcessWorkingSetSize**

这个我是跟踪 `realplay` 里面一个程序 `realsched.exe` 发现的，其它减少内存用量的方法，还没找到![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg)

对于那些大一点的程序，就不能用 `settimer` 每隔段时间调用，

........

nono, 不是大的程序自己搞的，是操作系统做的。

在你的窗口最小化的时候，操作系统自动帮你把内存最小化了。

所以，你自己就更没有必要去调这个函数了。

### 关于 MFC 动态库的讨论

**nono, mfc42.dll 从 win98se 开始，到 longhorn 都是默认有的。vc6 mfc 的程序，编成 release 就可以在每台机器上跑了，不用到处拷个 mfc42.dll**

可能是对我的话理解有误。我们的观点是一样的。

再补充一下:

Win95OSR2 (Win97) 虽然自带 `msvcrt40.dll` 和 `mfc30.dll`, 但不是目前的标准库了。

Win98SE 自带了 `msvcrt.dll` 和 `mfc42.dll`, Win98 第一版不知有没有。

WinXP 第一版肯定不带 `msvcr71.dll` 和 `mfc71.dll`, 这些是 VC7.1 才使用的动态库。

4dd466d0e1e0cc123afbe504da2462ca
