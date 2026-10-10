

# Thinking in ASM (1)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-37.htm`](https://www.pediy.com/kssd/pediy07/pediy7-37.htm)

Thinking in ASM  
           
       By RoBa  
  
  
Introduction   
  
呵呵，这个题目够吓人吧。别激动，不是什么经典著作又出来了，是我在这儿瞎侃呢。（我也没有系统地学习过计算机专业课，下面的议论都是大白话，高手莫笑。）  
  
前些天买了本《程序员 2004 年合订本》，忽然发现热门语言又变成了 Python 什么的，可怜我还准备好好学学 Java 呢，看来是永远跟不上形势了。郁闷之际，忽然想起候捷老师的一句话来“勿在浮沙筑高台”，自己基础还没打好，还是先别去赶时髦了，就算我能用几十种语言写出个“Hello ,world!”又有什么用？于是静下心来学习 C/C++和 WindowsSDK 编程，翻开 K&R、Stroustrup、Petzold 等等那些大师们的经典著作细读，常常在读到某一段时突然有醍醐灌顶的感觉，把自己以前东拼西凑的知识梳理得清楚了一些。不过可能是 Crack 弄得多了，干什么事老想寻根究底，于是就有了这篇文章。  
  
大家都知道，不管你是什么语言，不管你是面向过程也好，面向对象也好，或者以后又有面向什么也好，我们的 CPU 不懂得类和对象，它只知道从主存中取出机器码然后一条一条执行。（至少当前是这样）  
  
现在的程序语言越来越向人的方向靠拢，程序设计的思路越来越像人们平时自然的思考，也许在不久的将来我们就能直接用自然语言写程序了![](img/0cf410da109ac47387dd713fcee2cd79.jpg) 但是，到目前为止 CPU 仍然是只认识机器码的，高级语言只不过把人工作转移给了相应的编译器而己，虽然方便，但给人一种“知其然不知其所以然”的感觉，另外，编译器自动产生的代码毕竟不能随意控制，在某些特殊情况下我们必须亲自动手对机器指令开刀。  
  
这就好比你平时可以悠闲地在 KFC 啃着做好的炸鸡，不用管鸡是怎么做的，掏钱就行了，但当你在荒岛上饿了三天，突然发现面前一只活生生的大肥鸡时，你能再请麦当劳帮你做好吗？所以说，我们要学会自己来。毕竟，我们的身体最终需要的只是蛋白质和葡萄糖，正如我们的 CPU 最终需要的只是 0 和 1 一样。  
  
那么，让我们进入一片机器代码的蛮荒之地，看看那些高级语言的高级特性是如何在低级代码上实现的。Just thinking in ASM code!  
  
大家肯定已经哈哈大笑了，这种东西难道是一个人能够完成的？当然不行，所以我只把自己在学习过程中发现的一些有意思的小地方用汇编语言分析一下，作抛砖引玉之用，欢迎各位批评指正，多多补充。现在很流行写读书笔记，这也算一种吧。 

Thinking in ASM  
  
1.关于 C 中的函数调用（call by value）  
  
值传递，一个太老的话题了，我只不过是把多数编程书上讲的用汇编语言展开了，希望您能耐着性子看下去。  
  
先看一个例子：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> #include <stdio.h>
> 
> void swap(int a,int b)
> {
>   int temp;
>   temp=a;
>   a=b;
>   b=temp;
> }
> 
> int main()
> {
>   int a,b;
>   a=1; b=2;
>   swap(a,b);    /\* 交换了吗? \*/
>   printf("a=%d b=%d\\n",a,b);
>   return 0;
> }
> 
> * * *
> 
> 
> ```

  
  
我们设计了一个函数 swap(a,b)，希望它能对给定的两数交换次序，但实际输出为"a=1 b=2"，显然没有成功。刚学编程的初学者可能被这个问题困扰，为什么在函数中对参数的改变不会影响到原参数的实际值？  
  
下面我们用汇编来看一下。（注：这里我用的是 VC6，在调试时用 Disassembly\[Alt+8\]得到的。如果用默认的 Debug 设置，产生的多余代码太多，但 Release 设置又不含调试信息，只好自己定义了一种方式，把优化方案改为 Default，就可以带上调试信息了，代码也与 Release 版的差别不大。如果哪位有更好的方法请告诉我。）  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 1:    #include <stdio.h>
> 2:
> 3:    void swap(int a,int b)
> 4:    {
> 00401000   push        ebp      ;把当前 ebp 入栈，保留起来
> 00401001   mov         ebp,esp      ;esp 指向栈顶，这句执行完后下面对变量的访问均由 ebp 指示
> 00401003   sub         esp,44h      ;为函数内局部变量留出空间
> 00401006   push        ebx      ;保护现场
> 00401007   push        esi      ;保护现场
> 00401008   push        edi      ;保护现场
> 5:        int temp;
> 6:        temp=a;
> 00401009   mov         eax,dword ptr \[ebp+8\]  ;\[ebp+8\]值为第一个参数 a
> 0040100C   mov         dword ptr \[ebp-4\],eax  ;通过 eax 把\[ebp+8\]的值给了局部变量\[ebp-4\]
> 7:        a=b;
> 0040100F   mov         ecx,dword ptr \[ebp+0Ch\]  ;\[ebp+c\]值为第二个参数 b
> 00401012   mov         dword ptr \[ebp+8\],ecx  ;通过 ecx 把\[ebp+c\]的值给了\[ebp+8\]
> 8:        b=temp;
> 00401015   mov         edx,dword ptr \[ebp-4\]  ;\[ebp-4\]是局部变量，现在值为 a
> 00401018   mov         dword ptr \[ebp+0Ch\],edx  ;通过 edx 把\[ebp-4\]的值给了\[ebp+c\]
> 9:    }
> 0040101B   pop         edi      ;恢复现场
> 0040101C   pop         esi      ;恢复现场
> 0040101D   pop         ebx      ;恢复现场
> 0040101E   mov         esp,ebp      ;恢复 esp，这样\[ebp-xx\]的局部变量不再有效
> 00401020   pop         ebp      ;恢复第一行保留的 ebp
> 00401021   ret
> 10:
> 11:   int main()
> 12:   {
> 00401030   push        ebp
> 00401031   mov         ebp,esp
> 00401033   sub         esp,48h
> 00401036   push        ebx
> 00401037   push        esi
> 00401038   push        edi
> 13:       int a,b;
> 14:       a=1; b=2;
> 00401039   mov         dword ptr \[ebp-4\],1  ;这是变量 a
> 00401040   mov         dword ptr \[ebp-8\],2  ;这是变量 b
> 15:       swap(a,b);      /\* 交换了吗? \*/
> 00401047   mov         eax,dword ptr \[ebp-8\]  ;取出 b 的值
> 0040104A   push        eax      ;变量 b 入栈
> 0040104B   mov         ecx,dword ptr \[ebp-4\]  ;取出 a 的值
> 0040104E   push        ecx      ;变量 a 入栈
> 0040104F   call        swap (00401000)    ;调用 401000 处的 swap
> 00401054   add         esp,8      ;函数外平衡堆栈，结果 swap 函数内\[ebp+8\]\[ebp+c\]全无效
> 16:       printf("a=%d b=%d\\n",a,b);
> 00401057   mov         edx,dword ptr \[ebp-8\]
> 0040105A   push        edx
> 0040105B   mov         eax,dword ptr \[ebp-4\]
> 0040105E   push        eax
> 0040105F   push        offset string "a=%d b=%d\\n" (004060cc)
> 00401064   call        \_printf (00401075)
> 00401069   add         esp,0Ch
> 17:       return 0;
> 0040106C   xor         eax,eax
> 18:   }
> 0040106E   pop         edi
> 0040106F   pop         esi
> 00401070   pop         ebx
> 00401071   mov         esp,ebp
> 00401073   pop         ebp
> 00401074   ret
> 
> * * *
> 
> 
> ```

  
首先明确一下，esp 始终指向栈顶，在 C 中每个函数内都用 ebp 指针来指示传给它的参数和它自己的局部变量，不同的函数 ebp 值不同。在上面例子中 swap 函数的\[ebp+8\]\[ebp+C\]是参数 a,b，\[ebp-4\]是局部变量 temp，main 函数的\[ebp-8\]\[ebp-4\]是局部变量 a,b。  
  
可以看出，我们想交换的变量在 main 函数的局部变量\[ebp-8\]和\[ebp-4\]中，而我们调用 swap 之前先访问这两个地址取出两个值，然后把这两个值压入堆栈，接着一进函数马上把 main 函数的 ebp 保存起来，而用当前 esp 代替 ebp 在 swap 函数内指示变量，我们根本没有机会得到 main 函数中的 ebp 是多少，swap 只交换了它自己堆栈里两个参数的值，而当返回后平衡堆栈时 esp+8，这两个参数地址都无效了。main 函数的变量\[ebp-8\]\[ebp-4\]在整个过程中并没有被改动过，只是值被复制了一份传给函数 swap 而已，因此当然不会被交换，交换的只是两个临时的复制品。这就是 K&R 的《The C Programming Language》里所说的 call by value，即只传递参数的值，不传递参数的地址。  
  
最后写个 BT 的解决方案：![](img/710b5589904d0d2bf58a67748fca40e1.jpg)  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> void swap(int a,int b)
> {
>   int temp;
>   temp=a;
>   a=b;
>   b=temp;
> }
> 
> int main()
> {
>   int a,b;
>   a=1; b=2;
>   swap(a,b);    /\* 交换了吗? \*/
>   \_\_asm {
>     MOV EAX,\[ESP-8\]
>     MOV a,EAX
>     MOV EAX,\[ESP-4\]
>     MOV b,EAX
>   }      
>   printf("a=%d b=%d\\n",a,b);
>   return 0;
> }
> 
> * * *
> 
> 
> ```

  
呵呵，开个玩笑啦，这么写程序会让人发疯的。关于如何让函数返回多个值的解释，请关注下文。![](img/0cf410da109ac47387dd713fcee2cd79.jpg)  
  
第一篇写得有些无聊，因为大家肯定都很熟悉了，以后我好好学习，争取写一些复杂点的，比如 C++里的一些特性甚至分析 MFC 啦…… 

> 引用:
> 
> * * *
> 
> *最初由 乱乱 发布*  
> **工程->设置->c/c++-> gategory 选择 listing file  
> 下面选择 assembly with source code  
> 就可以得到一个 asm 文件 源代码和汇编代码对照  
> 而且这个 asm 文件可以用 ml 编译链接。 **
> 
> * * *

  
  
Thx very much!  
  
btw: 上面最后那个 BT 程序在 release 模式下结果错误，因为 RELEASE 很聪明，看到我那个 SWAP 其实什么也没干，于是 CALL 里面直接 RET 回来了，堆栈中没有相应的数据。 

就是 VB 下的结果确是正常的  
Function swap(a, b As Integer)  
Dim temp As Integer  
temp = a  
a = b  
b = temp  
End Function  
Private Sub Command1\_Click()  
Dim a, b As Integer  
a = 1  
b = 2  
Call swap(a, b)  
Print "a=" & a & " b=" & b  
End Sub 

> 引用:
> 
> * * *
> 
> *最初由 RoBa 发布*  
> **void swap(int a,int b)  
> {  
>   int temp;  
>   temp=a;  
>   a=b;  
>   b=temp;  
> }  
>   
> int main()  
> {  
>   int a,b;  
>   a=1; b=2;  
>   swap(a,b);    /\* 交换了吗? \*/  
>   \_\_asm {  
>     MOV EAX,\[ESP-8\]  
>     MOV a,EAX  
>     MOV EAX,\[ESP-4\]  
>     MOV b,EAX  
>   }        
>   printf("a=%d b=%d\\n",a,b);  
>   return 0;  
> }  
> **
> 
> * * *

  
  
  
用不着这么麻烦  
  
void swap(int &a,int &b)  
{  
  int temp;  
  temp=a;  
  a=b;  
  b=temp;  
}  
  
int main()  
{  
  int a,b;  
  a=1; b=2;  
  swap(a,b);    /\* 交换了吗? \*/  
   
  printf("a=%d b=%d\\n",a,b);  
  return 0;  
}  
传址就可以了 

> 引用:
> 
> * * *
> 
> *最初由 larblue 发布*  
> **  
>   
> 用不着这么麻烦  
>   
> void swap(int &a,int &b)  
> {  
>   int temp;  
>   temp=a;  
>   a=b;  
>   b=temp;  
> }  
>   
> int main()  
> {  
>   int a,b;  
>   a=1; b=2;  
>   swap(a,b);    /\* 交换了吗? \*/  
>    
>   printf("a=%d b=%d\\n",a,b);  
>   return 0;  
> }  
>   
> ........ **
> 
> * * *

  
  
正确的写法  
  
#include <stdio.h>  
  
void swap(int \*a,int \*b)  
{  
  int temp;  
  temp=\*a;  
  \*a=\*b;  
  \*b=temp;  
}  
  
int main()  
{  
  int a,b;  
  a=1; b=2;  
  swap(&a,&b);     
   
  printf("a=%d b=%d\\n",a,b);  
  return 0;  
} 

> 引用:
> 
> * * *
> 
> *最初由 riijj 发布*  
> **我当然知道 ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)  ~~  roba 写那个 bt 方案是讲学用的  
>   
> 我只是修正一下楼上那个 C 代码的 pointer 错误用法，以免一些兄弟误学了 **
> 
> * * *

  
  
呵呵  
我的没有错你的也没有错  
  
  
1、传值和传址：  
  
lippman 在说明这个问题的时候用了一个探索的过程，让初学者没有一点障碍的被领进了这个问题。  
  
什么是形参？什么是实参呢？简单的说，编写函数的时候说明的参数就是形参，在调用函数的时候的参数就是实参。  
  
当调用一个函数的时候，会在内存中建立一块特殊的区域，叫程序栈。他提供没个函数参数的储存空间。  
  
在默认情况下，参数都会被复制一份传入程序栈，这就是所谓的传值，架设给一个数组排序，用传值的方式是不会改变原有数列的，这是就要用到传址。在参数前面加一个“&”即可。  
  
什么时候该用到传址？当希望对传入的对象修改时，或者是如果传入参数对象过于庞大，用到传址就会大大提高程序的效率。  
  
当然也可以用指针来传递参数，其实也是一样的，因为指针的本质就是地址。  
  
引自重读《Essential C++ 读书笔记 2》 by sssa2000  
  
兄弟可以编译一下试试 

兄弟，我解释一下    
  
Roba 写的是 “ 关于 C 中的函数调用 “ ，那是 C 代码，不是 C++  
  
你这种写法只是 C++ 正确，在 C 是错误的  
  
  
![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

 4dd466d0e1e0cc123afbe504da2462ca