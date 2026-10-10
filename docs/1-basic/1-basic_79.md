

# 浮点运算简介

> 原文：[`www.pediy.com/kssd/pediy05/pediy50039.htm`](https://www.pediy.com/kssd/pediy05/pediy50039.htm)

浮点运算简介   
  
                                        Hume/冷雨飘心   
  
对于习惯于 C 的灵活多变的数据类型和方便的计算那些人而言,了解底层的浮点运算似乎没有什么意义,现在 Visual 盛行的时代还有多少人关心那些所谓的底层呢?   
对了 AfOs 来说,浮点运算是编程中很重要的一部分,因为我们可能会面临一些稍微复杂的运算,如果你和我一样是 Die-hard 的 asm 拥护者,不想轻易用 C 来解决问题,你肯定能想像在 asm 下用整数运算求 sin(2.3)的痛苦,实际上,微机早就为我们准备了解决之道:那就是浮点运算.但现在关于浮点运算的资料较少,相信很多人和我一样还不掌握这种强有力的技术,那好,我们一起来学习学习.   
                        
  
  
                                (一)浮点数   
                                    (This Part mainly Froe Bill's Article)   
在这之前,先来看几个术语:   
FPU->Floating Point Unit,浮点运算部件   
BCD->Binary Coded Decimal 压缩的二十进制数,是用 4 个位来表示数字 0~9,一个 byte 表示两个十进制数,比如 01111001 表示 89   
科学计数法:这是科学的~~~~具体含义查查初中还是小学的数学课本 D:)   
  
浮点运算使用三种不同的数据:   
        1)整数(Integer),又分为字,短整数(Short Integer)和长整数(Long Integer)   
        2)实数(Real)分单精度(Single Real)和双精度(Double Real)   
        3)压缩的二十进制数(BCD)   
      
    下面是其位数(bits)和能表示的大致范围和   
  
  
        Type            Length          Range   
        -----------------------------------------------   
        Word Integer    16 bit          -32768 to 32768   
        Short Integer  32 bit          -2.14e9 to 2.14e9   
        Long Integer    64 bit          -9.22e18 to 9.22e18   
        Single Real    32 bit          1.18e-38 to 3.40e38   
        Double Real    64 bit          2.23e-308 to 1.79e308   
        extended Real  80 bit          3.37e-1932 to 1.18e4932   
        Packed BCD      80 bit          -1e18 to 1e18   
  
双精度数和扩展精度数表示范围对一般应用来说已经足够大了!   
  
1)整数,以补码形式存储,正数的补码是其本身,负数补码是其绝对值的各位变反后加 1,下面是实际存储的例子:   
        0024            var1 dw 24   
        FFFE            var2 dw -2   
        000004D2        var3 dd 1234   
        FFFFFF85        var4 dd -123   
        0000000000002694var5 dq 9876   
        FFFFFFFFFFFFFEBFvar6 dq -321   
  
2)BCD 数   
        在 FPU 中用 80 位表示正好是浮点堆寄存器的宽度,在其格式如下存储:   
    Bit   
        79\_\_\_72\_71\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_0   
        符号            ---18 个二十进制数--------   
看下面的例子:   
        00000000000000012345        var1  dt    12345   
        80000000000000000100        var2  dt    -100   
  
3)浮点数,这个复杂点,有三种格式   
  
        单精度:\_31\_30\_\_\_\_\_\_\_\_23\_22\_\_\_\_\_\_\_\_\_\_\_0   
              符号    指数        有效数   
  
        双精度:\_63\_62\_\_\_\_\_\_\_\_\_\_52\_51\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_0   
              符号    指数            有效数   
  
        扩展精度数:   
              \_79\_78\_\_\_\_\_\_\_\_\_\_\_\_64\_63\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_0   
              符号    指数              有效数   
例子:   
        C377999A                    var1  dd      -247.6   
        40000000                    var2  dd      2.0   
        486F4200                    var3  real4  2.45e+5   
        4059100000000000            var4  dq      100.25   
        3F543BF727136A40            var5  real8  0.00123   
  
        C377999A                    var1  dd      -247.6   
        40000000                    var2  dd      2.0   
        486F4200                    var3  real4  2.45e+5   
        4059100000000000            var4  dq      100.25   
        3F543BF727136A40            var5  real8  0.001235   
        400487F34D6A161E4F76        var6  real10  33.9876   
  
DD 和 real4 都可以在 asm 中来定义单精度浮点数,4 bytes   
DQ 和 real8 都可以在 asm 中来定义双精度浮点数,8 bytes   
DT 和 real10 都可以在 asm 中来定义扩展精度浮点数,10 bytes   
                          
                        (二)浮点部件   
  
FPU 从功能上分为两个部分:控制单元和运算单元,控制单元主要面向 CPU,而算数单元负责具体算数运算.   
FPU 即浮点部件包括 8 个通用寄存器,5 个错误指针寄存器和三个控制寄存器.   
  
1)8 个通用寄存器每个 80 bit,形成一个寄存器堆栈,所有的计算结果都保存在寄存器堆栈中,其中数据全部是 80 位的扩展精度格式,即使是 BCD,整数,单精度和双精度等在装入寄存器的时候都要被 FPU 自动转化为 80 位的扩展精度格式,注意栈顶通常表示为 ST(0),然后是 ST(1)...ST(i),ST(i)是相对于栈顶而言的.   
  
和堆栈很相似,只不过宽度为 80bit,映像如下:   
                \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   
              |        ST(0)          |        
              |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|   
              |        ST(1)          |   
              |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|   
              |    ......            |   
              |    ......            |   
              |        ST(i)          |              
              |\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|   
  
2)控制寄存器,FPU 有三个控制寄存器:状态寄存器,控制寄存器和标记寄存器   
  
状态寄存器->SW   
        \_M\_\_\_\_\_D\_\_\_\_\_\_\_\_10\_\_\_9\_\_\_\_8\_\_\_7\_\_\_\_\_\_\_\_\_5\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_0\_\_   
      |  B |  C3| TOP| C2 | C1 | C0 | ES |    | PE | UE | OE | ZE | DE | IE |   
      |\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|   
  
B:    浮点部件正忙   
C0-C3  指示浮点运算的结果,不同指令有不同含义   
TOP    指示栈顶,通常是 0   
ES 以下任何位置位 (pe, ue, oe, ze, de, or ie) 则置位   
PE 精度故障    
UE 数字太小无法表示溢出    
OE 现有精度无法表示,数字太大溢出    
ZE 除 0 错    
DE 指示至少有一个操作数未规格化    
IE 无效错误,指示堆栈上溢或下溢,无效操作数等   
  
  
控制寄存器:   
        \_15\_\_\_\_\_\_\_\_\_\_\_\_10\_\_\_9\_\_\_\_8\_\_\_7\_\_\_\_\_\_\_\_\_5\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_0\_\_   
      |              |IC | RC | PC  |    | PM | UM | OM | ZM | DM | IM |   
      |\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_|\_\_|\_|\_\_|\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|\_\_\_\_|   
  
IC 无穷大控制,对 486,已经无效   
RC 舍入控制   
        00 = 朝最接近或者偶数舍入   
        01 = 朝负无穷大方向舍入   
        10 = 朝正无穷大方向舍入   
        11 = 超 0 方向截断   
PC 精度控制   
        00 = 单精度   
        01 = 保留   
        10 = 双精度   
        11 = 扩展精度   
PM~IM 屏蔽状态寄存器低 5 位指示的错误.为 1 则屏蔽.   
  
  
标记寄存器:   
        每 2 bit 表示一个对应堆栈寄存器的状态,具体含义如下:   
      15\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_3\_\_\_\_\_0   
      |Tag7 |...................................|tag1|   
      |\_\_\_\_\_|\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|\_\_\_\_|   
          
含义:   
        00 = 有效   
        01 = 零   
        10 = 无效或无穷大   
        11 = 为空   
  
                              

> (三)浮点指令系统及 MASM 下浮点程序设计   
>   
> 事实上最重要和比较难于找到资料在(一)和(二)部分中已经介绍,下面是为了完整性的考虑,如果你是第一次接触浮点指令,看看下面的摘要也无妨.另外本文未涉及到的一个方面是关于浮点处理异常的情况,因为涉及到保护模式和中断、任务切换以及 SEH 等较多内容,我相信介绍之后只会令人更加迷惑,况且我现在似乎也无法把这几个问题完全说清除,一般我们几乎不需要知道这些.让我们先来看主要内容.   
>   
> 关于浮点程序设计是一个大的话题,我只是提纲挈领地简述 Masm32V7(/V6)中的设计方法,因为 486 以上的 CPU 内建了浮点部件所以可以在程序里直接使用浮点指令.下面是一个小例子:   
>   
> \_\_MASMSTD  equ    1   
> .386p   
> .model flat, stdcall   
> option casemap :none  ; case sensitive   
> include c:\\hd\\hd.h   
> include c:\\hd\\mac.h   
>   
> ;;--------------   
>     .DATA   
> num1    dq      12345   
> num2    dq      98765   
> res    dd      0   
>         .DATA?   
> buf    db 200 dup(?)   
>   
> ;;-----------------------------------------   
>     .CODE   
> \_\_Start:   
>         finit                  ;初始化浮点部件   
>         fild    num1            ;装入 num1   
>         fild    num2            ;装入 num2   
>         fmul                    ;执行乘法   
>         fist    res            ;存储   
>         invoke    wsprintf,addr buf,CTEXT("the result is: %ld"),res          
>         invoke    StdOut,addr buf ;显示,注意是控制台显示,编译用/SUBSYSTEM:CONSOLE   
>           
>     invoke    StdIn,addr buf,20   
>         invoke    ExitProcess,0    
> END    \_\_Start   
>   
> 具体你要怎样运用指令,那就得看你自己所要进行的操作和要执行的算法了.注意在 fpu 内部寄存器总是以扩展精度数来表示数值的,因此进行整数运算最后要用 fist 来存储,这样才能得到正确的结果,这些转换是由 fpu 自动完成的.   
>   
> 浮点指令系统分为五类:数据传送类、算术运算类、超越函数类、比较类、环境及系统控制类.   
> 我并不想列出所有函数的参数以及用法,因为这会是劳动力的浪费.我打字用拼音的!:D)具体参考资料见文章最后,别的我就帮不上你了.    
>   
> 1)数据传送类,主要包括   
> 这类指令主要是从内存装入浮点寄存器堆数据,一般目的地址总是栈顶 ST(0),用调试器你可以清除的看到这一点.注意带 P 结尾的操作,是在前面操作完成之后出栈,也就是原来 ST(1)的内容现在成了 ST(0)的内容,注意到这一点,你可以方便地设计出灵活多变的程序.   
> 装入:       
>         FLD    Push real onto stack   
>         FILD    Convert two's complement integer to real and push   
>         FBLD    Convert BCD to real and push to stack   
> 存储:       
>         FST    Store floating-point number from stack   
>         FSTP    Convert top of stack to integer   
>         FIST       
>         FISTP    Convert top of stack to integer   
>         FBSTP    Store BCD to integer and pop stack   
> 交换:       
>         FXCH    Exchange top two stack elements   
> 常数装载:       
>         FLD1    装入常数 1.0   
>         FLDZ    装入常数 0.0   
>         FLDPI    装入常数 pi (=3.1415926....精度足够,放心使用)   
>         FLDL2E    装入常数 log(2)e   
>         FLDL2T    装入常数 log(2)10   
>         FLDLG2    装入常数 log(10)2   
>         FLDLN2    装入常数 Log(e)2   
>   
> 我逼并不想列出所有的浮点指令的详细格式,因为没有必要!很多资料都有这些指令格式的介绍,浮点指令均以 F 开头,LD 表示 Load,ILD 表示整数的 Load,BLD 是二十进制数的 Load,这样记起来就很容易了,很多指令功能都可以根据指令一眼看出来.   
>   
>   
> 2)算术运算类   
> 加法:       
>         FADD/FADDP    Add/add and pop   
>         FIADD    Integer add   
> 减法:       
>         FSUB/FSUBP    Subtract/subtract and pop   
>         FSUBR/FSUBRP    Subtract/subtract and pop with reversed operands   
>         FISUB    Integer subtract   
>         FISUBR    Integer subtract/subtract with reversed operands   
> 乘法:       
>         FMUL/FMULP    Multiply/multiply and pop   
>         FIMUL    Integer multiply   
> 除法:       
>         FDIV/FDIVP    Divide/divide and pop   
>         FIDIV    Integer divide   
>         FDIVR/FDIVRP    Divide/divide and pop with reversed operands   
>         FIDIVR    integer divide with reversed operands   
> 其他:       
>         FABS    Calculate absolute value   
>         FCHS    Change sign   
>         FRNDINT    Round to integer   
>         FSQRT    Calculate square root   
>         FSCALE    Scale top of stack by power of 2   
>         FXTRACT    Separate exponent and mantissa   
>         FPREM    Calculate partial remainder   
>         FPREM1    Calculate partial remainder in IEEE format   
>   
> 如果指令后面未带操作数,其默认的操作数为 ST(0)和 ST(1),关于带 R 后缀的指令是正常操作数的顺序变反,比如 fsub 执行的是 x-y,fsubr 执行的就是 y-x.   
>   
> 3)超越函数类   
> 三角函数       
>         FSIN    Calculate sine   
>         FCOS    Calculate cosine   
>         FSINCOS    Calculate quick sine and cosine   
>         FPTAN    Calculate partial tangent   
>         FPATAN    Calculate partial arctangent   
> Log 类       
>         FYL2X    Calculate y times log base 2 of x   
>         FYL2XP1    Calculate y times log base 2 of (x+1)   
>         F2XM1    Calculate (2^x)-1   
>   
> 4)比较类   
>         FCOM    Compare   
>         FCOMP    Compare and pop   
>         FICOM    Integer compare   
>         FTST    Integer compare and pop   
>         FUCOM    Unordered compare   
>         FUCOMP    Unordered compare and pop   
>         FXAM    Set condition code bits for value at top of stack   
>         FSTSW    Store status word   
>   
> 会根据结果设置,C0~C3,在上面并未就 C0~C3 进行具体介绍,C1 是用来判断上溢或者下溢的,C0 相当于 EFLAGS 里面的 CF,作用也基本一致,C2 相当于 PF,C3 相当于 ZF,你可能会看到如下指令   
>         FSTSW  ax   
>         SAHF   
>         JZ      label   
> 为什么如此呢,因为用如上指令将状态字存入 EFLAGS,C0 正好置于 CF 位,C3 正好置于 ZF 位.   
>   
> 5)环境及系统控制类   
>         FLDCW    Load control word   
>         FSTCW    Store control word   
>         FSTSW    Store status word   
>         FLDENV    Load environment block   
>         FSTENV    Store environment block   
>         FSAVE    Save coprocessor state   
>         FRSTOR    Restore coprocessor state   
>   
>         FINIT    Initialize coprocessor   
>         FCLEX    Clear exception flags   
>         FINCSTP    Increment stack pointer   
>         FDECSTP    Decrement stack pointer   
>         FFREE    Mark element as free   
>         FNOP    No operation   
>         FWAIT    Wait until floating-point instruction complete   
>   
> 我实在不想罗嗦了,因为这些指令的格式以及用法在 Masm32V7 的 help 目录下面的 fphelp.hlp 文件中有详细的说明,当然也还有很多其他的指令格式列表,我之所以列出来是为了完整性.这里还有一个比较困难的问题就是浮点数的显示,windows 没有现成的函数调用,wsprintf 只能显示整数,但有好多库支持,比如 LYB 主页上的浮点开发包,当然等你搞熟了这些东西,也可以自己写.   
>   
> 关于浮点程序的调试,建议使用 Softice,因为 Trw 不支持浮点堆栈的显示,现在网上有一个 fpu 插件,可以部分解决问题,不过不够好用.一切看你自己的选择了.  4dd466d0e1e0cc123afbe504da2462ca