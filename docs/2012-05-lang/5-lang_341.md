
# 万能转换 2.2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6064.htm`](https://www.pediy.com/kssd/pediy06/pediy6064.htm)

对象：万能转换 2.2.0

PJ 人：`lordor[BCG]`

目的：初学破解，属技术交流，无其它目的，请不要任意散布或用用商业用途。

工具：`ollydbg1.09B`

假设：

注册名称：`lordor[BCG]`

机器码：`TNXS-OVBR-83966`

注册码：`654321`

奇怪，每次选注册框，软件的机器码是每次不一样的

用`ollydbg`载入程序，动态跟踪如下：

```
01029885   .  C785 48FFFFFF>MOV DWORD PTR SS:[EBP-B8],0

0102988F   .  8B8D 74FEFFFF MOV ECX,DWORD PTR SS:[EBP-18C]

01029895   .  898D 34FFFFFF MOV DWORD PTR SS:[EBP-CC],ECX

0102989B   .  C785 2CFFFFFF>MOV DWORD PTR SS:[EBP-D4],8

010298A5   .  6A 05          PUSH 5

010298A7   .  8D95 2CFFFFFF LEA EDX,DWORD PTR SS:[EBP-D4]

010298AD   .  52            PUSH EDX

010298AE   .  8D85 1CFFFFFF LEA EAX,DWORD PTR SS:[EBP-E4]         

;  机器码入 eax

010298B4   .  50            PUSH EAX

010298B5   .  FF15 F0124000 CALL DWORD PTR DS:[<&MSVBVM60.#619>]   

;  MSVBVM60.rtcRightCharVar

010298BB   .  8D8D 1CFFFFFF LEA ECX,DWORD PTR SS:[EBP-E4]         

;  取机器码右边 5 位，即为数字 83966

010298C1   .  51            PUSH ECX

010298C2   .  FF15 30134000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaI4ErrVa>;  MSVBVM60.__vbaI4ErrVar

010298C8   .  8985 78FFFFFF MOV DWORD PTR SS:[EBP-88],EAX

010298CE   .  8D8D 40FFFFFF LEA ECX,DWORD PTR SS:[EBP-C0]

010298D4   .  FF15 34134000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeObj>;  MSVBVM60.__vbaFreeObj

010298DA   .  8D95 1CFFFFFF LEA EDX,DWORD PTR SS:[EBP-E4]

010298E0   .  52            PUSH EDX

010298E1   .  8D85 1CFFFFFF LEA EAX,DWORD PTR SS:[EBP-E4]

010298E7   .  50            PUSH EAX

010298E8   .  8D8D 2CFFFFFF LEA ECX,DWORD PTR SS:[EBP-D4]

010298EE   .  51            PUSH ECX

010298EF   .  6A 03         PUSH 3

010298F1   .  FF15 5C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeVar>;  MSVBVM60.__vbaFreeVarList

010298F7   .  83C4 10       ADD ESP,10

010298FA   .  C745 FC 09000>MOV DWORD PTR SS:[EBP-4],9

01029901   .  DB85 78FFFFFF FILD DWORD PTR SS:[EBP-88]             

;  83966 入 st0

01029907   .  DD9D 5CFEFFFF FSTP QWORD PTR SS:[EBP-1A4]            

;  st0 的值保存入[ebp-1a4]

0102990D   .  8B95 60FEFFFF MOV EDX,DWORD PTR SS:[EBP-1A0]

01029913   .  52            PUSH EDX

01029914   .  8B85 5CFEFFFF MOV EAX,DWORD PTR SS:[EBP-1A4]

0102991A   .  50            PUSH EAX

0102991B   .  FF15 B0124000 CALL DWORD PTR DS:[<&MSVBVM60.#614>]   

;  MSVBVM60.rtcSqr

01029921   .  DD9D A8FEFFFF FSTP QWORD PTR SS:[EBP-158]            

;  求 83966 的平方根，保存到[ebp-158]

01029927   .  DD05 B0234000 FLD QWORD PTR DS:[4023B0]             

;  装入 12345 入 st0

0102992D   .  DC8D A8FEFFFF FMUL QWORD PTR SS:[EBP-158]            

;  83966 的平方根的值与 12345 相乘，为 3577197.7

01029933   .  DFE0          FSTSW AX                               

;  保存状态字节到 eax

01029935   .  A8 0D         TEST AL,0D

01029937   .  0F85 EE160000 JNZ 万能转换.0102B02B

0102993D   .  FF15 BC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFpI4>]   

;  MSVBVM60.__vbaFpI4

01029943   .  8985 68FFFFFF MOV DWORD PTR SS:[EBP-98],EAX         

;  浮点数转转换整数，即为 3577197，此即为注册码的前 7 位

01029949   .  C745 FC 0A000>MOV DWORD PTR SS:[EBP-4],0A

01029950   .  8B8D 68FFFFFF MOV ECX,DWORD PTR SS:[EBP-98]

01029956   .  51            PUSH ECX

01029957   .  FF15 28104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrI4>] 

;  MSVBVM60.__vbaStrI4

0102995D   .  8BD0          MOV EDX,EAX                             

;  转换为十进制，eax=3577197

0102995F   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]

01029962   .  FF15 E0124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>;  MSVBVM60.__vbaStrMove

01029968   >  C745 FC 0B000>MOV DWORD PTR SS:[EBP-4],0B

0102996F   .  8B55 C0       MOV EDX,DWORD PTR SS:[EBP-40]

01029972   .  52            PUSH EDX

01029973   .  FF15 50104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaLenBstr>;  MSVBVM60.__vbaLenBstr

01029979   .  83F8 07       CMP EAX,7                               

;  产生的数是否为 7 为

0102997C   .  7D 23         JGE SHORT 万能转换.010299A1

0102997E   .  C745 FC 0C000>MOV DWORD PTR SS:[EBP-4],0C

01029985   .  68 981A4700   PUSH 万能转换.00471A98

0102998A   .  8B45 C0       MOV EAX,DWORD PTR SS:[EBP-40]

0102998D   .  50            PUSH EAX

0102998E   .  FF15 84104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat>>;  MSVBVM60.__vbaStrCat

01029994   .  8BD0          MOV EDX,EAX

01029996   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]

01029999   .  FF15 E0124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>;  MSVBVM60.__vbaStrMove

0102999F   .^ EB C7         JMP SHORT 万能转换.01029968

010299A1   >  C745 FC 0E000>MOV DWORD PTR SS:[EBP-4],0E

010299A8   .  66:C785 74FFF>MOV WORD PTR SS:[EBP-8C],0

010299B1   .  C745 FC 0F000>MOV DWORD PTR SS:[EBP-4],0F

010299B8   .  66:C785 8CFEF>MOV WORD PTR SS:[EBP-174],1

010299C1   .  66:C785 90FEF>MOV WORD PTR SS:[EBP-170],0FFFF

010299CA   .  8B4D C0       MOV ECX,DWORD PTR SS:[EBP-40]

010299CD   .  51            PUSH ECX

010299CE   .  FF15 50104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaLenBstr>;  MSVBVM60.__vbaLenBstr

010299D4   .  8BC8          MOV ECX,EAX

010299D6   .  FF15 70114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaI2I4>]   

;  MSVBVM60.__vbaI2I4

010299DC   .  66:8945 C8    MOV WORD PTR SS:[EBP-38],AX

010299E0   .  EB 15         JMP SHORT 万能转换.010299F7

010299E2   >  66:8B55 C8    MOV DX,WORD PTR SS:[EBP-38]

010299E6   .  66:0395 90FEF>ADD DX,WORD PTR SS:[EBP-170]           

;  加上－1

010299ED   .  0F80 3D160000 JO 万能转换.0102B030

010299F3   .  66:8955 C8    MOV WORD PTR SS:[EBP-38],DX

010299F7   >  66:8B45 C8    MOV AX,WORD PTR SS:[EBP-38]

010299FB   .  66:3B85 8CFEF>CMP AX,WORD PTR SS:[EBP-174]

01029A02   .  0F8C 9F000000 JL 万能转换.01029AA7

01029A08   .  C745 FC 10000>MOV DWORD PTR SS:[EBP-4],10

01029A0F   .  C785 34FFFFFF>MOV DWORD PTR SS:[EBP-CC],1

01029A19   .  C785 2CFFFFFF>MOV DWORD PTR SS:[EBP-D4],2

01029A23   .  8D8D 2CFFFFFF LEA ECX,DWORD PTR SS:[EBP-D4]         

;  ecx 处为 2

01029A29   .  51            PUSH ECX

01029A2A   .  0FBF55 C8     MOVSX EDX,WORD PTR SS:[EBP-38]         

;  长度入 edx，长度依次由 7 减为 0

01029A2E   .  52            PUSH EDX

01029A2F   .  8B45 C0       MOV EAX,DWORD PTR SS:[EBP-40]         

;  上面产生的 7 位数，此为 3577197

01029A32   .  50            PUSH EAX

01029A33   .  FF15 28114000 CALL DWORD PTR DS:[<&MSVBVM60.#631>]   

;  MSVBVM60.rtcMidCharBstr

01029A39   .  8BD0          MOV EDX,EAX                             

;  从第 edx 位开始，共取 1 位数

01029A3B   .  8D8D 70FFFFFF LEA ECX,DWORD PTR SS:[EBP-90]

01029A41   .  FF15 E0124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>;  取得的数保存好

01029A47   .  8D8D 2CFFFFFF LEA ECX,DWORD PTR SS:[EBP-D4]

01029A4D   .  FF15 3C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeVar>;  MSVBVM60.__vbaFreeVar

01029A53   .  C745 FC 11000>MOV DWORD PTR SS:[EBP-4],11

01029A5A   .  8B8D 70FFFFFF MOV ECX,DWORD PTR SS:[EBP-90]

01029A60   .  51            PUSH ECX

01029A61   .  FF15 E0114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaI2Str>] 

;  取得的数转换为十进制

01029A67   .  66:0FAF45 C8  IMUL AX,WORD PTR SS:[EBP-38]           

;  从 3577197 串中最后一位开始，各位与位置数相乘

01029A6C   .  0F80 BE150000 JO 万能转换.0102B030

01029A72   .  66:8985 50FFF>MOV WORD PTR SS:[EBP-B0],AX

01029A79   .  C745 FC 12000>MOV DWORD PTR SS:[EBP-4],12

01029A80   .  66:8B95 74FFF>MOV DX,WORD PTR SS:[EBP-8C]

01029A87   .  66:0395 50FFF>ADD DX,WORD PTR SS:[EBP-B0]             

;  从 3577197 串中最后一位开始，各位与位置数相乘，然后依次相加

01029A8E   .  0F80 9C150000 JO 万能转换.0102B030

01029A94   .  66:8995 74FFF>MOV WORD PTR SS:[EBP-8C],DX
```



```
01029A9B   .  C745 FC 13000>MOV DWORD PTR SS:\[EBP\-4\],13  

01029AA2   .^ E9 3BFFFFFF   JMP 万能转换.010299E2  

01029AA7   >  C745 FC 14000>MOV DWORD PTR SS:\[EBP\-4\],14  

01029AAE   .  66:8B85 74FFF>MOV AX,WORD PTR SS:\[EBP\-8C\]  

;  上面累加的值入 ax  

01029AB5   .  66:99         CWD  

01029AB7   .  66:B9 0700    MOV CX,7  

;  7 入 cx  

01029ABB   .  66:F7F9       IDIV CX  

01029ABE   .  66:8955 C8    MOV WORD PTR SS:\[EBP\-38\],DX  

;  累加的值除以 7，取模，此即为注册码为第 8 位  

01029AC2   .  C745 FC 15000>MOV DWORD PTR SS:\[EBP\-4\],15  

01029AC9   .  66:8B55 C8    MOV DX,WORD PTR SS:\[EBP\-38\]  

01029ACD   .  52            PUSH EDX  

01029ACE   .  FF15 14104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrI2>\] 

;  MSVBVM60.\_\_vbaStrI2  

01029AD4   .  8BD0          MOV EDX,EAX  

01029AD6   .  8D4D CC       LEA ECX,DWORD PTR SS:\[EBP\-34\]  

01029AD9   .  FF15 E0124000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaStrMove>;  MSVBVM60.\_\_vbaStrMove  

01029ADF   .  C745 FC 16000>MOV DWORD PTR SS:\[EBP\-4\],16  

01029AE6   .  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]  

01029AE9   .  8B08          MOV ECX,DWORD PTR DS:\[EAX\]  

01029AEB   .  8B55 08       MOV EDX,DWORD PTR SS:\[EBP+8\]  

01029AEE   .  52            PUSH EDX  

01029AEF   .  FF91 0C030000 CALL DWORD PTR DS:\[ECX+30C\]  

01029AF5   .  50            PUSH EAX  

01029AF6   .  8D85 40FFFFFF LEA EAX,DWORD PTR SS:\[EBP\-C0\]  

01029AFC   .  50            PUSH EAX  

01029AFD   .  FF15 E8104000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaObjSet>>;  MSVBVM60.\_\_vbaObjSet  

01029B03   .  8985 A4FEFFFF MOV DWORD PTR SS:\[EBP\-15C\],EAX  

01029B09   .  8B4D C0       MOV ECX,DWORD PTR SS:\[EBP\-40\]  

01029B0C   .  51            PUSH ECX  

01029B0D   .  8B55 CC       MOV EDX,DWORD PTR SS:\[EBP\-34\]  

01029B10   .  52            PUSH EDX  

01029B11   .  FF15 84104000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaStrCat>>;  MSVBVM60.\_\_vbaStrCat  

01029B17   .  8BD0          MOV EDX,EAX  

;  在数 3577197 后追加一位，变为 35771972  

01029B19   .  8D8D 48FFFFFF LEA ECX,DWORD PTR SS:\[EBP\-B8\]  

01029B1F   .  FF15 E0124000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaStrMove>;  MSVBVM60.\_\_vbaStrMove  

01029B25   .  50            PUSH EAX  

01029B26   .  8B85 A4FEFFFF MOV EAX,DWORD PTR SS:\[EBP\-15C\]  

01029B2C   .  8B08          MOV ECX,DWORD PTR DS:\[EAX\]  

01029B2E   .  8B95 A4FEFFFF MOV EDX,DWORD PTR SS:\[EBP\-15C\]  

01029B34   .  52            PUSH EDX  

01029B35   .  FF91 A4000000 CALL DWORD PTR DS:\[ECX+A4\]  

01029B3B   .  DBE2          FCLEX  

01029B3D   .  8985 A0FEFFFF MOV DWORD PTR SS:\[EBP\-160\],EAX  

01029B43   .  83BD A0FEFFFF>CMP DWORD PTR SS:\[EBP\-160\],0  

01029B4A   .  7D 26         JGE SHORT 万能转换.01029B72  

01029B4C   .  68 A4000000   PUSH 0A4  

01029B51   .  68 7C274700   PUSH 万能转换.0047277C  

01029B56   .  8B85 A4FEFFFF MOV EAX,DWORD PTR SS:\[EBP\-15C\]  

01029B5C   .  50            PUSH EAX  

01029B5D   .  8B8D A0FEFFFF MOV ECX,DWORD PTR SS:\[EBP\-160\]  

01029B63   .  51            PUSH ECX  

01029B64   .  FF15 A0104000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaHresult>;  MSVBVM60.\_\_vbaHresultCheckObj  

01029B6A   .  8985 58FEFFFF MOV DWORD PTR SS:\[EBP\-1A8\],EAX  

01029B70   .  EB 0A         JMP SHORT 万能转换.01029B7C  

01029B72   >  C785 58FEFFFF>MOV DWORD PTR SS:\[EBP\-1A8\],0  

01029B7C   >  8D8D 48FFFFFF LEA ECX,DWORD PTR SS:\[EBP\-B8\]  

01029B82   .  FF15 38134000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaFreeStr>;  MSVBVM60.\_\_vbaFreeStr  

01029B88   .  8D8D 40FFFFFF LEA ECX,DWORD PTR SS:\[EBP\-C0\]  

01029B8E   .  FF15 34134000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaFreeObj>;  MSVBVM60.\_\_vbaFreeObj  

01029B94   .  C745 FC 17000>MOV DWORD PTR SS:\[EBP\-4\],17  

01029B9B   .  8B55 08       MOV EDX,DWORD PTR SS:\[EBP+8\]  

01029B9E   .  8B02          MOV EAX,DWORD PTR DS:\[EDX\]  

01029BA0   .  8B4D 08       MOV ECX,DWORD PTR SS:\[EBP+8\]  

01029BA3   .  51            PUSH ECX  

01029BA4   .  FF90 14030000 CALL DWORD PTR DS:\[EAX+314\]  

01029BAA   .  50            PUSH EAX  

01029BAB   .  8D95 3CFFFFFF LEA EDX,DWORD PTR SS:\[EBP\-C4\]  

01029BB1   .  52            PUSH EDX  

01029BB2   .  FF15 E8104000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaObjSet>>;  MSVBVM60.\_\_vbaObjSet  

01029BB8   .  8985 A4FEFFFF MOV DWORD PTR SS:\[EBP\-15C\],EAX  

01029BBE   .  8D85 44FFFFFF LEA EAX,DWORD PTR SS:\[EBP\-BC\]  

01029BC4   .  50            PUSH EAX  

01029BC5   .  8B8D A4FEFFFF MOV ECX,DWORD PTR SS:\[EBP\-15C\]  

01029BCB   .  8B11          MOV EDX,DWORD PTR DS:\[ECX\]  

01029BCD   .  8B85 A4FEFFFF MOV EAX,DWORD PTR SS:\[EBP\-15C\]  

01029BD3   .  50            PUSH EAX  

01029BD4   .  FF92 A0000000 CALL DWORD PTR DS:\[EDX+A0\]  

01029BDA   .  DBE2          FCLEX  

01029BDC   .  8985 A0FEFFFF MOV DWORD PTR SS:\[EBP\-160\],EAX  

01029BE2   .  83BD A0FEFFFF>CMP DWORD PTR SS:\[EBP\-160\],0  

01029BE9   .  7D 26         JGE SHORT 万能转换.01029C11  

01029BEB   .  68 A0000000   PUSH 0A0  

01029BF0   .  68 7C274700   PUSH 万能转换.0047277C  

01029BF5   .  8B8D A4FEFFFF MOV ECX,DWORD PTR SS:\[EBP\-15C\]  

01029BFB   .  51            PUSH ECX  

01029BFC   .  8B95 A0FEFFFF MOV EDX,DWORD PTR SS:\[EBP\-160\]  

01029C02   .  52            PUSH EDX  

01029C03   .  FF15 A0104000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaHresult>;  MSVBVM60.\_\_vbaHresultCheckObj  

01029C09   .  8985 54FEFFFF MOV DWORD PTR SS:\[EBP\-1AC\],EAX  

01029C0F   .  EB 0A         JMP SHORT 万能转换.01029C1B  

01029C11   >  C785 54FEFFFF>MOV DWORD PTR SS:\[EBP\-1AC\],0  

01029C1B   >  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]  

01029C1E   .  8B08          MOV ECX,DWORD PTR DS:\[EAX\]  

01029C20   .  8B55 08       MOV EDX,DWORD PTR SS:\[EBP+8\]  

01029C23   .  52            PUSH EDX  

01029C24   .  FF91 0C030000 CALL DWORD PTR DS:\[ECX+30C\]  

01029C2A   .  50            PUSH EAX  

01029C2B   .  8D85 40FFFFFF LEA EAX,DWORD PTR SS:\[EBP\-C0\]  

01029C31   .  50            PUSH EAX  

01029C32   .  FF15 E8104000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaObjSet>>;  MSVBVM60.\_\_vbaObjSet  

01029C38   .  8985 9CFEFFFF MOV DWORD PTR SS:\[EBP\-164\],EAX  

01029C3E   .  8D8D 48FFFFFF LEA ECX,DWORD PTR SS:\[EBP\-B8\]  

01029C44   .  51            PUSH ECX  

01029C45   .  8B95 9CFEFFFF MOV EDX,DWORD PTR SS:\[EBP\-164\]  

01029C4B   .  8B02          MOV EAX,DWORD PTR DS:\[EDX\]  

01029C4D   .  8B8D 9CFEFFFF MOV ECX,DWORD PTR SS:\[EBP\-164\]  

01029C53   .  51            PUSH ECX  

01029C54   .  FF90 A0000000 CALL DWORD PTR DS:\[EAX+A0\]  

01029C5A   .  DBE2          FCLEX  

01029C5C   .  8985 98FEFFFF MOV DWORD PTR SS:\[EBP\-168\],EAX  

01029C62   .  83BD 98FEFFFF>CMP DWORD PTR SS:\[EBP\-168\],0  

01029C69   .  7D 26         JGE SHORT 万能转换.01029C91  

01029C6B   .  68 A0000000   PUSH 0A0  

01029C70   .  68 7C274700   PUSH 万能转换.0047277C  

01029C75   .  8B95 9CFEFFFF MOV EDX,DWORD PTR SS:\[EBP\-164\]  

01029C7B   .  52            PUSH EDX  

01029C7C   .  8B85 98FEFFFF MOV EAX,DWORD PTR SS:\[EBP\-168\]  

01029C82   .  50            PUSH EAX  

01029C83   .  FF15 A0104000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaHresult>;  MSVBVM60.\_\_vbaHresultCheckObj  

01029C89   .  8985 50FEFFFF MOV DWORD PTR SS:\[EBP\-1B0\],EAX  

01029C8F   .  EB 0A         JMP SHORT 万能转换.01029C9B  

01029C91   >  C785 50FEFFFF>MOV DWORD PTR SS:\[EBP\-1B0\],0  

01029C9B   >  8B8D 44FFFFFF MOV ECX,DWORD PTR SS:\[EBP\-BC\]  

;  输入的假码 654321  

01029CA1   .  51            PUSH ECX  

01029CA2   .  8B95 48FFFFFF MOV EDX,DWORD PTR SS:\[EBP\-B8\]  

;  真码  

01029CA8   .  52            PUSH EDX  

01029CA9   .  FF15 54114000 CALL DWORD PTR 
DS:\[<&MSVBVM60.\_\_vbaStrCmp>>;  MSVBVM60.\_\_vbaStrCmp  

01029CAF   .  F7D8          NEG EAX  

;  比较是否相等  

01029CB1   .  1BC0          SBB EAX,EAX  

01029CB3   .  40            INC EAX  
```



# 代码

## 汇编指令

`01029CB4` `.` `F7D8` `NEG EAX`

`01029CB6` `.` `66:8985 94FEF` `>MOV WORD PTR SS:[EBP-16C],AX`

`01029CBD` `.` `8D85 48FFFFFF` `LEA EAX,DWORD PTR SS:[EBP-B8]`

`01029CC3` `.` `50` `PUSH EAX`

`01029CC4` `.` `8D8D 44FFFFFF` `LEA ECX,DWORD PTR SS:[EBP-BC]`

`01029CCA` `.` `51` `PUSH ECX`

`01029CCB` `.` `6A 02` `PUSH 2`

`01029CCD` `.` `FF15 68124000` `CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStr>; MSVBVM60.__vbaFreeStrList`

`01029CD3` `.` `83C4 0C` `ADD ESP,0C`

`01029CD6` `.` `8D95 3CFFFFFF` `LEA EDX,DWORD PTR SS:[EBP-C4]`

`01029CDC` `.` `52` `PUSH EDX`

`01029CDD` `.` `8D85 40FFFFFF` `LEA EAX,DWORD PTR SS:[EBP-C0]`

`01029CE3` `.` `50` `PUSH EAX`

`01029CE4` `.` `6A 02` `PUSH 2`

`01029CE6` `.` `FF15 64104000` `CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeObj>; MSVBVM60.__vbaFreeObjList`

`01029CEC` `.` `83C4 0C` `ADD ESP,0C`

`01029CEF` `.` `0FBF8D 94FEFF` `>MOVSX ECX,WORD PTR SS:[EBP-16C]`

`01029CF6` `.` `85C9` `TEST ECX,ECX`

`01029CF8` `.` `0F84 140E0000` `JE 万能转换.0102AB12`

`01029CFE` `.` `C745 FC 18000` `>MOV DWORD PTR SS:[EBP-4],18`

`01029D05` `.` `C785 F0FEFFFF` `>MOV DWORD PTR SS:[EBP-110],0`

`01029D0F` `.` `C785 E8FEFFFF` `>MOV DWORD PTR SS:[EBP-118],2`

`01029D19` `.` `B8 10000000` `MOV EAX,10`

`01029D1E` `.` `E8 8DC83DFF` `CALL <JMP.&MSVBVM60.__vbaChkstk>`

`01029D23` `.` `8BD4` `MOV EDX,ESP`

`01029D25` `.` `8B85 E8FEFFFF` `MOV EAX,DWORD PTR SS:[EBP-118]`

`01029D2B` `.` `8902` `MOV DWORD PTR DS:[EDX],EAX`

`01029D2D` `.` `8B8D ECFEFFFF` `MOV ECX,DWORD PTR SS:[EBP-114]`

`01029D33` `.` `894A 04` `MOV DWORD PTR DS:[EDX+4],ECX`

`01029D36` `.` `8B85 F0FEFFFF` `MOV EAX,DWORD PTR SS:[EBP-110]`

`01029D3C` `.` `8942 08` `MOV DWORD PTR DS:[EDX+8],EAX`

`01029D3F` `.` `8B8D F4FEFFFF` `MOV ECX,DWORD PTR SS:[EBP-10C]`

`01029D45` `.` `894A 0C` `MOV DWORD PTR DS:[EDX+C],ECX`

`01029D48` `.` `68 FC564700` `PUSH 万能转换.004756FC`

; 这里保存注册信息

`01029D4D` `.` `68 801A4700` `PUSH 万能转换.00471A80`

; UNICODE "names"

`01029D52` `.` `68 701A4700` `PUSH 万能转换.00471A70`

; UNICODE "set"

`01029D57` `.` `FF15 8C124000` `CALL DWORD PTR DS:[<&MSVBVM60.#689>]`

; MSVBVM60.rtcGetSetting

`01029D5D` `.` `8BD0` `MOV EDX,EAX`

`01029D5F` `.` `8D8D 6CFFFFFF` `LEA ECX,DWORD PTR SS:[EBP-94]`

`01029D65` `.` `FF15 E0124000` `CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>; MSVBVM60.__vbaStrMove`

`01029D6B` `.` `C745 FC 19000` `>MOV DWORD PTR SS:[EBP-4],19`

`01029D72` `.` `8B55 08` `MOV EDX,DWORD PTR SS:[EBP+8]`

`01029D75` `.` `8B02` `MOV EAX,DWORD PTR DS:[EDX]`

---

## 总结

注册码共为 `8` 位，与用户名无关。

前面的 `7` 位这样产生：取机器码右边的 `5` 位数字（设为 `A`），求 `A` 的平方根，得 `B` 值，`B` 与 `12345` 相乘取整数（要四舍五入），所得即为注册码的前面 `7` 位(如不足 `7` 位则在前面补 `0`)。

最后一位这样产生：前面产生的 `7` 位数各位与位置数相乘（即为 `sum=1*s1+2*s2+3*s3+4*s4+5*s5+6*s6+7*s7`）,所得值取 `7` 的模数，即为最后一位。

## C++ 注册机源码

```c++
double sqr;
int regcode;
int tmp,i=0;
int sum=0;
char num[20]={0,0,0,0,0,0,0,0,0,0,0,0,0,0,0};

switch(message)
{
    case WM_COMMAND:
        switch(LOWORD(wParam))
        {
            case IDC_CREATE: 
                regcode=GetDlgItemInt(dhWnd,IDC_EDIT1,FALSE,FALSE);
                sqr=sqrt(regcode);
                sqr=sqr*12345;
                tmp=(int)sqr;
                if((sqr-tmp)>0.5)
                tmp=tmp+1;

                if(tmp<1000000)
                {
                    num[0]=48+0;//数字转换为字符,如为 6 位数，在前面补 0
                    num[1]=48+(int)tmp/100000;
                    tmp=tmp-((int)tmp/100000)*100000;
                    num[2]=48+(int)tmp/10000;
                    tmp=tmp-((int)tmp/10000)*10000;
                    num[3]=48+(int)tmp/1000;
                    tmp=tmp-((int)tmp/1000)*1000;
                    num[4]=48+(int)tmp/100;
                    tmp=tmp-((int)tmp/100)*100;
                    num[5]=48+(int)tmp/10;
                    tmp=tmp-((int)tmp/10)*10;
                    num[6]=48+tmp;
                }
                else
                {
                    num[0]=48+(int)tmp/1000000;
                    tmp=tmp-((int)tmp/1000000)*1000000;
                    num[1]=48+(int)tmp/100000;
                    tmp=tmp-((int)tmp/100000)*100000;
                    num[2]=48+(int)tmp/10000;
                    tmp=tmp-((int)tmp/10000)*10000;
                    num[3]=48+(int)tmp/1000;
                    tmp=tmp-((int)tmp/1000)*1000;
                    num[4]=48+(int)tmp/100;
                    tmp=tmp-((int)tmp/100)*100;
                    num[5]=48+(int)tmp/10;
                    tmp=tmp-((int)tmp/10)*10;
                    num[6]=48+tmp;
                }

                for(i=1;i<=7;i++)
                {
                    sum=sum+(num[i-1]-48)*i;
                }
                tmp=mod(sum,7);
                num[7]=48+tmp;
                
                //sprintf(temp,"%d",tmp);
                SetDlgItemText(dhWnd,IDC_EDIT3,num);
                //SetDlgItemText(dhWnd,IDC_EDIT3,temp);
                return 1;
        }
}
```

## 注册信息

注册名称：`lordor[BCG]`

机器码：`TNXS-OVBR-83966`

注册码：`35771972`

注册信息保存在：`[HKEY_CURRENT_USER\Software\VB and VBA Program Settings\MyApp\set]`

- `names` = `35771972`
- `nam` = `lordor[BCG]`

`cracked by lordor[BCG]`

`03.5.22`

`4dd466d0e1e0cc123afbe504da2462ca`
