

# 秦赢甲胄反盗版加密软件 v1.1 主程序简单脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-480.htm`](https://www.pediy.com/kssd/pediy07/pediy7-480.htm)

【作者大名】csjwaman\[DFCG\]  
  
【使用工具】OD 等经典工具  
  
【操作系统】Windows 2003  
  
【软件名称】秦赢甲胄反盗版加密软件 v1.1 主程序  
  
【下载地址】搜索  
  
【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
  
\--------------------------------------------------------------------------------  
  
【内容】  
  
  
秦赢甲胄反盗版加密软件 v1.1 主程序简单脱壳  
  
  
  
  
  
用 OD 载入主程序 Register.exe，隐藏 OD（否则会很。。。。）。  
  
00416000 >  E8 00000000     CALL Register.00416005///用 OD 载入后停在这里。  
00416005    58              POP EAX  
00416006    05 3B060000     ADD EAX,63B  
0041600B    9C              PUSHFD  
0041600C    50              PUSH EAX  
0041600D    C2 0400         RETN 4  
00416010    55              PUSH EBP  
00416011    8BEC            MOV EBP,ESP  
00416013    56              PUSH ESI  
00416014    57              PUSH EDI  
00416015    53              PUSH EBX  
00416016    34 99           XOR AL,99  
00416018    47              INC EDI  
00416019    49              DEC ECX  
0041601A    34 33           XOR AL,33  
0041601C    EF              OUT DX,EAX   
  
在 12FFC0 处下硬件访问断点，然后 F9 共 9 次，来到：  
  
00416682    E8 03890000     CALL Register.0041EF8A  
00416687    33D2            XOR EDX,EDX/////来到这里。  
00416689    B9 61000000     MOV ECX,61  
0041668E    F7F1            DIV ECX  
00416690    C1E2 03         SHL EDX,3  
00416693    2BE2            SUB ESP,EDX  
00416695    E8 00000000     CALL Register.0041669A  
0041669A    58              POP EAX  
0041669B    05 38000000     ADD EAX,38  
004166A0    E8 00000000     CALL Register.004166A5  
004166A5    5A              POP EDX  
004166A6    81C2 07040000   ADD EDX,407  
  
  
往下可以看到许多这类代码：  
  
004166E4   /0F84 99000000   JE Register.00416783  
004166EA   |E8 00000000     CALL Register.004166EF  
004166EF   |58              POP EAX  
004166F0   |05 11FAFFFF     ADD EAX,-5EF  
004166F5   |F740 38 0100000>TEST DWORD PTR DS:\[EAX+38\],1  
004166FC   |74 40           JE SHORT Register.0041673E  
004166FE   |70 03           JO SHORT Register.00416703  
00416700   |71 01           JNO SHORT Register.00416703  
00416702  ^|78 E8           JS SHORT Register.004166EC  
00416704   |0000            ADD BYTE PTR DS:\[EAX\],AL  
00416706   |0000            ADD BYTE PTR DS:\[EAX\],AL  
00416708   |58              POP EAX  
00416709   |05 36000000     ADD EAX,36  
0041670E   |E8 00000000     CALL Register.00416713  
00416713   |5A              POP EDX  
00416714   |81C2 6D010000   ADD EDX,16D  
0041671A   |E8 63070000     CALL Register.00416E82  
0041671F   |3BC2            CMP EAX,EDX  
00416721   |7A 06           JPE SHORT Register.00416729  
00416723   |7B 04           JPO SHORT Register.00416729  
00416725   |8562 7F         TEST DWORD PTR DS:\[EDX+7F\],ESP  
00416728  ^|E1 E8           LOOPDE SHORT Register.00416712/////解码。  
0041672A   |0000            ADD BYTE PTR DS:\[EAX\],AL  
0041672C   |0000            ADD BYTE PTR DS:\[EAX\],AL  
  
  
一直向下找，来到：  
  
00416D56    B9 3DAE0000     MOV ECX,0AE3D////再向下已经没有解码代码了。  
00416D5B    E8 00000000     CALL Register.00416D60  
00416D60    5F              POP EDI  
00416D61    81C7 BC000000   ADD EDI,0BC  
00416D67    F3:AA           REP STOS BYTE PTR ES:\[EDI\]  
00416D69    B9 810D0000     MOV ECX,0D81  
00416D6E    E8 00000000     CALL Register.00416D73  
00416D73    5F              POP EDI  
00416D74    81C7 8DF2FFFF   ADD EDI,-0D73  
00416D7A    23C1            AND EAX,ECX  
00416D7C    70 03           JO SHORT Register.00416D81  
00416D7E    71 01           JNO SHORT Register.00416D81  
00416D80    3E:F3:AA        REP STOS BYTE PTR ES:\[EDI\]  
00416D83    C3              RETN/////F4 直接到这里。  
00416D84    56              PUSH ESI  
00416D85    57              PUSH EDI  
00416D86    53              PUSH EBX  
00416D87    E8 00000000     CALL Register.00416D8C  
00416D8C    5E              POP ESI  
  
返回后来到：  
  
0040E10F    C3              RETN////返回到这里。F7。  
0040E110    F2:             PREFIX REPNE:                            ; 多余的前缀  
0040E111    FFFF            ???                                      ; 未知命令  
0040E113    CC              INT3  
  
返回到：  
  
0040DAA9    C3              RETN////继续返回！  
0040DAAA    8D4D F0         LEA ECX,DWORD PTR SS:\[EBP-10\]  
0040DAAD  ^ E9 98F5FFFF     JMP Register.0040D04A                    ; JMP to MFC42.#800  
0040DAB2    B8 E8024100     MOV EAX,Register.004102E8  
0040DAB7  ^ E9 1AF9FFFF     JMP Register.0040D3D6                    ; JMP to   
  
msvcrt.\_\_CxxFrameHandler2  
0040DABC    CC              INT3  
  
返回到：  
  
0040D818    C3              RETN////继续返回！  
0040D819    8D4D E8         LEA ECX,DWORD PTR SS:\[EBP-18\]  
0040D81C  ^ E9 29F8FFFF     JMP Register.0040D04A                    ; JMP to MFC42.#800  
0040D821    8D8D 38FDFFFF   LEA ECX,DWORD PTR SS:\[EBP-2C8\]  
0040D827  ^ E9 B446FFFF     JMP Register.00401EE0  
0040D82C    8D8D 38FDFFFF   LEA ECX,DWORD PTR SS:\[EBP-2C8\]  
  
如此返回一直来到：  
  
0040D46F    55              PUSH EBP////入口到了！！！  
0040D470    8BEC            MOV EBP,ESP  
0040D472    6A FF           PUSH -1  
0040D474    68 A0FE4000     PUSH Register.0040FEA0  
0040D479    68 D4D54000     PUSH Register.0040D5D4        ; JMP to msvcrt.\_except\_handler3  
0040D47E    64:A1 00000000  MOV EAX,DWORD PTR FS:\[0\]  
0040D484    50              PUSH EAX  
0040D485    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP  
0040D48C    83EC 68         SUB ESP,68  
0040D48F    53              PUSH EBX  
0040D490    56              PUSH ESI  
0040D491    57              PUSH EDI  
0040D492    8965 E8         MOV DWORD PTR SS:\[EBP-18\],ESP  
0040D495    33DB            XOR EBX,EBX  
0040D497    895D FC         MOV DWORD PTR SS:\[EBP-4\],EBX  
0040D49A    6A 02           PUSH 2  
0040D49C    FF15 C0F34000   CALL NEAR DWORD PTR DS:\[40F3C0\]          ; msvcrt.\_\_set\_app\_type  
0040D4A2    59              POP ECX  
0040D4A3    830D C0354100 F>OR DWORD PTR DS:\[4135C0\],FFFFFFFF  
0040D4AA    830D C4354100 F>OR DWORD PTR DS:\[4135C4\],FFFFFFFF  
0040D4B1    FF15 BCF34000   CALL NEAR DWORD PTR DS:\[40F3BC\]          ; msvcrt.\_\_p\_\_fmode  
0040D4B7    8B0D B4354100   MOV ECX,DWORD PTR DS:\[4135B4\]  
  
在入口点 DUMP 后，用 ImportREC 修复，入口 RVA 填：D46F，IAT 的 RVA 填：F000，大小：1000（懒人大法:)），搜  
  
索后有许多无效，追踪后把无效的直接 CUT，然后修复。  
  
  
  
\--------------------------------------------------------------------------------  
  
【总结】  
  
总结：这个壳儿作者有点言过其实，强度太低！反跟踪也很简单，只是检查调试标志和检查 API 的前 5 个字节是  
  
否下断，还有文件名检验。下面就是检查 API 前 5 个字节是否下断的代码：  
  
0041DF99    81C1 6881FFFF   ADD ECX,FFFF8168  
0041DF9F    F741 2C 0800000>TEST DWORD PTR DS:\[ECX+2C\],8  
0041DFA6    75 02           JNZ SHORT Register.0041DFAA  
0041DFA8    FFE0            JMP NEAR EAX  
0041DFAA    8038 CC         CMP BYTE PTR DS:\[EAX\],0CC////检测。  
0041DFAD    74 1E           JE SHORT Register.0041DFCD  
0041DFAF    8078 01 CC      CMP BYTE PTR DS:\[EAX+1\],0CC  
0041DFB3    74 18           JE SHORT Register.0041DFCD  
0041DFB5    8078 02 CC      CMP BYTE PTR DS:\[EAX+2\],0CC  
0041DFB9    74 12           JE SHORT Register.0041DFCD  
0041DFBB    8078 03 CC      CMP BYTE PTR DS:\[EAX+3\],0CC  
0041DFBF    74 0C           JE SHORT Register.0041DFCD  
0041DFC1    8078 04 CC      CMP BYTE PTR DS:\[EAX+4\],0CC  
0041DFC5    74 06           JE SHORT Register.0041DFCD  
0041DFC7    50              PUSH EAX  
0041DFC8    C3              RETN  
  
  
\--------------------------------------------------------------------------------  
  
【版权声明】本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

IAT 加密 CALL：  
0041A39E    8B16            MOV EDX,DWORD PTR DS:\[ESI\]  
0041A3A0    8D4424 28       LEA EAX,DWORD PTR SS:\[ESP+28\]  
0041A3A4    52              PUSH EDX  
0041A3A5    50              PUSH EAX  
0041A3A6    8BCB            MOV ECX,EBX  
0041A3A8    E8 78FAFFFF     CALL Register.00419E25  
0041A3AD    8B08            MOV ECX,DWORD PTR DS:\[EAX\]  
0041A3AF    57              PUSH EDI  
0041A3B0    890F            MOV DWORD PTR DS:\[EDI\],ECX  
0041A3B2    8BCB            MOV ECX,EBX  
0041A3B4    E8 4EFCFFFF     CALL Register.0041A007////这个就是加密 CALL 了。  
0041A3B9    8B93 00020000   MOV EDX,DWORD PTR DS:\[EBX+200\]           ; Register.00416100  
0041A3BF    F642 38 08      TEST BYTE PTR DS:\[EDX+38\],8  
0041A3C3    74 11           JE SHORT Register.0041A3D6  
0041A3C5    3BF7            CMP ESI,EDI  
0041A3C7    74 0D           JE SHORT Register.0041A3D6  
0041A3C9    6A 04           PUSH 4  
0041A3CB    68 CC000000     PUSH 0CC  
0041A3D0    56              PUSH ESI  
0041A3D1    E8 77F1FFFF     CALL Register.0041954D  
0041A3D6    8B4424 14       MOV EAX,DWORD PTR SS:\[ESP+14\]  
0041A3DA    83C6 04         ADD ESI,4  
0041A3DD    83C7 04         ADD EDI,4 

 4dd466d0e1e0cc123afbe504da2462ca