
# XXX 宝典

> 原文：[`www.pediy.com/kssd/pediy06/pediy6263.htm`](https://www.pediy.com/kssd/pediy06/pediy6263.htm)

XXX 宝典算法初探

说明：这个软件加了 `UPX` 的壳，但写这篇文章我是在没有脱壳的情况下破解的，用的是 `OD`，这个思路还没有理清，把这个方法成熟后，我想写一些关于 `OD` 的系列破解教学。为保护国产软件隐去相关信息。

破解作者：`yzez[DFCG]`

破解工具：`OLLYDBG1.09`

## 1. 过程分析

运行要破解的程序，然后用 `OD` 的附加功能载入，在关键点：`0051B93A` 设置好断点，这个关键点如何找到，我用的是万能断点：`hmemcpy`，这个方法在这篇破文中我不想总结，等下一篇我想专门写几篇关于 `OD` 的万能断点的设置，以及不脱壳 `OD` 的万能断点的设置方法，这是后话，下面看过程吧

```asm
0051B930    PUSH    EAX
0051B931    LEA     EDX, DWORD PTR SS:[EBP-8]
0051B934    MOV     EAX, DWORD PTR DS:[EBX+2FC]
0051B93A    CALL    Zgrbd1_2.00450168 ; 我们在此 CALL 设置好断点!程序中断后按 F8 往下
0051B93F    MOV     EDX, DWORD PTR SS:[EBP-8] ; 输入的用户名移入 EDX
0051B942    MOV     ECX, Zgrbd1_2.0051BBF8 ; 内置的字符串 "agenius" 入 ECX
0051B947    MOV     EAX, Zgrbd1_2.0051BC08 ; 内置的字符串 "fuck" 入 EAX
0051B94C    CALL    Zgrbd1_2.00500BDC ; 算法 CALL(1), 按 F7 跟进!
0051B951    MOV     EAX, DWORD PTR SS:[EBP-4] ; 把运算的结果 C0FBB849 大写转换成小写,即 c0fbb849 送入 EAX
0051B954    PUSH    EAX
0051B955    LEA     EDX, DWORD PTR SS:[EBP-C]
0051B958    MOV     EAX, DWORD PTR DS:[EBX+300]
0051B95E    CALL    Zgrbd1_2.00450168
0051B963    MOV     EDX, DWORD PTR SS:[EBP-C] ; 输入的第一组试验码: 12345678 入 EDX
0051B966    POP     EAX
0051B967    CALL    Zgrbd1_2.00404ED0 ; 比较 CALL
0051B96C    JNZ     Zgrbd1_2.0051BB93 ; 不相等就跳! 一跳就失败, 不跳的话就往下, 比较第二组!
0051B972    PUSH    Zgrbd1_2.0051BBEC
0051B977    LEA     EAX, DWORD PTR SS:[EBP-10]
0051B97A    PUSH    EAX
0051B97B    LEA     EDX, DWORD PTR SS:[EBP-14]
0051B97E    MOV     EAX, DWORD PTR DS:[EBX+320]
0051B984    CALL    Zgrbd1_2.00450168
0051B989    MOV     EDX, DWORD PTR SS:[EBP-14] ; 机器码: 562207418120 入 EDX
0051B98C    MOV     ECX, Zgrbd1_2.0051BBF8 ; 内置的字符串 "agenius" 入 ECX
0051B991    MOV     EAX, Zgrbd1_2.0051BC08 ; 内置的字符串 "fuck" 入 EAX
0051B996    CALL    Zgrbd1_2.00500BDC ; 算法 CALL(3), 按 F7 跟进, 因为调用同一子程序, 代码见后!
0051B99B    MOV     EAX, DWORD PTR SS:[EBP-10] ; 计算的结果 82a5446b 入 EAX
0051B99E    PUSH    EAX
0051B99F    LEA     EDX, DWORD PTR SS:[EBP-18]
0051B9A2    MOV     EAX, DWORD PTR DS:[EBX+304]
0051B9A8    CALL    Zgrbd1_2.00450168
0051B9AD    MOV     EDX, DWORD PTR SS:[EBP-18] ; 第二组试验码 87654321 入 EDX
0051B9B0    POP     EAX
0051B9B1    CALL    Zgrbd1_2.00404ED0 ; 比较 CALL
0051B9B6    JNZ     Zgrbd1_2.0051BB93 ; 不相等就跳, 一跳就失败
0051B9BC    PUSH    40
0051B9BE    PUSH    Zgrbd1_2.0051BC10
0051B9C3    PUSH    Zgrbd1_2.0051BC1C
0051B9C8    MOV     EAX, EBX
0051B9CA    CALL    Zgrbd1_2.004568C4
0051B9CF    50          PUSH    EAX
0051B9D0    CALL    Zgrbd1_2.00407D14 ; 到这里恭喜你成功了!
0051B9D5    MOV     EAX, DWORD PTR DS:[54F00C]
0051B9DA    MOV     EAX, DWORD PTR DS:[EAX]
0051B9DC    MOV     EDX, Zgrbd1_2.0051BC4C
0051B9E1    CALL    Zgrbd1_2.00450198
```

## 2. 算法 CALL(1)

```asm
; *** 算法 CALL(1) ***
00500BDC    PUSH    EBP ; 跟进算法 CALL 后我们停在这里!
; *** 省略部分代码! ***
00500C32    LEA     EAX, DWORD PTR SS:[EBP-14]
00500C35    PUSH    EAX
00500C36    MOV     ECX, DWORD PTR SS:[EBP-C] ; 内置的字符串 "agenius" 入 ECX
00500C39    MOV     EDX, DWORD PTR SS:[EBP-8] ; 输入的用户名 yzez[DFCG] 移入 EDX
00500C3C    MOV     EAX, DWORD PTR SS:[EBP-4] ; 内置的字符串 "fuck" 入 EAX
00500C3F    CALL    Zgrbd1_2.00500984 ; 此 CALL 对上述字符串处理, 使之变成 "fuckyzez[DFCG]agenius2\4\lvdn+YF"
; *** 代码太长, 我就不列了, 有兴趣的朋友自己跟一下看吧! ***
00500C44    PUSH    DWORD PTR SS:[EBP-14] ; 保存字串 "\4\lvdn+YF"
00500C47    LEA     EAX, DWORD PTR SS:[EBP-10] ; 赋地址值!
00500C4A    MOV     EDX, 5 ; 常数 5 送入 EDX, 干什么用?
00500C4F    CALL    Zgrbd1_2.00404E4C
00500C54    MOV     EDX, DWORD PTR SS:[EBP+8]
00500C57    MOV     EAX, DWORD PTR SS:[EBP-10] ; 变形后的字串: fuckyzez[DFCG]agenius2\4\lvdn+YF" 入 EAX
00500C5A    CALL    Zgrbd1_2.00500C90 ; 算法 CALL(2) 按 F7 跟进! 代码直接在下面给出!
```

## 3. 算法 CALL(2) 跟进

```asm
; *** 算法 CALL(2) 跟进: ***
00500C90    PUSH    EBP ; 来到这里!
00500C91    MOV     EBP, ESP
; *** 省略部分代码! ***
00500CA3    MOV     EAX, DWORD PTR SS:[EBP-4] ; 把变形的字串放 EAX
00500CA6    CALL    Zgrbd1_2.00404F74
00500CAB    XOR     EAX, EAX
00500CAD    PUSH    EBP
00500CAE    PUSH    Zgrbd1_2.00500D2F
00500CB3    PUSH    DWORD PTR FS:[EAX]
00500CB6    MOV     DWORD PTR FS:[EAX], ESP
00500CB9    XOR     EBX, EBX
00500CBB    MOV     EAX, DWORD PTR SS:[EBP-4] ; 把变形的字串放 EAX
00500CBE    CALL    Zgrbd1_2.00404D8C
00500CC3    TEST    EAX, EAX ; 测试 EAX 的位数是否是 20(十进制值是 32)
00500CC5    JLE     SHORT Zgrbd1_2.00500CF3 ; 小于就跳走!
00500CC7    MOV     ESI, 1 ; 赋 ESI 的初始值 1, ESI 做计数器!
00500CCC    MOV     EDX, DWORD PTR SS:[EBP-4] ; 变形字符串送 EDX!
00500CCF    MOV     DL, BYTE PTR DS:[EDX+ESI-1] ; 取第一位字符: f 送 DL, f 的 ASCII 码值是 66
00500CD3    XOR     DL, BL ; EBX 的初始值是 0, DL=DL^BL=66
00500CD5    AND     EDX, 0FF ; EDX=EDX AND 0FF=66
00500CDB    EDX, DWORD PTR DS:[EDX*4+54D698] ; 把地址 EDX*4+54D698 的值 A4D1C46D 移入 EDX
00500CE2    SHR     EBX, 8 ; EBX 右移 8 位
00500CE5    AND     EBX, 0FFFFFF ; EBX=EBX AND 0FFFFFF=00000000
00500CEB    XOR     EDX, EBX ; EDX=EDX^EBX=A4D1C46D⁰⁰⁰⁰⁰⁰⁰⁰=A4D1C46D
00500CED    MOV     EBX, EDX ; 把这个值移入 EBX 中
00500CEF    INC     ESI ; ESI 加 1
00500CF0    DEC     EAX ; EAX 减 1
00500CF1    ^ JNZ     SHORT Zgrbd1_2.00500CCC ; 循环! 循环的次数是 20(即 32 次), 最后的值是: C0FBB849
00500CF3    MOV     EAX, EBX ; 把循环后的值: C0FBB849 送入 EAX
00500CF5    XOR     EDX, EDX
00500CF7    PUSH    EDX
00500CF8    PUSH    EAX
00500CF9    LEA     EDX, DWORD PTR SS:[EBP-C]
00500CFC    MOV     EAX, 8
00500D01    CALL    Zgrbd1_2.00409738
00500D06    MOV     EAX, DWORD PTR SS:[EBP-C]
00500D09    MOV     EDX, DWORD PTR SS:[EBP-8]
00500D0C    CALL    Zgrbd1_2.0040921C
00500D11    XOR     EAX, EAX
00500D13    POP     EDX
00500D14    POP     ECX
```



```
00500D15    POP     ECX                             

00500D16    MOV     DWORD PTR FS:[EAX], EDX 

00500D19    PUSH    Zgrbd1_2.00500D36 

00500D1E    LEA     EAX, DWORD PTR SS:[EBP-C] 

00500D21    CALL    Zgrbd1_2.00404AC4 

00500D26    LEA     EAX, DWORD PTR SS:[EBP-4] 

00500D29    CALL    Zgrbd1_2.00404AC4 

00500D2E    RETN 

=========================================================================================== 

00500C5F    XOR     EAX, EAX                             

00500C61    POP     EDX                              

00500C62    POP     ECX                             

00500C63    POP     ECX                              

00500C64    MOV     DWORD PTR FS:[EAX], EDX 

00500C67    PUSH    Zgrbd1_2.00500C89 

00500C6C    LEA     EAX, DWORD PTR SS:[EBP-14] 

00500C6F    MOV     EDX, 5 

00500C74    CALL    Zgrbd1_2.00404AE8 

00500C79    LEA     EAX, DWORD PTR SS:[EBP+C] 

00500C7C    CALL    Zgrbd1_2.00404AC4 

00500C81    RETN 

```

算法我不想总结了,过程很明白,而且是明码比较,自己跟一跟看一看吧!

`4dd466d0e1e0cc123afbe504da2462ca`
