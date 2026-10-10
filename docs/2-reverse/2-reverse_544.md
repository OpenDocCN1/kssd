

# 从逆向工程的角度来看 C++ (九)

> 原文：[`www.pediy.com/kssd/pediy10/87595.html`](https://www.pediy.com/kssd/pediy10/87595.html)

进入本次正题：  
  
                           (九)\[  C++ 之 浮点数运算  \]  
  
  
今天来学学浮点数的反汇编吧.  
因为这几天电信宽带到期了, 暂时不想联网, 所以对于浮点数这里, 我没有查什么资料, 关于浮点数原理方面的,我也没做过多的阐述, 有兴趣的可以去看看特定的 CPU 手册,应该都差不多了,80X86 模式的. 或者也可以看看汇编的书籍,都会有介绍的, 今次我们的目的是为了在反汇编一个用到了浮点数的程序时不至于理解不了其中的指令.  
  
浮点数分为单精度和双精度, 单精度的 C 关键字为 float, 双精度的为 double. 在 IA-32 体系结构下, 一个 float 占 32 位,也即一个机器字(跟字 WORD 意义不一样) . 一个 double 占 64 位, 也即 2 个机器字, 其进度更高.  
  
OK,废话少说.  
来 cpp:  
#include <stdio.h>  
  
float floatcalc(float x,float y)  
{  
float f;  
  
\_\_asm  
{  
   nop  
}  
f = x + y;  
f = x - y;  
f = x \* y;  
f = x / y;  
f++;  
return f;  
}  
  
double doublecalc(double x,double y)  
{  
double d;  
  
\_\_asm  
{  
   nop  
}  
d = x + y;  
d = x - y;  
d = x \* y;  
d = x / y;  
d++;  
return d;  
}  
  
void main(void)  
{  
double x;  
double y;  
\_\_asm int 3  
x = 123.111;  
y = 456.888;  
  
doublecalc(x,y);  
floatcalc((float)x,(float)y);  
}  
  
////Debug 版:  
00401110 > 55             push    ebp  
00401111    8BEC           mov     ebp, esp  
00401113    83EC 58        sub     esp, 58  
00401116    53             push    ebx  
00401117    56             push    esi  
00401118    57             push    edi  
00401119    8D7D A8        lea     edi, \[ebp-58\]  
0040111C    B9 16000000    mov     ecx, 16  
00401121    B8 CCCCCCCC    mov     eax, CCCCCCCC  
00401126    F3:AB          rep     stos dword ptr es:\[edi\]  
00401128    90             nop  
00401129    C745 F8 C976BE>mov     dword ptr \[ebp-8\], 9FBE76C9      ; /这里的数据是编译器根据我们  
00401130    C745 FC 1AC75E>mov     dword ptr \[ebp-4\], 405EC71A      ; |给出的浮点数转换而来的.  
00401137    C745 F0 91ED7C>mov     dword ptr \[ebp-10\], 3F7CED91     ; |这四句是在为我们的两个 double  
0040113E    C745 F4 358E7C>mov     dword ptr \[ebp-C\], 407C8E35      ; \\变量 x 和 y 进行赋值.  
00401145    8B45 F4        mov     eax, \[ebp-C\]                     ; /先压 y 的高 32 位,再压 y 的  
00401148    50             push    eax                              ; |低 32 位,这四句完成  
00401149    8B4D F0        mov     ecx, \[ebp-10\]                    ; |对 y 这个 double 型参数的  
0040114C    51             push    ecx                              ; \\压栈过程.  
0040114D    8B55 FC        mov     edx, \[ebp-4\]                     ; /  
00401150    52             push    edx                              ; |这四句完成对 x 的压栈过程.  
00401151    8B45 F8        mov     eax, \[ebp-8\]                     ; |  
00401154    50             push    eax                              ; \\  
00401155    E8 B5FEFFFF    call    Float.0040100F                   ; 进去看看.  
0040115A    DDD8           fstp    st                               ;对 st 寄存器执行一次 pop.  
0040115C    83C4 10        add     esp, 10                          ; 恢复堆栈.  
0040115F    DD45 F0        fld     qword ptr \[ebp-10\]               ; /这两句是在做 double 型的 y 到  
00401162    D955 EC        fst     dword ptr \[ebp-14\]               ; \\float 的强制转换. 这里它另外使用了内存.  
00401165    51             push    ecx                              ; 开辟一个 float 所需的空间.  
00401166    D91C24         fstp    dword ptr \[esp\]                  ; 存储到指定地址,并清空 ST 寄存器.  
00401169    DD45 F8        fld     qword ptr \[ebp-8\]                ; /  
0040116C    D955 E8        fst     dword ptr \[ebp-18\]               ; |这四句和上面四句是一样的 ,  
0040116F    51             push    ecx                              ; |把 float 型的 x 压栈.  
00401170    D91C24         fstp    dword ptr \[esp\]                  ; \\  
00401173    E8 92FEFFFF    call    Float.0040100A                   ; 进去看看.  
00401178    DDD8           fstp    st                               ; 执行一次 pop 操作, 因为 ST0 寄存器用不到了,所以 pop 之.  
0040117A    83C4 08        add     esp, 8                           ; 两个 float 的参数丢弃.  
0040117D    5F             pop     edi  
0040117E    5E             pop     esi  
0040117F    5B             pop     ebx  
00401180    83C4 58        add     esp, 58  
00401183    3BEC           cmp     ebp, esp  
00401185    E8 D6000000    call    Float.\_chkesp  
0040118A    8BE5           mov     esp, ebp  
0040118C    5D             pop     ebp  
0040118D    C3             retn  
  
  
////doublecalc 内部:  
004010A0 >/> \\55           push    ebp  
004010A1 |. 8BEC         mov     ebp, esp  
004010A3 |. 83EC 48      sub     esp, 48   //Debug 默认的 40h 和我们的一个 double 型的 8.  
004010A6 |. 53           push    ebx  
004010A7 |. 56           push    esi  
004010A8 |. 57           push    edi  
004010A9 |. 8D7D B8      lea     edi, \[ebp-48\]  
004010AC |. B9 12000000 mov     ecx, 12  
004010B1 |. B8 CCCCCCCC mov     eax, CCCCCCCC  
004010B6 |. F3:AB        rep     stos dword ptr es:\[edi\]  
004010B8 |. 90           nop                                      ; 我们的标志.  
  
//加法  
004010B9 |. DD45 08      fld     qword ptr \[ebp+8\]     //装载浮点数,精度看其指定的内存大小.qword 为双,dword 为单.(相应 ST 置 valid,意正在使用中.)  
004010BC |. DC45 10      fadd    qword ptr \[ebp+10\]   //加上另一个浮点数, 这里面应该有一个隐含的装载过程.  
004010BF |. DD5D F8      fstp    qword ptr \[ebp-8\] // store and pop : 也即存储至指定位置, 且将 ST0 清空(置 empty).  
  
//减法  
004010C2 |. DD45 08      fld     qword ptr \[ebp+8\]  
004010C5 |. DC65 10      fsub    qword ptr \[ebp+10\]  
004010C8 |. DD5D F8      fstp    qword ptr \[ebp-8\]  
  
//乘法  
004010CB |. DD45 08      fld     qword ptr \[ebp+8\]  
004010CE |. DC4D 10      fmul    qword ptr \[ebp+10\]  
004010D1 |. DD5D F8      fstp    qword ptr \[ebp-8\]  
  
//除法  
004010D4 |. DD45 08      fld     qword ptr \[ebp+8\]  
004010D7 |. DC75 10      fdiv    qword ptr \[ebp+10\]  
004010DA |. DD55 F8      fst     qword ptr \[ebp-8\]   //仅仅 store 进指定位置. 没有清零 ST0.  
004010DD |. DC05 2060420>fadd    qword ptr \[\_real\] //加上 1, \_real 是个内部符号, 此值其实是 1 的双精度表示, 在.rdata 段.  
004010E3 |. DD55 F8      fst     qword ptr \[ebp-8\]   //再来 store 到指定位置.(我们的 double 型局部变量) 到这里其实 ST0 还是 valid 的, 还没有清空.  
  
  
004010E6 |. 5F           pop     edi  
004010E7 |. 5E           pop     esi  
004010E8 |. 5B           pop     ebx  
004010E9 |. 83C4 48      add     esp, 48  
004010EC |. 3BEC         cmp     ebp, esp  
004010EE |. E8 6D010000 call    Float.\_chkesp  
004010F3 |. 8BE5         mov     esp, ebp  
004010F5 |. 5D           pop     ebp  
004010F6 \\. C3           retn  
  
////float : 都差不多, 只不过 double 是 qword. Float 是 dword. 就不分析了.  
00401030 >/> \\55           push    ebp  
00401031 |. 8BEC         mov     ebp, esp  
00401033 |. 83EC 44      sub     esp, 44 //40h+4.  
00401036 |. 53           push    ebx  
00401037 |. 56           push    esi  
00401038 |. 57           push    edi  
00401039 |. 8D7D BC      lea     edi, \[ebp-44\]  
0040103C |. B9 11000000 mov     ecx, 11  
00401041 |. B8 CCCCCCCC mov     eax, CCCCCCCC  
00401046 |. F3:AB        rep     stos dword ptr es:\[edi\]  
00401048 |. 90           nop  
  
00401049 |. D945 08      fld     dword ptr \[ebp+8\]  
0040104C |. D845 0C      fadd    dword ptr \[ebp+C\]  
0040104F |. D95D FC      fstp    dword ptr \[ebp-4\]  
  
00401052 |. D945 08      fld     dword ptr \[ebp+8\]  
00401055 |. D865 0C      fsub    dword ptr \[ebp+C\]  
00401058 |. D95D FC      fstp    dword ptr \[ebp-4\]  
  
0040105B |. D945 08      fld     dword ptr \[ebp+8\]  
0040105E |. D84D 0C      fmul    dword ptr \[ebp+C\]  
00401061 |. D95D FC      fstp    dword ptr \[ebp-4\]  
  
00401064 |. D945 08      fld     dword ptr \[ebp+8\]  
00401067 |. D875 0C      fdiv    dword ptr \[ebp+C\]  
0040106A |. D955 FC      fst     dword ptr \[ebp-4\]  
0040106D |. D805 1C60420>fadd    dword ptr \[\_real\]  
00401073 |. D955 FC      fst     dword ptr \[ebp-4\]  
  
00401076 |. 5F           pop     edi  
00401077 |. 5E           pop     esi  
00401078 |. 5B           pop     ebx  
00401079 |. 83C4 44      add     esp, 44  
0040107C |. 3BEC         cmp     ebp, esp  
0040107E |. E8 DD010000 call    Float.\_chkesp  
00401083 |. 8BE5         mov     esp, ebp  
00401085 |. 5D           pop     ebp  
00401086 \\. C3           retn  
  
  
////Release 版本的.  
00401040    90              nop  
00401041    68 358E7C40     push    407C8E35  
00401046    68 91ED7C3F     push    3F7CED91  
0040104B    68 1AC75E40     push    405EC71A  
00401050    68 C976BE9F     push    9FBE76C9                         ; 干脆直接.4 个 push 搞定两个 double 的压参.  
00401055    E8 C6FFFFFF     call    Float.00401020  
0040105A    68 AA71E443     push    43E471AA  
0040105F    68 D538F642     push    42F638D5                         ; 干脆直接, 2 个 push 搞定两个 float 的压参.  
00401064    DDD8            fstp    st                               ; ST0 中的值"剪切"到 ST7 中.  
00401066    E8 95FFFFFF     call    Float.00401000  
0040106B    DDD8            fstp    st                               ;   
0040106D    83C4 18         add     esp, 18  
00401070    C3              retn  
  
////double test.  
00401020    55              push    ebp  
00401021    8BEC            mov     ebp, esp  
00401023    90              nop  
00401024    DD45 08         fld     qword ptr \[ebp+8\]                ; load 第一个 double.  
00401027    DC75 10         fdiv    qword ptr \[ebp+10\]               ; 除第二个 double.对第二个 double 隐含 load.我是这样理解的.  
0040102A    DC05 A8604000   fadd    qword ptr \[4060A8\]               ; 加 1  
00401030    5D              pop     ebp  
00401031    C3              retn  
////这里只有加法的原因是因为编译器优化的结果, 我对 Release 设置的是最小大小的优化.  
  
////float test.   
00401000    55              push    ebp  
00401001    8BEC            mov     ebp, esp  
00401003    90              nop  
00401004    D945 08         fld     dword ptr \[ebp+8\]  
00401007    D875 0C         fdiv    dword ptr \[ebp+C\]  
0040100A    D805 A0604000   fadd    dword ptr \[4060A0\]  
00401010    5D              pop     ebp  
00401011    C3              retn  
////几乎一样了, 只是 qword 和 dword 的区别.  
  
  
小结:  
浮点寄存器(FPU,Float Process Register, 存在与 CPU 的 FPU 中)在 IA-32 体系下有 8 个.  
汇编指令用 ST0~ST7 表示. ST0 一般简写作 ST.  
其状态标记我自己测试发现有 4 种:  
Empty: 就是空状态 , 没有使用.  
Valid: 就是正在使用. 这两种最常用.  
Zero: 当 ST 寄存器的值为 0 时,即置此状态.  
Bad: 脏数据的意思, 当我多次 fld 的时候, 超过了 7 次就会覆盖以前.  
浮点寄存器貌似是一种环形的堆栈（或者认为是环形链表）. 可以这样来理解:传统的堆栈看作是移动 esp 指针, 这里的堆栈可看成是指针固定,即 ST0~ST7 这 8 个寄存器始终在我们 OD 窗口中固定, push 和 pop 看成是将环形栈的数据推动或拉动.   
常用指令:  
Fld :用于将一个指定的数据块装载到 STx 寄存器中.(一般为一个 dword 或一个 qword),这里其实是一个 push 的动作.  
格式:后跟指定内存地址和大小. 如:  
Push XXXXXXXX  
Push XXXXXXXX  
Fld qword ptr \[esp\] . //XX … XXX 为编译器对我们的 123.345 …等转化后的数据.  
  
  
Fadd : 浮点加法(同类的有 fsub,fmul,fdiv.)  
如:  
Push XXXXXXXX   
Push XXXXXXXX  
Fld dword ptr\[esp\]  
Fadd dword ptr \[esp+4\]     (这样就默认目的地址为 ST0, 也可以指明为其他 FPR)  
  
Fst: 浮点数储存(store) .: 将 ST 寄存器中的数据储存到内存地址中.(默认为 ST0,也可指明为其他)  
  
Push XXXXXXXX   
Push XXXXXXXX  
Fld qword ptr\[esp\]  
Push ecx  
Push ecx // make space.  
Fst qword ptr \[esp\]  
  
Fstp: 这个后面多了一个 p, 也即 pop 的意思.除了执行 fst 的功能外, 另执行一次 pop 操作.  
  
  
1:\_\_ftol 函数.  
Double d = 123.456;  
Long l = (long)d;  
编译器在处理此句的时候会默认调用 CRT 库的\_\_ftol 来将 double 型的 d 转换为 long 型.  
这个函数\_\_ftol 并未被 CRT 库导出.我们在反汇编中看到的\_\_ftol 都是由于程序中使用了如上格式的类型强制转换.   
这个函数的功能: 将 STx 寄存器中的浮点数转换为相应的 long 型数据然后送入 EAX,EDX 寄存器.  
  
2:floor 函数,(math.h 头文件有声明)  
这个是被 CRT 库导出的函数, 其作用和数学中的高斯函数类似. 原型如下:  
Double floor(double); (可能会存在别的形式.)  
作用:对给定的一浮点数,求出不超过该浮点数的最大”整形”浮点数.  
(比如:floor(123.111… 111)=123.0000…000 , floor(99.123123…) = 99.000 … 000 等)  
  
规则:floor 函数相当于先 fld,再对 STx 寄存器取高斯值.  
  
其他的以后再补充吧.