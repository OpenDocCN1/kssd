

# VirtualPC 单步标志问题

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-105.htm`](https://www.pediy.com/kssd/pediy07/pediy7-105.htm)

VirtualPC 单步标志问题  
kongfoo/2005.2.24  
  
同样是虚拟机系统，VirtualPC 比 VmWare 好用很多，VmWare 里面的 win98 慢得出奇，  
按下开始按钮画了十几次才画出开始菜单，VPC 里面就完全不同，几下就出来了。  
相信一般情况下装 VPC 的比 VmWare 的人多些。下面转入正题。  
  
XP、2k 下面，设置 T 位将使执行之后的指令引发单步中断，代码如下：  
  pushfd  
  or \[esp\],100h  
  popfd  
  nop //这里就引发中断  
但在 VPC 里面的 win2k 和 win98，popfd 执行后并没有把 T 位置 1。  
换言之，可以写一段代码来检测是否在 VPC 中。  
mov OldESP,esp  
push @NotInVPC  
push fs:\[0\]  
mov fs:\[0\],esp  
pushfd  
or \[esp\],100h  
popfd  
nop  
jmp @InVPC  
@NotInVPC:  
mov esp,OldESP  
pop fs:\[0\]  
add esp,4  
mov UnderVPC,0  
jmp @Exit  
@InVPC:  
pop fs:\[0\]  
add esp,4  
mov UnderVPC,1  
@Exit:  
  
VPC 的这个特点，有好也有不好，对于调试人员来说，这种利用设置 T 位产生  
单步中断的代码不再起作用，使调试工作相对简单了一点，但同时也会失去  
程序的流程。对于编程人员来说，在 VPC 里面就不能用直接设置 T 位的办法产  
生单步中断了。如果真要设置，其中一个办法就是利用 SEH，让操作系统来设置：  
PUSH @Seh  
PUSH DWORD PTR FS:\[0\]  
MOV DWORD PTR FS:\[0\],ESP  
INT3  
@Seh:  
MOV EAX,DWORD PTR SS:\[ESP+C\]  
MOV DWORD PTR DS:\[EAX+B8\],@NewEIP  
OR DWORD PTR DS:\[EAX+C0\],100  
XOR EAX,EAX  
RETN  
@NewEIP:  
POP DWORD PTR FS:\[0\]  //单步中断  
ADD ESP,4  
  
操作系统：winXP/win2k，VPC 版本：2004  
VPC 操作系统：win2K/win98 

 4dd466d0e1e0cc123afbe504da2462ca