
# Hickwall 啄木鸟 算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-118.htm`](https://www.pediy.com/kssd/pediy07/pediy7-118.htm)

Hickwall 啄木鸟 可以狙击所有恶意代码软件调试！

---

在无法确定目标程序是否含有恶意代码时，应在调试库中选择所有的函数。尤其要注意程序是否将数据写入硬盘，是否修改注册表中的关键字段，是否通过网络发送了数据。

如果无法确认是否安全，可以使用"强迫失败"使这个函数无法完成。

下载：

`http://www.goldlimit.com/product/hickwall10cn.exe`

```asm
00409E5C  |> /8B55 EC        MOV EDX,DWORD PTR SS:[EBP-14]
00409E5F  |. |83C2 01        |ADD EDX,1
00409E62  |. |8955 EC        |MOV DWORD PTR SS:[EBP-14],EDX
00409E65  |> |8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]
00409E68  |. |3B45 F4        |CMP EAX,DWORD PTR SS:[EBP-C]
00409E6B  |. |0F8D DF000000 |JGE 00409F50
00409E71  |. |8B4D F8        |MOV ECX,DWORD PTR SS:[EBP-8]
00409E74  |. |83C1 01        |ADD ECX,1
00409E77  |. |894D F8        |MOV DWORD PTR SS:[EBP-8],ECX
00409E7A  |. |8B45 EC        |MOV EAX,DWORD PTR SS:[EBP-14]
00409E7D  |. |99             |CDQ
00409E7E  |. |B9 03000000    |MOV ECX,3
00409E83  |. |F7F9           |IDIV ECX
00409E85  |. |8955 F0        |MOV DWORD PTR SS:[EBP-10],EDX
00409E88  |. |837D F0 00     |CMP DWORD PTR SS:[EBP-10],0
00409E8C  |. |75 3B          |JNZ SHORT 00409EC9
00409E8E  |. |8B55 08        |MOV EDX,DWORD PTR SS:[EBP+8]
00409E91  |. |0355 EC        |ADD EDX,DWORD PTR SS:[EBP-14]
00409E94  |. |0FBE02         |MOVSX EAX,BYTE PTR DS:[EDX]                 ; 取机器码第 1，4，7，10，13
00409E97  |. |83E8 05        |SUB EAX,5                                   ; 减法
00409E9A  |. |83F8 41        |CMP EAX,41
00409E9D  |. |7E 16          |JLE SHORT 00409EB5                          ; 结果大于 41 就做加法运算
00409E9F  |. |8B4D 08        |MOV ECX,DWORD PTR SS:[EBP+8]
00409EA2  |. |034D EC        |ADD ECX,DWORD PTR SS:[EBP-14]
00409EA5  |. |0FBE11         |MOVSX EDX,BYTE PTR DS:[ECX]
00409EA8  |. |83EA 05        |SUB EDX,5
00409EAB  |. |8B45 0C        |MOV EAX,DWORD PTR SS:[EBP+C]
00409EAE  |. |0345 F8        |ADD EAX,DWORD PTR SS:[EBP-8]
00409EB1  |. |8810           |MOV BYTE PTR DS:[EAX],DL                    ; 转换计算后的 HEX 为字符
00409EB3  |. |EB 14          |JMP SHORT 00409EC9
00409EB5  |> |8B4D 08        |MOV ECX,DWORD PTR SS:[EBP+8]
00409EB8  |. |034D EC        |ADD ECX,DWORD PTR SS:[EBP-14]
00409EBB  |. |0FBE11         |MOVSX EDX,BYTE PTR DS:[ECX]
00409EBE  |. |83C2 05        |ADD EDX,5                                   ; 加法
00409EC1  |. |8B45 0C        |MOV EAX,DWORD PTR SS:[EBP+C]
00409EC4  |. |0345 F8        |ADD EAX,DWORD PTR SS:[EBP-8]
00409EC7  |. |8810           |MOV BYTE PTR DS:[EAX],DL                    ; 转换 HEX 为字符
00409EC9  |> |837D F0 01     |CMP DWORD PTR SS:[EBP-10],1
00409ECD  |. |75 3B          |JNZ SHORT 00409F0A
00409ECF  |. |8B4D 08        |MOV ECX,DWORD PTR SS:[EBP+8]
00409ED2  |. |034D EC        |ADD ECX,DWORD PTR SS:[EBP-14]
00409ED5  |. |0FBE11         |MOVSX EDX,BYTE PTR DS:[ECX]                 ; 取机器码第 2，5，8，11，14
00409ED8  |. |83C2 07        |ADD EDX,7                                   ; 加法
00409EDB  |. |83FA 5A        |CMP EDX,5A
00409EDE  |. |7D 16          |JGE SHORT 00409EF6                          ; 大于等于 5A 去做减法运算
00409EE0  |. |8B45 08        |MOV EAX,DWORD PTR SS:[EBP+8]
00409EE3  |. |0345 EC        |ADD EAX,DWORD PTR SS:[EBP-14]
00409EE6  |. |0FBE08         |MOVSX ECX,BYTE PTR DS:[EAX]
00409EE9  |. |83C1 07        |ADD ECX,7
00409EEC  |. |8B55 0C        |MOV EDX,DWORD PTR SS:[EBP+C]
00409EEF  |. |0355 F8        |ADD EDX,DWORD PTR SS:[EBP-8]
00409EF2  |. |880A           |MOV BYTE PTR DS:[EDX],CL
00409EF4  |. |EB 14          |JMP SHORT 00409F0A
00409EF6  |> |8B45 08        |MOV EAX,DWORD PTR SS:[EBP+8]
00409EF9  |. |0345 EC        |ADD EAX,DWORD PTR SS:[EBP-14]
00409EFC  |. |0FBE08         |MOVSX ECX,BYTE PTR DS:[EAX]
00409EFF  |. |83E9 07        |SUB ECX,7                                   ; 减法
00409F02  |. |8B55 0C        |MOV EDX,DWORD PTR SS:[EBP+C]
00409F05  |. |0355 F8        |ADD EDX,DWORD PTR SS:[EBP-8]
00409F08  |. |880A           |MOV BYTE PTR DS:[EDX],CL
00409F0A  |> |837D F0 02     |CMP DWORD PTR SS:[EBP-10],2
00409F0E  |. |75 3B          |JNZ SHORT 00409F4B
00409F10  |. |8B45 08        |MOV EAX,DWORD PTR SS:[EBP+8]
00409F13  |. |0345 EC        |ADD EAX,DWORD PTR SS:[EBP-14]
00409F16  |. |0FBE08         |MOVSX ECX,BYTE PTR DS:[EAX]                 ; 取机器码第 3，6，9，12
00409F19  |. |83E9 09        |SUB ECX,9                                   ; 减法
00409F1C  |. |83F9 41        |CMP ECX,41
00409F1F  |. |7E 16          |JLE SHORT 00409F37                          ; 小于等于 41 就做加法运算
00409F21  |. |8B55 08        |MOV EDX,DWORD PTR SS:[EBP+8]
00409F24  |. |0355 EC        |ADD EDX,DWORD PTR SS:[EBP-14]
00409F27  |. |0FBE02         |MOVSX EAX,BYTE PTR DS:[EDX]
00409F2A  |. |83E8 09        |SUB EAX,9                                   ; 减法
00409F2D  |. |8B4D 0C        |MOV ECX,DWORD PTR SS:[EBP+C]
00409F30  |. |034D F8        |ADD ECX,DWORD PTR SS:[EBP-8]
00409F33  |. |8801           |MOV BYTE PTR DS:[ECX],AL                    ; 转换 HEX 为字符，保存
00409F35  |. |EB 14          |JMP SHORT 00409F4B
00409F37  |> |8B55 08        |MOV EDX,DWORD PTR SS:[EBP+8]
00409F3A  |. |0355 EC        |ADD EDX,DWORD PTR SS:[EBP-14]
00409F3D  |. |0FBE02         |MOVSX EAX,BYTE PTR DS:[EDX]
00409F40  |. |83C0 09        |ADD EAX,9
00409F43  |. |8B4D 0C        |MOV ECX,DWORD PTR SS:[EBP+C]
00409F46  |. |034D F8        |ADD ECX,DWORD PTR SS:[EBP-8]
00409F49  |. |8801           |MOV BYTE PTR DS:[ECX],AL
00409F4B  |>\E9 0CFFFFFF     \JMP 00409E5C
00409F50  |>  C745 E4 00000>MOV DWORD PTR SS:[EBP-1C],0
00409F57  |.  C745 E8 00000>MOV DWORD PTR SS:[EBP-18],0
00409F5E  |.  C745 EC 00000>MOV DWORD PTR SS:[EBP-14],0
00409F65  |.  EB 09          JMP SHORT Hickwall.00409F70
00409F67  |>  8B55 EC        /MOV EDX,DWORD PTR SS:[EBP-14]
00409F6A  |.  83C2 01        |ADD EDX,1
00409F6D  |.  8955 EC        |MOV DWORD PTR SS:[EBP-14],EDX
00409F70  |>  8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]
00409F73  |.  3B45 F8        |CMP EAX,DWORD PTR SS:[EBP-8]
00409F76  |.  7D 17          |JGE SHORT Hickwall.00409F8F
00409F78  |.  8B4D 0C        |MOV ECX,DWORD PTR SS:[EBP+C]
00409F7B  |.  034D EC        |ADD ECX,DWORD PTR SS:[EBP-14]
00409F7E  |.  0FBE11         |MOVSX EDX,BYTE PTR DS:[ECX]                 ; 上面计算得到的每个字符的 HEX
00409F81  |.  8955 88        |MOV DWORD PTR SS:[EBP-78],EDX
00409F84  |.  DB45 88        |FILD DWORD PTR SS:[EBP-78]                  ; 浮点运算
00409F87  |.  DC45 E4        |FADD QWORD PTR SS:[EBP-1C]
00409F8A  |.  DD5D E4        |FSTP QWORD PTR SS:[EBP-1C] //把各次计算的结果累加
00409F8D  |.^ EB D8          \JMP SHORT Hickwall.00409F67
00409F8F  |>  8B45 E8        /MOV EAX,DWORD PTR SS:[EBP-18]
00409F92  |.  50             |PUSH EAX
00409F93  |.  8B4D E4        |MOV ECX,DWORD PTR SS:[EBP-1C]
00409F96  |.  51             |PUSH ECX
00409F97  |.  E8 42CF0000    |CALL <JMP.&MSVCRTD.sin>  //正弦计算，跟进
00409F9C  |.  DD1C24         |FSTP QWORD PTR SS:[ESP]
00409F9F  |.  E8 34CF0000    |CALL <JMP.&MSVCRTD.cos>
00409FA4  |.  DD1C24         |FSTP QWORD PTR SS:[ESP]
00409FA7  |.  E8 32CF0000    |CALL <JMP.&MSVCRTD.sin>
00409FAC  |.  DD1C24         |FSTP QWORD PTR SS:[ESP]
00409FAF  |.  E8 24CF0000    |CALL <JMP.&MSVCRTD.cos>
00409FB4  |.  DD1C24         |FSTP QWORD PTR SS:[ESP]
00409FB7  |.  E8 22CF0000    |CALL <JMP.&MSVCRTD.sin>
00409FBC  |.  83C4 08        |ADD ESP,8
00409FBF  |.  DD5D 8C        |FSTP QWORD PTR SS:[EBP-74]
00409FC2  |.  DD45 8C        |FLD QWORD PTR SS:[EBP-74]
00409FC5  |.  DC1D 78BB4200 |FCOMP QWORD PTR DS:[42BB78]
00409FCB  |.  DFE0           |FSTSW AX
00409FCD  |.  F6C4 01        |TEST AH,1
```



### 汇编代码分析

```
00409FD0  |.  74 0E         |JE SHORT Hickwall.00409FE0  
00409FD2  |.  DD45 E4       |FLD QWORD PTR SS:[EBP-1C]  
00409FD5  |.  DC0D 68BB4200 |FMUL QWORD PTR DS:[42BB68] //对字串四舍五入取小数点后 14 位，用到下面计算中。  
00409FDB  |.  DD5D E4       |FSTP QWORD PTR SS:[EBP-1C]  
00409FDE  |.^ EB AF         \JMP SHORT Hickwall.00409F8F  
00409FE0  |>  8BF4          MOV ESI,ESP  
00409FE2  |.  8B55 90       MOV EDX,DWORD PTR SS:[EBP-70]  
00409FE5  |.  52           PUSH EDX  
00409FE6  |.  8B45 8C       MOV EAX,DWORD PTR SS:[EBP-74]  
00409FE9  |.  50           PUSH EAX                           ; /<%.14f>  
00409FEA  |.  68 5CBB4200  PUSH Hickwall.0042BB5C             ; |format = "%.14f"  
00409FEF  |.  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]      ; |  
00409FF2  |.  51           PUSH ECX                           ; |s  
00409FF3  |.  FF15 7C274300 CALL DWORD PTR DS:[<&MSVCRTD.sprintf>] ; \sprintf  
00409FF9  |.  83C4 10       ADD ESP,10  
00409FFC  |.  3BF4          CMP ESI,ESP  
00409FFE  |.  E8 8BCC0000   CALL <JMP.&MSVCRTD.\_chkesp>  
0040A003  |.  C745 EC 00000>MOV DWORD PTR SS:[EBP-14],0  
0040A00A  |.  EB 09         JMP SHORT Hickwall.0040A015  
0040A00C  |>  8B55 EC       /MOV EDX,DWORD PTR SS:[EBP-14]  
0040A00F  |.  83C2 01       |ADD EDX,1  
0040A012  |.  8955 EC       |MOV DWORD PTR SS:[EBP-14],EDX  
0040A015  |>  8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]     ;  得到的结果四舍五入取小数点后 14 位（针对我机器的）  
0040A018  |.  50           |PUSH EAX                         ; /s  
0040A019  |.  E8 E4CD0000   |CALL <JMP.&MSVCRTD.strlen>        ; \strlen  
0040A01E  |.  83C4 04       |ADD ESP,4  
0040A021  |.  83E8 02       |SUB EAX,2  
0040A024  |.  3945 EC       |CMP DWORD PTR SS:[EBP-14],EAX  
0040A027  |.  73 15         |JNB SHORT Hickwall.0040A03E  
0040A029  |.  8B4D EC       |MOV ECX,DWORD PTR SS:[EBP-14]  
0040A02C  |.  0FBE540D 96   |MOVSX EDX,BYTE PTR SS:[EBP+ECX-6A]  
0040A031  |.  83C2 41       |ADD EDX,41  //HEX+41  
0040A034  |.  8B45 0C       |MOV EAX,DWORD PTR SS:[EBP+C]  
0040A037  |.  0345 EC       |ADD EAX,DWORD PTR SS:[EBP-14]  
0040A03A  |.  8810          |MOV BYTE PTR DS:[EAX],DL  
0040A03C  |.^ EB CE         \JMP SHORT Hickwall.0040A00C  //
取求得的数字结果的第 2 位开始的小数点后数字的 HEX 值+$41，得到的 HEX 转换为字符，顺序连接为 13 个字符的最后正确注册码字符串，下面转换为大写，完成/
0040A03E  |>  8B4D 0C       MOV ECX,DWORD PTR SS:[EBP+C]  //这里是注册码小写字符串  
0040A041  |.  034D F8       ADD ECX,DWORD PTR SS:[EBP-8]  
0040A044  |.  C601 00       MOV BYTE PTR DS:[ECX],0  
0040A047  |.  8BF4          MOV ESI,ESP  
0040A049  |.  8B55 0C       MOV EDX,DWORD PTR SS:[EBP+C]  
0040A04C  |.  52           PUSH EDX                           ; /StringOrChar  
0040A04D  |.  FF15 78284300 CALL DWORD PTR DS:[<&USER32.CharUpperA>] ; \CharUpperA  
0040A053  |.  3BF4          CMP ESI,ESP  
0040A055  |.  E8 34CC0000   CALL <JMP.&MSVCRTD.\_chkesp>  
0040A05A  |.  5F           POP EDI  
0040A05B  |.  5E           POP ESI  
0040A05C  |.  5B           POP EBX  
0040A05D  |.  81C4 B8000000 ADD ESP,0B8  
0040A063  |.  3BEC          CMP EBP,ESP  
0040A065  |.  E8 24CC0000   CALL <JMP.&MSVCRTD.\_chkesp>  
0040A06A  |.  8BE5          MOV ESP,EBP  
0040A06C  |.  5D           POP EBP  
0040A06D  \\.  C2 0800      RETN 8  
```

#### 背景说明

一开始在计算器里误认为是按角度计算，结果出错进跟进去了，与计算分析无关，嘿嘿。

#### 三角函数调用分析

```
00409F97  |.  E8 42CF0000   |CALL <JMP.&MSVCRTD.sin>  
```

跟入 `sin` 运算：

```
10246614  >  8D5424 04       LEA EDX,DWORD PTR SS:[ESP+4]  
10246618    E8 B8410000      CALL 1024A7D5  
1024661D    52             PUSH EDX  
1024661E    9B             WAIT  
1024661F    D93C24         FSTCW WORD PTR SS:[ESP]  
10246622    74 50          JE SHORT 10246674  
10246624    66:813C24 7F02  CMP WORD PTR SS:[ESP],27F  
1024662A    74 06          JE SHORT 10246632  
1024662C    D92D C8012510   FLDCW WORD PTR DS:[102501C8]  
10246632    D9FE           FSIN                              ; SIN（上面浮点运算的累加和）存入 ST0 按照弧度进行运算  
10246634    9B             WAIT  
10246635    DFE0           FSTSW AX  
10246637    9E             SAHF  
10246638    7A 1D          JPE SHORT 10246657  
```

同样下面的 `sin`, `cos` 也同样，把计算的结果，做为下一次三角函数计算的 `ST0`，最后对得到的数字字串进行处理。

#### 算法总结

1.  取 `ID` 的 1, 4, 7, 10, 13 位 `HEX` 与 `$5` 先做减法，如果结果不小于等于 `$41` 就直接把 `HEX` 结果转为字符保存，如果大于 `$41` 就改为与 `$5` 做加法运算，把 `HEX` 结果转换为字符保存；
2.  取 2, 5, 8, 11, 14 位 `HEX` 与 `$7` 做加法，如果结果不大于等于 `$5A`，就把 `HEX` 结果转换为字符保存，如果大于等于 `$5A`，就去与 `$7` 做减法运算，把 `HEX` 结果转换为字符保存；
3.  取 3, 6, 9 位 `HEX` 与 `$9` 做减法运算，如果结果不小等于于 `$41`，就把 `HEX` 结果保存为字符，如果小于等于 `$41`，就与 `$9` 做加法运算，把 `HEX` 结果转换后保存为字符。

把得到的各个 `HEX` 值累加后求 `sin`, 再 `cos`，再 `sin`，再 `cos`，再 `sin`，取结果的小数点后的第 2 到 14 位字符的 `HEX` 值分别加上 `$41`，把 `HEX` 结果转换为字符连接起来，转换为大写即为最后注册码。

有错误的地方，请指点，谢谢。

原来是浮点运算用来处理三角函数等计算，正好从另一帖子里学习到了，消化 ing 。

感谢 ing，引用一下。

#### 浮点运算参考问答

下面的内容是我以前的提问，别人的回答，如下：

>> 我想进行浮点数的运算，用来解决三角函数、乘、除等运算。

ok, 

>> 看了有关的 E 文资料，却对其中的 FPU 寄存器的存取和操作不太明白，谁能给点详细的资料吗？

`MASM32` `V8` 的 `HELP` 目录下有一个 `FPHELP.HLP` 文件，看看再练习练习就明白了。

>> 好象浮点数运算必须用到 FPU 寄存器吗？

是。

>> 资料里说 STO~ST6，这些寄存器怎么使用？

准确说应该说是 `ST(0)~ST(7)`，使用方法跟常规寄存器类似。

>> 比如我们用 `EAX`，可以直接 `LODSB`，或者 `LEA EAX, ADDRESS`，这些 FPU 寄存器怎么使用？

对应“`LODSB`”的指令有：

```
fld     Push real onto stack 
fild    Convert two's complement integer to real and push
```

对应“`LEA EAX, ADDRESS`”没有！你要这个干什么？

这是我 `Crackme3` 中的一段源程序，仅供参考。

```
    Code: [Copy to clipboard]
    
    mov dwAngle,edx        ;将上面的计算值存入变量     
    fld rMagic         ;将一个常数装到 st(0)
    fdiv r100          ;将这个数除以 100，结果在 st(0)
    fiadd dwAngle            ;加上面提到的值，结果在 st(0)
    fmul rPI           ;×π，结果在 st(0)。当然调π值有一个专门的指令 fldpi
    fdiv rDEG          ;除以一个常数，结果在 st(0)
    fcos                 ;cos 上面的结果，结果在 st(0)
    fabs                 ;取绝对值，结果在 st(0)
    fmul rty           ;×一个常数，结果在 st(0)
    fmul r100          ;×一个常数，结果在 st(0)
    frndint            ;取整，结果在 st(0) 
    fcom r123          ;st(0)中的值跟一个常数比较，影响 C0 - C3 标志位(word)
    fstsw ax           ;因上面的标志位在程序中无法直接使用，故将这个 Word 值存入 ax 寄存器
    sahf            ;Store AH Register into FLAGS
    jnz _Quit          ;这时我们就可以判断了 :)
    xor eax,eax          ;跟 FPU 无关
    inc al                 ;跟 FPU 无关
    jmp _Exit          ;跟 FPU 无关 
```

希望上面的解释对你有点帮助，看不清楚的地方再讨论。

> 引用:
>
> *最初由 prince 发布*
>
> **呵呵，支持，不过我有个疑问，请指教，谢谢。**
>
> **关于标准的 sin 函数，我们得到它的返回值不就行了？为什么还要跟进去看系统的计算过程呢？**

因为不明白计算，用计算器计算的时候误当为角度计算，死活不对，就跟进去看，发现没错误，只是应该当弧度计算。嘿嘿，才开始学习算法分析，天天向上 ing 。

#### Delphi 注册机源码

```
Var  
  a,b,i:integer;  
  c:Real;  
  id,sntemp,sn1,sn2,sn3,sn:String;  
begin  
  id:=pchar(Edit1.Text);  
  sntemp:='';  
  sn1:='';  
  sn2:='';  
  sn:='';  
  for i:=1 to 5 do       //计算 1，4，7，10，13 位  
  begin  
    a:=ord(id[3*i-2])-$5 ;  
    if (a<=$41)  then  
      sntemp:=char(a+$A)  
    else  
      sntemp:=char(a);  
      sn1:=sn1+sntemp;  
  end;  
    for i:=1 to 5 do    //计算，2，5，8，11，14 位  
  begin  
    a:=ord(id[3*i-1])+$7 ;  
    if (a>=$5A)  then  
      sntemp:=char(a-$E)  
    else  
      sntemp:=char(a);  
      sn2:=sn2+sntemp;  
  end;  
  for i:=1 to 4 do       //计算 3，6，9 位  
  begin  
    a:=ord(id[3*i])-$9 ;  
    if (a<=$41)  then  
      sntemp:=char(a+$12)  
    else  
      sntemp:=char(a);  
      sn3:=sn3+sntemp;  
  end;  
  sn:=sn1[1]+sn2[1]+sn3[1]+sn1[2]+sn2[2]+sn3[2]  
     +sn1[3]+sn2[3]+sn3[3]+sn1[4]+sn2[4]+sn3[4]+sn1[5]+sn2[5];  //连接计算结果  
//下面进行 HEX 累加，并求三角函数运算结果。  
  b:=0;  
  for i:=0 to 13 do  
  begin  
    b:=b+ord(sn[i]);  
  end;  
  c:=sin(cos(sin(cos(sin(b)))));  
  sntemp:=floattostr(c*1000000000000000);  //结果转为字串为 14 位  
  sn:='';  
  for i:=1 to 13 do   //取 14 位字串中的 2-14 位  
  begin  
    a:=ord(sntemp[i])+$41;  
    sn:=sn+char(a);  
  end;  
  Edit2.Text:=StrUpper(pchar(sn));  
end;  
```



```
====================================================================== 

00411C1F  |.  50            PUSH EAX                   ; /n  

00411C20  |.  8D85 48FFFFFF LEA EAX,DWORD PTR SS:[EBP-B8]     ; |  

00411C26  |.  50            PUSH EAX                   ; |s2  

00411C27  |.  8D8D F8FEFFFF LEA ECX,DWORD PTR SS:[EBP-108]    ; |  

00411C2D  |.  51            PUSH ECX                   ; |s1  

00411C2E  |.  E8 FD510000   CALL <JMP.&MSVCRTD.memcmp>       ; \\memcmp  
```

`s2` 是假，`s1` 是真。

谢谢楼主的算法分析，强！

`4dd466d0e1e0cc123afbe504da2462ca`
