

# 软件的注册码浮点算法的分析！

> 原文：[`www.pediy.com/kssd/pediy11/122413.html`](https://www.pediy.com/kssd/pediy11/122413.html)

今天是九九重阳节，杭州的钱塘江准备放三十万的烟花，到时一定很漂亮，本来一个朋友叫我去看的，只是公司说要一起去郊游，所以就没去了，遗憾![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")~~没办法，有时候鱼与熊掌不可兼得，有得必有失吧，失去了去看烟花的时间，却可以带领大家一起分析一下汇编的浮点运算的一些知识，自己也学到不少，因为自己以前也没有接触过浮点运算，只是听说过，这还是第一次跟踪，文章很简单，但对于没有接触过浮点运算的朋友，希望会有一些帮助！  
  
\[软件名称\]不知不觉背单词，网上有很多下载地址，大家可以自己搜索下载！(网上好像已经有了破解版！）注明：此文仅用于技术交流，尊重原作者版权！其实别人花那么多精力开发个软件也不容易啦！![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")  
  
分析这款软件我花了半天的时间，从爆破----->到分析出注册码------->到算法的分析！完全使用 OD 进行分析跟踪，没有使用 IDA，主要的原因是因为这款软件比较简单，没有必要，杀鸡何必用牛刀！呵呵~~![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")  
  
这款软件很简单，也没有加壳，所以直接 OD 载入分析就得了，前面的爆破和注册码的分析太简单了，我想在这里讲也不太合适，如果大家有什么不懂的，多去初学者版块去学习学习，自然就会了，我这里主要是想分析一下这款软件的使用的浮点算法部分，可以去浮点运算比较熟悉了，飘过~~好了，说了这么多进入正题吧！  
  
我先科普一下，汇编中的浮点运算规则吧，以免在后面的分析中大家找不到东南西北！  
网上也有很多讲解浮点运算的资料，我这里给大家收集整理对我们有帮助的一些，如果要学习详解的资料，请大家自己搜索学习，《Inter 汇编语言程序设计》第五版也有讲的很详细！  
  
浮点执行环境的寄存器主要是 8 个通用数据寄存器和几个专用寄存器，它们是状态寄存器、控制寄存器、标记寄存器等（这些大家可以在用 OD 调试的时候，看寄存器窗口就明白了！）  
  
8 个浮点数据寄存器(FPU Data Register)，编号 FPR0~FPR7。每个浮点寄存器都是 80 位，以扩展精度格式存储数据，当其他类型数据压入数据寄存器时，FPU 自动转换成扩展精度，相反，数据寄存器的数据取出时，系统也会自动转换成要求的数据类型。  
  
8 个浮点数据寄存器组成首尾相接的堆栈，当前栈顶 ST0 指向的 FPUX 由状态寄存器中 TOP 字段指明。数据寄存器不采用随机存取，而是按照“后进先出”的堆栈原则工作，并且首尾循环。向数据寄存器传送(Load)数据时是入栈，堆栈指针 TOP 先减 1，再将数据压入栈顶寄存器；从数据寄存器取出数据时就是出栈，先将栈顶寄存器数据弹出，再修改堆栈指针使 TOP 加 1。浮点寄存器还有首尾循环相连的特点。例如，若当前栈顶 TOP = 0（即 ST0 = FPR0),那么，入栈操作后就使 TOP= 7（即使 ST0=FPR7)，数据被压入 FPR7。所以，浮点数据寄存据常常被称为浮点数据堆栈！  
  
浮点运算指令有哪些？  
浮点传送指令  
  浮点数据传送指令完成主存与堆栈顶 ST0、数据寄存器 STX 与堆栈顶之间的浮点格式数据的传送。浮点数据寄存器是一个首尾相接的堆栈，所以它的数据传送实际上是对堆栈的操作，有些要改变堆栈指针 TOP，即修改当前栈顶。  
算术运算类指令  
  这类浮点指令实现浮点数、16/32 位整数的加、减、乘、除运算，它们支持的寻址方式相同。这组指令还包括有关算术运算的指令，倒如求绝对值、取整等。  
超越函数类指令  
  浮点指令集中包含有进行三角函数、指数和对数运算的指令。  
浮点比较类指令  
  浮点比较指令比较栈顶数据与指定的源操作数，比较结果通过浮点状态寄存器反映。  
FPU 控制类指令  
  FPU 控制类指令用于控制和检测浮点处理单元 FPU 的状态及操作方式。  
定义浮点数据  
  数据定义指令 DWORD,QWORD,TBYTE 依次说明 32/64/80 位数据；它们可以用于定义单精度、双精度和扩展精度浮点数，但不能出现纯整数（其实，整数后面补个小数就可以了）。相应的数据属性依次是 DWORD,QWORD,TBYTE。另外，实常数可以用 E 表示 10 的幂。  
  每当执行一个新的浮点程序时，第一条指令都应该是初始化 FPU 的指令 finit。该指令清除浮点数据寄存器堆栈和异常，为程序提供一个“干净”的初始化状态。否则，遗留在浮点寄存器堆栈中的数据可能会产生堆栈溢出。另一方面，浮点指令程序段结束，也最好清空浮点数据寄存器。  
  
汇编浮点运算指令集  
对下面的指令先做一些说明：  
ST(i)：代表浮点寄存器，所说的出栈，入栈操作都是对 ST(i)的影响。  
SRC,DST,DEST,OP 等指令是指指令的操作数，SRC 表示源操作数，DST/DEST 表示目的操作数。  
MEM8,MEM16,MEM64,MEM80 等表示是内存操作数，后面的数值表示该操作数的内存位数（8 位为一字节）。  
X<----Y 表示将 Y 的值放入 X，例 ST0<-----ST0－ST1，表示将 ST0－ST1 的值放入浮点寄存器 ST0  
  
一些常用的指令：  
指令格式       指令含义            执行的操作   
FLD src     装入实数到 st(0)    st(0) <- src (mem32/mem64/mem80)    
FILD src    装入整数到 st(0)    st(0) <- src (mem16/mem32/mem64)   
FBLD src    装入 BCD 数到 st(0)   st(0) <- src (mem80)   
FLDZ        将 0.0 装入 st(0)     st(0) <- 0.0    
FLD1        将 1.0 装入 st(0)     st(0) <- 1.0    
FLDPI       将 pi 装入 st(0)      st(0) <- ?(ie, pi)   
FLDL2T     将 log2(10)装入 st(0) st(0) <- log2(10)   
FLDL2E     将 log2(e)装入 st(0)  st(0) <- log2(e)   
FLDLG2     将 log10(2)装入 st(0) st(0) <- log10(2)   
FLDLN2     将 loge(2)装入 st(0)  st(0) <- loge(2)   
FST dest  保存实数 st(0)到 dest  dest <- st(0) (mem32/mem64)   
FSTP  dest dest <- st(0)   (mem32/mem64/mem80)；然后再执行一次出栈操作   
FIST dest 将 st(0)以整数保存到 dest dest <- st(0) (mem32/mem64)    
FISTP dest   dest <- st(0) (mem16/mem32/mem64)；然后再执行一次出栈操作   
FBST dest 将 st(0)以 BCD 保存到 dest dest <- st(0) (mem80)    
FBSTP dest dest<- st(0) (mem80)；然后再执行一次出栈操作  
2．  比较指令  
指令格式         指令含义           执行的操作   
FCOM             实数比较    将标志位设置为 st(0) - st(1) 的结果标志位   
FCOM op          实数比较  将标志位设置为 st(0) - op (mem32/mem64)的结果标志位   
FICOM op       和整数比较  将 Flags 值设置为 st(0)-op 的结果 op (mem16/mem32)  
FICOMP op     和整数比较 将 st(0)和 op 比较 op(mem16/mem32)后；再执行一次出栈操作  
FTST             零检测   将 st(0)和 0.0 比较   
FUCOM st(i)   比较 st(0) 和 st(i)                
FUCOMP st(i)  比较 st(0) 和 st(i)，并且执行一次出栈操作   
FUCOMPP st(i) 比较 st(0) 和 st(i)，并且执行两次出栈操作  
FXAM  Examine: Eyeball st(0) (set condition codes)  
3．  运算指令  
指令格式          指令含义                执行的操作  
加法  
FADD              加实数           st(0) <-st(0) + st(1)  
FADD src   st(0) <-st(0) + src (mem32/mem64)   
FADD st(i),st   st(i) <- st(i) + st(0)   
FADDP st(i),st  st(i) <- st(i) + st(0)；然后执行一次出栈操作   
FIADD src   加上一个整数   st(0) <-st(0) + src (mem16/mem32)  
减法   
FSUB        减去一个实数   st(0) <- st(0) - st(1)   
FSUB src    st(0) <-st(0) - src (reg/mem)   
FSUB st(i),st   st(i) <-st(i) - st(0)   
FSUBP st(i),st  st(i) <-st(i) - st(0)，然后执行一次出栈操作   
FSUBR st(i),st  用一个实数来减   st(0) <- st(i) - st(0)   
FSUBRP st(i),st   st(0) <- st(i) - st(0)，然后执行一次出栈操作   
FISUB src 减去一个整数 st(0) <- st(0) - src (mem16/mem32)   
FISUBR src 用一个整数来减 st(0) <- src - st(0) (mem16/mem32)   
乘法  
FMUL 乘上一个实数 st(0) <- st(0) \* st(1)   
FMUL st(i) st(0) <- st(0) \* st(i)   
FMUL st(i),st  st(i) <- st(0) \* st(i)   
FMULP st(i),st  st(i) <- st(0) \* st(i)，然后执行一次出栈操作   
FIMUL src 乘上一个整数 st(0) <- st(0) \* src (mem16/mem32)   
除法  
FDIV 除以一个实数  st(0) <-st(0) /st(1)   
FDIV st(i)  st(0) <- st(0) /t(i)   
FDIV st(i),st st(i) <-st(0) /st(i)   
FDIVP st(i),st  st(i) <-st(0) /st(i)，然后执行一次出栈操作   
FIDIV src 除以一个整数 st(0) <- st(0) /src (mem16/mem32)   
FDIVR st(i),st 用实数除 st(0) <- st(i) /st(0)   
FDIVRP st(i),st    
FDIVRP st(i),st  
FIDIVR src  用整数除 st(0) <- src /st(0) (mem16/mem32)   
FSQRT   平方根    st(0) <- sqrt st(0)   
FSCALE  2 的 st(0)次方 st(0) <- 2 ^ st(0)   
FXTRACT Extract exponent: st(0) <-exponent of st(0); and gets pushed   
st(0) <-significand of st(0)   
FPREM  取余数 st(0) <-st(0) MOD st(1)  
FPREM1 取余数（IEEE），同 FPREM，但是使用 IEEE 标准\[486\]  
FRNDINT 取整（四舍五入） st(0) <- INT( st(0) ); depends on RC flag  
FABS  求绝对值  st(0) <- ABS( st(0) ); removes sign  
FCHS  改变符号位(求负数） st(0) <-st(0)  
F2XM1 计算(2 ^ x)-1  st(0) <- (2 ^ st(0)) - 1  
FYL2X 计算 Y \* log2(X) st(0)为 Y；st(1)为 X；将 st(0)和 st(1)变为 st(0) \* log2( st(1) )的值  
FCOS    余弦函数 Cos  st(0) <- COS( st(0) )  
FPTAN   正切函数 tan st(0) <- TAN( st(0) )  
FPATAN  反正切函数 arctan  st(0) <- ATAN( st(0) )  
FSIN    正弦函数 sin       st(0) <- SIN( st(0) )  
FSINCOS  sincos 函数     st(0) <-SIN( st(0) )，并且压入 st(1)  
st(0) <- COS( st(0) )  
FYL2XP1 计算 Y \* log2(X+1)  st(0)为 Y； st(1)为 X；将 st(0)和 st(1)变为 st(0) \* log2( st(1)+1 )的值  
处理器控制指令  
FINIT  初始化 FPU  
FSTSW AX   保存状态字的值到 AX   AX<- MSW  
FSTSW dest 保存状态字的值到 dest dest<-MSW (mem16)  
FLDCW src  从 src 装入 FPU 的控制字 FPU CW <-src (mem16)  
FSTCW dest 将 FPU 的控制字保存到 dest  dest<- FPU CW  
FCLEX  清除异常  
FSTENV dest  保存环境到内存地址 dest 处保存状态字、控制字、标志字和异常指针的值  
FLDENV src   从内存地址 src 处装入保存的环境  
FSAVE dest   保存 FPU 的状态到 dest 处 94 字节  
FRSTOR src   从 src 处装入由 FSAVE 保存的 FPU 状态  
FINCSTP 增加 FPU 的栈指针值  st(6) <-st(5); st(5) <-st(4),...,st(0) <-?  
FDECSTP 减少 FPU 的栈指针值  st(0) <-st(1); st(1) <-st(2),...,st(7) <-?   
FFREE st(i)  标志寄存器 st(i)未被使用  
FNOP 空操作，等同 CPU 的 nop st(0) <-st(0)  
WAIT/FWAIT   同步 FPU 与 CPU：停止 CPU 的运行，直到 FPU 完成当前操作码  
FXCH  交换指令，交换 st(0)和 st(1)的值   st(0) <-st(1)  st(1) <- st(0)  
  
一些常用的指令也就这些，大家如果记忆不好，也不要紧，用的时候去查就行了，其实也很好记的，无非一些英文单词的缩写吧了~~  
  
前面的找注册码为 600341115 我就不多讲了，很容易找到的，如果有不懂就请先去初学者版块学习学习~~~  
我们找到注册码计算比较那段的段首地址为 004141EC，我们在那里下断点，单步跟踪就可以找到注册码算法了！  
004141EC 处的代码如下：  
004141EC              55                    push ebp  
004141ED              8BEC                  mov ebp,esp  
004141EF              81C4 70FFFFFF         add esp,-90  
004141F5              8955 C0               mov dword ptr ss:\[ebp-40\],edx  
004141F8              8945 C4               mov dword ptr ss:\[ebp-3C\],eax  
.............................................  
  
前面一堆代码基本也没用，只用于将注册对话框中输入的注册码，取出然后转化为字符串，很简单就不讲了！然后我们向下看一直到 00414234 这里，代码如下：  
00414234              8945 BC               mov dword ptr ss:\[ebp-44\],eax  
00414237              33D2                  xor edx,edx  
00414239              8955 B8               mov dword ptr ss:\[ebp-48\],edx  
0041423C              33C9                  xor ecx,ecx  
0041423E              894D B4               mov dword ptr ss:\[ebp-4C\],ecx  
00414241              EB 10                 jmp short Recite.00414253  
00414243              8B45 BC               mov eax,dword ptr ss:\[ebp-44\]  
00414246              8B55 B4               mov edx,dword ptr ss:\[ebp-4C\]  
00414249              0FBE0C10              movsx ecx,byte ptr ds:\[eax+edx\]  
0041424D              014D B8               add dword ptr ss:\[ebp-48\],ecx  
00414250              FF45 B4               inc dword ptr ss:\[ebp-4C\]  
00414253              8D45 FC               lea eax,dword ptr ss:\[ebp-4\]  
00414256              E8 69990800           call Recite.0049DBC4  
0041425B              48                    dec eax  
0041425C              3B45 B4               cmp eax,dword ptr ss:\[ebp-4C\]  
0041425F            ^ 7F E2                 jg short Recite.00414243  
  
经常跟踪软件人，一看会知道这段代码的功能了,其实就是将字符中的字符的各个 ASCII 码进行累加，累加和存入一个局部变量中。  
申请三个局部变量：  
\[ebp-44\]用来存放机器码:2141215019417  
\[ebp-48\]用来存放 ASCII 码之和  
\[ebp-4c\]用来存放取的是上这面这个字符串的第几个字符  
注册码为 600341115  
代码很简单，就是一利用一个循环，然后将机器码的各位的 ASCII 码进行累加！然后将结果存到 ss:\[ebp-48\]中  
也就是 32+31+34+31+32+31+35+30+31+39+34+31+37=25F(十六进制)=607(十进制)  
  
得到了这个值，下面就要开始计算注册码了，OD 有一个很好的功能就是注解功能，我经常用这个功能，因为有时软件比较长，也不是一次就可以分析的出来的，要反反复复不停的跟踪，所以做注释是一个很好的习惯，我想学逆向的朋友们都会有这个好习惯的！  
  
下面我就不多说了，大家看我在 OD 中的注释就会很清楚了！  
下面就一堆浮点计算指令，与很多 CALL，究竟那个 CALL 对自己有用呢？这就要靠经验了，我也说不清楚！  
  
其实我用 IDA 分析过一段代码，很简单，因为 IDA 有一个很强大的功能就是代码的自动识别功能，这里我不能不配服 IDA 的作者！  
我用 IDA 分析了下面的一小段代码，得到如下结果：  
.text:00414261                 fild    \[ebp+var\_48\]  
.text:00414264                 add     esp, 0FFFFFFF8h ; x  
.text:00414267                 fstp    \[esp+98h+var\_98\]  
.text:0041426A                 call    \_cos  
  
其实 0041426A 处就是计算浮点的\_COS 的值  
下面我完全用 OD 去分析注册码的算法，不借助 IDA！分析如下：  
  
上面的 IDA 对应的 OD 代码为  
00414261          DB45 B8        fild dword ptr ss:\[ebp-48\]         ; 将 EBP-48 中的实整数 607 装入到 ST(0)  
00414264          83C4 F8        add esp,-8  
00414267          DD1C24         fstp qword ptr ss:\[esp\]            ; 将 ST(0)的数值装入到 ESP 中  
0041426A          E8 1D2B0800    call Recite.00496D8C  
  
00496D8C 处的代码如下：  
  
00496D8C          55             push ebp  
00496D8D          8BEC           mov ebp,esp  
00496D8F          53             push ebx  
00496D90          DD45 08        fld qword ptr ss:\[ebp+8\]       ; 加载实数到 ST(0)  
00496D93          66:B8 F07F     mov ax,7FF0                    ; 将 7FF0 值传给 AX  
00496D97          66:2345 0E     and ax,word ptr ss:\[ebp+E\]     ; 将 EBP+E 中的 4082 与 AX 的 7FF0 进行与操作  
00496D9B          66:3D 4043     cmp ax,4340                    ; 将与的结果与 4340 进行比较  
00496D9F          73 08          jnb short Recite.00496DA9      ; 跳转没有实现  
00496DA1          E8 E6000000    call Recite.00496E8C  
  
我们在来看看 00496E8C 处的代码如下：  
  
00496E8C          B1 02          mov cl,2                       ; 将 2 传给 Cl  
00496E8E          EB 02          jmp short Recite.00496E92      ; 短跳转  
00496E90          B1 04          mov cl,4  
00496E92          D9E5           fxam                           ; 进行浮点检查  
00496E94          83EC 04        sub esp,4  
00496E97          9B             wait  
00496E98          DD3C24         fstsw word ptr ss:\[esp\]        ; 浮点检查保存状态器 ESP 值为 43C8  
00496E9B          9B             wait  
00496E9C          8A6424 01      mov ah,byte ptr ss:\[esp+1\]     ; 将 ESP+1 中保存的'<'传给 ah  
00496EA0          9E             sahf                           ; 恢复标志低八,AH=3C,S0,Z0,A1,P1,C0,FL=06  
00496EA1          72 0D          jb short Recite.00496EB0       ; CF=1 跳转，所以没跳  
00496EA3          75 2A          jnz short Recite.00496ECF      ; ZF=0 跳转，所以跳转了  
00496EA5          80F9 02        cmp cl,2  
00496EA8          75 04          jnz short Recite.00496EAE  
00496EAA          DDD8           fstp st  
00496EAC          D9E8           fld1  
00496EAE          EB 1A          jmp short Recite.00496ECA  
00496EB0          74 0C          je short Recite.00496EBE  
00496EB2          7B 0A          jpo short Recite.00496EBE  
00496EB4          DDD8           fstp st  
00496EB6          D905 98BF4A00  fld dword ptr ds:\[4ABF98\]  
00496EBC          D9E4           ftst  
00496EBE          EB 0A          jmp short Recite.00496ECA  
00496EC0          DED9           fcompp  
00496EC2          D905 98BF4A00  fld dword ptr ds:\[4ABF98\]  
00496EC8          D9E4           ftst  
00496ECA          E9 97000000    jmp Recite.00496F66  
00496ECF          D9E1           fabs                           ; 求 ST(0)绝对值  
00496ED1          DB2D 84BF4A00  fld tbyte ptr ds:\[4ABF84\]      ; 装载实数到 ST(0)值为 0.7853981633974482944  
00496ED7          D9E5           fxam                           ; 浮点检查  
00496ED9          D9C9           fxch st(1)                     ; 交换 ST(0)与 ST(1)的值 ST(1)为 607.00000000000000000  
00496EDB          D9F8           fprem                          ; ST(0)MODST(1)取余数 0.6726178571699049472  
00496EDD          B5 02          mov ch,2                       ; CH 赋值为 2  
00496EDF          22EC           and ch,ah                      ; 用 2 与上面 AH 的值（'<'）进行与操作，3CAND2  
00496EE1          D0ED           shr ch,1                       ; 与的结果为 0，然后右移一位  
00496EE3          9B             wait  
00496EE4          DD3C24         fstsw word ptr ss:\[esp\]        ; 保存状态字到 ESP  
00496EE7          9B             wait  
00496EE8          8A6424 01      mov ah,byte ptr ss:\[esp+1\]     ; 取 ESP 中的第二位数 1  
00496EEC          9E             sahf                           ; 恢复标志低八位 AH=31(S0,Z0,A1,P0,C1),FL=46  
00496EED          7A D1          jpe short Recite.00496EC0      ; PF=1 时跳转，显然这里不跳  
00496EEF          B0 03          mov al,3                       ; AL 赋值 3  
00496EF1          22C4           and al,ah                      ; AH('1') AND AL=03，结果为 1  
00496EF3          D0E4           shl ah,1                       ; AH 左移一位，31\*2=62('b')  
00496EF5          D0E4           shl ah,1                       ; AH 再左移一位  
00496EF7          D0D0           rcl al,1                       ; AL 循环进位左移，01\*2=02  
00496EF9          04 FC          add al,0FC                     ; AL 与 0FC 与操作  
00496EFB          D0D0           rcl al,1                       ; 将上面得到的结果 AL=FE，循环进位左移  
00496EFD          80F9 02        cmp cl,2                       ; CL 与 2 进行比较  
00496F00          75 04          jnz short Recite.00496F06      ; 不跳  
00496F02          02C1           add al,cl                      ; CL=02 AND AL=FE  
00496F04          B5 00          mov ch,0                       ; CH=0  
00496F06          24 07          and al,7                       ; AL=FE AND 7  
00496F08          A8 01          test al,1                      ; AL=06 检查是否为 1  
00496F0A          74 04          je short Recite.00496F10       ; 跳转  
00496F0C          DEE9           fsubp st(1),st  
00496F0E          EB 02          jmp short Recite.00496F12  
00496F10          DDD9           fstp st(1)                     ; 将 ST(0)--->ST(1)  
00496F12          D9F2           fptan                          ; 正切函数 0.6726178571699049472  
00496F14          80F9 04        cmp cl,4                       ; CL=02 与 4 进行比较  
00496F17          74 20          je short Recite.00496F39       ; 不跳  
00496F19          A8 03          test al,3                      ; AL=06 检查是为 3  
00496F1B          7A 02          jpe short Recite.00496F1F      ; 不跳 P 位为 0  
00496F1D          D9C9           fxch st(1)                     ; 将 ST(0)值 1.0000000000000000000 与 ST(1)值 0.7965240390979069952 进行交换  
00496F1F          D9C1           fld st(1)                      ; 将 ST(1)-->ST(0)  
00496F21          D8C8           fmul st,st                     ; ST(0)\*ST(0)  
00496F23          D9C9           fxch st(1)                     ; 交换 ST(0)值 1 与 ST(1)的值 0.7965240390979069952  
00496F25          D8C8           fmul st,st                     ; ST(0)值 0.7965240390979069952 乘以 ST(0)  
00496F27          DEC1           faddp st(1),st                 ; 得到 ST(0)值 0.6344505448608441344 加上 ST(1)的值 1  
00496F29          D9FA           fsqrt                          ; 求 ST(0)的值 1.6344505448608441344 的平方根得到 1.2784563132390735360  
00496F2B          D0E8           shr al,1                       ; AL 的值右移 1  
00496F2D          D0E8           shr al,1                       ; AL 的值右移 1  
00496F2F          32C5           xor al,ch                      ; 得到 AL=01 与 CH=00 进行异或  
00496F31          74 02          je short Recite.00496F35       ; 不跳  
00496F33          D9E0           fchs                           ; 改变 ST(0)的符号位求负数得到-1.2784563132390735360  
00496F35          DEF9           fdivp st(1),st                 ; ST(1)除以 ST(0)得到-0.7821933292866443264  
00496F37          EB 2D          jmp short Recite.00496F66      ; 跳出这个 CALL  
  
最后得到 ST(0)的值为-0.7821933292866443264  
  
我后返回到 00496DA6 处，接下来往下看  
00496DA6          5B             pop ebx  
00496DA7          5D             pop ebp  
00496DA8          C3             retn  
  
又一次返回，到 0041426F 处，代码如下所示：  
0041426F          83C4 08        add esp,8  
00414272          D805 34444100  fadd dword ptr ds:\[414434\]        ; 将得到的 ST(0)的值-0.7821933292866443264 加上 1 得到 0.2178066707133556224  
00414278          D83D 34444100  fdivr dword ptr ds:\[414434\]       ; 用 DS:\[00414434\]的值 1 去除 ST(0)得到 4.5912276089837926400  
0041427E          DD9D 70FFFFFF  fstp qword ptr ss:\[ebp-90\]        ; 将 ST(0)的值 4.5912276089837926400 传给 SS:\[EBP-90\]  
00414284          68 00804C40    push 404C8000  
00414289          6A 00          push 0  
0041428B          E8 7C2F0800    call Recite.0049720C  
  
然后我们再进入 0049720C 处看看此处的代码，如下所示：  
0049720C          55             push ebp  
0049720D          8BEC           mov ebp,esp  
0049720F          8D65 F8        lea esp,dword ptr ss:\[ebp-8\]  
00497212          53             push ebx  
00497213          DD45 08        fld qword ptr ss:\[ebp+8\]          ; 将 EBP+8 值 57.0000000000000 传给 ST(0)  
00497216          66:8B45 0E     mov ax,word ptr ss:\[ebp+E\]        ; 将 EBP+E 值 404C 传给 AL  
0049721A          66:D1E0        shl ax,1                          ; AX 左移一位  
0049721D          74 12          je short Recite.00497231          ; 没跳  
0049721F          72 31          jb short Recite.00497252          ; 没跳  
00497221          66:3D E0FF     cmp ax,0FFE0                      ; 将上面左移的 AX 的值勤 8098 与 0FFE0 进行比较  
00497225          74 1F          je short Recite.00497246          ; 没跳  
00497227          E8 18FCFFFF    call Recite.00496E44  
  
再进入 00496E44 处，如下所示：  
00496E4A          D9C9           fxch st(1)                        ; 将 ST(0)与 ST(1)的值 57 进行交换  
00496E4C          83EC 0C        sub esp,0C  
00496E4F          D9E5           fxam                              ; 浮点检查  
00496E51          9B             wait  
00496E52          DD7C24 0A      fstsw word ptr ss:\[esp+A\]         ; 保存状态字  
00496E56          9B             wait  
00496E57          8A6424 0B      mov ah,byte ptr ss:\[esp+B\]        ; 将 ESP+B 中的 4 传入到 AH  
00496E5B          9E             sahf                              ; 恢复标志低八位 AH=34(S0,Z0,A1,P1,C0),FL=16  
00496E5C          72 0B          jb short Recite.00496E69          ; 没跳  
00496E5E          74 05          je short Recite.00496E65          ; 没跳  
00496E60          F6C4 02        test ah,2                         ; 检查 AH 是否为 2  
00496E63          74 1D          je short Recite.00496E82          ; 跳转实现  
...................................................  
00496E82          D9F1           fyl2x                             ; 求 st(0) \* log2( st(1) )的值,ST(0)值 57.000000000000000000 与 ST(1)0.3010299956639812096  
00496E84          83C4 0C        add esp,0C  
00496E87          C3             retn                              ; 得到 ST(0)的值为 1.7558748556724915200  
  
返回到 00497227C 处  
0049722C          5B             pop ebx                           ; 恢复堆栈，返回  
0049722D          8BE5           mov esp,ebp  
0049722F          5D             pop ebp  
00497230          C3             retn  
  
又一次返回到 00414290 处，代码如下：  
00414293          DCBD 70FFFFFF  fdivr qword ptr ss:\[ebp-90\]     ; 前面得到的\[EBP-90\]的值 4.5912276089837926400 除以 ST(0)的值得到 ST(0)的值为  
  
1.7558748556724915200  
00414299          D95D B0        fstp dword ptr ss:\[ebp-50\]      ; 将上面得到的 ST(0)的值 2.6147806571473310720 传给\[EBP-50\]  
0041429C          66:C745 D8 080>mov word ptr ss:\[ebp-28\],8      ; 赋\[EBP-28\]的值为 8  
004142A2          D945 B0        fld dword ptr ss:\[ebp-50\]       ; 将\[EBP-50\]的值--->ST(0)  
004142A5          E8 062D0800    call Recite.00496FB0  
  
这里又有一个 CALL，再次进入 00496FB0，代码所下：  
00496FB0          55             push ebp  
00496FB1          8BEC           mov ebp,esp  
00496FB3          8D65 F4        lea esp,dword ptr ss:\[ebp-C\]  
00496FB6          9B             wait  
00496FB7          D97D FC        fstcw word ptr ss:\[ebp-4\]       ; 浮点检查保存控制器  
00496FBA          9B             wait  
00496FBB          8A45 FD        mov al,byte ptr ss:\[ebp-3\]      ; 将 EBP-3 的值 13 传给 AL  
00496FBE          804D FD 0C     or byte ptr ss:\[ebp-3\],0C       ; \[EBP-3\]的值 13 与 0C 或操作  
00496FC2          D96D FC        fldcw word ptr ss:\[ebp-4\]       ; 浮点加载控制器  
00496FC5          DF7D F4        fistp qword ptr ss:\[ebp-C\]      ; ST(0)的值 2.6147806571473310720 传给\[EBP-C\]  
00496FC8          8845 FD        mov byte ptr ss:\[ebp-3\],al      ; 将 AL=13 传给\[EBP-3\]  
00496FCB          D96D FC        fldcw word ptr ss:\[ebp-4\]       ; 从\[EBP-4\]值装入 FCW  
00496FCE          8B45 F4        mov eax,dword ptr ss:\[ebp-C\]    ; 将\[EBP-C\]的值赋值给 EAX  
00496FD1          8B55 F8        mov edx,dword ptr ss:\[ebp-8\]    ; 将\[EBP-8\]的值零赋值给 EDX  
00496FD4          8BE5           mov esp,ebp  
00496FD6          5D             pop ebp  
00496FD7          C3             retn                            ; 返回  
  
返回到 004142AA 处,代码如下：  
004142AA          8985 74FFFFFF  mov dword ptr ss:\[ebp-8C\],eax  
004142B0          DB85 74FFFFFF  fild dword ptr ss:\[ebp-8C\]      ; 将\[EBP-8C\]的值 2 传给 ST(0)  
004142B6          D86D B0        fsubr dword ptr ss:\[ebp-50\]     ; 用前面得到的\[EBP-50\]的值 2.6147806571473310720 减去 ST(0)的值 2  
004142B9          DC0D 38444100  fmul qword ptr ds:\[414438\]      ; 将得到的 ST(0)的值 0.6147806644439697408 乘以\[414438\]处的值 1000000000000.000  
004142BF          E8 EC2C0800    call Recite.00496FB0            ;利用这个 CALL 将 6.1478066444396974080e+11 转化为注册码，中间会有误差，但返回值是我们所关心的  
  
我们在进入 00496FB0 处，代码如下：  
其实这段代码我们以前跟踪过只是数值有上些改变吧了！  
00496FB0          55             push ebp  
00496FB1          8BEC           mov ebp,esp  
00496FB3          8D65 F4        lea esp,dword ptr ss:\[ebp-C\]  
00496FB6          9B             wait  
00496FB7          D97D FC        fstcw word ptr ss:\[ebp-4\]       ; 浮点检查保存控制器  
00496FBA          9B             wait  
00496FBB          8A45 FD        mov al,byte ptr ss:\[ebp-3\]      ; 将 EBP-3 的值 13 传给 AL  
00496FBE          804D FD 0C     or byte ptr ss:\[ebp-3\],0C       ; \[EBP-3\]的值 13 与 0C 或操作  
00496FC2          D96D FC        fldcw word ptr ss:\[ebp-4\]       ; 浮点加载控制器  
00496FC5          DF7D F4        fistp qword ptr ss:\[ebp-C\]      ; ST(0)的值 6.1478066444396974080e+11 传给\[EBP-C\]  
00496FC8          8845 FD        mov byte ptr ss:\[ebp-3\],al      ; 将 AL=13 传给\[EBP-3\]  
00496FCB          D96D FC        fldcw word ptr ss:\[ebp-4\]       ; 从\[EBP-4\]值装入 FCW  
00496FCE          8B45 F4        mov eax,dword ptr ss:\[ebp-C\]    ; 将\[EBP-C\]的值 eax=23C87A7B 赋值给 EAX  
00496FD1          8B55 F8        mov edx,dword ptr ss:\[ebp-8\]    ; 将\[EBP-8\]的值零赋值给 EDX  
00496FD4          8BE5           mov esp,ebp  
00496FD6          5D             pop ebp  
00496FD7          C3             retn                            ; 返回  
  
返回到 004142C4 处，代码如下：  
004142C4          8985 70FFFFFF  mov dword ptr ss:\[ebp-90\],eax   ; 将 eax=23C87A7B 传给 ebp-90  
004142CA          DB85 70FFFFFF  fild dword ptr ss:\[ebp-90\]      ; 将十六进制 23C87A7B 传给 ST(0)  
004142D0          83C4 F8        add esp,-8                      ; 堆栈平衡  
004142D3          DD1C24         fstp qword ptr ss:\[esp\]         ; ST0 的值（即为注册码）传给 ESP 中  
004142D6          E8 912C0800    call Recite.00496F6C  
  
后面还有一些 CALL，没有作实际的处理，基本上就是浮点数转换为整数的操作，这时太不分析了，这样我们的注册码就得到了 600341115  
  
算法总结如下：这个算法其实很简单，中间只有两个很简单的函数，我这里就不用这两个函数，而是去慢慢分析这两个函数是如何来计算的，主要是为了加强分析能力！  
  
607.00000000000000000 与 0.7853981633974482944 取余数得到  
0.6726178571699049472  
  
然后对上面得到的值求正切函数  
得到 ST0 的值为 1.0000000000000000000  
ST1 的值为 0.7965240390979069952  
交换上面的两个值  
ST0 值变为 0.7965240390979069952  
ST1 值变为 1.0000000000000000000  
  
又将 ST1--->ST0，然后 ST0\*ST0 得到 ST0 值还是 1  
然后再交换 ST0 与 ST1 的值  
ST0 值还是 0.7965240390979069952  
ST1 值还是 1.0000000000000000000  
然后 ST0\*ST0 得到 0.6344505448608441344，再将 ST0 的值与 ST1 的值相加，得到  
1.6344505448608440320，在求平方根，得到 ST0 的值为 1.2784563132390735360  
  
将 ST0 的值求负得-1.2784563132390735360  
用 ST1 的值 1.0000000000000000000 除以 ST0 的值-1.2784563132390735360 得到-0.7821933292866443264  
  
将 ST0 的值-0.7821933292866443264 加上 1 得到 0.2178066707133556224  
再用存储在数据段中常数 1 去除 ST0 的值 0.2178066707133556224 得到 4.5912276089837926400  
  
再将浮点数 57.00000000000000 传给 ST0，然后计算 log10(2)的值 0.3010299956639812096，并将值也压入 ST0 中，此时  
ST0 的值为 0.3010299956639812096,ST1 的值 57.00000000000000，交换得  
ST0 的值为 57.00000000000000,ST1 的值为 0.3010299956639812096  
然后再求 ST0\*Log2(ST1)的值得到 1.7558748556724915200--->ST0  
  
然后在将前面得到的 ST0 的值 4.5912276089837926400 除以 1.7558748556724915200 得到 2.6147806571473310720  
然后用 2.6147806571473310720-2 得到 0.6147806571473310720 的值然后乘以 1000000000000.000 得到  
  
6.147806571473310720e+11  
到这里我们的注册码就出来了！  
  
总体来说软件逆向=精力+经验+热爱，逆向一个软件是很累的活，但我相对于写软件的人来说还是比较年轻的，呵呵，希望大家想学逆向工程式的朋友们都有能有这三种东西，我想你将来会在这个领域小有成就，一切只是时间的问题！  
  
这最后说明一点，浮点数到最后转换成注册码的时候，会有一点误差，这是不可避免的，大家只要单步跟踪到注册码就行了，至于中间的结果，在浮点运算转换成其它数值的时候会有误差，也不用去管，这是不可避免的！![](img/c1376e2bd59b6be48d07a70822c7e47d.jpg "伤心")  
  
好了，文章就写到这了，说点题外话，最近比较忙![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")，QQ 基本不上线了，看雪上可能有时会看到我写的一些文章，最近在潜心去修练逆向工程与反汇编知识，争取能更上一层楼，我相信这一切只是时间的问题，只要自己肯努力去做!因为不想被太多人打扰 QQ 就不上了，请各位看雪上的朋友们原谅，不过只要熊猫正正有时间，一定会尽量能大家带来一些好文章！（只能尽力啦，因为毕竟熊猫正正现在还只是菜鸟一只，呵呵，还需要努力才行），我一直很崇拜几位科学家，钱学森，陈景润，王选等（其实还有好多无私奉献的科学家，我就不一一列举了），他们的技术那不用说了，中国没人能比啦，但我想说主要是他们的精神，真的值得我们去好好学习一下！只为自己喜欢的事业而努力奋斗，不在乎得到了多少荣誉，在他们喜欢的领域，他们总是默默无闻，不为名利，只为自己所热爱的事业！！（注：五笔打的比较快，难免会有一些笔误！请原谅！）![](img/cda69129882b40377a87144c782680e1.jpg "微笑")