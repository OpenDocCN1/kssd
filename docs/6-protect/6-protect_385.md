

# 调试反调试学习笔记

> 原文：[`www.pediy.com/kssd/pediy12/144819.html`](https://www.pediy.com/kssd/pediy12/144819.html)

调试反调试  
    
  科锐学习有快一年了， 最近 特迷软件的反反调试技术. ..发篇水帖看看能否找到有共同兴趣的朋友。。  
    
  反调试的方法太多了，先从最简单的开始...  
    
  1> IsDebuggerPresent    
  微软提供的 API,随时在自己的程序中调用 IsDebuggerPresent 都可以检测出自己的程序是否被调试...  
  先在 vc++ 6.0 创建一个控制台测试程序 代码如下：  
    
  #include <windows.h>  
    
  typedef BOOL ( \_stdcall \*LPAPI\_IDP)(VOID);  
    
  int main(int argc, char\* argv\[\])  
  {  
      HMODULE hModule = LoadLibrary("Kernel32");  // 加载模块 Kernel32  
      if (hModule == NULL)  
      {  
          ExitProcess(0);  // 如果发现程序被调试 直接退出进程  
      }  
        
      LPAPI\_IDP IsDebuggerPresent = GetProcAddress(hModule, "IsDebuggerPresent");  // 获取下地址  
      if (IsDebuggerPresent == NULL)  
      {  
          ExitProcess(0);  // 如果发现程序被调试 直接退出进程  
      }  
        
      if (\*(BYTE \*)IsDebuggerPresent == 0xcc || // 调用前检测下是否被下了断点  
          \*(BYTE \*)IsDebuggerPresent != 0x64 ||  
          IsDebuggerPresent())  // 调用  
      {  
          ExitProcess(0);  // 如果发现程序被调试 直接退出进程  
      }  
        
      // 如果程序能执行到这里 说明程序没有被调试状态  
      MessageBox(NULL, "Antidebug", NULL, MB\_OK);  
        
      return 0;  
  }  
    
  直接运行。。。  
    
  按 F5 调试运行 就会发现 程序直接推出..  
    
  单步跟踪可以测试到调用 IsDebuggerPresent 以后程序退出了..  
    
    
    
  
  现在把编译好的可执行文件 用 ollydbg（安装了带反调试插件的 od 会免疫， 学习的话最好把插件先清除）测试... 一不留神程序即退出了 ...  
    
  看如何解决掉这个小问题.. （这里因为代码是自己写的 ，很容易就能定位到关键点，实际中各种手段五花八门 ... ）  
    
  重新跑起 od 断到入口 找到 main 函数...跟进去就是我们的代码了。。。  
  
  // main()  
  00401000  /$  56            push    esi  
  00401001  |.  57            push    edi  
  00401002  |.  68 50604000   push    00406050                         ; /FileName = "Kernel32"  
  00401007  |.  FF15 08504000 call    dword ptr \[<&KERNEL32.LoadLibrar>; \\LoadLibraryA  
  0040100D  |.  8B3D 04504000 mov     edi, dword ptr \[<&KERNEL32.ExitP>;  kernel32.ExitProcess  
  00401013  |.  8BF0          mov     esi, eax  
  00401015  |.  85F6          test    esi, esi  
  00401017  |.  75 04         jnz     short 0040101D  
  00401019  |.  6A 00         push    0                                ; /ExitCode = 0  
  0040101B  |.  FFD7          call    edi                              ; \\ExitProcess  
  0040101D  |>  68 3C604000   push    0040603C                         ; /ProcNameOrOrdinal = "IsDebuggerPresent"  
  00401022  |.  56            push    esi                              ; |hModule  
  00401023  |.  FF15 00504000 call    dword ptr \[<&KERNEL32.GetProcAdd>; \\GetProcAddress  
  00401029  |.  8BF0          mov     esi, eax  
  0040102B  |.  85F6          test    esi, esi  
  0040102D  |.  75 03         jnz     short 00401032  
  0040102F  |.  50            push    eax  
  00401030  |.  FFD7          call    edi  
  00401032  |>  8A06          mov     al, byte ptr \[esi\]  
  00401034  |.  3C CC         cmp     al, 0CC  
  00401036  |.  74 0A         je      short 00401042  
  00401038  |.  3C 64         cmp     al, 64  
  0040103A  |.  75 06         jnz     short 00401042  
  0040103C  |.  FFD6          call    esi              // 调用 IsDebuggerPresent  
  0040103E  |.  85C0          test    eax, eax  
  00401040  |.  74 04         je      short 00401046  
  00401042  |>  6A 00         push    0  
  00401044  |.  FFD7          call    edi  
  00401046  |>  6A 00         push    0                                ; /Style = MB\_OK|MB\_APPLMODAL  
  00401048  |.  6A 00         push    0                                ; |Title = NULL  
  0040104A  |.  68 30604000   push    00406030                         ; |Text = "Antidebug"  
  0040104F  |.  6A 00         push    0                                ; |hOwner = NULL  
  00401051  |.  FF15 9C504000 call    dword ptr \[<&USER32.MessageBoxA>>; \\MessageBoxA  
  00401057  |.  5F            pop     edi  
  00401058  |.  33C0          xor     eax, eax  
  0040105A  |.  5E            pop     esi  
  0040105B  \\.  C3            retn  
    
  到了这里简单的方法估计就是把   0040103C  |.  FFD6          call    esi 修改  
  0040103C      33C0          xor     eax, eax  
  不给程序调用 IsDebuggerPresent 的机会.. 可是不能每次调试都来修改吧..而且有的应用中根本就是不知道什么时候来检查一下 ...  
    
  有效的方案则是在 API 中做点小手脚，让他返回错误的结果就可以.... 现在跟进到 IsDebuggerPresent 中 看看 IsDebuggerPresent 的实现  
    
  7C813133 kernel32.IsDebuggerPresent      /$  64:A1 1800000>mov     eax, dword ptr fs:\[18\]  
  7C813139                                 |.  8B40 30       mov     eax, dword ptr \[eax+30\]  
  7C81313C                                     0FB640 02     movzx   eax, byte ptr \[eax+2\]  
  7C813140                                     C3            retn  
  意外吧， 只有简单的几条指令 随便修改下 使函数返回值为 0 即可通过...   
    
  剩下的就可以写一个小工具或者是 ollydbg 的插件来帮我们每次修改代码...因为这个程序已经可以被调试，工具的意义就不大了， 写一个简单的插件.， 过程都差不多...  
    
  先说下原理：  
  定位到进程 中 kernel32.IsDebuggerPresent 函数的地址   
  把要修改的字节写入到内存...  
  
  插件开发 大家可以翻翻老帖子介绍的很详细   
    
  这里丢个代码...  
    
    
  /\*  
  7C813133 >/$  64:A1 1800000>mov     eax, dword ptr fs:\[18\]  
  7C813139  |.  8B40 30       mov     eax, dword ptr \[eax+30\]  
  7C81313C      0FB640 01     movzx   eax, byte ptr \[eax+1\]       // eax, byte ptr \[eax+2\]   
  7C813140  \\.  C3            retn  
  \*/  
  typedef BOOL (\_stdcall \*LP\_IDP)(VOID);  
    
  void Hook\_IsDebuggerPresent()  
  {  
      int hIn = Plugingetvalue(VAL\_HPROCESS);  
      if (hIn != NULL)  
      {  
          HMODULE hModule = GetModuleHandle("Kernel32");  
          if (hModule == NULL)  
          {  
              MessageBox(NULL, "Error GetModuleHandle( Kernel32 )", NULL, MB\_OK);  
              return;  
          }  
            
          LP\_IDP lpAddr = GetProcAddress(hModule, "IsDebuggerPresent");  
          if (hModule == NULL)  
          {  
              MessageBox(NULL, "Error GetProcAddress( IsDebuggerPresent )", NULL, MB\_OK);  
              return;  
          }  
            
          BYTE c = 0x01;  
            
          BYTE \*p = (BYTE \*)lpAddr;  
            
          Writememory(&c, (DWORD)(p + 0x0c), sizeof(c), MM\_RESTORE);  
      }  
  }  
    
  后记：  
  对于调用 api 的程序插件可以通用， 如果把 IsDebuggerPresent      的实现加到自己的程序中插件就没用了 .. 对于调试者只有重新分析了....  
    
  // IsDebuggerPresent 实现代码  
  7C813133 kernel32.IsDebuggerPresent      /$  64:A1 1800000>mov     eax, dword ptr fs:\[18\]  
  7C813139                                 |.  8B40 30       mov     eax, dword ptr \[eax+30\]  
  7C81313C                                     0FB640 02     movzx   eax, byte ptr \[eax+2\]  
  7C813140                                     C3            retn