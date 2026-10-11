
# 精确区分 windows 内核 Image 文件

> 原文：[`www.pediy.com/kssd/pediy10/98017.html`](https://www.pediy.com/kssd/pediy10/98017.html)

在阅读本文前，先确定几个概念。

`FileName`(文件名)：即文件在硬盘上的名字，这个名字随着复制、改名，会变化。

`InternalName`(文件原始名)：在附加了`.rc`编译时，编译器会将`.rc`中的名字链接进去，这个名字，不会随着文件的复制改名而变化(用`PE`工具修改后再改`CheckSum`除外), 在`Windows`中，右击文件名->属性->版本->内部名称 就是此字段

`FileVersion`(版本) 在附加了`.rc`编译时，编译器会将`.rc`中的版本链接进去，在`Windows`中，右击文件名->属性->版本->文件版本 就是此字段

`nt`内核：即`ntoskrnl.exe/ ntkrnlmp.exe/ ntkrnlpa.exe/ ntkrpamp.exe`

`win32k`内核：即`win32k.sys`

我们在做`windows`内核相关工作的时候，经常需要确定当前是哪个内核。

就`XP SP3`来说，`nt`内核就有以下几种历史版本

`5.1.2600.5512 (xpsp.080413-2111)`

`5.1.2600.5657 (xpsp_sp3_gdr.080814-1236)`

`5.1.2600.5755 (xpsp_sp3_gdr.090206-1234)`

`5.1.2600.5781 (xpsp_sp3_gdr.090321-1317)`

除了`gdr`版本外，还有`qfe`版本。

`win32k`内核

`5.1.2600.5796 (xpsp_sp3_gdr.090417-1242)`

`win32k.sys`不管系统是`PAE`还是非`PAE`，单`CPU`还是多`CPU`，都是`win32k.sys`，这点不会给我们造成困扰。但是`ntkrnl`就不一样了。对于上面说的`nt`内核的每`1`个历史版本，它都有`4`种版本，分别是

`PAE  多 CPU`

`ntoskrnl.exe  N  N`

`ntkrnlmp.exe  N  Y`

`ntkrnlpa.exe  Y  N`

`ntkrpamp.exe  Y  Y`

(假如你在`windows\`system32`下没找到这`4`个文件的话，你可以到`C:\`WINDOWS\`Driver Cache\`i386`里去找`sp2.cab`或`sp3.cab`，然后

`expand sp2.cab -F:ntoskrnl.exe c:\windows\`system32`)

当系统从单`CPU`升级为多`CPU`时，`windows`会“提示找到新硬件”，用多核的`nt`内核文件覆盖当前单核`nt`内核文件，并重新启动，启动后就用上多`CPU`的`nt`内核，但当从多`CPU`降为单`CPU`时，仍然在使用多`CPU`的`nt`内核，所以，我们不能根据系统中当前有几个`CPU`来判断当前是用哪个内核文件。但是可以用当前是否为`PAE`模式来判断当前内核文件。

如果你的目的只是想`Load`硬盘上的内核文件到内存中，进行`PE`解析(例如找到原始`SSDT/ShadowSSDT/IDT`及其他`ring0 inline hook check)`，那么你可以看方式`1`，如果你要精确识别,请看方式`2`

### 方式 1

使用`PsLoadedModuleList`或`NtQuerySystemInformation`或`ZWQuerySystemInformation`，找到`nt`内核和`win32k`内核，根据`PLDR_DATA_TABLE_ENTRY->FullDllName`这个`FileName`，载入硬盘上相应文件，展开，修正`IAT`，修正`Fixup`(载入/展开等就不列在这里了)，`ARK`工具就可以进行`hook`识别了。

(感谢`qihoocom 轩辕小聪`的提醒，我才发现，原来`PLDR_DATA_TABLE_ENTRY->FullDllName`是准确的硬盘`PE Image`文件名字)

### 方式 2

如果你想借助`.pdb`来找出内核未导出函数的名字、地址等，那你就一定要精确地区分出系统中是哪个内核(`InternalName`)和具体版本(`FileVersion`)。

而`PsLoadedModuleList`或`NtQuerySystemInformation`或`ZWQuerySystemInformation`中得到的`nt`内核`FileName`，与该文件所对应的`.pdb`，并不存在直接关联(因为`Copy`的原因，`FileName!= InternalName`)

那么，怎么办呢？如何用程序实现像`Windows`看软件版本一样的功能呢？

经实践，发现可以对`ntkernel`的`BaseAddr,`用`PE`解析的方法，找到`Resource->Version,`取`InternalName`和`FileVersion`，见下表

`CompanyName  Microsoft Corporation`

`InternalName  ntkrnlpa.exe`

`FileVersion  5.1.2600.5755 (xpsp_sp3_gdr.090206-1234)`

`ProductVersion  5.1.2600.5755`

对于同样的`5.1.2600 build 5755`的`windowsXP`而言，存在`2`个完全不同的版本：`gdr(General Distribution Releases)`和`qfe(Quick Fix Engineering),`所以`ProductVersion`不能作为内核的区别。

`InternalName`和`FileVersion`，这`2`个字符串能唯一地确定内核`Image`版本。

代码范例(以下代码在`VC++/DDK`都可编译)：

```
DWORD VA2RAW(DWORD va,PVA2RAW_REC pva2raw_rec)
{
  int i;
  for (i=0;pva2raw_rec[i].VA!=0;i++)
    if ((va>=pva2raw_rec[i].VA)&&(va<=pva2raw_rec[i].VAEnd))
      return pva2raw_rec[i].RAW+va-pva2raw_rec[i].VA;
  return 0;
}

int PE_Head_Pointer(char *pbuf,
    PIMAGE_DOS_HEADER  *pIDH,
    PIMAGE_NT_HEADERS  *pINH,
    PIMAGE_FILE_HEADER  *pIFH,
    PIMAGE_OPTIONAL_HEADER  *pIOH,
    PIMAGE_SECTION_HEADER  *pISH)
{
  int i;
  *pIDH=(PIMAGE_DOS_HEADER)pbuf;
  if  ((*pIDH)->e_magic!=IMAGE_DOS_SIGNATURE)
    return -1;
  *pINH=(PIMAGE_NT_HEADERS)((DWORD)pbuf+(*pIDH)->e_lfanew);
  if ((*pINH)->Signature!=IMAGE_NT_SIGNATURE)
    return -1;
  *pIFH=&((*pINH)->FileHeader);
  *pIOH=&((*pINH)->OptionalHeader);
  if ((*pIOH)->Magic!=IMAGE_NT_OPTIONAL_HDR32_MAGIC)
    return -1;
#ifdef COMPILE_DEBUG1
    DbgPrint("IMAGE_DIRECTORY_ENTRIES\n");
    for (i=0;i<IMAGE_NUMBEROF_DIRECTORY_ENTRIES;i++)
    {
      switch(i)
      {
      case IMAGE_DIRECTORY_ENTRY_EXPORT:
          DbgPrint("0 Export Directory");
        break;
      case IMAGE_DIRECTORY_ENTRY_IMPORT:
          DbgPrint("1 Import Directory");
        break;
      case IMAGE_DIRECTORY_ENTRY_RESOURCE:
          DbgPrint("2 Resource Directory");
        break;
      case IMAGE_DIRECTORY_ENTRY_EXCEPTION:
          DbgPrint("3 Exception Directory");
        break;
      case IMAGE_DIRECTORY_ENTRY_SECURITY:
          DbgPrint("4 Security Directory");
        break;
      case IMAGE_DIRECTORY_ENTRY_BASERELOC:
          DbgPrint("5 Base Relocation Table");
        break;
      case IMAGE_DIRECTORY_ENTRY_DEBUG:
          DbgPrint("6 Debug Directory");
        break;
      case IMAGE_DIRECTORY_ENTRY_ARCHITECTURE:
          DbgPrint("7 Architecture Specific Data");
        break;
      case IMAGE_DIRECTORY_ENTRY_GLOBALPTR:
          DbgPrint("8 RVA of GP");
        break;
      case IMAGE_DIRECTORY_ENTRY_TLS:
          DbgPrint("9 TLS Directory");
        break;
      case IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG:
          DbgPrint("10 Load Configuration Directory");
        break;
      case IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT:
          DbgPrint("11 Bound Import Directory in headers");
        break;
      case IMAGE_DIRECTORY_ENTRY_IAT:
          DbgPrint("12 Import Address Table");
        break;
      case IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT:
          DbgPrint("13 Delay Load Import Descriptors");
        break;
      case IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR:
          DbgPrint("14 COM Runtime descriptor");
        break;
      default:
          DbgPrint("%d",i);
      }
      DbgPrint("  %0.8X %0.8X\n",
        (*pIOH)->DataDirectory[i].VirtualAddress,
        (*pIOH)->DataDirectory[i].Size
        );   
    }
    DbgPrint("ImageBase=0x%0.8X\tBaseofCode=0x%0.8X\tIDAStart(CodeStart)=0x%0.8X\n",
      (*pIOH)->ImageBase,(*pIOH)->BaseOfCode,(*pIOH)->ImageBase+(*pIOH)->BaseOfCode);
#endif
  *pISH=(PIMAGE_SECTION_HEADER)((*pIOH)+1);
  return 1;
}

int PE_Dump_Resource_Dir(char *pbuf,BYTE bRAWFile,
        DWORD  dwFirstIRD_Offset,
        PIMAGE_RESOURCE_DIRECTORY pIRD,
        BYTE  bLevel,
        DWORD  dwResourceType,
        char *szCompanyName,
        int iCompanyNameLen,
        char *szFileVersion,
        int iFileVersionLen,
        PVA2RAW_REC  pva2raw_rec)
{
  int i,j,iLen=0;
  DWORD dwAddr;
  char Buf[1024],
    szTemp1[256]={0};
  PIMAGE_RESOURCE_DIRECTORY_ENTRY  pIRDE;
  PIMAGE_RESOURCE_DATA_ENTRY    pIRDataE;
  PIMAGE_RESOURCE_DIR_STRING_U  pRDSU;
  if (bLevel>=4)  //4 层及以上结构，就不再挖了
    return 1;
  pIRDE=(PIMAGE_RESOURCE_DIRECTORY_ENTRY)(pIRD +1);
  for (i=0;i <pIRD->NumberOfIdEntries+pIRD->NumberOfNamedEntries;i++,pIRDE++)
  {
    if (pIRDE->DataIsDirectory)
    {
      #if (!defined _WINDOWS) && (!defined _CONSOLE)
        DbgPrint("DataIsDirectory pbuf+%X=%X\n",
          pIRDE->OffsetToDirectory+(DWORD)pIRD-(DWORD)pbuf,
          pIRDE->OffsetToDirectory+(DWORD)pIRD);
      #endif
      PE_Dump_Resource_Dir(pbuf,bRAWFile,dwFirstIRD_Offset,
        (PIMAGE_RESOURCE_DIRECTORY)(pIRDE->OffsetToDirectory+(DWORD)pbuf+dwFirstIRD_Offset),bLevel+1,
        dwResourceType,szCompanyName,iCompanyNameLen,szFileVersion,iFileVersionLen,
        pva2raw_rec);
    }else
    {
      pIRDataE = (PIMAGE_RESOURCE_DATA_ENTRY)
            ((DWORD)pbuf+dwFirstIRD_Offset + pIRDE->OffsetToData);
      #if (!defined _WINDOWS) && (!defined _CONSOLE)
        DbgPrint("DataRVA: OffsetToData=%05X  Size=%X  CodePage=%X\n",
            pIRDataE->OffsetToData,
            pIRDataE->Size,  pIRDataE->CodePage);
      #endif
      if (dwResourceType==RESOURCE_ID_VERSION)
      {
        if (bRAWFile)
          dwAddr=VA2RAW(pIRDataE->OffsetToData,pva2raw_rec);
        else
          dwAddr=pIRDataE->OffsetToData;
```
