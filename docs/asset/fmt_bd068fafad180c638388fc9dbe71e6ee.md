
```
  mov   DWORD ptr [szRlt+8],edx  
  mov   eax,DWORD ptr [szRlt+0ch]  
  mov   edx,eax  
  and   edx,0f0h  
  shr   edx,4  
  and   eax,0fh  
  shl   eax,4  
  add   edx,eax  
  mov   DWORD ptr [szRlt+0ch],edx  
  mov   ebx,DWORD ptr [szRlt+4]  
  shl   ebx,18h  
  mov   eax,DWORD ptr [szRlt]  
  shl   eax,10h  
  add   ebx,eax  
  shl   edx,8  
  add   ebx,edx  
  add   ebx,DWORD ptr [szRlt+8]  
  mov   eax,ebx  
  mov  nVolNum,eax  
  invoke  dw2str,nVolNum,addr szRegNum  
  lea  eax,szRegNum  
  movzx  eax,BYTE ptr [eax]  
  lea  edx,szRegNum  
  movzx  edx,BYTE ptr [edx+1]  
  add  eax,edx  
  mov  ecx,5  
  cdq  
  idiv  ecx  
  add  dl,61h  
  mov  BYTE ptr [szChar],dl  
  lea  eax,szRegNum  
  movzx  eax,BYTE ptr [eax+2]  
  lea  edx,szRegNum  
  movzx  edx,BYTE ptr [edx+3]  
  add  eax,edx  
  mov  ecx,5  
  cdq  
  idiv  ecx  
  add  dl,61h  
  mov  BYTE ptr [szChar+1],dl  
  lea  eax,szRegNum  
  movzx  eax,BYTE ptr [eax+4]  
  lea  edx,szRegNum  
  movzx  edx,BYTE ptr [edx+5]  
  add  eax,edx  
  mov  ecx,5  
  cdq  
  idiv  ecx  
  add  dl,61h  
  mov  BYTE ptr [szChar+2],dl  
  mov  al,BYTE ptr [szChar]  
  mov  Char,al  
  invoke  InsertChar,addr szRegNum,addr Char,1  
  mov  al,BYTE ptr [szChar+1]  
  mov  Char,al  
  invoke  InsertChar,addr szRegNum,addr Char,5  
  mov  al,BYTE ptr [szChar+2]  
  mov  Char,al  
  invoke  InsertChar,addr szRegNum,addr Char,9  
  invoke  SetDlgItemText,hDlg,IDC_REG,addr szRegNum  
  popad  
  ret  
GetRegKey  endp  
```

【版权声明】本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
