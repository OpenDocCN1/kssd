
# 易经八卦彩票占卜程序.V6.3

> 原文：[`www.pediy.com/kssd/pediy03/forum876.htm`](https://www.pediy.com/kssd/pediy03/forum876.htm)

易经八卦彩票占卜程序.V6.3 暴力及注册码破解

- 作者：冷雨飘心 `humewen@263.net`
- 破解时间：2001-4-23
- 破解工具：`TRW2000 V1.23`, `Hiew V6.40`, `W32dasm` 黄金版
- 下载地址: `http://member.netease.com/~tr/`
- 说明: 彩票软件,比昨天那个丁氏软件强点,至少用了八卦占卜,还算有创意,另外至少还可以写出注册程序,价格也便宜

## 一、暴力破解方法

使用 `W32dasm` 可见:

```
:004067E1 8B8310030000             mov eax, dword ptr [ebx+00000310]
:004067E7 E860B1FFFF               call 0040194C
:004067EC 84C0                     test al, al
:004067EE 7428                     je 00406818      <---------------- 不让它跳即可！
                                  ^^^^改为 9090
:004067F0 A158525100               mov eax, dword ptr [00515258]
:004067F5 6A40                     push 00000040
* Possible StringData Ref from Data Obj ->"提示"
                                      |
:004067F7 B918AA5000               mov ecx, 0050AA18
* Possible StringData Ref from Data Obj ->"恭喜！注册成功！"
                                      |
:004067FC BA06AA5000               mov edx, 0050AA06
:00406801 8B00                     mov eax, dword ptr [eax]
:00406803 E830F60F00               call 00505E38
:00406808 8BC3                     mov eax, ebx
:0040680A E831000000               call 00406840
:0040680F 8BC3                     mov eax, ebx
:00406811 E8AE010000               call 004069C4
:00406816 EB18                     jmp 00406830
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004067EE(C)
|
:00406818 A158525100               mov eax, dword ptr [00515258]
:0040681D 6A10                     push 00000010
* Possible StringData Ref from Data Obj ->"错误"
                                      |
:0040681F B947AA5000               mov ecx, 0050AA47
* Possible StringData Ref from Data Obj ->"对不起，用户名和注册码不匹配。注册失败！"
                                      |
:00406824 BA1DAA5000               mov edx, 0050AA1D
:00406829 8B00                     mov eax, dword ptr [eax]
:0040682B E808F60F00               call 00505E38
```

但修改后，每次运行都要注册一次,很烦,估计还有陷阱,每次启动时还要检查,故祭出 `trw2000` 动态跟踪,发现实际上是调用 `yijing.dll` 进行注册码计算追踪过程:

```
:004019FB FFD3                     call ebx//调用 yijing.dll 的 wackywooky 函数
:004019FD 85C0                     test eax, eax
:004019FF 0F95C0                   setne al//置标志
:00401A02 83E001                   and eax, 00000001
:00401A05 8BD8                     mov ebx, eax
```

//因此:
4019ff `setnz al` => `setz al`
`hiew` `0fff:0f95c0` -> `0f94c0`

注册码随便输,只要不是正确的,即可,(正确的概率象中 5,000,000$$$嘿嘿)

## 二、注册码查找

在 `:004019FB FFD3  call ebx` 处追入

`GO`, `to here`:

```
0167:013012AF  CALL     01302738
0167:013012B4  POP      ECX
0167:013012B5  CMP      ESI,EAX
0167:013012B7  JZ       013012C0
```

//比较用户名和序列号的个数若相等,则进一步比较,否则跳走,你就什么也看不到了!退出填入相同个数的用户名和密码

我的是用户名:`812153`
`s:` `545454`

以下是注册码生成全过程,由于没有时间,所以不写注册机,`go on`

```
0167:013012B9  XOR      EAX,EAX
0167:013012BB  JMP      0130138A
0167:013012C0  PUSH     EBX
0167:013012C1  CALL     01302738
0167:013012C6  POP      ECX
0167:013012C7  MOV      [EBP-08],EAX
0167:013012CA  MOV      EDX,[EBP-08]
0167:013012CD  INC      EDX
0167:013012CE  PUSH     EDX
0167:013012CF  CALL     013015C8
0167:013012D4  POP      ECX
0167:013012D5  MOV      [EBP-04],EAX
0167:013012D8  MOV      EAX,[EBP+0C]
0167:013012DB  MOV      ECX,[EBP-04]
0167:013012DE  MOV      EDI,EAX
0167:013012E0  XOR      EAX,EAX
0167:013012E2  MOV      ESI,ECX
0167:013012E4  OR       ECX,BYTE -01
0167:013012E7  REPNE SCASB
0167:013012E9  NOT      ECX
0167:013012EB  SUB      EDI,ECX
0167:013012ED  MOV      EDX,ECX
0167:013012EF  XCHG     ESI,EDI
0167:013012F1  SHR      ECX,02
0167:013012F4  MOV      EAX,EDI
0167:013012F6  REP MOVSD
0167:013012F8  MOV      ECX,EDX
0167:013012FA  AND      ECX,BYTE +03
0167:013012FD  REP MOVSB
0167:013012FF  MOV      EAX,[EBP-04]
0167:01301302  XOR      ECX,ECX
0167:01301304  MOV      EDX,EAX
0167:01301306  MOV      EAX,EBX
0167:01301308  MOV      [EBP-14],EDX
0167:0130130B  MOV      [EBP-10],EAX
0167:0130130E  MOV      EDX,[EBP-08]
0167:01301311  CMP      ECX,EDX
0167:01301313  JNL      01301360           //开始注意了
0167:01301315  MOV      EAX,[EBP-10]
0167:01301318  LEA      EDI,[ECX+01]
0167:0130131B  MOV      BL,[EAX]
0167:0130131D  MOVSX    ESI,BL
0167:01301320  MOV      EAX,ESI
0167:01301322  MOV      EBX,4B
0167:01301327  IMUL     ESI
0167:01301329  IMUL     ESI
0167:0130132B  LEA      EDX,[ECX+01]
0167:0130132E  IMUL     EDX,EDI
0167:01301331  SUB      EAX,EDX
0167:01301333  LEA      EDX,[ECX+01]
0167:01301336  IMUL     EDX,ESI
0167:01301339  SUB      EAX,EDX
0167:0130133B  INC      ECX
0167:0130133C  MOV      [EBP-0C],EAX
0167:0130133F  MOV      EAX,[EBP-0C]
0167:01301342  CDQ      
0167:01301343  XOR      EAX,EDX
0167:01301345  SUB      EAX,EDX
0167:01301347  CDQ      
0167:01301348  IDIV     EBX
0167:0130134A  MOV      EAX,EDX
0167:0130134C  MOV      EDX,[EBP-14]
0167:0130134F  ADD      AL,30
0167:01301351  MOV      [EDX],AL
0167:01301353  INC      DWORD [EBP-14]
0167:01301356  INC      DWORD [EBP-10]
0167:01301359  MOV      EAX,[EBP-08]
0167:0130135C  CMP      ECX,EAX      //    
0167:0130135E  JL       01301315     //比较循环是否结束循环,计算    
0167:01301360  MOV      EAX,[EBP-04]
0167:01301363  MOV      EDX,[EBP+0C]
0167:01301366  MOV      CL,[EAX]----//[EAX],指向注册码
                                     //d eax ----->kFYn<9,很奇怪,但就是他!
0167:01301368  CMP      CL,[EDX]
0167:0130136A  JNZ      01301388
```

可以 `bpx 01301368 do "d eax"` 取得注册码,不用写注册机,注意如果一次注册成功,下次将无法注册,是因为在注册表里作了手脚:查找 `yijing`,你发现什么,不要犹豫,删之!(用户名和密码要相同长度)

推荐注册码: 用户名:`1`
序列号:`z`

附诗两句,聊表心情:

抽刀断水水更流，举杯销愁愁更愁。
人生在世不称意，明朝散发弄扁舟。  `4dd466d0e1e0cc123afbe504da2462ca`
