

# POINT H 收集翻译

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-712.htm`](https://www.pediy.com/kssd/pediy07/pediy7-712.htm)

POINT H，2K/XP 下所谓的万能断点。从网上收集的点滴，翻译而成。以后遇到相关材料，还将继续补充。  
  
1、什么是 POINT H？  
　　它是在 XP/2K 机子上的老的已知的 hmemcpy，搜索 user32 中：F3 A5 8B C8 83 E1 03 F3 A4 E8，就是 POINT H。  
　 或者搜索 8B C1 C1 E9 02 F3 A5 8B C8 83 E1 03 F3 A4 E8。一旦找到，它返回位置地址，也就是说在内存拷贝之前。（实际的操作：F9，在内存中搜索上述字符串）  
  
   该模式应该发生在 USER32 的第一个 OX3FFFF。比如：  
  
77D46706 mov eax, ecx   
77D46708 shr ecx, 2   
77D4670B repe movsd ; <-- 在该指令上中断   
77D4670D mov ecx, eax   
77D4670F and ecx, 3   
77D46712 repe movsb   
77D46714 call sub\_77D46722  
（呵呵，我在我的机子上也找到了，还是借用别人的吧，省得麻烦：）  
　  
　　该段代码显示了内存是在哪里从 gui-项被拷贝到提供给用户的缓冲器中。追踪一小段，出现 SendMessage 和 Getwindow×××代码段。  
  
2、在 OD 或 SICE 中输入：  
　 BP “POINT-H 的值”或在 SICE 中用 BPX。它类似 Win98 中的 hmemcpy，但工作在 2K/XP 的 80%的程序中（不用于 VC++)。但我们可以用一些技巧使它用于所有的程序(也可用于 VC++)。当在程序编辑框中输入假的序列号时，程序中断。  
  
3、实际上，输入 bpx user32!EcGetText+51 可以做到"d edi-ebx"。  
  
4、还有寻找 POINT H 的另外一个方法：如果有 USER32.DLL 的符号文件（USER32.PDB 或 USER32.DBG)，可以在 IDA 中打开 USER32.DLL，适用 PDB 插件加载上面的符号，然后可以搜索函数 ECGetText。它就是 POINT H。该函数是 USER32.DLL 的内部函数，调用来自 EditWndProc（也是一个内部函数）。  
\==============================================================  
译者注：我进行了该方法测试，的确找到了该位置。它实际上是对键盘输入和鼠标按键的一个中断，但是和万能断点插件中 POINT H 的地方不一致。万能断点插件中的 POINT H 好像常常不能中断，只有 VB5 和 VB6 的能中断。请教大虾们指导一下。 

> 引用:
> 
> * * *
> 
> *最初由 laoqian 发布*  
> **搜索 user32 中：F3 A5 8B C8 83 E1 03 F3 A4 E8，什么意思？在哪里搜索？  
> 我在 USER32.dll 里搜索只有 F3 A5 8B C8 83 E1 03 F3 A4 eb，和 F3 A5 8B C8 83 E1 03 F3 A4 E9，没有 E8，怎么回事？ **
> 
> * * *

  
  
搜索方法：  
　　菜单：查看－>执行模块->双击 USER32.DLL，在 USER32 模块窗口中：右键－>搜索－>二进制字符，在 HEX 里输入 F3 A5 8B C8 83 E1 03 F3 A4 E8，就可以找到 Point-H 的位置了。我这里完全可以啊。XP+SP2 

 4dd466d0e1e0cc123afbe504da2462ca