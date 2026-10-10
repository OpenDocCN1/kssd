
# 汇编基础 课题 7

> 原文：[`www.pediy.com/kssd/pediy10/61101.html`](https://www.pediy.com/kssd/pediy10/61101.html)

**\[课题 2.7\]汇编入门小程序联系 6**

课题要求：编写 3 个小程序

1. 把 `AX` 中存入的 16 位二进制数看作是 8 个二进制的“四分之一字节“。试编写一个程序，要求数一个值为 `3`（即 `11B`）的四分之一字节数。
2. 设有一段英文，其字符变量名为 `MEM`，并以 `$` 字符结束。试编写一个程序，查找单词 `MOON` 在该文本中的出现次数。
3. 有一个首地址为 `MEM` 的 `100D` 字数组，试编写程序删除数组中所有为 `0` 的项，并将后续项向前压缩，最后将数组的剩余部分补上 `0`。

因程序较小，以下程序的子程序的参数全部使用了全局变量，操作起来非常方便，但也有可能会引出一些问题，不过我意在解决具体的问题，所以我最终还是决定了使用全局变量作为子程序的参数。另外我对函数参数的进栈、出栈还不太熟悉，因此子程序写的不是很标准.

代码:

```
assume cs:code,ds:data,ss:stack

data segment
 c dw 0            ；计数器，存储 X 中 11B 的个数
 x     dw 0ccff   ；待计算 11B 的个数的参数
data ends

stack segment
 dw 64 dup(0)
stack ends

code segment
start:
    mov ax,data
    mov ds,ax　　　;初始化数据段
    
    mov ax,stack    ;初始化堆栈段
    mov ss,ax
    mov sp,128
      
    call Count         ;调用子程序计算 11B 的个数
    
    mov ax,4c00h   ;返回
    int 21h


Count proc near   
    push ax　　　　；保护在子程序中要使用的寄存器
    push bx
    push cx
    
    mov c,0　　　　;初始化计数器为 0
    mov cx,8　　　　;设置循环次数为 8，因一个字单元可分为 8 个“四分之一字节”
    mov bx,c000h  ;0c00h=1100 0000 0000 0000b，即从左到右始每次测试二位，直到 bx 移至为 0
s:
    mov ax,x          ;(AX)=X
    and ax,bx         ;测试 AX 中的某二位是否为 11B
    cmp ax,bx　　　;比较是否为 11B
    jz counter　　　;如果为 11B，跳转到 counter 处使计数器 C 加 1
    jmp s1　　　　　  ;比较不为 11B，直接跳过 inc c
counter:
    inc c
s1:          
    push cx             ;由于使用 cl 作移动位数寄存器，因此先保存一下
    mov cl,2　　　　;设置向右移动两位
    shr bx,cl　　　   ;(BX)向右移动两位，即如果(BX)=1100 0000 0000 0000B，则移动后为(BX)=0011 0000 0000 0000B
    pop cx　　　　　;恢复 CX 内容
  
    loop s                ;如果(CX)=(CX)-1 不为 0，则返回开始循环处
  
    pop cx　　　　　;恢复各个寄存器
    pop bx
    pop ax
    
    ret                     ;返回函数调用处，严格来说这个算不上是真正的函数，但为了好听点，就叫它函数吧
Count endp

code ends
end start
```

![](img/552738ad841c1a604a73ff93878c90cc.jpg)

![](img/a8d027d3a73c8a9f29ebc5ab7ad8f533.jpg)

代码:

```
assume cs:code,ds:data,ss:stack

data segment
 c  dw 0                                        ;计数器，存储字符串中“MOON”的个数
 str db 'MOONMOONMOONF','$'    ;待求字符串，传说中的数组，不过这里是以'$'结束的
data ends

stack segment
 dw 16 dup(0)                 
stack ends

code segment
start:
   mov ax,data                  ;初始化数据段和堆栈
   mov ds,ax

   mov ax,stack
   mov ss,ax

   mov cx,0                        ;初始化计数器为 0
   call Count　　　　　　　  ;调用子程序计算字符串中“MOON”的个数，结果存储在 C 中

   mov ah,4ch　　　　　　　;返回
   int 21h
   
Count proc near
    push si                          ;保护一下 SI 的内容，因下面要使用到它

    mov si,0                        ;初始化数组下标 SI 为 0
s2:
    cmp byte  ptr str[si],'$'       ;以下连续比较四个字符中是否有'$'，因为后面 0~4 个字符之中如果有'$'，则不必再检测是为为四个字符为“MOON”,经我反汇编查看了一下 WIN－TC 输出的汇编源码，我发现我写的这个检测方法要比它编译的稍微好一点点.如果没检测到’$'，则继续往下检测这四个字符是否为“MOON”，否则跳到 S3 处结束循环
    jz s3
    cmp byte ptr str[si+1],'$'
    jz s3
    cmp byte ptr str[si+2],'$'
    jz s3
    cmp byte ptr str[si+3],'$'
    jz s3

    cmp byte ptr str[si],'M'　　　;比较字符串 str 中是当前下标指向的及后面的字符是否为“MOON”,如果 Y，则继续往下执行，否则返回 S2 处执行
    jnz s2
    cmp byte ptr str[si+1],'O'
    jnz s2
    cmp byte ptr str[si+2],'O'
    jnz s2
    cmp byte ptr str[si+3],'N'
    jnz s2

    inc c　　　　　　　　　　　　　;如果运行到这里，则说明在 str 中检测到了一个"MOON"
    add si,4     　　　　　　　　　  ;(SI)=(SI)+4，指针指向字符串后四个字符的开始处
    jmp s2                                   ;进入下一轮检测

s3:
   pop si                                      ;恢复 SI
    
    ret                                          ;返回函数调用处
Count endp

code ends
end start
```

![](img/12faf40499b6fb5ed93fd340e4cb1799.jpg)

![](img/5e0850840b7003dc5467a9e4f2816fc0.jpg)

![](img/5e0850840b7003dc5467a9e4f2816fc0.jpg)

这个程由于我使用了优化后的算法，因此算法的时间复杂度由原来的 o(n²)变为了 o(n)。大概思想如下：

代码:

```
void DeleteAllZero(int iarray[],int len)
{
   int i,p;
   for(i=p=0;i<len-1;i++){
       if(iarray[i]!=0){
           p++;
       }
        else{
           if(iarray[i+1]!=0){
               iarray[p]=iarray[i+1];
               iarray[i+1]=0;
               p++;
            }
        }
   }

}
只需要存当前扫描到的第一个 0 的位置 p 即可。每次扫到一个不为 0 的数就移到 p，并将 p++；扫到一个 0 的话 p 不变，继续扫下一个数。
```

![](img/3973c6376445e7aab76a551cb00c6445.jpg)

![](img/f852890ecd0b756366290d4115304284.jpg)

以下为完整的程序，可以在程序中看到上面优化后的循环的影子。

代码:

```
assume cs:code,ds:data,ss:stack

data segment
iarray dw 1,0,2,0,3,0,4,0,5,0            ;100 个字型元素，好多啊，为了方便，我使用了小且顺序规律的元素
        dw 1,0,2,0,3,0,4,0,5,0
        dw 1,0,2,0,3,0,4,0,5,0
        dw 1,0,2,0,3,0,4,0,5,0
        dw 1,0,2,0,3,0,4,0,5,0

        dw 1,0,2,0,3,0,4,0,5,0
        dw 1,0,2,0,3,0,4,0,5,0
        dw 1,0,2,0,3,0,4,0,5,0
        dw 1,0,2,0,3,0,4,0,5,0
        dw 1,0,2,0,3,0,4,0,5,0

len dw 100                                          ;数组长度
data ends

stack segment
 dw 128 dup(0)
stack ends

code segment
start:
   mov ax,data                            ;初始化各段寄存器
   mov ds,ax

   mov ax,stack
   mov ss,ax
   mov sp,256

   call DelAllZero                           ;调用子程序对数组进行“删除”和”压缩“处理

   mov ah,4ch
   int 21h 

DelAllZero proc near
    push ax                                    ;保护各个使用到的寄存器
    push bx
    push si
    push di
  
    mov si,0                                    ;SI 存储当前数组中第一个为 0 的位置，这里的第一个并不仅仅指数组开始时的，它还指删除一个或几个 0 后的数组时的第一个为 0 的位置
    mov di,0                                    ;DI 用来遍历数组中的每一个元素
    mov ax, len                                ;AX 表示数组的长度,这个函数有两个数组：数组首地址和数组长度
    add ax,len                                  ;(AX)=(AX)+LEN，之所以要再加一次 LEN，因为数组元素为字单元，每次下标 SI 要加 2 才能取下一个元素，因为下标 SI 和 AX 的比较结果为循环退出的条件。为了方便，决定要将 AX 内容加倍
    sub ax,2                                     ;AX 内容减 2，表示 SI 只要扫描到数组倒数第二个元素即可
s4:
    cmp si,ax　　　　　　　　　　　　;（SI）＞＝（AX）时表示检测完毕，应该退出循环
    jnc exit1                                      ;不再满足条件时退出循环
    cmp word ptr iarray[si],0            ;检测 SI 指向的元素是否为 0，如果为 0 则 SI 加 2 检测下一个元素
    jz s5

    add si,2                                      ;SI 加 2 指向下一个元素
    add di,2                                      ;DI 加 2 指向下一个元素，直到指向首个为 0 的元素
    jmp s4
s5:
    add si,2　　　　　　　　　　　　　;移动 SI 检测下一个元素
    cmp word ptr iarray[si],0　　　　  ;如果非 0，则将该元素移至 DI 指向的位置，覆盖前面为 0 的元素
    jz s4　　　　　　　　　　　　　　;为 0，返回继续检测

    mov bx,iarray[si]　　　　　　　　;将 SI 指向的元素移至 DI 指向的位置，即覆盖前面的 0
    mov iarray[di],bx               
    mov word ptr iarray[si],0　　　　  ;移动之后将 SI 指向的元素清除为 0
    add di,2                                         ;DI 加 2 为指向被覆盖后的元素的后一个位置
    jmp s4　　　　　　　　　　　　　　  ;返回继续检测

exit1:
    pop di                                             ;恢复各个寄存器
    pop si
    pop bx
    pop ax
    ret　　　　　　　　　　　　　　　　;返回
DelAllZero endp

code ends
end start
```

总结：好久没做课题了，这两天终于做了两个课题。虽然仅仅是一点点小成就而已，但还是蛮开心的。

一二张图片为程序 1 的运行结果，三四五为程序 2 的程序结果，六为程序 3 的运行结果。

一为加载程序到内存中的源码，二为运行后的，其中 `00006H` 为 `C`，即 `11B` 个数，`CCFFH` 为 `X`。

三为代码，四为数据段的 100 个字单元，以及 `0064H` 为数组长度；五为删除、压缩后的结果，只是有点模糊。

六为运行后的结果，可以看出 `(X)=0003H` `str="MOONMOONMOONF$',`前面的 `03` 为 `str` 中“`MOON`”的个数。
