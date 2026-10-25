
# SDTrestore Version 0.2 学习注释存档

> 原文：[`www.pediy.com/kssd/pediy10/58157.html`](https://www.pediy.com/kssd/pediy10/58157.html)

```c
/*******************************************************************************
 * Name   : SDTrestore Version 0.2  
 * Author : Coded by Chew Keong TAN  
 * Learner: sudami [xiao_rui_119@163.com]  
 * Time   : 08/01/12  
 *  
 * Comment:  
 *  
 *  适用 XP 和 WIN2K  
 *  SDT 恢复是一个古老的技术了,这些天复习以前的知识,发现东西都忘了.正好找到了这份源码.  
 *  于是开始边注释边复习起来. 现在终于又把 RING3 下恢复 SSDT 的东西温习了一下,感觉很充实.  
 *  偶很菜,一开始看代码的时候,不知道如何下手,因为原作者的代码风格很乱,看得糊涂.于是偶  
 *  按照 M$ 的代码风格重新注释了一遍,方便以后温习. 如果你还木有看过这份源码的话,在偶注释  
 *  的基础上应该会理解的更快些了.  
 *  
 *  搜索了下坛子,发现没有关于这份源码的相关细节.于是偶来补充下.  
 *  存档的主要目的是方便以后学习这些知识的同学更快的找到资源,少走弯路. 老鸟飘过~![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  
 *  
 * Description:  
 *  
 *  在 RING3 获得\\device\\physicalmemory 的读写权限(前提必须是 Administrator). 从磁盘读取  
 *  ntoskrnl.exe,对齐后映射进内存(这需要我们自己来做),在其 EAT 中得到 KeServiceDescriptorTable  
 *  相对于 ntoskrnl.exe 的偏移.把系统高端加载的 ntoskrnl.exe 的 SSDT 地址映射到进程的虚拟地址空间里  
 *  推算出 ServiseTable 的地址后,把其所有内容映射到进程的虚拟地址空间里. 得到了 ServiseTable 的地址  
 *  但它里面的函数是否被 HOOK 过,并不知道.所以需要参照我们自己映射的 ntoskrnl.exe 中的 ServiseTable  
 *  里函数的偏移来作比较.不相同,则被 HOOK 了,然后恢复之.  
 *  
 *  btw: 90210 的那个搜索 SSDT 的方法好像不行.得到的地址不正确.而且导致系统直接重启  
 *  
 *******************************************************************************/
```

注释风格类似如下：

```c
BOOL 
LoadPE (
  char *exePtr, 
  MZHeader *inMZ, 
  PE_Header *inPE, 
  PE_ExtHeader *inpeXH, 
  SectionHeader *inSecHdr, 
  LPVOID ptrLoc 
  ) 
/*++
学习者: sudami [xiao_rui_119@163.com]
时间  : 08/01/11
功能:
  加载从磁盘读取的文件到内存,需要自己对齐
           <PE 加载到内存对齐后的示意图>
      所有文件头     节     节    ...     节 N 
     ------------------------------------------------
    |aa...aa######| a...a##| a...a###| ... | a...a##|
     ------------------------------------------------
  注: a 表示实际数据; # 表示空隙,是内存中对齐后留出的空隙
参数:
  exePtr - PE 文件在磁盘中时起始的地址.还没有经过对齐.所以需要把它映射到内存
  modulePos - [IN] 将磁盘上的文件读出来后,映射到进程的虚拟地址空间中,对齐后得到其起始地址.
  outMZ - [OUT] 保存 MZ 头 IMAGE_DOS_HEADER
  outPE - [OUT] 保存 PE 头 IMAGE_FILE_HEADER
  outpeXH - [OUT] 保存扩展 PE 头 IMAGE_OPTIONAL_HEADER
  outSecHdr - [OUT] 2 级指针,保存 Section 头 IMAGE_SECTION_HEADER
  ptrLoc - 根据内存中 PE 的大小调用 HeapAlloc 而分配的一块空闲内存区域.
返回: TRUE
--*/

{
    char *outPtr = (char *)ptrLoc;
    // 把所有文件头的数据全部拷贝到 ptrLoc 指向的地址处
    // 然后地址按照 PE 在内存中的对齐方式往后挪
    memcpy (outPtr, exePtr, inpeXH->sizeOfHeaders);
    outPtr += GetAlignedSize (inpeXH->sizeOfHeaders, inpeXH->sectionAlignment);
    // 拷贝所有的节数据到新的地址处. 复制的时候要知道每个节在文件中的实际大小.而不是在内存在对齐后的大小
    for (int i = 0; i < inPE->numSections; i++) {
        if (inSecHdr[i].sizeOfRawData > 0) { // sizeOfRawData 是节对齐后的长度
            ULONG toRead = inSecHdr[i].sizeOfRawData;
            if (toRead > inSecHdr[i].virtualSize) {
                toRead = inSecHdr[i].virtualSize;
            }
            // 拷贝没个节的实际数据,然后把指针挪到每个节在内存中对齐后的位置处
            memcpy (outPtr, exePtr + inSecHdr[i].pointerToRawData, toRead);
            outPtr += GetAlignedSize (inSecHdr[i].virtualSize, inpeXH->sectionAlignment);
        }
    }
    return true;
}
```

希望偶们这样的菜鸟能一起共同成长~ ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

上传的附件

| 附件 | 文件名 |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `RecoverSSDT.rar` |
