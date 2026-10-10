

# 奇虎 360 第 4 题的解法

> 原文：[`www.pediy.com/kssd/pediy10/79027.html`](https://www.pediy.com/kssd/pediy10/79027.html)

我的解法    比较臃肿  
  

代码:

```

;@echo off
;goto make



.386
.model flat, stdcall
option casemap:none

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                                  I N C L U D E   F I L E S                                        
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

include \\masm32\\include\\w2k\\ntstatus.inc
include \\masm32\\include\\w2k\\ntddk.inc
include \\masm32\\include\\w2k\\ntoskrnl.inc

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
; 数据段
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    .data

ShellCodeExe  db 0ebh,00eh,05bh,04bh,033h,0c9h,0b1h,09bh,080h,034h,00bh,0feh,0e2h,0fah,0ebh,005h
               db 0e8h,0edh,0ffh,0ffh,0ffh,017h,07bh,0feh,0feh,0feh,0a1h,09ah,05fh,0ceh,0feh,0feh
               db 0feh,075h,0beh,0f2h,075h,08eh,0e2h,053h,075h,096h,0f6h,075h,009h,094h,0fch,0a7h
               db 016h,0dbh,0feh,0feh,0feh,01ch,007h,096h,0cdh,0cch,0feh,0feh,096h,08bh,08dh,09bh
               db 08ch,0aah,001h,0e8h,075h,016h,094h,0ffh,0a7h,016h,0f2h,0feh,0feh,0feh,01ch,007h
               db 0afh,0a9h,0a9h,0afh,001h,0a8h,0f6h,001h,0a8h,0fah,0afh,0a8h,075h,08bh,0c2h,075h
               db 08ah,0d0h,086h,0fdh,00bh,0a8h,075h,088h,0deh,0fdh,00bh,0cdh,037h,0b7h,0bfh,053h
               db 0fdh,03bh,0cdh,025h,0f1h,040h,0eeh,0c4h,028h,08ah,0f6h,03fh,035h,0f9h,0fdh,024h
               db 0beh,015h,00fh,0c5h,0e1h,08bh,019h,0a0h,075h,0a0h,0dah,0fdh,023h,098h,075h,0f2h
               db 0b5h,075h,0a0h,0e2h,0fdh,023h,075h,0fah,075h,0fdh,03bh,055h,0a0h,0a7h,03dh,016h
               db 088h,001h,001h,001h,0cch,08ah,06fh,0f2h,09dh,077h,02fh,0b1h,094h,0f4h,0c6h,0e0h
               db 'i am in exe',0;


ShellCodeDll     db 0E9h,096h,000h,000h,000h,05Ah,064h,0A1h,030h,000h,000h,000h,08Bh,040h
            db 0Ch,08Bh,070h,01Ch,0ADh,08Bh,040h,008h,050h,052h,06Ah,00Ch,0E8h,01Eh
            db 00h,000h,000h,05Bh,083h,0C3h,00Dh,053h,0FFh,0D0h,083h,0C3h,007h,053h
            db 6Ah,00Bh,0E8h,00Ch,000h,000h,000h,05Bh,083h,0C3h,00Ch,06Ah,000h,053h
            db 53h,06Ah,000h,0FFh,0D0h,08Bh,0D8h,083h,0C0h,03Ch,08Bh,000h,003h,0C3h
            db 80h,038h,050h,075h,049h,08Bh,040h,078h,003h,0C3h,050h,08Bh,0C8h,08Bh
            db 49h,014h,08Bh,040h,020h,003h,0C3h,055h,08Bh,0E8h,033h,0D2h,051h,08Bh
            db 00h,003h,0C3h,08Bh,0F8h,08Bh,074h,024h,014h,08Bh,04Ch,024h,010h,0FCh
            db 0F3h,0A6h,075h,017h,083h,0C4h,004h,08Bh,044h,024h,004h,08Bh,040h,01Ch
            db 03h,0C3h,0C1h,0E2h,002h,003h,0C2h,08Bh,000h,003h,0C3h,0EBh,00Bh,042h
            db 83h,0C5h,004h,08Bh,0C5h,059h,0E2h,0CCh,033h,0C0h,05Dh,059h,0C2h,004h
            db 00h,0E8h,065h,0FFh,0FFh,0FFh,04Ch,06Fh,061h,064h,04Ch,069h,062h,072h
            db 61h,072h,079h,041h,000h,075h,073h,065h,072h,033h,032h,000h,04Dh,065h
            db 73h,073h,061h,067h,065h,042h,06Fh,078h,041h,000h
            db 'i am in the dll',0;
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

;                                         C O D E                                                   
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

.code


;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                              GetKernel32                                   
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
GetKernel32 proc uses esi
  assume fs:nothing
  mov eax,fs:\[30h\]
  assume fs:error
  mov eax,\[eax + 0ch\]
  mov esi,\[eax + 1ch\]
  lodsd
  mov eax,\[eax+08h\] 
  ret
GetKernel32 endp


;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                              MyGetProcAddress                                   
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
MyGetProcAddress proc uses ebx ecx esi edi ImageBase:DWORD,FuncName:DWORD
  LOCAL FunNameArray  :DWORD
  LOCAL PE  :DWORD
  LOCAL Count  :DWORD
  LOCAL IED  :DWORD
  LOCAL iedN  :DWORD
  LOCAL flen  :DWORD

  mov Count,0

        mov eax,FuncName
  test eax,eax
  je tmpret
  lea edx,DWORD PTR ds:\[eax+1\]
        strloop:
  mov cl,BYTE PTR ds:\[eax\]
  inc eax
  test cl,cl
  jnz strloop

  sub eax,edx
        tmpret:
   
      mov flen,eax

  mov eax,ImageBase
  add eax,3ch      
  mov eax,\[eax\]     
  add eax,ImageBase    
  cmp DWORD PTR \[eax\],00004550h
  jne NotFound     
  mov PE,eax
  mov eax,\[eax+78h\]
  add eax,ImageBase
  mov IED,eax      
  mov eax,\[eax+0ch\]
  add eax,ImageBase    
  mov iedN,eax
  mov eax,IED
  mov eax,\[eax+20h\]
  add eax,ImageBase
  mov FunNameArray,eax  
  mov ecx,IED
  mov ecx,\[ecx+14h\]    
FindLoop:
  mov eax,\[eax\]
  add eax,ImageBase
  mov esi,FuncName
  mov edi,eax
  push ecx
  mov ecx,flen      
  cld
  repe cmpsb
  jne FindNext    
  add esp,4        
  mov eax,IED
  mov eax,\[eax+24h\]
  add eax,ImageBase
  shl Count,1  

  add eax,Count
  mov eax,\[eax\]      
  and eax,0000ffffh
  mov ebx,eax
  mov eax,IED
  mov eax,\[eax+1ch\]
  add eax,ImageBase
  shl ebx,2
  mov eax,\[eax+ebx\]
  add eax,ImageBase
  jmp Found
FindNext:
  inc Count        
  add FunNameArray,4
  mov eax,FunNameArray  
  pop ecx         
  loop FindLoop     
NotFound:
  xor eax,eax        
Found:
  ret
MyGetProcAddress endp

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::



;                             usermain                                   
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
usermain proc
  LOCAL hKer32:DWORD
  LOCAL pGetModuleHandle:DWORD
  invoke GetKernel32
  mov hKer32,eax

  call GetHandle
  db "GetModuleHandleA",0
GetHandle:
  push hKer32
  call MyGetProcAddress
  mov pGetModuleHandle,eax

  push NULL
  call pGetModuleHandle

        cmp eax,10000h
  jne dll
  lea eax,ShellCodeExe
        call eax
        jmp  TheEnd

dll:
            call @F
            @@:
            pop ebx
            sub ebx,offset @B
            lea eax,\[ebx+offset ShellCodeDll\]
      call eax
            jmp  TheEnd
TheEnd:
  ret
usermain endp

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                              GetKernel                                  
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
GetKernel proc uses ecx
    mov eax, \[ebp+4\]
FindMZ:
    and eax,0fffff000h
    cmp word ptr \[eax\],'ZM'
    jne MoveUp
    mov ecx,\[eax+3ch\]
    add ecx,eax
    cmp word ptr \[ecx\],'EP'

    je Found 
MoveUp:
    sub eax ,1000h
    jmp FindMZ
Found:
    ret
GetKernel endp

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                              MyDbgPrint                                   
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
MyDbgPrint proc hKer:DWORD
  call DbgPrt
  db "DbgPrint",0
DbgPrt:
  push hKer
  call MyGetProcAddress
  call MyDbgPrt
  db "I am in the kernel!",13,10,0
MyDbgPrt:
  call eax
  ret
MyDbgPrint endp

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                                       DriverEntry                                                 
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

DriverEntry proc pDriverObject:PDRIVER\_OBJECT, pusRegistryPath:PUNICODE\_STRING


  call tmpadd1
tmpadd1:
  pop eax
  cmp eax,7fffffffh
  ja kernel
  invoke usermain
  jmp useret
kernel:
  invoke GetKernel
  invoke MyDbgPrint,eax
  mov eax, STATUS\_DEVICE\_CONFIGURATION\_ERROR
  ret 
useret:
  ret 0
  

DriverEntry endp

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                                                                                                   
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

end DriverEntry

:make

set drv=kyo

\\masm32\\bin\\ml /nologo /c /coff %drv%.bat
\\masm32\\bin\\link /nologo /driver /base:0x10000 /align:32 /out:%drv%.sys /subsystem:native %drv%.obj

del %drv%.obj

echo.

```