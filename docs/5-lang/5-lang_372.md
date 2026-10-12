
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



### 第一段代码分析

```
0045FED5   > 8D55 E8       LEA EDX,DWORD PTR SS:[EBP-18]  //移入刚才的字符表
0045FED8   . 8D45 BC       LEA EAX,DWORD PTR SS:[EBP-44]
0045FEDB   . 0FBFCE        MOVSX ECX,SI
0045FEDE   . 8995 74FFFFFF MOV DWORD PTR SS:[EBP-8C],EDX
0045FEE4   . 50           PUSH EAX
0045FEE5   . 8D95 6CFFFFFF LEA EDX,DWORD PTR SS:[EBP-94]
0045FEEB   . 51           PUSH ECX
0045FEEC   . 8D45 AC      LEA EAX,DWORD PTR SS:[EBP-54]
0045FEEF   . 52           PUSH EDX
0045FEF0   . 50           PUSH EAX          //这些是一些处理
0045FEF1   . C745 C4 010000>MOV DWORD PTR SS:[EBP-3C],1
0045FEF8   . C745 BC 020000>MOV DWORD PTR SS:[EBP-44],2
0045FEFF   . C785 6CFFFFFF >MOV DWORD PTR SS:[EBP-94],4008
0045FF09   . FF15 C4104000 CALL DWORD PTR DS:[<&MSVBVM60.#632>] ;  MSVBVM60.rtcMidCharVar   //这个是关键跟进
```

这个函数就是在这个地址上加上 `si+1de` 十进制数 就是对应位的注册码，在这你就可以推出注册码了，为了清楚我贴出了他的全部过程，没兴趣可以不看，不过想提高的话还是看下去。

### 字符表

`UNICODE` 别咱我

```
"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz123456789"
```

### 第二段代码分析

```
0045FF0F   . 8D4D D4       LEA ECX,DWORD PTR SS:[EBP-2C]
0045FF12   . 8D55 9C       LEA EDX,DWORD PTR SS:[EBP-64]
0045FF15   . 898D 54FFFFFF MOV DWORD PTR SS:[EBP-AC],ECX
0045FF1B   . 52           PUSH EDX
0045FF1C   . 8D85 4CFFFFFF LEA EAX,DWORD PTR SS:[EBP-B4]
0045FF22   . 53           PUSH EBX
0045FF23   . 8D4D 8C      LEA ECX,DWORD PTR SS:[EBP-74]
0045FF26   . 50           PUSH EAX
0045FF27   . 51           PUSH ECX
0045FF28   . C745 A4 010000>MOV DWORD PTR SS:[EBP-5C],1
0045FF2F   . C745 9C 020000>MOV DWORD PTR SS:[EBP-64],2
0045FF36   . C785 4CFFFFFF >MOV DWORD PTR SS:[EBP-B4],4008
0045FF40   . FF15 C4104000 CALL DWORD PTR DS:[<&MSVBVM60.#632>] ;  MSVBVM60.rtcMidCharVar       //这个是取假注册码的函数
0045FF46   . 8D55 AC      LEA EDX,DWORD PTR SS:[EBP-54]
0045FF49   . 8D45 8C      LEA EAX,DWORD PTR SS:[EBP-74]
0045FF4C   . 52          PUSH EDX 用 d edx+8 可看见真码的地址一次只能看见 1 位
0045FF4D   . 50          PUSH EAX 用 d eax+8 可看见真码的地址一次只能看见 1 位
```

因为 `vb` 程序的处理方式教特别所以用 `*+8`

```
0045FF4E   . FF15 F0104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarTs>;  MSVBVM60.__vbaVarTstEq     //断点函数  关键比较
0045FF54   . 8D4D 8C      LEA ECX,DWORD PTR SS:[EBP-74]
0045FF57   . 8BD8         MOV EBX,EAX
0045FF59   . 8D55 AC      LEA EDX,DWORD PTR SS:[EBP-54]
0045FF5C   . 51          PUSH ECX
0045FF5D   . 8D45 9C      LEA EAX,DWORD PTR SS:[EBP-64]
0045FF60   . 52          PUSH EDX
0045FF61   . 8D4D BC      LEA ECX,DWORD PTR SS:[EBP-44]
0045FF64   . 50          PUSH EAX
0045FF65   . 51          PUSH ECX
0045FF66   . 6A 04       PUSH 4
0045FF68   . FFD7        CALL EDI
0045FF6A   . 83C4 14     ADD ESP,14
0045FF6D   . 66:85DB     TEST BX,BX
0045FF70   . 74 11       JE SHORT XYZ_c271.0045FF83
0045FF72   . 66:8B55 E4  MOV DX,WORD PTR SS:[EBP-1C]
0045FF76   . 66:83C2 01  ADD DX,1
0045FF7A   . 0F80 84000000 JO XYZ_c271.00460004
0045FF80   . 8955 E4      MOV DWORD PTR SS:[EBP-1C],EDX
0045FF83   > B8 01000000  MOV EAX,1
0045FF88   . 66:0345 EC   ADD AX,WORD PTR SS:[EBP-14]
0045FF8C   . 70 76        JO SHORT XYZ_c271.00460004
0045FF8E   . 8945 EC      MOV DWORD PTR SS:[EBP-14],EAX
0045FF91   . 8BC8         MOV ECX,EAX
0045FF93   . 8B45 E0      MOV EAX,DWORD PTR SS:[EBP-20]
0045FF96   . ^E9 2AFEFFFF JMP XYZ_c271.0045FDC5//结束没有没有的话跳
0045FF9B   > 66:3945 E4   CMP WORD PTR SS:[EBP-1C],AX
0045FF9F   . 75 07        JNZ SHORT XYZ_c271.0045FFA8    回去
```

上面那个 `Call`

取内存中的序列号的第二位 5 计算正确地序列号

### 第三段代码 (Call 1)

```
6A36B403  > 55           PUSH EBP
6A36B404   8BEC         MOV EBP,ESP
6A36B406   83EC 10      SUB ESP,10
6A36B409   56           PUSH ESI
6A36B40A   57           PUSH EDI
6A36B40B   FF35 C00E396A PUSH DWORD PTR DS:[6A390EC0]
6A36B411   FF15 B810286A CALL DWORD PTR DS:[<&KERNEL32.TlsGetValu>; KERNEL32.TlsGetValue
6A36B417   8D70 50      LEA ESI,DWORD PTR DS:[EAX+50]
6A36B41A   56           PUSH ESI
6A36B41B   FF75 0C      PUSH DWORD PTR SS:[EBP+C]
6A36B41E   E8 5096F3FF  CALL MSVBVM60.6A2A4A73
6A36B423   83F8 FF      CMP EAX,-1
6A36B426   74 3A        JE SHORT MSVBVM60.6A36B462
6A36B428   FF75 14      PUSH DWORD PTR SS:[EBP+14]       2
6A36B42B   FF75 10      PUSH DWORD PTR SS:[EBP+10]      31 上面计算的结果
6A36B42E   50           PUSH EAX
```

字符串

```
 0012ED70  0017A504  UNICODE "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz123456789"
```

```
6A36B42F   E8 AB94F3FF CALL MSVBVM60.rtcMidCharBstr //关键 call2 跟进
6A36B434   66:833E 08   CMP WORD PTR DS:[ESI],8
6A36B438   8945 F8      MOV DWORD PTR SS:[EBP-8],EAX
6A36B43B   66:C745 F0 0800 MOV WORD PTR SS:[EBP-10],8
6A36B441   75 0D        JNZ SHORT MSVBVM60.6A36B450
6A36B443   FF76 08      PUSH DWORD PTR DS:[ESI+8]
6A36B446   FF15 F019286A CALL DWORD PTR DS:[<&OLEAUT32.#6>]     ; OLEAUT32.SysFreeString
6A36B44C   66:8326 00   AND WORD PTR DS:[ESI],0
6A36B450   8B45 08      MOV EAX,DWORD PTR SS:[EBP+8]
6A36B453   8D75 F0      LEA ESI,DWORD PTR SS:[EBP-10]
6A36B456   8BF8         MOV EDI,EAX
6A36B458   A5           MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
6A36B459   A5           MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
6A36B45A   A5           MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
6A36B45B   A5           MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
6A36B45C   5F           POP EDI
6A36B45D   5E           POP ESI
6A36B45E   C9           LEAVE
6A36B45F   C2 1000      RETN 10
```

### Call 2 的代码

```
6A2A48DF  > 55           PUSH EBP
  6A2A48E0   8BEC         MOV EBP,ESP
6A2A48E2   83EC 10      SUB ESP,10
6A2A48E5   8B45 10      MOV EAX,DWORD PTR SS:[EBP+10]
6A2A48E8   66:8338 0A   CMP WORD PTR DS:[EAX],0A
6A2A48EC   75 26        JNZ SHORT MSVBVM60.6A2A4914
6A2A48EE   8178 08 04000280 CMP DWORD PTR DS:[EAX+8],80020004
6A2A48F5   75 1D        JNZ SHORT MSVBVM60.6A2A4914
6A2A48F7   83C9 FF      OR ECX,FFFFFFFF
6A2A48FA   66:85C9      TEST CX,CX
6A2A48FD   74 19        JE SHORT MSVBVM60.6A2A4918
6A2A48FF   50           PUSH EAX
6A2A4900   8B45 0C      MOV EAX,DWORD PTR SS:[EBP+C]   eax 为余数
6A2A4903   8D4400 FF    LEA EAX,DWORD PTR DS:[EAX+EAX-1]   2×eax-1
6A2A4907   50           PUSH EAX          字符串
6A2A4908   FF75 08      PUSH DWORD PTR SS:[EBP+8]
6A2A490B   E8 1E000000  CALL MSVBVM60.rtcMidBstr   跟进 call3
6A2A4910   C9           LEAVE
6A2A4911   C2 0C00      RETN 0C
```

### Call 3 的代码

```
6A2A492E  > 8B4424 08    MOV EAX,DWORD PTR SS:[ESP+8]  6A2A4932   53           PUSH EBX         字符所在的    位置          123456
6A2A4933   56           PUSH ESI
6A2A4934   57           PUSH EDI
6A2A4935   8D78 FF      LEA EDI,DWORD PTR DS:[EAX-1]
6A2A4938   85FF         TEST EDI,EDI          eax-1        78
6A2A493A   0F8C B6290200 JL MSVBVM60.6A2C72F6
6A2A4940   81FF FFFFFFFF CMP EDI,7FFFFFFF
6A2A4946   0F8F AA290200 JG MSVBVM60.6A2C72F6
6A2A494C   8B4424 10    MOV EAX,DWORD PTR SS:[ESP+10]
6A2A4950   8BDF         MOV EBX,EDI          ebx=78
6A2A4952   85C0         TEST EAX,EAX
6A2A4954   0F84 A3290200 JE MSVBVM60.6A2C72FD
6A2A495A   8B70 FC      MOV ESI,DWORD PTR DS:[EAX-4]
6A2A495D   3BFE         CMP EDI,ESI
```



```
6A2A495F   0F87 9F290200     JA MSVBVM60.6A2C7304

6A2A4965   8B4C24 18         MOV ECX,DWORD PTR SS:[ESP+18]

6A2A4969   66:8339 0A        CMP WORD PTR DS:[ECX],0A

6A2A496D   75 35             JNZ SHORT MSVBVM60.6A2A49A4

6A2A496F   8179 08 04000280 CMP DWORD PTR DS:[ECX+8],80020004

6A2A4976   75 2C             JNZ SHORT MSVBVM60.6A2A49A4

6A2A4978   83C8 FF           OR EAX,FFFFFFFF

6A2A497B   66:85C0           TEST AX,AX

6A2A497E   74 28             JE SHORT MSVBVM60.6A2A49A8

6A2A4980   2BF3              SUB ESI,EBX

6A2A4982   8BC6              MOV EAX,ESI

6A2A4984   50                PUSH EAX

6A2A4985   8B4424 14         MOV EAX,DWORD PTR SS:[ESP+14]

6A2A4989   03D8              ADD EBX,EAX

6A2A498B   53                PUSH EBX     这个是关键

6A2A498C   FF15 FC19286A     CALL DWORD PTR DS:[<&OLEAUT32.#150>]     ; OLEAUT32.SysAllocStringByteLen

6A2A4992   8BF0              MOV ESI,EAX

6A2A4994   85F6              TEST ESI,ESI

6A2A4996   0F84 76290200     JE MSVBVM60.6A2C7312

6A2A499C   8BC6              MOV EAX,ESI

6A2A499E   5F                POP EDI

6A2A499F   5E                POP ESI

6A2A49A0   5B                POP EBX

6A2A49A1   C2 0C00           RETN 0C
```

唉终于写完了！奋战了几个小时，写的不太好，凑或着看吧！呵呵！

这个软件很容易，大家拿来作为`vb`的练手吧，欢迎你的批评与指导！

\_\_ \_\_ \_\_ \_\_ \_\_ \_\_ \_\_ \_\_ \_\_ \_\_ \_\_ \_\_ \_\_

\----------------\*\*\*\*\*\*---------------

\--------------"欢迎交流"------------------

\----------------\*\*\*\*\*\*---------------

【`BCG`】【`FCG`】【`DFCG`】【`NUKE`】【`IPB`】

`4dd466d0e1e0cc123afbe504da2462ca`
