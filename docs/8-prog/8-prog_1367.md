
# 通过重映射+CRC32 来过杀毒软件虚拟机的动态行为检查

> 原文：[`www.pediy.com/kssd/pediy11/125011.html`](https://www.pediy.com/kssd/pediy11/125011.html)

好几个朋友问我如何过虚拟机，虚拟机跑起来除了脱壳的作用。还有就是针对动态获取的 `API` 做调用做判别然后归纳行为。以下代码就是解决这个问题的。

代码:



```c
#include <string.h>
#include "crc.h"

#include "xGetProcAddress.h"

#define __GetDosHeader__(x)        ((PIMAGE_DOS_HEADER)(x))
#define __GetNtHeader__(x)        ((PIMAGE_NT_HEADERS)((DWORD)__GetDosHeader__(x)->e_lfanew + (DWORD)(x)))
#define __RvaToVa__(base,offset) ((PVOID)((ULONG)(base) + (ULONG)(offset)))
#define __VaToRva__(base,offset) ((PVOID)((ULONG)(offset) - (ULONG)(base)))

typedef HMODULE (WINAPI *FPLoadLibraryA)(LPCSTR pLibName);
typedef ULONG (__stdcall *FPHashFunc)(PUCHAR pTarget, ULONG iTargetSize, PUCHAR pHashValue);
typedef LPVOID (WINAPI *FPVirtualAlloc)(LPVOID lpAddress, SIZE_T dwSize, DWORD flAllocationType, DWORD flProtect);
typedef BOOL (WINAPI *FPDllMain)(HMODULE hinstDLL, DWORD fdwReason, LPVOID lpvReserved);

PIMAGE_DATA_DIRECTORY ExistDataDirectory(PUCHAR pMem, DWORD dwIndex) {
    PIMAGE_NT_HEADERS pNtHeader = __GetNtHeader__(pMem);
    return (PIMAGE_DATA_DIRECTORY)(&pNtHeader->OptionalHeader.DataDirectory[dwIndex]);
}

ULONG __stdcall MyHashFunc(PUCHAR pTarget, ULONG iTargetSize, PUCHAR pHashValue) {
    ULONG dwCrc32 = crc32(pTarget, iTargetSize);
    memcpy(pHashValue, &dwCrc32, sizeof(ULONG));
    return sizeof(ULONG);
}

PIMAGE_SECTION_HEADER GetFirstSectionByNtHeader(PIMAGE_NT_HEADERS pNtH) {
    return IMAGE_FIRST_SECTION(pNtH);
}

BOOL InThisSection(PIMAGE_SECTION_HEADER pSectH, DWORD ofOffset, BOOL bRva) {
    return (bRva ? (ofOffset >= (DWORD)(pSectH->VirtualAddress)) && (ofOffset < (DWORD)(pSectH->VirtualAddress + pSectH->Misc.VirtualSize)) :
        (ofOffset >= (DWORD)(pSectH->PointerToRawData)) && (ofOffset < (DWORD)(pSectH->PointerToRawData + pSectH->SizeOfRawData)));
}

PIMAGE_SECTION_HEADER Rva2Section(PUCHAR pMem, DWORD ofRva) {
    PIMAGE_NT_HEADERS pNtH = __GetNtHeader__(pMem);
    PIMAGE_SECTION_HEADER pSectH = GetFirstSectionByNtHeader(pNtH);
    WORD wNumOfSects = pNtH->FileHeader.NumberOfSections;
    while (wNumOfSects > 0) {
        if (InThisSection(pSectH, ofRva, TRUE))
            break;

        --wNumOfSects;
        ++pSectH;
    }

    return (0 == wNumOfSects ? NULL : pSectH);
}

PIMAGE_SECTION_HEADER Raw2Section(PUCHAR pMem, DWORD ofRaw) {
    PIMAGE_NT_HEADERS pNtH = __GetNtHeader__(pMem);
    PIMAGE_SECTION_HEADER pSectH = GetFirstSectionByNtHeader(pNtH);
    WORD wNumOfSects = pNtH->FileHeader.NumberOfSections;
    while (wNumOfSects > 0) {
        if (InThisSection(pSectH, ofRaw, FALSE))
            break;

        --wNumOfSects;
        pSectH++;
    }

    return (0 == wNumOfSects ? NULL : pSectH);
}

DWORD Rva2Raw(PUCHAR pMem, DWORD ofRva) {
    PIMAGE_SECTION_HEADER pSectH = Rva2Section(pMem, ofRva);
    return ((NULL == pSectH) ? NULL : (ofRva - pSectH->VirtualAddress + pSectH->PointerToRawData));
}

DWORD Raw2Rva(PUCHAR pMem, DWORD ofRaw) {
    PIMAGE_SECTION_HEADER pSectH = Raw2Section(pMem, ofRaw);
    return ((NULL == pSectH) ? NULL : (ofRaw - pSectH->PointerToRawData + pSectH->VirtualAddress));
}

// 函数声明
FARPROC xLdrGetExportByName(
    PUCHAR pBaseAddress, 
    PUCHAR pHashPoint, 
    ULONG iHashSize,
    FPHashFunc pHashFunc, FARPROC fpLoadLibraryA
    );
ULONG __stdcall xLdrFixupForwardHashFunc(PUCHAR pTarget, ULONG iTargetSize, PUCHAR pHashValue) {
    memcpy(pHashValue, pTarget, iTargetSize);
    return iTargetSize;
}

FARPROC xLdrFixupForward(PUCHAR pForwardName, FARPROC fpLoadLibraryA) {
    CHAR NameBuffer[128];
    PUCHAR pPoint;
    HMODULE hModule;
    PUCHAR pBaseAddress;
    FARPROC pFunction;
    FPLoadLibraryA pLoadLibraryA = (FPLoadLibraryA)fpLoadLibraryA;

    strcpy(NameBuffer, (char *)pForwardName);
    pPoint = (PUCHAR)strchr(NameBuffer, '.');
    if (pPoint) {
        ULONG iProcLen = 0;

        *pPoint = 0;
        hModule = pLoadLibraryA(NameBuffer);
        if (!hModule) return NULL;

        iProcLen = strlen((char *)pPoint + 1);
        pFunction = xLdrGetExportByName((PUCHAR)hModule, 
                                        (PUCHAR)(pPoint + 1), 
                                        iProcLen, 
                                        (FPHashFunc)xLdrFixupForwardHashFunc, 
                                        fpLoadLibraryA);
        return pFunction;
    }

    return NULL;
}

FARPROC xLdrGetExportByOrdinal(PUCHAR pBaseAddress, WORD wOrdinal, FARPROC fpLoadLibraryA) {
    PIMAGE_EXPORT_DIRECTORY pExportDir;
    ULONG iExportDirSize;
    ULONG **pExFunctionsPoint;
    FARPROC pFunction;
    PIMAGE_DATA_DIRECTORY pExportDataDirectory = ExistDataDirectory(pBaseAddress, IMAGE_DIRECTORY_ENTRY_EXPORT);
    pExportDir = (PIMAGE_EXPORT_DIRECTORY)(pBaseAddress + pExportDataDirectory->VirtualAddress);
    if (!pExportDir)
        return NULL;
    iExportDirSize = pExportDataDirectory->Size;

    pExFunctionsPoint = (ULONG **)__RvaToVa__(pBaseAddress, pExportDir->AddressOfFunctions);
    pFunction = (FARPROC)(0 != pExFunctionsPoint[wOrdinal - pExportDir->Base]
                ? __RvaToVa__(pBaseAddress, pExFunctionsPoint[wOrdinal - pExportDir->Base])
                : NULL);

    if (((DWORD)pFunction >= (DWORD)pExportDir) &&
        ((DWORD)pFunction < (DWORD)pExportDir + (DWORD)iExportDirSize))
        pFunction = xLdrFixupForward((BYTE *)pFunction, fpLoadLibraryA);

    return pFunction;
}

FARPROC xLdrGetExportByName(PUCHAR pBaseAddress, PUCHAR pHashPoint, ULONG iHashSize, \
                            FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
    WORD wOrdinal = 0;
    ULONG iDirCount = 0;
    ULONG *pAddrTable = NULL;
    ULONG addrAddr = 0;
    ULONG ofRVA = 0;
    ULONG iExpDataSize = 0;
    PIMAGE_EXPORT_DIRECTORY pEd = NULL;
    PIMAGE_NT_HEADERS pNt = NULL;
    PIMAGE_DATA_DIRECTORY pExportDataDirectory = NULL;
    if (pBaseAddress == NULL) return NULL;

    pNt = __GetNtHeader__(pBaseAddress);
    iDirCount = pNt->OptionalHeader.NumberOfRvaAndSizes;
    if (iDirCount < IMAGE_NUMBEROF_DIRECTORY_ENTRIES) return FALSE;
    pExportDataDirectory = ExistDataDirectory(pBaseAddress, IMAGE_DIRECTORY_ENTRY_EXPORT);
    if (!pExportDataDirectory)
        return NULL;
    iExpDataSize = pExportDataDirectory->Size;
    pEd = (PIMAGE_EXPORT_DIRECTORY)__RvaToVa__(pBaseAddress, pExportDataDirectory->VirtualAddress);     
    if (HIWORD((DWORD)pHashPoint)==0) {
        wOrdinal = (WORD)(LOWORD((DWORD)pHashPoint)) - pEd->Base;
    } else {
        ULONG i, iCount;
        ULONG *pdwNamePtr;
        WORD *pwOrdinalPtr;

        iCount = (ULONG)(pEd->NumberOfNames);
        pdwNamePtr = (ULONG *)__RvaToVa__(pBaseAddress, pEd->AddressOfNames);
        pwOrdinalPtr = (WORD *)__RvaToVa__(pBaseAddress, pEd->AddressOfNameOrdinals);

        for(i = 0; i < iCount; i++) {
            BYTE HashValue[1024];
            CHAR *svName = NULL;
            ULONG iHashValueSize = 0;
            ULONG iSvNameLen = 0;

            svName = (char *)__RvaToVa__(pBaseAddress, *pdwNamePtr);
            iSvNameLen = strlen(svName);
            iHashValueSize = pHashFunc((PUCHAR)svName, iSvNameLen, HashValue);
            if (strcmp("GetLastError", svName) == 0)
            {
                int i = 0;
            }
            if (iHashValueSize == iHashSize) {
                if (memcmp(HashValue, pHashPoint, iHashSize) == 0) {
                    wOrdinal = *pwOrdinalPtr;
                    break;
                }
            }
            pdwNamePtr++;
            pwOrdinalPtr++;
        }
        if (i == iCount) return NULL;
    }

    pAddrTable=(ULONG *)__RvaToVa__(pBaseAddress, pEd->AddressOfFunctions);
    ofRVA = pAddrTable[wOrdinal];
    addrAddr = (ULONG)__RvaToVa__(pBaseAddress, ofRVA);
    if (((ULONG)addrAddr >= (ULONG)pEd) &&
        ((ULONG)addrAddr < (ULONG)pEd + (ULONG)iExpDataSize))
        (FARPROC)addrAddr = xLdrFixupForward((PUCHAR)addrAddr, fpLoadLibraryA);

    return (FARPROC)addrAddr;
}

FARPROC xLdrGetProcedureAddress(PUCHAR pBaseAddress, PUCHAR pHashPoint, ULONG iHashSize, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
    FARPROC pProcedureAddress = NULL;
    ULONG dwOrdinal = (ULONG)pHashPoint;

    if (HIWORD((ULONG)pHashPoint)) {
        pProcedureAddress = xLdrGetExportByName(pBaseAddress, pHashPoint, iHashSize, pHashFunc, fpLoadLibraryA);
    } else {
        dwOrdinal &= 0x0000FFFF;
        pProcedureAddress = xLdrGetExportByOrdinal(pBaseAddress, (WORD)dwOrdinal, fpLoadLibraryA);
    }
    return pProcedureAddress;
}

FARPROC g_pLoadLibraryAInGetProcAddressByCrc32 = NULL;

VOID xInitGetProcAddress(FARPROC pLoadLibraryA) {
    g_pLoadLibraryAInGetProcAddressByCrc32 = pLoadLibraryA;
}

FARPROC xGetProcAddressByCrc32(HMODULE hModule, PUCHAR pCrc32Point) {
    FARPROC pProc = NULL;
    pProc = xLdrGetProcedureAddress((PUCHAR)hModule, 
                                    pCrc32Point, 
                                    sizeof(DWORD), 
                                    MyHashFunc, 
                                    (FARPROC)g_pLoadLibraryAInGetProcAddressByCrc32);
    return pProc;
}

// 重定位所需结构
#pragma pack(push,1)
// 修复入口
typedef struct _IMAGE_FIXUP_ENTRY {
    WORD offset:12;
    WORD type:4;
} IMAGE_FIXUP_ENTRY, *PIMAGE_FIXUP_ENTRY;
// 重定位块
typedef struct _IMAGE_FIXUP_BLOCK {
    DWORD dwPageRVA;
    DWORD dwBlockSize;
} IMAGE_FIXUP_BLOCK, *PIMAGE_FIXUP_BLOCK;
#pragma pack(pop)
BOOL BaseRelocation(PUCHAR pMem, DWORD addrOldImageBase, DWORD addrNewImageBase, BOOL bIsInFile) {
    PIMAGE_NT_HEADERS pNtHdr = NULL;
    DWORD delta = (DWORD)(addrNewImageBase - addrOldImageBase);
    DWORD *pFixAddRhi = NULL;
    BOOL bHaveFixAddRhi = FALSE;
    DWORD iRelocSize = 0;

    pNtHdr = __GetNtHeader__(pMem);
    iRelocSize = pNtHdr->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].Size;

    if ((delta) && (iRelocSize)) {
        PIMAGE_FIXUP_BLOCK pStartFB = NULL;
        PIMAGE_FIXUP_BLOCK pIBR = NULL;
        if (bIsInFile) {
            DWORD iRelocRaw = Rva2Raw(pMem, pNtHdr->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress);
            pIBR = (PIMAGE_FIXUP_BLOCK)(pMem + iRelocRaw);
        } else {
            pIBR = (PIMAGE_FIXUP_BLOCK)__RvaToVa__(addrNewImageBase, pNtHdr->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress);
        }
        pStartFB = pIBR;

        // 遍历每个重定位块
        while ((DWORD)(pIBR - pStartFB) < iRelocSize) {
            PIMAGE_FIXUP_ENTRY pFE;
            DWORD i, iCount = 0;
            if (pIBR->dwBlockSize > 0) {
                iCount=(pIBR->dwBlockSize - sizeof(IMAGE_FIXUP_BLOCK)) / sizeof(IMAGE_FIXUP_ENTRY);
                pFE = (PIMAGE_FIXUP_ENTRY)(((PUCHAR)pIBR) + sizeof(IMAGE_FIXUP_BLOCK));
            } else {
                //pIBR = (PIMAGE_FIXUP_BLOCK)(((memory)pIBR) + sizeof(IMAGE_FIXUP_BLOCK));         
                //continue;
                break;
            }

            // 修复每个入口
            for (i = 0; i < iCount; i++) {
                PUCHAR pFixAddr = NULL;
                if (bIsInFile) {//如果在文件中
                    DWORD ofRva = pIBR->dwPageRVA + pFE->offset;
                    DWORD ofRaw = Rva2Raw(pMem, ofRva);
                    pFixAddr = pMem + ofRaw;
                } else {//如果在内存中
                    pFixAddr = (PUCHAR)__RvaToVa__(addrNewImageBase, pIBR->dwPageRVA + pFE->offset);
                }

                switch (pFE->type)
                {
#if defined(_X86_)
                case IMAGE_REL_BASED_ABSOLUTE:
                    break;
                case IMAGE_REL_BASED_HIGH:
                    *((signed short *)pFixAddr) += (signed short)HIWORD(delta);
                    break;
                case IMAGE_REL_BASED_LOW:
                    *((signed short *)pFixAddr) += (signed short)LOWORD(delta);
                    break;
                case IMAGE_REL_BASED_HIGHLOW:
                    *((signed short *)pFixAddr) += (signed short)delta;
                    break;
                case IMAGE_REL_BASED_HIGHADJ: // This one's really fucked up.
                    {
                        DWORD dwAdjust;             
                        dwAdjust = ((*((WORD *)pFixAddr)) << 16) | (*(WORD *)(pFE + 1));
                        (signed long)dwAdjust += (signed long)delta;
                        dwAdjust += 0x00008000;
                        *((WORD *)pFixAddr) = HIWORD(dwAdjust);
                    }
                    pFE++;
                    break;
#endif
                default:
                    return FALSE;
                }/* end switch */
                pFE++;
            }/* end for */

            pIBR = (PIMAGE_FIXUP_BLOCK)((PUCHAR)pIBR + pIBR->dwBlockSize);
        }/* end while */
    }
    return TRUE;
}

// 重映射 DLL
PUCHAR RemapDllEx(PUCHAR pOrigMap, FPVirtualAlloc pVirtualAlloc) {
    PUCHAR pNewMap = NULL;
    DWORD iSizeOfImage = 0;
    FPDllMain pDllMain = NULL;
    PIMAGE_NT_HEADERS pOrigMapNtHdr = NULL;

    pOrigMapNtHdr = __GetNtHeader__(pOrigMap);
    iSizeOfImage = pOrigMapNtHdr->OptionalHeader.SizeOfImage;
    pNewMap = (PUCHAR)pVirtualAlloc(NULL, iSizeOfImage, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
    if (!pNewMap) return NULL;
    memset(pNewMap, 0, iSizeOfImage);
    memcpy(pNewMap, pOrigMap, iSizeOfImage);
    // 进行重定位
    BaseRelocation(pNewMap, (DWORD)pOrigMap, (DWORD)pNewMap, FALSE);
    // 获取到入口点地址,并运行
    //pDllMain = (FPDllMain)(pNewMap + (pOrigMapNtHdr->OptionalHeader.AddressOfEntryPoint));
    //pDllMain((HMODULE)pNewMap, DLL_PROCESS_ATTACH, NULL);
    return pNewMap;
}

PUCHAR RemapDll(PUCHAR pOrigMap) {
    return RemapDllEx(pOrigMap, VirtualAlloc);
}
```



在`LoadLibrary`后使用`Remap`重新映射出一个句柄，再使用`xGetProcAddress`使用预定义的`Hash`值获取API地址。这样虚拟机在跑的过程中就不能判别调用的地址到底是哪个函数，达到通过虚拟机启发式的检测。

代码上用到了`crc32`算法不贴了自己写吧。这是在`SOURCE`上的免杀，如果是`BIN`，就做壳。在处理引入表的时候，重新映射，或许可以加一些`stub`之类的。还有变形。这样保护性能也可以提高。。。

如果开启了`DEP`选项，在`SOUCE`时在`C++`编译便签修改为`DEP`不兼容，如果是`BIN`上把`BIN`的`DLL`属性的`DEP`兼容清位就可以了。如果目标机器开启`DEP 2`级。那么就神马都是浮云了。。。
![恐怖!](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")
