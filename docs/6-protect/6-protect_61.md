

# 用 Hash 搜索 API 演示

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-695.htm`](https://www.pediy.com/kssd/pediy08/pediy8-695.htm)

.386  
.model flat,stdcall  
option casemap:none  
  
include user32.inc  
includelib user32.lib  
  
FindKernel32 proto  
GetRor13Hash proto  
  
.data  
szExample db "GetProcAddress",0  
  
.data?  
Hash\_GetProcAddr dd ?  
  
.code  
start:  
     mov esi,offset szExample  
     call GetRor13Hash  
     mov Hash\_GetProcAddr,eax             ;计算"GetProcAddress"的 hash 值;当用在 shellcode 中时应先计算好  
;--------------------------------------------------  
;利用 SEH 搜索 kernel32 基址，不多说了  
;--------------------------------------------------  
     assume fs:nothing  
     mov esi,fs:\[0\]  
     mov eax,esi                          ;We user "lodsd" in C  ,I don't know why. Tell me please if you know  
FindUnException:  
     cmp DWORD ptr \[eax\],0ffffffffh  
     je  FindedUnException  
     mov eax,\[eax\]  
     jmp FindUnException  
FindedUnException:  
     mov eax,\[eax+4\]  
     and eax,0ffff0000h                   ; eax---->Address of UnhandledExceptionFilter  
SearchLoop:  
     cmp WORD ptr \[eax\],'ZM'  
     jnz NotFind  
     mov ebx,\[eax+3ch\]  
     add ebx,eax  
     cmp WORD ptr \[ebx\],'EP'  
     jz FindOk  
NotFind:  
     dec eax  
     jmp SearchLoop  
FindOk:  
   mov ebp,eax                   ;eax<------------>ebp  
     mov esi,\[ebp+03ch\]  
     add esi,ebp                   ;Address of Export table------->esi  
     mov esi,\[esi+078h\]  
     add esi,ebp  
     mov ecx,\[esi+018h\]            ;Number of Functions  
     mov ebx,\[esi+020h\]              
     add ebx,ebp                   ;AddressofNames      ---------->ebx  
     push esi  
FindApi:  
     dec ecx  
     mov esi,\[ebx+ecx\*4\]  
     add esi,ebp  
     call GetRor13Hash  
     cmp Hash\_GetProcAddr,eax  
     jne FindApi  
FindedIt:  
     pop esi  
     mov ebx,\[esi+024h\]  
     add ebx,ebp                  ;AddressofOrdinals  
     mov cx, \[ebx+ecx\*2\]  
     mov ebx,\[esi+01ch\]  
     add ebx,ebp                  ;AddressOfFunctions         
     add ebp,\[ebx+ecx\*4\]   
     mov eax,ebp  
     ret  
;---------------------------------------------------  
;循环右移 13 位 HASH 算法，很常用的一种  
;esi----->要进行 HASH 的字串地址  
;---------------------------------------------------  
GetRor13Hash proc uses edx  
      xor edx,edx  
RorLoop:  
      xor eax,eax  
      lodsb  
      cmp al,0  
      jz Finish  
      ror edx,13  
      add edx,eax  
      jmp RorLoop  
Finish:  
      mov eax,edx  
      ret  
GetRor13Hash endp  
end start  
  
参考资料:罗云宾关于搜索 kernel32.dll 基址的文章(具体记不清了)  
         Anskya 的 Hash 算法 

 4dd466d0e1e0cc123afbe504da2462ca