
# 第四题

> 原文：[`www.pediy.com/kssd/pediy10/78993.html`](https://www.pediy.com/kssd/pediy10/78993.html)

首先利用自定位弹出当前 IP，判断如果大于 `0x80000000` 即为驱动（打开 3G 开关也是如此，LINK 时考虑了模块定位情况），不大于 `0x80000000` 即为应用模式。

驱动中利用传入的 `DriverObject` 参数搜 `nt`！应用程序中利用 `SEH` 搜 `kernel32`。搜到后一切都好办了。

```asm
.386

.model flat, stdcall

option casemap:none

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

;                                  I N C L U D E   F I L E S                                         

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

include windows.inc

include masm32.inc

include user32.inc

include hal.inc

.data

NtosFunction  equ this byte

db 'DbgPrint',0

db 00h

KernelFunction  equ this byte

db 'GetSystemDirectoryA',0

db 'ReadFile',0

db 'WinExec',0

db 'OpenProcess',0

db 'VirtualAllocEx',0

db 'WriteProcessMemory',0

db 'CreateRemoteThread',0

db 'CloseHandle',0

db 'lstrcat',0

db 'CreateFileA',0

db 'LoadLibraryA',0

db 'GetProcAddress',0

db 'CreateToolhelp32Snapshot',0

db 'Process32First',0

db 'Process32Next',0

db 00h

UserFunction  equ this byte

db 'MessageBoxA',0

db 00h

URLMonFunction  equ this byte

db 'URLDownloadToFileA',0

db 00h

align dword

NtosAddress  equ this byte

_DbgPrint  dd 0

KernelAddress  equ this byte

_GetSystemDirectoryA  dd 0

_ReadFile    dd 0

_WinExec    dd 0

_OpenProcess    dd 0

_VirtualAllocEx    dd 0

_WriteProcessMemory  dd 0

_CreateRemoteThread  dd 0

_CloseHandle    dd 0

_lstrcat    dd 0

_CreateFileA    dd 0

_LoadLibraryA    dd 0

_GetProcAddress    dd 0

_CreateToolhelp32Snapshot  dd 0

_Process32First    dd 0

_Process32Next    dd 0

UserAddress  equ this byte

_MessageBoxA  dd 0

URLMonAddress  equ this byte

_URLDownloadToFileA  dd 0

.code

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

;                                       DriverEntry                                                 

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

GetAPI proc

  mov edx,esi

     g1:

  cmp byte ptr [esi],0

  jz g2

  inc esi

  jmp g1

     g2:

  inc esi

  sub esi,edx

  mov ecx,esi    ;ecx 中是字符串的长度

  mov ebx,eax    ;ebx 是基址

  add eax,dword ptr[eax+3ch]

  mov eax,dword ptr[eax+78h]

  add eax,ebx

  add eax,20h      ;指向函数名地址表

  mov edi,dword ptr[eax]

  add edi,ebx    ;edi 指向地址数组第一项

  push eax

  push edi

  mov eax,edi

     g3:

  mov edi,[eax]

  add edi,ebx    ;edi 指向函数名

  mov esi,edx

  push ecx

  cld

  repz cmpsb

  pop ecx

  jz g4

  add eax,4h

  jmp g3

     g4:

  pop edi

  sub eax,edi

  mov edi,eax

  pop eax

  add edi,ebx

  add edi,dword ptr[eax-4h]

  mov eax,dword ptr[edi]

  add eax,ebx

  ret

GetAPI endp

GetAPIs proc

      @begin:

  cmp byte ptr[esi],0

  jz finish

  push eax  

  push esi

  push edi

  call GetAPI

  pop edi

  pop esi

  cld

  stosd

  add esi,ecx

  pop eax

  jmp @begin

     finish:

  ret

GetAPIs endp

GetKernel32Base proc

  assume fs:nothing

  mov eax,fs:[30h]

  mov eax,[eax+0ch]

  mov eax,[eax+1ch]

  mov eax,[eax]

  mov eax,[eax+8h]

  ret

GetKernel32Base endp

GetNtBase proc

@@:      

    and     ax, 0F001h

        dec     eax

        cmp     word ptr [eax], 'ZM'

        jnz     @b

        ret

GetNtBase endp

MyGetProcAddress proc hModule:HMODULE,lpProcName:LPCSTR

local ProcNameLen:dword  

  push ebx

  push esi

  push edi

  mov esi,lpProcName

  cmp esi,10000h

  ja  @f

  mov ecx,lpProcName   ;lpProcName 为函数序号

  mov esi,hModule  

  mov edi,dword ptr[esi+3ch]

  add edi,esi  

  mov edi,[edi+78h]

  add edi,esi

  jmp Ordinals

@@:  

  xor ecx,ecx        ;lpProcName 为函数指针

GetlpProcNameLength:  

  cmp byte ptr[esi],0

  jz @f

  inc cl

  inc esi

  jmp GetlpProcNameLength

  @@:              ;cl=ProcName Length

  mov ProcNameLen,ecx

  mov edi,hModule  

  mov esi,dword ptr[edi+3ch]

  add esi,edi ;esi->PE HEADER

  mov edi,[esi+78h]

  add edi,hModule

assume edi:ptr IMAGE_EXPORT_DIRECTORY  

GetExportNameList:  

  mov   ebx,[edi].AddressOfNames ;//得到输出函数名表

  add   ebx,hModule     ;ebx->AddressOfNames(函数名字的指针地址).

  xor   eax,eax      ;//函数序号计数

  mov   edx,hModule     ;//暂存模块句柄;edx->ntoskrnl

  push edi   ;保存 EDI

LoopFindApiStr:

  mov ecx,ProcNameLen

  add   ebx,04        

  inc   eax          ;//增加函数计数

  mov   edi,dword ptr[ebx]

  add   edi,edx      ;//得到一个 Api 函数名字符串.edi->函数名

  StrGetProcAddress:  

  mov esi,lpProcName      ;//得到 Api 名字字符串

  cmpsd;比较前 4 个字符是否相等  

  jnz   short LoopFindApiStr  ;eax=函数名的 INDEX    

  sub   cl,4        ;//比较剩余的 GetProcAddress 串

  cld

  Goon:

  cmpsb

  jnz   short LoopFindApiStr  ;eax=函数名的 INDEX

  loop Goon

  pop edi ;恢复 EDI

  mov   esi,edx  

  mov ebx,[edi].AddressOfNameOrdinals

  add ebx,esi      ;//取函数序号地址列表,ebx->AddresssOfNameOrdinals

  movzx ecx,word ptr [ebx+eax*2]  

Ordinals:

  mov   ebx,[edi].AddressOfFunctions

  add   ebx,esi      ;//得到 ntoskrnl 函数地址列表

  mov   ebx,dword ptr[ebx+ecx*4]

  add   ebx,esi      ;//计算函数地址  

  mov   eax,ebx      ;eax=API 函数地址,esi=Kernel32.dll hModule

  pop edi

  pop esi

  pop ebx

  ret

MyGetProcAddress endp

DriverEntry proc pDriverObject:dword, pusRegistryPath:dword

  call @f

@@:

  pop eax

  .if eax > 80000000h

  pushad

  call @F

     @@:

  pop ebp

  sub ebp,offset @B

  mov eax,dword ptr[esp + 9*4]

  call GetNtBase

  call @f

  db "DbgPrint",0

  @@:

  push eax

  call MyGetProcAddress

  call @f

  db "zjjmj2002",0

@@:

  call eax

  pop eax

  popad

  .else

  pushad

  call @F

     @@:

  pop ebp

  sub ebp,offset @B

  call GetKernel32Base

  lea esi,[ebp+KernelFunction]

  lea edi,[ebp+KernelAddress]

  call GetAPIs

  call @F

  db  'user32',0

@@:

  call [ebp+\_LoadLibraryA\]

  lea esi,[ebp+UserFunction]

  lea edi,[ebp+UserAddress]

  call GetAPIs

  push 0

  call @f

  db "zjjmj2002",0

@@:

  call @f

  db "zjjmj2002",0

@@:

  push 0

  call [ebp+\_MessageBoxA\]

  popad

  .endif

  mov eax, 00C0000182h

  ret

DriverEntry endp

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

;                                                                                                     

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

end DriverEntry
```
