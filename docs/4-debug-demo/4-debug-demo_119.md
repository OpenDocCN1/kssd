

# 菜鸟注册机编写基础教程（五）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-234.htm`](https://www.pediy.com/kssd/pediy08/pediy8-234.htm)

五、实战篇  
  
    学习了前面的一些基础知识，这篇开始进入实战。挑选一个好的例子很难，我用 VB 写个 Crackeme（见附件），这样可以自己控制让算法尽量多涉及几个知识面，因为短短的几篇文章想让一个没有编程基础的人马上学会写注册机确实很难，最主要的还是靠自己多写、多查、多问。这个 Crackeme  有涉及逻辑算法，算术算法，MD5 加密算法，字符串查找替换等方面，我们先进入算法分析。  
很多人说 VB 的程序很难跟踪，其实主要是 VB 代码被反汇编后“垃圾代码”太多，再个就是 VB 中数据的存储格式和寻址方式比较特殊，动态调试时一般用 DD EAX+8 之类的就可以看到存储的数值信息。中断可以借用 VB 的资源编辑软件从字符串信息突破，不过这个 Crackeme 的字符串信息已经被我加密了，可以在函数\_\_vbaVarTstEq 上下断，另外，对 VB 的程序可以下万能断点 非常有效。还有就是多了解 VB 的内部函数，有时候看看它的函数就知道进行什么操作。下面是算法分析：  
004037FE   .  FF15 4C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresultChe>;  MSVBVM60.\_\_vbaHresultCheckObj  
00403804   >  8B45 98       MOV EAX,DWORD PTR SS:\[EBP-68\]                 ;  取邮箱名  
00403807   .  50            PUSH EAX  
00403808   .  FF15 1C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaLenBstr>\]  ;  取长度  
0040380E   .  8BC8          MOV ECX,EAX                                   ;  放入 ECX  
00403810   .  FF15 CC104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaI2I4>\]     ;  MSVBVM60.\_\_vbaI2I4  
00403816   .  8D4D 98       LEA ECX,DWORD PTR SS:\[EBP-68\]  
00403819   .  8985 98FEFFFF MOV DWORD PTR SS:\[EBP-168\],EAX                ;  邮箱长度  
0040381F   .  BB 01000000   MOV EBX,1                                     ;  EBX 赋值 1  
00403824   .  FF15 98114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeStr>\]  ;  MSVBVM60.\_\_vbaFreeStr  
0040382A   .  8D4D 88       LEA ECX,DWORD PTR SS:\[EBP-78\]  
0040382D   .  FF15 9C114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeObj>\]  ;  MSVBVM60.\_\_vbaFreeObj  
00403833   >  66:3B9D 98FEF>CMP BX,WORD PTR SS:\[EBP-168\]                  ;  循环开始，对比邮箱位数是否取完  
0040383A   .  0F8F 6D010000 JG crackeme.004039AD                          ;  如果取完就跳走，否则继续循环  
00403840   .  8B0F          MOV ECX,DWORD PTR DS:\[EDI\]  
00403842   .  57            PUSH EDI  
00403843   .  FF91 08030000 CALL DWORD PTR DS:\[ECX+308\]  
00403849   .  8D55 88       LEA EDX,DWORD PTR SS:\[EBP-78\]  
0040384C   .  50            PUSH EAX  
0040384D   .  52            PUSH EDX  
0040384E   .  FF15 7C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaObjSet>\]   ;  MSVBVM60.\_\_vbaObjSet  
00403854   .  8B08          MOV ECX,DWORD PTR DS:\[EAX\]  
00403856   .  8D55 98       LEA EDX,DWORD PTR SS:\[EBP-68\]  
00403859   .  52            PUSH EDX  
0040385A   .  50            PUSH EAX  
0040385B   .  8985 B0FEFFFF MOV DWORD PTR SS:\[EBP-150\],EAX  
00403861   .  FF91 A0000000 CALL DWORD PTR DS:\[ECX+A0\]  
00403867   .  3BC6          CMP EAX,ESI  
00403869   .  DBE2          FCLEX  
0040386B   .  7D 18         JGE SHORT crackeme.00403885  
0040386D   .  8B8D B0FEFFFF MOV ECX,DWORD PTR SS:\[EBP-150\]  
00403873   .  68 A0000000   PUSH 0A0  
00403878   .  68 B8294000   PUSH crackeme.004029B8  
0040387D   .  51            PUSH ECX  
0040387E   .  50            PUSH EAX  
0040387F   .  FF15 4C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresultChe>;  MSVBVM60.\_\_vbaHresultCheckObj  
00403885   >  8B45 98       MOV EAX,DWORD PTR SS:\[EBP-68\]                 ;  邮箱名  
00403888   .  8D95 64FFFFFF LEA EDX,DWORD PTR SS:\[EBP-9C\]  
0040388E   .  8985 7CFFFFFF MOV DWORD PTR SS:\[EBP-84\],EAX                 ;  邮箱名  
00403894   .  52            PUSH EDX  
00403895   .  0FBFC3        MOVSX EAX,BX                                  ;  位数递增  
00403898   .  8D8D 74FFFFFF LEA ECX,DWORD PTR SS:\[EBP-8C\]  
0040389E   .  50            PUSH EAX  
0040389F   .  8D95 54FFFFFF LEA EDX,DWORD PTR SS:\[EBP-AC\]  
004038A5   .  51            PUSH ECX  
004038A6   .  52            PUSH EDX  
004038A7   .  C785 6CFFFFFF>MOV DWORD PTR SS:\[EBP-94\],1  
004038B1   .  C785 64FFFFFF>MOV DWORD PTR SS:\[EBP-9C\],2  
004038BB   .  8975 98       MOV DWORD PTR SS:\[EBP-68\],ESI  
004038BE   .  C785 74FFFFFF>MOV DWORD PTR SS:\[EBP-8C\],8  
004038C8   .  FF15 B0104000 CALL DWORD PTR DS:\[<&MSVBVM60.#632>\]          ;  MSVBVM60.rtcMidCharVar  
004038CE   .  8D85 54FFFFFF LEA EAX,DWORD PTR SS:\[EBP-AC\]  
004038D4   .  8D4D 94       LEA ECX,DWORD PTR SS:\[EBP-6C\]  
004038D7   .  50            PUSH EAX  
004038D8   .  51            PUSH ECX  
004038D9   .  FF15 10114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrVarVal>>;  MSVBVM60.\_\_vbaStrVarVal  
004038DF   .  50            PUSH EAX  
004038E0   .  FF15 34104000 CALL DWORD PTR DS:\[<&MSVBVM60.#516>\]          ;  MSVBVM60.rtcAnsiValueBstr  
004038E6   .  0FBFD0        MOVSX EDX,AX                                  ;  逐个取邮箱名的 ASCII  
004038E9   .  8B07          MOV EAX,DWORD PTR DS:\[EDI\]  
004038EB   .  8D8D B4FEFFFF LEA ECX,DWORD PTR SS:\[EBP-14C\]  
004038F1   .  8995 B8FEFFFF MOV DWORD PTR SS:\[EBP-148\],EDX                ;  邮箱名的 ASCII  
004038F7   .  51            PUSH ECX  
004038F8   .  8D95 C0FEFFFF LEA EDX,DWORD PTR SS:\[EBP-140\]  
004038FE   .  8D8D B8FEFFFF LEA ECX,DWORD PTR SS:\[EBP-148\]  
00403904   .  52            PUSH EDX  
00403905   .  51            PUSH ECX  
00403906   .  57            PUSH EDI  
00403907      C785 C0FEFFFF>MOV DWORD PTR SS:\[EBP-140\],2                  ;  将 2 赋值给段寄存器 SS:\[EBP-140\]  
00403911   .  FF90 04070000 CALL DWORD PTR DS:\[EAX+704\]                   ;  自定义异或函数  
00403917   .  3BC6          CMP EAX,ESI  
00403919   .  7D 12         JGE SHORT crackeme.0040392D  
0040391B   .  68 04070000   PUSH 704  
00403920   .  68 C4264000   PUSH crackeme.004026C4  
00403925   .  57            PUSH EDI  
00403926   .  50            PUSH EAX  
00403927   .  FF15 4C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresultChe>;  MSVBVM60.\_\_vbaHresultCheckObj  
0040392D   >  8B95 B4FEFFFF MOV EDX,DWORD PTR SS:\[EBP-14C\]                ;  取异或 2 后的值  
00403933   .  8D45 C0       LEA EAX,DWORD PTR SS:\[EBP-40\]  
00403936   .  8995 0CFFFFFF MOV DWORD PTR SS:\[EBP-F4\],EDX                 ;  异或 2 后的值  
0040393C   .  8D8D 04FFFFFF LEA ECX,DWORD PTR SS:\[EBP-FC\]  
00403942   .  50            PUSH EAX  
00403943   .  8D95 44FFFFFF LEA EDX,DWORD PTR SS:\[EBP-BC\]  
00403949   .  51            PUSH ECX                                      ;  异或 2 后的结果  
0040394A   .  52            PUSH EDX  
0040394B   .  C785 04FFFFFF>MOV DWORD PTR SS:\[EBP-FC\],3  
00403955   .  FF15 58114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarAdd>\]   ;  MSVBVM60.\_\_vbaVarAdd  
0040395B   .  8BD0          MOV EDX,EAX                                   ;  累加  
0040395D   .  8D4D C0       LEA ECX,DWORD PTR SS:\[EBP-40\]  
00403960   .  FF15 10104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarMove>\]  ;  MSVBVM60.\_\_vbaVarMove  
00403966   .  8D4D 94       LEA ECX,DWORD PTR SS:\[EBP-6C\]  
00403969   .  FF15 98114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeStr>\]  ;  MSVBVM60.\_\_vbaFreeStr  
0040396F   .  8D4D 88       LEA ECX,DWORD PTR SS:\[EBP-78\]  
00403972   .  FF15 9C114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeObj>\]  ;  MSVBVM60.\_\_vbaFreeObj  
00403978   .  8D85 54FFFFFF LEA EAX,DWORD PTR SS:\[EBP-AC\]  
0040397E   .  8D8D 64FFFFFF LEA ECX,DWORD PTR SS:\[EBP-9C\]  
00403984   .  50            PUSH EAX  
00403985   .  8D95 74FFFFFF LEA EDX,DWORD PTR SS:\[EBP-8C\]  
0040398B   .  51            PUSH ECX  
0040398C   .  52            PUSH EDX  
0040398D   .  6A 03         PUSH 3  
0040398F   .  FF15 28104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeVarLis>;  MSVBVM60.\_\_vbaFreeVarList  
00403995   .  B8 01000000   MOV EAX,1  
0040399A   .  83C4 10       ADD ESP,10  
0040399D   .  66:03C3       ADD AX,BX  
004039A0   .  0F80 7E080000 JO crackeme.00404224  
004039A6   .  8BD8          MOV EBX,EAX  
004039A8   .^ E9 86FEFFFF   JMP crackeme.00403833                         ;  继续循环  
004039AD   >  8B07          MOV EAX,DWORD PTR DS:\[EDI\]  
004039AF   .  57            PUSH EDI  
004039B0   .  FF90 10030000 CALL DWORD PTR DS:\[EAX+310\]  
004039B6   .  8D4D 88       LEA ECX,DWORD PTR SS:\[EBP-78\]  
004039B9   .  50            PUSH EAX  
004039BA   .  51            PUSH ECX  
004039BB   .  FF15 7C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaObjSet>\]   ;  MSVBVM60.\_\_vbaObjSet  
004039C1   .  8BD8          MOV EBX,EAX  
004039C3   .  8D45 98       LEA EAX,DWORD PTR SS:\[EBP-68\]  
004039C6   .  50            PUSH EAX  
004039C7   .  53            PUSH EBX  
004039C8   .  8B13          MOV EDX,DWORD PTR DS:\[EBX\]  
004039CA   .  FF92 A0000000 CALL DWORD PTR DS:\[EDX+A0\]  
004039D0   .  3BC6          CMP EAX,ESI  
004039D2   .  DBE2          FCLEX  
004039D4   .  7D 12         JGE SHORT crackeme.004039E8  
004039D6   .  68 A0000000   PUSH 0A0  
004039DB   .  68 B8294000   PUSH crackeme.004029B8  
004039E0   .  53            PUSH EBX  
004039E1   .  50            PUSH EAX  
004039E2   .  FF15 4C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresultChe>;  MSVBVM60.\_\_vbaHresultCheckObj  
004039E8   >  8B4D 98       MOV ECX,DWORD PTR SS:\[EBP-68\]                 ;  取机器码  
004039EB   .  51            PUSH ECX  
004039EC   .  FF15 1C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaLenBstr>\]  ;  MSVBVM60.\_\_vbaLenBstr  
004039F2   .  8BC8          MOV ECX,EAX                                   ;  机器码长度  
004039F4   .  FF15 CC104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaI2I4>\]     ;  MSVBVM60.\_\_vbaI2I4  
004039FA   .  8D4D 98       LEA ECX,DWORD PTR SS:\[EBP-68\]  
004039FD   .  8985 90FEFFFF MOV DWORD PTR SS:\[EBP-170\],EAX                ;  机器码长度  
00403A03   .  BB 01000000   MOV EBX,1  
00403A08   .  FF15 98114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeStr>\]  ;  MSVBVM60.\_\_vbaFreeStr  
00403A0E   .  8D4D 88       LEA ECX,DWORD PTR SS:\[EBP-78\]  
00403A11   .  FF15 9C114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeObj>\]  ;  MSVBVM60.\_\_vbaFreeObj  
00403A17   >  66:3B9D 90FEF>CMP BX,WORD PTR SS:\[EBP-170\]                  ;  循环开始，机器码是否取完  
00403A1E   .  0F8F 33010000 JG crackeme.00403B57                          ;  没有则继续循环  
00403A24   .  8B17          MOV EDX,DWORD PTR DS:\[EDI\]  
00403A26   .  57            PUSH EDI  
00403A27   .  FF92 10030000 CALL DWORD PTR DS:\[EDX+310\]  
00403A2D   .  50            PUSH EAX  
00403A2E   .  8D45 88       LEA EAX,DWORD PTR SS:\[EBP-78\]  
00403A31   .  50            PUSH EAX  
00403A32   .  FF15 7C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaObjSet>\]   ;  MSVBVM60.\_\_vbaObjSet  
00403A38   .  8B08          MOV ECX,DWORD PTR DS:\[EAX\]  
00403A3A   .  8D55 98       LEA EDX,DWORD PTR SS:\[EBP-68\]  
00403A3D   .  52            PUSH EDX  
00403A3E   .  50            PUSH EAX  
00403A3F   .  8985 B0FEFFFF MOV DWORD PTR SS:\[EBP-150\],EAX  
00403A45   .  FF91 A0000000 CALL DWORD PTR DS:\[ECX+A0\]  
00403A4B   .  3BC6          CMP EAX,ESI  
00403A4D   .  DBE2          FCLEX  
00403A4F   .  7D 18         JGE SHORT crackeme.00403A69  
00403A51   .  8B8D B0FEFFFF MOV ECX,DWORD PTR SS:\[EBP-150\]  
00403A57   .  68 A0000000   PUSH 0A0  
00403A5C   .  68 B8294000   PUSH crackeme.004029B8  
00403A61   .  51            PUSH ECX  
00403A62   .  50            PUSH EAX  
00403A63   .  FF15 4C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresultChe>;  MSVBVM60.\_\_vbaHresultCheckObj  
00403A69   >  8B45 98       MOV EAX,DWORD PTR SS:\[EBP-68\]                 ;  机器码  
00403A6C   .  8D95 64FFFFFF LEA EDX,DWORD PTR SS:\[EBP-9C\]  
00403A72   .  8985 7CFFFFFF MOV DWORD PTR SS:\[EBP-84\],EAX                 ;  机器码  
00403A78   .  52            PUSH EDX  
00403A79   .  0FBFC3        MOVSX EAX,BX                                  ;  逐个统计机器码位数  
00403A7C   .  8D8D 74FFFFFF LEA ECX,DWORD PTR SS:\[EBP-8C\]  
00403A82   .  50            PUSH EAX  
00403A83   .  8D95 54FFFFFF LEA EDX,DWORD PTR SS:\[EBP-AC\]  
00403A89   .  51            PUSH ECX  
00403A8A   .  52            PUSH EDX  
00403A8B   .  C785 6CFFFFFF>MOV DWORD PTR SS:\[EBP-94\],1  
00403A95   .  C785 64FFFFFF>MOV DWORD PTR SS:\[EBP-9C\],2  
00403A9F   .  8975 98       MOV DWORD PTR SS:\[EBP-68\],ESI  
00403AA2   .  C785 74FFFFFF>MOV DWORD PTR SS:\[EBP-8C\],8  
00403AAC   .  FF15 B0104000 CALL DWORD PTR DS:\[<&MSVBVM60.#632>\]          ;  MSVBVM60.rtcMidCharVar  
00403AB2   .  8D85 54FFFFFF LEA EAX,DWORD PTR SS:\[EBP-AC\]  
00403AB8   .  8D4D 94       LEA ECX,DWORD PTR SS:\[EBP-6C\]  
00403ABB   .  50            PUSH EAX  
00403ABC   .  51            PUSH ECX  
00403ABD   .  FF15 10114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrVarVal>>;  MSVBVM60.\_\_vbaStrVarVal  
00403AC3   .  50            PUSH EAX  
00403AC4   .  FF15 34104000 CALL DWORD PTR DS:\[<&MSVBVM60.#516>\]          ;  MSVBVM60.rtcAnsiValueBstr  
00403ACA   .  66:8985 0CFFF>MOV WORD PTR SS:\[EBP-F4\],AX                   ;  逐个取机器码的 ASCII 码  
00403AD1   .  8D55 D8       LEA EDX,DWORD PTR SS:\[EBP-28\]  
00403AD4   .  8D85 04FFFFFF LEA EAX,DWORD PTR SS:\[EBP-FC\]  
00403ADA   .  52            PUSH EDX  
00403ADB   .  8D4D C0       LEA ECX,DWORD PTR SS:\[EBP-40\]  
00403ADE   .  50            PUSH EAX                                      ;  机器码的 ASCII 码  
00403ADF   .  8D95 44FFFFFF LEA EDX,DWORD PTR SS:\[EBP-BC\]  
00403AE5   .  51            PUSH ECX                                      ;  刚才邮箱名的计算结果  
00403AE6   .  52            PUSH EDX  
00403AE7   .  C785 04FFFFFF>MOV DWORD PTR SS:\[EBP-FC\],2  
00403AF1   .  FF15 5C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarXor>\]   ;  MSVBVM60.\_\_vbaVarXor  
00403AF7   .  50            PUSH EAX                                      ;  异或的结果  
00403AF8   .  8D85 34FFFFFF LEA EAX,DWORD PTR SS:\[EBP-CC\]  
00403AFE   .  50            PUSH EAX  
00403AFF   .  FF15 58114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarAdd>\]   ;  MSVBVM60.\_\_vbaVarAdd  
00403B05   .  8BD0          MOV EDX,EAX                                   ;  累加  
00403B07   .  8D4D D8       LEA ECX,DWORD PTR SS:\[EBP-28\]  
00403B0A   .  FF15 10104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarMove>\]  ;  MSVBVM60.\_\_vbaVarMove  
00403B10   .  8D4D 94       LEA ECX,DWORD PTR SS:\[EBP-6C\]  
00403B13   .  FF15 98114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeStr>\]  ;  MSVBVM60.\_\_vbaFreeStr  
00403B19   .  8D4D 88       LEA ECX,DWORD PTR SS:\[EBP-78\]  
00403B1C   .  FF15 9C114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeObj>\]  ;  MSVBVM60.\_\_vbaFreeObj  
00403B22   .  8D8D 54FFFFFF LEA ECX,DWORD PTR SS:\[EBP-AC\]  
00403B28   .  8D95 64FFFFFF LEA EDX,DWORD PTR SS:\[EBP-9C\]  
00403B2E   .  51            PUSH ECX  
00403B2F   .  8D85 74FFFFFF LEA EAX,DWORD PTR SS:\[EBP-8C\]  
00403B35   .  52            PUSH EDX  
00403B36   .  50            PUSH EAX  
00403B37   .  6A 03         PUSH 3  
00403B39   .  FF15 28104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeVarLis>;  MSVBVM60.\_\_vbaFreeVarList  
00403B3F   .  B8 01000000   MOV EAX,1  
00403B44   .  83C4 10       ADD ESP,10  
00403B47   .  66:03C3       ADD AX,BX  
00403B4A   .  0F80 D4060000 JO crackeme.00404224  
00403B50   .  8BD8          MOV EBX,EAX  
00403B52   .^ E9 C0FEFFFF   JMP crackeme.00403A17  
00403B57   >  8B0F          MOV ECX,DWORD PTR DS:\[EDI\]  
00403B59   .  57            PUSH EDI  
00403B5A   .  FF91 0C030000 CALL DWORD PTR DS:\[ECX+30C\]  
00403B60   .  8D55 88       LEA EDX,DWORD PTR SS:\[EBP-78\]  
00403B63   .  50            PUSH EAX  
00403B64   .  52            PUSH EDX  
00403B65   .  FF15 7C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaObjSet>\]   ;  MSVBVM60.\_\_vbaObjSet  
00403B6B   .  8BD8          MOV EBX,EAX  
00403B6D   .  8D4D 98       LEA ECX,DWORD PTR SS:\[EBP-68\]  
00403B70   .  51            PUSH ECX  
00403B71   .  53            PUSH EBX  
00403B72   .  8B03          MOV EAX,DWORD PTR DS:\[EBX\]  
00403B74   .  FF90 A0000000 CALL DWORD PTR DS:\[EAX+A0\]  
00403B7A   .  3BC6          CMP EAX,ESI  
00403B7C   .  DBE2          FCLEX  
00403B7E   .  7D 12         JGE SHORT crackeme.00403B92  
00403B80   .  68 A0000000   PUSH 0A0  
00403B85   .  68 B8294000   PUSH crackeme.004029B8  
00403B8A   .  53            PUSH EBX  
00403B8B   .  50            PUSH EAX  
00403B8C   .  FF15 4C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresultChe>;  MSVBVM60.\_\_vbaHresultCheckObj  
00403B92   >  8B45 98       MOV EAX,DWORD PTR SS:\[EBP-68\]                 ;  取出用户名  
00403B95   .  8D95 64FFFFFF LEA EDX,DWORD PTR SS:\[EBP-9C\]  
00403B9B   .  8985 7CFFFFFF MOV DWORD PTR SS:\[EBP-84\],EAX                 ;  用户名  
00403BA1   .  8D85 74FFFFFF LEA EAX,DWORD PTR SS:\[EBP-8C\]  
00403BA7   .  52            PUSH EDX  
00403BA8   .  8D8D 54FFFFFF LEA ECX,DWORD PTR SS:\[EBP-AC\]  
00403BAE   .  BB 08000000   MOV EBX,8  
00403BB3   .  50            PUSH EAX                                      ;  用户名  
00403BB4   .  51            PUSH ECX  
00403BB5   .  C785 6CFFFFFF>MOV DWORD PTR SS:\[EBP-94\],10  
00403BBF   .  C785 64FFFFFF>MOV DWORD PTR SS:\[EBP-9C\],2  
00403BC9   .  8975 98       MOV DWORD PTR SS:\[EBP-68\],ESI  
00403BCC   .  899D 74FFFFFF MOV DWORD PTR SS:\[EBP-8C\],EBX  
00403BD2   .  E8 C92B0000   CALL crackeme.004067A0                        ;  求 MD5 值  
00403BD7   .  8D95 54FFFFFF LEA EDX,DWORD PTR SS:\[EBP-AC\]  
00403BDD   .  8D85 44FFFFFF LEA EAX,DWORD PTR SS:\[EBP-BC\]  
00403BE3   .  52            PUSH EDX                                      ;  16 位的 MD5 结果  
00403BE4   .  50            PUSH EAX  
00403BE5   .  FF15 C0104000 CALL DWORD PTR DS:\[<&MSVBVM60.#528>\]          ;  MSVBVM60.rtcUpperCaseVar  
00403BEB   .  8D8D 44FFFFFF LEA ECX,DWORD PTR SS:\[EBP-BC\]                 ;  转成大写  
00403BF1   .  51            PUSH ECX  
00403BF2   .  FF15 20104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrVarMove>;  MSVBVM60.\_\_vbaStrVarMove  
00403BF8   .  8BD0          MOV EDX,EAX                                   ;  转大写后的结果  
00403BFA   .  8D4D B0       LEA ECX,DWORD PTR SS:\[EBP-50\]  
00403BFD   .  FF15 78114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrMove>\]  ;  MSVBVM60.\_\_vbaStrMove  
00403C03   .  8D4D 88       LEA ECX,DWORD PTR SS:\[EBP-78\]  
00403C06   .  FF15 9C114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeObj>\]  ;  MSVBVM60.\_\_vbaFreeObj  
00403C0C   .  8D95 44FFFFFF LEA EDX,DWORD PTR SS:\[EBP-BC\]  
00403C12   .  8D85 54FFFFFF LEA EAX,DWORD PTR SS:\[EBP-AC\]  
00403C18   .  52            PUSH EDX                                      ;  转大写后的结果  
00403C19   .  8D8D 64FFFFFF LEA ECX,DWORD PTR SS:\[EBP-9C\]  
00403C1F   .  50            PUSH EAX  
00403C20   .  8D95 74FFFFFF LEA EDX,DWORD PTR SS:\[EBP-8C\]  
00403C26   .  51            PUSH ECX  
00403C27   .  52            PUSH EDX  
00403C28   .  6A 04         PUSH 4  
00403C2A   .  FF15 28104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeVarLis>;  MSVBVM60.\_\_vbaFreeVarList  
00403C30   .  B8 02000000   MOV EAX,2                                     ;  EAX 赋值 2  
00403C35   .  83C4 14       ADD ESP,14  
00403C38   .  8985 1CFFFFFF MOV DWORD PTR SS:\[EBP-E4\],EAX                 ;  2  
00403C3E   .  8985 14FFFFFF MOV DWORD PTR SS:\[EBP-EC\],EAX                 ;  2  
00403C44   .  8D45 D8       LEA EAX,DWORD PTR SS:\[EBP-28\]  
00403C47   .  8D8D 14FFFFFF LEA ECX,DWORD PTR SS:\[EBP-EC\]  
00403C4D   .  50            PUSH EAX                                      ;  机器码计算结果  
00403C4E   .  8D95 74FFFFFF LEA EDX,DWORD PTR SS:\[EBP-8C\]  
00403C54   .  51            PUSH ECX                                      ;  2  
00403C55   .  52            PUSH EDX  
00403C56   .  89B5 0CFFFFFF MOV DWORD PTR SS:\[EBP-F4\],ESI  
00403C5C   .  C785 04FFFFFF>MOV DWORD PTR SS:\[EBP-FC\],8002  
00403C66   .  FF15 64114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarMod>\]   ;  MSVBVM60.\_\_vbaVarMod  
00403C6C   .  50            PUSH EAX                                      ;  除 2 的余数  
00403C6D   .  8D85 04FFFFFF LEA EAX,DWORD PTR SS:\[EBP-FC\]  
00403C73   .  50            PUSH EAX                                      ;  与 0 对比  
00403C74   .  FF15 C8104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarTstEq>\] ;  调用 vbaVarTstEq  
00403C7A   .  66:85C0       TEST AX,AX                                    ;  如果返回非 0 说明相等，如果是 0 结果说明不等  
00403C7D      74 7A         JE SHORT crackeme.00403CF9  
00403C7F   .  8D4D B0       LEA ECX,DWORD PTR SS:\[EBP-50\]  
00403C82   .  8D95 74FFFFFF LEA EDX,DWORD PTR SS:\[EBP-8C\]  
00403C88   .  898D 1CFFFFFF MOV DWORD PTR SS:\[EBP-E4\],ECX  
00403C8E   .  52            PUSH EDX  
00403C8F   .  8D85 14FFFFFF LEA EAX,DWORD PTR SS:\[EBP-EC\]  
00403C95   .  6A 05         PUSH 5                                        ;  5  
00403C97   .  8D8D 64FFFFFF LEA ECX,DWORD PTR SS:\[EBP-9C\]  
00403C9D   .  50            PUSH EAX                                      ;  取出用户名的 MD5 值  
00403C9E   .  51            PUSH ECX  
00403C9F   .  C785 7CFFFFFF>MOV DWORD PTR SS:\[EBP-84\],5                   ;  5  
00403CA9   .  C785 74FFFFFF>MOV DWORD PTR SS:\[EBP-8C\],2  
00403CB3   .  C785 14FFFFFF>MOV DWORD PTR SS:\[EBP-EC\],4008  
00403CBD   .  FF15 B0104000 CALL DWORD PTR DS:\[<&MSVBVM60.#632>\]          ;  MSVBVM60.rtcMidCharVar  
00403CC3   .  8D95 64FFFFFF LEA EDX,DWORD PTR SS:\[EBP-9C\]                 ;  从第 5 位开始取 5 位  
00403CC9   .  52            PUSH EDX  
00403CCA   .  FF15 20104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrVarMove>;  MSVBVM60.\_\_vbaStrVarMove  
00403CD0   .  8BD0          MOV EDX,EAX                                   ;  5 位结果  
00403CD2   .  8D4D D4       LEA ECX,DWORD PTR SS:\[EBP-2C\]  
00403CD5   .  FF15 78114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrMove>\]  ;  MSVBVM60.\_\_vbaStrMove  
00403CDB   .  8D85 64FFFFFF LEA EAX,DWORD PTR SS:\[EBP-9C\]  
00403CE1   .  8D8D 74FFFFFF LEA ECX,DWORD PTR SS:\[EBP-8C\]  
00403CE7   .  50            PUSH EAX  
00403CE8   .  51            PUSH ECX  
00403CE9   .  6A 02         PUSH 2  
00403CEB   .  FF15 28104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeVarLis>;  MSVBVM60.\_\_vbaFreeVarList  
00403CF1   .  83C4 0C       ADD ESP,0C  
00403CF4      E9 60010000   JMP crackeme.00403E59  
00403CF9   >  BB 01000000   MOV EBX,1                                     ;  1  
00403CFE   >  B8 05000000   MOV EAX,5                                     ;  5  
00403D03   .  66:3BD8       CMP BX,AX                                     ;  5 位是否取完  
00403D06   .  0F8F 48010000 JG crackeme.00403E54                          ;  循环开始  
00403D0C   .  8D55 B0       LEA EDX,DWORD PTR SS:\[EBP-50\]  
00403D0F   .  8D85 74FFFFFF LEA EAX,DWORD PTR SS:\[EBP-8C\]  
00403D15   .  0FBFCB        MOVSX ECX,BX  
00403D18   .  8995 1CFFFFFF MOV DWORD PTR SS:\[EBP-E4\],EDX  
00403D1E   .  50            PUSH EAX  
00403D1F   .  8D95 14FFFFFF LEA EDX,DWORD PTR SS:\[EBP-EC\]  
00403D25   .  51            PUSH ECX  
00403D26   .  8D85 64FFFFFF LEA EAX,DWORD PTR SS:\[EBP-9C\]  
00403D2C   .  52            PUSH EDX  
00403D2D   .  50            PUSH EAX  
00403D2E   .  C785 7CFFFFFF>MOV DWORD PTR SS:\[EBP-84\],1  
00403D38   .  C785 74FFFFFF>MOV DWORD PTR SS:\[EBP-8C\],2  
00403D42   .  C785 14FFFFFF>MOV DWORD PTR SS:\[EBP-EC\],4008  
00403D4C   .  FF15 B0104000 CALL DWORD PTR DS:\[<&MSVBVM60.#632>\]          ;  MSVBVM60.rtcMidCharVar  
00403D52   .  8B4D D4       MOV ECX,DWORD PTR SS:\[EBP-2C\]  
00403D55   .  8B55 B4       MOV EDX,DWORD PTR SS:\[EBP-4C\]                 ;  字符串 1  
00403D58   .  B8 08000000   MOV EAX,8  
00403D5D   .  898D CCFEFFFF MOV DWORD PTR SS:\[EBP-134\],ECX  
00403D63   .  8985 C4FEFFFF MOV DWORD PTR SS:\[EBP-13C\],EAX  
00403D69   .  8985 F4FEFFFF MOV DWORD PTR SS:\[EBP-10C\],EAX  
00403D6F   .  8D45 AC       LEA EAX,DWORD PTR SS:\[EBP-54\]  
00403D72   .  8D8D 44FFFFFF LEA ECX,DWORD PTR SS:\[EBP-BC\]  
00403D78   .  8995 FCFEFFFF MOV DWORD PTR SS:\[EBP-104\],EDX                ;  字符串 1  
00403D7E   .  8985 ECFEFFFF MOV DWORD PTR SS:\[EBP-114\],EAX                ;  字符串 1  
00403D84   .  51            PUSH ECX                                      ;  MD5 值  
00403D85   .  8D95 F4FEFFFF LEA EDX,DWORD PTR SS:\[EBP-10C\]  
00403D8B   .  6A 01         PUSH 1                                        ;  1  
00403D8D   .  8D85 64FFFFFF LEA EAX,DWORD PTR SS:\[EBP-9C\]  
00403D93   .  52            PUSH EDX                                      ;  字符串 1  
00403D94   .  50            PUSH EAX  
00403D95   .  8D8D 54FFFFFF LEA ECX,DWORD PTR SS:\[EBP-AC\]  
00403D9B   .  56            PUSH ESI  
00403D9C   .  51            PUSH ECX  
00403D9D   .  C785 4CFFFFFF>MOV DWORD PTR SS:\[EBP-B4\],1  
00403DA7   .  C785 44FFFFFF>MOV DWORD PTR SS:\[EBP-BC\],2  
00403DB1   .  C785 E4FEFFFF>MOV DWORD PTR SS:\[EBP-11C\],4008  
00403DBB   .  FF15 08114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaInStrVar>\] ;  MSVBVM60.\_\_vbaInStrVar  
00403DC1   .  50            PUSH EAX                                      ;  从字符串 1 中逐个查找相应的位置  
00403DC2   .  FF15 54114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaI4Var>\]    ;  MSVBVM60.\_\_vbaI4Var  
00403DC8   .  50            PUSH EAX                                      ;  字符串 1 中的位置数  
00403DC9   .  8D95 E4FEFFFF LEA EDX,DWORD PTR SS:\[EBP-11C\]  
00403DCF   .  8D85 34FFFFFF LEA EAX,DWORD PTR SS:\[EBP-CC\]  
00403DD5   .  52            PUSH EDX                                      ;  字符串 2  
00403DD6   .  50            PUSH EAX  
00403DD7   .  FF15 B0104000 CALL DWORD PTR DS:\[<&MSVBVM60.#632>\]          ;  MSVBVM60.rtcMidCharVar  
00403DDD   .  8D8D C4FEFFFF LEA ECX,DWORD PTR SS:\[EBP-13C\]                ;  字符串 2 中查找相应位置  
00403DE3   .  8D95 34FFFFFF LEA EDX,DWORD PTR SS:\[EBP-CC\]  
00403DE9   .  51            PUSH ECX  
00403DEA   .  8D85 24FFFFFF LEA EAX,DWORD PTR SS:\[EBP-DC\]  
00403DF0   .  52            PUSH EDX  
00403DF1   .  50            PUSH EAX  
00403DF2   .  FF15 14114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarCat>\]   ;  MSVBVM60.\_\_vbaVarCat  
00403DF8   .  50            PUSH EAX                                      ;  连接各个找出的位置数  
00403DF9   .  FF15 20104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrVarMove>;  MSVBVM60.\_\_vbaStrVarMove  
00403DFF   .  8BD0          MOV EDX,EAX  
00403E01   .  8D4D D4       LEA ECX,DWORD PTR SS:\[EBP-2C\]  
00403E04   .  FF15 78114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrMove>\]  ;  MSVBVM60.\_\_vbaStrMove  
00403E0A   .  8D8D 24FFFFFF LEA ECX,DWORD PTR SS:\[EBP-DC\]  
00403E10   .  8D95 34FFFFFF LEA EDX,DWORD PTR SS:\[EBP-CC\]  
00403E16   .  51            PUSH ECX  
00403E17   .  8D85 44FFFFFF LEA EAX,DWORD PTR SS:\[EBP-BC\]  
00403E1D   .  52            PUSH EDX  
00403E1E   .  8D8D 54FFFFFF LEA ECX,DWORD PTR SS:\[EBP-AC\]  
00403E24   .  50            PUSH EAX  
00403E25   .  8D95 64FFFFFF LEA EDX,DWORD PTR SS:\[EBP-9C\]  
00403E2B   .  51            PUSH ECX  
00403E2C   .  8D85 74FFFFFF LEA EAX,DWORD PTR SS:\[EBP-8C\]  
00403E32   .  52            PUSH EDX  
00403E33   .  50            PUSH EAX  
00403E34   .  6A 06         PUSH 6  
00403E36   .  FF15 28104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeVarLis>;  MSVBVM60.\_\_vbaFreeVarList  
00403E3C   .  B8 01000000   MOV EAX,1  
00403E41   .  83C4 1C       ADD ESP,1C  
00403E44   .  66:03C3       ADD AX,BX  
00403E47   .  0F80 D7030000 JO crackeme.00404224  
00403E4D   .  8BD8          MOV EBX,EAX  
00403E4F   .^ E9 AAFEFFFF   JMP crackeme.00403CFE                         ;  循环  
00403E54   >  BB 08000000   MOV EBX,8  
00403E59   >  8B4D D4       MOV ECX,DWORD PTR SS:\[EBP-2C\]                 ;  对应的结果  
00403E5C   .  B8 882A4000   MOV EAX,crackeme.00402A88  
00403E61   .  8985 1CFFFFFF MOV DWORD PTR SS:\[EBP-E4\],EAX  
00403E67   .  8985 FCFEFFFF MOV DWORD PTR SS:\[EBP-104\],EAX  
00403E6D   .  898D 0CFFFFFF MOV DWORD PTR SS:\[EBP-F4\],ECX  
00403E73   .  8D85 E4FEFFFF LEA EAX,DWORD PTR SS:\[EBP-11C\]  
00403E79   .  6A 04         PUSH 4                                        ;  4  
00403E7B   .  8D8D 34FFFFFF LEA ECX,DWORD PTR SS:\[EBP-CC\]  
00403E81   .  8D55 B0       LEA EDX,DWORD PTR SS:\[EBP-50\]  
00403E84   .  50            PUSH EAX                                      ;  MD5 值  
00403E85   .  51            PUSH ECX  
00403E86   .  899D 14FFFFFF MOV DWORD PTR SS:\[EBP-EC\],EBX  
00403E8C   .  899D 04FFFFFF MOV DWORD PTR SS:\[EBP-FC\],EBX  
00403E92   .  899D F4FEFFFF MOV DWORD PTR SS:\[EBP-10C\],EBX  
00403E98   .  8995 ECFEFFFF MOV DWORD PTR SS:\[EBP-114\],EDX  
00403E9E   .  C785 E4FEFFFF>MOV DWORD PTR SS:\[EBP-11C\],4008  
00403EA8   .  FF15 7C114000 CALL DWORD PTR DS:\[<&MSVBVM60.#619>\]          ;  MSVBVM60.rtcRightCharVar  
00403EAE   .  8D55 C0       LEA EDX,DWORD PTR SS:\[EBP-40\]                 ;  取 MD5 值的最后四位  
00403EB1   .  8D45 D8       LEA EAX,DWORD PTR SS:\[EBP-28\]  
00403EB4   .  52            PUSH EDX                                      ;  取出邮箱名计算结果  
00403EB5   .  8D8D 74FFFFFF LEA ECX,DWORD PTR SS:\[EBP-8C\]  
00403EBB   .  50            PUSH EAX                                      ;  取出机器码计算结果  
00403EBC   .  51            PUSH ECX  
00403EBD   .  FF15 5C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarXor>\]   ;  MSVBVM60.\_\_vbaVarXor  
00403EC3   .  8B1D 14114000 MOV EBX,DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarCat>>;  MSVBVM60.\_\_vbaVarCat  
00403EC9   .  50            PUSH EAX                                      ;  两者异或的结果  
00403ECA   .  8D95 14FFFFFF LEA EDX,DWORD PTR SS:\[EBP-EC\]  
00403ED0   .  8D85 64FFFFFF LEA EAX,DWORD PTR SS:\[EBP-9C\]  
00403ED6   .  52            PUSH EDX  
00403ED7   .  50            PUSH EAX  
00403ED8   .  FFD3          CALL EBX                                      ;  <&MSVBVM60.\_\_vbaVarCat>  
00403EDA   .  8D8D 04FFFFFF LEA ECX,DWORD PTR SS:\[EBP-FC\]                 ;  连接  
00403EE0   .  50            PUSH EAX  
00403EE1   .  8D95 54FFFFFF LEA EDX,DWORD PTR SS:\[EBP-AC\]  
00403EE7   .  51            PUSH ECX  
00403EE8   .  52            PUSH EDX  
00403EE9   .  FFD3          CALL EBX  
00403EEB   .  50            PUSH EAX                                      ;  异或结果连接 5 位的字符串  
00403EEC   .  8D85 F4FEFFFF LEA EAX,DWORD PTR SS:\[EBP-10C\]  
00403EF2   .  8D8D 44FFFFFF LEA ECX,DWORD PTR SS:\[EBP-BC\]  
00403EF8   .  50            PUSH EAX  
00403EF9   .  51            PUSH ECX  
00403EFA   .  FFD3          CALL EBX  
00403EFC   .  50            PUSH EAX  
00403EFD   .  8D95 34FFFFFF LEA EDX,DWORD PTR SS:\[EBP-CC\]  
00403F03   .  8D85 24FFFFFF LEA EAX,DWORD PTR SS:\[EBP-DC\]  
00403F09   .  52            PUSH EDX  
00403F0A   .  50            PUSH EAX  
00403F0B   .  FFD3          CALL EBX  
00403F0D   .  8BD0          MOV EDX,EAX                                   ;  连接 MD5 值的最后 4 位  
00403F0F   .  8D4D 9C       LEA ECX,DWORD PTR SS:\[EBP-64\]  
00403F12   .  FF15 10104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarMove>\]  ;  MSVBVM60.\_\_vbaVarMove  
00403F18   .  8B1D 28104000 MOV EBX,DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeVar>;  MSVBVM60.\_\_vbaFreeVarList  
00403F1E   .  8D8D 34FFFFFF LEA ECX,DWORD PTR SS:\[EBP-CC\]  
00403F24   .  8D95 44FFFFFF LEA EDX,DWORD PTR SS:\[EBP-BC\]  
00403F2A   .  51            PUSH ECX  
00403F2B   .  8D85 54FFFFFF LEA EAX,DWORD PTR SS:\[EBP-AC\]  
00403F31   .  52            PUSH EDX  
00403F32   .  8D8D 64FFFFFF LEA ECX,DWORD PTR SS:\[EBP-9C\]  
00403F38   .  50            PUSH EAX  
00403F39   .  51            PUSH ECX  
00403F3A   .  6A 04         PUSH 4  
00403F3C   .  FFD3          CALL EBX                                      ;  <&MSVBVM60.\_\_vbaFreeVarList>  
00403F3E   .  8B17          MOV EDX,DWORD PTR DS:\[EDI\]  
00403F40   .  83C4 14       ADD ESP,14  
00403F43   .  57            PUSH EDI  
00403F44   .  FF92 04030000 CALL DWORD PTR DS:\[EDX+304\]  
00403F4A   .  50            PUSH EAX  
00403F4B   .  8D45 88       LEA EAX,DWORD PTR SS:\[EBP-78\]  
00403F4E   .  50            PUSH EAX  
00403F4F   .  FF15 7C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaObjSet>\]   ;  MSVBVM60.\_\_vbaObjSet  
00403F55   .  8BF8          MOV EDI,EAX  
00403F57   .  8D55 98       LEA EDX,DWORD PTR SS:\[EBP-68\]  
00403F5A   .  52            PUSH EDX  
00403F5B   .  57            PUSH EDI  
00403F5C   .  8B0F          MOV ECX,DWORD PTR DS:\[EDI\]  
00403F5E   .  FF91 A0000000 CALL DWORD PTR DS:\[ECX+A0\]  
00403F64   .  3BC6          CMP EAX,ESI  
00403F66   .  DBE2          FCLEX  
00403F68   .  7D 12         JGE SHORT crackeme.00403F7C  
00403F6A   .  68 A0000000   PUSH 0A0  
00403F6F   .  68 B8294000   PUSH crackeme.004029B8  
00403F74   .  57            PUSH EDI  
00403F75   .  50            PUSH EAX  
00403F76   .  FF15 4C104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresultChe>;  MSVBVM60.\_\_vbaHresultCheckObj  
00403F7C   >  8B45 98       MOV EAX,DWORD PTR SS:\[EBP-68\]                 ;  取出假注册码  
00403F7F   .  8D4D 9C       LEA ECX,DWORD PTR SS:\[EBP-64\]  
00403F82   .  8985 7CFFFFFF MOV DWORD PTR SS:\[EBP-84\],EAX  
00403F88   .  8D85 74FFFFFF LEA EAX,DWORD PTR SS:\[EBP-8C\]  
00403F8E   .  50            PUSH EAX                                      ;  假注册码  
00403F8F   .  51            PUSH ECX                                      ;  真注册码  
00403F90   .  8975 98       MOV DWORD PTR SS:\[EBP-68\],ESI  
00403F93   .  C785 74FFFFFF>MOV DWORD PTR SS:\[EBP-8C\],8008  
00403F9D   .  FF15 C8104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaVarTstEq>\] ;  MSVBVM60.\_\_vbaVarTstEq  
00403FA3   .  8D4D 88       LEA ECX,DWORD PTR SS:\[EBP-78\]                 ;  对比  
00403FA6   .  8BF8          MOV EDI,EAX  
00403FA8   .  FF15 9C114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeObj>\]  ;  MSVBVM60.\_\_vbaFreeObj  
00403FAE   .  8D8D 74FFFFFF LEA ECX,DWORD PTR SS:\[EBP-8C\]  
00403FB4   .  FF15 18104000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeVar>\]  ;  MSVBVM60.\_\_vbaFreeVar  
00403FBA   .  66:3BFE       CMP DI,SI  
00403FBD   .  C785 7CFFFFFF>MOV DWORD PTR SS:\[EBP-84\],4B  
00403FC7   .  C785 74FFFFFF>MOV DWORD PTR SS:\[EBP-8C\],2  
00403FD1   .  BA E4294000   MOV EDX,crackeme.004029E4  
00403FD6   .  8D4D 98       LEA ECX,DWORD PTR SS:\[EBP-68\]  
00403FD9   .  0F84 C1000000 JE crackeme.004040A0                          ;  爆破点  
00403FDF   .  FF15 34114000 CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrCopy>\]  ;  MSVBVM60.\_\_vbaStrCopy  
00403FE5   .  8D95 74FFFFFF LEA EDX,DWORD PTR SS:\[EBP-8C\]  
00403FEB   .  8D45 98       LEA EAX,DWORD PTR SS:\[EBP-68\]  
00403FEE   .  52            PUSH EDX  
00403FEF   .  50            PUSH EAX  
  
  
算法总结：  
   字符串 1 为"1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ"  
   字符串 2 为"crackme6-lxbylangxangpostinbbs.pediy.com"  
1、取用户名的 16 位 MD5 值（即 32 位 MD5 值的第 8-24 位）转化为大写，在结果中从第 5 位开始取 5 位记为 A，并在字符串 1 中找出 A 各个字符的位置，然后在字符串 2 中找出相应的位置组成新的字符串 B。  
2、将邮箱名每个字符的 ASCII 码逻辑右移 2 位后累加计为 SUM。  
3、将机器码每个字符的 ASCII 码异或 SUM 后再累加计为 JQM。  
4、判断 JQM 为奇数还是偶数，如果是奇数，注册码为 SUM 和 JQM 的异或值“-”号连接 B“-”号连接用户名的 16 位 MD5 值的后 4 位，如果是偶数，注册码为 SUM 和 JQM 的异或值“-”号连接 A“-”号连接用户名的 16 位 MD5 值的后 4 位。  
  
现在我们来根据算法写出注册机（源码见附件），注释我就不写了（为了让你们自己去查）。  
一、VB，四个文本框加按钮：  
Public Function SHR(iShr As Long, n As Integer) As Long 'SHR 逻辑移位函数  
  Dim i As Integer  
  For i = 1 To n - 1  
    iShr = iShr \\ 2  
  Next i  
  CF = iShr And 1  
  SHR = iShr \\ 2  
End Function  
Private Sub Command1\_Click()  
   Dim str1 As String, str2 As String  
   Dim a As String, temp As String  
   Dim i As Integer  
   Dim c, sum, middle  
   
  str1 = "1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ"  
  str2 = "crackme6-lxbylangxangpostinbbs.pediy.com"  
  If IsNumeric(Text1.Text) = False Then  
  MsgBox "机器码为数字组成！", 48, "提示"  
  Else  
  If Len(Text2.Text) = 0 Or Len(Text2.Text) > 16 Then  
    MsgBox "用户名不能为空,长度小于 16 位！", 48, "提示"  
  Else  
  If Text3.Text = "" Then  
  MsgBox "请输入邮箱名！", 48, "提示"  
  Else  
  For i = 1 To Len(Text3.Text)  
  sum = sum + SHR(Asc(Mid(Text3.Text, i, 1)), 2)  
  Next  
  For i = 1 To Len(Text1.Text)  
  jqm = jqm + (Asc(Mid(Text1.Text, i, 1)) Xor sum)  
  Next  
  a = UCase(MD5(Text2.Text, 16))  
  temp = Mid(a, 5, 5)  
  If jqm Mod 2 = 0 Then  
  Text4.Text = (sum Xor jqm) & "-" & temp & "-" & Right(a, 4)  
  Else  
  For i = 1 To Len(temp)  
  middle = middle & Mid(str2, InStr(1, str1, Mid(temp, i, 1)), 1)  
  Next i  
  Text4.Text = (sum Xor jqm) & "-" & middle & "-" & Right(a, 4)  
  End If  
  End If  
  End If  
  End If  
  
二、DELPHI  
procedure TForm1.Button1Click(Sender: TObject);  
VAR  
str1,str2:string;  
i,sum,jqm:integer;  
a,b,c:string;  
begin  
     str1 := '1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ';  
     str2 := 'crackme6-lxbylangxangpostinbbs.pediy.com'  ;  
   If Length(edit2.text) = 0 Then MessageBox(0, '用户名不能为空！', '提示', MB\_OK + MB\_ICONWARNING)  
   else  
   If Length(edit3.text) = 0 Then  MessageBox(0, '邮箱名不能为空！', '提示', MB\_OK + MB\_ICONWARNING)  
   else  
   begin  
      sum:=0;  
      for i:=0 to length(edit3.text)-1 do  
      begin  
      sum:=sum+(ord(edit3.text\[i\])shr 2);  
      end;  
      jqm:=0;  
      for i:=0 to length(edit1.text)-1 do  
      begin  
      jqm:=jqm+(ord(edit1.text\[i\])xor sum);  
      end;  
      a :=UpperCase(md5print(md5string(edit2.text)));  
      b :=copy(a,13,5);  
      For i := 0 to Length(b)-1 do  
    begin  
      c := c + Str2\[Pos(b\[i\],Str1)\];  
    end;  
      if jqm mod 2 =0 then  edit4.text:=inttostr(sum xor jqm) + '-' + b + '-' + copy(a,21,4)  
      else  
      edit4.text:=inttostr(sum xor jqm) + '-' + c + '-' + copy(a,21,4);  
   end;  
end;  
  
三、MFC  
void CMDDlg::OnButtonResult()   
{  
  // TODO: Add your control notification handler code here  
    int i,sum=0,jqm=0,pos\[20\];  
  CString str,str0,last,a,b;  
    CString str1 = "1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ";  
    CString str2 = "crackme6-lxbylangxangpostinbbs.pediy.com";  
    char cName\[255\],cEmail\[255\],cMachine\[255\],first\[255\];  
    const char\* temp;  
        
      UpdateData(TRUE);  
       if(m\_Edit1.GetLength()==0 )MessageBox("机器码不能为空！", "error", MB\_OK);   
     else  
       if(m\_Edit2.GetLength()==0 )MessageBox("用户名不能为空！", "error", MB\_OK);   
     else  
     if(m\_Edit3.GetLength()==0 )MessageBox("邮箱名不能为空！", "error", MB\_OK);   
     else{  
     strcpy(cMachine,m\_Edit1);  
     strcpy(cName,m\_Edit2);  
     strcpy(cEmail,m\_Edit3);  
     for (i=0;i<m\_Edit3.GetLength ();i++) {  
       cEmail\[i\]=cEmail\[i\]>> 2 ;  
           sum+= cEmail\[i\];  
     }  
       for (i=0;i<m\_Edit1.GetLength ();i++) {  
       cMachine\[i\] ^=sum;  
           jqm+=cMachine\[i\];  
     }  
     jqm^=sum;  
     \_itoa(jqm,first,10);   
     a=first;  
       
    CMD5 md5;  
    md5.setPlainText(m\_Edit2);  
    temp = md5.getMD5Digest();  
    str=temp;  
    str.MakeUpper();  
    str0 = str.Mid(12,5);   
        last=str.Mid(20,4);  
        for (i=0;i<str0.GetLength ();i++){  
    pos\[i\]=str1.Find(str0\[i\]);  
        b+=str2.Mid(pos\[i\],1);  
    }  
    if(jqm%2==0)    m\_Edit4=a+'-'+str0+'-'+last;  
    else {  
      m\_Edit4=a+'-'+b+'-'+last;    
    }  
     }  
    UpdateData(FALSE);  
}  
  
    最好自己尝试写一下，这样也好消化。

附件下载:keygen4.rar  
  
【版权声明】:  本文由 langxang 原创于看雪论坛，转载请注明作者并保持文章的完整, 谢谢! 

 4dd466d0e1e0cc123afbe504da2462ca