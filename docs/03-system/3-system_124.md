
# 菜鸟注释 PEMaker6 源代码（一）

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-648.htm`](https://www.pediy.com/kssd/pediy09/pediy09-648.htm)

> 这两天看了精华 8 的“向导入表中注入代码“这篇文章，对阅读 `PEMaker6` 源代码进行了部分注释，要不总是忘了哪一部分是什么功能，为了增加点发帖量，将注释过的几个类逐一整理后发布，老鸟就无需看了,也是为了自己加强学习。

> 这篇介绍 `CPELibrary` 类的功能：找回并重建 PE 文件（具体可参照加壳技术->加壳软件的编写->向 PE 中注入代码)

> 类定义：

```cpp
class CPELibrary 
{ 
private: 
  //----------------------------------------- 
  PCHAR          pMem; 
  DWORD          dwFileSize; 
  //----------------------------------------- 
protected: 
  //----------------------------------------- 
  PIMAGE_DOS_HEADER    image_dos_header; 
  PCHAR          pDosStub; 
  DWORD          dwDosStubSize, dwDosStubOffset; 
  PIMAGE_NT_HEADERS    image_nt_headers;  //指针 
  PIMAGE_SECTION_HEADER  image_section_header[MAX_SECTION_NUM]; 
  PCHAR          image_section[MAX_SECTION_NUM]; 
  /*typedef struct _IMAGE_TLS_DIRECTORY32 { 
    DWORD   StartAddressOfRawData; 
    DWORD   EndAddressOfRawData; 
    PDWORD  AddressOfIndex; 
    PIMAGE_TLS_CALLBACK *AddressOfCallBacks; 
    DWORD   SizeOfZeroFill; 
    DWORD   Characteristics; 
  } IMAGE_TLS_DIRECTORY32*/ 
  PIMAGE_TLS_DIRECTORY32  image_tls_directory; 
  //----------------------------------------- 
protected: 
  //----------------------------------------- 
  DWORD PEAlign(DWORD dwTarNum,DWORD dwAlignTo); 
  void AlignmentSections(); 
  //----------------------------------------- 
  DWORD Offset2RVA(DWORD dwRO); 
  DWORD RVA2Offset(DWORD dwRVA); 
  //----------------------------------------- 
  PIMAGE_SECTION_HEADER ImageRVA2Section(DWORD dwRVA); 
  PIMAGE_SECTION_HEADER ImageOffset2Section(DWORD dwRO); 
  //----------------------------------------- 
  DWORD ImageOffset2SectionNum(DWORD dwRVA); 
  PIMAGE_SECTION_HEADER AddNewSection(char* szName,DWORD dwSize); 
  //----------------------------------------- 
public: 
  //----------------------------------------- 
  CPELibrary(); 
  ~CPELibrary(); 
  //----------------------------------------- 
  void OpenFile(char* FileName); 
  void SaveFile(char* FileName);   
  //----------------------------------------- 
}; 
```

> CPP 文件：

```cpp
CPELibrary::CPELibrary() 
{    //构造 DOS 文件头 
    image_dos_header=new (IMAGE_DOS_HEADER); 
    dwDosStubSize=0; 
    //构造 NT 文件头 
    image_nt_headers=new (IMAGE_NT_HEADERS); 
    //构造 20 个节表 
    for(int i=0;i<MAX_SECTION_NUM;i++) image_section_header[i]=new (IMAGE_SECTION_HEADER); 
    image_tls_directory=NULL; 
} 
//---------------------------------------------------------------- 
CPELibrary::~CPELibrary() 
{ 
    delete []image_dos_header; 
    dwDosStubSize=0; 
    delete []image_nt_headers; 
    for(int i=0;i<MAX_SECTION_NUM;i++) delete []image_section_header[i]; 
    if(image_tls_directory!=NULL) delete image_tls_directory; 
} 
//================================================================ 
//---------------------------------------------------------------- 
// returns aligned value 
//对齐 
DWORD CPELibrary::PEAlign(DWORD dwTarNum,DWORD dwAlignTo) 
{     
    return(((dwTarNum+dwAlignTo-1)/dwAlignTo)*dwAlignTo); 
} 
//---------------------------------------------------------------- 
void CPELibrary::AlignmentSections() 
{    //根据节的数量循环 
    for(int i=0;i<image_nt_headers->FileHeader.NumberOfSections;i++) 
    { 
        //节中数据的 RVA。 
        image_section_header[i]->VirtualAddress= 
            PEAlign(image_section_header[i]->VirtualAddress, 
            image_nt_headers->OptionalHeader.SectionAlignment); 
        //对齐 Misc.VirtualSize 
        image_section_header[i]->Misc.VirtualSize= 
            PEAlign(image_section_header[i]->Misc.VirtualSize, 
            image_nt_headers->OptionalHeader.SectionAlignment); 
        //对齐 PointerToRawData 
        image_section_header[i]->PointerToRawData= 
            PEAlign(image_section_header[i]->PointerToRawData, 
            image_nt_headers->OptionalHeader.FileAlignment); 
        //对齐 SizeOfRawData 
        image_section_header[i]->SizeOfRawData= 
            PEAlign(image_section_header[i]->SizeOfRawData, 
            image_nt_headers->OptionalHeader.FileAlignment); 
    } 
    //是个 RVA，重建映象文件基址 
    image_nt_headers->OptionalHeader.SizeOfImage=image_section_header[i-1]->VirtualAddress+ 
        image_section_header[i-1]->Misc.VirtualSize; 
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG].VirtualAddress=0; 
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG].Size=0; 
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].VirtualAddress=0; 
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].Size=0; 
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IAT].VirtualAddress=0; 
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IAT].Size=0; 
} 
//================================================================ 
//---------------------------------------------------------------- 
// calulates the Offset from a RVA 
// Base    - base of the MMF 
// dwRVA - the RVA to calculate 
// returns 0 if an error occurred else the calculated Offset will be returned 
DWORD CPELibrary::RVA2Offset(DWORD dwRVA) 
{ 
    DWORD _offset; 
    PIMAGE_SECTION_HEADER section; 
    section=ImageRVA2Section(dwRVA);//ImageRvaToSection(pimage_nt_headers,Base,dwRVA); 
    if(section==NULL) 
    { 
        return(0); 
    } 
    //文件偏移 
    _offset=(dwRVA-section->VirtualAddress)+section->PointerToRawData; 
    return(_offset); 
} 
//---------------------------------------------------------------- 
// calulates the RVA from a Offset 
// Base    - base of the MMF 
// dwRO - the Offset to calculate 
// returns 0 if an error occurred else the calculated Offset will be returned 
DWORD CPELibrary::Offset2RVA(DWORD dwRO) 
{ 
    PIMAGE_SECTION_HEADER section; 
    section=ImageOffset2Section(dwRO); 
    if(section==NULL) 
    { 
        return(0); 
    } 
    //返回 RVA 
    return((dwRO-section->PointerToRawData)+section->VirtualAddress); 
} 
//================================================================ 
//---------------------------------------------------------------- 
PIMAGE_SECTION_HEADER CPELibrary::ImageRVA2Section(DWORD dwRVA) 
{ 
    int i; 
    for(i=0;i<image_nt_headers->FileHeader.NumberOfSections;i++) 
    {    //确定包含在哪一个节中 
        if((dwRVA>=image_section_header[i]->VirtualAddress) && (dwRVA<=(image_section_header[i]->VirtualAddress+image_section_header[i]->SizeOfRawData))) 
        {    //返回该节 
            return ((PIMAGE_SECTION_HEADER)image_section_header[i]); 
        } 
    } 
    return(NULL); 
} 

//---------------------------------------------------------------- 
//The ImageOffset2Section function locates a Off Set address (RO) 
//within the image header of a file that is mapped as a file and 
//returns a pointer to the section table entry for that virtual 
//address. 
PIMAGE_SECTION_HEADER CPELibrary::ImageOffset2Section(DWORD dwRO) 
{ 
    for(int i=0;i<image_nt_headers->FileHeader.NumberOfSections;i++) 
    { 
        //确定包含在哪一个节中 
} 
```



```cpp
if((dwRO>=image_section_header[i]->PointerToRawData) && (dwRO<(image_section_header[i]->PointerToRawData+image_section_header[i]->SizeOfRawData)))
{    //返回该节
    return ((PIMAGE_SECTION_HEADER)image_section_header[i]);
}
}
return(NULL);
}
//================================================================
//----------------------------------------------------------------
// retrieve Enrty Point Section Number
// Base    - base of the MMF
// dwRVA - the RVA to calculate
// returns -1 if an error occurred else the calculated Offset will be returned
//由文件偏移到节号
DWORD CPELibrary::ImageOffset2SectionNum(DWORD dwRO)
{
    for(int i=0;i<image_nt_headers->FileHeader.NumberOfSections;i++)
    {
        if((dwRO>=image_section_header[i]->PointerToRawData) && (dwRO<(image_section_header[i]->PointerToRawData+image_section_header[i]->SizeOfRawData)))
        {
            return (i);
        }
    }
    return(-1);
}

//增加新节----------------------------------------------------------------
PIMAGE_SECTION_HEADER CPELibrary::AddNewSection(char* szName,DWORD dwSize)
{
    DWORD roffset,rsize,voffset,vsize;
    //最后一节
    int i=image_nt_headers->FileHeader.NumberOfSections;
    rsize=PEAlign(dwSize,
                  image_nt_headers->OptionalHeader.FileAlignment);
    vsize=PEAlign(rsize,
                  image_nt_headers->OptionalHeader.SectionAlignment);
    roffset=PEAlign(image_section_header[i-1]->PointerToRawData+image_section_header[i-1]->SizeOfRawData,
                  image_nt_headers->OptionalHeader.FileAlignment);
    voffset=PEAlign(image_section_header[i-1]->VirtualAddress+image_section_header[i-1]->Misc.VirtualSize,
                  image_nt_headers->OptionalHeader.SectionAlignment);
    //初始化新节表
    memset(image_section_header[i],0,(size_t)sizeof(IMAGE_SECTION_HEADER));
    //通过计算上一节得到
    image_section_header[i]->PointerToRawData=roffset;
    image_section_header[i]->VirtualAddress=voffset;
    //通过传递的参数得到
    image_section_header[i]->SizeOfRawData=rsize;
    image_section_header[i]->Misc.VirtualSize=vsize;
    //属性
    image_section_header[i]->Characteristics=0xC0000040;
    //新节名称
    memcpy(image_section_header[i]->Name,szName,(size_t)strlen(szName));
    //申请一个节的空间
    image_section[i]=(char*)GlobalAlloc(GMEM_FIXED | GMEM_ZEROINIT,rsize);
    //更新文件头
    image_nt_headers->FileHeader.NumberOfSections++;
    //返回新节
    return (PIMAGE_SECTION_HEADER)image_section_header[i];
}

//================================================================
//----------------------------------------------------------------
void CPELibrary::OpenFile(char* FileName)
{
    DWORD    dwBytesRead        = 0;
    HANDLE    hFile= NULL;
    DWORD SectionNum;
    DWORD i;
    DWORD dwRO_first_section;
    //基类成员
    pMem=NULL;
    //----------------------------------------
    hFile=CreateFile(FileName,
                    GENERIC_READ,
                    FILE_SHARE_WRITE | FILE_SHARE_READ,
                    NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
    if(hFile==INVALID_HANDLE_VALUE)
    {
        ShowErr(FileErr);
        return;
    }
    dwFileSize=GetFileSize(hFile,0);
    if(dwFileSize == 0)
    {
        CloseHandle(hFile);
        ShowErr(FsizeErr);
        return;
    }
    //不用内存映射文件,奇怪？
    pMem=(char*)GlobalAlloc(GMEM_FIXED | GMEM_ZEROINIT,dwFileSize);
    if(pMem == NULL)
    {
        CloseHandle(hFile);
        ShowErr(MemErr);
        return;
    }
    //读入文件
    ReadFile(hFile,pMem,dwFileSize,&dwBytesRead,NULL);
    //关闭句柄
    CloseHandle(hFile);
    //----------------------------------------
    //读出 DOS 文件头
    memcpy(image_dos_header,pMem,sizeof(IMAGE_DOS_HEADER));
    //MS-DOS Stub Program(位于 e_lfanew 和 Signature 之间)
    dwDosStubSize=image_dos_header->e_lfanew-sizeof(IMAGE_DOS_HEADER);
    dwDosStubOffset=sizeof(IMAGE_DOS_HEADER);
    pDosStub=new CHAR[dwDosStubSize];
    if((dwDosStubSize&0x80000000)==0x00000000)
    {
        CopyMemory(pDosStub,pMem+dwDosStubOffset,dwDosStubSize);
    }
    //读出 NT 文件头
    memcpy(image_nt_headers,
           pMem+image_dos_header->e_lfanew,
           sizeof(IMAGE_NT_HEADERS));
    //DOS 头大小+NT 头大小
    dwRO_first_section=image_dos_header->e_lfanew+sizeof(IMAGE_NT_HEADERS);
    if(image_dos_header->e_magic!=IMAGE_DOS_SIGNATURE)// MZ
    {
        ShowErr(PEErr);
        GlobalFree(pMem);
        return;
    }
    if(image_nt_headers->Signature!=IMAGE_NT_SIGNATURE)// PE00
    {
        ShowErr(PEErr);
        GlobalFree(pMem);
        return;
    }
    //----------------------------------------
    //节的数量
    SectionNum=image_nt_headers->FileHeader.NumberOfSections;
    //----------------------------------------
    for( i=0;i<SectionNum;i++)  
    {
        //读取节表，储存节表的空间已经初始化了。
        CopyMemory(image_section_header[i],pMem+dwRO_first_section+i*sizeof(IMAGE_SECTION_HEADER),
            sizeof(IMAGE_SECTION_HEADER));
    }
    //----------------------------------------
    for(i=0;i<SectionNum;i++)
    {
            //为每个节数据分配空间，大小为每个节在文件中的大小
            image_section[i]=(char*)GlobalAlloc(GMEM_FIXED | GMEM_ZEROINIT,
                PEAlign(image_section_header[i]->SizeOfRawData,
                image_nt_headers->OptionalHeader.FileAlignment));
            //循环读出每个节，按照文件对齐
            CopyMemory(image_section[i],
                    pMem+image_section_header[i]->PointerToRawData,
                    image_section_header[i]->SizeOfRawData);
    }
    //是否存在 TLS 节----------------------------------------
    if(image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress!=0)
    {
        //分配空间
        image_tls_directory = new (IMAGE_TLS_DIRECTORY32);
        //文件偏移
        DWORD dwOffset=RVA2Offset(image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress);
        //复制
        memcpy(image_tls_directory,
                pMem+dwOffset,
                sizeof(IMAGE_TLS_DIRECTORY32));        
    }
    //----------------------------------------
    GlobalFree(pMem);
}

//----------------------------------------------------------------
void CPELibrary::SaveFile(char* FileName)
{
    DWORD    dwBytesWritten    = 0;
    DWORD i;
    DWORD dwRO_first_section;
    DWORD SectionNum;
    HANDLE    hFile= NULL;
    pMem=NULL;
    //----------------------------------------
    hFile=CreateFile(FileName,
                    GENERIC_WRITE,
                    FILE_SHARE_WRITE | FILE_SHARE_READ,
                    NULL,CREATE_NEW,FILE_ATTRIBUTE_NORMAL,NULL);
    if(hFile==INVALID_HANDLE_VALUE)
    {
        hFile=CreateFile(FileName,
                    GENERIC_WRITE,
                    FILE_SHARE_WRITE | FILE_SHARE_READ,
                    NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
        if(hFile==INVALID_HANDLE_VALUE)
        {
```



```cpp
            ShowErr(FileErr);  
            return;  
        }  
    }  
    //对齐----------------------------------------  
    AlignmentSections();  
    //----------------------------------------  
    i=image_nt_headers->FileHeader.NumberOfSections;  
    //得到新文件的大小  
    dwFileSize=image_section_header[i-1]->PointerToRawData+  
        image_section_header[i-1]->SizeOfRawData;  
    //申请内存  
    pMem=(char*)GlobalAlloc(GMEM_FIXED | GMEM_ZEROINIT,dwFileSize);  
    if(pMem == NULL)  
    {  
        CloseHandle(hFile);  
        ShowErr(MemErr);  
        return;  
    }  
    //复制 DOS 头----------------------------------------  
    memcpy(pMem,image_dos_header,sizeof(IMAGE_DOS_HEADER));  
    //MS-DOS Stub Program  
    if((dwDosStubSize&0x80000000)==0x00000000)  
    {  
        memcpy(pMem+dwDosStubOffset,pDosStub,dwDosStubSize);  
    }  
    //复制 NT 头  
    memcpy(pMem+image_dos_header->e_lfanew,  
        image_nt_headers,  
        sizeof(IMAGE_NT_HEADERS));  
    //节表的偏移  
    dwRO_first_section=image_dos_header->e_lfanew+sizeof(IMAGE_NT_HEADERS);  
    //节的数量  
    SectionNum=image_nt_headers->FileHeader.NumberOfSections;  
    //复制节表----------------------------------------  
    for( i=0;i<SectionNum;i++)  
    {  
        CopyMemory(pMem+dwRO_first_section+i*sizeof(IMAGE_SECTION_HEADER),  
            image_section_header[i],  
            sizeof(IMAGE_SECTION_HEADER));  
    }  
    //复制节数据----------------------------------------  
    for(i=0;i<SectionNum;i++)  
    {  
        CopyMemory(pMem+image_section_header[i]->PointerToRawData,  
                image_section[i],  
                image_section_header[i]->SizeOfRawData);  
    }  
    // ----- WRITE FILE MEMORY TO DISK -----  
    //设置文件指针  
    SetFilePointer(hFile,0,NULL,FILE_BEGIN);  
    //写内存到磁盘  
    WriteFile(hFile,pMem,dwFileSize,&dwBytesWritten,NULL);  
      
    // ------ FORCE CALCULATED FILE SIZE ------  
    //强制计算文件大小  
    SetFilePointer(hFile,dwFileSize,NULL,FILE_BEGIN);  
    SetEndOfFile(hFile);  
    CloseHandle(hFile);  
    //----------------------------------------  
    GlobalFree(pMem);  
} 
```

今天注释的是 `CITMaker` 类，其中应该有不少的错误，请大家附加自己的思考吧

这个类的主要作用是根据下面的字符串计算所需的导入表大小以及构建导入表

`PEMaker6` 下载地址：`http://www.mydatabus.com/public/panzhibin/pemaker6.zip`

```cpp
static const char *sz_IT_EXE_strings[]=
{
    "Kernel32.dll",  
    "LoadLibraryA",  
    "GetProcAddress",  
    0,  
    0,  
};  

const char *sz_IT_OCX_strings[]=
{
    "Kernel32.dll",  
    "LoadLibraryA",  
    "GetProcAddress",  
    "GetModuleHandleA",  
    0,  
    "User32.dll",  
    "GetKeyboardType",  
    "WindowFromPoint",  
    0,  
    "AdvApi32.dll",  
    "RegQueryValueExA",  
    "RegSetValueExA",  
    "StartServiceA",  
    0,  
    "Oleaut32.dll",  
    "SysFreeString",  
    "CreateErrorInfo",  
    "SafeArrayPtrOfIndex",  
    0,  
    "Gdi32.dll",  
    "UnrealizeObject",  
    0,  
    "Ole32.dll",  
    "CreateStreamOnHGlobal",  
    "IsEqualGUID",  
    0,  
    "ComCtl32.dll",  
    "ImageList_SetIconSize",  
    0,  
    0,  
};  
//----------------------------------------------------------------  
using namespace std;  
//----------------------------------------------------------------  
typedef struct
{
    CHAR szFunction[32];  
}t_IMAGE_THUNK, *pt_IMAGE_THUNK;  
//----------------------------------------------------------------  
typedef struct
{
    CHAR szLibrary[32];  
    //链表
    list <t_IMAGE_THUNK> ThunksList;  
    //链表指针
    list <t_IMAGE_THUNK>::iterator ThunkIter;  
    CHAR szFunction[32];  
}t_IMAGE_IMPORT_TABLE, *pt_IMAGE_IMPORT_TABLE;  
//----------------------------------------------------------------  
static list <t_IMAGE_IMPORT_TABLE> ImportTable;  
static list <t_IMAGE_IMPORT_TABLE>::iterator ImportIter;  
//================================================================  
CITMaker::CITMaker(int iType)  
{
    //初始化
    Initialization(iType);  
    //计算出所需的导入表大小
    dwSize=Get_IT_Size();  
    //申请空间
    pMem=new CHAR[dwSize];  
}  

CITMaker::~CITMaker()  
{
    ImportTable.clear();  
    delete [] pMem;  
}  
//----------------------------------------------------------------  
// This function makes the dll name strings, saves them to the linked list  
void CITMaker::Initialization(int iType)  
{
    int i;  
    PCHAR *sz_IT_strings;  
    t_IMAGE_IMPORT_TABLE    imageimport;  
    t_IMAGE_THUNK            imagethunk;  
    switch(iType)  
    {
    case IMPORT_TABLE_EXE:  
        sz_IT_strings=(PCHAR *)sz_IT_EXE_strings;  
        break;  
    case IMPORT_TABLE_OCX:  
        sz_IT_strings=(PCHAR *)sz_IT_OCX_strings;  
        break;  
    }  
    //静态全局变量--------------------------------------------  
    ImportTable.clear();  
    i=0;  
    do
    {
        //复制库名
        strcpy(imageimport.szLibrary,sz_IT_strings[i]);  
        //清空 imageimport--Thunk 列表
        imageimport.ThunksList.clear();  
        do
        {
            
            i++;  
            //取 API 函数名
            if(sz_IT_strings[i]!=0)  
            {
                //向 imagethunk---复制函数名
                strcpy(imagethunk.szFunction,sz_IT_strings[i]);  
                //放入链表
                imageimport.ThunksList.push_back(imagethunk);  
            }  
        }while(sz_IT_strings[i]!=0);  //循环取得函数名
        ImportTable.push_back(imageimport);    //入 ImportTable 链表
        i++;  
    }
    while(sz_IT_strings[i]!=0);  
    //其实 EXE 文件只有一个表被建立--------------------------------------------  
}  
//----------------------------------------------------------------  
// This function calculated zise of Import Table.  
//计算导入表大小
//引入表实际上是一个 IMAGE_IMPORT_DESCRIPTOR 结构数组。
//每个结构包含 PE 文件引入函数的一个相关 DLL 的信息
//比如，如果该 PE 文件从 10 个不同的 DLL 中引入函数，那么这个数组就有 10 个成员。
//该数组以一个全 0 的成员结尾。
//这里引入另一篇文章的自建导入表的结构，借鉴一下更直观，不一定准确：

/*align 4
    v_ImportA  dd Ker_API-v_ImportA              ;IMAGE_THUNK_DATA 数组指针
    v_TimeDateA dd 0                             ;文件建立时间
    v_ForChainA dd 0                             ;0
    v_DllNameA  dd KerName-v_ImportA              ;DLL 名称
    v_FThunkA   dd vGetprocAddress-v_ImportA      ;IMAGE_THUNK_DATA 数组指针

    v_ImportB  dd Use_API-v_ImportA              ;IMAGE_THUNK_DATA 数组指针
    v_TimeDateB dd 0                             ;文件建立时间
    v_ForChainB dd 0                             ;0
    v_DllNameB  dd UserName-v_ImportA             ;DLL 名称
    v_FThunkB   dd vMessageBoxA-v_ImportA        ;IMAGE_THUNK_DATA 数组指针

               dd 20 dup (0)                      ;引入表结束符

    KerName db 'KERNEL32.DLL',0
    Ker_API dd KAPI_A-v_ImportA
           dd KAPI_B-v_ImportA
           dd KAPI_C-v_ImportA
           dd KAPI_D-v_ImportA
           dd KAPI_E-v_ImportA
           dd KAPI_F-v_ImportA
           dd KAPI_G-v_ImportA
           dd KAPI_H-v_ImportA
           dd KAPI_I-v_ImportA
           dd KAPI_J-v_ImportA
           dd KAPI_K-v_ImportA
           dd KAPI_L-v_ImportA
           dd KAPI_M-v_ImportA
           dd KAPI_N-v_ImportA
           dd KAPI_O-v_ImportA
           dd KAPI_P-v_ImportA
           dd KAPI_Q-v_ImportA
           dd 0


    UserName db 'USER32.DLL',0
    Use_API dd UAPI_A-v_ImportA
           dd 0

    vGetprocAddress     dd 0
```



```
vGetModuleHandleA   dd 0
vLoadLibraryA       dd 0
vExitprocess        dd 0
vCreateFileA        dd 0
vCreateFileMappingA dd 0
vGetTempPathA       dd 0
vGetTempFileNameA   dd 0
vlstrlen            dd 0
vMapViewOfFile      dd 0
vWriteFile          dd 0
vUnmapViewOfFile    dd 0
vCloseHandle        dd 0
vCopyFileA          dd 0
vGetModuleFileNameA dd 0
vDeleteFileA        dd 0
vWinExec            dd 0

vMessageBoxA        dd 0
                    dd 0

KAPI_A db 0,0,'GetprocAddress',0
KAPI_B db 0,0,'GetModuleHandleA',0
KAPI_C db 0,0,'LoadLibraryA',0
KAPI_D db 0,0,'Exitprocess',0
KAPI_E db 0,0,'CreateFileA',0
KAPI_F db 0,0,'CreateFileMappingA',0
KAPI_G db 0,0,'GetTempPathA',0
KAPI_H db 0,0,'GetTempFileNameA',0
KAPI_I db 0,0,'lstrlen',0
KAPI_J db 0,0,'MapViewOfFile',0
KAPI_K db 0,0,'WriteFile',0
KAPI_L db 0,0,'UnmapViewOfFile',0
KAPI_M db 0,0,'CloseHandle',0
KAPI_N db 0,0,'CopyFileA',0
KAPI_O db 0,0,'GetModuleFileNameA',0
KAPI_P db 0,0,'DeleteFileA',0
KAPI_Q db 0,0,'WinExec',0

UAPI_A db 0,0,'MessageBoxA',0

; 根据引入表的格式，自建的引入表。
;------------------------------------(上面的)--
align 4
*/
DWORD CITMaker::Get_IT_Size()
{
    DWORD dwDLLNum=0;
    DWORD dwFunNum=0;
    DWORD dwszDLLSize=0;
    DWORD dwszFuncSize=0;
    DWORD dwImportSize=0;
    t_IMAGE_IMPORT_TABLE    imageimport;
    t_IMAGE_THUNK           imagethunk;
    //ImportIter 是全局变量
    for(ImportIter=ImportTable.begin();ImportIter!=ImportTable.end();ImportIter++)
    {
        imageimport=*ImportIter;
        //注意库名长度计算，+1 就是结尾的 0
        dwszDLLSize=dwszDLLSize+strlen(imageimport.szLibrary)+1;
        //遍历 THUnk 链表
        for(imageimport.ThunkIter=imageimport.ThunksList.begin();
            imageimport.ThunkIter!=imageimport.ThunksList.end();
            imageimport.ThunkIter++)
        {
            imagethunk=*imageimport.ThunkIter;
            //注意长度计算，括号中为原始 THUNK 的结构
            //格式：0,0,函数名字符串,0
            dwszFuncSize=dwszFuncSize+(2+strlen(imagethunk.szFunction)+1);
            //函数计数
            dwFunNum++;
        }
        //以 0 结尾
        dwFunNum++;
        //库计数
        dwDLLNum++;
    }
    //以空 IMAGE_IMPORT_DESCRIPTOR 结构结尾
    dwDLLNum++;
    //每个 DLL 需要一个导入表(20 字节)+(API 函数数量)*(DWORD)+(所有 DLL 字符串长度)+(所有函数名字符串长度)
    //这个地方我计算的不是很明白，应该有两个 RVA 数组，一个是函数名 RVA 数组，一个是函数地址 RVA 数组
    //有知道的朋友请指出一下。
    dwImportSize=dwDLLNum*20+dwFunNum*4+dwszDLLSize+dwszFuncSize;
    return(dwImportSize);
}
//----------------------------------------------------------------
//----------------------------------------------------------------
// This function build the dll name strings, saves the ImageImportDescriptors to the loader data.
//建立导入表
void CITMaker::Build(DWORD dwRVA)
{
    //是个 RVA
    DWORD                 pITBaseRVA=dwRVA;
    DWORD                 temp;
    DWORD                 dwDLLNum, dwDLLName, dwDLLFirst, dwszDLLSize;
    DWORD                 dwIIDNum, dwFunNum, dwFunFirst, dwszFuncSize;
    DWORD                 dwFirstThunk, dwImportSize;
    //局部变量
    t_IMAGE_IMPORT_TABLE  imageimport;
    t_IMAGE_THUNK         imagethunk;
    //真正导入表结构
    IMAGE_IMPORT_DESCRIPTOR import_descriptor; // -> IID
    //初始化--------------------------------------------
    import_descriptor.OriginalFirstThunk=0;
    import_descriptor.TimeDateStamp=0;
    import_descriptor.ForwarderChain=0;
    import_descriptor.Name=0;
    import_descriptor.FirstThunk=0;
    dwDLLNum=dwDLLName=dwDLLFirst=dwszDLLSize=0;
    dwIIDNum=dwFunNum=dwFunFirst=dwszFuncSize=0;
    dwFirstThunk=dwImportSize=0;
    //遍历总链表--------------------------------------------
    for(ImportIter=ImportTable.begin();ImportIter!=ImportTable.end();ImportIter++)
    {
        imageimport=*ImportIter;
        dwszDLLSize=dwszDLLSize+strlen(imageimport.szLibrary)+1;
        for(imageimport.ThunkIter=imageimport.ThunksList.begin();
            imageimport.ThunkIter!=imageimport.ThunksList.end();
            imageimport.ThunkIter++)
        {
            imagethunk=*imageimport.ThunkIter;
            dwszFuncSize=dwszFuncSize+2+strlen(imagethunk.szFunction)+1;
            dwFunNum++;
        }
        dwFunNum++;
        dwDLLNum++;
    }
    dwDLLNum++;
    //重复得出大小，跟上面一样，为什么不调用函数计算呢，晕
    dwImportSize=dwDLLNum*20+dwFunNum*4+dwszDLLSize+dwszFuncSize;
    //pMem 已经初始化：在构造函数中，大小为所有引入表大小--------------------------------------------
    FillMemory(pMem,dwImportSize,0x00);
    //一些偏移量
    dwFirstThunk=dwDLLNum*20;
    dwDLLFirst=dwDLLNum*20+dwFunNum*4;
    dwFunFirst=dwDLLNum*20+dwFunNum*4+dwszDLLSize;
    //pITBaseRVA
    //--------------------------------------------
    for(ImportIter=ImportTable.begin();ImportIter!=ImportTable.end();ImportIter++)
    {
        imageimport=*ImportIter;
        //pITBaseRVA 为传递的参数
        //Name:含有指向 DLL 名字的 RVA
        //越过之前的所有 IMAGE_IMPORT_DESCRIPTOR 结构+所有 API 地址数组而到达指向 DLL 名字的 RVA
        import_descriptor.Name=pITBaseRVA+dwDLLFirst;
        //越过之前的所有 IMAGE_IMPORT_DESCRIPTOR 结构得到 FirstThunk 的 RVA
        import_descriptor.FirstThunk=pITBaseRVA+dwFirstThunk;
        //复制一系列 IMAGE_IMPORT_DESCRIPTOR
        memcpy(pMem+dwIIDNum*sizeof(IMAGE_IMPORT_DESCRIPTOR),
                   &import_descriptor,
                   sizeof(IMAGE_IMPORT_DESCRIPTOR));
        //复制动态库名称，注意位置 dwDLLFirst=dwDLLNum*20+dwFunNum*4！
        memcpy(pMem+dwDLLFirst,
                   imageimport.szLibrary,
                   strlen(imageimport.szLibrary)+1);
        //--------------------------------------------
        for(imageimport.ThunkIter=imageimport.ThunksList.begin();
            imageimport.ThunkIter!=imageimport.ThunksList.end();
            imageimport.ThunkIter++)
        {
            //函数名称链
            imagethunk=*imageimport.ThunkIter;
            //计算函数名称字符串 RVA
            temp=pITBaseRVA+dwFunFirst;
            //复制的是一个 RVA,包含函数名数组 RVA 的 RVA 数组
            //dwFirstThunk=dwDLLNum*20
            memcpy(pMem+dwFirstThunk,
                       &temp,
                       4);
            //THUNK 指向的 RVA 数组所指向的函数名数组
            //dwFunFirst=dwDLLNum*20+dwFunNum*4+dwszDLLSize
            memcpy(pMem+dwFunFirst+2,
                       imagethunk.szFunction,
                       strlen(imagethunk.szFunction)+1);
            //指向下一个位置
            dwFunFirst=dwFunFirst+2+strlen(imagethunk.szFunction)+1;
            dwFirstThunk=dwFirstThunk+4;
        }
        //--------------------------------------------
        temp=0;
        //以零结尾
        memcpy(pMem+dwFirstThunk,
                       &temp,
                       4);
        //指向下一个位置
        dwFirstThunk=dwFirstThunk+4;
        dwDLLFirst=dwDLLFirst+strlen(imageimport.szLibrary)+1;
        //表的数量加一
        dwIIDNum++;
    }
    //莫非是最后一个表，以空表结束--------------------------------------------
    import_descriptor.Name=0;
    import_descriptor.FirstThunk=0;
    //--------------------------------------------
    memcpy(pMem+dwIIDNum*sizeof(IMAGE_IMPORT_DESCRIPTOR),
                   &import_descriptor,
```



# 菜鸟注释 PEMaker6 源代码（三）

**作者**: 火影  
**时间**: 2007-09-19 00:09  
**链接**: [`bbs.pediy.com/showthread.php?t=51917`](http://bbs.pediy.com/showthread.php?t=51917)

今天大概看了一下精华 8《向 PE 中注入代码》的译文，之前没有仔细看过，因为发现看过之后对源代码还是不是很明白，所以就没有参照这篇译文进行注释，大多都是自己理解+网上搜索，今天发现很多地方有失误。![伤心](img/26c3a0df6b585d53c82b982893c84a26.jpg)

大框的地方一定要参照译文，不要被我蒙蔽了。![恐怖!](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg)

继续注释 `CPECryptor` 类，如下：

```cpp
//在 pecrypt.cpp 里，我提供了另外一个类——CPECryptor，用它组建新区段的数据。
//然而，新区段的数据是被 loader.cpp 里的 DynLoader()（在 DynLoader Step 1 中介绍的）创建的。
//因此，我们用 CPECryptor 类把这些数据（也有其它的数据）输入新区段。
//看完这段翻译基本没懂

const char *szWindowsAPIs[] = 
{ 
    "Kernel32.dll", 
    "GetModuleHandleA", 
    "VirtualProtect", 
    "GetModuleFileNameA", 
    "CreateFileA", 
    "GlobalAlloc", 
    "VirtualAlloc", 
    "LoadLibraryA", 
    "GetProcAddress", 
    0, 
    "User32.dll", 
    "MessageBoxA", 
    0, 
    0, 
}; 

//================================================================
//----------------------------------------------------------------
// Function: ReturnToBytePtr
// void* FuncNum:   Function Name
// DWORD findstr:   String to find
// This code was written by FEUERRADER [AHTeam], Thanks him!
//在 loader.cpp 文件中的 DynLoader 函数里寻找指定字符串，并返回其地址

void* CPECryptor::ReturnToBytePtr(void* FuncName, DWORD findstr) 
{ 
    void* tmpd; 
    __asm 
    { 
        mov eax, FuncName 
        jmp df 
hjg:    inc eax 
df:        mov ebx, [eax] 
        cmp ebx, findstr 
        jnz hjg 
        mov tmpd, eax 
    } 
    return tmpd; 
} 

//================================================================
//此函数构建新节及导入表----------------------------------------------------------------

void CPECryptor::CryptFile(int(__cdecl *progress) (unsigned int, unsigned int)) 
{ 
    PCHAR ch_temp; 
    PIMAGE_SECTION_HEADER pimage_section_header; 
    DWORD dwNewSectionSize; 
    DWORD dwCodeSize; 
    DWORD dwCodeOffset; 
    //进度条控件----------------------------------------
    progress(0,0); 
    //是否 DLL
    if((image_nt_headers->FileHeader.Characteristics&IMAGE_FILE_DLL)==IMAGE_FILE_DLL) 
    { 
        //构造 CITMaker 类
        ImportTableMaker = new CITMaker(IMPORT_TABLE_OCX); 
    } 
    else 
    { 
        ImportTableMaker = new CITMaker(IMPORT_TABLE_EXE); 
    } 
    //----------------------------------------
    //========================================
    //ch_temp 指向 DYN_LOADER_START_MAGIC 之后的地址
    ch_temp=(PCHAR)DWORD(ReturnToBytePtr(DynLoader, DYN_LOADER_START_MAGIC))+4; 
    //计算 DynLoader 函数代码长度
    dwCodeSize=DWORD(ReturnToBytePtr(DynLoader, DYN_LOADER_END_MAGIC))-DWORD(ch_temp); 
    //自定义导入表大小
    dwCodeOffset = ImportTableMaker->dwSize; 
    //新节大小=DynLoader 函数代码长度+自定义导入表大小
    dwNewSectionSize = dwCodeSize + ImportTableMaker->dwSize; 
    //分配空间
    pNewSection=new TCHAR[dwNewSectionSize]; 
    //复制 DynLoader 函数代码到偏移 dwCodeOffset 的位置之后
    memcpy(pNewSection+dwCodeOffset, ch_temp, dwCodeSize); 
    //========================================
    //----------------------------------------
    //添加新节，并返回新节指针
    pimage_section_header=AddNewSection(".xxx",dwNewSectionSize); 
    //----------------------------------------
    //========================================
    //参数为新节在内存的偏移地址
    CopyData1(pimage_section_header->VirtualAddress); 
    //参数同上，构建自定义导入表
    ImportTableMaker->Build(pimage_section_header->VirtualAddress);// build import table by the current virtual address
    //构建的导入表在新节数据前，注意相互位置
    memcpy(pNewSection, ImportTableMaker->pMem, ImportTableMaker->dwSize); 
    //========================================
    //将新节数据复制到 image_section(已经初始化)----------------------------------------
    memcpy(image_section[image_nt_headers->FileHeader.NumberOfSections-1], 
           pNewSection, 
           dwNewSectionSize); 
    //入口地址更改为新节的 DynLoader 函数代码处
    image_nt_headers->OptionalHeader.AddressOfEntryPoint=pimage_section_header->VirtualAddress + dwCodeOffset; 
    //导入表地址更改为新导入表地址
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress=pimage_section_header->VirtualAddress; 
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].Size=ImportTableMaker->dwSize; 
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress=0; 
    image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].Size=0; 
    SetSectionsWritePermission(); 
    //----------------------------------------
    delete []pNewSection; 
    delete ImportTableMaker; 
    //----------------------------------------
    progress(100,0); 
} 

//这个结构对应 DynLoader 函数里 DYN_LOADER_START_DATA1 标志后面的东西，具体
//是自定义结构？请高手指点，本人水平到此，分析不出了
//译文：为了找到 OEP 的 virtual address，我们立即保存原始的 OEP 和 Image Base。
//我在 DynLoader()的尾部预留了一块空地来保存它们

typedef struct 
{ 
    //保留
    DWORD dwReserved1; 
    //文件类型
    DWORD dwFileType; 
    //基地址
    DWORD dwImageBase; 
    //原始入口点
    DWORD dwOrgEntryPoint; 
    //导入表虚拟地址
    DWORD dwImportVirtualAddress; 
    //重定位表虚拟地址及大小
    DWORD dwRelocationVirtualAddress; 
    DWORD dwRelocationSize; 
    //TLS
    IMAGE_TLS_DIRECTORY32 image_tls_directory; 
}t_DATA_1,*pt_DATA_1; 

//此函数将实际数据填充到 DynLoader 函数指定位置里

void CPECryptor::CopyData1(DWORD dwVirtualAddress) 
{ 
    int i, API_num; 
    PCHAR pData1; 
    DWORD dwOffset; 
    size_t l; 
    UCHAR temp; 
    //为结构分配内存
    pt_DATA_1 pDataTable=new(t_DATA_1); 
    //----------------------------------------
    //保留,0xcccccccc 应该没有意义
    pDataTable->dwReserved1=0xCCCCCCCC; 
    //如果为 DLL，填入文件类型
    if((image_nt_headers->FileHeader.Characteristics&IMAGE_FILE_DLL)==IMAGE_FILE_DLL) 
        pDataTable->dwFileType=IMPORT_TABLE_OCX; 
    else 
        pDataTable->dwFileType=IMPORT_TABLE_EXE; 
    //填充镜像首地址
    pDataTable->dwImageBase=image_nt_headers->OptionalHeader.ImageBase; 
    //填充入口点
    pDataTable->dwOrgEntryPoint=image_nt_headers->OptionalHeader.AddressOfEntryPoint; 
    //填充导入表地址(RVA)
    pDataTable->dwImportVirtualAddress=image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress; 
    if((image_nt_headers->FileHeader.Characteristics&IMAGE_FILE_DLL)==IMAGE_FILE_DLL) 
    { 
        //如果是 DLL，填充重定位表
        pDataTable->dwRelocationVirtualAddress=image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress; 
        pDataTable->dwRelocationSize=image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].Size; 
    } 
    //前面已经将 DynLoader 函数读入我们申请的内存中----------------------------------------
    //故在 pNewSection 指向的内存中找到 DYN_LOADER_START_DATA1 标志，返回地址
    pData1=(PCHAR)ReturnToBytePtr(pNewSection, DYN_LOADER_START_DATA1); 
    //看了好几个 TLS 了，是什么？
    //通过使用线程本地储存（TLS），一个程序能够执行多线程程序，
    //这样子的程序大多数是用 Borland 链接器：Delphi 和 C++, Builder.
    //当你包装一个 PE 文件时，你应该小心的清空 TLS，
    //否则，你的打包器就不支持 Borland Delphi 和 C++ Builder 链接的 EXE 文件。
    //修正可选头部 TLS 目录入口是很有必要的。
    if(image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress!=0) 
```



`//image_tls_directory` 定义在 `pelib.cpp` 中

```
memcpy(&pDataTable->image_tls_directory,  
       image_tls_directory,  
       sizeof(IMAGE_TLS_DIRECTORY32));  
dwOffset=DWORD(pData1)-DWORD(pNewSection);  
//计算偏移  
dwOffset+=sizeof(t_DATA_1)-sizeof(IMAGE_TLS_DIRECTORY32);  
//更新文件头，指向新的 TLS 表  
//传递的参数在这里应用  
image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS].VirtualAddress=dwVirtualAddress + dwOffset;  
   }  
   //替换为实际数值----------------------------------------  
   memcpy(pData1,pDataTable,sizeof(t_DATA_1));  
   dwOffset=sizeof(t_DATA_1);  
   i=API_num=0;  
   temp=0;  
   //循环填充实际数值  
   do  
   {  
       l=strlen(szWindowsAPIs[i])+1;  
       //复制动态库名  
       memcpy(pData1+dwOffset,szWindowsAPIs[i],l);  
       //偏移长度  
       dwOffset+=l;  
       do  
       {  
           i++;  
           if(szWindowsAPIs[i]!=0)  
           {  
               //复制下一个函数名  
               l=strlen(szWindowsAPIs[i])+1;  
               memcpy(pData1+dwOffset,szWindowsAPIs[i],l);  
               dwOffset+=l;  
               //API 数量  
               API_num++;  
           }  
           else  
           {  
               //放置 0  
               CopyMemory(pData1+dwOffset,&temp,1);  
               dwOffset++;  
           }  
       }while(szWindowsAPIs[i]!=0);  
       i++;  
   }  
   while(szWindowsAPIs[i]!=0);  
   //----------------------------------------  
   delete pDataTable;  
}  
//----------------------------------------------------------------  
void CPECryptor::SetSectionsWritePermission()  
{  
    for(int i=0;i<image_nt_headers->FileHeader.NumberOfSections;i++)  
    {  
        //设置所有表的标志  
        image_section_header[i]->Characteristics=0xC0000040;  
    }  
}  
//---------------------------------------------------------------- 
```

### 标题: 菜鸟注释 PEMaker6 源代码（四）

**作者**: 火影

**时间**: 2007-09-25 23:16

*   链接：[`bbs.pediy.com/showthread.php?t=52332`](http://bbs.pediy.com/showthread.php?t=52332)

详细信息:

> 这是最后一篇了，最近被调去急诊科，三天一个夜班，头都大了，注释的很乱，连自己都搞晕了。

```
#include "stdafx.h"  
#include "loader.h"  
#include "itmaker.h"  

#ifdef _DEBUG  
#define DEBUG_NEW  
#endif  

__stdcall void DynLoader();  
//---------------------------------------------------------  
//  Thanks FEUERRADER [AHTeam]  for  idea  about  using  _emit 0!  
//---------------------------------------------------------  
//  byte  ptr        (1 bytes)  
//定义一个字节  
#define byte_type(x)         __asm _emit x  
//  word  ptr        (2 bytes)  
//低位放低字节  
#define word_type(x)         byte_type((x>>(0*8))&0xFF)   byte_type((x>>(1*8))&0xFF)  
//  dword ptr    (4 bytes)  
#define dword_type(x)        byte_type((x>>(0*8))&0xFF)   byte_type((x>>(1*8))&0xFF)   byte_type((x>>(2*8))&0xFF)   byte_type((x>>(3*8))&0xFF)  
//  dword64 ptr    (8 bytes)  
#define dword64_type(x)      dword_type(x)   dword_type(x)  
//  dword64 ptr    (16 bytes)  
#define dword128_type(x)     dword64_type(x)   dword64_type(x)  
//---------------------------------------------------------  
//定义字节  
#define bb(x)                __asm _emit x  
#define db                   byte_type(0xCC)  
#define __random_code1__     dword64_type(0X90909090) //  Reserve  for  random  code  generation  
//jmp  
#define __jmp_api            byte_type(0xFF)   byte_type(0x25)  

//----------------------------------------------------------------  
__stdcall void DynLoader()  
{  
_asm  
{  
//----------------------------------  
    dword_type(DYN_LOADER_START_MAGIC)  
//----------------------------------  
_main_0:  
    pushad    //  save  the  registers  context  in  stack  
    //重定向编码放 EBP  
    call _main_1  
_main_1:  
    pop ebp  
    sub ebp,offset _main_1 //  get  base  ebp  
    //----------------------------------------------------  
    //====================================================  
    //----------------  support  dll,  ocx   -----------------  
_support_dll_0:  
    jmp _support_dll_1 //  nop;  nop;  //  in  the  secon  time  OEP  
    jmp _support_dll_2  
_support_dll_1:  
    //定义在下面，就是那个类似 t_DATA_1 结构的东西  
    test [ebp+_p_dwFileType],IMPORT_TABLE_OCX  
    jz _no_dll_pe_file_0  
        //esp 怎么来的？  
        mov eax,[esp+24h]//  imagebase  
        //AddressOfEntryPoint 成员  
        mov ebx,[esp+30h]//  oep  
        cmp eax,ebx  
        ja _no_dll_pe_file_0  
            cmp word ptr [eax],IMAGE_DOS_SIGNATURE  
            jne _no_dll_pe_file_0  
                //存入镜像地址  
                mov [ebp+_p_dwImageBase],eax  
//无 DLL 的 PE 文件？  
_no_dll_pe_file_0:  
    //----------------------------------------------------  
    //====================================================  
    mov eax,[ebp+_p_dwImageBase]  
    add eax,[eax+03Ch]  
    add eax,080h  
    //获取导入表 RVA  
    mov ecx,[eax]    //  image_nt_headers->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress  
    //ecx 为导入表地址  
    add ecx,[ebp+_p_dwImageBase]  
    add ecx,010h    //  image_import_descriptor.FirstThunk  
    //取 FirstTHUNk 值  
    mov eax,[ecx]  
    add eax,[ebp+_p_dwImageBase]  
    mov ebx,[eax]  
    //存入 API 地址  
    mov [ebp+_p_LoadLibrary],ebx  
    add eax,04h  
    mov ebx,[eax]  
    //存入 API 地址  
    mov [ebp+_p_GetProcAddress],ebx  
    //----------------------------------------------------  
    //-------  load  library  and  build  api  call-jmp  --------  
    call _api_load  
    //----------------------------------------------------  

    push MB_OK | MB_ICONINFORMATION  
    lea eax,[ebp+_p_szCaption]  
    push eax  
    lea eax,[ebp+_p_szText]  
    push eax  
    push NULL  
    call _jmp_MessageBox  
    //  MessageBox(NULL,  szText,  szCaption,  MB_OK  |  MB_ICONINFORMATION)  ;  

    //...  
    //...  
    //...  

    //  Place  your  code  here  ...  

    //...  
    //...  
    //...  

    //----------------------------------------------------  
    //====================================================  
    //-----------  get  write  access  for  headers  -----------  
    mov edi,[ebp+_p_dwImageBase]  
    add edi,[edi+03Ch]//  edi  ->  IMAGE_NT_HEADERS  
    //  get  write  permission  by  VirtualProtect()  
    lea eax,[ebp+_p_ptempbuffer]  
    push eax  
    push PAGE_READWRITE  
    push [edi+0x54]//IMAGE_NT_HEADERS.OptionalHeader.SizeOfHeaders  
    push [ebp+_p_dwImageBase]  
    call _jmp_VirtualProtect  
    //VirtualProtect(dwImageBase,image_nt_header.OptionalHeader.SizeOfHeaders,PAGE_READWRITE,ptempbuffer);  
    //----------------------------------------------------  
    //----------------------------------------------------  
```



```asm
    > //--------------- import table fix up ----------------

    > lea eax,[ebp+_p_szKERNEL32_r]

    > push eax

    > push [ebp+_p_dwImportVirtualAddress]

    > push [ebp+_p_dwImageBase]

    > //以上传递参数

    > call _it_fixup_1

    >

    > //--------------- import table fix up ----------------

    > //call _it_fixup

    > //----------------------------------------------------

    > //====================================================

    > //---------------- support dll, ocx  1 ---------------

    > //by set second way to oep and relocation table effect

    > mov edi,[ebp+_p_dwImageBase]

    > add edi,[edi+03Ch]// edi -> IMAGE_NT_HEADERS

    > mov ax,word ptr [edi+016h]// edi -> image_nt_headers->FileHeader.Characteristics

    > // .IF ((image_nt_headers->FileHeader.Characteristics&IMAGE_FILE_DLL) ==IMAGE_FILE_DLL)

    > test ax,IMAGE_FILE_DLL

    > jz _no_dll_pe_file_1

    >     //--------------- relocation fix up --------------

    >     call _reloc_fixup

    >     //------------------------------------------------

    >     mov ax,9090h

    >     mov word ptr [ebp+_support_dll_0],ax

    > // .ENDIF

    > _no_dll_pe_file_1:

    > //----------------------------------------------------

    > //====================================================

    > _support_dll_2:

    > //----------------------------------------------------

    > //====================================================

    > //--------- Prepare the SEH for OEP approach ---------

    > mov eax,[ebp+_p_dwImageBase]

    > add eax,[ebp+_p_dwOrgEntryPoint]

    > mov [esp+10h],eax    // pStack.Ebx <- eax

    > lea eax,[ebp+_except_handler1_OEP_Jump]

    > mov [esp+1Ch],eax    // pStack.Eax <- eax

    > popad    // restore the first registers context from stack

    > //----------------------------------------------------

    >     // the structured exception handler (SEH) installation

    > push eax

    > xor  eax,  eax

    > push dword ptr fs:[0]        // NT_TIB32.ExceptionList

    > mov fs:[0],esp    // NT_TIB32.ExceptionList <-ESP

    > dword_type(0xCCCCCCCC)// raise a (int 3) exception

    > //----------------------------------------------------

    > //====================================================

    > //--------------------------------------------------------

    > //========================================================

    > //------------- t_size strlen(LPCTSTR lpStr) -------------

    > __strlen:

    >     push ebp

    >     mov ebp,esp

    >     push ecx

    >     push esi

    >     push ebx

    >     //取得参数

    >     mov esi,[ebp+08h]// -> destination

    >     mov ecx,255// -> length

    >     xor ebx,ebx

    > _strlenloop:

    >         lods byte ptr ds:[esi]

    >         cmp al,00h

    >         jz _endbufstrlen

    >         //ebx 计数

    >         inc ebx

    >     loop _strlenloop

    > _endbufstrlen:

    >     mov eax,ebx

    >     //返回值在 eax

    >     inc eax

    >     pop ebx

    >     pop esi

    >     pop ecx

    >     mov esp,ebp

    >     pop ebp

    >     ret

    > //--------------------------------------------------------

    > //========================================================

    > //------------------- fix up relocation ------------------

    > // I have stolen this reloc fix-up code from Morphine 2.7 !!

    > // so Thanks Holy_Father && Ratter/29A for it. (www.hxdef.org)

    > _reloc_fixup:

    >     mov eax,[ebp+_p_dwImageBase]

    >     //edx 首地址

    >     mov edx,eax

    >     mov ebx,eax

    >     //ebx-->NT 文件头

    >     add ebx,[ebx+3Ch] // edi -> IMAGE_NT_HEADERS

    >     //其实 RVA 等于 FC

    >     mov ebx,[ebx+034h]// ebx ->image_nt_headers->OptionalHeader.ImageBase

    >     //有可能为 0

    >     sub edx,ebx // edx -> reloc_correction

    >     je _reloc_fixup_end

    >     mov ebx,[ebp+_p_dwRelocationVirtualAddress]

    >     test ebx,ebx

    >     jz _reloc_fixup_end

    >     //ebx 是 RVA

    >     add ebx,eax

    > _reloc_fixup_block:

    >     //一个 32 位的“SizeOfBlock（块大小）”

    >     mov eax,[ebx+004h]            //ImageBaseRelocation.SizeOfBlock

    >     test eax,eax

    >     jz _reloc_fixup_end

    >     lea ecx,[eax-008h]            //ImageBaseRelocation.SizeOfBlock - sizeof(TImageBaseRelocation)

    >     //ecx 为跟在后面的重定位的数目

    >     shr ecx,001h                //WORD((ImageBaseRelocation.SizeOfBlock - sizeof(TImageBaseRelocation)) / sizeof(Word))

    >     //edi 为重定位数据开始地址

    >     lea edi,[ebx+008h]            //PImageBaseRelocation + sizeof(TImageBaseRelocation)

    > _reloc_fixup_do_entry:

    >         movzx eax,word ptr [edi]//Entry

    >         push edx

    >         mov edx,eax

    >         //高 4 位的重定位类型

    >         shr eax,00Ch            //Type = Entry >> 12

    >

    >         mov esi,[ebp+_p_dwImageBase]//ImageBase

    >         //低 12 位的重定位位置

    >         and dx,00FFFh

    >         //得到计算后的偏移地址

    >         add esi,[ebx]            //ImageBase + ImageBaseRelocation.VirtualAddress

    >         //得到重定位数据的偏移地址

    >         add esi,edx                //ImageBase + ImageBaseRelocation.VirtualAddress+Entry & 0x0FFF

    >         pop edx

    >         //------------------------------

    >         //---- IMAGE_REL_BASED_HIGH 重定位的高 16 位必须被用于被偏移量所指向的那个 16 位的 WORD 单元，

    >         //此 WORD 是一个 32 位的 DWORD 的高位 WORD----

    > _reloc_fixup_HIGH:

    >         //重定位类型

    >         dec eax

    >         jnz _reloc_fixup_LOW

    >             //edx 重定位偏移

    >             mov eax,edx

    >             //取高字节

    >             shr eax,010h        //HIWORD(Delta)

    >             jmp _reloc_fixup_LOW_fixup

    >         //------------------------------

    >         //---- IMAGE_REL_BASED_LOW -----

    > _reloc_fixup_LOW:

    >             dec eax

    >         jnz _reloc_fixup_HIGHLOW

    >         movzx eax,dx            //LOWORD(Delta)

    > _reloc_fixup_LOW_fixup:

    >             //

    >             add word ptr [esi],ax

    >         jmp _reloc_fixup_next_entry

    >         //------------------------------

    >         //-- IMAGE_REL_BASED_HIGHLOW ---

    > _reloc_fixup_HIGHLOW:

    >             dec eax

    >         jnz _reloc_fixup_next_entry

    >         add [esi],edx

    >         //------------------------------

    > _reloc_fixup_next_entry:

    >         inc edi

    >         inc edi                    //Entry++

    >         loop _reloc_fixup_do_entry

    >

    > _reloc_fixup_next_base:

    >     add ebx,[ebx+004h]                //ImageBaseRelocation + ImageBaseRelocation.SizeOfBlock

    >     jmp _reloc_fixup_block

    > _reloc_fixup_end:

    >     // clean relocation table

    > /*    mov eax,[ebp+_p_dwImageBase]

    >     add eax,[ebp+_p_dwRelocationVirtualAddress]

    >     lea edi,[eax]

    >     xor eax,eax

    >     add ecx,[ebp+_p_dwRelocationSize]

    >     rep stos byte  ptr [edi]*/

    >     ret

    > //--------------------------------------------------------

    > //========================================================

    > //--------------- 修复导入表 ----------------

    > /*
    > ebp+10h    [ebp+_p_szKERNEL32_r]
```



```
ebp+0ch    _p_dwImportVirtualAddress
ebp+08h    _p_dwImageBase

---------------------------------

ebp-04h    _p_dwNewITaddress
ebp-08h    _p_dwThunk
ebp-0ch    _p_dwHintName
ebp-10h    _p_dwLibraryName
ebp-14h    _p_dwAPIaddress

*/
_it_fixup_1:
    push ebp
    mov ebp,esp
    add esp,-14h
    push PAGE_READWRITE
    push MEM_COMMIT
    push 01D000h
    push 0
    call _jmp_VirtualAlloc
    //NewITaddress=VirtualAlloc(NULL, 0x01D000, MEM_COMMIT, PAGE_READWRITE);
    //存入地址
    mov [ebp-04h],eax
    mov ebx,[ebp+0ch]
    test ebx,ebx
    jz _it_fixup_1_end
    //esi:_p_dwImageBase
    mov esi,[ebp+08h]
    //导入表地址
    add ebx,esi                                     // dwImageBase + dwImportVirtualAddress
_it_fixup_1_get_lib_address_loop:
        //名称
        mov eax,[ebx+0ch]                            // image_import_descriptor.Name
        test eax,eax
        jz _it_fixup_1_end
        //FirstThunk RVA
        mov ecx,[ebx+10h]                            // image_import_descriptor.FirstThunk
        add ecx,esi
        //FirstThunk 指向的 RVA 数组
        mov [ebp-08h],ecx                             // dwThunk
        mov ecx,[ebx]                                 // image_import_descriptor.Characteristics
        //有些编译器会将此置 0
        test ecx,ecx
        jnz _it_fixup_1_table
            // image_import_descriptor.FirstThunk
            mov ecx,[ebx+10h]
_it_fixup_1_table:
        //esi:_p_dwImageBase
        add ecx,esi
        mov [ebp-0ch],ecx  // dwHintName
        add eax,esi                                // image_import_descriptor.Name + dwImageBase = ModuleName
        push eax                                    // lpLibFileName
        mov [ebp-10h],eax
        call _jmp_LoadLibrary                        // LoadLibrary(lpLibFileName);

        test eax,eax
        jz _it_fixup_1_end
        //edi:动态链接库句柄
        mov edi,eax
_it_fixup_1_get_proc_address_loop:
            //指向一个 IMAGE_THUNK_DATA 结构数组
            mov ecx,[ebp-0ch]            // dwHintName
            mov edx,[ecx]                             // image_thunk_data.Ordinal
            test edx,edx
            jz _it_fixup_1_next_module
            //以序数方式
            test edx,080000000h                         // .IF( import by ordinal )
            //对于每个数组元素，我们比对元素值是否等于 IMAGE_ORDINAL_FLAG32。
            //如果该元素值的最高二进位为 1，
            //那么函数是由序数引入的，可以从该值的低字节提取序数。
            jz _it_fixup_1_by_name
                //取出序数
                and edx,07FFFFFFFh                    // get ordinal
                jmp _it_fixup_1_get_addr
_it_fixup_1_by_name:
            add edx,esi                                // image_thunk_data.Ordinal + dwImageBase = OrdinalName
            inc edx
            //取出 API 名称
            inc edx                                     // OrdinalName.Name
_it_fixup_1_get_addr:
            push edx                                    // lpProcName
            push edi                                    // hModule
            call _jmp_GetProcAddress                    // GetProcAddress(hModule, lpProcName);
            //获得 API 地址
            mov [ebp-14h],eax                            //_p_dwAPIaddress
            //================================================================
            //mov [ecx],eax//<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
            //hModule
            push edi
            // dwImageBase
            push esi
            //ebx=引入表地址
            push ebx

            mov ebx,[ebp-10h]
            push ebx
            push ebx
            //大写
            call _char_upper
            //esi=_p_dwLibraryName
            mov esi,[ebp-10h]
            mov edi,[ebp+010h]    // [ebp+_p_szKernel32]
_it_fixup_1_check_dll_redirected:
            push edi
            call __strlen
            add esp, 4

            mov ebx,eax
            mov ecx,eax
            push edi
            push esi
            //比较两个动态库字符串
            repe cmps //byte ptr [edi], byte ptr [esi]
            jz _it_fixup_1_do_normal_it_0
            pop esi
            pop edi
            //下一个
            add edi,ebx
            //是否结束
            cmp byte ptr [edi],0
            jnz _it_fixup_1_check_dll_redirected
                //_p_dwThunk
                mov ecx,[ebp-08h]
                //api 地址
                mov eax,[ebp-014h]
                //FirstThunk 指向的数组被填充为真正的函数地址
                mov [ecx],eax
                jmp _it_fixup_1_do_normal_it_1
_it_fixup_1_do_normal_it_0:
            //对应 repe 之前的 push
                pop esi
                pop edi
                //新申请的内存空间
                mov edi,[ebp-04h]
                //插入 JMP
                mov byte ptr [edi], 0e9h
                //API 地址
                mov eax,[ebp-14h]
                //计算跳转偏移
                sub eax, edi
                sub eax,05h
                //插入偏移
                mov [edi+1],eax
                //0c8bh-->mov eax,eax,对齐
                mov word ptr [edi+05],0c8bh
                //THUNK
                mov ecx,[ebp-08h]
                //将原程序的 THUNK 指向我们的内存地址
                mov [ecx],edi
                //下一个位置
                add dword ptr [ebp-04h],07h
_it_fixup_1_do_normal_it_1:
            pop ebx
            pop esi
            pop edi
            //================================================================
            add dword ptr [ebp-08h],004h    // dwThunk => next dwThunk
            add dword ptr [ebp-0ch],004h// dwHintName => next dwHintName
        jmp _it_fixup_1_get_proc_address_loop
_it_fixup_1_next_module:
        add ebx,014h                         // sizeof(IMAGE_IMPORT_DESCRIPTOR)
    jmp _it_fixup_1_get_lib_address_loop
_it_fixup_1_end:
    mov esp,ebp
    pop ebp
    ret 0ch
//--------------------------------------------------------
//========================================================
//-- char_upper(LPCTSTR lpDestination, LPCTSTR lpSource )
_char_upper:
    push ebp
    mov ebp,esp
    push ecx
    push eax
    push esi
    push edi
    mov edi,dword ptr [ebp+08h]// -> Destination
    mov esi,dword ptr [ebp+0Ch]// -> Source
    mov ecx,255// -> Length
    xor eax,eax
__makeupperloop:
        lods byte ptr [esi]//ESI
```



```
    >       cmp al,00h  
    >       jz endofbuffer  
    >       cmp al,60h  
    >       jc notinlowerfield  
    >       cmp al,7bh  
    >       jnc notinlowerfield  
    >       sub al,20h  
    > notinlowerfield:  
    >       stos byte ptr [edi]//EDI  
    >   loop __makeupperloop  
    > endofbuffer:  
    >   pop edi  
    >   pop esi  
    >   pop eax  
    >   pop ecx  
    >   pop ebp  
    >   retn 08h  
    > //--------------------------------------------------------  
    > //========================================================  
    > //------------- load necessary api functions -------------  
    > //以下都是重定位之后的地址  
    > _api_load:  
    >   //取得_p_szKernel32 字符串地址  
    >   lea edi,[ebp+_p_szKernel32]  
    >   //取得_p_GetModuleHandle 的地址  
    >   lea ebx,[ebp+_p_GetModuleHandle]  
    >   //取得_jmp_GetModuleHandle 地址  
    >   lea ecx,[ebp+_jmp_GetModuleHandle]  
    >   //取得_jmp_GetModuleFileName 的地址？  
    >   add ecx,02h  
    > _api_get_lib_address_loop:  
    >       //保存 ECX  
    >       push ecx  
    >       push edi  
    >       mov eax,offset _p_LoadLibrary  
    >       //加载 kernel32.dll  
    >       call [ebp+eax]// LoadLibrary(lpLibFileName);  
    >       //-------------------  
    >       pop ecx  
    >       //kernel32.dll 模块地址  
    >       mov esi,eax    // esi -> hModule  
    >       //字符串长度  
    >       push edi  
    >       call __strlen  
    >       //平衡堆栈,弹出参数  
    >       add esp,04h  
    >       //指向下一个字符串  
    >       add edi,eax  
    > _api_get_proc_address_loop:  
    >           push ecx  
    >           //edi=getmodulhanda  
    >           push edi  
    >           //esi=hmodule  
    >           push esi  
    >           mov eax,offset _p_GetProcAddress  
    >           call [ebp+eax]// GetModuleHandle=GetProcAddress(hModule, lpProcName);  
    >           //--------------------  
    >           pop ecx  
    >           //存入 GetModuleHandle 的实际地址  
    >           mov [ebx],eax  
    >           //注意看这里！！  
    >           mov [ecx],ebx  
    >           //下一个  
    >           add ebx,04h  
    >           //下一个  
    >           add ecx,06h  
    >           //下一个  
    >           push edi  
    >           call __strlen  
    >           add esp,04h  
    >           add edi,eax  
    >           mov al,byte ptr [edi]  
    >       test al,al  
    >       //不为 0 继续循环  
    >       jnz _api_get_proc_address_loop  
    >       //edi 地址加一  
    >       inc edi  
    >       mov al,byte ptr [edi]  
    >   test al,al  
    >   //不为 0 则载入 user32.dll  
    >   jnz _api_get_lib_address_loop  
    >   ret  
    > //----------------------------------------------------------  
    > //--------------------------------------------------------  
    > //========================================================  
    > // -------- exception handler expression filter ----------  
    > _except_handler1_OEP_Jump:  
    >   push ebp  
    >   mov ebp,esp  
    >   mov eax,[ebp+010h]    // PCONTEXT: pContext <- eax  
    >   //---------------  
    >   push edi  
    >   // restore original SEH  
    >   mov edi,[eax+0C4h]    // pContext.Esp  
    >   push dword ptr ds:[edi]  
    >   pop dword ptr fs:[0]  
    >   add dword ptr [eax+0C4h],8    // pContext.Esp  
    >   
    >   // set the Eip to the OEP  
    >   mov edi,[eax+0A4h] // edi <- pContext.Ebx  
    >   mov [eax+0B8h],edi // pContext.Eip <- edi  
    >   //  
    >   pop edi  
    >   //---------------  
    >   mov eax, EXCEPTION_CONTINUE_SEARCH  
    >   leave  
    >   ret  
    > //--------------------------------------------------------  
    > //========================================================  
    >   dword_type(DYN_LOADER_START_DATA1)  
    > //0xcccccccc 自填充----------------------------------  
    > _p_dwFileType:                    dword_type(0xCCCCCCCC)  
    > _p_dwImageBase:                    dword_type(0xCCCCCCCC)  
    > _p_dwOrgEntryPoint:                dword_type(0xCCCCCCCC)  
    > _p_dwImportVirtualAddress:        dword_type(0xCCCCCCCC)  
    > _p_dwRelocationVirtualAddress:    dword_type(0xCCCCCCCC)  
    > _p_dwRelocationSize:            dword_type(0xCCCCCCCC)  
    > //----------------------------------  
    > _tls_dwStartAddressOfRawData:    dword_type(0xCCCCCCCC)  
    > _tls_dwEndAddressOfRawData:        dword_type(0xCCCCCCCC)  
    > _tls_dwAddressOfIndex:            dword_type(0xCCCCCCCC)  
    > _tls_dwAddressOfCallBacks:        dword_type(0xCCCCCCCC)  
    > _tls_dwSizeOfZeroFill:            dword_type(0xCCCCCCCC)  
    > _tls_dwCharacteristics:            dword_type(0xCCCCCCCC)  
    > //----------------------------------  
    > _p_szKernel32:                //db "Kernel32.dll",0,13  
    >       db db db db db db db db db db db db db  
    > _p_szGetModuleHandle:        //db "GetModuleHandleA",0,17  
    >       db db db db db db db db db db db db db db db db db  
    > _p_szVirtualProtect:        //db "VirtualProtect",0,15  
    >       db db db db db db db db db db db db db db db  
    > _p_szGetModuleFileName:    //db "GetModuleFileNameA",0,19  
    >       db db db db    db db db db    db db db db db db db db db db  
    > _p_szCreateFile:            //db "CreateFileA",0,12  
    >       db db db db db db db db db db db db  
    > _p_szGlobalAlloc:            //db "GlobalAlloc",0,12  
    >       db db db db db db db db db db db db  
    > _p_szVirtualAlloc:            //db "VirtualAlloc",0,13  
    >       db db db db db db db db db db db db db  
    > _p_szLoadLibrary:            //db "LoadLibraryA",0,13  
    >       db db db db db db db db db db db db db  
    > _p_szGetProcAddress:        //db "GetProcAddress",0,15  
    >       db db db db db db db db db db db db db db db  
    >       byte_type(0)  
    >   
    > _p_szUser32:                //db "User32.dll",0,11  
    >       db db db db db db db db db db db  
    > _p_szMessageBox:            //db "MessageBoxA",0,12  
    >       db db db db db db db db db db db db  
    >       byte_type(0)  
    >       byte_type(0)  
    > //----------------------------------  
    > _p_LoadLibrary:                    dword_type(0xCCCCCCCC)  
    > _p_GetProcAddress:                dword_type(0xCCCCCCCC)  
    > _p_GetModuleHandle:  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >                           dword_type(0xCCCCCCCC)  
    >   
    > _jmp_GetModuleHandle:            __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_VirtualProtect:            __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_GetModuleFileName:            __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_CreateFile:                __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_GlobalAlloc:                __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_VirtualAlloc:                __jmp_api    dword_type(0xCCCCCCCC)  
    > _jmp_LoadLibrary:                __jmp_api    dword_type(0xCCCCCCCC)  
```



> `\_jmp\_GetProcAddress: ` `__jmp_api` `dword_type(0xCCCCCCCC)`
>
> `\_jmp\_MessageBox: ` `__jmp_api` `dword_type(0xCCCCCCCC)`
> `\_p\_szKERNEL32\_r:`
> //`db "KERNEL32.DLL",0,13`
> `bb('K') bb('E') bb('R') bb('N') bb('E') bb('L') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)`
> //`db "USER32.DLL",0,11`
> `bb('U') bb('S') bb('E') bb('R') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)`
> //`db "GDI32.DLL",0,10`
> `bb('G') bb('D') bb('I') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)`
> //`db "ADVAPI32.DLL",0,13`
> `bb('A') bb('D') bb('V') bb('A') bb('P') bb('I') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)`
> //`db "SHELL32.DLL",0,12`
> `bb('S') bb('H') bb('E') bb('L') bb('L') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)`
> `byte_type(0)`
> `\_p\_szCaption:`
> `bb('A') bb('s') bb('h') bb('k') bb('b') bb('i') bb('z') bb(0)`
> `\_p\_szText:`
> `bb('T') bb('h') bb('i') bb('s') bb(' ') bb('E') bb('x') bb('a') bb('m') bb('p') bb('l')`
> `bb('e') bb(' ') bb('s') bb('h') bb('o') bb('w') bb('s') bb(' ') bb('h') bb('o') bb('w')`
> `bb(' ') bb('A') bb('P') bb('I') bb(' ') bb('r') bb('e') bb('d') bb('i') bb('r') bb('e')`
> `bb('c') bb('t') bb('i') bb('o') bb('n') bb(' ') bb('t') bb('e') bb('c') bb('h') bb('n')`
> `bb('i') bb('q') bb('u') bb('e') bb(' ') bb('w') bb('o') bb('r') bb('k') bb('s') bb('.')`
> `bb(0)`
> `\_p\_dwThunk: ` `dword_type(0xCCCCCCCC)`
> `\_p\_dwHintName: ` `dword_type(0xCCCCCCCC)`
> `\_p\_ptempbuffer: ` `dword_type(0xCCCCCCCC)`
> `//----------------------------------`
> `dword_type(DYN_LOADER_END_MAGIC)`
> `//----------------------------------`
> `}`
> `}`
> `//----------------------------------------------------------------`

```markdown
#BeginLibraryItem "/Library/copyright.lbi" CLASS="p9z"
```

|
* * *
|
| --- |
|
|  ||  ||  ©2000-2008 [**PEdiy.com**](http://www.pediy.com/) All rights reserved.
By PEDIY  |
| --- | --- | --- |
|

```markdown
#EndLibraryItem
```
