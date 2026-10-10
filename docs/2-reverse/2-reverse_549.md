

# 编译器优化分析-逆向工程和破解适用

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-226.htm`](https://www.pediy.com/kssd/pediy08/pediy8-226.htm)

DFCGVC 编译器优化分析-逆向工程和破解适用。\[color\]  
EVE\[color=#008000\]跃迁。。5 个跳跃，，趁这个时间我写点东西。  
希望能够对编程不是很熟悉的逆向工程者一些帮助  
一样以 VC8 编译器为例，有可能穿插一点 Inter 编译器。  
Release 为主。 Debug 版本基本都看得懂而且，一般也不会用 Debug 版本发布软件。  
1.  先从函数框架说起走  
代码这样子：  
  
#include "stdafx.h"  
#include <stdlib.h>  
void study1()  
  {  
  \_\_asm int 3  
  char str\[10\];  
  gets(str);  
  printf("你输入的内容是\\n%s",str);  
  \_\_asm int 3  }  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
  {  
  study1();  
  system("PAUSE");  
  return 0;  
  }  
  
默认情况 Release 编译吧。以两个\_\_ASM int 3 之间为代码  
汇编代码：  
#include "stdafx.h"  
#include <stdlib.h>  
void study1()  
  {  
  \_\_asm int 3  
  char str\[10\];  
  gets(str);  
  printf("你输入的内容是\\n%s",str);  
  \_\_asm int 3  
  }  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
  {  
00401000  sub         esp,10h   
00401003  mov         eax,dword ptr \[\_\_\_security\_cookie (403000h)\]   
00401008  xor         eax,esp   
0040100A  mov         dword ptr \[esp+0Ch\],eax   
  study1();  
0040100E  int         3      
0040100F  lea         eax,\[esp\]   
00401012  push        eax    
00401013  call        dword ptr \[\_\_imp\_\_gets (4020A4h)\]   
00401019  lea         ecx,\[esp+4\]   
0040101D  push        ecx    
0040101E  push        offset string "\\xc4\\xe3\\xca\\xe4\\xc8\\xebbbbbbbbbbbb" (4020F4h)   
00401023  call        dword ptr \[\_\_imp\_\_printf (40209Ch)\]   
00401029  add         esp,0Ch   
0040102C  int         3      
  system("PAUSE");  
0040102D  push        offset string "PAUSE" (402108h)   
00401032  call        dword ptr \[\_\_imp\_\_system (402098h)\]   
  return 0;  
  }  
00401038  mov         ecx,dword ptr \[esp+10h\]   
0040103C  add         esp,4   
0040103F  xor         ecx,esp   
00401041  xor         eax,eax   
00401043  call        \_\_security\_check\_cookie (40104Ch)   
00401048  add         esp,10h   
0040104B  ret                
我们用默认选项都可以看得出函数已经被内联了。  
00401000  sub         esp,10h   
分配 16 字节。 这里注意一下，一般的其他语言可能也多半是会用函数框架也就是 push esp  
Mov ebp,esp  
然后靠 ebp 来引用局部变量，这一点在 VC 中是没有的。  
你一定会觉得很奇怪明明分配的 10 字节怎么会变成了 16 字节。  
答案是这样的。  
00401003  mov         eax,dword ptr \[\_\_\_security\_cookie (403000h)\]   
00401008  xor         eax,esp   
0040100A  mov         dword ptr \[esp+0Ch\],eax  
保存 esp 函数指针防止堆溢出。主要是在 VC 中靠 esp 引用局部变量所以千万错不得。。  
但是还有两字节呢？ 是这样的。VC 中要进行指针对其 也就是说一般情况下会使 4 的倍数。  
试验一下：  
改为：  
  char str\[12\];  
  gets(str);  
代码依然是  
00401000  sub         esp,10h   
00401003  mov         eax,dword ptr \[\_\_\_security\_cookie (403000h)\]   
00401008  xor         eax,esp   
0040100A  mov         dword ptr \[esp+0Ch\],eax  
  
下面再来说说：  
2.  关于 ecx 指针。在 VC 中虽然可以指定链接方式也就是 C 样式还是标准 API 样式或者是 Fastcall  
不过一般而言是 C 样式 Fastcall 相对用的较少。另外还有 thiscall 方式。也就是类指针。我们来看看 MSDN  
The \_\_thiscall calling convention is used on member functions and is the default calling convention used by C++ member functions that do not use variable arguments. Under \_\_thiscall, the callee cleans the stack, which is impossible for vararg functions. Arguments are pushed on the stack from right to left, with the this pointer being passed via register ECX, and not on the stack, on the x86 architecture.  
ThisCall 的定义如上所示  
我们可以看到考 ecx 传递指针，参数有调用者恢复，以至于可以传递参数不定的函数。  
还是来看段代码  
void study2()  
  {  
  \_\_asm int 3  
  CString MyStr;  
  MyStr="PeDiy By Fox";  
  puts(MyStr);  
  \_\_asm int 3  
  }  
为了方便研究我们去掉了 esp 检查和 C++异常  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
  {  
00401040  push        ecx    
00401041  push        esi    
00401042  push        edi  保护寄存器就不说了  
  //study1();  
  study2();  
00401043  int         3      
00401044  mov         eax,dword ptr \[ATL::g\_strmgr (403380h)\]   
00401049  mov         edx,dword ptr \[eax+0Ch\]   
0040104C  mov         ecx,offset ATL::g\_strmgr (403380h)   
00401051  call        edx    
00401053  add         eax,10h   
00401056  mov         dword ptr \[esp+8\],eax   
0040105A  mov         edi,0Ch   
0040105F  lea         eax,\[esp+8\]   
00401063  call        ATL::CSimpleStringT<char,0>::SetString (4010B0h)   
00401068  mov         esi,dword ptr \[esp+8\]   
0040106C  push        esi    
0040106D  call        dword ptr \[\_\_imp\_\_puts (4020C8h)\]   
00401073  add         esp,4   
00401076  int         3      
00401077  lea         eax,\[esi\-10h\]   
0040107A  lea         ecx,\[eax+0Ch\]   
0040107D  or          edx,0FFFFFFFFh   
00401080  lock xadd   dword ptr \[ecx\],edx   
00401084  dec         edx    
00401085  test        edx,edx   
00401087  pop         edi    
00401088  pop         esi    
00401089  jg          main+55h (401095h)   
0040108B  mov         ecx,dword ptr \[eax\]   
0040108D  mov         edx,dword ptr \[ecx\]   
0040108F  push        eax    
00401090  mov         eax,dword ptr \[edx+4\]   
00401093  call        eax    
  system("PAUSE");  
00401095  push        offset string "PAUSE" (402160h)   
0040109A  call        dword ptr \[\_\_imp\_\_system (4020C4h)\]   
  return 0;  
004010A0  xor         eax,eax   
  }  
004010A2  add         esp,8   
004010A5  ret           
注意，这里不是什么类型的 CALL 而是为了减小体积而来的代码内联，不存在保护寄存器之类的问题。  
  
00401044  mov         eax,dword ptr \[ATL::g\_strmgr (403380h)\]   
00401049  mov         edx,dword ptr \[eax+0Ch\]   
0040104C  mov         ecx,offset ATL::g\_strmgr (403380h)   
00401051  call        edx    
后面的就不详细分析。我要说的是。最后一句 004010A2  add         esp,8   
并不代表局部变量为 2 个 DWORD 而是只有一个。  
原因是这样的，由于才用 ESP 传递指针，所以任何 push pop 操作都会影响指针。  
而最前面就有三个 PUSH 最后面对应的寄存器却只有两个。POP   
那么实际上就只有一个变量而本身 call        dword ptr \[\_\_imp\_\_system (4020C4h)\]就要恢复 add esp,4  再加上一个指针变量 就刚好是 add esp,8，可见编译器的高度优化。把保护寄存器和分配局部变量合并了、。令人佩服。  
不怎么好读得懂。。。：（   
再说一下  
3：结构和数组。  
struct test  
{  
int a;  
char b;  
};  
void study3()  
  {  
  
    \_\_asm int 3  
    DWORD MyStr\[\]={'PeDi','Y\\0'};  
    test MyTest;  
    MyTest.a='LOVE';  
    MyTest.b='\\0';  
    puts((char \*)&MyTest);  
    puts((char \*)MyStr);  
  \_\_asm int 3  
  }00401020   /$  83EC 10             sub esp,10  
00401023   |.  56                  push esi  
00401024   |.  CC                  int3  
00401025   |.  8B35 C4204000       mov esi,dword ptr ds:\[<&MSVCR80.puts>\]                   ;  MSVCR80.puts  
0040102B   |.  8D4424 04           lea eax,dword ptr ss:\[esp+4\]  
0040102F   |.  50                  push eax                                                 ; /s  
00401030   |.  C74424 10 69446550  mov dword ptr ss:\[esp+10\],50654469                       ; |  
00401038   |.  C74424 14 00590000  mov dword ptr ss:\[esp+14\],5900                           ; |  
00401040   |.  C74424 08 45564F4C  mov dword ptr ss:\[esp+8\],4C4F5645                        ; |  
00401048   |.  C64424 0C 00        mov byte ptr ss:\[esp+C\],0                                ; |  
0040104D   |.  FFD6                call esi                                                 ; \\puts  
0040104F   |.  8D4C24 10           lea ecx,dword ptr ss:\[esp+10\]  
00401053   |.  51                  push ecx                                                 ; /s  
00401054   |.  FFD6                call esi                                                 ; \\puts  
00401056   |.  83C4 08             add esp,8  
00401059   |.  CC                  int3  
0040105A   |.  68 50214000         push 编译器学.00402150                                       ; /command = "PAUSE"  
0040105F   |.  FF15 C0204000       call dword ptr ds:\[<&MSVCR80.system>\]                    ; \\system  
00401065   |.  83C4 04             add esp,4  
00401068   |.  33C0                xor eax,eax  
0040106A   |.  5E                  pop esi  
0040106B   |.  83C4 10             add esp,10  
0040106E   \\.  C3                  retn  
分配 10 个字节， 后面就是对堆操作. 但值得注意的是  
0040102B   |.  8D4424 04          lea eax,dword ptr ss:\[esp+4\]  
0040102F   |.  50                 push eax                                                 ; /s  
00401030   |.  C74424 10 50654469 mov dword ptr ss:\[esp+10\],69446550                       ; |  
00401038   |.  C74424 14 59000000 mov dword ptr ss:\[esp+14\],59                             ; |  
00401040   |.  C74424 08 4C4F5645 mov dword ptr ss:\[esp+8\],45564F4C                        ; |  
00401048   |.  C64424 0C 00       mov byte ptr ss:\[esp+C\],0                                ; |  
第一句没有错，但为什么后面没有对\[esp+4\]操作了。。push eax。。这个。 因此相应的就应该是  
从 esp+8 开始  
以及对其。即使是 CHAR 也是一个 INT 所以在 WINDOWS 编程中应当尽量就行 4 字节操作（64 位为 8 字节）  
今天就讲到这么里  
以后我会介绍 算数运算的编译器优化，以及条件语句。 

DFCGVC 编译器优化分析-2  
今天讲讲常见的算术运算的优化：  
1.  先来看看加减法运算  
\_\_asm int 3  
    int a,b;  
  scanf("%d %d",&a,&b);  
  int c;  
  int d;  
  c=a+b;  
  d=a-b;  
  printf("\\n%d %d",c,d);  
  \_\_asm int 3  
汇编代码如下  
00401020   /$  83EC 08            sub esp,8  
00401023   |.  CC                 int3  
00401024   |.  8D4424 04          lea eax,dword ptr ss:\[esp+4\]  
00401028   |.  50                 push eax  
00401029   |.  8D4C24 04          lea ecx,dword ptr ss:\[esp+4\]  
0040102D   |.  51                 push ecx  
0040102E   |.  68 50214000        push 编译器学.00402150                                       ; /format = "%d %d"  
00401033   |.  FF15 C4204000      call dword ptr ds:\[<&MSVCR80.scanf>\]                     ; \\scanf  
00401039   |.  8B4424 0C          mov eax,dword ptr ss:\[esp+C\]  
0040103D   |.  8B4C24 10          mov ecx,dword ptr ss:\[esp+10\]  
00401041   |.  8BD0               mov edx,eax  
00401043   |.  2BD1               sub edx,ecx  
00401045   |.  52                 push edx                                                 ; /<%d>  
00401046   |.  03C1               add eax,ecx                                              ; |  
00401048   |.  50                 push eax                                                 ; |<%d>  
00401049   |.  68 58214000        push 编译器学.00402158                                       ; |format = "  
%d %d"  
0040104E   |.  FF15 C8204000      call dword ptr ds:\[<&MSVCR80.printf>\]                    ; \\printf  
00401054   |.  83C4 18            add esp,18  
00401057   |.  CC                 int3  
  
虽然我们声明了 4 个变量。但编译器会认为这等价于  
printf("\\n%d %d",a+b,c+d);  
因此也只给我们分配了两个变量。  
其实这也告诉编程序的人，没有必要写成 printf("\\n%d %d",a+b,c+d);  
形式，这种形式显然更难维护和阅读。  
00401039   |.  8B4424 0C          mov eax,dword ptr ss:\[esp+C\]  
0040103D   |.  8B4C24 10          mov ecx,dword ptr ss:\[esp+10\]  
00401041   |.  8BD0               mov edx,eax  
00401043   |.  2BD1               sub edx,ecx  
00401045   |.  52                 push edx                                                 ; /<%d>  
00401046   |.  03C1               add eax,ecx                                              ; |  
这里比较巧妙由于涉及到两次运算，因此将两个变量统一读到两个寄存器中，而不象有些编译器会使用内存相加。  
加减法没有什么好说的。主要说乘除法：  
2.  乘除法  
先来看乘法运算：  
首先是有符号运算。  
void study5()  
  {  
  \_\_asm int 3  
    int a,b;  
  scanf("%d %d",&a,&b);  
  int c,d,e;  
  c=a\*2;  
  d=a\*b;  
  e=a\*7;  
  printf("\\n%d %d",c,d,e);  
  \_\_asm int 3    
}  
00401023   |.  CC                 int3  
00401024   |.  8D4424 04          lea eax,dword ptr ss:\[esp+4\]  
00401028   |.  50                 push eax  
00401029   |.  8D4C24 04          lea ecx,dword ptr ss:\[esp+4\]  
0040102D   |.  51                 push ecx  
0040102E   |.  68 50214000        push 编译器学.00402150                                 ; /format = "%d %d"  
00401033   |.  FF15 C4204000      call dword ptr ds:\[<&MSVCR80.scanf>\]               ; \\scanf  
00401039   |.  8B4424 0C          mov eax,dword ptr ss:\[esp+C\]  
0040103D   |.  8BC8               mov ecx,eax  
0040103F   |.  0FAF4C24 10        imul ecx,dword ptr ss:\[esp+10\]  
00401044   |.  8D14C5 00000000    lea edx,dword ptr ds:\[eax\*8\]  
0040104B   |.  2BD0               sub edx,eax  
0040104D   |.  52                 push edx  
0040104E   |.  51                 push ecx                                           ; /<%d>  
0040104F   |.  8D1400             lea edx,dword ptr ds:\[eax+eax\]                     ; |  
00401052   |.  52                 push edx                                           ; |<%d>  
00401053   |.  68 58214000        push 编译器学.00402158                                 ; |format = "  
%d %d"  
00401058   |.  FF15 C8204000      call dword ptr ds:\[<&MSVCR80.printf>\]              ; \\printf  
0040105E   |.  83C4 1C            add esp,1C  
00401061   |.  CC                 int3  
  
0040103F   |.  0FAF4C24 10        imul ecx,dword ptr ss:\[esp+10\]  
没有什么说的，整数乘法  
00401044   |.  8D14C5 00000000    lea edx,dword ptr ds:\[eax\*8\]  
0040104B   |.  2BD0               sub edx,eax  
呵呵。看到这里是否想到我们小学的时候得简便运算。我们是乘以 7  
但是编译器给我们编程了乘以 8 在减掉被乘数。神奇吧。  
同样的道理  
0040104F   |.  8D1400             lea edx,dword ptr ds:\[eax+eax\]                     ; |  
乘以二就是两个这个数相加。。想象力丰富吧。  
总结一下载 VC 中乘法如果在已知被除数的情况下用  
lea edx,dword ptr ds:\[eax\*x\]的形式，  
否则采用 IMUL 指令。  
现在我们把代码变成这样  
c=a\*-2;  
  d=a\*-4;  
  e=a\*-7;  
测试一下有符号运算。  
00401033   |.  FF15 C4204000      call dword ptr ds:\[<&MSVCR80.scanf>\]               ; \\scanf  
00401039   |.  8B4424 0C          mov eax,dword ptr ss:\[esp+C\]  
0040103D   |.  8D14C5 00000000    lea edx,dword ptr ds:\[eax\*8\]  
00401044   |.  8BC8               mov ecx,eax  
00401046   |.  2BD0               sub edx,eax  
00401048   |.  F7D9               neg ecx  
0040104A   |.  F7DA               neg edx  
0040104C   |.  52                 push edx  
0040104D   |.  03C9               add ecx,ecx  
0040104F   |.  03C9               add ecx,ecx  
00401051   |.  F7D8               neg eax  
00401053   |.  51                 push ecx                                           ; /<%d>  
00401054   |.  03C0               add eax,eax                                        ; |  
00401056   |.  50                 push eax                                           ; |<%d>  
00401057   |.  68 58214000        push 编译器学.00402158                                 ; |format = "  
%d %d"  
0040105C   |.  FF15 C8204000      call dword ptr ds:\[<&MSVCR80.printf>\]              ; \\printf  
  
代码有些变化 除了用了 NEG 指令以外。还有  
原来的  
lea edx,dword ptr ds:\[eax+eax\]                     ; |  
变成了  
0040104D   |.  03C9               add ecx,ecx  
0040104F   |.  03C9               add ecx,ecx  
  
不过大体上没有什么好说的。  
乘法总的来说都比较好识别  
代码在改一改 改为  
  d=short(a\*b);  
  c=c\*c\*c;  
  e=a\*100;  
我们来看看类型转换的问题  
其他的都差不多主要说一下  
0040104E   |.  0FBFD1             movsx edx,cx  
00401051   |.  8BC8               mov ecx,eax  
00401053   |.  0FAFC8             imul ecx,eax  
00401056   |.  0FAFC8             imul ecx,eax  
如果是 short 那么就会用 movsx 符号取得低位  
c=c\*c\*c---〉imul ecx,eax；imul ecx,eax  
乘法总的来说都比较容易识别也比较简单。  
除法就不一样了，无符号有符号完全不同，另外以及不同编译器的处理也很不一样  
且涉及到求模运算和除法运算两种。  
由于 CPU 中只有加法器和乘法器 所以一般而言除法会转变为乘法运算。  
我们先来看两组最简单的。  
先讲无符号的明天再说有符号的  
代码：  
  \_\_asm int 3  
  unsigned  int a,b;  
  scanf("%d %d",&a,&b);  
  unsigned int c,d,e;  
  d=a/2;  
  c=b/4;  
  printf("\\n%d %d",c,d);  
  \_\_asm int 3  
生成代码（部分）  
00401039   |.  8B5424 0C            mov edx,dword ptr ss:\[esp+C\]  
0040103D   |.  8B4424 10            mov eax,dword ptr ss:\[esp+10\]  
00401041   |.  D1EA                 shr edx,1  
00401043   |.  52                   push edx                                     ; /<%d>  
00401044   |.  C1E8 02              shr eax,2                                    ; |  
当乘以 2 的 n 次方的时候可以采用位移的方式。  
Shr 向右移  
其实就是一个乘方关系不是很难理解，但是要大家注意，不要看着 shr 就想着用 〉〉指令可读性肯定没有用/??好  
但是如果是求模式不同的  
看代码：  
d=a%2;  
  c=b%4;  
会变为  
00401039   |.  8B5424 0C            mov edx,dword ptr ss:\[esp+C\]  
0040103D   |.  8B4424 10            mov eax,dword ptr ss:\[esp+10\]  
00401041   |.  83E2 01              and edx,1  
00401044   |.  52                   push edx                                     ; /<%d>  
00401045   |.  83E0 03              and eax,3                                    ; |  
00401048   |.  50                   push eax                                     ; |<%d>  
00401049   |.  68 58214000          push 编译器学.00402158                           ; |format = "  
%d %d"  
即为 AND 运算。其实也比较好理解  
我们随便举个例子：  
比如 17 的二进制代码为        10001  
如果是除以 2 那么就是 AND     00001  
留下最后一位，  
如果是除以 4 那么就是 AND     00011  
余下最后两位。其实我们想象成 10 进制就好理解了  
比如 12345 mod 10 肯定就是 5 也就是保留个位  
12345 mod 100 肯定就是 45 保留十位  
二进制是同样一个道理当是 2 的 N 次方的时候就和 10 的 N 次方的时候是一样的  
也就是去掉 N 位的前面所有位就是余数  
这一点希望大家注意 。很多时候为什么我们在写注册机的时候往往会出现很多位操作原因就是我们对这些理解不是很到位。而且位操作也不便于反退注册过程。  
今天就讲到这里  
  
除法部分比较复杂。  
  
谢 谢  
   By Fox

 4dd466d0e1e0cc123afbe504da2462ca