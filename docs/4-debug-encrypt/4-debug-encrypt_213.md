

# 纯汇编实现 DES64 加密算法,求邀请码.

> 原文：[`www.pediy.com/kssd/pediy12/140532.html`](https://www.pediy.com/kssd/pediy12/140532.html)

第一次发帖，申请邀请码文章，当然版主大发慈悲，给个精华鼓励新人发帖就更好了。  
  
    想学习下 DES 得算法。网上找来的。C 语言代码，按照那份代码用汇编重写了下。  
另附上一个简单的 Demo 和原来 C 语言的代码。  
  
    原来 C 语言的文件里注释的很详细所以我就没多做注释了。函数名，变量啥的我基本没变，大家对着 C 的源文件里的注释应该很容易看懂。  
  
Demo 效果 如下图：  
加密  
  
![](img/d1c6296572c9213b2b6f91a00fecc416.jpg)  
  
解密  
![](img/a9fb263930557c259ab93d005b8dc6aa.jpg)  
  
  

代码:

```

;--------------------------------------------------------------------------------
;
;File: DES.asm
;Author: Sin
;Date: 2011.9.20
;
;--------------------------------------------------------------------------------

.386
.model flat,stdcall
option casemap:none



ByteToBit proto :BYTE,:DWORD
BitToByte proto :DWORD,:DWORD
Char8ToBit64 proto :DWORD,:DWORD
Bit64ToChar8 proto :DWORD,:DWORD
Bit64ToChar64 PROTO :DWORD,:DWORD
Char64ToBit64 PROTO :DWORD,:DWORD
DES\_MakeSubKeys proto :DWORD,:DWORD
DES\_PC1\_Transform proto :DWORD,:DWORD
DES\_PC2\_Transform proto :DWORD,:DWORD
DES\_ROL proto :DWORD,:DWORD
DES\_IP\_Transform proto :DWORD
DES\_IP\_1\_Transform proto :DWORD
DES\_E\_Transform proto :DWORD
DES\_P\_Transform proto :DWORD
DES\_SBOX proto :DWORD
DES\_XOR proto :DWORD,:DWORD,:DWORD
DES\_Swap proto :DWORD,:DWORD
DES\_EncryptBlock proto :DWORD,:DWORD,:DWORD
DES\_DecryptBlock proto :DWORD,:DWORD,:DWORD
DES\_Encrypt proto :DWORD,:DWORD,:DWORD,:DWORD
DES\_Decrypt proto :DWORD,:DWORD,:DWORD,:DWORD



.data
   align 4
  ;IP\_Table\[0\] ~ IP\_Table\[63\]
  IP\_Table  dd  57,49,41,33,25,17,9,1 
        dd  59,51,43,35,27,19,11,3
        dd  61,53,45,37,29,21,13,5
        dd  63,55,47,39,31,23,15,7
        dd  56,48,40,32,24,16,8,0
        dd  58,50,42,34,26,18,10,2
        dd  60,52,44,36,28,20,12,4
        dd  62,54,46,38,30,22,14,6
      
  ;IP\_1\_Table\[0\] ~ IP\_1\_Table\[63\]      
  IP\_1\_Table  dd  39,7,47,15,55,23,63,31
        dd  38,6,46,14,54,22,62,30
        dd  37,5,45,13,53,21,61,29
        dd  36,4,44,12,52,20,60,28
        dd  35,3,43,11,51,19,59,27
        dd  34,2,42,10,50,18,58,26
        dd  33,1,41,9,49,17,57,25
        dd  32,0,40,8,48,16,56,24
  
  E\_Table    dd  31,0,1,2,3,4
        dd  3,4,5,6,7,8
        dd  7,8,9,10,11,12
        dd  11,12,13,14,15,16
        dd  15,16,17,18,19,20
        dd  19,20,21,22,23,24
        dd  23,24,25,26,27,28
        dd  27,28,29,30,31,0
        
  P\_Table    dd  15,6,19,20,28,11,27,16
        dd  0,14,22,25,4,17,30,9
        dd  1,7,23,13,31,26,2,8
        dd  18,12,29,5,21,10,3,24        

          ; S\[0\]\[0\]\[0\] ~ S\[0\]\[3\]\[15\]
  S      dd  14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7
        dd  0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8
        dd  4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0
        dd  15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13
          ;S\[1\]\[0\]\[0\] ~ S\[1\]\[3\]\[15\]
        dd  15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10
        dd  3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5
        dd  0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15
        dd  13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9
          ;
        dd  10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8
        dd  13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1
        dd  13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7
        dd  1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12
          ;
        dd  7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15
        dd  13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9
        dd  10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4
        dd  3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14
          ;
        dd  2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9
        dd  14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6
        dd  4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14
        dd  11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3
          ;
        dd  12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11
        dd  10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8
        dd  9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6
        dd  4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13
          ;
        dd  4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1
        dd  13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6
        dd  1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2
        dd  6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12
          ;
        dd  13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7
        dd  1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2
        dd  7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8
        dd  2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11
        
  PC\_1    dd  56,48,40,32,24,16,8
        dd  0,57,49,41,33,25,17
        dd  9,1,58,50,42,34,26
        dd  18,10,2,59,51,43,35
        dd  62,54,46,38,30,22,14
        dd  6,61,53,45,37,29,21
        dd  13,5,60,52,44,36,28
        dd  20,12,4,27,19,11,3
        
  PC\_2    dd  13,16,10,23,0,4,2,27
        dd  14,5,20,9,22,18,11,3
        dd  25,7,15,6,26,19,12,1
        dd  40,51,30,36,46,54,29,39
        dd  50,44,32,46,43,48,38,55
        dd  33,52,45,41,49,35,28,31
        
  MOVE\_TIMES  dd  1,1,2,2,2,2,2,2,1,2,2,2,2,2,2,1        
          
.data?

align 4

.code
memcpy proc uses esi edi ecx dst:DWORD,src:DWORD,len:DWORD

  pushfd
  cld
  mov edi,dst
  mov esi,src
  mov ecx,len
  rep movsb
  
  xor eax,eax
  popfd
  ret
memcpy endp

ByteToBit proc uses edi ecx edx cb:BYTE,bit:DWORD
  
  pushfd
  cld
  mov ecx,8
  mov edi,bit
  mov dl,cb
  xor eax,eax
  
@@:
  shr dl,1
  setc al
  stosb
  dec ecx
  jnz @b
  
  popfd
  xor eax,eax
  ret
ByteToBit endp

BitToByte proc uses esi ecx edx bit:DWORD,cb:DWORD
  
  pushfd
  cld
  xor ecx,ecx
  xor edx,edx
  xor eax,eax
  mov esi,bit
  
@@:  
  lodsb
  shl al,cl
  or dl,al
  inc cl
  cmp cl,8
  jnz @b
  
  mov eax,cb
  mov BYTE PTR \[eax\],dl
  xor eax,eax

  popfd
  ret
BitToByte endp

Char8ToBit64 proc uses esi ebx ecx edx cb:DWORD,bit:DWORD
  
  pushfd
  cld
  
  mov esi,cb
  xor eax,eax
  xor ecx,ecx
  xor edx,edx
  
@@:  
  lodsb
  mov edx,ecx
  shl edx,3
  mov ebx,bit
  add ebx,edx
  invoke ByteToBit,al,ebx
  inc ecx
  cmp ecx,8
  jnz @b
  
  xor eax,eax
  popfd  
  ret
Char8ToBit64 endp

Bit64ToChar8 proc uses esi edi ecx bit:DWORD,cb:DWORD
  
  pushfd
  cld
  
  mov edi,cb
  mov ecx,8
  xor eax,eax
  rep stosb

  mov esi,bit
  mov edi,cb
  xor eax,eax
  xor ecx,ecx
  
@@:  
  mov eax,ecx
  shl eax,3
  mov esi,bit
  add esi,eax
  mov edi,cb
  add edi,ecx
  invoke BitToByte,esi,edi
  inc ecx
  cmp ecx,8
  jnz @b
  
  xor eax,eax
  popfd    
  ret
Bit64ToChar8 endp

Bit64ToChar64 proc uses esi edi ecx lpInString,lpOutString
  
  pushfd
  cld
  mov esi,lpInString
  mov edi,lpOutString
  xor eax,eax
  mov ecx,64
@@:  
  lodsb
  add al,30H
  stosb
  dec ecx
  jnz @b
  
  popfd
  xor eax,eax
  ret
Bit64ToChar64 endp

Char64ToBit64 proc uses esi edi ecx lpInstring,lpOutString
  
  pushfd
  cld
  mov esi,lpInstring
  mov edi,lpOutString
  xor eax,eax
  mov ecx,64
@@:
  lodsb
  sub al,30H
  stosb
  dec ecx
  jnz @b
  popfd
  xor eax,eax
  ret
Char64ToBit64 endp

DES\_MakeSubKeys proc uses ecx edx ebx key:DWORD,subkeys:DWORD
  
  LOCAL temp\[56\]:BYTE
  
  invoke DES\_PC1\_Transform,key,addr temp
  xor ecx,ecx
  mov edx,offset MOVE\_TIMES
  mov ebx,subkeys

@@:  
  mov eax,\[edx+ecx\*4\]
  push eax
  lea eax,temp
  push eax
  call DES\_ROL
  mov eax,ecx
  imul eax,48
  add eax,ebx
  push eax
  push key
  call DES\_PC2\_Transform
  inc ecx
  cmp ecx,16
  jnz @b
  
  xor eax,eax  
  ret
DES\_MakeSubKeys endp

DES\_PC1\_Transform proc uses edi ebx ecx edx key:DWORD,tempbits:DWORD
  
  pushfd
  cld
  mov edi,tempbits
  mov ebx,key
  mov edx,offset PC\_1
  xor ecx,ecx

@@:
  mov eax,\[edx+ecx\*4\]
  mov al,BYTE PTR \[ebx+eax\]
  stosb
  inc ecx
  cmp ecx,56
  jnz @b
  
  xor eax,eax
  popfd
  ret
DES\_PC1\_Transform endp

DES\_PC2\_Transform proc uses edi ebx ecx edx key:DWORD,tempbits:DWORD
  
  pushfd
  cld
  mov edi,tempbits
  mov ebx,key
  mov edx,offset PC\_2
  xor ecx,ecx
  
@@:
  mov eax,\[edx+ecx\*4\]
  mov al,BYTE PTR \[ebx+eax\]
  stosb
  inc ecx
  cmp ecx,48
  jnz @b

  xor eax,eax
  popfd  
  ret
DES\_PC2\_Transform endp


DES\_ROL proc uses edx ecx ebx data:DWORD,time:DWORD
  
  LOCAL temp\[56\]:BYTE
  
  lea ebx,temp
  mov edx,data
  mov ecx,time
  
  push ecx
  push edx
  push ebx
  call memcpy
  
  push ecx
  mov eax,edx
  add eax,28
  push eax
  mov eax,ebx
  add eax,ecx
  push eax
  call memcpy
  
  mov eax,28
  sub eax,ecx
  push eax
  mov eax,edx
  add eax,ecx
  push eax
  push edx
  call memcpy
  
  push ecx
  push ebx
  mov eax,edx
  add eax,28
  sub eax,ecx
  push eax
  call memcpy
  
  mov eax,28
  sub eax,ecx
  push eax
  mov eax,edx
  add eax,28
  add eax,ecx
  push eax
  mov eax,edx
  add eax,28
  push eax
  call memcpy
  
  push ecx
  mov eax,ebx
  add eax,ecx
  push eax
  mov eax,edx
  add eax,56
  sub eax,ecx
  push eax
  call memcpy
  
  xor eax,eax
  ret
DES\_ROL endp

DES\_IP\_Transform proc uses edi ebx ecx edx data:DWORD
  
  LOCAL temp\[64\]:BYTE
  
  pushfd
  cld
  
  lea edi,temp
  mov edx,offset IP\_Table
  mov ebx,data
  xor ecx,ecx
  
@@:
  mov eax,\[edx+ecx\*4\]
  mov al,BYTE PTR \[ebx+eax\]
  stosb
  inc ecx
  cmp ecx,64
  jnz @b
  
  invoke memcpy,data,addr temp,64
  xor eax,eax
  popfd
  ret
DES\_IP\_Transform endp

DES\_IP\_1\_Transform proc uses edi ebx ecx edx data:DWORD
  
  LOCAL temp\[64\]:BYTE
  
  pushfd
  cld
  
  lea edi,temp
  mov edx,offset IP\_1\_Table
  mov ebx,data
  xor ecx,ecx
  
@@:
  mov eax,\[edx+ecx\*4\]
  mov al,BYTE PTR \[ebx+eax\]
  stosb
  inc ecx
  cmp ecx,64
  jnz @b  
  
  invoke memcpy,data,addr temp,64
  xor eax,eax  
  popfd
  ret
DES\_IP\_1\_Transform endp

DES\_E\_Transform proc uses edi ebx ecx edx data:DWORD
  
  LOCAL temp\[48\]:BYTE
  
  pushfd
  cld
  lea edi,temp
  mov edx,offset E\_Table
  mov ebx,data
  xor ecx,ecx

@@:
  mov eax,\[edx+ecx\*4\]
  mov al,BYTE PTR \[ebx+eax\]  
  stosb
  inc ecx
  cmp ecx,48
  jnz @b

  invoke memcpy,data,addr temp,48
  xor eax,eax
  popfd  
  ret
DES\_E\_Transform endp

DES\_P\_Transform proc uses edi ebx ecx edx data:DWORD
  
  LOCAL temp\[32\]:BYTE
  
  pushfd
  cld
  lea edi,temp
  mov edx,offset P\_Table
  mov ebx,data
  xor ecx,ecx
  
@@:
  mov eax,\[edx+ecx\*4\]
  mov al,BYTE PTR \[ebx+eax\]
  stosb
  inc ecx
  cmp ecx,32
  jnz @b

  invoke memcpy,data,addr temp,32
  xor eax,eax
  popfd
  ret
DES\_P\_Transform endp

DES\_XOR proc uses esi edi ecx edx R:DWORD,L:DWORD,count:DWORD
  
  mov edi,R
  mov esi,L
  
  xor eax,eax
  xor edx,edx
  xor ecx,ecx

@@:  
  cmp ecx,count
  jz @Ret
  mov al,BYTE PTR \[esi+ecx\]
  mov dl,BYTE PTR \[edi+ecx\]
  xor al,dl
  inc ecx
  jmp @b
@Ret:
  xor eax,eax  
  ret
DES\_XOR endp

DES\_SBOX proc uses esi edi ebx ecx edx data:DWORD
  
  LOCAL \_line:DWORD,row:DWORD,output:DWORD
;  LOCAL cur1:DWORD,cur2:DWORD 
;  use cur1-> esi cur2->edi
  
  mov ebx,data
  xor ecx,ecx
  
@@:
  mov esi,ecx
  imul esi,6
  mov edi,ecx
  shl edi,2
  
  mov al,BYTE PTR \[ebx+esi\]
  shl al,1
  movzx edx,al
  
  mov al,BYTE PTR \[ebx+esi+5\]
  movzx eax,al
  add eax,edx
  mov \_line,eax
  
  mov al,BYTE PTR \[ebx+esi+1\]
  shl al,3
  movzx edx,al
  mov al,BYTE PTR \[ebx+esi+2\]
  shl al,2
  movzx eax,al
  add edx,eax
  mov al,BYTE PTR \[ebx+esi+3\]
  shl al,1
  movzx eax,al
  add edx,eax
  mov al,BYTE PTR \[ebx+esi+4\]
  movzx eax,al
  add eax,edx
  mov row,eax
  
  mov edx,offset S
  mov eax,\_line
  imul eax,row
  imul eax,ecx
  mov eax,DWORD PTR \[edx+eax\]
  mov output,eax
  
  and eax,08H
  shr eax,3
  mov BYTE PTR \[ebx+edi\],al
  mov eax,output
  and eax,04H
  shr eax,2
  mov BYTE PTR \[ebx+edi+1\],al
  mov eax,output
  and eax,02H
  shr eax,1
  mov BYTE PTR \[ebx+edi+2\],al
  mov eax,output
  and eax,01H
  mov BYTE PTR \[ebx+edi+3\],al
  inc ecx
  cmp ecx,8
  jnz @b
  
  xor eax,eax
  ret
DES\_SBOX endp

DES\_Swap proc left:DWORD,right:DWORD
  
  LOCAL temp\[32\]:BYTE
  
  invoke memcpy,addr temp,left,32
  invoke memcpy,left,right,32
  invoke memcpy,right,addr temp,32
  
  xor eax,eax
  ret
DES\_Swap endp

DES\_EncryptBlock proc uses ecx edx plainBlock:DWORD,subKeys:DWORD,cipherBlock:DWORD
  
  LOCAL plainBits\[64\]:BYTE
  LOCAL copyRight\[48\]:BYTE
  

  invoke Char8ToBit64,plainBlock,addr plainBits
  invoke DES\_IP\_Transform,addr plainBits
  
  xor ecx,ecx

@loop:
  push 32
  lea eax,plainBits
  add eax,32
  push eax
  lea eax,copyRight
  push eax
  call memcpy

  invoke DES\_E\_Transform,addr copyRight
  
  mov eax,subKeys
  mov edx,ecx
  imul edx,48
  add eax,edx
  invoke DES\_XOR,addr copyRight,eax,48
  invoke DES\_SBOX,addr copyRight
  invoke DES\_P\_Transform,addr copyRight
  invoke DES\_XOR,addr plainBits,addr copyRight,32
  
  cmp ecx,15
  jz @f
  lea eax,plainBits
  add eax,32
  invoke DES\_Swap,addr plainBits,eax
@@:  
  inc ecx
  cmp ecx,16
  jnz @loop
  
  invoke DES\_IP\_1\_Transform,addr plainBits
  invoke Bit64ToChar8,addr plainBits,cipherBlock
  xor eax,eax
  ret

DES\_EncryptBlock endp

DES\_DecryptBlock proc uses edx ecx cipherBlock:DWORD,subKeys:DWORD,plainBlock:DWORD

  LOCAL cipherBits\[64\]:BYTE
  LOCAL copyRight\[48\]:BYTE
  
  
  invoke Char8ToBit64,cipherBlock,addr cipherBits
  invoke DES\_IP\_Transform,addr cipherBits
  
  mov ecx,15

@loop:  
  push 32
  lea eax,cipherBits
  add eax,32
  push eax
  lea eax,copyRight
  push eax
  call memcpy
  invoke DES\_E\_Transform,addr copyRight
  
  mov edx,subKeys
  mov eax,ecx
  imul eax,48
  add eax,edx
  invoke DES\_XOR,addr copyRight,eax,48
  invoke DES\_SBOX,addr copyRight
  invoke DES\_P\_Transform,addr copyRight
  invoke DES\_XOR,addr cipherBits,addr copyRight,32
  
  test ecx,ecx
  jz @f
  lea eax,cipherBits
  add eax,32
  invoke DES\_Swap,addr cipherBits,eax
@@:
  dec ecx
  cmp ecx,0
  jge @loop
  
  invoke DES\_IP\_1\_Transform,addr cipherBits
  invoke Bit64ToChar8,addr cipherBits,plainBlock
  xor eax,eax  
  ret

DES\_DecryptBlock endp

DES\_Encrypt proc uses esi edi edx ecx lpInString,cbInString,lpKeyString,lpOutString
  
  local szSubKeys\[768\]:BYTE ;16\*48
  local bKey\[64\]:BYTE
  local szTemp\[8\]:BYTE
  local szResult\[8\]:BYTE
  
  invoke Char8ToBit64,lpKeyString,addr bKey
  invoke DES\_MakeSubKeys,addr bKey,addr szSubKeys
  
  mov ecx,cbInString
  mov esi,lpInString
  mov edi,lpOutString
  xor edx,edx
  
@loop:    
  cmp ecx,8
  jb @f
  invoke memcpy,addr szTemp,esi,8
  add esi,8
  invoke DES\_EncryptBlock,addr szTemp,addr szSubKeys,addr szResult
  jmp @copy
@@:
  invoke memcpy,addr szTemp,esi,ecx
  pushfd
  cld
  push edi
  lea edi,szTemp
  add edi,ecx
  mov eax,7
  sub eax,ecx
  mov BYTE PTR \[edi+7\],al
  mov ecx,eax
  xor eax,eax
  rep stosb
  pop edi
  popfd
  invoke DES\_EncryptBlock,addr szTemp,addr szSubKeys,addr szResult
@copy:
  invoke memcpy,edi,addr szResult,8
  add edi,8
  add edx,8
  sub ecx,8
  jg @loop

  mov eax,edx
  ret
DES\_Encrypt endp

DES\_Decrypt proc uses esi edi ebx ecx edx lpInString,cbInString,lpKeyString,lpOutString
  
  local szSubKeys\[768\]:BYTE
  local bKey\[64\]:BYTE
  local szResult\[8\]:BYTE

  invoke Char8ToBit64,lpKeyString,addr bKey
  invoke DES\_MakeSubKeys,addr bKey,addr szSubKeys  
  
  mov ecx,cbInString
  mov esi,lpInString
  mov edi,lpOutString
  
  xor ebx,ebx
  
@@:
  cmp ecx,0
  jbe @Ret
  invoke DES\_DecryptBlock,esi,addr szSubKeys,addr szResult 
  add esi,8
  invoke memcpy,edi,addr szResult,8
  add edi,8
  add ebx,8
  sub ecx,8
  test ecx,ecx
  jz @loop
  jmp @b
  
@loop:
  pushfd
  cld
  push esi
  lea esi,szResult
  xor eax,eax
  mov al,BYTE PTR \[esi+7\]
  cmp al,8
  jae @Ret ;>=8 跳出
  movzx eax,al
  mov ecx,8
  sub ecx,eax ;
  mov edx,ecx
@@:  
  lodsb
  test al,al
  jnz @f ;有不为 0 的跳出
  inc ecx
  cmp ecx,7
  jb @b
  push edx
  lea eax,szResult
  push eax
  sub edi,8
  push edi
  call memcpy
  add ebx,edx
  sub ebx,8
@@:  
  pop esi
  popfd
@Ret:  
  mov eax,ebx
  ret
DES\_Decrypt endp

end

```

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || Demo.rar  
 |

。刚才发现了个错误 在最后一个函数 注释 有不为 0 跳出 那个地方。直接跳到@Ret 。  
前面有   
pushfd  
push esi  
这样没 出栈 栈 就不平衡了。  
刚修改下。。附件不知道怎么删除。我重新上传个。

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || Demo.rar  
 |