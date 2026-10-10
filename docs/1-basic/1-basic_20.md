

# 汇编优化几种方式。

> 原文：[`www.pediy.com/kssd/pediy11/117767.html`](https://www.pediy.com/kssd/pediy11/117767.html)

最近从书中看到几中汇编优化方式，拿出分享分享。  
主要目的是提高运行效率，消除跳转指令。  
  
1.求有符号 32 位整数的绝对值。  
  
 逻辑： if (eax<0) eax=-eax  
 cdq  
 xor eax, edx  
 sub eax, edx  
  
 这段指令之所以可以正常工作，基于这个一个事实  
   
 neg eax 等价于 not eax; inc eax　  
 而 not eax 又等价于 xor eax, -1  
  
2.取两个数字中较小的一个  
  
 逻辑： if (ebx>eax) ebx=eax  
  
 无符号整数版本  
  
 sub eax, ebx  
 sbb edx, edx  
 and edx, eax  
 add ebx, edx  
  
 有符号版本  
   
 sub eax, ebx  
 cdq  
 and edx, eax  
 add ebx, edx  
  
3.根据条件来选择赋值  
  
 逻辑： edx=(eax<0)?ebx:ecx  
   
 cdq  
 xor ebx, ecx  
 and edx, ebx  
 xor edx, ecx 

mark.....  
  
还有最常见的  
  
eax = !! eax;  
  
sub 0, eax  
sbb eax, eax  
neg eax  
  
  
eax = ! eax;  
sub 0, eax  
sbb eax, eax  
inc eax