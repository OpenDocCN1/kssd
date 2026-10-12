
# 【成果 2.4】逆序排列字符串&字符串大写转换成小写&字符串小写转换成大写

> 原文：[`www.pediy.com/kssd/pediy10/57937.html`](https://www.pediy.com/kssd/pediy10/57937.html)

## \[课题 2.4\]汇编入门小程序联系 3

**课题要求：编写 3 个小程序**

### 1. 逆序排列字符串

编写一个通用的子程序来实现逆序排列字符串这个功能。

**子程序描述：**

**名称**：`revers_str`

**功能**：将一个指定的字符串逆序排列

**参数**：`ds:si` 指向字符串的首地址

**返回**：无

**应用举例**：将字符串 `I Love Masm!` 逆序排列并输出屏幕查看结果。

**分析**：程序交换算法很简单，就是定义一个头指针和一个尾指针指向字符串头尾，然后开始交换，如果相遇或交叉则表示字符串逆序完成。

代码:

```
;=====================================
;filename:top2o4a.asm
;purpose:逆序排列字符串
;=====================================

assume cs:code,ss:stack,ds:data

stack segment
 dw 64 dup(?)
stack ends

data segment
 str db 'I Love Masm!',0
data ends

code segment
start:
 mov ax,data
 mov ds,ax

 mov ax,stack
 mov ss,ax
 mov sp,128
 
 lea si,str
 mov dh,5
 mov dl,1
 mov cl,11b
 call show_str
 
 lea si,str
 call revers_str
 
 lea si,str
 mov dh,6
 mov dl,1
 mov cl,11b
 call show_str
 
 mov ax,4c00h
 int 21h

;======================================
;名称：str_len
;功能：求一个字符串的长度
;参数：ds:si 指向字符串的首地址
;返回：(cx)=字符串的长度
;======================================
str_len:
 push ax
 push si
 mov ax,0
s1:
 mov ch,ds:[si]
 mov cl,0
 jcxz ok1
 inc ax
 inc si
 jmp short s1
ok1:
 mov cx,ax
 pop si
 pop ax
 ret
;======================================
;名称：revers_str
;功能：将一个指定的字符串逆序排列
;参数：ds:si 指向字符串的首地址
;返回：无
;======================================
revers_str:
 push ax
 push bx
 push cx
 push dx
 push si
 push di

 call str_len
 mov di,cx
 dec di
 mov bx,2
 mov ax,cx
 mov dx,0
 div bx
 mov cx,ax

s2:
 mov al,ds:[si]
 mov ah,ds:[di]
 mov ds:[si],ah
 mov ds:[di],al
 inc si
 dec di
 loop s2
 
 pop di
 pop si
 pop dx
 pop cx
 pop bx
 pop ax
 ret

;===========================================================
;名称：show_str
;功能：在指定的位置，用指定的颜色，显示一个用 0 结束的字符串
;参数：(dh)=行号(取值范围 0~24),(dl)=列号(取值范围 0~79),
; (cl)=颜色,ds:si 指向字符串的首地址
;返回：无
;===========================================================

show_str:
 push dx
 push si
 push di
 push cx
 push ax

 mov ax,0b800h
 mov es,ax
 mov ax,160
 mul dh
 mov dh,0
 add ax,dx
 add ax,dx
 sub ax,2
 mov di,ax
 mov ah,cl
output:
 mov ch,ds:[si]
 mov cl,0
 jcxz ok

 mov byte ptr es:[di],ch
 mov byte ptr es:[di+1],ah
 inc si
 inc di
 inc di
 jmp short output
ok:
 pop ax
 pop cx
 pop di
 pop si
 pop dx
 ret
code ends
end start
```

**运行结果**：

[附件 11033](http://bbs.pediy.com/attachment.php?attachmentid=11033)

### 2. 字符串大写转换成小写

字符串大小写转换是日常经常用到的功能，编写一个通用的接口实现大写转换成小写的功能，这个接口使我们更灵活的处理字符串的大小写的转换。字符串必须以 0 结束。

**子程序描述：**

**名称**：`lowercase`

**功能**：将一个指定的字符串转换成成小写的形式

**参数**：`ds:si` 指向字符串的首地址

**返回**：无

**应用举例**：将字符串 `I Love 80X86 Assembly Language` 转换为小写形式，并输出屏幕查看转换后的字符串。

**分析**：子程序主要算法是判断是否是字符，如果不是则不处理，否则将该字符的 D5 位置 1，则可转换为相应的小写字母。

代码:

```
;=====================================
;filename:top2o4b.asm
;purpose:字符串大写转换成小写
;=====================================

assume cs:code,ss:stack,ds:data

stack segment
 dw 64 dup(?)
stack ends

data segment
 str db 'I Love Win32 Assembly Language!',0
data ends

code segment
start:
 mov ax,data
 mov ds,ax

 mov ax,stack
 mov ss,ax
 mov sp,128
 
 lea si,str
 mov dh,5
 mov dl,1
 mov cl,11b
 call show_str
 
 lea si,str
 call capital
 
 lea si,str
 mov dh,6
 mov dl,1
 mov cl,11b
 call show_str
 
 mov ax,4c00h
 int 21h

;================================================
;名称：capital
;功能：将一个指定的字符串转换成大写的形式
;参数：ds:si 指向字符串的首地址
;返回：无
;================================================
capital:
 push cx
 push si
s0:
 mov ch,ds:[si]
 mov cl,0
 jcxz ok0

 cmp byte ptr ds:[si],'a'
 jc next0
 cmp byte ptr ds:[si],'z'
 jnc t0
 jmp short next1
t0:
 cmp byte ptr ds:[si],'z'
 jz next1
 
next0:
 cmp byte ptr ds:[si],'A'
 jc next2
 cmp byte ptr ds:[si],'Z'
 jnc t1
 jmp short next1
t1:
 cmp byte ptr ds:[si],'Z'
 jz next1
 jmp short next2
 
next1:
 and byte ptr ds:[si],11011111b
next2:
 inc si
 jmp short s0
ok0:
 pop si
 pop cx
 ret

;===========================================================
;名称：show_str
;功能：在指定的位置，用指定的颜色，显示一个用 0 结束的字符串
;参数：(dh)=行号(取值范围 0~24),(dl)=列号(取值范围 0~79),
; (cl)=颜色,ds:si 指向字符串的首地址
;返回：无
;===========================================================
show_str:
 push dx
 push si
 push di
 push cx
 push ax

 mov ax,0b800h
 mov es,ax
 mov ax,160
 mul dh
 mov dh,0
 add ax,dx
 add ax,dx
 sub ax,2
 mov di,ax
 mov ah,cl
output:
 mov ch,ds:[si]
 mov cl,0
 jcxz ok

 mov byte ptr es:[di],ch
 mov byte ptr es:[di+1],ah
 inc si
 inc di
 inc di
 jmp short output
ok:
 pop ax
 pop cx
 pop di
 pop si
 pop dx
 ret
code ends
end start
```

运行结果：

[附件 11034](http://bbs.pediy.com/attachment.php?attachmentid=11034)

### 3. 字符串小写转换成大写

字符串大小写转换是日常经常用到的功能，编写一个通用的接口实现大写转换成小写的功能，这个接口使我们更灵活的处理字符串的大小写的转换。字符串必须以 0 结束。

**子程序描述：**

**名称**：`capital`

**功能**：将一个指定的字符串转换成大写的形式

**参数**：`ds:si` 指向字符串的首地址

**返回**：无

**应用举例**：将字符串 `I Love Win32 Assembly Language` 转换为大写形式，并输出屏幕查看转换后的字符串。

**分析**：子程序主要算法是判断是否是字符，如果不是则不处理，否则将该字符的 D5 位置 0，则可转换为相应的大写字母，子程序与上面子程序的差别是非常小的，通过源代码可以看出来。

代码:

```
;=====================================
;filename:top2o4c.asm
;purpose:字符串小写转换成大写
;=====================================

assume cs:code,ss:stack,ds:data

stack segment
 dw 64 dup(?)
stack ends

data segment
 str db 'I Love 80X86 Assembly Language!',0
data ends

code segment
start:
 mov ax,data
 mov ds,ax

 mov ax,stack
 mov ss,ax
 mov sp,128
 
 lea si,str
 mov dh,5
 mov dl,1
 mov cl,11b
 call show_str
 
 lea si,str
 call lowercase
 
 lea si,str
 mov dh,6
 mov dl,1
 mov cl,11b
 call show_str
 
 mov ax,4c00h
 int 21h

;================================================
;名称：lowercase
;功能：将一个指定的字符串转换成成小写的形式
;参数：ds:si 指向字符串的首地址
;返回：无
;================================================
lowercase:
 push cx
 push si
s0:
 mov ch,ds:[si]
 mov cl,0
 jcxz ok0

 cmp byte ptr ds:[si],'a'
 jc next0
 cmp byte ptr ds:[si],'z'
 jnc t0
 jmp short next1
t0:
 cmp byte ptr ds:[si],'z'
 jz next1
 
next0:
 cmp byte ptr ds:[si],'A'
 jc next2
 cmp byte ptr ds:[si],'Z'
 jnc t1
 jmp short next1
t1:
 cmp byte ptr ds:[si],'Z'
 jz next1
 jmp short next2
 
next1:
 or byte ptr ds:[si],00100000b
next2:
 inc si
 jmp short s0
ok0:
 pop si
 pop cx
 ret

;===========================================================
;名称：show_str
;功能：在指定的位置，用指定的颜色，显示一个用 0 结束的字符串
;参数：(dh)=行号(取值范围 0~24),(dl)=列号(取值范围 0~79),
; (cl)=颜色,ds:si 指向字符串的首地址
;返回：无
;===========================================================
show_str:
 push dx
 push si
 push di
 push cx
 push ax

 mov ax,0b800h
 mov es,ax
 mov ax,160
 mul dh
 mov dh,0
 add ax,dx
 add ax,dx
 sub ax,2
 mov di,ax
 mov ah,cl
output:
 mov ch,ds:[si]
 mov cl,0
 jcxz ok

 mov byte ptr es:[di],ch
 mov byte ptr es:[di+1],ah
 inc si
 inc di
 inc di
 jmp short output
ok:
 pop ax
 pop cx
 pop di
 pop si
 pop dx
 ret
code ends
end start
```

**运行结果**：

[附件 11035](http://bbs.pediy.com/attachment.php?attachmentid=11035)
