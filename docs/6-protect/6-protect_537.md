

# 翻译：建立自己的 PE Protector （部分 1）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-756.htm`](https://www.pediy.com/kssd/pediy08/pediy8-756.htm)

建立自己的 PE Protector （部分 1）: Your first EXE Protector  
By Ashkbiz Danehkar   
  
原文：http://www.codeproject.com/cpp/peprotector1.asp  
译者：arhat  
时间：2006 年 4 月 18 日  
关键词：PE Protector 调试器 反汇编器 OEP SEH 导入表  
  
本文主要是描述在 Visual C++编译器下，开发属于自己的 PE Protector。

*   [Download source files - 105 Kb ](http://www.codeproject.com/script/profile/whos_who.asp?id=1599836)
*   [Download demo project - 82.5 Kb](http://www.codeproject.com/cpp/peprotector1/peprotector1_src.zip)

![Yoda's Protector v1.01 - Free EXE Protector](http://www.codeproject.com/cpp/peprotector1/peprotector1_demo.zip)

**[序言](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)**

[本文是为那些还不熟悉本领域的人写的。在此，我不准备介绍 PE 的结构，因为 MSDN 中 Matt Pietrek 所写的文章已经解释得非常详细了。如果你不熟悉 PE 结构，我建议你在阅读本文之前先读 Matt Pietrek 的文章。他在文章结尾还提供了许多有用的连接。](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)

[本文将分成三个部分：](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)

1.  **[第一个 EXE Protector](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)**

[这部分介绍用 Visual C++ Win32 Programming 开发一个强大的 EXE protector 和 packer。](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)

2.  **[支持 OCX，DLL，和 SCR](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)**

[这部分介绍除 EXE 文件之外的其它类型的 PE 文件。会介绍一些你应该知道的、保护 OCX 和 DLL 的小技巧。](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)

3.  **[使用 Cryptography API](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)**

[这部分将演示怎样用 Microsoft 的 Cryptography API 函数加密/解密 PE 区段信息。](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)

[本文包含 yoda’s protector 1.01 到 1.03 版本的源码。它在网站上出现 6 个月后，人们才能看到它的源码。它以\[6\] Danilo Bzdok 所写的 yoda’s Crypter 汇编源码，\[7\] Markus F.X.J. Oberhumer & László Molnár 所写的 UPX library 中的压缩源码，\[9\] Joergen Ibsen 所写的 aPLib 压缩源码为基础。因此，非常感谢他们为我们提供了这些工具。我也要感谢那些在不同版本 Windows 上努力测试它的朋友。我想把本文做一个小礼物，送给所有关心爱护 yoda’s protector 的人们。](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)

**[Contents](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)**

*   [Introduction ](http://www.codeproject.com/cpp/peprotector1/peprotector1.gif)
*   [A short aspect about PE Structure ](http://www.codeproject.com/cpp/peprotector1.asp#Introduction0#Introduction0)
*   [Open PE files ](http://www.codeproject.com/cpp/peprotector1.asp#AshortaspectaboutPEStructure1#AshortaspectaboutPEStructure1)
*   [Verify if PE file is valid ](http://www.codeproject.com/cpp/peprotector1.asp#OpenPEfiles2#OpenPEfiles2)
*   [Make Extra Section ](http://www.codeproject.com/cpp/peprotector1.asp#VerifyifPEfileisvalid3#VerifyifPEfileisvalid3)
*   [Pack and Crypt Sections ](http://www.codeproject.com/cpp/peprotector1.asp#MakeExtraSection4#MakeExtraSection4)
*   [Built Import Table Directory ](http://www.codeproject.com/cpp/peprotector1.asp#PackandCryptSections5#PackandCryptSections5)
*   [Reload Import Table and API Redirection ](http://www.codeproject.com/cpp/peprotector1.asp#BuiltImportTableDirectory6#BuiltImportTableDirectory6)
*   [Anti-debug methods ](http://www.codeproject.com/cpp/peprotector1.asp#ReloadImportTableandAPIRedirection7#ReloadImportTableandAPIRedirection7)
*   [Eliminate unnecessary data ](http://www.codeproject.com/cpp/peprotector1.asp#Antidebugmethods8#Antidebugmethods8)
*   [Sample code ](http://www.codeproject.com/cpp/peprotector1.asp#Eliminateunnecessarydata9#Eliminateunnecessarydata9)
*   [Conclusion ](http://www.codeproject.com/cpp/peprotector1.asp#Samplecode10#Samplecode10)
*   [References](http://www.codeproject.com/cpp/peprotector1.asp#Conclusion11#Conclusion11)

**[介绍](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12)**

[PE 是 Microsoft Windows NT ®操作系统下标准的可执行文件格式。它包含代码，数据，资源，DLL（dynamic link libraries）等信息。通过那些强大的调试器（例如 OllyDbg 或 SoftICE），加上一些汇编语言的知识，可以修改它们。但这样做，软件开发公司将会延长产品开发周期，浪费盈利的时间。因此，他们一般购买类似于 EXE protector 之类的工具，从而阻止用户非法拷贝软件。我想，如果每一个人都有自己的 EXE Protector，世界将会怎样。Cracker 将面对不同类型的 EXE protector（这样的日子怎么过呀！）。因此，我想最好是我们每个人都有属于自己的 EXE protector。](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12)

**[PE 结构概述](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12)**

[PE 包括 MS-DOS，Windows NT，和区段信息。Windows 操作系统通过这些信息来分配内存，导入 DLL，执行代码。](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12)

**[Table 1](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12)**

| [MS-DOS information](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) || [IMAGE\_DOS\_HEADER](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) |
| --- | --- |
| [MS-DOS Stub Program](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) |
| [Windows NT information](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) || [PE Signature ("PE")](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) |
| [IMAGE\_FILE\_HEADER](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) |
| [IMAGE\_OPTIONAL\_HEADER](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) |
| [Sections information](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) || [IMAGE\_SECTION\_HEADER\[0\]  
…  
IMAGE\_SECTION\_HEADER\[n\]](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) |
| [SECTION\[0\]  
…  
SECTION\[n\]](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12) |

[你可以\[1\] “Microsoft Portable Executable and Common Object File Format Specification”里找到 PE 文件格式的详细描述。Matt Pietrek 在\[2\] “Peering Inside the PE: A Tour of the Win32 Portable Executable File Format”及\[3a/b\] “An In-Depth Look into the Win32 Portable Executable File Format”里也讲解的十分清楚。](http://www.codeproject.com/cpp/peprotector1.asp#References12#References12)

[此外，Wayne J. Radburn 的 PEView \[4\] 将帮助你看清楚 PE 文件格式的方方面面。](http://www.codeproject.com/cpp/peprotector1.asp#Reference3b#Reference3b)

**[Figure 1](http://www.codeproject.com/cpp/peprotector1.asp#Reference4#Reference4)**

 ![PEView](http://www.codeproject.com/cpp/peprotector1.asp#Reference4#Reference4)

[PE 中所有的数据结构都是在 Visual C++的*winnt.h*中定义的。要与 PE 共事，了解 IMAGE\_DOS\_HEADER, IMAGE\_NT\_HEADERS, IMAGE\_SECTION\_HEADER 就差不多了。这些结构的信息可以在\[5\] MSDN library 中找到。](http://www.codeproject.com/cpp/peprotector1/peview.gif)

**[打开 PE 文件](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)**

[为了使用这些信息，我们必须把 PE 文件格式载入内存。某些 Windows API 函数会帮我们做这些事情：CreateFile(), GetFileSize(), GlobalAlloc(), ReadFile(), CloseHandle()。](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[我为了和 PE 文件共事，专门编写了一个类。它帮助我打开文件，把 DOS 头，NT 头，区段头，和区段放到内存中分开的地方，然后把它们重建成新的 PE 文件。](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[class PEStructure ](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[{](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[private:](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      DWORD ReservedHeaderSize;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      DWORD ReservedHeaderRO;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[public:](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      DWORD                   dwRO\_first\_section;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      IMAGE\_DOS\_HEADER        image\_dos\_header;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      char                    \*reservedheader;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      IMAGE\_NT\_HEADERS        image\_nt\_headers;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      IMAGE\_SECTION\_HEADER    image\_section\_header\[MAX\_SECTION\_NUM\];](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      char                    \*image\_section\[MAX\_SECTION\_NUM\];](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      void OpenFileName(char\* FileName);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      void UpdateHeaders(BOOL bSaveAndValidate);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      void UpdateHeadersSections(BOOL bSaveAndValidate);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      void Free();](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[};](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[OpenFileName() 将打开 PE 文件，放入 image\_dos\_header, reservedheader, image\_nt\_headers, image\_section\_header\[\], 和 image\_section\[\]。在我的工具中，所有的 PE 结构都被这个函数抽象出来。](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[void PEStructure::OpenFileName(char\* FileName)](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[{](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      hFile=CreateFile(FileName,](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[                       GENERIC\_READ,](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[                       FILE\_SHARE\_WRITE | FILE\_SHARE\_READ,](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[                       NULL,OPEN\_EXISTING,](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[                       FILE\_ATTRIBUTE\_NORMAL,NULL);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      if(hFile==INVALID\_HANDLE\_VALUE)](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      {](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[            ShowErr(FileErr);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[            return;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      }](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      dwFsize=GetFileSize(hFile,0);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      if(dwFsize == 0)](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      {](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[            CloseHandle(hFile);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[            ShowErr(FsizeErr);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[            return;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      }](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      dwOutPutSize=](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[          dwFsize+IT\_SIZE+DEPACKER\_CODE\_SIZE+ALIGN\_CORRECTION;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      pMem=(char\*)GlobalAlloc(GMEM\_FIXED | GMEM\_ZEROINIT,](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[                                             dwOutPutSize);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      if(pMem == NULL)](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      {](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[            CloseHandle(hFile);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[            ShowErr(MemErr);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[            return;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      }](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      ReadFile(hFile,pMem,dwFsize,&dwBytesRead,NULL);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      CloseHandle(hFile);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      CopyMemory(&image\_dos\_header,pMem,sizeof(IMAGE\_DOS\_HEADER));](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      ReservedHeaderRO=sizeof(IMAGE\_DOS\_HEADER);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      ReservedHeaderSize=](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[             image\_dos\_header.e\_lfanew-sizeof(IMAGE\_DOS\_HEADER);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      reservedheader=new TCHAR\[ReservedHeaderSize\];](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      CopyMemory(&image\_nt\_headers,](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[                 pMem+image\_dos\_header.e\_lfanew,](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[                 sizeof(IMAGE\_NT\_HEADERS));](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      dwRO\_first\_section = ](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[           image\_dos\_header.e\_lfanew + sizeof(IMAGE\_NT\_HEADERS);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      UpdateHeadersSections(TRUE);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[}](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

**[检验 PE 文件是否正确](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)**

[为了防止不可预知的错误，需要检查 image\_nt\_header 中 image\_dos\_header 和 Signature 的 e\_magic 来检验文件是否是 Win32 PE 文件。](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[if(PEfile.image\_dos\_header.e\_magic!='ZM')](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[{](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      GlobalFree(pMem);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      CloseHandle(hFile);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      if(MakeBackup) DeleteFile(szFnameBackup);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      ShowErr(PEErr);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      return;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[}](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[if(PEfile.image\_nt\_headers.Signature!='EP')](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[{](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      GlobalFree(pMem);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      CloseHandle(hFile);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      if(MakeBackup) DeleteFile(szFnameBackup);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      ShowErr(PEErr);](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[      return;](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[}](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

**[生成额外的区段](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)**

[为了保护和 unpacking 代码，yoda’s Protector 中用了一个小技巧――生成额外的区段。Visual C++帮它做这些事情，不需要用汇编的编译器和连接器。如果你查看*CryptStuff.cpp*中的子例程――PE\_LOADER\_CODE()，你就会发现我正在介绍的内容。Danilo Bzdok\[6\] 的 yoda’s Crypter 也用了这个方法。当然，他只用了汇编语言，而没有用任何其它的高级语言。我从 PE\_LOAD\_CODE() 中抠出了一些代码，编成 GetFunctionRVA(), GetFunctionSize() ,和 CopyFunction()，用它们在目标 PE 文件中生成额外的区段。](http://www.codeproject.com/cpp/peprotector1.asp#Reference5#Reference5)

[DWORD GetFunctionRVA(void\* FuncName)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[{](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      void \*\_tempFuncName=FuncName;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      char \*ptempFuncName=PCHAR(\_tempFuncName);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      DWORD \_jmpdwRVA,dwRVA;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      CopyMemory(&\_jmpdwRVA,ptempFuncName+1,4);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      dwRVA=DWORD(ptempFuncName)+\_jmpdwRVA+5;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      return(dwRVA);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[}](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[DWORD GetFunctionSize(void\* FuncName)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[{](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      DWORD dwRVA=GetFunctionRVA(FuncName);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      char\* pFuncBody=PCHAR(dwRVA);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      UCHAR \_temp;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      bool notEnd=TRUE;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      char \*DepackerCodeEnd=new TCHAR\[10\];](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      DWORD l=0;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      do](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[            CopyMemory(&\_temp,pFuncBody+l,1);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[            if(\_temp==0xC3)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[            {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[                  CopyMemory(DepackerCodeEnd,pFuncBody+l+0x01,10);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[                  DepackerCodeEnd\[9\]=0x00;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[                  if(strcmp(DepackerCodeEnd,"ETGXZKATZ")==0)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[                  {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[                        notEnd=FALSE;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[                  }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[            }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[            l++;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      }while(notEnd);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      return(l);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[}](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[GetFunctionRVA() 将搜索 GetFunctionSize() 与 CopyFunction()使用的具体例程的 RVA。GetFunctionSize() 将返回被 CopyFunction()使用的例程的大小。它通过搜索关键字（“ETGXZKATZ”）来计算例程的大小。最后，CopyFunction() 是从 PE\_LOADER\_CODE()子程序中借用的完整的例程，放置在打包的 PE 文件中。](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[char\* CopyFunction(void\* FuncName)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[{](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    DWORD dwRVA=GetFunctionRVA(FuncName);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    DWORD dwSize=GetFunctionSize(FuncName);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    char\* pFuncBody=PCHAR(dwRVA);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    char\* filebuff=new TCHAR\[dwSize+1\];](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    CopyMemory(filebuff,pFuncBody,dwSize);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    return(filebuff);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[}](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[下面的代码可以说明这个方法：](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[char    \*pDepackerCode;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[DWORD    DEPACKER\_CODE\_SIZE;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[…](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[DEPACKER\_CODE\_SIZE=GetFunctionSize(PE\_LOADER\_CODE);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[pDepackerCode=new TCHAR\[DEPACKER\_CODE\_SIZE\];](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[pDepackerCode=CopyFunction(PE\_LOADER\_CODE);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[…](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[void PE\_LOADER\_CODE()](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[{](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      \_asm](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      *//----------------------------------------------------------*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      *//-------------- START OF THE PE LOADER CODE ---------------*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[DepackerCode:](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      …](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      …](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[      …](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[DepackerCodeEND:](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    RET](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    *//"ETGXZKATZ" <<-- KEY WORD*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    INC EBP     *//'E'*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    PUSH ESP    *//'T'*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    INC EDI     *//'G'*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    POP EAX     *//'X'*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    POP EDX     *//'Z'*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    DEC EBX     *//'K'*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    INC ECX     *//'A'*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    PUSH ESP    *//'T'*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    POP EDX     *//'Z'*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[    }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[}](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

**[打包并加密区段](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)**

[UPX compressor \[7\]的源码是打包 PE 区段的另一选择。我用\[8\] Markus F.X.J. Oberhumer 的 LZO data compression library 来打包代码段和数据段。Danilo Bzdok 的多形加密和解密方法\[6\] 比较简单，把它们略作修改（细节查看*PER.CPP*中的 EncryptBuff() 和 DecryptBuff()），对加密 PE 区段来说足够了。](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[这个 protector 把区段分别放在不同内存单元。然后，通过 CompressPE() 和 CryptPE()打包并加密区段。](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

*[//------ ENCRYPT THE SECTIONS -----](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)*

*[// generate PER](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)*

[PEfile.UpdateHeadersSections(TRUE);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[SecEncryptBuff=new TCHAR\[SEC\_PER\_SIZE\];](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[SecDecryptBuff=new TCHAR\[SEC\_PER\_SIZE\];](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[MakePER(SecEncryptBuff,SecDecryptBuff,SEC\_PER\_SIZE);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[CopyMemory(pDepackerCode+dwRO\_SEC\_DECRYPT,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[           SecDecryptBuff,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[           SEC\_PER\_SIZE);    ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

*[// encrypt !](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)*

[CompressPE(pMem);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[CryptPE(pMem);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[RemoveSectionNames(pMem);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[newsection.Misc.VirtualSize=DepackCodeVirtualSize+0x2000;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[PEfile.image\_section\_header\[](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[        PEfile.image\_nt\_headers.FileHeader.NumberOfSections-1\]](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[        .Misc.VirtualSize = newsection.Misc.VirtualSize;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[PEfile.UpdateHeadersSections(FALSE);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

*[//---------------------------------](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)*

[LZO data compression library \[8\]的 compressor 源码是 C++的，decompressor 源码有 C++和汇编两种。因此，我们打包时可以用高级语言，解包时用低级语言。我在 CompressPE() 内用来自 LZO library 的 lzo1x\_999\_compress\_level() 压缩区段，在\_DecompressPE() 内用 PE\_LOADER\_CODE()的 lzo1f\_decompress\_asm\_fast\_safe()汇编源码解压缩区段。](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[在 Windows98 版本以上，要着重注意 image\_nt\_headers.OptionalHeader.SectionAlignment 和 image\_nt\_headers.OptionalHeader.FileAlignment，防止发生 Win32 不兼容的文件错误。IMAGE\_SECTION\_HEADER 中的变量应当适应文件对齐和区段对齐。我保留了 UpdateHeadersSections() 函数，用它找回并重建所有的 PE 文件格式结构。](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[void PEStructure::UpdateHeadersSections(BOOL bSaveAndValidate)](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[{](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[    DWORD i;](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[    if(bSaveAndValidate)*//TRUE = data is being retrieved*](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[    {](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[        DWORD SectionNum = PEfile.image\_nt\_headers](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                           .FileHeader.NumberOfSections;](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[        CopyMemory(&image\_dos\_header,pMem,sizeof(IMAGE\_DOS\_HEADER));](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[        ReservedHeaderSize = image\_dos\_header.e\_lfanew –](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                             sizeof(IMAGE\_DOS\_HEADER);](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[        if((ReservedHeaderSize&0x80000000)==0x00000000)](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[        {](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[            CopyMemory(reservedheader, ](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                       pMem+ReservedHeaderRO,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                       ReservedHeaderSize);](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[        }](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[        CopyMemory(&image\_nt\_headers,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[             pMem+image\_dos\_header.e\_lfanew,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[             sizeof(IMAGE\_NT\_HEADERS));](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[     dwRO\_first\_section = image\_dos\_header.e\_lfanew +](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                            sizeof(IMAGE\_NT\_HEADERS);](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       CopyMemory(&image\_section\_header, ](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                  pMem+dwRO\_first\_section, ](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                  SectionNum\*sizeof(IMAGE\_SECTION\_HEADER));](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       for(i=0;i< SectionNum;i++)](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       {](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[           image\_section\[i\] = (char\*)GlobalAlloc(](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                GMEM\_FIXED | GMEM\_ZEROINIT,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[            PEAlign(image\_section\_header\[i\].SizeOfRawData,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                PEfile.image\_nt\_headers.OptionalHeader.FileAlignment));](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[           CopyMemory(image\_section\[i\],](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[               pMem + image\_section\_header\[i\].PointerToRawData,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[               image\_section\_header\[i\].SizeOfRawData);](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       }](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[    }](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[   else*//FALSE = data is being initialized *](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[   {](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       DWORD SectionNum = PEfile.image\_nt\_headers](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                          .FileHeader.NumberOfSections;](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       CopyMemory(pMem,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[            &image\_dos\_header,sizeof(IMAGE\_DOS\_HEADER));](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[            ReservedHeaderSize=image\_dos\_header.e\_lfanew –](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                               sizeof(IMAGE\_DOS\_HEADER);](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       if((ReservedHeaderSize&0x80000000)==0x00000000)](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       {](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[          CopyMemory(pMem + ReservedHeaderRO,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                     reservedheader,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                     ReservedHeaderSize);](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       }](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       CopyMemory(pMem+image\_dos\_header.e\_lfanew,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[            &image\_nt\_headers,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[            sizeof(IMAGE\_NT\_HEADERS));](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       dwRO\_first\_section = image\_dos\_header.e\_lfanew +](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                            sizeof(IMAGE\_NT\_HEADERS);](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       CopyMemory(pMem+dwRO\_first\_section,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                 &image\_section\_header,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                 SectionNum\*sizeof(IMAGE\_SECTION\_HEADER));](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       for(i=0;i< SectionNum;i++)](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       {](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[           CopyMemory(pMem+image\_section\_header\[i\].PointerToRawData,](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                      image\_section\[i\],](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[                      image\_section\_header\[i\].SizeOfRawData);](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[       }](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[    }](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

[} ](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

**[建立 Import Table Directory](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)**

[PE 解包区段需要导入两个原始的 API 函数来动态加载需要的 API 函数。*Kernel32.dll *的 LoadLibraryA() 和 GetProcAddress() 是导入其它 API 函数所必需的函数，当然，还需要点小技巧。保留的 AssembleIT() 就是用来完成这个任务的。为了转向额外区段里的新导入表目录，我们应当改变导入表的地址和大小。为了更正老的导入表，必须在下一步里重载并重建导入表目录来完成程序代码。加载器区段使用 LoadLibraryA() 和 GetProcAddress()搜索 DLL 的导入函数的 RVA。](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

**[Table 2](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)**

| [Import Table Address ->](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) || ["*Kernel32.dll*", 0x00](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) |
| --- | --- |
| [ ](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) || [LoadLibrary\_RVA](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) |
| [ ](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) || [GetProcessAddres\_RVA](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) |
| [ ](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) || [0x00,0x00](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) |
| [LoadLibrary\_RVA ->](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) || [0x00,0x00,"LoadLibraryA"](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) |
| [GetProcessAddres\_RVA ->](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) || [0x00,0x00,"GetProcessAddress"](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8) |

**[重载 Import Table 和 API Redirection](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)**

[使导入表远离逆向工程的魔爪很重要。Danilo Bzdok 在\[6\] yoda’s Crypter 里使用的技术包括销毁导入块数据，加密导入信息。加载器代码区段可以再次恢复这个部分。我在我的 PE Protector 中使用了他的方法，部分代码用 C，剩下的部分用汇编。ProcessOrgIT() 为我们提供了实现我们目的所需要的东西。在加载器区段里，INIT\_IMPORT\_TABLE() 例程帮我们实现了 API 重定向及导入表重建工作。](http://www.codeproject.com/cpp/peprotector1.asp#Reference8#Reference8)

**[反调试的方法](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)**

[PE Protector 应该能检测到程序是否被调试，并阻止被调试。OllyDbg 和 SoftICE 是使用最多的两个调试器，它们可以绕过许多阻碍调试器的技巧。尽管如此，我还是要介绍一些检测调试器的简单方法。当然，这些方法对这些使用最新插件的调试器来说，没有任何效果。](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

1.  **[IsDebuggerPresent Windows API: 无论如何，只要当前的进程运行在调试器的上下文中，IsDebuggerPresent() 总是返回非零值。](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)**
2.  **[SoftICE detection:它通过检查 windows NT 下的*NTICE* 驱动及 windows 98 下的*SICE* 驱动是否正在使用来发现 SoftICE 调试器。](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)**

[ 3.            if(CreateFile( "\\\\\\\\.\\\\NTICE", GENERIC\_READ | GENERIC\_WRITE, ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 4.                  FILE\_SHARE\_READ | FILE\_SHARE\_WRITE, ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 5.                  NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 6.                  NULL)!=INVALID\_HANDLE\_VALUE) ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 7.            { ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 8.                There is SoftICE NT on your system; ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 9.            } ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 10.        if(CreateFile( "\\\\\\\\.\\\\SICE", GENERIC\_READ | GENERIC\_WRITE, ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 11.              FILE\_SHARE\_READ | FILE\_SHARE\_WRITE, ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 12.              NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 13.              NULL)!=INVALID\_HANDLE\_VALUE) ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 14.        { ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 15.            There is SoftICE98 on your system; ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 16.        }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

17.  **[Probe Processes: 有时候，它需要搜索具体的进程或隔离具体的进程。在例子源码里，演示了它怎样寻找父进程，检查它是否是*EXPLORER.EXE* ，杀死除了 Explorer 窗口之外的所有父进程。在*CryptStuff.CPP* 里有 C++和汇编两种代码，说明了怎样实现这个任务。](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)**

[ 18.        void GetFileNameFromPath(char\* szSource)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 19.        {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 20.              char \*szTemp=strrchr(szSource,'\\\\');    ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 21.              if(szTemp!=NULL)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 22.              {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 23.                    szTemp++;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 24.                    DWORD l=DWORD(strlen(szTemp))+1;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 25.                    CopyMemory(szSource,szTemp,l);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 26.              }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 27.        }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 28.         ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 29.        void AntiDebug()](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 30.        {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 31.              char lpszSystemInfo\[MAX\_PATH\];](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 32.              HANDLE hSnapshot=NULL;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 33.              DWORD PID\_child;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 34.              DWORD PID\_parent,PID\_explorer;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 35.              HANDLE hh\_parnet = NULL;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 36.              PROCESSENTRY32    pe32 = {0};](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 37.              pe32.dwSize = sizeof(PROCESSENTRY32);*//0x128;*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 38.              PID\_child=GetCurrentProcessId();*//getpid();*](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 39.              hSnapshot = CreateToolhelp32Snapshot(TH32CS\_SNAPPROCESS,0);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 40.              if (Process32First(hSnapshot, &pe32))](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 41.              {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 42.                    while (Process32Next(hSnapshot, &pe32))](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 43.                    {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 44.                          GetFileNameFromPath(pe32.szExeFile);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 45.                          CharUpperBuff(pe32.szExeFile,strlen(pe32.szExeFile));](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 46.                          if(strcmp(pe32.szExeFile,"EXPLORER.EXE")==0)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 47.                          {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 48.                                PID\_explorer=pe32.th32ProcessID;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 49.                          }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 50.                          if(pe32.th32ProcessID==PID\_child)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 51.                          {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 52.                                PID\_parent=pe32.th32ParentProcessID;](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 53.                          }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 54.                    }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 55.              }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 56.              if(PID\_parent!=PID\_explorer)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 57.              {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 58.                    hh\_parnet= OpenProcess(PROCESS\_ALL\_ACCESS, ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 59.                                            TRUE, PID\_parent);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 60.                    TerminateProcess(hh\_parnet, 0);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 61.              }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 62.              else](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 63.              {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 64.                    MODULEENTRY32    me32 = {0};](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 65.                    me32.dwSize = sizeof(MODULEENTRY32);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 66.                    hSnapshot = CreateToolhelp32Snapshot(TH32CS\_SNAPMODULE,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 67.                                PID\_explorer);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 68.                    if (Module32First(hSnapshot, &me32))](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 69.                    {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 70.                          do](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 71.                          {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 72.                                if(PID\_explorer==me32.th32ProcessID)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 73.                                {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 74.                                      GetWindowsDirectory(lpszSystemInfo,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 75.                                       MAX\_PATH+1);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 76.                                      strcat(lpszSystemInfo,"\\\\");](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 77.                                      strcat(lpszSystemInfo,"EXPLORER.EXE");](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 78.                              CharUpperBuff(me32.szExePath,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 79.                                            strlen(me32.szExePath));](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 80.                                      if(strncmp(me32.szExePath,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 81.                                                  lpszSystemInfo,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 82.                                            strlen(lpszSystemInfo)))](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 83.                                      {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 84.                                         GetFileNameFromPath(me32.szExePath);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 85.                                         if(strcmp(me32.szExePath,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 86.                                            "EXPLORER.EXE")==0)](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 87.                                         {](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 88.                                            hh\_parnet= ](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 89.                                                OpenProcess(PROCESS\_ALL\_ACCESS,](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 90.                                                  TRUE, PID\_explorer);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 91.                                            TerminateProcess(hh\_parnet, 0);](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 92.                                         }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 93.                                      }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 94.                                }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 95.                          }while (Module32Next(hSnapshot, &me32));](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 96.                    }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 97.              }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[ 98.        }](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[因为缺少 CreateToolhelp32Snapshot(), Process32First(), Process32Next(), Module32First(), Module32Next()，这段代码在 Windows NT 4.0 下并不能使用。但这些 API 函数可以用来自*NTDLL.DLL*的未公开的 API 函数来构建。你可以阅读 Gary Nebbett 的\[10\] ‘Windows NT (2000) Native API reference’。](http://www.codeproject.com/cpp/peprotector1.asp#Reference6#Reference6)

[此外，我推荐你用 Mark Russinovich 的\[11\] Process Explorer 探究你系统中正在运行的进程。它可以帮助你更好的理解它们。](http://www.codeproject.com/cpp/peprotector1.asp#Reference10#Reference10)

**[Figure 2](http://www.codeproject.com/cpp/peprotector1.asp#Reference11#Reference11)**

 ![Process Explorer](http://www.codeproject.com/cpp/peprotector1.asp#Reference11#Reference11)

**[删除不必要的数据](http://www.codeproject.com/cpp/peprotector1/procexp.gif)**

[有时候，你可能会想着删除所有不必要的数据，例如调试信息，重定位区段，减小 DOS 头，删除 MS-DOS stub Program 等。但基于一些考虑，你应该在你的 PE Protector 中保留这些数据。例如，重定位区段在运行 EXE 文件中没有任何作用，但它在 OLE-Active Controls 和 DLL 中扮演着重要的角色。](http://www.codeproject.com/cpp/peprotector1/procexp.gif)

**[例子代码](http://www.codeproject.com/cpp/peprotector1/procexp.gif)**

[这个项目可以在 Visual C++ .NET 2003 下编译，而不需要任何其它的工具。除了 Windows NT 4.0 和 Windows 95 外，它可以在任何版本的 Windows 系统下工作。](http://www.codeproject.com/cpp/peprotector1/procexp.gif)

**[结论](http://www.codeproject.com/cpp/peprotector1/procexp.gif)**

[本文及它的源码演示了 PE protector 工具的工作原理，可以作为它们的入门读物。我希望它覆盖了开源领域有关此主题所缺少的内容。](http://www.codeproject.com/cpp/peprotector1/procexp.gif)

**[参考资料](http://www.codeproject.com/cpp/peprotector1/procexp.gif)**

[\[1\] "Microsoft Portable Executable and Common Object File Format Specification", Microsoft Corporation, Revision 6.0, February 1999.](http://www.codeproject.com/cpp/peprotector1/procexp.gif)

[\[2\] " Peering Inside the PE: A Tour of the Win32 Portable Executable File Format", Matt Pietrek, MSDN Library, March 1994.](http://www.microsoft.com/whdc/system/platform/firmware/PECOFF.mspx)

[\[3a\] "An In-Depth Look into the Win32 Portable Executable File Format", part 1, Matt Pietrek, MSDN Magazine, February 2002.](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dnwbgen/html/msdn_peeringpe.asp)

[\[3b\] "An In-Depth Look into the Win32 Portable Executable File Format", part 2, Matt Pietrek, MSDN Magazine, March 2002.](http://msdn.microsoft.com/msdnmag/issues/02/02/PE/default.aspx)

[\[4\] PEview Version 0.67, Wayne J. Radburn.](http://msdn.microsoft.com/msdnmag/issues/02/02/PE/default.aspx)

[\[5\] MSDN Library, Microsoft Corporation, April 2003.](http://www.magma.ca/~wjr/)

[\[6\] yoda’s Crypter, Danilo Bzdok.](http://msdn.microsoft.com/library/)

[\[7\] UPX, the Ultimate Packer for eXecutables, Markus F.X.J. Oberhumer & László Molnár.](http://y0da.cjb.net/)

[\[8\] LZO real-time data compression library, Markus F.X.J. Oberhumer.](http://upx.sourceforge.net/)

[\[9\] aPLib compression library, Joergen Ibsen.](http://www.oberhumer.com/opensource/lzo/)

[\[10\] "Windows NT (2000) Native API reference", Gary Nebbet.](http://www.ibsensoftware.com/)

[\[11\] Process Explorer, Mark Russinovich.](http://www.amazon.com/exec/obidos/ASIN/1578701996/)

**[Ashkbiz Danehkar](http://www.sysinternals.com/)**

| [ \[if gte vml 1\]><v:shape id="\_x0000\_i1028" type="#\_x0000\_t75" alt="" style='width:1.5pt;height:1.5pt'> <v:imagedata src="建立自已的 PE%20Protector（部分 1）.files/image004.jpg" o:href="http://www.codeproject.com/script/profile/images/%7bBABD49A1-25D7-4303-ADE7-800FC6561D3D%7d.jpg"/> </v:shape><!\[endif\] ![](img/e805b2caebfdd70e14e4eaa5a61d3a0d.jpg)](http://www.sysinternals.com/) || [Click here to view Ashkbiz Danehkar's online profile.](http://www.codeproject.com/script/profile/images/%7bBABD49A1-25D7-4303-ADE7-800FC6561D3D%7d.jpg) |
| --- | --- |

[ ](http://www.codeproject.com/script/profile/whos_who.asp?vt=arts&id=1599836)

| **[Other popular articles:](http://www.codeproject.com/script/profile/whos_who.asp?vt=arts&id=1599836)**

*   [Generating Fractals with SSE/SSE2  
    An article on generating Mandelbrot and Julia sets using Intel's Streaming SIMD Extensions (SSE, SSE2). ](http://www.codeproject.com/script/profile/whos_who.asp?vt=arts&id=1599836)
*   [Yet Another Generalized Functors Implementation in C++  
    An article on generalized functors implementation in C++. Generalized functor requirements, existing implementation problems and disadvantages are considered. Several new ideas and problem solutions together with the compete implementation are suggested. ](http://www.codeproject.com/cpp/fractalssse.asp)
*   [How a C++ compiler implements exception handling  
    An indepth discussion of how VC++ implements exception handling. Source code includes exception handling library for VC++. ](http://www.codeproject.com/cpp/genfunctors.asp)
*   [Scaling of memory intensive multi-threaded applications to SMMP computers  
    This article discusses impact of the multithreaded run-time library's memory manager on scaling memory-intensive server applications to Shared Memory Multiprocessor computers.](http://www.codeproject.com/cpp/exceptionhandler.asp)

 |
| --- |

 4dd466d0e1e0cc123afbe504da2462ca