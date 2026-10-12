
```
FD\_DebugObject\_NtQueryObject:
FD\_DebugObject\_NtQueryObject\_Arg\_Win32Api        equ 04h
    mov eax, dword ptr [esp+FD\_DebugObject\_NtQueryObject\_Arg\_Win32Api]
        
    push ebx
    push ecx
    push edx
    push edi
    push esi
    
    mov ebx, eax
    assume ebx : ptr WIN32APIBASE
    push edx    ; alloc the stack
    
    push esp    ; ReturnLength
    push 0
    push 0
    push ObjectAllTypeInformation
    push 0
    call dword ptr [ebx].xNtQueryObject
    pop ecx
    ;; make a tmp stack
    push ebp
    mov ebp, esp
    sub esp, ecx
    mov esi, esp
    ;; ObjectInformationLength
    push 0
    push ecx
    push esi
    push ObjectAllTypeInformation
    push 0
    call dword ptr [ebx].xNtQueryObject
    cld
    ;; NumberOfObjectsTypes
    lodsd
    xchg ecx, eax ; ecx = NumberOfObjectsTypes
    FD\_DebugObject\_NtQueryObject\_Loop:     
    ;; load string lengths
    lodsd
    movzx edx, ax
    ;; pointer to TypeName
    lodsd
    xchg esi, eax
    ;; sizeof(L"DebugObject")
    ;; avoids superstrings
    ;; like "DebugObjective"
    cmp edx, 16h
    jnz FD\_DebugObject\_NtQueryObject\_Tmp2
    xchg ecx, edx
    FD\_DebugObject\_NtQueryObject\_Tmp1:
    call FD\_DebugObject\_NtQueryObject\_UnicodeStr1
        dw 'D','e','b','u','g'
        dw 'O','b','j','e','c','t'
    FD\_DebugObject\_NtQueryObject\_UnicodeStr1:
    pop edi
    repe cmpsb
    xchg ecx, edx
    jnz FD\_DebugObject\_NtQueryObject\_Tmp2
    ;; TotalNumberOfObjects
    cmp dword ptr [eax], edx
    jnz Found\_FD\_DebugObject\_NtQueryObject
    ;; point to trailing  null
    FD\_DebugObject\_NtQueryObject\_Tmp2:
    add esi, edx
    ;; round down to dword
    and esi, -4
    ;; skip trailing null
    ;; and any alignment bytes
    lodsd
    loop FD\_DebugObject\_NtQueryObject\_Loop
    xor eax, eax
Exit\_FD\_DebugObject\_NtQueryObject:
    ;; clear the tmp stack
    mov esp, ebp
    pop ebp
    
    pop esi
    pop edi
    pop edx
    pop ecx
    pop ebx
    assume ebx : nothing     
    retn 04h
Found\_FD\_DebugObject\_NtQueryObject:     
    mov eax, 1
    jmp Exit\_FD\_DebugObject\_NtQueryObject
End\_FD\_DebugObject\_NtQueryObject:
```

此类方法，利用`FindWindow`函数通过寻找是否存在一些常见调试软件的Title。这里收集了一些  

如果有其他的Title，请朋友们告知。。。  

代码：

```
FD\_Find\_Debugger\_Window:
FD\_Find\_Debugger\_Window\_Arg\_WinApi32   equ 08h
    push ebp
    mov ebp, esp
    
    push ebx
    
    mov ebx, dword ptr [ebp+FD\_Find\_Debugger\_Window\_Arg\_WinApi32]
    assume ebx : ptr WIN32APIBASE
    
    push NULL
    call FD\_Find\_Debugger\_Window\_Str1
        db "1212121",0
    FD\_Find\_Debugger\_Window\_Str1:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window
    
    push NULL
    call FD\_Find\_Debugger\_Window\_Str2
        db "icu\_dbg",0
    FD\_Find\_Debugger\_Window\_Str2:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window     
    
    push NULL
    call FD\_Find\_Debugger\_Window\_Str3
        db "pe--diy",0
    FD\_Find\_Debugger\_Window\_Str3:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window      
    
    push NULL
    call FD\_Find\_Debugger\_Window\_Str5
        db "ollydbg",0
    FD\_Find\_Debugger\_Window\_Str5:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window 

    push NULL
    call FD\_Find\_Debugger\_Window\_Str6
        db "odbydyk",0
    FD\_Find\_Debugger\_Window\_Str6:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_Str7
        db "WinDbgFrameClass",0
    FD\_Find\_Debugger\_Window\_Str7:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_Str8
        db "TDeDeMainForm",0
    FD\_Find\_Debugger\_Window\_Str8:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_Str9
        db "TIdaWindow",0
    FD\_Find\_Debugger\_Window\_Str9:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_StrA
        db "TESTDBG",0
    FD\_Find\_Debugger\_Window\_StrA:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window

    push NULL
    call FD\_Find\_Debugger\_Window\_StrB
        db "kk1",0
    FD\_Find\_Debugger\_Window\_StrB:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window
    
    push NULL
    call FD\_Find\_Debugger\_Window\_StrC
        db "Eew75",0
    FD\_Find\_Debugger\_Window\_StrC:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window   
    
    push NULL
    call FD\_Find\_Debugger\_Window\_StrD
        db "Shadow",0
    FD\_Find\_Debugger\_Window\_StrD:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window        
    
    push NULL
    call FD\_Find\_Debugger\_Window\_StrE
        db "PEiD v0.94",0
    FD\_Find\_Debugger\_Window\_StrE:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window     
    
    push NULL
    call FD\_Find\_Debugger\_Window\_StrF
        db "Registry Monitor - Sysinternals: www.sysinternals.com",0
    FD\_Find\_Debugger\_Window\_StrF:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window     
 
    push NULL
    call FD\_Find\_Debugger\_Window\_Str10
        db "File Monitor - Sysinternals: www.sysinternals.com",0
    FD\_Find\_Debugger\_Window\_Str10:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window     

    push NULL
    call FD\_Find\_Debugger\_Window\_Str11
        db "Import REConstructor v1.6 FINAL (C) 2001-2003 MackT/uCF",0
    FD\_Find\_Debugger\_Window\_Str11:
    call dword ptr [ebx].xFindWindowA
    test eax, eax
    jnz Found\_FD\_Find\_Debugger\_Window
    jmp NotFound\_Found\_FD\_Find\_Debugger\_Window
Exit\_FD\_Find\_Debugger\_Window:
    pop ebx
    assume ebx : nothing
    mov esp, ebp
    pop ebp
    retn 04h
NotFound\_Found\_FD\_Find\_Debugger\_Window:
    xor eax, eax
    jmp Exit\_FD\_Find\_Debugger\_Window
Found\_FD\_Find\_Debugger\_Window:
    mov eax, 1
    jmp Exit\_FD\_Find\_Debugger\_Window     
End\_FD\_Find\_Debugger\_Window:
```

这种方面的核心思想是比对在进程表中是否出现了调试器进程名，如果出现则退出。  

不过这种方法很容易躲过，而且也可能造成误判。  

代码：
