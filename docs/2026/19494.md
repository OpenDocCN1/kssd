
# [原创][代码之美]题7

题7的标准解法自然要属表查询法，本人第一次见表查询法是在《代码大全》，该书作者花了比较大的篇幅详细介绍该技巧；其次是在《80x86汇编语言程序设计教程》，该书也有好几个表查询法的例子。

本人查看了海风大侠的代码在VC6.0编译的情况。其源代码如下（这里仅察看`calcdate`即计算天数的函数）：

```
// 天数速算表

int days[12]={0,31,59,90,120,151,181,212,243,273,304,334};

int calcdate(int year,int month,int day)
{
    //题目没要求检查日期正确性，直接查表
    if( ((year%4==0) && (year%100!=0) || (year%400==0)) && month>2 )
        return days[month-1]+day+1;
    else
        return days[month-1]+day;
}
```

用默认的Release环境编译的`exe`，在OD是下面的样子，这里仅有`calcdate`函数：

```
00401000  /$  8B4C24 04     mov     ecx, dword ptr [esp+4]
00401004  |?  56            push    esi
00401005  |?  8BC1          mov     eax, ecx
00401007  |.  25 03000080   and     eax, 80000003
0040100C  |.  79 05         jns     short test.00401013		//jmp
0040100E  |?  48            dec     eax
0040100F  |?  83C8 FC       or      eax, FFFFFFFC
00401012  |?  40            inc     eax
00401013  |?  8B7424 0C     mov     esi, dword ptr [esp+C]
00401017  |?  75 10         jnz     short test.00401029		//jmp
00401019  |?  8BC1          mov     eax, ecx
0040101B  |?  57            push    edi
0040101C  |.  99            cdq
0040101D  |?  BF 64000000   mov     edi, 64
00401022  |?  F7FF          idiv    edi				//div
00401024  |.  5F            pop     edi
00401025  |?  85D2          test    edx, edx
00401027  |?  75 0E         jnz     short test.00401037		//jmp
00401029  |?  8BC1          mov     eax, ecx
0040102B  |?  B9 90010000   mov     ecx, 190
00401030  |?  99            cdq
00401031  |?  F7F9          idiv    ecx				//div
00401033  |?  85D2          test    edx, edx
00401035  |?  75 16         jnz     short test.0040104D		//jmp
00401037  |?  83FE 02       cmp     esi, 2
0040103A  |?  7E 11         jle     short test.0040104D		//jmp
0040103C  |.  8B14B5 FC7F40>mov     edx, dword ptr [esi*4+407FFC]
00401043  |?  8B4424 10     mov     eax, dword ptr [esp+10]
00401047  |?  5E            pop     esi
00401048  |?  8D4402 01     lea     eax, dword ptr [edx+eax+1]
0040104C  |.  C3            retn
0040104D  |?  8B04B5 FC7F40>mov     eax, dword ptr [esi*4+407FFC]
00401054  |.  8B4C24 10     mov     ecx, dword ptr [esp+10]
00401058  |?  03C1          add     eax, ecx
0040105A  |?  5E            pop     esi
0040105B  |?  C3            retn
```

可以看到有2次除法，5条跳转，36条指令，共计92字节。

本人要讲的并没有什么创新，只是尽自己水平用汇编重写了该函数，希望您不要对我对字节斤斤计较的做法发笑：

```
int Count(int year, int month, int day)
{
    int num;
    _asm 
    {
        xor edx, edx	//div指令，应给edx赋初值，这里为0，很多人忘记这个，
                        //比如在这里你mov edx，88888888h代替该指令程序是会崩溃的，
                        //因为这样做会导致整数溢出，而你并没有相应的异常处理程序
                        //来管这个。这里既然已经清edx，那么不如将天数保存在edx中
        cmp month, 2    //只有从3月开始才需要判断闰年，并不是月份为2的概率比年
                        //份为润年的概率大，相反的，闰年基本是每4年一次，月份的
                        //概率则是1/12。真正的原因是如果月份不大于2，则没有必要
                        //判断是否闰年，因为判断闰年是为了判断2月是否需要加1，
                        //而2月还没过完，就肯定不需要加1了。
        jle _NeedNotCheckReapYear 
        mov eax, year
        mov ecx, 100
        div ecx			//先用年份除100
        or  edx, edx
        jz  _NoRemainder//若整除，则用除的结果（在eax中），下面继续除4，即等价于除400
        mov eax, year	//若不整除，则eax重新赋为年份
_NoRemainder:
        xchg eax, edx	//交换eax，edx，这里是为了将结果放在edx中
        and  edx, 3h	//判断是否能被4整除
        setz dl			//若可以整除则将dl置1，否则清0；而and  edx, 3h已经将edx中除低2位
                        //之外的位清0，所以等价与将edx置1或清0
_NeedNotCheckReapYear:	//如果月份小于等于2，则不考虑是否闰年，值得注意的是，如果上面
                        //直接跳到这里，edx是被清0的，这样做只是为了省两个跳转和另外的指令，
                        //须知跳转会极大的降低CPU流水线的效率
        mov ecx, month
        add edx, days[ecx*4-4]	//查表，将该月之前的天数加到edx
        add edx, day	//将该月的日期加到edx
        mov num, edx
    }
    return num;
}
```

其实我很不想内联汇编，但是用纯汇编的话，处理文件又不是很方便，所以写了一个静态库（在附件中），但这样又增加了文件的数目，似乎规则里规定最好是单文件的方式，所以只有内联汇编咯。

反汇编的情况如下：

```
00401000  /$  55            push    ebp
00401001  |.  8BEC          mov     ebp, esp
00401003  |.  51            push    ecx
00401004  |.  33D2          xor     edx, edx
00401006  |.  837D 0C 02    cmp     dword ptr [ebp+C], 2
0040100A  |.  7E 18         jle     short test.00401024		//jmp
0040100C  |.  8B45 08       mov     eax, dword ptr [ebp+8]
0040100F  |.  B9 64000000   mov     ecx, 64
00401014  |.  F7F1          div     ecx				//div
00401016  |.  0BD2          or      edx, edx
00401018  |.  74 03         je      short test.0040101D		//jmp
0040101A  |.  8B45 08       mov     eax, dword ptr [ebp+8]
0040101D  |>  92            xchg    eax, edx
0040101E  |.  83E2 03       and     edx, 3
00401021  |.  0F94C2        sete    dl
00401024  |>  8B4D 0C       mov     ecx, dword ptr [ebp+C]
00401027  |.  03148D FC7F40>add     edx, dword ptr [ecx*4+407FFC]
0040102E  |.  0355 10       add     edx, dword ptr [ebp+10]
00401031  |.  8955 FC       mov     dword ptr [ebp-4], edx
00401034  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
00401037  |.  8BE5          mov     esp, ebp
00401039  |.  5D            pop     ebp
0040103A  \.  C3            retn
```

仅有两条跳转，一次除法，23条指令，59字节，可能会错哦，我只是大概数了数 。

其中判断是否闰年的算法如下：

![](https://bbs.kanxue.com/upload/attach/200811/183513_6rxihkdxxhugtoz.jpg)

其他代码是直接拷贝了海风的，但是做了一些修改，因为我觉得这些修改应当是比较合适的。完整的代码如下：

```
#include <stdio.h>

// 天数速算表
int days[12]={0,31,59,90,120,151,181,212,243,273,304,334};

int Count(int year, int month, int day)
{
    int num;
    _asm 
    {
        xor edx, edx	//div指令，应给edx赋初值，这里为0，很多人忘记这个，
                        //比如在这里你mov edx，88888888h代替该指令程序是会崩溃的，
                        //因为这样做会导致整数溢出，而你并没有相应的异常处理程序
                        //来管这个。这里既然已经清edx，那么不如将天数保存在edx中
        cmp month, 2    //只有从3月开始才需要判断闰年，并不是月份为2的概率比年
                        //份为润年的概率大，相反的，闰年基本是每4年一次，月份的
                        //概率则是1/12。真正的原因是如果月份不大于2，则没有必要
                        //判断是否闰年，因为判断闰年是为了判断2月是否需要加1，
                        //而2月还没过完，就肯定不需要加1了。
        jle _NeedNotCheckReapYear 
        mov eax, year
        mov ecx, 100
        div ecx			//先用年份除100
        or  edx, edx
        jz  _NoRemainder//若整除，则用除的结果（在eax中），下面继续除4，即等价于除400
        mov eax, year	//若不整除，则eax重新赋为年份
_NoRemainder:
        xchg eax, edx	//交换eax，edx，这里是为了将结果放在edx中
        and  edx, 3h	//判断是否能被4整除
        setz dl			//若可以整除则将dl置1，否则清0；而and  edx, 3h已经将edx中除低2位
                        //之外的位清0，所以等价与将edx置1或清0
_NeedNotCheckReapYear:	//如果月份小于等于2，则不考虑是否闰年，值得注意的是，如果上面
                        //直接跳到这里，edx是被清0的，这样做只是为了省两个跳转和另外的指令，
                        //须知跳转会极大的降低CPU流水线的效率
        mov ecx, month
        add edx, days[ecx*4-4]	//查表，将该月之前的天数加到edx
        add edx, day	//将该月的日期加到edx
        mov num, edx
    }
    return num;
}

void main(void)
{
    FILE *fin, *fout;
    int year, month, day;
    int testCount;

    if( NULL == (fin=fopen("in.txt", "r")) )
    {
        printf("Open in.txt ERROR");
        return ;
    }
    if( NULL == (fout=fopen("out.txt", "w")) )
    {
        printf("Open out.txt ERROR");
        return ;
    }

    fscanf(fin, "%d", &testCount);
    while(testCount--)
    {
        fscanf(fin, "%d %d %d", &year, &month, &day );
        fprintf(fout, "%d\n", Count(year,month,day) );
    }

    fclose(fin);
    fclose(fout);
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [flow.jpg](javascript:void(0);)
  （9.77kb，71次下载）
- [lib.rar](javascript:void(0);)
  （1.06kb，4次下载）
- [题7.rar](javascript:void(0);)
  （18.20kb，4次下载）
