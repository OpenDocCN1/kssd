
# 神算超人 V3.0 算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-40.htm`](https://www.pediy.com/kssd/pediy07/pediy7-40.htm)

- 【软件大小】：339 KB
- 【软件语言】：简体中文
- 【软件类别】：国产软件 / 共享版 / 理科工具
- 【应用平台】：`Win9x/NT/2000/XP`
- 【界面预览】：无
- 【加入时间】：2005-01-24 10:50:14
- 【下载地址】：天空软件站
- 【推荐等级】：★★★
- 【在线注册】：点击这里进入注册中心==>
- 【作者声明】：初学破解，仅作学习交流之用，失误之处敬请大侠赐教!
- 【破解工具】：`Ollydbg1.10`

这个程序有点意思，调试一次后再调试会出错(`Run-time error'62',Input past end of file`)，必须重新安装后方可正常调试，好像是对安装文件夹下的文件大小进行比较什么的(`regedit` 和 `cus` 文件)，不大清楚，希望大侠指点!

这个程序中用了大量的浮点运算，由于浮点知识溃乏，以下分析错误在所难免，欢迎大家指正，也请哪位大侠有浮点知识的文章或链接给我一份!

程序为 `VB` 编写，定位 `VB` 程序最好是用`\_vbastrcmp`（或者其他 `VB` 函数，如果你对 `VB` 熟的话，我是 `VB` 白痴，只会一点汇编与 `TC`），这篇文章并未能全部把算法分析清楚，因为其中有一个函数 `MSVBVM60.\_\_vbaPowerR8` 不是很明白，而且 `VB` 的参数传递基本在堆栈中，感觉与汇编中有所不同，希望大家多多指正!

如上所说，用`\_vbastrcmp` 定位到如下代码（我是第一次接触 `VB`，把反汇编的程序从头粗略看到尾才定位到如下代码）

........

```
0058DFFC   push eax
0058DFFD   push ecx
0058DFFE   call dword ptr ds:[<&MSVBVM60.\_\_vbaStrVarVal>>;  从字符串特定位置上获取其值
0058E004   push eax
0058E005   call dword ptr ds:[<&MSVBVM60.#696>]            ;  MSVBVM60.rtcCharValueBstr
0058E00B   movsx edx,ax                                   ;  上面那个 Call 看函数名猜想是按位取
字串的 ASCII 值
0058E00E   mov dword ptr ss:[ebp-11C],edx                 ;  用户名第 1 位放到 ss:[ebp-11c]中
0058E014   fild dword ptr ss:[ebp-11C]                    ;  浮点指令,装入整数到 st0=115("s"的
Hex 值)
0058E01A   fstp qword ptr ss:[ebp-124]                    ;  浮点指令,再把 st0 弹出到 ss:[ebp-
124]
0058E020   mov eax,dword ptr ss:[ebp-120]
0058E026   mov ecx,dword ptr ss:[ebp-124]
0058E02C   push eax
0058E02D   push ecx
0058E02E   call dword ptr ds:[<&MSVBVM60.\_\_vbaPowerR8>]  ;  这个函数是求平方?似曾相识,往下看
0058E034   fmul qword ptr ds:[4011C8]                      ;  此时 st0=13225=115*115,印证上面的
猜想,不知对不对,这个指令为 st0=st0*ds:[4011c8]=13225*1010323
0058E03A   mov ebx,dword ptr ds:[<&MSVBVM60.\_\_vbaStrR8>] ;  字符串转换成双精度浮点数函数
0058E040   sub esp,8
0058E043   fstsw ax                                       ;  只查到 fst dest 是保存 st0 到 dest,而
fstsw 是什么呢?
0058E045   test al,0D                                     ;  这个判断有点不明白,0D 是回车符的
Hex
0058E047   jnz 神算超人.0058E6AC
0058E04D   fstp qword ptr ss:[esp]                        ;  st0 存入 ss:[esp],再执行一次出栈操
作
0058E050   call ebx                                       ;  还记得上面吗,字串转换成双精度数;
<&MSVBVM60.\_\_vbaStrR8>
0058E052   mov edx,eax                                    ;  edx=eax="13361521675",呵呵,返回
值在 eax 中
0058E054   lea ecx,dword ptr ss:[ebp-78]
0058E057   call esi
0058E059   push eax                                       ;  大家记住一点,VB 程序是利用堆栈传
递参数的
0058E05A   call dword ptr ds:[<&MSVBVM60.#581>]           ;  MSVBVM60.rtcR8ValFromBstr
0058E060   mov edi,dword ptr ds:[<&MSVBVM60.\_\_vbaVarMove>;  MSVBVM60.\_\_vbaVarMove
0058E066   lea edx,dword ptr ss:[ebp-DC]
0058E06C   fstp qword ptr ss:[ebp-D4]                      ;  不注释了,此时 st0 中的值仍为
13361521675
0058E072   lea ecx,dword ptr ss:[ebp-70]
0058E075   mov dword ptr ss:[ebp-DC],5
0058E07F   call edi                                       ;  <&MSVBVM60.\_\_vbaVarMove>
0058E081   lea edx,dword ptr ss:[ebp-78]
0058E084   lea eax,dword ptr ss:[ebp-74]
0058E087   push edx
0058E088   push eax
0058E089   push 2                                         ;  以下都是释放内存空间
0058E08B   call dword ptr ds:[<&MSVBVM60.\_\_vbaFreeStrLis>;  MSVBVM60.\_\_vbaFreeStrList
0058E091   add esp,0C
0058E094   lea ecx,dword ptr ss:[ebp-8C]
0058E09A   call dword ptr ds:[<&MSVBVM60.\_\_vbaFreeVar>]  ;  MSVBVM60.\_\_vbaFreeVar
0058E0A0   lea edx,dword ptr ss:[ebp-CC]
0058E0A6   push 1
0058E0A8   lea eax,dword ptr ss:[ebp-8C]
0058E0AE   lea ecx,dword ptr ss:[ebp-38]
0058E0B1   push edx
0058E0B2   push eax
0058E0B3   mov dword ptr ss:[ebp-C4],ecx
0058E0B9   mov dword ptr ss:[ebp-CC],4008
0058E0C3   call dword ptr ds:[<&MSVBVM60.#619>]           ;  MSVBVM60.rtcRightCharVar
0058E0C9   push 40000000                                   ;  上面那个函数从字符串右边取相应字
符
0058E0CE   lea ecx,dword ptr ss:[ebp-8C]
0058E0D4   push 0
0058E0D6   lea edx,dword ptr ss:[ebp-74]
0058E0D9   push ecx
0058E0DA   push edx
0058E0DB   call dword ptr ds:[<&MSVBVM60.\_\_vbaStrVarVal>>;  MSVBVM60.\_\_vbaStrVarVal
0058E0E1   push eax
0058E0E2   call dword ptr ds:[<&MSVBVM60.#696>]            ;  MSVBVM60.rtcCharValueBstr
0058E0E8   movsx eax,ax                                   ;  又一轮循环开始,下面不多注释了,取
用户名第 2 个字符"h"
0058E0EB   mov dword ptr ss:[ebp-128],eax
0058E0F1   fild dword ptr ss:[ebp-128]
0058E0F7   fstp qword ptr ss:[ebp-130]
0058E0FD   mov ecx,dword ptr ss:[ebp-12C]
0058E103   mov edx,dword ptr ss:[ebp-130]
0058E109   push ecx
0058E10A   push edx
0058E10B   call dword ptr ds:[<&MSVBVM60.\_\_vbaPowerR8>]  ;  MSVBVM60.\_\_vbaPowerR8
0058E111   fmul qword ptr ds:[4011C8]                      ;  在这里 ds:[4011c8]中的值还为
1010323,莫非为固定值
0058E117   sub esp,8                                       ;  程序在 ds:[4011c8]内存段中存放了 4
个固定值,后面会还将继续看到
0058E11A   fstsw ax
0058E11C   test al,0D
0058E11E   jnz 神算超人.0058E6AC
0058E124   fstp qword ptr ss:[esp]
0058E127   call ebx
0058E129   mov edx,eax                                    ;  edx=eax="13130157708"
0058E12B   lea ecx,dword ptr ss:[ebp-78]
0058E12E   call esi                                       ;  MSVBVM60.\_\_vbaStrMove
0058E130   push eax
0058E131   call dword ptr ds:[<&MSVBVM60.#581>]           ;  MSVBVM60.rtcR8ValFromBstr
0058E137   fstp qword ptr ss:[ebp-D4]                      ;  上面那个 Call 将字符串转化成双精度
数=13130157708
0058E13D   lea edx,dword ptr ss:[ebp-DC]
0058E143   lea ecx,dword ptr ss:[ebp-24]
0058E146   mov dword ptr ss:[ebp-DC],5
0058E150   call edi                                       ;  MSVBVM60.\_\_vbaVarMove
0058E152   lea eax,dword ptr ss:[ebp-78]
0058E155   lea ecx,dword ptr ss:[ebp-74]
0058E158   push eax
0058E159   push ecx
0058E15A   push 2
0058E15C   call dword ptr ds:[<&MSVBVM60.\_\_vbaFreeStrLis>;  MSVBVM60.\_\_vbaFreeStrList
0058E162   add esp,0C
0058E165   lea ecx,dword ptr ss:[ebp-8C]
0058E16B   call dword ptr ds:[<&MSVBVM60.\_\_vbaFreeVar>]  ;  MSVBVM60.\_\_vbaFreeVar
0058E171   mov edx,dword ptr ss:[ebp-38]                 ;  取用户名到 edx 中
0058E174   push edx
0058E175   call dword ptr ds:[<&MSVBVM60.\_\_vbaLenBstr>]  ;  MSVBVM60.\_\_vbaLenBstr
0058E17B   push eax                                       ;  上面函数不用说了吧,eax=8,用户名
长度
0058E17C   call dword ptr ds:[<&MSVBVM60.\_\_vbaStrI4>]    ;  MSVBVM60.\_\_vbaStrI4
0058E182   mov edx,eax                                    ;  上面的 Call 将一个长整型数转换成字
符串,eax 中返回的为地址
0058E184   lea ecx,dword ptr ss:[ebp-74]
0058E187   call esi                                       ;  MSVBVM60.\_vbaStrMove
0058E189   push eax
0058E18A   call dword ptr ds:[<&MSVBVM60.#581>]           ;  MSVBVM60.rtcR8ValFromBstr
0058E190   fstp qword ptr ss:[ebp-C4]                      ;  此时 st0 中为 8,用户名的长度,上面的
Call 按字面理解为双精度数到字符串
0058E196   lea edx,dword ptr ss:[ebp-CC]
0058E19C   lea ecx,dword ptr ss:[ebp-34]
0058E19F   mov dword ptr ss:[ebp-CC],5
0058E1A9   call edi                                       ;  MSVBVM60.\_vbaVarMove
0058E1AB   lea ecx,dword ptr ss:[ebp-74]
0058E1AE   call dword ptr ds:[<&MSVBVM60.\_\_vbaFreeStr>]  ;  MSVBVM60.\_\_vbaFreeStr
0058E1B4   lea eax,dword ptr ss:[ebp-70]
0058E1B7   lea ecx,dword ptr ss:[ebp-24]
0058E1BA   push eax
0058E1BB   lea edx,dword ptr ss:[ebp-8C]
0058E1C1   push ecx                                       ;
0058E1C2   push edx
0058E1C3   mov dword ptr ss:[ebp-C4],1
0058E1CD   mov dword ptr ss:[ebp-CC],2
0058E1D7   call dword ptr ds:[<&MSVBVM60.\_\_vbaVarAdd>]   ;  MSVBVM60.\_\_vbaVarAdd
```



```
0058E1DD   push eax                     ;  上面的 Call 将

13130157708+13361521675=26491679383

0058E1DE   lea eax,dword ptr ss:[ebp-34]

0058E1E1   lea ecx,dword ptr ss:[ebp-CC]

0058E1E7   push eax

0058E1E8   lea edx,dword ptr ss:[ebp-34]

0058E1EB   push ecx

0058E1EC   lea eax,dword ptr ss:[ebp-9C]

0058E1F2   push edx

0058E1F3   push eax

0058E1F4   call dword ptr ds:[<&MSVBVM60.__vbaVarDiv>]   ;  MSVBVM60.__vbaVarDiv

0058E1FA   lea ecx,dword ptr ss:[ebp-AC]                 ;  两数相除:1/8=0.125,8 为用户名位数

0058E200   push eax

0058E201   push ecx

0058E202   call dword ptr ds:[<&MSVBVM60.__vbaVarPow>]   ;  MSVBVM60.__vbaVarPow

0058E208   lea edx,dword ptr ss:[ebp-BC]                 ;  上面的值为 0.125,按前面这个函数为

求平方,实际操作结果却不是

0058E20E   push eax

0058E20F   push edx

0058E210   call dword ptr ds:[<&MSVBVM60.__vbaVarMul>]   ;  MSVBVM60.__vbaVarMul

0058E216   mov edx,eax                                   ;  上面的 Call 实现

26491679383*1.296839554651010=34355457693.00705

0058E218   lea ecx,dword ptr ss:[ebp-50]

0058E21B   call edi                      ;  MSVBVM60.__vbaVarMove

0058E21D   lea ecx,dword ptr ss:[ebp-8C]

0058E223   call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>]  ;  MSVBVM60.__vbaFreeVar

0058E229   fld qword ptr ds:[4011C0]                    ;  ds:[4011c0]中为 29(定值),存入 st0

中

0058E22F   cmp dword ptr ds:[5C8000],0                  ;

0058E236   jnz short 神算超人.0058E240

0058E238   fdiv qword ptr ds:[4011B8]                   ;  ds:[4011b8]中为 28,st0=st0/ds:

[4011b8]=1.0357142857142857140

0058E23E   jmp short 神算超人.0058E251

0058E240   push dword ptr ds:[4011BC]

0058E246   push dword ptr ds:[4011B8]

0058E24C   call <jmp.&MSVBVM60._adj_fdiv_m64>

0058E251   mov dword ptr ss:[ebp-CC],5

0058E25B   fstp qword ptr ss:[ebp-C4]

0058E261   fstsw ax

0058E263   test al,0D

0058E265   jnz 神算超人.0058E6AC

0058E26B   lea eax,dword ptr ss:[ebp-50]

0058E26E   lea ecx,dword ptr ss:[ebp-CC]

0058E274   push eax

0058E275   lea edx,dword ptr ss:[ebp-8C]

0058E27B   push ecx

0058E27C   push edx                      ;  又来了,下面这个 Call 确实不大明白,

也懒得往里再跟了

0058E27D   call dword ptr ds:[<&MSVBVM60.__vbaVarPow>]   ;  MSVBVM60.__vbaVarPow

0058E283   push eax

0058E284   call dword ptr ds:[<&MSVBVM60.__vbaR8Var>]    ;  MSVBVM60.__vbaR8Var

0058E28A   sub esp,8                     ;  上面的 Call 将一个字符串转换成双精

度浮点数形式

0058E28D   fstp qword ptr ss:[esp]                 ;  结果为 81711145863.91840

0058E290   call dword ptr ds:[<&MSVBVM60.#614>]       ;  MSVBVM60.rtcSqr 求平方根

=285851.615115112448

0058E296   call dword ptr ds:[<&MSVBVM60.__vbaFPInt>]    ;  MSVBVM60.__vbaFPInt 浮点数转换成

整形数=285851

0058E29C   sub esp,8

0058E29F   fstp qword ptr ss:[esp]                 ;  结果存入堆栈

0058E2A2   call ebx                      ;  MSVBVM60.__vbaStrR8 浮点数转换成

字符串形式

0058E2A4   mov edx,eax

0058E2A6   lea ecx,dword ptr ss:[ebp-74]

0058E2A9   call esi                      ;  MSVBVM60.__vbaStrMove

0058E2AB   push eax                      ;  eax="285851",上面求平方根后的结

果

0058E2AC   call dword ptr ds:[<&MSVBVM60.#581>]       ;  MSVBVM60.rtcR8ValFromBstr

0058E2B2   fadd qword ptr ds:[4011B0]                   ;  ds:[4011b0]中为 6060218610(定值),

与-st0 相加=6050218610 存入 st0 中

0058E2B8   lea edx,dword ptr ss:[ebp-DC]

0058E2BE   lea ecx,dword ptr ss:[ebp-60]

0058E2C1   mov dword ptr ss:[ebp-DC],5

0058E2CB   fstp qword ptr ss:[ebp-D4]

0058E2D1   fstsw ax

0058E2D3   test al,0D

0058E2D5   jnz 神算超人.0058E6AC

0058E2DB   call edi                      ;  MSVBVM60.__vbaVarMove

0058E2DD   lea ecx,dword ptr ss:[ebp-74]

0058E2E0   call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]  ;  MSVBVM60.__vbaFreeStr

0058E2E6   lea eax,dword ptr ss:[ebp-60]

0058E2E9   push eax

0058E2EA   call dword ptr ds:[<&MSVBVM60.__vbaStrVarCopy>;  MSVBVM60.__vbaStrVarCopy

0058E2F0   mov edx,eax                   ;  edx="6050504461",累加和转换成字

符串形式,真码!

0058E2F2   lea ecx,dword ptr ss:[ebp-3C]

0058E2F5   call esi                      ;  MSVBVM60.__vbaStrMove

0058E2F7   mov edi,dword ptr ss:[ebp+8]

0058E2FA   xor ebx,ebx

0058E2FC   jmp 神算超人.0058E3E5

0058E301   mov ecx,0A

0058E306   mov eax,80020004

0058E30B   mov dword ptr ss:[ebp-BC],ecx

0058E311   mov dword ptr ss:[ebp-AC],ecx

0058E317   mov dword ptr ss:[ebp-9C],ecx

0058E31D   lea edx,dword ptr ss:[ebp-CC]

0058E323   lea ecx,dword ptr ss:[ebp-8C]

0058E329   mov dword ptr ss:[ebp-B4],eax

0058E32F   mov dword ptr ss:[ebp-A4],eax

0058E335   mov dword ptr ss:[ebp-94],eax

0058E33B   mov dword ptr ss:[ebp-C4],神算超人.005719DC

0058E345   mov dword ptr ss:[ebp-CC],8

0058E34F   call dword ptr ds:[<&MSVBVM60.__vbaVarDup>]   ;  MSVBVM60.__vbaVarDup

0058E355   lea ecx,dword ptr ss:[ebp-BC]

0058E35B   lea edx,dword ptr ss:[ebp-AC]

0058E361   push ecx

0058E362   lea eax,dword ptr ss:[ebp-9C]

0058E368   push edx

0058E369   push eax

0058E36A   lea ecx,dword ptr ss:[ebp-8C]

0058E370   push 10

0058E372   push ecx

0058E373   call dword ptr ds:[<&MSVBVM60.#595>]         ;  MSVBVM60.rtcMsgBox

0058E379   lea edx,dword ptr ss:[ebp-BC]

0058E37F   lea eax,dword ptr ss:[ebp-AC]

0058E385   push edx

0058E386   lea ecx,dword ptr ss:[ebp-9C]

0058E38C   push eax

0058E38D   lea edx,dword ptr ss:[ebp-8C]

0058E393   push ecx

0058E394   push edx

0058E395   push 4

0058E397   call dword ptr ds:[<&MSVBVM60.__vbaFreeVarLis>;  MSVBVM60.__vbaFreeVarList

0058E39D   mov eax,dword ptr ds:[edi]

0058E39F   add esp,14

0058E3A2   push edi

0058E3A3   call dword ptr ds:[eax+30C]

0058E3A9   lea ecx,dword ptr ss:[ebp-7C]

0058E3AC   push eax

0058E3AD   push ecx

0058E3AE   call dword ptr ds:[<&MSVBVM60.__vbaObjSet>]   ;  MSVBVM60.__vbaObjSet

0058E3B4   mov esi,eax

0058E3B6   push 神算超人.005719DC

0058E3BB   push esi

0058E3BC   mov edx,dword ptr ds:[esi]

0058E3BE   call dword ptr ds:[edx+A4]

0058E3C4   cmp eax,ebx

0058E3C6   fclex

0058E3C8   jge short 神算超人.0058E3DC

0058E3CA   push 0A4

0058E3CF   push 神算超人.005719C0

0058E3D4   push esi

0058E3D5   push eax

0058E3D6   call dword ptr ds:[<&MSVBVM60.__vbaHresultChe>;  MSVBVM60.__vbaHresultCheckObj

0058E3DC   lea ecx,dword ptr ss:[ebp-7C]

0058E3DF   call dword ptr ds:[<&MSVBVM60.__vbaFreeObj>]  ;  MSVBVM60.__vbaFreeObj

0058E3E5   mov eax,dword ptr ds:[edi]

0058E3E7   push edi

0058E3E8   call dword ptr ds:[eax+30C]                   ;  这个 Call 有点意思,弄不明白

0058E3EE   lea ecx,dword ptr ss:[ebp-7C]

0058E3F1   push eax

0058E3F2   push ecx

0058E3F3   call dword ptr ds:[<&MSVBVM60.__vbaObjSet>]   ;  MSVBVM60.__vbaObjSet

0058E3F9   mov esi,eax

0058E3FB   lea eax,dword ptr ss:[ebp-74]

0058E3FE   push eax

0058E3FF   push esi

0058E400   mov edx,dword ptr ds:[esi]

0058E402   call dword ptr ds:[edx+A0]

0058E408   cmp eax,ebx

0058E40A   fclex                         ;  清除异常

0058E40C   jge short 神算超人.0058E420

0058E40E   push 0A0

0058E413   push 神算超人.005719C0

0058E418   push esi

0058E419   push eax

0058E41A   call dword ptr ds:[<&MSVBVM60.__vbaHresultChe>;  MSVBVM60.__vbaHresultCheckObj

0058E420   mov ecx,dword ptr ss:[ebp-74]                 ;  这里可看到假码

0058E423   mov edx,dword ptr ss:[ebp-3C]                 ;  这里存放着计算出来的真码

0058E426   push ecx                      ;  后面是经典句式,不用多说了吧

0058E427   push edx
```



```
0058E428   call dword ptr ds:[<&MSVBVM60.__vbaStrCmp>]     ;  MSVBVM60.__vbaStrCmp  
0058E42E   mov esi,eax  
0058E430   lea ecx,dword ptr ss:[ebp-74]  
0058E433   neg esi  
0058E435   sbb esi,esi  
0058E437   inc esi  
0058E438   neg esi  
0058E43A   call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]    ;  MSVBVM60.__vbaFreeStr  
0058E440   lea ecx,dword ptr ss:[ebp-7C]  
0058E443   call dword ptr ds:[<&MSVBVM60.__vbaFreeObj>]    ;  MSVBVM60.__vbaFreeObj  
0058E449   mov eax,80020004  
0058E44E   mov ecx,0A  
0058E453   cmp si,bx  
0058E456   mov dword ptr ss:[ebp-B4],eax  
0058E45C   mov dword ptr ss:[ebp-BC],ecx  
0058E462   mov dword ptr ss:[ebp-A4],eax  
0058E468   mov dword ptr ss:[ebp-AC],ecx  
0058E46E   je 神算超人.0058E56F  
0058E474   mov esi,dword ptr ds:[<&MSVBVM60.__vbaVarDup>]  ;  MSVBVM60.__vbaVarDup  
0058E47A   lea edx,dword ptr ss:[ebp-DC]  
0058E480   lea ecx,dword ptr ss:[ebp-9C]  
0058E486   mov dword ptr ss:[ebp-D4],神算超人.00571A20  
0058E490   mov dword ptr ss:[ebp-DC],8  
0058E49A   call esi                                          ;  <&MSVBVM60.__vbaVarDup>  
0058E49C   lea edx,dword ptr ss:[ebp-CC]  
0058E4A2   lea ecx,dword ptr ss:[ebp-8C]  
0058E4A8   mov dword ptr ss:[ebp-C4],神算超人.005719F0  
0058E4B2   mov dword ptr ss:[ebp-CC],8  
0058E4BC   call esi  
0058E4BE   lea eax,dword ptr ss:[ebp-BC]  
0058E4C4   lea ecx,dword ptr ss:[ebp-AC]  
0058E4CA   push eax  
0058E4CB   lea edx,dword ptr ss:[ebp-9C]  
0058E4D1   push ecx  
0058E4D2   push edx  
0058E4D3   lea eax,dword ptr ss:[ebp-8C]  
0058E4D9   push 40  
0058E4DB   push eax  
0058E4DC   call dword ptr ds:[<&MSVBVM60.#595>]             ;  MSVBVM60.rtcMsgBox  
0058E4E2   lea ecx,dword ptr ss:[ebp-BC]  
0058E4E8   lea edx,dword ptr ss:[ebp-AC]  
0058E4EE   push ecx  
0058E4EF   lea eax,dword ptr ss:[ebp-9C]  
0058E4F5   push edx  
0058E4F6   lea ecx,dword ptr ss:[ebp-8C]  
0058E4FC   push eax  
0058E4FD   push ecx  
0058E4FE   push 4  
0058E500   call dword ptr ds:[<&MSVBVM60.__vbaFreeVarList>>;  MSVBVM60.__vbaFreeVarList  
0058E506   mov eax,dword ptr ds:[5C8958]  
0058E50B   add esp,14  
0058E50E   cmp eax,ebx  
0058E510   jnz short 神算超人.0058E522  
0058E512   push 神算超人.005C8958  
0058E517   push 神算超人.00571538  
0058E51C   call dword ptr ds:[<&MSVBVM60.__vbaNew2>]        ;  MSVBVM60.__vbaNew2  
0058E522   mov esi,dword ptr ds:[5C8958]  
0058E528   lea eax,dword ptr ss:[ebp-7C]  
0058E52B   push edi  
0058E52C   push eax  
0058E52D   mov edx,dword ptr ds:[esi]  
0058E52F   mov dword ptr ss:[ebp-134],edx  
0058E535   call dword ptr ds:[<&MSVBVM60.__vbaObjSetAddref>;  MSVBVM60.__vbaObjSetAddref  
0058E53B   mov ecx,dword ptr ss:[ebp-134]  
0058E541   push eax  
0058E542   push esi  
0058E543   call dword ptr ds:[ecx+10]  
0058E546   cmp eax,ebx  
0058E548   fclex  
0058E54A   jge short 神算超人.0058E55B  
0058E54C   push 10  
0058E54E   push 神算超人.00571528  
0058E553   push esi  
0058E554   push eax  
0058E555   call dword ptr ds:[<&MSVBVM60.__vbaHresultCheck>;  MSVBVM60.__vbaHresultCheckObj  
0058E55B   lea ecx,dword ptr ss:[ebp-7C]  
0058E55E   call dword ptr ds:[<&MSVBVM60.__vbaFreeObj>]    ;  MSVBVM60.__vbaFreeObj  
0058E564   call dword ptr ds:[<&MSVBVM60.__vbaEnd>]        ;  MSVBVM60.__vbaEnd  
0058E56A   jmp 神算超人.0058E601  
0058E56F   mov esi,dword ptr ds:[<&MSVBVM60.__vbaVarDup>]  ;  MSVBVM60.__vbaVarDup  
0058E575   mov edi,8  
0058E57A   lea edx,dword ptr ss:[ebp-DC]  
0058E580   lea ecx,dword ptr ss:[ebp-9C]  
......
```

算法虽然未能详细分析出来，但基本框架我们知道了，注册码只与用户名前两位及用户名长度有关，本想把程序自显示注册码，可本人对于`VB`及其堆栈调用确实很不清楚，只好作罢，有知道的大侠还请给小弟指点一二。

给出一个可用注册码：用户名`sharpair`，注册码`6050504461`

欢迎发信到`sharpair@163.com`交流，QQ:`402800474`

`4dd466d0e1e0cc123afbe504da2462ca`
