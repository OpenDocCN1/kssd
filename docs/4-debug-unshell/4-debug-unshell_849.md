

# PECompact 2.x -> Jeremy Collake 的壳脱好后无法修复

> 原文：[`www.pediy.com/kssd/pediy10/74691.html`](https://www.pediy.com/kssd/pediy10/74691.html)

PECompact 2.x -> Jeremy Collake 的壳，脱好后无法运行，不知道怎么修复． 

以前调试时记的一个笔记，不需要 ImportREC 脱 PECompact 的方法：  
  
载入程序后断 BP VirtualAlloc，两次中断后会返回到这里：  
  
00EF0A54    6A 40           PUSH 40  
00EF0A56    68 00100000     PUSH 1000  
00EF0A5B    51              PUSH ECX  
00EF0A5C    6A 00           PUSH 0  
00EF0A5E    FF95 371F0010   CALL DWORD PTR SS:\[EBP+10001F37\]  
00EF0A64    8985 271F0010   MOV DWORD PTR SS:\[EBP+10001F27\],EAX  
00EF0A6A    56              PUSH ESI  
00EF0A6B    E8 F6030000     CALL 00EF0E66                                      ; 解码区段  
00EF0A70    8D8D BD1D0010   LEA ECX,DWORD PTR SS:\[EBP+10001DBD\]                ; ASCII "Application corrupt."，返回位置，内存中可以看到前面的字串  
00EF0A76    85C0            TEST EAX,EAX  
00EF0A78    0F85 94000000   JNZ 00EF0B12  
00EF0A7E    56              PUSH ESI  
00EF0A7F    E8 40030000     CALL 00EF0DC4                                      ; 这里面用解码后的内容填充原程序的各个区段，F7 跟进  
00EF0A84    56              PUSH ESI  
00EF0A85    E8 55020000     CALL 00EF0CDF  
00EF0A8A    90              NOP  
00EF0A8B    90              NOP  
00EF0A8C    90              NOP  
00EF0A8D    90              NOP  
00EF0A8E    90              NOP  
00EF0A8F    90              NOP  
00EF0A90    90              NOP  
00EF0A91    90              NOP  
00EF0A92    90              NOP  
00EF0A93    90              NOP  
00EF0A94    90              NOP  
00EF0A95    90              NOP  
00EF0A96    90              NOP  
00EF0A97    90              NOP  
00EF0A98    8B4E 34         MOV ECX,DWORD PTR DS:\[ESI+34\]  
00EF0A9B    85C9            TEST ECX,ECX  
00EF0A9D    0F84 89000000   JE 00EF0B2C  
00EF0AA3    034E 08         ADD ECX,DWORD PTR DS:\[ESI+8\]  
00EF0AA6    51              PUSH ECX  
00EF0AA7    56              PUSH ESI  
00EF0AA8    E8 47060000     CALL 00EF10F4                                      ; 处理输入表  
00EF0AAD    85C0            TEST EAX,EAX  
00EF0AAF    74 7B           JE SHORT 00EF0B2C  
\--------------------------------------------------------------------------------------------------  
一、区段：  
  
跟进 00EF0A7F 的 CALL：  
  
00EF0DC4    55              PUSH EBP                                           ; 00EF0A7F CALL 的起始地址，一路 F8  
00EF0DC5    8BEC            MOV EBP,ESP  
00EF0DC7    83C4 E8         ADD ESP,-18  
00EF0DCA    53              PUSH EBX  
00EF0DCB    57              PUSH EDI  
00EF0DCC    56              PUSH ESI  
00EF0DCD    E8 00000000     CALL 00EF0DD2  
00EF0DD2    5B              POP EBX  
00EF0DD3    81EB 41170010   SUB EBX,10001741  
00EF0DD9    8B75 08         MOV ESI,DWORD PTR SS:\[EBP+8\]  
00EF0DDC    33C0            XOR EAX,EAX  
00EF0DDE    33C9            XOR ECX,ECX  
00EF0DE0    034E 3C         ADD ECX,DWORD PTR DS:\[ESI+3C\]  
00EF0DE3    74 0A           JE SHORT 00EF0DEF  
00EF0DE5    8B56 08         MOV EDX,DWORD PTR DS:\[ESI+8\]  
00EF0DE8    8BFE            MOV EDI,ESI  
00EF0DEA    83C6 50         ADD ESI,50  
00EF0DED    EB 07           JMP SHORT 00EF0DF6  
00EF0DEF    5E              POP ESI  
00EF0DF0    5F              POP EDI  
00EF0DF1    5B              POP EBX  
00EF0DF2    C9              LEAVE  
00EF0DF3    C2 0400         RETN 4  
00EF0DF6    8B83 271F0010   MOV EAX,DWORD PTR DS:\[EBX+10001F27\]                ; 壳解码原程序数据的临时空间  
00EF0DFC    8945 FC         MOV DWORD PTR SS:\[EBP-4\],EAX  
00EF0DFF    8BDE            MOV EBX,ESI  
00EF0E01    0FB743 10       MOVZX EAX,WORD PTR DS:\[EBX+10\]  
00EF0E05    A9 02000000     TEST EAX,2  
00EF0E0A    74 4D           JE SHORT 00EF0E59  
00EF0E0C    51              PUSH ECX  
00EF0E0D    8B73 04         MOV ESI,DWORD PTR DS:\[EBX+4\]  
00EF0E10    8B7D FC         MOV EDI,DWORD PTR SS:\[EBP-4\]  
00EF0E13    8B4B 08         MOV ECX,DWORD PTR DS:\[EBX+8\]  
00EF0E16    03F2            ADD ESI,EDX  
00EF0E18    8BC1            MOV EAX,ECX  
00EF0E1A    C1F9 02         SAR ECX,2  
00EF0E1D    F3:A5           REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]  
00EF0E1F    03C8            ADD ECX,EAX  
00EF0E21    83E1 03         AND ECX,3  
00EF0E24    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]  
00EF0E26    8B7B 04         MOV EDI,DWORD PTR DS:\[EBX+4\]  
00EF0E29    8B4B 08         MOV ECX,DWORD PTR DS:\[EBX+8\]  
00EF0E2C    03FA            ADD EDI,EDX  
00EF0E2E    33C0            XOR EAX,EAX  
00EF0E30    52              PUSH EDX  
00EF0E31    8BD1            MOV EDX,ECX  
00EF0E33    C1F9 02         SAR ECX,2  
00EF0E36    F3:AB           REP STOS DWORD PTR ES:\[EDI\]  
00EF0E38    03CA            ADD ECX,EDX  
00EF0E3A    83E1 03         AND ECX,3  
00EF0E3D    F3:AA           REP STOS BYTE PTR ES:\[EDI\]  
00EF0E3F    5A              POP EDX  
00EF0E40    8B75 FC         MOV ESI,DWORD PTR SS:\[EBP-4\]  
00EF0E43    8B3B            MOV EDI,DWORD PTR DS:\[EBX\]  
00EF0E45    03FA            ADD EDI,EDX                                        ; 各个区段的开始偏移加上基址，从最后一个开始加  
00EF0E47    8B4B 08         MOV ECX,DWORD PTR DS:\[EBX+8\]  
00EF0E4A    8BC1            MOV EAX,ECX  
00EF0E4C    C1F9 02         SAR ECX,2  
00EF0E4F    F3:A5           REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]     ; 填充原程序各区段数据  
00EF0E51    03C8            ADD ECX,EAX  
00EF0E53    83E1 03         AND ECX,3  
00EF0E56    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]  
00EF0E58    59              POP ECX  
00EF0E59    83C3 1C         ADD EBX,1C  
00EF0E5C    49              DEC ECX  
00EF0E5D  ^ 75 A2           JNZ SHORT 00EF0E01  
00EF0E5F    5E              POP ESI  
00EF0E60    5F              POP EDI  
00EF0E61    5B              POP EBX  
00EF0E62    C9              LEAVE  
00EF0E63    C2 0400         RETN 4  
\--------------------------------------------------------------------------------------------------  
二、输入表处理  
  
跟进 00EF0AA8 处的那个处理输入表的 CALL：  
  
00EF10F4    55              PUSH EBP  
00EF10F5    8BEC            MOV EBP,ESP  
00EF10F7    53              PUSH EBX  
00EF10F8    57              PUSH EDI  
00EF10F9    56              PUSH ESI  
00EF10FA    8B75 0C         MOV ESI,DWORD PTR SS:\[EBP+C\]  
00EF10FD    8B5D 08         MOV EBX,DWORD PTR SS:\[EBP+8\]  
00EF1100    33C0            XOR EAX,EAX  
00EF1102    3946 10         CMP DWORD PTR DS:\[ESI+10\],EAX  
00EF1105    75 04           JNZ SHORT 00EF110B  
00EF1107    3906            CMP DWORD PTR DS:\[ESI\],EAX  
00EF1109    74 24           JE SHORT 00EF112F  
00EF110B    0306            ADD EAX,DWORD PTR DS:\[ESI\]  
00EF110D    74 03           JE SHORT 00EF1112  
00EF110F    0343 08         ADD EAX,DWORD PTR DS:\[EBX+8\]  
00EF1112    8B4E 0C         MOV ECX,DWORD PTR DS:\[ESI+C\]  
00EF1115    034B 08         ADD ECX,DWORD PTR DS:\[EBX+8\]  
00EF1118    8B7E 10         MOV EDI,DWORD PTR DS:\[ESI+10\]  
00EF111B    85FF            TEST EDI,EDI  
00EF111D    74 03           JE SHORT 00EF1122  
00EF111F    037B 08         ADD EDI,DWORD PTR DS:\[EBX+8\]  
00EF1122    50              PUSH EAX  
00EF1123    57              PUSH EDI  
00EF1124    51              PUSH ECX  
00EF1125    53              PUSH EBX  
00EF1126    E8 1F000000     CALL 00EF114A                                      ; 跟进去，这里面处理各个 DLL 的输入表  
00EF112B    40              INC EAX  
00EF112C    75 08           JNZ SHORT 00EF1136                                 ; 判断是否所有的 DLL 都处理完了  
00EF112E    48              DEC EAX  
00EF112F    5E              POP ESI  
00EF1130    5F              POP EDI  
00EF1131    5B              POP EBX  
00EF1132    C9              LEAVE  
00EF1133    C2 0800         RETN 8  
00EF1136    83C6 14         ADD ESI,14  
00EF1139  ^ EB C5           JMP SHORT 00EF1100  
\--------------------------------------------------------------------------------------------------  
跟进 00EF1126 的那个 CALL：  
  
00EF114A    55              PUSH EBP  
00EF114B    8BEC            MOV EBP,ESP  
省略若干代码...  
00EF1195    C9              LEAVE  
00EF1196    C2 1000         RETN 10  
00EF1199    90              NOP  
这里都是 NOP  
00EF11C3    8B75 10         MOV ESI,DWORD PTR SS:\[EBP+10\]   
00EF11C6    8B7D 08         MOV EDI,DWORD PTR SS:\[EBP+8\]  
省略若干代码...  
00EF120D    50              PUSH EAX  
00EF120E    FF75 FC         PUSH DWORD PTR SS:\[EBP-4\]  
00EF1211    FF93 1F1F0010   CALL DWORD PTR DS:\[EBX+10001F1F\]  
00EF1217    5A              POP EDX  
00EF1218    85C0            TEST EAX,EAX  
00EF121A  ^ 0F84 6FFFFFFF   JE 00EF118F                                        ; 下面两句改掉，这样不会用得到的函数地址填充输入表，可以得到原始的输入表，只要填一下 dump 文件的输入表相关 RVA 及大小即可  
00EF1220    8906            MOV DWORD PTR DS:\[ESI\],EAX                         ; 关键，改成 mov eax,\[edx\]  
00EF1222    8902            MOV DWORD PTR DS:\[EDX\],EAX                         ; 改成 mov dword ptr \[esi\],eax  
00EF1224    83C2 04         ADD EDX,4  
00EF1227    83C6 04         ADD ESI,4  
00EF122A  ^ EB AC           JMP SHORT 00EF11D8  
00EF122C    33C0            XOR EAX,EAX  
00EF122E    5E              POP ESI  
00EF122F    5F              POP EDI  
00EF1230    5B              POP EBX  
00EF1231    C9              LEAVE  
00EF1232    C2 1000         RETN 10  
\--------------------------------------------------------------------------------------------------  
现在就可以直接运行到 OEP dump 了。