
# 实现【课题 2.3】

> 原文：[`www.pediy.com/kssd/pediy10/57475.html`](https://www.pediy.com/kssd/pediy10/57475.html)

## [课题 2.3] 汇编入门小程序联系 2

课题要求：编写 3 个小程序

1. 在以 `BUF` 为起始地址的数组中存入了一个字符串，检测该字符串中是否有数字字符，若有，将 `DL` 的 `D3` 位置 `1`，否则将该位置 `0`。
2. 从键盘输入一个字符串，分类统计该字符串中字母、数字及其它字符的个数，并显示出分类统计值。
3. 在起始地址为 `ARRAY` 的数组中，存入了 `100` 个 `16` 位带符号数，试求该数组中的最大值、最小值及平均值，并将这 `3` 个值显示出来。

### 实现\课题 2.3\ 代码 1

```
assume CS:code,DS:data

data segment
  buf db 'caaaaabbbbbbbbbcccccecccaaabbb','$'
data ends

code segment
start:    mov AX,data
  mov DS,AX

  lea BX,buf
  mov CX,001ch
  mov DX,0h
s:
  mov AH,[BX]
  cmp AH,30h
  js theend
  cmp AH,39h
  jns theend
  
s1:  or DX,8h
  
theend:
  inc BX
  loop s
  
  mov ax,4c00h
  int 21h
code ends
end  start
```

### 实现\课题 2.3\ 代码 2

```
assume CS:code,DS:data

data segment
line db '23456@#%SDFghj','$'
da1 db 0
da2 db 0
da3 db 0
res1 db 0ah,0dh,'letter:','$'
res2 db 0ah,0dh,'number:','$'
res3 db 0ah,0dh,'others:','$'
data ends

code segment
start:
  mov AX,data
  mov DS,AX

  lea BX,line
  mov CX,000eh

s:
  mov DL,[BX]
  cmp DL,30h
  js s0
  cmp DL,39h
  jns c1

s1:
  mov DH,da1
  inc DH
  mov da1,DH
  jmp theends
c1:
  cmp DL,41h
  js s0

c2:
  cmp DL,5ah
  jns c3

s2:
  mov DH,da2
  inc DH
  mov da2,DH
  jmp theends

c3:
  cmp DL,61h
  js s0
  cmp DL,7ah
  js s2
  jz s2

s0:
  mov DH,da3
  inc DH
  mov da3,DH
  jmp theends

theends:
  inc BX
  loop s
  
  lea DX,line
  mov Ah,09
  int 21h

  lea DX,res1
  mov Ah,09
  int 21h

        mov DL,DS:[da1]
  add DL,48
        mov Ah,02
  int 21h

  lea DX,res2
  mov Ah,09
  int 21h

       mov DL,DS:[da2]
  add DL,48
        mov Ah,02
  int 21h

  lea DX,res3
  mov Ah,09
  int 21h

       mov DL,DS:[da3]
   add DL,48
        mov Ah,02
  int 21h
  
        mov AX,4c00h
   int 21h 
code ends
end  start
```

### 实现\课题 2.3\ 代码 3

```
assume cs:code,ds:data

data segment 
array dw 1,2,3,4,5,6,7,8,9,10
       dw 2,2,3,4,5,6,7,8,9,10
       dw 3,2,3,4,5,6,7,8,9,200
       dw 4,2,3,4,5,6,7,8,9,10
       dw 5,2,3,4,5,6,7,8,9,10
       dw 6,2,3,4,51,6,70,8,9,10
       dw 7,2,3,4,5,6,7,8,9,10
       dw 8,2,3,4,5,6,7,8,9,101
       dw 9,2,3,4,5,6,7,8,9,10
       dw 10,2,3,4,5,6,7,8,9,10
temp   dw ?
result dw ?
max    dw ?
min    dw ?
aver   dw ?
data ends  

code segment
start:
       mov ax,data
  mov ds,ax

  mov ax,0      ;求和
  mov bx,0
  mov cx,100
s:
  add ax,array[bx]
   inc bx
  inc bx
  loop s
  mov ds:[result],ax
  
  mov bx,0      ;求最大值
  mov ax,array[bx]
  mov ds:[temp],ax
  mov cx,100
s1:
  mov ax,ds:[temp]
  cmp array[bx],ax
  jc  less
  mov ax,array[bx]
  mov ds:[temp],ax
less:
  inc bx
  inc bx
  loop s1
  mov ax,ds:[temp]
  mov ds:[max],ax

  mov bx,0      ;求最小值
  mov ax,array[bx]
  mov ds:[temp],ax
  mov cx,100
s2:
  mov ax,ds:[temp]
  cmp array[bx],ax
  jnc  greater
  mov ax,array[bx]
  mov ds:[temp],ax
greater:
  inc bx
  inc bx
  loop s2
  mov ax,ds:[temp]
  mov ds:[min],ax
  
  mov ax,ds:[result]
  mov dx,0
  mov bx,100      ;求平均值
  div bx
  mov ds:[aver],ax    
    
  mov ax,4c00h
  int 21h

code ends
end start       
```
