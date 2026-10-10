

# 幻影 2.0b3

> 原文：[`www.pediy.com/kssd/pediy05/pediy50149.htm`](https://www.pediy.com/kssd/pediy05/pediy50149.htm)

幻影 2。0b3 的脱壳   
  
该版有了很大变化，代码得到优化，运行速度很快，是一个相当不错的加壳   
软件。   
  
1。anti-debug   
  
主要有：1）通过改变 int03 的中断门，解码   
        2）程序启动时用 pushf 设置一个暗桩解码。单步通过这 pushf，后面的解码就会错误。   
        3）通过 int0 和 int41 的中断门，xor,判断是否有调试软件   
        4）createfile 判断常用的调试和侦察软件   
        5）在 ring0，清除 drx 寄存器   
        6）运用 in、out 指令，判断调试跟踪(这里的原理我还没有弄明白）   
注册码的计算及比较在壳中。   
  
2。脱壳   
  
用 trw2000 装载运行 dbpe.exe，躲过上述 anti-debug,在 7b703d cmp \[esi\],00 处停下，从这里开始   
解码被加壳程序，当解码完毕，用 pedump dump.exe 出文件。这时 dump.exe 中包含了正确的输入表，   
输入表的 RVA 是 ca000.不过在 IMAGE\_IMPORT\_DESCRIPTOR 中，OriginalFirstThunk、Name、FirstThunk 三项   
已加密，它的加密很简单，先 NOT,再分别与 02020202、01010101、03030303 XOR 后，就是正确的 RVA 了。手工   
将这些加密的还原回来。   
  
它的 OEP 是 401000（RVA 1000)，Import Tatle 的 RVA 是 ca000,用 pe 编辑器将 dump.exe 的这两个值改回来。   
  
这样修改 dump.exe 后，还不行。原壳从 7B7620 处开始，将对 4b7b0c 后的代码进行处理。跟踪后发现，它将   
IAT 中的函数指针作了处理。在这里，适当将原代码作一下处理，我是将 mov ebx,\[ebx\] nop 掉，再将 mov \[edi\],esi 改   
为 mov \[edi\],ebx.这样改后，等这段代码处理完毕（计数寄存器 ecx 为零）后，将 4b7b0c 后的 a20 字节代码写入一个文件   
1.bin,退出原程序 dbpe.exe。将 1.bin 中的数据写进 dump.exe 的 offset b7b0c 开始的 a20 个字节中。   
  
dump.exe 经过这样处理，已成功脱壳并修复输入表。   
  
幻影新版真的不错。。。。建议去使用和注册。  4dd466d0e1e0cc123afbe504da2462ca