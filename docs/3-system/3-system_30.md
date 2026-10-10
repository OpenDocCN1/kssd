

# PE 文件内名存实亡的对齐

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-189.htm`](https://www.pediy.com/kssd/pediy09/pediy09-189.htm)

> PE 文件内名存实亡的对齐  
>   
> 大家知道 PE 文件中 IMAGE\_NT\_HEADERS 的 IMAGE\_OPTIONAL\_HEADER32 里有两个对其因子 SectionAlignment 和 FileAlignment 分别表示内存中块的对齐和文件中块的对齐。我同时翻阅了 Visual Studio, Microsoft Portable Executable and Common Object File Format Specification Revision 8.0 - May 16, 2006 英文原版以及各种中文版的 PE 文件格式解析（其中包括《加密与解密》），发现描述 Section 时特别指出块的 PointerToRawData 和 SizeOfRawData 必须遵循 FileAlignment（即必须是对齐因子的整数倍），而 VirtualAddress 必须遵循 SectionAlignment，同时两个对齐因子必须都是 2 的整数次幂，且 FileAlignment 必须小于等于 SectionAlignment。  
> 对于此点，各色文档参差不齐，例如英文文档中没有强调 VirtualAddress 的对齐，而《加密与解密》中没有强调 PointerToRawData 的对齐。众说风云，姑且先放一下，我用 LordPE+Win2k(SP4)作了一系列的试验来验证对齐问题。  
>   
> 首先必须研究一下 section（有人译成节，本人将其译成块，在下文中不再解释）。  
> 可执行文件中，块有 4 个主要参数和 2 个参考参数，分别为：  
> 4 个主要参数：  
> PointerToRawData(在 LordPE 中为 ROffset)表示块在文件中开始的偏移  
> SizeOfRawData(在 LordPE 中为 RSize)表示块在文件中对齐后的大小  
> VirtualAddress(在 LordPE 中为 VOffset)表示块在内存中的 RVA（相对虚拟地址）  
> VirtualSize(在 LordPE 中为 VSize)表示块从文件映射到内存时实际需要映射的大小（也就是对齐前实际使用了的大小）  
> 特别的，英文文档指出当 VirtualSize > SizeOfRawData 时表示块内数据全部是 0，通常下 VirtualSize <= SizeOfRawData。  
> 由此看来英文文档中的对齐要求不无理由，除 VirtualSize 和块实际存储信息有关（可能是代码多少，数据多少），其它三项都要求对齐。  
> 2 个参考参数：  
> 分别是 BYTE name\[8\];块名字，详细参见英文文档和 Characteristics 表示块的属性（可执行？保存已初始化数据？等等）  
> 还有一些和调试相关的参数详细参见英文文档。  
>   
> 既然对齐如此必要，那么理应严格遵循，但是试验结果令人大吃一惊。  
> FileAlignment 使用了 win2k\\system32\\notepad.exe 作为试验对象。  
> Notepad.exe 的块表：  
> name  VOffset    VSize    ROffset    RSize    Flags  
> .text  00001000  0000654A  00000600  00006600  60000020  
> .data  00008000  00001944  00006C00  00000600  C0000040  
> .rsrc  0000A000  00005238  00007200  00005400  40000040  
>   
> FileAlignment : 200h  SectionAlignment : 1000h    运行结果：ok  
> FileAlignment : 400h  SectionAlignment : 1000h    运行结果：ok  
> FileAlignment : 600h  SectionAlignment : 1000h    运行结果：错误，原因：600h 不是 2 的整数次幂  
> FileAlignment : 800h  SectionAlignment : 1000h    运行结果：ok  
> FileAlignment : 1000h  SectionAlignment : 1000h    运行结果：ok  
> FileAlignment : 2000h  SectionAlignment : 1000h    运行结果：错误，原因：FileAlignment > SectionAlignment  
>   
> 可见 FileAlignment 并没有严格遵循对齐原则。  
> 而 SectionAlignment 是严格遵循的，因为它关系到 SizeOfImage，EntryPoint 等与 RVA 和 VA 相关的所有内容。  
>   
> 最后的结果大出所料但又在情理之中，文件中 FileAlignment 没有严格遵循对齐原则，因为在映射到内存时地址会重新改写，只要内容上没有问题，映射后与映像文件对齐没有关系。但是 SectionAlignment 却要严格遵循，因为映射到内存后诸多寻址与其息息相关。  
> PELoader 和我们玩了一个游戏，只要 EXE 能够执行，不必细究那些次要因素带来的影响。  
> 当然这些推断只是基于本环境（win2k+sp4），其它系统上不敢妄言，由于看不到系统 PELoader 的代码实现，所以这些只是表象的推测而已，更待高手指点迷津。  
>   
> 附件中带有 win2k 下的 notepad，英文版的文档。 

> ROffset 可以不对齐,这个时候会在 load 的时按 FileAlignment 向下对齐  
> VSize 的大小只是效验下是否跨越下一个节了,或者是否超出了 SizeOfImage,如果出现越界问题,提示非法 32 位应用程序,否则的话,它的值没有意义,节的大小不是由它决定的......对非最后一个节,按节间 VOffset 之差,最后一节用 SizeOfImage-VOffset 

> 不管 VirtualSize 和 SizeOfRawData 的关系怎么样,貌似都是从磁盘上读取 SizeOfRawData 大小数据填充该节的头部,剩下的空间全填充 0 

 4dd466d0e1e0cc123afbe504da2462ca