

# PE 装载器如何装载节区猜想

> 原文：[`www.pediy.com/kssd/pediy12/132195.html`](https://www.pediy.com/kssd/pediy12/132195.html)

感谢各位看帖，回帖的朋友。在看雪各位朋友的帮助下，我想找到了我想要的答案，在此分享下^\_^（如有谬误，欢迎指正）  
  
求助贴参见：http://bbs.pediy.com/showthread.php?t=132101  
  
首先根据两个事实：  
1.一些 PE 文件节区的 VirtualSize 比 SizeOfRawData 大，一些 PE 文件节区的 VirtualSize 比 SizeOfRawData 小。  
2.一些 PE 文件节区的 VirtualSize 并不是 SectionAlignment 的整数倍，很多 VirtualSize 甚至为奇数。  
3.我将 PE 文件的 CODE 节区的 SizeOfRawData 增加了一个字节（变为奇数），不影响 PE 文件执行。  
  
得出推论：  
1.VirtualSize 不是数据装入内存后按内存对齐粒度对齐后占用空间大小，也不一定要按照 SectionAlignment 对齐（依据事实 2）  
2.VirtualSize 也不是节区内数据的实际大小（依据事实 1）  
3.SizeOfRawData 也并不一定要按照 FileAlignment 对齐。  
  
根据 QEver 朋友推荐 kenmark 大牛的帖子：PE 文件内名存实亡的对齐（地址：http://bbs.pediy.com/showthread.php?t=46849）结合上述推论得出以下猜测：  
  
PE 文件中的对齐“名存实亡”。  
1.kenmark 根据实验得出：SizeOfRawData 不一定要按照 FileAlignment 对齐，但是 FileAlignment 必须是 2 的整数次幂。  
2.根据上述推论，VirtualSize 也不一定要按照 SectionAlignment 对齐。  
  
据此，得出个人观点：  
VirtualSize 和 SizeOfRawData 分别应该按照 SectionAlignment 和 FileAlignment 对齐仅仅是 PE 装在器的推荐做法，当 PE 文件并没有这样对齐时 PE 装载器仍然会“智能”的进行装载操作，以便运行程序。  
  
那么，PE 装载器在将节区载入内存时是如何操作的呢？以下仅为我根据以上推论又进行了一些实验（实验环境：Windows 7,工具：C32ASM+PEID）：  
1.我将 PE 文件的 CODE 节区表中的 SizeOfRawData 的字段值逐渐增加，直到增加到 0x0FFFFF，PE 文件仍然可以正常执行，且内存占用无变化。  
2.我将 PE 文件的 CODE 节区表中的 SizeOfRawData 的字段组建减少，当减少到某个字节后就无法执行。  
3.我将 PE 文件的 CODE 节区表中的 VirtualSize 进行大小调整发行规律：只要 VirtualSize 按 SectionAlignment 对齐后+RVA=下一个节区 RVA，就能正常执行否则无法执行。  
  
根据实验和上述一些推论做如下猜测：  
PE 装载器首先申请一片内存空间，内存大小为节区 VirtualSize 按 SectionAlignment 对齐后的大小，然后判断 SizeOfRawData 是否大于已经申请的内存空间，如果 SizeOfRawData 大于申请的内存空间，则从 PE 文件节区的 PointToRawData 开始读取数据，当申请的内存空间存满即止。如果 SizeOfRawData 小于申请的内存空间则从 PE 文件节区的 PointToRawData 开始读取长度为 SizeOfRawData 的数据填入申请的内存空间，并将剩余空间用 0 填充。  
  
以上观点如有谬误欢迎指正。  
  
再次感谢 QEver，zapline 等朋友的帮助……