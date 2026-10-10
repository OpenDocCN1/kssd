

# 程序逆向技术之二变量和常量

> 原文：[`www.pediy.com/kssd/pediy10/59933.html`](https://www.pediy.com/kssd/pediy10/59933.html)

![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷") 今天来了，看到很热闹，我也热闹一下，大家春节过的好不好？？，实际，今天带给大家的比较麻烦，不容易  
  
理解，比起以往的来讲都要难点了，但是，相对很容易了，我是自己一点点反复的试中学习过来的，今天发出来给喜欢  
  
逆向的朋友，我觉得逆向比我想的要复杂，而切是件耐心的磨性子的事，强烈建议没耐心的就别折腾了。反正我是领教  
  
了。  
  在看这篇时大家需要有心理准备，就是汇编与 C++的关系，要对 C++和汇编都要很熟悉才可以，不然，很难理解  
  
的，还有就是 IDA 和 OD 是不可少的，最好有 WinHex 和 UE，我们要在 IDA 和 OD 中间寻找我们的答案。所以，在学习的时候大  
  
家可以选择一些简单的程序，最好不大的，几十 KB 到几 KB 的程序，方便逆向。现在逆向的人不多，我想想学的人不会也  
  
不多的。只是苦于没有这类书籍，记得有本什么黑客反汇编的书，是个老外写的还不错，大家有兴趣可以找找。  
  我这里说的不一定都是准确的，我只是将我的测试结果发出来，高手们要指点一二哦！我是个超级菜鸟，呵呵  
  
\~~~，希望别惹大家笑话，喜欢的就学习了，不喜欢的就看个乐子吧！  
  
  注意：在逆向的程序选择时，新手最好选择基本的 C 代码的程序，不建议选择 pascal 和 basic 的代码，这些代码  
  
不容易理解，而且和汇编的关系不是很明确的。逆向的时候选择 C 的代码，我发现很容易和汇编联系的。但是要将汇编反  
  
过来写成 C 语言那就又是麻烦了。:(! 这里不建议选择 MFC 类程序，只建议大家选择 Win32 程序，这样我们的难度就小了很  
  
多了，还有个就是不建议选择类程序，初学的时候就最好是函数、变量、指针。等学的差不多了，再加进去类和结构等  
  
等。  
  
//------------------------------------------------------------  
    第一部分  常量、变量  
//------------------------------------------------------------  
  
  在逆向的时候我们可以自己写点程序，来研究的，但是我推荐 Visual C++ 6.0 ，为什么是 6.0 呢？因为 2003 以  
  
后程序被编译以后不再像 6.0 那样的清晰，编译器给程序加了很多垃圾进去，不太好分析。在编译的时候使用 Release，  
  
这样代码量会小，花的工夫就相对来说少很多。  
  我们把下面的代码编译进程序，看起来很简单的。但是结果是不一样的，所有教 C 语言的老师都说下面的代码结  
  
果是一样的，但是，我发现在汇编里就不一定的哦！！很奇怪吧，大家可以自己多试试，我这里所说的都是常规状态下  
  
的。  
  
第一、局部变量：  
  
//-------------------------------------------  
  int  intbuf1=1,intbuf2=2;  
  
  int  intbuf3=3;  
  int  intbuf4=4;  
  
//-------------------------------------------  
  
OD 反汇编以后的结果和下面的差不多的，有朋友会问了，为什么我的就没有下面的代码，只有一个 retn 好像什么都没做   
  
哦，在你的变量没做任何事时，只对其进行初试化，那么编译器在 Release 模式下，是不被解释的。我们在设置的 C/C++  
  
选项中，将 Maximize Speed 改为 Default,这个时候我们就能看到了汇编代码。  
  
//-------------------------------------------  
0040135F  /$  55            push    ebp  
00401360  |.  8BEC          mov     ebp, esp  
00401362  |.  83EC 10       sub     esp, 10  
00401365  |.  C745 F8 01000>mov     dword ptr \[ebp-8\], 1  
0040136C  |.  C745 FC 02000>mov     dword ptr \[ebp-4\], 2  
00401373  |.  C745 F0 03000>mov     dword ptr \[ebp-10\], 3  
0040137A  |.  C745 F4 04000>mov     dword ptr \[ebp-C\], 4  
00401381  |.  8BE5          mov     esp, ebp  
00401383  |.  5D            pop     ebp  
00401384  \\.  C3            retn  
  
//-------------------------------------------  
  
我们不难分析出栈顺序 intbuf2-intbuf1-intbuf4-intbuf3，我不说了，大家自己思考编译器是如果解释变量的，而我们  
  
需要注意的是 dword ptr \[ebp-X\]，我们把例子改一下：  
  
//-------------------------------------------  
  BYTE  intbuf1=1;  
  char  intbuf2=2;  
  
  int    intbuf3=3;  
  DWORD  intbuf4=4;  
  LONG   intbuf5=5;  
  SHORT  intbuf6=6;  
  WORD   intbuf7=7;  
//-------------------------------------------  
  
不难从下面的代码中我们可以看出，char 和 BYTE 都对应 byte ptr,SHORT 和 WORD 都对应 word ptr，LONG 和 DWORD 都对应  
  
dword ptr，我们就很容易分辨出源代码的类型了，简单吧，现在看到如下的汇编代码就容易理解和明白了，我想谁都可  
  
以把它写成 C 代码或者其他语言的代码了。  
//-------------------------------------------  
0040135F  /$  55            push    ebp  
00401360  |.  8BEC          mov     ebp, esp  
00401362  |.  83EC 1C       sub     esp, 1C  
00401365  |.  C645 F8 01    mov     byte ptr \[ebp-8\], 1  
00401369  |.  C645 FC 02    mov     byte ptr \[ebp-4\], 2  
0040136D  |.  C745 F0 03000>mov     dword ptr \[ebp-10\], 3  
00401374  |.  C745 F4 04000>mov     dword ptr \[ebp-C\], 4  
0040137B  |.  C745 E8 05000>mov     dword ptr \[ebp-18\], 5  
00401382  |.  66:C745 EC 06>mov     word ptr \[ebp-14\], 6  
00401388  |.  66:C745 E4 07>mov     word ptr \[ebp-1C\], 7  
0040138E  |.  8BE5          mov     esp, ebp  
00401390  |.  5D            pop     ebp  
00401391  \\.  C3            retn  
  
//-------------------------------------------  
  
前面我们说了，定义地方不同在汇编中也是不同的，还有变量的类型我们先来看这样一段汇编代码，大家可以猜写出它  
  
的 C 语言代码^\_^  
  
//--------------------------------------------  
0040135F  /$  55            push    ebp  
00401360  |.  8BEC          mov     ebp, esp  
00401362  |.  83EC 0C       sub     esp, 0C  
00401365  |.  C745 F8 01000>mov     dword ptr \[ebp-8\], 1  
0040136C  |.  8B45 F8       mov     eax, dword ptr \[ebp-8\]  
0040136F  |.  8945 FC       mov     dword ptr \[ebp-4\], eax  
00401372  |.  8B4D FC       mov     ecx, dword ptr \[ebp-4\]  
00401375  |.  034D F8       add     ecx, dword ptr \[ebp-8\]  
00401378  |.  894D F8       mov     dword ptr \[ebp-8\], ecx  
0040137B  |.  8B55 F8       mov     edx, dword ptr \[ebp-8\]  
0040137E  |.  0355 FC       add     edx, dword ptr \[ebp-4\]  
00401381  |.  8955 F4       mov     dword ptr \[ebp-C\], edx  
00401384  |.  8BE5          mov     esp, ebp  
00401386  |.  5D            pop     ebp  
00401387  \\.  C3            retn  
  
//--------------------------------------------  
  
看了会不会头大了，呵呵~~~，是不是有点不同了，看看我怎么逆吧~~  
  
//--------------------------------------------  
DWORD  unk1=1; //mov     dword ptr \[ebp-8\], 1  
  
DWORD  unk2=unk1;  
//mov     eax, dword ptr \[ebp-8\]  
//mov     dword ptr \[ebp-4\], eax  
//我们发现在第 2 句里多了个 ebp-4 所以我们就能确定了，有新的变量出现了，而且是 DWORD 类型的，EAX 把变量 unk1 的  
  
内//容传递到了新的变量中。  
  
unk1=unk2+unk1;  
//mov     ecx, dword ptr \[ebp-4\]  
//add     ecx, dword ptr \[ebp-8\]  
//mov     dword ptr \[ebp-8\], ecx  
//我们知道先取出 unk2 的值放入 ecx,我们可以看到在相加后又放回了 unk1;  
  
unk3=unk1+unk2;  
//mov     edx, dword ptr \[ebp-8\]  
//add     edx, dword ptr \[ebp-4\]  
//mov     dword ptr \[ebp-C\], edx  
//我想我不用解释大家一看就明白了吧~~~~  
  
//----------------------------------------------------------  
源代码如下：  
  int  intbuf1=1;  
  int  intbuf2=intbuf1;  
  intbuf1=intbuf2+intbuf1;  
  int  intbuf3=intbuf1+intbuf2;  
//----------------------------------------------------------  
  是不是一句都没错哦~~，我是逆了好多了，所以一看就知道了，大家在练习的时候多试试，就会发现规律的，  
  
不是老师们说的，代码一但编译就不可能复原了，那是不准确的，可以做到几乎接近。  
  
第二、全局变量：  
  代码为下面：  
//----------------------------------------------------------  
  BYTE  intbuf1=0;  
  char  intbuf2=0;  
  
  int    intbuf3=0;  
  DWORD  intbuf4=0;  
  LONG   intbuf5=0;  
  SHORT  intbuf6=0;  
  WORD   intbuf7=0;  
  
void InitMain()  
{  
  intbuf1=1;  
  intbuf2=2;  
  
  intbuf3=3;  
  intbuf4=4;  
  intbuf5=5;  
  intbuf6=6;  
  intbuf7=7;  
}  
//----------------------------------------------------------  
  
汇编如下：  
//----------------------------------------------------------  
0040135F  /$  55            push    ebp  
00401360  |.  8BEC          mov     ebp, esp  
00401362  |.  C605 74854000>mov     byte ptr \[408574\], 1  
00401369  |.  C605 75854000>mov     byte ptr \[408575\], 2  
00401370  |.  C705 78854000>mov     dword ptr \[408578\], 3  
0040137A  |.  C705 7C854000>mov     dword ptr \[40857C\], 4  
00401384  |.  C705 80854000>mov     dword ptr \[408580\], 5  
0040138E  |.  66:C705 84854>mov     word ptr \[408584\], 6  
00401397  |.  66:C705 86854>mov     word ptr \[408586\], 7  
004013A0  |.  5D            pop     ebp  
004013A1  \\.  C3            retn  
  
//------------------------------------------------------------  
^\_^,大家在逆向的时候会发现了把，全局变量和局部变量的区别了吧~~~，我想大家一看就明白了，局部变量是放在栈中  
  
的，而全局变量是不放的，还有就是其它和局部变量都一样的，但是我们需要注意，指针问题，有的时候指针和变量是  
  
一样的，不太好分辨，以后我会再给大家解释。  
  
第三、常量  
  代码如下：  
//------------------------------------------------------------  
#define  test2 2  
#define  test3 "123456789"  
  
void InitMain()  
{  
  int mm=test2;  
  char \*pcc=test3;  
}  
//需要说明的是，我们用#define 做的宏函数在编译的时候，是被直接编译出结果的，根据宏函数来确定是镶嵌代码还是  
  
//直接编译结果。  
//-------------------------------------------------------------  
  
反汇编后，发现变量和指针是基本一样的，没什么明显的区别，是变量还是指针，大家多练习就是了，不是简单的说就  
  
是什么就什么，实际指针在汇编中是以变量的形式来处理的，我们在汇编中就把指针当变量，但是在逆回到 C 语言中时我  
  
们必须对其进行区分的。可以很明确的看出常量在汇编中什么怎么样处理的，我们把它逆回去不是很简单的事了吗？  
//-------------------------------------------------------------  
0040135F  /$  55            push    ebp  
00401360  |.  8BEC          mov     ebp, esp  
00401362  |.  83EC 08       sub     esp, 8  
00401365  |.  C745 FC 02000>mov     dword ptr \[ebp-4\], 2  
0040136C  |.  C745 F8 30604>mov     dword ptr \[ebp-8\], 00406030      ;  ASCII "123456789"  
00401373  |.  8BE5          mov     esp, ebp  
00401375  |.  5D            pop     ebp  
00401376  \\.  C3            retn  
//----------------------------------------------------------------  
  
逆向是件不容易的事，我已经给大家很简单的示范了，想想实际没什么的，是没什么，简单，但是真的做起来就不容易  
  
了，大家可以再多试试，就会发现，代码和汇编之间是有本质的联系的。我们可以逆出很接近原代码的代码来。  
  
第四、实践练手  
  
  只说不练，被说成那个什么假什么把势的，好，我们就来练习一下。我想大家要是把这段代码看明白了，一定  
  
会猜到我下次要给大家带来什么了，郁闷不？？说的很简单，练起手来还蛮难^\_^。  
  
//----------------------------------------------------------------  
0040128D  /$  55            push    ebp  
0040128E  |.  8BEC          mov     ebp, esp  
00401290  |.  81EC 0C010000 sub     esp, 10C  
00401296  |.  57            push    edi  
00401297  |.  8D85 F8FEFFFF lea     eax, dword ptr \[ebp-108\]  
0040129D  |.  50            push    eax  
0040129E  |.  E8 860E0000   call    00402129  ;time()  
004012A3  |.  83C4 04       add     esp, 4  
004012A6  |.  8D8D F8FEFFFF lea     ecx, dword ptr \[ebp-108\]  
004012AC  |.  51            push    ecx  
004012AD  |.  E8 170D0000   call    00401FC9  ;localtime()  
004012B2  |.  83C4 04       add     esp, 4  
004012B5  |.  8985 F4FEFFFF mov     dword ptr \[ebp-10C\], eax  
004012BB  |.  8B95 F4FEFFFF mov     edx, dword ptr \[ebp-10C\]  
004012C1  |.  8B42 14       mov     eax, dword ptr \[edx+14\]  
004012C4  |.  05 6C070000   add     eax, 76C  
004012C9  |.  A3 38E64000   mov     dword ptr \[40E638\], eax  
004012CE  |.  8B8D F4FEFFFF mov     ecx, dword ptr \[ebp-10C\]  
004012D4  |.  8B51 10       mov     edx, dword ptr \[ecx+10\]  
004012D7  |.  83C2 01       add     edx, 1  
004012DA  |.  8915 3CE64000 mov     dword ptr \[40E63C\], edx  
004012E0  |.  8B85 F4FEFFFF mov     eax, dword ptr \[ebp-10C\]  
004012E6  |.  8B48 0C       mov     ecx, dword ptr \[eax+C\]  
004012E9  |.  890D 40E64000 mov     dword ptr \[40E640\], ecx  
004012EF  |.  C685 FCFEFFFF>mov     byte ptr \[ebp-104\], 0  
004012F6  |.  B9 40000000   mov     ecx, 40  
004012FB  |.  33C0          xor     eax, eax  
004012FD  |.  8DBD FDFEFFFF lea     edi, dword ptr \[ebp-103\]  
00401303  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
00401305  |.  66:AB         stos    word ptr es:\[edi\]  
00401307  |.  AA            stos    byte ptr es:\[edi\]  
00401308  |.  68 60E64000   push    0040E660                         ; /<%s> = ""  
0040130D  |.  8B15 40E64000 mov     edx, dword ptr \[40E640\]          ; |  
00401313  |.  52            push    edx                              ; |<%02d> => 0  
00401314  |.  A1 3CE64000   mov     eax, dword ptr \[40E63C\]          ; |  
00401319  |.  50            push    eax                              ; |<%02d> => 0  
0040131A  |.  8B0D 38E64000 mov     ecx, dword ptr \[40E638\]          ; |  
00401320  |.  51            push    ecx                              ; |<%02d> => 0  
00401321  |.  68 E8B04000   push    0040B0E8                         ; |Format = "Log\\%02d%02d%02d\_%  
  
s.log"  
00401326  |.  8D95 FCFEFFFF lea     edx, dword ptr \[ebp-104\]         ; |  
0040132C  |.  52            push    edx                              ; |s  
0040132D  |.  FF15 28A14000 call    dword ptr \[<&USER32.wsprintfA>\]  ; \\wsprintfA  
00401333  |.  83C4 18       add     esp, 18  
00401336  |.  833D A0E64000>cmp     dword ptr \[40E6A0\], 0  
0040133D  |.  74 18         je      short 00401357  
0040133F  |.  A1 A0E64000   mov     eax, dword ptr \[40E6A0\]  
00401344  |.  50            push    eax  
00401345  |.  E8 290C0000   call    00401F73                         ;fclose()  
0040134A  |.  83C4 04       add     esp, 4  
0040134D  |.  C705 A0E64000>mov     dword ptr \[40E6A0\], 0  
00401357  |>  68 00B14000   push    0040B100                         ;  ASCII "a+t"  
0040135C  |.  8D8D FCFEFFFF lea     ecx, dword ptr \[ebp-104\]  
00401362  |.  51            push    ecx  
00401363  |.  E8 F80B0000   call    00401F60                         ;fopen()  
00401368  |.  83C4 08       add     esp, 8  
0040136B  |.  A3 A0E64000   mov     dword ptr \[40E6A0\], eax  
00401370  |.  833D A0E64000>cmp     dword ptr \[40E6A0\], 0  
00401377  |.  75 0E         jnz     short 00401387  
00401379  |.  68 04B14000   push    0040B104                         ;  ASCII "Log create file error!"  
0040137E  |.  FF15 ACE74000 call    XXXXXXXX        ;这里就不用逆了，这段代码是从我逆的代码中截下来的。  
00401384  |.  83C4 04       add     esp, 4  
00401387  |>  8D95 FCFEFFFF lea     edx, dword ptr \[ebp-104\]  
0040138D  |.  52            push    edx  
0040138E  |.  68 A4E64000   push    0040E6A4  
00401393  |.  E8 68090000   call    00401D00                        ;strcpy()  
00401398  |.  83C4 08       add     esp, 8  
0040139B  |.  5F            pop     edi  
0040139C  |.  8BE5          mov     esp, ebp  
0040139E  |.  5D            pop     ebp  
0040139F  \\.  C3            retn  
  
//----------------------------------------------------------------  
  我也把我的逆的结果发出来，大家看看，实际这个需要耐心的练习的，实际有几个 CALL 大家没必要逆，我这里  
  
就不贴出来了，实际有很多 C 函数是被解释成汇编，一块编译进程序的，我会在以后的内容中来介绍如何处理。  
我已经把函数给贴出来了，我想大家逆的时候就不郁闷了吧~，逆这段代码，我们的方法有很多，我这里就贴两种  
我最常用的方法：  
  
1、汇编的逆法  
//-----------------------------------------------------------------  
  
int year =0;  
int month=0;  
int day  =0;  
DWORD  file=NULL;  
char  \*type="a+t";  
char  \*str="test";  
char  \*format="Log\\\\%02d%02d%02d\_%s.log";  
TCHAR  buf\[MAX\_PATH\]={0};  
//------------------------------------------------------------------  
void \_callunk()  
{  
  \_asm{  
  push    ebp  
  mov     ebp, esp  
  sub     esp, 0x10C  
  push    edi  
  lea     eax, dword ptr \[ebp-0x108\]  
  push    eax  
  call    time  
  add     esp, 4  
  lea     ecx, dword ptr \[ebp-0x108\]  
  push    ecx  
  call    localtime  
  add     esp, 4  
  mov     dword ptr \[ebp-0x10C\], eax  
  mov     edx, dword ptr \[ebp-0x10C\]  
  mov     eax, dword ptr \[edx+0x14\]  
  add     eax, 0x76C  
  mov     year, eax  
  mov     ecx, dword ptr \[ebp-0x10C\]  
  mov     edx, dword ptr \[ecx+0x10\]  
  add     edx, 1  
  mov     month, edx  
  mov     eax, dword ptr \[ebp-0x10C\]  
  mov     ecx, dword ptr \[eax+0x0C\]  
  mov     day, ecx  
  mov     byte ptr \[ebp-0x104\], 0  
  mov     ecx, 0x40  
  xor     eax, eax  
  lea     edi, dword ptr \[ebp-0x103\]  
  rep     stos dword ptr es:\[edi\]  
  stos    word ptr es:\[edi\]  
  stos    byte ptr es:\[edi\]  
  push    str  
  mov     edx, day  
  push    edx  
  mov     eax, month  
  push    eax  
  mov     ecx, year  
  push    ecx  
  push    format  
  lea     edx, dword ptr \[ebp-0x104\]  
  push    edx  
  call    wsprintfA  
  add     esp, 0x18  
  cmp     file, 0  
  je L050  
  mov     eax, file  
  push    eax  
  call    fclose  
  add     esp, 4  
  mov     file, 0  
L050:  
  push    type  
  lea     ecx, dword ptr \[ebp-0x104\]  
  push    ecx  
  call    fopen  
  add     esp, 8  
  mov     file, eax  
  cmp     file, 0  
  jnz L061  
  ;push    0040B104  
  ;call    XXXXXXXX  
  ;add     esp, 4  
L061:  
  lea     edx, dword ptr \[ebp-104\]  
  push    edx  
  push    buf  
  call    strcpy  
  add     esp, 8  
  pop     edi  
  mov     esp, ebp  
  pop     ebp  
  retn  
}  
}  
//---------------------------------------------------------------  
2、直接逆成 C 语句  
  
//---------------------------------------------------------------  
void  LogDataSet()  
{  
  time\_t mtm;  
  time(&mtm);  
  tm \*ptm=::localtime(&mtm);  
  
  //按逆的原则来说，应该是全局变量。  
  int LogMYear=ptm->tm\_year+1900;  
  int LogMonth=ptm->tm\_mon+1;  
  int LogMDay =ptm->tm\_mday;  
  
  TCHAR  str\[MAX\_PATH\]={0};  
  
  wsprintf(str,"Log\\\\%02d%02d%02d\_%  
  
s.log",\_logdata.LogMYear,\_logdata.LogMonth,\_logdata.LogMDay,szServerName);  
  
  if(logfp!=NULL){fclose(logfp); logfp=NULL;}  
  
  logfp=fopen(str,"a+t");  
  
  if(logfp==NULL)  
  {  
    //这里是那个 CALL  
    //LogAdd("Log create file error!");  
  }  
  
  strcpy(m\_szLogFileName,str);  
}  
//---------------------------------------------------------------  
  
  以上只是参考性的代码，我没拿编译器试，直接按汇编把它写了出来，大家有兴趣的可以试的。今天说到这里  
  
大家觉得怎么样呢~~~，逆向这个东西，很多的。。一时半时，还真难给大家很明确的说明，希望高手多指点。  
  
                                                            -by easystudy