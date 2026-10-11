
```text
0043A170   . 0F80 E7070000  JO CollegeB.0043A95D  

0043A176   . 66:8955 DC     MOV WORD PTR SS:[EBP-24],DX   

0043A17A   > 66:8B45 DC     MOV AX,WORD PTR SS:[EBP-24] ；字符索引  

0043A17E   . 66:3B85 38FFFF>CMP AX,WORD PTR SS:[EBP-C8] ; 名字长度  

0043A185   . 0F8F 0A020000  JG CollegeB.0043A395  

0043A18B   . C745 FC 040000>MOV DWORD PTR SS:[EBP-4],4      
    
0043A192   . C745 A4 010000>MOV DWORD PTR SS:[EBP-5C],1  

0043A199   . C745 9C 020000>MOV DWORD PTR SS:[EBP-64],2  

0043A1A0   . 8D4D 9C        LEA ECX,DWORD PTR SS:[EBP-64]  

0043A1A3   . 51             PUSH ECX                             ;  variant 类型，整数 1   

0043A1A4   . 0FBF55 DC      MOVSX EDX,WORD PTR SS:[EBP-24]  

0043A1A8   . 52             PUSH EDX  

0043A1A9   . 8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]  

0043A1AC   . 8B08           MOV ECX,DWORD PTR DS:[EAX]  

0043A1AE   . 51             PUSH ECX  

0043A1AF   . FF15 00114000  CALL rtcMidCharBStr        ;取注册名字的一个字符  

0043A1B5   . 8BD0           MOV EDX,EAX  

0043A1B7   . 8D4D B4        LEA ECX,DWORD PTR SS:[EBP-4C]  

0043A1BA   . FF15 C8124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>]        ;  MSVBVM60.__vbaStrMove  

0043A1C0   . 50             PUSH EAX  

0043A1C1   . FF15 58104000  CALL MSVBVM60.rtcAnsivalueBstr    ;  该字符的 ascii   

0043A1C7   . 66:8985 48FFFF>MOV WORD PTR SS:[EBP-B8],AX      ;  存入[ebp-B8]  

0043A1CE   . C745 94 010000>MOV DWORD PTR SS:[EBP-6C],1  

0043A1D5   . C745 8C 020000>MOV DWORD PTR SS:[EBP-74],2  

0043A1DC   . 8D55 8C        LEA EDX,DWORD PTR SS:[EBP-74]  

0043A1DF   . 52             PUSH EDX  

0043A1E0   . 0FBF45 DC      MOVSX EAX,WORD PTR SS:[EBP-24]  

0043A1E4   . 50             PUSH EAX  

0043A1E5   . 8B4D 08        MOV ECX,DWORD PTR SS:[EBP+8]  

0043A1E8   . 8B11           MOV EDX,DWORD PTR DS:[ECX]  

0043A1EA   . 52             PUSH EDX  

0043A1EB   . FF15 00114000  CALL rtcMidCharBStr        ;取同一个字符  

0043A1F1   . 8BD0           MOV EDX,EAX  

0043A1F3   . 8D4D B0        LEA ECX,DWORD PTR SS:[EBP-50]  

0043A1F6   . FF15 C8124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>]        ;  MSVBVM60.__vbaStrMove  

0043A1FC   . 50             PUSH EAX  

0043A1FD   . FF15 58104000  CALL MSVBVM60.rtcAnsivalueBstr   ;  该字符的 ascii   

0043A203   . 66:8985 44FFFF>MOV WORD PTR SS:[EBP-BC],AX      ；存入此处  

0043A20A   . C745 84 010000>MOV DWORD PTR SS:[EBP-7C],1  

0043A211   . C785 7CFFFFFF >MOV DWORD PTR SS:[EBP-84],2  

0043A21B   . 8D85 7CFFFFFF  LEA EAX,DWORD PTR SS:[EBP-84]  

0043A221   . 50             PUSH EAX  

0043A222   . 0FBF4D DC      MOVSX ECX,WORD PTR SS:[EBP-24]  

0043A226   . 51             PUSH ECX  

0043A227   . 8B55 08        MOV EDX,DWORD PTR SS:[EBP+8]  

0043A22A   . 8B02           MOV EAX,DWORD PTR DS:[EDX]  

0043A22C   . 50             PUSH EAX  

0043A22D   . FF15 00114000  CALL rtcMidCharBStr        ;又来一遍（真笨）  

0043A233   . 8BD0           MOV EDX,EAX  

0043A235   . 8D4D AC        LEA ECX,DWORD PTR SS:[EBP-54]  

0043A238   . FF15 C8124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>]        ;  MSVBVM60.__vbaStrMove  

0043A23E   . 50             PUSH EAX  

0043A23F   . FF15 58104000  CALL MSVBVM60.rtcAnsivalueBstr   ;  该字符的 ascii   

0043A245   . 66:8985 40FFFF>MOV WORD PTR SS:[EBP-C0],AX      ；存入此处  

0043A24C   . 66:8B8D 48FFFF>MOV CX,WORD PTR SS:[EBP-B8]      ；第一遍取出的结果  

0043A253   . 66:0FAF8D 44FF>IMUL CX,WORD PTR SS:[EBP-BC]     ；乘以第二编的结果  

0043A25B   . 0F80 FC060000  JO CollegeB.0043A95D  

0043A261   . 0FBFD1         MOVSX EDX,CX  

0043A264   . 8995 08FFFFFF  MOV DWORD PTR SS:[EBP-F8],EDX    ；结果存在这里  

0043A26A   . DB85 08FFFFFF  FILD DWORD PTR SS:[EBP-F8]        ；作为整数载入  

0043A270   . DD9D 00FFFFFF  FSTP QWORD PTR SS:[EBP-100]       ；存成浮点  

0043A276   . DD45 D4        FLD QWORD PTR SS:[EBP-2C]         ；加到总和中  

0043A279   . DC85 00FFFFFF  FADD QWORD PTR SS:[EBP-100]       

0043A27F   . DFE0           FSTSW AX  

0043A281   . A8 0D          TEST AL,0D  

0043A283   . 0F85 CF060000  JNZ CollegeB.0043A958  

0043A289   . DD9D F8FEFFFF  FSTP QWORD PTR SS:[EBP-108]       ；结果暂时存在此处  

0043A28F   . 68 00000040    PUSH 40000000  

0043A294   . 6A 00          PUSH 0                              ; 在堆栈上构成浮点数 2.0  

0043A296   . 0FBF85 40FFFFF>MOVSX EAX,WORD PTR SS:[EBP-C0]    ；第三遍的结果  

0043A29D   . 8985 F4FEFFFF  MOV DWORD PTR SS:[EBP-10C],EAX  

0043A2A3   . DB85 F4FEFFFF  FILD DWORD PTR SS:[EBP-10C]       ;   

0043A2A9   . DD9D ECFEFFFF  FSTP QWORD PTR SS:[EBP-114]       ;   

0043A2AF   . 8B8D F0FEFFFF  MOV ECX,DWORD PTR SS:[EBP-110]  

0043A2B5   . 51             PUSH ECX  

0043A2B6   . 8B95 ECFEFFFF  MOV EDX,DWORD PTR SS:[EBP-114]  

0043A2BC   . 52             PUSH EDX                             

0043A2BD   . FF15 68124000  CALL MSVBVM60.__vbaPowerR8       ; 求平方  

0043A2C3   . DC85 F8FEFFFF  FADD QWORD PTR SS:[EBP-108]       ；加到上面的结果中  

0043A2C9   . DD5D D4        FSTP QWORD PTR SS:[EBP-2C]        ; 总数存回[ebp-2c]
```

以上代码实际上计算 `x*x+x**2 = 2 * x**2`。

.................................................................

```text
0043A307   . C745 FC 050000>MOV DWORD PTR SS:[EBP-4],5  

0043A30E   . DD45 D4        FLD QWORD PTR SS:[EBP-2C]          ;结果装入  

0043A311   . FF15 A8124000  CALL MSVBVM60.__vbaFpI4   ；转成长整型  

0043A317   . 8BF0           MOV ESI,EAX                   

0043A319   . 81E6 01000080  AND ESI,80000001   ；检查最高位和最低位  

0043A31F   . 79 05          JNS SHORT CollegeB.0043A326  

0043A321   . 4E             DEC ESI  

0043A322   . 83CE FE        OR ESI,FFFFFFFE  

0043A325   . 46             INC ESI  

0043A326   > F7DE           NEG ESI  

0043A328   . 1BF6           SBB ESI,ESI  

0043A32A   . F7DE           NEG ESI        ；最后反映的只是最低位，即奇偶位  

0043A32C   . DD45 D4        FLD QWORD PTR SS:[EBP-2C]          ；再次装入运算结果  

0043A32F   . FF15 A8124000  CALL MSVBVM60.__vbaFpI4 ；再次转型(VB 真笨得可以)  

0043A335   . 99             CDQ       ；符号扩展  

0043A336   . B9 03000000    MOV ECX,3  

0043A33B   . F7F9           IDIV ECX       ；除以 3  

0043A33D   . F7DA           NEG EDX    ；余数变号  

0043A33F   . 1BD2           SBB EDX,EDX  ；借位减  

0043A341   . F7DA           NEG EDX      ；变号  

0043A343   . 23F2           AND ESI,EDX ；和上面的结果按位与  

0043A345   . 85F6           TEST ESI,ESI      ；这里的条件是“奇数且不被 3 整除”  

0043A347   . 75 40          JNZ SHORT CollegeB.0043A389 ；条件满足则继续  

0043A349   . C745 FC 060000>MOV DWORD PTR SS:[EBP-4],6  ；这里不知道是什么标志   

0043A350   . 66:8B55 DC     MOV DX,WORD PTR SS:[EBP-24] ；条件不满足则取字符索引  

0043A354   . 66:6BD2 03     IMUL DX,DX,3               ；乘以 3  

0043A358   . 0F80 FF050000  JO CollegeB.0043A95D  

0043A35E   . 0FBFC2         MOVSX EAX,DX               ；并加到结果中  

0043A361   . 8985 E8FEFFFF  MOV DWORD PTR SS:[EBP-118],EAX  

0043A367   . DB85 E8FEFFFF  FILD DWORD PTR SS:[EBP-118]  

0043A36D   . DD9D E0FEFFFF  FSTP QWORD PTR SS:[EBP-120]  

0043A373   . DD45 D4        FLD QWORD PTR SS:[EBP-2C]  

0043A376   . DC85 E0FEFFFF  FADD QWORD PTR SS:[EBP-120]  

0043A37C   . DD5D D4        FSTP QWORD PTR SS:[EBP-2C]                            ;  ebp-2c again  

0043A37F   . DFE0           FSTSW AX  

0043A381   . A8 0D          TEST AL,0D  

0043A383   . 0F85 CF050000  JNZ CollegeB.0043A958  

0043A389   > C745 FC 080000>MOV DWORD PTR SS:[EBP-4],8  

0043A390   .^E9 D0FDFFFF    JMP CollegeB.0043A165       ；下一轮
```

以上代码从名字的第一个字符开始计算每一个字符的 `2*x**2`，并将结果相加。每一步的结果如果不满足条件 “奇数且不被 3 整除” 就再加上当前索引值的 3 倍。直到每一个字符都处理完。结果存在 `[ebp-2c]` 处。

下面的代码又进行了一轮完全同样的计算，结果也完全相同，存在 `[ebp-44]` 处。令人费解的重复劳动。省略。接下去：

```text
0043A626   > C745 FC 0F0000>MOV DWORD PTR SS:[EBP-4],0F  

0043A62D   . 8B4D C0        MOV ECX,DWORD PTR SS:[EBP-40]  

0043A630   . 51             PUSH ECX  

0043A631   . 8B55 BC        MOV EDX,DWORD PTR SS:[EBP-44]     ；取的（第二次计算）计算结果  

0043A634   . 52             PUSH EDX  
```
