
# [原创]pdf2word1.6算法分析

## 文章信息

【文章标题】: pdf2word1.6算法分析

【文章作者】: tjszlqq

【软件名称】: pdf2word1.6

【下载地址】: `454K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8G2N6$3&6D9L8$3q4V1i4K6u0W2j5$3!0E0i4K6u0r3f1p5c8r3x3W2N6G2M7X3c8Q4x3V1j5K6x3o6l9H3i4K6u0V1y4U0j5%4y4g2)9#2k6U0c8Q4x3X3b7I4x3o6t1#2x3o6M7J5x3#2)9J5k6h3S2@1L8h3H3`.

【加壳方式】: 无壳

【保护方式】: 序列号

【编写语言】: vc6.0

【使用工具】: `dfx.exe`,`immunity debugger`

【操作平台】: 正版windowsxp

【软件介绍】: 把pdf转换成rtf

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2008年02月23日 22:39:56

看雪老大不让分析国内的软件，那我就分析国外的，应该没问题，学了几十年的破解，终于找到一个超简单的让我破解了，太高兴了！

这个软件是共享软件，不注册只能试用一百次，运行时会弹出注册窗口，有错误提示“Series number error, please check it and try again."用`dfx.exe`查出`pdf2word1.6`为`vc6.0`编写，然后用`immunity debugger`载入，用ccdebuger教我们的字串参考，等这个软件载入后，在代码的某个地方右击，选所有参考字符串。在这个程序中一个有所有字符串的新窗口弹了出来，往上翻屏并且再次右击。然后选择搜索文本，再把“Series number error, please check it and try again."写进去。不过要记得勾上“区分大小写”，然后按ok.

你将会来到下面这里：

代码：
```
00429F6F | PUSH pdf2rtf.00468270 | ASCII "Series number error, please check it and try again."
```
现在按F2把这行设一个断点，在这一行的上面你会看到其它的一些字符串，也许有用，所以也把它们下断点，如下：

"Thank you registered" and "Thank you registered VeryPDF PDF2Word v1.6."

现在双击代码的某一行，按F9运行这个程序。

注册窗口又再次弹出来，然后填上一个电子邮件地址，一个假码，然后点ok.

返回到od,你会看到停在下面这行：

```
00429F6F . 68 70824600 PUSH pdf2rtf.00468270 ; |Text = "Series number error, please check it and try again."
```
这是那条错误消息，这个程序在显示它之前停了下来。

你会看到这条错误消息是`call MessageBoxA`的一部分，这是调用API函数显示这条消息，如果在这之用所有参考字串里面看过另外一条字符串，你会知道另外一个`MessageBoxA`和"Thank you registered VeryPDF PDF2Word v1.6."在一块。

这就意味着如我们输入正确的注册码，会显示"Thank you registered VeryPDF PDF2Word v1.6."这条消息。

这个程序会按照你的输入来显示那条信息，来做这事事的代码一般在这条消息的不远处，往上翻屏，你会看到一条包含`JNE`,`JE`,`JNZ`or `jz`的代码.这会跳转会发生在遇到一些事的时候。

一般测试条件就直接在跳转的上面，

你会看到如下代码：
```
00429F2E . 85C0 TEST EAX,EAX
00429F30 . 74 39 JE SHORT pdf2rtf.00429F6B
```
如`eax`为0,就跳，

如果我们跟踪这个跳转，发现跳过了"Thank you registered VeryPDF PDF2Word v1.6."，这样我们就知道`eax`为0就会显示错误消息。

我们可以用`nop`把 `JE SHORT pdf2rtf.00429F6B` 代替，让程序总显示正确消息，不过重启这个程序会继续要我们注册，所以我们需要继续搜索，我们需要找出`EAX`的值是什么时候得到的，

在`TEST EAX,EAX`上面二条指令我们可以看到
```
00429F26   . E8 F5F7FFFF    CALL pdf2rtf.00429720
```
这是一个`call`调用这个程序某一处的子程序，你可以用生命来打赌一定是这个子程序设定了这个`EAX`的值，所以我们需要找出这个子程序到底是怎么设的这个值，为了达到目的，我们按F2在 `CALL pdf2rtf.00429720` 上我们要设另外一个断点。

现在我们要再次重新载入这个程序，使它中断在这个`CALL`上，按快捷键`<CTRL>+<F2>`就可达到，当程序要退出的时候（另可以按左方向键和enter键）选是然后这个程序就重起了。

现在再次按`<F9>`让程序跑起来。

你会看到注册框会再次跳出来，所以你要把电子邮件地址和注册码填上，然后按ok.

就像你看到的那样程序中断在 `CALL pdf2rtf.00429720` 这条批令上了，

现在按 `<F7>`进入到这个`call`里面。

前面四条指令我们不感兴趣，所以我们从`00429725`开始分析，

下面就是我们看到的代码：
```
00429725  |. 8B7424 3C      MOV ESI,DWORD PTR SS:[ESP+3C]
00429729  |. 57             PUSH EDI
0042972A  |. 8A06           MOV AL,BYTE PTR DS:[ESI]
0042972C  |. 8A4E 01        MOV CL,BYTE PTR DS:[ESI+1]
0042972F  |. 8A56 0E        MOV DL,BYTE PTR DS:[ESI+E]
00429732  |. 884424 18      MOV BYTE PTR SS:[ESP+18],AL
00429736  |. 32C0           XOR AL,AL
00429738  |. 884C24 30      MOV BYTE PTR SS:[ESP+30],CL
0042973C  |. 8A4E 0F        MOV CL,BYTE PTR DS:[ESI+F]
0042973F  |. 884424 19      MOV BYTE PTR SS:[ESP+19],AL
00429743  |. 884424 31      MOV BYTE PTR SS:[ESP+31],AL
00429747  |. 884424 25      MOV BYTE PTR SS:[ESP+25],AL
0042974B  |. 884424 0D      MOV BYTE PTR SS:[ESP+D],AL
0042974F  |. 8A46 02        MOV AL,BYTE PTR DS:[ESI+2]
00429752  |. 3C 24          CMP AL,24
00429754  |. 885424 24      MOV BYTE PTR SS:[ESP+24],DL
00429758  |. 884C24 0C      MOV BYTE PTR SS:[ESP+C],CL
0042975C  |. 75 52          JNZ SHORT pdf2rtf.004297B0
```
我已经把到开始检查字串以上的整个块都放到上面了，现在开始分析。

第一行-->把我们输入的注册移到`esi`
第二行-->不重要
第三行-->把我们的注册码第一个字符移`al`
第四行-->把我们的注册码第二个字符移`cl`
第五行-->把我们的注册码第十五个字符移`dl`
第六行-->把`al`中的内容送`[esp+18]`
第七行-->清除`al`中的内容
第八行-->把`cl`中的内容移`[esp+30]`
第九行-->把第十六个字符移到`cl`
第十，十一，十二，十三行-->当`al`还是空的时候把`[ESP+19],[ESP+31],[ESP+25] & [ESP+D]`中的内容清空
第十四行-->把我们输入的第三个字节移到`al`
第十五行-->把`AL`的内容和`0x24`比较
第十六行-->把第十五个字节移至`[ESP+24]`
第十七行-->把第十六个字节移至 `[ESP+C]`
第十八行-->如果`AL`不等于`0x24`就跳到`004297B0`

如果你跟踪十八行的跳转你会看到它会跳到下面的代码：
```
004297B0  |> 5F             POP EDI
004297B1  |. 5E             POP ESI
004297B2  |. 33C0           XOR EAX,EAX
004297B4  |. 5D             POP EBP
004297B5  |. 83C4 30        ADD ESP,30
004297B8  \. C3             RETN
```
可以看到恢复被保存的值，把`eax`的值搞为0,然后返回到我们调用这个子程序的地方。

如果我们让它这么发生的话，那么`eax`会是0也就会给我们错误的提示框。

现在我们从上面这些代码可以知道什么？

-`al`必须等于`0x24`要不然就会得到错误提示框
-在等于`0x24`之前程序把第三个字符搞到`al`中
-这个程序把第十六个字节搞到`cl`

从上面的结论我可以得出第三个字符必须为`$`还有我们的注册码必有十六个字符好让我们把到搞到`cl`中去，

所以我们的注册码应该像这样 :..`$`.............

现在是接着的一片代码:
```
0042975E  |. 8B3D 4C964400  MOV EDI,DWORD PTR DS:[<&MSVCRT.atoi>]    ;  msvcrt.atoi
00429764  |. 8D5424 0C      LEA EDX,DWORD PTR SS:[ESP+C]
00429768  |. 52             PUSH EDX                                 ; /s
00429769  |. FFD7           CALL EDI                                 ; \atoi
0042976B  |. 8BE8           MOV EBP,EAX
0042976D  |. 8D4424 1C      LEA EAX,DWORD PTR SS:[ESP+1C]
00429771  |. 50             PUSH EAX
00429772  |. FFD7           CALL EDI
00429774  |. 03E8           ADD EBP,EAX
00429776  |. 83C4 08        ADD ESP,8
00429779  |. 83FD 0A        CMP EBP,0A
0042977C  |. 75 32          JNZ SHORT pdf2rtf.004297B0
```
我们来分析它吧：

第一行-->把函数`MSVCRT.atoi`的地址搞到`edi`中
第二行-->把第十六个字符搞到`edx`中
第三行-->把`edx`做为参数关起来
第四行-->来到系统区做运算，结果会算到`eax`中
第五行-->把`eax`中的内容搞到`ebp`中
第六行-->把我们的第一个码码搞到`eax`中
第七行-->把这次关`eax`
第八行-->把我们的码码搞到系统区做运算，最后再搞回来，
第九行-->加法运算，
第十行-->不重要
第十一行-->把`ebp`减去`0A`如果结果不是0就让`jnz`实现
第十二行-->见风使舵

现在我们从上面的代码得到什么了？

-第十六个字符搞到`atoi`,说明这个字符的值在0-9之中取
-第一个字符也被关过，说明也要在0-9中取
-第十六个加第一个要等于`0xa`。

所以我们得到下面的注册码：`1.$`............`9`

你可以想像，第一个字符各第十六个字符可以为任何东西只要它们都有是数字，还有它们加起来等于十进制的十，

到了看下一片代码的时候了:
```
0042977E  |. 8D4C24 24      LEA ECX,DWORD PTR SS:[ESP+24]
00429782  |. 51             PUSH ECX
00429783  |. FFD7           CALL EDI
00429785  |. 8D5424 34      LEA EDX,DWORD PTR SS:[ESP+34]
00429789  |. 8BE8           MOV EBP,EAX
0042978B  |. 52             PUSH EDX
0042978C  |. FFD7           CALL EDI
0042978E  |. 03E8           ADD EBP,EAX
00429790  |. 83C4 08        ADD ESP,8
00429793  |. 83FD 0A        CMP EBP,0A
00429796  |. 75 18          JNZ SHORT pdf2rtf.004297B0
```
还是让我们来分析一下:

第一条-->第十五个字符搞进`ecx`
第二条-->关起来
第三条-->用系统区的代码来搞`ecx`,最后把结果搞到`eax`
第四条-->第二个字符搞到`edx`
第五条-->被系统区搞过的`ecx`再搞到`ebp`中
第六条-->再把`ebp`关起来
第七条-->把`ebp`放出来，放到`eax`中
第八条-->第十五和第二个字符结婚，
第九条-->分析不出来
第十条-->生出来是不是男的，
第十一条-->不是男的，就丢了，

就像你看到的那样，这片代码跟上面分析的很类似，所以就不更一步分析了。

现在我们可以得到下面的代码:`12$`...........`89`

为了到达某个地方，我们还要分析一些代码：

最后要分析的一片代码：
```
00429798  |. 807E 03 24     CMP BYTE PTR DS:[ESI+3],24
0042979C  |. 75 12          JNZ SHORT pdf2rtf.004297B0
0042979E  |. 8A4E 05        MOV CL,BYTE PTR DS:[ESI+5]
004297A1  |. 33C0           XOR EAX,EAX
004297A3  |. 80F9 23        CMP CL,23
004297A6  |. 5F             POP EDI
004297A7  |. 5E             POP ESI
004297A8  |. 5D             POP EBP
004297A9  |. 0F94C0         SETE AL
004297AC  |. 83C4 30        ADD ESP,30
004297AF  |. C3             RETN
```
分析:

第一排-->第四个字符和`0x24`比较
第二排-->不相等就到`004297B0`
第三排-->送第六个字符到`cl`
第四排-->检查第六个字符是不是等于`0x23`
第五，六，七排-->不重要
第八行-->上面相等就把`al`设成1
第九行-->不重要
第十行-->回到调用的地方

得出第四节必须为`0x24`还有第六字节必须为`0x23`

如果都成立的话`al`就会设为1也就意谓着`eax`不必为零了，也就会使程序给我们正确的提示

概括:

- check if 3th == `$`
- check if 1st + 16th == 10
- check if 15th + 2nd == 10
- check if 4th == `$`
- check if 6th == `#`

其它的字符对建造一个有效的注册码来说不重要了，所以我们的注册码可以是这样:
`12$$.#`........`89`

你可以把点号的地方填入任何你想填的，只要你把第一个，第二个，第十五个还有十六个填对。

注册机源码:
```c
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{ char serial[50],name[50];
    int len,l1,l2,l15,l16;
printf("please input you email:");
scanf("%s",name);
printf("please input you series:");
    scanf("%s",serial );
    len=strlen(serial);
    if (len<16)
    printf("serial must more than 15 characters!");
    else
  { if(serial[0]<58&&serial[0]>48&&serial[1]<58&&serial[1]>48)
    {l1=serial[0];
l1=l1-48;
l2=serial[1];
l2=l2-48;
l15=10-l2;
l16=10-l1;
printf("serial is:%d%d$$%C#%C%C%C%C%C%C%C%C%d%d",l1,l2,serial[4],serial[6],serial[7],serial[8],serial[9],serial[10],serial[11],serial[12],serial[13],l15,l16);
}
else
{
```
