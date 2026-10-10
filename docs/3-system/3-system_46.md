

# \[原创\]扩展任意节大小.

> 原文：[`www.pediy.com/kssd/pediy10/84932.html`](https://www.pediy.com/kssd/pediy10/84932.html)

如题.  
很久以前就写这么个东西.  
然后一直懒觉得费力不讨好就没写.  
上个星期?上上个星期开始写.写了两个下午就没劲了.  
然后一直到昨天断断续续的写完了..  
  
代码.  
额- -~  
就是修正一些地址什么的吧.~  
好像没什么好讲的 哈哈  
  
  
  
C++语言: ExpandSection  
 BOOL CPatch::ExpandSection( PBYTE pImage, **int** ExpandSize, **int** SectionID)  
 {  
       
     **if** ( !InitPeHelp( pImage))  
     {  
         **return** **false**;  
     }  
   
     **int** iExpandVsize;    *//对齐后需要扩展的 Vsize*  
     **int** iExpandRsize;    *//对齐后需要扩展的 Rsize*  
     PBYTE pNewImageBase;    *//指向扩展后的文件镜像*  
     **int** iNewImageSize;    *//新文件的 ImageSize*  
   
     iExpandVsize = ( ( ExpandSize > pNtHeader->OptionalHeader.SectionAlignment ? ExpandSize/pNtHeader->OptionalHeader.SectionAlignment : 0 )  
         + ( ExpandSize % pNtHeader->OptionalHeader.SectionAlignment != 0 ? 1 : 0) ) \* pNtHeader->OptionalHeader.SectionAlignment;  
     iExpandVsize += iExpandVsize == 0 ? pNtHeader->OptionalHeader.SectionAlignment : 0;  
   
     iExpandRsize = ( ( ExpandSize > pNtHeader->OptionalHeader.FileAlignment ? ExpandSize/pNtHeader->OptionalHeader.FileAlignment : 0)  
         + ( ExpandSize % pNtHeader->OptionalHeader.FileAlignment !=0 ? 1 : 0) ) \* pNtHeader->OptionalHeader.FileAlignment;  
     iExpandRsize += iExpandRsize == 0? pNtHeader->OptionalHeader.FileAlignment : 0;  
   
     printf( "\\r\\niExpandVsize:%08X,iExpandRsize:%08X\\r\\n", iExpandVsize, iExpandVsize);  
   
   
     iNewImageSize = pNtHeader->OptionalHeader.SizeOfImage  + iExpandVsize;  
     pNewImageBase = (PBYTE)VirtualAlloc( NULL, iNewImageSize, MEM\_COMMIT, PAGE\_EXECUTE\_READWRITE);  
   
     memcpy( pNewImageBase, pImageBase, pNtHeader->OptionalHeader.SizeOfHeaders );    *//Copy 文件头*  
       
     *//调整新文件头指针*  
     PIMAGE\_DOS\_HEADER pNewDosHeader = (PIMAGE\_DOS\_HEADER)pNewImageBase;  
     printf( "pNewDosHeader:%p\\r\\n", pNewDosHeader);  
     PIMAGE\_NT\_HEADERS pNewNtHeader = (PIMAGE\_NT\_HEADERS)(pNewImageBase + pNewDosHeader->e\_lfanew);  
     printf( "pNewNtHeader:%p\\r\\n", pNewNtHeader);  
     PIMAGE\_SECTION\_HEADER pNewSectionHeader = (PIMAGE\_SECTION\_HEADER)((PBYTE)pNewNtHeader + **sizeof**(IMAGE\_NT\_HEADERS));  
     printf( "pNewSectionHeader:%p\\r\\n", pNewSectionHeader);  
   
     PIMAGE\_SECTION\_HEADER pTheAddSectionHeader = pNewSectionHeader + SectionID - 1;  
     **int** iTheBreakRVA = pTheAddSectionHeader->VirtualAddress + pTheAddSectionHeader->Misc.VirtualSize;    *//在这个 RVA 后添加*  
     **int** iTheBreakOffset = pTheAddSectionHeader->PointerToRawData + pTheAddSectionHeader->SizeOfRawData;    *//在这个 Offset 后添加*  
     *//所有在 BreakPoint 后面的参数(偏移 or RVA)都要加上 iExpandVSize or iExpandRSize*  
   
     pNewNtHeader->FileHeader.PointerToSymbolTable += pNewNtHeader->FileHeader.PointerToSymbolTable >= iTheBreakRVA  
         ? iExpandVsize : 0;    *// 指向调试符号,RVA*  
     pNewNtHeader->OptionalHeader.AddressOfEntryPoint += pNewNtHeader->OptionalHeader.AddressOfEntryPoint >= iTheBreakRVA  
         ? iExpandVsize : 0;    *//指向文件入口点,RVA*  
     pNewNtHeader->OptionalHeader.BaseOfCode += pNewNtHeader->OptionalHeader.BaseOfCode >= iTheBreakRVA  
         ? iExpandVsize : 0;    *//代码基址,RVA*  
     pNewNtHeader->OptionalHeader.BaseOfData += pNewNtHeader->OptionalHeader.BaseOfData >= iTheBreakRVA  
         ? iExpandVsize : 0; *//数据基址,RVA*  
     pNewNtHeader->OptionalHeader.SizeOfImage += iExpandVsize;    *//镜像大小 ,得加*  
     *//pNtHeader->OptionalHeader.SizeOfCode 不知道咋搞,不管他, 反正不管他也没事*  
       
     *//这个先不管, 到后面再去修改得了...*  
     *//for ( int i = 0; i < 16; i++)    //DataDirectory ,RVA, 一共有 16 项*  
     *//{*  
     \[I\]//    pNewNtHeader->OptionalHeader.DataDirectory\[i\].VirtualAddress += pNewNtHeader->OptionalHeader.DataDirectory\[i\].VirtualAddress > iTheBreakRVA && pNewNtHeader->OptionalHeader.DataDirectory*.VirtualAddress != 0*  
     *//        ? iExpandVsize : 0;*  
     *//}*  
           
     *//修改第 SectionID 个节表的参数(Vsize 和 RSize)*  
     pTheAddSectionHeader->SizeOfRawData += iExpandRsize;  
     pTheAddSectionHeader->Misc.VirtualSize += iExpandVsize;  
   
     *//修改第 SectionID 后的节表.*  
     **for** ( **int** i = SectionID; i < pNewNtHeader->FileHeader.NumberOfSections; i++)  
     {  
   
         PIMAGE\_SECTION\_HEADER pNowSecHeader = pNewSectionHeader + i;  
         pNowSecHeader->PointerToLinenumbers += pNowSecHeader->PointerToLinenumbers > 0   
             ? iExpandVsize : 0;    *//这个不确定是 RVA 还是 Roffset 估摸着是 RVA 吧*  
         pNowSecHeader->PointerToRawData += pNowSecHeader->PointerToRawData > 0   
             ? iExpandRsize : 0;    *//Offset*  
         pNowSecHeader->PointerToRelocations += pNowSecHeader->PointerToRelocations > 0   
             ? iExpandVsize : 0;    *//不确定 , 猜 RVA*  
         pNowSecHeader->VirtualAddress += pNowSecHeader->VirtualAddress > 0   
             ? iExpandVsize : 0;    *//RVA*  
     }  
   
     *//Copy 各个节*  
     **for** ( **int** i = 0; i < pNewNtHeader->FileHeader.NumberOfSections; i++)  
     {  
   
         PIMAGE\_SECTION\_HEADER pNowSecHeader = pNewSectionHeader + i;  
         PIMAGE\_SECTION\_HEADER pSrcSecHeader = pSectionHeader + i;  
   
         memcpy( pNowSecHeader->VirtualAddress + pNewImageBase, pSrcSecHeader->VirtualAddress + pImageBase, pSrcSecHeader->SizeOfRawData);  
     }  
   
   
     *//EAT*  
     **if** ( pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress )  
     {  
   
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress +=  
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         PIMAGE\_EXPORT\_DIRECTORY pNewExportDirectory = (PIMAGE\_EXPORT\_DIRECTORY)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress);  
           
         pNewExportDirectory->AddressOfFunctions +=   
             pNewExportDirectory->AddressOfFunctions >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         pNewExportDirectory->AddressOfNameOrdinals +=  
             pNewExportDirectory->AddressOfNameOrdinals >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         pNewExportDirectory->AddressOfNames +=  
             pNewExportDirectory->AddressOfNames >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         pNewExportDirectory->Base +=  
             pNewExportDirectory->Base >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         PDWORD pNames = PDWORD( pNewImageBase + pNewExportDirectory->AddressOfNames);  
         **for** ( **int** i = 0; i < pNewExportDirectory->NumberOfNames; i++)  
         {  
             *//\_\_asm int 3;*  
             pNames\[i\] += pNames\[i\] >= iTheBreakRVA ? iExpandVsize : 0;  
             \[I\]//printf( "\\r\\nExportAPI \\r\\nName :%s \\r\\nRVA %X\\r\\n", pNames\[i\] + pNewImageBase, pNames*);*  
         }  
   
         PDWORD pFuntions = PDWORD( pNewImageBase + pNewExportDirectory->AddressOfFunctions);  
         **for** ( **int** i = 0; i < pNewExportDirectory->NumberOfFunctions; i++)  
         {  
   
             pFuntions\[i\] += pFuntions\[i\] >= iTheBreakRVA ? iExpandVsize : 0;  
             \[I\]//printf( "\\r\\nExprtAPI RVA: %X\\r\\niTheBreakRVA:%08X\\r\\n", pFuntions*, iTheBreakRVA );*  
   
         }  
     }  
   
     *//Import Table*  
     **if** ( pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress)  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress +=   
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         PIMAGE\_IMPORT\_DESCRIPTOR pNewImpDesciptor = (PIMAGE\_IMPORT\_DESCRIPTOR)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtualAddress);  
   
         **while** ( pNewImpDesciptor->FirstThunk)  
         {  
   
             pNewImpDesciptor->FirstThunk += pNewImpDesciptor->FirstThunk >= iTheBreakRVA  
                 ? iExpandVsize : 0;  
   
             pNewImpDesciptor->Name += pNewImpDesciptor->Name >= iTheBreakRVA  
                 ? iExpandVsize : 0;  
   
             pNewImpDesciptor->OriginalFirstThunk += pNewImpDesciptor->OriginalFirstThunk >= iTheBreakRVA  
                 ? iExpandVsize : 0;  
   
             *//printf( "Import Dll Name : %s\\r\\n", pNewImageBase + pNewImpDesciptor->Name);*  
   
   
             PIMAGE\_THUNK\_DATA pOriginalThunk = (PIMAGE\_THUNK\_DATA)(pNewImageBase + pNewImpDesciptor->OriginalFirstThunk);  
             **while** ( pOriginalThunk->u1.Function)  
             {  
                 pOriginalThunk->u1.AddressOfData += pOriginalThunk->u1.AddressOfData >= iTheBreakRVA  
                     ? iExpandVsize : 0;  
                   
                 *//printf( "\\r\\nImportAPI Name : %s", pNewImageBase + pOriginalThunk->u1.AddressOfData + 2);*  
                 pOriginalThunk++;  
             }  
   
             *//这个应该不要填充~ 这里由系统填充的*  
             *//PIMAGE\_THUNK\_DATA pThunk = (PIMAGE\_THUNK\_DATA)(pNewImageBase + pNewImpDesciptor->FirstThunk);*  
             *//while ( pThunk->u1.Function)*  
             *//{*  
             *//    \_\_asm int 3;*  
             *//    pThunk->u1.Function += pThunk->u1.Function >= iTheBreakRVA*  
             *//        ? iTheBreakRVA : 0;*  
   
             *//    pThunk++;*  
             *//}*  
   
             pNewImpDesciptor++;  
         }  
     }  
   
     *//资源*  
     **if** ( pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_RESOURCE\].VirtualAddress)  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_RESOURCE\].VirtualAddress +=  
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_RESOURCE\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         PIMAGE\_RESOURCE\_DIRECTORY pTopResDir = (PIMAGE\_RESOURCE\_DIRECTORY)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_RESOURCE\].VirtualAddress);  
         MoveTheResource( (PBYTE)pTopResDir, pTopResDir, iTheBreakRVA, iExpandVsize);  
     }  
   
     *//IMAGE\_DIRECTORY\_ENTRY\_SECURITY 异常表?不处理吧.貌似...*  
   
     *//IMAGE\_DIRECTORY\_ENTRY\_SECURITY.貌似是绑定证书 事实上改了也没用 0 0*  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_SECURITY\].VirtualAddress +=   
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_SECURITY\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
     }  
   
     *//调试目录*  
     **if** ( pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DEBUG\].VirtualAddress )  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DEBUG\].VirtualAddress +=  
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DEBUG\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         **int** nCount = pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DEBUG\].Size / **sizeof**(IMAGE\_DEBUG\_DIRECTORY);  
         PIMAGE\_DEBUG\_DIRECTORY pDbgDir = (PIMAGE\_DEBUG\_DIRECTORY)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DEBUG\].VirtualAddress);  
         **while** ( nCount--)  
         {  
             pDbgDir->AddressOfRawData += pDbgDir->AddressOfRawData >= iTheBreakRVA ? iExpandVsize : 0;  
             pDbgDir->PointerToRawData += pDbgDir->PointerToRawData >= iTheBreakOffset ? iExpandRsize : 0;  
         }  
     }  
       
     *//IMAGE\_DIRECTORY\_ENTRY\_ARCHITECTURE //不晓得*  
       
     *//IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR 全局指针*  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR\].VirtualAddress +=   
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
     }  
   
     *//IMAGE\_DIRECTORY\_ENTRY\_TLS tls table*  
     *//就这样吧~没有看到适合测试这个的文件,貌似 Delphi 写的东西.*  
     *// - -! DLL 貌似没有 TLS? 哦 是 LoadLibrary 跟  \_\_declspec(thread)不和谐*  
     *//反正懒得管了*  
     **if** ( pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS\].VirtualAddress )  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS\].VirtualAddress +=   
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
           
         PIMAGE\_TLS\_DIRECTORY pTlsDir = (PIMAGE\_TLS\_DIRECTORY)(pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS\].VirtualAddress);  
   
         pTlsDir->AddressOfCallBacks += pTlsDir->AddressOfCallBacks >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;  
         pTlsDir->AddressOfIndex += pTlsDir->AddressOfIndex >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;  
         pTlsDir->StartAddressOfRawData += pTlsDir->StartAddressOfRawData >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;  
         pTlsDir->EndAddressOfRawData += pTlsDir->EndAddressOfRawData >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;  
   
         PDWORD pCallBacks = (PDWORD)( pNewImageBase + pTlsDir->AddressOfCallBacks - pNtHeader->OptionalHeader.ImageBase);    *// - -! 突然来个什么 Va.*  
           
         **for** ( **int** i = 0; pCallBacks\[i\]; i++)  
         {  
             pCallBacks\[i\] += pCallBacks\[i\] >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;    *//这里应该也是 VA 吧~不过懒得管他了*  
         }  
   
         PDWORD pAddressIndexs = (PDWORD)( pNewImageBase + pTlsDir->AddressOfIndex - pNtHeader->OptionalHeader.ImageBase);    *//VAVAVA~~~~~~*  
         **for** ( **int** i = 0; pAddressIndexs\[i\]; i++)  
         {  
             pAddressIndexs\[i\] += pAddressIndexs\[i\] >= iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase ? iExpandVsize : 0;    *//这里应该也是 VA 吧~不过懒得管他了*  
         }  
           
     }  
       
     *//IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG    //这个结构里面没有地址.*  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG\].VirtualAddress +=   
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
     }  
   
     *//IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT    //这个东西还算好搞吧*  
     **if** ( pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT\].VirtualAddress )  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT\].VirtualAddress +=  
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         *//PIMAGE\_BOUND\_IMPORT\_DESCRIPTOR*  
         *//这个里面的地址都是相对 pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT\].VirtualAddress 的地址,所以..高级编译器编译的不用管*  
     }  
   
   
     *//IMAGE\_DIRECTORY\_ENTRY\_IAT*  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IAT\].VirtualAddress +=  
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IAT\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
     }  
   
     *//IMAGE\_DIRECTORY\_ENTRY\_DELAY\_IMPORT*  
     *//反正我不懂这个结构.*  
     *// - -! 前面有勉强算是懂点 这个是真的...*  
     *//别来句 "这个可以懂." :)*  
   
     *//*  
     *//这个结构..我眼神不好.竟然在头文件没找到  - -~*  
     */\**  
 *    ->Delay Import Directory*  
 *    1. DelayImportDescriptor:*  
 *    grAttrs:          0x00000001  (dlattrRva)*  
 *    DLLName (R)VA:    0x00066160  ("MSIMG32.dll")*  
 *    Hmod (R)VA:       0x00069CE0*  
 *    IAT (R)VA:        0x00069000*  
 *    INT (R)VA:        0x0006618C*  
 *    BoundIAT (R)VA:   0x00000000*  
 *    UnloadIAT (R)VA:  0x00000000*  
 *    TimeDateStamp:    0x00000000  (GMT: Thu Jan 01 00:00:00 1970)*  
 *    \*/*  
     **typedef** **struct** \_IMAGE\_DELAY\_IMPORT\_DESCRIPTOR  
     {  
         DWORD grAttrs;  
         DWORD DLLName;  
         DWORD Hmod;  
         DWORD IAT;  
         DWORD INT;  
         DWORD BoundIAT;  
         DWORD UnloadIAT;  
         DWORD TimeDateStamp;  
     }IMAGE\_DELAY\_IMPORT\_DESCRIPTOR, \*PIMAGE\_DELAY\_IMPORT\_DESCRIPTOR;  
   
     **if** ( pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DELAY\_IMPORT\].VirtualAddress )  
     {  
           
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DELAY\_IMPORT\].VirtualAddress +=  
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DELAY\_IMPORT\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         PIMAGE\_DELAY\_IMPORT\_DESCRIPTOR pDelayDes = (PIMAGE\_DELAY\_IMPORT\_DESCRIPTOR)( pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DELAY\_IMPORT\].VirtualAddress);  
         **int** iTheBreakVA = iTheBreakRVA + pNewNtHeader->OptionalHeader.ImageBase;  
         **int** nCount = pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_DELAY\_IMPORT\].Size / **sizeof**(IMAGE\_DELAY\_IMPORT\_DESCRIPTOR);  
         **while** ( nCount--)  
         {  
             pDelayDes\[nCount\].BoundIAT += pDelayDes\[nCount\].BoundIAT >= iTheBreakVA ? iExpandVsize : 0;  
             pDelayDes\[nCount\].DLLName += pDelayDes\[nCount\].DLLName >= iTheBreakVA ? iExpandVsize : 0;  
             pDelayDes\[nCount\].Hmod += pDelayDes\[nCount\].Hmod >= iTheBreakVA ? iExpandVsize : 0;    *//!!*  
             pDelayDes\[nCount\].IAT += pDelayDes\[nCount\].IAT >= iTheBreakVA ? iExpandVsize : 0;  
             pDelayDes\[nCount\].INT += pDelayDes\[nCount\].INT >= iTheBreakVA ? iExpandVsize : 0;  
             pDelayDes\[nCount\].UnloadIAT += pDelayDes\[nCount\].UnloadIAT >= iTheBreakVA ? iExpandVsize : 0;  
         }  
     }  
   
     *//COM*  
     *//IMAGE\_DIRECTORY\_ENTRY\_COM\_DESCRIPTOR*  
     *//*  
     **if** ( pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_COM\_DESCRIPTOR\].VirtualAddress )  
     {  
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_COM\_DESCRIPTOR\].VirtualAddress +=  
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_COM\_DESCRIPTOR\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         *//日了..不支持 COM 了 !*  
           
     }  
   
     *// - -! 终于完了~~*  
     *// 我晕..我真是白痴..还有重定位..*  
   
     **if** ( !pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress )    *//如果没有重定位表,就无法修正代码里的绝对引用 - -~*  
     {  
         **goto** Failed;  
     }  
       
     *//IMAGE\_DIRECTORY\_ENTRY\_BASERELOC*  
     {  
   
         pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress +=  
             pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress >= iTheBreakRVA  
             ? iExpandVsize : 0;  
   
         PIMAGE\_BASE\_RELOCATION pBaseRelocal = (PIMAGE\_BASE\_RELOCATION)( pNewImageBase + pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress);  
         **int** iCount = pNewNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].Size / **sizeof**(IMAGE\_BASE\_RELOCATION);  
         **while** ( pBaseRelocal->VirtualAddress && iCount)  
         {  
             **typedef** **struct**    
             {  
                 WORD Offset:12;  
                 WORD Type:4;  
             }WORD\_RELOCAL, \*PWORD\_RELOCAL;  
   
             **if** ( pBaseRelocal->VirtualAddress >= iTheBreakRVA)  
             {  
   
                 pBaseRelocal->VirtualAddress += iExpandVsize;  
                 PWORD\_RELOCAL pRelocalWord = (PWORD\_RELOCAL)((PBYTE)pBaseRelocal + **sizeof**(IMAGE\_BASE\_RELOCATION));  
                 **for** ( **int** i = 0; i < pBaseRelocal->SizeOfBlock / **sizeof**(WORD\_RELOCAL); i++)  
                 {  
   
                     \*(PDWORD)(pNewImageBase + pBaseRelocal->VirtualAddress + pRelocalWord\[i\].Offset) +=  
                         pRelocalWord\[i\].Type == IMAGE\_REL\_BASED\_HIGHLOW && pRelocalWord\[i\].Offset  
                         ? iExpandVsize : 0;  
                 }  
             }  
   
             pBaseRelocal = (PIMAGE\_BASE\_RELOCATION)( (PBYTE)pBaseRelocal + pBaseRelocal->SizeOfBlock);  
   
         }  
           
     }  
       
   
     **return** **true**;  
   
   
 Failed:  
     *//释放内存啥的 - -~*  
     **return** **false**;  
   
 }