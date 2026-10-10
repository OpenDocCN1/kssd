

# 查表法反汇编引擎源码

> 原文：[`www.pediy.com/kssd/pediy10/89455.html`](https://www.pediy.com/kssd/pediy10/89455.html)

感谢：  
　　egogg 的文章[`bbs.pediy.com/showthread.php?t=75094`](http://bbs.pediy.com/showthread.php?t=75094)  
　　luocong 的文章  
　　Intel 64 and IA-32 Architectures Software Developer's Manuals  
  
　　套用一句话：捧着一本厚厚的 Intel 指令守则，不写出个反汇编引擎都觉得对不起他。(忘了是不是这么说的)  
  
　　原理不再介绍，上面的文章已经很详细了。读懂原理，剩下就是纯体力活了。  
  
源码：  
　　VirBin.rar(更新于 2009-05-31)  
  
环境：  
　　gcc (推荐 Code::Blocks，参考：[`www.d2school.com/codeblocks/d...cks\_setup.html`](http://www.d2school.com/codeblocks/doc/codeblocks_setup.html))  
　　nasm([`www.nasm.us/`](http://www.nasm.us/))  
  
编译：  
　　1.在系统环境中加入 nasm 的路径  
　　2.用 Code::Blocks 打开 default.workspace 文件，在【视图】中打开【管理器】。  
　　3.右键点击【test】工程，选择【构件】  
　　4.右键点击【VirBin】工程，选择【构件】  
  
备注：  
　　因特殊需要部分指令没有解析（见【不支持指令.txt】）。