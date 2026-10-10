

# 字串处理,asm,支持简单的表达式

> 原文：[`www.pediy.com/kssd/pediy10/74511.html`](https://www.pediy.com/kssd/pediy10/74511.html)

哦,是 fasm 不是 masm  
  

引用: i equ invoke 

引用: \_\_atoi,lpStr  
\_\_strDel,lpSrc,lpX  
\_\_strFindX,lpStr,lpX  
\_\_strlen,lpSrc  
\_\_strInsert,lpSrc,lpStr,lpX  
\_\_strReplace,lpSrc,lpStr,lpX  
\_\_strGet,lpSrc,lpRet,lpX  
Xsearch,lpStr,lpX 

支持的简单的表达式  

引用: ;        \\(123\\|321\\)  
;        \\?{}  
;        \\\*  
;        \\n  
;        \\t  
;        \\\\  
;        \\w{1}  
;        \\d{1} 

主要代码  
代码:

```
proc Xsearch; lpStr,lpX
  pushad
  mov esi,\[esp+4\*9\]
  mov edi,\[esp+4\*10\]
  call @F
  mov eax,\[esp+4\*3\]
  mov esp,\[eax+0xC4\]
  jmp Xsearch\_Err
@@:
  xor eax,eax
  push \[fs:eax\]
  mov \[fs:eax\],esp

.Start:  
  or ecx,-1
@@:  inc ecx
  mov al,byte\[edi+ecx\]
  cmp al,0
  je @F
  cmp al,'\\'
  jne @B
@@:  rep cmpsb
  jne Xsearch\_Err
  or ecx,ecx
  jne Xsearch\_Err
  cmp byte\[edi\],0
  je Xsearch\_Suc
  
  inc edi
  mov al,byte\[edi\]
  cmp al,'n'
  je .n
  cmp al,'w'
  je .w
  cmp al,'?'
  je .?
  cmp al,'t'
  je .t
  cmp al,'d'
  je .d
  cmp al,'\*'
  je .xx
  cmp al,'('
  je .c  ;\\(....\\|...\\)
  cmp al,'|'
  je Xsearch\_Suc
  cmp al,')'
  je Xsearch\_Suc
;.\\\\:
  inc edi
  lodsb
  cmp al,'\\'
  jne Xsearch\_Err
  jmp .Start
.xx:  ;.\*:
  inc edi
  dec esi
@@:  inc esi
  cmp byte\[esi\],0
  je Xsearch\_Err
  i Xsearch,esi,edi
  jnc @B
  xchg eax,esi
  jmp Xsearch\_Suc
.t:
  lodsb
  cmp al,9
  jne Xsearch\_Err
  inc edi
  jmp .Start
.n:
  lodsw
  cmp ax,0x0A0D
  jne Xsearch\_Err
  inc edi
  jmp .Start
.?:
  inc edi
  inc esi
  cmp byte\[edi\],'{'
  jne .Start
  i \_\_atoi,addr edi+1
  dec eax
  add esi,eax
@@:  inc edi
  cmp byte\[edi\],'}'
  jne @B
  inc edi
  jmp .Start
.w:
  mov ecx,1
  inc edi
  cmp byte\[edi\],'{'
  jne .w\_loop
  i \_\_atoi,addr edi+1
  xchg eax,ecx
@@:  inc edi
  cmp byte\[edi\],'}'
  jne @B
  inc edi
.w\_loop:
  lodsb
  or al,0x20
  cmp al,'a'
  jb Xsearch\_Err
  cmp al,'z'
  ja Xsearch\_Err
  loop .w\_loop
  jmp .Start
.d:
  mov ecx,1
  inc edi
  cmp byte\[edi\],'{'
  jne .d\_loop
  i \_\_atoi,addr edi+1
  xchg eax,ecx
@@:  inc edi
  cmp byte\[edi\],'}'
  jne @B
  inc edi
.d\_loop:
  lodsb
  cmp al,0
  jb Xsearch\_Err
  cmp al,9
  ja Xsearch\_Err
  loop .w\_loop
  jmp .Start
.c:
  inc edi
  i Xsearch,esi,edi
  jc .c\_end
@@:  inc edi
  cmp word\[edi\],'\\)'
  je Xsearch\_Err
  cmp word\[edi\],'\\|'
  jne @B
  inc edi
  jmp .c
.c\_end:
  xchg eax,esi
@@:  inc edi
  cmp word\[edi\],'\\)'
  jne @B
  inc edi
  inc edi
  jmp .Start
  
;-------------------------------
Xsearch\_Suc:
  mov dword\[esp+4\*9\],esi
  sub esi,\[esp+4\*11\]  
  mov dword\[esp+4\*8\],esi
  stc
  jmp Xsearch\_Ret
Xsearch\_Err:
  clc
Xsearch\_Ret:
  pop dword\[fs:0\]
  pop eax
  popad
  retn 4\*2
endp

```

全部代码在附件中.

上传的附件
| ![](img/0f24970173caf34cf302ee22e88648bc.jpg) || strlib.asm.txt  
 |