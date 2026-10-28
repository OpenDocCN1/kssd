
```
add si,ax

pop bx

pop ax

push ax

mov al,bl

mov ah,0

push bx

mov bl,2

div bl

add si,ax

pop bx

pop ax

cmp byte ptr [si],cl

jz lb1next4

mov num,1

jmp jrightoblique1

lb1next4:

inc num

cmp num,5

jz lb1victory

jmp jexit

lb1victory:

mov victory,1

mov num,1

jmp jexit

jrightoblique1:

mov al,row

mov bl,col

mov cl,flag

inc al

dec bl

dec bl

mov si,offset form

push ax

push bx

mov bl,40

mul bl

add si,ax

pop bx

pop ax

push ax

mov al,bl

mov ah,0

push bx

mov bl,2

div bl

add si,ax

pop bx

pop ax

cmp byte ptr [si],cl

jz rb1next1

mov num,1

jmp jrightoblique2

rb1next1:

inc num

inc al

dec bl

dec bl

mov si,offset form

push ax

push bx

mov bl,40

mul bl

add si,ax

pop bx

pop ax

push ax

mov al,bl

mov ah,0

push bx

mov bl,2

div bl

add si,ax

pop bx

pop ax

cmp byte ptr [si],cl

jz rb1next2

mov num,1

jmp jrightoblique2

rb1next2:

inc num

inc al

dec bl

dec bl

mov si,offset form

push ax

push bx

mov bl,40

mul bl

add si,ax

pop bx

pop ax

push ax

mov al,bl

mov ah,0

push bx

mov bl,2

div bl

add si,ax

pop bx

pop ax

cmp byte ptr [si],cl

jz rb1next3

mov num,1

jmp jrightoblique2

rb1next3:

inc num

inc al

dec bl

dec bl

mov si,offset form

push ax

push bx

mov bl,40

mul bl

add si,ax

pop bx

pop ax

push ax

mov al,bl

mov ah,0

push bx

mov bl,2

div bl

add si,ax

pop bx

pop ax

cmp byte ptr [si],cl

jz rb1next4

mov num,1

jmp jrightoblique2

rb1next4:

inc num

cmp num,5

jz rb1victory

jmp jexit

rb1victory:

mov victory,1

mov num,1

jmp jexit

jrightoblique2:

mov al,row

mov bl,col

mov cl,flag

dec al

inc bl

inc bl

mov si,offset form

push ax

push bx

mov bl,40

mul bl
add si,ax
pop bx
pop ax
push ax
mov al,bl
mov ah,0
push bx
mov bl,2
div bl
add si,ax
pop bx
pop ax
cmp byte ptr [si],cl
jz jr2next1
mov num,1
jmp jexit
jr2next1:
inc num
dec al
inc bl
inc bl
mov si,offset form
push ax
push bx
mov bl,40
mul bl
add si,ax
pop bx
pop ax
push ax
mov al,bl
mov ah,0
push bx
mov bl,2
div bl
add si,ax
pop bx
pop ax
cmp byte ptr [si],cl
jz jr2next2
mov num,1
jmp jexit
jr2next2:
inc num
dec al
inc bl
inc bl
mov si,offset form
push ax
push bx
mov bl,40
mul bl
add si,ax
pop bx
pop ax
push ax
mov al,bl
mov ah,0
push bx
mov bl,2
div bl
add si,ax
pop bx
pop ax
cmp byte ptr [si],cl
jz jr2next3
mov num,1
jmp jexit
jr2next3:
inc num
dec al
inc bl
inc bl
mov si,offset form
push ax
push bx
mov bl,40
mul bl
add si,ax
pop bx
pop ax
push ax
mov al,bl
mov ah,0
push bx
mov bl,2
div bl
add si,ax
pop bx
pop ax
cmp byte ptr [si],cl
jz jr2next4
mov num,1
jmp jexit
jr2next4:
inc num
cmp num,5
jz jr2victory
jmp jexit
jr2victory:
mov victory,1
mov num,1
jmp jexit
jexit:
mov num,1
ret
judgeperfect endp
code ends
end start
```

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
