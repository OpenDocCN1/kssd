
```assembly
.model flat,stdcall
option casemap:none

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

include Demo.inc

;--------------------------------------------------------------------
.data

RealNtCreateMutant NTCREATEMUTANT ?
DescriptorAddrToMutant dd ?
MapServiceDescriptorTable dd ?
MewMDL dd ?
SerTablBase dd ?
TablSize dd ?

.code
include SafeHook\_Func.asm

HookNtCreateMutant proc MutantHandle:dword,DesiredAccess:dword,ObjectAttributes:dword,InitialOwner:dword
        pushad
        invoke RealNtCreateMutant,MutantHandle,DesiredAccess,ObjectAttributes,InitialOwner
        popad
        xor eax,eax
        ret
HookNtCreateMutant endp

DriverUnload proc pDriverObject:PDRIVER\_OBJECT
        pushad
        ;----------------------------------------------------------------------
        invoke UnHookSSDT,RealNtCreateMutant,DescriptorAddrToMutant
        invoke UnLockMdl,MewMDL,MapServiceDescriptorTable
        ;----------------------------------------------------------------------
        popad
        ret
DriverUnload endp

DriverEntry proc pDriverObject:PDRIVER\_OBJECT,pusRegistryPath:PUNICODE\_STRING;入口函数
        local Status:dword
        pushad
        mov Status,STATUS\_SUCCESS
        ;----------------------------------------------------------------------
        invoke GetSerTablInf,offset SerTablBase,offset TablSize
        invoke LockMdl,SerTablBase,TablSize,offset MewMDL,offset MapServiceDescriptorTable
        .if eax==STATUS\_UNSUCCESSFUL
                mov Status,eax
                jmp Exit
        .endif
        invoke HookSSDT,offset RealNtCreateMutant,offset DescriptorAddrToMutant,offset HookNtCreateMutant,NULL,2bH
        ;----------------------------------------------------------------------
        mov esi,pDriverObject
        assume esi:ptr DRIVER\_OBJECT
        mov [esi].DriverUnload,offset DriverUnload;设置卸载例程
        assume esi:nothing
        Exit:
        popad
        mov eax,Status
        ret
DriverEntry endp

end DriverEntry
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
