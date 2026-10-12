
# 检测虚拟机代码(MASM)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-633.htm`](https://www.pediy.com/kssd/pediy09/pediy09-633.htm)

> 检测虚拟机的代码

> 引用:

```masm
.386  
.model flat, stdcall  
option casemap:none   

; VMware hessam salehi (kernex)  
; HKEY_LOCAL_MACHINE\\software\\VMWare, Inc.\\VMware tools  
  ; include .inc  
  include  \\MASM32\\include\\windows.inc  
  include  \\MASM32\\include\\user32.inc  
  include  \\MASM32\\include\\kernel32.inc  
  include  \\MASM32\\include\\advapi32.inc  
  ; include .lib  
  includelib  \\MASM32\\lib\\user32.lib  
  includelib  \\MASM32\\lib\\kernel32.lib  
  includelib  \\MASM32\\lib\\advapi32.lib   

.data  
MsgCaption      db “VMware Detector 1.0″,0  
Msgare          db “you are in a vmware !!”,0  
Msgnot  db “you are not in a vmware !!”,0  
szTestKey       db “software\\VMWare, Inc.\\VMware tools”,0  
hKey  db “InstallPath”,0  
.code  
start:  
 INVOKE     RegOpenKeyEx, HKEY_LOCAL_MACHINE, addr szTestKey, 0,\\  
          KEY_WRITE or KEY_READ, addr hKey  
 .if eax == ERROR_SUCCESS  
 invoke MessageBox, NULL,addr Msgare, addr MsgCaption, MB_OK  
 .else  
 invoke MessageBox, NULL,addr Msgnot,  addr MsgCaption, MB_OK  
 .endif  
 invoke ExitProcess,NULL  
end start 
```

> 引用:

```masm
start:  
  PUSH EBP  
  MOV EBP,ESP  
       try_again:  
       …..  
   rdtsc          
  mov ebx,eax  
  rdtsc  
  sub eax,ebx  
  push eax  
  cmp eax,1       ; infrequent some CPUs return the value 1,  
  jz try_again    ; which seems to be a bug! in this case we try again.  
  Invoke printf, ADDR print_result,eax   
  pop eax  
  cmp eax,0200h  
  jb no_vm  
  Invoke printf, ADDR print_vm ; you ARE in a VM  
  jmp finish  
no_vm:  
  Invoke printf, ADDR print_novm ; You are NOT in a VM  
finish:  
  MOV ESP,EBP  
  POP EBP  
  RETN  
```
