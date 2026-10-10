

# ASProtect\_SKE\_2.3Beta\_Build0319 stolen code 分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-639.htm`](https://www.pediy.com/kssd/pediy08/pediy8-639.htm)

ASProtect\_SKE\_2.3Beta\_Build0319  stolen code 分析  
  
ASProtect\_SKE\_2.3 版本的 stolen code 比 2.11 版本的要复杂多了，不过，建立在我上篇文章（ASProtect.SKE.2.11 stolen code 解密）的基础之上来分析就容易多了。。。  
  
如果没看过我上篇文章，我极力建议先看看。  
  
首先谈谈 OEP 的查找，因为 ASProtect\_SKE\_2.3 版本与以前的有所不同：2.3 似乎取消了 SEH,所以不可以象以前一样在最后一次异常时找特征码。这也好，省了不少麻烦。因为 IAT 加密了，因此我们可以在 IAT 上设内存写入断点，再找特征码，2.3 的特征码是 83C4285D5F5E5BC38BC0（注意：特征码较以前不同）  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*OEP 脚本\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
           lclr  
  dbh  
      
  //在 IAT 开始处设内存写断点       
  bpwm  00405000,1  
  
  run  
  
  bpmc    
  
  find  eip,#83C4285D5F5E5BC38BC0#   //特征串  
  sub  $RESULT,5  
  
  go  $RESULT  
  
  sti  
  rtr  
  
  ret  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
切入正题！  
  
废话不多说，让我们从 call    01240000 开始：  
  
011E0A5E    E8 9DF50500     call    01240000  
{  
  //第一层  
  01240000   /36:EB 01        jmp     short 01240004  
01240003   |F2:             prefix repne:  
….一路 F7  push 之类  
0124014E    5E  pop         esi    
0124014F    FFD6            call    esi      //F7  
{  
  //第二层  
  00A9FDA0    55              push    ebp  
00A9FDA1    8BEC            mov     ebp, esp  
….一路 F8 来到一个循环  
00A9FE36    8B55 18         mov     edx, \[ebp+18\]  
00A9FE39    8BC3            mov    eax, ebx  
00A9FE3B    E870F8FFF     call      00A9F6B0       //F4                  
00A9FE40    EB 1D           jmp     short 00A9FE5F  
00A9FE42    EB 01           jmp     short 00A9FE45  
00A9FE44  - E9 8B451050     jmp     50BA43D4  
00A9FE49    8B45 14         mov     eax, \[ebp+14\]  
00A9FE4C    50              push    eax  
00A9FE4D    E8 4AFCFFFF     call    00A9FA9C  
00A9FE52    50              push    eax  
00A9FE53    8BCE            mov     ecx, esi  
00A9FE55    8B55 18         mov     edx, \[ebp+18\]  
00A9FE58    8BC3            mov     eax, ebx  
00A9FE5A    E8 D5F9FFFF     call    00A9F834  
00A9FE5F    4F              dec     edi  
00A9FE60    0373 6C         add     esi, \[ebx+6C\]  
00A9FE63    85FF            test    edi, edi  
00A9FE65  ^ 77 A0           ja      short 00A9FE07  
  
在 00A9FE3B 处 F4,进去  
00A9FE3B    E870F8FFF     call      00A9F6B0  
{  
  //第三层  
  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  \*00A9F6B0    68 00000000     push    0      
\*00A9F6B5    68 B0F6A900    push    0A9F6B0  
\*00A9F6BA    68 B0E6B000    push    0B0E6B0  
\*00A9F6BF    E8 14F90000     call    00AAEFD8  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
上面这一小段代码在整个第三层你会经常遇到，其具体的作用是求取下个任务的执行地址。想想第一个任务是什么？既然是变形，那第一个任务肯定是求出变形的种类。因此第一次从 call    00AAEFD8 返回后就是求取类型代码的地址。  
至于 call    00AAEFD8，里面的循环很多，代码复杂，但其作用只是求取地址。因此 ASProtect\_SKE\_2.3 版本的特点可以说是执行地址的动态化。  
00A9F6BF    E8 14F90000     call    00AAEFD8  
{    
  00AAEFD8    60              pushad                  
00AAEFD9    89E0            mov     eax, esp  
00AAEFDB    9C              pushfd  
00AAEFDC    5A              pop     edx  
00AAEFDD    55              push    ebp  
。。。。。。后面不用看了，直到你来到  
00AAED9B    FF75 F8         push    dword ptr \[ebp-8\]  
00AAED9E    9D              popfd  
00AAED9F    8B65 F4         mov     esp, \[ebp-C\]  
00AAEDA2    61              popad  
00AAEDA3    C3              retn  
}  
XXXXXXXX              某任务的执行  
YYYYYYYY    RET        任务的执行结束  
  
00C50000      68 00000000     push    0      
00C5000x        68 B0F6A900    push    WWWW  
00C5000y        68 B0E6B000    push    QQQQQ  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
这里的 00C50000 是内存的分配得到的  
每个任务 PUSH 的 WWWW 和 QQQQQ 也不一样  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
00C5000z       E8 14F90000     call    00AAEFD8  
{  
  00AAEFD8    60              pushad                  
00AAEFD9    89E0            mov     eax, esp  
00AAEFDB    9C              pushfd  
00AAEFDC    5A              pop     edx  
00AAEFDD    55              push    ebp  
。。。。。。后面不用看了，直到你来到  
00AAED9B    FF75 F8         push    dword ptr \[ebp-8\]  
00AAED9E    9D              popfd  
00AAED9F    8B65 F4         mov     esp, \[ebp-C\]  
00AAEDA2    61              popad  
00AAEDA3    C3              retn  
  
}  
  
CCCCCCCC            某任务的执行  
SSSSSSSSS    RET         任务的执行结束  
  
。。。。。。。  
更多的任务  
。。。。。。。  
  
最后一次任务得到的地址肯定是退到第一层的地址  
ret  
}//退回到第一层  
  
  
  写了这么多不知你是否看懂（我写文章水平有限！），下面来分析：  
  
我们只要在 call    00AAEFD8 返回处（00AAEDA3）设断就可跟踪它的运行过程。  
  
我机器上每个 call    01240000 第一次中断后的返回地址是 011E2474（注意该地址是在被偷代码存放的空间里，不是在壳的代码里）  
011E2474    55              push    ebp  
011E2475    C1C5 87         rol     ebp, 87  
011E2478    BD 46154900     mov     ebp, 491546  
。。。。。  
011E2503    83C8 13         or      eax, 13  
011E2506    58              pop     eax      
011E2507    81E5 30504F46   and     ebp, 464F5030  
011E250D    5D              pop     ebp  
011E250E    C3              retn  
  
011E2474 处这段代码是求变形类型的，011E2506 处 eax 中值可为 0 1 2 3 四种情况：  
0:    call 变形  
1:    jmp 变形  
2:    jxx 变形  
3:    cmp    x,y 变形  
    jxx  
  
这四种变形接下来的任务不同，因此需要一个一个地分析：  
  
0:    call 变形  
Call 变形的任务有：  
Script Log Window  
Address    Message  
11E02FA    eip: 011E1CEE  
AAEFD8     \[tem\]: 00A9F6B0  
AAEFD8     eax: 00AE0684  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E2474      
AAEFD8     \[tem\]: 00A9F6D7  
AAEFD8     eax: 00000000  //变形类型为 0  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E23E9  
AAEFD8     \[tem\]: 00A9F6E9  
AAEFD8     eax: DBE56FB0  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E25C5  
AAEFD8     \[tem\]: 00A9F6FC  
AAEFD8     eax: DBE58E76  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9FA90  
AAEFD8     \[tem\]: 00A9F81A  
AAEFD8     eax: 0012FFE0  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 01250000  //返回第一层  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
说明：以上记录中 AAEDA3\[esp\]处为任务地址，AAEFD8\[tem\]处为 call    00AAEFD8 时 push 的值（没什么用，我们不需关心），AAEFD8 eax 处为任务返回时 eax 中的值。  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
由上可以看出，call 变形可在：  
可在 00A9FA90 处设断，执行到返回后可在\[EDX\]中得到 call 地址。  
  
如果\[EDX+8\]==0 则没有对原 call 中的 call 做处理，否则做了处理。  
  
1:    jmp 变形  
Script Log Window  
Address    Message  
11E02FA    eip: 011E0BD4  
AAEFD8     \[tem\]: 00A9F6B0  
AAEFD8     eax: 00AE0684  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E2474  
AAEFD8     \[tem\]: 00A9F6D7  
AAEFD8     eax: 00000001    //变形类型为 1  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E23E9  
AAEFD8     \[tem\]: 00A9F6E9  
AAEFD8     eax: DBE57715  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E25C5  
AAEFD8     \[tem\]: 00A9F6FC  
AAEFD8     eax: DBE56FB0  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9FA90  
AAEFD8     \[tem\]: 00A9F81A  
AAEFD8     eax: 0012FFE0  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 01250000  
  
Jmp 变形可在：  
00A9FA90 处设断，执行到返回后可在\[EDX\]中得到 jmp 地址。  
  
2:    jxx 变形  
Script Log Window  
Address    Message  
11E02FA    eip: 011E1825  
AAEFD8     \[tem\]: 00A9F6B0  
AAEFD8     eax: 00AE0684  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E2474  
AAEFD8     \[tem\]: 00A9F6D7  
AAEFD8     eax: 00000002  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E23E9  
AAEFD8     \[tem\]: 00A9F6E9  
AAEFD8     eax: DBE57805  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E25C5  
AAEFD8     \[tem\]: 00A9F6FC  
AAEFD8     eax: DBE579FD  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E2514    //求跳转类型  
AAEFD8     \[tem\]: 00A9F78F  
AAEFD8     eax: 0000009D  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9FAEC  
AAEFD8     \[tem\]: 00A9F7A4  
AAEFD8     eax: 00000001    //比较  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9FA90  
AAEFD8     \[tem\]: 00A9F81A  
AAEFD8     eax: 0012FFE0  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 01250000  
  
Jxx 变形可在：  
011E2514 处设断，执行到返回后 eax 中的值说明了跳转类型，我这儿是 9C,9D….对应 74，75…..，他们是顺序对应的。  
  
00A9FAEC 处设断，执行到返回后：  
若 eax==1  ：00A9FA90 处设断，执行到返回后可在\[EDX\]中得原条件为真的跳转地址。  
若 eax==0  ：00A9FA90 处设断，执行到返回后可在\[EDX\]中得不跳转的地址。  
  
3:    cmp    x,y 变形  
    jxx  
Script Log Window  
Address    Message  
11E02FA    eip: 011E0A5E  
11E0A5E    \[esp\]: 00A80000  
AAEFD8     \[tem\]: 00A9F6B0  
AAEFD8     eax: 00AE0684  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E2474  
AAEFD8     \[tem\]: 00A9F6D7  
AAEFD8     eax: 00000003  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E23E9  
AAEFD8     \[tem\]: 00A9F6E9  
AAEFD8     eax: DBE58D57  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E25C5  
AAEFD8     \[tem\]: 00A9F6FC  
AAEFD8     eax: DBE5835C  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9F464  
AAEFD8     \[tem\]: 00A9F464  
AAEFD8     eax: 00AE0684  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E2845  
AAEFD8     \[tem\]: 00A9F48A  
AAEFD8     eax: BF3C9D98  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E2259  
AAEFD8     \[tem\]: 00A9F49E  
AAEFD8     eax: 00000000  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9FD94  
AAEFD8     \[tem\]: 00A9F4B7  
AAEFD8     eax: 00000000  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E231A  
AAEFD8     \[tem\]: 00A9F4C8  
AAEFD8     eax: BF3C9D98  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E264F  
AAEFD8     \[tem\]: 00A9F4DC  
AAEFD8     eax: 00000006  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9FD94  
AAEFD8     \[tem\]: 00A9F4F5  
AAEFD8     eax: 0040B83E  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E27C0  
AAEFD8     \[tem\]: 00A9F514  
AAEFD8     eax: 00000004  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9FD68  
AAEFD8     \[tem\]: 00A9F573  
AAEFD8     eax: 00000202  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00C50000  
AAEFD8     \[tem\]: 00A9F71F  
AAEFD8     eax: 00000202  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 011E2514  
AAEFD8     \[tem\]: 00A9F732  
AAEFD8     eax: 0000009B  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9FAEC  
AAEFD8     \[tem\]: 00A9F747  
AAEFD8     eax: 00000001  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 00A9FA90  
AAEFD8     \[tem\]: 00A9F81A  
AAEFD8     eax: 0012FFE0  
AAEFD8     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
AAEDA3     \[esp\]: 01250000  
  
Cmp  x,y  
Jxx      变形可在：  
（\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
首先注意：见上面  
011E2259 处 y 为寄存器，则 011E2845 处返回值为固定值 BF3C9D98  
011E264F 处 x 为寄存器，则 011E231A 处返回值为固定值 BF3C9D98  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*）  
  
011E2845 处设断，执行到返回后 eax 中的值：  
如果 eax== BF3C9D98，则 y 为寄存器，否则 y=eax- BF3C9D98  
  
011E2259 处设断，执行到返回后 eax 中的值：  
如果 y 为寄存器，则指明了为哪个寄存器，否则为大于 8 的值。  
(0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi)  
  
011E231A 处设断，执行到返回后 eax 中的值：  
如果 eax== BF3C9D98，则 x 为寄存器，否则 x=eax- BF3C9D98  
  
011E264F 处设断，执行到返回后 eax 中的值：  
如果 x 为寄存器，则指明了为哪个寄存器，否则为大于 8 的值。  
 (0=eax,1=ecx,2=edx,3=ebx,4=esp,5=ebp,6=esi,7=edi)  
  
011E27C0 处设断，执行到返回后 eax 中的值指明了 cmp 的类型：  
若 eax=0：  cmp   dword ptr \[x\], y  
若 eax=1：  cmp   x,\[ y\]  
若 eax=2：  cmp   byte ptr \[x\], y  
若 eax=3：  cmp  x,byte ptr \[ y\]  
若 eax=4：  cmp   x, y  
  
011E2514 处设断，执行到返回后 eax 中的值说明了跳转类型，我这儿是 9C,9D….对应 74，75…..，它们是顺序对应的。  
  
00A9FAEC 处设断，执行到返回后：  
若 eax==1  ：00A9FA90 处设断，执行到返回后可在\[EDX\]中得原条件为真的跳转地址。  
若 eax==0  ：00A9FA90 处设断，执行到返回后可在\[EDX\]中得不跳转的地址。  
  
  
  
  
最后的修复可以写个脚本完整修复，所有 call    01240000 的地址可以全部找出来（我看见有人问怎么找）。  
011E0A42    E8 B9F50500     call    01240000  
011E0A47    59              pop     ecx  
011E0A48    8935 A0894000   mov     \[4089A0\], esi  
011E0A4E    C705 A08A4000 2>mov     dword ptr \[408AA0\], 20  
011E0A58    8D86 00010000   lea     eax, \[esi+100\]  
011E0A5E    E8 9DF50500     call    01240000  
011E0A63    8A4D 00         mov     cl, \[ebp\]  
011E0A66    E9 24140000     jmp     011E1E8F  
011E0A6B    68 470A1E01     push    11E0A47  
011E0A70    E8 8BF50500     call    01240000  
011E0A75  ^ 0F8C 63FDFFFF   jl      011E07DE  
011E0A7B    E9 8A0D0000     jmp     011E180A  
011E0A80    59              pop     ecx  
011E0A81    FF7424 14       push    dword ptr \[esp+14\]  
011E0A85    66:9C           pushfw  
011E0A87    51              push    ecx  
011E0A88    33CD            xor     ecx, ebp  
  
象上面这些 call    01240000 处的 01240000 原来是乱的，只要在相应的地址设个内存写断点，就可以找到处理这部分的代码。比如象上面的地址 011E0A43 处就可以。  
找到如下：  
00A9FF29    8B0424          mov     eax, \[esp\]  
00A9FF2C    0FB600          movzx   eax, byte ptr \[eax\]  
00A9FF2F    8B7483 40       mov     esi, \[ebx+eax\*4+40\]  
00A9FF33    8BC7            mov     eax, edi  
00A9FF35    FFD6            call    esi  
00A9FF37    8BF0            mov     esi, eax  
00A9FF39    0373 18         add     esi, \[ebx+18\]  
00A9FF3C    0373 68         add     esi, \[ebx+68\]  
00A9FF3F    8B53 1C         mov     edx, \[ebx+1C\]  
00A9FF42    8BC6            mov     eax, esi  
00A9FF44    E8 03EEFFFF     call    00A9ED4C       //地址；  
00A9FF49    46              inc     esi  
00A9FF4A    8906            mov     \[esi\], eax  
00A9FF4C    4D              dec     ebp  
00A9FF4D    037B 6C         add     edi, \[ebx+6C\]  
00A9FF50    85ED            test    ebp, ebp  
00A9FF52  ^ 77 D5           ja      short 00A9FF29  
  
  
Wak 4.27.2006@nuaa 

盯住计算  
 1。变形类型的函数  
 2。第一个 compare source 值的函数  
 3. 第二个 compare source 值的函数  
 4. 第一个 comapre destination 值的函数  
 5. 第二个 compare destination 值的函数  
 6. compare type (比较类型)的函数  
 7. 跳转类型的函数  
每个 Asprotect 程序中都能找到以上的函数, 这样跨版本的工具或脚本就能做出来。  
见树也要见林！