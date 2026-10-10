

# 简单反汇编阅读

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-420.htm`](https://www.pediy.com/kssd/pediy09/pediy09-420.htm)

> 《简单反汇编阅读》是我学习了《天书夜读》后写的心得，并做成了动画， 也可以说是我看《天书夜读》的读书笔记吧。我贴出文本，然后提供动画的下载地址，希望对还没入门的兄弟有所帮助，也希望高手能指出我的不足。  
>   
>   
>   
> 大家好,我是 BoXer ,今天为大家做的动画是《简单反汇编阅读（一）》  
>   
> 这类的文章是不少，但是动画好像很少，那我做一个抛砖引玉吧。  
>   
> 由于水平有限，而且动画定位于初学者，所以动画会显得有点唠叨，请各位多多包涵  
>   
>   
> \------------------------------------------------------------------------------  
>   
>   
> 我们先启动 VC++6.0，输入下面的程序  
>   
> 程序很短，大家看我输入一次，留意成对编码原则，防止对低级的错误  
>   
> void  boxer(int a,int b)  
> {  
>   int c=a+b;  
> }  
>   
> void  main()  
> {  
>   boxer(1,2);  
> }  
>   
> 一个简单的有 2 个形参的 boxer()函数，接着在 main()主函数调用 boxer()函数  
>   
> 然后我们 debug 看看，F10 单步  
>   
> \------------------------------------------------------------------------------  
>   
> 6:    void    main()  
> 7:    {  
> 00401060   push        ebp    ；保存 ebp，          执行这句前 ESP = 0012FF84，EBP = 0012FFC0  
>           ；push 的结果是 esp 总减少，执行后 ESP = 0012FF80，EBP = 0012FFC0  
> 00401061   mov         ebp,esp    ；将 esp 放入 ebp 中此时 ebp 和 esp 相同，即执行后 ESP = EBP = 0012FF80  
>   
>   
> ；原 EBP 值已经被压栈（位于栈顶），而新的 EBP 又恰恰指向栈顶。  
> ；此时 EBP 寄存器就已经处于一个非常重要的地位，该寄存器中存储着栈中的一个地址（原 EBP 入栈后的栈顶），  
> ；从该地址为基准，向上（栈底方向）能获取返回地址、参数值，向下（栈顶方向）能获取函数局部变量值，  
> ；而该地址处又存储着上一层函数调用时的 EBP 值！  
>   
> \------------------------------------------------------------------------------  
>   
> 00401063   sub         esp,40h    ；把 esp 往上移动一个范围  
>           ；等于在栈中空出一片空间来存局部变量  
>           ；执行这句后 ESP = 0012FF40  
> 00401066   push        ebx    ；下面 3 句都是保存 3 个寄存器  
> 00401067   push        esi  
> 00401068   push        edi  
> 00401069   lea         edi,\[ebp-40h\]  ；把 ebp-40h 加载到 edi 中，目的是保存局部变量的区域  
> 0040106C   mov         ecx,10h  
> 00401071   mov         eax,0CCCCCCCCh  ；从 ebp-40h 开始的区域初始化成全部 0CCCCCCCCh，就是 int3 断点  
> 00401076   rep stos    dword ptr \[edi\]  ；拷贝字符串,初始化局部变量空间  
>   
>   
>   
> ；以上的语句就是在栈中开辟一块空间放局部变量  
> ；然后把这块空间都初始化为 0CCCCCCCCh，就是 int3 断点，一个中断指令。  
> ；因为局部变量不可能被执行，执行了就会出错，这时候发生中断提示开发者。  
> ；到时候我们可以用 od 很直观的看到  
>   
> \------------------------------------------------------------------------------  
>   
> 8:        boxer(1,2);  
> 00401078   push        2      ；参数 2 入栈，执行前 ESP = 0012FF34，执行后 ESP = 0012FF30  
> 0040107A   push        1      ；参数 1 入栈，执行后 ESP = 0012FF2C  
> 0040107C   call        @ILT+0(boxer) (00401005)  ；调用 boxer()函数，可以按 F11 跟进  
> 00401081   add         esp,8      ；调用完函数后恢复/释放栈，执行后 ESP = 0012FF34  
>   
>   
> ；其实 call 指令调用一个过程, 但它有一个小动作  
> ；在参数入栈以后, 被调用函数执行 之前, 它会将当前函数的下一条指令地址, 即 EIP 的值压入  
> ；  
> ；在 c/c++ 中, 函数的默认调用约定为 cdecl, 它约定参数从右到左入栈,   
> ；由调用者清理堆栈, 所谓清理, 即调整 ESP 的值, 使得原来的局部数据不再属于栈  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 9:    }  
> 00401084   pop         edi      ；下面 3 句都是恢复寄存器，上面怎样 push，这里就要对应反过来 pop  
> 00401085   pop         esi      ；简单来说就是先进来最后才出去，最后进来的先出去  
> 00401086   pop         ebx  
> 00401087   add         esp,40h      ；恢复 esp，对应上面的 sub esp，40h  
> 0040108A   cmp         ebp,esp      ；检查 esp 是否恢复正常，不正常就进入下面的 call 里面 debug  
> 0040108C   call        \_\_chkesp (004010b0)  ；处理可能出现的堆栈错误（如果出错，将陷入 debug）。  
> 00401091   mov         esp,ebp      ；将栈顶指针放回 esp  
> 00401093   pop         ebp      ；恢复原来的 ebp 和 esp，让上一个调用的函数正常使用  
> 00401094   ret          ；将返回地址存入 EIP, 转移流程  
>   
>   
> ；以上那部分代码在 vc 的 debug 调试版才会有，主要检查栈是否被破坏了。这样就能及时在你出错的地方停下了告诉你。    
> ；发行版自动消除这些代码(调试产生的)。  
> ；如果函数有返回值，返回值将放在 eax 返回（这就是很多软件给秒杀爆破的原因了，因为 eax 的返回值是可以改的）  
>   
> \------------------------------------------------------------------------------  
>   
> 好了，到了这里我们基本弄清主函数的详细过程了，其实无论主函数还是其他的普通函数，过程都是基本是一样的  
>   
> 下面我们看看 boxer()有那些不同就行了  
>   
> \------------------------------------------------------------------------------  
>   
> 1:    void    boxer(int a,int b)  
> 2:    {  
> 00401020   push        ebp  
> 00401021   mov         ebp,esp  
> 00401023   sub         esp,44h  
> 00401026   push        ebx  
> 00401027   push        esi  
> 00401028   push        edi  
> 00401029   lea         edi,\[ebp-44h\]  
> 0040102C   mov         ecx,11h  
> 00401031   mov         eax,0CCCCCCCCh  
> 00401036   rep stos    dword ptr \[edi\]  
>   
> ；上面的代码跟前面介绍过的几乎一样了  
>   
> \------------------------------------------------------------------------------  
>   
> 3:        int c=a+b;  
> 00401038   mov         eax,dword ptr \[ebp+8\]  ；取第一个参数 a 放在 eax 里面  
> 0040103B   add         eax,dword ptr \[ebp+0Ch\]  ；取第二个参数 b，加上 a 的值放在 eax 中  
> 0040103E   mov         dword ptr \[ebp-4\],eax  ；将最终结果放在 c 中  
>   
>   
>   
> ；一般而言，ss:\[ebp+4\]处为返回地址  
> ；ss:\[ebp+8\]处为第一个参数值（这里是 a），ss:\[ebp+0Ch\]处为第二个参数（这里是 b，这里 8+4=12=0Ch）  
> ；ss:\[ebp-4\]处为第一个局部变量（这里是 c），ss:\[ebp\]处为上一层 EBP 值  
> ；ebp 和函数返回值是 32 位，所以占 4 个字节  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 4:    }  
> 00401041   pop         edi  
> 00401042   pop         esi  
> 00401043   pop         ebx  
> 00401044   mov         esp,ebp  
> 00401046   pop         ebp  
> 00401047   ret  
>   
>   
> ；恢复性的操作，只是比前面少了检查 esp 的语句而已  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 基本的函数反汇编阅读过程就这样的，下一次的动画我们将进入复杂一点的函数里面看看。  
>   
> 如果看动画的时候有什么疑问，可以通过我的 blog 留言或者直接发 E-mail 给我，期待你的交流  
>   
> [`boxer.yo2.cn`](http://boxer.yo2.cn)  
>   
> 《简单反汇编阅读（一）》动画下载地址：[`3800hk.com/donghua/g/18945.html`](http://3800hk.com/donghua/g/18945.html) 

> 简单反汇编阅读（二）  
>   
> 上一个动画我们已经知道了函数反汇编后最基本的调用过程，今天我们来看看复杂一点的函数  
>   
> 由于水平有限，而且动画定位于初学者，所以动画会显得有点唠叨，请各位多多包涵  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 看下面程序：  
>   
> int  boxer(int a,int b)  
> {  
>   int c=a+b;  
>   for(int i=0;i<10;i++)  
>     {  
>     c=c+i;  
>     }  
>   return c;  
> }  
>   
> void  main()  
> {  
>   boxer(1,2);  
> }  
>   
> 变动的只是 boxer()里面加了一个 for 循环，我们就从 for 循环那里看就行了  
> \------------------------------------------------------------------------------  
>   
> 4:        for(int i=0;i<10;i++)  
> 00401041   mov         dword ptr \[ebp-8\],0  ；上一个动画我们提到\[ebp-4\]处为第一个局部变量（这里是 c）  
>             ；那么这里的\[ebp-8\]就是第二个局部变量 i  
>             ；对 i 赋值 i=0  
> 00401048   jmp         boxer+33h (00401053)  ；赋值后跳到 00401053  
>   
> ；上面 2 句的模板是这样的：  
> ；  
> ；mov  <循环变量>，<初值>  
> ；jmp  检查循环条件  
>   
>   
> 0040104A   mov         ecx,dword ptr \[ebp-8\]  ；把 i 的值给 ecx  
> 0040104D   add         ecx,1      ；ecx 加 1  
> 00401050   mov         dword ptr \[ebp-8\],ecx  ；加 1 后的 ecx 给 i，这样通过 ecx 互换实现 i++  
>   
>   
> ；上面 3 句只是简单的实现 i++，然后让下面的比较语句去比较  
>   
>   
>   
> 00401053   cmp         dword ptr \[ebp-8\],0Ah  ；来到这里，i 的值与 0Ah 比较，也就是 10 比较  
> 00401057   jge         boxer+44h (00401064)  ；i 大于或等于 0Ah 就跳转，就是说小于 10 时执行循环  
>   
>   
> ；上面 2 句的模板是这样的：  
> ；cmp  <循环变量>，<限制条件>  
> ；jge  跳出循环  
> ；(循环体)  
>   
>   
>   
>   
> 5:            {  
> 6:            c=c+i;  
> 00401059   mov         edx,dword ptr \[ebp-4\]  ；把第一个局部变量 c 给 edx  
> 0040105C   add         edx,dword ptr \[ebp-8\]  ；把第二个局部变量 i 加 edx  
> 0040105F   mov         dword ptr \[ebp-4\],edx  ；把相加的结果给 c，就是 c=c+i  
>   
> ；上面 3 句是循环体了，非常的简单，只要弄懂\[ebp-N\]就行了  
>   
>   
>   
>   
> 7:            }  
> 00401062   jmp         boxer+2Ah (0040104a)  ；跳会修改循环变量继续循环，这里是跳回 i++那里  
>   
> ；这句也是模板  
> ；  
> ；jmp  修改循环变量  
>   
>   
>   
>   
> 8:        return c;  
> 00401064   mov         eax,dword ptr \[ebp-4\]  ；把 c 的值给 eax 作为函数返回的结果  
>   
>   
> ；所有的函数默认都是用 eax 作返回值。这个定理在爆破的时候非常的有用。  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 总结一个 for 循环的反汇编结构如下：  
>   
>   mov  <循环变量>，<初值>  
>   jmp  检查循环条件 B  
> A:  (修改循环变量)  
>   ...  
>   ...  
>   
> B:  cmp  <循环变量>，<限制条件>  
>   jge  跳出循环  
>   (循环体)  
>   ...  
>   ...  
>     jmp  修改循环变量 A  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 接下来我们用 OD 看看没有调试信息的 release 版的情况，这样比较接近实践呢！  
>   
> 00401000  /$  8B4424 08     mov     eax, dword ptr \[esp+8\]           ;  第一个局部参数 c  
> 00401004  |.  8B4C24 04     mov     ecx, dword ptr \[esp+4\]           ;  第二个局部参数 i  
> 00401008  |.  03C1          add     eax, ecx                         ;  c=c+i  
> 0040100A  |.  33C9          xor     ecx, ecx                         ;  ecx 清零，就是 i 为 0 开始循环  
> 0040100C  |>  03C1          /add     eax, ecx                        ;  c=c+i  
> 0040100E  |.  41            |inc     ecx                             ;  i++  
> 0040100F  |.  83F9 0A       |cmp     ecx, 0A  
> 00401012  |.^ 7C F8         \\jl      short 0040100C  
> 00401014  \\.  C3            retn  
>   
>   
>   
>   
>   
>   
> 可以看到 Release 后的代码跟 OD 的引擎优化后出来的代码简单明了！  
>   
> 但是在很多情况下，发行版的代码都比 debug 版的难读，这个就要靠我们多练习了！  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 今天提到了 for 循环，其他的循环体比如 do，while 循环也是差不多的，看完动画后不妨自己试试  
>   
> 动画下载地址：http://3800hk.com/donghua/g/18973.html 

> 简单反汇编阅读（三）  
>   
> 上次我们看了 for 循环的反汇编，今天我们来看看 if 等的判断分支的语句  
>   
> 由于水平有限，而且动画定位于初学者，所以动画会显得有点唠叨，请各位多多包涵  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 看下面程序：  
>   
> #include "stdio.h"  
>   
> int  boxer()  
> {  
>   printf("please input a number: ");  
>   int a;  
>   scanf("%d",&a);  
>   if(a>50 && a<100)  
>     {  
>     printf("The number is %d\\n",a);  
>     }  
>   else if(a<=50)  
>     {  
>     printf("Too small!\\n");  
>     }  
>   else  
>     {  
>     printf("Too large!\\n");  
>     }  
>   return 0;  
> }  
>   
> void  main()  
> {  
>   boxer();  
> }  
>   
>   
> 我们像往常一样 F10 单步调试，然后转到看反汇编，我们把 boxer()函数那段 copy 过来  
>   
> 按照上次我们学的分好各个功能，这样看起来比较清晰  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 3:    int boxer()  
> 4:    {  
> 00401020   push        ebp  
> 00401021   mov         ebp,esp  
> 00401023   sub         esp,44h  
> 00401026   push        ebx  
> 00401027   push        esi  
> 00401028   push        edi  
> 00401029   lea         edi,\[ebp-44h\]  
> 0040102C   mov         ecx,11h  
> 00401031   mov         eax,0CCCCCCCCh  
> 00401036   rep stos    dword ptr \[edi\]  
>   
>   
> ；上面的代码大家很熟悉了，就是函数初始化的过程  
>   
>   
> 5:        printf("please input a number: ");  
> 00401038   push        offset string "please input a number: " (00425058)  
> 0040103D   call        printf (00401100)  
> 00401042   add         esp,4  
>   
>   
> ；上面 3 句先字符串入栈，然后调用 C 语言函数显示字符串，最后用 add esp,4 恢复栈  
> ；add   esp,4 是因为前面有 1 个 push，4 字节  
>   
>   
>   
> 6:        int a;  
> 7:        scanf("%d",&a);  
> 00401045   lea         eax,\[ebp-4\]  ；将第一个局部变量 a 的地址给 eax 寄存器  
> 00401048   push        eax    ；将 a 的地址入栈  
> 00401049   push        offset string "please input a number: %d\\n" (00425054)  
> 0040104E   call        scanf (00401180)  ；调用输入函数等待输入  
> 00401053   add         esp,8    ；调用完后恢复栈  
>   
>   
> ；上面的语句也不难理解，add esp,8 是因为前面有 2 个 push，8 字节  
>   
>   
>   
> 8:        if(a>50 && a<100)  
> 00401056   cmp         dword ptr \[ebp-4\],32h  ；第一个局部变量 a 与 32h（50）比较  
> 0040105A   jle         boxer+55h (00401075)  ；大于就不跳  
> 0040105C   cmp         dword ptr \[ebp-4\],64h  ；与 64h（100）比较  
> 00401060   jge         boxer+55h (00401075)  ；小于就不跳  
>   
>   
> ；if 的模板如下  
> ；  
> ；cmp  <各种条件>  
> ；各种跳转跳到下一个分支  
>   
>   
> 9:            {  
> 10:           printf("The number is %d\\n",a);  
> 00401062   mov         ecx,dword ptr \[ebp-4\]  
> 00401065   push        ecx  
> 00401066   push        offset string "The number is %d\\n" (0042503c)  
> 0040106B   call        printf (00401100)  
> 00401070   add         esp,8  
>   
>   
> ；打印语句，没什么难度  
>   
>   
>   
> 11:           }  
> 12:       else if(a<=50)  
> 00401073   jmp         boxer+77h (00401097)  ；如果之前执行了 if 语句后，来到这里直接跳出判断体  
> 00401075   cmp         dword ptr \[ebp-4\],32h  ；如果之前没执行 if 语句，就跳到这里比较  
> 00401079   jg          boxer+6Ah (0040108a)  ；大于 32h 就跳走，直接执行最后的 else 分支  
>   
>   
> ；else if 模板  
> ；  
> ；jmp  跳出判断  
> ；cmp  <条件>  
> ；各种跳转跳到下一个分支  
>   
>   
>   
>   
> 13:           {  
> 14:           printf("Too small!\\n");  
> 0040107B   push        offset string "Too small!\\n" (0042502c)  
> 00401080   call        printf (00401100)  
> 00401085   add         esp,4  
>   
>   
> ；说过了。。。  
>   
>   
>   
>   
> 15:           }  
> 16:       else  
> 00401088   jmp         boxer+77h (00401097)    
>   
>   
> ；如果能执行上面的语句，就直接一个硬跳转跳出判断  
>   
>   
>   
>   
> 17:           {  
> 18:           printf("Too large!\\n");  
> 0040108A   push        offset string "Too large!\\n" (0042501c)  
> 0040108F   call        printf (00401100)  
> 00401094   add         esp,4  
> 19:           }  
> 20:       return 0;  
> 00401097   xor         eax,eax    ；清空寄存器  
> 21:   }  
> 00401099   pop         edi  
> 0040109A   pop         esi  
> 0040109B   pop         ebx  
> 0040109C   add         esp,44h  
> 0040109F   cmp         ebp,esp  
> 004010A1   call        \_\_chkesp (004011e0)  
> 004010A6   mov         esp,ebp  
> 004010A8   pop         ebp  
> 004010A9   ret  
>   
>   
> 看完了 if-else 语句，我们看看其他判断语句  
>   
> \------------------------------------------------------------------------------  
>   
> 再看下面代码：  
>   
> #include "stdio.h"  
>   
> int  boxer()  
> {  
>   printf("please input a number: ");  
>   int a;  
>   scanf("%d",&a);  
>   switch(a)  
>   {  
>   case 0:  
>     {  
>     printf("c=0");  
>     }  
>   case 1:  
>     {  
>     printf("c=1");  
>     break;  
>     }  
>   case 100:  
>     {  
>     printf("a=100");  
>     }  
>   
>   default:  
>     {  
>     printf("default");  
>     }  
>   }  
>     
>   return 0;  
> }  
>   
> void  main()  
> {  
>   boxer();  
> }  
>   
>   
> 反汇编看看。。。由于只是改了判断语句，所以我们只看改的部分  
>   
> \------------------------------------------------------------------------------  
>   
> 8:        switch(a)  
> 9:        {  
> 00401056   mov         ecx,dword ptr \[ebp-4\]  ；第一个局部变量 a 放进 ecx  
> 00401059   mov         dword ptr \[ebp-8\],ecx  ；把 a 的值放在 ebp-8 里面，下面都是用 ebp-8 进行比较  
> 0040105C   cmp         dword ptr \[ebp-8\],0  ；与 0 比较  
> 00401060   je          boxer+50h (00401070)  ；相等就跳 case 0 那里  
> 00401062   cmp         dword ptr \[ebp-8\],1  ；与 1 比较  
> 00401066   je          boxer+5Dh (0040107d)  ；相等就跳 case 1 那里    
> 00401068   cmp         dword ptr \[ebp-8\],64h  ；与 100 比较  
> 0040106C   je          boxer+6Ch (0040108c)  ；相等就跳 case 100 那里  
> 0040106E   jmp         boxer+79h (00401099)  ；上面都不符合就跳到 default 那里  
>   
>   
> ；这里唯一疑惑的是为什么会把原本放在 ebp-4 的第一个局部变量 a 又多此一举放在 ebp-8 比较  
> ；这个是 debug 版本的特点，估计是为了防止直接操作堆栈造成破坏  
> ；switch()的模板如下:  
> ；  
> ；cmp  <条件 1>  
> ；je  <case 分支 1>  
> ；cmp  <条件 2>  
> ；je  <case 分支 2>  
> ；...  
> ；...  
> ；jmp  <default 分支>  
> ；（下面就是一堆的执行代码，如果期间出现 jmp 就是对应的 break 语句了）  
>   
>   
>   
> 10:       case 0:  
> 11:           {  
> 12:           printf("c=0");  
> 00401070   push        offset string "c>0" (00425054)  
> 00401075   call        printf (00401100)  
> 0040107A   add         esp,4  
> 13:           }  
> 14:       case 1:  
> 15:           {  
> 16:           printf("c=1");  
> 0040107D   push        offset string "c=1" (0042503c)  
> 00401082   call        printf (00401100)  
> 00401087   add         esp,4  
>   
>   
>   
> ；跳到对应分支就直接执行里面的语句，不在做任何判断  
>   
>   
>   
>   
> 17:           break;  
> 0040108A   jmp         boxer+86h (004010a6)  
>   
> ；唯一注意的是 break 对应的是一个 jmp 硬跳转  
>   
>   
>   
> 18:           }  
> 19:       case 100:  
> 20:           {  
> 21:           printf("a=100");  
> 0040108C   push        offset string "case 100" (0042502c)  
> 00401091   call        printf (00401100)  
> 00401096   add         esp,4  
> 22:           }  
> 23:  
> 24:       default:  
> 25:           {  
> 26:           printf("default");  
> 00401099   push        offset string "Too large!\\n" (0042501c)  
> 0040109E   call        printf (00401100)  
> 004010A3   add         esp,4  
> 27:           }  
> 28:       }  
>   
>   
>   
> 由于时间关系，release 版的 OD 代码就留给大家自己练习了！  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 判断分支语句大致就到这里了，我们不单是学习怎么看汇编，最重要的是学习方法  
>   
> 运用我们学习反汇编的方法，不断去学习其它 C/C++语句或者是其它编程语言的反汇编  
>   
> 下次我们就会开始尝试只知道反汇编语句写出大概的原代码（C/C++）  
>   
> 动画下载地址：http://3800hk.com/donghua/g/18987.html 

> 简单反汇编阅读（四）  
>   
>   
> 上次我们看过了 if 等的判断分支的语句，今天开始进入由反汇编写出反 C 代码  
>   
> 由于水平有限，而且动画定位于初学者，所以动画会显得有点唠叨，请各位多多包涵  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 看下面 debug 版本的反汇编代码，我已经删掉源代码了，所以只能靠我们自己写出来  
>   
> 00401020   push        ebp  
> 00401021   mov         ebp,esp  
> 00401023   sub         esp,50h  
> 00401026   push        ebx  
> 00401027   push        esi  
> 00401028   push        edi  
> 00401029   lea         edi,\[ebp-50h\]  
> 0040102C   mov         ecx,14h  
> 00401031   mov         eax,0CCCCCCCCh  
> 00401036   rep stos    dword ptr \[edi\]  
>   
>   
> 00401038   mov         dword ptr \[ebp-4\],64h  
> 0040103F   jmp         boxer+2Ah (0040104a)  
> 00401041   mov         eax,dword ptr \[ebp-4\]  
> 00401044   add         eax,1  
> 00401047   mov         dword ptr \[ebp-4\],eax  
> 0040104A   cmp         dword ptr \[ebp-4\],3E8h  
> 00401051   jge         boxer+0A5h (004010c5)  
> 00401053   mov         eax,dword ptr \[ebp-4\]  
> 00401056   cdq  
> 00401057   mov         ecx,64h  
> 0040105C   idiv        eax,ecx  
> 0040105E   mov         dword ptr \[i\],eax  
> 00401061   mov         eax,dword ptr \[ebp-4\]  
> 00401064   cdq  
> 00401065   mov         ecx,0Ah  
> 0040106A   idiv        eax,ecx  
> 0040106C   mov         edx,dword ptr \[i\]  
> 0040106F   imul        edx,edx,0Ah  
> 00401072   sub         eax,edx  
> 00401074   mov         dword ptr \[j\],eax  
> 00401077   mov         eax,dword ptr \[ebp-4\]  
> 0040107A   cdq  
> 0040107B   mov         ecx,0Ah  
> 00401080   idiv        eax,ecx  
> 00401082   mov         dword ptr \[k\],edx  
> 00401085   mov         edx,dword ptr \[i\]  
> 00401088   imul        edx,dword ptr \[i\]  
> 0040108C   imul        edx,dword ptr \[i\]  
> 00401090   mov         eax,dword ptr \[j\]  
> 00401093   imul        eax,dword ptr \[j\]  
> 00401097   imul        eax,dword ptr \[j\]  
> 0040109B   add         edx,eax  
> 0040109D   mov         ecx,dword ptr \[k\]  
> 004010A0   imul        ecx,dword ptr \[k\]  
> 004010A4   imul        ecx,dword ptr \[k\]  
> 004010A8   add         edx,ecx  
> 004010AA   cmp         dword ptr \[ebp-4\],edx  
> 004010AD   jne         boxer+0A0h (004010c0)  
> 004010AF   mov         edx,dword ptr \[ebp-4\]  
> 004010B2   push        edx  
> 004010B3   push        offset string "%4d" (00422020)  
> 004010B8   call        printf (00401160)  
> 004010BD   add         esp,8  
> 004010C0   jmp         boxer+21h (00401041)  
> 004010C5   push        offset string "\\n" (0042201c)  
> 004010CA   call        printf (00401160)  
> 004010CF   add         esp,4  
> 004010D2   xor         eax,eax  
>   
>   
>   
> 004010D4   pop         edi  
> 004010D5   pop         esi  
> 004010D6   pop         ebx  
> 004010D7   add         esp,50h  
> 004010DA   cmp         ebp,esp  
> 004010DC   call        \_\_chkesp (004011e0)  
> 004010E1   mov         esp,ebp  
> 004010E3   pop         ebp  
> 004010E4   ret  
>   
>   
>   
> \------------------------------------------------------------------------------  
>   
>   
>   
> 00401020   push        ebp  
> 00401021   mov         ebp,esp  
> 00401023   sub         esp,50h  
> 00401026   push        ebx  
> 00401027   push        esi  
> 00401028   push        edi  
> 00401029   lea         edi,\[ebp-50h\]  
> 0040102C   mov         ecx,14h  
> 00401031   mov         eax,0CCCCCCCCh  
> 00401036   rep stos    dword ptr \[edi\]  
>   
> ；函数初始化以及分配栈的代码  
>   
> 00401038   mov         dword ptr \[ebp-4\],64h  ；将 100 给第一个局部变量，我们假设它是 n ,int n=100;  
> 0040103F   jmp         boxer+2Ah (0040104a)  ；之后一个只直接跳转到 0040104a  
>   
>   
> 00401041   mov         eax,dword ptr \[ebp-4\]  ；n 放到 eax  
> 00401044   add         eax,1      ；eax+1  
> 00401047   mov         dword ptr \[ebp-4\],eax  ；就是 n++  
>   
>   
>   
> 0040104A   cmp         dword ptr \[ebp-4\],3E8h  ；n 与 3E8 比较，我们看看 10 进制的数是什么，n<1000;  
> 00401051   jge         boxer+0A5h (004010c5)  ；小于就不跳，继续  
>   
>   
> 00401053   mov         eax,dword ptr \[ebp-4\]  ；n 放进 eax  
> 00401056   cdq          ；CDQ 双字扩展. (把 EAX 中的值与符号扩展为 EDX:EAX)  
> 00401057   mov         ecx,64h      ；100 放进 ecx  
> 0040105C   idiv        eax,ecx      ；整数除法，eax 除以 ecx，就是 n/100，商保存在 eax，余数放在 edx  
> 0040105E   mov         dword ptr \[i\],eax  ；把结果放在变量 i 中,i=n/100  
>   
>   
> 00401061   mov         eax,dword ptr \[ebp-4\]  ；再把 n 放进 eax  
> 00401064   cdq  
> 00401065   mov         ecx,0Ah      ；把 0Ah（10）放进 ecx  
> 0040106A   idiv        eax,ecx      ；n/10，商保存在 eax，余数放在 edx  
> 0040106C   mov         edx,dword ptr \[i\]  ；将之前 n/100 的结果放到 edx 中  
> 0040106F   imul        edx,edx,0Ah    ；edx 乘 10 结果放在 edx 中  
> 00401072   sub         eax,edx      ；n/10-i\*10  
> 00401074   mov         dword ptr \[j\],eax  ；j=n/10-i\*10;  
>   
>   
> 00401077   mov         eax,dword ptr \[ebp-4\]  ；n 放进 eax  
> 0040107A   cdq  
> 0040107B   mov         ecx,0Ah      ；10 放进 ecx  
> 00401080   idiv        eax,ecx      ；n/10，商放在 eax（其实是 ax），余数在 edx（其实是 dx）  
> 00401082   mov         dword ptr \[k\],edx  ；将 n/10 余数放进变量 k，就是 k=n%10;  
>   
>   
> 00401085   mov         edx,dword ptr \[i\]  ；这三句是 i\*i\*i  
> 00401088   imul        edx,dword ptr \[i\]  
> 0040108C   imul        edx,dword ptr \[i\]  
> 00401090   mov         eax,dword ptr \[j\]  ；这三句是 j\*j\*j  
> 00401093   imul        eax,dword ptr \[j\]  
> 00401097   imul        eax,dword ptr \[j\]  
> 0040109B   add         edx,eax      ；i\*i\*i+j\*j\*j  
> 0040109D   mov         ecx,dword ptr \[k\]  ；这三句是 k\*k\*k  
> 004010A0   imul        ecx,dword ptr \[k\]  
> 004010A4   imul        ecx,dword ptr \[k\]  
> 004010A8   add         edx,ecx      ；i\*i\*i+j\*j\*j+k\*k\*k，放在 edx  
>   
>   
> 004010AA   cmp         dword ptr \[ebp-4\],edx  ；将上面的结果和变量 n 比较  
> 004010AD   jne         boxer+0A0h (004010c0)  ；不等就跳 004010c0  
>             ；一个 cmp 加一个跳转，是 if 语句了  
>   
>   
> 004010AF   mov         edx,dword ptr \[ebp-4\]  ；n 放在 edx，然后入栈，调用 printf 函数  
> 004010B2   push        edx        
> 004010B3   push        offset string "%4d" (00422020)  
> 004010B8   call        printf (00401160)  
> 004010BD   add         esp,8  
>   
>   
>   
> 004010C0   jmp         boxer+21h (00401041)  ；一个硬跳转回到开头，上面应该是一个 for 循环体了  
>             ；我们可以复习一下 for 循环的 debug 版模板  
>   
>   
>             ；又一个 printf 语句  
> 004010C5   push        offset string "\\n" (0042201c)  
> 004010CA   call        printf (00401160)  
> 004010CF   add         esp,4  
> 004010D2   xor         eax,eax      清空 eax，就是返回值是 0，return 0;  
>   
>   
> ；上面就是代码的主体了，我们看看怎么写反 C 代码  
>   
> 004010D4   pop         edi  
> 004010D5   pop         esi  
> 004010D6   pop         ebx  
> 004010D7   add         esp,50h  
> 004010DA   cmp         ebp,esp  
> 004010DC   call        \_\_chkesp (004011e0)  
> 004010E1   mov         esp,ebp  
> 004010E3   pop         ebp  
> 004010E4   ret  
>   
>   
> ；函数结束时候恢复寄存器恢复堆栈检查 esp 等代码  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 完整分析了上面代码后，我们去写出 C 代码  
>   
>   
> int  boxer()  
> {  
>   for (int n=100; n<1000; n++)  
>   {  
>     int i=n/100;  
>     int j=n/10-i\*10;  
>     int k=n%10;  
>   
>     if (n==i\*i\*i+j\*j\*j+k\*k\*k)  
>     {  
>       printf("%4d",n);  
>     }  
>   
>   }  
>   
>   printf("\\n");  
>   
>   return 0;  
> }  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 写完代码后，我们补全这个程序然后验证一下，显示结果了看看程序的用途  
>   
> 比如说 153=1³+5³+3³，这个是一个水仙花数  
>   
> 水仙花数：一个 3 位数，其各位数字的立方和等于该数本身  
>   
> 所以这个程序是打印出所有的水仙花数的  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 今天我们看了从不知道源码的情况下尝试写出反 C 代码，不过这个程序是最基础的  
>   
> 下次我们不妨来点更难的算法  
>   
> 动画下载地址：http://3800hk.com/donghua/g/19001.html 

> 简单反汇编阅读（五）  
>   
>   
>   
> 上次我们尝试了从不知道源码的情况下写出反 C 代码，今天我们不妨来点更难的算法  
>   
> 而且是 OD 实战 release 发行版的代码  
>   
> 由于水平有限，而且动画定位于初学者，所以动画会显得有点唠叨，请各位多多包涵  
>   
> \------------------------------------------------------------------------------  
>   
>   
> 在实战代码前，我们先看看数组的反汇编代码  
>   
> C 源码如下：  
>   
> int  boxer()  
> {  
>   int a\[2\], b\[2\], c\[2\];  
>     
>   for (int i=0; i<2; i++)  
>   {  
>     c\[i\]=a\[i\]\*b\[i\];  
>   }  
>   
>   return 0;  
> }  
>   
> void  main()  
> {  
>   boxer();  
> }  
>   
> \------------------------------------------------------------------------------  
>   
> 数组部分反汇编代码如下：  
>   
>   
>   
> 3:        int a\[2\], b\[2\], c\[2\];  
> 4:  
> 5:        for (int i=0; i<2; i++)  
> 00401038   mov         dword ptr \[ebp-1Ch\],0    ；ebp-1Ch 为第 4 个局部变量 i，因为前面 3 个数组变量各占了 8 字节  
>               ；ebp-4   :a\[0\],  ebp-8    :a\[1\]  
>               ；ebp-Ch  :b\[0\],  ebp-10h :b\[1\]  
>               ；ebp-14h :c\[0\],  ebp-18h  :c\[1\]  
>               ；ebp-1Ch :i  
> 0040103F   jmp         boxer+2Ah (0040104a)  
>   
>   
> 00401041   mov         eax,dword ptr \[ebp-1Ch\]  
> 00401044   add         eax,1  
> 00401047   mov         dword ptr \[ebp-1Ch\],eax  
> 0040104A   cmp         dword ptr \[ebp-1Ch\],2  
> 0040104E   jge         boxer+48h (00401068)  
>   
>   
> 6:        {  
> 7:            c\[i\]=a\[i\]\*b\[i\];  
> 00401050   mov         ecx,dword ptr \[ebp-1Ch\]    ；把 i 值放进 ecx，这里假设 ecx 为 a，则这句意思是 a\[i\]  
> 00401053   mov         edx,dword ptr \[ebp-1Ch\]    ；把 i 值放进 edx，这里假设 edx 为 b，则这句意思是 b\[i\]  
> 00401056   mov         eax,dword ptr \[ebp+ecx\*4-8\]  ；ebp+ecx\*4-8 就是当 ecx（i）为 0 时，这句变成 ebp-8,a\[1\]  
>               ；当 ecx（i）为 1 时，这句变成 ebp+4-8==ebp-4,a\[0\]  
> 0040105A   imul        eax,dword ptr \[ebp+edx\*4-10h\]  ；ebp+edx\*4-10h 就是当 edx（i）为 0 时，这句变成 ebp-10h,b\[1\]  
>               ；当 edx（i）为 1 时，这句变成 ebp+4-10h==ebp-C,b\[0\]  
> 0040105F   mov         ecx,dword ptr \[ebp-1Ch\]    ；再把 i 放进 ecx  
> 00401062   mov         dword ptr \[ebp+ecx\*4-18h\],eax  ；ebp+ecx\*4-18h，ecx 为 0 时为 c\[1\]，为 1 时为 c\[0\]  
>               ；就是 c\[1\]=a\[1\]\*b\[1\],c\[0\]=a\[0\]\*b\[0\]  
>               ；从这里看出数组在反汇编中是从后面取起的，是不是有点意外呢？  
>   
>   
> 8:        }  
> 00401066   jmp         boxer+21h (00401041)  
> 9:  
> 10:       return 0;  
> 00401068   xor         eax,eax  
> 11:   }  
>   
>   
> \------------------------------------------------------------------------------  
>   
> 知道了数组的大概反汇编后，我们就可以进入 OD 反汇编发行版的代码看看  
>   
> 以下的一段来历不明的 OD 反汇编代码我也不知道源代码的，我们一起把它反 C 代码出来  
>   
>   
>   
> 00401000  /$  8>mov     eax, dword ptr \[esp+8\]        ;  第二个形参给 eax，我们假设是 b  
> 00401004  |.  5>push    edi                           ;  保存 edi，那么 edi 就是第一个循环变量了，假设为 i  
> 00401005  |.  8>lea     edi, dword ptr \[eax-1\]        ;  edi 指向 eax-1，就是说将 b-1 的值赋给 edi  
> 00401008  |.  8>test    edi, edi                      ;  判断 edi 是否为 0  
> 0040100A  |.  7>jl      short 00401033                ;  小于 0 就跳出循环，大于等于就执行循环体  i>=0  
> 0040100C  |.  5>push    ebx  
> 0040100D  |.  8>mov     ebx, dword ptr \[esp+C\]        ;  第一个形参（假设为 a）给 ebx  
> 00401011  |.  5>push    esi                           ;  保存 esi，那么 esi 就是第二个循环变量了，假设为 j  
> 00401012  |>  8>/test    edi, edi                     ;  edi 与 0 比较  
> 00401014  |.  7>|jle     short 0040102E               ;  小于或等于就跳转，就是大于 0 就执行循环体  
> 00401016  |.  8>|mov     eax, ebx                     ;  从下面看 eax，eax+4 可以看出 a 是数组，这里 eax 指向 a\[j\]  
> 00401018  |.  8>|mov     esi, edi                     ;  b-1 的值给 esi  
> 0040101A  |>  8>|/mov     ecx, dword ptr \[eax\]        ;  栈中的一个数放在 ecx  
> 0040101C  |.  8>||mov     edx, dword ptr \[eax+4\]      ;  接着下一个数放在 edx  
> 0040101F  |.  3>||cmp     ecx, edx                    ;  两数比较  
> 00401021  |.  7>||jle     short 00401028              ;  小于或者等于就跳转 a\[j\]>a\[j+1\]，好明显的 if 语句  
> 00401023  |.  8>||mov     dword ptr \[eax\], edx        ;  第一个数如果比第二个数大，就两数交换  
> 00401025  |.  8>||mov     dword ptr \[eax+4\], ecx      ;  小的数放在\[eax\]，大的放在\[eax+4\]  
> 00401028  |>  8>||add     eax, 4                      ;  eax 指向栈的下一个数  
> 0040102B  |.  4>||dec     esi                         ;  esi 循环变量减 1  
> 0040102C  |.^ 7>|\\jnz     short 0040101A              ;  循环还没有结束就跳回去继续  
> 0040102E  |>  4>|dec     edi                          ;  edi 循环变量减 1，就是 i--  
> 0040102F  |.^ 7>\\jns     short 00401012               ;  如果 edi 不为-1 时候，继续跳回去，j>-1  
> 00401031  |.  5>pop     esi  
> 00401032  |.  5>pop     ebx  
> 00401033  |>  5>pop     edi  
> 00401034  \\.  C>retn  
>   
>   
>   
> 由于时间关系我写好了注释，我们现在一句一句看，然后写出反 C 代码  
>   
> int  boxer(int a\[\], int b)  
> {  
>   for (int i=b-1; i>=0; i--)  
>   {  
>     for (int j=b-1; j>=0; j--)  
>     {  
>       if (a\[j\]>a\[j+1\])  
>       {  
>         int c=a\[j\];  
>         a\[j\]=a\[j+1\];  
>         a\[j+1\]=c;  
>       }  
>     }  
>   }  
> }  
>   
> 冒泡法，两两对比，然后把小的数放在前面，大的数就放在后面  
>   
> 实现了从小到大的一个排序  
>   
> 动画下载地址：http://3800hk.com/donghua/g/19016.html 

 4dd466d0e1e0cc123afbe504da2462ca