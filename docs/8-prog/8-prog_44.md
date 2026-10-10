

# Patch msvcrt.dll/ntdll.dll 提升系统浮点性能

> 原文：[`www.pediy.com/kssd/pediy10/98958.html`](https://www.pediy.com/kssd/pediy10/98958.html)

最近认真看了一下 Intel 的算术库,对其中的 SSE2 的运用印象深刻.  
然后猛然想到 windows 2003 在使用 FPU87 呢,还是 SSE2 呢?  
经过调试,很遗憾,系统使用的是传统的 FPU87 的指令集,这实在是浪费 CPU 的能量啊.  
于是操起 OllyDbg,打开 msvcrt.dll,ntdll.dll 改写了一些代码.  
1.\_ftol: VC7.1 之前编译的函数按默认设置都会调用此函数进行浮点转整形,对于脚本引擎等涉及频繁浮点整形转化的程序很有意义  
改写如下:  
 cvttsd2si eax, \[esp+arg\_0\]  
 retn  
2.fabs: VC 未开启内联,或低于 VC7.1 时被调用,用于 abs~  
改写如下:  
fld     qword ptr \[esp+4\]  
fabs  
retn  
3.  
其他浮点函数 atan/ceil/exp/floor/log/log10/modf/pow  
这些在 msvcrt.dll 是有开关函数的 \_set\_SSE2\_enable,虽然 MSDN 说如果 CPU 支持会自动开启,但是其实是如果 CPU 支持将允许你启用,不等于自动开启  
代码片段:  
  77BE1499      8325 845DBF77 00         and     dword ptr \[77BF5D84\], 0  
  77BE14A0      8325 805DBF77 00         and     dword ptr \[77BF5D80\], 0  // 直接都改成 or 1  
  77BE14A7  |.  F645 EB 04               test    byte ptr \[ebp-15\], 4  
  77BE14AB  |.  74 13                    je      short msvcrt.77BE14C0  
  77BE14AD  |.  E8 4EFFFFFF              call    msvcrt.77BE1400  //进行无效指令计算  
  77BE14B2  |.  85C0                     test    eax, eax  
  77BE14B4  |.  74 0A                    je      short msvcrt.77BE14C0  
  77BE14B6  |.  C705 845DBF77 01000000   mov     dword ptr \[77BF5D84\], 1 // 如果支持 77BF5D84 设为 1,但是却没有把 77BF5D80 设为 1  
  77BE14C0  |>  8B4D FC                  mov     ecx, dword ptr \[ebp-4\]  
  
OK,这就好办了  
or    dword ptr \[77BF5D80\], 1  
这样就强制开启了.  
对于 ntdll.dll,大部分函数没有 SSE2 的代码,只有 ceil,floor,直接去掉头部的 jmp 即可.  
   
额外的:  
vcpkg5.5 自带了这个 patch,当 VC6IDE 加载 vcpkg 时自动 patch msvcrt.dll,提升 VC6 的浮点性能.显然,内存 patch 仅作用于当前进程,对系统其他进程没有帮助,推荐 patch 系统文件,这样所有系统中运行的软件都会受益.  
  
对于开发人员:  
如果你在用 VC6 或可能会调用 msvcrt.dll 中的浮点运算函数(就算你不调用,系统自身 dll 也会调用,因为它是 NT CRT DLL),那么显然我们不能去 Patch 用户计算机中的文件,启动时执行以下代码,有益于提高速度.  
调用永远是安全的,因为这个函数会将设置与屏蔽位取与,所以,只有在 SSE2 指令集的计算机中在最终生效.  
  typedef int (\_cdecl \*PFN\_set\_SSE2\_enable)(int flag);  
  PFN\_set\_SSE2\_enable pfn\_set\_SSE2\_enable;  
  pfn\_set\_SSE2\_enable=(PFN\_set\_SSE2\_enable)GetProcAddress(GetModuleHandle(\_T("msvcrt.dll")),"\_set\_SSE2\_enable");  
  if (pfn\_set\_SSE2\_enable) pfn\_set\_SSE2\_enable(1);  
   
注意:  
1.如果你要跟随这个修改,务必保证你的 CPU 支持 SSE2 指令集,否则,.....  
2.关掉系统的 SFC,否则,会被自动还原  
3.ntdll.dll,这类系统 dll 的修改,记得替换前更新 CheckSum,这是好习惯