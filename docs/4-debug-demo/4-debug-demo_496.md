

# 恺撒大帝 3

> 原文：[`www.pediy.com/kssd/pediy05/pediy50147.htm`](https://www.pediy.com/kssd/pediy05/pediy50147.htm)

顺祝忘了自己喜得贵子！:)   
hdy 2002.05.27   
  
  
恺撒大帝 3 中文光盘版去除 CD 检测   
  
hdy 2002.05.24   
  
此老游戏今天被翻出来,想再玩一下,发现我的老光驱在读它的时候声音奇大,于是,就做个硬盘版吧. :)   
  
使用工具:FileInfo2.49,W32Dasm Gold8.93 中文版,UltraEdit32v9.00a   
  
破解程度:非常简单.   
  
1.分析   
安装恺撒大帝 3 中文光盘版,选择完全安装,然后从光驱中取出光盘,执行游戏程序 c3.exe(811,008 字节),程序在弹出一个 MessageBox 后,要你选择确定或取消,取消则退出.此处显示为乱码.   
先用 FI 查看一下,发现是 VC++6.0 编译的,没有加壳.于是用 W32Dasm 将其反汇编,查找字符串 MessageBoxA,得到下面两处,   
  
\* Reference To: USER32.MessageBoxA, Ord:01BEh   
|   
:0040C824 FF1564414A00 Call dword ptr \[004A4164\]   
:0040C82A 83F806 cmp eax, 00000006   
:0040C82D C605FAC7500000 mov byte ptr \[0050C7FA\], 00   
:0040C834 750C jne 0040C842   
:0040C836 881D706A6B00 mov byte ptr \[006B6A70\], bl   
:0040C83C 881DCCCA5000 mov byte ptr \[0050CACC\], bl   
  
\----------------------------------------------------------   
  
\* Possible StringData Ref from Data Obj ->"smk\\intro.smk"   
|   
:0049CCE8 68C8B04B00 push 004BB0C8   
:0049CCED E8BE75FCFF call 004642B0   
:0049CCF2 83C438 add esp, 00000038   
:0049CCF5 85C0 test eax, eax   
:0049CCF7 755E jne 0049CD57   
  
\* Reference To: USER32.MessageBoxA, Ord:01BEh   
|   
:0049CCF9 8B3564414A00 mov esi, dword ptr \[004A4164\]   
  
用 W32Dasm 的调试->加载处理,在这两处设下断点,F9 运行...结果中断后屏幕被游戏画面挡住了 :( 用任务管理器将游戏强行关闭,发现调试窗停在:0049CCF9 处的断点处.向上找发现了条件跳转指令 jne 0049CD57,只要将它改为 jmp 0049CD57 无条件跳转就 OK 了.   
  
2.修改   
使用 UltraEdit,根据 W32Dasm 给出的文件偏移量 0x9ccf7,将 jne 的机器码 75 改为 jmp 的机器码 EB,存盘测试,发现已经通过 CD 检测.  4dd466d0e1e0cc123afbe504da2462ca