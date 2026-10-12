
0043A635  . FF15 80114000  CALL `\[<&MSVBVM60.__vbaStrR8>\]`    ；R8 转成 string  

0043A63B  . 8BD0          MOV EDX,EAX  

0043A63D  . 8D4D B4       LEA ECX,DWORD PTR SS:`\[EBP-4C\]`  

0043A640  . FF15 C8124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaStrMove>\]`      ;  `MSVBVM60.__vbaStrMove`  

0043A646  . 50          PUSH EAX  

0043A647  . FF15 D0114000  CALL `rtcStrReverse`     ;字符串逆序  

0043A64D  . 8BD0          MOV EDX,EAX  

0043A64F  . 8D4D B0       LEA ECX,DWORD PTR SS:`\[EBP-50\]`  

0043A652  . FF15 C8124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaStrMove>\]`   

0043A658  . 50          PUSH EAX  

0043A659  . FF15 3C124000  CALL `\[<&MSVBVM60.__vbaR8Str>\]`   ;转回 R8  

0043A65F  . DC05 381B4000  FADD QWORD PTR DS:`\[401B38\]`    ；加上此处的内定值 38473.0  

0043A665  . DD5D BC       FSTP QWORD PTR SS:`\[EBP-44\]`    ；存回结果  

............................................................  

0043A685  . C745 FC 100000>MOV DWORD PTR SS:`\[EBP-4\]`,10  

0043A68C  . 8B55 C0       MOV EDX,DWORD PTR SS:`\[EBP-40\]`  

0043A68F  . 52          PUSH EDX  

0043A690  . 8B45 BC       MOV EAX,DWORD PTR SS:`\[EBP-44\]`  

0043A693  . 50          PUSH EAX  

0043A694  . FF15 80114000  CALL `\[<&MSVBVM60.__vbaStrR8>\]`  ;又转成字符串  

0043A69A  . 8BD0          MOV EDX,EAX  

0043A69C  . 8D4D B4       LEA ECX,DWORD PTR SS:`\[EBP-4C\]`  

0043A69F  . FF15 C8124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaStrMove>\]`   

0043A6A5  . 50          PUSH EAX  

0043A6A6  . FF15 D0114000  CALL `MSVBVM60.rtcStrReverse`  ；逆序  

0043A6AC  . 8BD0          MOV EDX,EAX  

0043A6AE  . 8D4D B0       LEA ECX,DWORD PTR SS:`\[EBP-50\]`  

0043A6B1  . FF15 C8124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaStrMove>\]`     

0043A6B7  . 50          PUSH EAX  

0043A6B8  . FF15 3C124000  CALL `\[<&MSVBVM60.__vbaR8Str>\]`  ；又回到 R8  

0043A6BE  . DCC0          FADD ST(0),ST(0)            ;加倍  

0043A6C0  . DD5D BC       FSTP QWORD PTR SS:`\[EBP-44\]`     ；存回  

.......................................................................  

0043A6E0  . C745 FC 110000>MOV DWORD PTR SS:`\[EBP-4\]`,11  

0043A6E7  . 8B45 D8       MOV EAX,DWORD PTR SS:`\[EBP-28\]`    ;取第一次计算结果  

0043A6EA  . 50          PUSH EAX  

0043A6EB  . 8B4D D4       MOV ECX,DWORD PTR SS:`\[EBP-2C\]`  

0043A6EE  . 51          PUSH ECX  

0043A6EF  . FF15 80114000  CALL `MSVBVM60.__vbaStrR8` ；转成 String  

0043A6F5  . 8BD0          MOV EDX,EAX  

0043A6F7  . 8D4D B4       LEA ECX,DWORD PTR SS:`\[EBP-4C\]`  

0043A6FA  . FF15 C8124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaStrMove>\]`   

0043A700  . 50          PUSH EAX  

0043A701  . FF15 D0114000  CALL  `MSVBVM60.rtcStrReverse` ;逆序  

0043A707  . 8BD0          MOV EDX,EAX  

0043A709  . 8D4D B0       LEA ECX,DWORD PTR SS:`\[EBP-50\]`  

0043A70C  . FF15 C8124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaStrMove>\]`   

0043A712  . 50          PUSH EAX  

0043A713  . FF15 3C124000  CALL `MSVBVM60.__vbaR8Str` ；转回 R8  

0043A719  . DD5D D4       FSTP QWORD PTR SS:`\[EBP-2C\]` ；存回  

......................................................................  

0043A736  . 68 00000040   PUSH 40000000  

0043A73B  . 6A 00         PUSH 0          ;  浮点数 2.0  

0043A73D  . 8B4D D8       MOV ECX,DWORD PTR SS:`\[EBP-28\]`  

0043A740  . 51          PUSH ECX  

0043A741  . 8B55 D4       MOV EDX,DWORD PTR SS:`\[EBP-2C\]`  

0043A744  . 52          PUSH EDX  

0043A745  . FF15 68124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaPowerR8>\]` ;  结果平方  

0043A74B  . DD5D D4       FSTP QWORD PTR SS:`\[EBP-2C\]`               ;  存回  

0043A74E  . C745 FC 130000>MOV DWORD PTR SS:`\[EBP-4\]`,13  

0043A755  . C745 A4 0C0000>MOV DWORD PTR SS:`\[EBP-5C\]`,0C  

0043A75C  . C745 9C 020000>MOV DWORD PTR SS:`\[EBP-64\]`,2  

0043A763  . 8D45 9C       LEA EAX,DWORD PTR SS:`\[EBP-64\]`    ;  var type integer value 0C  

0043A766  . 50          PUSH EAX  

0043A767  . 6A 05         PUSH 5  

0043A769  . 8B4D 0C       MOV ECX,DWORD PTR SS:`\[EBP+C\]`   ；注册码  

0043A76C  . 8B11          MOV EDX,DWORD PTR DS:`\[ECX\]`  

0043A76E  . 52          PUSH EDX  

0043A76F  . FF15 00114000  CALL `rtcMidCharBStr`    ;取注册码子串，从第 5 个字符开始，取 0xC 个字符  

0043A775  . 8BD0          MOV EDX,EAX          

0043A777  . 8D4D B4       LEA ECX,DWORD PTR SS:`\[EBP-4C\]`  

0043A77A  . FF15 C8124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaStrMove>\]`     

0043A780  . 50          PUSH EAX  

0043A781  . FF15 3C124000  CALL `\[<&MSVBVM60.__vbaR8Str>\]`        ;  转成 R8  

0043A787  . DD5D C4       FSTP QWORD PTR SS:`\[EBP-3C\]`        ;存在 `ebp-3C` 处  

.................................................................  

0043A79C  . C745 FC 140000>MOV DWORD PTR SS:`\[EBP-4\]`,14  

0043A7A3  . DD45 C4       FLD QWORD PTR SS:`\[EBP-3C\]`    ; 该子串表示的实数  

0043A7A6  . DC05 301B4000  FADD QWORD PTR DS:`\[401B30\]`   ；加上内定值 334.0  

0043A7AC  . DD5D C4       FSTP QWORD PTR SS:`\[EBP-3C\]`   ；存回  

0043A7AF  . DFE0          FSTSW AX  

0043A7B1  . A8 0D         TEST AL,0D  

0043A7B3  . 0F85 9F010000  JNZ `CollegeB.0043A958`  

0043A7B9  . C745 FC 150000>MOV DWORD PTR SS:`\[EBP-4\]`,15  

0043A7C0  . 8B45 C8       MOV EAX,DWORD PTR SS:`\[EBP-38\]`  

0043A7C3  . 50          PUSH EAX  

0043A7C4  . 8B4D C4       MOV ECX,DWORD PTR SS:`\[EBP-3C\]`  

0043A7C7  . 51          PUSH ECX  

0043A7C8  . FF15 80114000  CALL `\[<&MSVBVM60.__vbaStrR8>\]`    ;转成 String  

0043A7CE  . 8BD0          MOV EDX,EAX  

0043A7D0  . 8D4D B4       LEA ECX,DWORD PTR SS:`\[EBP-4C\]`  

0043A7D3  . FF15 C8124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaStrMove>\]`     

0043A7D9  . 50          PUSH EAX  

0043A7DA  . FF15 D0114000  CALL  `MSVBVM60.rtcStrReverse` ;逆序  

0043A7E0  . 8BD0          MOV EDX,EAX  

0043A7E2  . 8D4D B0       LEA ECX,DWORD PTR SS:`\[EBP-50\]`  

0043A7E5  . FF15 C8124000  CALL DWORD PTR DS:`\[<&MSVBVM60.__vbaStrMove>\]`     ;  `MSVBVM60.__vbaStrMove`  

0043A7EB  . 50          PUSH EAX  

0043A7EC  . FF15 3C124000  CALL `MSVBVM60.__vbaR8Str` ；转回 R8  

0043A7F2  . DD5D C4       FSTP QWORD PTR SS:`\[EBP-3C\]`     ；存回  

....................................................................  

0043A808  . C745 FC 160000>MOV DWORD PTR SS:`\[EBP-4\]`,16  

0043A80F  . DD45 C4       FLD QWORD PTR SS:`\[EBP-3C\]`       ;装入结果  

0043A812  . DC25 281B4000  FSUB QWORD PTR DS:`\[401B28\]`      ; 减去内定值 1032.0  

0043A818  . DD5D C4       FSTP QWORD PTR SS:`\[EBP-3C\]`      ; 存回  

0043A81B  . DFE0          FSTSW AX  

0043A81D  . A8 0D         TEST AL,0D  

0043A81F  . 0F85 33010000  JNZ `CollegeB.0043A958`  

0043A825  . C745 FC 170000>MOV DWORD PTR SS:`\[EBP-4\]`,17  

0043A82C  . DD45 C4       FLD QWORD PTR SS:`\[EBP-3C\]`       ;  这里的结果  

0043A82F  . DC5D D4       FCOMP QWORD PTR SS:`\[EBP-2C\]`      ;  与`[ebp-2c]`处的结果作实数比较  

0043A832  . DFE0          FSTSW AX  

0043A834  . F6C4 40       TEST AH,40                       ;  相等？  

0043A837  . 74 0D         JE SHORT `CollegeB.0043A846`    ;  if not equal then jmp  

0043A839  . C745 FC 180000>MOV DWORD PTR SS:`\[EBP-4\]`,18  

0043A840  . 66:C745 D0 FFF>MOV WORD PTR SS:`\[EBP-30\]`,0FFFF  ;  若相等则`[ebp-30]=-1`，看上去应该是成功标志  

0043A846  > C745 FC 1A0000>MOV DWORD PTR SS:`\[EBP-4\]`,1A    ;  但随后的代码使之无论如何不为-1  

0043A84D  . 66:C785 28FFFF>MOV WORD PTR SS:`\[EBP-D8\]`,4  

0043A856  . 66:C785 2CFFFF>MOV WORD PTR SS:`\[EBP-D4\]`,1  

0043A85F  . 66:C745 DC 010>MOV WORD PTR SS:`\[EBP-24\]`,1  

0043A865  . EB 15         JMP SHORT `CollegeB.0043A87C`  

0043A867  > 66:8B4D DC    MOV CX,WORD PTR SS:`\[EBP-24\]`  

0043A86B  . 66:038D 2CFFFF>ADD CX,WORD PTR SS:`\[EBP-D4\]`  

0043A872  . 0F80 E5000000  JO `CollegeB.0043A95D`  

0043A878  . 66:894D DC    MOV WORD PTR SS:`\[EBP-24\]`,CX  

0043A87C  > 66:8B55 DC    MOV DX,WORD PTR SS:`\[EBP-24\]`  
