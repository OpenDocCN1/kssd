
```
0045D7F8  20 00 20 00 20 00 20 00   . . .
.
0045D800  20 00 20 00 20 00 20 00   . . . .
0045D808  20
00 20 00 20 00 20 00   . . . .
0045D810  20 00 20 00 20 00 20 00
  . . . .
0045D818  20 00 48 00 10 00 10 00   .H...
0045D820
 10 00 10 00 10 00 10 00  ....
0045D828  10 00 10 00 10
00 10 00  ....
0045D830  10 00 10 00 10 00 10 00  ....
0045D838
 10 00 84 00 84 00 84 00  .???           ;由于偏移量最小是从45D83A开始的.
0045D840
 84 00 84 00 84 00 84 00  ????            ;分析下面的各个值没有符合条件的.
0045D848
 84 00 84 00 84 00 10 00  ???.
0045D850  10 00 10 00 10 00
10 00  ....
0045D858  10 00 10 00 81 00 81 00  ..??
0045D860
 81 00 81 00 81 00 81 00  ????
0045D868  01 00 01 00 01 00 01
00  ....
0045D870  01 00 01 00 01 00 01 00  ....
0045D878
 01 00 01 00 01 00 01 00  ....
0045D880  01 00 01 00 01
00 01 00  ....
0045D888  01 00 01 00 01 00 01 00  ....
0045D890
 10 00 10 00 10 00 10 00  ....
0045D898  10 00 10 00 82
00 82 00  ..??
0045D8A0  82 00 82 00 82 00 82 00  ????
0045D8A8
 02 00 02 00 02 00 02 00  ....
0045D8B0  02 00 02 00 02
00 02 00  ....
0045D8B8  02 00 02 00 02 00 02 00  ....
0045D8C0
 02 00 02 00 02 00 02 00  ....
0045D8C8  02 00 02 00 02
00 02 00  ....
0045D8D0  10 00 10 00 10 00 10 00  ....
0045D8D8
 20 00 00 00 00 00 00 00   .......
--------------
```

这一段只是为下面服务的

==============================================

继续:

-----

|

|

0041F790

```
XOR     EBX, EBX
0041F792  MOV     BL, BYTE
PTR DS:[ESI]          ;  BL <== DS:[0012F514]=30
('0') || =33 ||=37
0041F794  INC     ESI
                     ;
 ESI ++
0041F795  MOV     EDI, EBX
                ;  EDI <==
30
0041F797  CMP     EBX, 2D
0041F79A  JE
 SHORT Revival.0041F7A1
0041F79C  CMP     EBX, 2B
0041F79F
 JNZ     SHORT Revival.0041F7A6
0041F7A1  XOR
EBX, EBX
0041F7A3  MOV     BL, BYTE PTR DS:[ESI]
0041F7A5
 INC     ESI
0041F7A6  XOR     EBP, EBP
0041F7A8
 /CMP     DWORD PTR DS:[45D9DC], 1      ;  DS:[45D9DC]=01
0041F7AF
 |JLE     SHORT Revival.0041F7BE
0041F7B1  |PUSH
 4
0041F7B3  |PUSH    EBX
0041F7B4  |CALL
 Revival.00423610
0041F7B9  |ADD     ESP, 8
0041F7BC
 |JMP     SHORT Revival.0041F7CD
0041F7BE  |MOV
  ECX, DWORD PTR DS:[45D7D0]    ;  如果上面的条件都不成立,就跳到这里 ECX
<== DS:[00457D0] (基地址)
0041F7C4  |XOR     EAX, EAX
0041F7C6
 |MOV     AX, WORD PTR DS:[ECX+EBX*2]   ;  EBX =30
还是用第三位的hex值来做指针寻址
0041F7CA  |AND     EAX, 4
                 ;  计算方法
0041F7CD
 |TEST    EAX, EAX
       ;  注册码的判断条件,即DL*2偏移处的值AND 4 不能等于0
0041F7CF
 |JE      SHORT Revival.0041F7E1
0041F7D1  |LEA
  EAX, DWORD PTR SS:[EBP+EBP*4] ; EBP=0 ,第二次它就是把试验码*5==>EAX中,这个值有用
0041F7D5
 |INC     ESI
            ;  //EAX =0 ;EAX =HEX*5
0041F7D6
 |LEA     EBP, DWORD PTR DS:[EBX+EAX*2->;  EBP <==
01 |<==03  这个计算是把十六进制转化为ASCII值,因为EAX=0||第二次计算的时候
0041F7DA  |XOR
    EBX, EBX
     ;  //就是把后一位的hex+前一位值*A-30
0041F7DC  |MOV
    BL, BYTE PTR DS:[ESI-1]       ;  BL <==
DS:[0012F515]=31 ('1')
0041F7DF  \JMP     SHORT Revival.0041F7A8
0041F7E1
 MOV     EAX, EBP
        ;  EAX <==EBP (经过上面计算的值) ||(78901) =13435|||(56)=38
0041F7E3
 CMP     EDI, 2D
0041F7E6  JNZ     SHORT Revival.0041F7EF
0041F7E8
 NEG     EAX
0041F7EA  POP     EBP
0041F7EB
 POP     EDI
0041F7EC  POP     ESI
0041F7ED
 POP     EBX
0041F7EE  RETN
-----------------
```

这一段要分析一下:

下面要的四个关键值的计算

1.第一次用试验码的第三位,第四位进行如下计算:后一位值+前一位值\*A  例如:(01)==>1+0\*A=01

2.第二次用试验码的第六位,第七位进行相同计算:后一位值+前一位值\*A

 例如:(34)==>4+3\*A=22

3.第三次用试验码的最后五位进行如下计算:前一位值\*A+后一位值 ,然后把前面的值\*A+后一位的值

例如(78901)==>1+(0+(9+(8+7\*a)\*a)\*a)\*a=13435

4.第四次用试验码的第九位,第十位进行如下计算:前一位值\*A+后一位值

 例如:(56)==>6+5\*A=38

======================================================================================

```
00403347
  MOV     CX, WORD PTR DS:[ESI+5]        ;
 CX = 3433 ("34")
0040334B   ADD     ESP, 4
0040334E
  MOV     WORD PTR SS:[ESP+10], CX       ;  CX
==>SS:[0012F514]
00403353   SUB     AL, 13
                    ;  AL
=01 SUB 13=EE  |这里把第一组值-13
00403355   LEA     ECX, DWORD
PTR SS:[ESP+10]     ;  ECX <==0012F514, (ASCII "34")
00403359
  MOV     BYTE PTR SS:[ESP+C], AL        ;
 AL =EE  ==>SS:[0012F510]  |第一组的值保存在这里
0040335D   MOV
    BYTE PTR SS:[ESP+12], 0
00403362   PUSH    ECX
00403363
  CALL    Revival.0041F800
  \*\*\*\*\*\*\*\*\*第二次调用
00403368   LEA     EDX, DWORD PTR SS:[ESP+14]
    ;  EDX <== 0012F514,(ASCII "34")
0040336C
  ADD     ESP, 4
0040336F   LEA     EBX, DWORD
PTR DS:[EAX-25]     ;  EBX <== EAX(上面CALL计算的值)-25=FFFFFFFD
 |第二组计算的值-25
00403372   LEA     ECX, DWORD PTR DS:[ESI+A]
     ;  ECX <== DS:[ESI+A]=00932676,(ASCII "78901").
试验码的后五位
00403375   PUSH    EDX
                 ;  EDX
=34
00403376   MOV     EAX, DWORD PTR DS:[ECX]
   ;  EAX <== DS:[00932676]=30393837
00403378
MOV     DWORD PTR DS:[EDX], EAX        ;  DS:[0012F514]
<== 03393837
0040337A   MOV     CL, BYTE PTR DS:[ECX+4]
     ;  CL <== DS:[ECX+4]=31 ('1') 试验码的最后一位
0040337D
  MOV     BYTE PTR DS:[EDX+4], CL        ;
 第五位也转移过来
00403380   MOV     BYTE PTR SS:[ESP+19], 0
00403385
  CALL    Revival.0041F800
  \*\*\*\*\*\*\*\*\*第三次调用
0040338A   ADD     ESP, 4
0040338D
  MOV     EDI, EAX
        ;  EDI <==13435 (后五位经过上面计算的值)
0040338F
  XOR     DI, 5468
        ;  DI=3435 XOR 5468=605D  (只取这个值的后四位计算)
00403394
  MOV     AX, WORD PTR DS:[ESI+8]        ;
 AX <== DS:[00932674]=3635 ("56") ,试验码的第九第十位
00403398
MOV     WORD PTR SS:[ESP+10], AX
0040339D   LEA
EAX, DWORD PTR SS:[ESP+10]     ;  EAX <== SS:[ESP+10]=0012F514,(ASCII
"56901")
004033A1   MOV     BYTE PTR SS:[ESP+12],
0
004033A6   MOVZX   EDI, DI
           ;  EDI<==605D
004033A9
  PUSH    EAX
004033AA   CALL    Revival.0041F800
              \*\*\*\*\*\*\*\*\*第四次调用
004033AF
MOV     BYTE PTR SS:[ESP+14], AL       ;  SS:[0012F514]=38
 ,第四次的值
004033B3   ADD     ESP, 4
004033B6   XOR
    EAX, EAX
004033B8   MOV     ECX, 64
                   ;  ECX
=64    <--参数
004033BD   MOV     AL, BL
                    ;  AL
=FD     ,第二次的值
004033BF   MOV     EBX, 0A
                   ;  EBX
<=0A   <--参数
004033C4   LEA     EAX, DWORD PTR DS:[EAX+EDI+3]
 ;  EAX <==FD+6150D+3 ,第三次的值
004033C8   CDQ
004033C9
  IDIV    ECX
             ;  ECX = 64  IDIV
 EAX = 615D ==>EAX =F9 EDX=19
004033CB   MOV     CL,
DL
  ;  CL <== 19
004033CD   XOR     EAX, EAX
004033CF
  MOV     AL, BYTE PTR SS:[ESP+C]        ;
 AL <== SS:[0012F510]=EE ,第一次的值
004033D3   LEA     EAX,
DWORD PTR DS:[EAX+EDI+3]  ;  EAX =EE+605D+3= 614E
004033D7
CDQ
004033D8   IDIV    EBX
                 ;  EAX
=614E IDIV EBX=A ==>EAX=9BB EDX=000
004033DA   SUB     DL,
BYTE PTR DS:[ESI+4]        ;  DL =DL-32 ('2') ,试验码的第五位
004033DD
  CMP     DL, 0D0
         ;  DL =CE
004033E0   JNZ
  SHORT Revival.00403406         ;  注册码的正确条件7
004033E2
  CMP     CL, BYTE PTR SS:[ESP+10]       ;  CL
= 19    SS:[ESP+10] =SS:[0012F514]=38
004033E6   JNZ
  SHORT Revival.00403406         ;  注册码的正确条件8
004033E8
  MOV     EAX, 1
004033ED   POP     EDI
004033EE
  POP     ESI
004033EF   POP     EBX
004033F0
  ADD     ESP, 24
004033F3   RETN
#####################################################################################
```

以上算法分析已经完成,下面来总结注册码的计算方法:

注册码的长度=15位

  1.首先看注册码的前二位比较容易固定值rv

  2.注册码的第八位是 2D('-')

   3.注册码的其他几位必须是数字

   4.其他几位正确的条件:设第一次计算的值为M
,第二次的值为N ,第三次值的后四位为J ,第四次计算的值为K

           1).(M+(J
XOR 5468)+3) MOD A + D0 = 注册码的第五位

           2).(N+(J
XOR 5468)+3) MOD 64 = k

5.假如我们知道M N J K

  这样注册码的这几位可以知道假设了: rv01?34-??78901

6.为了使上式成立,我们做如下分析:

从 K 分析: 设K=19 ,第九位为T1 ,第十位为T2

 则:T2=19-T1\*A >=0 ; T1 的取值范围只能是 2  那么 T2 =19-2\*A=5

这样把注册码设成:rv01???-2578901

   现在来分析一下它对第一个条件能不能成立:

    1).(M+(J XOR 5468)+3) MOD A = 注册码的第五位

       上式改为注册码的第五位=6145 MOD A =0

把注册码改为:rv01034-2578901

几点说明:

   1.这个软件的注册码如果不进行假设的话,可能很难知道结果.

  2.SUB     DL, BYTE PTR DS:[ESI+4]        ;
 DL =DL-32 ('2') ,试验码的第五位
     (M+(J XOR 5468)+3) MOD A = 注册码的第五位,这个我是怎么知道的
    原来 MOD A 得到的值是<A -30以上的值是负数,我无法用计算器计算,只好用sice的计算功能才
 知道0-30=D0 ,因此上面式子就是:
     SUB     DL, BYTE PTR DS:[ESI+4]
