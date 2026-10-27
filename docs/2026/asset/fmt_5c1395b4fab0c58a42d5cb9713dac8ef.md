
## `NtCreateProcessEx` 函数

```
PAGE:004B4A0C ; NTSTATUS __stdcall NtCreateProcessEx(PHANDLE ProcessHandle, ACCESS_MASK DesiredAccess, POBJECT_ATTRIBUTES ObjectAttributes, HANDLE ParentProcess, ULONG Flags, HANDLE SectionHandle, HANDLE DebugPort, HANDLE ExceptionPort, BOOLEAN InJob)

PAGE:004B4A0C _NtCreateProcessEx@36 proc near         ; CODE XREF: NtCreateProcess(x,x,x,x,x,x,x,x)+3D↓p

PAGE:004B4A0C                                         ; DATA XREF: .text:0040D8E0↑o

PAGE:004B4A0C

PAGE:004B4A0C var_1C          = dword ptr -1Ch

PAGE:004B4A0C ms_exc          = CPPEH_RECORD ptr -18h

PAGE:004B4A0C ProcessHandle   = dword ptr  8

PAGE:004B4A0C DesiredAccess   = dword ptr  0Ch

PAGE:004B4A0C ObjectAttributes= dword ptr  10h

PAGE:004B4A0C ParentProcess   = dword ptr  14h

PAGE:004B4A0C Flags           = dword ptr  18h

PAGE:004B4A0C SectionHandle   = dword ptr  1Ch

PAGE:004B4A0C DebugPort       = dword ptr  20h

PAGE:004B4A0C ExceptionPort   = dword ptr  24h

PAGE:004B4A0C InJob           = byte ptr  28h

PAGE:004B4A0C                 push    0Ch

PAGE:004B4A0E                 push    offset stru_4285B0

PAGE:004B4A13                 call    __SEH_prolog

PAGE:004B4A18                 mov     eax, large fs:124h ; 获取当前线程地址

PAGE:004B4A1E                 xor     edx, edx

PAGE:004B4A20                 cmp     [eax+140h], dl  ; PreviousMode是否为0

PAGE:004B4A26                 jz      loc_4E9036
```

如果不为0，说明是从用户层模式调用的函数，此时就需要对句柄是否可读进行验证

```
PAGE:004B4A2F                 mov     ecx, [ebp+ProcessHandle]

PAGE:004B4A32                 mov     eax, _MmUserProbeAddress

PAGE:004B4A37                 cmp     ecx, eax

PAGE:004B4A39                 jnb     loc_52E165

PAGE:004B4A3F loc_4B4A3F:                             ; CODE XREF: NtCreateProcessEx(x,x,x,x,x,x,x,x,x)+7975B↓j

PAGE:004B4A3F                 mov     eax, [ecx]

PAGE:004B4A41                 mov     [ecx], eax
```

接着判断父进程句柄是否为空，如果是则报错

```
PAGE:004B4A47 loc_4B4A47:                             ; CODE XREF: NtCreateProcessEx(x,x,x,x,x,x,x,x,x)+3462D↓j

PAGE:004B4A47                 cmp     [ebp+ParentProcess], edx ; 判断父进程句柄是否为NULL

PAGE:004B4A4A                 jz      loc_4E903E

            。。。

PAGE:004E903E loc_4E903E:                             ; CODE XREF: NtCreateProcessEx(x,x,x,x,x,x,x,x,x)+3E↑j

PAGE:004E903E                 mov     eax, STATUS_INVALID_PARAMETER

PAGE:004E9043                 jmp     loc_4B4A6E
```

如果不是则将参数入栈以后调用`PspCreateProcess`

```
PAGE:004B4A50                 push    dword ptr [ebp+InJob] ; int

PAGE:004B4A53                 push    [ebp+ExceptionPort] ; HANDLE

PAGE:004B4A56                 push    [ebp+DebugPort] ; HANDLE

PAGE:004B4A59                 push    [ebp+SectionHandle] ; HANDLE

PAGE:004B4A5C                 push    [ebp+Flags]     ; int

PAGE:004B4A5F                 push    [ebp+ParentProcess] ; Handle

PAGE:004B4A62                 push    [ebp+ObjectAttributes] ; int

PAGE:004B4A65                 push    [ebp+DesiredAccess] ; AccessMask

PAGE:004B4A68                 push    ecx             ; int

PAGE:004B4A69                 call    _PspCreateProcess@36 ; PspCreateProcess(x,x,x,x,x,x,x,x,x)
```

## `PspCreateProcess` 函数

在`PspCreateProcess`函数中首先会对局部变量进行赋值，然后会判断创建标志中是否有相关标志

```
PAGE:004B450E ; int __stdcall PspCreateProcess(PHANDLE ProcessHandle, ACCESS_MASK DesiredAccess, POBJECT_ATTRIBUTES ObjectAttributes, HANDLE ParentProcess, ULONG Flags, HANDLE SectionHandle, HANDLE DebugPort, HANDLE ExceptionPort, ULONG JobMemberLevel)

PAGE:004B450E _PspCreateProcess@36 proc near          ; CODE XREF: NtCreateProcessEx(x,x,x,x,x,x,x,x,x)+5D↓p

PAGE:004B450E                                         ; PsCreateSystemProcess(x,x,x)+1B↓p ...

PAGE:004B450E

PAGE:004B450E var_12C         = dword ptr -12Ch

PAGE:004B450E var_B8          = dword ptr -0B8h

PAGE:004B450E var_9C          = dword ptr -9Ch

PAGE:004B450E SubjectSecurityContext= _SECURITY_SUBJECT_CONTEXT ptr -98h

PAGE:004B450E var_88          = dword ptr -88h

PAGE:004B450E var_CurThraed   = dword ptr -84h

PAGE:004B450E ParentProcObject= dword ptr -80h

PAGE:004B450E var_7C          = dword ptr -7Ch

PAGE:004B450E var_78          = dword ptr -78h

PAGE:004B450E var_74          = dword ptr -74h

PAGE:004B450E var_70          = dword ptr -70h

PAGE:004B450E var_6C          = dword ptr -6Ch

PAGE:004B450E var_WorkingSetMaximum= dword ptr -68h

PAGE:004B450E var_Affinity    = dword ptr -64h

PAGE:004B450E var_WorkingSetMinimum= dword ptr -60h

PAGE:004B450E var_5C          = dword ptr -5Ch

PAGE:004B450E var_CurEProcess = dword ptr -58h

PAGE:004B450E var_ProcObject  = dword ptr -54h

PAGE:004B450E var_50          = dword ptr -50h

PAGE:004B450E AccessStatus    = dword ptr -4Ch

PAGE:004B450E SpinLock        = dword ptr -48h

PAGE:004B450E var_44          = dword ptr -44h

PAGE:004B450E var_40          = dword ptr -40h

PAGE:004B450E var_3C          = dword ptr -3Ch

PAGE:004B450E MemoryAllocated = byte ptr -38h

PAGE:004B450E var_34          = dword ptr -34h

PAGE:004B450E BugCheckParameter1= dword ptr -30h

PAGE:004B450E SecurityDescriptor= dword ptr -2Ch

PAGE:004B450E var_28          = dword ptr -28h

PAGE:004B450E var_22          = byte ptr -22h

PAGE:004B450E AccessMode      = byte ptr -21h

PAGE:004B450E var_1D          = byte ptr -1Dh

PAGE:004B450E var_ParentProcObject= dword ptr -1Ch

PAGE:004B450E ms_exc          = CPPEH_RECORD ptr -18h

PAGE:004B450E ProcessHandle   = dword ptr  8

PAGE:004B450E AccessMask      = dword ptr  0Ch

PAGE:004B450E ObjectAttributes= dword ptr  10h

PAGE:004B450E ParentProcessHandle= dword ptr  14h

PAGE:004B450E Flags           = dword ptr  18h

PAGE:004B450E SectionHandle   = dword ptr  1Ch

PAGE:004B450E DebugPort       = dword ptr  20h

PAGE:004B450E ExceptionPort   = dword ptr  24h

PAGE:004B450E JobMemberLevel  = dword ptr  28h

PAGE:004B450E                 push    11Ch

PAGE:004B4513                 push    offset stru_4285C0

PAGE:004B4518                 call    __SEH_prolog

PAGE:004B451D                 mov     eax, large fs:124h ; 获取ETHREAD

PAGE:004B4523                 mov     [ebp+var_CurThraed], eax

PAGE:004B4529                 mov     cl, [eax+140h]  ; 获取PrevMode

PAGE:004B452F                 mov     [ebp+AccessMode], cl

PAGE:004B4532                 mov     eax, [eax+44h]  ; 获取当前进程EPROCESS

PAGE:004B4535                 mov     [ebp+var_CurEProcess], eax

PAGE:004B4538                 xor     esi, esi

PAGE:004B453A                 mov     [ebp+var_1D], 0

PAGE:004B453E                 mov     [ebp+SpinLock], esi

PAGE:004B4541                 mov     [ebp+var_44], esi

PAGE:004B4544                 test    [ebp+Flags], 0FFFFFFF0h ; 是否具有PROCESS_CREATE_FLAGS_LEGAL_MSK标志

PAGE:004B454B                 jnz     loc_52DE13
```

如果没有，则会返回错误

```
PAGE:0052DE13 loc_52DE13:                             ; CODE XREF: PspCreateProcess(x,x,x,x,x,x,x,x,x)+3D↑j

PAGE:0052DE13                 mov     eax, STATUS_INVALID_PARAMETER

PAGE:0052DE18                 jmp     loc_4B49D0
```

接下来会判断是否有父进程句柄

```
PAGE:004B4551                 cmp     [ebp+ParentProcessHandle], esi

PAGE:004B4554                 jz      loc_4F1B06
```

如果没有，则会对会父进程的`EPROCESS`赋值为空且使用`KeActiveProcessors`来赋值`eax`

```
PAGE:004F1B06 loc_4F1B06:                          

PAGE:004F1B06                 mov     [ebp+var_ParentProcObject], esi

PAGE:004F1B09                 mov     eax, _KeActiveProcessors

PAGE:004F1B0E                 jmp     loc_4B458F
```

否则会调用`ObReferenceObjectByHandle`来获得父进程的对象地址，并且使用父进程的`Affinity`来赋值`eax`
