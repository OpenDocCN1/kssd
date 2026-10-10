

# 逆向一个涵数有些不解,请指导下

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-320.htm`](https://www.pediy.com/kssd/pediy09/pediy09-320.htm)

> 如下  
> .text:00493370 sub\_493370      proc near               ; CODE XREF: sub\_445D00+90p  
> .text:00493370                                         ; sub\_445D00+9Ep ...  
> .text:00493370  
> .text:00493370 var\_C           = dword ptr -0Ch  
> .text:00493370 var\_4           = dword ptr -4  
> .text:00493370  
> .text:00493370                 mov     eax, large fs:0  
> .text:00493376                 mov     cl, byte ptr dword\_A75DA0  
> .text:0049337C                 push    0FFFFFFFFh  
> .text:0049337E                 push    offset loc\_60B26E  
> .text:00493383                 push    eax  
> .text:00493384                 mov     eax, 1  
> .text:00493389                 test    cl, al  
> .text:0049338B                 mov     large fs:0, esp  
> .text:00493392                 jnz     short loc\_4933BA  
> .text:00493394                 or      dword\_A75DA0, eax  
> .text:0049339A                 push    eax  
> .text:0049339B                 mov     ecx, offset unk\_A75D10  
> .text:004933A0                 mov     \[esp+10h+var\_4\], 0  
> .text:004933A8                 call    sub\_4932E0  
> .text:004933AD                 push    offset sub\_8F7140 ; void (\*)(void)  
> .text:004933B2                 call    \_atexit  
> .text:004933B7                 add     esp, 4  
> .text:004933BA  
> .text:004933BA loc\_4933BA:                             ; CODE XREF: sub\_493370+22j  
> .text:004933BA                 mov     ecx, \[esp+0Ch+var\_C\]  
> .text:004933BD                 mov     eax, offset unk\_A75D10  
> .text:004933C2                 mov     large fs:0, ecx  
> .text:004933C9                 add     esp, 0Ch  
> .text:004933CC                 retn  
> .text:004933CC sub\_493370      endp  
>   
> 可能是太菜的原因,反复看了这段代码 mov     eax, large fs:0 可能是传说中的异常处理吧  
> 涵数似乎是判断地址 byte ptr dword\_A75DA0 的字节是否为本,如果为 0 就 or      dword\_A75DA0, eax 把这个字节置为 1,然后进行一些必要的类似初始化(猜的),加之有异常,分析起来晕呼晕呼,不知道我对这涵数的理解是否正确,请指点下 

> 从这行看可能是一个类对象,因为设置 ecx 后并没有直接使用,而是调用了一个函数  
> text:0049339B                 mov     ecx, offset unk\_A75D10  
>   
> class\_type fun()  
> {  
>   static class\_type \_IDonotKnowName;  
>   int nTemp = -1;  
>   char \_cl = dword\_A75DA0;  //这个变量用于控制静态对象只初始化(构造)一次  
>   
>   try  
>   {  
>     if(0 == (\_cl & 0x01))  
>     {  
>       dword\_A75DA0 |= 1;  
>       nTemp = 0;  
>       \_IDonotKnowName.class\_type::class\_type(0); //调用构造函数(call sub\_4932E0)  
>       \_atexit(sub\_8F7140);  //这个是保证静态对象在程序结束时要析构  
>     }  
>   }  
>   catch(...)  
>   {  
>     call loc\_60B26E;  
>   }  
>   
>   return \_IDonotKnowName;  
> }  
>   
> 至于怎么抛出异常,偶就不知道了,代码里看不出来  
> 还有就是上面代码中的那个 nTemp 我也没明白为什么会有这个,不知道是作者代码中的东西,还是编译器搞上去有特殊用途的  
>   
> 作者真实的代码可能是  
> class\_type fun()  
> {  
>   static class\_type \_IDonotKnowName(0);  
>   
>   return \_IDonotKnowName;  
> }  
>   
> 其它的都是编译器加上去的 

> \--- f:\\projects\\classlearning2\\classlearning2\\classlearning2.cpp ---------------  
> // ClassLearning2.cpp : 定义控制台应用程序的入口点。  
> //  
>   
> #include "stdafx.h"  
> class StaticClass  
> {  
> public :  
>   StaticClass(int i)  
> 00401000  push        ecx    
> 00401001  push        esi    
> 00401002  mov         esi,ecx   
>   {  
>     \_\_asm {mov eax,eax}  
> 00401004  mov         eax,eax   
>     StaticClass(1);  
> 00401006  push        1      
> 00401008  lea         ecx,\[esp+0Bh\]   
> 0040100C  call        StaticClass::StaticClass (401000h)   
>   }  
> 00401011  mov         eax,esi   
> 00401013  pop         esi    
> 00401014  pop         ecx    
> 00401015  ret         4      
> \--- 无源文件 -----------------------------------------------------------------------  
> 00401018  int         3      
> 00401019  int         3      
> 0040101A  int         3      
> 0040101B  int         3      
> 0040101C  int         3      
> 0040101D  int         3      
> 0040101E  int         3      
> 0040101F  int         3      
> \--- f:\\projects\\classlearning2\\classlearning2\\classlearning2.cpp ---------------  
>   ~StaticClass()  
>   {  
>   }  
> };  
> StaticClass \* Test()  
> {  
> 00401020  mov         eax,dword ptr fs:\[00000000h\]   
> 00401026  push        0FFFFFFFFh   
> 00401028  push        offset \_\_ehhandler$?Test@@YAPAVStaticClass@@XZ (40186Eh)   
> 0040102D  push        eax    
>   static StaticClass myclass(1);  
> 0040102E  mov         eax,1   
> 00401033  mov         dword ptr fs:\[0\],esp   
> 0040103A  test        byte ptr \[$S1 (40337Ch)\],al   
> 00401040  jne         Test+48h (401068h)   
> 00401042  or          dword ptr \[$S1 (40337Ch)\],eax   
> 00401048  push        eax    
> 00401049  mov         ecx,offset myclass (403378h)   
> 0040104E  mov         dword ptr \[esp+0Ch\],0   
> 00401056  call        StaticClass::StaticClass (401000h)   
> 0040105B  push        offset \`Test'::\`2'::\`dynamic atexit destructor for 'myclass'' (401880h)   
> 00401060  call        atexit (40112Dh)   
> 00401065  add         esp,4   
>   try  
>   {    
>   }catch(...)  
>   {  
>   }  
>   return &myclass;  
> }  
> 00401068  mov         ecx,dword ptr \[esp\]   
> 0040106B  mov         eax,offset myclass (403378h)   
> 00401070  mov         dword ptr fs:\[0\],ecx   
> 00401077  add         esp,0Ch   
> 0040107A  ret                
> \--- 无源文件 -----------------------------------------------------------------------  
> 0040107B  int         3      
> 0040107C  int         3      
> 0040107D  int         3      
> 0040107E  int         3      
> 0040107F  int         3      
> \--- f:\\projects\\classlearning2\\classlearning2\\classlearning2.cpp ---------------  
> int \_tmain(int argc, \_TCHAR\* argv\[\])  
> {  
>   Test();  
> 00401080  call        Test (401020h)   
>   return 0;  
> 00401085  xor         eax,eax   
> }  
> 00401087  ret   
>   
> 从以上代码看的出来  
> 要达到这个编译优化  
> 构造函数应该是传得 1 如果在默认优化设置下  
> 且也不是返回一个对象  
> 而是这个对象的指针。因为如果是对象 会调用拷贝函数  
>   
>   
> VC8 Release 编译  
> LZ 的代码应该是 VC6 的 默认没有 ESP CHECK 

> 真是谢谢楼上 linxer 以及 foxabu 的代码和指点,我最喜欢就是看用代码来解释,很详细,我慢慢消化消化^o^  
> 还有感谢十三上次给我的代码,让我受益非浅  
>   
> 还有个问题  
> 对于  
> .text:00493370 var\_C           = dword ptr -0Ch  
> .text:00493370 var\_4           = dword ptr -4  
> 看似乎是这个涵数中使用的局部变量,但是汇编代码中并没有看到类似 sub esp,0ch 之类的东西,怎么理解这种情况呢 

> .text:0049337C                 push    0FFFFFFFFh  
> .text:0049337E                 push    offset loc\_60B26E  
> .text:00493383                 push    eax 

> 一个是普通变量，另一个被 SHE 拿去用了。  
> so  
> int nTemp = -1;这句是没必要写的，因为是编译器生成的。 

> 这个帖子被加精了,偶就再说两句吧,让它完美点![](img/4ec4a010bb6e6d79731981cd94fbb640.jpg "尴尬")   
>   
> .text:0049337C                 push    0FFFFFFFFh  
> .text:0049337E                 push    offset loc\_60B26E  
> .text:00493383                 push    eax  
>   
> 这三行代码是 vc SEH 异常处理用的,昨天晚上看的时候按标准的异常处理项了,就多出一个 nTemp = -1 了, 这是一个阔展了的 SEH 异常处理项  
>   
> sub esp,0ch 就是跟上面三行代码对着来的  
>   
> 罗云彬的 win32 汇编里 和 看雪的软件加密技术内幕里都有详尽论述 

> why not try to hex-rays? 

> 引用: *最初由 **Kendiv**发布* ![查看帖子](img/showthread.php?p=389683#post389683) why not try to hex-rays? 
> 
> LZ 发帖的时候还没有免费版的 ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

 4dd466d0e1e0cc123afbe504da2462ca