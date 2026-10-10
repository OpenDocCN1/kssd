

# 整数相除的问题

> 原文：[`www.pediy.com/kssd/pediy10/60874.html`](https://www.pediy.com/kssd/pediy10/60874.html)

如，函数：    

代码:

```
int func(int a){return a/0x1234;}

```

反汇编后，可得到：代码:

```
00401000  MOV ECX,DWORD PTR SS:\[ESP+4\]
00401004  MOV EAX,38410B35 ;<=== 注意这个数值
00401009  IMUL ECX
0040100B  MOV EAX,EDX
0040100D  SAR EAX,0A       
00401010  MOV ECX,EAX
00401012  SHR ECX,1F       
00401015  ADD EAX,ECX
00401017  RETN

```

这里采用一种倒数相乘的方法来实现整数相除。0x1234 转为倒数之后，是如何和 38410B35 这个数值联系起来的？  
  
请高手指点，谢谢！ 

0x38410B35 = 0x100000000\*0x400/0x1234  
  
取 EDX 相当于缩放 0x100000000 倍  
SAR EAX,0A 相当于缩放 0x400 倍