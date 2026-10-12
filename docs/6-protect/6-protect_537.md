
# 翻译：建立自己的 PE Protector （部分 1）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-756.htm`](https://www.pediy.com/kssd/pediy08/pediy8-756.htm)

建立自己的 PE Protector （部分 1）: Your first EXE Protector

By Ashkbiz Danehkar

原文：`http://www.codeproject.com/cpp/peprotector1.asp`

译者：`arhat`

时间：2006 年 4 月 18 日

关键词：PE Protector 调试器 反汇编器 OEP SEH 导入表

本文主要是描述在 `Visual C++` 编译器下，开发属于自己的 PE Protector。

*   [Download source files - 105 Kb ](`http://www.codeproject.com/script/profile/whos_who.asp?id=1599836`)
*   [Download demo project - 82.5 Kb](`http://www.codeproject.com/cpp/peprotector1/peprotector1_src.zip`)

![Yoda's Protector v1.01 - Free EXE Protector](`http://www.codeproject.com/cpp/peprotector1/peprotector1_demo.zip`)

## 序言

本文是为那些还不熟悉本领域的人写的。在此，我不准备介绍 PE 的结构，因为 MSDN 中 `Matt Pietrek` 所写的文章已经解释得非常详细了。如果你不熟悉 PE 结构，我建议你在阅读本文之前先读 `Matt Pietrek` 的文章。他在文章结尾还提供了许多有用的连接。

本文将分成三个部分：

1.  **第一个 EXE Protector**
    这部分介绍用 `Visual C++ Win32 Programming` 开发一个强大的 EXE protector 和 packer。
2.  **支持 OCX，DLL，和 SCR**
    这部分介绍除 EXE 文件之外的其它类型的 PE 文件。会介绍一些你应该知道的、保护 OCX 和 DLL 的小技巧。
3.  **使用 Cryptography API**
    这部分将演示怎样用 Microsoft 的 `Cryptography API` 函数加密/解密 PE 区段信息。

本文包含 `yoda's protector` 1.01 到 1.03 版本的源码。它在网站上出现 6 个月后，人们才能看到它的源码。它以 \[6\] `Danilo Bzdok` 所写的 `yoda's Crypter` 汇编源码，\[7\] `Markus F.X.J. Oberhumer` & `László Molnár` 所写的 `UPX library` 中的压缩源码，\[9\] `Joergen Ibsen` 所写的 `aPLib` 压缩源码为基础。因此，非常感谢他们为我们提供了这些工具。我也要感谢那些在不同版本 `Windows` 上努力测试它的朋友。我想把本文做一个小礼物，送给所有关心爱护 `yoda's protector` 的人们。

## Contents

*   [Introduction ](`http://www.codeproject.com/cpp/peprotector1/peprotector1.gif`)
*   [A short aspect about PE Structure ](`http://www.codeproject.com/cpp/peprotector1.asp#Introduction0#Introduction0`)
*   [Open PE files ](`http://www.codeproject.com/cpp/peprotector1.asp#AshortaspectaboutPEStructure1#AshortaspectaboutPEStructure1`)
*   [Verify if PE file is valid ](`http://www.codeproject.com/cpp/peprotector1.asp#OpenPEfiles2#OpenPEfiles2`)
*   [Make Extra Section ](`http://www.codeproject.com/cpp/peprotector1.asp#VerifyifPEfileisvalid3#VerifyifPEfileisvalid3`)
*   [Pack and Crypt Sections ](`http://www.codeproject.com/cpp/peprotector1.asp#MakeExtraSection4#MakeExtraSection4`)
*   [Built Import Table Directory ](`http://www.codeproject.com/cpp/peprotector1.asp#PackandCryptSections5#PackandCryptSections5`)
*   [Reload Import Table and API Redirection ](`http://www.codeproject.com/cpp/peprotector1.asp#BuiltImportTableDirectory6#BuiltImportTableDirectory6`)
*   [Anti-debug methods ](`http://www.codeproject.com/cpp/peprotector1.asp#ReloadImportTableandAPIRedirection7#ReloadImportTableandAPIRedirection7`)
*   [Eliminate unnecessary data ](`http://www.codeproject.com/cpp/peprotector1.asp#Antidebugmethods8#Antidebugmethods8`)
*   [Sample code ](`http://www.codeproject.com/cpp/peprotector1.asp#Eliminateunnecessarydata9#Eliminateunnecessarydata9`)
*   [Conclusion ](`http://www.codeproject.com/cpp/peprotector1.asp#Samplecode10#Samplecode10`)
*   [References](`http://www.codeproject.com/cpp/peprotector1.asp#Conclusion11#Conclusion11`)

## 介绍

PE 是 `Microsoft Windows NT` 操作系统下标准的可执行文件格式。它包含代码，数据，资源，`DLL`（dynamic link libraries）等信息。通过那些强大的调试器（例如 `OllyDbg` 或 `SoftICE`），加上一些汇编语言的知识，可以修改它们。但这样做，软件开发公司将会延长产品开发周期，浪费盈利的时间。因此，他们一般购买类似于 EXE protector 之类的工具，从而阻止用户非法拷贝软件。我想，如果每一个人都有自己的 EXE Protector，世界将会怎样。`Cracker` 将面对不同类型的 EXE protector（这样的日子怎么过呀！）。因此，我想最好是我们每个人都有属于自己的 EXE protector。

## PE 结构概述

PE 包括 MS-DOS，Windows NT，和区段信息。Windows 操作系统通过这些信息来分配内存，导入 DLL，执行代码。

### Table 1

| [MS-DOS information](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) || [`IMAGE_DOS_HEADER`](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) |

| --- | --- |

| [MS-DOS Stub Program](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) |

| [Windows NT information](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) || [`PE Signature ("PE")`](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) |

| [`IMAGE_FILE_HEADER`](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) |

| [`IMAGE_OPTIONAL_HEADER`](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) |

| [Sections information](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) || [`IMAGE_SECTION_HEADER[0] ... IMAGE_SECTION_HEADER[n]`](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) |

| [`SECTION[0] ... SECTION[n]`](`http://www.codeproject.com/cpp/peprotector1.asp#References12#References12`) |

你可以 \[1\] “`Microsoft Portable Executable and Common Object File Format Specification`”里找到 PE 文件格式的详细描述。`Matt Pietrek` 在 \[2\] “`Peering Inside the PE: A Tour of the Win32 Portable Executable File Format`”及 \[3a/b\] “`An In-Depth Look into the Win32 Portable Executable File Format`”里也讲解的十分清楚。

此外，`Wayne J. Radburn` 的 `PEView` \[4\] 将帮助你看清楚 PE 文件格式的方方面面。

### Figure 1

![`PEView`](`http://www.codeproject.com/cpp/peprotector1.asp#Reference4#Reference4`)

PE 中所有的数据结构都是在 `Visual C++` 的 `winnt.h` 中定义的。要与 PE 共事，了解 `IMAGE_DOS_HEADER`, `IMAGE_NT_HEADERS`, `IMAGE_SECTION_HEADER` 就差不多了。这些结构的信息可以在 \[5\] MSDN library 中找到。

## 打开 PE 文件

为了使用这些信息，我们必须把 PE 文件格式载入内存。某些 Windows API 函数会帮我们做这些事情：`CreateFile()`, `GetFileSize()`, `GlobalAlloc()`, `ReadFile()`, `CloseHandle()`。

我为了和 PE 文件共事，专门编写了一个类。它帮助我打开文件，把 DOS 头，NT 头，区段头，和区段放到内存中分开的地方，然后把它们重建成新的 PE 文件。

```cpp
class PEStructure 
{
private:
    DWORD ReservedHeaderSize;
    DWORD ReservedHeaderRO;
public:
    DWORD             dwRO_first_section;
    IMAGE_DOS_HEADER  image_dos_header;
    char             *reservedheader;
    IMAGE_NT_HEADERS  image_nt_headers;
}
```



## 类成员声明

```cpp
IMAGE_SECTION_HEADER image_section_header[MAX_SECTION_NUM];
char* image_section[MAX_SECTION_NUM];
void OpenFileName(char* FileName);
void UpdateHeaders(BOOL bSaveAndValidate);
void UpdateHeadersSections(BOOL bSaveAndValidate);
void Free();
};
```

## `OpenFileName()`

`OpenFileName()` 将打开 PE 文件，放入 `image_dos_header`, `reservedheader`, `image_nt_headers`, `image_section_header[]`, 和 `image_section[]`。在我的工具中，所有的 PE 结构都被这个函数抽象出来。

```cpp
void PEStructure::OpenFileName(char* FileName)
{
    hFile=CreateFile(FileName,
                     GENERIC_READ,
                     FILE_SHARE_WRITE | FILE_SHARE_READ,
                     NULL,OPEN_EXISTING,
                     FILE_ATTRIBUTE_NORMAL,NULL);
    if(hFile==INVALID_HANDLE_VALUE)
    {
        ShowErr(FileErr);
        return;
    }
    dwFsize=GetFileSize(hFile,0);
    if(dwFsize == 0)
    {
        CloseHandle(hFile);
        ShowErr(FsizeErr);
        return;
    }
    dwOutPutSize=
        dwFsize+IT_SIZE+DEPACKER_CODE_SIZE+ALIGN_CORRECTION;
    pMem=(char*)GlobalAlloc(GMEM_FIXED | GMEM_ZEROINIT,
                            dwOutPutSize);
    if(pMem == NULL)
    {
        CloseHandle(hFile);
        ShowErr(MemErr);
        return;
    }
    ReadFile(hFile,pMem,dwFsize,&dwBytesRead,NULL);
    CloseHandle(hFile);
    CopyMemory(&image_dos_header,pMem,sizeof(IMAGE_DOS_HEADER));
    ReservedHeaderRO=sizeof(IMAGE_DOS_HEADER);
    ReservedHeaderSize=
        image_dos_header.e_lfanew-sizeof(IMAGE_DOS_HEADER);
    reservedheader=new TCHAR[ReservedHeaderSize];
    CopyMemory(&image_nt_headers,
               pMem+image_dos_header.e_lfanew,
               sizeof(IMAGE_NT_HEADERS));
    dwRO_first_section = 
      image_dos_header.e_lfanew + sizeof(IMAGE_NT_HEADERS);
    UpdateHeadersSections(TRUE);
}
```

### 检验 PE 文件是否正确

为了防止不可预知的错误，需要检查 `image_nt_header` 中 `image_dos_header` 和 `Signature` 的 `e_magic` 来检验文件是否是 Win32 PE 文件。

```cpp
if(PEfile.image_dos_header.e_magic!='ZM')
{
    GlobalFree(pMem);
    CloseHandle(hFile);
    if(MakeBackup) DeleteFile(szFnameBackup);
    ShowErr(PEErr);
    return;
}
if(PEfile.image_nt_headers.Signature!='EP')
{
    GlobalFree(pMem);
    CloseHandle(hFile);
    if(MakeBackup) DeleteFile(szFnameBackup);
    ShowErr(PEErr);
    return;
}
```

### 生成额外的区段



为了保护和 unpacking 代码，`yoda’s Protector` 中用了一个小技巧――生成额外的区段。`Visual C++` 帮它做这些事情，不需要用汇编的编译器和连接器。如果你查看 `CryptStuff.cpp` 中的子例程――`PE_LOADER_CODE()`，你就会发现我正在介绍的内容。`Danilo Bzdok` 的 `yoda’s Crypter` 也用了这个方法。当然，他只用了汇编语言，而没有用任何其它的高级语言。我从 `PE_LOADER_CODE()` 中抠出了一些代码，编成 `GetFunctionRVA()`、`GetFunctionSize()` 和 `CopyFunction()`，用它们在目标 `PE` 文件中生成额外的区段。

```
DWORD GetFunctionRVA(void* FuncName)
{
     void **_tempFuncName=FuncName;
     char *ptempFuncName=PCHAR(_tempFuncName);
     DWORD _jmpdwRVA,dwRVA;
     CopyMemory(&_jmpdwRVA,ptempFuncName+1,4);
     dwRVA=DWORD(ptempFuncName)+_jmpdwRVA+5;
     return(dwRVA);
}
```

```
DWORD GetFunctionSize(void* FuncName)
{
     DWORD dwRVA=GetFunctionRVA(FuncName);
     char* pFuncBody=PCHAR(dwRVA);
     UCHAR _temp;
     bool notEnd=TRUE;
     char *DepackerCodeEnd=new TCHAR[10];
     DWORD l=0;
     do
     {
         CopyMemory(&_temp,pFuncBody+l,1);
         if(_temp==0xC3)
         {
               CopyMemory(DepackerCodeEnd,pFuncBody+l+0x01,10);
               DepackerCodeEnd[9]=0x00;
               if(strcmp(DepackerCodeEnd,"ETGXZKATZ")==0)
               {
                     notEnd=FALSE;
               }
         }
         l++;
     }while(notEnd);
     return(l);
}
```

`GetFunctionRVA()` 将搜索 `GetFunctionSize()` 与 `CopyFunction()` 使用的具体例程的 `RVA`。`GetFunctionSize()` 将返回被 `CopyFunction()` 使用的例程的大小。它通过搜索关键字（`"ETGXZKATZ"`）来计算例程的大小。最后，`CopyFunction()` 是从 `PE_LOADER_CODE()` 子程序中借用的完整的例程，放置在打包的 `PE` 文件中。

```
char* CopyFunction(void* FuncName)
{
    DWORD dwRVA=GetFunctionRVA(FuncName);
    DWORD dwSize=GetFunctionSize(FuncName);
    char* pFuncBody=PCHAR(dwRVA);
    char* filebuff=new TCHAR[dwSize+1];
    CopyMemory(filebuff,pFuncBody,dwSize);
    return(filebuff);
}
```

下面的代码可以说明这个方法：

```
char    *pDepackerCode;
DWORD    DEPACKER_CODE_SIZE;
…
DEPACKER_CODE_SIZE=GetFunctionSize(PE_LOADER_CODE);
pDepackerCode=new TCHAR[DEPACKER_CODE_SIZE];
pDepackerCode=CopyFunction(PE_LOADER_CODE);
…
```

```
void PE_LOADER_CODE()
{
     _asm
     {
     //----------------------------------------------------------
     //-------------- START OF THE PE LOADER CODE ---------------
     DepackerCode:
     …
     …
     …
     DepackerCodeEND:
    RET
    //"ETGXZKATZ" <<-- KEY WORD
    INC EBP      //'E'
    PUSH ESP     //'T'
    INC EDI      //'G'
    POP EAX      //'X'
    POP EDX      //'Z'
    DEC EBX      //'K'
    INC ECX      //'A'
    PUSH ESP     //'T'
```



### 打包并加密区段

`UPX compressor` [7] 的源码是打包 `PE` 区段的另一选择。我用 `Markus F.X.J. Oberhumer` 的 `LZO data compression library` 来打包代码段和数据段。`Danilo Bzdok` 的多形加密和解密方法 [6] 比较简单，把它们略作修改（细节查看 `PER.CPP` 中的 `EncryptBuff()` 和 `DecryptBuff()`），对加密 `PE` 区段来说足够了。

这个 `protector` 把区段分别放在不同内存单元。然后，通过 `CompressPE()` 和 `CryptPE()` 打包并加密区段。

```
//------ ENCRYPT THE SECTIONS -----
// generate PER
PEfile.UpdateHeadersSections(TRUE);
SecEncryptBuff=new TCHAR[SEC_PER_SIZE];
SecDecryptBuff=new TCHAR[SEC_PER_SIZE];
MakePER(SecEncryptBuff,SecDecryptBuff,SEC_PER_SIZE);
CopyMemory(pDepackerCode+dwRO_SEC_DECRYPT,
           SecDecryptBuff,
           SEC_PER_SIZE);    
// encrypt !
CompressPE(pMem);
CryptPE(pMem);
RemoveSectionNames(pMem);
newsection.Misc.VirtualSize=DepackCodeVirtualSize+0x2000;
PEfile.image_section_header[
       PEfile.image_nt_headers.FileHeader.NumberOfSections-1]
       .Misc.VirtualSize = newsection.Misc.VirtualSize;
PEfile.UpdateHeadersSections(FALSE);
//---------------------------------
```

`LZO data compression library` [8] 的 `compressor` 源码是 `C++` 的，`decompressor` 源码有 `C++` 和汇编两种。因此，我们打包时可以用高级语言，解包时用低级语言。我在 `CompressPE()` 内用来自 `LZO library` 的 `lzo1x_999_compress_level()` 压缩区段，在 `_DecompressPE()` 内用 `PE_LOADER_CODE()` 的 `lzo1f_decompress_asm_fast_safe()` 汇编源码解压缩区段。

在 `Windows98` 版本以上，要着重注意 `image_nt_headers.OptionalHeader.SectionAlignment` 和 `image_nt_headers.OptionalHeader.FileAlignment`，防止发生 `Win32` 不兼容的文件错误。`IMAGE_SECTION_HEADER` 中的变量应当适应文件对齐和区段对齐。我保留了 `UpdateHeadersSections()` 函数，用它找回并重建所有的 `PE` 文件格式结构。

```
void PEStructure::UpdateHeadersSections(BOOL bSaveAndValidate)
{
    DWORD i;
    if(bSaveAndValidate)//TRUE = data is being retrieved
    {
        DWORD SectionNum = PEfile.image_nt_headers
                           .FileHeader.NumberOfSections;
        CopyMemory(&image_dos_header,pMem,sizeof(IMAGE_DOS_HEADER));
        ReservedHeaderSize = image_dos_header.e_lfanew –
                             sizeof(IMAGE_DOS_HEADER);
        if((ReservedHeaderSize&0x80000000)==0x00000000)
        {
            CopyMemory(reservedheader, 
                       pMem+ReservedHeaderRO,
                       ReservedHeaderSize);
        }
        CopyMemory(&image_nt_headers,
           pMem+image_dos_header.e_lfanew,
           sizeof(IMAGE_NT_HEADERS));
     dwRO_first_section = image_dos_header.e_lfanew +
                         sizeof(IMAGE_NT_HEADERS);
       CopyMemory(&image_section_header, 
                 pMem+dwRO_first_section, 
               SectionNum*sizeof(IMAGE_SECTION_HEADER));
       for(i=0;i< SectionNum;i++)
       {
         image_section[i] = (char*)GlobalAlloc(
        GMEM_FIXED | GMEM_ZEROINIT,
    PEAlign(image_section_header[i].SizeOfRawData,
        PEfile.image_nt_headers.OptionalHeader.FileAlignment));
         CopyMemory(image_section[i],
               pMem + image_section_header[i].PointerToRawData,
               image_section_header[i].SizeOfRawData);
       }
    }
   else//FALSE = data is being initialized 
   {
       DWORD SectionNum = PEfile.image_nt_headers
```



```cpp
FileHeader.NumberOfSections;

CopyMemory(pMem,
    &image_dos_header, sizeof(IMAGE_DOS_HEADER));
ReservedHeaderSize = image_dos_header.e_lfanew –
                    sizeof(IMAGE_DOS_HEADER);
if ((ReservedHeaderSize & 0x80000000) == 0x00000000)
{
    CopyMemory(pMem + ReservedHeaderRO,
            reservedheader,
            ReservedHeaderSize);
}
CopyMemory(pMem + image_dos_header.e_lfanew,
    &image_nt_headers,
    sizeof(IMAGE_NT_HEADERS));
dwRO_first_section = image_dos_header.e_lfanew +
                    sizeof(IMAGE_NT_HEADERS);
CopyMemory(pMem + dwRO_first_section,
            &image_section_header,
            SectionNum * sizeof(IMAGE_SECTION_HEADER));
for (i = 0; i < SectionNum; i++)
{
    CopyMemory(pMem + image_section_header[i].PointerToRawData,
             image_section[i],
             image_section_header[i].SizeOfRawData);
}
}
}
```

### 建立 Import Table Directory

PE 解包区段需要导入两个原始的 API 函数来动态加载需要的 API 函数。`Kernel32.dll` 的 `LoadLibraryA()` 和 `GetProcAddress()` 是导入其它 API 函数所必需的函数，当然，还需要点小技巧。保留的 `AssembleIT()` 就是用来完成这个任务的。为了转向额外区段里的新导入表目录，我们应当改变导入表的地址和大小。为了更正老的导入表，必须在下一步里重载并重建导入表目录来完成程序代码。加载器区段使用 `LoadLibraryA()` 和 `GetProcAddress()` 搜索 `DLL` 的导入函数的 `RVA`。

**Table 2**

| Import Table Address -> |  | `"Kernel32.dll", 0x00` |
| :--- | :--- | :--- |
|  |  | `LoadLibrary_RVA` |
|  |  | `GetProcessAddres_RVA` |
|  |  | `0x00, 0x00` |
| `LoadLibrary_RVA` -> |  | `0x00, 0x00, "LoadLibraryA"` |
| `GetProcessAddres_RVA` -> |  | `0x00, 0x00, "GetProcessAddress"` |

### 重载 Import Table 和 API Redirection

使导入表远离逆向工程的魔爪很重要。Danilo Bzdok 在 [6] yoda’s Crypter 里使用的技术包括销毁导入块数据，加密导入信息。加载器代码区段可以再次恢复这个部分。我在我的 `PE Protector` 中使用了他的方法，部分代码用 C，剩下的部分用汇编。`ProcessOrgIT()` 为我们提供了实现我们目的所需要的东西。在加载器区段里，`INIT_IMPORT_TABLE()` 例程帮我们实现了 API 重定向及导入表重建工作。

### 反调试的方法

`PE Protector` 应该能检测到程序是否被调试，并阻止被调试。`OllyDbg` 和 `SoftICE` 是使用最多的两个调试器，它们可以绕过许多阻碍调试器的技巧。尽管如此，我还是要介绍一些检测调试器的简单方法。当然，这些方法对这些使用最新插件的调试器来说，没有任何效果。

1.  **`IsDebuggerPresent` Windows API:** 无论如何，只要当前的进程运行在调试器的上下文中，`IsDebuggerPresent()` 总是返回非零值。

2.  **`SoftICE` detection:** 它通过检查 windows NT 下的 `NTICE` 驱动及 windows 98 下的 `SICE` 驱动是否正在使用来发现 `SoftICE` 调试器。

```cpp
if (CreateFile( "\\\\.\\NTICE", GENERIC_READ | GENERIC_WRITE,
       FILE_SHARE_READ | FILE_SHARE_WRITE,
       NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL,
       NULL) != INVALID_HANDLE_VALUE)
{
    There is SoftICE NT on your system;
}
if (CreateFile( "\\\\.\\SICE", GENERIC_READ | GENERIC_WRITE,
       FILE_SHARE_READ | FILE_SHARE_WRITE,
       NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL,
       NULL) != INVALID_HANDLE_VALUE)
{
    There is SoftICE98 on your system;
}
```

3.  **Probe Processes:** 有时候，它需要搜索具体的进程或隔离具体的进程。在例子源码里，演示了它怎样寻找父进程，检查它是否是 `EXPLORER.EXE`，杀死除了 `Explorer` 窗口之外的所有父进程。在 `CryptStuff.CPP` 里有 C++ 和汇编两种代码，说明了怎样实现这个任务。

```cpp
void GetFileNameFromPath(char* szSource)
```



```c
char *szTemp=strrchr(szSource,'\\');
if(szTemp!=NULL)
{
    szTemp++;
    DWORD l=DWORD(strlen(szTemp))+1;
    CopyMemory(szSource,szTemp,l);
}

void AntiDebug()
{
    char lpszSystemInfo[MAX_PATH];
    HANDLE hSnapshot=NULL;
    DWORD PID_child;
    DWORD PID_parent,PID_explorer;
    HANDLE hh_parnet = NULL;
    PROCESSENTRY32    pe32 = {0};
    pe32.dwSize = sizeof(PROCESSENTRY32);//0x128;
    PID_child=GetCurrentProcessId();//getpid();
    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,0);
    if (Process32First(hSnapshot, &pe32))
    {
        while (Process32Next(hSnapshot, &pe32))
        {
            GetFileNameFromPath(pe32.szExeFile);
            CharUpperBuff(pe32.szExeFile,strlen(pe32.szExeFile));
            if(strcmp(pe32.szExeFile,"EXPLORER.EXE")==0)
            {
                PID_explorer=pe32.th32ProcessID;
            }
            if(pe32.th32ProcessID==PID_child)
            {
                PID_parent=pe32.th32ParentProcessID;
            }
        }
    }
    if(PID_parent!=PID_explorer)
    {
        hh_parnet= OpenProcess(PROCESS_ALL_ACCESS, TRUE, PID_parent);
        TerminateProcess(hh_parnet, 0);
    }
    else
    {
        MODULEENTRY32    me32 = {0};
        me32.dwSize = sizeof(MODULEENTRY32);
        hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, PID_explorer);
        if (Module32First(hSnapshot, &me32))
        {
            do
            {
                if(PID_explorer==me32.th32ProcessID)
                {
                    GetWindowsDirectory(lpszSystemInfo, MAX_PATH+1);
                    strcat(lpszSystemInfo,"\\");
                    strcat(lpszSystemInfo,"EXPLORER.EXE");
                    CharUpperBuff(me32.szExePath, strlen(me32.szExePath));
                    if(strncmp(me32.szExePath, lpszSystemInfo, strlen(lpszSystemInfo)))
                    {
```



```
84.        GetFileNameFromPath(me32.szExePath);
85.        if(strcmp(me32.szExePath,
86.            "EXPLORER.EXE")==0)
87.        {
88.            hh_pernet = 
89.                OpenProcess(PROCESS_ALL_ACCESS,
90.                    TRUE, PID_explorer);
91.            TerminateProcess(hh_pernet, 0);
92.        }
93.      }
94.    }
95.  }while (Module32Next(hSnapshot, &me32));
96. }
97. }
98. }
```

因为缺少 `CreateToolhelp32Snapshot()`, `Process32First()`, `Process32Next()`, `Module32First()`, `Module32Next()`，这段代码在 Windows NT 4.0 下并不能使用。但这些 API 函数可以用来自 `NTDLL.DLL` 的未公开的 API 函数来构建。你可以阅读 Gary Nebbett 的[10] ‘Windows NT (2000) Native API reference’。

此外，我推荐你用 Mark Russinovich 的[11] `Process Explorer` 探究你系统中正在运行的进程。它可以帮助你更好的理解它们。

### Figure 2

![Process Explorer](http://www.codeproject.com/cpp/peprotector1/procexp.gif)

### 删除不必要的数据

有时候，你可能会想着删除所有不必要的数据，例如调试信息，重定位区段，减小 DOS 头，删除 MS-DOS stub Program 等。但基于一些考虑，你应该在你的 `PE Protector` 中保留这些数据。例如，重定位区段在运行 `EXE` 文件中没有任何作用，但它在 `OLE-Active Controls` 和 `DLL` 中扮演着重要的角色。

### 例子代码

这个项目可以在 `Visual C++ .NET 2003` 下编译，而不需要任何其它的工具。除了 `Windows NT 4.0` 和 `Windows 95` 外，它可以在任何版本的 `Windows` 系统下工作。

### 结论

本文及它的源码演示了 `PE protector` 工具的工作原理，可以作为它们的入门读物。我希望它覆盖了开源领域有关此主题所缺少的内容。

### 参考资料

[1] "Microsoft Portable Executable and Common Object File Format Specification", Microsoft Corporation, Revision 6.0, February 1999.

[2] " Peering Inside the PE: A Tour of the Win32 Portable Executable File Format", Matt Pietrek, MSDN Library, March 1994.

[3a] "An In-Depth Look into the Win32 Portable Executable File Format", part 1, Matt Pietrek, MSDN Magazine, February 2002.

[3b] "An In-Depth Look into the Win32 Portable Executable File Format", part 2, Matt Pietrek, MSDN Magazine, March 2002.

[4] `PEview` Version 0.67, Wayne J. Radburn.

[5] `MSDN Library`, Microsoft Corporation, April 2003.

[6] `yoda’s Crypter`, Danilo Bzdok.

[7] `UPX`, the Ultimate Packer for eXecutables, Markus F.X.J. Oberhumer & László Molnár.

[8] `LZO` real-time data compression library, Markus F.X.J. Oberhumer.

[9] `aPLib` compression library, Joergen Ibsen.

[10] "Windows NT (2000) Native API reference", Gary Nebbet.

[11] `Process Explorer`, Mark Russinovich.

**Ashkbiz Danehkar**

[Click here to view Ashkbiz Danehkar's online profile.](http://www.sysinternals.com/)

**Other popular articles:**

*   Generating Fractals with SSE/SSE2  
    An article on generating Mandelbrot and Julia sets using Intel's Streaming SIMD Extensions (SSE, SSE2).
*   Yet Another Generalized Functors Implementation in C++  
    An article on generalized functors implementation in C++. Generalized functor requirements, existing implementation problems and disadvantages are considered. Several new ideas and problem solutions together with the compete implementation are suggested.
*   How a C++ compiler implements exception handling  
    An indepth discussion of how VC++ implements exception handling. Source code includes exception handling library for VC++.
*   Scaling of memory intensive multi-threaded applications to SMMP computers  
    This article discusses impact of the multithreaded run-time library's memory manager on scaling memory-intensive server applications to Shared Memory Multiprocessor computers.

4dd466d0e1e0cc123afbe504da2462ca
