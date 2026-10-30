
```asm
;--------
cld
xor bx,bx
xor dx,dx
mov ds,\_psp
mov si,80h
xor ah,ah
lodsb
inc ax
mov cx,ax                        ;CMD\_LINE Length (include \r);
push cs
pop es
lea di,CMD\_LINE                  ;BUFF
xor ax,ax
;-----------------------------------------------------------------
Processing label near
call NextChar
ja NotQuote
;------------------------------------
InString label near
jb BuildArgv
call NextChar
ja InString
;------------------------------------
NotQuote label near
cmp al,' '
je EndArgument
cmp al,13
je EndArgument
cmp al,9
jne Processing
;-----------------------------------
EndArgument label near
xor al,al
;------------------------------
push bx
;   push dx
;   dec dx
shl bx,1
mov word ptr cs:Argv2[bx],dx
;   pop dx
pop bx
;-------------------------------
jmp short Processing
NextChar proc near
or ax,ax
jz NextChar0
inc dx
stosb
or al,al
jnz NextChar0
inc bx
NextChar0 label near
xchg ah,al
xor al,al
stc
jcxz NextChar2
lodusb
dec cx
sub al,'"'
jz NextChar2
add al,'"'
cmp al,'\'
jnz NextChar1
cmp byte ptr ds:[si],'"'
jne NextChar1
lodusb
dec cx
NextChar1 label near
or si,si
NextChar2 label near
ret
NextChar endp
;-------------------------------------------------
BuildArgv label near
push cs
pop ds
push cs
pop es
;   mov ds:\_Argc,bx
push bx
mov cx,bx
;---added 95.6.29
jcxz Test\_argu
;-----
mov dx,cx
lea bx,CMD\_LINE
lea si,Argv1
mov di,si
@@1 label near
lodsw
add ax,bx
cmp dx,cx
jz @1st
inc ax
@1st label near
stosw
loop @@1
;------------------------------------
pop bx
cmp bx,3
jb  Test\_argu
;lea si,Argv3
mov si,cs:argv3
lodusb
;------------------
cmp al,'T'      ; to try !
jz  Forced            ; I want make it can unlock some unknowe version bitlok
cmp al,'t'
jz Forced       ;add 1995,9,20
;------------------
cmp al,'0'
jb Continue
cmp al,'K'
ja Continue
cmp al,'9'
jbe Ten
cmp al,'A'
jb Continue
Hex:
sub al,'A'-0ah
jmp Set\_Scheme
Ten:
sub al,'0'
Set\_Scheme:
mov byte ptr ds:SchemeSign,al
jmp short Continue
Test\_argu label near
cmp bx,2
jnb Continue
;---------------------------
lea dx,Useage\_Msg
Quit label near
mov ah,9
int 21h
mov ah,4ch
int 21h
;------------------------------------------------------
;------------------------------------------------------
Forced:
lea dx,Forced\_MSG            ;add 19950920
mov ah,9
int 21h
mov byte ptr cs:Force,1
;---------------------------------
Continue label near
;--------added 950629
lea dx,Processing\_MSG
mov ah,9
int 21h
;--------
mov dx,ds:Argv1
mov ax,3d00h
int 21h
jnb @@2
lea dx,Err1\_MSG
jmp Quit
@@2 label near
mov ds:handle1,ax
mov dx,ds:Argv2
mov ax,3c00h
int 21h
jnb @@3
lea dx,Err2\_MSG
jmp Quit
@@3 label near
mov ds:handle2,ax
lea dx,Buffer
mov bx,ds:handle1
mov ah,3fh
mov cx,20h
int 21h
jnb @@4
lea dx,Err3\_MSG
jmp Quit
@@4 label near
mov dx,word ptr ds:Buffer+8h
mov cl,4
shl dx,cl
mov ds:Head\_Size,dx
;push dx
xor cx,cx
mov ax,4200h
int 21h
mov ah,3fh
mov cx,110h
lea dx,buffer
int 21h
jnb @@5
lea dx,Err3\_MSG
jmp Quit
@@5 label near
cmp word ptr ds:Buffer[@Mark],Mark0
jz @@6
cmp word ptr ds:Buffer[@Mark],Mark1
jz @@6
lea dx,Err4\_MSG
jmp Quit
@@6 label near
mov ax,word ptr ds:Buffer[@Offset3\_L]
mov word ptr ds:Offset3\_L,ax
mov ax,word ptr ds:Buffer[@Offset3\_H]
mov ds:Offset3\_H,ax
mov ax,word ptr ds:Buffer[@Locker\_Size\_L]
mov ds:Locker\_Size\_L,ax
mov ax,word ptr ds:Buffer[@Locker\_Size\_H]
mov word ptr ds:Locker\_Size\_H,ax
mov ax,word ptr ds:Buffer[@Code\_Size\_L]
mov ds:Code\_Size\_L,ax
mov ax,word ptr ds:Buffer[@Code\_Size\_H]
mov ds:Code\_size\_H,ax
mov ax,word ptr ds:Buffer[@Relocations]
mov ds:Relocations,ax
mov ax,word ptr ds:Buffer[@Relocation\_Table\_Entry]
mov ds:Relocation\_Table\_Entry,ax
mov ax,word ptr ds:Buffer[@Ss0]
mov ds:Ss0,ax
mov ax,word ptr ds:Buffer[@Sp0]
mov ds:Sp0,ax
mov ax,word ptr ds:Buffer[@Cs0]
mov ds:Cs0,ax
mov ax,word ptr ds:Buffer[@Ip0]
mov ds:Ip0,ax
mov ax,word ptr ds:Buffer[@Key1]
mov ds:Key1,ax
mov ax,word ptr ds:Buffer[@Key2]
mov ds:Key2,ax
mov al,byte ptr ds:Buffer[@Scheme]
mov ds:Scheme,al
cmp byte ptr ds:SchemeSign,0ffh       ;Check if defined scheme
jnz @@A
;-----------------------------------------------------------------------
cmp byte ptr ds:Force,1
jz @@A
;-----------------------------------------------------------------------
cmp ds:Locker\_Size\_H,0                 ;Check if the defined version of bitlok
jz @@7                                 ;if not show massage and quit
@Err5:                                 ;else set mode and schemes
lea dx,Err5\_MSG                        ;
jmp Quit                               ;
@@7 label near                             ;
mov cx,ds:Versions                     ;
lea si,BITLOK12                        ;
@@8 label near                             ;
lodsw                                  ;
cmp ds:Locker\_Size\_L,ax                ;
jz @@9                                 ;
inc si                                 ;
inc si                                 ;
loop @@8                               ;
jcxz @Err5                             ;
@@9 label near                             ;
xor ah,ah                              ;
lodsb                                  ;
mov byte ptr ds:Case1,al               ;
lodsb                                  ;
mov byte ptr ds:Case2,al               ;
@@A label near                             ;
lea dx,Proc\_LNG
add dx,100h
cli
mov ax,cs
mov ss,ax
mov sp,dx
sti
add dx,100h
mov cl,4
shr dx,cl
mov ax,cs
add ax,dx
mov ds,ax
mov es,ax
xor si,si
xor di,di
xor dx,dx
xor cx,cx
mov ax,4200h
mov bx,cs:Handle1
int 21h
mov cx,cs:Head\_Size
xor dx,dx
mov ah,3fh
int 21h
cmp word ptr cs:relocations,0
jz @@10
mov bx,cs:Key1
mov cx,cs:Relocations
shl cx,1
shl cx,1
mov si,word ptr cs:Relocation\_table\_entry
call Unlock\_Process
@@10 label near
mov ax,word ptr cs:Relocations
mov ds:6h,ax
mov ax,word ptr cs:Ss0
mov ds:0eh,ax
mov ax,word ptr cs:Sp0
mov ds:10h,ax
mov ax,word ptr cs:Ip0
mov ds:14h,ax
mov ax,word ptr cs:Cs0
mov ds:16h,ax
;--------------------
mov ax,word ptr cs:Code\_Size\_L
mov dx,word ptr cs:Code\_Size\_H
add ax,cs:Head\_Size
adc dx,0
mov cs:Offset2\_L,ax
mov cs:Offset2\_H,dx
mov cx,200h
div cx
or dx,dx
jz @@11
inc ax
@@11 label near
mov ds:02h,dx
mov ds:04h,ax
xor dx,dx
mov cx,cs:Head\_size
call write
mov cx,cs:Code\_size\_L
mov dx,cs:Code\_size\_H
push cx
push dx
call read
mov bx,cs:handle1      ;;;;
mov dx,cs:Offset3\_L
mov cx,cs:Offset3\_H
mov ax,4200h
int 21h
mov cx,cs:Locker\_Size\_L
mov dx,cs:Locker\_Size\_H
;push cx
;push dx
call read
xor dx,dx
xor si,si
call decode
pop  dx
pop cx
call write
;-----------------------------------------------------------------
;   处理OVL
```
