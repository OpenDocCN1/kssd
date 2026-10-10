

# 【文章标题】: 逆向 C++之二 变量和常量

> 原文：[`www.pediy.com/kssd/pediy10/78246.html`](https://www.pediy.com/kssd/pediy10/78246.html)

【文章标题】: 逆向 C++之二 变量和常量  
【文章作者】: kanghtta  
【作者邮箱】: kanghtta@hotmail.com  
【作者主页】: http://kanghtta.cublog.cn  
\--------------------------------------------------------------------------------  
【详细过程】  
        大家好，其实是 china unix 的网友让写点 c++的内容，答应了好长时间，但由于复习考研，一直也没抽什么时间出来；  
   所以打算把午觉时间抽出来总结总结； 还是为了学习的目的，所以有什么不是之处，希望大家帮忙指正，也希望和大家一起  
  进步 ； 废话不说，进入正题：　  
    
  　　　常量和变量  
  　定义：　常量，是一种其值不被改变的量，　即在程序执行过程中不改变其本身的值；  
  　　　　　变量，相对而言，变量就是在程序执行过程中其值可以被改变的量，这样说也许没什么实际意义，你也可以  
  这样认为：　当你在程序中定义一个变量时，编译器把变量名和一个内存单元关联起来，任何时候的变量值就是和它关联  
  的内存单元中存储的那个值，因此，在计算机中变量这个术语表示可被用来存储内容，并且其内容可被取出和处理的内存单元；  
  变量名必须符合标识符的命名规则；  
  　　　　　对编译器而言，变量的声明意味着一个变量对象用来存储某种数据类型的数据；  
  　　　常量和变量的声明：  
  　常量定义的一般形式：　　const  type  CONSTANT\_NAME = expression  
    变量的一般形式：　　　　type   variable\_name  ;  
                            type   variable\_name = initializer-expression;   
     总的来说，无论是常量或者是变量的定义都包含如下三部分：　  
  　type         类型   
    name         名字   
    expression   值     
    我们主要讨论变量的三要素:   
    变量的类型分为 存储类型 和 数据类型两种 ，  
    
  存储类型  指的是变量存放在计算机中的位置不同，它和变量的作用域 ，  
  寿命有关，变量的作用域是指变量的有效范围 ，也叫可见性，变量的寿命是指变量的生存期，也叫存在性；  
  存储类型有四种 ：  
   分别是  
  1） 自动类    auto        说明在函数体内和分程序中，auto 类型说明符可省略不写；         
  2） 寄存器类  register    必须定义在函数体内或分程序中；  
  3） 外部类    extern      定义在函数体外，也叫全局变量  
  4） 静态类    static      分外部静态类和内部静态类两种，区别体现在作用域和寿命上；  
    
  数据类型，包含基本数据类型和构造数据类型； 是现实生活中一种对象的模拟或者是对对象某一属性的模拟；  
    
   下面我们来看一个例子： （ 为了方便大家学习，我用一下加密解密三上的例子）   
    
  问题描述 ： 例子中定义了一个求两个数的和的函数，并在 main（）函数中调用它； 这里我们稍做修改，我们加入一个求日工资的功能；  
  以便大家理解上面的内容；  
    
  

代码:

```
#include<iostream>

using namespace std;

int    AddTwoNumber(int x,register int y);

int main()
{
  const  double  HOURLY\_WAGE = 30;     //常量定于
  double  hours = 8;                        //变量定义
  static  double  pay;
  pay = hours \* HOURLY\_WAGE;

    int          a = 10,
  register  int    b = 20;

  extern  int z;
  
  z = AddTwoNumber(a,b);
  
  cout<<z<<endl;
  return 0;
}

int    z;

int    AddTwoNumber(int x,register int y)
{
  return x+y;
}
  

```

  
  
  要找 main 函数模块，找 getcommand 函数下面不远处 3 个 push 的下一个 call 就是了；上一节中我已经说过，这里不在重复；    
  00421A3B  |.  E8 709E0000   call    0042B8B0  
  00421A40  |.  FF15 9CE14700 call    dword ptr \[<&KERNEL32.GetCommand>; \[GetCommandLineA  
  00421A46  |.  A3 44DF4700   mov     dword ptr \[47DF44\], eax  
  00421A4B  |.  E8 409C0000   call    0042B690  
  00421A50  |.  A3 98C34700   mov     dword ptr \[47C398\], eax  
  00421A55  |.  E8 26970000   call    0042B180  
  00421A5A  |.  E8 D1950000   call    0042B030  
  00421A5F  |.  E8 DC400000   call    00425B40  
  00421A64  |.  8B0D F0C34700 mov     ecx, dword ptr \[47C3F0\]  
  00421A6A  |.  890D F4C34700 mov     dword ptr \[47C3F4\], ecx  
  00421A70  |.  8B15 F0C34700 mov     edx, dword ptr \[47C3F0\]  
  00421A76  |.  52            push    edx  
  00421A77  |.  A1 E8C34700   mov     eax, dword ptr \[47C3E8\]  
  00421A7C  |.  50            push    eax  
  00421A7D  |.  8B0D E4C34700 mov     ecx, dword ptr \[47C3E4\]  
  00421A83  |.  51            push    ecx  
  00421A84  |.  E8 9DF7FDFF   call    00401226                       //main 函数入口地址   
  
  F7 跟进，有一个 jmp ，F7 见到如下代码：  
  
  00401560   > \\55            push    ebp  
  00401561   .  8BEC          mov     ebp, esp  
  00401563   .  83EC 58       sub     esp, 58  
  00401566   .  53            push    ebx  
  00401567   .  56            push    esi  
  00401568   .  57            push    edi  
  00401569   .  8D7D A8       lea     edi, dword ptr \[ebp-58\]  
  0040156C   .  B9 16000000   mov     ecx, 16  
  00401571   .  B8 CCCCCCCC   mov     eax, CCCCCCCC  
  00401576   .  F3:AB         rep     stos dword ptr es:\[edi\]  
  00401578   .  C745 F8 00000>mov     dword ptr \[ebp-8\], 0  
  0040157F   .  C745 FC 00003>mov     dword ptr \[ebp-4\], 403E0000     ;双精度 double 占 8 个字节，故初始化时用条 mov 指令  
  00401586   .  C745 F0 00000>mov     dword ptr \[ebp-10\], 0  
  0040158D   .  C745 F4 00002>mov     dword ptr \[ebp-C\], 40200000     ；double  hours = 8;  
  00401594   .  DD45 F0       fld     qword ptr \[ebp-10\]  
  00401597   .  DC4D F8       fmul    qword ptr \[ebp-8\]  
  0040159A   .  DD1D F0BD4700 fstp    qword ptr \[47BDF0\]              ；47BDF0 存的是 static 静态局部变量 pay  
  004015A0   .  C745 EC 0A000>mov     dword ptr \[ebp-14\], 0A     ; int a = 10;  
  004015A7   .  C745 E8 14000>mov     dword ptr \[ebp-18\], 14     ;register int  b = 20;  
  004015AE   .  8B45 E8       mov     eax, dword ptr \[ebp-18\]  
  004015B1   .  50            push    eax  
  004015B2   .  8B4D EC       mov     ecx, dword ptr \[ebp-14\]  
  004015B5   .  51            push    ecx  
  004015B6   .  E8 F3FBFFFF   call    004011AE                            //调用 AddTwoNumber 函数  
  004015BB   .  83C4 08       add     esp, 8  
  004015BE   .  A3 E8BD4700   mov     dword ptr \[47BDE8\], eax  
  004015C3   .  68 C8104000   push    004010C8  
  004015C8   .  8B15 E8BD4700 mov     edx, dword ptr \[47BDE8\]  
  004015CE   .  52            push    edx  
  004015CF   .  B9 A0BE4700   mov     ecx, 0047BEA0  
  004015D4   .  E8 26FBFFFF   call    004010FF  
  004015D9   .  8BC8          mov     ecx, eax  
  004015DB   .  E8 05FCFFFF   call    004011E5  
  004015E0   .  33C0          xor     eax, eax  
  004015E2   .  5F            pop     edi  
  004015E3   .  5E            pop     esi  
  004015E4   .  5B            pop     ebx  
  004015E5   .  83C4 58       add     esp, 58  
  004015E8   .  3BEC          cmp     ebp, esp  
  004015EA   .  E8 71F00100   call    00420660  
  004015EF   .  8BE5          mov     esp, ebp  
  004015F1   .  5D            pop     ebp  
  004015F2   .  C3            retn  
  
  分析： 我们跟进的时候，主要是看堆栈中局部变量的分布，故在堆栈窗口中把相关的代码拷贝如下：  
  
  EBP-58   > CCCCCCCC  
  EBP-54   > CCCCCCCC  
  EBP-50   > CCCCCCCC  
  EBP-4C   > CCCCCCCC  
  EBP-48   > CCCCCCCC  
  EBP-44   > CCCCCCCC  
  EBP-40   > CCCCCCCC  
  EBP-3C   > CCCCCCCC  
  EBP-38   > CCCCCCCC  
  EBP-34   > CCCCCCCC  
  EBP-30   > CCCCCCCC  
  EBP-2C   > CCCCCCCC  
  EBP-28   > CCCCCCCC  
  EBP-24   > CCCCCCCC  
  EBP-20   > CCCCCCCC  
  EBP-1C   > CCCCCCCC  
  EBP-18   > 00000014  
  EBP-14   > 0000000A  
  EBP-10   > 00000000  
  EBP-C    > 40200000  
  EBP-8    > 00000000  
  EBP-4    > 403E0000  
  EBP ==>  >/0012FFC0  
  EBP+4    >|00421A89  返回到 variable.<模块入口点>+0E9 来自 variable.00401226  
  
  
  调用 AddTwoNumber（）函数的代码：   
  004015B6   .  E8 F3FBFFFF   call    004011AE                            //调用 AddTwoNumber 函数  
  
  F7 跟进  
  
  00401620  /> \\55            push    ebp  
  00401621  |.  8BEC          mov     ebp, esp  
  00401623  |.  83EC 40       sub     esp, 40  
  00401626  |.  53            push    ebx  
  00401627  |.  56            push    esi  
  00401628  |.  57            push    edi  
  00401629  |.  8D7D C0       lea     edi, dword ptr \[ebp-40\]  
  0040162C  |.  B9 10000000   mov     ecx, 10  
  00401631  |.  B8 CCCCCCCC   mov     eax, CCCCCCCC  
  00401636  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
  00401638  |.  8B45 08       mov     eax, dword ptr \[ebp+8\]          //  int a = 10,  
  0040163B  |.  0345 0C       add     eax, dword ptr \[ebp+C\]          //return x+y;  
  0040163E  |.  5F            pop     edi                              ;  0012FF80  
  0040163F  |.  5E            pop     esi  
  00401640  |.  5B            pop     ebx  
  00401641  |.  8BE5          mov     esp, ebp  
  00401643  |.  5D            pop     ebp  
  00401644  \\.  C3            retn  
  
  
  004015BB   .  83C4 08       add     esp, 8  
  
  堆栈数据：   
  
  EBP ==>  >/0012FF80  
  EBP+4    >|004015BB  返回到 variable.004015BB 来自 variable.004011AE  
  EBP+8    >|0000000A                                                     ;int a = 10; 0xA;  
  EBP+C    >|00000014                                                     ;register int b = 20; 0x14;  
  EBP+10   >|7C930208  ntdll.7C930208  
  EBP+14   >|FFFFFFFF  
  
  
  我们看到，只要是局部变量，也就是在函数体内定义的变量，无论是什么样的存储类型和数据类型，都有一个内存地址和它相对应；  
  
  先看 double 型的数，双精度浮点数是单精度浮点数的扩充，总长 64 位，1 位符号位，11 位指数，52 位尾数；  
  单精度 32 位，最高 1 位 符号位，接下来 8 位存放指数+127，127 叫偏移值  
  最低 23 位用来表示尾数小数点右边的部分，左边一位不需要存储，因为它恒位一；  
  
  十进制 30 划为 2 进制数：   
  现转换为 16 进制数：  0x 001E；   二进制： 0000 0000 0001 1110 = 1.111\*2 的 4 次方   
  故 double 最高位符号位 0 指数位 ： 3FF+4 = 403 = 0100 0000 0011 尾数为： 11100000...总共 52 位；  
  
  我们看  
  00401578   .  C745 F8 00000>mov     dword ptr \[ebp-8\], 0  
  0040157F   .  C745 FC 00003>mov     dword ptr \[ebp-4\], 403E0000  
  
  403E0000   二进制为： 0 10000000011 1110 0000 0000 0000 0000 0000 0000 0000 0000  正好是 const  double  HOURLY\_WAGE = 30;   
  说明局部常量和变量都存放在内存栈中；  
  
  
  全局变量：   
  004015B6   .  E8 F3FBFFFF   call    004011AE                            //调用 AddTwoNumber 函数  
  004015BB   .  83C4 08       add     esp, 8  
  004015BE   .  A3 E8BD4700   mov     dword ptr \[47BDE8\], eax  
  
  我们知道，函数的返回值存放在 eax 寄存器中，故 47BDE8 存放的是全局变量  extern  int z;  
  
    z = AddTwoNumber(a,b);  
  
  全局变量一般不存放在可变内存区，这和它的寿命和作用域有关，全局变量作用域是整个程序，包含该程序的各个文件；  
  它的寿命与整个程序同在，并可由同一程序的不同函数共享，因此安全性较差；应该尽量少用；  
  
  alt+M 查看内存映像文件：  47BDE8 位于点.date 区段； 相关属性如下：   
  Memory map, 条目 20  
   地址=00476000  
   大小=00008000 (32768.)  
   属主=variable 00400000  
   区段=.data  
   包含=数据  
   类型=Imag 01001002  
   访问=R  
   初始访问=RWE  
  
476000+8000= 47E000  范围： 476000--47E000 ；  
  
   到这，我们应该完成了这一部分； 在运算 pay 时，用到了浮点数指令，这一部分，我会在运算符与表达式中写出；  
  
  
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
  
                                                       2008 年 12 月 07 日 14:10:44