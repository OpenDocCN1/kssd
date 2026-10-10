

# Sparc 平台下的 FlexLM 4.1 明文比对破解

> 原文：[`www.pediy.com/kssd/pediy10/61983.html`](https://www.pediy.com/kssd/pediy10/61983.html)

Sparc 平台下的 FlexLM 4.1 明文比对破解  
Passion  
www.cnpack.org  
2008-03-26  
  
网上关于破解 FlexLM 的文章大多是 Windows 平台的，而且版本很高。但近日俺碰到的一款 SunOS Sparc 平台上的软件也是 FlexLM 保护，这可是个希罕物件，又加上整个公司只买了 2 个 License，一块用的时候俺基本轮不上，于是就带着尝试的想法拿它开刀。IDA 反了反这个 ELF 格式的主程序文件，字符串中看见是 FlexLM 4.1 版的，而且程序的字符串块没被 strip 掉，FlexLM 的相关函数名居然都在，省掉了找 Unix 下的 FlexLM SDK 的 Signature 的麻烦。而且，这个 4.1 的版本由于相当低，其密文字符串的对比居然是明码标价，实在不符合其商业产品的特性（不过这也是事后才知道的）。  
  
工具：IDA，DDD/GDB；  
方法：静态阅读＋动态跟踪。  
  
网上找不到 FlexLM 4.0 的 SDK，但能找到的比较全的 FlexLM 4.0 的开发帮助：  
  
http://wwweic.eri.u-tokyo.ac.jp/computer/manual/lx/SGI\_Developer/books/FLEXlm\_PG/sgi\_html/index.html  
  
这个对破解很有作用。  
  
先恶补一下基础知识：  
  
**一、Sparc 平台的寄存器架构与汇编规则**  
  
Sparc 的 CPU 有很多寄存器，但针对任一进程的任一时刻来说只开放 32 个，以寄存器窗口的方式切换。一般它的寄存器可命名为 r0 到 r31，但一般以其别名来命名，如下：  
  
1. 全局寄存器(8 个) －对所有程序可见。命名为 %g0 ～ %g7 ＝ %r0 ～ %r7  
2. 输出寄存器(8 个) －函数返回值，输出寄存器是下一个窗口的输入寄存器。命名为 %o0 ～ %o7，等同于 %r8 ～ %r15  
3. 局部寄存器(8 个) －仅本函数可见。命名为 %l0 ～ %l7，等同于 %r16 ～ %r23  
4. 输入寄存器(8 个) －本函数的输入参数，来自于上一窗口的输出寄存器。命名为 %i0 ～ %i7，等同于 %r24 ～ %r31  
  
通过 call 指令进行函数调用时，函数开始可调用 save 指令分配相应的栈空间并进行窗口切换，这个窗口切换会将切换前的 o 系列寄存器映射为切换后的 i 系列寄存器，函数返回时通过 restore 指令切换回来。这条规则要记住，否则进入函数前后参数和返回值都不知道去哪儿找。  
  
一般进入函数之前给 o0 到 o5 寄存器赋值作为传入参数（不够则用堆栈），进入函数之后访问对应的 i0 到 i5 便可访问对应的传入参数，函数内部给 o0 赋值可作为返回值使用。  
  
此外 Sparc 还有一些其他特点：访存用 ld/st 指令，和寄存器之间操作数据不同；跳转指令是 b(ranch)，后缀表示各种条件，和 80x86 平台下的 j 系列一样；call 与 b 跳转指令后都有一个 nop，估计是给 Sparc 的指令预取机制或者跳转预判断机制准备的。  
  
**二、GDB/DDD 常用命令：**  
  
Data Display Debugger（DDD）是 Unix 平台下基于 GDB 的一款图形化调试工具，虽然用起来没 OllyDbg 等方便，但毕竟比 gdb 的纯命令行好得多，至少不需要频繁敲 disas 命令来查看反汇编代码，而且还可方便设置多个 display 来查看寄存器值，也不需要频繁敲 info reg o0 之类的命令。因此凑合着用它了。  
  
对调试汇编代码比较有用的 gdb 命令有以下：  
  
b main  
b \*0x27aaa  
在某命名函数处或在某地址处设置断点。  
  
bt  
查看此刻的调用堆栈。  
  
info reg o0  
查看某寄存器值，如果只敲 info reg，则打印所有寄存器值。  
  
stepi/nexti  
单步执行机器代码。注意不是 step/next，后者是单步执行源码。  
  
disable 1  
禁用第一个断点。如只敲 disable 则禁用所有断点。  
  
enable 1  
使能第一个断点。如只敲 enable 则使能所有断点。  
  
x /8xb 0x45678  
从地址 0x45678 处开始查看 8 个 byte，以 HEx 的方式显示。  
  
x /s $o1  
查看 o1 寄存器内容所指内存处的内容，以字符串方式显示。  
  
set $o1=0  
将寄存器 o1 值设置为 0。  
  
r、c、k  
这三个命令的功能分别为：运行被调试程序、中断后继续运行、中止被调试程序。  
  
**三、FlexLM 解密过程**  
  
用 IDA 反汇编看，程序一开始经过必要的初始化后，便会调用 l\_init 开始 FlexLM 的检查，此间暴露了 Vendor Name、Feature 名，版本号等（见下）。然后程序逐步会进入核心的 lc\_checkout 函数。如需要爆破，则修改 lc\_checkout 的返回值即可，但俺的目的是想获得真正能用的 License，因此继续。  
  
.text:00017194 loc\_17194: ! <suspicious>               ! CODE XREF: main+5DCj  
.text:00017194                 sethi   %hi(0x50800), %i0 ! <suspicious>  
.text:00017198                 sethi   %hi(0x50800), %i1 ! <suspicious>  
.text:0001719C                 st      %g2, \[prog\]  
.text:000171A0                 mov     1, %g2  
.text:000171A4                 sethi   %hi(0x4C000), %o0 ! <suspicious>  
.text:000171A8                 st      %g2, \[num\_lic\]  
.text:000171AC                 set     lm\_job, %l5  
.text:000171B0                 set     s\_Lic\_sunw, %o1 ! "lic.SUNW"  
.text:000171B8                 add     %l5, 4, %o2     ! int  
.text:000171BC                 ld      \[lm\_job\], %o0   ! int  
.text:000171C0                 call    lc\_init  
  
lc\_init 函数调用前需要传入 Vendor Name，此处得知是“lic.SUNW”。  
  
.text:0001732C loc\_1732C:                              ! CODE XREF: main+714j  
.text:0001732C                                         ! main:loc\_172D0j  
.text:0001732C                 st      %g2, \[%sp+0xB10+var\_AB4\]  
.text:00017330                 set     s\_Stack\_1, %o1  ! "STACK"  
.text:00017338                 sethi   %hi(sccsid\_4), %g2  
.text:0001733C                 ld      \[%l5\], %o0  
.text:00017340                 set     s\_8\_0, %o2      ! "8.0"  
.text:00017344                 sethi   %hi(0x4C000), %g2 ! <suspicious>  
.text:00017348                 ld      \[%i1+0x13C\], %o3  
.text:0001734C                 set     code, %o5  
.text:00017350                 call    lc\_checkout  
  
Feature 名是“STACK”，版本号（Version Number）是“8.0”。  
然后可根据看到的 Feature 名、Vendor Name 和 Version Number 构造了一个就一行的 license.dat 文件：  
  
FEATURE STACK lic.SUNW 8.0 1-jan-0000 uncounted 1234567890ABCDEF HOSTID=ANY  
  
构造此 license 文件的目的是构造针对此 Feature 的无时间限制、无数量限制、无运行主机限制的完全 license。当然，其中签名部分是瞎凑的，目的是为了跟踪到和正确签名的比对的部分。  
构造完文件后，将环境变量 LM\_LICENSE\_FILE 指向此文件 /tmp/license.dat  
  
export LM\_LICENSE\_FILE=/tmp/license.dat  
  
根据网上翻译的 FlexLM 9.2 的解密文章，lc\_checkout 内部最终会调用 l\_string\_key 函数来进行比较，但资料里说 9.2 版的 FlexLM SDK 已经在此函数上做了部分手脚，会用伪造的签名来蒙混跟踪者，只有无签名返回时才是正确的场合。根据此点，我在 l\_string\_key 函数中还真绕了不少弯路（弯路部分略）。后来跟踪的事实证明，FlexLM 4.1 版本中还没有此狡猾的机制，而是非常直白的调用 l\_string\_key，并在 l\_crypt\_private 返回时便能生成正确的明文签名。获得 l\_crypt\_private 的结果后，上级的 good\_lic\_key 再调用 strncmp 进行比较，从而得到比对的结果，这就是 FlexLM 4.1 的核心对比。  
FlexLM 9.2 或者其他版本估计是察觉到了此问题，便将比对隐藏起来了，但仍然保留着 strncmp 来混淆视听。  
  
中断在 l\_string\_key 中时，用 bt 查看调用堆栈得到以下信息：  
  
#0  0x000277d8 in l\_string\_key ()  
#1  0x00027770 in l\_crypt\_private ()  
#2  0x0001ce98 in good\_lic\_key ()  
#3  0x0001c4a4 in lm\_start\_real ()  
#4  0x0001b61c in lc\_checkout ()  
#5  0x00017358 in main ()  
  
在此，无需刻舟求剑地在 l\_string\_key 处直接下断以企图获取正确签名值，而应该跳至 l\_crypt\_private 被调用的地方：  
  
.text:0001CE80                 mov     %o0, %o2  
.text:0001CE84                 add     %fp, var\_20, %o3  
.text:0001CE88                 ld      \[%fp+arg\_48\], %o1  
.text:0001CE8C                 ld      \[%fp+arg\_44\], %o0  
.text:0001CE90                 call    l\_crypt\_private  
.text:0001CE94                 nop  
.text:0001CE94  
.text:0001CE98                 st      %o0, \[%fp+s1\]  
.text:0001CE9C                 ld      \[%fp+arg\_48\], %o1  
.text:0001CEA0                 inc     0x48, %o1       ! s2  
.text:0001CEA4                 ld      \[%fp+s1\], %o0   ! s1  
.text:0001CEA8                 mov     0x14, %o2       ! n  
.text:0001CEAC                 call    \_strncmp  
.text:0001CEB0                 nop  
.text:0001CEB0  
.text:0001CEB4                 tst     %o0  
.text:0001CEB8                 bne     loc\_1CEC8  
.text:0001CEBC                 nop  
  
这里，strncpm 的两个参数 s1 和 s2，一个是我们输入的错误 license，一个是它计算出来的正确 license。在 0001CEAC 处中断时用 GDB 的 x /s $o0 和 x /s $o1 命令可看到正确的签名值 8503E11B1950D9B5AA12。  
  
最后，无限制的 License 文件生成如下：  
  
FEATURE STACK lic.SUNW 8.0 1-jan-0000 uncounted 8503E11B1950D9B5AA12 HOSTID=ANY  
  
保存此文件后退出 DDD，再次运行程序，一切 OK。  
  
**四、总结**  
  
FlexLM 4.1 的保护方式很薄弱，估计其低版本如 2.6 等也一样。如果 Unix 平台下编译的程序符号表还在的话，简直就是中门大开引狼入室。破解者无需 FlexLM 4.1 的 SDK，也不用寻找什么 seed 什么 key，找到 Feature、版本号和 Vendor Name 后就能在程序中直接跟踪到明文的签名，所以本文写到后来也就变成了初级水平。  
关于 FlexLM 的基础知识，可参考《加密与解密》书刊中的相关部分。  
  
附：很久没写破解方面的文章了，希望不至于太差。![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")