

# ASProtec 1.23 b18

> 原文：[`www.pediy.com/kssd/pediy05/pediy50141.htm`](https://www.pediy.com/kssd/pediy05/pediy50141.htm)

ASProtect 1.23 b18   
  
004D1001 >90                NOP   
004D1002  60                PUSHAD   
004D1003  E8 03000000      CALL ASPROTEC.004D100B   
004D1008  90                NOP\==================>花指令   
004D1009  EB 04            JMP SHORT ASPROTEC.004D100F   
004D100B  5D                POP EBP   
004D100C  45                INC EBP   
004D100D  55                PUSH EBP   
004D100E  C3                RETN   
  
004D100F  E8 01000000      CALL ASPROTEC.004D1015   
004D1014  90                NOP\==================>花指令   
004D1015  5D                POP EBP   
004D1016  BB ECFFFFFF      MOV EBX,-14   
004D101B  03DD              ADD EBX,EBP   
004D101D  81EB 00100D00    SUB EBX,0D1000   
004D1023  83BD 22040000 00  CMP DWORD PTR SS:\[EBP+422\],0   
004D102A  899D 22040000    MOV DWORD PTR SS:\[EBP+422\],EBX\==========>保存 ImageBase，   
                                                                    也就是 GetModuleHandleA(0)   
004D1030  0F85 65030000    JNZ ASPROTEC.004D139B   
004D1036  8D85 2E040000    LEA EAX,DWORD PTR SS:\[EBP+42E\]==========>指向 kernel32.dll   
004D103C  50                PUSH EAX   
004D103D  FF95 4C0F0000    CALL DWORD PTR SS:\[EBP+F4C\]=============>GetModuleHandleA()   
004D1043  8985 26040000    MOV DWORD PTR SS:\[EBP+426\],EAX   
004D1049  8BF8              MOV EDI,EAX   
004D104B  8D5D 5E          LEA EBX,DWORD PTR SS:\[EBP+5E\]   
004D104E  53                PUSH EBX\================================>VirtualAlloc   
004D104F  50                PUSH EAX   
004D1050  FF95 480F0000    CALL DWORD PTR SS:\[EBP+F48\]=============>GetProcAddress()   
004D1056  8985 4C050000    MOV DWORD PTR SS:\[EBP+54C\],EAX   
004D105C  8D5D 6B          LEA EBX,DWORD PTR SS:\[EBP+6B\]   
004D105F  53                PUSH EBX\================================>VirtualFree   
004D1060  57                PUSH EDI   
004D1061  FF95 480F0000    CALL DWORD PTR SS:\[EBP+F48\]=============>GetProcAddress()   
004D1067  8985 50050000    MOV DWORD PTR SS:\[EBP+550\],EAX   
.............   
...........   
004D1279  BE 00D00900      MOV ESI,9D000   
004D127E  8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]   
004D1284  03F2              ADD ESI,EDX   
  
004D1286  8B46 0C          MOV EAX,DWORD PTR DS:\[ESI+C\]======>每一个   
004D1289  85C0              TEST EAX,EAX   
004D128B  0F84 0A010000    JE ASPROTEC.004D139B=============>不存在导入 DLL   
004D1291  03C2              ADD EAX,EDX   
004D1293  8BD8              MOV EBX,EAX   
004D1295  50                PUSH EAX\========================>为原始导入表目录   
004D1296  FF95 4C0F0000    CALL DWORD PTR SS:\[EBP+F4C\]=====>GetModuleHandleA('')   
004D129C  85C0              TEST EAX,EAX\====================>   
004D129E  75 07            JNZ SHORT ASPROTEC.004D12A7   
004D12A0  53                PUSH EBX   
004D12A1  FF95 500F0000    CALL DWORD PTR SS:\[EBP+F50\]   
004D12A7  8985 44050000    MOV DWORD PTR SS:\[EBP+544\],EAX   
004D12AD  C785 48050000 000>MOV DWORD PTR SS:\[EBP+548\],0   
004D12B7  8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]   
004D12BD  8B06              MOV EAX,DWORD PTR DS:\[ESI\]   
004D12BF  85C0              TEST EAX,EAX   
004D12C1  75 03            JNZ SHORT ASPROTEC.004D12C6   
004D12C3  8B46 10          MOV EAX,DWORD PTR DS:\[ESI+10\]   
004D12C6  03C2              ADD EAX,EDX   
004D12C8  0385 48050000    ADD EAX,DWORD PTR SS:\[EBP+548\]   
004D12CE  8B18              MOV EBX,DWORD PTR DS:\[EAX\]   
004D12D0  8B7E 10          MOV EDI,DWORD PTR DS:\[ESI+10\]   
004D12D3  03FA              ADD EDI,EDX   
004D12D5  03BD 48050000    ADD EDI,DWORD PTR SS:\[EBP+548\]   
004D12DB  85DB              TEST EBX,EBX   
004D12DD  0F84 A2000000    JE ASPROTEC.004D1385   
004D12E3  F7C3 00000080    TEST EBX,80000000==============>检测导入类型   
\=============================>INT(Import Name Table)还是 IAT(Import Address Table)   
004D12E9  75 04            JNZ SHORT ASPROTEC.004D12EF   
004D12EB  03DA              ADD EBX,EDX   
004D12ED  43                INC EBX   
004D12EE  43                INC EBX   
004D12EF  53                PUSH EBX   
004D12F0  81E3 FFFFFF7F    AND EBX,7FFFFFFF   
004D12F6  53                PUSH EBX   
004D12F7  FFB5 44050000    PUSH DWORD PTR SS:\[EBP+544\]   
004D12FD  FF95 480F0000    CALL DWORD PTR SS:\[EBP+F48\]=======>GetProcAddress()   
004D1303  85C0              TEST EAX,EAX   
004D1305  5B                POP EBX   
004D1306  75 6F            JNZ SHORT ASPROTEC.004D1377   
004D12D5  03BD 48050000    ADD EDI,DWORD PTR SS:\[EBP+548\]   
004D12DB  85DB              TEST EBX,EBX   
004D12DD  0F84 A2000000    JE ASPROTEC.004D1385=============>完成一个 DLL 的覆盖，继续下一个   
  
004D1377  8907              MOV DWORD PTR DS:\[EDI\],EAX<========IAT 重写   
004D1379  8385 48050000 04  ADD DWORD PTR SS:\[EBP+548\],4   
004D1380  E9 32FFFFFF      JMP ASPROTEC.004D12B7   
004D1385  8906              MOV DWORD PTR DS:\[ESI\],EAX   
004D1387  8946 0C          MOV DWORD PTR DS:\[ESI+C\],EAX   
004D138A  8946 10          MOV DWORD PTR DS:\[ESI+10\],EAX   
004D138D  83C6 14          ADD ESI,14   
004D1390  8B95 22040000    MOV EDX,DWORD PTR SS:\[EBP+422\]   
004D1396  E9 EBFEFFFF      JMP ASPROTEC.004D1286=============>循环处理下一个 DLL   
  
\=====================>导入表重写完成   
004D139B  B8 64DD0800      MOV EAX,8DD64===========>原导入 RVA   
004D13A0  50                PUSH EAX   
004D13A1  0385 22040000    ADD EAX,DWORD PTR SS:\[EBP+422\]   
004D13A7  59                POP ECX   
004D13A8  0BC9              OR ECX,ECX   
004D13AA  8985 A8030000    MOV DWORD PTR SS:\[EBP+3A8\],EAX\====>修改 OEP   
004D13B0  61                POPAD   
004D13B1  75 08            JNZ SHORT ASPROTEC.004D13BB   
004D13B3  B8 01000000      MOV EAX,1   
004D13B8  C2 0C00          RETN 0C   
004D13BB  68 00000000      PUSH 0============>Here 返回到 OEP   
004D13C0  C3                RETN   
  
0048DD64  This is OEP  4dd466d0e1e0cc123afbe504da2462ca