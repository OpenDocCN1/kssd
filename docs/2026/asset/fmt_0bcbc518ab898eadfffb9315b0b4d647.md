
# [原创]第一章:1.5、switch-case识别技巧初探

要问接触C++逆向之前最有趣的知识点是什么？笔者认为非switch-case结构莫属了……

但是不得不说的是，有关于switch-case逆向真的可谓博大精深了（相对于基础薄弱者），因此笔者在提笔之前做了很多假设，并且自己也推翻了几种方案。最后得出的结论是此知识点一定要深入讲解，特别是到后面的东西更是要涉及到一些数据结构相关的知识，笔者会尽力用最简洁、最简单的语言为大家阐述清楚，我想既然我都有信心使各位读者能看懂本小节，那么各位读者就更因该努力了。

下面我们进入主题，我们都知道switch-case的诞生其实就是为了避免出现大量的、高重复if-else语句，换句话说，switch-case语句其实就是if-else语句的另一种体现形式，但是事实真的是这样吗？让我们一起为这个观点求证……

## 1.5.1、简单switch-case分支识别技巧

我们先看一段代码：

```
int _tmain(int argc, _TCHAR* argv[])
{
    int nNum = 2;
    switch (nNum)
    {
    case 0:
        printf("nNum=0");
        break;
    case 1:
        printf("nNum=1");
        break;
    case 2:
        printf("nNum=2");
        break;
    default:
        printf("nNum=%d,error!",nNum);
    }
    return 0;
}
```

看完这段代码，在看完本小节的标题，有些读者可能会产生一些疑问，例如switch-case的不可达分支会被剪掉吗、switch-case分支以常量为判断条件的优化效果与if-else有多大区别、以变量为判断条件的switch-case分支优化效果与if-else分支有多大区别等等问题。

现在就让我们带着这些问题继续，让我们一一为其找到答案。

先看Debug版的反汇编代码：

```
004133AE  MOV DWORD PTR SS:[EBP-8], 2              ; 给局部变量赋值
004133B5  MOV EAX, DWORD PTR SS:[EBP-8]
004133B8  MOV DWORD PTR SS:[EBP-D0], EAX
004133BE  CMP DWORD PTR SS:[EBP-D0], 0             ; 比较是否等于0
004133C5  JE SHORT Test_0.004133DB                 ; 如果等于0则跳到相应分支，否则继续
004133C7  CMP DWORD PTR SS:[EBP-D0], 1
004133CE  JE SHORT Test_0.004133F4                 ; 同上
004133D0  CMP DWORD PTR SS:[EBP-D0], 2
004133D7  JE SHORT Test_0.0041340D                 ; 同上
004133D9  JMP SHORT Test_0.00413426                ; 都不符合则直接跳转到最后一个分支处
004133DD  PUSH Test_0.00415808                     ; /format = "nNum=0"
004133E2  CALL DWORD PTR DS:[<&MSVCR90D.printf>]   ; \printf
004133E8  ADD ESP, 4
004133F2  JMP SHORT Test_0.00413441
004133F6  PUSH Test_0.004157B0                     ; /format = "nNum=1"
004133FB  CALL DWORD PTR DS:[<&MSVCR90D.printf>]   ; \printf
00413401  ADD ESP, 4
0041340B  JMP SHORT Test_0.00413441
0041340F  PUSH Test_0.00415C18                     ; /format = "nNum=2"
00413414  CALL DWORD PTR DS:[<&MSVCR90D.printf>]   ; \printf
0041341A  ADD ESP, 4
00413424  JMP SHORT Test_0.00413441
00413428  MOV EAX, DWORD PTR SS:[EBP-8]
0041342B  PUSH EAX                                 ; /<%d>
0041342C  PUSH Test_0.004157A0                     ; |format = "nNum=%d,error!"
00413431  CALL DWORD PTR DS:[<&MSVCR90D.printf>]   ; \printf
00413437  ADD ESP, 8
```

通过以上反汇编代码我们可以总结出以下特点：

```
cmp XXX,XXX
jXX CASE1_TAG
cmp XXX,XXX
jXX CASE2_TAG
cmp XXX,XXX
jXX CASE3_TAG
......
CMP XXX,XXX
JXX CASEN_TAG
......
JMP DEFAULT
CASE1_TAG:
......
CASE2_TAG:
......
CASE3_TAG:
......
......
CASEN_TAG:
......
......
DEFAULT:
......
SWITCH_END_TAG:
```

我们可以看到Debug版的反汇编指令与我们的源代码的相似度还是非常高的，都是通过开始的一连串判断，然后确定接下来走哪个Case分支。下面我们再看看Release版的反汇编代码：

```
00401000  PUSH Test_0.004020F4                     ; /format = "nNum=2"
00401005  CALL DWORD PTR DS:[<&MSVCR90.printf>]    ; \printf
0040100B  ADD ESP, 4
0040100E  XOR EAX, EAX
00401010  RETN
```

瞧瞧，简单至极呀！由此可见switch-case语句与我们之前接触的if-else语句一样，不可达分支都会被编译器优化掉，那么如果如果其部分分支相同，是否仍会像if-else分之一样呢？先看源码：

```
int _tmain(int argc, _TCHAR* argv[])
{
    switch (argc)
    {
    case 0:
        printf("argc=0",argc);
        break;
    case 1:
        printf("argc=%d",argc);
        break;
    case 2:
        printf("argc=%d",argc);
        break;
    default:
        printf("argc=%d,error!",argc);
    }
    return 0;
}
```

按照if-esle的优化逻辑，case 1 与 csae 2 会指向同一处，真的是这样吗？我们直接看Release版反汇编代码：

```
00401000  /$>MOV ECX, DWORD PTR SS:[ESP+4]
00401004  |.>MOV EAX, ECX
00401006  |.>SUB EAX, 0                               ;  Switch (cases 0..2)
00401009  |.>JE SHORT Test_0.0040104D
0040100B  |.>SUB EAX, 1                               ; 注意这里用的是减法
0040100E  |.>JE SHORT Test_0.0040103A
00401010  |.>SUB EAX, 1
00401013  |.>JE SHORT Test_0.00401027
00401015  |.>PUSH ECX                                 ; /<%d>; Default case of switch 00401006
00401016  |.>PUSH Test_0.00402104                     ; |format = "argc=%d,error!"
0040101B  |.>CALL DWORD PTR DS:[<&MSVCR90.printf>]    ; \printf
00401021  |.>ADD ESP, 8
00401024  |.>XOR EAX, EAX
00401026  |.>RETN                                     ; 执行完某一个分支后会直接返回
00401027  |>>PUSH 2                                   ; /<%d> = 2; Case 2 of switch 00401006
00401029  |.>PUSH Test_0.004020FC                     ; |format = "argc=%d"
0040102E  |.>CALL DWORD PTR DS:[<&MSVCR90.printf>]    ; \printf
00401034  |.>ADD ESP, 8
00401037  |.>XOR EAX, EAX
00401039  |.>RETN
0040103A  |>>PUSH 1                                   ; /<%d> = 1; Case 1 of switch 00401006
0040103C  |.>PUSH Test_0.004020FC                     ; |format = "argc=%d"
00401041  |.>CALL DWORD PTR DS:[<&MSVCR90.printf>]    ; \printf
00401047  |.>ADD ESP, 8
0040104A  |.>XOR EAX, EAX
0040104C  |.>RETN
0040104D  |>>PUSH 0                                   ;  Case 0 of switch 00401006
0040104F  |.>PUSH Test_0.004020F4                     ; /format = "argc=0"
00401054  |.>CALL DWORD PTR DS:[<&MSVCR90.printf>]    ; \printf
0040105A  |.>ADD ESP, 8
0040105D  |.>XOR EAX, EAX
0040105F  \.>RETN
```

看来switch-case并没有将相同的分支合并，我们可以很清楚的看到它的4个分支仍都存在。

既然它的分支并没有合并，那么我们就讨论点其他的，请各位读者回头仔细观察反汇编代码的第1-9行，我们可以发现Release在条件跳转前用的不再是`cmp`，而是`sub`，很显然编译器这样优化是有其理由的，但是这个理由究竟是什么？

我们通过阅读这块代码可知程序先将`main`函数的参数1传递给`EAX`，然后减0，这有点让人迷糊，我们接着看下面的那个跳转：

```
00401009  |.>JE SHORT Test_0.0040104D
```

让我们回顾一下汇编语言，我们应该都记得`JE`的跳转条件是`ZF=1`，因此当我们的`EAX`为0时，那么将其减0肯定会使`ZF`位置1，因此其实这就是一个变形的`CMP`指令，只不过这么做程程的代码体积更小、效率更高。

知道这些后，后面的优化自然就肯好理解了，现在假设我们的`EAX`等于2，因此按照上面代码的流程走会先将其减0，此时`ZF`位不变，接着下面又对其减1，此时`ZF`位仍然没变化，而当走到第三步时，此时`EAX`的值为1，又将其减1后肯定就等于0了，`ZF`位置为1，后面的`JZ`跳转生效……

我们可以看到其实就是做了一连串的减法，到哪等于0后，就证明这个值原先为多少，由此可见微软的编译器还是很聪明的。不过这些代码从本质上来说还是属于if-esle范畴内的。

## 1.5.2、多分支的switch-case识别

我们平时在写程序时都会遇到一些问题，而这些问题肯定必须要用多分支的switch-case才能解决，但是你知道这种情况在反汇编状态下应该怎么去识别吗？下面我们就一起看看switch-case的另外一种体现方式，我们先看代码：

```
int _tmain(int argc, _TCHAR* argv[])
{
    switch (argc)
    {
    case 0:
        printf("argc=0",argc);
        break;
    case 1:
        printf("argc=%d",argc);
        break;
    case 6:
        printf("argc=%d",argc);
        break;
    case 7:
        printf("argc=%d",argc);
        break;
    case 9:
        printf("argc=%d",argc);
        break;
    default:
        printf("argc=%d,error!",argc);
    }
    return 0;
}
```

注意上面的case条件并不是有规律的，我们直接看它的Release版反汇编代码：

```
00401000  MOV EAX, DWORD PTR SS:[ESP+4]
00401004  CMP EAX, 9                               ;  Switch (cases 0..9)
00401007  JA SHORT Test_0.0040106F                 ;  如果大于最大值9则直接跳到Default处
00401009  JMP DWORD PTR DS:[EAX*4+401084]          ;  注意这里！！
00401010  PUSH 0                                   ;  Case 0 of switch 00401004
00401012  PUSH Test_0.004020F4                     ; /format = "argc=0"
00401017  CALL DWORD PTR DS:[<&MSVCR90.printf>]    ; \printf
0040101D  ADD ESP, 8
00401020  XOR EAX, EAX
00401022  RETN
00401023  PUSH 1                                   ; /<%d> = 1; Case 1 of switch 00401004
00401025  PUSH Test_0.004020FC                     ; |format = "argc=%d"
0040102A  CALL DWORD PTR DS:[<&MSVCR90.printf>]    ; \printf
00401030  ADD ESP, 8
00401033  XOR EAX, EAX
00401035  RETN
00401036  PUSH 6                                   ; /<%d> = 6; Case 6 of switch 00401004
00401038  PUSH Test_0.004020FC                     ; |format = "argc=%d"
0040103D  CALL DWORD PTR DS:[<&MSVCR90.printf>]    ; \printf
00401043  ADD ESP, 8
00401046  XOR EAX, EAX
00401048  RETN
00401049  PUSH 7                                   ; /<%d> = 7; Case 7 of switch 00401004
0040104B  PUSH Test_0.004020FC                     ; |format = "argc=%d"
00401050  CALL DWORD PTR DS:[<&MSVCR90.printf>]    ; \printf
00401056  ADD ESP, 8
00401059  XOR EAX, EAX
0040105B  RETN
0040105C  PUSH 9                                   ; /<%d> = 9; Case 9 of switch 00401004
0040105E  PUSH Test_0.004020FC                     ; |format = "argc=%d"
```
