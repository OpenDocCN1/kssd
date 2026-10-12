
0047DE66&nbsp;&nbsp;. 0F80 C7040000 JO 扩展名仓.0047E333

0047DE6C&nbsp;&nbsp;. 8B1C98&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV EBX,DWORD PTR DS:[EAX+EBX*4]&nbsp;&nbsp;〈---- EBX 分别 =E0 ， D7 ，87 ，2C ，8C ，〈---- 这里是用户名计算后的几个值 。

0047DE6F&nbsp;&nbsp;. 031CB8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ADD EBX,DWORD PTR DS:[EAX+EDI*4]&nbsp;&nbsp;〈---- 两两相加，就是 E0 + D7 ，D7 + 87 ，87 + 2C ，2C + 8C

0047DE72&nbsp;&nbsp;. 0F80 BB040000 JO 扩展名仓.0047E333

0047DE78&nbsp;&nbsp;. 03D3&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ADD EDX,EBX

0047DE7A&nbsp;&nbsp;. 0F80 B3040000 JO 扩展名仓.0047E333

0047DE80&nbsp;&nbsp;. 83C2 01&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ADD EDX,1

0047DE83&nbsp;&nbsp;. 0F80 AA040000 JO 扩展名仓.0047E333

0047DE89&nbsp;&nbsp;. 03CA&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ADD ECX,EDX

0047DE8B&nbsp;&nbsp;. 0F80 A2040000 JO 扩展名仓.0047E333

0047DE91&nbsp;&nbsp;. FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI4Abs>]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; msvbvm60.\_\_vbaI4Abs

0047DE97&nbsp;&nbsp;. 8BC8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV ECX,EAX

0047DE99&nbsp;&nbsp;. FF15 64114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI2I4>]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; msvbvm60.\_\_vbaI2I4

0047DE9F&nbsp;&nbsp;. 8B5D EC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV EBX,DWORD PTR SS:[EBP-14]

0047DEA2&nbsp;&nbsp;. EB 5E&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP SHORT 扩展名仓.0047DF02

0047DEA4&nbsp;&nbsp;&gt;&nbsp;83FF 05&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CMP EDI,5

0047DEA7&nbsp;&nbsp;. 72 11&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JB SHORT 扩展名仓.0047DEBA

0047DEA9&nbsp;&nbsp;. FF15 40114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaGenerateBoundsEr>; msvbvm60.\_\_vbaGenerateBoundsError

0047DEAF&nbsp;&nbsp;. 83FF 05&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CMP EDI,5

0047DEB2&nbsp;&nbsp;. 72 06&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JB SHORT 扩展名仓.0047DEBA

0047DEB4&nbsp;&nbsp;. FF15 40114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaGenerateBoundsEr>; msvbvm60.\_\_vbaGenerateBoundsError

0047DEBA&nbsp;&nbsp;&gt;&nbsp;8B0D 18514A00 MOV ECX,DWORD PTR DS:[4A5118]

0047DEC0&nbsp;&nbsp;. 8B11&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV EDX,DWORD PTR DS:[ECX]

0047DEC2&nbsp;&nbsp;. 52&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EDX

0047DEC3&nbsp;&nbsp;. FFD6&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CALL ESI

0047DEC5&nbsp;&nbsp;. 8B4D E0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV ECX,DWORD PTR SS:[EBP-20]

0047DEC8&nbsp;&nbsp;. 8B55 BC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV EDX,DWORD PTR SS:[EBP-44]

0047DECB&nbsp;&nbsp;. 0FBFC0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOVSX EAX,AX

0047DECE&nbsp;&nbsp;. 8B0CB9&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV ECX,DWORD PTR DS:[ECX+EDI*4]

0047DED1&nbsp;&nbsp;. 0FAFCF&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IMUL ECX,EDI

0047DED4&nbsp;&nbsp;. 0F80 59040000 JO 扩展名仓.0047E333

0047DEDA&nbsp;&nbsp;. 030CBA&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ADD ECX,DWORD PTR DS:[EDX+EDI*4]

0047DEDD&nbsp;&nbsp;. 0F80 50040000 JO 扩展名仓.0047E333

0047DEE3&nbsp;&nbsp;. 83C1 4D&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ADD ECX,4D

0047DEE6&nbsp;&nbsp;. 0F80 47040000 JO 扩展名仓.0047E333

0047DEEC&nbsp;&nbsp;. 2BC8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SUB ECX,EAX

0047DEEE&nbsp;&nbsp;. 0F80 3F040000 JO 扩展名仓.0047E333

0047DEF4&nbsp;&nbsp;. FF15 CC104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI4Abs>]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; msvbvm60.\_\_vbaI4Abs

0047DEFA&nbsp;&nbsp;. 8BC8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV ECX,EAX

0047DEFC&nbsp;&nbsp;. FF15 64114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaI2I4>]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; msvbvm60.\_\_vbaI2I4

0047DF02&nbsp;&nbsp;&gt;&nbsp;66:3D 3000&nbsp;&nbsp;&nbsp;CMP AX,30

0047DF06&nbsp;&nbsp;. 7C 06&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JL SHORT 扩展名仓.0047DF0E

0047DF08&nbsp;&nbsp;. 66:3D 3900&nbsp;&nbsp;&nbsp;CMP AX,39

0047DF0C&nbsp;&nbsp;. 7E 0C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JLE SHORT 扩展名仓.0047DF1A

0047DF0E&nbsp;&nbsp;&gt;&nbsp;66:3D 6100&nbsp;&nbsp;&nbsp;CMP AX,61

0047DF12&nbsp;&nbsp;. 7C 5C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JL SHORT 扩展名仓.0047DF70

0047DF14&nbsp;&nbsp;. 66:3D 7A00&nbsp;&nbsp;&nbsp;CMP AX,7A

0047DF18&nbsp;&nbsp;. 7F 56&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JG SHORT 扩展名仓.0047DF70

0047DF1A&nbsp;&nbsp;&gt;&nbsp;0FBFC0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOVSX EAX,AX

0047DF1D&nbsp;&nbsp;. 8D4D 8C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA ECX,DWORD PTR SS:[EBP-74]

0047DF20&nbsp;&nbsp;. 50&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EAX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; /Arg2

0047DF21&nbsp;&nbsp;. 51&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH ECX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; |Arg1

0047DF22&nbsp;&nbsp;. FF15 E0114000 CALL DWORD PTR DS:[<&msvbvm60.rtcVarBstrFromAnsi>]&nbsp;&nbsp;; \\rtcVarBstrFromAnsi

0047DF28&nbsp;&nbsp;. 8D55 8C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA EDX,DWORD PTR SS:[EBP-74]

0047DF2B&nbsp;&nbsp;. 8D85 7CFFFFFF LEA EAX,DWORD PTR SS:[EBP-84]

0047DF31&nbsp;&nbsp;. 52&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EDX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; /Arg2

0047DF32&nbsp;&nbsp;. 50&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EAX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; |Arg1

0047DF33&nbsp;&nbsp;. FF15 3C114000 CALL DWORD PTR DS:[<&msvbvm60.rtcUpperCaseVar>]&nbsp;&nbsp;&nbsp;; \\rtcUpperCaseVar

0047DF39&nbsp;&nbsp;. 8D8D 7CFFFFFF LEA ECX,DWORD PTR SS:[EBP-84]

0047DF3F&nbsp;&nbsp;. 8D55 A0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA EDX,DWORD PTR SS:[EBP-60]

0047DF42&nbsp;&nbsp;. 51&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH ECX

0047DF43&nbsp;&nbsp;. 52&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EDX

0047DF44&nbsp;&nbsp;. FF15 F8114000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaStrVarVal>]&nbsp;&nbsp;&nbsp;&nbsp;; msvbvm60.\_\_vbaStrVarVal

0047DF4A&nbsp;&nbsp;. 50&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EAX

0047DF4B&nbsp;&nbsp;. FFD6&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CALL ESI

0047DF4D&nbsp;&nbsp;. 8D4D A0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA ECX,DWORD PTR SS:[EBP-60]

0047DF50&nbsp;&nbsp;. 8BF8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV EDI,EAX

0047DF52&nbsp;&nbsp;. FF15 04134000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeStr>]&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; msvbvm60.\_\_vbaFreeStr

0047DF58&nbsp;&nbsp;. 8D85 7CFFFFFF LEA EAX,DWORD PTR SS:[EBP-84]

0047DF5E&nbsp;&nbsp;. 8D4D 8C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA ECX,DWORD PTR SS:[EBP-74]

0047DF61&nbsp;&nbsp;. 50&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EAX

0047DF62&nbsp;&nbsp;. 51&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH ECX

0047DF63&nbsp;&nbsp;. 6A 02&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH 2

0047DF65&nbsp;&nbsp;. FF15 44104000 CALL DWORD PTR DS:[<&msvbvm60.\_\_vbaFreeVarList>]&nbsp;&nbsp;&nbsp;; msvbvm60.\_\_vbaFreeVarList

0047DF6B&nbsp;&nbsp;. 83C4 0C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ADD ESP,0C

0047DF6E&nbsp;&nbsp;. EB 7C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP SHORT 扩展名仓.0047DFEC

0047DF70&nbsp;&nbsp;&gt;&nbsp;66:99&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CWD

0047DF72&nbsp;&nbsp;. 66:B9 7B00&nbsp;&nbsp;&nbsp;MOV CX,7B

0047DF76&nbsp;&nbsp;. 66:F7F9&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IDIV CX

0047DF79&nbsp;&nbsp;. 8BFA&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV EDI,EDX

0047DF7B&nbsp;&nbsp;. 66:83FF 09&nbsp;&nbsp;&nbsp;CMP DI,9

0047DF7F&nbsp;&nbsp;. 7D 06&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JGE SHORT 扩展名仓.0047DF87

0047DF81&nbsp;&nbsp;. 66:83C7 30&nbsp;&nbsp;&nbsp;ADD DI,30

0047DF85&nbsp;&nbsp;. EB 5F&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP SHORT 扩展名仓.0047DFE6

0047DF87&nbsp;&nbsp;&gt;&nbsp;66:83FF 22&nbsp;&nbsp;&nbsp;CMP DI,22

0047DF8B&nbsp;&nbsp;. 7F 06&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JG SHORT 扩展名仓.0047DF93

0047DF8D&nbsp;&nbsp;. 66:83C7 58&nbsp;&nbsp;&nbsp;ADD DI,58

0047DF91&nbsp;&nbsp;. EB 53&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP SHORT 扩展名仓.0047DFE6

0047DF93&nbsp;&nbsp;&gt;&nbsp;66:83FF 2C&nbsp;&nbsp;&nbsp;CMP DI,2C

0047DF97&nbsp;&nbsp;. 7D 06&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JGE SHORT 扩展名仓.0047DF9F

0047DF99&nbsp;&nbsp;. 66:83C7 0D&nbsp;&nbsp;&nbsp;ADD DI,0D

0047DF9D&nbsp;&nbsp;. EB 47&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP SHORT 扩展名仓.0047DFE6

0047DF9F&nbsp;&nbsp;&gt;&nbsp;66:83FF 30&nbsp;&nbsp;&nbsp;CMP DI,30

0047DFA3&nbsp;&nbsp;. 7C 06&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JL SHORT 扩展名仓.0047DFAB

0047DFA5&nbsp;&nbsp;. 66:83FF 4D&nbsp;&nbsp;&nbsp;CMP DI,4D

0047DFA9&nbsp;&nbsp;. 75 07&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JNZ SHORT 扩展名仓.0047DFB2

0047DFAB&nbsp;&nbsp;&gt;&nbsp;BF 48000000&nbsp;&nbsp;MOV EDI,48

0047DFB0&nbsp;&nbsp;. EB 3A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP SHORT 扩展名仓.0047DFEC

0047DFB2&nbsp;&nbsp;&gt;&nbsp;66:83FF 39&nbsp;&nbsp;&nbsp;CMP DI,39

0047DFB6&nbsp;&nbsp;. 7E 0C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JLE SHORT 扩展名仓.0047DFC4

0047DFB8&nbsp;&nbsp;. 66:83FF 4C&nbsp;&nbsp;&nbsp;CMP DI,4C

0047DFBC&nbsp;&nbsp;. 7F 06&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JG SHORT 扩展名仓.0047DFC4

0047DFBE&nbsp;&nbsp;. 66:83C7 2A&nbsp;&nbsp;&nbsp;ADD DI,2A

0047DFC2&nbsp;&nbsp;. EB 22&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP SHORT 扩展名仓.0047DFE6

0047DFC4&nbsp;&nbsp;&gt;&nbsp;66:83FF 4D&nbsp;&nbsp;&nbsp;CMP DI,4D

0047DFC8&nbsp;&nbsp;. 7E 0C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JLE SHORT 扩展名仓.0047DFD6

0047DFCA&nbsp;&nbsp;. 66:83FF 57&nbsp;&nbsp;&nbsp;CMP DI,57

0047DFCE&nbsp;&nbsp;. 7F 0C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JG SHORT 扩展名仓.0047DFDC

0047DFD0&nbsp;&nbsp;. 66:83EF 1E&nbsp;&nbsp;&nbsp;SUB DI,1E

0047DFD4&nbsp;&nbsp;. EB 10&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP SHORT 扩展名仓.0047DFE6

0047DFD6&nbsp;&nbsp;&gt;&nbsp;66:83FF 57&nbsp;&nbsp;&nbsp;CMP DI,57

0047DFDA&nbsp;&nbsp;. 7E 10&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JLE SHORT 扩展名仓.0047DFEC

0047DFDC&nbsp;&nbsp;&gt;&nbsp;66:83FF 61&nbsp;&nbsp;&nbsp;CMP DI,61

0047DFE0&nbsp;&nbsp;. 7D 0A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JGE SHORT 扩展名仓.0047DFEC

0047DFE2&nbsp;&nbsp;. 66:83C7 11&nbsp;&nbsp;&nbsp;ADD DI,11

0047DFE6&nbsp;&nbsp;&gt;&nbsp;0F80 47030000 JO 扩展名仓.0047E333

0047DFEC&nbsp;&nbsp;&gt;&nbsp;0FBFD7&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOVSX EDX,DI

0047DFEF&nbsp;&nbsp;. 8D45 8C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA EAX,DWORD PTR SS:[EBP-74]

0047DFF2&nbsp;&nbsp;. 52&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EDX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; /Arg2

0047DFF3&nbsp;&nbsp;. 50&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EAX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; |Arg1

0047DFF4&nbsp;&nbsp;. FF15 E0114000 CALL DWORD PTR DS:[<&msvbvm60.rtcVarBstrFromAnsi>]&nbsp;&nbsp;; \\rtcVarBstrFromAnsi

0047DFFA&nbsp;&nbsp;. 8B3D 3C114000 MOV EDI,DWORD PTR DS:[<&msvbvm60.rtcUpperCaseVar>]&nbsp;&nbsp;; msvbvm60.rtcUpperCaseVar

0047E000&nbsp;&nbsp;. 8D4D 8C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA ECX,DWORD PTR SS:[EBP-74]

0047E003&nbsp;&nbsp;. 8D95 7CFFFFFF LEA EDX,DWORD PTR SS:[EBP-84]

0047E009&nbsp;&nbsp;. 51&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH ECX

0047E00A&nbsp;&nbsp;. 52&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH EDX

0047E00B&nbsp;&nbsp;. FFD7&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CALL EDI&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;; &<msvbvm60.rtcUpperCaseVar>&nbsp;&nbsp;〈---- 转为大写

0047E00D&nbsp;&nbsp;. A1 18514A00&nbsp;&nbsp;MOV EAX,DWORD PTR DS:[4A5118]

0047E012&nbsp;&nbsp;. 66:8BD3&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV DX,BX

0047E015&nbsp;&nbsp;. 83C0 04&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ADD EAX,4

0047E018&nbsp;&nbsp;. 66:83C2 01&nbsp;&nbsp;&nbsp;ADD DX,1

0047E01C&nbsp;&nbsp;. 8985 44FFFFFF MOV DWORD PTR SS:[EBP-BC],EAX

0047E022&nbsp;&nbsp;. 8D8D 6CFFFFFF LEA ECX,DWORD PTR SS:[EBP-94]

0047E028&nbsp;&nbsp;. 0F80 05030000 JO 扩展名仓.0047E333

0047E02E&nbsp;&nbsp;. 0FBFC2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOVSX EAX,DX
