

# "菜壳"重建导入表代码，极垃圾，高手免进

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-595.htm`](https://www.pediy.com/kssd/pediy09/pediy09-595.htm)

> 【  标题  】 发段"菜壳"重建导入表代码  
> 【  作者  】 linxer  
> 【  Q Q   】 3568599  
> 【  声明  】 俺系初级选手，高手略过。失误之处敬请诸位大侠赐教！  
>   
> 以下代码思路来源 ollydump 插件源码，在此向 ollydump 作者表示至敬！  
>   
> 以下代码系本人最近虚拟脱壳后，为 dump 所写，在 UPX，ASPACK 上调试而成，今天又在 PEcompact 2x 上看到效果，应该对一些"菜壳"具有一定通用性......  
>   
> 以下代码，由于有些函数没有公示，不能执行，不过对一些和我一样菜的朋友了解，怎么搜索 API 调用信息和重建"菜壳"导入表，有帮助吧，当然你也可以直接查阅 ollydump 源码  
>   
>   
> 1.一些宏定义  
>   
> #define u8 unsigned char  
> #define s8 char  
>   
> #define u16 unsigned short int  
> #define s16 short int  
>   
> #define u32 unsigned int  
> #define s32 int  
>   
> 2.dll 模块信息及其 API 信息登记表结构  
>   
> //解壳过程中 IAT 处理，导入函数信息登记表结构  
> typedef struct tagKillShell\_iat\_fun\_info  
> {  
>   s32          nFunID;      //函数 ID  
>   s32          nRVA;      //本函数在 IAT 表中的 RVA  
>   s32          nFunSerial;    //函数序号  
>   s8          pFunName\[32\];    //函数名称  
>   struct tagKillShell\_iat\_fun\_info \*   next;      //下一个函数地址  
> }KillShell\_iat\_fun\_info, \*PKillShell\_iat\_fun\_info;  
>   
>   
> //解壳过程中 IAT 处理，导入到模块信息登记表结构  
> typedef struct tagKillShell\_iat\_module\_info  
> {  
>   s32          nModuleID;    //模块 ID  
>   s32          nIAT;      //其在 IAT 中位置  
>   s32          nFunIDBegin;    //本模块导出函数起始 ID  
>   s32          nFunIDEnd;    //本模块导出函数结束 ID  
>   s8          pModuleName\[32\];  //模块名称  
>   struct tagKillShell\_iat\_module\_info \*   next;      //下一个模块地址  
>   PKillShell\_iat\_fun\_info      pFunInfo;    //函数信息  
> }KillShell\_iat\_module\_info, \*PKillShell\_iat\_module\_info;  
>   
>   
> 3.与模块有关的两个辅助函数  
>   
> PKillShell\_iat\_module\_info g\_pReBuildInfo = NULL;  //解壳后，扫描 PE 文件发现的 API 调用信息  
>   
>   
> //查询在重建 IAT 时对应模块信息  
> PKillShell\_iat\_module\_info search\_module\_info\_by\_nIAT(s32 ulVirAdd)  
> {  
>   PKillShell\_iat\_module\_info pModuleInfo;  
>     
>   for(pModuleInfo = g\_peDyncmicInfo.pReBuildInfo; NULL != pModuleInfo; pModuleInfo = pModuleInfo->next)  
>   {  
>     if(ulVirAdd == pModuleInfo->nIAT)  
>     {  
>       break;  
>     }  
>   }  
>   
>   return pModuleInfo;  
> }  
>   
> //获取模块数量  
> s32 get\_module\_count(PKillShell\_iat\_module\_info  pModuleInfo)  
> {  
>   s32 nCount = 0;  
>     
>   for(; NULL != pModuleInfo; pModuleInfo = pModuleInfo->next)  
>   {  
>     nCount++;  
>   }  
>   
>   return nCount;  
> }  
>   
>   
> 4.以下代码假定 g\_peLoad\_INH 是 PE 文件 IMAGE\_NT\_HEADERS 指针，并假定 malloc 总是能申请到内存  
>   
> 4.1 搜索 API 调用信息  
>   
> //搜索 API  
> //成功返回 0，否则-1  
> s32 find\_api\_call\_info(u8 \*lpExePEBuff, u32 nExePELen)  
> {  
>   PKillShell\_iat\_module\_info pModuleInfo;  
>   PKillShell\_iat\_module\_info pModuleIATInfo;  
>     
>   PKillShell\_iat\_fun\_info pFunInfo;  
>   PKillShell\_iat\_fun\_info pFunIATInfo;  
>   
>   s32 nSectionNum;  
>   u32 nImageBase;  
>   
>   u32 nVirAdd;  
>   s32 lAPIID;  
>   
>   u8 \* pBeginText;  
>   u8 \* pEndText;  
>   u8 \* pText;  
>   
>   IMAGE\_SECTION\_HEADER \* pISH;  
>   IMAGE\_SECTION\_HEADER \*pISHTemp;  
>     
>   u16 nInstructions\[2\] = {0x25ff, 0x15ff};   //用 jmp \[x\] / call \[x\]搜索 API 调用信息  
>   u16 nInstruction;  
>   
>   nSectionNum = g\_peLoad\_INH->FileHeader.NumberOfSections;  //节数量  
>   nImageBase = g\_peLoad\_INH->OptionalHeader.ImageBase;    //程序映射基址  
>   
>   pISH = (IMAGE\_SECTION\_HEADER \*)((s8 \*)g\_peLoad\_INH + sizeof(IMAGE\_NT\_HEADERS));  
>   
>   //开始搜索地址  
>   pBeginText = lpExePEBuff + pISH\[0\].VirtualAddress;  
>   
>   //结束搜索地址  
>   pEndText = lpExePEBuff + pISH\[nSectionNum - 1\].VirtualAddress + pISH\[nSectionNum - 1\].Misc.VirtualSize;  
>   
>   for(pText = pBeginText; pText < pEndText; pText++)  
>   {  
>     nInstruction = \*(u16 \*)pText;  
>     if(nInstruction == nInstructions\[0\] || nInstruction == nInstructions\[1\])  
>     {  
>       //取出指令操作数，它是一个虚拟地址，在这个虚拟地址中可能存放着 API 的 ID  
>       nVirAdd = \*(u32 \*)(pText + 2);  
>       if(nVirAdd < pISH\[0\].VirtualAddress + nImageBase || nVirAdd > nExePELen + nImageBase - 4)  
>       {  
>         //虚拟地址非法  
>         continue;  
>       }  
>       lAPIID = \*(s32 \*)(lpExePEBuff + (nVirAdd - nImageBase));  
>         
>         
>       if(0 == get\_iat\_api\_info(lAPIID, &pModuleInfo, &pFunInfo)) //get\_iat\_api\_info 函数效验 API 调用信息是否正常，没有展示其源码  
>       {  
>         pISHTemp = (IMAGE\_SECTION\_HEADER \*)rva\_to\_section(nVirAdd - nImageBase); //rva\_to\_section 这个函数，返回 RVA 所在节表地址  
>         if(NULL == pISHTemp)  
>         {  
>           return -1;  
>         }  
>   
>         //寻找这个 dll 在 IAT 中起始位置  
>         for(; 0 != lAPIID && (nVirAdd - nImageBase) >= pISHTemp->VirtualAddress; )  
>         {  
>           nVirAdd -= sizeof(IMAGE\_THUNK\_DATA);  
>           lAPIID = \*(s32 \*)(lpExePEBuff +(nVirAdd - nImageBase));  
>         }  
>         nVirAdd += sizeof(IMAGE\_THUNK\_DATA);  
>         lAPIID = \*(s32 \*)(lpExePEBuff +(nVirAdd - nImageBase));  
>   
>         /\*\*这里必须用其在 IAT 中起始项搜索，不能用 dll 名称，这是因为......，  
>         比如说，upx 壳，它导入 kernel32.dll 三次，对应有三个 IAT 起始地址，  
>         用 dll 名称搜索，修复的程序肯定不对\*/  
>         pModuleIATInfo = search\_module\_info\_by\_nIAT(nVirAdd- nImageBase);  
>         if(NULL != pModuleIATInfo)  
>         {  
>           continue; //这个 dll 已经处理  
>         }  
>           
>         //插入这个模块信息  
>         pModuleIATInfo = (PKillShell\_iat\_module\_info)malloc(sizeof(KillShell\_iat\_module\_info));  
>         memset((void \*)pModuleIATInfo, 0, sizeof(KillShell\_iat\_module\_info));  
>         \*pModuleIATInfo = \*pModuleInfo;  
>         pModuleIATInfo->nIAT = nVirAdd - nImageBase;  
>         pModuleIATInfo->next = NULL;  
>         pModuleIATInfo->pFunInfo = NULL;  
>   
>         //将本模块信息插入到 IAT 模块链表中  
>         pModuleIATInfo->next = g\_pReBuildInfo;  
>         g\_pReBuildInfo = pModuleIATInfo;  
>   
>         //处理这个 Dll 中的导入函数  
>         /\*\*这里必须这么做，是因为用 ff/2 ff/3 指令搜索不一定能找到 IAT 表中的每项，  
>         因此必须这么干，否则会出现 IAT 表中，有些项无法修复，比如 Aspack 中导入了 Sleep 函数，但是  
>         用 ff/2 ff/3 却搜不着，这样导致 IAT 中 Sleep 对应项空的，程序就无法执行了，load 不成功,  
>         所以对每个 dll 必须先找到其在 IAT 中起始地址，然后从这个地方出发搞定这个 dll 导入的所有函数\*/  
>         for(; 0 != lAPIID; )  
>         {  
>           if(0 != get\_iat\_api\_info(lAPIID, &pModuleInfo, &pFunInfo)) //这个函数说明见上  
>           {  
>             //没有办法,执行到这里的壳，脱后肯定不能运行，5555  
>             continue;  
>           }  
>             
>           pFunIATInfo = (PKillShell\_iat\_fun\_info)malloc(sizeof(KillShell\_iat\_fun\_info));  
>           memset((void \*)pFunIATInfo, 0, sizeof(KillShell\_iat\_fun\_info));  
>           \*pFunIATInfo = \*pFunInfo;  
>           pFunIATInfo->nRVA = nVirAdd - nImageBase;  
>           pFunIATInfo->next = NULL;  
>             
>           //插入到 IAT 模块函数链表中  
>           pFunIATInfo->next = pModuleIATInfo->pFunInfo;  
>           pModuleIATInfo->pFunInfo = pFunIATInfo;  
>             
>           nVirAdd += sizeof(IMAGE\_THUNK\_DATA);  
>           lAPIID = \*(s32 \*)(lpExePEBuff +(nVirAdd - nImageBase));  
>         }  
>       }  
>     }  
>   }  
>   
>   return 0  
> }  
>   
>   
> 4.2 重建导入表  
>   
>   
> //修复导入表  
> //返回 PE 文件新添长度，即新导入表长度  
> s32 rebuild\_imports(u8 \*lpExePEBuff, u32 nExePELen)  
> {  
>   s32 nOriLen;        //原 PE 文件加载后长度  
>   s32 nIIDLen;        //IID 表长  
>   s32 nOtherLen = 0;      //导入表中其它信息占用长度  
>   s32 nSectionNum;  
>   
>   s32 nNewSectionFileLen;      //新节磁盘中长度  
>   s32 nNewSectionMemoryLen;    //新节加到内存长度  
>   
>   s8 \* pOtherInfo;  
>   s8 \* pOtherInfoBegin;  
>   
>   IMAGE\_SECTION\_HEADER \* pISH;  
>   IMAGE\_SECTION\_HEADER \* pOldEndISH;  
>   
>   IMAGE\_IMPORT\_DESCRIPTOR \*pIID;  
>   
>   PKillShell\_iat\_module\_info pModuleInfo;    //dll 信息指针  
>   PKillShell\_iat\_fun\_info pFunInfo;    //API 信息指针  
>   
>   if-1 == find\_api\_call\_info(lpExePEBuff, nExePELen)) //find\_api\_call\_info 执行失败或者是没有找到 API  
>   {  
>     return 0;  
>   }  
>   
>   nSectionNum = g\_peLoad\_INH->FileHeader.NumberOfSections;              
>   
>   //加一个节(导入表用)  
>   pISH = (IMAGE\_SECTION\_HEADER \*) ((s8 \*)g\_peLoad\_INH + sizeof(IMAGE\_NT\_HEADERS));  
>   pOldEndISH = pISH + nSectionNum - 1;  
>   
>   nOriLen = pOldEndISH->VirtualAddress + pOldEndISH->Misc.VirtualSize;  
>   if(nOriLen != g\_peLoad\_INH->OptionalHeader.SizeOfImage)  
>   {  
>     return 0;  
>   }  
>   
>   //计算要新建导入表的 IID 表长  
>   nIIDLen = (get\_module\_count(g\_pReBuildInfo) + 1) \* sizeof(IMAGE\_IMPORT\_DESCRIPTOR);  
>   
>   //初始化 IID 表  
>   memset(lpExePEBuff + nOriLen, 0, nIIDLen);  
>     
>   //真正的重建新节(导入表)代码  
>   pIID = (IMAGE\_IMPORT\_DESCRIPTOR \*)(lpExePEBuff + nOriLen);  
>   pOtherInfo = (s8 \*)pIID + nIIDLen;  
>   pOtherInfoBegin = pOtherInfo;  //保存，求导入表除 IID 表长用到  
>     
>   for(pModuleInfo = g\_peDyncmicInfo.pReBuildInfo; NULL != pModuleInfo; pModuleInfo = pModuleInfo->next)  
>   {  
>     //FirstThunk 指向地址  
>     pIID->FirstThunk = pModuleInfo->nIAT;  
>   
>     //设置 Dll 名  
>     strcpy(pOtherInfo, pModuleInfo->pModuleName);  
>     pIID->Name = pOtherInfo - (s8 \*)lpExePEBuff;  
>     pOtherInfo += strlen(pModuleInfo->pModuleName) + 1;  
>   
>     //扫描 Dll 中导入函数  
>     for(pFunInfo = pModuleInfo->pFunInfo; NULL != pFunInfo; pFunInfo = pFunInfo->next)  
>     {  
>       if(pFunInfo->nRVA != 0)  
>       {  
>         //这里要注意的是,nFunSerial 和 pFunName 只有一个字段有效(脱壳中假定这样)  
>         if(0 != pFunInfo->nFunSerial)  
>         {  
>           //序号导入  
>           \*(s32 \*)(lpExePEBuff + pFunInfo->nRVA) = pFunInfo->nFunSerial | IMAGE\_ORDINAL\_FLAG;  
>         }  
>         else  
>         {  
>           IMAGE\_IMPORT\_BY\_NAME \* pIIBN = (IMAGE\_IMPORT\_BY\_NAME \*)pOtherInfo;  
>             
>           pIIBN->Hint = 0;  
>           strcpy((s8 \*)pIIBN->Name, pFunInfo->pFunName);  
>     
>           \*(s32 \*)(lpExePEBuff + pFunInfo->nRVA) = (u8 \*)pIIBN - lpExePEBuff;  
>           pOtherInfo += sizeof(pIIBN->Hint) + strlen(pFunInfo->pFunName) + 1;  
>         }  
>       }  
>     }  
>   
>     pIID++;  
>   }  
>   
>   //计算新节大小   
>   nNewSectionFileLen = nIIDLen + (pOtherInfo - pOtherInfoBegin);  
>   if(nNewSectionFileLen % g\_peLoad\_INH->OptionalHeader.FileAlignment != 0) //节对齐  
>   {  
>     s32 nTemp = nNewSectionFileLen;  
>     nNewSectionFileLen = (nNewSectionFileLen / g\_peLoad\_INH->OptionalHeader.FileAlignment + 1) \* \\  
>       g\_peLoad\_INH->OptionalHeader.FileAlignment;  
>     memset(lpExePEBuff + nOriLen + nTemp, 0, nNewSectionFileLen - nTemp);  
>   }  
>   
>   nNewSectionMemoryLen = nNewSectionFileLen;  
>   if(nNewSectionMemoryLen % g\_peLoad\_INH->OptionalHeader.SectionAlignment != 0)  
>   {  
>     nNewSectionMemoryLen = (nNewSectionMemoryLen / g\_peLoad\_INH->OptionalHeader.SectionAlignment + 1) \* \\  
>       g\_peLoad\_INH->OptionalHeader.SectionAlignment;  
>   }  
>   
>   //在节表中新加一个结(这里假定新加节所在位置没有有用数据，5555)  
>   pISH = pISH + nSectionNum;  
>   pISH->Characteristics = 0xC0000040;        //新节属性  
>   memcpy(pISH->Name, ".linxer\\0", 8);        //新节名称  
>   pISH->VirtualAddress = nOriLen;          //新节加到内存地址  
>   pISH->Misc.VirtualSize = nNewSectionMemoryLen;      //新节加到内存中长度  
>   pISH->PointerToRawData = nOriLen;        //新节在磁盘地址  
>   pISH->SizeOfRawData = nNewSectionFileLen;      //新节在磁盘中长度  
>   
>   g\_peLoad\_INH->OptionalHeader.DataDirectory\[1\].VirtualAddress = nOriLen;      //指向新导入表  
>   g\_peLoad\_INH->OptionalHeader.DataDirectory\[1\].Size = nNewSectionMemoryLen;    //新节大小  
>   
>   g\_peLoad\_INH->FileHeader.NumberOfSections += 1;     //节数量加 1  
>   
>   return nNewSectionFileLen;  
> }  
>   
>   
> 5.总结  
>   
> 不过感觉上面代码，在虚拟脱些"菜壳"，用来重建导入表，有些小题大作了，其实可以在脱壳过程中，记下一些如 dll 名称地址等信息，修复下就可以了，没有必要重建导入表的 

 4dd466d0e1e0cc123afbe504da2462ca