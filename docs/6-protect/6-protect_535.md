

# C/C++壳与汇编壳的不同以及 DLL 壳相对于 EXE 壳的难点， 申精

> 原文：[`www.pediy.com/kssd/pediy11/119570.html`](https://www.pediy.com/kssd/pediy11/119570.html)

标 题: 【原创】C/C++壳与汇编壳的不同以及 DLL 壳相对于 EXE 壳的难点  
作 者: soliddream  
时 间: 2010-8-31,12:40:47  
链 接: http://bbs.pediy.com/showthread.php?t=119570  
  
这个月主要是学习 PE 文件以及壳。革命尚未成功，先记下遇到的难点吧。C/C++壳与汇编壳的不同与 DLL 壳相对于 EXE 壳的难点看起来是不同的话题，但两者有一定的关联性，暂且放在一起吧。而且后面一个难题我还在攻克中，不能提供答案，只能让人思索。  
  
C/C++壳与汇编壳的不同  
很奇怪的一件事，我发现大多数壳都是汇编写的，C/C++壳很少。当然不可否认的是，壳主要是指针的操作（加壳程序还得算上文件 IO），汇编相对于 C/C++操纵指针起来更加得心应手。尤其是汇编可以直接操纵堆栈，由此有了两大优势：一、方便数据寻址，二、方便堆栈平衡。  
  
再加上操纵 PE 文件有许多现成的汇编代码可供参考(罗云彬写的 Windows 环境下 32 位汇编程序设计有专门的 PE 文件一章),调试壳也只能用汇编(除非有人能成功使得 C/C++壳的 pdb 调试符文件可用，这想来应该是件浩大的工程)。  
  
这样算来，无怪乎大多数壳是用汇编写的了。不过能熟练用汇编写程序的人毕竟是少数，C/C++相比于其它高级语言，算得上是最接近汇编的了。对我而言，在 C/C++壳基础上可以方便的重构，虽然有学好汇编的想法，但是真正学好汇编不是短期内能完成的事情。  
  
(以上是个人看法，实际情况不得而知)  
  
DLL 壳相对于 EXE 壳的难点  
由于 C/C++壳与汇编壳的不同，C/C++程序员写起壳来比汇编程序员遇到的困难更多一些，尤其是在支持 DLL 加壳上。  
  
我现在试图使一个现成的 C/C++壳支持 DLL，遇到困难如下：  
  
一、重定位  
  
这个问题的描述就引用 Writing your packer 一文中的话吧  
  
Since the stub is a parasite, and since it will have to be  
  
located in a spot at the original application's convenience,  
  
we will have to be relocating it dynamically in the packer  
  
application. To help with this we will make the stub with  
  
relocation records. These are usually used for dlls when  
  
they can't be loaded at their preferred address.  
  
......  
  
If you're an avid ASM coder, many things are more  
  
straightforward since you can take care to produce  
  
positionindependent  
  
code.  
  
不解释，你懂的。  
  
二、堆栈平衡  
  
引用《加密与解密》一书的话描述这个问题  
  
编写加壳软件时，必须保证外壳初始化的现场环境(各寄存器值)与原程序的现场环境是相同的(主要是 ESP、EBP 等重要的寄存器值)。加壳程序初始化时保存各寄存器的值，外壳执行完毕，再恢复各寄存器内容，最后再跳到原程序执行。  
  
......  
  
可以把整个外壳当作一个函数或子程序来理解，这个过程遵守堆栈平衡的原理。  
  
我现在的问题就是执行到原程序的 OEP，堆栈平衡被破坏了。大眼瞪小眼，有些不知所措的感觉。  
  
三、DLL 相对于 EXE 的特点，DLL 的入口点在整个过程中至少执行两次。一次是开始时，用来对 DLL 做一些初始化。至少还有一次是在退出时，用来清理 DLL 再退出。  
  
因此对于壳而言，退出时再次进入入口点，外壳跳过相关的初始化代码。  
  
由于没有完成这方面工作，我测试加壳 DLL 退出时在解压缩部分报错，提示访问非法地址。  
  
目前遇到以上三个难题，第一个已经解决，但是不清楚是否有副作用。也许在真正解决完这个问题的路上会碰到更多的难题也不一定。 

终于搞定第二个和第三个难题，dll 可以正常的 Loadlibrary 和 freelibrary 了，内牛满面！  
  
先说第二个难题堆栈平衡  
注意要点就是壳跳到原程序之后应该永远也不会返回到壳，所以需要将壳 pop 掉。  
调试的时候一个技巧就是观察运行到原程序入口点的参数，假设载入点是 10000000  
那么  
arg1 10000000  
arg2 00000001  
arg3 00000000  
如果不是的话，观察一下堆栈的偏移。  
  
再说说第三个难题，用 write your packer 文章的话来说很简单的事  
Easy enough, add a global boolean  
that indicates that stuff was done and set it to true after  
the first pass.  
另外一个注意事项是注意 push oep 的时候，注意 oep 的值有没有被改变  
  
看在我这么幸苦的份上，给我个精华吧。