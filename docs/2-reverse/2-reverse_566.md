

# 用汇编来看 C 语言的简单实现

> 原文：[`www.pediy.com/kssd/pediy11/117424.html`](https://www.pediy.com/kssd/pediy11/117424.html)

好久没更新日志了,一是学习的时间实在有限.二是多是走马观花的看书.今天闲下来,写篇技术文章吧.反正无聊.  
先来复习几个相关的 ASM 指令.  
  
1.堆栈相关指令  
  
push : 把一个 32 位的操作数压入堆栈中.(操作导致 esp 被减 4)  
  
pop : 相反,esp 加 4，一个数据出栈.  
  
sub : 减法.第一个参数是被减数所在的寄存器;第二个参数是减数(对应的还有 add 指令)  
  
add : 加法.  
  
ret :返回.(相当于跳转回调用函数的地方)  
  
call : 调用函数(ret 跳转到 call 的下一条指令)  
  
2.数据传送指令  
  
mov : 数据移动.第一个参数是目的,第二个参数是来源.(C 语言中相当于赋值运算符)  
  
xor : 异或.  
  
lea : 取得地址(第二个参数)后放入到前面的寄存器(第一个参数)中.  
  
比如 : lea edi ,\[ebp - 0cch\] (\[\]括号表示存储器,ebp-0cch 这个地址所指的存储器内容)  
  
把 ebp-0cch 放入 edi 中.等同于 mov edi ebp-0cch 但这样的方法是错的.  
  
我们知道,mov 不支持后一个数写成寄存器减去个数,但是 lea 支持.  
  
3.跳转与比较指令  
  
jmp : 无条件跳转.  
  
jp : 大于的时候跳转  
  
jl : 小于的时候跳转  
  
jge : 大于等于的时候跳转  
  
cmp : 比较 .是 jg jl jge 之类的条件跳转指令的执行条件.  
  
  
.进入实例  
  
// windows 编程.cpp : 定义控制台应用程序的入口点。  
  
//  
  
#include "stdafx.h"  
  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
  
{  
  
return 0;  
  
}  
  
void myfunction(int a,int b)  
  
{  
  
int c = a + b;  
  
}  
  
(本人用的是 VS2008 测试.)  
  
在 return 0 处.  
  
int c = a + b 处 设置 2 个断点.  
  
然后进入调试,窗口,反汇编.得到以下代码:  
  
  
#include "stdafx.h"  
  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
  
{  
  
00CF1370 push ebp   
  
00CF1371 mov ebp,esp ;把 esp,移动至 ebp 地址  
  
00CF1373 sub esp,0C0h   
  
00CF1379 push ebx ;保存 3 个寄存器 :ebx,esi,edi  
  
00CF137A push esi   
  
00CF137B push edi   
  
00CF137C lea edi,\[ebp-0C0h\];把 ebp-0c0h 地址内容移动到 edi 中  
  
00CF1382 mov ecx,30h   
  
00CF1387 mov eax,0CCCCCCCCh ;0cccccccch c 语言初始化地址  
  
00CF138C rep stos dword ptr es:\[edi\] ;串写入  
  
return 0;  
  
00CF138E xor eax,eax   
  
}  
  
00CF1390 pop edi 恢复 3 个寄存器 edi,esi,ebx  
  
00CF1391 pop esi   
  
00CF1392 pop ebx   
  
00CF1393 mov esp,ebp   
  
00CF1395 pop ebp   
  
00CF1396 ret   
  
  
  
void myfunction(int a,int b)  
  
{  
  
00CF14A0 push ebp ;保存 ebp,并把 esp 放入 ebp 中.此时 ebp 与 esp  
  
00CF14A1 mov ebp,esp ;同都是这次函数调用的栈顶.  
  
00CF14A3 sub esp,0CCh 把 esp 往上移动一个范围,等于在堆栈中放出一片新的空间用来存储局部变量  
  
00CF14A9 push ebx ;保存 3 个寄存器  
  
00CF14AA push esi   
  
00CF14AB push edi   
  
00CF14AC lea edi,\[ebp-0CCh\]   
  
00CF14B2 mov ecx,33h   
  
00CF14B7 mov eax,0CCCCCCCCh   
  
00CF14BC rep stos dword ptr es:\[edi\]   
  
int c = a + b;  
  
00CF14BE mov eax,dword ptr \[a\] ;把 a 的值放入 eax 这个寄存器中(把 a 压入堆栈)  
  
00CF14C1 add eax,dword ptr ;把 b 的值加到 eax 这个寄存器中(把 b 压入堆栈)  
  
00CF14C4 mov dword ptr \[c\],eax ;把寄存器中 a + b 的值移动至 c.  
  
}  
  
00CF14C7 pop edi   
  
00CF14C8 pop esi   
  
00CF14C9 pop ebx   
  
00CF14CA mov esp,ebp   
  
00CF14CC pop ebp   
  
00CF14CD ret   
  
  
现在,我大体的写出 C 语言循环的汇编指令  
  
(循环变量,常量均用 16 进制表示)  
  
for 循环  
  
mov <循环变量>,<初始值> ;给循环变量赋初值  
  
jmp B(地址) ;跳到第一次循环处  
  
A:(改动循环变量) ;修改循环变量  
  
.....  
  
B:cmp<循环变量>,<限制变量> ;检查循环条件  
  
jpe 跳出循环  
  
(循环体)  
  
...  
  
jmp A ; 跳回去修改循环变量  
  
while 循环   
  
A: cmp<循环变量>,<限制变量>  
  
jpe B  
  
(循环体)  
  
....  
  
jmp A  
  
B:(循环结束)  
  
  
这就实现了汇编语言相应和 C 语言之间的一些简单关系.  
  
好了.累死了..下次继续吧.洗了睡了  
  
相比 C 来说,汇编确实高效且快速.  
  
从以上的一个小程序的测试,我们可以看到.汇编对每个地址的操作.相比于纯粹的 C 语言.汇编更可能接触到计算机底层.  
  
我的建议是,多点旁内触及,这才是王道..