

# 自己动手打造 PE Show

> 原文：[`www.pediy.com/kssd/pediy06/pediy6695.htm`](https://www.pediy.com/kssd/pediy06/pediy6695.htm)

深入浅出 PE 文件格式---自己动手打造 PE Show  
                                                            作者：WiNrOOt  
////////////////////////////////////////////////////////////////////  
//                            开篇                               //  
///////////////////////////////////////////////////////////////////  
   大家好！我一位菜鸟，学习加密解密已经有一段时间了，可是对于脱壳总是似懂非懂，心中甚是不爽。  
于是就从 PE 结构开始，在学习的过程中我发现要真正了解 PE 文件结构就必须动手，只有动手您才能看懂他，  
感受他，直到你应用他………………这是我的一点废话希望大家不要嫌烦。下面是是我的学习笔记，希望能给大家  
带来一点提示，文中有不对的地方请各位大虾指正。谢谢！  
//////////////////////////////////////////////////////////////////  
//                            准备                             //  
/////////////////////////////////////////////////////////////////  
开篇之前我想大家起码要有几点准备：  
1。Iczelion's Win32 Assembly 的教程我们主要是围绕他的 PE 教程来实现我们的函数功能。  
    （其实 Win32ASM Tutorial Resource Kit v1.00 Collected and packed by dREAMtHEATER 就包含这些  
    还是翻译过的。下载地址[WWW.PEDIY.COM](http://WWW.PEDIY.COM))  
    下载回来希望您能看一下，这样大家就好交流:-)  
  
2。一个开发环境。（我用的是 VC++6.0）  
3。一个适合你研究东西的环境。  
4.源码本地下载（或点击右键另存为）   
////////////////////////////////////////////////////////////////  
//                           正文                            //  
///////////////////////////////////////////////////////////////  
  
     “ PE 的意思就是 Portable Executable（可移植的执行体）。它是 Win32 环境自身所带的执行体文件格  
式。它的一些特性继承自 Unix 的 Coff (common object file format)文件格式。"portable executable"  
（可移植的执行体）意味着此文件格式是跨 win32 平台的 :  
即使 Windows 运行在非 Intel 的 CPU 上，任何 win32 平台的 PE 装载器都能识别和使用该文件格式。当然，移植到不  
同的 CPU 上 PE 执行体必然得有一些改变。所有 win32 执行体  
(除了 VxD 和 16 位的 Dll)都使用 PE 文件格式，包括 NT 的内核模式驱动程序（kernel mode  
drivers）。因而研究 PE 文件格式给了我们洞悉 Windows 结构的良机。“  
      好了，上面这段话就是我们为什么要研究 PE 文件结构。 

看图![](img/42dda29a21b69bf4490d8979929e5886.jpg)，这张图我相信大家不陌生，第一块是 DOS MZ header 这是什么呢？  
      这张图的每一块都是什么意思呢？  
        
那么我们就来开始动手。  
 我们要设计我们自己的 PE TOOLS----PE Show  
 主要功能：1 判断文件是否是 PE 文件。  
           2 显示 pe 文件的相关信息。  
  
1.打开文件代码如下： 

> ```
> 代码:
> if(FALSE==PEfile.Open(m\_filename,CFile::typeBinary|CFile::shareDenyNone))
>   {
>     MessageBox("文件打不开!");
>     return; 
>   }
> ```
> 
>   

  
CFile 类的使用方法希望大及自己去查找 msdn  
2。文件我们打开了而且是以 Binary 方式打开的，下面我们该干什么了？  
  
  编写第一个功能-----检验 PE 文件的有效性  
在 Iczelion's 的教程中有这样一段话：  
“1。首先检验文件头部第一个字的值是否等于 IMAGE\_DOS\_SIGNATURE，是则 DOS MZ header 有效。  
 2。一旦证明文件的 DOS header 有效后，就可用 e\_lfanew 来定位 PE header 了。  
 3。比较 PE header 的第一个字的值是否等于 IMAGE\_NT\_HEADER。  
    如果前后两个值都匹配，那我们就认为该文件是一个有效的 PE 文件。  
这就是检验 PE 文件有效性的流程。  
从上面那段话我们看出判断的关键是 PE header 的第一个字的值是否等于 IMAGE\_NT\_HEADER  
直到这些我们就倒着找。  
PE header 的第一个字的值是什么？  
我么就来看一下 IMAGE\_NT\_HEADERS 的结构：（查看 WINNT.H 就找到了）  

> ```
> 代码:
> typedef struct \_IMAGE\_NT\_HEADERS {
>     DWORD Signature;
>     IMAGE\_FILE\_HEADER FileHeader;
>     IMAGE\_OPTIONAL\_HEADER32 OptionalHeader;
> } IMAGE\_NT\_HEADERS32, \*PIMAGE\_NT\_HEADERS32;
> ```
> 
>   

  
Signature 一 dword 类型，值为 50h, 45h, 00h, 00h（PE\\0\\0）。 本域为 PE 标记，我们可以此识别给定文件是否为有效 PE 文件。  
FileHeader 该结构域包含了关于 PE 文件物理分布的信息， 比如节数目、文件执行机器等。  
OptionalHeader 该结构域包含了关于 PE 文件逻辑分布的信息，虽然域名有"可选"字样，但实际上本结构总是存在的。  
我们目的很明确。如果 IMAGE\_NT\_HEADERS 的 signature 域值等于"PE\\0\\0"，那么就是有效的 PE 文件。实际上，为了比较方便，Microsoft 已定义了常量

> ```
> 代码:IMAGE\_NT\_SIGNATURE 供我们使用。
> IMAGE\_DOS\_SIGNATURE equ 5A4Dh 
> IMAGE\_OS2\_SIGNATURE equ 454Eh 
> IMAGE\_OS2\_SIGNATURE\_LE equ 454Ch 
> IMAGE\_VXD\_SIGNATURE equ 454Ch 
> IMAGE\_NT\_SIGNATURE equ 4550h 
> ```
> 
>   

  
  
判断的问题我们解决了，新的问题又来了我们如何定位 IMAGE\_NT\_HEADERS 结构的位置。  
MS 肯定有办法，PE 文件的开头是什么？DOS MZ header 结构，我们来看一下他的定义：

> ```
> 代码:
> typedef struct \_IMAGE\_DOS\_HEADER {      // DOS .EXE header
>     WORD   e\_magic;                     // Magic number
>     WORD   e\_cblp;                      // Bytes on last page of file
>     WORD   e\_cp;                        // Pages in file
>     WORD   e\_crlc;                      // Relocations
>     WORD   e\_cparhdr;                   // Size of header in paragraphs
>     WORD   e\_minalloc;                  // Minimum extra paragraphs needed
>     WORD   e\_maxalloc;                  // Maximum extra paragraphs needed
>     WORD   e\_ss;                        // Initial (relative) SS value
>     WORD   e\_sp;                        // Initial SP value
>     WORD   e\_csum;                      // Checksum
>     WORD   e\_ip;                        // Initial IP value
>     WORD   e\_cs;                        // Initial (relative) CS value
>     WORD   e\_lfarlc;                    // File address of relocation table
>     WORD   e\_ovno;                      // Overlay number
>     WORD   e\_res\[4\];                    // Reserved words
>     WORD   e\_oemid;                     // OEM identifier (for e\_oeminfo)
>     WORD   e\_oeminfo;                   // OEM information; e\_oemid specific
>     WORD   e\_res2\[10\];                  // Reserved words
>     LONG   e\_lfanew;                    // File address of new exe header
>   } IMAGE\_DOS\_HEADER, \*PIMAGE\_DOS\_HEADER;
> ```
> 
>   

  
  看一下最后一项！！！！！  
  发现了吗？他指向的就是 PE header  
  那么 DOS MZ header 的位置怎么确定呢？呵呵，他就是文件的开始  
  看代码：

> ```
> 代码:
>         PEfile.Read(&stPEDosHeader,sizeof(\_IMAGE\_DOS\_HEADER));
>   if(stPEDosHeader.e\_magic!=IMAGE\_DOS\_SIGNATURE)//"MZ"
>   {
>     MessageBox("DOS MZ header 无效!");
>     PEfile.Close();
>     return;
>   }
>   else
>   {
>     //-----------------------
>     //显示 DOS Header
>     //-----------------------
>     UpdateData(true);
>     m\_Magicnumber.Format(\_T("0x%.4X"),stPEDosHeader.e\_magic);
>     m\_cblp.Format(\_T("0x%.4X"),stPEDosHeader.e\_cblp);
>     m\_cp.Format(\_T("0x%.4X"),stPEDosHeader.e\_cp);
>     m\_crlc.Format(\_T("0x%.4X"),stPEDosHeader.e\_crlc);
>     m\_cparhdr.Format(\_T("0x%.4X"),stPEDosHeader.e\_cparhdr);
>     m\_minalloc.Format(\_T("0x%.4X"),stPEDosHeader.e\_minalloc);
>     m\_maxalloc.Format(\_T("0x%.4X"),stPEDosHeader.e\_maxalloc);
>     m\_ss.Format(\_T("0x%.4X"),stPEDosHeader.e\_ss);
>     m\_sp.Format(\_T("0x%.4X"),stPEDosHeader.e\_sp);
>     m\_csum.Format(\_T("0x%.4X"),stPEDosHeader.e\_csum);
>     m\_ip.Format(\_T("0x%.4X"),stPEDosHeader.e\_ip);
>     m\_cs.Format(\_T("0x%.4X"),stPEDosHeader.e\_cs);
>     m\_lfarlc.Format(\_T("0x%.4X"),stPEDosHeader.e\_lfarlc);
>     m\_ovno.Format(\_T("0x%.4X"),stPEDosHeader.e\_ovno);
>     m\_oemid.Format(\_T("0x%.4X"),stPEDosHeader.e\_oemid);
>     m\_oeminfo.Format(\_T("0x%.4X"),stPEDosHeader.e\_oeminfo);
>     m\_lfanew.Format(\_T("0x%.8X"),stPEDosHeader.e\_lfanew);
>     UpdateData(false);
>   }
> 
> 
>   buf=stPEDosHeader.e\_lfanew;  //确定\_IMAGE\_DOS\_HEADER 偏移
>   try{PEfile.Seek(buf,CFile::begin);}
>   catch(...)
>   {
>     MessageBox("\_IMAGE\_DOS\_HEADER.e\_lfanew 不对!");
>     PEfile.Close();
>     return;
>   }
>   PEfile.Read(&stPEHeader,sizeof(\_IMAGE\_NT\_HEADERS));//----------NT 头
>   if(stPEHeader.Signature!=IMAGE\_NT\_SIGNATURE)//"PE\\0\\0"
>   {
>     MessageBox("该文件不是 PE 格式!");
>     PEfile.Close();
>     return;
>   }
>   else
>        {
>            MessageBox("该文件是 PE 格式!");
>            PEfile.Close();
>     return;
>          }  
> 
> ```
> 
>   

  
  
好了现在我们已经确定了文件是否是有效的 PE 文件。顺便我们把 IMAGE\_DOS\_HEADER 的结构成员都显示出来了。  
我们已经写了一个功能。  
下面我们继续顺着往下看：  
IMAGE\_DOS\_HEADER 结构结束那么就是 IMAGE\_NT\_HEADERS 开始。结构体的成员情况前面我们已经介绍过了  
下面我们来提取他们：

> ```
> 代码:
>                 //---------------------------
>     //显示 IMAGE\_FILE\_HEADER 结构
>     //---------------------------
>     UpdateData(true);
>     m\_Machine.Format(\_T("0x%.4X"),stPEHeader.FileHeader.Machine);
>     m\_NumberOfSections.Format(\_T("0x%.4X"),stPEHeader.FileHeader.NumberOfSections);
>     m\_TimeDateStamp.Format(\_T("0x%.8X"),stPEHeader.FileHeader.TimeDateStamp);
>         m\_PointerToSymbolTable.Format(\_T("0x%.8X"),stPEHeader.FileHeader.PointerToSymbolTable);
>     m\_NumberOfSymbols.Format(\_T("0x%.8X"),stPEHeader.FileHeader.NumberOfSymbols);
>     m\_SizeOfOptionalHeader.Format(\_T("0x%.4X"),stPEHeader.FileHeader.SizeOfOptionalHeader);
>     m\_Characteristics.Format(\_T("0x%.4X"),stPEHeader.FileHeader.Characteristics);
>     UpdateData(false);
> ```
> 
>   

         
这是 IMAGE\_FILE\_HEADER FileHeader 的成员，我们已经将他们提取出来。  
看教程我们继续寻找 IMAGE\_OPTIONAL\_HEADER 的成员，他是结构体中的结构体我们就顺着找。

> ```
> 代码:
> typedef struct \_IMAGE\_OPTIONAL\_HEADER {
>     //
>     // Standard fields.
>     //
> 
>     WORD    Magic;
>     BYTE    MajorLinkerVersion;
>     BYTE    MinorLinkerVersion;
>     DWORD   SizeOfCode;
>     DWORD   SizeOfInitializedData;
>     DWORD   SizeOfUninitializedData;
>     DWORD   AddressOfEntryPoint;
>     DWORD   BaseOfCode;
>     DWORD   BaseOfData;
> 
>     //
>     // NT additional fields.
>     //
> 
>     DWORD   ImageBase;
>     DWORD   SectionAlignment;
>     DWORD   FileAlignment;
>     WORD    MajorOperatingSystemVersion;
>     WORD    MinorOperatingSystemVersion;
>     WORD    MajorImageVersion;
>     WORD    MinorImageVersion;
>     WORD    MajorSubsystemVersion;
>     WORD    MinorSubsystemVersion;
>     DWORD   Win32VersionValue;
>     DWORD   SizeOfImage;
>     DWORD   SizeOfHeaders;
>     DWORD   CheckSum;
>     WORD    Subsystem;
>     WORD    DllCharacteristics;
>     DWORD   SizeOfStackReserve;
>     DWORD   SizeOfStackCommit;
>     DWORD   SizeOfHeapReserve;
>     DWORD   SizeOfHeapCommit;
>     DWORD   LoaderFlags;
>     DWORD   NumberOfRvaAndSizes;
>     IMAGE\_DATA\_DIRECTORY DataDirectory\[IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES\];
> } IMAGE\_OPTIONAL\_HEADER32, \*PIMAGE\_OPTIONAL\_HEADER32;
> 
> 
> ```
> 
>   

  
  
看到这些成员将他们全部显示出来  

> ```
> 代码:
> 
>                  //--------------------------------
>     //显示 IMAGE\_OPTIONAL\_HEADER
>     //--------------------------------
>     UpdateData(true);
>     m\_Magic.Format(\_T("0x%.4X"),stPEHeader.OptionalHeader.Magic);
>     m\_MajorLinkerVersion.Format(\_T("0x%.2X"),stPEHeader.OptionalHeader.MajorLinkerVersion);
>     m\_MinorLinkerVersion.Format(\_T("0x%.2X"),stPEHeader.OptionalHeader.MinorLinkerVersion);
>     m\_SizeOfCode.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SizeOfCode);
>     m\_SizeOfInitializedData.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SizeOfInitializedData);
>     m\_SizeOfUninitializedData.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SizeOfUninitializedData);
>     m\_AddressOfEntryPoint.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.AddressOfEntryPoint);
>     m\_BaseOfCode.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.BaseOfCode);
>     m\_BaseOfData.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.BaseOfData);
>     m\_ImageBase.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.ImageBase);
>     m\_SectionAlignment.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SectionAlignment);
>     m\_FileAlignment.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.FileAlignment);
>     m\_MajorOperatingSystemVersion.Format(\_T("0x%.4X"),stPEHeader.OptionalHeader.MajorOperatingSystemVersion);
>     m\_MinorOperatingSystemVersion.Format(\_T("0x%.4X"),stPEHeader.OptionalHeader.MinorOperatingSystemVersion);
>     m\_MajorImageVersion.Format(\_T("0x%.4X"),stPEHeader.OptionalHeader.MajorImageVersion);
>     m\_MinorImageVersion.Format(\_T("0x%.4X"),stPEHeader.OptionalHeader.MinorImageVersion);
>     m\_MajorSubsystemVersion.Format(\_T("0x%.4X"),stPEHeader.OptionalHeader.MajorSubsystemVersion);
>     m\_MinorSubsystemVersion.Format(\_T("0x%.4X"),stPEHeader.OptionalHeader.MinorSubsystemVersion);
>     m\_Win32VersionValue.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.Win32VersionValue);
>     m\_SizeOfImage.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SizeOfImage);
>     m\_SizeOfHeaders.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SizeOfHeaders);
>     m\_CheckSum.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.CheckSum);
>     m\_Subsystem.Format(\_T("0x%.4X"),stPEHeader.OptionalHeader.Subsystem);
>     m\_DllCharacteristics.Format(\_T("0x%.4X"),stPEHeader.OptionalHeader.DllCharacteristics);
>     m\_SizeOfStackReserve.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SizeOfStackReserve);
>     m\_SizeOfStackCommit.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SizeOfStackCommit);
>     m\_SizeOfHeapReserve.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SizeOfHeapReserve);
>     m\_SizeOfHeapCommit.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.SizeOfHeapCommit);
>     m\_LoaderFlags.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.LoaderFlags);
>     m\_NumberOfRvaAndSizes.Format(\_T("0x%.8X"),stPEHeader.OptionalHeader.NumberOfRvaAndSizes);
>     UpdateData(false); 
>     
> 
> ```
> 
>   

  
我们已经学了许多关于 DOS header 和 PE header 的知识。接下来就该轮到 section table（节表）了。  
节表其实就是紧挨着 PE header 的一结构数组。该数组成员的数目由 file header (IMAGE\_FILE\_HEADER)   
结构中 NumberOfSections 域的域值来决定。节表结构又命名为 IMAGE\_SECTION\_HEADER。

> ```
> 代码:
> typedef struct \_IMAGE\_SECTION\_HEADER {
>     BYTE    Name\[IMAGE\_SIZEOF\_SHORT\_NAME\];
>     union {
>             DWORD   PhysicalAddress;
>             DWORD   VirtualSize;
>     } Misc;
>     DWORD   VirtualAddress;
>     DWORD   SizeOfRawData;
>     DWORD   PointerToRawData;
>     DWORD   PointerToRelocations;
>     DWORD   PointerToLinenumbers;
>     WORD    NumberOfRelocations;
>     WORD    NumberOfLinenumbers;
>     DWORD   Characteristics;
> } IMAGE\_SECTION\_HEADER, \*PIMAGE\_SECTION\_HEADER
> ```
> 
>   

  
  
Name     这儿的节名长不超过 8 字节。记住节名仅仅是个标记而已，我们选择任何名字甚至空着也行，注意这里不用 null 结束。命名不是一个 ASCIIZ 字符串，所以不用 null 结尾。   
VirtualAddress 本节的 RVA（相对虚拟地址）。PE 装载器将节映射至内存时会读取本值，因此如果域值是 1000h，而 PE 文件装在地址 400000h 处，那么本节就被载到 401000h。   
SizeOfRawData 经过文件对齐处理后节尺寸，PE 装载器提取本域值了解需映射入内存的节字节数。（译者注: 假设一个文件的文件对齐尺寸是 0x200，如果前面的 VirtualSize 域指示本节长度是 0x388 字节，则本域值为 0x400，表示本节是 0x400 字节长）。   
PointerToRawData 这是节基于文件的偏移量，PE 装载器通过本域值找到节数据在文件中的位置。   
Characteristics 包含标记以指示节属性，比如节是否含有可执行代码、初始化数据、未初始数据，是否可写、可读等。  
  
  
现在我们已知晓 IMAGE\_SECTION\_HEADER 结构，再来模拟一下 PE 装载器的工作吧:  
  
1 读取 IMAGE\_FILE\_HEADER 的 NumberOfSections 域，知道文件的节数目。   
2 SizeOfHeaders 域值作为节表的文件偏移量，并以此定位节表。   
3 遍历整个结构数组检查各成员值。   
4 对于每个结构，我们读取 PointerToRawData 域值并定位到该文件偏移量。然后再读取 SizeOfRawData 域值来决定映射内存的字节数。将 VirtualAddress 域值加上 ImageBase 域值等于节起始的虚拟地址。然后就准备把节映射进内存，并根据 Characteristics 域值设置属性。   
5 遍历整个数组，直至所有节都已处理完毕。  
代码如下：

> ```
> 代码:
>                 //------------------------------
>     //显示 Section 结构
>     //------------------------------
>     nSection=stPEHeader.FileHeader.NumberOfSections;
>     stSectionHeader=new \_IMAGE\_SECTION\_HEADER\[nSection\];
>     m\_ListCtrl.DeleteAllItems();
>     for(int i=0;i<nSection;i++)
>     {
>       
>       PEfile.Read(&stSectionHeader\[i\],sizeof(\_IMAGE\_SECTION\_HEADER));//-----节表
>       //NO
>       szTemp.Format(\_T("%.2d"), i+1);
>       m\_ListCtrl.InsertItem(i,szTemp,i);
>       //SectionName
>       strcpy(chSectionName,(LPCSTR)stSectionHeader\[i\].Name);
>              m\_ListCtrl.SetItemText(i,1,chSectionName);
>       //VirtualSize
>       szTemp.Format(\_T("0x%.8X"),stSectionHeader\[i\].Misc.VirtualSize);
>                 m\_ListCtrl.SetItemText(i,2,szTemp);
>       //VirtualAddress
>       szTemp.Format(\_T("0x%.8X"),stSectionHeader\[i\].VirtualAddress );
>                 m\_ListCtrl.SetItemText(i,3,szTemp);
>       //SizeOfRawData
>       szTemp.Format(\_T("0x%.8X"),stSectionHeader\[i\].SizeOfRawData );
>       m\_ListCtrl.SetItemText(i,4,szTemp);
>       //SizeOffset
>       szTemp.Format(\_T("0x%.8X"),stSectionHeader\[i\].PointerToRawData );
>       m\_ListCtrl.SetItemText(i,5,szTemp);
>       //Characteristics
>       szTemp.Format(\_T("0x%.8X"),stSectionHeader\[i\].Characteristics );
>       m\_ListCtrl.SetItemText(i,6,szTemp);
>       
>       
>     }
>     delete stSectionHeader;
> ```
> 
>   

  
好了，我们计划的两个功能已经写完  
  
不早了，我要睡觉了（2003-12-18 2:02）其他的功能下一篇再说吧，  
哈哈，对了差点忘了  
所有完整代码我都放在压缩文件里了自己看看代码应该对 pe 有个大概的了解,另外送大家一个我在网上收集的 pe 结构图。  
大家在写的时候别忘了#include <winnt.h>  
睡了睡了，，，，困死了!@#$%^&\*()  
886   
                                                                           WiNrOOt  
                                                                       2003-12-18 2:07   
 4dd466d0e1e0cc123afbe504da2462ca