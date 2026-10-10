

# Roadrush 跨平台运行

> 原文：[`www.pediy.com/kssd/pediy05/pediy50385.htm`](https://www.pediy.com/kssd/pediy05/pediy50385.htm)

Roadrush 是一款古老的摩托车赛车游戏。当我把 os 由 win95 升级到 win2k 简体中文版的时候，发现这个游戏不能运行了。启动时显示：   
  
0x77f88fc8 引用 0x00000000 内存，该内存不能为 read   
  
看来是游戏不能在 win2k 上运行，这也不是什么奇怪的事，就没有深究下去。   
  
前几天刚好在 ollydb 的官方网站上读了 ollydb 的 trace 教程，突发奇想，准备拿 roadrush 做个实验。   
  
用 ollydb 装载 roadrush，选择 trace into，非常走运，没有定义任何代码块，仅仅耐心的等待 1 分钟(PIII 500),程序就来到了出错的地方,察看 trace 的结果，发现直接的原因是因为 0042F43F 处的 call EBP,重新启动 roadrush，在 0042F43F 处设断点，看到如下的代码   
  
  
0042F410  /$ 83EC 08        SUB ESP,8   
0042F413  |. 8D4424 00      LEA EAX,DWORD PTR SS:\[ESP\]   
0042F417  |. 53            PUSH EBX   
0042F418  |. 55            PUSH EBP   
0042F419  |. 6A 07          PUSH 7                                  ; /BufSize = 7   
0042F41B  |. 50            PUSH EAX                                ; |Buffer   
0042F41C  |. 6A 03          PUSH 3                                  ; |InfoType = 3   
0042F41E  |. 51            PUSH ECX                                ; |LocaleId   
0042F41F  |. FF15 60A64C00  CALL DWORD PTR DS:\[<&KERNEL32.GetLocaleI>; \\GetLocaleInfoA   
0042F425  |. 8D5424 08      LEA EDX,DWORD PTR SS:\[ESP+8\]   
0042F429  |. 68 D4C34600    PUSH Roadrash.0046C3D4                  ;  ASCII ".dll"   
0042F42E  |. B9 CCC34600    MOV ECX,Roadrash.0046C3CC                ;  ASCII "Text/"   
0042F433  |. E8 483F0100    CALL Roadrash.00443380   
0042F438  |. 50            PUSH EAX                                ; /FileName   
0042F439  |. 8B2D 64A64C00  MOV EBP,DWORD PTR DS:\[<&KERNEL32.LoadLib>; |shim.78144662   
0042F43F  |. FFD5          CALL EBP                                ; \\LoadLibraryA   
0042F441  |. 5D            POP EBP   
0042F442  |. 5B            POP EBX   
0042F443  |. 83C4 08        ADD ESP,8   
0042F446  \\. C3            RETN   
  
跟踪发现，0042f438 处的 eax 为 0，用空指针调用 loadlibrarya，难怪会出错！   
  
熟悉 windows 编程的程序员其实很容易猜出这段代码的用意，调用 GetLocaleInfoA，取得 locale 信息，根据默认的语言，装载相应的资源 dll,在我的系统中，0042F425 处的代码之行后，edx 指向的字符串为"CHS",即简体中文，但./text/目录下，不存在 chs.dll 这个动态连接库，所以   
0042F433 处的 call 返回值为 0，但不知道哪个粗心的程序员不检查返回值，就调用 loadlibrary,导致上面所说的错误。   
  
修改的方法就容易了，可以有很多种改法，我用了最懒惰的一种，把 enu.dll 拷贝成 chs.dll。  4dd466d0e1e0cc123afbe504da2462ca