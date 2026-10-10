
# 段第三题】也秀秀俺的分析过程

> 原文：[`www.pediy.com/kssd/pediy10/74381.html`](https://www.pediy.com/kssd/pediy10/74381.html)

一个新手的道路！

## 1.分析过程

A) 由于不能通过窗口消息下断，因此只能另辟蹊径。这里我采用的是猜测法，根据从 `IDA` 中得到的 `CDialog` 的初始化函数 `OnInitDialog`，查找相应的调用者，发现继承对话框的初始化函数地址 `420EC0`，接着就在附近的几个函数地址处下断，点击‘Crash’来到函数 `420D20`（`OnCrash`），这里正是需要我们详细分析的函数

B）跟着来到地址 `420D59` 处,完成对函数 `_snwprintf`（`_swprintf_c`）的调用

```
00420D59  |.  DD05 C05D4200     FLD QWORD PTR DS:[425DC0]          ;  3.14159
00420D5F  |.  DD5D F8           FSTP QWORD PTR SS:[EBP-8]
00420D62  |.  DD45 F8           FLD QWORD PTR SS:[EBP-8]
00420D65  |.  DCC0              FADD ST,ST
00420D67  |.  DD5D F8           FSTP QWORD PTR SS:[EBP-8]          ;  6.28318
00420D6A  |.  83EC 08           SUB ESP,8
00420D6D  |.  DD45 F8           FLD QWORD PTR SS:[EBP-8]
00420D70  |.  DD1C24            FSTP QWORD PTR SS:[ESP]            ;  f = 6.28318
00420D73  |.  68 345A4200       PUSH TestFloa.00425A34             ;  "PI * 2 = %f"
00420D78  |.  6A 40             PUSH 40                            ;  0x40
00420D7A  |.  8D95 70FFFFFF     LEA EDX,DWORD PTR SS:[EBP-90]
00420D80  |.  52                PUSH EDX                           ;  buffer address [ebp-90]
00420D81  |.  E8 7CE0FEFF       CALL TestFloa.0040EE02             ;  _snwprintf(buffer address, 0x40, L"PI * 2 = %f", f);
00420D86  |.  83C4 14           ADD ESP,14
```

C）`_snwprintf` 函数处，首先进行必要的参数检查，然后来到函数 `413871`（`_woutput_l`），根据字符串格式进行相应的处理，我们这里是地址 `413E08` 处的函数调用。

```
00413E02  |.  FF35 C8A04200     |PUSH DWORD PTR DS:[42A0C8]
00413E08  |.  E8 3FEFFFFF       |CALL TestFloa.00412D4C            ;  __decode_pointer
00413E0D  |.  59                |POP ECX
00413E0E      FFD0              |CALL EAX                          ;  call decodepointer
```

这里就是异常出现并接着退出的地方，由于调用函数 `_decode_pointer` 时，传入的参数是数据区地址 `42A0C8` 处的值，通过 `IDA` 可以看到该值指向 `__fptrap` 函数地址，因此，通过调用函数 `_decode_pointer` 后，得到的将是函数 `__fptrap` 的地址，显然，接着 `CALL EAX` 调用的正是 `__fptrap` 函数，而该函数的代码正是进行“floating point support not loaded”的错误处理，因此，最后看到该消息的错误提示框，然后程序退出也就不奇怪啦。

根据该错误出现的解释，是因为程序在编译时在程序中没有发现浮点操作，为了提高效率而没有 Load 浮点运行库才会导致这个异常，而该程序在地址 `00420D59` 到 `00420D70` 处明显调用了浮点运算，浮点运算库完全是应该被 Load 的，然而为什么没有被 Load 呢，继续往下分析。

D）根据分析发现，初始化浮点运算操作的函数是 `__cfltcvt_init`（`00411dc6`），该函数将包含几个浮点操作函数指针的数组 `_cfltcvt_tab`（`0042A0B0`）进行初始化。而异常发生处本应该调用的函数 `_cfltcvt_l` 正是其中的第七个函数指针（`PF6`）。顺藤摸瓜，继续向上摸。

调用函数 `__cfltcvt_init` 的是函数 `__fpmath`（`00411E26`），调用函数 `__fpmath` 的是函数 `__cinit`（`0040F618`），调用函数 `__cinit` 的是函数 `__tmainCRTstartup`（`0040EA8F`）。

以上正是浮点操作函数初始化的逆过程。经过详细逐个函数分析，发现在函数 `__cinit` 中，通过函数 `__IsNonwritableInCurrentImage`（`00415D90`）判断地址 `0042405C` 是否不能被写入，如果是不能写入的，就会调用函数 `__pfmath(00411e26)`，而从上面的分析可知，该函数正是用来触发浮点函数初始化的。如果是可以写入的，函数 `__pfmath(00411e26)` 将不被调用，浮点函数也就不会被初始化。通过调试跟踪发现正是这里出现了问题，才导致了最后的程序异常。问题的根源已经找到，具体的修复请看下面。

```
/******************************_cinit*******************************/
0040F618  /$  833D 5C404200 00          CMP DWORD PTR DS:[42405C],0            ;  __cinit
0040F61F  |.  74 1A                     JE SHORT TestFloa.0040F63B
0040F621  |.  68 5C404200               PUSH TestFloa.0042405C
0040F626  |.  E8 65670000               CALL TestFloa.00415D90                  ;  __IsNonwritableInCurrentImage
0040F62B  |.  85C0                      TEST EAX,EAX
0040F62D  |.  59                        POP ECX
0040F62E  |.  74 0B                     JE SHORT TestFloa.0040F63B
0040F630  |.  FF7424 04                 PUSH DWORD PTR SS:[ESP+4]
0040F634  |.  FF15 5C404200             CALL DWORD PTR DS:[42405C]             ;  testfloa.00411e26   __fpmath
0040F63A  |.  59                        POP ECX
0040F63B  |>  E8 AC660000               CALL TestFloa.00415CEC
/******************************_cinit*******************************/

/******************************_cfltcvt_tab*******************************/
typedef void (* PFV)(void);
extern PFV _cfltcvt_tab[10];
#define _CFLTCVT_TAB(i)       (_decode_pointer(_cfltcvt_tab[i]))
typedef void (* PF0)(double*, char*, size_t, int, int, int);
#define _cfltcvt(a,b,c,d,e,f) (*((PF0)_CFLTCVT_TAB(0)))(a,b,c,d,e,f)
typedef void (* PF1)(char*);
#define _cropzeros(a)         (*((PF1)_CFLTCVT_TAB(1)))(a)
typedef void (* PF2)(int, char*, char*);
#define _fassign(a,b,c)       (*((PF2)_CFLTCVT_TAB(2)))(a,b,c)
typedef void (* PF3)(char*);
#define _forcdecpt(a)         (*((PF3)_CFLTCVT_TAB(3)))(a)
typedef int (* PF4)(double*);
#define _positive(a)          (*((PF4)_CFLTCVT_TAB(4)))(a)
typedef void (* PF5)(_LONGDOUBLE*, char*, size_t, int, int, int);
#define _cldcvt(a,b,c,d,e,f)  (*((PF5)_CFLTCVT_TAB(5)))(a,b,c,d,e,f)
typedef void (* PF6)(double*, char*, size_t, int, int, int, _locale_t);
#define _cfltcvt_l(a,b,c,d,e,f,g) (*((PF6)_CFLTCVT_TAB(6)))(a,b,c,d,e,f,g)
typedef void (* PF7)(int, char*, char*, _locale_t);
#define _fassign_l(a,b,c,d)       (*((PF7)_CFLTCVT_TAB(7)))(a,b,c,d)
typedef void (* PF8)(char*, _locale_t);
#define _cropzeros_l(a,b)         (*((PF8)_CFLTCVT_TAB(8)))(a,b)
typedef void (* PF9)(char*, _locale_t);
#define _forcdecpt_l(a,b)         (*((PF9)_CFLTCVT_TAB(9)))(a,b)
/******************************_cfltcvt_tab*******************************/

## 2. 修复方法

通过上面的分析，发现问题的根源是地址 `0042405C` 是可写的。查找其在镜像文件中的偏移位置 `0002405C`，位于 `.rdata` 段，而该段的标志是 `C0000004`，即可读取，可写入，包含已初始化数据。将可写入的属性去掉使该段标志变为 `40000004`，程序将能正常运行。至此关于该程序的异常分析以及如何修复完毕。
