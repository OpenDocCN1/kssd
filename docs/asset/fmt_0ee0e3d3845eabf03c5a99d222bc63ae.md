
# 【成果 2.5】查询字符串中与某个字符首个匹配位置&连接字符串&比较两个字符串

> 原文：[`www.pediy.com/kssd/pediy10/58757.html`](https://www.pediy.com/kssd/pediy10/58757.html)

## [课题 2.5]汇编入门小程序联系 4

课题要求：编写 3 个小程序

### 1. 在一个串中查找给定字符的第一个匹配之处

编写一个通用的子程序来实现在一个串中查找给定字符的第一个匹配之处的功能。字符串必须以 0 结束,区分大小写。

子程序描述：

名称：`string_char`

功能：在一个串中查找给定字符的第一个匹配之处

参数：`(ch)`=字符  
`ds:si` 指向字符串的首地址

返回：`(ax)`=匹配的位置

`(ax)=0` 表示未找到匹配位置

应用举例：在字符串 `I Love Masm!` 查找 `M` 的第一个匹配的位置，并输出测试结果。

分析：在字符串`'I Love Masm!'`中`'M'`的首个匹配的位置值为 8，结果应该为 8

算法很简单，逐个比较并计算位置即可，如果找不到返回 0

代码:

```
;==============================
;filename:top2o5a.asm
;date:2008/1/21
;==============================

assume cs:code,ds:data,ss:stack

stack segment
 dw 64 dup(0)
stack ends

data segment
 str db 'I Love Masm!',0
 m db 'M',0
 i db 6 dup('0')
data ends

code segment
start:
 mov ax,data
 mov ds,ax
 
 mov ax,stack
 mov ss,ax
 mov sp,128

;查找在字符串 str 中的首个与 CH 字符匹配的位置
 lea si,str ;(SI)=str
 mov ch,ds:[m] ;(CH)=DS:[m]
 call string_char
 
;将位置值转换为一个字符串,方便输出
 lea si,i ;(SI)=i
 call dtoc

;在第 5 行、第 1 列以白色输出父串
 lea si,str
 mov dh,5
 mov dl,1
 mov cl,1010b
 call show_str

;在第 6 行、第 1 列以白色输出要匹配的字符
 lea si,m
 mov dh,6
 mov dl,1
 mov cl,1010b
 call show_str

;输出结果
 lea si,i
 mov dh,7
 mov dl,1
 mov cl,1010b
 call show_str

 mov ax,4c00h
 int 21h

;==================================================
;名称：string_char
;功能：在一个串中查找给定字符的第一个匹配的位置
;参数：(ch)=字符
; ds:si 指向字符串的首地址
;返回：(ax)=匹配的位置
; (ax)=0 表示未找到匹配位置
;==================================================
string_char proc near
 push si
 push di

 mov di,si ;(DI)=(SI)
s2:
 push cx 
 mov cl,0 ;如果已搜索到字符串末尾,求找到,退出循环
 mov ch,ds:[di]
 jcxz outl
 
 pop cx
 cmp byte ptr ds:[di],ch ;判断当前字符是否为等于要匹配的字符
 jz ok2 ;DS:[DI]=(CH),搜索到匹配的字符,退出循环
 inc di ;(DI)=(DI)+1 指向下一个字符
 jmp short s2
outl: 
 mov ax,0 ;(AX)=0 未找到,直接返回
 pop di
 pop si 
 ret
ok2:
 sub di,si ;首个匹配的位置值为(DI)=(DI)-(SI)+1
 inc di
 mov ax,di
 
 pop di
 pop si
 ret
string_char endp

;=================================================================
;名称：dtoc
;功能：将 word 型数据转变为表示十进制数的字符串，字符串以 0 为结尾符
;参数：(ax)=word 型数据
; ds:si 指向字符串的首地址
;返回：无
;=================================================================

dtoc proc near
 push ax
 push bx
 push cx
 push si
 push di

 mov di,si
 mov bx,10
 mov cx,0
s:
 push cx
 mov cx,ax
 jcxz enddtoc
 pop cx
 mov dx,0
 div bx
 mov ds:[di],dl
 add byte ptr ds:[di],30h 
 inc di
 inc cx
 jmp short s
enddtoc:
 pop cx
 mov byte ptr ds:[di],0 ;在字符串末尾补 0
 dec di ;(DI)=(DI)-1 指向字符串最后一个字符
 mov ax,cx ;(cx)=字符串的长度
 mov bl,2 
 div bl
 mov cl,al ;(CL)=(AL)=(CX)/2
 jcxz r ;如果(CX)=0 时,会执行循环一次,然后(CX)=(CX)-1=0-1=FFFFH,发生溢出错误
 ;故添加代码(cx)=0 时不执行循环跳转指令 jcxz r
s1:
 mov al,ds:[di] ;求得余数之后，要将字符串逆序，把余数摆正
 mov bl,ds:[si]
 mov ds:[di],bl
 mov ds:[si],al
 inc si ;(SI)=(SI)+1
 dec di ;(DI)=(DI)-1
 loop s1
r:
 pop di
 pop si
 pop cx
 pop bx
 pop ax
 ret
dtoc endp

;===========================================================
;名称：show_str
;功能：在指定的位置，用指定的颜色，显示一个用 0 结束的字符串
;参数：(dh)=行号(取值范围 0~24),(dl)=列号(取值范围 0~79),
; (cl)=颜色,ds:si 指向字符串的首地址
;返回：无
;===========================================================
show_str proc near
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
show_str endp 

code ends
end start
```

测试结果:

![](img/c60ac5352063d4ff0bf8276dc1fa8b3a.jpg)

图 1 输出找出首个匹配的位置值 8

### 2. 字符串拼接

编写一个通用的子程序来实现将源字符串拼接到目的字符串的功能。字符串必须以 0 结束。

子程序描述：

名称：`string_cat`

功能：将源字符串拼接到目的字符串

参数：`ds:si` 指向源字符串的首地址  
`ds:di` 指向目的字符串的首地址

返回：无

应用举例：将字符串 `I Love Win32 Assembly Language!`拼接在 `I Love 80X86 Assembly Language!`后面，并输出结果到屏幕上。

分析：程序流程，1。使用一个循环查找第一个字符串的末尾 2。将第二个字符串逐个字符连接到第一个字符串 3。在连接后的字符串末尾添加 0，方便输出

代码:

```
;==============================
;filename:top2o5b.asm
;date:2008/1/22
;==============================

assume cs:code,ds:data,ss:stack

stack segment
 dw 64 dup(0)
stack ends

data segment
 str1 db 'I Love Win32 Assembly Language!',100 dup(0)
 str2 db 'I Love 80X86 Assembly Language!',0
data ends

code segment
start:
 mov ax,data
 mov ds,ax

 mov ax,stack
 mov ss,ax
 mov sp,128

;将目的串 str2 连接到源串 str1
 lea di,str1 ;(DI)=str1 取目的串的偏移地址
 lea si,str2 ;(SI)=str2 取源串的偏移地址
 call string_cat

;在第 10 行、第 10 列输出连接后的字符串
 lea si,str1 ;(SI)=str1 字符串首地址
 mov dh,10 ;(DH)=10 第 10 行
 mov dl,10 ;(DL)=10 第 10 列
 mov cl,00001010b ;(CL)=10001010B 字符串属性为高亮、绿色
 call show_str

 mov ax,4c00h
 int 21h

;====================================
;名称：string_cat
;功能：将源字符串拼接到目的字符串
;参数：ds:si 指向源字符串的首地址
; ds:di 指向目的字符串的首地址
;返回：无
;====================================
string_cat proc near

;保护现场
 push ax
 push si
 push di

;将 DI 指向目的数组的末尾 0
s1:
 cmp byte ptr ds:[di],0 ;DS:[DI]==0 时表示已到字符串末尾
 jz s2
 inc di ;(DI)=(DI)+1 DI 指向下一个字符
 jmp short s1

;将源数组 SI 连接到目的数组,循环终止条件为 SI 指向数组末尾 0
s2:
 cmp byte ptr ds:[si],0 ;DS:[SI]==0 时表示已将源串的每一个字符连接到目的串
 jz ok1
 mov al,ds:[si] ;DS:[DI]=DS:[SI]
 mov ds:[di],al
 inc si ;(SI)=(SI)+1 SI 指向下一个待连接的字符
 inc di ;(DI)=(DI)+1 DI 指向下一个单元
 jmp short s2

ok1:
 mov byte ptr ds:[di],0 ;在目的串添加终止符 0,因为不能保证每次连接字符串后目的串的末尾字符为 0
;恢复现场
 pop di
 pop si
 pop ax
 ret
string_cat endp

;===========================================================
;名称：show_str
;功能：在指定的位置，用指定的颜色，显示一个用 0 结束的字符串
;参数：(dh)=行号(取值范围 0~24),(dl)=列号(取值范围 0~79),
; (cl)=颜色,ds:si 指向字符串的首地址
;返回：无
;===========================================================

show_str proc near
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
show_str endp 

code ends
end start
```

测试结果：

![](img/dad11e21b8310d1af62dcc3909e7759b.jpg)

图 2 输出连接后的字符串

### 3. 串比较

编写一个通用的子程序来实现两个字符串的比较的功能。字符串必须以 0 结束。

子程序描述：

名称：`string_compare`

功能：比较两个字符串

参数：`ds:si` 指向第一个字符串的首地址  
`ds:di` 指向第二个字符串的首地址

返回：`(ah)=0` 两个字符串相等  
`(ah)=1` 第一个字符串大于第二个字符串  
`(ah)=-1` 第一个字符串小于第二字符串

应用举例：比较字符串 `I Love 80X86 Assembly Language!`和 `I Love Win32 Assembly Language!`，并在屏幕上输出比较的结果。

分析：字符串`'I Love 80X86 Assembly Language!'`比字符串`'I Love Win32 Assembly Language!'`要小，故应该输出－1。比较的方法就是逐个比较，以下比较子程序中有个循环是核心

来的，它比较全面的考虑了各种情况，如是否检测到字符串末尾等。当然程序还能更简单些，只能凑和着看了。![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

代码:
