
# 实现【课题 2.2】

> 原文：[`www.pediy.com/kssd/pediy10/57474.html`](https://www.pediy.com/kssd/pediy10/57474.html)

\[课题 2.2\]汇编入门小程序联系 1

课题要求：编写 3 个小程序，要求如下

（1）编写程序，从键盘输入一个字母字符，找出该字母的大写/小写字母前导字母和后续字母，并顺序显示出这 3 个字母字符。

实现（2）编写程序，比较两个字符串是否相同，并输出比较结果。

（3）从键盘上输入一个 4 位十六进制数，分别将该数以十六进制和十进制形式显示出来。

实现\课题 2.2\

代码:

```
;--------------------------------------------------------------------
;name:top2o2.asm
;
;purpose:show the capital or lowercase letter,leader letter and
;        the follow-up letter which based on the letter given by user
;--------------------------------------------------------------------

assume cs:code,ds:data

data segment
  out1      db 'please input a letter:','$'
  out2      db 'the capital/lowercase letter is:','$'
  out3     db 'the leader letter is:','$'
  out4      db 'the follow-up letter is:','$'
  out5     db 'press ESC to exit,or any other key to continue!','$'
  c         db 0
  c1        db 0
  leader    db 0
  followup  db 0
data ends

code segment
  start:mov ax,data
        mov ds,ax

      s:
  lea dx,out1
        mov ah,09
        int 21h

        mov ah,01
        int 21h
        mov ds:[c],al

  mov dl,0dh  ;换行
  mov ah,02
  int 21h
  mov dl,0ah
  mov ah,02
  int 21h

        mov bh,ds:[c]

  cmp bh,27  ;如果是 ESC 键,退出循环
  jz  e1
        jmp e2
     e1:jmp e
        
      e2:cmp bh,'a'  ;判断是否大于'a'
        jz  s0    ;等于时跳转到 s0 执行另一个分支
        cmp bh,'a'  ;
        jnc  s3    ;大于'a'时跳转到 s1 判断是否小于'z'
        jmp s0    ;不符合条件时跳转到 s0

     s3:cmp bh,'z'  ;判断是否小于'z'
  jc s1    ;小于'z'时跳转到 s1
  jmp s0
    
     s1:sub bh,32  ;符合条件则说明 c 为小写字母，求它的大写字母
        mov ds:[c1],bh
  jmp s2       ;不执行另一分支，直接执行条件分支后面的语句

     s0:cmp bh,'A'  ;判断是否大于'A'
        jz  s    ;等于'A'时跳转到 s3

        cmp bh,'A'  ;继续判断是否小于'Z'
        jnc  s4    ;大于'Z'则跳转到 s3 判断是否小于
        jmp s 
     s4:cmp bh,'Z'  ;判断是否小于'Z'
        jc  s5    ;小于'Z'则跳转到 s5 处，求小写字母
        jmp s

     s5:add bh,32  ;求 c 的小写字母
        mov ds:[c1],bh

     s2:
  mov bh,ds:[c]
  mov ds:[leader],bh
  mov ds:[followup],bh

  sub byte ptr ds:[leader],1    ;求 c 的前导字母
  add byte ptr ds:[followup],1  ;求 c 的后续字母

  lea dx,out2
        mov ah,09
  int 21h
  mov dl,ds:[c1]  ;输出大写/小写字母
  mov ah,02
  int 21h

  mov dl,0dh  ;换行
  mov ah,02
  int 21h
  mov dl,0ah
  mov ah,02
  int 21h

  lea dx,out3
  mov ah,09
  int 21h
  mov dl,ds:[leader]  ;输出前导字母
  mov ah,02
  int 21h

  mov dl,0dh  ;换行
  mov ah,02
  int 21h
  mov dl,0ah
  mov ah,02
  int 21h

  lea dx,out4
  mov ah,09
  int 21h
  mov dl,ds:[followup]  ;输出后续字母
  mov ah,02
  int 21h

  mov dl,0dh  ;换行
  mov ah,02
  int 21h
  mov dl,0ah
  mov ah,02
  int 21h
  
  lea dx,out5
  mov ah,09
  int 21h
  
  mov dl,0dh  ;换行
  mov ah,02
  int 21h
  mov dl,0ah
  mov ah,02
  int 21h
  
  mov dl,0dh  ;换行
  mov ah,02
  int 21h
  mov dl,0ah
  mov ah,02
  int 21h

  jmp s

      e:mov ax,4c00h
  int 21h
code ends
end  start
```

实现\课题 2.2\

代码:

```
DATA SEGMENT
    STRING1 DB "Anything is possible."    ；定义字符串 1
    STRING2 DB "Anything is possible."    ；定义字符串 2
    MESS1 DB "Matched",13,10,'$'              
    MESS2 DB "No Matched",13,10,'$'
DATA ENDS
CODE SEGMENT
    ASSUME CS:CODE,DS:DATA,ES:DATA
    START:PUSH DS                        ；DX 入栈
          XOR AX,AX                      ；AX 清零
          PUSH AX                        ；AX 入栈
          MOV AX,DATA            
          MOV DS,AX
          MOV ES,AX
          LEA SI,STRING1                  ；设置源串地址
          LEA DI,STRING2                  ；设置目标串地址
          CLD                            ；设置操作方向 DF，从低地址到高地址进行搜索
          MOV CX,21                      ；设置比较字符串的长度,也是循环的长度。
          REPZ CMPSB                        
          JZ MATCH                       ；比较字符串是否相等，相等则跳到 MATCH，不相等则顺序执行。
          LEA DX,MESS2                   ；显示字符串为 Unmatched
          JMP DISP
MATCH:    LEA DX,MESS1                   ；比较结束后，全部相同显示 MESS1，Matched                
DISP:     MOV AH,09H
          INT 21H                        ；显示字符串"Matched",或者是"Unmatched"
ENDS
END START
```

实现\课题 2.2\

代码:

```
code segment
assume cs:code
main:
  mov ah,08h
  int 21h
  cmp al,'a'
  jbe exit
  cmp al,'z'
  jb  next
  cmp al,'A'
  jbe exit
  cmp al,'Z'
  jae exit
next:
  mov dl,al
  dec dl
  mov cx,3
  mov ah ,02h
again:
  int 21h
  inc dl
  loop again
exit:
  mov ah ,4ch
  int 21h
code ends
end main
```
