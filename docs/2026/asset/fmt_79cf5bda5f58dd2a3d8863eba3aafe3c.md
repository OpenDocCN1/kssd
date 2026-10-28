
```c
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

                if(stricmp(pKernelName,sysFileName)==0)

                {

                        kernelBase=(DWORD)pModules->smi[i].Base;

                        return kernelBase;

                }

        }

        

        return 0;

}

typedef struct _old_fsd

{

        DWORD id;

        DWORD address;

}old_fsd,*pold_fsd;

old_fsd oldFsd[20];

BYTE fatFalg[3]={0xc7,0x46,0x38};

BYTE ntfsFalg[3]={0xc7,0x46,0x7c};

//搜索fastfat.sys或ntfs.sys中的特征代码，得到原始的fsd，并保存在结构数组中oldFsd中

BOOL GetOldFsd(PUNICODE_STRING fileName,DWORD fileSysBase,BOOL bfalg)

{

        char *code;

        PVOID lpBase=NULL;

        NTSTATUS status;

        DWORD i;

    HANDLE  hSection, hFile;

    DWORD imageBase;

    SIZE_T size=0;

    IO_STATUS_BLOCK iosb;

    OBJECT_ATTRIBUTES oa = {sizeof oa, 0, fileName, OBJ_CASE_INSENSITIVE};

        BYTE id;

        DWORD dwid;

        DWORD address;

        int j;

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

     status=ZwMapViewOfSection(hSection, NtCurrentProcess(), &lpBase, 0, 1000, 0, &size, (SECTION_INHERIT)1, MEM_TOP_DOWN, PAGE_READWRITE); 

    if(!NT_SUCCESS(status))

        {

                DbgPrint("ZwMapViewOfSection failed\n");

                return FALSE;

        }

    ZwClose(hFile);

        imageBase=GetImageBase(lpBase);

        if(imageBase==0)

        {

                DbgPrint("get ImageBase failed\n");

                return FALSE;

        }

        code=(char *)lpBase;

        if(bfalg)

        {

                for(i=0;i<143360;i++)

                {

                        if(memcmp(code,&fatFalg,3)==0)

                        {

                                for(j=0;j<20;j++)

                                {        

                                        if(*(WORD*)(code)==0x1c6a)

                                                return TRUE;

                                        if(*(WORD*)(code)==0x86c7)

                                        {

                                                code+=2;

                                                dwid=*(DWORD*)code;

                                                code+=4;

                                                address=*(DWORD*)code;

                                                        oldFsd[j].address=address-imageBase+fileSysBase;

                                        oldFsd[j].id=dwid;

                                        oldFsd[j].id=(oldFsd[j].id>>2)-0xE;

                                                code+=4;

                                                continue;

                                        }

                                        code+=2;;

                                        id=*(BYTE*)code;

                                        code++;

                                        address=*(DWORD*)code;

                                        oldFsd[j].address=address-imageBase+fileSysBase;

                                        oldFsd[j].id=(DWORD)id;

                                        oldFsd[j].id=(oldFsd[j].id>>2)-0xE;

                                        code+=4;        

                                }        

                        }

                        code++;

                }

        }

        else

        {

                for(i=0;i<574464;i++)

                {

                        if(memcmp(code,&ntfsFalg,3)==0)

                        {

                                for(j=0;j<20;j++)

                                {

                                        if(*(WORD*)(code)==0xb9b8)

                                                return TRUE;

                                        if(*(WORD*)(code)==0x86c7)

                                        {

                                                code+=2;

                                                dwid=*(DWORD*)code;

                                                code+=4;

                                                address=*(DWORD*)code;

                                                oldFsd[j].address=address-imageBase+fileSysBase;

                                                oldFsd[j].id=dwid;

                                                oldFsd[j].id=(oldFsd[j].id>>2)-0xE;

                                                code+=4;

                                                continue;

                                        }

                                        code+=2;;

                                        id=*(BYTE*)code;

                                        code++;

                                        address=*(DWORD*)code;

                                        oldFsd[j].address=address-imageBase+fileSysBase;

                                        oldFsd[j].id=(DWORD)id;

                                        oldFsd[j].id=(oldFsd[j].id>>2)-0xE;

                                        code+=4;        

                                }

                        }

                        code++;

                }

        }

        return FALSE;

}

//写入fsd

BOOL SetFsd(PUNICODE_STRING objectName)

{

        ULONG i;

        NTSTATUS ntStatus;

        PDRIVER_OBJECT      DriverObject;

        ntStatus = ObReferenceObjectByName(objectName, 

                OBJ_CASE_INSENSITIVE,

                NULL,

                0,

                *IoDriverObjectType,

                KernelMode,

                NULL,

                &DriverObject);

        if (!NT_SUCCESS(ntStatus))

        {

                DbgPrint("ObReferenceObjectByName failed\n");

                return FALSE;

        }

        for(i=0;i<20&&oldFsd[i].address;i++)

        { 

                

                if((ULONG)DriverObject->MajorFunction[oldFsd[i].id]!=oldFsd[i].address)

                {

                        _asm

                        {

                                CLI  ;                 

                                MOV    EAX, CR0  ;    

                                AND EAX, NOT 10000H ;

                                MOV    CR0, EAX;        

                        }

                        (ULONG)DriverObject->MajorFunction[oldFsd[i].id]=oldFsd[i].address;

                        _asm 

                        {

                                MOV    EAX, CR0;        

                                OR    EAX, 10000H;            

                                MOV    CR0, EAX ;              

                                STI;                    

                        }

                        DbgPrint("IRP_ID:%x\n",oldFsd[i].id);

                }

        }

        return TRUE;

}

//判断系统中有哪些格式的分区，还原fsd

void ResetFsd()

{

        DWORD kernelBase;

        UNICODE_STRING fileSysName;

        UNICODE_STRING objectName;

        WCHAR fatObjectString[]=L"\\FileSystem\\FastFat";

        WCHAR ntfsObjectString[]=L"\\FileSystem\\Ntfs";

        kernelBase=FoundSystemModule(FALSE,"fastfat.sys");

        if(kernelBase!=0)

        {

                RtlInitUnicodeString(&fileSysName, L"\\Device\\HarddiskVolume1\\Windows\\System32\\drivers\\fastfat.sys");

                GetOldFsd(&fileSysName,kernelBase,TRUE);

                RtlInitUnicodeString(&objectName,fatObjectString);

                if(!SetFsd(&objectName))

                        return;

                memset(oldFsd,0,sizeof(oldFsd));

        }

        kernelBase=FoundSystemModule(FALSE,"ntfs.sys");

        if(kernelBase!=0)

        {

                RtlInitUnicodeString(&fileSysName, L"\\Device\\HarddiskVolume1\\Windows\\System32\\drivers\\ntfs.sys");

                GetOldFsd(&fileSysName,kernelBase,FALSE);

                RtlInitUnicodeString(&objectName,ntfsObjectString);

                if(!SetFsd(&objectName))

                        return;

                memset(oldFsd,0,sizeof(oldFsd));

        }

}
```
