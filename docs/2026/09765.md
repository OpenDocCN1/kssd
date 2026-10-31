
# 《奥数2000》注册码的计算(VB5程序)

(6千字)

## 标 题:《奥数2000》注册码的计算(VB5程序)

发信人:laoqian

软件简介机注册机制：奥数2000未注册时只有“考试介绍”等功能。当您注册后成为注册用户时，下一步窗口的多个子窗口：“奥数课堂”、“招生考试及竞赛”、“奥赛2001”等可供使用。它们包括初、中、高级奥数训练题及重点初中招生考试试题, 试题包括图形、计算等多种形式试题，题库将有两千多道精选试题供您使用。VB5.0程序，注册时程序根据硬盘序列号不同产生软件的序列号，因此每台机器每次产生的的序列号是不同的，我们的目标就是分析其注册码的产生过程，作注册机。

使用工具：`SmartCheck 6.20`

## 注册计算过程：

首先安装《奥数2000》，运行`SmartCheck`，配置好`SmartCheck`后，load奥数2000，进入注册窗口，显示“本机硬件代码”：`010061134`（这是我的机，你的肯定不一样）在“本软件作者提供的注册码”填入：`123456789`（注册码当然是假的，只是为了好记，必须9位，有提示） 再按注册，这时跳出一个对话框.....，退出《奥数2000》。然后，再次load奥数2000，直到见到注册提示窗口。

切换到`NuMega SmartCheck`的主窗口。 软件运行过程可以在`SmartCheck`看到，

展开`Image1_Click`。（就是前面带＋的Click）

我们来到  “`Form11 (Form) created`”，往下看：

`Text4.Text` <-- "`010`" (String)

String "`010`"

Integer

`(1)` --> Long `(1)`

Integer `1`

Long `1`

......来到

Integer `(6)`

--> Long `(6)`

Integer `6`

Long `6`

`Text4.Text` <-- "`010061134`" (String)

String "`010061134`"

<======这是我的“本机硬件代码”，往下看：

这时需要展开详细信息，你不会？我X

`Mid(VARIANT:String:"010061134..", long:4, VARIANT:Integer:6)`

Arguments

--------------------

string (variant)

    String

.bstrVal = `00615654`

           = "`010061134`"

Long  length = `4` 0x00000004      <======从第4位开始

start

(variant)

    Integer  .iVal = `6` 0x0006

<======取6个数字

`__vbaVarMove(VARIANT:String:"061134", VARIANT:Empty)`

returns DWORD:`83F784`

Arguments

--------------------

pSource

(variant)

    unsigned short \* .bstrVal = `0061560C`

       = "`061134`"        <======取6个数字得到

pDest (variant)

    unsigned short .vt = `0` 0x0000

<====== "`061134`"为计算注册码的关键数

............

再往下看：

`Text2.Text`

`Mid(VARIANT:String:"loca=123...", long:6, VARIANT:Integer:6)`

Arguments

--------------------

string (variant)

    String

.bstrVal = `007102C4`

          = "`loca=123456789`

    <======这是我的假“注册码”，往下看：

"

Long  length = `6` 0x00000006

start (variant)

    Integer  .iVal = `6` 0x0006

`__vbaVarMove(VARIANT:String:"123456", VARIANT:Empty)` returns DWORD:`83F81C`

Arguments

--------------------

pSource (variant)

    unsigned short \* .bstrVal = `00615654`

          = "`123456`"

    <======取前6个数字

pDest (variant)

    unsigned

short .vt = `0` 0x0000

<====== "`123456`"以后会与"`061134`"计算出的注册码比较

....................

`__vbaVarMul(VARIANT:String:"061134", VARIANT:Integer:18)`

<======乘法

Arguments

--------------------

lhs (variant)

    unsigned short \* .bstrVal = `0061560C`

   = "`061134`"

rhs (variant)

    signed short .iVal

= `18` 0x0012    <======`061134`*`18`

`__vbaVarMul` returns DWORD:`83F734`

`__vbaVarAdd(VARIANT:Double:1.10041e+006, VARIANT:Long:2498699)` returns DWORD:`83F724`

<======加法

Arguments

--------------------

lhs (variant)

    double .dblVal = `1100412.`      <====== `061134`*`18`= `1100412`

rhs (variant)

    signed long .lVal = `2498699` 0x0026208B

<====== `1100412+2498699`

`__vbaVarMove(VARIANT:Double:3.59911e+006, VARIANT:Empty)` returns DWORD:`83F774`

Arguments

--------------------

pSource (variant)

    double .dblVal = `3599111.`  <======

`1100412+2498699=3599111`

pDest (variant)

    unsigned short

.vt = `0` 0x0000

...............

`Mid(VARIANT:Double:3.59911e+006, long:2, VARIANT:Integer:6)`

Arguments

--------------------

string (variant)

    double .dblVal = `3599111.`

Long  length = `2` 0x00000002

    <======从第2位开始

start (variant)

    Integer

.iVal = `6` 0x0006    <======取6个数字

Mid

`__vbaVarMove(VARIANT:String:"599111", VARIANT:Empty)` returns DWORD:`83F7CC`

Arguments

--------------------

pSource (variant)

    unsigned short \* .bstrVal = `006156E4`

          = "`599111`"

  <======取6个数字得到我鸡的关键数

pDest (variant)

    unsigned

short .vt = `0` 0x0000

..............

再往下看：怎样对待"`599111`",找到以下mid

`Mid(VARIANT:String:"599111", long:1, VARIANT:Integer:1)`

Arguments

--------------------

string (variant)

    String

.bstrVal = `006156E4`

          = "`599111`"

Long

length = `1` 0x00000001        <======取"`599111`"的第1位数

start (variant)

    Integer  .iVal = `1` 0x0001

`Mid(VARIANT:String:"599111", long:3, VARIANT:Integer:1)`

Arguments

--------------------

string (variant)

    String  .bstrVal

= `006156E4`

          = "`599111`"

Long

length = `3` 0x00000003        <======取"`599111`"的第3位数

start (variant)

    Integer  .iVal = `1` 0x0001

`Mid(VARIANT:String:"599111", long:5, VARIANT:Integer:1)`

Arguments

--------------------

string (variant)

    String  .bstrVal = `006156E4`

       = "`599111`"

Long  length = `5` 0x00000005

       <======取"`599111`"的第5位数

start (variant)

   Integer  .iVal = `1` 0x0001

`Mid(VARIANT:String:"599111", long:2, VARIANT:Integer:1)`

Arguments

--------------------

string (variant)

    String  .bstrVal = `006156E4`

       = "`599111`"

Long  length = `2` 0x00000002

       <======取"`599111`"的第2位数

start (variant)

   Integer  .iVal = `1` 0x0001

`Mid(VARIANT:String:"599111", long:4, VARIANT:Integer:1)`

Arguments

--------------------

string (variant)

    String  .bstrVal = `006156E4`

       = "`599111`"

Long  length = `4` 0x00000004

         <======取"`599111`"的第4位数

start (variant)

    Integer  .iVal = `1` 0x0001

`Mid(VARIANT:String:"599111", long:6, VARIANT:Integer:1)`

Arguments

--------------------

string (variant)

    String  .bstrVal = `006156E4`

       = "`599111`"

Long  length = `6` 0x00000006

           <======取"`599111`"的第6位数

start

(variant)

    Integer  .iVal = `1` 0x0001

.......

`__vbaVarMove(VARIANT:String:"591911", VARIANT:Empty)` returns DWORD:`83F7BC`

Arguments

--------------------

pSource (variant)

 unsigned short \* .bstrVal = `00615794`

= "`591911`"          <======由上面得到我鸡的关键数

pDest

(variant)

    unsigned short .vt = `0` 0x0000

............

再往下看：怎样对待"`591911`",找到以下：

`__vbaVarTstEq(VARIANT:String:"591911", VARIANT:String:"123456")` returns DWORD:`0`

Arguments

--------------------

lhs (variant)

    unsigned short \* .bstrVal = `00615794`

         = "`591911`"

rhs (variant)

unsigned short \* .bstrVal = `00615654`

          =

"`123456`"

<====== "`591911`"与"`123456`"比较是否相等，相等即注册成功！！！

看明白了吗，我的注册码为9位数"`591911***`"，其中`*`为任意字符

至于注册机，我就不写了。

## 注册机算法：

“本机硬件代码”为`A`（例如，`010061134`），从`A`第4位开始取6个数字得到`B`（`061134`），`C=B*18+2498699`

从`C`第2位开始取6个数字得到`D`（六位数），依次取`D`的1，3，5，2，4，6位，组成新数`E`,再在`E`的末尾加3个任意字符，得到你机的9位数注册码。
