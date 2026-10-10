

# asm 的魅力（二）

> 原文：[`www.pediy.com/kssd/pediy10/97107.html`](https://www.pediy.com/kssd/pediy10/97107.html)

**Asm 的魅力**（二）  
Author:charme  
Data:2009.9.4  
Index:hi.baidu.com/charme000  
  
废话：  
上一篇大牛牛们都说是出力不讨好，呼呼！我还是一贯作风吧：一笑而过。  
做个简单的类比：  
开发黑客软件的人笑只会使用软件入侵的菜菜肤浅，如果认可这个挂点的话，那研究 asm 本身实质的人就应该嘲笑利用 asm 搞逆向和破解的人了。![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")  
  
而实际上这是几个不同的领域，所以没有什么吃力不讨好的说法。  
会写软件的人必然懂得入侵的原理，所以懂 asm 的人逆向只是个时间问题。  
  
曾子曰：自作孽，不可活！  
如果为了凑篇精华，拼凑代码科普出来，基本等于没有创造！！！我不干那样的事情！！  
  
还是相信女王大牛的教诲：这个世界永远注重的是创造。![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")  
  
正文：  
程序语言中的结构体是常见的访问内存中存储数据的一种方式，有点不必说了，大家接触过结构的都明白。那么 asm 中的结构用的当然也比较多。  
  
结构可以嵌套，但是初学者必然的会有一些迷惑。  
  
**（一）明白结构访问的本质**  
首先看一个 C 语言中的结构使用：  
代码:

```
Struct A
{
   Int charme;
}\*a;

Struct B
{
  A c;
}\*b

```

那么我们常常见到这样的访问方法：b->c.charme  
那么->是个什么东西？（到现在我都不知道这叫个什么操作符，呼呼）为什么我们不写成这样的：b.c.charme？不就是访问成员嘛！！！到这里就是重点了，->操作符的本质是什么？  
  
再看一段程序：  
  
代码:

```
int  \_ClientLogin( LPVOID \_lpStruct)
{
  TranSocket  \*stSock;
  ...
  int k = stSock->dwIndex;    //相当于取 stSock.dwIndex 赋值给 k
  ...
  return (0);
}

```

这是含笑大哥程序里摘出来的一段，重点注释部分。  
看完这个我们可能就明白了，stSock 只是一个指针，指向 TranSocket 结构的一个指针。这样做只是为 TranSocket 结构预留了一个空间，方便以后的初始化。  
这就是->的本质！！！！！！！！！！！！！！！！！！这个必须搞清楚的，搞清楚了这个才能理解怎么嵌套。  
  
**（二）asm 中的结构嵌套的错误**  
那么就之前的那个结构嵌套。很多人可能会这样写（我之前也是这么写的，呼呼）：  
代码:

```
.386
.model flat,stdcall
option casemap:none

include windows.inc
include kernel32.inc
include user32.inc
includelib user32.lib
includelib kernel32.lib

aa struct
   charme dd ?
aa ends

bb struct
    charme1 dd ?
    cc aa <>
    
bb ends

.data?
p2 bb <>  ;;;;;;;注意
p1 aa <>  ;;;;;;;;注意

szEAX      db 8 dup('0'),0

strr1 dd ? 
.data
strr db "charme",0
.code
ExchangCode  proc w32BitCode:DWORD,lpCode:DWORD
    push  ebx
    push  ecx
    push  esi
    mov  esi,lpCode
    mov  eax,w32BitCode
    mov  ecx,8
    @@nextchar:  push  ecx
    rol  eax,4
    mov  ebx,eax
    and  ebx,0000000Fh
    cmp  bl,09
    jle  @@char0\_9
    add  bl,07h
    @@char0\_9:  add  bl,30h
    mov  BYTE PTR \[esi\],bl
    inc  esi
    pop  ecx
    loop  @@nextchar
    pop  esi
    pop  ecx
    pop  ebx
    ret
ExchangCode  endp 
main proc
    mov p1.charme,20h
    mov ebx, p2.cc.charme
        
    push offset szEAX
    push ebx
    call ExchangCode
    
    push 40h
    push offset strr
    push offset szEAX
    push 0
    call MessageBox
    
    
    ;push 03e8h
    ;call Sleep
    
    push 0
    call ExitProcess
main endp
end main

```

asm 中没有->这样的操作符，那怎么实现 b->c.charme 这样的访问呢？只好是像上面这样写了  
结果显示 p2.cc.charme 的值是 0，意图来看应该是显示 20h。  
错误在哪里？  
OD 调试看看，可以很清楚的看到,内存空间中：  
P1.charme--------------------------0400xxxxh  
Charme1  
P2.cc.charme----------------------0400yyyyh  
根本就是两个变量，怎么可能访问到呢，，照这样看来我们这样修改下：  
mov ebx, p2.cc.charme+4，就输出 20h，但实际上这只是投机，只是跳过了 charme1 的空间，访问了下 p1.charme，一点也不能通用。  
  
**（三）重写 asm 结构嵌套**  
之所以出现上面的错误，实际上还是因为没有真正的理解了结构访问成员的本质。Asm 的魅力在于最大限度的接近人的思维，所以他很灵活，但是灵活的话就需要你掌握本质。否则灵活的旁边就是混乱。（我也搞不清楚这俩词到底有啥区别，呼呼）  
  
那么在来看看  
代码:

```
.386
.model flat,stdcall
option casemap:none

include D:\\MASMPlus\\Include\\windows.inc
include D:\\MASMPlus\\Include\\kernel32.inc
include D:\\MASMPlus\\Include\\user32.inc
includelib D:\\MASMPlus\\Lib\\kernel32.lib
includelib D:\\MASMPlus\\Lib\\user32.lib


X struct
  a  dd  10h
  b  db  10 dup(?)
X ends

Y struct
  cc  dd  ?
  d  dd  ?
  e  db  ?
Y ends

M struct
  p  X  <>
  q  Y  <>
M ends

.data
stMy  M  <>

szEAX      db 8 dup('0'),0
szTitle db "charme",0

.code
ExchangCode  proc w32BitCode:DWORD,lpCode:DWORD
    push  ebx
    push  ecx
    push  esi
    mov  esi,lpCode
    mov  eax,w32BitCode
    mov  ecx,8
    @@nextchar:  push  ecx
    rol  eax,4
    mov  ebx,eax
    and  ebx,0000000Fh
    cmp  bl,09
    jle  @@char0\_9
    add  bl,07h
    @@char0\_9:  add  bl,30h
    mov  BYTE PTR \[esi\],bl
    inc  esi
    pop  ecx
    loop  @@nextchar
    pop  esi
    pop  ecx
    pop  ebx
    ret
ExchangCode  endp
start:
  ;;正确做法一
  ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
  ;lea   ebx, stMy ;;指向 M 结构的一个指针

  ;mov eax,(M ptr \[ebx\]).p.a;;必须加上 M ptr 因为 C 里面我们也看到了，实际上这个指向 M 结构的指针已 idngyao 确定是某个结构的，，因为你 lea   ebx, stMy 只是初始化了一个指针，并没有给你要初始化的结构分配空间
  ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
  
  ;;正确在做法二
  ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
  ;;mov ebx,sizeof M
  ;;mov eax,stMy\[ebx\].p.a
  ;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
  
  ;;正确做法三
  lea esi,stMy.p
  mov ebx,(X ptr \[esi\]).a
  
  
  
  push offset szEAX
  push ebx
 
  call ExchangCode
  
  push 40h
  push offset szTitle
  push offset szEAX
  push 0
  call MessageBox
  
  ret

end start

```

上面我用了三种方法，实际上本质是一样的。  
  
看看方法一和方法三，你可以对比出一个规律来：就是我们确定了指针，一定要明确他是要初始化一个什么样的结构。  
  
方法二是可扩展的。  
比如有一个叫做 AA 的结构,先假设他有两个成员  
我们定义一个结构数组：aa AA 3 dup(<0,0>)  
那么我们可以循环赋值，下面给一个示例：  
  
代码:

```
AA struct
   Ch1 dd ?
Ch2 dd ?
AA ends

.data
Align word
aa AA 3 dup(<0,0>)

.code
Start:
Mov edi,0
Mov ecx,3
Mov eax,1
Loopp:
Mov (AA ptr aa\[edi\]).ch1,eax
Mov (AA ptr aa\[edi\]).ch2,eax
Add edi,TYPE AA  ;;或者是 SIZEOF AA，这里获得的大小就是结构数组里单个数组的大小
Inc eax
Loop loopp

Push 0
Call ExitProcess
End start

```

当然我们也可以每个 ch1 ch2 初始化不同的值，含笑大哥提到了有两种方法：  
接受键盘输入和预设初值为一个数组  
  
方法一和 C 语言里的访问形式就很接近了，但是很多人这样写：  
代码:

```
lea   ebx, stMy

mov eax, \[ebx\].p.a

```

这样是不对的，，，\[EBX\]并不足以确定一个他属于什么样的结构，当然结构里的成员无从得知了。  
  
  
不管怎么样吧！asm 里使用结构对程序的组织和效率是很有帮助的。。  
  
基本就总结到这里吧！希望对大家有帮助！！不要小看这些小小的知识，如果你真的认真的看到这里了，理解了，运用了，逆向算什么，破解算什么，只要你足够聪明，无非变通的组织思维的过程！！！  
  
我也不是为了说大话，呼呼！！！！希望大家进步，大四了，每天一节课。写点东西。留个回忆吧！！![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")