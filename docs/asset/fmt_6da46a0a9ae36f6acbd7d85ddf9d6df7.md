
# xyz 计算机等级考试系统(二级 c)

> 原文：[`www.pediy.com/kssd/pediy06/pediy6284.htm`](https://www.pediy.com/kssd/pediy06/pediy6284.htm)

一个 vb 软件的破解(新手请进)

软件名称:xyz 计算机等级考试系统(二级 c)   
vb 编写   

难度系数: 容易   

破解声明:  只为技术而破解！这个软件较为简单，主要介绍一下 vb 程序破解的基本知识   

作者: `mejy`   

工具: `smartCheck`  ,`OD1.09` 汉化版   ，`fi`   

## 第一 `smartcheck` 的破解

首先我们看一下用 `smartCheck` 来对他进行破解，为什么呢？因为对 vb 的软件用 smart 分析一下有利于动态破解的思路?

可照看学第二版 238 页的方法进行。。。。。。先用 smart 载入 `xyz`,点击 `run`,然后在注册筐里输点什么？六位（分析后可知注册码为 6 位）

试练码 `abcdef`   我得序列号 `55555B`

来到这：

```
__vbaStrVarMove(VARIANT:String:"ABCDEFGH...") returns DWORD:13406C

Arguments

\--------------------

string (variant)

    unsigned short * .bstrVal = 0013406C

          = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz123456789"

经过上面的计算得到了一劝字符串！记下他的地址，后面的计算注册码就是根据这个字符表得来得

WideCharToMultiByte(unsigned int:00000000, FLAGS:00000000, LPWSTR:0E2D1F44, int:1, PTR:0012EDAA, DWORD:00000002, LPSTR:00000000, PTR:00000000) returns INT:1

Arguments

\--------------------

unsigned long CodePage = 0 0x00000000

unsigned long dwFlags = 0 0x00000000

unsigned short * lpWideCharStr = 0E2D1F44

      = "5"

signed long cchWideChar = 1 0x00000001

signed char * lpMultiByteStr = 0012EDAA

      = "-

signed long cchMultiByte = 2 0x00000002

signed char * lpDefaultChar = NULL

signed long * lpUsedDefaultChar = NULL

这一函数是将这个字符转化为十进制 53

计算他在上面的表中对应的位置 然后

从内存中 1340d8 处取得字符 3               正确注册码的第一位

一下与输入的假码的第一位进行比较

__vbaVarTstEq(VARIANT:String:"3", VARIANT:String:"a") returns DWORD:0

Arguments

\--------------------

lhs (variant)

    unsigned short * .bstrVal = 001F6E94

          = "3"

rhs (variant)

    unsigned short * .bstrVal = 001B5DA4

          = "a"

SysAllocStringByteLen(char *:001340E4, DWORD:00000002) returns LPVOID:E2D1F44

经过计算取得注册码的第二位（他的位置与基址相差 72）

Arguments

\--------------------

signed char * psz = 001340E4

      = "9"

unsigned long len = 2 0x00000002

SysAllocStringByteLen(char *:001F6E6E, DWORD:00000002) returns LPVOID:1B5DA4

取得假码的第二位

Arguments

\--------------------

signed char * psz = 001F6E6E

      = "b"

unsigned long len = 2 0x00000002

__vbaVarTstEq(VARIANT:String:"9", VARIANT:String:"b") returns DWORD:0

Arguments

\--------------------

lhs (variant)

    unsigned short * .bstrVal = 0E2D1F44

          = "9"

rhs (variant)

    unsigned short * .bstrVal = 001B5DA4

          = "b"

进行比较

后面各位的比较方法类似这儿就不一一列出了

这儿我们能得出他的注册码，但是要明白他的算法
```

## 第二部分 `od` 的动态分析

我们来动态调试一下，用 `od` 载入程序

载入后，先别急着运行，分析一下先，因为经过上面我们知道它采用的是变量比较的方法（见看学第二办 231 页）  断点函数为 `vbavarTstEq`;   点击右键  搜索――》当前模块中的名称――》找到 `vbavarTstEq`――》右键，查找参考，看见调用的地方了美，在这可以设断点了，（你也可以在运行之后再设，应为这个程序启动是也进行了注册码的比较，会妨碍你的分析）。运行 `f9`

程序运行有异常，`shift+f9` 继续 (前面没设断点的话这就应该设了，如果设了的话 `f9` 键 6 次

每次你可分析仪下，程序其实就是进行 6 次比较)

用 `d edx+8` 来看看内存中有什么 －－－－－172cdc 中正确地注册码 33

用 `d eax+8` 因为刚启动什么也没有，下面我们继续

直到注册矿出现

然后输入假码 `abcdef` 点注册，程序很快断在 `0045FF4E`   . `FF15 F0104000`  `CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarTs>`;  `MSVBVM60.__vbaVarTstEq`

再来看看 `d eax+8` 中的地址  18ce24----61  a 的 ascii 码

         `d edx+8` 处的地址 18a97c －－－－33   3 的 ascii 码

继续！下面进入分析过程

```
0045FDC5   > 66:3BC8        CMP CX,AX   //比较看看注册码的长度 6 结束否

0045FDC8   . 0F8F CD010000  JG XYZ_c271.0045FF9B 如果比较结束跳出循环

0045FDCE   . 0FBFD9         MOVSX EBX,CX      将 i 赋值给 ebx  第一次 i=1,第二次 i=2,第三次 i＝3，第四次 i＝4。。。。。

0045FDD1   . 8D55 D8  LEA EDX,DWORD PTR SS:[EBP-28]   ebp-28 中保存的是序列号

0045FDD4   . 8D45 BC        LEA EAX,DWORD PTR SS:[EBP-44]

0045FDD7   . 8995 74FFFFFF  MOV DWORD PTR SS:[EBP-8C],EDX

0045FDDD   . 50             PUSH EAX

0045FDDE   . 8D8D 6CFFFFFF  LEA ECX,DWORD PTR SS:[EBP-94]  将 4008－－ecx 中

0045FDE4   . 53               PUSH EBX

0045FDE5   . 8D55 AC        LEA EDX,DWORD PTR SS:[EBP-54]

0045FDE8   . 51             PUSH ECX

0045FDE9   . 52             PUSH EDX

0045FDEA   . C745 C4 010000>MOV DWORD PTR SS:[EBP-3C],1

0045FDF1   . C745 BC 020000>MOV DWORD PTR SS:[EBP-44],2

0045FDF8   . C785 6CFFFFFF >MOV DWORD PTR SS:[EBP-94],4008

0045FE02   . FF15 C4104000  CALL DWORD PTR DS:[<&MSVBVM60.#632>]     ;  MSVBVM60.rtcMidCharVar               //这个函数是取字符函数    

0045FE08   . 8D45 AC       LEA EAX,DWORD PTR SS:[EBP-54] 这是取序列号的字符

0045FE0B   . 50             PUSH EAX

0045FE0C   . FF15 2C104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVa>;  MSVBVM60.__vbaStrVarMove 移动字符可见是“5”

0045FE12   . 8BD0           MOV EDX,EAX

0045FE14   . 8D4D D0        LEA ECX,DWORD PTR SS:[EBP-30]

0045FE17   . FF15 FC114000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove

0045FE1D   . 8D4D AC        LEA ECX,DWORD PTR SS:[EBP-54]

0045FE20   . 8D55 BC        LEA EDX,DWORD PTR SS:[EBP-44]

0045FE23   . 51             PUSH ECX  //存的是 ECX +8     35    

0045FE24   . 52             PUSH EDX         2

0045FE25   . 6A 02          PUSH 2

0045FE27   . FFD7           CALL EDI

0045FE29   . 66:8B45 EC     MOV AX,WORD PTR SS:[EBP-14]    取位数 i

0045FE2D   . 66:B9 0300     MOV CX,3            将 3 付给 ecx 的第位

0045FE31   . 66:99          CWD

0045FE33   . 66:F7F9        IDIV CX       //i％3     
    
0045FE36   . 83C4 0C        ADD ESP,0C       
    
0045FE39   . 66:85D2        TEST DX,DX   
    
0045FE3C   . 66:8995 12FFFF>MOV WORD PTR SS:[EBP-EE],DX   看看余数是否为零

0045FE43   . 75 2F     JNZ SHORT XYZ_c271.0045FE74  不为 0 跳  （有 3 种情况）     
```

分支 1

```
0045FE45   . 8B55 D0        MOV EDX,DWORD PTR SS:[EBP-30]

0045FE48   . 52             PUSH EDX

0045FE49   . FF15 44104000  CALL DWORD PTR DS:[<&MSVBVM60.#516>]     ;  MSVBVM60.rtcAnsiValueBstr

0045FE4F   .8B0D 64A04600  MOV ECX,DWORD PTR DS:[46A064 //存的是 58         ;  XYZ_c271.00460058         
    
0045FE55   . 33C1    XOR EAX,ECX    将序列号对应位上的字符的 ascii 码与之异或          
    
0045FE57   . 66:B9 3D00     MOV CX,3D

0045FE5B   . 66:99          CWD

0045FE5D   . 66:F7F9        IDIV CX

0045FE6066:8BF2     mOV SI,DX                  dx 为余数     
    
0045FE63   . 66:8B95 12FFFF>MOV DX,WORD PTR SS:[EBP-EE]

0045FE6A. 66:83C6 01   ADD SI,1           0045FE6E   .    将余数加 1   

0F80 90010000   JO XYZ_c271.00460004
```

分支 2

```
0045FE74   > 66:83FA 01     CMP DX,1             dx 为余数  和 1 比较

0045FE78   . 75 2E          JNZ SHORT XYZ_c271.0045FEA8

0045FE7A   . 8B55 D0        MOV EDX,DWORD PTR SS:[EBP-30]

0045FE7D   . 52             PUSH EDX

0045FE7E   . FF15 44104000  CALL DWORD PTR DS:[<&MSVBVM60.#516>]     ;  MSVBVM60.rtcAnsiValueBstr

0045FE84   . 66:3305 66A046>XOR AX,WORD PTR DS:[46A066] // [46A066 存的是 46

将序列号对应位上的字符的 ascii 码与之异或

0045FE8B   . 66:B9 3D00     MOV CX,3D        //将 3d 十六进制赋给 ecx 的低位

0045FE8F   . 66:99          CWD

0045FE91   . 66:F7F9        IDIV CX                   

0045FE94   . 66:8BF2        MOV SI,DX         求余后转移

0045FE97   . 66:8B9512FFFF>MOV DX,WORD PTR SS:[EBP-EE]

0045FE9E   . 66:83C6 01     ADD SI,1           余数加 1

0045FEA2   . 0F80 5C010000  JO XYZ_c271.00460004   
```

分支 3

```
0045FEA8   > 66:83FA 02     CMP DX,2             和 2 比较

0045FEAC   . 75 27          JNZ SHORT XYZ_c271.0045FED5

0045FEAE   . 8B55 D0        MOV EDX,DWORD PTR SS:[EBP-30]   // 注册码的 ascii

0045FEB1   . 52             PUSH EDX   

0045FEB2   . FF15 44104000  CALL DWORD PTR DS:[<&MSVBVM60.#516>]     ;  MSVBVM60.rtcAnsiValueBstr

0045FEB8   . 66:3305 68A046>XOR AX,WORD PTR DS:[46A068]    // [46A068]  中 4c 注册码的 ascii 异或

0045FEBF   . 66:B9 3D00     MOV CX,3D           

0045FEC3   . 66:99          CWD

0045FEC5   . 66:F7F9        IDIV CX

0045FEC8   . 66:8BF2        MOV SI,DX   dx 中为余数    

0045FECB   . 66:83C6 01     ADD SI,1              si +1

0045FECF   . 0F80 2F010000  JO XYZ_c271.00460004
```

经过三个分之后来到这进行处理
