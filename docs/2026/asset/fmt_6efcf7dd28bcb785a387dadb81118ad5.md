
```
FT_INT1_IceBreakpoint:
    push ebp
    mov ebp, esp
    call Get_FT_INT1_IceBreakpoint_Eip
    Get_FT_INT1_IceBreakpoint_Eip:
    pop eax
    add eax, offset FT_INT1_IceBreakpoint_Exception - offset Get_FT_INT1_IceBreakpoint_Eip
    push eax
    assume fs : nothing
    push dword ptr fs:[0]
    mov dword ptr fs:[0], esp
    ;; reset eax
    xor eax, eax
    db 0CDh, 01h    ; int 1
    pop dword ptr fs:[0]
    add esp, 04h
    test eax, eax
    jz Found_FT_INT1_IceBreakpoint
    jmp NotFound_FT_INT1_IceBreakpoint
FT_INT1_IceBreakpoint_Exception:
    ;mov eax, dword ptr [esp+04h]
    ;cmp dword ptr [eax], 80000004h
    mov eax, dword ptr [esp+0ch]
    assume eax : ptr CONTEXT
    mov dword ptr [eax].regEax, 1
    inc dword ptr [eax].regEip
    xor eax, eax
    retn
Exit_FT_INT1_IceBreakpoint:
    mov esp, ebp
    pop ebp
    retn 0h
NotFound_FT_INT1_IceBreakpoint:
    xor eax, eax
    jmp Exit_FT_INT1_IceBreakpoint
Found_FT_INT1_IceBreakpoint:
    mov eax, 1
    jmp Exit_FT_INT1_IceBreakpoint
End_FT_INT1_IceBreakpoint:
```

以下为检测Virtual PC 与 VMWARE的两个例子，这类检测技术大多都是通过与真实环境中不同的异常，指令实现的。严格的描述可以在其规定的文档中找到。

```
FV_VPC_Exception:

    push ebx
    push ecx

    call Get_FV_VPC_Exception_Eip
    Get_FV_VPC_Exception_Eip:
    pop ecx
    add ecx, offset FV_VPC_Exception_Exception - offset Get_FV_VPC_Exception_Eip
    assume fs : nothing
    push ecx
    push dword ptr fs:[0]
    mov dword ptr fs:[0], esp

    ;; Flag
    xor ebx, ebx
    ;; VPC function number
    mov eax, 1

    ;; call VPC
    db 0Fh,3Fh,07h,0Bh

    pop dword ptr fs:[0]
    add esp, 04h
    test ebx, ebx
    setz al

    pop ecx
    pop ebx
    retn 0h

FV_VPC_Exception_Exception:
    mov ecx, dword ptr [esp+0ch]
    assume ecx : ptr CONTEXT
    ;; ebx = -1 : not running
    ;; ebx = 0 : running
    mov dword ptr [ecx].regEbx, -1
    ;; skip past the call to VPC
    add dword ptr [ecx].regEip, 04h
    xor eax, eax
    retn
End_FV_VPC_Exception:

FV_VMWare_VMX:

    call IsInsideVMWare
    retn 0h
IsInsideVMWare:

    push ebx
    push ecx
    push edx

    assume fs : nothing
    call Get_IsInsideVMWare_Eip
    Get_IsInsideVMWare_Eip:
    pop eax
    add eax, offset IsInsideVMWare_Exception - offset Get_IsInsideVMWare_Eip
    push eax
    push dword ptr fs:[0]
    mov dword ptr fs:[0], esp

    mov    eax, 'VMXh'
    mov    ebx, 0         ; any value but MAGIC VALUE
    mov    ecx, 10        ; get VMWare version
    mov    edx, 'VX'     ; port number
    in     eax, dx         ; read port
                           ; on return EAX returns the VERSION
    cmp    ebx, 'VMXh'     ; is it a reply from VMWare?
    setz   al             ; set return value
    movzx eax, al

    pop dword ptr fs:[0]
    add esp, 04h

    pop edx
    pop ecx
    pop ebx

    retn 0h
IsInsideVMWare_Exception:
    mov eax, dword ptr [esp+0ch]
    assume eax : ptr CONTEXT
    inc dword ptr [eax].regEip
    retn
End_IsInsideVMWare:

End_FV_VMWare_VMX:
```

通过计算文件的长度与原先的对比，来确定是否进行过修改。来发现补丁。

```
FP_Check_FileSize:
FP_Check_FileSize_Arg_Win32Api      equ 08h
FP_Check_FileSize_StackSize         equ MAX_PATH
FP_Check_FileSize_szPath            equ -MAX_PATH
    push ebp
    mov ebp, esp
    sub esp, FP_Check_FileSize_StackSize

    push ebx
    push ecx
    push edx
    mov ebx, dword ptr [ebp+FP_Check_FileSize_Arg_Win32Api]
    assume ebx : ptr WIN32APIBASE

    push MAX_PATH
    lea eax, [ebp+FP_Check_FileSize_szPath]
    push eax
    push NULL
    call dword ptr [ebx].xGetModuleFileNameA
    test eax, eax
    jz NotFound_FP_Check_FileSize

    ;; create file
    push NULL
    push FILE_ATTRIBUTE_NORMAL
    push OPEN_ALWAYS
    push NULL
    push FILE_SHARE_READ
    push GENERIC_READ
    lea eax, [ebp+FP_Check_FileSize_szPath]
    push eax
    call dword ptr [ebx].xCreateFileA
    cmp eax, INVALID_HANDLE_VALUE
    jz NotFound_FP_Check_FileSize
    mov ecx, eax

    push NULL
    push eax
    call dword ptr [ebx].xGetFileSize
    mov edx, eax

    push ecx
    call dword ptr [ebx].xCloseHandle

    call Get_FP_Check_FileSize_Eip
    Get_FP_Check_FileSize_Eip:
    pop eax
    add eax, offset FP_Check_FileSize_Orig_Size - offset Get_FP_Check_FileSize_Eip
    cmp edx, eax
    jnz Found_FP_Check_FileSize
    jmp NotFound_FP_Check_FileSize

Exit_FP_Check_FileSize:
    pop edx
    pop ecx
    pop ebx
    assume ebx : nothing

    mov esp, ebp
    pop ebp
    retn 04h
NotFound_FP_Check_FileSize:
    xor eax, eax
    jmp Exit_FP_Check_FileSize
Found_FP_Check_FileSize:
    mov eax, 1
    jmp Exit_FP_Check_FileSize
FP_Check_FileSize_Orig_Size:
    dd 0
End_FP_Check_FileSize:
```

通过计算文件的CRC值来对比是否被Cracker进行过修改打过补丁。
