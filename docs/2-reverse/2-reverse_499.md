

# 反 C 代码的总结

> 原文：[`www.pediy.com/kssd/pediy10/88331.html`](https://www.pediy.com/kssd/pediy10/88331.html)

学\`\`算法\`\`有段时间了\`\`  
该是对所学的\`\`小结一下了~  
下面是我\`对 C 语言的一些简单结构的反汇编学习的小结~  
不对之处\`望指出~相互学习~!![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")  
for 循环:  
  
mov 进行初始化。jmp 跳过循环变量改变代码。cmp 实现条件判断，jge 根据条件跳转。  
用 jmp 回到循环改变代码进行下一次循环  
mov <循环变量>,<初始值> ;给循环变量赋初值  
jmp B ;跳到第一次循环处  
  
A: (改动循环变量) ;修改循环变量。  
…  
B: cmp <循环变量>,<限制变量> ;检查循环条件  
jgp 跳出循环  
(循环体)  
…  
jmp A ;跳回去修改循环变量  
\-------------------------------------------  
do --- while 循环:  
d  
o 循环就是一个简单的条件跳转回去  
  
cmp <循环变量>,<限制变量>  
  
  
jl <循环开始点>  
  
例:  
do {  
c = c+i;  
00411A55 mov eax,dword ptr \[c\]  
00411A58 add eax,dword ptr \[i\]  
00411A5B mov dword ptr \[c\],eax  
} while(c< 100);  
00411A5E cmp dword ptr \[c\],64h  
00411A62 jl myfunction+35h (411A55h)  
return c;  
\-------------------------------------------  
while 循环:  
  
while 除了开始的时候判断循环条件之外，后面还必须有一条无条件跳转回到循环开始的地方，  
共用三条指令实现  
  
A: cmp <循环变量>,<限制变量>  
jge B  
(循环体)  
…  
jmp A  
B: (循环结束)  
例:  
while(c<100){  
00411A55 cmp dword ptr \[c\],64h  
00411A59 jge myfunction+46h (411A66h)  
c = c+i;  
00411A5B mov eax,dword ptr \[c\]  
00411A5E add eax,dword ptr \[i\]  
00411A61 mov dword ptr \[c\],eax  
}  
00411A64 jmp myfunction+35h (411A55h)  
return c  
\-----------------------------------  
if 语句:  
  
if 判断都是使用 cmp 再加上条件跳转指令。对于 if( A && B)的情况，一般都是使用否决法。如果 A   
  
不成立，立刻跳下一个分支。  
依次，如果 B 不成立，同样跳下一分支。:  
  
cmp <条件>  
  
jle <下一个分支>  
  
\----------------------------------  
switch 结构的反汇编情况  
  
条件分支中，有比较特殊的情况是 switch.switch 的特点是有多个判断。因为 swtich 显然不用判断  
  
大于小于，所以都是 je,分别跳  
到每个 case 处。最后一个是无条件跳转，直接跳到 default 处。以下的代码:  
switch(c)  
{  
case 0:  
printf(“c>0”);  
case 1:  
{  
printf(“c>10 && c<100”);  
break;  
}  
default:  
printf(“c>10 && c<100”);  
}  
反汇编的结果是:  
switch(c)  
00411A66 mov eax,dword ptr \[c\]  
00411A69 mov dword ptr \[ebp-0E8h\],eax  
00411A6F cmp dword ptr \[ebp-0E8h\],0  
00411A76 je myfunction+63h (411A83h)  
00411A78 cmp dword ptr \[ebp-0E8h\],1  
00411A7F je myfunction+70h (411A90h)  
00411A81 jmp myfunction+7Fh (411A9Fh)  
{  
…  
显然是比较 c 是否是 0，1，这两个数字。至于先把 c 移动到 ebp-0E8h 这个地址，然后再比较，这是  
  
调试版本编译的特点。可能是  
为了防止直接操作堆栈而导致堆栈破坏？最后一条直接跳转到 default 处。如果没有 default，就跳  
  
到 swtich 之外。  
case 0:  
printf(“c>0”);  
00411A83 push offset string “c>0” (4240DCh)  
00411A88 call @ILT+1300(\_printf) (411519h)  
00411A8D add esp,4  
case 1:  
{  
printf(“c>10 && c<100”);  
00411A90 push offset string “c>10 && c<100” (424288h)  
00411A95 call @ILT+1300(\_printf) (411519h)  
00411A9A add esp,4  
break;  
00411A9D jmp myfunction+8Ch (411AACh)  
}  
default:  
printf(“c>10 && c<100”);  
00411A9F push offset string “c>10 && c<100” (424288h)  
00411AA4 call @ILT+1300(\_printf) (411519h)  
00411AA9 add esp,4  
}  
至于 case 和 default 都非常简单。如果有 break,则会增加一个无条件跳转。没有 break 的情况下，没有任何循环控制代码  
  
而剩下的\`就是  函数的调用\`~  
 和\`\`数组\`结构体了\`\`  
 这就是偶学  学习反 C 代码的一些小总结~!  
![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")