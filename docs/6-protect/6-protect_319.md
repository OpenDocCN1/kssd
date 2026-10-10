

# 一个代码混淆器的思路

> 原文：[`www.pediy.com/kssd/pediy12/129018.html`](https://www.pediy.com/kssd/pediy12/129018.html)

首先，反汇编 PE 的.text 节，结合重定向表的数据，分析获得指令之间的跳转、调用等引用关系。  
然后，就可以进行混淆操作了。  
（1），用混淆引擎，将指令变换执行方式，但意义不变。  
（如 inc xxx ==> add xxx , 1）  
（2），将一个完整的指令段拆分成多个段，然后再指令结构体中建立新的引用关系。  
（如：  
a  
b  
c   
  
\==>  
  
jmp tag1  
tag2:  
b  
c  
jmp tag3  
tag1:  
a  
jmp tag2  
tag3:  
接下来的指令）  
（3），生成新的可执行程序。  
  
不知道这个思路是不是可行，望各大牛予以指点。谢谢！ 

加入大量奇怪的代码。  
  
例如： Push Edx 变形为....  
  

引用: push            ebx   
mov             ebx,esp   
push            ebp   
mov             ebp,00000004   
add              ebx,ebp   
pop              ebp   
sub              ebx,00000004   
xchg             ebx,\[esp\] <- here   
pop              esp   
mov             \[esp\],edx