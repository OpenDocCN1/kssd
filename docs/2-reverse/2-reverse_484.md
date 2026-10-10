

# 献给汇编初学者－函数调用堆栈变化分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-374.htm`](https://www.pediy.com/kssd/pediy09/pediy09-374.htm)

> 跟一个朋友谈堆栈的时候 就写下了这段文字，顺便发到这里给需要的看看吧  
> 汇编初学者比较头痛的一个问题  
> ////////////////////////////////////////////////////////////////////  
> 比如 我们有这样一个 C 函数  
> #include<stdio.h>  
> long test(int a,int b)  
> {  
>     a = a + 1;  
>     b = b + 100;  
>     return a + b;  
> }  
> void main()  
> {    
>   printf("%d",test(1000,2000));  
> }  
>   
> 写成 32 位汇编就是这样  
> ;//////////////////////////////////////////////////////////////////////////////////////////////////////  
> .386  
> .model flat,stdcall           ;这里我们用 stdcall 就是函数参数 压栈的时候从最后一个开始压，和被调用函数负责清栈  
> option casemap:none            ;区分大小写  
>   
> includelib msvcrt.lib          ;这里是引入类库 相当于 #include<stdio.h>了         
> printf  PROTO C:DWORD,:VARARG  ;这个就是声明一下我们要用的函数头，到时候 汇编程序会自动到 msvcrt.lib 里面找的了   
>                                 ;:VARARG 表后面的参数不确定 因为 C 就是这样的 printf(const char \*, ...);  
>                                ;这样的函数要注意 不是被调用函数负责清栈 因为它本身不知道有多少个参数  
>                                ;而是有调用者负责清栈  下面会详细说明  
> .data  
> szTextFmt  BYTE '%d',0        ;这个是用来类型转换的，跟 C 的一样,字符用字节类型  
> a          dword 1000         ;假设  
> b          dword 2000         ;处理数值都用双字 没有 int 跟 long 的区别  
>   
> ;/////////////////////////////////////////////////////////////////////////////////////////  
> .code  
>   
> \_test proc ;A:DWORD,B:DWORD   
>       push ebp  
>       mov  ebp,esp  
>       mov  eax,dword ptr ss:\[ebp+8\]  
>       add  eax,1  
>       mov  edx,dword ptr ss:\[ebp+0Ch\]  
>       add  edx,100  
>       add  eax,edx  
>       pop  ebp        
>       retn 8  
> \_test endp  
>   
> \_main proc   
>       push dword ptr ds:b       ;反汇编我们看到的 b 就不是 b 了而是一个\[\*\*\*\*\*\]数字 dword ptr 就是我们在 ds(数据段)把\[\*\*\*\*\*\]  
>                                 ;开始的一个双字长数值取出来  
>       push dword ptr ds:a       ;跟她对应的还有 byte ptr \*\*\*\*就是取一个字节出来 比如这样 mov  al,byte ptr ds:szTextFmt   
>                                 ;就把 % 取出来 而不包括 d  
>       call \_test                    
>       push eax                  ;假设 push eax 的地址是×××××  
>       push offset szTextFmt  
>       call printf  
>       add  esp,8  
>       ret               
> \_main endp  
> end  \_main  
>   
> ;////////////////////////////////////////////////////////////// 下面介绍堆栈的变化  
> 首先要明白的是 操作堆栈段 ss 只能用 esp 或 ebp 寄存器 其他的寄存器 eax ebx edx 等都不能够用 而 esp 永远指向堆栈栈顶 ebp 用来 在堆栈段  
>   
> 里面寻址  
> push 指令是压栈 ESP=ESP-4  
> pop  指令是出栈 ESP=ESP+4  
> 我们假设 main 函数一开始堆栈定是 ESP=400  
> push dword ptr ds:b                 ;ESP-4=396 ->里面的值就是 2000 就是 b 的数值  
> push dword ptr ds:a                 ;ESP-4=392 ->里面的值就是 1000 就是 a 的数值  
> call test                           ;ESP-4=388－>里面的数值是什么？这个太重要了 就是我们用来找游戏函数的原理所在。  
>                                                  里面的数值就是 call test 指令下一条指令的地址－>即 push eax 的地址×××××  
>   
> 到了 test 函数里面  
>   
> push ebp                           ;ESP-4=384->里面保存了当前 ebp 的值 而不是把 ebp 清零  
> mov  ebp,esp                       ;这里 ESP＝384 就没变化了，但是 ebp=esp=384,为什么要这样做呢 因为我们要用 ebp 到堆栈里面找参数  
> mov  eax,dword ptr ss:\[ebp+8\]      ;反汇编是这样的 想想为什么 a 就是\[ebp+8\]呢  
>                                    ;我们往上看看堆栈里地址 392 处就保存着 a 的值 这里 ebp=384 加上 8 正好就是 392 了  
>                                    ;这样就把传递过来的 1000 拿了出来 eax=1000  
> add  eax,1                         ;相当于 a+1 了 eax=1001  
> mov  edx,dword ptr ss:\[ebp+0Ch\]    ; 0Ch=12 一样道理这里指向堆栈的地址是 384+12=396 就是 2000 了 edx=2000  
> add  edx,100                       ;相当于 b+100 edx=2100  
> add  eax,edx                       ;eax=eax+edx＝1001＋2100＝3101 这里 eax 已经保存了最终的结果了   
>                                    ;因为 win32 汇编一般用 eax 返回结果 所以如果最终结果不是在 eax 里面的话 还要把它放到 eax  
>                                    ;比如假设我的结果保存在变量 nRet 里面 最后还是要这样 mov eax,dword ptr nRet  
> pop  ebp                           ;ESP=384+4=388 而保存在栈顶 384 的值 保存到 ebp 中 即恢复 ebp 原来的值                         
>                                    ;因为一开始我们就把 ebp 的值压栈了，mov ebp,esp 已经改变了 ebp 的值，这里恢复就是保证了堆栈平衡  
> retn  8                            ;ESP+8->396 这里 retn 是由系统调用的 我们不用管 系统会自动把 EIP 指针指向 原来的 call 的下一条指令  
>                                    ;由于是系统自动恢复了 call 那里的压栈所以 真正返回到的时候 ESP+4 就是恢复了 call 压栈的堆栈  
>                                    ;到了这个时候 ESP=400 就是函数调用开始的堆栈，就是说函数调用前跟函数调用后的堆栈是一样的  
>                                    ;这就是堆栈平衡   
> 由于我们用 stdcall 上面 retn 8 就是被调用者负责恢复堆栈的意思了，函数 test 是被调用者，所以负责把堆栈加 8,call 那里是系统自动恢复的  
>   
> push eax                ;ESP-4=396->里面保存了 eax 的值 3101  
>                         ;上面已经看到了 eax 保存着返回值，我们要把它传给 printf 也是通过堆栈传递         
> push offset szTextFmt   ;ESP-4=392->里面保存了 szTextFmt 的地址 也就是 C 里面的指针 实际上没有什么把字符串传递的，我们传的都是地址  
>                         ;无论是在汇编或 C 所以在汇编里没有什么字符串类型 用最多的就是 DWORD。嘿嘿游戏里面传递参数 简单多了  
> call printf             ;ESP-4=388->里面保存了下一条指令的地址  
> add  esp,8              ;ESP+8=400 恢复了调用 printf 前的堆栈状态  
>                         ;上面说了由于 printf 后面参数是:VARARG 这样的类型是有调用者恢复堆栈的 所以 printf 里面没有 retn 8 之类的指令  
>                         ;这是由调用者负责清栈 main 是调用者 所以下面一句就是 add esp,8 把堆栈恢复到调用 printf 之前  
>                         ;而 call printf 那里的压栈 是由系统做的 恢复的工作也是系统完成 我们不用理 只是知道里面保存是返回地址就够    
>   
>                       ;了  
> ret                     ;main 函数返回 其他的事情是系统自动搞定 我们不用理 任务完成 

 4dd466d0e1e0cc123afbe504da2462ca