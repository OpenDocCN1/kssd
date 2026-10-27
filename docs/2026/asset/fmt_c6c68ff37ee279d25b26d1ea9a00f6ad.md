
# 逆向分析片段

`004697E4`   `. 8D45 C8        LEA EAX,DWORD PTR SS:[EBP-38]`

`004697E7`   `. 50             PUSH EAX`

`004697E8`   `. 8D55 C4        LEA EDX,DWORD PTR SS:[EBP-3C]`

`004697EB`   `. 8D86 E8030000  LEA EAX,DWORD PTR DS:[ESI+3E8]`

`004697F1`   `. E8 62F2F9FF    CALL ip.00408A58`

`004697F6`   `. 8B45 C4        MOV EAX,DWORD PTR SS:[EBP-3C]`

`004697F9`   `. B9 03000000    MOV ECX,3`

`004697FE`   `. BA 02000000    MOV EDX,2`

`00469803`   `. E8 4CB6F9FF    CALL ip.00404E54`

`00469808`   `. 8B55 C8        MOV EDX,DWORD PTR SS:[EBP-38]`

`0046980B`   `. 8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]`

`0046980E`   `. E8 E9B3F9FF    CALL ip.00404BFC`   将”000”连到字串后面

`00469813`   `. 43             INC EBX`

`00469814`   `. 83FB 21        CMP EBX,21`  比较长度小于继续跳到`4697D7`

`00469817`   `.^75 BE          JNZ SHORT ip.004697D7`

经计算后（计算很简单不分析了为什么呢？因为这儿于注册码无关迷惑破解者的）

`"146154149134164189188184162164185188184162759000024669157212524249126291334064373055663047231726"`

`00469819`   `> 8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]`

`0046981C`   `. E8 13B1F9FF    CALL ip.00404934`

`00469821`   `. 8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]`

`00469824`   `. E8 CBB3F9FF    CALL ip.00404BF4`

`00469829`   `. 8BD8           MOV EBX,EAX`

`0046982B`   `. 83FB 01        CMP EBX,1`

`0046982E`   `. 7C 1F          JL SHORT ip.0046984F`

`00469830`   `> 8D45 C0        LEA EAX,DWORD PTR SS:[EBP-40]` 对上面的字符串逆序

`00469833`   `. 8B55 F4        MOV EDX,DWORD PTR SS:[EBP-C]`

`00469836`   `. 8A541A FF      MOV DL,BYTE PTR DS:[EDX+EBX-1]`

`0046983A`   `. E8 DDB2F9FF    CALL ip.00404B1C`

`0046983F`   `. 8B55 C0        MOV EDX,DWORD PTR SS:[EBP-40]`

`00469842`   `. 8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]`

`00469845`   `. E8 B2B3F9FF    CALL ip.00404BFC`

`0046984A`   `. 4B             DEC EBX`

`0046984B`   `. 85DB           TEST EBX,EBX`

`0046984D`   `.^75 E1          JNZ SHORT ip.00469830`

`0046984F`   `> 8B4D F8        MOV ECX,DWORD PTR SS:[EBP-8]`

结果为

`"627132740366550373460433192621942425212751966420000957261481881581461261481881981461431941451641"`

`00469852`   `. BA F0994600    MOV EDX,ip.004699F0          ;  ASCII "k1"`

将它写入注册表

`00469857`   `. 8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]`

`0046985A`   `. E8 0947FCFF    CALL ip.0042DF68`

`0046985F`   `> 33C0           XOR EAX,EAX`

`00469861`   `. 5A             POP EDX`

`00469862`   `. 59             POP ECX`

`00469863`   `. 59             POP ECX`

上面的`CALL` 慢慢的 经过一番寻找来到这里

``` 
004690C6  |> 8D5434 08      LEA EDX,DWORD PTR SS:[ESP+ESI+8]
004690CA  |. 8D43 1C        LEA EAX,DWORD PTR DS:[EBX+1C]
004690CD  |. B9 40000000    MOV ECX,40
004690D2  |. E8 85E2F9FF    CALL ip.0040735C
004690D7  |. 8BC3           MOV EAX,EBX
004690D9  |. E8 56F6FFFF    CALL ip.00468734  这个call中藏着让你害怕的宝宝跟进
004690DE  |. 83C6 40        ADD ESI,40
004690E1  |. 3B73 18        CMP ESI,DWORD PTR DS:[EBX+18]
004690E4  |.^75 E0          JNZ SHORT ip.004690C6
004690E6  |. 81C4 4C100000  ADD ESP,104C
004690EC  |. 5E             POP ESI
004690ED  |. 5B             POP EBX
004690EE  . C3             RETN
```

大家来看看，注册码算法呀！ 也是关键

```
00468620  /$ C740 5C 012345>MOV DWORD PTR DS:[EAX+5C],67452301
00468627  |. C740 60 89ABCD>MOV DWORD PTR DS:[EAX+60],EFCDAB89
0046862E  |. C740 64 FEDCBA>MOV DWORD PTR DS:[EAX+64],98BADCFE
00468635  |. C740 68 765432>MOV DWORD PTR DS:[EAX+68],10325476
0046863C  . C3             RETN
```

程序固定我们把这当成一个数组   //典型的`MD5`算法标志

`a[0]= 67452301`   地址为 `EAX+5C`后面`00468735`这儿变了一下 地址为: `EBX+5C`

`,a[1]= EFCDAB89,                                                EBX+60`

`a[2]= 98BADCFE,                                                EBX+64`

`a[3]= 10325476                                                   EBX+68`

`00CE3B58  00000040`

`00CE3B5C  656D7069`

`00CE3B60  425B796A`

`00CE3B64  5B5D4743`

`00CE3B68  5D474346`

`00CE3B6C  00000080`

设这个数组为`b[]`;下面函数的第三个参数 他是输入的用户名的十六进制字符

下面就要对他进行很多变换，  上面的`call`

```
00468734  /$ 53             PUSH EBX
00468735  |. 8BD8           MOV EBX,EAX
00468737  |. 8B43 5C        MOV EAX,DWORD PTR DS:[EBX+5C]
0046873A  |. 8943 6C        MOV DWORD PTR DS:[EBX+6C],EAX
0046873D  |. 8B43 60        MOV EAX,DWORD PTR DS:[EBX+60]
00468740  |. 8943 70        MOV DWORD PTR DS:[EBX+70],EAX
00468743  |. 8B43 64        MOV EAX,DWORD PTR DS:[EBX+64]
00468746  |. 8943 74        MOV DWORD PTR DS:[EBX+74],EAX
00468749  |. 8B43 68        MOV EAX,DWORD PTR DS:[EBX+68]
0046874C  |. 8943 78        MOV DWORD PTR DS:[EBX+78],EAX
0046874F  |. 8D43 64        LEA EAX,DWORD PTR DS:[EBX+64]
00468752  |. 50             PUSH EAX                                 ; /Arg5
00468753  |. 8D43 68        LEA EAX,DWORD PTR DS:[EBX+68]            ; |
00468756  |. 50             PUSH EAX                                 ; |Arg4
00468757  |. 8D43 1C        LEA EAX,DWORD PTR DS:[EBX+1C]            ; |
0046875A  |. 50             PUSH EAX                                 ; |Arg3
0046875B  |. 6A 07          PUSH 7                                   ; |Arg2 = 00000007
0046875D  |. 68 78A46AD7    PUSH D76AA478                            ; |Arg1 = D76AA478
00468762  |. 8D4B 60        LEA ECX,DWORD PTR DS:[EBX+60]            ; |
00468765  |. 8D53 5C        LEA EDX,DWORD PTR DS:[EBX+5C]            ; |
00468768  |. 8BC3           MOV EAX,EBX                              ; |
0046876A  |. E8 D1FEFFFF    CALL ip.00468640                         ; ip.00468640
```

中间省略`n`个类似的函数。。。。。。。。。。如果写注册机的话你会晕死

这个下面表示为`edx` 每个函数经过计算改变的就是这个之

```
00468F48  |. 8BC3           MOV EAX,EBX                              ; |
00468F4A  |. E8 A9F7FFFF    CALL ip.004686F8          ; ip.004686F8
```

这个`call`是计算的关键跟进

这个函数有5个参数 `i`开始时为7

第一个函数的(`a[2]`, `a[3]`,  `b[0]` , `i` , `\*\*\*\*\*\*\*\*\*` ) 其中该函数第一和第二个参数随着 `i=7`

第二个函数的(`a[1]`, `a[2]`,  `b[1]`, `i+=5`, `\*\*\*\*\*\*\*\*\*`)

第三个函数的(`a[0]` , `a[1]`,  `b[2]`, `i+=5`,`\*\*\*\*\*\*\*\*\*`)

第四个函数的(`a[3]`,  `a[0]`, `b[3]` ,`i=7`, `\*\*\*\*\*\*\*\*\*` )  `i=7`

然后开始循环了但是第五个参数每次都不一样，怎么做到这一点，有待研究！

一共循环了4次

开始做下一个循环

下一组循环第四个参数和上面的循环不同，初始`i=5` ，每次`i+=4`; 同样循环四次

然后做下一组循环 同样每次的 `b`都有改变 初始时`i=4`; `i+=7,i+=5`，`i+=7`  同样循环4次

然后做下一组循环，同样       初始时`i=6`,  `i+=4;i+=5, i+=4` 继续循环4次 注意`i`的值

(因为这些函数实在太长了我在`WORD`里贴了，足足30页)

```
00468F4F  |. 8B43 6C        MOV EAX,DWORD PTR DS:[EBX+6C]
00468F52  |. 0143 5C        ADD DWORD PTR DS:[EBX+5C],EAX
00468F55  |. 8B43 70        MOV EAX,DWORD PTR DS:[EBX+70]
00468F58  |. 0143 60        ADD DWORD PTR DS:[EBX+60],EAX
00468F5B  |. 8B43 74        MOV EAX,DWORD PTR DS:[EBX+74]
00468F5E  |. 0143 64        ADD DWORD PTR DS:[EBX+64],EAX
00468F61  |. 8B43 78        MOV EAX,DWORD PTR DS:[EBX+78]
00468F64  |. 0143 68        ADD DWORD PTR DS:[EBX+68],EAX
00468F67  |. 8D43 1C        LEA EAX,DWORD PTR DS:[EBX+1C]
00468F6A  |. 33C9           XOR ECX,ECX
00468F6C  |. BA 40000000    MOV EDX,40
00468F71  |. E8 9AA1F9FF    CALL ip.00403110
00468F76  |. 5B             POP EBX
00468F77  . C3             RETN
```

我们把上面的函数参数表示为 `K1,K2 ,K3 ,K4 ,K5`  `K1`和`K2`每次去的是数组`a`中的两个

那么剩下的两个我们表示为 `ecx`,和`edx`

上面的`call`

```
00468640  /$ 55             PUSH EBP   入栈据我估计他使用递归的很佩服作者
00468641  |. 8BEC           MOV EBP,ESP
00468643  |. 53             PUSH EBX
00468644  |. 56             PUSH ESI
00468645  |. 8BF1           MOV ESI,ECX  保存上面窜来的ecx 设为 M
00468647  |. 8BDA           MOV EBX,EDX  保存上面传来的edx 摄为N
00468649  |. 8B06           MOV EAX,DWORD PTR DS:[ESI]
  设TEMP1=（eax=esi=ecx）=M
0046864B  |. 8B55 18        MOV EDX,DWORD PTR SS:[EBP+18]
设TEMP2=K1 函数的第一个参数
0046864E  |. 2302           AND EAX,DWORD PTR DS:[EDX]
  TEMP1=TEMP2 AND TEMP2     (与运算)
00468650  |. 8B16           MOV EDX,DWORD PTR DS:[ESI]
TEMP2=ecx
00468652  |. F7D2           NOT EDX   TEMP2=~TEMP2 求反
00468654  |. 8B4D 14        MOV ECX,DWORD PTR SS:[EBP+14]   函数的第二个参数
00468657  |. 2311           AND EDX,DWORD PTR DS:[ECX]
TEMP2=TEMP2 AND K2  (与运算)
00468659  |. 0BC2           OR EAX,EDX
TEMP1=TEMP1 | TEMP2   或运算
0046865B  |. 0303           ADD EAX,DWORD PTR DS:[EBX]
TEMP1+=N
0046865D  |. 8B55 10        MOV EDX,DWORD PTR SS:[EBP+10]
这是第三个参数 对c++中函数参数传递不清楚地这下该明白了吧 K3
00468660  |. 0302           ADD EAX,DWORD PTR DS:[EDX]
TEMP1+=K3
00468662  |. 0345 08        ADD EAX,DWORD PTR SS:[EBP+8]
第五个参数K5    TEMP1+=K5;
00468665  |. 8903           MOV DWORD PTR DS:[EBX],EAX
N=TEMP1;
```
