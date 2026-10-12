
# 超级函数表达式运算器 v20 

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-68.htm`](https://www.pediy.com/kssd/pediy08/pediy8-68.htm)

## 软件名称
超级函数表达式运算器 v20 算法分析  

## 软件介绍  

超级函数表达式运算器既是方便实用的数字计算器，又是一个会列表达式就会运算的超级新颖的多功能公式化运算器。  

它包括算术计算、进制转换、逻辑处理、数值处理、字符处理、三角函数及推导函数处理等模块，可综合性的计算各种多层嵌套、复杂的函数表达式。你只要象在纸上列表达式一样列出符合要求的运算式，就立即可以看到运算结果，而且运算结果随着表达式的不断变化而不断变换显示。表达式修改方便，可复制粘贴，可用键盘快速输入。2.0 版界面更加美观、提示更加科学、操作更加简单、更加方便实用。2.0 版新增独特的批量运算功能，可一次运算多个表达式的值并可以计算各单项运算之和。可广泛应用于办公、学习及需要使用计算器进行复杂计算的各个领域。超级函数表达式运算器是进行数字计算、公式运算、函数应用、`vb` 编程的好帮手。  

## 使用限制  
试用 30 天，无功能限制  

## 保护方式  
采用机器码检查注册码，以保证注册码不得多机使用。并在程序退出时检验注册码，使破解增加了难度。  

## 破解工具  
`SmartCheck6.20`、`olldbg1.1`、`Peid0.93`、`QuickUnpack`、`Regmon7.02`  

## 破解目的  
研究算法分析, 了解 `VB` 的数据类型，掌握点儿知识  

## 破解作者  
`CCRLEI` `ccrlei@yahoo.com.cn`  

## 作者声明  
初学 Crack，只是感兴趣，没有其他目的。  

本破解纯以学习和交流为目的，偶得一点心得，大家分享。如有错误，蒙请大侠们指正。转载时请保持完整。  

之所以起这个题目，是因为表面上程序已经提示你注册成功了，但程序在退出时还要校验注册码是否含作者指定的内容，如果没有的话，就会自动删除该程序，够狠的。好了，废话少说，入正题吧！  

## 破解过程  

### 1、查壳  

`PEiDv0.93`  

`PECompact` 2.x -> Jeremy Collake  

### 2、脱壳  

`QuickUnpack`  

### 3、编程语言  

`PEiDv0.93`  

`Microsoft Visual Basic` 5.0 / 6.0  

### 4、破解调试  

1）运行程序，提示共享软件还可用多少天，欢迎注册使用等提示。进入注册选项发现：机器码 `22222222222854720884_754026158`，要求输入注册码。  

2）由于作者使用 `VB` 开发，所以先用 `SmartCheck` 看看作者的关于注册的大概思路。  

软件启动后（主要部分）  

（1）读取注册表 `"SOFTWARE\softjas\data\Nam"` 下键值  

```
RegCreateKeyA(HKEY:80000002, char \*:00140934, PTR:0012FD44) returns LONG:0
```

键值内容转换后和机器码比较（通过读硬盘型号、硬盘物理系列号生成机器码，由机器码变成注册码的一部分（另一部分在退出程序时使用）） 如不符则  

（2）读键值  

`HKEY_CURRENT_USER\Software\Microsoft\htoolse\Data\w1`、`w2`、`w3` 这一部分在后面介绍，它的目的是算还共享多少天。  

（3） 顺便说一下程序如何得到机器码：a、通过读硬盘型号`"85TC2088T"`、硬盘物理系列号`"TOSHIBA MK4026GAX"`  

b、`"TOSHIBA MK4026GAX"`取空格后的 `MK4026GAX`，第一个字符“M”Ascii 值 77，取后面的 7，第二个字符“K”75－》5，数字的不变  
“4”－》4，“0”－》0，“6”－》6，“G”71－》1，“A”65－》5，“X”88－》8－－－－－－－－－》`754026158`  

c、`"85TC2088T"`换成`"           85TC2088T"`（11 个空格共二十位），第一个“ ”（空格）Ascii 值 32 取后面“2”，依次共 11 个“2”  
即“`22222222222`” 数字不变“`85`”，“`T`”84－“`4`”。。。。。。。。。同上，所以本机机器码“`22222222222854720884_754026158`”  

以上是通过 `SmartCheck` 分析得(机器码部分没有经 `OD` 核实，哪位大侠如经汇编核实，请告诉我我的转换分析是否正确，尤其是 11 个空格那儿)，但输入的试炼码如何转换并不清楚，虽说通过 `SmartCheck` 完全可以试出注册码，但我们的目的应该是注册机，呵呵，所以 `OD` 上。  

3）用 `OD` 调试：（只作简单说明）  

由于作者采用`"InStr"`比较真假注册码，所以用`__vbainstr` 下断  

`OD` `Alt+E` `Ctrl+N` `__vbainstr` 下断  

运行，输入试练码 `876543210`  

断在 `7628BD94` > `55` `PUSH` `EBP`  

`Ctrl+F9`， `F8` 回到主程序 ，上下看看，左右找找，在这里  

```
00441460   . |50             PUSH EAX                                             
00441461   . |FF15 B0734400  CALL DWORD PTR DS:[<&msvbvm50.__vbaStrVarVal>]     ;  msvbvm50.__vbaStrVarVal 
00441467   . |50             PUSH EAX
00441468   . |FFD7           CALL EDI          ;  注册码的第一个字符 8 的值“38”  (16 进制）放入到 AX 中 
0044146A   . |68 FC994000    PUSH 超级函数.004099FC
0044146F   . |66:8BD8        MOV BX,AX                     ;   BX=AX=0038                             
00441472   . |FFD7           CALL EDI    ;msvbvm50.rtcAnsiValueBstr  Ascii 转换 “A”－>  41 (16 进制） 放到 AX 中 
00441474   . |66:2BD8        SUB BX,AX                 ;  BX=38－41＝FFF9   及十进制－9           
00441477   . |8D95 38FFFFFF  LEA EDX,DWORD PTR SS:[EBP-C8]                ； 
0044147D   . |0F80 B3010000  JO 超级函数.00441636
00441483   . |8D4D 9C        LEA ECX,DWORD PTR SS:[EBP-64]
00441486   . |66:899D 40FFF>MOV WORD PTR SS:[EBP-C0],BX                
。。。
004414B2   . |FF15 B0734400  CALL DWORD PTR DS:[<&msvbvm50.__vbaStrVarVal>]    ;   换成 10 进制的字符“－9”，放入 Eax
004414B8   . |50             PUSH EAX                   ；Eax＝“－9”入栈
004414B9   . |FF15 80744400  CALL DWORD PTR DS:[<&msvbvm50.rtcR8ValFromBstr>]  ;  msvbvm50.rtcR8ValFromBstr 
004414BF   . |DC25 48204000  FSUB QWORD PTR DS:[402048]  ；浮点减 st=－9.000000000000000000  ds:[00402048]=-0.50000000
                                                 ；St0=－8.500000000
004414C5   . |833D 00304400>CMP DWORD PTR DS:[443000],0
004414CC   . |75 08          JNZ SHORT 超级函数.004414D6
004414CE   . |DC35 50204000  FDIV QWORD PTR DS:[402050]   ；浮点除 st=-8.500000000000000000  ds:[00402050]=2.500000000
                                                 ; St0＝－3.400000
004414D4   . |EB 11          JMP SHORT 超级函数.004414E7   
004414D6   > |FF35 54204000  PUSH DWORD PTR DS:[402054]
004414DC   . |FF35 50204000  PUSH DWORD PTR DS:[402050]
004414E2   . |E8 DD0BFCFF    CALL <JMP.&msvbvm50._adj_fdiv_m64>
004414E7   > |DFE0           FSTSW AX
004414E9   . |A8 0D          TEST AL,0D
004414EB   . |0F85 40010000  JNZ 超级函数.00441631
004414F1   . |FF15 68744400  CALL DWORD PTR DS:[<&msvbvm50.__vbaFPInt>] ;  msvbvm50.__vbaFPInt 取整   St0=－4.00000
004414F7   . |DD9D 40FFFFFF  FSTP QWORD PTR SS:[EBP-C0]           ;St0=－4.00000           
.........
0044152C   .   FF15 B8734400  call   [<&msvbvm50.__vbaVarCat>]    ;  msvbvm50.__vbaVarCat       字符串连接 
。。。。
00441534   . |8D8D 6CFFFFFF  LEA ECX,DWORD PTR SS:[EBP-94]           ;ecx="－4"
0044153A   . |FFD6           CALL ESI     
0044153C   . |8D85 D8FEFFFF  LEA EAX,DWORD PTR SS:[EBP-128]
00441542   . |8D8D E8FEFFFF  LEA ECX,DWORD PTR SS:[EBP-118]
00441548   . |50             PUSH EAX
00441549   . |8D55 DC        LEA EDX,DWORD PTR SS:[EBP-24]
0044154C   . |51             PUSH ECX
0044154D   . |52             PUSH EDX
0044154E   . |FF15 70744400  CALL DWORD PTR DS:[<&msvbvm50.__vbaVarForNext>] ;  msvbvm50.__vbaVarForNext  
注册码下一个字符 00441554   . |8B1D 18734400 MOV EBX,DWORD PTR DS:[<&msvbvm50.rtcMidCharVar>]  ;  msvbvm50.rtcMidCharVar     
0044155A   . ^\E9 A2FEFFFF   JMP 超级函数.00441401
。。。
0044152C   .   FF15 B8734400  call   [<&msvbvm50.__vbaVarCat>]    ;  msvbvm50.__vbaVarCat 

876543210

注：读取试练码（个数为偶数）的顺序为最后一个、第一个、倒数第二个、第二个。。。。依此类推 ,个数为奇数的顺序为第一个、倒数第一个、第二个、倒数第二个。。。。依此类推  ，本例为奇数个  

计算结果`"-4-7-4-7-5-6-5-6-5"` 和本机的机器码 `22222222222854720884_754026158` 的前一部分即 `22222222222854720884` 用 `vbainstr` 比较如结果大于 0，`Vbainstr` 比较后一段 `754026158`，通过则注册成功。

另外忘了在哪里机器码的前十位每两位对应一个字符，后面一个对应一个（大侠帮忙）  

4）推注册码：

以我的机器为例: 机器码： `22222222222854720884_754026158`

```
x x x x xFUNKSFAUUK  SNKAFPDNU
```

由于校验时是后一个前一个的读注册码（机器码位数为偶数），所以还得给他调换一下，后一个前一个

`xxxxxFUNKSFAUUKSNKAFPDNU－》xxFNSAUSKFDUNPANKUFKUxxx` (每个机器码字符可对应 2～3 个注册码字符，因为上面有个除以 2.5)

好了注册码出来了，试试，运行程序－》注册－》填入注册码 嘿嘿嘿嘿，通过了，提示我“恭喜您，注册成功！”

5）话说几天后，又拿出这个软件用用，算算术什么的，还行，挺好用，关闭退出，咦，怎么桌面上的快捷方式图标变了，在点击，说找不到可执行文件了，由于是春节放假期间，我以为我的笔记本中了病毒，因为杀毒软件好几天没升级了，也没理他，再把原来备份的一个文件恢复出来接着用，运行了几次又没了，这下子怪了，升级杀毒软件，杀毒，没问题呀，我想我想，我使劲的想，哼哼，应该是程序本身的问题吧，是不是我的注册码中“x”太多，像是破解出来的，（因为把 x 改成 y 也可以注册），或者。。。。不明白，接着使用 `SmartCheck`，看看程序退出时有什么动作，哈哈

```
InStr(long:1, String:"xxFNSAUSK...", String:"CJHBYS", Integer:0) returns LONG:0
```

Arguments

```
\--------------------
Long  start = 1 0x00000001

String  string1 = 00141EC4
      = "xxFNSAUSKFDUNPANKUFKUxxx"
String  string2 = 0040952C
      = "CJHBYS"
Integer  compare = 0 0x0000
```

还要比较注册码中是否有字符串`"CJHBYS"`，小样的，往下看看，如果注册码中没有`"CJHBYS"`，就生成个批处理文件 `In.bat` 把程序删掉，够狠的吧。把它加到注册码里，运行（可先到注册表把 `"SOFTWARE\softjas\data\Nam"` 下值删掉，就可重新注册），重新注册 `xxFNSAUSKFDUJBSYHCNPANKUFKUxxx` 通过，再用 `SmartCheck` 看看退出，还要检验，这回检验是否含有“`5309582`”。。。。我烦了，我哪知道那个破数是怎么来的，检测不过去程序又会自动生成一个 `In.bat` 文件写上删除这个可执行文件，再删除 `In.bat`。

不想了，睡觉去。。。。。。

6）若干天后

还是得解决“`5309582`”的来源问题，接着 `OD` 吧，原来是 And 来得，如下：

```
；以下为退出时检验注册码
。。。。。。
00435623   .   FF15 F8734400 call   [<&msvbvm50.__vbaStrCopy>]    ;  msvbvm50.__vbaStrCopy 
00435629   >   8B45 E8        mov     eax, [ebp-18]             ;机器码前八位 22222222 放入 Eax
0043562C   .   50            push    eax                       ；推入
0043562D   .   FF15 80744400 call   [<&msvbvm50.rtcR8ValFromBstr>>;  msvbvm50.rtcR8ValFromBstr
```



### 00435633 代码分析

```
00435633   .  8B3D 40744400  mov     edi, [&msvbvm50.__vbaFpI4]   ; msvbvm50.__vbaFpI4
00435639   .  FFD7          call    edi               ; <&msvbvm50.__vbaFpI4> 将 22222222 转为 16 进制 放入 Eax
0043563B   .  8B4D E4       mov     ecx, [ebp-1C]            ;机器码后九位 754026158 放入 Ecx
0043563E   .  8BD8          mov     ebx, eax                 ；将前八位转为 16 进制的数再放入 Ebx
00435640   .  51            push    ecx                      ；把 754026158 推入
00435641   .  FF15 80744400 call    [&msvbvm50.rtcR8ValFromBstr];  msvbvm50.rtcR8ValFromBstr
00435647   .  FFD7          call    edi                      ；将后 9（或 8 位）转换为 16 进制放入 Eax
00435649   .  23D8          and     ebx, eax                ；两个数逻辑与 结果 51048E  即 10 进制的 5309582
0043564B   .  53            push    ebx
0043564C   .  FF15 70724400 call    [&msvbvm50.__vbaStrI4]  ;  msvbvm50.__vbaStrI4  51048E 的 10 进制 5309582
00435652   .  8BD0          mov     edx, eax
00435654   .  8D4D E0       lea     ecx, [ebp-20]
00435657   .  FFD6          call    esi
00435659   .  8B15 58304400 mov     edx, [443058]            ；注册码放入 Edx
0043565F   .  6A 01         push    1
00435661   .  52            push    edx                      ；Edx 入栈
00435662   .  33FF         xor     edi, edi
00435664   .  68 2C954000   push    0040952C             ;  UNICODE "CJHBYS" 入栈
00435669   .  57            push    edi
0043566A   .  FF15 DC734400 call    [&msvbvm50.__vbaInStr] ;  msvbvm50.__vbaInStr 检验是否有"CJHBYS"
00435670   .  83F8 01       cmp     eax, 1                ；"CJHBYS"出现在第几位开始（19）和 1 比较
00435673   .  7D 12         jge     short 00435687    ；大于等于 1（及注册码中含"CJHBYS"）则跳如下一步，
                                                     生成 In.bat 见下面
00435675   .  8D45 D0       lea     eax, [ebp-30]
00435678   .  50            push    eax
00435679   .  E8 92F7FFFF   call    00434E10
0043567E   .  8D4D D0       lea     ecx, [ebp-30]
00435681   .  FF15 78724400 call    [&msvbvm50.__vbaFreeVar]     ;  msvbvm50.__vbaFreeVar
00435687   >  8D4D A0       lea     ecx, [ebp-60]
0043568A   .  8D55 D0       lea     edx, [ebp-30]
0043568D   .  51            push    ecx
0043568E   .  52            push    edx
0043568F   .  C745 A8 58304>mov     dword ptr [ebp-58], 00443058
00435696   .  C745 A0 08400>mov     dword ptr [ebp-60], 4008
0043569D   .  E8 9EBF0000   call    00441640
004356A2   .  8B45 E0       mov     eax, [ebp-20]        ；将 5309582 放入 Eax
004356A5   .  8D4D D0       lea     ecx, [ebp-30]
004356A8   .  6A 01         push    1
004356AA   .  8D55 90       lea     edx, [ebp-70]
004356AD   .  8945 98       mov     [ebp-68], eax
004356B0   .  51            push    ecx
004356B1   .  52            push    edx
004356B2   .  8D45 C0       lea     eax, [ebp-40]
004356B5   .  57            push    edi
004356B6   .  50            push    eax
004356B7   .  C745 90 08000>mov     dword ptr [ebp-70], 8
004356BE   .  C745 88 01000>mov     dword ptr [ebp-78], 1
004356C5   .  C745 80 02800>mov     dword ptr [ebp-80], 8002
004356CC   .  FF15 AC734400 call    [&msvbvm50.__vbaInStrVar];msvbvm50.__vbaInStrVar 注册码换算后检验是
                                                    否含 5309582
004356D2   .  8D4D 80       lea     ecx, [ebp-80]
```

#### 7）推注册码

`22222222222854720884_754026158` 和 `5309582` 连接
`222222222228547208847540261585309582`

`xxxxxFUNKSFAUUKSNKAFPDNUNIAXNUF` 共 31 位为奇数,所以要头一个尾一个的排下序
`xxxUKFUKNAPNNANFUXIUDFKSUASNFxx` 最后再把要检验的 `CJHBYS` 插入
`xxxUKFUKNAPNNANFCJHBYSUXIUDFKSUASNFxx`

把注册码拷贝到注册表 `"SOFTWARE\softjas\data\Nam"` 下(程序把注册码放在这个地方),`SmartCheck` 运行程序,初始化时
校验注册码通过,退出程序,`Unload` 时校验注册码也通过,这回就不生成 `In.bat` 文件了,所以程序也就不被删除了.

#### 8）退出程序时代码分析

；以下为程序退出时生成新文件来删除程序  注: 我把 `<超级函数表达式运算器 v20.ece>` 改为了 `<1234.exe>` 方便调试

 （因为调试时的中文文件名有时会显示出乱码，所以改了）

```
0043507A   >  \\8B55 C8       mov     edx, [ebp-38]              ;文件名放倒 Edx 本事例为"1234"
0043507D   .  52            push    edx                      ;入堆栈
0043507E   .  68 A4944000   push    004094A4               ;
00435083   .  FFD7          call    edi                      ;字符串连接后 1234.exe 放入 Eax
00435085   .  8BD0          mov     edx, eax               ;edx 1234.exe
00435087   .  8D4D CC       lea     ecx, [ebp-34]             ;
```

```
004350A6   .  FF15 F8734400 call    [&msvbvm50.__vbaStrCopy]       ;  msvbvm50.__vbaStrCopy
004350AC   .  8B45 E4       mov     eax, [ebp-1C]                 ;in.bat 放入 Eax
004350AF   .  50            push    eax                  ；入栈
004350B0   .  6A 01         push    1
004350B2   .  6A FF         push    -1
004350B4   .  6A 02         push    2
004350B6   .  FF15 D0734400 call    [&msvbvm50.__vbaFileOpen]      ;  msvbvm50.__vbaFileOpen
                                                   打开文件 in.bat
004350BC   .  8B1D 8C734400 mov     ebx, [&msvbvm50.__vbaPrintFile];  msvbvm50.__vbaPrintFile
004350C2   .  68 C8944000   push    004094C8
004350C7   .  6A 01         push    1
004350C9   .  68 D8754000   push    004075D8
004350CE   .  FFD3          call    ebx                    ;  <&msvbvm50.__vbaPrintFile>
004350D0   .  8B4D CC       mov     ecx, [ebp-34]
004350D3   .  83C4 0C       add     esp, 0C
004350D6   .  68 DC944000   push    004094DC
004350DB   .  51            push    ecx
004350DC   .  FFD7          call    edi
004350DE   .  8BD0          mov     edx, eax                      ;edx=eax="del 1234.exe"
004350E0   .  8D4D C8       lea     ecx, [ebp-38]                 ;
004350E3   .  FFD6          call    esi
004350E5   .  50            push    eax
004350E6   .  6A 01         push    1
004350E8   .  68 D8754000   push    004075D8
004350ED   .  FFD3          call    ebx
004350EF   .  83C4 0C       add     esp, 0C
004350F2   .  8D4D C8       lea     ecx, [ebp-38]
004350F5   .  FF15 78744400 call    [&msvbvm50.__vbaFreeStr]       ;  msvbvm50.__vbaFreeStr
004350FB   .  8B55 CC       mov     edx, [ebp-34]               ；1234.exe
004350FE   .  68 EC944000   push    004094EC                 ;  UNICODE "if exist "
00435103   .  52            push    edx                      ；1234.exe
00435104   .  FFD7          call    edi                    ;msvbvm50.__vbaStrCat
00435106   .  8BD0          mov     edx, eax              ;UNICODE "if exist 1234.exe"
00435108   .  8D4D C8       lea     ecx, [ebp-38]
0043510B   .  FFD6          call    esi                    ;msvbvm50.__vbaStrCat
0043510D   .  50            push    eax
0043510E   .  68 04954000   push    00409504                 ;  UNICODE " GOTO START"
00435113   .  FFD7          call    edi                      ;msvbvm50.__vbaStrCat
00435115   .  8BD0          mov     edx, eax        ;UNICODE "if exist 1234.exe  GOTO START"
00435117   .  8D4D C4       lea     ecx, [ebp-3C]
0043511A   .  FFD6          call    esi
0043511C   .  50            push    eax
0043511D   .  6A 01         push    1
0043511F   .  68 D8754000   push    004075D8
00435124   .  FFD3          call    ebx                     ;msvbvm50.__vbaPrintFile
00435126   .  83C4 0C       add     esp, 0C
00435129   .  8D45 C4       lea     eax, [ebp-3C]
0043512C   .  8D4D C8       lea     ecx, [ebp-38]
0043512F   .  50            push    eax
```



## 汇编代码片段

```
00435130   .  51                push    ecx  
00435131   .  6A 02             push    2  
00435133   .  FF15 00744400 call    [< &msvbvm50.__vbaFreeStrList>]   ;  msvbvm50.__vbaFreeStrList  
00435139   .  83C4 0C           add     esp, 0C  
0043513C   .  68 DC944000      push    004094DC  
00435141   .  68 B4944000      push    004094B4  
00435146   .  FFD7            call    edi  
00435148   .  8BD0            mov     edx, eax                            ;(UNICODE "del in.bat")  
0043514A   .  8D4D C8         lea     ecx, [ebp-38]  
0043514D   .  FFD6            call    esi  
0043514F   .  50              push    eax  
00435150   .  6A 01            push    1  
00435152   .  68 D8754000      push    004075D8  
00435157   .  FFD3            call    ebx                                   ;msvbvm50.__vbaPrintFile  
00435159   .  83C4 0C          add     esp, 0C  
0043515C   .  8D4D C8          lea     ecx, [ebp-38]  
0043515F   .  FF15 78744400  call    [< &msvbvm50.__vbaFreeStr>]         ;   
00435165   .  6A 01            push    1  
00435167   .  FF15 20734400   call    [< &msvbvm50.__vbaFileClose>]     ;  msvbvm50.__vbaFileClose  
0043516D   .  8D55 E4         lea     edx, [ebp-1C]  
00435170   .  8D45 80         lea     eax, [ebp-80]  
00435173   .  6A 00            push    0  
00435175   .  50              push    eax  
00435176   .  8955 88         mov     [ebp-78], edx  
00435179   .  C745 80 08400> mov     dword ptr [ebp-80], 4008  
00435180   .  FF15 70734400   call    [< &msvbvm50.rtcShell>]           ;  msvbvm50.rtcShell  
                                                                          shell 执行文件 In.bat  
00435186   .  DDD8            fstp    st  
00435188   .  FF15 90724400   call    [< &msvbvm50.__vbaEnd>]           ;  msvbvm50.__vbaEnd  结束  
```

### 9）`In.bat` 文件内容

（注：我调试时把文件名改为了 `1234.exe`）

```batch
:START  
del 1234.EXE  
if exist 1234.EXE GOTO START  
del in.bat  
```

这个文件有时删除不了那个可执行文件，尤其是文件名中带括号的，原因不知，可能是全角和半角的事吧。

### 10）注册机算法：`VB` 做得

```vb
'超级函数表达式运算器 v20 注册机  
'ccrlei  
'2006-2-8  

Private Sub Command2_Click()  
Unload Me  
End Sub  

Private Sub Command1_Click()  
Dim SnWord, SJqm, SJqm1, Sntemp, Sntemp1, Sntemp2, Sntemp3 As String  
Dim i, j As Integer  
Dim Dsn, SnLen1, JqmLen, JqmLenL, JqmLenR As Integer  
Dim Sq, Sh, Stemp As String  

SJqm = Text1.Text               '机器码变量  
JqmLen = Len(SJqm) - 1         '机器码长度  
JqmLenL = InStr(1, SJqm, "\_", vbBinaryCompare) - 1 '"\_"左边字符串长度  
JqmLenR = JqmLen - JqmLenL       '"\_"右边字符串长度  

SJqm = Mid(SJqm, 1, JqmLenL) + Mid(SJqm, JqmLenL + 2, JqmLenR) '去掉“\_”  
Sq = Mid(SJqm, 1, 8)    '程序退出时检验注册码要用机器码前八位  
Sh = Mid(SJqm, JqmLenL + 1, JqmLenR)  '程序退出时检验注册码要用机器码原“\_”后字符串  

If Len(Sh) > 9 Then                  '机器码后段位数大于 9 位时取 8 位，否则取 9 位  
    Sh = Mid(SJqm, JqmLen - 7, 8)  
End If  

Stemp = CDbl(Sq) And CDbl(Sh)    '前八位和后 8 或 9 位逻辑与  
SJqm1 = SJqm + Stemp   '新生成的机器码  

i = 0  
For i = 1 To Len(SJqm1) - 5        '机器码前 10 位每 2 位对应一个注册码  
    If i <= 5 Then  
        SnWord = Mid(SJqm1, i * 2 - 1, 2)  
    Else  
        SnWord = Mid(SJqm1, 5 + i, 1)  
    End If  
Dsn = Int(Val(SnWord) * 2.5 - 0.5 + 65 + 1)     '转换 详见上介绍  
Sntemp = Sntemp + Chr(Dsn)                     '连接  
Next i  

SnLen1 = Len(Sntemp)      '新注册码长度  

If SnLen1 / 2 <> Int(SnLen1 / 2) Then                 '根据注册码位数的奇偶性重新排序  
    i = 0  
    For i = 1 To SnLen1  
        If i / 2 <> Int(i / 2) Then  
            Sntemp1 = Sntemp1 + Mid(Sntemp, i, 1)  
        Else  
            Sntemp2 = Mid(Sntemp, i, 1) + Sntemp2  
        End If  
    Next i  
    Sntemp3 = Sntemp1 + Sntemp2  
    sntemp4 = Mid(Sntemp3, 1, Int(SnLen1 / 2) + 1) + "CJHBYS" + Mid(Sntemp3, Int(SnLen1 / 2) + 2 _  
    , SnLen1 - Int(SnLen1 / 2) - 1)  
Else  
    i = 0  
    For i = 1 To SnLen1  
        If i / 2 = Int(i / 2) Then  
            Sntemp1 = Sntemp1 + Mid(Sntemp, i, 1)  
        Else  
            Sntemp2 = Mid(Sntemp, i, 1) + Sntemp2  
        End If  
    Next i  
    Sntemp3 = Sntemp1 + Sntemp2  
    sntemp4 = Mid(Sntemp3, 1, SnLen1 / 2) + "CJHBYS" + Mid(Sntemp3, SnLen1 / 2 + 1, SnLen1 / 2)    
    '插入检验所需的“CJHBYS”  
End If  

Text3.Text = sntemp4  
End Sub  

Private Sub Form_Load()  
Text1.Text = "22222222222854720884_754026158"    '本机机器码  
'Text1.Text = "22222222222257244519_3438081953"  
Text3.Text = ""          '注册码  
End Sub  
```

### 11）其他

安装完软件，第一次运行后程序会写入：

1. 注册表 `Software\Microsoft\htoolse\Data`
   - `\w1` `24EC652` 由 `hex(安装日期*999-234)` 得来
   - `\w2` `20D2118`     `hex( 最后一次运行日期 * 888-456)`
   - `\w3` `20D2118`     `hex((最后一次运行日期*999 ) Xor (最后一次运行日期*888 ))`

写文件： `"C:\WINNT\system32\config\cdelisub.hld"`

`"24EC652","20D2118","4449756"`     各项内容同上

并且这些内容卸载不掉，这样的话再重新安装该软件，共享期不会因此而改为 30 天，而是该日期从你第一次运行该软件算起。

### 12）破解心得

- a、善用多种工具进行分析，诸如 `Regmon`、`filemonnet`、`SmartCheck`，本次破解以 `SmartCheck` 分析为主，其他为辅；
- b、该软件作者在加密方面的却下了一番功夫，很多人都会被注册时“恭喜您，注册成功！”所迷惑，以为已成功破解，其实不然，软件在退出程序时再校验一次注册码，并且以删除文件来阻止他人用破解的注册码注册，还好，作者还是比较有善心的，只删除自己的文件，如果他要是删除些系统文件什么的，我的机器还不得重装几回呀，表扬表扬作者。
- c、在注册码和文件中写入第一次安装日期、最后一次运行日期的办法值得学习一下，但是乱丢垃圾就不好了，呵呵！

[版权声明]: 本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
