

# 菜鸟注释 PEMaker6 源代码（一）

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-648.htm`](https://www.pediy.com/kssd/pediy09/pediy09-648.htm)

> 这两天看了精华 8 的“向导入表中注入代码“这篇文章，对阅读 PEMaker6 源代码进行了部分注释，要不总是忘了哪一部分是什么功能，为了增加点发帖量，将注释过的几个类逐一整理后发布，老鸟就无需看了,也是为了自己加强学习。  
> 这篇介绍 CPELibrary 类的功能：找回并重建 PE 文件（具体可参照加壳技术->加壳软件的编写->向 PE 中注入代码)  
> 类定义：  
> class CPELibrary   
> {  
> private:  
>   //-----------------------------------------  
>   PCHAR          pMem;  
>   DWORD          dwFileSize;  
>   //-----------------------------------------  
> protected:  
>   //-----------------------------------------  
>   PIMAGE\_DOS\_HEADER    image\_dos\_header;  
>   PCHAR          pDosStub;  
>   DWORD          dwDosStubSize, dwDosStubOffset;  
>   PIMAGE\_NT\_HEADERS    image\_nt\_headers;  //指针  
>   PIMAGE\_SECTION\_HEADER  image\_section\_header\[MAX\_SECTION\_NUM\];  
>   PCHAR          image\_section\[MAX\_SECTION\_NUM\];  
>   /\*typedef struct \_IMAGE\_TLS\_DIRECTORY32 {  
>     DWORD   StartAddressOfRawData;  
>     DWORD   EndAddressOfRawData;  
>     PDWORD  AddressOfIndex;  
>     PIMAGE\_TLS\_CALLBACK \*AddressOfCallBacks;  
>     DWORD   SizeOfZeroFill;  
>     DWORD   Characteristics;  
>   } IMAGE\_TLS\_DIRECTORY32\*/  
>   PIMAGE\_TLS\_DIRECTORY32  image\_tls\_directory;  
>   //-----------------------------------------  
> protected:  
>   //-----------------------------------------  
>   DWORD PEAlign(DWORD dwTarNum,DWORD dwAlignTo);  
>   void AlignmentSections();  
>   //-----------------------------------------  
>   DWORD Offset2RVA(DWORD dwRO);  
>   DWORD RVA2Offset(DWORD dwRVA);  
>   //-----------------------------------------  
>   PIMAGE\_SECTION\_HEADER ImageRVA2Section(DWORD dwRVA);  
>   PIMAGE\_SECTION\_HEADER ImageOffset2Section(DWORD dwRO);  
>   //-----------------------------------------  
>   DWORD ImageOffset2SectionNum(DWORD dwRVA);  
>   PIMAGE\_SECTION\_HEADER AddNewSection(char\* szName,DWORD dwSize);  
>   //-----------------------------------------  
> public:  
>   //-----------------------------------------  
>   CPELibrary();  
>   ~CPELibrary();  
>   //-----------------------------------------  
>   void OpenFile(char\* FileName);  
>   void SaveFile(char\* FileName);    
>   //-----------------------------------------  
> };  
> CPP 文件：  
> CPELibrary::CPELibrary()  
> {    //构造 DOS 文件头  
>     image\_dos\_header=new (IMAGE\_DOS\_HEADER);  
>     dwDosStubSize=0;  
>     //构造 NT 文件头  
>     image\_nt\_headers=new (IMAGE\_NT\_HEADERS);  
>     //构造 20 个节表  
>     for(int i=0;i<MAX\_SECTION\_NUM;i++) image\_section\_header\[i\]=new (IMAGE\_SECTION\_HEADER);  
>     image\_tls\_directory=NULL;  
> }  
> //----------------------------------------------------------------  
> CPELibrary::~CPELibrary()  
> {  
>     delete \[\]image\_dos\_header;  
>     dwDosStubSize=0;  
>     delete \[\]image\_nt\_headers;  
>     for(int i=0;i<MAX\_SECTION\_NUM;i++) delete \[\]image\_section\_header\[i\];  
>     if(image\_tls\_directory!=NULL) delete image\_tls\_directory;  
> }  
> //================================================================  
> //----------------------------------------------------------------  
> // returns aligned value  
> //对齐  
> DWORD CPELibrary::PEAlign(DWORD dwTarNum,DWORD dwAlignTo)  
> {      
>     return(((dwTarNum+dwAlignTo-1)/dwAlignTo)\*dwAlignTo);  
> }  
> //----------------------------------------------------------------  
> void CPELibrary::AlignmentSections()  
> {    //根据节的数量循环  
>     for(int i=0;i<image\_nt\_headers->FileHeader.NumberOfSections;i++)  
>     {  
>         //节中数据的 RVA。  
>         image\_section\_header\[i\]->VirtualAddress=  
>             PEAlign(image\_section\_header\[i\]->VirtualAddress,  
>             image\_nt\_headers->OptionalHeader.SectionAlignment);  
>         //对齐 Misc.VirtualSize  
>         image\_section\_header\[i\]->Misc.VirtualSize=  
>             PEAlign(image\_section\_header\[i\]->Misc.VirtualSize,  
>             image\_nt\_headers->OptionalHeader.SectionAlignment);  
>         //对齐 PointerToRawData  
>         image\_section\_header\[i\]->PointerToRawData=  
>             PEAlign(image\_section\_header\[i\]->PointerToRawData,  
>             image\_nt\_headers->OptionalHeader.FileAlignment);  
>         //对齐 SizeOfRawData  
>         image\_section\_header\[i\]->SizeOfRawData=  
>             PEAlign(image\_section\_header\[i\]->SizeOfRawData,  
>             image\_nt\_headers->OptionalHeader.FileAlignment);  
>     }  
>     //是个 RVA，重建映象文件基址  
>     image\_nt\_headers->OptionalHeader.SizeOfImage=image\_section\_header\[i-1\]->VirtualAddress+  
>         image\_section\_header\[i-1\]->Misc.VirtualSize;  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG\].VirtualAddress=0;  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG\].Size=0;  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT\].VirtualAddress=0;  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT\].Size=0;  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IAT\].VirtualAddress=0;  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IAT\].Size=0;  
> }  
> //================================================================  
> //----------------------------------------------------------------  
> // calulates the Offset from a RVA  
> // Base    - base of the MMF  
> // dwRVA - the RVA to calculate  
> // returns 0 if an error occurred else the calculated Offset will be returned  
> DWORD CPELibrary::RVA2Offset(DWORD dwRVA)  
> {  
>     DWORD \_offset;  
>     PIMAGE\_SECTION\_HEADER section;  
>     section=ImageRVA2Section(dwRVA);//ImageRvaToSection(pimage\_nt\_headers,Base,dwRVA);  
>     if(section==NULL)  
>     {  
>         return(0);  
>     }  
>     //文件偏移  
>     \_offset=(dwRVA-section->VirtualAddress)+section->PointerToRawData;  
>     return(\_offset);  
> }  
> //----------------------------------------------------------------  
> // calulates the RVA from a Offset  
> // Base    - base of the MMF  
> // dwRO - the Offset to calculate  
> // returns 0 if an error occurred else the calculated Offset will be returned  
> DWORD CPELibrary::Offset2RVA(DWORD dwRO)  
> {  
>     PIMAGE\_SECTION\_HEADER section;  
>     section=ImageOffset2Section(dwRO);  
>     if(section==NULL)  
>     {  
>         return(0);  
>     }  
>     //返回 RVA  
>     return((dwRO-section->PointerToRawData)+section->VirtualAddress);  
> }  
> //================================================================  
> //----------------------------------------------------------------  
> PIMAGE\_SECTION\_HEADER CPELibrary::ImageRVA2Section(DWORD dwRVA)  
> {  
>     int i;  
>     for(i=0;i<image\_nt\_headers->FileHeader.NumberOfSections;i++)  
>     {    //确定包含在哪一个节中  
>         if((dwRVA>=image\_section\_header\[i\]->VirtualAddress) && (dwRVA<=(image\_section\_header\[i\]->VirtualAddress+image\_section\_header\[i\]->SizeOfRawData)))  
>         {    //返回该节  
>             return ((PIMAGE\_SECTION\_HEADER)image\_section\_header\[i\]);  
>         }  
>     }  
>     return(NULL);  
> }  
>   
> //----------------------------------------------------------------  
> //The ImageOffset2Section function locates a Off Set address (RO)   
> //within the image header of a file that is mapped as a file and  
> //returns a pointer to the section table entry for that virtual   
> //address.  
> PIMAGE\_SECTION\_HEADER CPELibrary::ImageOffset2Section(DWORD dwRO)  
> {  
>     for(int i=0;i<image\_nt\_headers->FileHeader.NumberOfSections;i++)  
>     {  
>         //确定包含在哪一个节中  
>         if((dwRO>=image\_section\_header\[i\]->PointerToRawData) && (dwRO<(image\_section\_header\[i\]->PointerToRawData+image\_section\_header\[i\]->SizeOfRawData)))  
>         {    //返回该节  
>             return ((PIMAGE\_SECTION\_HEADER)image\_section\_header\[i\]);  
>         }  
>     }  
>     return(NULL);  
> }  
> //================================================================  
> //----------------------------------------------------------------  
> // retrieve Enrty Point Section Number  
> // Base    - base of the MMF  
> // dwRVA - the RVA to calculate  
> // returns -1 if an error occurred else the calculated Offset will be returned  
> //由文件偏移到节号  
> DWORD CPELibrary::ImageOffset2SectionNum(DWORD dwRO)  
> {  
>     for(int i=0;i<image\_nt\_headers->FileHeader.NumberOfSections;i++)  
>     {  
>         if((dwRO>=image\_section\_header\[i\]->PointerToRawData) && (dwRO<(image\_section\_header\[i\]->PointerToRawData+image\_section\_header\[i\]->SizeOfRawData)))  
>         {  
>             return (i);  
>         }  
>     }  
>     return(-1);  
> }  
> //增加新节----------------------------------------------------------------  
> PIMAGE\_SECTION\_HEADER CPELibrary::AddNewSection(char\* szName,DWORD dwSize)  
> {  
>     DWORD roffset,rsize,voffset,vsize;  
>     //最后一节  
>     int i=image\_nt\_headers->FileHeader.NumberOfSections;  
>     rsize=PEAlign(dwSize,  
>                 image\_nt\_headers->OptionalHeader.FileAlignment);  
>     vsize=PEAlign(rsize,  
>                 image\_nt\_headers->OptionalHeader.SectionAlignment);  
>     roffset=PEAlign(image\_section\_header\[i-1\]->PointerToRawData+image\_section\_header\[i-1\]->SizeOfRawData,  
>                 image\_nt\_headers->OptionalHeader.FileAlignment);  
>     voffset=PEAlign(image\_section\_header\[i-1\]->VirtualAddress+image\_section\_header\[i-1\]->Misc.VirtualSize,  
>                 image\_nt\_headers->OptionalHeader.SectionAlignment);  
>     //初始化新节表  
>     memset(image\_section\_header\[i\],0,(size\_t)sizeof(IMAGE\_SECTION\_HEADER));  
>     //通过计算上一节得到  
>     image\_section\_header\[i\]->PointerToRawData=roffset;  
>     image\_section\_header\[i\]->VirtualAddress=voffset;  
>     //通过传递的参数得到  
>     image\_section\_header\[i\]->SizeOfRawData=rsize;  
>     image\_section\_header\[i\]->Misc.VirtualSize=vsize;  
>     //属性  
>     image\_section\_header\[i\]->Characteristics=0xC0000040;  
>     //新节名称  
>     memcpy(image\_section\_header\[i\]->Name,szName,(size\_t)strlen(szName));  
>     //申请一个节的空间  
>     image\_section\[i\]=(char\*)GlobalAlloc(GMEM\_FIXED | GMEM\_ZEROINIT,rsize);  
>     //更新文件头  
>     image\_nt\_headers->FileHeader.NumberOfSections++;  
>     //返回新节  
>     return (PIMAGE\_SECTION\_HEADER)image\_section\_header\[i\];  
> }  
> //================================================================  
> //----------------------------------------------------------------  
> void CPELibrary::OpenFile(char\* FileName)  
> {  
>     DWORD    dwBytesRead        = 0;  
>     HANDLE    hFile= NULL;  
>     DWORD SectionNum;  
>     DWORD i;  
>     DWORD dwRO\_first\_section;  
>     //基类成员  
>     pMem=NULL;  
>     //----------------------------------------  
>     hFile=CreateFile(FileName,  
>                      GENERIC\_READ,  
>                      FILE\_SHARE\_WRITE | FILE\_SHARE\_READ,  
>                      NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,NULL);  
>     if(hFile==INVALID\_HANDLE\_VALUE)  
>     {  
>         ShowErr(FileErr);  
>         return;  
>     }  
>     dwFileSize=GetFileSize(hFile,0);  
>     if(dwFileSize == 0)  
>     {  
>         CloseHandle(hFile);  
>         ShowErr(FsizeErr);  
>         return;  
>     }  
>     //不用内存映射文件,奇怪？  
>     pMem=(char\*)GlobalAlloc(GMEM\_FIXED | GMEM\_ZEROINIT,dwFileSize);  
>     if(pMem == NULL)  
>     {  
>         CloseHandle(hFile);  
>         ShowErr(MemErr);  
>         return;  
>     }  
>     //读入文件  
>     ReadFile(hFile,pMem,dwFileSize,&dwBytesRead,NULL);  
>     //关闭句柄  
>     CloseHandle(hFile);  
>     //----------------------------------------  
>     //读出 DOS 文件头  
>     memcpy(image\_dos\_header,pMem,sizeof(IMAGE\_DOS\_HEADER));  
>     //MS-DOS Stub Program(位于 e\_lfanew 和 Signature 之间)  
>     dwDosStubSize=image\_dos\_header->e\_lfanew-sizeof(IMAGE\_DOS\_HEADER);  
>     dwDosStubOffset=sizeof(IMAGE\_DOS\_HEADER);  
>     pDosStub=new CHAR\[dwDosStubSize\];  
>     if((dwDosStubSize&0x80000000)==0x00000000)  
>     {  
>         CopyMemory(pDosStub,pMem+dwDosStubOffset,dwDosStubSize);  
>     }  
>     //读出 NT 文件头  
>     memcpy(image\_nt\_headers,  
>                pMem+image\_dos\_header->e\_lfanew,  
>                sizeof(IMAGE\_NT\_HEADERS));  
>     //DOS 头大小+NT 头大小  
>     dwRO\_first\_section=image\_dos\_header->e\_lfanew+sizeof(IMAGE\_NT\_HEADERS);  
>     if(image\_dos\_header->e\_magic!=IMAGE\_DOS\_SIGNATURE)// MZ  
>     {  
>         ShowErr(PEErr);  
>         GlobalFree(pMem);  
>         return;  
>     }  
>     if(image\_nt\_headers->Signature!=IMAGE\_NT\_SIGNATURE)// PE00  
>     {  
>         ShowErr(PEErr);  
>         GlobalFree(pMem);  
>         return;  
>     }  
>     //----------------------------------------  
>     //节的数量  
>     SectionNum=image\_nt\_headers->FileHeader.NumberOfSections;  
>     //----------------------------------------  
>     for( i=0;i<SectionNum;i++)   
>     {  
>         //读取节表，储存节表的空间已经初始化了。  
>         CopyMemory(image\_section\_header\[i\],pMem+dwRO\_first\_section+i\*sizeof(IMAGE\_SECTION\_HEADER),  
>             sizeof(IMAGE\_SECTION\_HEADER));  
>     }  
>     //----------------------------------------  
>     for(i=0;i<SectionNum;i++)  
>     {  
>             //为每个节数据分配空间，大小为每个节在文件中的大小  
>             image\_section\[i\]=(char\*)GlobalAlloc(GMEM\_FIXED | GMEM\_ZEROINIT,  
>                 PEAlign(image\_section\_header\[i\]->SizeOfRawData,  
>                 image\_nt\_headers->OptionalHeader.FileAlignment));  
>             //循环读出每个节，按照文件对齐  
>             CopyMemory(image\_section\[i\],  
>                     pMem+image\_section\_header\[i\]->PointerToRawData,  
>                     image\_section\_header\[i\]->SizeOfRawData);  
>     }  
>     //是否存在 TLS 节----------------------------------------  
>     if(image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS\].VirtualAddress!=0)  
>     {  
>         //分配空间  
>         image\_tls\_directory = new (IMAGE\_TLS\_DIRECTORY32);  
>         //文件偏移  
>         DWORD dwOffset=RVA2Offset(image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS\].VirtualAddress);  
>         //复制  
>         memcpy(image\_tls\_directory,  
>                 pMem+dwOffset,  
>                 sizeof(IMAGE\_TLS\_DIRECTORY32));          
>     }  
>     //----------------------------------------  
>     GlobalFree(pMem);  
> }  
> //----------------------------------------------------------------  
> void CPELibrary::SaveFile(char\* FileName)  
> {  
>     DWORD    dwBytesWritten    = 0;  
>     DWORD i;  
>     DWORD dwRO\_first\_section;  
>     DWORD SectionNum;  
>     HANDLE    hFile= NULL;  
>     pMem=NULL;  
>     //----------------------------------------  
>     hFile=CreateFile(FileName,  
>                      GENERIC\_WRITE,  
>                      FILE\_SHARE\_WRITE | FILE\_SHARE\_READ,  
>                      NULL,CREATE\_NEW,FILE\_ATTRIBUTE\_NORMAL,NULL);  
>     if(hFile==INVALID\_HANDLE\_VALUE)  
>     {  
>         hFile=CreateFile(FileName,  
>                      GENERIC\_WRITE,  
>                      FILE\_SHARE\_WRITE | FILE\_SHARE\_READ,  
>                      NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,NULL);  
>         if(hFile==INVALID\_HANDLE\_VALUE)  
>         {  
>             ShowErr(FileErr);  
>             return;  
>         }  
>     }  
>     //对齐----------------------------------------  
>     AlignmentSections();  
>     //----------------------------------------  
>     i=image\_nt\_headers->FileHeader.NumberOfSections;  
>     //得到新文件的大小  
>     dwFileSize=image\_section\_header\[i-1\]->PointerToRawData+  
>         image\_section\_header\[i-1\]->SizeOfRawData;  
>     //申请内存  
>     pMem=(char\*)GlobalAlloc(GMEM\_FIXED | GMEM\_ZEROINIT,dwFileSize);  
>     if(pMem == NULL)  
>     {  
>         CloseHandle(hFile);  
>         ShowErr(MemErr);  
>         return;  
>     }  
>     //复制 DOS 头----------------------------------------  
>     memcpy(pMem,image\_dos\_header,sizeof(IMAGE\_DOS\_HEADER));  
>     //MS-DOS Stub Program  
>     if((dwDosStubSize&0x80000000)==0x00000000)  
>     {  
>         memcpy(pMem+dwDosStubOffset,pDosStub,dwDosStubSize);  
>     }  
>     //复制 NT 头  
>     memcpy(pMem+image\_dos\_header->e\_lfanew,  
>         image\_nt\_headers,  
>         sizeof(IMAGE\_NT\_HEADERS));  
>     //节表的偏移  
>     dwRO\_first\_section=image\_dos\_header->e\_lfanew+sizeof(IMAGE\_NT\_HEADERS);  
>     //节的数量  
>     SectionNum=image\_nt\_headers->FileHeader.NumberOfSections;  
>     //复制节表----------------------------------------  
>     for( i=0;i<SectionNum;i++)   
>     {  
>         CopyMemory(pMem+dwRO\_first\_section+i\*sizeof(IMAGE\_SECTION\_HEADER),  
>             image\_section\_header\[i\],  
>             sizeof(IMAGE\_SECTION\_HEADER));  
>     }  
>     //复制节数据----------------------------------------  
>     for(i=0;i<SectionNum;i++)  
>     {  
>         CopyMemory(pMem+image\_section\_header\[i\]->PointerToRawData,  
>                 image\_section\[i\],  
>                 image\_section\_header\[i\]->SizeOfRawData);  
>     }  
>     // ----- WRITE FILE MEMORY TO DISK -----  
>     //设置文件指针  
>     SetFilePointer(hFile,0,NULL,FILE\_BEGIN);  
>     //写内存到磁盘  
>     WriteFile(hFile,pMem,dwFileSize,&dwBytesWritten,NULL);  
>       
>     // ------ FORCE CALCULATED FILE SIZE ------  
>     //强制计算文件大小  
>     SetFilePointer(hFile,dwFileSize,NULL,FILE\_BEGIN);  
>     SetEndOfFile(hFile);  
>     CloseHandle(hFile);  
>     //----------------------------------------  
>     GlobalFree(pMem);  
> } 

> 今天注释的是 CITMaker 类，其中应该有不少的错误，请大家附加自己的思考吧  
> 这个类的主要作用是根据下面的字符串计算所需的导入表大小以及构建导入表  
> PEMaker6 下载地址：http://www.mydatabus.com/public/panzhibin/pemaker6.zip  
> static const char \*sz\_IT\_EXE\_strings\[\]=  
> {  
>     "Kernel32.dll",  
>     "LoadLibraryA",  
>     "GetProcAddress",  
>     0,  
>     0,  
> };  
>   
> const char \*sz\_IT\_OCX\_strings\[\]=  
> {  
>     "Kernel32.dll",  
>     "LoadLibraryA",  
>     "GetProcAddress",  
>     "GetModuleHandleA",  
>     0,  
>     "User32.dll",  
>     "GetKeyboardType",  
>     "WindowFromPoint",  
>     0,  
>     "AdvApi32.dll",  
>     "RegQueryValueExA",  
>     "RegSetValueExA",  
>     "StartServiceA",  
>     0,  
>     "Oleaut32.dll",  
>     "SysFreeString",  
>     "CreateErrorInfo",  
>     "SafeArrayPtrOfIndex",  
>     0,  
>     "Gdi32.dll",  
>     "UnrealizeObject",  
>     0,  
>     "Ole32.dll",  
>     "CreateStreamOnHGlobal",  
>     "IsEqualGUID",  
>     0,  
>     "ComCtl32.dll",  
>     "ImageList\_SetIconSize",  
>     0,  
>     0,  
> };  
> //----------------------------------------------------------------  
> using namespace std;  
>   
> //----------------------------------------------------------------  
> typedef struct  
> {  
>     CHAR szFunction\[32\];  
> }t\_IMAGE\_THUNK, \*pt\_IMAGE\_THUNK;  
> //----------------------------------------------------------------  
> typedef struct  
> {  
>     CHAR szLibrary\[32\];  
>     //链表  
>     list <t\_IMAGE\_THUNK> ThunksList;  
>     //链表指针  
>     list <t\_IMAGE\_THUNK>::iterator ThunkIter;  
>     CHAR szFunction\[32\];  
> }t\_IMAGE\_IMPORT\_TABLE, \*pt\_IMAGE\_IMPORT\_TABLE;  
> //----------------------------------------------------------------  
> static list <t\_IMAGE\_IMPORT\_TABLE> ImportTable;  
> static list <t\_IMAGE\_IMPORT\_TABLE>::iterator ImportIter;  
> //================================================================  
> CITMaker::CITMaker(int iType)  
> {  
>     //初始化  
>     Initialization(iType);  
>     //计算出所需的导入表大小  
>     dwSize=Get\_IT\_Size();  
>     //申请空间  
>     pMem=new CHAR\[dwSize\];  
> }  
>   
> CITMaker::~CITMaker()  
> {  
>     ImportTable.clear();  
>     delete \[\] pMem;  
> }  
> //----------------------------------------------------------------  
> // This function makes the dll name strings, saves them to the linked list  
> void CITMaker::Initialization(int iType)  
> {  
>     int i;  
>     PCHAR \*sz\_IT\_strings;  
>     t\_IMAGE\_IMPORT\_TABLE    imageimport;  
>     t\_IMAGE\_THUNK            imagethunk;  
>     switch(iType)  
>     {  
>     case IMPORT\_TABLE\_EXE:  
>         sz\_IT\_strings=(PCHAR \*)sz\_IT\_EXE\_strings;  
>         break;  
>     case IMPORT\_TABLE\_OCX:  
>         sz\_IT\_strings=(PCHAR \*)sz\_IT\_OCX\_strings;  
>         break;  
>     }  
>     //静态全局变量--------------------------------------------  
>     ImportTable.clear();  
>     i=0;  
>     do  
>     {  
>         //复制库名  
>         strcpy(imageimport.szLibrary,sz\_IT\_strings\[i\]);  
>         //清空 imageimport--Thunk 列表  
>         imageimport.ThunksList.clear();  
>         do  
>         {  
>   
>             i++;  
>             //取 API 函数名  
>             if(sz\_IT\_strings\[i\]!=0)  
>             {  
>                 //向 imagethunk---复制函数名  
>                 strcpy(imagethunk.szFunction,sz\_IT\_strings\[i\]);  
>                 //放入链表  
>                 imageimport.ThunksList.push\_back(imagethunk);  
>             }  
>         }while(sz\_IT\_strings\[i\]!=0);  //循环取得函数名  
>         ImportTable.push\_back(imageimport);    //入 ImportTable 链表  
>         i++;  
>     }  
>     while(sz\_IT\_strings\[i\]!=0);  
>     //其实 EXE 文件只有一个表被建立--------------------------------------------  
> }  
> //----------------------------------------------------------------  
> // This function calculated zise of Import Table.  
> //计算导入表大小  
> //引入表实际上是一个 IMAGE\_IMPORT\_DESCRIPTOR 结构数组。  
> //每个结构包含 PE 文件引入函数的一个相关 DLL 的信息  
> //比如，如果该 PE 文件从 10 个不同的 DLL 中引入函数，那么这个数组就有 10 个成员。  
> //该数组以一个全 0 的成员结尾。  
> //这里引入另一篇文章的自建导入表的结构，借鉴一下更直观，不一定准确：  
> /\*align 4  
> 　　v\_ImportA 　dd Ker\_API-v\_ImportA 　　　　　;IMAGE\_THUNK\_DATA 数组指针  
> 　　v\_TimeDateA dd 0 　　　　　　　　　　　　　;文件建立时间  
> 　　v\_ForChainA dd 0 　　　　　　　　　　　　　;0  
> 　　v\_DllNameA　dd KerName-v\_ImportA 　　　　　;DLL 名称  
> 　　v\_FThunkA 　dd vGetprocAddress-v\_ImportA 　;IMAGE\_THUNK\_DATA 数组指针  
>    
> 　　v\_ImportB 　dd Use\_API-v\_ImportA 　　　　　;IMAGE\_THUNK\_DATA 数组指针  
> 　　v\_TimeDateB dd 0 　　　　　　　　　　　　　;文件建立时间  
> 　　v\_ForChainB dd 0 　　　　　　　　　　　　　;0  
> 　　v\_DllNameB　dd UserName-v\_ImportA 　　　　 ;DLL 名称  
> 　　v\_FThunkB 　dd vMessageBoxA-v\_ImportA 　　 ;IMAGE\_THUNK\_DATA 数组指针  
>    
> 　　　　　　　　dd 20 dup (0) 　　　　　　　　 ;引入表结束符  
>    
> 　　KerName db 'KERNEL32.DLL',0  
> 　　Ker\_API dd KAPI\_A-v\_ImportA  
> 　　　　　　dd KAPI\_B-v\_ImportA  
> 　　　　　　dd KAPI\_C-v\_ImportA  
> 　　　　　　dd KAPI\_D-v\_ImportA  
> 　　　　　　dd KAPI\_E-v\_ImportA  
> 　　　　　　dd KAPI\_F-v\_ImportA  
> 　　　　　　dd KAPI\_G-v\_ImportA  
> 　　　　　　dd KAPI\_H-v\_ImportA  
> 　　　　　　dd KAPI\_I-v\_ImportA  
> 　　　　　　dd KAPI\_J-v\_ImportA  
> 　　　　　　dd KAPI\_K-v\_ImportA  
> 　　　　　　dd KAPI\_L-v\_ImportA  
> 　　　　　　dd KAPI\_M-v\_ImportA  
> 　　　　　　dd KAPI\_N-v\_ImportA  
> 　　　　　　dd KAPI\_O-v\_ImportA  
> 　　　　　　dd KAPI\_P-v\_ImportA  
> 　　　　　　dd KAPI\_Q-v\_ImportA  
> 　　　　　　dd 0  
>    
>    
> 　　UserName db 'USER32.DLL',0  
> 　　Use\_API dd UAPI\_A-v\_ImportA  
> 　　　　　　dd 0  
>    
> 　　vGetprocAddress 　　dd 0  
> 　　vGetModuleHandleA 　dd 0  
> 　　vLoadLibraryA 　　　dd 0  
> 　　vExitprocess 　　　 dd 0  
> 　　vCreateFileA 　　　 dd 0  
> 　　vCreateFileMappingA dd 0  
> 　　vGetTempPathA 　　　dd 0  
> 　　vGetTempFileNameA 　dd 0  
> 　　vlstrlen 　　　　　 dd 0  
> 　　vMapViewOfFile 　　 dd 0  
> 　　vWriteFile 　　　　 dd 0  
> 　　vUnmapViewOfFile 　 dd 0  
> 　　vCloseHandle 　　　 dd 0  
> 　　vCopyFileA 　　　　 dd 0  
> 　　vGetModuleFileNameA dd 0  
> 　　vDeleteFileA 　　　 dd 0  
> 　　vWinExec 　　　　　 dd 0  
>    
> 　　vMessageBoxA dd 0  
> 　　　　　　　　　dd 0  
>    
> 　　KAPI\_A db 0,0,'GetprocAddress',0  
> 　　KAPI\_B db 0,0,'GetModuleHandleA',0  
> 　　KAPI\_C db 0,0,'LoadLibraryA',0  
> 　　KAPI\_D db 0,0,'Exitprocess',0  
> 　　KAPI\_E db 0,0,'CreateFileA',0  
> 　　KAPI\_F db 0,0,'CreateFileMappingA',0  
> 　　KAPI\_G db 0,0,'GetTempPathA',0  
> 　　KAPI\_H db 0,0,'GetTempFileNameA',0  
> 　　KAPI\_I db 0,0,'lstrlen',0  
> 　　KAPI\_J db 0,0,'MapViewOfFile',0  
> 　　KAPI\_K db 0,0,'WriteFile',0  
> 　　KAPI\_L db 0,0,'UnmapViewOfFile',0  
> 　　KAPI\_M db 0,0,'CloseHandle',0  
> 　　KAPI\_N db 0,0,'CopyFileA',0  
> 　　KAPI\_O db 0,0,'GetModuleFileNameA',0  
> 　　KAPI\_P db 0,0,'DeleteFileA',0  
> 　　KAPI\_Q db 0,0,'WinExec',0  
>    
> 　　UAPI\_A db 0,0,'MessageBoxA',0  
>    
>    
> 　; 根据引入表的格式，自建的引入表。  
> 　;------------------------------------(上面的)--  
> 　align 4  
> \*/  
> DWORD CITMaker::Get\_IT\_Size()  
> {  
>     DWORD dwDLLNum=0;  
>     DWORD dwFunNum=0;  
>     DWORD dwszDLLSize=0;  
>     DWORD dwszFuncSize=0;  
>     DWORD dwImportSize=0;  
>     t\_IMAGE\_IMPORT\_TABLE    imageimport;  
>     t\_IMAGE\_THUNK            imagethunk;  
>     //ImportIter 是全局变量  
>     for(ImportIter=ImportTable.begin();ImportIter!=ImportTable.end();ImportIter++)  
>     {  
>         imageimport=\*ImportIter;  
>         //注意库名长度计算，+1 就是结尾的 0  
>         dwszDLLSize=dwszDLLSize+strlen(imageimport.szLibrary)+1;  
>         //遍历 THUnk 链表  
>         for(imageimport.ThunkIter=imageimport.ThunksList.begin();  
>             imageimport.ThunkIter!=imageimport.ThunksList.end();  
>             imageimport.ThunkIter++)  
>         {  
>             imagethunk=\*imageimport.ThunkIter;  
>             //注意长度计算，括号中为原始 THUNK 的结构  
>             //格式：0,0,函数名字符串,0  
>             dwszFuncSize=dwszFuncSize+(2+strlen(imagethunk.szFunction)+1);  
>             //函数计数  
>             dwFunNum++;  
>         }  
>         //以 0 结尾  
>         dwFunNum++;  
>         //库计数  
>         dwDLLNum++;  
>     }  
>     //以空 IMAGE\_IMPORT\_DESCRIPTOR 结构结尾  
>     dwDLLNum++;  
>     //每个 DLL 需要一个导入表(20 字节)+(API 函数数量)\*(DWORD)+(所有 DLL 字符串长度)+(所有函数名字符串长度)  
>     //这个地方我计算的不是很明白，应该有两个 RVA 数组，一个是函数名 RVA 数组，一个是函数地址 RVA 数组  
>     //有知道的朋友请指出一下。  
>     dwImportSize=dwDLLNum\*20+dwFunNum\*4+dwszDLLSize+dwszFuncSize;  
>     return(dwImportSize);  
> }  
> //----------------------------------------------------------------  
> //----------------------------------------------------------------  
> // This function build the dll name strings, saves the ImageImportDescriptors to the loader data.  
> //建立导入表  
> void CITMaker::Build(DWORD dwRVA)  
> {  
>     //是个 RVA  
>     DWORD                    pITBaseRVA=dwRVA;  
>     DWORD                    temp;  
>     DWORD                    dwDLLNum, dwDLLName, dwDLLFirst, dwszDLLSize;  
>     DWORD                    dwIIDNum, dwFunNum, dwFunFirst, dwszFuncSize;  
>     DWORD                    dwFirstThunk, dwImportSize;  
>     //局部变量  
>     t\_IMAGE\_IMPORT\_TABLE    imageimport;  
>     t\_IMAGE\_THUNK            imagethunk;  
>     //真正导入表结构  
>     IMAGE\_IMPORT\_DESCRIPTOR import\_descriptor;// -> IID  
>     //初始化--------------------------------------------  
>     import\_descriptor.OriginalFirstThunk=0;  
>     import\_descriptor.TimeDateStamp=0;  
>     import\_descriptor.ForwarderChain=0;  
>     import\_descriptor.Name=0;  
>     import\_descriptor.FirstThunk=0;  
>     dwDLLNum=dwDLLName=dwDLLFirst=dwszDLLSize=0;  
>     dwIIDNum=dwFunNum=dwFunFirst=dwszFuncSize=0;  
>     dwFirstThunk=dwImportSize=0;  
>     //遍历总链表--------------------------------------------  
>     for(ImportIter=ImportTable.begin();ImportIter!=ImportTable.end();ImportIter++)  
>     {  
>         imageimport=\*ImportIter;  
>         dwszDLLSize=dwszDLLSize+strlen(imageimport.szLibrary)+1;  
>         for(imageimport.ThunkIter=imageimport.ThunksList.begin();  
>             imageimport.ThunkIter!=imageimport.ThunksList.end();  
>             imageimport.ThunkIter++)  
>         {  
>             imagethunk=\*imageimport.ThunkIter;  
>             dwszFuncSize=dwszFuncSize+2+strlen(imagethunk.szFunction)+1;  
>             dwFunNum++;  
>         }  
>         dwFunNum++;  
>         dwDLLNum++;  
>     }  
>     dwDLLNum++;  
>     //重复得出大小，跟上面一样，为什么不调用函数计算呢，晕  
>     dwImportSize=dwDLLNum\*20+dwFunNum\*4+dwszDLLSize+dwszFuncSize;  
>     //pMem 已经初始化：在构造函数中，大小为所有引入表大小--------------------------------------------  
>     FillMemory(pMem,dwImportSize,0x00);  
>     //一些偏移量  
>     dwFirstThunk=dwDLLNum\*20;  
>     dwDLLFirst=dwDLLNum\*20+dwFunNum\*4;  
>     dwFunFirst=dwDLLNum\*20+dwFunNum\*4+dwszDLLSize;  
>     //pITBaseRVA  
>     //--------------------------------------------  
>     for(ImportIter=ImportTable.begin();ImportIter!=ImportTable.end();ImportIter++)  
>     {  
>         imageimport=\*ImportIter;  
>         //pITBaseRVA 为传递的参数  
>         //Name:含有指向 DLL 名字的 RVA  
>         //越过之前的所有 IMAGE\_IMPORT\_DESCRIPTOR 结构+所有 API 地址数组而到达指向 DLL 名字的 RVA  
>         import\_descriptor.Name=pITBaseRVA+dwDLLFirst;  
>         //越过之前的所有 IMAGE\_IMPORT\_DESCRIPTOR 结构得到 FirstThunk 的 RVA  
>         import\_descriptor.FirstThunk=pITBaseRVA+dwFirstThunk;  
>         //复制一系列 IMAGE\_IMPORT\_DESCRIPTOR  
>         memcpy(pMem+dwIIDNum\*sizeof(IMAGE\_IMPORT\_DESCRIPTOR),  
>                    &import\_descriptor,  
>                    sizeof(IMAGE\_IMPORT\_DESCRIPTOR));  
>         //复制动态库名称，注意位置 dwDLLFirst=dwDLLNum\*20+dwFunNum\*4！  
>         memcpy(pMem+dwDLLFirst,  
>                    imageimport.szLibrary,  
>                    strlen(imageimport.szLibrary)+1);  
>         //--------------------------------------------  
>         for(imageimport.ThunkIter=imageimport.ThunksList.begin();  
>             imageimport.ThunkIter!=imageimport.ThunksList.end();  
>             imageimport.ThunkIter++)  
>         {  
>             //函数名称链  
>             imagethunk=\*imageimport.ThunkIter;  
>             //计算函数名称字符串 RVA  
>             temp=pITBaseRVA+dwFunFirst;  
>             //复制的是一个 RVA,包含函数名数组 RVA 的 RVA 数组  
>             //dwFirstThunk=dwDLLNum\*20  
>             memcpy(pMem+dwFirstThunk,  
>                        &temp,  
>                        4);  
>             //THUNK 指向的 RVA 数组所指向的函数名数组  
>             //dwFunFirst=dwDLLNum\*20+dwFunNum\*4+dwszDLLSize  
>             memcpy(pMem+dwFunFirst+2,  
>                        imagethunk.szFunction,  
>                        strlen(imagethunk.szFunction)+1);  
>             //指向下一个位置  
>             dwFunFirst=dwFunFirst+2+strlen(imagethunk.szFunction)+1;  
>             dwFirstThunk=dwFirstThunk+4;  
>         }  
>         //--------------------------------------------  
>         temp=0;  
>         //以零结尾  
>         memcpy(pMem+dwFirstThunk,  
>                        &temp,  
>                        4);  
>         //指向下一个位置  
>         dwFirstThunk=dwFirstThunk+4;  
>         dwDLLFirst=dwDLLFirst+strlen(imageimport.szLibrary)+1;  
>         //表的数量加一  
>         dwIIDNum++;  
>     }  
>     //莫非是最后一个表，以空表结束--------------------------------------------  
>     import\_descriptor.Name=0;  
>     import\_descriptor.FirstThunk=0;  
>     //--------------------------------------------  
>     memcpy(pMem+dwIIDNum\*sizeof(IMAGE\_IMPORT\_DESCRIPTOR),  
>                &import\_descriptor,  
>                sizeof(IMAGE\_IMPORT\_DESCRIPTOR));  
> } 

标 题: 菜鸟注释 PEMaker6 源代码（三）  
作 者: 火影  
时 间: 2007-09-19 00:09    
*   链 接：[`bbs.pediy.com/showthread.php?t=51917`](http://bbs.pediy.com/showthread.php?t=51917)   
    详细信息:  

> 今天大概看了一下精华 8《向 PE 中注入代码》的译文，之前没有仔细看过，因为发现看过之后对源代码还是不是很明白，所以就没有参照这篇译文进行注释，大多都是自己理解+网上搜索，今天发现很多地方有失误。![](img/26c3a0df6b585d53c82b982893c84a26.jpg "伤心")   
> 大框的地方一定要参照译文，不要被我蒙蔽了。![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")   
> 继续注释 CPECryptor 类,如下：  
> //在 pecrypt.cpp 里，我提供了另外一个类——CPECryptor，用它组建新区段的数据。  
> //然而，新区段的数据是被 loader.cpp 里的 DynLoader()（在 DynLoader Step 1 中介绍的）创建的。  
> //因此，我们用 CPECryptor 类把这些数据（也有其它的数据）输入新区段。  
> //看完这段翻译基本没懂  
> const char \*szWindowsAPIs\[\]=  
> {  
>     "Kernel32.dll",  
>     "GetModuleHandleA",  
>     "VirtualProtect",  
>     "GetModuleFileNameA",  
>     "CreateFileA",  
>     "GlobalAlloc",  
>     "VirtualAlloc",  
>     "LoadLibraryA",  
>     "GetProcAddress",  
>     0,  
>     "User32.dll",  
>     "MessageBoxA",  
>     0,  
>     0,  
> };  
> //================================================================  
> //----------------------------------------------------------------  
> // Function: ReturnToBytePtr  
> // void\* FuncNum:   Function Name  
> // DWORD findstr:   String to find  
> // This code was written by FEUERRADER \[AHTeam\], Thanks him!  
> //在 loader.cpp 文件中的 DynLoader 函数里寻找指定字符串，并返回其地址  
> void\* CPECryptor::ReturnToBytePtr(void\* FuncName, DWORD findstr)  
> {  
>     void\* tmpd;  
>     \_\_asm  
>     {  
>         mov eax, FuncName  
>         jmp df  
> hjg:    inc eax  
> df:        mov ebx, \[eax\]  
>         cmp ebx, findstr  
>         jnz hjg  
>         mov tmpd, eax  
>     }  
>     return tmpd;  
> }  
> //================================================================  
> //此函数构建新节及导入表----------------------------------------------------------------  
> void CPECryptor::CryptFile(int(\_\_cdecl \*progress) (unsigned int, unsigned int))  
> {  
>     PCHAR ch\_temp;  
>     PIMAGE\_SECTION\_HEADER pimage\_section\_header;  
>     DWORD dwNewSectionSize;  
>     DWORD dwCodeSize;  
>     DWORD dwCodeOffset;  
>     //进度条控件----------------------------------------  
>     progress(0,0);  
>     //是否 DLL  
>     if((image\_nt\_headers->FileHeader.Characteristics&IMAGE\_FILE\_DLL)==IMAGE\_FILE\_DLL)  
>     {  
>         //构造 CITMaker 类  
>         ImportTableMaker = new CITMaker(IMPORT\_TABLE\_OCX);  
>     }  
>     else  
>     {  
>         ImportTableMaker = new CITMaker(IMPORT\_TABLE\_EXE);  
>     }  
>     //----------------------------------------  
>     //========================================  
>     //ch\_temp 指向 DYN\_LOADER\_START\_MAGIC 之后的地址  
>     ch\_temp=(PCHAR)DWORD(ReturnToBytePtr(DynLoader, DYN\_LOADER\_START\_MAGIC))+4;  
>     //计算 DynLoader 函数代码长度  
>     dwCodeSize=DWORD(ReturnToBytePtr(DynLoader, DYN\_LOADER\_END\_MAGIC))-DWORD(ch\_temp);  
>     //自定义导入表大小  
>     dwCodeOffset = ImportTableMaker->dwSize;  
>     //新节大小=DynLoader 函数代码长度+自定义导入表大小  
>     dwNewSectionSize = dwCodeSize + ImportTableMaker->dwSize;  
>     //分配空间  
>     pNewSection=new TCHAR\[dwNewSectionSize\];  
>     //复制 DynLoader 函数代码到偏移 dwCodeOffset 的位置之后  
>     memcpy(pNewSection+dwCodeOffset, ch\_temp, dwCodeSize);  
>     //========================================  
>     //----------------------------------------  
>     //添加新节，并返回新节指针  
>     pimage\_section\_header=AddNewSection(".xxx",dwNewSectionSize);  
>     //----------------------------------------  
>     //========================================  
>     //参数为新节在内存的偏移地址  
>     CopyData1(pimage\_section\_header->VirtualAddress);  
>     //参数同上，构建自定义导入表  
>     ImportTableMaker->Build(pimage\_section\_header->VirtualAddress);// build import table by the current virtual address  
>     //构建的导入表在新节数据前，注意相互位置  
>     memcpy(pNewSection, ImportTableMaker->pMem, ImportTableMaker->dwSize);  
>     //========================================  
>     //将新节数据复制到 image\_section(已经初始化)----------------------------------------  
>     memcpy(image\_section\[image\_nt\_headers->FileHeader.NumberOfSections-1\],  
>            pNewSection,  
>            dwNewSectionSize);  
>     //入口地址更改为新节的 DynLoader 函数代码处  
>     image\_nt\_headers->OptionalHeader.AddressOfEntryPoint=pimage\_section\_header->VirtualAddress + dwCodeOffset;  
>     //导入表地址更改为新导入表地址  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress=pimage\_section\_header->VirtualAddress;  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].Size=ImportTableMaker->dwSize;  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress=0;  
>     image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].Size=0;  
>     SetSectionsWritePermission();  
>     //----------------------------------------  
>     delete \[\]pNewSection;  
>     delete ImportTableMaker;  
>     //----------------------------------------  
>     progress(100,0);  
> }  
> //这个结构对应 DynLoader 函数里 DYN\_LOADER\_START\_DATA1 标志后面的东西，具体  
> //是自定义结构？请高手指点，本人水平到此，分析不出了  
> //译文：为了找到 OEP 的 virtual address，我们立即保存原始的 OEP 和 Image Base。  
> //我在 DynLoader()的尾部预留了一块空地来保存它们  
> typedef struct  
> {  
>     //保留  
>     DWORD dwReserved1;  
>     //文件类型  
>     DWORD dwFileType;  
>     //基地址  
>     DWORD dwImageBase;  
>     //原始入口点  
>     DWORD dwOrgEntryPoint;  
>     //导入表虚拟地址  
>     DWORD dwImportVirtualAddress;  
>     //重定位表虚拟地址及大小  
>     DWORD dwRelocationVirtualAddress;  
>     DWORD dwRelocationSize;  
>     //TLS  
>     IMAGE\_TLS\_DIRECTORY32 image\_tls\_directory;  
> }t\_DATA\_1,\*pt\_DATA\_1;  
> //此函数将实际数据填充到 DynLoader 函数指定位置里  
> void CPECryptor::CopyData1(DWORD dwVirtualAddress)  
> {  
>     int i, API\_num;  
>     PCHAR pData1;  
>     DWORD dwOffset;  
>     size\_t l;  
>     UCHAR temp;  
>     //为结构分配内存  
>     pt\_DATA\_1 pDataTable=new(t\_DATA\_1);  
>     //----------------------------------------  
>     //保留,0xcccccccc 应该没有意义  
>     pDataTable->dwReserved1=0xCCCCCCCC;  
>     //如果为 DLL，填入文件类型  
>     if((image\_nt\_headers->FileHeader.Characteristics&IMAGE\_FILE\_DLL)==IMAGE\_FILE\_DLL)  
>         pDataTable->dwFileType=IMPORT\_TABLE\_OCX;  
>     else  
>         pDataTable->dwFileType=IMPORT\_TABLE\_EXE;  
>     //填充镜像首地址  
>     pDataTable->dwImageBase=image\_nt\_headers->OptionalHeader.ImageBase;  
>     //填充入口点  
>     pDataTable->dwOrgEntryPoint=image\_nt\_headers->OptionalHeader.AddressOfEntryPoint;  
>     //填充导入表地址(RVA)  
>     pDataTable->dwImportVirtualAddress=image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress;  
>     if((image\_nt\_headers->FileHeader.Characteristics&IMAGE\_FILE\_DLL)==IMAGE\_FILE\_DLL)  
>     {  
>         //如果是 DLL，填充重定位表  
>         pDataTable->dwRelocationVirtualAddress=image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress;  
>         pDataTable->dwRelocationSize=image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].Size;  
>     }  
>     //前面已经将 DynLoader 函数读入我们申请的内存中----------------------------------------  
>     //故在 pNewSection 指向的内存中找到 DYN\_LOADER\_START\_DATA1 标志，返回地址  
>     pData1=(PCHAR)ReturnToBytePtr(pNewSection, DYN\_LOADER\_START\_DATA1);  
>     //看了好几个 TLS 了，是什么？  
>     //通过使用线程本地储存（TLS），一个程序能够执行多线程程序，  
>     //这样子的程序大多数是用 Borland 链接器：Delphi 和 C++, Builder.   
>     //当你包装一个 PE 文件时，你应该小心的清空 TLS，  
>     //否则，你的打包器就不支持 Borland Delphi 和 C++ Builder 链接的 EXE 文件。  
>     //修正可选头部 TLS 目录入口是很有必要的。  
>     if(image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS\].VirtualAddress!=0)  
>     {  
>         //image\_tls\_directory 定义在 pelib.cpp 中  
>         memcpy(&pDataTable->image\_tls\_directory,  
>                 image\_tls\_directory,  
>                 sizeof(IMAGE\_TLS\_DIRECTORY32));  
>         dwOffset=DWORD(pData1)-DWORD(pNewSection);  
>         //计算偏移  
>         dwOffset+=sizeof(t\_DATA\_1)-sizeof(IMAGE\_TLS\_DIRECTORY32);  
>         //更新文件头，指向新的 TLS 表  
>         //传递的参数在这里应用  
>         image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS\].VirtualAddress=dwVirtualAddress + dwOffset;  
>     }  
>     //替换为实际数值----------------------------------------  
>     memcpy(pData1,pDataTable,sizeof(t\_DATA\_1));  
>     dwOffset=sizeof(t\_DATA\_1);  
>     i=API\_num=0;  
>     temp=0;  
>     //循环填充实际数值  
>     do  
>     {  
>         l=strlen(szWindowsAPIs\[i\])+1;  
>         //复制动态库名  
>         memcpy(pData1+dwOffset,szWindowsAPIs\[i\],l);  
>         //偏移长度  
>         dwOffset+=l;  
>         do  
>         {  
>             i++;  
>             if(szWindowsAPIs\[i\]!=0)  
>             {  
>                 //复制下一个函数名  
>                 l=strlen(szWindowsAPIs\[i\])+1;  
>                 memcpy(pData1+dwOffset,szWindowsAPIs\[i\],l);  
>                 dwOffset+=l;  
>                 //API 数量  
>                 API\_num++;  
>             }  
>             else  
>             {  
>                 //放置 0  
>                 CopyMemory(pData1+dwOffset,&temp,1);  
>                 dwOffset++;  
>             }  
>         }while(szWindowsAPIs\[i\]!=0);  
>         i++;  
>     }  
>     while(szWindowsAPIs\[i\]!=0);  
>     //----------------------------------------  
>     delete pDataTable;  
> }  
> //----------------------------------------------------------------  
> void CPECryptor::SetSectionsWritePermission()  
> {  
>     for(int i=0;i<image\_nt\_headers->FileHeader.NumberOfSections;i++)  
>     {  
>         //设置所有表的标志  
>         image\_section\_header\[i\]->Characteristics=0xC0000040;  
>     }  
> }  
> //---------------------------------------------------------------- 

标 题: 菜鸟注释 PEMaker6 源代码（四）  
作 者: 火影  
时 间: 2007-09-25 23:16    

*   链 接：[`bbs.pediy.com/showthread.php?t=52332`](http://bbs.pediy.com/showthread.php?t=52332)   
    详细信息: 
    
    > 这是最后一篇了，最近被调去急诊科，三天一个夜班，头都大了，注释的很乱，连自己都搞晕了。  
    > #include "stdafx.h"  
    > #include "loader.h"  
    > #include "itmaker.h"  
    >   
    > #ifdef \_DEBUG  
    > #define DEBUG\_NEW  
    > #endif  
    >   
    > \_\_stdcall void DynLoader();  
    > //---------------------------------------------------------  
    > // Thanks FEUERRADER \[AHTeam\] for idea about using \_emit 0!  
    > //---------------------------------------------------------  
    > // byte ptr        (1 bytes)  
    > //定义一个字节  
    > #define byte\_type(x)            \_\_asm \_emit x  
    > // word ptr        (2 bytes)  
    > //低位放低字节  
    > #define word\_type(x)            byte\_type((x>>(0\*8))&0xFF)    byte\_type((x>>(1\*8))&0xFF)  
    > // dword ptr    (4 bytes)  
    > #define dword\_type(x)            byte\_type((x>>(0\*8))&0xFF)    byte\_type((x>>(1\*8))&0xFF)    byte\_type((x>>(2\*8))&0xFF)    byte\_type((x>>(3\*8))&0xFF)  
    > // dword64 ptr    (8 bytes)  
    > #define dword64\_type(x)            dword\_type(x)    dword\_type(x)  
    > // dword64 ptr    (16 bytes)  
    > #define dword128\_type(x)        dword64\_type(x)    dword64\_type(x)  
    > //---------------------------------------------------------  
    > //定义字节  
    > #define bb(x)                    \_\_asm \_emit x  
    > #define db                        byte\_type(0xCC)  
    > #define \_\_random\_code1\_\_        dword64\_type(0X90909090) // Reserve for random code generation  
    > //jmp  
    > #define \_\_jmp\_api                byte\_type(0xFF)    byte\_type(0x25)  
    >   
    > //----------------------------------------------------------------  
    > \_\_stdcall void DynLoader()  
    > {  
    > \_asm  
    > {  
    > //----------------------------------  
    >     dword\_type(DYN\_LOADER\_START\_MAGIC)  
    > //----------------------------------  
    > \_main\_0:  
    >     pushad    // save the registers context in stack  
    >     //重定向编码放 EBP  
    >     call \_main\_1  
    > \_main\_1:  
    >     pop ebp  
    >     sub ebp,offset \_main\_1 // get base ebp  
    >     //----------------------------------------------------  
    >     //====================================================  
    >     //---------------- support dll, ocx  -----------------  
    > \_support\_dll\_0:  
    >     jmp \_support\_dll\_1 // nop; nop; // in the secon time OEP  
    >     jmp \_support\_dll\_2  
    > \_support\_dll\_1:  
    >     //定义在下面，就是那个类似 t\_DATA\_1 结构的东西  
    >     test \[ebp+\_p\_dwFileType\],IMPORT\_TABLE\_OCX  
    >     jz \_no\_dll\_pe\_file\_0  
    >         //esp 怎么来的？  
    >         mov eax,\[esp+24h\]// imagebase  
    >         //AddressOfEntryPoint 成员  
    >         mov ebx,\[esp+30h\]// oep  
    >         cmp eax,ebx  
    >         ja \_no\_dll\_pe\_file\_0  
    >             cmp word ptr \[eax\],IMAGE\_DOS\_SIGNATURE  
    >             jne \_no\_dll\_pe\_file\_0  
    >                 //存入镜像地址  
    >                 mov \[ebp+\_p\_dwImageBase\],eax  
    > //无 DLL 的 PE 文件？  
    > \_no\_dll\_pe\_file\_0:  
    >     //----------------------------------------------------  
    >     //====================================================  
    >     mov eax,\[ebp+\_p\_dwImageBase\]  
    >     add eax,\[eax+03Ch\]  
    >     add eax,080h  
    >     //获取导入表 RVA  
    >     mov ecx,\[eax\]    // image\_nt\_headers->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress  
    >     //ecx 为导入表地址  
    >     add ecx,\[ebp+\_p\_dwImageBase\]  
    >     add ecx,010h    // image\_import\_descriptor.FirstThunk  
    >     //取 FirstTHUNk 值  
    >     mov eax,\[ecx\]  
    >     add eax,\[ebp+\_p\_dwImageBase\]  
    >     mov ebx,\[eax\]  
    >     //存入 API 地址  
    >     mov \[ebp+\_p\_LoadLibrary\],ebx  
    >     add eax,04h  
    >     mov ebx,\[eax\]  
    >     //存入 API 地址  
    >     mov \[ebp+\_p\_GetProcAddress\],ebx  
    >     //----------------------------------------------------  
    >     //------- load library and build api call-jmp --------  
    >     call \_api\_load  
    >     //----------------------------------------------------  
    >   
    >     push MB\_OK | MB\_ICONINFORMATION  
    >     lea eax,\[ebp+\_p\_szCaption\]  
    >     push eax  
    >     lea eax,\[ebp+\_p\_szText\]  
    >     push eax  
    >     push NULL  
    >     call \_jmp\_MessageBox  
    >     // MessageBox(NULL, szText, szCaption, MB\_OK | MB\_ICONINFORMATION) ;  
    >   
    >     //...  
    >     //...  
    >     //...  
    >   
    >     // Place your code here ...  
    >   
    >   
    >     //...  
    >     //...  
    >     //...  
    >   
    >   
    >     //----------------------------------------------------  
    >     //====================================================  
    >     //----------- get write access for headers -----------  
    >     mov edi,\[ebp+\_p\_dwImageBase\]  
    >     add edi,\[edi+03Ch\]// edi -> IMAGE\_NT\_HEADERS  
    >     // get write permission by VirtualProtect()  
    >     lea eax,\[ebp+\_p\_ptempbuffer\]  
    >     push eax  
    >     push PAGE\_READWRITE  
    >     push \[edi+0x54\]//IMAGE\_NT\_HEADERS.OptionalHeader.SizeOfHeaders  
    >     push \[ebp+\_p\_dwImageBase\]  
    >     call \_jmp\_VirtualProtect  
    >     //VirtualProtect(dwImageBase,image\_nt\_header.OptionalHeader.SizeOfHeaders,PAGE\_READWRITE,ptempbuffer);  
    >     //----------------------------------------------------  
    >     //----------------------------------------------------  
    >     //--------------- import table fix up ----------------  
    >     lea eax,\[ebp+\_p\_szKERNEL32\_r\]  
    >     push eax  
    >     push \[ebp+\_p\_dwImportVirtualAddress\]  
    >     push \[ebp+\_p\_dwImageBase\]  
    >     //以上传递参数  
    >     call \_it\_fixup\_1  
    >   
    >     //--------------- import table fix up ----------------  
    >     //call \_it\_fixup  
    >     //----------------------------------------------------  
    >     //====================================================  
    >     //---------------- support dll, ocx  1 ---------------  
    >     //by set second way to oep and relocation table effect  
    >     mov edi,\[ebp+\_p\_dwImageBase\]  
    >     add edi,\[edi+03Ch\]// edi -> IMAGE\_NT\_HEADERS  
    >     mov ax,word ptr \[edi+016h\]// edi -> image\_nt\_headers->FileHeader.Characteristics  
    >     // .IF ((image\_nt\_headers->FileHeader.Characteristics&IMAGE\_FILE\_DLL) ==IMAGE\_FILE\_DLL)  
    >     test ax,IMAGE\_FILE\_DLL  
    >     jz \_no\_dll\_pe\_file\_1  
    >         //--------------- relocation fix up --------------  
    >         call \_reloc\_fixup  
    >         //------------------------------------------------  
    >         mov ax,9090h  
    >         mov word ptr \[ebp+\_support\_dll\_0\],ax  
    >     // .ENDIF  
    > \_no\_dll\_pe\_file\_1:  
    >     //----------------------------------------------------  
    >     //====================================================  
    > \_support\_dll\_2:  
    >     //----------------------------------------------------  
    >     //====================================================  
    >     //--------- Prepare the SEH for OEP approach ---------  
    >     mov eax,\[ebp+\_p\_dwImageBase\]  
    >     add eax,\[ebp+\_p\_dwOrgEntryPoint\]  
    >     mov \[esp+10h\],eax    // pStack.Ebx <- eax  
    >     lea eax,\[ebp+\_except\_handler1\_OEP\_Jump\]  
    >     mov \[esp+1Ch\],eax    // pStack.Eax <- eax  
    >     popad    // restore the first registers context from stack  
    >     //----------------------------------------------------  
    >       // the structured exception handler (SEH) installation  
    >     push eax  
    >     xor  eax, eax  
    >     push dword ptr fs:\[0\]        // NT\_TIB32.ExceptionList  
    >     mov fs:\[0\],esp    // NT\_TIB32.ExceptionList <-ESP  
    >     dword\_type(0xCCCCCCCC)// raise a (int 3) exception  
    >     //----------------------------------------------------  
    >     //====================================================  
    > //--------------------------------------------------------  
    > //========================================================  
    > //------------- t\_size strlen(LPCTSTR lpStr) -------------  
    > \_\_strlen:  
    >     push ebp  
    >     mov ebp,esp  
    >     push ecx  
    >     push esi  
    >     push ebx  
    >     //取得参数  
    >     mov esi,\[ebp+08h\]// -> destination  
    >     mov ecx,255// -> length  
    >     xor ebx,ebx  
    > \_strlenloop:  
    >         lods byte ptr ds:\[esi\]  
    >         cmp al,00h  
    >         jz \_endbufstrlen  
    >         //ebx 计数  
    >         inc ebx  
    >     loop \_strlenloop  
    > \_endbufstrlen:  
    >     mov eax,ebx  
    >     //返回值在 eax  
    >     inc eax  
    >     pop ebx  
    >     pop esi  
    >     pop ecx  
    >     mov esp,ebp  
    >     pop ebp  
    >     ret  
    > //--------------------------------------------------------  
    > //========================================================  
    > //------------------- fix up relocation ------------------  
    > // I have stolen this reloc fix-up code from Morphine 2.7 !!  
    > // so Thanks Holy\_Father && Ratter/29A for it. (www.hxdef.org)  
    > \_reloc\_fixup:  
    >     mov eax,\[ebp+\_p\_dwImageBase\]  
    >     //edx 首地址  
    >     mov edx,eax  
    >     mov ebx,eax  
    >     //ebx-->NT 文件头  
    >     add ebx,\[ebx+3Ch\] // edi -> IMAGE\_NT\_HEADERS  
    >     //其实 RVA 等于 FC  
    >     mov ebx,\[ebx+034h\]// ebx ->image\_nt\_headers->OptionalHeader.ImageBase  
    >     //有可能为 0  
    >     sub edx,ebx // edx -> reloc\_correction  
    >     je \_reloc\_fixup\_end  
    >     mov ebx,\[ebp+\_p\_dwRelocationVirtualAddress\]  
    >     test ebx,ebx  
    >     jz \_reloc\_fixup\_end  
    >     //ebx 是 RVA  
    >     add ebx,eax  
    > \_reloc\_fixup\_block:  
    >     //一个 32 位的“SizeOfBlock（块大小）”  
    >     mov eax,\[ebx+004h\]            //ImageBaseRelocation.SizeOfBlock  
    >     test eax,eax  
    >     jz \_reloc\_fixup\_end  
    >     lea ecx,\[eax\-008h\]            //ImageBaseRelocation.SizeOfBlock - sizeof(TImageBaseRelocation)  
    >     //ecx 为跟在后面的重定位的数目  
    >     shr ecx,001h                //WORD((ImageBaseRelocation.SizeOfBlock - sizeof(TImageBaseRelocation)) / sizeof(Word))  
    >     //edi 为重定位数据开始地址  
    >     lea edi,\[ebx+008h\]            //PImageBaseRelocation + sizeof(TImageBaseRelocation)  
    > \_reloc\_fixup\_do\_entry:  
    >         movzx eax,word ptr \[edi\]//Entry  
    >         push edx  
    >         mov edx,eax  
    >         //高 4 位的重定位类型  
    >         shr eax,00Ch            //Type = Entry >> 12  
    >   
    >         mov esi,\[ebp+\_p\_dwImageBase\]//ImageBase  
    >         //低 12 位的重定位位置  
    >         and dx,00FFFh  
    >         //得到计算后的偏移地址  
    >         add esi,\[ebx\]            //ImageBase + ImageBaseRelocation.VirtualAddress  
    >         //得到重定位数据的偏移地址  
    >         add esi,edx                //ImageBase + ImageBaseRelocation.VirtualAddress+Entry & 0x0FFF  
    >         pop edx  
    >         //------------------------------  
    >         //---- IMAGE\_REL\_BASED\_HIGH 重定位的高 16 位必须被用于被偏移量所指向的那个 16 位的 WORD 单元，  
    >         //此 WORD 是一个 32 位的 DWORD 的高位 WORD----  
    > \_reloc\_fixup\_HIGH:  
    >         //重定位类型  
    >         dec eax  
    >         jnz \_reloc\_fixup\_LOW  
    >             //edx 重定位偏移  
    >             mov eax,edx  
    >             //取高字节  
    >             shr eax,010h        //HIWORD(Delta)  
    >             jmp \_reloc\_fixup\_LOW\_fixup  
    >         //------------------------------  
    >         //---- IMAGE\_REL\_BASED\_LOW -----  
    > \_reloc\_fixup\_LOW:  
    >             dec eax  
    >         jnz \_reloc\_fixup\_HIGHLOW  
    >         movzx eax,dx            //LOWORD(Delta)  
    > \_reloc\_fixup\_LOW\_fixup:  
    >             //  
    >             add word ptr \[esi\],ax  
    >         jmp \_reloc\_fixup\_next\_entry  
    >         //------------------------------  
    >         //-- IMAGE\_REL\_BASED\_HIGHLOW ---  
    > \_reloc\_fixup\_HIGHLOW:  
    >             dec eax  
    >         jnz \_reloc\_fixup\_next\_entry  
    >         add \[esi\],edx  
    >         //------------------------------  
    > \_reloc\_fixup\_next\_entry:  
    >         inc edi  
    >         inc edi                        //Entry++  
    >         loop \_reloc\_fixup\_do\_entry  
    >   
    > \_reloc\_fixup\_next\_base:  
    >     add ebx,\[ebx+004h\]                //ImageBaseRelocation + ImageBaseRelocation.SizeOfBlock  
    >     jmp \_reloc\_fixup\_block  
    > \_reloc\_fixup\_end:  
    >     // clean relocation table  
    > /\*    mov eax,\[ebp+\_p\_dwImageBase\]  
    >     add eax,\[ebp+\_p\_dwRelocationVirtualAddress\]  
    >     lea edi,\[eax\]  
    >     xor eax,eax  
    >     add ecx,\[ebp+\_p\_dwRelocationSize\]  
    >     rep stos byte  ptr \[edi\]\*/  
    >     ret  
    > //--------------------------------------------------------  
    > //========================================================  
    > //--------------- 修复导入表 ----------------  
    > /\*  
    > ebp+10h    \[ebp+\_p\_szKERNEL32\_r\]  
    > ebp+0ch    \_p\_dwImportVirtualAddress  
    > ebp+08h    \_p\_dwImageBase  
    > \---------------------------------  
    > ebp-04h        \_p\_dwNewITaddress  
    > ebp-08h        \_p\_dwThunk  
    > ebp-0ch        \_p\_dwHintName  
    > ebp-10h        \_p\_dwLibraryName  
    > ebp-14h        \_p\_dwAPIaddress  
    > \*/  
    > \_it\_fixup\_1:  
    >     push ebp  
    >     mov ebp,esp  
    >     add esp,-14h  
    >     push PAGE\_READWRITE  
    >     push MEM\_COMMIT  
    >     push 01D000h  
    >     push 0  
    >     call \_jmp\_VirtualAlloc  
    >     //NewITaddress=VirtualAlloc(NULL, 0x01D000, MEM\_COMMIT, PAGE\_READWRITE);  
    >     //存入地址  
    >     mov \[ebp\-04h\],eax  
    >     mov ebx,\[ebp+0ch\]  
    >     test ebx,ebx  
    >     jz \_it\_fixup\_1\_end  
    >     //esi:\_p\_dwImageBase  
    >     mov esi,\[ebp+08h\]  
    >     //导入表地址  
    >     add ebx,esi                                    // dwImageBase + dwImportVirtualAddress  
    > \_it\_fixup\_1\_get\_lib\_address\_loop:  
    >         //名称  
    >         mov eax,\[ebx+0ch\]                        // image\_import\_descriptor.Name  
    >         test eax,eax  
    >         jz \_it\_fixup\_1\_end  
    >         //FirstThunk RVA  
    >         mov ecx,\[ebx+10h\]                        // image\_import\_descriptor.FirstThunk  
    >         add ecx,esi  
    >         //FirstThunk 指向的 RVA 数组  
    >         mov \[ebp\-08h\],ecx                        // dwThunk  
    >         mov ecx,\[ebx\]                            // image\_import\_descriptor.Characteristics  
    >         //有些编译器会将此置 0  
    >         test ecx,ecx  
    >         jnz \_it\_fixup\_1\_table  
    >             // image\_import\_descriptor.FirstThunk  
    >             mov ecx,\[ebx+10h\]  
    > \_it\_fixup\_1\_table:  
    >         //esi:\_p\_dwImageBase  
    >         add ecx,esi  
    >         mov \[ebp\-0ch\],ecx  // dwHintName  
    >         add eax,esi                                // image\_import\_descriptor.Name + dwImageBase = ModuleName  
    >         push eax                                // lpLibFileName  
    >         mov \[ebp\-10h\],eax  
    >         call \_jmp\_LoadLibrary                    // LoadLibrary(lpLibFileName);  
    >   
    >         test eax,eax  
    >         jz \_it\_fixup\_1\_end  
    >         //edi:动态链接库句柄  
    >         mov edi,eax  
    > \_it\_fixup\_1\_get\_proc\_address\_loop:  
    >             //指向一个 IMAGE\_THUNK\_DATA 结构数组  
    >             mov ecx,\[ebp\-0ch\]            // dwHintName  
    >             mov edx,\[ecx\]                        // image\_thunk\_data.Ordinal  
    >             test edx,edx  
    >             jz \_it\_fixup\_1\_next\_module  
    >             //以序数方式  
    >             test edx,080000000h                    // .IF( import by ordinal )  
    >             //对于每个数组元素，我们比对元素值是否等于 IMAGE\_ORDINAL\_FLAG32。  
    >             //如果该元素值的最高二进位为 1，  
    >             //那么函数是由序数引入的，可以从该值的低字节提取序数。  
    >             jz \_it\_fixup\_1\_by\_name  
    >                 //取出序数  
    >                 and edx,07FFFFFFFh                // get ordinal  
    >                 jmp \_it\_fixup\_1\_get\_addr  
    > \_it\_fixup\_1\_by\_name:  
    >             add edx,esi                            // image\_thunk\_data.Ordinal + dwImageBase = OrdinalName  
    >             inc edx  
    >             //取出 API 名称  
    >             inc edx                                // OrdinalName.Name  
    > \_it\_fixup\_1\_get\_addr:  
    >             push edx                            // lpProcName  
    >             push edi                            // hModule  
    >             call \_jmp\_GetProcAddress            // GetProcAddress(hModule, lpProcName);  
    >             //获得 API 地址  
    >             mov \[ebp\-14h\],eax                    //\_p\_dwAPIaddress  
    >             //================================================================  
    >             //mov \[ecx\],eax//<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<  
    >             //hModule  
    >             push edi  
    >             // dwImageBase  
    >             push esi  
    >             //ebx=引入表地址  
    >             push ebx  
    >   
    >             mov ebx,\[ebp\-10h\]  
    >             push ebx  
    >             push ebx  
    >             //大写  
    >             call \_char\_upper  
    >             //esi=\_p\_dwLibraryName  
    >             mov esi,\[ebp\-10h\]  
    >             mov edi,\[ebp+010h\]    // \[ebp+\_p\_szKernel32\]  
    > \_it\_fixup\_1\_check\_dll\_redirected:  
    >             push edi  
    >             call \_\_strlen  
    >             add esp, 4  
    >   
    >             mov ebx,eax  
    >             mov ecx,eax  
    >             push edi  
    >             push esi  
    >             //比较两个动态库字符串  
    >             repe cmps //byte ptr \[edi\], byte ptr \[esi\]  
    >             jz \_it\_fixup\_1\_do\_normal\_it\_0  
    >             pop esi  
    >             pop edi  
    >             //下一个  
    >             add edi,ebx  
    >             //是否结束  
    >             cmp byte ptr \[edi\],0  
    >             jnz \_it\_fixup\_1\_check\_dll\_redirected  
    >                 //\_p\_dwThunk  
    >                 mov ecx,\[ebp\-08h\]  
    >                 //api 地址  
    >                 mov eax,\[ebp\-014h\]  
    >                 //FirstThunk 指向的数组被填充为真正的函数地址  
    >                 mov \[ecx\],eax  
    >                 jmp \_it\_fixup\_1\_do\_normal\_it\_1  
    > \_it\_fixup\_1\_do\_normal\_it\_0:  
    >             //对应 repe 之前的 push  
    >                 pop esi  
    >                 pop edi  
    >                 //新申请的内存空间  
    >                 mov edi,\[ebp\-04h\]  
    >                 //插入 JMP  
    >                 mov byte ptr \[edi\], 0e9h  
    >                 //API 地址  
    >                 mov eax,\[ebp\-14h\]  
    >                 //计算跳转偏移  
    >                 sub eax, edi  
    >                 sub eax,05h  
    >                 //插入偏移  
    >                 mov \[edi+1\],eax  
    >                 //0c8bh-->mov eax,eax,对齐  
    >                 mov word ptr \[edi+05\],0c8bh  
    >                 //THUNK  
    >                 mov ecx,\[ebp\-08h\]  
    >                 //将原程序的 THUNK 指向我们的内存地址  
    >                 mov \[ecx\],edi  
    >                 //下一个位置  
    >                 add dword ptr \[ebp\-04h\],07h  
    > \_it\_fixup\_1\_do\_normal\_it\_1:  
    >             pop ebx  
    >             pop esi  
    >             pop edi  
    >             //================================================================  
    >             add dword ptr \[ebp\-08h\],004h    // dwThunk => next dwThunk  
    >             add dword ptr \[ebp\-0ch\],004h// dwHintName => next dwHintName  
    >         jmp \_it\_fixup\_1\_get\_proc\_address\_loop  
    > \_it\_fixup\_1\_next\_module:  
    >         add ebx,014h                            // sizeof(IMAGE\_IMPORT\_DESCRIPTOR)  
    >     jmp \_it\_fixup\_1\_get\_lib\_address\_loop  
    > \_it\_fixup\_1\_end:  
    >     mov esp,ebp  
    >     pop ebp  
    >     ret 0ch  
    > //--------------------------------------------------------  
    > //========================================================  
    > //-- char\_upper(LPCTSTR lpDestination, LPCTSTR lpSource )  
    > \_char\_upper:  
    >     push ebp  
    >     mov ebp,esp  
    >     push ecx  
    >     push eax  
    >     push esi  
    >     push edi  
    >     mov edi,dword ptr \[ebp+08h\]// -> Destination  
    >     mov esi,dword ptr \[ebp+0Ch\]// -> Source  
    >     mov ecx,255// -> Length  
    >     xor eax,eax  
    > \_\_makeupperloop:  
    >         lods byte ptr \[esi\]//ESI  
    >         cmp al,00h  
    >         jz endofbuffer  
    >         cmp al,60h  
    >         jc notinlowerfield  
    >         cmp al,7bh  
    >         jnc notinlowerfield  
    >         sub al,20h  
    > notinlowerfield:  
    >         stos byte ptr \[edi\]//EDI  
    >     loop \_\_makeupperloop  
    > endofbuffer:  
    >     pop edi  
    >     pop esi  
    >     pop eax  
    >     pop ecx  
    >     pop ebp  
    >     retn 08h  
    > //--------------------------------------------------------  
    > //========================================================  
    > //------------- load necessary api functions -------------  
    > //以下都是重定位之后的地址  
    > \_api\_load:  
    >     //取得\_p\_szKernel32 字符串地址  
    >     lea edi,\[ebp+\_p\_szKernel32\]  
    >     //取得\_p\_GetModuleHandle 的地址  
    >     lea ebx,\[ebp+\_p\_GetModuleHandle\]  
    >     //取得\_jmp\_GetModuleHandle 地址  
    >     lea ecx,\[ebp+\_jmp\_GetModuleHandle\]  
    >     //取得\_jmp\_GetModuleFileName 的地址？  
    >     add ecx,02h  
    > \_api\_get\_lib\_address\_loop:  
    >         //保存 ECX  
    >         push ecx  
    >         push edi  
    >         mov eax,offset \_p\_LoadLibrary  
    >         //加载 kernel32.dll  
    >         call \[ebp+eax\]// LoadLibrary(lpLibFileName);  
    >         //-------------------  
    >         pop ecx  
    >         //kernel32.dll 模块地址  
    >         mov esi,eax    // esi -> hModule  
    >         //字符串长度  
    >         push edi  
    >         call \_\_strlen  
    >         //平衡堆栈,弹出参数  
    >         add esp,04h  
    >         //指向下一个字符串  
    >         add edi,eax  
    > \_api\_get\_proc\_address\_loop:  
    >             push ecx  
    >             //edi=getmodulhanda  
    >             push edi  
    >             //esi=hmodule  
    >             push esi  
    >             mov eax,offset \_p\_GetProcAddress  
    >             call \[ebp+eax\]// GetModuleHandle=GetProcAddress(hModule, lpProcName);  
    >             //--------------------  
    >             pop ecx  
    >             //存入 GetModuleHandle 的实际地址  
    >             mov \[ebx\],eax  
    >             //注意看这里！！  
    >             mov \[ecx\],ebx  
    >             //下一个  
    >             add ebx,04h  
    >             //下一个  
    >             add ecx,06h  
    >             //下一个  
    >             push edi  
    >             call \_\_strlen  
    >             add esp,04h  
    >             add edi,eax  
    >             mov al,byte ptr \[edi\]  
    >         test al,al  
    >         //不为 0 继续循环  
    >         jnz \_api\_get\_proc\_address\_loop  
    >         //edi 地址加一  
    >         inc edi  
    >         mov al,byte ptr \[edi\]  
    >     test al,al  
    >     //不为 0 则载入 user32.dll  
    >     jnz \_api\_get\_lib\_address\_loop  
    >     ret  
    > //----------------------------------------------------------  
    > //--------------------------------------------------------  
    > //========================================================  
    > // -------- exception handler expression filter ----------  
    > \_except\_handler1\_OEP\_Jump:  
    >     push ebp  
    >     mov ebp,esp  
    >     mov eax,\[ebp+010h\]    // PCONTEXT: pContext <- eax  
    >     //---------------  
    >     push edi  
    >     // restore original SEH  
    >     mov edi,\[eax+0C4h\]    // pContext.Esp  
    >     push dword ptr ds:\[edi\]  
    >     pop dword ptr fs:\[0\]  
    >     add dword ptr \[eax+0C4h\],8    // pContext.Esp  
    >   
    >     // set the Eip to the OEP  
    >     mov edi,\[eax+0A4h\] // edi <- pContext.Ebx  
    >     mov \[eax+0B8h\],edi // pContext.Eip <- edi  
    >     //  
    >     pop edi  
    >     //---------------  
    >     mov eax, EXCEPTION\_CONTINUE\_SEARCH  
    >     leave  
    >     ret  
    > //--------------------------------------------------------  
    > //========================================================  
    >     dword\_type(DYN\_LOADER\_START\_DATA1)  
    > //0xcccccccc 自填充----------------------------------  
    > \_p\_dwFileType:                    dword\_type(0xCCCCCCCC)  
    > \_p\_dwImageBase:                    dword\_type(0xCCCCCCCC)  
    > \_p\_dwOrgEntryPoint:                dword\_type(0xCCCCCCCC)  
    > \_p\_dwImportVirtualAddress:        dword\_type(0xCCCCCCCC)  
    > \_p\_dwRelocationVirtualAddress:    dword\_type(0xCCCCCCCC)  
    > \_p\_dwRelocationSize:            dword\_type(0xCCCCCCCC)  
    > //----------------------------------  
    > \_tls\_dwStartAddressOfRawData:    dword\_type(0xCCCCCCCC)  
    > \_tls\_dwEndAddressOfRawData:        dword\_type(0xCCCCCCCC)  
    > \_tls\_dwAddressOfIndex:            dword\_type(0xCCCCCCCC)  
    > \_tls\_dwAddressOfCallBacks:        dword\_type(0xCCCCCCCC)  
    > \_tls\_dwSizeOfZeroFill:            dword\_type(0xCCCCCCCC)  
    > \_tls\_dwCharacteristics:            dword\_type(0xCCCCCCCC)  
    > //----------------------------------  
    > \_p\_szKernel32:                //db "Kernel32.dll",0,13  
    >         db db db db db db db db db db db db db  
    > \_p\_szGetModuleHandle:        //db "GetModuleHandleA",0,17  
    >         db db db db db db db db db db db db db db db db db  
    > \_p\_szVirtualProtect:        //db "VirtualProtect",0,15  
    >         db db db db db db db db db db db db db db db  
    > \_p\_szGetModuleFileName:    //db "GetModuleFileNameA",0,19  
    >         db db db db    db db db db    db db db db db db db db db db db  
    > \_p\_szCreateFile:            //db "CreateFileA",0,12  
    >         db db db db db db db db db db db db  
    > \_p\_szGlobalAlloc:            //db "GlobalAlloc",0,12  
    >         db db db db db db db db db db db db  
    > \_p\_szVirtualAlloc:            //db "VirtualAlloc",0,13  
    >         db db db db db db db db db db db db db  
    > \_p\_szLoadLibrary:            //db "LoadLibraryA",0,13  
    >         db db db db db db db db db db db db db  
    > \_p\_szGetProcAddress:        //db "GetProcAddress",0,15  
    >         db db db db db db db db db db db db db db db  
    >         byte\_type(0)  
    >   
    > \_p\_szUser32:                //db "User32.dll",0,11  
    >         db db db db db db db db db db db  
    > \_p\_szMessageBox:            //db "MessageBoxA",0,12  
    >         db db db db db db db db db db db db  
    >         byte\_type(0)  
    >         byte\_type(0)  
    > //----------------------------------  
    > \_p\_LoadLibrary:                    dword\_type(0xCCCCCCCC)  
    > \_p\_GetProcAddress:                dword\_type(0xCCCCCCCC)  
    > \_p\_GetModuleHandle:  
    >                                 dword\_type(0xCCCCCCCC)  
    >                                 dword\_type(0xCCCCCCCC)  
    >                                 dword\_type(0xCCCCCCCC)  
    >                                 dword\_type(0xCCCCCCCC)  
    >                                 dword\_type(0xCCCCCCCC)  
    >                                 dword\_type(0xCCCCCCCC)  
    >                                 dword\_type(0xCCCCCCCC)  
    >                                 dword\_type(0xCCCCCCCC)  
    >                                 dword\_type(0xCCCCCCCC)  
    >   
    > \_jmp\_GetModuleHandle:            \_\_jmp\_api    dword\_type(0xCCCCCCCC)  
    > \_jmp\_VirtualProtect:            \_\_jmp\_api    dword\_type(0xCCCCCCCC)  
    > \_jmp\_GetModuleFileName:            \_\_jmp\_api    dword\_type(0xCCCCCCCC)  
    > \_jmp\_CreateFile:                \_\_jmp\_api    dword\_type(0xCCCCCCCC)  
    > \_jmp\_GlobalAlloc:                \_\_jmp\_api    dword\_type(0xCCCCCCCC)  
    > \_jmp\_VirtualAlloc:                \_\_jmp\_api    dword\_type(0xCCCCCCCC)  
    > \_jmp\_LoadLibrary:                \_\_jmp\_api    dword\_type(0xCCCCCCCC)  
    > \_jmp\_GetProcAddress:            \_\_jmp\_api    dword\_type(0xCCCCCCCC)  
    >   
    > \_jmp\_MessageBox:                \_\_jmp\_api    dword\_type(0xCCCCCCCC)  
    > \_p\_szKERNEL32\_r:  
    >         //db "KERNEL32.DLL",0,13  
    >         bb('K') bb('E') bb('R') bb('N') bb('E') bb('L') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)  
    >         //db "USER32.DLL",0,11  
    >         bb('U') bb('S') bb('E') bb('R') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)  
    >         //db "GDI32.DLL",0,10  
    >         bb('G') bb('D') bb('I') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)  
    >         //db "ADVAPI32.DLL",0,13  
    >         bb('A') bb('D') bb('V') bb('A') bb('P') bb('I') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)  
    >         //db "SHELL32.DLL",0,12  
    >         bb('S') bb('H') bb('E') bb('L') bb('L') bb('3') bb('2') bb('.') bb('D') bb('L') bb('L') bb(0)  
    >         byte\_type(0)  
    > \_p\_szCaption:  
    > bb('A') bb('s') bb('h') bb('k') bb('b') bb('i') bb('z') bb(0)  
    > \_p\_szText:  
    > bb('T') bb('h') bb('i') bb('s') bb(' ') bb('E') bb('x') bb('a') bb('m') bb('p') bb('l')  
    > bb('e') bb(' ') bb('s') bb('h') bb('o') bb('w') bb('s') bb(' ') bb('h') bb('o') bb('w')  
    > bb(' ') bb('A') bb('P') bb('I') bb(' ') bb('r') bb('e') bb('d') bb('i') bb('r') bb('e')  
    > bb('c') bb('t') bb('i') bb('o') bb('n') bb(' ') bb('t') bb('e') bb('c') bb('h') bb('n')  
    > bb('i') bb('q') bb('u') bb('e') bb(' ') bb('w') bb('o') bb('r') bb('k') bb('s') bb('.')  
    > bb(0)  
    > \_p\_dwThunk:                        dword\_type(0xCCCCCCCC)  
    > \_p\_dwHintName:                    dword\_type(0xCCCCCCCC)  
    > \_p\_ptempbuffer:                    dword\_type(0xCCCCCCCC)  
    > //----------------------------------  
    >     dword\_type(DYN\_LOADER\_END\_MAGIC)  
    > //----------------------------------  
    > }  
    > }  
    > //---------------------------------------------------------------- 
    
     #BeginLibraryItem "/Library/copyright.lbi" CLASS="p9z"   
    
    | 
    
    * * *
    
     |
    | --- |
    | 
    
    |  ||  ||  ©2000-2008 [**PEdiy.com**](http://www.pediy.com/) All rights reserved.   
    By PEDIY  |
    | --- | --- | --- |
    
     |
    
     #EndLibraryItem