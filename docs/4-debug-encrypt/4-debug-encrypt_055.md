
# 许源华程序之 MD5

> 原文：[`www.pediy.com/kssd/pediy11/111499.html`](https://www.pediy.com/kssd/pediy11/111499.html)

现在要找工作。只好嗦下：

我熟悉`WINDOWS API` `X86 32 位指令` `X87 FPU 编程`.

还有比较牛的特点：一天能写 12 个小时代码的。手都不累。。（这点一般的新人是不行。。呵呵）

有意思谈下 加 `QQ 522165473`

这个写了二天才写好。汗。。

(算法是别人的描述的，在 [http://en.wikipedia.org/wiki/MD5](http://en.wikipedia.org/wiki/MD5)。我只是翻译下）

```
.386

.model flat,stdcall

option casemap:none

include ..\masm32\include\windows.inc

include ..\masm32\include\kernel32.inc

include ..\masm32\include\user32.inc

include ..\masm32\include\gdi32.inc

include ..\masm32\include\comdlg32.inc

includelib ..\masm32\lib\comdlg32.lib

includelib ..\masm32\lib\kernel32.lib

includelib ..\masm32\lib\gdi32.lib

includelib ..\masm32\lib\user32.lib

.data

ofn OPENFILENAME <>

hfile dd 0

fsize1 dd 0

fsize dd 0

filter db 'all','*.*',0,0

file db 4096 dup (0)

title1 db 'MD5',0

byte1 dd 0

time1 dd 0

hmem dd 0

byte2  dd 0

w dd 80h,15 dup (0)

time2 dd 0

show1 dd 10 dup (0)

x1 dd 067452301h

x2 dd 0EFCDAB89h

x3 dd 098BADCFEh

x4 dd 010325476h

a1 dd 0

b1 dd 0

c1 dd 0

d1 dd 0

i  dd 0

f dd 0

g dd 0

r dd 7,12,17,22,7,12,17,22,7,12,17,22,7,12,17,22

  dd 5,9,14,20,5,9,14,20,5,9,14,20,5,9,14,20

  dd 4,11,16,23,4,11,16,23, 4,11,16,23, 4,11,16,23  

  dd 6,10,15,21,6,10,15,21,6,10,15,21,6,10,15,21

k dd 64 dup (0)

temp1 dd 1,2,3,4

temp2 dd 0.5

temp3 dd 180

x dd 0

temp4 dq 1.000

 ;;D76A A478

temp5 dq 0.0

;;;2^(11-1)-1=1023

 ktemp dq 0.0

.const

 cmembyte equ 65536

error1 db 'error in set file pointer!',0

.code

start:

 ；；；；；；；SIN(1)-SIN(60)注意思位限制时的处理

 lea ebx,k

 mov x,1

@@:

 cmp x,65

jz @f

 fild x

 fsin

 fabs

 fstp ktemp

 lea edx,ktemp

 mov edi,\[edx\]

 mov esi,\[edx+4\]

 mov eax,esi

 shl eax,1

 shr eax,21

 sub eax,1023

 sub eax,20

 add eax,32

 mov ecx,eax

 and esi,0fffffh

 or  esi,100000h

 shld esi,edi,cl

 mov \[ebx\],esi

 inc x

 add ebx,4

 jmp @b

@@:

 invoke RtlZeroMemory,offset ofn,sizeof OPENFILENAME

 mov ofn.lStructSize,sizeof OPENFILENAME

 mov ofn.Flags,OFN\_FILEMUSTEXIST or OFN\_PATHMUSTEXIST

 mov ofn.lpstrFilter,offset filter

 mov ofn.lpstrFile,offset file

 mov ofn.nMaxFile,4096

 mov ofn.lpstrTitle,offset title1

 invoke GetOpenFileName,offset ofn

 cmp eax,0

 jnz @f

 invoke ExitProcess,0

@@:

 invoke  CreateFile,offset file,GENERIC\_WRITE or GENERIC\_READ,0,0,OPEN\_EXISTING,FILE\_ATTRIBUTE\_SYSTEM,0

 cmp eax, INVALID\_HANDLE\_VALUE

 jnz @f

 invoke ExitProcess,0

@@:

 mov hfile,eax

 invoke SetFilePointer,hfile,0,0,FILE\_BEGIN

 cmp eax,INVALID\_SET\_FILE\_POINTER

 jnz @f

 invoke ExitProcess,0

@@:

 invoke GlobalAlloc,GMEM\_FIXED OR GMEM\_ZEROINIT,cmembyte

 cmp eax,0

 jnz @f

 invoke ExitProcess,0

@@:

 mov hmem,eax

 mov w,eax

 invoke GetFileSize,hfile,NULL

 mov fsize,eax  

 mov fsize1,eax

l12:

 sub fsize,cmembyte

 js l1

 invoke ReadFile,hfile,hmem,cmembyte,offset byte1,0

 cmp eax,0

 jnz @f

 invoke ExitProcess,0

@@:

 mov time2,1024

 call pmd5

 jmp l12

l1: 

 add fsize,cmembyte

 mov eax,fsize

 mov time2,eax

 shr time2,6

 and fsize,3fh

 sub eax,fsize

 invoke ReadFile,hfile,hmem,eax,offset byte1,0

 cmp eax,0

 jnz @f

 invoke ExitProcess,0

@@:

 call pmd5

 invoke ReadFile,hfile,hmem,fsize,offset byte1,0

 cmp eax,0

 jnz @f

 invoke ExitProcess,0

@@:

 mov edi,hmem

 add edi,fsize

 mov byte ptr \[edi\],80h

 inc edi

 mov eax,fsize

 add eax,9

 mov ebx,64

 mov time2,1

 cmp eax,65

 js @f

 add ebx,64

 mov time2,2

@@: 

 sub ebx,eax

 inc ebx

@@:

 sub ebx,1

 jz @f

 mov byte ptr \[edi\],0

 inc edi

 jmp @b

@@:

 mov eax,fsize1

 cdq

 mov ebx,8

 mul ebx

 mov \[edi\],eax

 mov dword ptr \[edi+4\],edx

 call pmd5

lshow: 

 lea edi,show1

 lea esi,x1

 mov ebx,0

lok1:

 cmp ebx,16

jz lok3

 mov al,\[esi\]

 inc esi

 mov ah,al

 shr ah,4

 add ah,30h

 cmp ah,3ah

 js @f

 add ah,7

@@:

 mov \[edi\],ah

 inc edi

 and al,0fh

 add al,30h

 cmp al,3ah

 js @f

 add al,7

@@:

 mov \[edi\],al

 inc edi

 inc ebx

 jmp lok1

lok3:

 invoke MessageBox,0,offset show1,OFFSET title1,MB\_OK

 invoke ExitProcess,NULL 

 invoke ExitProcess,0

pmd5:

 mov eax,hmem

 mov w,eax

 sub w,64

 inc time2

start1:

 sub time2,1

 jnz @f

 ret

@@:

 add w,64

 mov eax,x1

 mov a1,eax

 mov eax,x2

 mov b1,eax

 mov eax,x3

 mov c1,eax

 mov eax,x4

 mov d1,eax

 mov i,0

l1start:

 cmp i,64

 jz ll1

 mov eax,i

 shr eax,4

 cmp eax,0

 jnz @f

 mov eax,d1

 xor eax,c1

 and eax,b1

 xor eax,d1

 mov f,eax

 mov eax,i

 mov g,eax

 jmp ll2

@@:

 cmp eax,1

 jnz @f

 mov eax,c1

 xor eax,b1

 and eax,d1

 xor eax,c1

 mov f,eax

 mov eax,5

 cdq

 mul i

 inc eax

 and eax,0fh

 mov g,eax

 jmp ll2

@@:

 cmp eax,2

 jnz @f

 mov eax,b1

 xor eax,c1

 xor eax,d1

 mov f,eax

 mov eax,3

 cdq

 mul i

 add eax,5

 and eax,0fh

 mov g,eax

 jmp ll2

@@:

 cmp eax,3

 jnz @f

 mov eax,d1

 not eax

 or eax,b1

 xor eax,c1

 mov f,eax

 mov eax,7

 cdq

 mul i

 and eax,0fh

 mov g,eax

@@:

ll2:

 mov eax,d1

 mov temp1,eax

 mov eax,c1

 mov d1,eax

 mov eax,b1

 mov c1,eax

 mov edi,w

 mov esi,g

 mov eax,\[edi+esi\*4\]

 lea edi,k

 mov esi,i

 mov ebx,\[edi+esi\*4\]

 add eax,ebx

 add eax,f

 add eax,a1

 lea edi,r

 mov esi,i

 mov ecx,\[edi+esi\*4\]

 rol eax,cl

 add b1,eax

 mov eax,temp1

 mov a1,eax

 inc i

 jmp l1start

ll1: 

 mov eax,x1

 add eax,a1

 mov x1,eax

 mov eax,x2

 add eax,b1

 mov x2,eax

 mov eax,x3

 add eax,c1

 mov x3,eax

 mov eax,x4

 add eax,d1

 mov x4,eax

 jmp start1

 ret

end start
```

下面是程序：放心使用。。有问题直 `CALL QQ ：522165473`

## 上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `MD5.rar` |
|---|---|
