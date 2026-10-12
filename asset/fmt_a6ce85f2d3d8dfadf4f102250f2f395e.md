
```
SDPI:0047B78F jmp MessageBoxA ; MessageBoxA 函数，壳用的宏来处理，  

SDPI:0047B78F ; 判断函数的前 5 位是否为 CC  

SDPI:0047B78F ; 也就是判断有没有下 int3 断点  

SDPI:0047B78F ; ----------------------------------------------------------------------------  

SDPI:0047B794 db 90h ; ?  

SDPI:0047B795 db 90h ; ?  

SDPI:0047B796 db 90h ; ?  

SDPI:0047B797 db 90h ; ?  

SDPI:0047B798 ; ----------------------------------------------------------------------------  

SDPI:0047B798 push 7  

SDPI:0047B79A call loc_47B7A0  

SDPI:0047B79F nop  

SDPI:0047B7A0  

SDPI:0047B7A0 loc_47B7A0: ; CODE XREF: FNDDBG_47B749+51 p  

SDPI:0047B7A0 pop eax  

SDPI:0047B7A1 add eax, 11h  

SDPI:0047B7A6 push eax  

SDPI:0047B7A7 jmp ExitProcess  

SDPI:0047B7A7 ; ----------------------------------------------------------------------------  

SDPI:0047B7AC db 90h ; ?  

SDPI:0047B7AD db 90h ; ?  

SDPI:0047B7AE db 90h ; ?  

SDPI:0047B7AF db 90h ; ?  

SDPI:0047B7AF FNDDBG_47B749 endp  

SDPI:0047B7AF  

SDPI:0047B7B0 ; ----------------------------------------------------------------------------  

SDPI:0047B7B0  

SDPI:0047B7B0 Local_47B7B0: ; CODE XREF: SDPI:0047B72F p  

SDPI:0047B7B0 nop  

SDPI:0047B7B1 nop  

SDPI:0047B7B2 nop  

SDPI:0047B7B3 nop  

SDPI:0047B7B4 nop  

SDPI:0047B7B5 call loc_47B7BB  

SDPI:0047B7BA nop  

SDPI:0047B7BB  

SDPI:0047B7BB loc_47B7BB: ; CODE XREF: SDPI:0047B7B5 p  

SDPI:0047B7BB pop eax  

SDPI:0047B7BC add eax, 5Eh  

SDPI:0047B7C1 mov edx, eax  

SDPI:0047B7C3 add edx, 32h  

SDPI:0047B7C6 call Crypt_Decrypt_CODE ; 传入 edx(结束地址)传入 EAX(起始地址)  

SDPI:0047B7C6 ; 用于加密或解密代码,加解密操作是互换的  

SDPI:0047B7C6 ; 是只简单的 xor,因此可以用来加密也可以  

SDPI:0047B7C6 ; 用来解密  

SDPI:0047B7CB call loc_47B7D1  

SDPI:0047B7D0 nop  

SDPI:0047B7D1  

SDPI:0047B7D1 loc_47B7D1: ; CODE XREF: SDPI:0047B7CB p  

SDPI:0047B7D1 pop eax  

SDPI:0047B7D2 add eax, 38CCh  

SDPI:0047B7D7 call loc_47B7DD  

SDPI:0047B7DC nop  

SDPI:0047B7DD  

SDPI:0047B7DD loc_47B7DD: ; CODE XREF: SDPI:0047B7D7 p  

SDPI:0047B7DD pop ecx  

SDPI:0047B7DE add ecx, 396Dh  

SDPI:0047B7E4 push 0  

SDPI:0047B7E6 push ecx  

SDPI:0047B7E7 push eax  

SDPI:0047B7E8 push 0  

SDPI:0047B7EA call loc_47B7F0  

SDPI:0047B7EF nop  

SDPI:0047B7F0  

SDPI:0047B7F0 loc_47B7F0: ; CODE XREF: SDPI:0047B7EA p  

SDPI:0047B7F0 pop eax  

SDPI:0047B7F1 add eax, 11h  

SDPI:0047B7F6 push eax  

SDPI:0047B7F7 jmp MessageBoxA ; MessageBoxA 函数，壳用的宏来处理，  

SDPI:0047B7F7 ; 判断函数的前 5 位是否为 CC  

SDPI:0047B7F7 ; 也就是判断有没有下 int3 断点  

SDPI:0047B7F7 ; ----------------------------------------------------------------------------  

SDPI:0047B7FC dd 90909090h  

SDPI:0047B800 ; ----------------------------------------------------------------------------  

SDPI:0047B800 push 7  

SDPI:0047B802 call loc_47B808  

SDPI:0047B807 nop  

SDPI:0047B808  

SDPI:0047B808 loc_47B808: ; CODE XREF: SDPI:0047B802 p  

SDPI:0047B808 pop eax  

SDPI:0047B809 add eax, 11h  

SDPI:0047B80E push eax  

SDPI:0047B80F jmp ExitProcess  

SDPI:0047B80F ; ----------------------------------------------------------------------------  

SDPI:0047B814 dd 90909090h  

SDPI:0047B818 dd 401000h  

SDPI:0047B81C dd 69C89E0h  

SDPI:0047B820 ; ----------------------------------------------------------------------------  

SDPI:0047B820  

SDPI:0047B820 loc_47B820: ; CODE XREF: SDPI:0047B734 p  

SDPI:0047B820 call loc_47B826  

SDPI:0047B825 nop  

SDPI:0047B826  

SDPI:0047B826 loc_47B826: ; CODE XREF: SDPI:loc_47B820 p  

SDPI:0047B826 pop edi  

SDPI:0047B827 add edi, 0FFFFFF07h  

SDPI:0047B82D mov [edi], ebx  

SDPI:0047B82F mov [edi+4], edx  

SDPI:0047B832 pop eax  

SDPI:0047B833 call loc_47B839  

SDPI:0047B838 nop  

SDPI:0047B839  

SDPI:0047B839 loc_47B839: ; CODE XREF: SDPI:0047B833 p  

SDPI:0047B839 pop eax  

SDPI:0047B83A add eax, 124h  

SDPI:0047B83F push eax  

SDPI:0047B840 xor eax, eax  

SDPI:0047B842 push dword ptr fs:[eax]  

SDPI:0047B845 mov fs:[eax], esp  

SDPI:0047B848 mov ebp, 300EF1D3h  

SDPI:0047B84D add ebp, 12345678h  

SDPI:0047B853 mov ax, 17h  

SDPI:0047B857 sub ax, 13h  

SDPI:0047B857 ; ----------------------------------------------------------------------------  

SDPI:0047B85B aI@stRS@sRS@sfP3TS@s db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?  

SDPI:0047B85B db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0  

SDPI:0047B85B db '鑈 Xf?  

SDPI:0047B891 dd 90909090h  

SDPI:0047B895 ; ----------------------------------------------------------------------------  

SDPI:0047B895 nop  

SDPI:0047B896 int 3 ; Trap to Debugger  

SDPI:0047B897 nop ; SEH 句柄:0047B95C  

SDPI:0047B898 cmp al, 4  

SDPI:0047B89A jz short Pass_47B90D  

SDPI:0047B89C  

SDPI:0047B89C ; ****** S U B R O U T I N E ******  

SDPI:0047B89C  

SDPI:0047B89C  

SDPI:0047B89C fnddbg_47B89C proc near ; CODE XREF: SDPI:0047B926 j  

SDPI:0047B89C ; SDPI:0047B93E j ...  

SDPI:0047B89C nop  

SDPI:0047B89D nop  

SDPI:0047B89E nop  

SDPI:0047B89F nop  

SDPI:0047B8A0 nop  

SDPI:0047B8A1 call loc_47B8A7  

SDPI:0047B8A6 nop  

SDPI:0047B8A7  

SDPI:0047B8A7 loc_47B8A7: ; CODE XREF: fnddbg_47B89C+5 p  

SDPI:0047B8A7 pop eax  

SDPI:0047B8A8 add eax, 5Eh  

SDPI:0047B8AD mov edx, eax
```
