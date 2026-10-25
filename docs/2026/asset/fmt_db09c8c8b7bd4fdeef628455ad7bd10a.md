
```asm
EBX,02D4
|
0167:0041638D
 MOVSX    EAX,BYTE [EDI+ECX]
|
0167:00416391
 TEST     BYTE [EAX+00444089],08
|
0167:00416398  JZ
    NEAR 004164E0
|
0167:0041639E
 ADD      EBX,BYTE +04
|
0167:004163A1  JMP      004164E0
|
0167:004163A6  ADD      EBX,0340
|
0167:004163AC  MOVSX
   EDX,BYTE [EDI+ECX]
|
0167:004163B0  TEST
    BYTE [EDX+00444089],08
|
0167:004163B7  JZ
    NEAR 004164E0
|
0167:004163BD
 ADD      EBX,06DF
|
0167:004163C3  JMP      004164E0
|
0167:004163C8  ADD
 EBX,BYTE +12
|
0167:004163CB  MOVSX    EAX,BYTE [EDI+ECX]
|
0167:004163CF
 TEST     BYTE [EAX+00444089],08
|
0167:004163D6  JZ
    NEAR 004164E0
|
0167:004163DC
 ADD      EBX,05BE
|
0167:004163E2  JMP      004164E0
|
0167:004163E7  ADD
 EBX,16DA
|
0167:004163ED
 MOVSX    EDX,BYTE [EDI+ECX]
|
0167:004163F1
 TEST     BYTE [EDX+00444089],08
|
0167:004163F8  JZ
    NEAR 004164E0
|
0167:004163FE
 ADD      EBX,01BE
|
0167:00416404  JMP      004164E0
|
0167:00416409  ADD
 EBX,BYTE +22
|
0167:0041640C  MOVSX
   EAX,BYTE [EDI+ECX]
|
0167:00416410  TEST
    BYTE [EAX+00444089],08
|
0167:00416417  JZ
  NEAR 004164E0
|
0167:0041641D  ADD
     EBX,BYTE +16
|
0167:00416420
 JMP      004164E0
|
0167:00416425  ADD      EBX,BYTE +44
|
0167:00416428  MOVSX    EDX,BYTE [EDI+ECX]
|
0167:0041642C  TEST     BYTE [EDX+00444089],08
|
0167:00416433  JZ       NEAR 004164E0
|
0167:00416439  ADD      EBX,026A
|
0167:0041643F  JMP
     004164E0
|
0167:00416444  ADD      EBX,029A
|
0167:0041644A  MOVSX    EAX,BYTE
[EDI+ECX]
|
0167:0041644E  TEST     BYTE [EAX+00444089],08
|
0167:00416455  JZ       NEAR 004164E0
|
0167:0041645B  ADD      EBX,3039
|
0167:00416461  JMP
     SHORT 004164E0
|
0167:00416463
 ADD      EBX,0185
|
0167:00416469  MOVSX    EDX,BYTE [EDI+ECX]
|
0167:0041646D  TEST     BYTE [EDX+00444089],08
|
0167:00416474
 JZ       004164E0
|
0167:00416476  ADD      EBX,1272
|
0167:0041647C  JMP
 SHORT 004164E0
|
0167:0041647E  ADD
     EBX,0271
|
0167:00416484  MOVSX    EAX,BYTE [EDI+ECX]
|
0167:00416488  TEST     BYTE [EAX+00444089],08
|
0167:0041648F
 JZ       004164E0
|
0167:00416491  ADD      EBX,BA
|
0167:00416497  JMP
   SHORT 004164E0
|
0167:00416499
 ADD      EBX,014D
|
0167:0041649F  MOVSX    EDX,BYTE [EDI+ECX]
|
0167:004164A3  TEST     BYTE [EDX+00444089],08
|
0167:004164AA
 JZ       004164E0
|
0167:004164AC  ADD      EBX,0285
|
0167:004164B2  JMP
 SHORT 004164E0
|
0167:004164B4  ADD
     EBX,2342
|
0167:004164BA  MOVSX    EAX,BYTE [EDI+ECX]
|
0167:004164BE
 TEST     BYTE [EAX+00444089],08
|
0167:004164C5  JZ
    004164E0
|
0167:004164C7
 ADD      EBX,B0
|
0167:004164CD  JMP      SHORT 004164E0
|
0167:004164CF  INC      EBX
|
0167:004164D0
 MOVSX    EDX,BYTE [EDI+ECX]
|
0167:004164D4
 TEST     BYTE [EDX+00444089],08
|
0167:004164DB  JZ
    004164E0
|
0167:004164DD
 ADD      EBX,BYTE +03
----------------------|
0167:004164E0  TEST     ESI,ESI
               <---判断计算注册码的第几部分,
0167:004164E2  JNZ      004164F5
              从而转向不同的运算,见后
0167:004164E4
 MOV      EAX,EBX
0167:004164E6  PUSH
ECX
0167:004164E7  MOV      ECX,03
          <---ECX=3
0167:004164EC  XOR
   EDX,EDX
0167:004164EE  DIV      ECX
                 <---EAX=EAX/ECX
0167:004164F0
 POP      ECX
     <---恢复ECX值
0167:004164F1  ADD      EBX,EAX
               <---EBX=EBX+EAX
0167:004164F3
 JMP      SHORT 0041651B
0167:004164F5  CMP
   ESI,BYTE +01           <---计算注册码第2部分会走到这里|
0167:004164F8
 JNZ      00416502
         |
0167:004164FA  MOV      EAX,EBX
                      |
0167:004164FC
 SHR      EAX,1
            |
0167:004164FE  ADD
   EBX,EAX
    |
0167:00416500  JMP      SHORT 0041651B
        <------------------------------|
0167:00416502
 CMP      ESI,BYTE +02
<---计算注册码第3部分会走到这里|
0167:00416505  JNZ      0041651B
                     |
0167:00416507
 MOVSX    EDX,BYTE [EDI+01]
  |
0167:0041650B  TEST     BYTE [EDX+00444089],08
     |
0167:00416512  JZ       0041651B
                     |
0167:00416514
 MOV      EAX,EBX
          |
0167:00416516  SHR
 EAX,02
   |
0167:00416519  ADD      EBX,EAX
           <------------------------------|
0167:0041651B  INC      ECX
                 <---ECX加1
0167:0041651C  MOV
     EDX,[EBP-04]
0167:0041651F  CMP      ECX,EDX
               <---比较是否取完用户名
0167:00416521
 JL       NEAR 0041606B
0167:00416527  MOV
   EAX,EBX                <---取运算后的最终值存入EAX中,
0167:00416529
 POP      EDI
         即正确注册码的16进制值
0167:0041652A  POP
   ESI
0167:0041652B  POP      EBX
0167:0041652C
 POP      ECX
0167:0041652D  POP      EBP
0167:0041652E
 RET
```

唉！写到这头都晕了，还是没能把这个软件详细分析出来，正如前面所看到的`0167:00415DF2`处的判断是判断程序运行的次数，很显然是小于200次，所以在这个地方肯定会跳，也即无法注册。猜想可能在此之前已进行了注册验证，但我功力有限，跟了几次还是一样，实在不想跟了，还请高手跟一下看看！不过根据作者的说明好像这是个演示版，因为还有功能限制,我不确定作者是否将这部分功能做在程序中。

程序中运用查表法对用户名进行有关运算后得到注册码的值，我实在不知该如何描述，各位自己看看吧！呵呵头都晕了，如果看不明白是我写得不好，请见谅！

我把`0167:00415DF7`处改为两个`nop`，然后填入用户名:`ShenGe`和注册码:`73348-109543-59405`，注册成功！

嘿嘿！别高兴得太早，等到你运行到151次时你就会看到那个讨厌的对话框出来了，不能运行程序了。前面已经讲过，`[0046B12C]`中存放的为程序运行的次数，下`BPM 46B12C`，重新运行程序，你会很快发现如下代码：

```asm
\* Possible Reference to Dialog: IDD_MATRIX_EDIT,
CONTROL_ID:0096, ""
                   |
:00401121
3D96000000              cmp eax, 00000096
    <---判断是否运行超过了150次
:00401126 7628
           jbe 00401150
   <---改为jmp 00401150
:00401128 8B152CB14600
     mov edx, dword ptr [0046B12C]
:0040112E 81FAC8000000
         cmp edx, 000000C8       <---判断是否运行超过了200次
:00401134
731A                    jnb
00401150            <---上面改过了此处就不用改了
:00401136
6A40                    push
00000040
:00401138 68FEB04600              push
0046B0FE
\* Possible StringData Ref from Data Obj
->"Thank you for using Access Image "
       ->"software.
You were already accessed "
               ->"this software
more then 150 times.
If "
   ->"you want to continue using this "
           ->"software you must
buy it
for $18 "
                     ->"US
only."
               |
:0040113D 68B4F54300
             push 0043F5B4
:00401142 6A00
                   push 00000000
\*
Reference To: USER32.MessageBoxA, Ord:0000h
                       |
:00401144
E82BD50300              Call 0043E674
:00401149
33C0                    xor
eax, eax
:0040114B E967020000              jmp
004013B7
```

将上面的`:00401126 7628 jbe 00401150`改为`EB28`，再运行一下，OK！又可以用了！但是还有那该死的对话框，我实在不想再跟了，各位有兴趣的话试试看吧！等等，这不是爆破吗？前面那么多岂不是白写了吗？到头来还是爆破，唉，不干了！睡觉去了！

最后还是爆破了，但还有功能限制，高手不妨试试看！本不想贴出来，但老外的算法还是比较有意思，各位如果有兴趣不妨研究一下。水平太低，失误之处还请各位老大指点！

----------------------------------------ShenGe----------------------------------

爆破点：

1. 改`0167:00401126  JBE 00401150`为`JMP 00401150`
2. 改`0167:00415DF7 JNZ 00415E78`为`NOP NOP`
3. 改`0167:00415E16  JNZ 00415E78`为`JZ 00415E78`
4. 改`0167:00415E20 JNZ 00415E78`为`JZ 00415E78`
5. 改`0167:00415E2A  JNZ 00415E78`为`JZ 00415E78`
