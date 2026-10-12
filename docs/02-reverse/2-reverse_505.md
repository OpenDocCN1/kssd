
# 程序逆向技术之三流程控制

> 原文：[`www.pediy.com/kssd/pediy10/60451.html`](https://www.pediy.com/kssd/pediy10/60451.html)

很不容易，又来写字了，很不喜欢敲字，不知道为什么，今天就给大家说说逆向里的流程控制，说白了就是 `if` 和 `switch`，`for` 和 `while` 之间的事，今天很想给大家说很多，但是苦于语言很差，就简单明了点，跟前一篇来讲，这一篇比较难点，但是对于曾经的我来说已经是啃不动的骨头了，当 10 多天的耐心钻研后，我已经能在 `ASM` 和 `C++` 之间任意游走了，但是我现在只差的是耐心和经验。

当我看到 `ASM` 时，我就想到了 `C++`，虽然不一定编译后的结果和所看到的一样，而我接近正确的代码是越来越近，准确率也比以前搞了 1 个百分点，这对于我来说已经是进步了，毕竟做为一个典型的菜鸟，我无法解释我的智力为什么比平常人要少很多，但是我可以解释的是在努力后，我的收获却比平常人多，那大概是我的努力比较多或者来说这叫???????。

慢慢的苦傲总是一种痛苦，看着 `ASM` 的语句，突然发现一句变两句时，才记得自己已经需要休息了，虽然本本带给我的快乐是那么的少，我已经很荣幸、很满足了。但是它送给我的辐射让我知道，面对它不只是要带眼镜的问题，还有退化细胞的强力功效，如果想慢性自杀，那是最好的药了。今天精神状况不佳，没心思罗列条理，我就用一个例子说明吧~，大概是杀蚂蚁杀多了吧，电脑渐渐表现的不稳定，让我觉得不能再懒了，需要动手了，大开杀戒。当然，我没心思手动搞了，那瑞星、金山、卡巴过了一遍。我很荣幸的是基本上解决了，我不用在亲自动手了。或许依然在我的电脑里，我就不管了，随它去吧~~~~。实在没那么多的时间和精力去做那么无聊的事。........

对于学习是要有耐心的，我特意从茫茫代码中找了一段比较典型的代码，虽然比较多但是很容易说明问题的答案，我们来看看：

```
//-------------------------------------------------------------------------------------
00411EF0  /$  55             push    ebp
00411EF1  |.  8BEC           mov     ebp, esp
00411EF3  |.  81EC B0000000 sub     esp, 0B0
00411EF9  |.  53             push    ebx
00411EFA  |.  56             push    esi
00411EFB  |.  57             push    edi
00411EFC  |.  C605 2845E600>mov      byte ptr [E64528], 0          ;这里表示有个全局变量被声明了
00411F03  |>  A1 9045E600   /mov     eax, dword ptr [E64590]       ;根据函数就可以知道
00411F08  |.  50            |push    eax                            ;
00411F09  |.  E8 A1980400   |call    0045B7AF                       ;call fgetc,根据函数我们就知道 EAX 为 FILE 指针
00411F0E  |.  83C4 04       |add     esp, 4
00411F11  |.  8845 FC       |mov     byte ptr [ebp-4], al           ;AL 为读取到的字节,放入的是局部变量
00411F14  |.  0FBE4D FC     |movsx   ecx, byte ptr [ebp-4]          ;
//======================================================================
```

这里说明一下,在我逆的时候发现了一个很好玩的规律:

有符号类型转换为 `MOVSX`,无符号类型转换为 `AND`,怎么理解呢?

我来举个例子:

```
char cc=0x00;
int  ii=(int)cc;
//----------------------
mov     byte ptr [ebp-4], 0
movsx   eax, byte ptr [ebp-4]
mov     dword ptr[ebp-8],eax
//----------------------
BYTE byte=0x00;
int  ii=(int)byte;
//----------------------
mov     byte ptr [ebp-4], 0
mov     eax, byte ptr [ebp-4]
and     eax, 0xFF
mov     dword ptr[ebp-8],eax
//----------------------
```

看的明白了吧,在转换的时候方法是不一样的,所以 `BYTE` 和 `char` 是不完全一样的哦,以后别混淆了.

`//=====================================================================`

```
00411F18  |.  83F9 FF       |cmp     ecx, -1               ;这里一看就知道 byte ptr [ebp-4]为 char 类型不是 BYTE
00411F1B  |.  75 0A         |jnz     short 00411F27
00411F1D  |.  B8 02000000   |mov     eax, 2                ;EAX 在函数里通常被规定为返回值,也就是这个函数是有
00411F22  |.  E9 A2020000   |jmp     004121C9              ;返回值 2 的,可以断定这个函数为 int 类型
00411F27  |>  0FBE55 FC     |movsx   edx, byte ptr [ebp-4]
00411F2B  |.  83FA 2F       |cmp     edx, 2F               ;判断 0x2F,判断为 if(XX==XX)才会出现不等于.
00411F2E  |.  75 37         |jnz     short 00411F67
00411F30  |.  A1 9045E600   |mov     eax, dword ptr [E64590]
00411F35  |.  50            |push    eax
00411F36  |.  E8 74980400   |call    0045B7AF              ;fgetc
00411F3B  |.  83C4 04       |add     esp, 4
00411F3E  |.  8845 FC       |mov     byte ptr [ebp-4], al
00411F41  |.  0FBE4D FC     |movsx   ecx, byte ptr [ebp-4]
00411F45  |.  83F9 2F       |cmp     ecx, 2F
00411F48  |.  75 1D         |jnz     short 00411F67
00411F4A  |>  8B15 9045E600 |/mov     edx, dword ptr [E64590]
00411F50  |.  52            ||push    edx
00411F51  |.  E8 59980400   ||call    0045B7AF              ;fgetc
00411F56  |.  83C4 04       ||add     esp, 4
00411F59  |.  8845 FC       ||mov     byte ptr [ebp-4], al
00411F5C  |.  0FBE45 FC     ||movsx   eax, byte ptr [ebp-4]
00411F60  |.  83F8 0A       ||cmp     eax, 0A
00411F63  |.  74 02         ||je      short 00411F67
00411F65  |.^ EB E3         |\jmp     short 00411F4A
00411F67  |>  0FBE4D FC     |movsx   ecx, byte ptr [ebp-4]
00411F6B  |.  51            |push    ecx
00411F6C  |.  E8 56960400   |call    0045B5C7              ;isspace
00411F71  |.  83C4 04       |add     esp, 4
00411F74  |.  85C0          |test    eax, eax
00411F76  |.^ 75 8B         \jnz     short 00411F03
//===================================================================
```

`for` 和 `while` 的区别就是 `for` 在循环的时候都是 `jmp` 来跳转而 `while` 通常都是如 `jnz`,`je`,`jg`,`jl` 等等.

上面的汇编一看就是 `do..while` 循环,于是我把它的逻辑流程写出来:

```
////////////////////////////////////////////////////////////////////////
  do{
    XXXX=fgetc(XXXX);  //返回的值放入 char 类型变量
    if(XXXX==-1){return 2;}
    if(XXXX==0x2F)
    {
      XXXX=fgetc(XXXX);
      if(XXXX==0x2F)
      {
        for(;XXXX!=0x0A&&XXXX!=-1;){XXXX=fgetc(XXXX);}
        if(XXXX==-1){return 2;}
      }
    }
  }while(::isspace(XXXX)!=0);
/////////////////////////////////////////////////////////////////////
```

我编译后和原来汇编代码完全一样,不知道大家试了会不会有差别.

下面来说一点规律:

```
if(a==b)
//------------------------
JNZ XXXXXXXX
JMP XXXXXXXX
//------------------------

if(a!=b)
//-------------------------
JE  XXXXXXXX
JMP XXXXXXXX
//-------------------------
```

有符号：
`jle`:>
`jge`:<
`jl` :>=
`jg` :<=
...
(注:无符号类似)

在 `&&` 和 `||` 中的逻辑条件也是不一样的,大家可以试试.总的来说,只要是汇编里的和 `C++` 里的刚好是反的,碰到 `||` 又就是一样的.

`//===================================================================`

```
00411F78  |.  0FBE55 FC     movsx   edx, byte ptr [ebp-4]
00411F7C  |.  8955 90       mov     dword ptr [ebp-70], edx
00411F7F  |.  8B45 90       mov     eax, dword ptr [ebp-70]
00411F82  |.  83E8 22       sub     eax, 22
00411F85  |.  8945 90       mov     dword ptr [ebp-70], eax
00411F88  |.  837D 90 5B    cmp     dword ptr [ebp-70], 5B
00411F8C  |.  0F87 80010000 ja      00412112
00411F92  |.  8B55 90       mov     edx, dword ptr [ebp-70]
00411F95  |.  33C9          xor     ecx, ecx
00411F97  |.  8A8A F0214100 mov     cl, byte ptr [edx+4121F0]
00411F9D  |.  FF248D D02141>jmp     dword ptr [ecx*4+4121D0]
//=====================================================================
```

看到这样的代码,大家觉得会是什么呢?`C++` 怎么写才会是这样呢?实际很简单:

```
switch(XXX)
{
  case XXXX:
  ....
  default :
  {....}
}
```

这样的代码就是会被编译成上面的那种形式.

`sub eax, 22` 说明在这个判断里最小的值为 `0x22`
`cmp dword ptr [ebp-70], 5B` 说明最大值为 `0x22+0x5B=0x7D`

我们怎么知道有多少项呢?怎么知道每项应该多少呢??

实际我们可以看数组:

`mov cl, byte ptr [edx+4121F0]` 中的 `0x004121F0` 处为:

```
004121F0  00 01 07 07 07 07 07 07 07 07 02 03 03 07 03 03  .
00412200  03 03 03 03 03 03 03 03 07 04 07 07 07 07 07 07
00412210  07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07
00412220  07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07
00412230  07 07 07 07 07 07 07 07 07 07 07 07 07 07 07 07
00412240  07 07 07 07 07 07 07 07 07 05 07 06 CC CC CC CC  烫烫
```

根据上面的分析,我们知道总计处理了 8 项即 0-7 为 8.

而 `0x004121D0` 处为 8 个地址

```
004121D0   9D 20 41 00 A4 1F 41 00 CC 1F 41 00 08 20 41 00  ?A.?A.?A. A.
004121E0   B8 1F 41 00 E0 1F 41 00 F4 1F 41 00 12 21 41 00  ?A.?A.?A.!A.
```

根据汇编代码不难看出来,上面的 8 个地址代表 `case` 处理的 8 处代码.我们可以很容易的知道
`0x004121F0` 为对应 `case 0x22`,我们依次找出对应为:

`F0-22 F1-23 F2-24 F3-25 F4-26 F5-27 F6-28 F7-29 F8-2A F9-2B FA-2C FB-2D FC-2E FD-2F FE-30 FF-31`
....
可以推断处理方式为 `22,23,2C,39,3B,7B,7D` 为处理不同,其它都是第 7 种处理也就是 `default`,在对应排序的表中是按从小到大的排,在代码中是按源代码的编写来排,于是整理为下面代码:

```
  switch((int)ibuf)
  {
  case 0x23:    //处理对应 1
    {
      ...
    }
  case 0x3B:    //处理对应 4
    {
      ...
    }
  case 0x2C:    //处理对应 2
    {
      ...
    }
  case 0x7B:    //处理对应 5
    {
      ...
    }
  case 0x7D:    //处理对应 6
    {
      ...
    }
  case 0x2D:
  case 0x2E:
  case 0x30:
  case 0x31:
  case 0x32:
  case 0x33:
  case 0x34:
  case 0x35:
```



```c
case 0x36:  
case 0x37:  
case 0x38:  
case 0x39:    //处理对应 3  
{  
  ...  
}  
case 0x22:    //处理对应 0  
{  
  ...  
}  
default:      //处理对应 7  
{  
  ....  
}  
}  
```

```text
//=====================================================================  

00411FA4  |>  C705 2445E600>mov      dword ptr [E64524], 23      ;case 0x23  
00411FAE  |.  A1 2445E600   mov      eax, dword ptr [E64524]  
00411FB3  |.  E9 11020000   jmp      004121C9  
00411FB8  |>  C705 2445E600>mov      dword ptr [E64524], 3B      ;case 0x3B  
00411FC2  |.  A1 2445E600   mov      eax, dword ptr [E64524]  
00411FC7  |.  E9 FD010000   jmp      004121C9  
00411FCC  |>  C705 2445E600>mov      dword ptr [E64524], 2C      ;case 0x2C  
00411FD6  |.  A1 2445E600   mov      eax, dword ptr [E64524]  
00411FDB  |.  E9 E9010000   jmp      004121C9  
00411FE0  |>  C705 2445E600>mov      dword ptr [E64524], 7B        ;case 0x7B  
00411FEA  |.  A1 2445E600   mov      eax, dword ptr [E64524]  
00411FEF  |.  E9 D5010000   jmp      004121C9  
00411FF4  |>  C705 2445E600>mov      dword ptr [E64524], 7D         ;case  0x7D  
00411FFE  |.  A1 2445E600   mov      eax, dword ptr [E64524]  
00412003  |.  E9 C1010000   jmp      004121C9  
00412008  |>  A1 9045E600   mov      eax, dword ptr [E64590]   ;case 0x39  
0041200D  |.  50            push    eax  
0041200E  |.  0FBE4D FC     movsx   ecx, byte ptr [ebp-4]  
00412012  |.  51            push    ecx  
00412013  |.  E8 D3970400   call    0045B7EB                ;ungetc()  
00412018  |.  83C4 08       add     esp, 8  
0041201B  |.  8D55 94       lea     edx, dword ptr [ebp-6C]  
0041201E  |.  8955 F8       mov     dword ptr [ebp-8], edx  
00412021  |>  A1 9045E600   /mov     eax, dword ptr [E64590]  
00412026  |.  50            |push    eax  
00412027  |.  E8 B4970400   |call    0045B7E0                ;fgetc()  
0041202C  |.  83C4 04       |add     esp, 4  
0041202F  |.  8845 FC       |mov     byte ptr [ebp-4], al  
00412032  |.  0FBE4D FC     |movsx   ecx, byte ptr [ebp-4]  
00412036  |.  83F9 FF       |cmp     ecx, -1  
00412039  |.  74 36         |je      short 00412071  
0041203B  |.  0FBE55 FC     |movsx   edx, byte ptr [ebp-4]  
0041203F  |.  83FA 2E       |cmp     edx, 2E  
00412042  |.  74 1A         |je      short 0041205E  
00412044  |.  0FBE45 FC     |movsx   eax, byte ptr [ebp-4]  
00412048  |.  50            |push    eax  
00412049  |.  E8 24950400   |call    0045B572                ;isdigit()  
0041204E  |.  83C4 04       |add     esp, 4  
00412051  |.  85C0          |test    eax, eax  
00412053  |.  75 09         |jnz     short 0041205E  
00412055  |.  0FBE4D FC     |movsx   ecx, byte ptr [ebp-4]  
00412059  |.  83F9 2D       |cmp     ecx, 2D  
0041205C  |.  75 13         |jnz     short 00412071  
0041205E  |>  8B55 F8       |mov     edx, dword ptr [ebp-8]  
00412061  |.  8A45 FC       |mov     al, byte ptr [ebp-4]  
00412064  |.  8802          |mov     byte ptr [edx], al  
00412066  |.  8B4D F8       |mov     ecx, dword ptr [ebp-8]  
00412069  |.  83C1 01       |add     ecx, 1  
0041206C  |.  894D F8       |mov     dword ptr [ebp-8], ecx  
0041206F  |.^ EB B0         \\jmp     short 00412021  
00412071  |>  8B55 F8       mov     edx, dword ptr [ebp-8]  
00412074  |.  C602 00       mov     byte ptr [edx], 0  
00412077  |.  8D45 94       lea     eax, dword ptr [ebp-6C]  
0041207A  |.  50            push    eax  
0041207B  |.  E8 D8960400   call    0045B758                ;atof()  
00412080  |.  83C4 04       add     esp, 4  
00412083  |.  D91D 8C45E600 fstp    dword ptr [E6458C]  
```

```c
TokenNumber=(float)atof((char*)&dwUnk);   //float 为 fstp    dword ptr [E6458C]  

//double 为 fstp    qword ptr [E6458C]  

//=================================================  
```

```text
00412089  |.  C705 2445E600>mov      dword ptr [E64524], 1  
00412093  |.  A1 2445E600   mov      eax, dword ptr [E64524]  
00412098  |.  E9 2C010000   jmp      004121C9  
0041209D  |>  C745 F8 2845E>mov      dword ptr [ebp-8], 00E64528   ;case 0x22  
004120A4  |>  8B0D 9045E600 /mov      ecx, dword ptr [E64590]  
004120AA  |.  51            |push    ecx  
004120AB  |.  E8 30970400   |call    0045B7E0                     ;fgetc()  
004120B0  |.  83C4 04       |add     esp, 4  
004120B3  |.  8845 FC       |mov     byte ptr [ebp-4], al  
004120B6  |.  0FBE55 FC     |movsx   edx, byte ptr [ebp-4]  
004120BA  |.  83FA FF       |cmp     edx, -1  
004120BD  |.  74 1C         |je      short 004120DB  
004120BF  |.  0FBE45 FC     |movsx   eax, byte ptr [ebp-4]  
004120C3  |.  83F8 22       |cmp     eax, 22  
004120C6  |.  74 13         |je      short 004120DB  
004120C8  |.  8B4D F8       |mov     ecx, dword ptr [ebp-8]  
004120CB  |.  8A55 FC       |mov     dl, byte ptr [ebp-4]  
004120CE  |.  8811          |mov     byte ptr [ecx], dl  
004120D0  |.  8B45 F8       |mov     eax, dword ptr [ebp-8]  
004120D3  |.  83C0 01       |add     eax, 1  
004120D6  |.  8945 F8       |mov     dword ptr [ebp-8], eax  
004120D9  |.^ EB C9         \\jmp     short 004120A4                     ;判断为 for 循环  
004120DB  |>  0FBE4D FC     movsx   ecx, byte ptr [ebp-4]  
004120DF  |.  83F9 22       cmp     ecx, 22  
004120E2  |.  74 14         je      short 004120F8  
004120E4  |.  8B15 9045E600 mov     edx, dword ptr [E64590]  
004120EA  |.  52            push    edx  
004120EB  |.  0FBE45 FC     movsx   eax, byte ptr [ebp-4]  
004120EF  |.  50            push    eax  
004120F0  |.  E8 F6960400   call    0045B7EB                     ;ungetc()  
004120F5  |.  83C4 08       add     esp, 8  
004120F8  |>  8B4D F8       mov     ecx, dword ptr [ebp-8]  
004120FB  |.  C601 00       mov     byte ptr [ecx], 0  
004120FE  |.  C705 2445E600>mov      dword ptr [E64524], 0    ;返回 0  
00412108  |.  A1 2445E600   mov      eax, dword ptr [E64524]  
0041210D  |.  E9 B7000000   jmp      004121C9  
00412112  |>  0FBE55 FC     movsx   edx, byte ptr [ebp-4]            ;default:  
00412116  |.  52            push    edx  
00412117  |.  E8 D8930400   call    0045B4F4                ;isalpha()  
0041211C  |.  83C4 04       add     esp, 4  
0041211F  |.  85C0          test    eax, eax  
00412121  |.  0F84 93000000 je      004121BA  
00412127  |.  C745 F8 2845E>mov      dword ptr [ebp-8], 00E64528  
0041212E  |.  8B45 F8       mov     eax, dword ptr [ebp-8]  
00412131  |.  8A4D FC       mov     cl, byte ptr [ebp-4]  
00412134  |.  8808          mov     byte ptr [eax], cl  
00412136  |.  8B55 F8       mov     edx, dword ptr [ebp-8]  
00412139  |.  83C2 01       add     edx, 1  
0041213C  |.  8955 F8       mov     dword ptr [ebp-8], edx  
0041213F  |>  A1 9045E600   /mov     eax, dword ptr [E64590]  
00412144  |.  50            |push    eax  
00412145  |.  E8 96960400   |call    0045B7E0                ;fgetc()  
0041214A  |.  83C4 04       |add     esp, 4  
0041214D  |.  8845 FC       |mov     byte ptr [ebp-4], al  
00412150  |.  0FBE4D FC     |movsx   ecx, byte ptr [ebp-4]  
00412154  |.  83F9 FF       |cmp     ecx, -1  
00412157  |.  74 36         |je      short 0041218F  
00412159  |.  0FBE55 FC     |movsx   edx, byte ptr [ebp-4]  
0041215D  |.  83FA 2E       |cmp     edx, 2E  
00412160  |.  74 1A         |je      short 0041217C  
00412162  |.  0FBE45 FC     |movsx   eax, byte ptr [ebp-4]  
00412166  |.  83F8 5F       |cmp     eax, 5F  
00412169  |.  74 11         |je      short 0041217C  
0041216B  |.  0FBE4D FC     |movsx   ecx, byte ptr [ebp-4]  
0041216F  |.  51            |push    ecx  
00412170  |.  E8 A2940400   |call    0045B617            ;isalnum()  
00412175  |.  83C4 04       |add     esp, 4  
00412178  |.  85C0          |test    eax, eax  
```



### 汇编代码段

```
0041217A  |.  74 13         |je      short 0041218F  

0041217C  |>  8B55 F8       |mov     edx, dword ptr [ebp-8]  

0041217F  |.  8A45 FC       |mov     al, byte ptr [ebp-4]  

00412182  |.  8802         |mov     byte ptr [edx], al  

00412184  |.  8B4D F8       |mov     ecx, dword ptr [ebp-8]  

00412187  |.  83C1 01       |add     ecx, 1  

0041218A  |.  894D F8       |mov     dword ptr [ebp-8], ecx  

0041218D  |.^ EB B0         \jmp     short 0041213F                 ;判断为 for 循环  

0041218F  |>  8B15 9045E600 mov     edx, dword ptr [E64590]  

00412195  |.  52           push    edx  

00412196  |.  0FBE45 FC     movsx   eax, byte ptr [ebp-4]  

0041219A  |.  50           push    eax  

0041219B  |.  E8 4B960400   call    0045B7EB                   ;ungetc()  

004121A0  |.  83C4 08       add     esp, 8  

004121A3  |.  8B4D F8       mov     ecx, dword ptr [ebp-8]  

004121A6  |.  C601 00       mov     byte ptr [ecx], 0  

004121A9  |.  C705 2445E600>mov     dword ptr [E64524], 0  

004121B3  |.  A1 2445E600   mov     eax, dword ptr [E64524]      ;返回 0  

004121B8  |.  EB 0F         jmp     short 004121C9  

004121BA  |>  C705 2445E600>mov     dword ptr [E64524], 3C   ;返回 3C  

004121C4  |.  A1 2445E600   mov     eax, dword ptr [E64524]  

004121C9  |>  5F           pop     edi  

004121CA  |.  5E           pop     esi  

004121CB  |.  5B           pop     ebx  

004121CC  |.  8BE5         mov     esp, ebp  

004121CE  |.  5D           pop     ebp  

004121CF  \.  C3           retn  
```

### 分析说明

上面的代码很是简单了,前面我已经把重要的说了,我想后面不用说也呢看懂了.

我自己对上面代码逆向为下:

```c
int  GetToken()  

{  

  char    ibuf;  

  *TokenString=NULL;         //这样是和 BYTE TokenString=0 编译后是一样的代码,所以要看整体区分.  

  do{  

    ibuf=fgetc(SMDFile);  

    if((int)ibuf==-1){return 2;}  

    if((int)ibuf==0x2F)  

    {  

      ibuf=fgetc(SMDFile);  

      if((int)ibuf==0x2F)  

      {  

        for(;(int)ibuf!=0x0A&&(int)ibuf!=-1;)  

        {  

            ibuf=fgetc(SMDFile);  

        }  

        if((int)ibuf==-1){return 2;}  

      }  

    }  

  }while(::isspace(ibuf)!=0);  

  BYTE    *pbyte;  

  switch((int)ibuf)  

  {  

  case 0x23:  

    {  

      CurrentToken=0x23;  

      return CurrentToken;  

    }  

  case 0x3B:  

    {  

      CurrentToken=0x3B;  

      return CurrentToken;  

    }  

  case 0x2C:  

    {  

      CurrentToken=0x2C;  

      return CurrentToken;  

    }  

  case 0x7B:  

    {  

      CurrentToken=0x7B;  

      return CurrentToken;  

    }  

  case 0x7D:  

    {  

      CurrentToken=0x7D;  

      return CurrentToken;  

    }  

  case 0x2D:  

  case 0x2E:  

  case 0x30:  

  case 0x31:  

  case 0x32:  

  case 0x33:  

  case 0x34:  

  case 0x35:  

  case 0x36:  

  case 0x37:  

  case 0x38:  

  case 0x39:  

    {  

      DWORD   dwUnk;    //不推荐变量不初始化,而根据汇编逆了以后发现源代码 BUG  

      ::ungetc((int)ibuf,SMDFile);  

      pbyte=(PBYTE)&dwUnk;  

      for(;;)  

      {  

        ibuf=fgetc(SMDFile);  

        if((int)ibuf!=-1){  

          if((int)ibuf!=0x2E){  

            if(::isdigit((int)ibuf)==0&&(int)ibuf!=0x2D){break;}  //这里编译后多个 JMP,想想取掉它,不难.  

          }  

          *pbyte=(BYTE)ibuf;  

          pbyte++;  

          continue;  

        }  

        break;  

      }  

      *pbyte=0;  

      TokenNumber=(float)atof((char*)&dwUnk);  

      CurrentToken=1;  

      return CurrentToken;  

    }  

  case 0x22:  

    {  

      pbyte=(PBYTE)TokenString;   //TokenString 为指针，不要处理成字节，字节和指针很容易分辨，第一次，不注意搞错了．  

      for(;;)  

      {  

        ibuf=fgetc(SMDFile);  

        if((int)ibuf!=-1&&(int)ibuf!=0x22)  

        {  

          *pbyte=(BYTE)ibuf;  

          pbyte++;  

          continue;  

        }  

        break;  

      }  

      if((int)ibuf!=0x22)  

      {  

        ::ungetc((int)ibuf,SMDFile);  

      }  

      *pbyte=0;  

      CurrentToken=0;  

      return CurrentToken;  

    }  

  default:  

    {  

      if(::isalpha(ibuf)!=0)  

      {  

        pbyte=(PBYTE)TokenString;  

        *pbyte=(BYTE)ibuf;  

        pbyte++;  

        for(;;)    

        {  

          ibuf=::fgetc(SMDFile);  

          if((int)ibuf!=-1){  

            if((int)ibuf!=0x2E&&(int)ibuf!=0x5F){  

              if(::isalnum((int)ibuf)==0){break;}  

            }  

            *pbyte=(BYTE)ibuf;  

            pbyte++;  

            continue;  

          }  

          break;  

        }  

        ungetc((int)ibuf,SMDFile);  

        *pbyte=0;  

        CurrentToken=0;  

        return CurrentToken;  

      }  

      CurrentToken=0x3C;  

      return CurrentToken;  

    }  

  }  

  return 0;  

}
```

### 后续验证

大家一边逆一边对对....有个问题让大家去试,当然按我逆的运行是正确的,可原版下面没 `JMP`,而按我的代码编译后就有 `JMP`

有心思的可以试试~~~~~~~~~~ ^\_^ !

`004120C6  |.  74 13         |je      short 004120DB    `

- By EasyStudy For PhantomNet
