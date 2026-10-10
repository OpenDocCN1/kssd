

# 第一章:1.5、switch-case 识别技巧初探

> 原文：[`www.pediy.com/kssd/pediy11/114479.html`](https://www.pediy.com/kssd/pediy11/114479.html)

要问接触 C++逆向之前最有趣的知识点是什么？笔者认为非 switch-case 结构莫属了……  
    但是不得不说的是，有关于 switch-case 逆向真的可谓博大精深了（相对于基础薄弱者），因此笔者在提笔之前做了很多假设，并且自己也推翻了几种方案。最后得出的结论是此知识点一定要深入讲解，特别是到后面的东西更是要涉及到一些数据结构相关的知识，笔者会尽力用最简洁、最简单的语言为大家阐述清楚，我想既然我都有信心使各位读者能看懂本小节，那么各位读者就更因该努力了。  
    下面我们进入主题，我们都知道 switch-case 的诞生其实就是为了避免出现大量的、高重复 if-else 语句，换句话说，switch-case 语句其实就是 if-else 语句的另一种体现形式，但是事实真的是这样吗？让我们一起为这个观点求证……  
  
**1.5.1、简单 switch-case 分支识别技巧**  
  
    我们先看一段代码：  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{  
    int nNum = 2;  
    switch (nNum)  
    {  
    case 0:  
        printf("nNum=0");  
        break;  
    case 1:  
        printf("nNum=1");  
        break;  
    case 2:  
        printf("nNum=2");  
        break;  
    default:  
        printf("nNum=%d,error!",nNum);  
    }  
    return 0;  
}  
  
    看完这段代码，在看完本小节的标题，有些读者可能会产生一些疑问，例如 switch-case 的不可达分支会被剪掉吗、switch-case 分支以常量为判断条件的优化效果与 if-else 有多大区别、以变量为判断条件的 switch-case 分支优化效果与 if-else 分支有多大区别等等问题。  
    现在就让我们带着这些问题继续，让我们一一为其找到答案。  
    先看 Debug 版的反汇编代码：  
  
004133AE  MOV DWORD PTR SS:\[EBP-8\], 2              ; 给局部变量赋值  
004133B5  MOV EAX, DWORD PTR SS:\[EBP-8\]  
004133B8  MOV DWORD PTR SS:\[EBP-D0\], EAX  
004133BE  CMP DWORD PTR SS:\[EBP-D0\], 0             ; 比较是否等于 0  
004133C5  JE SHORT Test\_0.004133DB                 ; 如果等于 0 则跳到相应分支，否则继续  
004133C7  CMP DWORD PTR SS:\[EBP-D0\], 1  
004133CE  JE SHORT Test\_0.004133F4                 ; 同上  
004133D0  CMP DWORD PTR SS:\[EBP-D0\], 2  
004133D7  JE SHORT Test\_0.0041340D                 ; 同上  
004133D9  JMP SHORT Test\_0.00413426                ; 都不符合则直接跳转到最后一个分支处  
004133DD  PUSH Test\_0.00415808                     ; /format = "nNum=0"  
004133E2  CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]   ; \\printf  
004133E8  ADD ESP, 4  
004133F2  JMP SHORT Test\_0.00413441  
004133F6  PUSH Test\_0.004157B0                     ; /format = "nNum=1"  
004133FB  CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]   ; \\printf  
00413401  ADD ESP, 4  
0041340B  JMP SHORT Test\_0.00413441  
0041340F  PUSH Test\_0.00415C18                     ; /format = "nNum=2"  
00413414  CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]   ; \\printf  
0041341A  ADD ESP, 4  
00413424  JMP SHORT Test\_0.00413441  
00413428  MOV EAX, DWORD PTR SS:\[EBP-8\]  
0041342B  PUSH EAX                                 ; /<%d>  
0041342C  PUSH Test\_0.004157A0                     ; |format = "nNum=%d,error!"  
00413431  CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]   ; \\printf  
00413437  ADD ESP, 8  
  
    通过以上反汇编代码我们可以总结出以下特点：  
  
cmp XXX,XXX  
jXX CASE1\_TAG  
cmp XXX,XXX  
jXX CASE2\_TAG  
cmp XXX,XXX  
jXX CASE3\_TAG  
......  
CMP XXX,XXX  
JXX CASEN\_TAG  
......  
JMP DEFAULT  
CASE1\_TAG:  
......  
CASE2\_TAG:  
......  
CASE3\_TAG:  
......  
......  
CASEN\_TAG:  
......  
......  
DEFAULT:  
......  
SWITCH\_END\_TAG:  
  
    我们可以看到 Debug 版的反汇编指令与我们的源代码的相似度还是非常高的，都是通过开始的一连串判断，然后确定接下来走哪个 Case 分支。下面我们再看看 Release 版的反汇编代码：  
  
00401000  PUSH Test\_0.004020F4                     ; /format = "nNum=2"  
00401005  CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
0040100B  ADD ESP, 4  
0040100E  XOR EAX, EAX  
00401010  RETN  
  
    瞧瞧，简单至极呀！由此可见 switch-case 语句与我们之前接触的 if-else 语句一样，不可达分支都会被编译器优化掉，那么如果如果其部分分支相同，是否仍会像 if-else 分之一样呢？先看源码：  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{  
    switch (argc)  
    {  
    case 0:  
        printf("argc=0",argc);  
        break;  
    case 1:  
        printf("argc=%d",argc);  
        break;  
    case 2:  
        printf("argc=%d",argc);  
        break;  
    default:  
        printf("argc=%d,error!",argc);  
    }  
    return 0;  
}  
  
    按照 if-esle 的优化逻辑，case 1 与 csae 2 会指向同一处，真的是这样吗？我们直接看 Release 版反汇编代码：  
  
00401000  /$>MOV ECX, DWORD PTR SS:\[ESP+4\]  
00401004  |.>MOV EAX, ECX  
00401006  |.>SUB EAX, 0                               ;  Switch (cases 0..2)   
00401009  |.>JE SHORT Test\_0.0040104D  
0040100B  |.>SUB EAX, 1                               ; 注意这里用的是减法  
0040100E  |.>JE SHORT Test\_0.0040103A  
00401010  |.>SUB EAX, 1  
00401013  |.>JE SHORT Test\_0.00401027  
00401015  |.>PUSH ECX                                 ; /<%d>; Default case of switch 00401006  
00401016  |.>PUSH Test\_0.00402104                     ; |format = "argc=%d,error!"  
0040101B  |.>CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
00401021  |.>ADD ESP, 8  
00401024  |.>XOR EAX, EAX  
00401026  |.>RETN                                     ; 执行完某一个分支后会直接返回  
00401027  |>>PUSH 2                                   ; /<%d> = 2; Case 2 of switch 00401006  
00401029  |.>PUSH Test\_0.004020FC                     ; |format = "argc=%d"  
0040102E  |.>CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
00401034  |.>ADD ESP, 8  
00401037  |.>XOR EAX, EAX  
00401039  |.>RETN  
0040103A  |>>PUSH 1                                   ; /<%d> = 1; Case 1 of switch 00401006  
0040103C  |.>PUSH Test\_0.004020FC                     ; |format = "argc=%d"  
00401041  |.>CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
00401047  |.>ADD ESP, 8  
0040104A  |.>XOR EAX, EAX  
0040104C  |.>RETN  
0040104D  |>>PUSH 0                                   ;  Case 0 of switch 00401006  
0040104F  |.>PUSH Test\_0.004020F4                     ; /format = "argc=0"  
00401054  |.>CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
0040105A  |.>ADD ESP, 8  
0040105D  |.>XOR EAX, EAX  
0040105F  \\.>RETN  
  
    看来 switch-case 并没有将相同的分支合并，我们可以很清楚的看到它的 4 个分支仍都存在。  
    既然它的分支并没有合并，那么我们就讨论点其他的，请各位读者回头仔细观察反汇编代码的第 1-9 行，我们可以发现 Release 在条件跳转前用的不再是 cmp，而是 sub，很显然编译器这样优化是有其理由的，但是这个理由究竟是什么？  
    我们通过阅读这块代码可知程序先将 main 函数的参数 1 传递给 EAX，然后减 0，这有点让人迷糊，我们接着看下面的那个跳转：  
  
00401009  |.>JE SHORT Test\_0.0040104D  
   
    让我们回顾一下汇编语言，我们应该都记得 JE 的跳转条件是 ZF=1，因此当我们的 EAX 为 0 时，那么将其减 0 肯定会使 ZF 位置 1，因此其实这就是一个变形的 CMP 指令，只不过这么做程程的代码体积更小、效率更高。  
    知道这些后，后面的优化自然就肯好理解了，现在假设我们的 EAX 等于 2，因此按照上面代码的流程走会先将其减 0，此时 ZF 位不变，接着下面又对其减 1，此时 ZF 位仍然没变化，而当走到第三步时，此时 EAX 的值为 1，又将其减 1 后肯定就等于 0 了，ZF 位置为 1，后面的 JZ 跳转生效……  
    我们可以看到其实就是做了一连串的减法，到哪等于 0 后，就证明这个值原先为多少，由此可见微软的编译器还是很聪明的。不过这些代码从本质上来说还是属于 if-esle 范畴内的。  
  
      
**1.5.2、多分支的 switch-case 识别**  
  
    我们平时在写程序时都会遇到一些问题，而这些问题肯定必须要用多分支的 switch-case 才能解决，但是你知道这种情况在反汇编状态下应该怎么去识别吗？下面我们就一起看看 switch-case 的另外一种体现方式，我们先看代码：  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{  
    switch (argc)  
    {  
    case 0:  
        printf("argc=0",argc);  
        break;  
    case 1:  
        printf("argc=%d",argc);  
        break;  
    case 6:  
        printf("argc=%d",argc);  
        break;  
    case 7:  
        printf("argc=%d",argc);  
        break;  
    case 9:  
        printf("argc=%d",argc);  
        break;  
    default:  
        printf("argc=%d,error!",argc);  
    }  
    return 0;  
}  
  
    注意上面的 case 条件并不是有规律的，我们直接看它的 Release 版反汇编代码：  
  
00401000  MOV EAX, DWORD PTR SS:\[ESP+4\]  
00401004  CMP EAX, 9                               ;  Switch (cases 0..9)  
00401007  JA SHORT Test\_0.0040106F                 ;  如果大于最大值 9 则直接跳到 Default 处  
00401009  JMP DWORD PTR DS:\[EAX\*4+401084\]          ;  注意这里！！  
00401010  PUSH 0                                   ;  Case 0 of switch 00401004  
00401012  PUSH Test\_0.004020F4                     ; /format = "argc=0"  
00401017  CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
0040101D  ADD ESP, 8  
00401020  XOR EAX, EAX  
00401022  RETN  
00401023  PUSH 1                                   ; /<%d> = 1; Case 1 of switch 00401004  
00401025  PUSH Test\_0.004020FC                     ; |format = "argc=%d"  
0040102A  CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
00401030  ADD ESP, 8  
00401033  XOR EAX, EAX  
00401035  RETN  
00401036  PUSH 6                                   ; /<%d> = 6; Case 6 of switch 00401004  
00401038  PUSH Test\_0.004020FC                     ; |format = "argc=%d"  
0040103D  CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
00401043  ADD ESP, 8  
00401046  XOR EAX, EAX  
00401048  RETN  
00401049  PUSH 7                                   ; /<%d> = 7; Case 7 of switch 00401004  
0040104B  PUSH Test\_0.004020FC                     ; |format = "argc=%d"  
00401050  CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
00401056  ADD ESP, 8  
00401059  XOR EAX, EAX  
0040105B  RETN  
0040105C  PUSH 9                                   ; /<%d> = 9; Case 9 of switch 00401004  
0040105E  PUSH Test\_0.004020FC                     ; |format = "argc=%d"  
00401063  CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
00401069  ADD ESP, 8  
0040106C  XOR EAX, EAX  
0040106E  RETN  
0040106F  PUSH EAX                                 ; /<%d>; Default case of switch 00401004  
00401070  PUSH Test\_0.00402104                     ; |format = "argc=%d,error!"  
00401075  CALL DWORD PTR DS:\[<&MSVCR90.printf>\]    ; \\printf  
0040107B  ADD ESP, 8  
0040107E  XOR EAX, EAX  
00401080  RETN  
  
    看完上段反汇编代码后有些读者可能感觉很奇怪，难道那个位于第 4 行的 JMP 指令就能解决这些问题吗？当然不会这么简单……  
    我们现在就仔细分析一下头几条反汇编指令：  
  
00401000  MOV EAX, DWORD PTR SS:\[ESP+4\]            ; 取得局部变量后传递给 EAX  
00401004  CMP EAX, 9                               ; 与 9 作比较，我们通过源代码可知这个 switch-case 分支的最大值“case 9”，因  
                                                   ; 此如果传入的值大于 9 就肯定会执行 Default 处代码了。  
00401007  JA SHORT Test\_0.0040106F  
00401009  JMP DWORD PTR DS:\[EAX\*4+401084\]          ; EAX 此时保存的是输入的值，将其乘以 4 后再加上一个地址，这其实就是一个典型  
                                                   ; 的数组寻址，由于我们还没学数组寻址，所以这块先放一放。我们现在只需要要  
                                                   ; 知道这是一个读取 int 型的数组，里面保存的是地址指针。  
  
    那么目标地址里究竟保存了什么数据？这个机制的原理又是怎么回事呢？我们看看如下内容便可以猜出一二了……  
  
Address   Data      Tag  
00401084  00401010  Test\_0.00401010   ; Case0  
00401088  00401023  Test\_0.00401023   ; Case1  
0040108C  0040106F  Test\_0.0040106F   ; Case2  
00401090  0040106F  Test\_0.0040106F   ; Case3  
00401094  0040106F  Test\_0.0040106F   ; Case4  
00401098  0040106F  Test\_0.0040106F   ; Case5  
0040109C  00401036  Test\_0.00401036   ; Case6  
004010A0  00401049  Test\_0.00401049   ; Case7  
004010A4  0040106F  Test\_0.0040106F   ; Case8  
004010A8  0040105C  Test\_0.0040105C   ; Case9  
  
    通过上面表格的地址可以知道这就是我们上面提到的“数组指针”了，里面保存的内容是各个 Case 块的地址，我们将之称为“跳转表”。跳转表的作用就是通过数组的寻址运算代替复杂的 if-else 分支，这样可以大大提供程序的执行效率。  
    它的基本原理是建立一张表格，里面保存着从 case1 到 caseN 的所有分支应该到达的地址。以上面程序的情况为例子，我们可以看出从 case2 至 case5 里保存的地址都是跳向 Default 分支的地址，这就证明这几个 case 在程序的源代码中是属于未处理（或称为非正常）的状态。  
    但是什么时候编译器才会决定用跳转表的方式来优化程序呢？这显然不是我等简单的用个什么公式就能计算出来的，这需要综合各种条件、因素来决定是否使用跳转表。在这里我可以给出一个反例，既如果我们的最大 case 块为 999999 的话，难道程序还会用这个方法解决吗？肯定不会！  
    下一节我将阐述编译器是怎样解决这类复杂的 Switch-case 组合的。  
  
**[【返回到目录】：http://bbs.pediy.com/showthread.php?t=113689](http://bbs.pediy.com/showthread.php?t=113689)**