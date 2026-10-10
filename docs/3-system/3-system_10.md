

# PE 文件结构处理经验总结 1

> 原文：[`www.pediy.com/kssd/pediy12/142701.html`](https://www.pediy.com/kssd/pediy12/142701.html)

【文章标题】: PE 文件结构处理经验总结 1  
【文章作者】: ddlx  
【作者邮箱】: DdlxStudio@163.com  
【作者 QQ 号】: 383394019  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  该贴不是扫盲贴，如果对 Pe 文件不是很熟悉，请先查阅其他人写的 PE 文章。  
  该贴是我在写加壳工具的时候遇到问题的总结。记录下来，供以后温习，希望对各位也有一定帮助。  
  由于本人接触该方面时间不长，免不了存在失误之处，敬请拍砖。  
    
  1. IMAGE\_DOS\_HEADER  
  Dos 头需要注意的有两个字段:e\_magic 和 e\_lfanew。其它字段不用管它。这两个字段的意义大家都懂得~  
    
  2. IMAGE\_NT\_HEADER  
  通过 Nt 头的 Signature 字段和 Dos 头的 e\_magic 来判断该文件是否为 PE 文件。  
    
  3. IMAGE\_FILE\_HEADER  
  字段：Machine。该字段通常用来判断该 PE 文件是否为 64 位格式。我原先使用字段 Characteristics 的 IMAGE\_FILE\_32BIT\_MACHINE 位来判断是否为 64 位，发现这种方法不准确，32 位文件可以不设置 IMAGE\_FILE\_32BIT\_MACHINE 位。  
  字段:Characteristics。这里面记录了好多文件属性。我当前只判断 IMAGE\_FILE\_DLL，来确认该文件是否为动态链接库 Dll.（IMAGE\_FILE\_32BIT\_MACHINE 标记判断是否为 64 位是不可靠的）  
  字段: NumberOfSections。这个是区段(也叫节)个数，遍历区段头时使用该字段来控制是否遍历结束。我原先看 PE 文章说在遍历区段头时判断区段头数据是否为空就知道是否遍历结束了，经过我亲自试验，这种说法是错误的。  
  字段: SizeOfOptionalHeader. 想知道可选文件头大小就使用这个字段。有人计算可选文件头大小时使用 sizeof(IMAGE\_OPTIONAL\_HEADER32/64),这个也是错误的，因为数据目录的个数可能不止 16 个。  
    
  4. IMAGE\_OPTIONAL\_HEADER  
  该字段有 32 和 64 两种版本。  
  通过 BaseOfCode 和 SizeOfCode 来标记代码段起始位置和大小是不可取的。数据段也一样。  
  SectionAlignment 和 FileAlignment 在增加或删除区段的时候有用。如果区段变更，一定要修正 SizeOfImage。否则你的程序就无法被系统加载。  
  SectionAlignment 必须>=FileAlignment。并且都是 2 的幂数。  
  字段： SizeOfHeaders。这个是整个 Pe 文件头大小。CopyPE 文件头数据时使用。有些人在复制文件头时使用 memcpy(pToModBase, pFromModBase, pSectionH\[0\].PointerToRawData);以后不要这样用了，因为有一些加过壳(特别是压缩壳)的程序 pSectionH\[0\].PointerToRawData 被置成了 0.  
  字段：CheckSum。校验和只有在驱动文件中有用。驱动加载失败首先要看一下校验和是否正确。  
  字段：NumberOfRvaAndSizes。不是永远等于 16。可以任意。  
    
    
  5. IMAGE\_SECTION\_HEADER  
  字段：Name.获取 Name 时不要使用 strcpy(pSecName,pSecH\[0\].Name); 应该使用 memcpy(pSecName, pSecH\[0\].Name,IMAGE\_SIZEOF\_SHORT\_NAME);  
  字段：VirtualAddress。该值必须是 SectionAlignment 的倍数。VirtualAddress 不能为 0  
  字段：PointerToRawData。该值必须是 FileAlignment 的倍数。PointerToRawData 可以为 0.  
  字段：VirtualSize：不必是 SectionAlignment 的倍数。  
  字段：SizeOfRawData：不必是 FileAlignment 的倍数。可以为 0.  
  字段： Characteristics。 该字段包含了区段属性信息。可以通过 IMAGE\_SCN\_MEM\_EXECUTE 标志来判断是否为可执行区段，不是很准。  
  另外：  
  1. 相邻的两个节数据不一定是连续的!  
  2. 获取第一个区段头地址方法：  
  #define IMAGE\_FIRST\_SECTION( ntheader ) ((PIMAGE\_SECTION\_HEADER)        \\  
      ((ULONG\_PTR)ntheader +                                              \\  
       FIELD\_OFFSET( IMAGE\_NT\_HEADERS, OptionalHeader ) +                 \\  
       ((PIMAGE\_NT\_HEADERS)(ntheader))->FileHeader.SizeOfOptionalHeader   \\  
      ))  
  不要使用：  
  #define IMAGE\_FIRST\_SECTION( ntheader ) ((PIMAGE\_SECTION\_HEADER)        \\  
      ((ULONG\_PTR)ntheader +                                              \\  
       FIELD\_OFFSET( IMAGE\_NT\_HEADERS, OptionalHeader ) +                 \\  
       sizeof(IMAGE\_OPTIONAL\_HEADER)   \\  
      ))  
  3. 如果你要增加一个区段，一定要先判断一下区段头后面是否存在有用数据。如果你把有用数据给覆盖了，模块就加在不起来了。例如：记事本程序的区段头后面就是有用数据：Bound Import Directory。  
4. 不要看见区段名叫.text 或 Code 就认为是代码段。区段名可以任意写的。  
    
  6. IMAGE\_EXPORT\_DIRECTORY 导出表  
  字段:Base。意思是基数。通常为 1。可以为任意数。不能为 0.  
  字段：NumberOfNames。NumberOfNames <= NumberOfFunctions  
  字段：NumberOfFunctions。该值可以为 0，就是没有导出函数。  
  另外：1. 导出表只有 1 个。有的童鞋可能认为存在多张导出表。  
  2. 转发函数：这是一个特殊的导出函数。如果你自己实现 GetProcAddress 函数，可能会遇到这个问题。如果该导出函数是一个转发函数，那么这个函数地址就是不函数指令了，而是转发的那个函数字符串。例如 Kernel.dll 中得导出函数 HeapAlloc，函数地址处是字符串 NTDLL.RtlAllocateHeap。你如果调用系统的 GetProcAddress(pKernel32Base, "HeapAlloc"),得到的地址是 Ntdll 中得 RtlAllocateHeap 函数地址。  
  3. 转发函数判断方法：得到函数地址后，判断这个函数是否在导出表数据内。如果是，则为转发函数。  
  4. 根据函数序号得函数地址的方法： 1. 先得函数地址表索引：索引 = 序号 - 基数；2. 根据索引得函数地址。pAddressOfFunctions\[序号 - 基数\]  
    
  7. IMAGE\_IMPORT\_DESCRIPTOR 导入表  
  字段：OriginalFirstThunk。该字段可以为 0.也可以 FirstThunk == OriginalFirstThunk  
  字段：FirstThunk，该字段必须不为 0.如果 OriginalFirstThunk 为 0，则在模块加载时 FirstThunk 担任 OriginalFirstThunk 角色。  
    
  8. IMAGE\_BASE\_RELOCATION 重定位表  
  判断重定位表结束的方法：VirtualAddress==0 && SizeOfBlock==0. 只判断 VirtualAddress 是不行的。我就犯过这个错误~  
    
  9. IMAGE\_TLS\_DIRECTORY TLS  
  当时为了很好的处理 TLS，让加好壳的程序 TLS 运行正常，伤了很多脑细胞。  
  TLS 分为两块：静态数据 和 TLS CallBack  
  TLS 的静态数据，最需要注意的是初始化时不为 0 的数据。不要都给他填充为 0.  
  TLS CallBack，当时需要增加 TLS 反调试功能，所以对 CallBack 表重新构造了一下。  
    
  字段：StartAddressOfRawData：TLS 静态数据开始地址。  
  字段：EndAddressOfRawData：TLS 静态数据结束地址。  
  字段：SizeOfZeroFill，直接填充为 0 的大小。  
  上面 3 个字段都是跟静态数据大小有关的。静态数据总大小就是 EndAddressOfRawData-StartAddressOfRawData+SizeOfZeroFill.  
  字段：AddressOfIndex。跟 TLS 静态数据有关。如果你要压缩数据，一定要对该字段特殊处理。这个是地址索引，系统设置的，跟进程中拥有 TLS 的模块个数有关。  
  为什么说要特别注意？因为该数据存在数据段，如果数据段被压缩，加载时在解压的话，AddressOfIndex 会在你解压时填充为 0.由于 AddressOfIndex 的索引是在你解压之前设置的，就是说你把系统分配的索引又摸回去了。  
    
    
  10. Load Config Directory  
  该数据目录项，在加壳时一定要给他清空。否则壳中的异常可能得不到处理。  
  该结构中得最后两个字段保存了该模块出现异常时的异常处理地址表。如果你的异常处理地址不在那张列表中，系统就不会处理你的异常。  
    
\--------------------------------------------------------------------------------  
  不足之处，欢迎拍砖。 未完待续。。。  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2011 年 11 月 12 日 13:35:58