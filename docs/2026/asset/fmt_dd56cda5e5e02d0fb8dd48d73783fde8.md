
# [原创]16位汇编写的五子棋

```asm
data segment

SG DB '022K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0T1M7#2)9J5k6h3u0U0j5$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6Q4x3U0M7'.

SG3 DB 'Do you want to play again?(Yes/No)'

sg1 db 0dbh,0dbh

sg2 db 0dbh

row db 15

col db 40

circle DB 00H,00H,07H,0C0H,1FH,0F0H,3FH,0F8H,3FH,0F8H,7FH,0FCH,7FH,0FCH,7FH,0FCH
        DB 7FH,0FCH,7FH,0FCH,3FH,0F8H,3FH,0F8H,1FH,0F8H,07H,0C0H,00H,00H,00H,00H
flag db 1
status db 1
victory db 0
num db 1
form db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
      db 40 dup (0)
data ends

;
code segment

assume cs:code,ds:data,es:data
start:
push ds
sub ax,ax
push ax
mov ax,data
mov ds,ax
mov ax,data
mov es,ax
fflag:
mov num,1
mov row,15
mov col,40
mov status,1
mov victory,0
mov num,1
mov si,offset form
mov cx,488h
again:
mov byte ptr [si],0
inc si
loop again
MOV AH,00H
MOV AL,12H
INT 10H
MOV AH,0BH
MOV BH,01
MOV BL,00H
INT 10H
mov        dh,0
mov        dl,0
mov        ah,2
mov        bh,0
int        10h
mov cx,29
flag1:
push cx
mov cx,80
flag2:
push cx
mov        ah,9
mov        al,0dbh
mov        bh,0h
mov        bl,00111001B
mov        cx,1
int        10h
inc dl
mov        ah,2
mov        bh,0
int        10h
pop cx
loop flag2
inc dh
mov dl,0
pop cx
loop flag1
mov bp,offset sg1
mov cx,1
mov dx,0f28h
mov bh,0h
mov al,0h
mov bl,00000000b
mov ah,13h
int 10h
mov bp,offset sg1
mov cx,1
mov dx,0f29h
mov bh,0h
mov al,0h
mov bl,00000000b
mov ah,13h
int 10h
MOV BP,OFFSET SG
MOV     CX,20
MOV     DX,1D3BH
MOV BH,0H
MOV AL,0H
MOV BL,00111010B
MOV AH,13H
INT 10H
keyboard:
MOV  AH,0
INT  16H
PUSH AX
MOV  AH,1
INT  16H
JZ keyboard2
  POP AX
JMP keyboard
EXIT1:
JMP EXIT
keyboard2:
  POP AX
  CMP  AL,1BH
  jz exit1
  CMP al,'a'
  jz left
  CMP al,'s'
  jz down
  CMP al,'d'
  jz right
  CMP al,'w'
  jz up
  cmp al,' '
  jz  blank
  jmp keyboard
up:
call up1
jmp keyboard
down:
call down1
jmp keyboard
left:
call left1
jmp keyboard
right:
call right1
jmp keyboard
blank:
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
cmp byte ptr [si],1
jnz bno1
jmp keyboard
bno1:
cmp byte ptr [si],2
jnz bdrawcircle1
jmp keyboard
bdrawcircle1:
mov si,offset circle
cmp flag,1
jz bflag1
cmp flag,2
jz bflag2
bflag1:
call show2
call judgeperfect
cmp victory,1
jz flag3
mov flag,2
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
mov byte ptr [si],1
jmp keyboard
fflag1:
jmp fflag
bflag2:
call show2
call judgeperfect
cmp victory,1
jz flag3
mov flag,1
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
mov byte ptr [si],2
jmp keyboard
FLAG3:
MOV  AH,0
INT  16H
CMP  AL,1BH
JZ  EXIT
JMP FLAG3
EXIT:
mov ax,0003h
int 10h
MOV BP,OFFSET SG3
MOV     CX,34
MOV     DX,0H
MOV BH,0H
MOV AL,0H
MOV BL,00001111B
MOV AH,13H
INT 10H
MOV  AH,0
INT  16H
cmp al,'y'
jz fflag1
cmp al,'Y'
jz fflag1
MOV AX,4C00H
INT 21H
up1 proc near
mov bp,offset sg1
mov cx,2
mov dh,row
mov dl,col
mov bh,0h
mov al,0h
mov bl,00111001b
mov ah,13h
int 10h
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
cmp byte ptr [si],1
jnz uno1
mov status,1
jmp ucircle1
uno1:
cmp byte ptr [si],2
jnz uflag1
mov status,2
ucircle1:
mov si,offset circle
call show
uflag1:
cmp row,0
jz up11
dec row
up11:
mov bp,offset sg1
mov cx,2
mov dh,row
mov dl,col
mov bh,0h
mov al,0h
mov bl,00000000b
mov ah,13h
int 10h
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
cmp byte ptr [si],1
jnz uno12
mov status,1
jmp ucircle2
uno12:
cmp byte ptr [si],2
jnz uexit
mov status,2
ucircle2:
mov si,offset circle
call show1
uexit:
RET
up1 endp
down1 proc near
mov bp,offset sg1
mov cx,2
mov dh,row
mov dl,col
mov bh,0h
mov al,0h
mov bl,00111001b
mov ah,13h
int 10h
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
cmp byte ptr [si],1
jnz dno1
mov status,1
jmp dcircle1
dno1:
cmp byte ptr[si],2
jnz dflag1
mov status,2
dcircle1:
mov si,offset circle
call show
dflag1:
cmp row,28
jz down11
inc row
down11:
mov bp,offset sg1
mov cx,2
mov dh,row
mov dl,col
mov bh,0h
mov al,0h
mov bl,00000000b
mov ah,13h
int 10h
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
cmp byte ptr [si],1
jnz dno12
mov status,1
jmp dcircle2
dno12:
cmp byte ptr [si],2
jnz dexit
mov status,2
dcircle2:
mov si,offset circle
call show1
dexit:
ret
down1 endp
left1 proc near
mov bp,offset sg1
mov cx,2
mov dh,row
mov dl,col
mov bh,0h
mov al,0h
mov bl,00111001b
mov ah,13h
int 10h
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
cmp byte ptr [si],1
jnz lno1
mov status,1
jmp lcircle1
lno1:
cmp byte ptr [si],2
jnz lflag1
mov status,2
lcircle1:
mov si,offset circle
call show
lflag1:
cmp col,0
jz left11
dec col
dec col
left11:
mov bp,offset sg1
mov cx,2
mov dh,row
mov dl,col
mov bh,0h
mov al,0h
mov bl,00000000b
mov ah,13h
int 10h
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
cmp byte ptr [si],1
jnz lno12
mov status,1
jmp lcircle2
lno12:
cmp byte ptr [si],2
jnz lexit
mov status,2
lcircle2:
mov si,offset circle
call show1
lexit:
ret
left1 endp
right1 proc near
mov bp,offset sg1
mov cx,2
mov dh,row
mov dl,col
mov bh,0h
mov al,0h
mov bl,00111001b
mov ah,13h
int 10h
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
cmp byte ptr [si],1
jnz rno1
mov status,1
jmp rcircle1
rno1:
cmp byte ptr [si],2
jnz rflag1
mov status,2
rcircle1:
mov si,offset circle
call show
rflag1:
cmp col ,78
jz right11
inc col
inc col
right11:
mov bp,offset sg1
mov cx,2
mov dh,row
mov dl,col
mov bh,0h
mov al,0h
mov bl,00000000b
mov ah,13h
int 10h
mov si,offset form
mov al,row
mov bl,40
mul bl
add si,ax
mov al,col
mov ah,0
mov bl,2
div bl
add si,ax
cmp byte ptr [si],1
jnz rno12
mov status,1
jmp rcircle2
rno12:
cmp byte ptr [si],2
jnz rexit
mov status,2
rcircle2:
mov si,offset circle
call show1
rexit:
ret
right1 endp
show proc near
push ax
push bx
push cx
push dx
push si
push bp
  push ax
  mov al,row
  mov bl,16
  mul bl
  mov dx,ax
  pop ax
  mov bh,0    ;页
mov bp,16  ;外圈(行) ,垂直点数
sh0:
  push bp
  mov bp,16  ;内圈(列),水平点数
  push ax
  mov al,col
  mov bl,8
  mul bl
  mov cx,ax
  pop ax
  mov ax,[si] ;取值
xchg al,ah  ;交换
mov di,ax
  mov ah,0ch
  cmp status,1
  jz person1
  cmp status,2
  jz person2
person1:
  mov al,00001011b
  jmp sh2
person2:
mov al,00001100b
sh2:
  shl di,1  ;移出到 cf
jnc sh3 ;无点
int 10h ;有点印出
sh3:
  inc cx  ;下一点
dec bp  ;完了一列?
jnz sh2 ;未
add si,2 ;下一行
pop bp  ;
inc dx  ;坐标移下一行
dec bp  ;行回圈
jnz sh0
pop bp
pop si
pop dx
pop cx
pop bx
pop ax
ret
show endp
show1 proc near
push ax
push bx
push cx
push dx
push si
push bp
push ax
mov al,row
mov bl,16
mul bl
mov dx,ax
pop ax
mov bh,0    ;页
mov bp,16  ;外圈(行) ,垂直点数
```
