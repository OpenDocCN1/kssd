

# Armadillo 2.51

> 原文：[`www.pediy.com/kssd/pediy05/pediy50074.htm`](https://www.pediy.com/kssd/pediy05/pediy50074.htm)

Program          : The Armadillo Software Protection System Version2.51   
Type            : Software Protection System   
OS              : Win9x/Me/2k   
Protection      : Armadillo?   
Homepage        : http://www.siliconrealms.com/   
Download        : http://mud.sz.jsinfo.net/per/aaron/files/protectors/win/armd250.zip   
  
  
Cracked by      : zombieys\[CCG\]   
Release type    : full unpack cracked   
Release date    : Apr 13,2002   
Release name    : cr-Armadillo251.exe (749,568 bytes)   
Release Download : http://www.fuluck.com/newpage/cr-Armadillo251.rar   
  
  
技术参考：   
The Armadillo Software Protection System   
Version 2.51   
  
(1)找入口:   
  bpx VirtualProtect,在最后一次被中断后，按大约 2 次 f12 返回 Armadillo 领空,按 f10 带过一个 call,在下一个 call 进入,继续按 f10 走到 call edi 处，edi 里存放的就是入口地址   
  
(2)脱壳   
    
  请参考 Hying\[CCG\]老大的原文   
  
  首先一点是 Armadillo 是通过类似于调试的方法来运行程序的。   
新版本 Armadillo 脱壳的一个难点是它的代码段是分页解密的，我估计它的做法是将代码段锁定，即将其属性设为不可读、写、执行，当被执行到时会产生一个页面保护错误，而调试程序就会捕获这个错误，将对应的页面解锁并解密，然后返回执行。   
要对它脱壳的话可以考虑这样做，编一个小程序，如下：   
push    PAGE\_READWRITE   
push    MEM\_COMMIT   
push    100000h        <-申请内存的大小，可大一点。   
push    0   
call    VirtualAlloc    <-申请内存   
mov    edi,eax   
mov    ecx,？？？？    <-文件映象大小减 1000h   
mov    esi,401000h    <-基址加 1000h，有的程序不是这个值   
rep  movsb   
push    80000000h   
call    Sleep   
在原程序中找个空间，尽量不要在代码段范围内，将着这段程序放进去，然后执行它，执行完后，你申请到的内存中就放的是解密后的程序，只要 DUMP 出来，再加一个可用的文件头，再修复输入表什么的就可以执行了。   
  
zombieys\[CCG\]   
2002.04.13  4dd466d0e1e0cc123afbe504da2462ca