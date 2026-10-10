

# 利用 python+pefile 库做 PE 格式文件的快速开发

> 原文：[`www.pediy.com/kssd/pediy10/89838.html`](https://www.pediy.com/kssd/pediy10/89838.html)

发现很多的朋友经常用到 PE 格式相关的开发，如解析 PE 文件的格式，获取相关的内容。比如常常用到的静态的病毒启发式检测模型的建立、病毒样本分类、查壳脱壳等。  
搜索了一下发现论坛里面没有我要讲的这个东西，于是我在这里向大家推荐 pefile 这个 python 库。  
  
这个是基于 MIT licence 的一个开源项目，你可以在上面做更多的开发。  
开发包的下载地址  
[`code.google.com/p/pefile/`](http://code.google.com/p/pefile/)  
  
我觉得有以下几点大家可以注意：  
1. 这个需要使用 python 语言开发，优点是敏捷开发，方便快捷，而且源代码可读，易懂，当然肯定不会用于商业的，作为学习研究非常方便。  
2. 由于基于 PE 的结构 pefile 已经做了非常充分的解析，所以对于我们做二次开发非常方便。各种关键的数据结构能够非常容易的获得。  
3. 由于 python 的编写的快速、低门槛。另外 pefile 已经做了很多的功能，这个 pefile 模块非常适合需要快速达到目的和一些需要入门的朋友。  
4. 免费的开源项目  
  
话不多说，直接教大家使用，看完后，方可知道 pefile 的强大。  
  

代码:

```
1. 当然是要安装 python 开发包。
2. 下载 pefile 到本地，解压，新建一个文件 petest.py

```

首先实验一  
  
代码:

```
实验一
import os, string, shutil,re
import pefile ##记得 import pefile

PEfile\_Path = r"C:\\temp\\test.exe"

pe = pefile.PE(PEfile\_Path)
print PEfile\_Path
print pe

```

代码:

```
实验一结果
C:\\temp\\test.exe
\----------DOS\_HEADER----------

\[IMAGE\_DOS\_HEADER\]
e\_magic:                       0x5A4D    
e\_cblp:                        0x90      
e\_cp:                          0x3       
e\_crlc:                        0x0       
e\_cparhdr:                     0x4       
e\_minalloc:                    0x0       
e\_maxalloc:                    0xFFFF    
e\_ss:                          0x0       
e\_sp:                          0xB8      
e\_csum:                        0x0       
e\_ip:                          0x0       
e\_cs:                          0x0       
e\_lfarlc:                      0x40      
e\_ovno:                        0x0       
e\_res:                         
e\_oemid:                       0x0       
e\_oeminfo:                     0x0       
e\_res2:                        
e\_lfanew:                      0xD0      

\----------NT\_HEADERS----------

\[IMAGE\_NT\_HEADERS\]
Signature:                     0x4550    

\----------FILE\_HEADER----------

\[IMAGE\_FILE\_HEADER\]
Machine:                       0x14C     
NumberOfSections:              0x2       
TimeDateStamp:                 0x46A8C07C \[Thu Jul 26 15:40:44 2007 UTC\]
PointerToSymbolTable:          0x0       
NumberOfSymbols:               0x0       
SizeOfOptionalHeader:          0xE0      
Characteristics:               0x10F     
Flags: IMAGE\_FILE\_LOCAL\_SYMS\_STRIPPED, IMAGE\_FILE\_32BIT\_MACHINE, IMAGE\_FILE\_EXECUTABLE\_IMAGE, IMAGE\_FILE\_LINE\_NUMS\_STRIPPED, IMAGE\_FILE\_RELOCS\_STRIPPED

\----------OPTIONAL\_HEADER----------

\[IMAGE\_OPTIONAL\_HEADER\]
Magic:                         0x10B     
MajorLinkerVersion:            0x6       
MinorLinkerVersion:            0x0       
SizeOfCode:                    0x420     
SizeOfInitializedData:         0x130     
SizeOfUninitializedData:       0x0       
AddressOfEntryPoint:           0x522     
BaseOfCode:                    0x220     
BaseOfData:                    0x640     
ImageBase:                     0x400000  
SectionAlignment:              0x10      
FileAlignment:                 0x10      
MajorOperatingSystemVersion:   0x4       
MinorOperatingSystemVersion:   0x0       
MajorImageVersion:             0x0       
MinorImageVersion:             0x0       
MajorSubsystemVersion:         0x4       
MinorSubsystemVersion:         0x0       
Reserved1:                     0x0       
SizeOfImage:                   0x768     
SizeOfHeaders:                 0x420     
CheckSum:                      0x0       
Subsystem:                     0x2       
DllCharacteristics:            0x0       
SizeOfStackReserve:            0x100000  
SizeOfStackCommit:             0x1000    
SizeOfHeapReserve:             0x100000  
SizeOfHeapCommit:              0x1000    
LoaderFlags:                   0x0       
NumberOfRvaAndSizes:           0x10      
DllCharacteristics: 

\----------PE Sections----------

\[IMAGE\_SECTION\_HEADER\]
Name:                          .text
Misc:                          0x418     
Misc\_PhysicalAddress:          0x418     
Misc\_VirtualSize:              0x418     
VirtualAddress:                0x220     
SizeOfRawData:                 0x420     
PointerToRawData:              0x420     
PointerToRelocations:          0x0       
PointerToLinenumbers:          0x0       
NumberOfRelocations:           0x0       
NumberOfLinenumbers:           0x0       
Characteristics:               0x60000020
Flags: IMAGE\_SCN\_CNT\_CODE, IMAGE\_SCN\_MEM\_EXECUTE, IMAGE\_SCN\_MEM\_READ
Entropy: 6.385628 (Min=0.0, Max=8.0)
MD5     hash: 37ae973124ba5655ce156536f4018759
SHA-1   hash: 6354d772105b66ac33fb8950b76a289edafa230f
SHA-256 hash: f6dfe337c6c6278e60a687552d8fc3be2a2ed41a4278713cfd0dc631296befdc
SHA-512 hash: 9d22cdd011d7276f47e3b1844804d58be2e73eef826ad285769d449f03dbfcde743303b31a9172e513be571432b7b2080afe571e5819ec7968acd76c0d82207a

\[IMAGE\_SECTION\_HEADER\]
Name:                          .rsrc
Misc:                          0x128     
Misc\_PhysicalAddress:          0x128     
Misc\_VirtualSize:              0x128     
VirtualAddress:                0x640     
SizeOfRawData:                 0x130     
PointerToRawData:              0x840     
PointerToRelocations:          0x0       
PointerToLinenumbers:          0x0       
NumberOfRelocations:           0x0       
NumberOfLinenumbers:           0x0       
Characteristics:               0x40000040
Flags: IMAGE\_SCN\_CNT\_INITIALIZED\_DATA, IMAGE\_SCN\_MEM\_READ
Entropy: 2.905524 (Min=0.0, Max=8.0)
MD5     hash: cfd4f1a98445485c616ea2ff9390278e
SHA-1   hash: 7480ffe5427a540e17353df9c490dbba86fd0c3b
SHA-256 hash: 93f9ad56e464614b6aa9521f2b80f3f7f2fd5e2b6d8d6fd6489a0b1cdb1f948e
SHA-512 hash: b054ba77825a4bb92d9beecb606d04f7a4bf4d16529d909e03e6b882175e23fb495c1c3dc9d921c3124210a6567bf68e70879d3163ece1a1cbb786f3ec94af43

\----------Directories----------

\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\]
VirtualAddress:                0x574     
Size:                          0x3C      
\[IMAGE\_DIRECTORY\_ENTRY\_RESOURCE\]
VirtualAddress:                0x640     
Size:                          0x128     
\[IMAGE\_DIRECTORY\_ENTRY\_EXCEPTION\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_SECURITY\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_DEBUG\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_COPYRIGHT\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_TLS\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_IAT\]
VirtualAddress:                0x220     
Size:                          0x1C      
\[IMAGE\_DIRECTORY\_ENTRY\_DELAY\_IMPORT\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_COM\_DESCRIPTOR\]
VirtualAddress:                0x0       
Size:                          0x0       
\[IMAGE\_DIRECTORY\_ENTRY\_RESERVED\]
VirtualAddress:                0x0       
Size:                          0x0       

\----------Imported symbols----------

\[IMAGE\_IMPORT\_DESCRIPTOR\]
OriginalFirstThunk:            0x5B0     
Characteristics:               0x5B0     
TimeDateStamp:                 0x0        \[Thu Jan 01 00:00:00 1970 UTC\]
ForwarderChain:                0x0       
Name:                          0x5E0     
FirstThunk:                    0x220     

KERNEL32.dll.GetModuleHandleA Hint\[294\]

\[IMAGE\_IMPORT\_DESCRIPTOR\]
OriginalFirstThunk:            0x5B8     
Characteristics:               0x5B8     
TimeDateStamp:                 0x0        \[Thu Jan 01 00:00:00 1970 UTC\]
ForwarderChain:                0x0       
Name:                          0x62C     
FirstThunk:                    0x228     

USER32.dll.EndDialog Hint\[185\]
USER32.dll.GetDlgItemTextA Hint\[260\]
USER32.dll.DialogBoxParamA Hint\[147\]
USER32.dll.MessageBoxA Hint\[446\]

\----------Resource directory----------

\[IMAGE\_RESOURCE\_DIRECTORY\]
Characteristics:               0x0       
TimeDateStamp:                 0x0        \[Thu Jan 01 00:00:00 1970 UTC\]
MajorVersion:                  0x0       
MinorVersion:                  0x0       
NumberOfNamedEntries:          0x0       
NumberOfIdEntries:             0x1       
  Id: \[0x5\] (RT\_DIALOG)
  \[IMAGE\_RESOURCE\_DIRECTORY\_ENTRY\]
  Name:                          0x5       
  OffsetToData:                  0x80000018
    \[IMAGE\_RESOURCE\_DIRECTORY\]
    Characteristics:               0x0       
    TimeDateStamp:                 0x0        \[Thu Jan 01 00:00:00 1970 UTC\]
    MajorVersion:                  0x0       
    MinorVersion:                  0x0       
    NumberOfNamedEntries:          0x0       
    NumberOfIdEntries:             0x1       
      Id: \[0x65\]
      \[IMAGE\_RESOURCE\_DIRECTORY\_ENTRY\]
      Name:                          0x65      
      OffsetToData:                  0x80000030
        \[IMAGE\_RESOURCE\_DIRECTORY\]
        Characteristics:               0x0       
        TimeDateStamp:                 0x0        \[Thu Jan 01 00:00:00 1970 UTC\]
        MajorVersion:                  0x0       
        MinorVersion:                  0x0       
        NumberOfNamedEntries:          0x0       
        NumberOfIdEntries:             0x1       
          \[IMAGE\_RESOURCE\_DIRECTORY\_ENTRY\]
          Name:                          0x804     
          OffsetToData:                  0x48      
            \[IMAGE\_RESOURCE\_DATA\_ENTRY\]
            OffsetToData:                  0x6A0     
            Size:                          0xC8      
            CodePage:                      0x0       
            Reserved:                      0x0       

```

实验一只是做了简简单单的 print，但是可以看出 pefile 对 test.exe 做了全面的解析从 DOS\_Header 到 OPTIONAL\_HEADER 再到 PE SECTIONS。每个结构都可以完全的取得。细心的朋友还可以发现，他甚至可以做对一个 section header 的 hash 运算，包括 md5, sha1, sha-256, sha-512，对导入导出函数也做了列举。  
当然大家会问，未必我们就直接一个 print 就行了，然后做字符串解析，匹配来获得我们想要的信息？那 pefile 肯定不至于那么愚昧，当然要提供更多的接口。比如得到 entrypoint  
代码:

```
print hex(pe.OPTIONAL\_HEADER.AddressOfEntryPoint)

```

实验二  
代码:

```
实验二-节表
import os, string, shutil,re
import pefile ##记得 import pefile

PEfile\_Path = r"C:\\temp\\test.exe"

pe = pefile.PE(PEfile\_Path)
print PEfile\_Path

for section in pe.sections:
    print section

```

代码:

```
实验二结果
C:\\temp\\test.exe
\[IMAGE\_SECTION\_HEADER\]
Name:                          .text
Misc:                          0x418     
Misc\_PhysicalAddress:          0x418     
Misc\_VirtualSize:              0x418     
VirtualAddress:                0x220     
SizeOfRawData:                 0x420     
PointerToRawData:              0x420     
PointerToRelocations:          0x0       
PointerToLinenumbers:          0x0       
NumberOfRelocations:           0x0       
NumberOfLinenumbers:           0x0       
Characteristics:               0x60000020
\[IMAGE\_SECTION\_HEADER\]
Name:                          .rsrc
Misc:                          0x128     
Misc\_PhysicalAddress:          0x128     
Misc\_VirtualSize:              0x128     
VirtualAddress:                0x640     
SizeOfRawData:                 0x130     
PointerToRawData:              0x840     
PointerToRelocations:          0x0       
PointerToLinenumbers:          0x0       
NumberOfRelocations:           0x0       
NumberOfLinenumbers:           0x0       
Characteristics:               0x40000040

```

可以看出此文件有 2 个节.text 和 .rsrc，并且给出了节的相关信息。当然如果你需要获得某一节的具体的某个信息如 Characteristics，可以采用  
代码:

```
print hex(pe.sections\[i\].Characteristics)

```

实验三  
代码:

```
实验三-导入表
import os, string, shutil,re
import pefile ##记得 import pefile

PEfile\_Path = r"C:\\temp\\test.exe"

pe = pefile.PE(PEfile\_Path)
print PEfile\_Path

for importeddll in pe.DIRECTORY\_ENTRY\_IMPORT:
    print importeddll.dll
    ##or use
    #print pe.DIRECTORY\_ENTRY\_IMPORT\[0\].dll
    for importedapi in importeddll.imports:
        print importedapi.name
    ##or use
    #print pe.DIRECTORY\_ENTRY\_IMPORT\[0\].imports\[0\].name

```

代码:

```
实验三-结果
C:\\temp\\test.exe
KERNEL32.dll
GetModuleHandleA
USER32.dll
EndDialog
GetDlgItemTextA
DialogBoxParamA
MessageBoxA

```

实验三得出 test.exe 导入了 kernel32.dll 和 user32.dll 然后分别导入了 1 个和 4 个 API 函数。  
  
关于 pefile 的使用和他的强大功能想必大家也是有所体会，他还有很多的其他功能，比如修改 PE 结构，另外导入 PEiD 的特征库就可以支持查壳等等。大家可以试着用一下。  
  
希望这个 pefile 和强大功能和 python 的简单易用能帮助到大家。