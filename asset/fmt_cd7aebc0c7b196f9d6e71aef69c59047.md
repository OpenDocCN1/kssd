
```asm
.text:00011C60
.text:00011C60 ; ================= S U B R O U T I N E =======================
.text:00011C60
.text:00011C60 ; Attributes: bp-based frame
.text:00011C60
.text:00011C60 ; int __stdcall GetProcessFileName(int ProcessId, void *Buffer, int Size)
.text:00011C60 GetProcessFileName proc near            ; CODE XREF: CreateProcessNotifyRoutine+BCp
.text:00011C60
.text:00011C60 var_170           = dword ptr -170h
.text:00011C60 len               = dword ptr -16Ch
.text:00011C60 ImagePathBuffer   = dword ptr -168h
.text:00011C60 ImagePathName     = dword ptr -164h
.text:00011C60 ProcessPeb        = dword ptr -160h
.text:00011C60 ProcessInformation= PROCESS_BASIC_INFORMATION ptr -15Ch
.text:00011C60 ProcessParameters = dword ptr -144h
.text:00011C60 FreeSize          = dword ptr -140h
.text:00011C60 BaseAddress       = dword ptr -13Ch
.text:00011C60 var_138           = byte ptr -138h
.text:00011C60 var_137           = byte ptr -137h
.text:00011C60 status            = dword ptr -2Ch
.text:00011C60 ObjectAttributes  = OBJECT_ATTRIBUTES ptr -28h
.text:00011C60 Handle            = dword ptr -10h
.text:00011C60 ClientId          = CLIENT_ID ptr -0Ch
.text:00011C60 var_4             = dword ptr -4
.text:00011C60 ProcessId         = dword ptr  8
.text:00011C60 Buffer            = dword ptr  0Ch
.text:00011C60 Size              = dword ptr  10h
.text:00011C60
.text:00011C60                   push    ebp
.text:00011C61                   mov     ebp, esp
.text:00011C63                   sub     esp, 170h
.text:00011C69                   push    edi
.text:00011C6A                   mov     [ebp+var_170], ecx
.text:00011C70                   mov     [ebp+var_4], 0
.text:00011C77                   mov     [ebp+Handle], 0
.text:00011C7E                   mov     [ebp+ClientId.UniqueProcess], 0
.text:00011C85                   xor     eax, eax
.text:00011C87                   mov     [ebp+ClientId.UniqueThread], eax
.text:00011C8A                   cmp     [ebp+Buffer], 0
.text:00011C8E                   jnz     short loc_11C97
.text:00011C90                   xor     eax, eax
.text:00011C92                   jmp     loc_11F17
.text:00011C97 ; ---------------------------------------------------------------------------
.text:00011C97
.text:00011C97 loc_11C97:                       ; CODE XREF: GetProcessFileName+2Ej
.text:00011C97                   mov     [ebp+ObjectAttributes.Length], 18h
.text:00011C9E                   mov     [ebp+ObjectAttributes.RootDirectory], 0
.text:00011CA5                   mov     [ebp+ObjectAttributes.Attributes], 0
.text:00011CAC                   mov     [ebp+ObjectAttributes.ObjectName], 0
.text:00011CB3                   mov     [ebp+ObjectAttributes.SecurityDescriptor], 0
.text:00011CBA                   mov     [ebp+ObjectAttributes.SecurityQualityOfService], 0
.text:00011CC1                   mov     ecx, [ebp+ProcessId]
.text:00011CC4                   mov     [ebp+ClientId.UniqueProcess], ecx
.text:00011CC7                   lea     edx, [ebp+ClientId]
.text:00011CCA                   push    edx             ; ClientId
.text:00011CCB                   lea     eax, [ebp+ObjectAttributes]
.text:00011CCE                   push    eax             ; ObjectAttributes
.text:00011CCF                   push    1F0FFFh           ; DesiredAccess
.text:00011CD4                   lea     ecx, [ebp+Handle]
.text:00011CD7                   push    ecx             ; ProcessHandle
.text:00011CD8                   call    ds:ZwOpenProcess
.text:00011CDE                   mov     [ebp+status], eax
.text:00011CE1                   cmp     [ebp+status], 0
.text:00011CE5                   jl      loc_11F14
.text:00011CEB                   mov     edx, ds:FastMutex
.text:00011CF1                   cmp     dword ptr [edx+104h], 0
.text:00011CF8                   jz      loc_11F0A
.text:00011CFE                   mov     [ebp+var_138], 0
.text:00011D05                   mov     ecx, 40h
.text:00011D0A                   xor     eax, eax
.text:00011D0C                   lea     edi, [ebp+var_137]
.text:00011D12                   rep stosd
.text:00011D14                   stosw
.text:00011D16                   stosb
.text:00011D17                   mov     [ebp+BaseAddress], 0
.text:00011D21                   mov     [ebp+FreeSize], 1000h
.text:00011D2B                   push    PAGE_READWRITE  ; Protect
.text:00011D2D                   push    MEM_COMMIT        ; AllocationType
.text:00011D32                   lea     eax, [ebp+FreeSize]
.text:00011D38                   push    eax             ; AllocationSize
.text:00011D39                   push    0                 ; ZeroBits
.text:00011D3B                   lea     ecx, [ebp+BaseAddress]
.text:00011D41                   push    ecx             ; BaseAddress
.text:00011D42                   push    0FFFFFFFFh        ; ProcessHandle
.text:00011D44                   call    ds:ZwAllocateVirtualMemory
.text:00011D4A                   mov     [ebp+status], eax
.text:00011D4D                   cmp     [ebp+status], 0
.text:00011D51                   jl      loc_11F0A
.text:00011D57                   mov     [ebp+ProcessPeb], 0
.text:00011D61                   push    0                 ; ReturnLength
.text:00011D63                   push    18h               ; ProcessInformationLength
.text:00011D65                   lea     edx, [ebp+ProcessInformation]
.text:00011D6B                   push    edx             ; ProcessInformation
.text:00011D6C                   push    0                 ; ProcessInformationClass
.text:00011D6E                   mov     eax, [ebp+Handle]
.text:00011D71                   push    eax             ; ProcessHandle
.text:00011D72                   call    ds:ZwQueryInformationProcess
.text:00011D78                   test    eax, eax
.text:00011D7A                   jl      short loc_11D88
.text:00011D7C                   mov     ecx, [ebp+ProcessInformation.PebBaseAddress]
.text:00011D82                   mov     [ebp+ProcessPeb], ecx
.text:00011D88
.text:00011D88 loc_11D88:                       ; CODE XREF: GetProcessFileName+11Aj
.text:00011D88                   cmp     [ebp+ProcessPeb], 0
.text:00011D8F                   jz      short loc_11DDB
.text:00011D91                   push    0
.text:00011D93                   push    4
.text:00011D95                   mov     edx, [ebp+BaseAddress]
.text:00011D9B                   push    edx
.text:00011D9C                   mov     eax, [ebp+ProcessPeb]
.text:00011DA2                   add     eax, 10h
.text:00011DA5                   push    eax
.text:00011DA6                   mov     ecx, [ebp+Handle]
.text:00011DA9                   push    ecx
.text:00011DAA                   mov     edx, ds:FastMutex
.text:00011DB0                   call    dword ptr [edx+104h] ; NtReadVirtualMemory
.text:00011DB6                   mov     [ebp+status], eax
.text:00011DB9                   cmp     [ebp+status], 0
.text:00011DBD                   jnz     short loc_11DCF
.text:00011DBF                   mov     eax, [ebp+BaseAddress] ; ProcessParameters
.text:00011DC5                   mov     ecx, [eax]        ; _RTL_USER_PROCESS_PARAMETERS
.text:00011DC7                   mov     [ebp+ProcessParameters], ecx
.text:00011DCD                   jmp     short loc_11DD9
.text:00011DCF ; ---------------------------------------------------------------------------
.text:00011DCF
.text:00011DCF loc_11DCF:                       ; CODE XREF: GetProcessFileName+15Dj
.text:00011DCF                   mov     [ebp+ProcessParameters], 20000h
.text:00011DD9
.text:00011DD9 loc_11DD9:                       ; CODE XREF: GetProcessFileName+16Dj
.text:00011DD9                   jmp     short loc_11DE5
.text:00011DDB ; ---------------------------------------------------------------------------
.text:00011DDB
.text:00011DDB loc_11DDB:                       ; CODE XREF: GetProcessFileName+12Fj
.text:00011DDB                   mov     [ebp+ProcessParameters], 20000h
.text:00011DE5
.text:00011DE5 loc_11DE5:                       ; CODE XREF: GetProcessFileName:loc_11DD9j
.text:00011DE5                   mov     edx, [ebp+ProcessParameters]
.text:00011DEB                   add     edx, RTL_USER_PROCESS_PARAMETERS.ImagePathName
.text:00011DEE                   mov     [ebp+ImagePathName], edx
.text:00011DF4                   push    0
.text:00011DF6                   push    8
.text:00011DF8                   mov     eax, [ebp+BaseAddress]
.text:00011DFE                   push    eax
.text:00011DFF                   mov     ecx, [ebp+ImagePathName]
.text:00011E05                   push    ecx
.text:00011E06                   mov     edx, [ebp+Handle]
.text:00011E09                   push    edx
.text:00011E0A                   mov     eax, ds:FastMutex
.text:00011E0F                   call    dword ptr [eax+104h] ; NtReadVirtualMemory
.text:00011E15                   mov     [ebp+status], eax
.text:00011E18                   cmp     [ebp+status], 0
.text:00011E1C                   jl      loc_11EEF
.text:00011E22                   mov     ecx, [ebp+BaseAddress]
.text:00011E28                   mov     edx, [ecx+4]
.text:00011E2B                   mov     [ebp+ImagePathBuffer], edx
.text:00011E31                   mov     eax, [ebp+BaseAddress]
.text:00011E37                   movzx   ecx, word ptr [eax]
.text:00011E3A                   mov     [ebp+len], ecx
.text:00011E40                   mov     edx, [ebp+ImagePathBuffer]
.text:00011E46                   cmp     edx, [ebp+ProcessParameters]
.text:00011E4C                   jnb     short loc_11E60
.text:00011E4E                   mov     eax, [ebp+ImagePathBuffer]
.text:00011E54                   add     eax, [ebp+ProcessParameters]
.text:00011E5A                   mov     [ebp+ImagePathBuffer], eax
.text:00011E60
.text:00011E60 loc_11E60:                       ; CODE XREF: GetProcessFileName+1ECj
.text:00011E60                   cmp     [ebp+len], 0FFFh
.text:00011E6A                   jbe     short loc_11E76
.text:00011E6C                   mov     [ebp+len], 0FFFh
.text:00011E76
.text:00011E76 loc_11E76:                       ; CODE XREF: GetProcessFileName+20Aj
.text:00011E76                   push    0
.text:00011E78                   mov     ecx, [ebp+len]
.text:00011E7E                   push    ecx
.text:00011E7F                   mov     edx, [ebp+BaseAddress]
.text:00011E85                   push    edx
.text:00011E86                   mov     eax, [ebp+ImagePathBuffer]
.text:00011E8C                   push    eax
.text:00011E8D                   mov     ecx, [ebp+Handle]
.text:00011E90                   push    ecx
.text:00011E91                   mov     edx, ds:FastMutex
.text:00011E97                   call    dword ptr [edx+104h]
.text:00011E9D                   mov     [ebp+status], eax
.text:00011EA0                   cmp     [ebp+status], 0
.text:00011EA4                   jl      short loc_11EEF
.text:00011EA6                   mov     eax, [ebp+len]
.text:00011EAC                   add     eax, 2
.text:00011EAF                   cmp     eax, [ebp+Size]
.text:00011EB2                   jbe     short loc_11EB8
.text:00011EB4                   xor     eax, eax
.text:00011EB6                   jmp     short loc_11F17
.text:00011EB8 ; ---------------------------------------------------------------------------
.text:00011EB8
.text:00011EB8 loc_11EB8:                       ; CODE XREF: GetProcessFileName+252j
.text:00011EB8                   mov     ecx, [ebp+len]
.text:00011EBE                   push    ecx             ; size_t
.text:00011EBF                   mov     edx, [ebp+BaseAddress]
.text:00011EC5                   push    edx             ; void *
.text:00011EC6                   mov     eax, [ebp+Buffer]
.text:00011EC9                   push    eax             ; void *
.text:00011ECA                   call    memcpy
.text:00011ECF                   add     esp, 0Ch
.text:00011ED2                   mov     ecx, [ebp+Buffer]
.text:00011ED5                   push    ecx             ; wchar_t *
.text:00011ED6                   call    ConvertToNormalPath
.text:00011EDB                   mov     edx, [ebp+Buffer]
.text:00011EDE                   push    edx             ; wchar_t *
.text:00011EDF                   call    ds:_wcsupr
.text:00011EE5                   add     esp, 4
.text:00011EE8                   mov     [ebp+var_4], 1
.text:00011EEF
.text:00011EEF loc_11EEF:                       ; CODE XREF: GetProcessFileName+1BCj
.text:00011EEF                       ; GetProcessFileName+244j
.text:00011EEF                   push    8000h           ; FreeType
.text:00011EF4                   lea     eax, [ebp+FreeSize]
.text:00011EFA                   push    eax             ; FreeSize
.text:00011EFB                   lea     ecx, [ebp+BaseAddress]
.text:00011F01                   push    ecx             ; BaseAddress
.text:00011F02                   push    0FFFFFFFFh        ; ProcessHandle
.text:00011F04                   call    ds:ZwFreeVirtualMemory
.text:00011F0A
.text:00011F0A loc_11F0A:                       ; CODE XREF: GetProcessFileName+98j
.text:00011F0A                       ; GetProcessFileName+F1j
.text:00011F0A                   mov     edx, [ebp+Handle]
.text:00011F0D                   push    edx             ; Handle
.text:00011F0E                   call    ds:ZwClose
.text:00011F14
.text:00011F14 loc_11F14:                       ; CODE XREF: GetProcessFileName+85j
.text:00011F14                   mov     eax, [ebp+var_4]
.text:00011F17
.text:00011F17 loc_00011F17:                     ; CODE XREF: GetProcessFileName+32j
.text:00011F17                       ; GetProcessFileName+256j
.text:00011F17                   pop     edi
.text:00011F18                   mov     esp, ebp
.text:00011F1A                   pop     ebp
.text:00011F1B                   ret     0Ch
.text:00011F1B GetProcessFileName endp
.text:00011F1B
```
