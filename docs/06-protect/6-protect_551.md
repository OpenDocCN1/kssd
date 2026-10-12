
# DriverUpx 的设计与探索（一）（二）

> 原文：[`www.pediy.com/kssd/pediy10/58229.html`](https://www.pediy.com/kssd/pediy10/58229.html)

能够支持 Driver 加壳的软件还是很少很少的。

貌似除了 `vmp` `cv` 之外就没有了。

`vmp` `cv` 都是基于代码加密理念的，因此移植到驱动下面不是什么困难的事情。

驱动加密壳实现了代码加密已经足够，毕竟公开的驱动 dump 高端内存 dump 到文件的东东偶还没看见过。

偶想实现的东东是 `DriverUpx`，众所周知 `upx` 是把 `pe` 文件完整压缩了保存起来，从而可以实现完美自身脱壳回来。

偶也想实现这个功能：申请 1 块内存，将驱动文件完整释放，自己处理重定位、输入表。

最后建立线程执行 `driverentry`。

每个加壳软件都有他关键的几个 api：

- `GetModuleHandle`
- `GetProcAddress`
- `LoadLibrary`

我们将一一实现他们。

先别急着实现 `stub`，我们先的准备好 `packer`。

`packer` 的目的是压缩 `sys` 文件，并且组装到 `stub` 指定地方。

驱动有个东西必须处理。

驱动程序被加壳后必须重新进行校验和的计算，否则加壳后的驱动不能加载。

这个功能我们在 `r3` 下面完成。

代码:

```c
/\*++

Routine Description:

   Calculates a new checksum for the PE image by calling imagehlp.dll

Arguments:

   szPeFile - PE file name

Return Value:

   void

\--\*/
void
CalcChecksum(
   char *szPeFile
   )
{
   DWORD            dwHeaderSum = 0;
   DWORD            dwCheckSum = 0;
   HANDLE           hFile;
   DWORD            cb;
   IMAGE_DOS_HEADER dosHdr;
   IMAGE_NT_HEADERS ntHdr;

   //
   // Open the file and calculate the CheckSum
   //
   if( MapFileAndCheckSum(szPeFile, &dwHeaderSum, &dwCheckSum) != CHECKSUM_SUCCESS )
   {
      printf("Failed to open specified PE file!\n");
      return;
   }
   hFile = CreateFile( szPeFile,
                 GENERIC_READ | GENERIC_WRITE,
                 FILE_SHARE_READ | FILE_SHARE_WRITE,
                 NULL,
                 OPEN_EXISTING,
                 0,
                 NULL
                 );
   if( hFile == INVALID_HANDLE_VALUE )
   {
      printf("Failed to open specified PE file!\n");
      return;
   }

   //
   // Seek to the beginning of the file
   //
   SetFilePointer( hFile, 0, 0, FILE_BEGIN );

   //
   // Read in the DOS header
   //
   if( (ReadFile(hFile, &dosHdr, sizeof(dosHdr), &cb, 0) == FALSE)
      || (cb != sizeof(dosHdr)) )
   {
      printf("Failed to read DOS header!\n");
      CloseHandle(hFile);
      return;
   }

   //
   // Seek the PE header
   //
   if( (dosHdr.e_magic != IMAGE_DOS_SIGNATURE) ||
      (SetFilePointer(hFile, dosHdr.e_lfanew, 0, FILE_BEGIN) == -1L) )
   {
      printf("Failed to read NT header!\n");
      CloseHandle(hFile);
      return;
   }

   //
   // Read in the NT header
   //
   if( (!ReadFile(hFile, &ntHdr, sizeof(ntHdr), &cb, 0))
      || (cb != sizeof(ntHdr)) )
   {
      printf("Failed to read NT header!\n");
      CloseHandle(hFile);
      return;
   }

   //
   // Search the PE sisnature
   //
   if(ntHdr.Signature != IMAGE_NT_SIGNATURE)
   {
      printf("The file is not a valid PE file!\n");
      CloseHandle(hFile);
      return;
   }

   //
   // Check if the PE file's checksum need adjusted
   //
   if(ntHdr.OptionalHeader.CheckSum == dwCheckSum)
   {
      printf("The PE file CheckSum needn't to be adjusted\n");
      CloseHandle(hFile);
      return;
   }

   //
   // Seek the PE header
   //
   if( SetFilePointer(hFile, dosHdr.e_lfanew, 0, FILE_BEGIN) == -1L )
   {
      printf("Failed to locate PE header!\n");
      CloseHandle(hFile);
      return;
   }

   printf("Old Checksum = 0x%08X\n", ntHdr.OptionalHeader.CheckSum);
   printf("New Checksum = 0x%08X\n", dwCheckSum);

   //
   // Modify the CheckSum
   //
   ntHdr.OptionalHeader.CheckSum = dwCheckSum;
   if( !WriteFile(hFile, &ntHdr, sizeof(ntHdr), &cb, NULL) )
   {
      printf("Failed to Adjust Checksum!\n");
   }
   else
   {
      printf("Adjust Checksum successfully!\n");
   }

   CloseHandle(hFile);
   return;
}
```

2.接下来才是我们的关键部分 `stub` 的制作。

参考驱动壳编写总结。

引用: 在给程序加壳的时候一般都要添加新节，用于存放壳的代码，应用层程序的节表的最后一项和第一个节之间一般是有一个很大的空间可以用来添加新的节表项的，但一般情况下驱动程序节表的最后一项后面紧接着就是第一个节，根本没有足够的 `0x28` 大小的空间存放新的节表项。解决的方法有两种，第一种将所有的节向后移动，而第二种方法则是将 `PE` 头整体向前移动覆盖掉部分无用的 `dos` 头，留出足够的空间存放新的节表项。

不过偶不想这么做，原因：驱动文件毕竟比较“珍贵”，偶不想乱动，怕蓝啊。

偶还是把驱动完整的压缩起来，释放到 1 片新申请的内存。

这样偶不用处理复杂的区段 `VA` 转换 `Reloc` 修改了。

### 关键 API 编写

其实老 `V` 的 `reloadandcall` 已经给我们很多必须代码了。

另外老 `V` 代码的 `GetModuleHandle（NULL）` 其实应该返回驱动本身基址，其实加载基址可以在驱动 `driverenter` 的参数中获得。

驱动对象可以从堆栈当中找到，将驱动对象通过 `DRIVEROBJECT` 的 `DriverSection` 成员遍历这个链表，是能够获得的。

代码:

```c
PVOID GetModuleHandle(char *pModuleName)
{
    PVOID pModuleBase = NULL;
    ULONG i;

    PSYSTEM_MODULE_INFORMATION ModulesInfo = (PSYSTEM_MODULE_INFORMATION)GetSysInf(SystemModuleInformation);
    if (ModulesInfo == NULL)
        return NULL;

    if (!strcmp(pModuleName, "ntoskrnl.exe"))
    {
        pModuleBase = (PVOID)ModulesInfo->aSM[0].Base;
    } else {
        for (i = 0; i < ModulesInfo->uCount; i++)
        {
            if (strstr(ModulesInfo->aSM.ImageName, pModuleName))
                pModuleBase = (PVOID)ModulesInfo->aSM.Base;
        }
    }

    ExFreePool(ModulesInfo);

    return pModuleBase;
}

PVOID GetProcAddress(PVOID ModuleBase, char *pFunctionName)
{
    PVOID pFunctionAddress = NULL;
    PIMAGE_EXPORT_DIRECTORY exports;
    ULONG i, addr, ord, size = 0;
    PULONG names, functions;
    PSHORT ordinals;

    if (ModuleBase == NULL)
        return NULL;

    __try
    {
        exports = (PIMAGE_EXPORT_DIRECTORY)RtlImageDirectoryEntryToData(ModuleBase,
            TRUE, IMAGE_DIRECTORY_ENTRY_EXPORT, &size);

        addr = (ULONG)exports - (ULONG)ModuleBase;

        functions = (PULONG)((ULONG)ModuleBase + exports->AddressOfFunctions);
        ordinals  = (PSHORT)((ULONG)ModuleBase + exports->AddressOfNameOrdinals);
        names     = (PULONG)((ULONG)ModuleBase + exports->AddressOfNames);

        for (i = 0; i < exports->NumberOfNames; i++)
        {
            ord = ordinals;

            if (i >= exports->NumberOfNames || ord >= exports->NumberOfFunctions)
                return NULL;

            if (functions[ord] < addr || functions[ord] >= addr + size)
            {
                if (strcmp((char *)((ULONG)ModuleBase + names), pFunctionName)  == 0)
                {
                    pFunctionAddress =(PVOID)((ULONG)ModuleBase + functions[ord]);
                    break;
                }
            }
        }
    }
    __except(EXCEPTION_EXECUTE_HANDLER)
    {
        DbgMsg("KernelGetProcAddress() EXEPTION\n");
        pFunctionAddress = NULL;
    }

    return pFunctionAddress;
}

PVOID GetSysInf(SYSTEMINFOCLASS pdData)
{
    NTSTATUS ns;
    ULONG dSize = 4096;
    ULONG dData = 0;
    PVOID shi;

    do {
        shi = ExAllocatePool(NonPagedPool, dSize);

        if (shi == NULL) return 0;

        ns = ZwQuerySystemInformation(pdData, shi, dSize, &dData);

        if (ns == STATUS_INFO_LENGTH_MISMATCH)
        {
            ExFreePool(shi);
            dSize *= 2;
        }

    } while (ns != 0);

    return shi;
}
```

`LoadLibrary` 貌似不常用，原因之一是常用模块基本上都是已经加载过了，`GetModuleHandle` 肯定能获得其基址，貌似不需要 `LoadLibrary`，单独调用其他系统默认不加载的模块的 `sys` 偶真没见过。

3.申请一片大小为原始文件 `SizeOfImage` 的内存，用来存放我们的释放体。

这个不知道是不是需要必须是 `nonpagedpool`？

实现重定位表处理、输入表处理代码:



```c
BOOLEAN ProcessImports(ULONG ImageBase)
{
    IMAGE_THUNK_DATA32 *pThunk;
    PVOID LibAddr;
    char *LibName, *FuncName;
    PIMAGE_IMPORT_BY_NAME pImageImportByName;
    ULONG FuncAddr;

    __try
    {
        PIMAGE_NT_HEADERS pImageNtHeaders = (PIMAGE_NT_HEADERS)
            (ImageBase + ((PIMAGE_DOS_HEADER)ImageBase)->e_lfanew);

        PIMAGE_IMPORT_DESCRIPTOR pImageImportDescriptor = (PIMAGE_IMPORT_DESCRIPTOR)(RVATOVA(ImageBase,
            pImageNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress));

        while (pImageImportDescriptor->Name != 0)
        {
            LibName = (char *)RVATOVA(ImageBase, pImageImportDescriptor->Name);
            LibAddr = KernelGetModuleBase(LibName);
            if (LibAddr == NULL)
                return FALSE;

            DbgMsg("0x%.8x:%s\n", LibAddr, LibName);

            pThunk = (IMAGE_THUNK_DATA32 *)RVATOVA(ImageBase, pImageImportDescriptor->FirstThunk);

            while (pThunk->u1.Ordinal != 0)
            {
                pImageImportByName = (PIMAGE_IMPORT_BY_NAME)RVATOVA(ImageBase, pThunk->u1.AddressOfData);
                FuncName = (char *)(&pImageImportByName->Name);
                FuncAddr = (ULONG)KernelGetProcAddress(LibAddr, FuncName);

                DbgMsg(" 0x%.8x:%s\n", FuncAddr, FuncName);

                if (FuncAddr == 0)
                    return FALSE;
                
                *(PULONG)pThunk = FuncAddr;
                pThunk++;
            }

            pImageImportDescriptor++;
        }
    }
    __except(EXCEPTION_EXECUTE_HANDLER)
    {
        DbgMsg("ProcessImports() EXEPTION\n");
        return FALSE;
    }

    return TRUE;
}

BOOLEAN ProcessRelocs(ULONG ImageBase)
{
    __try
    {
        PIMAGE_NT_HEADERS pImageNtHeaders = (PIMAGE_NT_HEADERS)
            (ImageBase + ((PIMAGE_DOS_HEADER)ImageBase)->e_lfanew);

        ULONG RellocsSize = pImageNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].Size;

        PIMAGE_BASE_RELOCATION pImageBaseRelocation = (PIMAGE_BASE_RELOCATION)(RVATOVA(ImageBase,
            pImageNtHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress));

        PIMAGE_BASE_RELOCATION pRelocation = pImageBaseRelocation;

        ULONG ImageBaseDelta = ImageBase - pImageNtHeaders->OptionalHeader.ImageBase, Number, i, Size = 0;
        PUSHORT Rel;

        DbgMsg("pImageBaseRelocation: 0x%.8x; Size: %d\n", pImageBaseRelocation, RellocsSize);

        while (RellocsSize > Size)
        {
            Size += pRelocation->SizeOfBlock;

            Number = (pRelocation->SizeOfBlock - 8) / 2;
            Rel = (PUSHORT)((ULONG)pRelocation + 8);

            DbgMsg("VirtualAddress: 0x%.8x; Number of Relocs: %d; Size: %d\n", pRelocation->VirtualAddress,
                Number, pRelocation->SizeOfBlock);

            for (i = 0; i < Number - 1; i++)
            {
                DbgMsg(" %d:0x%.8x\n", i, Rel & 0x0FFF);
                *(PULONG)(RVATOVA(ImageBase, pRelocation->VirtualAddress + (Rel & 0x0FFF))) += ImageBaseDelta;
            }

            pRelocation = (PIMAGE_BASE_RELOCATION)((ULONG)pImageBaseRelocation + Size);
        }
    }
    __except(EXCEPTION_EXECUTE_HANDLER)
    {
        DbgMsg("ProcessRelocs() EXEPTION\n");
        return FALSE;
    }

    return TRUE;
}
```

其实多数代码都是从老 V 的 `reload` 里面复制过来的

其实一个简单的壳也就是自己实现 `PE` 文件加载运行的过程

下面这一步有点麻烦

我们已经把原始驱动文件放在了不分页内存里面

并且处理好了重定位 输入表

下面是启动这个驱动了

这个驱动对象 `DriverOblect` 偶不是很明白

不知道可否自己分配 1 个 还是只能继承系统分配下来的这一个

偶目前做的是直接 `PsCreateSystemThread(DriverEntry)`

使用系统传过来的

`IN PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath`

发送下去

不知道这个 `RegistryPath` 可否自己“乱改”呢？ `DriverObject` 可否自己分配？

先写到这里

休息去

看来 简单的事情 比最麻烦的事情 还麻烦

一些重定位 输入表处理的东西

还有 `getmodulehandle` 函数模拟反而比较简单

倒是 1 个代码解压到内存运行的过程总是出问题？ `why`？

以下代码参考 `linuxer` 的虚拟机

代码:

```c
BOOL PELoader(char *lpStaticPEBuff, long lStaticPELen)
{
  long lSectionNum;
  IMAGE_SECTION_HEADER *pISH;
  long lFileAlignMask;
  long lSectionAlignMask;
  int nIndex;
      PIMAGE_FILE_HEADER    pfh;
    PIMAGE_OPTIONAL_HEADER    poh;
    PIMAGE_SECTION_HEADER    psh;
    
  long lPESignOffset = *(long *)(lpStaticPEBuff + 0x3c);
  IMAGE_NT_HEADERS *pINH = (IMAGE_NT_HEADERS *)(lpStaticPEBuff + lPESignOffset);

  //取加载到内存中大小
  long lImageSize = pINH->OptionalHeader.SizeOfImage;
  char *lpDynPEBuff = ExAllocatePool(NonPagedPool,lImageSize);
  dprintf("[dvupx] peoep is %8X\n",lpDynPEBuff);

//  pePool=ExAllocatePool(NonPagedPool,  poh->SizeOfImage);

  memset(lpDynPEBuff, 0, lImageSize);

  //取 PE 文件的节数量
  lSectionNum = pINH->FileHeader.NumberOfSections;

  pISH = (IMAGE_SECTION_HEADER *)((char *)pINH + sizeof(IMAGE_NT_HEADERS));
  
  //加载 PE 文件第一个节前的信息
  memcpy(lpDynPEBuff, lpStaticPEBuff, pISH->VirtualAddress);

  //加载各个节
   lFileAlignMask = pINH->OptionalHeader.FileAlignment - 1;        //各节在磁盘中的对齐掩码
   lSectionAlignMask = pINH->OptionalHeader.SectionAlignment - 1;  //各节在 load 后内存中的对齐掩码
   
  for( nIndex = 0; nIndex < lSectionNum; nIndex++, pISH++)
  {
    /****************************************************************
    在对 FSG1.33 加壳程序进行 loader 的时候，其节对齐属性
    比较怪异，对 PE 文件接触时间不长，不过地方不知道为什么，
    请各位大虾指点迷津，故注掉节属性验证代码
    ****************************************************************/
    //判定各节的对齐属性，合法不
    /*if((pISH->VirtualAddress & lSectionAlignMask) || (pISH->SizeOfRawData & lFileAlignMask))
    {
      //出现非法节
      delete lpDynPEBuff;
      return false;
    }*/

    //加载改节
    memcpy(lpDynPEBuff + pISH->VirtualAddress, lpStaticPEBuff + pISH->PointerToRawData, pISH->SizeOfRawData);
  }
  ProcessImports((ULONG)lpDynPEBuff);
  ProcessRelocs((ULONG)lpDynPEBuff,(ULONG)lpDynPEBuff);
  GetHeaders(lpDynPEBuff,&pfh,&poh,&psh);
  dprintf("[dvupx] peoep is %8X\n",poh->AddressOfEntryPoint+lpDynPEBuff);
  peoep=(poh->AddressOfEntryPoint+(DWORD)lpDynPEBuff);
  dprintf("[dvupx] peoep is %8X\n",peoep);


  return 1;
}
```

加载结果

`00000000    0.00000000    [dvupx] Loaded     `

`00000001    0.00029277    [dvupx] peloadaddr is 81644C48     `

`00000002    0.00158456    [dvupx] peoep is 81644E48`

使用 `windbg` 查看 代码正确

`lkd> dd 81644e48`

`81644e48  b8ec8b55 c0000182 0008c2c9 00000000`

`81644e58  00000000 00000000 00000000 00000000`

`81644e68  00000000 00000008 00000000 00000000`

`81644e78  00000000 00000000 00000000 00000000`

`81644e88  0a040049 74726f50 00050005 00000000`

`81644e98  81644e98 81644e98 7fffffff e182d7e8`

`81644ea8  0a060004 ee657645 00000001 00000001`

`81644eb8  817babf8 00000000 80561580 00000000`

`lkd> u 81644e48`

`81644e48 55               push    ebp`

`81644e49 8bec             mov     ebp,esp`

`81644e4b b8820100c0       mov     eax,0C0000182h`

`81644e50 c9               leave`

`81644e51 c20800           ret     8`

`81644e54 0000             add     byte ptr [eax],al`

`81644e56 0000             add     byte ptr [eax],al`

`81644e58 0000             add     byte ptr [eax],al`

但是 1 运行，

`pehex` 为被加壳文件的解开的文件镜像

执行    `PELoader(pehex,sizeof(pehex));`

然后

`push pRegistryString`

`push pDriverObj`

`call peoep`

继续在这里写好了 

两边的接口问题都解决了 

下面应该是考虑压缩算法了 

不过这个偶真的没有发言权 还是大牛们来评论吧 

`aPLib` 和 `LZMA` 应该是最常用的 2 个选择了 

引用: 由 `dwing` 发布  

论压缩率大致是这样的:  

`zip` < `RAR16` < `aPLib` < `RAR32` < `LZMA` < `PPM`  

`TeLeMan`：  

`aPlib` 只是属于传统 `LZ` 方式，一般不会比 `LZH` 的高，所以 `aPlib` 在上面几种里压缩率是最低的。  

`dwing`：  

`aPlib` 压缩率不是最低的,因为使用的字典比 `zip` 大,匹配算法比 `zip` 完善(但比较慢).  

`LZH` 压缩率与 `zip` 差不多.  

`TeLeMan`：  

`aPlib` 之所以比 `LZH` 和 `zip` 压缩率要小，是因为没有用到 `Huffman` 编码。这一点不需要有什么质疑的，举个现实的例子就可以了。`ASPack` 就是用 `LZH` 算法，要比 `UPX` 和只用到 `aPlib` 库的 `Packer` 压缩率大。  

 `dwing` 

你说的也有道理,经过测试发现压缩较小的文件 `aPLib` 有优势,较大的文件使用 `Huffman` 算法的有优势,前者的优势只是较大的字典和更好的匹配算法.  

`LZMA` 则发挥了 `LZ` 算法最大的潜能,使用最完善的匹配算法+算术编码(比 `Huffman` 更好)  

理论偶就不仔细说了 因为偶也不懂  

`aplib` 

引用: 压缩函数  

`size_t aP_pack( const void *source,`



### API 函数说明

以下函数用于数据的压缩与解压缩处理。

#### 函数：aPsafe_pack

```c
size_t aPsafe_pack( const void *source, 
                    void *destination, 
                    size_t length, 
                    void *workmem, 
                    int (*callback)(size_t, size_t, size_t, void *), 
                    void *cbparam );
```

Wrapper 函数，针对 `aP_pack`，在压缩数据前添加一个头，包含原始数据的长度以及原始数据和压缩数据的 CRC32 校验和。

**参数：**

- `source` - 指向待压缩数据的指针。
- `destination` - 指向压缩数据存储位置的指针。
- `length` - 未压缩数据的字节长度。
- `workmem` - 指向压缩过程中使用的工作内存指针。
- `callback` - 指向回调函数的指针（或 `NULL`）。
- `cbparam` - 回调参数。

#### 函数：aPsafe_depack

```c
size_t aPsafe_depack( const void *source, 
                     size_t srclen, 
                     void *destination, 
                     size_t dstlen );
```

Wrapper 函数，针对 `aP_depack_asm_safe`，检查压缩数据的 CRC32，执行解压缩，并检查解压缩后数据的 CRC32。

**参数：**

- `source` - 指向压缩数据的指针。
- `srclen` - 源缓冲区的大小（字节）。
- `destination` - 指向解压缩数据存储位置的指针。
- `dstlen` - 目标缓冲区的大小（字节）。

**返回值：**

- 解压缩数据的长度，或在出错时返回 `APLIB_ERROR`。

### 解压缩函数

与之对应的解压缩函数：

```c
size_t aP_depack( const void *source, 
                  void *destination );
```

将 `source[]` 中的压缩数据解压缩到 `destination[]` 中。

`destination[]` 缓冲区必须足够大以容纳解压缩后的数据。

**参数：**

- `source` - 指向压缩数据的指针。
- `destination` - 指向解压缩数据存储位置的指针。

**返回值：**

- 解压缩数据的长度，或在出错时返回 `APLIB_ERROR`。

### 使用示例

其实这个过程一点也不复杂。

如下代码中 `data` 为原始数据，`compressed` 为压缩后的数据：

```c
/* allocate workmem and destination memory */
char *workmem    = malloc(aP_workmem_size(length));
char *compressed = malloc(aP_max_packed_size(length));

/* compress data[] to compressed[] */
size_t outlength = aPsafe_pack(data, compressed, length, workmem, NULL, NULL);

/* if APLIB_ERROR is returned, and error occured */
if (outlength == APLIB_ERROR)
{
   printf("An error occured!\n");
} else {
   printf("Compressed %u bytes to %u bytes\n", length, outlength);
}

//－－－－－－－－－－－－－－－－－－－－－－－－－－－

/* get original size */
size_t orig_size = aPsafe_get_orig_size(compressed);

/* allocate memory for decompressed data */
char *data = malloc(orig_size);

/* decompress compressed[] to data[] */
size_t outlength = aPsafe_depack(compressed, compressed_size, data, orig_size);

/* check decompressed length */
if (outlength != orig_size)
{
   printf("An error occured!\n");
} else {
   printf("Decompressed %u bytes\n", outlength);
}
```

压缩过程没有什么好说的，选择 1 个算法处理即可。

另外，前面有几个小问题的专门处理需要注意：

### 驱动退出处理

驱动退出的时候，如果注册了 `DriverUnload` 函数，那么就会去调用。

```c
VOID DriverUnload(PDRIVER_OBJECT pDriverObj)
```

我们需要将这个函数设置成壳的处理函数，先执行完被加壳程序的 `DriverUnload`，然后再 `ExFreePool`。

这里驱动有 2 种：

1. 正常驱动：注册了 `DriverUnload` 函数的，我们照上面处理即可。
2. 流氓驱动：没有注册 `DriverUnload` 函数，我们此时不能注册壳的 `DriverUnload` 处理函数，不然也许一些代码还在运行，你就 `ExFreePool` 了，那么你只能得到蓝屏。

我们需要 1 个函数 `MmIsAddressValid` 来验证 `DriverUnload` 处理函数是否被注册了。

在 `DriverEntry` 里面加入：

```c
if (MmIsAddressValid(pDriverObj->DriverUnload))
{
    OldDriverUnload=pDriverObj->DriverUnload;
    pDriverObj->DriverUnload = DriverUnload;
}
```

相应的，我们先判断 `OldDriverUnload` 是否有效，如果是正常驱动，那么去执行他，完成以后 `ExFreePool` 掉空间。

如果 `OldDriverUnload` 函数无效，那么我们只好什么也不做了。

```c
VOID DriverUnload(PDRIVER_OBJECT pDriverObj)
{     
    if (MmIsAddressValid(OldDriverUnload))
    {
        _asm
        {
            push pDriverObj
            call OldDriverUnload
        }
        ExFreePool(lpDynPEBuff);
    }
    dprintf("[dvupx] Unloaded\n");
}
```

基本上到这里，1 个半成品（偶只写完了解压部分）就应该差不多了，其实这个东西娱乐意义远远大于实用意义。
