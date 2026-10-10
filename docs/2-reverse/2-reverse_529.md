

# 第一章:1.4、循环分支的识别技巧

> 原文：[`www.pediy.com/kssd/pediy11/114283.html`](https://www.pediy.com/kssd/pediy11/114283.html)

循环几乎是在我们的代码中应用频率最高的一种编程方式，例如拷贝字符串、笨拙的从 1+2+3...100 的程序等等，数不尽数，本小节就带领各位了解循环分支，并熟悉各种循环互相之间的差异。  
    我们都知道 c 语言的循环主要分为三种，既 for、while 与 do-while，但是当我们以逆向的角度来看待这些循环结构的时候，我们会发现其本质上只有一种，而且即便我们再次细分的话，也仅仅能分离出两种情况。但是笔者仍然会将这一小节分为三部分讲解，以求尽量照顾读者们的接受能力。  
  
**1.4.1、do-while 循环的识别技巧**  
  
    “嘿！为什么要先学这里，不应该是 for 循环吗？”  
    相信很多读者都会产生以上疑问，要的就是这种效果！就让我们带着这个疑问开始这一节的学习，先看源码：  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{  
    int nNum = 26;  
    printf("Mom! I can sing my ABC!\\r\\n");  
  
    // 听！小 Baby 开始唱 ABC 了……  
    do   
    {  
        printf("%c ",0x41+(26-nNum) );  
        nNum--;  
    } while (nNum>0);  
    return 0;  
}  
  
    现在让我们从反汇编的角度看看小 Baby 是怎么唱歌的：  
  
004117DE  MOV DWORD PTR SS:\[EBP-8\], 1A             ; 16 进制的 0x1A 等于 10 进制的 26，不要在这里犯晕  
004117E7  PUSH Test\_0.00415C18                     ; /format = "Mom! I can sing my ABC!  
004117EC  CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]   ; \\printf  
004117F2  ADD ESP, 4  
004117FC  /MOV EAX, 1A                             ; <--!!!!!  
00411801  |SUB EAX, DWORD PTR SS:\[EBP-8\]  
00411804  |ADD EAX, 41                             ; EAX = 41+(1A-\[EBP-8\]) = 0x41+(26-nNum)  
00411809  |PUSH EAX                                ; /<%c>  
0041180A  |PUSH Test\_0.0041573C                    ; |format = "%c "  
0041180F  |CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]  ; \\printf  
00411815  |ADD ESP, 8  
0041181F  |MOV EAX, DWORD PTR SS:\[EBP-8\]  
00411822  |SUB EAX, 1  
00411825  |MOV DWORD PTR SS:\[EBP-8\], EAX           ; \[EBP-8\]-- = nNum--  
00411828  |CMP DWORD PTR SS:\[EBP-8\], 0             ; 看\[EBP-8\]是否仍大于 0，是的话则跳转到标记处继续  
0041182C  \\JG SHORT Test\_0.004117FC  
  
    是不是感觉很容易理解？这似乎与我们前面所讲的内容差距不大，那么就让我们在看看 Release 版的：  
  
00401000  PUSH ESI  
00401001  PUSH EDI  
00401002  MOV EDI, DWORD PTR DS:\[<&MSVCR90.printf>>;  MSVCR90.printf  
00401008  PUSH Test\_0.004020F4                     ; /format = "Mom! I can sing my ABC!  
0040100D  CALL EDI                                 ; \\printf  
0040100F  ADD ESP, 4  
00401012  MOV ESI, 41                              ; 将 ESI 加 0x41 后准备  
00401017  /PUSH ESI                                ; <--!!!!!  
00401018  |PUSH Test\_0.00402110                    ;  ASCII "%c "  
0040101D  |CALL EDI                                ; 又见此优化  
0040101F  |INC ESI                                 ; 直接将 ESI 加 1  
00401020  |ADD ESP, 8                              ; 为什么要将平衡堆栈操作放到这里？  
          |                                        ; 这与 CPU 的流水线有关系，我们目前先不深究。  
          |                                        ;  
00401023  |CMP ESI, 5B                             ; 快看看，直接与 0x5B 相比较了（Z 的 ASCII 码是 0x5A）  
00401026  \\JL SHORT Test\_0.00401017                ; 如果小于此值则继续  
00401028  POP EDI  
00401029  XOR EAX, EAX  
0040102B  POP ESI  
0040102C  RETN  
  
    通过以上代码我们不难看出，编译器直接将我们的代码优化为以下模式了：  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{  
    int nNum = 0x41;  
    printf("Mom! I can sing my ABC!\\r\\n");  
  
    // 听！小 Baby 开始唱 ABC 了……  
    do   
    {  
        printf("%c ",nNum++ );  
    } while (nNum<0x5B);  
    return 0;  
}  
  
    看看，多么聪明的编译器！直接看透了我们代码的本质！在感叹之余，不要忘记总结反汇编代码的特点，我们现在可以看到的最大的特点就是一个有条件判断的向上跳转，因此可以这样理解“如果我们看到了一个判断分支的跳转是向上的，那么这必然就是一个循环”。  
  
特点总结：  
DO\_TAG:  
  ......  
  ......  
  CMP XXX,XXX  
  JXX DO\_TAG  
  
  
**1.4.2、while 循环的识别技巧**  
  
    先看源码：  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{  
    int nNum = 26;  
    printf("Mom! I can sing my ABC!\\r\\n");  
  
    // 听！小 Baby 开始唱 ABC 了……  
    while(nNum>0)  
    {  
        printf("%c ",0x41+(26-nNum) );  
        nNum--;  
    }  
    return 0;  
}  
  
    再看 Debug 版反汇编代码：  
  
004117DE  MOV DWORD PTR SS:\[EBP-8\], 1A  
004117E7  PUSH Test\_0.00415C18                     ; /format = "Mom! I can sing my ABC!  
004117EC  CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]   ; \\printf  
004117F2  ADD ESP, 4  
004117FC  /CMP DWORD PTR SS:\[EBP-8\], 0  
00411800  |JLE SHORT Test\_0.00411830               ; 多了个判断，如果其小于等于 0 则跳出循环  
00411802  |MOV EAX, 1A  
00411807  |SUB EAX, DWORD PTR SS:\[EBP-8\]  
0041180A  |ADD EAX, 41  
0041180F  |PUSH EAX                                ; /<%c>  
00411810  |PUSH Test\_0.0041573C                    ; |format = "%c "  
00411815  |CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]  ; \\printf  
0041181B  |ADD ESP, 8  
00411825  |MOV EAX, DWORD PTR SS:\[EBP-8\]  
00411828  |SUB EAX, 1  
0041182B  |MOV DWORD PTR SS:\[EBP-8\], EAX  
0041182E  \\JMP SHORT Test\_0.004117FC  
  
    细心的读者可能发现了，这与 do-while 循环几乎如出一辙，仅仅在循环头部多了两条用于判断是否跳出循环的指令，那 Release 版的又会是怎样的呢？  
  
00401000  PUSH ESI  
00401001  PUSH EDI  
00401002  MOV EDI, DWORD PTR DS:\[<&MSVCR90.printf>>;  MSVCR90.printf  
00401008  PUSH Test\_0.004020F4                     ; /format = "Mom! I can sing my ABC!  
0040100D  CALL EDI                                 ; \\printf  
0040100F  ADD ESP, 4  
00401012  MOV ESI, 41  
00401017  /PUSH ESI  
00401018  |PUSH Test\_0.00402110                    ;  ASCII "%c "  
0040101D  |CALL EDI  
0040101F  |INC ESI  
00401020  |ADD ESP, 8  
00401023  |CMP ESI, 5B  
00401026  \\JL SHORT Test\_0.00401017  
00401028  POP EDI  
00401029  XOR EAX, EAX  
0040102B  POP ESI  
0040102C  RETN  
  
    请不要怀疑我复制错了代码，事实就是这样！do-while 与 while 生成的 Release 版在这里看就是 100%完全相同的。  
    编译器很明显的已经探测出了我们的循环判断用的是一个常量，因此就不存在首次执行条件不匹配的情况。既然如此，它为什么还要在循环前面加上那个判断分支来浪费我们的空间与时间呢？  
    当然，如果我们将它的判断条件改为一个变量，那么就是另外一番景象了：  
  
00401000  PUSH EBX  
00401001  MOV EBX, DWORD PTR DS:\[<&MSVCR90.printf>>;  MSVCR90.printf  
00401007  PUSH EDI  
00401008  PUSH Test\_0.004020F4                     ; /format = "Mom! I can sing my ABC!  
0040100D  CALL EBX                                 ; \\printf  
0040100F  MOV EDI, DWORD PTR SS:\[ESP+10\]           ; 取得参数（其实就是 main 函数的 argc）  
00401013  ADD ESP, 4  
00401016  TEST EDI, EDI                            ; 测试参数是否为 0  
00401018  JLE SHORT Test\_0.00401034                ; 如果为 0 则跳出循环  
0040101A  PUSH ESI  
0040101B  MOV ESI, 5B                              ; 0x5B = 0x41+26   
00401020  SUB ESI, EDI                             ; 用 0x5B 减去参数  
00401022  /PUSH ESI  
00401023  |PUSH Test\_0.00402110                    ;  ASCII "%c "  
00401028  |CALL EBX  
0040102A  |DEC EDI                                 ; 参数减 1  
0040102B  |ADD ESP, 8  
0040102E  |INC ESI                                 ; ESI 加 1  
0040102F  |TEST EDI, EDI  
00401031  \\JG SHORT Test\_0.00401022                ; 如果参数大于 ESI 则结束循环  
00401033  POP ESI  
00401034  POP EDI  
00401035  XOR EAX, EAX  
00401037  POP EBX  
00401038  RETN  
  
    我们可以看出用变量做判断条件很明显与常量不一样，而关于优化，很显然他只是单纯的将我们的“0x41+(26-argc)”优化成“0x5B-argc”。  
  
特点总结：  
WHILE\_TAG:  
  CMP XXX,XXX  
  JXX WHILE\_END\_TAG  
  ......  
  ......  
  CMP XXX,XXX  
  JXX WHILE\_TAG  
WHILE\_END\_TAG:  
  
  
**1.4.3、for 循环的识别技巧**  
  
    for 循环与 while 循环本质上都是一样的，唯一的不同在于 for 循环在循环体内多了一个步长部分，接下来我们一起看看 for 循环的样子，先看源码：  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{  
    printf("Mom! I can sing my ABC!\\r\\n");  
  
    // 听！小 Baby 开始唱 ABC 了……  
    for (int nNum = 26;nNum>0;nNum--)  
    {  
        printf("%c ",0x41+(26-nNum) );  
    }  
    return 0;  
}  
  
    接下来我们再看看 Debug 版的反汇编代码：  
  
004117E0  PUSH Test\_0.00415C18                     ; /format = "Mom! I can sing my ABC!  
004117E5  CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]   ; \\printf  
004117EB  ADD ESP, 4  
004117F5  MOV DWORD PTR SS:\[EBP-8\], 1A  
004117FC  JMP SHORT Test\_0.00411807  
004117FE  /MOV EAX, DWORD PTR SS:\[EBP-8\]           ; / 步长控制部分开始  
00411801  |SUB EAX, 1                              ; | 步长为 1  
00411804  |MOV DWORD PTR SS:\[EBP-8\], EAX           ; \\ 将操作后的结果传回给局部变量，步长操作结束  
00411807  |CMP DWORD PTR SS:\[EBP-8\], 0  
0041180B  |JLE SHORT Test\_0.00411832               ; 如果此变量小于等于 0 则结束循环  
0041180D  |MOV EAX, 1A  
00411812  |SUB EAX, DWORD PTR SS:\[EBP-8\]  
00411815  |ADD EAX, 41                             ; 0x41+(0x1A-\[EBP-8\])  
0041181A  |PUSH EAX                                ; /<%c>  
0041181B  |PUSH Test\_0.0041573C                    ; |format = "%c "  
00411820  |CALL DWORD PTR DS:\[<&MSVCR90D.printf>\]  ; \\printf  
00411826  |ADD ESP, 8  
00411830  \\JMP SHORT Test\_0.004117FE               ; 跳到循环头部  
  
    看到这里不知道各位读者们是否发现了什么，记得我当时学到这里时，直觉上认识到了以下两点：  
（1、）显然循环语句是 do-while 先诞生的，而后是 while，最后才是 for，这从侧面上讲 for 应该是最“高级”的了。  
（2、）从执行效率上看，代码最短且判断最少的就是 do-while 循环了。  
  
    Release 版反汇编：  
  
00401000  PUSH ESI  
00401001  PUSH EDI  
00401002  MOV EDI, DWORD PTR DS:\[<&MSVCR90.printf>>;  MSVCR90.printf  
00401008  PUSH Test\_0.004020F4                     ; /format = "Mom! I can sing my ABC!  
0040100D  CALL EDI                                 ; \\printf  
0040100F  ADD ESP, 4  
00401012  MOV ESI, 41  
00401017  /PUSH ESI  
00401018  |PUSH Test\_0.00402110                    ;  ASCII "%c "  
0040101D  |CALL EDI  
0040101F  |INC ESI  
00401020  |ADD ESP, 8  
00401023  |CMP ESI, 5B  
00401026  \\JL SHORT Test\_0.00401017  
00401028  POP EDI  
00401029  XOR EAX, EAX  
0040102B  POP ESI  
0040102C  RETN  
  
    又是常量惹的祸，这段代码与 do-while、while 一模一样，有疑问的读者可以返回上面仔细观察一下，连地址都是一样的。  
    现在正好印证了我开篇时讲的一句话“其本质上只有一种”，很明显的，我们的 while 与 for 都是以 do-while 为基础框架的，只不过是在里面加了一些小判断。为了让各位读者更清晰的看到它们之间的异同，我再为各位献上一个变量版的：  
  
00401000  PUSH EBX  
00401001  MOV EBX, DWORD PTR DS:\[<&MSVCR90.printf>>;  MSVCR90.printf  
00401007  PUSH EDI  
00401008  PUSH Test\_0.004020F4                     ; /format = "Mom! I can sing my ABC!  
0040100D  CALL EBX                                 ; \\printf  
0040100F  MOV EDI, DWORD PTR SS:\[ESP+10\]  
00401013  ADD ESP, 4  
00401016  TEST EDI, EDI  
00401018  JLE SHORT Test\_0.00401034  
0040101A  PUSH ESI  
0040101B  MOV ESI, 5B  
00401020  SUB ESI, EDI  
00401022  /PUSH ESI  
00401023  |PUSH Test\_0.00402110                    ;  ASCII "%c "  
00401028  |CALL EBX  
0040102A  |DEC EDI  
0040102B  |ADD ESP, 8  
0040102E  |INC ESI  
0040102F  |TEST EDI, EDI  
00401031  \\JG SHORT Test\_0.00401022  
00401033  POP ESI  
00401034  POP EDI  
00401035  XOR EAX, EAX  
00401037  POP EBX  
00401038  RETN  
  
    再重申一下，笔者并没有搞错，以变量为判断条件的 for 循环与 while 循环所生成的代码是完全相同的，连地址都一样……  
  
特点总结：  
FOR\_START\_TAG:  
    初始化块  
    JMP CMP\_TAG  
STEP\_TAG:  
    步长块  
CMP\_TAG：  
    反条件判断  
    JXX  FOR\_END\_TAG  
    ……  
    ……  
JXX STEP\_TAG  
  
    到此，我们应该可以做一个总结了，Debug 版下三种循环各不相同，Release 版下可总结如下：  
（1、）当循环采用常量为判断条件时，相同逻辑的三种循环生成的代码完全相同。  
（2、）当循环采用变量为判断条件时，相同逻辑的 while 与 for 生成的代码完全相同，而 do-while 则自成一格。  
  
  
**小插曲：循环体的语句外提优化**  
  
    我们看看下面这段代码：  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{  
  printf("Mom! I can sing my ABC!\\r\\n");  
  
  // 听！小 Baby 开始唱 ABC 了……  
  for (int nNum = 24;nNum>0;nNum--)  
  {  
    argc = (int)argv;  
    printf("%c ",0x41+(26-nNum) );  
  }  
  printf("%p",argc);  
  return 0;  
}  
  
   通过这段代码我们可以发现一处可以优化的地方，就是“argc = (int)argv”这条语句，很明显  
  
00401000  PUSH ESI  
00401001  PUSH EDI  
00401002  MOV EDI, DWORD PTR DS:\[<&MSVCR90.printf>>;  MSVCR90.printf  
00401008  PUSH Test\_0.004020F4                     ; /format = "Mom! I can sing my ABC!  
0040100D  CALL EDI                                 ; \\printf  
0040100F  ADD ESP, 4  
00401012  MOV ESI, 43  
00401017  /PUSH ESI  
00401018  |PUSH Test\_0.00402110                    ;  ASCII "%c "  
0040101D  |CALL EDI  
0040101F  |INC ESI  
00401020  |ADD ESP, 8  
00401023  |CMP ESI, 5B  
00401026  \\JL SHORT Test\_0.00401017  
00401028  MOV EAX, DWORD PTR SS:\[ESP+10\]           ; EAX = (int)argv; <--注意这里  
0040102C  PUSH EAX  
0040102D  PUSH Test\_0.00402114                     ;  ASCII "%p"  
00401032  CALL EDI  
00401034  ADD ESP, 8  
00401037  POP EDI  
00401038  XOR EAX, EAX  
0040103A  POP ESI  
0040103B  RETN  
  
    由上面代码可知，循环体内很明显没有我们“argc = (int)argv;”的代码，再向下看一行才知道，这段代码被提到了外面，这就是编译进行的代码外提优化。  
    本小节到此就结束了，笔者在本小节向大家详细的介绍了三种循环结构在逆向时的一些需要注意的特点，当你怎能快速的花柱这些特点之后，剩下的就是反复的练习了。  
  
**[【返回到目录】：http://bbs.pediy.com/showthread.php?t=113689](http://bbs.pediy.com/showthread.php?t=113689)** 

学完此节，将楼主的文章关键部分提取出来，方便记忆，呵呵。  
  
几种循环的汇编代码结构：  
  
do while 的：  
  
DO\_TAG:  
  ......  
  ......  
  CMP XXX,XXX  
  JXX DO\_TAG  
  
while 的 ：  
  
WHILE\_TAG:  
  CMP XXX,XXX  
  JXX WHILE\_END\_TAG  
  ......  
  ......  
  CMP XXX,XXX  
  JXX WHILE\_TAG  
WHILE\_END\_TAG:  
  
  
For 循环的(根据个人理解总结的，正确与否请自辨![](img/cda69129882b40377a87144c782680e1.jpg "微笑")) ：  
  
  xxxx;初始化变量  
  jmp For\_TAG2:  
For\_TAG:  
  xxx;变量递增或递加操作  
For\_TAG2:  
  CMP XXX,XXX  
  JXX For\_END\_TAG  
  ......  
  ......  
  CMP XXX,XXX  
  JXX For\_TAG  
For\_END\_TAG:  
  
从生成的汇编码来看，应该在代码中多使用 do...while 结构，呵呵。  
  
看了这几期，感觉编译器，真的太牛 X 了，优化真做到极致了，关键还能在那么短的时间内生成好可执行文件。  
  
楼主，辛苦了，不影响正常工作的情况下，能坚持下来。给点力，嚯嚯。。。。  
  
PS：楼主又忘加返回主目录的链接了。![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")