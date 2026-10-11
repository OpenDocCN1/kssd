
```c
    pInSrcAddr += vLen;  

    vNewLen -= vLen;  

    vLen = 4096;  

    if ( vNewLen < 0x1000 )  

      vLen = vNewLen;  

    --nCircleCount;  

  }  

  return ulLen - vNewLen;  

}  

typedef struct _SYSTEM_BASIC_INFORMATION  

{  

  ULONG Unknown;  // 00000000  

  ULONG MaximumIncrement;//一个时钟的计量单位  0002625a  

  ULONG PhysicalPageSize; //一个内存页的大小  0x1000   

  ULONG NumberOfPhysicalPages; //系统管理着多少个页  0001ff7c   

  ULONG LowestPhysicalPage;//低端内存页   00000001  

  ULONG HighestPhysicalPage;//高端内存页   0001ffff  

  ULONG AllocationGranularity;  // 00010000  

  ULONG LowestUserAddress;//低端用户地址  00010000  

  ULONG HighestUserAddress;//高端用户地址  7ffeffff  

  ULONG ActiveProcessors;//激活的处理器  00000001  

  UCHAR NumberProcessors; //有多少个处理器  00000001  

}SYSTEM_BASIC_INFORMATION;  

//全局变量，用来记录物理内存信息  

PHYSICAL_ADDRESS g_PhysicalPage = { 0 };  

PHYSICAL_ADDRESS g_PhysicalHighPage = { 0 };  

PHYSICAL_ADDRESS g_PhysicalLowPage = { 0 };  

BOOL GetPhysicalPage()  

{  

  ULONG uLen;  

  NTSTATUS status = STATUS_SUCCESS;  

  SYSTEM_BASIC_INFORMATION struBasicInfo;  

  status = NtQuerySystemInformation( SystemBasicInformation, &struBasicInfo, sizeof(SYSTEM_BASIC_INFORMATION), &uLen );  

  if( !NT_SUCCESS( status ) )  

  {  

    return FALSE;  

  }  

  _asm  

  {  

    mov eax, struBasicInfo.PhysicalPageSize; //一个内存页的大小  

    mul struBasicInfo.NumberOfPhysicalPages; //系统管理着多少个页  

    mov g_PhysicalPage.HighPart, edx;  

    mov g_PhysicalPage.LowPart, eax;  

    mov eax, struBasicInfo.PhysicalPageSize; //一个内存页的大小  

    mul struBasicInfo.HighestPhysicalPage; //高端内存页  

    mov g_PhysicalHighPage.HighPart, edx;  

    mov g_PhysicalHighPage.LowPart, eax;  

    mov eax, struBasicInfo.PhysicalPageSize; //一个内存页的大小  

    mul struBasicInfo.LowestPhysicalPage; //低端内存页  

    mov g_PhysicalLowPage.HighPart, edx;  

    mov g_PhysicalLowPage.LowPart, eax;  

  }  

  return TRUE;  

}
```

第三种：打开 `nt` 模块文件（在 `GetModuleBaseAndSize()` 里可得到），在文件里扫描函数特侦码，得到文件偏移 `fileoffset`，  
然后通过 `rva` 和 `fileoffset` 的关系就可以计算出来。这部分代码我还未写完，需要的话自己写吧。

![](img/5a486957018cc8d631c876cf55447279.jpg)

代码如下：

```c
ULONG GetRvaByFileoffset( IN ULONG ulFileBase, IN ULONG ulFileSize, IN ULONG ulFileOffset )  

{  

  const ULONG ulSizeOfNtheaders = 0x0f0;  

  const ULONG ulSizeOfSectionheader = 0x28;  

  ULONG ulRet = 0, ulTemp, ulSectionTable;  

  ULONG ulRawStart, ulRawSize, ulVirtualStart;  

  ULONG ulRawStartValue, ulRawSizeValue, ulVirtualStartValue;  

  WORD i, wNumOfSections;  

  BOOL bFind = FALSE;  

  do  

  {  

    if( 0 == ulFileBase || 0 == ulFileSize || 0 == ulFileOffset )     
  
      break;  

    if( 0x5A4D != *( (PWORD)ulFileBase ) ) // value of e_magic  

      break;  

    ulTemp = *( (PDWORD)( ulFileBase + 0x3c ) ); // value of e_lfanew  

    ulTemp += ulFileBase;  // nt headers 的起始地址  

    if( 0x00004550 != *( (PDWORD)ulTemp ) ) // value of Signature  

      break;  

    wNumOfSections = *( (PWORD)( ulTemp + 6 ) ); // NumberOfSections  

    if( 0 == wNumOfSections )  

      break;  

    ulTemp += ulSizeOfNtheaders;  

    ulTemp += 8;  // Section Headers 的起始地址  

    for( i = 0; i < wNumOfSections; i++ )  

    {  

      ulSectionTable = ulTemp + ulSizeOfSectionheader * i;  

      ulRawStart = ulSectionTable + 0x14;  

      ulRawStartValue = *( (PDWORD)ulRawStart );  

      if( ulFileOffset >= ulRawStartValue )  

      {  

        ulRawSize = ulSectionTable + 0x10;  

        ulRawSizeValue = *( (PDWORD)ulRawSize );  

        if( ulFileOffset <= ( ulRawStartValue + ulRawSizeValue ) )  

        {  

          bFind = TRUE;  

          break;  

        }  

      }       
  
    }  

    if( bFind )  

    {  

      ulVirtualStart = ulSectionTable + 0x0c;  

      ulVirtualStartValue = *( (PDWORD)ulVirtualStart );  

      ulRet = ulFileOffset + ulVirtualStartValue - ulRawStartValue; // RVA = 文件偏移 + voffset - roffset  

    }  

  } while ( 0 );  

  return ulRet;  

}  

// 返回值是 RVA  

ULONG GetUnDocApiAddr( IN ULONG ulDllBase, IN ULONG ulSize, IN PULONG pulArrayFeatures, IN ULONG ulArrayNum )  

{  

  ULONG i, j, k, ulTotalFeaturesSize, ulEnd, ulFind, ulValue;  

  if( 0 == ulDllBase || 0 == ulSize || NULL == pulArrayFeatures || 0 == ulArrayNum )     
  
    return 0;  

  ulTotalFeaturesSize = ulArrayNum * sizeof( ULONG );  

  ulEnd = (ULONG)ulDllBase + ulSize;  

  for( i = (ULONG)ulDllBase; i <= ulEnd; i++ )  

  {  

    if( ( i + ulTotalFeaturesSize ) > ulEnd )  

      break;  

    ulFind = 0;  

    for( j = i, k = 0; k < ulArrayNum; j += sizeof( ULONG ), k++ )  

    {  

      if( *( (PULONG)j ) !=  pulArrayFeatures[ k ] )  

        break;  

      ulFind++;  

    }  

    if( ulFind == ulArrayNum )  

    {  

      return GetRvaByFileoffset( ulDllBase, ulSize, i - ulDllBase );  

    }  

  }  

  return 0;  

}
```

### 上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) | `360_hookport.zip` |
| :--- | :--- |
| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) | `SysReveal_driver.zip` |

```c
PsGetVersion( &ulMajorVersion, &ulMinorVersion, &ulBuildNumber, 0 );  

  if( ulMajorVersion == 5 && ulMinorVersion == 1 ) // xp 32 位  

  {  

    bRet = GetModuleBaseAndSize( g_szNtKrnlName[ 0 ], &nModuleBase, &nModuleSize );  

    if( !bRet )  

    {  

      bRet = GetModuleBaseAndSize( g_szNtKrnlName[ 1 ], &nModuleBase, &nModuleSize );  

    }  

    if( bRet )  

    {  

      if( CopyModule( nModuleBase, nModuleSize, &nCopyModuleBase, &nCopyModuleSize ) )  

      {  

        ulArrayFeatures[0] = 0x0124a164;  

        ulArrayFeatures[1] = 0x85890000;  

        ulArrayFeatures[2] = 0x0ffffff7c;  

        ulArrayFeatures[3] = 0x0140888a;  

        ulArrayFeatures[4] = 0x4d880000;  

        ulArrayFeatures[5] = 0x44408bdf;  

        ulArrayFeatures[6] = 0x33a84589;  

        ulArrayFeatures[7] = 0x0e345c6f6;  

        ulArrayFeatures[8] = 0x0b8758900;  

        ulArrayFeatures[9] = 0x0f7bc7589;  

        ulArrayFeatures[10] = 0x0fff01845;  

        //ulArrayFeatures[11] = 0x4075ffff;  

        ulOffset = 0x805d1867 - 0x805d1858;  

        g_ulPspCreateProcess = ( ULONG )GetUnDocApiAddr( nCopyModuleBase, nCopyModuleSize, ulArrayFeatures, 11, ulOffset );  

        if( 0 != g_ulPspCreateProcess )  

        {  

          g_ulPspCreateProcess = nModuleBase + g_ulPspCreateProcess - nCopyModuleBase;  

          if( 0 != g_ulPspCreateProcess && NT_SUCCESS( MyHookPspCreateProcess() ) )  

          {  

            SIOCTL_KDPRINT( ( "MyHookPspCreateProcess()成功！PspCreateProcess:%#x...", g_ulPspCreateProcess ) );   

          }  

        }  

      }  

    }  

  }  

  else if( ulMajorVersion == 6 && ulMinorVersion == 1 ) // win7 32 位  

  {       
  
    bRet = GetModuleBaseAndSize( g_szNtKrnlName[ 0 ], &nModuleBase, &nModuleSize );  

    if( !bRet )  

    {  

      bRet = GetModuleBaseAndSize( g_szNtKrnlName[ 1 ], &nModuleBase, &nModuleSize );  

    }  

    if( bRet )  

    {  

      if( CopyModule( nModuleBase, nModuleSize, &nCopyModuleBase, &nCopyModuleSize ) )  

      {  

        ulArrayFeatures[0] = 0x8908458b;  

        ulArrayFeatures[1] = 0x0fff9b085;  

        ulArrayFeatures[2] = 0x0c458bff;  

        ulArrayFeatures[3] = 0x0f9ac8589;  

        ulArrayFeatures[4] = 0x5d8bffff;  

        ulArrayFeatures[5] = 0x0a09d8918;  

        ulArrayFeatures[6] = 0x8bfffff9;  

        ulArrayFeatures[7] = 0x85891c45;  

        ulArrayFeatures[8] = 0x0fffff98c;  

        ulArrayFeatures[9] = 0x8928458b;  

        ulArrayFeatures[10] = 0x0fff9c885;  

        ulArrayFeatures[11] = 0x2c458bff;  

        ulOffset = 0x83e86065 - 0x83e86056;  

        g_ulNtCreateUserProcess = ( ULONG )GetUnDocApiAddr( nCopyModuleBase, nCopyModuleSize, ulArrayFeatures, 12, ulOffset );  

        if( 0 != g_ulNtCreateUserProcess )  

        {  
```
