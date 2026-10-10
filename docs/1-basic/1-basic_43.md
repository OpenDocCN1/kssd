

# 检测位指令 TEST 的用法

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-823.htm`](https://www.pediy.com/kssd/pediy08/pediy8-823.htm)

我知道 TEST 是检测位指令的,但它是怎么影响标志位的呢,看了一些资料但还是不明白  
假如我用了 test ax,11111111b 后, 标志位会是什么样子的  
我反汇编一些程序,老是见有如下命令行,很不理解,大家给解释下  
0043CA56   .  85C0          TEST EAX,EAX  
0043CA58   .  0F85 9F000000 JNZ L2Wind1.0043CAFD 

  

TEST - Test For Bit Pattern  
//位测试指令  
        Usage:  TEST    dest,src  
//用法：TEST dest,src  
        Modifies flags: CF OF PF SF ZF (AF undefined)  
//受影响的标志位有：  
//你要注意的是这一部位才对。你上面的指令 test eax,eax 后面跟着的是 jnz 指令，jnz 指令作用于零标志位的，若零标志位不为零则跳，所以 test eax,eax 是测试 eax 是否为零的指令，若 eax 有数据，不管是什么数据(111 也好，101 也好)，jnz 都会跳。  
//更新一下，若 eax 有数据，不管是 111 也好，222 也好，只要他们两个操作数作逻辑与运算之后，结果不为零，则 jnz 都会跳。而 jz 则不跳。  
CF //进位标志，主要作用于 JC/JNC 指令。  
OF //溢出标志，主要作用于 JO/JNO 指令  
PF //奇偶标志，主要作用于 JP/JPE/JNP/JPO 指令  
SF //符号标志，主要作用于 JS/JNS 指令  
ZF //零标志，主要作用于 JZ/JE/JNZ/JNE 指令  
AF //辅助进位标志，不过未定义。  
        Performs a logical AND of the two operands updating the flags  
//执行两个操作数的逻辑与运算，并更新标志位。  
        register without saving the result.  
//该操作只修改标志位，不保留运算结果值。