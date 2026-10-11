
# `UPX` 0.89.6 - 1.02 / 1.05 - 1.24 -> `Markus` & `Laszlo` `\[Overlay\]` 壳分析

> 原文：`[`www.pediy.com/kssd/pediy11/123389.html`](https://www.pediy.com/kssd/pediy11/123389.html)`

代码：



```
006005D0 > $  60            PUSHAD
006005D1   .  BE 00C05400   MOV ESI,VSClient.0054C000               ;  ESI 指向 UPX1 块
006005D6   .  8DBE 0050EBFF LEA EDI,DWORD PTR DS:\[ESI+FFEB5000\]   ;  EDI 指向 UPX0 块
006005DC   .  57            PUSH EDI
006005DD   .  83CD FF       OR EBP,FFFFFFFF
006005E0   .  EB 10         JMP SHORT VSClient.006005F2
006005E2     90            NOP
006005E3     90            NOP
006005E4     90            NOP
006005E5     90            NOP
006005E6     90            NOP
006005E7     90            NOP
006005E8   >  8A06          MOV AL,BYTE PTR DS:\[ESI\]
006005EA   .  46            INC ESI
006005EB   .  8807          MOV BYTE PTR DS:\[EDI\],AL
006005ED   .  47            INC EDI
006005EE   >  01DB          ADD EBX,EBX
006005F0   .  75 07        JNZ SHORT VSClient.006005F9             ;  EBX!=0 && EBX!=80000000h 跳转
006005F2   >  8B1E          MOV EBX,DWORD PTR DS:\[ESI\]           ;  EBX 为标志控制字节;EBX 无论为何值,都会 ADD 32 次，因为 sub esi,-4 后 CF 恒为 1
006005F4   .  83EE FC       SUB ESI,-4                             ;  影响 CF 位,CF=1;正数减负数 CF=1,CF flags  indicate a borrow
006005F7   .  11DB          ADC EBX,EBX                            ;  EBX=EBX+EBX+CF
006005F9   >^ 72 ED         JB SHORT VSClient.006005E8             ;  CF=1 时直接从\[esi\]中拷贝;CF=0 时，从\[edi\]中拷贝
006005FB   .  B8 01000000   MOV EAX,1                              ;  EBX 最高位为 0
00600600   >  01DB          ADD EBX,EBX                            ;  拼接为偏移长度 EAX 的 bit 位
00600602   .  75 07        JNZ SHORT VSClient.0060060B
00600604   .  8B1E          MOV EBX,DWORD PTR DS:\[ESI\]
00600606   .  83EE FC       SUB ESI,-4
00600609   .  11DB          ADC EBX,EBX
0060060B   >  11C0          ADC EAX,EAX                            ;  EAX=EAX+EAX+EBX 的最高位
0060060D   .  01DB          ADD EBX,EBX                            ;  EBX 此时移出的 CF 表示是否偏移位;CF=1 表示偏移
0060060F   .  73 0B        JNB SHORT VSClient.0060061C            ;  CF=0 时跳转，需要修正 EAX 偏移
00600611   .  75 28        JNZ SHORT VSClient.0060063B
00600613   .  8B1E          MOV EBX,DWORD PTR DS:\[ESI\]           ;  进行此操作说明已经 ADD EBX ,EBX 32 次,这个控制字节应该从\[ESI\]中读取
00600615   .  83EE FC       SUB ESI,-4
00600618   .  11DB          ADC EBX,EBX
0060061A   .  72 1F        JB SHORT VSClient.0060063B
0060061C   >  48            DEC EAX
0060061D   .  01DB          ADD EBX,EBX
0060061F   .  75 07        JNZ SHORT VSClient.00600628
00600621   .  8B1E          MOV EBX,DWORD PTR DS:\[ESI\]
00600623   .  83EE FC       SUB ESI,-4
00600626   .  11DB          ADC EBX,EBX
00600628   >  11C0          ADC EAX,EAX
0060062A   .^ EB D4        JMP SHORT VSClient.00600600
0060062C   >  01DB          ADD EBX,EBX                            ;  修正数据长度
0060062E   .  75 07        JNZ SHORT VSClient.00600637
00600630   .  8B1E          MOV EBX,DWORD PTR DS:\[ESI\]
00600632   .  83EE FC       SUB ESI,-4
00600635   .  11DB          ADC EBX,EBX
00600637   >  11C9          ADC ECX,ECX
00600639   .  EB 52        JMP SHORT VSClient.0060068D
0060063B   >  31C9          XOR ECX,ECX
0060063D   .  83E8 03       SUB EAX,3
00600640   .  72 11        JB SHORT VSClient.00600653
00600642   .  C1E0 08       SHL EAX,8                               ;  11
00600645   .  8A06          MOV AL,BYTE PTR DS:\[ESI\]             ;  (EAX<<8)+byte ptr \[ESI\]
00600647   .  46            INC ESI
00600648   .  83F0 FF       XOR EAX,FFFFFFFF                       ;  EAX 各位取反,EAX==-1 时解码完毕
0060064B   .  74 75        JE SHORT VSClient.006006C2
0060064D   .  D1F8          SAR EAX,1
0060064F   .  89C5          MOV EBP,EAX
00600651   .  EB 0B        JMP SHORT VSClient.0060065E
00600653   >  01DB          ADD EBX,EBX                            ;  01 情况
00600655   .  75 07        JNZ SHORT VSClient.0060065E
00600657   .  8B1E          MOV EBX,DWORD PTR DS:\[ESI\]
00600659   .  83EE FC       SUB ESI,-4
0060065C   .  11DB          ADC EBX,EBX
0060065E   >^ 72 CC        JB SHORT VSClient.0060062C
00600660   .  41            INC ECX
00600661   .  01DB          ADD EBX,EBX
00600663   .  75 07        JNZ SHORT VSClient.0060066C
00600665   .  8B1E          MOV EBX,DWORD PTR DS:\[ESI\]
00600667   .  83EE FC       SUB ESI,-4
0060066A   .  11DB          ADC EBX,EBX
0060066C   >^ 72 BE        JB SHORT VSClient.0060062C
0060066E   >  01DB          ADD EBX,EBX                            ;  00
00600670   .  75 07        JNZ SHORT VSClient.00600679
00600672   .  8B1E          MOV EBX,DWORD PTR DS:\[ESI\]
00600674   .  83EE FC       SUB ESI,-4
00600677   .  11DB          ADC EBX,EBX
00600679   >  11C9          ADC ECX,ECX
0060067B   .  01DB          ADD EBX,EBX
0060067D   .^ 73 EF        JNB SHORT VSClient.0060066E
0060067F   .  75 09        JNZ SHORT VSClient.0060068A
00600681   .  8B1E          MOV EBX,DWORD PTR DS:\[ESI\]
00600683   .  83EE FC       SUB ESI,-4
00600686   .  11DB          ADC EBX,EBX
00600688   .^ 73 E4        JNB SHORT VSClient.0060066E
0060068A   >  83C1 02       ADD ECX,2
0060068D   >  81FD 00FBFFFF CMP EBP,-500                           ;  后面为从已解压的数据\[edx\]处复制 ecx 字节到\[edi\]中
00600693   .  83D1 02       ADC ECX,2                              ;  此句可见最少复制 2 字节数据
00600696   .  8D142F       LEA EDX,DWORD PTR DS:\[EDI+EBP\]
00600699   .  83FD FC       CMP EBP,-4
0060069C   .  76 0E        JBE SHORT VSClient.006006AC             ;  当偏移大于 4 时，可直接移动 32 位，而非 8 位，速度更快
0060069E   >  8A02          MOV AL,BYTE PTR DS:\[EDX\]             ;  从已经生成的数据\[EDX\]中复制 ECX 个字节到 EDI 中
006006A0   .  42            INC EDX
006006A1   .  8807          MOV BYTE PTR DS:\[EDI\],AL
006006A3   .  47            INC EDI
006006A4   .  49            DEC ECX
006006A5   .^ 75 F7        JNZ SHORT VSClient.0060069E
006006A7   .^ E9 42FFFFFF  JMP VSClient.006005EE
006006AC   >  8B02          MOV EAX,DWORD PTR DS:\[EDX\]
006006AE   .  83C2 04       ADD EDX,4
006006B1   .  8907          MOV DWORD PTR DS:\[EDI\],EAX
006006B3   .  83C7 04       ADD EDI,4
006006B6   .  83E9 04       SUB ECX,4
006006B9   .^ 77 F1        JA SHORT VSClient.006006AC
006006BB   .  01CF          ADD EDI,ECX                             ;  修正数据
006006BD   .^ E9 2CFFFFFF  JMP VSClient.00605EE
006006C2   >  5E            POP ESI                                 ;  解压完毕;修正指令中的相对偏移地址
006006C3   .  89F7          MOV EDI,ESI
006006C5   .  B9 8BA70000   MOV ECX,0A78B
006006CA   >  8A07          MOV AL,BYTE PTR DS:\[EDI\]
006006CC   .  47            INC EDI
006006CD   .  2C E8         SUB AL,0E8
006006CF   >  3C 01         CMP AL,1
006006D1   .^ 77 F7        JA SHORT VSClient.006006CA
006006D3   .  803F 42       CMP BYTE PTR DS:\[EDI\],42              ;  AL==E8(call Jz)或 E9(jmp jz 类型);指令中只有这两个指令是 Jz 类型.J 表示后面有加上 IP 的偏移,z 表示 32 位后面就跟 doubleword
006006D6   .^ 75 F2        JNZ SHORT VSClient.006006CA
006006D8   .  8B07          MOV EAX,DWORD PTR DS:\[EDI\]
006006DA   .  8A5F 04       MOV BL,BYTE PTR DS:\[EDI+4\]
006006DD   .  66:C1E8 08   SHR AX,8
006006E1   .  C1C0 10       ROL EAX,10
006006E4   .  86C4          XCHG AH,AL
006006E6   .  29F8          SUB EAX,EDI
006006E8   .  80EB E8       SUB BL,0E8
006006EB   .  01F0          ADD EAX,ESI
006006ED   .  8907          MOV DWORD PTR DS:\[EDI\],EAX            ;  JMP 地址修改
006006EF   .  83C7 05       ADD EDI,5
006006F2   .  88D8          MOV AL,BL
006006F4   .^ E2 D9        LOOPD SHORT VSClient.006006CF
006006F6   .  8DBE 00A01F00 LEA EDI,DWORD PTR DS:\[ESI+1FA000\]
006006FC   >  8B07          MOV EAX,DWORD PTR DS:\[EDI\]            ;  dll 名字偏移
006006FE   .  09C0          OR EAX,EAX
00600700   .  74 45        JE SHORT VSClient.00600747               ;  IAT 构造以 00000000 结束
00600702   .  8B5F 04       MOV EBX,DWORD PTR DS:\[EDI+4\]          ;  IAT 地址偏移
00600705   .  8D8430 845A20> LEA EAX,DWORD PTR DS:\[EAX+ESI+205A84\]
0060070C   .  01F3          ADD EBX,ESI
0060070E   .  50            PUSH EAX
0060070F   .  83C7 08       ADD EDI,8
00600712   .  FF96 185D2000 CALL DWORD PTR DS:\[ESI+205D18\]        ;  00606D18 >778D2864  kernel32.LoadLibraryA
00600718   .  95            XCHG EAX,EBP
00600719   >  8A07          MOV AL,BYTE PTR DS:\[EDI\]              ;  AL 为同一 dll 函数名分隔符
0060071B   .  47            INC EDI
0060071C   .  08C0          OR AL,AL
0060071E   .^ 74 DC        JE SHORT VSClient.006006FC
00600720   .  89F9          MOV ECX,EDI
00600722   .  79 07        JNS SHORT VSClient.0060072B
00600724   .  0FB707        MOVZX EAX,WORD PTR DS:\[EDI\]
00600727   .  47            INC EDI
00600728   .  50            PUSH EAX
00600729   .  47            INC EDI
0060072A     B9            DB B9
0060072B   .  57            PUSH EDI
0060072C   .  48            DEC EAX
0060072D   .  F2:AE         REPNE SCAS BYTE PTR ES:\[EDI\]
0060072F   .  55            PUSH EBP                                ;  dll handle
00600730   .  FF96 1C5D2000 CALL DWORD PTR DS:\[ESI+205D1C\]
00600736   .  09C0          OR EAX,EAX
00600738   .  74 07        JE SHORT VSClient.00600741               ;  不能得到输入函数
0060073A   .  8903          MOV DWORD PTR DS:\[EBX\],EAX            ;  修正 IAT 为实际 API 地址
0060073C   .  83C3 04       ADD EBX,4
0060073F   .^ EB D8        JMP SHORT VSClient.00600719
00600741   >  FF96 2C5D2000 CALL DWORD PTR DS:\[ESI+205D2C\]        ;  00606D2C >76262AEF  kernel32.ExitProcess
00600747   >  8BAE 205D2000 MOV EBP,DWORD PTR DS:\[ESI+205D20\]     ;  00606D20 >762550AB  kernel32.VirtualProtect
0060074D   .  8DBE 00F0FFFF LEA EDI,DWORD PTR DS:\[ESI-1000\]
00600753   .  BB 00100000   MOV EBX,1000
00600758   .  50            PUSH EAX
00600759   .  54            PUSH ESP
0060075A   .  6A 04         PUSH 4                                  ;  PAGE\_EXECUTE\_READWRITE
0060075C   .  53            PUSH EBX
0060075D   .  57            PUSH EDI
0060075E   .  FFD5          CALL EBP                                ;  获得文件头的写权限
00600760   .  8D87 37020000 LEA EAX,DWORD PTR DS:\[EDI+237\]
00600766   .  8020 7F       AND BYTE PTR DS:\[EAX\],7F              ;  remove sec UPX0 UNINITIALIZED\_DATA character
00600769   .  8060 28 7F    AND BYTE PTR DS:\[EAX+28\],7F           ;  remove sec UPX1 UNINITIALIZED\_DATA character
0060076D   .  58            POP EAX
0060076E   .  50            PUSH EAX
0060076F   .  54            PUSH ESP
00600770   .  50            PUSH EAX                                 ;  文件头原属性
00600771   .  53            PUSH EBX
00600772   .  57            PUSH EDI
00600773   .  FFD5          CALL EBP                                  ;  恢复文件头属性
00600775   .  58            POP EAX
00600776   .  61            POPAD
00600777   .  8D4424 80     LEA EAX,DWORD PTR SS:\[ESP-80\]
0060077B   >  6A 00         PUSH 0
0060077D   .  39C4          CMP ESP,EAX
0060077F   .^ 75 FA        JNZ SHORT VSClient.0060077B
00600781   .  83EC 80       SUB ESP,-80                              ;  同前面循环 80/4 次 push 0 保持 ESP 不变，只是初始化堆栈环境为 0
00600784   .- E9 2122F0FF   JMP VSClient.005029AA                    ;  跳到 OEP
```



目标程序是 VS 游戏竞技平台 2.7 测试版（不太确定，因为现在升级到 3.1，但是 `VSClient.exe` 这个主程序不会变的）。

第一次发贴，不对的地方请大家指正
